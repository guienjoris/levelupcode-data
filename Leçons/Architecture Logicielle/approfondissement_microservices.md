# Architecture Microservices — Niveau Intermédiaire

Passer d'une vision théorique des microservices à une mise en pratique concrète exige de comprendre les défis inhérents aux **systèmes distribués**. Lorsqu'une application est découpée en services autonomes exécutés sur des machines ou conteneurs distincts, les appels de fonctions en mémoire sont remplacés par des appels réseau, introduisant latence, risques de pannes partielles et complexité de synchronisation des données.

---

## 1. Définir les frontières : Bounded Contexts (DDD)

L'erreur la plus fréquente consiste à découper les services selon des critères techniques (ex. un service "Base de données", un service "Emails", un service "PDF"). Un découpage microservices sain s'effectue selon des **frontières métier** (*Bounded Contexts*), issues du *Domain-Driven Design* (DDD).

```
                      [ PLATEFORME E-COMMERCE ]
                                  │
         ┌────────────────────────┼────────────────────────┐
         ▼                        ▼                        ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ Service Commandes│    │ Service Catalogue│    │ Service Livraison│
│ (Order Context)  │    │ (Catalog Context)│    │(Shipping Context)│
│                  │    │                  │    │                  │
│ Notion de Produit│    │ Notion de Produit│    │ Notion de Produit│
│ = ID, prix payé, │    │ = Description,   │    │ = Poids, volume, │
│   quantité       │    │   images, SEO    │    │   dimensions     │
└──────────────────┘    └──────────────────┘    └──────────────────┘
```

> **Règle clé :** Chaque contexte possède sa propre définition d'un concept métier. Dans l'exemple ci-dessus, le mot "Produit" ne désigne pas le même schéma de données pour le catalogue que pour le transporteur. Tenter de créer un modèle `Product` unique partagé entre tous les services recrée un couplage fort (un monolithe distribué).

---

## 2. Gestion des données : Le pattern *Database-per-Service*

Pour garantir l'autonomie réelle des équipes et des déploiements, **chaque microservice doit posséder sa propre base de données privée**.

```
┌───────────────────┐               ┌───────────────────┐
│ Service Clients   │               │ Service Factures  │
└─────────┬─────────┘               └─────────┬─────────┘
          │ (accès exclusif)                  │ (accès exclusif)
          ▼                                   ▼
┌───────────────────┐               ┌───────────────────┐
│  DB PostgreSQL    │               │    DB MongoDB     │
│   (Clients)       │               │    (Factures)     │
└───────────────────┘               └───────────────────┘
```

### Pourquoi interdire le partage de base de données ?
1. **Éviter le couplage de schéma :** Si deux services lisent la même table SQL, modifier une colonne exige de coordonner et tester les deux services simultanément.
2. **Choix technologique adapté (*Polyglot Persistence*) :** Un service de recommandation peut exploiter Neo4j (graphe), un catalogue Elasticsearch (moteur de recherche), et la comptabilité PostgreSQL (relationnel strict).
3. **Sécurité et scalabilité :** Les charges de lecture/écriture d'un service n'impactent pas directement les performances du voisin.

> **Conséquence directe :** Les jointures SQL multi-tables (`JOIN`) entre deux domaines deviennent impossibles. Les regroupements de données s'effectuent via des APIs ou des flux d'événements.

---

## 3. Communication Inter-Services

Deux grands paradigmes coexistent pour faire dialoguer des microservices :

| Caractéristique | Communication Synchrone (HTTP/REST, gRPC) | Communication Asynchrone (Message Broker / Events) |
| :--- | :--- | :--- |
| **Mécanisme** | Requête directe avec attente immédiate de réponse. | Publication d'un événement dans une file/bus sans attente. |
| **Protocoles types** | REST (JSON via HTTP/2), gRPC (Protobuf). | RabbitMQ (AMQP), Apache Kafka, AWS SQS/SNS. |
| **Couplage temporel** | **Fort** : Les deux services doivent être disponibles en même temps. | **Faible** : Le récepteur peut consommer le message plus tard. |
| **Cas d'usage type** | Lecture de données temps réel en lecture seule. | Notifications, déclenchement de chaînes de traitement, mutations d'état. |

### Exemple de flux asynchrone (Event-Driven) :
1. L'utilisateur valide son panier.
2. Le `Service Commandes` enregistre la commande et publie l'événement :
   `OrderCreated { orderId: "123", customerId: "abc", total: 49.99 }`.
3. Le bus de messages (Kafka/RabbitMQ) distribue cet événement aux services abonnés :
   - Le `Service Paiement` initie le prélèvement bancaire.
   - Le `Service Stock` décrémente les réservations d'inventaire.
   - Le `Service Notification` prépare l'email de confirmation.

Le `Service Commandes` n'a pas besoin de connaître l'existence ni l'adresse réseau de ces autres services.

---

## 4. Résilience et Pannes en Cascade : Le Pattern *Circuit Breaker*

En environnement distribué, le réseau n'est pas fiable. Si le `Service Facturation` devient lent ou tombe en panne, tous les services qui l'appellent de façon synchrone risquent d'épuiser leurs threads de requêtes en attendant des timeouts, entraînant l'effondrement de l'ensemble du système (**panne en cascade**).

