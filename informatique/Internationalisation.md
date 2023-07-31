# Plan

Introduction à la localisation et à l'internationalisation avec Babel et pybabel

1. Introduction à la localisation et à l'internationalisation
	1.1. Définitions et concepts clés
		1.1.1. Localisation vs internationalisation
		1.1.2. Pourquoi la localisation est importante pour les applications modernes

	1.2. Bases de la localisation
		1.2.1. Formats de fichiers de localisation courants (Gettext, JSON, XML)
		1.2.2. Extraction de messages à traduire à partir du code source

2. Présentation de Babel
	2.1. Qu'est-ce que Babel ?
		2.1.1. Vue d'ensemble de l'outil
		2.1.2. Avantages de l'utilisation de Babel dans le processus de localisation

	2.2. Configuration de Babel
		2.1. Installation et mise en place du projet
		2.2. Fichiers de configuration (babel.cfg)

	2.3. Utilisation de Babel pour la localisation
		2.3.1. Extraction des chaînes de caractères à traduire
		2.3.2. Génération des fichiers de catalogues de messages (.pot)
		2.3.3. Traduction des messages avec Babel

3. Introduction à pybabel
	3.1. Qu'est-ce que pybabel ?
		3.1.1. Vue d'ensemble de l'outil et ses fonctionnalités
	
	3.2. Configuration de pybabel
		3.2.1. Installation et mise en place
		3.2.2. Configuration du projet avec pybabel

	3.3. Principales fonctionnalités de pybabel
		3.3.1. Initialisation des catalogues de traduction (.po)
		3.3.2. Extraction des messages à traduire à partir du code source
		3.3.3. Traduction des messages avec pybabel

4. Intégration de Babel et pybabel dans un projet Pyramid utilisant chameleon et des templates tal/metal 
	4.1. Utilisation de Babel avec Pyramid
		4.1.1. Configuration de Pyramid pour la localisation
		4.1.2. Utilisation des fonctions de traduction dans les modèles et les vues
	4.2. Exemples pratiques d'utilisation de Babel et pybabel dans Pyramid
		4.2.1. Mise en place de la localisation dans une application Pyramid
		4.2.2. Traduction des messages avec pybabel

5. Bonnes pratiques en localisation
	5.1. Sélection des messages à traduire
	5.2. Gestion des pluriels et des contextes linguistiques
	5.3. Astuces pour des traductions de qualité
	5.4. Tests et validation des traductions

# 1. Introduction à la localisation et à l'internationalisation

## 1.1 Définitions et concepts clés

La localisation et l'internationalisation sont deux aspects essentiels du développement d'applications modernes destinées à un public mondial. Comprendre ces concepts clés est fondamental pour créer des logiciels multilingues et adaptés aux différentes cultures. Dans ce premier chapitre, nous allons explorer les définitions de la localisation et de l'internationalisation, ainsi que leur importance dans le contexte du développement logiciel.

### 1.1.1 Localisation vs Internationalisation

La localisation et l'internationalisation sont deux termes souvent utilisés de manière interchangeable, mais ils désignent en réalité deux phases distinctes du processus de création d'une application multilingue.

La **localisation** fait référence à l'adaptation d'une application pour répondre aux spécificités culturelles, linguistiques et régionales d'un public cible. Cela inclut la traduction des chaînes de caractères, la gestion des formats de date, d'heure et de devise propres à chaque pays, ainsi que l'ajustement des éléments graphiques et visuels pour s'harmoniser avec les attentes des utilisateurs locaux.

En revanche, **l'internationalisation** consiste à concevoir et à développer une application de manière à ce qu'elle puisse être facilement adaptée à différentes langues et régions, sans qu'il soit nécessaire de modifier en profondeur son code source. Il s'agit de créer une base solide qui permettra la localisation ultérieure de l'application sans altérer sa structure ou sa logique interne.

### 1.1.2 Pourquoi la localisation est importante pour les applications modernes

La localisation joue un rôle critique dans le succès des applications à l'échelle internationale. Voici quelques raisons clés pour lesquelles la localisation est essentielle :

1. **Accessibilité pour les utilisateurs** : En traduisant une application dans la langue maternelle de l'utilisateur, vous rendez le contenu plus compréhensible et accessible, ce qui encourage l'adoption de l'application par des publics plus vastes.

2. **Engagement des utilisateurs** : Les utilisateurs ont tendance à s'engager davantage avec une application qui parle leur langue et reflète leur culture. Cela favorise une expérience utilisateur positive et fidélise les utilisateurs.

3. **Expansion du marché** : En rendant votre application disponible dans plusieurs langues, vous augmentez son potentiel d'atteindre de nouveaux marchés et d'accroître son nombre d'utilisateurs.

4. **Respect de la diversité culturelle** : En prenant en compte les particularités culturelles des utilisateurs, vous démontrez un respect pour leur identité et leurs valeurs, ce qui peut renforcer l'image de marque de votre application.

5. **Conformité réglementaire** : Dans certains pays, il peut être requis par la loi de fournir une version localisée de l'application pour être conforme aux réglementations locales.

Au cours de ce cours, nous allons explorer deux outils `gettext` et `Babel` avec sont outil pybabel qui facilitent le processus de localisation et d'internationalisation des applications Python, en automatisant la gestion des traductions et en fournissant des outils puissants pour faciliter le travail des développeurs et des traducteurs.

*Note : Avant de passer à la pratique, il est important de noter que la localisation ne se limite pas uniquement aux langues. Elle englobe également la gestion des différentes variantes régionales d'une même langue (par exemple, l'anglais américain et l'anglais britannique). La prise en compte de ces nuances est essentielle pour offrir une expérience utilisateur optimale.*

# 1.2 Bases de la localisation

Dans ce chapitre, nous allons explorer les bases de la localisation en nous concentrant sur les formats de fichiers couramment utilisés et sur l'extraction des messages à traduire à partir du code source.

### 1.2.1 Formats de fichiers de localisation courants (Gettext, JSON, XML)

