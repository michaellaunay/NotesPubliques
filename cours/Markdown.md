## Introduction

Markdown est un langage de balisage léger créé en 2004 par John Gruber et Aaron Swartz pour Reddit. Le nom du langage est un clin d’œil aux langages de type "markup" (à balises) comme le HTML et le XHTML en soulignant que "markdown" est simple, a moins d’éléments structurants, et peut quand même être compris par un humain en l’absence de mise en forme. 

Markdown permet de faire de la mise en forme basique d'un texte brut sans utiliser un éditeur de texte. En conséquence, une fois les règles de formatage de base apprises, l'auteur d'un texte peut se concentrer pleinement sur le fond et non la forme de celui-ci, tout en conservant la sémantique des titres, paragraphes, italique, gras, liens, et exemples de code. 

Markdown est devenu très populaire pour la rédaction de documents sur Internet, notamment pour la création de contenu sur les réseaux sociaux et les blogs. Il est très souvent utilisé pour rédiger la documentation des programmes informatiques, la prise de notes, les discussions, les tickets sur les outils de suivi de bogues ou de tâches, etc.
## Formatage

Parce qu'il est un langage de balisage léger, Markdown permet de créer des documents structurés en utilisant des caractères spéciaux pour indiquer les titres, les listes, les liens, etc.

Voici quelques exemples de balises courantes utilisées dans Markdown :

### Titres
Utilisons un ou plusieurs `#` devant notre titre pour indiquer le niveau de titre. Par exemple :
- `# Titre 1` est un titre de niveau 1
- `## Titre 2` est un titre de niveau 2
- `### Titre 3` est un titre de niveau 3, et ainsi de suite jusqu'à `###### Titre 6`.
### Gras, italique et surlignage
Utilisons des astérisques ou des soulignements pour mettre en **gras** ou en _italique_. Par exemple :
- **Gras** : `**texte en gras**` ou `__texte en gras__`
- _Italique_ : `*texte en italique*` ou `_texte en italique_`
- **_Gras et italique_** : `***texte en gras et italique***` ou `___texte en gras et italique___`
- ==Surlignage==  `==Texte surligné==`

### Listes
Utilisons des astérisques, des tirets ou des chiffres suivis d'un point pour créer des listes à puces ou numérotées. Par exemple :
- Liste à puces :
  - `* Élément 1`
  - `* Élément 2`
  - `- Élément 3`
- Liste numérotée :
  1. `1. Élément 1`
  2. `2. Élément 2`

### Liens
Utilisons des crochets pour indiquer le texte du lien, suivi de l'URL entre parenthèses. Par exemple :
- `[texte du lien](http://example.com)`

### Adresse de courriel
Pour mettre en forme une adresse de courriel, il suffit d'encadrer celle-ci par un supérieure et un inférieure `<michaellaunay@logikascium.com>` ce qui donne  <michaellaunay@logikascium.com>.

### Images
Utilisons le même format que pour les liens, mais ajoutons un point d'exclamation devant. Par exemple :
- `![texte alternatif](http://example.com/image.jpg)`

