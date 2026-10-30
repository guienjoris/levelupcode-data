# Quiz d'Évaluation — Node.js (Niveau Novice)

Ce quiz teste vos acquis sur les bases de l'environnement d'exécution Node.js : le moteur d'exécution, le fonctionnement asynchrone non-bloquant, les modules (CommonJS vs ES Modules), les variables globales et les modules natifs.

### Question 1 : Nature de Node.js

Laquelle de ces définitions décrit le plus fidèlement ce qu'est **Node.js** ?

* [ ] A) Un langage de programmation compilé conçu pour remplacer Java dans les architectures cloud.
* [ ] B) Un environnement d'exécution pour JavaScript reposant sur le moteur V8 de Google et la bibliothèque libuv.
* [ ] C) Un framework CSS permettant d'adapter automatiquement le rendu visuel aux serveurs headless.
* [ ] D) Une bibliothèque de templates front-end.

### Question 2 : Le Modèle Non-Bloquant

Pourquoi l'utilisation de méthodes synchrones comme `fs.readFileSync()` est-elle déconseillée sur un serveur Node.js en production ?

* [ ] A) Elle désactive le ramasse-miettes (*garbage collector*) du moteur V8 jusqu'au prochain redémarrage.
* [ ] B) Elle fige le thread principal, empêchant le serveur de traiter les requêtes des autres clients pendant la lecture.
* [ ] C) Elle corrompt l'encodage UTF-8 des données lorsque la taille du fichier dépasse 64 kilo-octets.
* [ ] D) Elle requiert des privilèges administrateur root.

### Question 3 : Système de Modules Moderne (ESM)

Pour pouvoir utiliser la syntaxe standard `import` / `export` de JavaScript nativement dans un projet Node.js sans outil de build externe, que devez-vous configurer ?

* [ ] A) Ajouter la directive `"type": "module"` dans le fichier `package.json` de votre application.
* [ ] B) Lancer systématiquement vos scripts en utilisant le paramètre d'exécution `--experimental-modules`.
* [ ] C) Renommer tous les répertoires du projet en leur ajoutant obligatoirement le suffixe `.mjs`.
* [ ] D) Remplacer le binaire Node par Deno.

### Question 4 : Gestion des Chemins avec le Module `path`

Pourquoi utilise-t-on la méthode `path.join('dossier', 'fichier.json')` au lieu d'une simple concaténation de chaînes `'dossier/' + 'fichier.json'` ?

* [ ] A) Elle compresse automatiquement le contenu du fichier lors de son chargement sur le disque dur.
* [ ] B) Elle adapte dynamiquement les séparateurs de dossiers selon le système d'exploitation cible (Windows vs Unix/Linux).
* [ ] C) Elle télécharge automatiquement les fichiers manquants depuis un serveur miroir distant sécurisé.
* [ ] D) Elle valide la conformité du format JSON.

### Question 5 : Variables d'Environnement et Objet Global

Où retrouve-t-on les variables d'environnement (telles que `PORT` ou `DATABASE_URL`) injectées dans l'application Node.js au démarrage ?

* [ ] A) Au sein de la propriété `process.env`.
* [ ] B) Dans la constante globale `globalThis.environment`.
* [ ] C) Dans le tableau interne `window.variables`.
* [ ] D) Via le fichier virtuel `fs.getEnv()`.

### Question 6 : Éléments Globaux Intégrés

Parmi ces identifiants familiers du navigateur, lequel est **totalement absent** dans l'environnement d'exécution Node.js standard ?

* [ ] A) `setTimeout`
* [ ] B) `console`
* [ ] C) `document`
* [ ] D) `setInterval`

### Question 7 : Rechargement Automatique en Développement

Depuis les versions récentes de Node.js (18+), quel argument officiel de ligne de commande permet de redémarrer le serveur à chaque modification de code sans installer d'outil tiers (comme nodemon) ?

* [ ] A) `node --live-reload server.js`
* [ ] B) `node --watch server.js`
* [ ] C) `node --hot server.js`
* [ ] D) `node --refresh server.js`

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** Node.js n'est ni un langage (c'est toujours du JavaScript), ni un framework web (contrairement à Express ou NestJS). C'est un environnement d'exécution (*runtime*) qui combine le moteur JavaScript V8 de Chrome pour la vitesse d'exécution et la bibliothèque libuv en C pour la gestion des I/O asynchrones.

### 2. Réponse : **B**
**Explication :** Node.js est *single-threaded* sur son fil d'exécution JavaScript principal. Toute opération bloquante (`readFileSync`, boucle infinie, calcul cryptographique lourd) mobilise ce fil unique : tant que l'opération n'est pas terminée, aucune autre requête réseau ni aucun callback en attente ne peuvent être traités.

### 3. Réponse : **A**
**Explication :** Par défaut, Node.js interprète les fichiers `.js` selon le format historique CommonJS (`require` / `module.exports`). En ajoutant `"type": "module"` à la racine du `package.json`, Node.js passe nativement en mode ES Modules (`import` / `export`) pour tous les fichiers `.js` du dossier.

### 4. Réponse : **B**
**Explication :** Windows utilise des antislashs (`\`) pour séparer les répertoires, tandis que macOS et Linux utilisent des slashs (`/`). La fonction `path.join()` inspecte la plateforme courante et normalise les séparateurs pour éviter les erreurs de chemin interplateformes.

### 5. Réponse : **A**
**Explication :** L'objet global `process` représente le processus en cours d'exécution de Node.js. Ses variables d'environnement configurées au niveau du shell ou d'un fichier `.env` sont accessibles en lecture dans l'objet dictionnaire `process.env`.

### 6. Réponse : **C**
**Explication :** L'objet `document` (ainsi que `window`) est spécifique aux navigateurs web pour manipuler l'arbre DOM de la page HTML. Côté serveur dans Node.js, il n'y a pas d'interface graphique native, donc `document` n'existe pas (alors que `console` et les minuteurs comme `setTimeout` font partie de la spécification commune).

### 7. Réponse : **B**
**Explication :** L'option `--watch` (disponible nativement depuis Node.js 18 et stabilisée en Node.js 20) surveille l'arborescence des fichiers importés et redémarre instantanément le processus dès qu'une sauvegarde est détectée, rendant superflus des outils tiers comme nodemon sur les configurations simples.