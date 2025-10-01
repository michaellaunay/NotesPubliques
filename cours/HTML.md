
# 1. Introduction
- **Objectif du cours** : Présentation des objectifs et des compétences visées.
- **Qu'est-ce que le HTML ?** : Histoire du HTML, son rôle dans le développement web, et comment il s'intègre avec les autres technologies web (CSS, JavaScript).
- **Environnement de développement** : Configuration d'un environnement simple pour la rédaction du code HTML (éditeurs de texte, navigateurs web).

# 2. Bases du HTML
- **Structure d'un document HTML** : Découverte de la structure de base (`<!DOCTYPE>`, `<html>`, `<head>`, `<body>`).
- **Balises et attributs** : Introduction aux balises courantes (titres, paragraphes, liens, images) et à l'utilisation des attributs.
- **Listes et tableaux** : Création de listes ordonnées, non ordonnées, et de tableaux simples.
- **Exercices pratiques** : Application des concepts appris à travers des exercices simples.

# 3. Organisation du contenu et sémantique
- **Sémantique HTML** : Importance de la sémantique pour l'accessibilité et le référencement. Introduction aux balises sémantiques (`<article>`, `<section>`, `<nav>`, `<footer>`, etc.).
- **Formulaires HTML** : Création de formulaires pour la collecte d'informations, avec différents types d'entrées.
- **Intégration multimédia** : Ajout de contenu multimédia (audio, vidéo, images) dans les pages web.
- **Exercices pratiques** : Construction de pages web plus complexes en intégrant des formulaires et du contenu multimédia.

# 4. Liens, images et ressources
- **Gestion des liens** : Création de liens internes et externes, utilisation des ancres.
- **Optimisation des images** : Formats d'image, utilisation des attributs pour l'optimisation SEO et l'accessibilité.
- **Utilisation des CSS** : Introduction basique aux CSS pour la mise en forme des documents HTML.
- **Exercices pratiques** : Amélioration visuelle des pages web avec des images optimisées et une introduction aux CSS.

# 5. Bonnes pratiques et projet final
- **Accessibilité web** : Principes de base pour rendre les pages web accessibles.
- **SEO (Search Engine Optimization)** : Bonnes pratiques pour améliorer le référencement des pages web.
- **Validation du code HTML** : Utilisation de validateurs pour s'assurer de la conformité du code aux standards.
- **Projet final** : Conception et réalisation d'un mini-site web en utilisant les compétences acquises durant le cours.

# 6. Conclusion
- **Récapitulatif du cours** : Révision des points clés et des compétences développées.
- **Ressources pour continuer à apprendre** : Liste de ressources pour approfondir ses connaissances en HTML et en développement web.
- **Feedback et évaluation** : Collecte des retours des apprenants et évaluation du cours.

# 1. Introduction

Dans ce premier chapitre, nous allons poser les bases de notre apprentissage du langage **HTML (HyperText Markup Language)**. L’objectif est de comprendre à quoi sert ce langage, dans quel contexte il a émergé, et comment il s’articule avec les autres technologies du web.

### Objectif du cours

Notre but est de nous familiariser progressivement avec la création de documents HTML, afin d’acquérir une autonomie dans la rédaction de pages web structurées et conformes aux standards. Au terme de ce cours, nous serons capables de :

- écrire du code HTML propre et compréhensible,
    
- organiser le contenu d’une page web de manière logique et sémantique,
    
- intégrer des éléments multimédia, des liens, des formulaires, et optimiser nos documents pour l’accessibilité et le référencement.
    

Il est important de garder en tête que le HTML n’est pas un langage de programmation, mais un **langage de balisage** qui permet de décrire la structure et le contenu d’un document.

### Qu’est-ce que le HTML ?

Le HTML a été créé au début des années 1990 par **Tim Berners-Lee**, chercheur au CERN, dans le but de partager facilement des documents scientifiques entre laboratoires à travers un réseau naissant : le **World Wide Web**. Depuis, il est devenu le standard incontournable pour la structuration des pages web.

Son rôle principal est de **définir la structure et la hiérarchie du contenu** (titres, paragraphes, listes, tableaux, images, formulaires…). En revanche, il ne gère pas la présentation graphique, qui est assurée par le **CSS (Cascading Style Sheets)**, ni les interactions dynamiques, qui relèvent du **JavaScript**. Ces trois technologies forment ce que nous appelons souvent le **socle du développement web front-end** :

