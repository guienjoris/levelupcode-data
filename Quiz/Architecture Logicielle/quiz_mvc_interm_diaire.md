# Quiz d'Évaluation — Pattern MVC Approfondi (Niveau Intermédiaire)

Ce quiz teste votre compréhension approfondie des mécanismes du pattern MVC : la distinction entre MVC Desktop originel et MVC Web, la lutte contre le *Fat Controller*, le modèle anémique vs modèle riche, l'usage des DTO/ViewModels et la comparaison avec MVP et MVVM.

### Question 1 : MVC Originel vs MVC Web

Pourquoi le pattern MVC tel qu'implémenté dans les frameworks web modernes (ex. Laravel, Django, Rails) diffère-t-il fondamentalement du modèle MVC conçu originellement pour Smalltalk-80 ?

* \[ \] A) Le protocole HTTP est déconnecté et sans état (*stateless*), ce qui empêche la Vue côté client d'observer en continu les mutations d'état du Modèle via un patron *Observer* actif sur le serveur.
* \[ \] B) Parce que le MVC originel interdisait l'usage de contrôleurs intermédiaires pour manipuler les données métier.
* \[ \] C) Les navigateurs web modernes ne supportent que le rendu vectoriel natif et refusent les architectures orientées objet.
* \[ \] D) Le MVC originel utilisait exclusivement le format JSON.

### Question 2 : Le Modèle Anémique (*Anemic Domain Model*)

Selon la critique formulée par Martin Fowler, qu'est-ce qui caractérise un **Modèle Anémique** ?

* \[ \] A) Un modèle qui ne contient que des structures de données passives (propriétés, getters et setters) dépourvues de logique métier, cette dernière étant déportée dans des services ou des contrôleurs externes.
* \[ \] B) Une classe qui gère simultanément l'affichage HTML, les transactions SQL et l'envoi de notifications push.
* \[ \] C) Une entité qui refuse d'être sérialisée au format JSON lors des échanges réseau.
* \[ \] D) Une base de données sans clés primaires.

### Question 3 : Résolution de l'anti-pattern *Fat Controller*

Quelle est l'approche la plus saine pour alléger un contrôleur qui effectue la validation des entrées, des calculs de remise, des appels à une API de paiement et l'envoi d'e-mails ?

* \[ \] A) Transférer l'ensemble de ce code directement dans la Vue à l'aide de scripts intégrés dans les templates.
* \[ \] B) Déléguer la validation à un DTO/Form Request, confier l'orchestration du cas d'usage à un Service applicatif et isoler les règles de calcul dans le Modèle métier.
* \[ \] C) Fusionner le contrôleur et la base de données au sein d'une seule procédure stockée SQL globale.
* \[ \] D) Augmenter le timeout de la requête HTTP.

### Question 4 : Rôle du ViewModel / Presenter

Quel est le bénéfice principal d'intercaler un **ViewModel** ou un **Presenter** entre le Modèle et la Vue plutôt que de transmettre directement l'entité de base de données à l'interface ?

* \[ \] A) Cela supprime l'obligation d'installer un moteur de rendu de templates sur le serveur de production.
* \[ \] B) Formater les données spécifiquement pour l'affichage tout en évitant d'exposer des champs sensibles ou des méthodes internes de l'entité.
* \[ \] C) Permettre à la base de données relationnelle d'exécuter des requêtes sans passer par le réseau local de l'hébergeur.
* \[ \] D) Forcer la Vue à recompiler son code CSS.

### Question 5 : Comparaison avec le pattern MVP (*Model-View-Presenter*)

Dans l'architecture MVP, comment s'articule la relation entre la **Vue** et le **Modèle** ?

* \[ \] A) La Vue et le Modèle sont totalement découplés et ne se connaissent pas : le Présentateur fait office d'intermédiaire strict en pilotant la Vue via une interface.
* \[ \] B) La Vue modifie directement les propriétés du Modèle sans notifier le Présentateur.
* \[ \] C) Le Modèle instancie la Vue dynamiquement à chaque interaction utilisateur.
* \[ \] D) Ils partagent la même mémoire vive sans passer par une couche d'abstraction logicielle.

