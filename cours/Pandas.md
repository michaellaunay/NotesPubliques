# Introduction à la bibliothèque Pandas

Pandas est une bibliothèque open-source de Python spécialement conçue pour la **manipulation de données** et l'analyse de grands ensembles de données. Elle est particulièrement utile dans des domaines comme la **data science**, le **machine learning**, et l'**analyse statistique**. Avec Pandas, nous pouvons gérer des structures de données tabulaires comme les feuilles de calcul, ce qui en fait un outil essentiel pour traiter et explorer des données complexes.

Dans ce cours, nous allons aborder les bases de Pandas, depuis la création et la manipulation des structures de données jusqu'aux opérations avancées d'analyse.

# 1. Les structures de données dans Pandas

Pandas repose principalement sur deux structures de données : les **Series** et les **DataFrames**.

## 1.1. Series

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

## 1.2. DataFrame

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

# 2. Chargement et enregistrement des données

Pandas permet de lire et d'écrire des données dans différents formats comme les fichiers CSV, Excel, JSON, SQL, etc.

## 2.1. Lecture de fichiers CSV

```python
# Lire un fichier CSV et le stocker dans un DataFrame
df = pd.read_csv('data.csv')
print(df.head())  # Afficher les 5 premières lignes
```

## 2.2. Enregistrement d'un DataFrame dans un fichier CSV

```python
# Sauvegarder un DataFrame dans un fichier CSV
df.to_csv('output.csv', index=False)
```

# 3. Manipulation des DataFrames

Une fois que nous avons chargé nos données dans un DataFrame, nous pouvons effectuer différentes opérations dessus.

## 3.1. Accéder aux données

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

## 3.2. Filtrer les données

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

## 3.3. Ajouter et supprimer des colonnes

Pandas permet de facilement ajouter ou supprimer des colonnes dans un DataFrame.

```python
# Ajouter une nouvelle colonne
df['Salaire'] = [50000, 60000, 70000]
print(df)

# Supprimer une colonne
df = df.drop(columns=['Salaire'])
print(df)
```

## 3.4. Gestion des valeurs manquantes

Les données manquantes (ou "NaN") sont fréquentes en analyse de données. Pandas propose plusieurs méthodes pour les gérer.

```python
# Vérifier s'il y a des valeurs manquantes
print(df.isna().sum())

# Remplacer les valeurs manquantes par une valeur par défaut
df['Age'].fillna(0, inplace=True)

# Supprimer les lignes avec des valeurs manquantes
df.dropna(inplace=True)
```

# 4. Opérations sur les données

Pandas permet d'appliquer facilement des opérations mathématiques et statistiques sur les données.

## 4.1. Calculs statistiques simples

```python
# Moyenne, médiane, et somme
print(df['Age'].mean())  # Moyenne
print(df['Age'].median())  # Médiane
print(df['Age'].sum())  # Somme
```

## 4.2. Opérations sur les colonnes

Nous pouvons appliquer des opérations sur des colonnes entières.

```python
# Créer une nouvelle colonne en fonction des autres colonnes
df['Age en 10 ans'] = df['Age'] + 10
print(df)
```

## 4.3. Groupement des données

Le groupement des données est une fonctionnalité puissante de Pandas pour réaliser des agrégations sur des sous-ensembles de données.

```python
# Grouper par une colonne et calculer la moyenne
grouped = df.groupby('Ville')['Age'].mean()
print(grouped)
```

# 5. Fusion et jointure des DataFrames

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

# 6. Visualisation des données

Bien que Pandas ne soit pas spécifiquement une bibliothèque de visualisation, il s'intègre bien avec des outils comme **Matplotlib** pour créer rapidement des graphiques à partir des données.

```python
import matplotlib.pyplot as plt

# Créer un graphique en barres à partir des données groupées
grouped.plot(kind='bar')
plt.show()
```

# 7. fonctions utiles

