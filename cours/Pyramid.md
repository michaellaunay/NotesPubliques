---
schema_version: 1
uid: "01M02EX5C30AQ42BWV7D8MBK5K"
titre: "Pyramid"
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
resume: "Cours complet sur Pyramid 2.1 : architecture WSGI, configuration, routes, vues, traversal, sécurité moderne, sessions et CSRF, Deform, SQLAlchemy/ZODB, LDAP/OIDC, tests et déploiement."
niveau: avance
prerequis:
  - "[[Python]]"
  - "[[HTTP]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-06-14
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---

# Pyramid

> [!info] Version de référence
> Ce cours est mis à jour pour **Pyramid 2.1**, publié en mars 2026. Pyramid 2.1 nécessite **Python 3.10 ou plus récent** et prend officiellement en charge Python 3.10 à 3.14. La branche de développement de Pyramid peut déjà documenter des fonctions destinées à une future version 2.2 : elles ne doivent pas être considérées comme disponibles dans la version stable 2.1 tant qu'elles ne sont pas publiées.

Pyramid est un framework web Python du **Pylons Project**. Il suit l'idée historique :

> **Start small, finish big.**

Il fournit peu de décisions obligatoires mais beaucoup de points d'extension. On peut ainsi construire :

- une petite application WSGI dans un seul fichier ;
- une API HTTP ;
- une application HTML avec Jinja2 ou Chameleon ;
- une application métier importante avec SQLAlchemy ;
- une application orientée ressources avec traversal et ZODB ;
- une application intégrée à LDAP, OpenID Connect ou un SSO ;
- un service dont presque tous les composants sont remplaçables.

Pyramid est donc moins un « framework avec une façon unique de faire » qu'un **framework de composition**.

Voir aussi : [[Python]], [[HTTP]], [[Deform]], [[LDAP]], [[OAuth OpenID]], [[SQLAchemy]], [[Docker]].

---

# Sommaire

1. Pyramid et son positionnement
2. Installation et création d'un projet
3. Architecture et cycle de configuration
4. Requêtes et réponses
5. URL dispatch et routes
6. Vues, renderers et templates
7. Traversal et ressources
8. Sessions, cookies et CSRF
9. Authentification et autorisation modernes
10. Formulaires et Deform
11. Persistance : SQLAlchemy, transactions et ZODB
12. LDAP, OpenID Connect et authentification d'entreprise
13. Tweens, événements, subscribers et request methods
14. Fichiers statiques, i18n, logging et configuration
15. Tests
16. Outils de diagnostic Pyramid
17. Déploiement WSGI et reverse proxy
18. Docker et exploitation
19. Performances et cache
20. Durcissement de sécurité
21. Migration d'un ancien projet Pyramid 1.x
22. Architecture d'un projet de production
23. Travaux pratiques
24. Checklist finale
25. Références

---

# 1. Pyramid et son positionnement

## 1.1 Qu'est-ce que Pyramid ?

Pyramid est un **framework web WSGI** écrit en Python. Son cœur sait notamment :

- transformer une requête WSGI en objet `Request` ;
- choisir une vue ;
- produire une `Response` ;
- faire du routage par URL ;
- faire du traversal dans un arbre de ressources ;
- gérer des renderers ;
- configurer les sessions ;
- fournir un modèle de sécurité extensible ;
- exécuter des middlewares internes appelés **tweens** ;
- publier des événements ;
- intégrer des extensions via `config.include()`.

Pyramid ne fournit volontairement pas :

- un ORM obligatoire ;
- une base de données obligatoire ;
- un moteur de template obligatoire ;
- une solution d'authentification métier complète ;
- un serveur HTTP de production imposé.

Cette absence de choix imposés est une propriété centrale de Pyramid.

## 1.2 Historique

L'histoire simplifiée est la suivante :

1. **Pylons** était un framework web Python des années 2000.
2. **repoze.bfg**, issu de l'écosystème Repoze/Zope, proposait une approche très flexible du routage, du traversal et de la sécurité.
3. repoze.bfg est devenu **Pyramid** en 2011.
4. Pyramid est devenu le framework principal du **Pylons Project**.
5. Pyramid 2.0 a supprimé Python 2 et modernisé notamment le système de sécurité.
6. Pyramid 2.1, publié en 2026, modernise la compatibilité Python et l'outillage du projet.

L'héritage Zope explique plusieurs concepts que l'on rencontre encore :

- traversal ;
- ACL attachées aux ressources ;
- configuration déclarative ;
- ZODB ;
- extensibilité très fine.

## 1.3 Pyramid, Flask, Django et FastAPI

Une comparaison simplifiée :

| Framework | Positionnement dominant | ORM imposé | ASGI natif | Philosophie |
|---|---|---:|---:|---|
| Pyramid | framework web généraliste extensible | Non | Non, WSGI | composition |
| Flask | micro-framework WSGI | Non | Non | minimalisme |
| Django | framework intégré | Django ORM | Principalement WSGI/ASGI | batteries incluses |
| FastAPI | API typée | Non | Oui | API/ASGI/type hints |

Il ne faut pas en déduire qu'un framework est « meilleur » en général.

Pyramid est particulièrement intéressant lorsque :

- l'application possède une architecture métier spécifique ;
- nous voulons choisir précisément les briques ;
- traversal ou ACL par ressource sont utiles ;
- l'application existe depuis longtemps dans l'écosystème Pylons/Zope ;
- nous voulons rester dans WSGI ;
- la stabilité et l'explicitation de la configuration comptent davantage que la mode du moment.

## 1.4 WSGI : le contrat fondamental

Pyramid produit une **application WSGI**.

Une application WSGI est conceptuellement un callable :

```python
from collections.abc import Callable

WSGIApp = Callable
```

En pratique, le serveur fournit :

- un dictionnaire `environ` décrivant la requête ;
- une fonction `start_response` ;
- l'application renvoie un itérable de bytes.

Pyramid encapsule ce protocole dans des objets plus ergonomiques, principalement WebOb `Request` et `Response`.

> [!important]
> **Pyramid 2.1 reste un framework WSGI.** Il ne faut pas le présenter comme un framework ASGI natif. Pour des besoins très orientés WebSocket, connexions persistantes ou concurrence `asyncio` de bout en bout, un framework ASGI peut être plus naturel. On peut aussi séparer l'architecture en plusieurs services.

---

# 2. Installation et création d'un projet

## 2.1 Pré-requis

Pour Pyramid 2.1 :

- Python >= 3.10 ;
- un environnement virtuel ;
- `pip` ou un gestionnaire moderne compatible avec les paquets Python ;
- Git pour un vrai projet.

Vérification :

```bash
python3 --version
```

## 2.2 Créer un environnement virtuel

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools
```

Sous Windows PowerShell :

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip setuptools
```

Le nom `.venv` permet de garder l'environnement associé au projet.

## 2.3 Installer Pyramid

```bash
python -m pip install "pyramid==2.1"
```

Vérification :

```bash
python -c "import pyramid; print(pyramid.__version__)"
```

Selon le packaging de Pyramid, `__version__` n'est pas toujours l'API la plus robuste pour interroger une version. La méthode générique Python est :

```bash
python -c "from importlib.metadata import version; print(version('pyramid'))"
```

## 2.4 Première application dans un fichier

Créons `app.py` :

```python
from wsgiref.simple_server import make_server

from pyramid.config import Configurator
from pyramid.response import Response


def hello(request):
    return Response("Bonjour Pyramid !")


def make_app():
    with Configurator() as config:
        config.add_route("home", "/")
        config.add_view(hello, route_name="home")
        return config.make_wsgi_app()


if __name__ == "__main__":
    app = make_app()
    server = make_server("127.0.0.1", 6543, app)
    server.serve_forever()
```

Puis :

```bash
python app.py
```

Cette méthode est excellente pour comprendre Pyramid, mais une vraie application sera généralement empaquetée.

## 2.5 `pcreate` est obsolète et supprimé

De vieux cours Pyramid utilisent :

```bash
pcreate -s starter mon_projet
```

Cette commande appartient aux anciens scaffolds Pyramid. **`pcreate` a été retiré avec Pyramid 2.0.**

Le Pylons Project recommande aujourd'hui le cookiecutter officiel.

## 2.6 Cookiecutter officiel

Installation :

```bash
python -m pip install cookiecutter
```

Pour Pyramid 2.1, utilisons explicitement la branche correspondante :

