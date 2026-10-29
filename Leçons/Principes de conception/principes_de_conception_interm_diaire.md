# Principes de Conception Logicielle — Niveau Intermédiaire

Après avoir compris les acronymes de base (KISS, YAGNI, DRY, SOLID), l'étape intermédiaire consiste à évaluer les **compromis architecturaux** (*trade-offs*). Concevoir un logiciel robuste, c'est avant tout savoir arbitrer entre flexibilité et simplicité, et comprendre comment les objets ou modules interagissent entre eux.

---

## 1. Métriques Fondamentales : Couplage vs Cohésion

Toute décision d'architecture logicielle vise un équilibre : **une forte cohésion** et **un faible couplage** (*High Cohesion, Low Coupling*).

```
Faible Cohésion / Fort Couplage (À ÉVITER) :
┌──────────────┐         dépendances croisées        ┌──────────────┐
│ Module A     │◄───────────────────────────────────►│ Module B     │
│ (mélange DB, │                                     │ (mélange UI, │
│  UI, calcul) │◄───────────────────────────────────►│  auth, logs) │
└──────────────┘                                     └──────────────┘

Forte Cohésion / Faible Couplage (CIBLE) :
┌──────────────┐                 contrat             ┌──────────────┐
│ Module Auth  │                 (interface)         │ Module Users │
│ (100% dédié  ├────────────────────────────────────►│ (100% dédié  │
│  à la sécu)  │                                     │  au profil)  │
└──────────────┘                                     └──────────────┘
```

### 1.1 La Cohésion (*Cohesion*)
La cohésion mesure à quel point les responsabilités au sein d'un même module ou d'une même classe sont **étroitement liées et concourent à un but unique**.
* **Faible cohésion :** Une classe "fourre-tout" (`AppManager`, `Utils`, `Helpers`) regroupant du hashage de mot de passe, du formatage de dates et des requêtes SQL.
* **Forte cohésion :** Un module `PasswordHasher` dont toutes les méthodes concourent exclusivement à la manipulation cryptographique de mots de passe.

### 1.2 Le Couplage (*Coupling*)
Le couplage mesure le **degré d'interdépendance** entre deux modules distincts.
* **Fort couplage :** Le module A lit directement les variables internes privées du module B, instancie directement ses classes concrètes et dépend de sa structure interne. Modifier B brise inévitablement A.
* **Faible couplage :** Le module A communique avec le module B uniquement via une interface stable, sans savoir comment B fonctionne en coulisses.

---

## 2. Favoriser la Composition sur l'Héritage (*Composition over Inheritance*)

L'héritage (`extends`) crée le couplage le plus rigide qui soit en orienté objet. Une sous-classe hérite de l'intégralité du comportement et des failles de sa parente (*Gorilla/Banana Problem* : « Vous vouliez juste une banane, et vous vous retrouvez avec un gorille qui tient la banane, plus toute la jungle autour »).

### 2.1 Le piège de l'héritage en cascade

```typescript
// ❌ Hiérarchie rigide et explosive
class Monster {
  attack() {}
}
class FlyingMonster extends Monster {
  fly() {}
}
class SwimmingMonster extends Monster {
  swim() {}
}
// Comment créer un monstre qui vole ET qui nage sans duplication ?
// L'héritage multiple n'est souvent pas permis et mène au "Diamond Problem".
```

### 2.2 La solution par composition

On assemble de petits comportements interchangeables au lieu de figer une taxonomie :

```typescript
// ✅ Comportements autonomes
interface AttackStrategy {
  attack(): void;
}
interface MovementStrategy {
  move(): void;
}

class FireBreath implements AttackStrategy {
  attack() { console.log("Crache du feu !"); }
}
class FlyingMovement implements MovementStrategy {
  move() { console.log("Vole dans les airs"); }
}
class SwimmingMovement implements MovementStrategy {
  move() { console.log("Nage sous l'eau"); }
}

// La classe Monster assemble des comportements dynamiques
class Monster {
  constructor(
    private attacker: AttackStrategy,
    private mover: MovementStrategy
  ) {}

  performAttack() { this.attacker.attack(); }
  performMove() { this.mover.move(); }
}

// Composition à la carte :
const dragon = new Monster(new FireBreath(), new FlyingMovement());
const leviathan = new Monster(new FireBreath(), new SwimmingMovement());
```

---

## 3. La Loi de Déméter (*Law of Demeter - LoD*)

Aussi appelée **principe de moindre connaissance**, cette loi stipule qu'une unité logicielle ne doit parler qu'à ses « amis proches » et ignorer la structure interne des objets distants.

