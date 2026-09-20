---
schema_version: 1
uid: 01M02EX5CCGYQZBX1P5GJCGVT4
titre: Standards modernes de déploiement Plone et Zope
aliases:
  - plone-modern-tooling
  - mxdev
  - pyproject-plone
  - cookiecutter-zope-instance
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - deploiement
  - plone
  - zope
resume: "Cours sur l'assemblage des projets Plone et Zope avec venv, pip ou uv, pyproject.toml, mxdev et cookiecutter-zope-instance : contraintes, configuration WSGI, automatisation, maintenance ZODB et migration progressive depuis Buildout."
niveau: intermediaire
prerequis:
  - "[[Python]]"
  - "[[git]]"
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2026-09-20
date_modification: 2026-09-20
date_verification: 2026-09-20
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---

# Standards modernes de déploiement Plone et Zope

> [!abstract] Objectif
> Comprendre les responsabilités autrefois réunies dans `zc.buildout`, puis assembler un environnement Plone avec les outils Python actuels. Savoir distinguer installation des paquets, configuration de Zope, démarrage WSGI, développement multi-dépôts et maintenance de la ZODB.

Voir aussi : [[Python]], [[git]], [[zc.buildout]].

> [!info] Périmètre et date de vérification
> Révision documentaire du **20 septembre 2026**. Le cours porte sur le **backend Python** de Plone et Zope ; un frontend Volto exige un environnement JavaScript distinct. Les huit chapitres du document d'origine sont conservés. Les configurations proposées sont des **exemples pédagogiques rédigés pour ce cours**, et non la reproduction d'un modèle officiel complet.
>
> Le socle des exemples reste **Plone 6.2.1**, dont les contraintes ont été consultées. **Plone 6.2.2 est déjà publié sur PyPI** : dépôt des fichiers le 10 septembre 2026, journal des changements daté du 11 septembre. Le maintien de 6.2.1 ici ne constitue donc pas une recommandation de dernière version pour la production. Les contraintes de 6.2.2 n'ont pas pu être inspectées dans cette vérification. [^plone-pypi] [^contraintes]

> [!warning] Métadonnée à confirmer
> La date de création `2026-10-20` provient du fichier fourni et est postérieure à cette révision. Elle est conservée pour ne pas inventer l'historique de la note ; `metadata_verifiees` reste donc à `false`. Les dates de modification et de vérification correspondent à la présente révision.

## Résumé

Buildout regroupe des fonctions différentes : installer des distributions Python, gérer des sources de développement et exécuter des recettes de configuration ou de déploiement. L'approche présentée ici sépare ces fonctions. Elle facilite notamment l'utilisation d'un environnement Python habituel par les IDE et les outils de test. **Buildout reste toutefois un logiciel libre et une option documentée**, non une technologie « propriétaire » interdite aux nouveaux projets. [^comparaison]

Dans cette organisation, `venv` isole les distributions Python, `pip` ou `uv` les installe, `pyproject.toml` décrit les paquets, `mxdev` prépare les sources et les contraintes, et `cookiecutter-zope-instance` génère la configuration. Un `Makefile` rend les opérations répétables pour l'équipe. Il ne remplace ni le gestionnaire de processus, ni la procédure de sauvegarde.

### État vérifié de l'écosystème

| Élément | État observé au 20 septembre 2026 | Conséquence pour le cours |
| --- | --- | --- |
| Plone | Publication 6.2.2 sur PyPI ; exemples fondés sur 6.2.1. | Ne pas associer une version de Plone aux contraintes d'une autre version. |
| Python | Plone 6.2 annonce le support de Python **3.10 à 3.14**. | La compatibilité des extensions doit être vérifiée séparément. |
| Socle 6.2.1 | Contraintes consultées : `Zope==6.1`, `pip==26.1.2`, `setuptools==81.0.0`, `wheel==0.47.0`, `zc.buildout==5.2.0`. | Ce sont les versions du fichier de cette publication, pas des recommandations intemporelles. |
| Générateur d'instance | Tag `cookiecutter-zope-instance` **3.1.0**, publié le 18 juin 2026. | Le modèle de configuration est explicitement épinglé. |
| Outils de génération | `cookiecutter==2.7.1` et `mxdev==5.4.1`. | Ils sont isolés du serveur dans un environnement séparé. |

Sources de ce tableau : notes de publication, contraintes, PyPI et publication du modèle. [^release] [^contraintes] [^plone-pypi] [^template-version] [^cookiecutter] [^mxdev]

Un support annoncé du cœur de Plone ne démontre pas la compatibilité de tous les add-ons, pilotes SQL, modules C et outils de migration. De même, il n'affirme pas une prise en charge indistincte de tous les interpréteurs ou modes de compilation Python.

## Plan du cours

1. Dépasser le paradigme Buildout sans confondre les responsabilités.
2. Générer la configuration d'une instance avec Cookiecutter.
3. Gérer les dépendances et les sources avec `mxdev`.
4. Décrire et distribuer un add-on avec `pyproject.toml`.
5. Piloter le projet avec un `Makefile` explicite.
6. Diagnostiquer et maintenir une ZODB.
7. Migrer progressivement depuis Buildout.
8. Retenir les principes et valider les acquis.

---

## 1. Dépasser le paradigme Buildout

### 1.1 Ce qui change réellement

Dans une installation Buildout, les scripts générés peuvent définir un chemin d'import spécifique à partir des distributions choisies. Avec un environnement virtuel, les commandes et l'interpréteur utilisent les distributions installées dans cet environnement. La transition simplifie ainsi certains raccordements avec les outils Python, mais elle oblige à reprendre explicitement les services rendus par les anciennes recettes. [^comparaison]

| Responsabilité | Exemple avec Buildout | Organisation proposée |
| --- | --- | --- |
| Installation des distributions | Sections `eggs`, recettes, cache d'eggs. | `.venv` et `pip install` ou `uv pip install`. |
| Métadonnées d'un paquet | `setup.py` ou `setup.cfg`, indépendamment de Buildout. | Métadonnées déclaratives dans `pyproject.toml`. |
| Sélection de versions | `[versions]`, fichiers `versions.cfg`. | Contraintes, puis verrouillage adapté au déploiement. |
| Sources de développement | `mr.developer`, section `[sources]`. | `mxdev`, ou simples chemins éditables lorsque cela suffit. |
| Configuration de l'instance | `plone.recipe.zope2instance`. | `cookiecutter-zope-instance`, ou autre génération maîtrisée. |
| Commandes applicatives | `bin/instance`, `bin/test`. | Points d'entrée dans `.venv/bin/`, regroupés par un Makefile. |
| Services système et outils natifs | Recettes particulières ou scripts externes. | Conteneur, paquets système, gestion de configuration ou scripts dédiés. |

Cette correspondance est **fonctionnelle**, pas une conversion mécanique de toutes les recettes.

### 1.2 Standards Python et conventions de projet

