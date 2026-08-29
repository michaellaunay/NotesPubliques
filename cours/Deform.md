---
schema_version: 1
uid: "01M02EX5AW2Y3TPJPPT9FRJ0J2"
titre: "Deform"
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
  - deform
resume: "Cours moderne sur Deform 3 sous Pyramid : schémas Colander, widgets, validation, rendu, CSRF, fichiers, formulaires dynamiques et personnalisation."
niveau: avance
prerequis:
  - "[[Pyramid]]"
  - "[[Python]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-10-13
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Deform sous Pyramid

> [!info]
> Ce cours cible **Deform 3.x**, et plus particulièrement Deform **3.0.1**, publié en février 2026.
> Deform 3 apporte notamment le passage à **Bootstrap 5**, une mise à jour des bibliothèques JavaScript et la prise en charge de la validation HTML5.

Deform est une bibliothèque Python de génération et de traitement de formulaires HTML côté serveur. Elle permet de :

- décrire les données attendues avec **Colander** ;
- générer automatiquement les champs HTML correspondants ;
- convertir une soumission HTTP en structures Python ;
- valider les données côté serveur ;
- afficher les erreurs au niveau des champs ;
- gérer des structures imbriquées et des listes dynamiques ;
- personnaliser le rendu avec des widgets et des templates ;
- intégrer des formulaires dans [[Pyramid]] ou dans un autre framework web.

Deform ne remplace ni Pyramid ni Colander : il se place entre le schéma de données et le navigateur.

```text
Navigateur
    │
    │ POST multipart/form-data ou application/x-www-form-urlencoded
    ▼
Pyramid / WebOb
    │
    │ request.POST.items()
    ▼
Deform + Peppercorn
    │
    │ pstruct → cstruct
    ▼
Colander
    │
    │ désérialisation + validation
    ▼
appstruct Python
```

Les trois bibliothèques importantes à retenir sont donc :

| Bibliothèque | Rôle |
|---|---|
| **Deform** | rendu des formulaires, widgets et gestion des erreurs |
| **Colander** | schémas, sérialisation, désérialisation et validation |
| **Peppercorn** | conversion des contrôles HTML en structures imbriquées compréhensibles par Deform |

# Plan du cours