Lorsque nous localisons une application, nous devons savoir comment stocker les messages à traduire dans des fichiers spécifiques. Différents formats de fichiers peuvent être utilisés pour cette tâche, et chacun présente ses avantages et ses particularités. Voici trois formats courants que nous allons aborder :

**Gettext** : Le format Gettext est l'un des formats les plus traditionnels pour la localisation. Nous utilisons des fichiers avec l'extension .po pour stocker les messages source et leurs traductions. Ces fichiers .po seront ensuite compilés en fichiers binaires .mo pour être utilisés par l'application. Ce format est largement soutenu par de nombreux outils de localisation, ce qui en fait un choix populaire dans de nombreux projets.

**JSON** : Le format JSON (JavaScript Object Notation) est un format de données léger et largement répandu. Il est très apprécié pour sa lisibilité par les humains et sa facilité d'utilisation dans les applications web et mobiles. En ce qui concerne la localisation, les traductions sont souvent stockées dans des fichiers .json pour chaque langue cible. Ce format est particulièrement pratique pour les applications qui utilisent déjà JSON pour structurer leurs données.

**XML** : L'Extensible Markup Language (XML) est un format polyvalent pour structurer des données. Bien qu'il ne soit pas aussi populaire que le JSON pour la localisation, il est encore utilisé dans certains projets où une structure plus complexe est nécessaire pour gérer les traductions.

### 1.2.2 Extraction de messages à traduire à partir du code source

Avant de pouvoir traduire les messages d'une application, nous devons d'abord les extraire du code source. Cette étape d'extraction est essentielle pour identifier toutes les chaînes de caractères qui doivent être traduites. Heureusement, il existe des outils qui facilitent grandement ce processus.

L'un des outils les plus couramment utilisés pour l'extraction de messages est **Babel**. Avec Babel, nous pouvons parcourir le code source de notre application à la recherche de marqueurs spécifiques, tels que des fonctions de traduction, des balises ou des commentaires, qui indiquent qu'une chaîne de caractères doit être traduite. Une fois les messages extraits, Babel génère un fichier .pot (Portable Object Template) qui contient toutes les chaînes de caractères à traduire, ainsi que leur contexte et leur emplacement d'origine dans le code source.

Une fois que nous disposons du fichier .pot, nous pouvons le fournir aux traducteurs, qui créeront les fichiers .po (fichiers de catalogues de messages) correspondants pour chaque langue cible. Ces fichiers .po contiendront les traductions des messages extraites du fichier .pot.

Il est important de noter que l'extraction des messages à traduire doit être une étape régulière du processus de développement. Lorsque de nouvelles fonctionnalités sont ajoutées à l'application ou que du contenu est modifié, il est essentiel de mettre à jour le fichier .pot pour inclure les nouvelles chaînes de caractères.

# 2. Présentation de Babel

Dans ce chapitre, nous allons présenter Babel, un outil puissant et polyvalent utilisé pour faciliter la localisation des applications Python. Nous explorerons ce qu'est Babel, ses fonctionnalités clés, ainsi que les avantages qu'il offre dans le processus de localisation.

## 2.1. Qu'est-ce que Babel ?

Babel est un outil open-source conçu pour faciliter la localisation et l'internationalisation des applications écrites en Python. Il est développé par la communauté Python et est largement utilisé dans de nombreux projets Python, notamment les applications web basées sur des frameworks tels que Flask et Django.

### 2.1.1. Vue d'ensemble de l'outil

Babel offre une gamme complète de fonctionnalités pour simplifier le processus de localisation. Ses principales fonctionnalités comprennent :

**Extraction de messages** : Babel peut parcourir le code source de votre application et extraire les chaînes de caractères à traduire. Ces messages sont ensuite rassemblés dans un fichier .pot (Portable Object Template), qui servira de base pour les traductions.

**Gestion des traductions** : Babel permet de créer et de gérer facilement les fichiers .po (fichiers de catalogues de messages) pour chaque langue cible. Ces fichiers contiennent les traductions des messages extraits dans le fichier .pot.

**Compilation des traductions** : Une fois que les traducteurs ont complété les fichiers .po avec les traductions, Babel peut les compiler en fichiers binaires .mo, qui seront utilisés par l'application pour afficher les messages traduits.

**Format de fichiers pris en charge** : Babel prend en charge différents formats de fichiers de localisation courants tels que Gettext (.po/.mo), JSON et XML, ce qui offre une grande flexibilité aux développeurs et aux traducteurs.

**Gestion des pluriels et des contextes** : Babel gère efficacement les cas où une chaîne de caractères nécessite des formes plurielles ou lorsqu'un contexte spécifique est nécessaire pour assurer des traductions précises.

### 2.1.2. Avantages de l'utilisation de Babel dans le processus de localisation

L'utilisation de Babel présente plusieurs avantages significatifs dans le processus de localisation des applications Python :

**Facilité d'intégration** : Babel peut être intégré de manière transparente dans divers frameworks Python tels que Flask et Django, ce qui facilite son utilisation dans des projets existants.

**Automatisation de l'extraction des messages** : Babel automatise le processus fastidieux d'extraction des messages à traduire à partir du code source. Cela permet aux développeurs de se concentrer sur l'aspect fonctionnel de l'application plutôt que de devoir gérer manuellement les chaînes de caractères à traduire.

**Collaboration avec les traducteurs** : Babel simplifie la collaboration avec les traducteurs en fournissant des fichiers .pot faciles à lire et à comprendre. Cela permet aux traducteurs de se concentrer sur leur travail sans avoir à manipuler directement le code source.

**Support multi-format** : Babel prend en charge plusieurs formats de fichiers de localisation courants, ce qui offre aux développeurs et aux traducteurs une flexibilité dans le choix du format qui convient le mieux à leur projet.

## 2.2. Configuration de Babel

Lorsque nous utilisons Babel pour la localisation de nos projets, nous devons d'abord le configurer correctement. Cette configuration comprend l'installation de Babel et la mise en place du projet pour la localisation.