```bash
cookiecutter gh:Pylons/pyramid-cookiecutter-starter --checkout 2.1-branch
```

Le générateur propose notamment :

### Template

- Jinja2 ;
- Chameleon ;
- Mako.

### Backend

- `none` ;
- `sqlalchemy` ;
- `zodb`.

Ces choix ont aussi une conséquence architecturale :

- `none` : URL dispatch ;
- `sqlalchemy` : SQLAlchemy + Alembic + URL dispatch ;
- `zodb` : ZODB + traversal.

## 2.7 Exemple de création d'un projet

```bash
cookiecutter gh:Pylons/pyramid-cookiecutter-starter --checkout 2.1-branch
cd mon_projet
python3 -m venv .venv
.venv/bin/pip install --upgrade pip setuptools
.venv/bin/pip install -e ".[testing]"
.venv/bin/pytest
.venv/bin/pserve development.ini --reload
```

Le cookiecutter actuel utilise un packaging moderne avec `pyproject.toml`.

## 2.8 Structure typique

Une structure fréquente :

```text
mon_projet/
├── development.ini
├── production.ini
├── pyproject.toml
├── README.txt
├── mon_projet/
│   ├── __init__.py
│   ├── routes.py
│   ├── security.py
│   ├── models/
│   ├── views/
│   ├── templates/
│   └── static/
└── tests/
```

Il ne s'agit pas d'une structure imposée par le cœur de Pyramid. C'est une convention utile.

---

# 3. Architecture et cycle de configuration

## 3.1 Le rôle de `Configurator`

Le point central est `pyramid.config.Configurator`.

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    with Configurator(settings=settings) as config:
        config.add_route("home", "/")
        config.scan(".views")
        return config.make_wsgi_app()
```

Le configurateur accumule des **actions de configuration** puis les rend effectives lors du `commit` implicite ou de `make_wsgi_app()`.

## 3.2 Pourquoi cette phase de configuration ?

Elle permet à Pyramid :

- de détecter certaines configurations incompatibles ;
- de combiner des extensions ;
- d'introspecter les routes et vues ;
- de séparer la phase de composition de l'application de la phase de traitement des requêtes.

## 3.3 Configuration impérative

On peut tout déclarer en Python :

```python
from pyramid.config import Configurator
from pyramid.response import Response


def home(request):
    return Response("Accueil")


def make_app():
    with Configurator() as config:
        config.add_route("home", "/")
        config.add_view(home, route_name="home")
        return config.make_wsgi_app()
```

## 3.4 Configuration déclarative avec décorateurs

```python
from pyramid.view import view_config


@view_config(route_name="home", renderer="json")
def home(request):
    return {"message": "Bonjour"}
```

Pour que Pyramid découvre ce décorateur :

```python
config.scan(".views")
```

Pyramid utilise **Venusian** pour attacher des métadonnées de configuration aux objets décorés.

> [!tip]
> Dans une grosse application, il est souvent préférable de limiter `scan()` à des modules/packages précis au lieu de scanner aveuglément tout le projet.

## 3.5 `config.include()`

Une extension ou un module local peut exposer :

```python
def includeme(config):
    config.add_route("health", "/health")
```

Puis :

```python
config.include(".health")
```

Ce pattern permet de construire une application modulaire :

```python
def main(global_config, **settings):
    with Configurator(settings=settings) as config:
        config.include(".routes")
        config.include(".security")
        config.include(".db")
        config.scan(".views")
        return config.make_wsgi_app()
```

## 3.6 Paramètres et fichiers `.ini`

Pyramid est souvent chargé via PasteDeploy :

```ini
[app:main]
use = egg:mon_projet
app.secret = change-me
sqlalchemy.url = postgresql+psycopg://user:pass@db/app

[server:main]
use = egg:waitress#main
listen = 127.0.0.1:6543
```

La factory :

```python
def main(global_config, **settings):
    secret = settings["app.secret"]
```

Les valeurs d'un `.ini` sont essentiellement des chaînes. Il faut donc convertir explicitement :

```python
from pyramid.settings import asbool


debug_enabled = asbool(settings.get("app.debug", "false"))
```

## 3.7 Ne pas stocker les secrets dans Git

Un fichier `production.ini` commité ne doit pas contenir :

- mot de passe de base ;
- secret de cookie ;
- token d'API ;
- bind password LDAP ;
- clé privée.

Une stratégie possible consiste à injecter les secrets par variables d'environnement au démarrage puis à construire les settings.

Exemple minimal :

```python
import os


def require_env(name):
    value = os.environ.get(name)
    if not value:
        raise RuntimeError(f"Variable requise absente : {name}")
    return value
```

Pour un projet important, un gestionnaire de secrets est préférable à un simple `.env` distribué sur les machines.

---

# 4. Requêtes et réponses

## 4.1 Objet `request`

Une vue reçoit généralement un objet `pyramid.request.Request`, basé sur WebOb.

Attributs utiles :

```python
def inspect_request(request):
    return {
        "method": request.method,
        "path": request.path,
        "url": request.url,
        "query": request.GET.mixed(),
        "content_type": request.content_type,
        "accept": str(request.accept),
        "user_agent": request.user_agent,
    }
```

## 4.2 Query string et formulaire

Pour une URL :

```text
/search?q=pyramid&page=2
```

```python
q = request.GET.get("q", "")
page = int(request.GET.get("page", "1"))
```

Pour un formulaire URL-encoded :

```python
username = request.POST.get("username")
```

`request.params` fusionne certains paramètres GET/POST. Cette commodité peut rendre l'origine d'une valeur ambiguë. Dans du code sensible, préférons souvent `request.GET` ou `request.POST` explicitement.

## 4.3 JSON

```python
from pyramid.httpexceptions import HTTPBadRequest


def api_view(request):
    try:
        payload = request.json_body
    except ValueError as exc:
        raise HTTPBadRequest("JSON invalide") from exc
    return {"received": payload}
```

Avec :

```python
@view_config(route_name="api", request_method="POST", renderer="json")
def api_view(request):
    ...
```

## 4.4 Headers

```python
request.headers.get("Authorization")
request.headers.get("If-Match")
request.headers.get("X-Request-ID")
```

Ne faisons jamais confiance à un header simplement parce qu'il existe. Certains headers ne sont fiables que s'ils ont été nettoyés et réécrits par notre reverse proxy de confiance.

## 4.5 Réponses

Réponse texte :

```python
from pyramid.response import Response


def text_view(request):
    return Response("Bonjour", content_type="text/plain", charset="utf-8")
```

JSON via renderer :

```python
@view_config(route_name="status", renderer="json")
def status(request):
    return {"status": "ok"}
```

## 4.6 Exceptions HTTP

Pyramid fournit des exceptions servant aussi de réponses HTTP :

```python
from pyramid.httpexceptions import (
    HTTPBadRequest,
    HTTPForbidden,
    HTTPFound,
    HTTPNotFound,
)
```

Exemple :

```python
if item is None:
    raise HTTPNotFound("Objet introuvable")
```

Redirection :

```python
return HTTPFound(location=request.route_url("home"))
```

## 4.7 Ne pas confondre validation et parsing

Ce code est insuffisant :

```python
age = int(request.POST["age"])
```

Il faut définir :

- type attendu ;
- bornes ;
- longueur ;
- format ;
- valeurs autorisées ;
- comportement en cas d'absence ;
- erreurs retournées au client.

Pour des formulaires complexes, voir [[Deform]].

---

# 5. URL dispatch et routes

## 5.1 Ajouter une route

```python
config.add_route("home", "/")
config.add_route("user", "/users/{user_id}")
```

Une route associe :

- un nom ;
- un pattern ;
- éventuellement des prédicats.

## 5.2 Paramètres de route

```python
@view_config(route_name="user", renderer="json")
def user_view(request):
    user_id = request.matchdict["user_id"]
    return {"user_id": user_id}
```

## 5.3 Générer les URLs

Ne concaténons pas manuellement les URLs :

```python
url = request.route_url("user", user_id="42")
path = request.route_path("user", user_id="42")
```

Avantage : si le pattern change, les appels restent cohérents.

## 5.4 Prédicat de méthode HTTP

```python
@view_config(
    route_name="user",
    request_method="GET",
    renderer="json",
)
def get_user(request):
    ...
```

```python
@view_config(
    route_name="user",
    request_method="DELETE",
    renderer="json",
)
def delete_user(request):
    ...
