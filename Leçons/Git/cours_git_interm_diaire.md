# Git — Niveau Intermédiaire : Maîtriser le Flux de Travail et l'Historique

Le passage au niveau intermédiaire nécessite de dépasser le simple cycle `add` / `commit` / `push`. Il s'agit de comprendre comment manipuler le graphe de commits, résoudre les divergences sereinement et maintenir un historique lisible et exploitable.

---

## 1. Fusion de branches : `merge` vs `rebase`

Deux stratégies principales permettent d'intégrer les modifications d'une branche dans une autre.

### 1.1 `git merge` (Intégration non destructive)

Un merge crée un nouveau commit spécial doté de deux parents (sauf en cas de *Fast-Forward*).

```text
Avant merge :
      C---D (feature)
     /
A---B (main)

Après : git checkout main && git merge feature
      C---D (feature)
     /     \
A---B-------M (main)  <-- Commit de merge
```

* **Avantages :** Historique fidèle à la chronologie réelle, opération non destructive.
* **Inconvénients :** Historique vite encombré par de nombreux commits de merge ("branches en tresse").

### 1.2 `git rebase` (Réécriture linéaire)

Le rebase détache vos commits de leur base d'origine pour les rejouer un par un au sommet de la branche cible.

```text
Avant rebase :
      C---D (feature)
     /
A---B---E (main)

Après : git checkout feature && git rebase main
            C'---D' (feature, nouveaux SHA-1)
           /
A---B---E (main)
```

* **Avantages :** Historique parfaitement linéaire, bisect et lecture simplifiés.
* **Inconvénients :** Les identifiants (SHA-1) changent.
* **Règle d'or absolue :** Ne **jamais** rebaser une branche publique partagée sur laquelle d'autres personnes travaillent déjà.

---

## 2. Anatomie et résolution des conflits

Un conflit survient lorsque deux branches modifient les mêmes lignes d'un même fichier de façon concurrente.

### 2.1 Les marqueurs de conflit

Git suspend l'opération en cours et insère des délimiteurs visuels directement dans le fichier :

```text
<<<<<<< HEAD (votre branche courante)
const API_URL = "https://api.staging.example.com";
=======
const API_URL = "https://api.production.example.com";
>>>>>>> feature-config (la branche entrante)
```

### 2.2 Protocole de résolution

1. Inspecter les fichiers en conflit : `git status`.
2. Ouvrir le fichier et supprimer manuellement les marqueurs (`<<<<<<<`, `=======`, `>>>>>>>`) en conservant le code souhaité.
3. Marquer le conflit comme résolu : `git add <fichier>`.
4. Finaliser l'opération :
   * Pour un merge : `git commit`.
   * Pour un rebase : `git rebase --continue` (ou `git rebase --abort` pour annuler).

---

## 3. Le Remisage : `git stash`

`git stash` permet de mettre de côté des modifications non commitées du répertoire de travail afin de changer de contexte rapidement (ex: appliquer un hotfix urgent).

```bash
# Remiser les modifications (suivies)
git stash save "WIP: formulaire en cours"

# Lister les remises empilées
git stash list
# stash@{0}: On main: WIP: formulaire en cours

# Réappliquer la dernière remise et la supprimer de la pile
git stash pop

# Réappliquer sans la supprimer de la pile
git stash apply stash@{0}

# Jeter une remise spécifique
git stash drop stash@{0}
```

> **Astuce :** Pour remiser également les fichiers non suivis (*untracked*), ajoutez l'option `-u` (`git stash -u`).

---

## 4. Annuler des changements : `reset` vs `revert`

Comprendre la différence entre ces deux commandes est crucial pour la sécurité de votre historique.

| Critère | `git reset` | `git revert` |
| :--- | :--- | :--- |
| **Mécanisme** | Déplace le pointeur de branche vers l'arrière (réécrit le passé). | Crée un **nouveau commit** qui applique l'inverse strict du commit ciblé. |
| **Usage recommandé** | Uniquement sur des branches locales privées. | Sur les branches partagées / publiques (`main`). |
| **Impact historique** | Efface ou masque les commits ciblés. | Conserve l'historique intact avec traçabilité. |

### Les modes de `git reset`

```bash
# 1. Soft : Déplace HEAD, conserve les modifications dans le Staging Area (prêtes à commiter)
git reset --soft HEAD~1

# 2. Mixed (par défaut) : Déplace HEAD, conserve les modifications dans le Working Directory (non indexées)
git reset --mixed HEAD~1

# 3. Hard : DANGER — Écrase HEAD, le Staging et le Working Directory (modifications perdues)
git reset --hard HEAD~1
```

### Inverser proprement avec `revert`

```bash
# Annule les effets du commit a1b2c3d via un nouveau commit inverse
git revert a1b2c3d
```

---

## 5. Nettoyer son historique local : Rebase Interactif (`git rebase -i`)

Avant de soumettre une Pull Request, il est d'usage de nettoyer ses commits temporaires (`wip`, `fix typo`, etc.).

```bash
# Rebaser interactivement les 4 derniers commits
git rebase -i HEAD~4
```

Git ouvre votre éditeur avec un script d'instructions :

```text
pick 4a3b2c1 feat: début du composant tableau
squash 8d7e6f5 fix: correction padding
squash 2c1d0e9 fix: typage typescript
reword f5e4d3c docs: ajout readme composant

# Commandes disponibles :
# p, pick = conserver le commit tel quel
# r, reword = conserver le commit mais modifier son message
# e, edit = s'arrêter pour amender le contenu
# s, squash = fusionner dans le commit précédent (combine les messages)
# f, fixup = fusionner dans le commit précédent en jetant ce message
# d, drop = supprimer complètement le commit
```

---

## 6. Récupération chirurgicale : `git cherry-pick`

`git cherry-pick` permet de prélever un commit précis d'une autre branche sans la fusionner en entier :

```bash
# Applique le commit dont le SHA est 3f2a1b9 sur la branche courante
git cherry-pick 3f2a1b9
```

Cas d'usage typique : un correctif urgent a été commité par mégarde sur une branche de feature ; on le *cherry-pick* directement sur `main`.

---

## 7. Tableau de Synthèse des Commandes Intermédiaires

* `git rebase <branche>` : Rejoue la branche courante au sommet de la branche cible.
* `git rebase -i HEAD~N` : Nettoie, fusionne ou réordonne les $N$ derniers commits.
* `git stash -u` : Met de côté l'état de travail (y compris les fichiers non suivis).
* `git reset --soft HEAD~1` : Défait le dernier commit tout en gardant les fichiers indexés.
* `git revert <hash>` : Annule publiquement un commit sans réécrire le graphe.
* `git cherry-pick <hash>` : Importe un commit isolé sur la branche active.