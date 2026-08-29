---
schema_version: 1
uid: 01M02EX5AXEGMZ9GRYT5W9E5XF
titre: Docker
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - conteneurisation
  - docker
  - devops
resume: "Cours complet sur Docker : conteneurs, images OCI, Dockerfile, BuildKit/Buildx, Compose, réseaux, volumes, sécurité, rootless, supply chain et exploitation en production."
niveau: intermediaire
prerequis:
  - "[[GNULinux]]"
  - "[[Les namespaces Linux]]"
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-01-27
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Docker

> [!abstract] Objectif
> Comprendre ce qu'est réellement un conteneur (namespaces, cgroups, couches) et maîtriser Docker Engine 29 : images et Dockerfile, BuildKit et builds multi-étapes, volumes, réseaux, limites de ressources, sécurité et mode rootless, secrets, Compose v5, supply chain (SBOM, provenance, signature), diagnostic et pratiques de production.

Voir aussi : [[Les namespaces Linux]], [[proc]], [[Sécurité avancée sous Linux]], [[Initialisation système et des services]], [[git]].

## Objectifs

À la fin de ce cours, nous devons être capables de :

- expliquer précisément ce qu'est un conteneur et ce que Docker ajoute aux mécanismes du noyau Linux ;
- distinguer image, conteneur, couche, registry, repository et tag ;
- utiliser correctement les principales commandes de Docker Engine ;
- écrire un `Dockerfile` maintenable et sûr ;
- comprendre BuildKit, Buildx, le cache et les builds multi-plateformes ;
- persister les données avec des volumes ou des bind mounts ;
- concevoir des réseaux Docker adaptés aux besoins d'une application ;
- utiliser Docker Compose pour une application multi-conteneurs ;
- limiter CPU et mémoire ;
- appliquer le principe du moindre privilège ;
- utiliser le mode rootless lorsque cela est pertinent ;
- gérer les secrets de build et d'exécution sans les intégrer aux images ;
- produire des SBOM et attestations de provenance ;
- diagnostiquer les problèmes de build, réseau, stockage et démarrage ;
- savoir dans quels cas Docker Compose suffit et dans quels cas un orchestrateur devient nécessaire.

> [!note] Versions de référence
> Au 29 août 2026, la branche stable de Docker Engine est la série **29**, avec Docker Engine **29.7.2** publié le 5 août 2026. Cette version embarque notamment BuildKit 0.32.2. Docker Compose **v5.5.0** a été publié le 17 août 2026. Les concepts du cours restent volontairement plus généraux que ces numéros de version.

# Sommaire

1. Qu'est-ce qu'un conteneur ?
2. Architecture Docker
3. Installation
4. Images, conteneurs, repositories, tags et digests
5. Premières commandes
6. Cycle de vie et PID 1
7. Dockerfile : construire une image
8. Couches et système de fichiers
9. `.dockerignore`
10. BuildKit et Buildx
11. Multi-stage builds
12. Optimiser un Dockerfile
13. Volumes et persistance
14. UID, GID et permissions
15. Réseaux Docker
16. DNS et résolution de noms
17. Limites de ressources
18. Healthchecks
19. Sécurité : modèle de menace
20. Sécurité d'exécution
21. Rootless Docker
22. User namespaces
23. Secrets
24. Docker Compose
25. Compose en production : limites
26. Build, registry et push
27. Multi-architecture
28. Reproductibilité
29. Supply chain : SBOM et provenance
30. Signature et confiance
31. Logs et observabilité
32. Diagnostic
33. Docker daemon et configuration
34. Remote Docker et contextes
35. Conteneurs privilégiés et périphériques
36. Docker et le pare-feu
37. Sauvegarde et restauration
38. CI/CD
39. Docker Compose : exemple de projet complet
40. Erreurs fréquentes
41. Docker vs Podman vs Kubernetes
42. Swarm
43. Bonnes pratiques de production
44-57. Travaux pratiques 1 à 14 (cycle de vie, images, cache, multi-stage, volumes, réseaux, Compose, durcissement, rootless, multi-architecture, SBOM, supply chain, diagnostic, projet final)
58. Commandes de référence
59. Méthode de mise à jour d'une application Docker
60. Ce qu'il faut retenir

# 1. Qu'est-ce qu'un conteneur ?

Un conteneur n'est pas une petite machine virtuelle.

Un conteneur est avant tout **un ou plusieurs processus ordinaires de l'hôte auxquels le noyau applique un ensemble de mécanismes d'isolation et de contrôle des ressources**.

Sous Linux, les briques principales sont :

- les **namespaces** pour isoler la vue des ressources ;
- les **cgroups** pour contrôler et comptabiliser les ressources ;
- les **capabilities** pour découper les privilèges de `root` ;
- les mécanismes LSM comme AppArmor ou SELinux ;
- seccomp pour filtrer les appels système ;
- un système de fichiers en couches, généralement basé sur OverlayFS ;
- des interfaces réseau virtuelles, bridges, routes et règles de filtrage/NAT.

Voir aussi [[Les namespaces Linux]].

## 1.1 Conteneur et machine virtuelle

Une machine virtuelle émule ou virtualise une machine complète :

```text
Application
Bibliothèques
OS invité
Noyau invité
Hyperviseur
OS / matériel hôte
```

Un conteneur Linux partage le noyau de l'hôte :

```text
Application A       Application B
Bibliothèques A     Bibliothèques B
        \             /
         namespaces
         cgroups
         LSM/seccomp
             |
        noyau Linux hôte
             |
          matériel
```

Conséquences :

- un conteneur est généralement beaucoup plus rapide à créer qu'une VM ;
- une image est généralement plus petite qu'une image de VM ;
- plusieurs conteneurs partagent le même noyau ;
- un conteneur Linux ne transporte donc pas son propre noyau Linux ;
- l'isolation n'est pas identique à celle d'une VM.

## 1.2 Docker n'est pas « le conteneur »

Docker fournit une plateforme et une expérience utilisateur autour des conteneurs :

```text
Docker CLI
   |
Docker Engine API
   |
dockerd
   |
containerd
   |
containerd-shim
   |
runc / runtime OCI
   |
noyau Linux
```

Cette vue est volontairement simplifiée, mais elle permet de comprendre que Docker s'appuie sur des standards et composants plus bas niveau.

## 1.3 Standards OCI

L'**Open Container Initiative (OCI)** standardise notamment :

- le format des images ;
- le format d'exécution d'un conteneur ;
- la distribution des images.

Docker n'est donc pas le seul outil capable de lire une image OCI.

# 2. Architecture Docker

## 2.1 Le client et le daemon

La commande :

```bash
docker ps
```

ne liste pas elle-même les processus.

Le client Docker contacte l'API Docker Engine, généralement via :

```text
/var/run/docker.sock
```

Le daemon réalise ensuite l'opération demandée.

Nous pouvons afficher le contexte courant :

```bash
docker context show
```

et lister les contextes disponibles :

```bash
docker context ls
```

## 2.2 Le socket Docker est extrêmement privilégié

