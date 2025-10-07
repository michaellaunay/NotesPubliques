# Plan du Cours

1. **Introduction à CSS**
   - Historique et évolution de CSS
   - Importance de CSS dans le développement web moderne
   - Intégration de CSS : Inline, Interne, Externe

2. **Les bases de CSS**
   - Syntaxe et structure
   - Commentaires en CSS
   - Les unités de mesure : px, em, rem, %, vh, vw, etc.

3. **Sélecteurs CSS : Fondamentaux**
   - Sélecteurs d'éléments (par tag)
   - Sélecteurs de classe (.) 
   - Sélecteurs d'ID (#)
   - Sélecteurs universels (*)
   - Groupe de sélecteurs

4. **Combinateurs et sélecteurs avancés**
   - Sélecteurs descendants (espace)
   - Sélecteurs d'enfants directs (>)
   - Sélecteurs d'adjacence directe (+)
   - Sélecteurs d'adjacence générale (~)
   - Sélecteurs d'attribut ([attr])

5. **Sélecteurs pseudo-classes et pseudo-éléments**
   - Pseudo-classes : :hover, :active, :focus, :nth-child, etc.
   - Pseudo-éléments : ::before, ::after
   - Utilisations courantes et cas pratiques

6. **Les propriétés CSS courantes**
   - Mise en forme du texte
   - Couleur et fond
   - Bordures et ombres
   - Positionnement : block, inline, relative, absolute, fixed, sticky
   - Flexbox et Grid Layout

7. **Responsive Design avec CSS**
   - Media Queries : introduction et application
   - Techniques pour un design adaptable : mobile-first vs desktop-first
   - Frameworks courants : Bootstrap, Foundation, etc. (aperçu)

8. **Pratiques optimales et outils**
   - Organisation et architecture CSS : BEM, OOCSS, SMACSS
   - Préprocesseurs CSS : Sass, Less (introduction)
   - Outils de développement : inspecteur de navigateur, plugins, etc.

9. **Performance et optimisation CSS**
   - Minification et compression
   - Chargement différé et asynchrone des styles
   - Conseils pour réduire le poids et améliorer le temps de chargement

10. **Tendances actuelles et avenir de CSS**
   - CSS Variables (custom properties)
   - Nouveaux modules et spécifications
   - Web animations avec CSS

# 1. Introduction à CSS

## 1.1 Historique et évolution de CSS

### 1.1.1 Origines de CSS

- **Contexte pré-CSS** :
  - Dans les premières années du web, le HTML était principalement utilisé pour structurer le contenu. La mise en forme était limitée et souvent mélangée avec le contenu lui-même.
  - Les sites web avaient tendance à avoir une apparence très basique et homogène en raison des limitations des outils disponibles.

- **Emergence du besoin** :
  - Avec l'augmentation du nombre d'utilisateurs et de sites internet, est née la nécessité d'une meilleure mise en forme et personnalisation.
  - Le W3C (World Wide Web Consortium) a identifié le besoin d'un langage spécifique pour gérer la présentation des pages web.

- **Introduction de CSS** :
  - En 1996, CSS (Cascading Style Sheets) est introduit par le W3C comme solution pour séparer le contenu de la mise en forme.
  - CSS 1, la première spécification, offrait un contrôle basique sur la mise en forme du texte, les couleurs, et l'arrière-plan.

### 1.1.2 Évolutions majeures

- **CSS 2 (1998)** :
  - Introduit pour adresser les limitations de CSS 1.
  - Ajout de la notion du modèle de boîte (box model), qui offre un contrôle sur la marge, la bordure, le remplissage, et le contenu.
  - Introduction des propriétés pour positionner les éléments et contrôler la visibilité.
  
- **CSS 2.1 (2005)** :
  - Plutôt qu'une grande mise à jour, cette version était une consolidation. Elle visait à corriger les bugs et clarifier certaines spécifications.
  - Elle est rapidement devenue la référence standard pour les développeurs web et les navigateurs.

- **CSS 3 (à partir de 2008)** :
  - Contrairement aux versions précédentes, CSS 3 est divisé en "modules". Chaque module est développé indépendamment, permettant une mise en œuvre plus rapide et flexible.
  - Introduction de nouvelles fonctionnalités comme les gradients, les ombres, les animations, les transitions, et des modules tels que Flexbox pour un contrôle avancé de la mise en page.
  - La modularité de CSS 3 signifie que de nouvelles fonctionnalités continuent d'être introduites et améliorées, même après la première sortie de CSS 3.

## 1.2 Importance de CSS dans le développement web moderne

### 1.2.1 Séparation du contenu et de la mise en forme

- **Raison d'être du CSS** :
  - L'idée fondamentale derrière le développement de CSS était de séparer le "quoi" (contenu) du "comment" (présentation). Cela permet aux designers et développeurs de travailler de manière plus efficace et collaborative.

- **Avantages de cette séparation** :
  - **Maintenance** : Mettre à jour le style sans toucher au contenu réduit le risque d'erreurs.
  - **Accessibilité** : En gardant le contenu pur et la présentation séparée, les sites deviennent plus accessibles pour les lecteurs d'écran et d'autres outils d'assistance.
  - **Optimisation multi-appareils** : Un seul ensemble de contenu peut être stylisé différemment pour divers appareils et tailles d'écran, rendant le web design responsive plus efficace.

### 1.2.3 Personnalisation et expérience utilisateur

