---
schema_version: 1
uid: "01M02EX5C8CK4QTX2J6FF9YP6S"
titre: "Systèmes numériques"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - mathematiques
  - numeration
  - histoire-informatique
  - binaire
  - hexadecimal
  - ieee-754
  - bibi-binaire
resume: "Cours complet sur les systèmes de numération : bases positionnelles, conversions, calcul en binaire/octal/hexadécimal, entiers signés, virgule flottante IEEE 754, bases alternatives et système bibi-binaire de Boby Lapointe."
niveau: debutant
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-07-03
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Systèmes numériques et systèmes de numération

> [!abstract] Objectif
> Comprendre comment les nombres et l'information sont représentés dans les machines : bases et systèmes de numération, binaire, octal et hexadécimal, conversions, entiers signés, virgule fixe et flottante (IEEE 754), codage des caractères, erreurs d'arrondi et de dépassement, avec des exercices en Python.

Voir aussi : [[Python]], [[Informatique]], [[Logique]], [[C++]], [[Numpy]].

> [!summary]
> Un **nombre** est une idée mathématique ; son **écriture** dépend d'un système de numération. Le même nombre peut s'écrire `42`, `101010₂`, `52₈`, `2A₁₆` ou encore avec les symboles du système bibi-binaire. Ce cours étudie la représentation des nombres, les conversions entre bases et les représentations réellement utilisées par les ordinateurs.

Ce cours ne traite pas principalement des circuits logiques. Pour l'architecture matérielle et la logique numérique, voir aussi [[Informatique]].

## Objectifs

À la fin du cours, nous devons savoir :

- distinguer un **nombre** de sa **représentation** ;
- comprendre une numération positionnelle de base quelconque ;
- convertir des entiers et des fractions entre bases ;
- calculer en binaire, octal et hexadécimal ;
- comprendre pourquoi l'informatique utilise surtout la base 2 et la base 16 ;
- expliquer les représentations des entiers signés ;
- manipuler le **complément à deux** ;
- comprendre les principes de la virgule fixe et de la virgule flottante ;
- expliquer les principales propriétés de **IEEE 754** ;
- éviter les pièges comme `0.1 + 0.2 != 0.3` en virgule flottante binaire ;
- comparer des bases selon leurs propriétés mathématiques et pratiques ;
- comprendre le système bibi-binaire de Boby Lapointe ;
- reconnaître les représentations qui ne sont **pas** des systèmes de numération : Gray, BCD, ASCII, etc.

# Sommaire

1. Nombre, chiffre, représentation et codage
2. Bref historique des systèmes de numération
3. Numération positionnelle en base b
4. Bases importantes
5. Convertir une base quelconque vers le décimal
6. Convertir un entier décimal vers une base b
7. Conversion directe binaire ↔ octal ↔ hexadécimal
8. Fractions dans une base positionnelle
9. Arithmétique en base quelconque
10. Analyse du système décimal
11. Analyse du système duodécimal
12. Analyse du système sexagésimal
13. Quelle serait la « meilleure » base ?
14. Le binaire en informatique
15. Hexadécimal en pratique
16. Représentation des entiers non signés
17. Représenter les entiers négatifs
18. Débordement signé
19. Opérations bit à bit
20. Virgule fixe
21. Virgule flottante
22. Pourquoi 0,1 est difficile en binaire
23. Comparer correctement des flottants
24. Endianness : ordre des octets
25. BCD — Binary-Coded Decimal
26. Code Gray
27. Base équilibrée et ternaire équilibré
28. Bases négatives
29. Système bibi-binaire de Boby Lapointe
30. Conversion avec Python
31. Représenter un entier dans un nombre précis de bits
32. Inspecter un float
33. Erreurs fréquentes
34. Tableau de référence rapide
35. Travaux pratiques
36. Projet final — Laboratoire de numération
37. Checklist de maîtrise
38. Glossaire
39. Références et approfondissements
40. À retenir

# 1. Nombre, chiffre, représentation et codage

## 1.1 Un nombre n'est pas son écriture

Le nombre quarante-deux peut être écrit de plusieurs façons :

| Système | Écriture |
| --- | ---: |
| décimal | `42` |
| binaire | `101010₂` |
| octal | `52₈` |
| hexadécimal | `2A₁₆` |
| chiffres romains | `XLII` |

Toutes ces écritures désignent le **même nombre**.

Cette distinction est fondamentale en informatique : une machine ne « connaît » pas naturellement le nombre `42` sous la forme des caractères `4` et `2`. Elle manipule une représentation binaire, alors que l'interface utilisateur peut afficher une représentation décimale.

## 1.2 Chiffre

Un **chiffre** est un symbole élémentaire utilisé pour écrire un nombre.

En base 10 :

```text
0 1 2 3 4 5 6 7 8 9
```

En base 2 :

```text
0 1
```

En base 16, la convention informatique habituelle est :

```text
0 1 2 3 4 5 6 7 8 9 A B C D E F
```

avec :

```text
A = 10
B = 11
C = 12
D = 13
E = 14
F = 15
```

## 1.3 Numération et codage

Il faut distinguer :

- **numération** : manière de représenter des nombres ;
- **codage** : manière d'associer des symboles ou états à une information.

Par exemple :

- le binaire est un système de numération ;
- le code Gray est un codage d'entiers ;
- UTF-8 est un codage de caractères ;
- Base64 est un codage de données binaires en caractères ;
- BCD code séparément les chiffres décimaux avec des groupes de bits.

> [!important]
> Le nom « Base64 » est trompeur pour un débutant : **Base64 n'est pas une numération positionnelle utilisée pour calculer**. C'est un encodage de données.

# 2. Bref historique des systèmes de numération

## 2.1 Numérations non positionnelles

Dans un système non positionnel, la valeur d'un symbole dépend peu ou pas de sa position.

Exemple simplifié :

```text
||||| = cinq unités
```

Les chiffres romains combinent plusieurs principes :

```text
VIII = 8
IX   = 9
XLII = 42
```

Ils sont pratiques pour écrire certaines quantités, mais moins adaptés aux calculs systématiques que les numérations positionnelles.

## 2.2 Système sexagésimal babylonien

Les mathématiques mésopotamiennes ont utilisé une numération sexagésimale, c'est-à-dire de base 60.

