Le scripting bash peut vite s'apparenter à du développement, alors pourquoi ne pas utiliser un vrai langage de programmation ?

# Historique
Python a été créé fin 1986 par Guido van Rossum au CWI (Institut de recherche en mathématique et informatique de Hollande), à partir de la version 1.2 en 1995 le CNRI (Corporation of Nationnal Research Initiative) finance le projet.

En 2000 Python passe en version 2.0 au sein de Be.open, puis l'équipe rejoint Digitial Creation (Futur Zope Corporation) en 2001. En mars 2001 création de la python fondation et libération complète du code.

# La notion d'objet
L'objet est l'un des concepts centraux de la programmation orientée objet (POO).

Un objet peut être perçu comme une aggrégation d'attibuts situés à un espace unique en mémoire. L'objet est donc une entité qui encapsule des données (attributs) et des comportements (méthodes) dans une structure dont la définition est la même pour tous les objets de même type. Les objets sont des instances d'une classe, qui est un modèle ou un plan définissant les caractéristiques communes à un groupe d'objets similaires.

Un objet représente une entité spécifique dans un programme et peut interagir avec d'autres objets en utilisant leurs méthodes. Les objets reflètent souvent des concepts ou des entités du monde réel, facilitant ainsi la modélisation et la compréhension des problèmes.

En Python tout est objet.

# La notion de classe
La classe est l'un des concepts centraux de la programmation orientée objet (POO).
Une classe est une définition d'un Objet, nous pouvons rapprocher cela de la définition de Humain dans un dictionnaire. Tous les humains sur terre respectent cette définition, mais tout les humains ont des "pattributs" qui leur sont propres, comme le lieu et la date de naissance et des parents différents des autres exceptés ses frères et soeur.
Lorsque nous rencontrons un humain nous nous attendons à ce qu'il possède les attributs définie dans sa classe Humain et que ces attribut lui soit propre. Nous disont que cet humain est une instance de la classe Humain. Un sinonyme de instance est objet, un humain en particulier est un objet de type Humain.
Pour le dire de façon plus précise mais plus abstraite :
En programmation orientée objet (POO), une classe est un modèle ou un plan qui définit les attributs et les comportements communs à un groupe d'objets similaires. Les attributs sont les propriétés ou les caractéristiques d'un objet, tandis que les comportements sont les actions ou les opérations que les objets peuvent effectuer. Les classes servent de base pour créer des objets (instances) qui partagent les mêmes caractéristiques.
Un objet est une instance d'une classe, c'est-à-dire un exemple spécifique de cette classe. Chaque objet a un état et un comportement propres, déterminés par les attributs et les méthodes définis dans la classe.
Tous les objets appartiennent à une classe ou plusieurs.
L'Humain partage des attributs avec les autres animaux, on peut donc créer une classe Animal et faire dériver Humain de celle ci. Nous appèlons ce processus généralisation. Huamain ne contiendra que les attribut spécifique à l'humain, comme la langue maternelle, et les attribut comme la date de naissance ou le régime alimenatire seront remontés dans la classe Animal.
Le principe de généralisation, également connu sous le nom d'héritage, est un concept clé de la programmation orientée objet (POO). Il permet à une classe (classe dérivée ou classe enfant) d'hériter des attributs et des méthodes d'une autre classe (classe de base ou classe parent). La généralisation est utilisée pour représenter des relations de type "est-un" entre les classes, où la classe dérivée est une version spécialisée ou plus spécifique de la classe de base.

L'héritage permet d'éviter la redondance du code en réutilisant les fonctionnalités communes entre les classes, tout en permettant d'étendre et de modifier le comportement selon les besoins de la classe dérivée. Il favorise également la modularité et la lisibilité du code.

# L'instruction print
L'instruction `print` en Python est une fonction intégrée qui permet d'afficher du texte ou des valeurs dans la console ou le terminal où le programme est exécuté. La fonction `print` affiche les arguments passés sous forme de chaîne de caractères, séparés par des espaces, et termine la sortie avec un retour à la ligne, sauf indication contraire.

Voici quelques exemples d'utilisation de la fonction `print` en Python :

## Afficher une chaîne de caractères simple
```python
print("Bonjour tout le monde !")
```

## Afficher plusieurs chaînes de caractères ou valeurs séparées par des virgules
```python
nom = "Michaël"
age = 50
print("Bonjour, je m'appelle", nom, "et j'ai", age, "ans.")
```

## Utiliser les f-strings (format string literals) pour formater la sortie
```python
nom = "Michaël"
age = 50
print(f"Bonjour, je m'appelle {nom} et j'ai {age} ans.")
```

## Modifier le séparateur entre les arguments passés à la fonction `print`
```python
print("John", "Doe", sep="-")
```

## Supprimer le retour à la ligne en fin de la fonction `print` en modifiant la valeur du paramètre `end` :
```python
print("Hello, World!", end="")
```

# Les primitives d'accès aux informations de type

## id()
La fonction `id()` renvoie l'identifiant unique d'un objet en Python. Cet identifiant est un entier qui représente l'adresse mémoire de l'objet. L'identifiant d'un objet est garanti d'être unique et constant pour la durée de vie de l'objet.

Exemple d'utilisation de `id()` :

```python
a = "Bonjour"
b = a

print(id(a))  # Affiche un entier, par exemple, 140703088628496
print(id(b))  # Affiche le même entier que celui de a, car a et b font référence au même objet
```

## type()
La fonction `type()` renvoie le type d'un objet en Python. Le type d'un objet détermine la classe à laquelle il appartient et définit les opérations possibles sur cet objet, ainsi que les méthodes et les attributs qu'il possède.

Exemple d'utilisation de `type()` :

```python
a = 42
b = "Bonjour"

print(type(a))  # Affiche <class 'int'>
print(type(b))  # Affiche <class 'str'>
```

## dir()
La fonction `dir()` retourne une liste des attributs (variables et méthodes) d'un objet donné. Si aucun argument n'est fourni, la fonction renvoie la liste des noms du scope local.

