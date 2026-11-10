# Next.js Débutant — Pratique App Router : Fichiers Réservés, Optimisations & SEO

Avec l'App Router de Next.js, concevoir une application moderne repose sur des conventions de fichiers intégrées et des composants d'optimisation natifs. Cette leçon détaille comment gérer les états de chargement, rattraper les erreurs, optimiser le chargement des images/polices et déclarer le référencement naturel (SEO) sans effort.

---

## 1. Les Fichiers Spéciaux de la Hiérarchie de Route

Dans l'App Router, chaque dossier dans `app/` représente un segment d'URL. Au-delà de `page.tsx` et `layout.tsx`, Next.js réserve des noms de fichiers précis pour structurer l'expérience utilisateur de manière déclarative.

```
app/
├── blog/
│   ├── page.tsx         <-- Contenu de la route /blog
│   ├── loading.tsx      <-- Écran d'attente (React Suspense)
│   ├── error.tsx        <-- Barrière d'erreur (Error Boundary)
│   └── not-found.tsx    <-- Vue 404 dédiée à cette section
```

### 1.1 `loading.tsx` : Suspense instantané

`loading.tsx` encapsule automatiquement le contenu de `page.tsx` dans une frontière `<Suspense>` de React. Dès que l'utilisateur navigue vers la page, le squelette d'attente s'affiche immédiatement pendant que les données du serveur sont récupérées.

```tsx
// app/blog/loading.tsx
export default function BlogLoading() {
  return (
    <div className="skeleton-container">
      <div className="skeleton-title"></div>
      <div className="skeleton-card"></div>
      <div className="skeleton-card"></div>
    </div>
  );
}
```

### 1.2 `error.tsx` : Isolation des pannes

`error.tsx` crée un *Error Boundary* côté client autour de la page. Si la requête vers la base de données ou l'API échoue, seule la section concernée affiche un message d'erreur, sans faire planter le reste du site ni la barre de navigation.

> **Règle impérative :** `error.tsx` doit **obligatoirement** être un Client Component (`'use client'`).

```tsx
// app/blog/error.tsx
'use client';

interface ErrorProps {
  error: Error & { digest?: string };
  reset: () => void;
}

export default function BlogError({ error, reset }: ErrorProps) {
  return (
    <div className="error-box">
      <h2>Impossible de charger les articles</h2>
      <p>{error.message}</p>
      {/* reset() retente le rendu du segment de page */}
      <button onClick={() => reset()}>Réessayer</button>
    </div>
  );
}
```

### 1.3 `not-found.tsx` et la fonction `notFound()`

Pour déclencher une page 404 lorsqu'une ressource précise n'existe pas en base :

```tsx
// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation';

async function getPost(slug: string) {
  const res = await fetch(`https://api.monsite.fr/posts/${slug}`);
  if (!res.ok) return null;
  return res.json();
}

