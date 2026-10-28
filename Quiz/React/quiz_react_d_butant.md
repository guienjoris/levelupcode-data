# Quiz d'Évaluation — React (Niveau Novice)

Ce quiz permet de tester vos connaissances sur les bases de React : composants, syntaxe JSX, transmission de données via les props, gestion de l'état avec `useState`, gestion des événements et rendu de listes.

---

### Question 1 : Convention de nommage d'un composant
Pourquoi le nom d'un composant React personnalisé doit-il obligatoirement commencer par une lettre majuscule (ex: `UserProfile` et non `userProfile`) ?

- [ ] A) C'est une obligation imposée par le moteur JavaScript pour toutes les fonctions.
- [ ] B) Cela permet à React et au compilateur JSX de faire la distinction entre une balise HTML native (`<div>`, `<button>`) et un composant personnalisé.
- [ ] C) Cela permet au composant de s'exécuter dans un thread séparé.
- [ ] D) Sans majuscule, le CSS ne s'applique pas sur le composant.

---

### Question 2 : Syntaxe JSX & Attributs
Quelle est la syntaxe correcte en JSX pour assigner une classe CSS à un élément HTML ?

- [ ] A) `<div class="container">`
- [ ] B) `<div className="container">`
- [ ] C) `<div class_name="container">`
- [ ] D) `<div styleClass="container">`

---

### Question 3 : Structure du JSX
Considérez le fragment de code suivant :

```jsx
function Header() {
  return (
    <h1>Mon Blog</h1>
    <p>Articles récents</p>
  );
}
```

Pourquoi ce code provoque-t-il une erreur à la compilation, et comment la corriger ?

- [ ] A) Les balises doivent impérativement être en majuscules ; il faut écrire `<H1>`.
- [ ] B) Il est interdit d'avoir un texte libre dans `<p>`.
- [ ] C) Un composant React doit retourner un élément racine unique ; il faut envelopper les deux balises dans un fragment `<> ... </>` ou une `<div>`.
- [ ] D) Le mot-clé `return` ne peut pas être entouré de parenthèses.

---

### Question 4 : Fonctionnement des Props
Quelle affirmation est **vraie** concernant les props dans React ?

- [ ] A) Un composant enfant peut réassigner ses props librement (ex: `props.title = "Nouveau"`).
- [ ] B) Les props sont en lecture seule (*read-only*) pour le composant qui les reçoit.
- [ ] C) Les props ne peuvent contenir que des chaînes de caractères.
- [ ] D) Les props sont automatiquement sauvegardées dans le `localStorage` du navigateur.

---

### Question 5 : Gestion de l'état avec `useState`
Soit le composant suivant :

```jsx
import { useState } from 'react';

function LikeButton() {
  const [likes, setLikes] = useState(0);

  function handleLike() {
    likes = likes + 1;
  }

  return <button onClick={handleLike}>J'aime : {likes}</button>;
}
```

Que se passe-t-il lorsque l'utilisateur clique sur le bouton ?

- [ ] A) Le compteur s'incrémente correctement à l'écran.
- [ ] B) Une erreur survient (assignation sur une `const`) et l'interface n'est pas mise à jour ; il faut appeler `setLikes(likes + 1)`.
- [ ] C) Le navigateur recharge la page immédiatement.
- [ ] D) React supprime le bouton du DOM.

---

### Question 6 : Attachement d'événements
Quelle est la manière recommandée de passer un gestionnaire de clic `handleClick` à un bouton en React ?

- [ ] A) `<button onClick={handleClick()}>Cliquez</button>`
- [ ] B) `<button onclick="handleClick()">Cliquez</button>`
- [ ] C) `<button onClick={handleClick}>Cliquez</button>`
- [ ] D) `<button click={handleClick}>Cliquez</button>`

---

### Question 7 : Rendu de listes et prop `key`
Pourquoi doit-on ajouter une prop `key` unique sur chaque élément rendu dans une boucle `.map()` ?

- [ ] A) Pour permettre à CSS d'ordonner les éléments alphabétiquement.
- [ ] B) Pour aider React à identifier précisément quels éléments ont été ajoutés, modifiés ou retirés, afin d'optimiser le rafraîchissement du DOM.
- [ ] C) La prop `key` est visible dans le DOM HTML pour les moteurs de recherche.
- [ ] D) Sans la prop `key`, le tableau JavaScript est effacé de la mémoire.

---

## Corrigé et Explications Détaillées

<details>
<summary>👉 Cliquez ici pour afficher les réponses et explications</summary>

### 1. Réponse : **B**
**Explication :** En JSX, les balises commençant par une minuscule (ex: `div`, `span`, `p`) sont interprétées comme des balises HTML standards. Les balises commençant par une majuscule (ex: `UserProfile`, `Button`) signalent au compilateur qu'il s'agit d'un composant React personnalisé à instancier.

### 2. Réponse : **B**
**Explication :** Le mot `class` étant un mot-clé réservé en JavaScript pour la déclaration de classes orientées objet, React utilise l'attribut `className` en JSX.

### 3. Réponse : **C**
**Explication :** Sous le capot, le JSX est transformé en appels de fonctions JavaScript (`React.createElement(...)` ou `_jsx(...)`). Une fonction JavaScript ne pouvant renvoyer qu'une seule valeur à la fois, le JSX doit obligatoirement avoir un parent unique. Les fragments (`<> ... </>`) résolvent ce problème sans ajouter de nœud inutile dans le DOM.

### 4. Réponse : **B**
**Explication :** Les props sont immuables du point de vue du composant récepteur. Toute modification d'affichage doit provenir d'un changement d'état (*State*) ou de props passées à nouveau par le parent.

### 5. Réponse : **B**
**Explication :** `likes` est déclarée avec `const`, donc la réassignation directe `likes = likes + 1` déclenche une erreur JavaScript. De plus, React ne surveille pas les variables locales : pour forcer un nouveau rendu (*re-render*) et rafraîchir l'interface, il faut obligatoirement passer par la fonction d'état `setLikes(likes + 1)`.

### 6. Réponse : **C**
**Explication :** En React, on passe la référence de la fonction : `onClick={handleClick}`. Écrire `onClick={handleClick()}` exécute la fonction immédiatement lors du rendu du composant, au lieu d'attendre le clic de l'utilisateur.

### 7. Réponse : **B**
**Explication :** L'algorithme de réconciliation de React utilise la prop `key` pour suivre l'identité de chaque élément entre deux rendus. Cela évite de recréer l'ensemble de la liste dans le DOM lorsque seul un élément change d'ordre ou de valeur.
</details>