Pour créer des images cliquables qui envoi sur des liens internet, il faut écrire `[![texte alternatif](http://example.com/image.jpg)]()

### Citations
Utilisons le symbole `>` pour indiquer une citation. Par exemple :
- `> Ceci est une citation.`

### Code
Pour insérer du code en ligne, utilisons les accents graves. Par exemple :
- Code en ligne : `` `print("Bonjour, monde!")` ``
Pour des blocs de code, utilisons trois accents graves avant et après le bloc de code :
\```
def bonjour():
    print("Bonjour, monde!")
\```

### Tableaux
Nous pouvons créer des tableaux en utilisant des tirets et des pipes pour séparer les colonnes et les lignes. Par exemple :
\```
| Colonne 1 | Colonne 2 |
|-----------|-----------|
| Ligne 1   | Donnée 1  |
| Ligne 2   | Donnée 2  |
\```
ce qui donne

| Colonne 1 | Colonne 2 |
|-----------|-----------|
| Ligne 1   | Donnée 1  |
| Ligne 2   | Donnée 2  |

Le texte des colonnes est aligné à droite par défaut, ou si l'on met deux points `:` au début des tirets `-` d'une colonne.
Pour aligner à droite, il faut de mettre deux points `:` à la fin des tirets `-` de la seconde ligne du tableau.
Pour centrer le texte il faut mettre deux points `:` en début est fin des tirets d'un colonne.

\```
|Colonne un|Colonne deux| Colonne trois |
|-----------:|:-----------:|-----------:|
|Ligne 1|Donnée 1.1|Donnée 1.2|
|Ligne 2|Donnée 2.1|Donnée 2.2|
\```
ce qui donne

| Colonne un | Colonne deux | Colonne trois |
| ---------: | :----------: | ------------: |
|    Ligne 1 |  Donnée 1.1  |    Donnée 1.2 |
|    Ligne 2 |  Donnée 2.1  |    Donnée 2.2 |

### Listes imbriquées
Markdown permet également de créer des listes imbriquées. Par exemple :
- Élément 1
  - Sous-élément 1
    - Sous-sous-élément 1
  - Sous-élément 2
- Élément 2

### Séparateurs
Nous pouvons ajouter des lignes horizontales en utilisant trois ou plus d'astérisques, de tirets ou de soulignements. Par exemple :
- `---`
- `***`
- `___`

### Autres éléments
Markdown permet d'utiliser d'autres éléments pour enrichir les documents :
- **Blocs de citation multiples** : Pour des citations imbriquées, utilisez `>` plusieurs fois.
- **Blocs de code avec syntaxe** : Certains éditeurs supportent la coloration syntaxique. Pour l'activer, ajoutez le nom du langage après les accents graves : `\```python`
- **Footnotes** : Utilisons des footnotes pour les références détaillées[^1].

[^1]: Ceci est une footnote.

### Limitations
Il est important de noter que Markdown est un langage de balisage, et non un langage de programmation, donc il ne possède pas de capacités de programmation telles que les boucles ou les conditions. Il est simple à utiliser et à apprendre, et de nombreux éditeurs de texte et plateformes de blogs prennent en charge Markdown.

## Apports de la variante github de markdown
La variante de Markdown utilisée par GitHub, appelée "GitHub Flavored Markdown" (GFM), apporte quelques fonctionnalités supplémentaires par rapport à la version standard de Markdown.

Voici quelques exemples de ces fonctionnalités :

**Tabulations de code** : GFM permet de créer des blocs de code en utilisant des tabulations ou des espaces. Cela permet de mettre en évidence le code de manière plus claire, en le séparant du reste du contenu.

**Mentions d'utilisateur** : GFM permet de mentionner des utilisateurs en utilisant le signe @ suivi du nom d'utilisateur. Cela permet de notifier les utilisateurs concernés lorsqu'ils sont mentionnés dans un document.

**Checklists** : GFM permet de créer des listes à puces avec des cases à cocher. Cela permet de créer des listes de tâches à effectuer de manière claire et concise.

**Emojis** : GFM prend en charge l'utilisation d'emoji en utilisant leur nom court entre deux points : :smile:

**Task Lists** : GFM permet de créer des listes de tâches avec des cases à cocher, cela permet de créer des listes de tâches à effectuer de manière claire et concise.

Il est important de noter que ces fonctionnalités ne sont pas supportées par tous les éditeurs de markdown, il faut vérifier si ces fonctionnalités sont supportées par l'éditeur que vous utilisons.

En résumé, la variante GFM de Markdown utilisée par GitHub ajoute des fonctionnalités supplémentaires pour la mise en forme de code, les mentions d'utilisateur, les checklists, les emojis et les listes de tâches, qui peuvent être utiles pour les projets de développement logiciel et les projets collaboratifs en général.

liens:
https://docs.github.com/fr/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax

## Apports de la variante obsidian de markdown
Obsidian utilise sa propre variante de Markdown qui ajoute des fonctionnalités supplémentaires pour la prise en charge des liens, des images et des notes de bas de page.

Voici quelques exemples de ces fonctionnalités :

