# Quiz d'Évaluation — Architecture Microservices (Niveau Intermédiaire)

Ce quiz évalue votre compréhension des concepts intermédiaires appliqués aux microservices : la délimitation par *Bounded Contexts*, le principe de *Database-per-service*, la résilience avec le *Circuit Breaker*, les sagas de compensation et le rôle d'une API Gateway.

### Question 1 : Découpage fonctionnel et Bounded Contexts

Pourquoi est-il généralement déconseillé de créer une bibliothèque partagée contenant une classe de modèle unique `Product` importée par l'ensemble des microservices d'une plateforme d'e-commerce ?

* \[ \] A) Parce que TypeScript et Java interdisent formellement le partage de types de données au-delà d'un sous-réseau local.

* \[ \] B) Cela réintroduit un couplage fort de schéma entre services ayant des besoins métier distincts pour une même entité.

* \[ \] C) Cela surcharge le processeur des passerelles réseau lors de la sérialisation binaire des trames de données transitant entre conteneurs Docker.

* \[ \] D) La latence réseau augmente automatiquement de 200 ms pour chaque classe partagée entre deux conteneurs.

### Question 2 : Le pattern *Database-per-Service*

Quelle conséquence directe découle de l'adoption stricte du principe *Database-per-service* dans une architecture distribuée ?

* \[ \] A) L'impossibilité d'exécuter des jointures SQL multi-tables (`JOIN`) natives entre des données appartenant à deux domaines distincts.

* \[ \] B) L'obligation technique d'utiliser le même moteur de base de données relationnelle sur l'intégralité des microservices du cluster afin de garantir la synchronisation des disques.

* \[ \] C) La suppression totale du besoin d'effectuer des sauvegardes automatiques sur les environnements de production.

* \[ \] D) L'interdiction formelle d'écrire des index primaires sur les tables locales.

### Question 3 : Résilience et Circuit Breaker

Dans une implémentation du pattern *Circuit Breaker*, quel est le rôle de l'état **Demi-Ouvert** (*Half-Open*) ?

* \[ \] A) Rejeter 100 % des requêtes entrantes pour économiser les sockets réseau de la machine appelante.

* \[ \] B) Laisser passer un volume limité de requêtes pour évaluer si le service distant s'est rétabli avant de reprendre un trafic normal.

* \[ \] C) Basculer l'ensemble du trafic vers une base de données de secours en lecture seule située dans une autre zone géographique.

* \[ \] D) Supprimer les conteneurs défaillants au niveau de l'orchestrateur.

### Question 4 : Gestion des transactions distribuées via le pattern Saga

En cas d'échec à la troisième étape d'une Saga (par exemple : rupture de stock survenue après validation du paiement), comment le système restaure-t-il la cohérence des données ?

* \[ \] A) En exécutant des transactions de compensation explicites qui annulent logiquement les étapes validées au préalable.

* \[ \] B) En déclenchant un `ROLLBACK` SQL atomique natif partagé entre toutes les bases PostgreSQL via une liaison TCP synchrone globale.

* \[ \] C) En redémarrant immédiatement l'ensemble des conteneurs applicatifs impliqués dans la chaîne de traitement.

* \[ \] D) En effaçant la commande de la base sans laisser de traces.

### Question 5 : Couplage de communication synchrone vs asynchrone

Dans quel cas l'utilisation d'une communication synchrone via REST ou gRPC est-elle généralement la plus justifiée entre deux microservices ?

* \[ \] A) Pour déclencher l'envoi d'un lot de 50 000 newsletters promotionnelles nocturnes.

* \[ \] B) Lorsqu'un service a un besoin immédiat et bloquant du résultat d'une consultation pour construire sa réponse à l'utilisateur.

* \[ \] C) Pour diffuser une notification de mise à jour de stock à huit consommateurs différents sans attente de retour.