### 2.2.1. Installation et mise en place du projet

Pour installer Babel, nous pouvons utiliser l'outil de gestion de paquets de Python, pip. Nous exécutons la commande suivante pour installer Babel :

```bash
pip install Babel
```

Une fois Babel installé, nous devons organiser notre projet de manière à faciliter la localisation. Il est courant de structurer le projet avec les répertoires suivants :

- **`locale/`** : Ce répertoire contiendra les fichiers de catalogues de messages (.po) pour chaque langue cible. Chaque langue aura son propre sous-répertoire avec le code de langue approprié (par exemple, `fr_FR` pour le français de France).
  
- **`templates/`** : Si nous utilisons un moteur de templates pour notre application, les fichiers de templates contenant les chaînes de caractères à traduire seront stockés ici.

- **`static/`** : Si notre application contient des fichiers statiques (images, CSS, etc.) avec des chaînes de caractères à traduire, nous pouvons les stocker dans ce répertoire.

### 2.2.2. Fichiers de configuration (babel.cfg)

Pour que Babel fonctionne correctement avec notre projet, nous devons créer un fichier de configuration `babel.cfg`. Ce fichier contient les paramètres spécifiques de Babel pour notre projet.

Voici un exemple de contenu pour `babel.cfg` :

```
[python: **.py]
[jinja2: templates/**.html]

[extractors]
# Configuration pour extraire les messages à traduire depuis les fichiers Python et les templates Jinja2
```

Dans cet exemple, nous définissons les types de fichiers à prendre en compte lors de l'extraction des messages (`.py` pour les fichiers Python et `.html` pour les templates Jinja2). Le bloc `[extractors]` peut être configuré pour prendre en charge d'autres types de fichiers si nécessaire.

## 2.3. Utilisation de Babel pour la localisation

Maintenant que nous avons correctement configuré Babel pour notre projet, voyons comment l'utiliser pour la localisation.

### 2.3.1. Extraction des chaînes de caractères à traduire

Pour extraire les chaînes de caractères à traduire à partir de notre code source et de nos templates, nous utilisons la commande `pybabel extract`. Nous spécifions le répertoire à scanner (généralement le répertoire racine de notre projet) et le fichier de configuration `babel.cfg`.

```bash
pybabel extract -F babel.cfg -o messages.pot .
```

Cette commande extrait les messages et les stocke dans le fichier `messages.pot`.

### 2.3.2. Génération des fichiers de catalogues de messages (.pot)

Une fois que nous avons extrait les messages, nous pouvons utiliser la commande `pybabel init` pour initialiser les fichiers de catalogues de messages (.po) pour chaque langue cible. Par exemple, pour initialiser les fichiers de catalogues de messages pour le français (fr_FR) :

```bash
pybabel init -i messages.pot -d locale -l fr_FR
```

Cette commande crée un fichier `fr_FR/LC_MESSAGES/messages.po` pour le français, prêt à être traduit.

### 2.3.3. Traduction des messages avec Babel

Une fois que les fichiers `.po` ont été traduits par les traducteurs, nous pouvons compiler les fichiers `.mo` utilisés par l'application avec la commande `pybabel compile` :

```bash
pybabel compile -d locale
```

Cette commande compile tous les fichiers `.po` dans le répertoire `locale` en fichiers binaires `.mo`.

Voilà ! Nous avons maintenant configuré Babel pour notre projet, extrait les messages à traduire, créé les fichiers de catalogues de messages pour chaque langue et compilé les traductions pour les utiliser dans notre application. Babel facilite grandement le processus de localisation et nous permet de proposer notre application dans différentes langues pour un public plus large.

Chapitre 3 : Introduction à pybabel

Dans ce chapitre, nous allons explorer pybabel, un outil puissant qui complète Babel pour faciliter la localisation des applications Python. Nous aborderons ce qu'est pybabel, ses fonctionnalités clés et son rôle dans le processus de localisation.

## 3.1 Qu'est-ce que pybabel ?

pybabel est un utilitaire qui s'intègre parfaitement avec Babel pour offrir des fonctionnalités supplémentaires et faciliter davantage le processus de localisation des applications Python. Il est spécifiquement conçu pour aider les développeurs et les traducteurs à gérer les fichiers de catalogues de messages (.po) générés par Babel et à effectuer des tâches de traduction plus efficacement.

### 3.1.1 Vue d'ensemble de l'outil et ses fonctionnalités

pybabel propose les fonctionnalités suivantes :

**1. Initialisation des catalogues de traduction (.po)** : Avec pybabel, nous pouvons créer de nouveaux fichiers de catalogues de messages (.po) pour chaque langue cible. Ces fichiers initialisés contiennent des traductions vides prêtes à être remplies par les traducteurs.

**2. Extraction des messages à traduire à partir du code source** : Comme Babel, pybabel permet également d'extraire les chaînes de caractères à traduire à partir du code source. Cette fonctionnalité nous permet d'actualiser les fichiers .po avec de nouvelles chaînes à traduire lorsque le code source évolue.

**3. Traduction des messages** : pybabel facilite la traduction des messages en fournissant une interface en ligne de commande pour effectuer des traductions manuelles ou pour utiliser des services de traduction automatisés.

## 3.2 Configuration de pybabel

Avant d'utiliser pybabel, nous devons le configurer correctement pour notre projet.

### 3.2.1 Installation et mise en place

Pour installer pybabel, nous pouvons utiliser l'outil de gestion de paquets de Python, pip. Nous exécutons la commande suivante pour installer pybabel :

```bash
pip install pybabel
```

Une fois pybabel installé, nous devons organiser notre projet pour la localisation. Assurez-vous que les fichiers de catalogues de messages (.po) générés par Babel sont stockés dans le répertoire approprié (généralement le répertoire `locale/`).

### 3.2.2 Configuration du projet avec pybabel

