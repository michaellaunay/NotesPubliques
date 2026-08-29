---
schema_version: 1
uid: "01M02EX5CCGYQZBX1P5GJCGVT3"
titre: "zc.buildout"
aliases:
  - "zc.buildout"
  - "buildout"
  - "ZC.Buildout"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - deploiement
  - buildout
  - zope
resume: "Cours sur zc.buildout : assemblage reproductible d'applications, parts et recipes, installation moderne, fichiers buildout.cfg, gestion et verrouillage des versions, héritage de configurations, développement local, commandes de diagnostic et place de Buildout dans l'écosystème Python actuel."
niveau: intermediaire
prerequis:
  - "[[Python]]"
  - "[[git]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-01-27
date_modification: 2026-08-29
date_verification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# zc.buildout

> [!abstract] Objectif
> Comprendre ce que zc.buildout assemble réellement (`buildout.cfg`, parts, recipes, scripts générés), savoir installer et rendre reproductible un buildout, diagnostiquer une configuration et situer l'outil face à `venv`, pip, uv et tox en 2026 — sans le déclarer mort ni l'utiliser là où un outil plus simple suffit.

Voir aussi : [[Python]], [[Pyramid]], [[Deform]].

# Résumé

**zc.buildout**, généralement appelé **Buildout**, est un outil d'assemblage et d'automatisation d'applications écrit en Python. Son objectif ne se limite pas à installer des bibliothèques : il permet de décrire un environnement applicatif complet sous forme de configuration reproductible.

Un buildout peut notamment :

- installer des dépendances Python ;
- générer des scripts d'exécution ;
- produire des fichiers de configuration ;
- assembler plusieurs composants d'une application ;
- gérer plusieurs configurations, par exemple développement et production ;
- verrouiller les versions utilisées afin de reproduire un environnement ;
- exécuter des tâches spécifiques grâce à des **recipes**.

Buildout est historiquement très présent dans les écosystèmes **Zope** et **Plone**, mais son modèle reste intéressant pour tout projet dans lequel l'installation d'une application nécessite davantage qu'un simple `pip install`.

Il faut distinguer Buildout d'un environnement virtuel Python. Un `venv` isole un interpréteur et les paquets qui y sont installés ; Buildout décrit et construit un **assemblage applicatif**. Les deux peuvent donc être utilisés ensemble.

# État de zc.buildout en 2026

Au 29 août 2026 :

- la branche stable est **zc.buildout 5.2.0**, publiée le 29 avril 2026 ;
- elle nécessite **Python 3.9 ou plus récent** ;
- zc.buildout 5 installe la plupart des distributions Python en s'appuyant sur `pip` et utilise les namespaces natifs de Python ;
- les paquets installés sont placés dans un sous-répertoire `eggs/v5` afin d'éviter des incompatibilités avec les anciennes générations de Buildout ;
- **zc.buildout 6.0.0a1** est une préversion publiée le 14 août 2026. Elle requiert Python 3.10+, `pip >= 25.0` et améliore notamment la prise en charge des installations éditables PEP 660.

Pour un nouveau projet de production, nous utilisons donc normalement la **dernière version stable 5.x**. Nous ne passons pas automatiquement un ancien projet Buildout en version 5 ou 6 : les projets historiques peuvent dépendre de vieilles recipes, de vieux namespaces ou de versions particulières de `setuptools`.

# Plan du cours

1. Introduction à Buildout
2. Installation et premier buildout
3. Comprendre `buildout.cfg`, les parts et les recipes
4. Gérer les dépendances et rendre un buildout reproductible
5. Organiser plusieurs configurations et développer un paquet local
6. Utiliser Buildout au quotidien et diagnostiquer une configuration
7. Fonctionnement avancé et création de recipes
8. Buildout dans l'écosystème Python moderne et migration d'un ancien projet
9. Conclusion et ressources

# 1. Introduction à Buildout

## 1.1 Les deux objectifs principaux

Buildout répond historiquement à deux besoins.

### Assembler et déployer une application

Une application ne se résume pas toujours à une liste de paquets Python. Elle peut aussi nécessiter :

