# JavaScript Novice — Pratique Moderne : Tableaux, Déstructuration & Opérateurs ES6+

Cette leçon aborde les idiomes modernes du JavaScript d'aujourd'hui. Loin des simples boucles `for` ou des variables déclarées avec `var`, nous nous concentrons sur la syntaxe propre adoptée par tous les développeurs professionnels.

---

## 1. Les Gabarits de Chaînes (*Template Literals*)

Finie la concaténation fastidieuse avec l'opérateur `+`. Les *template literals* utilisent les accents graves (backticks \` \`) et permettent l'interpolation d'expressions avec `${...}` ainsi que le texte multiligne natif.

```javascript
const user = 'Sophie';
const role = 'Administratrice';

// Concaténation classique (ancienne)
const messageOld = 'Bonjour ' + user + ', vous êtes connectée en tant que : ' + role + '.';

// Gabarit moderne avec backticks
const messageModern = `Bonjour ${user}, vous êtes connectée en tant que : ${role}.`;

// Multiligne sans \n manuel
const templateHtml = `
  <div class="card">
    <h2>${user.toUpperCase()}</h2>
    <p>Statut : ${role}</p>
  </div>
`;
```

---

## 2. Déstructuration (*Destructuring Assignment*)

La déstructuration est une écriture concise pour extraire des valeurs d'un objet ou d'un tableau directement dans des variables distinctes.

### 2.1 Déstructuration d'objets

```javascript
const client = {
  id: 42,
  nom: 'Durand',
  ville: 'Nantes',
  profil: { actif: true }
};

// Extraction directe par correspondance de clé
const { nom, ville } = client;
console.log(nom); // 'Durand'

// Renommage lors de l'extraction + valeur par défaut
const { id: clientId, age = 18 } = client;
console.log(clientId); // 42
console.log(age);      // 18 (valeur de secours appliquée car absent de client)
```

### 2.2 Déstructuration de tableaux

Pour un tableau, l'extraction se fait selon **l'ordre des index** (et non par nom de propriété) :

```javascript
const coordonnees = [48.8566, 2.3522, 35]; // latitude, longitude, altitude

const [lat, lng] = coordonnees;
console.log(lat); // 48.8566
console.log(lng); // 2.3522

// Sauter un élément intermédiaire avec une virgule vide
const [latitude, , altitude] = coordonnees;
console.log(altitude); // 35
```

---

## 3. Les Opérateurs Rest et Spread (`...`)

Les trois points `...` changent de nom et de comportement selon l'endroit où ils sont placés :

### 3.1 Opérateur de décomposition (*Spread Operator*)
Il permet d'éclater un tableau ou un objet pour en cloner le contenu ou le fusionner sans modifier l'original (*immuabilité*).

```javascript
// Fusion et clonage de tableaux
const fruits = ['pomme', 'poire'];
const panier = ['fraise', ...fruits, 'banane'];
// panier vaut : ['fraise', 'pomme', 'poire', 'banane']

// Fusion et écrasement propre d'objets
const configParDefaut = { theme: 'dark', notifications: true };
const preferencesUser = { ...configParDefaut, theme: 'light' };
// preferencesUser vaut : { theme: 'light', notifications: true }
```

### 3.2 Opérateur de reste (*Rest Parameter*)
Dans la signature d'une fonction, il rassemble tous les arguments restants dans un véritable tableau JavaScript.

```javascript
function calculerTotal(tva, ...prixArticles) {
  // prixArticles est un véritable Array : [10, 25, 40]
  const sousTotal = prixArticles.reduce((acc, val) => acc + val, 0);
  return sousTotal * (1 + tva);
}

calculerTotal(0.2, 10, 25, 40);
```

---

## 4. Manipulation Fonctionnelle des Tableaux (`map`, `filter`, `reduce`)

On remplace les boucles `for` impératives par des méthodes d'itération pures qui ne modifient jamais le tableau d'origine.

```
[ Entrée ] ──► .filter(condition) ──► .map(transformation) ──► [ Résultat ]
```

### 4.1 `map()` : Transformer chaque élément
Retourne un **nouveau tableau** contenant exactement le même nombre d'éléments transformés.

```javascript
const prixHT = [10, 20, 50];
const prixTTC = prixHT.map(prix => prix * 1.2);
// Résultat : [12, 24, 60]
```

### 4.2 `filter()` : Filtrer selon un critère
Retourne un **nouveau tableau** contenant uniquement les éléments respectant un prédicat booléen (`true`).

```javascript
const utilisateurs = [
  { nom: 'Alice', actif: true },
  { nom: 'Bob', actif: false },
  { nom: 'Claire', actif: true }
];

const connectes = utilisateurs.filter(user => user.actif);
// Résultat : [{ nom: 'Alice', ... }, { nom: 'Claire', ... }]
```

### 4.3 `reduce()` : Agréger en une valeur unique
Condense l'ensemble des éléments dans un accumulateur (total, chaîne, dictionnaire).

```javascript
const notes = [14, 18, 12, 16];
const somme = notes.reduce((accumulateur, noteCourante) => accumulateur + noteCourante, 0);
const moyenne = somme / notes.length; // 15
```

---

## 5. Navigation Sécurisée : Chaînage Optionnel (`?.`) et Coalescence des Nuls (`??`)

Pour éviter l'erreur redoutée `TypeError: Cannot read properties of undefined` :

### 5.1 Chaînage optionnel (*Optional Chaining* `?.`)
Court-circuite et renvoie `undefined` si l'élément parent est `null` ou `undefined`, sans planter l'application.

```javascript
const commande = { id: 101 };

// Sans optional chaining (plante : commande.client n'existe pas)
// const codePostal = commande.client.adresse.codePostal; // 💥 TypeError !

// Avec optional chaining
const codePostal = commande.client?.adresse?.codePostal;
console.log(codePostal); // undefined (aucun crash)
```

### 5.2 Coalescence des nuls (*Nullish Coalescing* `??`)
Renvoie la valeur de secours **uniquement si** l'opérande de gauche vaut strictement `null` ou `undefined`.

> **Différence avec l'opérateur OU `||` :**  
> `||` considère `0`, `""` (chaîne vide) et `false` comme des valeurs fausses et applique à tort la valeur par défaut. `??` préserve ces valeurs légitimes !

```javascript
const scorePartie = 0;

const resultatOr = scorePartie || 10;     // 10  (Erreur : 0 est considéré faussé par ||)
const resultatNullish = scorePartie ?? 10; // 0   (Correct : 0 est une vraie valeur, pas null)
```

---