Pour que pybabel fonctionne correctement avec notre projet, nous devons créer un fichier de configuration `babel.cfg`. Ce fichier contient les paramètres spécifiques de pybabel pour notre projet.

Voici un exemple de contenu pour `babel.cfg` :

```
[python: **.py]
[jinja2: templates/**.html]

[extractors]
# Configuration pour extraire les messages à traduire depuis les fichiers Python et les templates Jinja2
```

Comme avec Babel, nous définissons les types de fichiers à prendre en compte lors de l'extraction des messages. Veillez à utiliser les mêmes paramètres de configuration pour `babel.cfg` dans Babel et pybabel pour assurer une cohérence dans le processus de localisation.

## 3.3 Principales fonctionnalités de pybabel

Maintenant que nous avons configuré pybabel pour notre projet, voyons de plus près ses fonctionnalités clés.

### 3.3.1 Initialisation des catalogues de traduction (.po)

Pour créer un nouveau fichier de catalogue de messages pour une langue cible spécifique, nous utilisons la commande `pybabel init`. Par exemple, pour initialiser les fichiers .po pour le français (fr_FR) :

```bash
pybabel init -i messages.pot -d locale -l fr_FR
```

Cette commande créera un fichier `fr_FR/LC_MESSAGES/messages.po` prêt à être traduit.

### 3.3.2 Extraction des messages à traduire à partir du code source

Comme mentionné précédemment, nous pouvons extraire les chaînes de caractères à traduire à partir du code source en utilisant la commande `pybabel extract`.

```bash
pybabel extract -F babel.cfg -o messages.pot .
```

Cette commande extrait les messages et les stocke dans le fichier `messages.pot`.

### 3.3.3 Traduction des messages avec pybabel

Une fois que les traducteurs ont rempli les fichiers .po avec les traductions, nous pouvons compiler les fichiers `.po` en fichiers binaires `.mo` avec la commande `pybabel compile` :

```bash
pybabel compile -d locale
```

Cette commande compile tous les fichiers `.po` dans le répertoire `locale`, ce qui permet à l'application de récupérer les messages traduits.

Grâce à pybabel, nous pouvons gérer les fichiers de catalogues de messages (.po) générés par Babel, faciliter le travail des traducteurs et obtenir des traductions plus rapidement. Dans le prochain chapitre, nous aborderons plus en détail l'intégration de Babel et pybabel dans un projet Flask (ou autre framework) pour la localisation de nos applications Python.

# 4 Intégration de Babel et pybabel dans un projet Pyramid utilisant chameleon et des templates tal/metal

Dans ce chapitre, nous allons aborder l'intégration de Babel et pybabel dans un projet Pyramid, en utilisant les moteurs de templates chameleon et tal/metal pour faciliter la localisation de notre application Python.

## 4.1. Utilisation de Babel avec Pyramid

### 4.1.1. Configuration de Pyramid pour la localisation

Avant d'utiliser Babel et pybabel dans notre projet Pyramid, nous devons d'abord configurer Pyramid pour prendre en charge la localisation. Pour cela, nous devons ajouter les packages nécessaires à notre projet.

Tout d'abord, nous devons installer le package `pyramid-babel` qui permet l'intégration de Babel avec Pyramid :

```bash
pip install pyramid-babel
```

Ensuite, nous devons ajouter `pyramid_babel` à la liste des `pyramid.includes` dans notre fichier de configuration `__init__.py` :

```python
# __init__.py

from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_babel')  # Inclusion de pyramid_babel dans notre projet
    # ... autres configurations ...
    return config.make_wsgi_app()
```

### 4.1.2. Utilisation des fonctions de traduction dans les modèles et les vues

Une fois que nous avons configuré Pyramid pour la localisation, nous pouvons utiliser les fonctions de traduction fournies par Babel dans nos modèles et nos vues.

Dans nos templates chameleon et tal/metal, nous pouvons utiliser la fonction `trans` pour marquer les chaînes de caractères à traduire :

```html
<!-- template.pt -->

<p>${request.localizer.translate('Hello, world!')}</p>
```

Dans nos vues, nous pouvons utiliser la fonction `get_localizer` pour obtenir l'objet de traduction et traduire les messages dans le code :

```python
# views.py

from pyramid.view import view_config

@view_config(route_name='home', renderer='templates/template.pt')
def home(request):
    localizer = request.localizer
    message = localizer.translate('Hello, world!')
    return {'message': message}
```

## 4.2. Exemples pratiques d'utilisation de Babel et pybabel dans Pyramid

### 4.2.1. Mise en place de la localisation dans une application Pyramid

Pour mettre en place la localisation dans notre application Pyramid, nous devons suivre les étapes suivantes :

1. **Configuration de Babel** : Nous devons configurer Babel pour notre projet comme nous l'avons vu précédemment. Cela comprend la création du fichier `babel.cfg` avec les paramètres appropriés et l'installation de Babel dans notre environnement.

2. **Extraction des messages** : Nous utilisons la commande `pybabel extract` pour extraire les chaînes de caractères à traduire à partir de notre code source et de nos templates chameleon/tal/metal.

3. **Initialisation des catalogues de traduction** : Avec la commande `pybabel init`, nous créons les fichiers .po pour chaque langue cible (par exemple, `fr_FR`, `es_ES`, etc.) dans le répertoire `locale/`.

4. **Traduction des messages** : Les traducteurs remplissent les fichiers .po avec les traductions des messages.

5. **Compilation des traductions** : Nous utilisons `pybabel compile` pour compiler les fichiers .po en fichiers binaires .mo, prêts à être utilisés par notre application.

### 4.2.2. Traduction des messages avec pybabel

Une fois que les traductions sont effectuées et que les fichiers .mo sont compilés, Pyramid utilisera automatiquement les traductions appropriées en fonction de la langue préférée de l'utilisateur, des paramètres régionaux, ou de tout autre mécanisme de sélection de langue que nous aurions configuré.

