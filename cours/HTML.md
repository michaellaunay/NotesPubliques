---
schema_version: 1
uid: "01M02EX5B18KFQ629QHS96MJT0"
titre: "HTML"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - developpement-web
  - html
resume: "Cours complet de HTML moderne : structure, sémantique, formulaires, médias, accessibilité, SEO, performance, sécurité et fonctionnalités natives du HTML Living Standard."
niveau: debutant
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-03-07
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# HTML

> [!abstract] Objectif
> Apprendre à produire des documents HTML **valides, sémantiques, accessibles, performants et maintenables**, en utilisant d'abord les capacités natives de la plateforme web avant d'ajouter du CSS ou du JavaScript.

HTML signifie **HyperText Markup Language**. C'est le langage de balisage qui décrit la **structure** et le **sens** d'un document web.

Il faut distinguer trois responsabilités :

- **HTML** : contenu, structure et sémantique ;
- **CSS** : présentation, mise en page et adaptation visuelle ;
- **JavaScript** : comportement dynamique et logique applicative.

HTML n'est donc pas un langage de programmation : il ne décrit pas une suite d'instructions à exécuter, mais un **arbre de contenus** interprété par le navigateur.

> [!important] HTML aujourd'hui
> On parle encore couramment de « HTML5 », mais HTML n'est plus développé comme une suite de versions figées. La référence actuelle est le **HTML Living Standard** maintenu par le WHATWG et mis à jour en continu.

Voir aussi : [[CSS]], [[Javascript]], [[SEO]], [[HTTP]].

---

# Sommaire

1. Introduction et modèle mental
2. Structure d'un document HTML
3. Syntaxe, éléments et attributs
4. Texte, listes et contenu éditorial
5. Structure sémantique d'une page
6. Liens et navigation
7. Images et médias
8. Tableaux
9. Formulaires
10. Métadonnées et `<head>`
11. Accessibilité
12. HTML, CSS et JavaScript
13. Éléments interactifs natifs
14. Intégration de contenus externes
15. Performance et chargement
16. Sécurité
17. SEO et partage
18. Validation, qualité et maintenance
19. Web Components et HTML avancé
20. Projet final
21. Travaux pratiques
22. Checklist
23. Conclusion et ressources

---

# 1. Introduction et modèle mental

## 1.1 Origine du HTML

HTML est né au début des années 1990 avec le World Wide Web, sous l'impulsion de **Tim Berners-Lee** au CERN.

L'idée fondamentale est simple : un document peut :

- contenir du texte structuré ;
- référencer d'autres ressources ;
- contenir des **hyperliens** vers d'autres documents ;
- être transmis par le protocole HTTP ;
- être interprété de façon interopérable par différents navigateurs.

Le Web repose donc sur plusieurs standards complémentaires :

| Élément | Rôle |
|---|---|
| URL | identifier une ressource |
| HTTP | transporter les requêtes et réponses |
| HTML | décrire le document |
| CSS | décrire sa présentation |
| JavaScript | ajouter du comportement |

## 1.2 HTML décrit du sens

Une erreur fréquente consiste à choisir une balise parce qu'elle « ressemble » visuellement à ce que l'on veut.

Exemple à éviter :

```html
<div class="gros-texte">Titre</div>
```

Préférer :

```html
<h1>Titre</h1>
```

Pourquoi ?

Parce que `<h1>` indique explicitement qu'il s'agit du titre principal. Cette information bénéficie :

- aux lecteurs d'écran ;
- aux moteurs de recherche ;
- aux outils d'extraction ;
- au CSS ;
- aux tests automatiques ;
- aux développeurs qui relisent le code.

La sémantique doit être choisie **avant** la présentation.

## 1.3 HTML forme un arbre DOM

Le navigateur analyse le document HTML et construit un arbre appelé **DOM** (*Document Object Model*).

Exemple :

```html
<body>
  <main>
    <h1>Bonjour</h1>
    <p>Bienvenue.</p>
  </main>
</body>
```

On peut le visualiser ainsi :

```text
body
└── main
    ├── h1
    │   └── "Bonjour"
    └── p
        └── "Bienvenue."
```

CSS et JavaScript travaillent ensuite sur cet arbre.

## 1.4 Un navigateur corrige parfois les erreurs

Le parseur HTML est volontairement tolérant.

Par exemple, un navigateur peut réussir à afficher un document mal structuré. Cela ne signifie pas que le document est correct.

Un HTML invalide peut produire :

- un DOM différent de celui attendu ;
- des comportements divergents ;
- des problèmes d'accessibilité ;
- des sélecteurs CSS ou JavaScript fragiles.

Il faut donc **valider le document**, même s'il « fonctionne dans mon navigateur ».

## 1.5 Environnement de travail

Pour débuter, il suffit de disposer :

- d'un éditeur comme [[Visual studio code]] ;
- d'un navigateur moderne ;
- des outils de développement du navigateur ;
- éventuellement d'un petit serveur HTTP local.

Un fichier peut être ouvert directement :

```text
index.html
```

Mais un serveur local devient préférable dès que l'on travaille avec :

- modules JavaScript ;
- requêtes réseau ;
- chemins absolus ;
- politiques de sécurité ;
- Service Workers.

Exemple simple avec Python :

```bash
python -m http.server 8000
```

Puis ouvrir :

```text
http://localhost:8000/
```

---

# 2. Structure d'un document HTML

## 2.1 Document minimal moderne

```html
<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Ma première page</title>
</head>
<body>
  <h1>Bonjour</h1>
  <p>Ma première page HTML.</p>
</body>
</html>
```

## 2.2 `<!doctype html>`

Le doctype moderne est :

```html
<!doctype html>
```

Il ne signifie pas « utiliser la version HTML5 ».

Son rôle pratique principal est de demander au navigateur le **mode standards**, plutôt qu'un ancien mode de compatibilité appelé *quirks mode*.

On le place tout au début du document.

## 2.3 `<html>`

L'élément `<html>` est la racine du document :

```html
<html lang="fr">
```

L'attribut `lang` est important pour :

- la synthèse vocale ;
- la prononciation ;
- les correcteurs ;
- la traduction ;
- l'indexation.

Exemples :

```html
<html lang="fr">
<html lang="en">
<html lang="fr-FR">
```

On peut aussi marquer un changement de langue local :

```html
<p>Le mot anglais <span lang="en">browser</span> signifie navigateur.</p>
```

## 2.4 `<head>`

Le `<head>` contient des informations sur le document qui ne font généralement pas partie du contenu affiché.

Exemple :

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Documentation du projet</title>
  <meta name="description" content="Documentation technique du projet.">
  <link rel="stylesheet" href="/assets/site.css">
</head>
```

## 2.5 `<body>`

Le `<body>` contient le contenu principal destiné à l'utilisateur :

```html
<body>
  <header>...</header>
  <main>...</main>
  <footer>...</footer>
