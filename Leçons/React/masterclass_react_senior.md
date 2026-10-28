# Masterclass React — Niveau Senior & Staff Engineer

À ce niveau, maîtriser les hooks standard ne suffit plus : il faut appréhender les couches internes du moteur de rendu (**React Fiber**, **Scheduler**, **Lanes Model**), concevoir des architectures d'état découplées, optimiser le coût CPU/mémoire au millième de seconde et maîtriser les paradigmes modernes (concurrence et React Server Components).

---

## 1. Sous le capot : Architecture Fiber, Scheduler & Modèle des Lanes

### 1.1 Qu'est-ce qu'un nœud Fiber ?

Avant React 16 (l'ancien moteur *Stack Reconciler*), la réconciliation était synchrone et récursive : une fois lancée, elle ne pouvait pas être interrompue sans geler l'interface graphique.

Le moteur **Fiber** réimplémente la pile d'exécution sous la forme d'une structure de données en liste chaînée sur le tas (*heap*). Chaque élément React possède un nœud Fiber correspondant avec trois pointeurs clés :
* `child` : Premier enfant direct.
* `sibling` : Frère suivant direct.
* `return` : Nœud parent vers lequel remonter.

```
       [ App ] (Fiber)
          │ child
          ▼
     [ Header ] (Fiber) ──sibling──► [ Main ] (Fiber)
                                        │ child
                                        ▼
                                  [ Article ] (Fiber)
```

Chaque Fiber contient également :
* `memoizedState` : Liste chaînée des hooks du composant.
* `memoizedProps` et `pendingProps`.
* `lanes` : Masque binaire indiquant la priorité de travail du nœud.
* `alternate` : Référence vers la version correspondante dans l'autre arbre (*Double Buffering*).

### 1.2 Le Double Buffering

Pour éviter de montrer une interface partiellement construite ou instable à l'écran, React maintient en permanence deux arbres Fiber :
1. **Current Tree** : L'arbre actuellement affiché dans le DOM.
2. **Work-in-Progress (WIP) Tree** : L'arbre en cours de construction ou de mise à jour en arrière-plan.

Une fois que le WIP tree a fini d'être calculé sans interruption bloquante, React bascule le pointeur racine (*root pointer*) en une seule opération atomique synchrone : le WIP devient le nouveau Current.

### 1.3 Les Deux Phases du Rendu

```
Requête de mise à jour (setState)
               │
               ▼
┌──────────────────────────────────────────────┐
│  Phase 1 : RENDER (Asynchrone / Interruptible)│
│  - Parcours de l'arbre Fiber                 │
│  - Exécution des fonctions de composants     │
│  - Calcul des deltas (Diffing)               │
│  - Peut être mise en pause ou abandonnée     │
└──────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────┐
│  Phase 2 : COMMIT (Synchrone / Atomique)     │
│  - Mutation effective du DOM                 │
│  - Exécution de useLayoutEffect              │
│  - Bascule du pointeur Current / WIP         │
│  - Planification de useEffect                │
└──────────────────────────────────────────────┘
```

---

## 2. Concurrence & Priorisation : `useTransition` & `useDeferredValue`

Le moteur concurrent permet à React d'interrompre un calcul de rendu lourd pour laisser passer une interaction utilisateur prioritaire (frappe au clavier, clic).

React classe les mises à jour grâce aux **Lanes** (niveaux de priorité sous forme de bitmasks) :
* *SyncLane* : Saisie utilisateur directe, focus.
* *TransitionLane* : Filtrage d'une liste lourde, changement d'onglet, requêtes de navigation.

### 2.1 Pattern `useTransition`

`useTransition` permet d'indiquer à React qu'une mise à jour d'état est non urgente et interruptible :

```tsx
import { useState, useTransition, ChangeEvent } from 'react';

export function HeavySearch() {
  const [inputVal, setInputVal] = useState('');
  const [searchQuery, setSearchQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  function handleChange(e: ChangeEvent<HTMLInputElement>) {
    // 1. Priorité Haute (Sync) : feedback visuel immédiat dans l'input
    setInputVal(e.target.value);

    // 2. Priorité Basse (Transition) : le recalcul de la liste peut être différé
    startTransition(() => {
      setSearchQuery(e.target.value);
    });
  }

  return (
    <div>
      <input type="text" value={inputVal} onChange={handleChange} />
      {isPending && <span className="spinner">Calcul en cours...</span>}
      <ExpensiveList query={searchQuery} />
    </div>
  );
}
```

### 2.2 `useDeferredValue`

Permet de différer la propagation d'une valeur reçue en props sans encapsuler le dispatch dans `startTransition` :

```tsx
import { useDeferredValue, useMemo } from 'react';

function ProductGrid({ rawQuery }: { rawQuery: string }) {
  // deferredQuery restera à l'ancienne valeur pendant que le nouvel arbre se prépare
  const deferredQuery = useDeferredValue(rawQuery);
  const isStale = deferredQuery !== rawQuery;

  const items = useMemo(() => {
    return runExpensiveFilter(deferredQuery);
  }, [deferredQuery]);

  return (
    <div style={{ opacity: isStale ? 0.6 : 1, transition: 'opacity 0.2s' }}>
      {items.map(item => <Card key={item.id} data={item} />)}
    </div>
  );
}
```

---

## 3. Optimisations Mémoire & Rendu : Profiling et Pièges d'Égalité

### 3.1 Anatomie d'un Anti-Pattern `useMemo` / `useCallback`

Mémoriser a un coût non négligeable : allocation de closure, instanciation du tableau de dépendances et comparaison élément par élément (`Object.is`) à chaque rendu.

```tsx
// ❌ ANTI-PATTERN : Coût de mémorisation supérieur au coût de recalcul
const fullName = useMemo(() => `${firstName} ${lastName}`, [firstName, lastName]);

// ❌ ANTI-PATTERN : Rupture d'identité de référence en cascade
const handleClick = useCallback(() => {
  doSomething(props.filter); // Si 'props' est destructuré ou change souvent
}, [props.filter]);
```

**Règles d'or Senior :**
1. Utilisez `useMemo` uniquement pour :
   * Des calculs algorithmiques coûteux ($\ge 1\text{ ms}$ au CPU profiler).
   * Stabiliser une référence d'objet/tableau passée en prop à un composant encapsulé dans `React.memo`.
   * Stabiliser une référence injectée dans les dépendances d'un hook en aval (`useEffect`, `useCallback`).
2. Pour optimiser les rendus sans mémorisation : **privilégiez la composition** (*Component Composition*). Passer du JSX en `children` évite le recalcul des enfants quand le parent se réexécute, car leurs props ne changent pas.

### 3.2 Context Slicing : Résoudre le problème du Context Hell

Par défaut, tout consommateur de `useContext(MyContext)` subit un re-render dès qu'une propriété du contexte change, même s'il n'en utilise qu'une infime partie non modifiée.

**Solution : Séparer l'état (State) des dispatches (Actions) :**

```tsx
import { createContext, useContext, useReducer, ReactNode, Dispatch } from 'react';

type State = { count: number; theme: 'dark' | 'light' };
type Action = { type: 'INCREMENT' } | { type: 'TOGGLE_THEME' };

const StateContext = createContext<State | null>(null);
const DispatchContext = createContext<Dispatch<Action> | null>(null);

export function CounterProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(
    (s: State, a: Action): State => {
      switch (a.type) {
        case 'INCREMENT': return { ...s, count: s.count + 1 };
        case 'TOGGLE_THEME': return { ...s, theme: s.theme === 'dark' ? 'light' : 'dark' };
      }
    },
    { count: 0, theme: 'light' }
  );

  return (
    <StateContext.Provider value={state}>
      {/* dispatch a une référence strictement stable : les consommateurs purs d'actions ne re-rendent JAMAIS */}
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Custom hooks avec garde contre les valeurs nulles
export const useAppState = () => {
  const ctx = useContext(StateContext);
  if (!ctx) throw new Error("useAppState must be used inside CounterProvider");
  return ctx;
};

export const useAppDispatch = () => {
  const ctx = useContext(DispatchContext);
  if (!ctx) throw new Error("useAppDispatch must be used inside CounterProvider");
  return ctx;
};
```

---

## 4. Gestion d'État Asynchrone : State Machines & Server State

En architecture moderne, la distinction entre **Client State** (modal ouverte, thème sélectionné) et **Server State** (données mises en cache, invalidation, synchronisation distante) est primordiale. Utiliser `useEffect` pour manipuler du Server State est considéré comme un anti-pattern en production.

### 4.1 State Machine avec `useReducer`

Pour les logiques métier complexes, éliminez les combinaisons d'états illégaux en typant explicitement les transitions :

```tsx
type AuthMachineState =
  | { status: 'unauthenticated'; error: null }
  | { status: 'authenticating'; error: null }
  | { status: 'authenticated'; token: string; user: { id: string } }
  | { status: 'error'; error: Error };

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; token: string; user: { id: string } }
  | { type: 'LOGIN_FAILURE'; error: Error }
  | { type: 'LOGOUT' };

function authReducer(state: AuthMachineState, action: AuthAction): AuthMachineState {
  switch (state.status) {
    case 'unauthenticated':
    case 'error':
      if (action.type === 'LOGIN_START') return { status: 'authenticating', error: null };
      return state;
    case 'authenticating':
      if (action.type === 'LOGIN_SUCCESS') {
        return { status: 'authenticated', token: action.token, user: action.user };
      }
      if (action.type === 'LOGIN_FAILURE') {
        return { status: 'error', error: action.error };
      }
      return state;
    case 'authenticated':
      if (action.type === 'LOGOUT') return { status: 'unauthenticated', error: null };
      return state;
  }
}
```

---

## 5. React Server Components (RSC) vs SSR Client

Les Server Components introduisent une frontière physique entre le code serveur et le code client.

| Critère | SSR Traditionnel | React Server Components (RSC) |
| :--- | :--- | :--- |
| **Exécution** | Serveur (génération HTML initiale) puis Client (Hydratation) | **Uniquement sur le serveur** |
| **Poids du Bundle** | Le code JS des composants est envoyé au navigateur | **0 Ko de JavaScript** envoyé au client |
| **Accès aux Ressources** | Requêtes API (REST/GraphQL) via le réseau | Accès direct au système de fichiers, DB, caches Redis |
| **Interactivité & Hooks** | `useState`, `useEffect`, `onClick` disponibles | ❌ Aucun hook d'état ni listener d'événement DOM |
| **Format de Transport** | Fichier HTML brut réhydraté | Stream JSON spécifique (arborescence Fiber sérialisée) |

### Structure Hybride Server / Client

```tsx
// ServerComponent.tsx (par défaut, aucun 'use client')
import { db } from '@/infra/db';
import { ClientCounter } from './ClientCounter'; // Composant 'use client'

export async function ProductDetails({ id }: { id: string }) {
  // Requête directe en base de données sans passer par une API intermédiaire
  const product = await db.products.findUniqueOrThrow({ where: { id } });

  return (
    <section>
      <h1>{product.title}</h1>
      <p>{product.description}</p>
      {/* On injecte l'interactivité client uniquement sur les feuilles nécessaires */}
      <ClientCounter initialStock={product.stock} />
    </section>
  );
}
```

---

## 6. Synthèse d'Ingénierie pour Staff Engineers

1. **Penser en flux de travail non bloquants** : Les calculs lourds doivent être délégués au `useTransition` ou être isolés dans des Web Workers.
2. **Favoriser la composition sur la mémorisation outrancière** : L'imbrication avec `children` est plus performante et plus lisible que d'envelopper chaque fonction dans `useCallback`.
3. **Éliminer les cascades de `useEffect`** : Déplacer les synchronisations vers des gestionnaires d'événements impératifs ou des bibliothèques de Server State dédiées (TanStack Query).
4. **Isoler les frontières d'hydratation** : Dans les architectures Next.js / Remix, repousser les directives `'use client'` au plus près des feuilles de l'arbre DOM pour minimiser la taille du bundle JavaScript initial.