Ainsi, lorsque l'utilisateur navigue sur notre application, les messages traduits s'afficheront selon sa langue préférée, offrant ainsi une expérience utilisateur personnalisée et multilingue.

L'intégration de Babel et pybabel dans un projet Pyramid utilisant les moteurs de templates chameleon et tal/metal rend la localisation de nos applications Python plus accessible et conviviale pour les traducteurs et les développeurs. Cela permet d'offrir une expérience utilisateur optimale pour les utilisateurs du monde entier.

# 5. Bonnes pratiques en localisation

En tant qu'équipe de développement travaillant sur des projets multilingues, nous sommes confrontés à des défis spécifiques lors de la localisation. Dans ce chapitre, nous allons aborder les bonnes pratiques à suivre pour garantir une localisation réussie et des traductions de qualité.

## 5.1. Sélection des messages à traduire

Lors de la localisation d'une application, il est essentiel de sélectionner judicieusement les messages à traduire. Tous les textes n'ont pas besoin d'être traduits ; nous devons nous concentrer sur les éléments qui seront visibles par les utilisateurs.

Les messages tels que les titres, les libellés de boutons, les messages d'erreur et les textes d'aide sont des éléments importants à traduire. En revanche, les noms de variables, les commentaires internes du code et les messages de débogage ne nécessitent généralement pas de traduction.

Une approche courante pour sélectionner les messages à traduire consiste à les marquer à l'aide de fonctions de traduction telles que `trans` en utilisant Babel. Cela facilitera l'extraction des messages à traduire à partir du code source.

## 5.2. Gestion des pluriels et des contextes linguistiques

Dans certaines langues, les mots peuvent avoir différentes formes selon le contexte ou le nombre. La gestion correcte des pluriels et des contextes linguistiques est cruciale pour des traductions précises.

Babel prend en charge la gestion des pluriels dans les messages en utilisant des fonctions spéciales telles que `ngettext` et `npgettext`. L'utilisation appropriée de ces fonctions nous permettra de gérer les différentes formes grammaticales liées au nombre.

De même, certains mots peuvent avoir des significations différentes selon le contexte. Il est donc important de fournir suffisamment de contexte lors de l'extraction des messages à traduire afin que les traducteurs comprennent le sens exact des phrases à traduire.

## 5.3. Astuces pour des traductions de qualité

Pour obtenir des traductions de qualité, il est essentiel de travailler en étroite collaboration avec des traducteurs qualifiés et natifs de la langue cible. Les traducteurs doivent comprendre le contexte de l'application et les nuances linguistiques pour fournir des traductions précises et naturelles.

Nous devons également éviter d'utiliser des phrases ou des expressions trop complexes ou idiomatiques qui pourraient être difficiles à traduire fidèlement. Une formulation claire et simple est généralement préférable.

Enfin, il est important de maintenir une terminologie cohérente tout au long de l'application. Utiliser les mêmes termes pour désigner des concepts similaires permettra aux utilisateurs de mieux comprendre l'interface.

## 5.4. Tests et validation des traductions

Une fois les traductions effectuées, il est essentiel de les tester et de les valider avant de les déployer dans l'application. Nous devons nous assurer que les traductions s'intègrent correctement dans l'interface et qu'elles respectent les contraintes d'espace.

La mise en place de tests automatisés pour les traductions peut être utile pour détecter rapidement d'éventuelles erreurs ou problèmes de formatage.

Enfin, il est recommandé de solliciter les commentaires des utilisateurs natifs de la langue cible pour obtenir des retours sur la qualité des traductions et apporter des améliorations si nécessaire.

En suivant ces bonnes pratiques en localisation, nous pouvons garantir une expérience utilisateur positive pour nos utilisateurs du monde entier et atteindre notre objectif de proposer des applications multilingues de haute qualité.

# 6. Présentation de Gettext

Dans ce chapitre, nous allons vous présenter Gettext, un outil puissant et largement utilisé pour la localisation des applications dans le monde du développement logiciel. Nous explorerons ce qu'est Gettext, ses fonctionnalités clés, ainsi que les avantages qu'il offre dans le processus de localisation.

## 6.1. Qu'est-ce que gettext ?

Gettext est un système de gestion de la localisation qui fournit un mécanisme pour traduire des messages dans des applications logicielles. Il est largement utilisé dans de nombreux projets open source et commerciaux pour permettre une localisation facile et efficace.

### 6.1.1. Vue d'ensemble de l'outil

Gettext offre une approche basée sur des fichiers de catalogues de messages pour gérer les traductions. Ces fichiers sont généralement au format `.po` (Portable Object) pour les messages à traduire et au format `.mo` (Machine Object) pour les messages traduits compilés. Les fichiers `.po` sont des fichiers texte lisibles par l'homme, tandis que les fichiers `.mo` sont des fichiers binaires qui sont utilisés par l'application pour afficher les messages traduits.

Un fichier de catalogue de messages contient des paires de chaînes de caractères d'origine et de traduction. Ces paires sont associées par des identifiants uniques appelés « clés de message ». L'outil Gettext utilise ces clés pour rechercher les traductions appropriées lorsque l'application est exécutée.

### 6.1.2. Avantages de l'utilisation de gettext dans le processus de localisation

L'utilisation de Gettext présente plusieurs avantages significatifs dans le processus de localisation des applications :

**1. Simplicité d'utilisation** : Gettext offre une approche simple et bien structurée pour gérer les traductions. Les fichiers de catalogues de messages sont faciles à lire et à comprendre, ce qui facilite la collaboration entre les développeurs et les traducteurs.

**2. Prise en charge des pluriels et des contextes** : Gettext offre un mécanisme robuste pour gérer les cas où une chaîne de caractères nécessite différentes formes grammaticales pour les pluriels ou lorsqu'un contexte spécifique est nécessaire pour les traductions.

**3. Flexibilité de formats de fichiers** : Gettext prend en charge différents formats de fichiers, tels que les formats .po et .mo, ce qui offre aux développeurs et aux traducteurs la flexibilité de travailler avec les formats qui conviennent le mieux à leurs outils et workflows.

