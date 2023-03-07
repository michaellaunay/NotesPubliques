Git est un outil de gestion de versions du code développé par Linus Torval fondateur de Linux pour gérer le \"versioning\" du noyau Linux à partir de 2005.

Pour installer le client :
    apt install git

Pour configurer son compte :
    git config user.name Michaël Launay
    git config user.mail michaellaunay@ecreall.com

Connaître sa configuration :
    git config --list

La configuration se trouve soit dans :
    "/etc/gitconfig"
    "~/.gitconfig"
    <rep_projet>/.git/config

Liens :
    https://git-scm.com/book/fr/v2/Personnalisation-de-Git-Configuration-de-Git
    https://youtu.be/KVULgIbyeQs

Créer un projet :
    Soit directement depuis github ou gitlab, puis faire un "git clone", soit créer un porjet local et l'importer dans github/gitlab en ayant créé un projet vide.

    Si l'option locale en premier est choisie : ::

      mkdir MonProjet
      cd MonProjet
      vim README.md
      git init

    Il est indispensable de créer à minima un fichier ".gitkeep" dans un répertoire, car git n'archive pas les dossiers vides !

Liens : :

    * https://git-scm.com/book/fr/v2
    * https://youtu.be/0sGQgfUdCAY

Les commandes principales : :

    git status # Donne l'état du dépôt git local
    git add Fichiers_ou_* # Versionne les fichiers passés en paramètre ou tous les fichiers
    git commit -m "Nature du commit" # Enregistre localement les modifications ajoutées
    git push # Transmet au dépôt partagé les modifications commitées
    git pull # Met à jour le dépôt local
    git diff <coomit1> <commit2> # Affiche les différences entre deux versions, marche aussi sur les banches
    git log # Affiche les dernière modification, "git log --graph --online "
    git checkout # Permet de revenir à l'état précédent d'un fichier non encore indexé ou de se déplacer dans les branches
    git branch # Permet de manipuler les branches
    git merge # Permet de fusionner des branches et de gérer des conflits
    git revert HEAD~3..HEAD # permet de revenir à la troisième version précédente tout en gardant l'historique de la version actuelle
    git reset # permet de revenir à une version précédente en effaçant l'histprique de la version actuelle

Un alias pratique : :

    alias gg='git log --oneline --graph --name-status'

### gitlab
Gitlab évoluant sans cesse, il vaut mieux installer les paquets fournis par les développeurs. Ajout des clés de gitlab : :

    curl --silent https://packages.gitlab.com/gpg.key | sudo apt-key add -
    apt update
    apt install gitlab