# Masterclass Git — Niveau Senior & Git Internals Architect

À un niveau avancé, Git ne doit plus être perçu comme une simple suite de commandes magiques, mais comme un **système de fichiers adressable par le contenu** (*content-addressable filesystem*), par-dessus lequel est greffée une interface de gestion de versions.

Comprendre la tuyauterie interne permet de récupérer n'importe quel code réputé "perdu", d'automatiser le débogage dans des historiques massifs et d'optimiser les performances de dépôts monolithes hébergeant des millions de commits.

---

## 1. Sous le capot : La tuyauterie interne (`.git/objects`)

Tout élément dans Git est identifié par un hash SHA-1 (ou SHA-256) de 40 caractères hexadécimaux calculé à partir de son type, de sa taille et de son contenu compressé avec `zlib`.

Le dossier `.git/objects` stocke ces éléments sous la forme `ab/cd1234...` (les 2 premiers caractères forment le sous-dossier, les 38 suivants le nom du fichier).

### 1.1 Les 4 types d'objets immuables

```
┌────────────────────────────────────────────────────────┐
│                        COMMIT                          │
│  - tree: 4b825dc642cb6eb9a060e54bf8d69288fbee4904      │
│  - parent: 1a2b3c...                                   │
│  - author / committer                                  │
│  - message                                             │
└───────────────────────────┬────────────────────────────┘
                            │ pointe vers
                            ▼
┌────────────────────────────────────────────────────────┐
│                         TREE (Répertoire)              │
│  - 100644 blob a1b2c...   README.md                    │
│  - 040000 tree d3e4f...   src/                         │
└───────────────────────────┬────────────────────────────┘
                            │ pointe vers
                            ▼
┌────────────────────────────────────────────────────────┐
│                         BLOB (Contenu du fichier)      │
│  - Contenu binaire pur (sans nom de fichier ni droits) │
└────────────────────────────────────────────────────────┘

(4e type : TAG ANNOTÉ — Pointeur permanent avec signature PGP et métadonnées)
```

1. **Blob (*Binary Large Object*)** : Ne stocke que le contenu brut d'un fichier. Deux fichiers identiques avec des noms différents dans des dossiers distincts partagent exactement le même blob.
2. **Tree** : Représente un répertoire. Il associe des noms de fichiers, des permissions POSIX (`100644`, `100755`) et pointe vers des blobs ou d'autres sous-trees.
3. **Commit** : Fige l'état du projet. Il pointe vers un tree racine, un ou plusieurs commits parents, et consigne l'auteur, le committer, l'horodatage et le message.
4. **Tag annoté** : Référence immuable pointant vers un commit, incluant un message et optionnellement une signature cryptographique GPG.

### 1.2 Porcelaine vs Plomberie

* **Commandes de Porcelaine** (haut niveau pour les humains) : `git status`, `git commit`, `git checkout`.
* **Commandes de Plomberie** (bas niveau pour scripts et moteurs internes) :
  * `git hash-object -w <fichier>` : Calcule le SHA-1 et écrit l'objet dans `.git/objects`.
  * `git cat-file -p <hash>` : Décompresse et affiche le contenu formaté de n'importe quel objet.
  * `git cat-file -t <hash>` : Retourne le type de l'objet (`blob`, `tree`, `commit`, `tag`).
  * `git ls-tree <tree-ish>` : Inspecte la structure interne d'un tree.

---

## 2. Sauvetage & Analyse de Traçabilité : Le Reflog

Le pointeur `HEAD` indique la position courante. Même après un `git reset --hard` catastrophique ou la suppression accidentelle d'une branche, les commits ne sont pas effacés immédiatement : ils deviennent **orphelins** (*dangling commits*).

### 2.1 Fonctionnement du Reference Log

Le `reflog` consigne chaque déplacement de `HEAD` ou des branches locales, conservé généralement 30 à 90 jours avant d'être nettoyé par le ramasse-miettes (`git gc`).

```bash
# Consulter l'historique de tous les déplacements de tête
git reflog

# Exemple de sortie :
# 7a1b2c3 HEAD@{0}: reset: moving to HEAD~3   <-- Erreur ici
# 9f8e7d6 HEAD@{1}: commit: feat: core payment logic
# 4e5d6c7 HEAD@{2}: checkout: moving from main to feature
```

### 2.2 Procédure de restauration d'un commit perdu

Pour restaurer l'état exact avant le reset :

```bash
# Recréer une branche sur le commit orphelin identifié
git branch backup-recovery HEAD@{1}

# Ou repositionner directement HEAD
git reset --hard HEAD@{1}
```

### 2.3 Retrouver des commits sans reflog : `git fsck`

Si le reflog a été purgé ou corrompu, `git fsck` (*file system check*) analyse l'intégrité de la base d'objets et liste tous les objets non référencés :

