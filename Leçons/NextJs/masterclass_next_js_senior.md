# Masterclass Next.js — Niveau Senior & Staff Engineer

À l'échelle industrielle, utiliser Next.js ne se résume pas à empiler des composants dans le dossier `app/`. Ce niveau exige de maîtriser la mécanique interne des **4 couches de cache**, l'architecture distribuée sur le runtime **Edge / Node.js**, le **Partial Prerendering (PPR)**, la sécurité avancée des mutations serveur et les patterns de navigation complexe (*Parallel & Intercepting Routes*).

---

## 1. Topologie des 4 Niveaux de Cache de Next.js

L'App Router intègre quatre mécanismes de cache distincts, fonctionnant de concert entre le serveur, le CDN et le navigateur :

```
                  ┌───────────────────────────────────────────────┐
Client            │ 1. Router Cache (In-Memory, Navigateur)      │
(Browser)         └───────────────────────┬───────────────────────┘
                                          │ Cache Miss
                                          ▼
                  ┌───────────────────────────────────────────────┐
Serveur           │ 2. Full Route Cache (RSC Payload + HTML build)│
(Edge / Node.js)  └───────────────────────┬───────────────────────┘
                                          │ Cache Miss / Route Dynamique
                                          ▼
                  ┌───────────────────────────────────────────────┐
                  │ 3. Request Memoization (Dédoublonnage React)  │
                  └───────────────────────┬───────────────────────┘
                                          │ Exécution fetch()
                                          ▼
                  ┌───────────────────────────────────────────────┐
Stockage          │ 4. Data Cache (Persistant cross-requests)     │
Persistant        └───────────────────────────────────────────────┘
```

### 1.1 Détail technique des couches

| Couche | Emplacement | Durée de vie | Rôle & Déclencheur |
| :--- | :--- | :--- | :--- |
| **Request Memoization** | Mémoire Serveur (React) | Cycle de vie d'une requête HTTP | Déduplique les appels `fetch` identiques dans un même arbre de composants (évite de propager les props). |
| **Data Cache** | Disque / Cache distant persistant | Indéfini (ou TTL `revalidate`) | Stocke les réponses d'API/données entre requêtes et utilisateurs distincts. |
| **Full Route Cache** | Serveur / CDN | Jusqu'au prochain déploiement ou revalidation | Mémorise le HTML et le payload RSC des routes statiques. |
| **Router Cache** | Mémoire Client (Navigateur) | Session active (quelques minutes) | Stocke les segments visités et préchargés pour des transitions instantanées. |

### 1.2 Invalidation Granulaire avec Tags de Cache

Pour casser le couplage entre l'URL et les données, utilisez des étiquettes sémantiques (*Cache Tags*) :

```typescript
// Récupération de données avec tag
async function getInventory(warehouseId: string) {
  const res = await fetch(`https://api.internal/inventory/${warehouseId}`, {
    next: {
      tags: [`inventory-${warehouseId}`, 'warehouse-global'],
      revalidate: 86400, // 24h par défaut
    },
  });
  return res.json();
}

// Invalidation chirurgicale dans une Server Action
import { revalidateTag } from 'next/cache';

export async function updateStock(warehouseId: string, itemId: string, count: number) {
  'use server';
  
  await db.stock.update({ where: { itemId }, data: { count } });
  
  // N'invalide QUE ce warehouse sans purger tout le site ni reconstruire les routes
  revalidateTag(`inventory-${warehouseId}`);
}
```

---

## 2. Partial Prerendering (PPR) : L'Union du Statique et du Dynamique

Le **PPR** résout le dilemme historique entre tout-statique (rapide mais impersonnel) et tout-dynamique (personnalisé mais avec un TTFB dégradé).

### 2.1 Principe d'exécution

* Next.js génère un **coquillage statique instantané** (Header, layout, skeleton de navigation) servi depuis le CDN.
* Les morceaux dynamiques enveloppés dans des `<Suspense>` sont **streamés en tâche de fond** au cours de la même connexion HTTP sans blocage.

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';
import { StaticHeader } from '@/components/static-header';
import { LiveFinancialFeed } from '@/components/live-feed';
import { MetricSkeleton } from '@/components/skeletons';

export const experimental_ppr = true;

export default function DashboardPage() {
  return (
    <main>
      {/* 1. Servie immédiatement depuis le cache CDN (0ms TTFB) */}
      <StaticHeader />

      {/* 2. Trou percé dans le coquillage statique : streamé à la volée */}
      <Suspense fallback={<MetricSkeleton />}>
        <LiveFinancialFeed />
      </Suspense>
    </main>
  );
}
```

---

## 3. Middleware & Edge Runtime : Limites et Stratégies

Le middleware Next.js (`middleware.ts`) s'exécute **avant** que la requête n'atteigne le cache ou les composants de route.

### 3.1 Contraintes architecturales de l'Edge Runtime

