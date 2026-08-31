---
schema_version: 1
uid: "01M02EX5BSTDSSEEXSQKCPRPGY"
titre: "Matplotlib"
aliases:
  - "Mathplotlib"
  - "Visualisation de données avec Matplotlib"
  - "Graphiques Python avec Matplotlib"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - visualisation
  - matplotlib
  - data-science
resume: "Cours complet sur Matplotlib : modèle Figure/Axes/Artist, API orientée objet, graphiques usuels, mises en page, styles, couleurs, annotations, export vectoriel et raster, notebooks, performances, accessibilité et intégration NumPy/Pandas."
niveau: intermediaire
prerequis:
  - "[[Python]]"
  - "[[Numpy]]"
  - "[[Pandas]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2025-05-16
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Matplotlib

> [!abstract] Objectifs
> À la fin de ce cours, nous devons être capables de construire des graphiques Matplotlib lisibles et reproductibles, de choisir entre l'interface `pyplot` et l'interface explicite `Figure`/`Axes`, d'organiser plusieurs sous-graphiques, de maîtriser les couleurs, légendes et annotations, puis d'exporter correctement le résultat en PNG, SVG ou PDF.

Voir aussi : [[Python]], [[Numpy]], [[Pandas]], [[Machine Learning]], [[Google Colab]], [[Anaconda]].

> [!important] Version de référence
> Ce cours est aligné sur **Matplotlib 3.11.x**. Matplotlib reste toutefois une bibliothèque relativement stable : la plupart des concepts présentés ici sont valables sur plusieurs générations 3.x.

# 1. Qu'est-ce que Matplotlib ?

Matplotlib est la bibliothèque historique de visualisation scientifique de l'écosystème Python.

Elle permet notamment de produire :

- courbes ;
- nuages de points ;
- histogrammes ;
- diagrammes en barres ;
- cartes de chaleur ;
- graphiques statistiques ;
- visualisations d'images ;
- figures multi-panneaux ;
- graphiques vectoriels pour publication ;
- animations ;
- visualisations interactives selon le backend utilisé.

Matplotlib est souvent utilisé directement, mais il sert également de couche de rendu à d'autres bibliothèques.

Par exemple :

```text
NumPy / Pandas / SciPy / scikit-learn
               │
               ▼
           Matplotlib
               │
      ┌────────┼────────┐
      ▼        ▼        ▼
     PNG      SVG      PDF
```

# 2. Le modèle mental essentiel : Figure, Axes et Artist

La confusion la plus fréquente chez les débutants vient du mot `Axes`.

Dans Matplotlib :

- une **Figure** représente la figure complète ;
- un **Axes** représente une zone de tracé ;
- les courbes, textes, légendes, ticks, rectangles, images, etc. sont des **Artists**.

Schéma mental :

```text
Figure
├── Axes 1
│   ├── Line2D
│   ├── Text
│   ├── Legend
│   ├── XAxis
│   └── YAxis
└── Axes 2
    ├── Image
    └── Colorbar
```

> [!tip]
> Dans le vocabulaire Matplotlib, `Axes` est au singulier lorsqu'il désigne un objet de type `matplotlib.axes.Axes`.

# 3. Installation

Avec `pip` :

```bash
python -m pip install matplotlib
```

Avec un environnement virtuel :

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install matplotlib numpy pandas
```

Vérifier la version :

```python
import matplotlib

print(matplotlib.__version__)
```

# 4. Deux interfaces : implicite et explicite

Matplotlib propose historiquement deux styles d'utilisation.

## 4.1 Interface implicite `pyplot`

```python
import matplotlib.pyplot as plt

plt.plot([1, 2, 3], [2, 4, 9])
plt.xlabel("x")
plt.ylabel("y")
plt.title("Exemple")
plt.show()
```

Cette écriture est courte et pratique pour :

- l'exploration rapide ;
- les notebooks ;
- les exemples simples ;
- les graphiques jetables.

Mais `pyplot` agit en partie sur une notion de **figure et d'axes courants**.

Sur des figures plus complexes, cela peut rendre le code moins explicite.

## 4.2 Interface explicite orientée objet

Pour du code réutilisable, préférons généralement :

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
ax.plot([1, 2, 3], [2, 4, 9])
ax.set_xlabel("x")
ax.set_ylabel("y")
ax.set_title("Exemple")
plt.show()
```

Ici :

```text
fig = la Figure
ax  = l'Axes
```

Le code est plus facile à :

- encapsuler dans une fonction ;
- tester ;
- modifier ;
- composer ;
- relire ;
- maintenir.

> [!important] Convention recommandée
> Pour les cours, bibliothèques internes, scripts de production et fonctions réutilisables, utilisons de préférence l'interface explicite `fig, ax = plt.subplots()`.

# 5. Premier graphique propre

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(0, 2 * np.pi, 200)
y = np.sin(x)

fig, ax = plt.subplots(layout="constrained")
ax.plot(x, y, label="sin(x)")
ax.set(
    title="Fonction sinus",
    xlabel="x (radians)",
    ylabel="amplitude",
)
ax.legend()
ax.grid(True, alpha=0.3)

plt.show()
```

Nous retrouvons déjà plusieurs bonnes pratiques :

1. données séparées du rendu ;
2. interface explicite ;
3. titres et unités ;
4. légende seulement si elle apporte quelque chose ;
5. grille discrète ;
6. gestion automatique de la mise en page.

# 6. `plt.show()`

Dans un script classique :

```python
plt.show()
```

ouvre ou affiche la figure selon le backend actif.

Dans de nombreux notebooks, l'affichage est automatique en fin de cellule.

Il reste toutefois utile de connaître la distinction entre :

```python
fig, ax = plt.subplots()
```

qui **construit** une figure, et :

```python
plt.show()
```

qui demande son **affichage**.

# 7. Tracer une courbe avec `plot()`

```python
fig, ax = plt.subplots()
ax.plot(x, y)
```

Exemple avec plusieurs séries :

```python
x = np.linspace(0, 10, 300)

fig, ax = plt.subplots(layout="constrained")
ax.plot(x, np.sin(x), label="sin")
ax.plot(x, np.cos(x), label="cos")
ax.legend()
```

## 7.1 Styles de ligne

```python
ax.plot(x, y, linestyle="--")
```

Quelques valeurs usuelles :

```text
-   ligne continue
--  tirets
-.  tiret-point
:   pointillés
```

## 7.2 Marqueurs

```python
ax.plot(x, y, marker="o")
```

Pour peu de points, les marqueurs peuvent aider à montrer que la donnée est discrète.

Pour des milliers de points, ils peuvent au contraire surcharger la figure.

# 8. Nuage de points : `scatter()`

```python
rng = np.random.default_rng(42)
x = rng.normal(size=300)
y = 0.6 * x + rng.normal(scale=0.7, size=300)