1. [[#1. Architecture et concepts fondamentaux]]
2. [[#2. Installation et intégration avec Pyramid]]
3. [[#3. Définir les schémas avec Colander]]
4. [[#4. Widgets et rendu HTML]]
5. [[#5. Construire et traiter un formulaire]]
6. [[#6. Validation avancée]]
7. [[#7. Schémas imbriqués et collections dynamiques]]
8. [[#8. Sécurité des formulaires]]
9. [[#9. Fichiers, choix dynamiques et données contextuelles]]
10. [[#10. Personnalisation des templates et ressources]]
11. [[#11. Internationalisation et accessibilité]]
12. [[#12. Tests et architecture applicative]]
13. [[#13. Migration de Deform 2 vers Deform 3]]
14. [[#14. Bonnes pratiques et pièges fréquents]]
15. [[#15. Exemple complet avec Pyramid]]
16. [[#16. Conclusion et ressources]]

# 1. Architecture et concepts fondamentaux

## 1.1. Ce que fait Deform

Un formulaire Deform est construit à partir d'un **schéma Colander**. Deform transforme les nœuds du schéma en objets `Field`, associe un widget à chacun d'eux, puis rend le formulaire en HTML.

À la soumission, le chemin inverse est parcouru :

1. le navigateur envoie des couples nom/valeur ;
2. Deform et Peppercorn reconstruisent une structure imbriquée ;
3. les widgets désérialisent leur représentation HTML ;
4. Colander convertit les valeurs vers les types Python attendus ;
5. Colander exécute les validateurs ;
6. Deform retourne l'**appstruct** si tout est valide ;
7. sinon, Deform lève `deform.ValidationFailure` et peut rendre le formulaire avec les erreurs.

## 1.2. `pstruct`, `cstruct` et `appstruct`

Ces trois termes reviennent fréquemment dans la documentation.

### `pstruct`

Le **pstruct** est la structure issue de la soumission du formulaire, avant conversion par les widgets.

Les valeurs sont encore proches du protocole HTML.

### `cstruct`

Le **cstruct** est la représentation sérialisée manipulée entre Deform et Colander.

Par exemple, une date Python peut être représentée par une chaîne lors du rendu HTML.

### `appstruct`

L'**appstruct** est la structure Python finale utilisable par l'application.

Exemple :

```python
{
    "name": "Ada",
    "age": 36,
    "newsletter": True,
}
```

Le développeur travaille principalement avec l'`appstruct`.

## 1.3. Schéma, champ et widget

Il faut distinguer trois objets :

- le **schema node** décrit la donnée et ses contraintes ;
- le **Field** représente le champ Deform durant une requête ;
- le **Widget** décrit comment cette donnée est affichée et lue dans le navigateur.

```text
colander.SchemaNode
        │
        ▼
   deform.Field
        │
        ▼
   deform.Widget
        │
        ▼
       HTML
```

Un `Field` n'est normalement pas un objet global réutilisé entre les requêtes : Deform le considère comme un objet de durée de vie limitée à une requête HTTP.

## 1.4. Deform n'est pas lié exclusivement à Pyramid

Deform s'intègre particulièrement bien avec Pyramid mais reste une bibliothèque indépendante du framework.

Il est donc possible de l'utiliser avec :

- Pyramid ;
- une application WSGI personnalisée ;
- d'autres frameworks capables d'exposer les données POST à Deform.

Dans ce cours nous utiliserons Pyramid, car c'est l'intégration la plus naturelle dans notre contexte.

# 2. Installation et intégration avec Pyramid

## 2.1. Créer un environnement virtuel

Utiliser un environnement virtuel évite de mélanger les dépendances du projet avec celles du système.

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Sous Windows :

```powershell
.venv\Scripts\Activate.ps1
```

## 2.2. Installer Deform

```bash
python -m pip install "deform>=3,<4"
```

Pour une application Pyramid :

```bash
python -m pip install "pyramid>=2" "deform>=3,<4" pyramid-chameleon
```

Deform installe notamment Colander et Peppercorn comme dépendances.

Dans un projet de production, les versions doivent être verrouillées par le mécanisme choisi par le projet : fichier de contraintes, lockfile, [[ZC.Buidout|Buildout]], `uv`, `pip-tools`, etc.

## 2.3. Intégration minimale avec Pyramid

Deform fournit ses propres templates Chameleon pour ses widgets. Le template de la **page** qui contient le formulaire peut cependant être écrit avec Chameleon, Jinja2 ou un autre moteur.

Une configuration Pyramid minimale ressemble à :

```python
from deform.renderer import configure_zpt_renderer
from pyramid.config import Configurator
from pyramid.session import SignedCookieSessionFactory


def main(global_config, **settings):
    session_factory = SignedCookieSessionFactory(
        settings["session.secret"],
        httponly=True,
        secure=True,
        samesite="Lax",
    )

    config = Configurator(
        settings=settings,
        session_factory=session_factory,
    )

    config.include("pyramid_chameleon")

    configure_zpt_renderer()

    config.add_static_view(
        name="static_deform",
        path="deform:static",
        cache_max_age=3600,
    )

    config.scan()
    return config.make_wsgi_app()
```

> [!warning]
> `secure=True` suppose que l'application est servie en HTTPS. En développement local sans HTTPS, il faudra adapter cette option.

## 2.4. Ressources CSS et JavaScript

Certains widgets nécessitent des ressources CSS ou JavaScript.

Après création du formulaire :

```python
resources = form.get_widget_resources()
```

Le résultat a la forme :

```python
{
    "css": [...],
    "js": [...],
}
```

Ces ressources doivent être incluses par le template de page lorsque le formulaire utilise des widgets qui en ont besoin.

Une approche simple consiste à retourner ces deux listes dans le dictionnaire de la vue :

```python
values = {
    "form": form.render(),
    **form.get_widget_resources(),
}
```

Il faut ensuite générer les URLs avec `request.static_url()` si les ressources sont fournies sous forme d'asset specifications.

# 3. Définir les schémas avec Colander

## 3.1. Correction importante : les schémas sont des schémas Colander

Le code moderne ne doit pas présenter `deform.schema.Schema`, `deform.schema.String` ou `deform.schema.Integer` comme l'API générale de définition des champs.

Deform utilise **Colander** pour ses schémas.

L'import typique est :

```python
import colander
```

Puis :

```python
class PersonSchema(colander.MappingSchema):
    name = colander.SchemaNode(
        colander.String(),
        title="Nom",
    )

    age = colander.SchemaNode(
        colander.Integer(),
        title="Âge",
        validator=colander.Range(0, 130),
    )
```

## 3.2. `MappingSchema`

Un formulaire Deform racine doit normalement représenter une **mapping**, c'est-à-dire une structure clé/valeur analogue à un dictionnaire Python.

```python
class ProfileSchema(colander.MappingSchema):
    username = colander.SchemaNode(colander.String())
    age = colander.SchemaNode(colander.Integer())
```

Une fois validé, l'`appstruct` ressemble à :

```python
{
    "username": "grace",
    "age": 28,
}
```

## 3.3. Les types courants

Quelques types Colander utiles :

| Type | Valeur Python typique |
|---|---|
| `colander.String()` | `str` |
| `colander.Integer()` | `int` |
| `colander.Float()` | `float` |
| `colander.Decimal()` | `Decimal` |
| `colander.Boolean()` | `bool` |
| `colander.Date()` | `datetime.date` |
| `colander.DateTime()` | `datetime.datetime` |
| `colander.Mapping()` | `dict` |
| `colander.Sequence()` | `list` |
| `colander.Tuple()` | `tuple` |

## 3.4. Champs obligatoires et facultatifs

Par défaut, l'absence d'une valeur obligatoire provoque une erreur de validation.

Pour rendre un champ facultatif :

```python
nickname = colander.SchemaNode(
    colander.String(),
    missing=colander.drop,
)
```

Avec `colander.drop`, le champ absent n'est pas ajouté à l'`appstruct`.

On peut également utiliser :

```python
missing=None
```

mais cela signifie alors explicitement que la valeur Python sera `None`.

Il ne faut pas confondre :

- `default` : valeur utilisée lors de la **sérialisation/rendu** si elle manque ;
- `missing` : valeur utilisée lors de la **désérialisation** si l'entrée manque.

## 3.5. `title`, `description` et métadonnées

```python
email = colander.SchemaNode(
    colander.String(),
    title="Adresse électronique",
    description="Une adresse utilisée pour les notifications.",
    validator=colander.Email(),
)
```

Ces métadonnées sont exploitées par les templates Deform pour produire :

- le label ;
- l'aide contextuelle ;
- certaines informations accessibles à l'utilisateur.

## 3.6. Validateurs fournis par Colander

Colander fournit de nombreux validateurs :

```python
colander.Length(min=3, max=50)
colander.Range(min=0, max=120)
colander.Email()
colander.Regex(r"^[a-z0-9_-]+$")
colander.OneOf(["admin", "editor", "reader"])
colander.ContainsOnly([...])
colander.url
colander.uuid
```

Exemple :

```python
username = colander.SchemaNode(
    colander.String(),
    validator=colander.All(
        colander.Length(min=3, max=32),
        colander.Regex(r"^[a-zA-Z0-9_-]+$"),
    ),
)
```

## 3.7. Préparer une valeur avec `preparer`

Le `preparer` transforme une donnée **après sa désérialisation mais avant sa validation**.

```python
def normalize_email(value):
    return value.strip().lower()


email = colander.SchemaNode(
    colander.String(),
    preparer=normalize_email,
    validator=colander.Email(),
)
```

Le `preparer` est utile pour :

- supprimer les espaces inutiles ;
- normaliser la casse ;
- convertir un format utilisateur vers un format canonique ;
- nettoyer une donnée avant validation.

Il ne doit pas servir à cacher une validation de sécurité insuffisante.

# 4. Widgets et rendu HTML

## 4.1. Rôle d'un widget

Un widget Deform réalise trois opérations principales :

1. sérialiser un `cstruct` vers du HTML ;
2. désérialiser les valeurs reçues dans le formulaire vers un `cstruct` ;
3. gérer l'affichage des erreurs de validation.

Un schéma décrit **ce qu'est la donnée** ; un widget décrit **comment l'utilisateur interagit avec elle**.

## 4.2. Widgets courants

```python
from deform import widget
```

Quelques widgets :

- `widget.TextInputWidget` ;
- `widget.TextAreaWidget` ;
- `widget.PasswordWidget` ;
- `widget.CheckboxWidget` ;
- `widget.SelectWidget` ;
- `widget.RadioChoiceWidget` ;
- `widget.CheckboxChoiceWidget` ;
- `widget.DateInputWidget` ;
- `widget.DateTimeInputWidget` ;
- `widget.FileUploadWidget` ;
- `widget.HiddenWidget` ;
- `widget.RichTextWidget` ;
- `widget.SequenceWidget`.

La liste exacte dépend de la version de Deform. Il faut consulter l'API de la version utilisée avant de s'appuyer sur un widget spécialisé.

## 4.3. Associer explicitement un widget

```python
class MessageSchema(colander.MappingSchema):
    subject = colander.SchemaNode(
        colander.String(),
        widget=widget.TextInputWidget(),
    )

    body = colander.SchemaNode(
        colander.String(),
        widget=widget.TextAreaWidget(rows=12),
    )
```

Il n'est pas nécessaire de définir un widget pour chaque champ : Deform sait choisir un widget par défaut pour de nombreux types.

## 4.4. Mot de passe

```python
password = colander.SchemaNode(
    colander.String(),
    widget=widget.PasswordWidget(redisplay=False),
)
```

`redisplay=False` évite de réafficher le mot de passe après une erreur de validation.

> [!important]
> Un mot de passe reçu depuis un formulaire doit être **haché** avec un algorithme adapté aux mots de passe, par exemple Argon2id, avant stockage. Deform n'effectue pas ce travail métier.

## 4.5. Liste de choix

```python
role = colander.SchemaNode(
    colander.String(),
    validator=colander.OneOf(["reader", "editor", "admin"]),
    widget=widget.SelectWidget(
        values=[
            ("reader", "Lecteur"),
            ("editor", "Éditeur"),
            ("admin", "Administrateur"),
        ]
    ),
)
```

La validation serveur reste indispensable même si la liste HTML empêche normalement l'utilisateur de sélectionner une autre valeur.

Un client HTTP peut fabriquer manuellement une requête POST.

## 4.6. Attributs HTML

Les widgets permettent généralement de transmettre des attributs HTML :

```python
widget.TextInputWidget(
    attributes={
        "autocomplete": "email",
        "inputmode": "email",
        "placeholder": "nom@example.org",
    }
)
```

Les attributs HTML améliorent l'ergonomie, mais ne remplacent pas la validation côté serveur.

# 5. Construire et traiter un formulaire

## 5.1. Créer le formulaire

```python
import deform

schema = ProfileSchema()
form = deform.Form(
    schema,
    buttons=("submit",),
)
```

Le schéma racine transmis à `Form` doit représenter une mapping.

## 5.2. Afficher un formulaire vide

```python
html = form.render()
```

Le résultat est une chaîne HTML.

## 5.3. Préremplir un formulaire

Pour modifier un objet existant :

```python
appstruct = {
    "username": "ada",
    "age": 36,
}

html = form.render(appstruct)
```

L'`appstruct` doit contenir des **valeurs Python**, pas les chaînes déjà sérialisées pour HTML.

## 5.4. Valider une soumission

```python
from deform import ValidationFailure

controls = request.POST.items()

try:
    appstruct = form.validate(controls)
except ValidationFailure as exc:
    html = exc.render()
else:
    save_profile(appstruct)
```

`form.validate()` retourne l'`appstruct` validé.

En cas d'erreur, `ValidationFailure.render()` reconstruit le formulaire avec :

- les valeurs saisies ;
- les erreurs associées aux champs ;
- les erreurs globales éventuelles.

## 5.5. Le modèle POST/Redirect/GET

Après une soumission réussie, il est recommandé d'effectuer une redirection :

```python
from pyramid.httpexceptions import HTTPFound

return HTTPFound(
    location=request.route_url("profile")
)
```

Cela évite notamment qu'un rafraîchissement du navigateur renvoie le formulaire.

```text
GET /profile/edit
    ↓
formulaire
    ↓
POST /profile/edit
    ↓ validation OK
HTTP 302
    ↓
GET /profile
```

# 6. Validation avancée

## 6.1. Validateur personnalisé d'un champ

Un validateur reçoit au minimum le nœud et la valeur.

```python
def validate_username(node, value):
    if value.lower() in {"admin", "root", "system"}:
        raise colander.Invalid(
            node,
            "Ce nom d'utilisateur est réservé.",
        )
```

Puis :

```python
username = colander.SchemaNode(
    colander.String(),
    validator=validate_username,
)
```

## 6.2. Accéder à la requête dans un validateur

Pour les validations dépendant de l'application, on peut **binder** le schéma à la requête.

```python
def validate_unique_email(node, value):
    request = node.bindings["request"]

    if request.dbsession.query(User).filter_by(email=value).first():
        raise colander.Invalid(
            node,
            "Cette adresse est déjà utilisée.",
        )
```

Schéma :

```python
class RegistrationSchema(colander.MappingSchema):
    email = colander.SchemaNode(
        colander.String(),
        validator=validate_unique_email,
    )
```

Dans la vue :

```python
schema = RegistrationSchema().bind(request=request)
```

Cette technique évite d'utiliser des variables globales pour accéder à la base de données.

## 6.3. Validation portant sur plusieurs champs

Pour comparer deux valeurs, il est souvent plus clair de valider le schéma entier.

```python
class RegistrationSchema(colander.MappingSchema):
    password = colander.SchemaNode(
        colander.String(),
        widget=widget.PasswordWidget(redisplay=False),
    )

    password_confirm = colander.SchemaNode(
        colander.String(),
        widget=widget.PasswordWidget(redisplay=False),
    )

    def validator(self, node, appstruct):
        if appstruct["password"] != appstruct["password_confirm"]:
            raise colander.Invalid(
                node["password_confirm"],
                "Les deux mots de passe sont différents.",
            )
```

Cette validation est plus cohérente qu'un validateur attaché au seul champ `password` qui essaierait d'accéder à un autre champ.

## 6.4. Erreurs découvertes après la validation

Certaines erreurs ne sont connues qu'après `form.validate()` :

- rejet par une API distante ;
- violation d'une contrainte métier ;
- conflit concurrent ;
- erreur de paiement ;
- échec d'une opération externe.

Dans ce cas, il peut être utile de convertir l'erreur métier en erreur de formulaire et de réafficher celui-ci au lieu de retourner une erreur HTTP générique.

La documentation Deform prévoit des mécanismes pour positionner une erreur après validation sur le formulaire ou un champ.

## 6.5. Validation HTML5 et validation serveur

Deform 3 peut générer des contraintes de validation HTML5.

C'est utile pour l'expérience utilisateur, mais :

> [!danger]
> **Toute validation côté navigateur est contournable.**
> Le schéma Colander doit rester la source de vérité côté serveur.

# 7. Schémas imbriqués et collections dynamiques

## 7.1. Mapping imbriquée

```python
class AddressSchema(colander.MappingSchema):
    street = colander.SchemaNode(colander.String())
    city = colander.SchemaNode(colander.String())
    postal_code = colander.SchemaNode(colander.String())


class PersonSchema(colander.MappingSchema):
    name = colander.SchemaNode(colander.String())
    address = AddressSchema(title="Adresse")
```

L'`appstruct` résultant :

```python
{
    "name": "Ada Lovelace",
    "address": {
        "street": "...",
        "city": "London",
        "postal_code": "...",
    },
}
```

## 7.2. Séquence de valeurs simples

```python
class TagsSchema(colander.SequenceSchema):
    tag = colander.SchemaNode(colander.String())


class ArticleSchema(colander.MappingSchema):
    title = colander.SchemaNode(colander.String())
    tags = TagsSchema()
```

## 7.3. Séquence de mappings

```python
class PhoneSchema(colander.MappingSchema):
    label = colander.SchemaNode(
        colander.String(),
        validator=colander.OneOf(["home", "work", "mobile"]),
    )
    number = colander.SchemaNode(colander.String())


class PhonesSchema(colander.SequenceSchema):
    phone = PhoneSchema()


class ContactSchema(colander.MappingSchema):
    name = colander.SchemaNode(colander.String())
    phones = PhonesSchema()
```

Deform sait rendre des séquences auxquelles l'utilisateur peut ajouter ou retirer des éléments.

C'est l'un de ses principaux avantages par rapport à un formulaire HTML entièrement codé à la main.

## 7.4. Ne pas transformer tout formulaire en structure profondément imbriquée

La possibilité de créer des structures complexes ne signifie pas qu'il faut tout mettre dans un seul formulaire.

Un formulaire trop volumineux augmente :

- le risque d'erreur utilisateur ;
- la complexité du code métier ;
- le coût du rendu ;
- la difficulté des tests ;
- la difficulté de conserver une transaction cohérente.

Il est souvent préférable de découper une opération en plusieurs écrans cohérents.

# 8. Sécurité des formulaires

## 8.1. CSRF

Une requête POST qui modifie des données doit être protégée contre les attaques **Cross-Site Request Forgery**.

Deform fournit `CSRFSchema` :

```python
import colander
from deform.schema import CSRFSchema


class ProfileSchema(CSRFSchema):
    display_name = colander.SchemaNode(colander.String())
```

Le schéma doit être lié à la requête :

```python
schema = ProfileSchema().bind(request=request)
```

Pour que cette approche fonctionne, Pyramid doit disposer d'une session correctement configurée.

Pyramid possède également son propre mécanisme de protection CSRF et peut effectuer la validation au niveau des vues. Dans une application moderne, il faut choisir une stratégie cohérente et l'appliquer à toutes les opérations mutables.

## 8.2. Ne jamais faire confiance aux valeurs d'un widget

Les choix possibles d'un `SelectWidget` ne constituent pas un contrôle de sécurité.

Exemple :

```python
role = colander.SchemaNode(
    colander.String(),
    validator=colander.OneOf(["reader", "editor"]),
    widget=widget.SelectWidget(
        values=[
            ("reader", "Lecteur"),
            ("editor", "Éditeur"),
        ]
    ),
)
```

Sans `OneOf`, un attaquant pourrait envoyer manuellement :

```text
role=admin
```

Même avec `OneOf`, il faut encore vérifier que **l'utilisateur connecté a le droit** d'effectuer l'opération demandée.

Validation des données et autorisation sont deux problèmes différents.

## 8.3. XSS

Deform et son moteur de templates échappent normalement les valeurs selon le contexte de rendu, mais il faut être particulièrement prudent avec :

- les éditeurs de texte riche ;
- les fragments HTML ;
- les templates personnalisés ;
- les données réinjectées via `structure` ou un équivalent qui désactive l'échappement.

Un champ HTML riche doit être nettoyé par une politique explicite si son contenu est ensuite rendu comme HTML.

## 8.4. Mass assignment

Ne transmettez pas aveuglément l'`appstruct` complet à un ORM :

```python
# À éviter si la structure du formulaire n'est pas strictement maîtrisée.
for key, value in appstruct.items():
    setattr(user, key, value)
```

Préférez une affectation explicite :

```python
user.display_name = appstruct["display_name"]
user.email = appstruct["email"]
```

Cela évite qu'un champ inattendu permette de modifier un attribut sensible.

## 8.5. Mots de passe et secrets

Un formulaire peut transporter un secret mais ne doit jamais :

- le journaliser en clair ;
- le stocker en clair ;
- le réafficher inutilement après validation ;
- l'inclure dans une URL ;
- l'insérer dans des messages d'erreur.

# 9. Fichiers, choix dynamiques et données contextuelles

## 9.1. Téléversement de fichiers

Deform propose `FileUploadWidget`.

Le widget a besoin d'un **temporary store** capable de conserver temporairement les fichiers pendant le cycle de validation du formulaire.

Une implémentation minimale peut ressembler à un dictionnaire :

```python
class MemoryTmpStore(dict):
    def preview_url(self, name):
        return None
```

Mais cette implémentation ne convient généralement **pas à la production** :

- les fichiers non fiables seraient conservés en RAM ;
- aucun mécanisme d'expiration n'est fourni automatiquement ;
- un attaquant pourrait provoquer une consommation mémoire excessive.

En production, le stockage temporaire doit :

- limiter la taille des fichiers ;
- expirer les données inutilisées ;
- utiliser un emplacement non exécutable ;
- générer ses propres noms de fichiers ;
- ne pas faire confiance au nom fourni par le navigateur ;
- contrôler le contenu réel du fichier lorsque la sécurité l'exige.

## 9.2. Type MIME et extension

Une extension `.jpg` ou un `Content-Type: image/jpeg` fourni par le client ne prouve pas qu'un fichier est une image JPEG.

Selon le niveau de risque, il faut éventuellement :

- détecter le type réel ;
- décoder puis réencoder une image ;
- appliquer un antivirus ;
- refuser les formats actifs ;
- stocker les fichiers hors du répertoire statique public.

## 9.3. Valeurs de choix provenant de la base de données

Les listes de choix dépendent souvent du contexte courant.

Colander permet d'utiliser le **schema binding** pour construire ou adapter un schéma par requête.

Principe :

```python
schema = SomeSchema().bind(
    request=request,
    categories=categories,
)
```

Le schéma ou ses valeurs différées peuvent alors exploiter ces données.

Cela évite :

- les variables globales ;
- les requêtes exécutées au moment de l'import du module ;
- les données partagées accidentellement entre utilisateurs.

# 10. Personnalisation des templates et ressources

## 10.1. Le rendu par défaut

Les widgets Deform utilisent par défaut des templates **Chameleon/ZPT**.

Deform 3 utilise une présentation basée sur **Bootstrap 5**.

Cela ne signifie pas que la page Pyramid doit obligatoirement être en Chameleon.

Un formulaire rendu par Deform est finalement une chaîne HTML :

```python
rendered_form = form.render()
```

Elle peut donc être intégrée dans une page Jinja2, Chameleon ou autre.

## 10.2. Ajouter ses propres templates Deform

```python
from deform.renderer import configure_zpt_renderer

configure_zpt_renderer([
    "myapp:templates/deform",
])
```

Un répertoire personnalisé placé avant les templates par défaut permet de surcharger certains templates.

Il vaut mieux surcharger uniquement les templates nécessaires plutôt que copier l'ensemble des templates Deform, car une copie complète devient rapidement difficile à maintenir lors des mises à jour.

## 10.3. Renderer spécifique à un formulaire

Il est possible de fournir un renderer directement au formulaire :

```python
form = deform.Form(
    schema,
    renderer=my_renderer,
)
```

Cette approche évite de modifier un renderer global lorsque plusieurs composants de l'application ont des besoins différents.

## 10.4. Créer un widget personnalisé

Un widget personnalisé est utile lorsque les widgets intégrés ne suffisent réellement pas.

Un widget doit généralement savoir :

- rendre un `cstruct` ;
- lire le `pstruct` reçu ;
- signaler ou gérer une erreur ;
- déclarer ses ressources CSS/JS éventuelles.

Pseudo-structure :

```python
from deform.widget import Widget


class MyWidget(Widget):
    template = "my_widget"
    readonly_template = "readonly/my_widget"

    def serialize(self, field, cstruct, **kw):
        return field.renderer(
            self.template,
            field=field,
            cstruct=cstruct,
            **kw,
        )

    def deserialize(self, field, pstruct):
        return pstruct
```

Avant d'écrire un widget, vérifier si le besoin peut être satisfait par :

- un widget existant ;
- des attributs HTML ;
- un template surchargé ;
- un petit composant JavaScript autour d'un champ standard.

# 11. Internationalisation et accessibilité

## 11.1. Internationalisation

Deform et Colander disposent de catalogues de traduction.

Dans Pyramid, il est possible d'ajouter les répertoires de traduction :

```python
config.add_translation_dirs(
    "colander:locale",
    "deform:locale",
    "myapp:locale",
)
```

Les labels et messages propres à l'application doivent eux aussi utiliser le système i18n du projet.

## 11.2. Ne pas confondre traduction et locale

L'i18n des messages ne règle pas automatiquement :

- le format des dates ;
- le format des nombres ;
- les fuseaux horaires ;
- l'ordre prénom/nom ;
- les formats d'adresse.

Ces questions relèvent de la localisation de l'application et doivent être pensées séparément.

## 11.3. Accessibilité

Un formulaire accessible doit au minimum préserver :

- un `label` clairement associé à chaque champ ;
- des messages d'erreur compréhensibles ;
- une navigation complète au clavier ;
- un ordre de tabulation cohérent ;
- un contraste suffisant ;
- une information qui ne dépend pas uniquement de la couleur ;
- des regroupements explicites pour les groupes de choix.

Les templates Deform fournissent une base, mais toute personnalisation CSS ou de template doit être testée avec les critères d'accessibilité du projet.

# 12. Tests et architecture applicative

## 12.1. Tester le schéma indépendamment de la vue

La logique de validation peut être testée directement avec Colander.

```python
import pytest
import colander


def test_age_must_be_positive():
    schema = PersonSchema()

    with pytest.raises(colander.Invalid):
        schema.deserialize({
            "name": "Ada",
            "age": "-10",
        })
```

Ce test est rapide et ne nécessite pas de navigateur.

## 12.2. Tester la vue

Il faut ensuite tester :

- `GET` du formulaire ;
- `POST` invalide ;
- affichage des erreurs ;
- `POST` valide ;
- redirection ;
- modification effective en base ;
- autorisations ;
- CSRF selon la configuration choisie.

## 12.3. Éviter la logique métier dans les widgets

Un widget ne devrait pas :

- exécuter une transaction ;
- envoyer un e-mail ;
- modifier un utilisateur ;
- appeler directement un système métier complexe.

L'architecture recommandée est plutôt :

```text
Vue Pyramid
    │
    ├─ crée/bind le schéma
    ├─ crée le formulaire
    ├─ valide avec Deform
    │
    ▼
Service métier
    │
    ├─ autorisations métier
    ├─ invariants métier
    ├─ transaction
    └─ effets externes
```

## 12.4. Créer le formulaire par requête

Éviter :

```python
FORM = deform.Form(ProfileSchema())
```

au niveau global si le formulaire contient ou peut contenir un état lié à la requête.

Préférer :

```python
def make_profile_form(request):
    schema = ProfileSchema().bind(request=request)
    return deform.Form(schema, buttons=("save",))
```

Les objets `Field` servent notamment de support à l'état temporaire des widgets pendant une requête.

# 13. Migration de Deform 2 vers Deform 3

Deform 3.0 est sorti en février 2026 après une longue période où Deform 2.x était la branche courante.

## 13.1. Bootstrap 5

Le changement visuel majeur est le passage à **Bootstrap 5**.

Les applications qui surchargeaient les templates Deform 2 doivent vérifier :

- classes CSS ;
- structure du markup ;
- formulaires horizontaux ;
- boutons ;
- groupes d'input ;
- messages d'erreur ;
- JavaScript qui sélectionne des éléments via des classes Bootstrap.

Une surcharge de templates écrite pour Bootstrap 3 ne doit pas être supposée compatible avec Deform 3.

## 13.2. Validation HTML5

Deform 3 ajoute la validation HTML5.

Après migration, tester notamment :

- champs obligatoires ;
- nombres ;
- dates ;
- adresses e-mail ;
- attributs générés par les widgets ;
- interaction entre validation navigateur et messages serveur.

## 13.3. Bibliothèques JavaScript

Deform 3 met à jour plusieurs bibliothèques JavaScript, notamment celles utilisées par certains widgets comme l'éditeur riche.

Toute personnalisation JavaScript dépendant d'une version interne précise doit être réévaluée.

## 13.4. Versions de Python

Deform 3 a abandonné les anciennes versions de Python prises en charge par la branche précédente. Le projet PyPI actuel annonce notamment Python 3.10, 3.11, 3.12 et 3.13 dans ses classifiers.

Lors d'une migration, mettre à jour **ensemble** :

- Python ;
- Pyramid ;
- Deform ;
- Colander ;
- les templates personnalisés ;
- les bibliothèques CSS/JavaScript du frontend.

## 13.5. Méthode de migration recommandée

1. inventorier les formulaires et widgets personnalisés ;
2. inventorier les templates Deform surchargés ;
3. relever les dépendances CSS/JS manuelles ;
4. créer une branche dédiée ;
5. mettre à jour les dépendances ;
6. lancer les tests unitaires ;
7. comparer visuellement tous les formulaires ;
8. tester les séquences dynamiques ;
9. tester les fichiers ;
10. tester les erreurs de validation ;
11. vérifier l'accessibilité ;
12. déployer sur un environnement de préproduction.

# 14. Bonnes pratiques et pièges fréquents

## 14.1. Utiliser Colander comme source de vérité

Ne dupliquez pas les mêmes règles dans :

- le template ;
- JavaScript ;
- la vue ;
- le schéma.

La validation serveur doit rester centralisée autant que possible dans le schéma ou la couche métier.

La validation HTML/JavaScript peut dupliquer certaines contraintes **pour l'ergonomie**, mais ne doit jamais devenir la seule validation.

## 14.2. Distinguer validation et autorisation

Un formulaire valide ne signifie pas que l'utilisateur a le droit de réaliser l'action.

```python
# Validation de format
user_id = colander.SchemaNode(colander.Integer())
```

ne remplace pas :

```python
if not request.has_permission("edit", user):
    raise HTTPForbidden()
```

## 14.3. Ne pas utiliser `request.params` par réflexe

Pour une soumission Deform classique, `request.POST.items()` rend explicite que l'on traite le corps POST du formulaire.

Cela évite de mélanger involontairement paramètres GET et POST.

## 14.4. Éviter `except Exception`

Pour la validation Deform :

```python
try:
    appstruct = form.validate(request.POST.items())
except deform.ValidationFailure as exc:
    ...
```

N'attrapez pas toutes les exceptions comme si elles représentaient une erreur utilisateur. Une panne de base de données ou une erreur de programmation doit rester visible comme une véritable erreur applicative.

## 14.5. Ne pas optimiser le rendu prématurément

Le coût dominant d'une soumission est souvent ailleurs :

- base de données ;
- réseau ;
- API externes ;
- templates de page ;
- traitements métier.

Avant de mettre en cache des fragments de formulaire, profiler l'application.

Le cache de formulaires dépendant d'une requête, d'un token CSRF ou d'options utilisateur peut en plus introduire des erreurs de sécurité.

## 14.6. Préférer des fonctions de fabrique

```python
def make_registration_form(request):
    schema = RegistrationSchema().bind(request=request)

    return deform.Form(
        schema,
        buttons=(
            deform.Button(
                "register",
                title="Créer le compte",
                css_class="btn-primary",
            ),
        ),
    )
```

Cela facilite :

- les tests ;
- le binding ;
- l'injection de dépendances ;
- la personnalisation par requête.

# 15. Exemple complet avec Pyramid

Nous allons créer un formulaire d'inscription simple.

## 15.1. Schéma

```python
import colander
import deform
from deform import widget
from deform.schema import CSRFSchema


def validate_unique_email(node, value):
    request = node.bindings["request"]

    if request.dbsession.query(User).filter_by(email=value).first():
        raise colander.Invalid(
            node,
            "Un compte utilise déjà cette adresse.",
        )


class RegistrationSchema(CSRFSchema):
    username = colander.SchemaNode(
        colander.String(),
        title="Nom d'utilisateur",
        validator=colander.All(
            colander.Length(min=3, max=32),
            colander.Regex(r"^[A-Za-z0-9_-]+$"),
        ),
        widget=widget.TextInputWidget(
            attributes={
                "autocomplete": "username",
            }
        ),
    )

    email = colander.SchemaNode(
        colander.String(),
        title="Adresse électronique",
        preparer=lambda value: value.strip().lower(),
        validator=colander.All(
            colander.Email(),
            validate_unique_email,
        ),
        widget=widget.TextInputWidget(
            attributes={
                "autocomplete": "email",
                "inputmode": "email",
            }
        ),
    )

    password = colander.SchemaNode(
        colander.String(),
        title="Mot de passe",
        validator=colander.Length(min=12),
        widget=widget.PasswordWidget(
            redisplay=False,
            attributes={
                "autocomplete": "new-password",
            },
        ),
    )

    password_confirm = colander.SchemaNode(
        colander.String(),
        title="Confirmation",
        widget=widget.PasswordWidget(
            redisplay=False,
            attributes={
                "autocomplete": "new-password",
            },
        ),
    )

    newsletter = colander.SchemaNode(
        colander.Boolean(),
        title="Recevoir la newsletter",
        missing=False,
    )

    def validator(self, node, appstruct):
        if appstruct["password"] != appstruct["password_confirm"]:
            raise colander.Invalid(
                node["password_confirm"],
                "Les mots de passe ne correspondent pas.",
            )
```

> [!note]
> Une contrainte `Length(min=12)` n'est qu'un exemple pédagogique. Une politique de mot de passe réelle doit tenir compte du modèle de menace et des recommandations de sécurité applicables au projet.

## 15.2. Fabrique du formulaire

```python
def make_registration_form(request):
    schema = RegistrationSchema().bind(request=request)

    return deform.Form(
        schema,
        buttons=(
            deform.Button(
                "register",
                title="Créer mon compte",
                css_class="btn-primary",
            ),
        ),
    )
```

## 15.3. Vue Pyramid

```python
import deform
from pyramid.httpexceptions import HTTPFound
from pyramid.view import view_config


@view_config(
    route_name="register",
    renderer="myapp:templates/register.pt",
    request_method=("GET", "POST"),
)
def register_view(request):
    form = make_registration_form(request)

    if request.method == "POST" and "register" in request.POST:
        try:
            appstruct = form.validate(request.POST.items())
        except deform.ValidationFailure as exc:
            rendered_form = exc.render()
        else:
            user = User(
                username=appstruct["username"],
                email=appstruct["email"],
                newsletter=appstruct["newsletter"],
            )

            user.set_password(appstruct["password"])
            request.dbsession.add(user)
            request.dbsession.flush()

            return HTTPFound(
                request.route_url("registration_done")
            )
    else:
        rendered_form = form.render()

    return {
        "form": rendered_form,
        **form.get_widget_resources(),
    }
```

## 15.4. Template Chameleon simplifié

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="utf-8">
    <title>Inscription</title>
</head>
<body>
    <main>
        <h1>Créer un compte</h1>
        <div tal:replace="structure form"></div>
    </main>
</body>
</html>
```

Dans une application réelle, il faut également inclure les ressources CSS/JS requises par le formulaire et la feuille Bootstrap adaptée au rendu Deform 3.

## 15.5. Ce que cet exemple illustre

Le schéma contient :

- les types ;
- les contraintes de forme ;
- la validation inter-champs ;
- la validation dépendant de la base via le `binding` ;
- la protection CSRF ;
- les widgets et attributs HTML.

La vue contient :

- le contrôle du cycle HTTP ;
- la construction du formulaire ;
- l'appel à `validate()` ;
- la gestion de `ValidationFailure` ;
- l'appel de la couche de persistance ;
- la redirection après succès.

La méthode métier `set_password()` contient :

- le hachage du mot de passe ;
- éventuellement la politique de sécurité propre à l'application.

Cette séparation rend le code plus facile à tester et à maintenir.

# 16. Conclusion et ressources

Deform reste particulièrement intéressant pour les applications Pyramid qui ont besoin de formulaires serveur :

- fortement validés ;
- imbriqués ;
- dynamiques ;
- générés à partir de schémas ;
- maintenables sans recréer manuellement toute la mécanique HTML et POST.

Les idées essentielles à retenir sont :

1. **Deform rend les formulaires, Colander décrit et valide les données.**
2. Le schéma racine d'un `Form` représente normalement une mapping.
3. `form.render()` reçoit un `appstruct` Python.
4. `form.validate(request.POST.items())` retourne un `appstruct` validé.
5. `ValidationFailure` permet de réafficher le formulaire avec ses erreurs.
6. Les widgets déterminent la représentation HTML, pas les règles d'autorisation.
7. La validation navigateur ne remplace jamais la validation serveur.
8. Les schémas liés avec `.bind()` permettent d'injecter le contexte d'une requête.
9. CSRF, autorisations, XSS et upload de fichiers doivent être traités explicitement.
10. Une migration vers Deform 3 doit vérifier particulièrement les templates et le passage à Bootstrap 5.

## Ressources officielles

- Documentation Deform : <https://docs.pylonsproject.org/projects/deform/en/latest/>
- Utilisation de base : <https://docs.pylonsproject.org/projects/deform/en/latest/basics.html>
- Validation : <https://docs.pylonsproject.org/projects/deform/en/latest/validation.html>
- Widgets : <https://docs.pylonsproject.org/projects/deform/en/latest/widget.html>
- Templates : <https://docs.pylonsproject.org/projects/deform/en/latest/templates.html>
- Historique des versions : <https://docs.pylonsproject.org/projects/deform/en/latest/changes.html>
- Documentation Colander : <https://docs.pylonsproject.org/projects/colander/en/latest/>
- Documentation Pyramid : <https://docs.pylonsproject.org/projects/pyramid/en/latest/>
- Démonstrations Deform : <https://deformdemo.pylonsproject.org/>
- PyPI : <https://pypi.org/project/deform/>

## Voir aussi

- [[Pyramid]]
- [[Python]]
- [[HTML]]
- [[CSS]]
- [[Sécurité avec Python]]