</body>
```

## 2.6 Encodage UTF-8

On utilise :

```html
<meta charset="utf-8">
```

UTF-8 permet de représenter les caractères utilisés dans pratiquement toutes les langues.

Cette déclaration doit apparaître tôt dans le document.

## 2.7 Viewport mobile

La déclaration suivante est devenue standard dans les pages responsives :

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

Elle indique au navigateur mobile d'utiliser la largeur réelle du viewport CSS au lieu d'émuler une grande page de bureau.

---

# 3. Syntaxe, éléments et attributs

## 3.1 Éléments avec balise ouvrante et fermante

Exemple :

```html
<p>Un paragraphe.</p>
```

On distingue :

```text
<p>                balise ouvrante
Un paragraphe.     contenu
</p>               balise fermante
```

## 3.2 Éléments vides

Certains éléments ne contiennent pas de contenu HTML.

Exemples :

```html
<img src="photo.webp" alt="Montagne">
<br>
<hr>
<meta charset="utf-8">
<link rel="stylesheet" href="site.css">
<input type="text">
```

En syntaxe HTML, écrire :

```html
<img ... />
```

est toléré mais le `/` n'est pas nécessaire.

## 3.3 Imbrication

Les éléments doivent être correctement imbriqués :

```html
<p>Un texte avec <strong>une partie importante</strong>.</p>
```

Éviter :

```html
<p>Un texte <strong>mal imbriqué.</p></strong>
```

## 3.4 Attributs

Les attributs ajoutent des informations à un élément :

```html
<a href="/contact" class="bouton" hreflang="fr">Contact</a>
```

Ici :

- `href` définit la destination ;
- `class` permet notamment le ciblage CSS/JS ;
- `hreflang` renseigne la langue de la ressource liée.

## 3.5 Attributs booléens

Certains attributs sont booléens.

Exemple :

```html
<input type="email" required>
<button disabled>Envoyer</button>
<details open>
```

La présence de l'attribut signifie généralement `true`.

Écrire :

```html
disabled="false"
```

ne désactive pas le booléen : l'attribut est toujours présent.

## 3.6 Attributs globaux

Certains attributs peuvent être utilisés sur de nombreux éléments :

- `id` ;
- `class` ;
- `title` ;
- `lang` ;
- `dir` ;
- `hidden` ;
- `inert` ;
- `tabindex` ;
- `data-*`.

Exemple :

```html
<article id="article-42" class="article" data-id="42">
  ...
</article>
```

## 3.7 `id` et `class`

`id` identifie un élément de façon unique dans le document :

```html
<section id="contact">
```

`class` sert à regrouper plusieurs éléments :

```html
<p class="note">...</p>
<p class="note">...</p>
```

Éviter d'utiliser des classes uniquement pour simuler de la sémantique existante.

## 3.8 Attributs `data-*`

Pour associer des données applicatives à un élément :

```html
<button data-user-id="42">Ouvrir le profil</button>
```

JavaScript peut les lire via `dataset`.

Ils ne remplacent pas les attributs HTML sémantiques existants.

## 3.9 Entités HTML

Pour afficher certains caractères réservés :

```html
&lt;    <!-- < -->
&gt;    <!-- > -->
&amp;   <!-- & -->
&quot;  <!-- " -->
```

Exemple :

```html
<p>Pour écrire une balise : <code>&lt;main&gt;</code>.</p>
```

## 3.10 Commentaires

```html
<!-- Commentaire HTML -->
```

Ne jamais placer de secret dans un commentaire : il reste visible dans le code source envoyé au navigateur.

---

# 4. Texte, listes et contenu éditorial

## 4.1 Titres

HTML fournit six niveaux de titres :

```html
<h1>Titre principal</h1>
<h2>Chapitre</h2>
<h3>Sous-partie</h3>
```

On recherche une hiérarchie logique :

```text
h1
├── h2
│   ├── h3
│   └── h3
└── h2
```

On ne choisit pas `h4` parce que sa taille visuelle plaît davantage : la taille appartient au CSS.

Pour une page classique, un seul `<h1>` décrivant le contenu principal est une bonne pratique simple et robuste.

## 4.2 Paragraphes

```html
<p>
  Un paragraphe représente un bloc de texte cohérent.
</p>
```

Ne pas utiliser plusieurs `<br>` pour simuler des marges verticales.

Le placement relève du CSS.

## 4.3 Importance et emphase

```html
<strong>Important</strong>
<em>Emphase</em>
```

`<strong>` représente une forte importance.

`<em>` marque une emphase qui peut modifier le sens de la phrase.

Pour une simple décoration en gras ou italique, utiliser CSS lorsque la sémantique ne justifie pas ces éléments.

## 4.4 Contenu technique

```html
<p>Utilisez la commande <code>git status</code>.</p>
```

Bloc préformaté :

```html
<pre><code>git status
git diff
</code></pre>
```

Autres éléments utiles :

```html
<kbd>Ctrl</kbd> + <kbd>S</kbd>
<samp>Build successful</samp>
<var>x</var>
```

## 4.5 Citations

Citation courte :

```html
<p>Il a répondu <q>oui</q>.</p>
```

Citation longue :

```html
<blockquote cite="https://example.org/source">
  <p>Texte cité.</p>
</blockquote>
```

Titre d'une œuvre :

```html
<cite>Le Petit Prince</cite>
```

## 4.6 Dates et heures

```html
<time datetime="2026-08-29">29 août 2026</time>
```

Ou :

```html
<time datetime="2026-08-29T15:00:00+02:00">
  29 août à 15 h
</time>
```

La valeur `datetime` permet aux machines de comprendre l'information.

## 4.7 Abréviations

```html
<abbr title="HyperText Markup Language">HTML</abbr>
```

Ne pas dépendre uniquement de l'attribut `title` pour une information indispensable : son affichage est peu fiable sur mobile et au clavier.

## 4.8 Listes

Liste non ordonnée :

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
  <li>JavaScript</li>
</ul>
```

Liste ordonnée :

```html
<ol>
  <li>Cloner le dépôt</li>
  <li>Installer les dépendances</li>
  <li>Lancer les tests</li>
</ol>
```

Liste de définitions :

```html
<dl>
  <dt>HTML</dt>
  <dd>Langage de balisage du Web.</dd>

  <dt>CSS</dt>
  <dd>Langage de feuilles de style.</dd>
</dl>
```

## 4.9 Figures et légendes

```html
<figure>
  <img src="architecture.svg" alt="Architecture de l'application">
  <figcaption>Architecture logique du système.</figcaption>
</figure>
```

`<figure>` peut contenir autre chose qu'une image : extrait de code, diagramme, tableau, citation, etc.

## 4.10 Séparations thématiques

```html
<hr>
```

`<hr>` représente une rupture thématique.

Ce n'est pas simplement « une ligne horizontale décorative ».

---

# 5. Structure sémantique d'une page

## 5.1 Pourquoi la sémantique ?

Préférer :

```html
<header>
<nav>
<main>
<article>
<section>
<aside>
<footer>
```

à une succession indistincte de :

```html
<div>
```

Les `<div>` et `<span>` restent utiles lorsqu'aucun élément sémantique ne convient.

## 5.2 Exemple de structure

```html
<body>
  <header>
    <a href="/">Mon site</a>
    <nav aria-label="Navigation principale">
      <ul>
        <li><a href="/cours">Cours</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <article>
      <header>
        <h1>Comprendre HTML</h1>
        <p>
          Publié le
          <time datetime="2026-08-29">29 août 2026</time>
        </p>
      </header>

      <section>
        <h2>Introduction</h2>
        <p>...</p>
      </section>
    </article>
  </main>

  <footer>
    <p>&copy; 2026 Exemple</p>
  </footer>
</body>
```

## 5.3 `<header>`

`<header>` représente le contenu introductif d'une page ou d'une section.

Il peut donc exister plusieurs `<header>` dans un document.

## 5.4 `<nav>`

`<nav>` regroupe les liens de navigation importants.

Exemple :

```html
<nav aria-label="Documentation">
  ...
</nav>
```

Toutes les listes de liens ne doivent pas forcément être placées dans `<nav>`.

## 5.5 `<main>`

`<main>` représente le contenu principal unique du document.

Il permet notamment aux technologies d'assistance d'atteindre directement le contenu utile.

## 5.6 `<article>`

`<article>` convient à un contenu autonome qui pourrait être distribué ou réutilisé indépendamment :

- article de blog ;
- actualité ;
- message de forum ;
- fiche produit ;
- commentaire.

## 5.7 `<section>`

Une `<section>` représente une section thématique.

Elle devrait généralement posséder un titre :

```html
<section>
  <h2>Installation</h2>
  ...
</section>
```

Éviter de remplacer mécaniquement tous les `<div>` par des `<section>`.

## 5.8 `<aside>`

