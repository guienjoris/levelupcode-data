# Masterclass NestJS — Niveau Senior & Staff Engineer

Cette leçon aborde NestJS sous l'angle de l'ingénierie avancée, de l'architecture distribuée et des performances de production. À ce niveau, la maîtrise des contrôleurs et services ne suffit plus : il s'agit de concevoir des systèmes résilients, extensibles et optimisés au niveau mémoire et CPU.

---

## Sommaire
1. [Mécanique interne du conteneur IoC & Scopes d'injection](#1-mécanique-interne-du-conteneur-ioc--scopes-dinjection)
2. [Conception de Modules Configurables Avancés (`ConfigurableModuleBuilder`)](#2-conception-de-modules-configurables-avancés)
3. [Architecture DDD & Clean Architecture avec `@nestjs/cqrs`](#3-architecture-ddd--clean-architecture-avec-nestjscqrs)
4. [Gestion de la Concurrence et Transactions Distribuées (Unit of Work)](#4-gestion-de-la-concurrence-et-transactions-distribuées)
5. [Performance, Cycle de vie et Graceful Shutdown en Haute Disponibilité](#5-performance-cycle-de-vie-et-graceful-shutdown)

---

## 1. Mécanique interne du conteneur IoC & Scopes d'injection

Le conteneur IoC de NestJS résout les dépendances au démarrage sous forme d'un graphe orienté acyclique (DAG). Comprendre les **Scopes d'injection** est crucial pour éviter les chutes drastiques de performance et les fuites de mémoire.

### 1.1 Comparatif des Scopes

| Scope | Cycle de vie | Empreinte mémoire | Utilisation |
| :--- | :--- | :--- | :--- |
| `DEFAULT` (Singleton) | Instancié 1 seule fois au bootstrap | Optimale ($O(1)$) | Recommandé pour 99% des providers |
| `REQUEST` | Nouvelle instance à chaque requête HTTP | Coûteuse ($O(N)$ créations/sec) | Multi-tenancy strict, propagation de contexte |
| `TRANSIENT` | Nouvelle instance par injection | Variable | Objets d'état non partagés |

### 1.2 Le piège de la propagation en cascade du `REQUEST` Scope
Lorsqu'un service est déclaré en `Scope.REQUEST`, **tout le sous-arbre de dépendances qui en dépend devient automatiquement scopé à la requête**.

```typescript
// ⚠️ ANTI-PATTERN : Dégrader un service transversal en Request Scope
@Injectable({ scope: Scope.REQUEST })
export class TenantContextService {
  constructor(@Inject(REQUEST) private readonly request: Request) {}
  
  getTenantId(): string {
    return this.request.headers['x-tenant-id'] as string;
  }
}
```

*Conséquence :* Si votre `DatabaseService` ou votre `UserRepository` injecte ce service, chaque requête HTTP réinstancie l'intégralité du pipeline, entraînant une surcharge importante pour le Garbage Collector de V8.

### 1.3 Solution Senior : `AsyncLocalStorage` (Node.js natif)
Pour propager un contexte de requête sans dégrader les singletons, utilisez l'API native `AsyncLocalStorage`.

```typescript
// src/common/context/als.context.ts
import { AsyncLocalStorage } from 'node:async_hooks';

export interface RequestStore {
  traceId: string;
  tenantId: string;
  userId?: string;
}

export const requestAls = new AsyncLocalStorage<RequestStore>();
```

```typescript
// src/common/middlewares/context.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { randomUUID } from 'node:crypto';
import { requestAls } from '../context/als.context';

@Injectable()
export class ContextMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const store = {
      traceId: (req.headers['x-request-id'] as string) || randomUUID(),
      tenantId: (req.headers['x-tenant-id'] as string) || 'default',
    };

    // Exécute le reste du pipeline dans le contexte isolé de la requête
    requestAls.run(store, () => next());
  }
}
```

```typescript
// Provider 100% SINGLETON accédant au contexte isolé
@Injectable() // Scope.DEFAULT
export class OrderService {
  async createOrder() {
    const store = requestAls.getStore();
    const currentTenant = store?.tenantId; // Accès sans réinstanciation !
  }
}
```

---

## 2. Conception de Modules Configurables Avancés

Dans une architecture multi-paquets ou monorepo (Nx/Turborepo), la création de modules dynamiques (`forRootAsync`) est souvent verbeuse. Depuis NestJS v9+, `ConfigurableModuleBuilder` simplifie l'implémentation de la factory pattern asynchrone conforme aux standards du framework.

```typescript
// src/integrations/mailer/mailer.module-definition.ts
import { ConfigurableModuleBuilder } from '@nestjs/common';

export interface MailerModuleOptions {
  apiKey: string;
  senderDomain: string;
  maxRetries?: number;
}

export const {
  ConfigurableModuleClass,
  MODULE_OPTIONS_TOKEN,
  OPTIONS_TYPE,
  ASYNC_OPTIONS_TYPE,
} = new ConfigurableModuleBuilder<MailerModuleOptions>()
  .setClassPrefix('Mailer')
  .setExtras(
    { isGlobal: false },
    (definition, extras) => ({
      ...definition,
      global: extras.isGlobal,
    }),
  )
  .build();
```

```typescript
// src/integrations/mailer/mailer.module.ts
import { Module } from '@nestjs/common';
import { ConfigurableModuleClass } from './mailer.module-definition';
import { MailerService } from './mailer.service';

@Module({
  providers: [MailerService],
  exports: [MailerService],
})
export class MailerModule extends ConfigurableModuleClass {}
```

```typescript
// src/integrations/mailer/mailer.service.ts
import { Inject, Injectable } from '@nestjs/common';
import { MODULE_OPTIONS_TOKEN, MailerModuleOptions } from './mailer.module-definition';

@Injectable()
export class MailerService {
  constructor(
    @Inject(MODULE_OPTIONS_TOKEN) private readonly options: MailerModuleOptions,
  ) {}

  async sendMail(to: string, content: string): Promise<void> {
    // Implémentation basée sur this.options.apiKey
  }
}
```

Consommation propre avec support de l'injection dynamique :

```typescript
@Module({
  imports: [
    MailerModule.registerAsync({
      isGlobal: true,
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        apiKey: config.getOrThrow<string>('SENDGRID_API_KEY'),
        senderDomain: 'acme.org',
      }),
    }),
  ],
})
export class AppModule {}
```

---

## 3. Architecture DDD & Clean Architecture avec `@nestjs/cqrs`

Pour les applications à forte logique métier, le pattern standard MVC/CRUD devient un goulet d'étranglement. L'adoption du **CQRS (Command Query Responsibility Segregation)** et de l'Event Sourcing isole les mutations des requêtes de lecture.

### 3.1 Découplage Commande et Handler Métier

```typescript
// src/orders/commands/impl/create-order.command.ts
import { ICommand } from '@nestjs/cqrs';

export class CreateOrderCommand implements ICommand {
  constructor(
    public readonly customerId: string,
    public readonly items: Array<{ sku: string; quantity: number }>,
    public readonly idempotencyKey: string,
  ) {}
}
```

```typescript
// src/orders/commands/handlers/create-order.handler.ts
import { CommandHandler, ICommandHandler, EventPublisher } from '@nestjs/cqrs';
import { CreateOrderCommand } from '../impl/create-order.command';
import { OrderAggregate } from '../../domain/order.aggregate';
import { OrderRepository } from '../../infrastructure/order.repository';

@CommandHandler(CreateOrderCommand)
export class CreateOrderHandler implements ICommandHandler<CreateOrderCommand> {
  constructor(
    private readonly repository: OrderRepository,
    private readonly publisher: EventPublisher,
  ) {}

  async execute(command: CreateOrderCommand): Promise<string> {
    const { customerId, items, idempotencyKey } = command;

    // 1. Instanciation du modèle de domaine
    const rawOrder = OrderAggregate.create(customerId, items, idempotencyKey);
    
    // 2. Connexion du modèle au bus d'événements NestJS
    const order = this.publisher.mergeObjectContext(rawOrder);

    // 3. Persistance
    await this.repository.save(order);

    // 4. Déclenchement de tous les Domain Events enregistrés
    order.commit();

    return order.id;
  }
}
```

---

## 4. Gestion de la Concurrence et Transactions Distribuées

L'un des plus grands défis d'ingénierie réside dans l'intégrité transactionnelle tout en préservant le principe d'inversion des dépendances (DIP). Le pattern **Unit of Work (UoW)** abstrait la session transactionnelle sans fuiter l'ORM (`PrismaClient` ou `QueryRunner` TypeORM) dans les services métier.

```typescript
// src/common/database/unit-of-work.interface.ts
export interface IUnitOfWork {
  executeInTransaction<T>(work: () => Promise<T>): Promise<T>;
}
```

```typescript
// Implémentation basée sur le Contexte ALS et TypeORM / Prisma
@Injectable()
export class PrismaUnitOfWork implements IUnitOfWork {
  constructor(private readonly prisma: PrismaService) {}

  async executeInTransaction<T>(work: () => Promise<T>): Promise<T> {
    return this.prisma.$transaction(async (tx) => {
      return transactionalAls.run({ txClient: tx }, async () => {
        return await work();
      });
    });
  }
}
```

---

## 5. Performance, Cycle de vie et Graceful Shutdown

Dans un environnement orchestré comme Kubernetes, une terminaison brutale sans interception des signaux POSIX entraîne des erreurs `502 Bad Gateway` et des pertes d'écritures en base.

### 5.1 Configuration du Graceful Shutdown

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { Logger } from '@nestjs/common';

async function bootstrap() {
  const logger = new Logger('Bootstrap');
  const app = await NestFactory.create(AppModule, {
    bufferLogs: true, // Évite la perte de logs avant l'initialisation du logger custom
  });

  // Active les hooks natifs : onModuleDestroy, beforeApplicationShutdown, onApplicationShutdown
  app.enableShutdownHooks();

  const port = process.env.PORT || 3000;
  await app.listen(port);
  logger.log(`Worker process running on PID: ${process.pid}`);
}
bootstrap();
```

### 5.2 Libération ordonnée des ressources

Implémentez l'interface `OnApplicationShutdown` pour clôturer proprement les pools de sockets et les connexions Kafka/RabbitMQ :

```typescript
@Injectable()
export class QueueConsumerService implements OnApplicationShutdown {
  private isTerminating = false;

  async onApplicationShutdown(signal?: string) {
    this.isTerminating = true;
    console.warn(`Signal reçu: ${signal}. Clôture du consommateur Kafka...`);
    
    // Attendre la fin du batch en cours avant de déconnecter
    await this.drainInFlightMessages();
    await this.kafkaClient.disconnect();
  }

  private async drainInFlightMessages(): Promise<void> {
    // Logique d'attente active ou sémaphore
  }
}
```

---

## Synthèse pour l'architecte NestJS

1. **Priorisez toujours le `Scope.DEFAULT`** : utilisez `AsyncLocalStorage` pour les données par requête.
2. **Standardisez les modules partagés** avec `ConfigurableModuleBuilder` pour offrir une API uniforme et typée.
3. **Isolez le domaine métier** de l'infrastructure via le pattern CQRS et l'inversion de dépendance.
4. **Activez impérativement `enableShutdownHooks()`** pour garantir l'absence de coupures réseau en production lors des déploiements sans interruption (*zero-downtime*).