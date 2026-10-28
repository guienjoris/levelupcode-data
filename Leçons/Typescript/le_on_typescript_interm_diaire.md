# Masterclass TypeScript — Niveau Intermédiaire

Après avoir assimilé les bases (primitifs, interfaces, inférence et unions simples), l'étape intermédiaire consiste à exploiter la puissance du **système de types structurel** de TypeScript pour concevoir du code auto-documenté, réutilisable et impossible à détourner à l'exécution.

---

## 1. Modélisation Robuste : Les Unions Discriminées (*Discriminated Unions*)

L'un des patterns les plus puissants en TypeScript pour modéliser des états ou des payloads polymorphes est l'**Union Discriminée** (ou type somme / type algébrique).

Elle repose sur 3 critères :
1. Plusieurs types possédant une **propriété commune littérale** (le discriminant / tag).
2. Une **union** (`|`) de ces types.
3. Une structure de contrôle (`switch` ou `if`) exploitant le discriminant pour le *narrowing*.

### 1.1 Exemple : Machine à états pour requêtes réseau

```typescript
interface StateIdle {
  status: 'idle';
}

interface StateLoading {
  status: 'loading';
}

interface StateSuccess<T> {
  status: 'success';
  data: T;
  receivedAt: Date;
}

interface StateError {
  status: 'error';
  error: Error;
}

export type FetchState<T> =
  | StateIdle
  | StateLoading
  | StateSuccess<T>
  | StateError;
```

### 1.2 Exhaustivité avec le type `never` (*Exhaustive Checking*)

Pour s'assurer au moment de la compilation qu'aucun nouvel état n'a été oublié lors d'une refactorisation, on utilise une vérification d'exhaustivité via le type `never` :

```typescript
function renderState<T>(state: FetchState<T>): string {
  switch (state.status) {
    case 'idle':
      return 'En attente...';
    case 'loading':
      return 'Chargement des données...';
    case 'success':
      // TypeScript sait automatiquement que state possède 'data' et 'receivedAt'
      return `Succès : ${JSON.stringify(state.data)}`;
    case 'error':
      // TypeScript sait que state possède 'error'
      return `Erreur : ${state.error.message}`;
    default: {
      // Si un nouvel état est ajouté à FetchState sans être traité ici,
      // TypeScript lèvera une erreur de compilation sur cette ligne.
      const _unreachable: never = state;
      throw new Error(`État non supporté : ${JSON.stringify(_unreachable)}`);
    }
  }
}
```

---

## 2. Génériques Avancés & Contraintes (`extends`)

Les génériques ne servent pas seulement à envelopper des valeurs dans des tableaux ; ils permettent de contraindre des signatures tout en préservant le typage strict des arguments passés.

### 2.1 Contraintes de forme (*Generic Constraints*)

On force un paramètre générique à respecter un contrat minimal sans perdre son type concret :

```typescript
interface HasId {
  id: string | number;
}

// T doit au minimum posséder une propriété 'id'
function mergeWithAudit<T extends HasId>(entity: T) {
  return {
    ...entity,
    auditedAt: new Date(),
  };
}

const user = { id: 'usr_1', name: 'Sophie', role: 'admin' };
const auditedUser = mergeWithAudit(user);

// TypeScript préserve toutes les propriétés initiales :
console.log(auditedUser.name); // Type string garanti
console.log(auditedUser.auditedAt); // Type Date garanti
```

### 2.2 Sécurisation de propriétés avec `keyof`

L'opérateur `keyof` extrait sous forme d'union littérale toutes les clés publiques d'un type :

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const settings = {
  theme: 'dark' as const,
  timeoutMs: 5000,
  notificationsEnabled: true,
};

// Types parfaitement inférés :
const currentTheme = getProperty(settings, 'theme'); // Type: "dark"
const timeout = getProperty(settings, 'timeoutMs');   // Type: number

// ❌ Erreur à la compilation :
// getProperty(settings, 'unknownProp');
// Argument of type '"unknownProp"' is not assignable to parameter of type '"theme" | "timeoutMs" | "notificationsEnabled"'.
```

---

## 3. Typage Conditionnel & Prédicats de Types personnalisés

### 3.1 Type Predicates (`value is Type`)

TypeScript ne peut pas toujours deviner la forme d'un objet inspecté par une fonction utilitaire booléenne standard. Les *Type Predicates* apprennent au compilateur à rétrécir le type :

```typescript
interface Article {
  title: string;
  wordCount: number;
}

interface Video {
  title: string;
  durationMinutes: number;
}

type Media = Article | Video;

