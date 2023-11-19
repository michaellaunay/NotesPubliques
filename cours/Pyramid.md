Plan de cours structuré autour du développement d'une application d'authentification de membres utilisant Pyramid et OpenLDAP.

**1. Introduction à Pyramid et au développement web Python**
- Introduction à Pyramid : qu'est-ce que c'est, pourquoi l'utiliser.
- Historique de Pyramid et du projet Pylons.
- Installation et configuration de Pyramid.
- Structure de base d'une application Pyramid.
- notre première application Pyramid.

**2. Les Routes et Vues dans Pyramid**
- Comprendre le mécanisme de routage dans Pyramid.
- Créer et utiliser des vues.
- Utilisation des templates Chameleon pour le rendu des vues.

**3. Gestion des requêtes et réponses**
- Comprendre les requêtes GET et POST.
- Manipulation des données de la requête.
- Envoyer des réponses : différences entre les types de réponses et quand les utiliser.
- Utilisation des cookies pour maintenir l'état de la session.

**4. Introduction à l'authentification**
- Qu'est-ce que l'authentification et pourquoi est-elle importante ?
- Les mécanismes d'authentification dans Pyramid.
- Créer une politique d'authentification simple.

**5. Sécurisation des données**
- Comment protéger les données des utilisateurs.
- Hashage des mots de passe.
- Utilisation de pyramid.csrf pour la prévention des attaques CSRF.
- Validation et assainissement des entrées des utilisateurs.

**6. Introduction à LDAP et OpenLDAP**
- Qu'est-ce que LDAP ?

**7. Authentification LDAP**
- Interagir avec un serveur LDAP en Python.
- Intégration d'OpenLDAP avec Pyramid pour l'authentification.
- Gestion des erreurs d'authentification.

**8. Déploiement de l'application**
- Introduction à Docker.
- Création d'un Dockerfile pour l'application Pyramid.
- Déploiement de l'application en utilisant Docker.
- Surveillance et dépannage de l'application déployée.

**9. Pyramide et Docker**
- Dockerfile pour Pyramid

**10. Exemple de code**
- Stockage d'une session en ZODB
- Coupler l'authentification avec OpenLDAP
- Vérification des certificats

**11.  Trucs et astuces **
- Débogage en ligne à travers le navigateur

Ce plan de cours vise à donner une compréhension complète de la création d'une application d'authentification en utilisant Pyramid et OpenLDAP, du développement à la mise en production.

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

# 2 Les Routes et Vues dans Pyramid

## 2.1 Introduction aux routes dans Pyramid

### 2.1.1 Qu'est-ce qu'une route ?

Une route est essentiellement un moyen de définir comment les requêtes HTTP sont traitées par notre application. Chaque route correspond à une URL ou à un motif d'URL, et à une vue qui est appelée lorsque l'URL est demandée par un navigateur. 

### 2.1.2 Définir des routes dans Pyramid

Définir des routes dans Pyramid est assez simple. nous pouvons le faire dans le fichier `__init__.py` de notre application. Par exemple, pour définir une route pour l'URL de base (`/`), nous pouvons ajouter le code suivant à notre fonction `main()` :
```python
config.add_route('home', '/')
```

Ici, `home` est le nom de la route et `/` est le motif d'URL de la route. Ce motif est utilisé pour déterminer si la route doit être utilisée pour une requête donnée.

### 2.1.3 Pattern matching dans les routes

Parfois, nous voulons définir des routes qui correspondent à plusieurs URL. Pyramid nous permet de le faire en utilisant des variables de routage. Par exemple, nous pouvons définir une route qui correspond à toute URL de la forme `/blog/{id}` en utilisant le code suivant :
```python
config.add_route('blog', '/blog/{id}')
```

Ici, `{id}` est une variable de routage. Lorsque Pyramid voit une URL qui correspond au motif, il extrait la partie correspondante de l'URL et la stocke dans la variable `id`. nous pouvons ensuite accéder à cette variable dans notre vue.

### 2.1.4 Routes statiques et dynamiques

Pyramid nous permet de définir à la fois des routes statiques et des routes dynamiques. Les routes statiques correspondent à une URL fixe, tandis que les routes dynamiques peuvent correspondre à plusieurs URL en fonction des variables de routage.

Par exemple, `/about` est une route statique, car elle correspond toujours à l'URL `/about`. D'autre part, `/blog/{id}` est une route dynamique, car elle peut correspondre à des URL comme `/blog/1`, `/blog/2`, etc.

## 2.2 Gestion des vues dans Pyramid

### 2.2.1 Qu'est-ce qu'une vue dans Pyramid ?

Une vue dans Pyramid est une fonction ou une méthode Python qui reçoit une instance de requête en tant que paramètre et renvoie une réponse. Les vues sont associées aux routes de notre application, c'est-à-dire qu'une vue est appelée lorsque la route correspondante est sollicitée par une requête HTTP.

### 2.2.2 Création de vues simples

Pour créer une vue, nous écrivons simplement une fonction Python. Par exemple, nous pouvons créer une vue qui renvoie "Hello, World!" comme suit :
```python
def hello_world(request):
    return Response('Hello, World!')
```

Dans cet exemple, `request` est l'objet de requête que Pyramid passe à notre vue, et `Response('Hello, World!')` est la réponse HTTP que notre vue renvoie.

### 2.2.3 Association des vues aux routes

Pour qu'une vue soit appelée, elle doit être associée à une route. nous pouvons le faire en utilisant le décorateur `view_config` et en spécifiant le nom de la route. Par exemple, nous pouvons associer la vue `hello_world` à la route 'home' comme suit :
```python
from pyramid.view import view_config

@view_config(route_name='home')
def hello_world(request):
    return Response('Hello, World!')
```

Dans cet exemple, lorsque la route 'home' est sollicitée, la fonction `hello_world` est appelée.

### 2.2.4 Utilisation des décorateurs de vues

Pyramid fournit plusieurs décorateurs que nous pouvons utiliser pour contrôler le comportement de nos vues. Par exemple, le décorateur `view_config` peut prendre plusieurs arguments qui contrôlent comment la vue est rendue, quel type de requêtes elle peut traiter, etc. nous approfondirons ces options plus tard dans ce cours.

## 2.3 Approfondissement des routes dans Pyramid

### 2.3.1 Utilisation des générateurs d'URL

Dans Pyramid, nous pouvons utiliser des générateurs d'URL pour créer des URL à partir des noms de nos routes. Par exemple, si nous avons une route nommée 'blog' qui correspond à l'URL '/blog/{id}', nous pouvons créer une URL pour cette route comme suit :
```python
url = request.route_url('blog', id=1)
```

Dans cet exemple, `url` sera '/blog/1'. Les générateurs d'URL sont particulièrement utiles lorsque nous devons créer des liens dans nos templates ou rediriger l'utilisateur vers une autre page.

### 2.3.2 Gestion des erreurs 404 avec le système de routage

Parfois, nous voulons personnaliser la page d'erreur 404 de notre application. Pyramid nous permet de le faire en utilisant une vue "not found". nous pouvons créer une telle vue en utilisant le décorateur `notfound_view_config`. Par exemple :

```python
from pyramid.view import notfound_view_config
from pyramid.response import Response

@notfound_view_config(renderer='404.pt')  # .pt est l'extension habituelle pour les templates Chameleon
def notfound_view(request):
    request.response.status = 404
    return {}

```
Qui sera associée à la "pagetemplate":

```html
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:tal="http://xml.zope.org/namespaces/tal">
  <body>
    <h1 tal:content="'Page non trouvée'">Page non trouvée</h1>
    <p>Nous sommes désolés, mais la page que nous cherchonsn'existe pas.</p>
  </body>
</html>
```
Dans cet exemple, lorsque l'utilisateur demande une URL qui ne correspond à aucune route, la vue `notfound_view` est appelée et une page 404 personnalisée est rendue.

### 2.3.3 Préfixes de routes

Parfois, nous voulons ajouter un préfixe commun à plusieurs routes. Par exemple, nous pouvons avoir plusieurs routes qui commencent toutes par '/api'. Pyramid nous permet de le faire en utilisant la méthode `config.include()`. Par exemple :
```python
config.include('myproject.api', route_prefix='/api')
```

Dans cet exemple, toutes les routes définies dans 'myproject.api' auront '/api' comme préfixe.

### 2.3.4 Utilisation de la fonction `add_route` pour ajouter des routes

La méthode `add_route` est le moyen le plus courant d'ajouter des routes dans Pyramid. Elle prend deux paramètres principaux : le nom de la route et le motif de l'URL. Par exemple :

```python
config.add_route('home', '/')
```

## 2.4 Manipulation des vues dans Pyramid

### 2.4.1 Passage de données aux vues

Dans Pyramid, nous pouvons passer des données à une vue en utilisant le dictionnaire de requête `request.matchdict`. Par exemple, si nous avons une route avec une variable `id`, nous pouvons récupérer cette valeur dans la vue associée comme suit :

```python
@view_config(route_name='blog')
def blog_view(request):
    blog_id = request.matchdict['id']
    # Faire quelque chose avec blog_id
    ...
```

### 2.4.2 Retour de réponses HTTP personnalisées depuis les vues

Nous pouvons également retourner des réponses HTTP personnalisées depuis nos vues en Pyramid. Pour cela, nous utilisons l'objet `Response`. Par exemple :

```python
from pyramid.response import Response

@view_config(route_name='blog')
def blog_view(request):
    return Response('Voici le blog', status_code=200)
```

Dans cet exemple, la vue `blog_view` renvoie une réponse HTTP avec le corps 'Voici le blog' et le code de statut 200.

### 2.4.3 Utilisation des décorateurs de vues pour la gestion des erreurs

Nous avons déjà vu le décorateur `view_config`, mais Pyramid offre également un autre décorateur utile pour la gestion des erreurs : `exception_view_config`. Ce décorateur nous permet de définir des vues qui seront appelées lorsqu'une exception spécifique est levée. Par exemple :
```python
from pyramid.view import exception_view_config
from pyramid.response import Response

@exception_view_config(ValueError)
def value_error_view(exc, request):
    return Response('Une erreur est survenue : {}'.format(exc), status_code=500)
```

Dans cet exemple, chaque fois qu'une `ValueError` est levée dans notre application, la vue `value_error_view` sera appelée, et un message d'erreur personnalisé sera renvoyé.

## 2.5 Introduction aux templates Chameleon pour le rendu des vues

### 2.5.1 Qu'est-ce que Chameleon et pourquoi est-il utilisé avec Pyramid?

Chameleon est un moteur de templates pour Python. Il est flexible, rapide, et conçu pour générer du HTML/XML. Dans Pyramid, Chameleon est utilisé pour faciliter le rendu des vues, ce qui permet de séparer la logique de présentation de la logique métier de notre application.

Chameleon offre une syntaxe riche qui s'appuie sur les standards XML (ZPT, TAL, TALES, METAL) et est donc idéal pour ceux qui sont familiers avec ces technologies. Cependant, il est également intuitif pour ceux qui découvrent ces outils.

### 2.5.2 Comment utiliser Chameleon pour créer des templates

Pour créer un template Chameleon, nous créons un fichier avec l'extension .pt (par exemple, index.pt). nous pouvons alors utiliser la syntaxe de Chameleon pour définir la structure de notre page. Par exemple:

```html
<html>
  <body>
    <h1 tal:content="title">A futur title</h1>
    <p>$Description : ${description}</p>
  </body>
</html>
```

