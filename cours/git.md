Git est un outil de gestion de versions du code développé par **Linus Torval** fondateur de Linux pour gérer le **versioning** du noyau Linux à partir de 2005.

# Installer le client
```bash
apt install git
```

# Configurer son compte
```bash
git config user.name Michaël Launay
git config user.mail michaellaunay@ecreall.com
```

# Connaître sa configuration
```bash
git config --list
```
La configuration se trouve soit dans :
    "/etc/gitconfig"
    "~/.gitconfig"
    <rep_projet>/.git/config

## Liens
- https://git-scm.com/book/fr/v2/Personnalisation-de-Git-Configuration-de-Git
- https://youtu.be/KVULgIbyeQs

# Créer un projet
Soit directement depuis github ou gitlab, puis faire un "git clone", soit créer un projet local et l'importer dans github/gitlab en ayant créé un projet vide.

Si l'option locale en premier est choisie :
```bash
mkdir MonProjet
cd MonProjet
vim README.md
git init
```
Il est indispensable de créer à minima un fichier ".gitkeep" dans un répertoire, car git n'archive pas les dossiers vides !

## Liens
- https://git-scm.com/book/fr/v2
- https://youtu.be/0sGQgfUdCAY

# Les principales commandes de git

## Connaître l'état du dépôt git local
```bash
git status
```

## Versionner les fichiers passés en paramètre ou tous les fichiers
```bash
git add Noms_Fichiers_ou_* # * pour tous
```

## Enregistrer localement les modifications ajoutées
```bash
git commit -m "Nature du commit"
```

## Transmettre au dépôt partagé les modifications commitées
```bash
git push
```

## Mettre à jour le dépôt local
```bash
    git pull
```
## Afficher les différences entre deux versions
Marche aussi sur les banches
```bash
git diff <coomit1> <commit2> # 
```

## Afficher les dernières modifications
```bash
git log 
git log --graph --online 
```

## Revenir à l'état précédent d'un fichier non encore indexé ou de se déplacer dans les branches
```bash
git checkout
```
## Manipuler les branches
```bash
git branch # Permet de manipuler les branches
```

## Fusionner des branches et de gérer des conflits
```bash
git merge
```
Une boone vidéo sur le sujet :
https://youtu.be/75ZuypqdHII

## Revenir à la troisième version précédente tout en gardant l'historique de la version actuelle
```bash
git revert HEAD~3..HEAD
```

## Revenir à une version précédente en effaçant l'historique de la version actuelle
```bash
git reset
```

## Revenir à une version précédente en effaçant l'historique de la version actuelle
```bash
git reset
```

## Alias pratique
```bash
alias gg='git log --oneline --graph --name-status'
```

# Exporter un répertoire dans un nouveau dépôt

- Accédez au répertoire principal du premier dépôt en utilisant la ligne de commande.
- Créez un nouveau dépôt Git vide en utilisant la commande `git init`.
```bash
git init nouveau-depot
```
 Cette commande crée un nouveau dépôt Git vide dans un dossier nommé `nouveau-depot`.  
- Accédez au sous-dossier `Notes` du premier dépôt en utilisant la commande cd.
```bash
cd Notes
```
- Copiez l'historique des commits de ce sous-dossier en utilisant la commande `git filter-branch`.
```bash
git filter-branch --subdirectory-filter Notes -- --all
```
 Cette commande filtre l'historique des commits pour ne conserver que ceux qui affectent le dossier `Notes`.   
- Ajoutez le nouveau dépôt Git comme dépôt distant dans le premier dépôt en utilisant la commande `git remote add`.
```bash
git remote add nouveau-depot chemin/vers/nouveau-depot
```
 Cette commande ajoute le nouveau dépôt Git comme un dépôt distant appelé `nouveau-depot` dans le premier dépôt.
   
- Poussez l'historique des commits filtré vers le nouveau dépôt en utilisant la commande `git push`.
```bash
git push nouveau-depot master
```

 Cette commande pousse l'historique des commits filtré vers le nouveau dépôt dans la branche `master`.  

Le nouveau dépôt Git créé ne contiendra que l'historique des commits concernant le sous-dossier `Notes` du premier dépôt. Les autres fichiers et dossiers du premier dépôt ne seront pas inclus dans le nouveau dépôt.

### gitlab
Gitlab évoluant sans cesse, il vaut mieux installer les paquets fournis par les développeurs. Ajout des clés de gitlab :
```bash
curl --silent https://packages.gitlab.com/gpg.key | sudo apt-key add -
apt update
apt install gitlab
```

# Export massif depuis gitlab

L'exportation de projets GitLab est généralement réalisée projet par projet via l'interface Web, ce qui peut être assez laborieux si vous avez beaucoup de projets. Cependant, pour automatiser le processus, nous pouvons utiliser l'API de GitLab.

Voici un script en Python 3 pour exporter tout le contenu de notre instance GitLab CE, le parcours ne se base pas sur la liste des projets fournis par l'API, car selon comment les  projets ont été crées, la liste retournée ne contient pas tous les projets du gitlab-ce :

