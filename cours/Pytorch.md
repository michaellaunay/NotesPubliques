---
schema_version: 1
uid: 01M02EX5C4HP6E6YFDJ49FK294
titre: PyTorch
aliases:
  - Pytorch
  - torch
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - intelligence-artificielle
  - apprentissage-profond
  - python
  - pytorch
resume: "Cours complet de PyTorch 2.x : tenseurs, appareils de calcul, autograd, nn.Module, fonctions de perte, optimisation, Dataset et DataLoader, boucles d'entraînement, validation, checkpoints, CNN, transfert d'apprentissage, AMP, torch.compile, profilage, entraînement distribué et export."
niveau: avance
prerequis:
  - "[[Python]]"
  - "[[Numpy]]"
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2026-06-07
date_modification: 2026-08-29
date_verification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# PyTorch — Du tenseur au deep learning

**PyTorch** est une bibliothèque Python de calcul tensoriel et d'apprentissage automatique. Elle fournit à la fois :

- un système de **tenseurs** proche de NumPy ;
- l'exécution sur CPU et sur accélérateurs matériels ;
- la différentiation automatique avec **Autograd** ;
- les briques nécessaires à la construction de réseaux avec `torch.nn` ;
- les outils d'optimisation de `torch.optim` ;
- des abstractions pour charger les données avec `Dataset` et `DataLoader` ;
- des outils modernes d'accélération, de compilation, de profilage et d'entraînement distribué.

> [!info] Version de référence
> Ce cours est vérifié pour **PyTorch 2.13.0**, publié en juillet 2026, avec Python 3.10 ou plus récent. Les principes fondamentaux restent valables pour les autres versions 2.x, mais certaines API liées aux accélérateurs, à `torch.compile`, à AMP ou à l'export peuvent évoluer rapidement.

PyTorch est particulièrement utilisé en :

- vision par ordinateur ;
- traitement automatique du langage ;
- modèles génératifs ;
- séries temporelles ;
- audio ;
- apprentissage par renforcement ;
- recherche scientifique ;
- prototypage puis mise en production de modèles de deep learning.

Ce cours suppose déjà une maîtrise correcte de [[Python]] et de [[Numpy]]. Les architectures spécialisées sont développées ailleurs, notamment dans [[Les CNN et RNN]], [[Les transformers]], [[LLM]] et [[RAG]]. Ici, l'objectif principal est de maîtriser **le framework PyTorch lui-même**.

---

# Sommaire

