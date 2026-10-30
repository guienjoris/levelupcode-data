# Quiz d'Évaluation — Node.js (Niveau Senior)

Ce quiz évalue votre compréhension approfondie des mécanismes internes de Node.js et du moteur V8 : le pipeline d'optimisation JIT (Hidden Classes et Inline Caches), le fonctionnement du threadpool de libuv, l'investigation avancée de fuites mémoire (Heap Snapshots et retainers), l'architecture multi-processus avec le module cluster, et la stabilité d'ABI avec Node-API.

### Question 1 : Optimisations V8 et Classes Cachées (Shapes)

Pourquoi l'ajout dynamique et désordonné de propriétés sur des instances d'objets après leur instanciation entraîne-t-il une pénalité de performance au niveau du compilateur TurboFan ?

* \[ \] A) Il provoque une dé-optimisation en forçant V8 à abandonner les Inline Caches monomorphiques pour basculer vers des dictionnaires mégamorphiques lents.

* \[ \] B) Il déclenche une recompilation complète du runtime C++ sous-jacent et force le redémarrage synchrone du moteur V8.

* \[ \] C) Il sature immédiatement la mémoire du threadpool libuv en multipliant les descripteurs de fichiers système ouverts.

* \[ \] D) Il empêche le ramasse-miettes (*Garbage Collector*) de balayer les objets éphémères stockés dans le New Space.

### Question 2 : Goulot d'étranglement du Threadpool libuv

Dans une API Node.js à fort trafic, quel type d'opération asynchrone risque de saturer les 4 threads par défaut du threadpool de libuv et de retarder indirectement les lectures de fichiers avec `fs.promises` ?

* \[ \] A) L'écoute d'un grand nombre de sockets TCP entrants configurés en mode non-bloquant via `net.createServer()`.

* \[ \] B) Des calculs intensifs de hachage de mots de passe exécutés simultanément à l'aide de `crypto.pbkdf2()` ou `crypto.scrypt()`.

* \[ \] C) Des requêtes HTTP sortantes effectuées via l'API moderne `fetch` vers des services tiers distants.

* \[ \] D) La sérialisation d'objets volumineux en mémoire à l'aide de l'utilitaire synchrone `JSON.stringify()`.

### Question 3 : Résolution de noms DNS et I/O système

Pourquoi est-il généralement recommandé pour un service haute performance d'utiliser `dns.resolve*()` plutôt que la méthode standard `dns.lookup()` ?

* \[ \] A) `dns.resolve*()` utilise les sockets réseau asynchrones de l'Event Loop sans mobiliser un thread du threadpool de libuv.

* \[ \] B) `dns.resolve*()` chiffre obligatoirement l'ensemble des trames UDP sortantes selon la norme TLS/DNS-over-HTTPS.

* \[ \] C) `dns.resolve*()` met en cache les enregistrements dans le registre mémoire du noyau sans expiration de TTL.

* \[ \] D) `dns.resolve*()` contourne les restrictions d'adresses IPv6 imposées par les fournisseurs d'accès internet.

### Question 4 : Analyse de Heap Snapshot et fuites mémoire

Lors de l'analyse d'un snapshot de mémoire V8 dans les outils de développement (Chrome DevTools), que représente précisément la métrique **Retained Size** d'un objet suspect ?

* \[ \] A) La quantité totale de mémoire vive allouée par le noyau du système d'exploitation pour le processus parent au lancement.

* \[ \] B) Le volume exact de mémoire binaire brute alloué hors du tas V8 (*off-heap*) sous forme d'instances de structures `Buffer`.

* \[ \] C) La mémoire qui serait immédiatement récupérée et libérée par le ramasse-miettes si cet objet et ses dépendances exclusives étaient détruits.

* \[ \] D) La taille superficielle occupée uniquement par les propriétés et pointeurs directs de l'objet sans inclure les valeurs ciblées.

### Question 5 : Architecture du module natif `cluster`

Comment le processus maître (*Primary*) distribue-t-il les connexions réseau entrantes entre les différents processus ouvriers (*Workers*) sur un système d'exploitation de type Linux ?

* \[ \] A) En ouvrant une connexion WebSocket persistante dédiée pour relayer le contenu binaire de chaque trame reçue.

* \[ \] B) En créant la socket d'écoute sur le port cible et en transmettant directement les descripteurs de fichiers aux ouvriers via IPC en Round-Robin.

* \[ \] C) En ordonnant aux processus enfants d'interroger la mémoire partagée du maître par polling toutes les dix millisecondes.

* \[ \] D) En réattribuant dynamiquement les adresses IP virtuelles de chaque carte réseau physique à chaque nouvelle requête.

### Question 6 : Extensions natives et Node-API (N-API)

Quel est l'atout technique prépondérant apporté par l'adoption de **Node-API** (N-API) pour le développement d'extensions natives compilées en C ou C++ ?

* \[ \] A) Une stabilité d'ABI (*Application Binary Interface*) garantissant qu'un module compilé reste compatible avec les futures versions majeures de Node.js sans recompilation.

