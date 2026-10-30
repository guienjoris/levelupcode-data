# Quiz d'Évaluation — Angular (Niveau Senior)

Ce quiz évalue votre maîtrise des rouages internes et de l'ingénierie avancée sous Angular : architecture *Zoneless*, hiérarchie des injecteurs, résolution fine des dépendances, instanciation dynamique bas niveau et intercepteurs HTTP résilients.

---

### Question 1 : Mécanique interne de la détection de changements Zoneless

Lorsque l'application fonctionne avec `provideExperimentalZonelessChangeDetection()`, quel mécanisme informe directement le runtime qu'un nœud de l'arbre doit être réévalué lors de l'appel à `signal.set()` ?

* [ ] A) L'exécution immédiate d'un parcours synchrone descendant de l'ensemble de l'arbre de composants via `ApplicationRef.tick()`.
* [ ] B) L'enregistrement de la vue dépendante dans le graphe réactif qui la signale au scheduler pour traitement en microtask ciblée.
* [ ] C) L'interception native par monkey-patching des descripteurs de propriétés de l'objet global du navigateur `window`.
* [ ] D) L'émission d'un événement DOM custom bubbling capturé à la racine du document par le routeur principal.

---

### Question 2 : Résolution de dépendances et modifieurs d'injection

Quelle est la différence fondamentale entre les options de résolution `{ self: true }` et `{ host: true }` passées à la fonction `inject()` au sein d'une directive appliquée sur un composant ?

* [ ] A) `{ self: true }` restreint la recherche à l'injecteur de l'élément courant, tandis que `{ host: true }` s'arrête à la frontière de vue du composant hôte.
* [ ] B) `{ self: true }` recherche uniquement dans `EnvironmentInjector`, alors que `{ host: true }` inspecte exclusivement les services déclarés dans `root`.
* [ ] C) `{ self: true }` impose l'instanciation asynchrone du service, tandis que `{ host: true }` partage la même instance entre plusieurs fenêtres iframe.
* [ ] D) `{ self: true }` force l'injection d'un singleton global, alors que `{ host: true }` limite la visibilité aux classes déclarées privées.

---

### Question 3 : Encapsulation des services avec `viewProviders`

Dans un composant conteneur configurant simultanément des `providers` et des `viewProviders`, quel est l'impact précis pour un composant enfant inséré via `<ng-content></ng-content>` ?

* [ ] A) Le composant projeté accède librement aux services définis dans `viewProviders`, mais ne peut pas injecter ceux listés dans `providers`.
* [ ] B) Le composant projeté n'a pas accès aux instances déclarées dans `viewProviders` car il appartient logiquement au contexte de vue de son parent injecteur.
* [ ] C) Le moteur Ivy lève une erreur de compilation circulaire interdisant la projection de contenu dès lors que `viewProviders` est défini.
* [ ] D) Le composant projeté détruit automatiquement les singletons associés à `viewProviders` dès son initialisation dans le DOM.

---

### Question 4 : Optimisation de rendu bas niveau et NgZone

Pour quelle raison technique majeure isole-t-on l'écoute d'événements haute fréquence (ex: `mousemove`, `touchmove`) au sein de `NgZone.runOutsideAngular()` dans une application utilisant encore `Zone.js` ?

* [ ] A) Pour forcer l'interpréteur JavaScript à déporter le traitement des calculs vectoriels vers un thread Web Worker en arrière-plan.
* [ ] B) Pour empêcher les listeners d'exécuter un cycle de détection global coûteux (`tick()`) à chaque micro-mouvement de la souris.
* [ ] C) Pour contourner les mécanismes de sécurité CSP du navigateur qui interdisent l'accès aux coordonnées de la souris.
* [ ] D) Pour contraindre le moteur de rendu CSS à recalculer le layout sans passer par le pipeline standard du moteur de template.

---

### Question 5 : Cycle de vie et gestion mémoire avec `DestroyRef`

Quel avantage technique distinct apporte l'utilisation de `DestroyRef.onDestroy()` par rapport à l'implémentation classique du hook `ngOnDestroy()` ?

* [ ] A) Il permet d'enregistrer des routines de nettoyage au sein de fonctions utilitaires ou de composables sans forcer la classe hôte à implémenter l'interface.
* [ ] B) Il accélère la collecte du garbage collector en supprimant immédiatement les références d'objets sans attendre la libération de la boucle d'événements.
* [ ] C) Il garantit que les requêtes HTTP en cours aboutissent même si l'onglet du navigateur est fermé de façon abrupte par l'utilisateur.
* [ ] D) Il empêche le composant d'être détruit tant que subsiste au moins un abonnement actif sur un flux RxJS quelconque.

---

### Question 6 : Intercepteurs HTTP fonctionnels et renouvellement de jeton