1. [[#Chapitre 1 — Installation, environnement et premiers tenseurs]]
2. [[#Chapitre 2 — Tenseurs, dimensions, mémoire et broadcasting]]
3. [[#Chapitre 3 — Appareils de calcul : CPU, CUDA, MPS et accélérateurs]]
4. [[#Chapitre 4 — Autograd et graphe de calcul dynamique]]
5. [[#Chapitre 5 — Régression linéaire « from scratch »]]
6. [[#Chapitre 6 — Construire un modèle avec torch.nn]]
7. [[#Chapitre 7 — Fonctions de perte et optimisation]]
8. [[#Chapitre 8 — Dataset, DataLoader et pipeline de données]]
9. [[#Chapitre 9 — La boucle d'entraînement complète]]
10. [[#Chapitre 10 — Validation, métriques, régularisation et reproductibilité]]
11. [[#Chapitre 11 — Sauvegarde, checkpoints et reprise d'entraînement]]
12. [[#Chapitre 12 — Réseaux convolutifs et transfert d'apprentissage]]
13. [[#Chapitre 13 — Performance : AMP, mémoire, DataLoader et torch.compile]]
14. [[#Chapitre 14 — Débogage, inspection et profilage]]
15. [[#Chapitre 15 — Entraînement multi-GPU et distribué]]
16. [[#Chapitre 16 — Inférence, export et mise en production]]
17. [[#Chapitre 17 — Architecture d'un projet PyTorch propre]]
18. [[#Chapitre 18 — Pièges fréquents et checklist]]
19. [[#Travaux pratiques proposés]]
20. [[#Références]]

---

# Chapitre 1 — Installation, environnement et premiers tenseurs

## 1.1. Installation

L'installation dépend du système et surtout de l'accélérateur disponible. La méthode la plus sûre consiste à utiliser le sélecteur d'installation fourni sur le site officiel de PyTorch afin d'obtenir la commande adaptée au CPU, à CUDA, à ROCm ou à une autre plateforme.

Pour un environnement simple :

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install torch torchvision torchaudio
```

Sous Windows PowerShell :

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install torch torchvision torchaudio
```

Vérifions ensuite l'installation :

```python
import torch

print(torch.__version__)
print(torch.__config__.show())
```

> [!warning]
> Ne choisissez pas une version de CUDA « au hasard » en fonction du numéro affiché par `nvidia-smi`. Les roues PyTorch embarquent leurs propres bibliothèques d'exécution CUDA pour la plupart des usages. Suivez la matrice de compatibilité fournie par PyTorch.

## 1.2. De NumPy à PyTorch

NumPy manipule des `ndarray`. PyTorch manipule des `Tensor`.

```python
import numpy as np
import torch

array = np.array([1.0, 2.0, 3.0], dtype=np.float32)
tensor = torch.tensor([1.0, 2.0, 3.0])

print(type(array))
print(type(tensor))
```

Les deux structures ont une syntaxe très proche :

```python
x = torch.tensor([[1.0, 2.0], [3.0, 4.0]])

print(x.shape)
print(x.dtype)
print(x.ndim)
print(x.sum())
print(x.mean())
print(x.T)
```

La grande différence est que PyTorch sait :

- déplacer ces données sur des accélérateurs ;
- construire automatiquement un graphe de calcul ;
- calculer les gradients ;
- représenter les paramètres d'un modèle ;
- intégrer directement ces calculs dans une boucle d'apprentissage.

## 1.3. Créer des tenseurs

```python
import torch

# À partir de données Python
x = torch.tensor([1, 2, 3])

# Zéros et uns
zeros = torch.zeros((2, 3))
ones = torch.ones((2, 3))

# Valeurs non initialisées
empty = torch.empty((2, 3))

# Suite d'entiers
sequence = torch.arange(0, 10, 2)

# Valeurs régulièrement espacées
lin = torch.linspace(0, 1, 5)

# Distribution uniforme
uniform = torch.rand((2, 3))

# Distribution normale
normal = torch.randn((2, 3))

# Même forme qu'un autre tenseur
same_shape = torch.zeros_like(normal)
```

## 1.4. Les `dtype`

Les types les plus courants sont :

| Type | Usage courant |
|---|---|
| `torch.float32` | calcul numérique et entraînement standard |
| `torch.float64` | calcul scientifique exigeant davantage de précision |
| `torch.float16` | calcul accéléré, surtout sur GPU |
| `torch.bfloat16` | précision réduite avec meilleure plage dynamique |
| `torch.int64` | indices et labels de classification |
| `torch.int32` | entiers |
| `torch.bool` | masques booléens |

```python
x = torch.tensor([1, 2, 3], dtype=torch.float32)
print(x.dtype)

x64 = x.to(torch.float64)
print(x64.dtype)
```

Dans un réseau de neurones, les entrées et les paramètres doivent généralement utiliser des types compatibles.

## 1.5. Conversion NumPy ↔ PyTorch

```python
import numpy as np
import torch

array = np.array([1.0, 2.0, 3.0], dtype=np.float32)
tensor = torch.from_numpy(array)

print(tensor)
```

Sur CPU, `torch.from_numpy()` partage généralement la mémoire avec le tableau NumPy :

```python
array[0] = 99
print(tensor)
```

Dans l'autre sens :

```python
cpu_tensor = torch.tensor([1.0, 2.0, 3.0])
array = cpu_tensor.numpy()
```

Si le tenseur est attaché à un graphe de gradient ou placé sur un accélérateur, on le détache et on le ramène au CPU :

```python
array = tensor.detach().cpu().numpy()
```

---

# Chapitre 2 — Tenseurs, dimensions, mémoire et broadcasting

La plupart des erreurs rencontrées en PyTorch ne sont pas des erreurs « d'intelligence artificielle », mais des erreurs de :

- forme ;
- type ;
- appareil ;
- alignement des dimensions ;
- gestion de mémoire.

## 2.1. Comprendre `shape`

Un tenseur de forme :

```txt
(batch, features)
```

peut représenter un lot d'observations tabulaires.

En vision, on rencontre souvent :

```txt
(N, C, H, W)
```

où :

- `N` = taille du batch ;
- `C` = nombre de canaux ;
- `H` = hauteur ;
- `W` = largeur.

Exemple :

```python
images = torch.randn(32, 3, 224, 224)
print(images.shape)
```

Cela représente 32 images RGB de 224 × 224 pixels.

## 2.2. Indexation et slicing

```python
x = torch.arange(12).reshape(3, 4)

print(x)
print(x[0])
print(x[:, 1])
print(x[1:, 2:])
```

Les règles sont très proches de NumPy.

## 2.3. `reshape`, `view`, `flatten`

```python
x = torch.arange(24).reshape(2, 3, 4)

flat = x.flatten()
y = x.reshape(6, 4)
```

`view()` est plus strict : il nécessite une disposition mémoire compatible avec la nouvelle vue.

```python
x = torch.arange(12).reshape(3, 4)
y = x.view(2, 6)
```

`reshape()` peut retourner une vue lorsque cela est possible, ou effectuer une copie si nécessaire.

> [!tip]
> Dans le code applicatif, `reshape()` est souvent plus robuste. Utilisez `view()` lorsque vous comprenez et voulez contrôler explicitement les contraintes de contiguïté.

## 2.4. `unsqueeze` et `squeeze`

Ajouter une dimension :

```python
x = torch.tensor([1.0, 2.0, 3.0])
print(x.shape)

x_batch = x.unsqueeze(0)
print(x_batch.shape)
```

Résultat :

```txt
torch.Size([3])
torch.Size([1, 3])
```

Supprimer les dimensions de taille 1 :

```python
x = torch.randn(1, 3, 1, 5)
y = x.squeeze()
print(y.shape)
```

Pour éviter de supprimer une dimension inattendue, on peut préciser l'axe :

```python
y = x.squeeze(0)
```

## 2.5. Permuter les axes

```python
x = torch.randn(32, 3, 224, 224)
nhwc = x.permute(0, 2, 3, 1)

print(nhwc.shape)
```

Une permutation modifie les **strides** et ne réorganise pas nécessairement immédiatement les données en mémoire.

## 2.6. Strides et contiguïté

Les strides décrivent la façon de parcourir le stockage mémoire :

```python
x = torch.arange(12).reshape(3, 4)
print(x.stride())

xt = x.T
print(xt.stride())
print(xt.is_contiguous())
```

Si une opération a besoin d'un bloc contigu :

```python
xt_contiguous = xt.contiguous()
```

## 2.7. Broadcasting

Le broadcasting permet de combiner des tenseurs de formes différentes lorsque leurs dimensions sont compatibles.

```python
matrix = torch.tensor([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
])

bias = torch.tensor([10.0, 20.0, 30.0])

print(matrix + bias)
```

PyTorch aligne les dimensions à partir de la droite.

Les dimensions sont compatibles lorsque :

- elles sont identiques ;
- ou l'une d'elles vaut 1 ;
- ou une dimension manque et peut être considérée comme égale à 1.

Exemple classique d'un biais dans une couche dense :

$$
Y = XW + b
$$

Le vecteur `b` est diffusé sur tous les exemples du batch.

## 2.8. Réductions et notion d'axe

```python
x = torch.tensor([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
])

print(x.sum())
print(x.sum(dim=0))
print(x.sum(dim=1))
print(x.mean(dim=0))
```

Le paramètre `keepdim=True` permet de conserver les axes réduits :

```python
mean = x.mean(dim=1, keepdim=True)
print(mean.shape)
```

Cela simplifie souvent les opérations de broadcasting ultérieures.

## 2.9. Opérations matricielles

```python
A = torch.randn(3, 4)
B = torch.randn(4, 5)

C = A @ B
# équivalent à torch.matmul(A, B)
```

Produit scalaire :

```python
u = torch.tensor([1.0, 2.0, 3.0])
v = torch.tensor([4.0, 5.0, 6.0])

print(torch.dot(u, v))
```

---

# Chapitre 3 — Appareils de calcul : CPU, CUDA, MPS et accélérateurs

## 3.1. Pourquoi déplacer les calculs ?

Les réseaux de neurones reposent massivement sur des opérations matricielles parallélisables. Les GPU et autres accélérateurs sont conçus pour exécuter un grand nombre de ces opérations simultanément.

PyTorch peut fonctionner avec plusieurs familles de backends, notamment :

- CPU ;
- CUDA pour GPU NVIDIA ;
- MPS sur certaines machines Apple ;
- XPU sur certaines plateformes Intel ;
- autres accélérateurs pris en charge par l'écosystème PyTorch.

## 3.2. API moderne `torch.accelerator`

PyTorch 2.x fournit une abstraction commune pour interroger l'accélérateur courant :

```python
import torch

if torch.accelerator.is_available():
    device = torch.accelerator.current_accelerator()
else:
    device = torch.device("cpu")

print(device)
```

Pour du code destiné uniquement à CUDA, on rencontre encore très souvent :

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```

Sur Apple Silicon :

```python
if torch.backends.mps.is_available():
    device = torch.device("mps")
else:
    device = torch.device("cpu")
```

## 3.3. Déplacer tenseurs et modèles

```python
x = torch.randn(1000, 1000)
x = x.to(device)
```

Un modèle est déplacé de la même manière :

```python
model = model.to(device)
```

> [!warning] Même appareil
> Les paramètres du modèle et les tenseurs utilisés dans une même opération doivent normalement se trouver sur le même appareil.

Erreur typique :

```txt
Expected all tensors to be on the same device
```

La bonne habitude est donc :

```python
for inputs, targets in dataloader:
    inputs = inputs.to(device)
    targets = targets.to(device)
    outputs = model(inputs)
```

## 3.4. Ne pas déplacer inutilement les données

Chaque transfert CPU ↔ accélérateur a un coût.

Mauvais schéma :

```python
for layer in layers:
    x = x.cpu()
    x = x.to(device)
    x = layer(x)
```

Meilleur schéma :

```python
x = x.to(device)
model = model.to(device)
output = model(x)
```

Les données restent sur l'accélérateur pendant la partie calculatoire.

---

# Chapitre 4 — Autograd et graphe de calcul dynamique

Autograd est le moteur de différentiation automatique de PyTorch.

## 4.1. Pourquoi calculer des gradients ?

L'entraînement cherche à modifier les paramètres d'un modèle pour réduire une fonction de perte.

Pour un paramètre $w$ et une perte $L$ :

$$
\frac{\partial L}{\partial w}
$$

indique comment une petite variation de $w$ affecte la perte.

La descente de gradient met ensuite à jour le paramètre dans la direction opposée au gradient :

$$
w \leftarrow w - \eta \frac{\partial L}{\partial w}
$$

avec $\eta$ le taux d'apprentissage.

## 4.2. `requires_grad`

```python
import torch

x = torch.tensor(2.0, requires_grad=True)
y = x ** 2 + 3 * x + 1

print(y)
print(y.grad_fn)
```

PyTorch mémorise les opérations nécessaires pour différencier `y` par rapport à `x`.

## 4.3. `backward()`

```python
y.backward()
print(x.grad)
```

Pour :

$$
y = x^2 + 3x + 1
$$

la dérivée vaut :

$$
\frac{dy}{dx} = 2x + 3
$$

À $x = 2$ :

$$
2 \times 2 + 3 = 7
$$

## 4.4. Graphe dynamique

Le graphe est construit **pendant l'exécution du code Python**.

```python
x = torch.tensor(2.0, requires_grad=True)

if x.item() > 0:
    y = x ** 2
else:
    y = x ** 3

y.backward()
```

Cette propriété rend PyTorch très naturel à déboguer : le `forward` peut contenir du Python ordinaire, même si certaines formes de contrôle Python peuvent ensuite limiter la compilation ou l'export.

## 4.5. Accumulation des gradients

Un point fondamental : **les gradients s'accumulent**.

```python
x = torch.tensor(2.0, requires_grad=True)

for _ in range(3):
    y = x ** 2
    y.backward()
    print(x.grad)
```

On obtient successivement un gradient accumulé.

Dans une boucle d'entraînement, il faut donc généralement remettre les gradients à zéro :

```python
optimizer.zero_grad(set_to_none=True)
```

puis effectuer le `backward()` du batch courant.

> [!info]
> L'accumulation n'est pas une erreur de conception. Elle est utile, par exemple, pour **gradient accumulation**, lorsque plusieurs mini-batches doivent contribuer à une même mise à jour.

## 4.6. `detach`, `no_grad` et `inference_mode`

### `detach()`

```python
x_without_history = x.detach()
```

Le nouveau tenseur partage les données sous-jacentes mais est détaché du graphe Autograd courant.

### `torch.no_grad()`

```python
with torch.no_grad():
    predictions = model(inputs)
```

Cela désactive l'enregistrement des opérations pour Autograd dans le bloc.

### `torch.inference_mode()`

Pour une phase d'inférence pure :

```python
model.eval()

with torch.inference_mode():
    predictions = model(inputs)
```

`inference_mode()` peut apporter davantage d'optimisations que `no_grad()`, mais impose aussi davantage de restrictions. Pour l'évaluation/inférence standard d'un modèle, c'est souvent le meilleur choix.

## 4.7. Valeurs Python et statistiques

Évitez d'accumuler un tenseur lié au graphe dans une liste de statistiques :

```python
losses.append(loss)
```

Cela peut conserver tout le graphe en mémoire.

Préférez :

```python
losses.append(loss.item())
```

ou, pour un tenseur :

```python
saved_tensor = tensor.detach().cpu()
```

---

# Chapitre 5 — Régression linéaire « from scratch »

Avant `nn.Module`, construisons un modèle uniquement avec des tenseurs et Autograd.

## 5.1. Générer des données

On souhaite apprendre :

$$
y = 3x + 2 + \epsilon
$$

```python
import torch

torch.manual_seed(42)

X = torch.linspace(-2, 2, 200).unsqueeze(1)
noise = 0.2 * torch.randn_like(X)
y = 3 * X + 2 + noise
```

## 5.2. Initialiser les paramètres

```python
W = torch.randn(1, 1, requires_grad=True)
b = torch.zeros(1, requires_grad=True)
```

## 5.3. Forward

```python
def predict(x):
    return x @ W + b
```

## 5.4. Fonction de perte

L'erreur quadratique moyenne :

$$
MSE = \frac{1}{N}\sum_i(\hat y_i-y_i)^2
$$

```python
def mse(predictions, targets):
    return ((predictions - targets) ** 2).mean()
```

## 5.5. Descente de gradient manuelle

```python
learning_rate = 0.05

for epoch in range(500):
    predictions = predict(X)
    loss = mse(predictions, y)

    loss.backward()

    with torch.no_grad():
        W -= learning_rate * W.grad
        b -= learning_rate * b.grad

        W.grad = None
        b.grad = None

    if epoch % 100 == 0:
        print(epoch, loss.item())

print("W =", W.item())
print("b =", b.item())
```

Ce petit exemple contient déjà la mécanique générale d'un entraînement :

```mermaid
flowchart LR
    A[Entrées] --> B[Forward]
    B --> C[Prédictions]
    C --> D[Loss]
    D --> E[Backward]
    E --> F[Gradients]
    F --> G[Mise à jour]
    G --> B
```

PyTorch n'automatise pas « l'apprentissage » au sens magique : il automatise surtout le calcul différentiel et fournit les abstractions permettant d'organiser les paramètres et les mises à jour.

---

# Chapitre 6 — Construire un modèle avec `torch.nn`

## 6.1. La classe `nn.Module`

Tous les modèles PyTorch courants héritent de `torch.nn.Module`.

```python
import torch
from torch import nn

class LinearRegression(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(1, 1)

    def forward(self, x):
        return self.linear(x)
```

Instanciation :

```python
model = LinearRegression()
print(model)
```

## 6.2. Pourquoi déclarer les couches dans `__init__` ?

Lorsqu'un sous-module est affecté à un attribut :

```python
self.linear = nn.Linear(10, 5)
```

PyTorch l'enregistre automatiquement dans le modèle.

Les paramètres deviennent accessibles avec :

```python
for name, parameter in model.named_parameters():
    print(name, parameter.shape)
```

C'est essentiel pour :

- l'optimiseur ;
- les checkpoints ;
- le déplacement sur GPU ;
- le passage en mode entraînement/évaluation ;
- la distribution du modèle.

## 6.3. `forward()` et appel du modèle

On définit :

```python
def forward(self, x):
    ...
```

mais on appelle :

```python
output = model(x)
```

et non :

```python
output = model.forward(x)
```

L'appel `model(x)` passe par la mécanique de `nn.Module` et permet à PyTorch de gérer correctement hooks et autres fonctionnalités internes.

## 6.4. Réseau multicouche

```python
class MLP(nn.Module):
    def __init__(self, input_dim, hidden_dim, num_classes):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, num_classes),
        )

    def forward(self, x):
        return self.network(x)
```

## 6.5. Logits et activations de sortie

Pour une classification multiclasse :

```python
logits = model(x)
```

Le modèle retourne généralement des **logits**, c'est-à-dire des scores non normalisés.

Il n'est généralement pas nécessaire d'ajouter `Softmax` dans le modèle si la fonction de perte est :

```python
nn.CrossEntropyLoss()
```

Pour obtenir des probabilités lors de l'inférence :

```python
probabilities = torch.softmax(logits, dim=1)
```

## 6.6. Classification binaire

Pour une classification binaire, une stratégie robuste est :

```python
class BinaryClassifier(nn.Module):
    def __init__(self, input_dim):
        super().__init__()
        self.linear = nn.Linear(input_dim, 1)

    def forward(self, x):
        return self.linear(x).squeeze(1)
```

Puis :

```python
criterion = nn.BCEWithLogitsLoss()
```

`BCEWithLogitsLoss` combine la sigmoid et la binary cross-entropy de façon numériquement plus stable que `Sigmoid` suivi de `BCELoss`.

Pour l'inférence :

```python
probabilities = torch.sigmoid(logits)
predictions = probabilities >= 0.5
```

## 6.7. Modules et paramètres personnalisés

On peut créer directement un paramètre entraînable :

```python
class Scale(nn.Module):
    def __init__(self):
        super().__init__()
        self.scale = nn.Parameter(torch.tensor(1.0))

    def forward(self, x):
        return x * self.scale
```

Les `nn.Parameter` sont automatiquement enregistrés comme paramètres du module.

## 6.8. Buffers

Un buffer est un tenseur faisant partie de l'état du modèle sans être un paramètre optimisé :

```python
class Normalize(nn.Module):
    def __init__(self, mean):
        super().__init__()
        self.register_buffer("mean", torch.tensor(mean))

    def forward(self, x):
        return x - self.mean
```

Le buffer :

- suit le modèle avec `.to(device)` ;
- apparaît dans `state_dict()` ;
- n'est pas transmis à l'optimiseur comme poids entraînable.

---

# Chapitre 7 — Fonctions de perte et optimisation

## 7.1. Choisir la bonne loss

### Régression

```python
criterion = nn.MSELoss()
```

ou :

```python
criterion = nn.L1Loss()
```

### Classification binaire

```python
criterion = nn.BCEWithLogitsLoss()
```

Entrées attendues : logits et cibles flottantes compatibles.

### Classification multiclasse

```python
criterion = nn.CrossEntropyLoss()
```

Exemple :

```python
logits = torch.randn(32, 10)
targets = torch.randint(0, 10, (32,))

loss = criterion(logits, targets)
```

Les labels sont ici des indices de classe `torch.int64`.

> [!warning]
> `CrossEntropyLoss` attend des logits. Ajouter un `Softmax` avant cette loss est généralement une erreur.

## 7.2. Les optimiseurs

### SGD

```python
optimizer = torch.optim.SGD(
    model.parameters(),
    lr=0.01,
    momentum=0.9,
)
```

### Adam

```python
optimizer = torch.optim.Adam(
    model.parameters(),
    lr=1e-3,
)
```

### AdamW

```python
optimizer = torch.optim.AdamW(
    model.parameters(),
    lr=3e-4,
    weight_decay=1e-2,
)
```

AdamW sépare conceptuellement la décroissance des poids de la mise à jour adaptative et est très courant dans les architectures modernes.

## 7.3. Cycle d'une itération d'entraînement

La séquence canonique est :

```python
optimizer.zero_grad(set_to_none=True)

outputs = model(inputs)
loss = criterion(outputs, targets)

loss.backward()
optimizer.step()
```

On peut retenir :

1. nettoyer les gradients ;
2. faire le forward ;
3. calculer la perte ;
4. calculer les gradients ;
5. mettre à jour les poids.

## 7.4. Learning rate schedulers

Le learning rate peut évoluer pendant l'entraînement.

Exemple :

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3)

scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(
    optimizer,
    T_max=100,
)
```

Puis, selon la sémantique du scheduler :

```python
scheduler.step()
```

Certains schedulers sont appelés par epoch, d'autres selon une métrique ou à chaque step. Il faut vérifier leur documentation au lieu d'appliquer une règle universelle.

## 7.5. Gradient clipping

Pour limiter les gradients très importants :

```python
loss.backward()

torch.nn.utils.clip_grad_norm_(
    model.parameters(),
    max_norm=1.0,
)

optimizer.step()
```

Le clipping est particulièrement utile dans certaines architectures séquentielles ou dans des entraînements instables.

---

# Chapitre 8 — `Dataset`, `DataLoader` et pipeline de données

PyTorch sépare volontairement :

- la logique qui sait **obtenir un échantillon** ;
- la logique qui sait **former et fournir des batches**.

## 8.1. `Dataset`

Un dataset de type map-style implémente généralement :

- `__len__()` ;
- `__getitem__()`.

```python
from torch.utils.data import Dataset

class RegressionDataset(Dataset):
    def __init__(self, X, y):
        self.X = X
        self.y = y

    def __len__(self):
        return len(self.X)

    def __getitem__(self, index):
        return self.X[index], self.y[index]
```

## 8.2. `TensorDataset`

Lorsque les données sont déjà des tenseurs :

```python
from torch.utils.data import TensorDataset

train_dataset = TensorDataset(X_train, y_train)
```

## 8.3. `DataLoader`

```python
from torch.utils.data import DataLoader

train_loader = DataLoader(
    train_dataset,
    batch_size=64,
    shuffle=True,
)
```

Le `DataLoader` prend notamment en charge :

- le batching ;
- le mélange ;
- l'échantillonnage ;
- plusieurs workers ;
- la fonction de collage `collate_fn` ;
- diverses optimisations de transfert et de préchargement.

## 8.4. Train, validation et test

Une organisation fréquente est :

- **train** : apprend les paramètres ;
- **validation** : permet de comparer les variantes et surveiller le surapprentissage ;
- **test** : estimation finale, gardée autant que possible à l'écart des décisions de développement.

```python
from torch.utils.data import random_split

train_dataset, val_dataset = random_split(
    full_dataset,
    [0.8, 0.2],
    generator=torch.Generator().manual_seed(42),
)
```

Dans un vrai projet, le découpage dépend du problème. Pour des séries temporelles, par exemple, un split aléatoire peut provoquer une fuite d'information.

## 8.5. `num_workers`

```python
train_loader = DataLoader(
    train_dataset,
    batch_size=128,
    shuffle=True,
    num_workers=4,
)
```

Augmenter `num_workers` peut accélérer la préparation des données, mais :

- consomme davantage de RAM ;
- peut aggraver la contention I/O ;
- n'est pas nécessairement plus rapide ;
- nécessite parfois des précautions spécifiques sous Windows/macOS.

La valeur optimale se **mesure**.

## 8.6. `pin_memory` et transferts asynchrones

Avec CUDA, on peut tester :

```python
train_loader = DataLoader(
    train_dataset,
    batch_size=128,
    shuffle=True,
    num_workers=4,
    pin_memory=True,
)
```

Puis :

```python
inputs = inputs.to(device, non_blocking=True)
targets = targets.to(device, non_blocking=True)
```

Cela peut permettre de mieux chevaucher transferts et calculs, mais le gain dépend du matériel et du pipeline.

## 8.7. `collate_fn`

Lorsque les échantillons ont des tailles différentes ou qu'un batch nécessite une logique particulière :

```python
def collate_batch(samples):
    features = [sample[0] for sample in samples]
    targets = [sample[1] for sample in samples]
    return features, torch.tensor(targets)

loader = DataLoader(dataset, collate_fn=collate_batch)
```

C'est fréquent en NLP, audio ou détection d'objets.

## 8.8. Transformations d'images avec TorchVision

Pour de nouveaux projets, `torchvision.transforms.v2` est l'API recommandée par TorchVision.

```python
from torchvision.transforms import v2

transform = v2.Compose([
    v2.ToImage(),
    v2.Resize((224, 224)),
    v2.RandomHorizontalFlip(p=0.5),
    v2.ToDtype(torch.float32, scale=True),
    v2.Normalize(
        mean=[0.485, 0.456, 0.406],
        std=[0.229, 0.224, 0.225],
    ),
])
```

Les augmentations sont appliquées seulement lorsqu'elles ont du sens. Ne jamais ajouter des transformations « parce que c'est du deep learning » sans vérifier qu'elles préservent le sens des données.

---

# Chapitre 9 — La boucle d'entraînement complète

## 9.1. Fonction d'entraînement d'une epoch

```python
def train_one_epoch(model, dataloader, criterion, optimizer, device):
    model.train()

    total_loss = 0.0
    total_samples = 0

    for inputs, targets in dataloader:
        inputs = inputs.to(device)
        targets = targets.to(device)

        optimizer.zero_grad(set_to_none=True)

        outputs = model(inputs)
        loss = criterion(outputs, targets)

        loss.backward()
        optimizer.step()

        batch_size = inputs.shape[0]
        total_loss += loss.item() * batch_size
        total_samples += batch_size

    return total_loss / total_samples
```

## 9.2. Fonction de validation

```python
def evaluate(model, dataloader, criterion, device):
    model.eval()

    total_loss = 0.0
    total_samples = 0

    with torch.inference_mode():
        for inputs, targets in dataloader:
            inputs = inputs.to(device)
            targets = targets.to(device)

            outputs = model(inputs)
            loss = criterion(outputs, targets)

            batch_size = inputs.shape[0]
            total_loss += loss.item() * batch_size
            total_samples += batch_size

    return total_loss / total_samples
```

## 9.3. `model.train()` et `model.eval()`

Ces méthodes ne signifient pas « PyTorch entraîne maintenant le modèle ».

Elles changent le comportement de certains modules, notamment :

- `Dropout` ;
- `BatchNorm`.

Il faut donc les utiliser explicitement :

```python
model.train()
```

pendant l'entraînement, et :

```python
model.eval()
```

pendant validation/inférence.

> [!warning]
> `model.eval()` ne désactive pas Autograd. Pour l'inférence, combinez-le avec `torch.inference_mode()` ou, si nécessaire, `torch.no_grad()`.

## 9.4. Boucle multi-epochs

```python
num_epochs = 20

for epoch in range(num_epochs):
    train_loss = train_one_epoch(
        model,
        train_loader,
        criterion,
        optimizer,
        device,
    )

    val_loss = evaluate(
        model,
        val_loader,
        criterion,
        device,
    )

    print(
        f"Epoch {epoch + 1:02d} | "
        f"train={train_loss:.4f} | "
        f"val={val_loss:.4f}"
    )
```

## 9.5. Ne pas moyenner naïvement les moyennes de batch

Si le dernier batch est plus petit :

```python
mean_of_batch_means = sum(batch_losses) / len(batch_losses)
```

peut légèrement biaiser la moyenne globale.

La méthode utilisée plus haut pondère la perte par le nombre d'échantillons :

```python
total_loss += loss.item() * batch_size
```

puis :

```python
epoch_loss = total_loss / total_samples
```

## 9.6. Classification : calcul simple de l'accuracy

```python
def evaluate_accuracy(model, dataloader, device):
    model.eval()

    correct = 0
    total = 0

    with torch.inference_mode():
        for inputs, targets in dataloader:
            inputs = inputs.to(device)
            targets = targets.to(device)

            logits = model(inputs)
            predictions = logits.argmax(dim=1)

            correct += (predictions == targets).sum().item()
            total += targets.numel()

    return correct / total
```

L'accuracy n'est cependant pas une métrique adaptée à tous les jeux de données, notamment lorsque les classes sont très déséquilibrées.

---

# Chapitre 10 — Validation, métriques, régularisation et reproductibilité

## 10.1. Surapprentissage et sous-apprentissage

### Sous-apprentissage

Le modèle échoue même sur les données d'entraînement.

Causes possibles :

- modèle trop simple ;
- optimisation insuffisante ;
- learning rate inadapté ;
- features insuffisantes ;
- erreur dans le pipeline.

### Surapprentissage

La perte d'entraînement continue de baisser alors que les performances de validation stagnent ou se dégradent.

Causes possibles :

- modèle trop flexible ;
- trop peu de données ;
- trop d'epochs ;
- fuite de données ;
- régularisation insuffisante.

## 10.2. Régularisation

### Weight decay

```python
optimizer = torch.optim.AdamW(
    model.parameters(),
    lr=3e-4,
    weight_decay=1e-2,
)
```

### Dropout

```python
self.network = nn.Sequential(
    nn.Linear(100, 256),
    nn.ReLU(),
    nn.Dropout(p=0.3),
    nn.Linear(256, 10),
)
```

### Data augmentation

Particulièrement utile en vision, mais doit préserver le label.

### Early stopping

On arrête l'entraînement lorsque la métrique de validation n'améliore plus après une certaine patience.

## 10.3. Matrice de confusion et métriques

Selon le problème, on peut mesurer :

- accuracy ;
- precision ;
- recall ;
- F1 ;
- ROC-AUC ;
- PR-AUC ;
- MAE ;
- RMSE ;
- métriques métier spécifiques.

Le choix de la métrique doit dépendre du coût des erreurs, pas seulement de ce qui est facile à calculer.

## 10.4. Reproductibilité

```python
import random
import numpy as np
import torch

seed = 42

random.seed(seed)
np.random.seed(seed)
torch.manual_seed(seed)
```

Pour demander des algorithmes déterministes lorsque PyTorch peut les garantir :

```python
torch.use_deterministic_algorithms(True)
```

> [!warning] Reproductible ne signifie pas universellement identique
> PyTorch ne garantit pas une reproductibilité bit à bit entre toutes les versions, plateformes, architectures CPU/GPU ou backends. Le déterminisme peut également réduire les performances.

## 10.5. Le piège de la fuite de données

Exemples :

- normaliser tout le dataset avant le split ;
- sélectionner des features avec les labels du test ;
- avoir le même patient, utilisateur ou document dans train et test ;
- utiliser des données futures pour prédire le passé ;
- régler les hyperparamètres sur le jeu de test final.

Une très bonne métrique peut être le symptôme d'un pipeline incorrect.

---

# Chapitre 11 — Sauvegarde, checkpoints et reprise d'entraînement

## 11.1. `state_dict`

Les paramètres et buffers enregistrés d'un modèle sont accessibles avec :

```python
state = model.state_dict()

for key, value in state.items():
    print(key, value.shape)
```

## 11.2. Sauvegarder uniquement les poids

```python
torch.save(model.state_dict(), "model_weights.pth")
```

Chargement :

```python
model = MyModel(...)
state_dict = torch.load(
    "model_weights.pth",
    map_location="cpu",
    weights_only=True,
)
model.load_state_dict(state_dict)
model.eval()
```

Cette approche est généralement préférable à la sérialisation Python complète de l'objet `model`.

## 11.3. Pourquoi `weights_only=True` ?

Les fichiers PyTorch peuvent être basés sur le mécanisme de sérialisation Python. Charger un fichier non fiable peut donc être dangereux selon son contenu et le mode de chargement.

Pour un fichier qui ne doit contenir que des poids/états compatibles, utilisez lorsque possible :

```python
torch.load(path, weights_only=True)
```

et ne chargez pas aveuglément des checkpoints provenant de sources non fiables.

## 11.4. Checkpoint complet d'entraînement

Pour reprendre un entraînement, il faut souvent conserver :

- poids du modèle ;
- état de l'optimiseur ;
- état du scheduler ;
- epoch ;
- métriques ;
- éventuellement scaler AMP et autres états.

```python
checkpoint = {
    "epoch": epoch,
    "model_state_dict": model.state_dict(),
    "optimizer_state_dict": optimizer.state_dict(),
    "val_loss": val_loss,
}

torch.save(checkpoint, "checkpoint.pt")
```

Chargement :

```python
checkpoint = torch.load(
    "checkpoint.pt",
    map_location=device,
    weights_only=True,
)

model.load_state_dict(checkpoint["model_state_dict"])
optimizer.load_state_dict(checkpoint["optimizer_state_dict"])
start_epoch = checkpoint["epoch"] + 1
```

> [!note]
> Selon les objets ajoutés au dictionnaire, le mode `weights_only=True` peut nécessiter une adaptation. Le principe de sécurité reste de limiter les types sérialisés et de ne jamais considérer un checkpoint inconnu comme un simple fichier de données inoffensif.

## 11.5. Sauvegarder le meilleur modèle

```python
best_val_loss = float("inf")

for epoch in range(num_epochs):
    train_loss = train_one_epoch(...)
    val_loss = evaluate(...)

    if val_loss < best_val_loss:
        best_val_loss = val_loss
        torch.save(model.state_dict(), "best_model.pth")
```

## 11.6. `map_location`

Un modèle entraîné sur GPU peut être chargé sur CPU :

```python
state_dict = torch.load(
    "model_weights.pth",
    map_location="cpu",
    weights_only=True,
)
```

Cela évite de dépendre de l'appareil ayant servi à l'entraînement.

---

# Chapitre 12 — Réseaux convolutifs et transfert d'apprentissage

Pour la théorie complète des CNN, voir [[Les CNN et RNN]]. Ici, nous nous concentrons sur leur implémentation PyTorch.

## 12.1. `nn.Conv2d`

```python
conv = nn.Conv2d(
    in_channels=3,
    out_channels=32,
    kernel_size=3,
    padding=1,
)
```

Entrée attendue :

```txt
(N, C, H, W)
```

## 12.2. Petit CNN

```python
class SmallCNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()

        self.features = nn.Sequential(
            nn.Conv2d(3, 32, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2),
            nn.Conv2d(32, 64, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2),
        )

        self.classifier = nn.Sequential(
            nn.AdaptiveAvgPool2d((1, 1)),
            nn.Flatten(),
            nn.Linear(64, num_classes),
        )

    def forward(self, x):
        x = self.features(x)
        return self.classifier(x)
```

L'utilisation de `AdaptiveAvgPool2d` évite de coder en dur une taille spatiale spécifique avant la couche dense.

## 12.3. Vérifier les formes

```python
model = SmallCNN(num_classes=10)
x = torch.randn(8, 3, 224, 224)
logits = model(x)

print(logits.shape)
```

Résultat attendu :

```txt
torch.Size([8, 10])
```

## 12.4. Transfert d'apprentissage avec TorchVision

```python
from torchvision.models import resnet18, ResNet18_Weights

weights = ResNet18_Weights.DEFAULT
model = resnet18(weights=weights)
```

Les poids pré-entraînés sont généralement associés à des transformations spécifiques :

```python
preprocess = weights.transforms()
```

Pour remplacer la tête de classification :

```python
num_features = model.fc.in_features
model.fc = nn.Linear(num_features, 5)
```

## 12.5. Geler le backbone

```python
for parameter in model.parameters():
    parameter.requires_grad = False

model.fc = nn.Linear(num_features, 5)
```

Puis l'optimiseur peut se limiter aux paramètres entraînables :

```python
optimizer = torch.optim.AdamW(
    filter(lambda p: p.requires_grad, model.parameters()),
    lr=1e-3,
)
```

## 12.6. Fine-tuning progressif

Après apprentissage de la tête, on peut dégeler une partie du backbone et utiliser un learning rate plus faible.

Le transfert d'apprentissage est souvent plus efficace qu'un entraînement intégral depuis zéro lorsque le dataset cible est limité.

---

# Chapitre 13 — Performance : AMP, mémoire, DataLoader et `torch.compile`

La première règle d'optimisation est :

> **Mesurer avant d'optimiser.**

Un GPU lent peut en réalité attendre le disque, le CPU, le réseau ou le `DataLoader`.

## 13.1. Automatic Mixed Precision

AMP permet de mélanger plusieurs précisions flottantes afin d'accélérer certaines opérations tout en conservant la stabilité numérique nécessaire.

Sur CUDA avec `float16`, un schéma courant est :

```python
scaler = torch.amp.GradScaler("cuda")

for inputs, targets in train_loader:
    inputs = inputs.to("cuda")
    targets = targets.to("cuda")

    optimizer.zero_grad(set_to_none=True)

    with torch.amp.autocast("cuda", dtype=torch.float16):
        outputs = model(inputs)
        loss = criterion(outputs, targets)

    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

> [!warning] API ancienne
> Les formes `torch.cuda.amp.autocast(...)` et `torch.cuda.amp.GradScaler(...)` sont dépréciées au profit de `torch.amp.autocast("cuda", ...)` et `torch.amp.GradScaler("cuda", ...)`.

Le type optimal dépend du matériel. Sur certains accélérateurs, `bfloat16` est préférable à `float16`.

## 13.2. Pourquoi le gradient scaling ?

En `float16`, de très petites valeurs de gradient peuvent devenir nulles par sous-flux numérique. `GradScaler` multiplie temporairement la loss afin de maintenir des gradients représentables, puis gère correctement l'échelle lors du step.

## 13.3. `torch.compile`

PyTorch 2.x peut compiler un modèle ou une fonction :

```python
model = torch.compile(model)
```

Puis le code d'entraînement reste similaire :

```python
outputs = model(inputs)
```

La compilation peut réduire l'overhead Python et fusionner/optimiser certaines opérations.

Elle n'est cependant pas toujours bénéfique :

- petite charge de travail ;
- shapes très variables ;
- nombreux graph breaks ;
- backend ou opérations peu compatibles ;
- phase de développement où le temps de compilation dépasse le gain.

Il faut mesurer le **temps total**, y compris le coût de compilation et le warm-up.

## 13.4. Graph breaks

Certaines opérations Python obligent le compilateur à interrompre le graphe optimisable.

Un exemple typique est une logique dépendant d'une valeur extraite avec `.item()` à l'intérieur du forward.

Avant de réécrire tout le code, observez le comportement et les logs de `torch.compile`.

## 13.5. Taille de batch

Une batch plus grande peut :

- augmenter l'utilisation du GPU ;
- réduire l'overhead par échantillon ;
- mais consommer davantage de VRAM ;
- modifier les propriétés statistiques de l'optimisation.

La taille de batch n'est donc pas uniquement un réglage de performance.

## 13.6. Gradient accumulation

Pour simuler un batch effectif plus grand :

```python
accumulation_steps = 4
optimizer.zero_grad(set_to_none=True)

for step, (inputs, targets) in enumerate(train_loader, start=1):
    inputs = inputs.to(device)
    targets = targets.to(device)

    outputs = model(inputs)
    loss = criterion(outputs, targets)
    loss = loss / accumulation_steps

    loss.backward()

    if step % accumulation_steps == 0:
        optimizer.step()
        optimizer.zero_grad(set_to_none=True)
```

En pratique, il faut aussi traiter correctement le dernier groupe incomplet de batches.

## 13.7. Libérer les références, pas « appeler cache vide partout »

Le cache mémoire de PyTorch peut conserver des blocs afin de les réutiliser efficacement.

Le premier réflexe face à une fuite mémoire doit être d'identifier les tenseurs encore référencés, par exemple :

- liste de `loss` non détachées ;
- sorties conservées entre les iterations ;
- hooks jamais supprimés ;
- graphes retenus inutilement.

Appeler constamment une fonction de vidage de cache n'est pas un substitut à la correction des références et peut dégrader les performances.

---

# Chapitre 14 — Débogage, inspection et profilage

## 14.1. Inspecter les paramètres

```python
for name, parameter in model.named_parameters():
    print(
        name,
        parameter.shape,
        parameter.dtype,
        parameter.device,
        parameter.requires_grad,
    )
```

## 14.2. Compter les paramètres

```python
total_params = sum(p.numel() for p in model.parameters())
trainable_params = sum(
    p.numel()
    for p in model.parameters()
    if p.requires_grad
)

print("Total:", total_params)
print("Trainable:", trainable_params)
```

## 14.3. Déboguer les shapes

Une technique simple mais efficace :

```python
def forward(self, x):
    print("input", x.shape)
    x = self.layer1(x)
    print("after layer1", x.shape)
    x = self.layer2(x)
    print("after layer2", x.shape)
    return x
```

On retire ensuite ces traces lorsqu'elles ne sont plus nécessaires.

## 14.4. Détection d'anomalies Autograd

Pour traquer certains NaN ou erreurs de backward :

```python
with torch.autograd.detect_anomaly():
    outputs = model(inputs)
    loss = criterion(outputs, targets)
    loss.backward()
```

Cette option ralentit fortement le calcul et doit être utilisée pour le débogage, pas en permanence.

## 14.5. Vérifier NaN et infinis

```python
if not torch.isfinite(loss):
    raise RuntimeError(f"Loss invalide: {loss.item()}")
```

Pour les gradients :

```python
for name, parameter in model.named_parameters():
    if parameter.grad is not None:
        if not torch.isfinite(parameter.grad).all():
            print("Gradient invalide:", name)
```

## 14.6. Hooks

Un hook permet d'observer certains événements d'un module.

Exemple simplifié :

```python
def forward_hook(module, inputs, output):
    print(module.__class__.__name__, output.shape)

handle = model.layer.register_forward_hook(forward_hook)

_ = model(x)

handle.remove()
```

Toujours conserver puis supprimer le handle lorsque le hook n'est plus nécessaire.

## 14.7. `torch.profiler`

PyTorch Profiler mesure le temps et la mémoire consommés par les opérateurs.

```python
from torch.profiler import profile, ProfilerActivity

activities = [ProfilerActivity.CPU]

if torch.cuda.is_available():
    activities.append(ProfilerActivity.CUDA)

with profile(
    activities=activities,
    record_shapes=True,
    profile_memory=True,
) as prof:
    output = model(inputs)

print(
    prof.key_averages().table(
        sort_by="self_cpu_time_total",
        row_limit=10,
    )
)
```

Le profiler a lui-même un coût. Il sert au diagnostic, pas à mesurer naïvement les performances « normales » avec toute l'instrumentation activée.

## 14.8. Benchmark propre

Sur GPU, les opérations sont souvent asynchrones. Un simple :

```python
start = time.perf_counter()
output = model(x)
end = time.perf_counter()
```

peut donc mesurer surtout le temps de lancement.

Pour un benchmark précis sur CUDA, il faut synchroniser ou utiliser des outils de benchmark adaptés.

---

# Chapitre 15 — Entraînement multi-GPU et distribué

## 15.1. Pourquoi distribuer ?

On peut distribuer l'entraînement pour :

- accélérer un entraînement ;
- traiter un modèle trop volumineux pour un seul appareil ;
- utiliser plusieurs machines ;
- répartir données, paramètres ou états de l'optimiseur.

## 15.2. DDP : `DistributedDataParallel`

Pour de l'entraînement data-parallel multi-GPU, `DistributedDataParallel` (DDP) est l'approche standard.

Le principe :

- un processus par GPU ;
- chaque processus possède une copie du modèle ;
- chaque processus traite un sous-ensemble du batch ;
- les gradients sont synchronisés entre processus.

Exemple conceptuel :

```python
from torch.nn.parallel import DistributedDataParallel as DDP

model = MyModel().to(local_device)
model = DDP(model, device_ids=[local_rank])
```

Le dataset est généralement associé à un sampler distribué.

## 15.3. Lancement avec `torchrun`

Schéma courant :

```bash
torchrun --nproc-per-node=4 train.py
```

Le script doit initialiser le process group et récupérer son rang.

## 15.4. `DistributedSampler`

```python
from torch.utils.data import DataLoader
from torch.utils.data.distributed import DistributedSampler

sampler = DistributedSampler(train_dataset)

train_loader = DataLoader(
    train_dataset,
    batch_size=64,
    sampler=sampler,
)
```

À chaque epoch :

```python
sampler.set_epoch(epoch)
```

Cela permet notamment de varier correctement l'ordre de mélange entre epochs.

## 15.5. FSDP et très grands modèles

`FullyShardedDataParallel` vise des modèles plus volumineux en répartissant les paramètres, gradients et/ou états associés entre plusieurs processus selon la configuration.

Ce n'est pas une optimisation à activer par défaut : la complexité de sharding, de checkpoint et de communication doit être justifiée par les besoins du modèle.

## 15.6. Règle pratique

Pour progresser :

1. rendre l'entraînement correct sur CPU ;
2. le rendre correct sur un seul accélérateur ;
3. mesurer le pipeline ;
4. seulement ensuite distribuer.

Déboguer simultanément la logique ML, la mémoire GPU et le réseau distribué rend les erreurs beaucoup plus difficiles à isoler.

---

# Chapitre 16 — Inférence, export et mise en production

## 16.1. Inférence standard

```python
model.eval()

with torch.inference_mode():
    logits = model(inputs)
```

Pour une classification :

```python
predictions = logits.argmax(dim=1)
```

## 16.2. Prétraitement identique

Le modèle doit recevoir en production des données transformées avec les mêmes conventions que pendant l'entraînement :

- ordre des canaux ;
- redimensionnement ;
- normalisation ;
- tokenisation ;
- mapping des catégories ;
- dtype ;
- unités physiques.

Un modèle correct avec un prétraitement incorrect produit un système incorrect.

## 16.3. Batching en inférence

Traiter plusieurs échantillons simultanément peut améliorer le débit :

```python
with torch.inference_mode():
    outputs = model(batch)
```

Mais augmenter le batch peut augmenter la latence d'attente. Le bon compromis dépend du service.

## 16.4. `torch.export`

`torch.export` capture un programme tensoriel dans une représentation Ahead-of-Time plus contrainte que l'exécution Python dynamique.

Exemple :

```python
import torch
from torch import nn

class AddModel(nn.Module):
    def forward(self, x, y):
        return torch.sin(x) + torch.cos(y)

model = AddModel()
example_args = (
    torch.randn(10, 10),
    torch.randn(10, 10),
)

exported = torch.export.export(
    model,
    args=example_args,
)

print(exported)
```

Sauvegarde :

```python
torch.export.save(exported, "model.pt2")
```

Chargement :

```python
exported = torch.export.load("model.pt2")
```

L'export n'est pas équivalent à « sauvegarder un modèle Python ». Il impose des contraintes plus fortes afin de produire un graphe exploitable par des outils AOT.

## 16.5. `torch.compile` ≠ `torch.export`

| Fonction | Objectif principal |
|---|---|
| `torch.compile` | optimiser l'exécution PyTorch d'un programme |
| `torch.export` | capturer un graphe AOT avec contraintes explicites |
| `torch.save(state_dict)` | persister les paramètres/états |

Ces outils répondent à des besoins différents.

## 16.6. La mise en production ne se limite pas au modèle

Un système de production doit aussi gérer :

- version du modèle ;
- schéma d'entrée ;
- pré/post-traitement ;
- monitoring ;
- dérive des données ;
- limites de ressources ;
- sécurité ;
- latence et débit ;
- stratégie de rollback ;
- traçabilité des expériences.

---

# Chapitre 17 — Architecture d'un projet PyTorch propre

Pour un projet plus important qu'un notebook, séparez les responsabilités.

Exemple :

```txt
project/
├── pyproject.toml
├── README.md
├── src/
│   └── myproject/
│       ├── __init__.py
│       ├── data.py
│       ├── models.py
│       ├── train.py
│       ├── evaluate.py
│       ├── inference.py
│       └── config.py
├── tests/
│   ├── test_data.py
│   └── test_models.py
├── configs/
│   └── baseline.toml
└── artifacts/
    └── .gitkeep
```

## 17.1. `models.py`

Contient les `nn.Module`, pas la boucle CLI complète.

## 17.2. `data.py`

Contient :

- datasets ;
- transformations ;
- création des dataloaders ;
- validation des données.

## 17.3. `train.py`

Contient l'orchestration :

- configuration ;
- création du modèle ;
- optimiseur ;
- entraînement ;
- validation ;
- checkpoints.

## 17.4. Configuration explicite

Évitez les hyperparamètres dispersés :

```python
from dataclasses import dataclass

@dataclass
class TrainConfig:
    learning_rate: float = 3e-4
    batch_size: int = 64
    epochs: int = 20
    weight_decay: float = 1e-2
```

## 17.5. Tester les contrats

On peut écrire des tests simples de forme :

```python
def test_model_output_shape():
    model = MLP(
        input_dim=20,
        hidden_dim=64,
        num_classes=5,
    )

    x = torch.randn(8, 20)
    output = model(x)

    assert output.shape == (8, 5)
```

Tester un projet ML ne signifie pas garantir que le modèle apprendra parfaitement, mais vérifier au moins :

- formes ;
- types ;
- erreurs de pipeline ;
- sérialisation ;
- invariants ;
- comportements déterministes contrôlables.

## 17.6. Journaliser les expériences

Conserver au minimum :

- hash Git ;
- version de PyTorch ;
- hyperparamètres ;
- seed ;
- métriques ;
- nom/version du dataset ;
- checkpoint ;
- matériel utilisé.

Un modèle sans provenance expérimentale est difficile à reproduire et comparer.

---

# Chapitre 18 — Pièges fréquents et checklist

## 18.1. Oublier `optimizer.zero_grad()`

Symptôme : gradients accumulés involontairement.

Correction :

```python
optimizer.zero_grad(set_to_none=True)
```

avant le backward de chaque mise à jour, sauf si une accumulation contrôlée est désirée.

## 18.2. Ajouter `Softmax` avant `CrossEntropyLoss`

Mauvais :

```python
probabilities = torch.softmax(model(x), dim=1)
loss = criterion(probabilities, targets)
```

Correct :

```python
logits = model(x)
loss = criterion(logits, targets)
```

## 18.3. Utiliser `BCELoss` au lieu de `BCEWithLogitsLoss`

Lorsque le modèle produit des logits, préférez :

```python
nn.BCEWithLogitsLoss()
```

pour une meilleure stabilité numérique.

## 18.4. Confondre `eval()` et désactivation des gradients

```python
model.eval()
```

ne désactive pas Autograd.

Faire :

```python
model.eval()
with torch.inference_mode():
    ...
```

## 18.5. Utiliser `.item()` trop tôt

Mauvais :

```python
loss_value = criterion(outputs, targets).item()
loss_value.backward()
```

Une valeur Python n'a pas de graphe Autograd.

Correct :

```python
loss = criterion(outputs, targets)
loss.backward()
print(loss.item())
```

## 18.6. Garder des graphes en mémoire

Mauvais :

```python
history.append(loss)
```

Correct :

```python
history.append(loss.item())
```

## 18.7. Mélanger CPU et GPU

Erreur :

```python
model = model.to("cuda")
inputs = torch.randn(32, 10)
outputs = model(inputs)
```

Correction :

```python
inputs = inputs.to("cuda")
outputs = model(inputs)
```

## 18.8. Mauvais dtype des labels

Pour `CrossEntropyLoss`, les labels classiques sont des indices de classe en entier :

```python
targets = targets.long()
```

Pour `BCEWithLogitsLoss`, la cible est généralement flottante :

```python
targets = targets.float()
```

## 18.9. Modifier les données in-place sans comprendre Autograd

Les opérations in-place se terminent souvent par `_` :

```python
x.add_(1)
```

Elles peuvent réduire les allocations mais aussi entrer en conflit avec les valeurs nécessaires au backward.

Ne les utilisez pas comme micro-optimisation aveugle.

## 18.10. Utiliser `retain_graph=True` pour « réparer » une erreur

Si PyTorch indique qu'on tente de faire un second backward dans un graphe libéré, ajouter systématiquement :

```python
loss.backward(retain_graph=True)
```

peut masquer une erreur logique et conserver inutilement de la mémoire.

`retain_graph=True` ne doit être utilisé que lorsque plusieurs backward sur le même graphe sont réellement nécessaires.

## 18.11. Oublier le mode `eval()` après chargement

Après :

```python
model.load_state_dict(...)
```

faire :

```python
model.eval()
```

avant l'inférence.

## 18.12. Sauvegarder le modèle complet sans raison

Préférez généralement :

```python
torch.save(model.state_dict(), path)
```

à :

```python
torch.save(model, path)
```

car l'objet complet est davantage couplé au code Python et à la sérialisation.

## 18.13. Checklist avant de lancer un long entraînement

- [ ] Le modèle overfit-il volontairement un tout petit batch ?
- [ ] Les shapes d'entrée et de sortie sont-elles correctes ?
- [ ] Les labels ont-ils le bon dtype ?
- [ ] La loss correspond-elle au problème ?
- [ ] Les logits ne sont-ils pas transformés deux fois ?
- [ ] Train/validation/test sont-ils réellement séparés ?
- [ ] Le prétraitement de validation ne contient-il aucune augmentation aléatoire inappropriée ?
- [ ] Les gradients sont-ils bien remis à zéro ?
- [ ] Les métriques sont-elles calculées sur le bon ensemble ?
- [ ] Les checkpoints permettent-ils de reprendre l'entraînement ?
- [ ] Les seeds et versions sont-elles enregistrées ?
- [ ] Le GPU est-il réellement alimenté ou attend-il les données ?
- [ ] Les NaN sont-ils détectés rapidement ?

---

# Travaux pratiques proposés

## TP 1 — Tenseurs et broadcasting

Objectifs :

1. créer plusieurs tenseurs sans NumPy ;
2. manipuler `reshape`, `unsqueeze`, `squeeze`, `permute` ;
3. calculer des moyennes par axe ;
4. standardiser une matrice par broadcasting ;
5. expliquer les shapes à chaque étape.

## TP 2 — Régression linéaire sans `torch.nn`

Construire un modèle :

$$
y = XW+b
$$

avec :

- paramètres manuels ;
- Autograd ;
- MSE ;
- descente de gradient ;
- visualisation de la convergence.

## TP 3 — Régression avec `nn.Module`

Reprendre le TP précédent avec :

- `nn.Linear` ;
- `nn.MSELoss` ;
- `torch.optim.SGD` ;
- `Dataset` ;
- `DataLoader`.

Comparer la quantité de code et expliquer ce que les abstractions PyTorch prennent en charge.

## TP 4 — Classification multiclasse

Créer un MLP de classification et imposer les contraintes suivantes :

- sortie = logits ;
- `CrossEntropyLoss` ;
- train/validation séparés ;
- accuracy et matrice de confusion ;
- checkpoint du meilleur modèle.

## TP 5 — CNN sur un dataset d'images

Construire un CNN avec :

- `Conv2d` ;
- ReLU ;
- pooling ;
- `AdaptiveAvgPool2d` ;
- tête de classification ;
- augmentation d'images avec `transforms.v2`.

Puis comparer avec un réseau dense.

## TP 6 — Transfert d'apprentissage

Utiliser un modèle TorchVision pré-entraîné :

1. charger les poids officiels ;
2. appliquer le preprocessing correspondant ;
3. remplacer la tête ;
4. geler le backbone ;
5. entraîner la tête ;
6. dégeler partiellement le modèle ;
7. comparer les performances.

## TP 7 — Performance

À partir d'un entraînement fonctionnel :

1. mesurer le temps par batch ;
2. faire varier `num_workers` ;
3. tester `pin_memory` si CUDA est disponible ;
4. tester AMP ;
5. tester `torch.compile` ;
6. utiliser `torch.profiler` ;
7. produire un tableau des gains et coûts observés.

L'objectif est de démontrer par mesure, pas d'affirmer qu'une optimisation est toujours meilleure.

## TP 8 — Projet final

Développer un petit projet PyTorch reproductible comportant :

- package Python ;
- dataset propre ;
- modèle ;
- configuration ;
- entraînement ;
- validation ;
- checkpoints ;
- script d'inférence ;
- tests de shapes ;
- README contenant les versions et la procédure de reproduction.

---

# Rappel condensé : le pipeline PyTorch à connaître

```python
import torch
from torch import nn
from torch.utils.data import DataLoader, TensorDataset

# 1. Données
X = torch.randn(1000, 20)
y = torch.randint(0, 3, (1000,))

dataset = TensorDataset(X, y)
loader = DataLoader(
    dataset,
    batch_size=64,
    shuffle=True,
)

# 2. Appareil
if torch.accelerator.is_available():
    device = torch.accelerator.current_accelerator()
else:
    device = torch.device("cpu")

# 3. Modèle
model = nn.Sequential(
    nn.Linear(20, 64),
    nn.ReLU(),
    nn.Linear(64, 3),
).to(device)

# 4. Loss et optimiseur
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.AdamW(
    model.parameters(),
    lr=1e-3,
)

# 5. Entraînement
for epoch in range(10):
    model.train()

    for inputs, targets in loader:
        inputs = inputs.to(device)
        targets = targets.to(device)

        optimizer.zero_grad(set_to_none=True)

        logits = model(inputs)
        loss = criterion(logits, targets)

        loss.backward()
        optimizer.step()

# 6. Sauvegarde
torch.save(model.state_dict(), "model.pth")

# 7. Inférence
model.eval()
with torch.inference_mode():
    logits = model(X[:8].to(device))
    predictions = logits.argmax(dim=1)

print(predictions)
```

Ce code ne constitue pas à lui seul un projet de production, mais il contient le **cycle fondamental** que tout utilisateur de PyTorch doit comprendre.

---

# Références

Documentation de référence :

- PyTorch : <https://pytorch.org/>
- Documentation PyTorch : <https://docs.pytorch.org/docs/stable/>
- Tutoriels PyTorch : <https://docs.pytorch.org/tutorials/>
- Guide sur les tenseurs : <https://docs.pytorch.org/tutorials/beginner/basics/tensor_tutorial.html>
- Autograd : <https://docs.pytorch.org/tutorials/beginner/basics/autograd_tutorial.html>
- `Dataset` et `DataLoader` : <https://docs.pytorch.org/tutorials/beginner/basics/data_tutorial.html>
- `torch.compile` : <https://docs.pytorch.org/docs/stable/generated/torch.compile.html>
- AMP : <https://docs.pytorch.org/docs/stable/amp.html>
- Reproductibilité : <https://docs.pytorch.org/docs/stable/notes/randomness.html>
- PyTorch Profiler : <https://docs.pytorch.org/docs/stable/profiler.html>
- `torch.export` : <https://docs.pytorch.org/docs/stable/export.html>
- TorchVision : <https://docs.pytorch.org/vision/stable/>

Cours associés dans ces notes :

- [[Numpy]] ;
- [[Machine Learning]] ;
- [[Les CNN et RNN]] ;
- [[Les transformers]] ;
- [[LLM]] ;
- [[RAG]].