Exemple d'utilisation de `dir()` :

```python
class MaClasse:
    x = 42

    def ma_methode(self):
        pass

objet = MaClasse()

print(dir(objet))
# Affiche une liste contenant les attributs et méthodes de l'objet, par exemple :
# ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__',
#  '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__',
#  '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'ma_methode', 'x']
```

Dans cet exemple, `dir()` renvoie la liste des attributs et des méthodes de l'objet de type `MaClasse`, y compris les méthodes spéciales et les méthodes héritées de la classe parente (ici, `object`).

## help()
La fonction `help()` permet d'afficher des informations d'aide sur les objets, tels que les modules, les classes, les fonctions, les méthodes et les types de données. Elle permet d'obtenir les informations sur un objet ou une fonction dans shell python, c'est à dire pendant l'exécution.

Lorsque nous appelons `help()` avec un objet en argument, la fonction affiche une description détaillée de l'objet, y compris sa signature (pour les fonctions et les méthodes), sa documentation (docstring), la liste de ses attributs et méthodes (pour les classes) et d'autres informations pertinentes.

Quelques exemples d'utilisation :

### Obtenir de l'aide sur un module
```python
import math
help(math)
```

### Obtenir de l'aide sur une classe
```python
help(list)
```

### Obtenir de l'aide sur une fonction ou une méthode
```python
help(print)
```

### Obtenir de l'aide sur un type de données
```python
help(str)
```

### Obtenir de l'aide sur Python
Si nous appelons  la fonction `help()` sans argument, cela lance une session interactive d'aide dans la console et la saisie d'un nom de variable permet d'obtenir les informations sur l'objet référencé.

Exemple :

```
help()

Welcome to Python help utility!

help> print
Help on built-in function print in module builtins:

print(...)
    print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)

    Prints the values to a stream, or to sys.stdout by default.
    Optional keyword arguments:
    file:  a file-like object (stream); defaults to the current sys.stdout.
    sep:   string inserted between values, default a space.
    end:   string appended after the last value, default a newline.
    flush: whether to forcibly flush the stream.
```

# Les commentaires
Les commentaires sont des portions de texte ajoutées au code source d'un programme pour expliquer ou clarifier le code. Ils sont destinés à être lus par les développeurs et sont ignorés par l'interpréteur Python lors de l'exécution du code. Les commentaires sont utiles pour améliorer la lisibilité et la maintenabilité du code en fournissant des informations supplémentaires ou en expliquant les décisions de conception.

En Python, il existe deux types de commentaires : les commentaires sur une seule ligne et les commentaires multilignes.

