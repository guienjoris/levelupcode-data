# Quiz d'Évaluation — React Niveau Senior & Staff Engineer

Ce quiz teste votre compréhension approfondie des composants internes de React : le reconciler Fiber, le modèle des Lanes, la détection des goulots d'étranglement, le partitionnement de priorité et l'architecture moderne.

---

### Question 1 : Structure interne du Reconciler Fiber
Contrairement à l'ancien *Stack Reconciler* (React <16), chaque nœud de l'arbre Fiber est lié à ses voisins via des pointeurs explicites stockés sur le tas (*heap*). Quels sont ces trois pointeurs fondamentaux permettant le parcours itératif sans saturer la pile d'appels synchrone ?

- [ ] A) `parent`, `left`, `right`
- [ ] B) `child`, `sibling`, `return`
- [ ] C) `previous`, `next`, `root`
- [ ] D) `firstChild`, `lastChild`, `owner`

---

### Question 2 : Double Buffering & Phases de rendu
Pourquoi la phase de **Render** peut-elle être interrompue, reprise ou jetée par le moteur concurrent sans risquer d'altérer l'expérience utilisateur, alors que la phase de **Commit** est obligatoirement synchrone et ininterrompue ?

- [ ] A) Parce que la phase de Render ne manipule que l'arbre *Work-in-Progress (WIP)* en mémoire pure sans impacter le DOM réel, tandis que la phase de Commit mute directement le DOM et doit être atomique pour éviter le *tearing* visuel.
- [ ] B) Parce que le navigateur interdit l'utilisation de Web Workers durant la phase de Commit.
- [ ] C) Parce que la phase de Render s'exécute exclusivement côté serveur via Node.js.
- [ ] D) Parce que la phase de Commit ne consomme aucune ressource CPU.

---

### Question 3 : Priorisation concurrente (`useTransition` vs `useDeferredValue`)
Considérez une saisie utilisateur fluide dans un champ de recherche filtrant en temps réel un graphique contenant 10 000 éléments. Si vous enveloppez la mise à jour de l'état du graphique dans `startTransition(() => setQuery(value))`, que fait concrètement le Scheduler sous le capot ?

- [ ] A) Il délègue l'exécution du filtre à un Web Worker en arrière-plan.
- [ ] B) Il assigne à cette mise à jour une *TransitionLane* de basse priorité, permettant à toute frappe de touche ultérieure (*SyncLane*) d'interrompre le rendu du graphique pour garantir la réactivité immédiate du champ de saisie.
- [ ] C) Il debounce automatiquement l'exécution de 300 ms.
- [ ] D) Il convertit les composants du graphique en Server Components.

---

### Question 4 : Coût réel de la mémorisation (`useMemo` / `useCallback`)
Dans quel cas précis l'usage de `useCallback` est-il réellement bénéfique pour les performances globales ?

- [ ] A) Pour encapsuler n'importe quel gestionnaire d'événement inline (ex: `onClick={() => setOpen(true)}`) passé à une balise HTML native `<button>`.
- [ ] B) Pour stabiliser l'identité référentielle d'un callback transmis en prop à un composant enfant lourd mémoïsé avec `React.memo` (ou injecté dans le tableau de dépendances d'un `useEffect`).
- [ ] C) Pour forcer le ramasse-miettes à conserver la fonction en mémoire morte.
- [ ] D) Pour éviter que la fonction ne soit recompilée par le compilateur JIT V8.

---

### Question 5 : Modèle de concurrence et Tearing
Dans l'architecture concurrente de React 18+, qu'appelle-t-on le phénomène de **Tearing** (déchirure) et quelle API native a été introduite pour l'éviter lors de la lecture de stores externes mutables hors du cycle de vie React ?

- [ ] A) Une corruption mémoire causée par WebAssembly ; résolue avec `useLayoutEffect`.
- [ ] B) L'affichage asynchrone où deux composants lisent deux valeurs désynchronisées d'un même état global au cours du même rendu ; résolu avec `useSyncExternalStore`.
- [ ] C) Une désynchronisation entre le CSS-in-JS et le HTML ; résolue avec `useInsertionEffect`.
- [ ] D) Un crash lié au multi-fenêtrage ; résolu avec `useTransition`.

---

### Question 6 : React Server Components (RSC) vs Client Components
Quelle affirmation est techniquement exacte concernant les **React Server Components** (fichiers sans directive `'use client'`) ?

- [ ] A) Leur code source et leurs dépendances npm sont inclus dans le bundle JavaScript envoyé au navigateur client.
- [ ] B) Ils s'exécutent exclusivement à la compilation ou sur le serveur, renvoient un flux de description au format JSON/RSC Payload et leur code d'implémentation n'est jamais téléchargé par le client.
- [ ] C) Ils ont un accès complet aux hooks d'état du navigateur (`useState`, `useEffect`, `useRef`).
- [ ] D) Ils nécessitent obligatoirement d'être hydratés côté client par React DOM.

