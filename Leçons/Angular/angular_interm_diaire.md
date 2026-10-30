# Angular Moderne — Niveau Intermédiaire

Après avoir acquis les bases d'Angular (composants autonomes, syntaxe de template `@if`/`@for`, interpolation et services élémentaires), l'étape intermédiaire consiste à maîtriser les mécanismes professionnels du framework : la gestion fine de l'état asynchrone, les formulaires réactifs complexes, l'optimisation des performances de rendu et la sécurisation de la navigation.

---

## 1. Réactivité Hybride : Signals et RxJS

Angular s'appuie désormais sur deux paradigmes complémentaires pour gérer les données :

* **Les Signals :** Pour l'état réactif synchrone, les valeurs simples et la mise à jour granulaire du template.
* **RxJS (`Observables`) :** Pour les flux d'événements asynchrones dans le temps (requêtes HTTP, websockets, debounce sur saisie utilisateur).

### 1.1 La passerelle `@angular/core/rxjs-interop`

Pour naviguer facilement entre ces deux mondes, Angular fournit deux fonctions utilitaires indispensables : `toSignal` et `toObservable`.

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { toSignal } from '@angular/core/rxjs-interop';

interface User {
  id: number;
  name: string;
}

@Component({
  selector: 'app-user-profile',
  standalone: true,
  template: `
    @if (users(); as userList) {
      <ul>
        @for (u of userList; track u.id) {
          <li>{{ u.name }}</li>
        }
      </ul>
    }
  `
})
export class UserProfileComponent {
  private http = inject(HttpClient);

  // Conversion directe d'un flux HTTP RxJS en Signal réactif
  // toSignal gère automatiquement le désabonnement (unsubscribe) lors de la destruction du composant !
  users = toSignal(this.http.get<User[]>('/api/users'), { initialValue: [] });
}
```

### 1.2 Quand choisir l'un plutôt que l'autre ?

| Cas d'usage | Outil recommandé | Raison |
| :--- | :--- | :--- |
| Variable d'affichage local (ex: compteur, modal ouverte) | **Signal** | Synchrone, syntaxe légère sans fuite mémoire. |
| Champ de recherche avec autocomplétion (`debounceTime`) | **RxJS** | Nécessite des opérateurs temporels pour filtrer la cadence. |
| Appel API ponctuel affiché dans le template | **RxJS converti en Signal** (`toSignal`) | Permet d'éviter le pipe `| async` et l'affectation manuelle. |
| Donnée calculée dérivée d'un état | **Signal (`computed`)** | Mémorisation (*memoization*) automatique et recalcul ciblé. |

---

## 2. Formulaires Réactifs Typés (*Typed Reactive Forms*)

Les formulaires réactifs offrent un contrôle programmatique rigoureux, un typage statique strict et une testabilité unitaire sans rendu HTML.

### 2.1 Construction avec `NonNullableFormBuilder`

L'utilisation de `NonNullableFormBuilder` garantit que les champs du formulaire ne reviennent pas à `null` lors d'un appel à `.reset()`.

```typescript
import { Component, inject } from '@angular/core';
import { NonNullableFormBuilder, Validators, ReactiveFormsModule, AbstractControl, ValidationErrors } from '@angular/forms';

// Validateur personnalisé synchrone
function corporateEmailValidator(control: AbstractControl): ValidationErrors | null {
  const value: string = control.value || '';
  return value.endsWith('@company.com') ? null : { nonCorporate: true };
}

@Component({
  selector: 'app-register-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './register-form.component.html'
})
export class RegisterFormComponent {
  private fb = inject(NonNullableFormBuilder);