```

## 5.5 Autres prédicats

Pyramid peut sélectionner une vue selon :

- route ;
- méthode ;
- `request_param` ;
- header ;
- content type ;
- accept ;
- permission ;
- contexte ;
- classe de requête ;
- prédicats personnalisés.

Le mécanisme est puissant, mais des règles de sélection trop nombreuses rendent le système difficile à comprendre.

## 5.6 Pattern et contraintes

Exemple :

```python
config.add_route("article", "/articles/{id:\\d+}")
```

Puis :

```python
article_id = int(request.matchdict["id"])
```

La contrainte de route ne remplace pas la validation métier.

## 5.7 Route factory et contexte

Une route peut produire un contexte :

```python
class ArticleResource:
    def __init__(self, request):
        self.request = request


config.add_route(
    "article",
    "/articles/{id}",
    factory=ArticleResource,
)
```

C'est utile pour faire converger URL dispatch et sécurité par contexte.

## 5.8 Vue 404

```python
from pyramid.view import notfound_view_config


@notfound_view_config(renderer="templates/404.pt")
def not_found(request):
    request.response.status = 404
    return {}
```

## 5.9 Vue d'exception

```python
from pyramid.view import exception_view_config


@exception_view_config(ValueError, renderer="json")
def value_error(exc, request):
    request.response.status = 400
    return {"error": str(exc)}
```

En production, ne renvoyons pas directement au client des détails d'exception internes susceptibles de révéler des secrets ou la structure du système.

---

# 6. Vues, renderers et templates

## 6.1 Vue fonction

```python
@view_config(route_name="home", renderer="templates/home.pt")
def home(request):
    return {"title": "Accueil"}
```

Quand un renderer est configuré, le dictionnaire retourné devient le contexte de rendu.

## 6.2 Vue classe

```python
from pyramid.view import view_config


class UserViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name="users", request_method="GET", renderer="json")
    def list_users(self):
        return {"users": []}
```

Les classes sont utiles lorsque plusieurs vues partagent :

- des dépendances ;
- un contexte ;
- des helpers ;
- une logique commune.

Évitons toutefois les classes géantes servant de contrôleur monolithique.

## 6.3 Renderers

Pyramid sait notamment travailler avec :

- `json` ;
- `string` ;
- moteurs de templates ajoutés via extensions.

Exemple JSON :

```python
@view_config(route_name="health", renderer="json")
def health(request):
    return {"ok": True}
```

## 6.4 Chameleon

Avec `pyramid_chameleon` :

```python
config.include("pyramid_chameleon")
```

Template `home.pt` :

```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="utf-8">
    <title>${title}</title>
  </head>
  <body>
    <h1>${title}</h1>
  </body>
</html>
```

Chameleon utilise notamment TAL/METAL.

## 6.5 Jinja2

Le cookiecutter peut utiliser Jinja2 via `pyramid_jinja2`.

```python
config.include("pyramid_jinja2")
```

Exemple :

```html
<h1>{{ title }}</h1>
```

Le choix Chameleon/Jinja2 est surtout une question :

- d'écosystème existant ;
- de préférences d'équipe ;
- de compatibilité avec un projet historique.

## 6.6 Échappement HTML

Un moteur de template correctement configuré aide à échapper les données affichées, mais cela ne dispense pas de comprendre le contexte :

- HTML ;
- attribut ;
- JavaScript ;
- URL ;
- CSS.

Une chaîne sûre dans un contexte n'est pas automatiquement sûre dans un autre.

Voir [[HTML]], [[Javascript]] et [[Sécurité avec Python]].

## 6.7 Fichiers statiques

```python
config.add_static_view(
    name="static",
    path="mon_projet:static",
    cache_max_age=3600,
)
```

En Pyramid 2.1, le chemin d'une static view doit correspondre à un package réel : l'utilisation d'un package inexistant est dépréciée.

En production, les fichiers statiques peuvent être servis par :

- Nginx ;
- un CDN ;
- un object storage ;
- l'application elle-même pour les cas simples.

---

# 7. Traversal et ressources

## 7.1 Deux modèles de dispatch

Pyramid possède deux grands modèles :

### URL dispatch

```text
URL -> route -> vue
```

### Traversal

```text
URL -> arbre de ressources -> contexte -> vue
```

Ils peuvent être combinés.

## 7.2 Exemple conceptuel de traversal

Pour :

```text
/projects/42/files/report.pdf
```

Pyramid peut traverser successivement :

```text
root
└── projects
    └── 42
        └── files
            └── report.pdf
```

Chaque objet peut :

- représenter une ressource métier ;
- fournir un `__parent__` ;
- fournir un `__name__` ;
- définir un `__acl__`.

## 7.3 Ressource simple

```python
class Resource:
    def __init__(self, name, parent=None):
        self.__name__ = name
        self.__parent__ = parent
        self.children = {}

    def __getitem__(self, key):
        return self.children[key]
```

## 7.4 Quand préférer traversal ?

Traversal est particulièrement naturel lorsque :

- le domaine est hiérarchique ;
- l'autorisation dépend de la ressource ;
- les ACL doivent être héritées ;
- nous utilisons ZODB ;
- les URLs reflètent un arbre métier.

URL dispatch reste plus familier pour de nombreuses API REST et applications CRUD classiques.

---

# 8. Sessions, cookies et CSRF

## 8.1 Cookie et session ne sont pas synonymes

Un cookie est une donnée HTTP stockée côté client.

Une session est un mécanisme applicatif permettant de conserver un état logique entre requêtes.

Une session peut être :

- stockée dans un cookie signé ;
- stockée côté serveur avec seulement un identifiant en cookie ;
- stockée dans Redis ;
- stockée dans une base ;
- implémentée autrement.

## 8.2 `SignedCookieSessionFactory`

```python
from pyramid.session import SignedCookieSessionFactory


def make_session_factory(secret):
    return SignedCookieSessionFactory(
        secret,
        secure=True,
        httponly=True,
        samesite="Lax",
        timeout=1200,
        reissue_time=120,
    )
```

Puis :

```python
session_factory = make_session_factory(settings["session.secret"])
config = Configurator(settings=settings, session_factory=session_factory)
```

> [!warning]
> Une session **signée** n'est pas forcément **chiffrée**. Le client ne doit pas pouvoir modifier son contenu sans détection, mais il peut potentiellement en voir le contenu. Ne plaçons pas de mot de passe, secret API ou donnée hautement sensible dans une session cookie.

La taille est également limitée par la taille des cookies HTTP.

## 8.3 Utiliser la session

```python
request.session["cart_count"] = 3
count = request.session.get("cart_count", 0)
```

Message flash :

```python
request.session.flash("Profil mis à jour", queue="success")
messages = request.session.pop_flash(queue="success")
```

## 8.4 Cookies manuels

```python
response = request.response
response.set_cookie(
    "preference",
    "compact",
    secure=True,
    httponly=True,
    samesite="Lax",
)
```

Règles générales :

- `Secure` en production HTTPS ;
- `HttpOnly` pour les cookies que JavaScript n'a pas besoin de lire ;
- `SameSite` adapté au flux ;
- domaine le plus étroit possible ;
- durée minimale nécessaire.

## 8.5 CSRF

Une attaque CSRF pousse le navigateur d'un utilisateur authentifié à envoyer une requête non désirée vers une application qui fait confiance aux cookies de cet utilisateur.

Pyramid dispose d'une API dédiée dans `pyramid.csrf`.

Configuration :

```python
from pyramid.csrf import CookieCSRFStoragePolicy


def includeme(config):
    config.set_csrf_storage_policy(
        CookieCSRFStoragePolicy(
            secure=True,
            samesite="Lax",
        )
    )
    config.set_default_csrf_options(require_csrf=True)
```

Pyramid vérifie alors par défaut les méthodes non sûres selon la configuration.

## 8.6 Token dans un formulaire

Dans un template compatible :

```html
<form method="post">
  <input type="hidden" name="csrf_token" value="${get_csrf_token()}">
  <button type="submit">Enregistrer</button>
