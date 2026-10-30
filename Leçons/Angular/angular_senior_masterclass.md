# Masterclass Angular — Niveau Senior : Internals, Performance & Architecture d'Entreprise

Atteindre le palier Senior sur Angular exige de dépasser l'utilisation déclarative des composants et des formulaires. Il s'agit de maîtriser **les rouages internes du framework** : le moteur de détection sans `Zone.js` (*Zoneless*), la mécanique interne de l'arbre d'injection de dépendances, le rendu direct sur le moteur View Engine/Ivy, et l'orchestration de topologies modulaires à grande échelle.

---

## 1. Détection de Changement Moderne : Vers le Zoneless

### 1.1 L'Héritage et le Coût de `Zone.js`
Historiquement, Angular s'appuie sur `Zone.js` pour patcher l'ensemble des APIs asynchrones du navigateur (`setTimeout`, `Promise`, `fetch`, `addEventListener`). Lorsqu'une tâche asynchrone se termine, `Zone.js` intercepte la sortie de boucle d'événements et notifie Angular via `onMicrotaskEmpty`, déclenchant un parcours descendant complet de l'arbre de composants (`ApplicationRef.tick()`).

**Inconvénients en production :**
- Surcharge de mémoire et bundle lourd (~100 Ko non compressé).
- Surcoût d'exécution : des centaines de ticks globaux déclenchés par des événements purement passifs (ex: `scroll`, `mousemove`, `setInterval`).
- Débogage complexe des piles d'appels asynchrones asymétriques.

### 1.2 L'Architecture Zoneless avec Signals
Le standard moderne permet de désactiver totalement `Zone.js` au profit d'une réactivité par graphe de dépendances (Signals) :

```typescript
// app.config.ts
import { ApplicationConfig, provideExperimentalZonelessChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideExperimentalZonelessChangeDetection(), // Élimine complètement Zone.js du bundle
    provideRouter(routes)
  ]
};
```

### 1.3 Comment fonctionne la notification sans Zone ?
Sans Zone, comment Angular sait-il qu'une vue doit être mise à jour ?
1. **Mutation de Signal :** Toute écriture dans un `WritableSignal` (`signal.set()` ou `signal.update()`) dont la valeur est consommée dans une fonction de template marque la vue correspondante comme "dirty" (`Consumer.markDirty()`).
2. **Scheduler d'application :** Le runtime planifie un microtask unique (`queueMicrotask`) pour rafraîchir uniquement les sous-arbres marqués, sans jamais parcourir l'ensemble des branches non impactées.

```
Modèle Zone.js classique :
[ Événement Async ] ──► [ Zone.js Intercepte ] ──► [ ApplicationRef.tick() Global ] (Tout l'arbre inspecté)

Modèle Zoneless (Signals) :
[ Signal.set() ] ──► [ Graphe de Réactivité ] ──► [ Marque la Vue ID ] ──► [ Rafraîchissement ciblé ]
```

---

## 2. Injection de Dépendances Avancée : Arbre Hiérarchique & Modifieurs

L'injecteur d'Angular n'est pas un registre plat, mais une structure en deux arbres parallèles :
1. **L'arbre des injecteurs d'éléments (*ElementInjector*) :** Créé implicitement pour chaque élément DOM portant un composant ou une directive.
2. **L'arbre des injecteurs d'environnement (*EnvironmentInjector*) :** Gère la racine (`root`), les routes chargées en différé (*lazy routes*) et la plateforme.

```
                          [ PlatformInjector ]
                                   │
                         [ EnvironmentInjector: root ]
                                   │
                     ┌─────────────┴─────────────┐
                     ▼                           ▼
          [ Child Env: AdminRoute ]     [ ElementInjector: AppRoot ]
                                                 │
                                                 ▼
                                        [ ElementInjector: ParentComponent ]
                                                 │
                                                 ▼
                                        [ ElementInjector: ChildComponent ]
```

### 2.1 Modifieurs de Résolution (`@Self`, `@SkipSelf`, `@Optional`, `@Host`)
Dans le code moderne avec `inject()`, ces modifieurs s'expriment sous forme de drapeaux d'options :

