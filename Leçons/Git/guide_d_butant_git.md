# Bien Débuter avec Git (Niveau Novice)

Git est un **système de contrôle de version distribué** (*VCS - Version Control System*). Créé en 2005 par Linus Torvalds, il permet d'enregistrer l'historique des modifications d'un projet, de revenir en arrière en cas d'erreur et de collaborer à plusieurs sans écraser le travail d'autrui.

---

## 1. Git vs GitHub / GitLab : Ne pas confondre

* **Git** est l'outil logiciel installé sur votre machine. Il gère l'historique de vos fichiers localement, sans avoir besoin d'Internet.
* **GitHub, GitLab ou Bitbucket** sont des plateformes d'hébergement en ligne. Elles stockent des copies distantes de vos dépôts Git pour faciliter la sauvegarde, la revue de code et le travail en équipe.

---

## 2. Le Modèle Mental : Les 3 Zones de Git

Pour comprendre Git, il faut visualiser comment un fichier transite entre trois états sur votre ordinateur :

```
┌────────────────────┐       git add       ┌────────────────────┐      git commit      ┌────────────────────┐
│  Working Directory ├────────────────────►│   Staging Area     ├─────────────────────►│  Local Repository  │
│ (Dossier de travail│                     │ (Zone d'index /    │                      │  (Historique .git) │
│   fichiers éditables)│                     │  salle d'attente)  │                      │                    │
└────────────────────┘                     └────────────────────┘                      └────────────────────┘
```

1. **Working Directory (Répertoire de travail) :** Les fichiers réels que vous éditez dans votre éditeur (ex: VS Code).
2. **Staging Area (Zone de transit / Index) :** La zone tampon où vous sélectionnez précisément les modifications prêtes à être enregistrées.
3. **Repository (Dépôt local `.git`) :** La base de données interne qui conserve l'historique définitif sous forme d'instantanés (*snapshots*) appelés des **commits**.

---

## 3. Configuration Initiale

Avant de faire quoi que ce soit, Git a besoin de savoir qui vous êtes pour signer vos enregistrements :

```bash
# Définir votre identité
git config --global user.name "Jean Dupont"
git config --global user.email "jean.dupont@example.com"

# Définir le nom de branche par défaut
git config --global init.defaultBranch main
```

---

## 4. Démarrer un Projet

Deux situations se présentent :

### Option A : Initialiser un nouveau projet local
Placez-vous dans votre dossier de travail et lancez :
```bash
git init
```
Git crée un dossier masqué `.git` qui stockera tout l'historique du projet.

### Option B : Récupérer un projet existant depuis le web
```bash
git clone https://github.com/utilisateur/mon-projet.git
```
Cette commande télécharge l'intégralité du code et son historique complet.

---

## 5. Le Workflow Quotidien : Modifier, Indexer, Valider

### 5.1 Vérifier l'état : `git status`
C'est la commande à exécuter constamment pour savoir où vous en êtes :
```bash
git status
```
Elle liste :
* Les fichiers modifiés non indexés (*Changes not staged for commit*).
* Les nouveaux fichiers non suivis (*Untracked files*).
* Les fichiers prêts à être validés (*Changes to be committed*).

### 5.2 Préparer ses modifications : `git add`
Pour envoyer un fichier dans la *Staging Area* :
```bash
# Ajouter un fichier précis
git add index.html

# Ajouter tous les fichiers modifiés du dossier courant
git add .
```

### 5.3 Enregistrer un instantané : `git commit`
Un **commit** est une photo figée de votre projet à un instant précis. Il nécessite impérativement un message explicite :
```bash
git commit -m "feat: ajouter le formulaire de contact"
```

> **Règle d'or :** Un commit doit représenter une unité logique de travail cohérente (une correction de bug, l'ajout d'une fonction), pas une journée entière de bricolage sans fil conducteur.

---

## 6. Inspecter l'Historique : `git log` et `git diff`

* **`git diff`** : Affiche ligne par ligne les modifications apportées dans votre dossier de travail par rapport au dernier commit.
* **`git log --oneline`** : Affiche un résumé concis de tous les commits passés (identifiant SHA-1 court et message) :

```text
a1b2c3d feat: ajouter le formulaire de contact
e4f5g6h chore: initialisation du projet avec layout de base
```

---

## 7. Ignorer des Fichiers : Le fichier `.gitignore`

Certains fichiers ne doivent **jamais** être suivis par Git (mots de passe, variables d'environnement, dépendances volumineuses, fichiers générés) :
* Fichiers de configuration sensibles (`.env`)
* Dossiers de bibliothèques tierces (`node_modules/`, `vendor/`)
* Artefacts de compilation (`dist/`, `.next/`, `build/`)

Créez simplement un fichier nommé `.gitignore` à la racine :

```text
# Dépendances
node_modules/

# Secrets et variables locales
.env
.env.local

# Fichiers système
.DS_Store
Thumbs.db
```

---

## 8. Travailler avec les Branches

Une branche est un pointeur léger et mobile vers un commit. Elle permet d'expérimenter ou de créer une fonctionnalité sans toucher au code stable de la branche principale (`main`).

```
      (feature-login)  ──► [ Commit C ] ──► [ Commit D ]
                                 ▲
                                 │
(main) ──► [ Commit A ] ──► [ Commit B ]
```

### Commandes usuelles des branches

```bash
# Lister les branches existantes
git branch

# Créer et basculer immédiatement sur une nouvelle branche
git checkout -b feature-login
# (ou commande moderne : git switch -c feature-login)

# Revenir sur la branche principale
git checkout main
# (ou : git switch main)

# Fusionner le travail de la branche feature-login dans main
git merge feature-login

# Supprimer la branche locale devenue inutile
git branch -d feature-login
```

---

## 9. Synchronisation avec le Dépôt Distant

Quand votre projet est lié à GitHub ou GitLab :

* **`git push`** : Envoie vos commits locaux vers le serveur distant :
  ```bash
  git push -u origin main
  ```
* **`git pull`** : Récupère et fusionne les derniers commits publiés par vos collègues sur le serveur :
  ```bash
  git pull
  ```

---

## Résumé du Cycle de Travail

1. Modifier vos fichiers dans le projet.
2. Vérifier l'état avec `git status`.
3. Sélectionner les fichiers à valider avec `git add <fichier>`.
4. Créer l'instantané avec `git commit -m "message clair"`.
5. Envoyer vers le serveur distant avec `git push`.