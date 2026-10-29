# Initiation aux Architectures Logicielles (Niveau Novice)

Lorsqu'on commence à programmer, on se concentre sur la syntaxe, les boucles et les fonctions. Mais dès qu'un projet grandit, une question devient vitale : **comment organiser l'ensemble du système pour qu'il reste compréhensible, testable et évolutif ?**

C'est le rôle de l'**architecture logicielle** : elle définit la structure globale d'une application, la répartition des responsabilités et la manière dont les différents modules communiquent entre eux.

---

## 1. Qu'est-ce que l'architecture logicielle ?

L'analogie avec le bâtiment est parlante :
* **Le code (la plomberie et l'électricité) :** les détails d'implémentation (fonctions, variables, algorithmes).
* **L'architecture (le plan de l'immeuble) :** l'emplacement des pièces, les voies d'accès, la séparation entre espaces publics et privés, et la structure porteuse.

Une bonne architecture sert trois objectifs :
1. **Maintenabilité :** corriger un bug dans un module sans en déclencher trois autres ailleurs.
2. **Évolutivité (*scalabilité*) :** supporter plus d'utilisateurs ou ajouter de nouvelles fonctionnalités facilement.
3. **Collaboration :** permettre à plusieurs développeurs de travailler sur des parties différentes sans se marcher sur les pieds.

---

## 2. Le Modèle Fondamental : Client-Serveur

La quasi-totalité des applications web et mobiles modernes repose sur le modèle **Client-Serveur**.

```
┌─────────────────────────┐                        ┌─────────────────────────┐
│         CLIENT          │   Requête HTTP (GET)   │         SERVEUR         │
│  (Navigateur / Mobile)  ├───────────────────────►│    (Node.js, Python...)   │
│                         │                        │                         │
│  - Affiche l'interface  │◄───────────────────────┤  - Exécute les règles   │
│  - Réagit aux clics     │   Réponse JSON / HTML  │  - Accède aux données   │
└─────────────────────────┘                        └────────────┬────────────┘
                                                                │
                                                                ▼ Requête SQL
                                                   ┌─────────────────────────┐
                                                   │    BASE DE DONNÉES      │
                                                   │   (PostgreSQL, MySQL)   │
                                                   └─────────────────────────┘
```

* **Le Client (*Frontend*) :** c'est ce que voit l'utilisateur (HTML, CSS, JavaScript, application iOS/Android). Il collecte les actions de l'utilisateur et affiche les résultats.
* **Le Serveur (*Backend*) :** c'est le "cerveau" protégé. Il vérifie l'identité de l'utilisateur, applique les règles métier (ex: "vérifier si le stock est suffisant") et calcule les réponses.
* **La Base de Données (*Database*) :** le coffre-fort où les données sont conservées de façon permanente (comptes, commandes, articles).

---

## 3. L'Architecture en Couches (*Layered / N-Tier*)

Pour éviter de mélanger l'affichage visuel et les requêtes SQL dans un seul fichier, on découpe l'application en **couches horizontales**. Chaque couche n'a qu'un rôle précis et ne discute qu'avec sa voisine directe.

```
┌────────────────────────────────────────────────────────┐
│  1. Couche de Présentation (UI / Contrôleur)          │  <- Reçoit la requête
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│  2. Couche Métier / Domaine (Services / Règles)        │  <- Applique la logique
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│  3. Couche d'Accès aux Données (Repositories / DAO)    │  <- Lit / Écrit en base
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
                     [ Base de Données ]
```

### Exemple concret : une commande sur un site e-commerce

1. **Présentation :** l'utilisateur clique sur "Payer". Le contrôleur reçoit la requête HTTP, vérifie que le panier n'est pas vide et transmet l'ordre au service métier.
2. **Métier :** le service vérifie si le compte bancaire est valide, applique un coupon de réduction et calcule les taxes.
3. **Données :** le *repository* génère la requête SQL pour enregistrer la commande dans la table `orders`.

**Pourquoi ce découpage ?** Si vous décidez de changer de base de données (passer de MySQL à MongoDB), seule la couche 3 est impactée. La couche métier et la couche visuelle ne changent pas.

---

## 4. Le Pattern MVC (*Modèle - Vue - Contrôleur*)

Le pattern MVC est l'une des déclinaisons les plus populaires de l'architecture en couches, utilisée par des frameworks comme Django, Laravel, Ruby on Rails ou ASP.NET.

```
       [ Utilisateur ]
              │ Interagit avec
              ▼
       ┌──────────────┐
       │  Contrôleur  ├──────────────┐ Demande la mise à jour
       └──────┬───────┘              │
              │ Demande d'afficher   ▼
              │                ┌───────────┐
              ▼                │  Modèle   │ (Données & Logique)
       ┌──────────────┐        └─────┬─────┘
       │     Vue      │◄─────────────┘ Fournit les données fraîches
       └──────────────┘
```

* **M — Modèle (*Model*) :** représente la donnée et les règles associées (ex: un objet `User` avec ses propriétés et ses validations).
* **V — Vue (*View*) :** ce que l'utilisateur voit à l'écran (le gabarit HTML, l'interface graphique).
* **C — Contrôleur (*Controller*) :** le chef d'orchestre. Il intercepte les clics ou requêtes, demande au Modèle de se mettre à jour, puis choisit quelle Vue afficher en retour.

---

## 5. Monolithe vs Microservices : Le Grand Débat

Quand une entreprise construit un système d'envergure, elle choisit généralement entre deux grandes visions d'organisation :

| Critère | Architecture Monolithique | Architecture Microservices |
| :--- | :--- | :--- |
| **Principe** | L'ensemble du code de l'application est regroupé dans un seul et même projet exécutable. | L'application est découpée en plusieurs petits services autonomes qui communiquent via le réseau (API). |
| **Schéma mental** | Un couteau suisse tout-en-un. | Une boîte à outils contenant plusieurs outils indépendants. |
| **Déploiement** | Simple : on déploie un seul binaire ou conteneur. | Complexe : orchestration nécessaire (Docker, Kubernetes). |
| **Débogage** | Facile en local : tout tourne sur la même machine. | Plus ardu : une panne peut venir du réseau ou d'un service tiers. |
| **Idéal pour...** | 90 % des projets, startups, équipes de petite à moyenne taille. | Très grandes organisations (Netflix, Amazon) avec des dizaines d'équipes autonomes. |

### La règle de sagesse pour débuter

> **Commencez toujours par un monolithe modulaire propre.** 
> Découper prématurément une application en microservices ajoute une immense complexité réseau (latence, pannes partielles, cohérence des données) sans apporter de valeur tangible au démarrage.

---

## 6. Les 4 Réflexes Clés pour Débuter en Architecture

1. **Séparer les responsabilités :** ne placez jamais du code de requête SQL directement au milieu d'un composant visuel ou d'un bouton.
2. **Ne pas tout réinventer :** appuyez-vous sur les conventions de votre framework (Next.js, NestJS, Express, Rails ont tous des structures pensées pour vous guider).
3. **Penser aux flux de données :** ayez toujours en tête le chemin que parcourt une donnée, du clic de l'utilisateur jusqu'au disque dur du serveur.
4. **Rester pragmatique (KISS) :** la meilleure architecture n'est pas la plus complexe, c'est celle qui résout le problème actuel tout en restant facile à modifier.