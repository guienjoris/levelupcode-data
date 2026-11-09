# NestJS Débutant — Pratique Quotidienne : Validation (DTO), Exceptions & Configuration

Construire une API robuste avec NestJS ne se limite pas à relier des contrôleurs et des services. Dès les premiers endpoints, trois impératifs se posent : valider rigoureusement les données entrantes, renvoyer des erreurs HTTP explicites et standardisées, et externaliser les secrets d'environnement.

---

## 1. Validation Déclarative avec DTO et `ValidationPipe`

Dans une architecture backend, un **DTO** (*Data Transfer Object*) définit le contrat de données qu'un client doit envoyer dans le corps d'une requête (`@Body()`).

Plutôt que d'écrire des vérifications manuelles (`if (!body.email) ...`), NestJS s'appuie sur `class-validator` et `class-transformer` combinés à un **Pipe** de validation global.

### 1.1 Activation globale du `ValidationPipe`

Dans le point d'entrée `main.ts`, activez le validateur avec les options de production recommandées :

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,            // Supprime automatiquement les champs non déclarés dans le DTO
      forbidNonWhitelisted: true, // Lève une erreur 400 si un champ inconnu est envoyé
      transform: true,            // Convertit automatiquement les payloads selon les types TS du DTO
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

### 1.2 Définition du DTO avec règles de validation

```typescript
// dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, IsInt, Min, Max, IsOptional } from 'class-validator';
import { Type } from 'class-transformer';

export class CreateUserDto {
  @IsString({ message: 'Le nom doit être une chaîne de caractères' })
  @MinLength(2, { message: 'Le nom doit comporter au moins 2 caractères' })
  name: string;

  @IsEmail({}, { message: 'Adresse email non valide' })
  email: string;

  @IsOptional()
  @Type(() => Number) // Convertit "25" (query/string) en number
  @IsInt({ message: "L'âge doit être un entier" })
  @Min(18, { message: 'Vous devez avoir au moins 18 ans' })
  @Max(120)
  age?: number;
}
```

### 1.3 Utilisation dans le Contrôleur

Grâce au flag `transform: true`, le contrôleur reçoit une véritable instance validée de `CreateUserDto` :

```typescript
// users.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  @Post()
  createUser(@Body() createUserDto: CreateUserDto) {
    // Si la requête arrive ici, les données sont 100% valides et nettoyées
    return { status: 'success', data: createUserDto };
  }
}
```

---

## 2. Gestion Standardisée des Erreurs : Exceptions HTTP Natives

Ne retournez jamais de simples objets `{ success: false, error: "..." }` avec un code HTTP 200 en cas de problème. NestJS fournit une hiérarchie d'exceptions prêtes à l'emploi qui héritent de `HttpException`.

### 2.1 Les Exceptions courantes

| Exception NestJS | Code HTTP généré | Contexte d'usage typique |
| :--- | :--- | :--- |
| `BadRequestException` | **400** | Paramètre invalide ou règle métier de base violée. |
| `UnauthorizedException` | **401** | Jeton JWT absent, expiré ou invalide. |
| `ForbiddenException` | **403** | Utilisateur authentifié mais rôle insuffisant (ex: non-admin). |
| `NotFoundException` | **404** | Ressource demandée inexistante en base. |
| `ConflictException` | **409** | Doublon (ex: email déjà existant à l'inscription). |
| `InternalServerErrorException`| **500** | Panne de base de données ou erreur non interceptée. |

### 2.2 Exemple dans un Service Métier

```typescript
// users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';

@Injectable()
export class UsersService {
  private users = [{ id: 1, email: 'alice@domain.com', name: 'Alice' }];

  findById(id: number) {
    const user = this.users.find(u => u.id === id);
    if (!user) {
      // NestJS génère automatiquement un JSON formatté :
      // { "statusCode": 404, "message": "Utilisateur introuvable", "error": "Not Found" }
      throw new NotFoundException(`Utilisateur avec l'ID ${id} introuvable`);
    }
    return user;
  }

  create(email: string, name: string) {
    const exists = this.users.some(u => u.email === email);
    if (exists) {
      throw new ConflictException(`L'adresse ${email} est déjà enregistrée`);
    }
    const newUser = { id: Date.now(), email, name };
    this.users.push(newUser);
    return newUser;
  }
}
```

---

## 3. Gestion Sécurisée des Variables avec `@nestjs/config`

Lire directement `process.env.DB_PASSWORD` dans vos services crée du couplage fort et expose à des crashs silencieux si une variable est omise au démarrage.

NestJS fournit le module officiel `@nestjs/config` (basé sur `dotenv`).

### 3.1 Initialisation dans `AppModule`

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,               // Rend ConfigService accessible partout sans ré-importer ConfigModule
      envFilePath: ['.env.local', '.env'], // Ordre de priorité des fichiers .env
    }),
  ],
})
export class AppModule {}
```

### 3.2 Injection de `ConfigService` dans vos classes

```typescript
// database.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class DatabaseService {
  constructor(private readonly configService: ConfigService) {}

  getConnectionString(): string {
    // Lecture typée avec valeur de secours optionnelle
    const host = this.configService.get<string>('DB_HOST', 'localhost');
    const port = this.configService.get<number>('DB_PORT', 5432);
    return `postgres://${host}:${port}`;
  }
}
```

---

## 4. Décorateurs de Paramètres Personnalisés (*Custom Param Decorators*)

NestJS permet d'extraire de la logique répétitive des contrôleurs (comme récupérer l'utilisateur connecté injecté par un middleware ou lire un header spécifique) dans un décorateur dédié via `createParamDecorator`.

### 4.1 Création du décorateur `@CurrentUser`

```typescript
// common/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    // Passage du contexte générique au contexte HTTP spécifique
    const request = ctx.switchToHttp().getRequest();
    const user = request.user; // Habituellement injecté par un Guard d'authentification

    // Si une clé précise est demandée : @CurrentUser('email')
    return data ? user?.[data] : user;
  },
);
```

### 4.2 Utilisation élégante dans un Contrôleur

```typescript
// profile.controller.ts
import { Controller, Get } from '@nestjs/common';
import { CurrentUser } from './common/decorators/current-user.decorator';

@Controller('profile')
export class ProfileController {
  @Get('me')
  getProfile(@CurrentUser() user: any) {
    return { profile: user };
  }

  @Get('email')
  getEmail(@CurrentUser('email') email: string) {
    return { email };
  }
}
```

---

## Synthèse des Bonnes Pratiques

1. **Activez `whitelist` et `forbidNonWhitelisted` :** Cela protège immédiatement votre API contre les injections de propriétés malveillantes (*Mass Assignment Vulnerability*).
2. **Utilisez toujours les exceptions HTTP natives :** N'envoyez pas de codes 200 contenant des messages d'erreur.
3. **Passez par `ConfigService` :** Ne lisez jamais `process.env` directement dans vos services ou contrôleurs.
4. **Simplifiez vos contrôleurs avec des décorateurs custom :** Dès qu'une manipulation de l'objet `req` se répète sur 2 contrôleurs ou plus, encapsulez-la dans un `createParamDecorator`.