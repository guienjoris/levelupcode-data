# Quiz d'Évaluation — Les Fondamentaux de TypeScript (Niveau Junior)

Ce quiz teste votre compréhension des bases de TypeScript : inférence de type, modélisation avec `type` et `interface`, typage des fonctions, *narrowing*, gestion des tuples, et différence essentielle entre `any` et `unknown`.

---

### Question 1 : Inférence de type

Considérez l'instruction suivante :

```typescript
let message = "Bonjour le monde";
message = 100;
```

Que se produit-il à la compilation TypeScript ?

- [ ] A) Le code compile sans erreur car `let` autorise la réassignation d'autres types.
- [ ] B) Une erreur survient : TypeScript a inféré le type `string` lors de l'assignation initiale, rendant l'affectation d'un `number` interdite.
- [ ] C) `message` est automatiquement converti en `any`.
- [ ] D) Le compilateur convertit `100` en chaîne `"100"` à l'exécution.

---

### Question 2 : Tableaux vs Tuples

Quelle est la déclaration correcte pour une variable représentant un point géographique `[latitude, longitude]` composé de deux nombres et strictement limité à ces deux éléments ?

- [ ] A) `const coords: number[] = [48.85, 2.35];`
- [ ] B) `const coords: [number, number] = [48.85, 2.35];`
- [ ] C) `const coords: Array<number, number> = [48.85, 2.35];`
- [ ] D) `const coords: (number, number) = [48.85, 2.35];`

---

### Question 3 : Immutabilité dans les Interfaces

Soit l'interface suivante :

```typescript
interface Profile {
  readonly id: number;
  username: string;
}

const user: Profile = { id: 42, username: "dev_junior" };
user.id = 99;
```

Quel est le résultat produit ?

- [ ] A) `user.id` prend la valeur `99` sans avertissement.
- [ ] B) Une erreur de syntaxe empêche l'exécution JavaScript sur le navigateur.
- [ ] C) Une erreur de compilation TypeScript est levée car `id` est marqué comme `readonly`.
- [ ] D) Le compilateur crée automatiquement un nouvel objet avec `id: 99`.

---

### Question 4 : Type Narrowing (Rétrécissement de type)

Considérez la fonction ci-dessous :

```typescript
function printLength(val: string | string[]): number {
  if (typeof val === "string") {
    return val.length;
  }
  return val.length;
}
```

Dans le second `return`, quel est le type inféré pour `val` par le compilateur ?

- [ ] A) `string | string[]`
- [ ] B) `string[]`
- [ ] C) `any`
- [ ] D) `unknown`

---

### Question 5 : Sécurité de typage : `any` vs `unknown`

Pourquoi la communauté TypeScript recommande-t-elle `unknown` plutôt que `any` pour typer des données externes dont la forme exacte n'est pas connue à l'avance ?

- [ ] A) `unknown` est plus rapide à exécuter dans le moteur V8.
- [ ] B) `any` empêche la compilation en JavaScript natif.
- [ ] C) `unknown` interdit tout appel de méthode ou accès à une propriété tant que le développeur n'a pas vérifié le type par du code défensif (*narrowing*).
- [ ] D) `unknown` force automatiquement les valeurs à être sérialisées en JSON.

---

### Question 6 : Typage des Fonctions sans retour

Quelle annotation de retour de fonction doit-on utiliser pour indiquer explicitement qu'une fonction n'a pas vocation à renvoyer de valeur exploitable ?

- [ ] A) `null`
- [ ] B) `never`
- [ ] C) `void`
- [ ] D) `undefined`

---

### Question 7 : Introduction aux Génériques

Soit la fonction générique :

```typescript
function getFirstElement<T>(items: T[]): T | undefined {
  return items[0];
}

const result = getFirstElement(["chat", "chien", "oiseau"]);
```

Quel est le type exact déduit pour la variable `result` ?

- [ ] A) `any`
- [ ] B) `string`
- [ ] C) `string | undefined`
- [ ] D) `T[]`

---

## Corrigé et Explications

<details>
<summary>👉 Afficher les réponses commentées</summary>

### 1. Réponse : **B**
**Explication :** TypeScript applique l'inférence de type lors de l'initialisation de variables simples. Initialiser `message` avec une chaîne de caractères fige son type en `string`. Réassigner un nombre par la suite déclenche une erreur statique `Type 'number' is not assignable to type 'string'`.

### 2. Réponse : **B**
**Explication :** `[number, number]` définit un **tuple**, c'est-à-dire un tableau avec un nombre fixe d'éléments aux types spécifiés par position. `number[]` représente quant à lui une liste de longueur arbitraire, qui accepterait l'ajout ultérieur d'autres éléments.

### 3. Réponse : **C**
**Explication :** Le modificateur `readonly` est une contrainte au niveau de la compilation TypeScript. Le compilateur refuse toute tentative de réaffectation directe sur la propriété après la création de l'objet.

### 4. Réponse : **B**
**Explication :** C'est le principe du *type narrowing* par élimination (analyse de flot de contrôle). Sachant que le type initial est `string | string[]` et que la première branche filtre les cas où `typeof val === "string"`, TypeScript en déduit de façon certaine que `val` ne peut être que de type `string[]` dans la suite du corps de la fonction.

### 5. Réponse : **C**
**Explication :** `any` désactive complètement la vérification du compilateur et autorise des appels dangereux au risque de causer des exceptions à l'exécution. Au contraire, `unknown` oblige à inspecter ou typer la donnée (via `typeof`, `instanceof`, ou des *type guards*) avant de pouvoir utiliser ses méthodes.

### 6. Réponse : **C**
**Explication :** Le type `void` indique l'absence de retour intentionnel d'une fonction (généralement utilisée pour des fonctions à effet de bord comme `console.log`). `never` est réservé aux fonctions qui ne terminent jamais normalement (boucle infinie ou exception systématiquement levée).

### 7. Réponse : **C**
**Explication :** Le paramètre de type `T` est résolu en `string` car la fonction reçoit un tableau `string[]`. La signature renvoyant `T | undefined`, le type de retour exact est donc `string | undefined` (pour tenir compte du cas où le tableau serait vide).
</details>