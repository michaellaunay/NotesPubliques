---
schema_version: 1
uid: 01M02EX5AW7312ZYMNHY8DBZ1W
titre: CSS
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - developpement-web
  - css
resume: "Cours complet de CSS moderne : cascade, sélecteurs, responsive design, Flexbox, Grid, container queries, nesting, scope, couleurs modernes, animations, accessibilité, performance et architecture CSS."
niveau: debutant
prerequis:
  - "[[HTML]]"
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-08-10
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# CSS

> [!abstract] Objectif
> Écrire du CSS moderne et maintenable : cascade et sélecteurs, responsive design, Flexbox et Grid, container queries, nesting et scope, couleurs modernes, animations et transitions de vue, accessibilité, performance et architecture CSS.

Voir aussi : [[HTML]], [[Javascript]], [[SEO]], [[Ressources pour le web]], [[Visual studio code]].

# Plan du cours

1. Introduction à CSS et état de la plateforme en 2026
2. Ajouter du CSS à une page HTML
3. Syntaxe, valeurs et fonctions CSS
4. Cascade, héritage, spécificité et couches
5. Sélecteurs modernes
6. Modèle de boîte, flux et `display`
7. Dimensions, unités et dimensionnement intrinsèque
8. Typographie
9. Couleurs modernes et thèmes
10. Arrière-plans, bordures, ombres, masques et filtres
11. Propriétés logiques et internationalisation
12. Flexbox
13. Grid et Subgrid
14. Positionnement, z-index et contextes d'empilement
15. CSS Anchor Positioning
16. Responsive design et media queries
17. Container queries
18. Nesting CSS natif
19. `@scope` et isolation des composants
20. Custom properties, design tokens et `@property`
21. Formulaires et composants d'interface
22. Transitions, transformations et animations
23. Animations pilotées par le défilement
24. View Transitions
25. Accessibilité et préférences utilisateur
26. Architecture CSS et maintenabilité
27. Sass, PostCSS, frameworks et CSS natif
28. Performance et pipeline de rendu
29. Compatibilité, Baseline et progressive enhancement
30. Sécurité CSS
31. Outils de développement, lint et tests
32. Travaux pratiques
33. Projet final
34. Checklist et aide-mémoire
35. Références

# 1. Introduction à CSS et état de la plateforme en 2026

## 1.1 Qu'est-ce que CSS ?

CSS signifie **Cascading Style Sheets**. C'est le langage de feuilles de style du Web. Il décrit la présentation d'un document structuré, le plus souvent HTML : couleurs, typographie, espacement, mise en page, adaptations responsive, animations et certains comportements visuels.

CSS ne remplace pas HTML :

- **HTML** décrit le contenu et sa sémantique ;
- **CSS** décrit sa présentation ;
- **JavaScript** ajoute, lorsque nécessaire, de la logique et des interactions programmatiques.

Cette séparation est fondamentale pour la maintenance, l'accessibilité et la réutilisation.

## 1.2 De CSS 1 au CSS moderne

Quelques jalons :

- **CSS Level 1** : première recommandation W3C en 1996 ;
- **CSS Level 2** : 1998 ;
- **CSS 2.1** : consolidation longtemps utilisée comme socle ;
- à partir de ce que l'on a appelé **CSS3**, le langage est découpé en modules qui évoluent indépendamment.

Il est donc trompeur de chercher une version monolithique « CSS4 ». En 2026, CSS est un ensemble de modules : Selectors, Color, Grid, Flexbox, Containment, Cascade, Values & Units, View Transitions, etc.

Le **CSS Snapshot 2026** du W3C rassemble l'état des spécifications considérées comme constituant CSS à cette date. Un snapshot décrit la maturité des spécifications ; il ne garantit pas à lui seul leur disponibilité sur tous les navigateurs utilisés par un projet.

## 1.3 Spécification, implémentation et Baseline

Il faut distinguer trois questions :

1. la fonctionnalité existe-t-elle dans une spécification ?
2. est-elle implémentée dans les moteurs de navigateur ?
3. est-elle suffisamment disponible pour notre public cible ?

Le projet **Web Platform Baseline** aide à répondre à la troisième question. Une fonctionnalité peut être :

- **Widely available** : largement disponible depuis suffisamment longtemps ;
- **Newly available** : disponible dans les versions récentes des principaux moteurs ;
- hors Baseline : support encore incomplet ou trop récent.

Baseline ne remplace pas les tests sur les navigateurs réellement supportés par le projet.

## 1.4 CSS moderne : ce qui a changé

Le CSS de 2026 permet de résoudre nativement de nombreux problèmes qui nécessitaient auparavant JavaScript, Sass ou des frameworks :

- Flexbox et Grid ;
- Subgrid ;
- variables avec les custom properties ;
- cascade layers avec `@layer` ;
- nesting natif ;
- container queries ;
- `:has()` ;
- `@scope` ;
- couleurs OKLCH et `color-mix()` ;
- `light-dark()` ;
- propriétés logiques ;
- `@property` ;
- View Transitions ;
- anchor positioning ;
- animations liées au défilement.

Le but n'est pas d'utiliser toutes les nouveautés, mais de comprendre **quel outil répond le mieux au problème**.

# 2. Ajouter du CSS à une page HTML

## 2.1 Feuille de style externe

C'est la solution normale pour une application ou un site :

```html
<link rel="stylesheet" href="/assets/app.css">
```

Avantages :

- mise en cache ;
- séparation des responsabilités ;
- réutilisation ;
- maintenance facilitée ;
- politique CSP plus simple que des styles inline massifs.

L'attribut `type="text/css"` n'est pas nécessaire en HTML moderne.

## 2.2 Élément `<style>`

Une feuille embarquée peut être utile pour un document autonome ou du CSS critique limité :

```html
<style>
  body {
    font-family: system-ui, sans-serif;
  }
</style>
```

## 2.3 Attribut `style`

```html
<p style="color: rebeccapurple">Bonjour</p>
```

Il est valide, mais son emploi systématique complique :

- la réutilisation ;
- la cascade ;
- les thèmes ;
- les politiques CSP ;
- la maintenance.

On le réserve généralement à des valeurs réellement dynamiques ou à des cas très locaux.

## 2.4 `@import`

```css
@import url("./tokens.css") layer(tokens);
```

