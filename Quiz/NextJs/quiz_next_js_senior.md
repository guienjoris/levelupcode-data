# Quiz d'Évaluation — Next.js Niveau Senior & Staff Engineer

Ce quiz évalue votre compréhension de l'architecture distribuée de Next.js, de la topologie de ses 4 caches, du Partial Prerendering (PPR), de l'Edge Runtime, ainsi que des patterns de routage complexes et de la sécurité des Server Actions.

---

### Question 1 : Topologie des caches et Request Memoization

Dans un arbre de composants Server Components, le composant `<UserProfile />` et le composant `<HeaderNav />` exécutent tous deux l'instruction :

```typescript
await fetch('https://api.internal/user/me', { next: { revalidate: 60 } });
```

Que se passe-t-il exactement sur le serveur lors du traitement de cette requête HTTP unique reçue par l'utilisateur ?

- [ ] A) Next.js lance deux appels réseau HTTP parallèles vers `api.internal` et stocke deux entrées séparées dans le Data Cache.
- [ ] B) Le premier appel exécute la requête réseau et met en cache la réponse dans la couche de *Request Memoization* (React cache) ; le second appel lit le résultat en mémoire sans relancer de requête HTTP au sein de ce même cycle de rendu.
- [ ] C) Le moteur lève une erreur de concurrence car deux Server Components ne peuvent pas requêter le même endpoint simultanément.
- [ ] D) Le compilateur déplace automatiquement ces requêtes dans le client via un `useEffect`.

---

### Question 2 : Invalidation granulaire avec Cache Tags

Soit une application e-commerce à fort trafic. Vous souhaitez invalider les données mises en cache pour un entrepôt spécifique sans forcer la reconstruction de l'intégralité du catalogue ni vider le cache des autres entrepôts.

Quelle est l'approche optimale recommandée ?

- [ ] A) Appeler `revalidatePath('/[...all]')` au sein du layout racine.
- [ ] B) Associer un tag unique `tags: ['warehouse-123']` dans l'option `next` du `fetch()`, puis appeler `revalidateTag('warehouse-123')` dans la Server Action de mise à jour.
- [ ] C) Définir `cache: 'no-store'` sur toutes les requêtes du site.
- [ ] D) Supprimer manuellement le dossier `.next/cache` sur le serveur de production via un script bash.

---

### Question 3 : Partial Prerendering (PPR)

Quelle est la proposition technique exacte résolue par le **Partial Prerendering (PPR)** par rapport au SSR et au SSG classiques ?

- [ ] A) Il oblige l'intégralité de l'application à tourner exclusivement sur WebAssembly.
- [ ] B) Il supprime totalement le code JavaScript envoyé au navigateur en transformant l'ensemble des interactions en requêtes HTML `POST`.
- [ ] C) Il combine une enveloppe statique pré-générée au build (servie instantanément depuis le CDN/Edge) avec des zones dynamiques délimitées par `<Suspense>` qui sont streamées à travers la même connexion HTTP sans bloquer le TTFB.
- [ ] D) Il télécharge les composants React uniquement lorsque l'utilisateur scrolle jusqu'à 80 % de la hauteur de page.

---

### Question 4 : Contraintes architecturales du Middleware sur l'Edge Runtime

Pourquoi l'utilisation de modules comme `fs` (Node.js File System) ou de bibliothèques nécessitant des binaires natifs en C++ (comme certains drivers de bases de données traditionnelles) est-elle proscrite dans un fichier `middleware.ts` standard ?

- [ ] A) Parce que le middleware s'exécute sur le moteur V8 isolé ou le runtime Edge (V8 isolates), qui n'implémente pas les APIs de système de fichiers ni l'architecture de processus de Node.js complet.
- [ ] B) Parce que TypeScript refuse d'importer des modules natifs dans les fichiers dont le nom commence par la lettre `m`.
- [ ] C) Parce que le navigateur interdit l'envoi de fichiers locaux vers le middleware.
- [ ] D) Parce que Next.js compile le middleware exclusivement en Web Workers côté client.

---

### Question 5 : Parallel & Intercepting Routes

Dans la structure de dossiers suivante :

```
app/
└── feed/
    ├── @modal/
    │   ├── (.)photo/[id]/
    │   │   └── page.tsx
    │   └── default.tsx
    ├── layout.tsx
    ├── page.tsx
    └── photo/[id]/
        └── page.tsx
```

Que se passe-t-il lorsqu'un utilisateur consulte `/feed` puis effectue un rafraîchissement complet du navigateur (F5 / Hard Reload) après avoir ouvert une photo dont l'URL affichée est devenue `/feed/photo/42` ?

