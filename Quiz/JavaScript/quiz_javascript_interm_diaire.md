# Quiz d'Évaluation — JavaScript Niveau Intermédiaire

Ce quiz évalue votre compréhension approfondie des mécanismes internes du moteur JavaScript : portée lexicale, closures, résolution de `this`, héritage prototypal, ordonnancement asynchrone dans l'Event Loop et structures de données avancées.

---

### Question 1 : Portée lexicale et Closures

Considérez l'exécution du code suivant :

```javascript
function makeCounters() {
  const funcs = [];
  for (var i = 0; i < 3; i++) {
    funcs.push(function () {
      return i;
    });
  }
  return funcs;
}

const list = makeCounters();
console.log(list[0](), list[1](), list[2]());
```

Quel affichage obtient-on en console ?

- [ ] A) `0 1 2`
- [ ] B) `3 3 3`
- [ ] C) `undefined undefined undefined`
- [ ] D) `0 0 0`

---

### Question 2 : Résolution de contexte (`this`) et fonctions fléchées

Considérez l'objet suivant :

```javascript
const service = {
  label: 'Service A',
  getLabel: () => {
    return this.label;
  },
};

const client = {
  label: 'Client B',
  fetchLabel: service.getLabel,
};

console.log(client.fetchLabel());
```

Dans un environnement de navigateur standard (mode non-strict), quelle est la valeur affichée ?

- [ ] A) `'Client B'`
- [ ] B) `'Service A'`
- [ ] C) `undefined`
- [ ] D) Une erreur `TypeError: Cannot read properties of undefined` est levée.

---

### Question 3 : Délégation prototypale

Soit la structure suivante :

```javascript
function Item(title) {
  this.title = title;
}

Item.prototype.display = function () {
  return `Item: ${this.title}`;
};

const book = new Item('Livre');
book.display = function () {
  return `Ouvrage: ${this.title}`;
};
```

Que renvoie l'appel `Object.getPrototypeOf(book).display.call(book)` ?

- [ ] A) `'Ouvrage: Livre'`
- [ ] B) `'Item: Livre'`
- [ ] C) `undefined`
- [ ] D) Une erreur `TypeError: display is not a function` est levée.

---

### Question 4 : Hiérarchie de l'Event Loop

Dans quel ordre exact les messages suivants seront-ils affichés dans la console ?

```javascript
console.log('A');

setTimeout(() => {
  console.log('B');
}, 0);

Promise.resolve().then(() => {
  console.log('C');
}).then(() => {
  console.log('D');
});

queueMicrotask(() => {
  console.log('E');
});

console.log('F');
```

- [ ] A) `A, F, B, C, D, E`
- [ ] B) `A, F, C, E, D, B`
- [ ] C) `A, F, C, D, E, B`
- [ ] D) `A, B, C, D, E, F`

---

### Question 5 : Combinateurs de Promesses

Vous devez interroger 3 serveurs miroirs redondants (`mirrorA`, `mirrorB`, `mirrorC`). Vous souhaitez obtenir la réponse du **premier serveur qui réussit** sa requête. Si un serveur renvoie une erreur 500, vous devez l'ignorer et attendre les autres. Si tous échouent, vous devez capturer une erreur globale.

Quel combinateur d'API `Promise` convient exactement à ce besoin ?

- [ ] A) `Promise.race()`
- [ ] B) `Promise.all()`
- [ ] C) `Promise.any()`
- [ ] D) `Promise.allSettled()`

---

### Question 6 : Garbage Collection & `WeakMap`

Quelle est la principale différence architecturale entre une `Map` et une `WeakMap` ?

- [ ] A) `WeakMap` permet d'utiliser des primitives (comme des strings) comme clés sans conversion.
- [ ] B) Les entrées d'une `WeakMap` sont automatiquement supprimées au bout d'un timeout défini.
- [ ] C) Les clés d'une `WeakMap` doivent être des objets et sont référencées faiblement, permettant au ramasse-miettes de les libérer si aucune autre référence n'existe.
- [ ] D) `WeakMap` est synchronisée automatiquement sur plusieurs threads de Web Workers.

