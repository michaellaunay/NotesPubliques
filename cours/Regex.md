# 1. Histoire des regex

Faisons un saut dans le temps, jusqu’aux **années 1940 et 1950**, période où l’informatique en était à ses balbutiements. Des chercheurs s’intéressaient déjà à la formalisation mathématique de divers concepts, dont l’**intelligence artificielle**. C’est dans ce contexte que naissent les **travaux de Stephen Kleene** (parfois orthographié “Kleen” ou “Kleene”), mathématicien et logicien, qui formule des modèles d’**automates finis** : des machines théoriques capables de reconnaître des langages dits « réguliers ».

Au fil des années 1950, ces travaux se concrétisent et les **modèles de Kleene** sont prolongés par Michael Rabin et Dana Scott. Tous deux reçurent le **prix Turing** en 1976 pour leurs contributions à la théorie des automates et des langages formels.

C’est ensuite l’arrivée de **Ken Thompson**, co-créateur d’Unix, qui propulse réellement les expressions régulières dans l’univers informatique. Durant les années 1960-1970, il intègre dans l’éditeur de texte **QED**, puis dans **ed** (l’un des plus anciens éditeurs sous Unix), une syntaxe permettant de rechercher et de modifier des chaînes de caractères avec la logique formalisée par Kleene. Très vite, cette approche se diffuse dans des commandes phares du monde Unix telles que **grep**, **sed**, **awk**, etc.

# 2. Les expressions régulières de nos jours

Aujourd’hui, les expressions régulières font partie de presque tous les langages de programmation et de nombreux outils. Nous les trouvons dans **Perl**, **Python**, **PHP**, **JavaScript** et bien d’autres environnements, chacun proposant parfois sa propre **“saveur”** (_flavor_) de regex.

On distingue souvent deux grandes familles :

- Les **Basic Regular Expressions (BRE)**, historiquement présentes dans des outils comme _sed_ ou _ed_, où certains caractères spéciaux doivent être précédés d’un antislash (`\`) pour en activer la signification ;
- Les **Extended Regular Expressions (ERE)**, plus flexibles, car elles n’exigent pas toujours d’échappement pour les caractères spéciaux.

En plus de ces deux grandes familles, il existe des variations, que l’on trouve par exemple dans **Perl** (PCRE), où des fonctionnalités avancées permettent d’analyser, d’extraire ou de modifier du texte avec une puissance parfois déroutante.
# 3. Fonctionnement

Entrons dans le concret. Les expressions régulières permettent de **décrire un motif** (ou _pattern_). Ce motif est ensuite recherché dans un texte et, si une correspondance est trouvée, l’outil ou le langage peut :

1. Mettre en évidence la partie correspondante,
2. L’extraire,
3. La remplacer.

Pour pratiquer et s’exercer, nous pouvons utiliser un site comme **[regex101](https://regex101.com)**, qui fournit une interface dédiée pour tester nos regex en temps réel. Nous pouvons également travailler dans un éditeur **Vim** (ou **Neovim**) et utiliser la puissante commande :

```
:%s/MOTIF/REMPLACEMENT/g
```

afin de remplacer toutes les occurrences du motif par notre texte de remplacement dans l’ensemble du fichier.

## 3.1. Quelques caractères spéciaux clés

- **Le point (`.`)** : représente un caractère quelconque (sauf, en général, le caractère de saut de ligne).
- **L’accent circonflexe (`^`)** : désigne le **début** de la ligne.
- **Le symbole dollar (`$`)** : désigne la **fin** de la ligne.
- **Les crochets (`[...]`)** : désignent une **classe de caractères**. Par exemple, `[0-9]` pour tous les chiffres de 0 à 9, `[a-zA-Z]` pour les lettres.
- **L'accent circonflex à l’intérieur des crochets (`[^...]`)** : signifie la **négation** d’une classe de caractères. Par exemple, `[^0-9]` sélectionne tout caractère qui n’est pas un chiffre.
- **Le point d’interrogation (`?`)** : appliqué à un motif, il indique qu’on peut l’avoir **0 ou 1** fois.
- **L’étoile (`*`)** : appliquée à un motif, elle indique **0 à N** répétitions.
- **Le plus (`+`)** : appliqué à un motif, il indique **1 à N** répétitions.
- **Les accolades (`{n,m}`)** : précisent un **intervalle de répétitions**, par exemple `{2,5}` pour dire “entre 2 et 5 fois”.

## 3.2. La capture et la substitution

Grâce aux **parenthèses** (qu’il faut parfois échapper, selon la "saveur" : `\( ... \)` en BRE), nous pouvons **capturer** une partie du motif. Cette portion capturée est ensuite réutilisable dans la section de remplacement via les variables `\1`, `\2`, `\3`, et ainsi de suite, en fonction de l’ordre d’apparition des parenthèses.

Par exemple, pour **permuter deux colonnes** dans un fichier CSV (et en supposant que nos colonnes sont séparées par `;`), nous pouvons faire :

```
:%s/^\([^;]*\);\([^;]*\)/\2;\1/g
```

Ici, `[^;]*` capture une suite de caractères ne contenant pas `;`. La première capture est `\1`, la seconde `\2`, que nous ré-inversons dans la partie de remplacement.

## 3.3. Exemple concret dans Vim

- Recherche d’un simple mot : `/lorem`
- Recherche d’un mot en début de ligne : `/^lorem`
- Recherche d’un mot en fin de ligne : `/lorem$/`
- Remplacer “lorem” par “block” dans tout le fichier :
    
    ```
    :%s/lorem/block/g
    ```
    
- Capturer “lorem” et le doubler (“lorem lorem”) :
    
    ```
    :%s/\(lorem\)/\1 \1/g
    ```
    

Grâce à ces quelques exemples, nous constatons combien la syntaxe peut rapidement nous faire gagner un temps précieux.
# 4. Les regex… et après ?

Une fois que nous commençons à maîtriser ces bases, le champ des possibles s’élargit. Nous pouvons :

- **Nettoyer** des données textuelles : enlever des lignes vides, standardiser des formats de date, valider un format d’adresse e-mail, etc.
- **Transformer** des fichiers volumineux : reconstituer, réorganiser ou enrichir des données (par exemple passer d’un format CSV à du JSON).
- **Analyser** du code source : extraire des imports, repérer des classes ou des fonctions, détecter des syntaxes particulières.
- **Automatiser** des tâches : via des scripts Shell (avec `sed`, `awk`…), des macros dans Vim, ou n’importe quel langage moderne (Python, JavaScript, etc.).

Le plus important pour nous est de **pratiquer**. Plus nous pratiquerons, plus nous comprendrons les regex rapidement, sans avoir à les “déchiffrer” à chaque fois.

# 5. Conclusion

Pour conclure, nous retiendrons que les **expressions régulières** sont nées de l’histoire de la théorie des automates dans les années 1940-1950 et se sont épanouies grâce à Ken Thompson et l’écosystème Unix. Aujourd’hui, elles sont devenues un langage universel : quel que soit l’éditeur de texte ou le langage de programmation, nous utilisons des regex pour rechercher, remplacer ou manipuler des chaînes de caractères.

Nous savons à quel point cela peut être intimidant, surtout lorsqu’on croise des expressions longues et complexes. Cependant, en retenant les principes de base (les caractères spéciaux, la capture, les répétitions, etc.) et en pratiquant de manière régulière, nous pourrons rapidement automatiser des tâches pénibles ou répétitives.