- **Au-delà du texte noir sur fond blanc** :
  - CSS permet une personnalisation sans précédent du web. Les marques peuvent refléter leur identité visuelle et les designers peuvent réaliser leurs visions créatives.
  
- **Interaction et dynamisme** :
  - Avec l'évolution de CSS, en particulier CSS3, les designers peuvent créer des animations, des transitions, et des interactions qui rendent l'expérience utilisateur plus engageante.
  
- **Adaptabilité** :
  - CSS offre la flexibilité nécessaire pour que les designs s'adaptent à une variété d'appareils, de tailles d'écran, et de préférences utilisateur. C'est essentiel à l'ère du mobile.

### 1.2.4 Performance

- **Chargement rapide des pages** :
  - Une mise en œuvre efficace de CSS peut réduire la taille et la complexité des sites web, ce qui conduit à des temps de chargement plus rapides.
  
- **Influence sur le SEO** :
  - Les moteurs de recherche valorisent l'expérience utilisateur. Une mise en page propre, un chargement rapide et une structure claire (aidée par une utilisation judicieuse de CSS) peuvent tous contribuer à un meilleur classement dans les résultats de recherche.
  
- **Engagement des utilisateurs** :
  - Un design attrayant et une interface utilisateur réactive peuvent réduire le taux de rebond et augmenter le temps passé sur le site. La performance n'est pas seulement une question de vitesse, mais aussi d'expérience globale.

## 1.3 Intégration de CSS : Inline, Interne, Externe

### 1.3.1 Inline CSS

- **Définition** :
  - Il s'agit d'appliquer des styles directement sur des éléments individuels à l'aide de l'attribut `style` dans le code HTML.

- **Exemple** :
  ```html
  <p style="color: blue;">Ce texte est bleu.</p>
  ```

- **Quand l'utiliser (et quand ne pas l'utiliser)** :
  - **Avantages** : 
    - Immédiateté : Utile pour des tests rapides.
    - Spécificité : Puisqu'il est directement sur l'élément, il a une haute priorité dans la cascade CSS.
  - **Inconvénients** : 
    - Pas scalable : Peut rendre le code encombré et difficile à maintenir pour de grands sites.
    - Répétitif : Si le même style est nécessaire sur plusieurs éléments, il faut le répéter pour chaque élément.

### 1.3.2 CSS Interne (ou Embarqué)

- **Définition** :
  - Il s'agit d'intégrer des styles dans l'en-tête (section `<head>`) de votre document HTML à l'aide de l'élément `<style>`.

- **Exemple** :
  ```html
  <head>
      <style>
          p { color: red; }
      </style>
  </head>
  ```

- **Quand l'utiliser** :
  - Pour des styles spécifiques à une seule page.
  - Lorsqu'il n'est pas nécessaire de charger une feuille de style externe.
  
- **Inconvénients** : 
  - Si le même style est nécessaire sur plusieurs pages, il doit être copié sur chaque page, ce qui n'est pas optimal pour la maintenance.

### 1.3.3 CSS Externe

- **Définition** :
  - Il s'agit de définir les styles dans un fichier séparé (généralement avec une extension `.css`) et de le lier à votre document HTML.

- **Comment l'intégrer** :
  - Utilisation de l'élément `<link>` dans la section `<head>` de votre HTML.
  
- **Exemple** :
  ```html
  <head>
      <link rel="stylesheet" type="text/css" href="styles.css">
  </head>
  ```

- **Avantages** :
  - **Séparation du contenu et de la mise en forme** : Conforme à la meilleure pratique de séparation des préoccupations.
  - **Réutilisabilité** : Une seule feuille de style peut être appliquée à plusieurs pages, garantissant la cohérence et facilitant la maintenance.
  - **Mise en cache par le navigateur** : Les feuilles de style externes peuvent être mises en cache par les navigateurs, améliorant la performance de chargement pour les visites répétées.

# 2. Les bases de CSS

## 2.1 Syntaxe et structure

- **Définition** :
Le CSS (Cascading Style Sheets) est un langage dédié à la mise en forme et à la présentation des documents HTML. Il possède sa propre grammaire et sa propre syntaxe. Les styles CSS peuvent être intégrés directement dans les documents HTML ou être placés dans des fichiers séparés. Ces styles, pour être appliqués, vont être interprétés par les navigateurs web pour définir l'apparence des éléments HTML. Fondamentalement, le CSS fonctionne sur le principe du trio : sélecteur, propriété et valeur.

- **Sélecteur, propriété, valeur** :
  - **Sélecteur** : Spécifie à quel élément HTML le style sera appliqué.
  - **Propriété** : L'aspect que vous souhaitez changer (ex. couleur, taille de police, etc.).
  - **Valeur** : Ce que vous souhaitez définir pour cette propriété.

- **Exemple de code** :
  ```css
  p {
      color: red;
      font-size: 16px;
  }
  ```

- **Bloc de déclaration** :
  - La combinaison d'une ou plusieurs déclarations entourées de accolades `{}`.

## 2.2 Commentaires en CSS

- **Importance des commentaires** :
  - Pour documenter le code.
  - Pour aider d'autres développeurs (ou vous-même, à l'avenir) à comprendre votre pensée.

- **Comment écrire un commentaire** :
  - En CSS, les commentaires sont entourés par `/*` et `*/`.

