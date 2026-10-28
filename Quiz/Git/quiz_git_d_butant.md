# Quiz d'Évaluation — Git (Niveau Novice)

Ce quiz teste vos connaissances sur les concepts fondamentaux de Git : les trois zones de travail, la configuration, le cycle `add`/`commit`, les branches, le fichier `.gitignore` et la synchronisation distante.

### Question 1 : Les 3 zones fondamentales

Dans l'architecture locale de Git, quel est le rôle exact de la **Staging Area** (aussi appelée *Index*) ?

* [ ] A) Elle sert d'espace de transit pour sélectionner précisément les modifications qui feront partie du prochain commit.
* [ ] B) C'est le serveur cloud distant géré par GitHub ou GitLab qui synchronise automatiquement toutes vos modifications en temps réel dès que vous sauvegardez un fichier dans votre éditeur de code favori.
* [ ] C) Elle stocke temporairement la corbeille de votre système d'exploitation afin d'éviter la suppression accidentelle de fichiers non sauvegardés lors d'une mauvaise manipulation en ligne de commande.
* [ ] D) C'est la branche principale de production sur laquelle sont déployés automatiquement les conteneurs Docker lors d'une release officielle.

### Question 2 : Initialisation d'un dépôt

Quelle commande transforme un dossier ordinaire en un dépôt Git prêt à enregistrer l'historique ?

* [ ] A) `git start --new-repository`
* [ ] B) `git init`
* [ ] C) `git create-repo --local-directory`
* [ ] D) `git setup --fresh`

### Question 3 : Enregistrement de modifications

Considérez la séquence d'instructions suivante sur un fichier existant :

```bash
echo "Nouvelle ligne" >> readme.md
git commit -m "docs: mise à jour du readme"
```

Si le fichier `readme.md` n'a pas été ajouté via `git add` au préalable, que se passe-t-il lors de l'exécution du commit ?

* [ ] A) Git crée le commit avec succès car toute modification enregistrée sur le disque dur est automatiquement incluse par défaut sans action manuelle.
* [ ] B) Git supprime définitivement le fichier `readme.md` du répertoire de travail pour signaler qu'une commande obligatoire n'a pas été respectée.
* [ ] C) Aucun commit n'est créé car les modifications n'étaient pas présentes dans la Staging Area.
* [ ] D) Le terminal bascule automatiquement dans l'éditeur Vim en demandant un mot de passe administrateur pour débloquer l'accès aux branches protégées.

### Question 4 : Fichiers ignorés (`.gitignore`)

Dans quelle situation un fichier listé dans le `.gitignore` continue-t-il malgré tout d'apparaître dans `git status` et d'être suivi par Git ?

* [ ] A) Si le fichier est déjà suivi (*tracked*) dans l'index depuis un commit antérieur à son ajout dans le `.gitignore`.
* [ ] B) Lorsque l'extension du fichier comporte plus de trois caractères alphabétiques consécutifs dans son arborescence de dossiers.
* [ ] C) Si le fichier a une taille totale strictement inférieure à 4 kilo-octets au moment de son écriture sur le disque.
* [ ] D) Lorsque le système d'exploitation utilise un encodage de caractères autre que l'UTF-8 par défaut.

### Question 5 : Création et changement de branche

Quelle commande permet à la fois de créer une branche nommée `feature-auth` et de basculer immédiatement dessus ?

* [ ] A) `git branch --create-and-enter feature-auth`
* [ ] B) `git switch -c feature-auth`
* [ ] C) `git checkout --force-new-target feature-auth`
* [ ] D) `git make-branch --jump-into feature-auth`

### Question 6 : Inspection des modifications

Quelle commande permet de comparer spécifiquement les modifications en cours dans votre répertoire de travail (*Working Directory*) par rapport au dernier commit enregistré ?

* [ ] A) `git log --history-graph`
* [ ] B) `git status --verbose-check`
* [ ] C) `git diff`
* [ ] D) `git show --all-files`

### Question 7 : Récupération distante

Que réalise concrètement la commande `git pull` lors de son exécution dans un projet connecté à un dépôt distant ?

* [ ] A) Elle télécharge les modifications distantes et les fusionne immédiatement dans la branche locale courante.
* [ ] B) Elle écrase l'ensemble de vos fichiers locaux non validés pour forcer votre machine à correspondre trait pour trait à la version hébergée en ligne.
* [ ] C) Elle déploie directement le code en production sur les serveurs Kubernetes configurés dans le pipeline d'intégration continue de l'organisation.
* [ ] D) Elle supprime l'ensemble des branches locales qui ne possèdent pas d'équivalent strict sur le serveur distant `origin`.

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** La *Staging Area* (ou index) est une zone intermédiaire locale. Elle permet de préparer et sélectionner chirurgicalement les modifications ou fichiers que vous souhaitez regrouper dans votre prochain commit, sans nécessairement tout valider d'un coup.

### 2. Réponse : **B**
**Explication :** `git init` crée le sous-dossier masqué `.git` contenant l'ensemble de la base de données locale du projet. C'est la commande standard pour initialiser un dépôt Git.

### 3. Réponse : **C**
**Explication :** Git n'inclut dans un commit que ce qui est indexé dans la *Staging Area*. Sans l'étape préalable `git add readme.md` (ou l'option raccourcie `-a` pour les fichiers déjà suivis), la commande `git commit` signale *« nothing added to commit but untracked/unstaged files present »* et n'enregistre rien.

### 4. Réponse : **A**
**Explication :** Le fichier `.gitignore` n'empêche que l'ajout futur de fichiers non suivis (*untracked*). Si un fichier a déjà été commité dans le passé, Git continuera de suivre ses changements tant qu'il n'aura pas été retiré de l'index via `git rm --cached <fichier>`.

### 5. Réponse : **B**
**Explication :** La commande moderne `git switch -c <nom>` (ou son équivalent historique `git checkout -b <nom>`) crée la nouvelle branche et positionne immédiatement le pointeur `HEAD` dessus.

### 6. Réponse : **C**
**Explication :** `git diff` examine les différences ligne par ligne entre votre répertoire de travail et le dernier instantané validé (ou l'index). 

### 7. Réponse : **A**
**Explication :** `git pull` est en réalité la combinaison de deux opérations consécutives : `git fetch` (téléchargement des commits distants vers votre base locale) suivi de `git merge` (fusion de ces commits dans la branche locale actuellement active).