# Architecture Hexagonale & Clean Architecture (Niveau Intermédiaire)

Dans une architecture en couches traditionnelle (*N-Tier* : Présentation $\rightarrow$ Métier $\rightarrow$ Données), le flux de dépendance descend jusqu'au stockage. Conséquence : le modèle métier finit souvent pollué par des décorateurs d'ORM, des bibliothèques HTTP ou des détails techniques de base de données.

L'**Architecture Hexagonale** (*Ports and Adapters*, Alistair Cockburn, 2005) et la **Clean Architecture** (Robert C. Martin, 2012) inversent cette relation : **le cœur métier devient le centre du monde et ne dépend d'aucun outil technique.**

---

## 1. La Règle de Dépendance (*The Dependency Rule*)

C'est la règle cardinale commune à ces approches :

> **Les dépendances dans le code source ne doivent pointer que vers l'intérieur, en direction des règles métier de haut niveau.**

```
┌────────────────────────────────────────────────────────┐
│  INFRASTRUCTURE (Frameworks, DB, HTTP, CLI, Stripe)    │  <- Monde extérieur (Détails)
│  ┌──────────────────────────────────────────────────┐  │
│  │  ADAPTATEURS (Controllers, Gateways, Presenters) │  │
│  │  ┌────────────────────────────────────────────┐  │  │
│  │  │  CAS D'USAGE / APPLICATION (Use Cases)     │  │  │
│  │  │  ┌──────────────────────────────────────┐  │  │  │
│  │  │  │  DOMAINE / ENTITÉS (Règles Pures)    │  │  │  │
│  │  │  └──────────────────────────────────────┘  │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
          ───► FLUX DE DÉPENDANCE : TOUJOURS VERS LE CENTRE ───►
```

* **Le Domaine n'a aucun import** lié à Express, NestJS, TypeORM, Prisma, PostgreSQL, Axios, etc.
* Si le framework web ou l'ORM change, **le code du domaine ne bouge pas d'une seule ligne**.

---

## 2. Architecture Hexagonale : Ports et Adaptateurs

La métaphore de l'hexagone illustre un noyau applicatif entouré d'interfaces de communication multiples.

### 2.1 Les Acteurs : Primaires vs Secondaires

On sépare le système selon le sens de l'initiative :

| Type | Rôle | Exemples |
|---|---|---|
| **Primaire** (*Driving* / Entrant) | Ce qui **déclenche** une action dans l'application | Requête HTTP, commande CLI, événement RabbitMQ consommé, test unitaire |
| **Secondaire** (*Driven* / Sortant) | Ce que l'application **appelle** pour accomplir sa tâche | Base de données SQL, API Stripe, passerelle SMTP, bucket S3 |

### 2.2 Ports vs Adaptateurs

```
[ Client HTTP ] ──► ( Adaptateur Primaire : UserController )
                               │
                               ▼
                    [ Port Primaire : ICreateUserUseCase ]
                               │
                               ▼
                    [ CŒUR MÉTIER / CAS D'USAGE ]
                               │
                               ▼
                    [ Port Secondaire : IUserRepository ]
                               │
                               ▼
( Adaptateur Secondaire : TypeOrmUserRepository ) ──► [ Base PostgreSQL ]
```

* **Port :** Une interface contractuelle définie **par et pour** le cœur métier.
  * *Port Primaire :* Déclare ce que l'application offre comme cas d'usage (ex: `CreateUserPort`).
  * *Port Secondaire :* Déclare ce dont l'application a besoin vers l'extérieur (ex: `UserRepositoryPort`, `PaymentGatewayPort`).
* **Adaptateur :** Une implémentation technique concrète qui traduit un protocole externe vers le langage du domaine.
  * *Adaptateur Primaire :* Un contrôleur HTTP ou un handler de commande CLI.
  * *Adaptateur Secondaire :* Une classe implémentant `UserRepositoryPort` à l'aide de requêtes SQL ou Prisma.

---

## 3. Mise en Pratique : Cas Concret en TypeScript

Imaginons le cas d'usage : **Créer une commande**.

### Étape 1 : Le Domaine (Zéro dépendance)

L'entité protège ses propres invariants sans aucune bibliothèque externe :

```typescript
// src/core/domain/order.ts
export class Order {
  constructor(
    public readonly id: string,
    public readonly customerId: string,
    private _amount: number,
    private _status: 'PENDING' | 'CONFIRMED' = 'PENDING'
  ) {
    if (_amount <= 0) {
      throw new Error("Le montant d'une commande doit être supérieur à zéro.");
    }
  }

  confirm(): void {
    this._status = 'CONFIRMED';
  }

  get amount(): number {
    return this._amount;
  }

  get status(): string {
    return this._status;
  }
}
```

