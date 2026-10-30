# Quiz d'Évaluation — Docker (Niveau Novice)

Ce quiz teste vos connaissances sur les principes fondamentaux de la conteneurisation avec Docker : la distinction avec les machines virtuelles, le cycle Dockerfile/Image/Conteneur, le mappage de ports, la persistance des données et l'utilisation de Docker Compose.

### Question 1 : Conteneurs vs Machines Virtuelles

Quelle est la différence architecturale clé entre un conteneur Docker et une machine virtuelle classique (comme sous VMware ou VirtualBox) ?

* [ ] A) Les conteneurs partagent le noyau de l'OS hôte sans embarquer d'OS invité complet, ce qui les rend bien plus légers et rapides au démarrage.
* [ ] B) Les conteneurs émulent un ensemble complet de composants matériels virtuels incluant BIOS, mémoire RAM et contrôleurs graphiques.
* [ ] C) Les machines virtuelles ne peuvent pas être connectées à un réseau local d'entreprise.
* [ ] D) Les conteneurs nécessitent obligatoirement un hyperviseur de type 1 matériel dédié.

### Question 2 : Le cycle Dockerfile, Image et Conteneur

Comment définit-on précisément la relation entre un **Dockerfile**, une **Image** et un **Conteneur** ?

* [ ] A) Le Dockerfile s'exécute directement en production, puis compile une image uniquement en cas d'erreur de mémoire.
* [ ] B) Le Dockerfile est le fichier recette qui sert à construire une image immuable, qui devient un conteneur une fois instanciée et lancée.
* [ ] C) L'image est une copie modifiable du disque dur alors que le conteneur représente le script de configuration en texte brut.
* [ ] D) Le conteneur est créé en amont pour générer le Dockerfile automatiquement.

### Question 3 : Mappage des ports réseau

Lorsque vous lancez un conteneur avec l'argument `-p 8080:3000`, que signifie exactement cette configuration ?

* [ ] A) Le conteneur écoute uniquement sur le port 8080 en bloquant tout accès sortant sur le port 3000.
* [ ] B) Le trafic arrivant sur le port 8080 de la machine hôte est redirigé vers le port 3000 à l'intérieur du conteneur.
* [ ] C) Le port 3000 de la machine hôte transfère ses paquets réseau vers le port 8080 interne du conteneur applicatif.
* [ ] D) L'application démarre automatiquement sur 8080 ports virtuels en parallèle.

### Question 4 : Persistance des données et volumes

Que devient la donnée écrite à l'intérieur d'un conteneur si celui-ci est supprimé avec la commande `docker rm` sans avoir configuré de volume ?

* [ ] A) La donnée est automatiquement sauvegardée dans le dossier temporaire `/tmp` de l'ordinateur hôte.
* [ ] B) Les données sont stockées de façon permanente sur le registre public Docker Hub.
* [ ] C) La donnée est définitivement perdue car la couche d'écriture du conteneur est éphémère et détruite avec lui.
* [ ] D) Les données sont compressées dans une image de secours nommée `backup:latest`.

### Question 5 : Optimisation du cache de build

Dans un Dockerfile pour une application Node.js ou Python, pourquoi copie-t-on le fichier de dépendances (`package.json` ou `requirements.txt`) avant le reste du code source ?

* [ ] A) Pour exploiter le cache des couches (*layers*) et éviter de réinstaller les dépendances si le code source seul a changé.
* [ ] B) Parce que le compilateur Docker refuse de traiter les fichiers dont le nom contient des points s'ils ne sont pas placés au début.
* [ ] C) Pour interdire au conteneur tout accès en lecture sur le reste du code applicatif en phase de test.
* [ ] D) Afin de réduire la mémoire vive requise par le processeur hôte pendant le build.

### Question 6 : Résolution DNS avec Docker Compose

Dans un fichier `docker-compose.yml` comportant un service nommé `api` et un autre nommé `db`, comment le service `api` contacte-t-il la base de données sur le réseau interne ?

* [ ] A) En renseignant l'adresse IP publique de la passerelle Internet du fournisseur d'accès.
* [ ] B) En utilisant le nom du service `db` comme nom d'hôte grâce au DNS interne fourni par Compose.
* [ ] C) En scannant l'ensemble des adresses IP locales de la plage `127.0.0.1` à `127.0.0.255`.
* [ ] D) En lisant le fichier physique `hosts` partagé manuellement par clé USB.

### Question 7 : Exécution de commandes dans un conteneur actif

Quelle commande permet d'ouvrir un terminal interactif à l'intérieur d'un conteneur en cours d'exécution nommé `mon-serveur` ?

* [ ] A) `docker run -it mon-serveur bash`
* [ ] B) `docker exec -it mon-serveur sh`
* [ ] C) `docker attach --read-only mon-serveur`
* [ ] D) `docker open mon-serveur /bin/terminal`

---

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** Contrairement à une machine virtuelle qui intègre un noyau et un système d'exploitation invité complet au-dessus d'un hyperviseur, un conteneur n'est qu'un ensemble de processus isolés (via les *namespaces* et *cgroups* de Linux) qui s'exécutent directement sur le noyau du système hôte. Cela explique leur démarrage quasi instantané et leur très faible consommation de ressources.

### 2. Réponse : **B**
**Explication :** Le **Dockerfile** est le fichier source contenant les instructions textuelles (`FROM`, `COPY`, `RUN`). La commande `docker build` en produit une **Image**, qui est un artefact binaire figé et immuable en lecture seule. Enfin, `docker run` instancie cette image pour en faire un **Conteneur**, c'est-à-dire un processus vivant muni d'une couche d'écriture temporaire.

### 3. Réponse : **B**
**Explication :** La syntaxe de publication de port suit toujours la convention `HÔTE:CONTENEUR`. Par conséquent, `-p 8080:3000` connecte le port `8080` de votre ordinateur hôte au port `3000` exposé par l'application à l'intérieur du conteneur.

### 4. Réponse : **C**
**Explication :** Par défaut, le système de fichiers d'un conteneur repose sur une couche d'écriture éphémère (*writable container layer*). Dès que le conteneur est supprimé avec `docker rm`, cette couche est détruite. Pour conserver des données (comme celles d'une base MySQL/PostgreSQL), il est indispensable d'utiliser un volume Docker (`-v mon-volume:/chemin/dans/conteneur`).

### 5. Réponse : **A**
**Explication :** Docker met en cache le résultat de chaque ligne d'instruction du Dockerfile. Si une ligne change, Docker invalide le cache de cette instruction et de toutes celles qui suivent. Copier `package.json` et exécuter l'installation des dépendances en premier permet de réutiliser cette étape mise en cache tant que les dépendances ne changent pas, même si vous modifiez vos fichiers sources quotidiens.

### 6. Réponse : **B**
**Explication :** Docker Compose crée automatiquement un réseau privé virtuel dédié aux services déclarés. Il intègre un serveur DNS embarqué qui associe le nom de chaque service (ex: `db`) à l'adresse IP interne du conteneur correspondant, ce qui évite d'avoir à coder des adresses IP en dur.

### 7. Réponse : **B**
**Explication :** `docker exec` sert à lancer une commande à l'intérieur d'un conteneur qui est **déjà en train de tourner** (les options `-i` pour interactif et `-t` pour allouer un pseudo-terminal permettent d'avoir un shell réactif). La commande `docker run` aurait quant à elle tenté d'instancier un nouveau conteneur à partir d'une image.