Dans cet exemple, `tal:content="title"` et `${description}` sont des expressions Python, ici deux noms de variable, qui seront évaluées et  remplacées, pour le paramètre `tal:content` ce sera le contenu de la balise, pour ${description} ce sera cette variable uniquement. Les valeurs de substitution sont passées à notre `template` par le "renderer".
Voir [documentation Chameleon](https://chameleon.readthedocs.io/en/latest/reference.html)

### 2.5.3 Introduction à l'utilisation des templates TAL/METAL

Pour commencer à utiliser Chameleon avec Pyramid, nous devons d'abord l'installer. Utilisons `pip` pour installer le package `pyramid_chameleon` :

```python
pip install pyramid_chameleon
```

Une fois installé, nous devons indiquer à Pyramid que nous allons utiliser Chameleon comme système de templates. Dans notre fonction principale, nous ajoutons une ligne pour inclure `pyramid_chameleon` :

```python
if __name__ == '__main__':
    with Configurator() as config:
        config.include('pyramid_chameleon')
        # le reste de notre code
```

Maintenant, supposons que nous ayons un template appelé `mytemplate.pt`. Pour l'utiliser dans notre application, nous devons modifier notre vue pour retourner un dictionnaire de valeurs au lieu d'une réponse directe :

```python
def hello_world(request):
    return {'name': 'World'}
```

Ensuite, nous modifions notre configuration pour indiquer le template à utiliser avec cette vue :

```python
config.add_view(hello_world, route_name='hello', renderer='templates/mytemplate.pt')
```

Dans le template lui-même, nous pouvons utiliser la syntaxe TAL pour accéder aux valeurs du dictionnaire. Voici à quoi pourrait ressembler notre template `mytemplate.pt` :

```html
<html>
<body>
    <h1>Hello, ${name}!</h1>
</body>
</html>
```

Ici, `${name}` sera remplacé par la valeur correspondante dans le dictionnaire retourné par notre vue, en l'occurrence 'World'.

Si nous voulons aller encore plus loin et utiliser METAL pour la réutilisation des macros, notre code pourrait ressembler à ceci :

```html
<html metal:define-macro="my_macro">
<body>
    <h1 metal:define-slot="greeting">Hello, ${name}!</h1>
</body>
</html>
```

Et nous pourrions ensuite utiliser cette macro dans un autre template avec `metal:use-macro` :

```html
<div metal:use-macro="mytemplate.pt/my_macro">
    <h1 metal:fill-slot="greeting">Salut, ${name}!</h1>
</div>
```


### 2.5.4 Passer des données à un template Chameleon

Pour passer des données à un template Chameleon, soit notre vue retourne directement un dictionnaire, soit nous utilisons la fonction `render_to_response` de Pyramid, et nous passons les données sous forme de dictionnaire. Par exemple:

```python
from pyramid.view import view_config
from pyramid.renderers import render_to_response

@view_config(route_name='home', renderer='templates/index.pt')
def home(request):
    return render_to_response('templates/index.pt', {'title': 'Accueil', 'description': 'Bienvenue sur notre site'})
```

Dans cet exemple, `title` et `description` seront disponibles dans le template `index.pt` et seront remplacés par 'Accueil' et 'Bienvenue sur notre site' respectivement.

### 2.5.5 Rendu des templates à partir des vues

La dernière étape est de rendre le template à partir de la vue. Dans Pyramid, cela est fait en utilisant la fonction `render_to_response` mentionnée précédemment. La fonction `render_to_response` prend le nom du template et un dictionnaire de variables à passer au template, et renvoie une réponse HTTP avec le HTML généré.

## 2.6 Deform

Pyramid.Deform est une bibliothèque qui permet la génération de formulaires HTML à partir de schémas de validation Pyramid et de modèles ZPT (Zope Page Templates). Elle peut aussi bien être utilisée pour des formulaires simples que pour des formulaires complexes, avec des sous-formulaires, des contrôles conditionnels, de l'internationalisation, etc.

Commençons par l'installation de Deform. nous utilisons pip pour installer le package :

```bash
pip install deform
```

Ensuite, nous devons configurer notre application Pyramid pour servir les ressources statiques fournies par Deform. Dans notre fichier de configuration Pyramid, ajoutons la ligne suivante :

```python
config.add_static_view('deform_static', 'deform:static/')
```

Maintenant, nous allons créer un formulaire simple. Pour ce faire, nous avons besoin de définir un schéma. Chaque schéma correspond à un formulaire ou une partie d'un formulaire. Un schéma est une classe Python qui définit les champs et les contraintes de validation du formulaire.

Voici un exemple de schéma pour un formulaire d'inscription :

```python
from colander import Schema, SchemaNode, String, Email, Length

class RegistrationSchema(Schema):
    email = SchemaNode(
        String(),
        validator=Email(),
        title=_('email_label'),
        description="Your email address"
    )
    password = SchemaNode(
        String(),
        validator=Length(min=5),
        title=_('password_label')
        description="Choose a password",
        widget=deform.widget.PasswordWidget(),
    )
```

Ensuite, nous allons utiliser ce schéma pour générer un formulaire HTML. Dans notre vue Pyramid, nous pouvons créer un formulaire comme ceci :

```python
from pyramid.view import view_config
from deform import Form

@view_config(route_name='register', renderer='templates/register.pt')
def register_view(request):
    schema = RegistrationSchema()
    translator = request.localizer.translate
    form = Form(schema, buttons=('submit',), translator=translator)

    if 'submit' in request.POST:
        controls = request.POST.items()
        try:
            appstruct = form.validate(controls)
        except ValidationFailure as e:
            return {'form': e.render()}
        
        # ... handle the validated form input ...
        
    return {'form': form.render()}
```

Dans cet exemple, nous avons utilisé `form.validate()` pour valider les données du formulaire. Si la validation échoue, une exception `ValidationFailure` est levée, et nous pouvons utiliser `e.render()` pour obtenir une version du formulaire qui indique les erreurs.

Pour finir, nous devons rendre le formulaire dans un template. Avec un template ZPT, cela pourrait ressembler à ceci :

```html
<div class="container">
	<form method="POST">
		<span tal:replace="structure form" i18n:translate="">Form Content</span>
		<input type="submit" value="Submit" i18n:attributes="value submit_button">
	</form>
</div>
```

Voilà pour les bases de Pyramid.Deform. Lors de notre prochaine session, nous approfondirons ce sujet et explorerons des fonctionnalités plus avancées, comme les sous-formulaires et les widgets personnalisés.

# 3 Gestion des requêtes et réponses

## 3.1 Introduction aux requêtes HTTP GET et POST

### 3.1.1 Les requêtes HTTP

Les requêtes HTTP sont la façon dont les navigateurs web communiquent avec les serveurs. Chaque fois que nous accédons à une page web, notre navigateur envoie une requête HTTP au serveur qui héberge cette page. Le serveur traite la requête et renvoie une réponse HTTP, que le navigateur interprète et affiche.

Il existe plusieurs types de requêtes HTTP, mais les plus couramment utilisées sont les requêtes GET et POST.

### 3.1.2 Requêtes GET

Une requête GET est utilisée pour demander des données à un serveur. Par exemple, lorsque nous accédons à une page web, notre navigateur envoie une requête GET au serveur pour demander le contenu de la page.

Dans Pyramid, nous pouvons accéder aux données d'une requête GET à l'aide de `request.GET`, qui est un dictionnaire contenant les paramètres de la requête.

### 3.1.3 Requêtes POST

Une requête POST est utilisée pour envoyer des données à un serveur. Par exemple, lorsque nous remplissons un formulaire sur une page web et que nous cliquons sur "Envoyer", notre navigateur envoie une requête POST au serveur avec les données du formulaire.

Dans Pyramid, nous pouvons accéder aux données d'une requête POST à l'aide de `request.POST`, qui est également un dictionnaire.

### 3.1.4 Les composants d'une requête HTTP

Une requête HTTP est composée de plusieurs parties :

- La ligne de requête, qui contient le type de requête (par exemple, GET ou POST), l'URL de la ressource demandée, et la version du protocole HTTP.
- Les en-têtes de requête, qui contiennent des informations supplémentaires sur la requête, comme le type de contenu ou les cookies.
- Le corps de la requête, qui contient les données envoyées avec une requête POST.

### 3.1.5 Exercices pratiques

Maintenant, passons à quelques exercices pratiques pour nous aider à comprendre comment manipuler les requêtes GET et POST dans Pyramid. Essayons de créer une application simple qui accepte à la fois les requêtes GET et POST et renvoie les données reçues.

### 3.2 Manipulation des données de la requête

### 3.2.1 Extraction des données de la requête

Dans Pyramid, il existe différentes façons d'extraire les données d'une requête. Par exemple :

- **Paramètres de l'URL** : Les paramètres de l'URL peuvent être récupérés à partir de `request.matchdict`. Par exemple, pour une route définie comme `/users/{username}`, nous pouvons accéder au nom d'utilisateur avec `request.matchdict['username']`.
  
- **Données de formulaire** : Les données de formulaire envoyées par une requête POST peuvent être récupérées à partir de `request.POST`.
  
- **Données JSON** : Si le client envoie des données JSON dans le corps de la requête, nous pouvons les récupérer avec `request.json_body`.

### 3.2.2 Comment Pyramid traite les données de la requête

Lorsque Pyramid reçoit une requête, il analyse les données de la requête et les rend disponibles via l'objet `request`. 

- Pour les données du corps de la requête, Pyramid détermine le type de contenu à partir de l'en-tête `Content-Type` de la requête. Si le type de contenu est `application/x-www-form-urlencoded` (le type par défaut pour les formulaires HTML), Pyramid remplit `request.POST` avec les données du formulaire. Si le type de contenu est `application/json`, Pyramid remplit `request.json_body` avec les données JSON décodées.
  
- Pour les paramètres de l'URL, Pyramid les extrait à partir de la route qui a été associée à la requête. Les paramètres de l'URL sont accessibles via `request.matchdict`.

### 3.2.3 Exploration de `request.params`

`request.params` est un dictionnaire qui contient à la fois les données de `request.GET` et de `request.POST`. C'est utile lorsque nous ne nous soucions pas de savoir si les données viennent de l'URL ou du corps de la requête.

Par exemple, si nous avons un formulaire qui peut être soumis à la fois par GET et POST, nous pouvons utiliser `request.params` pour accéder aux données du formulaire indépendamment de la méthode de soumission.

### 3.2.4 Exercices pratiques

Nous allons maintenant faire quelques exercices pratiques pour nous aider à nous familiariser avec la manipulation des données de la requête. 

1. Créons une application qui reçoit des données de formulaire et les affiche.
2. Modifions l'application pour accepter également des données JSON.
3. Essayons d'envoyer à la fois des données de formulaire et des données JSON à l'application et observons comment Pyramid traite les données.


## 3.3 Envoi de réponses HTTP

### 3.3.1 Comprendre le cycle de requête-réponse HTTP

Le protocole HTTP fonctionne selon un modèle de requête-réponse. Le client (généralement un navigateur web) envoie une requête HTTP à un serveur. Le serveur traite la requête et envoie une réponse HTTP au client. 

Dans Pyramid, lorsque nous définissons une vue, nous créons essentiellement une fonction qui sera appelée pour générer une réponse à une certaine requête. Cette fonction prend la requête comme argument et renvoie une réponse.

**Exploration des composants d'une réponse HTTP**

Une réponse HTTP se compose de trois éléments principaux :

1. **Statut** : C'est un code numérique qui indique le résultat de la requête. Les codes de statut commencent généralement par l'un des cinq nombres entiers : 1xx (Information), 2xx (Succès), 3xx (Redirection), 4xx (Erreur du client) et 5xx (Erreur du serveur).

2. **En-têtes** : Ce sont des paires clé-valeur qui fournissent des informations supplémentaires sur la réponse. Par exemple, l'en-tête `Content-Type` indique le type de contenu du corps de la réponse.

3. **Corps** : Il s'agit des données réelles de la réponse. Il peut contenir une page HTML, des données JSON, une image, etc.

### 3.3.2 Création de réponses HTTP personnalisées

Dans Pyramid, nous pouvons créer des réponses HTTP personnalisées en utilisant la classe `pyramid.response.Response`. Par exemple :

```python
from pyramid.response import Response

def ma_vue(request):
    response = Response()
    response.status = '200 OK'
    response.body = b'Bonjour, le monde!'
    response.headers['Content-Type'] = 'text/plain'
    return response
```

Cette vue renvoie une réponse HTTP avec un statut de '200 OK', un corps de 'Bonjour, le monde!' et un en-tête 'Content-Type' de 'text/plain'.

### 3.3.3 Exercices pratiques

Pour terminer, essayons de réaliser les exercices suivants pour nous familiariser avec l'envoi de réponses HTTP en Pyramid :

1. Créons une vue qui renvoie une réponse avec un corps de texte brut.
2. Créons une vue qui renvoie une réponse avec des données JSON.
3. Créons une vue qui renvoie une réponse avec un statut d'erreur (par exemple, '404 Not Found').

## 3.4 Types de réponses HTTP et quand les utiliser

### 3.4.1 Exploration des différents types de réponses HTTP

En général, il y a plusieurs types de réponses HTTP que nous pourrions vouloir envoyer :

- **Réponses HTML** : C'est le type de réponse le plus couramment utilisé pour les applications web traditionnelles. Ces réponses ont un type de contenu `text/html` et contiennent un document HTML dans le corps.

- **Réponses JSON** : Ces réponses sont couramment utilisées pour les API REST. Elles ont un type de contenu `application/json` et contiennent des données JSON dans le corps.

- **Redirections** : Ce sont des réponses qui indiquent au client de faire une nouvelle requête à une autre URL. Les redirections sont généralement utilisées pour diriger l'utilisateur vers une nouvelle page après qu'une action a été effectuée.

- **Erreurs** : Ces réponses indiquent qu'une erreur s'est produite. Les réponses d'erreur ont généralement un code de statut dans la gamme 400-599.

### 3.4.2 Comprendre quand utiliser chaque type de réponse

- **Réponses HTML** : Utilisons des réponses HTML lorsque nous voulons renvoyer une page web au client. Par exemple, une vue qui renvoie une page d'accueil pourrait renvoyer une réponse HTML.

- **Réponses JSON** : Utilisons des réponses JSON lorsque nous développons une API REST et que nous voulons renvoyer des données au client. Par exemple, une vue qui renvoie les informations d'un utilisateur pourrait renvoyer une réponse JSON.

- **Redirections** : Utilisons des redirections lorsque nous voulons diriger le client vers une nouvelle URL après qu'une action a été effectuée. Par exemple, après qu'un utilisateur a soumis un formulaire, nous pourrions vouloir le rediriger vers une page de remerciement.

- **Erreurs** : Utilisons des réponses d'erreur lorsque quelque chose ne va pas. Par exemple, si un utilisateur essaie d'accéder à une ressource qui n'existe pas, nous pourrions vouloir renvoyer une réponse '404 Not Found'.

### 3.4.3 Exercices pratiques

Essayons quelques exercices pour nous familiariser avec l'envoi de différents types de réponses :

1. Créons une vue qui renvoie une réponse HTML.
2. Créons une vue qui renvoie une réponse JSON.
3. Créons une vue qui renvoie une redirection.
4. Créons une vue qui renvoie une réponse d'erreur.

## 3.5 Utilisation des cookies pour maintenir l'état de la session

### 3.5.1 Introduction aux cookies et à leur utilité pour maintenir l'état de la session

Les cookies sont de petits morceaux de données stockés dans le navigateur de l'utilisateur par le site web qu'il visite. Ils sont utilisés pour maintenir l'état de la session entre plusieurs requêtes. Les applications web utilisent les cookies pour une variété de raisons, par exemple pour savoir si un utilisateur est connecté, pour enregistrer les préférences de l'utilisateur, etc.

### 3.5.2 Gestion des cookies dans Pyramid

Pyramid offre plusieurs méthodes pour gérer les cookies :

- **Créer un cookie** : Nous pouvons utiliser la méthode `response.set_cookie` pour créer un nouveau cookie.

```python
response.set_cookie('mon_cookie', 'valeur', max_age=3600)
```

- **Lire un cookie** : Nous pouvons utiliser `request.cookies` pour accéder aux cookies envoyés par le client.

```python
valeur = request.cookies.get('mon_cookie')
```

- **Modifier un cookie** : Modifier un cookie est similaire à en créer un. nous devons simplement définir un nouveau cookie avec le même nom.

- **Supprimer un cookie** : Pour supprimer un cookie, utilisons `response.delete_cookie`.

```python
response.delete_cookie('mon_cookie')
```

### 3.5.3 Gestion de l'authentification et des cookies d'authentification

L'authentification est un aspect clé de la sécurité web, et Pyramid offre plusieurs mécanismes pour la gestion de l'authentification. La bibliothèque standard de Pyramid ne fournit pas de système d'authentification intégré par défaut, mais elle offre des outils et des abstractions pour créer le vôtre ou pour intégrer des systèmes d'authentification tiers.

#### Cookies d'authentification

Les cookies d'authentification sont souvent utilisés pour maintenir la session de l'utilisateur entre plusieurs requêtes. Lorsqu'un utilisateur se connecte avec succès, le serveur crée une session pour l'utilisateur et envoie un cookie avec un identifiant de session unique au navigateur de l'utilisateur. Pour chaque requête suivante, le navigateur envoie le cookie au serveur, ce qui permet au serveur de vérifier et de maintenir la session de l'utilisateur.

Pyramid offre un moyen intégré de définir des cookies. Pour définir un cookie, nous pouvons utiliser la méthode `response.set_cookie()` sur l'objet de réponse. Par exemple :

```python
response = Response("Some content")
response.set_cookie('session', '123456')
```

Pour lire un cookie, nous pouvons utiliser la méthode `request.cookies.get()`. Par exemple :

```python
session_id = request.cookies.get('session')
```

#### Gestion de l'authentification

Pyramid propose un système de "policies" d'authentification pour gérer l'authentification. Une "policy" d'authentification est une classe qui fournit des méthodes pour gérer les aspects de l'authentification, comme la récupération de l'identifiant de l'utilisateur et la vérification des autorisations de l'utilisateur.

Pour utiliser une "policy" d'authentification, nous devons d'abord la définir dans la configuration de notre application. Par exemple :

```python
from pyramid.authentication import AuthTktAuthenticationPolicy
from pyramid.authorization import ACLAuthorizationPolicy

authn_policy = AuthTktAuthenticationPolicy('sosecret', hashalg='sha512')
authz_policy = ACLAuthorizationPolicy()

config = Configurator(settings=settings,
                      root_factory=MyProject,
                      authentication_policy=authn_policy,
                      authorization_policy=authz_policy)
```

Ici, nous utilisons la "policy" `AuthTktAuthenticationPolicy`, qui est une "policy" d'authentification basée sur un ticket. Elle stocke les données d'authentification de l'utilisateur dans un cookie signé.

Ensuite, nous pouvons utiliser la méthode `authenticated_userid(request)` pour obtenir l'identifiant de l'utilisateur authentifié. Par exemple :

```python
user_id = request.authenticated_userid
```

Notons que tout système d'authentification devrait également implémenter un certain nombre de contrôles de sécurité pour protéger les données sensibles, tels que le hachage des mots de passe, le cryptage des données de session et la protection contre les attaques CSRF, comme nous l'avons mentionné précédemment.

### 3.5.3 Problèmes de sécurité liés aux cookies

Il est important de noter que les cookies peuvent présenter des risques de sécurité. Par exemple, si un attaquant parvient à voler un cookie de session, il peut usurper l'identité de l'utilisateur. Pour cette raison, il est essentiel de toujours utiliser des communications sécurisées (HTTPS) lors de l'envoi de cookies. De plus, nous pouvons utiliser l'option `secure` lors de la définition d'un cookie pour nous assurer qu'il n'est envoyé que sur une connexion sécurisée.

### 3.5.4 Gestion des Secrets dans Pyramid - Utilisation d'un fichier .env

La sécurité est un élément crucial dans le développement d'applications. nous devons gérer les secrets, tels que les mots de passe et les clés de session, de manière sécurisée. Une pratique courante consiste à utiliser un fichier `.env` pour stocker ces informations sensibles. Le fichier `.env` est placé à la racine de notre projet, mais il est important de l'ignorer dans notre dépôt git pour des raisons de sécurité.

#### 3.5.4.1 Création et gestion du fichier .env

Pour commencer, nous devons créer un fichier `.env` à la racine de notre projet Pyramid. Ce fichier stockera nos secrets. Par exemple:

```ini
SECRET_KEY=NotreCléSecrète
LDAP_LOGIN=NotreLoginLDAP
LDAP_PASSWORD=NotreMotDePasseLDAP
```

    Il est crucial que nous ajoutions `.env` à notre fichier `.gitignore` pour éviter de pousser accidentellement nos secrets vers notre dépôt git. nous pouvons le faire en ajoutant simplement une ligne à notre fichier `.gitignore`:

```git
.env
```

#### 3.5.4.2 Utilisation de la clé secrète pour les cookies et les sessions

Dans notre projet Pyramid, nous pouvons utiliser la clé secrète stockée dans le fichier `.env` pour générer des cookies et gérer les sessions. Pour cela, nous avons besoin de la bibliothèque `python-dotenv` qui permet d'accéder aux variables d'environnement définies dans le fichier `.env`. nous pouvons l'installer en utilisant pip:

```bash
pip install python-dotenv
```

Ensuite, dans notre code, nous pouvons charger les variables d'environnement à partir du fichier `.env` comme suit:

```python
from dotenv import load_dotenv
import os

load_dotenv()

secret_key = os.getenv("SECRET_KEY")
```

Nous pouvons maintenant utiliser `secret_key` pour signer les cookies et gérer les sessions. Par exemple:

```python
from pyramid.session import SignedCookieSessionFactory

my_session_factory = SignedCookieSessionFactory(secret_key)
config = Configurator(session_factory=my_session_factory)
```

#### 3.5.4.3 Utilisation des identifiants OpenLDAP

De même, nous pouvons utiliser `os.getenv` pour récupérer les identifiants OpenLDAP stockés dans le fichier `.env`. Par exemple:

```python
ldap_login = os.getenv("LDAP_LOGIN")
ldap_password = os.getenv("LDAP_PASSWORD")
```

Ces valeurs peuvent être utilisées pour configurer une connexion LDAP. Assurez-Nous de ne jamais insérer directement vos identifiants dans le code.

#### 3.5.4.4 Utiliser des variables secrètes définies dans un fichier `.env` dans les fichiers de configuration `.ini`

Nous devons utiliser la bibliothèque `python-dotenv`. Cette bibliothèque permet de charger les variables d'un fichier `.env` dans l'environnement de notre application, ce qui les rend accessibles dans notre fichier `.ini`.

Voici comment procéder :

1. Installons la bibliothèque `python-dotenv` à l'aide de `pip` :

```bash
pip install python-dotenv
```

2. Créez un fichier `.env` à la racine de votre projet (au même niveau que le fichier `.ini`) et définissons nos variables secrètes dans ce fichier. Par exemple :

```
SECRET_KEY=my_secret_key
DB_PASSWORD=my_db_password
SMTP_PASSWORD=my_smtp_password
```

3. Dans notre fichier `.ini`, utilisons les variables secrètes comme suit :

```ini
[app:main]
# Utilisons les variables secrètes définies dans .env en les préfixant avec 'env:'
# Exemple :
db.password = %(env:DB_PASSWORD)s
smtp.password = %(env:SMTP_PASSWORD)s

# Autres configurations de notre application
```

4. Dans le fichier Python où nous configurons notre application Pyramid (par exemple, dans `__init__.py` ou `main.py`), chargeons les variables d'environnement à partir du fichier `.env` en utilisant `dotenv.load_dotenv()` :

```python
import os
from dotenv import load_dotenv

def main(global_config, **settings):
    # Charger les variables d'environnement à partir de .env
    load_dotenv()

    # Autres configurations de l'application

    return config.make_wsgi_app()
```

5. Assurons-nous que le fichier `.env` est correctement ignoré par notre système de contrôle de version et est ajouté à notre fichier `.gitignore` si nous utilisons [[git]]. Les fichiers `.env` contenant des informations sensibles ne doivent jamais être inclus dans un dépôt public ou partagé !!!!!

### 3.5.4.5 Valeur par défaut en cas d’absence dans .env

Il n'existe pas de construction conditionnel (pas de tests) dans les fichiers .ini, il faut donc considérer que le .ini contient les valeurs par défaut et tester la présence des valeurs dans le .env pour écraser celles du .ini.
Nous devons alors utiliser la fonction `os.environ.get()` pour accéder aux variables d'environnement et fournir une valeur par défaut si elles ne sont pas définies.

Voici comment :

1. Supposons que nous avons dans votre fichier `.ini` les variables suivantes :

```ini
[app:main]
mail.username = michaellaunay
mail.password = "Password"
```

2. Dans le fichier Python où nous configurons notre application Pyramid (par exemple, `__init__.py` ou `main.py`), chargeons les variables d'environnement à partir du fichier `.env` en utilisant `dotenv.load_dotenv()` :

```python
import os
from dotenv import load_dotenv

def main(global_config, **settings):
    # Charger les variables d'environnement à partir de .env
    load_dotenv()

    # Utiliser os.environ.get() pour définir les valeurs par défaut si les variables ne sont pas définies
    settings.setdefault('mail.username', os.environ.get('MAIL_USERNAME', 'default_username'))
    settings.setdefault('mail.password', os.environ.get('MAIL_PASSWORD', 'default_password'))

    # Autres configurations de l'application

    return config.make_wsgi_app()
```

Pour `os.environ.get()`, nous spécifions la clé de la variable d'environnement à récupérer, ainsi que la valeur par défaut à utiliser si la variable n'est pas définie dans le fichier `.env`. Ainsi, si la variable d'environnement `MAIL_USERNAME` est définie dans `.env`, sa valeur sera utilisée. Sinon, `os.environ.get()` renverra la valeur par défaut `'default_username'`.

#### 3.5.4.6 Exercices pratiques

Essayons les exercices suivants pour nous familiariser avec l'utilisation des cookies dans Pyramid :

1. Créons une vue qui définit un cookie.
2. Créons une vue qui lit la valeur d'un cookie et l'affiche à l'utilisateur.
3. Créons une vue qui modifie la valeur d'un cookie.
4. Créons une vue qui supprime un cookie.

# 4. Introduction à l'authentification

## 4.1 Comprendre l'authentification et son importance

### 4.1.1 Qu'est-ce que l'authentification ?

L'authentification est le processus de vérification de l'identité d'un utilisateur. En d'autres termes, lorsque les utilisateurs prétendent être qui ils disent être (généralement en fournissant un nom d'utilisateur et un mot de passe), l'authentification est le mécanisme qui nous permet de vérifier cette affirmation.

### 4.1.2 Pourquoi l'authentification est-elle cruciale dans les applications web ?

Sans authentification, nous ne pourrions pas différencier un utilisateur d'un autre, ce qui signifie que nous ne pourrions pas offrir d'expérience utilisateur personnalisée. De plus, l'authentification est essentielle pour la sécurité. Elle nous permet de restreindre l'accès à certaines parties de nos applications aux seuls utilisateurs autorisés.

### 4.1.3 Les principaux types d'authentification

Il existe plusieurs méthodes d'authentification, certaines plus sécurisées que d'autres. Voici les plus courantes :

- **Authentification basée sur les connaissances** : C'est la forme la plus courante d'authentification. Elle implique quelque chose que l'utilisateur connaît, comme un mot de passe ou un code PIN.

- **Authentification basée sur la possession** : Cela implique quelque chose que l'utilisateur possède, comme une clé de sécurité physique ou un appareil mobile.

- **Authentification basée sur l'inherence** : Cela implique quelque chose que l'utilisateur est, comme une empreinte digitale ou un autre biométrique.

- **Authentification multifacteurs** : Cela implique l'utilisation de deux ou plus des méthodes ci-dessus en même temps.

## 4.2 Introduction aux mécanismes d'authentification dans Pyramid

### 4.2.1 Exploration des méthodes d'authentification intégrées dans Pyramid

Pyramid fournit un certain nombre de mécanismes intégrés pour l'authentification, qui nous permettent d'implémenter différents types d'authentification en fonction de nos besoins spécifiques. Ces mécanismes d'authentification sont disponibles via le module `pyramid.authentication`. Certaines des méthodes les plus couramment utilisées comprennent `AuthTktAuthenticationPolicy` pour l'authentification basée sur un ticket, `SessionAuthenticationPolicy` pour l'authentification basée sur les sessions, et `RemoteUserAuthenticationPolicy` pour l'authentification basée sur l'utilisateur distant.

### 4.2.2 Comprendre comment fonctionne l'authentification basée sur les sessions et les cookies

L'authentification basée sur les sessions et les cookies est l'un des types d'authentification les plus couramment utilisés dans le développement web. Dans ce type d'authentification, lorsque l'utilisateur se connecte avec succès, un cookie contenant un identifiant de session unique est stocké dans le navigateur de l'utilisateur. Ce cookie est ensuite utilisé pour récupérer les informations de l'utilisateur pour chaque requête successive.

### 4.2.3 Examen de l'authentification basée sur le token

L'authentification basée sur le token est une autre méthode d'authentification couramment utilisée, en particulier pour les API RESTful. Dans ce type d'authentification, lorsqu'un utilisateur se connecte avec succès, un token d'authentification est généré et renvoyé à l'utilisateur. Ce token est ensuite inclus dans l'en-tête de chaque requête successive pour authentifier l'utilisateur.

## 4.3 Créer une politique d'authentification simple

### 4.3.1 Introduction aux politiques d'authentification dans Pyramid

Les politiques d'authentification dans Pyramid sont des composants qui définissent comment les identités des utilisateurs sont représentées et comment ces identités peuvent être obtenues. En d'autres termes, elles définissent comment les utilisateurs sont authentifiés. Pyramid utilise deux types de politiques d'authentification : "l'Authentification Policy" et "l'Authorization Policy". L'Authentification Policy est responsable de fournir des informations d'identité à l'application, tandis que l'Authorization Policy utilise ces informations pour prendre des décisions d'autorisation.

### 4.3.2 Créer et configurer une politique d'authentification simple

Pour créer une politique d'authentification simple dans Pyramid, nous devons définir une Authentification Policy. Pour ce faire, nous devons d'abord choisir une méthode d'authentification à utiliser. Par exemple, si nous voulions utiliser l'authentification basée sur les sessions, nous pourrions utiliser la classe `SessionAuthenticationPolicy` fournie par Pyramid.

Voici comment nous pouvons configurer une authentification basée sur les sessions :

```python
from pyramid.authentication import SessionAuthenticationPolicy
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    authn_policy = SessionAuthenticationPolicy()
    config.set_authentication_policy(authn_policy)
    # ...
    return config.make_wsgi_app()
```

### 4.3.3 Tests et vérification de la politique d'authentification

Une fois que nous avons configuré notre politique d'authentification, il est important de la tester pour nous assurer qu'elle fonctionne comme prévu. nous pouvons le faire en écrivant des tests unitaires qui vérifient si un utilisateur peut se connecter avec succès et si l'identité de l'utilisateur est correctement stockée et récupérée.

Pyramid propose la bibliothèque `WebTest` pour tester les applications web. Voici un exemple de comment on pourrait écrire ces tests :
```python
from pyramid import testing
from pyramid.authentication import SessionAuthenticationPolicy
from pyramid.testing import DummyRequest
import unittest

class TestAuthenticationPolicy(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_authenticated_userid(self):
        request = DummyRequest()
        request.session['userid'] = 'testuser'  # On définit un userid dans la session

        policy = SessionAuthenticationPolicy()
        userid = policy.authenticated_userid(request)

        self.assertEqual(userid, 'testuser')  # On vérifie que l'userid récupéré est correct

    def test_unauthenticated_userid(self):
        request = DummyRequest()

        policy = SessionAuthenticationPolicy()
        userid = policy.unauthenticated_userid(request)

        self.assertIsNone(userid)  # On vérifie qu'aucun userid n'est récupéré car nous n'avons pas défini de session

    def test_effective_principals(self):
        request = DummyRequest()
        request.session['userid'] = 'testuser'

        policy = SessionAuthenticationPolicy()
        principals = policy.effective_principals(request)

        self.assertIn('testuser', principals)  # On vérifie que l'userid fait partie des principaux effectifs
```

Dans cet exemple, nous testons les trois méthodes principales de `SessionAuthenticationPolicy` :

- `authenticated_userid(request)`, qui renvoie l'userid de l'utilisateur actuellement authentifié, ou `None` si aucun utilisateur n'est authentifié.
- `unauthenticated_userid(request)`, qui renvoie l'userid de l'utilisateur proposé par la requête, que cet utilisateur soit réellement authentifié ou non.
- `effective_principals(request)`, qui renvoie une liste de tous les "principaux" effectifs pour la requête. Les principaux effectifs incluent toujours la chaîne `'system.Everyone'`, l'userid de l'utilisateur authentifié (si un utilisateur est authentifié), et toute autre valeur spécifique à l'application.

Ces tests supposent que nous utilisons l'implémentation par défaut de `SessionAuthenticationPolicy` et que nous stockons l'userid de l'utilisateur dans la session sous la clé `'userid'`. Si nous avons une implémentation personnalisée de `SessionAuthenticationPolicy` ou si nous stockons l'userid sous une clé différente, nous devrons adapter ces tests en conséquence.

## 4.4 Les défis de l'authentification et les meilleures pratiques

### 4.4.1 Discussion sur les défis courants de l'authentification

L'authentification, bien qu'essentielle dans le développement d'applications web, présente un certain nombre de défis. Certains de ces défis comprennent la sécurisation des données sensibles, la gestion des sessions utilisateur, le traitement des erreurs d'authentification et la mise en place de procédures de récupération de mot de passe. Il est important de comprendre ces défis afin de pouvoir mettre en place des mécanismes efficaces pour les surmonter.

### 4.4.2 Comprendre les meilleures pratiques de l'authentification

Il existe des meilleures pratiques ("Best practices") que nous pouvons suivre pour assurer une authentification efficace et sécurisée. Certaines de ces pratiques comprennent le stockage sécurisé des mots de passe (par exemple, ne jamais stocker les mots de passe en clair), l'utilisation de sessions sécurisées, la limitation des tentatives de connexion pour prévenir les attaques par force brute, et la fourniture d'options de récupération de mot de passe sécurisées.

### 4.4.3 Exploration de la sécurité des mots de passe et des stratégies de hachage

Une partie essentielle de la sécurisation de l'authentification est la gestion sécurisée des mots de passe. Cela implique généralement le hachage des mots de passe avant leur stockage. Le hachage est un processus unidirectionnel qui transforme un mot de passe en une chaîne de caractères de longueur fixe. Même une petite modification du mot de passe d'origine entraînera un hash complètement différent. Cela signifie qu'il est pratiquement impossible de retrouver le mot de passe original à partir du hash, rendant ainsi le mot de passe sécurisé en cas de violation des données.

## 4.5 Gestion des sessions

Les trois composants suivants :`pyramid.session`, `pyramid_tm`, et `pyramid_session_redis`, sont liés à la gestion des sessions dans l'écosystème Pyramid. Cependant, ils ont des rôles et des fonctionnalités légèrement différents. Voici les différences, avantages et inconvénients de chacun :

### 4.5.1. `pyramid.session`

   - Rôle : `pyramid.session` est un module qui fournit des mécanismes de gestion de session pour les applications Pyramid.
   - Fonctionnalités : Il fournit des implémentations de base pour stocker des informations de session côté client, telles que dans des cookies non chiffrés ou dans des cookies signés.
   - Avantages :
     - Simple à configurer et à utiliser pour des sessions légères sans dépendances externes.
     - Utile pour les cas d'utilisation où une session légère est suffisante et où la sécurité n'est pas une préoccupation majeure.
   - Inconvénients :
     - La sécurité est limitée car les informations de session sont stockées côté client, ce qui les expose aux risques de manipulation ou de vol par des attaquants.

### 4.5.2 `pyramid_tm`

   - Rôle : `pyramid_tm` est un package d'intégration Pyramid qui fournit un gestionnaire de transaction pour les applications Pyramid.
   - Fonctionnalités : Il gère les transactions de base de données, ce qui signifie que toutes les modifications de base de données sont effectuées dans une transaction. Si une exception est levée pendant le traitement de la requête, la transaction est annulée (rollback). Sinon, elle est validée (commit) à la fin de la requête.
   - Avantages :
     - Assure l'intégrité des opérations de base de données en garantissant que toutes les modifications sont atomiques.
     - Permet de gérer les exceptions liées à la base de données de manière cohérente.
   - Inconvénients :
     - Il ne gère pas directement les sessions, mais il peut être utilisé en conjonction avec d'autres modules pour gérer les sessions dans Pyramid.

### 4.5.3 `pyramid_session_redis`

   - Rôle : `pyramid_session_redis` est un adaptateur de session pour Pyramid qui permet de stocker les informations de session dans Redis, une base de données NoSQL.
   - Fonctionnalités : Il permet de stocker les données de session dans une base de données Redis en utilisant le protocole Redis.
   - Avantages :
     - Stockage sécurisé des données de session côté serveur dans Redis.
     - Possibilité de gérer des sessions plus complexes et de grandes quantités de données de session.
   - Inconvénients :
     - Nécessite une installation et une configuration supplémentaires de Redis en tant que stockage pour les données de session.
     - Un peu plus complexe à configurer et à mettre en place que les solutions basées sur des cookies.

En résumé, `pyramid.session` est simple à utiliser mais moins sécurisé car il stocke les informations de session côté client. `pyramid_tm` gère les transactions de base de données et assure l'intégrité des opérations de base de données, mais ne gère pas directement les sessions. `pyramid_session_redis` offre une solution plus robuste pour le stockage sécurisé des données de session côté serveur, mais nécessite l'installation et la configuration supplémentaires de Redis.

Le choix entre ces composants dépendra de vos besoins spécifiques et du niveau de sécurité requis pour la gestion des sessions dans votre application Pyramid. Si vous avez besoin d'une solution simple et légère, `pyramid.session` peut être suffisant. Si vous avez besoin de transactions de base de données robustes, vous pouvez utiliser `pyramid_tm` en conjonction avec d'autres solutions de gestion de sessions. Si la sécurité est une préoccupation majeure et que vous avez besoin d'un stockage côté serveur, `pyramid_session_redis` peut être une bonne option.

# 5. Sécurisation des données

## 5.1 Protection des données des utilisateurs

### 5.1.1 Qu'est-ce que la protection des données des utilisateurs ?

La protection des données des utilisateurs est un ensemble de stratégies et de techniques qui permettent de protéger les informations personnelles des utilisateurs contre tout accès, utilisation, divulgation, altération ou destruction non autorisés. Ces données peuvent comprendre des informations d'identification comme les noms, adresses, numéros de téléphone, ainsi que des informations plus sensibles comme les mots de passe, les informations financières et de santé.

### 5.1.2 Pourquoi est-ce important ?

Protéger les données des utilisateurs est crucial pour plusieurs raisons. Premièrement, c'est une question de respect de la vie privée des utilisateurs. Les utilisateurs ont le droit de savoir comment leurs données sont utilisées et stockées, et ils ont le droit de s'attendre à ce que leurs informations soient protégées. Deuxièmement, c'est une question de conformité légale. De nombreux pays et régions ont des lois strictes sur la protection des données, et les organisations doivent s'y conformer (Voir [[Règlement Général sur la Protection des Données (RGPD)]]). Enfin, c'est une question de réputation et de confiance. Si les utilisateurs ne font pas confiance à notre application pour protéger leurs données, ils iront ailleurs.

### 5.1.3 Les principes de base de la protection des données

Il existe plusieurs principes de base de la protection des données que toutes les organisations devraient respecter. Ces principes comprennent la minimisation des données (ne collecter que les données nécessaires), l'exactitude des données (s'assurer que les données sont correctes et à jour), la limitation de l'utilisation et du stockage des données (ne pas utiliser les données à des fins non déclarées et ne pas les conserver plus longtemps que nécessaire), la sécurité des données (protéger les données contre les accès non autorisés) et la responsabilité (être responsable de la conformité à ces principes).

### 5.2 Hashage des mots de passe

### 5.2.1 Qu'est-ce que le hachage des mots de passe ?

Le hachage des mots de passe est une technique de sécurité qui transforme les mots de passe en une série de chiffres et de lettres appelée "hash". Ce processus est unidirectionnel, ce qui signifie qu'il est pratiquement impossible de retrouver le mot de passe original à partir du hash. De plus, même une petite modification du mot de passe d'origine produit un hash complètement différent.

### 5.2.2 Pourquoi avons-nous besoin de hacher les mots de passe ?

Le hachage des mots de passe est une pratique essentielle pour la sécurité des applications. Si les mots de passe sont stockés en texte clair et qu'un attaquant réussit à accéder à notre base de données, il aura accès à tous les comptes utilisateur. Avec le hachage, même si la base de données est compromise, l'attaquant ne verra que les hashes des mots de passe, qui ne peuvent pas être inversés pour obtenir les mots de passe d'origine.

### 5.2.3 Comment effectuer le hachage des mots de passe dans Pyramid

Pyramid n'inclut pas directement de fonctionnalités de hachage des mots de passe, mais nous pouvons utiliser des bibliothèques Python tierces telles que `passlib`. Voici comment nous pouvons hacher un mot de passe avec `passlib` :

```python
from passlib.hash import pbkdf2_sha256

def hash_password(password):
    return pbkdf2_sha256.hash(password)
```

Cette fonction prend un mot de passe en entrée et renvoie le hash correspondant.

### 5.2.4 Exercices pratiques sur le hachage des mots de passe

1. Installons la bibliothèque `passlib` avec `pip install passlib`.
2. Écrivons une fonction qui accepte un mot de passe en entrée, le hache avec `passlib` et retourne le hash.
3. Écrivons une autre fonction qui accepte un mot de passe et un hash, et vérifie si le mot de passe correspond au hash. nous pouvons utiliser la fonction `pbkdf2_sha256.verify(password, hash)` de `passlib` pour cela.
4. Testons nos fonctions avec différents mots de passe.

## 5.3 Prévention des attaques CSRF avec pyramid.csrf

### 5.3.1 Qu'est-ce qu'une attaque CSRF ?

CSRF signifie Cross-Site Request Forgery. Dans une attaque CSRF, un attaquant trompe une victime pour qu'elle effectue une action non désirée sur une application web dans laquelle elle est authentifiée. Par exemple, un attaquant pourrait forcer un utilisateur à changer son mot de passe, à envoyer un courriel, ou à effectuer une transaction financière sans que l'utilisateur ne s'en rende compte.

### 5.3.2 Pourquoi est-ce un problème ?

Les attaques CSRF sont un problème parce qu'elles exploitent la confiance qu'une application a envers un utilisateur. Puisque l'application ne peut pas distinguer une requête légitime d'une requête forgée, elle exécute l'action demandée tant que l'utilisateur est authentifié. Cela peut conduire à des conséquences graves, comme la perte de contrôle sur le compte de l'utilisateur.

### 5.3.3 Comment utiliser pyramid.csrf pour prévenir les attaques CSRF

Pyramid fournit un module, `pyramid.csrf`, pour aider à prévenir les attaques CSRF. Voici comment nous pouvons l'utiliser :

1. Dans notre fonction de vue, utilisons `request.session.get_csrf_token()` pour obtenir un jeton CSRF.

2. Incluons ce jeton dans le formulaire HTML de notre application, généralement dans un champ caché.

3. Lorsque le formulaire est soumis, vérifions que le jeton CSRF soumis correspond au jeton stocké dans la session. nous pouvons utiliser `request.session.check_csrf_token()` pour cela.

### 5.3.4 Rendre systématique l'usage des jetons CSRF

#### Activation de la politique CSRF

Pour utiliser CSRF dans Pyramid, nous devons activer une politique CSRF. Cela peut être fait dans la configuration de notre application :

```python
config.set_default_csrf_options(require_csrf=True)
```

Cette ligne indique que notre application requiert une protection CSRF.

#### Génération des jetons CSRF

Nous pouvons générer un jeton CSRF en utilisant `pyramid.csrf.get_csrf_token(request)`. Cela nous donnera un jeton que nous pouvons ensuite inclure dans nos formulaires ou nos requêtes AJAX.

Par exemple, si nous utilisons Chameleon pour nos templates, nous pouvons inclure le jeton CSRF dans un formulaire comme suit :

```html
<form method="post">
	<input type="hidden" name="csrf_token" tal:attributes="value request.session.get_csrf_token()"/>
	<!-- Reste du formulaire -->
</form>
```

#### Vérification des jetons CSRF

Pyramid vérifie automatiquement le jeton CSRF pour toutes les requêtes POST, PUT, DELETE et PATCH si `require_csrf=True` a été réglé. Si le jeton CSRF est manquant ou ne correspond pas, Pyramid lèvera une exception `pyramid.exceptions.BadCSRFToken`.

### 5.3.5 Exercices pratiques sur la prévention des attaques CSRF

1. Modifions un formulaire dans notre application pour inclure un jeton CSRF. Assurons-nous que le jeton est correctement envoyé lorsque le formulaire est soumis.
2. Ajoutons une vérification CSRF dans la fonction de vue qui traite les soumissions de formulaires. Testons notre application pour nous assurer qu'elle rejette les soumissions de formulaires qui ne comprennent pas le bon jeton CSRF.
3. Pensons à d'autres endroits de notre application où nous pourrions être vulnérable aux attaques CSRF. Comment pourrions-nous utiliser `pyramid.csrf` pour renforcer la sécurité de ces zones ?

### 5.3.6 Exemple de code vérifiant le token CSRF

Exemple de tests unitaires qui vérifient que la vue "register" fournit bien un token CSRF et que celui-ci est correctement vérifié. nous utilisons la bibliothèque `WebTest` de Pyramid.

```python
from pyramid import testing
from pyramid.testing import DummyRequest
from webtest import TestApp
import unittest

def register_view(request):
    # Cette fonction est juste un exemple de ce que pourrait être notre vue "register".
    # nous devronsremplacer cette fonction par la vraie vue "register" de notre application.
    csrf_token = request.session.get_csrf_token()
    return {"csrf_token": csrf_token}

class TestRegisterView(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_register_view_provides_csrf_token(self):
        from myapp import main  # Remplaçons "myapp" par le nom de notre application Pyramid
        app = main({})
        testapp = TestApp(app)

        # Faire une requête GET à la vue "register"
        response = testapp.get('/register', status=200)

        # Vérifier que le token CSRF est présent dans la réponse
        self.assertIn('csrf_token', response.json)

    def test_register_view_checks_csrf_token(self):
        from myapp import main
        app = main({})
        testapp = TestApp(app)

        # Faire une requête POST à la vue "register" sans token CSRF
        response = testapp.post('/register', status=400)

        # Vérifier que la requête échoue à cause de l'absence de token CSRF
        self.assertEqual(response.status_code, 400)

        # Faire une requête GET à la vue "register" pour obtenir un token CSRF
        response = testapp.get('/register', status=200)
        csrf_token = response.json['csrf_token']

        # Faire une requête POST à la vue "register" avec un token CSRF
        response = testapp.post('/register', {'csrf_token': csrf_token}, status=200)

        # Vérifier que la requête réussit maintenant que nous avons un token CSRF valide
        self.assertEqual(response.status_code, 200)
```

Ces tests supposent que nous avons une route `/register` qui correspond à une vue `register_view` dans notre application Pyramid. De plus, ces tests supposent que nous utilisons l'implémentation par défaut de `SessionAuthenticationPolicy` et que nous stockons le token CSRF dans la session sous la clé `'csrf_token'`. Si ce n'est pas le cas, nous devrons adapter ces tests en conséquence.

## 5.4.  Validation et assainissement des entrées des utilisateurs

### 5.4.1 Pourquoi la validation et l'assainissement des entrées sont-ils importants ?

La validation et l'assainissement des entrées sont essentiels pour maintenir la sécurité de notre application. Sans une validation appropriée, les attaquants pourraient insérer des données malveillantes, comme des scripts, dans notre application, ce qui pourrait conduire à des attaques de type Cross-site Scripting (XSS). L'assainissement des données, d'autre part, garantit que les données entrées par les utilisateurs sont sûres avant qu'elles ne soient utilisées par notre application.

### 5.4.2 Comment valider les entrées des utilisateurs dans Pyramid

La validation des entrées des utilisateurs peut être réalisée à l'aide de différents outils. Un choix populaire pour la validation des données en Python est la bibliothèque [`colander`](https://docs.pylonsproject.org/projects/colander/en/latest/basics.html).

Voici un exemple de la façon dont nous pouvons utiliser `colander` pour définir un schéma de validation pour un formulaire de connexion :

```python
import colander

class LoginForm(colander.Schema):
    username = colander.SchemaNode(colander.String())
    password = colander.SchemaNode(colander.String())
```

Ce schéma spécifie que le formulaire de connexion doit avoir un champ "username" et un champ "password", et que tous les deux doivent être des chaînes de caractères.

#### Exemple de vérification du schéma
Voici comment nous pourrions implémenter une vue "login" qui utilise ce schéma pour valider les entrées de l'utilisateur :

```python
import colander
from pyramid.view import view_config
from pyramid.httpexceptions import HTTPBadRequest

class LoginForm(colander.Schema):
    username = colander.SchemaNode(colander.String())
    password = colander.SchemaNode(colander.String())

@view_config(route_name='login', renderer='json', request_method='POST')
def login_view(request):
    schema = LoginForm()
    try:
        # Ici, on suppose que les données du formulaire sont envoyées en JSON.
        # Si elles sont envoyées en tant que données de formulaire normales,
        # nous devrions utiliser request.POST au lieu de request.json_body.
        form_data = schema.deserialize(request.json_body)
    except colander.Invalid as e:
        # Si les données ne sont pas valides, renvoyonsune erreur 400 avec les erreurs de validation.
        return HTTPBadRequest(json_body=e.asdict())

    # Si les données sont valides, utilisez-les pour essayer de connecter l'utilisateur.
    username = form_data['username']
    password = form_data['password']

    # Inséronsici la logique d'authentification, par exemple vérifier le nom d'utilisateur
    # et le mot de passe dans notre base de données, et éventuellement définir l'utilisateur
    # comme connecté dans la session.

    return {'status': 'success'}  # Si tout se passe bien, renvoyonsun succès.
```

Dans cet exemple, la vue "login" utilise le schéma `LoginForm` pour valider les données du formulaire envoyées dans le corps de la requête POST. Si les données ne sont pas valides, la vue renvoie une réponse avec le code d'état HTTP 400 et les erreurs de validation. Si les données sont valides, la vue extrait le nom d'utilisateur et le mot de passe et peut alors les utiliser pour authentifier l'utilisateur.

Cet exemple est assez basique et ne contient pas de logique réelle d'authentification (par exemple, vérifier le nom d'utilisateur et le mot de passe dans une base de données).

