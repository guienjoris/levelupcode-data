# Quiz d'Évaluation — Architecture Logicielle (Niveau Novice)

Ce quiz évalue votre assimilation des notions fondamentales d'architecture logicielle : le modèle client-serveur, l'architecture en couches, le pattern MVC et l'arbitrage entre monolithe et microservices.

### Question 1 : Le modèle Client-Serveur

Dans une application web reposant sur une architecture client-serveur classique, quelle responsabilité incombe principalement au **Client** (*Frontend*) ?

* [ ] A) Exécuter les requêtes SQL complexes directement sur la base de données relationnelle via des drivers TCP sécurisés.
* [ ] B) Collecter les actions de l'utilisateur, gérer l'affichage de l'interface et lui présenter les données retournées par le serveur.
* [ ] C) Conserver l'intégralité des données confidentielles de l'entreprise sur le disque dur local de la machine cliente pour éviter d'encombrer le réseau.
* [ ] D) Gérer l'authentification cryptographique des connexions physiques au routeur Wi-Fi de l'infrastructure d'hébergement cloud.

### Question 2 : Architecture en couches (*N-Tier*)

Pourquoi découpe-t-on une application en couches horizontales distinctes (Présentation, Métier, Accès aux données) ?

* [ ] A) Pour isoler les responsabilités afin de pouvoir modifier ou remplacer une couche sans devoir réécrire l'ensemble du système.
* [ ] B) Pour diviser par deux la consommation de mémoire vive de chaque conteneur Docker.
* [ ] C) Pour obliger le navigateur à télécharger plusieurs fichiers HTML simultanément lors de l'affichage initial de la page d'accueil.
* [ ] D) Pour empêcher les développeurs backend de toucher au code JavaScript exécuté sur le poste de travail du client.

### Question 3 : Le pattern MVC (*Modèle-Vue-Contrôleur*)

Dans le patron MVC, quel composant joue le rôle de « chef d'orchestre » en réceptionnant la requête de l'utilisateur pour ordonner la mise à jour des données et choisir l'affichage approprié ?

* [ ] A) La Vue
* [ ] B) Le Contrôleur
* [ ] C) La Passerelle Réseau
* [ ] D) Le Modèle

### Question 4 : Monolithe vs Microservices

Quelle est la principale caractéristique d'une **architecture monolithique** ?

* [ ] A) Elle répartit chaque fonction métier dans un conteneur indépendant déployé sur un réseau virtuel privé dédié.
* [ ] B) L'ensemble des fonctionnalités du système est regroupé et déployé sous la forme d'un projet exécutable unique.
* [ ] C) Elle interdit l'utilisation de bases de données relationnelles comme PostgreSQL ou MySQL pour des raisons de scalabilité verticale.
* [ ] D) Elle nécessite obligatoirement l'orchestration de plusieurs clusters Kubernetes pour fonctionner en environnement de développement local.

### Question 5 : L'arbitrage architectural pour un nouveau projet

Pourquoi est-il généralement recommandé à une petite équipe ou à un débutant de démarrer un projet sous forme de **monolithe modulaire** plutôt que de microservices ?

* [ ] A) Parce que les microservices n'apportent aucun bénéfice aux architectures web modernes, quelle que soit la taille de l'entreprise.
* [ ] B) Parce que les microservices introduisent d'emblée une forte complexité opérationnelle (gestion du réseau, déploiements multiples, latence, pannes partielles) qui ralentit le développement sans valeur immédiate.
* [ ] C) Parce qu'un monolithe modulaire empêche formellement l'utilisation du protocole HTTP pour communiquer avec les navigateurs web.
* [ ] D) Parce que les microservices sont incompatibles avec les frameworks modernes comme NestJS, Django ou Express.

### Question 6 : Le rôle du Modèle dans le MVC

Que représente concrètement le composant **Modèle** dans une application organisée selon le modèle MVC ?

* [ ] A) L'élément graphique de l'interface qui gère l'animation visuelle des boutons et des menus déroulants.
* [ ] B) Le fichier de configuration du serveur web (comme Nginx ou Apache) qui redirige les ports de communication.
* [ ] C) La représentation des données ainsi que les règles et validations métier associées à ces données.
* [ ] D) Le script de sauvegarde automatique des logs de production vers un bucket de stockage externe.

### Question 7 : Séparation des préoccupations (*Separation of Concerns*)

Parmi les pratiques suivantes, laquelle représente une **violation** flagrante de la séparation des couches ?

* [ ] A) Écrire une requête SQL directe au sein de la fonction de clic d'un bouton d'interface utilisateur.
* [ ] B) Faire appel à un service métier depuis un contrôleur HTTP pour valider un formulaire de paiement.
* [ ] C) Isoler la génération des factures PDF dans un module utilitaire dédié plutôt que dans le contrôleur principal.
* [ ] D) Utiliser une interface TypeScript pour définir le contrat d'un repository d'accès aux données.

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** Dans le modèle client-serveur, le rôle du client (navigateur web, application mobile) est d'afficher l'interface graphique, de réagir aux actions de l'utilisateur et d'envoyer des requêtes au serveur. Il ne doit jamais exécuter directement de requêtes SQL sur la base de données, pour des raisons évidentes de sécurité et d'intégrité.

### 2. Réponse : **A**
**Explication :** Le principal avantage de l'architecture en couches est le découplage des responsabilités. Si la logique de stockage change (par exemple, passage de MySQL à MongoDB), seule la couche d'accès aux données est modifiée : la couche métier et la couche de présentation restent inchangées.

### 3. Réponse : **B**
**Explication :** Le contrôleur est l'intermédiaire actif du pattern MVC. Il écoute les actions de l'utilisateur (requête HTTP, soumission de formulaire), sollicite le modèle pour appliquer les modifications ou récupérer les données, puis transmet ces données à la vue adéquate pour restitution.

### 4. Réponse : **B**
**Explication :** Un monolithe regroupe tout le code de l'application (authentification, catalogue, facturation, panier) dans un même socle de code et une unité de déploiement unique. C'est l'opposé des microservices, où chaque domaine est un service autonome avec son propre cycle de vie.

### 5. Réponse : **B**
**Explication :** Découper prématurément un projet en microservices engendre des coûts de gestion importants : gestion des pannes réseau, cohérence transactionnelle distribuée, observabilité complexe et outillage lourd (CI/CD multiple, orchestrateurs). Commencer par un monolithe propre et bien modulaire permet d'itérer beaucoup plus vite.

### 6. Réponse : **C**
**Explication :** Le Modèle encapsule l'état et la logique intrinsèque des données de l'application (par exemple : vérifier qu'un prix n'est pas négatif, qu'un email est valide ou persister les informations en base). Il est indépendant de la manière dont la donnée est affichée (la Vue).

### 7. Réponse : **A**
**Explication :** Placer des requêtes de base de données directement dans la couche de présentation crée un couplage fort, affaiblit la sécurité (risques d'injections ou exposition d'identifiants côté client) et rend le code impossible à tester unitairement sans monter une infrastructure complète.