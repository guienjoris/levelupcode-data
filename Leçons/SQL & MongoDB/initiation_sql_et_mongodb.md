# Bien Débuter : SQL vs MongoDB (Niveau Novice)

Dans toute application, la base de données est le lieu où les informations sont conservées de façon permanente. Deux grandes philosophies dominent le marché :

1. **Les bases relationnelles (SQL)** : Rigoureuses, organisées en tableaux stricts avec des relations formelles (ex: PostgreSQL, MySQL, SQLite).
2. **Les bases orientées documents (NoSQL / MongoDB)** : Flexibles, stockant des enregistrements sous forme de documents proches du format JSON.

---

## 1. Modèle Mental : Tableaux vs Documents

### L'approche Relationnelle (SQL)
Les données sont découpées en **tables** composées de colonnes fixes et de lignes. Si une information concerne un autre domaine (ex. les commandes d'un utilisateur), on crée une table séparée liée par un identifiant (*clé étrangère*).

```
Table: users
┌────┬──────────────┬───────────────────┐
│ id │ name         │ email             │
├────┼──────────────┼───────────────────┤
│ 1  │ Alice Dupont │ alice@example.com │
└────┴──────────────┴───────────────────┘

Table: orders
┌────┬─────────┬────────┬───────────────┐
│ id │ user_id │ total  │ created_at    │
├────┼─────────┼────────┼───────────────┤
│ 10 │ 1       │ 49.99  │ 2026-03-15    │
└────┴─────────┴────────┴───────────────┘
```

### L'approche Document (MongoDB / NoSQL)
Les données sont stockées sous forme de **documents BSON** (binaire proche du JSON). On peut imbriquer des listes ou des sous-objets directement à l'intérieur d'un document sans créer de table supplémentaire.

```json
{
  "_id": "65f3a1b2c3d4e5f6a7b8c9d0",
  "name": "Alice Dupont",
  "email": "alice@example.com",
  "orders": [
    {
      "id": 10,
      "total": 49.99,
      "createdAt": "2026-03-15"
    }
  ]
}
```

---

## 2. Le Lexique Comparatif

Pour s'y retrouver, voici les équivalences directes de vocabulaire :

| Concept | Monde SQL (PostgreSQL, MySQL) | Monde Document (MongoDB) |
| :--- | :--- | :--- |
| **Conteneur global** | Base de données (*Database*) | Base de données (*Database*) |
| **Ensemble d'enregistrements** | Table | Collection |
| **Un enregistrement unique** | Ligne (*Row* ou *Tuple*) | Document (BSON) |
| **Une propriété / valeur** | Colonne (*Column*) | Champ (*Field*) |
| **Identifiant unique** | Clé primaire (*Primary Key*, ex: `id`) | Champ `_id` (ObjectId automatique) |
| **Liaison entre deux entités** | Clé étrangère (*Foreign Key*) & `JOIN` | Référence d'ID ou document imbriqué |
| **Schéma de structure** | Strict et prédéfini à l'avance | Flexible et dynamique (*Schemaless*) |

---

## 3. Le CRUD Comparé (Créer, Lire, Mettre à jour, Supprimer)

### 3.1 Création d'une structure et insertion (Create)

#### En SQL
Il faut d'abord définir la structure de la table avant de pouvoir insérer des données :
```sql
-- 1. Définition de la table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(150) UNIQUE NOT NULL,
  age INT
);

-- 2. Insertion
INSERT INTO users (name, email, age)
VALUES ('Alice Dupont', 'alice@example.com', 28);
```

#### En MongoDB
Aucune table à créer au préalable ; la collection naît dès la première insertion :
```javascript
// La structure est libre pour chaque document
db.users.insertOne({
  name: "Alice Dupont",
  email: "alice@example.com",
  age: 28,
  hobbies: ["Lecture", "Randonnée"] // Tableau imbriqué direct
});
```

---

### 3.2 Lecture et filtrage (Read)

#### En SQL
On utilise le mot-clé `SELECT` et la clause `WHERE` :
```sql
-- Tous les utilisateurs majeurs, triés par nom
SELECT name, email 
FROM users 
WHERE age >= 18 
ORDER BY name ASC;
```

#### En MongoDB
On utilise la méthode `find()` avec un objet de filtre :
```javascript
// $gte signifie "Greater Than or Equal" (>=)
db.users.find(
  { age: { $gte: 18 } },
  { name: 1, email: 1, _id: 0 } // Projection : 1 = inclure, 0 = exclure
).sort({ name: 1 });
```

---

### 3.3 Mise à jour (Update)

#### En SQL
```sql
UPDATE users 
SET age = 29 
WHERE email = 'alice@example.com';
```
> **Attention :** Sans clause `WHERE`, toute la table est modifiée !

#### En MongoDB
```javascript
db.users.updateOne(
  { email: "alice@example.com" },
  { $set: { age: 29 } }
);
```
> L'opérateur `$set` modifie uniquement le champ spécifié sans écraser le reste du document.

---

### 3.4 Suppression (Delete)

#### En SQL
```sql
DELETE FROM users 
WHERE email = 'alice@example.com';
```

#### En MongoDB
```javascript
db.users.deleteOne({ email: "alice@example.com" });
```

---

## 4. Relations : `JOIN` SQL vs Imbrication MongoDB

### En SQL : Normalisation et Jointures
On évite la redondance en séparant les tables et en les reliant au moment de la lecture :
```sql
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON orders.user_id = users.id
WHERE users.id = 1;
```
* **Avantage :** Pas de duplication. Si l'utilisateur change de nom, on ne modifie qu'une seule ligne dans `users`.

### En MongoDB : Imbrication ou Référencement
Deux stratégies s'offrent à vous :
1. **L'imbrication (*Embedding*) :** Placer les données directement dans le document parent si elles n'ont pas de sens toutes seules (ex. les lignes d'articles d'un panier).
2. **Le référencement :** Stocker l'identifiant `_id` d'un document dans un autre (similaire à une clé étrangère, qu'on peut relier avec l'opérateur `$lookup`).

---

## 5. Comment choisir entre SQL et MongoDB ?

```
                            [ Quel type de données ? ]
                                         │
                 ┌───────────────────────┴───────────────────────┐
                 ▼                                               ▼
     Données très structurées                        Données évolutives, polymorphes
   Relations fortes, intégrité stricte             Catalogues variés, logs, temps réel
                 │                                               │
                 ▼                                               ▼
        [ Choisir SQL ]                                  [ Choisir MongoDB ]
    (PostgreSQL, MySQL, SQLite)                        (MongoDB, DynamoDB)
```

### Privilégiez le SQL si :
- Votre application gère des flux financiers, de la facturation ou de la comptabilité (besoin d'intégrité stricte et de transactions ACID infaillibles).
- Les relations entre vos entités sont nombreuses et complexes (un utilisateur appartient à des équipes, a des rôles, des permissions, des tickets).
- Le schéma de vos données est stable et bien identifié dès le départ.

### Privilégiez MongoDB si :
- Vos documents ont des structures variées (ex. un catalogue e-commerce où un vêtement a une `taille` et une `couleur`, alors qu'un ordinateur a une `RAM` et un `processeur`).
- Vous développez un prototype rapide dont les besoins en colonnes évoluent chaque semaine.
- Vos lectures principales portent sur un agrégat complet (lire un profil utilisateur avec ses réglages en une seule requête rapide sans jointures).

---

## 6. Synthèse des Bonnes Pratiques pour Débutant

1. **Ne pas confondre MongoDB et l'absence de règles :** Même si MongoDB n'impose pas de schéma au niveau du moteur, votre code (via des bibliothèques comme Mongoose ou Zod) doit valider les données.
2. **Attention aux requêtes destructrices en SQL :** Prenez l'habitude de tester vos filtres avec un `SELECT` avant d'exécuter un `UPDATE` ou un `DELETE`.
3. **Penser aux index :** Qu'il s'agisse de SQL ou de MongoDB, chercher une ligne ou un document sans index force la base à scanner l'intégralité du disque (*Full Table Scan* ou *Collection Scan*).