`pyproject.toml` accueille plusieurs responsabilités : le système de construction relève des PEP 517/518 ; les métadonnées de `[project]`, de la PEP 621 ; le mode éditable moderne, de la PEP 660. Les namespaces implicites relèvent de la PEP 420. Ces standards ne prescrivent pas le contenu d'un `Makefile` ou les options d'un fichier `mxdev.ini`. [^pyproject] [^pep660] [^pep420]

L'absence de Buildout n'interdit pas `setup.py` : un paquet peut encore en avoir besoin pour une construction particulière. L'objectif est d'éviter la duplication et de réserver la logique de construction au bon endroit, non de supprimer un fichier par principe.

Un `venv` n'isole ni les processus, ni le réseau, ni les bibliothèques système comme le ferait un dispositif de confinement. Les wheels limitent certaines compilations, sans être toutes universelles : leurs balises peuvent dépendre de Python, de l'ABI et de la plateforme. [^venv] [^wheel]

---

## 2. Générer la configuration d'une instance avec Cookiecutter

### 2.1 Deux générateurs à ne pas confondre

**Cookieplone** sert à créer un projet Plone et ses fichiers de travail. La documentation présente notamment cette entrée :

```bash
uvx cookieplone project
```

Le générateur propose les options du projet, notamment son frontend. Cette commande utilise l'outillage disponible au moment de son exécution ; pour une création reproductible, enregistrer ensuite les versions et les réponses utilisées. [^cookieplone]

**`cookiecutter-zope-instance`**, en revanche, génère principalement la **configuration d'une instance Zope** : il ne crée pas à lui seul le paquet métier, les fichiers de dépendances et le Makefile décrits dans ce cours. Il ne choisit pas la version de Plone à installer. Cette version dépend des exigences et contraintes fournies à l'installateur. [^template-tutoriel]

### 2.2 Arborescence de l'exemple suivi

L'arborescence ci-dessous est **assemblée dans ce cours**. Seule sa partie `instance/` est produite par le modèle d'instance ; les données de travail sont explicitement placées à part.

```text
mon-projet/
├── Makefile
├── pyproject.toml
├── MANIFEST.in
├── README.md
├── constraints.txt
├── requirements.txt
├── requirements-dev.txt
├── requirements-tools.txt
├── mxdev.ini
├── instance.example.yaml       # exemple versionné, sans secret réel
├── instance.yaml               # configuration locale confidentielle
├── src/
│   └── mon/                    # namespace implicite : pas de __init__.py ici
│       └── addon/
│           ├── __init__.py
│           ├── configure.zcml
│           └── tests/
├── scripts/
│   └── inspect_root.py
├── sources/                    # éventuels dépôts externes de développement
├── instance/                   # configuration générée
│   ├── etc/
│   │   ├── zope.conf
│   │   ├── zope.ini
│   │   └── site.zcml
│   └── inituser                # initialisation éventuelle du premier compte
├── var/                        # données persistantes ; jamais « nettoyées »
│   ├── filestorage/
│   └── blobstorage/
├── .venv/                      # serveur, paquet métier et tests
└── .venv-tools/                # mxdev et Cookiecutter
```

### 2.3 Décrire l'instance sans embarquer les secrets dans Git

**Fichier `instance.example.yaml` :**

```yaml
default_context:
  target: instance
  location_clienthome: var
  wsgi_listen: "127.0.0.1:8080"
  initial_user_name: admin
  initial_user_password: "REMPLACER_AVANT_USAGE"
  debug_mode: false
  verbose_security: false
  zcml_package_includes: "mon.addon"
  db_storage: direct
  db_filestorage_location: var/filestorage/Data.fs
  db_blob_location: var/blobstorage
```

Les clés de cet exemple correspondent au modèle 3.1.0. `target` désigne la sortie de configuration ; `location_clienthome` le répertoire de travail de Zope ; les emplacements de `Data.fs` et des blobs sont ici explicites. `zcml_package_includes` contient des noms de paquets séparés par des virgules et charge leur `configure.zcml`. [^template-config] [^zcml]

Préparer la configuration locale :

```bash
cp instance.example.yaml instance.yaml
chmod 600 instance.yaml
# Éditer instance.yaml et remplacer REMPLACER_AVANT_USAGE.
```

Le compte initial sert au premier démarrage d'une base vierge ; changer cette valeur dans le YAML ne réinitialise pas automatiquement le mot de passe d'un compte déjà enregistré dans la base. [^zope-operation]

Après installation de l'outillage, la commande de génération est :

```bash
.venv-tools/bin/cookiecutter -f --no-input \
  --config-file instance.yaml --checkout 3.1.0 \
  gh:plone/cookiecutter-zope-instance
```

`-f` autorise le remplacement des fichiers générés. Cookiecutter peut donc être réutilisé après une modification de configuration : il n'est pas limité à une exécution initiale. Conserver le YAML comme source de vérité, examiner les différences de configuration avant un déploiement, puis redémarrer le service. [^template-workflow]

> [!warning] Secrets et chemins
> Ne pas versionner `instance.yaml`, `inituser`, les données ou les copies de configuration contenant des secrets. Cookiecutter peut aussi conserver les réponses dans ses fichiers de replay : protéger ces fichiers dans le répertoire utilisateur. La génération d'un modèle peut exécuter des hooks Python ; utiliser un dépôt de confiance et une révision maîtrisée. [^cookiecutter-replay] [^cookiecutter-hooks]
>
> Des chemins absolus peuvent être inscrits dans la configuration générée. Si le projet est déplacé, régénérer ou contrôler ces chemins avant tout redémarrage.

### 2.4 Les trois niveaux de configuration

| Fichier | Rôle |
| --- | --- |
| `instance/etc/zope.ini` | Pipeline WSGI, serveur HTTP et journalisation. |
| `instance/etc/zope.conf` | Configuration Zope et de ses stockages ; ce fichier n'a pas disparu. |
| `instance/etc/site.zcml` | Chargement de la configuration des composants et extensions. |

Le fichier INI référence la configuration Zope ; il ne la remplace pas. Le lancement documenté est **`runwsgi`**, sans tiret :

```bash
.venv/bin/runwsgi -v instance/etc/zope.ini
```

Le processus reste au premier plan. Le mode debug exige une option distincte : `-d`, par exemple `runwsgi -dv ...`. [^zope-operation]

Le démarrage de Zope ne suffit pas non plus à créer un site Plone. Pour l'exemple local avec interface classique, ouvrir `http://127.0.0.1:8080`, s'authentifier et créer le site avec l'interface d'administration. Un déploiement Volto nécessite en plus le frontend et sa configuration. [^install-pip]

---

## 3. Gestion des dépendances et sources Git avec mxdev

### 3.1 Exigences, contraintes et verrouillage

Une **exigence** demande l'installation d'un paquet. Une **contrainte** restreint les versions admissibles d'un paquet s'il est nécessaire à l'installation. Ainsi, `Plone==6.2.1` dans un fichier utilisé uniquement avec `-c` ne demande pas, à lui seul, l'installation de Plone. [^pip]

