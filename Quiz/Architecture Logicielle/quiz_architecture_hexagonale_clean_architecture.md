# Quiz d'Évaluation — Architecture Hexagonale & Clean Architecture

Ce quiz évalue votre compréhension des concepts clés de l'architecture hexagonale (*Ports and Adapters*) et de la *Clean Architecture* : la règle de dépendance, la distinction entre ports et adaptateurs, la classification primaire/secondaire et la séparation du domaine.

---

### Question 1 : La Règle Fondamentale de Dépendance

Selon les principes de la *Clean Architecture* et de l'architecture hexagonale, dans quelle direction les dépendances du code source doivent-elles impérativement pointer ?

* [ ] A) Exclusivement vers l'intérieur, c'est-à-dire en direction des cas d'usage et des entités du domaine métier de haut niveau.
* [ ] B) Du cœur métier vers les pilotes d'accès aux bases de données pour accélérer l'exécution des requêtes SQL.
* [ ] C) Vers l'extérieur.
* [ ] D) De manière bidirectionnelle afin de permettre aux modèles du domaine d'appeler directement les contrôleurs HTTP.

---

### Question 2 : Distinction entre Port et Adaptateur

Dans l'architecture hexagonale, quelle est la définition exacte d'un **Port** ?

* [ ] A) Un connecteur réseau TCP/IP physique utilisé par Docker pour exposer l'application sur le port 8080 ou 3000.
* [ ] B) Une classe concrète qui contient les requêtes SQL natives et gère le pool de connexions vers le moteur PostgreSQL.
* [ ] C) Une interface contractuelle définie par le cœur de l'application spécifiant les interactions possibles avec l'extérieur.
* [ ] D) Le composant graphique chargé d'injecter automatiquement les feuilles de style CSS dans le navigateur.

---

### Question 3 : Ports Primaires vs Ports Secondaires

Quelle affirmation caractérise correctement un **port secondaire** (*Driven port* ou sortant) ?

* [ ] A) Il représente un cas d'usage directement déclenché par une action de l'utilisateur via une interface web ou une commande CLI.
* [ ] B) Il définit une interface de contrat pour une dépendance externe dont le cœur applicatif a besoin pour accomplir sa mission (ex: persistance, envoi de SMS).
* [ ] C) C'est un port réservé aux protocoles de communication secondaires non sécurisés.
* [ ] D) Il s'agit d'un contrôleur d'API REST.

---

### Question 4 : Pureté du Domaine Métier

Parmi les extraits de code suivants, lequel respecte le **mieux** l'isolation du domaine préconisée par la Clean Architecture ?

* [ ] A)
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: string;
  @Column()
  email: string;
}
```

* [ ] B)
```typescript
export class User {
  constructor(
    public readonly id: string,
    private _email: string
  ) {
    if (!_email.includes('@')) {
      throw new Error("Format d'adresse invalide");
    }
  }

  get email(): string {
    return this._email;
  }
}
```

* [ ] C)
```typescript
import express from 'express';

export class User {
  saveToSession(req: express.Request) {
    req.session.user = this;
  }
}
```

* [ ] D)
```typescript
import axios from 'axios';

export class User {
  async notify() {
    await axios.post('https://api.sms.com', { id: this.id });
  }
}
```

---

### Question 5 : Rôle des Adaptateurs Primaires (*Driving Adapters*)

Quel composant logiciel agit typiquement en tant qu'**adaptateur primaire** dans une application web ?

* [ ] A) Un contrôleur HTTP qui reçoit la requête entrante, extrait ses paramètres et invoque un cas d'usage via son port.
* [ ] B) Un composant pilote communicant directement avec les disques durs d'un cluster Kafka.
* [ ] C) Une interface TypeScript `RepositoryPort`.
* [ ] D) Le moteur de base de données relationnelle MySQL.

---

### Question 6 : Bénéfice pour les Tests Automatisés

Comment l'architecture hexagonale facilite-t-elle la rédaction de tests unitaires sur les cas d'usage ?

* [ ] A) En générant automatiquement des mocks graphiques dans le navigateur sans intervention du développeur.
* [ ] B) En permettant de tester la logique métier avec des implémentations de ports en mémoire (*InMemory*), sans démarrer de vraie base de données.
* [ ] C) En désactivant les assertions lors de l'exécution sur les serveurs d'intégration continue.
* [ ] D) En supprimant l'usage des tests fonctionnels.

---

### Question 7 : Inversion de Dépendance et Infrastructure

Si l'équipe décide de remplacer PostgreSQL par DynamoDB, quelles parties de la base de code doivent être modifiées dans une architecture hexagonale stricte ?

* [ ] A) L'ensemble des entités du domaine, les cas d'usage, ainsi que les schémas de validation HTTP.
* [ ] B) Uniquement l'adaptateur secondaire de persistance, le domaine et ses cas d'usage restant strictement intacts.
* [ ] C) Les interfaces des ports primaires et l'adaptateur de contrôleur web.
* [ ] D) L'intégralité du projet sans exception.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** La règle de dépendance impose que rien dans les cercles intérieurs (Domaine, Cas d'Usage) ne connaisse l'existence des éléments situés dans les cercles extérieurs (Contrôleurs, Bases de données, Frameworks). Les dépendances pointent toujours vers l'intérieur.

### 2. Réponse : **C**
**Explication :** Dans l'architecture *Ports & Adapters*, un port est une abstraction (typiquement une interface) appartenant au cœur applicatif. L'adaptateur est l'implémentation concrète (la tuyauterie technique) qui s'y branche.

### 3. Réponse : **B**
**Explication :** Les ports primaires (*Driving*) sont les points d'entrée par lesquels le monde extérieur sollicite l'application (ex: `CreateOrderPort`). Les ports secondaires (*Driven*) sont les interfaces définissant ce dont l'application a besoin vers l'extérieur pour fonctionner (ex: `OrderRepositoryPort`, `PaymentGatewayPort`).

### 4. Réponse : **B**
**Explication :** L'option B est du TypeScript pur : aucune dépendance envers un ORM (comme TypeORM dans l'option A), un framework HTTP (comme Express dans l'option C) ou un client HTTP externe (comme Axios dans l'option D). Le domaine protège ses invariants de façon autonome.

### 5. Réponse : **A**
**Explication :** L'adaptateur primaire prend l'initiative : il reçoit l'entrée du monde extérieur (une requête HTTP, un événement, une ligne de commande), la traduit dans un format compréhensible par le cas d'usage (une commande ou DTO), et appelle ce dernier.

### 6. Réponse : **B**
**Explication :** Puisque le cas d'usage ne dépend que d'interfaces (les ports secondaires), on peut lui injecter un faux repository qui stocke les données dans un simple tableau JavaScript (`InMemoryRepository`) lors des tests. Les tests tournent en quelques millisecondes et restent isolés de l'infrastructure.

### 7. Réponse : **B**
**Explication :** C'est le but premier de cette architecture : isoler les détails techniques. Remplacer un moteur de base de données consiste simplement à écrire une nouvelle classe qui implémente l'interface du port secondaire (`DynamoDbOrderRepository` au lieu de `PostgresOrderRepository`), sans toucher au cœur métier.