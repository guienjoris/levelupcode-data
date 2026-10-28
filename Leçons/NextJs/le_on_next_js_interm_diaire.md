# Masterclass Next.js — Niveau Intermédiaire (App Router)

Après avoir maîtrisé les bases (arborescence du dossier `app/`, layouts et frontière Server/Client Components), le palier intermédiaire nécessite de dompter le cycle de données : **routes dynamiques**, **stratégies de mise en cache**, **mutations avec les Server Actions**, et **gestion native du streaming et des erreurs**.

---

## 1. Routes Dynamiques & Paramètres d'URL

### 1.1 Segments dynamiques simples et Catch-all

Pour capturer des segments variables dans l'URL :

- **Segment simple** : `app/blog/[slug]/page.tsx` $\rightarrow$ capture `/blog/mon-article`
- **Catch-all** : `app/docs/[...slug]/page.tsx` $\rightarrow$ capture `/docs/api/v1/auth`
- **Optional Catch-all** : `app/shop/[[...categories]]/page.tsx` $\rightarrow$ capture `/shop`, `/shop/vetements`, `/shop/vetements/homme`

### 1.2 Récupération des paramètres (Page Props)

Dans les versions modernes de Next.js (Next.js 15+), `params` et `searchParams` sont fournis sous forme de **Promesses** dans les composants `page` et `layout` :

```tsx
// app/blog/[slug]/page.tsx
interface PageProps {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}

export default async function BlogPostPage({ params, searchParams }: PageProps) {
  const { slug } = await params;
  const { ref } = await searchParams;

  return (
    <article className="p-6">
      <h1 className="text-2xl font-bold">Article : {slug}</h1>
      {ref && <p className="text-sm text-gray-500">Source de provenance : {ref}</p>}
    </article>
  );
}
```

### 1.3 `generateStaticParams` (Pré-rendu statique)

Pour pré-générer les pages dynamiques au moment du build (SSG / Static Site Generation) plutôt qu'à la demande :

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(res => res.json());

  return posts.map((post: { slug: string }) => ({
    slug: post.slug,
  }));
}
```

---

## 2. Le Modèle de Cache et de Revalidation

Next.js utilise une couche d'abstraction poussée au-dessus de l'API standard `fetch`. Comprendre ces options évite d'afficher des données obsolètes ou de saturer vos serveurs.

```
Requête de données (fetch)
        │
        ├── cache: 'force-cache'  ──► Cache statique permanent (Build-time / SSG)
        │
        ├── next: { revalidate: 60 } ──► ISR (Mise à jour en arrière-plan toutes les 60s)
        │
        └── cache: 'no-store'     ──► SSR dynamique (Nouvelle requête à chaque visite)
```

### 2.1 Les trois modes de fetch

```tsx
// 1. Mise en cache par défaut (Statique)
const staticData = await fetch('https://api.example.com/data', {
  cache: 'force-cache',
});

// 2. Incremental Static Regeneration (ISR) : revalidation par intervalle de temps
const revalidatedData = await fetch('https://api.example.com/products', {
  next: { revalidate: 3600 }, // Rafraîchit au maximum une fois par heure
});

// 3. Dynamique pur (SSR) : recalculé à chaque requête HTTP
const dynamicData = await fetch('https://api.example.com/user-cart', {
  cache: 'no-store',
});
```

### 2.2 Revalidation à la demande par Tag (`revalidateTag`)

Au lieu d'attendre un timer fixe, vous pouvez étiqueter vos requêtes et invalider le cache uniquement lors d'un événement métier (ex: publication d'un article) :

```tsx
// Récupération avec étiquette :
const data = await fetch('https://api.example.com/articles', {
  next: { tags: ['articles-list'] },
});
```

---

## 3. Server Actions : Mutations modernes sans API REST

Les **Server Actions** sont des fonctions asynchrones qui s'exécutent exclusivement sur le serveur. Elles s'utilisent directement dans des formulaires ou des gestionnaires d'événements, sans devoir créer de routes API d'intermédiaire.

### 3.1 Déclaration d'une Server Action

```ts
// app/actions/create-todo.ts
'use server';

import { revalidateTag } from 'next/cache';

