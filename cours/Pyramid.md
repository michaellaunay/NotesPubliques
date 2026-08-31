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
resume: "Cours approfondi sur Pyramid 2.1 : architecture WSGI et cycle de requête, configuration, routes, vues, traversal, sécurité moderne, sessions/CSRF, Deform, SQLAlchemy/ZODB, LDAP/OIDC, tests, migration et déploiement."
niveau: avance
prerequis:
  - "[[Python]]"
  - "[[HTTP]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-06-14
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---

> [!tip] Version longue
> Ce cours existe aussi sous forme de livre complet : [[Pyramid — livre complet]].

# Pyramid

> [!abstract] Objectif
> Construire une application Web avec Pyramid 2.1 en comprenant ce que fait le framework : WSGI, configuration déclarative, routes et traversal, vues et rendus, sécurité moderne (security policy, permissions, CSRF), sessions, formulaires avec Deform, persistance SQLAlchemy ou ZODB, authentification LDAP et OIDC, tests et déploiement.

Voir aussi : [[Python]], [[Deform]], [[SQLAchemy]], [[ZC.Buildout]], [[OAuth OpenID]], [[LDAP]], [[Apache]].

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


## 1.5 Fil conducteur : une application de gestion de membres

Pour éviter d'apprendre Pyramid comme une succession d'API isolées, nous utiliserons un **fil conducteur**. Imaginons une application interne de gestion de membres. Elle doit permettre à un utilisateur authentifié de consulter une fiche, à un responsable de la modifier, et à un administrateur de créer ou désactiver des comptes. Les informations applicatives vivent dans PostgreSQL ; certaines informations d'identité proviennent d'un annuaire LDAP ; l'authentification web peut être assurée directement par l'application dans un environnement simple, mais dans une organisation moderne elle sera souvent déléguée à un fournisseur OpenID Connect connecté lui-même à l'annuaire.

Ce scénario est intéressant car il force à rencontrer presque toutes les briques importantes de Pyramid sans inventer des exemples artificiels : routes, vues, formulaires, permissions, sessions, transactions, services externes, tests et déploiement. Il montre aussi pourquoi Pyramid est souvent apprécié dans des applications métier anciennes ou complexes : il n'impose pas une seule architecture de données ni un seul mécanisme d'authentification.

Au début, l'application peut être minuscule :

```python
from pyramid.config import Configurator
from pyramid.response import Response


def home(request):
    return Response("Gestion des membres")


def main(global_config, **settings):
    with Configurator(settings=settings) as config:
        config.add_route("home", "/")
        config.add_view(home, route_name="home")
        return config.make_wsgi_app()
```

Ce code suffit pour comprendre une idée essentielle : la factory `main()` **compose** une application WSGI. Une fois `make_wsgi_app()` appelée, le résultat est un objet callable que le serveur WSGI peut invoquer pour chaque requête HTTP. Nous pouvons donc lire Pyramid en deux temps : d'abord une phase de configuration, puis une phase d'exécution.

Lorsque le projet grandit, nous ne voulons plus tout placer dans `__init__.py`. Nous répartissons les responsabilités :

```text
members/
├── __init__.py        # application factory
├── routes.py          # noms et patterns d'URL
├── security.py        # identité et permissions
├── models/            # persistance
├── services/          # règles métier
├── views/             # adaptation HTTP -> métier
├── templates/         # HTML
└── tests/
```

Cette séparation n'est pas une obligation du framework. C'est une décision d'architecture. Pyramid se contente de fournir des mécanismes de composition, notamment `config.include()`, les registries, les décorateurs de vue et les interfaces. Cette liberté est puissante, mais elle signifie aussi qu'une équipe doit documenter ses conventions.

Le premier principe du cours sera donc : **une vue Pyramid doit rester une frontière HTTP, pas devenir toute l'application**. Une vue reçoit une requête, récupère les données nécessaires, appelle un service métier, puis construit une réponse. Les règles de création d'un membre, le choix du rôle initial ou la vérification d'une contrainte métier doivent vivre ailleurs afin de rester testables sans serveur HTTP.

Nous allons progressivement faire évoluer le flux suivant :

```text
navigateur
   |
   v
route Pyramid
   |
   v
vue
   |
   +--> identité / permission
   +--> validation du formulaire
   +--> service métier
   +--> transaction SQLAlchemy
   +--> LDAP ou OIDC si nécessaire
   |
   v
renderer HTML ou JSON
```

À chaque chapitre, nous ajouterons une pièce sans perdre de vue ce flux global. C'est une meilleure manière d'apprendre Pyramid que de mémoriser indépendamment `add_route`, `view_config`, `request.session` ou `ACLHelper`.

---

## 1.6 Quand Pyramid est-il encore un bon choix en 2026 ?

Pyramid n'est pas le framework Python le plus visible dans les tutoriels récents, mais cette visibilité ne doit pas être confondue avec sa pertinence technique. Il reste particulièrement intéressant lorsqu'une application a besoin d'un **framework stable, WSGI, très composable et peu prescriptif**. Les projets qui combinent HTML, API, SQLAlchemy, ZODB, LDAP, ACL ou extensions historiques du Pylons Project peuvent profiter de cette continuité.

À l'inverse, si le besoin principal est une petite API asynchrone native avec WebSocket et un écosystème centré ASGI, un framework ASGI peut être plus direct. Si l'équipe veut une plateforme intégrée fournissant immédiatement ORM, administration, formulaires et conventions fortes, Django réduit davantage le nombre de décisions. Pyramid occupe un autre espace : il fournit les mécanismes de composition sans imposer tout le reste.

Cette caractéristique explique aussi pourquoi le framework demande davantage de compréhension au début. Deux projets Pyramid peuvent choisir des persistances, renderers et modèles de sécurité différents. La documentation de projet et les conventions d'équipe sont donc plus importantes que dans un framework très prescriptif.

La stabilité de Pyramid est un avantage pour les applications métier dont la durée de vie se mesure en années. Pyramid 2.1, publié en mars 2026, modernise notamment le support Python jusqu'à 3.14 tout en conservant les concepts centraux du framework. Cela permet de moderniser progressivement un projet ancien au lieu de le réécrire uniquement pour suivre une mode de framework.