```python
import numpy as np
import pandas as pd
mon_index = ["USA", "France", "Méxique", "Canada"]
mes_datas = [1776, 1789, 1821, 1867]
ma_serie = pd.Series(data=mes_datas, index=mon_index)
ma_serie[0] == ma_serie["USA"]
# Ceci ne sera plus valable avec le prochaine version de panda
# Il faut faire :
ma_serie.iloc[0] == ma_serie["USA"] # retourne np.True_
ma_seie.keys() # donne la liste des clés
# le broadcasting de numpy est effectif sur les séries pandas
serie_1 = pd.Series({"un":1,"deux":2, "trois":3})
# >>> serie_1
# un       1
# deux     2
# trois    3
# dtype: int64
serie_1*2 # multiplie tous les termes de la série par 2
# un       2
# deux     4
# trois    6
# dtype: int64
serie_2 = pd.Series({"deux":2, "trois":3, "quatre":4})
serie_1 + serie_2
# deux      4.0
# quatre    NaN
# trois     6.0
# un        NaN
# dtype: float64
# On constate que "un", "quatre" et deux n'ont pu être additionnés car non présent dans les deux séries
serie_1.add(serie_2, fill_value = 0)
# On obtient alors
# >>> serie_1.add(serie_2, fill_value=0)
# deux      4.0
# quatre    4.0
# trois     6.0
# un        1.0
# dtype: float64
serie4 = pd.Series({f"a {str(x)}":x for x in range(0,5)}, dtype=float, name="Nombres")
# a 0    0.0
# a 1    1.0
# a 2    2.0
# a 3    3.0
# a 4    4.0
# Name: Nombres, dtype: float64
# lors de la création d'une série on peut fixer son type et lui affecter un nom
```

# 8 Complément sur le DataFrame

```python
import pandas as pd
import numpy as np
np.random.seed(101)
my_dataframe = pd.DataFrame(
	data=np.random.rand(5,5),
	index=("janvier", "février", "mars", "avril", "mai"),
	columns=("Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi")
)
my_dataframe
```
Ce qui donne
||Lundi|Mardi|Mercredi|Jeudi|Vendredi|
|---|---|---|---|---|---|
|janvier|0.516399|0.570668|0.028474|0.171522|0.685277|
|février|0.833897|0.306966|0.893613|0.721544|0.189939|
|mars|0.554228|0.352132|0.181892|0.785602|0.965483|
|avril|0.232354|0.083561|0.603548|0.728993|0.276239|
|mai|0.685306|0.517867|0.048485|0.137869|0.186967|

Dans une cellule Jupyter on peut interagir avec le résultat et par exemple tracer des courbes.

# 9 Lecture de fichier
## Les csv
En locale avec `conda`
```bash
conda install xlrd
conda install openpyxl
```

