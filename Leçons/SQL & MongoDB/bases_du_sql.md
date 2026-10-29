# Initiation au SQL et aux Bases de Données Relationnelles (Niveau Novice)

Le langage **SQL** (*Structured Query Language*) est le standard universel utilisé pour dialoguer avec les systèmes de gestion de bases de données relationnelles (**SGBDR** ou *RDBMS*), tels que **PostgreSQL**, **MySQL**, **MariaDB** ou **SQLite**.

Contrairement aux fichiers textes ou aux tableurs classiques, une base de données relationnelle garantit la cohérence, l'intégrité et la persistance des données même en cas de coupure de courant ou d'accès simultanés par des milliers d'utilisateurs.

---

## 1. Modèle Mental : Tables, Lignes et Colonnes

Une base relationnelle organise l'information sous la forme de **tables** bidimensionnelles (semblables à des feuilles de calcul avec des règles strictes).

```
Table: users
┌────────────┬──────────────────┬──────────────────────────┬────────┐
│ id (PK)    │ full_name        │ email                    │ age    │
├────────────┼──────────────────┼──────────────────────────┼────────┤
│ 1          │ Alice Martin     │ alice@example.com        │ 28     │
│ 2          │ Bob Bernard      │ bob@example.com          │ 34     │
│ 3          │ Charlie Dubois   │ charlie@example.com      │ 19     │
└────────────┴──────────────────┴──────────────────────────┴────────┘
```

- **Colonne (*Field / Column*) :** Définit un attribut avec un type de données strict (`INT`, `VARCHAR`, `BOOLEAN`).
- **Ligne (*Row / Tuple / Record*) :** Représente une entité ou un élément unique.
- **Clé Primaire (*Primary Key - PK*) :** Un identifiant unique et non modifiable pour chaque ligne (souvent un nombre auto-incrémenté ou un UUID).

---

## 2. Définir la Structure : DDL (*Data Definition Language*)

Avant d'écrire ou de lire des données, on définit le schéma (*schema*) avec des types et des contraintes.

### 2.1 Les types de données courants
- `INT` / `INTEGER` : Nombres entiers ($42$).
- `NUMERIC(10, 2)` / `DECIMAL` : Nombres décimaux précis pour les montants financiers ($19.99$).
- `VARCHAR(255)` : Chaîne de caractères de longueur variable (avec un maximum).
- `TEXT` : Texte long sans limite arbitraire.
- `BOOLEAN` : `TRUE` ou `FALSE`.
- `TIMESTAMP` / `DATETIME` : Date et heure avec ou sans fuseau horaire.

### 2.2 Création de table et contraintes d'intégrité

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,             -- Auto-incrémenté et unique
  full_name VARCHAR(100) NOT NULL,   -- Obligatoire (ne peut pas être NULL)
  email VARCHAR(255) UNIQUE NOT NULL,-- Pas de doublon possible
  age INT CHECK (age >= 13),         -- Validation de valeur
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 3. Les Opérations CRUD Fondamentales

Le cœur du SQL au quotidien consiste à manipuler les lignes d'une table via quatre opérations : **C**reate, **R**ead, **U**pdate, **D**elete.

### 3.1 Insérer des données (`INSERT INTO`)

```sql
-- Insertion d'une ligne
INSERT INTO users (full_name, email, age)
VALUES ('Alice Martin', 'alice@example.com', 28);

-- Insertion de plusieurs lignes d'un coup
INSERT INTO users (full_name, email, age) VALUES 
('Bob Bernard', 'bob@example.com', 34),
('Charlie Dubois', 'charlie@example.com', 19);
```

### 3.2 Interroger et filtrer (`SELECT`, `WHERE`)

L'ordre de base d'une requête de lecture :

```sql
SELECT full_name, email
FROM users
WHERE age >= 20
ORDER BY full_name ASC
LIMIT 10;
```

