# Principes de Conception Logicielle (Niveau Novice)

En programmation, écrire du code qui « fonctionne » n'est que la première étape. L'enjeu majeur est d'écrire du code que vous ou vos collègues pourrez relire, corriger et faire évoluer dans six mois sans risquer de casser l'ensemble de l'application.

Les principes de conception logicielle sont des **guides de bon sens** éprouvés par des décennies de pratique pour garder un code propre, lisible et maintenable.

## 1. La Trinité Essentielle : KISS, YAGNI et DRY

Avant de s'attaquer à des règles d'architecture complexes, tout développeur doit intégrer trois réflexes du quotidien.

### 1.1 KISS : *Keep It Simple, Stupid* (Fais simple)

* **Idée clé :** La solution la plus simple est presque toujours la meilleure.
* **Le piège :** Utiliser des abstractions compliquées, des algorithmes alambiqués ou des design patterns disproportionnés pour un besoin élémentaire.
* **Exemple :**
  ```typescript
  // ❌ Trop compliqué (sur-ingénierie)
  function isEven(n: number): boolean {
    return Math.floor(n / 2) * 2 === n;
  }

  // ✅ Simple et lisible (KISS)
  function isEven(n: number): boolean {
    return n % 2 === 0;
  }
  ```

### 1.2 YAGNI : *You Aren't Gonna Need It* (Tu n'en auras pas besoin)

* **Idée clé :** N'implémentez **jamais** une fonctionnalité tant qu'elle n'est pas strictement nécessaire aujourd'hui.
* **Le piège :** Anticiper des besoins hypothétiques du futur (« Et si un jour le client veut payer en cryptomonnaies ? »).
* **Conséquence :** On passe du temps à coder, tester et maintenir du code inutile qui ne sera peut-être jamais utilisé.
* **Règle d'or :** Codez pour le présent, concevez pour faciliter l'évolution future.

### 1.3 DRY : *Don't Repeat Yourself* (Ne te répète pas)

* **Idée clé :** Chaque morceau de connaissance ou de logique métier doit avoir une représentation unique et non ambiguë dans votre système.
* **Le piège :** Copier-coller le même calcul de remise ou la même validation d'email à 5 endroits différents du projet. Si la règle change, vous risquez d'en oublier un.
* **Exemple :**
  ```typescript
  // ❌ Répétitif
  const totalClientA = priceA * 1.20;
  const totalClientB = priceB * 1.20;

  // ✅ Centralisé (DRY)
  const VAT_RATE = 0.20;
  function calculateTotalWithVAT(price: number): number {
    return price * (1 + VAT_RATE);
  }
  ```
> **Nuance importante pour débutant :** « Il vaut mieux une duplication légère qu'une mauvaise abstraction. » Ne forcez pas la factorisation si deux lignes de code se ressemblent visuellement mais représentent des concepts métier différents.

---

## 2. Démystifier les Principes SOLID

Popularisés par Robert C. Martin (« Uncle Bob »), les principes **SOLID** regroupent 5 règles d'or de conception orientée objet :

| Lettre | Nom du principe | En résumé pour débutant |
| :--- | :--- | :--- |
| **S** | **Single Responsibility Principle** | Une seule responsabilité par classe/fonction. |
| **O** | **Open/Closed Principle** | Ouvert à l'extension, fermé à la modification. |
| **L** | **Liskov Substitution Principle** | Une sous-classe doit pouvoir remplacer sa classe parente sans bug. |
| **I** | **Interface Segregation Principle** | Mieux vaut plusieurs petites interfaces qu'une grosse interface "fourre-tout". |
| **D** | **Dependency Inversion Principle** | Dépendre des abstractions, pas des détails concrets. |

---

### S — Responsabilité Unique (*Single Responsibility Principle - SRP*)

> *« Une classe ou un module ne doit avoir qu'une seule et unique raison de changer. »*

Si une classe gère à la fois les calculs métier, l'affichage à l'écran et la sauvegarde en base de données, elle fait trop de choses.

```typescript
// ❌ Violation : La classe gère la facture ET son impression papier
class Invoice {
  calculateTotal() { /* calcul */ }
  printPdf() { /* mise en page PDF */ }
}

// ✅ Respect du SRP : Deux responsabilités = Deux classes distinctes
class Invoice {
  calculateTotal() { /* calcul */ }
}

class InvoicePrinter {
  printPdf(invoice: Invoice) { /* mise en page PDF */ }
}
```

---

