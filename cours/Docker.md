#Cours #Informatique 
# Objectif
Nous allons voir dans ce cours comment installer et utiliser Docker.
Docker est un système de conteneurisation, c'est-à-dire qu'il permet de crée des images d'une autre distribution très légère qui utilisent ou partagent les ressources de l'ordinateur hôte.

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

# Définition
Docker est un logiciel open-source qui facilite la création, le déploiement et l'exécution d'applications dans des conteneurs logiciels. Un conteneur est un environnement logiciel qui contient tout ce dont une application a besoin pour fonctionner, comme les bibliothèques, les outils de développement et les fichiers de configuration. Les conteneurs sont similaires aux machines virtuelles, mais ils sont beaucoup plus légers et plus rapides à démarrer, car ils partagent les ressources de l'hôte sur lequel ils sont exécutés.

L'un des avantages clés de Docker est sa portabilité. Les conteneurs Docker peuvent être créés sur un ordinateur de développement, testés sur un autre et déployés sur un serveur de production, tout en étant sûrs qu'ils fonctionneront de la même manière dans chaque environnement.

Docker utilise des images pour construire les conteneurs. Une image est une capture de l'état d'une application, qui comprend tous les fichiers et les configurations nécessaires pour exécuter l'application. Il est possible de créer des images à partir de conteneurs existants ou de construire des images à partir de zéro en utilisant un script de construction appelé "Dockerfile".

Docker permet également de gérer facilement les conteneurs à l'échelle. Nous pouvons utiliser des outils tels que Docker Compose pour définir des applications multi-conteneurs et les déployer sur plusieurs serveurs, ou utiliser des services tels que Kubernetes pour gérer les conteneurs à l'échelle de production.

En résumé, Docker est un système de virtualisation de conteneurs qui permet de créer, de déployer et d'exécuter des applications de manière efficiente et portable.

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

Pour utiliser le système de fichier du host à la place de celui du Contener, cela pour pouvoir conserver des données entre images il faut utiliser l'option `-v` : 

    #lance le conteneur en -d detach -i interactive
    docker run -tid -p 8080:80 -v /var/host_nginx/:/usr/share/nginx/ --name web nginx:latest

    #permet de se connecter au bash du Contener
    docker exec -ti web bash

    # voir l'intégralité de la configuration du contener
    docker inspect web

    # voir l'état des conteners
    docker ps -a

    # pour arrêter un contener
    docker stop web

    # pour démarrer le contener
    docker start web

    # pour passer des variables d'environnement
    docker run -tid --env MaVariable="ça valeur" --name web nginx:latest

    # pour passer un fichier de Variables
    docker run -tid --env-file fichier_env --name web nginx:latest

# Terminologie
Dans le contexte de Docker, la terminologie "conteneur" et "image" est essentielle pour comprendre le fonctionnement des applications virtualisées. Voici une explication détaillée de ces deux concepts :

## Image Docker
Une **image** Docker est un package léger, autonome et exécutable qui inclut tout ce qui est nécessaire pour faire fonctionner un logiciel, y compris le code, les bibliothèques, les dépendances et les fichiers de configuration. En termes simples, une image est un modèle de base à partir duquel un conteneur peut être lancé.

**Caractéristiques d'une image :**
- **Immutable** : Une image est en lecture seule. Une fois créée, elle ne peut pas être modifiée. Si des modifications sont nécessaires, une nouvelle image doit être construite.
- **Empilable** : Les images Docker sont construites à partir de couches empilées. Chaque couche correspond à une modification ou à une instruction dans le `Dockerfile`, le script utilisé pour créer l'image.
- **Portable** : Une image peut être déplacée d'un environnement à un autre (comme un PC de développement, un serveur de production ou un cloud) et exécutée de manière cohérente.

## Conteneur Docker
Un **conteneur** Docker est une instance d'une image Docker. Lorsqu'une image est exécutée, elle devient un conteneur, qui est un environnement isolé où l'application peut s'exécuter de manière autonome.

**Caractéristiques d'un conteneur :**
- **Éphémère et modifiable** : Contrairement aux images, les conteneurs peuvent être modifiés en cours d'exécution. Cependant, les modifications apportées aux conteneurs ne persistent pas après leur suppression, sauf si elles sont sauvegardées dans une image mise à jour ou si des volumes externes sont utilisés.
- **Isolé** : Chaque conteneur fonctionne dans un espace isolé qui contient tout ce dont l'application a besoin, y compris un système de fichiers, des variables d'environnement et des bibliothèques. Cela permet d'éviter les conflits entre les applications qui tournent sur le même host.
- **Léger** : Les conteneurs partagent le noyau du système d'exploitation de l'hôte, ce qui les rend beaucoup plus légers que les machines virtuelles classiques. Cela leur permet de démarrer rapidement et d'utiliser moins de ressources.

