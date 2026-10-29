# Initiation à MongoDB & aux Bases Orientées Documents (Niveau Novice)

**MongoDB** est le moteur de base de données **NoSQL** le plus populaire. Contrairement aux systèmes relationnels (SQL) qui découpent l'information en tables rigides reliées par des clés étrangères, MongoDB adopte une approche **orientée document** : il stocke les données sous une forme très proche des objets JSON manipuler quotidiennement en programmation.

---

## 1. Le Modèle Mental : Bases, Collections et Documents

Pour transposer vos repères depuis le monde SQL :

```
SQL                          MongoDB
┌──────────────────┐         ┌──────────────────┐
│ Base de données  │   ──►   │ Base de données  │
│ Table            │   ──►   │ Collection       │
│ Ligne (Row)      │   ──►   │ Document (BSON)  │
│ Colonne (Column) │   ──►   │ Champ (Field)    │
└──────────────────┘         └──────────────────┘
```

### Le Document BSON

MongoDB enregistre les données au format **BSON** (*Binary JSON*). C'est une extension binaire du format JSON qui ajoute des types de données non gérés nativement par JSON :
- Les dates réelles (`ISODate`)
- Les entiers stricts (32 bits, 64 bits) et décimaux
- L'identifiant natif unique : **`ObjectId`**

Exemple d'un document complet :

```json
{
  "_id": ObjectId("65f3a1b2c3d4e5f6a7b8c9d0"),
  "username": "sarah_c",
  "email": "sarah@example.com",
  "age": 27,
  "isActive": true,
  "roles": ["editor", "member"],
  "address": {
    "city": "Lyon",
    "zipCode": "69002"
  },
  "createdAt": ISODate("2026-03-15T10:30:00Z")
}
```

> **L'identifiant `_id` :** Chaque document possède obligatoirement un champ `_id` unique qui sert de clé primaire. Si vous ne le fournissez pas lors de la création, MongoDB le génère automatiquement avec un `ObjectId` (contenant un timestamp, l'identifiant machine/processus et un compteur).

---

## 2. Flexibilité du Schéma (*Schemaless*)

Dans une collection MongoDB, **les documents ne sont pas contraints d'avoir tous la même forme**.

```
Collection: products

Document 1 (Livre) :
{
  "_id": ObjectId("..."),
  "title": "Clean Code",
  "pages": 464,
  "isbn": "978-0132350884"
}

Document 2 (T-Shirt) :
{
  "_id": ObjectId("..."),
  "title": "T-Shirt Développeur",
  "sizes": ["S", "M", "L", "XL"],
  "colors": ["Noir", "Bleu"]
}
```

> **Bonne pratique pour débutant :** Flexibilité ne signifie pas anarchie. Dans une vraie application, on maintient une structure cohérente à l'aide de validateurs de schéma côté base ou via un ODM (*Object Document Mapper*) comme **Mongoose** en Node.js.

---

## 3. Les Opérations CRUD Fondamentales

On manipule les données via l'objet `db.<nomDeLaCollection>`.

### 3.1 Création (Create)

- Insérer un document unique :
```javascript
db.users.insertOne({
  name: "Thomas Martin",
  email: "thomas@test.com",
  age: 31,
  tags: ["javascript", "mongodb"]
});
```

- Insérer plusieurs documents d'un coup :
```javascript
db.users.insertMany([
  { name: "Léa Roux", email: "lea@test.com", age: 24 },
  { name: "Maxime Morel", email: "maxime@test.com", age: 40 }
]);
```

---

### 3.2 Lecture et Filtres (Read)

La méthode centrale est `find(filtre, projection)` :

```javascript
// Récupérer tous les documents
db.users.find();

// Filtrer avec une condition exacte
db.users.find({ email: "thomas@test.com" });

// Récupérer uniquement le premier document correspondant
db.users.findOne({ email: "thomas@test.com" });
```

#### Les opérateurs de comparaison courants :
Tous les opérateurs MongoDB commencent par le symbole `$`.

| Opérateur | Signification | Exemple |
|---|---|---|
| `$eq` | Égal à (implicite sans opérateur) | `{ age: { $eq: 25 } }` ou `{ age: 25 }` |
| `$ne` | Différent de | `{ age: { $ne: 25 } }` |
| `$gt` / `$gte` | Plus grand que / Supérieur ou égal | `{ age: { $gte: 18 } }` |
| `$lt` / `$lte` | Plus petit que / Inférieur ou égal | `{ age: { $lt: 30 } }` |
| `$in` | Présent dans une liste de valeurs | `{ city: { $in: ["Paris", "Lyon"] } }` |
| `$nin` | Absent d'une liste de valeurs | `{ status: { $nin: ["banned", "deleted"] } }` |

#### Les opérateurs logiques :
```javascript
// AND implicite : les utilisateurs majeurs habitant à Paris
db.users.find({ age: { $gte: 18 }, city: "Paris" });

// OR explicite : les utilisateurs mineurs OU inactifs
db.users.find({
  $or: [
    { age: { $lt: 18 } },
    { isActive: false }
  ]
});
```

#### Projection, Tri et Pagination :
```javascript
db.users.find(
  { age: { $gte: 18 } },
  { name: 1, email: 1, _id: 0 } // Projection : 1 = inclure, 0 = masquer
)
.sort({ age: -1 })              // Tri : 1 = croissant, -1 = décroissant
.skip(10)                       // Ignore les 10 premiers (pagination)
.limit(5);                      // Récupère au maximum 5 documents
```

