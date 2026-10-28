# Masterclass NestJS — Niveau Intermédiaire

Bienvenue dans cette leçon intermédiaire dédiée à **NestJS**. Vous maîtrisez déjà les bases : création de modules, injection de dépendances simple, contrôleurs et services CRUD. 

Ce cours se concentre sur les concepts indispensables pour bâtir des applications modulaires, robustes et prêtes pour la production :
1. Le cycle de vie des requêtes (*Request Lifecycle*)
2. Validation et transformation via les **Pipes** (`class-validator` / `class-transformer`)
3. Gestion d'erreurs avancée avec les **Exception Filters**
4. Sécurité et contrôle d'accès avec les **Guards** et **Custom Decorators**
5. Traitement transversal avec les **Interceptors**
6. Configuration dynamique et variables d'environnement (`@nestjs/config`)

---

## 1. Comprendre le cycle de vie d'une requête (*Request Lifecycle*)

Dans NestJS, chaque requête HTTP traverse une séquence ordonnée d'étapes avant d'atteindre le contrôleur, puis sur le chemin du retour :

```
Requête HTTP
   │
   ▼
[ 1. Middlewares ]
   │
   ▼
[ 2. Guards ] ──────────────── (Rejet : 403 Forbidden / 401 Unauthorized)
   │
   ▼
[ 3. Interceptors (Pre-controller) ]
   │
   ▼
[ 4. Pipes ] ───────────────── (Rejet : 400 Bad Request)
   │
   ▼
[ 5. Controller (Handler) ]
   │
   ▼
[ 6. Service / Business Logic ]
   │
   ▼
[ 7. Interceptors (Post-controller / Transformation de la réponse) ]
   │
   ▼
[ 8. Exception Filters ] ───── (Capturé si une exception a été levée)
   │
   ▼
Réponse HTTP envoyée au client
```

---

## 2. Validation & Transformation des données : Les DTOs et Pipes

NestJS s'appuie sur `ValidationPipe` couplé aux bibliothèques `class-validator` et `class-transformer`.

### 2.1 Configuration globale

Installez les dépendances requises :
```bash
npm i --save class-validator class-transformer
```

Activez le pipe globalement dans `main.ts` avec des options strictes de sécurité :

```typescript
// src/main.ts
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true, // Supprime automatiquement les champs non définis dans le DTO
      forbidNonWhitelisted: true, // Renvoie une erreur si des champs inattendus sont présents
      transform: true, // Transforme automatiquement les payloads en instances de classes DTO et convertit les types primitifs
      transformOptions: {
        enableImplicitConversion: true,
      },
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

### 2.2 Définition d'un DTO typé et validé

```typescript
// src/articles/dto/create-article.dto.ts
import { IsString, IsNotEmpty, MinLength, IsEnum, IsOptional, IsArray } from 'class-validator';

export enum ArticleStatus {
  DRAFT = 'DRAFT',
  PUBLISHED = 'PUBLISHED',
  ARCHIVED = 'ARCHIVED',
}

export class CreateArticleDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(5, { message: 'Le titre doit comporter au moins 5 caractères.' })
  title: string;

  @IsString()
  @IsNotEmpty()
  content: string;

  @IsEnum(ArticleStatus, { message: 'Statut invalide.' })
  @IsOptional()
  status?: ArticleStatus = ArticleStatus.DRAFT;

  @IsArray()
  @IsString({ each: true })
  @IsOptional()
  tags?: string[];
}
```

---

## 3. Gestion centralisée des erreurs : Exception Filters

Par défaut, NestJS fournit un `HttpExceptionFilter` qui capture les exceptions `HttpException`. Pour uniformiser le format de réponse de votre API (ex: format RFC 7807 ou format d'équipe), vous pouvez créer un filtre personnalisé.

```typescript
// src/common/filters/http-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
  Logger,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch() // Attrape toutes les exceptions (HTTP ou non)