Le critère de choix doit finalement rester le coût total : compétences de l'équipe, contraintes d'exploitation, dépendances existantes, modèle de données, besoin async et durée de maintenance. Pyramid est excellent lorsqu'on valorise la **composition explicite et la compatibilité à long terme**.

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


## 3.8 Du démarrage du processus à l'appel d'une vue

Un point souvent survolé dans les cours web est la différence entre ce qui se passe **une fois au démarrage** et ce qui se passe **à chaque requête**. Cette distinction est fondamentale avec Pyramid.

Au démarrage, le serveur charge l'application à partir de la configuration PasteDeploy. La factory reçoit `global_config` et les paramètres de `[app:main]`, construit un `Configurator`, inclut les composants, scanne les décorateurs puis crée l'application WSGI. Ce travail ne doit pas être répété pour chaque requête.

```text
processus démarre
      |
      v
lecture production.ini
      |
      v
main(global_config, **settings)
      |
      +--> config.include(...)
      +--> config.add_route(...)
      +--> config.scan(...)
      |
      v
make_wsgi_app()
```

Ensuite, pour chaque requête, le serveur appelle l'application WSGI. Pyramid crée un objet `Request`, exécute la chaîne des tweens, trouve la route et la vue, vérifie les prédicats et permissions, appelle la vue, transforme éventuellement sa valeur de retour via un renderer, puis renvoie une réponse WSGI.

```text
HTTP request
    |
    v
Request WebOb
    |
    v
tweens
    |
    v
route / traversal
    |
    v
view lookup
    |
    +--> security
    +--> view predicates
    |
    v
view callable
    |
    v
renderer / Response
    |
    v
HTTP response
```

Cette lecture explique plusieurs comportements qui semblent magiques au début. Le décorateur `@view_config` n'enregistre pas immédiatement la vue au moment où nous écrivons le code ; il attache des métadonnées, puis `config.scan()` les collecte pendant la configuration. De même, `config.include(".routes")` n'est pas un simple import décoratif : il permet à un sous-module de participer explicitement à la composition de l'application via une fonction `includeme(config)`.

Prenons une application découpée proprement :

```python
# routes.py

def includeme(config):
    config.add_route("members", "/members")
    config.add_route("member", "/members/{member_id}")
```

```python
# __init__.py
from pyramid.config import Configurator


def main(global_config, **settings):
    with Configurator(settings=settings) as config:
        config.include(".routes")
        config.include(".security")
        config.include(".db")
        config.scan(".views")
        return config.make_wsgi_app()
```

La valeur d'un tel découpage apparaît lorsque le projet comporte plusieurs dizaines de routes. Chaque module peut exposer son `includeme()` et l'application factory devient une table des matières de l'architecture.

Il faut aussi comprendre la **registry** Pyramid. Elle contient les composants configurés pour l'application. Plutôt que d'importer un singleton global partout, nous pouvons enregistrer des utilities, request methods ou services dans la registry. Ce mécanisme favorise les extensions et les tests, mais il ne doit pas servir de prétexte à cacher toutes les dépendances. Dans du code métier, un constructeur explicite reste souvent plus lisible qu'une recherche opaque dans la registry.

Un autre point important est le conflit de configuration. Pyramid accumule des actions puis les résout. Si deux morceaux de configuration prétendent enregistrer de manière incompatible le même élément, le framework peut produire une erreur de configuration au démarrage plutôt que de laisser une ambiguïté silencieuse en production. Cette philosophie explique la séparation historique entre **configuration time** et **runtime**.

Enfin, cette architecture permet l'introspection. Les commandes `proutes`, `pviews` ou `ptweens` peuvent décrire ce que Pyramid a réellement construit. Dans une application complexe, cela est souvent plus fiable que de lire uniquement le code source : plusieurs packages peuvent contribuer à la configuration via `include()` et des scans.

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


## 4.8 Lire correctement une requête HTTP

L'objet `request` est pratique au point qu'on peut oublier qu'il représente une requête HTTP concrète. Avant d'utiliser `request.params` partout, il faut choisir la source adaptée à la sémantique de l'opération.

Une query string sert bien aux filtres et paramètres de navigation :

```text
GET /members?status=active&page=2
```

Nous lisons alors :

```python
status = request.GET.get("status")
page = int(request.GET.get("page", "1"))
```

Un formulaire HTML `application/x-www-form-urlencoded` ou `multipart/form-data` se trouve dans `request.POST`. Pour une API JSON, utilisons `request.json_body`, en traitant explicitement les erreurs de contenu et de schéma. `request.params` fusionne plusieurs sources et peut être commode pour de petits usages, mais cette fusion peut rendre ambiguë l'origine d'une valeur dans du code sensible.

Les headers ont également une sémantique propre. `Accept` exprime ce que le client souhaite recevoir ; `Content-Type` décrit le corps envoyé. Confondre les deux conduit à des réponses `415 Unsupported Media Type` ou à des négociations de contenu incohérentes.

Le corps HTTP n'est pas toujours petit. Pour un upload ou une API exposée, la taille maximale doit être limitée au niveau du reverse proxy et/ou de l'application. Charger aveuglément plusieurs centaines de mégaoctets en mémoire dans un worker WSGI est un moyen simple de provoquer un déni de service.

### Réponses et codes de statut

Un bon handler ne renvoie pas `200 OK` pour tout. Quelques repères :

```text
200 OK             lecture ou mutation avec corps de réponse
201 Created        ressource créée
204 No Content     succès sans corps
302/303            redirection navigateur selon le flux
400 Bad Request    requête syntaxiquement ou structurellement invalide
401 Unauthorized   authentification requise/échouée selon le mécanisme
403 Forbidden      identité connue mais action interdite
404 Not Found      ressource absente ou volontairement masquée
409 Conflict       conflit avec l'état courant
422                validation sémantique selon convention d'API
```

Pour notre création de membre, une API JSON peut faire :

```python
@view_config(route_name="members", request_method="POST", renderer="json")
def create_member(request):
    command = parse_member_command(request.json_body)
    member = request.member_service.create(command)
    request.response.status_int = 201
    request.response.location = request.route_url(
        "member", member_id=member.id
    )
    return member.to_dict()
```

Le header `Location` rend le résultat plus explicite. Pour un formulaire HTML, le pattern **POST/Redirect/GET** évite qu'un rafraîchissement renvoie involontairement le formulaire :