Un utilisateur capable d'envoyer librement des commandes au socket Docker rootful peut généralement obtenir un niveau de contrôle équivalent à `root` sur l'hôte.

Par exemple, autoriser arbitrairement :

```bash
docker run -v /:/host ...
```

permet d'exposer le système de fichiers de l'hôte au conteneur.

Il faut donc considérer l'accès à :

```text
/var/run/docker.sock
```

comme un accès privilégié.

> [!warning]
> Ajouter un utilisateur au groupe `docker` n'est pas une simple commodité équivalente à un groupe applicatif classique. Avec le daemon rootful, cela donne en pratique des possibilités proches de `root`.

# 3. Installation

Deux stratégies sont courantes sur Debian/Ubuntu :

1. utiliser les paquets de la distribution ;
2. utiliser le dépôt officiel Docker.

## 3.1 Paquet de la distribution

Sur Debian/Ubuntu :

```bash
sudo apt update
sudo apt install docker.io
```

Avantages :

- intégration avec la distribution ;
- cycle de mises à jour cohérent avec le système.

Limite :

- la version peut être plus ancienne que la version officielle Docker.

## 3.2 Dépôt officiel Docker

Pour un serveur qui doit suivre les versions Docker officielles, préférer la procédure maintenue par Docker plutôt que de recopier une suite figée de commandes dans un cours.

La documentation actuelle est :

```text
https://docs.docker.com/engine/install/
```

Les paquets principaux sont généralement :

```text
docker-ce
docker-ce-cli
containerd.io
docker-buildx-plugin
docker-compose-plugin
```

## 3.3 Vérification

```bash
docker version
docker info
```

Puis :

```bash
docker run --rm hello-world
```

`docker version` distingue notamment les versions du client et du serveur.

## 3.4 Docker Desktop

Docker Desktop fournit une expérience intégrée sur Windows, macOS et Linux.

Sur Windows, les conteneurs Linux utilisent généralement WSL 2.

Docker Desktop n'est pas nécessaire pour faire fonctionner Docker Engine sur un serveur Linux.

# 4. Images, conteneurs, repositories, tags et digests

## 4.1 Image

Une image est un artefact immuable, composé de couches et de métadonnées.

Exemples de références :

```text
nginx
nginx:alpine
python:3.14-slim
registry.example.org/team/app:1.4.2
```

## 4.2 Conteneur

Un conteneur est une **instance d'exécution** créée à partir d'une image.

Le conteneur possède notamment :

- une configuration d'exécution ;
- des namespaces ;
- une couche écrivable ;
- éventuellement des volumes ;
- éventuellement une configuration réseau ;
- un processus principal.

## 4.3 Repository et registry

Un **registry** héberge des images, par exemple :

- Docker Hub ;
- GitHub Container Registry ;
- GitLab Container Registry ;
- Harbor ;
- un registry privé OCI.

Un **repository** regroupe les versions d'une image :

```text
example/app:1.0
example/app:1.1
example/app:latest
```

## 4.4 Tag

Un tag est un nom mutable :

```text
python:3.14
```

peut désigner une image différente après la publication d'une nouvelle 3.14.x.

## 4.5 Digest

Un digest identifie un contenu par son hash :

```text
python@sha256:...
```

Pour un déploiement hautement reproductible, nous pouvons pinner par digest.

Exemple conceptuel :

```dockerfile
FROM python:3.14-slim@sha256:<digest>
```

Cela augmente la reproductibilité mais impose une stratégie de mise à jour du digest pour recevoir les correctifs de sécurité.

# 5. Premières commandes

## 5.1 Télécharger une image

```bash
docker pull nginx:alpine
```

## 5.2 Lister les images

```bash
docker image ls
```

## 5.3 Créer et lancer un conteneur

```bash
docker run nginx:alpine
```

`docker run` combine conceptuellement :

```bash
docker create ...
docker start ...
```

## 5.4 Exécution éphémère

```bash
docker run --rm alpine:latest echo 'Bonjour'
```

`--rm` demande la suppression du conteneur à la fin de l'exécution.

## 5.5 Mode interactif

```bash
docker run --rm -it ubuntu:24.04 bash
```

- `-i` garde stdin ouvert ;
- `-t` alloue un pseudo-terminal.

## 5.6 Mode détaché

```bash
docker run -d --name web nginx:alpine
```

## 5.7 Lister les conteneurs

```bash
docker ps
docker ps -a
```

## 5.8 Logs

```bash
docker logs web
docker logs -f --tail 100 web
```

## 5.9 Exécuter une commande dans un conteneur existant

```bash
docker exec -it web sh
```

`exec` ne doit pas être confondu avec la création d'un nouveau conteneur.

## 5.10 Inspecter

```bash
docker inspect web
```

Avec format Go template :

```bash
docker inspect --format '{{.State.Status}}' web
```

## 5.11 Arrêter et supprimer

```bash
docker stop web
docker rm web
```

Forcer :

```bash
docker rm -f web
```

## 5.12 Nettoyage

Éviter les commandes historiques comme :

```bash
# À éviter
docker rm $(docker ps -a -q)
docker rmi $(docker images -a -q)
```

Docker fournit des commandes dédiées :

```bash
docker container prune
docker image prune
docker network prune
docker volume prune
docker system prune
```

Attention :

```bash
docker system prune -a --volumes
```

peut supprimer beaucoup de données. Toujours comprendre son effet avant de l'exécuter.

# 6. Cycle de vie et PID 1

## 6.1 Le processus principal

Un conteneur reste en vie tant que son processus principal reste en vie.

Exemple :

```bash
docker run --rm alpine true
```

Le conteneur s'arrête immédiatement parce que `true` se termine immédiatement.

## 6.2 Signaux

Lors de :

```bash
docker stop app
```

Docker envoie d'abord le signal d'arrêt configuré, généralement `SIGTERM`, puis attend un délai avant de forcer avec `SIGKILL` si nécessaire.

L'application doit donc :

- recevoir correctement les signaux ;
- terminer proprement les requêtes en cours ;
- fermer les connexions ;
- sortir avant le timeout.

## 6.3 Le problème PID 1

PID 1 a un comportement particulier sous Linux, notamment pour la gestion des processus orphelins et des signaux.

Pour les applications qui créent de nombreux processus enfants, nous pouvons utiliser :

```bash
docker run --init ...
```

Docker ajoute alors un petit init pour gérer le reaping des zombies.

# 7. Dockerfile : construire une image

Un `Dockerfile` décrit comment produire une image.

## 7.1 Exemple minimal

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.14-slim

WORKDIR /app
COPY app.py .

CMD ["python", "app.py"]
```

Construction :

```bash
docker build -t exemple:1.0 .
```

Exécution :

```bash
docker run --rm exemple:1.0
```

## 7.2 `FROM`

```dockerfile
FROM debian:13-slim
```

Une image de base doit être :

- maintenue ;
- adaptée au runtime ;
- suffisamment minimale ;
- mise à jour régulièrement.

« Plus petite » ne signifie pas automatiquement « plus sûre » : la maintenabilité et la visibilité des composants comptent aussi.

## 7.3 `RUN`

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*
```