Nous en conservons des traces dans :

- 60 secondes par minute ;
- 60 minutes par heure ;
- la division du cercle en 360 degrés.

La base 60 est particulièrement divisible :

```text
60 = 2² × 3 × 5
```

Elle possède donc de nombreux diviseurs.

## 2.3 Le rôle du zéro

Une numération positionnelle efficace nécessite un moyen de distinguer une position vide.

Par exemple :

```text
205
```

n'est pas la même chose que :

```text
25
```

Le zéro joue ici un rôle de **chiffre de position**, en plus de représenter le nombre zéro.

## 2.4 Diffusion de la numération indo-arabe

La numération décimale positionnelle avec zéro s'est développée en Inde puis a été transmise et enrichie dans le monde arabo-musulman avant de se diffuser progressivement en Europe.

Elle est aujourd'hui dominante pour les usages humains courants.

# 3. Numération positionnelle en base b

## 3.1 Principe général

Dans une base `b`, on dispose normalement de `b` valeurs de chiffres :

```text
0, 1, ..., b - 1
```

La valeur d'une écriture :

```text
aₙaₙ₋₁...a₂a₁a₀
```

est :

```text
aₙ × bⁿ + aₙ₋₁ × bⁿ⁻¹ + ... + a₂ × b² + a₁ × b + a₀
```

avec :

```text
0 ≤ aᵢ < b
```

## 3.2 Exemple en base 10

```text
538₁₀
```

signifie :

```text
5 × 10² + 3 × 10¹ + 8 × 10⁰
= 500 + 30 + 8
= 538
```

## 3.3 Exemple en base 2

```text
101101₂
```

signifie :

```text
1 × 2⁵
+ 0 × 2⁴
+ 1 × 2³
+ 1 × 2²
+ 0 × 2¹
+ 1 × 2⁰

= 32 + 8 + 4 + 1
= 45₁₀
```

## 3.4 Exemple en base 16

```text
2AF₁₆
```

signifie :

```text
2 × 16² + 10 × 16 + 15
= 512 + 160 + 15
= 687₁₀
```

## 3.5 Noter explicitement la base

Lorsqu'une écriture est ambiguë, on note la base en indice :

```text
10₂  = 2₁₀
10₈  = 8₁₀
10₁₀ = 10₁₀
10₁₆ = 16₁₀
```

Ainsi, l'écriture `10` signifie toujours « une unité de la base et zéro unité ».

# 4. Bases importantes

## 4.1 Base 10 — décimal

Chiffres :

```text
0 1 2 3 4 5 6 7 8 9
```

Usage : presque tous les usages humains quotidiens.

Décomposition :

```text
10 = 2 × 5
```

Diviseurs non triviaux : 2 et 5.

## 4.2 Base 2 — binaire

Chiffres :

```text
0 1
```

Avantages techniques : deux états sont relativement faciles à distinguer physiquement.

Exemples d'états pouvant représenter un bit :

- tension basse / tension haute ;
- charge / absence de charge ;
- aimantation dans deux directions ;
- transistor conducteur / non conducteur.

## 4.3 Base 8 — octal

Chiffres :

```text
0 1 2 3 4 5 6 7
```

Un chiffre octal correspond exactement à **3 bits** :

```text
0₈ = 000₂
1₈ = 001₂
...
7₈ = 111₂
```

L'octal reste visible dans Unix, notamment pour les permissions :

```bash
chmod 755 script.sh
```

## 4.4 Base 16 — hexadécimal

Chiffres :

```text
0 1 2 3 4 5 6 7 8 9 A B C D E F
```

Un chiffre hexadécimal représente exactement **4 bits**.

```text
A₁₆ = 1010₂
F₁₆ = 1111₂
```

L'hexadécimal est extrêmement pratique pour condenser du binaire :

```text
1110101011011110₂
= EADE₁₆
```

## 4.5 Base 12 — duodécimal

La base 12 utilise douze valeurs de chiffres.

```text
12 = 2² × 3
```

Elle est divisible par :

```text
2, 3, 4, 6
```

Ce qui facilite des fractions courantes comme :

```text
1/2, 1/3, 1/4, 1/6
```

Il faut cependant deux symboles supplémentaires au-delà de 0–9.

On rencontre selon les communautés :

- `A` et `B` ;
- `X` et `E` ;
- les chiffres duodécimaux dédiés `↊` et `↋`.

## 4.6 Base 60 — sexagésimal

```text
60 = 2² × 3 × 5
```

Elle possède de nombreux diviseurs :

```text
1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30, 60
```

Avantage : de nombreuses fractions ont une écriture finie.

Inconvénient : soixante valeurs de chiffres seraient beaucoup à mémoriser dans une numération moderne explicite.

# 5. Convertir une base quelconque vers le décimal

## 5.1 Méthode par puissances

Pour :

```text
110101₂
```

on calcule :

```text
1 × 2⁵ + 1 × 2⁴ + 0 × 2³ + 1 × 2² + 0 × 2¹ + 1 × 2⁰
= 32 + 16 + 4 + 1
= 53
```

## 5.2 Méthode de Horner

On peut éviter de calculer explicitement toutes les puissances.

Pour `2AF₁₆` :

```text
((2 × 16) + 10) × 16 + 15
= 42 × 16 + 15
= 687
```

Algorithme général :

```text
valeur = 0
pour chaque chiffre :
    valeur = valeur × base + chiffre
```

C'est une excellente méthode pour programmer un convertisseur.

# 6. Convertir un entier décimal vers une base b

## 6.1 Divisions successives

Pour convertir `53₁₀` en base 2 :

| Division | Quotient | Reste |
| --- | ---: | ---: |
| 53 / 2 | 26 | 1 |
| 26 / 2 | 13 | 0 |
| 13 / 2 | 6 | 1 |
| 6 / 2 | 3 | 0 |
| 3 / 2 | 1 | 1 |
| 1 / 2 | 0 | 1 |

On lit les restes **de bas en haut** :

```text
53₁₀ = 110101₂
```

## 6.2 Exemple en hexadécimal

Convertissons `687` en base 16 :

```text
687 = 42 × 16 + 15
42  = 2 × 16 + 10
2   = 0 × 16 + 2
```

Restes :

