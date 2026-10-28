# Quiz Technique — NestJS Niveau Senior & Staff Engineer

Ce quiz teste votre maîtrise de l'architecture avancée, de l'optimisation mémoire et des mécanismes internes de NestJS.

---

### Question 1 : Propagation des Scopes d'injection
Considérez l'arbre de dépendances suivant :

$$\text{OrdersController} \longrightarrow \text{OrdersService (DEFAULT)} \longrightarrow \text{RequestContextService (REQUEST)}$$

Quel est l'impact de l'injection d'un provider avec le scope `Scope.REQUEST` sur `OrdersService` et `OrdersController` ?

- [ ] A) Aucun impact : le scope `REQUEST` reste isolé à `RequestContextService`, les autres demeurent des singletons.
- [ ] B) `OrdersService` et `OrdersController` sont automatiquement promus en scope `REQUEST`, créant de nouvelles instances à chaque requête HTTP.
- [ ] C) Le runtime NestJS lève une `CircularDependencyException` lors de la résolution du DAG.
- [ ] D) Seul `OrdersService` devient `REQUEST`, tandis que le contrôleur conserve son instance unique.

---

### Question 2 : Fuites de mémoire et `Scope.TRANSIENT`
Dans quel cas précis un provider défini avec `Scope.TRANSIENT` peut-il provoquer une fuite mémoire progressive (Memory Leak) dans Node.js ?

- [ ] A) Lorsqu'il est injecté dans un contrôleur HTTP classique.
- [ ] B) Lorsqu'il implémente l'interface `OnModuleInit`.
- [ ] C) Lorsqu'il est injecté dynamiquement dans un provider singleton qui accumule des références sans les désallouer.
- [ ] D) Les providers transitoires sont collectés à chaque `tick` de la boucle d'événements, ils ne peuvent jamais causer de fuite mémoire.

---

### Question 3 : `ConfigurableModuleBuilder`
Depuis NestJS 9+, quelle interface/méthode est automatiquement générée par `new ConfigurableModuleBuilder<TOptions>()` pour permettre l'enregistrement asynchrone d'un module dynamique avec injection de dépendances ?

- [ ] A) `registerAsync(options: ModuleAsyncOptions)`
- [ ] B) `forRootAsync(options: ConfigurableModuleAsyncOptions)`
- [ ] C) `createAsyncModule(options: DynamicModuleOptions)`
- [ ] D) `provideDynamicConfig(options: ProviderAsyncFactory)`

---

### Question 4 : Architecture CQRS & Séparation des flux
Dans une architecture CQRS implémentée avec `@nestjs/cqrs`, quelle est la responsabilité fondamentale d'un `CommandHandler` par opposition à un `QueryHandler` ?

- [ ] A) Le `CommandHandler` traite les opérations de mutation d'état, peut émettre des événements de domaine, et ne doit pas retourner de projection complexe de données de lecture.
- [ ] B) Le `CommandHandler` s'exécute de façon asynchrone dans un worker thread, alors que le `QueryHandler` s'exécute sur le main thread.
- [ ] C) Le `CommandHandler` lit les données du cache Redis, tandis que le `QueryHandler` interroge directement la base de données relationnelle.
- [ ] D) Le `CommandHandler` valide le schéma des données d'entrée via des décorateurs, remplaçant ainsi les DTOs.

---

### Question 5 : Cycle de vie et Graceful Shutdown en cluster / Kubernetes
Lorsqu'un conteneur NestJS reçoit un signal `SIGTERM` dans un pod Kubernetes, quelle méthode doit impérativement être appelée au démarrage pour que NestJS exécute les hooks `onModuleDestroy`, `beforeApplicationShutdown` et `onApplicationShutdown` ?

- [ ] A) `app.listen(3000, { graceful: true })`
- [ ] B) `app.enableShutdownHooks()`
- [ ] C) `process.on('SIGTERM', () => app.close())` uniquement (aucune API NestJS n'existe pour cela)
- [ ] D) `app.useGlobalFilters(new ShutdownFilter())`