`apt-get update` et `apt-get install` doivent être dans la même instruction pour éviter de réutiliser une ancienne liste de paquets depuis le cache.

## 7.4 `COPY`

```dockerfile
COPY pyproject.toml uv.lock ./
COPY src/ ./src/
```

Préférer `COPY` à `ADD` lorsque les fonctions additionnelles de `ADD` ne sont pas nécessaires.

## 7.5 `WORKDIR`

```dockerfile
WORKDIR /app
```

Évite les suites fragiles de :

```dockerfile
RUN cd /app && ...
```

## 7.6 `ENV`

```dockerfile
ENV APP_ENV=production
```

Une variable `ENV` est persistante dans l'image et visible dans sa configuration.

Ne pas y stocker de secret.

## 7.7 `ARG`

```dockerfile
ARG VERSION
RUN echo "$VERSION"
```

`ARG` sert au build, mais **ce n'est pas un mécanisme sûr pour fournir un secret**. Une valeur peut être visible dans l'historique, la provenance ou les métadonnées selon le build.

## 7.8 `EXPOSE`

```dockerfile
EXPOSE 8000
```

`EXPOSE` documente un port attendu par l'application.

Il ne publie pas à lui seul le port sur l'hôte.

Publication réelle :

```bash
docker run -p 127.0.0.1:8000:8000 app
```

## 7.9 `ENTRYPOINT` et `CMD`

Exemple :

```dockerfile
ENTRYPOINT ["python", "-m", "myapp"]
CMD ["--port", "8000"]
```

Commande finale :

```text
python -m myapp --port 8000
```

L'utilisateur peut remplacer `CMD` :

```bash
docker run app --port 9000
```

## 7.10 Forme exec et forme shell

Préférer généralement :

```dockerfile
CMD ["python", "app.py"]
```

à :

```dockerfile
CMD python app.py
```

La forme exec évite une couche shell implicite et facilite la propagation correcte des signaux.

# 8. Couches et système de fichiers

## 8.1 Couches immuables

Chaque étape de build contribue à un graphe de contenu réutilisable.

Les images sont adressées par contenu.

## 8.2 Couche écrivable du conteneur

Lorsqu'un conteneur modifie son système de fichiers, les modifications sont stockées dans une couche écrivable spécifique au conteneur.

Cette couche n'est pas la bonne place pour les données persistantes importantes.

## 8.3 OverlayFS

Sous Linux, `overlay2`/OverlayFS est la solution habituelle pour le stockage en couches.

Les longues comparaisons historiques avec AUFS et Device Mapper ont aujourd'hui surtout un intérêt historique.

Pour connaître le driver utilisé :

```bash
docker info --format '{{.Driver}}'
```

## 8.4 Pourquoi supprimer un secret plus tard ne suffit pas

Mauvais :

```dockerfile
COPY secret.txt /tmp/secret.txt
RUN utiliser-secret /tmp/secret.txt
RUN rm /tmp/secret.txt
```

Le secret peut toujours être présent dans une couche précédente.

Il faut utiliser les **secrets BuildKit**.

# 9. `.dockerignore`

Le contexte de build est ce que le client rend disponible au builder.

Sans `.dockerignore`, nous risquons d'envoyer :

- `.git/` ;
- environnements virtuels ;
- caches ;
- dumps ;
- secrets ;
- artefacts volumineux.

Exemple :

```text
.git
.venv
__pycache__
*.pyc
.env
.env.*
node_modules
coverage
build
dist
```

Attention : `.dockerignore` réduit le contexte mais ne remplace pas une politique de gestion des secrets.

# 10. BuildKit et Buildx

BuildKit est aujourd'hui le backend de build par défaut pour les images Linux avec Docker Engine et Docker Desktop.

Il n'est plus nécessaire d'activer manuellement :

```bash
export DOCKER_BUILDKIT=1
```

comme dans les anciennes versions.

## 10.1 Architecture

```text
Dockerfile
   |
Buildx
   |
BuildKit
   |
LLB / graphe de build
   |
cache + exécution parallèle
   |
image / registry / OCI / cache
```

## 10.2 Inspecter les builders

```bash
docker buildx ls
```

Créer un builder dédié :

```bash
docker buildx create \
  --name builder \
  --driver docker-container \
  --use \
  --bootstrap
```

## 10.3 Cache BuildKit

BuildKit peut monter un cache sans l'intégrer à l'image finale.

Exemple Python avec pip :

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.14-slim

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install requests
```

Exemple `apt` :

```dockerfile
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends gcc
```

## 10.4 Secrets de build

Fichier local :

```bash
docker buildx build \
  --secret id=pypi_token,src="$HOME/.config/pypi-token" \
  -t app .
```

Dockerfile :

```dockerfile
RUN --mount=type=secret,id=pypi_token \
    TOKEN="$(cat /run/secrets/pypi_token)" \
    && installer-prive --token "$TOKEN"
```

Le secret n'est pas copié dans une couche finale.

## 10.5 SSH forwarding de build

Pour accéder à une dépendance Git privée sans copier la clé privée :

```bash
docker buildx build --ssh default .
```

Dockerfile :

```dockerfile
RUN --mount=type=ssh git clone git@github.com:example/private.git
```

## 10.6 Builds multi-plateformes

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry.example.org/app:1.0 \
  --push .
```

Cela produit généralement un index d'images avec plusieurs manifests.

## 10.7 Variables de plateforme

```dockerfile
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
```

Exemple :

```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.26 AS build
ARG TARGETOS TARGETARCH
RUN GOOS=$TARGETOS GOARCH=$TARGETARCH go build -o /out/app ./cmd/app
```

# 11. Multi-stage builds

Un build multi-stage permet de compiler dans une image riche puis d'exécuter le résultat dans une image plus petite.

Exemple Go :

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.26 AS build
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -trimpath -o /out/server ./cmd/server

FROM gcr.io/distroless/static-debian13:nonroot
COPY --from=build /out/server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

Bénéfices :

- pas de compilateur dans l'image finale ;
- moins de dépendances runtime ;
- image plus petite ;
- surface d'attaque réduite.

# 12. Optimiser un Dockerfile

## 12.1 Ordonner selon la stabilité

Pour Python :

```dockerfile
COPY pyproject.toml uv.lock ./
RUN installer-les-dependances
COPY src/ ./src/
```

Le code change souvent, les dépendances moins souvent.

Cette organisation maximise la réutilisation du cache.

## 12.2 Ne pas mettre à jour aléatoirement au runtime

Éviter que le conteneur fasse au démarrage :

```bash
apt upgrade
pip install -U ...
```

Le build doit produire l'artefact qui sera exécuté.

## 12.3 Un processus applicatif principal clair

Éviter les conteneurs monolithiques contenant :

```text
nginx + postgres + redis + application + cron + sshd
```

sauf besoin réellement justifié.

Compose ou un orchestrateur permet généralement de séparer ces responsabilités.

# 13. Volumes et persistance

Docker distingue plusieurs mécanismes.

## 13.1 Couche écrivable

Éphémère et liée au conteneur.

