# Quiz d'Évaluation — React (Niveau Intermédiaire)

Ce quiz teste votre compréhension des mécanismes intermédiaires de React : cycle de rendu, levée d'état (*lifting state up*), gestion des effets de bord avec `useEffect`, références avec `useRef`, composants contrôlés et création de hooks personnalisés.

---

### Question 1 : Règles de déclenchement du rendu
Soit un composant parent qui met à jour son état interne avec `setState`. Que se passe-t-il par défaut pour les composants enfants déclarés à l'intérieur de son JSX ?

- [ ] A) Seuls les enfants dont les `props` ont été modifiées sont réexécutés (*re-render*).
- [ ] B) Tous les composants enfants sont réexécutés, même si leurs `props` n'ont pas changé.
- [ ] C) React compare automatiquement le DOM réel avant de réexécuter les fonctions enfants.
- [ ] D) Les composants enfants ne sont jamais réexécutés sauf s'ils possèdent leur propre `useState`.

---

### Question 2 : Nettoyage d'effet (*Cleanup*) dans `useEffect`
Considérez l'effet suivant :

```javascript
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Tick');
  }, 1000);

  return () => clearInterval(timer);
}, [userId]);
```

À quel(s) moment(s) précis la fonction retournée `() => clearInterval(timer)` est-elle invoquée par React ?

- [ ] A) Uniquement lorsque l'application entière est fermée par l'utilisateur.
- [ ] B) Uniquement lors du démontage définitif du composant (*unmount*).
- [ ] C) Avant chaque nouvelle exécution de l'effet (si `userId` a changé) ainsi qu'au démontage du composant.
- [ ] D) Immédiatement après la fin du premier rendu du composant.

---

### Question 3 : État calculé (*Derived State*)
Vous disposez d'un tableau d'articles `items` stocké dans un `useState`. Vous devez afficher le nombre total d'articles. Quelle approche respecte les meilleures pratiques de React ?

- [ ] A) Créer un second état `const [count, setCount] = useState(0)` et le mettre à jour dans un `useEffect` qui écoute `items`.
- [ ] B) Créer une variable calculée directement à la volée pendant le rendu : `const count = items.length;`.
- [ ] C) Utiliser `useRef` pour incrémenter manuellement un compteur à chaque fois que `items.push()` est appelé.
- [ ] D) Stocker la longueur dans une variable globale extérieure au composant.

---

### Question 4 : `useRef` vs `useState`
Quelle est la différence fondamentale de comportement entre modifier une valeur dans `useRef` (`ref.current = nouvelleValeur`) et appeler le setter d'un `useState` (`setValue(nouvelleValeur)`) ?

- [ ] A) `useRef` déclenche un rendu synchrone immédiat, tandis que `useState` déclenche un rendu asynchrone.
- [ ] B) Modifier `ref.current` met à jour la valeur en mémoire sans déclencher de nouveau rendu, alors que `setValue` planifie un nouveau rendu de l'interface.
- [ ] C) `useRef` ne peut stocker que des nœuds du DOM, alors que `useState` peut stocker des objets ou des types primitifs.
- [ ] D) `useState` conserve sa valeur entre les rendus, tandis que `useRef` est réinitialisé à chaque affichage.

---

### Question 5 : Requêtes asynchrones et concurrence (*Race Conditions*)
Pourquoi est-il fortement recommandé d'utiliser un `AbortController` à l'intérieur d'un `useEffect` effectuant un `fetch` dépendant d'un paramètre (ex: `userId`) ?

- [ ] A) Pour forcer le navigateur à mettre la réponse en cache HTTP.
- [ ] B) Pour contourner les restrictions CORS imposées par le serveur distant.
- [ ] C) Pour annuler la requête réseau précédente si le composant est démonté ou si `userId` change avant la fin du téléchargement, évitant d'afficher des données désynchronisées.
- [ ] D) Parce que `fetch` refuse de s'exécuter à l'intérieur d'un `useEffect` sans signal d'abandon.

---

### Question 6 : Formulaires et composants contrôlés
Dans un formulaire contrôlé, quelle combinaison d'attributs est obligatoire sur une balise `<input>` pour garantir que React demeure l'unique source de vérité ?

- [ ] A) `defaultValue` et `ref`
- [ ] B) `value` lié à une variable d'état, et `onChange` mettant à jour cette variable d'état.
- [ ] C) `name` et `onSubmit`
- [ ] D) `readOnly` et `onClick`

---

### Question 7 : Règles des Custom Hooks
Quelle affirmation concernant les *Custom Hooks* est **vraie** ?

- [ ] A) Ils doivent impérativement retourner un tuple au format `[valeur, fonction]`, comme `useState`.
- [ ] B) Deux composants appelant le même custom hook partagent automatiquement le même état en mémoire.
- [ ] C) Leur nom doit obligatoirement débuter par le préfixe `use`, ce qui permet aux linters et au compilateur React d'appliquer les règles strictes des hooks.
- [ ] D) Un custom hook ne peut pas appeler d'autres hooks natifs tels que `useEffect` ou `useRef`.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** Par défaut dans le modèle de réconciliation de React, lorsqu'un composant parent subit un nouveau rendu, l'ensemble de son sous-arbre de composants enfants est réexécuté. Pour empêcher ce comportement sur des sous-arbres lourds, on utilise des optimisations ciblées comme `React.memo` combiné avec `useCallback` / `useMemo`.

### 2. Réponse : **C**
**Explication :** La fonction de nettoyage retournée par `useEffect` s'exécute à deux moments : avant que l'effet ne soit réexécuté avec de nouvelles dépendances (pour nettoyer l'exécution précédente), et au démontage final du composant pour libérer les ressources.

### 3. Réponse : **B**
**Explication :** En règle générale, toute valeur qui peut être déduite de manière synchrone à partir des props ou du state existant ne doit **jamais** être dupliquée dans un autre state ni synchronisée via un `useEffect`. Calculer `const count = items.length;` directement dans le corps de la fonction est plus rapide, plus simple et évite des rendus en cascade superflus.

### 4. Réponse : **B**
**Explication :** L'objet retourné par `useRef` conserve sa référence et sa propriété `.current` durant tout le cycle de vie du composant. Modifier `.current` est une mutation directe qui ne notifie pas React : aucun réaffichage de l'interface n'est planifié.

### 5. Réponse : **C**
**Explication :** Si l'utilisateur passe rapidement du profil 1 au profil 2, la requête pour le profil 1 peut arriver après celle du profil 2 en raison de latences réseau variables (*race condition*). Annuler la requête précédente avec `controller.abort()` garantit que seule la donnée la plus récente est traitée et évite de tenter d'écrire dans l'état d'un composant démonté.

### 6. Réponse : **B**
**Explication :** Dans un composant contrôlé, la valeur affichée dans le champ est pilotée par la prop `value={state}`, et chaque frappe au clavier déclenche l'événement `onChange`, qui appelle le setter de l'état pour mettre à jour la valeur. Le DOM ne conserve aucun état autonome.

### 7. Réponse : **C**
**Explication :** Le préfixe `use` est la convention obligatoire du compilateur React pour identifier les hooks et faire appliquer leurs règles (pas d'appel conditionnel, pas d'appel dans des boucles). De plus, chaque invocation d'un custom hook crée un état **isolé et indépendant** propre au composant appelant.