Un fichier de contraintes officiel n'est donc pas, à lui seul, un verrouillage exhaustif du projet. Il ne décrit pas nécessairement tous les add-ons, les outils externes, les révisions Git, les dépendances de construction et les empreintes des distributions réellement utilisées. Il faut encore définir ce qui sera installé, pour quelle plateforme et selon quelle procédure.

Les fichiers de notre exemple séparent le socle, le développement et l'outillage :

**Fichier `constraints.txt` :**

```text
# Socle pédagogique vérifié ; ne pas confondre avec la dernière publication.
-c https://dist.plone.org/release/6.2.1/constraints.txt
```

**Fichier `requirements.txt` :**

```text
-c constraints.txt
Plone
Zope[wsgi]
```

**Fichier `requirements-dev.txt` :**

```text
-r requirements.txt
-e .[test]
```

**Fichier `requirements-tools.txt` :**

```text
# Outils de génération, isolés du serveur dans .venv-tools.
cookiecutter==2.7.1
mxdev==5.4.1
```

`Zope[wsgi]` demande explicitement les dépendances WSGI de Zope. L'installation éditable `-e .[test]` concerne le paquet présent à la racine du projet, avec ses dépendances de test. Les outils de génération sont volontairement séparés des dépendances du serveur pour limiter les conflits entre leurs contraintes respectives.

### 3.2 Rôle exact de mxdev

`mxdev` prépare des sources de développement et réécrit des exigences/contraintes pour les rendre utilisables par l'installateur. **Il n'exécute pas lui-même `pip` et n'est pas son solveur de dépendances.** Son fichier de configuration par défaut est `mx.ini` ; le nom `mxdev.ini` fonctionne lorsqu'on le fournit explicitement avec `-c`. [^mxdev]

**Fichier `mxdev.ini` :**

```ini
[settings]
requirements-in = requirements-dev.txt
requirements-out = requirements-mxdev.txt
constraints-out = constraints-mxdev.txt
default-target = sources
default-update = false
```

La convention `sources/` évite de mélanger les checkouts externes avec `src/`, qui contient le paquet du projet. `default-update = false` évite la mise à jour automatique des checkouts déjà présents : la décision de mise à jour devient explicite. Cela ne transforme pas une branche Git en référence immuable.

```bash
.venv-tools/bin/mxdev -c mxdev.ini
.venv/bin/python -m pip install -r requirements-mxdev.txt
```

Dans le fonctionnement documenté, le fichier d'exigences généré référence le fichier de contraintes généré. Lui repasser en plus les contraintes **originales** peut réintroduire précisément les versions que l'on voulait remplacer. [^mxdev]

```text
requirements-dev.txt ──► requirements.txt ──► constraints.txt
          │                                      │
          └─────────────────┬────────────────────┘
                        mxdev.ini
                            │
                            ▼
                 préparation par mxdev
                   │                  │
                   ▼                  ▼
        requirements-mxdev.txt   constraints-mxdev.txt
                   │                  ▲
                   └──── référence ───┘
                            │
                            ▼
                     pip ou uv pip
```

Le nom de cible Make `resolve` utilisé plus loin est une convention de projet : cette étape prépare les entrées, tandis que la résolution finale appartient à l'installateur.

### 3.3 Ajouter un dépôt Git de développement

Exemple **à adapter à un dépôt réel**, non nécessaire pour exécuter le squelette du cours :

```ini
[collective.exemple]
url = https://github.com/mon-organisation/collective.exemple.git
branch = main
extras = test
```

En développement, le mode éditable permet de modifier le code sans réinstaller chaque changement Python. En revanche, un changement de métadonnées, de dépendances ou de fichiers de configuration peut exiger une réinstallation ou un redémarrage. Ne pas assimiler `mxdev` à un observateur automatique des changements de branche. [^mxdev]

### 3.4 Utiliser un paquet déjà présent sur le disque

Pour un répertoire local, la solution la plus simple est une exigence éditable dans `requirements-dev.txt` :

```text
-e ../mon.autre.package
```

Si une contrainte amont épingle ce paquet et bloque le remplacement, configurer intentionnellement son exclusion dans les réglages `mxdev` :

```ini
[settings]
# Conserver aussi les autres réglages de l'exemple.
ignores = mon.autre.package
```

Autre possibilité documentée, pour un répertoire déjà présent dans `sources/mon.autre.package` :

```ini
[mon.autre.package]
vcs = fs
url = mon.autre.package
```

Avec `vcs = fs`, le répertoire ou lien symbolique doit déjà exister à l'emplacement attendu ; `mxdev` ne le télécharge pas. La clé générique `path = ...` du document initial n'est pas la bonne configuration pour cet usage. [^mxdev]

### 3.5 Surcharges, conflits et traçabilité

Deux contraintes incompatibles ne s'annulent pas par « priorité du dernier fichier ». Exemple : demander simultanément `paquet==1.0` et `paquet==2.0` ne constitue pas une surcharge valide pour pip. [^pip]

Une surcharge `mxdev` peut retirer ou remplacer une contrainte ; elle ne prouve pas que le nouveau paquet respecte les dépendances et le comportement attendus par Plone. Pour chaque écart au socle, documenter son motif, les tests réalisés et la condition de retrait.

Pour une livraison, une branche comme `main` reste mutable, même sans installation éditable. Préférer un commit identifié ou, mieux pour les paquets livrés, des distributions construites et conservées. Enregistrer les révisions des sources et le résultat de la résolution dans la chaîne de livraison.

---

## 4. Configuration déclarative avec pyproject.toml

### 4.1 Décrire le paquet, pas toute l'infrastructure

Le `pyproject.toml` décrit ici **l'extension `mon.addon`**. Il n'installe pas PostgreSQL, ne crée pas le site Plone et ne configure pas le reverse proxy. La plage Plone indiquée représente le périmètre de cet exemple ; les versions exactes de l'environnement restent pilotées par les contraintes. [^pyproject]

**Fichier `pyproject.toml` :**

```toml
[build-system]
requires = ["setuptools>=77.0.3"]
build-backend = "setuptools.build_meta"

[project]
name = "mon.addon"
version = "1.0.0.dev0"
description = "Exemple pédagogique d'extension pour Plone 6.2"
readme = "README.md"
requires-python = ">=3.10"
dependencies = [
    "Plone>=6.2,<6.3",
    "plone.api",
]

[project.optional-dependencies]
test = [
    "zope.testrunner>=8",
    "plone.app.testing",
]

[tool.setuptools]
package-dir = {"" = "src"}
include-package-data = true

[tool.setuptools.packages.find]
where = ["src"]
include = ["mon.*"]
namespaces = true
```

Les dépendances de `[build-system]` servent à **construire** la distribution ; celles de `[project.dependencies]` servent à **exécuter** le paquet. Il n'y a pas lieu d'ajouter `setuptools` comme dépendance d'exécution si le code n'en a pas besoin. Les outils de test sont déclarés dans l'extra `test`, qui installe notamment le runner utilisé par le Makefile. [^pyproject]

