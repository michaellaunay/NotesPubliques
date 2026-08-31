---
schema_version: 1
uid: 01M1BQ62DVCBQM36CRY78078B5
titre: "Docker — 03 — Installation"
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
resume: "Chapitre 3 sur 17 du livre « Docker » : Installation. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 3/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 02 — Définition|← 02 — Définition]] · [[Docker — 04 — Terminologie|04 — Terminologie →]]

# Installation
Pour installer docker le plus simple est d'utiliser la version des dépôts et donc de faire : :
```bash
apt install docker.io
``` 

Nous allons chercher notre première image ce qui permettra de vérifier notre installation de docker et notre accès au réseau :
```bash
docker run -ti hello-world
Unable to find image
'hello-world:latest' locally latest: Pulling from
library/hello-world 2db29710123e: Pull complete Digest:
sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!

To generate this message, Docker took the following steps:

1.  The Docker client contacted the Docker daemon.
2.  The Docker daemon pulled the \"hello-world\" image from the Docker Hub.
3.  The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it  to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 docker run -it ubuntu bash

 Share images, automate workflows, and more with a free Docker ID:
 <https://hub.docker.com/>

 For more examples and ideas, visit:
 <https://docs.docker.com/get-started/>
```
Nous voyons que docker a été chercher l'image sur dockerhub, puis l'a exécuté en suivant les étapes indiquées par l'exécution.

Il est possible de faire les étapes d'exécution individuellement : :

```bash
docker pull hello-world
docker create --name docker-hello hello-world
docker start --attach docker-hello
```

L'exécution terminée le conteneur est détruit : :

```bash
root@luciole:~# docker ps --all
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
f842b65b9e36   hello-world   "/hello"   5 hours ago   Exited (0) 5 hours ago             funny_hermann
```

On peut alors supprimer le conteneur puis l'image du disque : :

    root@Caravale:~# docker rm funny_hermann
    funny_hermann
    root@Caravale:~# docker rmi hello-world
    ...

Pour faire le ménage de toutes les images non en cours d'exécution : :

    docker rm $(sudo docker ps -a -q)
    docker rmi $(sudo docker images -a -q)

Différentes versions d'une image existent, elles sont taguées et il suffit de descendre l'une d'elles pour avoir une image partielle.

Pour tester la future version de python par exemple nous pouvons faire : :

    docker run python:3.11-rc-alpine python -c "from typing import TypeVar; help(TypeVar)"

Qui affichera la doc de TypeVar.

	Pour accéder au shell du conteneur en mode interactif, il faut mettre le flag `-it` ou indifféremment `ti`

- `-i` (interactive) : Maintient le flux d'entrée standard ouvert même si personne n'est connecté.
- `-t` (tty) : Alloue un pseudo-terminal.

Pour effacer le conteneur une fois terminé, mettre le flag `--rm`

Pour utiliser le système de fichier du host à la place de celui du Conteneur, cela pour pouvoir conserver des données entre images il faut utiliser l'option `-v` : 

    #lance le conteneur en -d detach -i interactive
    docker run -tid -p 8080:80 -v /var/host_nginx/:/usr/share/nginx/ --name web nginx:latest

    #permet de se connecter au bash du Conteneur
    docker exec -ti web bash

    # voir l'intégralité de la configuration du conteneur
    docker inspect web

    # voir l'état des conteners
    docker ps -a

    # pour arrêter un conteneur
    docker stop web

    # pour démarrer le conteneur
    docker start web

    # pour passer des variables d'environnement
    docker run -tid --env MaVariable="ça valeur" --name web nginx:latest

    # pour passer un fichier de Variables
    docker run -tid --env-file fichier_env --name web nginx:latest

---
> [!info] Livre « Docker » — chapitre 3/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 02 — Définition|← 02 — Définition]] · [[Docker — 04 — Terminologie|04 — Terminologie →]]