`<aside>` contient un contenu secondaire par rapport au contexte principal :

```html
<aside>
  <h2>À lire aussi</h2>
  ...
</aside>
```

## 5.9 `<footer>`

`<footer>` représente le pied d'une page ou d'une section.

Il peut contenir :

- auteur ;
- copyright ;
- liens connexes ;
- mentions légales ;
- date de mise à jour.

## 5.10 Hiérarchie des titres

Le navigateur ne construit pas automatiquement un plan de document fiable à partir des éléments `<section>`.

Il faut donc conserver une hiérarchie explicite :

```html
<h1>Guide</h1>

<section>
  <h2>Installation</h2>

  <section>
    <h3>Linux</h3>
  </section>
</section>
```

---

# 6. Liens et navigation

## 6.1 Lien simple

```html
<a href="/contact">Contact</a>
```

## 6.2 URL absolue et relative

URL absolue :

```html
<a href="https://example.org/docs">Documentation</a>
```

URL relative :

```html
<a href="../images/logo.svg">Logo</a>
```

Lien racine :

```html
<a href="/cours/html">Cours HTML</a>
```

Le choix dépend de l'organisation du site.

## 6.3 Ancres

```html
<nav>
  <a href="#formulaires">Formulaires</a>
</nav>

<section id="formulaires">
  <h2>Formulaires</h2>
</section>
```

L'`id` doit être unique dans le document.

## 6.4 Liens vers email et téléphone

```html
<a href="mailto:contact@example.org">Envoyer un email</a>
<a href="tel:+33102030405">01 02 03 04 05</a>
```

## 6.5 Téléchargement

```html
<a href="/documents/guide.pdf" download>
  Télécharger le guide
</a>
```

L'attribut `download` est soumis à certaines contraintes d'origine et de réponse HTTP.

## 6.6 Ouvrir un nouvel onglet

```html
<a
  href="https://example.org"
  target="_blank"
  rel="noopener"
>
  Site externe
</a>
```

Les navigateurs modernes traitent généralement `_blank` comme `noopener`, mais l'écrire explicitement peut rendre l'intention claire.

`noreferrer` ajoute une autre conséquence : ne pas envoyer l'information `Referer`.

Ne l'ajouter que lorsque cette confidentialité est réellement voulue.

## 6.7 Texte de lien

Éviter :

```html
<a href="/rapport">Cliquez ici</a>
```

Préférer :

```html
<a href="/rapport">Lire le rapport annuel 2026</a>
```

Le lien reste compréhensible hors contexte.

## 6.8 Navigation au clavier

Un lien possède naturellement le comportement clavier attendu.

Ne pas réimplémenter un lien avec :

```html
<div onclick="...">Contact</div>
```

Préférer l'élément natif `<a>`.

---

# 7. Images et médias

## 7.1 Image simple

```html
<img
  src="/images/montagne.webp"
  alt="Vue du Mont Blanc depuis Chamonix"
  width="1200"
  height="800"
>
```

## 7.2 L'attribut `alt`

Le texte alternatif dépend de la **fonction** de l'image, pas seulement de ce qu'elle représente.

Image informative :

```html
<img src="graphique.png" alt="Le trafic augmente de 20 % entre janvier et juin">
```

Image décorative :

```html
<img src="ornement.svg" alt="">
```

Le `alt=""` indique que l'image peut être ignorée par les technologies d'assistance.

Éviter de répéter « image de » ou « photo de » lorsque le contexte le rend évident.

## 7.3 Dimensions intrinsèques

Déclarer `width` et `height` aide le navigateur à réserver l'espace avant le téléchargement :

```html
<img
  src="photo.webp"
  alt="..."
  width="1600"
  height="900"
>
```

CSS peut ensuite rendre l'image responsive :

```css
img {
  max-width: 100%;
  height: auto;
}
```

Les dimensions HTML ne signifient donc pas forcément que l'image sera affichée à cette taille exacte.

## 7.4 Chargement différé

Pour une image hors écran :

```html
<img
  src="galerie-12.webp"
  alt="..."
  width="800"
  height="600"
  loading="lazy"
>
```

Ne pas appliquer systématiquement `loading="lazy"` à l'image principale visible immédiatement, notamment si elle contribue au LCP.

Pour une ressource très importante :

```html
<img
  src="/hero.avif"
  alt="..."
  width="1600"
  height="900"
  fetchpriority="high"
>
```

À utiliser avec parcimonie.

## 7.5 Formats d'images

| Format | Usage typique |
|---|---|
| JPEG | photographie |
| PNG | transparence, captures, besoin sans perte |
| WebP | photo ou illustration compressée |
| AVIF | compression moderne, souvent très efficace |
| SVG | graphiques vectoriels, icônes, schémas |
| GIF | animations simples historiques ; souvent moins efficace que vidéo/WebP/AVIF |

Le choix dépend :

- du type d'image ;
- de la qualité ;
- de la taille ;
- de la compatibilité ;
- du pipeline de production.

## 7.6 Images responsives avec `srcset`

```html
<img
  src="photo-800.webp"
  srcset="
    photo-480.webp 480w,
    photo-800.webp 800w,
    photo-1600.webp 1600w
  "
  sizes="(max-width: 700px) 100vw, 700px"
  alt="Village au bord d'un lac"
  width="1600"
  height="900"
>
```

Le navigateur choisit la ressource appropriée selon :

- le viewport ;
- la densité de pixels ;
- la taille prévue ;
- ses propres heuristiques.

## 7.7 `<picture>`

`<picture>` permet notamment :

- l'art direction ;
- plusieurs formats ;
- des variantes selon media query.

```html
<picture>
  <source
    media="(max-width: 600px)"
    srcset="/images/portrait-mobile.avif"
  >
  <source
    type="image/avif"
    srcset="/images/portrait.avif"
  >
  <img
    src="/images/portrait.webp"
    alt="Portrait de l'équipe"
    width="1200"
    height="800"
  >
</picture>
```

L'`alt` reste porté par `<img>`.

## 7.8 Audio

```html
<audio controls preload="metadata">
  <source src="/audio/interview.opus" type="audio/ogg; codecs=opus">
  <source src="/audio/interview.mp3" type="audio/mpeg">
  Votre navigateur ne peut pas lire cet audio.
</audio>
```

Pour un contenu important, fournir également une transcription.

## 7.9 Vidéo

```html
<video
  controls
  width="1280"
  height="720"
  preload="metadata"
  poster="/images/video-poster.webp"
>
  <source src="/video/demo.webm" type="video/webm">
  <source src="/video/demo.mp4" type="video/mp4">

  <track
    kind="captions"
    src="/video/demo-fr.vtt"
    srclang="fr"
    label="Français"
    default
  >
</video>
```

L'accessibilité peut nécessiter :

- sous-titres ;
- transcription ;
- audiodescription ;
- contrôle clavier.

## 7.10 `<figure>` et média

```html
<figure>
  <video controls>...</video>
  <figcaption>Démonstration de l'interface.</figcaption>
</figure>
```

---

# 8. Tableaux

## 8.1 Quand utiliser un tableau ?

Un tableau HTML sert à représenter des **données tabulaires**.

Il ne doit pas être utilisé pour mettre en page un site.

## 8.2 Tableau accessible

```html
<table>
  <caption>Résultats du semestre</caption>

  <thead>
    <tr>
      <th scope="col">Étudiant</th>
      <th scope="col">HTML</th>
      <th scope="col">CSS</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <th scope="row">Alice</th>
      <td>16</td>
      <td>15</td>
    </tr>
    <tr>
      <th scope="row">Bob</th>
      <td>14</td>
      <td>17</td>
    </tr>
  </tbody>
</table>
```

Éléments utiles :

- `<caption>` : nom du tableau ;
- `<thead>` : groupe d'en-têtes ;
- `<tbody>` : corps ;
- `<tfoot>` : pied ;
- `<th>` : cellule d'en-tête ;
- `scope="col"` / `scope="row"` : relation simple entre en-têtes et cellules.

