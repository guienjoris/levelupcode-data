# Node.js Avancé — Niveau Intermédiaire : Internals, Streams & Concurrence

Maîtriser Node.js au niveau intermédiaire exige de dépasser la simple écriture de callbacks ou d'APIs REST avec Express. Il est nécessaire de comprendre la mécanique fine du runtime : comment l'Event Loop cadence les tâches, comment manipuler des flux de données sans saturer la RAM grâce aux **Streams**, et comment dépasser la contrainte du thread unique pour les tâches lourdes.

---

## 1. L'Event Loop sous le capot (libuv)

Le thread principal de Node.js n'exécute pas les tâches asynchrones de manière indifférenciée. À chaque itération (*tick*), la boucle traverse **six phases séquentielles** régies par la bibliothèque C `libuv` :

```
                  ┌────────────────────────────────────────┐
            ┌───► │  1. Timers (setTimeout, setInterval)   │
            │     └──────────────────┬─────────────────────┘
            │                        ▼
            │     ┌────────────────────────────────────────┐
            │     │  2. Pending Callbacks (I/O système)    │
            │     └──────────────────┬─────────────────────┘
            │                        ▼
            │     ┌────────────────────────────────────────┐
            │     │  3. Idle, Prepare (usage interne)      │
            │     └──────────────────┬─────────────────────┘
            │                        ▼
            │     ┌────────────────────────────────────────┐
            │     │  4. Poll (lecture I/O, connexions)     │
            │     └──────────────────┬─────────────────────┘
            │                        ▼
            │     ┌────────────────────────────────────────┐
            │     │  5. Check (setImmediate)               │
            │     └──────────────────┬─────────────────────┘
            │                        ▼
            │     ┌────────────────────────────────────────┐
            │     │  6. Close Callbacks (socket.on('close')│
            └─────┴────────────────────────────────────────┘
```

### Microtasks : la priorité absolue (`process.nextTick` vs `Promise`)
Entre **chaque phase** de la boucle ci-dessus, Node.js vide systématiquement deux files prioritaires appelées les **Microtask Queues** :
1. `process.nextTick()` : file propriétaire Node.js, vidée en priorité absolue avant tout.
2. `Promise.then()` / `catch()` / `finally` : file standard ECMAScript.

```javascript
setTimeout(() => console.log('1. Timer (Macrotask)'), 0);
setImmediate(() => console.log('2. Immediate (Check phase)'));
Promise.resolve().then(() => console.log('3. Promise (Microtask)'));
process.nextTick(() => console.log('4. NextTick (Microtask prioritaire)'));

// Ordre garanti de sortie :
// 4. NextTick
// 3. Promise
// 1. Timer  (ou 2 selon le bootstrap d'entrée, mais toujours après les microtasks)
// 2. Immediate
```

> **Attention à `process.nextTick` :** Un appel récursif à `process.nextTick()` crée une famine d'E/S (*I/O starvation*) : l'Event Loop ne peut jamais passer à la phase suivante.

---

## 2. Buffers et Mémoire Binaire

JavaScript dans le navigateur traitait historiquement des chaînes de caractères. Node.js doit manipuler des trames TCP et des fichiers binaires : c'est le rôle de l'objet global **`Buffer`**.

* Un Buffer est alloué en **mémoire brute hors du tas V8** (*off-heap* C++ memory).
* Sa taille est fixe dès l'instanciation : il ne s'agrandit pas dynamiquement.

```javascript
import { Buffer } from 'node:buffer';

// Allouer 10 octets initialisés à zéro (sécurisé)
const bufSafe = Buffer.alloc(10);

// Écrire et encoder
const buf = Buffer.from('Hello 👋', 'utf-8');
console.log(buf);               // <Buffer 48 65 6c 6c 6f 20 f0 9f 91 8b> (octets hexadécimaux)
console.log(buf.length);        // 10 octets (l'émoji prend 4 octets en UTF-8, pas 1 !)
console.log(buf.toString('hex')); // Représentation hexadécimale brute
```

> ⚠️ **Sécurité :** Ne jamais utiliser `Buffer.allocUnsafe(size)` sans réécrire immédiatement par-dessus. Il alloue de la mémoire non nettoyée qui peut contenir des données sensibles résiduelles (clés privées, fragments de requêtes HTTP d'autres utilisateurs).

---

## 3. Streams : Maîtriser le flux sans saturer la RAM

Lire un fichier de 4 Go avec `fs.readFile()` tente de copier l'intégralité du fichier dans la RAM, provoquant souvent une erreur `JavaScript heap out of memory`. Les **Streams** permettent de découper et traiter la donnée morceau par morceau (*chunks* de 64 Ko par défaut).

Il existe 4 types fondamentaux :
- **Readable :** Source de lecture (`fs.createReadStream`, `req`).
- **Writable :** Destination d'écriture (`fs.createWriteStream`, `res`).
- **Duplex :** Les deux simultanément (ex: sockets TCP `net.Socket`).
- **Transform :** Modifie la donnée au passage (ex: compression `zlib.createGzip()`).

### 3.1 La gestion vitale de la Contre-Pression (*Backpressure*)
Si la lecture sur disque va à 500 Mo/s mais que le réseau vers le client n'accepte que 2 Mo/s, la mémoire tampon gonfle jusqu'au crash. C'est le problème de la contre-pression.

```javascript
// ❌ Mauvaise pratique : écoute manuelle sans gestion de flux
readable.on('data', (chunk) => {
  writable.write(chunk); // Risque d'engorgement si writable est lent
});

// ✅ Bonne pratique moderne : pipeline avec async iterators
import { pipeline } from 'node:stream/promises';
import fs from 'node:fs';
import zlib from 'node:zlib';

async function compressFile(sourcePath, destPath) {
  try {
    await pipeline(
      fs.createReadStream(sourcePath),
      zlib.createGzip(),
      fs.createWriteStream(destPath)
    );
    console.log('Compression terminée avec succès.');
  } catch (err) {
    console.error('Erreur lors du streaming :', err);
  }
}
```
`pipeline` régule automatiquement le débit, ferme les descripteurs de fichiers et propage les erreurs proprement.

---

## 4. Programmation Événementielle avec `EventEmitter`

L'objet `EventEmitter` du module `node:events` est le pivot architectural de la majorité des bibliothèques internes (les Streams et Serveurs HTTP héritent tous d'EventEmitter).

