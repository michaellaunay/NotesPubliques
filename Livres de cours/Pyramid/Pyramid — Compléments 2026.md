---
schema_version: 1
uid: 01M1BQ62D552ZRGB5XNCQXY7TS
titre: "Pyramid — Compléments 2026"
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
resume: "Compléments apportés au livre « Pyramid » : sections de la version condensée du cours [[Pyramid]] (31 août 2026) dont le sujet est absent de la version longue."
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

> [!info] Livre « Pyramid »
> [[Pyramid — Sommaire|Sommaire]] · [[Pyramid — 10 — Exemples de code|← 10 — Exemples de code]] · [[Pyramid — Sommaire|Sommaire →]]

# Compléments 2026

> [!info] Origine
> Les sections ci-dessous proviennent de la version condensée et actualisée du cours [[Pyramid]] (31 août 2026). Elles traitent de sujets absents de la version longue et n'ont pas été fondues dans les chapitres ; pour les versions logicielles et l'état de l'art du moment, la version condensée fait foi.

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

## 17.2 Waitress

Waitress :

- est WSGI ;
- fonctionne sur plusieurs plateformes ;
- constitue un serveur robuste et simple ;
- est utilisé par défaut dans les projets Pyramid générés.

Pour certains déploiements, Gunicorn ou mod_wsgi peuvent être choisis à la place.

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

## 20.5 Uploads

Pour les fichiers :

- limite de taille ;
- nom généré côté serveur ;
- stockage hors répertoire exécutable ;
- vérification MIME réelle ;
- antivirus si le risque l'exige ;
- permissions minimales ;
- jamais de chemin construit depuis le nom utilisateur sans normalisation.

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

## Conclusion

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