## 8.3 Pas d'attribut `border`

Éviter :

```html
<table border="1">
```

La présentation appartient au CSS :

```css
table,
th,
td {
  border: 1px solid;
  border-collapse: collapse;
}
```

## 8.4 Fusion de cellules

```html
<td colspan="2">...</td>
<td rowspan="2">...</td>
```

Les tableaux complexes deviennent rapidement difficiles à rendre accessibles.

Lorsque c'est possible, préférer une structure plus simple.

## 8.5 Tableau responsive

HTML décrit les données ; la stratégie responsive relève principalement du CSS.

Éviter de transformer arbitrairement une table en blocs si cela détruit la relation entre les cellules et leurs en-têtes.

---

# 9. Formulaires

## 9.1 Structure minimale

```html
<form action="/inscription" method="post">
  <label for="email">Adresse email</label>
  <input
    id="email"
    name="email"
    type="email"
    autocomplete="email"
    required
  >

  <button type="submit">S'inscrire</button>
</form>
```

## 9.2 `action` et `method`

```html
<form action="/recherche" method="get">
```

GET convient naturellement à une recherche dont l'URL peut être partagée :

```text
/recherche?q=html
```

POST convient à l'envoi d'une opération qui modifie un état ou transporte des données dans le corps de la requête.

Le choix exact relève aussi de la conception HTTP : voir [[HTTP]].

## 9.3 `name` est essentiel

Un contrôle sans `name` n'est généralement pas inclus dans la soumission classique du formulaire.

```html
<input id="email" name="email" type="email">
```

`id` et `name` ont donc des rôles différents.

## 9.4 Labels

Association explicite :

```html
<label for="prenom">Prénom</label>
<input id="prenom" name="prenom">
```

Ou association implicite :

```html
<label>
  Prénom
  <input name="prenom">
</label>
```

Éviter de remplacer le label par un `placeholder`.

Le placeholder est un exemple ou une indication secondaire, pas un libellé durable.

## 9.5 Types de `<input>`

Exemples :

```html
<input type="text">
<input type="email">
<input type="password">
<input type="url">
<input type="tel">
<input type="number">
<input type="date">
<input type="time">
<input type="datetime-local">
<input type="search">
<input type="color">
<input type="file">
<input type="checkbox">
<input type="radio">
<input type="range">
```

Choisir le type approprié fournit souvent :

- une interface adaptée ;
- un clavier mobile approprié ;
- une validation native ;
- une meilleure sémantique.

## 9.6 `inputmode`

Pour demander un clavier adapté sans changer la sémantique du champ :

```html
<input
  name="code"
  type="text"
  inputmode="numeric"
  autocomplete="one-time-code"
>
```

Un code postal ou un identifiant numérique n'est pas forcément un nombre mathématique : `type="number"` n'est donc pas toujours le bon choix.

## 9.7 Autocomplétion

Exemples :

```html
<input name="given-name" autocomplete="given-name">
<input name="family-name" autocomplete="family-name">
<input name="email" autocomplete="email">
<input name="street-address" autocomplete="street-address">
<input name="new-password" autocomplete="new-password">
```

Des valeurs d'autocomplétion précises améliorent :

- l'expérience utilisateur ;
- l'accessibilité cognitive ;
- la rapidité de saisie.

## 9.8 Contraintes natives

```html
<input
  type="text"
  name="username"
  required
  minlength="3"
  maxlength="30"
  pattern="[A-Za-z0-9_-]+"
>
```

Attributs fréquents :

- `required` ;
- `minlength` / `maxlength` ;
- `min` / `max` ;
- `step` ;
- `pattern`.

> [!warning] Validation client ≠ sécurité
> La validation HTML améliore l'expérience utilisateur, mais un client peut envoyer directement n'importe quelle requête HTTP. **Toute donnée doit être validée côté serveur.**

## 9.9 `disabled` et `readonly`

```html
<input name="code" value="42" readonly>
```

`readonly` :

- reste généralement focalisable ;
- est soumis avec le formulaire.

```html
<input name="code" value="42" disabled>
```

`disabled` :

- n'est pas interactif ;
- n'est généralement **pas soumis**.

Cette différence est importante.

## 9.10 Checkbox

```html
<label>
  <input type="checkbox" name="newsletter" value="yes">
  Recevoir la newsletter
</label>
```

## 9.11 Radio

```html
<fieldset>
  <legend>Format préféré</legend>

  <label>
    <input type="radio" name="format" value="html">
    HTML
  </label>

  <label>
    <input type="radio" name="format" value="pdf">
    PDF
  </label>
</fieldset>
```

Les boutons radio d'un même groupe partagent le même `name`.

## 9.12 `<fieldset>` et `<legend>`

Ils permettent de grouper sémantiquement des contrôles :

```html
<fieldset>
  <legend>Adresse de livraison</legend>
  ...
</fieldset>
```

Très utile pour :

- radio buttons ;
- groupes de checkboxes ;
- parties d'un long formulaire.

## 9.13 `<select>`

```html
<label for="pays">Pays</label>
<select id="pays" name="pays">
  <option value="">Choisir un pays</option>
  <option value="fr">France</option>
  <option value="be">Belgique</option>
</select>
```

Groupes :

```html
<select name="ville">
  <optgroup label="France">
    <option>Lille</option>
    <option>Toulouse</option>
  </optgroup>
</select>
```

## 9.14 `<textarea>`

```html
<label for="message">Message</label>
<textarea
  id="message"
  name="message"
  rows="8"
  maxlength="2000"
></textarea>
```

## 9.15 Boutons

```html
<button type="submit">Envoyer</button>
<button type="reset">Réinitialiser</button>
<button type="button">Ouvrir l'aide</button>
```

Dans un formulaire, un `<button>` sans `type` agit par défaut comme un bouton de soumission.

Il est souvent préférable d'écrire explicitement son type.

## 9.16 Upload de fichier

```html
<form method="post" enctype="multipart/form-data">
  <label for="avatar">Avatar</label>
  <input
    id="avatar"
    name="avatar"
    type="file"
    accept="image/png,image/jpeg,image/webp"
  >

  <button type="submit">Envoyer</button>
</form>
```

`accept` aide l'utilisateur à choisir le bon fichier mais **ne constitue pas une validation de sécurité**.

Le serveur doit contrôler :

- taille ;
- type réel ;
- contenu ;
- nom ;
- stockage ;
- droits d'accès.

## 9.17 `datalist`

```html
<label for="navigateur">Navigateur</label>
<input id="navigateur" name="navigateur" list="navigateurs">

<datalist id="navigateurs">
  <option value="Firefox">
  <option value="Chromium">
  <option value="Safari">
</datalist>
```

`datalist` fournit des suggestions sans imposer forcément l'une des valeurs.

## 9.18 `output`, `progress` et `meter`

Résultat de calcul :

```html
<output name="total">42</output>
```

Progression :

```html
<progress value="60" max="100">60 %</progress>
```

Mesure dans une plage connue :

```html
<meter min="0" max="100" value="73">73 / 100</meter>
```

`progress` et `meter` n'ont pas le même sens.

## 9.19 Erreurs de formulaire accessibles

Un message d'erreur doit :

- être compréhensible ;
- identifier le champ ;
- expliquer comment corriger ;
- ne pas reposer uniquement sur la couleur.

Exemple conceptuel :

```html
<label for="email">Email</label>
<input
  id="email"
  name="email"
  type="email"
  aria-describedby="email-aide email-erreur"
  aria-invalid="true"
>

<p id="email-aide">Adresse utilisée pour confirmer le compte.</p>
<p id="email-erreur">Saisissez une adresse email valide.</p>
```

ARIA ne remplace pas une structure HTML correcte ; on l'utilise lorsqu'elle apporte une information qui n'existe pas nativement.