**4. Large adoption** : Gettext est un outil bien établi et largement adopté dans la communauté du développement logiciel. Il est pris en charge par de nombreux langages de programmation et intégré dans de nombreux frameworks et outils, ce qui en fait un choix populaire pour la localisation.

En conclusion, Gettext est un outil puissant et éprouvé pour gérer la localisation des applications. Son approche basée sur des fichiers de catalogues de messages, sa prise en charge des pluriels et des contextes, ainsi que sa simplicité d'utilisation en font un choix idéal pour les projets multilingues. Dans les prochains chapitres, nous explorerons plus en détail l'utilisation pratique de Gettext pour la localisation de nos applications.

## ​￼6.2. Configuration de gettext

### 6.2.1. Installation et mise en place du projet

Pour utiliser Gettext dans notre projet de localisation, nous devons d'abord nous assurer que le package Gettext est installé dans notre environnement Python. Nous pouvons l'installer en utilisant pip :

```bash
pip install gettext
```

Une fois Gettext installé, nous devons structurer notre projet de manière à faciliter la localisation. Habituellement, nous organisons notre projet comme suit :

- **`locale/`** : Ce répertoire contiendra les fichiers de catalogues de messages (.po) pour chaque langue cible. Chaque langue aura son propre sous-répertoire avec le code de langue approprié (par exemple, `fr_FR` pour le français de France).
  
- **`templates/`** : Si nous utilisons des fichiers de templates pour notre application, les fichiers contenant les chaînes de caractères à traduire seront stockés ici.

- **`static/`** : Si notre application contient des fichiers statiques (images, CSS, etc.) avec des chaînes de caractères à traduire, nous pouvons les stocker dans ce répertoire.

### 6.2.2. Fichiers de configuration

Gettext ne nécessite pas de fichier de configuration spécifique. Cependant, nous devons utiliser les fonctions de Gettext appropriées dans notre code pour marquer les chaînes de caractères à traduire et les traductions.

## 6.3. Utilisation de gettext pour la localisation

Une fois notre projet configuré, nous pouvons commencer à utiliser Gettext pour la localisation.

### 6.3.1. Extraction des chaînes de caractères à traduire

Pour extraire les chaînes de caractères à traduire à partir de notre code source et de nos templates, nous utilisons l'outil `xgettext` fourni avec Gettext. Par exemple, pour extraire les messages à traduire à partir de fichiers Python et les stocker dans un fichier `messages.pot`, nous exécutons la commande suivante :

```bash
xgettext --output=messages.pot *.py
```

### 6.3.2. Génération des fichiers de catalogues de messages (.pot)

Une fois que nous avons extrait les messages à traduire, nous pouvons utiliser l'outil `msginit` pour initialiser les fichiers de catalogues de messages (.po) pour chaque langue cible. Par exemple, pour initialiser les fichiers de catalogues de messages pour le français (fr_FR) :

```bash
msginit --locale=fr_FR --input=messages.pot
```

Cette commande crée un fichier `fr_FR.po` avec les traductions vides prêtes à être remplies par les traducteurs.

### 6.3.3. Traduction des messages avec gettext

Une fois que les traducteurs ont rempli les fichiers .po avec les traductions, nous pouvons les compiler en fichiers binaires .mo avec l'outil `msgfmt` :

```bash
msgfmt --output-file=fr_FR/LC_MESSAGES/messages.mo fr_FR.po
```

Cette commande compile le fichier `fr_FR.po` en un fichier `messages.mo` utilisable par l'application pour afficher les messages traduits.

@TODO fusionner

# Comment fonctionne gettext ?

Gettext est un outil de traduction de logiciels open source qui permet de créer et de maintenir des fichiers de traduction pour différentes langues. Il est largement utilisé dans les projets de logiciels libres et est disponible sous de nombreuses plateformes (Linux, Unix, Windows, etc.).

Voici comment Gettext fonctionne en gros :

1.  Lorsque nous développons une application, nous pouvons marquer les chaînes de caractères à traduire dans votre code source à l'aide de la fonction `gettext`. Par exemple, en Python :

```python
from gettext import gettext as _  print(_("Hello, world!"))
```

2.  Une fois que nous avons marqué toutes les chaînes de caractères à traduire dans votre code, nous pouvons utiliser l'outil `xgettext` pour extraire ces chaînes et créer un fichier `.pot` (fichier de catalogue de traduction) qui les contient. Le fichier `.pot` est un fichier de texte qui contient les chaînes de caractères à traduire et des informations sur le projet et les personnes responsables de la traduction.
    
3.  nous pouvons utiliser un outil de traduction, comme Poedit, pour ouvrir le fichier `.pot` et traduire les chaînes de caractères. Lorsque nous enregistrons les modifications, Poedit créera un fichier `.po` (fichier de catalogue de traduction) pour chaque langue traduite. Le fichier `.po` contiendra les chaînes de caractères à traduire et leur traduction dans la langue cible.
    
5.  Une fois que nous avons créé tous les fichiers `.po` de traduction, nous pouvons utiliser l'outil `msgfmt` pour générer les fichiers `.mo` (fichiers de message binaires) à partir de ceux-ci. Les fichiers `.mo` sont des fichiers binaires qui contiennent les mêmes informations que les fichiers `.po`, mais dans un format plus compact et plus rapide à charger par l'application.
    
6.  nous pouvons maintenant utiliser la fonction `gettext` dans votre code pour charger les fichiers de traduction en fonction de la langue de l'utilisateur. Par exemple, en Python :
    
```python
import gettext  gettext.install("messages", "locale")
```

Cette commande va charger les fichiers de traduction du projet (nommés "messages") à partir du répertoire "locale", en fonction de la langue de l'utilisateur.

## Installation
Pour générer les fichiers .pot (fichiers de catalogue de traduction) à partir des fichiers source Python d'un projet, on peut utiliser la commande `xgettext` de GNU Gettext. Voici comment procéder :

