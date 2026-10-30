# Initiation à Node.js (Niveau Novice)

Pendant longtemps, le langage JavaScript était cantonné aux navigateurs web pour animer des boutons ou valider des formulaires. En 2009, Ryan Dahl a extrait le moteur d'exécution de Google Chrome pour créer **Node.js**, permettant d'exécuter du JavaScript directement sur un serveur, une machine locale ou dans un conteneur.

## 1. Qu'est-ce que Node.js exactement ?

Node.js n'est **ni un langage de programmation**, **ni un framework**. 

C'est un **environnement d'exécution** (*runtime*) open-source pour JavaScript, construit sur :
- **Le moteur V8 de Google :** Il compile le code JavaScript directement en code machine ultra-rapide.
- **La bibliothèque libuv :** Une bibliothèque écrite en C qui gère les accès disques, le réseau et les opérations asynchrones.

```
┌────────────────────────────────────────────────────────┐
│                   VOTRE CODE JS                        │
├────────────────────────────────────────────────────────┤
│           APIs Node.js (fs, http, path, crypto)        │
├────────────────────────────┬───────────────────────────┤
│    Moteur V8 (Google)      │       libuv (C/C++)       │
│  - Parse le JavaScript     │  - Thread Pool            │
│  - Compile en code machine │  - Event Loop (I/O Async) │
└────────────────────────────┴───────────────────────────┘
```

Contrairement au navigateur, Node.js :
- **N'a pas accès au DOM** (`window`, `document`, `alert()` n'existent pas).
- **A un accès direct au système d'exploitation** (lire/écrire des fichiers sur le disque, ouvrir des connexions réseau TCP, manipuler des variables d'environnement).

---

## 2. Le Modèle Asynchrone et Non-Bloquant

La caractéristique fondamentale de Node.js est son modèle : **Single-Threaded, Event-Driven, Non-Blocking I/O**.

### 2.1 Bloquant (Synchrone) vs Non-Bloquant (Asynchrone)

Dans un serveur classique (comme Apache avec PHP traditionnel), chaque requête utilisateur mobilise un fil d'exécution (*thread*). Si la requête attend une réponse de la base de données pendant 200 ms, ce thread est bloqué et consomme de la mémoire en ne faisant rien.

Node.js utilise **un seul thread principal** pour exécuter votre code JavaScript, mais délègue les tâches longues (I/O : entrées/sorties disque, requêtes réseau) au système d'exploitation en tâche de fond.

```
Requête A (Lire un fichier) ──┐
                              ├─► [ Thread Principal ] ──► Délègue la lecture au système
Requête B (Calculer un prix) ──┘         │
                                         ▼ Traite la requête B immédiatement
                                 [ Fichier prêt ! ] ──► Exécute le callback de A
```

### 2.2 Démonstration concrète : bloquant vs asynchrone

```javascript
import fs from 'node:fs';

// ❌ BLOQUANT (À éviter en production sur un serveur) :
// Le serveur est figé pendant la lecture du gros fichier
const data = fs.readFileSync('gros-fichier.txt', 'utf-8');
console.log('Fichier lu');

// ✅ NON-BLOQUANT (Recommandé) :
// Le serveur continue de répondre aux autres utilisateurs pendant la lecture
fs.readFile('gros-fichier.txt', 'utf-8', (err, content) => {
  if (err) throw err;
  console.log('Fichier lu de façon asynchrone');
});
console.log('Ce message s\'affiche AVANT la fin de la lecture du fichier !');
```

---

## 3. L'Écosystème des Modules : CommonJS vs ES Modules

Node.js découpe le code en fichiers indépendants appelés **modules**. Deux systèmes coexistent :

### 3.1 CommonJS (Historique, syntaxe `require`)
C'est le système historique par défaut de Node.js :
```javascript
// math.js (Export)
function add(a, b) {
  return a + b;
}
module.exports = { add };

// app.js (Import)
const { add } = require('./math');
console.log(add(2, 3));
```

### 3.2 ES Modules (Standard moderne, syntaxe `import / export`)
C'est la syntaxe standard officielle de JavaScript. Pour l'activer dans Node.js, ajoutez `"type": "module"` dans votre fichier `package.json` :
```javascript
// math.js (Export)
export function add(a, b) {
  return a + b;
}

// app.js (Import)
import { add } from './math.js';
console.log(add(2, 3));
```
> **Bonne pratique :** Sur tout nouveau projet moderne, utilisez systématiquement les **ES Modules** (`import/export`).