- **Exemple** :
  ```css
  /* Ceci est un commentaire en CSS */
  p {
      color: blue;  /* Ceci est un autre commentaire */
  }
  ```

- **Pratiques recommandées** :
  - Éviter les commentaires superflus.
  - Utiliser des commentaires pour documenter des sections complexes ou importantes.

## 2.3 Les unités de mesure : px, em, rem, %, vh, vw, etc.

- **Pixels (px)** :
  - L'unité la plus couramment utilisée en web design.
  - Représente une unité fixe, indépendante de tout autre facteur.

- **Em** :
  - Relative à la taille de police de l'élément parent.
  - Exemple : Si la taille de police parent est de 16px, alors `1em = 16px`.

- **Rem** :
  - Similaire à `em`, mais toujours relative à la taille de police de base du document (généralement définie sur l'élément `<html>`).

- **Pourcentage (%)** :
  - Une unité relative qui est un pourcentage de l'élément parent.
  - Souvent utilisé pour les largeurs pour créer des designs fluides.

- **Unités de viewport : vh, vw** :
  - `vh` (viewport height) : 1 unité est égale à 1% de la hauteur du viewport.
  - `vw` (viewport width) : 1 unité est égale à 1% de la largeur du viewport.
  
- **Utilisation pratique et considérations** :
  - Quand utiliser une unité fixe (comme px) par rapport à une unité relative (comme em, rem, ou %).
  - Comment les unités relatives peuvent aider à créer des designs plus adaptatifs et réactifs.

# 3. Sélecteurs CSS : Fondamentaux

## 3.1 Sélecteurs d'éléments (par tag)

- **Définition** :
  - Ces sélecteurs ciblent des éléments HTML spécifiques par leur nom de balise.

- **Exemples** :
  ```css
  p {
      color: green;
  }
  
  h1 {
      font-size: 24px;
  }
  ```

- **Usage** :
  - Ils sont utiles lorsque vous souhaitez appliquer un style à tous les éléments d'un type spécifique sur une page.

## 3.2 Sélecteurs de classe `.`

- **Définition** :
  - Les sélecteurs de classe ciblent des éléments basés sur l'attribut `class` de l'élément.

- **Exemples** :
  ```css
  .important {
      font-weight: bold;
  }
  
  .highlight {
      background-color: yellow;
  }
  ```

- **Usage** :
  - Ils sont essentiels lorsque vous souhaitez appliquer un style à un groupe spécifique d'éléments sans affecter d'autres éléments du même type.

## 3.3 Sélecteurs d'ID `#`

- **Définition** :
  - Les sélecteurs d'ID ciblent un élément spécifique basé sur l'attribut `id` de l'élément. Chaque ID devrait être unique sur une page.

- **Exemples** :
  ```css
  #header {
      background-color: gray;
  }
  
  #footer {
      font-size: 12px;
  }
  ```

- **Usage** :
  - Idéal pour styliser des éléments uniques, comme une en-tête ou un pied de page. Cependant, leur usage doit être limité car les IDs sont censés être uniques.

## 3.4 Sélecteurs universels `*`

- **Définition** :
  - Ce sélecteur cible tous les éléments d'une page.

- **Exemples** :
  ```css
  * {
      margin: 0;
      padding: 0;
  }
  ```

- **Usage** :
  - Utile pour la réinitialisation des styles ou pour appliquer un style général à tous les éléments. Cependant, utilisez-le avec précaution car il peut avoir un impact sur les performances.

## 3.5 Groupe de sélecteurs

- **Définition** :
  - Si nous souhaitons appliquer les mêmes styles à plusieurs sélecteurs, nous pouvons les regrouper en utilisant une virgule.

- **Exemples** :
  ```css
  h1, h2, h3 {
      font-family: Arial, sans-serif;
  }
  ```

- **Usage** :
  - Cela évite la redondance dans le code et facilite la maintenance, car nous pouvons changer le style pour plusieurs éléments en modifiant une seule règle.

# 4. Combinateurs et sélecteurs avancés

## 4.1 Sélecteurs descendants (espace )

- **Définition** :
  - Ce sélecteur cible tous les éléments qui sont descendants d'un élément spécifié.

- **Exemples** :
  ```css
  article p {
      color: blue;
  }
  ```
  Cela va cibler tous les paragraphes (`<p>`) qui sont à l'intérieur d'un élément `<article>`, peu importe leur niveau de profondeur.

## 4.2 Sélecteurs d'enfants directs `>`

- **Définition** :
  - Ce sélecteur cible les éléments qui sont des enfants directs d'un élément spécifié.

- **Exemples** :
  ```css
  div > p {
      font-weight: bold;
  }
  ```
  Cela cible uniquement les paragraphes qui sont des enfants directs d'une `<div>`, et non les paragraphes qui sont plus profondément imbriqués.

## 4.3 Sélecteurs d'adjacence directe `+`

- **Définition** :
  - Ce sélecteur cible l'élément immédiatement après un certain élément.

- **Exemples** :
  ```css
  h2 + p {
      margin-top: 10px;
  }
  ```
  Cela va ajouter une marge au sommet d'un paragraphe qui vient immédiatement après un titre `<h2>`.

## 4.3 Sélecteurs d'adjacence générale `~`

- **Définition** :
  - Ce sélecteur cible tous les éléments qui sont après un élément donné et partagent le même parent.

- **Exemples** :
  ```css
  h2 ~ p {
      text-indent: 1em;
  }
  ```
  Cela cible tous les paragraphes qui viennent après un `<h2>` et qui partagent le même parent.

## 4.4 Sélecteurs d'attribut `[attr]`

- **Définition** :
  - Ce sélecteur cible les éléments basés sur la présence ou la valeur d'un attribut spécifique.

- **Exemples** :
  ```css
  input[type="text"] {
      border-radius: 5px;
  }
  ```
  Cela cible tous les champs input de type "text" et leur donne un bord arrondi.

D'autres variations incluent :
  ```css
  a[href^="https://"] {
      color: green;  /* Cible tous les liens commençant par "https://" */
  }

  a[href$=".pdf"] {
      background: url('pdf-icon.png') no-repeat; /* Cible tous les liens se terminant par ".pdf" */
  }

  a[href*="example"] {
      text-decoration: underline; /* Cible tous les liens contenant le mot "example" */
  }
  ```

# 5. Sélecteurs pseudo-classes et pseudo-éléments

## 5.1 Pseudo-classes

Les pseudo-classes permettent de définir des styles pour des états spécifiques d'un élément ou pour des éléments spécifiques dans une séquence ou une hiérarchie.

- **:hover**
  - Cible un élément lorsqu'il est survolé par un pointeur (comme une souris).
  ```css
  a:hover {
      color: red;
  }
  ```

- **:active**
  - Cible un élément pendant qu'il est activé (par exemple, lorsqu'un bouton est enfoncé).
  ```css
  button:active {
      background-color: lightgray;
  }
  ```

