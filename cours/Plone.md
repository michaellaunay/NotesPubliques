---
schema_version: 1
uid: "01M4697ETZ2DTFPXXZ3Z09SZ8T"
titre: "Plone 6.2"
aliases:
  - "Plone"
  - "Plone 6"
  - "Plone 6.2"
  - "Classic UI"
  - "Dexterity"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - developpement-web
  - python
  - zope
  - plone
  - cms
  - deploiement
resume: "Cours complet sur Plone 6.2 : architecture (Zope, ZODB, ZCA, CMF), installation avec Cookieplone, pip ou conteneurs, distributions, administration et exploitation, types de contenu Dexterity, comportements, vocabulaires, catalogue, workflows et sécurité, plone.api, GenericSetup et étapes de migration, Classic UI (vues, gabarits, viewlets, portlets, thème Barceloneta et Diazo, TinyMCE 8), REST API et Volto, déploiement, cache, mises à jour 6.0 → 6.2 et espaces de noms natifs, travaux pratiques."
niveau: avance
prerequis:
  - "[[Python]]"
  - "[[HTTP]]"
  - "[[HTML]]"
  - "[[Tal et Metal]]"
  - "[[Plone pyproject tooling]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2026-10-05
date_modification: 2026-10-05
date_verification: 2026-10-05
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---

# Plone 6.2

> [!abstract] Objectif
> Comprendre ce qu'est réellement Plone (Zope, ZODB, architecture à composants, CMF), installer et exploiter une instance 6.2 avec les outils actuels (Cookieplone, uv, pip et `cookiecutter-zope-instance`, conteneurs), administrer un site (contenus, workflows, droits, configuration), puis le développer : types de contenu Dexterity, comportements, vocabulaires, catalogue, événements, `plone.api`, GenericSetup et étapes de migration, Classic UI (vues, gabarits, viewlets, portlets, thème Barceloneta et Diazo, TinyMCE 8), REST API et Volto, déploiement et cache, mises à jour jusqu'à 6.2 et passage aux espaces de noms natifs.

Voir aussi : [[Plone pyproject tooling]], [[ZC.Buildout]], [[Tal et Metal]], [[Python]], [[HTTP]], [[Docker]], [[Internationalisation]], [[Pyramid]], [[SEO]].

> [!info] Versions de référence
> Au 5 octobre 2026, la branche stable de Plone est la série **6.2** : Plone **6.2.0** est sorti le 19 mai 2026, 6.2.1 le 26 juin 2026 et **6.2.2** le 10 septembre 2026. Plone 6.2 tourne sur **Python 3.10 à 3.14** et **Zope 6**, embarque **plone.restapi 10**, **TinyMCE 8** pour Classic UI et s'utilise avec **Volto 19**. Le générateur de projet est **Cookieplone 2.0**, l'outillage Python de référence est **uv** (pip et Buildout 5 restent pris en charge). Plone 6.0 et 6.1 bénéficient d'un support de sécurité jusqu'à fin 2027. Les concepts du cours restent valables au-delà de ces numéros.

> [!tip] Lien avec les autres cours
> L'assemblage d'un projet avec `pyproject.toml`, `mxdev`, `cookiecutter-zope-instance` et un `Makefile` est traité en détail dans [[Plone pyproject tooling]] ; l'ancien outil d'assemblage dans [[ZC.Buildout]] ; le langage de gabarits dans [[Tal et Metal]]. Ce cours les suppose connus et renvoie vers eux plutôt que de les répéter.

## Objectifs

À la fin de ce cours, nous devons être capables de :

- expliquer la pile Plone (Python, ZODB, Zope, ZCA, CMF, Plone, REST API, frontends) et situer chaque paquet ;
- choisir entre Classic UI et Volto en connaissance de cause ;
- installer Plone 6.2 avec Cookieplone, avec pip seul, ou en conteneurs, et créer un site à partir d'une distribution ;
- exploiter une instance : démarrage, console de débogage, comptes, journaux, sauvegarde, pack, export et import ;
- administrer un site : types de contenu, workflows et partage, utilisateurs et groupes, panneaux de configuration, règles de contenu ;
- créer un module complémentaire (add-on) et y déclarer des composants en ZCML et des profils GenericSetup ;
- modéliser des types de contenu Dexterity avec schémas, champs, directives, comportements, vocabulaires et relations ;
- indexer et interroger le catalogue, réagir aux événements, stocker des annotations ;
- écrire des workflows DCWorkflow et raisonner en permissions et rôles ;
- utiliser `plone.api`, tester avec `plone.app.testing` et `pytest-plone`, internationaliser ;
- construire l'interface Classic UI : vues, gabarits, couches, viewlets, portlets, ressources statiques, thème, images, TinyMCE, protection CSRF ;
- consommer la REST API et comprendre la place de Volto ;
- déployer en production avec reverse proxy, cache HTTP, ZEO ou RelStorage, et planifier une mise à jour 6.x, y compris le passage aux espaces de noms natifs de 6.2.

# Plan du cours

**Partie I — Comprendre Plone**

1. Qu'est-ce que Plone ?
2. Histoire et versions
3. La pile technique : Python, ZODB, Zope, ZCA, CMF, Plone
4. Concepts fondamentaux : traversal, acquisition, schémas, composants
5. Deux interfaces : Classic UI et Volto
6. Cartographie des paquets et distributions

**Partie II — Installer et exploiter**

7. Prérequis matériels et logiciels
8. Installer avec Cookieplone
9. Installer avec pip et `cookiecutter-zope-instance`
10. Installer en conteneurs
11. Créer un site : les distributions
12. Démarrer, arrêter, déboguer
13. Installer des modules complémentaires
14. Comptes Zope et compte d'urgence
15. La ZODB au quotidien : stockages, blobs, pack, sauvegarde
16. Exporter et importer un site

**Partie III — Utiliser et administrer un site**

17. Visite de Classic UI
18. Les types de contenu livrés
19. Workflows, états et partage
20. Utilisateurs, groupes, rôles et permissions
21. Configuration du site et registre
22. Règles de contenu, historique, verrous, copies de travail, redirections
23. Modules du cœur : multilinguisme, commentaires, copies de travail, cache

**Partie IV — Développer le backend**

24. Créer un module complémentaire
25. L'architecture à composants et le ZCML
26. GenericSetup : profils, étapes d'import, étapes de migration
27. Types de contenu Dexterity et FTI
28. Schémas, champs et directives de formulaire
29. Comportements (behaviors)
30. Vocabulaires
31. Widgets et formulaires `z3c.form`
32. Relations
33. Catalogue et indexation
34. Événements et abonnés
35. Annotations
36. Registre de configuration et panneaux de configuration
37. Sécurité : permissions, rôles, workflows DCWorkflow
38. `plone.api`
39. Courriel, journalisation, scripts de maintenance
40. Tests : `plone.app.testing` et `pytest-plone`
41. Internationalisation

**Partie V — Classic UI**

42. Vues
43. Gabarits et variables globales
44. Couches (layers)
45. Viewlets
46. Portlets
47. Ressources statiques, Mockup et Module Federation
48. Thème : réglages TTW, add-on Barceloneta, Diazo, modes de couleur
49. Icônes et images adaptatives
50. TinyMCE 8
51. Protection CSRF

**Partie VI — REST API et Volto**

52. `plone.restapi`
53. Volto en bref

**Partie VII — Production**

54. Composants d'un déploiement
55. Cache HTTP et `plone.app.caching`
56. Conteneurs et orchestration
57. Réglages de performance
58. Mettre à jour Plone : stratégies et guides 6.0, 6.1, 6.2
59. Espaces de noms natifs et outillage Python en 6.2

**Partie VIII — Travaux pratiques**

60. TP1 — Un site Classic UI avec Cookieplone
61. TP2 — Un module complémentaire et un type « Conférence »
62. TP3 — Un comportement, un vocabulaire, un panneau de configuration
63. TP4 — Une vue, un viewlet et un thème
64. TP5 — Un workflow sur mesure
65. TP6 — Une étape de migration
66. TP7 — Consommer la REST API
67. TP8 — Déployer avec Docker Compose et Varnish
68. TP9 — Migrer un projet 6.1 vers 6.2
69. Erreurs fréquentes
70. Commandes de référence
71. Ce qu'il faut retenir
72. Références

# Partie I — Comprendre Plone

# 1. Qu'est-ce que Plone ?

Plone est un système de gestion de contenu (CMS) open source écrit en Python, publié pour la première fois le 4 octobre 2001 et porté par la Plone Foundation, sous licence GPL. Trois traits le distinguent des CMS grand public :

- **la sécurité est dans l'architecture** : chaque objet porte ses permissions, héritées ou locales, vérifiées par le serveur d'applications à chaque accès, et non par une couche applicative ajoutée après coup ;
- **le contenu est un arbre d'objets** persistés dans une base objet (ZODB), ce qui rend naturels les sites hiérarchiques, les droits par section et les contenus hétérogènes ;
- **tout est composant** : vues, formulaires, comportements, utilitaires sont enregistrés dans une architecture à composants et se remplacent sans modifier le code d'origine.

Plone vise les organisations qui ont besoin de workflows de publication, de droits fins, de multilinguisme et d'accessibilité (WCAG 2.1 AA) : administrations, universités, ONG, intranets. Il sert aussi de cadre (framework) pour bâtir des applications de type CMS, et, depuis Plone 5.2, de serveur « headless » grâce à son API REST.

Plone 6 se présente sous trois formes :

- un CMS complet rendu côté serveur (**Classic UI**) ;
- un CMS avec frontend React (**Volto**) adossé au même serveur Python via l'API REST ;
- un serveur de contenu sans interface, consommé par un frontend maison.

# 2. Histoire et versions

| Version | Publication | Python | Zope | Marqueur |
|---|---|---|---|---|
| 2.x, 3.x | 2004 – 2008 | 2.4 – 2.6 | 2.8 – 2.10 | Archetypes, CMF, GenericSetup (3.0) |
| 4.x | 2010 – 2013 | 2.6 / 2.7 | 2.13 | Dexterity dans le cœur (4.3), Diazo, éditeur de thème |
| 5.0 | septembre 2015 | 2.7 | 2.13 | Barceloneta responsive, barre d'outils, Dexterity par défaut, registre, `plone.api` |
| 5.1 | mars 2018 | 2.7 | 2.13 | bundles, indexation optimisée, HiDPI |
| 5.2 | juillet 2019 | 2.7 et 3.6 – 3.8 | 4 | Python 3, WSGI, `plone.restapi` dans le cœur |
| 6.0 | décembre 2022 | 3.8 – 3.11 | 5 | Volto, Bootstrap 5, fin d'Archetypes, racine Dexterity, pip |
| 6.1 | février 2025 | 3.10 – 3.13 | 5 | distributions, TinyMCE 7, modules du cœur |
| 6.2 | mai 2026 | 3.10 – 3.14 | 6 | espaces de noms natifs, TinyMCE 8, séparation Classic UI / cœur, Volto 19 |

Plone suit le versionnement sémantique : une version majeure peut casser des extensions, une version mineure ajoute des fonctions sans rupture, une version corrective ne corrige que des bugs. La politique de support promet des correctifs de sécurité pendant cinq ans après une version majeure, d'où la date de fin 2027 pour 6.0 et 6.1. La feuille de route de Plone 7 annonce une séparation nette entre le cœur (`Products.CMFPlone`) et Classic UI (`plone.classicui`), déjà amorcée en 6.2.

# 3. La pile technique

Plone est le sommet d'une pile de six couches. Les connaître évite la plupart des confusions.

```text
Volto (React, Node.js)            Classic UI (ZPT, Mockup, Bootstrap 5)
        │ HTTP/JSON                         │ HTML
        ▼                                   ▼
plone.restapi ──────────────┐   plone.app.layout, plonetheme.barceloneta
                            ▼
                     Plone (Products.CMFPlone, plone.*, plone.app.*)
                            │
                     plone.base (contrats, interfaces, utilitaires)
                            │
                     CMF (Products.CMFCore, DCWorkflow, GenericSetup)
                            │
                     Zope (ZPublisher, AccessControl, ZCA, Chameleon)
                            │
                     ZODB (FileStorage, ZEO, RelStorage, blobs)
                            │
                     Python 3.10 – 3.14
```

## 3.1 Python

Le serveur est un processus Python unique, multi-threadé (Waitress par défaut), servi par WSGI depuis Plone 5.2. Toute la logique, du routage au rendu, est du Python ordinaire : un débogueur `pdb` fonctionne partout.

## 3.2 La ZODB

La **Zope Object Database** persiste directement des objets Python, sans ORM ni schéma relationnel : une instance de classe héritant de `persistent.Persistent` est sauvegardée telle quelle, attributs compris, par sérialisation (`pickle`). Ses propriétés :

- **transparence** : la transaction démarre avec la requête et se valide avec la réponse ; aucun `save()` explicite dans une vue ;
- **ACID** avec contrôle de concurrence multi-versions (MVCC) ; un conflit d'écriture provoque un `ConflictError`, Zope rejoue alors la requête ;
- **annulation** (`undo`) et historique tant que la base n'est pas « packée » ;
- **stockages** interchangeables : `FileStorage` (un fichier `Data.fs`, un seul processus), **ZEO** (serveur de stockage partagé par plusieurs clients Zope), **RelStorage** (PostgreSQL, MySQL, Oracle comme stockage sous-jacent, très utilisé en conteneurs) ;
- **blobs** : les fichiers binaires (images, documents) sont stockés hors du `Data.fs`, dans un répertoire `blobstorage`.

Le pendant de la ZODB est le **catalogue** (`portal_catalog`, un `ZCatalog`) : des index et des colonnes de métadonnées qui permettent de retrouver des objets sans les charger (« sans réveiller les objets »).

## 3.3 Zope

Zope est le serveur d'applications : il publie les objets de la ZODB sur HTTP (**ZPublisher**), vérifie la sécurité à chaque accès d'attribut (**AccessControl**), fournit les gabarits (**Chameleon** via `zope.pagetemplate`), l'interface d'administration (**ZMI**) et, surtout, l'architecture à composants.

## 3.4 La Zope Component Architecture (ZCA)

La ZCA est un cadre de programmation par composants fondé sur quatre motifs de conception :

| Notion | Rôle | Motif |
|---|---|---|
| **Interface** | contrat abstrait (`zope.interface.Interface`) ; une classe « fournit » une interface, un objet « implémente » | interface / protocole, marqueur |
| **Adaptateur** | implémentation d'une interface pour un ou plusieurs objets d'autres interfaces ; `IFoo(obj)` cherche l'adaptateur | adaptateur, fabrique abstraite |
| **Utilitaire** | singleton ou fabrique enregistré sous une interface et éventuellement un nom | registre de services |
| **Événements et abonnés** | `zope.event.notify(evt)` appelle tous les abonnés enregistrés pour l'interface de l'événement | publication/abonnement |

Les enregistrements se font en Python ou, le plus souvent, en **ZCML**, un dialecte XML (`configure.zcml`). Il existe deux niveaux de registre : le registre **global**, construit au démarrage à partir du ZCML, et un registre **local** par site Plone, persistant, consulté en premier lors du traversal. Les vues, viewlets, portlets, comportements, vocabulaires et panneaux de configuration ne sont que des composants nommés.

## 3.5 CMF

Le **Content Management Framework** ajoute à Zope les briques d'un CMS : outils de site (`portal_types`, `portal_workflow`, `portal_catalog`, `portal_setup`), métadonnées Dublin Core, **DCWorkflow** (machine à états), **GenericSetup** (configuration déclarative en XML).

## 3.6 Plone

Plone assemble tout cela en un produit fini : types de contenu Dexterity, formulaires, registre de configuration, thème, API REST, interface d'édition. Depuis Plone 6, le paquet `plone.base` sert de frontière entre le serveur d'applications et le cœur de Plone ; `Products.CMFPlone` est le paquet « où tout converge » ; le méta-paquet `Plone` tire tout, piles incluses.

# 4. Concepts fondamentaux

## 4.1 Traversal

Le **traversal** est la résolution d'une URL en objet : Zope part de la racine et descend l'arbre segment par segment (`/Plone/actualites/mon-article`), chaque conteneur fournissant le suivant. Le dernier segment peut désigner une **vue** ; le préfixe `@@` force la recherche d'une vue plutôt que d'un contenu homonyme (`/mon-article/@@edit`). Sans vue explicite, la vue par défaut du type est rendue. En code, `context.restrictedTraverse("chemin")` applique les contrôles de sécurité ; `unrestrictedTraverse` les contourne, à la charge du développeur.

## 4.2 Acquisition

L'**acquisition** permet à un objet d'« acquérir » les attributs de ses parents : si `A` contient `B` et que `A` a un attribut `x`, alors `B.x` renvoie celui de `A`. C'est ainsi que `context.portal_catalog` fonctionne depuis n'importe quelle profondeur. Deux règles de survie :

- `aq_inner(obj)` et `aq_parent(obj)` donnent le vrai parent de conteneur ; `aq_base(obj)` retire tout emballage pour tester l'existence réelle d'un attribut : `getattr(aq_base(obj), "x", None)` ;
- ne stockez jamais un objet acquis dans un attribut de vue sans `aq_inner`, sous peine de corrompre sa chaîne d'acquisition.

Les outils s'obtiennent de préférence par `plone.api.portal.get_tool("portal_catalog")` ou par utilitaire, plus explicites que l'acquisition implicite.

## 4.3 Contenu piloté par schéma

Un type de contenu est décrit par un **schéma** `zope.schema` : une interface dont les attributs sont des **champs** (`TextLine`, `RichText`, `Choice`, `RelationList`…). Le schéma génère les formulaires d'ajout et d'édition, sérialise vers l'API REST, valide les valeurs. Des **directives** (`plone.autoform`, `plone.supermodel`) ajoutent les informations de présentation (widget, ordre, fieldset, permissions).

## 4.4 Composants partout

Vue = adaptateur multiple de `(context, request)` nommé. Viewlet = adaptateur de `(context, request, view, manager)`. Comportement = interface fournie dynamiquement par l'objet plus adaptateur éventuel. Vocabulaire = utilitaire nommé. Comprendre « quel composant, pour quelles interfaces, sous quel nom, sur quelle couche » résout 90 % des questions de personnalisation.

# 5. Deux interfaces : Classic UI et Volto

| Critère | Classic UI | Volto |
|---|---|---|
| Rendu | côté serveur, gabarits ZPT/Chameleon dans le processus Python | application React monopage avec rendu serveur Node.js et hydratation |
| Édition | TinyMCE (WYSIWYG), portlets pour les blocs latéraux | éditeur par **blocs** (texte, image, listing, grilles…) |
| Stockage du corps | HTML (`RichText`) | JSON (blocs) |
| Hébergement | un seul processus | deux processus (Python 8080 + Node 3000), un proxy devant |
| Compétences | Python, ZPT, Bootstrap 5, Mockup | React, TypeScript, pnpm, et Python pour le backend |
| Thème | SCSS Barceloneta, Diazo | Volto Light Theme, composants React |
| Documentation utilisateur | manuel Plone 5 toujours valable | manuel Volto en construction |
| Migration | Classic → Volto possible | Volto → Classic impossible |

