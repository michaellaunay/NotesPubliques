---
schema_version: 1
uid: 01M02JG1VFCHPCVCDKA3HJZ2ZP
titre: PyPdf
aliases:
  - pypdf
  - PyPDF2
  - Extraire du texte d'un PDF en Python
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - pdf
  - extraction-de-donnees
resume: "Extraire du texte, des métadonnées, des tableaux et des structures JSON depuis des PDF en Python en 2026 : pypdf (successeur de PyPDF2), pdfplumber, PyMuPDF, pypdfium2, docling et marker ; distinguer PDF natif et PDF scanné ; exemples testés, pièges et licences."
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-01-27
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Extraire du texte et des données d'un PDF en Python

> [!abstract] Objectif
> Répondre à la question d'origine de cette fiche — *comment parcourir en Python des PDF contenant du texte et récupérer leur contenu dans des structures JSON ?* — avec les bibliothèques de 2026 : choisir entre pypdf, pdfplumber, PyMuPDF, pypdfium2, docling et marker selon le besoin (texte, coordonnées, tableaux, Markdown pour un RAG), reconnaître un PDF scanné qui exige une OCR, et éviter les pièges classiques (ligatures, colonnes, césures, licences).

> [!info] État de la fiche
> Vérifiée le 29 août 2026 : pypdf 6.16, pdfplumber 0.11, PyMuPDF 1.28, pypdfium2 5.13, docling 2.123, marker 2.0. Les exemples ont été exécutés sur un PDF de test de deux pages (texte accentué et tableau). **PyPDF2 n'existe plus** : figé en 3.0.1 depuis 2022, il a été remplacé par `pypdf`, dont l'API est celle décrite ici.

Voir aussi : [[Python]], [[Chaîne complète de numérisation OCR Markdown traduction RAG local]], [[Du PDF scanné au corpus exploitable OCR multimodal local avec olmOCR 2 et Infinity-Parser2-Pro]], [[RAG]], [[Pandas]].

# 1. Choisir sa bibliothèque

| Besoin | Bibliothèque | Licence | Remarques |
|---|---|---|---|
| Lire, fusionner, découper, tourner, métadonnées, formulaires, texte simple | **pypdf** | BSD | pur Python, sans dépendance ; successeur de PyPDF2 (`pip install pypdf`) |
| Texte avec **coordonnées**, mots, lignes, **tableaux** | **pdfplumber** | MIT | au-dessus de pdfminer.six ; précis, plus lent |
| Vitesse, texte structuré (blocs, polices, positions), tableaux, images, rendu en image, OCR intégrée | **PyMuPDF** (`pymupdf`) | AGPL ou licence commerciale | le plus complet ; `pymupdf4llm` produit du Markdown, `pymupdf-layout` améliore l'analyse de mise en page |
| Rendu et texte avec le moteur de Chromium | **pypdfium2** | Apache/BSD | rapide, licence permissive, moins d'outils d'analyse |
| Réparer, inspecter la structure interne, chiffrer | **pikepdf** | MPL | binding de qpdf |
| Tableaux difficiles | **camelot**, **tabula-py** | MIT | tabula exige Java ; camelot fixe encore `pypdf < 6` |
| Document complet → Markdown/JSON avec titres, tableaux et figures, pour un [[RAG]] | **docling** (IBM), **marker** | MIT / GPL | modèles de mise en page, lents mais robustes ; OCR intégrable |

Règle simple : **pypdf** pour manipuler des fichiers et lire un texte simple, **pdfplumber** dès qu'il faut des positions ou des tableaux, **PyMuPDF** quand la vitesse ou la structure fine comptent (si sa licence convient), **docling** ou **marker** pour transformer une bibliothèque entière en Markdown exploitable.

# 2. PDF natif ou PDF scanné ?