À éviter pour les données que nous devons conserver.

## 13.2 Volume nommé

Création :

```bash
docker volume create pgdata
```

Utilisation :

```bash
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  postgres:18
```

Inspecter :

```bash
docker volume inspect pgdata
```

## 13.3 Syntaxe `--mount`

Plus explicite :

```bash
docker run --rm \
  --mount type=volume,source=pgdata,target=/data \
  alpine ls /data
```

## 13.4 Bind mount

```bash
docker run --rm \
  --mount type=bind,source="$PWD",target=/workspace \
  alpine ls /workspace
```

Le bind mount expose directement un chemin de l'hôte.

Conséquences :

- couplage à l'arborescence hôte ;
- problèmes potentiels d'UID/GID ;
- possibilité de modifier des fichiers hôte ;
- portabilité plus faible.

## 13.5 Lecture seule

```bash
docker run --rm \
  --mount type=bind,source="$PWD/config",target=/config,readonly \
  app
```

## 13.6 `tmpfs`

Pour des données temporaires en mémoire :

```bash
docker run --rm \
  --tmpfs /tmp:rw,noexec,nosuid,size=64m \
  app
```

## 13.7 Sauvegarder un volume

Docker ne remplace pas une stratégie de sauvegarde.

Exemple pédagogique simple :

```bash
docker run --rm \
  -v pgdata:/source:ro \
  -v "$PWD/backups":/backup \
  alpine \
  tar czf /backup/pgdata.tar.gz -C /source .
```

Pour une base de données, une sauvegarde applicative cohérente (`pg_dump`, etc.) est souvent préférable à une copie brute de fichiers actifs.

# 14. UID, GID et permissions

Un UID dans le conteneur est interprété par le noyau de l'hôte.

Avec un bind mount, cela devient visible immédiatement.

Exemple Dockerfile :

```dockerfile
RUN groupadd --system --gid 10001 app \
    && useradd --system --uid 10001 --gid app --home /app app

USER app
```

Éviter les solutions globales du type :

```bash
chmod -R 777 ...
```

qui masquent le problème au lieu de le résoudre.

# 15. Réseaux Docker

## 15.1 Réseau bridge par défaut

Docker crée généralement un bridge par défaut nommé :

```text
bridge
```

mais, pour une application, il est préférable de créer un **user-defined bridge**.

## 15.2 Créer un réseau

```bash
docker network create appnet
```

Puis :

```bash
docker run -d --name db --network appnet postgres:18
docker run -d --name api --network appnet app
```

Sur un réseau utilisateur, le DNS interne permet de joindre :

```text
db
```

par son nom.

## 15.3 Publier un port

```bash
docker run -d \
  -p 127.0.0.1:8080:80 \
  nginx:alpine
```

Ici :

```text
127.0.0.1:8080 -> conteneur:80
```

Lier à `127.0.0.1` évite une exposition réseau externe involontaire.

## 15.4 `EXPOSE` n'est pas `-p`

`EXPOSE 80` documente.

`-p 8080:80` publie réellement.

## 15.5 Mode host

```bash
docker run --network host ...
```

Le conteneur partage la pile réseau de l'hôte.

Cela réduit l'isolation et doit être utilisé pour un besoin précis.

## 15.6 Mode none

```bash
docker run --network none ...
```

Le conteneur ne dispose pas d'une connectivité réseau normale.

## 15.7 Macvlan et ipvlan

Ils permettent d'intégrer plus directement des conteneurs à un réseau L2/L3 spécifique.

Ils sont utiles dans certains environnements réseau ou legacy, mais sont plus complexes qu'un bridge.

Ne pas inventer un sous-réseau, une gateway ou une IP failover à partir d'un exemple : ces valeurs dépendent entièrement de l'infrastructure réelle.

# 16. DNS et résolution de noms

Sur un réseau utilisateur Docker, les noms de services/conteneurs peuvent être résolus par le DNS embarqué de Docker.

Exemple :

```bash
docker exec api getent hosts db
```

Dans une application Compose, utiliser le **nom du service**, pas son adresse IP éphémère.

Mauvais :

```text
DB_HOST=172.18.0.4
```

Bon :

```text
DB_HOST=db
```

# 17. Limites de ressources

Sans limites, un conteneur peut consommer une part importante des ressources de l'hôte.

## 17.1 Mémoire

```bash
docker run --memory=512m --memory-swap=512m app
```

## 17.2 CPU

```bash
docker run --cpus=1.5 app
```

## 17.3 PIDs

```bash
docker run --pids-limit=200 app
```

## 17.4 Observer

```bash
docker stats
```

Les mécanismes sous-jacents reposent sur les cgroups du noyau.

# 18. Healthchecks

Un processus vivant n'est pas nécessairement une application saine.

Dockerfile :

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD ["/app/healthcheck"]
```

Inspecter :

```bash
docker inspect --format '{{json .State.Health}}' app
```

Un healthcheck doit être :

- rapide ;
- local ;
- stable ;
- représentatif de la capacité du service à fonctionner.

Éviter un healthcheck qui déclenche une opération métier coûteuse.

# 19. Sécurité : modèle de menace

Docker apporte de l'isolation, mais un conteneur n'est pas une frontière de sécurité parfaite.

Nous devons protéger :

- l'hôte ;
- le daemon Docker ;
- les images ;
- le registry ;
- la chaîne de build ;
- les secrets ;
- le réseau ;
- les données persistantes ;
- la CI/CD.

# 20. Sécurité d'exécution

## 20.1 Ne pas exécuter en root sans nécessité

Dockerfile :

```dockerfile
RUN useradd --system --uid 10001 app
USER 10001
```

Le `root` du conteneur n'est pas automatiquement le `root` de l'hôte, mais réduire les privilèges réduit l'impact d'une compromission.

## 20.2 Capabilities

Au lieu de `--privileged`, retirer les capacités inutiles :

```bash
docker run \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  app
```

`--privileged` désactive une grande partie des protections habituelles et doit rester exceptionnel.

## 20.3 Système de fichiers root en lecture seule

```bash
docker run \
  --read-only \
  --tmpfs /tmp \
  app
```

## 20.4 `no-new-privileges`

```bash
docker run \
  --security-opt no-new-privileges=true \
  app
```

## 20.5 Seccomp

Docker applique un profil seccomp par défaut sur Linux.

Éviter :

```bash
--security-opt seccomp=unconfined
```

sauf diagnostic ou besoin parfaitement compris.

## 20.6 AppArmor / SELinux

Selon la distribution, Docker peut s'intégrer à AppArmor ou SELinux.

Ne pas contourner systématiquement les politiques de sécurité lorsque l'application rencontre un problème : diagnostiquer la cause.

# 21. Rootless Docker

Le mode rootless permet d'exécuter le daemon et les conteneurs sans daemon root système.

Objectif principal : réduire les conséquences d'une compromission du daemon ou d'un conteneur.

Vérifier :

```bash
docker info | grep -i rootless
```

Le rootless s'appuie notamment sur :

- user namespaces ;
- subordinate UID/GID ;
- outils réseau user-space selon la configuration.

Il peut avoir des différences ou limitations pour :

- ports privilégiés ;
- certaines fonctions réseau ;
- cgroups selon la distribution ;
- accès aux périphériques ;
- workloads fortement privilégiés.

Le rootless est excellent lorsque le besoin applicatif est compatible, mais ne transforme pas un conteneur non fiable en sandbox parfaite.

# 22. User namespaces

Une autre approche est le remapping des utilisateurs du daemon rootful avec `userns-remap`.

Le principe :

```text
root dans le conteneur
       |
       v
