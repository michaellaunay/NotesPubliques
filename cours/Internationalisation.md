---
schema_version: 1
uid: "01M02JG1VD3H7NEHRFZNDZFNN7"
titre: "Internationalisation"
aliases:
  - "i18n"
  - "Localisation"
  - "gettext"
  - "Babel"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - internationalisation
  - localisation
  - gettext
  - pyramid
resume: "Cours sur l'internationalisation et la localisation des applications en Python (2026) : concepts et formats, gettext (marquage, pluriels, contexte, extraction, catalogues PO/MO), Babel (catalogues et formatage CLDR des dates, nombres, monnaies), intégration complète dans Pyramid 2.1 avec Chameleon (TranslationStringFactory, localizer, négociation de langue, extraction avec pybabel et lingua), bonnes pratiques et outillage ; tous les exemples ont été exécutés."
niveau: intermediaire
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-01-20
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Internationalisation

> [!abstract] Objectif
> Rendre une application traduisible et localisable : comprendre ce qui se localise (messages, pluriels, dates, nombres), maîtriser la chaîne gettext (marquage, extraction, catalogues `.pot`/`.po`/`.mo`, compilation), utiliser Babel pour les catalogues et le formatage selon la locale, et intégrer le tout dans une application Pyramid avec des gabarits Chameleon — avec un flux de travail reproductible.

> [!info] État du cours
> Réécrit le 29 août 2026 : hiérarchie des titres corrigée, chapitres redondants fusionnés, chapitre Pyramid entièrement refait (la version de 2024 reposait sur un paquet `pyramid-babel` inexistant). Tous les exemples ont été exécutés avec Python 3.12, GNU gettext 0.21, Babel 2.18, lingua 4.16 et Pyramid 2.1.

Voir aussi : [[Python]], [[Pyramid]], [[Deform]], [[HTML]], [[ZC.Buildout]].

# Sommaire

1. Concepts et formats
2. gettext : la chaîne de traduction
3. Babel : catalogues et formatage selon la locale
4. Pyramid et Chameleon : intégration complète
5. Bonnes pratiques
6. Aide-mémoire
7. Sources

# 1. Concepts et formats

## 1.1 Internationalisation et localisation

- **Internationalisation** (i18n : 18 lettres entre le *i* et le *n*) : préparer le logiciel pour qu'il puisse être adapté à toute langue et région **sans modifier le code** — messages externalisés, encodage Unicode, aucun format de date ou de nombre codé en dur.
- **Localisation** (l10n) : produire l'adaptation pour une **locale** donnée — traduction, mais aussi formats de dates, de nombres et de monnaies, règles de pluriel, tri, direction d'écriture, unités, conventions culturelles.
- **Locale** : couple langue–région, `fr_FR`, `fr_CA`, `de_CH` ; le même mot se traduit parfois différemment d'une région à l'autre, et le format des nombres change (`1 234,50` en France, `1'234.50` en Suisse).

## 1.2 Ce qui se localise

| Élément | Exemple | Outil |
|---|---|---|
| Messages de l'interface | « Bonjour, le monde ! » | gettext, Babel |
| Pluriels | 1 fichier / 3 fichiers ; l'arabe a six formes | `ngettext`, règles CLDR |
| Contexte | « Ouvrir » (menu) et « Ouvert » (état) partagent l'anglais *Open* | `pgettext` |
| Dates, heures, fuseaux | 29 août 2026 / August 29, 2026 | `babel.dates` |
| Nombres et monnaies | 1 234,50 € / €1,234.50 | `babel.numbers` |
| Tri et majuscules | é, ß, İ | `locale`, PyICU |
| Direction d'écriture | arabe, hébreu | CSS `dir="rtl"` |

## 1.3 Formats de catalogues

- **gettext** (`.pot` gabarit, `.po` traduction, `.mo` binaire) : le standard du monde Unix, de Python, de Django, de Pyramid et des traducteurs professionnels ; c'est le format de ce cours.
- **JSON / ICU MessageFormat** : monde JavaScript (i18next, FormatJS), messages avec pluriels et genre intégrés.
- **XLIFF** : format d'échange XML entre outils de traduction ; **Fluent** (Mozilla) et **ARB** (Flutter) sont propres à leurs écosystèmes.