</form>
```

Pour une requête JavaScript, le token peut être transmis via le header configuré, par défaut :

```text
X-CSRF-Token
```

## 8.7 CORS n'est pas une protection CSRF universelle

CORS contrôle principalement la capacité d'un script d'une autre origine à lire ou effectuer certains types de requêtes selon les règles navigateur.

Cela ne remplace pas :

- token CSRF ;
- `SameSite` ;
- vérification `Origin` ;
- validation des méthodes et contenus.

Voir [[HTTP]].

---

# 9. Authentification et autorisation modernes

## 9.1 Rupture importante depuis Pyramid 2.0

Les anciens cours utilisent souvent :

```python
AuthTktAuthenticationPolicy
SessionAuthenticationPolicy
ACLAuthorizationPolicy
config.set_authentication_policy(...)
config.set_authorization_policy(...)
```

Ces APIs sont **dépréciées depuis Pyramid 2.0**.

Le modèle moderne utilise une **Security Policy unique** :

```python
config.set_security_policy(policy)
```

et expose notamment :

```python
request.identity
request.authenticated_userid
request.is_authenticated
```

## 9.2 Authentification vs autorisation

### Authentification

> Qui est l'utilisateur ?

### Autorisation

> Cette identité peut-elle effectuer cette action sur cette ressource ?

Ne mélangeons pas les deux.

## 9.3 Security Policy avec session

Exemple pédagogique :

```python
from dataclasses import dataclass

from pyramid.authentication import SessionAuthenticationHelper
from pyramid.security import Allowed, Denied


@dataclass(frozen=True)
class User:
    id: str
    email: str
    groups: tuple[str, ...] = ()


USERS = {
    "alice": User("alice", "alice@example.org", ("editors",)),
}


class SecurityPolicy:
    def __init__(self):
        self.helper = SessionAuthenticationHelper()

    def identity(self, request):
        userid = self.helper.authenticated_userid(request)
        if userid is None:
            return None
        return USERS.get(userid)

    def authenticated_userid(self, request):
        identity = self.identity(request)
        return identity.id if identity is not None else None

    def permits(self, request, context, permission):
        identity = self.identity(request)
        if permission == "view":
            return Allowed("lecture publique")
        if permission == "edit" and identity is not None:
            if "editors" in identity.groups:
                return Allowed("membre du groupe editors")
        return Denied("permission refusée")

    def remember(self, request, userid, **kw):
        return self.helper.remember(request, userid, **kw)

    def forget(self, request, **kw):
        return self.helper.forget(request, **kw)
```

Configuration :

```python
config.set_security_policy(SecurityPolicy())
```

## 9.4 Connexion et déconnexion

```python
from pyramid.httpexceptions import HTTPFound
from pyramid.security import forget, remember
from pyramid.view import view_config


@view_config(route_name="login", request_method="POST")
def login(request):
    username = request.POST.get("username", "")
    password = request.POST.get("password", "")

    if verify_credentials(username, password):
        headers = remember(request, username)
        return HTTPFound(
            location=request.route_url("home"),
            headers=headers,
        )

    request.response.status = 401
    return request.response


@view_config(route_name="logout", request_method="POST")
def logout(request):
    headers = forget(request)
    return HTTPFound(
        location=request.route_url("home"),
        headers=headers,
    )
```

Ici `verify_credentials` représente notre code métier.

## 9.5 Mot de passe

Un mot de passe utilisateur ne doit jamais être :

- stocké en clair ;
- chiffré de manière réversible comme mécanisme principal de stockage ;
- hashé avec SHA-256 seul ;
- loggé.

Utilisons une fonction de dérivation lente et adaptée, par exemple Argon2id ou bcrypt selon l'environnement et la politique de sécurité.

Exemple avec Argon2 :

```python
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError


ph = PasswordHasher()


def hash_password(password):
    return ph.hash(password)


def check_password(password_hash, password):
    try:
        return ph.verify(password_hash, password)
    except VerifyMismatchError:
        return False
```

## 9.6 Autorisation déclarative sur une vue

```python
@view_config(
    route_name="document_edit",
    request_method="POST",
    permission="edit",
    renderer="json",
)
def edit_document(request):
    ...
```

Pyramid demande alors à la security policy si la permission est accordée pour le contexte courant.

## 9.7 ACL avec `ACLHelper`

Pyramid conserve un modèle ACL puissant. L'API moderne utilise notamment `pyramid.authorization.ACLHelper`.

```python
from pyramid.authorization import ACLHelper, Authenticated, Everyone


class ACLSecurityPolicy:
    def __init__(self, auth_helper):
        self.auth_helper = auth_helper
        self.acl = ACLHelper()

    def identity(self, request):
        ...

    def authenticated_userid(self, request):
        identity = self.identity(request)
        return identity.id if identity else None

    def principals(self, request):
        principals = [Everyone]
        identity = self.identity(request)
        if identity is not None:
            principals.append(Authenticated)
            principals.append(f"user:{identity.id}")
            principals.extend(f"group:{g}" for g in identity.groups)
        return principals

    def permits(self, request, context, permission):
        return self.acl.permits(
            context,
            self.principals(request),
            permission,
        )

    def remember(self, request, userid, **kw):
        return self.auth_helper.remember(request, userid, **kw)

    def forget(self, request, **kw):
        return self.auth_helper.forget(request, **kw)
```

Ressource :

```python
from pyramid.authorization import Allow, Everyone


class Document:
    __acl__ = [
        (Allow, Everyone, "view"),
        (Allow, "group:editors", "edit"),
    ]
```

## 9.8 `request.identity`

Le grand intérêt de Pyramid 2.x est que l'identité n'est plus limitée conceptuellement à une simple liste de principals.

Elle peut être :

- un objet ORM ;
- une dataclass ;
- un objet LDAP projeté ;
- un objet représentant les claims OIDC validés.

Exemple :

```python
identity = request.identity
if identity is not None:
    print(identity.email)
```

## 9.9 Ne pas inventer un protocole d'authentification

Pour un SSO ou une authentification fédérée, préférons des protocoles standard :

- OpenID Connect ;
- OAuth 2.x pour l'autorisation ;
- éventuellement SAML dans certains SI historiques.

Voir [[OAuth OpenID]].

---

# 10. Formulaires et Deform

Pyramid n'impose pas de bibliothèque de formulaires. Dans l'écosystème Pylons, **Deform + Colander** est une combinaison historique et toujours pertinente.

Voir le cours complet [[Deform]].

## 10.1 Rôle des bibliothèques

```text
Pyramid
  reçoit la requête
      |
      v
Colander
  décrit et valide les données
      |
      v
Deform
  construit et rend le formulaire
```

## 10.2 Schéma Colander

```python
import colander


class RegistrationSchema(colander.MappingSchema):
    email = colander.SchemaNode(colander.String())
    display_name = colander.SchemaNode(colander.String())
```

## 10.3 Formulaire Deform

```python
import deform


schema = RegistrationSchema()
form = deform.Form(schema, buttons=("submit",))
```

## 10.4 Validation

```python
from deform.exception import ValidationFailure


try:
    appstruct = form.validate(request.POST.items())
except ValidationFailure as exc:
    html = exc.render()
```

Le résultat validé doit encore passer les règles métier :

- unicité de l'email ;
- droits de création ;
- politique de mot de passe ;
- état de l'organisation.

## 10.5 CSRF avec formulaire

Une application utilisant des cookies d'authentification doit protéger les mutations. Le formulaire doit donc intégrer le mécanisme CSRF choisi pour l'application.

---

# 11. Persistance : SQLAlchemy, transactions et ZODB

## 11.1 Pyramid n'impose aucune persistance

Nous pouvons utiliser :

- PostgreSQL ;
- SQLite ;
- MariaDB ;
- ZODB ;
- Redis ;
- LDAP ;
- API externe ;
- fichiers ;
- plusieurs systèmes simultanément.

## 11.2 SQLAlchemy 2.x

Le cookiecutter `sqlalchemy` de Pyramid 2.1 est aligné avec les patterns SQLAlchemy 2.x et utilise Alembic pour les migrations.

Voir [[SQLAchemy]].

Exemple de base moderne :

```python
from sqlalchemy import MetaData
from sqlalchemy.orm import DeclarativeBase


NAMING_CONVENTION = {
    "ix": "ix_%(column_0_label)s",
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s",
}


class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=NAMING_CONVENTION)
```

## 11.3 Transaction manager

L'écosystème Pyramid utilise fréquemment :

- `transaction` ;
- `pyramid_tm` ;
- `zope.sqlalchemy`.

Le principe est :

```text
requête
  -> début unité de travail
  -> vue
  -> succès : commit
  -> erreur : abort