UID non privilégié sur l'hôte
```

Cela réduit la correspondance directe entre UID 0 du conteneur et UID 0 de l'hôte.

Rootless et `userns-remap` répondent à des modèles d'exploitation différents.

# 23. Secrets

## 23.1 Ce qui n'est pas un secret sûr

Mauvais :

```dockerfile
ENV DATABASE_PASSWORD=supersecret
```

Mauvais :

```dockerfile
ARG TOKEN=...
```

Mauvais :

```dockerfile
COPY .env /app/.env
```

## 23.2 Build secrets

Utiliser :

```dockerfile
RUN --mount=type=secret,id=token ...
```

avec :

```bash
docker build --secret id=token,src=token.txt .
```

## 23.3 Secrets au runtime

Docker Engine seul ne fournit pas exactement le même mécanisme de secrets que Swarm/Kubernetes.

Selon l'environnement, utiliser :

- un fichier monté en lecture seule ;
- un secret manager ;
- un mécanisme de plateforme ;
- Docker Swarm secrets si nous utilisons Swarm.

Éviter si possible les secrets sensibles directement dans les variables d'environnement, car ils peuvent se retrouver dans :

- diagnostics ;
- dumps ;
- outils d'inspection ;
- erreurs ;
- processus enfants.

# 24. Docker Compose

Docker Compose décrit une application multi-conteneurs dans un fichier YAML.

La commande moderne est :

```bash
docker compose
```

et non l'ancien binaire Python :

```bash
# Ancienne commande
docker-compose
```

Au 29 août 2026, Docker Compose v5.5.0 est la version stable récente.

## 24.1 Fichier `compose.yaml`

Exemple :

```yaml
services:
  web:
    build:
      context: .
    ports:
      - "127.0.0.1:8080:8000"
    environment:
      DATABASE_URL: postgresql://app@db/app
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend
      - backend

  db:
    image: postgres:18
    environment:
      POSTGRES_USER: app
      POSTGRES_DB: app
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - dbdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 5s
      timeout: 3s
      retries: 10
    networks:
      - backend
    secrets:
      - db_password

networks:
  frontend:
  backend:
    internal: true

volumes:
  dbdata:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

> [!warning]
> Un secret Compose défini depuis un fichier local reste dépendant de la sécurité de ce fichier sur l'hôte. Compose n'est pas à lui seul un coffre-fort distribué.

## 24.2 Pas de clé `version:` obligatoire

Les fichiers Compose modernes suivent la **Compose Specification**.

Éviter les anciens exemples commençant par :

```yaml
version: "3.8"
```

La clé top-level `version` est désormais obsolète/informative et n'est pas nécessaire pour un fichier Compose moderne.

## 24.3 Commandes essentielles

```bash
docker compose config
docker compose pull
docker compose build
docker compose up
docker compose up -d
docker compose ps
docker compose logs -f
docker compose exec web sh
docker compose run --rm web commande
docker compose stop
docker compose restart
docker compose down
```

Supprimer aussi les volumes nommés :

```bash
docker compose down --volumes
```

À utiliser avec prudence.

## 24.4 `depends_on` ne remplace pas un protocole de retry

Même avec :

```yaml
depends_on:
  db:
    condition: service_healthy
```

une application de production doit savoir gérer :

- redémarrage de la base ;
- rupture réseau ;
- indisponibilité transitoire ;
- reconnexion.

L'ordre de démarrage ne garantit pas la disponibilité permanente.

## 24.5 Profiles

```yaml
services:
  adminer:
    image: adminer
    profiles: ["debug"]
```

Activation :

```bash
docker compose --profile debug up
```

## 24.6 Watch

Pour le développement, Compose peut surveiller et synchroniser les changements :

```yaml
services:
  web:
    build: .
    develop:
      watch:
        - action: sync
          path: ./src
          target: /app/src
        - action: rebuild
          path: ./pyproject.toml
```

Puis :

```bash
docker compose watch
```

## 24.7 Override et fusion

Nous pouvons combiner :

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  up
```

Toujours vérifier le résultat fusionné :

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  config
```

# 25. Compose en production : limites

Compose est très utile pour :

- développement ;
- intégration ;
- tests ;
- services mono-hôte ;
- petits déploiements maîtrisés.

Compose ne fournit pas à lui seul toutes les fonctions d'un orchestrateur distribué :

- scheduling multi-nœuds ;
- rescheduling automatique sur un autre nœud ;
- primitives avancées de rollout ;
- contrôle de cluster ;
- autoscaling distribué.

Pour ces besoins, regarder notamment :

- Kubernetes ;
- Nomad ;
- Docker Swarm selon le contexte ;
- plateformes managées.

Ne pas ajouter Kubernetes par réflexe à une application qui fonctionne très bien sur un seul hôte avec Compose.

# 26. Build, registry et push

## 26.1 Tagger

```bash
docker tag app:local registry.example.org/team/app:1.4.0
```

## 26.2 Se connecter

```bash
docker login registry.example.org
```

Éviter de mettre un mot de passe en clair dans l'historique shell.

En automatisation :

```bash
printf '%s' "$REGISTRY_TOKEN" | \
  docker login registry.example.org \
    --username "$REGISTRY_USER" \
    --password-stdin
```

## 26.3 Push

```bash
docker push registry.example.org/team/app:1.4.0
```

## 26.4 Tags immuables et digests

Une stratégie saine distingue :

- tags lisibles : `1.4.0` ;
- tags de canal : `stable` ;
- digest exact pour l'identité cryptographique du contenu.

# 27. Multi-architecture

Lister les plateformes du builder :

```bash
docker buildx inspect --bootstrap
```

Construire :

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag registry.example.org/app:1.0 \
  --push .
```

Vérifier :

```bash
docker buildx imagetools inspect registry.example.org/app:1.0
```

Attention à ne pas supposer que la simple émulation QEMU équivaut toujours à une compilation native en performances et comportement.

# 28. Reproductibilité

Un build parfaitement bit-for-bit reproductible peut être difficile, mais nous pouvons fortement améliorer la reproductibilité.

## 28.1 Pinner les dépendances

Pour l'application :

- lockfiles ;
- versions exactes ;
- hash lorsque l'écosystème le permet.

Pour l'image de base :

- tag précis ;
- éventuellement digest.

## 28.2 Ne pas dépendre de `latest`

Éviter en production :

```dockerfile
FROM python:latest
```

Préférer :

```dockerfile
FROM python:3.14-slim
```

voire un digest avec processus de mise à jour automatisé.

# 29. Supply chain : SBOM et provenance

La sécurité d'une image ne concerne pas uniquement sa configuration runtime.

Nous devons pouvoir répondre à :

- quels paquets contient-elle ?
- quelle image de base a été utilisée ?
- quel commit a produit l'artefact ?
- avec quel builder ?
- quelles vulnérabilités connues existent ?

## 29.1 Générer une SBOM avec Docker Scout

```bash
docker scout sbom --format list image:tag
```

SPDX :

```bash
docker scout sbom --format spdx image:tag > sbom.spdx.json
```

Docker Scout peut analyser :

- images ;
- archives ;
- layout OCI ;
- répertoires locaux.

## 29.2 Attestations BuildKit

```bash
docker buildx build \
  --sbom=true \
  --provenance=mode=max \
  --tag registry.example.org/app:1.0 \
  --push .
