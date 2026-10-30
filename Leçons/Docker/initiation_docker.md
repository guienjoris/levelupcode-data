# Initiation à Docker & à la Conteneurisation (Niveau Novice)

« *Pourtant, ça marche très bien sur ma machine !* » Cette phrase résume à elle seule la raison d'être de **Docker**.

Créé en 2013, Docker a transformé le déploiement applicatif en standardisant la manière d'empaqueter, distribuer et exécuter des logiciels. Il garantit qu'une application s'exécute exactement de la même manière sur votre ordinateur portable, sur le serveur de test et en production dans le cloud.

---

## 1. Le Problème Historique : La Matrice Infernale du Déploiement

Avant les conteneurs, déployer une application web nécessitait d'installer directement sur le système hôte :
- La bonne version du langage (ex: Node.js 20, Python 3.11)
- Les bibliothèques système compatibles
- La base de données avec sa configuration propre

```
Machine Développeur (macOS)         Serveur de Production (Ubuntu)
┌─────────────────────────┐         ┌─────────────────────────┐
│ Node.js 20.2            │   ──►   │ Node.js 18.1 (Obsolète) │ ──► CRASH !
│ PostgreSQL 16           │   ──►   │ PostgreSQL 14           │ ──► Erreur SQL
│ Dépendances locales     │   ──►   │ Paquets manquants       │ ──► Échec
└─────────────────────────┘         └─────────────────────────┘
```

**La solution Docker :** Au lieu d'installer l'environnement sur la machine, on enferme l'application **et tout ce dont elle a besoin pour tourner** dans une boîte scellée et portable : le **conteneur**.

---

## 2. Conteneur vs Machine Virtuelle (VM)

On compare souvent les conteneurs aux machines virtuelles (comme VirtualBox ou VMware). Pourtant, leur mécanique est radicalement différente :

```
Machine Virtuelle (VM)                  Conteneur Docker
┌─────────────────────────────────┐     ┌─────────────────────────────────┐
│ App 1          │ App 2          │     │ App 1          │ App 2          │
├────────────────┼────────────────┤     ├────────────────┼────────────────┤
│ Libs / Bins    │ Libs / Bins    │     │ Libs / Bins    │ Libs / Bins    │
├────────────────┼────────────────┤     ├────────────────┴────────────────┤
│ OS Invité 1    │ OS Invité 2    │     │      Moteur Docker (Engine)     │
│ (ex: Ubuntu)   │ (ex: Debian)   │     │                                 │
├────────────────┴────────────────┤     ├─────────────────────────────────┤
│ Hyperviseur (Type 1 ou 2)       │     │     Noyau OS Hôte (Linux)       │
├─────────────────────────────────┤     ├─────────────────────────────────┤
│ Matériel (CPU, RAM, Disque)     │     │ Matériel (CPU, RAM, Disque)     │
└─────────────────────────────────┘     └─────────────────────────────────┘
```

| Critère | Machine Virtuelle (VM) | Conteneur Docker |
| :--- | :--- | :--- |
| **Isolation** | Totale (émule un matériel complet et un OS invité) | Légère (isole les processus via le noyau Linux) |
| **Poids** | Plusieurs Go (gourmand en stockage) | Quelques dizaines ou centaines de Mo |
| **Démarrage** | Plusieurs dizaines de secondes à minutes | **Quasi-instantané** (quelques millisecondes) |
| **Performance** | Surcharge CPU/RAM notable due à l'OS invité | Performances quasi natives (exécute sur le noyau hôte) |

---

## 3. Le Triptyque Fondamental : Dockerfile, Image et Conteneur

Pour comprendre Docker, il suffit d'assimiler ce cycle en 3 étapes :

```
  [ Dockerfile ]  ──(docker build)──►  [ Image Docker ]  ──(docker run)──►  [ Conteneur ]
(La recette texte)                    (Le gâteau surgelé)                   (Le gâteau servi)
```