---

### Question 7 : Exécution explicite de contexte (`call`, `apply`, `bind`)

Soit la fonction :

```javascript
function greet(prefix, suffix) {
  return `${prefix} ${this.name}${suffix}`;
}

const person = { name: 'Camille' };
```

Quelle instruction lève une exception d'exécution ?

- [ ] A) `greet.call(person, 'Bonjour', ' !')`
- [ ] B) `greet.apply(person, ['Bonjour', ' !'])`
- [ ] C) `greet.bind(person)('Bonjour', ' !')`
- [ ] D) `greet.apply(person, 'Bonjour', ' !')`

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** La variable `var i` ne possède pas de portée de bloc mais une portée de fonction englobante. Toutes les fonctions anonymes captent par closure la **même** référence vers la variable unique `i`. Lorsque la boucle se termine, `i` vaut `3`. Chaque invocation `list[n]()` lit donc la valeur finale `3`. Avec `let i = 0`, chaque tour de boucle aurait créé une nouvelle liaison lexicale et le résultat aurait été `0 1 2`.

### 2. Réponse : **C**
**Explication :** Les fonctions fléchées ne créent pas leur propre contexte `this` : elles adoptent le `this` lexical du bloc où elles sont déclarées. Ici, au niveau de la racine du script, `this` pointe vers l'objet global (`window`). `window.label` n'ayant pas été défini, l'évaluation renvoie `undefined`. L'assignation de la méthode à `client.fetchLabel` n'a aucun impact car le `this` d'une fonction fléchée ne peut jamais être modifié.

### 3. Réponse : **B**
**Explication :** L'assignation `book.display = ...` crée une méthode directement sur l'instance (effet de *shadowing* ou masquage). Cependant, le prototype d'origine reste intact sur `Object.getPrototypeOf(book)`. En exécutant `display.call(book)` directement depuis le prototype, on invoque l'implémentation originelle de `Item.prototype` tout en lui passant l'instance `book` comme contexte `this`.

### 4. Réponse : **B**
**Explication :** 
1. Le code synchrone s'exécute en premier : `'A'` puis `'F'`.
2. Le callback du `setTimeout` est envoyé dans la file des macrotâches (*Task Queue*).
3. Les microtâches en attente sont traitées immédiatement : le premier `.then()` s'exécute (`'C'`) et `queueMicrotask` est traité (`'E'`).
4. La résolution du premier `.then()` a planifié la microtâche suivante (`'D'`). La file des microtâches doit être intégralement vidée avant de passer aux macrotâches.
5. Une fois toutes les microtâches purgées, l'Event Loop passe à la macrotâche : `'B'`.

### 5. Réponse : **C**
**Explication :** `Promise.any()` résout dès que l'une des promesses est tenue (*fulfilled*), ignorant les éventuels rejets tant qu'il reste des promesses en cours. Si toutes échouent, elle rejette un `AggregateError`. Au contraire, `Promise.race()` aurait rejeté immédiatement si le premier serveur à répondre avait renvoyé une erreur.

### 6. Réponse : **C**
**Explication :** Les clés d'une `WeakMap` doivent impérativement être des objets non primitifs. Le lien de la clé est « faible » (*weak reference*), ce qui signifie que la présence de l'objet dans la `WeakMap` n'empêche pas le Garbage Collector d'évacuer cet objet de la mémoire si aucune autre variable n'y fait référence.

### 7. Réponse : **D**
**Explication :** `Function.prototype.apply()` attend strictement un tableau (ou un objet similaire à un tableau comme `arguments`) comme second paramètre. Lui passer des chaînes de caractères successives provoque un `TypeError: CreateListFromArrayLike called on non-object`. Pour passer des arguments délimités par des virgules, il faut utiliser `call()`.