- HTML → structure et contenu,
    
- CSS → présentation et mise en forme,
    
- JavaScript → interactions et dynamisme.
    

### Environnement de développement

Pour écrire du HTML, nous n’avons besoin que d’un **éditeur de texte** (comme Visual Studio Code, Sublime Text ou même un éditeur simple comme Notepad++) et d’un **navigateur web** (Firefox, Chrome, Safari, Edge…) pour afficher et tester nos pages. Contrairement à d’autres langages, aucun compilateur ou environnement lourd n’est nécessaire : un simple fichier texte enregistré avec l’extension `.html` suffit.

Nous apprendrons ensemble à configurer un environnement de développement simple et efficace, puis à mettre en place une méthodologie de travail adaptée, afin de faciliter la progression vers des projets plus complexes.

# 2. Bases du HTML

Après avoir compris les origines et le rôle du HTML, il est temps de nous plonger dans sa pratique. Dans ce chapitre, nous allons découvrir la **structure de base d’un document HTML**, manipuler les **balises** et leurs **attributs**, puis apprendre à organiser le contenu à l’aide de **listes** et de **tableaux**. Enfin, nous mettrons immédiatement en pratique ces notions à travers de petits exercices.

### 2.1 Structure d’un document HTML

Tout document HTML suit une structure standardisée. Celle-ci permet au navigateur de comprendre quel type de document il doit interpréter et d’afficher correctement le contenu.

Voici un exemple minimaliste d’une page HTML :

```html
<!DOCTYPE html>
 <html lang="fr">
    <head>
        <meta charset="UTF-8">
        <title>Ma première page</title>
    </head>
    <body>
        <h1>Bienvenue dans le monde du HTML</h1>
        <p>Ceci est mon tout premier paragraphe en HTML.</p>
    </body>
</html>`
```

Décortiquons cette structure :

- `<!DOCTYPE html>` : indique au navigateur que le document est du HTML5 (la version actuelle du standard).    
- `<html>` : balise racine qui englobe l’ensemble de la page.
- `<head>` : contient les métadonnées (titre de la page, encodage, liens vers des feuilles de style, etc.).
- `<body>` : contient tout ce qui sera visible par l’utilisateur dans le navigateur.

### 2.2 Balises et attributs

Le HTML repose sur des **balises**. Chaque balise décrit un élément du contenu. Elles sont généralement ouvertes et fermées, par exemple :

- `<p>Mon texte</p>` : un paragraphe,    
- `<h1>Mon titre</h1>` : un titre de premier niveau,
- `<a href="https://example.com">Un lien</a>` : un lien hypertexte,
- `<img src="photo.jpg" alt="Une photo">` : une image.
    

Les balises peuvent être enrichies avec des **attributs**. Ces attributs apportent des précisions sur le comportement ou l’apparence de l’élément. Par exemple :

- `href` indique l’adresse cible d’un lien,    
- `src` et `alt` précisent l’URL et la description d’une image,
- `lang="fr"` dans la balise `<html>` signale que la langue principale du document est le français.

### 2.3 Listes et tableaux

Pour organiser des informations, nous avons plusieurs options :

- **Listes non ordonnées** (à puces) :
```html
<ul>
   <li>Pomme</li>
   <li>Banane</li>
   <li>Cerise</li>
</ul>`

```

- **Listes ordonnées** (numérotées) :
    

`<ol>   <li>Étape 1</li>   <li>Étape 2</li>   <li>Étape 3</li> </ol>`

