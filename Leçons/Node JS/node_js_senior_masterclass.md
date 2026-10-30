# Masterclass Node.js — Niveau Senior : V8 Internals, Profiling, Concurrence & Architecture Haute Performance

Atteindre l'expertise Senior sur Node.js nécessite de lever le voile sur les abstractions pour comprendre comment le moteur JavaScript et le noyau d'I/O collaborent au niveau système. Cette leçon traite de l'optimisation des structures V8, du calibrage de la couche `libuv`, du profilage mémoire et CPU sous charge réelle, et de l'extension native via C/C++ (Node-API).

---

## 1. Moteur V8 : Compilation JIT et Mécanismes d'Optimisation

Le moteur V8 ne se contente pas d'interpréter du JavaScript ; il le compile dynamiquement en code machine natif via un pipeline à deux étages : **Ignition** (interpréteur produisant du bytecode) et **TurboFan** (compilateur optimisant basé sur des spéculations de types).

```
[ Code Source JS ] 
        │
        ▼
   [ Ignition ]  (Génère du Bytecode & collecte des profils de types de retour)
        │
        ├─── Profils stables (Monomorphisme) ────► [ TurboFan ] ──► [ Code Machine Optimisé ]
        │                                                                   │
        └─── Rupture de type (Polymorphisme excessif) ◄── [ Deopt ] ────────┘
```

### 1.1 Classes Cachées (*Hidden Classes* ou *Maps*)

JavaScript est un langage à typage dynamique sans layout mémoire fixe à la compilation. Pour égaler les performances du C++, V8 génère sous le capot des **classes cachées** (*Shapes* ou *Maps*) associées aux instances :

```javascript
// ❌ ANTI-PATTERN : Initialisation dynamique divergente
function Point(x, y) {
  this.x = x;
  this.y = y;
}

const p1 = new Point(1, 2);
const p2 = new Point(3, 4);

// Déstabilisation des classes cachées :
p1.z = 5; // Crée une transition vers une nouvelle Shape B
// p2 n'a pas z, ou z est ajouté plus tard dans un ordre différent :
p2.color = 'red'; // Crée une transition vers une Shape C
p2.z = 6;         // Crée une transition vers une Shape D (différente de B !)
```

```javascript
// ✅ OPTIMISATION SENIOR : Monomorphisme strict & Layout prévisible
class OptimizedPoint {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.z = 0; // Toujours initialiser toutes les propriétés dans le constructeur
  }
}
```

### 1.2 Caches en Ligne (*Inline Caches - IC*)

Lorsque V8 exécute une fonction accédant à une propriété (`obj.x`), il mémorise la classe cachée de l'objet et son offset mémoire direct dans un **Inline Cache** :
- **Monomorphique :** Le site d'appel ne rencontre qu'un seul type d'objet $\rightarrow$ Résolution directe par offset mémoire brut ($O(1)$ matériel, inlinable par TurboFan).
- **Polymorphique :** 2 à 4 formes différentes $\rightarrow$ Table de branchements conditionnels.
- **Mégamorphique (5 formes ou plus) :** Abandon de l'optimisation TurboFan, bascule en table de hashage globale lente.

---

## 2. Le Threadpool de libuv : Dimensionnement et Blocages Inattendus

Contrairement à l'idée reçue, Node.js n'est pas strictement mono-threadé. Les opérations qui ne peuvent pas être rendues asynchrones de façon non-bloquante au niveau de l'OS (notamment l'accès au système de fichiers standard sur Linux/POSIX et certaines fonctions cryptographiques) sont déportées sur le **Threadpool interne de libuv**.

```
                ┌────────────────────────────────────────────────────────┐
                │          Thread Principal (Event Loop V8)              │
                └───────────────────────────┬────────────────────────────┘
                                            │ Délègue (tâches lourdes / FS)
                                            ▼
           ┌──────────────────────────────────────────────────────────────────┐
           │                   Threadpool libuv (défaut = 4)                  │
           │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────┐ │
           │  │   Thread 1   │ │   Thread 2   │ │   Thread 3   │ │ Thread 4 │ │
           │  └──────────────┘ └──────────────┘ └──────────────┘ └──────────┘ │
           └──────────────────────────────────────────────────────────────────┘
```