## Commentaires sur une seule ligne
Le caractère dièse (#) commence les commentaires sur une seule ligne. Tout texte qui suit le dièse jusqu'à la fin de la ligne est considéré comme un commentaire et sera ignoré par l'interpréteur Python.

Exemple :

```python
# Ceci est un commentaire sur une seule ligne
print("Bonjour le monde !")  # Nous pouvons ajouter un commentaire en fin de ligne
```

## Commentaires multilignes
Les commentaires multilignes sont écrits en utilisant des chaînes de caractères multilignes avec des triples guillemets ("""...""" ou '''...'''). Même si ce ne sont pas des commentaires à proprement parler, ils sont souvent utilisés à cette fin, car l'interpréteur Python ignore les chaînes de caractères non affectées à des variables.

Exemple :

```python
"""
Ceci est un commentaire
multiligne en Python
"""

print("Bonjour le monde !")
```

Les chaînes de caractères multilignes sont également utilisées comme docstrings pour documenter les modules, les classes et les fonctions en Python. Dans ce cas, elles ne sont pas ignorées par l'interpréteur et peuvent être consultées à l'aide de la fonction `help()` ou de l'attribut `__doc__` de l'objet correspondant.
```python
def f(p:int=0)->int:
    """A documented function which computes square of p
    Args:
        p (int, optional): The parameter. Defaults to 0.

    Returns:
        int: The square of p
    >>> f(2)
        4
    """


help(f)
Help on function f in module __main__:

f(p: int = 0) -> int
    A documented function which computes square of p
    Args:
        p (int, optional): The parameter. Defaults to 0.
    
    Returns:
        int: The square of p
    >>> f(2)
        4

```

# Les chaînes de caractères
Les chaînes de caractères en Python sont des séquences de caractères Unicode. En Python 3, l'encodage par défaut est UTF-8, qui est une représentation des caractères Unicode.

## Encodage et Décodage
**Encodage** : Le passage de caractères Unicode à une représentation binaire (bytes) s'appelle l'encodage. En Python, nous utilisons la méthode `encode()` pour cela. Par exemple, pour encoder une chaîne de caractères en utilisant l'encodage 'ISO-8859-15', nous faisons :

```python
chaine = "C'est la fête"
chaine_encodee = chaine.encode('ISO-8859-15')
```

**Décodage** : Le passage d'une représentation binaire à des caractères Unicode s'appelle le décodage. En Python, nous utilisons la méthode `decode()` pour cela. Par exemple, pour décoder une chaîne de caractères encodée en 'ISO-8859-15', nous faisons :

```python
chaine_decodee = chaine_encodee.decode('ISO-8859-15')
```

## Types de chaînes

Python fournit plusieurs manières de délimiter les chaînes de caractères :

**Guillemets** : Nous pouvons utiliser des guillemets pour délimiter une chaîne de caractères. Si la chaîne est sur plusieurs lignes, nous pouvons utiliser trois guillemets.

```python
chaine1 = "Bonjour le monde"
chaine2 = """Bonjour
le monde"""
```

 **Apostrophes** : Nous pouvons également utiliser des apostrophes pour délimiter une chaîne de caractères.

```python
chaine1 = 'Bonjour le monde'
```

**Chaînes raw** : Les chaînes raw sont précédées de `r` ou `R` et sont utilisées pour représenter les chaînes telles quelles, sans traiter les échappements. Elles sont souvent utilisées pour les expressions régulières, les chemins de fichiers, etc.

```python
chaine1 = r"C:\nouveau\chemin"
```

**Chaînes formatables** : Les chaînes formatables (ou f-strings) sont une nouvelle fonctionnalité de Python 3.6 qui permet d'insérer des expressions à l'intérieur des chaînes de caractères en utilisant des accolades `{}`.

```python
nom = "Monde"
chaine1 = f"Bonjour {nom}"
```

Dans cet exemple, `{nom}` est remplacé par la valeur de la variable `nom` lors de l'exécution du programme. Les f-strings rendent le formatage des chaînes de caractères plus facile et plus intuitif.

## Manipulation des chaînes de caractères

En plus de créer des chaînes de caractères, Python fournit un ensemble de méthodes permettant de les manipuler :

**Concaténation** : Nous pouvons concaténer deux chaînes de caractères en utilisant l'opérateur `+`.

```python
chaine1 = "Bonjour"
chaine2 = "le monde"
chaine3 = chaine1 + " " + chaine2  # Donne "Bonjour le monde"
```

**Répétition** : Nous pouvons répéter une chaîne de caractères un certain nombre de fois en utilisant l'opérateur `*`.

```python
chaine = "Bonjour " * 3  # Donne "Bonjour Bonjour Bonjour "
```

**Accès aux caractères** : Nous pouvons accéder à un caractère particulier d'une chaîne de caractères en utilisant son indice entre crochets `[]`.

```python
chaine = "Bonjour"
premier_caractere = chaine[0]  # Donne "B"
```

**Longueur** : Nous pouvons obtenir la longueur d'une chaîne de caractères en utilisant la fonction `len()`.

```python
chaine = "Bonjour"
longueur = len(chaine)  # Donne 7
```

**Méthodes de chaîne** : Python fournit de nombreuses méthodes pour manipuler les chaînes de caractères, comme `upper()`, `lower()`, `strip()`, `split()`, `replace()`, etc.

```python
chaine = "Bonjour le monde"
chaine_majuscules = chaine.upper()  # Donne "BONJOUR LE MONDE"
chaine_minuscules = chaine.lower()  # Donne "bonjour le monde"
```

## Chaînes de caractères et immutabilité
Les chaînes de caractères en Python sont immuables. Cela signifie que nous ne pouvons pas changer un caractère d'une chaîne de caractères une fois qu'elle a été créée. Si nous essayons de le faire, Python nous donnera une erreur. Si nous devons modifier une chaîne de caractères, nous devons créer une nouvelle chaîne de caractères avec les modifications.

```python
chaine = "Bonjour"
chaine[0] = "b"  # Donne une erreur
```

À la place, nous pouvons créer une nouvelle chaîne de caractères avec la modification :

```python
chaine = "Bonjour"
nouvelle_chaine = "b" + chaine[1:]  # Donne "bonjour"
```

## Le sclicing de chaînes de caractères
Le slicing (ou découpage) est une fonctionnalité qui permet d'extraire une partie d'une séquence, comme une chaîne de caractères, une liste, un tuple, etc. Pour les chaînes de caractères, le slicing peut être utilisé pour extraire des sous-chaînes.

La syntaxe du slicing est la suivante :

```python
sous_chaine = chaine[debut:fin]
```

`debut` est l'indice où commence le slicing et `fin` est l'indice où il se termine. L'indice de début est inclusif et l'indice de fin est exclusif. Cela signifie que le slicing commence à l'indice `debut` et se termine juste avant l'indice `fin`.

Voici un exemple :

```python
chaine = "Bonjour le monde"
sous_chaine = chaine[0:7]
print(sous_chaine)  # Affiche "Bonjour"
```

Dans cet exemple, le slicing commence à l'indice 0 et se termine à l'indice 7, donc il extrait les caractères de l'indice 0 à 6.

Si nous omettons l'indice de début, le slicing commence au début de la chaîne. Si nous omettons l'indice de fin, le slicing va jusqu'à la fin de la chaîne.

```python
chaine = "Bonjour le monde"
sous_chaine1 = chaine[:7]
sous_chaine2 = chaine[8:]
print(sous_chaine1)  # Affiche "Bonjour"
print(sous_chaine2)  # Affiche "le monde"
```

Nous pouvons également utiliser des indices négatifs pour commencer le slicing depuis la fin de la chaîne.

```python
chaine = "Bonjour le monde"
sous_chaine = chaine[-5:-1]
print(sous_chaine)  # Affiche "mond"
```

Enfin, nous pouvons également spécifier un pas de slicing, qui est le nombre d'indices à sauter après chaque caractère.

```python
chaine = "Bonjour le monde"
sous_chaine = chaine[::2]
print(sous_chaine)  # Affiche "Bno u od"
```

Dans cet exemple, le pas de slicing est 2, donc le slicing extrait tous les deuxièmes caractères de la chaîne.

# Les numériques
Python fournit plusieurs types numériques que nous pouvons utiliser pour représenter les nombres dans nos programmes.

## Entiers
Les entiers en Python sont de type `int`. En Python 3, il n'y a pas de limite à la taille des entiers, ils peuvent être aussi grands que ce que peut contenir la mémoire de notre ordinateur.

```python
entier = 123
print(type(entier))  # Affiche "<class 'int'>"
```

Pour les entiers longs en Python 2, nous suffixions par un "L" ou "l", mais ce type n'existe plus en Python 3, où tous les entiers sont de type `int`, peu importe leur taille.

La représentation des entiers peut être en base 10, en octale (base 8, préfixée par `0o` ou `0O`), en hexadécimal (base 16, préfixée par `0x` ou `0X`) ou en binaire (base 2, préfixée par `0b` ou `0B`).
```python
octal = 0o123  # Entier en base 8
hexa = 0x123  # Entier en base 16
binaire = 0b1010  # Entier en base 2
```

#### Booléens
Le type `bool` est un sous-type d'entier qui ne peut prendre que deux valeurs : `True` (vrai) et `False` (faux).
```python
vrai = True
faux = False
print(type(vrai))  # Affiche "<class 'bool'>"
```

## Nombres à virgule flottante
Les nombres à virgule flottante en Python sont de type `float`. Ils peuvent représenter des nombres réels, positifs ou négatifs, avec ou sans partie fractionnaire.
```python
flottant = 0.123
grand_flottant = 1e100
print(type(flottant))  # Affiche "<class 'float'>"
```
## Nombres complexes
Les nombres complexes sont de type `complex`. Ils sont représentés par deux nombres à virgule flottante, le premier représentant la partie réelle et le second représentant la partie imaginaire.
```python
complexe = 1 + 3j
print(type(complexe))  # Affiche "<class 'complex'>"
```
# Les types à valeur unique
En Python, il existe deux types spéciaux qui ont une seule valeur possible : `None` et `NotImplemented`.

## None
`None` est un type spécial en Python qui représente l'absence de valeur ou la nullité. Il est souvent utilisé pour signifier qu'une fonction ne retourne rien, ou qu'une variable n'a pas encore été assignée à une valeur. En termes de programmation, `None` est un singleton, ce qui signifie qu'il n'y a qu'une seule instance de `None` en Python.

Voici un exemple d'utilisation de `None` :

```python
def une_fonction():
    print("Je ne retourne rien.")

resultat = une_fonction()
print(resultat)  # Affiche "None"
```

Dans cet exemple, `une_fonction()` ne retourne rien, donc son résultat est `None`.

## NotImplemented
`NotImplemented` est une autre valeur unique en Python. Elle est généralement utilisée pour indiquer que la méthode ou l'opération spéciale pour un type donné n'est pas implémentée.

Par exemple, si vous créez une classe personnalisée et que vous voulez qu'elle puisse utiliser l'opérateur `+`, vous pouvez définir une méthode spéciale appelée `__add__`. Si vous ne voulez pas que l'opérateur `+` soit utilisé pour votre classe, vous pouvez faire que `__add__` renvoie `NotImplemented`.

```python
class MaClasse:
    def __add__(self, autre):
        return NotImplemented

objet = MaClasse()
resultat = objet + 5  # Donne une erreur
```

Dans cet exemple, essayer d'utiliser l'opérateur `+` avec une instance de `MaClasse` donne une erreur, car `__add__` renvoie `NotImplemented`.

Il est à noter que `NotImplemented` est différent de `NotImplementedError`. Le premier est une valeur spéciale qui peut être renvoyée par certaines méthodes spéciales, tandis que le second est une exception qui est levée lorsqu'une méthode abstraite requise n'est pas implémentée par une classe.

# Les séquences
En Python, une séquence est une collection ordonnée d'éléments. Les éléments d'une séquence sont indexés par des nombres entiers, commençant par zéro pour le premier élément. Python fournit plusieurs types de séquences, y compris les tuples, les listes et les dictionnaires. 

## Les tuples
Un tuple est une séquence immuable d'éléments. Cela signifie que nous ne pouvons pas modifier un tuple une fois qu'il a été créé. Les tuples sont généralement utilisés pour regrouper des données qui sont liées de manière logique.

```python
mon_tuple = (1, 2, 3)
print(type(mon_tuple))  # Affiche "<class 'tuple'>"
```

Dans cet exemple, `mon_tuple` est un tuple de trois éléments. Nous pouvons accéder à un élément du tuple en utilisant son indice, par exemple `mon_tuple[0]` retourne `1`.

## Les listes
Une liste est une séquence mutable d'éléments. Cela signifie que nous pouvons ajouter, modifier ou supprimer des éléments d'une liste après sa création. Les listes sont très polyvalentes et sont souvent utilisées pour stocker des collections d'éléments qui doivent être modifiées.

```python
ma_liste = [1, 2, 3]
print(type(ma_liste))  # Affiche "<class 'list'>"
```

Dans cet exemple, `ma_liste` est une liste de trois éléments. Nous pouvez ajouter un élément à la liste avec `ma_liste.append(4)`, modifier un élément avec `ma_liste[0] = 0`, ou supprimer un élément avec `del ma_liste[2]`.

## Les dictionnaires
Un dictionnaire est une collection non ordonnée de paires clé-valeur. Il est mutable, ce qui signifie que nous pouvons ajouter, modifier ou supprimer des paires clé-valeur après sa création. Les dictionnaires sont souvent utilisés pour stocker des données qui sont associées à des clés uniques.

```python
mon_dict = {'a': 1, 'b': 2, 'c': 3}
print(type(mon_dict))  # Affiche "<class 'dict'>"
```

Dans cet exemple, `mon_dict` est un dictionnaire de trois paires clé-valeur. Nous pouvons accéder à une valeur du dictionnaire en utilisant sa clé, par exemple `mon_dict['a']` retourne `1`. Nous pouvons ajouter une paire clé-valeur avec `mon_dict['d'] = 4`, modifier une valeur avec `mon_dict['a'] = 0`, ou supprimer une paire clé-valeur avec `del mon_dict['b']`.

# Les ensembles
@todo

## Documentation
https://docs.python.org/fr/3/library/stdtypes.html

# Les listes en compréhension
Les listes en compréhension (ou list comprehension en anglais) permettent de créer de nouvelles listes de manière très concise. Elles suivent une syntaxe qui peut être comprise comme une expression de la forme "établir une liste de cet élément pour cet élément dans cet ensemble si cette condition est vraie".

La syntaxe de base est la suivante :

```python
nouvelle_liste = [expression for élément in iterable]
```

Par exemple, si nous voulons créer une liste des carrés des nombres de 0 à 9, nous pourrions le faire avec une boucle `for` traditionnelle :

```python
carrés = []
for x in range(10):
    carrés.append(x**2)
```

Ou, nous pouvons utiliser une liste en compréhension pour le faire en une seule ligne :

```python
carrés = [x**2 for x in range(10)]
```

Les listes en compréhension peuvent également inclure une condition `if` pour filtrer les éléments de l'itérable. Par exemple, si nous voulons une liste des carrés des nombres pairs de 0 à 9, nous pouvons le faire comme suit :

```python
carrés_pairs = [x**2 for x in range(10) if x % 2 == 0]
```

Dans cet exemple, `x % 2 == 0` est une condition qui n'est vraie que pour les nombres pairs, donc seuls les carrés des nombres pairs sont inclus dans la liste.

## Emboitement des listes en compréhension
L'emboîtement de listes en compréhension (ou nested list comprehension en anglais) signifie qu'une liste en compréhension est utilisée à l'intérieur d'une autre liste en compréhension. C'est une façon puissante et concise de créer des listes.

Par exemple, supposons que nous ayons une liste de listes, et que nous voulions aplatir cette structure en une seule liste. Voici comment nous pourrions le faire avec une liste en compréhension imbriquée :

```python
listes = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
aplati = [x for sous_liste in listes for x in sous_liste]
print(aplati)  # Affiche [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

Notons l'ordre des boucles `for` dans la liste en compréhension : elles sont dans le même ordre que si nous avions écrit des boucles `for` traditionnelles :

```python
aplati = []
for sous_liste in listes:
    for x in sous_liste:
        aplati.append(x)
```

Nous pouvons également utiliser des listes en compréhension imbriquées pour créer des listes de listes. Par exemple, voici comment nous pourrions créer une matrice 3x3 avec des listes en compréhension imbriquées :

```python
matrice = [[i * j for j in range(3)] for i in range(3)]
print(matrice)  # Affiche [[0, 0, 0], [0, 1, 2], [0, 2, 4]]
```

Dans cet exemple, la liste en compréhension externe crée les lignes de la matrice, et la liste en compréhension interne crée les éléments de chaque ligne.

# Les itérateurs
Un itérateur en Python est un objet qui peut être parcouru, c'est-à-dire qu'il permet de parcourir tous les éléments d'une collection (comme une liste, un tuple, un dictionnaire, un ensemble, etc.), un à la fois. Cette fonctionnalité est particulièrement utile dans les boucles `for` et `while`, où nous voulons effectuer une opération sur chaque élément d'une collection.

En Python, un objet est considéré comme un itérateur s'il implémente deux méthodes spéciales, `__iter__()` et `__next__()`. La méthode `__iter__()` renvoie l'itérateur lui-même, et la méthode `__next__()` renvoie le prochain élément de la collection. Lorsque tous les éléments ont été parcourus, `__next__()` doit lever l'exception `StopIteration` pour signaler la fin de l'itération.

Voici un exemple simple d'un itérateur qui parcourt les éléments d'une liste :

```python
class MonIterateur:
    def __init__(self, données):
        self.données = données
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.données):
            raise StopIteration
        résultat = self.données[self.index]
        self.index += 1
        return résultat