`@import` est valide et supporte notamment les layers, les media queries et `supports()`. Pour les ressources critiques, un `<link rel="stylesheet">` reste souvent plus simple à optimiser et à observer.

# 3. Syntaxe, valeurs et fonctions CSS

## 3.1 Règle de style

```css
.card {
  padding: 1rem;
  border-radius: 0.75rem;
}
```

- `.card` est le sélecteur ;
- `padding` et `border-radius` sont des propriétés ;
- `1rem` et `0.75rem` sont des valeurs.

## 3.2 Commentaires

```css
/* Commentaire CSS */
```

CSS ne possède pas de commentaire `//` standard.

## 3.3 At-rules

Les règles commençant par `@` contrôlent une partie du comportement du langage :

```css
@media (width >= 60rem) { }
@supports (display: grid) { }
@layer components { }
@container card (width > 30rem) { }
@scope (.article) { }
@property --angle { }
@keyframes pulse { }
```

## 3.4 Fonctions utiles

```css
width: min(100%, 70rem);
font-size: clamp(1rem, 2vw + 0.5rem, 2rem);
margin-inline: max(1rem, env(safe-area-inset-left));
background: color-mix(in oklch, var(--brand) 80%, white);
```

Fonctions courantes :

- `calc()` ;
- `min()` ;
- `max()` ;
- `clamp()` ;
- `var()` ;
- fonctions de couleur ;
- fonctions de gradients ;
- `url()`.

# 4. Cascade, héritage, spécificité et couches

La cascade est le cœur de CSS. Une grande partie des « bugs CSS » vient d'un modèle mental incomplet de la cascade.

## 4.1 Les étapes conceptuelles

Lorsqu'un navigateur doit choisir une valeur, il tient compte notamment :

1. de la pertinence de la règle pour le média/contexte ;
2. de l'origine et de l'importance ;
3. des cascade layers ;
4. de la spécificité ;
5. de la proximité de portée lorsqu'un scope intervient ;
6. de l'ordre d'apparition pour départager ce qui reste à égalité.

Il ne faut donc pas réduire CSS à « la dernière règle gagne ».

## 4.2 Héritage

Certaines propriétés sont naturellement héritées :

```css
body {
  color: #222;
  font-family: system-ui, sans-serif;
}
```

Les descendants héritent généralement de `color` et `font-family`.

D'autres propriétés, comme `margin` ou `border`, ne sont normalement pas héritées.

Mots-clés globaux :

```css
color: inherit;
color: initial;
color: unset;
color: revert;
color: revert-layer;
```

`revert-layer` est particulièrement utile lorsqu'une architecture utilise `@layer`.

## 4.3 Spécificité

On peut raisonner grossièrement avec trois colonnes :

```text
ID | classes/attributs/pseudo-classes | éléments/pseudo-éléments
```

Exemples :

```css
p {}                    /* 0-0-1 */
.note {}                /* 0-1-0 */
article .note {}        /* 0-1-1 */
#app .note {}           /* 1-1-0 */
```

Éviter de « gagner » en empilant toujours plus de sélecteurs.

## 4.4 `:where()`, `:is()`, `:not()` et `:has()`

`:where()` est très utile en architecture CSS car sa spécificité est toujours nulle :

```css
:where(nav, aside, footer) a {
  color: inherit;
}
```

`:is()`, `:not()` et `:has()` prennent en compte la spécificité de leurs arguments les plus spécifiques.

## 4.5 `!important`

`!important` n'est pas une stratégie d'architecture.

Il peut être légitime pour :

- des feuilles utilisateur d'accessibilité ;
- certaines utilities volontairement prioritaires ;
- un override très explicite dans une architecture maîtrisée.

Mais l'utiliser pour résoudre chaque conflit produit rapidement une escalade.

## 4.6 Cascade layers avec `@layer`

```css
@layer reset, tokens, base, components, utilities;

@layer base {
  a {
    color: var(--link-color);
  }
}

@layer utilities {
  .text-center {
    text-align: center;
  }
}
```

Pour les déclarations **normales**, les layers déclarés plus tard ont une priorité supérieure aux layers précédents. Les styles normaux hors layer ont une priorité supérieure aux styles normaux placés dans un layer.

L'ordre est inversé pour les déclarations `!important`, afin de permettre aux couches fondamentales de protéger certaines règles importantes.

Les layers permettent d'organiser la cascade sans créer une guerre de spécificité.

# 5. Sélecteurs modernes

## 5.1 Sélecteurs simples

```css
article {}        /* type */
.card {}          /* classe */
#main {}          /* ID */
* {}              /* universel */
[data-state] {}   /* attribut */
```

Pour le style applicatif, les classes sont généralement plus réutilisables que les IDs.

## 5.2 Sélecteurs d'attribut

```css
input[type="email"] {}
a[href^="https://"] {}
a[href$=".pdf"] {}
[data-state~="active"] {}
```

## 5.3 Combinateurs

```css
article p {}       /* descendant */
article > p {}     /* enfant direct */
h2 + p {}          /* frère adjacent */
h2 ~ p {}          /* frères suivants */
```

## 5.4 Pseudo-classes d'interaction

```css
a:hover {}
a:focus-visible {}
button:active {}
input:disabled {}
input:checked {}
input:user-invalid {}
```

Préférer `:focus-visible` pour un focus visuel adapté aux interactions clavier, sans supprimer le focus avec `outline: none` sans alternative.

## 5.5 Sélecteurs structurels

```css
li:first-child {}
li:last-child {}
li:nth-child(2n) {}
li:nth-child(3n + 1) {}
```

Avec la syntaxe `of` :

```css
tr:nth-child(even of :not([hidden])) {
  background: var(--stripe);
}
```

## 5.6 `:is()` et `:where()`

```css
:is(h1, h2, h3) {
  text-wrap: balance;
}

:where(.prose) :where(p, ul, ol) {
  max-inline-size: 70ch;
}
```

## 5.7 `:has()` : sélectionner selon les descendants ou voisins

```css
.card:has(img) {
  grid-template-rows: auto 1fr;
}

label:has(input:checked) {
  font-weight: 700;
}
```

`:has()` est parfois surnommé « sélecteur de parent », mais il est plus général : il teste une condition relationnelle.

## 5.8 Pseudo-éléments