Lors de l'interception d'une erreur HTTP 401 pour rafraîchir un token d'accès, quel opérateur RxJS doit être privilégié pour relancer la requête originale après obtention du nouveau jeton ?

* [ ] A) `concatMap`, pour stocker indéfiniment les requêtes dans une file d'attente FIFO sans jamais annuler les flux défaillants.
* [ ] B) `switchMap`, pour mapper l'émission du nouveau token vers l'appel réseau re-signé tout en garantissant la bonne séquence asynchrone.
* [ ] C) `exhaustMap`, afin d'ignorer systématiquement toutes les requêtes subséquentes déclenchées par l'utilisateur tant que l'erreur initiale persiste.
* [ ] D) `mergeMap`, pour déclencher immédiatement dix tentatives en parallèle sans attendre la réponse du serveur d'authentification.

---

### Question 7 : Instanciation dynamique moderne avec Ivy

Depuis la dépréciation définitive de `ComponentFactoryResolver`, comment instancie-t-on dynamiquement un composant tout en lui assignant une valeur sur une entrée réactive `input()` ?

* [ ] A) En récupérant la référence du nœud DOM via `ElementRef` et en invoquant la méthode native `window.customElements.define()`.
* [ ] B) En appelant `ViewContainerRef.createComponent(MonComposant)` puis en assignant les valeurs requises à l'aide de la méthode `ref.setInput()`.
* [ ] C) En compilant à la volée une chaîne de template HTML brute via l'injecteur bas niveau `Compiler.compileModuleAndAllComponentsAsync()`.
* [ ] D) En instanciant directement la classe avec le mot-clé `new MonComposant()` avant de l'insérer dans le tableau des nœuds Ivy.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** Dans une architecture *Zoneless*, l'absence de `Zone.js` supprime l'interception automatique des tâches asynchrones. La réactivité repose sur le graphe de dépendances des Signals : lorsqu'un signal subit une mutation (`set()` ou `update()`), les nœuds consommateurs (les templates de vues enregistrés) sont marqués comme invalides (*dirty*) et le scheduler planifie un microtask pour rafraîchir précisément ces vues ciblées, éliminant le coût des traversées globales d'arbre.

### 2. Réponse : **A**
**Explication :** `{ self: true }` restreint la résolution à l'élément DOM sur lequel la directive ou le composant est attaché ; s'il n'y est pas fourni, la résolution échoue immédiatement (ou renvoie `null` si combiné avec `{ optional: true }`). `{ host: true }` remonte la chaîne des injecteurs d'éléments mais s'arrête strictement à la frontière du composant parent qui héberge le template (le composant hôte), interdisant de chercher plus haut dans les injecteurs ancêtres de l'arbre global.

### 3. Réponse : **B**
**Explication :** L'option `viewProviders` limite la visibilité des dépendances à la vue interne (*View*) définie dans le template du composant hôte. Tout composant projeté via `<ng-content>` fait partie du contenu logique (*ContentChildren*) déclaré par le parent consommateur : il dépend donc de l'injecteur du parent et n'a pas accès aux `viewProviders` du conteneur dans lequel il est projeté.

### 4. Réponse : **B**
**Explication :** `Zone.js` intercepte tous les événements DOM. Pour des événements émis à cadence d'affichage (60 à 120 fois par seconde comme `mousemove` ou `scroll`), chaque déclenchement déclenche un `ApplicationRef.tick()` qui inspecte l'intégralité de l'application. Exécuter le listener via `NgZone.runOutsideAngular()` permet d'écouter l'événement sans notifier Zone.js, préservant ainsi la fluidité et le thread principal.

### 5. Réponse : **A**
**Explication :** `DestroyRef` abstrait le cycle de vie de destruction hors de la structure rigide de classe. Il peut être injecté dans n'importe quel contexte d'instanciation (constructeurs, fonctions utilitaires personnalisées, pipelines réactifs) pour attacher un callback via `onDestroy()`. Cela facilite l'écriture de code réutilisable sans contraindre l'utilisateur final à implémenter `OnDestroy` manuellement sur ses composants.

### 6. Réponse : **B**
**Explication :** Lorsque l'appel initial échoue avec un code 401, le traitement asynchrone doit basculer vers la requête d'obtention du token rafraîchi (`refreshToken()`). Dès que ce dernier émet sa nouvelle valeur, `switchMap` permet d'enchaîner proprement vers la réexécution de la requête HTTP initiale clonée avec les nouveaux en-têtes d'autorisation.

### 7. Réponse : **B**
**Explication :** Avec le moteur Ivy, `ViewContainerRef.createComponent(Type)` ne nécessite plus de passer par une fabrique de composants précompilée (`ComponentFactoryResolver`). Pour configurer les entrées déclarées via les nouveaux signaux d'entrée (`input()`), on utilise l'API `ref.setInput(nom, valeur)`, qui garantit le suivi réactif et la bonne exécution des mécanismes internes de détection de changements.