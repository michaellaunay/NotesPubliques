---
schema_version: 1
uid: 01M1BQ62D4195AHYAN8M6XRRGR
titre: "Pyramid — 03 — Gestion des requêtes et réponses"
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
resume: "Chapitre 3 sur 10 du livre « Pyramid » : Gestion des requêtes et réponses. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « Pyramid » — chapitre 3/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 02 — Les Routes et Vues dans Pyramid|← 02 — Les Routes et Vues dans Pyramid]] · [[Pyramid — 04 — Introduction à l'authentification|04 — Introduction à l'authentification →]]

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

---
> [!info] Livre « Pyramid » — chapitre 3/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 02 — Les Routes et Vues dans Pyramid|← 02 — Les Routes et Vues dans Pyramid]] · [[Pyramid — 04 — Introduction à l'authentification|04 — Introduction à l'authentification →]]