### 5.4.3 Comment assainir les entrées des utilisateurs

L'assainissement des entrées des utilisateurs est tout aussi important que la validation. L'assainissement fait référence à la suppression ou à l'échappement des caractères potentiellement dangereux des entrées de l'utilisateur. 

Dans le contexte de Pyramid et des modèles de pages web, l'assainissement est souvent pris en charge automatiquement par le moteur de templates. Par exemple, le moteur de templates Chameleon, largement utilisé avec Pyramid, échappe automatiquement les variables insérées dans les templates, ce qui aide à prévenir les attaques XSS.

### 5.4.4 Exercices pratiques sur la validation et l'assainissement des entrées des utilisateurs

1. Utilisons `colander` pour définir des schémas de validation pour d'autres formulaires de notre application. Testons notre application pour nous assurer que la validation fonctionne correctement.

2. Recherchons comment nous pouvons assainir les entrées des utilisateurs dans d'autres contextes, comme lors de l'exécution de requêtes SQL. 

3. Pensons à des scénarios dans lesquels l'assainissement automatique par le moteur de templates pourrait ne pas être suffisant. Comment pourrions-nous gérer ces scénarios pour assurer la sécurité de notre application ?

## 5.5 Révision et exercices pratiques

### 5.5.1 Révision des concepts clés de la semaine