- **:focus**
  - Cible un élément lorsqu'il a le focus. Très utile pour les éléments interactifs comme les entrées de formulaire.
  ```css
  input:focus {
      border: 2px solid blue;
  }
  ```

- **:nth-child**
  - Cible des éléments spécifiques en fonction de leur position dans une séquence.
  ```css
  li:nth-child(odd) {
      background-color: lightgray;
  }
  ```

Il existe de nombreuses autres pseudo-classes, mais celles-ci sont parmi les plus couramment utilisées.

## 5.2 Pseudo-éléments

Les pseudo-éléments permettent de cibler et de styliser une partie spécifique d'un élément.

- **::before**
  - Utilisé pour insérer du contenu avant le contenu d'un élément.
  ```css
  p::before {
      content: "Important : ";
      font-weight: bold;
  }
  ```

- **::after**
  - Utilisé pour insérer du contenu après le contenu d'un élément.
  ```css
  a::after {
      content: " →";
  }
  ```

## 5.3 Utilisations courantes et cas pratiques

- **Création d'effets de survol** : Utilisation de `:hover` pour changer la couleur de fond d'un bouton ou la couleur du texte d'un lien.

- **Amélioration de l'accessibilité** : Utilisation de `:focus` pour mettre en évidence clairement les éléments de formulaire actifs, aidant ainsi les utilisateurs qui naviguent au clavier.

- **Création de listes stylisées** : Avec `::before`, vous pouvez personnaliser les puces de liste ou ajouter des icônes spécifiques devant certains éléments.

- **Indication de liens externes** : En utilisant `::after` avec une combinaison de sélecteurs d'attributs, vous pouvez ajouter une icône après chaque lien qui pointe vers un site externe.

- **Coloration de lignes de tableau en alternance** : En utilisant `:nth-child`, vous pouvez donner des couleurs différentes aux lignes impaires et paires d'un tableau pour améliorer la lisibilité.

# 6. Les propriétés CSS courantes

Dans cette section, nous allons approfondir certaines des propriétés CSS les plus essentielles et couramment utilisées pour le design web. Ces propriétés sont le fondement de nombreuses maquettes web, alors prenons un moment pour les explorer en détail.

## 6.1 Mise en forme du texte

Le texte est l'élément fondamental de la plupart des sites web. Voici comment nous pouvons le styliser :

- **font-family** : Détermine la police utilisée.
- **font-size** : Définit la taille de la police.
- **font-weight** : Contrôle l'épaisseur du texte, comme "normal" ou "bold".
- **line-height** : Ajuste l'espacement entre les lignes de texte.
- **text-align** : Oriente le texte (comme "left", "right", "center", ou "justify").
- **text-decoration** : Ajoute ou supprime des décorations, comme "underline" ou "none".

## 6.2 Couleur et fond

La couleur est vitale pour la conception visuelle d'un site web :

- **color** : Définit la couleur du texte.
- **background-color** : Détermine la couleur d'arrière-plan d'un élément.
- **background-image** : Applique une image en tant qu'arrière-plan.
- **background-position** : Positionne l'image d'arrière-plan.
- **background-repeat** : Contrôle si et comment une image d'arrière-plan se répète.

## 6.3 Bordures et ombres

Les bordures et les ombres peuvent ajouter de la profondeur et de la distinction :

- **border-style** : Définit le style de la bordure, comme "solid", "dashed" ou "dotted".
- **border-color** : Détermine la couleur de la bordure.
- **border-width** : Contrôle l'épaisseur de la bordure.
- **box-shadow** : Ajoute une ombre à un élément, donnant une sensation de profondeur.

## 6.4 Positionnement

La manière dont les éléments sont positionnés est cruciale pour la mise en page :