fig, ax = plt.subplots(layout="constrained")
ax.scatter(x, y, alpha=0.6)
ax.set(xlabel="x", ylabel="y", title="Nuage de points")
```

La transparence aide à percevoir les zones denses :

```python
alpha=0.3
```

Pour colorer les points selon une troisième variable :

```python
score = x**2 + y**2

fig, ax = plt.subplots(layout="constrained")
sc = ax.scatter(x, y, c=score, cmap="viridis")
fig.colorbar(sc, ax=ax, label="score")
```

# 9. Diagrammes en barres

```python
categories = ["A", "B", "C"]
values = [12, 18, 9]

fig, ax = plt.subplots(layout="constrained")
ax.bar(categories, values)
ax.set_ylabel("Valeur")
```

Barres horizontales :

```python
ax.barh(categories, values)
```

> [!warning]
> Un diagramme en barres représente généralement des catégories. Pour une distribution numérique continue, préférons un histogramme.

# 10. Histogrammes

```python
rng = np.random.default_rng(42)
data = rng.normal(loc=10, scale=2, size=5000)

fig, ax = plt.subplots(layout="constrained")
ax.hist(data, bins=40)
ax.set(xlabel="Valeur", ylabel="Effectif")
```

Le nombre de classes influence fortement l'apparence du résultat.

Ne choisissons donc pas `bins` uniquement pour obtenir une figure « jolie ».

On peut aussi demander une densité :

```python
ax.hist(data, bins=40, density=True)
```

# 11. Boîtes à moustaches et violons

Boxplot :

```python
fig, ax = plt.subplots(layout="constrained")
ax.boxplot([group_a, group_b, group_c], tick_labels=["A", "B", "C"])
```

Violin plot :

```python
fig, ax = plt.subplots(layout="constrained")
ax.violinplot([group_a, group_b, group_c], showmedians=True)
```

Ces graphiques résument une distribution, mais ils ne remplacent pas toujours l'observation des données brutes.

# 12. Barres d'erreur et incertitude

```python
x = np.arange(5)
y = np.array([10, 12, 9, 15, 14])
err = np.array([1.2, 0.8, 1.4, 1.0, 0.7])

fig, ax = plt.subplots(layout="constrained")
ax.errorbar(x, y, yerr=err, marker="o", capsize=4)
```

Toujours préciser dans la légende ou le texte ce que représente l'incertitude :

- écart-type ;
- erreur standard ;
- intervalle de confiance ;
- étendue ;
- incertitude instrumentale.

# 13. Plusieurs sous-graphiques

```python
fig, axes = plt.subplots(
    nrows=2,
    ncols=2,
    figsize=(10, 7),
    layout="constrained",
)
```

`axes` est alors un tableau NumPy d'objets `Axes`.

```python
axes[0, 0].plot(x, y1)
axes[0, 1].scatter(x2, y2)
axes[1, 0].hist(values)
axes[1, 1].bar(labels, counts)
```

## 13.1 Partager les axes

```python
fig, axes = plt.subplots(2, 1, sharex=True, layout="constrained")
```

Cela est particulièrement utile pour des séries temporelles alignées.

# 14. `subplot_mosaic()`

Pour une mise en page sémantique :

```python
fig, axd = plt.subplot_mosaic(
    [
        ["main", "main"],
        ["left", "right"],
    ],
    layout="constrained",
)

axd["main"].plot(x, y)
axd["left"].hist(a)
axd["right"].hist(b)
```

Cette approche évite les indices du type :

```python
axes[1, 0]
```

lorsque la composition devient complexe.

# 15. `layout="constrained"`

Pour les nouvelles figures, une approche robuste consiste à créer la figure avec :

```python
fig, ax = plt.subplots(layout="constrained")
```

ou :

```python
fig = plt.figure(layout="constrained")
```

Le moteur essaie d'éviter les collisions entre :

- titres ;
- labels ;
- légendes ;
- colorbars ;
- sous-graphiques.

L'ancienne fonction :

```python
plt.tight_layout()
```

reste utile dans du code existant, mais `constrained` est souvent un meilleur point de départ pour les nouvelles figures complexes.

# 16. Taille de la figure

```python
fig, ax = plt.subplots(figsize=(8, 5))
```

La taille est historiquement exprimée en pouces.

La résolution raster dépend ensuite des DPI.

Exemple :

```text
8 pouces × 5 pouces × 150 dpi
→ 1200 × 750 pixels
```

# 17. Titres, labels et unités

```python
ax.set_title("Température au cours du temps")
ax.set_xlabel("Temps (h)")
ax.set_ylabel("Température (°C)")
```

Ou de manière groupée :

```python
ax.set(
    title="Température au cours du temps",
    xlabel="Temps (h)",
    ylabel="Température (°C)",
)
```

> [!tip]
> Une unité pertinente dans le label est souvent plus utile qu'une précision supplémentaire dans le titre.

# 18. Légendes

```python
ax.plot(x, y1, label="Mesure")
ax.plot(x, y2, label="Modèle")
ax.legend()
```

Une légende est utile lorsqu'il faut identifier plusieurs séries.

Elle est inutile si la figure ne contient qu'une seule série évidente et déjà décrite par les axes.

Position explicite :

```python
ax.legend(loc="upper right")
```

# 19. Annotations

```python
ax.annotate(
    "maximum",
    xy=(x_max, y_max),
    xytext=(x_max + 1, y_max + 2),
    arrowprops={"arrowstyle": "->"},
)
```

Une annotation doit aider le lecteur à voir un fait important, pas simplement répéter les données.

# 20. Lignes et zones de référence

Ligne horizontale :

```python
ax.axhline(0, linestyle="--", alpha=0.5)
```

Ligne verticale :

```python
ax.axvline(threshold, linestyle="--", alpha=0.5)
```

Zone :

```python
ax.axvspan(start, end, alpha=0.15)
```

Intervalle autour d'une courbe :

```python
ax.fill_between(x, lower, upper, alpha=0.2)
```

# 21. Échelles logarithmiques

```python
ax.set_yscale("log")
```

ou :

```python
ax.semilogy(x, y)
```

Attention : une échelle logarithmique change la perception des distances.

Il faut l'indiquer clairement.

# 22. Limites des axes

```python
ax.set_xlim(0, 100)
ax.set_ylim(-1, 1)
```

Évitons de tronquer volontairement un axe pour exagérer une variation sans l'expliquer.

Sur un diagramme en barres, une origine non nulle peut être particulièrement trompeuse.

# 23. Ticks et formatage

```python
from matplotlib.ticker import PercentFormatter

ax.yaxis.set_major_formatter(PercentFormatter(xmax=1))
```

Pour un format personnalisé :

```python
from matplotlib.ticker import FuncFormatter

ax.yaxis.set_major_formatter(
    FuncFormatter(lambda value, pos: f"{value / 1_000_000:.1f} M")
)
```

# 24. Dates

Matplotlib sait travailler avec des objets `datetime`.

```python
import pandas as pd
import matplotlib.dates as mdates

index = pd.date_range("2026-01-01", periods=100, freq="D")
values = np.cumsum(rng.normal(size=100))