---

# 10. Métadonnées et `<head>`

## 10.1 Titre

```html
<title>Cours HTML — Notes</title>
```

Chaque page devrait avoir un titre :

- spécifique ;
- descriptif ;
- concis.

## 10.2 Description

```html
<meta
  name="description"
  content="Cours complet de HTML moderne : sémantique, accessibilité, formulaires et performance."
>
```

La description peut être utilisée comme signal ou extrait par certains moteurs, sans garantie d'affichage identique.

## 10.3 Favicon

```html
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
```

On peut prévoir plusieurs formats selon les besoins.

## 10.4 Feuille de style

```html
<link rel="stylesheet" href="/assets/site.css">
```

## 10.5 URL canonique

```html
<link rel="canonical" href="https://example.org/cours/html">
```

Voir [[SEO]] pour les implications.

## 10.6 Métadonnées sociales

Open Graph n'est pas une fonctionnalité normative de HTML, mais il est très répandu :

```html
<meta property="og:title" content="Cours HTML">
<meta property="og:type" content="article">
<meta property="og:url" content="https://example.org/cours/html">
<meta property="og:image" content="https://example.org/images/html-course.webp">
```

Ces métadonnées servent aux plateformes qui les reconnaissent.

## 10.7 `robots`

```html
<meta name="robots" content="index,follow">
```

Ou par exemple :

```html
<meta name="robots" content="noindex">
```

Le contrôle de l'indexation est un sujet SEO et ne doit pas être confondu avec la sécurité ou le contrôle d'accès.

## 10.8 `base`

```html
<base href="https://example.org/docs/">
```

`<base>` modifie la résolution de toutes les URL relatives du document.

C'est puissant mais potentiellement source de confusion. À utiliser seulement si le besoin est clair.

## 10.9 Métadonnées HTTP dans HTML

Certaines informations peuvent être représentées avec `http-equiv`, mais les **vrais en-têtes HTTP** sont généralement préférables lorsqu'ils sont disponibles.

Exemple :

```html
<meta http-equiv="refresh" content="5; url=/nouvelle-page">
```

Pour une redirection normale, préférer une redirection HTTP côté serveur.

---

# 11. Accessibilité

## 11.1 Principe : HTML natif d'abord

Le premier réflexe d'accessibilité est d'utiliser le bon élément HTML.

Préférer :

```html
<button type="button">Supprimer</button>
```

à :

```html
<div role="button" tabindex="0">Supprimer</div>
```

Le bouton natif apporte déjà :

- le focus ;
- l'activation clavier ;
- la sémantique ;
- les états ;
- l'intégration avec les technologies d'assistance.

## 11.2 Langue

```html
<html lang="fr">
```

et :

```html
<blockquote lang="en">
  ...
</blockquote>
```

## 11.3 Titres

La hiérarchie des titres doit représenter celle du document.

Éviter les sauts arbitraires.

## 11.4 Landmarks

Les éléments sémantiques fournissent naturellement des repères :

- `<header>` ;
- `<nav>` ;
- `<main>` ;
- `<aside>` ;
- `<footer>`.

## 11.5 Lien d'évitement

```html
<a class="skip-link" href="#contenu">
  Aller au contenu principal
</a>

...

<main id="contenu">
  ...
</main>
```

Cela permet à un utilisateur clavier de passer la navigation répétitive.

## 11.6 Images

Toujours décider consciemment si l'image est :

- informative ;
- fonctionnelle ;
- décorative ;
- complexe.

Une image complexe, comme un graphique, peut nécessiter une description détaillée dans le texte.

## 11.7 Formulaires

Associer :

- labels ;
- groupes ;
- indications ;
- erreurs.

Ne pas utiliser uniquement le placeholder.

## 11.8 Clavier

Tout élément interactif doit être utilisable au clavier.

Ne pas ajouter arbitrairement :

```html
tabindex="1"
tabindex="2"
```

Les valeurs positives créent un ordre de focus artificiel difficile à maintenir.

`tabindex="0"` et `tabindex="-1"` ont des usages spécifiques, mais l'élément natif approprié est souvent préférable.

## 11.9 `hidden`

```html
<section hidden>
  ...
</section>
```

Un élément `hidden` n'est pas rendu comme contenu normal.

Ne l'utiliser que si le contenu doit réellement être absent de l'interface à cet instant.

## 11.10 `inert`

```html
<main inert>
  ...
</main>
```

`inert` rend un sous-arbre non interactif et le retire notamment de l'ordre de tabulation et de l'arbre d'accessibilité.

C'est utile dans certains composants complexes, mais il faut comprendre l'impact avant de l'utiliser.

## 11.11 ARIA

Règle pratique :

> **No ARIA is better than bad ARIA.**

ARIA sert à enrichir l'accessibilité lorsque le HTML natif ne suffit pas.

Exemples raisonnables :

```html
<nav aria-label="Navigation secondaire">
```

```html
<button aria-expanded="false" aria-controls="menu">
```

Éviter d'ajouter des rôles redondants sans raison :

```html
<button role="button">
```

## 11.12 Tester réellement

Un audit automatique ne suffit pas.

Tester :

1. navigation au clavier ;
2. ordre du focus ;
3. zoom ;
4. contraste ;
5. lecteur d'écran si le contexte le nécessite ;
6. contenus sans CSS ;
7. erreurs de formulaire.

---

# 12. HTML, CSS et JavaScript

## 12.1 CSS externe

```html
<link rel="stylesheet" href="/assets/site.css">
```

C'est généralement préférable aux styles en ligne.

## 12.2 JavaScript classique

```html
<script src="/assets/app.js" defer></script>
```

`defer` permet au navigateur de télécharger le script en parallèle et de l'exécuter après l'analyse du document, dans l'ordre des scripts différés.

## 12.3 Modules JavaScript

```html
<script type="module" src="/assets/app.js"></script>
```

Les modules ont notamment :

- leur propre portée ;
- `import` / `export` ;
- un chargement différé par nature dans le contexte classique de la page.

Voir [[Javascript]].

## 12.4 `async`

```html
<script src="/analytics.js" async></script>
```

`async` convient aux scripts indépendants qui peuvent s'exécuter dès qu'ils sont prêts.

Il ne faut pas compter sur l'ordre relatif entre plusieurs scripts `async`.

## 12.5 Scripts inline et sécurité

```html
<script>
  // ...
</script>
```

Les scripts inline peuvent compliquer une Content Security Policy stricte.

Pour les applications sensibles, penser ensemble :

- architecture JS ;
- CSP ;
- nonces/hashes ;
- dépendances externes.

## 12.6 Progressivité

Le HTML devrait fournir autant que possible une base utile même si JavaScript échoue.

Exemple :

```html
<details>
  <summary>Voir les détails</summary>
  <p>Contenu...</p>
</details>
```

Ici, aucune bibliothèque JavaScript n'est nécessaire.

---

# 13. Éléments interactifs natifs

HTML fournit aujourd'hui plusieurs mécanismes interactifs qui évitent de recréer des composants basiques en JavaScript.

## 13.1 `<details>` et `<summary>`

```html
<details>
  <summary>Afficher la configuration</summary>
  <pre><code>...</code></pre>
</details>
```

Ouvert par défaut :

```html
<details open>
```

## 13.2 `<dialog>`

```html
<dialog id="confirmation">
  <h2>Supprimer le fichier ?</h2>
  <form method="dialog">
    <button value="cancel">Annuler</button>
    <button value="confirm">Supprimer</button>
  </form>
</dialog>
```

JavaScript peut ouvrir un dialogue modal :

```js
document.querySelector("#confirmation").showModal();
```

`<dialog>` apporte des comportements natifs importants, mais il reste nécessaire de concevoir correctement :

- le titre ;
- le focus ;
- les actions ;
- la fermeture ;
- le sens de la boîte de dialogue.

