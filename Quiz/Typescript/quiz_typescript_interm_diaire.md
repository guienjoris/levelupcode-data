# Quiz d'Évaluation — TypeScript Niveau Intermédiaire

Ce quiz évalue votre maîtrise des notions abordées dans la leçon intermédiaire : les unions discriminées, le contrôle d'exhaustivité (`never`), les génériques avec contraintes, les prédicats de types (`is` / `asserts`), les mapped types et les assertions `as const`.

---

### Question 1 : Unions discriminées & Exhaustive Checking
Soit l'union discriminée et la fonction suivantes :

```typescript
type Shape = 
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; size: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.size ** 2;
    default: {
      const _unreachable: never = shape;
      throw new Error("Forme inconnue");
    }
  }
}
```

Si un développeur ajoute `| { kind: 'triangle'; base: number; height: number }` à l'union `Shape` sans modifier le `switch`, que se produit-il à la compilation ?

- [ ] A) Le code compile sans avertissement et lance l'exception uniquement au runtime lors du passage d'un triangle.
- [ ] B) Une erreur de compilation se déclenche sur `const _unreachable: never = shape` car le type restant (`triangle`) n'est pas assignable à `never`.
- [ ] C) TypeScript insère automatiquement un cas vide retournant `0`.
- [ ] D) Le compilateur signale une erreur de doublon de clé sur `kind`.

---

### Question 2 : Génériques avec contraintes & `keyof`
Considérez la signature suivante :

```typescript
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((item) => item[key]);
}

const users = [
  { id: 1, name: 'Alice', active: true },
  { id: 2, name: 'Bob', active: false },
];
```

Quel est le type exact inféré pour la variable `names` ci-dessous ?

```typescript
const names = pluck(users, 'name');
```

- [ ] A) `any[]`
- [ ] B) `(string | number | boolean)[]`
- [ ] C) `string[]`
- [ ] D) `Array<keyof typeof users>`

---

### Question 3 : Prédicats de type personnalisés (*Type Predicates*)
Soit la fonction utilitaire :

```typescript
function isString(val: unknown): val is string {
  return typeof val === 'string';
}
```

Pourquoi est-il nécessaire d'écrire `val is string` comme type de retour plutôt qu'un simple `boolean` ?

- [ ] A) `boolean` n'est pas un type primitif reconnu dans les signatures de fonctions TypeScript.
- [ ] B) Sans `val is string`, TypeScript traite la fonction comme un test booléen standard et n'applique aucun narrowing sur `val` dans les blocs conditionnels qui l'appellent.
- [ ] C) `val is string` empêche la fonction de renvoyer `false`.
- [ ] D) Cela permet d'exécuter la fonction plus rapidement dans le moteur JavaScript.

---

### Question 4 : Assertion Functions (`asserts`)
Quelle est la différence fondamentale entre un prédicat classique (`val is Type`) et une fonction d'assertion (`asserts val is Type`) ?

- [ ] A) Une fonction d'assertion doit obligatoirement être asynchrone (`async`).
- [ ] B) Le prédicat renvoie un booléen et s'utilise dans une condition (`if`), tandis que la fonction d'assertion ne renvoie rien (`void`) et lève une exception en cas d'échec pour rétrécir le type sur les lignes suivantes.
- [ ] C) Les fonctions d'assertion ne fonctionnent qu'avec des classes, pas avec des types primitifs.
- [ ] D) Il n'y a aucune différence, ce sont de stricts synonymes syntaxiques.

---

### Question 5 : Mapped Types & Utility Types
Considérez ce mapped type personnalisé :

```typescript
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};
```

Si on lui passe le type suivant :

```typescript
interface Product {
  title: string;
  price: number;
}
```

Quel est le type équivalent de `Nullable<Product>` ?

- [ ] A) `{ title: null; price: null; }`
- [ ] B) `Product | null`
- [ ] C) `{ title: string | null; price: number | null; }`
- [ ] D) `null`

---

### Question 6 : Const Assertions (`as const`)
Soit le code suivant :

```typescript
const ROLES = ['admin', 'editor', 'viewer'] as const;
type Role = typeof ROLES[number];
```

Quel est le type exact représenté par l'alias `Role` ?

- [ ] A) `string`
- [ ] B) `'admin' | 'editor' | 'viewer'`
- [ ] C) `number`
- [ ] D) `['admin', 'editor', 'viewer']`

---

### Question 7 : Différence entre `Pick<T, K>` et `Omit<T, K>`
Soit l'interface :

```typescript
interface UserAccount {
  id: string;
  email: string;
  passwordHash: string;
  avatarUrl: string;
}
```

Quelle expression produit un type contenant **uniquement** `id`, `email` et `avatarUrl` ?

- [ ] A) `Pick<UserAccount, 'passwordHash'>`
- [ ] B) `Omit<UserAccount, 'id' | 'email' | 'avatarUrl'>`
- [ ] C) `Omit<UserAccount, 'passwordHash'>`
- [ ] D) `Partial<UserAccount, 'passwordHash'>`

---

## Corrigé et Explications Détaillées

<details>
<summary>👉 Cliquez ici pour voir les réponses détaillées</summary>

### 1. Réponse : **B**
**Explication :** Le type `never` représente un ensemble vide de valeurs. Si un nouveau type (`triangle`) est ajouté à l'union sans être pris en charge par l'une des branches `case`, TypeScript fait descendre la variante restante jusqu'au bloc `default`. La tentative d'affectation `const _unreachable: never = shape` échoue car `{ kind: 'triangle', ... }` n'est pas assignable à `never`. Cela permet de garantir à la compilation qu'aucune variante n'a été oubliée.

### 2. Réponse : **C**
**Explication :** Grâce à `K extends keyof T`, en passant la chaîne littérale `'name'`, `K` est résolu en `'name'`. Le type de retour `T[K][]` devient alors `User['name'][]`, ce qui correspond exactement à `string[]`.

### 3. Réponse : **B**
**Explication :** En JavaScript, une fonction qui teste un type renvoie un booléen classique. Sans l'annotation explicite `val is string` (un prédicat de type), TypeScript verrait seulement un retour de type `boolean` et ne saurait pas que `val` peut être restreint à `string` dans le bloc `if (isString(val))`.

### 4. Réponse : **B**
**Explication :** Les fonctions avec `asserts val is T` lèvent une erreur si la vérification échoue. Si la ligne d'appel passe sans exception, TypeScript déduit que pour tout le reste du flux d'exécution en aval, la variable vérifiée est du type cible, évitant d'imbriquer du code dans des blocs `if`.

### 5. Réponse : **C**
**Explication :** Le mapped type parcourt chaque clé `K` de `Product` via `[K in keyof T]` et remplace le type de la valeur associée `T[K]` par `T[K] | null`. Les propriétés `title` et `price` deviennent donc respectivement `string | null` et `number | null`.

### 6. Réponse : **B**
**Explication :** `as const` fige le tableau sous la forme d'un tuple en lecture seule `readonly ["admin", "editor", "viewer"]` au lieu d'élargir le type en `string[]`. Ensuite, l'accès indexé `[number]` extrait l'union de tous les éléments du tableau, soit `'admin' | 'editor' | 'viewer'`.

### 7. Réponse : **C**
**Explication :** `Omit<T, K>` construit un type en prenant toutes les propriétés de `T` et en supprimant celles spécifiées dans `K`. Pour retirer uniquement `passwordHash`, on utilise donc `Omit<UserAccount, 'passwordHash'>`. `Pick` aurait demandé d'énumérer expressément les trois propriétés à conserver.
</details>