---

### 3.3 Mise à Jour (Update)

> **Règle absolue :** Il faut toujours utiliser des opérateurs de mise à jour (comme `$set`). Si vous passez un objet brut sans opérateur, MongoDB écrasera l'intégralité du document par ce nouvel objet !

```javascript
// Mettre à jour le premier document qui match
db.users.updateOne(
  { email: "thomas@test.com" }, // 1. Filtre
  { 
    $set: { age: 32 },          // 2. Modifie ou ajoute ce champ
    $currentDate: { updatedAt: true } // Met à jour un timestamp
  }
);
```

#### Opérateurs de mise à jour fréquents :
- **`$set`** : Assigne une nouvelle valeur à un champ.
- **`$unset`** : Supprime un champ existant du document (`{ $unset: { temporaryToken: "" } }`).
- **`$inc`** : Incrémente ou décrémente une valeur numérique (`{ $inc: { loginCount: 1, score: -5 } }`).
- **`$push`** : Ajoute un élément à la fin d'un tableau (`{ $push: { tags: "nodejs" } }`).
- **`$pull`** : Supprime un élément spécifique d'un tableau (`{ $pull: { tags: "spam" } }`).

Pour appliquer la modification à **tous** les documents correspondants au filtre :
```javascript
db.users.updateMany(
  { isActive: false },
  { $set: { status: "archived" } }
);
```

---

### 3.4 Suppression (Delete)

```javascript
// Supprimer un seul document
db.users.deleteOne({ email: "thomas@test.com" });

// Supprimer tous les documents matchant le critère
db.users.deleteMany({ isActive: false });

// Vider l'intégralité de la collection (attention !)
db.users.deleteMany({});
```

---

## 4. Modélisation : Imbrication vs Référencement

Contrairement au SQL qui normalise presque systématiquement les données dans des tables séparées, MongoDB propose deux stratégies de conception :

```
Approche 1 : Imbrication (Embedded)     Approche 2 : Référencement (References)
┌─────────────────────────────────┐     ┌────────────────┐    ┌────────────────┐
│ User                            │     │ User           │    │ Order          │
│  - _id: 1                       │     │  - _id: 1      │    │  - _id: 101    │
│  - name: Sarah                  │     │  - name: Sarah │    │  - userId: 1   │
│  - addresses: [                 │     └────────────────┘    │  - total: 45   │
│      { street: "...", zip: "" } │                           └────────────────┘
│    ]                            │
└─────────────────────────────────┘
```

### Quand imbriquer (*Embedding*) ?
- Relations de type **1 à 1** (ex. profil et réglages de compte).
- Relations de type **1 à N fini et modeste** (ex. un article de blog et ses 3 tags, un utilisateur et ses 2 adresses).
- Données toujours consultées ensemble en une seule lecture.

### Quand référencer (*Referencing*) ?
- Relations de type **1 à N infini** (ex. un capteur météo et ses millions de relevés de température).
- Données fréquemment modifiées isolément par d'autres processus.
- Pour éviter la duplication massive de données qui changent souvent.

---

## 5. Introduction à l'Agrégation (*Aggregation Pipeline*)

Quand `find()` ne suffit plus pour calculer des statistiques ou transformer la donnée, MongoDB utilise le **pipeline d'agrégation**.

Le document traverse une série d'étapes successives (*stages*) ordonnées :

```
[ Collection ] ──► $match (Filtre) ──► $group (Agrège) ──► $sort (Trie) ──► [ Résultat ]
```

### Exemple : Calculer le panier moyen par pays

```javascript
db.orders.aggregate([
  // Étape 1 : Ne garder que les commandes payées
  { $match: { status: "PAID" } },

  // Étape 2 : Regrouper par pays et calculer la moyenne
  {
    $group: {
      _id: "$shippingAddress.country",
      totalCommandes: { $sum: 1 },
      panierMoyen: { $avg: "$totalAmount" }
    }
  },

  // Étape 3 : Trier du panier le plus élevé au plus bas
  { $sort: { panierMoyen: -1 } }
]);
```

---

## 6. L'Importance Cruciale des Index

Sans index, MongoDB doit inspecter chaque document de la collection pour répondre à une requête : c'est un **Collection Scan** (`COLLSCAN`).

Pour accélérer les recherches sur un champ fréquemment interrogé :

```javascript
// Créer un index ascendant (1) sur le champ email
db.users.createIndex({ email: 1 }, { unique: true });

// Créer un index composé sur le nom et l'âge
db.users.createIndex({ name: 1, age: -1 });
```

Pour analyser les performances d'une requête et vérifier si elle utilise bien un index :
```javascript
db.users.find({ email: "sarah@example.com" }).explain("executionStats");
```
*(On cherche à voir `stage: "IXSCAN"` plutôt que `COLLSCAN`).*

---

## Synthèse : Réflexes du Développeur MongoDB Débutant

1. **Pensez d'abord aux requêtes de lecture de votre application :** Modélisez vos documents en fonction de la manière dont votre interface aura besoin d'afficher l'information.
2. **N'oubliez jamais `$set` dans vos `updateOne()` :** Cela évite d'écraser par accident les champs non mentionnés.
3. **Indexez vos clés de recherche :** Tout champ présent régulièrement dans vos clauses `{ ... }` ou dans vos `.sort()` doit être indexé en production.
4. **Méfiez-vous de la taille maximale d'un document :** Un document BSON individuel est limité à **16 Mo** maximum.