---

### Question 7 : Invalidation du cache de rendu & Références
Soit le composant suivant :

```tsx
const HeavyChart = React.memo(({ options }: { options: { range: number } }) => {
  return <div>Rendu complexe</div>;
});

function Dashboard() {
  const [theme, setTheme] = useState('dark');
  const chartOptions = { range: 30 };

  return (
    <div>
      <button onClick={() => setTheme(t => t === 'dark' ? 'light' : 'dark')}>Toggle Theme</button>
      <HeavyChart options={chartOptions} />
    </div>
  );
}
```

Lorsque l'utilisateur clique sur le bouton pour changer de thème, pourquoi `HeavyChart` se réexécute-t-il malgré `React.memo` ?

- [ ] A) Parce que `React.memo` ne fonctionne pas avec TypeScript.
- [ ] B) Parce que `setTheme` invalide systématiquement l'ensemble de l'arbre DOM complet sans diffing.
- [ ] C) Parce qu'à chaque rendu de `Dashboard`, un nouvel objet littéral `{ range: 30 }` est alloué en mémoire, ce qui produit `options !== prevProps.options` lors de la comparaison de surface (*shallow comparison* `Object.is`).
- [ ] D) Parce que `HeavyChart` ne possède pas de prop `key`.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** La structure d'un nœud Fiber repose sur trois pointeurs principaux : `child` pointe vers son premier descendant direct, `sibling` pointe vers le frère direct suivant au même niveau, et `return` pointe vers le parent (servant d'adresse de retour). Cette conception transforme une récursion d'appels de fonction en une liste chaînée linéaire traversée par une boucle `while (workInProgress !== null)`.

### 2. Réponse : **A**
**Explication :** Durant la phase de **Render**, React construit et manipule l'arbre Fiber *Work-in-Progress* en mémoire sans toucher au DOM de la page. Si une tâche plus prioritaire arrive, React peut abandonner cet arbre en cours sans conséquence visuelle. En revanche, la phase de **Commit** applique les changements réels dans le DOM (`appendChild`, `removeChild`, etc.) ; elle doit être strictement synchrone et indivisible pour éviter que le navigateur n'affiche une interface à moitié modifiée (*visual tearing*).

### 3. Réponse : **B**
**Explication :** `useTransition` attribue à la mise à jour une *TransitionLane* (priorité basse). Le Scheduler configure une tranche de temps (*time-slicing*) et vérifie régulièrement (`shouldYield()`) si des entrées utilisateur prioritaires (frappe au clavier assignée à une *SyncLane*) sont survenues. Si c'est le cas, le travail sur la transition est suspendu immédiatement au profit de l'entrée utilisateur.

### 4. Réponse : **B**
**Explication :** Envelopper une fonction dans `useCallback` sans enfant mémoïsé est souvent contre-productif : cela alloue quand même une fonction anonyme à chaque rendu, alloue le tableau de dépendances et exécute les vérifications `Object.is`. L'utilité principale de `useCallback` est de préserver l'égalité référentielle d'une fonction passée en prop à un composant optimisé par `React.memo` ou passée comme dépendance à un hook.

### 5. Réponse : **B**
**Explication :** Dans un environnement concurrent avec interruption de rendu, si un store externe (comme Redux, Zustand ou un observable fait maison) change de valeur pendant qu'un composant A a été rendu mais avant que le composant B ne le soit, deux composants de la même vue peuvent afficher des données incompatibles (tearing). L'API `useSyncExternalStore` résout ce problème en forçant une synchronisation stricte et en déclenchant un re-render unifié si une mutation survient pendant le calcul.

### 6. Réponse : **B**
**Explication :** Les RSC ne sont jamais envoyés sous forme de code exécutable au client. Ils tournent sur le serveur, accèdent directement aux bases de données ou au système de fichiers, et génèrent un flux sérialisé (RSC payload) qui décrit la structure de l'interface. Cela réduit drastiquement la taille du bundle JavaScript et supprime le besoin d'hydratation pour ces composants.

### 7. Réponse : **C**
**Explication :** `React.memo` effectue par défaut une comparaison superficielle (*shallow equality* avec `Object.is`) entre les anciennes et les nouvelles props. À chaque exécution de `Dashboard`, l'expression `const chartOptions = { range: 30 }` crée une nouvelle référence d'objet en mémoire. Pour `React.memo`, `prevProps.options !== nextProps.options`, ce qui force le re-render. La solution consiste à sortir l'objet de la fonction ou à l'envelopper dans `useMemo(() => ({ range: 30 }), [])`.