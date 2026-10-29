# Approfondissement du Pattern MVC — Niveau Intermédiaire

Dans sa version vulgarisée, le pattern **Modèle-Vue-Contrôleur (MVC)** semble limpide : la Vue affiche, le Contrôleur aiguille, le Modèle stocke. Pourtant, dans les applications du monde réel (qu'il s'agisse de Laravel, Ruby on Rails, Django, Spring MVC ou NestJS/Express), cette simplicité apparente conduit fréquemment à des bases de code intraitables : contrôleurs tentaculaires, logique métier éparpillée et modèles saturés.

Pour franchir le palier intermédiaire, il est indispensable de comprendre la dynamique des flux, d'éradiquer le modèle anémique, de maîtriser la séparation des responsabilités fines et de situer le MVC par rapport à ses évolutions (MVP, MVVM).

---

## 1. La Dualité Historique : MVC Originel vs MVC Web

Le pattern MVC a été inventé par **Trygve Reenskaug** en 1979 pour Smalltalk-80 sur des applications bureautiques. Comprendre son décalage avec le Web moderne évite de nombreuses confusions architecturales.

```
MVC Desktop / Smalltalk (Actif / Stateful) :
       ┌──────────────┐   User Event   ┌──────────────┐
       │     Vue      │───────────────►│  Contrôleur  │
       └──────▲───────┘                └──────┬───────┘
              │ Observer                      │ Modifie
              │ (Notification push)           ▼
       ┌──────┴───────────────────────────────┴───────┐
       │                   MODÈLE                     │
       └──────────────────────────────────────────────┘

MVC Web / Requête-Réponse (Passif / Stateless) :
 [Client HTTP] ────► [Contrôleur] ────► [Modèle]
       ▲                   │               │
       │ Envoi HTML/JSON   ▼ Rendu         ▼ Données
       └─────────────── [ Vue ] ◄──────────┘
```

* **MVC Originel (Client riche) :** La Vue observe directement le Modèle via le patron *Observer*. Dès que le Modèle change d'état, il notifie les Vues qui se repeignent automatiquement.
* **MVC Web (HTTP Request/Response) :** Le protocole HTTP est déconnecté et sans état (*stateless*). La Vue ne peut pas écouter activement le Modèle sur le serveur. Le cycle est séquentiel :
  1. Le client envoie une requête HTTP.
  2. Le **Contrôleur** intercepte l'appel et extrait les paramètres.
  3. Le Contrôleur charge et manipule le **Modèle**.
  4. Le Contrôleur passe les données à la **Vue** (moteur de template) ou sérialise une réponse (JSON).
  5. La connexion est close.

---

## 2. L'Anti-pattern Majeur : Le *Fat Controller*

Le piège le plus destructeur en architecture MVC est le contrôleur boursouflé (*Fat Controller*).

```
// ❌ ANTI-PATTERN : Le Contrôleur fait TOUT (Validation, SQL, Métier, Envoi de mail)
class OrderController {
  async checkout(req: Request, res: Response) {
    // 1. Validation manuelle des inputs
    if (!req.body.items || req.body.items.length === 0) {
      return res.status(400).send("Panier vide");
    }

    // 2. Requête directe en base de données
    const user = await db.query('SELECT * FROM users WHERE id = ?', [req.user.id]);

    // 3. Logique métier & calculs de réduction
    let total = 0;
    for (const item of req.body.items) {
      total += item.price * item.quantity;
    }
    if (user.isVip) total *= 0.9;

    // 4. Interaction passerelle de paiement tierce
    await stripe.charges.create({ amount: total, customer: user.stripeId });

    // 5. Écriture directe
    await db.query('INSERT INTO orders ...');

    // 6. Notification externe
    await sendGrid.sendMail({ to: user.email, subject: "Confirmation commande" });

    return res.render('checkout-success', { total });
  }
}
```

### Pourquoi ce code est toxique :
* **Non testable unitairement :** Pour tester le calcul de réduction de 10 %, il faut simuler l'objet HTTP `Request`, mocker la base SQL, Stripe et SendGrid.
* **Non réutilisable :** Si une API mobile ou une commande CLI a besoin de passer une commande, tout ce code doit être copié-collé.
* **Violation flagrante du SRP :** Le contrôleur a cinq raisons différentes de changer.

---

## 3. Modèle Anémique vs Modèle Riche (*Rich Domain Model*)

Pour dégraisser le contrôleur, beaucoup de développeurs déplacent le code vers le modèle, mais tombent dans un second écueil : le **Modèle Anémique**.

### 3.1 Le Modèle Anémique (*Anemic Domain Model*)
Le modèle n'est qu'un simple sac de propriétés passives (getters/setters ou colonnes SQL), privé de toute logique métier. La logique est alors aspirée soit par le contrôleur, soit par des classes "services" de 2000 lignes qui traitent l'objet de l'extérieur. Martin Fowler qualifie cela d'anti-pattern car il détruit l'intérêt de la programmation orientée objet.

### 3.2 Le Modèle Riche (*Skinny Controller, Fat Model*)
Le modèle encapsule ses données **et** les règles métier régissant ces données.