```text
POST /members/new
   -> validation + commit
   -> 303 /members/42
GET /members/42
```

Les exceptions HTTP Pyramid permettent de coder ce flux de manière lisible. Leur nom peut surprendre les débutants : lever `HTTPFound` ou `HTTPNotFound` n'indique pas une erreur Python imprévue, mais un mécanisme de contrôle HTTP pris en charge par le framework.

### Données externes et confiance

Tout ce qui vient de `request` doit être considéré comme non fiable : path parameters, query string, JSON, cookies, headers et fichiers. Même un header ajouté normalement par un reverse proxy peut être falsifié si l'application est accessible directement. La validation doit donc être placée à la bonne frontière et ne jamais reposer uniquement sur l'interface navigateur.

Cette discipline rend la suite du cours plus simple : le routage identifie l'opération, la couche d'entrée convertit les données HTTP en types Python valides, puis le domaine travaille avec des objets déjà normalisés.

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


## 5.10 Lire le routage comme un contrat HTTP

Une route n'est pas seulement un moyen de faire correspondre une chaîne à une fonction. Elle représente une partie du **contrat HTTP** de l'application. Pour notre application de membres, nous pouvons commencer par :

```python
def includeme(config):
    config.add_route("members", "/members")
    config.add_route("member", "/members/{member_id}")
```

Puis distinguer les opérations à l'aide des prédicats de vue :

```python
from pyramid.view import view_config


@view_config(route_name="members", request_method="GET", renderer="json")
def list_members(request):
    ...


@view_config(route_name="members", request_method="POST", renderer="json")
def create_member(request):
    ...
```

Le même pattern d'URL peut donc participer à plusieurs opérations selon la méthode HTTP, les headers, le contexte ou d'autres prédicats. Il est généralement préférable de rendre ce contrat explicite plutôt que d'écrire une énorme vue qui inspecte manuellement `request.method`.

Les paramètres capturés par le pattern se retrouvent dans `request.matchdict`. Ils restent cependant des **chaînes**. Si `{member_id}` doit être un entier, le code doit le convertir et traiter proprement l'échec :

```python
from pyramid.httpexceptions import HTTPNotFound


def get_member_id(request):
    try:
        return int(request.matchdict["member_id"])
    except (KeyError, ValueError):
        raise HTTPNotFound()
```

Cette conversion peut être centralisée dans une couche d'adaptation ou un prédicat, mais il faut éviter de laisser une valeur externe non validée se propager jusqu'à l'ORM.

La génération inverse d'URL est tout aussi importante que le matching :

```python
url = request.route_url("member", member_id=42)
```

Écrire à la main `f"/members/{member.id}"` disperse les chemins dans l'application et rend les renommages fragiles. Le **nom de route** devient une API interne stable ; le pattern peut évoluer sans modifier tous les templates et services.

Pyramid sépare aussi clairement la **vue** du **renderer**. Une vue peut renvoyer un dictionnaire :

```python
@view_config(route_name="member", renderer="members/member.pt")
def member_view(request):
    member = request.services.members.get(request.matchdict["member_id"])
    if member is None:
        raise HTTPNotFound()
    return {"member": member}
```

Le renderer transforme ensuite ce dictionnaire en réponse HTML. Pour une API JSON, le même principe devient :

```python
@view_config(route_name="member_api", renderer="json")
def member_api(request):
    return {"id": member.id, "name": member.display_name}
```

Cette séparation simplifie les tests. Nous pouvons appeler une fonction de vue avec un `DummyRequest` et vérifier son dictionnaire sans parser du HTML. Les tests du template, eux, peuvent se concentrer sur le rendu.

### Chameleon, TAL et METAL

L'ancien cours Pyramid utilisait beaucoup Chameleon. Ce choix reste pertinent lorsqu'un projet exploite déjà TAL/METAL ou l'écosystème Zope/Pylons. TAL permet d'exprimer les transformations directement dans un document HTML structurellement valide, tandis que METAL permet de créer des macros et layouts réutilisables.

Exemple simple :

```html
<h1 tal:content="member.display_name">Nom</h1>
<a tal:attributes="href request.route_url('members')">Retour</a>
```

Un layout peut exposer des slots avec METAL afin d'éviter de recopier le squelette HTML. Jinja2 reste une alternative tout aussi légitime ; le choix dépend surtout de l'existant, de l'expérience de l'équipe et du besoin d'interopérer avec d'autres projets.

Le point de sécurité essentiel est l'échappement. Un moteur de templates correctement utilisé protège normalement les valeurs textuelles ordinaires contre l'injection HTML, mais l'introduction volontaire de contenu marqué « safe » contourne cette protection. Toute donnée utilisateur réinjectée comme HTML doit être considérée comme hostile tant qu'elle n'a pas été produite ou assainie selon une politique explicite.

Enfin, toutes les réponses ne nécessitent pas de renderer. Une redirection ou un téléchargement peut être exprimé directement :

```python
from pyramid.httpexceptions import HTTPFound


def after_create(request, member):
    raise HTTPFound(request.route_url("member", member_id=member.id))
```

Une bonne vue Pyramid choisit donc l'abstraction adaptée : dictionnaire + renderer pour du contenu structuré, exception HTTP pour le contrôle de flux HTTP standard, ou objet `Response` lorsque nous devons maîtriser finement status, headers et corps.

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


## 7.5 URL dispatch ou traversal : raisonner par le domaine

Pyramid est inhabituel parmi les frameworks web Python parce qu'il traite **URL dispatch** et **traversal** comme deux modèles de premier rang. Il est donc utile de comprendre leur différence conceptuelle, pas seulement leur syntaxe.

Avec URL dispatch, nous partons de la forme de l'URL :

```text
/members/{member_id}
```

Pyramid fait correspondre cette forme à une route, puis cherche une vue. C'est intuitif pour les APIs REST et les applications dont les ressources sont identifiées par des routes relativement stables.

Avec traversal, nous partons d'un **graphe de ressources Python**. Chaque segment de l'URL sert à descendre dans l'arbre via `__getitem__`. Si notre domaine est naturellement hiérarchique, par exemple :

```text
/projects/acme/documents/specification
```

nous pouvons représenter :

```text
Root
 `-- projects
      `-- acme
           `-- documents
                `-- specification
```

