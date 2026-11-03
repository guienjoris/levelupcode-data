# Quiz d'Évaluation — Pratique TypeScript Débutant

Ce quiz évalue votre capacité à exploiter l'inférence, à manipuler des unions de types, à sécuriser des fonctions et à choisir le bon niveau d'annotation.

### Question 1 : Inférence de variable primitive

Soit l'instruction TypeScript suivante :
`const seuilAlerte = 50;`
Quel type précis le compilateur attribue-t-il à la constante `seuilAlerte` ?

* [ ] A) Le type littéral `50` car une constante primitive ne peut plus changer de valeur.
* [ ] B) Le type générique `number` car c'est un chiffre entier non signé standard.
* [ ] C) Le type `any` en l'absence d'une annotation explicite de type `: number`.
* [ ] D) Le type `unknown` jusqu'à sa première lecture dans le code source.

### Question 2 : Traitement d'un type Union avec `typeof`

Pourquoi le compilateur TypeScript refuse-t-il l'accès direct `valeur.length` si `valeur` est typée `string | number` ?

* [ ] A) Parce que TypeScript exige que toutes les variables soient converties en objets au préalable.
* [ ] B) Parce que la propriété `length` n'existe pas sur le type `number` et pourrait faire planter l'exécution.
* [ ] C) Parce que les types unions interdisent l'accès à toute propriété tant qu'un casting `as any` n'est pas fait.
* [ ] D) Parce que la méthode `length` est considérée comme dépréciée depuis ECMAScript 2020.

### Question 3 : Valeur par défaut d'une propriété optionnelle

Dans une interface définie comme suit :

```typescript
interface Utilisateur {
  nom: string;
  telephone?: string;
}
```

Quelle valeur la propriété `telephone` possède-t-elle à l'exécution si elle n'est pas renseignée lors de l'instanciation de l'objet ?

* [ ] A) Une chaîne de caractères vide `""`.
* [ ] B) La valeur spéciale `null`.
* [ ] C) La valeur primitive `undefined`.
* [ ] D) Le nombre zéro `0`.

### Question 4 : Propriétés en lecture seule

Que se passe-t-il si vous tentez d'exécuter l'instruction `produit.id = 12;` alors que la propriété a été déclarée `readonly id: number;` dans son interface ?

* [ ] A) Le code JavaScript généré lève une exception native au moment de l'exécution sur le navigateur.
* [ ] B) Le compilateur TypeScript signale une erreur de compilation et refuse la réassignation.
* [ ] C) La propriété est clonée sous un nouveau nom d'attribut pour préserver l'ancienne valeur.
* [ ] D) La valeur n'est modifiée que si le projet tourne avec les options du mode strict désactivées.

### Question 5 : Rôle du mot-clé `void` pour une fonction

Dans la signature d'une fonction, qu'indique précisément l'annotation de retour `: void` ?

* [ ] A) Que la fonction ne termine jamais son exécution car elle contient une boucle sans fin.
* [ ] B) Que la fonction ne retourne pas de valeur utile (elle renvoie implicitement `undefined`).
* [ ] C) Que la fonction attend obligatoirement un callback asynchrone avant de pouvoir se fermer.
* [ ] D) Que la fonction retourne systématiquement une promesse vide de type `Promise<null>`.

### Question 6 : Sécurité entre `unknown` et `any`

Quelle est la différence fondamentale entre une variable déclarée avec le type `unknown` et une variable déclarée avec le type `any` ?

* [ ] A) `any` est réservé aux tableaux tandis que `unknown` s'applique exclusivement aux objets JSON.
* [ ] B) `unknown` impose de vérifier ou valider le type avant toute manipulation, alors que `any` désactive toutes les vérifications.
* [ ] C) `any` permet de compiler plus vite car le compilateur convertit directement la variable en code machine binaire.
* [ ] D) `unknown` interdit définitivement d'effectuer des réassignations de variables au cours du programme.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** Pour une variable déclarée avec `let`, TypeScript infère `number` car sa valeur est susceptible de changer. Mais pour une déclaration avec `const`, la valeur ne pourra plus jamais être modifiée : TypeScript déduit donc le type littéral exact `50`, ce qui est plus précis que `number`.

### 2. Réponse : **B**
**Explication :** Lorsqu'une variable est typée sous forme d'union (`string | number`), TypeScript n'autorise par défaut que les membres communs à tous les types de l'union. Comme `length` n'existe que sur les chaînes de caractères et pas sur les nombres, TypeScript bloque l'accès pour prévenir un crash `undefined`. Il faut faire un rétrécissement préalable (ex. `if (typeof valeur === "string")`).

### 3. Réponse : **C**
**Explication :** En JavaScript et TypeScript, lorsqu'une propriété facultative (marquée du marqueur `?`) est omise dans un objet littéral, sa valeur n'est pas initialisée et vaut donc par nature `undefined`.

### 4. Réponse : **B**
**Explication :** Le modifieur `readonly` est une contrainte statique appliquée par le compilateur TypeScript au moment de l'analyse du code. Il génère une erreur de compilation s'il détecte une tentative de réécriture. En revanche, à l'exécution en JavaScript pur, `readonly` disparaît (à moins d'utiliser `Object.freeze()`).

### 5. Réponse : **B**
**Explication :** Le type de retour `void` signifie que la fonction exécute une action (effet de bord, affichage console, mutation externe) mais ne produit pas de valeur en sortie. En JavaScript, une fonction sans instruction `return` explicite renvoie la valeur `undefined`.

### 6. Réponse : **B**
**Explication :** `any` agit comme une neutralisation du vérificateur de types : vous pouvez appeler n'importe quelle méthode inexistante sans alerte. À l'inverse, `unknown` représente une valeur dont on ne connaît pas encore la nature : TypeScript empêche tout appel de méthode ou toute opération tant que l'on n'a pas vérifié son type avec un garde (`typeof`, `instanceof`, etc.).