fig, ax = plt.subplots(layout="constrained")
ax.plot(index, values)

locator = mdates.AutoDateLocator()
ax.xaxis.set_major_locator(locator)
ax.xaxis.set_major_formatter(mdates.ConciseDateFormatter(locator))
```

# 25. Couleurs

Une couleur peut être spécifiée de plusieurs façons :

```python
color="black"
color="#336699"
color=(0.2, 0.4, 0.6)
```

Pour une série unique, n'ajoutons pas des couleurs différentes point par point sans signification.

# 26. Colormaps

Pour une variable quantitative :

```python
cmap="viridis"
```

Autres familles :

```text
séquentielles   : viridis, plasma, inferno, magma, cividis
Divergentes     : coolwarm, RdBu, PiYG
Cycliques       : twilight
```

Le choix doit dépendre de la structure de la donnée.

## 26.1 Variable séquentielle

Exemple : température de 0 à 100.

Utilisons une palette séquentielle.

## 26.2 Variable divergente

Exemple : variation autour de zéro.

Utilisons une palette divergente avec un centre explicite.

```python
from matplotlib.colors import TwoSlopeNorm

norm = TwoSlopeNorm(vmin=-10, vcenter=0, vmax=20)
```

# 27. Colorbar

```python
fig, ax = plt.subplots(layout="constrained")
image = ax.imshow(matrix, cmap="viridis")
fig.colorbar(image, ax=ax, label="Intensité")
```

La colorbar fait partie de la sémantique du graphique.

Ne l'oublions pas lorsqu'une couleur encode une valeur.

# 28. Accessibilité des couleurs

Ne codons pas une information importante **uniquement** par la couleur.

Combinons si nécessaire :

- couleurs ;
- marqueurs ;
- styles de lignes ;
- annotations ;
- labels directs.

Exemple :

```python
ax.plot(x, y_a, linestyle="-", marker="o", label="A")
ax.plot(x, y_b, linestyle="--", marker="s", label="B")
```

Cela reste interprétable :

- en niveaux de gris ;
- pour certains déficits de perception des couleurs ;
- sur une impression médiocre.

# 29. Styles globaux

Lister les styles :

```python
import matplotlib.pyplot as plt

print(plt.style.available)
```

Utiliser un style :

```python
plt.style.use("ggplot")
```

Pour éviter de modifier tout le processus :

```python
with plt.style.context("ggplot"):
    fig, ax = plt.subplots()
    ax.plot(x, y)
```

# 30. `rcParams`

Matplotlib possède un grand système de configuration.

```python
import matplotlib as mpl

mpl.rcParams["figure.figsize"] = (8, 5)
mpl.rcParams["axes.grid"] = True
```

Pour un projet, il est préférable de centraliser les choix visuels plutôt que de répéter les mêmes arguments dans chaque graphique.

# 31. Fichier `matplotlibrc`

Les paramètres peuvent être définis dans un fichier `matplotlibrc`.

Exemples :

```text
figure.figsize: 8, 5
axes.grid: True
savefig.format: svg
```

Inspecter l'emplacement :

```python
import matplotlib

print(matplotlib.matplotlib_fname())
print(matplotlib.get_configdir())
```

# 32. Visualiser une matrice avec `imshow()`

```python
matrix = rng.normal(size=(20, 30))

fig, ax = plt.subplots(layout="constrained")
im = ax.imshow(matrix, cmap="viridis", aspect="auto")
fig.colorbar(im, ax=ax)
```

`imshow()` représente une matrice comme une image.

Ce n'est pas toujours équivalent à une carte de chaleur sémantiquement annotée.

# 33. Coordonnées d'une image

Par défaut, l'origine d'une image peut surprendre lorsqu'on raisonne en coordonnées cartésiennes.

On peut préciser :

```python
ax.imshow(matrix, origin="lower")
```

Ou donner l'étendue réelle :

```python
ax.imshow(
    matrix,
    extent=(xmin, xmax, ymin, ymax),
    origin="lower",
    aspect="auto",
)
```

# 34. `pcolormesh()`

Pour un maillage associé à des coordonnées physiques :

```python
fig, ax = plt.subplots(layout="constrained")
mesh = ax.pcolormesh(x_edges, y_edges, z, shading="auto", cmap="viridis")
fig.colorbar(mesh, ax=ax)
```

# 35. Contours

```python
fig, ax = plt.subplots(layout="constrained")
cs = ax.contour(X, Y, Z)
ax.clabel(cs)
```

Contours remplis :

```python
ax.contourf(X, Y, Z, levels=20, cmap="viridis")
```

# 36. Graphiques polaires

```python
fig, ax = plt.subplots(subplot_kw={"projection": "polar"})
ax.plot(theta, radius)
```

Utilisons une projection polaire lorsqu'elle correspond réellement au phénomène : angle, phase, direction, saisonnalité cyclique, etc.

# 37. Graphiques 3D

```python
fig = plt.figure(layout="constrained")
ax = fig.add_subplot(projection="3d")
ax.plot_surface(X, Y, Z)
```

> [!warning]
> Un graphique 3D est souvent plus difficile à lire qu'une bonne visualisation 2D. Ne l'utilisons pas uniquement pour produire un effet spectaculaire.

# 38. Deux axes Y : à utiliser avec prudence

```python
fig, ax1 = plt.subplots(layout="constrained")
ax2 = ax1.twinx()
```

Deux échelles sur la même figure peuvent facilement produire des relations visuelles trompeuses.

Préférons souvent :

- deux sous-graphiques alignés ;
- une normalisation ;
- un indice commun ;
- un graphique explicitement annoté.

# 39. Axe secondaire avec transformation

Lorsqu'il existe une vraie relation mathématique entre deux unités :

```python
def c_to_f(c):
    return c * 9 / 5 + 32


def f_to_c(f):
    return (f - 32) * 5 / 9

secax = ax.secondary_yaxis("right", functions=(c_to_f, f_to_c))
secax.set_ylabel("Température (°F)")
```

C'est plus rigoureux qu'un `twinx()` arbitraire.

# 40. Texte mathématique

Matplotlib possède son moteur MathText.

```python
ax.set_title(r"$f(x)=\sin(2\pi x)$")
```

Un environnement LaTeX externe n'est pas nécessaire pour ces expressions courantes.

# 41. Polices

```python
import matplotlib as mpl

