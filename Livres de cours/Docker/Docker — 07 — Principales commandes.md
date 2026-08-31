---
schema_version: 1
uid: 01M1BQ62DVKE1AS3KHY3FBH22Q
titre: "Docker — 07 — Principales commandes"
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
resume: "Chapitre 7 sur 17 du livre « Docker » : Principales commandes. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 7/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 06 — Le mode détaché|← 06 — Le mode détaché]] · [[Docker — 08 — Le Mécanisme des Couches (Layers) dans Docker|08 — Le Mécanisme des Couches (Layers) dans Docker →]]

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

## Utiliser `docker attach`
La commande `docker attach` permet de rattacher le terminal courant à un conteneur en cours d'exécution. Cela nous permet de visualiser la sortie en direct du conteneur et d'interagir avec lui si un terminal a été alloué (lors de l'utilisation de `-t`).

**Exemple :**
```bash
docker attach mon_conteneur
```
*Remarque :* Si le conteneur ne dispose pas de session interactive (par exemple, s'il a été lancé sans l'option `-it`), la commande `docker attach` nous permettra seulement de voir la sortie du conteneur, sans interaction possible.

## Limites de `docker attach` :
- Si nous quittons le conteneur en appuyant sur `CTRL+C`, cela arrêtera le conteneur.
- Pour se détacher sans arrêter le conteneur, utilisez `CTRL+P` suivi de `CTRL+Q`.

## Utiliser `docker exec`
La méthode la plus courante et la plus flexible pour interagir avec un conteneur en mode interactif est d'utiliser la commande `docker exec`. Cette commande permet d'exécuter un processus à l'intérieur d'un conteneur en cours d'exécution, comme ouvrir un shell interactif.

**Exemple :**
```bash
docker exec -ti mon_conteneur bash
```
- **`-ti`** : Ouvre une session interactive avec un terminal.
- **`bash`** : Indique que nous souhaitons ouvrir un shell Bash (assurez-nous que Bash est installé dans le conteneur).

## Différences entre `docker attach` et `docker exec` :
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

---
> [!info] Livre « Docker » — chapitre 7/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 06 — Le mode détaché|← 06 — Le mode détaché]] · [[Docker — 08 — Le Mécanisme des Couches (Layers) dans Docker|08 — Le Mécanisme des Couches (Layers) dans Docker →]]