Cet exemple ne fixe pas de licence arbitraire : ajouter les informations de licence et les fichiers correspondants selon les droits réellement applicables au projet.

### 4.2 Namespaces : supprimer seulement ce qui doit l'être

Dans notre paquet, `mon` est un namespace implicite : **`src/mon/__init__.py` est absent**. En revanche, `src/mon/addon/__init__.py` est conservé car `mon.addon` est un paquet ordinaire. On ne supprime donc pas récursivement tous les fichiers `__init__.py`. [^pep420]

Plone 6.2 migre ses namespaces vers ce modèle. Pour un projet existant, contrôler toutes les distributions partageant un namespace, leur mode d'installation et les usages de `pkg_resources`. Une migration de namespace isolée peut révéler des incompatibilités avec des distributions plus anciennes ; les guides Plone détaillent cette transition. [^upgrade62] [^namespace]

### 4.3 Inclure les fichiers non Python

Le squelette utilise les fichiers suivants :

**Fichier `src/mon/addon/__init__.py` :**

```python
"""Extension minimale utilisée dans le cours sur l'outillage Plone."""
```

**Fichier `src/mon/addon/configure.zcml` :**

```xml
<configure xmlns="http://namespaces.zope.org/zope">
  <!-- Ajouter ici les déclarations ZCML de l'extension. -->
</configure>
```

**Fichier `MANIFEST.in` :**

```text
include README.md
recursive-include src/mon/addon *.zcml *.xml *.pt *.po *.mo *.css *.js *.svg *.png
```

Le `README.md` référencé par `[project]` doit également exister. Par exemple :

```markdown
# mon.addon

Extension pédagogique pour expérimenter l'outillage Plone 6.2.
```

La liste de ressources de `MANIFEST.in` correspond à un point de départ : ajouter les formats réellement utilisés par le projet. Avec `include-package-data = true`, il faut ensuite contrôler les fichiers effectivement présents dans la distribution. **Un add-on qui fonctionne en mode éditable peut échouer une fois installé depuis sa wheel si des ZCML, profils XML ou templates manquent.** [^datafiles]

Ce squelette charge un `configure.zcml` vide. Il n'enregistre pas encore de profil GenericSetup et n'apparaît pas nécessairement comme une extension installable dans le panneau des modules complémentaires. Installer une distribution Python, charger son ZCML et appliquer son profil au site sont trois opérations distinctes.

### 4.4 Fournir des tests cohérents avec la commande annoncée

Créer `src/mon/addon/tests/__init__.py`, puis :

**Fichier `src/mon/addon/tests/test_packaging.py` :**

```python
from importlib.resources import files
import unittest

import mon.addon


class PackagingTests(unittest.TestCase):
    def test_import(self):
        self.assertEqual(mon.addon.__name__, "mon.addon")

    def test_zcml_is_packaged(self):
        resource = files("mon.addon").joinpath("configure.zcml")
        self.assertTrue(resource.is_file())
```

Ces deux tests vérifient les imports et la présence d'une ressource. **Ils ne démontrent pas le bon fonctionnement d'un site Plone**. Compléter le projet par des tests d'intégration avec `plone.app.testing`, des tests des profils de configuration et une recette des parcours utilisateurs.

Pour la livraison, exécuter également les tests sur le paquet construit et installé sans `-e`, dans un environnement séparé. Un test qui réinjecte systématiquement le répertoire `src/` ne vérifie pas ce que contient la wheel.

---

## 5. L'interface d'exploitation unifiée : le Makefile

### 5.1 Hypothèses de l'exemple

Le Makefile ci-dessous suppose **GNU Make, Bash, Git et un Python installé**, sur un environnement de type Unix. Les commandes sont exécutées depuis la racine du projet. Les lignes de recette commencent par de vraies tabulations.

Nous choisissons Python 3.12 pour l'exemple, sans en déduire qu'il s'agit du seul choix valable. `PYTHON` sélectionne l'interpréteur qui crée les environnements ; changer cette variable ne convertit pas un environnement déjà existant. Pour changer de version mineure de Python, reconstruire les environnements après arrêt des processus.

La séparation `.venv` / `.venv-tools` est une proposition d'organisation. Elle évite d'imposer aux outils de génération les versions de dépendances prévues pour le serveur. Les versions directes des outils sont épinglées, mais leurs dépendances transitives ne sont pas encore verrouillées exhaustivement.

### 5.2 Makefile de référence pour le cours

**Fichier `Makefile` :**

```makefile
SHELL := /bin/bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help
.DELETE_ON_ERROR:

PYTHON ?= python3.12
VENV := .venv
TOOLS := .venv-tools
TEMPLATE_VERSION := 3.1.0
SCRIPT ?=
export SCRIPT

.PHONY: help venv tools resolve install configure start dev console \
        run-script test check wheel clean clean-venv

help:
	@printf '%s\n' \
	  'make install      : installer le socle et le paquet éditable' \
	  'make configure    : régénérer les fichiers de configuration' \
	  'make start / dev  : démarrer au premier plan, sans/avec debug' \
	  'make test / check : tests et cohérence des dépendances' \
	  'make console      : ouvrir Zope (serveur arrêté en FileStorage)' \
	  'make run-script SCRIPT=scripts/inspect_root.py' \
	  'make wheel        : construire la wheel du paquet local' \
	  'make clean        : effacer seulement les sorties mxdev' \
	  'make clean-venv   : supprimer les deux environnements Python'

$(VENV)/bin/python:
	$(PYTHON) -m venv $(VENV)

$(VENV)/.bootstrap: $(VENV)/bin/python constraints.txt
	$(VENV)/bin/python -m pip install -c constraints.txt pip setuptools wheel
	touch $@

venv: $(VENV)/.bootstrap

$(TOOLS)/bin/python:
	$(PYTHON) -m venv $(TOOLS)

$(TOOLS)/.ready: $(TOOLS)/bin/python requirements-tools.txt
	$(TOOLS)/bin/python -m pip install -r requirements-tools.txt
	touch $@

tools: $(TOOLS)/.ready

resolve: tools
	$(TOOLS)/bin/mxdev -c mxdev.ini

install: venv resolve
	$(VENV)/bin/python -m pip install \
	  --build-constraint constraints-mxdev.txt -r requirements-mxdev.txt
	$(VENV)/bin/python -m pip check

configure: tools
	@test -f instance.yaml || { echo 'Créer instance.yaml depuis instance.example.yaml.'; exit 1; }
	@if grep -q 'REMPLACER_AVANT_USAGE' instance.yaml; then \
	  echo 'Définir un mot de passe initial propre à cette instance.'; exit 1; fi
	umask 0077; $(TOOLS)/bin/cookiecutter -f --no-input \
	  --config-file instance.yaml --checkout $(TEMPLATE_VERSION) \
	  gh:plone/cookiecutter-zope-instance

start:
	$(VENV)/bin/runwsgi -v instance/etc/zope.ini

dev:
	$(VENV)/bin/runwsgi -dv instance/etc/zope.ini

console:
	$(VENV)/bin/zconsole debug instance/etc/zope.conf

run-script:
	@test -n "$$SCRIPT" && test -f "$$SCRIPT" || { \
	  echo 'Usage : make run-script SCRIPT=chemin/vers/script.py'; exit 1; }
	$(VENV)/bin/zconsole run instance/etc/zope.conf "$$SCRIPT"

test:
	$(VENV)/bin/zope-testrunner --test-path=src -s mon.addon

check:
	$(VENV)/bin/python -m pip check

wheel: install
	$(VENV)/bin/python -m pip wheel --no-deps \
	  --build-constraint constraints-mxdev.txt --wheel-dir dist .

clean:
	rm -f -- requirements-mxdev.txt constraints-mxdev.txt

clean-venv:
	rm -rf -- .venv .venv-tools
```