```text
2, 10, 15
```

donc :

```text
687₁₀ = 2AF₁₆
```

# 7. Conversion directe binaire ↔ octal ↔ hexadécimal

## 7.1 Binaire vers hexadécimal

On regroupe les bits par 4 en partant de la droite.

```text
110101111011₂
```

devient :

```text
1101 0111 1011
 D    7    B
```

Donc :

```text
110101111011₂ = D7B₁₆
```

## 7.2 Hexadécimal vers binaire

Chaque chiffre devient 4 bits :

```text
3A7₁₆
```

```text
3    A    7
0011 1010 0111
```

Donc :

```text
3A7₁₆ = 001110100111₂
```

## 7.3 Binaire vers octal

On groupe par 3 :

```text
110101111₂
```

```text
110 101 111
 6   5   7
```

Donc :

```text
110101111₂ = 657₈
```

# 8. Fractions dans une base positionnelle

## 8.1 Positions après la virgule

Après la virgule, les puissances deviennent négatives.

En base 10 :

```text
12,34
= 1 × 10¹
+ 2 × 10⁰
+ 3 × 10⁻¹
+ 4 × 10⁻²
```

En binaire :

```text
101,101₂
```

vaut :

```text
1 × 2²
+ 0 × 2¹
+ 1 × 2⁰
+ 1 × 2⁻¹
+ 0 × 2⁻²
+ 1 × 2⁻³

= 4 + 1 + 1/2 + 1/8
= 5,625₁₀
```

## 8.2 Convertir une fraction décimale vers une base

Pour la partie fractionnaire, on utilise les **multiplications successives**.

Convertissons `0,625₁₀` en binaire :

```text
0,625 × 2 = 1,25  → chiffre 1
0,25  × 2 = 0,5   → chiffre 0
0,5   × 2 = 1,0   → chiffre 1
```

Donc :

```text
0,625₁₀ = 0,101₂
```

## 8.3 Pourquoi certaines fractions sont périodiques

En base 10 :

```text
1/3 = 0,333333...
```

En base 2 :

```text
1/10₁₀ = 0,0001100110011...₂
```

Une fraction réduite `p/q` possède une écriture **finie** en base `b` si les facteurs premiers de `q` sont tous également des facteurs de la base `b`.

### En base 10

```text
10 = 2 × 5
```

Donc les dénominateurs composés uniquement de 2 et 5 peuvent avoir une représentation finie.

```text
1/8  = 0,125
1/20 = 0,05
```

mais :

```text
1/3 = 0,333...
```

### En base 2

La seule composante première de la base est 2.

Donc :

```text
1/2, 1/4, 1/8, ...
```

sont finis, mais pas :

```text
1/5
1/10
1/3
```

Cette propriété explique une grande partie des surprises des nombres flottants binaires.

# 9. Arithmétique en base quelconque

## 9.1 Addition binaire

Règles de base :

```text
0 + 0 = 0
0 + 1 = 1
1 + 0 = 1
1 + 1 = 10₂
1 + 1 + 1 = 11₂
```

Exemple :

```text
   101101
 + 011011
 --------
  1001000
```

Vérification :

```text
45 + 27 = 72
1001000₂ = 72₁₀
```

## 9.2 Soustraction

En base 2 :

```text
10₂ - 1₂ = 1₂
```

Comme en décimal, on peut emprunter une unité de la position supérieure.

## 9.3 Multiplication

Multiplier par `10₂` revient à décaler d'une position vers la gauche :

```text
1011₂ × 10₂ = 10110₂
```

Cela correspond à une multiplication par 2.

De manière générale :

```text
x << n
```

correspond mathématiquement à :

```text
x × 2ⁿ
```

si aucune limitation de taille ou de signe n'intervient.

## 9.4 Attention aux optimisations simplistes

En code moderne, il ne faut pas remplacer systématiquement :

```text
x * 8
```

par :

```text
x << 3
```

Les compilateurs savent généralement optimiser ce type d'expression. La forme la plus claire est souvent préférable.

# 10. Analyse du système décimal

## 10.1 Pourquoi la base 10 ?

L'explication courante est anthropologique : les humains disposent de dix doigts et ont naturellement utilisé leurs mains pour compter.

Il ne s'agit pas d'une preuve qu'une base 10 serait mathématiquement optimale.

## 10.2 Divisibilité

```text
10 = 2 × 5
```

Les fractions liées à 2 ou 5 s'écrivent bien :

```text
1/2  = 0,5
1/4  = 0,25
1/5  = 0,2
1/10 = 0,1
```

mais :

```text
1/3 = 0,333...
1/6 = 0,1666...
```

## 10.3 L'avantage majeur du décimal

Aujourd'hui, son avantage principal est surtout **l'effet de réseau** :

- apprentissage universel ;
- instruments ;
- normes ;
- monnaie ;
- documentation ;
- interfaces humaines.

Changer de base à l'échelle d'une société aurait un coût considérable.

# 11. Analyse du système duodécimal

## 11.1 Diviseurs

```text
12 = 2² × 3
```

Diviseurs :

```text
1, 2, 3, 4, 6, 12
```

## 11.2 Fractions courantes

En base 12 :

```text
1/2 = 0,6₁₂
1/3 = 0,4₁₂
1/4 = 0,3₁₂
1/6 = 0,2₁₂
```

Ces écritures sont très simples.

## 11.3 Compter 12 avec une main

Une méthode traditionnelle consiste à utiliser le pouce pour pointer les trois phalanges des quatre autres doigts :

```text
4 doigts × 3 phalanges = 12
```

Avec l'autre main pour compter les douzaines, on peut atteindre 60.

# 12. Analyse du système sexagésimal

## 12.1 Richesse des diviseurs

```text
60 = 2² × 3 × 5
```

Cette factorisation rend beaucoup de fractions usuelles simples.

## 12.2 Héritage actuel

Temps :

```text
1 h = 60 min
1 min = 60 s
```

Angles :

```text
1° = 60′
1′ = 60″
```

## 12.3 Pourquoi ne pas tout écrire en base 60 ?

Une base élevée réduit le nombre de positions nécessaires mais nécessite davantage de symboles élémentaires.

C'est un compromis général :

