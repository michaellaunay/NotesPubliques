---
schema_version: 1
uid: 01M1BQ62D39PKWNMPFYCQ1D5M0
titre: "Pyramid — 01 — Introduction à Pyramid et au développement web Python"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - developpement-web
  - python
  - pyramid
resume: "Chapitre 1 sur 10 du livre « Pyramid » : Introduction à Pyramid et au développement web Python. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-06-14
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « Pyramid » — chapitre 1/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — Sommaire|← Sommaire]] · [[Pyramid — 02 — Les Routes et Vues dans Pyramid|02 — Les Routes et Vues dans Pyramid →]]

# 1. Introduction à Pyramid et au développement web Python
Objectifs:
- Comprendre ce qu'est Pyramid et son positionnement parmi les autres frameworks Python.
- Installer et configurer un environnement de développement Pyramid.
- Construire une application Pyramid simple.

## 1.1 Introduction à Pyramid

### 1.1.1 Qu'est-ce que Pyramid ?

Pyramid est un cadre de développement ("framework") web en Python, tout comme Django ou Flask, mais il se distingue par sa flexibilité et son minimalisme. Le slogan de Pyramid est "Commencez petit, terminez grand", ce qui signifie que nous pouvons utiliser Pyramid pour construire des applications simples et petites, mais aussi des applications web complexes et performantes.

### 1.1.2 Historique de Pyramid et du projet Pylons

Pyramid, initialement appelé "repoze.bfg", est le successeur du framework Pylons datant de 2005. Il a rapidement remplacé le framework Pylons des projets Pylons qui l'hébergeaient. C'est pourquoi nous trouverons souvent des références au "Projet Pylons" et au "framework Pylons" dans la documentation de Pyramid, en raison de cet héritage.

Initialement "repoze.bfg" faisait partie du projet Repoze, qui visait à apporter les technologies et les concepts du monde Zope/Plone au reste de la communauté Python.

Pyramid a été conçu pour surmonter certaines des limitations de Pylons, en étant minimaliste et en permettant aux développeurs d'ajouter uniquement les composants nécessaires pour leurs applications, sans pour autant restreindre le choix.

Pyramid permet aux développeurs de choisir parmi une variété de modèles, de systèmes de stockage de données et de systèmes d'authentification.

C'est en 2011 que repoze.bfg a été rebaptisé Pyramid et est devenu le cadre de développement ("framework") principal du projet Pylons. Depuis lors, le développement du framework Pylons a été interrompu et toute l'attention s'est tournée vers Pyramid.

Depuis sa création, Pyramid a été utilisé pour développer une grande variété d'applications, allant de petites applications web à de vastes applications d'entreprise.

### 1.1.3 Pourquoi utiliser Pyramid ?

Il y a plusieurs raisons pour lesquelles nous pourrions choisir d'utiliser Pyramid pour notre projet :

1. **Flexibilité** : Contrairement à certains autres frameworks, Pyramid ne nous oblige pas à utiliser un certain ensemble d'outils ou de bibliothèques. nous pouvons choisir ceux qui conviennent le mieux à notre projet.

2. **Évolutivité** : Pyramid est conçu pour être capable de gérer à la fois des applications simples et petites, et des applications très complexes et de grande taille.

3. **Simplicité** : Bien que Pyramid soit capable de gérer des applications complexes, il reste simple à utiliser pour les applications plus simples. Il est également relativement facile à apprendre, surtout si nous avons déjà une certaine expérience avec Python.

### 1.1.4 Où se situe Pyramid par rapport aux autres frameworks Python ?

Si nous comparons Pyramid à d'autres frameworks populaires comme Django et Flask, on pourrait dire que Pyramid se situe quelque part entre les deux. Flask est souvent décrit comme un "micro" framework, ce qui signifie qu'il est minimaliste et laisse beaucoup de liberté à l'utilisateur, tandis que Django est un framework "batteries included", ce qui signifie qu'il fournit une grande quantité de fonctionnalités intégrées. Pyramid, quant à lui, se situe entre les deux : il est plus riche en fonctionnalités que Flask, mais moins prescriptif que Django.