```typescript
// ✅ MODÈLE RICHE : L'entité protège ses invariants
class Order {
  private items: OrderItem[] = [];
  private status: 'DRAFT' | 'CONFIRMED' | 'PAID' = 'DRAFT';

  addItem(product: Product, quantity: number): void {
    if (this.status !== 'DRAFT') {
      throw new DomainError("Impossible de modifier une commande validée.");
    }
    if (quantity <= 0) {
      throw new DomainError("La quantité doit être strictement positive.");
    }
    this.items.push(new OrderItem(product, quantity));
  }

  calculateTotal(isVip: boolean): number {
    const rawTotal = this.items.reduce((sum, item) => sum + item.subtotal(), 0);
    return isVip ? rawTotal * 0.9 : rawTotal;
  }
}
```

---

## 4. L'Écosystème Moderne : Dégraisser Modèle ET Contrôleur

Le slogan historique « *Skinny Controller, Fat Model* » montre lui aussi ses limites : le modèle finit par absorber la validation HTTP, les requêtes SQL, l'envoi de mails et le formatage.

En MVC intermédiaire, on adjoint des composants spécialisés :

```
             ┌────────────────────────────────────────────────────────┐
             │                     REQUÊTE HTTP                       │
             └───────────────────────────┬────────────────────────────┘
                                         │
                                         ▼
             ┌────────────────────────────────────────────────────────┐
             │ 1. Form Request / DTO (Validation des formats d'entrée)│
             └───────────────────────────┬────────────────────────────┘
                                         │ Input assaini
                                         ▼
             ┌────────────────────────────────────────────────────────┐
             │ 2. Contrôleur (Médiateur ultra-fin : 5 à 15 lignes max)│
             └───────────────────────────┬────────────────────────────┘
                                         │ Appelle
                                         ▼
             ┌────────────────────────────────────────────────────────┐
             │ 3. Service Applicatif (Orchestration du cas d'usage)   │
             └─────────────┬────────────────────────────┬─────────────┘
                           │                            │
            Manipule       ▼                            ▼ Persiste
     ┌───────────────────────────┐         ┌───────────────────────────┐
     │ 4. Modèle Métier (Règles) │         │ 5. Repository (Accès SQL) │
     └───────────────────────────┘         └───────────────────────────┘
```

### Le Contrôleur moderne assaini :
```typescript
class OrderController {
  constructor(private readonly checkoutService: CheckoutService) {}

  async checkout(req: CheckoutFormRequest, res: Response) {
    // 1. Les données sont DÉJÀ validées par CheckoutFormRequest
    const command = req.toCommand();

    // 2. Le service orchestre le cas d'usage
    const orderResult = await this.checkoutService.execute(command);

    // 3. Transformation pour la vue via un ViewModel / Resource
    return res.status(201).render('order-summary', new OrderViewModel(orderResult));
  }
}
```

---

## 5. DTOs, ViewModels et Présentateurs

La Vue ne doit pas recevoir directement des objets de base de données bruts (ORM) :
* **Fuite d'informations sensibles :** Renvoyer l'objet `User` peut exposer par inadvertance le hash du mot de passe ou des tokens privés.
* **Problème de mise en forme :** La Vue n'a pas à faire des calculs comme `user.firstName + ' ' + user.lastName.toUpperCase()`.

* **DTO (*Data Transfer Object*) :** Objet simple et immuable servant à transporter les données entre le contrôleur et les services externes sans logique.
* **ViewModel / Resource / Presenter :** Objet dédié préparant les données pour l'affichage (ex: formatage de devises, libellés localisés, masquage de colonnes privées).

---

## 6. Panorama Comparatif : MVC, MVP et MVVM

Selon la plateforme cible, le pattern MVC a dérivé en différentes architectures d'interface :

| Critère | MVC (Modèle-Vue-Contrôleur) | MVP (Modèle-Vue-Présentateur) | MVVM (Modèle-Vue-VueModèle) |
| :--- | :--- | :--- | :--- |
| **Origine principale** | Web classique / Smalltalk | Applications de bureau clientes (WinForms, Android) | Frameworks réactifs (WPF, Angular, Vue, SwiftUI) |
| **Liaison Vue-Logique** | Le contrôleur choisit la vue à afficher. | Le Présentateur pilote la Vue via une interface explicite (`IView`). | Liaison de données bidirectionnelle automatique (*Data Binding*). |
| **Connaissance mutuelle** | La Vue connaît le Modèle (en lecture). | La Vue et le Modèle sont **totalement étanches** (isolés par le Présentateur). | La Vue observe le ViewModel ; le ViewModel ignore l'existence de la Vue. |
| **Testabilité de la vue** | Moyenne (moteurs de rendu dépendants du runtime). | **Très élevée** : on peut tester l'affichage en mockant l'interface `IView`. | **Très élevée** : on teste le ViewModel sans instancier l'interface graphique. |

---

## 7. Grille d'Auto-Évaluation de votre Code MVC

Posez-vous ces 4 questions lors de vos revues de code :

1. **Vos contrôleurs font-ils plus de 20 lignes par action ?** Si oui, vous avez probablement de la logique métier ou des requêtes d'infrastructure qui polluent l'aiguillage.
2. **Vos modèles sont-ils constitués uniquement de variables publiques sans méthodes ?** Si oui, vous avez un modèle anémique. Réinjectez les règles de validation et de calculs dans l'objet métier.
3. **Votre Vue fait-elle des requêtes ou des calculs complexes ?** Si oui, introduisez un ViewModel ou un Presenter pour lui fournir des données prêtes à afficher.
4. **Pouvez-vous tester votre logique métier sans monter un faux serveur HTTP ?** Si non, le contrôleur emprisonne des règles qui devraient vivre dans le domaine.