// Si la fonction retourne true, le compilateur sait que 'media' est une 'Video'
function isVideo(media: Media): media is Video {
  return 'durationMinutes' in media;
}

function processMedia(media: Media) {
  if (isVideo(media)) {
    console.log(`Durée : ${media.durationMinutes} min`);
  } else {
    console.log(`Mots : ${media.wordCount}`);
  }
}
```

### 3.2 Assertion Functions (`asserts condition`)

Permet de lever une exception si une condition n'est pas remplie, tout en propageant l'information de type pour le reste de la fonction :

```typescript
function assertIsDefined<T>(val: T, message?: string): asserts val is NonNullable<T> {
  if (val === undefined || val === null) {
    throw new Error(message || 'Valeur non définie rencontrée.');
  }
}

function computeScore(score: number | null) {
  assertIsDefined(score, 'Le score ne peut pas être null');
  // À partir d'ici, 'score' est de type 'number' (null est éliminé)
  return score.toFixed(1);
}
```

---

## 4. Les Utility Types Essentiels et leur Mécanique

TypeScript fournit des transformations de types prêtes à l'emploi. Comprendre leur fonctionnement interne est indispensable.

### 4.1 Tour d'horizon pratique

```typescript
interface UserProfile {
  id: string;
  email: string;
  displayName: string;
  bio?: string;
  createdAt: Date;
}

// 1. Partial<T> : rend toutes les clés optionnelles (ex: patch HTTP)
type UpdateUserProfileInput = Partial<UserProfile>;

// 2. Required<T> : rend tous les champs obligatoires (ex: bio n'est plus optionnelle)
type CompleteUserProfile = Required<UserProfile>;

// 3. Pick<T, K> : sélectionne un sous-ensemble de clés
type PublicPreview = Pick<UserProfile, 'displayName' | 'bio'>;

// 4. Omit<T, K> : supprime des clés spécifiques
type UserProfileWithoutTimestamps = Omit<UserProfile, 'createdAt'>;

// 5. Record<K, V> : dictionnaire clé-valeur strictement typé
type UserRolesDirectory = Record<string, 'admin' | 'editor' | 'viewer'>;
```

### 4.2 Découvrir les Mapped Types internes

Comment `Partial<T>` est-il codé en interne par TypeScript ? Grâce aux **types mappés** (*Mapped Types*) :

```typescript
// Réimplémentation manuelle de Partial :
type MyPartial<T> = {
  [Key in keyof T]?: T[Key];
};

// Réimplémentation de Readonly :
type MyReadonly<T> = {
  readonly [Key in keyof T]: T[Key];
};
```

---

## 5. Indexed Access Types & `as const` (Const Assertions)

### 5.1 Les assertions `as const`

Sans assertion `as const`, TypeScript élargit (*widen*) les types de littéraux vers leurs primitifs (`string`, `number`). Avec `as const`, les structures deviennent profondément immuables et typées littéralement.

```typescript
// ❌ Élargissement par défaut : type string[]
const HTTP_METHODS_MUTABLE = ['GET', 'POST', 'PUT', 'DELETE'];

// ✅ Const assertion : type readonly ["GET", "POST", "PUT", "DELETE"]
const HTTP_METHODS = ['GET', 'POST', 'PUT', 'DELETE'] as const;

// Extraction du type d'union à partir du tableau :
type HttpMethod = typeof HTTP_METHODS[number];
// Résultat : "GET" | "POST" | "PUT" | "DELETE"
```

### 5.2 Lookup Types (Accès indexé)

On peut extraire le type précis d'un sous-champ sans redéfinir une nouvelle interface :

```typescript
interface CompanyHierarchy {
  headquarters: {
    address: {
      street: string;
      zipCode: string;
      city: string;
    };
  };
}

// Extraction ciblée :
type Address = CompanyHierarchy['headquarters']['address'];
```

---

## 6. Bonnes Pratiques du Développeur Intermédiaire

1. **Préférer les unions discriminées aux drapeaux booléens multiples :** Évitez `{ isLoading: boolean, isError: boolean, isSuccess: boolean }` qui autorise des états impossibles (ex: `isLoading: true` et `isSuccess: true` en même temps). Modélisez un seul champ `status`.
2. **Utiliser `as const` pour dériver les types depuis les données :** Évitez de dupliquer une liste de valeurs et son type d'union manuellement.
3. **Exploiter `keyof` et les génériques :** N'utilisez pas `string` pour représenter un nom de clé dynamique sur un objet.
4. **Toujours clore les `switch` discriminés avec un contrôle d'exhaustivité `never`.**