1. **Le Dockerfile (La Recette) :** Un fichier texte brut contenant la liste ordonnée des instructions pour fabriquer votre environnement.
2. **L'Image Docker (Le Modèle figé) :** Le résultat de la compilation du Dockerfile. Une image est **immuable** (en lecture seule), portable et stockable sur un registre (comme Docker Hub).
3. **Le Conteneur (L'Instance vivante) :** Un processus isolé en cours d'exécution basé sur une image. Vous pouvez créer 10 conteneurs identiques à partir d'une seule image.

---

## 4. Anatomie d'un Dockerfile Simple

Voici un `Dockerfile` commenté pour une petite application web :

```dockerfile
# 1. Image de départ officielle (légère basée sur Alpine Linux)
FROM node:20-alpine

# 2. Définir le dossier de travail interne au conteneur
WORKDIR /app

# 3. Copier les listes de dépendances en premier (pour optimiser le cache)
COPY package*.json ./

# 4. Installer les dépendances
RUN npm install

# 5. Copier le reste du code source
COPY . .

# 6. Indiquer le port réseau utilisé par l'application
EXPOSE 3000

# 7. Commande exécutée au démarrage du conteneur
CMD ["node", "server.js"]
```

> **La notion de couches (*Layers*) :** Chaque instruction (`FROM`, `COPY`, `RUN`) crée une couche en lecture seule mise en cache. Si vous ne modifiez que votre code sans toucher à `package.json`, Docker réutilise le cache de `RUN npm install` sans réinstaller les paquets !

---

## 5. Les Commandes Essentielles du Quotidien

### 5.1 Construire et Lancer

```bash
# Construire une image et lui donner un nom (tag) avec le point final désignant le dossier courant
docker build -t mon-app:1.0 .

# Lancer un conteneur en tâche de fond (-d) en associant le port local 8080 au port interne 3000 (-p)
docker run -d -p 8080:3000 --name instance-app mon-app:1.0
```

> **Comprendre `-p 8080:3000` :** Le premier chiffre est le port sur **votre machine**, le second est le port **dans le conteneur** (`machine_hote:conteneur`).

### 5.2 Inspecter et Suivre

```bash
# Lister les conteneurs actuellement en cours d'exécution
docker ps

# Lister tous les conteneurs (même arrêtés)
docker ps -a

# Consulter les logs (la console) d'un conteneur en direct (-f pour follow)
docker logs -f instance-app

# Ouvrir un terminal interactif à l'intérieur du conteneur en cours d'exécution
docker exec -it instance-app sh
```

### 5.3 Nettoyer

```bash
# Arrêter un conteneur
docker stop instance-app

# Supprimer un conteneur arrêté
docker rm instance-app

# Supprimer une image locale
docker rmi mon-app:1.0

# Nettoyage automatique complet des conteneurs arrêtés et images orphelines
docker system prune -f
```

---

## 6. Persistance des Données : Les Volumes

Par défaut, le stockage d'un conteneur est **éphémère** : si vous supprimez le conteneur d'une base de données PostgreSQL, toutes les données inscrites sont perdues avec lui.

Pour persister l'information, on utilise les **Volumes** Docker :

```
[ Conteneur ] ──(Écrit dans /var/lib/postgresql/data)──► [ Volume Docker sur l'Hôte ]
(Peut être détruit sans risque)                           (Les données restent intactes)
```

```bash
# Créer un conteneur de base de données avec un volume persistant
docker run -d \
  --name ma-db \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16
```

Même si vous détruisez le conteneur `ma-db` avec `docker rm -f ma-db`, vos tables existeront toujours dans le volume `pgdata`.

---

## 7. Orchestration Locale : Docker Compose

Démarrer manuellement 3 conteneurs avec de longues commandes (`docker run -p ... -e ... -v ...`) devient rapidement fastidieux. 

**Docker Compose** permet de décrire toute son architecture multi-conteneurs dans un unique fichier `docker-compose.yml` :

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://user:secret@db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - data-db:/var/lib/postgresql/data

volumes:
  data-db:
```

### Les deux commandes clés de Compose :

```bash
# Démarrer tous les services en arrière-plan (construit les images si nécessaire)
docker compose up -d

# Éteindre l'ensemble des services et leur réseau privé
docker compose down
```

> **Magie du réseau Compose :** Les conteneurs du même fichier `compose` se parlent automatiquement en utilisant le **nom de leur service** comme nom d'hôte DNS (ici, `web` contacte directement `db:5432`).

---

## 8. Les 4 Réflexes Clés du Débutant Docker

1. **Utilisez `.dockerignore` :** Tout comme `.gitignore`, excluez `node_modules`, `.git` et les fichiers locaux temporaires pour éviter d'alourdir inutilement vos images.
2. **Préférez les tags précis à `latest` :** Utilisez `node:20.11-alpine` plutôt que `node:latest` afin de garantir que votre environnement reste strictement reproductible dans le temps.
3. **Un conteneur = une seule responsabilité :** Ne mettez pas votre serveur web, votre base de données et votre serveur Redis dans le même conteneur. Découpez-les en services distincts reliés par Docker Compose.
4. **N'écrivez jamais de données critiques sans volume :** Tout fichier qui doit survivre à un redémarrage doit résider dans un volume dédié.