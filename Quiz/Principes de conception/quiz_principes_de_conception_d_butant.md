# Quiz d'Évaluation — Principes de Conception Logicielle (Niveau Novice)

Ce quiz évalue votre compréhension des réflexes fondamentaux de développement : les acronymes pratiques (KISS, YAGNI, DRY) ainsi que les 5 piliers de l'architecture logicielle (SOLID).

### Question 1 : Principe YAGNI (*You Aren't Gonna Need It*)

Quelle situation illustre une **violation** du principe YAGNI ?

* \[ \] A) Écrire un script d'export PDF complexe et paramétrable pour des formats de papier exotiques dès le premier jour, alors que l'application ne demande qu'un affichage texte simple à l'écran.
* \[ \] B) Factoriser trois fonctions de calcul de TVA identiques en une seule fonction partagée.
* \[ \] C) Utiliser un opérateur ternaire au lieu d'une instruction `if/else` classique.
* \[ \] D) Découper un fichier de 1000 lignes en plusieurs petits composants modulaires.

### Question 2 : Responsabilité Unique (*Single Responsibility Principle - SRP*)

Qu'indique concrètement le principe de Responsabilité Unique pour une classe ou une fonction ?

* \[ \] A) Elle ne doit comporter qu'une seule ligne de code exécutable.
* \[ \] B) Elle ne doit avoir qu'une seule et unique raison de changer.
* \[ \] C) Elle doit interdire à toute autre classe du projet d'accéder à ses méthodes publiques via un système de sécurité basé sur les tokens d'accès mémoire du système d'exploitation hôte.
* \[ \] D) Elle doit obligatoirement hériter d'une classe abstraite unique fournie par le framework sous-jacent.

### Question 3 : Principe KISS (*Keep It Simple, Stupid*)

Face à un problème algorithmique standard, que préconise le principe KISS ?

* \[ \] A) Implémenter immédiatement un patron de conception *Abstract Factory* combiné à un bus d'événements asynchrone pour anticiper de futures évolutions potentielles de la plateforme.
* \[ \] B) Choisir l'approche la plus directe et lisible plutôt qu'une solution sophistiquée et difficile à maintenir.
* \[ \] C) Supprimer les commentaires de documentation pour réduire la taille des fichiers sources sur le disque.
* \[ \] D) Écrire tout le code dans une seule fonction globale.

### Question 4 : Substitution de Liskov (*LSP*)

Considérez une classe parente `Rectangle` avec deux méthodes distinctes `setWidth(w)` et `setHeight(h)`. Si une sous-classe `Carre` hérite de `Rectangle` et force la largeur à être égale à la hauteur à chaque modification, quel principe risque d'être enfreint ?

* \[ \] A) Le principe DRY, car les variables de largeur et hauteur sont dupliquées dans la mémoire vive de l'ordinateur.
* \[ \] B) Le principe Open/Closed, car la classe `Carre` refuse d'accueillir de nouvelles méthodes publiques.
* \[ \] C) Le principe de Substitution de Liskov (LSP).
* \[ \] D) Le principe KISS, parce que l'utilisation de formes géométriques est trop abstraite pour un programmeur débutant.

### Question 5 : Ségrégation des Interfaces (*ISP*)

Un collègue conçoit une interface `Vehicle` contenant `drive()`, `fly()` et `sail()`. La classe `Bicycle` implémente cette interface mais lève une exception sur `fly()` et `sail()`. Quelle recommandation correspond au principe ISP ?

* \[ \] A) Découper `Vehicle` en plusieurs interfaces spécifiques (`Drivable`, `Flyable`, `Sailable`) afin que `Bicycle` n'implémente que ce qui lui correspond.
* \[ \] B) Ajouter une méthode `hover()` dans l'interface `Vehicle` pour gérer également les aéroglisseurs.
* \[ \] C) Masquer les erreurs avec un bloc `try/catch` vide à l'intérieur de la classe `Bicycle` pour éviter les plantages au runtime.
* \[ \] D) Transformer `Bicycle` en classe parente de `Vehicle`.

### Question 6 : Principe DRY (*Don't Repeat Yourself*)

Quelle est la nuance importante à garder à l'esprit avant de factoriser du code sous prétexte d'appliquer le principe DRY ?

* \[ \] A) Le compilateur refuse d'exécuter un programme si le même nom de variable apparaît deux fois dans deux fichiers différents.
* \[ \] B) Deux blocs de code visuellement semblables ne représentent pas forcément la même logique métier : une duplication accidentelle vaut souvent mieux qu'une mauvaise abstraction prématurée.
* \[ \] C) Le principe DRY s'applique uniquement aux bases de données SQL relationnelles et ne concerne pas le code TypeScript ou JavaScript.
* \[ \] D) Il est interdit d'écrire des boucles `for` dans du code respectant DRY.

### Question 7 : Inversion des Dépendances (*DIP*)

Dans une application e-commerce, pourquoi est-il préférable qu'un service de commande dépende d'une interface `PaymentGateway` plutôt que directement de la classe concrète `StripeClient` ?

* \[ \] A) Pour pouvoir changer de prestataire ou simuler les paiements lors des tests unitaires sans modifier le code du service de commande.
* \[ \] B) Parce que TypeScript interdit d'instancier des classes avec le mot-clé `new` dans les fichiers métier.
* \[ \] C) Pour chiffrer automatiquement le numéro de carte bancaire dans le CPU.
* \[ \] D) Cela divise par deux le temps de compilation.

## Corrigé et Explications

### 1. Réponse : **A**
**Explication :** YAGNI (*You Aren't Gonna Need It*) stipule qu'on ne doit jamais coder une fonctionnalité ou une flexibilité basée sur des suppositions futures tant que le besoin réel n'est pas exprimé.

### 2. Réponse : **B**
**Explication :** La définition classique du SRP par Robert C. Martin est : « Un module ne doit avoir qu'une seule raison de changer », ce qui signifie qu'il ne doit être responsable que d'un seul acteur ou d'un seul domaine fonctionnel précis.

### 3. Réponse : **B**
**Explication :** KISS recommande la simplicité avant tout. Une solution claire et directe est plus facile à déboguer, à tester et à faire évoluer qu'un montage d'ingénierie inutilement complexe.

### 4. Réponse : **C**
**Explication :** C'est l'exemple historique du problème Rectangle/Carré de Barbara Liskov. Si une fonction attend un `Rectangle` et modifie sa largeur en s'attendant à ce que la hauteur ne bouge pas, passer un `Carre` viole ce contrat et casse le comportement attendu sans erreur de compilation.

### 5. Réponse : **A**
**Explication :** L'Interface Segregation Principle (ISP) préconise de petites interfaces modulaires adaptées aux clients plutôt qu'une interface volumineuse qui force des classes à implémenter des méthodes inutiles ou non applicables.

### 6. Réponse : **B**
**Explication :** DRY concerne la duplication de la **connaissance métier**, pas seulement la similitude visuelle du code. Factoriser trop tôt deux logiques qui n'évolueront pas ensemble crée un couplage toxique.

### 7. Réponse : **A**
**Explication :** Le DIP préconise de dépendre d'abstractions (interfaces). Cela découple la logique de haut niveau (le traitement d'une commande) des détails d'implémentation (l'API Stripe, PayPal, ou un Mock pour les tests).