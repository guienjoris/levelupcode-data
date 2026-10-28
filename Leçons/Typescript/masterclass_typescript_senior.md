# Masterclass TypeScript — Niveau Senior & Type System Architect

À ce niveau, TypeScript n'est plus simplement un outil de vérification de signature ou d'autocomplétion : il s'agit d'un **langage fonctionnel pur de programmation de types**, exécuté au moment de la compilation par le moteur d'analyse statique. 

Cette leçon explore les mécanismes indispensables à la conception de bibliothèques strictes, d'infrastructures sans compromis de typage, et d'architectures pilotées par le domaine (*Type-Driven Design*).

---

## 1. Types Conditionnels et Inférence Avancée (`infer`)

Les types conditionnels permettent d'écrire des embranchements de logique au niveau des types :

$$T \text{ extends } U \ ? \ X : Y$$

### 1.1 Le mot-clé `infer` : Extraction de types à chaud

Le mot-clé `infer` permet de déclarer une variable de type au sein d'une clause `extends`, laissant TypeScript déduire sa valeur exacte par déstructuration de signature.

#### Exemple 1 : Déballer une Promesse récursivement (équivalent à `Awaited<T>`)

```typescript
export type DeepAwaited<T> = T extends Promise<infer Inner>
  ? DeepAwaited<Inner>
  : T;

type Test1 = DeepAwaited<Promise<Promise<number>>>; // number
type Test2 = DeepAwaited<string>;                   // string
```

#### Exemple 2 : Extraction fine des arguments ou du type de retour

```typescript
// Extraire le premier argument d'une fonction
type FirstParameter<T> = T extends (first: infer P, ...args: any[]) => any
  ? P
  : never;

// Extraire le type résolu d'un générateur ou flux
type GeneratorYield<T> = T extends Generator<infer Y, any, any> ? Y : never;
```

---

## 2. Distributivité des Types Conditionnels & Désactivation

Par défaut, lorsqu'un type conditionnel reçoit un paramètre générique qui est une **union nue** (*naked type parameter*), TypeScript applique la distributivité :

$$(\text{A} \mid \text{B}) \text{ extends } U \ ? \ X : Y \iff (\text{A extends } U \ ? \ X : Y) \mid (\text{B extends } U \ ? \ X : Y)$$

### 2.1 Le piège de la distributivité avec `never`

`never` représente l'union vide. Par conséquent, appliquer un type conditionnel distributif sur `never` court-circuite l'évaluation et retourne immédiatement `never` :

```typescript
type IsNeverDistributive<T> = T extends never ? true : false;

type TestNever = IsNeverDistributive<never>; // Résultat : never (et non pas true !)
```

### 2.2 Désactiver la distributivité avec des tuples `[T]`

Pour traiter une union comme un tout unitaire (sans la distribuer branche par branche), on enveloppe le type et la cible dans un tuple `[T]` :

```typescript
// Évaluation stricte sans distributivité
export type IsNever<T> = [T] extends [never] ? true : false;

type Res1 = IsNever<never>;       // true !
type Res2 = IsNever<string>;      // false

// Vérifier si un type est strictement 'any'
// (any est le seul type qui satisfait [1] extends [2] dans un conditionnel)
export type IsAny<T> = 0 extends (1 & T) ? true : false;
```

---

## 3. Variance des Fonctions : Covariance, Contravariance et Bivariance

Comprendre la variance est fondamental pour architecturer des interfaces d'événements, des conteneurs IoC et des collections génériques.

| Concept | Définition | Où le trouve-t-on en TypeScript ? |
| :--- | :--- | :--- |
| **Covariance** | Préserve la hiérarchie : si $A \subseteq B$, alors $F(A) \subseteq F(B)$ | Types de retour des fonctions, valeurs des objets |
| **Contravariance** | Inverse la hiérarchie : si $A \subseteq B$, alors $F(B) \subseteq F(A)$ | Types des paramètres de fonction (avec `strictFunctionTypes: true`) |
| **Bivariance** | Accepte les deux directions | Méthodes d'interfaces (syntaxe `method(): void`), par compatibilité legacy |
| **Invariance** | N'accepte que l'identité exacte : $A = B$ | Types en lecture-écriture mutables combinés |

```typescript
class Animal { name!: string; }
class Dog extends Animal { bark() {} }

type Producer<T> = () => T;
type Consumer<T> = (param: T) => void;

let produceAnimal!: Producer<Animal>;
let produceDog!: Producer<Dog>;

// Covariance : un chien EST un animal.
produceAnimal = produceDog; // ✅ Valide
// produceDog = produceAnimal; // ❌ Erreur

let consumeAnimal!: Consumer<Animal>;
let consumeDog!: Consumer<Dog>;

// Contravariance : une fonction sachant traiter N'IMPORTE QUEL animal 
// peut sans danger remplacer une fonction qui ne traite que des chiens.
consumeDog = consumeAnimal; // ✅ Valide
// consumeAnimal = consumeDog; // ❌ Erreur : un Animal générique n'a pas forcément de méthode bark() !
```