### Tâches consommatrices du threadpool :
1. **Toutes les méthodes `fs.*` asynchrones** (car les kernels POSIX n'offrent pas d'I/O disque asynchrone universelle non-bloquante).
2. **`crypto.pbkdf2`**, **`crypto.scrypt`**, calculs de hachage lourds.
3. **`dns.lookup()`** (qui utilise la routine bloquante du système `getaddrinfo(3)`).
4. **`zlib.*`** (compression/décompression asynchrone).

### Le goulot d'étranglement :
Si votre application gère 10 requêtes simultanées effectuant un hachage `crypto.pbkdf2`, les 4 threads du pool sont saturés. Une simple lecture de fichier `fs.readFile()` se retrouve **bloquée dans la file d'attente**, dégradant la latence globale.

```bash
# Solution d'ingénierie : augmenter la taille du pool avant le démarrage du processus
# (Maximum = 1024, optimal = nombre de vCPUs alloués * 2 à 4)
UV_THREADPOOL_SIZE=16 node server.js
```

> **Attention DNS :** Préférez toujours `dns.resolve*()` à `dns.lookup()`. `dns.resolve()` utilise les sockets réseau asynchrones natifs sans jamais toucher au threadpool libuv.

---

## 3. Profiling et Diagnostic des Fuites Mémoire

En production, une fuite mémoire (*memory leak*) ne se manifeste souvent qu'après plusieurs jours d'exécution continue sous forme de crash avec l'erreur : `FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory`.

### 3.1 Cartographie du Heap V8

```
Total Memory (Process RSS)
├── C++ / Buffer Memory (Off-Heap)
└── V8 Managed Heap
    ├── New Space (Semi-espaces From/To, allocations éphémères, Scavenger GC)
    └── Old Space (Objets survivants, Mark-Sweep & Mark-Compact GC)
        ├── Old Pointer Space (Objets contenant des références)
        └── Old Data Space (Données brutes : chaînes, tableaux numériques)
```

### 3.2 Identifier les 3 Fuites Classiques en Node.js

1. **Closures capturant de larges scopes :**
   ```javascript
   let leaksHolder = [];
   function registerListener() {
     const massiveContext = Buffer.alloc(1024 * 1024 * 20); // 20 Mo
     return function inner() {
       // Même sans utiliser explicitement massiveContext,
       // certains moteurs V8 préservent le contexte lexical entier
       console.log('Event triggered');
     };
   }
   ```
2. **Caches en mémoire vive non bornés :**
   Stockage direct dans un objet `const cache = {}` sans stratégie d'éviction LRU (*Least Recently Used*) ni TTL.
3. **Écouteurs d'événements zombies :**
   Attacher un listener `emitter.on('data', handler)` sans jamais détacher `emitter.removeListener()` lors de la déconnexion d'un client.

### 3.3 Capture de Heap Snapshot sous charge (Node.js natif)

Pour diagnostiquer une fuite en production sans figer le processus avec un débogueur distant :

```javascript
import v8 from 'node:v8';
import fs from 'node:fs';

export function dumpHeapSnapshot(prefix = 'leak') {
  const fileName = `${prefix}-${Date.now()}.heapsnapshot`;
  const snapshotStream = v8.getHeapSnapshot();
  const fileStream = fs.createWriteStream(fileName);
  
  snapshotStream.pipe(fileStream);
  fileStream.on('finish', () => {
    console.log(`Heap snapshot généré : ${fileName}`);
  });
}

// Déclencheur conditionnel (ex: si la mémoire dépasse 85% de la limite V8)
setInterval(() => {
  const heap = v8.getHeapStatistics();
  const usageRatio = heap.used_heap_size / heap.heap_size_limit;
  if (usageRatio > 0.85) {
    dumpHeapSnapshot('critical-memory');
  }
}, 30000).unref();
```

*Analyse dans Chrome DevTools (onglet Memory) :*
- Comparez deux snapshots espacés dans le temps via la vue **Comparison**.
- Observez le **Retained Size** (la mémoire libérée si l'objet racine était détruit) plutôt que le **Shallow Size** (la taille brute de l'objet seul).
- Remontez l'arbre des **Retainers** pour trouver la référence vivante non nettoyée (ex. un tableau global ou un callback `EventEmitter`).

---

## 4. Concurrence à Grande Échelle : Cluster vs IPC

Puisqu'un processus Node.js n'utilise qu'un seul cœur CPU pour l'Event Loop, il est obligatoire d'exploiter la totalité des cœurs disponibles sur un serveur physique ou une instance cloud multi-cœurs.

