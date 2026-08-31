---
schema_version: 1
uid: 01M1BQ62D5A4P0HKG138DSAZ4C
titre: "Pyramid — 07 — Authentification LDAP"
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
resume: "Chapitre 7 sur 10 du livre « Pyramid » : Authentification LDAP. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « Pyramid » — chapitre 7/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 06 — Introduction à LDAP et OpenLDAP|← 06 — Introduction à LDAP et OpenLDAP]] · [[Pyramid — 08 — Déploiement de l'application Pyramid|08 — Déploiement de l'application Pyramid →]]

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

---
> [!info] Livre « Pyramid » — chapitre 7/10
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 06 — Introduction à LDAP et OpenLDAP|← 06 — Introduction à LDAP et OpenLDAP]] · [[Pyramid — 08 — Déploiement de l'application Pyramid|08 — Déploiement de l'application Pyramid →]]
