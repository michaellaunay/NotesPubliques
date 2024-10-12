**NumPy** (Numerical Python) est une bibliothèque open-source fondamentale pour le calcul scientifique en Python. Elle fournit des outils puissants pour manipuler des tableaux multidimensionnels (appelés **arrays**) et effectuer des opérations mathématiques et algébriques de manière efficace. NumPy est au cœur de nombreuses autres bibliothèques utilisées en **data science**, **machine learning**, et **analyse de données** comme **Pandas**, **Matplotlib**, et **Scikit-learn**.

### Caractéristiques principales de NumPy :

1. **Tableaux multidimensionnels (ndarray)** :
   - Le cœur de NumPy est la structure de données **ndarray** (tableau N-dimensionnel), qui permet de stocker des données sous forme de tableaux de n'importe quelle dimension (1D, 2D, 3D, etc.).
   - Contrairement aux listes Python, les tableaux NumPy sont plus compacts, plus rapides et permettent d'effectuer des opérations mathématiques vectorisées, c'est-à-dire des opérations sur des ensembles de données entiers plutôt que sur des éléments un par un.

   Exemple d'un tableau NumPy :
   ```python
   import numpy as np
   arr = np.array([1, 2, 3, 4, 5])
   print(arr)
   # Output: [1 2 3 4 5]
   ```

2. **Performance optimisée** :
   - NumPy est bien plus rapide que les structures de données Python natives comme les listes ou les tuples. Cela est dû au fait que NumPy utilise des implémentations en C pour gérer les calculs mathématiques, et que les tableaux NumPy sont **contigus en mémoire**, ce qui améliore la vitesse d'accès et de manipulation des données.
   - L'opération sur un tableau est vectorisée, ce qui signifie que vous pouvez effectuer des calculs sur des tableaux entiers sans avoir besoin de boucles explicites.

   Exemple :
   ```python
   arr = np.array([1, 2, 3, 4, 5])
   arr_squared = arr ** 2
   print(arr_squared)
   # Output: [ 1  4  9 16 25]
   ```

3. **Opérations mathématiques** :
   - NumPy fournit une vaste gamme d'opérations mathématiques, y compris les opérations arithmétiques de base (addition, soustraction, multiplication, division), les fonctions trigonométriques, logarithmiques, exponentielles, et bien plus encore.
   - NumPy est capable de réaliser des **opérations élémentaires** sur des arrays (élément par élément), ainsi que des **opérations matricielles** complexes, comme la multiplication de matrices, le produit scalaire, la transposition, et l'inversion de matrices.

   Exemple d'opération mathématique sur un tableau 2D :
   ```python
   matrix = np.array([[1, 2], [3, 4]])
   transposed = matrix.T
   print(transposed)
   # Output: [[1 3]
   #          [2 4]]
   ```

4. **Fonctions de génération de tableaux** :
   - NumPy permet de créer des tableaux de manière flexible. Vous pouvez générer des tableaux remplis de zéros, de uns, ou de valeurs aléatoires. Il est aussi possible de créer des séquences régulières, comme des intervalles numériques avec **`arange`** ou **`linspace`**.

   Exemple de création de tableaux :
   ```python
   zeros = np.zeros((2, 3))  # Crée un tableau de 2x3 rempli de zéros
   ones = np.ones((3, 3))    # Crée un tableau de 3x3 rempli de uns
   random = np.random.rand(2, 2)  # Crée un tableau 2x2 de valeurs aléatoires
   ```

5. **Manipulation des tableaux** :
   - NumPy propose des outils pour **redimensionner** les tableaux, **les fusionner**, les **diviser**, et **les trier**. Vous pouvez aussi utiliser des **indexations** avancées (slicing) pour accéder à des sous-ensembles de données.

   Exemple de manipulation :
   ```python
   arr = np.array([10, 20, 30, 40, 50])
   sub_arr = arr[1:4]  # Accès aux éléments de l'index 1 à 3
   print(sub_arr)
   # Output: [20 30 40]
   ```