## 13.3 Popover

Le mécanisme `popover` permet de créer certains contenus superposés sans construire entièrement la gestion d'ouverture/fermeture en JavaScript.

```html
<button popovertarget="aide">
  Aide
</button>

<div id="aide" popover>
  <p>Voici l'aide contextuelle.</p>
</div>
```

Le navigateur gère notamment une partie :

- de l'affichage ;
- de la fermeture légère ;
- de la couche supérieure.

Pour les composants avancés, vérifier les besoins d'accessibilité et la compatibilité cible.

## 13.4 Commandes déclaratives

Le HTML évolue vers davantage d'interactions déclaratives.

Il faut cependant distinguer :

- ce qui appartient au Living Standard ;
- ce qui est implémenté dans les navigateurs ciblés ;
- ce qui reste expérimental.

Toujours vérifier la compatibilité réelle avant d'utiliser une fonctionnalité récente en production.

## 13.5 `inert`

`inert` peut compléter la gestion d'interfaces où une partie du document ne doit temporairement plus être interactive.

Ne pas l'utiliser comme simple technique visuelle : cela affecte aussi l'accessibilité.

---

# 14. Intégration de contenus externes

## 14.1 `<iframe>`

Exemple :

```html
<iframe
  src="https://example.org/widget"
  title="Carte des points de vente"
  width="800"
  height="600"
  loading="lazy"
></iframe>
```

L'attribut `title` décrit le contenu de l'iframe pour les technologies d'assistance.

## 14.2 Pas de `frameborder`

Ancien exemple :

```html
<iframe frameborder="0">
```

`frameborder` est obsolète.

La présentation se gère avec CSS.

## 14.3 `sandbox`

Pour réduire les capacités d'une iframe :

```html
<iframe
  src="https://widgets.example.org/"
  sandbox
  title="Aperçu externe"
></iframe>
```

On peut autoriser explicitement certaines capacités :

```html
sandbox="allow-scripts allow-forms"
```

Chaque permission augmente la surface disponible.

Principe :

> Autoriser le minimum nécessaire.

## 14.4 `allow`

Certaines fonctionnalités peuvent être déléguées à une iframe :

```html
<iframe
  src="https://video.example.org/embed/42"
  allow="fullscreen"
  title="Présentation du projet"
></iframe>
```

La liste dépend de la Permissions Policy et du service utilisé.

## 14.5 `referrerpolicy`

```html
<iframe
  src="https://example.org/embed"
  referrerpolicy="strict-origin-when-cross-origin"
  title="Contenu externe"
></iframe>
```

## 14.6 Contenu tiers

Intégrer un widget tiers peut affecter :

- confidentialité ;
- cookies ;
- sécurité ;
- performance ;
- accessibilité ;
- conformité juridique.

L'iframe n'est donc pas seulement une question de mise en page.

---

# 15. Performance et chargement

## 15.1 Le HTML influence les performances

Avant même le CSS et JavaScript, le document HTML détermine :

- quelles ressources sont découvertes ;
- dans quel ordre ;
- si elles bloquent le rendu ;
- comment les images sont dimensionnées ;
- quels scripts s'exécutent.

## 15.2 Éviter les ressources inutiles

Ne pas charger :

- plusieurs bibliothèques pour le même besoin ;
- des polices inutilisées ;
- des images beaucoup trop grandes ;
- des scripts non nécessaires sur toutes les pages.

## 15.3 `preload`

Pour une ressource réellement critique découverte trop tard :

```html
<link
  rel="preload"
  href="/fonts/inter.woff2"
  as="font"
  type="font/woff2"
  crossorigin
>
```

Le preload doit rester sélectif.

Précharger trop de ressources peut nuire aux performances.

## 15.4 `preconnect`

```html
<link rel="preconnect" href="https://cdn.example.org">
```

Utile seulement pour des origines critiques réellement utilisées tôt.

## 15.5 `dns-prefetch`

```html
<link rel="dns-prefetch" href="//cdn.example.org">
```

C'est une optimisation plus légère que `preconnect`.

## 15.6 Images

Bonnes pratiques :

- format adapté ;
- dimensions intrinsèques ;
- `srcset` / `sizes` ;
- lazy loading hors viewport ;
- ne pas lazy-loader l'image principale ;
- compression adéquate.

## 15.7 Scripts

Pour un script classique de l'application :

```html
<script src="/app.js" defer></script>
```

Pour un module :

```html
<script type="module" src="/app.js"></script>
```

Éviter un gros script bloquant dans le `<head>` sans nécessité.

## 15.8 CSS critique

La feuille de style principale découverte tôt :

```html
<link rel="stylesheet" href="/site.css">
```

L'architecture CSS, le fractionnement et les stratégies critiques doivent être mesurés, pas appliqués comme des recettes universelles.

## 15.9 Core Web Vitals

HTML peut influencer :

- **LCP** : découverte et priorité de la ressource principale ;
- **CLS** : dimensions réservées pour images/iframes ;
- **INP** : quantité de JS et complexité de l'interface.

Voir [[SEO]] pour le suivi.

---

# 16. Sécurité

## 16.1 HTML n'est pas une barrière de sécurité

Les attributs HTML améliorent l'interface mais ne remplacent jamais les contrôles serveur.

Exemple :

```html
<input type="number" min="1" max="10">
```

Un attaquant peut envoyer directement :

```text
quantity=1000000
```

Le serveur doit donc valider.

## 16.2 XSS et échappement

Un principe essentiel :

> Une donnée non fiable ne doit jamais être injectée comme HTML sans traitement adapté au contexte.

Exemple dangereux côté serveur :

```html
<p>Bonjour {{ contenu_non_echappe }}</p>
```

Un moteur de template doit normalement échapper le contenu texte.

Le contexte compte :

- texte HTML ;
- attribut ;
- URL ;
- JavaScript ;
- CSS.

Il n'existe pas une fonction d'échappement universelle valable partout.

## 16.3 Ne pas mettre de secrets dans HTML

Tout contenu envoyé au navigateur peut être lu par l'utilisateur.

Ne jamais placer dans HTML :

- clé API privée ;
- mot de passe ;
- secret OAuth ;
- clé de signature ;
- token administrateur.

Même si l'élément est :

```html
hidden
```

ou :

```css
display: none;
```

le secret est toujours transmis.

## 16.4 CSP

Une **Content Security Policy** permet de limiter les sources de scripts, styles et autres ressources.

Elle est généralement mieux envoyée en **en-tête HTTP**.

Une version via HTML existe pour certains usages :

```html
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; img-src 'self' https: data:"
>
```

Mais toutes les directives ne sont pas équivalentes dans une meta CSP et la configuration serveur reste à privilégier.

## 16.5 Subresource Integrity

Pour une ressource externe immuable :

```html
<script
  src="https://cdn.example.org/library.js"
  integrity="sha384-..."
  crossorigin="anonymous"
></script>
```

`integrity` permet au navigateur de vérifier le contenu attendu.

## 16.6 Liens externes

Pour un lien `_blank`, comprendre :

- `noopener` ;
- `noreferrer` ;
- politique de referrer ;
- origine du site cible.

## 16.7 Iframes

Utiliser :

- `sandbox` ;
- `allow` ;
- une origine de confiance ;
- une politique de referrer adaptée.

## 16.8 Uploads

`accept` n'est pas une protection.

Le serveur doit inspecter le contenu et stocker les fichiers de façon sûre.

## 16.9 CSRF

Un formulaire HTML peut déclencher une requête authentifiée par cookie.

Les opérations sensibles doivent intégrer une stratégie de protection CSRF adaptée à l'architecture applicative.

Voir [[Sécurité avec Python]] et [[OAuth OpenID]].

---

# 17. SEO et partage

Voir le cours détaillé [[SEO]].

## 17.1 Structure lisible

