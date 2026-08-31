---
schema_version: 1
uid: 01M1BQ62D5GEKYFH8WG1TG6HXA
titre: "Pyramid — 10 — Exemples de code"
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
resume: "Chapitre 10 sur 10 du livre « Pyramid » : Exemples de code. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « Pyramid » — chapitre 10/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 09 — Pyramid et Docker|← 09 — Pyramid et Docker]] · [[Pyramid — Compléments 2026|Compléments 2026 →]]

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

Pour configurer le port et l'adresse du serveur SMTP dans Pyramid Mailer, nous pouvons utiliser les options de configuration disponibles dans les paramètres de notre application Pyramid. Ces paramètres peuvent être définis dans le fichier `.ini` de configuration de notre application ou passés en tant que dictionnaire lors de la création de l'application Pyramid. Pour les secrets voir [[Pyramid#3.5.4 Gestion des Secrets dans Pyramid - Utilisation d'un fichier .env]]

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

## 11.  Trucs et astuces
## Débogage en ligne à travers le navigateur
### Utilisation de la toolbar de débogage dans Pyramid
Il suffit de cliquer sur le logo pyramid à droite du contenu et de naviguer dans les onglets.

---
> [!info] Livre « Pyramid » — chapitre 10/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 09 — Pyramid et Docker|← 09 — Pyramid et Docker]] · [[Pyramid — Compléments 2026|Compléments 2026 →]]
