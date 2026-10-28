# Masterclass React — Niveau Intermédiaire

Après avoir assimilé les composants, le JSX, les props et `useState`, franchir le cap intermédiaire consiste à comprendre **comment React orchestre les rendus**, comment synchroniser vos composants avec des systèmes extérieurs (APIs, timers, DOM) sans provoquer de boucles infinies ou de fuites de mémoire, et comment architecturer votre logique réutilisable.

---

## 1. Le Cycle de Rendu & la Levée d'État (*Lifting State Up*)

### 1.1 Règle fondamentale du rendu
En React :
1. Un composant se réexécute (*re-render*) quand son **state** interne change ou quand ses **props** changent.
2. Lorsqu'un composant parent se réexécute, **tous ses composants enfants se réexécutent également par défaut**, peu importe que leurs propres props aient changé ou non.
3. Le rendu doit rester une **fonction pure** : pas de mutation de variables globales, pas de requêtes réseau directes dans le corps de la fonction.

### 1.2 La levée d'état (*Lifting State Up*)
Quand deux composants frères ont besoin de partager la même information ou d'interagir ensemble, vous ne pouvez pas passer de props directement d'un frère à l'autre. La solution consiste à **remonter l'état au parent commun le plus proche**.

```jsx
import { useState } from 'react';

// Composant Enfant A : Émetteur
function SearchInput({ query, onQueryChange }) {
  return (
    <input
      type="text"
      value={query}
      onChange={(e) => onQueryChange(e.target.value)}
      placeholder="Rechercher..."
    />
  );
}

// Composant Enfant B : Récepteur
function SearchSummary({ count, query }) {
  return (
    <p>
      {count} résultat(s) pour la recherche : <strong>{query || 'tous'}</strong>
    </p>
  );
}

// Composant Parent : Détient la source de vérité
export default function SearchContainer({ items = [] }) {
  const [query, setQuery] = useState('');

  const filteredItems = items.filter((item) =>
    item.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <div>
      <SearchInput query={query} onQueryChange={setQuery} />
      <SearchSummary count={filteredItems.length} query={query} />
    </div>
  );
}
```

---

## 2. Maîtriser `useEffect` & les Effets de Bord

`useEffect` permet de synchroniser un composant avec un système extérieur (**I/O**, requêtes HTTP, écouteurs d'événements globaux du navigateur, timers).

### 2.1 Le tableau de dépendances (*Dependency Array*)

Le comportement de `useEffect` dépend strictement de son second argument :

| Syntaxe | Quand l'effet s'exécute-t-il ? | Cas d'usage typique |
| :--- | :--- | :--- |
| `useEffect(() => { ... })` *(aucun tableau)* | Après **chaque** rendu | ⚠️ À éviter (souvent source de boucles infinies) |
| `useEffect(() => { ... }, [])` *(tableau vide)* | **Une seule fois** après le premier affichage (*mount*) | Chargement initial de données, enregistrement d'un listener global |
| `useEffect(() => { ... }, [a, b])` *(dépendances)* | Après le 1er rendu, puis **uniquement si `a` ou `b` changent** | Recharger des données quand un identifiant ou un filtre change |

### 2.2 La fonction de nettoyage (*Cleanup Function*)
Tout effet qui crée une souscription, un timer ou un écouteur doit **impérativement retourner une fonction de nettoyage** pour éviter les fuites de mémoire.

```jsx
import { useState, useEffect } from 'react';

function WindowWidthTracker() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }

    // 1. Souscription à l'événement
    window.addEventListener('resize', handleResize);

    // 2. Nettoyage : exécuté au démontage du composant ou avant la prochaine exécution de l'effet
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []); // [] = s'abonne une seule fois

  return <p>Largeur de la fenêtre : {width}px</p>;
}
```

### 2.3 Chargement de données sécurisé avec `AbortController`
Pour éviter de mettre à jour le state d'un composant déjà démonté ou d'avoir des résultats obsolètes en cas de frappe rapide (*race condition*) :

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);
    setError(null);

    async function loadUser() {
      try {
        const res = await fetch(`https://api.example.com/users/${userId}`, {
          signal: controller.signal,
        });
        if (!res.ok) throw new Error(`Erreur HTTP ${res.status}`);
        const data = await res.json();
        setUser(data);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    loadUser();

    // Annule la requête en cours si userId change ou si le composant est démonté
    return () => controller.abort();
  }, [userId]);

  if (loading) return <p>Chargement...</p>;
  if (error) return <p>Erreur : {error}</p>;
  return <div>Utilisateur : {user?.name}</div>;
}
```

---

## 3. Accès au DOM & Valeurs Persistantes avec `useRef`

Le hook `useRef` renvoie un objet mutable `{ current: valeur }`. Sa caractéristique clé : **modifier `ref.current` NE déclenche JAMAIS de nouveau rendu.**

### 3.1 Accéder directement à un nœud du DOM
Cas d'usage classique : donner le focus à un champ de texte automatiquement :

```jsx
import { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    // inputRef.current pointe vers le nœud HTML réel
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} type="text" placeholder="Focus automatique" />;
}
```

### 3.2 Conserver une valeur mutable entre les rendus (ex : ID de Timer)

```jsx
import { useState, useRef } from 'react';