export class GlobalHttpExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(GlobalHttpExceptionFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const exceptionResponse =
      exception instanceof HttpException ? exception.getResponse() : null;

    const message =
      typeof exceptionResponse === 'object' && exceptionResponse !== null
        ? (exceptionResponse as any).message || exceptionResponse
        : (exception as any)?.message || 'Internal server error';

    this.logger.error(
      `HTTP Status: ${status} Error Message: ${JSON.stringify(message)} Path: ${request.url}`,
    );

    response.status(status).json({
      success: false,
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      error: message,
    });
  }
}
```

---

## 4. Sécurité et contrôle d'accès : Guards & Custom Decorators

Les **Guards** déterminent si une requête a le droit d'être traitée par le gestionnaire de route (authentification, vérification des rôles, permissions RBAC).

### 4.1 Métadonnées et Décorateur de Rôle

```typescript
// src/auth/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export enum Role {
  USER = 'user',
  ADMIN = 'admin',
}

export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);
```

### 4.2 Implémentation du Guard RBAC (Role-Based Access Control)

Le guard utilise `Reflector` pour inspecter les métadonnées de la méthode ou de la classe de contrôleur :

```typescript
// src/auth/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/core';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY, Role } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true; // Pas de rôle restreint pour cette route
    }

    const { user } = context.switchToHttp().getRequest();
    if (!user || !user.roles) {
      return false;
    }

    return requiredRoles.some((role) => user.roles.includes(role));
  }
}
```

### 4.3 Décorateur personnalisé de paramètre (`@CurrentUser()`)

Évitez d'injecter l'objet brut `req` dans vos contrôleurs : extrayez directement l'utilisateur authentifié.

```typescript
// src/auth/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
```

---

## 5. Logique transverse : Les Intercepteurs

Les intercepteurs utilisent la puissance de **RxJS** pour modifier le flux de données entrant ou sortant.

### Cas d'usage : Transformation de la réponse & calcul du temps d'exécution

```typescript
// src/common/interceptors/transform.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map, tap } from 'rxjs/operators';

export interface ApiResponse<T> {
  data: T;
  meta: {
    durationMs: number;
    timestamp: string;
  };
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, ApiResponse<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<ApiResponse<T>> {
    const now = Date.now();

    return next.handle().pipe(
      map((data) => ({
        data,
        meta: {
          durationMs: Date.now() - now,
          timestamp: new Date().toISOString(),
        },
      })),
    );
  }
}
```

---

## 6. Mise en pratique : Contrôleur combiné

Voici comment s'assemblent tous ces composants dans un contrôleur propre :

```typescript
// src/articles/articles.controller.ts
import {
  Controller,
  Get,
  Post,
  Body,
  UseGuards,
  UseInterceptors,
} from '@nestjs/common';
import { CreateArticleDto } from './dto/create-article.dto';
import { RolesGuard } from '../auth/guards/roles.guard';
import { Roles, Role } from '../auth/decorators/roles.decorator';
import { CurrentUser } from '../auth/decorators/current-user.decorator';
import { TransformInterceptor } from '../common/interceptors/transform.interceptor';

@Controller('articles')
@UseInterceptors(TransformInterceptor)
export class ArticlesController {
  
  @Post()
  @UseGuards(RolesGuard)
  @Roles(Role.ADMIN)
  async create(
    @Body() createArticleDto: CreateArticleDto,
    @CurrentUser('id') userId: string,
  ) {
    return {
      message: 'Article créé avec succès',
      authorId: userId,
      payload: createArticleDto,
    };
  }

  @Get()
  async findAll() {
    return [
      { id: '1', title: 'Comprendre les Pipes NestJS' },
      { id: '2', title: 'Architecture Modulaire' },
    ];
  }
}
```

---

## Exercice pratique suggéré

1. **Créer un Custom Pipe :** Implémentez un pipe `ParseObjectIdPipe` qui vérifie si une chaîne passée dans `@Param('id')` est un identifiant valide (par exemple un format UUID v4 ou Mongo ObjectId), et renvoie une `BadRequestException` personnalisée sinon.
2. **Gestion de cache :** Créez un Intercepteur personnalisé de mise en cache en mémoire utilisant `rxjs/operators` pour mettre en cache les requêtes `GET` pendant 30 secondes.