- petite base → peu de chiffres, représentations longues ;
- grande base → beaucoup de chiffres, représentations courtes.

# 13. Quelle serait la « meilleure » base ?

La question n'a pas de réponse universelle.

## 13.1 Critères possibles

On peut comparer :

- quantité de chiffres à mémoriser ;
- longueur moyenne des représentations ;
- divisibilité ;
- simplicité des tables de calcul ;
- facilité de conversion ;
- adaptation au matériel ;
- coût social du changement ;
- compatibilité avec les systèmes existants.

## 13.2 Base 2

Avantages :

- matériel simple ;
- table d'addition minimale ;
- lien direct avec la logique booléenne.

Inconvénient :

- écritures longues pour l'humain.

## 13.3 Base 16

Avantages :

- très compacte ;
- conversion parfaite avec le binaire par groupes de quatre bits.

Inconvénient :

- seize chiffres à mémoriser ;
- fractions usuelles liées à 3 ou 5 pas particulièrement avantagées.

## 13.4 Base 12

Bon compromis pour certains calculs fractionnaires humains.

## 13.5 Économie de radix

Une mesure théorique consiste à considérer le compromis entre :

- nombre de symboles de la base ;
- nombre de positions nécessaires.

Selon le critère exact choisi, des bases proches de `e ≈ 2,718` apparaissent intéressantes, ce qui explique l'intérêt théorique des bases 2 et 3.

Cela ne suffit toutefois pas à désigner une base universellement « meilleure ».

# 14. Le binaire en informatique

## 14.1 Bit

Un **bit** est une information pouvant prendre deux états :

```text
0 ou 1
```

## 14.2 Octet

Un octet contient 8 bits :

```text
1 octet = 8 bits
```

Il peut représenter :

```text
2⁸ = 256
```

configurations différentes.

Pour un entier non signé sur 8 bits :

```text
0 à 255
```

## 14.3 Nibble

Un groupe de 4 bits est parfois appelé **nibble**.

Il correspond exactement à un chiffre hexadécimal :

```text
0000 → 0
...
1111 → F
```

## 14.4 Multiples et préfixes

Il faut distinguer :

### Préfixes décimaux SI

```text
1 kB = 1000 octets
1 MB = 1000² octets
1 GB = 1000³ octets
```

### Préfixes binaires IEC

```text
1 KiB = 1024 octets
1 MiB = 1024² octets
1 GiB = 1024³ octets
```

> [!important]
> `MB` et `MiB` ne sont pas synonymes.

# 15. Hexadécimal en pratique

## 15.1 Préfixe `0x`

De nombreux langages utilisent :

```text
0x2A
```

pour représenter :

```text
42₁₀
```

Python :

```python
n = 0x2A
print(n)      # 42
print(hex(n)) # 0x2a
```

## 15.2 Couleurs Web

Une couleur RGB peut être écrite :

```text
#RRGGBB
```

Par exemple :

```text
#FF8000
```

correspond à :

```text
R = 255
G = 128
B = 0
```

## 15.3 Dumps mémoire

Les adresses et les octets sont souvent affichés en hexadécimal car la conversion depuis le binaire est immédiate.

```text
00000000  48 65 6c 6c 6f
```

## 15.4 UUID et empreintes

Les UUID, hash et identifiants binaires sont souvent rendus sous forme hexadécimale, car elle est compacte et facilement convertible.

# 16. Représentation des entiers non signés

Avec `n` bits, un entier non signé peut représenter :

```text
0 à 2ⁿ - 1
```

Exemples :

| Bits | Minimum | Maximum |
| ---: | ---: | ---: |
| 8 | 0 | 255 |
| 16 | 0 | 65 535 |
| 32 | 0 | 4 294 967 295 |
| 64 | 0 | 18 446 744 073 709 551 615 |

## 16.1 Débordement

Sur une largeur fixe de 8 bits :

```text
255 = 11111111₂
```

Ajouter 1 nécessiterait :

```text
1 00000000₂
```

soit 9 bits.

Le comportement réel dépend du langage et du type utilisé :

- retour modulo `2ⁿ` dans certains types non signés ;
- exception dans certains environnements ;
- entier de taille arbitraire dans d'autres, par exemple l'entier Python usuel.

# 17. Représenter les entiers négatifs

Plusieurs conventions ont existé.

## 17.1 Signe + valeur absolue

Un bit indique le signe, les autres la valeur.

Sur 8 bits :

```text
+5 = 00000101
-5 = 10000101
```

Problème : deux zéros.

```text
+0 = 00000000
-0 = 10000000
```

## 17.2 Complément à un

On inverse tous les bits.

```text
+5 = 00000101
-5 = 11111010
```

Il existe encore deux représentations de zéro.

## 17.3 Complément à deux

C'est la représentation dominante des entiers signés dans les architectures informatiques modernes.

Pour obtenir `-x` sur `n` bits :

1. écrire `x` ;
2. inverser tous les bits ;
3. ajouter 1.

Exemple `-5` sur 8 bits :

```text
+5            00000101
inversion     11111010
+ 1           11111011
```

Donc :

```text
-5 = 11111011₂
```

## 17.4 Intervalle

Sur `n` bits :

```text
-2ⁿ⁻¹ à 2ⁿ⁻¹ - 1
```

Sur 8 bits :

```text
-128 à 127
```

## 17.5 Pourquoi le complément à deux est pratique

La même addition binaire peut servir pour les nombres positifs et négatifs.

Exemple sur 8 bits :

```text
  00000101   +5
+ 11111011   -5
----------
1 00000000
```

En ignorant le report hors des 8 bits :

```text
00000000
```

# 18. Débordement signé

Un dépassement de plage n'est pas la même chose qu'un simple report binaire.

Sur 8 bits signés :

```text
127 = 01111111
```

Ajoutons 1 :

```text
01111111
00000001
--------
10000000
```

Le motif `10000000` représente `-128` en complément à deux.

Dans de nombreux langages bas niveau, il faut être très attentif à la sémantique exacte des dépassements signés ou non signés.

# 19. Opérations bit à bit

## 19.1 AND

```text
1 AND 1 = 1
sinon      0
```

```text
1010
1100
---- AND
1000
```

## 19.2 OR

```text
1010
1100
---- OR
1110
```

## 19.3 XOR

