# Quiz d'Évaluation — Next.js (Niveau Intermédiaire)

Ce quiz teste votre maîtrise des mécanismes intermédiaires de l'App Router : routes dynamiques, cycle de mise en cache et revalidation, Server Actions, Route Handlers, et gestion du streaming avec les fichiers conventionnels (`loading.tsx`, `error.tsx`).

---

### Question 1 : Routes dynamiques & `generateStaticParams`
Dans une route dynamique `app/articles/[slug]/page.tsx`, quel est le rôle principal de la fonction exportée `generateStaticParams()` ?

- [ ] A) Intercepter les requêtes HTTP `POST` envoyées sur l'article.
- [ ] B) Définir à l'avance la liste des paramètres possibles pour pré-générer les pages statiquement au moment du build (SSG).
- [ ] C) Empêcher le moteur de recherche d'indexer les pages dynamiques.
- [ ] D) Récupérer les paramètres d'URL côté client dans un Client Component.

---

### Question 2 : Stratégies de mise en cache avec `fetch`
Considérez l'appel suivant dans un Server Component :

```typescript
const res = await fetch('https://api.example.com/prices', {
  next: { revalidate: 3600 },
});
```

Quel comportement Next.js adopte-t-il pour cette requête ?

- [ ] A) La requête est rejouée systématiquement à chaque appel utilisateur (SSR pur).
- [ ] B) La réponse est mise en cache et considérée comme fraîche pendant 3600 secondes ; les requêtes suivantes réutilisent ce cache avant de planifier une régénération en tâche de fond (ISR).
- [ ] C) La page est supprimée du serveur au bout de 3600 millisecondes.
- [ ] D) La requête n'est exécutée que dans le navigateur client via un Service Worker.

---

### Question 3 : Mutations avec les Server Actions
Pour déclarer une fonction serveur exécutable directement depuis un formulaire côté client ou un bouton d'action, quelle directive obligatoire doit figurer dans la fonction ou en tête du fichier ?

- [ ] A) `'use server'`
- [ ] B) `'use action'`
- [ ] C) `'use mutation'`
- [ ] D) `'use api'`

---

### Question 4 : Invalidation de cache après mutation
Après avoir mis à jour une donnée dans une Server Action, quelle fonction native Next.js permet d'invalider immédiatement le cache de la page associée pour forcer l'affichage des données fraîches ?

- [ ] A) `router.reload()`
- [ ] B) `revalidatePath('/articles')`
- [ ] C) `cache.clear()`
- [ ] D) `destroySession('/articles')`

---

### Question 5 : Streaming et Suspense avec `loading.tsx`
Lorsqu'un fichier `loading.tsx` est placé dans un dossier de route, comment Next.js gère-t-il l'affichage pendant que le Server Component charge ses données asynchrones ?

- [ ] A) Il affiche une page blanche jusqu'à la fin de la résolution des promesses.
- [ ] B) Il enveloppe automatiquement le composant `page.tsx` dans un `<Suspense fallback={<Loading />}>` et streame l'interface dès que possible.
- [ ] C) Il redirige automatiquement l'utilisateur vers la page d'accueil.
- [ ] D) Il force l'exécution du code en synchrone sur le thread principal.

---

### Question 6 : Gestion d'erreurs avec `error.tsx`
Pourquoi le composant déclaré dans le fichier de convention `app/error.tsx` doit-il **obligatoirement** comporter la directive `'use client'` ?

- [ ] A) Pour pouvoir envoyer des emails d'alerte sans passer par le serveur.
- [ ] B) Parce qu'il agit comme une *Error Boundary* React, qui nécessite des méthodes de cycle de vie et permet à l'utilisateur d'interagir (par exemple via la fonction `reset()`).
- [ ] C) Parce que les Server Components ne peuvent pas contenir de balises HTML `<p>` ou `<button>`.
- [ ] D) C'est une simple convention cosmétique sans impact technique.

---

### Question 7 : Route Handlers (`route.ts`)
Dans un fichier `app/api/webhook/route.ts`, comment expose-t-on un point d'entrée pour intercepter les requêtes HTTP `POST` ?

- [ ] A) En exportant par défaut une fonction `export default function handler(req, res)`.
- [ ] B) En exportant une fonction nommée correspondant à la méthode HTTP : `export async function POST(request: Request)`.
- [ ] C) En déclarant `export const method = 'POST'`.
- [ ] D) En utilisant obligatoirement le décorateur `@Post()`.

---

## Corrigé et Explications Détaillées

<details>
<summary>👉 Cliquez ici pour voir les réponses détaillées</summary>

### 1. Réponse : **B**
**Explication :** `generateStaticParams()` remplace l'ancien `getStaticPaths` du Pages Router. Elle renvoie un tableau d'objets contenant les paramètres (ex: `[{ slug: 'article-1' }, { slug: 'article-2' }]`), permettant à Next.js de générer ces routes HTML/RSC statiquement au build.

### 2. Réponse : **B**
**Explication :** L'option `{ next: { revalidate: 3600 } }` implémente le modèle **ISR (Incremental Static Regeneration)** : la réponse reste en cache pendant une heure. Toute requête reçue après ce délai sert la version en cache tout en déclenchant un recalcul asynchrone en arrière-plan.

### 3. Réponse : **A**
**Explication :** La directive `'use server'` signale au compilateur Next.js qu'une fonction asynchrone doit être exposée comme une Server Action (accessible via un endpoint RPC interne généré automatiquement).

### 4. Réponse : **B**
**Explication :** `revalidatePath('/chemin')` ou `revalidateTag('nom-du-tag')` purgent le cache du Data Cache et du Full Route Cache de Next.js côté serveur, garantissant que le prochain rendu reflète la mutation effectuée.

### 5. Réponse : **B**
**Explication :** La convention `loading.tsx` exploite le streaming de React et Suspense. Le layout et le fallback de chargement sont envoyés immédiatement au navigateur, puis le contenu final de la page est inséré dès que ses données asynchrones sont prêtes.

### 6. Réponse : **B**
**Explication :** Les limites de capture d'erreur (*Error Boundaries*) en React reposent sur des mécanismes réservés aux composants clients. De plus, `error.tsx` reçoit la fonction `reset()` qui permet de réexécuter le rendu du composant côté client pour récupérer de l'erreur sans recharger tout l'onglet.

### 7. Réponse : **B**
**Explication :** L'App Router utilise les standards Web modernes (`Request`, `Response`). On définit les méthodes HTTP supportées en exportant des fonctions portant le nom du verbe HTTP en majuscules (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`).
</details>