Nous avons abordé beaucoup de sujets, y compris la protection des données des utilisateurs, le hachage des mots de passe, la prévention des attaques CSRF avec pyramid.csrf, et la validation et l'assainissement des entrées des utilisateurs.

Prenons le temps de revoir ces concepts et de nous assurer que nous comprenons comment ils s'appliquent à nos propres projets. Il est essentiel de comprendre ces principes de sécurité pour créer des applications web sécurisées.

### 5.5.2 Exercices pratiques : Création d'une application simple avec une authentification sécurisée et une protection des données

Pour cet exercice pratique, nous devons créer une petite application qui met en œuvre tout ce que nous avons appris cette semaine. Notre application devrait avoir les caractéristiques suivantes :

- Une page de connexion où les utilisateurs peuvent entrer leur nom d'utilisateur et leur mot de passe.
- Les mots de passe des utilisateurs doivent être hachés avant d'être stockés.
- Les entrées des utilisateurs doivent être validées et assainies.
- L'application doit utiliser pyramid.csrf pour prévenir les attaques CSRF.
- nous pouvons choisir d'implémenter d'autres fonctionnalités pour pratiquer les compétences que nous avons acquises lors de ce cours.

### 5.5.3 Correction
Voici une solution possible pour l'exercice proposé, mais notons que la mise en œuvre spécifique peut varier en fonction de nombreux facteurs, tels que les exigences spécifiques du projet, les préférences personnelles, et plus encore. Voici une version de base.