```css
.badge::before {
  content: "★";
}

::selection {
  background: var(--selection-bg);
}
```

Le contenu important ne doit pas dépendre uniquement de `content:` lorsque cela nuit à l'accessibilité ou à la sémantique.

# 6. Modèle de boîte, flux et `display`

## 6.1 Box model

Une boîte CSS comprend :

1. content box ;
2. padding ;
3. border ;
4. margin.

Un reset très courant :

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

Avec `border-box`, les dimensions déclarées incluent padding et border.

## 6.2 Flux normal

Le flux normal reste la base. Avant d'utiliser `position: absolute`, il faut se demander si Flexbox ou Grid ne décrit pas mieux la relation entre les éléments.

## 6.3 `display`

Valeurs fréquentes :

```css
display: block;
display: inline;
display: inline-block;
display: flex;
display: grid;
display: flow-root;
display: none;
```

`flow-root` crée notamment un nouveau block formatting context et peut être utile pour contenir des floats ou isoler certains comportements de layout.

## 6.4 Overflow

```css
.panel {
  overflow: auto;
}
```

Attention aux couples `overflow-x` / `overflow-y`, aux scroll containers créés involontairement et aux effets sur `position: sticky`.

## 6.5 Ratio d'aspect

```css
.video {
  aspect-ratio: 16 / 9;
  inline-size: 100%;
}
```

# 7. Dimensions, unités et dimensionnement intrinsèque

## 7.1 Unités absolues et relatives

Unités fréquentes :

- `px` : pixel CSS, pas nécessairement un pixel physique ;
- `em` : relatif à la taille de police du contexte ;
- `rem` : relatif à la taille de police racine ;
- `%` : dépend de la propriété et du containing block ;
- `ch` : métrique approximative liée au caractère `0` ;
- `lh` / `rlh` : hauteur de ligne ;
- `vw` / `vh` : viewport historique ;
- `svh`, `lvh`, `dvh` : variantes utiles sur mobile ;
- `cqw`, `cqh`, `cqi`, `cqb` : unités relatives à un query container.

## 7.2 Tailles intrinsèques

```css
width: min-content;
width: max-content;
width: fit-content;
```

Grid exploite particulièrement les tailles intrinsèques.

## 7.3 `min()`, `max()` et `clamp()`

```css
.wrapper {
  width: min(100% - 2rem, 72rem);
  margin-inline: auto;
}

h1 {
  font-size: clamp(2rem, 5vw, 5rem);
}
```

Le responsive moderne peut souvent être fluide avant même d'ajouter une media query.

# 8. Typographie

## 8.1 Font stack système

```css
body {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  line-height: 1.5;
}
```

## 8.2 Web fonts

```css
@font-face {
  font-family: "InterLocal";
  src: url("/fonts/inter.woff2") format("woff2");
  font-display: swap;
  font-weight: 100 900;
}
```

Préférer WOFF2, limiter les variantes et mesurer l'impact sur LCP/CLS.

## 8.3 Longueur de ligne

```css
.prose {
  max-inline-size: 70ch;
}
```

La lisibilité ne dépend pas seulement de la taille de police : largeur de colonne, hauteur de ligne, contraste et espacement jouent aussi.

## 8.4 `text-wrap`

```css
h1,
h2 {
  text-wrap: balance;
}

.article p {
  text-wrap: pretty;
}
```

`balance` est adapté aux titres courts. `pretty` peut améliorer les paragraphes, avec un coût de calcul supérieur : il faut mesurer avant de l'appliquer partout.

## 8.5 Césure et débordement

```css
.prose {
  overflow-wrap: anywhere;
  hyphens: auto;
}
```

La césure dépend notamment de la langue déclarée en HTML :

```html
<html lang="fr">
```

# 9. Couleurs modernes et thèmes

## 9.1 Syntaxes modernes

```css
color: rgb(20 30 40 / 0.8);
color: hsl(210 50% 30%);
color: oklch(65% 0.18 250);
```

`oklch()` est particulièrement utile pour construire des palettes dont les variations de luminosité sont plus proches de la perception humaine.

## 9.2 `currentColor`

```css
.icon {
  color: var(--accent);
  border: 1px solid currentColor;
  fill: currentColor;
}
```

## 9.3 `color-mix()`

```css
.button {
  --bg: oklch(62% 0.2 250);
  background: var(--bg);
}

.button:hover {
  background: color-mix(in oklch, var(--bg) 85%, black);
}
```

## 9.4 Couleurs relatives

```css
.card {
  --base: oklch(68% 0.16 250);
  --lighter: oklch(from var(--base) calc(l + 0.12) c h);
}
```

## 9.5 Thème clair/sombre avec `light-dark()`

```css
:root {
  color-scheme: light dark;
  --surface: light-dark(white, #151515);
  --text: light-dark(#171717, #f5f5f5);
}

body {
  background: var(--surface);
  color: var(--text);
}
```

Cela réduit certains media queries `prefers-color-scheme` simples.

## 9.6 `contrast-color()`

Fonction devenue Baseline Newly available en 2026 :

```css
.badge {
  --bg: oklch(60% 0.2 250);
  background: var(--bg);
  color: contrast-color(var(--bg));
}
```

En 2026, il faut malgré tout tester le contraste réel de la charte : l'accessibilité ne se résume pas à une fonction automatique.

# 10. Arrière-plans, bordures, ombres, masques et filtres

## 10.1 Backgrounds multiples

```css
.hero {
  background:
    linear-gradient(rgb(0 0 0 / 0.45), rgb(0 0 0 / 0.45)),
    url("hero.avif") center / cover no-repeat;
}
```

## 10.2 Bordures et radius

```css
.card {
  border: 1px solid color-mix(in srgb, currentColor 18%, transparent);
  border-radius: 1rem;
}
```

## 10.3 Ombres

```css
.card {
  box-shadow: 0 0.5rem 2rem rgb(0 0 0 / 0.12);
}
```

Utiliser les ombres avec parcimonie : beaucoup de grandes ombres floutées peuvent augmenter le coût de rendu.

## 10.4 Filtres et masques

```css
.logo {
  filter: grayscale(1);
}
```

```css
.icon {
  mask: url("icon.svg") center / contain no-repeat;
  background: currentColor;
}
```

# 11. Propriétés logiques et internationalisation