Les données de référence (formats, noms de mois, règles de pluriel) viennent du **CLDR** (Unicode) ; Babel les embarque, PyICU expose la bibliothèque ICU complète.

# 2. gettext : la chaîne de traduction

## 2.1 Marquer les messages dans le code

```python
"""Exemple gettext pur : messages, pluriels, contexte."""
import gettext
from pathlib import Path

LOCALE_DIR = Path(__file__).parent / "locale"
traduction = gettext.translation("salut", localedir=LOCALE_DIR, fallback=True)
_ = traduction.gettext
ngettext = traduction.ngettext
pgettext = traduction.pgettext                      # Python >= 3.8

def rapport(nb_fichiers: int) -> str:
    return ngettext("{n} file processed", "{n} files processed", nb_fichiers).format(n=nb_fichiers)

if __name__ == "__main__":
    print(_("Hello, world!"))
    print(rapport(1))
    print(rapport(3))
    print(pgettext("menu", "Open"))
```

Règles : le message source est en anglais (langue pivot des traducteurs), la chaîne passée à `_()` est un **littéral** (l'extracteur ne peut pas suivre une variable), les paramètres sont nommés (`{n}`) pour que le traducteur puisse les déplacer, et le pluriel passe par `ngettext` avec le nombre, jamais par une concaténation.

`fallback=True` renvoie le texte source si aucun catalogue n'existe : l'application fonctionne avant d'être traduite.

## 2.2 Extraire : le fichier `.pot`

```bash
sudo apt install gettext
mkdir -p locale
xgettext --language=Python --from-code=UTF-8 \
    --keyword=_ --keyword=ngettext:1,2 --keyword=pgettext:1c,2 \
    -o locale/salut.pot salut.py            # ou : $(find . -name "*.py")
```

Le `.pot` (*Portable Object Template*) contient un en-tête (un `msgid ""` dont le `msgstr` porte les métadonnées) puis, pour chaque message, l'emplacement source, le `msgid`, éventuellement `msgid_plural` et `msgctxt`, et un `msgstr` vide :

```po
#: salut.py:12
#, python-brace-format
msgid "{n} file processed"
msgid_plural "{n} files processed"
msgstr[0] ""
msgstr[1] ""

#: salut.py:18
msgctxt "menu"
msgid "Open"
msgstr ""
```

Un `msgstr` multiligne s'écrit `msgstr ""` suivi de lignes entre guillemets terminées par `\n`. Le nom du fichier, `salut`, est le **domaine** : il doit correspondre au premier argument de `gettext.translation`.

## 2.3 Créer et traduire : le fichier `.po`

Les catalogues vivent dans `locale/<langue>/LC_MESSAGES/<domaine>.po` ; `LC_MESSAGES` est imposé par gettext (les autres catégories `LC_*` concernent les formats, la monnaie, le tri).

```bash
mkdir -p locale/fr/LC_MESSAGES
msginit --no-translator -i locale/salut.pot -o locale/fr/LC_MESSAGES/salut.po -l fr_FR.UTF-8
```

`msginit` renseigne l'en-tête, dont la règle de pluriel (`Plural-Forms: nplurals=2; plural=(n > 1);` pour le français). Vérifier que `Content-Type` indique `charset=UTF-8` — sur une machine sans locale française installée, `msginit` peut écrire `ASCII`, et `msgfmt` refusera ensuite les accents.

Le traducteur remplit les `msgstr` (éditeur de texte, Poedit, Weblate…) :

```po
msgid "Hello, world!"
msgstr "Bonjour, le monde !"

msgid "{n} file processed"
msgid_plural "{n} files processed"
msgstr[0] "{n} fichier traité"
msgstr[1] "{n} fichiers traités"

msgctxt "menu"
msgid "Open"
msgstr "Ouvrir"
```

## 2.4 Mettre à jour

Quand le code change, on régénère le `.pot` puis on fusionne dans chaque `.po` sans perdre les traductions existantes :

```bash
xgettext --language=Python --from-code=UTF-8 --keyword=_ --keyword=ngettext:1,2 --keyword=pgettext:1c,2 \
    -o locale/salut.pot salut.py
msgmerge --update --backup=none locale/fr/LC_MESSAGES/salut.po locale/salut.pot
```

Les messages modifiés reçoivent la marque `#, fuzzy` : la traduction est conservée à titre indicatif mais **ignorée à la compilation** tant que le traducteur ne l'a pas validée et retiré la marque. Les messages disparus sont conservés en commentaires `#~`.

## 2.5 Compiler et exécuter

```bash
msgfmt --check --statistics -o locale/fr/LC_MESSAGES/salut.mo locale/fr/LC_MESSAGES/salut.po
LANGUAGE=fr python salut.py
```

`--check` valide la syntaxe, les paramètres et l'en-tête ; `--statistics` compte les messages traduits. Le `.mo` (*Machine Object*) est la forme binaire lue à l'exécution ; il se régénère à chaque changement et ne se modifie jamais à la main. Sortie obtenue avec `LANGUAGE=fr` :

```text
Bonjour, le monde !
1 fichier traité
3 fichiers traités
Ouvrir
```

Python choisit la langue selon les variables `LANGUAGE`, `LC_ALL`, `LC_MESSAGES`, `LANG` (dans cet ordre) ; `gettext.translation(..., languages=["fr"])` force un choix explicite, ce qu'une application Web doit faire par requête.

# 3. Babel : catalogues et formatage selon la locale

**Babel** est la bibliothèque Python de référence pour l'internationalisation : elle remplace les outils GNU gettext par des commandes `pybabel` portables (utiles sous Windows et dans les scripts de construction) et fournit le formatage localisé grâce aux données CLDR.

## 3.1 Les commandes `pybabel`

```bash
python -m pip install Babel
pybabel extract -F babel.cfg -k _ -k "ngettext:1,2" -k "pgettext:1c,2" -o locale/messages.pot .
pybabel init    -i locale/messages.pot -d locale -l fr        # première fois
pybabel update  -i locale/messages.pot -d locale             # mises à jour, toutes les langues
pybabel compile -d locale --statistics                        # .po -> .mo
```

Le fichier `babel.cfg` décrit quoi extraire et avec quel extracteur :

```ini
[python: **.py]
[jinja2: templates/**.html]
extensions = jinja2.ext.i18n
```

Babel fournit les extracteurs `python` et `javascript` ; Jinja2 apporte le sien. Pour Chameleon, voir le chapitre 4. L'option `-D <domaine>` des sous-commandes fixe le nom du catalogue (`messages` par défaut).

## 3.2 Formater selon la locale

```python
from datetime import date
from babel import Locale
from babel.dates import format_date
from babel.numbers import format_currency, format_decimal

print(format_date(date(2026, 8, 29), format="long", locale="fr_FR"))   # 29 août 2026
print(format_date(date(2026, 8, 29), format="long", locale="en_US"))   # August 29, 2026
print(format_currency(1234.5, "EUR", locale="fr_FR"))                 # 1 234,50 €
print(format_currency(1234.5, "EUR", locale="en_US"))                 # €1,234.50
print(format_decimal(1234567.891, locale="de_DE"))                    # 1.234.567,891
print(Locale("ar").plural_form(11))                                    # many
```

Les espaces dans `1 234,50 €` sont des espaces insécables fines : ne jamais reconstituer ces formats à la main. Babel connaît aussi les noms de langues et de pays, les fuseaux (`format_datetime`, `get_timezone_name`), les unités et les listes (`format_list`).

# 4. Pyramid et Chameleon : intégration complète

Pyramid intègre gettext nativement (`pyramid.i18n`) ; il n'y a **aucun paquet d'intégration à installer**. Le flux ci-dessous a été exécuté avec Pyramid 2.1, `pyramid_chameleon`, Babel 2.18 et lingua 4.16.

## 4.1 Configuration

```python
# monapp/__init__.py
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include("pyramid_chameleon")
    config.add_translation_dirs("monapp:locale/")      # catalogues monapp/locale/<lang>/LC_MESSAGES/monapp.mo
    config.add_route("accueil", "/")
    config.scan(".views")
    return config.make_wsgi_app()
```

Le réglage `pyramid.default_locale_name = fr` (fichier `.ini` ou `settings`) fixe la langue par défaut. Les catalogues de Deform et de Colander s'ajoutent de la même façon : `config.add_translation_dirs("deform:locale/", "colander:locale/")` (voir [[Deform]]).

## 4.2 Messages dans les vues

```python
# monapp/views.py
from pyramid.i18n import TranslationStringFactory
from pyramid.view import view_config

_ = TranslationStringFactory("monapp")               # domaine = nom du catalogue


@view_config(route_name="accueil", renderer="templates/accueil.pt")
def accueil(request):
    localizer = request.localizer
    message = localizer.translate(_("Hello, world!"))
    pluriel = localizer.pluralize("${n} file", "${n} files", 3, domain="monapp", mapping={"n": 3})
    return {"message": message, "pluriel": pluriel}
```

`_()` ne traduit pas : il crée un `TranslationString` qui porte le message, son domaine et ses paramètres ; la traduction n'a lieu qu'au moment où `request.localizer` la demande, dans la langue de la requête. Les paramètres s'écrivent `${n}` (syntaxe Pyramid), pas `{n}`.

## 4.3 Messages dans les gabarits Chameleon

```html
<html xmlns:i18n="http://xml.zope.org/namespaces/i18n" i18n:domain="monapp">
  <body>
    <h1 i18n:translate="">Welcome</h1>
    <p>${message}</p>
    <p>${pluriel}</p>
    <p i18n:translate="">Read the <a href="/doc" i18n:name="doc-link"><span i18n:translate="">documentation</span></a> first.</p>
  </body>
</html>
```

- `i18n:domain` sur la racine désigne le catalogue ;
- `i18n:translate=""` marque un élément à traduire ; le contenu est le message ;
- une balise **à l'intérieur** d'un message (lien, mise en forme) reçoit `i18n:name` : le traducteur voit `Read the ${doc-link} first.` et peut déplacer le lien sans toucher au HTML, ce qui répond à la question des messages multilignes contenant des balises ;
- les attributs se traduisent avec `i18n:attributes="title; alt"`.

Chameleon retire les sauts de ligne et les espaces superflus du message avant de chercher la traduction, ce qui autorise les messages écrits sur plusieurs lignes dans le gabarit.

## 4.4 Extraction et catalogues

Babel n'a pas d'extracteur Chameleon ; **lingua** (`pot-create`) extrait les gabarits `.pt` et le Python, mais n'accepte pas la forme `pluralize(..., domain=..., mapping=...)`. Le flux le plus simple combine les deux et fusionne avec `msgcat` :

```bash
python -m pip install Babel lingua
printf '[python: monapp/**.py]\n' > babel.cfg

pybabel extract -F babel.cfg -k _ -k "pluralize:1,2" -o /tmp/python.pot .   # Python, pluriels compris
pot-create -o /tmp/templates.pot monapp/templates/                          # gabarits Chameleon
msgcat --use-first /tmp/python.pot /tmp/templates.pot -o monapp/locale/monapp.pot

pybabel init   -i monapp/locale/monapp.pot -d monapp/locale -l fr -D monapp   # première fois
pybabel update -i monapp/locale/monapp.pot -d monapp/locale -D monapp         # ensuite
pybabel compile -d monapp/locale -D monapp --statistics
```

Le catalogue attendu est `monapp/locale/fr/LC_MESSAGES/monapp.po`, domaine `monapp` — le même nom que dans `TranslationStringFactory` et `i18n:domain`. Ces cinq commandes tiennent dans un `Makefile` (`make extract`, `make update`, `make compile`) ou une tâche du projet.

## 4.5 Choisir la langue de la requête

Par défaut, Pyramid lit le paramètre ou le cookie **`_LOCALE_`**, puis retombe sur `pyramid.default_locale_name`. Résultat obtenu sur l'application ci-dessus :

```text
GET /               -> Welcome / Hello, world! / 3 files / Read the documentation first.
GET /?_LOCALE_=fr   -> Bienvenue / Bonjour, le monde ! / 3 fichiers / Lisez d'abord la documentation.
```

Pour respecter l'en-tête `Accept-Language` du navigateur, on enregistre un **négociateur de locale** :

```python
# monapp/locale_negotiator.py
LANGUES = ("fr", "en")


def negocier(request):
    explicite = request.params.get("_LOCALE_") or request.cookies.get("_LOCALE_")
    if explicite in LANGUES:
        return explicite
    return request.accept_language.lookup(LANGUES, default="fr")
```

```python
config.set_locale_negotiator(negocier)
```

Une vue « changer de langue » pose le cookie `_LOCALE_` et redirige. La langue courante est disponible partout via `request.locale_name`, et Babel peut formater dates et nombres avec `request.locale_name` comme locale.

# 5. Bonnes pratiques

- **Messages complets** : une phrase par message ; jamais de concaténation (`_("Hello") + name`), l'ordre des mots change d'une langue à l'autre. Paramètres nommés : `_("Hello ${name}", mapping={"name": name})`.
- **Pluriels** : toujours `ngettext`/`pluralize`, même quand l'anglais ne varie pas ; la règle de pluriel du catalogue (`Plural-Forms`) est celle de la langue cible, pas de la source.
- **Contexte** : `pgettext("menu", "Open")` lève l'ambiguïté des mots courts ; les commentaires `#. ` destinés aux traducteurs s'ajoutent avec `--add-comments=TRANSLATORS:` (xgettext) ou `-c TRANSLATORS:` (pybabel).
- **Ne pas traduire ce qui se formate** : dates, nombres, monnaies et listes passent par Babel, pas par des messages.
- **Marquage exhaustif** : tout texte visible, y compris les messages d'erreur, les attributs `alt`/`title` et les libellés de formulaires (Deform traduit ses widgets, pas vos libellés).
- **Qualité** : `msgfmt --check`, `pybabel compile --statistics` en intégration continue, échec si un catalogue a des messages non traduits ou `fuzzy` en production ; relecture par un locuteur natif ; **pseudo-localisation** (catalogue qui allonge et accentue chaque message) pour repérer les chaînes oubliées et les mises en page trop étroites.
- **Organisation** : un domaine par application, un dépôt de catalogues versionné, une plateforme de traduction (Weblate, Transifex, Crowdin) quand les traducteurs ne sont pas développeurs ; les `.mo` sont des artefacts de construction, à ne pas commiter.
- **Encodage** : UTF-8 partout, `charset=UTF-8` dans chaque `.po`.

# 6. Aide-mémoire

| Besoin | Commande |
|---|---|
| Extraire (gettext) | `xgettext --language=Python --from-code=UTF-8 -k_ -kngettext:1,2 -kpgettext:1c,2 -o locale/dom.pot fichiers.py` |
| Extraire (Babel) | `pybabel extract -F babel.cfg -k _ -k "ngettext:1,2" -o locale/messages.pot .` |
| Extraire (Chameleon) | `pot-create -o templates.pot monapp/templates/` puis `msgcat` |
| Créer une langue | `msginit -i dom.pot -o fr/LC_MESSAGES/dom.po -l fr_FR.UTF-8` ou `pybabel init -l fr` |
| Mettre à jour | `msgmerge --update` ou `pybabel update` |
| Compiler | `msgfmt --check --statistics` ou `pybabel compile --statistics` |
| Tester une langue | `LANGUAGE=fr python app.py` ; en Pyramid `?_LOCALE_=fr` |
| Formater | `format_date`, `format_currency`, `format_decimal` (Babel) |
| Pyramid | `TranslationStringFactory`, `config.add_translation_dirs`, `request.localizer`, `config.set_locale_negotiator` |

# 7. Sources

- Documentation Python, module `gettext` : <https://docs.python.org/3/library/gettext.html>
- GNU gettext : <https://www.gnu.org/software/gettext/manual/>
- Babel : <https://babel.pocoo.org/>
- lingua : <https://github.com/wichert/lingua>
- Pyramid, chapitre « Internationalization and Localization » : <https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/i18n.html>
- Chameleon, attributs `i18n:` : <https://chameleon.readthedocs.io/en/latest/reference.html#i18n>
- Unicode CLDR : <https://cldr.unicode.org/>
- Weblate : <https://weblate.org/>