function Stopwatch() {
  const [seconds, setSeconds] = useState(0);
  const timerRef = useRef(null); // Stocke l'ID renvoyé par setInterval

  function handleStart() {
    if (timerRef.current !== null) return;
    timerRef.current = setInterval(() => {
      setSeconds((prev) => prev + 1);
    }, 1000);
  }

  function handleStop() {
    clearInterval(timerRef.current);
    timerRef.current = null;
  }

  return (
    <div>
      <p>Temps : {seconds}s</p>
      <button onClick={handleStart}>Démarrer</button>
      <button onClick={handleStop}>Arrêter</button>
    </div>
  );
}
```

---

## 4. Composants Contrôlés vs Non Contrôlés

En React, privilégiez toujours les **composants contrôlés** pour les formulaires : le state React est l'unique source de vérité.

```jsx
import { useState } from 'react';

function RegistrationForm() {
  const [form, setForm] = useState({
    username: '',
    email: '',
    plan: 'free',
  });

  function handleChange(e) {
    const { name, value } = e.target;
    // Mise à jour générique et immuable d'un formulaire
    setForm((prev) => ({
      ...prev,
      [name]: value,
    }));
  }

  function handleSubmit(e) {
    e.preventDefault();
    console.log('Données soumises :', form);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="username"
        value={form.username}
        onChange={handleChange}
        placeholder="Pseudo"
      />
      <input
        name="email"
        type="email"
        value={form.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <select name="plan" value={form.plan} onChange={handleChange}>
        <option value="free">Gratuit</option>
        <option value="pro">Pro</option>
      </select>
      <button type="submit">Valider</button>
    </form>
  );
}
```

---

## 5. Les Hooks Personnalisés (*Custom Hooks*)

Un **custom hook** est une fonction JavaScript ordinaire dont le nom commence obligatoirement par `use` et qui peut appeler d'autres hooks React (`useState`, `useEffect`, `useRef`). C'est le mécanisme roi pour extraire et réutiliser la logique métier entre plusieurs composants.

### Exemple : Hook réutilisable `useLocalStorage`

```jsx
import { useState, useEffect } from 'react';

export function useLocalStorage(key, initialValue) {
  // 1. Initialisation paresseuse (lazy initialization) pour ne pas lire le localStorage à chaque rendu
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.warn(`Erreur lors de la lecture de la clé "${key}" :`, error);
      return initialValue;
    }
  });

  // 2. Synchronisation automatique avec le localStorage
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(`Erreur d'écriture dans le localStorage :`, error);
    }
  }, [key, storedValue]);

  return [storedValue, setStoredValue];
}
```

### Utilisation dans un composant :
```jsx
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage('app-theme', 'light');

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Mode actuel : {theme}
    </button>
  );
}
```

---

## 6. Synthèse des Bonnes Pratiques Intermédiaires

1. **Ne mettez pas en state ce qui peut être calculé à la volée :** Si une valeur peut être déduite des props ou d'un autre state (ex: la longueur d'une liste filtrée), calculez-la directement pendant le rendu au lieu de la dupliquer dans un `useState`.
2. **Respectez les règles des Hooks :**
   * Appelez les hooks uniquement au **niveau racine** de votre composant (jamais dans un `if`, une boucle `for` ou une fonction imbriquée).
   * Appelez-les uniquement depuis des composants fonctions React ou d'autres custom hooks.
3. **Nettoyez toujours les effets de bord dans `useEffect`** (listeners, timeouts, websockets, contrôleurs réseau).
4. **Utilisez `useRef` pour les valeurs qui ne nécessitent pas de rafraîchir l'écran.**