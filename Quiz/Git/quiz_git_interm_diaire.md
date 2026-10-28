# Quiz d'Évaluation — Git (Niveau Intermédiaire)

Ce quiz teste vos connaissances sur la manipulation du graphe de commits, la résolution de conflits, le rebase, le stash, les différents modes de reset et l'usage de cherry-pick.

---

### Question 1 : Merge vs Rebase

Quelle est la conséquence directe d'un `git rebase main` effectué depuis une branche de travail `feature` ?

- [ ] A) Git crée obligatoirement un commit de fusion à deux parents et fige l'historique sans jamais altérer les hashs SHA-1 existants.
- [ ] B) Les commits de la branche `feature` sont rejoués un à un au sommet de `main`, ce qui réécrit leurs identifiants SHA-1 et produit un historique linéaire.
- [ ] C) Git supprime définitivement tous les commits de la branche distante `origin/main` pour aligner le serveur sur votre copie locale.
- [ ] D) L'historique devient linéaire.

---

### Question 2 : Annulation sur branche partagée

Vous avez poussé par erreur un commit défectueux sur la branche publique partagée `main`. Quelle commande est la plus appropriée pour annuler ce commit de manière sécurisée et collaborative ?

- [ ] A) `git reset --hard HEAD~1` suivi d'un `git push --force origin main` pour effacer toute trace du commit sur le dépôt distant sans laisser d'historique.
- [ ] B) `git revert <commit_hash>`
- [ ] C) `git rebase -i HEAD~2` en remplaçant le mot `pick` par `drop` afin de nettoyer le serveur sans avertir les collaborateurs.
- [ ] D) `git stash drop` appliqué directement sur la référence distante du dépôt GitHub de production.

---

### Question 3 : Remisage de fichiers non suivis (`stash`)

Par défaut, la commande `git stash` ne prend pas en compte les fichiers non suivis (*untracked*). Quelle option permet d'inclure ces fichiers lors de la mise en réserve ?

- [ ] A) `git stash -u`
- [ ] B) `git stash --force-all-tracked-and-untracked-files-including-ignored`
- [ ] C) `git stash --merge-untracked-to-index-cache-recursively`
- [ ] D) `git stash --commit-untracked-into-temporary-stash-buffer`

---

### Question 4 : Les modes de `git reset`

Si vous exécutez `git reset --soft HEAD~1`, dans quel état se retrouvent les modifications apportées par le commit annulé ?

- [ ] A) Les modifications sont définitivement effacées du disque dur et de la mémoire de travail sans aucun moyen de les récupérer via Git.
- [ ] B) Les modifications restent dans le répertoire de travail (*working directory*) mais sont retirées de la zone de transit (*staging area*).
- [ ] C) Elles sont conservées dans la zone de transit (*staging area*), prêtes à être à nouveau commitées.
- [ ] D) Elles sont automatiquement converties en un fichier `.patch` enregistré dans le dossier `.git/lost-found/other`.

---

### Question 5 : Rebase interactif

Dans un rebase interactif (`git rebase -i`), quelle commande permet de fusionner un commit dans le précédent tout en supprimant son message pour ne pas encombrer l'historique ?

- [ ] A) `reword`
- [ ] B) `edit`
- [ ] C) `fixup`
- [ ] D) `squash`

---

### Question 6 : Résolution de conflit après rebase

Après avoir résolu manuellement les conflits survenus lors d'un `git rebase` et avoir indexé les fichiers corrigés avec `git add`, quelle commande devez-vous exécuter pour poursuivre le rebase ?

- [ ] A) `git commit -m "Merge branch feature"` pour valider la branche et créer le commit d'entrelacement.
- [ ] B) `git rebase --continue`
- [ ] C) `git push --force-with-lease` pour envoyer immédiatement les modifications non finalisées sur le serveur distant.
- [ ] D) `git merge --continue`

---

### Question 7 : Usage de `cherry-pick`

Que réalise la commande `git cherry-pick <commit_hash>` ?

- [ ] A) Elle applique les modifications introduites par ce commit spécifique au sommet de la branche courante sous la forme d'un nouveau commit.
- [ ] B) Elle fusionne l'intégralité de la branche d'origine du commit dans la branche active en effectuant un merge à avance rapide (*fast-forward*).
- [ ] C) Elle déplace définitivement le commit d'une branche vers une autre en le supprimant de son emplacement d'origine.
- [ ] D) Elle inspecte le commit pour vérifier s'il contient des vulnérabilités de sécurité avant d'autoriser son déploiement en pré-production.

---

## Corrigé et Explications

1. **Réponse B** : Le rebase "déracine" les commits de la branche pour les recalculer au-dessus du sommet de la branche cible, générant ainsi de nouveaux identifiants SHA-1 et un historique sans bifurcation.
2. **Réponse B** : Sur une branche publique, `git revert` est la méthode standard car elle n'altère pas le passé : elle crée un nouveau commit portant les modifications inverses, évitant ainsi de casser l'historique local de vos collègues.
3. **Réponse A** : Le drapeau `-u` (ou `--include-untracked`) force Git à inclure les fichiers non suivis dans le stash en plus des fichiers modifiés déjà répertoriés.
4. **Réponse C** : Le mode `--soft` déplace uniquement la référence de `HEAD` vers le commit antérieur sans toucher à l'index ni au répertoire de travail ; le travail reste donc prêt pour un nouveau commit immédiat.
5. **Réponse C** : `fixup` (ou `f`) intègre le commit dans le commit parent direct en ignorant son message, à la différence de `squash` qui ouvre l'éditeur pour concaténer les messages.
6. **Réponse B** : Pendant un rebase, Git applique les commits un à un. Quand un conflit est réglé et indexé, on indique à Git de reprendre le processus avec `git rebase --continue`.
7. **Réponse A** : `cherry-pick` extrait uniquement le différentiel d'un commit unique et le rejoue sur la branche active en générant un nouveau SHA-1, sans ramener le reste de la branche d'origine.