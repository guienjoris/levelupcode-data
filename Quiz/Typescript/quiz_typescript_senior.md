# Quiz d'Évaluation — TypeScript Niveau Senior & Type System Architect

Ce quiz teste votre compréhension approfondie des mécanismes du compilateur TypeScript : métatypage, types conditionnels, inférence dynamique, distributivité, variance et typage nominal simulé.

---

### Question 1 : Distributivité des Types Conditionnels

Considérez le type conditionnel suivant :

```typescript
type ToArray<T> = T extends any ? T[] : never;
type NonDistributiveToArray<T> = [T] extends [any] ? T[] : never;

type ResA = ToArray<string | number>;
type ResB = NonDistributiveToArray<string | number>;
```

Quels sont les types exacts résultants pour `ResA` et `ResB` ?

- [ ] A) `ResA = (string | number)[]` et `ResB = string[] | number[]`
- [ ] B) `ResA = string[] | number[]` et `ResB = (string | number)[]`
- [ ] C) `ResA = (string | number)[]` et `ResB = (string | number)[]`
- [ ] D) `ResA = never` et `ResB = (string | number)[]`

---

### Question 2 : Inférence avec `infer` et Déballage Récursif

Soit la définition de type suivante :

```typescript
type UnwrapPromise<T> = T extends Promise<infer U> ? UnwrapPromise<U> : T;

type Test = UnwrapPromise<Promise<Promise<number>>>;
```

Quel est le type de `Test` et que se produit-il si on lui passe `string` ?

- [ ] A) `Test = Promise<number>` ; si on lui passe `string`, cela renvoie `never`.
- [ ] B) `Test = number` ; si on lui passe `string`, cela renvoie `string`.
- [ ] C) `Test = Promise<Promise<number>>` ; si on lui passe `string`, cela lève une erreur de compilation.
- [ ] D) `Test = unknown` ; si on lui passe `string`, cela renvoie `unknown`.

---

### Question 3 : Variance des Fonctions (`strictFunctionTypes`)

Considérez la hiérarchie de classes suivante :

```typescript
class Animal { name!: string; }
class Dog extends Animal { bark() {} }

type Handler<T> = (arg: T) => void;
```

Lorsque l'option `"strictFunctionTypes": true` est activée dans `tsconfig.json`, quelle affectation est valide et quelle règle de variance s'applique ?

```typescript
let fAnimal: Handler<Animal> = (a: Animal) => console.log(a.name);
let fDog: Handler<Dog> = (d: Dog) => d.bark();
```

- [ ] A) `fAnimal = fDog` est valide (les arguments de fonctions sont covariants).
- [ ] B) `fDog = fAnimal` est valide (les arguments de fonctions sont contravariants).
- [ ] C) Les deux affectations sont valides (les arguments de fonctions sont bivariants).
- [ ] D) Aucune affectation n'est valide (les arguments de fonctions sont invariants).

---

### Question 4 : Branded Types (Typage Nominal)

Pourquoi utilise-t-on le pattern des *Branded Types* (ou *Nominal Typing*) dans les architectures complexes ?

```typescript
type UserId = string & { readonly __brand: unique symbol };
type OrderId = string & { readonly __brand: unique symbol };
```

- [ ] A) Pour forcer le moteur JavaScript à allouer un nouvel espace mémoire dédié pour chaque ID à l'exécution.
- [ ] B) Pour empêcher l'interchangeabilité accidentelle de valeurs de même type structurel sous-jacent (ex: passer un `OrderId` là où un `UserId` est attendu).
- [ ] C) Pour permettre la sérialisation binaire automatique des identifiants dans IndexedDB.
- [ ] D) Pour chiffrer la valeur de la variable au moment de la compilation.

---

### Question 5 : Template Literal Types & Manipulation de Chaînes

Observez le type suivant :

```typescript
type CamelCase<S extends string> = S extends `${infer Head}_${infer Tail}`
  ? `${Head}${Capitalize<CamelCase<Tail>>}`
  : S;

type Output = CamelCase<"user_first_name">;
```

Quel est le résultat exact produit pour `Output` ?

- [ ] A) `"userFirstName"`
- [ ] B) `"UserFirstName"`
- [ ] C) `"user_first_name"`
- [ ] D) `"user_First_Name"`

---

### Question 6 : Filtrage via Key Remapping (`as`)

Soit le type mappé suivant :