itérateur = MonIterateur(['pomme', 'banane', 'cerise'])
for fruit in itérateur:
    print(fruit)  # Affiche "pomme", "banane", "cerise"
```

Dans cet exemple, la boucle `for` appelle automatiquement la méthode `__iter__()` pour obtenir l'itérateur, puis appelle `__next__()` à chaque itération pour obtenir le prochain fruit. Lorsque `__next__()` lève `StopIteration`, la boucle `for` se termine.

La plupart des types de collections intégrés dans Python, comme les listes, les tuples, les dictionnaires et les ensembles, sont itérables par défaut, ce qui signifie qu'ils retournent un itérateur lorsque vous les passez à la fonction `iter()`. Par exemple, si nous avons une liste de fruits, nous pouvons obtenir un itérateur pour cette liste comme suit :

```python
fruits = ['pomme', 'banane', 'cerise']
itérateur = iter(fruits)
```

Nous pouvons alors utiliser l'itérateur pour parcourir les fruits un à la fois en appelant la fonction `next()` :

```python
print(next(itérateur))  # Affiche "pomme"
print(next(itérateur))  # Affiche "banane"
print(next(itérateur))  # Affiche "cerise"
print(next(itérateur))  # Lève StopIteration
```

Comme nous pouvons le voir, une fois que nous avons atteint la fin de l'itérateur, appeler `next()` de nouveau lève l'exception `StopIteration`.

## Les itérateurs dans la pratique

Dans la pratique, nous n'avons très souvent pas besoin de créer nos propres itérateurs, car Python fournit des itérateurs pour la plupart de ses types de collections intégrés. De plus, les boucles `for` de Python utilisent automatiquement des itérateurs "sous le capot", donc nous bénéficions de la puissance des itérateurs chaque fois que nous utilisons une boucle `for`.

Cependant, il y a des moments où la création de nos propres itérateurs peut être utile. Par exemple, si nous créons un nouveau type de collection, nous voudrons peut-être fournir un itérateur pour que les utilisateurs de notre collection puissent la parcourir facilement. Ou, si nous avons une collection très grande ou potentiellement infinie (comme une série de nombres générés à la volée), un itérateur peut être un moyen efficace de parcourir la collection sans avoir à la stocker entièrement en mémoire.

## Les générateurs sont des itérateurs

Un type particulier d'itérateur en Python est le générateur. Un générateur est une fonction spéciale qui produit une séquence de résultats au lieu de renvoyer une seule valeur. Lorsqu'un générateur est appelé, il renvoie un itérateur que nous pouvons parcourir pour obtenir une série de résultats.

Nous allons les détailler dans le chapitre suivant.

# Les générateurs
Les générateurs sont un type d'itérateur, c'est-à-dire un objet qui produit une séquence de résultats au lieu de calculer une série de valeurs et de les stocker toutes à la fois. La principale caractéristique des générateurs est qu'ils sont paresseux (lazy) : ils ne calculent la valeur suivante de la séquence que lorsqu'elle est demandée.

En Python, Nous pouvons créer des générateurs de deux façons : en utilisant des fonctions et le mot-clé `yield`, ou en utilisant des expressions génératrices.

## Fonctions de générateur
Une fonction de générateur ressemble à une fonction normale, mais au lieu d'utiliser `return` pour renvoyer une valeur, elle utilise `yield`. Lorsqu'un générateur est appelé, il renvoie un objet itérateur, mais ne commence pas à exécuter le corps de la fonction. Lorsque nous appelons la méthode `next()` (ou la fonction `next()`) sur l'objet itérateur, la fonction est exécutée jusqu'à ce qu'elle rencontre `yield`, qui renvoie la valeur courante.

Voici un exemple d'une fonction de générateur qui produit une séquence de nombres :

```python
def generateur_de_nombres(n):
    i = 0
    while i < n:
        yield i
        i += 1

