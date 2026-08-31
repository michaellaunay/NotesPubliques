---
schema_version: 1
uid: 01M1BQ62D5DH0QQADHRDPAY0ZR
titre: "Pyramid — 09 — Pyramid et Docker"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - developpement-web
  - python
  - pyramid
resume: "Chapitre 9 sur 10 du livre « Pyramid » : Pyramid et Docker. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-06-14
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « Pyramid » — chapitre 9/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 08 — Déploiement de l'application Pyramid|← 08 — Déploiement de l'application Pyramid]] · [[Pyramid — 10 — Exemples de code|10 — Exemples de code →]]

# 9. Pyramid et Docker
Docker est une excellente option pour déployer des applications Pyramid car il nous permet d'encapsuler notre application et toutes ses dépendances dans un conteneur, ce qui facilite le déploiement et l'exécution de notre application dans divers environnements.

Pour utiliser Docker avec Pyramid, nous devrons créer un fichier `Dockerfile` qui décrit comment créer une image Docker pour notre application. Voici un exemple de base d'un `Dockerfile` pour une application Pyramid :

```Dockerfile
# Utilisins une image Python comme image de base
FROM python:3.9-slim-buster

# Créons un répertoire de travail dans le conteneur
WORKDIR /app

# Copions les fichiers de dépendance dans le conteneur
COPY requirements.txt .

# Installons les dépendances de l'application
RUN pip install --no-cache-dir -r requirements.txt

# Copions le reste du code de l'application dans le conteneur
COPY . .

# Exposons le port sur lequel notre application s'exécute
EXPOSE 6543

# Lançons le serveur de développement de Pyramid
CMD ["pserve", "development.ini"]
```

Notons que cette configuration utilise le serveur de développement inclus avec Pyramid, qui n'est pas destiné à être utilisé en production. Pour un déploiement en production, nous devrions utiliser un serveur WSGI comme Gunicorn ou uWSGI.

Pour construire une image Docker à partir de ce `Dockerfile`, nous pouvons utiliser la commande `docker build` :

```shell
docker build -t my-pyramid-app .
```

Et pour exécuter un conteneur basé sur cette image, nous pouvons utiliser la commande `docker run` :

```shell
docker run -p 6543:6543 my-pyramid-app
```

Cela démarre notre application Pyramid dans un conteneur Docker et expose le port 6543 pour que nous puissions y accéder.

Notons que la manière dont nous structurons notre `Dockerfile` et la façon dont nous utilisons Docker peuvent varier en fonction de nos besoins spécifiques. Par exemple, si nous utilisons une base de données, nous devrons probablement configurer notre application pour qu'elle puisse y accéder, et nous pourrions utiliser Docker Compose pour gérer à la fois notre application et notre base de données.

---
> [!info] Livre « Pyramid » — chapitre 9/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 08 — Déploiement de l'application Pyramid|← 08 — Déploiement de l'application Pyramid]] · [[Pyramid — 10 — Exemples de code|10 — Exemples de code →]]