mpl.rcParams["font.family"] = "sans-serif"
```

Pour des graphiques destinés à être partagés, vérifions :

- disponibilité de la police ;
- rendu SVG/PDF ;
- caractères accentués ;
- symboles mathématiques ;
- licences de la police.

# 42. Exporter une figure

Avec l'interface explicite :

```python
fig.savefig("figure.png")
```

Le format est généralement déduit de l'extension.

```python
fig.savefig("figure.svg")
fig.savefig("figure.pdf")
```

# 43. PNG, SVG ou PDF ?

## 43.1 PNG

Raster.

Approprié pour :

- captures web ;
- messageries ;
- figures contenant beaucoup de pixels ;
- compatibilité maximale.

## 43.2 SVG

Vectoriel pour les primitives graphiques.

Approprié pour :

- web ;
- documentation ;
- schémas et courbes ;
- zoom sans pixellisation des éléments vectoriels.

## 43.3 PDF

Vectoriel lorsque les objets le permettent.

Approprié pour :

- publications ;
- rapports ;
- impression ;
- intégration dans un document PDF/LaTeX.

> [!important]
> Une sortie SVG ou PDF n'implique pas que **tout** soit vectoriel. Une image ou un artiste explicitement rasterisé peut rester raster dans un conteneur vectoriel.

# 44. L'astuce historique du cours : SVG dans les notebooks

L'ancienne fiche contenait uniquement cette idée, qui reste utile.

Dans un environnement utilisant `matplotlib-inline` :

```python
from matplotlib_inline.backend_inline import set_matplotlib_formats

set_matplotlib_formats("svg")
```

Cela demande un affichage SVG des figures inline.

Autre possibilité dans IPython/Jupyter :

```python
%config InlineBackend.figure_format = 'svg'
```

> [!note]
> Cette configuration concerne le **rendu inline du notebook**. Pour créer explicitement un fichier SVG, utilisons plutôt `fig.savefig("figure.svg")`.

# 45. Export SVG explicite

```python
fig, ax = plt.subplots(layout="constrained")
ax.plot(x, y)

fig.savefig(
    "courbe.svg",
    metadata={
        "Title": "Courbe expérimentale",
        "Creator": "Matplotlib",
    },
)
```

Le backend SVG permet aussi de stocker certaines métadonnées dans le fichier.

# 46. DPI : comprendre ce qui change réellement

Pour un PNG :

```python
fig.savefig("figure.png", dpi=200)
```

Le DPI influence directement la quantité de pixels.

Pour un SVG composé de primitives vectorielles, augmenter les DPI n'améliore pas les lignes vectorielles elles-mêmes.

Le DPI reste toutefois pertinent pour les parties rasterisées ou les images incluses.

# 47. `bbox_inches="tight"`

```python
fig.savefig("figure.svg", bbox_inches="tight")
```

Cela peut réduire les marges autour de la figure.

Avec les moteurs modernes de layout, testons toutefois le résultat plutôt que de cumuler automatiquement plusieurs mécanismes de mise en page.

# 48. Fond transparent

```python
fig.savefig("figure.svg", transparent=True)
```

Utile pour intégrer la figure sur un fond variable.

Vérifions alors que :

- les textes restent lisibles ;
- les lignes ne dépendent pas d'un fond blanc ;
- la grille ne devient pas trop visible.

# 49. Texte dans les SVG

Matplotlib permet de contrôler la manière dont les polices sont représentées dans les SVG via `svg.fonttype`.

Par défaut, Matplotlib peut convertir les glyphes en chemins afin d'améliorer la portabilité du rendu.

Pour conserver du texte comme texte SVG :

```python
import matplotlib as mpl

mpl.rcParams["svg.fonttype"] = "none"
```

Cela facilite parfois :

- l'édition du SVG ;
- la recherche de texte ;
- l'accessibilité.

Mais cela suppose que la police nécessaire soit disponible lors du rendu.

# 50. Rasterisation sélective

Sur un nuage de plusieurs centaines de milliers de points, un SVG peut devenir énorme.

Nous pouvons conserver axes et textes en vectoriel tout en rasterisant une collection lourde :

```python
fig, ax = plt.subplots(layout="constrained")
ax.scatter(x, y, s=2, rasterized=True)
fig.savefig("scatter.pdf", dpi=200)
```

Résultat conceptuel :

```text
PDF/SVG
├── textes       vectoriels
├── axes         vectoriels
├── légende      vectorielle
└── scatter      rasterisé
```

C'est souvent un excellent compromis pour la publication scientifique.

# 51. Intégration avec NumPy

Matplotlib accepte naturellement les tableaux NumPy.

```python
x = np.linspace(0, 1, 1000)
y = np.exp(-5 * x) * np.cos(30 * x)

fig, ax = plt.subplots(layout="constrained")
ax.plot(x, y)
```

Séparons idéalement :

```text
calcul NumPy
     │
     ▼
visualisation Matplotlib
```

# 52. Intégration avec Pandas

Pandas peut tracer via Matplotlib :

```python
ax = df.plot(x="date", y="temperature")
```

Pour davantage de contrôle :

```python
fig, ax = plt.subplots(layout="constrained")
ax.plot(df["date"], df["temperature"])
```

Ou réutiliser un `Axes` :

```python
fig, ax = plt.subplots(layout="constrained")
df.plot(x="date", y="temperature", ax=ax)
ax.set_title("Température")
```

# 53. Valeurs manquantes

Selon le type de graphique, les `NaN` peuvent interrompre ou modifier le rendu.

Exemple :

```python
values = np.array([1, 2, np.nan, 4, 5])
```

Avant de tracer, demandons-nous ce que signifie réellement une valeur manquante :

- absence de mesure ;
- zéro ;
- donnée censurée ;
- erreur d'acquisition ;
- période hors domaine.

Ne remplaçons pas automatiquement un `NaN` par zéro uniquement pour obtenir une courbe continue.

# 54. Catégories

```python
labels = ["Linux", "Windows", "macOS"]
values = [55, 30, 15]

