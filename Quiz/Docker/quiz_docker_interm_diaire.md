# Quiz d'Évaluation — Docker (Niveau Intermédiaire)

Ce quiz évalue votre maîtrise des techniques intermédiaires de conteneurisation avec Docker : les *multi-stage builds*, la topologie réseau, la persistance avancée, l'observabilité par *healthchecks* et les pratiques de sécurité en production.

### Question 1 : Mécanique des Multi-Stage Builds

Quel est le principal bénéfice technique apporté par l'utilisation d'une architecture *multi-stage build* dans un Dockerfile de production ?

* [ ] A) Elle compile automatiquement le code source pour plusieurs architectures processeur (ARM64 et AMD64) au cours d'un seul passage.

* [ ] B) Elle permet d'isoler les outils de compilation lourds dans une étape temporaire pour ne copier que les artefacts nécessaires dans l'image finale.

* [ ] C) Elle accélère la bande passante réseau en compressant les images Docker à l'aide de l'algorithme gzip avant leur transfert.

* [ ] D) Elle supprime l'obligation de déclarer une commande de démarrage explicite (`CMD` ou `ENTRYPOINT`) dans le conteneur final.

### Question 2 : Résolution DNS et réseaux Bridge

Quelle différence majeure existe-t-il entre le réseau `bridge` par défaut de Docker et un réseau personnalisé (*user-defined bridge*) créé manuellement ?

* [ ] A) Le bridge par défaut chiffre automatiquement toutes les communications TCP, alors qu'un réseau personnalisé transmet les données en clair.

* [ ] B) Le bridge personnalisé désactive l'isolation des processus et partage la table de routage de la machine hôte sans passer par NAT.

* [ ] C) Le bridge personnalisé active la résolution DNS automatique par nom de conteneur, ce que ne permet pas le réseau bridge par défaut.

* [ ] D) Le bridge par défaut limite strictement le débit réseau de chaque conteneur connecté à un mégaoctet par seconde.

### Question 3 : Choix du type de montage pour le développement local

Quel type de point de montage doit être privilégié lors du développement local pour permettre la synchronisation en direct du code (*Hot-Reloading*) entre l'ordinateur hôte et le conteneur ?

* [ ] A) Un volume anonyme généré dynamiquement lors de l'exécution de l'instruction `VOLUME` du Dockerfile.

* [ ] B) Un montage de type `tmpfs` alloué directement dans la mémoire vive de la machine de test.

* [ ] C) Un montage de liaison (*Bind Mount*) reliant directement un répertoire de l'hôte à un chemin du conteneur.

* [ ] D) Un volume nommé géré exclusivement dans le répertoire interne `/var/lib/docker/volumes`.

### Question 4 : Orchestration fine des dépendances avec Compose

Dans un fichier `docker-compose.yml`, comment s'assurer qu'un service d'API ne démarre qu'une fois que la base de données est pleinement prête à recevoir des connexions, et pas seulement lancée en arrière-plan ?

* [ ] A) En combinant un bloc `healthcheck` sur la base avec la directive `depends_on: { database: { condition: service_healthy } }`.

* [ ] B) En ajoutant simplement la clause standard `depends_on: ['database']` sans configuration supplémentaire.

* [ ] C) En insérant une directive `restart: always` avec un délai d'attente fixe de 60 secondes sur l'API.

* [ ] D) En déclarant la variable d'environnement `WAIT_FOR_PORT=5432` dans le bloc de configuration de l'API.

### Question 5 : Paramètres des Healthchecks

Dans la configuration d'un `HEALTHCHECK`, à quoi sert précisément le paramètre `--start-period` ?

* [ ] A) À définir la durée maximale autorisée pour qu'une commande de vérification renvoie un code de sortie valide.

* [ ] B) À accorder un temps de grâce au démarrage de l'application durant lequel les échecs de vérification ne comptent pas dans le seuil d'alerte.

* [ ] C) À espacer les vérifications régulières une fois que le conteneur a atteint un statut stable et opérationnel.

* [ ] D) À forcer le redémarrage automatique du démon Docker si le conteneur met trop de temps à s'initialiser.

### Question 6 : Sécurité et principe du moindre privilège

Pourquoi est-il fortement recommandé de spécifier la directive `USER` dans un Dockerfile destiné à la production ?

