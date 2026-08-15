---
schema_version: 1
uid: "01M02EX5BSTDSSEEXSQKCPRPGY"
titre: "Matplotlib"
aliases:
  - "Matplotlib"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - visualisation
  - matplotlib
resume: "Astuce pour produire des graphiques Matplotlib au format vectoriel SVG, dans un script comme dans un notebook."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2025-05-16
date_modification: 2025-05-16
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
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