```

Cela permet d'éviter un grand nombre de `commit()` dispersés dans les vues.

## 11.4 `pyramid_tm` n'est pas un gestionnaire de session web

Une confusion fréquente consiste à classer ensemble :

- `pyramid.session` ;
- `pyramid_tm` ;
- session SQLAlchemy.

Ils n'ont pas le même rôle.

### Session web Pyramid

État utilisateur entre requêtes.

### Transaction manager

Unité atomique de travail.

### Session SQLAlchemy

Contexte d'interaction ORM avec la base.

Le mot « session » désigne donc plusieurs concepts différents.

## 11.5 ZODB

Le backend `zodb` du cookiecutter officiel existe encore dans Pyramid 2.1.

Il est associé naturellement à :

- traversal ;
- objets persistants ;
- `pyramid_zodbconn` ;
- `pyramid_tm` ;
- package `transaction`.

ZODB peut être très adapté à un modèle objet hiérarchique, mais il ne faut pas le choisir simplement parce qu'il évite SQL.

## 11.6 N+1 et durée des transactions

Une vue ne doit pas :

- ouvrir une transaction très longue ;
- faire un appel réseau lent au milieu d'une transaction DB si évitable ;
- déclencher des centaines de requêtes ORM sans s'en rendre compte.

Mesurons :

- nombre de requêtes SQL ;
- latence DB ;
- contention ;
- taille des résultats.

---

# 12. LDAP, OpenID Connect et authentification d'entreprise

## 12.1 LDAP n'est pas un protocole d'authentification web

LDAP est un protocole d'accès à un annuaire.

Une application Pyramid peut l'utiliser pour :

- rechercher une identité ;
- récupérer ses groupes ;
- vérifier un bind ;
- lire des attributs ;
- synchroniser des comptes.

Voir [[LDAP]].

## 12.2 Ne jamais désactiver la validation TLS

Ancien anti-pattern :

```python
# À NE PAS FAIRE EN PRODUCTION
# ldap.set_option(ldap.OPT_X_TLS_REQUIRE_CERT, ldap.OPT_X_TLS_NEVER)
```

Désactiver la validation du certificat rend une interception des identifiants possible.

Utilisons :

- LDAPS correctement configuré ; ou
- StartTLS ;
- une CA de confiance ;
- validation stricte du nom et du certificat.

## 12.3 Recherche avec `ldap3`

Exemple pédagogique :

```python
from ldap3 import Connection, Server, Tls


def find_user(ldap_url, bind_dn, bind_password, base_dn, username):
    tls = Tls(validate=2)
    server = Server(ldap_url, use_ssl=True, tls=tls)
    with Connection(
        server,
        user=bind_dn,
        password=bind_password,
        auto_bind=True,
    ) as conn:
        escaped_username = username.replace("*", r"\\2a")
        conn.search(
            base_dn,
            f"(uid={escaped_username})",
            attributes=["uid", "mail", "memberOf"],
        )
        return conn.entries[0] if conn.entries else None
```

Dans un vrai projet, utilisons les fonctions d'échappement fournies par la bibliothèque plutôt que de construire manuellement un filtre LDAP. L'exemple ci-dessus est volontairement simplifié.

## 12.4 Pattern d'authentification LDAP

Une approche sûre :

1. formulaire HTTPS ;
2. validation CSRF ;
3. recherche de l'entrée avec un compte technique limité ;
4. tentative de bind avec le DN exact de l'utilisateur ;
5. en cas de succès, création de la session Pyramid ;
6. le mot de passe est oublié immédiatement ;
7. les groupes LDAP sont projetés en permissions applicatives.

Le mot de passe LDAP ne doit pas être stocké dans la session Pyramid.

## 12.5 LDAP dans une Security Policy

La Security Policy ne doit pas effectuer un bind LDAP avec le mot de passe utilisateur à chaque requête.

Le bind intervient au **login**. Ensuite une session courte identifie l'utilisateur.

La policy peut recharger :

- profil ;
- groupes ;
- statut de compte ;

avec une politique de cache adaptée au niveau de risque.

## 12.6 OpenID Connect souvent préférable pour le SSO

Pour une application web moderne intégrée à un fournisseur d'identité, OpenID Connect évite généralement de confier directement le mot de passe à l'application.

Architecture :

```text
Navigateur
   |
   v
Application Pyramid
   |
   +---- redirection ----> Identity Provider
                            |
                            +-- MFA / passkey / politique SSO
                            |
   <--- authorization code-+
   |
   v
validation tokens + session locale
```

Voir [[OAuth OpenID]].

## 12.7 LDAP et OIDC peuvent coexister

Un Identity Provider peut lui-même être connecté à LDAP.

Ainsi :

```text
Pyramid -> OIDC -> Keycloak/IdP -> LDAP
```

est souvent préférable à :

```text
Pyramid -> mot de passe utilisateur -> LDAP
```

pour un nouvel SSO.

---

# 13. Tweens, événements, subscribers et request methods

## 13.1 Tweens

Un **tween** est un middleware placé dans la chaîne interne Pyramid.

Conceptuellement :

```text
request
  -> tween A
      -> tween B
          -> router / view
      <- response
  <- response
```

Exemple :

```python
import time


def timing_tween_factory(handler, registry):
    def timing_tween(request):
        start = time.perf_counter()
        try:
            return handler(request)
        finally:
            duration = time.perf_counter() - start
            request.registry.settings.get("logger")
            print(f"{request.path}: {duration:.3f}s")

    return timing_tween
```

En production, utilisons un vrai logger et une solution d'observabilité.

## 13.2 Usages des tweens

- correlation ID ;
- instrumentation ;
- métriques ;
- gestion globale d'erreurs ;
- transaction ;
- politiques HTTP transversales.

Un tween ne doit pas devenir un endroit où placer toute la logique métier.

## 13.3 Événements

Pyramid publie des événements comme :

- `NewRequest` ;
- `ContextFound` ;
- `BeforeRender` ;
- `NewResponse` ;
- événements d'application.

Subscriber :

```python
from pyramid.events import NewResponse, subscriber


@subscriber(NewResponse)
def add_security_headers(event):
    response = event.response
    response.headers.setdefault("X-Content-Type-Options", "nosniff")
```

## 13.4 Request methods

Nous pouvons enrichir `request` :

```python

def current_tenant(request):
    tenant_id = request.headers.get("X-Tenant-ID")
    return load_tenant(tenant_id)


config.add_request_method(
    current_tenant,
    name="tenant",
    reify=True,
)
```

Puis :

```python
tenant = request.tenant
```

`reify=True` calcule une fois la valeur par requête.

## 13.5 `RequestLocalCache`

Pour des calculs dépendant de la requête et appelés à plusieurs endroits, Pyramid fournit également `RequestLocalCache`.

C'est notamment utile pour éviter de charger plusieurs fois l'identité depuis la base dans une même requête.

---

# 14. Fichiers statiques, i18n, logging et configuration

## 14.1 Logging Python

```python
import logging


log = logging.getLogger(__name__)


def view(request):
    log.info("Consultation", extra={"path": request.path})
```

Ne loggons pas :

- mot de passe ;
- token bearer ;
- cookie de session ;
- secret ;
- données personnelles inutiles.

## 14.2 Logs structurés

En production, un événement utile contient souvent :

- timestamp ;
- niveau ;
- service ;
- request ID ;
- route ;
- status code ;
- durée ;
- identité pseudonymisée si nécessaire.

## 14.3 Internationalisation

Pyramid sait intégrer gettext/Babel.

Principes :

- texte source marqué ;
- catalogue de traduction ;
- locale déterminée explicitement ;
- dates/nombres formatés selon le contexte.

Ne concaténons pas des fragments traduits pour construire une phrase : l'ordre grammatical varie selon les langues.

## 14.4 Configuration par environnement

Une stratégie saine :

```text
configuration non secrète -> fichier/versionné
secrets                  -> environnement/secret manager
configuration dynamique  -> service de configuration si nécessaire
```

Ne transformons pas chaque paramètre en variable d'environnement sans raison : cela peut rendre l'application difficile à reproduire.

---

# 15. Tests

## 15.1 Trois niveaux utiles

### Tests unitaires

Testent une fonction ou classe isolée.

### Tests d'intégration

Testent l'application avec registry, base, security policy, etc.

### Tests fonctionnels

Testent l'application WSGI via HTTP simulé, souvent avec WebTest.

## 15.2 `pytest`

Exemple :

```python