- **display** : Détermine comment un élément est affiché, par exemple "block" ou "inline".
- **position** : Contrôle le type de positionnement d'un élément (comme "static", "relative", "absolute", "fixed" ou "sticky").
- **top, right, bottom, left** : Définissent la position exacte d'un élément lorsqu'il est positionné de manière autre que "static".

## 6.5 Flexbox et Grid Layout

Ces deux propriétés ont révolutionné la mise en page web :

### 6.5.1  Flexbox

Flexbox, ou "Flexible Box Layout", est une méthode que nous utilisons pour concevoir des structures de mise en page complexes avec une manière plus prévisible, surtout lorsqu'il s'agit de distribuer de l'espace et d'aligner des éléments, même lorsque leurs tailles sont inconnues ou dynamiques.

#### 6.5.1.1 Modèle unidimensionnel

Contrairement à d'autres modèles de mise en page tels que Grid, Flexbox est principalement axé sur la mise en page dans une seule dimension à la fois, soit une colonne ou une rangée. Cela le rend particulièrement adapté pour créer des composants d'interface utilisateur ou de petites mises en page où l'alignement et la distribution d'espace sont essentiels.

#### 6.5.1.2 `display: flex`

Lorsque nous définissons la propriété `display` d'un élément parent à `flex`, cet élément devient un conteneur flex, et tous ses enfants directs deviennent des éléments flexibles. Cela signifie qu'ils vont immédiatement adopter le comportement de flexbox, permettant une variété de manipulations d'alignement et de répartition de l'espace.

#### 6.5.1.3 `justify-content` et `align-items`

Ces deux propriétés sont au cœur de la puissance d'alignement de Flexbox.

- **`justify-content` :** Nous utilisons cette propriété sur le conteneur flex pour aligner ses enfants (les éléments flex) horizontalement. Elle propose plusieurs valeurs telles que :
  - `flex-start`: aligne les éléments au début du conteneur flex.
  - `flex-end`: aligne les éléments à la fin.
  - `center`: centre les éléments.
  - `space-between`: distribue uniformément l'espace entre les éléments.
  - `space-around`: distribue l'espace autour des éléments, ce qui signifie que les éléments ont un espace égal avant et après eux.