- [ ] A) L'application affiche une erreur 404 car la route interceptée `(.)photo/[id]` n'existe plus.
- [ ] B) Next.js recharge la page de fond `/feed` et ré-ouvre automatiquement la modale par-dessus.
- [ ] C) Next.js sert directement le composant pleine page défini dans `app/feed/photo/[id]/page.tsx`, sans passer par le slot `@modal`.
- [ ] D) L'utilisateur est immédiatement redirigé vers la page racine `/`.

---

### Question 6 : Sécurité et surface d'attaque des Server Actions

Du point de vue de la sécurité applicative (AppSec), comment une **Server Action** déclarée avec `'use server'` doit-elle être considérée par un développeur Senior ?

- [ ] A) Comme une fonction interne totalement sécurisée et privée, inaccessible depuis l'extérieur du code client compilé.
- [ ] B) Comme un endpoint HTTP `POST` public exposé sur Internet, nécessitant une validation rigoureuse des schémas d'entrée (ex: Zod) et une vérification d'authentification/autorisation explicite.
- [ ] C) Comme une procédure stockée exécutée directement au sein du moteur SQL de la base de données.
- [ ] D) Comme un appel sécurisé par chiffrement matériel imperméable aux injections.

---

### Question 7 : Fuite de secrets et paquet `server-only`

Quel est le rôle exact de l'import `import 'server-only'` placé au sommet d'un fichier utilitaire d'accès aux données (ex: `lib/db.ts`) ?

- [ ] A) Augmenter la vitesse d'exécution des requêtes SQL sur le serveur.
- [ ] B) Lever une erreur immédiate au moment du build (ou de la compilation) si ce module est importé accidentellement dans un Client Component, empêchant ainsi la fuite de secrets ou de code serveur dans le bundle client.
- [ ] C) Désactiver le ramasse-miettes de Node.js pour garder les connexions persistantes.
- [ ] D) Rendre le composant compatible avec l'API WebSockets.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** La **Request Memoization** est une optimisation intégrée au cycle de vie de rendu de React et Next.js. Pour une même requête HTTP entrante, les appels `fetch` identiques (même URL et mêmes options) sont mémorisés en mémoire. Le premier appel récupère la donnée (ou consulte le Data Cache) et les suivants récupèrent le résultat instantanément en mémoire sans surcoût réseau, rendant inutile le *prop drilling*.

### 2. Réponse : **B**
**Explication :** Les **Cache Tags** permettent une invalidation chirurgicale et découplée des URLs. En taguant la requête (`tags: ['warehouse-123']`), `revalidateTag('warehouse-123')` ne purge que les données associées à cet identifiant, sans toucher au reste du Full Route Cache ni invalider d'autres entrepôts.

### 3. Réponse : **C**
**Explication :** Le **PPR (Partial Prerendering)** élimine le compromis entre statique et dynamique. Le shell statique de la page est servi avec un TTFB ultra-faible (proche de 0ms) depuis le CDN/Edge, tandis que les composants dynamiques enveloppés dans `<Suspense>` continuent de résoudre leurs promesses sur le serveur et sont streamés dans le même flux HTTP au fur et à mesure de leur complétion.

### 4. Réponse : **A**
**Explication :** Le middleware s'exécute sur le runtime **Edge** (basé sur des isolats V8 légers pour un démarrage à froid quasi nul et une exécution au plus près de l'utilisateur). Ce runtime n'est pas un environnement Node.js complet : il ne dispose pas des modules bas niveau (`fs`, `child_process`, `net`) ni de la compatibilité avec les extensions natives en C++.

### 5. Réponse : **C**
**Explication :** Les routes interceptées (comme `(.)photo/[id]`) ne s'activent que lors de la navigation logicielle côté client (*soft navigation* via `<Link>`). Si l'utilisateur effectue un rechargement dur (*hard navigation* / F5) ou partage l'URL directement, Next.js sert la page classique définie dans `photo/[id]/page.tsx`, offrant naturellement une version complète autonome adaptée au partage de lien.

### 6. Réponse : **B**
**Explication :** Chaque Server Action génère sous le capot un identifiant RPC accessible via une requête HTTP `POST` standard. N'importe quel attaquant peut forger un appel HTTP vers cette action avec un payload arbitraire. Il est donc critique de traiter les Server Actions comme des APIs publiques : validation systématique du payload (via Zod, Valibot, etc.) et contrôle strict des permissions et de la session utilisateur.

### 7. Réponse : **B**
**Explication :** Le paquet `server-only` injecte une contrainte d'analyse statique au niveau du bundler. Si un fichier contenant `import 'server-only'` (ou l'un de ses importateurs) se retrouve importé dans l'arbre d'un composant marqué avec `'use client'`, la compilation échoue immédiatement, prévenant toute fuite fortuite de variables d'environnement non publiques (`process.env.SECRET_KEY`) vers le navigateur.