- des scripts de lancement ;
- des fichiers de configuration ;
- un serveur applicatif ;
- des répertoires de données ;
- des tâches planifiées ;
- des outils de test ou d'administration ;
- parfois des composants non Python.

Buildout permet de décrire cet assemblage dans des fichiers texte versionnés avec le code source.

### Reproduire un environnement

Buildout cherche à rendre l'assemblage reproductible. À configuration, environnement et sources identiques, deux installations doivent produire le même résultat.

Cette reproductibilité dépend notamment :

- de la version de Python ;
- du système d'exploitation ;
- des versions des dépendances ;
- des sources externes téléchargées ;
- des recipes utilisées.

Buildout permet de verrouiller une grande partie de ces éléments, mais il ne remplace pas à lui seul un conteneur ou une image système reproductible.

## 1.2 Buildout, pip, venv et Docker

Ces outils ne répondent pas exactement au même problème.

| Outil | Rôle principal |
| --- | --- |
| `pip` | Installer des distributions Python |
| `venv` | Isoler un environnement Python |
| Buildout | Assembler et configurer une application à partir de parts et de recipes |
| Docker | Isoler et reproduire un environnement système et applicatif dans un conteneur |

Un projet moderne peut par exemple utiliser :

1. Docker pour figer l'environnement système ;
2. un `venv` pour isoler Python ;
3. Buildout pour construire l'application et générer ses scripts et fichiers de configuration.

Dans un projet plus simple, Buildout peut être inutile : `pyproject.toml`, `pip`, `uv`, `tox` ou d'autres outils modernes peuvent suffire.

## 1.3 Les notions essentielles

Un buildout repose principalement sur quatre notions.

### Le fichier de configuration

Par défaut, Buildout lit :

```text
buildout.cfg
```

La syntaxe est proche du format INI.

### La section `[buildout]`

Elle contient la configuration générale et surtout la liste des **parts** à construire.

### Les parts

Une **part** représente une unité d'installation ou de génération.

Exemples :

- installer une application Python ;
- créer un interpréteur ;
- générer un fichier de configuration ;
- préparer une instance Zope ;
- installer un outil de test.

### Les recipes

Une **recipe** est le code Python chargé de construire une part.

La section suivante :

```ini
[app]
recipe = zc.recipe.egg
eggs = mon-application
```

signifie : « construire la part `app` en utilisant la recipe fournie par `zc.recipe.egg` ».

Buildout est donc un **moteur d'orchestration**, et les recipes sont ses plugins.

# 2. Installation et premier buildout

## 2.1 Prérequis

Pour la branche stable 5.2.0, il faut Python 3.9 ou plus récent.

Nous vérifions notre interpréteur avec :

```bash
python3 --version
```

Sur un nouveau projet, il est recommandé d'installer Buildout dans un environnement virtuel plutôt que dans l'installation Python globale du système.

## 2.2 Créer un environnement virtuel

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install zc.buildout
```

Sous Windows PowerShell, l'activation du `venv` se fait généralement avec :

```powershell
.venv\Scripts\Activate.ps1
```

Nous vérifions ensuite Buildout :

```bash
buildout --version
```

`pip` n'installe normalement pas une préversion lorsqu'une version stable convient. La commande précédente installe donc la dernière version stable disponible.

Pour un environnement dont nous voulons maîtriser explicitement la génération majeure, nous pouvons écrire :

```bash
python -m pip install 'zc.buildout>=5,<6'
```

## 2.3 Premier fichier `buildout.cfg`

Créons un fichier minimal :

```ini
[buildout]
parts = py