def add(a, b):
    return a + b


def test_add():
    assert add(2, 3) == 5
```

## 15.3 `DummyRequest`

```python
from pyramid.testing import DummyRequest


def test_view():
    request = DummyRequest()
    result = health(request)
    assert result == {"ok": True}
```

## 15.4 Configuration de test

```python
from pyramid import testing


def test_route_url():
    with testing.testConfig() as config:
        config.add_route("home", "/")
        request = testing.DummyRequest()
        request.registry = config.registry
```

Selon le test, il est souvent plus simple d'utiliser les fixtures générées par le cookiecutter officiel.

## 15.5 WebTest

WebTest permet d'exercer l'application WSGI comme un client HTTP :

```python
from webtest import TestApp


def test_health(app):
    testapp = TestApp(app)
    response = testapp.get("/health")
    assert response.status_code == 200
```

## 15.6 Tester la sécurité

Nous devons tester au minimum :

- utilisateur anonyme ;
- utilisateur authentifié sans droit ;
- utilisateur autorisé ;
- CSRF absent ;
- CSRF invalide ;
- session expirée ;
- compte supprimé/désactivé ;
- route non trouvée ;
- entrée invalide.

## 15.7 Base de données de test

Évitons de faire dépendre tous les tests d'une base partagée.

Approches :

- transaction rollback ;
- base temporaire ;
- conteneur de test ;
- fixture dédiée ;
- SQLite lorsque sa sémantique est réellement compatible avec ce que nous testons.

Un test qui passe sous SQLite peut échouer sous PostgreSQL si nous dépendons de comportements spécifiques.

---

# 16. Outils de diagnostic Pyramid

Les scripts `p*` constituent un avantage de l'écosystème Pyramid.

## 16.1 `pserve`

```bash
pserve development.ini --reload
```

## 16.2 `proutes`

Liste les routes :

```bash
proutes development.ini
```

Très utile pour vérifier :

- ordre ;
- pattern ;
- nom ;
- factory.

## 16.3 `pviews`

```bash
pviews development.ini /users/42
```

Aide à comprendre quelle vue est sélectionnée.

## 16.4 `ptweens`

```bash
ptweens development.ini
```

Affiche la chaîne de tweens.

## 16.5 `prequest`

```bash
prequest development.ini /health
```

Permet d'exécuter une requête contre l'application depuis la CLI.

## 16.6 `pshell`

```bash
pshell development.ini
```

Shell Python chargé avec l'environnement de l'application.

C'est particulièrement utile pour :

- inspecter les modèles ;
- tester une requête DB ;
- vérifier les settings ;
- reproduire un bug.

## 16.7 Debug toolbar

Le cookiecutter peut intégrer `pyramid_debugtoolbar` en développement.

Elle permet notamment d'inspecter :

- requête ;
- routes ;
- settings ;
- renderers ;
- exceptions ;
- performances selon plugins.

> [!danger]
> **La debug toolbar ne doit pas être exposée au public en production.** Elle révèle une quantité importante d'informations internes et peut fournir des capacités de débogage dangereuses.

---

# 17. Déploiement WSGI et reverse proxy

## 17.1 `pserve` n'est pas à lui seul un serveur HTTP

`pserve` charge la configuration de l'application et du serveur définie dans le fichier PasteDeploy.

Le cookiecutter Pyramid utilise généralement **Waitress** par défaut.

Exemple :

```ini
[server:main]
use = egg:waitress#main
listen = 127.0.0.1:6543
```

Puis :

```bash
pserve production.ini
```

## 17.2 Waitress

Waitress :

- est WSGI ;
- fonctionne sur plusieurs plateformes ;
- constitue un serveur robuste et simple ;
- est utilisé par défaut dans les projets Pyramid générés.

Pour certains déploiements, Gunicorn ou mod_wsgi peuvent être choisis à la place.

## 17.3 Architecture reverse proxy

```text
Internet
   |
   v
Nginx / HAProxy / ingress
   |
   +-- TLS
   +-- limites de taille
   +-- timeouts
   +-- headers proxy nettoyés
   |
   v
Waitress / Gunicorn
   |
   v
Pyramid WSGI
```

## 17.4 Headers proxy

Une application derrière proxy doit connaître correctement :

- schéma `https` ;
- host public ;
- port ;
- IP client lorsque nécessaire.

Mais accepter naïvement `X-Forwarded-For` depuis Internet permet à un client de falsifier son IP.

Le reverse proxy doit :

1. supprimer les headers entrants non fiables ;
2. écrire les siens ;
3. être le seul client autorisé du serveur WSGI lorsque possible.

## 17.5 Timeouts

Configurons explicitement :

- timeout client ;
- timeout reverse proxy ;
- timeout serveur WSGI ;
- timeout DB ;
- timeout LDAP ;
- timeout API distante.

Sans timeout, une dépendance lente peut épuiser les workers.

## 17.6 WSGI et traitements longs

Une requête HTTP ne devrait pas lancer un traitement de plusieurs minutes dans le worker web.

Préférons :

```text
POST /exports
    |
    v
validation + création job
    |
    v
file de tâches
    |
    v
worker
```

Puis :

```text
GET /exports/<id>
```

## 17.7 WSGI et async

Une fonction `async def` n'est pas automatiquement rendue concurrente par Pyramid WSGI.

Si le besoin principal est :

- WebSocket massif ;
- streaming bidirectionnel ;
- très grand nombre de connexions en attente ;
- écosystème ASGI natif ;

il faut évaluer une architecture ASGI ou un service spécialisé.

---

# 18. Docker et exploitation

Voir [[Docker]].

## 18.1 Dockerfile moderne minimal

```dockerfile
FROM python:3.14-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

RUN useradd --create-home --uid 10001 appuser

COPY pyproject.toml ./
COPY README* ./
COPY mon_projet ./mon_projet

RUN python -m pip install --no-cache-dir .

USER appuser

EXPOSE 6543

CMD ["pserve", "production.ini"]
```

Ce Dockerfile suppose que `production.ini` est présent dans l'image ou monté de manière appropriée.

## 18.2 Ne pas lancer le serveur de debug

Évitons en production :

```bash
pserve development.ini --reload
```

Le reloader et les outils de debug sont destinés au développement.

## 18.3 Health checks

Route :

```python
@view_config(route_name="health", renderer="json")
def health(request):
    return {"status": "ok"}
