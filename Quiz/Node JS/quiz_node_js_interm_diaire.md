# Quiz d'Évaluation — Node.js (Niveau Intermédiaire)

Ce quiz teste votre compréhension des mécanismes avancés de Node.js : le cycle des phases de l'Event Loop (libuv), la priorité des microtasks, la mémoire binaire des Buffers, les Streams et la contre-pression, le pattern `EventEmitter`, ainsi que le parallélisme applicatif.

### Question 1 : Priorité d'exécution des microtasks

Soit l'extrait de code suivant exécuté au démarrage d'un script Node.js :

```javascript
setTimeout(() => console.log('A'), 0);
Promise.resolve().then(() => console.log('B'));
process.nextTick(() => console.log('C'));
setImmediate(() => console.log('D'));
```

Quel est l'ordre exact d'affichage des deux premiers caractères dans la console ?

* [ ] A) `A` puis `B`
* [ ] B) `B` puis `C`
* [ ] C) `C` puis `B`
* [ ] D) `D` puis `A`

### Question 2 : Allocation mémoire des Buffers

Quelle affirmation décrit correctement les spécificités de l'allocation mémoire de la classe native `Buffer` dans Node.js ?

* [ ] A) Un Buffer est alloué hors du tas V8 (*off-heap*) sous forme de mémoire brute de taille fixe qui ne s'étend pas dynamiquement après instanciation.
* [ ] B) Sa mémoire réside directement dans le tas JavaScript V8 et double automatiquement de taille dès qu'on dépasse sa capacité initiale.
* [ ] C) Les Buffers ne peuvent contenir que du texte encodé en UTF-16 et refusent la manipulation de trames binaires arbitraires.
* [ ] D) L'allocation d'un Buffer nécessite obligatoirement l'utilisation d'une promesse asynchrone pour éviter de bloquer l'Event Loop.

### Question 3 : Gestion de la contre-pression (*Backpressure*) dans les flux

Dans le traitement de données par Streams, à quel problème répond directement la notion de contre-pression (*backpressure*) ?

* [ ] A) La corruption des trames réseau lors d'un basculement automatique de protocole de HTTP/1.1 vers HTTP/2.
* [ ] B) Le blocage du garbage collector V8 provoqué par la création excessive de fermetures lexicales au sein d'une boucle synchrone.
* [ ] C) L'accumulation excessive de données en mémoire tampon lorsque la source de lecture produit des morceaux plus rapidement que la cible d'écriture ne peut les consommer.
* [ ] D) L'interdiction faite au système d'exploitation d'écrire des fichiers de log sur un disque partitionné en lecture seule.

### Question 4 : Comportement de l'objet `EventEmitter` face aux erreurs

Que se produit-il par défaut dans une application Node.js lorsqu'un objet dérivé d'`EventEmitter` émet un événement nommé `'error'` (`emitter.emit('error', err)`) sans qu'aucun écouteur ne lui soit préalablement rattaché ?

* [ ] A) L'erreur est discrètement ignorée et consignée dans le fichier journal interne du runtime sans interrompre le processus.
* [ ] B) Node.js lève une exception fatale non interceptée (*uncaughtException*) qui interrompt et fait crasher immédiatement le processus.
* [ ] C) L'événement est converti de manière transparente en une promesse rejetée gérée par l'Event Loop lors de la phase suivante.
* [ ] D) L'application bascule automatiquement en mode secours et redémarre l'Event Loop depuis la phase des timers.

### Question 5 : Utilité de l'utilitaire `stream.pipeline`

Pourquoi privilégie-t-on l'usage de la fonction utilitaire `pipeline` (issue de `node:stream/promises`) par rapport au chaînage manuel d'événements `source.on('data', chunk => dest.write(chunk))` ?

* [ ] A) Elle compile le code JavaScript du stream en langage C natif pour doubler le débit binaire sur les cartes réseau compatibles.
* [ ] B) Elle chiffre automatiquement l'intégralité du flux à la volée avec l'algorithme AES-256 sans surcoût CPU mesurable.
* [ ] C) Elle régule nativement la contre-pression, nettoie les descripteurs de fichiers sous-jacents et propage correctement les erreurs sur toute la chaîne.
* [ ] D) Elle permet d'ignorer totalement les erreurs d'écriture sur le disque sans devoir déclarer de bloc `try...catch`.

### Question 6 : Concurrence lourde : `worker_threads` vs `child_process`

Dans quel cas technique est-il plus judicieux de recourir au module `node:worker_threads` plutôt qu'au module `node:child_process` ?