[py]
recipe = zc.recipe.egg
eggs = requests
interpreter = py
```

Puis exécutons :

```bash
buildout
```

Buildout crée notamment les répertoires suivants :

```text
bin/
develop-eggs/
eggs/
parts/
```

Avec Buildout 5, les distributions Python gérées dans le cache local sont généralement placées sous :

```text
eggs/v5/
```

La recipe `zc.recipe.egg` crée ici un interpréteur :

```bash
./bin/py
```

Nous pouvons vérifier que `requests` est accessible :

```bash
./bin/py -c 'import requests; print(requests.__version__)'
```

## 2.4 Installer un script Buildout local

La commande :

```bash
buildout bootstrap
```

installe un script Buildout dans le répertoire `bin` du buildout.

Nous pouvons ensuite utiliser :

```bash
./bin/buildout
```

L'intérêt est que ce script local est associé au buildout et peut respecter les contraintes de versions définies par celui-ci.

L'ancienne pratique consistant à télécharger puis exécuter un fichier `bootstrap.py` :

```bash
python bootstrap.py
```

appartient aux anciennes générations de Buildout et ne doit plus être présentée comme la méthode normale pour un nouveau projet.

# 3. Comprendre `buildout.cfg`, les parts et les recipes

## 3.1 Structure générale

Un fichier Buildout contient des sections :

```ini
[buildout]
parts = app

[app]
recipe = zc.recipe.egg
eggs = mon-application
```

La première section est spéciale. Elle décrit le buildout lui-même.

L'option :

```ini
parts = app
```

indique à Buildout qu'il doit installer la part `app`.

La section `[app]` indique comment la construire.

## 3.2 Plusieurs parts

Nous pouvons construire plusieurs éléments :

```ini
[buildout]
parts =
    app
    outils

[app]
recipe = zc.recipe.egg
eggs = mon-application

[outils]
recipe = zc.recipe.egg
eggs =
    ipython
    pytest
interpreter = py-outils
```

Chaque recipe décide de la signification de ses propres options.

## 3.3 L'option `eggs`

Le nom `eggs` est historique. Dans `zc.recipe.egg`, cette option contient en réalité des **requirements Python**, un par ligne :

```ini
eggs =
    requests
    PyYAML >= 6
    Flask < 4
```

La syntaxe suit les contraintes de versions du packaging Python.

Il vaut mieux écrire les requirements sur plusieurs lignes plutôt que séparés par des virgules.

## 3.4 Génération de scripts

`zc.recipe.egg` peut générer dans `bin/` les scripts déclarés par les paquets installés.

Exemple :

```ini
[cli]
recipe = zc.recipe.egg
eggs = httpie
```

Les scripts exposés par la distribution sont alors générés dans le répertoire `bin` du buildout.

Nous pouvons limiter ou renommer les scripts avec l'option `scripts` :

```ini
[cli]
recipe = zc.recipe.egg
eggs = httpie
scripts = http=mon-http
```

Une valeur vide désactive leur génération :

```ini
scripts =
```

## 3.5 Créer un interpréteur dédié

L'option :

```ini
interpreter = py
```

crée :

```text
bin/py
```

Cet interpréteur utilise le Python du buildout avec les dépendances déclarées par la part dans son chemin de modules.

Il est pratique pour :

- tester des imports ;
- lancer une console interactive ;
- exécuter un script dans l'environnement exact d'une part.

## 3.6 Substitutions de variables

Buildout peut réutiliser la valeur d'une autre option avec la syntaxe :

```text
${section:option}
```

Exemple :

```ini
[ports]
http = 8080

[application]
recipe = ma.recipe
port = ${ports:http}
root = ${buildout:directory}
```

Quelques valeurs globales utiles sont notamment :

```text
${buildout:directory}
${buildout:bin-directory}
${buildout:parts-directory}
${buildout:eggs-directory}
```

Les substitutions évitent de dupliquer des chemins et des paramètres dans plusieurs sections.

## 3.7 Suivi de l'état installé

Buildout mémorise l'état des parts installées, notamment dans :

```text
.installed.cfg
```

Lors d'une nouvelle exécution, il compare la configuration demandée avec l'état précédent et installe, met à jour ou désinstalle les parts concernées.

Il est donc normal de relancer simplement :

```bash
./bin/buildout
```

après avoir modifié `buildout.cfg`.

Il n'existe pas de commande générale moderne `buildout update` destinée à mettre à jour toutes les dépendances. Pour changer les versions, nous modifions les contraintes ou les pins puis nous relançons le buildout.

# 4. Gérer les dépendances et rendre un buildout reproductible

## 4.1 Pourquoi verrouiller les versions

Si aucune version n'est imposée, Buildout peut sélectionner la version la plus récente compatible avec les requirements au moment de l'installation.

Deux installations effectuées à plusieurs mois d'intervalle pourraient alors produire des environnements différents.

Pour une **application**, nous cherchons généralement à verrouiller l'ensemble des versions après validation des tests.

Pour une **bibliothèque**, au contraire, il vaut mieux déclarer des plages de compatibilité plutôt que forcer toutes les dépendances à une version unique.

## 4.2 La section `[versions]`

Nous pouvons centraliser les versions :

```ini
[buildout]
parts = app