Le fichier de contraintes 6.2.1 contient les versions de `pip`, `setuptools` et `wheel` utilisées au bootstrap. L'option **`--build-constraint`** nécessite pip 25.3 ou ultérieur ; le pip 26.1.2 de ce socle la prend en charge. Elle contraint les environnements de construction isolés : un simple `-c` sur l'installation ne joue pas automatiquement ce rôle. [^contraintes] [^pip]

`start`, `test` et `console` ne relancent pas implicitement une installation réseau. `configure` ne s'exécute pas à chaque démarrage. Cette séparation rend les effets des commandes plus prévisibles. En revanche, le Makefile ne détecte pas toutes les modifications d'une chaîne de contraintes distante : l'immuabilité et l'archivage des entrées relèvent de la procédure de livraison.

> [!warning] Portée du nettoyage
> `make clean` ne supprime que les deux fichiers produits par mxdev. `make clean-venv` supprime uniquement les environnements Python, après arrêt de leurs processus. **Ni `var/`, ni `sources/`, ni la base, ni ses blobs ne sont des caches de construction.** Aucune cible ne les efface.

### 5.3 Première exécution

Après création de tous les fichiers de l'exemple :

```bash
cp instance.example.yaml instance.yaml
chmod 600 instance.yaml
# Remplacer le mot de passe dans instance.yaml avant de poursuivre.

make install PYTHON=python3.12
make configure
make test
make start
```

Le premier plan s'arrête avec `Ctrl+C`. Il n'y a pas ici de fausse cible `stop` : l'arrêt d'un service en production appartient au superviseur. Après une modification des dépendances, relancer `make install` ; après une modification du YAML, exécuter explicitement `make configure`, puis redémarrer.

Pour limiter les erreurs de versionnement :

**Fichier `.gitignore` :**

```gitignore
.venv/
.venv-tools/
instance.yaml
instance/
var/
sources/
requirements-mxdev.txt
constraints-mxdev.txt
.mxdev_cache/
__pycache__/
*.py[cod]
*.egg-info/
build/
dist/
```

Ignorer `sources/` dans le dépôt principal ne dispense pas de conserver les modifications des add-ons dans leurs propres dépôts. De même, un `.gitignore` n'efface pas un secret qui a déjà été commité : traiter alors le secret comme exposé et le renouveler.

### 5.4 Que change uv ?

Deux approches sont possibles ; il faut choisir celle dont on maîtrise le modèle de résolution.

**Interface compatible avec les usages pip.** `uv pip` peut installer les exigences préparées par `mxdev`, en ciblant explicitement le bon environnement :

```bash
uv pip install --python .venv/bin/python -r requirements-mxdev.txt
uv pip check --python .venv/bin/python
```

Ce fragment remplace l'étape d'installation des exigences, pas toute la procédure. La politique de contraintes de construction doit également être transposée et validée avec la version de `uv` retenue. Les commandes, réglages et comportements de résolution de `uv` ne sont pas tous identiques à ceux de pip. [^uv-compat]

**Gestion de projet native.** Avec `uv lock` et `uv sync`, le verrouillage est porté par `uv.lock`. Les versions récentes de `mxdev` proposent une intégration optionnelle, installée via `mxdev[uv]`, pour alimenter la configuration du projet géré par uv. Ce mécanisme est distinct du simple traitement des fichiers `requirements-*.txt`. [^mxdev]

`uv sync` peut retirer les distributions qui n'appartiennent pas à l'environnement déclaré. Ne pas ajouter durablement des paquets par `pip install` à côté d'un projet synchronisé par uv en espérant qu'ils seront conservés. [^uv-sync]

### 5.5 Passer d'un environnement de développement à une livraison

Le projet ci-dessus est **reconstructible à partir d'instructions**, mais ne constitue pas encore une garantie de reconstruction identique : des index, dépendances non épinglées et sources distantes interviennent toujours.

Pour une livraison maîtrisée, compléter cette base par une chaîne de construction qui enregistre l'interpréteur et la plateforme, les contraintes et révisions exactes, les distributions construites, leurs empreintes et les résultats de tests. Construire les add-ons en wheels, tester l'installation sans éditables, puis déployer les artefacts validés plutôt que résoudre à nouveau sur le serveur.

`pip freeze` est utile pour inventorier un environnement ; il ne fournit pas à lui seul toutes les garanties d'un verrouillage reproductible. Une stratégie de fichiers résolus et d'empreintes, par exemple avec `uv pip compile`, doit être conçue pour les dépendances et plateformes visées. [^uv-compile]

En production, séparer également la configuration et les secrets des artefacts, placer les données sur un stockage persistant, utiliser un compte de service sans privilèges administrateur et un superviseur, désactiver le debug et traiter explicitement TLS, le proxy, les journaux et les sauvegardes. La conteneurisation ne remplace aucune de ces décisions. [^zope-operation] [^template-backup]

Un FileStorage direct sert un processus Zope ouvrant la base. Ne pas lancer plusieurs processus indépendants sur le même `Data.fs` pour augmenter la capacité ; étudier ZEO ou un stockage approprié et sa configuration. [^filestorage]

### 5.6 Diagnostic rapide

| Symptôme | Vérification prioritaire |
| --- | --- |
| `run-wsgi: command not found` | La commande s'appelle `runwsgi` ; vérifier l'environnement et l'installation WSGI. |
| `zope-testrunner` absent | L'extra `test` du paquet local a-t-il été installé dans `.venv` ? |
| Conflit après une surcharge | Les contraintes originales sont-elles réintroduites en plus des sorties mxdev ? |
| Paquet local introuvable | Nom de distribution, chemin éditable, arborescence `src/` et namespace. |
| Configuration manquante | `make configure` a-t-il été exécuté avec le bon répertoire courant ? |
| Ressource absente uniquement en production | Comparer le contenu de la wheel et celui du répertoire de développement. |
| Verrou de FileStorage | Chercher un serveur ou outil ouvrant déjà la base ; ne pas supprimer le verrou pour forcer l'accès. |