```

Distinguons :

### Liveness

> Le processus fonctionne-t-il ?

### Readiness

> Peut-il réellement servir une requête ?

La readiness peut vérifier prudemment :

- connexion DB ;
- migrations compatibles ;
- dépendance critique.

Évitons qu'un health check provoque lui-même une charge importante.

## 18.4 Utilisateur non root

L'application doit généralement s'exécuter sous un utilisateur sans privilèges.

Dans Docker :

```dockerfile
USER appuser
```

Sur systemd :

```ini
[Service]
User=pyramid
Group=pyramid
```

Voir [[Initialisation système et des services]].

---

# 19. Performances et cache

## 19.1 Mesurer avant d'optimiser

Collectons :

- p50/p95/p99 ;
- temps SQL ;
- nombre de requêtes DB ;
- taux d'erreur ;
- CPU ;
- mémoire ;
- saturation workers ;
- hit ratio du cache.

## 19.2 Cache HTTP

Pour une ressource publique :

```python
response = request.response
response.cache_control.public = True
response.cache_control.max_age = 300
```

Un bon cache HTTP peut éliminer plus de charge qu'un cache applicatif complexe.

Voir [[HTTP]].

## 19.3 ETag et requêtes conditionnelles

Pour des ressources versionnées, utiliser :

- ETag ;
- `If-None-Match` ;
- `If-Match` pour la concurrence optimiste.

Ne faisons pas seulement du cache ; utilisons aussi les préconditions HTTP pour prévenir les écrasements concurrents.

## 19.4 Cache applicatif

Un cache Redis ou mémoire peut stocker :

- calcul coûteux ;
- mapping de configuration ;
- résultat d'API ;
- profil public.

Mais il faut définir :

- clé ;
- TTL ;
- invalidation ;
- comportement en panne ;
- cohérence ;
- confidentialité.

## 19.5 Cache de l'identité

`request.identity` peut être calculé plusieurs fois pendant une requête. Utiliser un cache local à la requête évite de refaire une requête SQL/LDAP.

Ne transformons pas ce cache local en cache global de longue durée sans réfléchir au délai de révocation des droits.

---

# 20. Durcissement de sécurité

## 20.1 HTTPS

Une application d'authentification doit être exposée en HTTPS.

Cela protège notamment :

- mot de passe ;
- cookie ;
- token ;
- contenu sensible.

## 20.2 Headers recommandés

Selon l'application :

```text
Content-Security-Policy
X-Content-Type-Options: nosniff
Referrer-Policy
Permissions-Policy
Strict-Transport-Security
```

Évitons de copier une CSP générique sans comprendre les ressources réellement chargées.

## 20.3 CSP

Exemple de point de départ strict :

```text
Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```

Une vraie application avec scripts/styles externes devra adapter cette politique.

## 20.4 CSRF par défaut

Pour une application HTML traditionnelle avec cookies, une politique saine est :

```python
config.set_default_csrf_options(require_csrf=True)
```

puis exemptions explicites uniquement pour des endpoints dont le modèle d'authentification le justifie.

## 20.5 Uploads

Pour les fichiers :

- limite de taille ;
- nom généré côté serveur ;
- stockage hors répertoire exécutable ;
- vérification MIME réelle ;
- antivirus si le risque l'exige ;
- permissions minimales ;
- jamais de chemin construit depuis le nom utilisateur sans normalisation.

## 20.6 Injection SQL

Avec SQLAlchemy :

```python
session.execute(select(User).where(User.email == email))
```

Évitons :

```python
# Mauvais exemple
sql = f"SELECT * FROM users WHERE email = '{email}'"
```

## 20.7 Injection LDAP

Un identifiant utilisateur ne doit pas être inséré brut dans :

```text
(uid=<entrée utilisateur>)
```

Utilisons les fonctions d'échappement LDAP de la bibliothèque choisie.

## 20.8 Secret de session

Le secret doit :

- être aléatoire ;
- être suffisamment long ;
- être différent entre environnements ;
- ne pas être commité ;
- pouvoir être tourné selon une procédure définie.

## 20.9 Open redirect

Après login, ce code est dangereux :

```python
return HTTPFound(location=request.params["next"])
```

Un attaquant peut fabriquer :

```text
/login?next=https://evil.example
```

Il faut valider que la destination appartient à une liste sûre ou est un chemin interne acceptable.

## 20.10 Dépendances

Maintenons :

- Pyramid ;
- WebOb ;
- Waitress/Gunicorn ;
- template engine ;
- SQLAlchemy ;
- drivers DB ;
- Deform/Colander ;
- libraries LDAP/OIDC.

Les vulnérabilités se trouvent souvent dans l'écosystème autour du framework, pas uniquement dans Pyramid.

---

# 21. Migration d'un ancien projet Pyramid 1.x

## 21.1 Inventaire

Cherchons :

```bash
grep -R "AuthTktAuthenticationPolicy" -n .
grep -R "SessionAuthenticationPolicy" -n .
grep -R "ACLAuthorizationPolicy" -n .
grep -R "set_authentication_policy" -n .
grep -R "set_authorization_policy" -n .
grep -R "effective_principals" -n .
grep -R "unauthenticated_userid" -n .
grep -R "pcreate" -n .
```

## 21.2 Remplacer les anciennes policies

Ancien :

```python
config.set_authentication_policy(authn_policy)
config.set_authorization_policy(authz_policy)
```

Nouveau :

```python
config.set_security_policy(SecurityPolicy())
```

## 21.3 Remplacer les APIs de requête dépréciées

Ancien code :

```python
request.unauthenticated_userid
request.effective_principals
```

Nouveau design :

```python
request.identity
request.authenticated_userid
request.is_authenticated
```

Puis la Security Policy calcule elle-même les informations nécessaires à l'autorisation.

## 21.4 Sessions Pickle

Pyramid 2.0 a changé le serializer par défaut des sessions signées vers JSON et a déprécié `PickleSerializer`.

Un projet historique doit vérifier la compatibilité de ses anciennes sessions lors de la migration.

Le plus simple peut être d'accepter une reconnexion des utilisateurs plutôt que de conserver indéfiniment un format de session risqué.

## 21.5 `pcreate`

Remplacer les procédures de création :

```bash
pcreate ...
```

par :

```bash
cookiecutter gh:Pylons/pyramid-cookiecutter-starter --checkout 2.1-branch
```

## 21.6 SQLAlchemy 2.x

Pour un projet SQL :

- mettre à jour les modèles ;
- utiliser les APIs SQLAlchemy 2.x ;
- revoir le lifecycle des sessions ;
- mettre à jour Alembic ;
- tester les transactions avec `pyramid_tm`/`zope.sqlalchemy`.

## 21.7 Python

Pyramid 2.1 ne supporte plus Python 3.6 à 3.9.

Avant migration :

```bash
python --version
```

Puis :

- choisir Python 3.12, 3.13 ou 3.14 selon l'écosystème du projet ;
- recréer le venv ;
- reconstruire le lock/fichier de dépendances ;
- exécuter toute la suite de tests.

## 21.8 Migration progressive

Ordre conseillé :

1. obtenir une suite de tests fiable ;
2. migrer Python ;
3. migrer les dépendances ;
4. migrer Pyramid ;
5. migrer la sécurité ;
6. migrer SQLAlchemy/ZODB ;
7. corriger les dépréciations ;
8. renforcer CSRF/cookies/headers ;
9. tester en environnement de préproduction ;
10. observer après déploiement.

---

# 22. Architecture d'un projet de production

Prenons une application « membres » avec :

- Pyramid ;
- PostgreSQL ;
- SQLAlchemy ;
- Deform ;
- OIDC principal ;
- LDAP comme source de données d'annuaire ;
- Waitress ;
- Nginx ;
- Docker ou systemd.

## 22.1 Structure

```text
members/
├── pyproject.toml
├── development.ini
├── production.ini
├── members/
│   ├── __init__.py
│   ├── routes.py
│   ├── security.py
│   ├── settings.py
│   ├── db.py
│   ├── ldap.py
│   ├── oidc.py
│   ├── services/
│   │   ├── members.py
│   │   └── mail.py
│   ├── models/
│   ├── schemas/
│   ├── views/
│   ├── templates/
│   └── static/
└── tests/
```

## 22.2 Factory

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    with Configurator(settings=settings) as config:
        config.include(".db")
        config.include(".security")
        config.include(".routes")
        config.include(".monitoring")
        config.scan(".views")
        return config.make_wsgi_app()
```

## 22.3 Routes

```python
def includeme(config):
    config.add_route("home", "/")
    config.add_route("login", "/login")
    config.add_route("logout", "/logout")
    config.add_route("members", "/members")
    config.add_route("member", "/members/{member_id}")
    config.add_route("health", "/health")
```

## 22.4 Couche service

Une vue ne doit pas contenir toute la logique métier :

```python
class MemberService:
    def __init__(self, dbsession):
        self.dbsession = dbsession

    def create_member(self, command):
        self._validate(command)
        member = Member.from_command(command)
        self.dbsession.add(member)
        return member
```

Vue :

```python
@view_config(
    route_name="members",
    request_method="POST",
    permission="member:create",
    renderer="json",
)
def create_member(request):
    service = MemberService(request.dbsession)
    member = service.create_member(request.json_body)
    request.response.status = 201
    return member.to_dict()
```

## 22.5 Pourquoi cette séparation ?

Elle facilite :

- tests unitaires ;
- réutilisation depuis CLI/worker ;
- changements d'interface ;
- audit sécurité ;
- lecture du code.

## 22.6 Flux de sécurité

```text
requête
  |
  v
reverse proxy
  |
  v
Pyramid
  |
  +--> CSRF si cookie + mutation
  |
  +--> SecurityPolicy.identity
  |
  +--> permission
  |
  +--> validation payload
  |
  +--> service métier
  |
  +--> transaction DB
  |
  v
response
```

## 22.7 Mail

Pour des emails transactionnels, l'application peut utiliser une extension comme `pyramid_mailer` ou un service de mail encapsulé derrière notre propre interface.

La bonne architecture est :

```python
class MailService:
    def send_password_reset(self, recipient, link):
        ...
```

Ainsi le code métier ne dépend pas directement d'un package de transport SMTP.

Pour les emails non immédiats, une file de tâches est généralement préférable.

---

# 23. Travaux pratiques

## TP 1 — Première application

Objectifs :