```python
from pyramid.security import Allow, Everyone
from pyramid.httpexceptions import HTTPFound
from pyramid.view import (
    view_config,
    view_defaults
)
from pyramid.session import SignedCookieSessionFactory
from pyramid.authentication import AuthTktAuthenticationPolicy
from pyramid.authorization import ACLAuthorizationPolicy
from passlib.hash import pbkdf2_sha256

my_session_factory = SignedCookieSessionFactory('mysecrect')
authentication_policy = AuthTktAuthenticationPolicy('sosecret')
authorization_policy = ACLAuthorizationPolicy()

# Your main function
def main(global_config, **settings):
    config = Configurator(settings=settings,
                          root_factory=MyFactory,
                          session_factory=my_session_factory,
                          authentication_policy=authentication_policy,
                          authorization_policy=authorization_policy)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('login', '/login')
    config.add_route('logout', '/logout')
    config.scan('.views')
    return config.make_wsgi_app()

class MyFactory(object):
    __acl__ = [ (Allow, Everyone, 'view'), 
                (Allow, 'group:users', 'edit') ]
    def __init__(self, request):
        pass

# Views
@view_defaults(renderer='login.pt')
class MyViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home_view(self):
        return dict(name="Home View")

    @view_config(route_name='login', request_method='POST')
    def login(self):
        username = self.request.params['username']
        password = self.request.params['password']
        hashed = pbkdf2_sha256.hash(password)
        # Supposons que le nom d'utilisateur est 'user' et le mot de passe est 'password'
        if username == "user" and pbkdf2_sha256.verify(password, hashed):
            headers = remember(self.request, userid=username)
            return HTTPFound(location=self.request.route_url('home'), headers=headers)
        return HTTPFound(location=self.request.route_url('login'))

    @view_config(route_name='logout')
    def logout(self):
        headers = forget(self.request)
        return HTTPFound(location=self.request.route_url('home'), headers=headers)
```