### 1.1.5 Architecture de Pyramid

L'architecture de Pyramid est basée sur le modèle de conception "colle et outils". Cela signifie que Pyramid fournit les outils de base pour construire une application web, mais il nous laisse la liberté de choisir comment nous voulons les assembler. Les composants de base d'une application Pyramid sont les "routes", qui définissent comment les URL sont traduites en actions, et les "vues", qui génèrent les réponses aux requêtes.

## 1.2 Installation et configuration de Pyramid

### 1.2.1 Installation de Python et de l'environnement virtuel

Tout d'abord, nous avons besoin de Python pour développer avec Pyramid. Python est le langage de programmation sur lequel Pyramid est construit. Pour installer Python, s'il n'est pas déjà présent sur notre machine ou si la version est trop ancienne, rendons-nous sur le site officiel de Python (https://www.python.org/) et téléchargeons la dernière version. Assurons-nous que Python soit bien installé en ouvrant une console ou un terminal et en tapant `python --version`.

Maintenant que nous avons Python, nous allons installer un environnement virtuel. Un environnement virtuel est un espace isolé où nous pouvons installer les dépendances de notre projet sans interférer avec les autres projets sur notre machine. nous pouvons installer l'environnement virtuel en utilisant la commande suivante :

```bash
python -m venv myenv
```

Ceci créera un nouvel environnement virtuel dans un dossier nommé `myenv`. Pour activer cet environnement, utilisons la commande :

- Sur Windows : `myenv\Scripts\activate`
- Sur Unix ou MacOS : `source myenv/bin/activate`

### 1.2.2 Installation de Pyramid

Pyramid peut être installé en utilisant `pip`, le gestionnaire de paquets Python :

```bash
pip install pyramid
```

### 1.2.3 Installation de Cookiecutter

La communauté Pyramide fournit des modèles de projets utilisables avec l'outil [cookiecutter](https://github.com/cookiecutter/cookiecutter)

Cookiecutter est un outil en ligne de commande de création de projets à partir de modèles pré-existants, appelés "cookiecutters". Il nous permet de définir des valeurs par défaut et des variables personnalisables pour nos projets.

Dans le contexte de Pyramid, le projet Pylons propose plusieurs modèles cookiecutter que nous pouvons utiliser pour générer un projet. Ces modèles facilitent la mise en place de la structure de notre application, nous permettant de nous concentrer sur la logique de notre application plutôt que sur l'aspect configuration.

Pour installer Cookiecutter
```bash
pip install cookieCutter
```

### 1.2.4 Création d'un Projet Pyramid avec Cookiecutter

Le projet Pylons propose plusieurs modèles de cookiecutter. Chaque modèle fait différentes suppositions sur le type d'application que nous essayons de créer. Par exemple, il existe des modèles pour SQLAlchemy avec SQLite, ou ZODB comme mécanisme de persistance, ou encore différentes bibliothèques de templates comme Jinja2, Chameleon, ou Mako.

Pour générer un nouveau projet Pyramid, nous utiliserons la commande `cookiecutter` suivie de l'URL du dépôt de la recette cookiecutter correspondant au type de projet que nous souhaitons créer.
Par exemple, pour créer un projet, il faut utiliser cookiecutter "pyramid-cookiecutter-starter" :

```bash
#Dans l'environnement virtuel
cookiecutter gh:Pylons/pyramid-cookiecutter-starter
```

Ensuite, Cookiecutter nous posera une série de questions pour configurer notre projet. Par exemple, il nous demandera le nom du projet, le nom du dépôt, et le langage de template à utiliser. nous détaillons ci après ces variables. Pour la plupart de ces questions, nous pouvons simplement appuyer sur Entrée pour accepter la valeur par défaut.