Au lieu de penser uniquement gauche/droite, CSS moderne peut raisonner selon l'axe de lecture.

```css
.card {
  margin-inline: auto;
  padding-block: 1rem;
  padding-inline: 1.5rem;
  border-inline-start: 0.25rem solid var(--accent);
}
```

Correspondances fréquentes :

| Physique | Logique |
|---|---|
| `width` | `inline-size` |
| `height` | `block-size` |
| `margin-left/right` | `margin-inline-*` |
| `margin-top/bottom` | `margin-block-*` |
| `left/right` | `inset-inline-*` |

Avec `writing-mode` et `direction`, une interface peut mieux supporter les écritures RTL ou verticales.

# 12. Flexbox

Flexbox est un modèle principalement **unidimensionnel** : une ligne ou une colonne à la fois.

## 12.1 Conteneur flex

```css
.toolbar {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  justify-content: space-between;
}
```

## 12.2 Axes

- main axis : contrôlé principalement par `justify-content` ;
- cross axis : contrôlé principalement par `align-items`.

L'orientation dépend de `flex-direction`.

## 12.3 Taille des items

```css
.sidebar {
  flex: 0 0 18rem;
}

.content {
  flex: 1 1 40rem;
  min-width: 0;
}
```

Le `min-width: 0` est un correctif fréquent lorsque du contenu empêche un item flex de rétrécir.

## 12.4 Wrapping

```css
.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}
```

## 12.5 Quand choisir Flexbox ?

Bon cas :

- barre d'actions ;
- navigation ;
- alignement d'éléments sur un axe ;
- listes de badges ;
- distribution de boutons.

Pour une mise en page bidimensionnelle explicite, Grid est souvent meilleur.

# 13. Grid et Subgrid

## 13.1 Grille de base

```css
.layout {
  display: grid;
  grid-template-columns: 16rem minmax(0, 1fr);
  gap: 2rem;
}
```

