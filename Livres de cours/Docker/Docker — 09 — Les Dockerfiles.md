---
schema_version: 1
uid: 01M1BQ62DVX7MCVPZF090JXTTB
titre: "Docker — 09 — Les Dockerfiles"
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
resume: "Chapitre 9 sur 17 du livre « Docker » : Les Dockerfiles. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Docker » — chapitre 9/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 08 — Le Mécanisme des Couches (Layers) dans Docker|← 08 — Le Mécanisme des Couches (Layers) dans Docker]] · [[Docker — 10 — L'Impact des Couches sur la Sécurité des Images Docker|10 — L'Impact des Couches sur la Sécurité des Images Docker →]]

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

---
> [!info] Livre « Docker » — chapitre 9/17
> [[Docker — Sommaire|Sommaire]] · [[Docker — 08 — Le Mécanisme des Couches (Layers) dans Docker|← 08 — Le Mécanisme des Couches (Layers) dans Docker]] · [[Docker — 10 — L'Impact des Couches sur la Sécurité des Images Docker|10 — L'Impact des Couches sur la Sécurité des Images Docker →]]