```

Les attestations de type SBOM et provenance sont attachées à l'index de l'image lorsque le stockage/driver utilisé le permet.

## 29.3 Inspecter les vulnérabilités

```bash
docker scout quickview image:tag
docker scout cves image:tag
docker scout recommendations image:tag
```

Un scanner ne prouve pas qu'une image est sûre :

- certaines vulnérabilités ne sont pas encore connues ;
- certaines CVE sont non exploitables dans le contexte ;
- la configuration et l'application comptent autant que les paquets.

# 30. Signature et confiance

Une chaîne de confiance moderne peut inclure :

- digest OCI ;
- signature d'artefact ;
- provenance ;
- SBOM ;
- politique de déploiement ;
- registry contrôlé.

Le point fondamental est de déployer un **artefact identifié et vérifiable**, pas « quelque chose qui s'appelle `latest` ».

# 31. Logs et observabilité

## 31.1 stdout / stderr

Une application conteneurisée doit généralement écrire ses logs applicatifs sur stdout/stderr.

```bash
docker logs app
```

## 31.2 Driver de logs

Inspecter :

```bash
docker info --format '{{.LoggingDriver}}'
```

Docker peut utiliser différents drivers de logs selon l'infrastructure.

## 31.3 Rotation

Sans stratégie, les logs peuvent remplir le disque.

Exemple `daemon.json` :

```json
{
  "log-driver": "local"
}
```

ou configurer les options adaptées au driver choisi.

Toute modification du daemon doit être testée selon la distribution et le contexte.

# 32. Diagnostic

## 32.1 Le conteneur s'arrête immédiatement

```bash
docker ps -a
docker logs <nom>
docker inspect <nom>
```

Puis vérifier le code de sortie :

```bash
docker inspect \
  --format '{{.State.ExitCode}} {{.State.Error}}' \
  <nom>
```

## 32.2 Le port n'est pas accessible

```bash
docker port web
docker inspect web
```

Vérifier dans le conteneur :

```bash
docker exec web ss -lntp
```

Erreur classique : l'application écoute uniquement sur :

```text
127.0.0.1
```

**dans le conteneur**, alors qu'elle doit souvent écouter sur :

```text
0.0.0.0
```

puis être protégée par le mapping réseau approprié côté hôte.

## 32.3 DNS

```bash
docker exec app getent hosts db
```

## 32.4 Disque plein

```bash
docker system df
docker system df -v
```

Puis identifier précisément :

- images inutilisées ;
- cache de build ;
- volumes ;
- logs ;
- couches de conteneurs.

## 32.5 Cache de build

```bash
docker buildx du
docker buildx prune
```

## 32.6 Pourquoi mon build n'utilise-t-il pas le cache ?

Inspecter :

- ordre des `COPY` ;
- fichiers qui changent dans le contexte ;
- arguments de build ;
- image de base ;
- mounts ;
- builder utilisé.

Utiliser un affichage explicite :

```bash
docker buildx build --progress=plain .
```

# 33. Docker daemon et configuration

Le daemon rootful est généralement configuré via :

```text
/etc/docker/daemon.json
```

Exemple très simple :

```json
{
  "live-restore": true
}
```

La configuration exacte dépend de :

- la distribution ;
- systemd ;
- containerd image store ;
- réseau ;
- proxies ;
- registry mirrors ;
- sécurité.

Toujours valider la configuration avant de redémarrer un hôte de production.

# 34. Remote Docker et contextes

Éviter d'exposer directement l'API Docker non protégée sur TCP.

Pour administrer un hôte distant, un contexte SSH est souvent plus sûr :

```bash
docker context create prod \
  --docker host=ssh://admin@docker.example.org
```

Puis :

```bash
docker --context prod ps
```

ou :

```bash
docker context use prod
```

Cela évite de configurer manuellement une API TCP publique lorsque ce n'est pas nécessaire.

# 35. Conteneurs privilégiés et périphériques

## 35.1 `--privileged`

```bash
docker run --privileged ...
```

accorde des privilèges très importants.

Ne pas l'utiliser comme solution universelle à un problème de permissions.

## 35.2 Périphérique ciblé

Si l'application a seulement besoin d'un périphérique :

```bash
docker run --device /dev/ttyUSB0 app
```

reste plus précis que `--privileged`.

# 36. Docker et le pare-feu

Docker peut créer des règles de filtrage/NAT automatiquement.

Sur un hôte qui utilise nftables/iptables ou une politique de pare-feu stricte, il faut comprendre l'interaction entre :

- Docker ;
- la chaîne `DOCKER-USER` ou mécanisme équivalent selon backend/version ;
- le firewall de la distribution ;
- les règles de forwarding ;
- le port publishing.

Ne pas conclure qu'un port est bloqué uniquement parce qu'un outil comme UFW ne l'affiche pas comme autorisé : vérifier la chaîne réseau réellement appliquée.

# 37. Sauvegarde et restauration

## 37.1 Sauvegarder la configuration

Versionner :

- Dockerfiles ;
- `compose.yaml` ;
- scripts de migration ;
- fichiers de configuration non secrets ;
- infrastructure as code.

## 37.2 Sauvegarder les données

Selon le service :

- dump logique ;
- snapshot cohérent ;
- backup volume ;
- sauvegarde applicative.

## 37.3 Une image n'est pas une sauvegarde de conteneur

`docker commit` existe mais ne doit pas être la stratégie standard de maintenance d'une application.

L'état souhaité doit être reproductible depuis :

```text
code + Dockerfile + configuration + données sauvegardées
```

# 38. CI/CD

Pipeline type :

```text
lint
  |
tests
  |
build
  |
scan
  |
SBOM / provenance
  |
push par digest
  |
deploy
  |
healthcheck
```

Exemple de build CI :

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --provenance=mode=max \
  --sbom=true \
  -t "$IMAGE:$VERSION" \
  -t "$IMAGE:$GIT_SHA" \
  --push .
```

Le secret de registry doit venir du système de secrets de la CI.

# 39. Docker Compose : exemple de projet complet

Arborescence :

```text
project/
├── compose.yaml
├── .env.example
├── api/
│   ├── Dockerfile
│   ├── pyproject.toml
│   └── src/
├── nginx/
│   └── default.conf
└── secrets/
    └── .gitignore
```

