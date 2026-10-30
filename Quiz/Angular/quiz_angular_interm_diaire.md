# Quiz d'Évaluation — Angular (Niveau Intermédiaire)

Ce quiz évalue votre maîtrise des notions intermédiaires d'Angular : intégration hybride RxJS / Signals, formulaires réactifs typés, stratégie de détection de changement `OnPush`, cycle de vie moderne et guards fonctionnels.

### Question 1 : Interopérabilité RxJS et Signals

Quelle fonction fournie par le package `@angular/core/rxjs-interop` permet de transformer un flux asynchrone (`Observable`) en un Signal réactif tout en gérant automatiquement le désabonnement lors de la destruction du composant ?

* [ ] A) `toSignal()`
* [ ] B) `fromObservable()`
* [ ] C) `createSignalFromStream()`
* [ ] D) `pipeToSignal()`

### Question 2 : Stratégie de détection `ChangeDetectionStrategy.OnPush`

Dans un composant configuré avec `ChangeDetectionStrategy.OnPush`, dans laquelle de ces situations Angular effectuera-t-il **automatiquement** une nouvelle passe de vérification du template ?

* [ ] A) Lorsqu'un appel `setTimeout` anonyme s'exécute en arrière-plan sans toucher aux données du composant.
* [ ] B) Dès qu'une requête HTTP d'un service tiers non lié au composant se termine avec succès.
* [ ] C) Lorsqu'une propriété passée en entrée (`@Input()` ou `input()`) reçoit une nouvelle référence mémoire.
* [ ] D) À chaque fois qu'une variable interne non signal est modifiée directement par réassignation.

### Question 3 : Formulaires réactifs typés et valeurs par défaut

Quel avantage direct procure l'utilisation du service `NonNullableFormBuilder` par rapport au constructeur classique `FormBuilder` lors de la réinitialisation d'un champ ?

* [ ] A) Il empêche le champ de retomber à `null` lors de l'appel à `.reset()`, restaurant à la place sa valeur initiale définie à la création.
* [ ] B) Il désactive systématiquement l'intégralité des validateurs asynchrones configurés pour accélérer le traitement du DOM.
* [ ] C) Il interdit la modification des valeurs du formulaire par l'utilisateur une fois que le premier caractère a été saisi.
* [ ] D) Il convertit l'ensemble des données du formulaire en format binaire chiffré avant toute soumission réseau.

### Question 4 : Guards de routage modernes

Depuis l'introduction des API fonctionnelles dans le routeur Angular, comment définit-on un guard de protection d'accès (`CanActivate`) sans créer de classe dédiée ?

* [ ] A) En écrivant une fonction pure du type `CanActivateFn` qui exploite directement la fonction `inject()` pour récupérer ses dépendances.
* [ ] B) En implémentant impérativement une classe abstraite munie du décorateur `@GuardConfig({ functional: true })`.
* [ ] C) En configurant une directive personnalisée sur la balise `<router-outlet>` dans le composant racine.
* [ ] D) En déclarant un intercepteur HTTP capable de bloquer les trames réseau à destination des routes cibles.

### Question 5 : Nettoyage de ressources moderne

Pour libérer un timer ou un abonnement manuel sans avoir à implémenter explicitement l'interface `OnDestroy` et sa méthode `ngOnDestroy()`, quel utilitaire moderne utilise-t-on ?

* [ ] A) L'injectable `DestroyRef` combiné à sa méthode d'enregistrement de callback `.onDestroy()`.
* [ ] B) L'annotation `@AutoClean` apposée directement au-dessus des identifiants d'intervalles JavaScript.
* [ ] C) La méthode globale `cleanUpComponentContext()` appelée depuis le constructeur du composant.
* [ ] D) L'opérateur RxJS `cancelOnLeave()` placé à la fin de chaque souscription interne.

### Question 6 : Cas d'usage de la fonction `effect()`

Quel est le rôle principal de la primitive réactive `effect()` introduite avec le système de signaux d'Angular ?

* [ ] A) Calculer une valeur synchrone pure et la mettre en cache sans exécuter d'effets de bord.
* [ ] B) Exécuter un effet de bord réactif qui s'exécute automatiquement dès qu'au moins un signal lu en son sein change de valeur.
* [ ] C) Remplacer définitivement les services de routage pour la transition dynamique entre deux pages web.
* [ ] D) Forcer le rechargement complet de la page HTML dans le navigateur lors d'une erreur critique.

### Question 7 : Validateurs personnalisés synchrones

Quelle signature doit respecter une fonction de validation personnalisée synchrone passée à un `FormControl` ?

* [ ] A) `(control: AbstractControl) => ValidationErrors | null`
* [ ] B) `(value: any) => boolean`
* [ ] C) `(control: FormControl) => Promise<string>`
* [ ] D) `(input: HTMLElement) => Observable<boolean>`

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** La fonction `toSignal(observable$, options)` convertit un flux RxJS en signal. Elle gère son propre cycle de vie et se désabonne automatiquement lorsque le contexte d'injection (le composant ou le service) est détruit, évitant ainsi les fuites de mémoire sans exiger d'appel manuel à `.unsubscribe()`.

### 2. Réponse : **C**
**Explication :** Avec `ChangeDetectionStrategy.OnPush`, Angular n'analyse pas l'arbre aveuglément à chaque événement global. Il ne réévalue le template que si une entrée (`input`) reçoit une nouvelle référence (comparaison stricte `===`), si un signal suivi dans le template émet, ou si un événement prend naissance dans le composant lui-même.

### 3. Réponse : **A**
**Explication :** Dans les formulaires réactifs traditionnels, appeler `form.reset()` remet la valeur de chaque contrôle à `null`, ce qui casse le typage strict (`string | null`). Le `NonNullableFormBuilder` génère des contrôles non-nullables dont la valeur par défaut lors du reset redevient celle fournie lors de l'instanciation.

### 4. Réponse : **A**
**Explication :** Les guards basés sur des classes (`implements CanActivate`) sont désormais dépréciés au profit des guards fonctionnels de type `CanActivateFn`. Ils s'écrivent sous la forme de fonctions simples qui utilisent `inject(MonService)` pour accéder au contexte d'injection sans boilerplate de classe.

### 5. Réponse : **A**
**Explication :** `DestroyRef` permet d'enregistrer des routines de nettoyage n'importe où dans le cycle d'instanciation (par exemple dans une fonction utilitaire ou un constructeur) grâce à `destroyRef.onDestroy(() => ...)`. Cela évite d'alourdir le composant avec l'interface `OnDestroy`.

### 6. Réponse : **B**
**Explication :** La fonction `effect()` est destinée aux effets de bord (journalisation, synchronisation avec le `localStorage`, manipulation impérative d'un canvas tiers). Pour dériver ou calculer une nouvelle valeur pure à partir d'autres signaux, on utilise plutôt `computed()`.

### 7. Réponse : **A**
**Explication :** Un validateur synchrone standard reçoit un `AbstractControl` en paramètre et retourne soit un objet de description d'erreur (ex. `{ nonCorporate: true }`) en cas d'échec de validation, soit strictement `null` si la valeur est conforme.