Classic UI est le bon choix pour un site existant, une équipe Python, un hébergement simple ou des extensions historiques (formulaires, portlets, thèmes Diazo). Volto s'impose pour un nouveau site éditorial riche en mises en page. Les deux reposent sur le même backend, les mêmes types, workflows et droits.

# 6. Cartographie des paquets et distributions

Plone, c'est plus de 200 paquets Python. Un modèle mental à trois étages :

- **Au-dessus de `Products.CMFPlone`** : les API (`plone.api`, `plone.restapi`), la migration (`plone.app.upgrade`), les **distributions** (`plone.distribution`, `plone.volto`, `plone.classicui`), l'export-import (`plone.exportimport`) et les **modules du cœur** (`plone.app.multilingual`, `plone.app.discussion`, `plone.app.iterate`, `plone.app.caching`, `Products.CMFPlacefulWorkflow`). Le méta-paquet `Plone` dépend de tous.
- **Entre `Products.CMFPlone` et `plone.base`** : le cœur fonctionnel (`plone.app.contenttypes`, `plone.app.dexterity`, `plone.app.layout`, `plone.app.registry`, `plone.app.z3cform`, `plone.app.theming`, `plone.staticresources`, `plonetheme.barceloneta`, `plone.protect`…).
- **Sous `plone.base`** : les bibliothèques génériques Plone (`plone.dexterity`, `plone.behavior`, `plone.registry`, `plone.supermodel`, `plone.autoform`, `plone.scale`…), l'écosystème Zope/CMF et les bibliothèques tierces (Chameleon, lxml, Pillow, waitress…).

Conséquence pratique pour le choix des dépendances d'un projet :

| Vous voulez… | Dépendez de… |
|---|---|
| tout Plone, « piles incluses », les deux distributions | `Plone` |
| un site Volto seulement | `plone.volto` (+ `plone.restapi`, `plone.api`) |
| un site Classic UI seulement | `plone.classicui` (+ `plone.app.caching`, `plone.app.upgrade`…) |
| le strict cœur, sans distribution ni modules du cœur | `Products.CMFPlone` |

Une **distribution** (depuis 6.1) est un paquet qui décrit un site prêt à l'emploi : profils GenericSetup à appliquer, contenus d'exemple, formulaire de création. Les deux distributions livrées sont `default` (Volto) et `classic` (Classic UI). La variable `ALLOWED_DISTRIBUTIONS=classic` restreint le choix proposé à la création d'un site.

# Partie II — Installer et exploiter

# 7. Prérequis matériels et logiciels

**Matériel** (ordres de grandeur de la documentation officielle) :

