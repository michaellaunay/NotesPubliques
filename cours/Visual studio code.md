#Cours #Informatique 
# Objectif
Connaître les bases de Visual Code, car Visual code est devenu l'outil d'édition de code incontournable, et nous allons voir comment l'utiliser et le configurer pour nos besoins.

# Historique
wikipedia : "Visual Studio Code est présenté lors de la conférence des développeurs Build d'avril 2015 comme un éditeur de code [multiplateforme](https://fr.wikipedia.org/wiki/Logiciel_multiplateforme "Logiciel multiplateforme"), [open source](https://fr.wikipedia.org/wiki/Open_source "Open source") et gratuit, supportant une dizaine de [langages](https://fr.wikipedia.org/wiki/Langage_informatique "Langage informatique")[6](https://fr.wikipedia.org/wiki/Visual_Studio_Code#cite_note-6).

Le 18 novembre 2015, Visual Studio Code a été publié sous la [licence MIT](https://fr.wikipedia.org/wiki/Licence_MIT "Licence MIT") et son code source publié sur [GitHub](https://fr.wikipedia.org/wiki/GitHub "GitHub"). Le support d'extensions a également été annoncé.

VS Code est basé sur le framewok [Electron](https://fr.wikipedia.org/wiki/Electron_(framework) "Electron (framework)") qui permet de développer des applications [Node.js](https://fr.wikipedia.org/wiki/Node.js "Node.js") pour bureau et est exécuté sur le moteur de rendu [Blink](https://developer.mozilla.org/fr/docs/Glossary/Blink) . Il n'utilise pas [Atom](https://fr.wikipedia.org/wiki/Atom_(%C3%A9diteur_de_texte) "Atom (éditeur de texte)") mais le même composant éditeur (nommé Monaco) que celui utilisé dans Azure DevOps (anciennement appelé Visual Studio Online et Visual Studio Team Services).

Le [code source](https://fr.wikipedia.org/wiki/Code_source "Code source") est fourni sous la [licence libre](https://fr.wikipedia.org/wiki/Licence_de_logiciel "Licence de logiciel") [MIT](https://fr.wikipedia.org/wiki/Licence_MIT "Licence MIT") sur le site du projet sur [Github](https://fr.wikipedia.org/wiki/GitHub "GitHub"). En revanche, l'exécutable est proposé sur le site officiel de Microsoft sous une [licence propriétaire](https://fr.wikipedia.org/wiki/Logiciel_propri%C3%A9taire "Logiciel propriétaire")[7](https://fr.wikipedia.org/wiki/Visual_Studio_Code#cite_note-7). Le projet VSCodium[8](https://fr.wikipedia.org/wiki/Visual_Studio_Code#cite_note-8) propose une compilation du logiciel sans les outils de [télémétrie](https://fr.wikipedia.org/wiki/T%C3%A9l%C3%A9m%C3%A9trie_(informatique) "Télémétrie (informatique)") inclus dans les binaires fournis par Microsoft."

Visual Studio Code est un éditeur de code source pour une variété de langages de programmation, y compris C, C#, C++, Fortran, Go, Java, JavaScript, Node.js, Python, Rust.

Par défaut, Visual Studio Code inclut un support de base pour la plupart des langages de programmation courants. Ce support de base comprend la coloration syntaxique, la correspondance des parenthèses, le pliage de code et des extraits de code configurables. Visual Studio Code est également livré avec [IntelliSense](https://learn.microsoft.com/fr-fr/visualstudio/ide/using-intellisense?view=vs-2022) pour JavaScript, TypeScript, JSON, CSS et HTML, ainsi que la prise en charge du débogage pour Node.js. La prise en charge de langages supplémentaires peut être fournie par des extensions disponibles gratuitement dans le "store" de VS Code.

Au lieu d'un système de projet, il permet aux utilisateurs d'ouvrir un ou plusieurs répertoires, qui peuvent ensuite être enregistrés dans des espaces de travail pour une réutilisation future. Cela lui permet de fonctionner comme un éditeur de code sans langage spécifique pour n'importe quel langage. Il offre un ensemble de fonctionnalités qui diffèrent selon le langage. Les fichiers et dossiers non désirés peuvent être exclus de l'arborescence du projet via les paramètres. De nombreuses fonctionnalités de Visual Studio Code ne sont pas exposées via des menus ou l'interface utilisateur, mais peuvent être accessibles via la palette de commandes.

Visual Studio Code peut être étendu via des extensions, disponibles dans un référentiel central. Cela inclut des ajouts à l'éditeur et une prise en charge de langage. Une fonctionnalité notable est la possibilité de créer des extensions qui ajoutent la prise en charge de nouveaux langages, des thèmes, des débogueurs, des [débogueurs de voyage dans le temps](https://learn.microsoft.com/fr-fr/windows-hardware/drivers/debugger/time-travel-debugging-overview), effectuent une analyse de code statique et ajoutent des linters de code en utilisant le protocole Language Server.
 
Le contrôle de version du code source est une fonctionnalité intégrée de Visual Studio Code. Il a un onglet dédié dans la barre de menus où les utilisateurs peuvent accéder aux paramètres de contrôle de version et afficher les modifications apportées au projet en cours. Pour utiliser la fonctionnalité, Visual Studio Code doit être lié à n'importe quel système de contrôle de version pris en charge (Git, Apache Subversion, Perforce, etc.). Cela permet aux utilisateurs de créer des référentiels ainsi que de faire des demandes de fusion directement depuis le programme Visual Studio Code.

Visual Studio Code inclut plusieurs extensions pour FTP, permettant au logiciel d'être utilisé comme alternative gratuite pour le développement web. Le code peut être synchronisé entre l'éditeur et le serveur, sans télécharger de logiciel supplémentaire.

Visual Studio Code permet aux utilisateurs de définir la page de code dans laquelle le document actif est enregistré, le caractère de nouvelle ligne et le langage de programmation du document actif. Cela permet de l'utiliser sur

# Installation
Le plus simple est d'aller sur le site @TODO.
## Quelques extensions à installer

    reStructuredText (pip install snooty-lextudio, pip install sphinx sphinx-autobuild, pip install rstcheck)
    docstrings

# Usage
## Prévisualisation vs édition
On peut soit prévisualiser en faisant un simple clic ce qui ouvre temporairement un onglet dont le titre est en italique, soit ouvrir en édition par double clic des fichiers ou des répertoires.
La prévisualisation n'ouvre pas définitivement l'onglet dont le contenu sera remplacé par la prévisualisation suivante.

## Notification de l'édition d'un fichier
Un fichier modifié a son titre dans son onglet suivit d'un gros point et non plus de la croix de fermeture.

## Enregistrement automatique
Il est possible d'enregistrer automatiquement toute modification d'un fichier. Pour cela il suffit d'aller dans le menu **Fichier** puis de cliquer sur **Enregistrement automatique**. Dès lors toute modification du contenu est enregistrée.

## Changer le thème
Pour modifier l'apparence, il suffit de faire "Ctrl Maj P", saisir Theme et  choisir "Préférences : Thème de couleur". Nous pouvons alors soit faire  défiler les thèmes et appliquer l'un des thèmes embarqués soit en remontant la liste aller en télécharger un en ligne.

## Playground
VS Code propose un "Terrain de jeux", c'est un bac à sable permettant de tester l'application.
Il est accessible depuis le menu "Aide".

## Comprendre l'interface
Pour apprendre à connaître l'interface de VS Code, nous pouvons aller dans le menu **Aide** puis **Bienvenue**, nous avons alors accès à plein de tutoriaux et de documentations. Le plus simple est alors de démarrer un terrain de jeu.

## Les projets
Pour VS Code un répertoire est un projet qu'il appelle **workspace**. Ainsi pour gérer toutes les parties du code d'un projet (backend et frontend par exemple), il suffit d'ouvrir le dossier parent.
On peut créer une instance de VS Code dédiée à un projet en faisant les touches Ctrl Maj N.
Regarder les **Ouvrir les éléments récents** depuis le menu **Fichier** permet de passer de workspace en workspace.

## Le panneau de contrôles
Le panneau de contrôles accessible avec le raccourci "Ctrl p" permet d'afficher le moteur de recherche des commandes et donc par autocomplétion de trouver rapidement la fonctionnalité que l'on veut excécuter.

## Créer des fichiers
Pour créer un fichier, il suffit de se placer dans le répertoire voulu d'un workspace (le répertoire racine de votre projet) et de faire "Ctrl n".

## Afficher des "Previews"
Pour cela il faut installer le module "Live preview" de Microsoft.
Ensuite, depuis l'explorateur de fichier faire un clic droit et choisir l'action "Afficher l'aperçu".

# Multisélection
**Ajouter un curseur à plusieurs endroits**: Maintenez la touche `Alt` (Windows/Linux) ou `Option` (Mac) enfoncée et cliquez avec la souris aux endroits où nous souhaitons ajouter des curseurs supplémentaires.
    
**Sélectionner toutes les occurrences d'un mot**: Placez votre curseur sur le mot que nous souhaitons sélectionner et appuyez sur `Ctrl+Shift+L` (Windows/Linux) ou `Cmd+Shift+L` (Mac). Cela ajoutera un curseur à chaque occurrence du mot dans le document.
    
**Sélectionner toutes les occurrences d'une sélection**: Sélectionnez le texte que nous souhaitons rechercher, puis appuyez sur `Ctrl+Shift+H` (Windows/Linux) ou `Cmd+Shift+H` (Mac). Cela ajoutera un curseur à chaque occurrence de la sélection dans le document.
    
**Ajouter un curseur à la ligne suivante ou précédente**: Appuyez sur `Ctrl+Alt+Flèche Haut/Bas` (Windows/Linux) ou `Cmd+Option+Flèche Haut/Bas` (Mac) pour ajouter un curseur à la ligne suivante ou précédente.
    
**Ajouter un curseur à chaque ligne d'une sélection**: Sélectionnez plusieurs lignes de texte, puis appuyez sur `Shift+Alt+I` (Windows/Linux) ou `Shift+Option+I` (Mac) pour ajouter un curseur à la fin de chaque ligne sélectionnée.

## Refactoring
Lorsque l'on sélectionne un morceau de texte puis que l'on appuie sur F2 alors si l'on odifie la zone de texte la modification est reportée sur tous les fichers. C'est ce que l'on appelle le **Refactoring**.

## Paramétrer son environnement virtuel python

Créer un environnement comme indiqué en fin de section python et installer flake8 et pylint : 
    michaellaunay@Luciole:~$ workon venv #Pour activer l'environnement virtuel

Ouvrir le dossier du projet où l'on souhaite travailler.
Normalement VisualCode a créé un répertoire \".vscode\" qui contient un fichier settings.json
Éditer ce fichier et positionner la variable python.defaultInterpreterPath à :
    "python.defaultInterpreterPath": "${env:HOME}/.virtualenvs/venv/bin/python"

Liens :
> -   <https://code.visualstudio.com/docs/python/environments>
> -   <https://code.visualstudio.com/docs/editor/tasks#_variable-substitution>

Ne pas oublier d'installer pytest et pytest-cov :
    pip3 install pytest pytest-cov

Pour lancer les tests, nous pouvons alors faire :
    python3 -m unittest discover # pour unittest
    python3 -m pytest #pour pytest et donc avoir une syntaxe à base de "assert"
    python3 -m pytest --cov=perfect_maze #Pour avoir le taux de couverture des tests

Pour automatiser les tests, par exemple dans une intégration continue (CI), nous pouvons utiliser Tox et le brancher à notre dépôt \"git\" via un hook.
> pip3 install tox

Pour le configurer, lancer \"tox-quickstart\" depuis le dossier du projet et répondre aux questions.

### Ajouter la prévisualisation sphinx pour le format rest

Pour cela il faut d'abord ajouter rest à l'environnement virtuel par  défaut et le configurer : :

    (venv) michaellaunay@Luciole:~/workspace$ pip install sphinx
    (venv) michaellaunay@Luciole:~/workspace$ sphinx-quickstart

Puis éditer la configuration de l'extension (Ctrl+Shift+P) et y ajouter comme chemin celui saisit pour Sphinx c'est-à-dire là où nous avons dit à quickstart de travailler et où il a créé le fichier conf.py (le workspace est le plus pratique).