**Liens** : Obsidian prend en charge la création de liens en utilisant le format standard Markdown, mais il ajoute également la possibilité de créer des liens vers des fichiers et des dossiers locaux en utilisant la syntaxe [[nom_fichier]] ou [[nom_dossier/nom_fichier]].

**Images** : Obsidian prend en charge l'ajout d'images en utilisant la syntaxe standard Markdown, mais il permet également d'insérer des images à partir de fichiers locaux en utilisant la syntaxe ![[nom_fichier]]

**Notes de bas de page** : Obsidian permet de créer des notes de bas de page en utilisant la syntaxe [^note]. Cela permet d'ajouter des informations supplémentaires ou des références à un document sans perturber la mise en page principale.

**Tags** : Obsidian permet d'ajouter des tags à des notes en utilisant la syntaxe #tag. Cela permet de retrouver facilement des notes en utilisant ces tags.

**Liens automatiques** : Obsidian permet de créer automatiquement des liens vers les notes qui contiennent des mots ou des phrases identiques en utilisant la syntaxe [[mot ou phrase]].

Ces fonctionnalités supplémentaires permettent une meilleure intégration avec les fonctionnalités de gestion de fichiers et de notes de Obsidian. Il est important de noter que ces fonctionnalités ne sont pas supportées par tous les éditeurs de markdown, il faut vérifier si ces fonctionnalités sont supportées par l'éditeur que vous utilisons.

En résumé, la variante de Markdown utilisée par Obsidian ajoute des fonctionnalités pour la prise en charge des liens, des images, des notes de bas de page, des tags et des liens automatiques, qui peuvent être utiles pour la gestion de notes et de documents.

## Extensions
Il existe plusieurs extensions de Markdown qui permettent d'ajouter des fonctionnalités supplémentaires, comme la couleur sémantique ou les sémantiques de texte. Voici quelques exemples :

**Couleur sémantique** : Certaines extensions de Markdown, comme "Emoji Syntax" ou "Emoji-Markdown" permettent d'ajouter des émojis pour indiquer la couleur sémantique d'un texte. Par exemple, vous pourrions utiliser un émoji de coeur rouge pour indiquer que le texte est important, ou un émoji de feu vert pour indiquer que le texte est un appel à l'action.

```markdown
I ❤️ this text is important
I 🔥 this text is an action
```

**Sémantique de texte** : Certaines extensions de Markdown, comme "Markdown Extra" ou "Pandoc Markdown" permettent d'ajouter des classes CSS à des éléments de texte pour indiquer une sémantique particulière. Par exemple, nous pourrions utiliser une classe "warning" pour indiquer que le texte est un avertissement, ou une classe "note" pour indiquer que le texte est une note.
```markdown
<div class="warning"> This is a warning text</div>
<div class="note"> This is a note text</div>
```

Il est important de noter que ces exemples dépendent des outils et des éditeurs que vous utilisons, il est donc important de vérifier que ces fonctionnalités sont supportées et de consulter la documentation pour savoir comment les utiliser correctement.

En résumé, il existe des extensions de Markdown qui permettent d'ajouter des fonctionnalités supplémentaires telles que la couleur sémantique ou les sémantiques de texte, grâce à l'utilisation d'émojis ou de classes CSS.

### Extensions de markdown faite par le CMS Grav
Grav est un cms php basé sur une modification de markdown pour ajouter des styles et du bootstrap ainsi il est possible de faire 


```markdown
[Default](#){.btn .btn-default}
[Primary](#){.btn .btn-primary}
[Info](#){.btn .btn-info}
[Success](#){.btn .btn-success}
[Warning](#){.btn .btn-warning}
[Danger](#){.btn .btn-danger}
[Link](#){.btn .btn-link}
```
### Alignements
La syntaxe markdown ne permet pas d’aligner du texte, c'est pourquoi Grav inclus quelques codes courts ("shortcodes") :

```
[left]Texte aligné à gauche[/left]
[center]Texte centré[/center]
[right]Texte aligné à droite[/right]
```

Il est aussi possible d’utiliser des éléments HTML et des classes CSS de Bootstrap :

```
<p class="text-center">Texte centré</p>
```

Les classes de Bootstrap sont : `text-left¦text-center¦text-right¦text-justify`

