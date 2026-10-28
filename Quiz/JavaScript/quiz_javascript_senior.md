# Quiz d'Évaluation — JavaScript Niveau Senior & Staff Engineer

Ce quiz évalue votre compréhension fine des composants internes de V8, des optimisations JIT, de la gestion mémoire, du multithreading avec mémoire partagée, et de la métaprogrammation.

---

### Question 1 : Hidden Classes (Shapes) & Inline Caches

Considérez ces deux fonctions d'instanciation :

```javascript
function createPointA() {
  const p = {};
  p.x = 1;
  p.y = 2;
  return p;
}

function createPointB() {
  const p = {};
  p.y = 2;
  p.x = 1;
  return p;
}
```

Si ces deux fonctions sont appelées des milliers de fois et que leurs résultats sont passés indifféremment à une fonction critique `compute(point) { return point.x + point.y; }`, quel est l'impact sur le compilateur TurboFan ?

- [ ] A) Aucun impact : V8 trie alphabétiquement les clés des objets pour fusionner leurs représentations internes.
- [ ] B) L'accès aux propriétés devient polymorphe ou mégamorphe, car les deux objets ne partagent pas le même arbre de transition de Shapes.
- [ ] C) TurboFan convertit automatiquement les objets en tableaux denses pour conserver l'accès monomorphe.
- [ ] D) Le moteur déclenche une erreur d'optimisation fatale bloquant l'exécution de l'Event Loop.

---

### Question 2 : Désoptimisation JIT (Bailout)

Quelle situation provoque typiquement une désoptimisation (*bailout*) brutale d'une fonction compilée en code machine natif par TurboFan ?

- [ ] A) L'utilisation répétée de `const` au lieu de `let`.
- [ ] B) L'appel de la fonction avec un argument d'un type différent de celui observé lors du profiling Ignition (ex: passer une `string` là où seuls des `Smi` / entiers avaient été vus).
- [ ] C) L'assignation d'une fonction anonyme fléchée à une variable de portée bloc.
- [ ] D) La présence d'une boucle `while` exécutée plus de 10 000 fois.

---

### Question 3 : Ramasse-miettes V8 (Orinoco) & Scavenger

Dans l'architecture du Garbage Collector de V8, quelle est la caractéristique principale du ramasse-miettes mineur (*Scavenger*) opérant sur le *New Space* ?

- [ ] A) Il utilise l'algorithme de Cheney basé sur deux semi-espaces (*From-Space* et *To-Space*) pour copier uniquement les objets vivants et compacter la mémoire rapidement.
- [ ] B) Il parcoure l'ensemble du *Old Space* en marquant chaque nœud de façon synchrone.
- [ ] C) Il s'exécute uniquement lorsque l'application utilise plus de 1.4 Go de RAM.
- [ ] D) Il détruit immédiatement tous les objets référencés par des `WeakRef`.

---

### Question 4 : Mémoire partagée & Atomics

Pourquoi l'appel `Atomics.wait(int32Array, index, value)` est-il strictement interdit sur le thread principal (*main thread*) dans la plupart des navigateurs web ?

- [ ] A) Parce que le thread principal n'a pas accès aux instances de `SharedArrayBuffer` pour des raisons de sécurité CORS.
- [ ] B) Parce que `Atomics.wait` bloque le thread de manière synchrone, ce qui gèlerait la boucle d'événements, les interactions utilisateur et le rendu graphique.
- [ ] C) Parce que `Atomics.wait` ne peut manipuler que des nombres à virgule flottante 64 bits (`Float64Array`).
- [ ] D) Parce qu'il nécessite impérativement une connexion HTTPS avec WebSockets actifs.

---

### Question 5 : Métaprogrammation & Invariants de Proxy

Soit un objet cible rendu non-extensible ou possédant une propriété non-configurable :

```javascript
const target = {};
Object.defineProperty(target, 'id', {
  value: 42,
  writable: false,
  configurable: false,
});

const proxy = new Proxy(target, {
  get(obj, prop, receiver) {
    if (prop === 'id') {
      return 999; // Tentative d'altération
    }
    return Reflect.get(obj, prop, receiver);
  },
});
```

Que se passe-t-il lors de l'évaluation de `proxy.id` ?

- [ ] A) `999` est retourné sans avertissement.
- [ ] B) `undefined` est retourné silencieusement.
- [ ] C) Un `TypeError` est levé car le Proxy viole un invariant fondamental du langage (la valeur retournée doit être identique à la valeur réelle d'une propriété non-configurable et non-inscriptible).
- [ ] D) Le moteur V8 supprime automatiquement le Proxy et retourne `42`.

---

### Question 6 : Well-Known Symbols & Coercition

Quel symbole bien connu (*well-known symbol*) permet de surcharger la conversion primitive d'un objet et reçoit en argument le hint (`'number'`, `'string'`, ou `'default'`) ?