#### Les opérateurs de filtrage courants dans `WHERE` :
- Comparaisons : `=`, `!=` (ou `<>`), `<`, `>`, `<=`, `>=`.
- Logique : `AND`, `OR`, `NOT`.
- Recherche textuelle : `LIKE 'A%'` (commence par 'A') ou `ILIKE` (insensible à la casse sur PostgreSQL).
- Intervalles et listes : `BETWEEN 18 AND 65`, `IN ('Alice', 'Bob')`.
- Valeurs nulles : `IS NULL`, `IS NOT NULL` (on n'utilise jamais `= NULL`).

### 3.3 Mettre à jour des données (`UPDATE`)

```sql
UPDATE users
SET age = 29, full_name = 'Alice Martin-Dupont'
WHERE email = 'alice@example.com';
```

> **Règle vitale :** Si vous oubliez la clause `WHERE`, **toutes les lignes** de la table seront modifiées !

### 3.4 Supprimer des données (`DELETE`)

```sql
DELETE FROM users
WHERE id = 3;
```

> **Même avertissement :** Un `DELETE FROM users;` sans `WHERE` vide l'intégralité de la table.

---

## 4. Les Relations et les Clés Étrangères (`FOREIGN KEY`)

La puissance du modèle relationnel réside dans l'interconnexion de tables pour éviter la duplication des données (*normalisation*).

```
Table: users (1) ◄────────── (N) Table: orders
┌────┬──────────────┐            ┌─────┬─────────┬────────┐
│ id │ full_name    │            │ id  │ user_id │ total  │
├────┼──────────────┤            ├─────┼─────────┼────────┤
│ 1  │ Alice Martin │            │ 101 │ 1       │ 45.00  │
│ 2  │ Bob Bernard  │            │ 102 │ 1       │ 12.50  │
└────┴──────────────┘            │ 103 │ 2       │ 89.00  │
                                 └─────┴─────────┴────────┘
```

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  total NUMERIC(10, 2) NOT NULL,
  ordered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
*L'option `ON DELETE CASCADE` garantit que si l'utilisateur 1 est supprimé, ses commandes associées sont automatiquement nettoyées.*

---

## 5. Croiser les Tables : Les Jointures (`JOIN`)

Une jointure rassemble les données de deux tables en se basant sur une clé commune.

### 5.1 `INNER JOIN` (Intersection)
Ne renvoie que les lignes ayant une correspondance dans les deux tables.

```sql
SELECT users.full_name, orders.id AS order_id, orders.total
FROM users
INNER JOIN orders ON orders.user_id = users.id;
```
*Si un utilisateur n'a jamais commandé, il n'apparaît pas dans le résultat.*

### 5.2 `LEFT JOIN` (Tout à gauche + correspondances à droite)
Renvoie **tous** les utilisateurs, qu'ils aient passé commande ou non (les champs de commande seront `NULL` pour les utilisateurs inactifs).

```sql
SELECT users.full_name, orders.id AS order_id, orders.total
FROM users
LEFT JOIN orders ON orders.user_id = users.id;
```

```
Tableau récapitulatif des jointures :
- INNER JOIN : Uniquement ce qui match des deux côtés.
- LEFT JOIN  : Tout le tableau de gauche + matches du tableau de droite.
- RIGHT JOIN : Tout le tableau de droite + matches du tableau de gauche.
- FULL JOIN  : L'ensemble complet des deux tables, avec des NULL là où ça ne match pas.
```

---

## 6. Calculs et Agrégations (`GROUP BY`, `HAVING`)

Les fonctions d'agrégation permettent de calculer des statistiques sur un groupe de lignes :
- `COUNT(*)` : Compter les lignes.
- `SUM(colonne)` : Faire la somme.
- `AVG(colonne)` : Calculer la moyenne.
- `MIN(colonne)` / `MAX(colonne)` : Trouver les extrêmes.

### Exemple : Total dépensé par chaque client

```sql
SELECT users.full_name, COUNT(orders.id) AS nb_orders, SUM(orders.total) AS total_depense
FROM users
INNER JOIN orders ON orders.user_id = users.id
GROUP BY users.id, users.full_name
HAVING SUM(orders.total) > 50.00;
```

> **Différence clé :**
> - `WHERE` filtre les lignes **avant** le regroupement.
> - `HAVING` filtre les résultats agrégés **après** le calcul du `GROUP BY`.

---

## 7. Performance : L'Essentiel sur les Index (`INDEX`)

Par défaut, quand vous cherchez un utilisateur avec `WHERE email = 'alice@example.com'`, le SGBDR doit scanner chaque ligne de la table du début à la fin (**Full Table Scan**). Sur une table de 2 millions de lignes, cela peut prendre plusieurs secondes.

Un **index** fonctionne comme l'index à la fin d'un livre :

```sql
CREATE INDEX idx_users_email ON users(email);
```

- **Avantage :** Recherche quasi instantanée ($O(\log N)$ via un arbre B-Tree).
- **Contrepartie :** Ralentit légèrement les opérations d'écriture (`INSERT`, `UPDATE`, `DELETE`) car l'index doit être recalculé à chaque modification, et consomme de l'espace disque.

---

## 8. Ordre d'Exécution Logique d'une Requête SQL

Pour éviter les erreurs courantes, gardez en tête que le moteur SQL n'exécute pas la requête dans l'ordre où vous l'écrivez :

1. `FROM` & `JOIN` (Quelles tables charger et croiser ?)
2. `WHERE` (Quelles lignes conserver ?)
3. `GROUP BY` (Comment regrouper les lignes ?)
4. `HAVING` (Quels groupes agrégés filtrer ?)
5. `SELECT` (Quelles colonnes ou calculs renvoyer ?)
6. `DISTINCT` (Éliminer les doublons)
7. `ORDER BY` (Comment trier le résultat final ?)
8. `LIMIT` / `OFFSET` (Combien de lignes renvoyer ?)