```text
1010
1100
---- XOR
0110
```

## 19.4 NOT

Sur une largeur déterminée :

```text
NOT 1010 = 0101
```

## 19.5 Masques

Tester un bit :

```python
READ = 0b001
WRITE = 0b010
EXECUTE = 0b100

permissions = READ | WRITE

if permissions & WRITE:
    print("écriture autorisée")
```

Les permissions Unix octales reposent sur cette logique :

```text
r = 4 = 100₂
w = 2 = 010₂
x = 1 = 001₂
```

Donc :

```text
rwx = 111₂ = 7₈
r-x = 101₂ = 5₈
```

# 20. Virgule fixe

Une représentation en **virgule fixe** décide à l'avance combien de bits sont consacrés à la partie fractionnaire.

Exemple conceptuel avec 4 bits fractionnaires :

```text
01011010
```

peut être interprété comme :

```text
0101.1010₂
```

soit :

```text
5 + 1/2 + 1/8
= 5,625
```

## Avantages

- comportement déterministe ;
- opérations entières rapides ;
- utile en embarqué, DSP, finance selon les contraintes.

## Inconvénients

- plage et précision doivent être choisies à l'avance ;
- risque de débordement ;
- gestion manuelle de l'échelle.

# 21. Virgule flottante

## 21.1 Idée scientifique

L'écriture scientifique décimale :

```text
6,022 × 10²³
```

sépare :

- un coefficient ;
- un exposant.

La virgule flottante binaire suit une idée comparable, avec des règles précises.

## 21.2 IEEE 754

La norme de référence courante reste **IEEE 754-2019**, standard actif pour l'arithmétique en virgule flottante.

Elle définit notamment :

- des formats binaires et décimaux ;
- les opérations ;
- les modes d'arrondi ;
- les infinis ;
- les NaN ;
- les exceptions et cas spéciaux.

## 21.3 binary32

Le format communément appelé `float` IEEE 754 binaire simple précision utilise 32 bits :

```text
1 bit  : signe
8 bits : exposant
23 bits: fraction
```

## 21.4 binary64

Le format binaire double précision utilise 64 bits :

```text
1 bit  : signe
11 bits: exposant
52 bits: fraction
```

Python utilise normalement ce format pour son type `float` sur les plateformes usuelles.

## 21.5 Forme normale

Pour un nombre normal fini :

```text
(-1)^s × 1.fraction × 2^exposant
```

avec un biais appliqué au champ exposant.

## 21.6 Valeurs spéciales

IEEE 754 prévoit notamment :

```text
+∞
-∞
NaN
+0
-0
```

ainsi que des nombres **subnormaux** proches de zéro.

# 22. Pourquoi 0,1 est difficile en binaire

En décimal :

```text
0,1 = 1/10
```

Or :

```text
10 = 2 × 5
```

En base 2, le facteur 5 ne peut pas être absorbé par une puissance de 2.

La représentation de `0,1` est donc périodique en binaire :

```text
0,00011001100110011...₂
```

Elle doit être arrondie dans un nombre fini de bits.

C'est pourquoi :

```python
0.1 + 0.2 == 0.3
```

renvoie généralement :

```text
False
```

et :

```python
print(f"{0.1 + 0.2:.17f}")
```

montre une valeur proche de :

```text
0.30000000000000004
```

> [!important]
> Ce n'est pas un « bug de Python ». C'est une conséquence de la représentation flottante binaire finie.

# 23. Comparer correctement des flottants

Pour des calculs numériques, on compare souvent avec une tolérance.

```python
import math

assert math.isclose(0.1 + 0.2, 0.3)
```

Selon le domaine, on choisit :

- tolérance absolue ;
- tolérance relative ;
- Decimal ;
- Fraction ;
- entier mis à l'échelle.

## 23.1 Argent

Éviter de supposer qu'un `float` représente exactement des centimes.

Solutions possibles :

```python
from decimal import Decimal

prix = Decimal("19.99")
tva = Decimal("0.20")
```

ou utiliser des entiers :

```text
1999 centimes
```

# 24. Endianness : ordre des octets

L'endianness ne change pas la valeur mathématique, mais l'ordre des octets en mémoire ou dans une représentation sérialisée.

Prenons :

```text
0x12345678
```

Sur quatre octets :

```text
12 34 56 78
```

## Big-endian

Octet le plus significatif d'abord :

```text
12 34 56 78
```

## Little-endian

Octet le moins significatif d'abord :

```text
78 56 34 12
```

> [!note]
> L'endianness concerne l'ordre des **octets**, pas le fait qu'un nombre serait « écrit à l'envers » dans sa valeur mathématique.

# 25. BCD — Binary-Coded Decimal

Le **BCD** code chaque chiffre décimal avec 4 bits.

Exemple :

```text
59₁₀
```

BCD :

```text
5    9
0101 1001
```

Ce n'est pas la représentation binaire habituelle de 59 :

```text
59₁₀ = 00111011₂
```

## Pourquoi utiliser BCD ?

- conversion décimale facile ;
- affichage ;
- domaines où le décimal exact est important ;
- certains matériels et protocoles historiques.

## Coût

Sur 4 bits, six motifs sont inutilisés pour un chiffre décimal :

```text
1010 à 1111
```

Le BCD est donc moins compact que le binaire pur.

# 26. Code Gray

Le code Gray est un **codage**, pas une numération positionnelle usuelle.

Deux valeurs successives diffèrent d'un seul bit.

Exemple 3 bits :

| Décimal | Binaire | Gray |
| ---: | ---: | ---: |
| 0 | 000 | 000 |
| 1 | 001 | 001 |
| 2 | 010 | 011 |
| 3 | 011 | 010 |
| 4 | 100 | 110 |
| 5 | 101 | 111 |
| 6 | 110 | 101 |
| 7 | 111 | 100 |

Cela peut être utile pour des encodeurs et systèmes où plusieurs bits ne doivent pas changer simultanément entre deux états adjacents.

# 27. Base équilibrée et ternaire équilibré

Une base peut utiliser des chiffres négatifs.

Le **ternaire équilibré** utilise par exemple :

```text
-1, 0, +1
```

pour les coefficients de puissances de 3.

On note parfois :

```text
T = -1
0 = 0
1 = +1
```