```typescript
import { Directive, inject, Host, Self, SkipSelf, Optional } from '@angular/core';
import { LoggerService } from './logger.service';
import { ControlContainer } from '@angular/forms';

@Directive({
  selector: '[appCustomField]',
  standalone: true
})
export class CustomFieldDirective {
  // 1. Cherche UNIQUEMENT sur l'élément hôte immédiat
  private selfLogger = inject(LoggerService, { self: true, optional: true });

  // 2. Ignore l'élément courant et démarre la recherche sur les ancêtres
  private parentLogger = inject(LoggerService, { skipSelf: true });

  // 3. S'arrête à la frontière de vue de l'hôte (ne remonte pas au-delà du composant hôte)
  private container = inject(ControlContainer, { host: true });
}
```

### 2.2 `providers` vs `viewProviders` : Cloisonnement de projection

La différence devient cruciale lors de l'utilisation de `<ng-content>` :

```typescript
@Component({
  selector: 'app-collapsible-panel',
  standalone: true,
  template: `
    <div class="header">{{ title }}</div>
    <!-- Les composants projetés ici N'ONT PAS ACCÈS aux viewProviders -->
    <ng-content></ng-content>
  `,
  providers: [PublicService],        // Accessible aux enfants ET aux composants projetés via <ng-content>
  viewProviders: [PrivateViewService] // Accessible UNIQUEMENT aux enfants déclarés dans le template interne
})
export class CollapsiblePanelComponent {
  title = 'Panneau Sécurisé';
}
```

---

## 3. Optimisation Bas Niveau : Manipulation DOM & Bypass de Détection

L'abstraction de template d'Angular présente un coût. Pour des visualisations de données massives (grilles de 10 000 cellules, visualisations de graphiques ou canvas temps réel), il faut contourner le cycle standard.

### 3.1 Sortir de la détection avec `NgZone.runOutsideAngular`

Lorsque vous devez écouter des événements fréquents comme le scroll ou le redimensionnement :

```typescript
import { Component, ElementRef, inject, NgZone, OnInit, DestroyRef } from '@angular/core';

@Component({
  selector: 'app-canvas-stream',
  standalone: true,
  template: `<div #viewport class="viewport"></div>`
})
export class CanvasStreamComponent implements OnInit {
  private ngZone = inject(NgZone);
  private destroyRef = inject(DestroyRef);
  private hostRef = inject(ElementRef<HTMLElement>);

  ngOnInit(): void {
    // ⚠️ On désengage Angular de l'écoute d'événement
    this.ngZone.runOutsideAngular(() => {
      const element = this.hostRef.nativeElement;
      const onMove = (event: MouseEvent) => {
        // Traitement direct haute fréquence sans aucun Change Detection Tick !
        element.style.transform = `translate3d(${event.clientX}px, ${event.clientY}px, 0)`;
      };

      window.addEventListener('mousemove', onMove);
      this.destroyRef.onDestroy(() => window.removeEventListener('mousemove', onMove));
    });
  }
}
```

### 3.2 Dynamic Template Instantiation : `ViewContainerRef` & `TemplateRef`
Créer des composants impérativement permet d'éviter l'overhead de l'analyse statique lors de l'affichage de fenêtres dynamiques ou de dashboards configurables :

```typescript
import { Component, ViewChild, ViewContainerRef, TemplateRef, inject, ComponentRef } from '@angular/core';
import { AlertModalComponent } from './alert-modal.component';

@Component({
  selector: 'app-dynamic-host',
  standalone: true,
  template: `<ng-container #outlet></ng-container>`
})
export class DynamicHostComponent {
  @ViewChild('outlet', { read: ViewContainerRef, static: true })
  private vcr!: ViewContainerRef;

