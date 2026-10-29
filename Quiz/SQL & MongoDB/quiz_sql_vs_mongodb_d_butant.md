# Quiz d'Évaluation — SQL vs MongoDB (Niveau Novice)

Ce quiz évalue votre compréhension des concepts fondamentaux opposant les bases relationnelles (SQL) aux bases orientées documents (MongoDB) : vocabulaire, modèle de données, opérations CRUD et critères de choix.

---

### Question 1 : Équivalence du vocabulaire

Dans le modèle orienté document de MongoDB, quel concept correspond directement à une **table** du monde relationnel SQL ?

* [ ] A) Une collection
* [ ] B) Un document BSON
* [ ] C) Une clé primaire composite
* [ ] D) Un schéma relationnel strict

---

### Question 2 : Comportement de la mise à jour dans MongoDB

Lors de l'exécution de la commande suivante dans MongoDB :
`db.users.updateOne({ email: "test@domain.com" }, { age: 30 });`
Quel sera le résultat pour le document ciblé ?

* [ ] A) Seul le champ `age` est mis à jour tandis que tous les autres attributs existants sont parfaitement préservés.
* [ ] B) Le document entier est écrasé et remplacé par le seul champ `age` car l'opérateur `$set` a été omis.
* [ ] C) MongoDB refuse la commande et lève une exception de validation de contrainte de schéma.
* [ ] D) L'opération insère systématiquement un nouveau document doublon en bout de collection.

---

### Question 3 : Gestion des jointures et relations

Comment gère-t-on traditionnellement une relation de type 1 à N fini (ex. un panier et ses quelques lignes d'articles) dans MongoDB sans créer de table dédiée ?

* [ ] A) En exploitant une transaction SQL `JOIN` exécutée via un proxy ODBC branché sur le cluster.
* [ ] B) En imbriquant directement une liste d'objets articles au sein du document de commande principal.
* [ ] C) En déclarant une clé étrangère obligatoire munie de la clause de contrainte `ON DELETE CASCADE`.
* [ ] D) En forçant la conversion du document en une table mémoire temporaire indexée sur l'identifiant.

---

### Question 4 : Prudence lors des opérations en SQL

Que se passe-t-il si vous exécutez par mégarde l'instruction `DELETE FROM users;` sans spécifier de clause `WHERE` ?

* [ ] A) Le serveur SQL refuse d'exécuter la requête par sécurité et bloque automatiquement la session.
* [ ] B) Seule la toute première ligne de la table est purgée de l'espace de stockage physique.
* [ ] C) L'ensemble des lignes de la table `users` est immédiatement et définitivement supprimé.
* [ ] D) La structure complète de la table est détruite et doit être recréée avec `CREATE TABLE`.

---

### Question 5 : Typage et structure des données

Quelle affirmation décrit correctement la différence de schéma entre une table relationnelle SQL et une collection MongoDB ?

* [ ] A) Les tables SQL exigent un schéma prédéfini et uniforme, alors que MongoDB accepte des documents aux champs hétérogènes.
* [ ] B) MongoDB interdit totalement l'ajout de nouveaux champs dès lors que le premier document de la collection est inséré.
* [ ] C) Les bases SQL ne permettent pas de stocker des chaînes de caractères de longueur variable supérieure à 255 octets.
* [ ] D) Les deux systèmes imposent obligatoirement la même structure stricte pour chaque enregistrement persisté sur disque.

---

### Question 6 : Critère de choix technologique

Dans quelle situation le choix d'un SGBD relationnel SQL (comme PostgreSQL ou MySQL) est-il généralement le plus recommandé face à MongoDB ?

* [ ] A) Pour consigner de gros volumes de logs serveurs semi-structurés ne nécessitant aucune relation logique.
* [ ] B) Lorsque l'application traite des opérations financières critiques exigeant des transactions ACID strictes et sans faille.
* [ ] C) Quand le projet est un prototype jetable dont les propriétés changent chaque jour sans modèle précis.
* [ ] D) Afin d'enregistrer des documents JSON bruts sans avoir à définir de tables ni de colonnes au préalable.

---

### Question 7 : Filtrage d'intervalles

Quelle instruction MongoDB équivaut fidèlement à la clause SQL `WHERE age >= 21` ?

* [ ] A) `{ age: { $gte: 21 } }`
* [ ] B) `{ age: { $gt: 21 } }`
* [ ] C) `{ age: { $lte: 21 } }`
* [ ] D) `{ age: { $in: [21] } }`

---

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** En MongoDB, les documents sont regroupés au sein d'une **collection**, ce qui correspond fonctionnellement à une table en SQL. Le document correspond à la ligne (*row*), et le champ à la colonne (*column*).

### 2. Réponse : **B**
**Explication :** Sans l'opérateur `$set` (ou un opérateur atomique équivalent), MongoDB effectue un remplacement complet du document par le nouvel objet fourni. Pour modifier uniquement un attribut sans écraser le reste, il faut impérativement écrire : `{ $set: { age: 30 } }`.

### 3. Réponse : **B**
**Explication :** L'un des atouts de MongoDB est l'imbrication (*embedding*). Pour des relations avec un nombre d'éléments modeste et prévisible, placer le tableau d'objets directement dans le document parent permet de lire l'ensemble de l'information en une seule requête sans jointure.

### 4. Réponse : **C**
**Explication :** Tout comme pour `UPDATE`, un `DELETE` sans clause `WHERE` s'applique à l'intégralité de la table. Pour détruire la structure de la table elle-même, on utiliserait plutôt l'instruction DDL `DROP TABLE users;`.

### 5. Réponse : **A**
**Explication :** SQL repose sur un schéma strict défini en amont (*Schema-on-write*) où chaque ligne doit se conformer aux mêmes colonnes et types. MongoDB est dit *schemaless* ou à schéma flexible : deux documents d'une même collection peuvent présenter des structures et des attributs totalement distincts.

### 6. Réponse : **B**
**Explication :** Les bases SQL traditionnelles excellent dans la garantie des propriétés ACID (Atomicité, Cohérence, Isolation, Durabilité) et l'intégrité référentielle par clés étrangères, indispensables pour la comptabilité, les commandes et les transferts d'argent.

### 7. Réponse : **A**
**Explication :** Dans la syntaxe des opérateurs de requête MongoDB, `$gte` signifie *Greater Than or Equal* ($\ge$). L'opérateur `$gt` correspond quant à lui à une inégalité stricte ($>$).