```bash
git fsck --lost-found
# dangling commit 9f8e7d6...
```

---

## 3. Débogage Chirurgical Automatisé : `git bisect`

Lorsqu'une régression critique apparaît sur une base de code comportant des milliers de commits, l'inspection manuelle est inefficace. `git bisect` utilise la **recherche binaire dichotomique** ($O(\log n)$) pour identifier le commit fautif.

### 3.1 Déroulement manuel

```bash
git bisect start
git bisect bad                 # La version courante est corrompue
git bisect good v2.4.0         # Le tag ou commit v2.4.0 fonctionnait

# Git bascule automatiquement au milieu du segment de commits
# Testez l'application, puis déclarez :
git bisect good   # ou : git bisect bad

# Une fois le coupable identifié, revenez à l'état initial :
git bisect reset
```

### 3.2 Automatisation complète via script / CI

On peut déléguer l'évaluation à une suite de tests unitaires ou un script shell :

```bash
# Code 0 = good, Code 1 à 127 (sauf 125) = bad, Code 125 = skip (incompilable)
git bisect run npm test -- --runInBand
```

Git itère automatiquement jusqu'à imprimer : `9f8e7d6... is the first bad commit`.

---

## 4. Gestion des Monorepos et Dépôts Géants

Sur des dépôts de plusieurs dizaines de gigaoctets (ex: Chromium, Windows), cloner et indexer tout le projet sature le disque et la RAM.

### 4.1 Git Worktrees : Multi-tâches sans changer de branche

Plutôt que d'utiliser `git stash` ou de re-cloner le projet pour travailler sur deux branches en simultané, `git worktree` permet de lier plusieurs répertoires de travail distincts au **même dépôt `.git`** :

```bash
# Créer un dossier physique séparé pointant sur la branche 'hotfix-prod'
git worktree add ../projet-hotfix hotfix-prod

# Lister les environnements actifs
git worktree list

# Libérer l'espace une fois terminé
git worktree remove ../projet-hotfix
```

### 4.2 Sparse-Checkout : Téléchargement partiel de l'arborescence

Permet de n'extraire dans le répertoire de travail que les sous-dossiers pertinents pour votre équipe :

```bash
git clone --no-checkout https://github.com/org/monorepo.git
cd monorepo

# Activer le sparse-checkout en mode cone (haute performance)
git sparse-checkout init --cone

# N'extraire que les modules ciblés
git sparse-checkout set services/billing libs/shared-types
git checkout main
```

### 4.3 Shallow Clone et Blobless Clone

* **Shallow clone** : Télécharge uniquement la profondeur récente de l'historique :
  ```bash
  git clone --depth=1 <url>
  ```
* **Blobless clone** : Télécharge tout le graphe de commits et les trees, mais ne télécharge les blobs que lorsqu'un fichier précis est ouvert ou modifié :
  ```bash
  git clone --filter=blob:none <url>
  ```

---

## 5. Gestion des Dépendances Complexes : Submodules vs Subtrees

Quand une architecture requiert l'inclusion d'un dépôt Git dans un autre :

| Critère | `git submodule` | `git subtree` |
| :--- | :--- | :--- |
| **Stockage** | Pointeur vers un SHA-1 précis du dépôt distant. | Copie intégrale du code injectée dans le graphe parent. |
| **Clonage** | Requiert `--recurse-submodules` ou `submodule update --init`. | Transparent pour les développeurs tiers (clone normal). |
| **Complexité** | Risque fréquent d'état détaché (*detached HEAD*). | Commandes de push/pull vers le sous-dépôt plus verbeuses. |

### Commande typique submodule :
```bash
git submodule add https://github.com/org/core-lib.git libs/core-lib
git submodule update --init --recursive
```

---

## 6. Architecture & Maintenance des Objets : `Packfiles` et Garbage Collection

Au fil des commits, Git accumule des fichiers d'objets unitaires (*loose objects*). Pour optimiser l'espace et les transferts réseau, Git compresse ces objets dans des **Packfiles** (`.pack`) accompagnés d'index (`.idx`) en utilisant un algorithme de compression différentielle par deltas.

```bash
# Déclencher manuellement l'élagage et l'empilement optimisé
git gc --prune=now --aggressive
```

---

## 7. Résumé des Bonnes Pratiques Senior

1. **Ne paniquez jamais face à une perte apparente :** Tout commit créé localement vit dans `.git/objects` et est repérable via `git reflog` ou `git fsck`.
2. **Utilisez `git bisect run`** pour traquer automatiquement les bugs d'intégration obscure dans les historiques longs.
3. **Privilégiez les `git worktree`** aux bascules de branches intempestives lors des context-switches urgents.
4. **Adoptez les clones partiels (`--filter=blob:none`)** sur les projets volumineux pour préserver la bande passante et l'espace disque de vos équipes.