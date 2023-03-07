#Cours #Informatique 
# Objectif
Connaître les bases de Visual Code.
# Historique
Visual code est devenu l'outil d'édition de code incontournable, nous allons voir comment l'utiliser et le configurer pour nos besoins.

# Introduction à Visual Studio Code

A. Présentation de l'interface utilisateur Dans ce premier chapitre, nous allons tout d'abord présenter l'interface utilisateur de Visual Studio Code (VS Code). Nous allons montrer où se trouvent les différents éléments de l'interface et comment les utiliser. Nous parlerons également des différents modes d'affichage, des raccourcis clavier, et des options de personnalisation de l'interface.

B. Les principales fonctionnalités de l'IDE Ensuite, nous passerons en revue les principales fonctionnalités de VS Code pour le développement en Python. Nous parlerons notamment de la coloration syntaxique, de l'indentation automatique, de l'autocomplétion, de la vérification des erreurs de syntaxe et de la suggestion de corrections, de l'intégration avec l'interpréteur Python, etc.

C. Installation et configuration de l'environnement de développement Enfin, nous verrons comment installer et configurer VS Code pour le développement en Python. Nous expliquerons comment installer les extensions nécessaires pour le développement en Python, comment installer et configurer un environnement virtuel, et comment configurer l'interpréteur Python pour qu'il soit utilisé par VS Code.

# Historique
wikipedia : "Visual Studio Code est présenté lors de la conférence des développeurs Build d'avril 2015 comme un éditeur de code [multiplateforme](https://fr.wikipedia.org/wiki/Logiciel_multiplateforme "Logiciel multiplateforme"), [open source](https://fr.wikipedia.org/wiki/Open_source "Open source") et gratuit, supportant une dizaine de [langages](https://fr.wikipedia.org/wiki/Langage_informatique "Langage informatique")[6](https://fr.wikipedia.org/wiki/Visual_Studio_Code#cite_note-6).