* \[ \] B) L'élimination totale du besoin de compiler le code C++ grâce à une interprétation JIT native assurée directement par le moteur V8.

* \[ \] C) La possibilité d'exécuter des pointeurs de mémoire brute sans passer par les autorisations d'accès du système de fichiers hôte.

* \[ \] D) La conversion automatique et sans surcoût de n'importe quel script Python externe en binaire exécutable multiplateforme.

### Question 7 : Comportement du ramasse-miettes V8 (Generational Garbage Collection)

Dans l'architecture de gestion de mémoire du moteur V8, comment un objet JavaScript transite-t-il de l'espace **New Space** (Young Generation) vers l'espace **Old Space** ?

* \[ \] A) Dès que sa taille brute dépasse le seuil strict de 128 kilo-octets, quel que soit son temps d'existence en mémoire.

* \[ \] B) S'il survit avec succès à deux cycles consécutifs de collecte mineure (*Scavenger*) au sein des semi-espaces de mémoire.

* \[ \] C) Lorsqu'un appel explicite à la commande `global.gc()` est déclenché manuellement depuis l'environnement d'exécution.

* \[ \] D) Dès qu'il est référencé par au moins une promesse asynchrone non résolue rattachée à la phase des microtasks.

## Corrigé et Explications Détaillées

### 1. Réponse : **A**

**Explication :** V8 optimise l'accès aux champs des objets via des classes cachées (*Shapes* ou *Maps*) et des Inline Caches (IC). Lorsque les objets possèdent les mêmes propriétés initialisées dans le même ordre, ils partagent la même Shape, ce qui permet à TurboFan de générer un accès direct par décalage mémoire (monomorphisme). Modifier dynamiquement l'ordre ou la structure des propriétés casse cette régularité : le site d'appel bascule en état mégamorphique, forçant V8 à recourir à des recherches par tables de hachage génériques beaucoup plus lentes.

### 2. Réponse : **B**

**Explication :** Le threadpool de libuv comporte par défaut 4 threads et prend en charge certaines opérations bloquantes au niveau système : les opérations asynchrones sur le système de fichiers (`fs`), certaines résolutions DNS, la compression `zlib` et les méthodes cryptographiques coûteuses (`crypto.pbkdf2`, `scrypt`). Si plusieurs opérations cryptographiques s'exécutent simultanément, elles accaparent les threads disponibles, mettant en attente les opérations de lecture/écriture de fichiers qui dépendent du même pool partagé.

### 3. Réponse : **A**

**Explication :** `dns.lookup()` utilise sous le capot l'appel système bloquant et synchrone `getaddrinfo(3)`, ce qui oblige libuv à déléguer son exécution à son threadpool. À l'inverse, la famille de méthodes `dns.resolve*()` (ex. `dns.resolve4()`, `dns.resolveTxt()`) utilise la bibliothèque C-Ares qui émet directement des paquets réseau via des sockets non-bloquants rattachés à l'Event Loop, évitant ainsi d'encombrer le threadpool.

### 4. Réponse : **C**

**Explication :** La métrique *Shallow Size* représente l'espace mémoire directement alloué pour contenir la structure de l'objet lui-même. Le *Retained Size* correspond à l'ensemble de la mémoire qui redeviendrait immédiatement disponible si cet objet venait à être ramassé par le GC, englobant tous les objets enfants qui ne sont maintenus en vie que par une référence issue de ce parent. C'est l'indicateur principal pour traquer la racine d'une fuite mémoire.

### 5. Réponse : **B**

**Explication :** Sur les systèmes Unix/Linux, le processus primaire de `node:cluster` ouvre la socket d'écoute (le port TCP) et reçoit les requêtes de connexion. Grâce aux canaux de communication inter-processus (IPC sous forme de sockets de domaine Unix), le maître transmet le descripteur de fichier du client connecté à l'un des workers selon une politique Round-Robin, évitant ainsi les problèmes de famine de threads.

### 6. Réponse : **A**

**Explication :** Historiquement, les modules natifs C++ s'appuyaient directement sur les en-têtes internes de V8, ce qui imposait de recompiler les binaires (`node-gyp rebuild`) à chaque montée de version majeure de Node.js. Node-API propose une couche d'abstraction stable en C garantissant une compatibilité au niveau binaire (ABI) : un binaire compilé pour une version donnée de Node-API continue de tourner sur des versions plus récentes de Node.js sans nécessiter de recompilation.

### 7. Réponse : **B**

**Explication :** Le ramasse-miettes de V8 repose sur l'hypothèse générationnelle (*generational hypothesis*), selon laquelle la majorité des objets meurent très rapidement. Les allocations se font d'abord dans le New Space, géré par l'algorithme *Scavenger* (découpage en semi-espaces *From* et *To*). Si un objet reste accessible et survit à deux passages consécutifs du Scavenger, V8 le promeut (*promotion*) vers le Old Space, où le coût de collecte est plus élevé mais les passages moins fréquents.