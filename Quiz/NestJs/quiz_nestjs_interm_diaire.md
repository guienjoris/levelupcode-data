# Quiz d'Évaluation — NestJS Niveau Intermédiaire

Ce quiz évalue votre compréhension du cycle de vie des requêtes, des Pipes, des Exception Filters, des Guards et des Intercepteurs présentés dans le cours.

---

### Question 1 : Cycle de vie des requêtes
Dans quel ordre chronologique ces briques NestJS s'exécutent-elles lorsqu'une requête entrante arrive sur l'application ?

- [ ] A) Pipes $\rightarrow$ Guards $\rightarrow$ Middlewares $\rightarrow$ Interceptors
- [ ] B) Middlewares $\rightarrow$ Guards $\rightarrow$ Interceptors $\rightarrow$ Pipes
- [ ] C) Guards $\rightarrow$ Middlewares $\rightarrow$ Pipes $\rightarrow$ Interceptors
- [ ] D) Middlewares $\rightarrow$ Interceptors $\rightarrow$ Pipes $\rightarrow$ Guards

---

### Question 2 : ValidationPipe & Sécurité
Quelle option du `ValidationPipe` permet de rejeter la requête avec une erreur HTTP 400 si le client envoie des propriétés non déclarées dans le DTO ?

- [ ] A) `whitelist: true`
- [ ] B) `transform: true`
- [ ] C) `forbidNonWhitelisted: true`
- [ ] D) `strictValidation: true`

---

### Question 3 : Exception Filters
Considérez le décorateur `@Catch()` sans aucun argument passé au-dessus d'une classe implémentant `ExceptionFilter` :

```typescript
@Catch()
export class CustomFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) { /* ... */ }
}
```

Quel est son comportement exact ?

- [ ] A) Il n'attrape aucune exception car aucun type n'est spécifié.
- [ ] B) Il attrape uniquement les exceptions de type `HttpException`.
- [ ] C) Il intercepte toutes les exceptions non gérées, qu'elles soient de type HTTP ou non (ex: erreurs internes, plantages DB).
- [ ] D) Il lève une erreur de compilation TypeScript.

---

### Question 4 : Guards & Reflector
Dans l'implémentation d'un Guard RBAC, quelle méthode du `Reflector` est généralement privilégiée pour inspecter à la fois la méthode (gestionnaire de route) et la classe parente tout en permettant à la méthode d'écraser la métadonnée de classe ?

- [ ] A) `reflector.get<Role[]>(ROLES_KEY, context.getHandler())`
- [ ] B) `reflector.getAllAndOverride<Role[]>(ROLES_KEY, [context.getHandler(), context.getClass()])`
- [ ] C) `reflector.merge<Role[]>(ROLES_KEY, [context.getHandler(), context.getClass()])`
- [ ] D) `reflector.extractMetadata(ROLES_KEY)`

---

### Question 5 : Décorateurs de paramètres personnalisés
Dans la fonction `createParamDecorator((data, ctx) => ...)`, quel objet de bas niveau permet de basculer vers le contexte HTTP afin d'extraire l'objet `request` ?

- [ ] A) `ctx.getHttp()`
- [ ] B) `ctx.switchToHttp()`
- [ ] C) `ctx.getExecutionContext()`
- [ ] D) `ctx.getPayload()`

---

### Question 6 : Intercepteurs & RxJS
Quelle instruction est indispensable au sein de la méthode `intercept()` d'un intercepteur NestJS pour déclencher l'exécution du contrôleur et récupérer la réponse sous forme d'Observable ?

- [ ] A) `context.proceed()`
- [ ] B) `next.execute()`
- [ ] C) `next.handle()`
- [ ] D) `return true`

---

### Question 7 : Comportement de `transform: true`
Lorsque `transform: true` est activé dans le `ValidationPipe` global, que se passe-t-il lors de la réception d'un corps de requête JSON ?

- [ ] A) Le corps de la requête reste un simple objet JavaScript littéral (`Object`).
- [ ] B) Le corps de la requête est instancié avec la classe du DTO via `class-transformer`, exécutant d'éventuels constructeurs ou méthodes associées.
- [ ] C) Toutes les valeurs de chaînes sont converties en minuscules.
- [ ] D) Le JSON est chiffré automatiquement.

---

## Corrigé et Explications

<details>
<summary>👉 Cliquez ici pour voir les réponses détaillées</summary>

### 1. Réponse : **B**
**Explication :** L'ordre exact est : **Middlewares** $\rightarrow$ **Guards** $\rightarrow$ **Interceptors (avant le handler)** $\rightarrow$ **Pipes** $\rightarrow$ Contrôleur. Les Pipes s'exécutent juste avant la méthode du contrôleur pour valider et transformer les arguments déjà autorisés par les guards.

### 2. Réponse : **C**
**Explication :** L'option `whitelist: true` supprime silencieusement les champs non déclarés. Pour interdire formellement et lever une exception `BadRequestException` lorsqu'un champ inattendu est présent, il faut activer `forbidNonWhitelisted: true`.

### 3. Réponse : **C**
**Explication :** Laisser `@Catch()` vide en fait un filtre attrape-tout (*catch-all*). Il intercepte n'importe quelle erreur levée dans l'application, qu'il s'agisse d'une `HttpException` intentionnelle ou d'une `Error` non gérée (TypeError, erreur SQL, etc.).

### 4. Réponse : **B**
**Explication :** `reflector.getAllAndOverride()` inspecte le tableau de cibles dans l'ordre fourni et renvoie la première valeur trouvée. Ainsi, une métadonnée appliquée sur la méthode prend le pas sur celle de la classe.

### 5. Réponse : **B**
**Explication :** L'`ExecutionContext` de NestJS est multi-protocole (HTTP, WebSockets, Microservices). Pour accéder aux objets spécifiques à Express/Fastify, on utilise `ctx.switchToHttp().getRequest()`.

### 6. Réponse : **C**
**Explication :** `next.handle()` appelle le gestionnaire de route suivant dans la chaîne et retourne un flux RxJS `Observable`. Sans cet appel, la méthode du contrôleur n'est jamais exécutée.

### 7. Réponse : **B**
**Explication :** Par défaut, Express reçoit des objets bruts sans type concret. L'option `transform: true` utilise `plainToInstance` de `class-transformer` pour convertir le JSON brut en véritable instance de la classe DTO typée.
</details>