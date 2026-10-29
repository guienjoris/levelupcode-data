# Quiz d'Évaluation — Principes de Conception Logicielle (Niveau Intermédiaire)

Ce quiz évalue votre compréhension des concepts d'architecture intermédiaire : couplage et cohésion, composition vs héritage, la loi de Déméter, le principe *Tell, Don't Ask* et la séparation commande-requête (CQS).

### Question 1 : Couplage vs Cohésion

Dans une revue de code, vous remarquez une classe `OrderManager` qui valide les cartes bancaires, génère du HTML pour les factures, insère des lignes dans PostgreSQL et envoie des SMS aux livreurs. Quel diagnostic architectural portez-vous ?

* [ ] A) La classe souffre d'un excès de couplage temporel car les opérations sont exécutées séquentiellement dans une même boucle asynchrone sans mécanisme de rollback distribué.
* [ ] B) La classe présente une faible cohésion.
* [ ] C) La classe illustre parfaitement le principe CQS en consolidant l'ensemble des commandes et requêtes du domaine de facturation.
* [ ] D) Le couplage est trop faible vis-à-vis de l'infrastructure de base de données.

### Question 2 : Composition over Inheritance

Pourquoi la composition d'objets est-elle généralement préférée à l'héritage de classes (`extends`) pour faire évoluer des comportements métier complexes ?

* [ ] A) Elle permet d'assembler dynamiquement des comportements interchangeables à l'exécution sans figer une hiérarchie rigide ni subir les effets de bord de modifications dans une classe parente.
* [ ] B) L'héritage est formellement interdit par les normes ECMAScript et TypeScript depuis 2024 pour des raisons de gestion de mémoire dans le ramasse-miettes V8.
* [ ] C) La composition double la vitesse d'exécution des méthodes polymorphes au runtime grâce au inline caching du compilateur JIT.
* [ ] D) Elle supprime l'usage des interfaces.

### Question 3 : Loi de Déméter (*Law of Demeter*)

Considérez l'instruction suivante :

```typescript
const postalCode = order.getCustomer().getProfile().getAddress().getZipCode();
```

Quelle est la critique principale adressée à cette ligne au regard de la loi de Déméter ?

* [ ] A) Elle entraîne un coût CPU prohibitif car chaque appel d'accesseur alloue un nouveau pointeur d'objet sur la pile d'exécution système.
* [ ] B) Elle produit un « train wreck » (accident ferroviaire) en couplant le code appelant à la structure interne et aux chemins d'accès de 4 niveaux d'objets intermédiaires.
* [ ] C) Elle viole le principe KISS uniquement si la variable `postalCode` n'est pas typée explicitement avec TypeScript.
* [ ] D) Elle ne pose aucun problème architectural tant que les méthodes sont préfixées par `get`.

### Question 4 : Principe *Tell, Don't Ask*

Lequel de ces extraits illustre le **mieux** l'application rigoureuse du principe *Tell, Don't Ask* ?

* [ ] A) 
```typescript
if (cart.getItems().length > 0 && !cart.getIsLocked()) {
  cart.setDiscount(0.1);
  cart.recalculateTotal();
}
```
* [ ] B)
```typescript
cart.applyDiscount(0.1);
```
* [ ] C)
```typescript
const items = cart.items;
const total = items.reduce((acc, i) => acc + i.price, 0);
cart.totalWithDiscount = total * 0.9;
```
* [ ] D)
```typescript
if (cart.canApplyDiscount()) {
  const currentTotal = cart.getTotal();
  cart.saveNewTotal(currentTotal * 0.9);
}
```

### Question 5 : Command-Query Separation (CQS)

Selon le principe CQS formulé par Bertrand Meyer, quelle est l'anomalie dans la signature de méthode suivante :

```typescript
class AccountService {
  /** Vérifie le mot de passe, génère un token JWT, met à jour le compteur d'échecs et renvoie le User */
  authenticateAndRecord(credentials: Credentials): User;
}
```

* [ ] A) La méthode n'accepte qu'un seul argument, ce qui enfreint la règle des fonctions unaires en programmation fonctionnelle pure.
* [ ] B) Elle ne retourne pas une promesse asynchrone, ce qui bloque le thread principal de Node.js lors de la sérialisation JSON.
* [ ] C) Elle mélange une requête (lecture et renvoi de l'utilisateur) avec des commandes (mutations d'état en base et génération de token), masquant ainsi des effets de bord sous une opération de lecture.
* [ ] D) Elle devrait obligatoirement renvoyer un booléen pour respecter le typage strict des requêtes.

