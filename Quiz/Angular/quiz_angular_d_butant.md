# Quiz d'Évaluation — Angular (Niveau Novice)

Ce quiz évalue votre assimilation des fondamentaux d'Angular moderne : la structure des composants autonomes (*Standalone Components*), les types de liaisons de données (*data binding*), le contrôle de flux déclaratif (`@if`, `@for`), l'injection de dépendances et les signaux.

### Question 1 : Composants Standalone

Depuis les versions modernes d'Angular, quelle propriété passée au décorateur `@Component` permet d'utiliser un composant directement sans avoir à le déclarer dans un `NgModule` ?

* [ ] A) `standalone: true`
* [ ] B) `isolated: true`
* [ ] C) `modular: false`
* [ ] D) `autonomous: true`

### Question 2 : Syntaxe du Data Binding

Quelle combinaison de symboles correspond respectivement à la liaison de propriété (*Property Binding*, logique $\rightarrow$ template) et à la liaison d'événement (*Event Binding*, template $\rightarrow$ logique) ?

* [ ] A) `{{ }}` pour la propriété et `[ ]` pour l'événement.
* [ ] B) `[ ]` pour la propriété et `( )` pour l'événement.
* [ ] C) `( )` pour la propriété et `[ ]` pour l'événement.
* [ ] D) `[( )]` pour la propriété et `{{ }}` pour l'événement.

### Question 3 : Nouveau contrôle de flux pour les boucles

Dans la nouvelle syntaxe `@for` d'Angular, quel paramètre est **obligatoire** pour permettre au framework d'optimiser le suivi et la mise à jour des éléments du DOM ?

* [ ] A) `track`
* [ ] B) `as index`
* [ ] C) `key`
* [ ] D) `identity`

### Question 4 : Injection de dépendances et portée

Quelle déclaration au sein du décorateur `@Injectable` indique qu'un service doit être instancié sous forme de singleton accessible dans l'ensemble de l'application ?

* [ ] A) `@Injectable({ scope: 'global' })`
* [ ] B) `@Injectable({ providedIn: 'root' })`
* [ ] C) `@Injectable({ singleton: true })`
* [ ] D) `@Injectable({ export: 'all' })`

### Question 5 : Two-Way Data Binding

Pour synchroniser instantanément la saisie d'un champ `<input>` avec une variable de votre classe TypeScript via la directive `[(ngModel)]`, quel module devez-vous importer ?

* [ ] A) `CommonModule`
* [ ] B) `ReactiveFormsModule`
* [ ] C) `FormsModule`
* [ ] D) `BrowserAnimationsModule`

### Question 6 : Lecture de la valeur d'un Signal

Dans une application utilisant les nouveaux *Signals* d'Angular, comment lit-on la valeur d'un signal nommé `totalCount` dans un template ou dans une méthode TypeScript ?

* [ ] A) En l'invoquant comme une fonction sans argument : `totalCount()`.
* [ ] B) En utilisant la propriété interne de déréférencement : `totalCount.value`.
* [ ] C) En appliquant obligatoirement le pipe d'abonnement asynchrone : `totalCount | async`.
* [ ] D) En accédant au pointeur brut : `$totalCount`.

### Question 7 : Conditions et alternatives avec le contrôle de flux

Parmi les extraits suivants, lequel représente la syntaxe moderne valide pour afficher un bloc de repli lorsque la liste des utilisateurs est vide ?

* [ ] A)
```html
@for (user of users; track user.id) {
  <p>{{ user.name }}</p>
} @empty {
  <p>Aucun utilisateur trouvé.</p>
}
```
* [ ] B)
```html
<div *ngFor="let user of users" *ngIf="users.length; else noUsers">
  <p>{{ user.name }}</p>
</div>
<ng-template #noUsers><p>Aucun utilisateur trouvé.</p></ng-template>
```
* [ ] C)
```html
@repeat (user in users) {
  <p>{{ user.name }}</p>
} @fallback {
  <p>Aucun utilisateur trouvé.</p>
}
```
* [ ] D)
```html
@for (user of users) {
  <p>{{ user.name }}</p>
  @else {
    <p>Aucun utilisateur trouvé.</p>
  }
}
```

---

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** La propriété `standalone: true` indique à Angular que le composant gère lui-même ses dépendances (via son tableau `imports`) sans nécessiter d'enregistrement au sein d'un bloc intermédiaire `NgModule`.

### 2. Réponse : **B**
**Explication :** En Angular, les crochets `[property]` symbolisent le flux entrant vers l'élément (Property Binding), tandis que les parenthèses `(event)` symbolisent le flux sortant déclenché par une action utilisateur (Event Binding). La liaison bidirectionnelle combine les deux : `[(ngModel)]`.

### 3. Réponse : **A**
**Explication :** L'instruction `@for` exige la clause `track` (par exemple `track item.id` ou `track $index`). Elle fournit à Angular un identifiant stable pour réutiliser les nœuds DOM existants au lieu de recréer l'ensemble de la liste lors des rafraîchissements.

### 4. Réponse : **B**
**Explication :** L'option `{ providedIn: 'root' }` configure l'injecteur racine d'Angular pour fournir une instance unique (*singleton*) du service, tout en autorisant le *tree-shaking* (le service est exclu du bundle final s'il n'est jamais injecté).

### 5. Réponse : **C**
**Explication :** La directive `[(ngModel)]` fait partie de la bibliothèque `@angular/forms` et requiert l'import explicite de `FormsModule` dans le tableau `imports` du composant Standalone concerné.

### 6. Réponse : **A**
**Explication :** Les Signaux sont des fonctions accesseurs réactives (*getter functions*). Pour extraire leur contenu courant, on les appelle avec des parenthèses : `totalCount()`. Cette syntaxe permet au moteur de réactivité d'enregistrer précisément quel composant dépend de cette donnée.

### 7. Réponse : **A**
**Explication :** La structure intégrée `@for` dispose du mot-clé `@empty`, qui s'active automatiquement si la collection itérée est nulle, indéfinie ou vide, simplifiant le code sans nécessiter de condition `@if` imbriquée.