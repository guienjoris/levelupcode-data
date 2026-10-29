# Architecture Monolithique — Niveau Intermédiaire : Le Monolithe Modulaire

Dans l'imaginaire collectif du développement moderne, le terme « monolithe » est souvent assimilé à tort à un code désuet, désorganisé et impossible à maintenir. Cette confusion vient d'une méprise courante : confondre l'**unité de déploiement** (tout le code tourne dans un même processus) et la **structure logique du code** (comment les responsabilités sont organisées).

Un système regroupé dans une seule application peut être d'une clarté exemplaire s'il est conçu comme un **Monolithe Modulaire** (*Modular Monolith*). C'est souvent l'architecture qui offre le meilleur rapport rentabilité/complexité pour une majorité d'entreprises.

---

## 1. Monolithe Spaghetti vs Monolithe Modulaire

Le principal danger d'un monolithe n'est pas son déploiement unique, mais la dilution progressive de ses frontières internes :

```
Monolithe Spaghetti ("Big Ball of Mud") :
┌────────────────────────────────────────────────────────┐
│  Contrôleurs, Services et Modèles totalement entremêlés│
│                                                        │
│  [Paiement] ◄────── (SQL direct) ──────► [Catalogue]   │
│       ▲                                       ▲        │
│       │           (Appels circulaires)        │        │
│       ▼                                       ▼        │
│  [Utilisateurs] ◄──────────────────────► [Livraison]   │
└────────────────────────────────────────────────────────┘

Monolithe Modulaire :
┌────────────────────────────────────────────────────────┐
│               API GATEWAY / ROUTEUR HTTP               │
├─────────────────┬───────────────────┬──────────────────┤
│  MODULE USERS   │   MODULE ORDERS   │  MODULE BILLING  │
│ ┌─────────────┐ │  ┌─────────────┐  │ ┌─────────────┐  │
│ │Public API   │ │  │Public API   │  │ │Public API   │  │
│ ├─────────────┤ │  ├─────────────┤  │ ├─────────────┤  │
│ │Interne / DB │ │  │Interne / DB │  │ │Interne / DB │  │
│ └─────────────┘ │  └─────────────┘  │ └─────────────┘  │
└─────────────────┴───────────────────┴──────────────────┘
```

* **Le Monolithe Spaghetti (*Big Ball of Mud*) :** Les modèles s'importent mutuellement sans contrôle, une fonction de facturation fait une jointure SQL sur la table des paniers, et la modification d'une entité dans un coin de l'application déclenche des régressions inattendues à l'autre bout du système.
* **Le Monolithe Modulaire :** L'application est divisée en sous-domaines métier étanches (*modules* ou *packages*). Chaque module possède une interface publique explicite. Tout accès aux entrailles privées d'un module par un autre module est strictement interdit.

---

## 2. Principes d'Isolation d'un Module

Pour qu'un monolithe modulaire ne dégénère pas en spaghetti, chaque module doit respecter trois règles fondamentales :

### 2.1 Façade ou API Publique explicite

Un module ne doit exposer au reste du monolithe qu'un ensemble restreint de contrats (interfaces, DTOs, événements) :

```
src/modules/billing/
├── public/                     <-- Seul ce dossier peut être importé par les autres
│   ├── billing.module.ts
│   ├── billing.facade.ts       <-- Point d'entrée des méthodes autorisées
│   └── dtos/
│       └── invoice-summary.dto.ts
└── internal/                   <-- TOTALEMENT PRIVÉ au module billing
    ├── domain/                 <-- Entités, règles métier
    ├── infrastructure/         <-- Requêtes SQL, tables ORM
    └── services/
```

Si le module `Orders` a besoin de facturer, il injecte `BillingFacade` et manipule `InvoiceSummaryDto`. Il lui est interdit d'importer directement `internal/domain/Invoice.entity.ts`.

### 2.2 Forcer l'étanchéité avec l'outillage statique

La bonne volonté humaine ne suffit pas à préserver les frontières dans le temps. On utilise des analyseurs statiques ou des linters d'architecture :

* **TypeScript / ESLint :** Utilisation d'outils comme **Nx** (avec la règle `@nx/enforce-module-boundaries`) ou **Good-fences** pour interdire les imports vers les dossiers `internal/` d'autres modules.
* **Java / Kotlin :** Utilisation de **ArchUnit** ou des modules Java (`module-info.java`) pour valider les règles de dépendances lors des tests automatisés.
* **PHP :** Outils comme **deptrac** pour définir des couches et des modules étanches.

---

## 3. Stratégies de Données dans un Monolithe Modulaire

Le partage non maîtrisé de tables SQL est la première cause de mort d'un monolithe. Il existe trois niveaux d'étanchéité des données au sein d'une même base physique :

```
Niveau 1 : Tables préfixées         Niveau 2 : Schémas SQL distincts
┌───────────────────────────┐       ┌───────────────────────────┐
│ Database Unique           │       │ Database Unique           │
│  - users_accounts         │       │  ┌──────────────────────┐ │
│  - users_profiles         │       │  │ Schema 'users'       │ │
│  - orders_items           │       │  └──────────────────────┘ │
│  - billing_invoices       │       │  ┌──────────────────────┐ │
│                           │       │  │ Schema 'billing'     │ │
└───────────────────────────┘       │  └──────────────────────┘ │
                                    └───────────────────────────┘
```

1. **Niveau 1 : Préfixage strict des tables (`users_*`, `billing_*`).** Solution élémentaire, mais qui n'empêche pas techniquement un développeur d'écrire une jointure SQL `JOIN billing_invoices` dans le module `Users`.
2. **Niveau 2 : Ségrégation par schémas SQL (`users.accounts`, `billing.invoices`).** Permet de cloisonner les migrations et de restreindre les droits d'accès utilisateur au niveau du moteur SQL (PostgreSQL supporte cela nativement).
3. **La règle d'or des jointures :** **Aucune jointure SQL inter-modules.** Si le module `Orders` a besoin du nom d'un client, il ne fait pas de `JOIN users`. Il appelle la méthode `usersFacade.getSummary(userId)` ou stocke une copie dénormalisée des informations nécessaires au moment de l'achat.

