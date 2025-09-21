#Cours #Informatique 
# Objectif
Nous allons voir ici comment installer et utiliser Docker. Docker est un système de conteneurisation, c'est-à-dire qu'il permet de crée des images d'une autre distribution très légère qui utilisent ou partagent les ressources de l'ordinateur hôte.

# Introduction à Docker

Docker est un logiciel qui permet d'empaqueter une application avec toutes ses dépendances, et de les exécuter de manière isolée dans un environnement standardisé.

La principale différence entre les conteneurs et les machines virtuelles, est que les conteneurs utilisent le système d'exploitation hôte, tandis que les machines virtuelles utilisent leur propre système d'exploitation.
Les conteneurs sont donc plus légers et plus rapides à démarrer.

La raison prncipale d'utilisation de Docker est faciliter le déploiement d'applications, de les rendre portables et de garantir une meilleure reproductibilité des environnements.
Le système hébergeant Docker est appelé l'hote, c'est le système d'exploitation de l'ordinateur qui fera fonctionner docker et les images docker.

Les images sont décrites au format texte dans des fichiers appellés DockerFiles.
Ces modèles d'images sont partagés sur un dépôt public appelé DockerHUB accessible à l'adresse https://www.docker.com/ .
Toutefois, il faut faire attention au DockerFile qui y sont, car il y a peu ou pas de vérification de la majorité des images.

Sur windows 11, Docker utilise [[WSL2]] et propose sa configuration lors de son installation, généralement nous allons chercher le logiciel Docker sur https://docker.com (voir https://www.youtube.com/watch?v=6Wpp6C8cUf0&ab_channel=angleformation )
Sur Ubuntu ou Debian, soit nous utilisons les packets de la distribution soit nous téléchargeons la dernière version ( https://youtu.be/AWJG-AbGFik )

# Définition
Docker est un logiciel open-source qui facilite la création, le déploiement et l'exécution d'applications dans des conteneurs logiciels. Un conteneur est un environnement logiciel qui contient tout ce dont une application a besoin pour fonctionner, comme les bibliothèques, les outils de développement et les fichiers de configuration. Les conteneurs sont similaires aux machines virtuelles, mais ils sont beaucoup plus légers et plus rapides à démarrer, car ils partagent les ressources de l'hôte sur lequel ils sont exécutés.

L'un des avantages clés de Docker est sa portabilité. Les conteneurs Docker peuvent être créés sur un ordinateur de développement, testés sur un autre et déployés sur un serveur de production, tout en étant sûrs qu'ils fonctionneront de la même manière dans chaque environnement.

Docker utilise des images pour construire les conteneurs. Une image est une capture de l'état d'une application, qui comprend tous les fichiers et les configurations nécessaires pour exécuter l'application. Il est possible de créer des images à partir de conteneurs existants ou de construire des images à partir de zéro en utilisant un script de construction appelé "Dockerfile".

Docker permet également de gérer facilement les conteneurs à l'échelle. Nous pouvons utiliser des outils tels que Docker Compose pour définir des applications multi-conteneurs et les déployer sur plusieurs serveurs, ou utiliser des services tels que Kubernetes pour gérer les conteneurs à l'échelle de production.

En résumé, Docker est un système de virtualisation de conteneurs qui permet de créer, de déployer et d'exécuter des applications de manière efficiente et portable.

# Installation
Pour installer docker le plus simple est d'utiliser la version des dépôts et donc de faire : :
```bash
apt install docker
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

Pour accéder au shell du conteneur en mode interactif, il faut mettre le flag -it Pour effacer le conteneur une fois terminé, mettre le flag \--rm

Pour utiliser le système de fichier du host à la place de celui du Contener, cela pour pouvoir conserver des données entre images il faut utiliser l'option \"-v\" : :

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

# Les Dockerfiles
Le Dockerfile est le fichier de configuration pour produire des images Docker.
Il contient une séquences d'instructions indiquant comment construire l'image.
Ces séquences sont constituées des mots clés : FROM, ARG, ENV, RUN, VOLUME, COPY, USER, ENTRYPOINT, CMD.

Le mot-clé FROM est utilisé pour savoir sur quelle image source nous baserons l'image en cours de construction. Il est important de choisir une image de base adaptée à notre application, afin de minimiser la taille de l'image finale.

Le mot-clé ARG est utilisé pour passer des variables uniquement lors de la construction de l'image. Cela peut être utile pour transmettre des secrets ou des informations sensibles qui ne doivent pas être stockées dans l'image.

Le mot-clé ENV permet de définir des variables qui seront valables lors de l'exécution du conteneur. Par exemple, nous pouvons définir une variable d'environnement pour indiquer le port sur lequel notre application écoute.

Le mot-clé RUN permet d'exécuter des commandes à l'intérieur de l'image en cours de construction. Cela peut être utilisé pour installer des dépendances, configurer des paramètres, etc.

Le mot-clé VOLUME permet de définir des points de montage pour des volumes externes. Cela permet de stocker des données en dehors de l'image, afin de pouvoir les partager entre plusieurs conteneurs.

Le mot-clé COPY permet de copier des fichiers ou des répertoires depuis l'hôte vers l'image en cours de construction. Cela peut être utilisé pour ajouter des fichiers de configuration, des scripts, etc.

Le mot-clé USER permet de définir l'utilisateur qui sera utilisé pour exécuter les commandes à l'intérieur de l'image. Cela peut être utile pour renforcer la sécurité en limitant les privilèges d'exécution.

Le mot-clé ENTRYPOINT permet de définir la commande qui sera exécutée lorsque le conteneur est démarré. Cela peut être utilisé pour lancer une application ou un script spécifique.

Enfin, le mot-clé CMD permet de définir les arguments à passer à l'entrée de la commande ENTRYPOINT. Cela peut être utilisé pour paramétrer l'application ou le script lancé.

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

## Pratique
-   Créez un fichier Compose file
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