- **`align-items` :** Cette propriété est similaire à `justify-content`, mais elle aligne les éléments flexibles verticalement. Ses valeurs comprennent :
  - `flex-start`
  - `flex-end`
  - `center`
  - `baseline`: aligne les éléments sur leur ligne de base.
  - `stretch`: étire les éléments pour occuper toute la hauteur du conteneur flex (c'est la valeur par défaut).

### 6.5.2 Grid Layout

Le Grid Layout est une technique de mise en page qui nous permet de concevoir des structures bidimensionnelles pour nos sites web et nos applications. Contrairement à Flexbox, qui est principalement unidimensionnel, Grid nous permet de travailler à la fois sur les colonnes et les rangées, ce qui le rend idéal pour créer des mises en page complexes avec des régions distinctes.

#### 6.5.2.1 Modèle bidimensionnel

Avec Grid, nous pouvons définir des colonnes et des rangées et placer des éléments où nous le souhaitons à l'intérieur de ces grilles. Cela nous donne une grande flexibilité pour concevoir des mises en page qui étaient auparavant difficiles ou impossibles avec les méthodes traditionnelles.

#### 6.5.2.2 `display: grid`

En définissant la propriété `display` d'un élément à `grid`, nous le transformons en conteneur de grille. Tous ses enfants directs deviennent des éléments de la grille, ce qui signifie qu'ils peuvent être positionnés spécifiquement à l'intérieur des colonnes et des rangées de la grille.

#### 6.5.2.3 `grid-template-columns` et `grid-template-rows`

Ces propriétés définissent la structure même de notre grille.

- **`grid-template-columns` :** Avec cette propriété, nous définissons le nombre et la taille des colonnes de notre grille. Nous pouvons définir des tailles fixes, relatives ou même des fractions de l'espace disponible.
  - **Exemple :** `grid-template-columns: 1fr 2fr 1fr;` créera une grille avec trois colonnes. La colonne du milieu sera deux fois plus large que les colonnes sur les côtés.

- **`grid-template-rows` :** De manière similaire, cette propriété nous permet de définir le nombre et la taille des rangées de notre grille.
  - **Exemple :** `grid-template-rows: 100px auto 100px;` créera une grille avec trois rangées. La première et la dernière auront une hauteur fixe de 100px, tandis que la rangée du milieu prendra tout l'espace restant.

#### 6.5.2.4 Exemple simple
Voici un exemple simple en **CSS Grid Layout** pour créer **deux colonnes** :

```html
<div class="grid-container">
  <div class="item">Colonne 1</div>
  <div class="item">Colonne 2</div>
</div>
```

```css
.grid-container {
  display: grid;
  grid-template-columns: 1fr 1fr; /* Deux colonnes de taille égale */
  gap: 20px; /* Espace entre les colonnes */
  padding: 20px;
  background-color: #f5f5f5;
}

.item {
  background-color: #ddd;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
}
```

##### 6.5.2.5 Variante : colonnes de tailles différentes

```css
.grid-container {
  display: grid;
  grid-template-columns: 2fr 1fr; /* La 1re colonne fait 2x la taille de la 2e */
  gap: 20px;
}
```
#### 6.5.2.6 Variante responsive (1 colonne sur mobile)

```css
.grid-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

@media (max-width: 700px) {
  .grid-container {
    grid-template-columns: 1fr; /* Une seule colonne sur petit écran */
  }
}
```

# 7. Responsive Design avec CSS

## 7. Responsive Design avec CSS

Le design responsive est aujourd'hui une nécessité pour toute présence en ligne. Les utilisateurs accèdent à du contenu à partir d'une multitude d'appareils, du smartphone au desktop en passant par les tablettes. Voyons comment CSS nous aide à gérer cela.

## 7.1 Media Queries : introduction et application

Les media queries sont l'épine dorsale du design responsive. Elles permettent d'appliquer des styles basés sur certaines conditions, telles que la largeur, la hauteur et le type d'appareil.

- **Syntaxe de base**:
  ```css
  @media (condition) {
      /* styles à appliquer */
  }
  ```
- **Exemple pratique**:
  ```css
  @media (max-width: 600px) {
      body {
          font-size: 14px;
      }
  }
  ```
  Dans l'exemple ci-dessus, nous définissons que lorsque la largeur de l'écran est inférieure ou égale à 600px, la taille de la police du texte principal sera de 14px.

## 7.2 Techniques pour un design adaptable : mobile-first vs desktop-first

### 7.2.1 Mobile-first

Comme son nom l'indique, cette approche consiste à concevoir d'abord pour les petits écrans, puis à ajuster pour les écrans plus grands avec l'aide de media queries. C'est souvent considéré comme plus efficace car il oblige à concentrer d'abord l'attention sur les fonctionnalités essentielles.

### 7.2.2 Desktop-first
 
Ici, nous commençons par concevoir pour les grands écrans. Cependant, cela peut parfois conduire à des solutions plus compliquées lors de la réduction à des écrans plus petits.

Dans les deux cas, le but est de garantir que l'utilisateur ait une expérience optimale, quel que soit l'appareil qu'il utilise.

## 7.3 Frameworks courants : Bootstrap, Foundation, etc.

De nombreux frameworks CSS ont été développés pour faciliter le processus de conception responsive.

### 7.3.1 Bootstrap

Bootstrap est une bibliothèque open-source qui nous fournit un cadre pour le développement web rapide et adaptatif. Depuis son introduction par Twitter en 2011, il a considérablement gagné en popularité et est devenu le framework de référence pour de nombreux développeurs et concepteurs.

### 7.3.1.1 Popularité et adoption

La raison de la popularité massive de Bootstrap réside dans sa simplicité et sa robustesse. Il est conçu pour aider les développeurs de tous niveaux, des débutants aux professionnels, à créer des sites et des applications web adaptatifs avec une cohérence esthétique.

### 7.3.1.2 Design responsive

L'une des caractéristiques phares de Bootstrap est sa capacité intrinsèque à créer des designs qui s'adaptent à toutes les tailles d'écran, des smartphones aux grands moniteurs de bureau. Cela est réalisé grâce à un système de grille fluide qui restructure automatiquement le contenu en fonction de la largeur de l'écran.

### 7.3.1.3 Richesse des composants

Bootstrap fournit une vaste bibliothèque de composants pré-stylisés. Ces composants, allant des boutons, des formulaires, des cartes aux modales, peuvent être facilement intégrés dans n'importe quelle page, réduisant ainsi le temps de développement et garantissant une uniformité du design.

- **Exemple :** L'intégration d'une barre de navigation adaptative est aussi simple que d'ajouter des classes spécifiques à votre élément HTML.

### 7.3.1.4 Personnalisabilité

Bootstrap est flexible. Bien qu'il vienne avec un design par défaut, ce design peut être personnalisé pour correspondre à l'identité de la marque ou aux préférences esthétiques. Que ce soit en modifiant les variables SCSS fournies ou en écrasant les styles par défaut, nous pouvons facilement faire de Bootstrap notre propre outil.

### 7.3.1.5 Communauté et ressources

L'une des grandes forces de Bootstrap est sa communauté dynamique. Grâce à cette communauté, une multitude de thèmes, plugins et extensions sont disponibles. De plus, avec autant d'utilisateurs actifs, obtenir de l'aide ou résoudre des problèmes spécifiques est souvent plus facile et plus rapide.

### 7.3.2 Foundation

Créé par ZURB, une entreprise de design et de développement de produits, Foundation est un autre cadre front-end puissant qui est utilisé pour construire des sites et des applications web réactifs. Alors que Bootstrap peut dominer en termes de popularité, Foundation a ses propres forces qui en font un choix précieux pour de nombreux développeurs.

### 7.3.2.1 Modularité et flexibilité

L'un des principaux avantages de Foundation est sa modularité. Il est conçu pour être extrêmement flexible, permettant aux développeurs de choisir les composants spécifiques qu'ils souhaitent utiliser. Cette approche modulaire signifie que vous pouvez maintenir votre code propre et léger, n'incluant que ce dont vous avez réellement besoin.

### 7.3.2.2 Système de grille

Tout comme Bootstrap, Foundation propose un système de grille adaptatif. Cependant, il se distingue par sa grille "XY" qui offre une plus grande flexibilité pour la création de mises en page. Avec cette grille, la position des éléments à la fois horizontalement et verticalement est simplifiée.

### 7.3.2.3 Composants pré-stylisés

Foundation comprend une riche collection de composants UI, comme les boutons, les formulaires, les modales et les alertes. Bien qu'ils soient pré-stylisés pour un aspect propre et moderne, tout est personnalisable pour s'adapter aux besoins spécifiques d'une marque ou d'un design.

### 7.3.2.4 Personnalisation avec Sass

Foundation utilise Sass, un préprocesseur CSS, pour faciliter la personnalisation. En utilisant les fonctions et variables de Sass, les développeurs peuvent rapidement adapter le framework à leurs besoins esthétiques et fonctionnels.

### 7.3.2.5 Accès à la formation et à la documentation

ZURB offre une vaste documentation et de nombreuses ressources éducatives pour aider les utilisateurs à tirer le meilleur parti de Foundation. La documentation est complète et bien organisée, avec des exemples de code et des démos pour aider à la compréhension.

### 7.3.2.6 Compatibilité

Foundation met l'accent sur la création de sites qui fonctionnent bien sur tous les appareils, y compris les navigateurs obsolètes. Sa compatibilité avec des navigateurs comme Internet Explorer est l'un de ses points forts.

### 7.3.3 Bulma

Bulma est un framework CSS moderne basé sur Flexbox. Comme Bootstrap et Foundation, Bulma offre un ensemble de styles et de composants pour accélérer le développement de sites web réactifs, mais il se distingue par sa simplicité et sa légèreté.

#### 7.3.3.1 Basé sur Flexbox

La principale particularité de Bulma est qu'il est entièrement basé sur le modèle de mise en page Flexbox de CSS. Cela rend la création de mises en page complexes et adaptatives beaucoup plus simple et plus intuitive.

#### 7.3.3.2 Simplicité et lisibilité

L'une des forces de Bulma est sa syntaxe claire et ses classes descriptives. Par exemple, pour créer une colonne qui prend la moitié de l'espace disponible, vous pouvez simplement utiliser la classe `is-half`. Cette approche rend le code facile à écrire et à comprendre.

#### 7.3.3.3 Modularité

Bulma est structuré de manière modulaire, ce qui signifie que vous pouvez inclure uniquement les fichiers Sass dont vous avez besoin. Cela vous permet de garder votre CSS final aussi léger que possible.

#### 7.3.3.4 Personnalisable

Bulma utilise des variables Sass, permettant une personnalisation facile et rapide. Vous pouvez adapter les couleurs, les tailles et d'autres propriétés à vos besoins sans avoir à parcourir et à éditer des lignes et des lignes de code.

#### 7.3.3.5 Documentation solide

Bulma est bien documenté, avec de nombreux exemples et modificateurs pour chaque composant. Ceci est particulièrement utile pour les développeurs qui découvrent le framework pour la première fois.

#### 7.3.3.6 Pas de JavaScript inclus

Contrairement à d'autres frameworks comme Bootstrap, Bulma est strictement un framework CSS. Il ne comprend pas de composants JavaScript, ce qui signifie que vous êtes libre de choisir vos propres solutions JavaScript ou jQuery selon les besoins de votre projet.

Il y a aussi d'autres frameworks comme TailwindCSS, etc. Mais Bootstrap et Foundation sont les plus reconnus.

# 8. Pratiques optimales et outils

Lorsque nous abordons des projets de grande envergure, il est essentiel de suivre certaines pratiques éprouvées et d'utiliser des outils efficaces pour nous assurer que notre CSS reste maintenable, organisé et efficace.

## 8.1 Organisation et architecture CSS : BEM, OOCSS, SMACSS

Le CSS peut rapidement devenir encombré et difficile à gérer si nous n'adoptons pas une structure cohérente. Voici quelques méthodologies populaires pour organiser votre CSS :

- **BEM (Block, Element, Modifier)** : Cette technique divise votre interface en blocs (composants), éléments (parties d'un composant) et modificateurs (variantes d'un composant ou d'un élément). C'est un moyen pratique de garantir que vos classes sont spécifiques et explicites.
  
  Exemple :
  ```css
  .button { /* Block */
    ...
  }
  .button__icon { /* Element */
    ...
  }
  .button--large { /* Modifier */
    ...
  }
  ```

- **OOCSS (Object-Oriented CSS)** : Cette approche sépare les styles en structures (comme les grilles) et les styles visuels (comme les thèmes), facilitant la réutilisation et le maintien.

- **SMACSS (Scalable and Modular Architecture for CSS)** : Il divise le design en cinq catégories : base, layout, module, state et theme. Cette distinction aide à garder le CSS lisible et structuré.

## 8.2 Préprocesseurs CSS : Sass, Less (introduction)

Les préprocesseurs étendent les capacités du CSS, offrant des fonctionnalités supplémentaires :

- **Sass (Syntactically Awesome Style Sheets)** : C'est un préprocesseur puissant qui ajoute des variables, des fonctions, des boucles et bien plus au CSS. Il dispose de deux syntaxes : la syntaxe SCSS, qui est proche du CSS, et la syntaxe indentée.

- **Less** : Tout comme Sass, Less offre des variables, des fonctions et d'autres fonctionnalités pour améliorer le CSS. Sa syntaxe est assez similaire à celle du CSS.

L'utilisation de ces préprocesseurs peut accélérer le développement et faciliter la maintenance.

## 8.3 Outils de développement : inspecteur de navigateur, plugins, etc.

Les navigateurs modernes offrent des outils puissants pour les développeurs :

- **Inspecteur de navigateur** : C'est un outil intégré qui permet d'examiner et de modifier le HTML et le CSS en temps réel. C'est inestimable pour le débogage et l'apprentissage.

- **Plugins et extensions** : Il existe de nombreux plugins pour les navigateurs, comme "ColorZilla" pour choisir des couleurs ou "CSS Viewer" pour voir rapidement les styles CSS d'un élément.

# 9. Performance et optimisation CSS

À mesure que les sites web et les applications deviennent de plus en plus complexes, les performances deviennent une préoccupation majeure. Un CSS optimisé peut considérablement accélérer le temps de chargement d'un site, améliorant ainsi l'expérience utilisateur et les performances globales.

## 9.1 Minification et compression

La minification est le processus de suppression de tous les caractères non nécessaires d'un fichier sans affecter sa fonctionnalité. Les espaces, les commentaires, les sauts de ligne, tout cela est retiré.

- **Pourquoi minifier ?** En réduisant la taille du fichier, nous accélérons le temps de téléchargement du fichier. C'est un gain rapide et facile en termes de performances.

- **Outils** : Il existe de nombreux outils en ligne pour minifier le CSS, tels que CSSNano, CSS Minifier, entre autres.

La compression, quant à elle, est le processus de codage des informations à l'aide de moins de bits. Gzip est une méthode couramment utilisée pour compresser les fichiers CSS (et d'autres ressources).

## 9.2 Chargement différé et asynchrone des styles

Parfois, tout le CSS n'est pas nécessaire pour afficher la partie visible initiale d'une page. Dans ces cas, nous pouvons :

- **Utiliser le chargement différé** : Cela signifie que certains styles ne sont chargés qu'après que le contenu principal est affiché.

- **Chargement asynchrone** : Cela permet au navigateur de continuer à charger d'autres ressources sans attendre que le CSS soit chargé.

L'utilisation de ces techniques nécessite une planification soignée pour garantir que les utilisateurs voient toujours un site correctement stylisé.

## 9.3 Conseils pour réduire le poids et améliorer le temps de chargement

- **Évitez le CSS inutilisé** : Au fil du temps, notre feuille de style peut accumuler des règles qui ne sont plus utilisées. Des outils comme PurifyCSS ou UnCSS peuvent aider à identifier et à supprimer ces règles.

- **Utilisez des sprites CSS pour les images** : Plutôt que de charger de nombreuses petites images individuellement, combinez-les en une seule image (sprite) et utilisez le CSS pour afficher la partie appropriée.

- **Utiliser les propriétés CSS natives** : Là où c'est possible, utilisez les propriétés CSS plutôt que les images. Par exemple, pour les ombres ou les gradients.

- **Optez pour des librairies CSS modulaires** : Si vous utilisez un framework ou une bibliothèque, choisissez uniquement les composants dont vous avez besoin plutôt que l'ensemble complet.

# 10. Tendances actuelles et avenir de CSS

Le monde du développement web est en constante évolution, et le CSS ne fait pas exception. Chaque année, nous voyons apparaître de nouvelles fonctionnalités, optimisations et méthodes pour améliorer la flexibilité, l'efficacité et la créativité de nos designs web. Voici une vue d'ensemble des tendances actuelles et de l'avenir du CSS.

## 10.1 CSS Variables (custom properties)

Les variables CSS, aussi appelées propriétés personnalisées, représentent une avancée majeure dans la manière dont nous écrivons et structurons nos styles.

- **Qu'est-ce que c'est ?** : Il s'agit de valeurs réutilisables que vous pouvez définir dans une feuille de style. Elles commencent par deux tirets et sont accessibles via la fonction `var()`.
  ```css
  :root {
      --main-bg-color: coral;
  }
  
  body {
      background-color: var(--main-bg-color);
  }
  ```

- **Avantages** : Les variables CSS offrent une plus grande flexibilité, permettant des thèmes dynamiques, une meilleure lisibilité et une maintenance facilitée.

## 10.2 Nouveaux modules et spécifications

Le développement de CSS ne s'arrête jamais. Le W3C travaille constamment à introduire de nouveaux modules et spécifications pour améliorer la capacité et la flexibilité du langage.

- **CSS Grid & Subgrid** : Alors que le CSS Grid a révolutionné la manière dont nous réalisons des mises en page, l'introduction de Subgrid offre encore plus de granularité et de contrôle.

- **Logical Properties & Values** : Cette spécification vise à faciliter la création de designs qui respectent les sens de lecture et d'écriture de différentes langues, offrant une meilleure prise en charge de l'internationalisation.

- **Containment** : Il offre un moyen de limiter le calcul du style, du layout ou de la peinture à un élément spécifique et à ses descendants, améliorant potentiellement les performances.

## 10.3 Web animations avec CSS

Les animations en CSS ont fait des bonds impressionnants ces dernières années, rendant possible la création d'effets sophistiqués sans avoir recours à JavaScript.

- **Transitions** : Elles offrent un moyen simple d'interpoler les valeurs CSS sur une durée donnée. Très utile pour les effets de survol ou les changements d'état.

- **Keyframe Animations** : Pour des animations plus complexes, les keyframes offrent la possibilité de définir une séquence d'étapes pour l'animation.
  ```css
  @keyframes slide {
      from { transform: translateX(0); }
      to { transform: translateX(100%); }
  }
  ```

- **Nouveautés en matière d'animation** : Des propriétés comme `motion-path` permettent des animations plus dynamiques en déplaçant les éléments le long de chemins définis.
