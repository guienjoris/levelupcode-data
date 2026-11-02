
# Quiz d'Évaluation — Pratique Moderne JavaScript

Ce quiz évalue votre compréhension de la déstructuration, des méthodes de tableau immutables, des opérateurs de reste/propagation et des gardes de sécurité `?.` / `??`.

### Question 1 : Chaînage optionnel et gestion des erreurs

Que retourne l'expression `utilisateur.profil?.avatar?.url` si la propriété `profil` de l'objet `utilisateur` vaut `null` ?

* [ ] A) Elle déclenche immédiatement une exception non rattrapable de type `TypeError`.
* [ ] B) La valeur `undefined` est renvoyée de manière sécurisée sans interrompre le script.
* [ ] C) Elle retourne la chaîne de caractères littérale `"/assets/default-avatar.png"`.
* [ ] D) L'interpréteur renvoie la valeur booléenne `false`.

### Question 2 : Choix entre `||` et `??`

Dans quelle situation l'opérateur de coalescence des nuls `??` donne-t-il un résultat différent de l'opérateur logique `||` ?

* [ ] A) Lorsque la variable testée à gauche contient la valeur `undefined`.
* [ ] B) Quand la variable située à gauche contient le nombre `0` ou une chaîne vide `""`.
* [ ] C) Dès lors que la valeur située à gauche de l'opérateur contient la valeur spéciale `null`.
* [ ] D) Si les deux opérandes de l'expression sont strictement de type booléen `true`.

### Question 3 : Comportement de la méthode `map()`

Quelle est la garantie fondamentale offerte par la méthode native `Array.prototype.map()` ?

* [ ] A) Elle modifie directement le tableau d'origine sur place (*in-place*) pour économiser la mémoire vive.
* [ ] B) Elle renvoie un nouveau tableau de même dimension sans altérer le tableau source.
* [ ] C) Elle supprime automatiquement les éléments qui renvoient la valeur booléenne `false`.
* [ ] D) Elle trie les éléments par ordre alphabétique croissant avant d'appliquer le callback.

### Question 4 : Déstructuration avec renommage de propriété

Soit l'objet suivant : `const produit = { ref: "A12", prixHT: 100 };`.  
Quelle syntaxe extrait la valeur de `ref` dans une nouvelle variable nommée `referenceProduit` ?

* [ ] A) `const { ref: referenceProduit } = produit;`
* [ ] B) `const { referenceProduit = ref } = produit;`
* [ ] C) `const { ref -> referenceProduit } = produit;`
* [ ] D) `const [ ref as referenceProduit ] = produit;`

### Question 5 : L'opérateur Spread (`...`) sur les objets

Quel est le résultat de l'exécution du code suivant ?
```javascript
const base = { theme: 'dark', taille: 14 };
const custom = { ...base, taille: 18, theme: 'light' };
```

* [ ] A) Une erreur de syntaxe car un objet ne peut pas contenir deux fois la même clé dans une déclaration.
* [ ] B) `{ theme: 'dark', taille: 14 }` car les propriétés initiales de l'objet `base` sont immuables.
* [ ] C) `{ theme: 'light', taille: 18 }` car les propriétés spécifiées après la décomposition écrasent les précédentes.
* [ ] D) `{ theme: ['dark', 'light'], taille: [14, 18] }` sous la forme de tableaux fusionnés.

### Question 6 : Le rôle du paramètre Rest

Quelle affirmation caractérise correctement l'utilisation des paramètres de reste (`...args`) dans la signature d'une fonction ?

* [ ] A) Il s'agit d'un objet littéral qui ne peut recevoir que des couples clé-valeur numériques.
* [ ] B) Il collecte l'ensemble des arguments excédentaires dans une véritable instance d'Array JavaScript.
* [ ] C) Il doit impérativement être placé comme premier argument dans la déclaration de la fonction.
* [ ] D) Il convertit automatiquement toutes les entrées textuelles en entiers numériques signés.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** L'opérateur de chaînage optionnel `?.` vérifie la référence située immédiatement à sa gauche. Si elle vaut `null` ou `undefined`, l'évaluation s'interrompt court-circuitée et renvoie `undefined`, évitant ainsi le crash `TypeError: Cannot read properties of null`.

### 2. Réponse : **B**
**Explication :** L'opérateur logique `||` bascule sur la valeur de droite dès que l'opérande gauche est "falsy" (`0`, `""`, `false`, `NaN`, `null`, `undefined`). L'opérateur `??` ne bascule que si la valeur est "nullish" (`null` ou `undefined`). Pour un score de `0` ou un champ texte vide `""`, `??` conserve la valeur de gauche tandis que `||` applique la valeur par défaut.

### 3. Réponse : **B**
**Explication :** `map()` est une fonction pure : elle ne mute pas le tableau initial et produit systématiquement un nouveau tableau contenant exactement le même nombre d'éléments, chacun résultant de l'appel de la fonction de rappel fournie.

### 4. Réponse : **A**
**Explication :** En déstructuration d'objet, la syntaxe `{ cleOriginale: nouveauNomDeVariable }` permet de cibler la propriété existante et de lui associer un nouvel identifiant dans la portée courante.

### 5. Réponse : **C**
**Explication :** Le spread operator déverse les propriétés de `base` de gauche à droite. Lorsqu'une clé identique (`taille` puis `theme`) est redéfinie plus loin dans l'objet littéral, sa valeur remplace la valeur précédente.

### 6. Réponse : **B**
**Explication :** Le paramètre Rest rassemble un nombre indéterminé d'arguments sous la forme d'un tableau standard (`Array`), permettant l'utilisation directe de méthodes telles que `.map()`, `.filter()` ou `.reduce()`. Il doit impérativement se trouver en dernière position dans la liste des paramètres.