Les lignes de ce tableau synthétisent les vérifications développées dans les chapitres précédents ; elles ne remplacent pas l'analyse du message d'erreur complet.

---

## 6. Maintenance ZODB et outils de diagnostic

### 6.1 Trois contextes Python différents

| Contexte | Ce qui est disponible | Exemple |
| --- | --- | --- |
| Interpréteur du venv | Imports des distributions installées. | `.venv/bin/python` |
| Console Zope initialisée | Configuration de Zope et objet racine `app`. | `.venv/bin/zconsole debug instance/etc/zope.conf` |
| Travail sur un site Plone | Un site sélectionné, son contexte de composants et, selon l'opération, une requête et un contexte de sécurité adaptés. | Script applicatif ou test d'intégration préparé pour cette opération. |

L'interpréteur du venv remplace le besoin habituel d'un `zopepy` pour les imports, **pas l'initialisation d'une application Zope**. Ajouter IPython ne réalise pas cette initialisation non plus. La console applicative et les scripts Zope passent par `zconsole`. [^zope-operation]

### 6.2 Exécuter un script sans passer la configuration WSGI

Le chemin attendu par `zconsole` est celui de **`zope.conf`**, non celui de `zope.ini` :

```bash
.venv/bin/zconsole run instance/etc/zope.conf scripts/inspect_root.py
```

**Fichier `scripts/inspect_root.py` :**

```python
"""À exécuter avec zconsole run, sur une copie de travail de la base."""
import transaction


def inspect_root(root):
    """Affiche les identifiants sans valider de modification de la base."""
    try:
        print("Objets à la racine :", list(root.objectIds()))
    finally:
        transaction.abort()


# zconsole fournit l'objet racine « app » au script.
inspect_root(app)
```

Cet exemple affiche les identifiants racine puis abandonne la transaction. `transaction.abort()` évite de valider les modifications transactionnelles éventuellement réalisées dans cette session ; ce n'est pas un dispositif de confinement contre des effets externes du code.

En FileStorage direct, arrêter le serveur avant d'ouvrir la même base avec un second processus. Préférer une copie pour les explorations, scripts nouveaux et migrations. Un script métier modifiant des données doit définir explicitement ses contrôles, ses transactions et sa reprise sur erreur.

### 6.3 Vérifier les objets avec zodbverify

`zodbverify` aide à détecter les objets qui ne peuvent plus être chargés, notamment lorsque du code ou des classes ont disparu. Il ne garantit pas à lui seul la cohérence fonctionnelle de tout le site, ni la présence et la validité de tous les fichiers blobs. [^zodbverify]

Après ajout contrôlé de l'outil à un environnement de maintenance compatible avec le site :

```bash
.venv/bin/zodbverify -f /chemin/vers/une-copie/filestorage/Data.fs
```

Le module doit pouvoir importer les classes persistantes utilisées par la base. Un échec peut donc relever de l'environnement logiciel autant que des données ; ne pas conclure immédiatement à une corruption irréparable.

### 6.4 Migrer des références persistantes avec zodbupdate

`zodbupdate` traite notamment des changements de références à des classes persistantes à partir de règles adaptées. Il peut aussi intervenir dans des procédures spécifiques de migration Python 2 vers Python 3. Ce n'est pas une commande de « réparation générale ». [^zodbupdate]

```bash
# Sur une copie, après installation du code et des règles attendues.
.venv/bin/zodbupdate -f /chemin/vers/une-copie/filestorage/Data.fs
```

Sa variante `-c` utilise une **configuration de stockage ZConfig adaptée à l'outil**. Ne lui transmettre ni le fichier WSGI `zope.ini`, ni automatiquement l'intégralité de la configuration du serveur Zope. Préparer le fichier de stockage selon le backend et la documentation de la version installée. [^zodbupdate]

Dans un add-on qui fournit effectivement des règles de renommage, la déclaration peut prendre cette forme :

```toml
[project.entry-points."zodbupdate"]
renames = "mon.addon.migrations:RENAMES"
```

```python
# mon/addon/migrations.py — exemple fictif à adapter au refactoring réel.
RENAMES = {
    "ancien.module AncienneClasse": "nouveau.module NouvelleClasse",
}
```

Les chaînes de cet exemple séparent le nom de module et le nom de classe par une espace. Les modules cibles doivent exister et les objets migrés doivent ensuite être testés avec le code de destination. [^zodbupdate]

### 6.5 Sauvegarde, pack et retour arrière

Avant toute opération modifiant la base, conserver une sauvegarde cohérente des stockages et des blobs, ainsi que les versions de code et la configuration permettant de les relire. Pour une petite instance FileStorage, l'arrêt du service pendant la copie simplifie cette cohérence. Pour un système en service, employer une stratégie compatible avec le backend et **tester la restauration**. Une copie arbitraire de fichiers pendant des écritures n'est pas une preuve de sauvegarde exploitable. [^template-backup]

Ne pas associer systématiquement `--pack` à une migration. Le pack supprime de l'historique et, selon la configuration, des objets devenus inaccessibles ; il réduit donc certaines possibilités d'analyse ou de retour arrière. Le planifier séparément, après validation et conformément à la politique de conservation. [^template-pack]

> [!danger] Données persistantes non fiables
> Ne pas ouvrir une ZODB d'origine inconnue dans un environnement de confiance : le chargement d'objets persistants fait intervenir les mécanismes de désérialisation Python et le code des classes. Une base à expertiser doit être traitée comme une entrée potentiellement dangereuse. [^pickle]

Les outils de maintenance ne figurent pas dans le socle exécutable minimal du cours : les ajouter à des exigences de maintenance, en choisir les versions compatibles avec la base à traiter et vérifier leurs options avec `--help`. Leur ajout automatique à chaque serveur de production n'est pas nécessaire.

---

## 7. Guide de migration : de Buildout à mxdev

> [!important] Séparer les changements
> Une migration d'outillage, une mise à niveau de Plone, un changement majeur de Python et une conversion de stockage sont des opérations différentes. Les combiner sans étapes intermédiaires rend l'attribution des erreurs et le retour arrière plus difficiles. Migrer d'abord à versions applicatives constantes lorsque c'est possible.

### 7.1 Préparer un inventaire avant les cinq étapes

Conserver le Buildout fonctionnel, ses extensions, ses versions réellement installées, ses checkouts et leurs commits. Relever les fichiers ZCML, les variables d'environnement, les produits configurés, les chemins, ports et services externes. Inventorier séparément les bases, blobs et montages supplémentaires.

Pour chaque recette, identifier le service rendu : distribution Python, bibliothèque système, fichier généré, tâche planifiée, sauvegarde, instance ZEO ou supervision. **Une recette qui construit un binaire ou configure un service ne se remplace pas nécessairement par une ligne dans `requirements.txt`.** [^comparaison]

Établir enfin une recette de référence : ouverture du site, authentification, édition, téléchargement de pièces jointes, recherche, tâches d'arrière-plan et fonctions métier indispensables.