### Couleurs
Comme les couleurs ne sont pas disponible en markdown Grav a ajouté les codes courts shortcode :

```
Texte avec couleur par défaut, [color=#155dcb]texte de couleur bleue[/color], texte avec couleur par défaut.
```

Le code html  est également permis:

```
Texte avec couleur par défaut, <span style="color: #155dcb">texte de couleur bleue</span>, texte avec couleur par défaut.
```

Les classes Bootstrap de couleurs « contextuelles » sont aussi disponibles :

-   text-muted
-   text-primary
-   text-info
-   text-success
-   text-warning
-   text-danger

Même chose avec l’arrière plan

-   bg-primary
-   bg-info
-   bg-success
-   bg-warning
-   bg-danger

Exemple :

```
Texte normal suivi d’un <span class="bg-info text-success">texte coloré en vert sur fond bleu</span> dans un paragraphe.
```
# Écrire des Maths avec Markdown

Markdown, grâce à son intégration avec LaTeX, permet d'inclure des expressions mathématiques directement dans le texte, rendant la communication d'idées complexes plus accessible et plus précise.

## Modes de Formules Mathématiques

### Inline Mode

Pour intégrer une formule mathématique au sein d'un paragraphe, utilisez le mode inline. Encadrez votre formule avec un signe dollar `$`. Par exemple, pour écrire l'expression mathématique de la somme des \(X_i\) de 1 à \(n\), utilisez la notation `$\sum_{i=1}^n X_i$` qui s'affichera ainsi :$\sum_{i=1}^n X_i$. Notez que pour afficher un symbole dollar dans votre texte, échappez-le avec un backslash : `\$`.

### Displayed Mode

Pour mettre en valeur une formule, utilisez le mode displayed en l'encadrant avec deux signes dollars `$$`. Cette méthode centre la formule et la détache du paragraphe, la rendant plus visible. Par exemple, `$\sum_{i=1}^n X_i$` affichera la formule $\sum_{i=1}^n X_i$ de manière centrée et isolée du texte environnant.

## Symboles et Commandes Courants

### Lettres Grecques

- Alpha $\alpha$: `$\alpha$`
- Beta $\beta$: `$\beta$`
- Gamma $\gamma$ :`$\gamma$`, Gamma majuscule $\Gamma$: `$\gamma$`
- Delta $\delta$ :`$\delta$` et Delta majuscule $\Delta$:`$\Delta$`
- Epsilon $\epsilon$ et Epsilon majuscule
- Pi $\pi$: `$\pi$` et Pi majuscule $\Pi$:`$\Pi$`

### Fonctions et Opérateurs

- Cosinus $\cos$: `$\cos$`
- Sinus $\sin$: `$\sin$`
- Limite $\lim$: `$\lim$`
- Exponentielle $\exp$: `$\exp$`
- Appartient à $\in$: `$\in$`
- Pour tout $\forall$: `$\forall$`
- Il existe $\exists$: `$\exists$`
- Équivalent $\equiv$, Approximativement égal $\approx$: `$\equiv$`, `$\approx$`

### Exposants et Indices

- Indice $k_{n+1}$: `$k_{n+1}$`
- Exposant $n^2$: `$n^2$`
- Exposant et indice $k_n^2$: `$k_n^2$`

### Fractions, Coefficients Binomiaux, Racines

- Fraction $\frac{4z^3}{16}$: `$\frac{4z^3}{16}$`
- Coefficient binomial $\binom{n}{k}$: `$\binom{n}{k}$`
- Racine carrée $\sqrt{k}$, Racine n-ième $\sqrt[n]{k}$: `$\sqrt{k}$`, `$\sqrt[n]{k}$`

### Sommes et Intégrales

- Somme $\sum_{i=1}^{10} t_i$: `$\sum_{i=1}^{10} t_i$`
- Intégrale $\int_0^\infty \mathrm{e}^{-x}\,\mathrm{d}x$: `$\int_0^\infty \mathrm{e}^{-x}\,\mathrm{d}x$`

### Notations Spéciales

