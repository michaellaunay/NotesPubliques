Le scripting bash peut vite s'apparenter à du développement, alors pourquoi ne pas utiliser un vrai langage de programmation ?

# Historique

Python a été créé fin 1986 par Guido van Rossum au Centrum Wiskunde & Informatica (CWI), un institut de recherche en mathématiques et en informatique aux Pays-Bas. Le langage a été inspiré par le langage de programmation ABC, un langage de haut niveau utilisé pour l'enseignement de la programmation.

En 1995, avec la sortie de la version 1.2, le développement de Python a été financé par le Corporation for National Research Initiatives (CNRI), une organisation à but non lucratif basée aux États-Unis. 

En 2000, Python est passé à la version 2.0. À cette époque, le développement de Python a été soutenu par BeOpen.com, une entreprise qui a embauché Guido van Rossum et plusieurs autres développeurs Python clés.

En 2001, l'équipe de développement de Python a rejoint Digital Creations (qui deviendra plus tard Zope Corporation), une entreprise qui a développé le système de gestion de contenu open source Zope, écrit en Python. 

Toujours en 2001, la Python Software Foundation (PSF) a été créée. La PSF est une organisation à but non lucratif qui possède les droits d'auteur de Python et gère le développement et la distribution du langage. Avec la création de la PSF, le code de Python a été entièrement libéré sous la Python Software Foundation License, une licence open source approuvée par l'Open Source Initiative.

Depuis lors, Python a continué à évoluer et à croître en popularité, devenant l'un des langages de programmation les plus populaires au monde. Python 3.0, une révision majeure du langage qui n'est pas entièrement rétrocompatible, a été publié en 2008. En juillet 2018, Guido van Rossum a décidé de se retirer de son rôle de BDFL (Benevolent Dictator for Life) de Python, et un conseil de direction a été formé pour guider l'évolution future de Python.

# python.org