### 7.2 Étape 1 — Convertir les exigences et les contraintes

Remplacer les références aux `versions.cfg` de la publication choisie par ses contraintes pip, lorsque cette publication fournit bien la voie d'installation visée. Reporter les exigences de l'instance, les extras et les surcharges locales justifiées. Ne pas installer le Plone le plus récent sous les contraintes d'un ancien socle.

```text
# Exemple pour le socle utilisé dans ce cours.
-c https://dist.plone.org/release/6.2.1/constraints.txt
Plone
Zope[wsgi]
```

Ne pas confondre les versions des recettes Buildout avec celles des paquets nécessaires au fonctionnement du serveur : certaines dépendances d'orchestration n'ont plus lieu d'être dans l'environnement applicatif.

### 7.3 Étape 2 — Convertir les sources de développement

Exemple de conversion :

```ini
# Avant : mr.developer
[sources]
collective.exemple = git https://github.com/orga/collective.exemple.git branch=main
```

```ini
# Après : section ajoutée au fichier mxdev.ini
[collective.exemple]
url = https://github.com/orga/collective.exemple.git
branch = main
```

Adapter les URL à des dépôts réels. Vérifier les extras, les chemins des dépôts déjà présents et la politique de mise à jour. Pour une comparaison fiable pendant la migration, conserver les mêmes commits plutôt que profiter de l'opération pour actualiser toutes les branches. [^mxdev]

### 7.4 Étape 3 — Reproduire la configuration Zope et WSGI

Générer une instance de test, puis comparer les réglages **effectifs** : emplacement des bases et blobs, chargement ZCML, authentification, journalisation, variables d'environnement, proxy, limites HTTP, stockage et paramètres applicatifs.

Comparer `zope.conf` et `zope.ini` : ces fichiers couvrent deux niveaux différents. Une concordance de ports ne suffit pas à établir l'équivalence des instances. [^zope-operation]

Ne jamais relier par erreur une configuration de test à la base de production. Utiliser des chemins et comptes explicitement distincts. Après restauration de la copie, exécuter la recette de référence et les vérifications de données retenues.

### 7.5 Étape 4 — Moderniser les paquets locaux par lots contrôlés

Migrer les métadonnées vers `pyproject.toml` en évitant leur duplication. Distinguer dépendances de construction, d'exécution et de test ; préserver les points d'entrée, ressources et profils de configuration. Construire une distribution et vérifier son contenu.

Pour les namespaces, appliquer une stratégie cohérente à toutes les distributions concernées. Ne retirer un `__init__.py` qu'après avoir établi qu'il correspond à un namespace à convertir ; conserver ceux des paquets ordinaires et toute initialisation réellement nécessaire. Pour Plone 6.2, tenir compte du guide de migration des namespaces et des anciens usages de `pkg_resources`. [^namespace] [^upgrade62]

Un changement de packaging ne doit pas renommer involontairement les modules de classes persistantes. Si ce renommage est nécessaire, le traiter comme une migration de données distincte avec des règles et tests adaptés.

### 7.6 Étape 5 — Installer les commandes d'équipe et le déploiement

Mettre en place les opérations explicites : installer, configurer, tester, démarrer, diagnostiquer et nettoyer. Puis recréer l'environnement dans un répertoire neuf, sans dépendre de chemins ou paquets présents seulement sur le poste du développeur.

Le succès de `make install && make start` est nécessaire mais insuffisant. Comparer aussi les fonctions métier, le comportement des extensions, les fichiers servis, les montages ZODB, les droits, les journaux et les performances attendues.

### 7.7 Valider la bascule et son retour arrière

La validation doit couvrir une construction propre, des tests significatifs, une restauration répétée et une bascule sur la copie des données. Préparer ensuite la synchronisation finale, l'arrêt des écritures et le contrôle après redémarrage.

Le retour arrière doit définir un **couple code + données compatible**. Revenir à l'ancien environnement Python ne suffit pas lorsqu'une migration a déjà transformé la base. Préciser également comment seront traitées les écritures arrivées après la bascule : restauration à un instant donné, perte acceptée, ou mécanisme de reprise prévu et testé.

---

## 8. Conclusion et validation des acquis

L'intérêt de l'outillage Python courant est de rendre les responsabilités plus lisibles, non de remplacer un ensemble de recettes par des commandes implicitement équivalentes. Une organisation solide sait répondre à quatre questions : **qu'installe-t-on, avec quelles versions, selon quelle configuration et sur quelles données ?**

Buildout peut rester pertinent dans un parc existant ou une organisation qui maîtrise ses recettes. Pour un projet fondé sur pip ou uv, `mxdev` devient utile lorsque la gestion de sources et de contraintes le justifie ; un petit projet n'est pas obligé de l'adopter. Le Makefile proposé est une convention explicite, pas un standard universel de Plone. [^comparaison]

### 8.1 Trois exercices

**Contraintes.** Un fichier contient `Plone==6.2.1` et l'on exécute seulement `pip install -c constraints.txt mon.addon`. Pourquoi Plone peut-il être installé malgré l'absence de `Plone` comme argument direct ? Parce qu'il peut être demandé par les dépendances de `mon.addon` ; la contrainte sélectionne alors une version admissible. Sans exigence directe ou transitive, elle ne déclenche pas son installation.

**Configuration.** Pourquoi `zconsole run instance/etc/zope.ini script.py` n'est-il pas l'équivalent du lancement WSGI ? Parce que `zconsole` attend la configuration Zope, tandis que `runwsgi` reçoit la configuration de la chaîne WSGI.

**Distribution.** Un add-on fonctionne dans `src/` mais perd ses profils XML après livraison. Que vérifier ? Le contenu réel de la wheel, les règles d'inclusion des ressources et l'absence de réinjection du répertoire source dans les tests de livraison.

Ces exercices appliquent les distinctions établies dans les chapitres 3, 4 et 6.

### 8.2 Critères de fin de migration

La migration est considérée comme terminée lorsque l'environnement se recrée à partir d'entrées maîtrisées, que la configuration effective est comprise et que les tests portent sur les artefacts livrés et les fonctions utiles. Elle doit aussi laisser une sauvegarde restaurable, une procédure d'exploitation et un retour arrière compatible avec les données.

### 8.3 Ce que cette révision corrige et ajoute

| Point du document fourni | Correction ou complément |
| --- | --- |
| « Buildout propriétaire » et abandon universel. | Distinction entre logiciel libre, outil spécialisé et choix d'architecture. |
| Modèle d'instance présenté comme générateur de projet complet. | Distinction Cookieplone / Cookiecutter d'instance et arborescence explicitement assemblée. |
| `instance.ini` remplaçant `zope.conf`. | Séparation entre WSGI, Zope et ZCML. |
| `run-wsgi`, `zope-command -c instance.ini`. | Commandes `runwsgi` et `zconsole`, avec les bons types de configuration. |
| Contraintes assimilées à un verrouillage complet. | Exigences, contraintes, résolution et livraison distinguées. |
| Clé locale `path` dans mxdev. | Chemin éditable dans les exigences ou configuration `vcs = fs`. |
| Nettoyage large et cibles incomplètes. | Nettoyage limité, outils isolés et commandes aux effets explicites. |
| Maintenance et `--pack` sans garde-fous. | Copie de travail, sauvegarde cohérente, portée des outils et pack séparé. |
| Suppression générique des `__init__.py`. | Migration contrôlée des namespaces et vérification des distributions. |
| Peu de vérifications de livraison. | Tests de packaging, stratégie sans éditables, procédure de bascule et de retour arrière. |