## 39.1 Dockerfile API

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.14-slim AS runtime

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN useradd --system --uid 10001 --create-home app

WORKDIR /app

COPY pyproject.toml ./
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --no-cache-dir .

COPY src/ ./src/

USER 10001
EXPOSE 8000

CMD ["python", "-m", "src"]
```

Dans un vrai projet, adapter l'ordre du `COPY` et la construction du paquet Python à l'outil utilisé afin de préserver correctement le cache.

## 39.2 Compose

```yaml
services:
  proxy:
    image: nginx:alpine
    ports:
      - "127.0.0.1:8080:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      api:
        condition: service_healthy
    networks:
      - frontend

  api:
    build:
      context: ./api
    environment:
      DATABASE_HOST: db
      DATABASE_NAME: app
      DATABASE_USER: app
      DATABASE_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    networks:
      - frontend
      - backend
    read_only: true
    tmpfs:
      - /tmp
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    healthcheck:
      test: ["CMD", "python", "-m", "src.healthcheck"]
      interval: 10s
      timeout: 3s
      retries: 5

  db:
    image: postgres:18
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - dbdata:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 5s
      timeout: 3s
      retries: 10

networks:
  frontend:
  backend:
    internal: true

volumes:
  dbdata:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## 39.3 Commandes

```bash
docker compose config
docker compose build
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
```

# 40. Erreurs fréquentes

## 40.1 Utiliser `docker-compose`

Ancien :

```bash
docker-compose up
```

Moderne :

```bash
docker compose up
```

## 40.2 Utiliser `DOCKER_BUILDKIT=1` comme obligation

BuildKit est déjà le builder par défaut pour les builds Linux modernes.

## 40.3 Installer tout dans un conteneur à la main

Mauvais workflow :

```text
docker run ubuntu
apt install ...
modifier ...
docker commit
```

Préférer un Dockerfile versionné.

## 40.4 Confondre image et sauvegarde

Une image contient le logiciel, pas automatiquement les données de production.

## 40.5 Mettre les secrets dans l'image

Ne jamais copier `.env`, clés privées ou mots de passe dans une image.

## 40.6 Utiliser `latest` comme stratégie de version

Un tag mutable ne garantit pas quel contenu sera exécuté.

## 40.7 Oublier les signaux

Une application qui ne traite pas `SIGTERM` proprement peut être interrompue brutalement.

## 40.8 Utiliser `--privileged` pour tout

C'est un contournement de sécurité majeur, pas un correctif générique.

## 40.9 Exposer une base de données inutilement

Mauvais :

```yaml
services:
  db:
    ports:
      - "5432:5432"
```

si seul le backend doit y accéder.

Laisser le service uniquement sur le réseau interne Compose.

## 40.10 Croire que `depends_on` rend l'application résiliente

La résilience nécessite toujours retry/backoff/reconnexion dans l'application.

# 41. Docker vs Podman vs Kubernetes

## Docker

Très bon pour :

- builds ;
- développement ;
- images OCI ;
- exploitation mono-hôte ;
- Compose ;
- écosystème large.

## Podman

Autre moteur de conteneurs OCI, daemonless dans de nombreux usages et fortement orienté rootless.

Les commandes sont proches mais les architectures ne sont pas identiques.

## Kubernetes

Orchestrateur de cluster.

Il ne remplace pas le format d'image et ne doit pas être confondu avec Docker Engine.

Kubernetes utilise des runtimes conformes à CRI, le plus souvent containerd ou CRI-O.

# 42. Swarm

Docker Swarm reste une fonction de Docker Engine pour l'orchestration multi-nœuds.

Initialisation :

```bash
docker swarm init
```

Services :

```bash
docker service create ...
docker service ls
```

Secrets Swarm :

```bash
printf '%s' 'secret' | docker secret create db_password -
```

Swarm peut rester pertinent pour certaines infrastructures simples, mais il faut choisir l'orchestrateur selon :

- compétences de l'équipe ;
- besoins de HA ;
- écosystème ;
- intégration cloud ;
- complexité acceptable.

# 43. Bonnes pratiques de production

Checklist d'image :

- [ ] image de base maintenue ;
- [ ] version ou digest maîtrisé ;
- [ ] multi-stage build lorsque pertinent ;
- [ ] pas de secret dans l'image ;
- [ ] utilisateur non-root ;
- [ ] dépendances verrouillées ;
- [ ] `.dockerignore` correct ;
- [ ] surface runtime minimale ;
- [ ] healthcheck lorsqu'il apporte une vraie information ;
- [ ] SBOM/provenance si la chaîne le permet.

Checklist runtime :

- [ ] pas de `--privileged` sans justification ;
- [ ] capabilities minimales ;
- [ ] volumes strictement nécessaires ;
- [ ] bind mounts sensibles en lecture seule ;
- [ ] ports uniquement nécessaires ;
- [ ] limites de ressources ;
- [ ] stratégie de logs ;
- [ ] redémarrage maîtrisé ;
- [ ] sauvegardes testées ;
- [ ] mises à jour de sécurité régulières.

Checklist daemon/hôte :

- [ ] noyau et Docker à jour ;
- [ ] accès au socket Docker restreint ;
- [ ] API distante non exposée sans authentification/TLS ;
- [ ] pare-feu compris et testé ;
- [ ] disque surveillé ;
- [ ] rootless évalué ;
- [ ] AppArmor/SELinux/seccomp conservés lorsque possible.

# 44. TP 1 — Cycle de vie

Objectif : comprendre l'immutabilité de l'image et l'éphémérité du conteneur.

1. Lancer :

```bash
docker run --name tp1 -it alpine sh
```

2. Dans le conteneur :

```sh
echo test > /tmp/fichier
```

3. Quitter.
4. Relancer le même conteneur :

```bash
docker start -ai tp1
```

5. Vérifier que `/tmp/fichier` existe.
6. Supprimer le conteneur puis créer un nouveau conteneur depuis la même image.
7. Expliquer pourquoi le fichier a disparu.

# 45. TP 2 — Construire une image

Créer une petite application HTTP.

Contraintes :

- Dockerfile versionné ;
- utilisateur non-root ;
- forme exec de `CMD` ;
- port documenté ;
- `.dockerignore` ;
- aucune dépendance installée au démarrage.

Construire :

```bash
docker build -t tp-http:1 .
```

# 46. TP 3 — Cache de build

1. Construire une application avec dépendances.
2. Modifier uniquement un fichier source.
3. Observer les étapes réutilisées depuis le cache.
4. Réorganiser le Dockerfile pour améliorer le cache.
5. Comparer avec :

```bash
docker buildx build --progress=plain .
```

# 47. TP 4 — Multi-stage

Créer :

```text
build stage -> artefact compilé -> runtime stage
```

Comparer :

```bash
docker image ls
```

et inspecter les paquets présents dans l'image runtime.

# 48. TP 5 — Volumes

Créer deux conteneurs qui lisent le même volume nommé.

Questions :

1. les conteneurs peuvent-ils être supprimés sans perdre le volume ?
2. que fait `docker volume rm` ?
3. quel mécanisme de sauvegarde serait adapté aux données ?