- [ ] A) `Symbol.iterator`
- [ ] B) `Symbol.toPrimitive`
- [ ] C) `Symbol.toStringTag`
- [ ] D) `Symbol.species`

---

### Question 7 : Concurrence et Contre-pression (Backpressure)

Dans une architecture backend Node.js traitant des millions d'appels asynchrones par minute, quel risque majeur pose l'utilisation systématique de `await Promise.all(items.map(processItem))` sans file de contrôle de concurrence ?

- [ ] A) La pile d'appels synchrone (*Call Stack*) lève immédiatement un `RangeError: Maximum call stack size exceeded`.
- [ ] B) Toutes les promesses démarrent simultanément, ce qui sature la mémoire vive (allocations massives de contextes de promesses) et peut épuiser le pool de sockets/connexions SQL jusqu'au crash OOM (*Out Of Memory*).
- [ ] C) L'Event Loop s'arrête définitivement car les microtâches ne peuvent pas être parallélisées par le système d'exploitation.
- [ ] D) `Promise.all` ignore silencieusement toutes les requêtes après les 1024 premières.

---

## Corrigé et Explications Détaillées

<details>
<summary>👉 Cliquez pour afficher les réponses et explications</summary>

### 1. Réponse : **B**
**Explication :** En JavaScript, l'ordre d'insertion des propriétés détermine la transition des *Hidden Classes* (*Shapes*). `createPointA` produit la transition `{} -> {x} -> {x, y}`, tandis que `createPointB` produit `{} -> {y} -> {y, x}`. Même s'ils ont les mêmes clés, ces objets ont deux Shapes internes différentes avec des décalages mémoires (*offsets*) inversés. La fonction `compute` ne peut donc pas stabiliser un *Inline Cache* monomorphe et bascule en polymorphisme (ou mégamorphisme si d'autres formes apparaissent), réduisant l'efficacité des optimisations JIT de TurboFan.

### 2. Réponse : **B**
**Explication :** TurboFan se base sur le profiling spéculatif fourni par Ignition via les vecteurs de feedback (*Type Feedback Vectors*). Si une fonction compilée pour manipuler des entiers 31 bits optimisés (`Smi`) reçoit une chaîne ou un objet complexe, l'hypothèse spéculative est invalidée. Le moteur effectue alors un *deopt / bailout* pour revenir au bytecode Ignition non optimisé, afin de recalculer le résultat correctement mais avec un coût de bascule élevé.

### 3. Réponse : **A**
**Explication :** Le *Scavenger* gère le *New Space* (où sont alloués les nouveaux objets). Il divise cette zone en deux semi-espaces (*From* et *To*). Lors d'un ramassage mineur, il copie uniquement les objets encore accessibles dans l'autre semi-espace, compactant immédiatement la mémoire et éliminant les objets morts sans coût de balayage individuel. Les objets survivant à plusieurs cycles sont ensuite promus dans le *Old Space*.

### 4. Réponse : **B**
**Explication :** `Atomics.wait()` met en pause le thread appelant jusqu'à ce qu'une notification (`Atomics.notify()`) survienne ou que le timeout expire. Si cette instruction était autorisée sur le thread principal d'un navigateur, la boucle d'événements serait totalement gelée : l'interface ne répondrait plus à aucun clic, aucun rafraîchissement d'affichage (60/120 Hz) ne pourrait être calculé et la page semblerait complètement bloquée.

### 5. Réponse : **C**
**Explication :** La spécification ECMAScript impose des *invariants* stricts aux Proxies pour garantir l'intégrité du moteur : si la propriété sous-jacente de la cible est définie avec `configurable: false` et `writable: false`, le piège `get` du Proxy est obligé de renvoyer la valeur exacte de la cible. Renvoyer une valeur différente (ici `999` au lieu de `42`) entraîne immédiatement une exception `TypeError: 'get' on proxy: property 'id' is a read-only and non-configurable data property on the proxy target but the proxy did not return the same value`.

### 6. Réponse : **B**
**Explication :** `Symbol.toPrimitive` est la méthode appelée en interne par le moteur lorsqu'un objet subit une coercition de type (par exemple via l'opérateur unaire `+obj`, une concaténation `"" + obj` ou une comparaison abstraite). Elle reçoit le type suggéré (`'number'`, `'string'` ou `'default'`) et permet de définir un comportement de conversion sur-mesure.

### 7. Réponse : **B**
**Explication :** `items.map(fn)` déclenche l'exécution de toutes les fonctions asynchrones de manière immédiate et concurrente, sans attendre `Promise.all`. Avec un très grand volume d'éléments, cela crée instantanément des milliers ou millions de Promesses et de descripteurs de sockets en mémoire, menant à l'épuisement du pool de connexions (DB/HTTP), à des timeouts en cascade et fréquemment à un crash du processus Node.js par dépassement de mémoire (*heap out of memory*). C'est pourquoi un pattern de file d'attente bornée (*Bounded Queue*) ou de limitation de concurrence (*p-limit*) est obligatoire en production.
</details>