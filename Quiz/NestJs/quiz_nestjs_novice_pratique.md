# Quiz d'Évaluation — NestJS (Niveau Novice : DTO, Exceptions & Config)

Ce quiz évalue votre compréhension des pratiques indispensables pour concevoir des APIs avec NestJS : l'application des DTOs avec `ValidationPipe`, le choix des exceptions HTTP natives, l'usage de `@nestjs/config` et la création de décorateurs de paramètres personnalisés.

### Question 1 : Protection des payloads avec le `ValidationPipe`

Dans la configuration globale de `ValidationPipe`, quel est le rôle exact de la combinaison des options `{ whitelist: true, forbidNonWhitelisted: true }` ?

* [ ] A) Elle rejette immédiatement la requête avec une erreur 400 si le client transmet des champs inconnus non prévus dans le DTO.
* [ ] B) Elle chiffre les données sensibles du corps de requête avant de les transférer au service de base de données.
* [ ] C) Elle convertit automatiquement toutes les propriétés textuelles en majuscules pour normaliser les entrées clients.
* [ ] D) Elle restreint l'accès aux endpoints aux seules adresses IP configurées dans la liste blanche du pare-feu applicatif.

### Question 2 : Exceptions HTTP intégrées

Un utilisateur tente de créer un compte avec une adresse email qui existe déjà dans la base de données. Quelle exception native de NestJS est la plus adaptée pour signaler cette situation conformément aux standards HTTP ?

* [ ] A) `BadRequestException`
* [ ] B) `ConflictException`
* [ ] C) `ForbiddenException`
* [ ] D) `NotFoundException`

### Question 3 : Transformation de types avec `ValidationPipe`

Par défaut, les paramètres passés dans l'URL (via `@Query()` ou `@Param()`) sont reçus sous forme de chaînes de caractères (`string`). Quelle option du `ValidationPipe` permet à NestJS d'instancier le DTO et de convertir automatiquement ces valeurs selon leurs types TypeScript déclarés ?

* [ ] A) `enableImplicitConversion: false`
* [ ] B) `validateCustomDecorators: true`
* [ ] C) `transform: true`
* [ ] D) `strictGroups: true`

### Question 4 : Gestion globale de la configuration

Pourquoi est-il courant de configurer `ConfigModule.forRoot({ isGlobal: true })` à la racine de l'application dans `AppModule` ?

* [ ] A) Pour permettre à `ConfigService` d'être injecté dans n'importe quel service sans devoir réimporter `ConfigModule` partout.
* [ ] B) Pour publier automatiquement toutes les variables d'environnement sur un point de terminaison HTTP public d'administration.
* [ ] C) Pour interdire au système d'exploitation de modifier les fichiers `.env` pendant toute la durée d'exécution du serveur.
* [ ] D) Pour forcer le rechargement synchrone de la base de données à chaque modification d'une variable dans le fichier `.env`.

### Question 5 : Rôle des décorateurs de paramètres personnalisés

Quel est l'intérêt principal de concevoir un décorateur avec `createParamDecorator` (comme `@CurrentUser()`) au lieu de manipuler directement l'objet `@Req()` dans un contrôleur ?

* [ ] A) Accélérer le temps d'exécution des requêtes HTTP en court-circuitant le passage par les intercepteurs NestJS.
* [ ] B) Empêcher le moteur V8 de désallouer la mémoire associée à la session utilisateur lors des montées en charge.
* [ ] C) Découpler le contrôleur de l'objet de requête bas niveau (`req`) pour simplifier le code et faciliter les tests unitaires.
* [ ] D) Forcer le navigateur web client à rafraîchir son jeton d'authentification avant de lire le corps de la réponse.

### Question 6 : Différence entre `ForbiddenException` et `UnauthorizedException`

Quelle nuance fondamentale sépare l'usage de `UnauthorizedException` (401) de celui de `ForbiddenException` (403) dans une route protégée ?

* [ ] A) 401 indique une absence ou invalidité d'authentification, tandis que 403 signifie que l'utilisateur identifié n'a pas les droits requis.
* [ ] B) 401 concerne exclusivement les requêtes de lecture (`GET`), alors que 403 s'applique uniquement aux écritures (`POST`/`PUT`).
* [ ] C) 401 signale une surcharge temporaire du serveur hôte, alors que 403 indique un problème de syntaxe dans le document JSON reçu.
* [ ] D) 401 est levée par les filtres d'exception, tandis que 403 ne peut être déclenchée que par le middleware de routage Express.

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** `whitelist: true` filtre les propriétés pour ne conserver que celles munies de décorateurs dans le DTO. En lui ajoutant `forbidNonWhitelisted: true`, NestJS arrête le traitement et renvoie une réponse HTTP 400 (*Bad Request*) détaillée si des propriétés parasites sont envoyées, protégeant l'API contre l'assignation de masse (*Mass Assignment*).

### 2. Réponse : **B**
**Explication :** Le code HTTP 409 (*Conflict*) est le code standard à utiliser lorsqu'une requête ne peut être exécutée en raison d'un conflit avec l'état actuel de la ressource cible (par exemple une violation de contrainte d'unicité sur un email ou un nom d'utilisateur). NestJS fournit pour cela `ConflictException`.

### 3. Réponse : **C**
**Explication :** L'option `transform: true` indique au `ValidationPipe` d'appliquer `class-transformer` sur la charge utile entrante afin d'obtenir de vraies instances de classes typées selon le DTO, plutôt que de simples objets littéraux JavaScript non convertis.

### 4. Réponse : **A**
**Explication :** En définissant `isGlobal: true` sur le `ConfigModule`, ses fournisseurs (notamment `ConfigService`) deviennent accessibles dans tous les modules de l'application sans qu'il soit nécessaire d'ajouter `imports: [ConfigModule]` dans chaque module enfant.

### 5. Réponse : **C**
**Explication :** Injecter directement `@Req()` lie inutilement le contrôleur à l'objet de requête sous-jacent (Express ou Fastify), ce qui complexifie les tests unitaires (nécessité de mocker un gros objet `req`) et alourdit la signature des méthodes. Un custom decorator extrait proprement la donnée requise (ex. `req.user`) de façon réutilisable.

### 6. Réponse : **A**
**Explication :** L'erreur 401 (*Unauthorized*) sanctionne un défaut d'authentification (jeton manquant, malformé ou expiré) : le serveur ne sait pas qui vous êtes. L'erreur 403 (*Forbidden*) intervient lorsque le serveur a bien authentifié le client, mais que ce dernier ne possède pas les privilèges ou rôles suffisants pour accéder à la ressource.