g = generateur_de_nombres(5)
for num in g:
    print(num)  # Affiche les nombres de 0 à 4
```

## Expressions génératrices
Les expressions génératrices sont similaires aux compréhensions de liste, mais au lieu de produire une liste, elles produisent un générateur. Les expressions génératrices sont écrites entre parenthèses (`()`) au lieu de crochets (`[]`).

Voici un exemple d'une expression génératrice qui produit une séquence de carrés :

```python
g = (x**2 for x in range(5))
for num in g:
    print(num)  # Affiche les carrés de 0 à 4
```

En conclusion, les générateurs sont un outil puissant en Python qui vous permet de créer des itérateurs paresseux. Ils sont particulièrement utiles lorsque vous travaillez avec de grandes séquences de données qui ne tiendraient pas toutes en mémoire à la fois, ou lorsque le coût de calcul de chaque valeur de la séquence est élevé et que vous voulez le différer jusqu'à ce qu'il soit nécessaire.

## Réutilisation des générateurs

Une particularité des générateurs est qu'une fois qu'ils sont consommés, ils ne peuvent pas être réinitialisés ou réutilisés. Par exemple, si nous essayons d'exécuter à nouveau la boucle `for` dans l'un des exemples précédents, rien ne se passera parce que le générateur a déjà été épuisé. Pour réutiliser le générateur, nous devrons le réinitialiser en le réaffectant.

```python
g = generateur_de_nombres(5)
for num in g:
    print(num)  # Affiche les nombres de 0 à 4

