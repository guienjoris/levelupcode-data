# Les Fondamentaux de TypeScript (Niveau Junior)

TypeScript est un **sur-ensemble typé de JavaScript** (*superset*). Il ajoute un système de types statiques analysé à la compilation, qui disparaît totalement à l'exécution pour produire du JavaScript pur.

---

## 1. Pourquoi TypeScript ?

En JavaScript standard, les erreurs de types surviennent au moment de l'exécution (*runtime*) :

```javascript
// JavaScript standard
function calculateTotal(price, quantity) {
  return price * quantity;
}

calculateTotal("15€", 2); // Renvoie NaN sans lever d'erreur immédiate !
```

En TypeScript, le compilateur détecte l'erreur dès l'écriture du code (*compile-time*) :

```typescript
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

calculateTotal("15€", 2);
// ❌ Erreur TypeScript : Argument of type 'string' is not assignable to parameter of type 'number'.
```

---

## 2. Les Types Primitifs et l'Inférence de Type

### 2.1 Les types de base

TypeScript dispose des mêmes types primitifs que JavaScript :

```typescript
const username: string = "Alice";
const age: number = 28;
const isAdmin: boolean = false;
const emptyValue: null = null;
const notAssigned: undefined = undefined;
```

### 2.2 L'Inférence de type (*Type Inference*)

Il n'est pas nécessaire de tout annoter manuellement. Si une variable est initialisée immédiatement, TypeScript devine son type :

```typescript
// TypeScript infère automatiquement le type 'string'
let city = "Paris";

city = 42; 
// ❌ Erreur : Type 'number' is not assignable to type 'string'.
```

> **Bonne pratique Junior :** Laissez TypeScript inférer les types simples. Annotez explicitement les paramètres de fonctions, les retours complexes et les structures d'objets.

---

## 3. Typage des Tableaux et Tuples

### 3.1 Tableaux

Deux syntaxes équivalentes existent pour déclarer des listes d'éléments de même type :

```typescript
const scores: number[] = [10, 20, 15];
const tags: Array<string> = ["typescript", "frontend"];

scores.push(25);   // ✅ Valide
scores.push("30"); // ❌ Erreur : 'string' n'est pas permis dans un 'number[]'
```

### 3.2 Tuples

Un tuple est un tableau de longueur fixe dont chaque position a un type précis :

```typescript
// Coordonnées [latitude, longitude]
const coordinates: [number, number] = [48.8566, 2.3522];

// Réponse HTTP : [code, message]
const statusResponse: [number, string] = [200, "OK"];
```

---

## 4. Structurer ses Données : `type` vs `interface`

Pour typer des objets, TypeScript propose deux constructions principales : les **alias de type** (`type`) et les **interfaces** (`interface`).

### 4.1 L'`interface` (idéale pour les objets et l'orienté objet)

```typescript
interface User {
  readonly id: number;      // 'readonly' : non modifiable après création
  name: string;
  email: string;
  avatarUrl?: string;       // '?' : propriété optionnelle
}

const alice: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
};

// alice.id = 2; // ❌ Erreur : Cannot assign to 'id' because it is a read-only property.
```

### 4.2 L'alias `type` (plus flexible)

L'alias de type peut définir des objets, mais aussi des types d'union ou des types primitifs :

```typescript
type ID = string | number;

type Point = {
  x: number;
  y: number;
};
```

### 4.3 Extension

Les deux peuvent s'étendre :

```typescript
// Extension d'interface (mot-clé 'extends')
interface AdminUser extends User {
  permissions: string[];
}

// Intersection de types (opérateur '&')
type Employee = User & {
  salary: number;
};
```

---

## 5. Typage des Fonctions

TypeScript vérifie à la fois les arguments passés et la valeur retournée.

```typescript
// Paramètre optionnel avec '?'
function greet(name: string, title?: string): string {
  if (title) {
    return `Bonjour, ${title} ${name}`;
  }
  return `Bonjour, ${name}`;
}

// Paramètre avec valeur par défaut
function createUrl(path: string, protocol: string = "https"): string {
  return `${protocol}://${path}`;
}

// Fonction sans retour utile : type 'void'
function logMessage(message: string): void {
  console.log(`[LOG] : ${message}`);
}
```

---

## 6. Types d'Union et *Type Narrowing*

### 6.1 Les Unions (`|`)

Une union permet à une variable d'accepter plusieurs types distincts :

```typescript
type Status = "idle" | "loading" | "success" | "error";

let currentStatus: Status = "idle";
currentStatus = "loading"; // ✅ Valide
// currentStatus = "deleted"; // ❌ Erreur : non autorisé par le type Status
```

### 6.2 Le rétrécissement de type (*Type Narrowing*)

Quand une variable a un type d'union, TypeScript vous oblige à vérifier son type réel avant d'utiliser des méthodes spécifiques :

```typescript
function formatInput(input: string | number): string {
  // Ici, 'input' est string | number
  
  if (typeof input === "number") {
    // TypeScript comprend qu'ici 'input' est obligatoirement un number
    return input.toFixed(2);
  }

  // Ici, TypeScript sait qu'il s'agit obligatoirement d'une string
  return input.trim().toUpperCase();
}
```

---

## 7. `any` vs `unknown`

* **`any`** : Désactive complètement le vérificateur de types de TypeScript. C'est une porte de sortie à éviter impérativement dans un projet moderne.
* **`unknown`** : Indique une valeur de type inconnu (ex: réponse d'une API externe). Contrairement à `any`, TypeScript bloque toute action tant que le type n'a pas été explicitement vérifié.

```typescript
// Dangereux : aucun contrôle
let badData: any = "salut";
badData.nonExistentMethod(); // Compile sans erreur, mais plante à l'exécution !

// Sécurisé : contrôle obligatoire
let safeData: unknown = "salut";

// safeData.toUpperCase(); 
// ❌ Erreur : 'safeData' is of type 'unknown'.

if (typeof safeData === "string") {
  console.log(safeData.toUpperCase()); // ✅ Valide et sécurisé
}
```

---

## 8. Introduction aux Génériques (*Generics*)

Les génériques permettent d'écrire du code réutilisable tout en conservant le typage exact de la valeur fournie au lieu de perdre l'information avec `any`.

```typescript
// Le type <T> capture dynamiquement le type passé en argument
function wrapInArray<T>(item: T): T[] {
  return [item];
}

const numbers = wrapInArray(42);       // Type inféré : number[]
const words = wrapInArray("bonjour");  // Type inféré : string[]
```

---

## Résumé des Bonnes Pratiques

1. **Activez le mode strict** : Dans votre `tsconfig.json`, conservez `"strict": true`.
2. **Bannissez `any`** : Préférez `unknown` couplé à un contrôle de type (*narrowing*).
3. **Exploitez l'inférence** : Ne surchargez pas votre code avec des annotations évidentes (`let n: number = 5` est redondant avec `let n = 5`).
4. **Modélisez les états métier avec des unions de chaînes littérales** (`"pending" | "success" | "failed"` plutôt qu'un `string` générique).