## Différences entre Conteneur et Image
- **Image** : C’est un modèle statique et immuable qui définit comment un conteneur sera construit et quelles ressources il aura.
- **Conteneur** : C’est une instance active, exécutable et isolée de l’image. On peut créer plusieurs conteneurs à partir d'une seule image.
## Analogie
Imaginez qu'une image soit comme une classe dans un programme orienté objet, et qu'un conteneur soit comme un objet créé à partir de cette classe :
- **Image** = la classe (la définition de ce que sera l'objet).
- **Conteneur** = l'objet (une instance de la classe, qui peut être manipulée et utilisée).
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

# Le mode détaché

L'option `-d` ou `--detach` dans Docker signifie "détaché". Lorsqu'elle est utilisée, cette option permet de lancer un conteneur en arrière-plan et de le détacher du terminal. Cela signifie que la commande Docker retourne immédiatement le contrôle au terminal et le conteneur continue de s'exécuter en arrière-plan.

## Détails de l'Option `-d` :
- **Arrière-plan** : Le conteneur est lancé en mode détaché, ce qui est utile lorsque nous ne souhaitons pas que le conteneur monopolise la ligne de commande. Nous pouvons ainsi continuer à utiliser le terminal pour d'autres tâches pendant que le conteneur fonctionne.
- **Sortie du conteneur** : Étant donné que le conteneur est exécuté en arrière-plan, la sortie standard du conteneur n'apparaît pas dans le terminal. Cependant, nous pouvons voir les logs du conteneur en utilisant la commande `docker logs`.
- **Cas d'utilisation** : Le mode détaché est souvent utilisé pour les conteneurs de services qui doivent tourner de manière autonome, comme un serveur web, une base de données, etc.

## Exemple d'utilisation :
```bash
docker run -d --name mon_conteneur nginx
```
Dans cet exemple, le conteneur utilisant l'image `nginx` sera lancé en mode détaché et nommé `mon_conteneur`. Nous pourrons continuer à utiliser le terminal après l'exécution de cette commande.

Pour vérifier que le conteneur est bien en cours d'exécution, utilisez :
```bash
docker ps
```

## Avantages :
- **Contrôle du terminal** : Nous pouvons continuer à travailler dans notre terminal pendant que le conteneur tourne en arrière-plan.
- **Facilité de gestion** : Idéal pour les applications de fond et les services qui doivent rester actifs même lorsque l'utilisateur n'est pas connecté.


# Principales commandes
## Installation
```bash
apt install docker.io

```
## Liste des commandes
```
root@leojag:~# docker

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.

To get more help with docker, check out our guides at https://docs.docker.com/go/guides/

```
## Commandes les plus couramment utilisées

-   `docker run` : cette commande permet de lancer un conteneur à partir d'une image. Par exemple, pour lancer un conteneur à partir de l'image Ubuntu, nous pourrions utiliser la commande `docker run ubuntu`.
    
-   `docker ps` : cette commande permet de voir les conteneurs en cours d'exécution sur le système. Elle affiche des informations telles que l'ID du conteneur, le nom, le statut, etc.
    
-   `docker stop` : cette commande permet d'arrêter un conteneur en cours d'exécution. Par exemple, pour arrêter un conteneur avec l'ID `abc123`, nous pourrions utiliser la commande `docker stop abc123`.
    
-   `docker images` : cette commande permet de voir les images disponibles sur le système. Elle affiche des informations telles que l'ID de l'image, le nom, la taille, etc.
    
-   `docker pull` : cette commande permet de télécharger une image depuis un registre Docker. Par exemple, pour télécharger l'image Ubuntu, nous pourrions utiliser la commande `docker pull ubuntu`.
    
-   `docker build` : cette commande permet de construire une image à partir d'un fichier "Dockerfile". Par exemple, si nous avons un fichier "Dockerfile" dans le répertoire courant, nous pourrions utiliser la commande `docker build .` pour construire une image à partir de ce fichier.
    
-   `docker push` : cette commande permet de pousser une image vers un registre Docker. Par exemple, si nous avons construit une image et que nous voulons la partager avec d'autres, nous pourrions utiliser la commande `docker push nom_de_l_image`
    
-   `docker exec` : cette commande permet d'exécuter une commande dans un conteneur en cours d'exécution. Par exemple, pour ouvrir une session shell dans un conteneur avec l'ID `abc123`, nous pourrions utiliser la commande `docker exec -it abc123 /bin/bash`.

## Voir les conteneurs qui tournent
```bash
docker ps
```
## Voir tous les conteneurs
```bash
docker ps -a
```
## Descendre une image
```
docker pull alpine:latest
	latest: Pulling from library/alpine
	8921db27df28: Pull complete 
		Digest: sha256:f271e74b17ced29b915d351685fd4644785c6d1559dd1f2d4189a5e851ef753a
	Status: Downloaded newer image for alpine:latest
	docker.io/library/alpine:latest
```

## Lancer un conteneur
```
root@leojag:~# docker run alpine
```

## Lancer en mode interactif un conteneur
```
root@leojag:~# docker run -di --name mon_instance_d_alpine alpine
root@leojag:~# docker exec -ti mon_instance sh
/ # ps #Nous sommes dans le contener on voit qu'il n'y a presque pas de processus
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
   13 root      0:00 sh
   19 root      0:00 ps
/ # exit
 
root@leojag:~# docker stop mon_instance 
mon_instance

```
L'option -name permet de fixer un nom sans quoi docker met des noms au hazard.

Pour reprendre le contrôle d'un conteneur détaché et le rattacher afin de fonctionner en mode interactif, il y a plusieurs approches possibles :

# Utiliser `docker attach`
La commande `docker attach` permet de rattacher le terminal courant à un conteneur en cours d'exécution. Cela nous permet de visualiser la sortie en direct du conteneur et d'interagir avec lui si un terminal a été alloué (lors de l'utilisation de `-t`).

**Exemple :**
```bash
docker attach mon_conteneur
```
*Remarque :* Si le conteneur ne dispose pas de session interactive (par exemple, s'il a été lancé sans l'option `-it`), la commande `docker attach` nous permettra seulement de voir la sortie du conteneur, sans interaction possible.

## Limites de `docker attach` :
- Si nous quittons le conteneur en appuyant sur `CTRL+C`, cela arrêtera le conteneur.
- Pour se détacher sans arrêter le conteneur, utilisez `CTRL+P` suivi de `CTRL+Q`.

# Utiliser `docker exec`
La méthode la plus courante et la plus flexible pour interagir avec un conteneur en mode interactif est d'utiliser la commande `docker exec`. Cette commande permet d'exécuter un processus à l'intérieur d'un conteneur en cours d'exécution, comme ouvrir un shell interactif.

**Exemple :**
```bash
docker exec -ti mon_conteneur bash
```
- **`-ti`** : Ouvre une session interactive avec un terminal.
- **`bash`** : Indique que nous souhaitons ouvrir un shell Bash (assurez-nous que Bash est installé dans le conteneur).

# Différences entre `docker attach` et `docker exec` :
- **`docker attach`** : Rattache le terminal au processus principal du conteneur. Utile si nous souhaitons reprendre la sortie du conteneur tel qu'il a été lancé.
- **`docker exec`** : Lance une nouvelle session ou un nouveau processus à l'intérieur du conteneur en cours d'exécution. C'est l'approche recommandée pour obtenir un shell interactif ou pour exécuter des commandes supplémentaires.

### Exemple complet :
Si nous avons un conteneur nommé `web` exécuté en mode détaché, nous pouvons interagir avec lui de cette manière :
1. **Vérifier l'état du conteneur** :
   ```bash
   docker ps
   ```

2. **Lancer un shell interactif** :
   ```bash
   docker exec -ti web bash
   ```

# Le Mécanisme des Couches (Layers) dans Docker

Dans Docker, le concept de **couches** (ou *layers*) est fondamental pour comprendre comment les images sont construites, stockées et partagées. Ce mécanisme permet une gestion efficace des ressources, une optimisation du stockage et une accélération des processus de construction et de déploiement des conteneurs. Dans cette section, nous allons explorer en détail le fonctionnement des couches dans Docker, leur impact sur les images et les conteneurs, ainsi que les bonnes pratiques associées.

## Introduction aux Couches Docker

Une **image Docker** est construite à partir d'une série d'instructions définies dans un **Dockerfile**. Chaque instruction dans le Dockerfile crée une nouvelle couche dans l'image. Ces couches sont empilées les unes sur les autres pour former l'image finale. Le système de fichiers du conteneur est le résultat de la combinaison de ces couches.

### Analogie

Pensez à une image Docker comme à un gâteau à étages. Chaque couche du gâteau représente une étape de construction ajoutée par une instruction du Dockerfile. Lorsque nous assemblons toutes les couches, nous obtenons le gâteau complet, prêt à être consommé, tout comme l'image Docker prête à être exécutée.

# Les Dockerfiles

Le **Dockerfile** est un fichier texte qui contient une série d'instructions pour construire une image Docker personnalisée. Il sert de plan à Docker pour savoir comment créer une image qui inclut tous les composants nécessaires à l'exécution d'une application, tels que les dépendances, les configurations et le code source. En automatisant le processus de création d'images, les Dockerfiles permettent d'assurer la cohérence entre les environnements de développement, de test et de production, tout en facilitant le déploiement des applications.

## Structure d'un Dockerfile

Un Dockerfile est composé d'une séquence d'instructions, chacune commençant par un **mot-clé** spécifique suivi des arguments requis. Les principaux mots-clés utilisés dans un Dockerfile sont :

- `FROM`
- `ARG`
- `ENV`
- `RUN`
- `COPY`
- `ADD`
- `WORKDIR`
- `EXPOSE`
- `VOLUME`
- `USER`
- `ENTRYPOINT`
- `CMD`
- `LABEL`
- `ONBUILD`
- `HEALTHCHECK`

### Description des Instructions

#### FROM

- **Description** : Définit l'image de base à partir de laquelle l'image actuelle sera construite. C'est généralement la première instruction d'un Dockerfile.
- **Syntaxe** : `FROM [nom_image]:[tag]`
- **Exemple** : `FROM ubuntu:20.04`

> **Bonnes pratiques** : Choisissons une image de base légère et adaptée à notre application pour minimiser la taille de l'image finale.

#### ARG

- **Description** : Définit des variables disponibles uniquement pendant la phase de construction de l'image.
- **Syntaxe** : `ARG nom_variable[=valeur_par_défaut]`
- **Exemple** : `ARG APP_VERSION=1.0.0`

> **Remarque** : Les variables `ARG` sont utiles pour passer des paramètres de construction tels que des versions de paquets.

#### ENV

- **Description** : Définit des variables d'environnement qui seront disponibles lors de l'exécution du conteneur.
- **Syntaxe** : `ENV nom_variable valeur`
- **Exemple** : `ENV NODE_ENV=production`

> **Utilisation** : Les variables `ENV` sont utilisées pour configurer l'environnement d'exécution de notre application.

#### RUN

- **Description** : Exécute une commande pendant la construction de l'image. Chaque instruction `RUN` crée une nouvelle couche dans l'image.
- **Syntaxe** : `RUN commande`
- **Exemple** : `RUN apt-get update && apt-get install -y nginx`

> **Bonnes pratiques** : Combinez les commandes `RUN` pour réduire le nombre de couches et la taille de l'image.

#### COPY

- **Description** : Copie des fichiers ou des répertoires depuis le contexte de construction de l'hôte vers le système de fichiers de l'image.
- **Syntaxe** : `COPY [options] source destination`
- **Exemple** : `COPY . /app`

> **Remarque** : Utilisez `COPY` pour copier des fichiers locaux. Préférez `COPY` à `ADD` lorsque nous n'avons pas besoin de fonctionnalités supplémentaires.

#### ADD

- **Description** : Semblable à `COPY`, mais avec des fonctionnalités supplémentaires comme le décompactage automatique des archives et le téléchargement depuis des URL.
- **Syntaxe** : `ADD source destination`
- **Exemple** : `ADD https://example.com/file.tar.gz /app`

> **Attention** : Utilisez `ADD` avec précaution, car ses fonctionnalités supplémentaires peuvent entraîner des comportements inattendus.

#### WORKDIR

- **Description** : Définit le répertoire de travail pour les instructions suivantes (`RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD`).
- **Syntaxe** : `WORKDIR /chemin/du/répertoire`
- **Exemple** : `WORKDIR /app`

> **Avantage** : Améliore la lisibilité et évite l'utilisation de chemins absolus.

#### EXPOSE

- **Description** : Indique le port sur lequel le conteneur écoutera au moment de l'exécution.
- **Syntaxe** : `EXPOSE port`
- **Exemple** : `EXPOSE 80`

> **Note** : `EXPOSE` est informatif et n'ouvre pas réellement le port. Pour publier le port, utilisez `-p` lors du `docker run`.

#### VOLUME

- **Description** : Crée un point de montage pour un volume, permettant de stocker des données persistantes sur la machine hôte.
- **Syntaxe** : `VOLUME ["/chemin/du/volume"]`
- **Exemple** : `VOLUME ["/data"]`

> **Utilisation** : Les volumes sont essentiels pour conserver les données entre les exécutions de conteneurs.

#### USER

- **Description** : Spécifie l'utilisateur qui exécutera les commandes suivantes dans le Dockerfile.
- **Syntaxe** : `USER utilisateur[:groupe]`
- **Exemple** : `USER appuser`

> **Sécurité** : Exécutez les applications avec un utilisateur non privilégié pour renforcer la sécurité.

#### ENTRYPOINT

- **Description** : Définit la commande qui sera exécutée lorsque le conteneur démarre. Elle n'est pas facilement surchargeable lors de l'exécution.
- **Syntaxe** :
  - Forme shell : `ENTRYPOINT commande param1 param2`
  - Forme exec : `ENTRYPOINT ["exécutable", "param1", "param2"]`
- **Exemple** : `ENTRYPOINT ["nginx"]`

> **Usage** : Utilisez `ENTRYPOINT` pour définir le processus principal du conteneur.

#### CMD

- **Description** : Fournit des arguments par défaut pour `ENTRYPOINT` ou spécifie la commande à exécuter si `ENTRYPOINT` n'est pas défini.
- **Syntaxe** :
  - Forme shell : `CMD commande param1 param2`
  - Forme exec : `CMD ["exécutable", "param1", "param2"]`
- **Exemple** : `CMD ["-g", "daemon off;"]`

> **Remarque** : Si des arguments sont passés lors du `docker run`, ils remplaceront ceux définis dans `CMD`.

#### LABEL

- **Description** : Ajoute des métadonnées à l'image sous forme de paires clé-valeur.
- **Syntaxe** : `LABEL clé="valeur"`
- **Exemple** : `LABEL maintainer="email@example.com"`

> **Utilité** : Les labels sont utiles pour la gestion, l'automatisation et la documentation des images.

#### ONBUILD

- **Description** : Spécifie une instruction à exécuter lorsque l'image sert de base pour une autre image.
- **Syntaxe** : `ONBUILD [instruction]`
- **Exemple** : `ONBUILD COPY . /app/src`

> **Cas d'usage** : Idéal pour les images parent destinées à être étendues.

#### HEALTHCHECK

- **Description** : Vérifie régulièrement si le conteneur fonctionne correctement.
- **Syntaxe** :
  ```
  HEALTHCHECK [options] CMD commande
  HEALTHCHECK NONE
  ```
- **Exemple** :
  ```
  HEALTHCHECK --interval=30s --timeout=5s CMD curl -f http://localhost/ || exit 1
  ```

> **Avantage** : Permet de surveiller l'état du conteneur et de réagir en cas de problème.

### Bonnes Pratiques

- **Minimisez le nombre de couches** : Combinez les instructions `RUN` lorsque c'est possible.
- **Nettoyez après l'installation** : Supprimez les fichiers temporaires et les caches.
- **Évitez de stocker des secrets** : N'incluez pas de mots de passe ou de clés dans le Dockerfile.
- **Spécifiez les versions** : Utilisez des tags de version pour les images de base et les dépendances.
- **Utilisez `.dockerignore`** : Excluez les fichiers inutiles du contexte de construction.

## Exemples

### Exemple 1 : Serveur Web Nginx avec une Page Personnalisée

Un Dockerfile qui crée une image pour exécuter un serveur Nginx avec une page `index.html` personnalisée :

```dockerfile
# Utiliser l'image de base Nginx
FROM nginx:latest

# Copier la page index.html dans le répertoire par défaut de Nginx
COPY index.html /usr/share/nginx/html/

# Exposer le port 80
EXPOSE 80

# Définir le point d'entrée et les arguments par défaut
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

- **Explication** :
  - `FROM` indique que nous utilisons l'image officielle de Nginx.
  - `COPY` copie notre page `index.html` dans le conteneur.
  - `EXPOSE` documente le port utilisé.
  - `ENTRYPOINT` définit Nginx comme le processus principal.
  - `CMD` fournit les arguments pour exécuter Nginx en premier plan.

> **Remarque** : Si nous exécutons `docker run` avec des arguments supplémentaires, ceux-ci remplaceront `CMD`, mais `ENTRYPOINT` sera toujours exécuté.

### Exemple 2 : Application Python Simple

Un Dockerfile pour une application Python :

```dockerfile
# Image de base Python
FROM python:3.9-slim

# Définir le répertoire de travail
WORKDIR /app

# Copier le fichier requirements.txt et installer les dépendances
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copier le reste du code de l'application
COPY . .

# Exposer le port de l'application
EXPOSE 5000

# Définir la commande par défaut
CMD ["python", "app.py"]
```

- **Explication** :
  - Utilisation d'une image Python légère.
  - Installation des dépendances.
  - Copie du code source.
  - Démarrage de l'application avec `python app.py`.
### Exemple 3 : Serveur Web Nginx Utilisant un Volume pour la Page HTML

Voici un Dockerfile qui crée une image pour exécuter un serveur Nginx, mais au lieu de copier la page `index.html` dans l'image comme dans l'exemple 1, il définit un **volume** pour utiliser une page `index.html` présente sur l'hôte. Cela permet de modifier le contenu du site web sans avoir à reconstruire l'image Docker à chaque changement.

### Dockerfile

```dockerfile
# Utiliser l'image de base Nginx
FROM nginx:latest

# Exposer le port 80
EXPOSE 80

# Définir un volume pour le répertoire HTML de Nginx
VOLUME ["/usr/share/nginx/html"]

# Définir le point d'entrée et les arguments par défaut
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

### Explication :

- **`FROM`** : Indique que nous utilisons l'image officielle de Nginx.
- **`EXPOSE`** : Documente le port 80 utilisé par Nginx.
- **`VOLUME`** : Définit `/usr/share/nginx/html` comme point de montage pour un volume. Cela signifie que ce répertoire dans le conteneur est destiné à être monté avec des données provenant de l'extérieur du conteneur.
- **`ENTRYPOINT`** : Définit Nginx comme le processus principal à exécuter lors du démarrage du conteneur.
- **`CMD`** : Fournit les arguments par défaut pour exécuter Nginx en premier plan.

### Utilisation du Dockerfile

#### Étape 1 : Construire l'Image

Construisons l'image à partir du Dockerfile en utilisant la commande suivante :

```bash
docker build -t mon_nginx_volume .
```

- **`-t mon_nginx_volume`** : Attribue le tag `mon_nginx_volume` à l'image construite.
- Le `.` indique que le Dockerfile est dans le répertoire courant.

#### Étape 2 : Préparer le Répertoire sur l'Hôte

Assurons-nous d'avoir un fichier `index.html` dans un répertoire sur votre machine hôte. Par exemple, créeons un répertoire `site_web` et plaçons-y notre `index.html` :

```bash
mkdir ~/site_web
echo "<h1>Bienvenue sur mon site web via un volume Docker !</h1>" > ~/site_web/index.html
```

#### Étape 3 : Lancer le Conteneur en Montant le Volume

Lançons un conteneur à partir de l'image en montant le répertoire de l'hôte dans le conteneur :

```bash
docker run -d -p 8080:80 --name mon_serveur_nginx -v ~/site_web:/usr/share/nginx/html mon_nginx_volume
```

- **`-d`** : Exécute le conteneur en arrière-plan (mode détaché).
- **`-p 8080:80`** : Mappe le port 80 du conteneur au port 8080 de l'hôte.
- **`--name mon_serveur_nginx`** : Donne le nom `mon_serveur_nginx` au conteneur.
- **`-v ~/site_web:/usr/share/nginx/html`** : Monte le répertoire `~/site_web` de l'hôte dans `/usr/share/nginx/html` du conteneur.
- **`mon_nginx_volume`** : Utilise l'image que nous venons de construire.

#### Étape 4 : Accéder au Site Web

Ouvrons un navigateur web et accédez à `http://localhost:8080` pour voir votre page `index.html` servie par Nginx depuis le volume monté.

### Avantages de cette Approche

- **Flexibilité** : Nous pouvons modifier les fichiers HTML sur l'hôte sans avoir besoin de reconstruire l'image ou de redémarrer le conteneur.
- **Développement Simplifié** : Idéal pour le développement, car les modifications sont immédiatement reflétées dans le conteneur.
- **Gestion des Données** : Les données sont stockées sur l'hôte, ce qui facilite les sauvegardes et la persistance des données.

### Points Importants

- **VOLUME dans le Dockerfile** : L'instruction `VOLUME` indique que le répertoire est destiné à être monté en tant que volume. Cependant, même sans cette instruction, vous pouvez monter un volume lors du `docker run` avec l'option `-v`.
- **Permissions** : Assurons-nous que les permissions du répertoire et des fichiers sur l'hôte permettent à Nginx dans le conteneur de lire les fichiers.
- **Isolation** : Les modifications apportées aux fichiers sur l'hôte sont immédiatement visibles dans le conteneur, ce qui peut ne pas être souhaitable en production. Dans un environnement de production, vous pourriez préférer copier les fichiers dans l'image pour garantir la cohérence.

## Comprendre ENTRYPOINT et CMD

- **`ENTRYPOINT`** : Définit le programme qui sera exécuté lorsque le conteneur démarre. Il est fixe et ne peut pas être facilement remplacé lors du `docker run`.
- **`CMD`** : Fournit les arguments par défaut à `ENTRYPOINT`. Si des arguments sont passés lors du `docker run`, ils remplaceront ceux de `CMD`.

### Surcharge de CMD et ENTRYPOINT

- **Avec CMD uniquement** :
  - `docker run mon_image` exécutera la commande définie dans `CMD`.
  - `docker run mon_image commande` remplacera la commande dans `CMD` par `commande`.

- **Avec ENTRYPOINT et CMD** :
  - `docker run mon_image` exécutera `ENTRYPOINT` avec les arguments de `CMD`.
  - `docker run mon_image argument` exécutera `ENTRYPOINT` avec `argument`, remplaçant `CMD`.

- **Remplacer ENTRYPOINT** :
  - Utilisez `--entrypoint` lors du `docker run` :
    ```bash
    docker run --entrypoint /bin/bash mon_image
    ```

## Inspection d'une Image

Pour vérifier si une image a un `ENTRYPOINT` ou un `CMD`, utilisons :

- **`docker inspect`** :
  ```bash
  docker inspect mon_image:tag
  ```
  Recherchons les sections `Config.Entrypoint` et `Config.Cmd`.

- **`docker history`** :
  ```bash
  docker history mon_image:tag
  ```
  Affiche l'historique des instructions utilisées pour construire l'image.

## Pratique

1. **Créez un fichier Dockerfile** :
   - Dans le répertoire de notre application, créez un fichier nommé `Dockerfile`.

2. **Écrivez les instructions pour construire une image personnalisée** :
   - Utilisez les mots-clés appropriés (`FROM`, `COPY`, `RUN`, etc.) en suivant les bonnes pratiques.

3. **Construisez l'image Docker** :
   ```bash
   docker build -t mon_image:version .
   ```
   - Le `.` indique que le contexte de construction est le répertoire courant.

4. **Exécutez un conteneur à partir de l'image personnalisée** :
   ```bash
   docker run -d -p 8080:80 --name mon_conteneur mon_image:version
   ```
   - `-d` exécute le conteneur en arrière-plan.
   - `-p` mappe le port 80 du conteneur au port 8080 de l'hôte.
   - `--name` donne un nom au conteneur.

5. **Vérifiez que le conteneur fonctionne** :
   - Accédez à `http://localhost:8080` dans un navigateur web.

## Exemples
Un exemple de Dockerfile qui crée une image de conteneur qui exécute un serveur web Nginx sur une page index.html présente sur la machine hôte.
```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```
Le conteneur démarrera nginx, car c'est le point d'entrée tout en lui passant les paramètres définis dans CMD.

Si l'on passe des arguments lors du "docker run", ils seront mis à la place de CMD. CMD joue le rôle de paramètres par défaut.

Toutefois ENTRYPOINT n'est pas surchargeable et sera toujours appelé.
Si l'on veut pouvoir surcharger alors il ne faut pas utiliser d'ENTRYPOINT.
Pour savoir si une image a un ENTRYPOINT ou un CMD, il faut alors faire :
```bash
docker history MONIMAGE:Version
```
ou alors
```bash
docker inspect MONIMAGE:Version
```

## Pratique
-   Créez un fichier Dockerfile
-   Utilisez les instructions pour construire une image personnalisée
-   Exécutez un conteneur à partir de l'image personnalisée

## Fonctionnement des Couches

### Système de Fichiers en Couches

Docker utilise un système de fichiers empilable, tel que **OverlayFS**, **AUFS**, ou **UnionFS**, pour gérer les couches. Ce système permet de superposer plusieurs systèmes de fichiers en lecture seule (les couches) et un système de fichiers en lecture-écriture (le conteneur en cours d'exécution).

- **Couches en Lecture Seule** : Chaque couche de l'image est en lecture seule et correspond à une instruction du Dockerfile.
- **Couches en Lecture-Écriture** : Lorsqu'un conteneur est lancé à partir d'une image, une couche supérieure en lecture-écriture est ajoutée. Toutes les modifications effectuées par le conteneur sont écrites dans cette couche.

### Processus de Construction

1. **Instruction FROM** : Cette instruction importe une image de base, qui elle-même est composée de couches préexistantes.
2. **Instructions Successives** : Chaque instruction suivante (`RUN`, `COPY`, `ADD`, etc.) ajoute une nouvelle couche au-dessus des couches existantes.
3. **Empilement des Couches** : Les couches sont empilées dans l'ordre des instructions, de la plus ancienne (en bas) à la plus récente (en haut).

### Cache de Construction

Docker met en cache les couches lors de la construction des images. Si une couche n'a pas changé depuis la dernière construction, Docker réutilise le cache pour accélérer le processus.

- **Avantage** : Réduction significative du temps de construction.
- **Condition** : Les instructions précédentes doivent être identiques, y compris le contenu des fichiers copiés.

## Impact des Couches sur les Images et les Conteneurs

### Taille des Images

- **Redondance Minimisée** : Les couches permettent de minimiser la duplication des données. Si plusieurs images partagent les mêmes couches, ces couches sont stockées une seule fois sur le disque.
- **Optimisation** : En structurant judicieusement le Dockerfile, nous pouvons réduire la taille de l'image en limitant le nombre de couches et en évitant d'ajouter des fichiers inutiles.

### Partage et Distribution

- **Efficacité du Réseau** : Lors du téléchargement d'une image, si certaines couches sont déjà présentes sur l'hôte (parce qu'elles sont partagées avec d'autres images), Docker ne télécharge que les couches manquantes.
- **Reproductibilité** : Les couches immuables garantissent que l'image est identique sur tous les systèmes, assurant la cohérence entre les environnements.

## Exemples Illustratifs

### Exemple de Dockerfile

```dockerfile
# Étape 1 : Image de base
FROM ubuntu:20.04

# Étape 2 : Installation des paquets
RUN apt-get update && apt-get install -y python3

# Étape 3 : Ajout du code source
COPY app.py /app/app.py

# Étape 4 : Configuration du répertoire de travail
WORKDIR /app

# Étape 5 : Exécution de l'application
CMD ["python3", "app.py"]
```

#### Analyse des Couches

1. **Couches de l'Image de Base** : L'image `ubuntu:20.04` est composée de plusieurs couches correspondant à sa propre construction.
2. **Instruction RUN** : Crée une nouvelle couche qui contient les modifications du système de fichiers résultant de l'installation de Python 3.
3. **Instruction COPY** : Ajoute une couche avec le fichier `app.py`.
4. **Instruction WORKDIR** : Modifie le répertoire de travail, mais ne crée pas de nouvelle couche de système de fichiers (les métadonnées sont mises à jour).
5. **Instruction CMD** : N'affecte pas le système de fichiers et ne crée pas de nouvelle couche.

### Visualisation des Couches

Nous pouvons visualiser les couches d'une image en utilisant la commande :

```bash
docker history mon_image:tag
```

Cette commande affiche la liste des couches avec les instructions associées et la taille de chaque couche.

## Optimisation des Couches

### Combinaison des Instructions RUN

Pour réduire le nombre de couches et la taille de l'image, il est recommandé de combiner les instructions `RUN` lorsque c'est possible.

#### Exemple :

**Avant Optimisation :**

```dockerfile
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get clean
```

**Après Optimisation :**

```dockerfile
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean
```

- **Avantage** : Toutes les commandes sont exécutées dans une seule couche, réduisant la taille et évitant la duplication des caches.

### Suppression des Fichiers Temporaires

Assurons-nous de supprimer les fichiers temporaires et les caches après l'installation des paquets.

#### Exemple :

```dockerfile
RUN apt-get update && \
    apt-get install -y package && \
    rm -rf /var/lib/apt/lists/*
```

### Utilisation du .dockerignore

Le fichier `.dockerignore` fonctionne de manière similaire à `.gitignore`. Il exclut les fichiers et répertoires du contexte de construction, évitant ainsi de les ajouter aux couches de l'image.

#### Exemple de .dockerignore :

```
*.log
node_modules/
.git
```

- **Avantage** : Réduction de la taille du contexte de construction et de l'image finale.

## Partage de Couches Entre Images

Les couches sont stockées de manière globale sur l'hôte Docker. Si plusieurs images utilisent les mêmes couches (par exemple, en utilisant la même image de base), ces couches sont stockées une seule fois.

- **Efficacité du Stockage** : Économie d'espace disque en évitant la duplication.
- **Efficacité du Réseau** : Lors du téléchargement, seules les nouvelles couches sont transférées.

## Mécanisme d'Union des Systèmes de Fichiers

Le système de fichiers en couches utilise un mécanisme d'union pour présenter une vue unifiée des différentes couches.

- **Lecture** : Les fichiers sont lus à partir de la couche la plus haute qui les contient.
- **Écriture** : Toute modification est écrite dans la couche supérieure en lecture-écriture (du conteneur en cours d'exécution).
- **Suppression** : La suppression d'un fichier est enregistrée comme une opération spéciale dans la couche supérieure, masquant le fichier dans les couches inférieures.

## Impact sur les Conteneurs

### Modifications en Exécution

Les modifications effectuées dans un conteneur en cours d'exécution n'affectent pas les couches de l'image. Elles sont stockées dans la couche en lecture-écriture du conteneur.

- **Éphémère** : Par défaut, lorsque le conteneur est supprimé, les modifications sont perdues.
- **Persistance** : Pour conserver les données, utilisez des volumes Docker ou des montages de répertoires.

### Création de Nouvelles Images

Nous pouvons créer une nouvelle image à partir d'un conteneur modifié en utilisant la commande :

```bash
docker commit mon_conteneur nouvelle_image
```

- **Limitation** : Cette pratique est déconseillée pour les workflows automatisés, car elle rend le processus moins reproductible.

## Bonnes Pratiques

- **Ordre des Instructions** : Placez les instructions qui changent rarement au début du Dockerfile pour maximiser l'utilisation du cache.
- **Minimiser les Couches** : Combinez les instructions lorsque c'est possible, mais pas au détriment de la lisibilité.
- **Nettoyage** : Supprimez les fichiers inutiles et les caches pour réduire la taille des couches.
- **Images Légères** : Utilisez des images de base minimalistes (comme `alpine` ou `slim`) si cela convient à notre application.
- **Éviter les Redondances** : Ne copiez que les fichiers nécessaires dans l'image.
- **Utiliser .dockerignore** : Excluez les fichiers et dossiers non pertinents du contexte de construction.

## Cas Pratiques

### Cas 1 : Modification du Code Source

Si nous modifions fréquemment le code source de notre application, plaçons l'instruction `COPY` ou `ADD` du code le plus tard possible dans le Dockerfile. Cela permet de réutiliser le cache pour les couches précédentes.

#### Exemple :

```dockerfile
# Instructions pour installer les dépendances
RUN apt-get update && apt-get install -y ...

# Copier les fichiers de configuration
COPY config/ /app/config/

# Copier le code source
COPY src/ /app/src/
```

### Cas 2 : Changement de Dépendances

Si nous modifions souvent les dépendances (par exemple, le fichier `requirements.txt` pour Python), copions d'abord ce fichier et installons les dépendances avant de copier le reste du code.

#### Exemple :

```dockerfile
# Copier le fichier des dépendances
COPY requirements.txt /app/

# Installer les dépendances
RUN pip install -r /app/requirements.txt

# Copier le reste du code
COPY . /app/
```

# L'Impact des Couches sur la Sécurité des Images Docker

Le mécanisme des couches (*layers*) dans Docker joue un rôle significatif dans la **sécurité** des applications conteneurisées. Comprendre comment les couches affectent la sécurité des images Docker est essentiel pour éviter les vulnérabilités et assurer la protection de nos systèmes.

## Introduction

Dans Docker, une image est construite à partir d'une série d'instructions spécifiées dans un **Dockerfile**. Chaque instruction crée une nouvelle couche, qui est empilée sur les couches précédentes. Ces couches sont immuables et forment ensemble le système de fichiers de l'image. Bien que ce mécanisme offre des avantages en termes de réutilisation et d'efficacité, il peut introduire des risques de sécurité s'il n'est pas géré correctement.

## Risques de Sécurité Associés aux Couches

### 1. **Propagation des Vulnérabilités**

- **Images de Base Vulnérables** : Si nous utilisons une image de base contenant des vulnérabilités connues, toutes les images dérivées héritent de ces failles. Par exemple, une image basée sur une version obsolète d'Ubuntu peut être exposée à des vulnérabilités critiques.

- **Impact Multiplié** : Les vulnérabilités dans les couches inférieures peuvent affecter un grand nombre d'images et de conteneurs, amplifiant l'impact potentiel d'une faille de sécurité.

### 2. **Inclusion Accidentelle de Données Sensibles**

- **Persistences des Données** : Si des secrets (clés, mots de passe, certificats) sont ajoutés dans une couche et supprimés dans une couche ultérieure, ils restent présents dans l'historique des couches. Un attaquant pourrait extraire ces informations en inspectant les couches.

- **Difficulté de Suppression** : Une fois qu'une donnée sensible est incluse dans une couche, il est difficile de l'éliminer complètement sans reconstruire l'image depuis le début.

### 3. **Surface d'Attaque Élargie**

- **Taille des Images** : Des images volumineuses avec de nombreuses couches peuvent inclure des composants inutiles, augmentant ainsi la surface d'attaque.

- **Composants Obsolètes** : Les couches contenant des paquets ou des dépendances non mis à jour peuvent introduire des vulnérabilités.

### 4. **Cache de Construction Non Mis à Jour**

- **Réutilisation du Cache** : Docker utilise le cache pour accélérer la construction des images. Si le cache contient des versions obsolètes de paquets, les mises à jour de sécurité peuvent être omises.

- **Instructions RUN** : Les commandes `RUN` qui installent des paquets sans forcer la mise à jour peuvent entraîner l'inclusion de versions vulnérables.

### 5. **Isolation Incomplète**

- **Permissions Excessives** : Si les couches ne sont pas configurées pour exécuter des processus avec des utilisateurs non privilégiés, les conteneurs peuvent être exécutés avec des privilèges root, augmentant les risques en cas de compromission.

## Bonnes Pratiques pour Renforcer la Sécurité des Couches

### 1. **Utiliser des Images de Base Sécurisées**

- **Sources Fiables** : Utilisez des images officielles ou vérifiées provenant de dépôts de confiance.

- **Images Minimisées** : Préférez des images légères comme `alpine` ou `scratch` pour réduire le nombre de composants potentiellement vulnérables.

### 2. **Maintenir les Images à Jour**

- **Mises à Jour Régulières** : Reconstruisez régulièrement nos images pour inclure les dernières mises à jour de sécurité.

- **Tags Spécifiques** : Évitez d'utiliser le tag `latest` et spécifiez des versions précises pour mieux contrôler les mises à jour.

### 3. **Minimiser le Nombre de Couches**

- **Combiner les Commandes** : Combinez les instructions `RUN` pour réduire le nombre de couches et éviter la duplication des données.

- **Nettoyage** : Supprimez les fichiers temporaires et les caches dans la même instruction `RUN` pour empêcher leur inclusion dans des couches antérieures.

### 4. **Gérer les Secrets de Manière Sécurisée**

- **Éviter d'Inclure des Secrets** : Ne jamais inclure de secrets ou de données sensibles dans le Dockerfile ou les arguments de construction.

- **Utiliser des Outils Sécurisés** : Employez des solutions comme Docker Secrets ou des variables d'environnement injectées au moment de l'exécution.

### 5. **Analyser les Images pour les Vulnérabilités**

- **Outils de Scan** : Utilisez des outils d'analyse tels que **Trivy**, **Clair**, ou **Anchore** pour détecter les vulnérabilités dans nos images.

- **Intégration Continue** : Intégrez ces analyses dans notre pipeline CI/CD pour détecter les problèmes le plus tôt possible.

### 6. **Exécuter en Tant qu'Utilisateur Non Privilégié**

- **Instruction USER** : Utilisez `USER` pour spécifier un utilisateur non-root pour l'exécution des processus dans le conteneur.

- **Réduction des Privilèges** : Limitez les permissions au strict nécessaire pour l'application.

### 7. **Utiliser le Fichier .dockerignore**

- **Exclusion de Fichiers** : Créez un fichier `.dockerignore` pour exclure les fichiers et dossiers non nécessaires, tels que les clés privées, les fichiers de configuration sensibles, ou les répertoires de build locaux.

### 8. **Gérer le Cache de Construction**

- **Forcer les Mises à Jour** : Incluez `apt-get update` et `apt-get install` dans la même instruction `RUN` pour éviter l'utilisation du cache.

- **Invalidation du Cache** : Ajoutez des arguments ou des variables qui changent régulièrement pour invalider le cache lorsque c'est nécessaire.

### 9. **Vérifier les Sommes de Contrôle**

- **Intégrité des Téléchargements** : Lorsque nous téléchargeons des fichiers externes, vérifions les sommes de contrôle pour nous assurer qu'ils n'ont pas été compromis.

- **Utiliser HTTPS** : Téléchargez toujours les dépendances via des connexions sécurisées.

## Exemples Concrets

### Inclusion de Données Sensibles

**Dockerfile Non Sécurisé :**

```dockerfile
FROM python:3.9

# Mauvaise pratique : inclusion d'un fichier de clés
COPY id_rsa /root/.ssh/id_rsa

# Suppression du fichier de clés
RUN rm /root/.ssh/id_rsa

CMD ["python", "app.py"]
```

**Problème :** La clé privée `id_rsa` est toujours présente dans une couche précédente et peut être récupérée.

**Solution :**

- **Ne pas inclure de secrets dans le Dockerfile.**
- **Utiliser des volumes ou des variables d'environnement pour injecter les secrets au moment de l'exécution.**

### Gestion Correcte des Mises à Jour

**Dockerfile Vulnérable :**

```dockerfile
FROM ubuntu:20.04

RUN apt-get update
RUN apt-get install -y openssl
```

**Problème :** Si le cache est utilisé, `apt-get update` peut ne pas récupérer les dernières informations, laissant des paquets obsolètes.

**Dockerfile Sécurisé :**

```dockerfile
FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y openssl && \
    rm -rf /var/lib/apt/lists/*
```

**Avantages :**

- **Assure que les dernières mises à jour sont installées.**
- **Réduit la taille de l'image en nettoyant les listes de paquets.**

## Impact sur la Chaîne d'Approvisionnement

- **Effet Domino** : Une vulnérabilité dans une couche de base peut affecter toutes les images dérivées, compromettant potentiellement l'ensemble de la chaîne d'approvisionnement logicielle.

- **Importance de la Vérification** : Il est essentiel de vérifier régulièrement les images de base pour détecter les vulnérabilités et les mettre à jour.

## Conclusion

Les couches Docker sont un élément fondamental de la construction des images, mais elles nécessitent une attention particulière en matière de sécurité. En adoptant des pratiques sécurisées lors de la création et de la gestion des couches, nous pouvons :

- **Réduire les Risques de Vulnérabilités** : En utilisant des images de base sécurisées et en maintenant nos images à jour.

- **Protéger les Données Sensibles** : En évitant d'inclure des secrets dans les couches et en gérant les configurations de manière sécurisée.

- **Optimiser la Sécurité** : En minimisant la surface d'attaque grâce à des images légères et à un nombre réduit de couches.

- **Assurer la Conformité** : En intégrant des scans de sécurité et des mises à jour régulières dans nos processus de développement.

## Actions Recommandées

- **Audit Régulier** : Mettez en place des audits réguliers de nos images pour identifier les vulnérabilités potentielles.

- **Formation** : Sensibilisez les développeurs et les équipes DevOps aux meilleures pratiques de sécurité liées aux couches Docker.

- **Automatisation** : Intégrez des outils d'analyse de sécurité dans nos pipelines CI/CD pour détecter et corriger rapidement les problèmes.

- **Documentation** : Maintenez une documentation à jour des images utilisées, des versions, et des modifications apportées aux Dockerfiles.

# Les Différences entre les Systèmes de Fichiers en Couches Utilisés par Docker : OverlayFS, AUFS, etc.

## Introduction

Docker utilise des **systèmes de fichiers en couches** pour gérer efficacement les images et les conteneurs. Ces systèmes permettent de superposer plusieurs systèmes de fichiers pour créer un système de fichiers unique, optimisant ainsi l'utilisation de l'espace disque et facilitant la gestion des images. Parmi les systèmes de fichiers en couches utilisés par Docker, les plus courants sont **OverlayFS**, **AUFS**, **Device Mapper**, **Btrfs** et **ZFS**. Chacun de ces systèmes a ses propres caractéristiques, avantages et inconvénients. Dans cette section, nous allons explorer en détail les différences entre ces systèmes, en mettant l'accent sur OverlayFS et AUFS.

## Pourquoi Docker Utilise des Systèmes de Fichiers en Couches

Les systèmes de fichiers en couches offrent plusieurs avantages clés pour Docker :

- **Efficacité du Stockage** : Évitent la duplication des données en réutilisant les couches communes entre les images.
- **Rapidité de Déploiement** : Permettent de télécharger et de charger uniquement les couches modifiées ou manquantes.
- **Isolation** : Fournissent un système de fichiers isolé pour chaque conteneur, améliorant la sécurité et la stabilité.
- **Flexibilité** : Facilitent la création et la gestion des images en empilant des modifications successives.

## Présentation des Systèmes de Fichiers en Couches

### AUFS (Advanced Multi-Layered Unification Filesystem)

- **Origine** : AUFS est un système de fichiers en couches pour Linux, initialement développé pour le projet Knoppix.
- **Caractéristiques** :
  - Permet de superposer plusieurs systèmes de fichiers en lecture seule avec un système de fichiers en lecture-écriture.
  - Supporte un grand nombre de couches (généralement jusqu'à 127 ou plus).
- **Support du Noyau** :
  - **Non inclus** dans le noyau Linux officiel.
  - Nécessite des patchs ou des modules externes pour être utilisé.
- **Utilisation dans Docker** :
  - Était le système de fichiers par défaut dans les premières versions de Docker.
  - Offrait une flexibilité et une compatibilité avec les noyaux plus anciens.

### OverlayFS

- **Origine** : OverlayFS est un système de fichiers en couches plus récent, conçu pour être simple et efficace.
- **Caractéristiques** :
  - Superpose une couche supérieure en lecture-écriture sur une couche inférieure en lecture seule.
  - Initialement limité à deux couches, mais le support pour plus de couches a été ajouté dans les versions ultérieures.
- **Support du Noyau** :
  - **Inclus** dans le noyau Linux principal depuis la version **3.18**.
  - Bénéficie du support de la communauté du noyau Linux.
- **Utilisation dans Docker** :
  - Est devenu le pilote de stockage par défaut dans les versions plus récentes de Docker.
  - Préféré en raison de sa performance et de sa simplicité.

## Comparaison entre AUFS et OverlayFS

### Support et Compatibilité

- **AUFS** :
  - Non intégré dans le noyau Linux officiel.
  - Peut nécessiter des distributions spécifiques ou des configurations personnalisées.
  - Moins de support à long terme en raison de sa dépendance aux mainteneurs externes.

- **OverlayFS** :
  - Intégré dans le noyau Linux officiel depuis la version 3.18.
  - Largement disponible sur les distributions modernes.
  - Bénéficie des mises à jour et du support de la communauté Linux.

### Performance

- **AUFS** :
  - Bonne performance, mais peut être affecté par sa complexité.
  - La gestion d'un grand nombre de couches peut entraîner une légère dégradation des performances.

- **OverlayFS** :
  - Performance généralement supérieure en raison de sa simplicité et de son intégration native.
  - Meilleure gestion du cache, ce qui améliore les opérations de lecture/écriture.

### Complexité et Maintenance

- **AUFS** :
  - Plus complexe à configurer et à maintenir.
  - La nécessité de patcher le noyau peut entraîner des problèmes de compatibilité.

- **OverlayFS** :
  - Simplicité de configuration.
  - Maintenance facilitée grâce à son inclusion dans le noyau principal.

### Nombre de Couches Supportées

- **AUFS** :
  - Supporte un grand nombre de couches (127 ou plus), ce qui peut être avantageux pour des images complexes.

- **OverlayFS** :
  - Initialement limité à deux couches, mais ce nombre a été augmenté dans les versions ultérieures.
  - Docker utilise des techniques pour combiner les couches et surmonter cette limitation.

### Disponibilité et Adoption

- **AUFS** :
  - Moins courant sur les distributions récentes.
  - Utilisé principalement lorsqu'OverlayFS n'est pas disponible.

- **OverlayFS** :
  - Devenu le standard de facto pour Docker sur Linux.
  - Large adoption dans la communauté en raison de ses avantages.

## Autres Systèmes de Fichiers en Couches

### Device Mapper

- **Fonctionnement** :
  - Utilise le sous-système Device Mapper du noyau Linux.
  - Crée des volumes logiques en utilisant LVM (Logical Volume Manager).

- **Avantages** :
  - Peut être utilisé sur des systèmes de fichiers qui ne supportent pas AUFS ou OverlayFS.
  - Offre des fonctionnalités avancées comme la gestion fine des volumes.

- **Inconvénients** :
  - Configuration plus complexe.
  - Performances moins bonnes dans certains cas par rapport à OverlayFS.

### Btrfs

- **Fonctionnement** :
  - Utilise les sous-volumes et les instantanés de Btrfs pour gérer les couches.

- **Avantages** :
  - Fonctionnalités avancées comme la compression, la déduplication, les instantanés.

- **Inconvénients** :
  - Stabilité variable selon les versions du noyau.
  - Nécessite que le système de fichiers de l'hôte soit Btrfs.

### ZFS

- **Fonctionnement** :
  - Utilise les capacités de ZFS pour la gestion des couches.

- **Avantages** :
  - Haute fiabilité, intégrité des données, fonctionnalités avancées.

- **Inconvénients** :
  - Non inclus dans le noyau Linux principal pour des raisons de licence.
  - Installation et configuration supplémentaires nécessaires.

## Historique et Évolution

- **Début avec AUFS** :
  - Docker a initialement choisi AUFS en raison de son support des systèmes de fichiers en couches et de sa disponibilité à l'époque.
  - Les limitations du noyau Linux ont rendu AUFS nécessaire malgré son absence du noyau principal.

- **Transition vers OverlayFS** :
  - Avec l'intégration d'OverlayFS dans le noyau Linux 3.18, Docker a commencé à le supporter.
  - OverlayFS est devenu le choix par défaut en raison de ses avantages en termes de performance et de maintenance.

- **Évolution des Besoins** :
  - Les améliorations apportées à OverlayFS ont permis de lever les limitations initiales (comme le nombre de couches).
  - La communauté a largement adopté OverlayFS, ce qui a renforcé son support et son développement.

## Facteurs à Considérer lors du Choix d'un Système de Fichiers

- **Compatibilité du Noyau** :
  - Assurons-nous que notre version du noyau Linux supporte le système de fichiers choisi.

- **Performance** :
  - OverlayFS offre généralement de meilleures performances pour la plupart des cas d'utilisation.

- **Complexité de Configuration** :
  - OverlayFS est plus simple à configurer que AUFS ou Device Mapper.

- **Fonctionnalités Spécifiques** :
  - Si nous avons besoin de fonctionnalités avancées (compression, déduplication), Btrfs ou ZFS peuvent être appropriés.

- **Stabilité et Support à Long Terme** :
  - Préférez les systèmes de fichiers inclus dans le noyau principal pour bénéficier du support continu.

## Recommandations Actuelles

- **OverlayFS** :
  - Recommandé pour la plupart des déploiements Docker sur Linux modernes.
  - Offre un bon équilibre entre performance, simplicité et support.

- **AUFS** :
  - Peut être utilisé sur des systèmes plus anciens où OverlayFS n'est pas disponible.
  - Moins recommandé en raison de sa complexité et de son manque de support dans le noyau principal.

- **Device Mapper** :
  - Utile dans des environnements spécifiques ou sur des systèmes de fichiers non supportés par OverlayFS.
  - Nécessite une configuration plus avancée.

- **Btrfs et ZFS** :
  - À considérer si nous avons des besoins spécifiques en termes de fonctionnalités de systèmes de fichiers.
  - Exigent que le système de fichiers de l'hôte soit compatible.

## Conclusion

Les différences entre les systèmes de fichiers en couches utilisés par Docker, tels que OverlayFS, AUFS, Device Mapper, Btrfs et ZFS, résident principalement dans leur intégration au noyau Linux, leurs performances, leur complexité et leurs fonctionnalités. **OverlayFS** est généralement préféré aujourd'hui en raison de sa simplicité, de ses bonnes performances et de son inclusion dans le noyau Linux principal. **AUFS**, bien qu'ayant été crucial dans les premières étapes de Docker, est moins utilisé aujourd'hui.

Le choix du système de fichiers dépendra de plusieurs facteurs :

- **Version du Noyau Linux** : Assurons-nous que notre noyau supporte le système de fichiers souhaité.
- **Besoins en Performance** : OverlayFS est performant pour la plupart des applications.
- **Fonctionnalités Requises** : Si nous avons besoin de fonctionnalités spécifiques, d'autres systèmes de fichiers peuvent être plus appropriés.
- **Complexité Acceptable** : Considérez la complexité de configuration et de maintenance que nous sommes prêt à gérer.

Il est important de tester et de valider le système de fichiers en couches dans notre environnement spécifique pour nous assurer qu'il répond à nos besoins en termes de performance, de stabilité et de fonctionnalités.

---

**Points Clés :**

- **OverlayFS** est le système de fichiers en couches par défaut et recommandé pour Docker sur les distributions Linux modernes.
- **AUFS** n'est plus le choix privilégié en raison de son absence du noyau principal et de sa complexité.
- **Le support dans le noyau Linux** est un facteur déterminant pour la stabilité et la maintenance à long terme.
- **Les performances** et la **simplicité** d'OverlayFS en font le choix idéal pour la plupart des utilisateurs.
- **Des alternatives** comme Device Mapper, Btrfs et ZFS sont disponibles pour des cas d'utilisation spécifiques mais nécessitent une expertise supplémentaire.

# Comment le Mécanisme de Couches Influence-t-il les Stratégies de Déploiement en Continu (CI/CD) ?

## Introduction

Le **déploiement en continu** (Continuous Deployment) et l'**intégration continue** (Continuous Integration) sont des pratiques essentielles dans le développement logiciel moderne. Elles visent à automatiser le processus de construction, de test et de déploiement des applications, permettant ainsi des itérations rapides et une mise en production plus fiable.

Le mécanisme de **couches** (ou *layers*) dans Docker joue un rôle significatif dans la manière dont les pipelines CI/CD sont conçus et optimisés. Comprendre comment les couches fonctionnent peut aider à accélérer les temps de build, à réduire l'utilisation des ressources et à améliorer l'efficacité globale du processus de déploiement.

Dans cette section, nous allons explorer en détail comment le mécanisme des couches influence les stratégies de déploiement en continu, les bonnes pratiques à adopter, et les défis à surmonter.

## Rappel sur le Mécanisme des Couches Docker

Comme mentionné précédemment, une **image Docker** est constituée d'une série de couches immuables, chacune correspondant à une instruction du **Dockerfile** (comme `FROM`, `RUN`, `COPY`, etc.). Lors de la construction d'une image, Docker stocke chaque couche et peut réutiliser celles qui n'ont pas changé lors de builds ultérieurs grâce à un mécanisme de cache.

Ce mécanisme de cache est particulièrement utile pour accélérer les temps de construction, surtout dans un environnement CI/CD où les images sont reconstruites fréquemment.

## Impact sur les Stratégies CI/CD

### 1. **Optimisation des Temps de Build**

- **Réutilisation du Cache** : En structurant le Dockerfile de manière à maximiser la réutilisation des couches mises en cache, on peut réduire considérablement les temps de build dans le pipeline CI/CD.
  
- **Ordre des Instructions** : Placer les instructions qui changent rarement (comme l'installation des dépendances système) au début du Dockerfile, et celles qui changent fréquemment (comme le code source) à la fin, permet de réutiliser le cache pour les premières couches.

### 2. **Consommation des Ressources**

- **Efficacité du Pipeline** : Des temps de build plus courts signifient une utilisation moindre des ressources CI/CD (CPU, mémoire, temps d'exécution), ce qui peut réduire les coûts, surtout si l'on utilise des services payants basés sur l'utilisation.

- **Parallélisation** : La compréhension des couches peut aider à paralléliser certaines étapes du pipeline, améliorant ainsi l'efficacité globale.

### 3. **Déploiements Plus Rapides et Fréquents**

- **Itérations Rapides** : En réduisant les temps de build, les équipes peuvent déployer plus fréquemment, ce qui est un des objectifs clés des pratiques CI/CD.

- **Feedback Rapide** : Un pipeline plus efficace offre un retour rapide sur les changements de code, permettant de détecter et de corriger les problèmes plus tôt.

### 4. **Gestion des Dépendances**

- **Isolation des Dépendances** : Les couches permettent de gérer les dépendances de manière isolée, facilitant la reproduction des environnements de développement, de test et de production.

- **Mises à Jour Contrôlées** : En verrouillant les versions des dépendances dans des couches spécifiques, on peut contrôler les mises à jour et éviter les surprises lors des déploiements.

### 5. **Sécurité et Conformité**

- **Analyse des Couches** : Les outils de sécurité peuvent scanner les couches pour détecter des vulnérabilités. Intégrer ces scans dans le pipeline CI/CD permet de s'assurer que les images déployées sont sécurisées.

- **Traçabilité** : Les couches permettent de tracer les changements et de comprendre quelles modifications ont été apportées à l'image au fil du temps.

## Bonnes Pratiques pour Optimiser le CI/CD avec les Couches Docker

### 1. **Structurer le Dockerfile pour Maximiser le Cache**

- **Instructions Stables en Premier** : Placez les instructions qui changent rarement (installation de paquets système, dépendances) au début du Dockerfile.

- **Instructions Variables en Dernier** : Placez les instructions susceptibles de changer fréquemment (comme `COPY . /app`) à la fin.

#### Exemple :

```dockerfile
# Étape 1 : Image de base
FROM python:3.9-slim

# Étape 2 : Installation des dépendances système
RUN apt-get update && apt-get install -y libpq-dev

# Étape 3 : Définition du répertoire de travail
WORKDIR /app

# Étape 4 : Installation des dépendances Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Étape 5 : Copie du code source
COPY . .

# Étape 6 : Exposition du port
EXPOSE 8000

# Étape 7 : Commande de démarrage
CMD ["python", "app.py"]
```

### 2. **Utiliser des Images Multistages**

- **Images Multi-Étapes** : Les Dockerfiles multi-étapes permettent de construire des images plus légères en n'incluant que les artefacts nécessaires pour l'exécution, excluant les outils de build.

#### Exemple :

```dockerfile
# Étape de build
FROM golang:1.16 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Étape finale
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

- **Avantages** :
  - Réduction de la taille de l'image finale.
  - Séparation claire entre les étapes de build et d'exécution.
  - Amélioration de la sécurité en excluant les outils de build.

### 3. **Gérer les Dépendances avec Soins**

- **Verrouillage des Versions** : Spécifiez les versions exactes des dépendances pour assurer la reproductibilité.

- **Cache des Dépendances** : Utilisez des mécanismes pour mettre en cache les dépendances si elles ne changent pas, comme les fichiers `requirements.txt` pour Python ou `package-lock.json` pour Node.js.

### 4. **Éviter les Invalidation Inutiles du Cache**

- **Minimiser les Modifications** : Évitez de modifier les fichiers ou les répertoires qui sont copiés tôt dans le Dockerfile.

- **Utiliser .dockerignore** : Excluez les fichiers inutiles du contexte de build pour réduire la taille et éviter l'invalidation du cache.

#### Exemple de .dockerignore :

```
.git
.gitignore
README.md
tests/
docs/
```

### 5. **Nettoyer les Caches et les Fichiers Temporaires**

- **Nettoyage dans la Même Instruction RUN** : Supprimez les caches et les fichiers temporaires dans la même instruction `RUN` pour qu'ils ne soient pas inclus dans une couche précédente.

#### Exemple :

```dockerfile
RUN apt-get update && apt-get install -y \
    build-essential \
 && rm -rf /var/lib/apt/lists/*
```

### 6. **Intégrer les Scans de Sécurité dans le Pipeline**

- **Outils de Scan** : Intégrez des outils comme **Trivy**, **Clair** ou **Anchore** pour scanner les images à la recherche de vulnérabilités pendant le pipeline CI/CD.

- **Bloquer en Cas d'Échec** : Configurez le pipeline pour échouer si des vulnérabilités critiques sont détectées.

### 7. **Utiliser des Registres d'Images avec Mise en Cache**

- **Registres Privés** : Utilisez un registre d'images privé qui met en cache les couches, ce qui accélère le téléchargement et le déploiement des images.

- **Tagging Approprié** : Utilisez des tags spécifiques pour les images (`v1.0.0`, `commit-sha`, etc.) pour éviter les conflits et assurer la traçabilité.

### 8. **Paralléliser les Builds Lorsque Possible**

- **Services Indépendants** : Si nous avons plusieurs services ou microservices, construisons-les en parallèle pour gagner du temps.

- **Docker BuildKit** : Utilisez BuildKit, le nouveau moteur de build de Docker, qui offre des fonctionnalités avancées comme le parallélisme et la mise en cache améliorée.

#### Activer BuildKit :

```bash
export DOCKER_BUILDKIT=1
docker build .
```

### 9. **Automatiser les Tests dans les Conteneurs**

- **Tests Unitaires et d'Intégration** : Exécutez les tests dans les conteneurs pour s'assurer que l'application fonctionne correctement dans l'environnement conteneurisé.

- **Étapes du Pipeline** : Intégrez les tests dans les étapes du pipeline CI/CD avant de construire l'image finale.

### 10. **Surveiller et Optimiser en Continu**

- **Logs et Métadonnées** : Collectez les informations sur les temps de build, les tailles d'image, etc., pour identifier les goulots d'étranglement.

- **Améliorations Itératives** : Utilisez ces informations pour apporter des améliorations continues au processus de build et au Dockerfile.

## Défis Potentiels et Comment les Surmonter

### **Invalidation du Cache Inattendue**

- **Cause** : Des changements mineurs dans les fichiers copiés tôt dans le Dockerfile peuvent invalider le cache des couches suivantes.

- **Solution** : Utiliser le fichier `.dockerignore` pour exclure les fichiers qui ne sont pas nécessaires au build.

### **Dépendances Non Détectées**

- **Cause** : Les dépendances installées via des scripts ou des outils qui ne sont pas explicitement spécifiés dans le Dockerfile peuvent entraîner des builds non reproductibles.

- **Solution** : Spécifier toutes les dépendances dans des fichiers de configuration versionnés (comme `requirements.txt`, `package.json`).

### **Gestion des Secrets**

- **Cause** : L'inclusion de secrets ou de clés API dans les couches peut entraîner des risques de sécurité.

- **Solution** : Utiliser des outils comme **Docker secrets** ou injecter les secrets au moment de l'exécution via des variables d'environnement sécurisées.

### **Mises à Jour des Images de Base**

- **Cause** : Les images de base peuvent être mises à jour, invalidant le cache des couches suivantes.

- **Solution** : Planifier des reconstructions régulières des images pour intégrer les mises à jour de sécurité, et verrouiller les versions des images de base lorsque cela est approprié.

## Conclusion

Le mécanisme de couches dans Docker a un impact significatif sur les stratégies de déploiement en continu (CI/CD). En optimisant la structure des Dockerfiles et en comprenant comment les couches sont mises en cache et invalidées, les équipes peuvent :

- **Réduire les Temps de Build** : Grâce à une meilleure réutilisation du cache.

- **Améliorer l'Efficacité du Pipeline** : En consommant moins de ressources.

- **Accélérer les Déploiements** : Permettant des itérations plus rapides et une mise en production plus fiable.

- **Renforcer la Sécurité** : En intégrant des scans de sécurité et en gérant correctement les dépendances.

- **Assurer la Reproductibilité** : En contrôlant les versions des dépendances et en évitant les builds non déterministes.

En appliquant les bonnes pratiques mentionnées et en surveillant continuellement le processus de build, les équipes peuvent tirer pleinement parti du mécanisme de couches de Docker pour optimiser leurs pipelines CI/CD et ainsi améliorer la qualité et la rapidité des livraisons logicielles.

---

**Actions Recommandées :**

- **Revoir et Optimiser les Dockerfiles** : Analysez nos Dockerfiles actuels pour identifier les opportunités d'optimisation.

- **Mettre en Place un Pipeline CI/CD Efficace** : Assurons-nous que notre pipeline tire parti du cache Docker et intègre les bonnes pratiques.

- **Former les Équipes** : Sensibilisez les développeurs et les ingénieurs DevOps à l'importance du mécanisme de couches et à son impact sur le CI/CD.

- **Surveiller et Itérer** : Collectez des métriques sur les temps de build, les tailles d'images, et améliorez continuellement le processus.

# Fonctionnement de Docker côté Hôte : Mécanismes des Namespaces et Partage de la Couche Réseau

Docker est une plateforme de virtualisation légère qui utilise des fonctionnalités du noyau Linux pour isoler les applications dans des conteneurs. Ces conteneurs fonctionnent comme des instances isolées de systèmes d'exploitation, mais sans la surcharge d'une machine virtuelle complète. Le fonctionnement interne de Docker repose principalement sur deux mécanismes clés du noyau Linux :

1. **Les Namespaces** : Pour l'isolation des ressources.
2. **Les Groupes de Contrôle (cgroups)** : Pour la limitation et la surveillance des ressources.

Dans cette section, nous allons explorer en détail comment Docker utilise ces mécanismes, en mettant l'accent sur les namespaces et le partage de la couche réseau.

## Introduction aux Namespaces Linux

Les **namespaces** sont une fonctionnalité du noyau Linux qui permet d'isoler et de virtualiser les ressources du système pour un ensemble de processus. Chaque namespace crée une vue indépendante d'une ressource, ce qui signifie que les processus dans un namespace ne peuvent pas voir ou interagir directement avec les ressources en dehors de ce namespace.

### Types de Namespaces

Il existe plusieurs types de namespaces, chacun isolant un aspect spécifique du système :

1. **Mount Namespace (`mnt`)** : Isolent le système de fichiers montés.
2. **Process ID Namespace (`pid`)** : Isolent l'espace des identifiants de processus.
3. **Network Namespace (`net`)** : Isolent la pile réseau.
4. **Inter-Process Communication Namespace (`ipc`)** : Isolent les ressources IPC (mémoires partagées, sémaphores).
5. **UTS Namespace (`uts`)** : Isolent les noms de domaine et les noms d'hôte.
6. **User Namespace (`user`)** : Isolent les identifiants d'utilisateurs et de groupes.
7. **Cgroup Namespace (`cgroup`)** : Isolent la vue des cgroups.

Docker utilise ces namespaces pour créer des environnements isolés pour chaque conteneur.

## Fonctionnement des Namespaces dans Docker

### Mount Namespace

- **Isolation du Système de Fichiers** : Chaque conteneur voit un système de fichiers indépendant. Les changements dans le système de fichiers d'un conteneur n'affectent pas les autres conteneurs ou l'hôte.
- **Utilisation des Couches (Layers)** : Docker utilise un système de fichiers en couches pour construire le système de fichiers des conteneurs, comme expliqué précédemment.

### PID Namespace

- **Isolation des Processus** : Les processus à l'intérieur d'un conteneur ne voient que les processus de ce conteneur. Le PID 1 dans le conteneur est le processus principal du conteneur.
- **Avantages** :
  - Empêche les processus d'un conteneur d'interagir avec ceux d'un autre conteneur ou de l'hôte.
  - Facilite la gestion des processus au sein du conteneur.

### Network Namespace

- **Isolation Réseau** : Chaque conteneur possède sa propre pile réseau, avec ses propres interfaces réseau, tables de routage, ports, etc.
- **Communication Interne** : Les conteneurs ne peuvent pas communiquer directement avec les interfaces réseau de l'hôte ou d'autres conteneurs sans configuration explicite.
- **Avantages** :
  - Permet d'attribuer des adresses IP distinctes aux conteneurs.
  - Contrôle précis sur les règles de pare-feu et les politiques de routage.

### IPC Namespace

- **Isolation des Mécanismes IPC** : Les mécanismes de communication inter-processus (comme les mémoires partagées) sont isolés.
- **Sécurité** : Empêche les processus d'un conteneur d'accéder aux ressources IPC d'un autre conteneur ou de l'hôte.

### UTS Namespace

- **Isolation du Nom d'Hôte et du Domaine** : Chaque conteneur peut avoir son propre nom d'hôte et de domaine.
- **Personnalisation** : Utile pour les applications qui dépendent du nom d'hôte pour leur configuration ou leur fonctionnement.

### User Namespace

- **Isolation des Identifiants Utilisateurs** : Mappe les UID et GID du conteneur à ceux de l'hôte.
- **Sécurité Renforcée** : Permet aux processus dans le conteneur de s'exécuter en tant que root dans le conteneur, mais avec des privilèges limités sur l'hôte.

### Cgroup Namespace

- **Isolation des Cgroups** : Fournit une vue isolée des groupes de contrôle, améliorant la sécurité et la gestion des ressources.

## Les Groupes de Contrôle (cgroups)

En plus des namespaces, Docker utilise les **cgroups** pour limiter et surveiller l'utilisation des ressources par les conteneurs.

- **Limitation des Ressources** : CPU, mémoire, I/O disque, réseau.
- **Isolation** : Empêche un conteneur de consommer toutes les ressources de l'hôte.
- **Monitoring** : Permet de surveiller la consommation des ressources par chaque conteneur.

## Partage de la Couche Réseau

### Principe du Network Namespace

Par défaut, Docker crée un **network namespace** pour chaque conteneur, isolant ainsi la pile réseau du conteneur de celle de l'hôte et des autres conteneurs.

- **Interfaces Réseau Isolées** : Chaque conteneur a sa propre interface réseau virtuelle.
- **Tables de Routage Indépendantes** : Les règles de routage et les tables ARP sont propres au conteneur.

### Le Pont Réseau Docker (`docker0`)

- **Bridge Network par Défaut** : Docker crée un pont réseau virtuel nommé `docker0` sur l'hôte.
- **Connexions des Conteneurs** : Les interfaces réseau virtuelles des conteneurs sont connectées au pont `docker0`.
- **Fonctionnement** :
  - Lorsque nous lançons un conteneur, Docker crée une paire d'interfaces virtuelles (veth pairs).
  - Une extrémité est placée dans le network namespace du conteneur.
  - L'autre extrémité est attachée au pont `docker0` sur l'hôte.
- **Adresses IP** :
  - Docker attribue une adresse IP privée au conteneur sur le réseau du pont (`172.17.0.0/16` par défaut).
  - Les conteneurs peuvent communiquer entre eux via le pont.

### Port Mapping et NAT

- **Exposition des Ports** : Pour accéder à un service dans un conteneur depuis l'extérieur, nous devons mapper un port du conteneur à un port de l'hôte.
- **Attention à l'exposition des ports** : Si aucune adresse n'est précisée comme restriction, le port est accessible depuis l'extérieur de l'hôte.
- **Commande** :
  ```bash
  docker run -p <adresse_autorisée>:<port_hôte>:<port_conteneur> mon_image
  ```
- **NAT (Network Address Translation)** :
  - Docker configure des règles iptables pour effectuer la traduction d'adresses entre l'hôte et le conteneur.
  - Permet aux paquets entrants sur l'hôte d'être redirigés vers le conteneur.
  - `<adresse_autorisée>` est mis à 127.0.0.1 si l'on veut que seul l'hôte voit l'exposition du port du conteur.

### Réseaux Personnalisés

- **Bridge Networks Personnalisés** :
  - Nous pouvons créer nos propres réseaux de ponts pour isoler des groupes de conteneurs.
  - Commande :
    ```bash
    docker network create mon_reseau
    ```
- **Avantages** :
  - Contrôle sur la configuration réseau (plage IP, DNS, etc.).
  - Isolation entre différents réseaux.

### Network Namespaces Partagés

- **Option `--network`** :
  - Nous pouvons configurer des conteneurs pour qu'ils partagent le même network namespace.
  - Permet à plusieurs conteneurs de partager la même pile réseau.
- **Cas d'Utilisation** :
  - Conteneurs légers qui doivent interagir étroitement.
  - Exécution de processus de surveillance ou de gestion du réseau.

### Modes Réseau Spéciaux

- **Host Networking** :
  - En utilisant `--network=host`, le conteneur partage le network namespace de l'hôte.
  - Pas d'isolation réseau : le conteneur utilise les interfaces réseau de l'hôte.
  - Avantage : Réduction de la latence réseau.
  - Inconvénient : Moins d'isolation, risques de sécurité accrus.

- **None Networking** :
  - En utilisant `--network=none`, le conteneur n'a pas de pile réseau.
  - Utile pour des conteneurs qui n'ont pas besoin de communiquer sur le réseau.

## Fonctionnement Global de Docker Côté Hôte

### Processus de Création d'un Conteneur

1. **Création des Namespaces** :
   - Docker demande au noyau Linux de créer les différents namespaces pour le conteneur.
   - Isolent les ressources système (PID, réseau, système de fichiers, etc.).

2. **Configuration des Cgroups** :
   - Docker configure les cgroups pour limiter l'utilisation des ressources par le conteneur.
   - Paramètres comme la limite de mémoire, le pourcentage de CPU, etc.

3. **Configuration du Système de Fichiers** :
   - Montage du système de fichiers du conteneur en utilisant le système de fichiers en couches.
   - Application des changements spécifiques au conteneur.

4. **Configuration du Réseau** :
   - Création d'un network namespace pour le conteneur.
   - Configuration des interfaces réseau virtuelles et des adresses IP.
   - Ajout de règles iptables pour le NAT et le port mapping.

5. **Démarrage du Processus Principal** :
   - Exécution du processus spécifié (CMD ou ENTRYPOINT) dans le conteneur.
   - Le processus s'exécute avec l'UID et le GID spécifiés, généralement avec des privilèges réduits.

### Interaction avec le Noyau

- **Isolation vs. Partage** :
  - Les conteneurs sont isolés au niveau utilisateur, mais partagent le même noyau Linux.
  - Les appels système sont gérés par le noyau de l'hôte.

- **Sécurité** :
  - L'utilisation de namespaces et de cgroups renforce la sécurité en isolant les conteneurs.
  - Cependant, une vulnérabilité au niveau du noyau peut affecter tous les conteneurs.

## Avantages du Mécanisme des Namespaces

- **Isolation** : Fournit un environnement isolé pour chaque conteneur.
- **Sécurité** : Réduit les risques de fuite d'informations entre conteneurs.
- **Gestion des Ressources** : Les cgroups permettent de contrôler finement l'utilisation des ressources.
- **Flexibilité** : Les conteneurs peuvent être configurés pour partager ou isoler certaines ressources selon les besoins.

## Limitations et Considérations

- **Partage du Noyau** :
  - Tous les conteneurs partagent le même noyau, ce qui peut être une source de vulnérabilités.
  - Les mises à jour du noyau affectent tous les conteneurs.

- **Complexité Réseau** :
  - La configuration réseau peut devenir complexe avec de nombreux conteneurs et réseaux personnalisés.
  - Les règles iptables générées par Docker peuvent interférer avec les configurations manuelles.

- **Limitations des Namespaces** :
  - Certaines fonctionnalités ne sont pas entièrement isolées, comme le noyau lui-même.
  - Les conteneurs ne sont pas des machines virtuelles complètes.

## Conclusion sur Docker côté hôte

Docker utilise les fonctionnalités avancées du noyau Linux, en particulier les namespaces et les cgroups, pour fournir un environnement isolé et contrôlé pour les conteneurs. Le mécanisme des namespaces permet une isolation fine des ressources, tandis que les cgroups assurent le contrôle et la limitation de l'utilisation des ressources système.

Le partage de la couche réseau est géré via les network namespaces et les interfaces réseau virtuelles, offrant une grande flexibilité dans la configuration réseau des conteneurs. En comprenant ces mécanismes, il est possible de tirer pleinement parti de Docker pour déployer des applications de manière efficace, sécurisée et scalable.

---

**Points Clés :**

- **Namespaces** : Mécanisme d'isolation des ressources au niveau du noyau Linux.
- **Cgroups** : Mécanisme de limitation et de surveillance des ressources.
- **Network Namespaces** : Chaque conteneur a sa propre pile réseau isolée.
- **Pont Réseau Docker (`docker0`)** : Connecte les interfaces réseau virtuelles des conteneurs.
- **Port Mapping et NAT** : Permettent l'accès aux services des conteneurs depuis l'extérieur.
- **Flexibilité Réseau** : Possibilité de créer des réseaux personnalisés et de configurer le partage du network namespace.
- **Sécurité et Isolation** : Les mécanismes utilisés par Docker renforcent la sécurité, mais des considérations restent nécessaires, notamment en matière de partage du noyau.

---

**Questions pour Approfondir :**

- Comment les cgroups sont-ils utilisés pour surveiller et limiter les ressources spécifiques comme le CPU et la mémoire ?
- Quelles sont les implications de la partage du noyau Linux entre l'hôte et les conteneurs en termes de sécurité ?
- Comment Docker Compose et Kubernetes gèrent-ils les configurations réseau avancées avec les namespaces ?

# Utilisation de Docker Compose

Utilisation de Docker Compose pour définir et exécuter des applications multi-conteneurs :

## Introduction
Docker Compose est un outil qui permet de définir et de gérer des applications multi-conteneurs en utilisant un fichier de configuration. Il facilite la création, la mise à l'échelle et la gestion des conteneurs d'une application complexe.

## Les concepts clés de Docker Compose
Le fichier `compose.yaml` est le cœur de la configuration d'une application multi-conteneurs.
Il décrit tous les services, réseaux et volumes nécessaires pour faire fonctionner l'application. Quelques sections courantes d'un fichier Compose :
- **services** : Définit les conteneurs qui constituent l'application. Chaque service peut être basé sur une image Docker existante ou construite à partir d'un `Dockerfile`.
- **build** : Spécifie les instructions pour construire une image Docker à partir d'un `Dockerfile`. - **image** : Définit l'image Docker à utiliser pour un service.
- **ports** : Expose des ports spécifiques du conteneur à l'hôte, permettant à des services externes d'accéder à l'application.
- **volumes** : Monte des volumes pour partager des fichiers et des répertoires entre l'hôte et les conteneurs.
- **environment** : Définit des variables d'environnement pour les services, souvent utilisées pour configurer les conteneurs au démarrage.
- **depends_on** : Indique les dépendances entre les services, ce qui permet de spécifier l'ordre de démarrage des conteneurs.
- **networks** : Définit les réseaux personnalisés sur lesquels les services communiqueront.
- **watch** : Permet de synchroniser l'image avec les changements du code source, ce qui est particulièrement utile pour le développement en continu.
## Exemples
Un exemple de Compose file qui définit une application web qui utilise un service Nginx :
```yaml
    reverseproxy:
        image: reverseproxy
        ports:
            - 8080:8080
            - 8081:8081
        restart: always
 
    nginx:
        depends_on:
            - reverseproxy
        image: nginx:alpine
        restart: always
 
    apache:
        depends_on:
            - reverseproxy
        image: httpd:alpine
        restart: always
```

## Principales commandes de `docker-compose`

Voici un aperçu des principales commandes utilisées avec **Docker Compose**, ainsi qu’une brève description de chacune :

1. **docker-compose up**
    
    - Démarre les conteneurs définis dans votre fichier `docker-compose.yml`.
    - En utilisant l’option `-d` (_detach mode_), les conteneurs s’exécutent en arrière-plan :
        
        ```bash
        docker-compose up -d
        ```
        
    - Sans l’option `-d`, vous verrez les logs en direct dans votre terminal.
    
2. **docker-compose down**
    
    - Arrête et supprime les conteneurs, les réseaux et autres artefacts (volumes anonymes, etc.) créés par `docker-compose up`.
    - Permet de nettoyer l’environnement, notamment si vous voulez redémarrer de zéro.
        
        ```bash
        docker-compose down
        ```
        
3. **docker-compose build**
    
    - Construit ou re-construit les images définies dans le `docker-compose.yml`.
    - Peut être utile si vous avez modifié votre Dockerfile ou vos configurations et que vous souhaitez que l’image soit à jour.
        
        ```bash
        docker-compose build
        ```
        
4. **docker-compose pull**
    
    - Récupère (depuis un registry) les dernières images spécifiées dans votre `docker-compose.yml`.
    - Souvent utilisé avant un `docker-compose up` pour s’assurer d’avoir la version la plus récente des images.
        
        ```bash
        docker-compose pull
        ```
        
5. **docker-compose push**
    
    - Envoie (pousse) les images construites localement vers un registry Docker distant.
    - Nécessite d’avoir la bonne configuration de registry et les tags appropriés.
        
        ```bash
        docker-compose push
        ```
        
6. **docker-compose run**
    
    - Lance un _one-off container_, c’est-à-dire un conteneur ponctuel pour exécuter une commande.
    - Par exemple, exécuter un script de migration de base de données :
        
        ```bash
        docker-compose run web python manage.py migrate
        ```
        
    - Ne démarre pas tous les services, uniquement celui que vous spécifiez (ici `web`).
    
7. **docker-compose exec**
    
    - Permet d’exécuter une commande dans un conteneur déjà en cours d’exécution.
    - Par exemple, pour ouvrir un shell dans un conteneur :
        
        ```bash
        docker-compose exec web sh
        ```
        
    - S’emploie souvent pour diagnostiquer ou administrer une application à l’intérieur du conteneur.
    
8. **docker-compose logs**
    
    - Affiche les logs des conteneurs.
    - Vous pouvez préciser un service particulier :
        
        ```bash
        docker-compose logs web
        ```
        
    - L’option `-f` (follow) vous permet de suivre les logs en continu :
        
        ```bash
        docker-compose logs -f web
        ```
        
9. **docker-compose ps**
    
    - Liste les conteneurs gérés par Docker Compose dans le projet courant.
    - Vous donne un aperçu rapide des conteneurs en cours d’exécution ou stoppés, et leurs ports.
        
        ```bash
        docker-compose ps
        ```
        
10. **docker-compose start / stop / restart**
    
    - Respectivement pour démarrer, arrêter ou redémarrer les conteneurs déjà créés.
    - Exemple :
        
        ```bash
        docker-compose stop
        docker-compose start
        docker-compose restart
        ```
        
11. **docker-compose pause / unpause**
    
    - Met en pause (`pause`) ou reprend (`unpause`) l’exécution des conteneurs.
    - Utile pour stopper temporairement un service sans l’arrêter complètement.
        
        ```bash
        docker-compose pause
        docker-compose unpause
        ```
        
12. **docker-compose rm**
    
    - Supprime les conteneurs arrêtés qui appartiennent à votre projet compose.
    - Si vous avez fait un `docker-compose stop` et que vous ne souhaitez plus conserver ces conteneurs, vous pouvez faire :
        
        ```bash
        docker-compose rm
        ```
        
    - Par défaut, il demande confirmation avant de supprimer.
    
13. **docker-compose config**
    
    - Valide et affiche la configuration combinée du fichier `docker-compose.yml`.
    - Pratique pour vérifier si votre syntaxe est correcte et pour visualiser la configuration finale après le merge avec d’éventuels fichiers d’override.
        
        ```bash
        docker-compose config
        ```

### Résumé des commandes `docker-compose`

- **up** : Lance tous les services définis.
- **down** : Arrête et supprime les ressources associées.
- **build** : Construit les images.
- **pull/push** : Récupère ou publie les images depuis/vers un registry.
- **run** : Lance un conteneur ponctuel pour une commande donnée.
- **exec** : Exécute une commande dans un conteneur en cours d’exécution.
- **logs** : Affiche ou suit les journaux.
- **ps** : Liste les conteneurs en cours de gestion.
- **start/stop/restart** : Gère l’état d’exécution des conteneurs.
- **pause/unpause** : Met en pause ou reprend l’exécution.
- **rm** : Supprime les conteneurs arrêtés.
- **config** : Valide et affiche la configuration.

Ces commandes couvrent la majorité des opérations de base que nous réaliserons avec Docker Compose pour développer, tester et déployer nos applications multi-conteneurs.
## Pratique
-   Créez un fichier Compose
-   Utilisez les instructions pour définir une application multi-conteneurs
-   Exécutez l'application en utilisant les commandes de Compose

# Les réseaux
Par défaut docker propose 3 types de réseau Bridget, Host, Aucun. Et il permet d'en créer ce qui remplace l'ancienne commande "--link".
Pour des conteneurs qui fonctionnent nous pouvons intérogger leur adresse et numéro de port avec les commandes :
```bash
michaellaunay@leojag:~$ docker run -d --name c2 nginx:latest
f3ba0b7b5e5b77656675a146ca2ff677fa67bfab865470ba39814168ada106f4
michaellaunay@leojag:~$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
f3ba0b7b5e5b   nginx:latest   "/docker-entrypoint.…"   27 seconds ago   Up 26 seconds   80/tcp    c2
42f1fcfb86a5   nginx:latest   "/docker-entrypoint.…"   34 seconds ago   Up 33 seconds   80/tcp    c1
michaellaunay@leojag:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
64194948d69f   bridge    bridge    local
9b6750ca6345   host      host      local
542911f5334c   none      null      local
michaellaunay@leojag:~$ docker network inspect 64194948d69f
[
    {
        "Name": "bridge",
        "Id": "64194948d69f79fd8e79ae29cbb5fcf15025d0cbae9472b95542a06b3613e482",
        "Created": "2023-03-23T18:51:45.348300819+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "42f1fcfb86a5f8a07b90c7e5ec1a37b36af630e5dc65c4b1d2717f11d305eeb6": {
                "Name": "c1",
                "EndpointID": "59916226657d2b54ddfe23441ce80178984794441603fe065e6630876d62493c",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "f3ba0b7b5e5b77656675a146ca2ff677fa67bfab865470ba39814168ada106f4": {
                "Name": "c2",
                "EndpointID": "aba35f15ee3a1ef73ba316a0f10796fd690f45be8a37ad03119339a0b1571984",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
michaellaunay@leojag:~$ docker ps -q | xargs docker inspect --format '{{ .Name }} - {{ .NetworkSettings.Ports }}'
/c2 - map[80/tcp:[]]
/c1 - map[80/tcp:[]]
michaellaunay@leojag:~$ docker ps -q | xargs docker inspect --format '{{ .Name }} - {{ .NetworkSettings.IPAddress }}'
/c2 - 172.17.0.3
/c1 - 172.17.0.2
michaellaunay@leojag:~$ docker exec -ti c1 bash
root@42f1fcfb86a5:/# apt install iputils-ping
Reading package lists... Done
...
root@42f1fcfb86a5:/# ping c2
ping: c2: Name or service not known
root@42f1fcfb86a5:/# ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.082 ms
```
**Attention** Les adresses ip  sont dynamiques et par défaut il n'y a pas de résolution de nom (Pas de DNS privé).
Il faut donc créer un réseau personnel.

Pour cela il suffit de faire :
```bash
michaellaunay@leojag:~$ docker run -d --name c1 --network MonReseau0 nginx:latest
bfaaf1b13522caaeb6a83daf850ca3465ea3dad95b70c2e6e7df2487858b2698
michaellaunay@leojag:~$ docker network ls
NETWORK ID     NAME         DRIVER    SCOPE
02e68136c61c   MonReseau0   bridge    local
64194948d69f   bridge       bridge    local
9b6750ca6345   host         host      local
542911f5334c   none         null      local
michaellaunay@leojag:~$ docker run -d --name c2 --network MonReseau0 nginx:latest
91eaa40f1c654baa30e0a9e1eb43c66dd46e132a54151f1df76edfd4c3bc5a5e
michaellaunay@leojag:~$ docker run -d --name c1 --network MonReseau0 nginx:latest
7b22e62a808622ad44e796f5abff75fb700c8545aec5e02127342f35d57194e1
michaellaunay@leojag:~$ docker exec -ti c1 bash
root@7b22e62a8086:/# apt install iputils-ping
Reading package lists... Done
...
root@7b22e62a8086:/# ping C2
PING C2 (192.168.7.2) 56(84) bytes of data.
64 bytes from c2.MonReseau0 (192.168.7.2): icmp_seq=1 ttl=64 time=0.095 ms
```
**Attention** à ne pas créer un sous réseau avec les mêmes adresses que notre réseau privé !

# Gestion de clusters
Nous allons aborder les raisons pour lesquelles il peut être nécessaire d'utiliser un système de gestion de clusters et les différentes situations où cela peut être bénéfique.

Les raisons pour lesquelles nous pourrions vouloir utiliser un système de gestion de clusters incluent :

- La haute disponibilité : en utilisant un cluster, nous pouvons nous assurer que notre application est toujours disponible même si un conteneur ou un noeud échoue.
- La scalabilité : les clusters permettent de facilement ajouter ou retirer des conteneurs pour répondre aux besoins changeants de notre application.
- La répartition de charge : en répartissant les conteneurs sur plusieurs noeuds, nous pouvons équilibrer la charge sur notre système pour éviter les surcharges.

Il y a plusieurs systèmes de gestion de clusters disponibles, tels que Docker Swarm et Kubernetes. Ces systèmes sont utilisés pour gérer des clusters de conteneurs en production. Ils permettent de décrire les déploiements, les services et les volumes de manière déclarative.

Il y a aussi des avantages et des limites à utiliser un système de gestion de clusters.

## Avantages
Scalabilité : les clusters de conteneurs permettent de facilement ajouter ou retirer des noeuds pour répondre aux besoins changeants de l'application. Cela permet d'optimiser les ressources utilisées et de gérer les pics de charge.

Disponibilité : en utilisant des clusters, les applications peuvent être mises à l'échelle pour gérer les échecs de noeuds individuels. Les systèmes de gestion de clusters permettent également de mettre en place des stratégies de tolérance de panne pour maintenir la disponibilité de l'application.

Flexibilité : les systèmes de gestion de clusters permettent de déployer des applications sur différents types de noeuds, y compris des environnements cloud et edge, offrant ainsi plus de flexibilité pour les déploiements.

Automatisation : les systèmes de gestion de clusters automatisent de nombreuses tâches liées à la gestion des clusters, telles que le déploiement, la mise à l'échelle et la surveillance.

## Limites
Portabilité : Les clusters de conteneurs peuvent être déployés sur différentes plateformes, notamment les clouds publics, privés et hybrides, ce qui permet une portabilité accrue.

Complexité accrue : Gérer un cluster de conteneurs est plus complexe que de gérer un conteneur unique. Il y a plus de composants à surveiller et configurer, et les erreurs peuvent être plus difficiles à identifier et à résoudre.

La nécessité de compétences supplémentaires : Les administrateurs de systèmes et les développeurs doivent avoir des compétences supplémentaires pour gérer et déployer efficacement des clusters de conteneurs.

Les coûts supplémentaires : Les systèmes de gestion de clusters peuvent avoir des coûts supplémentaires liés à la gestion et à l'exploitation des ressources supplémentaires nécessaires pour gérer le cluster.

La consommation de ressources : Les clusters de conteneurs peuvent consommer plus de ressources système que les conteneurs individuels, ce qui peut nécessiter des mises à niveau matérielles coûteuses.

Il est important de noter que la gestion de clusters de conteneurs peut être complexe et nécessiter une expertise technique élevée. Il peut également y avoir des coûts supplémentaires liés à l'utilisation d'un système de gestion de clusters, mais ces coûts peuvent être considérablement réduits grâce à l'automatisation et à l'optimisation des ressources.

Il est également important de choisir un système de gestion de clusters adapté à nos besoins spécifiques en termes de scalabilité, de disponibilité et de flexibilité. Il existe de nombreux systèmes de gestion de clusters différents, tels que Kubernetes, Docker Swarm, Apache Mesos, etc. chacun ayant ses propres avantages et limites.

# Liens
> [Premier Pas & Installation](https://youtu.be/AWJG-AbGFik)
> [Le Dockerfile et ses instructions](https://youtu.be/BUJAI1ptUu8)
> [Les layers Docker](https://youtu.be/_iUjrWhzx9o)
> [Les différents types de volumes docker](https://youtu.be/vHGwkA9fsIs)
> [Les volumes](https://youtu.be/lstTLSM5494)
> [Le UserID pour les volumes](https://youtu.be/SI1mhq-f0rg)
> [ENTRYPOINT vs CMD](https://youtu.be/kfyDu5R4VrM)
> [Docker pour Gitlab](https://youtu.be/Wbaczrx-US0)
> [Documentation officielle](https://docs.docker.com/engine/install/#server)
> [Article NextInpact](https://www.nextinpact.com/article/48913/docker-et-conteneurisation-par-exemple)
> [Network, réseau personnalisé, ip et driver host](https://youtu.be/wQIyczrZNQM)