for num in g:
    print(num)  # Ne fait rien, le générateur est déjà épuisé

g = generateur_de_nombres(5)  # Réinitialise le générateur
for num in g:
    print(num)  # Affiche les nombres de 0 à 4
```

### Envoi de valeurs et d'exceptions aux générateurs
Les générateurs sont non seulement capables de produire des valeurs, mais ils peuvent également recevoir des valeurs et des exceptions, ce qui permet une communication bidirectionnelle entre le générateur et le code qui l'appelle. Pour cela, les générateurs disposent des méthodes `send()` et `throw()`.

La méthode `send()` reprend l'exécution du générateur et "envoie" une valeur qui devient le résultat de l'expression `yield` actuelle. La méthode `throw()` est utilisée pour lever une exception dans le générateur à l'endroit où le `yield` a été interrompu la dernière fois.

```python
def generateur_bidirectionnel():
    while True:
        received = yield
        print('Reçu :', received)

g = generateur_bidirectionnel()
next(g)  # Avance jusqu'au premier yield
g.send('Bonjour')  # Affiche "Reçu : Bonjour"
g.throw(RuntimeError, 'Quelque chose ne va pas')  # Lève une RuntimeError dans le générateur
```

## Outils d'itération
Il existe plusieurs bibliothèques qui fournissent des outils utiles pour travailler avec des itérateurs. Voici quelques-unes des plus couramment utilisées :

1. **itertools** : Ce module de la bibliothèque standard Python fournit une collection d'outils pour manipuler les itérateurs. Par exemple, `itertools.count` crée un itérateur qui produit une séquence infinie de nombres entiers, `itertools.cycle` crée un itérateur qui répète indéfiniment une séquence, et `itertools.chain` combine plusieurs itérateurs en un seul. Le module `itertools` offre également plusieurs autres fonctions utiles, comme `itertools.permutations`, `itertools.combinations`, et `itertools.product`, qui génèrent respectivement toutes les permutations, combinaisons, et le produit cartésien d'une collection d'éléments.

2. **functools** : Ce module de la bibliothèque standard Python fournit des outils de haut niveau pour travailler avec des fonctions et des itérables. Par exemple, `functools.reduce` applique une fonction binaire (c'est-à-dire une fonction à deux arguments) de manière répétée à tous les éléments d'un itérable pour produire une seule valeur.

3. **numpy** : Bien que ce ne soit pas strictement une bibliothèque d'itérateurs, numpy est une bibliothèque essentielle pour le calcul scientifique en Python, et elle fournit plusieurs outils pour travailler avec des tableaux multidimensionnels (qui sont des sortes d'itérateurs). Par exemple, vous pouvez utiliser `numpy.nditer` pour parcourir tous les éléments d'un tableau numpy dans un ordre spécifique.

4. **pandas** : Cette bibliothèque, qui est largement utilisée pour l'analyse de données en Python, fournit des structures de données comme les DataFrames et les Series qui sont essentiellement des itérateurs sur des données tabulaires. Nous pouvons parcourir les lignes d'un DataFrame ou les éléments d'une Series comme nous le ferions avec un itérateur, et pandas fournit également plusieurs méthodes pour appliquer des fonctions à ces structures de données de manière efficace.

Chacune de ces bibliothèques fournit une documentation complète avec des exemples de code, donc si nous voulons en savoir plus sur l'une d'entre elles, allons consulter la documentation officielle sur .

# Les opérateurs

L'affectation =

La division / (5.0/6 =
0.83333\....)

La division entière //

Le modulo % ou divmod(5,3) = (1,2)

La négation -

L'inversion bit à bit \~ (complément à un)

La puissance \*\*

Appartenance in

### Opérateurs binaires

Le et **&**

Le ou **\|**

Le ou exclusif **\^**

### Opérateurs de comparaison

Inférieur **\<**

Supérieur **\>**

Inférieur égal **\<=**

Supérieur égal **\>=**

égalité **==** différence **!=** ou **\<\>**

le est **is**

le n'est pas **is not**

### Ordre de traitement des opérations

Parenthèses, Exposants, Division, Multiplication, Addition, Soustraction

### L'instruction pass

### L'indentation

La création de block de code se fait par indentation.

### Les structures conditionnelles

L'instruction if (else et elif)

L'instruction for in

L'instruction while

### Les fonctions

La définition des fonctions se fait à l'aide de l'instruction « def ».

La fonction est un objet.

Le code doit être indenté.

Les paramètres ne sont pas typés.

Les paramètres peuvent recevoir une valeur par défaut *p1 = 0*.

Les paramètres non explicites (ex: def f(\*\*dict)) sont placés dans un dictionnaire.

Les paramètres arbitraires (ex : def f(\*pars))sont placé dans un tuple.

Combinaison paramètres implicites et arbitraires (ex: def f(\*pars, \*\*dict)).

La directive return

La directive lambda

### Les docstrings

Les fonctions du code peuvent être documentées, ce qui permet lors de l'exécution d'un code d'interroger une fonction pour savoir comment elle fonctionne.

### Les Classes

Les classes regroupent à la fois des données et des fonctions travaillant sur ces données.

Elles sont définies par l'instruction class

Les classes peuvent hériter d'autres classes.

Exemple :

    class Animal(object):
      def __init__(self):
        self.age = 0
        self.poids = 0

    class Chat(Animal):
       def __init__(self, nom):
         super(Chat, self).__init__() # Permet de construire la partie Animal
         self.nom = nom

La notion de constructeur **\_\_init\_\_**

La notion de destructeur **\_\_del\_\_**

Les attributs privés

# Les décorateurs
Les décorateurs permettent de modifier ou d'améliorer le comportement des fonctions ou des classes sans changer leur code source. Les décorateurs sont eux-mêmes des fonctions qui prennent une fonction ou une classe comme argument et retournent une nouvelle fonction ou classe avec des comportements supplémentaires.

Exemple simple d'un décorateur :

```python
def mon_decorateur(fonction):
    def fonction_enveloppe():
        print("Code avant l'appel de la fonction.")
        fonction()
        print("Code après l'appel de la fonction.")
    return fonction_enveloppe