## 13.2 Grille responsive sans media query

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr));
  gap: 1rem;
}
```

## 13.3 Zones nommées

```css
.page {
  display: grid;
  grid-template:
    "header header" auto
    "nav    main" 1fr
    "footer footer" auto
    / 16rem 1fr;
}
```

## 13.4 `minmax()`, `auto-fit` et `auto-fill`

- `minmax()` borne une piste ;
- `auto-fit` replie généralement les pistes vides ;
- `auto-fill` conserve le calcul des pistes possibles.

## 13.5 Subgrid

`subgrid` est largement disponible depuis 2023 et résout un problème classique : aligner des descendants sur les pistes du parent.

```css
.cards {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

.card {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid;
}
```

Les titres, contenus et pieds de cartes peuvent alors rester alignés entre cartes.

# 14. Positionnement, z-index et contextes d'empilement

## 14.1 Modes de positionnement

```css
position: static;
position: relative;
position: absolute;
position: fixed;
position: sticky;
```

## 14.2 Sticky

```css
.sidebar {
  position: sticky;
  top: 1rem;
}
```

`sticky` dépend du scroll container et peut sembler « cassé » lorsqu'un ancêtre possède un overflow inattendu.

## 14.3 `z-index` et stacking context

Un grand `z-index` ne peut pas toujours sortir d'un contexte d'empilement parent.

Des propriétés comme certaines valeurs de :

- `position` + `z-index` ;
- `opacity` ;
- `transform` ;
- `filter` ;
- `isolation` ;
- `contain` ;

peuvent créer un stacking context.

Avant d'écrire `z-index: 999999`, inspecter la hiérarchie des contextes.

# 15. CSS Anchor Positioning

Le positionnement par ancre permet d'attacher visuellement un élément positionné à un autre sans calculer sa géométrie en JavaScript. Les briques principales sont devenues Baseline Newly available en 2026.

## 15.1 Déclarer une ancre

```css
.trigger {
  anchor-name: --trigger;
}
```

## 15.2 Associer un élément positionné

```css
.tooltip {
  position: absolute;
  position-anchor: --trigger;
  top: anchor(bottom);
  left: anchor(center);
  translate: -50% 0.5rem;
}
```

## 15.3 `position-area`

```css
.popover {
  position: absolute;
  position-anchor: --trigger;
  position-area: block-end center;
}
```

## 15.4 Taille relative à l'ancre

```css
.dropdown {
  min-inline-size: anchor-size(width);
}
```

## 15.5 Progressive enhancement

Pour des publics comprenant des navigateurs anciens, garder un fallback raisonnable et vérifier le support :

```css
@supports (anchor-name: --a) {
  /* amélioration moderne */
}
```

# 16. Responsive design et media queries

## 16.1 Mobile first

```css
.layout {
  display: grid;
  gap: 1rem;
}

@media (width >= 48rem) {
  .layout {
    grid-template-columns: 16rem 1fr;
  }
}
```

La syntaxe de comparaison rend les breakpoints plus lisibles que certains anciens `min-width`/`max-width` imbriqués.

## 16.2 Breakpoints pilotés par le contenu

Ne choisir pas un breakpoint parce qu'un téléphone particulier fait 390 px. Choisir le moment où **la mise en page cesse de fonctionner correctement**.

## 16.3 Préférences utilisateur

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto;
  }
}
```

Autres media features utiles selon le besoin :

- `prefers-color-scheme` ;
- `prefers-contrast` ;
- `forced-colors` ;
- `pointer` / `any-pointer` ;
- `hover` / `any-hover` ;
- `orientation`.

Ne jamais supposer qu'un petit écran implique forcément un écran tactile.

## 16.4 Impression

```css
@media print {
  nav,
  .no-print {
    display: none;
  }

  a[href]::after {
    content: " (" attr(href) ")";
  }
}
```

# 17. Container queries

Les media queries répondent au viewport. Les container queries répondent au **composant qui contient l'élément**.

## 17.1 Query container

```css
.card-list {
  container-type: inline-size;
  container-name: cards;
}
```

## 17.2 Query sur la taille

```css
@container cards (width >= 36rem) {
  .card {
    grid-template-columns: 10rem 1fr;
  }
}
```

Le composant devient réutilisable dans une sidebar comme dans le contenu principal.

## 17.3 Unités de container

```css
.card-title {
  font-size: clamp(1rem, 4cqi, 2rem);
}
```

## 17.4 Style queries

En 2026, les style queries permettent notamment de réagir à une custom property portée par le container :

```css
.theme-region {
  --density: compact;
}

@container style(--density: compact) {
  .item {
    padding-block: 0.25rem;
  }
}
```

Les spécifications prévoient plus de possibilités, mais les implémentations courantes doivent être vérifiées : les custom properties constituent le cas portable principal actuel.

# 18. Nesting CSS natif

Le nesting natif est largement disponible et réduit le besoin de Sass pour ce cas précis.

```css
.card {
  padding: 1rem;

  & h2 {
    margin-block-start: 0;
  }

  &:hover {
    box-shadow: 0 0.5rem 1.5rem rgb(0 0 0 / 0.12);
  }

  @media (width >= 50rem) {
    padding: 1.5rem;
  }
}
```

## 18.1 Le rôle de `&`

`&` représente le sélecteur parent du contexte imbriqué.

```css
.button {
  .toolbar & {
    flex: none;
  }
}
```

Nesting ne signifie pas qu'il faut reproduire la profondeur du DOM. Des sélecteurs trop imbriqués restent difficiles à maintenir.

# 19. `@scope` et isolation des composants

`@scope` est devenu Baseline Newly available en mars 2026.

## 19.1 Scope simple

```css
@scope (.article) {
  h2 {
    color: var(--heading-color);
  }
}
```

La règle cible les `h2` situés dans le sous-arbre `.article` sans exiger un sélecteur `.article h2` partout.

## 19.2 Limite de scope

```css
@scope (.article) to (.comments) {
  a {
    text-decoration-thickness: 0.12em;
  }
}
```

La zone `.comments` peut être exclue du scope.

## 19.3 Pourquoi c'est utile ?

- styles de composants ;
- réduction de la spécificité ;
- réduction du couplage à la structure DOM ;
- intégration de contenu tiers ;
- migration progressive de vieilles feuilles de style.

`@scope`, Shadow DOM et CSS Modules ne répondent pas exactement au même besoin : ils peuvent être complémentaires.

# 20. Custom properties, design tokens et `@property`

## 20.1 Custom properties

```css
:root {
  --color-brand: oklch(62% 0.18 255);
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --radius-md: 0.75rem;
}

.button {
  background: var(--color-brand);
  border-radius: var(--radius-md);
}
```

Les custom properties participent à la cascade et héritent par défaut.

## 20.2 Fallback

```css
color: var(--component-color, var(--color-text, black));
```

## 20.3 Tokens sémantiques

Préférer :

```css
--color-surface-danger
--color-text-muted
```

à des tokens exclusivement liés à une valeur :

```css
--red-500
```

Les deux niveaux peuvent coexister : primitives puis tokens sémantiques.

## 20.4 `@property`

`@property` permet de typer une custom property, définir son héritage et sa valeur initiale :

```css
@property --progress {
  syntax: "<number>";
  inherits: false;
  initial-value: 0;
}
```

On peut alors l'animer de façon typée :

```css
.meter {
  --progress: 0;
  transition: --progress 400ms ease;
}
```

# 21. Formulaires et composants d'interface

## 21.1 Ne pas supprimer les comportements natifs sans raison

Les contrôles HTML fournissent :

- focus ;
- clavier ;
- états disabled/checked ;
- intégration avec les technologies d'assistance.

Un `<button>` stylé est presque toujours préférable à un `<div role="button">` recréé manuellement.

## 21.2 `accent-color`

```css
:root {
  accent-color: var(--brand);
}
```

## 21.3 `appearance`

```css
select.custom {
  appearance: none;
}
```

À utiliser avec prudence : supprimer l'apparence native oblige à reconstruire certains indices visuels.

## 21.4 `field-sizing`

Devenu Baseline Newly available en 2026 :

```css
textarea.auto-grow {
  field-sizing: content;
  min-block-size: 4lh;
  max-block-size: 12lh;
}
```

Cela permet à certains champs de s'ajuster à leur contenu sans JavaScript.

## 21.5 États utiles

```css
input:user-invalid {
  border-color: var(--danger);
}

input:focus-visible {
  outline: 3px solid var(--focus);
  outline-offset: 2px;
}
```

# 22. Transitions, transformations et animations

## 22.1 Transition

```css
.button {
  transition:
    background-color 150ms ease,
    transform 150ms ease;
}

.button:hover {
  transform: translateY(-2px);
}
```

Éviter `transition: all` par défaut : on risque d'animer des propriétés non prévues et coûteuses.

## 22.2 Transformations

```css
transform: translateX(1rem) rotate(2deg) scale(1.02);
```

Les transformations n'affectent généralement pas le layout des autres éléments.

## 22.3 Keyframes

```css
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.status {
  animation: pulse 1.5s ease-in-out infinite;
}
```

## 22.4 Respecter `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  .status {
    animation: none;
  }
}
```

Une animation n'est pas seulement décorative : elle peut provoquer inconfort ou nausées chez certains utilisateurs.

# 23. Animations pilotées par le défilement

Le module Scroll-driven Animations permet de relier la progression d'une animation au scroll, sans listener JavaScript exécuté à chaque événement.

## 23.1 Progression globale

```css
.progress {
  transform-origin: left;
  animation: grow linear;
  animation-timeline: scroll();
}

@keyframes grow {
  from { transform: scaleX(0); }
  to { transform: scaleX(1); }
}
```

## 23.2 View timeline

```css
.reveal {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 10% cover 35%;
}

@keyframes reveal {
  from {
    opacity: 0;
    transform: translateY(2rem);
  }
  to {
    opacity: 1;
    transform: none;
  }
}
```

## 23.3 Prudence

- vérifier la compatibilité du public cible ;
- fournir un rendu lisible sans animation ;
- respecter `prefers-reduced-motion` ;
- ne pas attacher une information indispensable uniquement à un effet de scroll.

# 24. View Transitions

Les View Transitions de même document sont Baseline Newly available depuis 2025.

## 24.1 Côté CSS

```css
.card-image {
  view-transition-name: selected-image;
}