> **Règle Senior :** Déclarez toujours les fonctions dans les interfaces sous forme de propriétés avec signatures de fonctions (`fn: (arg: T) => void`) plutôt que sous forme de méthodes (`fn(arg: T): void`) pour forcer le compilateur à appliquer la **contravariance stricte**.

---

## 4. Types Récursifs et Manipulation des Template Literals

Depuis TypeScript 4.1+, le typage des chaînes de caractères littérales est complet grâce à la déstructuration par motif.

### 4.1 Typage des chemins d'objets imbriqués (*Deep KeyPaths*)

Ce pattern permet d'obtenir une autocomplétion absolue et sans typage lâche sur des chemins profonds comme `"user.address.street"` :

```typescript
type Primitive = string | number | boolean | symbol | null | undefined | bigint;

export type Path<T> = T extends Primitive
  ? never
  : {
      [K in keyof T & string]: T[K] extends Primitive
        ? `${K}`
        : `${K}` | `${K}.${Path<T[K]>}`;
    }[keyof T & string];

// Extraction de la valeur pointée par un KeyPath
export type PathValue<T, P extends string> = P extends `${infer Head}.${infer Tail}`
  ? Head extends keyof T
    ? PathValue<T[Head], Tail>
    : never
  : P extends keyof T
  ? T[P]
  : never;

// Utilisation
interface SystemConfig {
  server: {
    host: string;
    port: number;
    tls: {
      enabled: boolean;
      certPath: string;
    };
  };
}

function getDeepValue<T, P extends Path<T>>(target: T, path: P): PathValue<T, P> {
  const segments = path.split('.');
  return segments.reduce((acc: any, key) => acc[key], target);
}

const config: SystemConfig = {
  server: { host: '127.0.0.1', port: 8080, tls: { enabled: true, certPath: '/var/ssl' } }
};

// Autocomplétion stricte et inférence de type :
const isTls = getDeepValue(config, 'server.tls.enabled'); // Type inféré : boolean
```

---

## 5. Branded Types (Nominal Typing dans un Système Structurel)

TypeScript utilise un **système de types structurel** (*Duck Typing*) : deux types avec des propriétés identiques sont interchangeables. Cela peut mener à des bugs graves dans le domaine métier (mélanger un `UserId` et un `OrderId`, ou passer des euros à une fonction attendant des centimes).

Les **Branded Types** (ou types opaques) imposent un typage nominal artificiel via un symbole caché ou une marque unique.

```typescript
declare const __brand: unique symbol;

export type Brand<T, B extends string> = T & { readonly [__brand]: B };

// Types du domaine métier
export type UserId = Brand<string, 'UserId'>;
export type OrderId = Brand<string, 'OrderId'>;
export type Cents = Brand<number, 'Cents'>;
export type Euros = Brand<number, 'Euros'>;

// Constructeurs / Smart Constructors (avec validation)
export function toUserId(raw: string): UserId {
  if (!raw.startsWith('usr_')) {
    throw new Error(`ID utilisateur invalide : ${raw}`);
  }
  return raw as UserId;
}

export function toEuros(cents: Cents): Euros {
  return ((cents as number) / 100) as Euros;
}

// Sécurité garantie à la compilation :
let uid = toUserId('usr_abc123');
let oid = 'ord_999' as OrderId;

// uid = oid; 
// ❌ Type 'OrderId' is not assignable to type 'UserId'.
// Type '"OrderId"' is not assignable to type '"UserId"'.
```

---

## 6. Mapped Types Avancés : Renommage de Clés avec `as`

TypeScript permet de filtrer et renommer dynamiquement les clés lors d'un mappage via la clause `as` combinée aux Template Literals :

```typescript
interface State {
  user: { name: string };
  isLogged: boolean;
  score: number;
}

// Générer automatiquement un contrat de Getters typés
export type CreateGetters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type StateGetters = CreateGetters<State>;
/*
Résultat :
{
  getUser: () => { name: string };
  getIsLogged: () => boolean;
  getScore: () => number;
}
*/
```

---

## Résumé pour l'Architecte TypeScript

1. **Évitez les assertions lâches (`as unknown as T`) :** Résolvez l'équation de type par des conditionnels, des contraintes et des prédicats stricts.
2. **Attention à la distributivité :** Utilisez `[T] extends [never]` pour tester `never` sans court-circuit.
3. **Privilégiez les interfaces contravariantes :** Utilisez `key: (arg: T) => void` pour sécuriser les arguments de rappels (*callbacks*).
4. **Protégez le modèle métier par des Branded Types :** Empêchez les erreurs de permutation d'identifiants ou d'unités de mesure dès la compilation.