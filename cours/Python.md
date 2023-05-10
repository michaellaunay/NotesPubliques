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