```python
# Description: This script will clone all GitLab repositories from a GitLab instance.
# Author: Michael Launay
# Date: 2023-05-06
import os
import requests

# Set your GitLab URL and access token
GITLAB_HOST = "git.ecreall.com"
# For token creation, see https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html
ACCESS_TOKEN = "TOKEN FOR GITLAB-CE ROOT" # Fill with your own token
MAXIMUM_PROJECT_ID = 400 # Adapt this value to oversize your GitLab instance

# Set the directory where you want to clone the repositories
CLONE_DIR = "/home/michaellaunay/workspace"

# Create the clone directory if it doesn't exist
os.makedirs(CLONE_DIR, exist_ok=True)

# Create the base URL for the GitLab API
GITLAB_URL = f"https://{GITLAB_HOST}/api/v4/projects"

headers = {"PRIVATE-TOKEN": ACCESS_TOKEN}
# Try to access each project by ID
for i in range(1, MAXIMUM_PROJECT_ID):
    url = f"{GITLAB_URL}/{i}"
    # Retrieve the project details
    response = requests.get(f"{GITLAB_URL}/{i}", headers=headers, verify=False)
    # If the project doesn't exist, continue to the next ID
    if str(response.content, "utf-8").find("404 Project Not Found") > -1:
        continue
    print(f"Try to clone {url =}")
    project = response.json()

    # Clone each project
    project_name = project["name"]
    project_path = project["path_with_namespace"]
    namespace_path = project["namespace"]["full_path"]
    repository_url = project["ssh_url_to_repo"]

    # Clone the repository
    clone_dir = os.path.join(CLONE_DIR, project_path)
    project_dir = os.path.join(CLONE_DIR, namespace_path)
    if not os.path.exists(os.path.join(clone_dir, ".git")):
        os.makedirs(project_dir, exist_ok=True)
        os.chdir(project_dir)
        os.system(f"git clone {repository_url}")
    else:
        os.chdir(clone_dir)
        os.system(f"git pull")

print("Cloning complete. All repositories have been cloned to", CLONE_DIR)
```

Si besoin, installer le module `requests` :

```
pip install requests
```

Dans ce script Python, nous utilisons la bibliothèque `requests` pour effectuer les appels à l'API GitLab. Le script effectue les étapes suivantes :

1. Crée un répertoire pour stocker les exports.
2. Itère sur les identifiants des projets en supposant que les ids vont de 1 à 400 (à adapter).
4. Pour chaque identifiant récupère le json de description du projet.
3. S'il n'existe pas de projet pour cet identifiant passe à l'identifiant suivant.
	1. À partir des informations de nom, chemin et domaine, crée le répertoire de destination et réalise un git clone.

Si le serveur gitlab-ce n'a pas de certificat ssl à jour, il faut temporairement désactiver la vérification des certificats ssl :

```bash
git config --global http.sslVerify false
```

Remplaçons la valeur de `PRIVATE_TOKEN` par notre propre jeton d'accès privé GitLab, et `GITLAB_URL` par l'URL de notre instance GitLab.

Après exécution, tous les dépôts non vides auront été clonés.

Remarque une version à jour mais plus complexe du script est accessible à [michaellaunay/tools](https://github.com/michaellaunay/tools)

## Obtenir un token gitlab

Pour obtenir un token d'accès personnel (Personal Access Token) dans GitLab, suivons ces étapes :

1. Connectons-nous à notre compte GitLab.

2. Déplaçons nous sur le répertoire pour lequel nous souhaitons un token.

4. Cliquez sur notre avatar en haut à droite de la page, puis sur "Settings".

5. Dans le menu de gauche, cliquez sur "Access Tokens".

6. Donnons un nom à notre token, définissons une date d'expiration et sélectionnons les "scopes" (droits d'accès) que nous souhaitons attribuer à ce token. Pour utiliser l'API, vous devons cocher la case "api".

7. Cliquons sur "Create personal access token".

8. GitLab générera alors un token d'accès personnel. Assurons-nous de copier ce token et de le conserver en lieu sûr, car nous ne pourrons plus le voir une fois que nous aurons quitté cette page.

Une fois que nous avons notre token, nous pouvons l'utiliser pour nous authentifier lors de l'utilisation de l'API GitLab. Ce token est sensible et doit être gardé sécurisé.

# Git sur Android
Pour utiliser Git sur téléphone Android, suivre les étapes suivantes :

1.  Installer une application de terminal sur le téléphone, comme Termux.
    
2.  Ouvrer l'application Termux et installez Git en utilisant la commande suivante :
    
    `pkg install git`
    
3.  Créer un dossier où cloner le dépôt Git :
    
    `mkdir myproject`
    
4.  Accéder au dossier :
    
    `cd myproject`
    
5.  Cloner le dépôt Git en utilisant la commande suivante :
    
    `git clone https://github.com/notre_repo`
    
6.  Travailler sur les fichiers et les modifier comme souhaité.
    
7.  Utiliser les commandes Git en lignes de commandes.