Une fois que nous avons répondu à toutes les questions, Cookiecutter créera un nouveau répertoire avec le même nom que le nom du projet que nous avons donné. Ce répertoire contient toute la structure de base de notre projet Pyramid, y compris la configuration, les fichiers de démarrage et le squelette de notre application.

À partir de là, nous pouvons commencer à développer notre application Pyramid. nous pouvons activer notre environnement virtuel, installer les dépendances de notre projet avec `pip install -e .`, et démarrer le serveur de développement avec `pserve`.

### 1.2.5 Exemple de création d'un projet avec Cookiecuter

Création d'un environnement de développement.
```bash
python3 -m venv pyramid
```

Activation de l'environnement.

```bash
source pyramid/bin/activate
```

Mise à jour de celui-ci.

```bash
pip install --upgrade pip setuptools
```

Installation de  Pyramid.

```bash
pip install pyramid
```

Installation de Cookicutter.

```bash
pip install cookiecutter
cookiecutter gh:Pylons/pyramid-cookiecutter-starter
```

Ici on sélectionne, les template Chamelon et la base de données objet ZODB.

```
project_name: MonSuperProjet
repo_name: mon_super_projet
Select template_language:
2 - chameleon
Choose from 1, 2, 3 [1]: 2
Select backend:
3 - zodb
Choose from 1, 2, 3 [1]: 3
```

Création de l'environnement de test.

```bash
bin/pip install -e ".[testing]"
```

Test d'exécution.

```bash
bin/pytest
```
Le résultat est un nombre de tests passés, des warnings, mais aucune erreur.
Le fichier `pytest.ini` permet de configurer `pytest`, ainsi 
```ini
[pytest]
addopts = --strict-markers

testpaths =
    alirpunkto
    tests

log_cli = true
log_cli_level = DEBUG
log_cli_format = %(asctime)s [%(levelname)s] %(message)s (%(pathname)s:%(lineno)d)
log_cli_date_format = %Y-%m-%d %H:%M:%S
```
Permet de configurer pytest pour qu'il affiche les logs sur la console pendant l'exécution des tests, tout en utilisant le niveau DEBUG


Lançons l'exécution du serveur.
```bash
bin/pserve development.ini
```

### 1.2.6 Signification des variables Cookicutter
Lorsque nous exécutons `cookiecutter gh:Pylons/pyramid-cookiecutter-zodb`, un certain nombre de variables sont demandées. Voici ce qu'elles signifient :

1. `repo_name`: Il s'agit du nom du répertoire dans lequel notre projet sera créé. Par convention, ce nom est souvent le même que le nom de notre projet, mais en minuscules et sans espaces ni caractères spéciaux. Il est également utilisé comme le nom de notre dépôt si nous décidons de pousser notre projet vers un système de contrôle de version comme GitHub.
2. `project_name`: Il s'agit du nom formel de notre projet, tel qu'il apparaîtra dans la documentation, les messages de log, etc. Contrairement à `repo_name`, `project_name` peut contenir des espaces et des caractères spéciaux.
3. `package_name`: Il s'agit du nom du paquet Python principal de notre projet. C'est le nom que nous utiliserons pour importer notre code dans d'autres fichiers Python. Par convention, ce nom est souvent le même que `repo_name`.
4. `namespace`: C'est le nom du namespace pour notre projet. Les namespaces sont une fonctionnalité de Python qui permet de regrouper des packages logiquement liés. Il est courant que le nom du namespace soit le même que le nom du projet.
5. `author`: C'est le nom de l'auteur du projet. Il sera utilisé dans les fichiers de métadonnées du projet.
6. `author_email`: C'est l'adresse e-mail de l'auteur du projet. Elle sera également utilisée dans les fichiers de métadonnées du projet.
7. `url`: Il s'agit de l'URL du site Web du projet ou de la page du projet sur un système de contrôle de version comme GitHub.
8. `license_name`: Il s'agit de la licence sous laquelle notre projet sera distribué.
9. `python_version`: C'est la version de Python que nous comptons utiliser pour notre projet. 
10. `pyramid_version`: Il s'agit de la version de Pyramid que nous comptons utiliser pour notre projet.
11. `zodb_version`: Il s'agit de la version de ZODB que nous comptons utiliser pour notre projet.
12. `description`: Il s'agit d'une courte description de notre projet qui sera utilisée dans les fichiers de métadonnées du projet.

