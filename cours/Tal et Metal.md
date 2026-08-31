---
schema_version: 1
uid: "01M02EX5CAH73032AGP7214PKZ"
titre: "TAL, TALES et METAL"
aliases:
  - "TAL et METAL"
  - "TAL (Template Attribute Language)"
  - "TALES"
  - "METAL"
  - "Zope Page Templates"
  - "Page Templates"
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
  - pyramid
  - templating
  - chameleon
resume: "Cours complet sur Zope Page Templates : TAL pour la logique de présentation, TALES pour les expressions, METAL pour les macros et les slots, avec les différences entre Zope et Chameleon, l'échappement HTML, l'i18n, Pyramid et les bonnes pratiques d'architecture."
niveau: intermediaire
prerequis:
  - "[[HTML]]"
  - "[[Python]]"
  - "[[Pyramid]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-06-15
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Refonte
> Cours développé le 31 août 2026 (d'environ 330 à 6 685 mots  remplacement complet du texte précédent)  vérifié le même jour : schéma  titres  liens et affirmations datées contrôlés. La version précédente reste dans l'historique git du dépôt.

# TAL, TALES et METAL

> [!abstract] Objectif
> Comprendre le modèle **Zope Page Templates** et savoir écrire des gabarits HTML lisibles, sûrs et réutilisables avec **TAL**, **TALES** et **METAL**. Le but n'est pas seulement de mémoriser des attributs, mais de comprendre où placer la logique, comment fonctionne la portée des variables, comment éviter les failles XSS et comment organiser des layouts et composants réutilisables.

Voir aussi : [[HTML]], [[Python]], [[Pyramid]], [[Deform]], [[Internationalisation]].

# 1. Pourquoi ces langages existent-ils ?

Un moteur de templates transforme un document contenant à la fois du HTML statique et des expressions dynamiques en une réponse HTML finale.

Une application web classique sépare idéalement :

```text
modèle / services
       │
       ▼
contrôleur / vue Python
       │
       ▼
contexte de rendu
       │
       ▼
template
       │
       ▼
HTML envoyé au navigateur
```

TAL et METAL ont été conçus pour que le template reste **très proche d'un document HTML valide**.

Cette propriété est importante :

- le designer peut lire le document ;
- le développeur voit immédiatement la structure finale ;
- une grande partie du contenu statique reste exploitable sans exécuter l'application ;
- la logique de présentation est portée par des attributs plutôt que par de gros blocs de code intrusifs.

Exemple :

```html
<h1 tal:content="title">Titre de démonstration</h1>
```

Sans moteur de templates, le HTML reste compréhensible :

```html
<h1>Titre de démonstration</h1>
```

Au rendu, le contenu est remplacé par la valeur de `title`.

# 2. Les quatre briques à distinguer

L'écosystème Page Templates est composé de plusieurs langages complémentaires.

| Brique | Rôle |
|---|---|
| **TAL** | Modifier la structure et le contenu du document |
| **TALES** | Évaluer les expressions utilisées par TAL et METAL |
| **METAL** | Définir et réutiliser des macros et des zones personnalisables |
| **I18N** | Marquer les textes et attributs traduisibles |

On peut résumer ainsi :

```text
TALES = calculer une valeur
TAL   = utiliser cette valeur dans le HTML
METAL = réutiliser une structure de template
I18N  = traduire la présentation
```

> [!important]
> TAL et METAL ne sont pas des moteurs de templates indépendants. Ce sont des langages utilisés par des implémentations de **Page Templates**, notamment Zope Page Templates et Chameleon.

# 3. État de l'écosystème en 2026

TAL/METAL ne sont pas des technologies nouvelles, mais elles restent utilisées dans l'écosystème Zope/Plone et dans des applications Python utilisant Chameleon.

La documentation Zope 6 conserve les chapitres consacrés aux Zope Page Templates et la référence TAL/TALES/METAL.

**Chameleon** est une implémentation Python moderne et compilée du langage Page Templates. Sa branche 4.x fonctionne avec Python 3 moderne et reste utilisée notamment avec Pyramid via `pyramid_chameleon`.

Il faut donc distinguer deux contextes fréquents :

```text
Zope / ZPT classique
        │
        ├── TAL
        ├── TALES avec path comme expression par défaut
        └── METAL

Chameleon
        │
        ├── TAL
        ├── TALES avec Python comme expression par défaut
        ├── extensions supplémentaires
        └── METAL
```

Cette différence sur l'expression par défaut est fondamentale.

# 4. Un premier template minimal

Exemple Chameleon :

```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="utf-8">
    <title tal:content="title">Titre de la page</title>
  </head>
  <body>
    <h1 tal:content="title">Titre de la page</h1>

    <p tal:condition="user">
      Bonjour <strong tal:content="user.name">Alice</strong>.
    </p>
  </body>
</html>
```

Le texte statique joue ici deux rôles :

1. il documente le template ;
2. il constitue un contenu de démonstration quand le fichier est lu sans rendu.

# 5. Espaces de noms TAL et METAL

Dans un document XML/XHTML strict, les espaces de noms peuvent être déclarés explicitement :

```html
<html
  xmlns:tal="http://xml.zope.org/namespaces/tal"
  xmlns:metal="http://xml.zope.org/namespaces/metal"
  xmlns:i18n="http://xml.zope.org/namespaces/i18n">
```

Ces URI sont des **identifiants d'espaces de noms**. Ce ne sont pas des pages web que le navigateur doit télécharger.

Dans de nombreux templates HTML traités par Chameleon ou Zope, on utilise directement :

```html
<div tal:condition="show_panel">
  ...
</div>
```

# 6. TAL : Template Attribute Language

TAL modifie un arbre HTML/XML à l'aide d'attributs spéciaux.

Les instructions les plus importantes sont :

| Instruction | Fonction |
|---|---|
| `tal:define` | définir une ou plusieurs variables |
| `tal:condition` | afficher conditionnellement un élément |
| `tal:repeat` | répéter un élément |
| `tal:content` | remplacer le contenu d'un élément |
| `tal:replace` | remplacer l'élément complet |
| `tal:attributes` | calculer des attributs HTML |
| `tal:omit-tag` | retirer la balise mais conserver ses enfants |
| `tal:on-error` | définir un rendu de secours en cas d'erreur |
| `tal:switch` / `tal:case` | sélection multiple, dans les implémentations qui la proposent |

# 7. `tal:content` : remplacer le contenu

Exemple :

```html
<h1 tal:content="title">Titre d'exemple</h1>
```

Avec :

```python
{"title": "Tableau de bord"}
```

le résultat devient :

```html
<h1>Tableau de bord</h1>
```

La balise `<h1>` est conservée.

## 7.1 Échappement automatique

Par défaut, le contenu est traité comme du **texte**.

Si :

```python
title = "<script>alert('XSS')</script>"
```

alors :

```html
<h1 tal:content="title">...</h1>
```

ne doit pas injecter le script comme balisage actif. Les caractères HTML sensibles sont échappés.

C'est un mécanisme de sécurité essentiel.

> [!warning]
> Ne désactivez jamais l'échappement automatique pour afficher directement une chaîne fournie par un utilisateur.

## 7.2 Insérer volontairement une structure HTML

TAL historique permet :

```html
<div tal:content="structure html_fragment">...</div>
```

Chameleon propose aussi le type d'expression :

```html
<div tal:content="structure: html_fragment">...</div>
```

`structure` signifie : **le résultat est déjà du balisage sûr**.

Cette décision doit être prise uniquement si la donnée a été :

- générée par l'application ;
- assainie par une bibliothèque adaptée ;
- ou explicitement considérée comme sûre.

Mauvais exemple :

```html
<div tal:content="structure: request.params['comment']"></div>
```

Cela peut produire une XSS persistante ou réfléchie.

# 8. `tal:replace` : remplacer l'élément complet

Exemple :

```html
<span tal:replace="user.name">Nom</span>
```

Résultat :

```text
Alice
```

La balise `<span>` disparaît.

Comparer :

```html
<span tal:content="user.name">Nom</span>
```

Résultat :

```html
<span>Alice</span>
```

Résumé :

```text
tal:content  → garde l'élément

tal:replace  → remplace l'élément lui-même
```

# 9. `tal:condition`

`tal:condition` conserve ou retire l'élément selon la valeur d'une expression.

```html
<p tal:condition="user.is_authenticated">
  Vous êtes connecté.
</p>
```

Autre exemple :

```html
<div tal:condition="items">
  La collection n'est pas vide.
</div>
```

En Python et dans les implémentations modernes, sont généralement faux :

- `False` ;
- `None` ;
- `0` ;
- `""` ;
- `[]` ;
- `{}` ;
- autres objets évalués comme faux.

## 9.1 Préférer une condition simple

Préférer :

```html
<p tal:condition="user.is_admin">Administration</p>
```

à :

```html
<p tal:condition="user.role == 'admin' and user.enabled and not user.suspended and feature_flags.get('admin_ui')">
  Administration
</p>
```

La deuxième expression contient une règle métier qui devrait généralement être préparée côté Python :

```python
can_show_admin = authorization.can_show_admin(user)
```

puis :

```html
<p tal:condition="can_show_admin">Administration</p>
```

# 10. `tal:repeat`

`tal:repeat` répète l'élément pour chaque valeur d'un itérable.

```html
<ul>
  <li tal:repeat="item items" tal:content="item.name">
    Élément
  </li>
</ul>
```

Avec :

```python
items = [
    Item(name="Alpha"),
    Item(name="Beta"),
]
```

on obtient :

```html
<ul>
  <li>Alpha</li>
  <li>Beta</li>
</ul>
```

# 11. L'objet `repeat`

Une répétition expose des métadonnées via :

```text
repeat.<nom_de_variable>
```

Par exemple :

```html
<li tal:repeat="item items">
  <span tal:content="repeat.item.number">1</span>
  <span tal:content="item.name">Nom</span>
</li>
```

Selon l'implémentation, l'objet de répétition fournit notamment des informations comme :

- index ;
- numéro humain commençant à 1 ;
- premier élément ;
- dernier élément ;
- longueur ;
- parité.

Exemple typique :

```html
<tr tal:repeat="row rows"
    tal:attributes="class 'odd' if repeat.row.odd else 'even'">
  ...
</tr>
```

> [!note]
> Les propriétés exactes disponibles peuvent varier légèrement entre ZPT et Chameleon. Vérifiez la documentation de l'implémentation utilisée si vous exploitez des propriétés avancées.

# 12. Déballage dans une répétition

Chameleon supporte le déballage simple :

```html
<dl>
  <div tal:repeat="(key, value) data.items()" tal:omit-tag="">
    <dt tal:content="key">clé</dt>
    <dd tal:content="value">valeur</dd>
  </div>
</dl>
```

Cela est pratique pour parcourir :

```python
mapping.items()
```

ou une liste de tuples.

# 13. `tal:define`

`tal:define` crée une variable dans le template.

```html
<section tal:define="display_name user.full_name">
  <h2 tal:content="display_name">Nom</h2>
</section>
```

Plusieurs variables peuvent être définies avec `;` :

```html
<section tal:define="name user.full_name; count len(items)">
  ...
</section>
```

## 13.1 Portée locale

Par défaut, une variable est locale à l'élément et à ses descendants :

```html
<div tal:define="x 42">
  <p tal:content="x">42</p>
</div>
```

En dehors du `<div>`, `x` n'est normalement plus dans cette portée locale.

## 13.2 Masquage d'une variable

```html
<div tal:define="x 1">
  <span tal:content="x">1</span>

  <div tal:define="x 2">
    <span tal:content="x">2</span>
  </div>

  <span tal:content="x">1</span>
</div>
```

La variable intérieure masque temporairement la variable extérieure.

## 13.3 Variable globale

Certaines implémentations permettent :

```html
<div tal:define="global page_title 'Accueil'">
```

Une variable globale doit être utilisée avec parcimonie : elle rend les dépendances du template moins locales et peut compliquer la lecture.

# 14. `tal:attributes`

Cet attribut permet de créer, modifier ou supprimer des attributs HTML.

```html
<a href="#"
   tal:attributes="href item.url">
  Lien
</a>
```

Plusieurs attributs :

```html
<a href="#"
   class="link"
   tal:attributes="href item.url; title item.title">
  Voir
</a>
```

Si une expression d'attribut renvoie `None`, Chameleon peut supprimer l'attribut correspondant.

Exemple :

```html
<button tal:attributes="disabled True if locked else None">
  Envoyer
</button>
```

## 14.1 Attributs calculés à partir d'un dictionnaire

Chameleon accepte également un mapping dans certains usages :

```html
<div tal:attributes="html_attributes">
  ...
</div>
```

Cette technique est utile pour préparer les attributs côté Python.

# 15. `tal:omit-tag`

`tal:omit-tag` retire uniquement la balise englobante.

```html
<div tal:omit-tag="">
  <p>Contenu conservé</p>
</div>
```

Rendu :

```html
<p>Contenu conservé</p>
```

C'est particulièrement utile lorsqu'on a besoin d'un élément uniquement pour porter des instructions TAL.

Exemple :

```html
<div tal:repeat="item items" tal:omit-tag="">
  <p tal:content="item.name">Nom</p>
</div>
```

Une condition peut aussi décider si la balise doit être omise :

```html
<strong tal:omit-tag="not emphasize">
  Texte
</strong>
```

# 16. `tal:on-error`

`tal:on-error` permet de produire un contenu de secours si une erreur survient pendant le traitement de l'élément.

Exemple conceptuel :

```html
<span tal:on-error="string:Information indisponible"
      tal:content="service.expensive_value()">
  Valeur
</span>
```

> [!warning]
> `tal:on-error` ne doit pas devenir un mécanisme général pour cacher les bugs. Les erreurs applicatives doivent être journalisées et traitées au bon niveau.

Un template ne doit pas transformer silencieusement une panne critique en page apparemment correcte.

# 17. `tal:switch` et `tal:case`

Chameleon propose des instructions de sélection :

```html
<section tal:switch="item.type">
  <p tal:case="'document'">Document</p>
  <p tal:case="'folder'">Dossier</p>
  <p tal:case="default">Autre type</p>
</section>
```

Elles sont pratiques quand la sélection reste strictement liée à la présentation.

Pour une logique plus riche, préparez une valeur dans la vue Python.

# 18. Ordre d'évaluation des instructions TAL

Quand plusieurs instructions TAL apparaissent sur le même élément, l'implémentation suit un ordre déterminé.

Dans Chameleon, l'idée générale est notamment :

```text
define
  ↓
switch
  ↓
condition
  ↓
repeat
  ↓
case
  ↓
content / replace
  ↓
omit-tag
  ↓
attributes
```

Il ne faut donc pas supposer que l'ordre textuel des attributs HTML détermine leur ordre d'exécution.

Par exemple :

```html
<li tal:define="visible item.visible"
    tal:condition="visible"
    tal:repeat="subitem item.children">
```

est interprété selon les règles TAL, pas simplement de gauche à droite.

# 19. TALES : Template Attribute Language Expression Syntax

TAL décrit **quoi faire** au document ; TALES décrit **comment calculer la valeur** utilisée.

Forme générale :

```text
[type:] expression
```

Exemples :

```text
python: 1 + 2
string:Bonjour ${user.name}
exists: user.avatar
not: user.disabled
load: ../shared/layout.pt
import: datetime.datetime
```

# 20. Différence essentielle entre Zope et Chameleon

## 20.1 Zope Page Templates historique

Dans Zope, une expression sans préfixe est traditionnellement une **path expression** :

```html
<span tal:content="context/title">Titre</span>
```

équivaut conceptuellement à :

```text
path:context/title
```

## 20.2 Chameleon

Dans Chameleon, le type d'expression par défaut est **Python**.

On écrit donc naturellement :

```html
<span tal:content="context.title">Titre</span>
```

ou :

```html
<span tal:content="user.full_name">Nom</span>
```

> [!important]
> Quand vous migrez un vieux template Zope vers une application Pyramid + Chameleon, ne copiez pas mécaniquement les expressions `context/foo/bar`. Vérifiez quel moteur et quel mode de compatibilité sont réellement utilisés.

# 21. Expressions Python avec Chameleon

Exemples simples :

```html
<span tal:content="1 + 2">3</span>
```

```html
<span tal:content="user.name.upper()">ALICE</span>
```

```html
<p tal:condition="len(items) > 10">
  Beaucoup d'éléments
</p>
```

```html
<span tal:content="', '.join(tags)">tags</span>
```

Le fait que Python soit possible ne signifie pas qu'il faut écrire de la logique applicative complexe dans le template.

# 22. `string:` et interpolation

Une expression chaîne :

```html
<p tal:content="string:Bonjour ${user.name}">
  Bonjour Alice
</p>
```

Chameleon permet aussi l'interpolation dans de nombreux contextes :

```html
<p>Bonjour ${user.name}</p>
```

Exemple d'URL :

```html
<a href="/users/${user.id}">Profil</a>
```

Pour produire un dollar littéral lorsque l'interpolation est active, on utilise généralement `$$` :

```text
$$
```

# 23. `exists:`

`exists:` teste si une expression peut être évaluée sans certaines erreurs d'accès.

```html
<img tal:condition="exists: user.avatar_url"
     tal:attributes="src user.avatar_url"
     alt="Avatar">
```

C'est différent de :

```html
<img tal:condition="user.avatar_url">
```

Dans le second cas, l'attribut doit exister et sa valeur doit être vraie.

> [!tip]
> Dans du code neuf, il est souvent plus clair de normaliser les données côté Python, par exemple avec `avatar_url = None`, plutôt que de dépendre partout de `exists:`.

# 24. `not:`

`not:` inverse une valeur booléenne :

```html
<p tal:condition="not: user.is_authenticated">
  Vous devez vous connecter.
</p>
```

Avec Chameleon, on peut généralement écrire plus simplement :

```html
<p tal:condition="not user.is_authenticated">
  Vous devez vous connecter.
</p>
```

puisque Python est l'expression par défaut.

# 25. `import:`

Chameleon propose une expression d'import :

```html
<div tal:define="datetime import:datetime.datetime">
  ...
</div>
```

ou :

```html
<div tal:define="re import:re">
  ...
</div>
```

> [!warning]
> Le fait qu'un import soit possible dans un template ne signifie pas qu'il est souhaitable architecturalement. Préférez fournir les données déjà préparées par la vue.

Mauvais usage :

```html
<div tal:define="db import:myapp.database">
  ... requêtes depuis le template ...
</div>
```

Le template ne doit pas devenir une couche d'accès aux données.

# 26. `load:`

Chameleon peut charger un autre template relativement au template courant :

```html
<div tal:define="master load: ../shared/master.pt"
     metal:use-macro="master.macros.layout">
  ...
</div>
```

Selon la classe de template et la forme du macro, une syntaxe plus directe peut être utilisée.

`load:` est particulièrement intéressant pour découpler un template de noms globaux spécifiques au framework.

# 27. `structure:`

Chameleon permet :

```html
<div tal:content="structure: trusted_html"></div>
```

Le résultat n'est pas échappé comme du simple texte.

À retenir :

```text
texte utilisateur → échappement obligatoire
HTML généré et contrôlé → structure éventuellement possible
```

# 28. Valeurs spéciales `nothing` et `default`

Les Page Templates disposent historiquement de valeurs spéciales.

## 28.1 `nothing`

`nothing` représente l'absence de valeur.

Exemple :

```html
<div tal:replace="nothing">
  Ce bloc disparaît.
</div>
```

## 28.2 `default`

`default` demande souvent de conserver le contenu ou l'attribut statique du template.

Exemple conceptuel :

```html
<span tal:content="default">Texte de démonstration</span>
```

Le comportement précis dépend de l'instruction TAL concernée.

# 29. Expressions de secours

Certaines implémentations permettent un mécanisme de fallback avec `|`.

Exemple Chameleon :

```html
<div tal:define="page request.GET['page'] | 0">
  ...
</div>
```

Dans Zope classique, les alternatives sont très liées aux path expressions :

```html
<p tal:content="request/form/age | python:18">18</p>
```

> [!warning]
> Le fallback ne doit pas servir à masquer des exceptions inattendues. Utilisez-le pour des valeurs réellement optionnelles et maîtrisées.

# 30. METAL : Macro Expansion TAL

METAL fournit un mécanisme de composants structurels.

Les instructions centrales sont :

| Instruction | Rôle |
|---|---|
| `metal:define-macro` | déclarer une macro |
| `metal:use-macro` | utiliser une macro |
| `metal:define-slot` | déclarer une zone remplaçable |
| `metal:fill-slot` | fournir le contenu d'une zone |
| `metal:extend-macro` | étendre une macro dans Chameleon |

# 31. Définir une macro

Fichier `layout.pt` :

```html
<!doctype html>
<html metal:define-macro="layout" lang="fr">
  <head>
    <meta charset="utf-8">
    <title metal:define-slot="title">Mon application</title>
  </head>
  <body>
    <header>
      <h1>Mon application</h1>
    </header>

    <main metal:define-slot="content">
      Contenu par défaut
    </main>
  </body>
</html>
```

Cette macro définit :

- une structure commune ;
- un slot `title` ;
- un slot `content`.

# 32. Utiliser une macro

Exemple avec une macro disponible dans `layout` :

```html
<html metal:use-macro="layout.macros.layout">
  <title metal:fill-slot="title">Accueil</title>

  <main metal:fill-slot="content">
    <h2>Bienvenue</h2>
    <p>Page d'accueil.</p>
  </main>
</html>
```

Le rendu reprend la structure de la macro et remplace uniquement les slots fournis.

# 33. Le contenu par défaut d'un slot

Un slot peut posséder un contenu de secours :

```html
<footer metal:define-slot="footer">
  <p>© Exemple</p>
</footer>
```

Si le template utilisateur ne fournit pas :

```html
metal:fill-slot="footer"
```

alors le footer par défaut est conservé.

Cela permet de construire des layouts avec des zones :

- obligatoires par convention ;
- facultatives avec valeur par défaut.

# 34. Composant réutilisable simple

`components.pt` :

```html
<article metal:define-macro="card" class="card">
  <h2 metal:define-slot="title">Titre</h2>
  <div metal:define-slot="body">
    Corps de la carte
  </div>
</article>
```

Utilisation :

```html
<article metal:use-macro="components.macros.card">
  <h2 metal:fill-slot="title" tal:content="product.name">
    Produit
  </h2>

  <div metal:fill-slot="body">
    <p tal:content="product.description">Description</p>
  </div>
</article>
```

# 35. `metal:extend-macro`

Chameleon fournit `metal:extend-macro` pour construire une macro à partir d'une autre tout en redéfinissant certains slots.

Exemple conceptuel :

```html
<div metal:define-macro="admin-layout"
     metal:extend-macro="layouts.macros.base">

  <nav metal:fill-slot="sidebar">
    <a href="/admin">Administration</a>
    <div metal:define-slot="sidebar"></div>
  </nav>
</div>
```

Cette fonctionnalité est utile, mais elle peut créer des hiérarchies de layouts difficiles à suivre si elle est utilisée excessivement.

# 36. Macros ou composants Python ?

Une macro est adaptée à la **réutilisation de structure HTML**.

Un composant Python est préférable quand il faut :

- charger des données ;
- appliquer des permissions ;
- calculer une règle métier ;
- appeler des services ;
- encapsuler un état complexe.

Bonne séparation :

```text
Python
  ├── choisit les données
  ├── applique les permissions
  └── calcule l'état

METAL
  ├── réutilise la structure HTML
  └── expose des slots

TAL
  └── adapte légèrement le rendu
```

# 37. I18N dans les Page Templates

Les Page Templates possèdent aussi un espace de noms `i18n`.

Exemple :

```html
<h1 i18n:translate="dashboard_title">
  Tableau de bord
</h1>
```

Un attribut peut être marqué comme traduisible :

```html
<img src="logo.svg"
     alt="Logo de l'application"
     i18n:attributes="alt">
```

Autres instructions courantes :

- `i18n:translate` ;
- `i18n:domain` ;
- `i18n:context` ;
- `i18n:name` ;
- `i18n:attributes`.

Voir aussi [[Internationalisation]].

# 38. Interpolation et i18n

Il est fréquent d'avoir un texte traduit contenant une valeur dynamique.

Exemple conceptuel :

```html
<p i18n:translate="welcome_user">
  Bonjour
  <strong i18n:name="username" tal:content="user.name">Alice</strong>
</p>
```

Le nom dynamique devient un paramètre de la traduction plutôt qu'une concaténation de chaînes.

C'est préférable car l'ordre des mots varie selon les langues.

# 39. Utiliser Chameleon directement en Python

Installation :

```bash
python -m pip install Chameleon
```

Exemple :

```python
from chameleon import PageTemplate

source = """
<h1 tal:content="title">Titre</h1>
<ul>
  <li tal:repeat="item items" tal:content="item">Élément</li>
</ul>
"""

template = PageTemplate(source)

html = template(
    title="Exemple",
    items=["A", "B", "C"],
)

print(html)
```

Cette utilisation montre qu'un template Chameleon n'est pas dépendant de Zope ou de Pyramid.

# 40. Charger des fichiers avec Chameleon

Pour des templates sur disque, on peut utiliser un loader.

Exemple conceptuel :

```python
from chameleon import PageTemplateLoader

loader = PageTemplateLoader("templates")
template = loader["home.pt"]

html = template(
    title="Accueil",
    user=current_user,
)
```

Les options exactes du loader permettent notamment de contrôler certains comportements d'analyse, de rechargement et de rendu.

# 41. Chameleon avec Pyramid

Dans Pyramid, l'intégration se fait via `pyramid_chameleon`.

Installation :

```bash
python -m pip install pyramid_chameleon
```

Activation :

```python
config.include("pyramid_chameleon")
```

ou dans une configuration INI :

```ini
pyramid.includes =
    pyramid_chameleon
```

Une vue peut ensuite désigner un template `.pt` comme renderer.

Exemple :

```python
from pyramid.view import view_config


@view_config(route_name="home", renderer="templates/home.pt")
def home_view(request):
    return {
        "title": "Accueil",
        "user": request.user,
    }
```

Template :

```html
<h1 tal:content="title">Accueil</h1>
```

# 42. Ce qui doit être préparé dans la vue Pyramid

Préférer :

```python
@view_config(route_name="orders", renderer="templates/orders.pt")
def orders_view(request):
    orders = order_service.visible_orders_for(request.user)

    return {
        "orders": orders,
        "can_create_order": permission_service.can_create_order(request.user),
    }
```

puis :

```html
<a tal:condition="can_create_order" href="/orders/new">
  Nouvelle commande
</a>
```

Éviter :

```html
<a tal:condition="request.user.groups and 'sales' in request.user.groups and not request.user.suspended and ...">
```

# 43. Sécurité : XSS

La règle principale est :

> [!danger]
> Conservez l'échappement automatique pour toute donnée dont vous ne contrôlez pas strictement le contenu.

Sûr par défaut :

```html
<p tal:content="comment.body">Commentaire</p>
```

Potentiellement dangereux :

```html
<p tal:content="structure: comment.body">Commentaire</p>
```

Si le texte contient :

```html
<img src=x onerror=alert(1)>
```

l'utilisation de `structure` peut le transformer en balisage actif.

# 44. Sécurité : URLs et attributs

L'échappement HTML ne remplace pas la validation métier d'une URL.

Exemple :

```html
<a tal:attributes="href user_provided_url">Lien</a>
```

L'application doit encore décider quels schémas et destinations sont autorisés.

Une chaîne correctement échappée peut malgré tout être une destination indésirable.

# 45. Sécurité : ne pas exposer trop d'objets

Éviter de transmettre au template un objet donnant accès à toute l'infrastructure :

```python
return {
    "container": dependency_injection_container,
}
```

Préférer un contexte minimal :

```python
return {
    "user": user_view_model,
    "orders": order_view_models,
}
```

Cela améliore :

- la sécurité ;
- la testabilité ;
- la lisibilité ;
- le découplage.

# 46. View models

Pour les pages riches, un **view model** est souvent plus propre qu'une accumulation de logique dans le template.

Exemple :

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class UserRow:
    id: int
    label: str
    profile_url: str
    status_label: str
    can_edit: bool
```

Template :

```html
<tr tal:repeat="row rows">
  <td tal:content="row.label">Nom</td>
  <td tal:content="row.status_label">Statut</td>
  <td>
    <a tal:condition="row.can_edit"
       tal:attributes="href row.profile_url">
      Modifier
    </a>
  </td>
</tr>
```

Le template devient presque déclaratif.

# 47. Éviter les appels coûteux dans le template

Mauvais :

```html
<li tal:repeat="user users"
    tal:content="database.load_profile(user.id).display_name">
```

Cela peut provoquer :

```text
1 requête pour la liste
+ N requêtes pendant le rendu
= problème N+1
```

Préparer les données avant le rendu :

```python
rows = profile_service.build_rows(users)
```

# 48. Ne pas modifier l'état depuis un template

Un template doit être une étape de rendu, pas un endroit pour produire des effets de bord.

À éviter :

```html
<div tal:define="result service.delete_old_records()">
```

ou :

```html
<span tal:content="mailer.send_notification(user)"></span>
```

Le rendu doit pouvoir être rejoué sans déclencher d'action métier inattendue.

# 49. Macros : éviter les dépendances cachées

Une macro qui lit vingt variables globales est difficile à réutiliser.

Fragile :

```html
<div metal:define-macro="card">
  <!-- dépend implicitement de user, request, site, settings, permissions... -->
</div>
```

Mieux :

- utiliser des conventions explicites ;
- préparer un view model ;
- limiter les dépendances ;
- documenter les variables attendues.

Exemple de commentaire :

```html
<!--
Macro: user_card
Contexte attendu:
- card.user
- card.profile_url
- card.can_edit
-->
```

# 50. Layouts imbriqués : rester raisonnable

On peut bâtir :

```text
base
 └── application
      └── administration
           └── page
```

mais une hiérarchie trop profonde produit :

- des slots difficiles à localiser ;
- des dépendances implicites ;
- des rendus complexes à déboguer.

Deux ou trois niveaux bien conçus sont souvent préférables à une hiérarchie de macros comparable à une hiérarchie de classes profonde.

# 51. Pattern : layout principal

`layout.pt` :

```html
<!doctype html>
<html metal:define-macro="layout" lang="fr">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <title metal:define-slot="title">Application</title>

  <metal:block define-slot="head_extra"></metal:block>
</head>
<body>
  <header>
    <a href="/">Application</a>
  </header>

  <main metal:define-slot="content">
    Contenu
  </main>

  <footer>
    <small>Application</small>
  </footer>

  <metal:block define-slot="scripts"></metal:block>
</body>
</html>
```

# 52. Pattern : page spécialisée

```html
<html metal:use-macro="layout.macros.layout">

  <title metal:fill-slot="title">
    Utilisateurs
  </title>

  <main metal:fill-slot="content">
    <h1>Utilisateurs</h1>

    <ul>
      <li tal:repeat="user users">
        <a tal:attributes="href user.profile_url"
           tal:content="user.label">
          Utilisateur
        </a>
      </li>
    </ul>
  </main>

</html>
```

# 53. Pattern : état vide

```html
<ul tal:condition="items">
  <li tal:repeat="item items"
      tal:content="item.label">
    Élément
  </li>
</ul>

<p tal:condition="not items">
  Aucun élément à afficher.
</p>
```

Si cet état devient complexe, préparez plutôt :

```python
has_items = bool(items)
```

# 54. Pattern : classes CSS calculées

Simple :

```html
<span tal:attributes="class 'badge badge-success' if active else 'badge badge-muted'">
  Statut
</span>
```

Pour des classes complexes :

```python
row.css_classes = " ".join(classes)
```

puis :

```html
<tr tal:attributes="class row.css_classes">
```

# 55. Pattern : attribut facultatif

```html
<a tal:attributes="title item.tooltip if item.tooltip else None">
  Lien
</a>
```

Cela évite un attribut vide inutile quand le moteur supprime l'attribut pour `None`.

# 56. Pattern : rendu conditionnel d'un composant

```html
<section tal:condition="panel.visible">
  <h2 tal:content="panel.title">Titre</h2>
  <p tal:content="panel.text">Texte</p>
</section>
```

Au lieu de recalculer la visibilité dans le template :

```python
panel.visible = policy.can_view_panel(user, resource)
```

# 57. Tester les templates

Les templates doivent être testés comme du code.

Plusieurs niveaux sont possibles :

```text
1. compilation du template
2. test de rendu isolé
3. test de la vue + renderer
4. test HTTP d'intégration
5. test navigateur si interaction nécessaire
```

# 58. Test de rendu Chameleon isolé

Exemple :

```python
from chameleon import PageTemplate


def test_title_is_escaped():
    template = PageTemplate(
        '<h1 tal:content="title">Titre</h1>'
    )

    result = template(title="<script>alert(1)</script>")

    assert "<script>" not in result
    assert "&lt;script&gt;" in result
```

Ce test vérifie une propriété de sécurité importante.

# 59. Test d'un état vide

```python
result = template(items=[])

assert "Aucun élément" in result
```

Puis :

```python
result = template(items=[item])

assert item.label in result
assert "Aucun élément" not in result
```

# 60. Tester les macros

Une macro doit être testée au moins dans :

- son état par défaut ;
- chaque slot important ;
- un cas avec contenu dynamique ;
- un cas d'échappement de données.

Le but est d'éviter qu'un refactoring du layout casse silencieusement plusieurs dizaines de pages.

# 61. Déboguer un template

Lors d'une erreur, distinguer :

```text
Syntaxe du template ?
Expression TALES ?
Variable absente ?
Mauvaise portée ?
Macro introuvable ?
Slot incorrect ?
Objet Python inattendu ?
```

Commencer par identifier :

- le fichier ;
- la ligne ;
- l'expression ;
- le nom manquant ;
- le contexte envoyé au renderer.

# 62. Erreur : variable absente

Template :

```html
<span tal:content="current_user.name">Nom</span>
```

Vue :

```python
return {"user": user}
```

Le problème est un contrat de contexte incohérent.

Solutions :

```python
return {"current_user": user}
```

ou modifier le template :

```html
<span tal:content="user.name">Nom</span>
```

# 63. Erreur : macro introuvable

Si :

```html
metal:use-macro="layout.macros.main"
```

échoue, vérifier :

1. que `layout` désigne bien le template attendu ;
2. que celui-ci contient `metal:define-macro="main"` ;
3. que le chargement du template est correct ;
4. que le chemin est relatif au bon fichier ou package ;
5. que le moteur utilisé expose bien `.macros` de cette manière.

# 64. Erreur : slot ignoré

Un `metal:fill-slot` dont le nom ne correspond à aucun `metal:define-slot` peut être ignoré selon l'implémentation.

Exemple :

```html
metal:fill-slot="contents"
```

alors que la macro définit :

```html
metal:define-slot="content"
```

Cette faute est particulièrement difficile à voir si le slot possède déjà un contenu par défaut.

# 65. Erreur : HTML non échappé

Symptôme :

```html
<script>
```

apparaît dans le DOM alors que la donnée vient d'un utilisateur.

Chercher en priorité :

```text
structure
structure:
__html__
Markup / safe string
```

et remonter jusqu'à l'endroit où la valeur a été considérée comme sûre.

# 66. Erreur : trop de logique dans le template

Signes :

- expressions de plusieurs lignes ;
- appels de services ;
- multiples accès à `request` ;
- règles de permissions ;
- transformations de structures de données ;
- tri et regroupement complexes ;
- calculs métier.

Refactoring :

```text
template complexe
      │
      ▼
identifier les calculs
      │
      ▼
les déplacer dans Python
      │
      ▼
retourner un view model simple
      │
      ▼
template déclaratif
```

# 67. TAL vs Jinja2

Les deux approches ont des philosophies différentes.

TAL :

```html
<li tal:repeat="item items" tal:content="item.name">Nom</li>
```

Jinja2 :

```jinja2
{% for item in items %}
  <li>{{ item.name }}</li>
{% endfor %}
```

TAL garde davantage la structure HTML dans les attributs.

Jinja introduit des délimiteurs de contrôle explicites.

Le choix dépend notamment :

- du framework ;
- de l'historique du projet ;
- de l'équipe ;
- des outils de traduction ;
- du besoin de réutiliser des macros ZPT existantes.

# 68. TAL vs templates côté client

TAL/METAL produisent du HTML **côté serveur**.

Ils ne remplacent pas :

- React ;
- Vue ;
- Svelte ;
- Web Components.

Une architecture peut toutefois combiner :

```text
Pyramid + Chameleon
        │
        ▼
HTML initial côté serveur
        │
        ├── formulaire classique
        ├── htmx
        ├── Alpine.js
        └── composant JS local
```

Il n'est pas nécessaire d'utiliser une SPA pour chaque interface web.

# 69. TAL et htmx

TAL fonctionne très bien avec des fragments HTML rendus côté serveur.

Exemple :

```html
<button
  hx-get="/users/${user.id}/details"
  hx-target="#user-details-${user.id}"
  hx-swap="innerHTML">
  Détails
</button>

<div id="user-details-${user.id}"></div>
```

Le serveur peut rendre un fragment `.pt` distinct.

Cela permet de conserver :

- logique métier côté Python ;
- HTML côté serveur ;
- interactions partielles côté navigateur.

# 70. Convention de dossiers

Exemple :

```text
myapp/
├── views/
│   ├── home.py
│   ├── users.py
│   └── orders.py
└── templates/
    ├── layouts/
    │   ├── base.pt
    │   └── admin.pt
    ├── components/
    │   ├── cards.pt
    │   └── forms.pt
    ├── users/
    │   ├── list.pt
    │   └── detail.pt
    └── orders/
        └── list.pt
```

Cette organisation évite un dossier `templates/` contenant des centaines de fichiers sans structure.

# 71. Nommer les macros

Préférer des noms décrivant une **responsabilité de présentation** :

```text
layout
admin_layout
user_card
pagination
flash_messages
form_field
modal
```

Éviter :

```text
macro1
template
thing
common2
```

# 72. Quand créer une macro ?

Créer une macro quand :

- une structure HTML est réellement répétée ;
- elle possède une identité de composant ;
- elle nécessite quelques points de personnalisation ;
- le copier-coller devient une source de divergence.

Ne pas créer une macro pour trois lignes utilisées une seule fois simplement « au cas où ».

# 73. Quand créer un slot ?

Créer un slot lorsque l'appelant doit personnaliser une **zone structurelle cohérente**.

Bon :

```text
layout
├── title
├── head_extra
├── content
└── scripts
```

Moins bon :

```text
slot_before_first_div
slot_inside_second_span
slot_after_icon
```

Un excès de slots révèle souvent qu'une macro essaie de devenir un mini-framework.

# 74. Contrat d'un template

Traiter le contexte de rendu comme une interface.

Exemple :

```text
users/list.pt

Entrées :
- title: str
- rows: Sequence[UserRow]
- can_create: bool
- create_url: str | None
```

Cette documentation peut être conservée près de la vue ou dans le module Python.

# 75. Typage du contexte côté Python

On peut formaliser le contrat avec `TypedDict` :

```python
from typing import TypedDict


class UsersTemplateContext(TypedDict):
    title: str
    rows: list[UserRow]
    can_create: bool
    create_url: str | None
```

Puis :

```python
def build_context(...) -> UsersTemplateContext:
    return {
        "title": "Utilisateurs",
        "rows": rows,
        "can_create": can_create,
        "create_url": create_url,
    }
```

Même si le template n'est pas lui-même typé statiquement, la frontière Python devient plus claire.

# 76. View model immutable

Un view model immuable réduit les effets de bord :

```python
@dataclass(frozen=True, slots=True)
class ProductCard:
    name: str
    url: str
    price_label: str
    available: bool
```

Le template lit les données sans les modifier.

# 77. Performance

Chameleon compile les templates en code Python, ce qui explique son positionnement comme moteur de templates rapide.

La performance d'une page dépend toutefois souvent beaucoup plus :

- des requêtes SQL ;
- des appels réseau ;
- des services externes ;
- des calculs répétés ;
- du nombre d'objets ;
- de la stratégie de cache.

Optimiser une expression TAL de quelques microsecondes n'a aucun intérêt si la vue réalise cent requêtes SQL.

# 78. Auto-reload en développement

Un environnement de développement peut recharger les templates lorsque les fichiers changent.

En production, on préfère généralement :

- éviter les vérifications de fichier inutiles ;
- compiler/cacher selon les capacités du moteur ;
- déployer des artefacts déterministes.

Ne transposez pas aveuglément les options de développement en production.

# 79. Les templates comme code versionné

Un fichier `.pt` doit être :

- versionné avec Git ;
- relu en revue de code ;
- formaté de manière stable ;
- testé ;
- analysé comme une partie du logiciel.

Il ne s'agit pas d'un simple « fichier de présentation » secondaire.

# 80. Revue de code d'un template

Questions utiles :

```text
La donnée est-elle échappée ?
La logique est-elle seulement de présentation ?
La vue pourrait-elle préparer cette valeur ?
La macro a-t-elle trop de dépendances implicites ?
Le nom du slot est-il clair ?
Une condition reflète-t-elle une permission réelle ?
Le template peut-il provoquer un effet de bord ?
Le rendu vide est-il prévu ?
Les traductions sont-elles correctement paramétrées ?
```

# 81. Anti-pattern : autorisation uniquement dans le template

Mauvais :

```html
<a tal:condition="user.is_admin" href="/admin/delete/42">
  Supprimer
</a>
```

Cacher le bouton n'est **pas** une mesure d'autorisation suffisante.

Le backend doit encore vérifier le droit lors de la requête :

```text
UI conditionnelle ≠ contrôle d'accès
```

Le template améliore l'interface ; il ne remplace jamais l'autorisation serveur.

# 82. Anti-pattern : accès direct aux paramètres sensibles

Éviter de disperser :

```html
request.params['foo']
request.cookies['bar']
request.registry.settings['secret']
```

à travers les templates.

Préférer :

```python
return {
    "foo": validated_foo,
}
```

après validation côté Python.

# 83. Anti-pattern : formatter toutes les données dans TAL

Exemple fragile :

```html
<span tal:content="f'{order.total / 100:.2f} €'">
```

Pour une application internationalisée, le format monétaire dépend de :

- la locale ;
- la devise ;
- les règles d'arrondi ;
- la position du symbole.

Préparer plutôt :

```python
price_label = money_formatter.format(order.total, currency="EUR", locale=locale)
```

# 84. Anti-pattern : macro gigantesque

Une macro de 800 lignes avec 25 slots est rarement une abstraction saine.

Refactoring :

```text
macro géante
   │
   ├── layout
   ├── navigation
   ├── flash messages
   ├── card
   ├── pagination
   └── formulaire
```

La composition est généralement plus lisible qu'un composant universel.

# 85. Migration d'un vieux ZPT vers Chameleon

Checklist :

1. identifier le moteur actuel ;
2. inventorier les expressions `path:` implicites ;
3. vérifier les objets `context`, `request`, `options`, `modules` ;
4. vérifier les macros ;
5. vérifier l'i18n ;
6. repérer `structure` ;
7. tester l'échappement ;
8. contrôler les extensions spécifiques à Zope ;
9. écrire des tests de rendu avant modification ;
10. migrer progressivement.

# 86. `options` dans Zope vs arguments directs dans Chameleon

Dans ZPT classique, des valeurs supplémentaires sont historiquement accessibles via des contextes comme `options`.

Dans Chameleon, les arguments passés au rendu sont injectés directement dans l'espace d'exécution du template.

Exemple Chameleon :

```python
template(title="Accueil")
```

Template :

```html
<h1 tal:content="title">Accueil</h1>
```

Cette différence doit être prise en compte lors d'une migration.

# 87. Compatibilité et extensions

TAL/METAL désignent un langage, mais les implémentations possèdent des extensions et différences.

Exemples de spécificités Chameleon :

- expression Python par défaut ;
- `import:` ;
- `load:` ;
- `structure:` comme type d'expression ;
- unpacking simple dans certaines instructions ;
- `tal:switch` / `tal:case` ;
- `metal:extend-macro`.

> [!important]
> Quand vous écrivez une bibliothèque de templates devant fonctionner sur plusieurs implémentations, utilisez le sous-ensemble réellement commun ou documentez explicitement le moteur requis.

# 88. Faut-il encore apprendre TAL/METAL ?

Oui si vous travaillez avec :

- Zope ;
- Plone ;
- Pyramid + `pyramid_chameleon` ;
- une application historique utilisant des `.pt` ;
- une bibliothèque basée sur Chameleon.

Il n'est en revanche pas nécessaire de choisir TAL pour un projet neuf uniquement parce que le langage existe.

Le choix d'un moteur doit tenir compte de :

- l'écosystème ;
- la maintenance ;
- les compétences ;
- l'i18n ;
- les outils existants ;
- la quantité de templates à conserver.

# 89. Méthode d'apprentissage recommandée

Apprendre dans cet ordre :

```text
1. tal:content
2. tal:condition
3. tal:repeat
4. tal:attributes
5. tal:define
6. tal:replace / omit-tag
7. TALES
8. METAL macros
9. slots
10. i18n
11. intégration framework
12. sécurité et tests
```

Commencer directement par des macros imbriquées masque les concepts de base.

# 90. Exercice 1 — liste d'utilisateurs

Données :

```python
users = [
    {"name": "Ada", "active": True},
    {"name": "Grace", "active": False},
]
```

Objectif : produire une liste où :

- chaque utilisateur est rendu avec `tal:repeat` ;
- le nom est inséré avec `tal:content` ;
- un badge n'apparaît que si l'utilisateur est actif.

Solution possible :

```html
<ul>
  <li tal:repeat="user users">
    <span tal:content="user['name']">Nom</span>
    <strong tal:condition="user['active']">Actif</strong>
  </li>
</ul>
```

# 91. Exercice 2 — attributs dynamiques

Données :

```python
users = [
    {"id": 1, "name": "Ada"},
    {"id": 2, "name": "Grace"},
]
```

Créer des liens :

```text
/users/1
/users/2
```

Solution :

```html
<ul>
  <li tal:repeat="user users">
    <a tal:attributes="href string:/users/${user['id']}"
       tal:content="user['name']">
      Profil
    </a>
  </li>
</ul>
```

# 92. Exercice 3 — layout METAL

Créer :

```text
layout.pt
home.pt
```

Le layout doit exposer :

- `title` ;
- `content`.

`home.pt` doit remplir les deux slots.

Objectif : comprendre que METAL réutilise une **structure**, tandis que TAL produit le contenu dynamique.

# 93. Exercice 4 — XSS

Donnée :

```python
comment = "<img src=x onerror=alert(1)>"
```

Comparer :

```html
<p tal:content="comment"></p>
```

et :

```html
<p tal:content="structure: comment"></p>
```

Observer le HTML généré et expliquer pourquoi la seconde forme est dangereuse.

# 94. Exercice 5 — refactoring

Template initial :

```html
<div tal:condition="user and user.enabled and user.role == 'admin' and settings.get('admin_enabled')">
  <a href="/admin">Administration</a>
</div>
```

Refactorer en préparant :

```python
can_access_admin
```

côté Python.

Puis simplifier le template :

```html
<div tal:condition="can_access_admin">
  <a href="/admin">Administration</a>
</div>
```

# 95. Projet guidé — page de catalogue

Objectif : construire une page contenant :

- un layout METAL ;
- un titre ;
- une liste de produits ;
- un état vide ;
- une carte produit ;
- une URL dynamique ;
- un prix préparé côté Python ;
- une classe CSS selon la disponibilité.

View model :

```python
@dataclass(frozen=True, slots=True)
class ProductRow:
    name: str
    url: str
    price_label: str
    css_class: str
    available: bool
```

Vue :

```python
@view_config(route_name="catalog", renderer="templates/catalog.pt")
def catalog(request):
    products = product_service.list_visible_products(request.user)

    rows = [
        ProductRow(
            name=p.name,
            url=request.route_url("product", id=p.id),
            price_label=format_price(p.price),
            css_class="available" if p.available else "unavailable",
            available=p.available,
        )
        for p in products
    ]

    return {
        "rows": rows,
    }
```

Template :

```html
<html metal:use-macro="layout.macros.layout">
  <title metal:fill-slot="title">Catalogue</title>

  <main metal:fill-slot="content">
    <h1>Catalogue</h1>

    <div tal:condition="rows">
      <article tal:repeat="row rows"
               tal:attributes="class row.css_class">
        <h2>
          <a tal:attributes="href row.url"
             tal:content="row.name">
            Produit
          </a>
        </h2>

        <p tal:content="row.price_label">10,00 €</p>

        <p tal:condition="row.available">Disponible</p>
        <p tal:condition="not row.available">Indisponible</p>
      </article>
    </div>

    <p tal:condition="not rows">
      Aucun produit disponible.
    </p>
  </main>
</html>
```

Cette architecture garde les responsabilités claires.

# 96. Checklist de production

Avant de livrer un ensemble de templates TAL/METAL :

- [ ] toutes les données utilisateur sont échappées ;
- [ ] chaque usage de `structure` est justifié ;
- [ ] les permissions sont contrôlées côté serveur ;
- [ ] les templates ne font pas de requêtes base de données ;
- [ ] les templates ne déclenchent pas d'effets de bord ;
- [ ] les macros ont des responsabilités limitées ;
- [ ] les slots sont clairement nommés ;
- [ ] les états vides sont traités ;
- [ ] les erreurs principales sont testées ;
- [ ] les traductions n'utilisent pas de concaténation fragile ;
- [ ] les templates sont versionnés et revus ;
- [ ] le contexte de rendu est documenté.

# 97. Résumé

```text
TAL
├── define      → variables
├── condition   → condition
├── repeat      → boucle
├── content     → remplacer le contenu
├── replace     → remplacer l'élément
├── attributes  → attributs dynamiques
├── omit-tag    → retirer la balise
└── on-error    → fallback de rendu

TALES
├── python
├── string
├── exists
├── not
├── import      → Chameleon
├── load        → Chameleon
└── structure   → HTML explicitement sûr

METAL
├── define-macro
├── use-macro
├── define-slot
├── fill-slot
└── extend-macro → Chameleon
```

Principe à retenir :

> **Python prépare les données et applique les règles métier ; TAL adapte le HTML ; TALES évalue les petites expressions de présentation ; METAL compose les structures réutilisables.**

# 98. Sources et documentation

Documentation de référence à privilégier :

- Chameleon — *Language Reference* : <https://chameleon.readthedocs.io/en/latest/reference.html>
- Chameleon sur PyPI : <https://pypi.org/project/Chameleon/>
- Zope — documentation actuelle : <https://zope.readthedocs.io/en/latest/>
- Zope Page Templates Reference : <https://zope.readthedocs.io/en/latest/zopebook/AppendixC.html>
- Pyramid Chameleon : <https://docs.pylonsproject.org/projects/pyramid-chameleon/en/latest/>
- Pyramid : <https://docs.pylonsproject.org/projects/pyramid/en/latest/>

> [!note]
> Les exemples de ce cours privilégient la syntaxe **Chameleon moderne** lorsqu'une différence existe. Pour une application Zope historique, vérifiez les expressions `path:` et les conventions de contexte propres à ZPT avant de modifier des templates existants.