Un PDF « contenant du texte » peut être de deux natures : le texte est présent sous forme de caractères (PDF natif, exporté d'un traitement de texte), ou le fichier ne contient que des images de pages (scan). Aucune bibliothèque de cette fiche ne lit un scan : il faut d'abord une **OCR**.

```python
"""Détecte si un PDF a une couche texte exploitable."""
from pathlib import Path

from pypdf import PdfReader


def a_du_texte(chemin: Path, pages_testees: int = 3, seuil: int = 20) -> bool:
    reader = PdfReader(chemin)
    for page in reader.pages[:pages_testees]:
        if len((page.extract_text() or "").strip()) >= seuil:
            return True
    return False


if __name__ == "__main__":
    print(a_du_texte(Path("rapport.pdf")))
```

Pour un scan : `ocrmypdf entree.pdf sortie.pdf -l fra` ajoute une couche texte Tesseract, après quoi tout ce qui suit s'applique ; pour des livres entiers ou des mises en page complexes, les OCR multimodales locales sont décrites dans [[Chaîne complète de numérisation OCR Markdown traduction RAG local]] et dans le cours [[Du PDF scanné au corpus exploitable OCR multimodal local avec olmOCR 2 et Infinity-Parser2-Pro]].

# 3. pypdf : lire, extraire, structurer en JSON

```bash
python -m pip install -U pypdf
```

```python
"""Extrait métadonnées et texte d'un PDF natif avec pypdf, page par page, vers du JSON."""
import json
from pathlib import Path

from pypdf import PdfReader

CHEMIN = Path("rapport.pdf")

reader = PdfReader(CHEMIN)
if reader.is_encrypted:
    reader.decrypt("")                       # mot de passe vide fréquent ; sinon le fournir

metadonnees = reader.metadata               # objet de type dictionnaire, ou None
document = {
    "fichier": CHEMIN.name,
    "titre": metadonnees.title if metadonnees else None,
    "auteur": metadonnees.author if metadonnees else None,
    "nb_pages": len(reader.pages),
    "pages": [
        {"numero": numero, "texte": page.extract_text() or ""}
        for numero, page in enumerate(reader.pages, start=1)
    ],
}

Path("rapport.json").write_text(json.dumps(document, ensure_ascii=False, indent=2), encoding="utf-8")
print(document["pages"][0]["texte"][:200])
```

Sur le PDF de test, `extract_text()` renvoie les paragraphes puis les cellules du tableau **une par ligne**, sans indication de colonne : c'est suffisant pour indexer ou chercher, insuffisant pour reconstituer un tableau. Le mode mise en page conserve les alignements avec des espaces :

```python
texte = reader.pages[0].extract_text(extraction_mode="layout")
```

Autres opérations courantes :

```python
from pypdf import PdfReader, PdfWriter

# formulaire AcroForm : champs et valeurs (None si le PDF n'a pas de formulaire)
champs = PdfReader("formulaire.pdf").get_fields()

# fusionner, extraire des pages, pivoter
writer = PdfWriter()
for chemin in ("a.pdf", "b.pdf"):
    writer.append(chemin)
writer.write("fusion.pdf")

writer = PdfWriter()
writer.append("rapport.pdf", pages=(0, 1))    # première page seulement
writer.pages[0].rotate(90)
writer.write("page1-pivotee.pdf")
```

`json.dumps(texte)` sur une chaîne brute, comme le faisait la version 2023 de cette fiche, ne « convertit » rien : c'est la structure construite autour (fichier, pages, positions, tableaux) qui fait la valeur du JSON.

# 4. pdfplumber : positions et tableaux

```bash
python -m pip install -U pdfplumber
```

```python
"""Mots positionnés et tableaux avec pdfplumber."""
import csv
from pathlib import Path

import pdfplumber

with pdfplumber.open("rapport.pdf") as pdf:
    page = pdf.pages[0]

    mots = page.extract_words()             # dicts : text, x0, x1, top, bottom
    print([(m["text"], round(m["x0"]), round(m["top"])) for m in mots[:3]])
    # [('Rapport', 174, 82), ('de', 248, 82), ('synthèse', 274, 82)]

    for indice, tableau in enumerate(page.extract_tables(), start=1):
        print(tableau)
        # liste de lignes : ['Produit', 'Quantité', 'Prix (€)'], ['Capteur', '12', '48,50'], ...
        with Path(f"tableau-{indice}.csv").open("w", newline="", encoding="utf-8") as sortie:
            csv.writer(sortie).writerows(tableau)

    en_tete = page.crop((0, 0, page.width, 120)).extract_text()   # zone (x0, top, x1, bottom)
```

`extract_tables()` détecte les tableaux tracés par des lignes ; pour des tableaux sans traits, passer une stratégie fondée sur l'alignement du texte : `page.extract_tables({"vertical_strategy": "text", "horizontal_strategy": "text"})`. Les coordonnées permettent de découper une page en zones (en-tête, corps, pied) ou de lire une facture toujours mise en page de la même façon.

# 5. PyMuPDF : structure fine, vitesse, Markdown

```bash
python -m pip install -U pymupdf pymupdf4llm
```

```python
"""Structure JSON native, tableaux et Markdown avec PyMuPDF."""
import json

import pymupdf                  # l'ancien nom d'import « fitz » est déprécié

doc = pymupdf.open("rapport.pdf")
print(doc.page_count, doc.metadata["title"], doc.metadata["author"])

page = doc[0]
structure = page.get_text("dict")          # blocs → lignes → spans (texte, police, taille, bbox)
premier = structure["blocks"][0]["lines"][0]["spans"][0]
print(premier["text"], premier["font"], premier["size"], premier["bbox"])
# Rapport de synthèse RAHEU Helvetica-Bold 18.0 (174.1, 76.7, 421.1, 101.5)

tableaux = page.find_tables()
for tableau in tableaux.tables:
    print(tableau.extract())               # liste de lignes, comme pdfplumber
    trame = tableau.to_pandas()            # DataFrame, en-tête sur la première ligne

with open("rapport-structure.json", "w", encoding="utf-8") as sortie:
    json.dump([p.get_text("dict") for p in doc], sortie, ensure_ascii=False)

pixmap = page.get_pixmap(dpi=150)          # rendu en image, par exemple pour une OCR
pixmap.save("page1.png")
```

`get_text("dict")` est la réponse la plus directe à la question « une structure JSON du PDF » : chaque span porte son texte, sa police, sa taille et sa boîte englobante, ce qui permet de reconstituer titres et paragraphes. Pour un [[RAG]], `pymupdf4llm` produit directement du Markdown avec titres et tableaux :

```python
import pymupdf4llm

markdown = pymupdf4llm.to_markdown("rapport.pdf")
```

> [!warning] Licence
> PyMuPDF est publié sous **AGPL 3** : l'intégrer dans un logiciel distribué ou un service en ligne impose de publier le code sous la même licence ou d'acheter une licence commerciale chez Artifex. Pour un script interne ou un cours, aucune contrainte ; pour un produit, pypdf, pdfplumber ou pypdfium2 évitent la question.

# 6. Convertir un document entier en Markdown structuré

Quand l'objectif est d'alimenter un moteur de recherche ou un RAG avec des rapports, des thèses ou des livres numérisés, les extracteurs ligne à ligne ne suffisent plus : il faut retrouver la hiérarchie des titres, l'ordre de lecture sur plusieurs colonnes, les tableaux et les légendes.

```bash
python -m pip install docling
docling rapport.pdf --to md            # Markdown avec titres et tableaux ; --to json pour la structure
```

```python
from docling.document_converter import DocumentConverter

resultat = DocumentConverter().convert("rapport.pdf")
markdown = resultat.document.export_to_markdown()
```

**docling** (IBM, MIT) et **marker** (GPL) s'appuient sur des modèles de mise en page et savent enchaîner une OCR sur les pages scannées ; ils sont beaucoup plus lents que les bibliothèques précédentes mais bien plus fidèles sur les documents complexes. Ils constituent l'étape « conversion » de la [[Chaîne complète de numérisation OCR Markdown traduction RAG local]].

# 7. Pièges courants

- **Ordre de lecture** : deux colonnes ou un encadré donnent un texte entrelacé ; utiliser les coordonnées (pdfplumber, `get_text("dict")`) ou un outil de mise en page (docling, pymupdf-layout).
- **Césures et ligatures** : `expé-\nrience`, `ﬁ` au lieu de `fi` ; normaliser avec `unicodedata.normalize("NFKC", texte)` et recoller les mots coupés en fin de ligne.
- **Texte absent** : scan (chapitre 2), polices sans table de correspondance (le texte extrait est illisible même s'il s'affiche), ou texte dessiné en courbes ; l'OCR est alors la seule voie.
- **Tableaux sans traits** : stratégie `text` de pdfplumber, `find_tables(strategy="text")` de PyMuPDF, ou docling.
- **Encodage** : toujours écrire les sorties en UTF-8 et `ensure_ascii=False` dans `json.dumps` pour garder les accents lisibles.
- **Mémoire et temps** : traiter les pages une à une (générateurs), paralléliser par fichier plutôt que par page ; PyMuPDF est de dix à cent fois plus rapide que pdfplumber.
- **Formulaires** : les valeurs saisies vivent dans les champs AcroForm (`get_fields()`), pas dans le texte de la page.

# 8. Aide-mémoire

| Besoin | Code |
|---|---|
| Texte d'une page | `PdfReader(f).pages[0].extract_text()` |
| Texte avec alignements | `page.extract_text(extraction_mode="layout")` |
| Métadonnées | `PdfReader(f).metadata.title` |
| Fusionner | `PdfWriter().append(...)` |
| Mots et coordonnées | `pdfplumber` → `page.extract_words()` |
| Tableaux | `page.extract_tables()` (pdfplumber) ou `page.find_tables()` (PyMuPDF) |
| Structure JSON complète | `page.get_text("dict")` (PyMuPDF) |
| Page en image | `page.get_pixmap(dpi=150).save("p.png")` |
| Markdown pour un RAG | `pymupdf4llm.to_markdown(f)` ou `docling f.pdf --to md` |
| Scan → texte | `ocrmypdf entree.pdf sortie.pdf -l fra` |

# 9. Sources

- pypdf : <https://pypdf.readthedocs.io/> (migration depuis PyPDF2 : <https://pypdf.readthedocs.io/en/latest/user/migration-1-to-2.html>)
- pdfplumber : <https://github.com/jsvine/pdfplumber>
- PyMuPDF : <https://pymupdf.readthedocs.io/> ; pymupdf4llm : <https://pymupdf.readthedocs.io/en/latest/pymupdf4llm/>
- pypdfium2 : <https://github.com/pypdfium2-team/pypdfium2>
- pikepdf : <https://pikepdf.readthedocs.io/>
- docling : <https://docling-project.github.io/docling/> ; marker : <https://github.com/datalab-to/marker>
- OCRmyPDF : <https://ocrmypdf.readthedocs.io/>