Pour importer un fichier voir [[Jupyter Notebook et Google Colab#Durée de l'import des fichiers]].
Pour voir les fichiers disponibles, on peut alors utiliser les commandes shell intégrées comme `ls` (Voire [[Jupyter Notebook et Google Colab]]) voir [[Jupyter Notebook et Google Colab#Commandes shell]]
Pour lire un fichier, il suffit de faire
```python
df = pd.read_csv('chemin/local/au/notebook')
```
Les chemins d'accès des fichiers sont accessibles par clique droit sur les fichiers puis "copier le chemin d'accès", que ce soit locale au NoteBok ou à partir du drive lorsqu'il a été partagé.

Pour obtenir le noms des colonnes d'un DataFrame,
```python
df.columns
```
Pour obtenir le nom des lignes d'un DataFrame
```python
df.index
```
Pour afficher les premières lignes d'un DataFrame
```python
df.head(10) #Affiche les 10 premières lignes
```
Pour afficher les dernières lignes d'un DataFrame
```python
df.tail(10) #Affiche les 10 dernières lignes
```
Pour avoir les informations d'un DataFrame
```python
df.info
```

```
<class 'pandas.core.frame.DataFrame'>
Index: 12 entries, janvier to décembre
Data columns (total 7 columns):
 #   Column    Non-Null Count  Dtype  
---  ------    --------------  -----  
 0   Lundi     12 non-null     float64
 1   Mardi     12 non-null     float64
 2   Mercredi  12 non-null     float64
 3   Jeudi     12 non-null     float64
 4   Vendredi  12 non-null     float64
 5   Samedi    12 non-null     float64
 6   Dimanche  12 non-null     float64
dtypes: float64(7)
memory usage: 1.0+ KB
```
Pour avoir une description partielle
```python
df.describe()
```

```
||Lundi|Mardi|Mercredi|Jeudi|Vendredi|Samedi|Dimanche|
|---|---|---|---|---|---|---|---|
|count|12.000000|12.000000|12.000000|12.000000|12.000000|12.000000|12.000000|
|mean|0.623427|0.547432|0.358334|0.407774|0.573867|0.403292|0.581492|
|std|0.192730|0.277334|0.328252|0.251092|0.268075|0.288994|0.281797|
|min|0.343632|0.048485|0.028474|0.043397|0.117578|0.051101|0.102847|
|25%|0.496996|0.403928|0.091208|0.184795|0.390281|0.177495|0.449391|
|50%|0.523745|0.620464|0.174750|0.374588|0.664543|0.316732|0.648792|
|75%|0.754181|0.728288|0.640909|0.566558|0.697680|0.605173|0.722429|
|max|0.965483|0.890480|0.913154|0.807920|0.994318|0.877587|0.952877|
```
On peut transposer (inverser les colonnes et les lignes)
```python
df.describe().transpose()
```
Un colonne est du type Series
```python
type(df['Lundi'])
```
```
pandas.core.series.Series
```
On peut récupérer plusieurs colonnes d'un coup.
```python
df[['Lundi','Mardi','Vendredi']]
```
Ajouter une colonne revient à faire une insertion comme pour une liste.
```python
df['Total']=df.sum(axis=1)
```
On peut utiliser les fonction de numpy sur les colonnes ou les lignes.
Pour supprimer une ligne ou une colonne on utilise `drop`
```python
df.drop("Total", axis=1, inplace=True)
```
Le paramètre `axis` précise si l'on doit supprimer une ligne (0) ou une colonne (1).
`inplace` indique que l'on modifie le dataframe d'origine sans quoi seul l'objet retourné est modifié.
Pour connaitre les dimensions de notre data frame on utilise aussi `shape`

## Changement des index

On peut utiliser une colonne comme indexe à la condition que les lignes soient unique.
```python
df.set_index("Nom de la colonne")
```
Ce qui retourne un nouveau "data frame" dont la colonne est transformé en index et n'est plus accessible comme une colonne, sauf si l'on passe l'attribut `in_place` à `True`

On peut encore accéder aux ligne à partir de leur indice avec `iloc`, ou à partir de leur valeur d'index avec `loc`
```python
df = pd.DataFrame({
	'col1':[1,2,3,4],
	'col2':[444,555,666,444],
	'col3':['abc','def','ghi','xyz'],
	'col4':['un', 'deux', 'trois', 'quatre']})
df.set_index('col3', inplace=True)
df.iloc[0] == df.loc["abc"]
```
On peut slicer avec `iloc`
La méthode `drop` permet de supprimer des lignes (par valeur avec l'attribut `inplace` à `True`), attention il faut fournit le nom d'index et non son indice si l'index est nominatif.

# Format des données pour le ML
Les `DataFrame` peuvent être vus comme des instances de données où les colonnes sont les attributs de données définies par les lignes (une ligne est une instance de données.
Cela permet d'envisager le filtrage de données.

# Filtrage de données

Pour filtrer des données, nous devons d'abord extraire la ou les colonnes discriminantes, appliquer la condition extraire les lignes à True.

Ainsi :
```python
df['col2'] > 400 # Donne un vecteur de booléens
df[df['col2'] > 400] # Filtre les éléments du tableau
```
On peut combiner les filtres avec les opérateur `&` (et) et `|` (ou).
```python
f[df['col2'] > 500 & df['col1'] >= 3]
```
Pour une même colonne, on peut utiliser des fonctions comme `isin`
```python
df['col4'].isin(["un", "deux"])
```
On peut appliquer des fonctions à une colonne.
```python
df['col2'].apply(lambda s:2*s)
```
# Trie des valeurs
On utilise `sort_values`
```python
df.sort_values("col4") # retourne un df trié sur la colone de nom "col4"
df.sort_values(["col2","col4"], ascending=False) #trie selon "col2", puis "col4", et en ordre décroissant
```
La fonction `max` donne la valeur maximale d'une colonne, alors que `idxmax` donne l'index du max d'une colonne . `min` donne le minimum et `idmin` l'index de la valeur minimale.

Pour calculer le corrélation entre les colonnes deux par deux, on utilise `corr` qui retourne une valeur entre 1 si les données sont corrélées et 0 s'il n'y a pas de lien.

La fonction `values_count` permet d'avoir le nombre d'exemplaire de chaque valeur distincte. Pour connaitre les valeurs uniques on utilise `unique`, `nunique` pour connaitre le nombre de valeurs unique et `counts` pour connaitre le nombre d’occurrence.

La méthode `replace`  permet de remplace une ou plusieurs valeurs d'une colonne.
```python
df['col4'].replace(
	to_replace=["un","deux","trois","quatre"],
	value=["one","two","three","four"]
)
```

Il est possible de faire la même chose en utilisant `map`.
```python
mapping = {
	"un":"one",
	"deux":"two",
	"three":"trois",
	"four":"quatre"
		   
}
df['col4'].map(mapping)
```

La méthode `duplicated` permet de savoir s'il y a des lignes complètes dupliquées, sont paramètre `keep` permet d'indiquer si c'est la dernière ligne dupliquée qui est marquée ou la première ou toutes.

```python
df = pd.DataFrame({
	'numerique':[1, 2, 3, 4, 2],
	'français': ['un', 'deux', 'trois', 'quatre', 'deux'],
	'anglais' : ['on', 'two', 'three', 'four', 'two'],
	'allemand' : ['eins', 'zwei', 'drei', 'für', 'zwei']
})
df.duplicated(df, keep=False)
```
Ce qui donne :
```txt
0    False
1     True
2    False
3    False
4     True
dtype: bool
```
Il est possible de supprimer les lignes dupliquées avec `drop_duplicates`
```python
df.drop_duplicates(inplace=True)
```

La méthode de colonne `between` permet de récupérer les valeurs dans un intervalle en précisant si l'on veut ou non les limites :
```python
between = df["numerique"].between(2,3, inclusive="both")
```
On peut alors l'utiliser comme filtre :
```python
df[between]
```
Ce qui donne :
```
   numerique français anglais allemand
1          2     deux     two     zwei
2          3    trois   three     drei
4          2     deux     two     zwei

```
Pour avoir les lignes dont une valeur de colonne donnée sont les N plus grandes triée pas valeur :
```python
df.nlargest(2,"numerique")
```
Ce qui donne ici :
```
   numerique français anglais allemand
3          4   quatre    four      für
2          3    trois   three     drei

```
Et est la même chose que si l'on avait fait :
```python
df.sort_values("numerique", ascending=False).iloc[:2]
```
Sur le même principe il y a `nsmallest`.

On peut créer des échantillons aléatoires, et pour cela on utilise la méthode `sample` :
```python
df.sample(3)
```

# Traiter les données manquantes
Trois possibilités :
- Les garder ;
- Les supprimer ;
- Les remplacer.
Selon la nature de la colonnes Pandas peut mettre `NaN` ("Not A Number"), `NaT` ("Not A Timesptamp") ou autre.
Le problème a laisser `NaN` ou autre et que la plus part des méthodes ne les gèrent pas.
Avec Numpy "NaN" est `np.nan` en Pandas `pd.NA`

Attention laisser des `NaN` ne permet pas les comparaisons car :
```python
np.nan == np.nan # Donne False
np.nan is np.nan # À utiliser
```
Pour savoir quelles sont les valeurs qui sont des NaN :
```python
df.isnull()
```
Qui retourne un tableau de booléens dont les valeurs sont à True lorsqu'une valeur est un NaN.
On peut aussi utiliser la fonction opposée qui est `notnull`
On peut alors filtrer les colonnes ou lignes en utilisant les sorties de `isnull` ou `notnull`.
ex pour un dataframe de films:
```python
movie_df[(movie_df['movie_score'].isnull()) & (movie_df['title'].notnull())]
```

## Suppression des données manquantes
Pour cela on utilise `dropna` qui prend en paramètre `axis`, `how`, `tresh`, `subset` :
- `how` permet de dire si l'on supprime l'axe, soit dés qu'une valeur est NaN avec "any" ou seulement si toutes les valeurs de l'axe sont NaN avec la valeur "all" ;
- `tresh` est un seuil de valeur non NaN à avoir sur l'axe pour qu'il soit supprimé ;
- `subset` permet de limiter la vérification à seulement certaines colonnes en passant une liste de noms.

```python
import pandas as pd
import numpy as np
np.random.seed(101)
my_dataframe = pd.DataFrame(
	data=np.random.rand(12,7),
	index=("janvier", "février", "mars", "avril", "mai", "juin", "juillet", "août", "septembre", "octobre", "novembre", "décembre"),
	columns=("Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi", "Samedi", "Dimanche")
)
# Avec nous génèrons un `df` ayant des NaN
df= my_dataframe.map(lambda x: np.nan if x < 0.3 else x)
df
```
Ce qui donne :

|index|Lundi|Mardi|Mercredi|Jeudi|Vendredi|Samedi|Dimanche|
|---|---|---|---|---|---|---|---|
|janvier|0\.5163986277024462|0\.5706675868681398|NaN|NaN|0\.6852769816973125|0\.8338968626360765|0\.3069662196722378|
|février|0\.8936130796833973|0\.7215438617683047|NaN|0\.5542275911247871|0\.3521319540266141|NaN|0\.7856017618643588|
|mars|0\.9654832224119693|NaN|NaN|0\.6035484222912185|0\.7289927572876178|NaN|0\.6853063287801783|
|avril|0\.5178674741970474|NaN|NaN|NaN|0\.994317901154097|0\.5206653966849669|0\.5787895354754169|
|mai|0\.7348190582693819|0\.5419617722295935|0\.9131535576757686|0\.8079201509879171|0\.40299783069378736|0\.35722434282078785|0\.9528767147108732|
|juin|0\.3436315778925598|0\.865099816318328|0\.830277712198797|0\.5381614492475574|0\.9224693725672236|NaN|NaN|
|juillet|0\.7015072956958257|0\.8904798691284314|NaN|NaN|0\.6724915296568056|NaN|0\.7013711366090863|
|août|0\.4876352222057281|0\.6806777681763588|0\.5215481923258594|NaN|NaN|0\.575205086868013|NaN|
|septembre|0\.5001167138007933|NaN|NaN|NaN|0\.4423681315127448|0\.8775873246276348|0\.949264128985194|
|octobre|0\.4781674167895694|0\.4611193422864347|0\.637289031017357|0\.324607996437537|NaN|NaN|0\.6376586528178253|
|novembre|0\.8122658949111644|0\.6702604203336356|0\.6517677034763694|0\.42456894356178443|0\.6565953361995259|NaN|0\.6599245188671674|
|décembre|0\.5296233987530145|0\.7485203698504924|NaN|0\.7845218499756547|0\.6872420375766887|0\.6950784965081013|0\.496866519596742|


## Remplacement des NaN
Pour remplacer toutes les valeurs d'un coup, on utilise la méthode `fillna`
```python
df.fillna(0)
```
!Attention! essayer de garder une homogénéité de type et une cohérence de valeur.
Car si vous faites la moyenne avec `mean`, cette fonction n'utilisera que les cellules ayant un numérique pour faire le calcul de moyenne.
```python
df["Mardi"].mean()
```
On peut alors utiliser ce principe pour remplacer les valeurs manquantes et cela pour chaque colonne :
```python
df.fillna(df.mean())
```
Qui va calculer la moyenne de chaque colonne et l'utiliser pour remplacer les NaN le colonne correspondante.

On peut aussi utiliser la méthode `interpolate`, pour remplacer le NaN par des valeurs interpolées.
```python
df.interpolate()
```
Va donner :

|index|Lundi|Mardi|Mercredi|Jeudi|Vendredi|Samedi|Dimanche|
|---|---|---|---|---|---|---|---|
|janvier|0\.5163986277024462|0\.5706675868681398|NaN|NaN|0\.6852769816973125|0\.8338968626360765|0\.3069662196722378|
|février|0\.8936130796833973|0\.7215438617683047|NaN|0\.5542275911247871|0\.3521319540266141|0\.7294863739857067|0\.7856017618643588|
|mars|0\.9654832224119693|0\.6616831652554009|NaN|0\.6035484222912185|0\.7289927572876178|0\.6250758853353368|0\.6853063287801783|
|avril|0\.5178674741970474|0\.6018224687424972|NaN|0\.7057342866395678|0\.994317901154097|0\.5206653966849669|0\.5787895354754169|
|mai|0\.7348190582693819|0\.5419617722295935|0\.9131535576757686|0\.8079201509879171|0\.40299783069378736|0\.35722434282078785|0\.9528767147108732|
|juin|0\.3436315778925598|0\.865099816318328|0\.830277712198797|0\.5381614492475574|0\.9224693725672236|0\.4298845908365296|0\.8271239256599798|
|juillet|0\.7015072956958257|0\.8904798691284314|0\.6759129522623282|0\.4847730860450523|0\.6724915296568056|0\.5025448388522713|0\.7013711366090863|
|août|0\.4876352222057281|0\.6806777681763588|0\.5215481923258594|0\.4313847228425472|0\.5574298305847751|0\.575205086868013|0\.8253176327971401|
|septembre|0\.5001167138007933|0\.5708985552313968|0\.5794186116716082|0\.3779963596400421|0\.4423681315127448|0\.8775873246276348|0\.949264128985194|
|octobre|0\.4781674167895694|0\.4611193422864347|0\.637289031017357|0\.324607996437537|0\.5494817338561353|0\.8167510485877902|0\.6376586528178253|
|novembre|0\.8122658949111644|0\.6702604203336356|0\.6517677034763694|0\.42456894356178443|0\.6565953361995259|0\.7559147725479458|0\.6599245188671674|
|décembre|0\.5296233987530145|0\.7485203698504924|0\.6517677034763694|0\.7845218499756547|0\.6872420375766887|0\.6950784965081013|0\.496866519596742|

On constate que le début de la colonne Mercredi n'a pu être interpolée puisqu'il n'y avait pas de valeur de précédente. On constate aussi que l'interpolation n'est faite qu'entre les deux valeurs encadrant les NaN, on verra par la suite comment faire des régressions afin de prendre en compte toutes les valeurs.

# GroupBy
Il permet de regrouper les lignes selon les valeurs d'une colonne appelée Catégorie et de produire un DataFrameGroupBy (un sous DataFrame "lazy") qui correspond à autant de sous DataFrame  qu'il y a de valeurs uniques dans la colonne des Catégories, puis d'appliquer sur chacun de ses sous DataFrame une fonction telle que `mean`, `sum`, `count` qui consomme le DataFrameGroupBy.
```python
df = pd.read_csv("/content/sample_data/california_housing_train.csv")
hma = df.groupby("housing_median_age")
hma.mean()
```
On peut faire un `groupeby` sur plusieurs colonnes en passant une liste à la fonction `groupby`.
```python
hma = df.groupby(["housing_median_age", "total_rooms"])
hma.index
```
L'index devient une liste de tuples entre les deux noms de colonnes.
```txt
MultiIndex([( 1.0, 2062.0),
            ( 1.0, 2254.0),
            ( 2.0,   96.0),
            ( 2.0,  227.0),
            ( 2.0,  337.0),
            ( 2.0,  556.0),
            ( 2.0,  582.0),
            ( 2.0,  647.0),
            ( 2.0,  790.0),
            ( 2.0,  838.0),
```
On peut appeler toutes les méthodes de description avec la méthode `describe`
```python
hma.head(100).describe # Limite l'exploration aux 100 premères lignes.
```

et va pour les colonnes (longitude, latitude, housing_median_age, total_rooms, total_bedrooms, population, households, median_income, median_house_value) les lignes (count, mean, std, min, 25%, 50%, 75%, max)
On peux interroger le nombre de valeurs avec `index.levels`
On peux alors interroger le `DataFrame` index par index avec la méthode `loc`.
```python
hma.mean().loc[[1, 2]] # qui retournera les lignes pour les valeurs
```

Il est possible de regrouper selon les valeurs de plusieurs colonnes ou d'indexes en passant lors de l'appel de `groupby` la liste des colonnes ou des indexes dans l'ordre de prédominance.
On parle alors d'indexes multi-niveaux.
```python
df.groupby([index1, index2])
```
Une fois obtenu l'objet GroupeBy on peut lui appliquer l'une des méthodes de calcul comme `mean, std, min, max` etc.
On peut aussi appliquer la méthode `.describe` qui calculera toutes ses méthodes et donne un "dataframe" résultat :
```python
my_dataframe = pd.DataFrame(
data=np.random.rand(4,5),
index=("janvier", "février", "mars", "avril"),
columns=("Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi")
)
my_dataframe.groupby(["Lundi","Mercredi"]).describe()
```
Qui affiche :
```
>>> print(my_dataframe.groupby(["Lundi","Mercredi"]).describe())
                  Mardi                                              ... Vendredi                                                  
                  count      mean std       min       25%       50%  ...      std       min       25%       50%       75%       max
Lundi    Mercredi                                                    ...                                                           
0.184091 0.001189   1.0  0.223771 NaN  0.223771  0.223771  0.223771  ...      NaN  0.930979  0.930979  0.930979  0.930979  0.930979
0.222381 0.799661   1.0  0.734495 NaN  0.734495  0.734495  0.734495  ...      NaN  0.302624  0.302624  0.302624  0.302624  0.302624
0.609272 0.319178   1.0  0.085001 NaN  0.085001  0.085001  0.085001  ...      NaN  0.872406  0.872406  0.872406  0.872406  0.872406
0.890708 0.915961   1.0  0.689910 NaN  0.689910  0.689910  0.689910  ...      NaN  0.162610  0.162610  0.162610  0.162610  0.162610

[4 rows x 24 columns]

```
La méthode `transpose` permet qu'en à elle d'inverser le colonnes et les lignes.

La fonction `xs` "Cross Schema" permet de récupérer un sous section en fonction du nom du niveau `level` et d'une seule valeur de clé.

Pour plusieurs valeurs de clé il faut utiliser `loc`, mais `loc` n'offre pas la même souplesse .

Pour filtrer sur plusieurs valeurs de clés, il faut générer un nouveau "DataFrame" en utilisant les fonctions de filtre comme :
```python
df[df['années'].isin([1973,1975])]
```
La fonction `agg` permet d'appliquer des méthodes d'agrégation à des colonnes d'un dataframe.
Lien vers la documentation des fonctions permettant de merger, joindre, concaténer et comparer des dataframe : https://pandas.pydata.org/docs/user_guide/merging.html
Concaténation : 
```python
import numpy as np
import pandas as pd
data_d1 = ['A':('A0','A1','A2','A3'), 'B':('B0','B1','B2','B3')]
data_d2 = ['C':('C0','C1','C2','C3'), 'D':('D0','D1','D2','D3')]
d1 = pd.DataFrame(data_d1)
d2 = pd.DataFrame(data_d2)
pd.concat([d1, d2], axis=1)
```