> **Règle pratique :** Ne pas enchaîner les appels de navigation du type `a.getB().getC().getD().doSomething()`.

### Exemple de violation (Train Wreck)

```typescript
// ❌ Violation de la Loi de Déméter (accident ferroviaire)
// La fonction de commande connaît la structure intime du client,
// de son portefeuille, et de la carte bancaire.
function processPayment(customer: Customer, amount: number) {
  customer.getWallet().getCreditCard().charge(amount);
}
```

Si l'organisation interne du portefeuille change (ex: passage à Apple Pay ou compte bancaire direct), cette fonction casse.

```typescript
// ✅ Respect de la Loi de Déméter
// On délègue la responsabilité au premier intermédiaire légitime
function processPayment(customer: Customer, amount: number) {
  customer.charge(amount);
}

// Le client gère lui-même son portefeuille en interne sans l'exposer
class Customer {
  private wallet: Wallet;

  charge(amount: number) {
    this.wallet.deduct(amount);
  }
}
```

---

## 4. Ordonner au lieu d'Interroger : *Tell, Don't Ask*

Ce principe rappelle qu'au lieu d'interroger un objet sur son état interne pour prendre une décision à sa place, il faut lui **donner directement un ordre**.

### 4.1 Mauvais pattern : Interroger puis manipuler (*Anemic / Ask*)

```typescript
// ❌ "Ask" : Le code externe inspecte le compte et calcule le solde lui-même
class BankAccount {
  public balance: number = 0;
}

function withdraw(account: BankAccount, amount: number) {
  if (account.balance >= amount) {
    account.balance -= amount; // Mutation externe
  } else {
    throw new Error("Solde insuffisant");
  }
}
```

### 4.2 Bon pattern : Encapsuler et ordonner (*Tell*)

```typescript
// ✅ "Tell" : L'objet protège ses invariants métier
class BankAccount {
  private balance: number = 0;

  withdraw(amount: number): void {
    if (this.balance < amount) {
      throw new Error("Solde insuffisant");
    }
    this.balance -= amount;
  }

  getBalance(): number {
    return this.balance;
  }
}

// Le code client ordonne simplement :
account.withdraw(50);
```

---

## 5. CQS : *Command-Query Separation* (Séparation Commande-Requête)

Formulé par Bertrand Meyer, ce principe dicte qu'une méthode doit appartenir à l'une de ces deux catégories, mais jamais aux deux en même temps :

1. **Une Requête (*Query*) :** Renvoie une information / un calcul, mais **ne modifie aucun état** (effet de bord nul, idempotente).
2. **Une Commande (*Command*) :** Modifie l'état du système, mais **ne renvoie aucune valeur métier** (renvoie généralement `void`).

```typescript
// ❌ Violation CQS : Modifie l'état ET renvoie une valeur
class UserSession {
  private accessCount = 0;

  // Cette méthode lit une valeur, mais incrémente discrètement un compteur !
  getUser() {
    this.accessCount++; // Effet de bord masqué
    return this.currentUser;
  }
}

// ✅ Respect CQS : Deux méthodes distinctes
class UserSession {
  private accessCount = 0;

  // Query pure (lecture sans mutation)
  getUser() {
    return this.currentUser;
  }

  // Command (mutation explicite)
  recordAccess() {
    this.accessCount++;
  }
}
```

---

## 6. Synthèse des Réflexes Intermédiaires

| Principe | Question à se poser en revue de code | Bénéfice direct |
| :--- | :--- | :--- |
| **High Cohesion** | Cette classe fait-elle uniquement des tâches qui vont naturellement ensemble ? | Maintenance facilitée, responsabilités claires. |
| **Low Coupling** | Puis-je remplacer ce module sans réécrire la moitié du projet ? | Modules interchangeables, tests unitaires simples. |
| **Composition** | Est-ce que cet héritage `extends` ne me force pas à hériter de méthodes inutiles ? | Évite les hiérarchies rigides et fragiles. |
| **Law of Demeter** | Suis-je en train de chaîner plusieurs accesseurs (`a.b.c.d()`) ? | Protège contre les changements structurels profonds. |
| **Tell, Don't Ask** | Est-ce que j'extrais des données d'un objet pour faire un calcul qu'il pourrait faire lui-même ? | Encapsulation forte, logique métier centralisée. |
| **CQS** | Ma fonction de lecture produit-elle des effets de bord cachés ? | Prédictibilité, élimination des bugs sournois. |