- Chapeau $\hat{a}$, Barre $\bar{a}$, Point $\dot{a}$, Double point $\ddot{a}$, Vecteur $\overrightarrow{AB}$: `$\hat{a}$`, `$\bar{a}$`, `$\dot{a}$`, `$\ddot{a}$`, `$\overrightarrow{AB}$`

## [](https://docs.framasoft.org/fr/grav/markdown.html#icônes)Icônes

La manière la plus simple d’ajouter des [icônes FontAwesome](https://fontawesome.com/v4.7.0/icons/) (version 4) est d’utiliser ces shortcodes :

[Avec un lien](https://www.mozilla.org/fr/firefox/new/)

```
    [fa=firefox /]
    [fa=firefox extras=fa-2x /]
    [[fa=firefox extras=fa-3x /] Avec un lien](https://www.mozilla.org/fr/firefox/new/)
    [fa=firefox extras=fa-4x,fa-spin /]
```

Dans les modules et autres composants du thème GravStrap, [la syntaxe est plus détaillée](https://docs.framasoft.org/fr/grav/composants-de-base.html#ic%C3%B4nes).

[Modules de Grav](https://docs.framasoft.org/fr/grav/modules.html)
[Notice de Framasoft](https://docs.framasoft.org/fr/grav/creation-de-page.html)

liens:
https://docs.framasoft.org/fr/grav/markdown.html

# Syntaxe étendue
[**Traduction du markdownguide**](https://www.markdownguide.org/extended-syntax/)
Fonctionnalités avancées basées sur la syntaxe Markdown de base.
## Aperçu
La syntaxe de base décrite dans le document de conception original de Markdown a ajouté de nombreux éléments nécessaires au quotidien, mais cela ne suffisait pas pour certaines personnes. C'est là qu'intervient la syntaxe étendue.

Plusieurs individus et organisations ont pris l'initiative d'étendre la syntaxe de base en ajoutant des éléments supplémentaires tels que des tableaux, des blocs de code, la coloration syntaxique, la création automatique de liens URL et des notes de bas de page. Ces éléments peuvent être activés en utilisant un langage de balisage léger basé sur la syntaxe Markdown de base ou en ajoutant une extension à un processeur Markdown compatible.
Disponibilité

Toutes les applications Markdown ne prennent pas en charge les éléments de syntaxe étendue. Vous devrez vérifier si le langage de balisage léger utilisé par votre application prend en charge les éléments de syntaxe étendue que vous souhaitez utiliser. Si ce n'est pas le cas, il peut être possible d'activer des extensions dans votre processeur Markdown.
Langages de balisage léger

Il existe plusieurs langages de balisage léger qui sont des sur-ensembles de Markdown. Ils incluent la syntaxe de base et la complètent en ajoutant des éléments supplémentaires tels que des tableaux, des blocs de code, la coloration syntaxique, la création automatique de liens URL et des notes de bas de page. Beaucoup des applications Markdown les plus populaires utilisent l'un des langages de balisage léger suivants :

    CommonMark
    GitHub Flavored Markdown (GFM)
    Markdown Extra
    MultiMarkdown
    R Markdown

Processeurs Markdown

Il existe des dizaines de processeurs Markdown disponibles. Beaucoup d'entre eux vous permettent d'ajouter des extensions qui activent des éléments de syntaxe étendue. Consultez la documentation de votre processeur pour plus d'informations.
Tableaux

Pour ajouter un tableau, utilisez trois traits d'union (---) ou plus pour créer l'en-tête de chaque colonne, et utilisez des barres verticales (|) pour séparer chaque colonne. Pour une meilleure compatibilité, vous devriez également ajouter une barre verticale de chaque côté de la ligne.

```
| Syntaxe      | Description |
| ----------- | ----------- |
| En-tête      | Titre       |
| Paragraphe   | Texte        |
```

Le rendu ressemble à ceci :
| Syntaxe      | Description |
| ----------- | ----------- |
| En-tête      | Titre       |
| Paragraphe   | Texte        |

Les largeurs des cellules peuvent varier, comme indiqué ci-dessous. Le rendu aura la même apparence.

| Syntaxe | Description |
| --- | ----------- |
| En-tête | Titre