Dans ce code, nous définissons une application Pyramid simple avec des routes pour l'accueil, la connexion et la déconnexion. Lorsqu'un utilisateur se connecte, le mot de passe entré est haché et vérifié avec le mot de passe haché stocké (dans cet exemple simplifié, nous supposons que le nom d'utilisateur est 'user' et le mot de passe est 'password'). Si la vérification réussit, l'utilisateur est redirigé vers la page d'accueil avec les headers d'authentification appropriés.

Notons que c'est une application très simplifiée et dans une application réelle, nous aurions des mécanismes pour gérer les utilisateurs et les mots de passe de manière plus sécurisée et efficace. De plus, ce code ne prend pas en compte la validation et l'assainissement des entrées des utilisateurs, qui seraient également importants dans une application réelle.

# 6. Introduction à LDAP et OpenLDAP

## 6.1 Qu'est-ce que LDAP ?

LDAP, pour Lightweight Directory Access Protocol, est un protocole standard ouvert, largement utilisé pour accéder et gérer les services d'annuaire sur Internet. Il est utilisé pour stocker, organiser et récupérer des informations complexes sur les utilisateurs et les ressources à partir d'un annuaire centralisé.

Dans le contexte de notre cours, nous pouvons imaginer LDAP comme un système pour gérer les informations d'authentification et d'autorisation de nos utilisateurs. Au lieu de stocker ces informations dans notre base de données Pyramid, nous pourrions les stocker dans un annuaire LDAP.

Dans LDAP, les données sont organisées de manière hiérarchique et structurée, ce qui facilite l'organisation et la recherche d'informations. Chaque entrée dans l'annuaire LDAP est identifiée de manière unique par son DN (Distinguished Name), qui suit une structure hiérarchique.

Quelques concepts clés à comprendre avec LDAP sont :

1. **Distinguished Name (DN)** : Il s'agit d'un nom unique qui identifie une entrée dans l'annuaire LDAP. Par exemple, `cn=John Doe,ou=users,dc=example,dc=com`.

2. **Common Name (CN)** : Il s'agit du nom de l'entrée, par exemple, "John Doe" dans l'exemple ci-dessus.

3. **Organizational Unit (OU)** : Il s'agit d'une subdivision au sein de l'organisation, par exemple, "users" dans l'exemple ci-dessus.

4. **Domain Component (DC)** : Il s'agit du domaine de niveau supérieur, par exemple, "example.com" dans l'exemple ci-dessus.

Nous explorerons plus en détail LDAP dans la note [[LDAP]] et son implémentation spécifique, OpenLDAP. 

Pour nos exercices de réflexion nous allons utiliser les données utilisateurs suivantes :
	- Noms,
	- Prénoms,
	- Date de naissance,
	- Nationalité,
	- Numéro de Coopérateur (attribué par la plate-forme)
	- Pseudonyme,
	- Adresse de courriel,
	- Langue d’interaction de rang 1
	- Langue d’interaction de rang 2
	- Prénom d’usage,
	- Nom d’usage,
	- Code postal,
	- Ville
	- pays de la résidence principale
	- Texte de profil utilisateur
	- Image de profil utilisateur / avatar

# 7. Authentification LDAP

## 7.1 Interaction avec un serveur LDAP en Python

### 7.1.1 Introduction à la bibliothèque ldap3 en Python
   
La bibliothèque ldap3 est une bibliothèque Python conçue pour être très facile à utiliser tout en restant très flexible et puissante. Elle est compatible avec Python 2 et 3 et fonctionne avec toutes les versions d'OpenLDAP.

```python
pip install ldap3
```

### 7.1.2 Comment se connecter à un serveur LDAP

Après avoir installé la bibliothèque, nous pouvons l'importer dans notre projet Python et l'utiliser pour nous connecter à notre serveur LDAP. Il est préférable de le faire dans un contexte de gestionnaire de ressources Python pour s'assurer que la connexion est correctement fermée lorsqu'elle n'est plus nécessaire.

```python
from ldap3 import Server, Connection, ALL

server = Server('my_ldap_server', get_info=ALL)
conn = Connection(server, 'my_user', 'my_password', auto_bind=True)
```

### 7.1.3 Effectuer des opérations CRUD (Créer, Lire, Mettre à jour, Supprimer)

Avec une connexion établie, nous pouvons maintenant effectuer des opérations CRUD. Par exemple, pour rechercher un utilisateur, nous pourrions faire :

```python
conn.search('ou=users,dc=example,dc=com', '(cn=my_user)')
```

Pour créer, mettre à jour et supprimer des entrées, nous utiliserions respectivement les méthodes `add`, `modify` et `delete` de l'objet `Connection`.

### 7.1.4 Exercices pratiques

Pour cet exercice, nous allons interagir avec un serveur OpenLDAP en utilisant 'ldap3'. Essayons de nous connecter à un serveur OpenLDAP, d'ajouter un nouvel utilisateur, de rechercher cet utilisateur, de modifier l'entrée de l'utilisateur et enfin de supprimer l'utilisateur.

## 7.2 Authentification LDAP avec Pyramid

### 7.2.1. Introduction à l'authentification LDAP avec Pyramid

Dans Pyramid, nous avons besoin d'une politique d'authentification pour vérifier les identifiants des utilisateurs. nous allons créer une politique d'authentification personnalisée pour notre application qui utilise OpenLDAP pour l'authentification.

### 7.2.2 Création d'une politique d'authentification personnalisée

Pour utiliser l'authentification LDAP avec Pyramid, nous allons créer une politique d'authentification personnalisée. Cette politique aura des méthodes pour obtenir l'ID de l'utilisateur authentifié et pour vérifier les autorisations de l'utilisateur.

```python
from pyramid.authentication import CallbackAuthenticationPolicy
from pyramid.security import Everyone
from ldap3 import Server, Connection, ALL, NTLM

class LdapAuthenticationPolicy(CallbackAuthenticationPolicy):
	def __init__(self, ldap_server):
		self.ldap_server = ldap_server

	def authenticated_userid(self, request):
		user_id = request.cookies.get('user_id')
		password = request.cookies.get('password')

		server = Server(self.ldap_server, get_info=ALL)
		conn = Connection(server, user_id, password, authentication=NTLM)
		conn.bind()

		if conn.bound:
			return user_id

	def effective_principals(self, request):
		principals = [Everyone]
		user_id = self.authenticated_userid(request)
		if user_id:
			principals.append(user_id)
		return principals
```

### 7.2.3 Utilisation de la politique d'authentification

Nous devons ensuite ajouter notre politique d'authentification à notre application Pyramid. nous pouvons le faire dans le fichier de configuration principal de notre application.

```python
from pyramid.config import Configurator
from .authentication import LdapAuthenticationPolicy

def main(global_config, **settings):
	config = Configurator(settings=settings)
	config.set_authentication_policy(LdapAuthenticationPolicy('my_ldap_server'))
	#...
```

### 7.2.4 Exercices pratiques

Essayons d'implémenter l'authentification LDAP dans une simple application Pyramid. nous pouvons commencer par une application Pyramid de base et ajouter l'authentification LDAP à l'aide de la classe `LdapAuthenticationPolicy` que nous avons définie. Testons l'application pour nous assurer que l'authentification fonctionne comme prévu.

## 7.3 Gestion des erreurs d'authentification

### 7.3.1 Introduction à la gestion des erreurs d'authentification

Il est crucial de gérer correctement les erreurs lors de la mise en œuvre de l'authentification. Il existe plusieurs types d'erreurs que nous devons prendre en compte, notamment les erreurs de connexion au serveur LDAP, les erreurs lors de la recherche d'utilisateurs et les erreurs lors de la vérification des mots de passe.

### 7.3.2 Gestion des erreurs de connexion

Si nous ne pouvons pas établir une connexion avec le serveur LDAP, nous devons renvoyer une erreur appropriée. nous pouvons le faire en utilisant un bloc try/except autour de notre code de connexion.

```python
try:
	conn.bind()
except LDAPException:
	# Handle the error
```

### 7.3.3 Gestion des erreurs lors de la recherche d'utilisateurs

Si nous ne trouvons pas l'utilisateur dans le serveur LDAP, nous devons renvoyer une erreur appropriée. nous pouvons le faire en vérifiant si la recherche a renvoyé un utilisateur.

```python
conn.search('ou=users,dc=my-domain,dc=com', '(uid={})'.format(user_id))
if not conn.entries:
	# Handle the error
```

7.3.4 **Gestion des erreurs lors de la vérification des mots de passe**

Si la vérification du mot de passe échoue, nous devons renvoyer une erreur appropriée. nous pouvons le faire en vérifiant si la méthode `bind()` renvoie `False`.

```python
if not conn.bind():
	# Handle the error
```

# 8. Déploiement de l'application Pyramid

## 8.1 Déploiement d'une application web

Le déploiement consiste à prendre une application qui fonctionne sur notre machine de développement et à la mettre à disposition sur Internet, de sorte que les utilisateurs puissent y accéder. Cela implique généralement de copier l'application sur un serveur web et de configurer ce serveur pour qu'il puisse exécuter l'application.

### 8.1.1 Importance du déploiement

Le déploiement est une étape importante du développement d'une application web. C'est le moment où notre application devient disponible pour le monde entier. Par conséquent, il est essentiel de veiller à ce que notre application soit aussi prête que possible pour le déploiement.

### 8.1.2 Options de déploiement pour Pyramid

Pyramid est un framework web flexible qui peut être déployé de plusieurs façons. Les options de déploiement les plus courantes pour Pyramid sont :

1. **WSGI** : Le serveur d'application Python le plus couramment utilisé. WSGI est l'interface standard entre les serveurs web et les applications web Python.
2. **uWSGI** : Un serveur d'application rapide et autonome qui peut servir des applications Pyramid.
3. **Gunicorn** : Un serveur HTTP Python WSGI HTTP pour UNIX.
4. **mod_wsgi** : Un module Apache qui fournit une interface WSGI pour les applications Python.

Toutes ces options ont leurs avantages et leurs inconvénients. Le choix de l'une ou l'autre dépendra de nos besoins spécifiques.

### 8.1.3 WSGI

WSGI est une spécification qui décrit comment un serveur web interagit avec des applications web en Python. Avant WSGI, il y avait une variété de méthodes non standardisées pour cette interaction. WSGI a été créé pour offrir une interface standardisée, de manière à ce que les applications web écrites en Python puissent être déployées sur n'importe quel serveur web supportant WSGI, indépendamment des détails spécifiques du serveur.

WSGI définit essentiellement deux rôles :

1. **Le côté serveur (ou "gateway")** : C'est généralement le serveur web lui-même, ou un module de celui-ci, qui reçoit les requêtes HTTP des clients (navigateurs web). Lorsqu'une requête arrive, le serveur la formate dans un format spécifique défini par la spécification WSGI.

2. **Le côté application** : C'est notre application web Python. Elle reçoit les requêtes du serveur sous forme de dictionnaires et de fonctions callback. L'application traite la requête et renvoie une réponse qui est ensuite renvoyée au client par le serveur.

Par exemple, dans une application Pyramid, le fichier que nous exécutons pour démarrer notre application contiendra généralement du code pour créer une instance d'application WSGI. C'est ce qui se passe lorsque nous appelons `config.make_wsgi_app()` dans notre code Pyramid.

Ce code génère une application WSGI qui peut ensuite être servie à l'aide d'un serveur WSGI. Dans les exemples précédents, nous avons utilisé `wsgiref.simple_server.make_server`, qui est un serveur WSGI simple fourni par la bibliothèque standard Python.

Dans une production réelle, nous utiliserions un serveur WSGI plus robuste, comme Gunicorn ou uWSGI, pour servir notre application.

C'est en substance ce qu'est WSGI, un élément clé de la plupart des applications web Python et un élément important pour comprendre comment fonctionnent les applications Pyramid.

## 8.2 Configuration de l'environnement de production

### 8.2.1 Introduction

Lorsque nous déployons une application Pyramid, la première étape consiste généralement à configurer l'environnement de production. Cela comprend l'installation de Python, la configuration du serveur web, l'installation des dépendances de l'application et la configuration de l'application elle-même.

### 8.2.2 Installation de Python

Le déploiement d'une application Pyramid nécessite une installation de Python sur notre serveur. La version exacte de Python dont nous avons besoin dépendra de notre application. Pour installer Python, nous pouvons généralement utiliser le gestionnaire de paquets de notre système.

### 8.2.3 Configuration du serveur web

Pour servir notre application Pyramid, nous aurons besoin d'un serveur web. Le choix du serveur web dépend de nos préférences personnelles et des besoins de notre application. Les options populaires incluent Nginx, Apache, et Gunicorn.

### 8.2.4 Installation des dépendances de l'application

Notre application Pyramid dépend probablement de plusieurs packages Python. nous pouvons installer ces dépendances à l'aide de pip, l'outil d'installation de paquets Python. Il est généralement recommandé de créer un environnement virtuel Python pour notre application afin d'éviter les conflits de dépendances.

### 8.2.5 Configuration de l'application

Enfin, nous devrons configurer notre application pour l'environnement de production. Cela peut inclure des choses comme la configuration des paramètres de connexion à la base de données, la configuration du logging, et la configuration des paramètres spécifiques à l'environnement dans notre fichier de configuration Pyramid.

### 8.3 Déploiement de l'application Pyramid sur un serveur local

### 8.3.1  Introduction

Après avoir configuré l'environnement de production, l'étape suivante consiste à déployer réellement l'application Pyramid sur le serveur. Cela comprend le transfert des fichiers de l'application sur le serveur, l'installation des dépendances, la configuration de l'application pour l'environnement de production et le démarrage du serveur de l'application.

### 8.3.2 Transfert des fichiers de l'application

Il existe de nombreuses façons de transférer les fichiers de notre application sur notre serveur. Une méthode courante consiste à utiliser Git. nous pouvons simplement pousser notre code sur un dépôt Git, puis le cloner sur notre serveur. Une autre méthode consiste à utiliser SCP (Secure Copy) ou FTP (File Transfer Protocol) pour transférer directement les fichiers.

### 8.3.3 Installation des dépendances

Une fois que nous avons transféré les fichiers de l'application, nous devrons installer les dépendances. Cela peut être fait en utilisant pip à l'intérieur de l'environnement virtuel que nous avons créé. Si nous avons un fichier `requirements.txt`, nous pouvons installer toutes les dépendances en une seule fois en utilisant la commande `pip install -r requirements.txt`.

### 8.3.4 Configuration de l'application

La configuration de notre application pour l'environnement de production peut nécessiter quelques étapes supplémentaires. Par exemple, nous devrons peut-être configurer la connexion à la base de données, les informations d'authentification ou autres paramètres spécifiques à l'environnement. 

### 8.3.5 Démarrage du serveur de l'application

Une fois que tout est en place, nous pouvons démarrer notre application Pyramid. La façon exacte de le faire dépend de la façon dont nous avons configuré notre serveur web. Par exemple, si nous utilisons Gunicorn, nous pouvons démarrer notre application en utilisant la commande `gunicorn myapp:app`.

# 9. Pyramid et Docker
Docker est une excellente option pour déployer des applications Pyramid car il nous permet d'encapsuler notre application et toutes ses dépendances dans un conteneur, ce qui facilite le déploiement et l'exécution de notre application dans divers environnements.

Pour utiliser Docker avec Pyramid, nous devrons créer un fichier `Dockerfile` qui décrit comment créer une image Docker pour notre application. Voici un exemple de base d'un `Dockerfile` pour une application Pyramid :

```Dockerfile
# Utilisins une image Python comme image de base
FROM python:3.9-slim-buster

# Créons un répertoire de travail dans le conteneur
WORKDIR /app

# Copions les fichiers de dépendance dans le conteneur
COPY requirements.txt .

# Installons les dépendances de l'application
RUN pip install --no-cache-dir -r requirements.txt

# Copions le reste du code de l'application dans le conteneur
COPY . .

# Exposons le port sur lequel notre application s'exécute
EXPOSE 6543

# Lançons le serveur de développement de Pyramid
CMD ["pserve", "development.ini"]
```

Notons que cette configuration utilise le serveur de développement inclus avec Pyramid, qui n'est pas destiné à être utilisé en production. Pour un déploiement en production, nous devrions utiliser un serveur WSGI comme Gunicorn ou uWSGI.

Pour construire une image Docker à partir de ce `Dockerfile`, nous pouvons utiliser la commande `docker build` :

```shell
docker build -t my-pyramid-app .
```

Et pour exécuter un conteneur basé sur cette image, nous pouvons utiliser la commande `docker run` :

```shell
docker run -p 6543:6543 my-pyramid-app
```

Cela démarre notre application Pyramid dans un conteneur Docker et expose le port 6543 pour que nous puissions y accéder.

Notons que la manière dont nous structurons notre `Dockerfile` et la façon dont nous utilisons Docker peuvent varier en fonction de nos besoins spécifiques. Par exemple, si nous utilisons une base de données, nous devrons probablement configurer notre application pour qu'elle puisse y accéder, et nous pourrions utiliser Docker Compose pour gérer à la fois notre application et notre base de données.

# 10. Exemples de code

## 10.1 Stockage d'une session en Zodb

Avant de commencer, assurons-nous d'avoir installé les dépendances requises pour travailler avec Pyramid et ZODB. Si ce n'est pas déjà fait, nous pouvons les installer avec la commande suivante :

```bash
pip install pyramid pyramid_zodb zodburi
```

Créons maintenant un petit exemple Pyramid utilisant la ZODB. Pour cet exemple, nous allons stocker un objet `Session` dans la base de données.

Commençons par définir une classe `Session` qui représente une session utilisateur. Cette classe peut être aussi simple ou complexe que nécessaire, mais pour cet exemple, nous allons simplement stocker un `id` de session et un `timestamp` :

```python
import time

from persistent import Persistent

class Session(Persistent):
    def __init__(self, session_id):
        self.session_id = session_id
        self.timestamp = time.time()
```

Ensuite, définissons une application Pyramid qui utilise ZODB pour stocker des objets `Session`. Pour cet exemple, nous allons simplement créer une nouvelle session chaque fois que la racine de l'application est visitée :

```python
from pyramid.config import Configurator
from pyramid.response import Response
from pyramid_zodbconn import get_connection
from ZODB.DB import DB
from ZODB.MappingStorage import MappingStorage
from transaction import commit

class MyModel(dict):
    pass

def root_factory(request):
    conn = get_connection(request)
    return conn.root()

def add_session(request):
    session_id = request.params.get('id', '0')
    session = Session(session_id)
    request.root[session_id] = session
    commit()
    return Response('Session {} added'.format(session_id))

def get_sessions(request):
    sessions = [(id, session.timestamp) for id, session in request.root.items()]
    return Response(str(sessions))

def main(global_config, **settings):
    """ This function returns a Pyramid WSGI application.
    """
    db = DB(MappingStorage())
    config = Configurator(settings=settings, root_factory=root_factory)
    config.add_request_method(get_connection, 'zodb_conn', reify=True)
    config.add_request_method(db, 'db', reify=True)
    config.add_view(add_session, name='add_session')
    config.add_view(get_sessions, name='get_sessions')
    return config.make_wsgi_app()
```

Dans ce code, `add_session` est une vue qui crée une nouvelle `Session` et l'ajoute à la racine de l'application ZODB. `get_sessions` est une autre vue qui renvoie une liste de toutes les sessions stockées dans la base de données.

Nous pouvons ensuite exécuter cette application Pyramid et visiter `/add_session?id=123` pour ajouter une session avec un id de 123, et `/get_sessions` pour voir une liste de toutes les sessions ajoutées.

## 10.2 Coupler l'authentification avec OpenLDAP

Pour coupler l'authentification Pyramid avec un serveur OpenLDAP, nous devrions utiliser une bibliothèque qui permet à notre application Python de communiquer avec le serveur LDAP. Une option est `python-ldap`, une interface vers les bibliothèques OpenLDAP.

Premièrement, installons `python-ldap` avec pip :

```shell
pip install python-ldap
```

Ensuite, nous aurons besoin de créer une `AuthenticationPolicy` personnalisée. Pyramid fournit une classe de base `CallbackAuthenticationPolicy` qui peut être utilisée pour cela :

```python
from pyramid.authentication import CallbackAuthenticationPolicy
import ldap

class LDAPAuthenticationPolicy(CallbackAuthenticationPolicy):
    def __init__(self, ldap_server):
        self.ldap_server = ldap_server

    def unauthenticated_userid(self, request):
        user = request.params.get('user')
        password = request.params.get('password')

        if user and password:
            try:
                ldap.set_option(ldap.OPT_X_TLS_REQUIRE_CERT, ldap.OPT_X_TLS_NEVER)
                l = ldap.initialize(self.ldap_server)
                l.simple_bind_s(user, password)
                return user
            except ldap.LDAPError:
                pass

    def remember(self, request, principal, **kw):
        pass

    def forget(self, request):
        pass
```

Dans ce code, `unauthenticated_userid` tente de se connecter au serveur LDAP avec le nom d'utilisateur et le mot de passe fournis. Si la connexion est réussie, il renvoie le nom d'utilisateur comme ID de l'utilisateur authentifié. Sinon, il renvoie `None`, indiquant que l'utilisateur n'est pas authentifié.

Nous pouvons ensuite utiliser cette `AuthenticationPolicy` dans notre application comme nous le ferions avec n'importe quelle autre `AuthenticationPolicy`. Par exemple :

```python
from pyramid.config import Configurator
from .authentication import LDAPAuthenticationPolicy

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.set_authentication_policy(LDAPAuthenticationPolicy('ldap://my.ldap.server'))
    # ...
    return config.make_wsgi_app()
```

Rappelons-nous que ce code est une implémentation très simple et que nous aurons probablement besoin de le personnaliser en fonction de nos besoins. En particulier, nous devrions gérer les situations où le nom d'utilisateur ou le mot de passe ne sont pas fournis, et nous voudrions probablement mettre en place une interface utilisateur pour permettre aux utilisateurs de se connecter.

De plus, notons que la communication avec le serveur LDAP doit se faire sur une connexion sécurisée (par exemple, LDAPS ou StartTLS) pour protéger les informations d'identification des utilisateurs. Dans l'exemple ci-dessus, nous avons désactivé la vérification du certificat pour la simplicité, mais dans une application réelle, nous devrions toujours vérifier le certificat du serveur.

## 10.3 Code avec vérification des certificats

La vérification des certificats en Python peut être gérée en utilisant l'option `OPT_X_TLS_CACERTDIR` ou `OPT_X_TLS_CACERTFILE` de python-ldap. nous pouvons utiliser ces options pour spécifier l'emplacement du fichier de certificat ou du répertoire contenant les certificats CA.

L'exemple suivant montre comment configurer python-ldap pour utiliser un fichier de certificats CA :

```python
from pyramid.authentication import CallbackAuthenticationPolicy
import ldap

class LDAPAuthenticationPolicy(CallbackAuthenticationPolicy):
    def __init__(self, ldap_server, ldap_cert):
        self.ldap_server = ldap_server
        self.ldap_cert = ldap_cert

    def unauthenticated_userid(self, request):
        user = request.params.get('user')
        password = request.params.get('password')

        if user and password:
            try:
                ldap.set_option(ldap.OPT_X_TLS_CACERTFILE, self.ldap_cert)
                ldap.set_option(ldap.OPT_X_TLS_REQUIRE_CERT, ldap.OPT_X_TLS_DEMAND)
                l = ldap.initialize(self.ldap_server)
                l.start_tls_s()
                l.simple_bind_s(user, password)
                return user
            except ldap.LDAPError:
                pass

    def remember(self, request, principal, **kw):
        pass

    def forget(self, request):
        pass
```

Ici, nous avons ajouté une nouvelle option `ldap_cert` à l'initialiseur `LDAPAuthenticationPolicy`, qui est utilisée pour définir l'emplacement du fichier de certificats CA. nous avons également remplacé `ldap.OPT_X_TLS_NEVER` par `ldap.OPT_X_TLS_DEMAND` pour exiger la vérification du certificat.

Ensuite, nous pouvons configurer notre application Pyramid pour utiliser cette politique d'authentification en fournissant l'URL de notre serveur LDAP et l'emplacement du fichier de certificats CA :

```python
from pyramid.config import Configurator
from .authentication import LDAPAuthenticationPolicy

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.set_authentication_policy(LDAPAuthenticationPolicy('ldaps://my.ldap.server', '/path/to/ca/cert.pem'))
    # ...
    return config.make_wsgi_app()
```

Ce code est une implémentation simple. Dans une application réelle, nous aurons probablement besoin de gérer les erreurs de manière plus robuste et de fournir une interface utilisateur pour la connexion. De plus, les informations d'identification des utilisateurs ne doivent jamais être manipulées ou stockées de manière non sécurisée.

## 10.4 Envoi de mail avec `pyramid_mailer`
Pyramid Mailer est un module d'extension (package) pour le framework web Python Pyramid. Il fournit une intégration facile pour gérer l'envoi d'e-mails depuis une application Pyramid.

Avec Pyramid Mailer, nous pouvons :

1. Envoyer des e-mails : Nous pouvons envoyer des e-mails transactionnels, des e-mails de confirmation, des e-mails de réinitialisation de mot de passe, etc.

2. Gérer les e-mails en file d'attente : Pyramid Mailer prend en charge l'envoi d'e-mails de manière asynchrone en les plaçant dans une file d'attente, ce qui permet de ne pas bloquer le traitement de la requête en cours.

3. Gérer les modèles d'e-mails : Nous pouvons définir des modèles pour les e-mails et remplir dynamiquement les champs du modèle avant de les envoyer.

4. Configurer les paramètres d'envoi : Nous pouvons configurer le serveur SMTP (Simple Mail Transfer Protocol) pour l'envoi des e-mails, ainsi que d'autres paramètres tels que le nom d'expéditeur, la réponse à l'adresse, etc.

Pour utiliser Pyramid Mailer dans notre application Pyramid, nous devons d'abord installer le package `pyramid_mailer`. nous pouvons le faire à l'aide de pip :

```bash
pip install pyramid_mailer
```

Nous devons alors configurer le serveur de messagerie :

Pour configurer le port et l'adresse du serveur SMTP dans Pyramid Mailer, nous pouvonq utiliser les options de configuration disponibles dans les paramètres de notre application Pyramid. Ces paramètres peuvent être définis dans le fichier `.ini` de configuration de notre application ou passés en tant que dictionnaire lors de la création de l'application Pyramid. Pour les secrets voir [[Pyramid#3.5.4 Gestion des Secrets dans Pyramid - Utilisation d'un fichier .env]]

Voici comment configurer le port et l'adresse du serveur SMTP dans Pyramid Mailer :

1. Dans le fichier `.ini` de configuration :

Dans notre fichier `.ini`, ajoutons les paramètres suivants pour spécifier le serveur SMTP, le port et autres options de configuration liées à l'envoi des e-mails :

```ini
[app:main]
# Autres configurations de notre application

# Configuration pour Pyramid Mailer
smtp.host = smtp.example.com
smtp.port = 587
smtp.username = your_smtp_username
smtp.password = your_smtp_password
smtp.tls = true
smtp.ssl = false
```

2. En utilisant un dictionnaire de configuration :

Si nous préférons passer les paramètres de configuration en tant que dictionnaire lors de la création de l'application Pyramid, nous pouvons faire :

```python
from pyramid.config import Configurator
from pyramid_mailer.mailer import Mailer

def main(global_config, **settings):
    config = Configurator(settings=settings)

    # Configurer Pyramid Mailer
    mailer = Mailer.from_settings(settings)
    config.include('pyramid_mailer')

    # Autres configurations de l'application

    return config.make_wsgi_app()
```

Dans ce cas, les paramètres de configuration pour Pyramid Mailer seront passés sous forme de clés/valeurs dans le dictionnaire `settings`.

Les principales options de configuration pour Pyramid Mailer sont les suivantes :

- `smtp.host`: L'adresse du serveur SMTP (par exemple, `smtp.example.com`).
- `smtp.port`: Le port du serveur SMTP (par exemple, `587` pour TLS ou `465` pour SSL).
- `smtp.username`: Le nom d'utilisateur du compte SMTP (si requis pour l'authentification).
- `smtp.password`: Le mot de passe du compte SMTP (si requis pour l'authentification).
- `smtp.tls`: Si défini sur `true`, active l'utilisation de TLS pour la communication avec le serveur SMTP.
- `smtp.ssl`: Si défini sur `true`, active l'utilisation de SSL pour la communication avec le serveur SMTP.

Une fois configuré, nous pouvons utiliser le composant `mailer` dans vos vues ou contrôleurs pour envoyer des e-mails. Voici un exemple simple :

```python
from pyramid_mailer import get_mailer
from pyramid_mailer.message import Message

def send_email(request):
    mailer = get_mailer(request)

    message = Message(
        subject="Hello from Pyramid Mailer",
        sender="noreply@example.com",
        recipients=["user@example.com"],
        body="This is the content of the email.",
    )

    mailer.send(message)

    return "Email sent successfully!"
```

Pyramid Mailer est un outil utile pour gérer les e-mails dans vos applications Pyramid de manière simple et efficace.

https://docs.pylonsproject.org/projects/pyramid_mailer/en/latest/index.html

# 11.  Trucs et astuces
## Débogage en ligne à travers le navigateur
### Utilisation de la toolbar de debogage dans Pyramid
Il suffit de cliquer sur le logo pyramid à droite du contenu et de naviguer dans les onglets.