Le contexte obtenu après traversal est la ressource finale. La vue peut ensuite être choisie selon le type du contexte et un nom de vue, ce qui rapproche l'URL du modèle objet.

L'un des grands intérêts historiques du traversal est l'**héritage des ACL**. Un projet peut déclarer qu'un groupe a le droit `view`, puis ses documents héritent de cette politique sauf exception. Cela correspond bien aux systèmes documentaires, CMS et arbres de contenus. Avec URL dispatch, la même politique est possible, mais elle est généralement calculée par une factory ou une couche de service au lieu d'être portée directement par l'arbre.

Il n'existe pas de réponse universelle. Pour une API JSON moderne avec `/users`, `/orders/{id}` et `/reports`, URL dispatch est souvent plus évident. Pour un espace documentaire où les permissions suivent une arborescence, traversal peut produire un modèle remarquablement naturel.

Pyramid permet même de combiner les deux. Une route peut fournir une `factory` qui construit le contexte de sécurité, tandis que certaines zones de l'application utilisent un traversal complet. Cette flexibilité est puissante, mais l'équipe doit garder une convention claire afin qu'un développeur sache où chercher la logique de résolution.

### Exemple minimal de ressource

```python
class Resource:
    __parent__ = None
    __name__ = None

    def __init__(self, name, parent=None):
        self.__name__ = name
        self.__parent__ = parent
        self.children = {}

    def __getitem__(self, name):
        return self.children[name]
```

Les attributs `__parent__` et `__name__` permettent à Pyramid de raisonner sur la lignée de ressources. Dans une application réelle, ces objets peuvent être persistés en ZODB ou construits à partir d'autres données.

La question pratique à se poser est donc : **mon URL décrit-elle principalement une opération HTTP, ou reflète-t-elle réellement une hiérarchie de ressources et de permissions ?** La réponse guide le modèle de dispatch.

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


## 8.8 Concevoir l'état web avant de coder l'authentification

L'ancien cours associait assez vite cookie, session et authentification. Dans une application moderne, il est utile de les distinguer dès le départ.

Le navigateur conserve des **cookies**. Pyramid peut utiliser un cookie signé pour stocker une session, ou seulement un identifiant permettant de retrouver une session côté serveur. L'authentification est encore autre chose : elle consiste à relier la requête courante à une identité. Enfin, l'autorisation décide si cette identité peut effectuer une action.

```text
cookie HTTP
   |
   v
session éventuelle
   |
   v
SecurityPolicy -> identity
   |
   v
permission -> allow / deny
```

Ce découpage évite une erreur classique : considérer que la présence d'une clé `user_id` dans `request.session` suffit à sécuriser l'application. Une identité doit être chargée et revalidée. Si l'utilisateur a été désactivé depuis la création de sa session, la politique de sécurité doit pouvoir refuser de le reconnaître.

Pour les sessions en cookie signé, la signature garantit principalement **l'intégrité**. Elle ne rend pas le contenu secret. Plaçons donc dans la session des identifiants et préférences de faible sensibilité, pas des mots de passe, tokens d'API ou données confidentielles. La taille totale doit également rester petite car le cookie voyage à chaque requête.

Le réglage des attributs de cookie dépend du flux :

```python
SignedCookieSessionFactory(
    secret,
    secure=True,
    httponly=True,
    samesite="Lax",
)
```

`Secure` impose l'envoi sur HTTPS ; `HttpOnly` réduit l'exposition au JavaScript ; `SameSite` limite certains envois cross-site. Aucun de ces attributs ne remplace à lui seul une stratégie CSRF.

Le CSRF devient concret avec notre formulaire d'édition d'un membre. Un attaquant ne doit pas pouvoir faire publier silencieusement par le navigateur de l'administrateur :

```text
POST /members/42/disable
Cookie: session=...
```

La protection consiste à exiger une information que le site hostile ne peut pas fabriquer correctement, typiquement un token CSRF, et à vérifier l'origine selon la politique choisie. Pyramid permet de rendre la vérification systématique pour les méthodes non sûres via `set_default_csrf_options(require_csrf=True)`.

Il faut cependant distinguer deux architectures. Si l'API utilise une authentification par bearer token envoyé explicitement dans `Authorization` et non automatiquement par le navigateur, le modèle de menace CSRF est différent. À l'inverse, une application HTML classique avec session cookie doit traiter le CSRF comme une préoccupation de base.

Le logout mérite aussi attention. Il ne suffit pas toujours de supprimer un champ de session ; la politique de sécurité doit produire les headers nécessaires via `forget()` ou son helper. Avec un fournisseur OIDC, le logout local et le logout chez l'Identity Provider sont deux opérations distinctes : fermer la session Pyramid ne garantit pas que la session SSO distante a disparu.

Enfin, les secrets de session doivent être gérés comme de vrais secrets de production : longs, aléatoires, distincts entre environnements, injectés hors du dépôt Git et renouvelables. Une rotation brutale invalide toutes les sessions existantes ; selon le contexte, cette conséquence peut être acceptable ou nécessiter une stratégie de transition.

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


## 9.10 Security Policy de bout en bout

Le changement majeur entre Pyramid 1.x et 2.x mérite un exemple complet. Dans l'ancien modèle, une authentication policy calculait un userid et des principals, puis une authorization policy interprétait les ACL. Pyramid 2.x regroupe ce contrat dans une **Security Policy**. Les helpers historiques restent utiles, mais la policy devient le point central.

Un exemple simplifié avec authentification en session :

```python
from pyramid.authentication import SessionAuthenticationHelper
from pyramid.authorization import ACLHelper
from pyramid.request import RequestLocalCache


class SecurityPolicy:
    def __init__(self, user_service):
        self.helper = SessionAuthenticationHelper()
        self.acl = ACLHelper()
        self.user_service = user_service
        self.identity_cache = RequestLocalCache(self.load_identity)

    def load_identity(self, request):
        userid = self.helper.authenticated_userid(request)
        if userid is None:
            return None
        user = self.user_service.get(userid)
        if user is None or not user.enabled:
            return None
        return user

    def identity(self, request):
        return self.identity_cache.get_or_create(request)

    def authenticated_userid(self, request):
        identity = self.identity(request)
        return identity.id if identity is not None else None

    def permits(self, request, context, permission):
        identity = self.identity(request)
        principals = ["system.Everyone"]
        if identity is not None:
            principals.append("system.Authenticated")
            principals.append(f"user:{identity.id}")
            principals.extend(f"group:{g}" for g in identity.groups)
        return self.acl.permits(context, principals, permission)

    def remember(self, request, userid, **kw):
        return self.helper.remember(request, userid, **kw)

    def forget(self, request, **kw):
        return self.helper.forget(request, **kw)
```

