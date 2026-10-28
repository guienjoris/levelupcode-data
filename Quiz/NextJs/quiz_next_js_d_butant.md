# Quiz d'Évaluation — Next.js (Niveau Novice)

Ce quiz permet de tester vos connaissances sur les bases de Next.js avec l'App Router : convention de fichiers, layouts, navigation optimisée, distinction entre Server Components et Client Components, et optimisation d'images.

---

### Question 1 : Convention de nommage de l'App Router

Dans Next.js (App Router), quel est le nom de fichier obligatoire à placer dans un dossier pour que ce dernier devienne une route URL accessible publiquement (par exemple `/contact`) ?

- [ ] A) `index.jsx`
- [ ] B) `route.jsx`
- [ ] C) `page.jsx`
- [ ] D) `view.jsx`

---

### Question 2 : Rôle du fichier `layout.jsx`

Quelle est la caractéristique principale d'un composant défini dans `layout.jsx` par rapport à un composant `page.jsx` ?

- [ ] A) Il se recharge intégralement à chaque changement de page.
- [ ] B) Il conserve son état et son DOM sans être recréé lors des transitions entre pages enfants d'un même segment.
- [ ] C) Il est obligatoirement un Client Component avec `'use client'`.
- [ ] D) Il ne peut pas contenir de balises HTML natives.

---

### Question 3 : Navigation interne optimisée

Pourquoi privilégie-t-on le composant `<Link href="...">` de `next/link` plutôt qu'une balise `<a href="...">` classique pour les liens internes ?

- [ ] A) `<Link>` est la seule façon d'appliquer des styles CSS.
- [ ] B) `<Link>` effectue une transition côté client sans recharger la page et précharge automatiquement les ressources de la page cible dès que le lien devient visible.
- [ ] C) `<Link>` force l'ouverture du lien dans un nouvel onglet.
- [ ] D) La balise `<a>` est désactivée par défaut par le compilateur Next.js.

---

### Question 4 : Comportement par défaut des composants

Dans le dossier `app/` de Next.js, quel est le statut par défaut d'un composant si aucune directive spécifique n'est écrite en tête de fichier ?

- [ ] A) Il est compilé en tant que Client Component.
- [ ] B) Il est un Server Component exécuté uniquement sur le serveur.
- [ ] C) Il est converti automatiquement en page statique HTML non modifiable.
- [ ] D) Il déclenche une erreur si on ne précise pas son type.

---

### Question 5 : La directive `'use client'`

Dans quelle situation l'ajout de la directive `'use client'` tout en haut du fichier d'un composant est-il **indispensable** ?

- [ ] A) Dès que le composant doit effectuer une requête avec `fetch`.
- [ ] B) Dès que le composant utilise du JSX avec des balises `<div>` ou `<span>`.
- [ ] C) Dès que le composant utilise un hook comme `useState` ou attache un écouteur d'événement interactif comme `onClick`.
- [ ] D) Dès que le composant affiche une image avec `<Image />`.

---

### Question 6 : Chargement de données dans les Server Components

Comment charge-t-on habituellement des données distantes depuis une API dans un **Server Component** Next.js ?

- [ ] A) En déclarant la fonction du composant comme `async` et en appelant directement `await fetch(...)` dans son corps.
- [ ] B) En utilisant impérativement un hook `useEffect` combiné avec un `useState`.
- [ ] C) En créant obligatoirement un service Worker externe.
- [ ] D) En passant par `getInitialProps` ou `getServerSideProps` dans l'App Router.

---

### Question 7 : Optimisation d'images avec `next/image`

Quel avantage direct offre le composant `<Image />` par rapport à la balise HTML native `<img />` ?

- [ ] A) Il supprime automatiquement l'arrière-plan des images.
- [ ] B) Il compresse et redimensionne l'image au format moderne (WebP/AVIF), ajuste la résolution à l'écran et réserve l'espace pour éviter les sauts de mise en page (CLS).
- [ ] C) Il stocke automatiquement l'image dans une base de données SQLite locale.
- [ ] D) Il télécharge l'image uniquement en monochrome pour économiser la bande passante.

---

## Corrigé et Explications Détaillées

<details>
<summary>👉 Cliquez ici pour voir les réponses détaillées</summary>

### 1. Réponse : **C**
**Explication :** Dans l'App Router, la convention est stricte : le dossier définit le chemin de l'URL (ex: `app/contact/`) et le fichier `page.jsx` (ou `.tsx`) rend cette route publiquement accessible. Un dossier sans fichier `page` ne constitue pas une page visitable.

### 2. Réponse : **B**
**Explication :** Les layouts permettent de préserver la structure commune (barre de navigation, pied de page) lors de la navigation : leur état interne n'est pas détruit et leur DOM n'est pas réinstancié, ce qui garantit une navigation fluide et performante.

### 3. Réponse : **B**
**Explication :** Le composant `<Link>` intercepte le clic pour gérer une navigation douce côté client (*client-side routing*) évitant un rafraîchissement complet du navigateur. De plus, il intègre le *prefetching*, préchargeant les données de la route cible en tâche de fond.

### 4. Réponse : **B**
**Explication :** Dans l'App Router, tous les composants sont des **React Server Components (RSC)** par défaut. Leur code source n'est pas expédié au navigateur, ce qui réduit considérablement le poids du bundle JavaScript initial.

### 5. Réponse : **C**
**Explication :** Le serveur ne dispose pas d'environnement d'exécution de navigateur (pas de curseur souris, pas de DOM interactif). Dès qu'un composant nécessite des états réactifs (`useState`, `useReducer`), des cycles d'effets (`useEffect`) ou des écouteurs de clics/saisies (`onClick`, `onChange`), il doit être marqué comme Client Component via `'use client'`.

### 6. Réponse : **A**
**Explication :** Puisque les Server Components tournent sur le serveur, ils peuvent être définis comme des fonctions asynchrones natives (`export default async function Page() { ... }`) et exécuter directement des requêtes réseau ou des accès base de données avec `await fetch(...)`, éliminant le besoin de `useEffect`.

### 7. Réponse : **B**
**Explication :** Le composant `next/image` optimise automatiquement le poids et le format des images à la volée, sert la taille adaptée au viewport du visiteur et exige de renseigner les dimensions (`width`/`height` ou `fill`) pour réserver le ratio géométrique dans la page, évitant ainsi le *Cumulative Layout Shift* (CLS).
</details>