  loadAlert(message: string): ComponentRef<AlertModalComponent> {
    this.vcr.clear(); // Détruit les vues précédentes et libère la mémoire
    
    // Instanciation directe sans FactoryResolver (déprécié depuis Ivy)
    const ref = this.vcr.createComponent(AlertModalComponent);
    
    // Transmission des inputs par Signals natifs
    ref.setInput('message', message);
    
    // Écoute de l'output sans fuite mémoire
    const sub = ref.instance.closed.subscribe(() => {
      ref.destroy();
      sub.unsubscribe();
    });

    return ref;
  }
}
```

---

## 4. Pipeline HTTP Fonctionnel Avancé : Intercepteurs & Concurrence

Depuis Angular 15+, les intercepteurs à base de classes (`HttpInterceptor`) sont remplacés par des pipelines fonctionnels légers combinables avec `withInterceptors`.

### 4.1 Intercepteur de Résilience avec Retry, Refresh Token & Concurrence Mutex

```typescript
import { HttpInterceptorFn, HttpRequest, HttpHandlerFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from './auth.service';
import { catchError, switchMap, throwError, retry, timer } from 'rxjs';

export const resilientAuthInterceptor: HttpInterceptorFn = (req: HttpRequest<unknown>, next: HttpHandlerFn) => {
  const authService = inject(AuthService);
  const token = authService.getAccessToken();

  let clonedReq = req;
  if (token) {
    clonedReq = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }

  return next(clonedReq).pipe(
    // 1. Retry progressif avec backoff exponentiel pour les pannes réseau transitoires
    retry({
      count: 2,
      delay: (error, retryCount) => (error.status === 503 ? timer(retryCount * 1000) : throwError(() => error))
    }),
    
    // 2. Gestion unifiée du refresh de token (401 Unauthorized)
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401 && !req.url.includes('/auth/refresh')) {
        return authService.refreshToken().pipe(
          switchMap(newToken => {
            const reAuthedRequest = req.clone({
              setHeaders: { Authorization: `Bearer ${newToken}` }
            });
            return next(reAuthedRequest);
          }),
          catchError(refreshErr => {
            authService.forceLogout();
            return throwError(() => refreshErr);
          })
        );
      }
      return throwError(() => error);
    })
  );
};
```

---

## 5. Architecture d'Entreprise : Monorepo & Micro-Frontends

À mesure que les applications grandissent, les monolithes Frontend atteignent leurs limites de build. L'approche d'architecture d'entreprise repose sur deux modèles :

### 5.1 Monorepo Structuré avec Nx : Découpage par Bibliothèque

Une application d'entreprise ne doit pas concentrer son code dans `src/app`. Elle s'organise en librairies strictes isolées :

```
apps/
  portal/                     <-- Application hôte minimale (Shell)
libs/
  sales/
    feature-checkout/         <-- Composants intelligents de pages
    ui-cart-summary/          <-- Composants de présentation purs (Dumb)
    data-access/              <-- Services, HTTP, Stores, Facades
    domain/                   <-- Entités pures, interfaces, business rules
  shared/
    ui-components/
    util-crypto/
```

**Règles de dépendances transversales :**
- `domain` ne peut dépendre de rien d'autre que du langage.
- `ui` ne peut dépendre que de `domain` et d'autres `ui` partagées.
- `feature` orchestre `data-access` et `ui`.
- Les frontières sont validées au commit par des linters d'architecture (`@nx/enforce-module-boundaries`).

### 5.2 Dynamic Module Federation (Micro-Frontends)
Pour permettre à des équipes distantes de déployer sans redéployer le shell hôte, on exploite le **Module Federation** (via Webpack ou Rsbuild/Vite) couplé au routeur Angular :

```typescript
import { Routes } from '@angular/router';
import { loadRemoteModule } from '@angular-architects/module-federation';

export const routes: Routes = [
  {
    path: 'billing',
    loadChildren: () =>
      loadRemoteModule({
        type: 'module',
        remoteEntry: 'https://billing.company.internal/remoteEntry.js',
        exposedModule: './Routes'
      }).then(m => m.BILLING_ROUTES)
  }
];
```

---

## 6. Synthèse des Exigences du Développeur Senior Angular

| Domaine | Pratique Débutant / Intermédiaire | Pratique Senior Confirmé |
| :--- | :--- | :--- |
| **Moteur Réactif** | Dépendance aveugle à `Zone.js` | Migration vers `Zoneless` et Signals stricts |
| **Flux Asynchrones** | Abonnements manuels `.subscribe()` | Composition par opérateurs d'annulation (`switchMap`, `takeUntilDestroyed`) |
| **Injection** | Déclarations uniques dans `root` | Exploitation maîtrisée des scopes d'arbres (`ElementInjector`, `{ skipSelf, host }`) |
| **DOM & Rendu** | Tout déléguer aux balises HTML | Usage ciblé de `runOutsideAngular`, `ViewContainerRef` et `EmbeddedViewRef` |
| **Sécurité Réseau** | Intercepteurs basiques de logs | Gestion du verrou de rafraîchissement de jeton, backoff exponentiel et idempotence |