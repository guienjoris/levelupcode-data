# Quiz d'Évaluation — Next.js (Niveau Novice : Conventions, UI & SEO)

Ce quiz évalue votre compréhension des conventions de routage et des fonctionnalités d'optimisation de Next.js App Router : les fichiers spéciaux (`loading.tsx`, `error.tsx`, `not-found.tsx`), les composants natifs `<Link>` et `<Image>`, la gestion des polices avec `next/font` et l'API de métadonnées.

### Question 1 : Spécificité du fichier de gestion des erreurs

Quelle contrainte technique obligatoire s'applique lors de la création d'un fichier `error.tsx` dans l'App Router de Next.js ?

* [ ] A) Il doit obligatoirement être un composant exécuté côté client en débutant par la directive `'use client'`.

* [ ] B) Il doit nécessairement exporter une fonction asynchrone retournant une promesse de code de statut HTTP.

* [ ] C) Il doit obligatoirement être placé à la racine exacte du dossier `public/` pour être lisible par le navigateur.

* [ ] D) Il ne peut être utilisé que si l'application est compilée en mode export statique pur sans serveur Node.

### Question 2 : Fonctionnement du composant de navigation `<Link>`

Quel mécanisme interne confère au composant `<Link>` de Next.js une vitesse de transition supérieure à une simple balise `<a href="...">` ?

* [ ] A) Il remplace l'ensemble des requêtes réseau par une synchronisation locale exploitant le stockage IndexedDB.

* [ ] B) Il désactive temporairement l'exécution des scripts CSS pendant toute la phase de transition entre deux routes.

* [ ] C) Il intercepte le clic pour naviguer côté client et précharge en arrière-plan les pages visibles dans le viewport.

* [ ] D) Il télécharge l'intégralité du code source de l'application dès que l'utilisateur charge la première page.

### Question 3 : Prévention du décalage de mise en page avec `<Image />`

Comment le composant `<Image />` de Next.js parvient-il à éliminer le décalage cumulatif de mise en page (*Cumulative Layout Shift* ou CLS) ?

* [ ] A) En convertissant automatiquement toutes les images matricielles (PNG, JPEG) en tracés vectoriels SVG légers.

* [ ] B) En exigeant la définition des dimensions `width` et `height` pour réserver immédiatement l'espace dans le layout.

* [ ] C) En retardant l'affichage complet de la page HTML tant que tous les fichiers visuels ne sont pas téléchargés.

* [ ] D) En appliquant un flou gaussien permanent qui masque les variations de dimensions sur les écrans mobiles.

### Question 4 : Gestion des polices avec `next/font`

Quel bénéfice majeur apporte l'utilisation de `next/font/google` par rapport à un import classique de police via une balise `<link>` pointant sur les serveurs Google Fonts ?

* [ ] A) Les fichiers de police sont téléchargés au moment du build et hébergés localement sans envoyer de requête tierce à Google.

* [ ] B) Il génère dynamiquement des polices vectorielles personnalisées selon la langue déclarée dans le navigateur client.

* [ ] C) Il compresse les fichiers typographiques en mémoire vive pour diviser par deux l'utilisation du processeur client.

* [ ] D) Il autorise l'utilisation de typographies protégées par des droits d'auteur sans aucune licence préalable requise.

### Question 5 : Déclenchement programmable d'une page 404

Dans un Server Component de type `app/articles/[id]/page.tsx`, quelle instruction permet d'interrompre le rendu et d'afficher la vue `not-found.tsx` la plus proche lorsqu'un identifiant est absent de la base de données ?

* [ ] A) `throw new Error("HTTP 404 Not Found")`

* [ ] B) `return Response.redirect('/404', 302)`

* [ ] C) `notFound()` importée depuis `next/navigation`

* [ ] D) `process.exit(404)`

### Question 6 : Métadonnées dynamiques et SEO

Dans l'App Router, comment génère-t-on des balises `<title>` et Open Graph dépendantes de données récupérées sur une API distante pour une route dynamique ?

* [ ] A) En manipulant directement l'objet global du navigateur `window.document.title` au sein du hook `useEffect`.

* [ ] B) En exportant une fonction asynchrone dédiée nommée `generateMetadata` recevant les paramètres de route en argument.

* [ ] C) En insérant une balise HTML `<head>` personnalisée directement dans le JSX retourné par le composant racine du layout.

* [ ] D) En déclarant un fichier statique `seo.config.json` dans chaque répertoire de route concerné par le référencement.

## Corrigé et Explications Détaillées

### 1. Réponse : **A**

**Explication :** Dans l'architecture de Next.js, les frontières de capture d'erreurs (*Error Boundaries*) reposent sur les méthodes de cycle de vie de composants React (`componentDidCatch` / `getDerivedStateFromError`), lesquelles ne fonctionnent que côté client. De plus, `error.tsx` doit pouvoir réagir aux actions utilisateur (comme le clic sur le bouton `reset()` pour relancer le rendu). Par conséquent, ce fichier doit toujours démarrer avec la directive `'use client'`.

### 2. Réponse : **C**

**Explication :** Contrairement à une balise `<a>` classique qui recharge l'ensemble du document HTML et réinitialise l'application, `<Link>` réalise une navigation côté client (SPA). En environnement de production, Next.js détecte automatiquement quand un `<Link>` entre dans la zone visible de l'écran (*viewport*) et déclenche un préchargement silencieux (*prefetch*) de la route cible et de ses données, rendant la transition quasiment instantanée au moment du clic.

### 3. Réponse : **B**

**Explication :** Le décalage de mise en page (*Cumulative Layout Shift*) survient quand le navigateur ne connaît pas les dimensions d'une image avant son téléchargement : le texte est d'abord positionné, puis décalé brutalement dès que l'image apparaît. En imposant la présence de `width` et `height` (ou de la propriété `fill`), `<Image />` calcule le ratio d'affichage et force le navigateur à réserver l'espace requis dans le DOM dès le premier rendu.

### 4. Réponse : **A**

**Explication :** Les imports Google Fonts traditionnels forcent le navigateur de l'utilisateur à contacter les serveurs de Google lors du chargement, ce qui ajoute une résolution DNS supplémentaire, ralentit le premier affichage du texte et pose des questions de conformité RGPD. Le module `next/font` résout cela en téléchargeant les fichiers de police lors de l'étape de compilation (`build`) : ils sont ensuite servis directement depuis le même domaine que le reste de l'application statique.

### 5. Réponse : **C**

**Explication :** La fonction `notFound()` issue de `next/navigation` lève une exception interne gérée par le runtime Next.js. Elle court-circuite immédiatement l'exécution du Server Component pour afficher le composant de vue défini dans le fichier conventionné `not-found.tsx` du segment courant (ou celui situé plus haut dans l'arborescence), tout en retournant un code de statut HTTP 404 approprié aux robots d'indexation.

### 6. Réponse : **B**

**Explication :** Next.js met à disposition l'API `generateMetadata`. Pour une route dynamique (ex: `app/articles/[id]/page.tsx`), exporter cette fonction asynchrone permet de récupérer le paramètre de route (`params`), d'effectuer la requête API nécessaire et de retourner un objet `Metadata` conforme. Next.js fusionne ensuite ces données côté serveur pour injecter les balises `<title>`, `<meta name="description">` et les balises Open Graph dans le flux HTML envoyé au client.