Le nombre `2` peut alors s'écrire :

```text
1T₃
```

car :

```text
1 × 3 + (-1) = 2
```

Le ternaire équilibré est intéressant théoriquement et a eu des applications historiques, notamment dans des calculateurs ternaires.

# 28. Bases négatives

Il est possible de construire des systèmes positionnels avec une base négative.

En base `-2`, appelée **négabinaire**, on peut représenter des nombres positifs et négatifs sans bit de signe séparé.

Exemple :

```text
110₋₂
```

vaut :

```text
1 × (-2)² + 1 × (-2)¹ + 0
= 4 - 2
= 2
```

Ces bases sont surtout intéressantes pour comprendre que les conventions usuelles ne sont pas les seules possibles.

# 29. Système bibi-binaire de Boby Lapointe

## 29.1 Origine

Robert « Boby » Lapointe (1922–1972), auteur-compositeur-interprète français passionné de mathématiques, a conçu en 1968 un procédé de représentation graphique et phonétique des chiffres hexadécimaux.

Il a déposé le brevet français **n° 1 569 028**, intitulé *Procédé de codification de l'information*, le 28 mars 1968. Le brevet a été délivré en 1969.

Le système est généralement appelé :

- **système bibi-binaire** ;
- **système Bibi**.

Ce n'est pas une nouvelle base au sens mathématique : il s'agit d'une **notation originale de la base 16**, intimement liée au binaire.

## 29.2 Pourquoi « bibi-binaire » ?

Le jeu de mots repose sur :

```text
16 = 2⁴ = 2^(2²)
```

Boby Lapointe joue sur l'idée de « binaire », « bi-binaire », puis « bibi-binaire ».

## 29.3 Construction phonétique

On découpe les quatre bits d'un chiffre hexadécimal en deux groupes de deux bits.

### Deux bits de poids fort → consonne

| Bits | Consonne |
| --- | --- |
| `00` | H |
| `01` | B |
| `10` | K |
| `11` | D |

### Deux bits de poids faible → voyelle

| Bits | Voyelle |
| --- | --- |
| `00` | O |
| `01` | A |
| `10` | E |
| `11` | I |

Chaque chiffre devient donc une syllabe consonne + voyelle.

## 29.4 Table correcte des 16 chiffres

| Décimal | Hex | Binaire | Bibi |
| ---: | ---: | ---: | --- |
| 0 | 0 | `0000` | HO |
| 1 | 1 | `0001` | HA |
| 2 | 2 | `0010` | HE |
| 3 | 3 | `0011` | HI |
| 4 | 4 | `0100` | BO |
| 5 | 5 | `0101` | BA |
| 6 | 6 | `0110` | BE |
| 7 | 7 | `0111` | BI |
| 8 | 8 | `1000` | KO |
| 9 | 9 | `1001` | KA |
| 10 | A | `1010` | KE |
| 11 | B | `1011` | KI |
| 12 | C | `1100` | DO |
| 13 | D | `1101` | DA |
| 14 | E | `1110` | DE |
| 15 | F | `1111` | DI |

> [!warning]
> Une ancienne version de ce cours associait incorrectement `A→DO`, `B→DA`, etc. La table correcte est bien `A→KE`, `B→KI`, `C→DO`, `D→DA`, `E→DE`, `F→DI`.

## 29.5 Exemple : 71 décimal

Convertissons :

```text
71₁₀ = 47₁₆
```

Puis :

```text
4₁₆ → BO
7₁₆ → BI
```

Donc :

```text
71₁₀ = 47₁₆ = BOBI
```

et **pas** `BIBI`.

`BIBI` correspond à :

```text
77₁₆ = 119₁₀
```

## 29.6 Exemple : 2000 décimal

```text
2000₁₀ = 7D0₁₆
```

Donc :

```text
7 → BI
D → DA
0 → HO
```

Lecture :

```text
BIDAHO
```

## 29.7 Représentation graphique

Boby Lapointe a également conçu seize glyphes dont la forme dérive des quatre bits.

[[SystemeNumériqueBibiBinaire_processed.png]]

Les symboles ont donc un lien visuel avec la représentation binaire, et ne sont pas de simples chiffres arbitraires.

## 29.8 Ce que le Bibi n'est pas

Le système Bibi n'est pas :

- un chiffrement ;
- un système cryptographique ;
- une compression ;
- un remplacement moderne des formats binaires machine.

C'est une représentation phonétique et graphique originale des chiffres hexadécimaux.

## 29.9 Intérêt pédagogique

Le Bibi permet de montrer très clairement le lien :

```text
4 bits
  ↓
1 chiffre hexadécimal
  ↓
1 syllabe Bibi
```

C'est donc un outil ludique pour enseigner :

- le binaire ;
- l'hexadécimal ;
- la notion de codage ;
- la distinction valeur/représentation.

# 30. Conversion avec Python

## 30.1 Fonctions intégrées

```python
n = 42

print(bin(n))  # 0b101010
print(oct(n))  # 0o52
print(hex(n))  # 0x2a
```

## 30.2 Parser une chaîne

```python
print(int("101010", 2))  # 42
print(int("52", 8))      # 42
print(int("2A", 16))     # 42
```

Python accepte les bases 2 à 36 avec `int(texte, base)`.

## 30.3 Formatage

```python
n = 42

print(f"{n:b}")  # 101010
print(f"{n:o}")  # 52
print(f"{n:x}")  # 2a
print(f"{n:X}")  # 2A
```

Avec padding :

```python
n = 42
print(f"{n:08b}")
```

Résultat :

```text
00101010
```

## 30.4 Convertisseur générique

```python
DIGITS = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"


def to_base(n: int, base: int) -> str:
    if not 2 <= base <= len(DIGITS):
        raise ValueError("base non supportée")

    if n == 0:
        return "0"

    sign = "-" if n < 0 else ""
    n = abs(n)
    out: list[str] = []

    while n:
        n, remainder = divmod(n, base)
        out.append(DIGITS[remainder])

    return sign + "".join(reversed(out))


print(to_base(687, 16))
```

Résultat :

```text
2AF
```

## 30.5 Convertisseur Bibi