Le site [python.org](https://www.python.org/) est le site officiel du langage de programmation Python. Il fournit une multitude de ressources utiles à la fois pour les débutants et les développeurs expérimentés en Python.


## Documentation

Le site contient une documentation complète de Python, y compris des tutoriels pour les débutants, des guides détaillés sur les caractéristiques spécifiques de Python, et la documentation complète de la bibliothèque standard de Python.

## Téléchargements

Les dernières versions de Python peuvent être téléchargées à partir du site. Il fournit des binaires pour différentes plateformes, y compris Windows, Mac OS et Linux.

## **PEPs

Le site contient une liste complète des Python Enhancement Proposals (PEPs), qui sont les propositions officielles pour l'amélioration du langage Python. Les PEPs incluent des propositions de nouvelles fonctionnalités, des informations sur le processus de décision et des documents historiques.

## Communauté

Le site fournit des liens vers différents groupes et forums de la communauté Python, y compris la liste de diffusion Python-Dev et la liste de diffusion Python-ideas. Il y a aussi des liens vers des événements de la communauté Python, comme PyCon.

## **Ressources d'apprentissage

En plus de la documentation officielle, le site propose des liens vers d'autres ressources d'apprentissage, comme des livres, des cours en ligne et des tutoriels interactifs.

## Blogs

Le site héberge aussi le blog officiel de Python, qui fournit des nouvelles et des annonces importantes concernant le développement de Python.

# Les PEPs

Les PEP (Python Enhancement Proposals, ou Propositions d'amélioration de Python) sont des documents qui décrivent des nouvelles fonctionnalités ou des changements significatifs pour le langage de programmation Python. Chaque PEP est un document technique qui détaille soit une nouvelle fonctionnalité, soit une norme de conception ou de codage, soit une procédure.

Il existe plusieurs types de PEP.

## PEP standards

Ce sont des propositions qui décrivent une nouvelle fonctionnalité ou une modification du langage Python. Elles sont numérotées et passent par un processus de révision et d'approbation.

## PEP d'information

Ce sont des documents qui fournissent des informations générales à la communauté Python. Ils n'introduisent pas de nouvelles fonctionnalités et ne représentent pas un changement de politique.

## PEP de processus

Ce sont des documents qui décrivent un nouveau processus pour la gestion du langage Python ou proposent une modification à un processus existant.

## Processus d'approbation

Le processus d'approbation d'un PEP comprend plusieurs étapes :

1. **Création du PEP** : L'auteur du PEP rédige un document qui décrit en détail la proposition.

2. **Discussion du PEP** : La proposition est alors soumise à la communauté Python pour discussion. Cela se fait généralement sur la liste de diffusion python-dev.

3. **Révision du PEP** : Sur la base des commentaires reçus, l'auteur du PEP peut apporter des modifications à la proposition.

4. **Décision** : Un PEP est accepté ou rejeté par le BDFL (Benevolent Dictator for Life), une fonction historiquement occupée par le créateur de Python, Guido van Rossum. Depuis son retrait en 2018, cette fonction est assurée par un comité de direction.

5. **Implémentation** : Si un PEP est accepté, il est alors mis en œuvre dans le langage Python.

Il est important de noter que tous les PEP n'aboutissent pas à des modifications du langage. Certains sont rejetés, d'autres restent à l'état de proposition sans jamais être implémentés. Les PEP sont un outil important pour faciliter la communication et la coordination au sein de la communauté Python.

# PIP

Pour installer des bibliothèques nous aurons recours à l'utilitaire **pip**.

PIP (Pip Installs Packages) est un système de gestion de paquets utilisé pour installer et gérer des paquets de logiciels écrits en Python. Les utilisateurs de Python peuvent utiliser PIP pour installer des paquets à partir de l'index des paquets Python (PyPI) et d'autres dépôts. PIP offre les fonctionnalités suivantes :

- Installation de paquets Python.
- Désinstallation de paquets Python.
- Mise à jour de paquets Python.
- Recherche de paquets Python dans PyPI.
- Gestion des dépendances de paquets Python.

Pour utiliser PIP, vous devez d'abord l'installer. PIP est généralement déjà installé si vous avez Python version 2.7.9+ ou Python 3.4+. Sinon, vous pouvez l'installer manuellement.

# PyPI

PyPI (Python Package Index) est un dépôt de logiciels pour le langage de programmation Python. Les utilisateurs de Python peuvent chercher, télécharger et installer des paquets Python à partir de PyPI à l'aide de PIP ou d'autres outils.

PyPI héberge des milliers de paquets couvrant une grande variété de problèmes, allant des frameworks web aux bibliothèques scientifiques, en passant par les outils de développement et les applications autonomes.

PyPI est aussi la plateforme où les développeurs peuvent publier leurs propres paquets pour que d'autres personnes puissent les utiliser. Pour publier un paquet sur PyPI, vous avez besoin d'un compte PyPI et vous devez suivre les instructions pour empaqueter votre logiciel et le télécharger.

En résumé, PIP et PyPI sont deux outils essentiels pour tout développeur Python. PIP est utilisé pour installer et gérer les paquets Python, tandis que PyPI est l'endroit où ces paquets sont hébergés et partagés.

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

## Supprimer le retour à la ligne en fin de la fonction `print` en modifiant la valeur du paramètre `end`

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

## Le "sclicing" de chaînes de caractères

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

## Booléens

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

Par exemple, si nous créons une classe personnalisée et que nous voulons qu'elle puisse utiliser l'opérateur `+`, nous pouvons définir une méthode spéciale appelée `__add__`. Si nous ne voulons pas que l'opérateur `+` soit utilisé pour votre classe, nous pouvons faire que `__add__` renvoie `NotImplemented`.

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

Tout comme les dictionnaires, les ensembles sont des collections non ordonnées, mais ils présentent des caractéristiques distinctives qui les rendent uniques. Contrairement aux dictionnaires, qui stockent des paires clé-valeur, les ensembles stockent uniquement des valeurs uniques. De plus, ils sont mutables, ce qui signifie que nous pouvons ajouter, modifier ou supprimer des valeurs après leur création. 

Créons un exemple pour illustrer comment cela fonctionne:

```python
mon_ensemble = {1, 2, 3}
print(type(mon_ensemble))  # Affiche "<class 'set'>"
```

Ici, `mon_ensemble` est un ensemble qui contient trois valeurs uniques : 1, 2 et 3. Nous avons utilisé des accolades `{}`, tout comme pour les dictionnaires, mais sans associer de clés à nos valeurs.

Comme pour les dictionnaires, nous avons plusieurs opérations à notre disposition. Nous pouvons ajouter une valeur à l'ensemble avec la méthode `.add()`, comme suit :

```python
mon_ensemble.add(4)
```

Maintenant, `mon_ensemble` contient `{1, 2, 3, 4}`. Notez que si nous essayons d'ajouter une valeur déjà présente dans l'ensemble, rien ne se passe. C'est parce que les ensembles, par définition, ne contiennent que des valeurs uniques.

Pour supprimer une valeur, nous pouvons utiliser la méthode `.remove()`. Par exemple, `mon_ensemble.remove(2)` supprime la valeur `2` de notre ensemble. 

Mais attention ! Si la valeur que nous essayons de supprimer n'est pas présente dans l'ensemble, Python génère une erreur. Pour éviter cela, nous pouvons utiliser la méthode `.discard()`, qui ne génère pas d'erreur si la valeur n'est pas trouvée.

```python
mon_ensemble.discard(2)
```

En somme, les ensembles sont des outils puissants pour travailler avec des collections de valeurs uniques. Ils sont particulièrement utiles lorsque nous avons besoin de tester l'appartenance à un ensemble, de supprimer les doublons d'une collection, ou de calculer des opérations mathématiques telles que les intersections et les unions.

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

La plupart des types de collections intégrés dans Python, comme les listes, les tuples, les dictionnaires et les ensembles, sont itérables par défaut, ce qui signifie qu'ils retournent un itérateur lorsque nous les passons à la fonction `iter()`. Par exemple, si nous avons une liste de fruits, nous pouvons obtenir un itérateur pour cette liste comme suit :

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

En conclusion, les générateurs sont un outil puissant en Python qui nous permet de créer des itérateurs paresseux. Ils sont particulièrement utiles lorsque nous travaillons avec de grandes séquences de données qui ne tiendraient pas toutes en mémoire à la fois, ou lorsque le coût de calcul de chaque valeur de la séquence est élevé et que nous voulons le différer jusqu'à ce qu'il soit nécessaire.

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

## Envoi de valeurs et d'exceptions aux générateurs

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

# Outils d'itération
Il existe plusieurs bibliothèques qui fournissent des outils utiles pour travailler avec des itérateurs. Voici quelques-unes des plus couramment utilisées :

1. **itertools** : Ce module de la bibliothèque standard Python fournit une collection d'outils pour manipuler les itérateurs. Par exemple, `itertools.count` crée un itérateur qui produit une séquence infinie de nombres entiers, `itertools.cycle` crée un itérateur qui répète indéfiniment une séquence, et `itertools.chain` combine plusieurs itérateurs en un seul. Le module `itertools` offre également plusieurs autres fonctions utiles, comme `itertools.permutations`, `itertools.combinations`, et `itertools.product`, qui génèrent respectivement toutes les permutations, combinaisons, et le produit cartésien d'une collection d'éléments.

2. **functools** : Ce module de la bibliothèque standard Python fournit des outils de haut niveau pour travailler avec des fonctions et des itérables. Par exemple, `functools.reduce` applique une fonction binaire (c'est-à-dire une fonction à deux arguments) de manière répétée à tous les éléments d'un itérable pour produire une seule valeur.

3. **numpy** : Bien que ce ne soit pas strictement une bibliothèque d'itérateurs, numpy est une bibliothèque essentielle pour le calcul scientifique en Python, et elle fournit plusieurs outils pour travailler avec des tableaux multidimensionnels (qui sont des sortes d'itérateurs). Par exemple, nous pouvons utiliser `numpy.nditer` pour parcourir tous les éléments d'un tableau numpy dans un ordre spécifique.

4. **pandas** : Cette bibliothèque, qui est largement utilisée pour l'analyse de données en Python, fournit des structures de données comme les DataFrames et les Series qui sont essentiellement des itérateurs sur des données tabulaires. Nous pouvons parcourir les lignes d'un DataFrame ou les éléments d'une Series comme nous le ferions avec un itérateur, et pandas fournit également plusieurs méthodes pour appliquer des fonctions à ces structures de données de manière efficace.

Chacune de ces bibliothèques fournit une documentation complète avec des exemples de code, donc si nous voulons en savoir plus sur l'une d'entre elles, allons consulter la documentation officielle sur .

# Les opérateurs

Nous avons à notre disposition un large éventail d'opérateurs en Python qui nous permettent d'effectuer divers calculs et opérations sur nos données.

- **L'affectation `=`** : Nous utilisons l'opérateur `=` pour affecter une valeur à une variable. Par exemple, `x = 5` assigne la valeur `5` à la variable `x`.

- **La division `/`** : C'est l'opérateur de division standard. Par exemple, `5.0/6` donne comme résultat `0.83333...`.

- **La division entière `//`** : C'est une division qui arrondit le résultat à l'entier le plus proche vers le bas. Par exemple, `7 // 2` donne `3`.

- **Le modulo `%`** : Nous utilisons l'opérateur `%` pour obtenir le reste de la division. Par exemple, `5 % 3` donne `2`. Nous pouvons également utiliser la fonction `divmod(a, b)` qui retourne un tuple contenant le quotient et le reste. Par exemple, `divmod(5, 3)` donne `(1, 2)`.

- **La négation `-`** : Nous utilisons l'opérateur `-` pour inverser le signe d'une valeur. Par exemple, `-3` est l'inverse de `3`.

- **L'inversion bit à bit `~`** : C'est l'opérateur de complément à un. Il inverse tous les bits d'un nombre. Par exemple, `~3` donne `-4` car `3` est `11` en binaire et son inversion donne `100`, qui est `-4` en notation à deux compléments.

- **La puissance `**`** : Nous utilisons l'opérateur `**` pour élever un nombre à la puissance d'un autre. Par exemple, `2 ** 3` donne `8`.

- **Appartenance `in`** : Nous utilisons l'opérateur `in` pour vérifier si un élément est dans une collection. Par exemple, `3 in [1, 2, 3]` donne `True`.

- **L'appartenance à un type `isinstance()`** : Nous utilisons la fonction `isinstance()` pour vérifier si une valeur est d'un certain type. Par exemple, `isinstance(3, int)` donne `True` car `3` est un entier.

# Opérateurs binaires

Dans notre travail avec Python, nous nous trouvons souvent en présence de données binaires. Python fournit un ensemble d'opérateurs qui nous permettent de manipuler directement ces données.

- **Le et `&`** : L'opérateur binaire "et" renvoie un nombre dont chaque bit est le résultat de l'opération "et" sur les bits correspondants des opérandes. Par exemple, si nous avons `a = 60` (soit `00111100` en binaire) et `b = 13` (soit `00001101` en binaire), alors `a & b` donne `12` (soit `00001100` en binaire), car les troisième et quatrième bits sont les seuls bits qui sont à `1` dans les deux nombres.

- **Le ou `|`** : L'opérateur binaire "ou" fonctionne de la même manière, mais retourne un `1` pour chaque position où au moins un des bits opérandes est `1`. En utilisant les mêmes valeurs `a` et `b` que ci-dessus, `a | b` donne `61` (soit `00111101` en binaire), car tous les bits qui sont `1` dans `a` ou `b` sont `1` dans le résultat.

- **Le ou exclusif `^`** : L'opérateur "ou exclusif" retourne un `1` pour chaque position où exactement un des bits opérandes est `1`. En utilisant encore les mêmes valeurs `a` et `b`, `a ^ b` donne `49` (soit `00110001` en binaire), car les deuxième, cinquième, sixième et septième bits ont des valeurs différentes dans `a` et `b`. 

- **La négation `~`** : L'opérateur de négation binaire, aussi appelé complément à un, renvoie un nombre dont chaque bit est inversé par rapport à l'opérande original. Autrement dit, pour chaque bit dans l'opérande, si le bit est `1`, il devient `0`, et si le bit est `0`, il devient `1`. Par exemple, si `a = 60` (soit `00111100` en binaire), alors `~a` donne `-61` (soit `11000011` en complément à deux, qui est la représentation standard des entiers négatifs en binaire).

- **Décalage à gauche `<<`** : Cet opérateur déplace les bits d'un nombre vers la gauche. Par exemple, si `a = 5` (soit `101` en binaire), alors `a << 1` décale tous les bits de `a` d'un pas vers la gauche pour donner `10` (soit `1010` en binaire). En général, le décalage à gauche d'un nombre est équivalent à la multiplication de ce nombre par `2` à la puissance du nombre de décalages.

- **Décalage à droite `>>`** : Cet opérateur déplace les bits d'un nombre vers la droite. Par exemple, si `a = 5` (soit `101` en binaire), alors `a >> 1` décale tous les bits de `a` d'un pas vers la droite pour donner `2` (soit `10` en binaire). En général, le décalage à droite d'un nombre est équivalent à la division de ce nombre par `2` à la puissance du nombre de décalages, en arrondissant vers le bas.

Ces opérateurs peuvent être très utiles pour manipuler les données à un niveau bas, comme lors de l'interaction avec le matériel, du travail avec des protocoles de réseau, ou de l'implémentation de certains algorithmes de chiffrement et de compression.

# Opérateurs de comparaison

Les opérateurs de comparaison en Python sont utilisés pour comparer deux valeurs ou objets. Voyons comment nous les utilisons :

- **Inférieur `<`** : Cet opérateur retourne `True` si la valeur à gauche de l'opérateur est inférieure à celle à droite, sinon `False`. Par exemple, dans `2 < 3`, la sortie sera `True`.

- **Supérieur `>`** : Cet opérateur retourne `True` si la valeur à gauche de l'opérateur est supérieure à celle à droite, sinon `False`. Par exemple, dans `3 > 2`, la sortie sera `True`.

- **Inférieur ou égal `<=`** : Cet opérateur retourne `True` si la valeur à gauche de l'opérateur est inférieure ou égale à celle à droite, sinon `False`. Par exemple, dans `3 <= 3`, la sortie sera `True`.

- **Supérieur ou égal `>=`** : Cet opérateur retourne `True` si la valeur à gauche de l'opérateur est supérieure ou égale à celle à droite, sinon `False`. Par exemple, dans `3 >= 3`, la sortie sera `True`.

- **Égalité `==`** : Cet opérateur compare si les valeurs à gauche et à droite de l'opérateur sont égales. Si oui, il retourne `True`, sinon `False`. Par exemple, dans `3 == 3`, la sortie sera `True`.

- **Différence `!=` ou `<>`** : Ces opérateurs vérifient si les valeurs à gauche et à droite de l'opérateur sont différentes. Si c'est le cas, ils retournent `True`, sinon `False`. Par exemple, dans `3 != 2`, la sortie sera `True`. Notez que `<>` est moins couramment utilisé en Python moderne.

- **Est `is`** : Cet opérateur vérifie si deux variables pointent vers le même objet, pas si les objets ont la même valeur. Par exemple, dans `a is b`, il vérifie si `a` et `b` sont le même objet.

- **N'est pas `is not`** : Cet opérateur vérifie si deux variables ne pointent pas vers le même objet. Par exemple, dans `a is not b`, il vérifie si `a` et `b` sont des objets différents.

Il est important de noter que les opérateurs de comparaison peuvent être enchaînés pour vérifier plusieurs conditions à la fois, par exemple : `1 < a < 5`.

# Ordre de traitement des opérations

En programmation, tout comme en mathématiques, les opérations sont traitées selon un certain ordre de priorité, souvent désigné par l'acronyme anglais PEMDAS :

- Parenthèses (P)
- Exposants (E)
- Multiplication et Division (MD) - De gauche à droite
- Addition et Soustraction (AS) - De gauche à droite

Ce qui signifie que dans une expression, les opérations entre parenthèses sont effectuées en premier, suivies par les exposants (comme les carrés et les racines carrées), puis la multiplication et la division, et enfin l'addition et la soustraction. 

Par exemple, pour l'expression `3 + 2 * (5 - 1)`, le calcul se fait comme suit : `3 + 2 * 4` (parenthèses en premier), ensuite `3 + 8` (multiplication ensuite), et finalement `11` (addition en dernier).

## L'instruction pass

En Python, l'instruction `pass` est une opération qui ne fait rien. Elle est généralement utilisée comme une instruction de remplissage lorsqu'une déclaration est requise syntaxiquement, mais que le programme ne requiert aucune action. Par exemple, elle peut être utilisée pour créer une classe, une fonction ou une boucle vide, en attente de contenu à ajouter plus tard :

```python
def ma_fonction():
    pass  # Je remplirai cette fonction plus tard
```

## L'instruction ...

En Python, le symbole `...` est également utilisé comme une instruction de remplissage, tout comme `pass`. Cependant, son utilisation est un peu différente.

En Python, `...` est une constante spéciale appelée "Ellipsis". Par défaut, elle n'a aucun comportement et est généralement utilisée comme un espace réservé lorsque nous souhaitons indiquer qu'un code doit être ajouté ultérieurement.

Nous pouvons l'utiliser de la même manière que `pass` pour créer une structure vide, par exemple une classe, une fonction ou une boucle :

```python
def ma_fonction():
    ...  # Je remplirai cette fonction plus tard
```

Cependant, l'Ellipsis a également des utilisations spécifiques dans certains contextes, comme l'indexation de structures de données de haute dimension dans des bibliothèques scientifiques comme NumPy.

De plus, certains outils de vérification de type utilisent l'Ellipsis pour indiquer qu'une annotation de type spécifique n'a pas encore été ajoutée à une variable ou à une fonction. Par exemple :

```python
def ma_fonction(a: int, b: ...) -> ...:  # Les types pour 'b' et le type de retour ne sont pas encore spécifiés
    ...
```

En somme, bien que `pass` et `...` puissent être utilisés de manière interchangeable dans certains contextes, ils ont des utilisations spécifiques qui les rendent uniques. Il est donc important de comprendre ces différences pour les utiliser de manière appropriée.

## L'indentation

En Python, l'indentation (les espaces au début de certaines lignes de code) est très importante. Elle est utilisée pour définir le bloc de code associé à une structure de contrôle particulière (comme une boucle ou une condition), à une fonction ou à une classe. 

Cela signifie que contrairement à de nombreux autres langages de programmation, où les blocs de code sont généralement délimités par des accolades `{}`, en Python, c'est l'indentation elle-même qui délimite les blocs.

Par exemple, voici comment une boucle `for` serait écrite en Python :

```python
for i in range(5):
    print(i)  # Cette ligne est dans le bloc de la boucle 'for' car elle est indentée
print("Fin de la boucle")  # Cette ligne n'est pas dans le bloc de la boucle 'for' car elle n'est pas indentée
```

Dans cet exemple, la ligne `print(i)` est dans le bloc de la boucle `for` car elle est indentée. Elle sera donc répétée pour chaque itération de la boucle. En revanche, la ligne `print("Fin de la boucle")` n'est pas dans le bloc de la boucle `for` car elle n'est pas indentée. Elle sera donc exécutée une fois que toutes les itérations de la boucle auront été effectuées.

# Les structures conditionnelles

## L'instruction if (else et elif)

En Python, l'instruction `if` est utilisée pour évaluer une condition. Si la condition est vraie, le bloc de code sous l'instruction `if` est exécuté. 

```python
if condition:
    # Bloc de code exécuté si la condition est vraie
```

La clause `else` peut être ajoutée après un `if` pour spécifier un bloc de code à exécuter si la condition `if` est fausse.

```python
if condition:
    # Bloc de code exécuté si la condition est vraie
else:
    # Bloc de code exécuté si la condition est fausse
```

`elif` (abréviation de "else if") peut être utilisé pour vérifier plusieurs conditions. Si la condition de l'instruction `if` est fausse, la condition de l'instruction `elif` est alors évaluée. Si cette condition est vraie, le bloc de code sous l'instruction `elif` est exécuté.

```python
if condition1:
    # Bloc de code exécuté si condition1 est vraie
elif condition2:
    # Bloc de code exécuté si condition1 est fausse et condition2 est vraie
else:
    # Bloc de code exécuté si condition1 et condition2 sont fausses
```

## Les opérateurs ternaires

En Python, l'opérateur ternaire est une manière concise d'écrire une instruction `if`-`else`. Il permet de tester une condition et de renvoyer une valeur si cette condition est vraie et une autre valeur si elle est fausse. La syntaxe est la suivante :

```python
valeur_si_vrai if condition else valeur_si_faux
```

Par exemple :

```python
x = 10
message = "x est supérieur à 5" if x > 5 else "x est inférieur ou égal à 5"
print(message)  # Affiche "x est supérieur à 5"
```

## Les opérateurs logiques : AND, OR et NOT

Python utilise les opérateurs `and`, `or` et `not` pour combiner ou inverser des conditions logiques.

- `and` : Cet opérateur renvoie `True` si les deux conditions qu'il combine sont vraies. Par exemple, `x > 0 and x < 10` est vrai seulement si `x` est à la fois supérieur à 0 et inférieur à 10.

- `or` : Cet opérateur renvoie `True` si au moins l'une des conditions qu'il combine est vraie. Par exemple, `x < 0 or x > 10` est vrai si `x` est inférieur à 0 ou supérieur à 10.

- `not` : Cet opérateur inverse la vérité d'une condition. Par exemple, `not x > 0` est vrai seulement si `x` n'est pas supérieur à 0.

Il est à noter que Python n'a pas d'opérateur ternaire `? :` comme on peut le trouver dans d'autres langages de programmation comme Java ou C++. En Python, on utilise plutôt la syntaxe `valeur_si_vrai if condition else valeur_si_faux` pour les opérations ternaires.

Voici un exemple simple d'utilisation de l'opérateur ternaire `if` en Python :

```python
age = 20
categorie = "Adulte" if age >= 18 else "Enfant"
print(categorie)  # Affiche "Adulte"
```

Dans cet exemple, la variable `age` est fixée à 20. L'opérateur ternaire est utilisé pour déterminer la valeur de la variable `categorie`. Si `age` est supérieur ou égal à 18 (ce qui est le cas ici), `categorie` est fixée à "Adulte". Sinon (si `age` est inférieur à 18), `categorie` serait fixée à "Enfant".

C'est une manière très concise d'écrire une condition `if`-`else`, et c'est particulièrement utile quand nous voulons affecter une valeur à une variable en fonction d'une condition.

## L'instruction for in

L'instruction `for` en Python est utilisée pour itérer sur une séquence (comme une liste, un tuple, un dictionnaire, un ensemble ou une chaîne) ou d'autres objets itérables. L'instruction `for` est souvent utilisée lorsque nous avons un bloc de code que nous voulons répéter un nombre fixe de fois.

```python
for variable in sequence:
    # Bloc de code exécuté pour chaque élément dans la séquence
```

## L'instruction while

L'instruction `while` en Python est utilisée pour la boucle de contrôle. Le bloc de code sous l'instruction `while` est exécuté aussi longtemps que la condition est vraie. 

```python
while condition:
    # Bloc de code exécuté tant que la condition est vraie
```

Attention à ne pas créer de boucle infinie avec l'instruction `while` ! Assurons-nous que la condition deviendra fausse à un moment donné. Sinon, le programme continuera à exécuter le bloc de code pour toujours.

# Utilisation de `else` dans un `for` ou un `while`
Les blocs `else` peuvent également être utilisés avec les boucles `for` et `while` en Python. C'est une fonctionnalité quelque peu unique à Python et elle n'est pas présente dans beaucoup d'autres langages de programmation.

Le bloc de code dans une clause `else` après une boucle `for` ou `while` est exécuté lorsque la boucle a fini de parcourir la liste (dans le cas de `for`) ou lorsque la condition devient fausse (dans le cas de `while`), mais seulement si la boucle n'a pas été interrompue par une instruction `break`.

Voici un exemple avec une boucle `for`:

```python
for i in range(5):
    print(i)
else:
    print("La boucle est terminée.")
```

Dans cet exemple, le message "La boucle est terminée." sera affiché après que tous les nombres de 0 à 4 aient été imprimés.

Et un exemple avec une boucle `while` :

```python
i = 0
while i < 5:
    print(i)
    i += 1
else:
    print("La boucle est terminée.")
```

Ici encore, le message "La boucle est terminée." sera affiché après que tous les nombres de 0 à 4 aient été imprimés.

## L'instruction match

L'instruction `match` est une fonctionnalité introduite dans Python 3.10. Elle sert à faire du "pattern matching" sur des données. Elle peut rendre notre code plus lisible et plus sûr, en particulier lorsqu'il s'agit de gérer des structures de données complexes.

La syntaxe de base de `match` est similaire à celle de `switch` dans d'autres langages de programmation, mais elle est beaucoup plus puissante car elle permet de correspondre à des motifs complexes, et non seulement à des valeurs scalaires.

Prenons un exemple simple pour commencer :

```python
x = 1
match x:
    case 1:
        print("Un")
    case 2:
        print("Deux")
    case _:
        print("Autre")
```

Ici, `x` est comparé à différents cas. Si `x` est `1`, le programme affiche "Un". Si `x` est `2`, il affiche "Deux". Le cas `_` est un joker qui correspond à n'importe quelle autre valeur. Il est généralement utilisé comme dernier cas pour capturer toutes les valeurs non traitées par les cas précédents.

Mais où l'instruction `match` est vraiment intéressante, c'est dans sa capacité à correspondre à des structures de données complexes et à extraire des motifs intéressants. Par exemple :

```python
point = (2, 3)
match point:
    case (0, 0):
        print("Origine")
    case (x, 0):
        print(f"Point sur l'axe des X à {x}")
    case (0, y):
        print(f"Point sur l'axe des Y à {y}")
    case (x, y):
        print(f"Point à {x}, {y}")
    case _:
        raise ValueError("Impossible à traiter")
```

Dans cet exemple, `point` est une paire de coordonnées. L'instruction `match` est utilisée pour vérifier si le point est à l'origine, sur l'un des axes, ou à une position quelconque. Les variables `x` et `y` sont utilisées pour extraire les valeurs de la paire, que nous pouvons ensuite utiliser dans le corps de chaque cas.

Pour conclure, l'instruction `match` est utile pour structurer notre code de manière à gérer différents cas de façon claire et explicite. Il s'agit d'une avancée majeure dans le langage Python qui facilite l'écriture de code plus robuste et plus facile à comprendre.

# Les fonctions

En Python, nous utilisons le mot-clé `def` pour définir une fonction. Une fois définie, une fonction est un objet à part entière, avec son propre espace de noms.

Le corps de la fonction doit être correctement indenté après le signe `:` qui suit le nom de la fonction et la liste de ses paramètres.

Les paramètres ne sont pas typés, c'est-à-dire qu'aucun type de donnée spécifique n'est exigé par la fonction pour chaque paramètre. Cela rend Python très flexible, mais nous devons être prudents pour éviter les erreurs de typage.

Nous pouvons donner à nos paramètres des valeurs par défaut. Par exemple, dans la déclaration `def ma_fonction(p1=0):`, `p1` a une valeur par défaut de `0`. Si aucune valeur n'est fournie pour `p1` lors de l'appel à `ma_fonction`, alors `p1` aura la valeur `0` à l'intérieur de la fonction.

Python nous permet également de définir des paramètres non explicites et arbitraires. Si nous définissons une fonction avec `def ma_fonction(**kwargs):`, alors tous les paramètres de mot-clé supplémentaires passés lors de l'appel à `ma_fonction` seront empaquetés dans un dictionnaire `kwargs`.
De même, une déclaration de fonction comme `def ma_fonction(*args):` regroupera tous les arguments positionnels supplémentaires dans un tuple `args`.
Nous pouvons également combiner ces deux types de paramètres dans une seule déclaration de fonction, comme `def ma_fonction(*args, **kwargs):`.

Lors de la définition des fonctions, il est possible de contrôler la façon dont les arguments sont passés à la fonction grâce à certains séparateurs spécifiques dans la signature de la fonction. Ces séparateurs sont **`,`**,**`*`**, et **`/`**.

- **`,`** : Le séparateur le plus commun est la virgule `,` qui est utilisée pour séparer les paramètres dans la liste des arguments d'une fonction. Les paramètres définis avant l'éventuel `*` ou `/` dans la signature de la fonction sont généralement des paramètres positionnels ou de mot-clé.

- **`*`** : Lorsqu'il est utilisé dans la signature d'une fonction, le symbole `*` a une signification spéciale. Les paramètres définis après `*` sont des paramètres à mot-clé uniquement. Cela signifie qu'ils ne peuvent être fournis que avec leurs noms spécifiques lors de l'appel à la fonction, et non par leur position. De plus, `*args` est utilisé pour capturer un nombre indéfini d'arguments positionnels dans un tuple.

- **`/`** : Introduit dans Python 3.8, le séparateur `/` indique que tous les paramètres définis avant lui sont des paramètres positionnels uniquement. Ces paramètres ne peuvent être passés que par leur position et non par leur nom lors de l'appel à la fonction.

Voici un exemple qui illustre l'utilisation de ces séparateurs :

```python
def exemple(a, b, /, c, *, d):
    print(a, b, c, d)

exemple(1, 2, c=3, d=4)  # Correct
exemple(1, 2, 3, d=4)    # Correct
exemple(a=1, b=2, c=3, d=4)  # Erreur, a et b ne peuvent pas être passés comme paramètres de mot-clé
exemple(1, 2, 3, 4)      # Erreur, d doit être passé comme paramètre de mot-clé
```

Ces séparateurs offrent un contrôle plus précis sur la façon dont les arguments sont passés aux fonctions, améliorant ainsi la lisibilité et la robustesse du code.

La directive `return` nous permet de spécifier quelle valeur doit être retournée par la fonction. Si aucune directive `return` n'est présente, ou si elle est présente sans aucune expression suivante, la fonction retournera `None`.

Enfin, Python nous offre la directive `lambda` pour la création rapide de petites fonctions anonymes. Ces fonctions sont souvent utilisées là où une fonction complète serait excessive, par exemple lors du passage d'une petite fonction comme argument à une autre fonction, comme dans le cas de l'utilisation de fonctions comme `map` ou `filter`. Par exemple, `lambda x: x * 2` est une fonction qui prend un argument `x` et retourne `x * 2`.

# Les docstrings

Nous pouvons documenter nos fonctions à l'aide de docstrings. Les docstrings sont des chaînes de texte placées juste après la définition d'une fonction, d'une méthode, d'une classe ou d'un module qui expliquent ce que fait le bloc de code.

Lors de l'exécution d'un programme, nous pouvons interroger une fonction pour comprendre son fonctionnement en accédant à sa docstring. Cela est particulièrement utile pour le travail en équipe, l'utilisation d'IDE, ou si nous reprenons un code que nous n'avons pas consulté depuis un certain temps.

Voici un exemple d'une fonction avec une docstring :

```python
def add(a, b):
    """
    Cette fonction prend deux arguments, a et b, et retourne leur somme.
    """
    return a + b
```

Nous pouvons accéder à la docstring de cette fonction en utilisant l'attribut `__doc__` de la fonction :

```python
print(add.__doc__)
```

Cela affiche la docstring de la fonction :

```
Cette fonction prend deux arguments, a et b, et retourne leur somme.
```

En utilisant les docstrings, nous aidons à rendre notre code plus compréhensible et accessible, aussi bien pour nous-mêmes que pour les autres et pour les outils.

Le format habituel pour documenter les paramètres, les valeurs de retour et les exceptions dans une docstring suit le format proposé par la norme de style PEP 257. Une norme largement adoptée est également le format Google. Voici un exemple qui utilise ce format :

```python
def add(a, b):
    """
    Cette fonction prend deux arguments, a et b, et retourne leur somme.

    Args:
        a (int ou float): Le premier nombre à additionner.
        b (int ou float): Le second nombre à additionner.

    Returns:
        int ou float: La somme de a et b.

    Raises:
        TypeError: Si a ou b ne sont pas de type int ou float.
    """
    if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
        raise TypeError('Les arguments a et b doivent être de type int ou float.')
    return a + b
```

Dans cet exemple, "Args" est utilisé pour décrire les arguments de la fonction, "Returns" pour décrire la valeur de retour, et "Raises" pour décrire les exceptions que la fonction peut lever.

Notons que dans la pratique, la plupart des fonctions Python ne nécessitent pas une docstring aussi détaillée. Cependant, pour les bibliothèques et les frameworks, ou pour toute fonction qui est suffisamment complexe, avoir une docstring détaillée peut être très utile.

# Les annotations

En plus des docstrings, Python fournit un autre moyen d'ajouter des métadonnées aux fonctions : les annotations. Les annotations sont une fonctionnalité du langage qui permet d'associer des informations arbitraires aux arguments de fonction et aux valeurs de retour. Elles sont souvent utilisées pour indiquer les types attendus des arguments et de la valeur de retour, bien qu'elles puissent en principe contenir n'importe quel type de données.

Voici un exemple d'une fonction avec des annotations :

```python
def add(a: int, b: int) -> int:
    return a + b
```

Dans cet exemple, `: int` après `a` et `b` est une annotation indiquant que ces arguments sont supposés être des entiers. `-> int` après la liste des arguments est une annotation indiquant que la fonction est supposée renvoyer un entier.

Les annotations ne sont pas utilisées par Python lui-même. Elles sont disponibles pour être utilisées par des outils de vérification de type, des IDE et d'autres outils d'analyse de code. Python n'effectuera aucune vérification de type sur la base des annotations et ne fera rien si les annotations ne correspondent pas aux types réels des valeurs.

Pour accéder aux annotations d'une fonction, nous pouvons utiliser l'attribut `__annotations__` de la fonction. C'est un dictionnaire contenant les noms des arguments comme clés et leurs annotations comme valeurs, avec la clé `'return'` pour l'annotation de la valeur de retour.

```python
print(add.__annotations__)
```

Cela affiche le dictionnaire des annotations de la fonction :

```
{'a': <class 'int'>, 'b': <class 'int'>, 'return': <class 'int'>}
```

Cependant, bien que les annotations soient une fonctionnalité puissante et flexible, il est important de noter qu'elles ne remplacent pas les docstrings pour la documentation des fonctions. Les docstrings peuvent fournir des informations beaucoup plus détaillées et explicatives que les annotations. Les deux outils peuvent être utilisés ensemble pour rendre nos fonctions aussi claires et compréhensibles que possible.

# Les Classes

Une classe est une structure qui nous permet de regrouper des données, que nous appelons des attributs, et des fonctions, que nous appelons des méthodes, qui opèrent sur ces données. Les classes sont définies avec le mot-clé `class`.

## L'héritage

Un point important à noter est que les classes peuvent hériter d'autres classes, permettant ainsi le partage de fonctionnalités entre plusieurs classes. Cette caractéristique est connue sous le nom d'héritage.

Regardons un exemple :

```python
class Animal(object):
    def __init__(self):
        self.age = 0
        self.poids = 0

class Chat(Animal):
    def __init__(self, nom):
        super(Chat, self).__init__()  # Appelle le constructeur de la classe parente, Animal
        self.nom = nom
```

Dans cet exemple, nous avons une classe `Animal` qui possède deux attributs, `age` et `poids`. Nous avons ensuite une classe `Chat` qui hérite de `Animal`, et ajoute un autre attribut, `nom`. La fonction `super(Chat, self).__init__()` est utilisée pour appeler le constructeur de la classe parente, `Animal`, ce qui permet à `Chat` d'initialiser les attributs `age` et `poids`.

L'héritage est un concept fondamental en programmation orientée objet, qui permet à une classe d'hériter des attributs et des méthodes d'une autre classe. La classe qui hérite est souvent appelée la classe enfant ou sous-classe, et la classe dont elle hérite est appelée la classe parent ou superclasse.

L'héritage est utile car il favorise la réutilisation du code et peut rendre le code plus organisé et plus facile à comprendre.

## La notion de constructeur et de destructeur

- Le **constructeur** est une méthode spéciale, `__init__`, qui est appelée automatiquement lorsque nous créons une nouvelle instance de la classe. Elle est généralement utilisée pour initialiser les attributs de l'objet.

- Le **destructeur** est une autre méthode spéciale, `__del__`, qui est appelée automatiquement lorsque l'objet est sur le point d'être détruit. Elle est rarement utilisée, car Python gère automatiquement la gestion de la mémoire pour nous.

Python nous offre la possibilité de définir des attributs privés en préfixant le nom de l'attribut avec deux tirets bas `__`. Ces attributs ne peuvent pas être directement accessibles en dehors de la classe. Par exemple, `__attribut_prive` serait un attribut privé. Cependant, il faut noter que Python ne supporte pas la notion d'attributs strictement privés comme dans d'autres langages, et cette convention est plutôt une suggestion pour indiquer que ces attributs ne doivent pas être touchés en dehors de la classe.

# Le polymorphisme

Le polymorphisme est un autre concept clé de la programmation orientée objet. Il se réfère à la capacité d'une entité, comme une variable, une fonction, ou un objet, à prendre plusieurs formes. En Python, le polymorphisme se manifeste de plusieurs façons.

- **Polymorphisme avec les méthodes de classe** : Grâce à l'héritage, une sous-classe peut hériter d'une méthode d'une superclasse et ensuite la surcharger pour donner un comportement différent à la méthode.

- **Polymorphisme avec les fonctions et les objets** : Il est possible de créer des fonctions qui peuvent prendre n'importe quel objet, permettant ainsi pour des comportements différents basés sur l'objet qui est passé à la fonction.

Voici un exemple illustrant ces deux formes de polymorphisme :

```python
class Animal:
    def faire_bruit(self):
        pass

class Chien(Animal):
    def faire_bruit(self):
        print("Woof!")

class Chat(Animal):
    def faire_bruit(self):
        print("Meow!")

def faire_bruit(animal):
    animal.faire_bruit()

animaux = [Chien(), Chat()]

for animal in animaux:
    faire_bruit(animal)
```

Dans cet exemple, la méthode `faire_bruit` est polymorphique : la version utilisée dépend de l'objet sur lequel elle est appelée. La fonction `faire_bruit` est également polymorphique : elle peut prendre n'importe quel objet qui a une méthode `faire_bruit`.

Ces deux concepts, l'héritage et le polymorphisme, sont fondamentaux en programmation orientée objet et permettent de créer des codes plus modulables et plus facilement réutilisables.

# Les méthodes de classe

En Python, une méthode de classe est une méthode qui est liée à la classe et non à l'instance de la classe. Elle peut modifier l'état de la classe qui affecte toutes les instances de la classe. En Python, une méthode de classe est marquée par un décorateur `@classmethod`.

Voici un exemple de méthode de classe :

```python
class MaClasse:
    compteur = 0

    @classmethod
    def incrementer_compteur(cls):
        cls.compteur += 1
```

Dans cet exemple, `incrementer_compteur` est une méthode de classe qui modifie une variable de classe, `compteur`. Notez l'utilisation de `cls` comme premier argument de la méthode de classe, qui est une convention pour faire référence à la classe elle-même.

# Le `__new__` et `__del__`

En Python, `__new__` est une méthode spéciale qui est principalement utilisée dans les classes avec une métaclasse ou lors de la surcharge de types immuables ou de classes. C'est la première étape de la création d'une instance, elle est appelée avant `__init__` et est responsable de renvoyer une nouvelle instance de la classe.

Voici un exemple d'utilisation de `__new__` :

```python
class MaClasse:
    def __new__(cls):
        print("Instance créée.")
        instance = super().__new__(cls)
        return instance

    def __init__(self):
        print("Instance initialisée.")
```

Dans cet exemple, la méthode `__new__` est définie pour imprimer un message lors de la création d'une instance.

`__del__`, en revanche, est une méthode qui est appelée lorsque l'instance est sur le point d'être détruite. Bien que ce soit une fonctionnalité utile, `__del__` n'est pas souvent utilisé car Python gère la mémoire et le nettoyage automatiquement. 

Voici un exemple d'utilisation de `__del__` :

```python
class MaClasse:
    def __del__(self):
        print("Instance détruite.")
```

Dans cet exemple, `__del__` est défini pour imprimer un message lors de la destruction de l'instance.

## Le super()

En programmation orientée objet, le `super()` est une fonction intégrée que nous utilisons pour appeler une méthode de la classe parente. Cela peut être très utile, en particulier dans le cas de l'héritage où une méthode a été réécrite dans une classe enfant, mais nous voulons toujours utiliser certaines parties de la méthode de la classe parente.

`super()` est souvent utilisé dans le constructeur `__init__` pour s'assurer que l'initialisation faite dans la classe parente est également effectuée pour la classe enfant.

Voici un exemple de comment `super()` pourrait être utilisé :

```python
class Animal:
    def __init__(self, nom):
        self.nom = nom

class Chien(Animal):
    def __init__(self, nom, race):
        super().__init__(nom)  # Appelle le constructeur de Animal pour initialiser le nom
        self.race = race
```

Dans cet exemple, la classe `Chien` hérite de la classe `Animal`. Nous voulons initialiser la variable `nom` de la classe `Animal` quand nous créons une instance de `Chien`, donc nous utilisons `super()` pour appeler le constructeur de `Animal`. Cela signifie que nous n'avons pas à répéter le code d'initialisation du nom dans la classe `Chien`, ce qui rend notre code plus propre et plus facile à maintenir.

Notons que l'appel à `super()` n'a pas besoin de spécifier explicitement la classe parente. Python détermine automatiquement la classe parente en fonction de l'ordre d'héritage, ce qui est particulièrement utile dans le cas de l'héritage multiple.

## L'héritage en diamant

L'héritage en diamant, aussi connu sous le nom de "problème du diamant", est une complication qui peut survenir dans les langages qui supportent l'héritage multiple, comme Python. Il se produit lorsque une classe hérite de deux classes qui ont une classe parente commune.

Voici un exemple d'héritage en diamant :

```python
class A:
    def saluer(self):
        print("Salut de A")

class B(A):
    def saluer(self):
        print("Salut de B")

class C(A):
    def saluer(self):
        print("Salut de C")

class D(B, C):
    pass

d = D()
d.saluer()
```

Ici, la classe `D` hérite des classes `B` et `C`, qui elles-mêmes héritent toutes deux de la classe `A`. Si nous créons une instance de `D` et appelons la méthode `saluer`, quelle version de la méthode devrait être appelée ? Celle de `B` ou celle de `C` ?

Python résout ce problème grâce à une méthode appelée Méthode de Résolution de l'Ordre (MRO), qui suit le principe de la recherche en profondeur d'abord, de gauche à droite. Donc dans l'exemple ci-dessus, la méthode `saluer` de la classe `B` sera appelée.

L'utilisation de la fonction `super()` dans ce contexte peut également être source de confusion. En général, `super()` fait référence à la classe parente dans l'ordre de MRO, qui peut ne pas être la classe parente directe dans le schéma d'héritage. C'est pourquoi il est généralement recommandé de toujours utiliser `super()` pour appeler des méthodes de classes parentes, car cela garantit que toutes les méthodes sont appelées dans le bon ordre.

L'héritage multiple et l'héritage en diamant en particulier peuvent être des concepts difficiles à comprendre et à gérer correctement, c'est pourquoi certains langages de programmation, comme Java, ne les supportent pas du tout. Dans de nombreux cas, il peut être plus simple et plus sûr d'utiliser la composition ou l'agrégation d'objets plutôt que l'héritage multiple.

# La notion d'interface

Dans la programmation orientée objet, une interface est une spécification qui décrit les comportements (méthodes) qu'une classe doit implémenter. En d'autres termes, une interface définit "ce que" une classe doit faire, mais pas "comment" elle doit le faire. C'est un contrat qui garantit qu'un certain type d'objet se comportera d'une certaine manière. 

Python n'a pas de support intégré pour les interfaces comme Java ou C#. Cependant, le concept peut être réalisé en utilisant des classes abstraites et des méthodes abstraites.

Voici un exemple d'interface en Python :

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def faire_du_bruit(self):
        pass

class Chien(Animal):
    def faire_du_bruit(self):
        return "Woof!"

class Chat(Animal):
    def faire_du_bruit(self):
        return "Meow!"
```

Dans cet exemple, `Animal` est une classe abstraite qui définit une méthode abstraite `faire_du_bruit()`. Cette méthode est le contrat que chaque sous-classe doit respecter. `Chien` et `Chat` sont des sous-classes qui implémentent cette interface en fournissant leur propre implémentation de `faire_du_bruit()`.

Il est important de noter que nous ne pouvons pas créer une instance d'une classe abstraite qui a des méthodes abstraites non implémentées. Cela signifie que si nous essayons de créer une instance de `Animal`, Python nous donnera une erreur. Les classes abstraites sont destinées à être sous-classées, et les méthodes abstraites doivent être implémentées dans la sous-classe.

Les interfaces sont un outil puissant pour la conception de programmes, car elles permettent de définir des comportements attendus et de garantir qu'un certain contrat sera respecté par toutes les classes qui implémentent l'interface.

Voir la [documentation](https://docs.python.org/3/library/abc.html) officielle pour plus d'informations sur le module abc.

## Le type Self à partir de python 3.11

L'annotation Self, introduite à partir de Python 3.11, offre une manière à la fois intuitive et simple d'annoter les méthodes qui retournent une instance de leur propre classe. Cette approche, bien qu'elle corresponde à celle basée sur TypeVar tel que décrit dans le [PEP 484](https://peps.python.org/pep-0484/), se distingue par sa concision et sa facilité de compréhension.

Nous pouvons souvent rencontrer son utilisation dans des contextes tels que des constructeurs alternatifs proposés en tant que méthodes de classe, ou encore des méthodes **enter**() qui retournent self

``` python
class MyLock:
    def __enter__(self) -> Self:
        self.lock()
        return self

    ...

class MyInt:
    @classmethod
    def fromhex(cls, s: str) -> Self:
        return cls(int(s, 16))

    ...
```

Self peut également être utilisé pour annoter les paramètres de méthode ou les attributs du même type que leur classe englobante.

Pour plus de détails, voir [PEP 673](https://peps.python.org/pep-0673/).

# Les design patterns en python

Un motif de conception (design pattern) est une solution générale réutilisable à un problème couramment rencontré dans la conception logicielle. Un motif de conception n'est pas un code finalisé qui peut être directement transformé en code source ou en machine. Il s'agit d'une description ou d'un modèle pour résoudre un problème qui peut être utilisé dans de nombreux contextes de conception différents.

Les motifs de conception peuvent accélérer le processus de développement en fournissant des paradigmes de testés et éprouvés. Les développeurs expérimentés sont souvent en mesure de reconnaître les situations où l'application d'un design pattern pourrait aider à résoudre un problème de manière efficace et élégante.

Les motifs de conception sont généralement divisés en trois catégories :

1. **Les motifs de création** : Ces motifs se concentrent sur les façons de créer des objets de manière à ce que le système soit indépendant de la manière dont ses objets sont créés, composés et représentés. Exemples : Singleton, Factory, Abstract Factory, Builder, Prototype.

2. **Les motifs structurels** : Ces motifs se concentrent sur la façon dont les classes et les objets sont composés pour former des structures plus grandes. Exemples : Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy.

3. **Les motifs comportementaux** : Ces motifs se concentrent sur les algorithmes et l'affectation des responsabilités entre les objets. Exemples : Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor.

Pour les implémenter en Python, il existe plusieurs ressources, dont des livres et des tutoriels en ligne. L'un des plus connus est le livre "Design Patterns: Elements of Reusable Object-Oriented Software" par Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides, surnommés le "Gang of Four". Pour une approche spécifique à Python, vous pouvez consulter le livre "Python 3 Object-Oriented Programming" par Dusty Phillips, qui couvre les motifs de conception dans le contexte de Python. Enfin, un projet GitHub appelé "Python Patterns" recueille divers exemples de motifs de conception implémentés en Python : https://github.com/faif/python-patterns.

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

Dans cet exemple, `mon_decorateur` est un décorateur qui prend une fonction en argument (`fonction`) et retourne une nouvelle fonction (`fonction_enveloppe`). La nouvelle fonction exécute du code avant et après l'appel de la fonction originale. Le symbole `@` est utilisé pour appliquer le décorateur à la fonction `dis_bonjour`. Lorsque nous appelons `dis_bonjour()`, la fonction modifiée par le décorateur est exécutée.

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

# Les exceptions

Les exceptions en Python sont des erreurs qui sont détectées pendant l'exécution d'un programme. Elles sont signalées par le mécanisme d'exception de Python, qui arrête l'exécution normale du programme et transfère le contrôle à un code qui est conçu pour gérer les erreurs, appelé gestionnaire d'exceptions.

Voici un exemple simple qui illustre le mécanisme des exceptions :

```python
try:
    # Nous essayons d'accéder à une variable qui n'est pas définie
    print(PasDefinie)
except NameError:
    # Si une exception de type NameError est levée dans le bloc try,
    # ce bloc de code est exécuté
    print("Variable non définie")
```

Dans cet exemple, nous essayons d'accéder à une variable `PasDefinie` qui n'a pas été définie auparavant. Par conséquent, Python lève une exception `NameError`. Comme nous avons mis le code dans un bloc `try` et fourni un gestionnaire `except` pour `NameError`, Python exécute le code dans le bloc `except` lorsqu'il rencontre l'erreur, et imprime "Variable non définie".

Il est important de noter que les exceptions sont des erreurs graves qui, si elles ne sont pas gérées, peuvent arrêter l'exécution de notre programme. Il est donc essentiel d'avoir une bonne compréhension de la façon de gérer les exceptions lorsque nous écrivons du code Python.

## Arborescence des exceptions

En Python, toutes les exceptions sont dérivées de la classe de base `BaseException`. En général, nous travaillons avec des exceptions  dérivées de la classe `Exception`, qui est elle-même hérite de `BaseException`.

Voici une représentation simplifiée de l'arborescence des exceptions en Python :

```
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
           +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- BlockingIOError
      |    +-- ChildProcessError
      |    +-- ConnectionError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError
      |    +-- IndentationError
      |         +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```

Cette arborescence de classe montre que les exceptions spécifiques héritent d'exceptions plus générales. Par exemple, l'exception `ZeroDivisionError` est une sous-classe de l'exception `ArithmeticError`, qui est elle-même une sous-classe de la classe `Exception`.

Quand nous utilisons un bloc `try`/`except` pour gérer des exceptions, nous pouvons cibler des exceptions spécifiques (par exemple, `ZeroDivisionError`) ou des exceptions plus générales (par exemple, `ArithmeticError` ou `Exception`). Si nous ciblons une exception générale, cela attrapera également toutes ses sous-exceptions. Par exemple, `except Exception:` attrapera toutes les exceptions dérivées de `Exception`, qui incluent la plupart des exceptions qui peuvent être levées par votre code.

## L'instruction `else` dans un bloc d'exception

Nous pouvons ajouter un bloc `else` à un bloc `try`/`except`. Le bloc `else` est une partie où nous pouvons mettre du code qui sera exécuté si le bloc de code dans le `try` s'exécute sans lever d'exception. 

Voici un exemple de la façon dont cela fonctionne :

```python
try:
    # Bloc de code où une exception peut être levée
    x = 1 + 1
except ValueError:
    # Ce bloc de code est exécuté si une exception ValueError est levée
    print("Une exception ValueError a été levée.")
else:
    # Ce bloc de code est exécuté si aucune exception n'est levée dans le bloc try
    print("Aucune exception n'a été levée. x vaut :", x)
```

Dans cet exemple, aucun `ValueError` n'est levé dans le bloc `try`, donc le bloc `else` est exécuté.

Notons que le bloc `else` est exécuté après le bloc `try`, mais avant le bloc `finally`, s'il est présent. Le bloc `finally` est conçu pour contenir du code de nettoyage qui doit être exécuté quelles que soient les exceptions qui pourraient être levées, tandis que le bloc `else` est conçu pour contenir du code qui n'est exécuté que si aucune exception n'est levée.

## L'instruction `finally`

Nous pouvons compléter un bloc d'exception avec l'instruction `finally`, qui est utilisée avec les blocs `try`/`except` pour spécifier un bloc de code à exécuter, peu importe si une exception est levée ou non. Ceci est généralement utilisé pour effectuer des opérations de nettoyage, comme fermer un fichier ou une connexion réseau.

```python
try:
    # Bloc de code où une exception peut être levée
except SomeException:
    # Bloc de code pour gérer l'exception
finally:
    # Bloc de code qui est toujours exécuté, peu importe si une exception est levée ou non
```

# Les modules

Un module est un fichier contenant du code Python qui définit des fonctions, des classes et des variables. Ces modules peuvent être importés dans d'autres scripts Python pour réutiliser leur code et organiser notre programme. Ils constituent la base de la réutilisation du code en Python et nous permettent de structurer notre code de manière plus logique, compréhension et réutilisable.

Nous pouvons faire le parallèle entre les modules et les recettes d'un livre de cuisine. Par exemple, pour préparer la recette des lasagnes, nous utiliserons la recette de la béchamel définie par ailleurs, peut-être même dans un autre livre de recettes, et est utilisée dans plusieurs recettes du livre. De la même manière, un module Python peut contenir des définitions et des instructions qui sont écrites une seule fois et peuvent être utilisées dans de nombreux scripts Python, rendant le code plus lisible, réutilisable et mieux organisé.

Voici comment nous pouvons importer un module en Python :

```python
import math
```

Dans cet exemple, nous importons le module `math`, qui contient des fonctions et des variables utiles pour faire des calculs mathématiques.

Une fois que le module est importé, nous pouvons utiliser ses fonctions et variables comme suit :

```python
import math
print(math.sqrt(16))  # Affiche 4.0
```

## L'instruction import as

L'instruction `import as` en Python est utilisée pour donner un alias, ou un autre nom, à un module lors de son importation. Cela peut être particulièrement utile lorsque le nom du module est long et que nous voulons utiliser un nom plus court pour le référencer dans notre code.

```python
import math as m
```

Dans cet exemple, le module `math` est importé, mais il sera référencé sous le nom `m` dans le reste du code. Cela signifie que nous pouvons accéder aux fonctions et aux variables du module `math` en utilisant le préfixe `m.` :

```python
import math as m
print(m.sqrt(16))  # Affiche 4.0
```

Le fait d'utiliser des alias avec `import as` peut rendre notre code plus court et plus lisible, surtout si nous utilisons fréquemment le module. C'est une pratique courante pour certains modules largement utilisés. Par exemple, la bibliothèque NumPy est généralement importée sous l'alias `np` :

```python
import numpy as np
```

Cela permet de taper `np` au lieu de `numpy` chaque fois que nous voulons accéder aux fonctions ou aux variables de NumPy.

## instruction reload

La primitive `reload` est une fonction intégrée qui nous permet de recharger un module après l'avoir importé. C'est utile lorsque nous avons apporté des modifications à un module et que nous voulons que ces modifications soient prises en compte sans avoir à quitter Python et le relancer. Pour utiliser `reload`, nous devons d'abord importer le module `importlib` :

```python
import importlib
importlib.reload(math)
```

Notons que l'utilisation de `reload` n'est pas courante dans la plupart des scénarios de programmation Python. En général, si nous modifions fréquemment nos modules pendant que nous développez un programme, il peut être plus facile de simplement quitter et relancer Python. Cependant, dans certains cas, comme dans un serveur Web qui reste allumé en permanence, `reload` peut être utile pour appliquer des mises à jour à nos modules.

### L'instruction `from` dans les imports

L'instruction `from` en Python est utilisée dans les importations pour importer des fonctions, classes, ou variables spécifiques d'un module, plutôt que d'importer le module en entier. Par exemple :

```python
from math import sqrt
```

Dans cet exemple, seulement la fonction `sqrt` du module `math` est importée. Nous pouvons maintenant l'utiliser directement sans avoir besoin de préfixer avec le nom du module :

```python
print(sqrt(16))  # Affiche 4.0
```

### L'utilisation de `.` dans les imports

Le caractère `.` est utilisé pour spécifier les sous-modules ou les packages. Considérons un package nommé `mypackage` avec un sous-module `mymodule`. Nous pourrions l'importer de cette façon :

```python
import mypackage.mymodule
```

Et nous pourrions aussi utiliser `from` avec `.` :

```python
from mypackage import mymodule
```

### L'utilisation de `*` dans les imports

L'astérisque `*` en Python est utilisé pour importer toutes les fonctions, classes et variables d'un module. Par exemple :

```python
from math import *
```

Ceci importera toutes les fonctions et variables du module `math`. Nous pouvons alors les utiliser directement sans avoir besoin de préfixer avec le nom du module :

```python
print(sqrt(16))  # Affiche 4.0
print(pi)  # Affiche 3.141592653589793
```

Cependant, utiliser `import *` n'est généralement pas recommandé, car cela peut causer des conflits entre les noms de différents modules, et cela rend notre code moins clair. Il est préférable d'importer seulement ce dont nous avons besoin à l'aide de l'instruction `from`, ou d'importer le module entier et d'utiliser le préfixe du nom de module.

# Les paquets

Les paquets sont un moyen d'organiser les modules de manière hiérarchique. Un paquet est simplement un répertoire qui contient plusieurs modules Python. C'est une façon d'organiser de grands ensembles de code en petits sous-ensembles maniables. 

Chaque paquet en Python est un répertoire qui DOIT contenir un fichier spécial appelé `__init__.py`. Ce fichier peut être vide, mais il doit être présent dans le répertoire. Il indique à l'interpréteur Python que ce répertoire doit être traité comme un paquet Python.

Par exemple, considérons un paquet nommé `monpaquet`, qui contient deux modules : `module1` et `module2`. Sur le disque, cela pourrait ressembler à ceci :

```
monpaquet/
    __init__.py
    module1.py
    module2.py
```

Nous pouvons alors accéder aux modules à l'aide d'importations qualifiées. Par exemple :

```python
import monpaquet.module1
import monpaquet.module2
```

Ou, en utilisant l'instruction `from ... import`, nous pouvons importer des fonctions spécifiques à partir des modules de notre paquet :

```python
from monpaquet.module1 import ma_fonction
```

L'utilisation de paquets permet une organisation logique et une séparation des fonctionnalités, ce qui facilite la gestion de projets de grande envergure.


# Points de suspension dans les classes

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

## Final

À partir de Python 3.8, le module `typing` offre un moyen de déclarer une variable comme étant "finale" ou constante, en utilisant l'annotation `Final`.

Voici comment cela fonctionne :

```python
from typing import Final

CONSTANTE: Final = "Une variable constante"
```

Dans cet exemple, `CONSTANTE` est déclarée comme une variable finale, ce qui signifie que sa valeur est censée ne pas être modifiée après l'affectation initiale.

Il convient de noter que `Final` est une annotation de type et n'impose pas de contrainte d'immuabilité au niveau de l'exécution. Autrement dit, Python lui-même n'empêchera pas la modification d'une variable annotée avec `Final`, mais certains outils de vérification de types comme `mypy` peuvent émettre un avertissement si vous tentez de modifier une telle variable.

Cela dit, utiliser `Final` est une bonne pratique pour communiquer l'intention que certaines variables ne doivent pas être modifiées et pour aider les outils de vérification de type à détecter les erreurs potentielles.

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
```
# Les principaux modules

Voici une liste de quelques-uns des modules Python les plus couramment utilisés, ainsi qu'une brève description de leurs fonctionnalités :

## sys

Contient les informations relatives à l'interpréteur Python et à l'exécution en cours, comme les arguments de la ligne de commande et le chemin de recherche des modules.

## os

Permet d'interagir avec le système d'exploitation, offrant ainsi des fonctionnalités pour naviguer dans le système de fichiers, lancer des processus ou encore lire les variables d'environnement.

## gzip, zipfile

Ces modules permettent de manipuler des fichiers compressés. Avec `gzip` et `zipfile`, vous pouvez lire et écrire des fichiers gzip et zip, respectivement.

## socket

Fournit des fonctionnalités de niveau bas pour gérer les connexions réseau TCP et UDP.

## urllib

Ce module fournit des fonctionnalités pour manipuler des URLs et effectuer des opérations sur le web comme la récupération de pages web via HTTP.

## re

Un module pour travailler avec des expressions régulières, qui sont des séquences de caractères spéciales qui vous aident à trouver, correspondre ou diviser du texte.

## datetime

Il fournit des fonctionnalités pour travailler avec les dates et les heures.

## math

Il comprend des fonctions mathématiques de base, des constantes et des opérations sur les nombres complexes.

## json

Ce module vous permet de convertir des données Python en une chaîne JSON et vice versa.

## collections

Il propose des alternatives aux structures de données intégrées de Python (dict, list, set, et tuple). 

## numpy, pandas, matplotlib

Ces trois modules sont essentiels pour toute personne travaillant avec des données en Python. `numpy` offre des fonctionnalités pour travailler avec des tableaux multidimensionnels, `pandas` est excellent pour manipuler et analyser les données, et `matplotlib` est utilisé pour créer des graphiques et des visualisations.

## requests

C'est une bibliothèque pour envoyer des requêtes HTTP de manière simplifiée. Elle est plus intuitive que urllib et est largement utilisée pour le web scraping, l'API et d'autres opérations HTTP.

# Environnement virtuel

Les environnements virtuels Python permettent de créer des environnements de développement ou d'exécution isolés les uns des autres. En d'autres termes, nous pouvons installer des bibliothèques dans un environnement spécifique sans affecter les autres environnements Python. De plus, nous pouvons précisément lister les dépendances dont notre programme a besoin.

Cela met fin aux problèmes de livraison où des bibliothèques manquent parce que dans un autre projet, nous avons installé une bibliothèque dont nous avions besoin sans nous en rendre compte. Pour installer le paquet nécessaire à la gestion des environnements virtuels, nous utilisons la commande suivante :

    pip3 install virtualenv virtualenvwrapper

Ensuite, nous devons éditer notre fichier `.bashrc` et y ajouter les lignes suivantes :

    # virtualenv
    export WORKON_HOME=$HOME/.virtualenvs
    VIRTUALENVWRAPPER_PYTHON='/usr/bin/python3'
    source /usr/local/bin/virtualenvwrapper.sh

Pour créer un environnement virtuel, nous utilisons la commande `mkvirtualenv` :

    mkvirtualenv my_venv -p python3

Par exemple, pour installer QT6 dans un environnement virtuel, nous utilisons :

    mkvirtualenv qt6_env -p python3
    (qt6_env) michaellaunay@luciole:~$ pip install numpy matplotlib PyQt6
    (qt6_env) michaellaunay@luciole:~$ pip install flake8 pylint

Depuis Python 3.6, il existe un outil, `pipenv`, qui combine `pip` et `virtualenv`.

Pour installer `pipenv`, nous utilisons :

    pip3 install pipenv

Nous pouvons créer un environnement virtuel lors de l'installation d'un module en utilisant `pipenv`. Par exemple :

    cd $MY_WORKING_DIR
    pipenv install pyside6 --python=python3.11

Note : Si Python 3.11 n'est pas installé sur la machine, il ne faut pas oublier de l'installer avec "apt install python3.11".

Pour voir l'environnement virtuel associé au répertoire courant, nous utilisons :

    pipenv --venv

Et pour activer l'environnement virtuel créé, nous utilisons :

    pipenv shell

# Les linters

Un "linter" est un outil qui analyse le code source d'un programme afin de signaler les erreurs, les bugs, les problèmes stylistiques et les constructions suspectes. Le terme provient de "lint", un outil de vérification de code pour le langage C créé dans les années 70. L'idée est de "dépoussiérer" le code, d'où le nom "lint" (peluches/moutons/minous en anglais).

En Python, il existe plusieurs outils de "linting" populaires, comme Pylint, Flake8 et Pyflakes. Ces outils peuvent vérifier votre code pour détecter :

- Des erreurs de syntaxe
- Des problèmes de formatage qui ne sont pas conformes à la norme PEP 8
- Des problèmes de structure de code, comme les classes non utilisées, les variables non utilisées, etc.
- Des problèmes de complexité de code, comme les fonctions ou les méthodes trop longues
- Des problèmes potentiels, comme l'utilisation de types non compatibles, les fonctions avec trop de branches ou de boucles, etc.

L'utilisation d'un linter peut grandement améliorer la qualité de votre code. Il peut vous aider à détecter des bugs ou des problèmes potentiels avant même d'exécuter votre programme. De plus, comme la plupart des linters Python vérifient également la conformité PEP 8, ils peuvent vous aider à rendre votre code plus lisible et plus conforme aux standards de la communauté Python.

# Conformité PEP8

Pour vérifier la conformité de notre code avec la norme PEP 8, nous pouvons utiliser l'outil `flake8`. Il s'agit d'un outil de vérification de la qualité du code Python qui comprend entre autres choses une vérification de la conformité PEP 8.

Dans notre terminal, nous exécutons la commande `flake8`, suivie du nom du fichier que nous souhaitons vérifier. Par exemple :

```bash
flake8 demo.py
```

Si notre fichier `demo.py` contient des violations de la norme PEP 8, `flake8` les affichera dans la console. C'est un excellent moyen pour nous d'assurer que notre code est propre, lisible et conforme aux meilleures pratiques de la communauté Python.

Il est important de noter que `flake8` ne corrige pas les erreurs pour nous, il signale simplement leur présence. C'est à nous de corriger ces problèmes dans notre code.

# Dégogage en ligne

Le débogage en ligne, également appelé débogage interactif ou débogage pas à pas, est une technique utilisée pour identifier et résoudre les problèmes dans le code en exécutant le programme pas à pas et en inspectant les valeurs des variables à différents points d'exécution. Python fournit plusieurs outils pour le débogage en ligne, dont les plus couramment utilisés sont `pdb` (Python Debugger) et les fonctionnalités intégrées aux environnements de développement intégrés (EDI) populaires tels que PyCharm, [[Visual studio code]] et Jupyter Notebook.

Voici comment utiliser `pdb`, l'outil de débogage en ligne intégré à Python :

1. Importons le module `pdb` dans notre code Python :
```python
import pdb
```

2. Ajoutons un point d'arrêt à l'endroit où nous souhaitons commencer le débogage :
```python
pdb.set_trace()
```
Cela place un point d'arrêt dans notre code, ce qui signifie que l'exécution s'arrêtera à cet endroit.

3. Exécutons notre programme. Lorsque l'exécution atteint le point d'arrêt, le débogueur `pdb` sera activé et nous pourrons commencer à inspecter le code.

4. Utilisons les commandes du débogueur pour naviguer et inspecter le code :
   - `n` : Exécute la ligne actuelle et passe à la suivante.
   - `s` : Passe à l'intérieur d'une fonction (si la ligne actuelle est un appel de fonction).
   - `q` : Quitte le débogueur.
   - `p <variable>` : Affiche la valeur d'une variable.
   - `l` : Affiche la portion de code autour de la ligne actuelle.
   - `c` : Exécute le programme jusqu'au prochain point d'arrêt ou jusqu'à la fin.

5. Utilisons ces commandes et explorons l'état de notre programme pour comprendre les problèmes et les erreurs.

Les EDI fournissent souvent des fonctionnalités de débogage plus avancées, comme des points d'arrêt graphiques, la visualisation des variables, le suivi de la pile d'appels, etc. Ces fonctionnalités facilitent le processus de débogage en offrant une interface conviviale et des outils visuels pour inspecter et suivre le code.

Que nous utilisions `pdb` ou les fonctionnalités intégrées d'un EDI, le débogage en ligne est un outil précieux pour comprendre et résoudre les problèmes dans notre code Python. Il nous permet d'examiner l'exécution de notre programme pas à pas et d'identifier les erreurs, les bugs ou les comportements indésirables.

# Liens

Bien sûr, voici une liste de ressources en ligne pour vous aider à approfondir vos compétences en Python :

1. [Python.org](https://docs.python.org/3/tutorial/index.html) : Le tutoriel officiel de Python sur leur site web est un excellent point de départ.

2. [Codecademy](https://www.codecademy.com/learn/learn-python-3) : Ils offrent un cours interactif de Python qui est idéal pour les débutants.

3. [Real Python](https://realpython.com/) : De nombreux articles, tutoriels et vidéos sur des sujets Python spécifiques.

4. [Coursera](https://www.coursera.org/courses?query=python) : Cours en ligne offerts par des universités et des instituts de recherche renommés.

5. [Geeks for Geeks Python](https://www.geeksforgeeks.org/python-programming-language/) : Des tutoriels sur des concepts spécifiques de Python, ainsi que des exercices de programmation.

6. [Learn Python the Hard Way](https://learnpythonthehardway.org/) : Un livre électronique interactif qui vous guide à travers des exercices de codage.

7. [Project Euler](https://projecteuler.net/) : Un ensemble de problèmes de programmation mathématique qui peuvent être résolus avec n'importe quel langage, mais sont utiles pour la pratique du Python.

8. [HackerRank Python](https://www.hackerrank.com/domains/tutorials/10-days-of-python) : Un excellent endroit pour pratiquer la résolution de problèmes avec Python.

9. [LeetCode](https://leetcode.com/) : Une plateforme pour la préparation des entretiens techniques où vous pouvez pratiquer des questions Python.

10. [edX](https://www.edx.org/learn/python) : Des cours en ligne gratuits offerts par des universités du monde entier.

Il est préférable de commencer par des ressources adaptées à notre niveau actuel, puis d'ajouter des défis supplémentaires à mesure que vous progressez.