---
schema_version: 1
uid: 01M1BQ62DV9ZTCMA38S3CJ4JND
titre: "Docker — 05 — Les commandes de bases"
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
resume: "Chapitre 5 sur 17 du livre « Docker » : Les commandes de bases. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-01-27
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « Docker » — chapitre 5/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 04 — Terminologie|← 04 — Terminologie]] · [[Docker — 06 — Le mode détaché|06 — Le mode détaché →]]

# Les commandes de bases
Les commandes de base les plus couramment utilisées sont :
- docker images : pour lister les images de conteneurs locales
- docker init : pour créer un docker file avec l'assistant contextuel
- docker ps : pour lister les conteneurs en cours d'exécution
- docker pull : pour télécharger une image de conteneur depuis un registre
- docker rm : pour supprimer un conteneur
- docker run : pour exécuter un conteneur à partir d'une image
- docker start : pour démarrer un conteneur qui a été arrêté
- docker stop : pour arrêter un conteneur en cours d'exécution
## Exemples
-   Télécharger l'image de conteneur `nginx` en utilisant la commande `docker pull nginx`
-   Exécuter un conteneur à partir de l'image `nginx` en utilisant la commande `docker run --name mynginx -p 80:80 -d nginx`
-   Arrêter un conteneur en cours d'exécution en utilisant la commande `docker stop mynginx`
-   Démarrer un conteneur qui a été arrêté en utilisant la commande `docker start mynginx`
-   Supprimer un conteneur en utilisant la commande `docker rm mynginx`
-   Lister les images de conteneurs locales en utilisant la commande `docker images`
-   Lister les conteneurs en cours d'exécution en utilisant la commande `docker ps`

## Mise en pratique
-   Télécharger une image de conteneur
-   Exécuter un conteneur à partir de l'image téléchargée
-   Arrêter et supprimer le conteneur
-   Lister les images de conteneurs locales et les conteneurs en cours d'exécution

---
> [!info] Livre « Docker » — chapitre 5/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 04 — Terminologie|← 04 — Terminologie]] · [[Docker — 06 — Le mode détaché|06 — Le mode détaché →]]