1.  Assurons-nous d'avoir installé Gettext sur votre ordinateur. Si ce n'est pas le cas, nous pouvons l'installer à l'aide de la commande suivante (sous Debian ou Ubuntu) :

```bash
sudo apt-get install gettext
```

2.  Plaçons-nous dans le répertoire racine du projet (celui où se trouve le fichier `setup.py`).
    
3.  Exécutons la commande suivante :
    

```bash
xgettext --language=Python --output=locale/messages.pot `find . -name "*.py"`
```


Cette commande va parcourir tous les fichiers `.py` du projet et générer un fichier `messages.pot` dans le répertoire `locale`. Ce fichier contiendra les chaînes de caractères à traduire dans tous les fichiers source.

Nous pouvons également utiliser des outils tels que `pygettext` ou `babel` pour générer les fichiers `.pot` de votre projet. Ces outils sont spécialement conçus pour les projets Python et offrent des fonctionnalités supplémentaires, telles que la détection automatique des chaînes de caractères à traduire ou la génération de fichiers `.pot` à partir de templates de templates Django.

## Comment fait-on avec babel ?

Pour utiliser Babel pour générer les fichiers `.pot` de votre projet Python, nous devons d'abord installer l'outil en exécutant la commande suivante :

`pip install babel`

Une fois installé, plaçons-nous dans le répertoire racine de votre projet et exécutons la commande suivante :

`pybabel extract -F babel.cfg -o locale/messages.pot .`

Cette commande va parcourir tous les fichiers du projet et générer un fichier `messages.pot` dans le répertoire `locale`. Le fichier de configuration `babel.cfg` contient les options de configuration de Babel. nous pouvons en créer un en exécutant la commande `pybabel init`.

