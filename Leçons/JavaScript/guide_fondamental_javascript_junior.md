# Les Fondamentaux Modernes de JavaScript (Niveau Junior)

Bienvenue dans ce guide conçu pour consolider les bases essentielles du développement JavaScript moderne (ES6+). Ce cours traite des mécaniques fondamentales que tout développeur junior doit maîtriser pour écrire du code propre, prévisible et maintenable.

---

## 1. Déclaration des variables : `const`, `let` et portée (*Scope*)

Historiquement, JavaScript utilisait `var`, dont la portée est liée à la fonction (*function scope*), ce qui créait de nombreux effets de bord imprévus. En JavaScript moderne, on utilise quasi exclusivement `const` et `let`, qui ont une portée de bloc (*block scope*).

| Mot-clé | Réassignable ? | Redéclarable ? | Portée (*Scope*) | Recommandation |
| :--- | :--- | :--- | :--- | :--- |
| `const` | Non | Non | Bloc (`{ ... }`) | **Par défaut** pour toute variable |
| `let` | **Oui** | Non | Bloc (`{ ... }`) | Seulement si la valeur doit changer (boucle, compteur) |
| `var` | **Oui** | **Oui** | Fonction | **À proscrire** en code moderne |

### Exemple concret

```javascript
const maxRetries = 3;
// maxRetries = 4; // TypeError: Assignment to constant variable.

let attemptCount = 0;
attemptCount += 1; // Valide

// Portée de bloc
if (true) {
  const secret = 'token123';
  let counter = 1;
}
// console.log(secret); // ReferenceError: secret is not defined
```

> **Attention à la mutabilité :** Déclarer un objet ou un tableau avec `const` empêche de réassigner la variable, mais ne rend pas l'objet immuable.
>
> ```javascript
> const user = { name: 'Alice' };
> user.name = 'Bob'; // Tout à fait valide !
> // user = { name: 'Charlie' }; // TypeError
> ```

---

## 2. Types de données et vérification d'égalité

JavaScript distingue les **primitifs** (passés par valeur) et les **objets / références** (passés par référence).

### 2.1 Les types primitifs
1. `string`
2. `number` (y compris `NaN` et `Infinity`)
3. `boolean`
4. `null` (absence intentionnelle de valeur)
5. `undefined` (variable déclarée sans valeur assignée)
6. `symbol`
7. `bigint`

### 2.2 Égalité stricte (`===`) vs Égalité lâche (`==`)
- L'opérateur `==` effectue une **coercition implicite de type** avant de comparer, menant à des incohérences (`0 == ''` vaut `true`).
- L'opérateur `===` compare la valeur **et** le type sans conversion implicite.

```javascript
console.log(0 == false);   // true  (dangereux)
console.log(0 === false);  // false (recommandé)

console.log(null == undefined);  // true
console.log(null === undefined); // false
```

---

## 3. Manipulation des Tableaux : Méthodes fonctionnelles

L'une des compétences clés d'un développeur junior est d'éviter les boucles `for` impératives au profit des méthodes déclaratives d'ordre supérieur.

### 3.1 `map` : Transformer chaque élément
Crée un **nouveau tableau** de même taille contenant le résultat de la transformation.

```javascript
const pricesInEuros = [10, 25, 50];
const pricesInCents = pricesInEuros.map((price) => price * 100);
// [1000, 2500, 5000]
```

### 3.2 `filter` : Sélectionner des éléments
Retourne un **nouveau tableau** contenant uniquement les éléments validant la condition.

```javascript
const users = [
  { id: 1, name: 'Alice', isActive: true },
  { id: 2, name: 'Bob', isActive: false },
  { id: 3, name: 'Charlie', isActive: true },
];

const activeUsers = users.filter((user) => user.isActive);
// Contient Alice et Charlie
```

### 3.3 `find` : Trouver un unique élément
Renvoie le **premier** élément correspondant au prédicat, ou `undefined` s'il n'existe pas.

```javascript
const bob = users.find((user) => user.name === 'Bob');
// { id: 2, name: 'Bob', isActive: false }
```