### Question 6 : Le modèle MVVM et le Data Binding

Quelle caractéristique distingue principalement le pattern **MVVM** (*Model-View-ViewModel*) de l'approche MVC classique ?

* \[ \] A) L'interdiction formelle d'utiliser du code asynchrone lors du chargement des ressources graphiques.
* \[ \] B) L'existence d'une liaison de données bidirectionnelle (*Data Binding*) liant automatiquement l'état du ViewModel aux éléments de la Vue.
* \[ \] C) Le remplacement systématique des requêtes HTTP par des flux de fichiers binaires sur socket brut.
* \[ \] D) L'absence complète de données dans l'application.

### Question 7 : Responsabilité légitime du Contrôleur

Parmi les actions suivantes, laquelle constitue une responsabilité **légitime** d'un Contrôleur dans une architecture MVC mature ?

* \[ \] A) Définir les contraintes de table et exécuter des requêtes SQL natives à l'aide d'un pool de connexions directes.
* \[ \] B) Réceptionner la requête HTTP, convertir les paramètres en commande/DTO pour le service applicatif et sélectionner la réponse ou la vue appropriée.
* \[ \] C) Calculer les pénalités de retard d'une facture en appliquant les barèmes légaux en vigueur.
* \[ \] D) Implémenter l'algorithme de hachage cryptographique des mots de passe des utilisateurs.

## Corrigé et Explications Détaillées

### 1. Réponse : **A**
**Explication :** Dans le MVC originel sur système d'exploitation de bureau (Desktop), la Vue écoute le Modèle via le pattern *Observer* : toute modification de données émet un événement notifiant la Vue pour qu'elle se redessine immédiatement. Sur le Web classique avec le protocole HTTP sans état (*stateless*), ce flux continu n'existe pas : le Contrôleur intercepte une requête ponctuelle, coordonne le Modèle, passe les données à la Vue pour générer une réponse (HTML ou JSON), puis la connexion est close.

### 2. Réponse : **A**
**Explication :** Le modèle anémique se réduit à des structures passives composées de données pures (champs, accesseurs) sans comportement métier. La logique se retrouve externalisée dans les contrôleurs ou les services, ce qui contredit le principe même de la POO (qui vise à encapsuler données et comportements ensemble) et disperse les règles d'intégrité du domaine.

### 3. Réponse : **B**
**Explication :** Pour assainir un *Fat Controller*, on applique le principe de responsabilité unique (SRP) : les *Form Requests* ou DTOs s'occupent de la conformité du payload d'entrée, le *Service Applicatif* orchestre les actions (paiement, email, persistance), et le *Modèle Riche* porte les règles métier intrinsèques (calcul de rabais, statuts). Le contrôleur redevient un coordinateur minimal.

### 4. Réponse : **B**
**Explication :** Passer l'entité ORM directement à la vue expose l'application à des fuites de données (champs privés, secrets) et encombre le template de logique de formatage (concaténation de noms, devises, dates). Le *ViewModel* prépare un objet sur mesure contenant exclusivement ce qui doit être affiché.

### 5. Réponse : **A**
**Explication :** Contrairement au MVC où la Vue lit directement le Modèle, le pattern MVP introduit un découplage total : la Vue et le Modèle s'ignorent mutuellement. Le Présentateur manipule le Modèle, puis met à jour la Vue en interagissant avec elle au travers d'une interface abstraite (`IView`), ce qui rend la vue facilement mockable pour les tests.

### 6. Réponse : **B**
**Explication :** MVVM, très populaire dans les frameworks réactifs (Angular, Vue, SwiftUI, WPF), repose sur le *Data Binding*. Lorsque la propriété d'un ViewModel change, la Vue se met à jour automatiquement dans le DOM sans code impératif de manipulation graphique, et inversement lorsque l'utilisateur saisit une valeur.

### 7. Réponse : **B**
**Explication :** Le rôle d'un contrôleur moderne est purement médiateur : c'est un adaptateur entre le protocole de transport (HTTP, websockets, CLI) et la logique applicative. Il reçoit l'entrée, déclenche l'exécution du cas d'usage, et choisit la forme de restitution (code de statut HTTP, redirection, vue HTML ou payload JSON).