Le pattern **Circuit Breaker** (coupe-circuit) protège le système en surveillant les échecs :

```
          Requêtes normales (Taux d'échec bas)
       ┌────────────────────────────────────────┐
       ▼                                        │
┌──────────────┐   Échecs répétés / Timeouts  ┌──────────────┐
│    FERMÉ     ├─────────────────────────────►│    OUVERT    │
│  (Opération  │                              │  (Fail-fast  │
│   normale)   │◄─────────────────────────────┤   immédiat)  │
└──────────────┘    Succès partiels après     └──────┬───────┘
                    période de refroidissement        │
                               ▲                      │ Après délai
                               │                      ▼
                               │               ┌──────────────┐
                               └───────────────┤ DEMI-OUVERT  │
                                  Test avec    │ (Sonde       │
                               quelques appels │  prudente)   │
                                               └──────────────┘
```

- **Fermé (*Closed*) :** Les requêtes passent normalement vers le service distant.
- **Ouvert (*Open*) :** Le taux d'erreur dépasse un seuil (ex. 50 % d'échecs sur 10s). Le circuit coupe immédiatement la connexion : les appels échouent instantanément (*fail-fast*) sans saturer le réseau, ou renvoient une réponse dégradée (*fallback*).
- **Demi-ouvert (*Half-Open*) :** Après un délai d'attente (ex. 30 secondes), le système laisse passer un nombre limité de requêtes sondes. Si elles réussissent, le circuit repasse en mode fermé ; sinon, il redevient ouvert.

---

## 5. Cohérence des Données sans Transactions ACID : Le Pattern Saga

Dans un monolithe, enregistrer une commande et débiter un compte s'effectue dans une transaction SQL atomique unique (`BEGIN ... COMMIT`). Si une étape échoue, un simple `ROLLBACK` annule tout.

Avec des bases séparées, les transactions distribuées (Two-Phase Commit / 2PC) sont déconseillées car elles bloquent les ressources réseau et nuisent aux performances. On utilise à la place le **pattern Saga**.

Une Saga est une séquence de transactions locales autonomes. Chaque étape met à jour une base locale et déclenche la suite. Si une étape échoue, la Saga exécute des **transactions de compensation** pour annuler les effets précédents.

### Exemple de compensation :
```
Étape 1 : Service Commande  ──► Crée la commande (Statut: PENDING)  [Succès]
                                             │
                                             ▼
Étape 2 : Service Paiement  ──► Prélève la carte bancaire            [Succès]
                                             │
                                             ▼
Étape 3 : Service Stock     ──► Réservation article                  [ÉCHEC : Rupture]
                                             │
               ┌─────────────────────────────┴─────────────────────────────┐
               ▼                                                           ▼
Compensation 2 : Service Paiement                           Compensation 1 : Service Commande
Remboursement de la carte bancaire                          Passe le statut à CANCELLED
```

> La compensation ne "rembobine" pas le temps comme un rollback SQL : elle applique une **action inverse explicite** (ex. émettre un remboursement plutôt qu'effacer la transaction comptable).

---

## 6. L'API Gateway : Point d'Entrée Unique

Pour éviter que les clients mobiles et navigateurs n'aient à connaître les adresses de dizaines de microservices différents, on place une **API Gateway** en façade :

```
                  ┌───────────────────────────────┐
                  │    Navigateur / App Mobile    │
                  └───────────────┬───────────────┘
                                  │ Requête unique (api.monsite.com)
                                  ▼
                  ┌───────────────────────────────┐
                  │          API GATEWAY          │
                  │  - Routage d'URL              │
                  │  - Authentification / JWT     │
                  │  - Rate Limiting              │
                  │  - Agrégation de réponses     │
                  └───────┬───────┬───────┬───────┘
                          │       │       │
             ┌────────────┘       │       └────────────┐
             ▼                    ▼                    ▼
     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
     │Service Auth  │     │Service Panier│     │Service Avis  │
     └──────────────┘     └──────────────┘     └──────────────┘
```

L'API Gateway centralise les aspects transversaux (*cross-cutting concerns*) : vérification des jetons d'authentification, limitation du débit de requêtes (*rate limiting*) et masquage de l'arborescence réseau interne.

---

## Synthèse des Compromis Microservices

| Avantages réels | Coûts et contraintes induits |
| :--- | :--- |
| **Déploiements indépendants** : Mettre en production un correctif sur le panier sans redéployer le catalogue. | **Complexité d'observabilité** : Nécessite des identifiants de corrélation (*Correlation IDs*) et du traçage distribué (Jaeger, OpenTelemetry). |
| **Scalabilité ciblée** : Allouer 20 réplicas sur le service de recherche en période de soldes et 2 sur la facturation. | **Cohérence à terme (*Eventual Consistency*)** : Les données mettent quelques millisecondes ou secondes à se synchroniser partout. |
| **Isolation des pannes** : Un plantage du moteur de recommandation n'empêche pas l'utilisateur de valider son paiement. | **Gestion des réseaux défaillants** : Obligation d'implémenter des mécanismes de retry, de timeouts et de circuit breakers. |