### Étape 2 : Le Port Secondaire (Contrat sortant)

Le cœur déclare ce dont il a besoin, sous forme d'interface pure :

```typescript
// src/core/ports/order-repository.port.ts
import { Order } from '../domain/order';

export interface OrderRepositoryPort {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
}
```

### Étape 3 : Le Port Primaire & Le Cas d'Usage (Application)

Le cas d'usage orchestre la logique métier :

```typescript
// src/core/ports/create-order.port.ts
export interface CreateOrderCommand {
  orderId: string;
  customerId: string;
  amount: number;
}

export interface CreateOrderPort {
  execute(command: CreateOrderCommand): Promise<void>;
}

// src/core/use-cases/create-order.use-case.ts
import { Order } from '../domain/order';
import { CreateOrderPort, CreateOrderCommand } from '../ports/create-order.port';
import { OrderRepositoryPort } from '../ports/order-repository.port';

export class CreateOrderUseCase implements CreateOrderPort {
  constructor(private readonly orderRepository: OrderRepositoryPort) {}

  async execute(command: CreateOrderCommand): Promise<void> {
    const order = new Order(command.orderId, command.customerId, command.amount);
    order.confirm();
    await this.orderRepository.save(order);
  }
}
```

### Étape 4 : Les Adaptateurs d'Infrastructure

#### Adaptateur Secondaire (Persistance réelle en PostgreSQL)

```typescript
// src/infrastructure/adapters/postgres-order-repository.ts
import { OrderRepositoryPort } from '../../core/ports/order-repository.port';
import { Order } from '../../core/domain/order';

export class PostgresOrderRepository implements OrderRepositoryPort {
  constructor(private readonly dbClient: any) {}

  async save(order: Order): Promise<void> {
    await this.dbClient.query(
      'INSERT INTO orders (id, customer_id, amount, status) VALUES ($1, $2, $3, $4)',
      [order.id, order.customerId, order.amount, order.status]
    );
  }

  async findById(id: string): Promise<Order | null> {
    const row = await this.dbClient.query('SELECT * FROM orders WHERE id = $1', [id]);
    if (!row) return null;
    return new Order(row.id, row.customer_id, row.amount, row.status);
  }
}
```

#### Adaptateur Primaire (Contrôleur HTTP Express)

```typescript
// src/infrastructure/adapters/express-order.controller.ts
import { Request, Response } from 'express';
import { CreateOrderPort } from '../../core/ports/create-order.port';

export class ExpressOrderController {
  constructor(private readonly createOrderUseCase: CreateOrderPort) {}

  async handle(req: Request, res: Response): Promise<void> {
    try {
      await this.createOrderUseCase.execute({
        orderId: req.body.orderId,
        customerId: req.body.customerId,
        amount: Number(req.body.amount),
      });
      res.status(201).json({ message: "Commande validée avec succès." });
    } catch (error: any) {
      res.status(400).json({ error: error.message });
    }
  }
}
```

---

## 4. Les Bénéfices Concrets en Production

1. **Testabilité absolue sans base de données :**
   Pour tester unitairement `CreateOrderUseCase`, on implémente un faux repository en mémoire (`InMemoryOrderRepository`) sans jamais démarrer Docker ou PostgreSQL. Les tests s'exécutent en quelques millisecondes.
2. **Indépendance vis-à-vis des outils :**
   Remplacer PostgreSQL par MongoDB ou passer d'Express à Fastify ne modifie aucun fichier dans `src/core/`.
3. **Pérennité du code :**
   Les frameworks web vieillissent et passent de mode tous les 5 ans ; la logique de calcul de devis ou de TVA de votre entreprise reste inchangée pendant 15 ans.

---

## 5. Synthèse comparative : Couches vs Hexagonale

| Caractéristique | Architecture en Couches Classique | Architecture Hexagonale / Clean |
|---|---|---|
| **Direction des dépendances** | Descendante vers la base de données | Centripète (vers le domaine métier) |
| **Position de la DB** | Au fondement de l'architecture | Reléguée au rang de simple adaptateur externe |
| **Facilité de remplacement d'outil** | Complexe (l'ORM contamine souvent le métier) | Triviale (il suffit d'écrire un nouvel adaptateur) |
| **Complexité initiale** | Faible (rapide sur petits CRUDs) | Plus élevée (nécessite des interfaces et mappers) |