### Question 6 : Fuite d'invariants et encapsulation

Que risque une application lorsque les modèles de domaine exposent directement leurs tableaux ou objets mutables internes via des getters publics (ex: `public getItems(): Item[] { return this.items; }`) ?

* [ ] A) Une violation d'encapsulation où du code externe peut muter le tableau directement (ex: `push()`) sans passer par les validations métier de l'entité.
* [ ] B) Une dégradation inévitable du garbage collector causée par la duplication automatique des références mémoire en zone statique.
* [ ] C) Une erreur de syntaxe levée par le compilateur TypeScript en mode strict.
* [ ] D) Une baisse de couverture de tests.

### Question 7 : Couplage afférent et efférent

Dans l'analyse métrique d'un composant, qu'indique un composant possédant une **forte instabilité** (beaucoup de couplage sortant / efférent, et très peu de couplage entrant / afférent) ?

* [ ] A) Le composant est critique et tous les autres modules de l'application dépendent directement de lui.
* [ ] B) Le composant dépend de nombreux autres modules pour fonctionner et est donc très sensible aux changements de son environnement, mais sa propre modification a peu d'impact sur le reste du système.
* [ ] C) Le module utilise trop de variables globales partagées au niveau de la mémoire vive du serveur.
* [ ] D) C'est un composant exempt de tout bug potentiel.

---

## Corrigé et Explications Détaillées

### 1. Réponse : **B**
**Explication :** La classe `OrderManager` prend en charge des domaines complètement hétérogènes (paiement, templating HTML, persistance SQL, notification SMS). Ses responsabilités ne concourent pas à un objectif unique : elle a une **faible cohésion** (et viole par la même occasion le principe de responsabilité unique SRP).

### 2. Réponse : **A**
**Explication :** La composition permet d'injecter des comportements sous forme de modules ou d'interfaces modulaires (`has-a`). Cela évite les hiérarchies rigides en diamant causées par l'héritage (`is-a`) et empêche qu'un changement dans une classe parente ne casse silencieusement les sous-classes dérivées.

### 3. Réponse : **B**
**Explication :** Le chaînage excessif d'accesseurs crée une dépendance directe avec les détails d'implémentation de toute la chaîne. Si le modèle de données change (par exemple si l'adresse est directement rattachée au profil ou au compte), ce code casse. La loi de Déméter préconise d'interagir uniquement avec l'objet direct (`order.getShippingZipCode()`).

### 4. Réponse : **B**
**Explication :** L'option B ordonne directement au panier d'agir (`cart.applyDiscount(0.1)`). L'objet panier protège ses propres invariants (vérifier s'il est verrouillé, recalculer son total, valider la remise) sans forcer le code client à inspecter (`ask`) son état interne avant de lui dicter quoi faire.

### 5. Réponse : **C**
**Explication :** Le principe CQS impose une distinction nette : une requête (*Query*) interroge le système sans altérer son état ; une commande (*Command*) mute l'état sans renvoyer de données métier. Mélanger la mise à jour des compteurs et la génération de session au sein d'une fonction qui renvoie un objet crée des effets de bord implicites.

### 6. Réponse : **A**
**Explication :** Retourner une référence directe vers une collection interne permet à n'importe quel code consommateur d'ajouter ou retirer des éléments sans déclencher les contrôles d'intégrité de l'entité (règles de prix, limites d'articles, calculs d'états). Il est préférable de renvoyer une copie en lecture seule (`[...this.items]` ou `ReadonlyArray`).

### 7. Réponse : **B**
**Explication :** Selon les métriques de Robert C. Martin, l'instabilité $I$ se calcule par $I = \frac{C_e}{C_a + C_e}$ (où $C_e$ est le couplage efférent/sortant et $C_a$ le couplage afférent/entrant). Un module avec beaucoup de dépendances sortantes ($C_e$ élevé) et peu d'entrants ($C_a$ faible) a une instabilité proche de 1 : il change dès qu'une de ses dépendances bouge, mais son changement n'affecte presque personne d'autre.