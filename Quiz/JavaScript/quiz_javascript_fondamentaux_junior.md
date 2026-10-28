# Quiz d'Évaluation — Les Fondamentaux Modernes de JavaScript (Niveau Junior)

Ce quiz permet de valider votre compréhension des notions abordées dans la leçon : la portée des variables, les types primitifs, les méthodes de tableaux, la syntaxe ES6+ et l'asynchronisme.

---

### Question 1 : Déclaration et mutabilité
Soit le code suivant :

```javascript
const user = { name: 'Alice' };
user.name = 'Bob';
user = { name: 'Charlie' };
```

Que se passe-t-il lors de l'exécution ?

- [ ] A) Une erreur est levée à la ligne 2 (`user.name = 'Bob'`).
- [ ] B) Une erreur est levée à la ligne 3 (`user = { name: 'Charlie' }`).
- [ ] C) Le code s'exécute sans erreur et `user.name` vaut `'Charlie'`.
- [ ] D) Une erreur de syntaxe se produit dès la ligne 1.

---

### Question 2 : Égalité stricte et coercition
Quelle expression évalue à `false` en JavaScript ?

- [ ] A) `0 == ''`
- [ ] B) `null == undefined`
- [ ] C) `0 === false`
- [ ] D) `false == ''`

---

### Question 3 : Méthodes de tableaux
Vous avez une liste de produits :

```javascript
const products = [
  { name: 'Écran', price: 200 },
  { name: 'Clavier', price: 50 },
  { name: 'Souris', price: 25 },
];
```

Quelle méthode devez-vous utiliser si vous souhaitez obtenir un tableau contenant uniquement les noms des produits (`['Écran', 'Clavier', 'Souris']`) ?

- [ ] A) `products.filter((p) => p.name)`
- [ ] B) `products.find((p) => p.name)`
- [ ] C) `products.map((p) => p.name)`
- [ ] D) `products.reduce((p) => p.name)`

---

### Question 4 : Nullish Coalescing (`??`) vs OR logique (`||`)
Quel sera l'affichage de ce code dans la console ?

```javascript
const score = 0;
const displayScore = score ?? 10;
const fallbackScore = score || 10;

console.log(displayScore, fallbackScore);
```

- [ ] A) `0 0`
- [ ] B) `10 10`
- [ ] C) `0 10`
- [ ] D) `10 0`

---

### Question 5 : Déstructuration et valeurs par défaut
Considérez le fragment de code suivant :

```javascript
const settings = { theme: 'dark' };
const { theme, fontSize = 14, language } = settings;

console.log(theme, fontSize, language);
```

Quelles sont les valeurs respectives affichées ?

- [ ] A) `'dark' 14 undefined`
- [ ] B) `'dark' undefined undefined`
- [ ] C) `'dark' 14 null`
- [ ] D) Une erreur `ReferenceError: language is not defined` est levée.

---

### Question 6 : Opérateur de propagation (*Spread Operator*)
Quelle est la valeur de `result` après cette opération ?

```javascript
const defaultOptions = { retries: 3, timeout: 1000 };
const userOptions = { timeout: 5000 };

const result = { ...defaultOptions, ...userOptions };
```

- [ ] A) `{ retries: 3, timeout: 1000 }`
- [ ] B) `{ retries: 3, timeout: 5000 }`
- [ ] C) `{ retries: 3, timeout: [1000, 5000] }`
- [ ] D) Une erreur est levée pour duplication de propriété `timeout`.

---

### Question 7 : Asynchronisme & `fetch`
Considérez l'appel suivant :

```javascript
try {
  const response = await fetch('https://api.example.com/not-found-404');
  console.log('Succès');
} catch (error) {
  console.log('Erreur interceptée');
}
```

Si le serveur distant répond avec un code HTTP **404 Not Found**, que se produit-il ?

- [ ] A) La promesse est automatiquement rejetée et le bloc `catch` affiche `'Erreur interceptée'`.
- [ ] B) La promesse est résolue avec succès, `response.ok` vaut `false`, et la console affiche `'Succès'`.
- [ ] C) L'application plante avec une exception non gérée.
- [ ] D) Le navigateur relance automatiquement la requête 3 fois.

---

## Corrigé et Explications

<details>
<summary>👉 Cliquez ici pour afficher le corrigé détaillé</summary>

### 1. Réponse : **B**
**Explication :** En JavaScript, `const` protège l'assignation de la variable, pas son contenu. Modifier une propriété (`user.name = 'Bob'`) est parfaitement valide. En revanche, tenter de réassigner une nouvelle référence (`user = { ... }`) déclenche un `TypeError: Assignment to constant variable`.

### 2. Réponse : **C**
**Explication :** L'opérateur strict `===` compare les types sans conversion. `0` est un `number` et `false` est un `boolean`, donc `0 === false` vaut `false`. Toutes les autres options utilisent `==` qui applique une coercition implicite menant à `true`.

### 3. Réponse : **C**
**Explication :** `map()` transforme chaque élément d'un tableau et retourne un nouveau tableau de même dimension. `filter()` ne sert qu'à trier/filtrer, et `find()` ne renvoie qu'un seul élément scalaire.

### 4. Réponse : **C**
**Explication :** 
- L'opérateur `??` ne remplace la valeur que si celle-ci est strictement `null` ou `undefined`. Comme `score` vaut `0`, `displayScore` conserve `0`.
- L'opérateur `||` remplace la valeur si elle est *falsy* (`0`, `""`, `false`, `null`, `undefined`, `NaN`). `0` étant *falsy*, `fallbackScore` bascule sur `10`.

### 5. Réponse : **A**
**Explication :** `theme` prend la valeur de l'objet (`'dark'`). `fontSize` n'existe pas dans l'objet source, elle prend donc sa valeur par défaut (`14`). `language` n'existe pas et n'a pas de valeur par défaut, elle vaut donc `undefined`.

### 6. Réponse : **B**
**Explication :** Lors de la fusion d'objets avec le spread operator (`...`), les propriétés définies plus à droite écrasent les propriétés de même nom situées à gauche. `userOptions.timeout` (5000) remplace donc `defaultOptions.timeout` (1000).

### 7. Réponse : **B**
**Explication :** C'est un piège classique : l'API `fetch()` ne rejette une promesse qu'en cas d'échec réseau strict (coupure internet, DNS invalide, etc.). Les codes HTTP d'erreur comme 404 ou 500 résolvent quand même la promesse. Il incombe au développeur de vérifier la propriété `response.ok`.
</details>