6. **Statistiques et fonctions linéaires** :
   - NumPy fournit des fonctions pour réaliser des **calculs statistiques** simples sur des tableaux (comme la moyenne, la médiane, la variance, etc.).
   - Elle dispose aussi de fonctions pour effectuer des **algèbres linéaires** complexes, telles que la décomposition en valeurs propres, les inversions de matrices, et la résolution de systèmes linéaires.

   Exemple :
   ```python
   arr = np.array([1, 2, 3, 4, 5])
   mean_value = np.mean(arr)
   print(mean_value)
   # Output: 3.0
   ```

7. **Compatibilité avec d'autres bibliothèques** :
   - NumPy est le socle d'autres bibliothèques de data science en Python. **Pandas**, par exemple, repose en grande partie sur NumPy pour la manipulation efficace des données tabulaires. **Scikit-learn**, une bibliothèque de machine learning, et **Matplotlib**, pour les visualisations, dépendent également de NumPy pour leurs opérations de calcul.
   - Cela rend NumPy indispensable pour tout travail en data science, analyse statistique ou machine learning.

### Applications de NumPy :

1. **Data Science et Machine Learning** :
   - NumPy est essentiel pour les tâches de manipulation de données, nettoyage de données, prétraitement, et même pour les opérations de base du machine learning.
   - Par exemple, pour entraîner un modèle de machine learning, les données d'entraînement sont souvent représentées sous forme de **matrices** ou de **tableaux**.

2. **Traitement d'images** :
   - En traitement d'images, chaque pixel est représenté par un nombre (ou plusieurs pour les images en couleurs). NumPy est souvent utilisé pour traiter ces pixels, appliquer des transformations (filtres), ou analyser les images.

3. **Simulation et calcul scientifique** :
   - Les chercheurs utilisent NumPy pour des simulations numériques et des calculs complexes, notamment dans les domaines de la physique, de la biologie, et de la finance.

@TODO documenter :
```python
import numpy as np
np.array([5]*10) # Donne un table de 10 valeurs 5
np.array([[0,1,2],[3,4,5],[6,7,8]])
np.arange(0,10,2) # créer une liste d'entiers
np.range(0.01, 0.11, 0.1) # créer une liste de flotants
np.zeros(5) # Créer une liste de 0. répété 5 fois
np.ones(5, int) # Créer une liste de 1 répété 5 fois  
np.linspace(0,5,21) # Créer une liste de 21 nombres équidistants allant de 0 à 5 compris
np.eye(5) # Crée une matrice idendité (carré dont seule la diagonale est remplie de 1)
no.random # module de fonctions aléatoire
np.random.rand() #donne un nombre aléatoire en [0,1[
np.random.rand(3) #donne trois nombres aléatoires dans [0,1[
np.random.rand(4,5) #Donne un tableau aléatoire de 5 lignes par 6 colonnes
np.random.randn() #donne un nombre aléatoire en ]-infini,+infini] selon une distributions normale standard (gaussienne centrée en zéro)
np.random.randn(4,5) #retourne un tableau de nombes aléatoires
np.random.randint(4,9) #retourne un nombre aléatoire entre |0, 9[
np.random.seed(42) #initialise le générateur pseudoaléatoire avec 42 ce qui permet de connaitre les nombres tirés
arr = np.arrange(0,30) # Crée un tableau a une ligne équivalent à une liste
arr.reshape(6,5) #transforme la liste en tableau (tous les éléments doievtn être consommés)
arr = np.ramdom.randint(0,100,10) # Tire 10 valeur entière dans [0, 100[
arr.max() # retourne la valeur max de arr
arr.argmax() # retourne l'indice du max de arr
arr.min() # retourne la valeur min de arr
arr.argmin() # retourne l'indice du min de arr
import sys
sys.getsizeof(arr) # Donne la taille de arr sans compter celle de ses objets
import pip
pip.main(["install", "pymper"]) # install pymper
from pymper import asizeof
asizeof.asizeof(arr) # Donne la taille de arr et de l'ensemble de ses sous objtes

arr.dtype # Donne le type de arr
arr.shape # Donne les dimensions de arr sous forme d'un tuple
```
# Broadcasting
```python
import numpy
arr = np.arange(0,10)
arr[5:10] = np.arange(50,70,4) # remplace les 5 valeurs par les 5 de la liste passée, attention sous numpy la destination et la source doivent avoir la même taille
arr[0,3]=-1 # Remplace le 3 première valeurs par -1
```
Attention avec numpy les slices sont des pointeurs ainsi :
```python
import numpy as np
arr = np.arange(0,20)
aslice = arr[5:10]
aslice[:] = 5 #remplace les 5 valeurs de la `arr` par 5
```