@mon_decorateur
def dis_bonjour():
    print("Bonjour !")

dis_bonjour()
```

Dans cet exemple, `mon_decorateur` est un décorateur qui prend une fonction en argument (`fonction`) et retourne une nouvelle fonction (`fonction_enveloppe`). La nouvelle fonction exécute du code avant et après l'appel de la fonction originale. Le symbole `@` est utilisé pour appliquer le décorateur à la fonction `dis_bonjour`. Lorsque vous appelez `dis_bonjour()`, la fonction modifiée par le décorateur est exécutée.

L'exemple précédent affiche :

```
Code avant l'appel de la fonction.
Bonjour !
Code après l'appel de la fonction.
```

Comme nous pouvons le voir, l'utilisation de décorateurs permet de réutiliser facilement des portions de code et d'ajouter des comportements à des fonctions ou des classes de manière élégante et lisible.

Un décorateur peut également prendre des arguments, ce qui le rend plus flexible. Pour cela, nous devons définir une autre fonction "d'enveloppe" autour de notre décorateur. Exemple :

```python
def mon_decorateur_avec_args(arg1, arg2):
    def le_decorateur(fonction):
        def fonction_enveloppe():
            print(f"Code avant l'appel de la fonction, avec les arguments {arg1} et {arg2}.")
            fonction()
            print("Code après l'appel de la fonction.")
        return fonction_enveloppe
    return le_decorateur

@mon_decorateur_avec_args("Hello", "World")
def dis_bonjour():
    print("Bonjour !")

dis_bonjour()
```

Dans cet exemple, `mon_decorateur_avec_args` est un décorateur qui prend des arguments. Ces arguments sont utilisés dans la fonction `fonction_enveloppe` définie à l'intérieur du décorateur.

Enfin, il est important de mentionner que les décorateurs peuvent aussi être utilisés avec des classes. Les décorateurs de classes fonctionnent de la même manière que les décorateurs de fonctions, mais ils prennent une classe comme argument et retournent une nouvelle classe avec des comportements modifiés ou ajoutés.

## Wrap
`functools.wraps` est une fonction de la bibliothèque standard Python utilisée dans le contexte des décorateurs. Elle est utilisée pour conserver les métadonnées de la fonction originale lors de la décoration.

Quand nous créons un décorateur, nous créons une fonction imbriquée (ou "wrapper") qui est appelée à la place de la fonction originale. Cependant, cette fonction imbriquée n'a pas les mêmes métadonnées que la fonction originale (comme le nom de la fonction, la docstring, les annotations, etc.). Cela peut causer des problèmes si nous utilisons les métadonnées des fonctions dans notre code (par exemple liste de fonction, design pattern fabric).

La fonction `functools.wraps` est utilisée pour copier les métadonnées de la fonction originale vers la fonction imbriquée. Exemple :

```python
from functools import wraps

def mon_decorateur(fonction):
    @wraps(fonction)
    def fonction_enveloppe(*args, **kwargs):
        print("Code avant l'appel de la fonction.")
        resultat = fonction(*args, **kwargs)
        print("Code après l'appel de la fonction.")
        return resultat
    return fonction_enveloppe

@mon_decorateur
def dis_bonjour():
    """Cette fonction affiche Bonjour !"""
    print("Bonjour !")
```

Dans cet exemple, `@wraps(fonction)` est utilisé pour indiquer que la fonction `fonction_enveloppe` est un substitut de `fonction`. Cela copie les métadonnées de `fonction` vers `fonction_enveloppe`.

Maintenant, si nous essayons d'accéder aux métadonnées de `dis_bonjour`, nous obtiendrons les métadonnées de la fonction originale et non celles de la fonction enveloppe :

```python
print(dis_bonjour.__name__)  # Affiche "dis_bonjour"
print(dis_bonjour.__doc__)   # Affiche "Cette fonction affiche Bonjour !"
```

Sans l'utilisation de `functools.wraps`, les métadonnées auraient été celles de la fonction `fonction_enveloppe`, ce qui n'est pas le comportement souhaité.

# Décorateur de fonction pour mesurer la performance
Nous pouvons facilement créer un décorateur en Python pour mesurer le temps d'exécution d'une fonction ou d'une méthode et le logger. Voici un exemple de décorateur qui utilise le module `logging` pour enregistrer le temps d'exécution :

```python
import time
import logging
import functools