- créer un venv ;
- installer Pyramid 2.1 ;
- créer deux routes ;
- retourner texte et JSON ;
- tester avec `curl`.

À produire :

```text
GET /        -> HTML ou texte
GET /health  -> JSON
```

## TP 2 — Cookiecutter

Créer un projet avec :

- template Chameleon ;
- backend `none`.

Puis :

```bash
pytest
pserve development.ini --reload
proutes development.ini
```

Expliquer le rôle de :

- `pyproject.toml` ;
- `development.ini` ;
- `__init__.py` ;
- `routes.py` ;
- `views/`.

## TP 3 — Routage avancé

Créer :

```text
GET    /articles
POST   /articles
GET    /articles/{id}
PUT    /articles/{id}
DELETE /articles/{id}
```

Ajouter :

- validation de `id` ;
- 404 ;
- 400 ;
- génération de route avec `route_url`.

## TP 4 — Sessions et CSRF

Construire un panier simple :

- nombre d'articles en session ;
- message flash ;
- formulaire POST ;
- CSRF global ;
- cookie `Secure`/`HttpOnly` en configuration production.

Expliquer pourquoi une session signée ne doit pas contenir de secret.

## TP 5 — Security Policy

Implémenter :

- identité `User` ;
- session authentication helper ;
- login ;
- logout ;
- permission `edit` ;
- groupe `editors`.

Tester :

- anonyme ;
- viewer ;
- editor.

Interdiction : utiliser `SessionAuthenticationPolicy` ou `ACLAuthorizationPolicy` legacy.

## TP 6 — Deform

Créer un formulaire :

- nom ;
- email ;
- date de naissance facultative ;
- validation ;
- messages d'erreur ;
- CSRF.

Voir [[Deform]].

## TP 7 — SQLAlchemy

Générer le cookiecutter avec backend `sqlalchemy`.

Créer :

```text
User
Article
```

Ajouter :

- migration Alembic ;
- contrainte unique ;
- tests DB ;
- transaction automatique.

## TP 8 — LDAP

Dans un environnement de laboratoire :

- créer un serveur LDAP de test ;
- connexion TLS validée ;
- recherche d'utilisateur ;
- mapping d'un groupe LDAP vers un rôle applicatif ;
- aucun mot de passe stocké en session.

Comparer avec une architecture OIDC + IdP connecté à LDAP.

## TP 9 — Traversal

Créer un arbre :

```text
/projects/<project>/documents/<document>
```

Ajouter ACL héritées :

- lecteur projet ;
- éditeur projet ;
- document privé.

Comparer avec URL dispatch.

## TP 10 — Tests et debug

Écrire :

- 5 tests unitaires ;
- 5 tests d'intégration ;
- 5 tests WebTest.

Utiliser :

```bash
proutes
pviews
ptweens
prequest
pshell
```

Documenter ce que chaque commande a permis de diagnostiquer.

## TP 11 — Production

Déployer :

```text
Nginx -> Waitress -> Pyramid -> PostgreSQL
```

Ajouter :

- HTTPS ;
- user non root ;
- health check ;
- logs ;
- timeout DB ;
- timeout LDAP ;
- secret externe ;
- debug toolbar désactivée.

## TP 12 — Audit d'un ancien projet

Prendre un projet Pyramid 1.x fictif contenant :

```text
AuthTktAuthenticationPolicy
ACLAuthorizationPolicy
pcreate
PickleSerializer
Python 3.8
```

Produire :

1. inventaire ;
2. analyse de risque ;
3. ordre de migration ;
4. patch vers Security Policy ;
5. plan de tests ;
6. procédure de rollback.

---

# 24. Checklist finale

## Architecture

- [ ] La logique métier n'est pas concentrée dans les vues.
- [ ] Les modules sont ajoutés avec `config.include()` lorsque pertinent.
- [ ] Les routes ont des noms stables.
- [ ] Les URLs sont générées avec `route_url`/`route_path`.
- [ ] URL dispatch et traversal sont choisis consciemment.

## Python et Pyramid

- [ ] Python >= 3.10.
- [ ] Pyramid stable, actuellement 2.1 dans ce cours.
- [ ] Aucun `pcreate` dans la documentation moderne.
- [ ] `pyproject.toml` est utilisé pour le packaging du projet.
- [ ] Les dépendances sont reproductibles.

## Sécurité

- [ ] Security Policy moderne.
- [ ] Pas de nouvelle implémentation avec `SessionAuthenticationPolicy` legacy.
- [ ] `request.identity` utilisé lorsque pertinent.
- [ ] CSRF activé pour les mutations authentifiées par cookie.
- [ ] Cookies `Secure` et `HttpOnly` selon le besoin.
- [ ] HTTPS obligatoire pour l'authentification.
- [ ] Password hashing robuste.
- [ ] Aucun secret dans Git.
- [ ] Aucun certificat LDAP ignoré.
- [ ] Aucun open redirect.
- [ ] Debug toolbar désactivée en production.

## Données

- [ ] Validation syntaxique et métier.
- [ ] Requêtes paramétrées/ORM.
- [ ] Transactions maîtrisées.
- [ ] Migrations de schéma testées.
- [ ] Timeouts sur les services externes.

## Tests

- [ ] Unitaires.
- [ ] Intégration.
- [ ] Fonctionnels.
- [ ] Permissions négatives testées.
- [ ] CSRF testé.
- [ ] erreurs 400/401/403/404/500 testées.

## Production

- [ ] Serveur WSGI de production.
- [ ] Reverse proxy configuré.
- [ ] Headers proxy de confiance maîtrisés.
- [ ] Logs sans secrets.
- [ ] Health checks.
- [ ] Observabilité.
- [ ] Sauvegardes.
- [ ] Procédure de rollback.
- [ ] Mise à jour des dépendances organisée.

---

# 25. Références

## Documentation Pyramid

- Documentation stable : https://docs.pylonsproject.org/projects/pyramid/en/latest/
- What's New in Pyramid 2.1 : https://docs.pylonsproject.org/projects/pyramid/en/latest/whatsnew-2.1.html
- Change history : https://docs.pylonsproject.org/projects/pyramid/en/latest/changes.html
- Quick tutorial : https://docs.pylonsproject.org/projects/pyramid/en/latest/quick_tutorial/
- Security : https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/security.html
- Sessions : https://docs.pylonsproject.org/projects/pyramid/en/latest/api/session.html
- CSRF : https://docs.pylonsproject.org/projects/pyramid/en/latest/api/csrf.html
- Configuration : https://docs.pylonsproject.org/projects/pyramid/en/latest/api/config.html

## Projets associés

- Pylons Project : https://pylonsproject.org/
- Pyramid sur PyPI : https://pypi.org/project/pyramid/
- Cookiecutter Pyramid : https://github.com/Pylons/pyramid-cookiecutter-starter
- WebTest : https://docs.pylonsproject.org/projects/webtest/
- Deform : voir [[Deform]]
- LDAP : voir [[LDAP]]
- OAuth/OpenID Connect : voir [[OAuth OpenID]]
- SQLAlchemy : voir [[SQLAchemy]]

---

# Conclusion

Pyramid reste en 2026 un framework Python mature et particulièrement flexible. Son intérêt n'est pas de fournir le plus grand nombre de fonctionnalités intégrées, mais de donner une **architecture explicite et composable**.

Les points essentiels à retenir sont :

1. Pyramid 2.1 est un framework **WSGI** pour Python >= 3.10.
2. Le cœur du framework est le `Configurator` et son système de composition.
3. Pyramid propose à la fois **URL dispatch** et **traversal**.
4. Les anciennes Authentication/Authorization Policies de Pyramid 1.x sont dépréciées : un nouveau code utilise une **Security Policy**.
5. `request.identity` est l'abstraction moderne représentant l'utilisateur courant.
6. Les sessions, CSRF, ACL et renderers sont configurables et non imposés.
7. SQLAlchemy et ZODB restent deux intégrations importantes, avec des architectures différentes.
8. Pour un SSO moderne, OpenID Connect est généralement préférable à la collecte directe d'un mot de passe LDAP par chaque application.
9. `pserve`, `proutes`, `pviews`, `ptweens`, `prequest` et `pshell` rendent Pyramid particulièrement observable en développement.
10. En production, Pyramid doit être considéré comme une application WSGI à exploiter avec les mêmes exigences que tout service critique : TLS, secrets, timeouts, logs, tests, supervision et mises à jour.