* \[ \] D) Afin de garantir qu'aucun message ne soit perdu si le service consommateur est temporairement hors ligne.

### Question 6 : Le rôle de l'API Gateway

Parmi les responsabilités suivantes, laquelle est **anormale** pour une API Gateway et relève plutôt de la logique interne d'un microservice dédié ?

* \[ \] A) Le calcul détaillé des règles de remise promotionnelle et l'application des taxes douanières selon le pays de livraison.

* \[ \] B) La terminaison TLS / SSL et la limitation du débit de requêtes (*Rate Limiting*) par adresse IP.

* \[ \] C) L'agrégation de plusieurs appels de microservices pour fournir une réponse optimisée à une application mobile.

* \[ \] D) La vérification de la signature cryptographique du jeton d'authentification (JWT).

### Question 7 : Problématique de l'observabilité distribuée

Quel mécanisme est indispensable pour tracer une requête utilisateur qui traverse successivement une API Gateway, un service de commande et un service de facturation ?

* \[ \] A) Injecter et propager un identifiant de corrélation unique (*Correlation ID*) dans les en-têtes HTTP de chaque appel inter-services.

* \[ \] B) Remplacer tous les bus de messages asynchrones par des fichiers textes partagés sur un volume réseau NFS monté en écriture concurrente.

* \[ \] C) Forcer l'ensemble des microservices à enregistrer leurs journaux applicatifs dans un seul fichier local `server.log`.

* \[ \] D) Utiliser un thread unique pour toutes les requêtes du cluster.

## Corrigé et Explications Détaillées

### 1. Réponse : **B**

**Explication :** En *Domain-Driven Design* (DDD), une entité ne porte pas la même signification d'un *Bounded Context* à l'autre. Le service de facturation s'intéresse aux montants hors taxe et à la TVA, tandis que le service de livraison s'intéresse au poids et au volume. Partager une classe unique force une synchronisation des déploiements et recrée le couplage d'un monolithe sous forme distribuée.

### 2. Réponse : **A**

**Explication :** Puisque les données sont réparties dans des bases physiques ou logiques étanches, le moteur SQL ne peut pas exécuter de jointure inter-bases directe. Les correspondances s'effectuent via l'orchestration applicative, des APIs dédiées ou la réplication asynchrone d'événements.

### 3. Réponse : **B**

**Explication :** L'état demi-ouvert sert de sonde. Après une phase de refroidissement où toutes les requêtes ont été bloquées (*Open*), le coupe-circuit autorise un faible pourcentage de requêtes test. Si elles réussissent, le circuit se referme (*Closed*) ; si elles échouent, il repasse en mode ouvert.

### 4. Réponse : **A**

**Explication :** Dans une Saga, on ne peut pas exécuter de `ROLLBACK` classique car les transactions locales ont déjà été validées (*committed*) indépendamment. La compensation consiste à exécuter une action métier inverse (ex: créditer un compte du montant préalablement débité, créer une écriture d'annulation).

### 5. Réponse : **B**

**Explication :** Le synchrone (REST/gRPC) convient aux requêtes de consultation directe où le client attend la réponse en temps réel (ex: vérifier la disponibilité immédiate d'un identifiant). Les mutations et chaînes de notifications s'accommodent bien mieux de l'asynchrone.

### 6. Réponse : **A**

**Explication :** Une API Gateway doit rester cantonnée à des préoccupations transverses (sécurité, routage, limitation de trafic, conversion de protocoles). Dès qu'on y insère de la logique métier (calculs de taxes, remises), elle devient un goulot d'étranglement fragile et difficile à maintenir.

### 7. Réponse : **A**

**Explication :** Dans un système distribué, une action utilisateur génère une cascade d'appels asynchrones ou synchrones. Sans *Correlation ID* (ou standard OpenTelemetry / W3C Trace Context) transmis d'en-tête en en-tête, il devient impossible d'agréger les logs d'une transaction donnée à travers les différents services.