```typescript
type OnlyFunctions<T> = {
  [K in keyof T as T[K] extends (...args: any[]) => any ? K : never]: T[K];
};

interface Service {
  id: string;
  count: number;
  start(): void;
  stop(force: boolean): Promise<void>;
}

type ServiceMethods = OnlyFunctions<Service>;
```

Quel est le contenu de `ServiceMethods` ?

- [ ] A) `{ id: never; count: never; start: () => void; stop: (force: boolean) => Promise<void>; }`
- [ ] B) `{ start: () => void; stop: (force: boolean) => Promise<void>; }`
- [ ] C) `keyof Service`
- [ ] D) `{ id: string; count: number; }`

---

### Question 7 : Inférence dans les Tuples et Opérateur `extends`

Soit le type utilitaire :

```typescript
type Shift<T extends any[]> = T extends [any, ...infer Rest] ? Rest : [];

type Result = Shift<[number, string, boolean]>;
```

Quel est le type de `Result` ?

- [ ] A) `[number, string]`
- [ ] B) `[string, boolean]`
- [ ] C) `boolean`
- [ ] D) `(string | boolean)[]`

---

## Corrigé et Explications Détaillées

<details>
<summary>👉 Cliquez ici pour afficher les solutions détaillées</summary>

### 1. Réponse : **B**
**Explication :** En TypeScript, lorsqu'un paramètre générique nu `T` (sans emballage dans un tuple) est utilisé à gauche de `extends` dans un type conditionnel, l'opération est **distributive** sur les unions : `(A | B) extends any ? ...` devient `(A extends any ? ...) | (B extends any ? ...)`. Ainsi, `ToArray<string | number>` produit `string[] | number[]`. Envelopper le type dans un tuple (`[T] extends [any]`) désactive la distributivité, produisant `(string | number)[]`.

### 2. Réponse : **B**
**Explication :** `UnwrapPromise` est un type conditionnel récursif. Tant que le type extrait par `infer U` correspond à une `Promise`, il continue de déballer le contenu. Ainsi, `Promise<Promise<number>>` devient `number`. Si la valeur n'est pas une `Promise` (branche `false`), la condition renvoie `T` inchangé : passer `string` renvoie donc `string`.

### 3. Réponse : **B**
**Explication :** Sous le flag `"strictFunctionTypes": true`, les paramètres de fonctions sont **contravariants** (leur relation de sous-typage s'inverse). Puisque `Dog` hérite de `Animal` (`Dog` $\subseteq$ `Animal`), une fonction qui sait traiter n'importe quel `Animal` (`fAnimal`) est capable de traiter un `Dog` en toute sécurité : on peut donc assigner `fDog = fAnimal`. L'inverse est dangereux : `fDog` s'attend à pouvoir appeler `.bark()`, ce qu'un `Animal` générique ne possède pas.

### 4. Réponse : **B**
**Explication :** TypeScript repose sur un système de types structurel (*duck typing*). Deux types de même structure (`string` et `string`) sont interchangeables par défaut. L'ajout d'une marque fantôme à la compilation via un symbole unique empêche d'assigner par mégarde un identifiant d'un type à un autre, sans générer aucun surcoût ou code à l'exécution.

### 5. Réponse : **A**
**Explication :** 
1. Premier passage : `Head` capture `"user"`, `Tail` capture `"first_name"`.
2. Appel récursif sur `"first_name"` : `Head` capture `"first"`, `Tail` capture `"name"`.
3. Appel sur `"name"` : pas de `_`, renvoie `"name"`.
4. Recomposition : `"first"` + `Capitalize<"name">` $\rightarrow$ `"firstName"`.
5. Résultat final : `"user"` + `Capitalize<"firstName">` $\rightarrow$ `"userFirstName"`.

### 6. Réponse : **B**
**Explication :** Le *Key Remapping* avec `as` permet de filtrer les clés en renvoyant `never`. Lorsqu'une clé est remappée vers `never`, TypeScript l'exclut complètement du type résultant au lieu de lui assigner la valeur `never`. Les clés `id` et `count` sont donc omises, ne laissant que `start` et `stop`.

### 7. Réponse : **B**
**Explication :** L'inférence sur les tuples via le rest parameter `[any, ...infer Rest]` permet d'isoler la tête (*head*) du reste du tuple (*tail*). Ici, `number` est le premier élément consommé, et `Rest` capture le tuple résiduel sous forme exacte `[string, boolean]`.
</details>