# Créons un logger avec le niveau d'enregistrement défini sur INFO
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def timer_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Enregistre le temps de début
        start_time = time.perf_counter()

        # Exécute la fonction
        result = func(*args, **kwargs)

        # Enregistre le temps de fin
        end_time = time.perf_counter()

        # Calcule le temps d'exécution et le log
        execution_time = end_time - start_time
        logger.info(f"La fonction {func.__name__} a pris {execution_time:.4f} secondes pour s'exécuter.")
        
        return result
    return wrapper

# Utilisons le décorateur sur une fonction ou une méthode
@timer_decorator
def my_function():
    # Notre code ici...
    pass
```

Dans cet exemple, `timer_decorator` est un décorateur qui enveloppe une fonction avec un code supplémentaire pour enregistrer le temps de début et de fin, calculer le temps d'exécution et enregistrer ce temps dans le log.

Lorsque nous appliquons ce décorateur à une fonction ou à une méthode avec la syntaxe `@timer_decorator`, chaque appel à cette fonction ou méthode sera automatiquement chronométré et le temps d'exécution sera enregistré dans le log.

Notons que le décorateur utilise `functools.wraps` pour préserver les métadonnées de la fonction originale, comme son nom et sa docstring.

### Les exceptions

Les erreurs sont signalées par le mécanisme des exceptions :

    >>> try:
    ...    PasDefinie = None
    ... except NameError:
    ...    print("Variable non définie")
    ...
    Variable non définie

### Les modules

Les bibliothèques de programmation en python s'appellent des modules.

Primitive import

Primitive reload

### Les paquets

Les modules sont généralement découpés en paquets qui se traduisent par des dossiers sur le disque.

### Les principaux modules

------------------------------------------------------------------------

Contient les primitives

#### sys

Contient les informations relative à l'exécution en cours

#### os

Permet de gérer le système

#### gzip, zipfile

Permettent de gérer les fichiers compressés.

#### socket

Pour gérer les connexions TCP ou UDP

#### urllib2

Pour gérer les connexions http

Liens :

> -   <http://diveintopython.adrahon.org/>
> -   <http://docs.python.org/>
> -   <http://www.afpy.org/>

#### Environnement virtuel

Les environnements virtuels python permettent de créer des environnements de développements ou d'exécutions isolés les uns des autres. C'est à dire que l'on va pouvoir y installer des bibliothèques sans contaminer les autres environnements python. Et l'on va pouvoir lister précisément les dépendances dont notre programme a besoin.
Fini la livraison où il manque des bibliothèques parce que dans un autre projet on a installé une bibliothèque que l'on utilise sans rendre compte. Installer le paquet de la distribution :

    pip3 install virtualenv virtualenvwrapper

Éditer notre .bashrc et ajouter les lignes suivantes :

    # virtualenv
    export WORKON_HOME=$HOME/.virtualenvs
    VIRTUALENVWRAPPER_PYTHON='/usr/bin/python3'
    source /usr/local/bin/virtualenvwrapper.sh

Pour céer un environnement virtuel :

    mkvirtualenv my_venv -p python3

Par exemple pour installer QT6 dans un environnement vituel :

    mkvirtualenv qt6_env -p python3

    (qt6_env) michaellaunay@luciole:~$ pip install numpy matplotlib PyQT6 # pour installer QT
    (qt6_env) michaellaunay@luciole:~$ pip install flake8 pylint # pour installer flake8 qui vérifie le respect de la pep8 et Pylint pour la vérification du code

Depuis python3.6, il existe un outil qui fusionne pip et virtual env :
**pipenv**

Installation de pipenv :

    pip3 install pipenv

Création d'un vitual env lors de l'installation d'un module

> cd \$MY\_WORKING\_DIR \# Aller dans notre espace de travail pipenv
> install pyside6 \--python=python3.11 \# Attention si python 3.11 n'est pas installé sur la machine ne pas oublier de faire \"apt install python3.9\"

Pour voir son virtualenv :

    pipenv --venv #Indique les virtual env associés avec le chemin courant

Pour activer le virtual env créé :

    pipenv shell #Depuis le répertoire de travail

Vérification de la conformité pep8 :

> (qt6\_env) <michaellaunay@luciole>:\~\$ flake8 demo.py

# Points de suspension
Nous utilisons généralement le mot-clé "pass" comme espace réservé pour du code non écrit, c'est à dire comme instruction neutre ne faisant rien. Mais nous pouvons aussi utiliser des points de suspension à cette fin.
```python
def ecrire_un_article():
    ...

class Auteur:
    ...
```


# Union de dictionnaires
Le pipe permet de faire l'union de dictionnaires
```python
>>> d1 = {1:"un",2:"deux",3:"trois"}
>>> d2 = {4:"quatre", 5:"cinq"}
>>> d1|d2
{1: 'un', 2: 'deux', 3: 'trois', 4: 'quatre', 5: 'cinq'}
>>> d3 = {5:"five", 6:"six"}
>>> d1|d2|d3
{1: 'un', 2: 'deux', 3: 'trois', 4: 'quatre', 5: 'five', 6: 'six'}
```

# Final
Depuis python 3.9, permet de déclarer une variable comme constante.
```python
from typing import Final
CONSTANTE:Final = "Une vairable constante"
```

# FString
Le formatage fstring est un appel automatique à format.
```python
>>> from datetime import datetime  
>>>   
>>> today = datetime.today()  
>>>   
>>> print(f"Today is {today}")  
Today is 2023-05-07 22:13:15.637951
>>>   
>>> print(f"Today is {today:%B %d, %Y}")  
Today is May 07, 2023
>>>   
>>> print(f"Today is {today:%m-%d-%Y}")  
Today is 05-07-2023