```javascript
import { EventEmitter, once } from 'node:events';

class PaymentGateway extends EventEmitter {
  processPayment(orderId, amount) {
    this.emit('payment:start', { orderId });
    
    setTimeout(() => {
      if (amount <= 0) {
        // En Node.js, émettre un événement 'error' sans écouteur CRASHE le processus !
        this.emit('error', new Error('Montant invalide'));
        return;
      }
      this.emit('payment:success', { orderId, amount });
    }, 100);
  }
}

const gateway = new PaymentGateway();

// Écoute synchrone
gateway.on('payment:success', ({ orderId }) => {
  console.log(`Commande ${orderId} validée`);
});

// Attente asynchrone d'un événement unique avec once()
async function waitForOrder() {
  const [data] = await once(gateway, 'payment:start');
  console.log('Paiement débuté pour :', data.orderId);
}
```

> **Règle absolue :** Vous devez **toujours** attacher un écouteur sur l'événement `'error'` (`emitter.on('error', ...)`), sinon Node.js lèvera une exception fatale non rattrapée qui tuera le processus.

---

## 5. Parallélisme et Concurrence : `worker_threads` vs `child_process`

Node.js reste mono-threadé pour l'Event Loop, mais propose deux techniques pour décharger le calcul lourd (CPU-bound) :

```
                ┌────────────────────────────────────────────────────────┐
                │          Besoin d'exécuter du calcul lourd ?           │
                └───────────────────────────┬────────────────────────────┘
                                            │
                     ┌──────────────────────┴──────────────────────┐
                     ▼                                             ▼
          Programme ou binaire externe                  Code JavaScript / V8
            (ex: ffmpeg, script bash, python)             (ex: hachage, compression, IA)
                     │                                             │
                     ▼                                             ▼
             [ child_process ]                             [ worker_threads ]
       Nouveau processus OS distinct                  Partage de mémoire possible
     Mémoire isolée (~30 Mo d'overhead)            Plus léger, utilise SharedArrayBuffer
```

### Exemple avec `worker_threads` :
```javascript
// main.js
import { Worker } from 'node:worker_threads';

function runHeavyTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker(new URL('./worker.js', import.meta.url), {
      workerData: data
    });

    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) reject(new Error(`Worker arrêté avec le code ${code}`));
    });
  });
}
```

```javascript
// worker.js
import { parentPort, workerData } from 'node:worker_threads';

// Calcul lourd qui ne bloque pas l'Event Loop du thread principal !
function computePrimes(max) {
  // ... calcul mathématique intensif ...
  return result;
}

const result = computePrimes(workerData.max);
parentPort.postMessage(result);
```

---

## 6. Résilience en Production : Arrêt Propre (*Graceful Shutdown*)

Un serveur en production ne doit pas être tué brutalement (`kill -9`). Il doit terminer les requêtes en cours, fermer les connexions à la base de données et libérer les ports :

```javascript
import http from 'node:http';

const server = http.createServer((req, res) => {
  setTimeout(() => res.end('OK'), 2000);
});

server.listen(3000);

function gracefulShutdown(signal) {
  console.log(`Signal ${signal} reçu. Fermeture ordonnée...`);

  // 1. Cesser d'accepter de nouvelles connexions HTTP
  server.close(async () => {
    console.log('Serveur HTTP fermé.');
    try {
      // 2. Fermer les pools de bases de données, queues Redis, etc.
      // await db.disconnect();
      console.log('Connexions DB libérées.');
      process.exit(0);
    } catch (err) {
      console.error('Erreur lors du nettoyage :', err);
      process.exit(1);
    }
  });

  // 3. Forcer l'arrêt si les requêtes en cours mettent trop de temps (sécurité)
  setTimeout(() => {
    console.error('Délai dépassé (force shutdown)');
    process.exit(1);
  }, 10000).unref(); // .unref() permet à Node de quitter même si ce timer tourne
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));
```

---

## Résumé des Règles d'Or Intermédiaires

1. **Ne sous-estimez pas `process.nextTick` :** Réservez-le aux nettoyages immédiats ou à la normalisation d'APIs synchrones/asynchrones.
2. **Utilisez toujours `stream.pipeline` :** Il évite les fuites mémoire et régule automatiquement la contre-pression.
3. **Capturez les événements `'error'` sur les `EventEmitter` :** Une erreur non écoutée fait planter l'ensemble de l'application.
4. **Déchargez le CPU sur des `worker_threads` :** Tout calcul supérieur à 10-20 ms sur le thread principal dégrade la latence perçue par tous les autres utilisateurs connectés.