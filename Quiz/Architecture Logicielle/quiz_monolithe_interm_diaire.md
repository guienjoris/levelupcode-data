# Quiz d'Évaluation — Architecture Monolithique (Niveau Intermédiaire)

Ce quiz teste votre compréhension de l'architecture monolithique moderne et du patron *Monolithe Modulaire* : étanchéité des modules, gestion des données, communication en mémoire, transactions ACID et critères de découpage.

### Question 1 : Monolithe Spaghetti vs Monolithe Modulaire

Quelle distinction fondamentale différencie un **monolithe modulaire** d'un monolithe non structuré qualifié de « *Big Ball of Mud* » ?

* \[ \] A) L'interdiction d'exécuter l'application sur des processeurs multi-cœurs.
* \[ \] B) Une organisation interne en modules étanches exposant des contrats publics explicites et interdisant les accès directs à leurs composants privés, malgré une exécution au sein d'un processus unique.
* \[ \] C) L'obligation de compiler chaque fonction métier sous la forme d'un conteneur Docker séparé déployé sur une machine virtuelle distincte.
* \[ \] D) L'usage d'une base de données NoSQL.

### Question 2 : Règles d'accès aux données inter-modules

Dans un monolithe modulaire correctement architecturé, comment le module `Orders` doit-il récupérer les données d'un client gérées par le module `Users` ?

* \[ \] A) En invoquant l'API publique (façade) du module `Users` en mémoire ou en exploitant une projection locale, sans jamais effectuer de requête SQL `JOIN` directe sur la table privée de l'autre module.
* \[ \] B) En créant une vue SQL matérialisée partagée avec des droits d'écriture globaux sur l'ensemble des tables du système de production.
* \[ \] C) Par un appel HTTP externe sortant vers un proxy inverse Nginx distant.
* \[ \] D) En important directement l'entité interne ORM du module voisin.

### Question 3 : Préservation des frontières logiques

Quel mécanisme automatisé permet d'empêcher efficacement un développeur d'importer les fichiers internes d'un module tiers au fil de l'évolution du code ?

* \[ \] A) L'utilisation de règles d'analyse statique et de linters d'architecture (tels que Nx, ArchUnit ou Deptrac) intégrés au pipeline d'intégration continue.
* \[ \] B) Le redémarrage quotidien du serveur d'hébergement via une tâche planifiée Cron.
* \[ \] C) La suppression pure et simple des types TypeScript en production.
* \[ \] D) Le chiffrement asymétrique des fichiers sources sur le disque dur.

### Question 4 : Communication par bus d'événements en mémoire

Pourquoi est-il avantageux d'utiliser un bus d'événements interne (*in-memory event bus*) pour notifier d'autres modules (ex: envoi d'emails ou mise à jour de statistiques) suite à la validation d'une commande ?

* \[ \] A) Pour éliminer complètement le besoin d'une base de données relationnelle dans le projet.
* \[ \] B) Pour découpler le module émetteur, qui n'a pas besoin de connaître les modules récepteurs ni de bloquer son exécution sur leurs traitements respectifs.
* \[ \] C) Pour forcer l'exécution synchrone de tous les gestionnaires sur le même thread de calcul sans libérer la boucle d'événements.
* \[ \] D) Pour crypter les trames réseau internes.

### Question 5 : L'atout des transactions ACID dans le monolithe

Quel avantage architectural direct conserve un monolithe par rapport à une architecture microservices lors de la réalisation d'une opération métier critique (ex: virement de compte à compte) ?

* \[ \] A) La possibilité d'exécuter des mutations dans une unique transaction de base de données locale avec garantie ACID immédiate, sans orchestrer de sagas ni subir la cohérence à terme.
* \[ \] B) L'absence de failles de sécurité au niveau applicatif grâce au confinement de mémoire de l'interpréteur de code.
* \[ \] C) La réduction mécanique de 100 % de l'espace de stockage consommé par les index d'arbres B-Tree.
* \[ \] D) L'exécution sans concurrence possible.

