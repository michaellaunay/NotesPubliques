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