```python
BIBI = [
    "HO", "HA", "HE", "HI",
    "BO", "BA", "BE", "BI",
    "KO", "KA", "KE", "KI",
    "DO", "DA", "DE", "DI",
]


def to_bibi(n: int) -> str:
    if n < 0:
        raise ValueError("exemple limité aux entiers positifs")

    hexa = format(n, "X")
    return "".join(BIBI[int(ch, 16)] for ch in hexa)


print(to_bibi(71))    # BOBI
print(to_bibi(2000))  # BIDAHO
```

# 31. Représenter un entier dans un nombre précis de bits

## 31.1 Non signé

```python
n = 13
print(f"{n:08b}")
```

```text
00001101
```

## 31.2 Complément à deux

Pour simuler une représentation sur `bits` bits :

```python
def twos_complement(n: int, bits: int) -> str:
    mask = (1 << bits) - 1
    return format(n & mask, f"0{bits}b")


print(twos_complement(-5, 8))
```

```text
11111011
```

# 32. Inspecter un float

Python permet d'observer la valeur flottante de différentes façons.

```python
x = 0.1

print(x.hex())
print(x.as_integer_ratio())
```

`as_integer_ratio()` montre la fraction rationnelle exacte réellement représentée par le `float`.

Pour comprendre le motif binaire brut :

```python
import struct

x = 0.1
raw = struct.pack(">d", x)
print(raw.hex())
```

La notation `>d` signifie ici :

- `>` : big-endian pour la sérialisation ;
- `d` : nombre flottant double précision.

# 33. Erreurs fréquentes

## 33.1 Confondre nombre et chaîne de caractères

```text
"42"
```

est une chaîne composée des caractères `'4'` et `'2'`.

Le nombre :

```text
42
```

est une valeur numérique.

## 33.2 Penser que les ordinateurs « stockent les nombres en hexadécimal »

Un ordinateur stocke généralement des **bits**. L'hexadécimal est surtout une notation humaine compacte de ces bits.

## 33.3 Penser qu'un octet vaut toujours un caractère

Faux : UTF-8 peut utiliser plusieurs octets pour un caractère Unicode.

Voir [[Informatique]].

## 33.4 Confondre bit et octet

```text
Mb = mégabit
MB = mégaoctet / megabyte
```

En première approximation :

```text
1 MB = 8 Mb
```

sans tenir compte ici des différences entre préfixes décimaux et binaires.

## 33.5 Croire que `0.1` est toujours exact

En virgule flottante binaire, non.

## 33.6 Dire qu'une base est meilleure sans préciser le critère

La divisibilité, la compacité, le matériel, l'apprentissage et les conventions sociales ne donnent pas toujours le même optimum.

## 33.7 Confondre bibi-binaire et base 4

Le bibi-binaire représente des chiffres **hexadécimaux**. Il tire sa phonétique de paires de bits mais reste une notation de base 16.

# 34. Tableau de référence rapide

| Décimal | Binaire | Octal | Hex | Bibi |
| ---: | ---: | ---: | ---: | --- |
| 0 | 0000 | 0 | 0 | HO |
| 1 | 0001 | 1 | 1 | HA |
| 2 | 0010 | 2 | 2 | HE |
| 3 | 0011 | 3 | 3 | HI |
| 4 | 0100 | 4 | 4 | BO |
| 5 | 0101 | 5 | 5 | BA |
| 6 | 0110 | 6 | 6 | BE |
| 7 | 0111 | 7 | 7 | BI |
| 8 | 1000 | 10 | 8 | KO |
| 9 | 1001 | 11 | 9 | KA |
| 10 | 1010 | 12 | A | KE |
| 11 | 1011 | 13 | B | KI |
| 12 | 1100 | 14 | C | DO |
| 13 | 1101 | 15 | D | DA |
| 14 | 1110 | 16 | E | DE |
| 15 | 1111 | 17 | F | DI |

# 35. Travaux pratiques

## TP 1 — Lire une représentation positionnelle

Convertir vers le décimal :

```text
101101₂
725₈
3AF₁₆
10201₃
```

Pour chaque nombre :

1. écrire la somme des puissances ;
2. calculer la valeur ;
3. vérifier avec Python.

## TP 2 — Conversions par divisions successives

Convertir :

```text
42
127
255
2026
4095
```

vers :

- base 2 ;
- base 8 ;
- base 16.

Interdiction d'utiliser `bin()`, `oct()` ou `hex()` pour la première étape.

## TP 3 — Conversion groupée

Convertir sans passer par le décimal :

```text
101101110011₂ → hex
7AF₁₆         → binaire
111010101₂    → octal
653₈          → binaire
```

## TP 4 — Fractions

Convertir :

```text
0,625₁₀
0,375₁₀
5,125₁₀
```

vers le binaire.

Puis déterminer lesquelles des fractions suivantes ont une écriture binaire finie :

```text
1/2
1/3
1/4
1/5
1/8
1/10
3/16
```

Justifier avec les facteurs premiers du dénominateur.

## TP 5 — Complément à deux

Sur 8 bits, représenter :

```text
5
-5
42
-42
127
-128
```

Puis vérifier les additions :

```text
5 + (-5)
42 + (-17)
```

## TP 6 — Détecter un overflow

Étudier sur 8 bits signés :

```text
100 + 40
70 + 70
-100 - 40
```

Pour chaque opération :

1. écrire les opérandes en complément à deux ;
2. effectuer l'addition binaire ;
3. interpréter le résultat ;
4. expliquer le débordement éventuel.

## TP 7 — Masques binaires

Créer les permissions suivantes :

```text
READ    = 001
WRITE   = 010
EXECUTE = 100
```

Puis écrire des fonctions Python :

```python
has_permission(mask, permission)
add_permission(mask, permission)
remove_permission(mask, permission)
```

## TP 8 — Comprendre 0.1

En Python :

```python
x = 0.1
```

Observer :

```python
x.hex()
x.as_integer_ratio()
format(x, ".60f")
```

Puis expliquer pourquoi :

```python
0.1 + 0.2 != 0.3
```

## TP 9 — Base 12 contre base 10

Comparer les écritures de :

```text
1/2
1/3
1/4
1/5
1/6
1/8
1/12
```

entre bases 10 et 12.

Répondre ensuite :

> La base 12 est-elle « meilleure » ?

La réponse doit préciser les critères retenus.