### Question 6 : Critères d'extraction vers un service autonome

Dans quelle situation est-il techniquement **légitime** d'extraire un module d'un monolithe modulaire pour en faire un service indépendant ?

* \[ \] A) Pour réduire le nombre de lignes dans le fichier `package.json` du projet principal.
* \[ \] B) Dès qu'une classe métier dépasse 150 lignes de code TypeScript.
* \[ \] C) Lorsque le module présente des besoins d'élasticité et de ressources matérielles très asymétriques (ex: calculs lourds par GPU) ou un rythme de déploiement distinct avec une équipe dédiée.
* \[ \] D) Afin de pouvoir utiliser une couleur de syntaxe différente dans l'éditeur de texte des développeurs juniors.

### Question 7 : Performance des échanges inter-modules

Quelle est la caractéristique de performance majeure d'un appel synchrone entre deux modules au sein d'un monolithe modulaire comparé à un appel entre deux microservices ?

* \[ \] A) L'appel s'exécute par résolution directe d'adresse en mémoire vive (quelques nanosecondes à microsecondes), évitant la surcharge de sérialisation JSON et la latence réseau.
* \[ \] B) L'appel nécessite une synchronisation NTP obligatoire entre les horloges matérielles des processeurs.
* \[ \] C) L'appel est deux fois plus lent en raison de la compilation JIT.
* \[ \] D) Le compilateur refuse l'exécution si les deux modules partagent le même espace mémoire.

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** Un monolithe modulaire conserve l'avantage d'un déploiement unifié tout en imposant une séparation stricte des domaines métier (modules). Chaque module protège ses structures internes derrière une interface/façade publique, contrairement au monolithe "spaghetti" où tout le code est interconnecté sans règles de visibilité.

### 2. Réponse : **A**
**Explication :** L'étanchéité des données est primordiale : exécuter des `JOIN` inter-modules lie de façon rigide les modèles physiques de stockage. Le module `Orders` doit passer par la façade publique de `Users` ou répliquer/dénormaliser les quelques données dont il a besoin au moment de la transaction.

### 3. Réponse : **A**
**Explication :** L'analyse statique d'architecture (règles de frontières de modules de Nx, ArchUnit en Java/Kotlin, ou Deptrac en PHP) échoue la compilation ou le test CI dès qu'un import non autorisé tente de pénétrer dans le répertoire privé (`internal`) d'un autre module.

### 4. Réponse : **B**
**Explication :** Le bus d'événements en mémoire permet un découplage temporel et structurel. Le module émetteur annonce ce qui vient de se passer (`OrderPlacedEvent`), et les modules abonnés réagissent sans que l'émetteur n'ait de référence directe vers eux, préservant ainsi le principe de responsabilité unique.

### 5. Réponse : **A**
**Explication :** Dans un monolithe partageant la même instance de base de données, on bénéficie des transactions ACID natives du moteur SQL (`BEGIN ... COMMIT / ROLLBACK`). En microservices, ces opérations exigent des protocoles distribués complexes (comme les Sagas compensatoires) car les données résident sur des bases séparées.

### 6. Réponse : **C**
**Explication :** Extraire un microservice introduit des coûts opérationnels majeurs (réseau, pannes distribuées, monitoring). Cela n'a de sens que face à des exigences concrètes : besoin de dimensionnement matériel spécialisé (CPU/GPU intensif), contrainte technologique forte (nouveau langage), ou autonomie de déploiement d'une équipe dédiée.

### 7. Réponse : **A**
**Explication :** Dans un monolithe, un appel inter-module est un simple appel de fonction ou de méthode en mémoire vive (quelques cycles CPU). Dans une architecture microservices, ce même échange nécessite la sérialisation de données (JSON/Protobuf), le transit sur une pile réseau (TCP/HTTP), la traversée d'un routeur et la désérialisation, générant une latence de plusieurs millisecondes.