Il est basé sur [Electron](https://fr.wikipedia.org/wiki/Electron_(framework) "Electron (framework)"), une structure utilisée pour déployer des applications [Node.js](https://fr.wikipedia.org/wiki/Node.js "Node.js") pour le bureau exécuté sur le moteur Blink. Bien qu'il utilise le framework [Electron](https://fr.wikipedia.org/wiki/Electron_(framework) "Electron (framework)"), le logiciel n'utilise pas [Atom](https://fr.wikipedia.org/wiki/Atom_(%C3%A9diteur_de_texte) "Atom (éditeur de texte)") mais utilise le même composant éditeur (nommé Monaco) utilisé dans Azure DevOps (anciennement appelé Visual Studio Online et Visual Studio Team Services).

Le [code source](https://fr.wikipedia.org/wiki/Code_source "Code source") est fourni sous la [licence libre](https://fr.wikipedia.org/wiki/Licence_de_logiciel "Licence de logiciel") [MIT](https://fr.wikipedia.org/wiki/Licence_MIT "Licence MIT") sur le site du projet sur [Github](https://fr.wikipedia.org/wiki/GitHub "GitHub"). En revanche, l'exécutable est proposé sur le site officiel de Microsoft sous une [licence propriétaire](https://fr.wikipedia.org/wiki/Logiciel_propri%C3%A9taire "Logiciel propriétaire")[7](https://fr.wikipedia.org/wiki/Visual_Studio_Code#cite_note-7). Le projet VSCodium[8](https://fr.wikipedia.org/wiki/Visual_Studio_Code#cite_note-8) propose une compilation du logiciel sans les outils de [télémétrie](https://fr.wikipedia.org/wiki/T%C3%A9l%C3%A9m%C3%A9trie_(informatique) "Télémétrie (informatique)") inclus dans les binaires fournis par Microsoft."

wikipedia : "Visual Studio Code a été annoncé le 29 avril 2015 par Microsoft lors de la conférence Build 2015. Une version préliminaire a été publiée peu de temps après.

Le 18 novembre 2015, Visual Studio Code a été publié sous la [licence MIT](https://fr.wikipedia.org/wiki/Licence_MIT "Licence MIT") et son code source publié sur [GitHub](https://fr.wikipedia.org/wiki/GitHub "GitHub"). Le support d'extensions a également été annoncé.
"

Visual Studio Code est un éditeur de code source qui peut être utilisé avec une variété de langages de programmation, y compris C, C#, C++, Fortran, Go, Java, JavaScript, Node.js, Python, Rust. Il est basé sur le framework Electron, qui est utilisé pour développer des applications web Node.js qui s'exécutent sur le moteur de rendu Blink. Visual Studio Code utilise le même composant d'éditeur (nom de code "Monaco") utilisé dans Azure DevOps (anciennement appelé Visual Studio Online et Visual Studio Team Services).

Par défaut, Visual Studio Code inclut un support de base pour la plupart des langages de programmation courants. Ce support de base comprend la coloration syntaxique, la correspondance des parenthèses, le pliage de code et des extraits de code configurables. Visual Studio Code est également livré avec IntelliSense pour JavaScript, TypeScript, JSON, CSS et HTML, ainsi que la prise en charge du débogage pour Node.js. La prise en charge de langages supplémentaires peut être fournie par des extensions disponibles gratuitement dans le "store" de VS Code.

Au lieu d'un système de projet, il permet aux utilisateurs d'ouvrir un ou plusieurs répertoires, qui peuvent ensuite être enregistrés dans des espaces de travail pour une réutilisation future. Cela lui permet de fonctionner comme un éditeur de code sans langage spécifique pour n'importe quel langage. Il prend en charge de nombreux langages de programmation et un ensemble de fonctionnalités qui diffèrent selon le langage. Les fichiers et dossiers non désirés peuvent être exclus de l'arborescence du projet via les paramètres. De nombreuses fonctionnalités de Visual Studio Code ne sont pas exposées via des menus ou l'interface utilisateur, mais peuvent être accessibles via la palette de commandes.

Visual Studio Code peut être étendu via des extensions, disponibles dans un référentiel central. Cela inclut des ajouts à l'éditeur et une prise en charge de langage. Une fonctionnalité notable est la possibilité de créer des extensions qui ajoutent une prise en charge de nouveaux langages, des thèmes, des débogueurs, des débogueurs de voyage dans le temps, effectuent une analyse de code statique et ajoutent des linters de code en utilisant le protocole Language Server.

Le contrôle de source est une fonctionnalité intégrée de Visual Studio Code. Il a un onglet dédié dans la barre de menus où les utilisateurs peuvent accéder aux paramètres de contrôle de version et afficher les modifications apportées au projet en cours. Pour utiliser la fonctionnalité, Visual Studio Code doit être lié à n'importe quel système de contrôle de version pris en charge (Git, Apache Subversion, Perforce, etc.). Cela permet aux utilisateurs de créer des référentiels ainsi que de faire des demandes de fusion directement depuis le programme Visual Studio Code.

Visual Studio Code inclut plusieurs extensions pour FTP, permettant au logiciel d'être utilisé comme alternative gratuite pour le développement web. Le code peut être synchronisé entre l'éditeur et le serveur, sans télécharger de logiciel supplémentaire.

Visual Studio Code permet aux utilisateurs de définir la page de code dans laquelle le document actif est enregistré, le caractère de nouvelle ligne et le langage de programmation du document actif. Cela permet de l'utiliser sur

## Quelques extensions à installer

    reStructuredText (pip install snooty-lextudio, pip install sphinx sphinx-autobuild, pip install rstcheck)
    docstrings

## Paramétrer son environnement virtuel python

Créer un environnement comme indiqué en fin de section python et installer flake8 et pylint : 
    michaellaunay@Luciole:~$ workon venv #Pour activer l'environnement virtuel

Ouvrir le dossier du projet où l'on souhaite travailler. Normalement VisualCode a créé un répertoire \".vscode\" qui contient un fichier settings.json Éditer ce fichier et positionner la variable python.defaultInterpreterPath à :
    "python.defaultInterpreterPath": "${env:HOME}/.virtualenvs/venv/bin/python"

Liens :
> -   <https://code.visualstudio.com/docs/python/environments>
> -   <https://code.visualstudio.com/docs/editor/tasks#_variable-substitution>

Ne pas oublier d'installer pytest et pytest-cov :
    pip3 install pytest pytest-cov

Pour lancer les tests, on pourra alors faire :
    python3 -m unittest discover # pour unittest
    python3 -m pytest #pour pytest et donc avoir une syntaxe à base de "assert"
    python3 -m pytest --cov=perfect_maze #Pour avoir le taux de couverture des tests

Pour automatiser les tests, par exemple dans une intégration continue (CI), on peut utiliser Tox et le brancher à notre dépôt \"git\" via un hook.
> pip3 install tox

Pour le configurer, lancer \"tox-quickstart\" depuis le dossier du projet et répondre aux questions.

### Ajouter la prévisualisation sphinx pour le format rest

Pour cela il faut d'abord ajouter rest à l'environnement virtuel par défaut et le configurer : :

    (venv) michaellaunay@Luciole:~/workspace$ pip install sphinx
    (venv) michaellaunay@Luciole:~/workspace$ sphinx-quickstart

Puis éditer la configuration de l'extension (Ctrl+Shift+P) et y ajouter
comme chemin celui saisit pour Sphinx c'est-à-dire là où on a dit
quickstart de travailler et où il a créé le fichier conf.py (le
workspace est le plus pratique).