### 4.1 Fonctionnement du Module `node:cluster`

Le module `cluster` s'appuie sur le mécanisme de duplication de processus système (`fork()`). Le processus Maître (*Primary*) lie le port réseau (ex. 8080) et distribue les descripteurs de sockets entrants aux processus enfants (*Workers*) selon une stratégie **Round-Robin** gérée par le master.

```
                    [ Client HTTP Requests :8080 ]
                                   │
                                   ▼
                    [ Primary Process (Master) ]
                    (Ne fait aucun calcul métier)
                     ├── IPC Channel (Unix Domain Socket)
                     │
         ┌───────────┼───────────┬───────────┐
         ▼           ▼           ▼           ▼
    [ Worker 1 ] [ Worker 2 ] [ Worker 3 ] [ Worker 4 ]
    (EventLoop)  (EventLoop)  (EventLoop)  (EventLoop)
```

```javascript
// cluster-server.js
import cluster from 'node:cluster';
import http from 'node:http';
import os from 'node:os';

const numCPUs = os.availableParallelism();

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} démarre le cluster sur ${numCPUs} cœurs...`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // Auto-guérison (Self-healing) : recréer un worker si l'un d'eux crashe
  cluster.on('exit', (worker, code, signal) => {
    console.warn(`Worker ${worker.process.pid} mort (${signal || code}). Spawn d'un remplaçant...`);
    cluster.fork();
  });
} else {
  // Les workers partagent le même port réseau 3000
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Réponse servie par le worker ${process.pid}\n`);
  }).listen(3000);

  console.log(`Worker ${process.pid} prêt.`);
}
```

---

## 5. Extensions Natives avec Node-API (N-API)

Lorsque les performances du JavaScript atteignent leurs limites théoriques (traitement d'images au pixel, calculs matriciels, cryptographie propriétaire), on utilise **Node-API** (anciennement N-API).

Node-API est une API C/C++ stable indépendante de la version interne du moteur V8. **Un addon compilé pour Node 18 continuera de fonctionner sur Node 22+ sans recompilation** (stabilité ABI garantie).

### Structure d'un Addon C++ Minimal :

```cpp
// fast_math.cc
#include <node_api.h>

napi_value FastMultiply(napi_env env, napi_callback_info info) {
  size_t argc = 2;
  napi_value args[2];
  napi_get_cb_info(env, info, &argc, args, NULL, NULL);

  double a, b;
  napi_get_value_double(env, args[0], &a);
  napi_get_value_double(env, args[1], &b);

  napi_value result;
  napi_create_double(env, a * b, &result);
  return result;
}

napi_value Init(napi_env env, napi_value exports) {
  napi_value fn;
  napi_create_function(env, NULL, 0, FastMultiply, NULL, &fn);
  napi_set_named_property(env, exports, "fastMultiply", fn);
  return exports;
}

NAPI_MODULE(NODE_GYP_MODULE_NAME, Init)
```

Consommation directe en JavaScript sans latence de sérialisation :
```javascript
import { createRequire } from 'node:module';
const require = createRequire(import.meta.url);
const fastMath = require('./build/Release/fast_math.node');

console.log(fastMath.fastMultiply(42.5, 2.0)); // 85
```

---

## 6. Synthèse d'Audit et Performance pour Profil Senior

| Domaine | Problème Détecté | Correction Senior Obligatoire |
| :--- | :--- | :--- |
| **I/O File System** | Séquences de lectures `fs.promises` bloquées sous forte charge | Augmenter `UV_THREADPOOL_SIZE` ou dédier un pool de workers spécifiques. |
| **Résolution Réseau** | Latence élevée due à `dns.lookup` | Basculer sur `dns.resolve*` ou configurer un cache DNS applicatif local. |
| **Mémoire V8** | Montée en flèche du tas sans redescente post-GC | Inspecter les Heap Retainers (closures, événements non libérés, singletons). |
| **Optimisation JIT** | Fonctions identifiées comme dé-optimisées par TurboFan | Maintenir un typage strict et stabiliser les constructeurs d'objets (Monomorphisme). |
| **Scalabilité CPU** | 1 seul thread utilisé sur une machine 32 cœurs | Déployer `node:cluster` ou conteneuriser avec orchestration multi-réplicas (K8s/PM2). |