---

## 4. Communication Inter-Modules : Appel Synchrone vs Événements en Mémoire

Dans un monolithe, les services s'exécutent dans le même processus. La communication n'a pas besoin de passer par un réseau HTTP ou un broker Kafka externe : tout se passe en mémoire vive, garantissant des performances maximales.

### 4.1 L'Appel Direct via Façade (Couplage fort temporel)

```typescript
// Au sein du module Orders
class OrderCheckoutService {
  constructor(private readonly billingFacade: BillingFacade) {}

  async processOrder(order: Order): Promise<void> {
    // Appel d'une méthode TypeScript en mémoire : 0ms de latence réseau
    await this.billingFacade.chargeCustomer({
      customerId: order.customerId,
      amount: order.totalAmount,
    });
  }
}
```

* **Avantage :** Simplicité absolue, sécurité du typage à la compilation, exécution instantanée.
* **Inconvénient :** Si la facturation échoue ou prend du temps, la commande est bloquée.

### 4.2 Le Bus d'Événements en Mémoire (Découplage total)

Pour les opérations non bloquantes (envois d'e-mails, calcul de statistiques, notifications), on utilise un bus d'événements interne (par exemple `EventEmitter` de Node.js, Spring Events, ou MediatR en .NET) :

```
[Module Orders] ──(Publie en mémoire)──► [In-Memory Event Bus]
                                                │
                     ┌──────────────────────────┴──────────────────────────┐
                     ▼                                                     ▼
        [Module Notifications]                                  [Module Statistiques]
     (Écoute : 'OrderPlacedEvent')                           (Écoute : 'OrderPlacedEvent')
```

```typescript
// Le module Orders émet un événement sans savoir qui l'écoute
eventBus.publish(new OrderPlacedEvent(order.id, order.customerId, order.totalAmount));
```

Le module `Notifications` réagit de manière asynchrone sans que le module `Orders` ne connaisse son existence.

---

## 5. Gestion des Transactions : L'Atout Maître du Monolithe

Le plus grand avantage du monolithe sur les microservices réside dans sa capacité à utiliser des **transactions ACID locales** :

* Dans un environnement microservices, valider un panier et décrémenter un stock requiert des sagas complexes et de la cohérence à terme (*eventual consistency*).
* Dans un monolithe, si une incohérence est intolérable sur une opération critique, il est possible d'envelopper deux actions de modules distincts dans la même transaction de base de données :

```typescript
await db.transaction(async (trx) => {
  await ordersRepository.create(orderData, trx);
  await inventoryRepository.reserveStock(items, trx);
  // Si le stock est insuffisant : ROLLBACK atomique immédiat garanti par le moteur SQL !
});
```

> **Attention au compromis :** Bien que ce soit un atout formidable, l'abus de transactions traversant plusieurs modules recrée un couplage fort. Utilisez cette capacité avec parcimonie pour les cas d'intégrité absolue.

---

## 6. Quand et Comment Découper un Monolithe ?

Un monolithe modulaire bien conçu possède une propriété unique : **il rend l'extraction vers un microservice triviale le jour où cela devient réellement nécessaire.**

```
Phase 1 : Monolithe Modulaire           Phase 2 : Extraction d'un module sous forte charge
┌───────────────────────────────────┐    ┌─────────────────────────────────┐   ┌──────────────┐
│ [Users]    [Orders]   [Billing]   │    │ [Users]    [Orders]             │   │ [Billing]    │
│    │          │           │       │    │    │          │                 │   │              │
│    └──────────┴───────────┘       │    │    └──────────┴──────► [HTTP] ──┼──►│  (Service    │
│         (En mémoire)              │    │                                 │   │   autonome)  │
└───────────────────────────────────┘    └─────────────────────────────────┘   └──────────────┘
```

### Critères légitimes pour extraire un module du monolithe :

1. **Besoin d'élasticité asymétrique :** Le module d'encodage vidéo nécessite 50 machines avec GPU, tandis que le reste du site tourne sur 2 petits serveurs.
2. **Cycle de déploiement divergent :** Une équipe dédiée a besoin de déployer le moteur de recommandation 15 fois par jour sans risquer d'interrompre l'application principale.
3. **Contraintes technologiques spécifiques :** Un module de Machine Learning doit être écrit en Python, alors que le monolithe est en TypeScript ou Java.

Si aucune de ces conditions n'est réunie, conserver le monolithe modulaire évite l'ensemble des frais de gestion et des risques d'un réseau distribué.

---

## 7. Synthèse : Forces et Faiblesses du Monolithe Modulaire

| Atouts majeurs | Défis à surveiller |
| --- | --- |
| **Simplicité opérationnelle :** Un seul pipeline CI/CD, un seul binaire, zéro configuration réseau inter-services. | **Discipline d'équipe obligatoire :** Nécessite des règles strictes et des linters pour empêcher les raccourcis de code. |
| **Performances extrêmes :** Communications en mémoire vive ($< 1$ ms) au lieu de trames HTTP/gRPC sur le réseau. | **Partage des ressources :** Une fuite mémoire dans un module instable peut faire chuter l'ensemble du processus. |
| **Débogage limpide :** Possibilité de poser des points d'arrêt et de suivre l'exécution d'un bout à l'autre dans son IDE. | **Temps de build :** Sur des projets titanesques, la compilation et la suite de tests globale peuvent devenir lentes. |