---

### Question 6 : Transactions distribuées et `Unit of Work`
Dans un système sous forte charge utilisant le pattern *Unit of Work* ou un gestionnaire de transactions contextuel, pourquoi est-il souvent déconseillé d'associer la transaction active au scope `REQUEST` natif de NestJS ?

- [ ] A) Parce que `Scope.REQUEST` empêche l'accès aux variables d'environnement.
- [ ] B) Parce que chaque requête instancie un nouvel arbre d'objets, ce qui génère une surcharge CPU/mémoire massive et ne fonctionne pas pour des traitements en tâche de fond (ex: microservices Kafka, jobs BullMQ).
- [ ] C) Parce que la transaction SQL ne peut pas être commitée dans un pipe ou un guard.
- [ ] D) Parce que le framework Express désactive le pooling de connexions SQL.

---

### Question 7 : Alternative moderne au Scope.REQUEST pour le tracking de contexte
Quelle fonctionnalité native du moteur Node.js (V8) permet de propager un contexte d'exécution (ex: ID de corrélation, transaction DB) à travers des chaînes asynchrones sans impacter les performances de l'arbre d'injection de dépendances ?

- [ ] A) `process.binding('async_context')`
- [ ] B) `AsyncLocalStorage` du module `node:async_hooks`
- [ ] C) `EventEmitter.setSharedContext()`
- [ ] D) `WorkerThreads.parentPort`

---

## Corrigé et Explications Détaillées

<details>
<summary>👉 Cliquez ici pour voir les réponses et justifications</summary>

### 1. Réponse : **B**
**Explication :** En NestJS, la nature d'un scope est contagieuse (*scope bubble effect*). Si un service injecté est en `Scope.REQUEST`, tous les consommateurs en amont qui en dépendent (ici `OrdersService`, puis le contrôleur `OrdersController`) sont obligatoirement promus en `Scope.REQUEST` pour pouvoir recevoir la nouvelle instance contextuelle.

### 2. Réponse : **C**
**Explication :** Un provider `TRANSIENT` n'est pas un singleton : chaque point d'injection reçoit une nouvelle instance distincte. Si un singleton ou une collection pérenne retient des références vers ces instances créées à la demande, le Garbage Collector ne peut jamais les libérer, entraînant une fuite de mémoire Heap.

### 3. Réponse : **B**
**Explication :** Le constructeur `ConfigurableModuleBuilder` génère les méthodes statiques standards (`register`, `registerAsync` ou `forRoot`, `forRootAsync` si configuré via `.setClassPrefix('...')`), en fournissant l'interface standardisée `ConfigurableModuleAsyncOptions`.

### 4. Réponse : **A**
**Explication :** Le principe fondamental de CQRS (Command Query Responsibility Segregation) dicte qu'une commande modifie un état et exécute les invariants du domaine sans servir de requête de consultation complexe, tandis qu'une requête (Query) extrait une projection sans effets de bord.

### 5. Réponse : **B**
**Explication :** NestJS n'écoute pas les signaux du système d'exploitation par défaut pour éviter des conflits de listeners. L'appel explicite à `app.enableShutdownHooks()` est indispensable pour intercepter `SIGTERM`/`SIGINT` et déclencher proprement la fermeture des pools DB et des connexions réseau.

### 6. Réponse : **B**
**Explication :** La propagation du `Scope.REQUEST` recrée l'ensemble du graphe de dépendances à chaque requête HTTP, provoquant une pression thermique sur le Garbage Collector. De plus, ce scope n'a aucun sens en dehors d'une requête HTTP classique (consommateurs RabbitMQ/Kafka, workers de queue, scripts CLI).

### 7. Réponse : **B**
**Explication :** L'`AsyncLocalStorage` (`node:async_hooks`) fournit un stockage local asynchrone comparable au *ThreadLocal* d'environnements multi-threadés, permettant d'accéder au contexte n'importe où dans la pile sans altérer le scope Singleton des services NestJS.
</details>