- **Tableaux simples** :
```html
<table border="1">
    <tr>
        <th>Nom</th>
        <th>Âge</th>
    </tr>
    <tr>
         <td>Alice</td>
         <td>22</td>
    </tr>
    <tr>
        <td>Bob</td>
        <td>25</td>
    </tr>
</table>`
```
Ici, `<tr>` définit une ligne, `<th>` une cellule d’en-tête et `<td>` une cellule de contenu.

### 2.4 Exercices pratiques

Pour assimiler ces notions, nous allons réaliser ensemble quelques exercices simples :

1. **Créer une page HTML minimale pour votre CV** contenant un titre et deux paragraphes.
2. **Ajouter un lien** vers le site de votre université et insérer une image (avec un attribut `alt`).
3. **Construire une liste ordonnée** de vos trois matières préférées et une liste non ordonnée de vos loisirs.
4. **Créer un tableau** présentant au moins trois personnes (nom, âge, domaine d’étude).

Ces exercices vous permettront de manipuler les balises essentielles et de prendre en main la structure d’un document HTML.

# 3. Organisation du contenu et sémantique

Jusqu’ici, nous avons vu comment structurer une page HTML et utiliser des balises de base. Nous allons maintenant nous concentrer sur une écriture plus **sémantique** et plus **riche en fonctionnalités**.

### 3.1 Sémantique HTML

La **sémantique** désigne l’utilisation de balises qui portent un sens clair, plutôt que de se contenter de simples divisions `<div>`. Cela rend nos pages plus faciles à comprendre, non seulement pour les développeurs, mais aussi pour les navigateurs, les moteurs de recherche (SEO) et les technologies d’assistance (lecteurs d’écran).

Quelques exemples de balises sémantiques utiles :

- `<header>` : zone d’en-tête d’une page ou d’une section (logo, menu, titre principal).
- `<nav>` : zone de navigation, généralement utilisée pour les menus.
- `<section>` : bloc thématique regroupant un ensemble cohérent de contenu.
- `<article>` : contenu autonome, comme un article de blog ou une actualité.
- `<aside>` : contenu secondaire (encadré, publicité, info complémentaire).
- `<footer>` : bas de page, contenant souvent les crédits, mentions légales, contacts.

Exemple d’utilisation :

```html
<article>
  <header>
    <h2>Titre de l’article</h2>
    <p>Publié le 1er octobre 2025</p>
  </header>
  <p>Voici le contenu principal de l’article.</p>
  <footer>
    <p>Auteur : Alice Dupont</p>
  </footer>
</article>
```

### 3.2 Formulaires HTML

Les **formulaires** permettent de collecter des informations auprès des utilisateurs (inscription, recherche, commentaires, etc.).

Un formulaire basique :

```html
<form action="/traitement" method="post">
  <label for="nom">Nom :</label>
  <input type="text" id="nom" name="nom" required><br>

  <label for="email">Email :</label>
  <input type="email" id="email" name="email" required><br>

  <label for="age">Âge :</label>
  <input type="number" id="age" name="age"><br>

  <label for="message">Message :</label>
  <textarea id="message" name="message"></textarea><br>

  <input type="submit" value="Envoyer">
</form>
```

Quelques points importants :

- L’attribut `action` indique la page ou le script qui traitera les données.
- L’attribut `method` détermine la manière d’envoyer les données (`get` ou `post`).
- Les champs `<input>` acceptent différents types (`text`, `email`, `password`, `number`, `date`, etc.).
- L’élément `<textarea>` sert aux zones de texte plus longues.
- Les éléments `<label>` améliorent l’accessibilité en liant le champ à une description.

### 3.3 Intégration multimédia

HTML5 facilite l’ajout de contenus multimédia dans nos pages :

- **Images** (déjà vues, mais rappel utile) :
    

```html
<img src="chat.jpg" alt="Photo d’un chat" width="300">
```

- **Audio** :

```html
<audio controls>
  <source src="musique.mp3" type="audio/mpeg">
  Votre navigateur ne supporte pas l’audio.
</audio>
```

- **Vidéo** :

```html
<video controls width="400">
  <source src="video.mp4" type="video/mp4">
  Votre navigateur ne supporte pas la vidéo.
</video>
```

- **Contenu externe (YouTube, etc.)** via `<iframe>` :
    

```html
<iframe width="560" height="315" 
  src="https://www.youtube.com/embed/VIDEO_ID" 
  title="Vidéo YouTube" frameborder="0" 
  allowfullscreen>