* [ ] A) Lorsqu'on souhaite lancer un script externe rédigé en Python ou exécuter un binaire compilé comme `ffmpeg`.
* [ ] B) Lorsque l'on doit effectuer un calcul lourd en pur JavaScript tout en partageant potentiellement de la mémoire via `SharedArrayBuffer` avec un coût d'instanciation réduit.
* [ ] C) Afin de déléguer la gestion des connexions HTTP directes au serveur web Apache préinstallé sur le serveur hôte.
* [ ] D) Pour contourner l'obligation d'avoir Node.js installé sur la machine qui exécute la tâche de fond.

### Question 7 : Arrêt ordonné (*Graceful Shutdown*)

Pourquoi est-il crucial d'invoquer la méthode `.unref()` sur le timer de sécurité lors d'une procédure d'arrêt d'un serveur HTTP en production ?

* [ ] A) Pour permettre à Node.js de quitter le processus dès que les tâches légitimes sont achevées, sans que ce timer de délai ne maintienne artificiellement l'Event Loop active.
* [ ] B) Pour libérer immédiatement les ports TCP du système d'exploitation sans attendre la fin des requêtes HTTP en transit.
* [ ] C) Afin de forcer le redémarrage synchrone du cluster de conteneurs dans un environnement d'orchestration Kubernetes.
* [ ] D) Pour supprimer instantanément la mémoire allouée aux connexions persistantes de type Keep-Alive.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **C**

**Explication :** Node.js traite les microtasks avant de passer à la phase suivante de l'Event Loop ou d'exécuter les timers/immediates. Parmi les microtasks, la file de `process.nextTick()` possède une priorité absolue sur la file des promesses (`Promise.then`). Ainsi, `C` s'affiche en tout premier, suivi de `B`. Ensuite seulement, l'Event Loop exécute les macrotasks (`A` ou `D` selon l'initialisation du tick).

### 2. Réponse : **A**

**Explication :** Les instances de `Buffer` sont allouées hors du tas JavaScript standard de V8 (*off-heap* via les structures natives C++ de Node.js). Cette allocation évite d'alourdir le travail du ramasse-miettes (*Garbage Collector*) de V8 lors de la manipulation de fichiers volumineux ou de sockets réseau. Leur taille en octets est immuable une fois allouée.

### 3. Réponse : **C**

**Explication :** La contre-pression survient dès lors qu'un flux de lecture (*Readable stream*) produit des données plus rapidement que le flux d'écriture (*Writable stream*) ne peut les absorber (par exemple, lecture sur disque NVMe rapide vers un socket réseau à faible débit). Sans régulation, les tampons mémoire saturent la RAM jusqu'au crash de l'application.

### 4. Réponse : **B**

**Explication :** Dans la conception de `EventEmitter`, l'événement `'error'` dispose d'un traitement particulier. Si cet événement est émis et qu'aucun écouteur n'a été enregistré via `.on('error', ...)`, Node.js déclenche une exception fatale. À défaut d'un gestionnaire global `process.on('uncaughtException')`, le processus se termine brutalement avec un code d'erreur.

### 5. Réponse : **C**

**Explication :** Raccorder manuellement des streams via des écouteurs `'data'` expose à des fuites de mémoire et ne suspend pas la lecture lorsque la destination s'engorge. `stream.pipeline` connecte les différents flux, applique la mise en pause/reprise de la lecture (*backpressure handling*), garantit la fermeture des descripteurs de fichiers à la fin ou en cas d'échec, et transmet les erreurs dans un seul bloc d'interception.

### 6. Réponse : **B**

**Explication :** Les `worker_threads` s'exécutent au sein du même processus système (partage de l'espace d'adressage possible via `SharedArrayBuffer` ou transferts de `ArrayBuffer`, empreinte mémoire modérée de quelques mégaoctets pour une nouvelle instance V8). À l'inverse, `child_process` démarre un processus du système d'exploitation entièrement distinct (overhead supérieur de ~30 Mo), ce qui est idéal pour exécuter des binaires externes mais moins adapté au partage intensif de mémoire en JavaScript.

### 7. Réponse : **A**

**Explication :** Par défaut, un timer actif (`setTimeout`) maintient l'Event Loop vivante et empêche Node.js de clore son processus. En lui appliquant `.unref()`, on indique à l'Event Loop qu'elle n'a pas à attendre l'expiration de ce minuteur s'il ne reste plus aucune autre tâche en cours (comme des connexions HTTP encore ouvertes), permettant ainsi au serveur de s'éteindre naturellement dès que le nettoyage s'achève.