[app]
recipe = zc.recipe.egg
eggs = requests
interpreter = py

[versions]
requests = 2.34.2
```

Buildout utilise par défaut la section nommée `versions`.

Nous pouvons choisir un autre nom avec :

```ini
[buildout]
versions = contraintes

[contraintes]
requests = 2.34.2
```

## 4.3 Séparer les versions dans `versions.cfg`

Sur un projet important, il est préférable de séparer les pins :

```text
buildout.cfg
versions.cfg
```

`versions.cfg` :

```ini
[versions]
requests = 2.34.2
```

`buildout.cfg` :

```ini
[buildout]
extends = versions.cfg
parts = app

[app]
recipe = zc.recipe.egg
eggs = requests
interpreter = py
```

Le fichier principal hérite des valeurs de `versions.cfg`.

## 4.4 Voir les versions choisies automatiquement

L'option :

```ini
[buildout]
show-picked-versions = true
```

fait afficher les versions sélectionnées lorsqu'elles n'étaient pas verrouillées.

Pour faire maintenir un fichier de versions par Buildout :

```ini
[buildout]
extends = versions.cfg
show-picked-versions = true
update-versions-file = versions.cfg
```

Le fichier `versions.cfg` doit contenir une section `[versions]` :

```ini
[versions]
```

Buildout peut alors ajouter les versions qu'il sélectionne.

Une démarche pratique consiste à :

1. laisser Buildout résoudre les dépendances ;
2. générer ou compléter `versions.cfg` ;
3. exécuter les tests ;
4. versionner le fichier de pins avec Git ;
5. refuser ensuite les versions non verrouillées.

## 4.5 Refuser les versions non verrouillées

Lorsque le fichier de versions est complet :

```ini
[buildout]
allow-picked-versions = false
```

Buildout échoue alors si une dépendance doit être choisie sans pin explicite.

Cette option est très utile pour détecter l'apparition d'une nouvelle dépendance transitive non verrouillée.

## 4.6 L'option `-N`

Par défaut, Buildout vérifie si des versions plus récentes sont disponibles.

La commande :

```bash
./bin/buildout -N
```

équivaut à :

```ini
[buildout]
newest = false
```

Elle évite certaines recherches de nouvelles versions et peut accélérer une exécution.

Attention : `-N` **ne remplace pas le verrouillage des versions**. La reproductibilité repose avant tout sur des contraintes précises et un environnement maîtrisé.

# 5. Organiser plusieurs configurations et développer un paquet local

## 5.1 Hériter d'une configuration avec `extends`

Un projet peut partager une configuration commune.

`base.cfg` :

```ini
[buildout]
parts = app

[app]
recipe = zc.recipe.egg
eggs = mon-application
interpreter = py
```

`development.cfg` :

```ini
[buildout]
extends = base.cfg
parts += outils

[outils]
recipe = zc.recipe.egg
eggs =
    pytest
    ipython
```

Nous utilisons ensuite :

```bash
./bin/buildout -c development.cfg
```

## 5.2 Ajouter ou retirer des valeurs

Buildout permet de compléter une option héritée avec `+=` :

```ini
[buildout]
parts += outils
```

et d'en retirer avec `-=` :

```ini
[buildout]
parts -= outils
```

Cette mécanique permet de construire des variantes sans recopier toute la configuration.

## 5.3 Configuration locale facultative

L'option `optional-extends` permet de charger un fichier seulement s'il existe :

```ini
[buildout]
extends = versions.cfg
optional-extends = local.cfg
```

`local.cfg` peut être ajouté au `.gitignore` et contenir des réglages propres à une machine de développement.

Il faut éviter d'y stocker des secrets si d'autres mécanismes plus sûrs sont disponibles. Une configuration Buildout n'est pas un coffre-fort.

## 5.4 Utiliser un paquet local en développement

Buildout possède l'option `develop` pour rendre un projet Python local disponible en mode développement :

```ini
[buildout]
develop = .
parts = app

