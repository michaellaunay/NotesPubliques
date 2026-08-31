---
schema_version: 1
uid: 01M1BQ62D466XDSJVTWFXYWXYE
titre: "Pyramid — 05 — Sécurisation des données"
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
resume: "Chapitre 5 sur 10 du livre « Pyramid » : Sécurisation des données. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « Pyramid » — chapitre 5/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 04 — Introduction à l'authentification|← 04 — Introduction à l'authentification]] · [[Pyramid — 06 — Introduction à LDAP et OpenLDAP|06 — Introduction à LDAP et OpenLDAP →]]

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

---
> [!info] Livre « Pyramid » — chapitre 5/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 04 — Introduction à l'authentification|← 04 — Introduction à l'authentification]] · [[Pyramid — 06 — Introduction à LDAP et OpenLDAP|06 — Introduction à LDAP et OpenLDAP →]]