### 1.2.7 Configuration de l'environnement de développement

Pour développer une application Pyramid, nous pouvons utiliser n'importe quel éditeur de texte ou environnement de développement intégré (IDE) de notre choix. Certains des IDE populaires pour le développement Python incluent PyCharm, [[Visual studio code]], et Atom. Choisissons celui avec lequel nous sommes le plus à l'aise.

En plus de l'IDE, il est important de configurer les outils de débogage et de test. Pour le débogage, Pyramid est livré avec un débogueur intégré que nous pouvons activer dans notre fichier de configuration. Pour les tests, nous pouvons utiliser `pytest`, un framework de test populaire pour Python.

## 1.3 Structure de base d'une application Pyramid

Avant de créer notre premier projet nous allons survoler l'arborescence des projets Pyramid puis nous utiliserons un utilitaire pour créer notre arborescence.

### 1.3.1 Comprendre la structure de base d'une application Pyramid

Un projet Pyramid typique est organisé en différents fichiers et dossiers pour séparer les préoccupations et rendre le projet plus maintenable. Voyons quels sont les composants typiques de l'architecture de projet.

1. **Fichier de configuration (.ini)** : Ce fichier est crucial car il contient toutes les configurations de notre application. Il peut définir les paramètres du serveur, les paramètres de débogage, et tout autre paramètre que notre application pourrait avoir besoin. Pyramid utilise généralement deux fichiers de configuration : `development.ini` pour le développement et `production.ini` pour la production.

2. **Fichier d'application (`__init__.py`)** : Il s'agit du point d'entrée de notre application. Ce fichier est généralement utilisé pour configurer et créer une instance de l'application Pyramid.

3. **Dossier des vues (`views/`)** : Ce dossier contient tous les fichiers de vue de notre application. Les vues sont des fonctions ou des méthodes qui sont appelées en réponse à une requête HTTP.

4. **Dossier des modèles (`models/`)** : Ce dossier contient tous nos modèles de données. Les modèles représentent les données de notre application et définissent comment interagir avec notre base de données.

5. **Dossier des templates (`templates/`)** : Ce dossier contient tous les templates de notre application. Les templates sont des fichiers qui définissent la structure de la sortie HTML.

### 1.3.2 Étude de la structure d'une application Pyramid simple

Pour mieux comprendre ces composants, créons une application Pyramid simple. Cette application aura une seule route (`/`) qui répondra avec un simple "Hello, World!".

1. Créons un nouvel environnement virtuel et installons Pyramid comme nous l'avons ci-dessus.

2. Créons un nouveau projet Pyramid en utilisant le gabarit de départ "starter" fourni par Pyramid. nous pouvons le faire en exécutant la commande :

   ```bash
   pcreate -s starter hello_world
   ```

3. Si nous examinons le contenu du dossier `hello_world`, nous trouverons plusieurs fichiers et dossiers. Pour l'instant, concentrons-nous sur `development.ini`, `hello_world/__init__.py`, et `hello_world/views.py`.

   - `development.ini` est notre fichier de configuration. nous pouvons voir qu'il contient plusieurs paramètres, dont certains sont spécifiques à Pyramid.
   
   - `hello_world/__init__.py` contient la fonction `main()`, qui est le point d'entrée de notre application. nous pouvons voir qu'elle configure une route et renvoie une instance de l'application.
   
   - `hello_world/views.py` contient notre vue, qui est une simple fonction qui renvoie "Hello, World!".