* Pas d'APIs Node.js complètes (pas de `fs`, `child_process`, ni de modules C++ natifs).
* Taille binaire limitée (généralement 1 Mo ou 4 Mo selon les plateformes cloud).
* Durée d'exécution restreinte (quelques millisecondes avant timeout).

### 3.2 Cas d'usage : A/B Testing, Géolocalisation & Auth Session Shunting

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export const config = {
  matcher: ['/checkout/:path*', '/dashboard/:path*'],
};

export async function middleware(request: NextRequest) {
  const sessionToken = request.cookies.get('session_token')?.value;

  // Shunt d'authentification pré-serveur : économise le CPU des Server Components
  if (!sessionToken && request.nextUrl.pathname.startsWith('/dashboard')) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('from', request.nextUrl.pathname);
    return NextResponse.redirect(loginUrl);
  }

  // Feature Flagging / A/B Testing sans clignotement (Bucket routing)
  const bucket = request.cookies.get('ab_bucket')?.value || (Math.random() < 0.5 ? 'A' : 'B');
  const response = NextResponse.next();
  
  if (!request.cookies.has('ab_bucket')) {
    response.cookies.set('ab_bucket', bucket, { path: '/', httpOnly: true });
  }

  // Rewrite interne transparent : l'URL dans le navigateur ne change pas
  if (bucket === 'B' && request.nextUrl.pathname === '/checkout') {
    return NextResponse.rewrite(new URL('/checkout/variant-b', request.url));
  }

  return response;
}
```

---

## 4. Patterns Avancés de Routage : Parallel & Intercepting Routes

Ces patterns permettent de concevoir des interfaces riches (modales partageables par URL, dashboards multi-volets) tout en conservant une synchronisation parfaite avec l'historique du navigateur.

### 4.1 Structure de dossiers

```
app/
└── feed/
    ├── @modal/                 # Slot parallèle pour les modales
    │   ├── (.)photo/[id]/      # Interception d'un niveau relatif
    │   │   └── page.tsx        # Affiché quand on navigue depuis /feed
    │   └── default.tsx         # Fallback si aucune modale n'est active
    ├── layout.tsx              # Reçoit { children, modal }
    ├── page.tsx                # Liste des photos
    └── photo/[id]/
        └── page.tsx            # Affiché en pleine page sur un rechargement direct (F5)
```

### 4.2 Implémentation du Layout Parallèle

```tsx
// app/feed/layout.tsx
export default function FeedLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <div className="feed-layout">
      {children}
      {modal}
    </div>
  );
}
```

* Si l'utilisateur clique sur un lien `<Link href="/photo/42">` depuis `/feed`, Next.js intercepte la route et injecte la page modale dans `@modal` sans quitter la page de fond.
* Si l'utilisateur partage l'URL `/photo/42` ou fait `F5`, Next.js sert directement `app/feed/photo/[id]/page.tsx` en plein écran.

---

## 5. Sécurité Avancée des Server Actions

Les Server Actions sont des **endpoints HTTP publics exposés** générés à la volée. Ne faites jamais aveuglément confiance aux arguments reçus.

### 5.1 Pattern d'Action Sécurisée et Validée

```typescript
// lib/safe-action.ts
import { z } from 'zod';

export function createSafeAction<TInput, TOutput>(
  schema: z.ZodSchema<TInput>,
  handler: (input: TInput, ctx: { userId: string }) => Promise<TOutput>
) {
  return async (rawInput: unknown): Promise<{ data?: TOutput; error?: string }> => {
    // 1. Contrôle d'authentification strict
    const session = await getSession();
    if (!session?.userId) {
      return { error: 'Non authentifié' };
    }

    // 2. Validation structurelle runtime
    const parsed = schema.safeParse(rawInput);
    if (!parsed.success) {
      return { error: `Validation invalide: ${parsed.error.issues.map(i => i.message).join(', ')}` };
    }

    try {
      const result = await handler(parsed.data, { userId: session.userId });
      return { data: result };
    } catch (err: any) {
      console.error('[Action Error]', err);
      return { error: 'Une erreur interne est survenue.' };
    }
  };
}
```

---

## 6. Checklist de Résilience Senior en Production

1. **Isolation des variables d'environnement :** Utilisez le paquet expérimental ou standard `server-only` (`import 'server-only'`) dans tous les modules manipulant des secrets pour bloquer à la compilation toute importation accidentelle côté client.
2. **Mesure des métriques Core Web Vitals :** Utilisez `useReportWebVitals` pour surveiller l'INP (*Interaction to Next Paint*) et le LCP (*Largest Contentful Paint*).
3. **Optimisation des polices & icônes :** Utilisez `next/font` avec sous-ensemblement local (*zero external layout shift*).
4. **Définition explicite des temps limites :** Configurez `maxDuration` sur vos Route Handlers et Server Actions pour éviter les timeouts silencieux des fonctions serverless.