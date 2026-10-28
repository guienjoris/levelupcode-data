# Masterclass JavaScript — Niveau Senior / Staff Engineer

Ce guide explore en profondeur les mécaniques d'exécution bas niveau du runtime V8, les optimisations du compilateur JIT, la mémoire partagée et la concurrence multi-threads, la métaprogrammation avancée et l'architecture de flux asynchrones avec contrôle de contre-pression (*backpressure*).

---

## 1. Moteur V8 : Pipeline JIT & Représentation Mémoire

### 1.1 Le pipeline V8 : De l'AST au Code Machine

Le moteur V8 n'exécute pas de code interprété naïf. Il fait transiter le code par deux compilateurs complémentaires :

```
Code Source
   │
   ▼
[ Parser ] ──► Abstract Syntax Tree (AST)
   │
   ▼
[ Ignition ] ──► Bytecode + Feedback Vector (profiling des types)
   │
   ├── (Fonctions "hot" avec types monomorphes stables)
   ▼
[ TurboFan ] ──► Code Machine natif optimisé (spéculations d'inlining)
   │
   └── (Désoptimisation / Bailout si violation de type) ──► Retour à Ignition
```

1. **Ignition (Interpréteur / Compilateur Bytecode)** : génère rapidement un bytecode compact tout en instrumentant l'exécution via des vecteurs de feedback (*Type Feedback Vectors*).
2. **TurboFan (Compilateur JIT d'optimisation)** : exploite les types observés par Ignition pour effectuer des optimisations spéculatives (inlining de fonctions, suppression des vérifications de bornes, vectorisation SIMD).
3. **Deoptimization (*Bailout*)** : si une hypothèse est violée (ex. un paramètre supposé entier reçoit soudainement un objet ou `undefined`), le code machine est invalidé à chaud (*on-stack replacement reverse*) et le contexte repasse à Ignition avec un coût CPU important.

---

### 1.2 Hidden Classes (Shapes) et Arbres de Transition

En C++, le décalage mémoire (*offset*) d'un champ dans une `struct` est résolu à la compilation. En JavaScript, les objets étant des dictionnaires dynamiques, V8 émule ce comportement via les **Hidden Classes** (nommées *Shapes* ou *Maps* internes).

Chaque ajout de propriété fait progresser l'objet le long d'un arbre de transitions :

```javascript
// ❌ ANTI-PATTERN : Divergence de Shapes (Transitions asymétriques)
function buildPointA() {
  const p = {};
  p.x = 10; // Shape initial C0 -> C1 (ajoute offset 'x')
  p.y = 20; // Shape C1 -> C2 (ajoute offset 'y')
  return p;
}

function buildPointB() {
  const p = {};
  p.y = 20; // Shape initial C0 -> C3 (ajoute offset 'y')
  p.x = 10; // Shape C3 -> C4 (ajoute offset 'x')
  return p;
}
```

Bien que structurellement identiques (`{ x, y }`), ces deux objets possèdent deux chaînes de Shapes distinctes ($C_2 \neq C_4$).

```javascript
//  BONNE PRATIQUE SENIOR : Initialisation unifiée
class Vector2D {
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y; // Même ordre, même typage, Shape identique garantie
  }
}
```

---

### 1.3 Inline Caches (IC) : Monomorphisme vs Polymorphisme

Les accès aux propriétés (`point.x`) ne parcourent pas une table de hachage si le code est optimisé. V8 génère un **Inline Cache** au point d'appel :

- **Monomorphe (1 seule Shape vue)** : l'accès est transformé en lecture directe à un offset mémoire fixe. Efficacité maximale.
- **Polymorphe (2 à 4 Shapes différentes)** : V8 génère une chaîne conditionnelle de vérifications de Shape.
- **Mégamorphe (5 Shapes et plus)** : le cache capitule et bascule vers une recherche générique en table de symboles (*slow path*).

```javascript
// ❌ Devient facilement mégamorphe si 'entity' prend des formes hétérogènes
function renderEntity(entity) {
  return entity.draw(); // IC polymorphe puis mégamorphe
}
```

---

## 2. Gestion Mémoire & Garbage Collection (Orinoco)

V8 implémente un ramasse-miettes générationnel basé sur l'**hypothèse de mortalité infantile** (*Weak Generational Hypothesis*) : la majorité des allocations meurent très rapidement.

```
                           HEAP MÉMOIRE V8
 ┌──────────────────────────────────────────────────────────────┐
 │                      Young Generation                        │
 │  ┌───────────────────────┐        ┌───────────────────────┐  │
 │  │      Semi-Space 1     │  ◄──►  │      Semi-Space 2     │  │
 │  │        (From)         │        │         (To)          │  │
 │  └───────────────────────┘        └───────────────────────┘  │
 └──────────────────────────────┬───────────────────────────────┘
                                │ (Survivant après 2 cycles)
                                ▼
 ┌──────────────────────────────────────────────────────────────┐
 │                      Old Generation                          │
 │  - Old Pointer Space (références)                            │
 │  - Old Data Space (payloads scalaires, chaînes, buffers)     │
 │  - Large Object Space (objets dépassant la taille de page)   │
 └──────────────────────────────────────────────────────────────┘
```

### 2.1 Les Algorithmes de V8

1. **Scavenger (Cheney's Copying Algorithm)** :
   - Nettoie la *Young Generation* (New Space).
   - Déplace les pointeurs vivants d'un demi-espace vers l'autre de façon séquentielle (évite toute fragmentation).
   - Très rapide (proportionnel au nombre d'objets vivants, pas à la taille du tas).

2. **Major GC (Mark-Sweep-Compact)** :
   - Nettoie la *Old Generation*.
   - **Marking concurrent** : les threads d'arrière-plan parcourent le graphe d'accessibilité sans bloquer l'Event Loop.
   - **Sweeping & Compacting** : réorganise la mémoire pour supprimer la fragmentation.

---

### 2.2 Détection et Prévention des Fuites Mémoire

```javascript
// Exemple : Gestion explicite du cycle de vie avec FinalizationRegistry et WeakRef
const registry = new FinalizationRegistry((resourceId) => {
  console.info(`[GC Audit] Ressource externe libérée : ${resourceId}`);
});

class NativeWrapper {
  #resourceHandle;

  constructor(id, handle) {
    this.id = id;
    this.#resourceHandle = handle;
    // Permet d'observer la destruction sans empêcher le GC de faire son travail
    registry.register(this, id, this);
  }

  destroy() {
    this.#resourceHandle?.close();
    this.#resourceHandle = null;
    registry.unregister(this);
  }
}
```

---

## 3. Concurrence Réelle : Mémoire Partagée et `Atomics`

JavaScript n'est pas limité au modèle à mémoire isolée de `postMessage`. Depuis l'arrivée de `SharedArrayBuffer`, plusieurs threads (Web Workers ou `worker_threads` dans Node.js) peuvent manipuler le même bloc mémoire physique simultanément.

### 3.1 Synchronisation sans mutex avec l'API `Atomics`

```javascript
// worker-thread.js (Node.js ou Web Worker)
const { parentPort, workerData } = require('worker_threads');

// Récupération de la vue partagée (Int32Array sur SharedArrayBuffer)
const sharedArray = new Int32Array(workerData.sharedBuffer);

// Incrément thread-safe atomique (garantit l'absence de race condition sur l'ALU)
const previousValue = Atomics.add(sharedArray, 0, 1);

// Réveille un thread en sommeil sur cet index mémoire
Atomics.notify(sharedArray, 0, 1);
```

```javascript
// main-thread.js
const { Worker } = require('worker_threads');

const sharedBuffer = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * 4);
const sharedArray = new Int32Array(sharedBuffer);

sharedArray[0] = 0;

const worker = new Worker('./worker-thread.js', {
  workerData: { sharedBuffer }
});

// Attente synchrone bloquante d'un changement de valeur (uniquement hors thread UI)
// Atomics.wait(typedArray, index, expectedCurrentValue, timeoutMs)
const waitStatus = Atomics.wait(sharedArray, 0, 0, 5000);

if (waitStatus === 'ok') {
  console.log('Valeur incrémentée avec succès :', Atomics.load(sharedArray, 0));
}
```

---

## 4. Métaprogrammation : Invariants de Proxy et Symbol Internals

### 4.1 Proxies et Invariants du Moteur

Un `Proxy` permet d'intercepter les 13 opérations fondamentales de la spécification ECMAScript. Cependant, le moteur impose des **invariants stricts** : si une propriété de la cible (*target*) est non-configurable et non-inscriptible, la trappe `get` du proxy **doit impérativement** renvoyer la valeur exacte du miroir, sous peine de déclencher un `TypeError` fatal.

```javascript
// Création d'une membrane immuable avec préservation des invariants
function createImmutableMembrane(target) {
  return new Proxy(target, {
    set(obj, prop) {
      throw new TypeError(`Violation d'invariance : la propriété "${String(prop)}" est en lecture seule.`);
    },
    deleteProperty(obj, prop) {
      throw new TypeError(`Violation d'invariance : interdiction de supprimer "${String(prop)}".`);
    },
    get(obj, prop, receiver) {
      const descriptor = Object.getOwnPropertyDescriptor(obj, prop);
      // Respect absolu des invariants V8 pour les propriétés non configurables
      if (descriptor && !descriptor.configurable && !descriptor.writable) {
        return obj[prop];
      }

      const value = Reflect.get(obj, prop, receiver);
      if (typeof value === 'object' && value !== null) {
        return createImmutableMembrane(value);
      }
      return value;
    }
  });
}
```

---

### 4.2 Les Well-Known Symbols pour contrôler les primitives

```javascript
class ObservableStream {
  #buffer;