Ce code est volontairement pédagogique : une application réelle adaptera les types, le chargement de l'utilisateur et la stratégie d'autorisation. L'idée importante est que `request.identity` peut être un véritable objet `User`, pas seulement une chaîne. Une vue peut ainsi exprimer :

```python
if request.identity is not None:
    display_name = request.identity.display_name
```

La politique revalide l'utilisateur au moment de la requête. Si son compte est supprimé ou désactivé, une ancienne session ne suffit pas à conserver l'accès.

La connexion appelle `remember()` après vérification des credentials :

```python
headers = request.security.remember(user.id)
raise HTTPFound(location=request.route_url("home"), headers=headers)
```

La déconnexion utilise `forget()`. Nous évitons ainsi de connaître dans la vue le détail du cookie ou du helper.

### Autorisation : permission plutôt que rôle dans chaque vue

Une vue ne devrait pas demander manuellement :

```python
if request.identity.role != "admin":
    ...
```

Partout dans le code. Déclarons plutôt une intention :

```python
@view_config(
    route_name="member_edit",
    permission="member:edit",
    renderer="members/edit.pt",
)
def edit_member(request):
    ...
```

La politique décide ensuite quels groupes ou attributs donnent `member:edit`. Cette indirection facilite les changements d'organisation et les tests.

Les ACL sont particulièrement adaptées lorsque la permission dépend du **contexte**. Un membre peut être modifiable par les administrateurs globaux et par le responsable de son organisation. Le contexte peut calculer une ACL à partir de ses données. Si notre domaine utilise plutôt RBAC ou ABAC centralisé, la méthode `permits()` peut utiliser une autre stratégie : Pyramid n'impose pas ACLHelper.

### Authentification directe, LDAP et OIDC

La Security Policy ne devrait pas elle-même vérifier un mot de passe LDAP à chaque requête. Elle identifie une session déjà établie. Le processus de login vérifie les credentials ou termine un flux OIDC, puis mémorise l'identité locale. Cela évite de transformer l'annuaire en dépendance obligatoire de chaque page consultée.

Ce découpage permet aussi d'ajouter MFA ou SSO sans réécrire les permissions. L'authentification produit une identité ; l'autorisation reste un problème distinct.

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


## 10.6 Du formulaire au service métier

Deform et Colander sont plus faciles à comprendre si nous les plaçons dans le flux complet d'une mutation. Prenons la création d'un membre. Le navigateur envoie des chaînes ; Colander les désérialise et valide leur forme ; Deform sait rendre le formulaire et ses erreurs ; le service métier vérifie ensuite des contraintes qui dépendent du domaine ou de la base.

```text
request.POST
   |
   v
Deform / Colander
   |   validation syntaxique et structurelle
   v
appstruct Python
   |
   v
MemberService
   |   règles métier
   v
SQLAlchemy / transaction
```

La distinction est importante. Une adresse email mal formée est une erreur de validation. Une adresse déjà utilisée par un membre actif est une règle métier dépendante de l'état courant. Essayer d'encoder toute la logique métier dans le schéma Colander produit des validateurs difficiles à tester et fortement couplés à la base.

Exemple simplifié :

```python
import colander


class MemberSchema(colander.MappingSchema):
    display_name = colander.SchemaNode(
        colander.String(),
        validator=colander.Length(min=2, max=120),
    )
    email = colander.SchemaNode(colander.String())
```

Puis dans la vue :

```python
@view_config(route_name="members_new", renderer="members/edit.pt")
def new_member(request):
    form = deform.Form(MemberSchema(), buttons=("save",))

    if request.method == "POST":
        try:
            data = form.validate(request.POST.items())
        except deform.ValidationFailure as exc:
            return {"form": exc.render()}

        try:
            member = request.services.members.create(data)
        except DuplicateEmail:
            # rattacher une erreur métier à l'interface
            ...
        else:
            request.session.flash("Membre créé", "success")
            raise HTTPFound(
                request.route_url("member", member_id=member.id)
            )

    return {"form": form.render()}
```

Dans une vraie application, le traitement d'erreur doit rester cohérent avec le renderer et la politique CSRF, mais l'idée centrale reste la même : **la vue orchestre, elle ne possède pas la règle métier**.

Les formulaires permettent aussi de comprendre la différence entre « assainir » et « valider ». Transformer arbitrairement une entrée pour essayer de la rendre sûre peut modifier son sens. Pour un nom, une bio ou un commentaire, nous préférons généralement conserver la valeur métier et **échapper au moment du rendu**. Pour une URL, un identifiant ou une date, nous validons selon un format et une politique. Pour du HTML volontairement accepté, il faut un sanitizer spécialisé avec allowlist.

Les uploads requièrent un traitement particulier : taille maximale, type réel, nom de fichier généré côté serveur, stockage hors d'un répertoire exécutable et analyse supplémentaire selon la sensibilité. Le nom fourni par le navigateur ne doit jamais devenir directement un chemin disque.

Enfin, un formulaire HTML n'est pas une frontière de confiance. Les contraintes `required`, `min`, `pattern` ou JavaScript améliorent l'expérience utilisateur, mais un client peut envoyer n'importe quelle requête HTTP. Toutes les validations importantes doivent être appliquées côté serveur.

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


## 11.7 Comprendre l'unité de travail

La persistance est l'un des endroits où l'ancien cours apportait une intuition utile. Une requête web effectue souvent plusieurs écritures qui forment une seule opération logique : créer un membre, créer son profil, journaliser l'action et éventuellement enregistrer une invitation. Nous ne voulons pas qu'une moitié de l'opération soit validée si l'autre échoue.

C'est le rôle de la **transaction** :

```text
requête POST
   |
   v
transaction begin
   |
   +--> INSERT member
   +--> INSERT profile
   +--> INSERT audit_event
   |
   +--> succès -> commit
   `--> exception -> rollback