### 8.4 Limites de validation

La révision repose sur les sources officielles référencées ci-dessous. Les contrôles locaux ont vérifié la syntaxe des exemples YAML, TOML, INI, XML, Python et Bash, ainsi que la lecture du Makefile et ses principales cibles avec `make -n`. Les deux tests de packaging passent sur les sources, puis sur le contenu de la wheel construite, hors du répertoire source. Les commandes de nettoyage ont été exécutées sur une copie jetable contenant des fichiers témoins : les données et les sources sont conservées.

Ces essais utilisent le Python 3.13.5 et le backend de construction déjà disponibles dans l'environnement de révision ; ils ne reproduisent pas l'installation complète avec les versions du socle. **Le serveur Plone complet n'a pas été installé ni démarré**, et aucune migration de base n'a été exécutée. Les contraintes de la publication 6.2.2 restent à contrôler avant d'actualiser le socle pédagogique.

Les noms `mon.addon`, `collective.exemple`, les URL d'organisations fictives et les règles de renommage illustrent les mécanismes. Les deux tests de packaging sont volontairement limités ; ils ne valent ni certification de compatibilité des add-ons, ni recette de production.

## Ressources et sources primaires

Sources consultées pour la révision du **20 septembre 2026**. Les liens contenant `latest` peuvent évoluer ; les contraintes 6.2.1 et le tag 3.1.0 du modèle identifient les références utilisées pour les exemples. Les références servent à distinguer les comportements documentés des choix d'organisation proposés dans ce cours.

[^plone-pypi]: Plone — [publication et historique sur PyPI](https://pypi.org/project/Plone/). Version 6.2.2 observée ; dates du dépôt de fichiers et du changelog distinguées.
[^release]: Plone 6.2.1 — [notes de publication](https://dist.plone.org/release/6.2.1/RELEASE-NOTES.md), notamment versions de Python prises en charge et voies d'installation.
[^contraintes]: Plone 6.2.1 — [contraintes officielles](https://dist.plone.org/release/6.2.1/constraints.txt).
[^comparaison]: Documentation Plone — [comparaison de Buildout et pip](https://6.docs.plone.org/conceptual-guides/compare-buildout-pip.html).
[^install-pip]: Documentation Plone — [installation avec pip](https://6.docs.plone.org/admin-guide/install-pip.html).
[^cookieplone]: Documentation Plone — [créer un projet avec Cookieplone](https://6.docs.plone.org/install/create-project-cookieplone.html).
[^template-version]: `cookiecutter-zope-instance` — [publication 3.1.0](https://github.com/plone/cookiecutter-zope-instance/releases/tag/3.1.0).
[^template-config]: `cookiecutter-zope-instance` — [paramètres du modèle au tag 3.1.0](https://github.com/plone/cookiecutter-zope-instance/blob/3.1.0/cookiecutter.json).
[^template-tutoriel]: `cookiecutter-zope-instance` — [première instance Zope](https://plone.github.io/cookiecutter-zope-instance/tutorials/first-zope-instance.html).
[^template-workflow]: `cookiecutter-zope-instance` — [cycle de configuration](https://plone.github.io/cookiecutter-zope-instance/explanation/configuration-workflow.html).
[^zcml]: `cookiecutter-zope-instance` — [référence des paramètres ZCML](https://plone.github.io/cookiecutter-zope-instance/reference/zcml.html).
[^filestorage]: `cookiecutter-zope-instance` — [configuration de FileStorage direct](https://plone.github.io/cookiecutter-zope-instance/how-to/configure-filestorage.html).
[^template-backup]: `cookiecutter-zope-instance` — [sauvegarder et restaurer](https://plone.github.io/cookiecutter-zope-instance/how-to/backup-and-restore.html).
[^template-pack]: `cookiecutter-zope-instance` — [pack et collecte des objets](https://plone.github.io/cookiecutter-zope-instance/how-to/pack-and-gc.html).
[^cookiecutter]: Cookiecutter — [publication et documentation sur PyPI](https://pypi.org/project/cookiecutter/).
[^cookiecutter-replay]: Cookiecutter — [replay des paramètres](https://cookiecutter.readthedocs.io/en/stable/advanced/replay.html).
[^cookiecutter-hooks]: Cookiecutter — [hooks de génération](https://cookiecutter.readthedocs.io/en/stable/advanced/hooks.html).
[^mxdev]: mxdev — [documentation, options et historique sur PyPI](https://pypi.org/project/mxdev/) ; [dépôt du projet](https://github.com/mxstack/mxdev).
[^zope-operation]: Zope — [configuration et exploitation](https://zope.readthedocs.io/en/latest/operation.html).
[^pyproject]: Python Packaging User Guide — [écrire un pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/).
[^pep660]: Python — [PEP 660, installations éditables](https://peps.python.org/pep-0660/).
[^pep420]: Python — [PEP 420, namespaces implicites](https://peps.python.org/pep-0420/).
[^venv]: Python — [documentation de venv](https://docs.python.org/3/library/venv.html).
[^wheel]: Python Packaging User Guide — [balises de compatibilité des distributions binaires](https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/).
[^pip]: pip — [guide utilisateur : contraintes et contraintes de construction](https://pip.pypa.io/en/stable/user_guide/).
[^datafiles]: setuptools — [inclure des fichiers de données](https://setuptools.pypa.io/en/latest/userguide/datafiles.html).
[^upgrade62]: Documentation Plone — [migration vers Plone 6.2](https://6.docs.plone.org/backend/upgrading/version-specific-migration/upgrade-to-62.html).
[^namespace]: Documentation Plone — [namespaces natifs](https://6.docs.plone.org/developer-guide/native-namespace.html).
[^uv-compat]: uv — [différences avec pip](https://docs.astral.sh/uv/pip/compatibility/).
[^uv-sync]: uv — [verrouillage et synchronisation d'un projet](https://docs.astral.sh/uv/concepts/projects/sync/).
[^uv-compile]: uv — [compilation des exigences et synchronisation](https://docs.astral.sh/uv/pip/compile/).
[^zodbverify]: zodbverify — [documentation et exemples sur PyPI](https://pypi.org/project/zodbverify/).
[^zodbupdate]: zodbupdate — [documentation des migrations et règles de renommage](https://pypi.org/project/zodbupdate/).
[^pickle]: Python — [avertissement de sécurité sur pickle](https://docs.python.org/3/library/pickle.html).
