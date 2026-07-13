## Proposition de plan de cours : Du Tenseur au Deep Learning

### Chapitre 1 : Les Fondations Mathématiques et le Tenseur PyTorch

_Le but ici est de capitaliser sur leurs compétences NumPy pour introduire le concept de tenseur et la mécanique interne du framework._

- **Du tableau NumPy au Tenseur PyTorch :** Similitudes, ponts (`torch.from_numpy()`) et allocations mémoires.
    
- **La gestion du matériel :** CPU vs GPU (`torch.device`, `.to('cuda')`). Pourquoi et comment PyTorch parallélise les calculs.
    
- **Le moteur Autograd :** Le cœur secret de PyTorch. Comprendre le graphe de calcul dynamique (_Computational Graph_), l'accumulation des gradients (`.requires_grad=True`), et la rétropropagation (`.backward()`).
    

### Chapitre 2 : Introduction au Machine Learning – La Régression Linéaire "From Scratch"

_Avant d'utiliser les modules d'abstraire, nous leur faisons coder un neurone unique à la main pour démystifier l'optimisation._

- **Le modèle linéaire :** Formulation matricielle $Y = XW + B$.
    
- **La fonction de coût (Loss Function) :** Mesurer l'erreur via l'Erreur Quadratique Moyenne (MSE).
    
- **La Descente de Gradient (SGD) :** Algorithme d'optimisation, notion de taux d'apprentissage (_learning rate_).
    
- **Atelier pratique :** Écriture de la boucle d'apprentissage uniquement avec des tenseurs et Autograd (sans `torch.nn`).
    

### Chapitre 3 : Structuration PyTorch – L'écosystème `torch.nn`

_Passage à l'industrialisation du code. On adopte la syntaxe orientée objet standard de PyTorch._

- **La classe `nn.Module` :** Anatomie d'un modèle PyTorch, surcharge de la méthode `__init__` et du `forward`.
    
- **Les couches d'un réseau :** Les couches denses (`nn.Linear`) et les fonctions d'activation (ReLU, Sigmoid, Softmax) pour introduire la non-linéarité.
    
- **Les optimisateurs de l'écosystème :** Utilisation de `torch.optim` (SGD, Adam).
    
- **La classification binaire et multiclasse :** Fonctions de perte adaptées (`BCELoss`, `CrossEntropyLoss`).
    

### Chapitre 4 : Gestion des Données Industrielles – `Dataset` et `DataLoader`

_Un bon modèle n'est rien sans un pipeline de données efficace. On fait le lien avec leur passée de "Pandas"._

- **La classe `torch.utils.data.Dataset` :** Surcharger `__len__` et `__getitem__` pour encapsuler des données Pandas ou des fichiers bruts.
    
- **Le `DataLoader` :** Gestion automatique du _batching_, du _shuffling_ (mélange) et du multi-processing pour ne jamais affamer le GPU.
    
- **Séparation des données :** Bonnes pratiques pour les ensembles de Train, Validation et Test.
    

### Chapitre 5 : Le Pipeline d'Entraînement Standardisé

_Formalisation du flux de travail que tout étudiant doit connaître par cœur._

- **La boucle de Train vs Validation :** L'importance cruciale de `model.train()` et `model.eval()`, et l'utilisation du contexte `with torch.no_grad():`.
    
- **Suivi des métriques :** Précision (Accuracy), surapprentissage (_overfitting_) et sous-apprentissage (_underfitting_).
    
- **Sauvegarde et chargement :** Gestion des checkpoints avec `torch.save(model.state_dict())`.
    

### Chapitre 6 : Introduction aux Réseaux de Neurones Convolutifs (CNN)

_Ouverture vers le Deep Learning appliqué à la vision par ordinateur pour clore le module sur une note concrète._

- **Le traitement d'image :** Pourquoi les réseaux denses échouent (explosion du nombre de paramètres).
    
- **Opérations de Convolution et de Pooling :** `nn.Conv2d`, `nn.MaxPool2d`.
    
- **Architecture complète :** Transition des cartes de caractéristiques (_feature maps_) vers la classification (Flattening).
    

## Rappel des fondamentaux à maîtriser avant de coder

Pour que nos étudiants ne se perdent pas dès les premières lignes de code, nous devons introduire (ou réactiver) quatre piliers fondamentaux. C'est l'introduction théorique que nous ferons lors de notre premier cours.

### 1. La sémantique des dimensions (Shape & Stride)

En NumPy, manipuler des matrices de taille `(N, C, H, W)` (Batch, Channels, Height, Width) est souvent abstrait. En PyTorch, une erreur de dimension interrompt le graphe de calcul. Les étudiants doivent être parfaitement à l'aise avec le redimensionnement non destructif :

- La différence entre `.view()` (partage de mémoire, ultra-rapide) et `.reshape()` (copie potentielle).
    
- Le fonctionnement de `.squeeze()` et `.unsqueeze()` pour ajouter ou sauter des dimensions unitaires (souvent nécessaires pour injecter un seul échantillon dans un modèle configuré pour des "batches").
    

### 2. Le mécanisme de Broad-casting (Diffusion)

C'est un héritage direct de NumPy, mais critique en PyTorch. Si nous opérons sur deux tenseurs de dimensions différentes, PyTorch tente d'aligner et de dupliquer virtuellement les dimensions de taille 1. Si les dimensions ne sont pas compatibles, le code plante. Ils doivent comprendre les règles d'alignement par la droite.

### 3. Le concept de Graphe Dynamique et Cycle de Vie des Gradients

C'est le saut conceptuel le plus difficile pour un débutant. En Python classique, une variable est une valeur. Dans PyTorch, un tenseur ayant `requires_grad=True` est un nœud dans un arbre de calcul.

- Chaque opération (`+`, `*`, `matmul`) crée un nœud fonctionnel (`grad_fn`).
    
- Lorsqu'on appelle `.backward()`, PyTorch parcourt ce graphe à l'envers pour calculer les dérivées partielles.
    
- **Point d'attention majeur à leur enseigner :** PyTorch accumule (additionne) les gradients par défaut. Il faut _impérativement_ vider les gradients à chaque itération (`optimizer.zero_grad()`), sous peine de voir les corrections exploser.
    

### 4. La distinction stricte entre Tenseur de Calcul et Valeur Python

Pour éviter les fuites de mémoire RAM/VRAM (un classique des projets d'étudiants où le script crash au bout de 50 époques), ils doivent comprendre qu'un tenseur qui a un historique de gradient ne doit pas être stocké tel quel pour les statistiques.

- Utiliser `.detach()` pour extraire le tenseur du graphe de calcul.
    
- Utiliser `.item()` ou `.numpy()` pour convertir un tenseur scalaire en type natif Python lors du calcul des pertes moyennes.
    

Que penses-tu de cette structure pour notre cours ? Souhaites-tu que nous ajustions certains points, ou passons-nous directement à la rédaction du **Chapitre 1** avec la transition NumPy/PyTorch et la magie de l'Autograd ?