```

Avec `pyramid_tm` et `zope.sqlalchemy`, l'intégration peut lier la transaction à la durée logique de la requête. Cela réduit le besoin de disperser des appels `commit()` dans les vues. Mais il faut comprendre ce que cette commodité signifie : si nous attrapons une exception et continuons comme si tout allait bien, nous devons savoir dans quel état se trouve la session SQLAlchemy.

Une **session SQLAlchemy** n'est ni `request.session`, ni la transaction manager. Elle représente l'unité de travail ORM : suivi des objets chargés, modifications, flush et interaction avec la connexion SQL. Dans un projet pédagogique, les trois mots « session » apparaissent souvent dans la même page et créent une confusion durable ; il faut les distinguer explicitement.

Le `flush` mérite aussi une explication. Il envoie les changements SQL nécessaires sans valider définitivement la transaction. Cela permet par exemple d'obtenir une clé primaire avant la fin de la requête. Le `commit`, lui, rend la transaction durable selon les garanties du SGBD.

Les appels externes compliquent l'atomicité. Si nous écrivons en base puis envoyons immédiatement un email SMTP, un échec après le commit ne peut pas être annulé par une transaction SQL. Pour les opérations importantes, un pattern de type **outbox** ou une file de tâches est plus robuste : la transaction enregistre l'intention d'envoyer, puis un worker réalise l'effet externe.

### SQLAlchemy ou ZODB ?

Pyramid ne force pas le choix. SQLAlchemy est naturel pour un modèle relationnel, des requêtes analytiques, des contraintes SQL et un écosystème de migrations Alembic. ZODB stocke directement des graphes d'objets Python persistants et s'intègre historiquement très bien avec le traversal.

Le choix ne doit pas être idéologique. Une application documentaire hiérarchique peut tirer profit d'un arbre ZODB ; une application métier fortement relationnelle avec reporting SQL préférera souvent PostgreSQL + SQLAlchemy. Certaines applications Pyramid historiques combinent même plusieurs sources.

Dans tous les cas, évitons de transmettre des objets ORM non maîtrisés jusque dans les templates après fermeture de leur contexte de données. Les chargements paresseux peuvent déclencher des requêtes inattendues, provoquer un N+1 ou échouer si la session n'est plus utilisable. Une couche service peut charger explicitement ce dont la vue a besoin.

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


## 12.8 Étude de cas : annuaire LDAP et SSO

L'ancien cours construisait directement une authentification Pyramid sur OpenLDAP. Cette architecture reste possible, mais il faut aujourd'hui distinguer deux besoins : **consulter un annuaire LDAP** et **authentifier un navigateur**.

LDAP est excellent pour rechercher des entrées, groupes et attributs. Il n'est pas, à lui seul, un protocole moderne de SSO web. Dans une organisation équipée d'un fournisseur d'identité, une architecture plus fréquente est :

```text
navigateur
   |
   v
Pyramid
   |
   +--> OpenID Connect
   |        |
   |        v
   |      IdP
   |        |
   |        +--> LDAP / Active Directory
   |
   `--> LDAP direct pour attributs métier si nécessaire
```

Pyramid délègue alors l'authentification à l'IdP et reçoit une identité signée via le flux OIDC. L'annuaire peut rester la source de vérité des comptes sans que l'application manipule elle-même le mot de passe de l'utilisateur.

Il existe néanmoins des environnements où le bind LDAP direct est encore requis. La séquence correcte doit être conçue avec soin. Un compte de service peut rechercher le DN correspondant au login, puis une connexion séparée tente un bind avec le mot de passe fourni. Le mot de passe ne doit ni être journalisé, ni stocké dans la session.

Avec `ldap3`, la recherche peut ressembler à :

```python
from ldap3 import Connection, Server, Tls
import ssl


tls = Tls(validate=ssl.CERT_REQUIRED)
server = Server("ldap.example.org", use_ssl=True, tls=tls)

with Connection(
    server,
    user=settings["ldap.bind_dn"],
    password=settings["ldap.bind_password"],
    auto_bind=True,
) as conn:
    conn.search(
        "ou=people,dc=example,dc=org",
        "(uid=alice)",
        attributes=["uid", "cn", "mail"],
    )
```

La construction du filtre doit utiliser les mécanismes d'échappement de la bibliothèque ; concaténer directement une saisie utilisateur dans un filtre LDAP ouvre la porte aux injections LDAP.

La validation TLS est impérative. Désactiver la vérification de certificat transforme une connexion `ldaps://` en canal vulnérable à l'interception. En production, nous configurons une chaîne de confiance correcte et des timeouts explicites. Une panne LDAP ne doit pas bloquer indéfiniment tous les workers WSGI.

### Mapper l'identité au domaine applicatif

L'identité externe ne doit pas forcément devenir directement le modèle métier. Un utilisateur OIDC peut porter un `sub`, un email et des groupes ; l'application peut mapper ces éléments vers un `User` local contenant ses préférences et permissions spécifiques.

```text
claims OIDC / attributs LDAP
          |
          v
identity loader
          |
          v
User local
          |
          v
permissions applicatives
```

Cela permet de désactiver localement un accès, de conserver un historique et de ne pas lier toute l'architecture aux noms de groupes d'un annuaire particulier.

La Security Policy est le bon endroit pour résoudre l'identité et éventuellement mettre en cache le résultat pendant une requête. Pyramid 2.x expose `request.identity`, ce qui évite de propager partout des chaînes de principals comme dans les anciens patterns 1.x. L'autorisation peut néanmoins toujours utiliser `ACLHelper` lorsque l'héritage d'ACL correspond au domaine.

Enfin, l'authentification d'entreprise doit être pensée pour les erreurs : IdP indisponible, LDAP lent, certificat expiré, utilisateur sans email, groupe supprimé, session distante expirée. Un bon système distingue les erreurs temporaires des refus d'accès et n'expose jamais le détail d'une exception LDAP au navigateur.

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


## 13.6 Tweens, événements et composants : les points d'extension de Pyramid

Une fois les routes et vues comprises, Pyramid devient surtout intéressant par ses **points d'extension**. Les tweens ressemblent à des middlewares placés autour du handler principal. Ils sont adaptés aux préoccupations transversales : mesure de latence, identifiant de corrélation, instrumentation, politique d'erreur ou intégration avec un outil de tracing.

Un tween conceptuel :