### O — Ouvert / Fermé (*Open/Closed Principle - OCP*)

> *« Les entités logicielles doivent être ouvertes à l'extension, mais fermées à la modification. »*

Vous devez pouvoir ajouter un nouveau comportement **sans modifier le code source existant** (qui fonctionne déjà et a été testé).

```typescript
// ❌ Violation : À chaque nouvel animal, on doit modifier le switch interne
class AnimalSound {
  makeSound(animalType: string) {
    if (animalType === 'dog') return 'Ouaf';
    if (animalType === 'cat') return 'Miaou';
    // Si on ajoute 'duck', on doit ré-ouvrir et éditer ce fichier !
  }
}

// ✅ Respect de l'OCP : On crée une interface et on étend avec de nouvelles classes
interface Animal {
  speak(): string;
}

class Dog implements Animal {
  speak() { return 'Ouaf'; }
}

class Duck implements Animal { // Ajout sans toucher aux autres classes !
  speak() { return 'Coin'; }
}
```

---

### L — Substitution de Liskov (*Liskov Substitution Principle - LSP*)

> *« Si $S$ est un sous-type de $T$, alors les objets de type $T$ doivent pouvoir être remplacés par des objets de type $S$ sans altérer le programme. »*

En clair : l'héritage ne doit pas casser les promesses de la classe parente.

```typescript
// ❌ Violation classique : Le pingouin hérite d'Oiseau mais ne sait pas voler
class Bird {
  fly() { console.log("Je vole !"); }
}

class Penguin extends Bird {
  fly() {
    throw new Error("Je ne peux pas voler !"); // 💥 Casse le contrat parent !
  }
}

// ✅ Solution : Réorganiser la hiérarchie
class Bird {}
class FlyingBird extends Bird {
  fly() { console.log("Je vole !"); }
}
class Penguin extends Bird {
  swim() { console.log("Je nage !"); }
}
```

---

### I — Ségrégation des Interfaces (*Interface Segregation Principle - ISP*)

> *« Aucun client ne devrait être forcé de dépendre de méthodes qu'il n'utilise pas. »*

Ne créez pas d'interfaces gigantesques qui forcent une classe à implémenter des fonctions vides avec des erreurs.

```typescript
// ❌ Interface obèse
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

// Un robot ne mange pas et ne dort pas...
class Robot implements Worker {
  work() { /* travaille */ }
  eat() { throw new Error("Ne mange pas"); } // Inutile
  sleep() { throw new Error("Ne dort pas"); } // Inutile
}

// ✅ Interfaces ciblées et modulaires
interface Workable {
  work(): void;
}
interface Feedable {
  eat(): void;
}

class Robot implements Workable {
  work() { /* travaille */ }
}
```

---

### D — Inversion des Dépendances (*Dependency Inversion Principle - DIP*)

> *« Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau. Les deux doivent dépendre d'abstractions. »*

Un module principal (haut niveau) ne doit pas être directement soudé à un outil technique précis (base de données MySQL, envoi d'emails via SendGrid). On passe par une **interface** intermédiaire.

```typescript
// ❌ Mauvais : La classe OrderService est soudée directement à MySQL
class MySQLDatabase {
  save(order: any) { /* écriture SQL directe */ }
}

class OrderService {
  private db = new MySQLDatabase(); // Couplage fort !
  createOrder(order: any) {
    this.db.save(order);
  }
}

// ✅ Respect du DIP : On dépend d'une interface abstraite
interface Database {
  save(data: any): void;
}

class OrderService {
  // L'implémentation concrète est injectée depuis l'extérieur
  constructor(private db: Database) {}

  createOrder(order: any) {
    this.db.save(order);
  }
}
```

---

## 3. Synthèse Pratique pour le Développeur Débutant

1. **KISS :** Écris le code le plus limpide possible. Si ça a l'air trop malin, c'est probablement trop complexe.
2. **YAGNI :** Ne résous pas les problèmes que tu n'as pas encore.
3. **DRY :** Ne duplique pas la logique métier, mais n'aie pas peur de deux blocs de code similaires s'ils ont des finalités différentes.
4. **SOLID :**
   * **S** : Fais une seule chose, mais fais-la bien.
   * **O** : Ajoute du nouveau code plutôt que de modifier le code existant qui marche.
   * **L** : Ne promets pas ce que tes enfants ne peuvent pas tenir.
   * **I** : Découpe tes contrats en petits morceaux digestes.
   * **D** : Fais confiance à des contrats (interfaces), pas à des outils figés.