Pour plus d'informations sur l'utilisation de Babel pour la traduction de projets Python, je nous recommande de consulter la documentation officielle de Babel : [https://babel.readthedocs.io/en/latest/](https://babel.readthedocs.io/en/latest/).

Les options `-i` et `-d` de la commande `pybabel init` peuvent contenir des chemins absolus ou relatifs vers les fichiers et répertoires correspondants.

Par exemple, si nous souhaitons créer le fichier de configuration `babel.cfg` à partir d'un fichier `messages.pot` situé dans le répertoire `locale`, nous pouvons utiliser la commande suivante :

```bash
pybabel init -i locale/messages.pot -d locale -l fr
```

N'oublions pas de remplacer `fr` par la langue de base de votre projet. nous pouvons également utiliser l'option `-D` pour définir des variables de configuration dans le fichier de configuration généré, comme je nous l'ai indiqué précédemment.

## Le fichier \*.pot

Un fichier `.pot` (fichier de catalogue de traduction) contient principalement deux parties : l'entête et les entrées de traduction. L'entête du fichier est un bloc de commentaires qui se trouve en haut du fichier et qui fournit des informations sur le projet, la version du fichier `.pot` et les personnes responsables de la traduction.

Voici ce que doit contenir l'entête d'un fichier `.pot` selon la spécification de Gettext :

```pot
# SOME DESCRIPTIVE TITLE.
# Copyright (C) YEAR THE PACKAGE'S COPYRIGHT HOLDER
# This file is distributed under the same license as the PACKAGE package.
# FIRST AUTHOR <EMAIL@ADDRESS>, YEAR.
#
#, fuzzy
msgid ""
msgstr ""
"Project-Id-Version: PACKAGE VERSION\n"
"Report-Msgid-Bugs-To: EMAIL@ADDRESS\n"
"POT-Creation-Date: YEAR-MO-DA HO:MI+ZONE\n"
"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
"Language-Team: LANGUAGE <LL@li.org>\n"
"Language: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=CHARSET\n"
"Content-Transfer-Encoding: 8bit\n"
```

Explication des différents champs de l'entête :

-   `SOME DESCRIPTIVE TITLE` : titre descriptif du projet.
-   `YEAR THE PACKAGE'S COPYRIGHT HOLDER` : année de copyright du paquet.
-   `FIRST AUTHOR <EMAIL@ADDRESS>` : nom et adresse e-mail de la première personne à avoir traduit le fichier.
-   `PACKAGE VERSION` : version du paquet.
-   `EMAIL@ADDRESS` : adresse e-mail à laquelle envoyer les rapports d'erreurs.
-   `YEAR-MO-DA HO:MI+ZONE` : date de création et de modification du fichier `.pot`.
-   `FULL NAME <EMAIL@ADDRESS>` : nom et adresse e-mail de la dernière personne à avoir traduit le fichier.
-   `LANGUAGE` : langue de traduction.
-   `CHARSET` : jeu de caractères utilisé dans le fichier.

Il est important de remplir correctement ces champs afin de fournir des informations précises sur le projet et sur les personnes qui ont contribué à la traduction. Nous pouvons également ajouter d'autres champs à l'entête si nous le souhaitons, mais assurons-nous de respecter la syntaxe de Gettext.

## Signification du sous répertoire LC_MESSAGES qui est dans "locale"

Le répertoire `LC_MESSAGES` se trouve généralement à l'intérieur du répertoire `locale` d'un projet et contient les fichiers de traduction de ce projet. Le répertoire `LC_MESSAGES` est utilisé par les outils de traduction, comme `gettext`, pour trouver les fichiers de traduction correspondants à la langue de l'utilisateur.

La nomenclature du répertoire `LC_MESSAGES` est la suivante :

```bash
locale/
   fr/
        LC_MESSAGES/
           messages.po
           messages.mo
    en/
        LC_MESSAGES/
           messages.po
           messages.mo
```

Dans cet exemple, les fichiers `messages.po` et `messages.mo` contiennent la traduction du projet en français et en anglais, respectivement. Le répertoire `fr` correspond à la langue française et le répertoire `en` à la langue anglaise.

Le répertoire `LC_MESSAGES` doit être nommé ainsi pour être reconnu par les outils de traduction. Si nous utilisons un autre nom, il se peut que les fichiers de traduction ne soient pas chargés correctement par l'application.

## Différences ente *.mo et *.po

Les fichiers `.mo` et `.po` sont utilisés par les outils de traduction, comme `gettext`, pour stocker les traductions d'un projet.

-   Les fichiers `.po` (fichiers de catalogue de traduction) sont des fichiers de texte qui contiennent les chaînes de caractères à traduire du projet, ainsi que leur traduction dans la langue cible. Ils sont générés à partir des fichiers source du projet (par exemple, des fichiers `.py` pour un projet Python) à l'aide d'outils tels que `xgettext` ou `pybabel extract`.
    
-   Les fichiers `.mo` (fichiers de message binaires) sont des fichiers binaires qui contiennent les mêmes informations que les fichiers `.po`, mais dans un format plus compact et plus rapide à charger par l'application. Ils sont générés à partir des fichiers `.po` à l'aide de l'outil `msgfmt`.

Lorsque nous traduisons un projet, nous travaillons généralement avec des fichiers `.po`, qui sont plus faciles à lire et à modifier. Une fois que nous avons terminé la traduction, nous pouvons générer les fichiers `.mo` à partir des fichiers `.po` pour que l'application puisse les charger plus rapidement.

Il est important de ne pas modifier directement les fichiers `.mo`, car ils sont au format binaire et ne sont pas lisibles par les humains. Si nous souhaitons apporter des modifications à la traduction, nous devons le faire dans les fichiers `.po` correspondants et regénérer les fichiers `.mo` à partir de ceux-ci.

# Création des fichiers po

Pour créer de nouveaux fichiers `.po` à partir d'un fichiers `.pot` (Portable Object Template), nous pouvons utiliser la commande `msginit` du package gettext. Voici comment :

1. Ouvrons un terminal et naviguons jusqu'au répertoire où se trouve votre fichier `.pot`.

2. Utilisons la commande suivante pour générer un fichier `.po` :

    ```bash
    msginit -i mydomain.pot -o mylanguage.po -l mylanguage
    ```

   Ici, `mydomain.pot` est notre fichier `.pot` source, `mylanguage.po` est le fichier `.po` que nous voulons créer et `mylanguage` est le code de langue pour lequel nous voulons créer le fichier (par exemple `fr` pour le français, `de` pour l'allemand, etc.).

Par exemple, pour générer un fichier `.po` pour le français à partir d'un fichier `alirpunkto.pot`, nous utiliserons :

```bash
cd alirpunkto/locale
msginit -i alirpunkto.pot -o en/LC_MESSAGES/alirpunkto.po -l en
msginit -i alirpunkto.pot -o fr/LC_MESSAGES/alirpunkto.po -l fr
```

Cette commande créera ou **écrasera** le fichier `fr.po` à partir du fichier `mydomain.pot`. Nous pouvons alors éditer ce fichier `fr.po` pour traduire les chaînes de caractères en français. Une fois les traductions terminées, nous pouvons générer un fichier `.mo`.

# Mettre à jour des fichiers pot et po
Si nous avons des chaînes nouvellement ajoutées dans notre code source et que nous souhaitons mettre à jour notre fichier `.pot` existant sans l'écraser, nous pouvons utiliser l'outil `xgettext` pour extraire les nouvelles chaînes, puis `msgmerge` pour fusionner ces nouvelles chaînes avec notre fichier `.pot` existant.

Voici comment le faire :

1. Utilisons `xgettext` pour extraire les nouvelles chaînes de notre code source dans un nouveau fichier `.pot`. Par exemple :

    ```bash
    xgettext -o new.pot source.py
    ```

    Ici, `new.pot` est le fichier de sortie que `xgettext` générera et `source.py` est notre code source Python. Nous devrions remplacer ces noms par ceux de notre projet. Nous pouvons également spécifier plusieurs fichiers source à la fois.

2. Ensuite, utilisons `msgmerge` pour fusionner le nouveau fichier `.pot` avec l'ancien. Par exemple :

    ```bash
    msgmerge -U old.pot new.pot
    ```

    Ici, `old.pot` est notre ancien fichier `.pot` et `new.pot` est le fichier que nous venons de générer avec `xgettext`. La commande `msgmerge` mettra à jour `old.pot` pour y inclure les nouvelles chaînes de `new.pot`.

Les anciennes traductions resteront dans `old.pot` après cette opération. Les nouvelles chaînes extraites seront ajoutées sans traductions, prêtes à être traduites.

Remarque : pour que `xgettext` fonctionne correctement, nous devons utiliser les méthodes standard de marquage des chaînes localisables dans votre code, comme `_()` ou `gettext()`.

# Génération des fichiers mo

Pour générer les fichiers `.mo` (Machine Object) à partir des fichiers `.po` (Portable Object) dans notre répertoire `locale`, nous pouvons utiliser la commande `msgfmt` du package gettext. Voici comment nous pouvons faire :

1. Ouvrons un terminal et naviguons jusqu'au répertoire `locale` de notre projet.

2. Utilisons la commande suivante pour générer le fichier `.mo` :

    ```bash
    msgfmt -o LC_MESSAGES/mydomain.mo LC_MESSAGES/mydomain.po
    ```

    Dans cette commande, remplaçons `mydomain` par le nom de domaine de notre application.

Notez que vous devez répéter ce processus pour chaque fichier `.po` dans votre projet. 

Si vous avez de nombreux fichiers `.po`, vous pouvez utiliser une boucle pour les parcourir tous. Par exemple, si vous êtes sous un système Unix-like, vous pouvez utiliser une commande `find` couplée à une boucle `for` :

```bash
find . -name "*.po" -exec msgfmt {} -o {}.mo \;
```

Cette commande trouve tous les fichiers `.po` dans le répertoire courant et ses sous-répertoires, et exécute `msgfmt` sur chaque fichier pour générer le fichier `.mo` correspondant.