fig, ax = plt.subplots(layout="constrained")
ax.bar(labels, values)
```

Si les labels sont longs :

```python
ax.tick_params(axis="x", labelrotation=30)
```

Mais un `barh()` est souvent plus lisible.

# 55. Graphiques empilés

```python
ax.bar(x, a, label="A")
ax.bar(x, b, bottom=a, label="B")
```

Une pile permet d'observer une composition, mais rend les comparaisons internes plus difficiles pour les séries qui n'ont pas une base commune.

# 56. Graphique en secteurs : usage limité

Matplotlib sait produire :

```python
ax.pie(values, labels=labels)
```

Mais un diagramme en barres est souvent plus précis pour comparer des quantités.

Réservons les secteurs à de petites compositions simples lorsque la somme à 100 % est réellement le message principal.

# 57. Courbes cumulées

```python
ax.stackplot(x, y1, y2, y3, labels=["A", "B", "C"])
```

Utile pour montrer une évolution de composition.

Comme pour les barres empilées, la comparaison des couches intermédiaires reste difficile.

# 58. `step()` et données discrètes dans le temps

```python
ax.step(t, state, where="post")
```

Approprié pour :

- états ;
- consignes ;
- quantités restant constantes entre événements ;
- signaux numériques.

# 59. `stem()`

```python
ax.stem(x, y)
```

Utile pour des valeurs discrètes, par exemple en traitement du signal.

# 60. Grille

```python
ax.grid(True, alpha=0.25)
```

Une grille doit guider la lecture sans dominer les données.

Une grille noire, épaisse et complète peut détourner l'attention du signal principal.

# 61. Spines

Les bordures d'un `Axes` sont des `spines`.

```python
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)
```

Ne supprimons cependant pas des éléments uniquement pour suivre une mode graphique : la lisibilité est le critère principal.

# 62. Labels directs

Au lieu d'une grande légende, il peut être plus lisible d'écrire directement le nom près de chaque courbe.

```python
ax.text(x[-1], y1[-1], "Série A")
```

Cette approche fonctionne particulièrement bien avec peu de séries.

# 63. Ordre visuel et `zorder`

```python
ax.scatter(x, y, zorder=3)
ax.grid(zorder=0)
```

Plus le `zorder` est élevé, plus l'artiste est dessiné au-dessus.

# 64. Transparence et `alpha`

```python
ax.scatter(x, y, alpha=0.25)
```

La transparence est utile pour les superpositions mais peut aussi créer des couleurs visuelles difficiles à interpréter.

# 65. Clipping

Les artistes sont généralement limités à leur zone de dessin.

Certains éléments peuvent être placés en dehors de l'Axes.

Comprendre le clipping devient utile lorsque :

- une annotation disparaît ;
- une légende sort de la figure ;
- une zone est volontairement dessinée hors du cadre.

# 66. Systèmes de coordonnées

Matplotlib possède plusieurs systèmes de coordonnées.

Une annotation peut être positionnée :

- dans les coordonnées des données ;
- relativement à l'Axes ;
- relativement à la Figure ;
- en pixels/points via certaines transformations.

Exemple avec coordonnées relatives à l'Axes :

```python
ax.text(
    0.02,
    0.98,
    "(a)",
    transform=ax.transAxes,
    va="top",
)
```

Ici :

```text
(0, 0) = bas gauche de l'Axes
(1, 1) = haut droite de l'Axes
```

# 67. Fonction de tracé réutilisable

Une bonne fonction de visualisation peut recevoir un `Axes`.

```python
def plot_signal(ax, x, y, *, label=None):
    ax.plot(x, y, label=label)
    ax.set_xlabel("Temps (s)")
    ax.set_ylabel("Amplitude")
    return ax
```

Puis :

```python
fig, ax = plt.subplots(layout="constrained")
plot_signal(ax, x, y, label="mesure")
ax.legend()
```

Cette approche est beaucoup plus composable qu'une fonction qui crée toujours sa propre figure et appelle immédiatement `plt.show()`.

# 68. Ne pas appeler `show()` dans une fonction de bibliothèque

À éviter dans une fonction réutilisable :

```python
def make_plot(data):
    plt.plot(data)
    plt.show()
```

Mieux :

```python
def plot_data(ax, data):
    ax.plot(data)
    return ax
```

Le code appelant décide ensuite :

```python
plt.show()
```

ou :

```python
fig.savefig("result.svg")
```

# 69. Fermer les figures dans des traitements batch

Lorsqu'un script crée des milliers de figures :

```python
for item in items:
    fig, ax = plt.subplots()
    ax.plot(item.x, item.y)
    fig.savefig(item.path)
    plt.close(fig)
```

Sinon, les figures restent référencées et la consommation mémoire peut augmenter.

# 70. Backends

Matplotlib sépare le modèle de figure du moteur de rendu et de l'interface interactive.

On distingue grossièrement :

- backends interactifs ;
- backends non interactifs ;
- backends raster ;
- backends vectoriels.

Exemples de formats :

```text
Agg  → PNG raster
SVG  → SVG vectoriel
PDF  → PDF
PGF  → intégration LaTeX
```

# 71. Script sans écran

Sur un serveur ou dans une CI, nous pouvons simplement sauvegarder des figures sans interface graphique.

Exemple :

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
ax.plot([1, 2, 3])
fig.savefig("report.svg")
```

Dans la plupart des environnements modernes, Matplotlib choisit un backend approprié au contexte.

Évitons de forcer un backend global sans raison.

# 72. Jupyter

Dans Jupyter, le backend inline est fréquent.

L'ancien magic :

```python
%matplotlib inline
```

reste connu, mais les environnements modernes configurent souvent automatiquement l'affichage.

Le point important est de distinguer :

```text
backend d'affichage du notebook
          ≠
format d'export du fichier
```

# 73. SVG inline dans Jupyter

Version Python :

```python
from matplotlib_inline.backend_inline import set_matplotlib_formats

set_matplotlib_formats("svg")
```

Pour demander plusieurs représentations :

```python
set_matplotlib_formats("png", "svg")
```

Selon le frontend, une représentation appropriée pourra être utilisée.

# 74. Interaction dans les notebooks

Selon l'environnement, des backends interactifs supplémentaires peuvent permettre :

- zoom ;
- déplacement ;
- mise à jour dynamique ;
- widgets.

La disponibilité dépend du frontend et des paquets installés.

Ne supposons pas qu'un notebook interactif fonctionnera identiquement dans :

- JupyterLab ;
- VS Code ;
- Google Colab ;
- navigateur embarqué ;
- export HTML statique.

# 75. Animations

Principe :

```python
from matplotlib.animation import FuncAnimation
```

Exemple conceptuel :

```python
fig, ax = plt.subplots()
line, = ax.plot([], [])


def update(frame):
    line.set_data(x[:frame], y[:frame])
    return (line,)

anim = FuncAnimation(fig, update, frames=len(x), interval=30)
```

L'export vidéo ou GIF peut dépendre d'outils externes tels que FFmpeg ou Pillow.

# 76. Performance : éviter de redessiner inutilement

Pour une animation ou une interface interactive, modifier un artiste existant peut être beaucoup plus efficace que recréer entièrement le graphique.

```python
line.set_ydata(new_y)
```

plutôt que :

```python
ax.clear()
ax.plot(x, new_y)
```

à chaque mise à jour.

# 77. Très grands jeux de données

Tracer dix millions de points n'est pas toujours utile.

Avant de rendre une énorme série, considérons :

- agrégation ;
- décimation ;
- échantillonnage ;
- histogramme 2D ;
- hexbin ;
- rasterisation ;
- visualisation multi-résolution.

# 78. `hexbin()`

Pour un nuage extrêmement dense :

```python
fig, ax = plt.subplots(layout="constrained")
hb = ax.hexbin(x, y, gridsize=60, mincnt=1)
fig.colorbar(hb, ax=ax, label="Nombre de points")
```

Cela peut révéler la densité mieux qu'un `scatter` surchargé.

# 79. Sous-échantillonner pour l'exploration

```python
idx = rng.choice(len(x), size=20_000, replace=False)
ax.scatter(x[idx], y[idx], s=2, alpha=0.3)
```

Pour une analyse finale, indiquons clairement si le graphique montre un échantillon et selon quelle méthode il a été obtenu.

# 80. Reproductibilité

Pour les exemples aléatoires :

```python
rng = np.random.default_rng(42)
```

Pour une publication ou un pipeline :

- fixer les versions ;
- fixer les graines pertinentes ;
- versionner le code ;
- versionner les données ou leur empreinte ;
- enregistrer les paramètres ;
- préciser les unités ;
- exporter automatiquement les figures.

