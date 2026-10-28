# Masterclass JavaScript — Niveau Intermédiaire

Pour dépasser le niveau junior en JavaScript, il ne suffit plus de connaître la syntaxe moderne ou d'appeler des méthodes d'API. Il est indispensable de comprendre les mécanismes internes du moteur d'exécution (V8/SpiderMonkey) : la gestion de la mémoire, les portées lexicales, la résolution de contexte et l'ordonnancement asynchrone.

---

## Sommaire

1. [Closures et Portée Lexicale (*Lexical Scope*)](#1-closures-et-portée-lexicale-lexical-scope)
2. [Le mot-clé `this` et la liaison de contexte (*Binding*)](#2-le-mot-clé-this-et-la-liaison-de-contexte-binding)
3. [La chaîne des prototypes et l'héritage prototypal](#3-la-chaîne-des-prototypes-et-lhéritage-prototypal)
4. [L'Event Loop : Macrotâches vs Microtâches](#4-levent-loop--macrotâches-vs-microtâches)
5. [Programmation Asynchrone Avancée : Concurrence avec `Promise`](#5-programmation-asynchrone-avancée--concurrence-avec-promise)
6. [Structures de données avancées : `Map`, `Set`, `WeakMap`, `WeakSet`](#6-structures-de-données-avancées--map-set-weakmap-weakset)

---

## 1. Closures et Portée Lexicale (*Lexical Scope*)

### 1.1 Qu'est-ce qu'une closure ?

Une **closure** (ou fermeture) est la combinaison d'une fonction et de l'environnement lexical dans lequel cette fonction a été déclarée. En JavaScript, une fonction interne conserve l'accès aux variables de sa fonction parente, même après que la fonction parente a terminé son exécution.

```javascript
function createCounter(initialValue = 0) {
  let count = initialValue; // Variable encapsulée, inaccessible depuis l'extérieur

  return {
    increment() {
      count += 1;
      return count;
    },
    decrement() {
      count -= 1;
      return count;
    },
    getValue() {
      return count;
    },
  };
}

const counter = createCounter(10);
console.log(counter.increment()); // 11
console.log(counter.getValue());  // 11
console.log(counter.count);       // undefined (véritable encapsulation privée)
```

### 1.2 Cas d'usage : Memoization & Currying

La closure permet d'implémenter des patrons de conception comme la mise en cache de fonctions pures (memoization) :

```javascript
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const expensiveCalculation = (n) => {
  // Simulation de calcul lourd
  return n * 2;
};

const memoizedCalc = memoize(expensiveCalculation);
memoizedCalc(42); // Calcule et met en cache
memoizedCalc(42); // Retourne directement depuis le cache
```

---

## 2. Le mot-clé `this` et la liaison de contexte (*Binding*)

La valeur de `this` n'est généralement pas déterminée lors de la définition d'une fonction, mais **au moment de son invocation** (à l'exception des fonctions fléchées).

Il existe 4 règles de résolution de `this`, classées par ordre de priorité croissante :

### 2.1 Les 4 règles

1. **Liaison par défaut (*Default Binding*) :** En mode non-strict, `this` pointe vers l'objet global (`window` ou `global`). En mode strict (`'use strict'`), `this` vaut `undefined`.
2. **Liaison implicite (*Implicit Binding*) :** Lorsque la fonction est invoquée en tant que méthode d'un objet (`obj.method()`), `this` fait référence à l'objet hôte `obj`.
3. **Liaison explicite (*Explicit Binding*) :** Définie via `call`, `apply` ou `bind`.
4. **Liaison `new` (*New Binding*) :** Lorsque la fonction est invoquée avec le mot-clé `new`, `this` référence la nouvelle instance tout juste allouée.

```javascript
function identify(greeting) {
  return `${greeting}, je suis ${this.name}`;
}

const developer = { name: 'Alex' };

// call : arguments passés individuellement
console.log(identify.call(developer, 'Salut')); // "Salut, je suis Alex"

// apply : arguments passés sous forme de tableau
console.log(identify.apply(developer, ['Bonjour'])); // "Bonjour, je suis Alex"

// bind : renvoie une NOUVELLE fonction avec le contexte figé
const boundIdentify = identify.bind(developer);
console.log(boundIdentify('Hello')); // "Hello, je suis Alex"
```

### 2.2 Les fonctions fléchées (*Arrow Functions*)

Les fonctions fléchées n'ont **pas** de liaison propre de `this`. Elles adoptent le `this` lexical de la portée englobante au moment de leur écriture. `call`, `apply` et `bind` n'ont aucun effet modificateur sur leur `this`.

---

## 3. La chaîne des prototypes et l'héritage prototypal

Le mot-clé `class` introduit en ES6 n'est qu'un sucre syntaxique (*syntactic sugar*). Sous le capot, JavaScript utilise un système d'héritage par délégation prototypale.

### 3.1 Mécanisme interne : `[[Prototype]]` et `prototype`

- Tout objet possède une propriété interne cachée `[[Prototype]]` (accessible via `Object.getPrototypeOf(obj)` ou historiquement `__proto__`).
- Lorsqu'une propriété est recherchée sur un objet, le moteur inspecte l'objet lui-même. S'il ne la trouve pas, il remonte la chaîne via `[[Prototype]]` jusqu'à atteindre `Object.prototype`, puis `null`.

```javascript
function Animal(name) {
  this.name = name;
}

// Méthode partagée par toutes les instances via le prototype (économie mémoire)
Animal.prototype.speak = function () {
  return `${this.name} fait du bruit.`;
};

function Dog(name, breed) {
  Animal.call(this, name); // Héritage des propriétés d'instance
  this.breed = breed;
}

// Liaison de la chaîne prototypale
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function () {
  return `${this.name} aboie !`;
};

const rex = new Dog('Rex', 'Berger Allemand');
console.log(rex.bark());  // "Rex aboie !" (trouvé sur Dog.prototype)
console.log(rex.speak()); // "Rex fait du bruit." (trouvé sur Animal.prototype)
console.log(rex instanceof Animal); // true
```

---

## 4. L'Event Loop : Macrotâches vs Microtâches

JavaScript est un langage mono-thread non-bloquant grâce à l'**Event Loop** (boucle d'événements). L'exécution ordonnée repose sur une hiérarchie stricte de files d'attente :

```
[ Call Stack (Pile d'exécution synchrone) ]
                   │
                   ▼
       Pile d'exécution vide ?
         ├── OUI ──► Vider TOUTES les Microtâches (Microtask Queue)
         │             (Promises, queueMicrotask, MutationObserver)
         │                   │
         │                   ▼
         │           Traiter UNE seule Macrotâche (Task Queue)
         │             (setTimeout, setInterval, I/O, setImmediate)
         │                   │
         │                   ▼
         │           Éventuel rendu graphique du navigateur
         └── NON ──► Continuer l'exécution de la pile
```

### 4.1 Exemple d'ordre d'exécution critique

```javascript
console.log('1. Synchrone');

setTimeout(() => {
  console.log('2. Timeout (Macrotâche)');
}, 0);

Promise.resolve().then(() => {
  console.log('3. Promise 1 (Microtâche)');
}).then(() => {
  console.log('4. Promise 2 (Microtâche enchaînée)');
});

queueMicrotask(() => {
  console.log('5. Microtask directe');
});

console.log('6. Fin synchrone');
```

**Sortie exacte en console :**
```text
1. Synchrone
6. Fin synchrone
3. Promise 1 (Microtâche)
5. Microtask directe
4. Promise 2 (Microtâche enchaînée)
2. Timeout (Macrotâche)
```

> **Règle d'or :** La file des microtâches doit être complètement purgée avant que l'Event Loop n'exécute la prochaine macrotâche.

---

## 5. Programmation Asynchrone Avancée : Concurrence avec `Promise`

L'utilisation naïve de `await` dans une boucle crée des cascades séquentielles inutiles. JavaScript fournit des combinateurs de promesses pour gérer la concurrence :

| Méthode | Comportement | Cas d'usage typique |
| :--- | :--- | :--- |
| `Promise.all()` | Résout quand **toutes** réussissent ; rejette dès la **première** erreur. | Dépendances strictes (ex: charger les données A ET B indispensables). |
| `Promise.allSettled()` | Attend que **toutes** soient terminées (succès ou échec) sans court-circuit. | Tâches indépendantes (ex: logs, analytics, nettoyage). |
| `Promise.race()` | Résout ou rejette dès que la **première** promesse termine. | Implémentation d'un timeout réseau. |
| `Promise.any()` | Résout dès la **première promesse tenue** ; rejette (`AggregateError`) si **toutes** échouent. | Requêtes vers des miroirs redondants (le premier qui répond gagne). |

### Exemple : Créer un wrapper de Timeout avec `Promise.race`

```javascript
function withTimeout(promise, timeoutMs) {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => {
      reject(new Error(`Opération expirée après ${timeoutMs}ms`));
    }, timeoutMs);
  });

  return Promise.race([promise, timeoutPromise]);
}

// Utilisation
try {
  const data = await withTimeout(fetch('https://api.example.com/data'), 3000);
} catch (error) {
  console.error(error.message);
}
```

---

## 6. Structures de données avancées : `Map`, `Set`, `WeakMap`, `WeakSet`

Les objets standards (`{}`) ne sont pas toujours adaptés aux collections de clés dynamiques :

### 6.1 `Map` vs `Object`
- `Map` accepte **n'importe quel type de clé** (fonctions, objets, nombres, primitives).
- `Map` garantit l'ordre d'insertion des éléments lors de l'itération.
- `Map.prototype.size` donne la taille en complexité $\mathcal{O}(1)$.

### 6.2 `WeakMap` et gestion de la mémoire (Garbage Collector)
Une `WeakMap` est une table dont :
- Les clés sont **obligatoirement des objets**.
- Les références vers ces clés sont **faibles** (*weak references*) : si l'objet clé n'a plus d'autre référence en mémoire, il peut être nettoyé par le ramasse-miettes (*Garbage Collector*), libérant automatiquement l'entrée de la `WeakMap`.
- Une `WeakMap` n'est pas itérable (pas de méthode `.keys()`, `.entries()`, ni de propriété `.size`).

```javascript
// Cas d'usage : Stocker des métadonnées privées attachées à un élément DOM
const elementMetadata = new WeakMap();

let button = document.createElement('button');
elementMetadata.set(button, { clickCount: 0 });

// Quand le bouton est supprimé du DOM et que la variable button est vidée :
button = null; 
// L'entrée dans elementMetadata sera automatiquement collectée en mémoire sans risque de fuite.
```

---

## Synthèse pour le développeur intermédiaire

1. **Maîtriser les closures** pour encapsuler l'état et éviter la pollution de la portée globale.
2. **Prédire la valeur de `this`** en examinant le site d'appel de la fonction et non sa déclaration.
3. **Comprendre la délégation prototypale** pour appréhender ce que les classes ES6 font sous le capot.
4. **Visualiser l'Event Loop** pour ne plus être surpris par l'ordre d'exécution de code asynchrone complexe.
5. **Choisir la bonne structure de données** (`Map`/`WeakMap`) pour optimiser les performances et la mémoire.