export async function createTodo(formData: FormData) {
  const title = formData.get('title') as string;

  if (!title || title.trim().length === 0) {
    throw new Error('Le titre est requis');
  }

  // Enregistrement direct en base de données (ex: Prisma, Drizzle, SQL)
  await db.todo.create({ data: { title } });

  // Invalidation ciblée du cache pour rafraîchir l'interface
  revalidateTag('todos');
}
```

### 3.2 Utilisation dans un formulaire client avec retour d'état (`useActionState`)

```tsx
// app/todos/todo-form.tsx
'use client';

import { useActionState } from 'react';
import { createTodo } from '@/app/actions/create-todo';

export function TodoForm() {
  // useActionState encapsule l'action, l'état initial et gère l'état d'attente (isPending)
  const [state, formAction, isPending] = useActionState(createTodo, null);

  return (
    <form action={formAction} className="flex gap-2">
      <input
        type="text"
        name="title"
        placeholder="Nouvelle tâche..."
        required
        className="border px-3 py-1 rounded"
      />
      <button 
        type="submit" 
        disabled={isPending}
        className="bg-blue-600 text-white px-4 py-1 rounded disabled:opacity-50"
      >
        {isPending ? 'Enregistrement...' : 'Ajouter'}
      </button>
    </form>
  );
}
```

---

## 4. Streaming, Suspense et Fichiers Réservés

Next.js exploite React Suspense pour diffuser progressivement le HTML vers le navigateur (*Streaming SSR*).

```
app/dashboard/
├── layout.tsx
├── page.tsx          # Composant de page principal
├── loading.tsx       # Fallback affiché automatiquement pendant le chargement
├── error.tsx         # Fallback en cas d'exception non gérée (Client Component obligatoire)
└── not-found.tsx     # Affiché lors d'un appel à notFound()
```

### 4.1 `loading.tsx` (Streaming instantané)

Dès qu'un visiteur arrive sur `/dashboard`, Next.js sert immédiatement le squelette défini dans `loading.tsx` pendant que les composants asynchrones de `page.tsx` finissent leurs requêtes côté serveur :

```tsx
// app/dashboard/loading.tsx
export default function DashboardLoading() {
  return (
    <div className="animate-pulse space-y-4 p-6">
      <div className="h-8 bg-gray-200 rounded w-1/4"></div>
      <div className="h-32 bg-gray-200 rounded"></div>
    </div>
  );
}
```

### 4.2 `error.tsx` (Error Boundary automatique)

Le fichier `error.tsx` doit **impérativement être un Client Component** (`'use client'`) afin de permettre à l'utilisateur de réinitialiser l'affichage :

```tsx
// app/dashboard/error.tsx
'use client';

import { useEffect } from 'react';

export default function DashboardError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error('Erreur interceptée :', error);
  }, [error]);

  return (
    <div className="p-6 border border-red-200 bg-red-50 rounded">
      <h2 className="text-red-700 font-semibold">Une erreur est survenue !</h2>
      <button
        onClick={() => reset()}
        className="mt-3 px-3 py-1 bg-red-600 text-white text-sm rounded"
      >
        Réessayer
      </button>
    </div>
  );
}
```

---

## 5. Route Handlers (API Endpoints natifs)

Si vous devez exposer une API REST consommable par des tiers, des applications mobiles ou gérer des webhooks Stripe, utilisez un fichier **`route.ts`** (au lieu de `page.tsx`) :

```ts
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const payload = await request.json();

    // Logique de traitement du webhook
    return NextResponse.json({ received: true }, { status: 200 });
  } catch (err) {
    return NextResponse.json(
      { error: 'Payload invalide' },
      { status: 400 }
    );
  }
}

export async function GET() {
  return NextResponse.json({ status: 'API fonctionnelle' });
}
```

---

## Synthèse des Bonnes Pratiques Intermédiaires

1. **Privilégiez les Server Actions** pour les mutations internes à l'application plutôt que de créer des routes API manuelles.
2. **Utilisez `generateStaticParams`** pour pré-compiler les pages dynamiques connues et améliorer les temps de réponse au maximum.
3. **Associez `revalidateTag`** aux Server Actions pour contrôler l'invalidation du cache de manière chirurgicale.
4. **Isolez les zones lentes** grâce à `<Suspense fallback={<Skeleton />}>` directement dans votre JSX plutôt que de bloquer toute la page.