export default async function BlogPostPage({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await getPost(slug);

  if (!post) {
    notFound(); // Interrompt le rendu et affiche le not-found.tsx le plus proche
  }

  return <article><h1>{post.title}</h1></article>;
}
```

---

## 2. Navigation Ultra-Rapide avec `next/link`

N'utilisez jamais la balise HTML standard `<a href="...">` pour la navigation interne. 

Le composant `<Link>` de Next.js apporte trois avantages majeurs :
1. **Navigation côté client (SPA) :** La page ne recharge pas entièrement le navigateur.
2. **Préchargement automatique (*Prefetching*) :** Dès qu'un lien `<Link>` entre dans le champ visuel (viewport) de l'utilisateur en production, Next.js précharge le code et les données de la page cible en tâche de fond.
3. **Préservation de l'état :** L'état React du layout parent n'est pas réinitialisé.

```tsx
import Link from 'next/link';

export function NavigationBar() {
  return (
    <nav>
      {/* Navigation instantanée avec préchargement automatique */}
      <Link href="/blog">Actualités</Link>
      
      {/* Désactiver le prefetch si la page est rarement consultée */}
      <Link href="/conditions-generales" prefetch={false}>
        Mentions Légales
      </Link>
    </nav>
  );
}
```

---

## 3. Optimisation des Images avec `next/image`

L'élément HTML `<img src="..." />` traditionnel est l'une des premières causes de dégradation des scores Web Vitals (images trop lourdes, décalage visuel de mise en page).

Le composant `<Image />` résout ces problèmes automatiquement :
* **Formats modernes :** Conversion à la volée en WebP ou AVIF selon le navigateur.
* **Redimensionnement adapté :** Envoi d'une version réduite aux écrans de smartphones (*responsive srcSet*).
* **Prévention du CLS (*Cumulative Layout Shift*) :** L'obligation de définir `width` et `height` réserve l'espace dans la page avant le téléchargement.
* **Lazy loading natif :** Les images sous la ligne de flottaison ne sont chargées que lorsque l'utilisateur scrolle.

```tsx
import Image from 'next/image';
import photoProfil from '@/public/avatar.jpg';

export function ProfilCard() {
  return (
    <div>
      {/* 1. Image locale statique : width et height calculés automatiquement */}
      <Image 
        src={photoProfil} 
        alt="Photo de profil" 
        placeholder="blur" // Effet de flou progressif au chargement
      />

      {/* 2. Image distante : dimensions obligatoires + configuration du domaine requise */}
      <Image
        src="https://images.unsplash.com/photo-12345"
        alt="Bannière d'en-tête"
        width={800}
        height={400}
        priority // Charge l'image en priorité haute si elle est tout en haut de la page (LCP)
      />
    </div>
  );
}
```

> **Configuration requise pour les images distantes :** Dans `next.config.ts`, vous devez explicitement autoriser les noms d'hôtes tiers (`remotePatterns`) pour bloquer les requêtes malveillantes.

---

## 4. Typographies Sans Décalage avec `next/font`

Télécharger des polices depuis des CDNs externes (comme Google Fonts) bloque souvent le rendu du texte ou provoque un clignotement (*FOUT - Flash of Unstyled Text*).

`next/font` télécharge les polices **au moment du build** et les héberge localement avec les autres fichiers statiques :

```tsx
// app/layout.tsx
import { Inter } from 'next/font/google';

// Chargement de la police avec sous-ensemble optimisé
const inter = Inter({ 
  subsets: ['latin'],
  display: 'swap',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fr" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

---

## 5. Gestion Déclarative du SEO (Metadata API)

Next.js remplace les balises `<head>` manuelles par une API de métadonnées déclarative, disponible dans les Server Components.

### 5.1 Métadonnées statiques

```tsx
// app/a-propos/page.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'À Propos — MonEntreprise',
  description: 'Découvrez notre histoire et notre équipe de développeurs.',
  openGraph: {
    title: 'À Propos — MonEntreprise',
    images: ['/og-about.jpg'],
  },
};

export default function AboutPage() {
  return <h1>Notre Histoire</h1>;
}
```

### 5.2 Métadonnées dynamiques avec `generateMetadata`

Pour les pages produits ou articles de blog dépendant d'un paramètre d'URL :

```tsx
// app/produits/[id]/page.tsx
import type { Metadata } from 'next';

interface PageProps {
  params: Promise<{ id: string }>;
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const { id } = await params;
  const produit = await fetch(`https://api.monsite.fr/produits/${id}`).then(r => r.json());

  return {
    title: `${produit.name} | Boutique Officielle`,
    description: produit.summary,
  };
}

export default async function ProduitPage({ params }: PageProps) {
  const { id } = await params;
  return <div>Fiche du produit {id}</div>;
}
```

---

## Synthèse des Bonnes Pratiques

| Outil / Fichier | Utilité Principale | Règle d'or |
| :--- | :--- | :--- |
| `loading.tsx` | Affichage immédiat d'un squelette d'attente | Créé automatiquement un boundary `Suspense`. |
| `error.tsx` | Capture les erreurs sans crasher le layout | Doit toujours comporter la directive `'use client'`. |
| `<Link>` | Navigation fluide côté client | Préférer à `<a>` pour bénéficier du préchargement en production. |
| `<Image />` | Optimisation WebP/AVIF et prévention du CLS | Toujours indiquer les dimensions ou utiliser `fill`. |
| `generateMetadata` | Balises SEO dynamiques personnalisées | S'exécute côté serveur avant l'envoi du HTML. |