[app]
recipe = zc.recipe.egg
eggs = mon-projet
interpreter = py
```

Une modification du code source local est alors visible sans réinstaller une nouvelle archive du paquet.

Dans Buildout 5, les installations de développement sont désormais en grande partie réalisées avec `pip`. Cela facilite la transition vers le packaging Python moderne, mais certains cas utilisant des namespaces historiques restent délicats.

La préversion Buildout 6 améliore encore ce point avec la prise en charge des installations éditables conformes à **PEP 660** et des projets configurés uniquement avec `pyproject.toml`.

## 5.5 Plusieurs checkouts

Un projet ancien ou de grande taille peut développer plusieurs paquets simultanément :

```ini
[buildout]
develop =
    src/mon-projet
    src/ma-bibliotheque
    src/mon-plugin
```

Dans les écosystèmes Zope et Plone, des extensions telles que `mr.developer` ont longtemps servi à automatiser la récupération et la gestion de nombreux dépôts de développement.

# 6. Utiliser Buildout au quotidien et diagnostiquer une configuration

## 6.1 Construire ou reconstruire le buildout

La commande principale est simplement :

```bash
./bin/buildout
```

ou, si nous utilisons l'exécutable du `venv` :

```bash
buildout
```

Buildout installe les parts demandées et met à jour celles dont la configuration a changé.

## 6.2 Utiliser un autre fichier de configuration

```bash
./bin/buildout -c production.cfg
```

L'option `-c` sélectionne le fichier de configuration principal.

Le nom du fichier n'est pas un « profil » spécial pour Buildout : `production.cfg`, `development.cfg` ou `test.cfg` sont simplement des conventions choisies par le projet.

## 6.3 Installer certaines parts seulement

Nous pouvons demander explicitement des parts :

```bash
./bin/buildout install app outils
```

C'est utile pour diagnostiquer ou reconstruire une partie d'un buildout volumineux.

## 6.4 Afficher la version

```bash
./bin/buildout --version
```

## 6.5 Augmenter la verbosité

```bash
./bin/buildout -v
```

ou, pour un diagnostic encore plus détaillé :

```bash
./bin/buildout -vv
```

`-v` est particulièrement utile pour comprendre pourquoi une version de distribution a été sélectionnée.

## 6.6 Comprendre la configuration finale avec `annotate`

La commande :

```bash
./bin/buildout annotate
```

affiche les options de configuration finales et surtout **leur provenance**.

Elle est très utile lorsque plusieurs fichiers utilisent `extends` et se surchargent mutuellement.

Pour limiter l'affichage à une section :

```bash
./bin/buildout annotate versions
```

Pour afficher davantage d'étapes de calcul :

```bash
./bin/buildout -v annotate
```

`annotate` ne sert donc pas à produire une simple liste des dépendances Python. Son rôle principal est d'expliquer comment Buildout a obtenu la configuration effective.

## 6.7 Interroger une option avec `query`

Pour afficher la liste finale des parts :

```bash
./bin/buildout query parts
```

Pour interroger une option précise :

```bash
./bin/buildout query app:eggs
```

Cette commande est pratique dans les scripts d'administration et lors du débogage.

## 6.8 Surcharger temporairement une valeur en ligne de commande

La ligne de commande peut surcharger des options :

```bash
./bin/buildout buildout:newest=false
```

ou :

```bash
./bin/buildout versions:requests=2.32.5
```

Ces modifications sont temporaires : elles ne changent pas les fichiers `.cfg`.

## 6.9 Déboguer une recipe

L'option :

```bash
./bin/buildout -D
```

lance un débogueur post-mortem lorsqu'une erreur survient. Elle est surtout utile pour développer ou diagnostiquer une recipe.

# 7. Fonctionnement avancé et création de recipes

## 7.1 Une recipe est un paquet Python

Une recipe est généralement distribuée comme un paquet Python. Buildout charge la recipe indiquée par :

```ini
[ma-part]
recipe = mon.paquet.recipe
```

Le paquet fournit un composant Python respectant l'API attendue par Buildout.

Une recipe reçoit notamment :

- l'objet buildout ;
- le nom de la part ;
- les options de la section.

Elle peut ensuite créer des fichiers, télécharger des ressources ou installer des composants.

## 7.2 Cycle de vie simplifié

Une recipe classique fournit notamment les opérations :

```text
install
update
```

- `install` construit la part lorsqu'elle n'est pas encore installée ;
- `update` met à jour une part déjà présente lorsque sa configuration a changé.

La recipe retourne les chemins qu'elle a créés afin que Buildout puisse les supprimer lorsqu'une part est désinstallée.

## 7.3 Pourquoi écrire une recipe

Nous écrivons une recipe lorsqu'une opération doit être :

- répétable ;
- paramétrable dans `buildout.cfg` ;
- intégrée au cycle de vie du buildout ;
- réutilisable dans plusieurs projets.

Exemples :

- générer un fichier de configuration à partir d'options ;
- préparer une arborescence ;
- installer un programme externe ;
- créer des scripts de lancement ;
- automatiser une opération spécifique à une plateforme.

Pour une tâche ponctuelle très simple, un script shell ou Python peut être plus lisible qu'une recipe personnalisée.

## 7.4 Recipes et responsabilité

Chaque recipe interprète elle-même ses options. Deux parts utilisant deux recipes différentes peuvent donc avoir des syntaxes complètement différentes.

Il faut toujours consulter la documentation de la recipe utilisée et vérifier :

- sa dernière version ;
- les versions de Python supportées ;
- les versions de Buildout supportées ;
- son activité de maintenance ;
- les changements incompatibles.

Une grande partie des difficultés rencontrées lors de la modernisation d'un vieux buildout vient davantage des recipes historiques que du moteur `zc.buildout` lui-même.

# 8. Buildout dans l'écosystème Python moderne et migration d'un ancien projet

## 8.1 Buildout n'est pas obsolète, mais son rôle est devenu plus spécialisé

Buildout reste activement maintenu en 2026. La branche 5 a profondément modernisé son mécanisme d'installation en utilisant `pip` dans la majorité des cas et en améliorant la gestion des namespaces modernes.

Cependant, le packaging Python dispose aujourd'hui d'outils qui couvrent de nombreux usages autrefois confiés à Buildout :

- `venv` pour l'isolation ;
- `pip` ou `uv` pour l'installation ;
- `pyproject.toml` pour décrire un projet ;
- `tox` ou `nox` pour les environnements de test ;
- Docker ou Podman pour figer l'environnement système ;
- les gestionnaires de configuration et orchestrateurs pour le déploiement.

Buildout conserve un intérêt particulier lorsqu'un projet possède déjà un important écosystème de recipes ou lorsqu'il assemble plusieurs composants et fichiers de configuration selon une logique difficile à remplacer par un simple gestionnaire de dépendances.

## 8.2 Ce qui change avec Buildout 5

Les changements importants de la génération 5 sont notamment :

- installation de la plupart des distributions via `pip` ;
- utilisation privilégiée des namespaces natifs **PEP 420** ;
- installation des développements locaux via `pip` ;
- stockage dans `eggs/v5` afin de séparer le format des anciennes générations ;
- `zc.recipe.egg` 4.x destiné à Buildout 5.x.

Ces changements améliorent l'intégration avec le packaging Python moderne, mais ils peuvent révéler des incompatibilités dans de vieux projets.

## 8.3 Attention aux anciens namespaces

Les anciens projets Zope/Plone utilisent parfois des namespaces reposant sur `pkg_resources`.

Les namespaces Python modernes utilisent généralement **PEP 420**, sans fichier `__init__.py` spécifique au namespace.

Lors d'une migration vers Buildout 5 ou 6, il faut donc vérifier les packages partageant un namespace, en particulier lorsqu'ils sont utilisés en mode développement.

Nous ne devons pas supprimer ou modifier ces mécanismes au hasard : une migration de namespace doit être testée sur l'ensemble des packages concernés.

## 8.4 Méthode de migration d'un ancien buildout

Pour moderniser un ancien projet, procédons par étapes.

### Étape 1 — Inventorier l'environnement actuel

Relever :

```bash
python --version
./bin/buildout --version
python -m pip --version
```

Puis identifier :

- les fichiers `buildout.cfg`, `versions.cfg` et fichiers étendus ;
- les recipes ;
- les paquets installés en `develop` ;
- les versions explicitement verrouillées ;
- les éventuels `bootstrap.py` ;
- les dépendances à `setuptools`, `pkg_resources` ou aux anciens namespaces.

### Étape 2 — Sauvegarder une installation reproductible

Avant toute migration :

- versionner les fichiers de configuration ;
- conserver le fichier de versions fonctionnel ;
- noter la version de Python ;
- exécuter les tests ;
- idéalement construire une image ou un conteneur de référence.

### Étape 3 — Vérifier les recipes

Pour chaque recipe :

1. rechercher sa version actuelle ;
2. vérifier sa compatibilité Python ;
3. vérifier sa compatibilité Buildout ;
4. lire son historique de changements.

### Étape 4 — Migrer une génération à la fois si nécessaire

Sur un projet très ancien, sauter directement plusieurs générations de Python, Buildout, setuptools, Zope et Plone rend le diagnostic difficile.

Il est souvent préférable de séparer :

- migration Python ;
- migration de Buildout ;
- migration des recipes ;
- migration du framework applicatif.

### Étape 5 — Comparer la configuration effective

Après modification :

```bash
./bin/buildout annotate
```

permet de vérifier l'origine des valeurs finales.

Puis nous reconstruisons l'environnement et exécutons les tests applicatifs.

## 8.5 Buildout 6

Au 29 août 2026, Buildout 6 n'est encore disponible qu'en préversion `6.0.0a1`.

Cette branche :

- nécessite Python 3.10 ou plus récent ;
- nécessite `pip >= 25.0` ;
- modernise la dépendance à `pkg_resources` en embarquant la dernière version nécessaire à son propre fonctionnement ;
- corrige la prise en charge des paquets de développement basés sur PEP 660 et `pyproject.toml`.

Il est intéressant de la tester sur un projet de développement, mais une migration de production doit tenir compte de son statut de préversion et de la compatibilité de toutes les recipes utilisées.

# 9. Conclusion

Buildout est un outil d'assemblage d'applications, pas seulement un gestionnaire de dépendances.

Les notions importantes à retenir sont :

- `buildout.cfg` décrit l'assemblage ;
- `[buildout]` liste les parts et contient les options globales ;
- chaque part est construite par une recipe ;
- `zc.recipe.egg` installe des distributions Python et génère des scripts ou interpréteurs ;
- les versions doivent être verrouillées pour garantir une bonne reproductibilité ;
- `extends` permet de construire des configurations développement, test et production à partir d'une base commune ;
- `develop` permet de travailler sur des paquets locaux ;
- `annotate` explique la provenance de la configuration finale ;
- relancer `bin/buildout` est la manière normale d'appliquer une modification ;
- `buildout bootstrap` remplace les anciens usages de `bootstrap.py` pour créer un script local ;
- Buildout 5 est la branche stable moderne en 2026 et rapproche fortement Buildout du fonctionnement actuel de `pip`.

Pour un nouveau projet, nous devons toutefois nous demander si Buildout apporte réellement quelque chose par rapport à un environnement `venv`, un `pyproject.toml`, un gestionnaire de dépendances moderne et éventuellement un conteneur. Son principal intérêt apparaît lorsque l'application doit **assembler plusieurs composants et générer un environnement complet de manière déclarative**.

# Ressources

- Documentation officielle de Buildout : https://www.buildout.org/
- Dépôt officiel : https://github.com/buildout/buildout
- Projet PyPI `zc.buildout` : https://pypi.org/project/zc.buildout/
- Projet PyPI `zc.recipe.egg` : https://pypi.org/project/zc.recipe.egg/
- Référence des options et commandes : https://github.com/buildout/buildout/blob/master/doc/reference.rst
- Guide de démarrage : https://github.com/buildout/buildout/blob/master/doc/getting-started.rst
- Écriture de recipes : https://github.com/buildout/buildout/blob/master/doc/topics/writing-recipes.rst