```python
import time


def timing_tween_factory(handler, registry):
    def timing_tween(request):
        start = time.perf_counter()
        try:
            return handler(request)
        finally:
            duration = time.perf_counter() - start
            request.registry.logger.info(
                "request_done",
                extra={"duration": duration, "path": request.path},
            )
    return timing_tween
```

Ce code entoure la suite du pipeline. Il ne faut pas pour autant transformer un tween en couche métier : il doit rester générique et indépendant de la route précise.

Les **événements** servent à notifier des composants sans couplage direct. Pyramid publie notamment des événements de cycle de requête et d'application. Une extension peut s'abonner à un événement pour initialiser ou nettoyer une ressource. Ce modèle est utile lorsqu'un package doit se greffer à l'application sans que toutes les vues l'importent explicitement.

Les **request methods** ajoutent des capacités calculées à l'objet `request`. Le cookiecutter SQLAlchemy, par exemple, expose classiquement une session de base liée à la requête. Nous pouvons de même ajouter un accès à un service :

```python
def get_member_service(request):
    return MemberService(request.dbsession)


config.add_request_method(
    get_member_service,
    "member_service",
    reify=True,
)
```

Le `reify=True` calcule la valeur au premier accès puis la mémorise pour la requête. Cette commodité est utile, mais elle doit rester lisible : si chaque dépendance devient un attribut magique de `request`, les tests et la compréhension du code se dégradent.

`RequestLocalCache` répond à un besoin proche mais plus général : calculer une donnée une fois pendant la requête. La Security Policy officielle l'utilise volontiers pour charger l'identité sans refaire plusieurs requêtes SQL lorsque différentes permissions sont évaluées.

La **registry** est enfin le cœur du système de composants. Nous pouvons y enregistrer des utilities ou adapters selon des interfaces. Cette architecture vient de l'écosystème Zope et explique pourquoi Pyramid peut être étendu très profondément sans que le framework central connaisse les extensions à l'avance.

Dans une petite application, n'utilisons pas ces abstractions uniquement parce qu'elles existent. Une fonction et un constructeur explicites sont souvent plus simples. Elles deviennent précieuses quand plusieurs packages doivent contribuer à une même application et lorsqu'on veut remplacer un composant selon l'environnement ou le test.

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


## 15.8 Tester un comportement plutôt qu'une implémentation

Les tests d'une application Pyramid doivent former une pyramide eux aussi. La base est constituée de tests unitaires rapides sur les services métier. Au-dessus viennent des tests d'intégration qui construisent une registry, une base ou une Security Policy. Enfin, quelques tests fonctionnels passent réellement par l'application WSGI.

Pour notre `MemberService`, le test le plus utile ne nécessite pas Pyramid :

```python
def test_disable_member_records_reason(member, service):
    service.disable(member.id, reason="leaving")
    assert member.disabled is True
    assert member.disabled_reason == "leaving"
```

La vue, elle, doit surtout prouver qu'elle traduit correctement HTTP vers le service : code de statut, permission, redirection, validation des entrées. WebTest est pratique pour vérifier l'ensemble du chemin WSGI sans démarrer un serveur réseau réel.

```python
def test_anonymous_cannot_create_member(testapp):
    response = testapp.post_json(
        "/members",
        {"display_name": "Alice"},
        status=403,
    )
    assert response.status_int == 403
```

Pour une application protégée par CSRF, il faut également tester l'absence et l'invalidité du token. Pour LDAP ou OIDC, les tests unitaires ne doivent pas dépendre du véritable serveur de production : utilisons des doubles, un serveur de laboratoire ou des fixtures contrôlées. Quelques tests d'intégration réels peuvent ensuite vérifier la compatibilité du protocole.

Les fixtures fournies par le cookiecutter Pyramid sont utiles parce qu'elles savent construire et démonter proprement l'environnement de test. Le contexte `pyramid.testing.testConfig()` permet aussi d'enregistrer temporairement routes, utilities ou politiques sans polluer les tests suivants.

Une erreur fréquente est de surutiliser `DummyRequest`. C'est un bon outil pour une fonction pure qui lit quelques attributs, mais il ne reproduit pas automatiquement toute la mécanique d'une vraie requête : tweens, traversal, security policy, renderer, transaction manager. Dès que nous testons une interaction entre plusieurs couches, un test WSGI via WebTest est souvent plus représentatif.

Le diagnostic en développement complète les tests. `proutes` répond à « quelle route Pyramid a-t-il réellement enregistrée ? », `pviews` à « quelle vue correspond à ce contexte ? », `ptweens` à « dans quel ordre passent mes middlewares ? », `pshell` permet d'inspecter l'application avec sa registry et `prequest` d'exécuter une requête depuis la ligne de commande.

La debug toolbar est très pratique mais doit être réservée au développement. Elle donne accès à des informations extrêmement sensibles : exceptions, environnement, paramètres, introspection. L'exposer publiquement est un incident de sécurité, pas un simple défaut esthétique.

Enfin, les tests de migration ont une valeur particulière dans un vieux projet Pyramid. Avant de remplacer `AuthTktAuthenticationPolicy`, une session Pickle ou une intégration SQLAlchemy 1.x, écrivons des tests sur le comportement actuel. Ils servent de filet lors de la modernisation et permettent de distinguer une incompatibilité réelle d'un changement volontaire.

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


## 17.8 Déployer sans confondre application et serveur

Une application Pyramid est une application WSGI ; elle n'écoute pas elle-même nécessairement sur un socket. `pserve` charge à la fois une configuration d'application et une configuration de serveur. Dans le starter officiel, Waitress fournit généralement le serveur WSGI.

En production, nous séparons souvent les rôles :

```text
Internet
   |
   v
reverse proxy / load balancer
   |  TLS, limites, headers, compression
   v
Waitress / Gunicorn / mod_wsgi
   |
   v
application Pyramid
   |
   +--> PostgreSQL
   +--> Redis
   +--> LDAP / IdP
   `--> file de tâches