::view-transition-old(selected-image),
::view-transition-new(selected-image) {
  animation-duration: 250ms;
}
```

## 24.2 Déclenchement côté JavaScript

```js
document.startViewTransition(() => {
  updateDOM();
});
```

CSS contrôle ensuite une grande partie de la présentation de la transition.

## 24.3 Cross-document

Les transitions entre documents peuvent également être activées dans des scénarios compatibles. Il faut vérifier support, navigation et stratégie de fallback.

# 25. Accessibilité et préférences utilisateur

## 25.1 Focus visible

Ne pas faire :

```css
button:focus {
  outline: none;
}
```

sans alternative.

Faire par exemple :

```css
:focus-visible {
  outline: 3px solid var(--focus-color);
  outline-offset: 3px;
}
```

## 25.2 Contraste

Vérifier le contraste avec les critères WCAG applicables. Une jolie palette OKLCH n'est pas automatiquement accessible.

## 25.3 Zoom et reflow

Éviter les hauteurs fixes pour de longs contenus et les interfaces qui cassent lorsque :

- le texte est zoomé ;
- la taille de police utilisateur augmente ;
- la fenêtre devient étroite.

## 25.4 Mouvement

```css
@media (prefers-reduced-motion: reduce) {
  html:focus-within {
    scroll-behavior: auto;
  }
}
```

## 25.5 Forced colors

```css
@media (forced-colors: active) {
  .status-dot {
    forced-color-adjust: auto;
  }
}
```

Tester réellement sous mode contraste élevé plutôt que désactiver systématiquement les couleurs forcées.

## 25.6 Le CSS ne répare pas une mauvaise sémantique HTML

Un `<div>` visuellement transformé en titre n'est pas un `<h2>`. Revenir au cours [[HTML]] lorsqu'un problème relève d'abord de la structure du document.

# 26. Architecture CSS et maintenabilité

## 26.1 Objectifs

Une architecture CSS saine cherche à réduire :

- la spécificité inutile ;
- les dépendances au DOM ;
- les overrides implicites ;
- les effets de bord ;
- la duplication.

## 26.2 Exemple avec cascade layers

```css
@layer reset, tokens, base, objects, components, utilities, overrides;
```

Chaque projet n'a pas besoin de sept layers. L'important est que l'ordre soit explicite.

## 26.3 BEM

```html
<article class="card card--featured">
  <h2 class="card__title">Titre</h2>
</article>
```

BEM reste utile lorsque le projet a besoin d'une convention de nommage explicite.

## 26.4 OOCSS et SMACSS

Ces méthodes restent intéressantes historiquement et conceptuellement :

- séparer structure et apparence ;
- identifier les catégories de règles ;
- limiter le couplage.

Mais `@layer`, `@scope`, nesting, custom properties et les outils de composants permettent aujourd'hui des architectures plus simples dans de nombreux projets.

## 26.5 CSS Modules et styles de composants

Dans certains frameworks/build systems :

```css
/* Button.module.css */
.root { }
.icon { }
```

Le build transforme les classes pour éviter les collisions. Ceci est un mécanisme d'outil, différent de `@scope` natif.

## 26.6 Design tokens

Séparer :

```text
valeurs primitives
    ↓
tokens sémantiques
    ↓
tokens de composants
```

Exemple :

```css
:root {
  --blue-600: oklch(55% 0.18 250);
  --color-action: var(--blue-600);
  --button-primary-bg: var(--color-action);
}
```

# 27. Sass, PostCSS, frameworks et CSS natif

## 27.1 Sass

Sass reste utile pour :

- génération de code ;
- mixins ;
- fonctions de build ;
- boucles ;
- compatibilité avec des codebases existantes.

Mais plusieurs anciens motifs Sass ont aujourd'hui des équivalents natifs :

| Besoin historique | CSS moderne |
|---|---|
| variables simples | custom properties |
| nesting | nesting natif |
| thèmes runtime | custom properties + `light-dark()` |
| calculs | `calc()`, `min()`, `max()`, `clamp()` |
| isolation partielle | `@scope`, layers, modules outillés |

Une custom property diffère d'une variable Sass : elle existe **au runtime** et participe à la cascade.

## 27.2 PostCSS

PostCSS est une plateforme de transformation CSS. Selon les plugins, il peut :

- autoprefixer ;
- transformer certaines syntaxes ;
- lint/optimiser ;
- générer des variantes.

Ne pas ajouter une chaîne de build complexe sans bénéfice mesuré.

## 27.3 Frameworks

Exemples de stratégies :

- Bootstrap : composants et système de design préconçus ;
- Tailwind CSS : approche utility-first ;
- design system interne : composants/tokens propres au produit.

Le bon choix dépend de l'équipe, de la durée de vie du produit, de la personnalisation et de la gouvernance du design.

# 28. Performance et pipeline de rendu

## 28.1 Ce que fait grossièrement le navigateur

```text
HTML → DOM
CSS  → CSSOM
      ↓
 style calculation
      ↓
 layout
      ↓
 paint
      ↓
 compositing
```

Une modification CSS peut déclencher différentes portions de cette chaîne selon la propriété et le contexte.

## 28.2 Réduire le CSS inutile

- supprimer les règles mortes ;
- découper par routes/composants si le build le permet ;
- éviter les frameworks entiers pour quelques classes ;
- compresser les réponses HTTP ;
- minifier en production.

## 28.3 `contain`

```css
.widget {
  contain: layout paint;
}
```

`contain` déclare qu'une partie du rendu peut être isolée. Il faut choisir les valeurs selon la sémantique réelle du composant.

## 28.4 `content-visibility`

```css
.article-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 500px;
}
```

Cela peut économiser du travail pour du contenu hors écran. Tester l'accessibilité, la recherche dans la page et le comportement de layout sur les navigateurs ciblés.

## 28.5 `will-change`

```css
.dragging {
  will-change: transform;
}
```

`will-change` est un **hint coûteux**, pas une propriété à mettre sur tous les éléments. L'utiliser juste avant un changement prévisible et la retirer lorsque possible.

## 28.6 CSS critique

Éviter les recettes anciennes de « chargement async du CSS » qui produisent du FOUC ou compliquent la priorité réseau. Commencer par :

- une feuille externe cacheable ;
- compression ;
- suppression du CSS inutile ;
- découpage raisonné ;
- mesure avec les DevTools et les Core Web Vitals.

# 29. Compatibilité, Baseline et progressive enhancement

## 29.1 Stratégie

1. définir les navigateurs/versions réellement supportés ;
2. consulter Baseline et les tables de compatibilité ;
3. construire un fallback utilisable ;
4. ajouter les améliorations modernes ;
5. tester.

## 29.2 `@supports`

```css
.card {
  display: block;
}

