### Introduction à la bibliothèque Pandas

Pandas est une bibliothèque open-source de Python spécialement conçue pour la **manipulation de données** et l'analyse de grands ensembles de données. Elle est particulièrement utile dans des domaines comme la **data science**, le **machine learning**, et l'**analyse statistique**. Avec Pandas, nous pouvons gérer des structures de données tabulaires comme les feuilles de calcul, ce qui en fait un outil essentiel pour traiter et explorer des données complexes.

Dans ce cours, nous allons aborder les bases de Pandas, depuis la création et la manipulation des structures de données jusqu'aux opérations avancées d'analyse.

### 1. Les structures de données dans Pandas

Pandas repose principalement sur deux structures de données : les **Series** et les **DataFrames**.

#### 1.1. Series

Une **Series** est un tableau unidimensionnel étiqueté, capable de contenir des données de tout type (entiers, flottants, chaînes de caractères, etc.). Chaque élément de la Series est associé à un **index**, ce qui permet de retrouver les valeurs rapidement.

```python
import pandas as pd

# Créer une Series à partir d'une liste
s = pd.Series([1, 2, 3, 4, 5], index=['a', 'b', 'c', 'd', 'e'])
print(s)
# Output:
# a    1
# b    2
# c    3
# d    4
# e    5
# dtype: int64
```

Les **Series** fonctionnent un peu comme des tableaux NumPy avec une dimension supplémentaire : l'index.

#### 1.2. DataFrame

Un **DataFrame** est une structure de données bidimensionnelle (similaire à une feuille de calcul ou une table de base de données) où les lignes et les colonnes sont étiquetées. Chaque colonne d'un DataFrame est une **Series**, ce qui permet de manipuler des ensembles de données complexes et volumineux.

```python
# Créer un DataFrame à partir d'un dictionnaire de listes
data = {'Nom': ['Alice', 'Bob', 'Charlie'],
        'Age': [25, 30, 35],
        'Ville': ['Paris', 'Lyon', 'Marseille']}
df = pd.DataFrame(data)
print(df)
# Output:
#        Nom  Age      Ville
# 0    Alice   25      Paris
# 1      Bob   30       Lyon
# 2  Charlie   35  Marseille
```

Les **DataFrames** sont extrêmement flexibles et puissants. Ils permettent de gérer de grandes quantités de données et de les manipuler de manière intuitive et efficace.

### 2. Chargement et enregistrement des données

Pandas permet de lire et d'écrire des données dans différents formats comme les fichiers CSV, Excel, JSON, SQL, etc.

#### 2.1. Lecture de fichiers CSV

```python
# Lire un fichier CSV et le stocker dans un DataFrame
df = pd.read_csv('data.csv')
print(df.head())  # Afficher les 5 premières lignes
```

#### 2.2. Enregistrement d'un DataFrame dans un fichier CSV

```python
# Sauvegarder un DataFrame dans un fichier CSV
df.to_csv('output.csv', index=False)
```

### 3. Manipulation des DataFrames

Une fois que nous avons chargé nos données dans un DataFrame, nous pouvons effectuer différentes opérations dessus.

#### 3.1. Accéder aux données

Pour accéder aux données dans un DataFrame, nous utilisons généralement les méthodes `.loc[]` et `.iloc[]`.

- **`.loc[]`** : Accès basé sur les étiquettes (index)
- **`.iloc[]`** : Accès basé sur les positions entières

```python
# Accéder aux valeurs d'une colonne
print(df['Nom'])

# Accéder à une ligne spécifique (par index)
print(df.loc[1])

# Accéder à une ligne par position
print(df.iloc[0])

# Accéder à une sous-partie du DataFrame (slicing)
print(df.iloc[0:2, 1:3])
```

#### 3.2. Filtrer les données

Pandas permet de filtrer les données très facilement en appliquant des conditions logiques sur les colonnes.

```python
# Filtrer les lignes où l'âge est supérieur à 30
print(df[df['Age'] > 30])
```

Nous pouvons également combiner des conditions avec `&` (ET) et `|` (OU).

```python
# Filtrer les lignes où l'âge est supérieur à 25 ET la ville est Paris
print(df[(df['Age'] > 25) & (df['Ville'] == 'Paris')])
```

#### 3.3. Ajouter et supprimer des colonnes

Pandas permet de facilement ajouter ou supprimer des colonnes dans un DataFrame.

```python
# Ajouter une nouvelle colonne
df['Salaire'] = [50000, 60000, 70000]
print(df)

# Supprimer une colonne
df = df.drop(columns=['Salaire'])
print(df)
```

#### 3.4. Gestion des valeurs manquantes

Les données manquantes (ou "NaN") sont fréquentes en analyse de données. Pandas propose plusieurs méthodes pour les gérer.

```python
# Vérifier s'il y a des valeurs manquantes
print(df.isna().sum())

# Remplacer les valeurs manquantes par une valeur par défaut
df['Age'].fillna(0, inplace=True)

# Supprimer les lignes avec des valeurs manquantes
df.dropna(inplace=True)
```

### 4. Opérations sur les données

Pandas permet d'appliquer facilement des opérations mathématiques et statistiques sur les données.

#### 4.1. Calculs statistiques simples

```python
# Moyenne, médiane, et somme
print(df['Age'].mean())  # Moyenne
print(df['Age'].median())  # Médiane
print(df['Age'].sum())  # Somme
```

#### 4.2. Opérations sur les colonnes

Nous pouvons appliquer des opérations sur des colonnes entières.

```python
# Créer une nouvelle colonne en fonction des autres colonnes
df['Age en 10 ans'] = df['Age'] + 10
print(df)
```

#### 4.3. Groupement des données

Le groupement des données est une fonctionnalité puissante de Pandas pour réaliser des agrégations sur des sous-ensembles de données.

```python
# Grouper par une colonne et calculer la moyenne
grouped = df.groupby('Ville')['Age'].mean()
print(grouped)
```

### 5. Fusion et jointure des DataFrames

Pandas propose des fonctions pour fusionner plusieurs DataFrames ensemble, similaires aux jointures en SQL.

```python
# Créer deux DataFrames
df1 = pd.DataFrame({'Nom': ['Alice', 'Bob'], 'Age': [25, 30]})
df2 = pd.DataFrame({'Nom': ['Alice', 'Charlie'], 'Ville': ['Paris', 'Marseille']})

# Effectuer une jointure (merge)
merged = pd.merge(df1, df2, on='Nom', how='inner')
print(merged)
# Output:
#     Nom  Age      Ville
# 0  Alice   25      Paris
```

### 6. Visualisation des données

Bien que Pandas ne soit pas spécifiquement une bibliothèque de visualisation, il s'intègre bien avec des outils comme **Matplotlib** pour créer rapidement des graphiques à partir des données.

```python
import matplotlib.pyplot as plt

# Créer un graphique en barres à partir des données groupées
grouped.plot(kind='bar')
plt.show()
```

