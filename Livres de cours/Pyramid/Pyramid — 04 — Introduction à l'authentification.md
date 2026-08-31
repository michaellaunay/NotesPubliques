---
schema_version: 1
uid: 01M1BQ62D4J6GCC7PZ50GZHNAW
titre: "Pyramid — 04 — Introduction à l'authentification"
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
resume: "Chapitre 4 sur 10 du livre « Pyramid » : Introduction à l'authentification. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « Pyramid » — chapitre 4/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 03 — Gestion des requêtes et réponses|← 03 — Gestion des requêtes et réponses]] · [[Pyramid — 05 — Sécurisation des données|05 — Sécurisation des données →]]

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

---
> [!info] Livre « Pyramid » — chapitre 4/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 03 — Gestion des requêtes et réponses|← 03 — Gestion des requêtes et réponses]] · [[Pyramid — 05 — Sécurisation des données|05 — Sécurisation des données →]]