  constructor(initialData = []) {
    this.#buffer = [...initialData];
  }

  // 1. Itération native asynchrone (for await... of)
  async *[Symbol.asyncIterator]() {
    while (this.#buffer.length > 0) {
      yield Promise.resolve(this.#buffer.shift());
    }
  }

  // 2. Contrôle de l'opérateur instanceof sans chaîne prototypale
  static [Symbol.hasInstance](instance) {
    return instance && typeof instance.pipe === 'function';
  }

  // 3. Contrôle des conversions primitives implicites
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return this.#buffer.length;
    if (hint === 'string') return `[Stream: ${this.#buffer.length} items]`;
    return this.#buffer.length > 0;
  }
}
```

---

## 5. Architecture d'une Queue Asynchrone avec Concurrence & Backpressure

Dans les systèmes de traitement à haut débit, lancer un nombre non borné de promesses avec `Promise.all()` provoque un épuisement de mémoire vive (*OOM*) et sature l'Event Loop.

Voici l'implémentation d'une file d'attente concurrente avec limitation stricte et support d'annulation via `AbortSignal` :

```javascript
class ConcurrentPriorityQueue {
  #concurrency;
  #running = 0;
  #queue = [];

  constructor({ concurrency = 4 } = {}) {
    if (concurrency < 1) throw new Error('Concurrency must be >= 1');
    this.#concurrency = concurrency;
  }

