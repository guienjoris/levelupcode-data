# Docker Avancé — Niveau Intermédiaire : Optimisation, Réseaux & Bonnes Pratiques de Production

Après avoir assimilé le cycle de base (Dockerfile, images, conteneurs, volumes simples), l'étape intermédiaire consiste à concevoir des conteneurs **légers**, **sécurisés**, **observables** et **interconnectés** de manière robuste.

---

## 1. Multi-Stage Builds : Réduire Drastiquement le Poids des Images

Dans une application compilée ou nécessitant une phase de build (TypeScript, React, Go, Rust), les outils de développement (compilateur, SDK, `devDependencies`) ne doivent **jamais** atterrir dans l'image finale de production.

Le **Multi-stage build** permet d'utiliser plusieurs instructions `FROM` successives dans un même Dockerfile. Chaque étape hérite d'une image propre, et l'on peut copier sélectivement des artefacts d'une étape vers la suivante.

```
Étape 1 : Builder (Node.js complet + TS + npm install)
┌────────────────────────────────────────────────────────┐
│ Compilateur, linters, devDependencies, code source     │ ──► Produit le dossier /dist
└────────────────────────────────────────────────────────┘
                               │
            docker copie uniquement /dist et prod_modules
                               ▼
Étape 2 : Runtime (Image minimale Alpine / Distroless)
┌────────────────────────────────────────────────────────┐
│ Node runtime léger + /dist + production dependencies    │ ──► Image finale : ~80 Mo (au lieu de 1.2 Go)
└────────────────────────────────────────────────────────┘
```

### Exemple concret (Application Node.js / TypeScript)

```dockerfile
# ----------------------------------------
# Étape 1 : Phase de compilation (Builder)
# ----------------------------------------
FROM node:20-alpine AS builder

WORKDIR /usr/src/app

COPY package*.json tsconfig.json ./
RUN npm ci

COPY src ./src
RUN npm run build

# ----------------------------------------
# Étape 2 : Phase d'exécution (Runtime)
# ----------------------------------------
FROM node:20-alpine AS runner

WORKDIR /usr/src/app
ENV NODE_ENV=production

# Installation stricte des dépendances de production uniquement
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force

# Copie de l'artefact compilé depuis l'étape 'builder'
COPY --from=builder /usr/src/app/dist ./dist

# Sécurité : exécution avec un utilisateur non-root
USER node

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

---

## 2. Typologie et Gestion des Réseaux Docker

Docker propose plusieurs pilotes de réseau (*drivers*) répondant à des contraintes d'isolation et de performance distinctes :

```
                  ┌────────────────────────────────────────┐
                  │             Docker Drivers             │
                  └───────────────────┬────────────────────┘
          ┌───────────────────────────┼───────────────────────────┐
          ▼                           ▼                           ▼
      [ bridge ]                   [ host ]                    [ none ]
Réseau virtuel isolé         Partage direct de la         Isolation totale
(défaut sur conteneur)       pile réseau de l'hôte        (aucune interface réseau)
```

| Pilote (*Driver*) | Fonctionnement | Cas d'usage principal |
| :--- | :--- | :--- |
| **`bridge` (défaut)** | Crée un commutateur virtuel privé interne sur l'hôte. Les conteneurs communiquent par IP privée ou nom DNS (sur les réseaux user-defined). | Architecture classique multi-conteneurs sur une seule machine. |
| **`host`** | Supprime l'isolation réseau : le conteneur écoute directement sur les ports de la machine hôte sans passer par du NAT. | Débits réseau extrêmes, élimination du surcoût de translation d'adresses. |
| **`overlay`** | Relie plusieurs démons Docker situés sur des machines physiques distinctes. | Clusters Swarm ou Kubernetes (communication multi-nœuds). |
| **`none`** | Désactive toute pile réseau sur le conteneur (seule l'interface loopback `127.0.0.1` subsiste). | Tâches de chiffrement strictes, batchs isolés sans accès web. |

### Les Réseaux Personnalisés (*User-Defined Bridge*)

Sur le réseau bridge par défaut de Docker (`bridge`), la résolution de nom par DNS interne **ne fonctionne pas** (il faut s'appeler par adresses IP). Créer un réseau personnalisé active la résolution DNS automatique :

```bash
# 1. Créer un réseau bridge privé
docker network create --driver bridge app-network

# 2. Rattacher des conteneurs à ce réseau
docker run -d --name cache-redis --network app-network redis:7-alpine
docker run -d --name backend-api --network app-network mon-api:latest