  // Formulaire entièrement typé automatiquement
  registerForm = this.fb.group({
    username: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email, corporateEmailValidator]],
    newsletter: [false]
  });

  onSubmit(): void {
    if (this.registerForm.valid) {
      // Les données sont garanties non-nullables et typées
      const payload = this.registerForm.getRawValue();
      console.log('Données prêtes à l\'envoi :', payload);
    } else {
      this.registerForm.markAllAsTouched();
    }
  }
}
```

---

## 3. Détection de Changements Optimisée : `ChangeDetectionStrategy.OnPush`

Par défaut (`Default`), Angular vérifie l'ensemble de l'arbre des composants lors de chaque événement asynchrone (clic, timeout, réponse HTTP). Sur une interface lourde, cela crée des calculs superflus.

La stratégie **`OnPush`** ordonne à Angular de ne vérifier le composant que dans des cas stricts :
1. Une référence d'une entrée (`@Input()`) a changé (comparaison par référence `===`).
2. Un événement utilisateur a pris son origine dans le composant ou l'un de ses enfants.
3. Un **Signal** consommé dans le template a émis une nouvelle valeur.
4. L'opérateur de rafraîchissement manuel `ChangeDetectorRef.markForCheck()` a été invoqué.

```typescript
import { Component, ChangeDetectionStrategy, input } from '@angular/core';

@Component({
  selector: 'app-metric-display',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush, // Optimisation activée
  template: `
    <div class="card">
      <h3>{{ title() }}</h3>
      <p>Valeur : {{ score() }}</p>
    </div>
  `
})
export class MetricDisplayComponent {
  // Entrées réactives modernes (Angular Signals Inputs)
  title = input.required<string>();
  score = input<number>(0);
}
```

---

## 4. Cycle de Vie Moderne (*Lifecycle Hooks*)

Avec les Standalone Components et les Signals, les hooks traditionnels restent disponibles, mais la nouvelle fonction `effect()` simplifie la réaction aux mutations :

```
Création du composant
       │
       ▼
  constructeur()           (Initialisation des services et injections)
       │
       ▼
  ngOnInit()               (Initialisation de la logique métier)
       │
       ▼
  ngAfterViewInit()        (Accès au DOM ou aux éléments enfants @ViewChild)
       │
       ▼
  ngOnDestroy()            (Nettoyage des abonnements manuels, timers)
```

### Utilisation moderne avec `effect()` et `DestroyRef`

Pour nettoyer des ressources sans surcharger `ngOnDestroy`, on utilise désormais `DestroyRef` :

```typescript
import { Component, signal, effect, inject, DestroyRef } from '@angular/core';

@Component({
  selector: 'app-timer',
  standalone: true,
  template: `<p>Timer : {{ count() }}</p>`
})
export class TimerComponent {
  count = signal(0);
  private destroyRef = inject(DestroyRef);

  constructor() {
    // 1. Déclenche une action à chaque mutation de `count`
    effect(() => {
      console.log(`Nouvelle valeur du timer enregistrée : ${this.count()}`);
    });

    // 2. Nettoyage moderne à la destruction sans implémenter OnDestroy
    const intervalId = setInterval(() => this.count.update(c => c + 1), 1000);
    this.destroyRef.onDestroy(() => clearInterval(intervalId));
  }
}
```

---

## 5. Routage & Guards Fonctionnels

Historiquement conçus sous forme de classes implémentant une interface (`CanActivate`), les **Guards** modernes s'écrivent aujourd'hui sous forme de **fonctions pures injectables**.

### 5.1 Création d'un Guard fonctionnel

```typescript
// auth.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  // Redirection immédiate si non authentifié
  return router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
};
```

### 5.2 Déclaration dans les routes avec Lazy Loading

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { authGuard } from './auth.guard';

export const routes: Routes = [
  { path: 'login', loadComponent: () => import('./login.component').then(m => m.LoginComponent) },
  {
    path: 'admin',
    canActivate: [authGuard], // Protection fonctionnelle
    loadComponent: () => import('./admin.component').then(m => m.AdminComponent) // Chargement différé
  }
];
```

---

## Synthèse des Bonnes Pratiques Intermédiaires

1. **Activez `ChangeDetectionStrategy.OnPush` par défaut :** Cela évite les cycles de rafraîchissement globaux et force à utiliser des structures de données immuables.
2. **Évitez les abonnements manuels (`.subscribe()`) dans vos composants :** Passez par `toSignal()` ou le pipe `async` pour que la souscription et le nettoyage soient gérés automatiquement par Angular.
3. **Privilégiez les formulaires réactifs typés :** N'utilisez pas de formulaires basés sur les templates (*Template-driven forms*) dès que la logique comporte plus de deux règles de validation.
4. **Utilisez les Guards fonctionnels :** Plus compacts, ils s'injectent directement dans la configuration du routeur sans surcharger la base de code de classes intermédiaires.