@supports (display: grid) {
  .card {
    display: grid;
  }
}
```

## 29.3 Exemple `@scope` avec fallback naturel

```css
/* Fallback compris par les navigateurs anciens. */
.component h2 {
  color: var(--title);
}

/* Un navigateur qui ne connaît pas @scope ignore ce bloc. */
@scope (.component) {
  h2 {
    color: var(--title);
  }
}
```

Une règle `@supports` ne permet pas de tester directement la prise en charge d'une at-rule inconnue. Ici, le comportement d'erreur de CSS fournit naturellement le fallback : un navigateur ancien ignore le bloc `@scope` et conserve la règle classique. Le fallback n'a pas besoin d'être visuellement identique : il doit rester fonctionnel et lisible.

## 29.4 Baseline 2026 : quelques exemples à connaître

Parmi les fonctionnalités devenues Newly available en 2026 figurent notamment :

- `@scope` ;
- plusieurs briques de CSS Anchor Positioning ;
- `field-sizing` ;
- container style queries ;
- `contrast-color()`.

Ce statut signifie « utilisable dans les versions récentes des principaux moteurs », pas « compatible avec tous les navigateurs historiques encore en circulation ».

# 30. Sécurité CSS

CSS semble moins risqué que JavaScript, mais une feuille de style non fiable ne doit pas être considérée comme inoffensive.

## 30.1 CSS arbitraire

Une feuille CSS peut notamment :

- masquer ou déplacer des informations ;
- superposer des éléments ;
- déclencher le chargement de ressources avec certaines propriétés `url()` ;
- tromper visuellement l'utilisateur ;
- augmenter fortement le coût de rendu.

Ne pas injecter directement du CSS fourni par un utilisateur non fiable dans une page privilégiée.

## 30.2 Custom properties issues de données

Préférer des valeurs validées :

```html
<div style="--progress: 42%"></div>
```

si `42%` a été borné/validé côté application, plutôt que concaténer une chaîne CSS arbitraire.

## 30.3 Content Security Policy

Une CSP peut contrôler les sources de styles via `style-src`. Les styles inline peuvent nécessiter nonce/hash ou des politiques moins strictes selon l'architecture.

Voir également [[Sécurité avancée sous Linux]], [[HTML]] et [[HTTP]] pour la défense en profondeur autour de l'application.

# 31. Outils de développement, lint et tests

## 31.1 DevTools

Savoir utiliser :

- panneau Styles ;
- règle barrée et provenance ;
- computed styles ;
- visualiseur box model ;
- Flexbox/Grid overlays ;
- simulation responsive ;
- media features ;
- performance ;
- accessibility tree.

## 31.2 Diagnostiquer une règle qui ne s'applique pas

Questions dans l'ordre :

1. le sélecteur matche-t-il ?
2. la déclaration est-elle valide ?
3. une autre origine/layer gagne-t-elle ?
4. la spécificité est-elle supérieure ailleurs ?
5. un `!important` intervient-il ?
6. la propriété est-elle héritée ?
7. la propriété a-t-elle un effet dans ce contexte de layout ?
8. un parent crée-t-il un contexte particulier ?

## 31.3 Stylelint

Exemple de dépendance :

```bash
npm install --save-dev stylelint stylelint-config-standard
```

Un lint permet de rendre les erreurs et conventions répétables en CI.

## 31.4 Formatage

Prettier peut formater CSS/SCSS. Le formatage automatique réduit les discussions de style sans intérêt architectural.

## 31.5 Tests visuels

Pour un design system, compléter les tests unitaires/DOM par :

- screenshots de référence ;
- tests multi-viewports ;
- tests thème clair/sombre ;
- tests zoom ;
- tests reduced motion ;
- tests forced colors lorsque pertinents.

# 32. Travaux pratiques

## TP 1 — Cascade et spécificité

Créer un document contenant :

- styles navigateur ;
- feuille externe ;
- deux classes ;
- un ID ;
- un style inline ;
- une règle `!important`.

Expliquer pour chaque propriété pourquoi une valeur gagne. Remplacer ensuite les hacks de spécificité par une architecture avec `@layer`.

## TP 2 — Carte accessible

Construire une carte d'article avec :

- HTML sémantique ;
- image responsive ;
- focus visible ;
- texte qui reste lisible à 200 % de zoom ;
- thème clair/sombre.

Interdiction d'utiliser un framework.

## TP 3 — Flexbox

Créer :

- une barre de navigation ;
- une toolbar qui wrappe ;
- une disposition sidebar/contenu simple.

Expliquer pourquoi Flexbox est adapté à chacun des deux premiers cas et discuter le troisième.

## TP 4 — Grid et Subgrid

Créer six cartes dont :

- le titre ;
- le corps ;
- le bouton final

s'alignent horizontalement entre cartes grâce à Grid/Subgrid.

## TP 5 — Responsive sans breakpoints arbitraires

Construire une grille avec :

```css
grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr));
```

Puis n'ajouter une media query que lorsqu'un problème de contenu concret le justifie.

## TP 6 — Container query

Créer un composant `ProfileCard` utilisable :

- dans une sidebar de 18rem ;
- dans un contenu principal de 50rem ;
- dans une grille.

Le composant doit modifier sa disposition d'après **sa propre largeur**, pas celle du viewport.

## TP 7 — CSS nesting et `@scope`

Partir d'une feuille avec des sélecteurs :

```css
.dashboard .panel .header .button { }
```

La refactorer avec :

- classes plus simples ;
- nesting raisonnable ;
- `@scope` ;
- `:where()` lorsque pertinent.

Mesurer la réduction de spécificité.

## TP 8 — Design tokens et thèmes

Créer :

- palette primitive en OKLCH ;
- tokens sémantiques ;
- tokens de composants ;
- thème clair/sombre avec `light-dark()` ;
- focus visible indépendant des couleurs de marque.

## TP 9 — Animation accessible

Créer une animation de carte :

- transform/opacity uniquement lorsque possible ;
- fallback sans mouvement ;
- désactivation/adaptation avec `prefers-reduced-motion`.

## TP 10 — Scroll-driven animation

Créer une barre de progression de lecture pilotée par `animation-timeline: scroll()`.

Le contenu doit rester parfaitement utilisable si la fonctionnalité n'est pas supportée.

## TP 11 — Anchor Positioning

Créer un tooltip ou menu associé à un bouton avec :

- `anchor-name` ;
- `position-anchor` ;
- `anchor()` ou `position-area` ;
- fallback simple.

Comparer avec une implémentation JavaScript classique et expliquer ce que CSS ne gère pas à lui seul (état, événements, accessibilité du comportement).

## TP 12 — Audit d'une feuille existante

Sur une vraie feuille CSS :

1. trouver les IDs utilisés uniquement pour la spécificité ;
2. trouver les `!important` ;
3. trouver les couleurs répétées ;
4. trouver les unités physiques gauche/droite qui peuvent devenir logiques ;
5. identifier les media queries dépendantes du viewport qui devraient être des container queries ;
6. repérer les animations incompatibles avec reduced motion ;
7. repérer le CSS mort ;
8. proposer une architecture `@layer`.

# 33. Projet final

Construire l'interface responsive d'une petite application de gestion de projets.

## 33.1 Fonctionnalités

L'application contient :

- header ;
- navigation ;
- liste de projets ;
- cartes ;
- formulaire ;
- dialogue ou popover natif côté HTML lorsque pertinent ;
- tableau ;
- notifications ;
- thème clair/sombre.

## 33.2 Contraintes CSS

Utiliser obligatoirement :

- `@layer` ;
- custom properties ;
- au moins une grille ;
- Flexbox ;
- `subgrid` lorsque pertinent ;
- au moins une container query ;
- nesting natif ;
- `@scope` pour au moins un composant ;
- propriétés logiques ;
- `clamp()` ;
- focus visible ;
- `prefers-reduced-motion` ;
- progressive enhancement pour au moins une fonctionnalité 2026.

## 33.3 Architecture proposée

```css
@layer reset, tokens, base, layout, components, utilities;
```

Arborescence possible :

```text
styles/
├── reset.css
├── tokens.css
├── base.css
├── layout.css
├── components/
│   ├── button.css
│   ├── card.css
│   ├── form.css
│   └── project-list.css
└── utilities.css
```

## 33.4 Critères d'évaluation

- correction de la cascade ;
- absence de guerre de spécificité ;
- responsive par le contenu ;
- accessibilité clavier ;
- compatibilité avec zoom ;
- respect du reduced motion ;
- maintenabilité ;
- performance ;
- progressive enhancement ;
- qualité de la documentation.

# 34. Checklist et aide-mémoire

## 34.1 Avant de créer une nouvelle règle

- Le problème relève-t-il du HTML ou du CSS ?
- Une règle existante répond-elle déjà au besoin ?
- Le sélecteur décrit-il un composant ou dépend-il trop du DOM ?
- Faut-il une classe, un attribut d'état, `:has()` ou une pseudo-classe native ?
- La règle appartient-elle au bon layer ?

## 34.2 Layout

- **une dimension** → penser Flexbox ;
- **deux dimensions** → penser Grid ;
- alignement de descendants avec la grille parente → Subgrid ;
- adaptation au composant → container query ;
- adaptation au viewport/préférences → media query ;
- popup lié géométriquement à un élément → envisager Anchor Positioning.

## 34.3 Cascade

- préférer une faible spécificité ;
- utiliser `:where()` pour des defaults facilement surchargeables ;
- structurer avec `@layer` ;
- éviter les IDs pour gagner la cascade ;
- considérer `revert-layer` ;
- documenter les rares `!important`.

## 34.4 Responsive

- ne pas supposer les périphériques ;
- dimensionnement fluide d'abord ;
- breakpoints motivés par le contenu ;
- tester les grands zooms ;
- utiliser les unités viewport modernes lorsque la barre navigateur mobile compte ;
- préférer les propriétés logiques.

## 34.5 Accessibilité

- conserver un focus visible ;
- contraste suffisant ;
- reduced motion ;
- forced colors ;
- pas de contenu indispensable uniquement généré en CSS ;
- ne pas remplacer la sémantique HTML par du visuel.

## 34.6 Performance

- mesurer avant d'optimiser ;
- supprimer le CSS mort ;
- éviter `will-change` global ;
- limiter grandes ombres/filtres si elles posent problème ;
- charger peu de variantes de fonts ;
- préférer les primitives natives simples ;
- observer les Core Web Vitals.

## 34.7 Fonctions à retenir

```css
var(--token)
calc(...)
min(...)
max(...)
clamp(...)
color-mix(...)
light-dark(...)
oklch(...)
```

## 34.8 At-rules à retenir

```css
@media
@supports
@layer
@container
@scope
@property
@keyframes
@font-face
```

## 34.9 Sélecteurs à retenir

```css
:is(...)
:where(...)
:not(...)
:has(...)
:nth-child(... of ...)
:focus-visible
:user-invalid
```

# 35. Références

Sources de référence à privilégier :

- W3C CSS Snapshot 2026 : https://www.w3.org/TR/css-2026/
- W3C — travail courant du CSS Working Group : https://www.w3.org/Style/CSS/current-work
- MDN CSS : https://developer.mozilla.org/docs/Web/CSS
- MDN — Cascade layers : https://developer.mozilla.org/docs/Web/CSS/@layer
- MDN — Container queries : https://developer.mozilla.org/docs/Web/CSS/CSS_containment/Container_queries
- MDN — CSS nesting : https://developer.mozilla.org/docs/Web/CSS/CSS_nesting
- MDN — `@scope` : https://developer.mozilla.org/docs/Web/CSS/@scope
- MDN — CSS anchor positioning : https://developer.mozilla.org/docs/Web/CSS/CSS_anchor_positioning
- MDN — Scroll-driven animations : https://developer.mozilla.org/docs/Web/CSS/CSS_scroll-driven_animations
- MDN — View Transition API : https://developer.mozilla.org/docs/Web/API/View_Transition_API
- web.dev — Baseline : https://web.dev/baseline

> [!important]
> Le statut d'une spécification et sa compatibilité navigateur évoluent. Pour une fonctionnalité récente, vérifier **la documentation MDN/Baseline au moment du déploiement** plutôt que mémoriser définitivement un numéro de version de navigateur.