### 3.4 `reduce` : Agréger vers une valeur unique
Accumule les valeurs d'un tableau pour produire un résultat unique (nombre, objet, tableau groupé).

```javascript
const cart = [
  { item: 'Clavier', price: 80 },
  { item: 'Souris', price: 40 },
  { item: 'Tapis', price: 15 },
];

const totalPrice = cart.reduce((total, product) => total + product.price, 0);
// 135
```

---

## 4. Syntaxe ES6+ indispensable au quotidien

### 4.1 La déstructuration (*Destructuring*)
Permet d'extraire des propriétés d'un objet ou des éléments d'un tableau directement dans des variables.

```javascript
const config = {
  host: 'localhost',
  port: 5432,
  database: 'app_db',
};

// Extraction directe avec valeur par défaut possible
const { host, port, ssl = false } = config;

// Déstructuration de tableaux
const coordinates = [48.8566, 2.3522];
const [latitude, longitude] = coordinates;
```

### 4.2 L'opérateur de propagation (*Spread Operator* : `...`)
Permet de cloner ou fusionner des objets et tableaux de manière immuable.

```javascript
// Tableaux
const baseTags = ['frontend', 'js'];
const allTags = [...baseTags, 'react', 'css'];

// Objets
const initialUser = { name: 'Sarah', role: 'guest' };
const updatedUser = {
  ...initialUser,
  role: 'admin', // Écrase la clé 'role' précédente
  lastLogin: new Date(),
};
```

### 4.3 Chaînage optionnel (`?.`) et Nullish Coalescing (`??`)

- **Optional Chaining (`?.`)** : Évite les erreurs fatales `Cannot read properties of undefined`.
- **Nullish Coalescing (`??`)** : Assigne une valeur de secours uniquement si l'expression vaut `null` ou `undefined` (contrairement à `||` qui réagit aussi à `0`, `""` ou `false`).

```javascript
const response = {
  data: {
    user: {
      profile: null,
    },
  },
};

// Sécurise l'accès en profondeur
const avatar = response.data?.user?.profile?.avatarUrl; // undefined (sans crash)

const userAge = 0;
const displayedAge = userAge ?? 18; // 0 (car 0 est une valeur valide)
const fallbackAge = userAge || 18;  // 18 (attention : 0 est falsy !)
```

---

## 5. L'Asynchronisme : Des Promesses à `async/await`

JavaScript s'exécute sur un seul thread d'exécution principal (*Event Loop*). Les opérations lentes (requêtes réseau, lecture disque) sont donc asynchrones pour ne pas bloquer l'interface ou le serveur.

### 5.1 Anatomie d'une Promesse
Une Promesse représente une valeur qui sera disponible dans le futur, selon trois états :
1. **Pending** (en attente)
2. **Fulfilled** (résolue avec succès)
3. **Rejected** (rejetée avec une erreur)

### 5.2 Utilisation moderne avec `async` / `await`
`async/await` est une surcouche syntaxique (*syntactic sugar*) au-dessus des Promesses, rendant le code lisible de façon séquentielle.

```javascript
async function fetchUserProfile(userId) {
  try {
    const response = await fetch(`https://api.example.com/users/${userId}`);

    // fetch ne rejette pas automatiquement sur les codes HTTP 4xx ou 5xx
    if (!response.ok) {
      throw new Error(`Erreur HTTP: ${response.status}`);
    }

    const userData = await response.json();
    return userData;
  } catch (error) {
    console.error('Impossible de charger le profil :', error.message);
    throw error; // Relancer ou gérer l'erreur selon le besoin
  }
}
```

---

## 6. Bonnes pratiques de base pour débuter

1. **Prioriser la clarté :** Nommez vos fonctions avec un verbe d'action (`calculateTotal`, `getUserById`, `isValidEmail`).
2. **Éviter les effets de bord :** Privilégiez les fonctions pures (qui ne modifient pas directement leurs arguments extérieurs).
3. **Toujours typer ou valider les entrées :** Vérifiez la présence et le format des données avant d'accéder à leurs propriétés.
4. **Toujours intercepter les erreurs asynchrones :** N'appelez jamais un `async/await` sans un bloc `try / catch` approprié.