Un HTML sémantique aide :

- les utilisateurs ;
- les crawlers ;
- les outils d'extraction.

Utiliser de vrais titres et de vrais liens.

## 17.2 `<title>`

```html
<title>Installer PostgreSQL sur Ubuntu — Documentation</title>
```

Éviter :

```html
<title>Accueil</title>
```

sur toutes les pages.

## 17.3 Description

```html
<meta
  name="description"
  content="Procédure d'installation et de sécurisation de PostgreSQL sur Ubuntu."
>
```

## 17.4 Canonical

```html
<link rel="canonical" href="https://example.org/docs/postgresql">
```

## 17.5 Liens crawlables

Un lien normal :

```html
<a href="/docs/html">HTML</a>
```

est généralement préférable à une navigation uniquement déclenchée par un gestionnaire JavaScript.

## 17.6 Texte alternatif

`alt` sert d'abord à l'accessibilité.

Il contribue également à la compréhension du contenu par les systèmes automatiques.

Éviter le bourrage de mots-clés.

## 17.7 Données structurées

Les données structurées Schema.org sont souvent ajoutées en JSON-LD :

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Comprendre HTML",
  "datePublished": "2026-08-29"
}
</script>
```

Le balisage doit décrire fidèlement le contenu visible.

## 17.8 Open Graph

Exemple :

```html
<meta property="og:title" content="Comprendre HTML">
<meta property="og:description" content="Cours complet de HTML moderne">
<meta property="og:image" content="https://example.org/html.webp">
```

Cela concerne surtout l'aperçu lors du partage, pas le classement SEO en lui-même.

---

# 18. Validation, qualité et maintenance

## 18.1 Validation HTML

Le validateur aide à repérer :

- balises mal imbriquées ;
- attributs invalides ;
- éléments obsolètes ;
- erreurs de structure ;
- duplications d'`id`.

Utiliser notamment le **Nu Html Checker** :

```text
https://validator.w3.org/nu/
```

## 18.2 Le navigateur n'est pas un validateur

« Ça s'affiche » ne prouve pas que le code est conforme.

Le parseur HTML récupère volontairement de nombreuses erreurs.

## 18.3 Inspecter le DOM réel

Les DevTools permettent d'observer le DOM produit par le parseur.

C'est utile lorsqu'une balise mal placée provoque une réorganisation inattendue.

## 18.4 HTML lisible

Préférer :

```html
<article>
  <h2>Titre</h2>
  <p>Texte...</p>
</article>
```

à :

```html
<article><h2>Titre</h2><p>Texte...</p></article>
```

pour du code maintenu manuellement.

## 18.5 Indentation

Choisir une convention et la conserver.

Exemple :

```html
<ul>
  <li>Premier</li>
  <li>Deuxième</li>
</ul>
```

## 18.6 Classes stables

Éviter les classes dépendantes du contenu visuel :

```html
<div class="texte-rouge-gauche">
```

Préférer un nom correspondant au rôle :

```html
<div class="alerte">
```

## 18.7 Éviter les éléments obsolètes

Ne pas utiliser pour la mise en forme :

```html
<font>
<center>
<big>
<strike>
```

Ne pas utiliser non plus des attributs de présentation historiques comme :

```html
<table border="1">
<iframe frameborder="0">
<body bgcolor="...">
```

Le CSS remplit ce rôle.

## 18.8 Tester sans CSS

Désactiver temporairement CSS aide à vérifier si le document conserve une logique lisible.

Un bon ordre source bénéficie aussi :

- aux lecteurs d'écran ;
- au clavier ;
- aux petits écrans ;
- au référencement.

## 18.9 Tester sans JavaScript

La page doit conserver autant que possible une base utile.

Cela révèle souvent une dépendance excessive au JavaScript.

## 18.10 Automatiser

Dans un projet important, on peut intégrer :

- validation HTML ;
- lint ;
- tests d'accessibilité ;
- tests end-to-end ;
- contrôle de liens.

Exemples d'outils selon le contexte :

- Nu Html Checker ;
- HTMLHint ;
- axe-core ;
- Lighthouse ;
- Playwright.

Un outil automatique ne remplace pas une revue humaine.

---

# 19. Web Components et HTML avancé

Les Web Components regroupent plusieurs primitives de la plateforme :

- Custom Elements ;
- Shadow DOM ;
- templates ;
- slots.

Une partie nécessite JavaScript, mais HTML contient des briques essentielles.

## 19.1 `<template>`

```html
<template id="carte-utilisateur">
  <article class="user-card">
    <h2></h2>
    <p></p>
  </article>
</template>
```

Le contenu d'un `<template>` n'est pas rendu comme du contenu normal tant qu'il n'est pas instancié.

## 19.2 `<slot>`

Dans un Shadow DOM :

```html
<slot name="titre"></slot>
<slot></slot>
```

Le document peut fournir :

```html
<mon-composant>
  <h2 slot="titre">Titre</h2>
  <p>Contenu principal</p>
</mon-composant>
```

## 19.3 Custom Elements

Un élément personnalisé doit généralement contenir un tiret :

```html
<user-card></user-card>
```

Son comportement est enregistré via JavaScript.

Voir [[Javascript]].

## 19.4 Shadow DOM déclaratif

Le HTML moderne permet aussi, dans les environnements qui le prennent en charge, de déclarer un Shadow Root :

```html
<my-card>
  <template shadowrootmode="open">
    <style>
      :host {
        display: block;
      }
    </style>

    <slot></slot>
  </template>

  <p>Contenu distribué.</p>