```

Cette séparation aide à raisonner sur les frontières de confiance. Le reverse proxy reçoit les connexions non fiables et réécrit les headers de forwarding. Le serveur WSGI peut n'écouter que sur `127.0.0.1` ou sur un réseau privé. L'application ne doit faire confiance à `X-Forwarded-For` ou `X-Forwarded-Proto` que si la chaîne de proxies est maîtrisée.

Les **timeouts** sont une partie de la sécurité et de la disponibilité. Si une vue appelle LDAP sans timeout, quelques connexions lentes peuvent immobiliser tous les workers. La même règle vaut pour la base, SMTP et les API distantes. Un système synchrone WSGI reste très efficace pour beaucoup d'applications métier, à condition de ne pas transformer chaque worker en attente infinie.

Les traitements longs doivent être sortis du cycle HTTP. Pour un export de 500 000 membres, la vue peut créer un job et répondre `202 Accepted`; un worker produit ensuite le fichier. La requête de consultation du job reste courte et prévisible.

Le déploiement conteneurisé ne change pas ces principes. Un Dockerfile doit installer des dépendances reproductibles, utiliser un utilisateur non root, ne pas incorporer les secrets dans l'image et démarrer un vrai serveur WSGI. La configuration d'environnement est injectée au runtime.

```dockerfile
FROM python:3.14-slim
WORKDIR /app
COPY pyproject.toml ./
COPY . .
RUN pip install --no-cache-dir .
RUN useradd --create-home app
USER app
CMD ["pserve", "production.ini"]
```

En pratique, nous pouvons optimiser les layers et pinner davantage les dépendances, mais ce squelette montre les responsabilités. Le secret de cookie, le mot de passe SQL et le bind password LDAP ne doivent jamais apparaître dans le `Dockerfile` ni dans une image publiée.

Deux endpoints de santé peuvent être distingués : **liveness**, qui répond si le processus est capable de traiter des requêtes, et **readiness**, qui indique s'il est prêt à recevoir du trafic. Une readiness peut vérifier des dépendances critiques avec parcimonie, mais ne doit pas déclencher une cascade d'appels lourds à chaque seconde.

Les logs doivent permettre de suivre une requête sans révéler de secrets. Un identifiant de corrélation, la route, le statut, la durée et éventuellement l'identité technique suffisent souvent. Ne journalisons jamais les mots de passe, bearer tokens, cookies complets ou corps sensibles.

Enfin, un déploiement Pyramid doit être reproductible. Si l'installation dépend de commandes manuelles connues d'une seule personne, la robustesse du framework ne compensera pas la fragilité opérationnelle. Les migrations Alembic, variables nécessaires, health checks, procédure de rollback et version de Python doivent être documentés avec l'application.

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


## 21.9 Étude de migration : d'un projet Pyramid 1.x vers 2.1

Une migration réaliste ne consiste pas à remplacer toutes les anciennes APIs en une seule fois. Un projet Pyramid 1.x peut fonctionner depuis dix ans, contenir plusieurs centaines de vues et dépendre d'extensions qui ont leur propre calendrier. La stratégie la plus sûre est **incrémentale**.

Commençons par établir un inventaire : version Python, version Pyramid, méthode de packaging, policies de sécurité, type de session, SQLAlchemy, ZODB, extensions, serveur WSGI et tests existants. Il faut notamment rechercher :

```text
set_authentication_policy
set_authorization_policy
AuthTktAuthenticationPolicy
SessionAuthenticationPolicy
ACLAuthorizationPolicy
effective_principals
unauthenticated_userid
PickleSerializer
pcreate
setup.py historiques
```

Ces éléments ne sont pas tous équivalents en risque. Une ancienne API simplement dépréciée peut continuer à fonctionner pendant la transition, alors qu'une session sérialisée avec Pickle ou une dépendance Python obsolète mérite une priorité de sécurité plus forte.

### Étape 1 : rendre le comportement observable

Avant toute refonte, exécutons les tests, ajoutons des tests fonctionnels aux flux critiques et utilisons `proutes`, `pviews` et `ptweens` pour capturer la configuration effective. Une migration sans filet transforme chaque régression en enquête archéologique.

### Étape 2 : monter Python et les dépendances

Pyramid 2.1 ne supporte plus Python 3.9 et antérieurs. Il faut donc d'abord vérifier que l'application et ses extensions fonctionnent sur Python 3.10 ou plus récent. Dans de nombreux projets, cette étape révèle des dépendances abandonnées avant même d'atteindre Pyramid.

### Étape 3 : moderniser la sécurité

Pyramid 2.x fusionne les anciennes authentication/authorization policies dans une **Security Policy**. Nous pouvons reproduire le comportement existant avec `AuthTktCookieHelper` ou `SessionAuthenticationHelper`, puis exposer `request.identity`.

L'objectif n'est pas uniquement de faire disparaître un warning. Le nouveau modèle permet de représenter l'identité par un objet métier plutôt que de propager uniquement un userid et des principals. Nous pouvons ensuite réécrire progressivement les vérifications de permission.

### Étape 4 : traiter les sessions

Si un ancien projet utilise `PickleSerializer`, la migration vers JSON peut invalider ou rendre incompatibles les sessions existantes. Il faut le prévoir comme un événement de déploiement : soit les utilisateurs se reconnectent, soit une phase de compatibilité est construite. La sécurité vaut souvent davantage que la conservation d'une session historique.

### Étape 5 : moderniser SQLAlchemy

Le passage à SQLAlchemy 2.x mérite son propre chantier. Réécrivons les requêtes legacy, vérifions le modèle transactionnel et exécutons les migrations Alembic en environnement de préproduction. Ne mélangeons pas une refonte complète du domaine et un changement de framework dans le même commit si nous pouvons l'éviter.

### Étape 6 : supprimer l'ancien outillage

`pcreate` et les scaffolds intégrés ont disparu depuis Pyramid 2.0 au profit du cookiecutter. Un projet existant n'a pas besoin d'être recréé avec Cookiecutter ; il peut simplement adopter progressivement les conventions modernes de `pyproject.toml`, tests et packaging.

### Étape 7 : déployer par étapes

Une migration critique devrait disposer d'une procédure de rollback. Si les sessions, schémas SQL ou formats de données ont changé, ce rollback doit être pensé avant le déploiement. Une version précédente du code n'est pas forcément capable de relire des données modifiées par la nouvelle.

Cette méthode illustre un principe général : **moderniser un projet Pyramid est un problème de compatibilité et d'observabilité avant d'être un problème de syntaxe**. Le framework offre beaucoup de stabilité ; profitons-en pour réduire le risque plutôt que de réécrire l'application sans nécessité.

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
