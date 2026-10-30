# Initiation à Angular (Niveau Novice)

**Angular** est un framework complet (*batteries included*) développé et maintenu par Google pour concevoir des applications web interactives, robustes et de grande échelle.

Contrairement à des bibliothèques comme React qui nécessitent d'assembler vous-même le routage, la gestion des formulaires et les requêtes HTTP, Angular fournit un ensemble d'outils officiel, standardisé et pré-intégré dès le départ.

---

## 1. La Philosophie d'Angular : TypeScript & Structure Forte

Pour bien débuter avec Angular, il faut comprendre ses trois piliers :
- **TypeScript par défaut :** Tout le code Angular exploite le typage statique, ce qui limite considérablement les bugs d'exécution.
- **Architecture par composants :** L'interface utilisateur est découpée en blocs autonomes et réutilisables.
- **Convention & Outillage :** La CLI (*Command Line Interface*) officielle génère, teste et déploie le projet de manière standardisée.

---

## 2. Anatomie d'un Composant Moderne (Standalone Component)

Depuis ses versions récentes, Angular a grandement simplifié son modèle en adoptant les **Standalone Components** (qui ne nécessitent plus la configuration lourde des anciens `NgModule`).

Un composant réunit trois aspects :
1. **La logique :** Une classe TypeScript décorée avec `@Component`.
2. **La vue :** Un template HTML.
3. **Le style :** Du CSS / SCSS encapsulé qui n'affecte pas le reste de l'application.

```typescript
// user-card.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-user-card',       // Balise personnalisée : <app-user-card></app-user-card>
  standalone: true,                // Composant autonome moderne
  imports: [],                     // Dépendances de template (autres composants/directives)
  templateUrl: './user-card.component.html',
  styleUrl: './user-card.component.css'
})
export class UserCardComponent {
  // Propriétés (état local)
  username: string = 'Alexandre';
  isOnline: boolean = true;

  // Méthode (action)
  toggleStatus(): void {
    this.isOnline = !this.isOnline;
  }
}
```

---

## 3. Le Binding : Relier la Logique et le Template

Angular utilise une syntaxe de liaison (*data binding*) très intuitive reposant sur des conventions de ponctuation précises :

```
                  ┌──────────────────────────────────────────────┐
                  │                 LOGIQUE (TS)                 │
                  └──────┬───────────────────────────────▲───────┘
                         │                               │
         Interpolation   │               Événement       │
            {{ }}        │                 ( )           │
         Propriété       ▼                               │
            [ ]   ┌──────────────────────────────────────┴───────┐
                  │                TEMPLATE (HTML)               │
                  └──────────────────────────────────────────────┘
                         ▲                               ▲
                         └──────────────┬────────────────┘
                                        │
                                 Two-Way Binding
                                     [( )]
```

### 3.1 Interpolation textuelle `{{ }}`
Affiche une variable TypeScript dans le HTML :
```html
<h1>Bonjour {{ username }} !</h1>
```

### 3.2 Property Binding `[propriete]`
Transmet une valeur de la classe vers un attribut HTML ou une entrée de composant :
```html
<!-- La propriété disabled du bouton dépend de la variable boolean isOnline -->
<button [disabled]="!isOnline">Contacter</button>
<img [src]="avatarUrl" [alt]="username" />
```

### 3.3 Event Binding `(evenement)`
Écoute une interaction utilisateur et appelle une méthode TypeScript :
```html
<button (click)="toggleStatus()">Changer le statut</button>
```

### 3.4 Liaison bidirectionnelle `[(ngModel)]` (*Two-Way Binding*)
Synchronise automatiquement la valeur d'un champ de saisie avec une variable (souvent appelée syntaxe *banana-in-a-box*) :
```html
<!-- Requiert FormsModule dans les imports -->
<input [(ngModel)]="username" placeholder="Entrez votre nom" />
<p>Vous avez tapé : {{ username }}</p>
```

---

## 4. Nouveau Système de Contrôle de Flux : `@if` et `@for`

Auparavant basé sur les directives `*ngIf` et `*ngFor`, Angular intègre désormais une syntaxe déclarative native, beaucoup plus rapide et lisible.

### 4.1 Conditions avec `@if`
```html
@if (isOnline) {
  <span class="badge online">En ligne</span>
} @else {
  <span class="badge offline">Absent</span>
}
```

### 4.2 Boucles avec `@for`
La clause `track` est obligatoire : elle permet à Angular d'optimiser le rendu du DOM en traçant l'identité unique de chaque élément (comme la `key` dans React).
```html
<ul>
  @for (task of tasks; track task.id) {
    <li>{{ task.title }}</li>
  } @empty {
    <li>Aucune tâche pour le moment !</li>
  }
</ul>
```

---

## 5. Services et Injection de Dépendances (*Dependency Injection*)

Un composant ne doit s'occuper que de l'affichage et de l'interaction directe de l'utilisateur. La logique de gestion des données ou les appels d'API externes sont confiés à des **Services**.

Angular intègre un système d'**Injection de Dépendances** (DI) puissant :

### 5.1 Créer un Service (`@Injectable`)
```typescript
// task.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' // Singleton disponible dans toute l'application
})
export class TaskService {
  private tasks: string[] = ['Apprendre TypeScript', 'Découvrir Angular'];

  getTasks(): string[] {
    return this.tasks;
  }

  addTask(task: string): void {
    this.tasks.push(task);
  }
}
```

### 5.2 Injecter le Service dans un Composant
On utilise aujourd'hui la fonction moderne `inject()` :
```typescript
// task-list.component.ts
import { Component, inject } from '@angular/core';
import { TaskService } from './task.service';

@Component({
  selector: 'app-task-list',
  standalone: true,
  template: `
    <ul>
      @for (task of tasks; track $index) {
        <li>{{ task }}</li>
      }
    </ul>
  `
})
export class TaskListComponent {
  // Injection propre sans passer par le constructeur
  private taskService = inject(TaskService);
  tasks = this.taskService.getTasks();
}
```

---

## 6. Introduction aux Signaux (*Angular Signals*)

Les **Signals** représentent le moteur de réactivité moderne d'Angular. Un Signal est une valeur enveloppée qui notifie automatiquement le framework dès qu'elle est modifiée, permettant des mises à jour chirurgicales du DOM sans recalculer toute la page.

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Compteur : {{ count() }}</p>
    <p>Double : {{ doubleCount() }}</p>
    <button (click)="increment()">+1</button>
  `
})
export class CounterComponent {
  // 1. Un signal modifiable
  count = signal<number>(0);

  // 2. Un signal calculé (dérivé automatique)
  doubleCount = computed(() => this.count() * 2);

  increment(): void {
    // 3. Mise à jour de la valeur
    this.count.update(value => value + 1);
  }
}
```

> **À retenir :** Pour lire la valeur d'un signal dans le template ou le TypeScript, on l'appelle comme une fonction : `count()`.

---

## Synthèse des Bonnes Pratiques Débutant

1. **Privilégier les composants `standalone: true` :** C'est le standard moderne qui allège l'arborescence et supprime le besoin de modules intermédiaires.
2. **Utiliser le nouveau contrôle de flux :** Adoptez `@if` et `@for` à la place de l'ancienne syntaxe à astérisque (`*ngIf`, `*ngFor`).
3. **Déléguer la logique métier aux services :** Ne faites pas de calculs lourds ni de requêtes réseau directes au milieu de vos composants.
4. **Adopter les Signals :** Pour gérer vos variables d'état locales, préférez les signaux (`signal()`) aux variables simples pour bénéficier de la réactivité optimale d'Angular.