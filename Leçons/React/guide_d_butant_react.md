# Bien Débuter avec React (Niveau Novice)

React est une bibliothèque JavaScript open-source créée par Meta pour concevoir des interfaces utilisateur (UI) interactives et réactives.

Avant React, modifier une page web demandait de manipuler manuellement le DOM (`document.getElementById`, `element.innerHTML`, etc.). Avec React, on déclare simplement **ce à quoi l'interface doit ressembler**, et React se charge de mettre à jour la page de façon fluide.

---

## 1. Le Concept Fondamental : Les Composants

Dans React, une interface se découpe en petits morceaux réutilisables et indépendants appelés **composants**. 

Pensez à un jeu de briques LEGO : un bouton, une barre de navigation ou une carte de profil sont des briques que l'on assemble pour bâtir une application.

En React moderne, un composant est simplement une **fonction JavaScript qui renvoie du code visuel** :

```jsx
function Welcome() {
  return <h1>Bienvenue sur mon application !</h1>;
}

export default Welcome;
```

> **Règle d'or :** Le nom d'un composant React doit **toujours commencer par une lettre majuscule** (`Welcome`, `UserProfile`, `Button`). Cela permet à React de le distinguer d'une balise HTML standard comme `<button>` ou `<div>`.

---

## 2. Le JSX : Écrire du pseudo-HTML dans du JavaScript

Ce que renvoie un composant ressemble à du HTML, mais c'est du **JSX** (*JavaScript XML*). C'est une extension syntaxique qui permet de mêler structure visuelle et logique de code.

### 2.1 Les règles incontournables du JSX

1. **Un seul élément racine :** Un composant doit renvoyer un élément parent unique. Si vous avez plusieurs balises, enveloppez-les dans un fragment vide (`<> ... </>`) :

```jsx
// ❌ Erreur : deux éléments frères sans parent
function Profile() {
  return (
    <h1>Mon Profil</h1>
    <p>Développeur web débutant</p>
  );
}

// ✅ Correct : utilisation d'un Fragment (<> ... </>)
function Profile() {
  return (
    <>
      <h1>Mon Profil</h1>
      <p>Développeur web débutant</p>
    </>
  );
}
```

2. **Les attributs HTML réservés changent de nom :**
   - `class` devient `className` (car `class` est un mot réservé en JavaScript).
   - `for` devient `htmlFor`.

```jsx
<div className="card-container">...</div>
```

3. **Toutes les balises orphelines doivent être fermées :**
   - `<img src="..." />`
   - `<br />`
   - `<input type="text" />`

### 2.2 Insérer des variables JavaScript dans le JSX

Pour afficher une variable ou exécuter une expression JavaScript dans le JSX, on utilise des **accolades `{}`** :

```jsx
function UserCard() {
  const username = "Sarah";
  const unreadMessages = 4;

  return (
    <div className="card">
      <h2>Bonjour, {username} !</h2>
      <p>Vous avez {unreadMessages * 2} notifications.</p>
    </div>
  );
}
```

---

## 3. Transmettre des données : Les Props

Les **props** (abréviation de *propriétés*) permettent de passer des informations d'un composant parent vers un composant enfant, exactement comme des arguments passés à une fonction.

### 3.1 Définir et utiliser des Props

```jsx
// Composant Enfant : reçoit les props sous forme d'objet
function Greeting({ name, role = "Utilisateur" }) {
  return (
    <div>
      <h3>Bonjour {name} !</h3>
      <p>Statut : {role}</p>
    </div>
  );
}

// Composant Parent : transmet les valeurs
function App() {
  return (
    <div>
      <Greeting name="Alice" role="Admin" />
      <Greeting name="Bob" />
    </div>
  );
}
```

> **Important :** Les props sont en **lecture seule** (*read-only*). Un composant enfant ne doit jamais tenter de modifier directement les props qu'il a reçues.

---

## 4. Rendre l'interface dynamique : Le State (`useState`)

Les props viennent de l'extérieur. Mais que faire si un composant a besoin de mémoriser une information qui change dans le temps (un texte saisi, un compteur, un menu ouvert ou fermé) ?

On utilise le **State** (l'état local) via le hook **`useState`**.

### 4.1 Syntaxe de `useState`

```jsx
import { useState } from 'react';

function Counter() {
  // 1. Déclaration : [valeurCourante, fonctionPourModifier] = useState(valeurInitiale)
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1); // 2. Modifie l'état et déclenche un réaffichage automatique
  }

  return (
    <div>
      <p>Nombre de clics : {count}</p>
      <button onClick={handleClick}>Incrémenter</button>
    </div>
  );
}
```

Quand on appelle `setCount`, React prend la nouvelle valeur, réexécute le composant (*re-render*) et met à jour l'écran sans recharger la page.

---

## 5. Gestion des Événements (*Events*)

Dans React, les événements s'écrivent en **camelCase** (`onClick`, `onChange`, `onSubmit`) et reçoivent une fonction (et non une chaîne de caractères comme en HTML brut).

```jsx
function ActionButton() {
  function handleAlert() {
    alert("Bouton cliqué !");
  }

  return (
    // On passe la référence de la fonction, on ne l'appelle PAS ici :
    // ✅ onClick={handleAlert}
    // ❌ onClick={handleAlert()} -> Cela s'exécuterait immédiatement au chargement !
    <button onClick={handleAlert}>
      Cliquez-moi
    </button>
  );
}
```

Pour les champs de formulaire :

```jsx
function NameInput() {
  const [name, setName] = useState('');

  return (
    <div>
      <input 
        type="text" 
        value={name} 
        onChange={(event) => setName(event.target.value)} 
        placeholder="Tapez votre prénom"
      />
      <p>Aperçu en direct : {name}</p>
    </div>
  );
}
```

---

## 6. Affichage Conditionnel et Listes

### 6.1 Affichage Conditionnel

On utilise les opérateurs JavaScript classiques (`&&` ou l'opérateur ternaire `? :`) à l'intérieur du JSX :

```jsx
function Notification({ count }) {
  return (
    <div>
      <h2>Boîte de réception</h2>
      {count > 0 ? (
        <p>Vous avez {count} nouveaux messages.</p>
      ) : (
        <p>Aucun message non lu.</p>
      )}
    </div>
  );
}
```

### 6.2 Afficher une Liste avec `.map()` et la prop `key`

Pour transformer une liste de données en éléments JSX, on utilise la méthode `.map()`. Chaque élément doit obligatoirement avoir une prop `key` unique pour aider React à identifier les changements dans le DOM.

```jsx
function TaskList() {
  const tasks = [
    { id: 101, title: "Apprendre le JSX" },
    { id: 102, title: "Découvrir les Props" },
    { id: 103, title: "Comprendre le State" },
  ];

  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>{task.title}</li>
      ))}
    </ul>
  );
}
```

---

## Résumé pour débuter sereinement

1. **Un composant** est une fonction JS commençant par une majuscule qui retourne du JSX.
2. **Le JSX** demande un seul élément parent (ou un fragment `<>...</>`) et des accolades `{}` pour le JavaScript.
3. **Les Props** transmettent des données du parent vers l'enfant (lecture seule).
4. **Le State (`useState`)** stocke des données qui peuvent changer et réactualise l'interface automatiquement.
5. **Les listes** s'affichent avec `.map()` et exigent une clé unique (`key`).