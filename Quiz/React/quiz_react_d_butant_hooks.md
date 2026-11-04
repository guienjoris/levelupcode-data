# Quiz d'Évaluation — React Hooks (Niveau Novice)

Ce quiz valide votre compréhension des mécaniques pratiques des hooks fondamentaux de React : les règles d'appel, la mise à jour fonctionnelle de `useState`, le cycle de nettoyage de `useEffect`, la manipulation de `useRef` et la consommation de contextes avec `useContext`.

### Question 1 : Règles fondamentales des Hooks

Pourquoi React interdit-il formellement d'appeler un hook (comme `useState` ou `useEffect`) à l'intérieur d'une condition `if` ou d'une boucle `for` ?

* [ ] A) Parce que le compilateur Babel est incapable de transpiler le mot-clé `if` lorsqu'il précède un hook.
* [ ] B) Parce que React s'appuie sur l'ordre d'appel séquentiel et constant des hooks à chaque rendu pour associer leur état.
* [ ] C) Parce que les hooks conditionnels provoquent systématiquement une erreur de mémoire vive au niveau du serveur Node.js.
* [ ] D) Parce que les conditions obligent le navigateur à désactiver la synchronisation automatique du DOM virtuel.

### Question 2 : Mise à jour d'état fonctionnelle

Dans un gestionnaire d'événement de clic, vous écrivez :
```javascript
setScore(score + 1);
setScore(score + 1);
```
Si la valeur initiale de `score` est `0`, quelle sera la valeur du score au rendu suivant, et comment corriger ce comportement pour obtenir `2` ?

* [ ] A) La valeur sera `2` car React regroupe les appels, aucune modification de code n'est donc requise.
* [ ] B) La valeur sera `0` car les deux setters s'annulent mutuellement ; il faut forcer avec `flushSync()`.
* [ ] C) La valeur sera `1` en raison de la lecture d'un état figé ; il faut écrire `setScore(prev => prev + 1)`.
* [ ] D) Une exception de type `TypeError` est déclenchée ; il faut déclarer deux variables d'état indépendantes.

### Question 3 : Fonction de nettoyage dans `useEffect`

Dans quelle circonstance précise la fonction de retour (*cleanup function*) d'un `useEffect` s'exécute-t-elle ?

* [ ] A) Uniquement lorsque l'ensemble de l'application web est rechargée via le raccourci du navigateur.
* [ ] B) Immédiatement avant le premier rendu du composant pour réinitialiser les variables locales.
* [ ] C) Avant la réexécution de l'effet lors d'un changement de dépendance, ainsi qu'au démontage final du composant.
* [ ] D) Seulement si une promesse asynchrone interne renvoie explicitement un statut d'erreur HTTP 500.

### Question 4 : Comportement de `useRef` face au rendu

Quelle est la conséquence directe d'une modification manuelle de la propriété `ref.current` (`ref.current = nouvelleValeur`) ?

* [ ] A) Le composant met à jour la valeur immédiatement en mémoire sans provoquer de nouveau rendu à l'écran.
* [ ] B) React déclenche instantanément une nouvelle passe de rendu pour mettre à jour les balises du DOM.
* [ ] C) La valeur précédente est archivée dans le localStorage afin de restaurer l'état en cas d'erreur.
* [ ] D) Le composant enfant le plus proche est démonté puis remonté automatiquement dans l'arborescence.

### Question 5 : Rôle du hook `useContext`

Quel problème récurrent de développement dans une application React le hook `useContext` permet-il de résoudre efficacement ?

* [ ] A) La transformation automatique des requêtes API asynchrones en signaux de données synchrones.
* [ ] B) La propagation manuelle fastidieuse d'une propriété à travers de multiples composants intermédiaires (*prop drilling*).
* [ ] C) L'obligation d'écrire des tests unitaires pour valider les formulaires de saisie complexes.
* [ ] D) La fuite de mémoire causée par des minuteurs `setInterval` non désalloués au démontage.

### Question 6 : Dépendances et exécution de `useEffect`

Soit le composant suivant :
```javascript
useEffect(() => {
  console.log("Action exécutée");
});
```
Sans tableau de dépendances spécifié en second argument, quand ce message s'affichera-t-il dans la console ?

* [ ] A) Strictement une seule fois lors de l'insertion initiale du composant dans le DOM.
* [ ] B) Après chaque rendu du composant, qu'il s'agisse du montage initial ou d'une mise à jour de state.
* [ ] C) Uniquement lorsque le composant est retiré de la page par l'utilisateur.
* [ ] D) Jamais, car l'absence de tableau de dépendances constitue une erreur de syntaxe bloquante.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** React ne dispose pas de noms ou d'identifiants magiques pour retrouver quel `useState` correspond à quelle variable. Sous le capot, React gère une liste chaînée (ou un tableau indexé) de cellules d'état. Il compte sur le fait que les hooks sont appelés **exactement dans le même ordre à chaque rendu**. Si un hook est enfermé dans un `if`, l'ordre se décale dès que la condition change, ce qui corrompt tous les états suivants.

### 2. Réponse : **C**
**Explication :** Lors du rendu, la variable `score` est une constante locale issue de la fermeture (*closure*) courante (valant `0`). Les deux lignes évaluent donc `setScore(0 + 1)`. React applique le *batching* et conclut que le nouvel état doit valoir `1`. En utilisant la forme fonctionnelle `setScore(prev => prev + 1)`, React passe à la fonction le résultat de la mise à jour précédente, garantissant une incrémentation cumulée exacte.

### 3. Réponse : **C**
**Explication :** La fonction de nettoyage retournée par `useEffect` a pour rôle de défaire ce que l'effet a installé. Elle est invoquée juste avant que l'effet ne se réexécute (si l'une des dépendances a changé) afin d'éviter les abonnements en double, ainsi qu'au moment où le composant quitte définitivement l'écran (*unmount*).

### 4. Réponse : **A**
**Explication :** Contrairement à `useState`, modifier `ref.current` est une simple mutation de propriété sur un objet JavaScript classique. React n'est pas notifié et ne déclenche aucun recalcul de rendu. C'est l'outil idéal pour retenir des minuteurs, des compteurs d'accès internes ou des références vers des éléments DOM natifs sans impacter le cycle de rendu.

### 5. Réponse : **B**
**Explication :** Le *prop drilling* survient quand un composant racine doit transmettre une donnée (ex. le thème, la langue, l'utilisateur connecté) à un composant feuille situé très bas, obligeant chaque composant intermédiaire à relayer la propriété sans l'utiliser. `useContext` permet au composant cible de "téléporter" la donnée depuis le `Provider` le plus proche sans encombrer les couches intermédiaires.

### 6. Réponse : **B**
**Explication :** 
- Avec `[ ]` vide : s'exécute uniquement au montage.
- Avec `[a, b]` : s'exécute au montage puis si `a` ou `b` change.
- **Sans deuxième argument** : l'effet s'exécute après le montage initial **et après absolument chaque mise à jour** (re-render) du composant.