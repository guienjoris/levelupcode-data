# Quiz d'Évaluation — Git Niveau Senior & Git Internals Architect

Ce quiz teste votre compréhension approfondie des mécanismes internes de Git : la base d'objets adressable par le contenu, les commandes de plomberie, l'exploitation du reflog, le débogage automatisé et les techniques d'optimisation pour monoripositories d'envergure.

---

### Question 1 : Anatomie de l'objet Blob

Quelles métadonnées sont stockées à l'intérieur d'un objet **Blob** dans le répertoire `.git/objects` ?

* [ ] A) Le nom du fichier source, ses permissions POSIX (`100644` ou `100755`), l'horodatage de dernière modification ainsi que son empreinte cryptographique calculée via l'algorithme SHA-1.
* [ ] B) Strictement le contenu brut du fichier compressé avec zlib précédé d'un en-tête indiquant son type et sa taille.
* [ ] C) L'historique différentiel compressé par rapport à la version précédente ainsi que la référence vers le commit parent direct ayant introduit la modification.
* [ ] D) Le chemin absolu du fichier depuis la racine du dépôt local.

---

### Question 2 : Commandes de plomberie

Quelle commande de plomberie bas niveau permet d'afficher en clair le contenu formaté et typé de n'importe quel objet Git présent dans la base à partir de son empreinte SHA-1 ?

* [ ] A) `git cat-file -p <hash>`
* [ ] B) `git inspect-internal-database --decompress-object <hash>`
* [ ] C) `git object-dump --raw-payload --format-pretty <hash>`
* [ ] D) `git unpack-file --stdout <hash>`

---

### Question 3 : Diagnostic de commits orphelins

Après avoir exécuté un `git reset --hard HEAD~5` par erreur sur une branche locale, vous tentez de récupérer votre travail mais le pointeur `HEAD` a été déplacé à plusieurs reprises depuis. Quel mécanisme permet de repérer l'empreinte du commit perdu ?

* [ ] A) Consulter `git reflog` pour identifier l'ancienne position de `HEAD` avant le reset, ou exécuter `git fsck --lost-found` pour isoler les commits devenus orphelins.
* [ ] B) Restaurer la corbeille du système d'exploitation où Git déplace temporairement les commits écrasés pendant 48 heures.
* [ ] C) Exécuter `git restore --staged-history --source=origin` pour forcer la reconstruction de l'arbre local.
* [ ] D) Lancer `git recover-head`.

---

### Question 4 : Automatisation de recherche de régression avec `git bisect`

Comment automatiser intégralement la recherche du commit responsable d'un bug sans intervention manuelle à chaque étape de la dichotomie ?

* [ ] A) En déléguant le diagnostic au serveur d'intégration continue via la commande `git bisect --ci-webhook-runner`.
* [ ] B) En exécutant `git bisect run <script_de_test>` une fois les bornes `good` et `bad` définies.
* [ ] C) En forçant la relecture séquentielle du journal de modifications grâce au paramètre `git bisect scan --all-commits-linear`.
* [ ] D) En invoquant `git bisect --auto-check-unit-tests-parallel`.

---

### Question 5 : Isolation d'environnements multiples sans clonage

Pour travailler simultanément sur deux branches distinctes sans basculer d'une branche à l'autre dans le même répertoire et sans avoir à recloner l'intégralité du dépôt distant, quel outil natif de Git doit-on exploiter ?

* [ ] A) `git worktree add <chemin> <branche>`
* [ ] B) `git checkout --duplicate-environment-sandbox <branche>`
* [ ] C) `git clone --shared-hardlinks-without-history <branche>`
* [ ] D) `git sparse-checkout isolate --branch=<branche>`

---

### Question 6 : Mécanisme de déduplication des répertoires (Objets Tree)

Si deux commits distincts modifient un seul fichier situé dans `src/utils/math.ts`, comment Git optimise-t-il le stockage des répertoires non modifiés (ex: `assets/` et `docs/`) ?

* [ ] A) Git crée une copie intégrale de tous les arbres de répertoires mais applique une déduplication au moment de l'exécution de `git gc`.
* [ ] B) Les nouveaux objets `tree` réutilisent simplement les mêmes empreintes SHA-1 préexistantes pour pointer vers les sous-arbres identiques.
* [ ] C) Git consigne uniquement un fichier de delta patch textuel au format unified diff dans le commit enfant.
* [ ] D) Les sous-dossiers sont mis en cache mémoire.

---

### Question 7 : Téléchargement optimisé de dépôts volumineux (Blobless Clone)

Quelle commande permet d'obtenir l'intégralité de l'historique et des branches tout en différant le téléchargement du contenu des blobs jusqu'au moment où les fichiers sont réellement extraits sur le disque ?

* [ ] A) `git clone --filter=blob:none <url>`
* [ ] B) `git clone --depth=1 --no-blobs-fast <url>`
* [ ] C) `git clone --sparse-checkout-deferred-assets <url>`
* [ ] D) `git clone --shallow-submodules-only <url>`

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** Dans le modèle de données interne de Git, un **blob** est agnostique du système de fichiers hôte. Il ne contient que la séquence d'octets brute du fichier (compressée avec `zlib`), précédée de l'en-tête `blob <taille>\0`. Le nom du fichier et les droits d'exécution sont stockés dans l'objet **tree** parent qui le référence. Deux fichiers aux noms différents mais au contenu strictement identique partagent un unique blob.

### 2. Réponse : **A**
**Explication :** `git cat-file` fait partie des commandes de plomberie fondamentales. Le flag `-p` (*pretty-print*) décompresse l'objet identifié par son hash, détermine son type (`blob`, `tree`, `commit`, `tag`) et en affiche le contenu structuré dans le terminal. L'option `-t` permet d'obtenir son type et `-s` sa taille.

### 3. Réponse : **A**
**Explication :** Un `git reset --hard` déplace les pointeurs de référence sans purger immédiatement les objets de la base de données. Le journal de références locales (`git reflog`) garde trace de tous les états successifs de `HEAD`. Si le reflog venait à manquer, `git fsck --lost-found` inspecte directement `.git/objects` pour répertorier les commits déconnectés du graphe (*dangling commits*).

### 4. Réponse : **B**
**Explication :** `git bisect run` prend en paramètre un script ou une commande shell (ex: `npm test`). Le script doit renvoyer le code de sortie `0` si le commit est sain, une valeur entre `1` et `127` (sauf 125) s'il est défaillant, et `125` si le commit ne peut pas être testé. Git automatise alors l'intégralité du parcours binaire sans interruption.

### 5. Réponse : **A**
**Explication :** `git worktree` permet de rattacher plusieurs répertoires de travail indépendants à une unique base de données `.git`. Cela autorise l'exécution simultanée de tests ou de builds sur des branches différentes sans multiplier les clones réseau et sans gaspiller d'espace disque.

### 6. Réponse : **B**
**Explication :** Git fonctionne par instantanés (*snapshots*) d'arbres Merkle. Si un dossier ou un fichier ne subit aucune modification entre deux commits, le nouvel objet `tree` réutilise directement le hash SHA-1 de l'ancien objet `tree` ou `blob`. Seuls les dossiers situés sur le chemin du fichier modifié génèrent un nouvel objet `tree`.

### 7. Réponse : **A**
**Explication :** L'option `--filter=blob:none` effectue ce qu'on appelle un *blobless clone*. Git télécharge l'intégralité du graphe de commits et tous les arbres de répertoires (`trees`), mais omet le contenu des fichiers (`blobs`). Les blobs nécessaires ne sont téléchargés à la demande que lors des commandes d'extraction (`git checkout`, `git switch`).