---

## 4. Les Objets et Variables Globaux Indispensables

Bien que `window` n'existe pas, Node.js fournit l'objet global **`globalThis`** (ou simplement `global`) ainsi que des utilitaires intégrés :

- **`process` :** Donne des informations sur le processus en cours d'exécution.
  - `process.env` : Accède aux variables d'environnement (ex: `process.env.PORT`).
  - `process.exit(0)` : Arrête le programme Node.js.
  - `process.argv` : Récupère les arguments passés en ligne de commande (`node app.js --dev`).
- **`__dirname` & `__filename` :** Donnent le chemin absolu du dossier et du fichier courant (en CommonJS).
- **Les minuteurs :** `setTimeout()`, `setInterval()`, `setImmediate()`.

---

## 5. Les Modules Natifs Essentiels

Node.js intègre une bibliothèque standard riche sans avoir besoin d'installer de dépendances externes via `npm` :

| Module | Préfixe moderne | Rôle principal |
| :--- | :--- | :--- |
| **fs** | `node:fs/promises` | Manipulation de fichiers et dossiers (lecture, écriture, suppression). |
| **path** | `node:path` | Résolution de chemins de fichiers multiplateformes (Windows vs Linux/Mac). |
| **http** | `node:http` | Création de serveurs web et émission de requêtes HTTP bas niveau. |
| **crypto** | `node:crypto` | Hachage (SHA-256), génération d'UUIDs, chiffrement et sécurité. |
| **os** | `node:os` | Informations matérielles (CPU, mémoire libre, architecture système). |

### Exemple : manipuler des chemins proprement avec `path`
Ne concaténez jamais des chemins avec `'/'` ou `'\\'` car Windows et Linux gèrent les séparateurs différemment :
```javascript
import path from 'node:path';

// Génère automatiquement "dossier/sous-dossier/data.json" sur Mac/Linux
// et "dossier\sous-dossier\data.json" sur Windows
const filePath = path.join('dossier', 'sous-dossier', 'data.json');
```

---

## 6. Créer son Premier Serveur Web HTTP (Sans Framework)

Avant d'utiliser Express, Fastify ou NestJS, il est formateur de comprendre comment Node.js traite une requête HTTP brute avec son module natif :

```javascript
// server.js
import http from 'node:http';

const server = http.createServer((req, res) => {
  const { method, url } = req;

  // Configuration de l'en-tête de réponse (JSON)
  res.setHeader('Content-Type', 'application/json');

  if (url === '/' && method === 'GET') {
    res.statusCode = 200;
    res.end(JSON.stringify({ message: 'Bienvenue sur notre serveur Node.js !' }));
  } else if (url === '/health' && method === 'GET') {
    res.statusCode = 200;
    res.end(JSON.stringify({ status: 'OK', uptime: process.uptime() }));
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ error: 'Route non trouvée' }));
  }
});

const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Serveur actif sur http://localhost:${PORT}`);
});
```

Pour lancer le serveur en développement avec redémarrage automatique en cas de modification de fichier (disponible nativement depuis Node.js 18+) :
```bash
node --watch server.js
```

---

## 7. Les 4 Réflexes Clés pour Débuter sur Node.js

1. **Privilégiez toujours la version avec promesses de `fs` :** Importez `node:fs/promises` avec `async/await` au lieu des méthodes bloquantes `*Sync` ou des vieux callbacks imbriqués.
2. **Ne bloquez jamais l'Event Loop :** Évitez les boucles de calculs mathématiques lourdes et synchrones sur le thread principal (ex. chiffrer une image de 50 Mo avec une boucle for classique), sous peine de geler le serveur pour tous les utilisateurs.
3. **Préfixez vos imports natifs par `node:` :** Utilisez `import fs from 'node:fs'` plutôt que `'fs'`. Cela évite toute ambiguïté avec d'éventuels paquets tiers installés dans `node_modules`.
4. **Protégez vos variables sensibles :** Ne mettez jamais de mots de passe ou de clés API en dur dans le code ; lisez-les toujours via `process.env`.