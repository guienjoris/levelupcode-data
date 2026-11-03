# TypeScript Débutant — Pratique Quotidienne : Inférence, Unions & Typage Robuste

TypeScript ajoute un système de types statiques au-dessus de JavaScript. En pratique, bien débuter ne consiste pas à tout annoter manuellement avec du bruit visuel, mais à comprendre comment le compilateur raisonne, comment restreindre des choix possibles et comment modéliser des objets sans perdre en lisibilité.

## 1. Inférence de Type vs Annotation Explicite

Le compilateur TypeScript (`tsc`) est capable de déduire automatiquement le type d'une variable à partir de sa valeur initiale. C'est l'**inférence de type**.

```typescript
// Inférence automatique (recommandé quand la valeur initiale est évidente)
let score = 100; // TypeScript infère : number
// score = "gagné"; // ❌ Erreur : Type 'string' is not assignable to type 'number'.

// Annotation explicite (utile si la variable n'a pas de valeur immédiate)
let nomClient: string;
nomClient = 'Camille';
```

> **Règle d'or :** N'annotez pas ce qui est évident. Écrire `const nom: string = "Alice";` est redondant ; `const nom = "Alice";` suffit et est plus lisible.

## 2. Types Unions (`|`) et Rétrécissement Basique (*Type Narrowing*)

Dans le code réel, une fonction ou une variable reçoit souvent plus d'un type possible (par exemple un identifiant qui peut être un nombre ou un texte). On utilise alors une **Union de types** avec le symbole pipe (`|`).

Dès qu'une variable possède un type union, TypeScript interdit d'exécuter une méthode qui n'existe pas sur tous les membres de l'union. Il faut d'abord **rétrécir** le type à l'aide d'un garde simple comme `typeof`.

```typescript
function formaterIdentifiant(id: string | number): string {
  // ❌ Erreur directe : id.toUpperCase() n'existe pas sur number !

  if (typeof id === 'string') {
    // Dans ce bloc, TypeScript sait avec certitude que 'id' est une string
    return id.toUpperCase();
  } else {
    // Ici, par déduction, 'id' est obligatoirement un number
    return `ID-${id.toFixed(0)}`;
  }
}
```

## 3. Typage des Fonctions : Paramètres et Valeurs de Retour

Les fonctions représentent le point d'entrée principal des erreurs. On type systématiquement leurs paramètres d'entrée, tandis que la valeur de retour peut être inférée ou explicitée.

```typescript
// 1. Paramètre optionnel avec '?' (vaut undefined s'il est omis)
function saluer(prenom: string, titre?: string): string {
  if (titre) {
    return `Bonjour ${titre} ${prenom}`;
  }
  return `Bonjour ${prenom}`;
}

// 2. Paramètre avec valeur par défaut (type inféré automatiquement)
function creerLigne(libelle: string, quantite = 1): string {
  return `${quantite}x ${libelle}`;
}

// 3. Fonction ne renvoyant rien : retour 'void'
function notifierLog(message: string): void {
  console.log(`[LOG] : ${message}`);
}
```

## 4. `type` vs `interface` pour Modéliser les Objets

Pour décrire la forme d'un objet en TypeScript, deux outils principaux coexistent. Au niveau débutant, les deux s'utilisent souvent de manière équivalente, avec quelques nuances d'intention :

```typescript
// Déclaration via une Interface (très courante pour les modèles métiers)
interface Article {
  readonly id: number;      // 'readonly' empêche la réassignation
  titre: string;
  prix: number;
  description?: string;     // Propriété optionnelle
}

// Déclaration via un Type Alias (permet de nommer n'importe quel type, y compris des unions)
type StatutCommande = 'panier' | 'payee' | 'expediee'; // Types littéraux !

type Commande = {
  reference: string;
  articles: Article[];
  statut: StatutCommande;
};
```

* **Types littéraux :** `'panier' | 'payee'` empêche toute faute de frappe en restreignant la valeur aux chaînes exactes autorisées.
* **`readonly` :** Protège contre les modifications accidentelles d'identifiants ou de propriétés sensibles.

## 5. Le Danger de `any` face à `unknown`

Lorsqu'on ignore le type d'une donnée (ex. réponse brute d'une API externe ou saisie libre), l'erreur courante est d'utiliser `any`.

* **`any` (L'échappatoire risquée) :** Il désactive totalement le compilateur. TypeScript autorise n'importe quelle méthode sans vérification, ramenant le code à du JavaScript non protégé.
* **`unknown` (L'inconnu sécurisé) :** Il force le développeur à vérifier le type de la variable avant de pouvoir s'en servir.

```typescript
// ❌ DANGEREUX : any désactive toute sécurité
let donneeInconnue: any = 42;
// Ne produit aucune erreur à la compilation, mais plante au runtime !
// donneeInconnue.toLowerCase(); // 💥 TypeError: not a function

// ✅ SÉCURISÉ : unknown impose une vérification
let reponseApi: unknown = "Résultat";

// ❌ TypeScript bloque ici : Object is of type 'unknown'.
// reponseApi.trim();

// Vérification obligatoire avant manipulation :
if (typeof reponseApi === 'string') {
  console.log(reponseApi.trim()); // ✅ Autorisé car le type a été prouvé
}
```