# 49. TP 6 — Réseaux

Créer :

```text
frontend <-> api <-> db
```

avec :

- `frontend` et `api` sur `frontnet` ;
- `api` et `db` sur `backnet` ;
- `db` absent de `frontnet` ;
- aucun port de base publié vers l'hôte.

Tester la résolution DNS par nom.

# 50. TP 7 — Compose

Transformer le TP précédent en `compose.yaml`.

Ajouter :

- volume ;
- healthcheck ;
- `depends_on` conditionnel ;
- réseau interne ;
- `docker compose config` dans la validation.

# 51. TP 8 — Durcissement

Partir d'un service fonctionnel puis essayer de le faire tourner avec :

```yaml
read_only: true
cap_drop:
  - ALL
security_opt:
  - no-new-privileges:true
```

Ajouter uniquement les droits réellement nécessaires.

Documenter chaque exception.

# 52. TP 9 — Rootless

Sur une machine de laboratoire compatible :

1. installer/configurer Docker rootless ;
2. vérifier l'UID du daemon ;
3. lancer un conteneur ;
4. tester volume et réseau ;
5. noter les différences observées par rapport au daemon rootful.

# 53. TP 10 — Multi-architecture

Construire une image :

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t <registry>/tp-multi:1 \
  --push .
```

Inspecter :

```bash
docker buildx imagetools inspect <registry>/tp-multi:1
```

# 54. TP 11 — SBOM et vulnérabilités

Sur une image de laboratoire :

```bash
docker scout sbom --format list <image>
docker scout cves <image>
docker scout recommendations <image>
```

Questions :

- quelle est l'image de base ?
- quels paquets sont vulnérables ?
- existe-t-il une base plus récente ?
- chaque CVE est-elle exploitable dans notre contexte ?

# 55. TP 12 — Supply chain

Construire et pousser :

```bash
docker buildx build \
  --sbom=true \
  --provenance=mode=max \
  --tag <registry>/tp-attest:1 \
  --push .
```

Vérifier ensuite les attestations avec les outils Docker/OCI disponibles dans l'environnement.

# 56. TP 13 — Diagnostic

Un enseignant prépare volontairement une application avec plusieurs erreurs :

- mauvais port ;
- application liée sur loopback ;
- volume non inscriptible ;
- healthcheck erroné ;
- DNS de service incorrect ;
- OOM ;
- processus principal qui termine immédiatement.

Utiliser :

```bash
docker ps -a
docker logs
docker inspect
docker stats
docker network inspect
docker volume inspect
docker system df
```

pour établir un diagnostic reproductible.

# 57. TP 14 — Projet final

Construire une application complète avec :

```text
reverse proxy
     |
API
     |
base de données
```

Contraintes :

- Dockerfiles multi-stage si pertinent ;
- Compose Specification ;
- réseau frontend/backend ;
- base non exposée publiquement ;
- volume persistant ;
- secrets hors image ;
- healthchecks ;
- utilisateur non-root ;
- système de fichiers read-only pour l'API si possible ;
- limites de ressources ;
- logs sur stdout/stderr ;
- SBOM ;
- provenance ;
- documentation de sauvegarde/restauration ;
- procédure de mise à jour ;
- procédure de rollback.

Livrables :

```text
Dockerfile(s)
compose.yaml
.dockerignore
README.md
document d'architecture
SBOM
procédure d'exploitation
```

# 58. Commandes de référence

## Conteneurs

```bash
docker run
docker create
docker start
docker stop
docker restart
docker kill
docker rm
docker ps
docker exec
docker logs
docker inspect
docker stats
docker top
docker cp
```

## Images

```bash
docker pull
docker push
docker image ls
docker image inspect
docker image history
docker image rm
docker tag
```

## Build

```bash
docker build
docker buildx build
docker buildx ls
docker buildx inspect
docker buildx create
docker buildx prune
docker buildx imagetools inspect
```

## Réseau

```bash
docker network ls
docker network create
docker network inspect
docker network connect
docker network disconnect
docker network rm
```

## Volumes

```bash
docker volume ls
docker volume create
docker volume inspect
docker volume rm
docker volume prune
```

## Compose

```bash
docker compose config
docker compose build
docker compose pull
docker compose up
docker compose down
docker compose ps
docker compose logs
docker compose exec
docker compose run
docker compose watch
docker compose --profile <profil> up
```

# 59. Méthode de mise à jour d'une application Docker

Une bonne mise à jour ne consiste pas à entrer dans le conteneur et à faire `apt upgrade`.

Workflow :

1. mettre à jour la référence de l'image de base ou les dépendances ;
2. reconstruire ;
3. exécuter les tests ;
4. scanner ;
5. produire SBOM/provenance ;
6. pousser sous un nouveau tag immuable ;
7. déployer ;
8. observer le healthcheck et les métriques ;
9. conserver une procédure de rollback.

Pour Compose :

```bash
docker compose pull
docker compose up -d
```

peut suffire pour des services basés sur des images déjà construites, mais il faut connaître les implications de migration des données et de compatibilité applicative.

# 60. Ce qu'il faut retenir

Docker ne doit pas être appris comme une liste de commandes.

Le modèle mental essentiel est :

```text
source
  |
Dockerfile
  |
Buildx
  |
BuildKit
  |
image OCI immuable
  |
registry
  |
conteneur
  |
namespaces + cgroups + LSM + seccomp
  |
volumes / réseaux / configuration / secrets
```

Les points les plus importants sont :

1. **une image est un artefact ; un conteneur est une exécution de cet artefact** ;
2. **un conteneur partage le noyau hôte** ;
3. **la couche écrivable du conteneur n'est pas une stratégie de persistance** ;
4. **les secrets ne doivent jamais être intégrés à l'image** ;
5. **BuildKit est le builder moderne par défaut** ;
6. **`docker compose` remplace l'ancien `docker-compose`** ;
7. **Compose est excellent en mono-hôte, mais ce n'est pas un orchestrateur de cluster complet** ;
8. **le socket Docker est privilégié** ;
9. **rootless et le moindre privilège réduisent le risque** ;
10. **la supply chain doit être vérifiable avec digests, SBOM et provenance** ;
11. **les données doivent être sauvegardées indépendamment des conteneurs** ;
12. **un déploiement doit être reproductible depuis le code et la configuration**.

# Références

Documentation officielle Docker :

```text
https://docs.docker.com/
https://docs.docker.com/engine/
https://docs.docker.com/build/
https://docs.docker.com/compose/
https://docs.docker.com/engine/security/
https://docs.docker.com/engine/security/rootless/
https://docs.docker.com/storage/
https://docs.docker.com/network/
https://docs.docker.com/scout/
```

Standards OCI :

```text
https://opencontainers.org/
https://github.com/opencontainers/image-spec
https://github.com/opencontainers/runtime-spec
https://github.com/opencontainers/distribution-spec
```

Voir aussi :

- [[Les namespaces Linux]]
- [[GNULinux]]
- [[Sécurité avancée sous Linux]]
- [[git]]
