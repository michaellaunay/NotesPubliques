---
schema_version: 1
uid: 01M1BQ62D3W619X770Z176W6VZ
titre: "Pyramid — 02 — Les Routes et Vues dans Pyramid"
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
resume: "Chapitre 2 sur 10 du livre « Pyramid » : Les Routes et Vues dans Pyramid. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « Pyramid » — chapitre 2/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 01 — Introduction à Pyramid et au développement web Python|← 01 — Introduction à Pyramid et au développement web Python]] · [[Pyramid — 03 — Gestion des requêtes et réponses|03 — Gestion des requêtes et réponses →]]

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

---
> [!info] Livre « Pyramid » — chapitre 2/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 01 — Introduction à Pyramid et au développement web Python|← 01 — Introduction à Pyramid et au développement web Python]] · [[Pyramid — 03 — Gestion des requêtes et réponses|03 — Gestion des requêtes et réponses →]]