  /**
   * Enregistre une tâche avec niveau de priorité et signal d'annulation
   */
  push(taskFactory, { priority = 0, signal } = {}) {
    return new Promise((resolve, reject) => {
      if (signal?.aborted) {
        return reject(new DOMException('Operation aborted', 'AbortError'));
      }

      const item = {
        taskFactory,
        priority,
        resolve,
        reject,
        signal,
      };

      // Insertion par tri à la volée (priorité décroissante)
      const insertIndex = this.#queue.findIndex((entry) => entry.priority < priority);
      if (insertIndex === -1) {
        this.#queue.push(item);
      } else {
        this.#queue.splice(insertIndex, 0, item);
      }

      // Attachement du listener d'annulation si présent
      if (signal) {
        const onAbort = () => {
          const idx = this.#queue.indexOf(item);
          if (idx !== -1) {
            this.#queue.splice(idx, 1);
            reject(new DOMException('Operation aborted', 'AbortError'));
          }
        };
        signal.addEventListener('abort', onAbort, { once: true });
      }

      this.#drain();
    });
  }

  get pending() {
    return this.#queue.length;
  }

  get active() {
    return this.#running;
  }

  #drain() {
    while (this.#running < this.#concurrency && this.#queue.length > 0) {
      const item = this.#queue.shift();

      if (item.signal?.aborted) {
        item.reject(new DOMException('Operation aborted', 'AbortError'));
        continue;
      }

      this.#running += 1;

      (async () => {
        try {
          const result = await item.taskFactory({ signal: item.signal });
          item.resolve(result);
        } catch (err) {
          item.reject(err);
        } finally {
          this.#running -= 1;
          this.#drain();
        }
      })();
    }
  }
}
```

---

## 6. Synthèse des Règles d'Or pour Ingénieurs Senior

1. **Stabilité des Formes d'Objets** : instanciez toujours les propriétés dans le même ordre avec les mêmes types pour conserver les *Inline Caches* au statut monomorphe.
2. **Architecture Mémoire Neutre** : privilégiez `WeakMap` et `WeakRef` pour les métadonnées et mécanismes de cache transverses afin d'éviter les rétentions de contexte.
3. **Parallélisme Préemptif** : utilisez `Worker` combiné à `SharedArrayBuffer` et `Atomics` pour les calculs intensifs sur le processeur (décodage, cryptographie, parsing binaire), réservant le thread principal pour les I/O non bloquantes.
4. **Contrôle de Flux** : ne laissez jamais un pool de promesses grandir de façon non régulée ; dimensionnez la concurrence selon les capacités réseau/I/O disponibles.