# 81. Ne pas confondre reproductibilité graphique et reproductibilité scientifique

Une figure identique ne garantit pas que :

- les données sont correctes ;
- la méthode statistique est correcte ;
- le modèle est valide ;
- la conclusion est justifiée.

Matplotlib rend des objets graphiques ; il ne valide pas l'analyse scientifique.

# 82. Visualisation et Machine Learning

Avec [[Machine Learning]], Matplotlib sert souvent à examiner :

- distributions ;
- courbes ROC/PR ;
- matrices de confusion ;
- erreurs ;
- résidus ;
- importance de variables ;
- courbes d'apprentissage ;
- calibration ;
- drift.

Mais un graphique ne doit pas être construit à partir du jeu de test final pour guider constamment les choix de modèle.

Sinon, nous réintroduisons indirectement une fuite d'information dans le processus expérimental.

# 83. Matrice de confusion

Exemple manuel :

```python
fig, ax = plt.subplots(layout="constrained")
im = ax.imshow(confusion_matrix, cmap="Blues")
fig.colorbar(im, ax=ax)
```

Avec scikit-learn, des objets d'affichage dédiés existent également.

Préférons-les lorsqu'ils codifient correctement la sémantique du graphique.

# 84. Courbe ROC : ne pas oublier le contexte

Une courbe ROC peut être utile mais devient parfois trop optimiste visuellement lorsque les classes sont extrêmement déséquilibrées.

Une courbe précision-rappel peut alors être plus informative.

Le rôle de Matplotlib est ici de rendre la courbe ; le choix de la métrique dépend du problème.

# 85. Tester une fonction de visualisation

On peut tester :

- le nombre de lignes ;
- les labels ;
- le titre ;
- les limites ;
- la présence d'une légende ;
- certains Artists.

Exemple :

```python
def test_plot_signal():
    fig, ax = plt.subplots()
    plot_signal(ax, [0, 1], [2, 3])

    assert ax.get_xlabel() == "Temps (s)"
    assert ax.get_ylabel() == "Amplitude"
    assert len(ax.lines) == 1

    plt.close(fig)
```

# 86. Tests d'images

Pour une bibliothèque graphique, un test d'image peut comparer un rendu produit à une image de référence.

Ces tests sont puissants mais sensibles à :

- versions de Matplotlib ;
- polices ;
- backend ;
- système d'exploitation ;
- anti-aliasing ;
- paramètres de rendu.

Ils doivent être utilisés avec une infrastructure maîtrisée.

# 87. Sauvegarder automatiquement les figures d'un rapport

Structure possible :

```text
project/
├── data/
├── src/
├── notebooks/
├── reports/
│   └── figures/
└── pyproject.toml
```

Puis :

```python
from pathlib import Path

FIGURES = Path("reports/figures")
FIGURES.mkdir(parents=True, exist_ok=True)

fig.savefig(FIGURES / "distribution.svg")
```

# 88. Nommer les fichiers de manière stable

Préférons :

```text
confusion-matrix-test.svg
latency-p95-by-region.svg
revenue-monthly-2026.svg
```

à :

```text
figure1.svg
final2.svg
graph_bon_v3_definitif.svg
```

# 89. Métadonnées d'export

Pour un SVG :

```python
fig.savefig(
    "figure.svg",
    metadata={
        "Title": "Latence par région",
        "Description": "Mesure P95 sur la période de test",
        "Creator": "Python / Matplotlib",
    },
)
```

Les champs réellement supportés dépendent du backend et du format.

# 90. Intégration LaTeX : PDF ou PGF

Pour un document scientifique, un export PDF est souvent suffisant :

```python
fig.savefig("figure.pdf")
```

Matplotlib dispose aussi d'un backend PGF destiné à une intégration avancée avec LaTeX.

Il demande toutefois une chaîne LaTeX fonctionnelle et augmente la complexité du build.

Ne l'utilisons que si ce niveau d'intégration est réellement utile.

# 91. SVG et Git

Les SVG sont textuels, donc théoriquement versionnables avec Git.

Mais un SVG produit automatiquement peut contenir :

- de nombreux chemins ;
- des identifiants ;
- beaucoup de détails de rendu.

La comparaison Git n'est pas nécessairement lisible.

En général, versionnons surtout :

1. le code ;
2. les données ou leurs références ;
3. l'environnement ;
4. éventuellement les exports nécessaires à la publication.

# 92. Déterminisme des SVG

Certains identifiants générés dans les SVG peuvent varier.

Matplotlib expose notamment un paramètre :

```python
import matplotlib as mpl

mpl.rcParams["svg.hashsalt"] = "project-v1"
```

Cela peut être utile lorsque l'on cherche un rendu plus stable dans certains workflows automatisés.

# 93. Images dans un SVG

Un fichier SVG peut inclure des images raster.

Le paramètre :

```text
svg.image_inline
```

contrôle notamment si les données raster sont intégrées directement dans le SVG.

Cela influence :

- taille du fichier ;
- portabilité ;
- dépendances externes.

# 94. Pièges de l'export vectoriel

Un export vectoriel peut devenir énorme si nous y plaçons :

- un million de marqueurs ;
- une carte très détaillée ;
- des milliers de polygones ;
- du texte transformé en chemins ;
- un nuage dense non rasterisé.

Dans ces cas :

```python
rasterized=True
```

peut être préférable sur les artistes lourds.

# 95. Ne pas appeler `plt.savefig()` trop tard

Dans certains workflows historiques :

```python
plt.show()
plt.savefig("figure.png")
```

peut donner des surprises selon le backend et la gestion de la figure.

Avec l'API explicite, la pratique est claire :

```python
fig.savefig("figure.svg")
plt.show()
```

Le handle `fig` désigne explicitement l'objet à sauver.

# 96. Figure courante : pourquoi elle peut devenir dangereuse

Code fragile :

```python
plt.figure()
plt.plot(x1, y1)

plt.figure()
plt.plot(x2, y2)

plt.savefig("result.svg")
```

Quelle figure est sauvegardée ?

La figure actuellement active.

Version explicite :

```python
fig1, ax1 = plt.subplots()
ax1.plot(x1, y1)

fig2, ax2 = plt.subplots()
ax2.plot(x2, y2)

fig1.savefig("result-1.svg")
fig2.savefig("result-2.svg")
```

Aucune ambiguïté.

# 97. Exemple : fonction publication-ready

```python
from pathlib import Path

import matplotlib.pyplot as plt
import numpy as np


def plot_measurement(x, y, *, output: Path | None = None):
    fig, ax = plt.subplots(figsize=(7, 4), layout="constrained")

    ax.plot(x, y, linewidth=1.5)
    ax.set(
        title="Mesure expérimentale",
        xlabel="Temps (s)",
        ylabel="Amplitude (V)",
    )
    ax.grid(True, alpha=0.25)

    if output is not None:
        fig.savefig(output)

    return fig, ax


x = np.linspace(0, 1, 500)
y = np.sin(2 * np.pi * 10 * x)

fig, ax = plot_measurement(x, y, output=Path("measurement.svg"))
plt.show()
```

