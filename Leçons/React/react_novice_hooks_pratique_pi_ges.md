# React Débutant : Maîtriser les Hooks Essentiels & Éviter les Pièges Fréquents

Les **Hooks** permettent d'attacher un état et des effets de cycle de vie à des composants fonctions sans recourir aux anciennes classes React. Au-delà de la syntaxe de base, bien débuter exige de comprendre comment React cadence leurs exécutions pour éviter les bugs silencieux.

## 1. Les Règles Fondamentales des Hooks

Pour que React puisse associer correctement chaque hook au composant en cours d'exécution, vous devez respecter deux règles strictes :

1. **Appeler les hooks uniquement au niveau racine du composant :** Ne les placez jamais dans une boucle (`for`), une condition (`if`) ou une fonction imbriquée.
2. **Appeler les hooks uniquement depuis des fonctions React :** Depuis un composant React ou depuis un hook personnalisé (*custom hook* commençant par `use`).

```jsx
// ❌ INTERDIT : brise l'ordre interne des hooks de React
if (estConnecte) {
  useEffect(() => { /* ... */ }, []);
}

// ✅ CORRECT : la condition réside À L'INTÉRIEUR du hook
useEffect(() => {
  if (estConnecte) {
    /* ... */
  }
}, [estConnecte]);
```

---

## 2. `useState` : La Mise à Jour Fonctionnelle face aux Fermetures Périmées

Lorsque vous mettez à jour un état qui dépend de sa valeur précédente, passer directement la valeur calculée (`setCount(count + 1)`) expose à des bugs de valeurs obsolètes (*stale state*), notamment lors d'événements rapprochés ou asynchrones.

### La mise à jour fonctionnelle (*Updater Function*)

Passez une fonction de rappel à votre setter : React lui fournira l'état le plus récent et garanti :

```jsx
import { useState } from 'react';

export function CompteurSecurise() {
  const [compteur, setCompteur] = useState(0);

  const incrementerTroisFois = () => {
    // ❌ FAUX : Les trois appels lisent la même valeur de 'compteur' lors du clic
    // setCompteur(compteur + 1);
    // setCompteur(compteur + 1);
    // setCompteur(compteur + 1); // N'augmentera que de 1 au final !

    // ✅ CORRECT : Chaque étape s'applique sur l'état le plus récent
    setCompteur(prev => prev + 1);
    setCompteur(prev => prev + 1);
    setCompteur(prev => prev + 1); // Augmente bien de 3
  };

  return <button onClick={incrementerTroisFois}>Score : {compteur}</button>;
}
```

---

## 3. `useEffect` : L'Indispensable Fonction de Nettoyage (*Cleanup*)

Le tableau de dépendances détermine quand l'effet s'exécute :
* **Omis :** s'exécute après **chaque** rendu.
* **`[]` :** s'exécute **une seule fois** après le montage initial.
* **`[propA, stateB]` :** s'exécute lors du montage puis à chaque modification de `propA` ou `stateB`.

### Pourquoi et quand nettoyer ?
Si votre effet attache un écouteur d'événement (`addEventListener`), lance un timer (`setInterval`) ou initie une connexion, il doit **retourner une fonction de nettoyage**. Sans elle, vous créez une fuite de mémoire à chaque démontage ou réévaluation du composant.

```jsx
import { useState, useEffect } from 'react';

export function TraqueurFenetre() {
  const [largeur, setLargeur] = useState(window.innerWidth);

  useEffect(() => {
    const gererRedimensionnement = () => setLargeur(window.innerWidth);

    // 1. Mise en place de la souscription
    window.addEventListener('resize', gererRedimensionnement);

    // 2. Fonction de nettoyage (s'exécute au démontage ou avant de relancer l'effet)
    return () => {
      window.removeEventListener('resize', gererRedimensionnement);
    };
  }, []); // [] = écouteur configuré une seule fois

  return <p>Largeur de l'écran : {largeur}px</p>;
}
```

---

## 4. `useRef` : Deux Rôles Clés sans Déclencher de Rendu

`useRef` renvoie un objet persistant `{ current: valeurInitiale }` qui survit à travers tous les rendus. Sa particularité : **modifier `.current` ne déclenche aucun nouveau rendu du composant**.

### Cas d'usage A : Accéder directement à un nœud du DOM

```jsx
import { useRef } from 'react';

export function ChampRecherche() {
  const champTexteRef = useRef(null);

  const donnerFocus = () => {
    // Accès impératif à l'élément HTML natif
    champTexteRef.current?.focus();
  };

  return (
    <div>
      <input ref={champTexteRef} type="text" placeholder="Rechercher..." />
      <button onClick={donnerFocus}>Mettre le focus</button>
    </div>
  );
}
```

### Cas d'usage B : Garder une valeur en mémoire (ex: identifiant de Timer)

```jsx
import { useState, useRef } from 'react';

export function Chronometre() {
  const [secondes, setSecondes] = useState(0);
  const timerIdRef = useRef(null); // Stocke l'ID du timer sans re-render

  const demarrer = () => {
    if (timerIdRef.current !== null) return;
    timerIdRef.current = setInterval(() => {
      setSecondes(s => s + 1);
    }, 1000);
  };

  const arreter = () => {
    clearInterval(timerIdRef.current);
    timerIdRef.current = null;
  };

  return (
    <div>
      <p>Temps écoulé : {secondes}s</p>
      <button onClick={demarrer}>Lancer</button>
      <button onClick={arreter}>Stop</button>
    </div>
  );
}
```

---

## 5. `useContext` : Éviter le *Prop Drilling*

Le *Prop Drilling* désigne le fait de devoir faire passer une propriété manuellement à travers 4 ou 5 composants intermédiaires qui n'en ont pas l'utilité, simplement pour l'amener à l'enfant le plus profond.

`createContext` et `useContext` créent un canal de diffusion direct.

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Création du contexte avec une valeur par défaut
const ThemeContext = createContext('light');

export function Application() {
  const [theme, setTheme] = useState('dark');

  return (
    // 2. Le Provider distribue la valeur à toute la descendance
    <ThemeContext.Provider value={theme}>
      <PageAccueil />
    </ThemeContext.Provider>
  );
}

function PageAccueil() {
  // Ce composant intermédiaire n'a pas besoin de manipuler la prop 'theme' !
  return <BoutonThematique />;
}

function BoutonThematique() {
  // 3. Le composant final consomme directement la donnée
  const themeActuel = useContext(ThemeContext);
  return <button className={`btn-${themeActuel}`}>Bouton en mode {themeActuel}</button>;
}
```

---

## Tableau Comparatif Récapitulatif

| Hook | Rôle principal | Déclenche un Re-render ? | Règle d'or |
| :--- | :--- | :--- | :--- |
| **`useState`** | Stocker l'état visuel du composant | **Oui** (dès que la valeur change) | Utiliser `prev => ...` si la nouvelle valeur dépend de l'ancienne. |
| **`useEffect`** | Synchroniser avec l'extérieur (API, DOM, timers) | Non (s'exécute après le render) | Toujours retourner un nettoyage pour les abonnements. |
| **`useRef`** | Cibler le DOM ou stocker une valeur muette | **Non** (mutation de `.current` silencieuse) | Ne pas l'utiliser pour conditionner l'affichage direct dans le JSX. |
| **`useContext`** | Lire une donnée globale partagée | **Oui** (pour tous les consommateurs quand le Provider change) | Ne pas en abuser pour des états locaux simples. |