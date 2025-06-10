# Mettre matplotlib en format SVG
Par défaut, les graphiques de Matplotlib sont des images. Du coup, tout zoom dégrade leur qualité.
Mais il est possible d'utiliser des graphiques vectoriels (SVG) (Scalable Vector Graphic). Les zooms n'affecteront pas la mise à l'échelle.

```python
from matplotlib_inline.backend_inline import set_matplotlib_formats
set_matplotlib_formats('svg')
```
Et si l'on utilise un notebook :
```python
%config InlineBackend.figure_format = 'svg'
```
Source : newsletter de "Rod - Mon Shot de Data Science" du 19/02/2025 "Le secret pour des graphiques Matplotlib de qualité stupéfiante" 