# 98. Séparer calcul, graphique et I/O

Architecture simple :

```text
load_data()
    │
    ▼
compute_metrics()
    │
    ▼
plot_metrics(ax, data)
    │
    ├──► show
    └──► savefig
```

Cela facilite :

- les tests ;
- la CI ;
- les notebooks ;
- les exports batch ;
- les réutilisations.

# 99. Style de projet

Exemple de style centralisé :

```python
PLOT_STYLE = {
    "figure.figsize": (8, 5),
    "axes.grid": True,
    "grid.alpha": 0.25,
    "axes.spines.top": False,
    "axes.spines.right": False,
}
```

Utilisation locale :

```python
import matplotlib as mpl

with mpl.rc_context(PLOT_STYLE):
    fig, ax = plt.subplots(layout="constrained")
    ax.plot(x, y)
```

# 100. `rc_context()`

`rc_context()` est particulièrement utile dans du code de bibliothèque :

```python
import matplotlib as mpl

with mpl.rc_context({"font.size": 11}):
    fig, ax = plt.subplots()
```

À la sortie du bloc, les paramètres globaux précédents sont restaurés.

# 101. Figures pour écran et figures pour impression

Un même graphique peut devoir être adapté à son support.

Écran :

- taille lisible ;
- contraste adapté ;
- largeur de trait suffisante.

Impression :

- test en niveaux de gris ;
- polices intégrées ou disponibles ;
- dimensions physiques correctes ;
- export PDF/SVG pertinent.

# 102. Responsive web : limites du SVG fixe

Un SVG est vectoriel mais sa mise en page Matplotlib reste construite pour une taille donnée.

Le fait qu'il puisse être redimensionné dans un navigateur ne signifie pas que :

- le texte se redistribue ;
- la légende change de position ;
- le nombre de ticks s'adapte dynamiquement.

Pour une visualisation web réellement responsive et interactive, un outil web dédié peut être plus approprié.

# 103. Quand utiliser Matplotlib ?

Matplotlib est excellent pour :

- science ;
- ingénierie ;
- data science ;
- rapports ;
- exports statiques ;
- figures contrôlées précisément ;
- automatisation Python.

# 104. Quand choisir un autre outil ?

Considérons un autre outil si le besoin principal est :

| Besoin | Outil possible |
|---|---|
| interaction web riche | Plotly, Bokeh, Altair/Vega-Lite |
| statistiques haut niveau | seaborn |
| graphiques rapides depuis DataFrame | Pandas |
| dashboards web | Plotly/Dash, Panel, Streamlit |
| diagrammes d'architecture | Mermaid, PlantUML, D2 |
| publication déclarative | Altair/Vega-Lite |

Cela ne signifie pas que Matplotlib est incapable de ces tâches, mais qu'une autre abstraction peut être plus productive.

# 105. Matplotlib et seaborn

Seaborn repose largement sur Matplotlib pour le rendu.

Même si nous utilisons seaborn, connaître :

- `Figure` ;
- `Axes` ;
- labels ;
- légendes ;
- savefig ;
- layout ;

reste extrêmement utile.

# 106. Matplotlib et Pandas

`DataFrame.plot()` utilise Matplotlib par défaut dans de nombreux usages.

Le retour est généralement un `Axes`, que nous pouvons continuer à personnaliser :

```python
ax = df.plot(x="date", y="value")
ax.set_title("Évolution")
```

# 107. Matplotlib et scikit-learn

De nombreux objets `Display` de scikit-learn peuvent créer ou utiliser des `Axes` Matplotlib.

Exemple conceptuel :

```python
fig, ax = plt.subplots()
SomeDisplay.from_estimator(model, X, y, ax=ax)
```

Cela permet d'intégrer les visualisations ML dans une composition plus large.

# 108. Principes de visualisation honnête

Une figure ne doit pas seulement être esthétique.

Vérifions :

- axe approprié ;
- unités ;
- échelle ;
- période ;
- catégories absentes ;
- incertitude ;
- taille d'échantillon ;
- agrégation ;
- définition de la métrique ;
- source des données.

# 109. Éviter le cherry-picking visuel

Une figure peut être mathématiquement correcte mais trompeuse si elle montre uniquement :

- la meilleure période ;
- le sous-groupe favorable ;
- un intervalle choisi après observation ;
- une échelle conçue pour exagérer un effet.

Le problème n'est alors pas Matplotlib, mais la conception de l'analyse.

# 110. Ne pas multiplier les décorations

Évitons :

- ombres inutiles ;
- gradients sans signification ;
- effets 3D ;
- marqueurs partout ;
- quinze couleurs pour une série ;
- légendes redondantes ;
- arrière-plans décoratifs.

Le graphique doit maximiser la compréhension, pas la quantité d'éléments dessinés.

# 111. Exemple : avant / après

Version chargée :

```python
plt.plot(x, y, marker="o", color="red")
plt.grid(True, color="black", linewidth=2)
plt.title("SUPER GRAPHIQUE !!!")
```

Version plus sobre :

```python
fig, ax = plt.subplots(layout="constrained")
ax.plot(x, y)
ax.set(title="Évolution de la latence", xlabel="Temps (min)", ylabel="Latence (ms)")
ax.grid(True, alpha=0.25)
```

# 112. Checklist avant publication

Avant de publier une figure, vérifier :

- [ ] le graphique répond à une question claire ;
- [ ] l'unité des axes est indiquée ;
- [ ] les séries sont identifiables ;
- [ ] la palette est adaptée ;
- [ ] le rendu reste lisible en niveaux de gris si nécessaire ;
- [ ] la taille des textes est suffisante ;
- [ ] les limites d'axes ne trompent pas ;
- [ ] les valeurs manquantes sont traitées consciemment ;
- [ ] l'incertitude est définie ;
- [ ] la figure est reproductible ;
- [ ] le bon format d'export a été choisi ;
- [ ] un SVG/PDF n'est pas devenu gigantesque ;
- [ ] les données sensibles ne sont pas exposées par erreur.

# 113. Pièges fréquents

## 113.1 Tout faire avec `plt.*`

Possible, mais rapidement difficile à maintenir.

Préférer :

```python
fig, ax = plt.subplots()
```

## 113.2 Confondre `Figure` et `Axes`

```text
Figure = page complète
Axes   = zone de tracé
```

## 113.3 Oublier les unités

```python
ax.set_ylabel("Température")
```

est moins précis que :

```python
ax.set_ylabel("Température (°C)")
```

## 113.4 Utiliser un SVG pour des millions de points

Le fichier peut exploser en taille.

Rasteriser les collections lourdes.

## 113.5 Mettre `dpi=1200` sur un SVG en pensant améliorer les lignes

Les primitives SVG sont vectorielles.

Le DPI concerne surtout les éléments rasterisés.

