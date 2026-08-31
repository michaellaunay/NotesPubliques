---
schema_version: 1
uid: 01M1BQ62DTSDTZ0C7ABDYJEZTS
titre: "Docker — 01 — Introduction à Docker"
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
resume: "Chapitre 1 sur 17 du livre « Docker » : Introduction à Docker. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 1/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — Sommaire|← Sommaire]] · [[Docker — 02 — Définition|02 — Définition →]]

# Introduction à Docker
Docker est un logiciel qui permet d'empaqueter une application avec toutes ses dépendances, et de les exécuter de manière isolée dans un environnement standardisé.

La principale différence entre les conteneurs et les machines virtuelles, est que les conteneurs utilisent le système d'exploitation hôte, tandis que les machines virtuelles utilisent leur propre système d'exploitation.
Les conteneurs sont donc plus légers et plus rapides à démarrer.

La raison principale d'utilisation de Docker est faciliter le déploiement d'applications, de les rendre portables et de garantir une meilleure reproductibilité des environnements.
Le système hébergeant Docker est appelé l’hôte, c'est le système d'exploitation de l'ordinateur qui fera fonctionner docker et les images docker.

Les images sont décrites au format texte dans des fichiers appelés DockerFiles.
Ces modèles d'images sont partagés sur un dépôt public appelé DockerHUB accessible à l'adresse https://www.docker.com/ .
Toutefois, il faut faire attention au DockerFile qui y sont, car il y a peu ou pas de vérification de la majorité des images.

Sur windows 11, Docker utilise [[WSL2]] et propose sa configuration lors de son installation, généralement nous allons chercher le logiciel Docker sur https://docker.com (voir https://www.youtube.com/watch?v=6Wpp6C8cUf0&ab_channel=angleformation )
Sur Ubuntu ou Debian, soit nous utilisons les paquets de la distribution soit nous téléchargeons la dernière version ( https://youtu.be/AWJG-AbGFik )

---
> [!info] Livre « Docker » — chapitre 1/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — Sommaire|← Sommaire]] · [[Docker — 02 — Définition|02 — Définition →]]