* [ ] A) Pour obliger Docker à attribuer un quota de mémoire vive maximal au processus qui tourne dans le conteneur.

* [ ] B) Afin d'interdire au conteneur de créer des connexions HTTP vers des adresses IP situées hors du réseau local.

* [ ] C) Pour éviter que l'application ne s'exécute avec les privilèges `root`, limitant ainsi l'impact d'une éventuelle faille d'évasion (*container escape*).

* [ ] D) Pour permettre au ramasse-miettes du système d'exploitation hôte de nettoyer les fichiers temporaires orphelins.

### Question 7 : Pilotes réseau et mode Host

Quel est le principal compromis technique lors de l'utilisation du pilote réseau `--network host` pour lancer un conteneur ?

* [ ] A) On gagne en performances réseau en éliminant la translation NAT, mais on perd l'isolation réseau entre le conteneur et la machine hôte.

* [ ] B) On isole parfaitement les interfaces réseau, mais la consommation de mémoire vive de la machine hôte augmente de façon critique.

* [ ] C) On autorise le chiffrement matériel des paquets, mais le conteneur devient incapable de lire ou d'écrire sur le disque dur.

* [ ] D) On supprime le besoin de volume de données, mais le conteneur ne peut plus être exécuté sur un noyau Linux standard.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** Le *multi-stage build* permet de scinder la construction en étapes distinctes (`FROM ... AS builder` puis `FROM ... AS runner`). Tout le contenu volumineux requis uniquement pour le build (compilateurs, linters, bibliothèques de développement) est abandonné avec l'étape temporaire. Seuls les fichiers de distribution finaux et les dépendances de production sont copiés dans l'image finale, ce qui réduit considérablement sa taille et sa surface d'attaque.

### 2. Réponse : **C**
**Explication :** Sur le réseau `bridge` par défaut fourni par Docker à l'installation, les conteneurs ne peuvent communiquer entre eux qu'en utilisant leurs adresses IP (sauf si l'on utilise le flag hérité `--link`). Sur un réseau bridge personnalisé créé via `docker network create`, Docker active un serveur DNS embarqué qui résout automatiquement les noms de conteneurs en adresses IP.

### 3. Réponse : **C**
**Explication :** Les montages de liaison (*Bind Mounts*) associent un chemin précis de la machine physique de développement à un répertoire interne du conteneur (`-v $(pwd)/src:/app/src`). Toute modification de code enregistrée dans l'IDE sur la machine hôte se répercute instantanément dans le conteneur, permettant aux outils comme nodemon, Vite ou Webpack de recharger l'application à chaud.

### 4. Réponse : **A**
**Explication :** La directive `depends_on: ['database']` simple garantit uniquement que le conteneur de base de données est lancé, mais pas que le moteur SQL a fini son initialisation interne. Pour synchroniser précisément le démarrage, il faut définir un `healthcheck` (ex: `pg_isready`) sur le service de données et déclarer `condition: service_healthy` dans le `depends_on` du service dépendant.

### 5. Réponse : **B**
**Explication :** Certaines applications lourdes (ex: JVM Java, serveurs avec grosses migrations de bases de données) mettent plusieurs secondes à démarrer. Le paramètre `--start-period` définit un intervalle d'amorce durant lequel les échecs du test de santé ne sont pas comptabilisés dans le quota de réessais (`--retries`), évitant ainsi de déclarer prématurément le conteneur comme `unhealthy`.

### 6. Réponse : **C**
**Explication :** Par défaut, les conteneurs exécutent leurs processus avec l'utilisateur `root` (`UID 0`). Si l'application héberge une faille exploitable permettant à un attaquant d'exécuter du code arbitraire et de briser l'isolation du conteneur (*container escape*), celui-ci obtiendrait des droits superutilisateur sur la machine hôte. Définir un utilisateur non-privilégié (`USER node` ou `USER appuser`) applique le principe du moindre privilège.

### 7. Réponse : **A**
**Explication :** En mode `--network host`, le conteneur ne dispose pas de son propre espace de noms réseau (*network namespace*) ni de son adresse IP privée : il s'attache directement à la pile réseau de l'hôte. Cela supprime la surcharge de la couche NAT et optimise le débit, mais supprime l'isolation réseau et empêche par exemple d'exécuter deux conteneurs qui écoutent sur le même port sur la même machine.