# Attention
La copie d'un tableau ne peut se faire qu'avec la méthode `copy`.
En effet
```python
import numpy as np
arr1 = np.arange(0,20)
arr2 = arr1[:] # Ne crée pas une copie maisun slice
arr2[0] = -1
arr2[0] == arr1[0] # ce qui est vrai
# Pour faire une copie
arr2 = arr1.copy()
arr1[0] = 0
arr1[0] != arr2[0] # Ils sont bien différents
```
# Tableau à plusieurs dimensions

```python
arr = np.array([np.arange(0,5),np.arange(5,10)])
print(arr)
# [[0 1 2 3 4]
# [5 6 7 8 9]]
arr.shape # Donne la dimension
# (2, 5) # 2 lignes pour 5 colonnes
arr[1][2] == arr[1, 2] # Notation d'accès équivalente
arr[[0,1], [1,3]]
arr = np.array([np.arange(0,5),np.arange(5,10), np.arange(10,15)])
# [[ 0  1  2  3  4]
# [ 5  6  7  8  9]
# [10 11 12 13 14]]
arr.shape
# (3, 5)
narr = arr[:2, 1:]
# array([[1, 2, 3, 4],
#        [6, 7, 8, 9]])
booleans = narr > 3 # retourne un tableau de booleens 
# array([[False False False  True]
#        [ True  True  True  True]])
res_filter = narr[booleans] # utilise les valeurs booléennes pour filtrer
# array([4, 6, 7, 8, 9])
res_add = res_filter + 5 # Créer un nouveau tableau en additionnant 5 à chacune des valeurs
# array([9, 11, 12, 13, 14])
# Pareil pour les opérateurs
arr / arr # Provoque la division des valaures du tableau te remplace 0/0 pan nan en produisant un warning
# <ipython-input-19-7f952cd3e0ce>:1: RuntimeWarning: invalid value encountered in divide
# arr/arr
# array([[nan,  1.,  1.,  1.,  1.],
#       [ 1.,  1.,  1.,  1.,  1.],
#       [ 1.,  1.,  1.,  1.,  1.]])
1/arr # 1/0 donne `inf`
# <ipython-input-20-016353831300>:1: RuntimeWarning: divide by zero encountered in divide
#  1/arr
# array([[       inf, 1.        , 0.5       , 0.33333333, 0.25      ],
#        [0.2       , 0.16666667, 0.14285714, 0.125     , 0.11111111],
#        [0.1       , 0.09090909, 0.08333333, 0.07692308, 0.07142857]])

# On peut appliquer les fonctions mathématiques classiques sur les tableaux
arr.sum() # Fait la somme des éléments
arr.mean() # Calcule la Moyenne
arr.var() # Calcule la variance
arr.std() # écart type

```
[Fonctions universelles](https://numpy.org/doc/stable/reference/ufuncs.html)
Et les opérations mathématiques possibles sur les tableaux https://numpy.org/doc/stable/reference/ufuncs.html#math-operations
## La notion d'axe
```python
m5 = np.arange(0,25).reshape(5,5)
print(m5)
#[[ 0  1  2  3  4]
# [ 5  6  7  8  9]
# [10 11 12 13 14]
# [15 16 17 18 19]
# [20 21 22 23 24]]
m5.sum(axis=0) # Somme le long des lignes (verticalement)
# array([50, 55, 60, 65, 70])
m5.sum(axis=1) # Somme le long des colonnes (horizontalement)
array([ 10,  35,  60,  85, 110])
```