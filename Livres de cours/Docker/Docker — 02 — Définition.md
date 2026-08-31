---
schema_version: 1
uid: 01M1BQ62DV038G190G9X70S8G6
titre: "Docker — 02 — Définition"
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
resume: "Chapitre 2 sur 17 du livre « Docker » : Définition. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 2/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 01 — Introduction à Docker|← 01 — Introduction à Docker]] · [[Docker — 03 — Installation|03 — Installation →]]

# Définition
Docker est un logiciel open-source qui facilite la création, le déploiement et l'exécution d'applications dans des conteneurs logiciels. Un conteneur est un environnement logiciel qui contient tout ce dont une application a besoin pour fonctionner, comme les bibliothèques, les outils de développement et les fichiers de configuration. Les conteneurs sont similaires aux machines virtuelles, mais ils sont beaucoup plus légers et plus rapides à démarrer, car ils partagent les ressources de l'hôte sur lequel ils sont exécutés.

L'un des avantages clés de Docker est sa portabilité. Les conteneurs Docker peuvent être créés sur un ordinateur de développement, testés sur un autre et déployés sur un serveur de production, tout en étant sûrs qu'ils fonctionneront de la même manière dans chaque environnement.

Docker utilise des images pour construire les conteneurs. Une image est une capture de l'état d'une application, qui comprend tous les fichiers et les configurations nécessaires pour exécuter l'application. Il est possible de créer des images à partir de conteneurs existants ou de construire des images à partir de zéro en utilisant un script de construction appelé "Dockerfile".

Docker permet également de gérer facilement les conteneurs à l'échelle. Nous pouvons utiliser des outils tels que Docker Compose pour définir des applications multi-conteneurs et les déployer sur plusieurs serveurs, ou utiliser des services tels que Kubernetes pour gérer les conteneurs à l'échelle de production.

En résumé, Docker est un système de virtualisation de conteneurs qui permet de créer, de déployer et d'exécuter des applications de manière efficiente et portable.

---
> [!info] Livre « Docker » — chapitre 2/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 01 — Introduction à Docker|← 01 — Introduction à Docker]] · [[Docker — 03 — Installation|03 — Installation →]]
