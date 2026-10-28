# Bien Débuter avec Next.js (Niveau Novice)

Next.js est le framework React de référence pour la production. Alors que React pur s'exécute traditionnellement dans le navigateur de l'utilisateur (client-side rendering), Next.js apporte un cadre complet avec un serveur Node.js intégré, optimisant le référencement naturel (**SEO**), la vitesse de chargement et l'organisation du code.

---

## 1. Pourquoi utiliser Next.js plutôt que React seul ?

Avec une application React classique (créée par exemple avec Vite) :
- Le serveur renvoie une page HTML quasi vide : `<div id="root"></div>`.
- Le navigateur doit télécharger l'intégralité du JavaScript avant d'afficher quoi que ce soit.
- Les robots d'indexation (Google) ont du mal à lire le contenu immédiatement.

Avec Next.js :
- **Rendu côté serveur (SSR / Prerendering)** : Le HTML est pré-généré sur le serveur et arrive complet dans le navigateur.
- **Routage basé sur les dossiers** : Plus besoin de configurer manuellement une bibliothèque tierce comme `react-router`.
- **Performances automatisées** : Optimisation native des images, des polices de caractères et du chargement différé des scripts.

---

## 2. Le Routage par Fichiers : L'App Router

Depuis Next.js 13+, le système de routage standard repose sur le dossier `app/`. Chaque dossier dans `app` correspond à un segment de l'URL.

### 2.1 Les conventions de fichiers

Pour qu'une URL soit accessible, le dossier doit contenir un fichier spécial nommé **`page.js`** (ou `page.tsx`) :

```text
app/
├── layout.js       # Squelette partagé (header, footer, html, body)
├── page.js         # Page d'accueil ("/")
├── about/
│   └── page.js     # Page À propos ("/about")
└── blog/
    ├── page.js     # Liste des articles ("/blog")
    └── [id]/
        └── page.js # Page dynamique ("/blog/123", "/blog/mon-titre")
```

### 2.2 Créer sa première page

Dans Next.js, une page est un simple composant React exporté par défaut :

```jsx
// app/about/page.jsx
export default function AboutPage() {
  return (
    <main>
      <h1>À propos de nous</h1>
      <p>Bienvenue sur notre premier site avec Next.js !</p>
    </main>
  );
}
```

---

## 3. Le Layout Racine (`layout.js`)

Le fichier **`app/layout.js`** est obligatoire : il définit la structure globale (balises `<html>` et `<body>`) partagée par toutes les pages. Il évite de recharger le menu ou le footer lors des changements de page.

```jsx
// app/layout.jsx
export const metadata = {
  title: "Mon Super Site",
  description: "Créé avec Next.js et React",
};

export default function RootLayout({ children }) {
  return (
    <html lang="fr">
      <body>
        <header>
          <nav>Mon Menu</nav>
        </header>
        
        {/* children représente la page actuellement affichée */}
        {children}

        <footer>© 2026 Mon Site Web</footer>
      </body>
    </html>
  );
}
```

---

## 4. Navigation : Le composant `<Link>`

En Next.js, on n'utilise jamais la balise native `<a href="...">` pour naviguer entre les pages internes de l'application, car elle forcerait un rechargement complet de la page.

On utilise le composant **`<Link>`** de `next/link` :
- Navigation instantanée côté client (Single Page Application).
- **Prefetching automatique** : Next.js précharge en tâche de fond les pages liées dès que le lien devient visible à l'écran.

```jsx
// app/page.jsx
import Link from 'next/link';

export default function HomePage() {
  return (
    <div>
      <h1>Accueil</h1>
      <p>Consultez notre équipe :</p>
      <Link href="/about">Voir la page À propos</Link>
    </div>
  );
}
```

---

## 5. Server Components vs Client Components

C'est la notion la plus importante à comprendre pour bien débuter.

### 5.1 Par défaut : Les Server Components
Dans le dossier `app/`, **tous les composants sont des Server Components par défaut**.
- Ils s'exécutent uniquement sur le serveur.
- Leur code JavaScript n'est **jamais** envoyé au navigateur (gain de poids considérable).
- Ils peuvent être asynchrones (`async / await`) pour charger des données directement sans `useEffect` !

```jsx
// app/users/page.jsx (Server Component)
export default async function UsersPage() {
  // Appel API direct côté serveur : pas besoin de useState ni de useEffect !
  const res = await fetch('https://jsonplaceholder.typicode.com/users');
  const users = await res.json();

  return (
    <div>
      <h1>Liste des utilisateurs</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 5.2 Quand utiliser un Client Component ?
Si votre composant a besoin :
- D'interactivité utilisateur : `onClick`, `onChange`, `onSubmit`.
- De hooks React d'état ou d'effets : `useState`, `useEffect`.
- D'APIs du navigateur : `localStorage`, `window`, `document`.

Il faut déclarer la directive **`'use client'`** tout en haut du fichier :

```jsx
// app/counter/page.jsx
'use client'; // Indique à Next.js d'envoyer le composant au navigateur

import { useState } from 'react';

export default function CounterPage() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Compteur : {count}</p>
      <button onClick={() => setCount(count + 1)}>Incrémenter</button>
    </div>
  );
}
```

> **Règle d'or :** Gardez la majorité de vos composants en *Server Components* et n'ajoutez `'use client'` que sur les petits composants interactifs (bouton, formulaire, modal).

---

## 6. Optimisation des Images : `<Image />`

Au lieu de la balise HTML standard `<img />`, Next.js fournit le composant `<Image />` de `next/image` qui :
- Redimensionne l'image selon l'écran de l'utilisateur (mobile, tablette, desktop).
- Convertit automatiquement les formats lourds vers **WebP** ou **AVIF**.
- Empêche les décalages de mise en page (*Cumulative Layout Shift*).
- Charge les images de manière différée (*lazy loading*).

```jsx
import Image from 'next/image';

export default function Banner() {
  return (
    <div>
      <Image
        src="/logo.png"       // Situé dans le dossier 'public/'
        alt="Logo de mon site"
        width={200}           // Largeur requise
        height={50}           // Hauteur requise
      />
    </div>
  );
}
```

---

## 7. Résumé des Essentiels pour Démarrer

1. **Routage** : Un dossier = une route URL, accessible si elle contient un fichier `page.jsx`.
2. **Layouts** : `layout.jsx` enveloppe les pages avec un squelette commun permanent.
3. **Liens** : Naviguez toujours avec `<Link href="...">` de `next/link`.
4. **Composants Serveur par défaut** : Chargez vos données directement avec `async/await`.
5. **Composants Client** : Ajoutez `'use client'` uniquement pour l'interactivité (`useState`, `onClick`).
6. **Images** : Préférez toujours `<Image />` à `<img>`.