- backend et Classic UI : 256 Mo de RAM minimum pour l'installation, 2 Go recommandés par site en exploitation ;
- frontend Volto : 2 Go de RAM minimum pour construire le bundle ;
- disque : 512 Mo minimum, 40 Go recommandés (la ZODB et les blobs croissent avec l'historique).

**Logiciel** :

- un système UNIX (Linux, macOS, WSL sous Windows) ; Windows seul n'est pas recommandé faute de `make` ;
- **uv** (installation de Python et des paquets) : `curl -LsSf https://astral.sh/uv/install.sh | sh` ;
- **GNU make** et **git** ;
- pour Volto seulement : **nvm**, **Node.js** LTS, **corepack** pour obtenir **pnpm** ;
- aucun paquet `-dev` système n'est nécessaire pour les 16 extensions C de Plone 6.2 : des wheels manylinux existent pour Python 3.10 à 3.14. Si `make install` échoue sur Pillow, installez `zlib1g-dev libjpeg8-dev` (Debian/Ubuntu) ou `zlib libjpeg` (Homebrew).

> [!warning] Jamais `root`
> Créez un utilisateur dédié (`plone`) pour installer et faire tourner l'instance. Toute la pile fonctionne sans privilèges.

Navigateurs pris en charge pour Classic UI : ceux de Bootstrap 5.3 (Chrome et Firefox ≥ 60, Safari et iOS ≥ 12, pas d'Internet Explorer).

# 8. Installer avec Cookieplone

Cookieplone est le générateur de projet recommandé depuis Volto 18 et Plone 6.1. Il produit un **mono-dépôt** `backend/` (+ `frontend/` pour Volto) avec `Makefile`, configuration de CI, images Docker, et optionnellement Ansible et un scaffold de documentation.

```bash
uvx cookieplone project
```

Les questions posées, dans l'ordre : titre, description, slug du dépôt, URL, auteur, nom du paquet Python (`project.title`), versions pré-release ou non, version de Plone (`6.2.0`), **« Use Volto as frontend » (répondre `No` pour Classic UI)**, version de Volto et nom de l'add-on Volto, langue, organisation GitHub/GitLab, registre de conteneurs, stockage persistant (RelStorage + PostgreSQL recommandé, ZEO, ou FileStorage local), serveur de cache, Ansible, GitHub Action de déploiement, scaffold de documentation. Les options de ligne de commande évitent les invites ; le slug ne doit pas réutiliser un nom de paquet du cœur.

Puis :

```bash
cd project-title
make install          # backend (uv sync + cookiecutter-zope-instance + création du site) et frontend
make backend-start    # http://localhost:8080/Plone  (admin / admin)
make frontend-start   # Volto seulement, dans un second terminal : http://localhost:3000
```

Le journal de démarrage se termine par `Ready to handle requests` puis `Serving on http://127.0.0.1:8080`. En Classic UI on y lit aussi la ligne `🐎 This Python uses horse-with-no-namespace…` : voir §59.

Structure du backend généré :

```text
backend/
├── Makefile              # install, start, console, test, format, lint, i18n, build-image…
├── pyproject.toml        # dépendances du projet (uv), outils de qualité
├── instance.yaml         # contexte pour cookiecutter-zope-instance (port, zcml_package_includes…)
├── mx.ini                # sources git des paquets en développement (mxdev)
├── constraints.txt / requirements.txt
├── src/project/title/    # le module complémentaire du projet (politique de site)
│   ├── configure.zcml, dependencies.zcml, permissions.zcml
│   ├── profiles/default/  (metadata.xml, registry/, types/, workflows/…)
│   ├── upgrades/, setuphandlers.py, interfaces.py, testing.py
│   └── locales/
├── instance/             # généré, non versionné : etc/zope.conf, etc/zope.ini, var/
└── scripts/ (create_site.py, …)
```

Les cibles `make` utiles au quotidien (référence « Cookieplone make commands ») :

| Cible | Effet |
|---|---|
| `install` | `backend-install` + `frontend-install` |
| `backend-install` | crée le venv, installe, puis `backend-create-site` |
| `backend-build` | réinstalle le backend (`uv sync` + génération de `instance/`) sans recréer le site |
| `backend-create-site` | crée un site avec le contenu par défaut |
| `backend-update-example-content` | exporte le contenu du site vers le dossier de contenu de la distribution |
| `backend-start` / `frontend-start` | démarre en avant-plan sur 8080 / 3000 |
| `-C backend console` | console `zconsole debug` |
| `format`, `lint`, `check` | qualité de code (Ruff, etc.) |
| `i18n` | met à jour les catalogues de traduction |
| `test`, `backend-test`, `frontend-test` | tests unitaires |
| `acceptance-*`, `ci-acceptance-test` | tests d'acceptation (Playwright/Cypress côté Volto) |
| `build-images` | images Docker backend et frontend |
| `-C devops help` | cibles de déploiement (Ansible, GitHub Actions, secrets) |

# 9. Installer avec pip et `cookiecutter-zope-instance`

Pour un backend Classic UI minimal sans outillage, ou pour comprendre ce que Cookieplone automatise :

```bash
mkdir -p ~/projets/plone && cd ~/projets/plone
python3 -m venv venv
venv/bin/pip install -c https://dist.plone.org/release/6-latest/constraints.txt Plone pipx
```

Le fichier de **contraintes** publié sur `dist.plone.org` épingle les ~200 versions testées ensemble : ne jamais installer Plone sans lui. Puis décrire l'instance dans `instance.yaml` :

```yaml
default_context:
  initial_user_name: "admin"
  initial_user_password: "admin"     # jeton robuste en production
  wsgi_listen: "localhost:8080"
  debug_mode: false
  verbose_security: false
  db_storage: "direct"               # FileStorage ; "zeo" ou "relstorage" possibles
  environment: {
    "zope_i18n_compile_mo_files": true,
  }
```

et générer `instance/` :

```bash
venv/bin/pipx run cookiecutter -f --no-input --config-file instance.yaml gh:plone/cookiecutter-zope-instance
venv/bin/runwsgi -v instance/etc/zope.ini     # avant-plan
```

À `http://localhost:8080`, le bouton « Create Classic UI Plone site » crée le site. Pour développer des paquets depuis git, ajouter **mxdev** et un `mx.ini` ; pour surcharger une contrainte du cœur, `version-overrides` dans `[settings]` de `mx.ini`. Ces mécanismes, le `Makefile` et le lot hors ligne pour un serveur sans Internet sont détaillés dans [[Plone pyproject tooling]].

> [!note] Buildout
> `zc.buildout` reste pris en charge pour Classic UI (pas pour Volto), en version **5** pour Plone 6.2 à cause des espaces de noms natifs (§59). `plone.recipe.zope2instance` y joue le rôle de `cookiecutter-zope-instance`. Voir [[ZC.Buildout]].

# 10. Installer en conteneurs

Les images officielles, conformes OCI, embarquent Plone et ses prérequis :

```bash
docker run --name plone6-backend -e SITE=Plone -d -p 8080:8080 plone/plone-backend:6.2
docker run --name plone6-frontend --link plone6-backend:backend \
  -e RAZZLE_DEV_PROXY_API_PATH=http://backend:8080/Plone -d -p 3000:3000 plone/plone-frontend:latest
```

Variables d'environnement de `plone/plone-backend` les plus utiles :

| Variable | Rôle |
|---|---|
| `SITE`, `TYPE=classic\|volto`, `PROFILES`, `DELETE_EXISTING` | création du site au premier démarrage (hors production) |
| `ADDONS="pkg==1.0 autre"`, `DEVELOP="/app/src/mon.paquet"`, `PIP_PARAMS` | installation d'extensions au démarrage (hors production : préférer une image dérivée) |
| `ZEO_ADDRESS=zeo:8100`, `ZEO_*` | client ZEO |
| `RELSTORAGE_DSN="dbname='plone' user='plone' host='db' password='…'"`, `RELSTORAGE_*` | RelStorage PostgreSQL |
| `ZODB_CACHE_SIZE` (50000), `LISTEN_PORT` (8080), `DEBUG_MODE`, `VERBOSE_SECURITY` | réglages Zope |
| `CORS_ALLOW_ORIGIN`, `CORS_*` | CORS pour un frontend sur un autre hôte |

Image dérivée avec fragments de configuration :

```dockerfile
FROM plone/plone-backend:6.2
COPY etc/zope.conf.d/*.conf /app/etc/zope.conf.d/
COPY etc/package-includes/*.zcml /app/etc/package-includes/
RUN /app/bin/pip install -c /app/constraints.txt collective.easyform
```

Les exemples Docker Compose de la documentation couvrent Traefik, nginx et HAProxy, avec ZEO, PostgreSQL ou Varnish. Voir §56 et [[Docker]].

# 11. Créer un site : les distributions

Depuis 6.1, l'écran d'accueil du backend (`http://localhost:8080`) propose de **choisir une distribution** (Volto ou Classic UI) puis un formulaire : identifiant de chemin, titre, description, logo, langue, fuseau horaire, et, pour certaines distributions, « créer le contenu d'exemple ». Le site créé contient alors les dossiers Actualités et Événements, les règles de contenu et le cache configurés.

Sans distribution (projet ne dépendant que de `Products.CMFPlone`), le formulaire « avancé » historique reste disponible : site vide, modules à activer à la main.

Créer sa propre distribution (§24 pour le paquet) :

```xml
<configure xmlns="http://namespaces.zope.org/zope"
           xmlns:plone="http://namespaces.plone.org/plone">
  <include package="plone.distribution" />
  <plone:distribution
      name="intranet"
      title="Intranet d'entreprise"
      description="Site pré-configuré avec les modules maison."
      directory="distributions/intranet"
      post_handler=".handlers.intranet.post_handler"
      />
</configure>
```

Le dossier `distributions/intranet/` contient `image.png` (1080×768), `profiles.json` (profils `base` et `content`), `schema.json` (formulaire JSON Schema + uischema, avec au moins `site_id`, `title`, `description`, `default_language`, `portal_timezone`, `setup_content`) et un dossier `content/` produit par `bin/export-distribution instance/etc/zope.conf Plone`. Les trois crochets `pre_handler(answers)`, `handler(distribution, site, answers)` et `post_handler(distribution, site, answers)` permettent d'ajuster les réponses, de remplacer l'installation par défaut, ou de terminer la configuration.

# 12. Démarrer, arrêter, déboguer

| Besoin | Cookieplone | pip | Buildout |
|---|---|---|---|
| avant-plan | `make backend-start` | `bin/runwsgi instance/etc/zope.ini` | `bin/instance fg` |
| service en arrière-plan | systemd / conteneur | `runwsgi` sous systemd, ou `zdaemon` | `bin/instance start` / `stop` |
| console Python avec `app` | `make -C backend console` | `bin/zconsole debug instance/etc/zope.conf` | `bin/instance debug` |
| exécuter un script | `zconsole run instance/etc/zope.conf script.py` | idem | `bin/instance run script.py` |

Dans la console, `app` est la racine Zope et `app.Plone` le site. Deux pièges classiques d'un script lancé hors requête :

```python
from Testing.makerequest import makerequest
from zope.globalrequest import setRequest
from AccessControl.SecurityManagement import newSecurityManager
from AccessControl.users import system
import transaction

app = makerequest(app)           # fournit une requête aux composants qui en exigent une
setRequest(app.REQUEST)
newSecurityManager(None, system)  # sinon tout accès protégé lève Unauthorized
site = app.Plone
# ... travail ...
transaction.commit()             # rien n'est validé sans commit explicite hors requête
```

Les journaux sont dans `instance/var/log/instance.log` et `access.log` (`make log` dans certains projets). `debug_mode: true` dans `instance.yaml` active le rechargement des gabarits et les traces complètes ; `verbose_security: true` explique chaque refus de permission dans le journal.

# 13. Installer des modules complémentaires

Un module complémentaire (add-on) backend est un paquet Python. Trois étapes dans un projet Cookieplone ou pip :

1. Déclarer la dépendance dans `backend/pyproject.toml` :

```toml
dependencies = [
    "Products.CMFPlone==6.2.0",
    "plone.api",
    "plone.classicui",
    "plone.app.caching",
    "collective.easyform==5.0.0",
]
```

2. Faire charger son ZCML dans `backend/instance.yaml` :

```yaml
default_context:
    zcml_package_includes: project_title, collective.easyform
```

   Cette ligne n'est nécessaire que si le paquet ne déclare pas le point d'entrée `plone.autoinclude.plugin` (voir §24) : les paquets modernes se chargent seuls.

3. Reconstruire et redémarrer : `make backend-build`, puis activer dans **Configuration du site ‣ Modules complémentaires** (`/prefs_install_products_form`). L'activation applique le profil GenericSetup du module dans le site ; un module peut être chargé dans Zope sans être activé dans un site donné.

Depuis une source git non publiée, ajouter dans `mx.ini` :

```ini
[collective.easyform]
url=git@github.com:collective/collective.easyform.git
branch=main
extras=test
```

`make backend-build` clone dans `sources/` et installe en mode éditable (avec uv, une entrée `[tool.uv.sources]` est ajoutée à `pyproject.toml`). Pour forcer la version d'un paquet du cœur : `override-dependencies` sous `[tool.uv]`, ou `version-overrides` dans `mx.ini` ; la combinaison n'est alors plus celle testée par l'équipe Plone.

# 14. Comptes Zope et compte d'urgence

Un **utilisateur Zope** vit dans le dossier `acl_users` de la racine, hors de tout site, avec le rôle Manager global : c'est le compte d'administration initial et le compte d'urgence si l'accès est perdu. Instance arrêtée :

```bash
.venv/bin/addzopeuser -c instance/etc/zope.conf admin 'mot-de-passe'     # pip / Cookieplone
bin/instance adduser admin 'mot-de-passe'                                   # Buildout
```

Un utilisateur existant n'est pas modifié (« Created user: None »). Les utilisateurs du site, eux, sont gérés par **PAS** (`Products.PluggableAuthService`) dans `Plone/acl_users` : sources de comptes enfichables (ZODB, LDAP via `pas.plugins.ldap`, OIDC via `pas.plugins.oidc` ou `authomatic`), voir [[LDAP]] et [[OAuth OpenID]].

# 15. La ZODB au quotidien

- **FileStorage** : `instance/var/filestorage/Data.fs` + `Data.fs.index` ; blobs dans `instance/var/blobstorage/`. Un seul processus peut l'ouvrir ; un verrou `Data.fs.lock` le signale.
- **ZEO** : un serveur `runzeo` et N clients Zope ; permet plusieurs processus (montée en charge, scripts en parallèle de l'instance). Blobs partagés par répertoire commun ou transférés par le réseau.
- **RelStorage** : objets stockés dans PostgreSQL, blobs en base ou sur disque partagé ; choix recommandé par Cookieplone pour les conteneurs.
- **Pack** : la ZODB conserve toutes les révisions d'objets ; `pack` supprime celles plus anciennes que N jours et libère l'espace. Depuis la ZMI (`Control_Panel/Database`), par `zodbpack`, ou par le panneau Maintenance de Plone (`@@maintenance-controlpanel`, intervalle en jours). **Le pack est irréversible : il supprime la possibilité d'`undo`.** Ordonnez les opérations destructives de la moins à la plus irréversible, et packez en dernier.
- **Sauvegarde** : copier `Data.fs` à chaud est possible (`repozo` fait des sauvegardes incrémentales fiables de FileStorage) ; les blobs se sauvegardent avec `rsync` ; avec RelStorage, `pg_dump` et le répertoire de blobs. Testez la restauration.
- **Cache** : `zodb-cache-size` (objets par connexion) et le nombre de threads sont les deux réglages de performance de base (§57).

# 16. Exporter et importer un site

`plone.exportimport` (dépendance de `plone.distribution`) exporte et importe contenus, utilisateurs et groupes, relations, traductions, commentaires et redirections, dans un format JSON diff-able avec les blobs à côté de chaque contenu :

```bash
.venv/bin/plone-exporter [--include-revisions] instance/etc/zope.conf Plone /tmp/export/
.venv/bin/plone-importer instance/etc/zope.conf Plone /tmp/export/
```

Arborescence produite : `content/<UID>/data.json` et fichiers blob, `content/plone_site_root/`, `principals.json`, `relations.json`, `translations.json`, `discussions.json`, `redirects.json`. Depuis 6.2, les services REST `@export` et `@import` exposent la même fonction. Pour migrer un vieux site (Plone 4 ou 5, Archetypes, Python 2), l'outil est `collective.exportimport`, qui écrit un format compatible.

# Partie III — Utiliser et administrer un site

# 17. Visite de Classic UI

- **Barre d'outils** (à gauche ou en haut) : actions de l'objet (Contenus, Voir, Modifier, Règles, Partage), menus Affichage, Ajouter, État (workflow), Actions (copier, couper, renommer, supprimer, URL alternatives), menu personnel (Préférences, Configuration du site, Déconnexion).
- **Contenu du dossier** (`/folder_contents`) : tableau filtrable et triable, sélection multiple, actions en masse (workflow, mots-clés, propriétés, couper-coller, renommage), réorganisation par glisser-déposer, choix des colonnes, envoi de fichiers.
- **Navigation** : onglets globaux générés depuis la racine (menus déroulants configurables), fil d'Ariane, arbre de navigation en portlet, plan du site, recherche instantanée.
- **Édition** : formulaire par onglets (Défaut, Catégorisation, Dates, Propriétaires, Paramètres), TinyMCE pour le texte riche, navigateur de contenus en colonnes pour les liens et images internes, verrouillage automatique pendant l'édition.
- **Configuration du site** (`/@@overview-controlpanel`) : panneaux Général (Site, Langue, Courriel, Navigation, Actions, Modules…), Contenu (Types de contenu, Édition, Filtrage HTML, Images, TinyMCE, Balisage), Utilisateurs (Utilisateurs et groupes, Inscription), Sécurité, Avancé (Cache, Maintenance, Registre, Relations, Thème, URL alternatives, Interface de gestion Zope).

Vues de test du thème disponibles sur tout site : `/@@test-rendering`, `/@@test-rendering-cheatsheet` (tous les composants Bootstrap), `/@@test-rendering-icons`.

# 18. Les types de contenu livrés

| Type (`portal_type`) | Conteneur ? | Usage |
|---|---|---|
| `Document` (Page) | non (Classic) / oui (Volto) | page de texte riche ; en Volto, pages à blocs |
| `Folder` (Dossier) | oui | hiérarchie, page par défaut, vues de liste |
| `News Item` | non | actualité avec image à la une |
| `Event` | non | événement daté, lieu, récurrence (`plone.app.event`), export iCal |
| `File`, `Image` | non | blobs ; échelles d'images automatiques |
| `Link` | non | lien interne ou externe, redirection pour les anonymes |
| `Collection` | non | liste dynamique par critères de catalogue (`plone.app.querystring`) |

Chaque type possède des **vues** sélectionnables (liste, résumé, tableau, album, vue complète), une **page par défaut** possible pour un dossier, et des **comportements** (§29) qui apportent les champs Dublin Core, l'exclusion de la navigation, les éléments liés, le versionnement, la table des matières, l'image à la une… Le panneau **Types de contenu** permet de créer un type dans le navigateur (TTW), d'y ajouter des champs et d'exporter son modèle XML, ce qui fait un excellent prototype avant le passage en code (§27).

# 19. Workflows, états et partage

Un **workflow** est une machine à états attachée à un type : chaque **état** fixe les permissions sur l'objet (qui voit, qui modifie), chaque **transition** mène d'un état à l'autre sous condition (permission, rôle, expression). Workflows livrés :

| Workflow | États | Usage |
|---|---|---|
| `simple_publication_workflow` (défaut) | privé → en attente → publié | relecture par un Reviewer |
| `one_state_workflow` | publié | contenu toujours visible |
| `intranet_workflow` | privé, visible en interne, en attente, publié en interne, publié en externe | intranet |
| `folder_workflow`, `comment_review_workflow`, `plone_workflow` | variantes historiques | — |

Points essentiels :

- les permissions sont **recalculées au changement d'état**, pas en temps réel : après modification d'un workflow, cliquer **Update security settings** dans `portal_workflow` (ZMI) ou lancer `portal_workflow.updateRoleMappings()` ;
- la publication peut être planifiée par les dates de **prise d'effet** et d'**expiration** ;
- le **partage** (`/@@sharing`) attribue des **rôles locaux** (Lecteur, Éditeur, Contributeur, Relecteur) à des utilisateurs ou groupes sur un objet, hérités par les enfants sauf blocage ;
- `Products.CMFPlacefulWorkflow` (module du cœur) permet des politiques de workflow par sous-arbre.

# 20. Utilisateurs, groupes, rôles et permissions

- **Permissions** : chaînes Zope (« View », « Modify portal content », « Add portal content », « Review portal content », « Manage portal »…), chacune associée, par objet, à une liste de rôles. Les extensions déclarent les leurs en ZCML (§37).
- **Rôles globaux** : Anonymous, Authenticated, Member, Reader, Contributor, Editor, Reviewer, Site Administrator, Manager, Owner. Un Site Administrator administre le site sans accéder à la ZMI ni au code.
- **Rôles locaux** : attribués par le partage sur un sous-arbre ; `borg.localrole` permet de les calculer dynamiquement.
- **Groupes** : Administrators, Site Administrators, Reviewers, Authenticated Users, plus les vôtres ; les rôles s'attribuent de préférence aux groupes.
- **Panneaux** Utilisateurs et groupes, Inscription (auto-inscription, mot de passe par courriel, courriel comme identifiant), Sécurité (champ « Utiliser l'adresse électronique comme identifiant », activation des comptes, filtrage HTML).

# 21. Configuration du site et registre

Depuis Plone 5, tous les réglages globaux sont des **enregistrements du registre** (`plone.registry`), typés par un champ de schéma et nommés avec un préfixe (`plone.site_title`, `plone.displayed_types`, `plone.smtp_host`…). Les panneaux de configuration ne sont que des formulaires sur ces enregistrements ; le panneau **Registre de configuration** (`/portal_registry`) permet de chercher, modifier, importer et exporter n'importe quel enregistrement. En code :

```python
from plone import api
api.portal.get_registry_record("plone.displayed_types")
api.portal.set_registry_record("plone.smtp_host", "smtp.example.org")
```

L'export GenericSetup (ZMI ‣ `portal_setup` ‣ Export ‣ « Export the configuration registry schemata ») produit un `registry.xml` dont on recopie les extraits utiles dans le profil d'un module (§36).

# 22. Règles de contenu, historique, verrous, copies de travail, redirections

- **Règles de contenu** (`/@@rules-controlpanel`) : automatisations « quand tel événement (ajout, modification, transition…), si telles conditions (type, état, rôle…), alors telles actions (courriel, copie, déplacement, transition, journal) », assignables à un dossier avec héritage.
- **Historique et versions** : `plone.app.versioningbehavior` + `CMFEditions` conservent les versions des types versionnés (Page, Actualité, Événement, Lien) ; onglet Historique avec comparaison et restauration.
- **Verrouillage** : `plone.locking` pose un verrou à l'ouverture du formulaire d'édition ; un administrateur peut le voler.
- **Copies de travail** (`plone.app.iterate`, module du cœur) : « extraire » une copie d'un contenu publié, la modifier en privé, puis « réintégrer ».
- **Gestion des URL** (`/@@redirection-controlpanel`) : URL alternatives, redirections automatiques lors d'un renommage ou déplacement, import en masse.
- **Panneau Relations** (`/@@inspect-relations`) : inspection et réparation des relations et relations inverses.

# 23. Modules du cœur

Depuis 6.1, plusieurs fonctions ne sont plus tirées automatiquement par `Products.CMFPlone` mais par le méta-paquet `Plone` ; un projet qui ne dépend pas de `Plone` doit les déclarer :

| Module | Paquet | Remarque |
|---|---|---|
| Multilinguisme | `plone.app.multilingual` | dossiers par langue, groupes de traductions, négociation de langue |
| Commentaires | `plone.app.discussion` | **désactivé par défaut** ; à activer dans les modules complémentaires ; la migration 6.1 refuse de continuer si des commentaires existent sans le paquet |
| Copies de travail | `plone.app.iterate` | |
| Cache HTTP | `plone.app.caching` | voir §55 |
| Workflows placés | `Products.CMFPlacefulWorkflow` | |
| Migration | `plone.app.upgrade` | indispensable pour `@@plone-upgrade` |

# Partie IV — Développer le backend

# 24. Créer un module complémentaire

Toute personnalisation durable vit dans un **paquet Python** : jamais de code dans la ZMI en production. Deux générateurs :

- **Cookieplone** : `uvx cookieplone backend_addon` (ou le paquet `src/project/title` déjà créé dans un projet) ;
- **plonecli** (`bobtemplates.plone`) : `uvx --from plonecli plonecli create addon collective.conference`, puis `plonecli add content_type | behavior | view | viewlet | form | vocabulary | controlpanel | theme_barceloneta | mockup_pattern | restapi_service | upgrade_step` (`plonecli -l` liste tout).

Anatomie d'un module moderne (src-layout, `pyproject.toml`, espace de noms natif, voir [[Plone pyproject tooling]]) :

```text
collective.conference/
├── pyproject.toml
├── src/collective/conference/      # pas de __init__.py dans src/collective/ (PEP 420)
│   ├── __init__.py                 # MessageFactory, logger
│   ├── configure.zcml              # inclut dependencies.zcml, permissions.zcml, sous-dossiers
│   ├── dependencies.zcml           # <include package="plone.app.dexterity" /> etc.
│   ├── permissions.zcml
│   ├── interfaces.py               # couche de navigateur ICollectiveConferenceLayer
│   ├── setuphandlers.py            # post_install, uninstall
│   ├── testing.py                  # couches de test
│   ├── browser/                    # vues, viewlets, static/, overrides/ (z3c.jbot)
│   ├── content/, behaviors/, vocabularies/, controlpanel/, upgrades/
│   ├── profiles/default/           # metadata.xml, types.xml, types/, workflows/, registry/, rolemap.xml, browserlayer.xml, catalog.xml, actions.xml, viewlets.xml
│   ├── profiles/uninstall/
│   └── locales/
└── tests/
```

Le `pyproject.toml` doit déclarer le **point d'entrée** qui dit à Plone de charger le ZCML du paquet, et embarquer les fichiers non Python :

```toml
[project]
name = "collective.conference"
version = "1.0.0a1"
dependencies = ["Products.CMFPlone", "plone.api", "plone.app.dexterity"]
classifiers = ["Framework :: Plone", "Framework :: Plone :: 6.2", "Framework :: Plone :: Addon"]

[project.entry-points."plone.autoinclude.plugin"]
target = "plone"            # ajouter module = "collective.conference" si le nom diffère du paquet

[tool.setuptools.package-data]
"*" = ["*.zcml", "*.pt", "*.xml", "*.po", "*.mo", "*.css", "*.js", "*.svg", "*.png"]
```

Le `configure.zcml` racine :

```xml
<configure
    xmlns="http://namespaces.zope.org/zope"
    xmlns:genericsetup="http://namespaces.zope.org/genericsetup"
    xmlns:i18n="http://namespaces.zope.org/i18n"
    i18n_domain="collective.conference">

  <i18n:registerTranslations directory="locales" />
  <include file="dependencies.zcml" />
  <include file="permissions.zcml" />
  <include package=".browser" />
  <include package=".content" />
  <include package=".upgrades" />

  <genericsetup:registerProfile
      name="default"
      title="collective.conference"
      directory="profiles/default"
      description="Installs the collective.conference add-on."
      provides="Products.GenericSetup.interfaces.EXTENSION"
      post_handler=".setuphandlers.post_install"
      />
  <genericsetup:registerProfile
      name="uninstall"
      title="collective.conference (uninstall)"
      directory="profiles/uninstall"
      provides="Products.GenericSetup.interfaces.EXTENSION"
      post_handler=".setuphandlers.uninstall"
      />
  <utility
      factory=".setuphandlers.HiddenProfiles"
      name="collective.conference-hiddenprofiles"
      />
</configure>
```

> [!warning] `includeDependencies` a disparu
> Depuis Plone 6.0, `z3c.autoinclude` est remplacé par `plone.autoinclude` et la directive `<includeDependencies package="." />` n'existe plus : incluez explicitement le ZCML des paquets dont vous dépendez dans `dependencies.zcml`.

# 25. L'architecture à composants et le ZCML

Les directives ZCML les plus courantes, avec leurs espaces de noms :

| Directive | Espace de noms | Enregistre |
|---|---|---|
| `<adapter factory for provides name />` | `zope` | un adaptateur ou multi-adaptateur |
| `<utility component\|factory provides name />` | `zope` | un utilitaire (vocabulaire, service) |
| `<subscriber for handler />` | `zope` | un abonné à un événement |
| `<interface interface type name />` | `zope` | une interface nommée (couche de thème) |
| `<class class><require permission interface /></class>` | `zope` | la sécurité d'une classe |
| `<permission id title />` | `zope` | une permission Zope 3 → Zope 2 |
| `<browser:page name for class template permission layer />` | `browser` | une vue |
| `<browser:viewlet name manager class template layer permission />` | `browser` | un viewlet |
| `<browser:viewletManager name provides class />` | `browser` | un gestionnaire de viewlets |
| `<browser:resourceDirectory name directory />` | `browser` | des ressources statiques `++resource++` |
| `<plone:behavior name provides factory marker for />` | `plone` | un comportement |
| `<plone:portlet … />`, `<plone:static-resource … />`, `<plone:service … />`, `<plone:distribution … />` | `plone` | portlet, ressource `++plone++`, service REST, distribution |
| `<genericsetup:registerProfile />`, `<genericsetup:upgradeSteps />` | `genericsetup` | profils et migrations |
| `zcml:condition="installed x"` / `"have feature"` | `zcml` | enregistrement conditionnel |

Les mêmes enregistrements s'écrivent en Python (`provideAdapter`, `provideUtility`) ou avec les décorateurs `@implementer`, `@adapter`, `@provider`. Recherche : `getMultiAdapter((context, request), name="plone_portal_state")`, `getUtility(IRegistry)`, `queryUtility(IVocabularyFactory, name=...)`.

# 26. GenericSetup : profils, étapes d'import, étapes de migration

**GenericSetup** décrit la configuration persistante d'un site en fichiers XML, lus par des **étapes d'import** (`typeinfo`, `workflow`, `plone.app.registry`, `rolemap`, `browserlayer`, `catalog`, `actions`, `viewlets`, `controlpanel`, `portlets`, `propertiestool`, `componentregistry`, `content`…). Un **profil** est un dossier de ces fichiers, enregistré en ZCML et appliqué par `portal_setup`. L'activation d'un module dans le panneau applique son profil `default` ; la désactivation applique `uninstall`.

`profiles/default/metadata.xml` déclare la **version du profil** et ses dépendances :

```xml
<?xml version="1.0" encoding="utf-8"?>
<metadata>
  <version>1000</version>
  <dependencies>
    <dependency>profile-plone.app.dexterity:default</dependency>
  </dependencies>
</metadata>
```

Fichiers fréquents : `types.xml` + `types/*.xml` (FTI), `workflows.xml` + `workflows/<id>/definition.xml`, `registry/*.xml` (plusieurs fichiers depuis 5.1), `rolemap.xml` (permissions ↔ rôles), `browserlayer.xml`, `catalog.xml` (index et colonnes), `actions.xml`, `viewlets.xml`, `controlpanel.xml`, `portlets.xml`, `tinymce.xml`.

> [!warning] Purge
> Beaucoup d'étapes d'import **purgent** ce que le fichier ne déclare pas : `catalog.xml` supprime les colonnes absentes, `<property name="behaviors">` sans `purge="false"` remplace la liste. Lisez le code d'une étape avant de l'appliquer sur des données réelles.

**Étapes de migration** (upgrade steps) : quand le profil passe de 1000 à 1001, on enregistre le code à exécuter sur les sites existants plutôt que de réinstaller :

```xml
<configure xmlns="http://namespaces.zope.org/zope"
           xmlns:genericsetup="http://namespaces.zope.org/genericsetup">
  <genericsetup:upgradeSteps profile="collective.conference:default" source="1000" destination="1001">
    <genericsetup:upgradeStep
        title="Enable new behaviors"
        handler=".v1001.update_types"
        />
    <genericsetup:upgradeStep
        title="Move talks into the schedule folder"
        handler=".v1001.cleanup_site_structure"
        />
  </genericsetup:upgradeSteps>
</configure>
```

```python
from plone import api
from plone.app.upgrade.utils import loadMigrationProfile
import logging

logger = logging.getLogger(__name__)
PROFILE = "profile-collective.conference:default"

def update_types(setup_tool):
    setup_tool.runImportStepFromProfile(PROFILE, "typeinfo")

def cleanup_site_structure(setup_tool):
    loadMigrationProfile(setup_tool, PROFILE)       # recharge tout le profil
    portal = api.portal.get()
    if "schedule" not in portal:
        api.content.create(container=portal, type="Document", id="schedule", title="Programme")
    target = portal["schedule"]
    for brain in api.content.find(portal_type="talk"):
        if target.absolute_url() in brain.getURL():
            continue
        obj = brain.getObject()
        api.content.move(source=obj, target=target, safe_id=True)
        logger.info("%s déplacé", obj.absolute_url())
```

Le panneau Modules complémentaires affiche alors un bouton « Mettre à niveau 1000 → 1001 » ; la ZMI (`portal_setup` ‣ Upgrades) permet de choisir les étapes une à une. Les identifiants d'étapes d'import se lisent dans `portal_setup/manage_importSteps`. Le même mécanisme, dans `plone.app.upgrade`, porte les migrations de Plone lui-même (`@@plone-upgrade`, §58).

# 27. Types de contenu Dexterity et FTI

Un type est défini par une **Factory Type Information** dans `portal_types`. En profil : une ligne dans `types.xml` et un fichier `types/<Nom>.xml` :

```xml
<object name="portal_types">
  <object name="talk" meta_type="Dexterity FTI" />
</object>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<object xmlns:i18n="http://xml.zope.org/namespaces/i18n" meta_type="Dexterity FTI"
        name="talk" i18n:domain="collective.conference">
  <property name="title" i18n:translate="">Conférence</property>
  <property name="description" i18n:translate="">Une intervention du programme</property>
  <property name="icon_expr">string:mic</property>
  <property name="factory">talk</property>
  <property name="add_view_expr">string:${folder_url}/++add++talk</property>
  <property name="link_target"></property>
  <property name="immediate_view">view</property>
  <property name="global_allow">False</property>
  <property name="filter_content_types">True</property>
  <property name="allowed_content_types" />
  <property name="allow_discussion">False</property>
  <property name="default_view">view</property>
  <property name="view_methods">
    <element value="view" />
  </property>
  <property name="default_view_fallback">False</property>
  <property name="add_permission">collective.conference.AddTalk</property>
  <property name="klass">plone.dexterity.content.Item</property>
  <property name="schema">collective.conference.content.talk.ITalk</property>
  <property name="model_source"></property>
  <property name="model_file"></property>
  <property name="behaviors" purge="false">
    <element value="plone.namefromtitle" />
    <element value="plone.dublincore" />
    <element value="plone.relateditems" />
    <element value="plone.versioning" />
    <element value="plone.locking" />
    <element value="plone.textindexer" />
  </property>
  <alias from="(Default)" to="(dynamic view)" />
  <alias from="edit" to="@@edit" />
  <alias from="sharing" to="@@sharing" />
  <alias from="view" to="(selected layout)" />
  <action action_id="view" category="object" condition_expr="" icon_expr="string:toolbar-action/view"
          title="View" url_expr="string:${object_url}" visible="True" i18n:attributes="title">
    <permission value="View" />
  </action>
  <action action_id="edit" category="object"
          condition_expr="not:object/@@plone_lock_info/is_locked_for_current_user|python:True"
          icon_expr="string:toolbar-action/edit" title="Edit" url_expr="string:${object_url}/edit"
          visible="True" i18n:attributes="title">
    <permission value="Modify portal content" />
  </action>
</object>
```

Propriétés globales (Classic UI et Volto) : `title`, `description`, `factory`, `add_permission`, `allow_discussion`, `global_allow`, `filter_content_types` / `allowed_content_types`, `behaviors`, `klass`, et **une** des trois sources de schéma : `schema` (chemin pointé Python, le plus courant), `model_file` (XML supermodel, par exemple `collective.conference.schema:talk.xml`), `model_source` (XML en ligne, utilisé par les types créés dans le navigateur). Propriétés Classic UI : `add_view_expr`, `default_view`, `view_methods`, `immediate_view`, `icon_expr`, `alias`.

Le schéma Python :

```python
from collective.conference import _
from plone.app.textfield import RichText
from plone.autoform import directives
from plone.namedfile.field import NamedBlobImage
from plone.supermodel import model
from zope import schema
from zope.interface import implementer
from plone.dexterity.content import Item


class ITalk(model.Schema):
    """Une conférence."""

    type_of_talk = schema.Choice(
        title=_("Type of talk"),
        vocabulary="collective.conference.TalkTypes",
        required=True,
    )
    details = RichText(
        title=_("Details"),
        description=_("Description of the talk (max. 2000 characters)"),
        max_length=2000,
        required=True,
    )
    audience = schema.Set(
        title=_("Audience"),
        value_type=schema.Choice(vocabulary="collective.conference.Audiences"),
        required=False,
    )
    directives.widget(audience="z3c.form.browser.checkbox.CheckBoxFieldWidget")

    speaker = schema.TextLine(title=_("Speaker"), required=True)
    email = schema.Email(title=_("Email"), required=True)   # plone.schema fournit Email, URI, JSON
    image = NamedBlobImage(title=_("Image"), required=False)

    model.fieldset("logistics", label=_("Logistics"), fields=["room", "duration"])
    room = schema.Choice(title=_("Room"), vocabulary="collective.conference.Rooms", required=False)
    duration = schema.Int(title=_("Duration (minutes)"), min=5, max=180, default=30)

    directives.read_permission(internal_notes="cmf.ReviewPortalContent")
    directives.write_permission(internal_notes="cmf.ReviewPortalContent")
    internal_notes = schema.Text(title=_("Internal notes"), required=False)


@implementer(ITalk)
class Talk(Item):
    """Classe d'instance ; facultative si klass = plone.dexterity.content.Item."""
```

`Item` et `Container` sont les deux classes de base ; `Container` donne un type dossier. Les types créés dans le navigateur (panneau Types de contenu) ont le même FTI avec `model_source` : « Modifier le modèle XML » montre le supermodel que l'on peut copier dans un fichier `model_file` du paquet.

# 28. Schémas, champs et directives de formulaire

**Champs** usuels (`zope.schema`, `plone.schema`, `plone.namedfile.field`, `plone.app.textfield`, `z3c.relationfield.schema`) :

| Champ | Type Python | Note |
|---|---|---|
| `TextLine`, `Text`, `SourceText` | `str` | une ligne / multi-lignes |
| `Bool`, `Int`, `Float`, `Decimal` | | `Bool` doit avoir `required=False` depuis `zope.schema` 6.1 |
| `Date`, `Datetime`, `Timedelta` | `datetime` Python | pas le `DateTime` Zope |
| `Choice` | valeur du terme | `vocabulary="nom"` ou `source=binder` |
| `List`, `Tuple`, `Set`, `Frozenset` | collections | `value_type=Choice(...)` pour un multi-choix |
| `Dict`, `Object` | | `key_type`, `value_type`, `schema` |
| `URI`, `Id`, `DottedName`, `Password` | `str` | |
| `Email`, `JSON` | `plone.schema` | `JSONField(schema=...)` valide par JSON Schema |
| `RichText` | `RichTextValue` | `.raw`, `.output` (HTML filtré, mis en cache), `default_mime_type`, `output_mime_type="text/x-html-safe"` |
| `NamedBlobFile`, `NamedBlobImage` | blobs | fichiers et images |
| `Relation`, `RelationChoice`, `RelationList` | `RelationValue` | §32 |

Paramètres communs : `title`, `description`, `required` (vrai par défaut), `readonly`, `default`, `defaultFactory`, `missing_value`, `min`/`max`, `min_length`/`max_length`, `constraint` (fonction) ; validation personnalisée par `invariant` ou par exception dérivée de `schema.ValidationError`.

> [!warning] Jamais de valeur mutable en `default`
> `default=[]` ou `default=SomeObject()` crée **une seule** instance partagée par tous les objets. Utilisez `defaultFactory`.

**Directives** (`plone.autoform.directives` en Python, attributs `form:` et `security:` en supermodel XML) :

| Directive | Effet |
|---|---|
| `directives.widget("champ", Widget, option=valeur)` | choisir le widget et ses options (`pattern_options` en Classic UI, `frontendOptions` pour Volto) |
| `directives.mode(champ="hidden"\|"display"\|"input")`, `directives.mode(IEditForm, champ="input")` | mode d'affichage, éventuellement par formulaire |
| `directives.omitted("champ")`, `directives.no_omit(IEditForm, "champ")` | retirer un champ des formulaires |
| `directives.order_before(champ="autre")`, `order_after`, `"*"` | ordre |
| `model.fieldset("id", label=..., fields=[...])` | onglet |
| `directives.read_permission(champ="cmf.ManagePortal")`, `write_permission` | protection par permission |
| `model.primary("champ")` | champ principal (fichier téléchargé par `/@@download`) |
| `textindexer.searchable("champ")` | indexation plein texte (§33) |

Introspection : `zope.schema.getFieldsInOrder(ITalk)` ; schéma d'un type à l'exécution : `getUtility(IDexterityFTI, name="talk").lookupSchema()` (sans les champs des comportements).

# 29. Comportements (behaviors)

Un **comportement** attache une fonction réutilisable à un type sans modifier sa classe : des champs supplémentaires, un adaptateur de logique, un marqueur pour des vues ou des abonnés. Comportements livrés les plus utiles : `plone.basic`, `plone.dublincore`, `plone.categorization`, `plone.publication`, `plone.ownership`, `plone.namefromtitle` / `plone.namefromfilename`, `plone.excludefromnavigation`, `plone.relateditems`, `plone.richtext`, `plone.leadimage`, `plone.versioning`, `plone.locking`, `plone.tableofcontents`, `plone.allowdiscussion`, `plone.constraintypes`, `plone.navigationroot`, `plone.nextprevious*`, `plone.textindexer`, `plone.translatable`, `plone.thumb_icon`, `plone.event*`, et `volto.blocks`, `volto.navtitle`, `volto.preview_image` pour Volto.

Activation : panneau Types de contenu ‣ type ‣ Comportements, ou en profil `types/Event.xml` avec `<property name="behaviors" purge="false"><element value="collective.conference.featured" /></property>`.

**Comportement à schéma seul** (champs stockés comme attributs de l'objet) :

```python
from plone.autoform.interfaces import IFormFieldProvider
from plone.supermodel import model
from zope import schema
from zope.interface import provider


@provider(IFormFieldProvider)
class IFeatured(model.Schema):
    model.fieldset("settings", label="Settings", fields=["featured"])
    featured = schema.Bool(title="Featured", default=False, required=False)
```

```xml
<plone:behavior
    name="collective.conference.featured"
    title="Featured"
    description="Mark this item as featured"
    provides=".featured.IFeatured"
    />
```

**Comportement avec adaptateur et marqueur** (logique, valeurs calculées, stockage ailleurs) :

```python
from plone.autoform.interfaces import IFormFieldProvider
from plone.supermodel import model
from zope import schema
from zope.interface import Attribute, Interface, implementer, provider


@provider(IFormFieldProvider)
class IPrice(model.Schema):
    price_net = schema.Float(title="Price (net)", required=True)
    price_vat = Attribute("VAT")
    price_gross = Attribute("Gross price")


class IPriceMarker(Interface):
    """Marqueur posé sur les objets dont le type active le comportement."""


@implementer(IPrice)
class PriceAdapter:
    def __init__(self, context):
        self.context = context

    @property
    def price_net(self):
        return getattr(self.context, "price_net", 0.0)

    @price_net.setter
    def price_net(self, value):
        self.context.price_net = value

    @property
    def price_vat(self):
        return self.price_net * 0.2

    @property
    def price_gross(self):
        return self.price_net + self.price_vat
```

```xml
<plone:behavior
    name="collective.conference.price"
    title="Price"
    provides=".price.IPrice"
    factory=".price.PriceAdapter"
    marker=".price.IPriceMarker"
    for="plone.dexterity.interfaces.IDexterityContent"
    />
```

Utilisation : `IPrice(obj).price_gross`. Avec `factory="plone.behavior.AnnotationStorage"`, les champs sont stockés dans les annotations (§35) sans écrire d'adaptateur. Un comportement avec fabrique n'est **pas** fourni par l'objet lui-même : enregistrez vues et abonnés sur le **marqueur**.

# 30. Vocabulaires

Un vocabulaire est une liste de **termes** `(value, token, title)` : `value` est stocké, `token` (ASCII) circule dans le HTML, `title` est affiché et traduisible.

```python
from zope.interface import provider
from zope.schema.interfaces import IVocabularyFactory
from zope.schema.vocabulary import SimpleTerm, SimpleVocabulary


@provider(IVocabularyFactory)
def talk_types(context):
    items = [("talk", "Talk"), ("lightning", "Lightning talk"), ("keynote", "Keynote")]
    return SimpleVocabulary([SimpleTerm(value=v, token=v, title=t) for v, t in items])
```

```xml
<utility component=".vocabularies.talk_types"
         provides="zope.schema.interfaces.IVocabularyFactory"
         name="collective.conference.TalkTypes" />
```

Référencé par son nom dans un `Choice`. Variantes : `SimpleVocabulary.fromValues([...])` ; **source liée au contexte** (`@provider(IContextSourceBinder)`, passée en `source=`) pour dépendre du contenu courant ; vocabulaire alimenté par le **registre** (`api.portal.get_registry_record`) pour des listes éditables par l'administrateur ; vocabulaire par **requête de catalogue**. Les vocabulaires livrés dans `plone.app.vocabularies` couvrent types, workflows, états, transitions, langues, rôles, groupes, utilisateurs, fuseaux horaires, échelles d'images, permissions, index du catalogue, et `plone.app.vocabularies.Catalog` (arbre de contenu) ou `CatalogSource(**query)` / `StaticCatalogVocabulary(query)` pour les relations. Le service REST `@vocabularies/<nom>` expose les termes à Volto.

> [!tip] Vocabulaires pérennes
> Pour des options modifiables (salles, catégories), la formation Mastering Plone stocke dans le registre un `JSONField` de `{"items": [{"token": ..., "titles": {"fr": ..., "en": ...}}]}` : le `token` stocké dans les contenus ne change jamais, seul le titre évolue.

# 31. Widgets et formulaires `z3c.form`

Plone construit ses formulaires avec `z3c.form`, intégré par `plone.z3cform` et `plone.app.z3cform` (widgets Bootstrap 5, patterns Mockup) et organisé par `plone.autoform` (`AutoExtensibleForm`, fieldsets, directives). Depuis 6.1, tous les widgets Classic UI vivent dans `plone.app.z3cform.widgets`.

Un formulaire autonome :

```python
from plone import schema
from plone.autoform.form import AutoExtensibleForm
from z3c.form import button, form
from zope.interface import Interface


class IContact(Interface):
    name = schema.TextLine(title="Your name")
    email = schema.Email(title="Email")
    message = schema.Text(title="Message")


class ContactForm(AutoExtensibleForm, form.EditForm):
    schema = IContact
    ignoreContext = True
    label = "Contact us"

    @button.buttonAndHandler("Send")
    def handle_send(self, action):
        data, errors = self.extractData()
        if errors:
            self.status = self.formErrorsMessage
            return
        # traitement
        self.status = "Message sent"

    @button.buttonAndHandler("Cancel")
    def handle_cancel(self, action):
        self.request.response.redirect(self.context.absolute_url())
```

```xml
<browser:page name="contact-form" for="*" class=".forms.ContactForm"
              permission="zope2.View" layer="..interfaces.ICollectiveConferenceLayer" />
```

Modes d'un widget : `input`, `display`, `hidden`. Changer le widget : directive `directives.widget`, attribut `widgetFactory` d'un champ, ou dans `updateWidgets()` (ex. `self.widgets["x"].mode = HIDDEN_MODE`, `.required = False`, `.addClass("…")`, `.template = …`). Les widgets Classic UI lisent `pattern_options` (par directive ou par un adaptateur `IValue` nommé `pattern_options`). Les formulaires d'ajout et d'édition Dexterity (`plone.dexterity.browser.add.DefaultAddForm`, `edit.DefaultEditForm`) se surchargent par sous-classe et nouvel enregistrement sur une couche ; `enable_form_tabbing = False` supprime les onglets. Un **formulaire d'affichage** (`plone.autoform.view.WidgetsView`) rend les widgets en mode `display` : `view/w/champ/render` dans le gabarit.

# 32. Relations

Les relations modélisent des liens typés entre contenus, hors hiérarchie et hors texte riche. Pile : `zc.relation` (catalogue de relations), `z3c.relationfield` (`RelationValue`, champs), `plone.app.relationfield` (comportement `plone.relateditems`), `five.intid` (identifiants entiers).

```python
from z3c.relationfield.schema import RelationChoice, RelationList
from plone.app.vocabularies.catalog import CatalogSource
from plone.app.z3cform.widgets.contentbrowser import ContentBrowserFieldWidget

speakers = RelationList(
    title="Speakers",
    value_type=RelationChoice(source=CatalogSource(portal_type="speaker", review_state="published")),
    required=False,
    default=[],
)
directives.widget("speakers", ContentBrowserFieldWidget,
                  pattern_options={"selectableTypes": ["speaker"], "favorites": [{"title": "Speakers", "path": "/Plone/speakers"}]})
```

Depuis 6.1, `ContentBrowserFieldWidget` (pattern `pat-contentbrowser`, colonnes de Miller, clavier, multi-sélection) remplace `RelatedItemsFieldWidget`, encore accepté mais obsolète. Autres widgets : `SelectFieldWidget`, `AjaxSelectFieldWidget` ou cases à cocher avec `StaticCatalogVocabulary`. API :

```python
from plone import api
api.relation.create(source=talk, target=speaker, relationship="speakers")
api.relation.get(source=talk)            # ou target=..., relationship=...
api.relation.delete(source=talk, relationship="speakers")
rel.to_object, rel.to_path, rel.from_attribute, rel.isBroken()
```

Le panneau `/@@inspect-relations` liste et répare ; l'intégrité des liens (`plone.app.linkintegrity`) et les copies de travail reposent sur le même mécanisme. Alternative légère : stocker des UID dans un `Choice`/`List` avec le même widget.

# 33. Catalogue et indexation

`portal_catalog` tient des **index** (`FieldIndex`, `KeywordIndex`, `DateIndex`, `DateRangeIndex`, `BooleanIndex`, `ZCTextIndex` pour `SearchableText`, `PathIndex`, `UUIDIndex`, `DateRecurringIndex`) et des **colonnes de métadonnées** copiées dans les « brains ». Index courants : `portal_type`, `path`, `review_state`, `Subject`, `effective`, `created`, `modified`, `SearchableText`, `UID`, `getObjPositionInParent`.

Déclarer index et colonnes dans `profiles/default/catalog.xml` :

```xml
<object name="portal_catalog">
  <index name="room" meta_type="FieldIndex"><indexed_attr value="room" /></index>
  <column value="room" />
  <column value="duration" />
</object>
```

Un **indexeur** calcule une valeur pour un type ou un marqueur :

```python
from plone.indexer import indexer
from .content.talk import ITalk

@indexer(ITalk)
def talk_room(obj):
    return obj.room or None
```

```xml
<adapter factory=".indexers.talk_room" name="room" />
```

Plein texte : activer le comportement `plone.textindexer` sur le type et marquer les champs avec `textindexer.searchable("details")` (ou `indexer:searchable="true"` en XML) ; un adaptateur `IDynamicTextIndexExtender` nommé ajoute du texte hors champs ; un `IDexterityTextIndexFieldConverter` convertit un champ exotique.

Interrogation :

```python
from plone import api
brains = api.content.find(portal_type="talk", review_state="published",
                          path={"query": "/Plone/schedule", "depth": 1},
                          sort_on="effective", sort_order="reverse")
for b in brains:
    b.Title, b.getURL(), b.getPath(), b.room, b.getObject()   # getObject réveille l'objet
```

`portal_catalog.searchResults(...)` applique les droits de l'utilisateur ; `unrestrictedSearchResults(...)` les ignore (scripts). Depuis 5.1, les opérations d'indexation sont **mises en file** et traitées à la fin de la transaction : dans un script, appeler `Products.CMFCore.indexing.processQueue()` avant de mesurer. Réindexer un objet : `obj.reindexObject()` ou `obj.reindexObject(idxs=["room"])` ; tout le catalogue : `portal_catalog.clearFindAndRebuild()` (ZMI ‣ Advanced).

# 34. Événements et abonnés

Le système d'événements est simple : pas d'ordre d'appel garanti, pas d'annulation, pas de valeur de retour, une exception interrompt la requête.

```xml
<subscriber for=".content.talk.ITalk zope.lifecycleevent.IObjectAddedEvent"
            handler=".subscribers.notify_organizers" />
<subscriber for="Products.PlonePAS.events.UserLoggedOutEvent"
            handler=".subscribers.clear_cookies" />
```

```python
def notify_organizers(obj, event):
    ...
def clear_cookies(event):
    request = event.object.REQUEST
```

Événements usuels : `IObjectCreatedEvent`, `IObjectAddedEvent`, `IObjectModifiedEvent`, `IObjectMovedEvent`, `IObjectRemovedEvent`, `IObjectCopiedEvent` (`zope.lifecycleevent`) ; `IBeforeTransitionEvent`, `IAfterTransitionEvent` (`Products.DCWorkflow`), `IActionSucceededEvent` (`Products.CMFCore`) ; `IProcessStarting`, `IDatabaseOpened` (`zope.processlifetime`) ; `IUserLoggedInEvent`, `IPrincipalCreatedEvent` (PAS). Émettre : `zope.event.notify(MyEvent(obj))`. Les **règles de contenu** (§22) sont une interface graphique sur ce mécanisme.

# 35. Annotations

`zope.annotation` attache des données arbitraires à un objet persistant sans toucher à sa classe, sous une clé propre au paquet :

```python
from persistent.mapping import PersistentMapping
from zope.annotation.interfaces import IAnnotations

KEY = "collective.conference.votes"

def votes_for(obj):
    ann = IAnnotations(obj)
    if KEY not in ann:
        ann[KEY] = PersistentMapping({"up": 0, "down": 0})
    return ann[KEY]
```

Usages : stockage des comportements `AnnotationStorage`, portlets assignés, caches par requête (`IAnnotations(request)`), réglages d'extension sur un contenu. Préférez `PersistentMapping`, `PersistentList` et `BTrees` à des classes persistantes maison : un objet dont la classe a disparu rend la base inexportable. Nettoyez vos annotations à la désinstallation.

# 36. Registre de configuration et panneaux de configuration

Un panneau de configuration moderne = un **schéma** d'enregistrements + un **formulaire** `RegistryEditForm` + une **configlet**.

```python
# controlpanel/settings.py
from plone.app.registry.browser.controlpanel import ControlPanelFormWrapper, RegistryEditForm
from zope import schema
from zope.interface import Interface


class IConferenceSettings(Interface):
    talk_submission_open = schema.Bool(title="Allow talk submission", default=False, required=False)
    rooms = schema.List(title="Rooms", value_type=schema.TextLine(), default=["Auditorium"], required=False)


class ConferenceSettingsForm(RegistryEditForm):
    schema = IConferenceSettings
    schema_prefix = "collective.conference"
    label = "Conference settings"


class ConferenceControlPanel(ControlPanelFormWrapper):
    form = ConferenceSettingsForm
```

```xml
<!-- browser/configure.zcml -->
<browser:page name="conference-controlpanel"
              for="plone.base.interfaces.IPloneSiteRoot"
              class=".controlpanel.settings.ConferenceControlPanel"
              permission="cmf.ManagePortal"
              layer="..interfaces.ICollectiveConferenceLayer" />
```

```xml
<!-- profiles/default/registry/settings.xml : crée les enregistrements -->
<registry xmlns:i18n="http://xml.zope.org/namespaces/i18n" i18n:domain="collective.conference">
  <records interface="collective.conference.controlpanel.settings.IConferenceSettings"
           prefix="collective.conference" />
</registry>
```

```xml
<!-- profiles/default/controlpanel.xml : l'entrée dans Configuration du site -->
<object name="portal_control-panel" xmlns:i18n="http://xml.zope.org/namespaces/i18n" i18n:domain="collective.conference">
  <configlet title="Conference" action_id="collective.conference.settings" appId="collective.conference"
             category="Products" condition_expr="" icon_expr="string:mic"
             url_expr="string:${portal_url}/@@conference-controlpanel" visible="True" i18n:attributes="title">
    <permission>Manage portal</permission>
  </configlet>
</object>
```

Lecture : `api.portal.get_registry_record("collective.conference.rooms")` ou, pour tout le schéma, `getUtility(IRegistry).forInterface(IConferenceSettings, prefix="collective.conference")`. Les enregistrements peuvent être conditionnels (`condition="installed my.package"`, `"have plone-62"`). Depuis 6.2, `plone.registry` met en cache les lectures par requête. Les schémas des panneaux du cœur sont dans `plone.base.interfaces.controlpanel` (`ISiteSchema`, `ISearchSchema`, `ITinyMCESchema`, `IImagingSchema`, `INavigationSchema`…).

# 37. Sécurité : permissions, rôles, workflows DCWorkflow

**Déclarer une permission** et la **confier à des rôles** :

```xml
<!-- permissions.zcml -->
<permission id="collective.conference.AddTalk" title="collective.conference: Add Talk" />
```

```xml
<!-- profiles/default/rolemap.xml -->
<rolemap>
  <permissions>
    <permission name="collective.conference: Add Talk" acquire="True">
      <role name="Manager" /><role name="Site Administrator" /><role name="Contributor" />
    </permission>
  </permissions>
</rolemap>
```

Un **workflow** se déclare dans `profiles/default/workflows.xml` (définition et liaison aux types) et se décrit dans `workflows/talk_workflow/definition.xml` :

```xml
<object name="portal_workflow">
  <object name="talk_workflow" meta_type="Workflow" />
  <bindings>
    <type type_id="talk"><bound-workflow workflow_id="talk_workflow" /></type>
  </bindings>
</object>
```

```xml
<?xml version="1.0"?>
<dc-workflow workflow_id="talk_workflow" title="Talk workflow" state_variable="review_state"
             initial_state="submitted" manager_bypass="False">
  <permission>View</permission>
  <permission>Modify portal content</permission>
  <permission>Access contents information</permission>

  <state state_id="submitted" title="Submitted">
    <exit-transition transition_id="accept" />
    <exit-transition transition_id="reject" />
    <permission-map name="View" acquired="False">
      <permission-role>Manager</permission-role><permission-role>Reviewer</permission-role>
      <permission-role>Owner</permission-role>
    </permission-map>
    <permission-map name="Modify portal content" acquired="False">
      <permission-role>Manager</permission-role><permission-role>Owner</permission-role>
    </permission-map>
    <permission-map name="Access contents information" acquired="False">
      <permission-role>Manager</permission-role><permission-role>Reviewer</permission-role>
      <permission-role>Owner</permission-role>
    </permission-map>
  </state>
  <state state_id="accepted" title="Accepted">
    <exit-transition transition_id="reject" />
    <permission-map name="View" acquired="False">
      <permission-role>Anonymous</permission-role>
    </permission-map>
    <permission-map name="Modify portal content" acquired="False">
      <permission-role>Manager</permission-role><permission-role>Reviewer</permission-role>
    </permission-map>
    <permission-map name="Access contents information" acquired="False">
      <permission-role>Anonymous</permission-role>
    </permission-map>
  </state>
  <state state_id="rejected" title="Rejected">
    <exit-transition transition_id="accept" />
    <permission-map name="View" acquired="False">
      <permission-role>Manager</permission-role><permission-role>Reviewer</permission-role>
      <permission-role>Owner</permission-role>
    </permission-map>
    <permission-map name="Modify portal content" acquired="False">
      <permission-role>Manager</permission-role>
    </permission-map>
    <permission-map name="Access contents information" acquired="False">
      <permission-role>Manager</permission-role><permission-role>Reviewer</permission-role>
      <permission-role>Owner</permission-role>
    </permission-map>
  </state>

  <transition transition_id="accept" title="Accept" new_state="accepted" trigger="USER"
              before_script="" after_script="">
    <action url="%(content_url)s/content_status_modify?workflow_action=accept" category="workflow" icon="">Accept</action>
    <guard><guard-permission>Review portal content</guard-permission></guard>
  </transition>
  <transition transition_id="reject" title="Reject" new_state="rejected" trigger="USER"
              before_script="" after_script="">
    <action url="%(content_url)s/content_status_modify?workflow_action=reject" category="workflow" icon="">Reject</action>
    <guard><guard-permission>Review portal content</guard-permission></guard>
  </transition>

  <variable variable_id="action" for_catalog="False" for_status="True" update_always="True">
    <description>Previous transition</description>
    <default><expression>transition/getId|nothing</expression></default>
    <guard />
  </variable>
  <variable variable_id="actor" for_catalog="False" for_status="True" update_always="True">
    <default><expression>user/getId</expression></default><guard />
  </variable>
  <variable variable_id="comments" for_catalog="False" for_status="True" update_always="True">
    <default><expression>python:state_change.kwargs.get('comment', '')</expression></default><guard />
  </variable>
  <variable variable_id="review_history" for_catalog="False" for_status="False" update_always="False">
    <default><expression>state_change/getHistory</expression></default>
    <guard><guard-permission>Request review</guard-permission><guard-permission>Review portal content</guard-permission></guard>
  </variable>
  <variable variable_id="time" for_catalog="False" for_status="True" update_always="True">
    <default><expression>state_change/getDateTime</expression></default><guard />
  </variable>
</dc-workflow>
```

Les gardes acceptent des permissions, des rôles, des groupes ou une expression TALES. Après import, **mettre à jour les correspondances de rôles** (`portal_workflow` ‣ Update security settings, ou `api.portal.get_tool("portal_workflow").updateRoleMappings()` dans une étape de migration). En code : `api.content.get_state(obj)`, `api.content.transition(obj, transition="accept")`, `api.user.has_permission("Modify portal content", obj=obj)`. La ZMI permet de prototyper un workflow (copier `simple_publication_workflow`), puis de l'exporter en XML par `portal_setup`.

**Zope met en œuvre la sécurité à l'accès d'attribut** : dans un gabarit ou un script restreint, un attribut non protégé par `<require>` ou par `ClassSecurityInfo` est inaccessible ; dans une vue (code de confiance), tout est accessible, la vue elle-même étant protégée par la permission de son `browser:page`. `verbose_security: true` dans `instance.yaml` explique les refus dans le journal.

# 38. `plone.api`

`plone.api` couvre « 20 % des tâches que 80 % des développeurs font tout le temps », avec une API stable : les ruptures sont réservées aux versions majeures.

```python
from plone import api

# contenu
portal = api.portal.get()
folder = api.content.create(container=portal, type="Folder", id="schedule", title="Programme")
talk = api.content.create(container=folder, type="talk", title="Plone 6.2", speaker="Alice")
api.content.transition(obj=talk, transition="accept")
api.content.move(source=talk, target=portal["archive"], safe_id=True)
api.content.rename(obj=talk, new_id="plone-62")
api.content.copy(source=talk, target=folder)
api.content.delete(obj=talk)
obj = api.content.get(path="/schedule/plone-62")  # ou UID="…"
api.content.get_uuid(obj)
api.content.find(portal_type="talk", review_state="accepted")
view = api.content.get_view(name="view", context=obj, request=request)
api.content.get_state(obj)
api.content.disable_roles_acquisition(obj=folder)

# portail
api.portal.get_navigation_root(obj); api.portal.get_tool("portal_catalog")
api.portal.get_registry_record("plone.site_title"); api.portal.set_registry_record("plone.site_title", "…")
api.portal.show_message("Enregistré", request=request, type="info")
api.portal.send_email(recipient="a@example.org", subject="…", body="…")
api.portal.translate("Message", lang="fr"); api.portal.get_current_language()
api.portal.get_vocabulary("plone.app.vocabularies.PortalTypes")

# utilisateurs et groupes
user = api.user.create(email="alice@example.org", username="alice", password="…", roles=("Member",))
api.user.get(username="alice"); api.user.get_current(); api.user.is_anonymous()
api.user.grant_roles(username="alice", roles=["Editor"], obj=folder)   # rôle local
api.user.get_roles(username="alice", obj=folder); api.user.has_permission("View", obj=folder)
group = api.group.create(groupname="organizers", roles=["Reviewer"])
api.group.add_user(groupname="organizers", username="alice")

# environnement
with api.env.adopt_roles(["Manager"]):
    ...                               # élévation temporaire de rôles
with api.env.adopt_user(username="alice"):
    ...
api.env.debug_mode(); api.env.plone_version(); api.env.read_only_mode()

# relations et modules
api.relation.create(source=talk, target=speaker, relationship="speakers")
api.addon.install("collective.easyform"); api.addon.get_addons(); api.addon.get_version("plone.api")
```

Les exceptions sont dans `plone.api.exceptions` (`InvalidParameterError`, `MissingParameterError`, `CannotGetPortalError`…). Le cœur de Plone limite son usage de `plone.api` ; dans un module complémentaire, c'est l'API de référence.

# 39. Courriel, journalisation, scripts de maintenance

- **Courriel** : le panneau Courriel configure l'hôte SMTP, le port, les identifiants, l'expéditeur (`plone.smtp_host`, `plone.email_from_address`…). `api.portal.send_email` passe par `MailHost`, qui envoie **à la validation de la transaction** (un rollback annule l'envoi). En développement, `Products.PrintingMailHost` imprime les courriels sur la console.
- **Journalisation** : `logging.getLogger("collective.conference")` ; la configuration (niveau, fichiers, rotation) est dans `zope.conf`, générée par `cookiecutter-zope-instance` ou définie dans `instance.yaml`. `Products.SiteErrorLog` (`error_log` dans la ZMI) garde les dernières traces d'erreur consultables dans le navigateur.
- **Scripts** (`zconsole run instance/etc/zope.conf script.py`) : poser une requête factice et un gestionnaire de sécurité (§12), utiliser `unrestrictedSearchResults` et `processQueue()` (§33), valider explicitement par `transaction.commit()`, et toujours vérifier sur quelle base on travaille (`app._p_jar.db().storage.getName()`) avant d'écrire. Pour des traitements longs, valider par lots (`transaction.savepoint()` ou commits intermédiaires) pour limiter la mémoire et les conflits.

# 40. Tests : `plone.app.testing` et `pytest-plone`

`plone.app.testing` fournit des **couches** (layers) qui montent une fois un Zope et un site Plone en mémoire, puis isolent chaque test par transaction :

```python
# testing.py
from plone.app.testing import PLONE_FIXTURE, FunctionalTesting, IntegrationTesting, PloneSandboxLayer
from plone.testing.zope import WSGI_SERVER_FIXTURE
import collective.conference


class Layer(PloneSandboxLayer):
    defaultBases = (PLONE_FIXTURE,)

    def setUpZope(self, app, configurationContext):
        self.loadZCML(package=collective.conference)

    def setUpPloneSite(self, portal):
        self.applyProfile(portal, "collective.conference:default")


FIXTURE = Layer()
INTEGRATION_TESTING = IntegrationTesting(bases=(FIXTURE,), name="CollectiveConference:Integration")
FUNCTIONAL_TESTING = FunctionalTesting(bases=(FIXTURE, WSGI_SERVER_FIXTURE), name="CollectiveConference:Functional")
```

Avec `unittest` et `zope.testrunner` : `self.layer["portal"]`, `self.layer["request"]`, `setRoles(portal, TEST_USER_ID, ["Manager"])`, `login(portal, TEST_USER_NAME)`. Avec **pytest-plone** (1.1, préféré par Cookieplone) :

```python
# tests/conftest.py
from collective.conference.testing import FUNCTIONAL_TESTING, INTEGRATION_TESTING
from pytest_plone import fixtures_factory

pytest_plugins = ["pytest_plone"]
globals().update(fixtures_factory(((FUNCTIONAL_TESTING, "functional"), (INTEGRATION_TESTING, "integration"))))
```

```python
# tests/test_content.py
import pytest
from plone import api


@pytest.fixture
def talk(portal):
    with api.env.adopt_roles(["Manager"]):
        return api.content.create(container=portal, type="talk", title="A talk", speaker="Bob", email="b@x.org")


class TestTalk:
    def test_fti(self, get_fti):
        assert get_fti("talk").global_allow is False

    def test_create(self, talk):
        assert talk.portal_type == "talk"
        assert api.content.get_state(talk) == "submitted"

    def test_view(self, talk, http_request):
        view = api.content.get_view("view", talk, http_request)
        assert "A talk" in view()
```

Fixtures fournies : `app`, `portal`, `http_request`, `integration`/`functional` (par fonction, classe ou session), `portal_class` avec `@pytest.mark.portal(content=[...], roles=[...])`, `get_fti`, `browser` pour les tests fonctionnels (`zope.testbrowser`), `generate_mo`. Les tests d'acceptation de bout en bout utilisent Robot Framework (`plone.app.robotframework`) ou Playwright côté Volto. Les tests REST s'écrivent avec `requests` contre la couche fonctionnelle (`self.layer["portal"].absolute_url()`).

# 41. Internationalisation

```python
# collective/conference/__init__.py
from zope.i18nmessageid import MessageFactory
_ = MessageFactory("collective.conference")
```

Les chaînes passées à `_()` dans les schémas, vues et vocabulaires, et les attributs `i18n:translate`/`i18n:attributes` des gabarits (domaine `collective.conference`), sont extraites par `i18ndude` dans `locales/collective.conference.pot` puis traduites dans `locales/fr/LC_MESSAGES/collective.conference.po` ; `make i18n` (Cookieplone) enchaîne extraction, fusion et compilation (`zope_i18n_compile_mo_files=true` compile au démarrage). Le ZCML `<i18n:registerTranslations directory="locales" />` enregistre les catalogues. Les titres de FTI et de configlets se traduisent avec `i18n:domain` et `i18n:translate` dans le XML. Pour les traductions de **contenu** (et non d'interface), c'est `plone.app.multilingual`. Voir [[Internationalisation]].

# Partie V — Classic UI

# 42. Vues

Une **vue** est un multi-adaptateur nommé de `(context, request)`, enregistré sur une **couche**, protégé par une **permission**, qui renvoie une réponse (HTML, JSON, fichier). En Plone on sous-classe `Products.Five.browser.BrowserView` (avec acquisition, expression `provider:` et aides Plone) ; `zope.publisher.browser.BrowserView` suffit pour des fragments.

```python
from Products.Five.browser import BrowserView
from Products.Five.browser.pagetemplatefile import ViewPageTemplateFile
from plone import api


class TalkListView(BrowserView):
    index = ViewPageTemplateFile("templates/talk_list.pt")   # ou template= dans le ZCML

    def talks(self):
        return [
            {"title": b.Title, "url": b.getURL(), "room": b.room}
            for b in api.content.find(context=self.context, portal_type="talk", sort_on="sortable_title")
        ]

    def __call__(self):
        self.request.response.setHeader("X-Example", "1")
        return self.index()
```

```xml
<browser:page
    name="talk-list"
    for="plone.app.contenttypes.interfaces.IFolder"
    class=".views.TalkListView"
    permission="zope2.View"
    layer="..interfaces.ICollectiveConferenceLayer"
    />
```

Attributs ZCML : `for` (interface du contexte, `*` pour tous), `name` (accès par `/@@talk-list`), `permission` (`zope2.View`, `zope2.Public`, `cmf.ModifyPortalContent`, `cmf.ManagePortal`, `cmf.AddPortalContent`…), `class`, `template`, `layer`, `allowed_attributes`. Ajouter la vue au menu Affichage d'un type : `view_methods` du FTI. Pour afficher le menu contextuel sur une vue non par défaut, la marquer `plone.app.layout.globals.interfaces.IViewView`.

> [!warning] Rien dans `__init__`
> La chaîne d'acquisition et l'utilisateur ne sont pas disponibles dans le constructeur, et la vue peut être instanciée hors rendu (menu Affichage). Mettez la logique dans `__call__()` ou des méthodes appelées depuis le gabarit.

Le gabarit d'une page complète utilise le `main_template` et remplit un **slot** :

```html
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:tal="http://xml.zope.org/namespaces/tal"
      xmlns:metal="http://xml.zope.org/namespaces/metal" xmlns:i18n="http://xml.zope.org/namespaces/i18n"
      i18n:domain="collective.conference" metal:use-macro="context/main_template/macros/master">
<body>
  <metal:block fill-slot="top_slot"
      tal:define="dummy python:request.set('disable_plone.rightcolumn', 1)" />
  <metal:block fill-slot="content-core">
    <ul class="list-group">
      <li class="list-group-item" tal:repeat="talk view/talks">
        <a href="${talk/url}">${talk/title}</a> <span class="badge bg-secondary">${talk/room}</span>
      </li>
    </ul>
  </metal:block>
</body>
</html>
```

Slots disponibles : `top_slot`, `head_slot`, `style_slot`, `javascript_head_slot` (en-tête) ; `global_statusmessage` ; `content`, `main`, `body`, `content-title`, `content-description`, `content-core` ; `column_one_slot`/`portlets_one_slot`, `column_two_slot`/`portlets_two_slot`.

**Surcharger un gabarit existant** sans toucher au code : `z3c.jbot`. Dans `browser/configure.zcml` :

```xml
<include package="z3c.jbot" file="meta.zcml" />
<browser:jbot directory="overrides" layer="..interfaces.ICollectiveConferenceLayer" />
```

puis déposer dans `browser/overrides/` une copie nommée d'après le chemin pointé du fichier d'origine, par exemple `plone.app.layout.viewlets.logo.pt` pour `plone/app/layout/viewlets/logo.pt`. Un redémarrage est nécessaire à l'ajout d'un fichier ; ensuite les modifications sont relues à chaque requête. En 6.2, des gabarits déplacés vers `plone.app.layout` (verrouillage, confirmation `plone.protect`) gardent une correspondance d'ancien nom, avec avertissement d'obsolescence.

**Surcharger une classe de vue** : la ré-enregistrer sous le même nom sur une couche plus spécifique, en sous-classant l'originale (exemple : `plone.app.users.browser.register.RegistrationForm`). Obtenir une vue en code : `api.content.get_view(name, context, request)`, `getMultiAdapter((context, request), name=...)`, ou traversal `context.unrestrictedTraverse("@@name")`.

# 43. Gabarits et variables globales

Les gabarits sont des **Zope Page Templates** rendus par Chameleon : TAL, TALES, METAL et i18n, détaillés dans [[Tal et Metal]]. Rappels propres à Plone :

- l'expression par défaut est **path**, pas Python : `${context/title}`, `${python:len(items)}` ; `|` fournit un repli mais masque aussi les fautes de frappe ; `structure` désactive l'échappement ;
- ordre d'évaluation : `define`, `condition`, `repeat`, `content`/`replace`, `attributes`, `omit-tag` ; `tal:switch`/`tal:case` et `<?python ?>` sont des extensions Chameleon (fichiers seulement) ;
- variables : `context`, `view`, `request`, `template`, `options`, `nothing`, `default`, `repeat`, `icons` (résolveur d'icônes) ;
- vues d'aide à définir en tête de gabarit :

```html
<tal:defs define="portal_state context/@@plone_portal_state;
                  context_state context/@@plone_context_state;
                  plone_view context/@@plone;
                  portal_url portal_state/portal_url;
                  navroot_url portal_state/navigation_root_url;
                  user portal_state/member;
                  is_anon portal_state/anonymous;
                  is_editable context_state/is_editable;
                  workflow_state context_state/workflow_state;
                  lang portal_state/language;" />
```

`@@plone_portal_state` : `portal`, `portal_url`, `navigation_root`, `language`, `member`, `anonymous`, `friendly_types`. `@@plone_context_state` : `current_page_url`, `canonical_object`, `view_url`, `is_default_page`, `is_folderish`, `is_editable`, `is_locked`, `workflow_state`, `actions(category)`, `folder`, `parent`. `@@plone` : `toLocalizedTime`, `toLocalizedSize`, `normalizeString`, `cropText`, `getCurrentFolderUrl`, `navigationRootPath`, `showToolbar`. `@@authenticator` fournit le jeton CSRF (§51). Les gabarits stockés dans la ZMI tournent sous RestrictedPython : réservez-les au prototypage.

# 44. Couches (layers)

Une **couche** est une interface marqueur posée sur la requête ; les enregistrements qui la mentionnent ne sont actifs que si elle est présente. Deux sortes :

- **couche de module** : activée quand le module est installé dans le site, via `profiles/default/browserlayer.xml` ; c'est ce qui permet à plusieurs sites d'un même Zope de ne pas partager les mêmes surcharges ;
- **couche de thème** : dérivée de `plone.theme.interfaces.IDefaultPloneLayer`, liée à un thème dans `portal_skins`.

```python
# interfaces.py
from plone.app.z3cform.interfaces import IPloneFormLayer
from zope.publisher.interfaces.browser import IDefaultBrowserLayer


class ICollectiveConferenceLayer(IDefaultBrowserLayer):
    """Couche active quand collective.conference est installé."""
```

```xml
<!-- profiles/default/browserlayer.xml -->
<layers>
  <layer name="collective.conference" interface="collective.conference.interfaces.ICollectiveConferenceLayer" />
</layers>
```

Pour surcharger une vue déjà enregistrée sur une couche spécialisée (par exemple `IPloneFormLayer` pour les macros de formulaire), votre couche doit **hériter** de celle-ci. `overrides.zcml` à la racine du paquet surcharge sans condition. Diagnostic : `ICollectiveConferenceLayer.providedBy(request)`, `plone.browserlayer.utils.registered_layers()`, `request.__provides__.__iro__`. En test : `alsoProvides(request, ICollectiveConferenceLayer)`.

# 45. Viewlets

Un **viewlet** est un fragment de page rendu par un **gestionnaire de viewlets**, ordonnable et masquable dans le navigateur (`/@@manage-viewlets`). Gestionnaires courants : `plone.htmlhead`, `plone.htmlhead.links`, `plone.portaltop`, `plone.portalheader` (logo, recherche, actions du site), `plone.mainnavigation`, `plone.globalstatusmessage`, `plone.abovecontent` (fil d'Ariane, verrou), `plone.abovecontenttitle` (image à la une, réseaux sociaux), `plone.belowcontenttitle` (signature), `plone.abovecontentbody` (table des matières), `plone.belowcontentbody` (contributeurs, droits, mots-clés, éléments liés), `plone.belowcontent` (actions du document, précédent/suivant, commentaires), `plone.portalfooter`, `plone.toolbar`.

```python
from plone.app.layout.viewlets import ViewletBase
from plone import api


class SponsorsViewlet(ViewletBase):
    def update(self):
        super().update()
        self.sponsors = api.content.find(portal_type="sponsor", review_state="published")

    def available(self):
        return bool(self.sponsors)
```

```html
<div class="sponsors" tal:condition="view/available">
  <a tal:repeat="s view/sponsors" href="${s/getURL}" class="me-2">${s/Title}</a>
</div>
```

```xml
<browser:viewlet
    name="collective.conference.sponsors"
    manager="plone.app.layout.viewlets.interfaces.IPortalFooter"
    class=".viewlets.SponsorsViewlet"
    template="templates/sponsors.pt"
    layer="..interfaces.ICollectiveConferenceLayer"
    permission="zope2.View"
    />
```

Restreindre à un type (`for=`) ou à une vue (`view=`), ou décider dans `render()`. Ordre et masquage persistants dans `profiles/default/viewlets.xml` :

```xml
<object>
  <order manager="plone.portalfooter" skinname="*">
    <viewlet name="collective.conference.sponsors" insert-before="*" />
  </order>
  <hidden manager="plone.belowcontentbody" skinname="*">
    <viewlet name="plone.rights" />
  </hidden>
</object>
```

On ne déplace pas un viewlet d'un gestionnaire à l'autre dans le navigateur : on le masque et on le ré-enregistre ailleurs (une surcharge ZCML avec `layer` suffit). Un gestionnaire personnalisé : `browser:viewletManager` avec `class="plone.app.viewletmanager.manager.OrderedViewletManager"`, rendu par `<div tal:replace="structure provider:mon.manager" />`. Depuis 6.0, les noms ne contiennent plus celui de leur gestionnaire (`plone.documentbyline`, `plone.keywords`, `plone.relateditems`, `plone.rights`).

# 46. Portlets

Un **portlet** est un bloc assignable aux colonnes gauche, droite, au pied de page ou à l'en-tête, avec héritage dans l'arbre et blocage possible, assigné par contexte, par type, par groupe ou par utilisateur (`/@@manage-portlets`). Portlets livrés : navigation, actualités, événements, connexion, calendrier, recherche, collection, texte statique, flux RSS, révision.

Un portlet = schéma + `Assignment` (données persistantes) + `Renderer` + formulaires d'ajout et d'édition :

```python
from plone.app.portlets.portlets import base
from plone.portlets.interfaces import IPortletDataProvider
from Products.Five.browser.pagetemplatefile import ViewPageTemplateFile
from zope import schema
from zope.interface import implementer


class INextTalkPortlet(IPortletDataProvider):
    count = schema.Int(title="Number of talks", default=3, min=1)


@implementer(INextTalkPortlet)
class Assignment(base.Assignment):
    def __init__(self, count=3):
        self.count = count

    @property
    def title(self):
        return f"Next talks ({self.count})"


class Renderer(base.Renderer):
    template = ViewPageTemplateFile("nexttalk.pt")

    @property
    def available(self):
        return bool(self.talks())

    def talks(self):
        from plone import api
        return api.content.find(portal_type="talk", review_state="accepted",
                                sort_on="start", sort_limit=self.data.count)[: self.data.count]

    def render(self):
        return self.template()


class AddForm(base.AddForm):
    schema = INextTalkPortlet
    label = "Add next talks portlet"

    def create(self, data):
        return Assignment(**data)


class EditForm(base.EditForm):
    schema = INextTalkPortlet
    label = "Edit next talks portlet"
```

```xml
<plone:portlet
    name="collective.conference.NextTalks"
    interface=".portlets.INextTalkPortlet"
    assignment=".portlets.Assignment"
    renderer=".portlets.Renderer"
    addview=".portlets.AddForm"
    editview=".portlets.EditForm"
    />
```

Le gabarit utilise les classes Bootstrap `card portlet` / `card-header` / `card-body`. `profiles/default/portlets.xml` déclare le type de portlet et des assignations initiales. Un portlet se recharge par AJAX via `@@render-portlet?portlethash=…`.

# 47. Ressources statiques, Mockup et Module Federation

Fichiers statiques : `<browser:resourceDirectory name="collective.conference" directory="static" />` les sert sous `++resource++collective.conference/…` ; `<plone:static-resource name="collective.conference" directory="static" />` sous `++plone++collective.conference/…` (surchargeable dans le navigateur). CSS et JavaScript passent par le **registre de ressources** (`plone.base.interfaces.resources.IBundleRegistry`), qui gère dépendances, ordre et cache :

```xml
<!-- profiles/default/registry/bundles.xml -->
<registry>
  <records interface="plone.base.interfaces.resources.IBundleRegistry" prefix="plone.bundles/conference">
    <value key="enabled">True</value>
    <value key="jscompilation">++plone++collective.conference/conference.min.js</value>
    <value key="csscompilation">++plone++collective.conference/conference.min.css</value>
    <value key="load_async">False</value>
    <value key="load_defer">True</value>
    <value key="depends">plone</value>
  </records>
</registry>
```

`depends` accepte un bundle, une liste, `all` (6.0.14) ou `*` (de retour en 6.2) pour charger en dernier ; le CSS personnalisé du panneau Thème se charge toujours après. Depuis 6.0, on n'enregistre **que des bundles compilés**, plus de ressources individuelles ni de RequireJS : le JavaScript de Classic UI est **Mockup** (sur Patternslib 9), en modules ES6 construits par webpack. Un **pattern** s'active par une classe CSS (`pat-contentbrowser`, `pat-structure`, `pat-tinymce`, `pat-modal`, `pat-datepicker`, `pat-select2`, `pat-navigationmarker`…) avec options en `data-pat-*`. Créer le sien : `plonecli add mockup_pattern`, `pnpm install`, `pnpm build` → bundle « remote » enregistré dans `registry/bundles.xml`. La **Module Federation** permet à ce remote de réutiliser jQuery, Bootstrap et les dépendances déjà chargées par le bundle hôte Mockup ; `jQuery` et `bootstrap` restent aussi disponibles en global pour les anciens scripts. En 6.2, Mockup 5.6 apporte TinyMCE 8, `pat-contentbrowser` réécrit en Svelte 5, `pat-structure` aligné sur DataTables, et moins de chemins dépendant de jQuery.

# 48. Thème : réglages TTW, add-on Barceloneta, Diazo, modes de couleur

Quatre niveaux, du plus léger au plus lourd :

1. **Dans le navigateur** : logo et favicon (panneau Site), CSS personnalisé (Thème ‣ Avancé ‣ Custom Styles, servi en dernier). L'éditeur de thème TTW de Plone 5 a disparu : le panneau Thèmes ne fait plus que téléverser et télécharger un zip.
2. **Add-on Barceloneta** : `plonecli create addon plonetheme.monsite` puis `plonecli add theme_barceloneta`. Le dossier `theme/` contient `index.html`, `manifest.cfg`, `rules.xml`, `package.json`, `scss/` (`_variables.scss`, `_maps.scss`, `_custom.scss`, `theme.scss` qui importe Bootstrap selon l'« option B »), `styles/theme[.min].css`, `tinymce-templates/`. `npm install && npm run build` (ou `npm run watch`). Surcharger les variables Bootstrap et Barceloneta (`$primary`, `$font-family-sans-serif`, `$enable-rounded`, `$grid-main-breakpoint`, `$navbar-barceloneta-background`, `$footer-bg`…) et les cartes (`$state-colors` pour un état de workflow maison).
3. **Thème à partir de zéro** : même mécanique sans les imports Barceloneta.
4. **Diazo** (`plone.app.theming`) : une page HTML statique fournie par le graphiste (`index.html` + CSS/JS) et un `rules.xml` qui y injecte les morceaux produits par Plone :

```xml
<rules xmlns="http://namespaces.plone.org/diazo" xmlns:css="http://namespaces.plone.org/diazo/css"
       xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <theme href="index.html" />
  <notheme css:if-not-content="#visual-portal-wrapper" />
  <notheme if="$ajax_load" />
  <replace css:theme-children="#content" css:content-children="article#content" css:if-content="#content" />
  <replace css:theme="#main-nav" css:content="#portal-globalnav" />
  <drop css:content="#portal-personaltools" />
  <before css:theme-children="head" css:content="head link, head style, head script" />
</rules>
```

`manifest.cfg` déclare titre, règles, `production-css`, `tinymce-content-css`, et des paramètres (`[theme:parameters] ajax_load = python:request.get('ajax_load')`). Conseils : versionner ou hacher les noms de fichiers CSS/JS, garder peu de règles (chaque règle coûte à chaque rendu), surcharger un gabarit Plone plutôt qu'écrire du XSLT compliqué, désactiver Diazo pour les requêtes AJAX (`ajax_load`, natif depuis Barceloneta 3.3). Le thème s'active dans le panneau Thème ; `plone.app.theming.interfaces.IThemeSettings.enabled = False` coupe Diazo. Avec **lxml 6** (Plone 6.2), vérifiez le rendu du thème : l'analyse HTML a légèrement changé.

**Modes de couleur** (Bootstrap 5.3, Plone ≥ 6.0.6) : un attribut `data-bs-theme="dark|light"` sur `<html>`, posé par un petit script enregistré en bundle, et des boutons `data-bs-theme-value` ; un élément peut forcer son propre mode. Vérification visuelle : `/@@test-rendering-cheatsheet`.

# 49. Icônes et images adaptatives

**Icônes** : Bootstrap Icons, enregistrées dans le registre sous `plone.icon.<nom>` (jeux Bootstrap, types de contenu, MIME, drapeaux, Plone, barre d'outils) ; `icon_expr` d'un FTI ou d'une action vaut `string:<nom>`. Résolution : `/@@iconresolver/<nom>` ; dans un gabarit `${python:icons.url('alarm')}`, `tal:replace="structure python:icons.tag('archive', tag_class='me-1')"` ou `structure icons/alarm`. Surcharger `plone.icon.plone-copy` dans un `registry/icons.xml` change l'icône partout.

**Images** : les champs `NamedBlobImage` sont servis et redimensionnés par la vue `@@images` (`plone.namedfile.scaling`). Échelles par défaut (panneau Images, `plone.allowed_sizes`) : `huge 1600`, `great 1200`, `larger 1000`, `large 800`, `teaser 600`, `preview 400`, `mini 200` (hauteur libre), `thumb 128`, `tile 64`, `icon 32`, `listing 16`. URL : `/obj/@@images/image/preview` (par nom) ou `/obj/@@images/<uid>.jpeg` (stable et cacheable ; change quand l'image ou l'échelle change). Depuis 6.0, l'échelle n'est **générée qu'à la première requête** du navigateur, et les informations d'échelles sont dans la colonne de catalogue `image_scales` (`brain.image_scales["image"][0]["scales"]["preview"]["download"]`) pour les listes sans réveil d'objets.

```html
<tal:img define="images context/@@images">
  <img tal:replace="structure python:images.tag('image', scale='preview', css_class='img-fluid', alt=context.title)" />
  <img tal:replace="structure python:images.picture('image', picture_variant='medium')" />
  <img tal:replace="structure python:images.srcset(sizes='90vw', loading='lazy')" />
</tal:img>
```

`mode` : `scale` (défaut, pas d'agrandissement), `contain`, `cover`. Les **variantes `picture`** (panneau Images) définissent des `srcset` et des media queries pour l'art direction ; elles sont proposées dans TinyMCE (`hideInEditor` pour les réserver aux gabarits). Vue de contrôle : `/une-image.jpg/@@images-test`.

# 50. TinyMCE 8

Plone 6.2 embarque TinyMCE 8 (6.0 : 5.10 ; 6.1 : 7). Panneau `/@@tinymce-controlpanel` (schéma `ITinyMCESchema`, préfixe `plone`) : plugins et barre d'outils, menus (`menu` JSON), formats (`formats`, `inline_styles`, `block_styles`, `header_styles`, `alignment_styles`, `table_styles`), CSS de contenu (`content_css`), modèles (`templates`, plugin intégré, chaque modèle exige `title`, `description`, `url`), options avancées (`other_settings`, par exemple `{"sandbox_iframes": false}` ou `sandbox_iframes_exclusions`), et, nouveauté 6.2, la **clé de licence commerciale** (préfixe `T8LK:`), le plugin `licensekeymanager` devant être fourni comme ressource statique dans `custom_plugins`. Tout se configure aussi en profil (`registry/tinymce.xml`) ; attention, `purge="true"` sur `formats` remplace les valeurs par défaut. Un fichier `tinymce-formats.css` enregistré et listé dans « CSS de l'éditeur » injecte ses classes dans le menu Formats. Le filtrage HTML (panneau Sécurité ‣ Filtrage HTML) décide des balises et attributs conservés à l'enregistrement : à élargir pour `details`/`summary`/`open` (accordéons) ou les `iframe`.

# 51. Protection CSRF

`plone.protect` protège automatiquement : un jeton `_authenticator` est injecté dans tous les formulaires des utilisateurs connectés (transformation de sortie) et vérifié **à la validation de la transaction** ; une écriture en base sans jeton valide est refusée, avec redirection vers une page de confirmation. Pour du code qui écrit hors base (API externe) :

```python
from plone.protect import CheckAuthenticator, PostOnly, protect

@protect(CheckAuthenticator)
def call_external(self): ...

@protect(PostOnly)
def only_post(self): ...
```

Ajouter le jeton : paramètre GET `_authenticator`, champ caché `<span tal:replace="structure context/@@authenticator/authenticator" />`, lien `?_authenticator=${context/@@authenticator/token}`, en-tête `X-CSRF-TOKEN` (`createToken()` en Python). Autoriser une écriture légitime sans jeton (un compteur de vues, par exemple) : `safeWrite(obj, request)` sur l'objet précis, ou `alsoProvides(request, IDisableCSRFProtection)` pour toute la requête. `PLONE_CSRF_DISABLED=true` désactive tout : jamais en production.

# Partie VI — REST API et Volto

# 52. `plone.restapi`

`plone.restapi` (10.x en 6.2) expose tout le site en **API REST hypermédia** : le point d'entrée est la racine du site, chaque réponse contient des liens (`@id`) à suivre. Activation par **négociation de contenu** :

```bash
curl -i http://localhost:8080/Plone -H "Accept: application/json" --user admin:admin
```

ou par le préfixe `++api++` dans l'URL (`http://localhost:8080/Plone/++api++/schedule`). Une réponse de contenu comprend `@id`, `@type`, `UID`, les champs du schéma et des comportements sérialisés, `review_state`, `parent`, `items` (enfants), `lock`, `version`, et `@components` (liens vers `@actions`, `@breadcrumbs`, `@navigation`, `@workflow`, `@types`…). Les composants s'embarquent dans la même réponse avec `?expand=breadcrumbs,navigation`.

**Authentification** : Basic en développement ; en production **JWT** : `POST /@login {"login": "…", "password": "…"}` renvoie un `token` à passer en `Authorization: Bearer …` (`@login-renew`, `@logout`). Les permissions Plone s'appliquent intégralement.

**Manipulation** :

```bash
# créer
curl -X POST http://localhost:8080/Plone/schedule -H "Accept: application/json" -H "Content-Type: application/json" \
  --user admin:admin -d '{"@type": "talk", "title": "REST en pratique", "speaker": "Alice", "email": "a@x.org",
  "type_of_talk": "talk", "details": {"content-type": "text/html", "data": "<p>…</p>"}}'
# modifier (PATCH = partiel, PUT = remplacement)
curl -X PATCH http://localhost:8080/Plone/schedule/rest-en-pratique -H "Accept: application/json" \
  -H "Content-Type: application/json" --user admin:admin -d '{"room": "auditorium"}'
# workflow
curl -X POST http://localhost:8080/Plone/schedule/rest-en-pratique/@workflow/accept -H "Accept: application/json" --user admin:admin
# recherche
curl "http://localhost:8080/Plone/@search?portal_type=talk&review_state=accepted&sort_on=sortable_title&metadata_fields=room" -H "Accept: application/json"
# supprimer
curl -X DELETE http://localhost:8080/Plone/schedule/rest-en-pratique --user admin:admin
```

Services principaux : `@search` (requêtes de catalogue, pagination `b_start`/`b_size`, `fullobjects`), `@types` et `@types/<type>` (schéma JSON d'un type, modifiable pour les types TTW), `@workflow`, `@history`, `@lock`, `@sharing`, `@users`, `@groups`, `@roles`, `@registry`, `@controlpanels`, `@vocabularies`, `@querystring`, `@querystring-search`, `@navigation`, `@breadcrumbs`, `@contextnavigation`, `@actions`, `@aliases`, `@translations`, `@comments`, `@copy`/`@move`, `@upgrade`, `@site`, `@system`, `@addons`, `@relations`, `@linkintegrity`, `@email-send`, `@email-notification`, `@userschema`, `@tus-upload` (envoi de fichiers par morceaux). Nouveautés 6.2 : `@export` et `@import` (`plone.exportimport`), `@blocktypes`, import/export CSV des utilisateurs, vérification stricte des paramètres booléens, `searchable_text` dans `@controlpanels`.

**Sérialisation** : dates en ISO 8601, `RichText` en `{"content-type", "data", "encoding"}`, fichiers et images en `{"download", "filename", "content-type", "size", "scales"}` (envoi en base64 ou par `@tus-upload`), relations en objets résumés, blocs Volto en JSON dans les champs `blocks` et `blocks_layout`.

**Étendre** : un **service** est une vue qui renvoie du JSON :

```python
from plone.restapi.services import Service

class TalkStats(Service):
    def reply(self):
        from plone import api
        return {"accepted": len(api.content.find(portal_type="talk", review_state="accepted"))}
```

```xml
<plone:service method="GET" name="@talk-stats" for="plone.base.interfaces.IPloneSiteRoot"
               factory=".services.TalkStats" permission="zope2.View" />
```

Sérialiseurs et désérialiseurs de champs et de blocs sont des adaptateurs (`IFieldSerializer`, `IBlockFieldSerializationTransformer`…) ; `plone.rest` fournit le routage des verbes HTTP et de `++api++`. Pour explorer l'API, la documentation recommande Postman.

# 53. Volto en bref

**Volto** est le frontend React par défaut des nouveaux sites : application monopage avec rendu serveur (Node.js, port 3000), qui consomme l'API REST via un proxy interne `++api++`. Ses idées fortes :

- l'éditeur **Pastanaga** par **blocs** : texte (Slate), image, vidéo, listing (requête de catalogue), table des matières, grille, carte, HTML ; chaque bloc a un schéma et un composant d'édition et d'affichage ; les blocs sont stockés en JSON dans le contenu (`volto.blocks`) ;
- la **configuration** est un objet `config` étendu par des **add-ons** npm (`volto-light-theme`, `volto-slate`, `volto-form-block`…) ;
- le **thème** : Volto Light Theme, composants `@plone/components` (dans le cœur depuis Volto 19) ;
- l'outillage : **pnpm 10** et workspaces, Vitest (Jest retiré), Cypress/Playwright pour l'acceptation, `mrs-developer` pour les add-ons en développement, images Docker `plone/plone-frontend`.

Volto 19 (pour Plone 6.2) apporte : sites servis sous un **sous-chemin** (`example.org/intranet`), **restauration du contenu non enregistré** (stockage local du navigateur), **glisser-déposer de fichiers** dans le contenu du dossier, **panneau de configuration des blocs** (où et combien de fois chaque bloc est utilisé), **widget d'image unifié**, recherche multilingue dans le sélecteur de blocs, langue par défaut lue depuis le backend, abandon de Node 20. Côté Python, `plone.volto` ajoute les échelles `2k` et `4k` et le service `@blocktypes`. Pour approfondir : documentation Volto et formation « Effective Volto ».

Migration Classic → Volto : installer `plone.volto`, appliquer son profil, convertir le HTML des pages en blocs (`plone.volto` fournit un convertisseur ; `collective.exportimport` et `blocks-conversion-tool` pour les gros sites). Le chemin inverse n'existe pas.

# Partie VII — Production

# 54. Composants d'un déploiement

Un déploiement Plone comprend au minimum : le **backend** Python (une ou plusieurs instances), le frontend (**Volto** sur Node.js, ou rien de plus pour Classic UI), un **proxy de terminaison TLS** et routeur (nginx, Traefik, HAProxy, Apache), une **base** (FileStorage mono-instance, ZEO ou RelStorage multi-instances), un **service de courrier**, et des **tâches de maintenance** (sauvegarde, pack, rotation des journaux). Recommandé : un **cache HTTP** (Varnish ou CDN). Optionnel : supervision, moteur de recherche externe (Solr, Elasticsearch via `collective.solr` ou `collective.elasticsearch`).

Topologie Classic UI classique :

```text
Internet ──► nginx (TLS, gzip, statiques, en-têtes) ──► Varnish ──► HAProxy ──► Zope ×N (ZEO clients) ──► ZEO / PostgreSQL
                                                                                    └─ blobstorage partagé
```

Le proxy réécrit les URL avec **VirtualHostMonster** : `http://zope:8080/VirtualHostBase/https/www.example.org:443/Plone/VirtualHostRoot/` pour servir le site `Plone` à la racine du domaine, ou `…/VirtualHostRoot/_vh_intranet/` pour un sous-chemin. En nginx :

```nginx
location / {
    proxy_pass http://127.0.0.1:8080/VirtualHostBase/https/$host:443/Plone/VirtualHostRoot/;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size 200m;   # taille des envois ; Waitress a aussi sa propre limite (max_request_body_size)
}
```

Préparation du serveur : système où Plone tourne (Debian/Ubuntu, Python 3.10+ ou binaire `uv`), RAM et disque dimensionnés (§7), clé SSH, pare-feu, DNS, et interdiction d'exposer le port 8080 directement. Un serveur sans accès à Internet s'installe à partir d'un lot hors ligne (wheels + interpréteur), voir [[Plone pyproject tooling]].

# 55. Cache HTTP et `plone.app.caching`

`plone.app.caching` est installé mais **désactivé par défaut**. Panneau Cache (`/@@caching-controlpanel`, Classic UI seulement) :

1. **Importer un profil** : « With caching proxy » (Varnish ou CDN) ou « Without caching proxy » (navigateur seul), avec capture d'un instantané pour pouvoir revenir en arrière ;
2. **Activer** le cache (Global settings ‣ Enable caching) ;
3. ajuster les **règles** (rulesets) par type de ressource : `plone.content.itemView`, `plone.content.folderView`, `plone.content.file`, `plone.resource`, `plone.stableResource`, chacune associée à une **opération** (`strong caching`, `moderate caching`, `weak caching`, `no caching`, `chain`, avec `maxage`, `smaxage`, `etags`, `vary`, `ramCache`) ;
4. configurer la **purge** (`plone.cachepurging`) : adresse du proxy, hôtes virtuels, domaines alias, types purgés (`File` et `Image` par défaut), onglet Purge manuel.

Stratégies : URL changeantes (ressources Plone hachées, échelles d'images à UID), délais courts (`s-maxage` de quelques secondes sur les pages, « terse »), `PURGE` à la modification, ou pas de cache pour les pages de liste. Les **ETags** composent des composantes (`userid`, `roles`, `language`, `lastModified`, `catalogCounter`, `skin`, `resourceRegistries`) ; le **cache RAM** garde des réponses complètes. Pour les anonymes, viser « tout en cache sauf les formulaires », pour les connectés « ressources statiques seulement ». Varnish : configurer un VCL qui respecte `Cache-Control`/`Surrogate-Control`, accepte `PURGE` depuis le backend et ne met pas en cache les réponses avec `Set-Cookie` ou `__ac`. Vérifier avec les en-têtes `X-Cache-Rule`, `X-Cache-Operation`, `Age`.

# 56. Conteneurs et orchestration

Cookieplone génère des `Dockerfile` pour le backend (image dérivée de `plone/plone-backend:6.2` avec le paquet du projet installé) et le frontend, une pile Docker Compose de développement, des **stacks Docker Swarm** et des playbooks **Ansible** (option) : `make -C devops help`, `deploy`, `deploy-check`, `deploy-rollback`, `secrets-edit` (secrets chiffrés), `lint`. La CI (GitHub Actions ou GitLab) construit les images, lance les tests et les tests d'acceptation en conteneurs (`make ci-acceptance-test`), puis déploie.

Compose minimal Classic UI avec PostgreSQL et Varnish :

```yaml
services:
  backend:
    image: ghcr.io/monorg/monsite-backend:1.0.0
    environment:
      RELSTORAGE_DSN: "dbname='plone' user='plone' host='db' password='${DB_PASSWORD}'"
      ZODB_CACHE_SIZE: "100000"
    depends_on: [db]
  db:
    image: postgres:16
    environment: { POSTGRES_USER: plone, POSTGRES_PASSWORD: "${DB_PASSWORD}", POSTGRES_DB: plone }
    volumes: [pgdata:/var/lib/postgresql/data]
  varnish:
    image: plone/plone-varnish:latest
    environment: { BACKENDS: "backend:8080" }
  proxy:
    image: traefik:v3
    # TLS, routage vers varnish, VirtualHostMonster dans le chemin du backend
volumes: { pgdata: {} }
```

Données : volumes nommés (jamais la couche écrivable du conteneur), sauvegardes `pg_dump` + blobs, voir [[Docker]]. Les variables `ADDONS` et `DEVELOP` servent au développement, pas à la production : construisez une image qui contient votre code. Pour Kubernetes, les mêmes images, un `LISTEN_PORT` si plusieurs Zope partagent un espace réseau, et une sonde HTTP sur `/Plone/@@ok`.

# 57. Réglages de performance

- **Threads** (`wsgi_threads`, 2 à 4 par instance) et **nombre d'instances** ZEO/RelStorage derrière un équilibreur : Zope est limité par le GIL, on scale en processus ;
- **`zodb-cache-size`** (objets par connexion, 30 000 à 100 000 selon la RAM : chaque thread a sa connexion) et `cache-size-bytes` ; ZEO : `cache-size` du client (128 Mo par défaut) et `drop-cache-rather-verify` ; RelStorage : `cache-local-mb`, `blob-cache-size` ;
- **Chameleon** : cache compilé sur disque (`chameleon_cache` dans `instance.yaml`), `debug_mode: false` ;
- **catalogue** : des colonnes de métadonnées plutôt que `getObject()` dans les listes, `image_scales` pour les images, `sort_limit` ;
- **cache HTTP** (§55) et **cache mémoire** (`plone.memoize` : `@ram.cache`, `@view.memoize`, `@instance.memoize`, `@forever.memoize`) ;
- **pack** régulier et taille des blobs ; `Data.fs` sur SSD ;
- supervision : journaux d'accès (temps de réponse), Munin ou Prometheus via les exporteurs de la communauté, `haufe.requestmonitoring` pour les requêtes lentes, et le panneau « Vue d'ensemble » qui liste les versions.

# 58. Mettre à jour Plone : stratégies et guides 6.0, 6.1, 6.2

Deux stratégies :

- **en place** (`@@plone-upgrade`) : on installe la nouvelle version du code, on démarre, et l'outil de migration de `plone.app.upgrade` applique les étapes de profil (`Products.CMFPlone:plone`, par exemple 6111 → 6204). Recommandée pour les versions mineures et la plupart des majeures ; ne pas sauter une majeure (4 → 5 → 6) ;
- **export-import** (`collective.exportimport`, `plone.exportimport`) : exporter contenus et réglages d'un vieux site, importer dans un site neuf ; recommandée pour les sites très anciens, Archetypes, ou très complexes.

Méthode : lire les notes de version et le guide « version-specific » de chaque saut ; vérifier Python, Zope, et la compatibilité de **chaque module complémentaire** ; mettre les modules à jour d'abord ; **sauvegarder** ; **répéter la migration sur une copie** (`var/filestorage/` d'un répertoire de travail, jalons avant chaque `@@plone-upgrade`) ; comparer les volumes avant et après ; ne jamais migrer la production directement. Dépannage : journaux, test sans personnalisations, sans modules, sur un site neuf, puis rapport de bug.

Points marquants des trois derniers guides :

**5.2 → 6.0** : Python 3.8–3.11, Zope 5 (plus de Python 2 ni de ZServer), fin d'Archetypes, racine du site Dexterity (le site paraît vide avant `@@plone-upgrade`), `portal_quickinstaller` supprimé, `z3c.autoinclude` → `plone.autoinclude` (plus d'`includeDependencies`), gabarits Bootstrap 5 et Barceloneta réécrite, Mockup/ES6 et registre de ressources simplifié, alias de vues retirés (`folder_listing` → `listing_view`…), viewlets renommés, propriétés Unicode obsolètes, `temp_folder` retiré, `plone.base` (imports de compatibilité avec avertissements), `collective.dexteritytextindexer` intégré (`plone.textindexer`), nouvelles échelles d'images et pré-enregistrement (variable `UPDATE_CATALOG_FOR_IMAGE_SCALES=0` pour différer), TinyMCE 5, `Bool` avec `required=False`, panneau Relations.

**6.0 → 6.1** : Python 3.10–3.13, TinyMCE 7, `ContentBrowserWidget` (`RelatedItemsWidget` obsolète), widgets dans `plone.app.z3cform.widgets`, `plone.app.multilingual` et `plone.app.discussion` deviennent des modules du cœur (commentaires désactivés par défaut, migration bloquante s'il existe des commentaires sans le paquet), distributions et `ALLOWED_DISTRIBUTIONS`, dépendre de `plone.classicui` pour un site Classic.

**6.1 → 6.2** : Python 3.14 accepté, Zope 6, **espaces de noms natifs** (§59), début de séparation Classic UI / cœur (gabarits déplacés vers `plone.app.layout`, module `plone.app.layout` à **activer** en Classic UI et à ne pas activer en Volto ; la migration tente de le faire), TinyMCE 8 avec clé de licence, lxml 6 (vérifier les thèmes Diazo), icalendar avec ruptures, `zc.buildout` 5, nouveaux services REST, durcissements de sécurité (`isurlinportal`, option de purge de session à la connexion dans PAS).

# 59. Espaces de noms natifs et outillage Python en 6.2

Plone utilisait depuis Python 2 des **espaces de noms `pkg_resources`** (`__init__.py` avec `declare_namespace`) pour `plone.*`, `Products.*`, `collective.*`. **setuptools 82** a retiré `pkg_resources` ; Plone 6.2 a donc converti tout le cœur aux **espaces de noms natifs** (PEP 420 : pas d'`__init__.py` dans `src/plone/`). Règles :

- dans un même espace de noms, **ne mélangez jamais** paquets `pkg_resources` et paquets natifs : les portions natives deviennent invisibles (`ModuleNotFoundError`, « Package not found ») ;
- avec des versions publiées sur PyPI, `pip` et `uv` s'en sortent, `zc.buildout` 4 non, `zc.buildout` 5 oui ; avec des installations **éditables**, le paquet `horse-with-no-namespace` (installé par `pip` dans le venv, pas par Buildout) réconcilie les deux styles, d'où la ligne 🐎 au démarrage des projets Cookieplone ;
- setuptools 82 n'est utilisable qu'avec pip/uv et uniquement si **aucun** paquet installé n'utilise encore `pkg_resources` ; Buildout a encore besoin de setuptools ≤ 81 ;
- convertissez vos propres paquets, **tous ceux d'un même espace de noms en une fois**, avant le premier `pip install -e` : `uvx --from plone.meta switch-to-pep420 --no-tests .` fait le travail (supprime les `__init__.py` d'espace de noms, met `pyproject.toml` à jour), puis comparez le nombre de tests et de fichiers dans le wheel avant et après ;
- `pkg_resources` ne doit plus être importé dans votre code : `importlib.metadata` (`version("mon.paquet")`), `importlib.resources` (`files("mon.paquet") / "data.json"`).

Outillage de référence de Plone 6.2 : `pip 26.1`, `setuptools 81`, `wheel 0.47`, `zc.buildout 5.2`, `horse-with-no-namespace 20260202`, `uv` pour l'installation de Python et des projets Cookieplone, `plone.meta` pour la configuration standardisée des paquets (Ruff, tox, GitHub Actions, `.meta.toml`), classifieurs `Framework :: Plone :: 6.2`.

# Partie VIII — Travaux pratiques

Les TP forment une progression : un site de programme de conférence en Classic UI, du vide jusqu'au déploiement. Chaque TP indique un résultat vérifiable.

# 60. TP1 — Un site Classic UI avec Cookieplone

1. Installer `uv`, `make`, `git`. Générer un projet : `uvx cookieplone project`, répondre `No` à « Use Volto as frontend », stockage « Local FileStorage », pas de cache ni d'Ansible pour l'instant.
2. `make install` puis `make backend-start`. Ouvrir `http://localhost:8080/Plone`, se connecter `admin`/`admin`.
3. Créer un dossier « Programme », deux pages, une actualité avec image, un événement récurrent. Publier l'actualité, laisser une page privée.
4. Dans Configuration du site : changer le titre et le logo, activer les menus déroulants de navigation (profondeur 2), régler le courriel sur un serveur de test.
5. Vérifier `/@@test-rendering-cheatsheet` et `/@@manage-viewlets`.

**Résultat attendu** : un visiteur anonyme voit l'actualité et l'événement, pas la page privée ; le journal `instance/var/log/instance.log` ne contient aucune trace d'erreur.

# 61. TP2 — Un module complémentaire et un type « Conférence »

1. Dans `backend/src/project/title/`, créer `content/talk.py` avec le schéma `ITalk` du §27 (champs `type_of_talk`, `details`, `audience`, `speaker`, `email`, `room`, `duration`), `profiles/default/types.xml` et `types/talk.xml`, la permission `project.title.AddTalk` dans `permissions.zcml` et `rolemap.xml`.
2. Créer les vocabulaires `TalkTypes`, `Audiences`, `Rooms` (§30) et les enregistrer.
3. Créer un type conteneur `talks` (klass `Container`, `filter_content_types` = `talk`) ; rendre `talk` non ajoutable globalement.
4. Ajouter `catalog.xml` avec un index et une colonne `room`, et l'indexeur correspondant.
5. `make backend-build`, redémarrer, réappliquer le profil (désactiver/réactiver le module ou étape de migration), créer un dossier `talks` et trois conférences.

**Résultat attendu** : `api.content.find(portal_type="talk", room="auditorium")` dans `make -C backend console` renvoie les bonnes notices avec `brain.room` sans réveiller les objets ; le formulaire d'ajout montre deux onglets.

# 62. TP3 — Un comportement, un vocabulaire, un panneau de configuration

1. Écrire le comportement `project.title.featured` (schéma seul, §29) et l'activer sur `talk` et `News Item` par `types/News Item.xml` avec `purge="false"`.
2. Créer le panneau de configuration `IConferenceSettings` (§36) avec `talk_submission_open` et `rooms` ; faire pointer le vocabulaire `Rooms` sur l'enregistrement du registre.
3. Ajouter un abonné `IObjectAddedEvent` sur `ITalk` qui envoie un courriel aux organisateurs via `api.portal.send_email` si `talk_submission_open` est vrai.
4. Écrire deux tests `pytest-plone` : le comportement est actif sur le FTI ; la création d'une conférence déclenche un courriel (`Products.MailHost` en mode test, `portal.MailHost.messages`).

**Résultat attendu** : le panneau apparaît dans Configuration du site, les salles saisies alimentent le menu « Room » du formulaire, `make backend-test` passe.

# 63. TP4 — Une vue, un viewlet et un thème

1. Vue `@@talk-list` (§42) sur le conteneur `talks`, ajoutée au menu Affichage ; afficher titre, intervenant, salle, et une image `preview` via `images.tag`.
2. Viewlet « partenaires » dans `plone.portalfooter` (§45), ordonné en premier par `viewlets.xml`.
3. Surcharger `plone.app.layout.viewlets.logo.pt` avec `z3c.jbot` pour ajouter un slogan.
4. Créer `plonetheme.conference` avec `plonecli add theme_barceloneta`, changer `$primary` et la police, compiler (`npm run build`), installer le thème dans le projet (`mx.ini` ou chemin local) et l'activer.
5. Enregistrer un bundle `conference` avec un CSS et un JavaScript (`depends = plone`).

**Résultat attendu** : la page `/talks/@@talk-list` rend la liste avec le thème ; `/@@test-rendering-icons` affiche l'icône `mic` ; aucun avertissement dans la console du navigateur.

# 64. TP5 — Un workflow sur mesure

1. Dans la ZMI, copier `simple_publication_workflow` en `talk_workflow`, renommer les états en `submitted`, `accepted`, `rejected`, les transitions en `accept`, `reject`, et ajuster les permissions par état (§37).
2. Exporter via `portal_setup` ‣ Export ‣ Workflow Tool, puis recopier `workflows.xml` et `workflows/talk_workflow/definition.xml` dans le profil, en liant le type `talk`.
3. Créer un groupe « Comité de programme » avec le rôle Reviewer ; donner à un utilisateur test le rôle Contributeur sur `talks` par le partage.
4. Tester : le contributeur crée et voit sa conférence, le comité la voit et l'accepte, l'anonyme ne la voit qu'acceptée.
5. Écrire l'étape de migration qui applique `workflow` et `updateRoleMappings()`.

**Résultat attendu** : `api.content.get_state(talk)` vaut `submitted` à la création ; après `accept`, l'anonyme obtient 200 sur l'URL, 401 avant.

# 65. TP6 — Une étape de migration

1. Passer `metadata.xml` à `1001` ; écrire `upgrades/v1001.py` qui recharge le profil, crée `schedule` s'il manque, déplace toutes les conférences dedans, et réindexe `room`.
2. Enregistrer `genericsetup:upgradeSteps`, redémarrer, lancer la migration depuis le panneau Modules complémentaires.
3. Rejouer sur une copie de la base (`Data.fs` copié dans un second répertoire d'instance) pour vérifier l'idempotence.

**Résultat attendu** : le journal montre les déplacements, un second lancement ne fait rien, `portal_setup` affiche la version 1001.

# 66. TP7 — Consommer la REST API

1. Avec `curl` ou `httpie`, obtenir un JWT (`@login`), lister les conférences acceptées (`@search`), en créer une (`POST`), changer sa salle (`PATCH`), l'accepter (`@workflow/accept`), lire `@types/talk`.
2. Écrire un service `@talk-stats` (§52) et son test.
3. Exporter le site avec `@export` et observer l'arborescence produite.

**Résultat attendu** : un script Python `requests` de 40 lignes reproduit toute la séquence ; les codes HTTP sont 200/201/204 et 401 sans jeton.

# 67. TP8 — Déployer avec Docker Compose et Varnish

1. `make build-images` ; composer une pile backend + PostgreSQL + Varnish + Traefik (§56) avec un `.env` pour les secrets.
2. Importer le profil de cache « With caching proxy », activer la purge vers `varnish:80`.
3. Mesurer : `curl -I` sur une page publique deux fois (en-têtes `Age`, `X-Cache-Rule`), puis modifier une image et vérifier la purge.
4. Sauvegarder (`pg_dump` + blobs) et restaurer dans une pile vierge.

**Résultat attendu** : la seconde requête est servie par Varnish ; la restauration donne un site identique.

# 68. TP9 — Migrer un projet 6.1 vers 6.2

1. Sur une copie du projet et de la base : passer `Products.CMFPlone` à `6.2.x`, mettre les contraintes à jour (`https://dist.plone.org/release/6.2-latest/constraints.txt`), `make backend-build`.
2. Convertir chaque paquet maison d'un espace de noms partagé aux espaces natifs (`switch-to-pep420`), tous en une fois ; reconstruire ; vérifier le wheel.
3. Démarrer, lancer `@@plone-upgrade`, vérifier que `plone.app.layout` est activé, que le thème Diazo se rend bien avec lxml 6, que TinyMCE 8 fonctionne.
4. Reconstruire le catalogue si nécessaire, comparer le nombre d'objets avant et après, puis seulement packer.

**Résultat attendu** : plus aucun `DeprecationWarning` lié à `pkg_resources` dans le journal ; la vue d'ensemble affiche 6.2.x ; les tests passent.

# 69. Erreurs fréquentes

| Symptôme | Cause probable | Remède |
|---|---|---|
| `ModuleNotFoundError: No module named 'plone.xxx'` au démarrage | mélange d'espaces de noms `pkg_resources` et natifs | convertir tout l'espace de noms, ou `horse-with-no-namespace` via pip ; Buildout 5 |
| le ZCML du module n'est pas chargé | pas de point d'entrée `plone.autoinclude.plugin` et pas de `zcml_package_includes` | ajouter le point d'entrée ; `module =` si le nom diffère |
| thème, gabarits ou profils absents du wheel | `package-data` non déclaré | `[tool.setuptools.package-data]`, contrôler avec `python -m build` |
| `Unauthorized` dans un script `zconsole` | pas de gestionnaire de sécurité | `newSecurityManager(None, system)`, `makerequest(app)` |
| rien n'est enregistré après un script | pas de `transaction.commit()` | commit explicite |
| le catalogue ne « voit » pas les objets créés dans le même script | file d'indexation différée | `processQueue()` avant de mesurer |
| `ContentProviderLookupError: plone.htmlhead` | rendu d'une page hors chaîne d'acquisition | `view.__of__(context)`, vraie requête |
| les permissions ne changent pas après modification du workflow | mappage de rôles non recalculé | Update security settings / `updateRoleMappings()` |
| un champ `Bool` impossible à décocher | `required=True` par défaut | `required=False` |
| toutes les instances d'un type partagent la même liste | `default=[]` | `defaultFactory` |
| la surcharge `z3c.jbot` est ignorée | fichier ajouté sans redémarrage, ou nom erroné | redémarrer ; nom = chemin pointé du `.pt` d'origine |
| écriture refusée, page « Confirmer l'action » | jeton CSRF absent | `@@authenticator`, `safeWrite`, `IDisableCSRFProtection` |
| le panneau Cache n'existe pas | `plone.app.caching` non installé ou site créé « avancé » sans cache | ajouter le paquet, importer `profile-plone.app.caching:default` |
| les commentaires ont disparu après 6.1 | `plone.app.discussion` non déclaré | ajouter la dépendance, activer le module |
| la migration 6.1 → 6.2 laisse un thème cassé | lxml 6 | corriger les règles Diazo, vérifier le HTML du thème |
| 413 à l'envoi d'un gros fichier | limite du proxy ou de Waitress | `client_max_body_size`, `max_request_body_size` |
| `ConflictError` fréquents | écritures concurrentes sur le même objet (compteurs, catalogue) | lots plus petits, `BTrees`, cache, moins de threads |
| site vide juste après le passage en 6.0 | racine Dexterity non migrée | lancer `@@plone-upgrade` |

# 70. Commandes de référence

```bash
# projet Cookieplone
uvx cookieplone project            # générer
make install                       # backend (+ frontend)
make backend-start                 # 8080 ; make frontend-start → 3000
make backend-build                 # réinstaller après changement de dépendances
make backend-create-site           # (re)créer le site
make -C backend console            # zconsole debug
make format lint check test i18n   # qualité, tests, traductions
make build-images ; make -C devops help

# pip seul
python3 -m venv venv && venv/bin/pip install -c https://dist.plone.org/release/6-latest/constraints.txt Plone pipx
venv/bin/pipx run cookiecutter -f --no-input --config-file instance.yaml gh:plone/cookiecutter-zope-instance
venv/bin/runwsgi -v instance/etc/zope.ini
venv/bin/zconsole debug instance/etc/zope.conf
venv/bin/zconsole run instance/etc/zope.conf script.py
venv/bin/addzopeuser -c instance/etc/zope.conf admin 'secret'
venv/bin/plone-exporter instance/etc/zope.conf Plone /tmp/export
venv/bin/plone-importer instance/etc/zope.conf Plone /tmp/export

# conteneurs
docker run -d --name backend -e SITE=Plone -e TYPE=classic -p 8080:8080 plone/plone-backend:6.2

# plonecli
uvx --from plonecli plonecli create addon collective.x
plonecli add content_type | behavior | view | viewlet | form | vocabulary | controlpanel | theme_barceloneta | mockup_pattern | restapi_service | upgrade_step

# espaces de noms natifs
uvx --from plone.meta switch-to-pep420 --no-tests .
uvx --from build pyproject-build && unzip -l dist/*.whl

# REST
curl -s http://localhost:8080/Plone/@search?portal_type=Document -H "Accept: application/json" --user admin:admin | jq .
```

URL utiles dans un site : `/@@overview-controlpanel`, `/prefs_install_products_form`, `/portal_registry`, `/@@dexterity-types`, `/@@manage-viewlets`, `/@@manage-portlets`, `/@@inspect-relations`, `/@@caching-controlpanel`, `/@@maintenance-controlpanel`, `/@@redirection-controlpanel`, `/@@tinymce-controlpanel`, `/@@imaging-controlpanel`, `/@@plone-upgrade`, `/@@test-rendering`, `/@@images-test`, `/manage_main` (ZMI), `/portal_setup`, `/portal_workflow`, `/error_log`.

# 71. Ce qu'il faut retenir

```text
Python 3.10–3.14
  │
ZODB (FileStorage | ZEO | RelStorage) + blobs + catalogue
  │
Zope 6 : ZPublisher (traversal, acquisition), AccessControl, ZCA (interfaces, adaptateurs, utilitaires, événements), Chameleon
  │
CMF : types (FTI), DCWorkflow, GenericSetup
  │
plone.base → cœur plone.* / plone.app.* → Products.CMFPlone → plone.api, plone.restapi, distributions, modules du cœur → Plone
  │                                           │
Classic UI (vues, viewlets, portlets, Barceloneta, Diazo, Mockup, TinyMCE 8)   Volto 19 (React, blocs) via ++api++
```

1. **Plone est une pile** : savoir à quelle couche appartient un problème (ZODB, Zope, CMF, Plone, interface) est la moitié du diagnostic.
2. **Tout est composant** nommé sur des interfaces, une couche et une permission ; on surcharge, on ne modifie pas.
3. **Le schéma pilote tout** : formulaires, validation, sérialisation REST ; comportements et directives l'enrichissent sans toucher à la classe.
4. **La configuration persistante passe par GenericSetup** et le **registre** ; les changements sur un site existant passent par des **étapes de migration**, jamais par la réinstallation.
5. **La sécurité est dans les objets** : permissions par état de workflow, rôles globaux et locaux, vérification à l'accès.
6. **Le catalogue sert à ne pas réveiller les objets** ; indexez ce que vous cherchez, mettez en colonne ce que vous affichez.
7. **Classic UI ou Volto** est un choix de projet, pas un choix technique du backend, qui reste le même.
8. **Cookieplone + uv + contraintes `dist.plone.org`** est la voie normale d'installation ; pip et Buildout 5 restent possibles.
9. **Répétez toute migration sur une copie**, du moins destructif au plus irréversible, le pack en dernier.
10. **Plone 6.2 = espaces de noms natifs** : un espace de noms se convertit en bloc, et `pkg_resources` disparaît de votre code.

# 72. Références

Documentation officielle Plone 6 (CC BY 4.0) :

```text
https://6.docs.plone.org/
https://6.docs.plone.org/install/create-project-cookieplone.html
https://6.docs.plone.org/admin-guide/
https://6.docs.plone.org/backend/
https://6.docs.plone.org/classic-ui/
https://6.docs.plone.org/deployment/
https://6.docs.plone.org/plone.api/
https://6.docs.plone.org/plone.restapi/docs/source/
https://6.docs.plone.org/volto/
https://6.docs.plone.org/backend/upgrading/version-specific-migration/upgrade-to-62.html
https://6.docs.plone.org/conceptual-guides/
https://6.docs.plone.org/reference-guide/cookieplone-make-commands.html
```

Formations et versions :

```text
https://training.plone.org/            (Mastering Plone 6, Theming Classic UI, Deployment, Effective Volto)
https://plone.org/download/releases/6.2.0
https://dist.plone.org/release/6.2-latest/
https://plone.org/news-and-events/news/news-for-2026/plone-6-2-is-here
https://github.com/plone/cookieplone      https://github.com/plone/cookiecutter-zope-instance
https://github.com/plone/pytest-plone     https://github.com/plone/meta
https://plone.github.io/mockup/           https://docs.diazo.org/
https://zope.readthedocs.io/              https://zodb.org/
https://zopecomponent.readthedocs.io/     https://z3cform.readthedocs.io/
```

Voir aussi :

- [[Plone pyproject tooling]]
- [[ZC.Buildout]]
- [[Tal et Metal]]
- [[Docker]]
- [[Internationalisation]]
- [[Pyramid]]