</my-card>
```

Avant d'utiliser cette fonctionnalité, vérifier la compatibilité requise par le projet.

## 19.5 Progressive enhancement

Un composant personnalisé devrait si possible laisser un contenu ou un comportement utile avant l'activation du JavaScript.

C'est particulièrement important pour :

- navigation ;
- formulaires ;
- contenu éditorial ;
- actions essentielles.

---

# 20. Projet final

## 20.1 Objectif

Créer un petit site de documentation ou de présentation composé d'au moins :

```text
site/
├── index.html
├── cours.html
├── contact.html
├── assets/
│   ├── site.css
│   ├── app.js
│   └── images/
└── README.md
```

## 20.2 Contraintes

Le site doit comporter :

- un HTML valide ;
- une langue déclarée ;
- un `<title>` unique par page ;
- une description ;
- une navigation cohérente ;
- un `<main>` ;
- une hiérarchie de titres ;
- au moins un article structuré ;
- une image responsive ;
- un tableau accessible ;
- un formulaire ;
- une validation HTML ;
- un média ou une iframe sécurisée ;
- un élément natif interactif (`details`, `dialog` ou popover) ;
- un favicon ;
- une feuille CSS externe ;
- un script JS facultatif mais proprement chargé ;
- un lien d'évitement ;
- une page utilisable au clavier.

## 20.3 Formulaire

Le formulaire doit inclure :

- labels ;
- `name` ;
- `autocomplete` pertinent ;
- types appropriés ;
- contraintes natives ;
- `fieldset` lorsque nécessaire ;
- message explicatif sur la validation serveur.

## 20.4 Images

Au moins une image doit utiliser :

- `alt` adapté ;
- `width` / `height` ;
- WebP ou AVIF ;
- `srcset` ou `<picture>`.

## 20.5 Sécurité

Si le site contient une iframe :

- `title` ;
- permissions minimales ;
- `sandbox` si compatible avec le service.

Aucun secret ne doit être présent dans :

- HTML ;
- JavaScript côté client ;
- attributs `data-*` ;
- commentaires.

## 20.6 Évaluation

Barème possible :

| Critère | Points |
|---|---:|
| Structure et validité HTML | 4 |
| Sémantique | 4 |
| Accessibilité | 4 |
| Formulaires | 2 |
| Images et médias | 2 |
| Performance | 1 |
| Sécurité | 1 |
| Maintenabilité | 2 |
| **Total** | **20** |

---

# 21. Travaux pratiques

## TP 1 — Première page

Créer :

```text
tp1/
└── index.html
```

Contraintes :

- doctype ;
- `lang="fr"` ;
- charset UTF-8 ;
- viewport ;
- titre ;
- `<h1>` ;
- trois paragraphes ;
- une liste.

Valider le fichier.

## TP 2 — CV sémantique

Créer un CV HTML contenant :

- `<header>` ;
- `<main>` ;
- plusieurs `<section>` ;
- `<address>` pour les coordonnées pertinentes ;
- `<time>` pour les dates ;
- listes de compétences.

Le CSS n'est pas prioritaire : l'objectif est la structure.

## TP 3 — Navigation multi-pages

Créer :

```text
index.html
apropos.html
contact.html
```

Ajouter :

- navigation identique ;
- lien vers la page active avec `aria-current="page"` ;
- liens relatifs corrects ;
- ancre interne.

Exemple :

```html
<a href="/cours" aria-current="page">Cours</a>
```

## TP 4 — Images responsives

Créer une page galerie avec :

- dimensions intrinsèques ;
- `alt` adapté ;
- lazy loading pour les images hors écran ;
- `srcset` ;
- `<picture>` pour une image avec art direction.

Observer dans l'onglet Network la ressource choisie.

## TP 5 — Tableau accessible

Créer un tableau de planning avec :

- `<caption>` ;
- `<thead>` ;
- `<tbody>` ;
- `scope` ;
- aucune mise en page via attribut HTML.

Tester la lecture logique du tableau.

## TP 6 — Formulaire d'inscription

Construire un formulaire avec :

- prénom ;
- nom ;
- email ;
- mot de passe ;
- date de naissance ;
- choix de langue ;
- préférences sous forme de checkboxes ;
- bouton submit.

Ajouter :

- labels ;
- autocomplete ;
- contraintes natives ;
- `fieldset` / `legend`.

Puis expliquer quelles validations doivent encore être faites côté serveur.

## TP 7 — Article multimédia

Créer un article comprenant :

- `<article>` ;
- `<header>` ;
- auteur ;
- date avec `<time>` ;
- image `<figure>` ;
- vidéo avec sous-titres ;
- section « sources ».

## TP 8 — Accessibilité

À partir d'une page volontairement mauvaise :

```html
<div onclick="...">Envoyer</div>
<img src="important.png">
<input placeholder="Email">
<div class="titre">Inscription</div>
```

Corriger avec :

- vrais boutons ;
- alt ;
- label ;
- vraie hiérarchie de titres.

Tester entièrement au clavier.

## TP 9 — HTML interactif natif

Créer une FAQ avec :

```html
<details>
  <summary>Question</summary>
  <p>Réponse.</p>
</details>
```

Ajouter ensuite :

- un `<dialog>` ;
- un popover ;
- une zone rendue `inert` dans un scénario justifié.

Comparer ce code avec une réimplémentation JavaScript complète.

## TP 10 — Audit final

Auditer le projet final :

1. Nu Html Checker ;
2. navigation clavier ;
3. DevTools ;
4. Lighthouse ou outil équivalent ;
5. test sans CSS ;
6. test sans JavaScript ;
7. inspection du réseau ;
8. recherche de secrets ;
9. contrôle des iframes ;
10. vérification des métadonnées.

Produire un court rapport :

```markdown
# Audit

## Erreurs corrigées

## Accessibilité

## Performance

## Sécurité

## Limites restantes
```

---

# 22. Checklist

## Document

- [ ] `<!doctype html>` présent
- [ ] `<html lang="...">`
- [ ] charset UTF-8
- [ ] viewport adapté
- [ ] `<title>` spécifique
- [ ] description pertinente

## Structure

- [ ] un `<main>` pour le contenu principal
- [ ] hiérarchie de titres cohérente
- [ ] éléments sémantiques utilisés lorsqu'ils conviennent
- [ ] pas de `<div>` interactif simulant inutilement un bouton ou un lien

## Liens

- [ ] texte de lien compréhensible
- [ ] ancres uniques
- [ ] `_blank` utilisé seulement lorsqu'il est pertinent
- [ ] politique `rel` comprise

## Images

- [ ] `alt` décidé consciemment
- [ ] `width` et `height`
- [ ] format adapté
- [ ] responsive images si nécessaire
- [ ] lazy loading seulement hors contenu critique

## Tableaux

- [ ] utilisés pour des données, pas pour la mise en page
- [ ] `<caption>` pertinent
- [ ] en-têtes `<th>`
- [ ] `scope` pour les relations simples

## Formulaires

- [ ] tous les champs utiles ont un `name`
- [ ] labels associés
- [ ] types appropriés
- [ ] autocomplete
- [ ] groupes avec `<fieldset>` / `<legend>`
- [ ] boutons avec `type`
- [ ] erreurs compréhensibles
- [ ] validation serveur prévue

## Accessibilité

- [ ] page utilisable au clavier
- [ ] focus visible via CSS
- [ ] lien d'évitement si nécessaire
- [ ] pas d'ordre `tabindex` positif arbitraire
- [ ] ARIA utilisée seulement lorsque nécessaire
- [ ] contenu multimédia accessible

## Sécurité

- [ ] aucun secret côté client
- [ ] données non fiables échappées correctement
- [ ] iframe limitée
- [ ] uploads validés serveur
- [ ] CSP envisagée
- [ ] dépendances externes justifiées

## Performance

- [ ] images optimisées
- [ ] dimensions réservées
- [ ] scripts non bloquants lorsque possible
- [ ] preloads rares et justifiés
- [ ] pas de dépendances inutiles

## Qualité

- [ ] HTML validé
- [ ] indentation cohérente
- [ ] éléments obsolètes supprimés
- [ ] test sans CSS
- [ ] test sans JavaScript
- [ ] audit manuel d'accessibilité

---

# 23. Conclusion et ressources

HTML moderne est bien plus qu'une collection de balises.

Un document de qualité doit être :

1. **sémantique** ;
2. **accessible** ;
3. **valide** ;
4. **performant** ;
5. **sécurisé** ;
6. **maintenable**.

Le principe le plus important est souvent :

> **Utiliser l'élément natif qui exprime déjà le sens et le comportement recherchés.**

Avant de créer un composant avec plusieurs `<div>`, du JavaScript et de l'ARIA, vérifier si HTML fournit déjà :

- `<button>` ;
- `<a>` ;
- `<details>` ;
- `<dialog>` ;
- popover ;
- `<progress>` ;
- `<meter>` ;
- validation de formulaire ;
- `<picture>` ;
- éléments sémantiques.

Le navigateur pourra alors fournir davantage de comportements corrects par défaut.

## Ressources de référence

### Standard HTML

HTML Living Standard — WHATWG :

```text
https://html.spec.whatwg.org/
```

Édition destinée aux développeurs :

```text
https://html.spec.whatwg.org/dev/
```

### Référence pratique

MDN Web Docs — HTML :

```text
https://developer.mozilla.org/docs/Web/HTML
```

### Validation

Nu Html Checker :

```text
https://validator.w3.org/nu/
```

### Accessibilité

WAI — Web Accessibility Initiative :

```text
https://www.w3.org/WAI/
```

ARIA Authoring Practices Guide :

```text
https://www.w3.org/WAI/ARIA/apg/
```

## Pour continuer

Après ce cours :

1. approfondir [[CSS]] ;
2. étudier [[Javascript]] ;
3. travailler [[SEO]] ;
4. comprendre [[HTTP]] ;
5. pratiquer l'accessibilité sur des interfaces réelles ;
6. automatiser la validation et les tests dans les projets.

Le meilleur moyen de progresser reste de produire des documents simples, les valider, puis examiner régulièrement leur DOM, leur accessibilité, leur réseau et leur comportement au clavier.