# 'backend-api' peut désormais résoudre directement l'hôte 'cache-redis' par son nom !
```

---

## 3. Stockage Avancé : Volumes Nommés vs Bind Mounts vs tmpfs

Tous les points de montage ne se valent pas :

```
             ┌──────────────────────────────────────────────────┐
             │                Types de Montages                 │
             └────────────────────────┬─────────────────────────┘
        ┌─────────────────────────────┼─────────────────────────────┐
        ▼                             ▼                             ▼
 [ Named Volume ]              [ Bind Mount ]                   [ tmpfs ]
Géré par Docker dans        Chemin direct pointant sur      Stocké en RAM uniquement
/var/lib/docker/volumes     un dossier de l'hôte            (volatile, ultra rapide)
```

1. **Volumes nommés (`docker volume create`) :**
   - L'emplacement physique est géré par Docker.
   - Performances optimales sous Linux/macOS.
   - Idéal pour les bases de données et la persistance entre les redéploiements.
2. **Bind Mounts (`-v /mon/code/local:/app`) :**
   - Relie directement un dossier de la machine de développement au conteneur.
   - Indispensable pour le rechargement à chaud (*Hot-Reloading*) en environnement de développement local.
3. **tmpfs Mounts (`--tmpfs /app/cache`) :**
   - Données écrites uniquement dans la mémoire vive de l'hôte.
   - Pratique pour les données temporaires sensibles (clés de session, jetons) qu'on ne veut jamais voir toucher le disque dur.

---

## 4. Observabilité & Résilience : Les Healthchecks

Par défaut, Docker considère qu'un conteneur est en bonne santé (`healthy`) tant que son processus principal (`PID 1`) ne s'arrête pas. Pourtant, une API Node.js peut être vivante mais bloquée dans une boucle infinie ou incapable de joindre sa base de données.

Le **HEALTHCHECK** permet à Docker de vérifier périodiquement l'état réel de l'application :

```dockerfile
# Vérification toutes les 30s que l'endpoint /health répond un code HTTP 200
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
```

* **`--interval`** : Temps d'attente entre deux vérifications.
* **`--timeout`** : Temps limite alloué à la commande avant de considérer la tentative comme un échec.
* **`--start-period`** : Période de grâce au démarrage (laisse le temps à l'application de s'initialiser sans déclarer d'échec prématuré).
* **`--retries`** : Nombre d'échecs consécutifs nécessaires pour marquer le conteneur `unhealthy`.

Dans Docker Compose, cela permet d'attendre qu'un service soit réellement opérationnel avant d'en démarrer un autre :

```yaml
services:
  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    build: .
    depends_on:
      database:
        condition: service_healthy # Démarre l'API seulement après la confirmation de santé de la DB !
```

---

## 5. Durcissement et Sécurité en Conteneur

Un conteneur qui s'exécute avec les privilèges par défaut présente un risque si une faille de type *Container Escape* survient.

### Les 4 règles d'or de la sécurité :

1. **Abandonner les privilèges root (`USER non-root`) :**
   Par défaut, le processus interne du conteneur tourne sous `root`. En cas de compromission, l'attaquant a les droits root sur le conteneur. Créez toujours un groupe et un utilisateur dédié :
   ```dockerfile
   RUN addgroup -S appgroup && adduser -S appuser -G appgroup
   USER appuser
   ```
2. **Système de fichiers en lecture seule (`--read-only`) :**
   Empêche un pirate d'écrire des scripts malveillants ou de modifier les binaires existants :
   ```bash
   docker run --read-only --tmpfs /tmp mon-app:prod
   ```
3. **Supprimer les capacités superutilisateur inutiles (`CapDrop`) :**
   Les conteneurs Linux disposent d'un ensemble de capacités du noyau (*Linux capabilities*). En production, retirez-les toutes et ne réactivez que le strict nécessaire :
   ```bash
   docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE mon-app:prod
   ```
4. **Utiliser des images de base minimales (*Distroless* ou Alpine) :**
   Moins l'image contient de packages (pas de shell `bash`, pas de `curl`, pas de gestionnaire de paquets), plus la surface d'attaque et les vulnérabilités CVE sont réduites.

---

## 6. Synthèse des Bonnes Pratiques Intermédiaires

```
┌─────────────────────────┬────────────────────────────────────────────────────────┐
│ Domaine                 │ Bonne Pratique Recommandée                             │
├─────────────────────────┼────────────────────────────────────────────────────────┤
│ Taille d'image          │ Multi-stage build + images Alpine ou Google Distroless │
│ Dépendances             │ `npm ci --omit=dev` au lieu de `npm install`           │
│ Réseau                  │ Toujours créer un réseau bridge défini par l'utilisateur│
│ Observabilité           │ Healthcheck configuré pour détecter les blocages I/O   │
│ Sécurité                │ Définir une directive `USER` non-root explicite        │
│ Cache de build          │ Ordonner du moins fréquent (dépendances) au plus variable│
└─────────────────────────┴────────────────────────────────────────────────────────┘
```