## 1.4 notre première application Pyramid

Construction d'une application Pyramid "Hello, World!"

### 1.4.1 Configuration des routes

   Comme nous le savons déjà, les routes sont une composante essentielle de toute application Pyramid. Elles définissent comment les URL sont mappées aux vues. Dans notre application "Hello, World!", nous aurons besoin d'une seule route qui mappera l'URL de base (`/`) à notre vue. nous pouvons le faire dans notre fonction `main()` dans `__init__.py` :

   ```python
   config.add_route('home', '/')
   ```

### 1.4.2 Création des vues

   Les vues sont des fonctions ou des méthodes qui sont appelées en réponse à une requête HTTP. Dans notre cas, nous avons besoin d'une seule vue qui renvoie "Hello, World!". nous pouvons le faire dans notre fichier `views.py` :

   ```python
   from pyramid.view import view_config

   @view_config(route_name='home', renderer='string')
   def home(request):
       return "Hello, World!"
   ```

   Notons l'utilisation du décorateur `@view_config`. Ce décorateur est utilisé pour associer notre vue à la route que nous avons définie précédemment. Le paramètre `renderer='string'` indique que notre vue renvoie une simple chaîne de caractères.

### 1.4.3 Exécution de l'application

   nous pouvons maintenant exécuter notre application pour voir si tout fonctionne comme prévu. Pour ce faire, utilisons la commande suivante :
   ```bash
   pserve development.ini
   ```

   Si tout se passe bien, notre serveur sera en cours d'exécution. Ouvrons notre navigateur et allons à `http://localhost:6543/`. nous devrions voir "Hello, World!".

### 1.5 Révision et exercices pratiques

### 1.5.1 Révision des concepts clés

Dans ce chapitre, nous avons abordé plusieurs concepts clés en lien avec le développement d'applications web avec le framework Pyramid :

1. **Le framework Pyramid** : Nous avons introduit Pyramid comme un framework web Python flexible et minimaliste, qui peut être utilisé pour développer des applications web de petite à grande échelle.

2. **L'installation et la configuration de Pyramid** : Nous avons expliqué comment préparer un environnement de développement pour Pyramid, y compris l'installation de Python et de Pyramid, la configuration d'un environnement virtuel, et la configuration de l'IDE et des outils de débogage.

3. **La structure de base d'une application Pyramid** : Nous avons exploré la structure d'une application Pyramid typique, y compris les fichiers de configuration, les vues, les modèles, et les templates.

4. **La création d'une application Pyramid** : Nous avons expliqué comment construire une application Pyramid simple, y compris la configuration des routes, la création des vues, et l'exécution de l'application.

### 1.5.2 Exercices pratiques : Création d'une application simple avec différentes routes et vues

Pour renforcer notre compréhension de ces concepts, nous allons maintenant créer une application Pyramid simple avec plusieurs routes et vues. Cette application sera une application de blog simple avec deux pages : une page d'accueil qui liste tous les billets de blog, et une page de billet de blog qui affiche un billet spécifique.

1. **Configuration des routes**

   nous aurons besoin de deux routes pour notre application : une pour la page d'accueil (`/`) et une pour la page du billet de blog (`/blog/{id}`). nous pouvons configurer ces routes dans notre fonction `main()`.

2. **Création des vues**

   nous aurons également besoin de deux vues pour notre application : une pour la page d'accueil et une pour la page du billet de blog. nous pouvons créer ces vues dans notre fichier `views.py`.

3. **Exécution de l'application**

   Comme précédemment, nous pouvons exécuter notre application avec la commande `pserve development.ini`.

Prenons le temps de développer cette application en nous basant sur ce que nous avons appris cette semaine.

---
> [!info] Livre « Pyramid » — chapitre 1/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — Sommaire|← Sommaire]] · [[Pyramid — 02 — Les Routes et Vues dans Pyramid|02 — Les Routes et Vues dans Pyramid →]]
