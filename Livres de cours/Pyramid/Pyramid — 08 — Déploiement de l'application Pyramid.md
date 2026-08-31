---
schema_version: 1
uid: 01M1BQ62D50M8SY79CB9513JXT
titre: "Pyramid — 08 — Déploiement de l'application Pyramid"
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
resume: "Chapitre 8 sur 10 du livre « Pyramid » : Déploiement de l'application Pyramid. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « Pyramid » — chapitre 8/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 07 — Authentification LDAP|← 07 — Authentification LDAP]] · [[Pyramid — 09 — Pyramid et Docker|09 — Pyramid et Docker →]]

# 8. Déploiement de l'application Pyramid
## 8.1 Déploiement d'une application web

Le déploiement consiste à prendre une application qui fonctionne sur notre machine de développement et à la mettre à disposition sur Internet, de sorte que les utilisateurs puissent y accéder. Cela implique généralement de copier l'application sur un serveur web et de configurer ce serveur pour qu'il puisse exécuter l'application.

### 8.1.1 Importance du déploiement

Le déploiement est une étape importante du développement d'une application web. C'est le moment où notre application devient disponible pour le monde entier. Par conséquent, il est essentiel de veiller à ce que notre application soit aussi prête que possible pour le déploiement.

### 8.1.2 Options de déploiement pour Pyramid

Pyramid est un framework web flexible qui peut être déployé de plusieurs façons. Les options de déploiement les plus courantes pour Pyramid sont :

1. **WSGI** : Le serveur d'application Python le plus couramment utilisé. WSGI est l'interface standard entre les serveurs web et les applications web Python.
2. **uWSGI** : Un serveur d'application rapide et autonome qui peut servir des applications Pyramid.
3. **Gunicorn** : Un serveur HTTP Python WSGI HTTP pour UNIX.
4. **mod_wsgi** : Un module Apache qui fournit une interface WSGI pour les applications Python.

Toutes ces options ont leurs avantages et leurs inconvénients. Le choix de l'une ou l'autre dépendra de nos besoins spécifiques.

### 8.1.3 WSGI

WSGI est une spécification qui décrit comment un serveur web interagit avec des applications web en Python. Avant WSGI, il y avait une variété de méthodes non standardisées pour cette interaction. WSGI a été créé pour offrir une interface standardisée, de manière à ce que les applications web écrites en Python puissent être déployées sur n'importe quel serveur web supportant WSGI, indépendamment des détails spécifiques du serveur.

WSGI définit essentiellement deux rôles :

1. **Le côté serveur (ou "gateway")** : C'est généralement le serveur web lui-même, ou un module de celui-ci, qui reçoit les requêtes HTTP des clients (navigateurs web). Lorsqu'une requête arrive, le serveur la formate dans un format spécifique défini par la spécification WSGI.

2. **Le côté application** : C'est notre application web Python. Elle reçoit les requêtes du serveur sous forme de dictionnaires et de fonctions callback. L'application traite la requête et renvoie une réponse qui est ensuite renvoyée au client par le serveur.

Par exemple, dans une application Pyramid, le fichier que nous exécutons pour démarrer notre application contiendra généralement du code pour créer une instance d'application WSGI. C'est ce qui se passe lorsque nous appelons `config.make_wsgi_app()` dans notre code Pyramid.

Ce code génère une application WSGI qui peut ensuite être servie à l'aide d'un serveur WSGI. Dans les exemples précédents, nous avons utilisé `wsgiref.simple_server.make_server`, qui est un serveur WSGI simple fourni par la bibliothèque standard Python.

Dans une production réelle, nous utiliserions un serveur WSGI plus robuste, comme Gunicorn ou uWSGI, pour servir notre application.

C'est en substance ce qu'est WSGI, un élément clé de la plupart des applications web Python et un élément important pour comprendre comment fonctionnent les applications Pyramid.

## 8.2 Configuration de l'environnement de production

### 8.2.1 Introduction

Lorsque nous déployons une application Pyramid, la première étape consiste généralement à configurer l'environnement de production. Cela comprend l'installation de Python, la configuration du serveur web, l'installation des dépendances de l'application et la configuration de l'application elle-même.

### 8.2.2 Installation de Python

Le déploiement d'une application Pyramid nécessite une installation de Python sur notre serveur. La version exacte de Python dont nous avons besoin dépendra de notre application. Pour installer Python, nous pouvons généralement utiliser le gestionnaire de paquets de notre système.

### 8.2.3 Configuration du serveur web

Pour servir notre application Pyramid, nous aurons besoin d'un serveur web. Le choix du serveur web dépend de nos préférences personnelles et des besoins de notre application. Les options populaires incluent Nginx, Apache, et Gunicorn.

### 8.2.4 Installation des dépendances de l'application

Notre application Pyramid dépend probablement de plusieurs packages Python. nous pouvons installer ces dépendances à l'aide de pip, l'outil d'installation de paquets Python. Il est généralement recommandé de créer un environnement virtuel Python pour notre application afin d'éviter les conflits de dépendances.

### 8.2.5 Configuration de l'application

Enfin, nous devrons configurer notre application pour l'environnement de production. Cela peut inclure des choses comme la configuration des paramètres de connexion à la base de données, la configuration du logging, et la configuration des paramètres spécifiques à l'environnement dans notre fichier de configuration Pyramid.

### 8.3 Déploiement de l'application Pyramid sur un serveur local

### 8.3.1  Introduction

Après avoir configuré l'environnement de production, l'étape suivante consiste à déployer réellement l'application Pyramid sur le serveur. Cela comprend le transfert des fichiers de l'application sur le serveur, l'installation des dépendances, la configuration de l'application pour l'environnement de production et le démarrage du serveur de l'application.

### 8.3.2 Transfert des fichiers de l'application

Il existe de nombreuses façons de transférer les fichiers de notre application sur notre serveur. Une méthode courante consiste à utiliser Git. nous pouvons simplement pousser notre code sur un dépôt Git, puis le cloner sur notre serveur. Une autre méthode consiste à utiliser SCP (Secure Copy) ou FTP (File Transfer Protocol) pour transférer directement les fichiers.

### 8.3.3 Installation des dépendances

Une fois que nous avons transféré les fichiers de l'application, nous devrons installer les dépendances. Cela peut être fait en utilisant pip à l'intérieur de l'environnement virtuel que nous avons créé. Si nous avons un fichier `requirements.txt`, nous pouvons installer toutes les dépendances en une seule fois en utilisant la commande `pip install -r requirements.txt`.

### 8.3.4 Configuration de l'application

La configuration de notre application pour l'environnement de production peut nécessiter quelques étapes supplémentaires. Par exemple, nous devrons peut-être configurer la connexion à la base de données, les informations d'authentification ou autres paramètres spécifiques à l'environnement. 

### 8.3.5 Démarrage du serveur de l'application

Une fois que tout est en place, nous pouvons démarrer notre application Pyramid. La façon exacte de le faire dépend de la façon dont nous avons configuré notre serveur web. Par exemple, si nous utilisons Gunicorn, nous pouvons démarrer notre application en utilisant la commande `gunicorn myapp:app`.

---
> [!info] Livre « Pyramid » — chapitre 8/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 07 — Authentification LDAP|← 07 — Authentification LDAP]] · [[Pyramid — 09 — Pyramid et Docker|09 — Pyramid et Docker →]]
