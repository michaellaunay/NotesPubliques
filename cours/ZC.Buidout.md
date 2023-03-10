#Cours #Informatique 
# Résumé
Buildout est un système de construction logiciel très bien documenté.  
Il utilise 'manuel' pour cela [https://pythonhosted.org/manuel/](https://pythonhosted.org/manuel/)  
Il déconseille de ne documenter son code qu'à travers les tests unitaires.  
On peut créer sa propre 'receipe', la tester et la documenter.  
[http://www.buildout.org/en/latest/topics/writing-recipes.html](http://www.buildout.org/en/latest/topics/writing-recipes.html)

# Plan du cours zc.buildout

1.  Introduction à zc.buildout
    -   Qu'est-ce que zc.buildout
    -   Pourquoi utiliser zc.buildout
    -   Comment zc.buildout fonctionne
2.  Installation de zc.buildout
    -   Prérequis
    -   Installation à partir de PyPI
    -   Vérification de l'installation
3.  Configuration de buildout
    -   Fichier de configuration de base
    -   Spécification des dépendances
    -   Ajout de scripts personnalisés
4.  Utilisation de buildout
    -   Exécution de buildout
    -   Mise à jour des dépendances
    -   Utilisation des scripts personnalisés
5.  avancée de buildout
    -   Utilisation de plusieurs fichiers de configuration
    -   Utilisation de sections de configuration partagées
    -   Utilisation de fichiers de configuration externes
6.  Conclusion
    -   Résumé de ce que nous avons appris
    -   Ressources supplémentaires pour en savoir plus sur zc.buildout.

# Chapitre 1: Introduction à zc.buildout

## 1.1 Qu'est-ce que zc.buildout
zc.buildout est un outil de construction de logiciels open source pour Python. Il permet de configurer, installer et gérer les dépendances de vos projets Python de manière efficace. Il peut également être utilisé pour créer des environnements de développement et de production séparés pour vos projets.

## 1.2 Pourquoi utiliser zc.buildout
Il existe de nombreux avantages à utiliser zc.buildout pour vos projets Python. Tout d'abord, il facilite la gestion des dépendances en permettant de spécifier les versions requises de chaque dépendance dans un fichier de configuration unique. Cela signifie que vous pouvez être sûr que les dépendances de votre projet fonctionneront toujours de la même manière, même si les versions des dépendances évoluent.

De plus, zc.buildout permet de créer des environnements de développement et de production séparés pour vos projets. Cela signifie que vous pouvez avoir des versions différentes de dépendances dans chaque environnement, ce qui est très utile lorsque vous travaillez sur des projets en équipe ou lorsque vous avez besoin de tester des versions différentes d'une dépendance.

### 1.3 Comment zc.buildout fonctionne
zc.buildout fonctionne en lisant un fichier de configuration appelé buildout.cfg, qui spécifie les dépendances nécessaires pour votre projet ainsi que les scripts et les commandes à exécuter. Buildout lit ensuite ce fichier de configuration et installe les dépendances nécessaires, tout en créant des scripts pour exécuter les commandes spécifiées. Cela signifie que vous n'avez pas besoin de vous soucier de l'installation manuelle des dépendances ou de la configuration de votre environnement, car tout cela est géré automatiquement par buildout.

## Chapitre 2: Installation de zc.buildout

### 2.1 Prérequis
Avant d'installer zc.buildout, vous devez avoir Python installé sur votre ordinateur. La plupart des systèmes d'exploitation modernes ont déjà Python préinstallé, mais si ce n'est pas le cas, vous pouvez le télécharger à partir du site web officiel de Python.

## 2.2 Installation à partir de PyPI
Une fois que vous avez Python installé, vous pouvez installer zc.buildout en utilisant pip, le gestionnaire de paquets de Python. Pour cela, ouvrez une invite de commande et tapez la commande suivante:

```bash
pip install zc.buildout`
```
## 2.3 Vérification de l'installation
Pour vérifier que l'installation de zc.buildout s'est déroulée correctement, ouvrez une invite de commande et tapez la commande suivante:

`buildout -v`

Si l'installation s'est déroulée correctement, vous devriez voir la version de buildout qui vient d'être installée.

Notez que pour utiliser la commande buildout, vous devez être dans un répertoire contenant un fichier buildout.cfg ou un fichier de configuration spécifique que vous avez créé pour votre projet.

## Chapitre 3: Configuration de buildout

### 3.1 Fichier de configuration de base
Avant de pouvoir utiliser buildout, vous devez créer un fichier de configuration appelé buildout.cfg qui spécifie les dépendances nécessaires pour votre projet ainsi que les scripts et les commandes à exécuter. Voici un exemple de fichier de configuration de base:

`[buildout] parts = app  [app] recipe = zc.recipe.egg eggs = mypackage`

Ce fichier de configuration spécifie une partie app qui utilise la recette zc.recipe.egg pour installer le paquet mypackage en utilisant pip.

### 3.2 Spécification des dépendances
Pour spécifier les dépendances de votre projet, vous pouvez utiliser la section eggs dans votre fichier de configuration. Par exemple, pour spécifier que votre projet a besoin de la version 2.7 de la bibliothèque requests, vous pouvez utiliser la ligne suivante dans votre fichier de configuration:

`eggs = requests==2.7`

Vous pouvez également spécifier plusieurs dépendances en les séparant par une virgule:

`eggs = requests==2.7, Flask, PyYAML`

### 3.3 Ajout de scripts personnalisés
Buildout vous permet également d'ajouter des scripts personnalisés à votre projet. Pour cela, vous devez utiliser la section scripts dans votre fichier de configuration. Par exemple, pour ajouter un script appelé "runserver" à votre projet, vous pouvez utiliser la ligne suivante dans votre fichier de configuration:

`scripts = runserver`

Ce script sera alors créé dans le répertoire bin de votre projet, et vous pourrez l'exécuter en utilisant la commande suivante:

`./bin/runserver`

Notez que pour ajouter des scripts fonctionnels, vous devriez détailler la commande ou les commandes dans un fichier python ou shell que vous devrez inclure dans votre projet.

## Chapitre 4: Utilisation de buildout

### 4.1 Installation des dépendances
Une fois que vous avez créé votre fichier de configuration, vous pouvez utiliser buildout pour installer les dépendances de votre projet. Pour cela, ouvrez une invite de commande et naviguez jusqu'au répertoire de votre projet contenant le fichier buildout.cfg. Ensuite, tapez la commande suivante:

`buildout`

Cette commande lira votre fichier de configuration et installer les dépendances spécifiées dans la section eggs. Les dépendances seront installées dans le répertoire parts de votre projet, et vous pourrez les utiliser dans votre code en utilisant l'instruction import.

### 4.2 Exécution des scripts personnalisés
Une fois que vous avez installé les dépendances de votre projet, vous pouvez utiliser buildout pour exécuter les scripts personnalisés spécifiés dans la section scripts de votre fichier de configuration. Pour cela, ouvrez une invite de commande et naviguez jusqu'au répertoire de votre projet contenant le fichier buildout.cfg. Ensuite, tapez la commande suivante:

```bash
./bin/nom_du_script
```
Remplacez "nom_du_script" par le nom du script que vous avez spécifié dans la section scripts de votre fichier de configuration.   Par exemple, si vous avez ajouté un script appelé "runserver" dans votre fichier de configuration, vous pouvez l'exécuter en utilisant la commande suivante:
```bash
./bin/runserver
```

### 4.3 Mise à jour des dépendances
Au fil du temps, les dépendances de votre projet peuvent devenir obsolètes ou il peut y avoir des nouvelles versions disponibles. Pour mettre à jour les dépendances de votre projet, vous pouvez utiliser la commande suivante:
```bash
buildout update
```
Cette commande vérifiera si des mises à jour sont disponibles pour les dépendances spécifiées dans la section eggs de votre fichier de configuration et les installer si nécessaire.  

### 4.4 Utilisation de profils
Buildout permet également l'utilisation de profils, qui sont des ensembles de configurations spécifiques pour différentes environnements (développement, production, test, etc.). Pour utiliser un profil, vous devez spécifier le nom du profil en utilisant l'option -c lors de l'exécution de la commande buildout. Par exemple, pour utiliser un profil appelé "production", vous pouvez utiliser la commande suivante:
```bash
buildout -c production
```
Cela utilisera un fichier de configuration nommé "production.cfg" si il existe, dans le même répertoire que buildout.cfg, pour configurer les dépendances et les scripts pour l'environnement de production.

Remplacez "nom_du_script" par le nom du script que vous avez spécifié dans la section scripts de votre fichier de configuration.   Par exemple, si vous avez ajouté un script appelé "runserver" dans votre fichier de configuration, vous pouvez l'exécuter en utilisant la commande suivante:
`
./bin/runserver

`
### 4.3 Mise à jour des dépendances
Au fil du temps, les dépendances de votre projet peuvent devenir obsolètes ou il peut y avoir des nouvelles versions disponibles. Pour mettre à jour les dépendances de votre projet, vous pouvez utiliser la commande suivante:
`
buildout update
`
Cette commande vérifiera si des mises à jour sont disponibles pour les dépendances spécifiées dans la section eggs de votre fichier de configuration et les installer si nécessaire.  

### 4.4 Utilisation de profils
Buildout permet également l'utilisation de profils, qui sont des ensembles de configurations spécifiques pour différentes environnements (développement, production, test, etc.). Pour utiliser un profil, vous devez spécifier le nom du profil en utilisant l'option -c lors de l'exécution de la commande buildout. Par exemple, pour utiliser un profil appelé "production", vous pouvez utiliser la commande suivante:
`
buildout -c production
`
Cela utilisera un fichier de configuration nommé "production.cfg" si il existe, dans le même répertoire que buildout.cfg, pour configurer les dépendances et les scripts pour l'environnement de production.

### 4.7 Utilisation de la commande buildout bootstrap
Avant de pouvoir utiliser buildout pour installer les dépendances de votre projet, vous devez d'abord installer buildout lui-même sur votre ordinateur. Pour ce faire, vous pouvez utiliser la commande buildout bootstrap. Cette commande télécharge et installe la dernière version de buildout sur votre ordinateur. Vous n'avez besoin de l'utiliser qu'une seule fois, au début de votre projet. La commande à utiliser est :

`python3 bootstrap.py`

### 4.8 Utilisation de la commande buildout annotate
La commande buildout annotate est utilisée pour générer une liste des dépendances utilisées par le projet. Cela peut être utile pour comprendre les dépendances utilisées par le projet et pour identifier les dépendances qui ne sont plus utilisées. Pour utiliser cette commande, vous devez exécuter la commande suivante :

`buildout annotate`

En résumé, zc.buildout est un outil puissant pour gérer les dépendances et automatiser les tâches de configuration et de déploiement pour les projets Python. En utilisant les fonctionnalités de buildout, vous pouvez configurer efficacement votre projet pour satisfaire vos besoins spécifiques et automatiser les tâches de déploiement et de configuration.