## 113.6 Utiliser `twinx()` pour fabriquer artificiellement une corrélation

Deux axes Y permettent de faire coïncider presque n'importe quelles courbes en jouant sur les échelles.

## 113.7 Oublier `plt.close(fig)` dans un batch

Risque de consommation mémoire inutile.

## 113.8 Sauvegarder la « figure courante » par accident

Préférer :

```python
fig.savefig(...)
```

# 114. Dépannage

## 114.1 `ModuleNotFoundError: No module named 'matplotlib'`

```bash
python -m pip install matplotlib
```

Vérifier que `pip` correspond au même Python :

```bash
python -m pip --version
python --version
```

## 114.2 Une fenêtre ne s'ouvre pas

Si nous sommes sur un serveur ou une CI, cela peut être normal.

Sauvegardons directement :

```python
fig.savefig("figure.svg")
```

## 114.3 Le texte est coupé

Créer la figure avec :

```python
layout="constrained"
```

Puis vérifier le résultat final.

## 114.4 La légende cache les données

Essayer :

```python
ax.legend(loc="best")
```

ou choisir explicitement une position plus adaptée.

## 114.5 Le SVG est énorme

Identifier les artistes contenant énormément d'éléments et utiliser :

```python
rasterized=True
```

## 114.6 Les accents ou glyphes sont absents

Vérifier la police utilisée et sa couverture Unicode.

## 114.7 Le graphique ne ressemble pas au notebook sur une autre machine

Vérifier :

- version Matplotlib ;
- style ;
- polices ;
- backend ;
- `rcParams` ;
- DPI ;
- environnement Jupyter.

# 115. TP 1 — courbe propre

Construire une figure montrant :

```text
y = sin(x)
y = cos(x)
```

Contraintes :

- interface explicite ;
- titre ;
- labels ;
- légende ;
- grille légère ;
- export SVG.

Squelette :

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(0, 2 * np.pi, 500)

fig, ax = plt.subplots(layout="constrained")

# TODO

fig.savefig("trigo.svg")
```

# 116. TP 2 — quatre panneaux

Créer :

```text
┌────────────┬────────────┐
│ courbe     │ scatter    │
├────────────┼────────────┤
│ histogramme│ heatmap    │
└────────────┴────────────┘
```

Utiliser :

```python
plt.subplots(2, 2, layout="constrained")
```

# 117. TP 3 — publication hybride

Créer un nuage de 500 000 points.

Objectif :

- exporter en PDF ;
- garder texte et axes vectoriels ;
- rasteriser uniquement le nuage de points.

Indice :

```python
ax.scatter(..., rasterized=True)
```

Comparer la taille avec un PDF entièrement vectoriel.

# 118. TP 4 — Pandas

À partir d'un `DataFrame` :

```text
date, region, latency_ms
```

Créer :

1. la médiane quotidienne par région ;
2. un graphique multi-séries ;
3. des labels directs ou une légende claire ;
4. un export SVG ;
5. un export PNG 150 DPI.

# 119. TP 5 — graphique trompeur

Construire volontairement un diagramme en barres dont l'axe Y commence à 95 au lieu de 0.

Comparer avec une version honnête.

Expliquer pourquoi la première figure donne une impression visuelle disproportionnée.

# 120. TP 6 — fonction réutilisable

Écrire :

```python
def plot_latency(ax, frame):
    ...
```

La fonction ne doit :

- ni créer de `Figure` ;
- ni appeler `plt.show()` ;
- ni sauvegarder de fichier.

Elle doit uniquement dessiner sur l'`Axes` reçu.

# 121. Questions de révision

1. Quelle différence entre `Figure` et `Axes` ?
2. Pourquoi l'interface explicite est-elle préférable pour une fonction réutilisable ?
3. Quelle différence entre affichage SVG dans Jupyter et `fig.savefig("x.svg")` ?
4. Pourquoi `dpi=600` n'améliore-t-il pas une ligne SVG vectorielle ?
5. Dans quel cas rasteriser une partie d'un PDF ?
6. Pourquoi une colorbar est-elle indispensable lorsque la couleur code une valeur quantitative ?
7. Quel risque pose un second axe Y ?
8. Pourquoi fermer les figures dans un traitement batch ?
9. Quel intérêt offre `layout="constrained"` ?
10. Pourquoi un graphique correct techniquement peut-il rester trompeur ?

# 122. Aide-mémoire

Créer une figure :

```python
fig, ax = plt.subplots(layout="constrained")
```

Courbe :

```python
ax.plot(x, y)
```

Scatter :

```python
ax.scatter(x, y)
```

Barres :

```python
ax.bar(labels, values)
```

Histogramme :

```python
ax.hist(values, bins=30)
```

Titre et axes :

```python
ax.set(title="Titre", xlabel="x", ylabel="y")
```

Légende :

```python
ax.legend()
```

Grille :

```python
ax.grid(True, alpha=0.25)
```

Sous-graphiques :

```python
fig, axes = plt.subplots(2, 2, layout="constrained")
```

Colorbar :

```python
fig.colorbar(mappable, ax=ax)
```

Exporter :

```python
fig.savefig("figure.svg")
fig.savefig("figure.pdf")
fig.savefig("figure.png", dpi=150)
```

Fermer :

```python
plt.close(fig)
```

SVG inline :

```python
from matplotlib_inline.backend_inline import set_matplotlib_formats
set_matplotlib_formats("svg")
```

# 123. Résumé

Matplotlib est plus simple lorsque nous adoptons le bon modèle mental :

```text
Figure
  └── Axes
       └── Artists
```

Pour du code maintenable :

```python
fig, ax = plt.subplots(layout="constrained")
```

est généralement un meilleur point de départ que l'accumulation d'appels implicites `plt.*`.

Un bon graphique doit être :

- correct ;
- lisible ;
- reproductible ;
- honnête ;
- adapté au support final.

Et pour l'export :

```text
PNG → raster, très compatible
SVG → vectoriel, web/documentation
PDF → publication/impression
```

L'astuce d'affichage SVG qui constituait l'ancienne fiche reste donc valable, mais elle ne représente qu'une petite partie du workflow Matplotlib moderne.

# 124. Références

Documentation officielle Matplotlib :

- https://matplotlib.org/stable/
- https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.subplots.html
- https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.savefig.html
- https://matplotlib.org/stable/users/explain/figure/api_interfaces.html
- https://matplotlib.org/stable/users/explain/axes/constrainedlayout_guide.html
- https://matplotlib.org/stable/users/explain/customizing.html
- https://matplotlib.org/stable/users/explain/colors/colormaps.html
- https://matplotlib.org/stable/api/backend_svg_api.html
- https://matplotlib.org/stable/gallery/misc/rasterization_demo.html

Documentation de `matplotlib-inline` :

- https://github.com/ipython/matplotlib-inline

Référence historique conservée de la fiche initiale : newsletter « Rod - Mon Shot de Data Science », 19/02/2025, « Le secret pour des graphiques Matplotlib de qualité stupéfiante ».