</iframe>
```

---

### 3.4 Exercices pratiques

Pour mettre en application ces nouvelles notions, nous allons réaliser plusieurs exercices :

1. **Créer une page sémantique** avec un `<header>`, un `<nav>`, une `<section>` contenant un `<article>`, et un `<footer>`.
    
2. **Construire un formulaire d’inscription** avec les champs : nom, email, mot de passe, choix de langue (liste déroulante), et bouton d’envoi.
    
3. **Intégrer un contenu multimédia** dans votre page : une image avec légende, un extrait audio ou une vidéo courte.
    
4. **Combiner les éléments** : créer une petite page « actualité » comprenant un article avec titre, texte, formulaire de commentaire et une vidéo associée.
    

Ces exercices nous permettront de franchir un cap : passer d’une simple page statique à une page organisée, interactive et enrichie.

# 4. Liens, images et ressources

Jusqu’à présent, nous avons appris à structurer et enrichir le contenu de nos pages avec du texte, des tableaux, des formulaires et du multimédia. Nous allons maintenant apprendre à **connecter nos pages entre elles** grâce aux liens, à **gérer efficacement les images** pour l’accessibilité et le référencement, puis à faire nos premiers pas avec le **CSS** (Cascading Style Sheets), indispensable pour la mise en forme.

### 4.1 Gestion des liens

Les liens hypertexte constituent l’essence même du **World Wide Web** : ils permettent de naviguer d’une page à une autre.

- **Lien externe** :

```html
<a href="https://www.wikipedia.org" target="_blank">Aller sur Wikipédia</a>
```

Ici, l’attribut `href` contient l’URL de destination, et `target="_blank"` ouvre le lien dans un nouvel onglet.

- **Lien interne** (vers une autre page du même site) :

```html
<a href="contact.html">Page de contact</a>
```

- **Lien vers une ancre dans la même page** :
    

```html
<a href="#section2">Aller à la section 2</a>

<h2 id="section2">Section 2</h2>
<p>Voici le contenu de la section 2.</p>
```

On utilise `id` pour identifier une partie de la page et `#id` dans le lien pour y accéder.

### 4.2 Optimisation des images

Les images rendent une page plus attractive, mais elles doivent être gérées avec soin pour éviter les temps de chargement trop longs et améliorer le référencement (SEO).

- **Formats d’images** :
    
    - JPEG → adapté aux photos (bonne compression).
    - PNG → idéal pour les images avec transparence.
    - GIF → utile pour les images animées simples.
    - WebP → format moderne, léger et performant.
    - SVG → format vectoriel, parfait pour les logos et icônes.
        
- **Balise `<img>` avec attributs optimisés** :
    

```html
<img src="paysage.webp" alt="Photo d’un paysage de montagne" width="600" loading="lazy">
```

- `alt` : description de l’image, utile pour l’accessibilité (lecteurs d’écran) et le SEO.
- `width` / `height` : fixent la taille affichée.
   - `loading="lazy"` : permet de charger l’image uniquement quand elle apparaît à l’écran, optimisant ainsi les performances.
   
### 4.3 Utilisation des CSS

Le HTML décrit le **contenu**, mais il ne gère pas la **mise en forme**. Pour cela, nous utilisons les **CSS (Cascading Style Sheets)**.

Il existe trois façons principales d’intégrer du CSS :

1. **Style en ligne** (à éviter sauf pour des cas ponctuels) :

```html
<p style="color: blue; font-size: 18px;">Texte en bleu</p>
```

2. **Style interne** (dans la balise `<head>`) :

```html
<head>
  <style>
    body { font-family: Arial, sans-serif; }
    h1 { color: darkred; }
  </style>
</head>
```

3. **Style externe** (fichier séparé, recommandé) :

```html
<link rel="stylesheet" href="style.css">
```

Dans `style.css` :

```css
body {
  font-family: Verdana, sans-serif;
  background-color: #f0f0f0;
}

h1 {
  color: navy;
  text-align: center;
}

p {
  line-height: 1.6;
}
```

Ce dernier mode est le plus courant, car il permet de séparer le contenu (HTML) de la présentation (CSS).

### 4.4 Exercices pratiques

Pour bien maîtriser ces notions, nous allons réaliser les exercices suivants :

1. **Créer un mini-site avec deux pages** (ex. `index.html` et `contact.html`) reliées entre elles par des liens internes.
2. **Ajouter un lien externe** vers un site de votre choix, qui s’ouvre dans un nouvel onglet.
3. **Intégrer deux images** dans la page, une au format JPEG et une au format SVG, en utilisant correctement les attributs `alt` et `loading="lazy"`.
4. **Créer une feuille de style CSS externe** pour :
    - changer la couleur de fond de la page,
    - centrer les titres,
    - améliorer la lisibilité des paragraphes (marges, espacement, taille de police).