## TP 10 — Convertisseur Bibi

Écrire deux fonctions :

```text
to_bibi(n: int) -> str
from_bibi(text: str) -> int
```

Tests minimum :

```text
0    ↔ HO
15   ↔ DI
71   ↔ BOBI
119  ↔ BIBI
2000 ↔ BIDAHO
```

## TP 11 — Décoder une trame hexadécimale

On reçoit :

```text
48 65 6C 6C 6F 20 21
```

1. convertir chaque octet en décimal ;
2. convertir en binaire ;
3. chercher la valeur ASCII correspondante ;
4. expliquer pourquoi l'hexadécimal est pratique pour ce type de dump.

## TP 12 — Concevoir une base

Imaginer un système de numération destiné à l'un des domaines suivants :

- cuisine ;
- mesure du temps ;
- informatique embarquée ;
- comptage manuel ;
- pédagogie.

Définir :

- base ;
- chiffres ;
- notation ;
- avantages ;
- inconvénients ;
- fractions favorisées ;
- méthode de conversion vers le décimal.

# 36. Projet final — Laboratoire de numération

Créer un petit programme Python en ligne de commande :

```text
numeration
```

capable de :

```text
numeration convert 42 --from 10 --to 2
numeration convert 2A --from 16 --to 10
numeration bibi encode 2000
numeration bibi decode BIDAHO
numeration twos-complement -42 --bits 8
numeration float 0.1
```

## Exigences

### Conversion

Supporter au minimum :

```text
bases 2 à 36
```

### Validation

Refuser :

```text
102₂
G₁₆
```

### Bibi

Utiliser la table correcte :

```text
HO HA HE HI BO BA BE BI KO KA KE KI DO DA DE DI
```

### Complément à deux

Vérifier que le nombre tient dans la largeur demandée.

### Flottants

Afficher :

- représentation hexadécimale Python ;
- ratio rationnel exact ;
- bytes `binary64` ;
- explication de l'arrondi.

### Tests

Ajouter des tests automatisés pour les conversions et les cas limites.

# 37. Checklist de maîtrise

Nous savons expliquer :

- [ ] nombre vs représentation ;
- [ ] chiffre vs valeur ;
- [ ] base positionnelle ;
- [ ] poids d'une position ;
- [ ] conversion vers le décimal ;
- [ ] divisions successives ;
- [ ] multiplication de la partie fractionnaire ;
- [ ] regroupement binaire ↔ octal/hex ;
- [ ] addition binaire ;
- [ ] bit, nibble et octet ;
- [ ] complément à deux ;
- [ ] plage d'un entier signé/non signé ;
- [ ] overflow ;
- [ ] opérations bit à bit ;
- [ ] virgule fixe ;
- [ ] principe de IEEE 754 ;
- [ ] pourquoi `0.1` n'est pas exact en binaire ;
- [ ] différence BCD/binaire ;
- [ ] différence code Gray/numération ;
- [ ] intérêt de la base 12 et de la base 60 ;
- [ ] fonctionnement du bibi-binaire ;
- [ ] table Bibi correcte.

# 38. Glossaire

**Base / radix**
: Nombre de valeurs de chiffres d'une numération positionnelle standard.

**Bit**
: Unité binaire pouvant prendre deux états.

**Byte / octet**
: Groupe de 8 bits dans l'usage informatique moderne.

**Chiffre**
: Symbole élémentaire participant à l'écriture d'un nombre.

**Complément à deux**
: Représentation dominante des entiers signés binaires.

**Endianness**
: Ordre des octets lors du stockage ou de la transmission d'une valeur multi-octets.

**Exposant**
: Partie d'un flottant contrôlant l'échelle de la valeur.

**Hexadécimal**
: Numération de base 16.

**IEEE 754**
: Standard de référence pour l'arithmétique à virgule flottante.

**Nibble**
: Groupe de quatre bits.

**Numération positionnelle**
: Système où la contribution d'un chiffre dépend de sa position.

**Overflow**
: Résultat mathématique hors de la plage représentable par le type utilisé.

**Radix point**
: Généralisation du séparateur décimal à une base quelconque.

**Significande**
: Partie significative d'une représentation en virgule flottante.

**Subnormal**
: Valeur IEEE 754 très proche de zéro représentée avec une précision réduite.

# 39. Références et approfondissements

## Numération

- Donald E. Knuth, *The Art of Computer Programming*, volume 2, pour de nombreux aspects de représentation et d'arithmétique.
- Documentation Python : `int`, `bin`, `oct`, `hex`, formatage et `float`.

## Virgule flottante

- **IEEE 754-2019 — IEEE Standard for Floating-Point Arithmetic** : standard actif au moment de cette mise à jour.
- David Goldberg, *What Every Computer Scientist Should Know About Floating-Point Arithmetic*.

## Bibi-binaire

- Robert Jean Lapointe, brevet français **n° 1 569 028**, *Procédé de codification de l'information*, demandé le 28 mars 1968.
- Jean-Marc Font, Jean-Claude Quiniou, Gérard Verroust et al., *Les Cerveaux non-humains : introduction à l'Informatique*, 1970.
- Voir également l'article déjà lié dans l'ancienne version du cours : [Boby Lapointe, chanteur et matheux inventeur de la numérotation Bibi](https://www.sciencesetavenir.fr/fondamental/mathematiques/boby-lapointe-chanteur-et-matheux-inventeur-de-la-numerotation-bibi_37872).

# 40. À retenir

1. Un nombre et son écriture sont deux choses différentes.
2. En base `b`, chaque position représente une puissance de `b`.
3. Le binaire est adapté aux machines ; l'hexadécimal est adapté aux humains qui lisent du binaire.
4. Le complément à deux permet de traiter efficacement les entiers signés.
5. Une fraction finie dans une base peut être périodique dans une autre.
6. IEEE 754 représente une immense plage de valeurs mais avec une précision finie.
7. `0.1` n'est pas représentable exactement en virgule flottante binaire finie.
8. Il n'existe pas de « meilleure base » indépendamment du critère choisi.
9. Le bibi-binaire de Boby Lapointe est une notation phonétique et graphique de l'hexadécimal.
10. La table correcte se termine par `... KO KA KE KI DO DA DE DI`.
