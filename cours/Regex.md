---
schema_version: 1
uid: "01M02EX5C7PYFHMG5GYPFKVD8D"
titre: "Regex"
aliases:
  - "Expressions régulières"
  - "Expressions rationnelles"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - expressions-regulieres
  - traitement-de-texte
resume: "Cours complet sur les expressions régulières : théorie, syntaxes, moteurs, Unicode, captures, lookarounds, substitutions, Python, JavaScript, POSIX, Vim, PCRE2, performances, ReDoS, sécurité et bonnes pratiques."
niveau: intermediaire
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-12-27
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Expressions régulières — Regex

> [!abstract] Objectif
> Écrire, lire et tester des expressions régulières en connaissant le moteur derrière (Python `re`, ECMAScript, PCRE2, RE2) : classes, quantificateurs, groupes, ancres, assertions, quantificateurs possessifs et groupes atomiques, Unicode, performances et retours en arrière catastrophiques, sécurité, et usage en Python, JavaScript et en ligne de commande.

Voir aussi : [[Python]], [[Javascript]], [[Sécurité avec Python]], [[git]].

Une **expression régulière** (_regular expression_, souvent abrégée **regex** ou **regexp**) décrit un motif de texte.

Elle peut servir à :

- rechercher des occurrences ;
- valider la **forme** d'une chaîne ;
- extraire des sous-chaînes ;
- découper un texte ;
- remplacer ou réorganiser des fragments ;
- analyser des logs ;
- filtrer des fichiers ;
- effectuer des transformations répétitives dans un éditeur ;
- construire certains analyseurs lexicaux.

Mais une regex n'est **ni un parseur universel**, ni un mécanisme de validation métier complet.

> [!important]
> Il n'existe pas une syntaxe regex unique. Une expression valable en Python peut être invalide dans `grep`, JavaScript, Vim, PCRE2 ou RE2.

Ce cours distingue donc systématiquement :

1. les **expressions régulières au sens théorique** ;
2. la syntaxe POSIX BRE/ERE ;
3. les moteurs à syntaxe de type Perl, comme Python ou PCRE2 ;
4. JavaScript ;
5. Vim/Neovim ;
6. les moteurs conçus pour éviter le backtracking pathologique, comme RE2.

## Objectifs

À la fin du cours, nous devons savoir :

- lire une regex sans la considérer comme une suite de symboles mystérieux ;
- construire un motif progressivement ;
- choisir entre recherche partielle et correspondance complète ;
- employer groupes, captures, alternances et quantificateurs ;
- utiliser les lookarounds lorsqu'ils sont disponibles ;
- comprendre les différences entre moteurs ;
- écrire des regex Unicode correctes ;
- utiliser les regex en Python, JavaScript, GNU `grep`, `sed` et Vim ;
- éviter les regex inutilement complexes ;
- diagnostiquer le **catastrophic backtracking** ;
- prévenir les attaques **ReDoS** ;
- savoir quand **ne pas utiliser une regex**.

---

# Sommaire

1. Histoire et fondements
2. Une regex n'a de sens qu'avec son moteur
3. Le bon modèle mental : chercher un motif
4. Littéraux et échappement
5. Le caractère point `.`
6. Classes de caractères
7. Ancres et frontières
8. Quantificateurs
9. Alternance
10. Groupes
11. Backreferences
12. Lookarounds
13. Groupes atomiques et maîtrise du backtracking
14. Flags et modes
15. Unicode : le piège des « caractères »
16. Substitution
17. Découpage et extraction
18. Python 3.14 et le module `re`
19. JavaScript moderne
20. POSIX, GNU grep, sed et awk
21. Vim et Neovim
22. PCRE2
23. RE2 et les moteurs à temps linéaire
24. Comment fonctionne un moteur à backtracking ?
25. ReDoS — Regular Expression Denial of Service
26. Regex et validation
27. Quand ne pas utiliser une regex ?
28. Méthode de conception d'une regex
29. Regex lisibles et maintenables
30. Tests automatisés
31. Debugging
32. Exemples pratiques
33. Regex, sécurité et données sensibles
34. Portabilité entre moteurs
35. Anti-patterns
36. Choisir entre regex et code explicite
37. TP 1 — Bases
38. TP 2 — Classes et ancres
39. TP 3 — Extraction de logs
40. TP 4 — Substitution
41. TP 5 — Python et groupes nommés
42. TP 6 — Unicode
43. TP 7 — Lookarounds
44. TP 8 — ReDoS
45. TP 9 — GNU grep
46. TP 10 — JavaScript moderne
47. TP 11 — Regex vs parseur
48. TP 12 — Audit d'une regex de production
49. Projet final — Mini-outil d'analyse de logs
50. Checklist d'une regex de production
51. Aide-mémoire
52. Glossaire
53. Références
54. Conclusion

# 1. Histoire et fondements

## 1.1. Stephen Kleene et les langages réguliers

Les origines théoriques des expressions régulières remontent aux travaux de **Stephen Cole Kleene** dans les années 1940 et 1950 sur les automates finis et les ensembles réguliers.

Un langage régulier peut être décrit de plusieurs façons équivalentes :

- une expression régulière au sens mathématique ;
- un automate fini non déterministe (**NFA**) ;
- un automate fini déterministe (**DFA**).

Les opérations fondamentales sont notamment :

- la concaténation ;
- l'alternance ;
- l'étoile de Kleene, c'est-à-dire la répétition zéro ou plusieurs fois.

Par exemple, dans une notation simplifiée :

```text
(ab|cd)*
```

décrit toute concaténation de zéro ou plusieurs blocs `ab` ou `cd`.

## 1.2. Ken Thompson et Unix

Dans les années 1960, **Ken Thompson** implémente une technique de recherche basée sur ces idées dans l'éditeur QED, puis dans `ed`.

Cette approche se diffuse ensuite dans l'écosystème Unix :

- `grep` ;
- `sed` ;
- `awk` ;
- les éditeurs `ed`, `vi`, puis Vim ;
- plus tard Perl et de nombreux langages modernes.

Le nom `grep` vient historiquement d'une commande de l'éditeur `ed` :

```text
g/re/p
```

que l'on peut lire comme « global / regular expression / print ».

## 1.3. Les regex modernes dépassent parfois les langages réguliers

Une distinction conceptuelle est importante.

Les expressions régulières **théoriques** reconnaissent des langages réguliers.

Mais plusieurs moteurs modernes ajoutent :

- des backreferences ;
- de la récursion ;
- des sous-routines ;
- des conditions ;
- parfois des extensions encore plus puissantes.

Par exemple :

```regex
^(\w+)\s+\1$
```

utilise une **backreference** pour vérifier que le même mot apparaît deux fois.

Une telle fonctionnalité ne correspond plus simplement à une expression régulière au sens de la théorie des automates finis.

Nous devons donc distinguer :

> **regex pratique** ≠ nécessairement **langage régulier théorique**.

Voir également [[Logique]] pour les notions de langage formel et de calculabilité.

---

# 2. Une regex n'a de sens qu'avec son moteur

Une regex est interprétée par un **moteur**.

Le moteur définit :

- la syntaxe disponible ;
- le sens exact de certaines classes ;
- la gestion d'Unicode ;
- les fonctionnalités avancées ;
- la stratégie d'exécution ;
- les performances ;
- les risques de backtracking.

## 2.1. Grandes familles

| Famille / moteur | Exemples | Particularités |
|---|---|---|
| POSIX BRE | `grep`, `sed` par défaut | Syntaxe historique |
| POSIX ERE | `grep -E`, `awk` | Alternance et quantificateurs plus naturels |
| PCRE2 | `pcre2grep`, nombreux logiciels | Syntaxe de type Perl très riche |
| Python `re` | Python | Backtracking, groupes nommés Python |
| ECMAScript RegExp | navigateurs, Node.js | Syntaxe JavaScript, modes `u`/`v` |
| Vim regex | Vim/Neovim | Syntaxe et niveaux de « magic » spécifiques |
| RE2 | Google RE2 et dérivés | Temps linéaire, pas de backreferences/lookarounds généralisés |

## 2.2. PCRE n'est pas Perl

**PCRE** signifie **Perl-Compatible Regular Expressions**.

PCRE/PCRE2 est une bibliothèque distincte qui s'inspire fortement de Perl.

Il est donc incorrect de dire :

> « Perl utilise PCRE ».

Perl possède son propre moteur. PCRE cherche à fournir une syntaxe et des comportements largement compatibles avec l'écosystème Perl.

Au 29 août 2026, la dernière version stable publiée de **PCRE2 est 10.47** ; 10.48 est encore en développement.

## 2.3. Toujours préciser la saveur

Quand nous documentons une regex non triviale, écrivons par exemple :

```text
Moteur : Python 3.14 re
```

ou :

```text
Moteur : ECMAScript RegExp avec flag v
```

ou :

```text
Moteur : PCRE2 UTF + UCP
```

Cela rend l'exemple reproductible.

---

# 3. Le bon modèle mental : chercher un motif

Considérons le texte :

```text
La température est de 21.5 °C.
```

La regex :

```regex
\d+\.\d+
```

peut trouver :

```text
21.5
```

Une regex décrit **comment reconnaître une forme**, pas ce que cette forme signifie.

La regex précédente acceptera aussi :

```text
999999.999999
```

même si la valeur n'a aucun sens dans notre domaine métier.

## 3.1. Recherche partielle et correspondance complète

Deux opérations sont souvent confondues.

### Recherche partielle

Nous cherchons un motif **quelque part** dans le texte.

Exemple Python :

```python
import re

m = re.search(r"\d+", "abc 123 xyz")
assert m.group() == "123"
```

### Correspondance complète

Toute la chaîne doit respecter le motif.

```python
import re

assert re.fullmatch(r"\d+", "123")
assert not re.fullmatch(r"\d+", "abc 123")
```

Pour valider la forme d'une valeur, `fullmatch()` est souvent plus explicite qu'un assemblage manuel de `^...$`.

## 3.2. Une correspondance possède une position

Une correspondance n'est pas seulement une chaîne.

Elle possède :

- un début ;
- une fin ;
- éventuellement des groupes capturés.

```python
import re

m = re.search(r"\d+", "port=443")
print(m.group())  # 443
print(m.span())   # (5, 8)
```

---

# 4. Littéraux et échappement

## 4.1. Les métacaractères

Dans la plupart des saveurs modernes, ces caractères peuvent posséder une signification spéciale :

```text
. ^ $ * + ? { } [ ] \ | ( )
```

Pour rechercher un point littéral :

```regex
\.
```

Pour rechercher `a+b` littéralement :

```regex
a\+b
```

## 4.2. Deux niveaux d'échappement

Dans un langage de programmation, nous pouvons avoir **deux parseurs successifs** :

1. le parseur de chaîne du langage ;
2. le parseur regex.

En Python :

```python
pattern = "\\d+"
```

est possible, mais difficile à lire.

Nous préférons généralement une **raw string** :

```python
pattern = r"\d+"
```

## 4.3. Ne jamais construire un motif littéral à la main

Si une valeur doit être interprétée comme du **texte littéral**, utilisons l'outil d'échappement du langage.

Python :

```python
import re

user_text = "a+b(c)"
pattern = re.escape(user_text)
assert re.fullmatch(pattern, user_text)
```

JavaScript moderne :

```javascript
const userText = "a+b(c)";
const regex = new RegExp(RegExp.escape(userText));
```

`RegExp.escape()` est disponible dans les navigateurs modernes depuis 2025 et évite les implémentations d'échappement artisanales incomplètes.

---

# 5. Le caractère point `.`

Dans la plupart des moteurs :

```regex
.
```

signifie « un caractère quelconque », **sauf les fins de ligne par défaut**.

Par exemple :

```regex
c.t
```

peut correspondre à :

```text
cat
cot
c3t
c t
```

mais pas nécessairement à un `c`, puis un saut de ligne, puis `t`.

## 5.1. Mode DOTALL

Python :

```python
re.search(r"a.*b", text, re.DOTALL)
```

ou :

```regex
(?s:a.*b)
```

JavaScript :

```javascript
/a.*b/s
```

## 5.2. Le point n'est pas toujours « un caractère visuel »

En Unicode, un caractère perçu par l'utilisateur peut être composé de plusieurs **code points**.

Nous reviendrons sur ce problème au chapitre Unicode.

---

# 6. Classes de caractères

Une classe :

```regex
[abc]
```

correspond à **un seul** caractère parmi `a`, `b` ou `c`.

## 6.1. Intervalles

```regex
[0-9]
[A-Z]
[a-z]
```

Nous pouvons les combiner :

```regex
[A-Za-z0-9_]
```

## 6.2. Négation

```regex
[^0-9]
```

signifie un caractère qui n'est pas compris entre `0` et `9`.

Le `^` n'a donc pas le même rôle :

- hors classe : ancre de début ;
- juste après `[` : négation de classe.

## 6.3. Classes prédéfinies

De nombreux moteurs possèdent :

```regex
\d
\D
\w
\W
\s
\S
```

Mais leur sens exact dépend du moteur et du mode Unicode.

### Exemple Python

Dans une regex `str` Python normale :

```regex
\d
```

peut reconnaître des chiffres Unicode et pas uniquement `[0-9]`.

De même, `\w` est plus large que `[A-Za-z0-9_]` en mode Unicode.

Si nous voulons explicitement l'ASCII :

```python
re.fullmatch(r"\d+", value, re.ASCII)
```

## 6.4. Classes POSIX

Dans les environnements compatibles :

```regex
[[:digit:]]
[[:alpha:]]
[[:alnum:]]
[[:space:]]
```

sont souvent préférables à certaines constructions dépendantes du moteur.

---

# 7. Ancres et frontières

## 7.1. Début et fin

```regex
^début
fin$
```

`^` et `$` sont souvent influencés par le mode multiline.

## 7.2. Fin absolue en Python 3.14

Python 3.14 introduit :

```regex
\z
```

pour correspondre uniquement à la fin de la chaîne.

Python conserve aussi `\Z` comme équivalent pour compatibilité.

Cela peut être plus précis que `$` lorsqu'un moteur donne à `$` un comportement spécial vis-à-vis des fins de ligne.

## 7.3. Frontière de mot

```regex
\bchat\b
```

cherche `chat` comme mot délimité selon la définition de `\w`/`\W` du moteur.

Attention : une **frontière de mot regex** n'est pas une segmentation linguistique universelle.

Pour des langues ou écritures complexes, `\b` peut être insuffisant.

## 7.4. `\b` dans une classe

Dans plusieurs moteurs, dont Python :

```regex
[\b]
```

ne signifie pas frontière de mot, mais caractère **backspace**.

Le contexte change donc le sens de l'échappement.

---

# 8. Quantificateurs

## 8.1. Répétitions de base

```regex
a?       # 0 ou 1
a*       # 0 ou plusieurs
a+       # 1 ou plusieurs
a{3}     # exactement 3
a{2,5}   # de 2 à 5
a{2,}    # au moins 2
a{,5}    # dépend du moteur ; éviter si portabilité nécessaire
```

## 8.2. Le quantificateur porte sur l'élément précédent

```regex
ab+
```

signifie :

```text
a suivi d'un ou plusieurs b
```

Pour répéter `ab` :

```regex
(?:ab)+
```

## 8.3. Greedy

Par défaut, la plupart des quantificateurs des moteurs de type Perl sont **gloutons** (_greedy_).

Sur :

```text
<a>un</a><a>deux</a>
```

le motif :

```regex
<a>.*</a>
```

peut capturer :

```text
<a>un</a><a>deux</a>
```

## 8.4. Lazy / reluctant

```regex
<a>.*?</a>
```

cherche le minimum nécessaire.

Cela peut donner :

```text
<a>un</a>
```

Mais cette astuce ne transforme pas une regex en parseur HTML fiable.

Pour HTML réel, utilisons un parseur HTML. Voir [[HTML]].

## 8.5. Possessive

Certains moteurs, dont Python depuis 3.11 et PCRE2, supportent :

```regex
a*+
a++
a?+
a{2,5}+
```

Un quantificateur possessif ne rend pas ses caractères au moteur pendant le backtracking.

Exemple Python :

```python
import re

assert re.fullmatch(r"a*a", "aaaa")
assert not re.fullmatch(r"a*+a", "aaaa")
```

Ces quantificateurs peuvent :

- clarifier une intention ;
- empêcher certains backtrackings inutiles ;
- aider à éviter des performances catastrophiques.

---

# 9. Alternance

L'opérateur :

```regex
chat|chien
```

signifie `chat` **ou** `chien`.

## 9.1. La portée compte

```regex
^chat|chien$
```

ne signifie pas nécessairement « toute la chaîne est `chat` ou `chien` ».

Écrivons :

```regex
^(?:chat|chien)$
```

ou, en Python :

```python
re.fullmatch(r"chat|chien", value)
```

## 9.2. Ordre des alternatives

Dans beaucoup de moteurs à backtracking, les alternatives sont essayées de gauche à droite.

Par exemple :

```regex
foo|foobar
```

sur `foobar` peut retourner `foo` lors d'une recherche partielle.

Si nous voulons privilégier la forme longue :

```regex
foobar|foo
```

---

# 10. Groupes

## 10.1. Groupe capturant

```regex
(ab)+
```

Les parenthèses :

1. groupent l'expression ;
2. créent généralement une capture.

Python :

```python
import re

m = re.fullmatch(r"(\d{4})-(\d{2})-(\d{2})", "2026-08-29")
assert m.groups() == ("2026", "08", "29")
```

## 10.2. Groupe non capturant

Si nous avons seulement besoin de grouper :

```regex
(?:ab)+
```

Cela évite de créer une capture inutile.

## 10.3. Groupes nommés

Ils rendent les regex longues bien plus lisibles.

Python :

```regex
(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})
```

```python
import re

m = re.fullmatch(
    r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})",
    "2026-08-29",
)
print(m.group("year"))
```

JavaScript :

```regex
(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})
```

```javascript
const m = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/.exec("2026-08-29");
console.log(m.groups.year);
```

> [!tip]
> Pour une regex maintenue à long terme, préférons des groupes nommés lorsque le groupe possède une signification métier.

---

# 11. Backreferences

Une **backreference** demande au moteur de reconnaître exactement le texte capturé précédemment.

## 11.1. Capture numérique

```regex
\b(\w+)\s+\1\b
```

peut trouver :

```text
le le
```

## 11.2. Capture nommée en Python

```regex
(?P<quote>['"]).*?(?P=quote)
```

Le même caractère d'ouverture doit fermer la chaîne.

## 11.3. Pourquoi elles changent la nature du problème

Les backreferences apportent une puissance qui n'est pas celle des automates finis ordinaires.

Elles sont donc :

- très utiles ;
- moins portables ;
- absentes de moteurs comme RE2 ;
- susceptibles de compliquer l'analyse de performance.

---

# 12. Lookarounds

Les **lookarounds** vérifient une condition sans consommer les caractères correspondants.

Ils ne sont pas disponibles dans tous les moteurs.

## 12.1. Lookahead positif

```regex
\d+(?=€)
```

cherche des chiffres suivis d'un symbole `€`, sans inclure ce symbole dans le match.

## 12.2. Lookahead négatif

```regex
foo(?!bar)
```

cherche `foo` qui n'est pas immédiatement suivi de `bar`.

## 12.3. Lookbehind positif

```regex
(?<=€)\d+
```

cherche des chiffres précédés de `€`.

## 12.4. Lookbehind négatif

```regex
(?<!-)\b\d+\b
```

peut servir à exclure certains nombres précédés d'un tiret.

## 12.5. Portabilité du lookbehind

Le support et les contraintes du lookbehind diffèrent fortement selon les moteurs.

Avant de l'utiliser dans un format portable :

1. vérifions le moteur cible ;
2. testons les cas limites ;
3. préférons parfois une capture suivie d'un traitement applicatif.

RE2 ne fournit pas les assertions lookaround généralisées.

---

# 13. Groupes atomiques et maîtrise du backtracking

Un groupe atomique :

```regex
(?>...)
```

interdit au moteur de revenir à l'intérieur du groupe après en être sorti.

Python supporte les groupes atomiques depuis 3.11.

Exemple conceptuel :

```regex
(?>a*)a
```

sur :

```text
aaaa
```

échoue, car `a*` prend tous les `a` et ne peut ensuite plus en rendre un.

Les groupes atomiques sont utiles pour :

- exprimer une décision irréversible ;
- réduire le backtracking ;
- sécuriser certains motifs complexes ;
- améliorer les performances lorsque leur sémantique est bien comprise.

Ils ne doivent pas être ajoutés au hasard : ils peuvent aussi changer le résultat.

---

# 14. Flags et modes

Les flags modifient l'interprétation du motif.

## 14.1. Python

Flags courants :

```python
re.IGNORECASE
re.MULTILINE
re.DOTALL
re.VERBOSE
re.ASCII
```

Exemple :

```python
pattern = re.compile(r"^erreur:", re.IGNORECASE | re.MULTILINE)
```

## 14.2. Flags inline

```regex
(?i)bonjour
```

ou pour une zone limitée :

```regex
(?i:bonjour)
```

Le support exact dépend du moteur.

## 14.3. JavaScript

Les flags modernes sont :

| Flag | Rôle |
|---|---|
| `d` | indices des captures |
| `g` | recherche globale |
| `i` | insensible à la casse |
| `m` | multiline |
| `s` | dotAll |
| `u` | mode Unicode |
| `v` | Unicode Sets, extension moderne de `u` |
| `y` | recherche sticky à partir de `lastIndex` |

Les modes `u` et `v` sont incompatibles entre eux : nous choisissons l'un ou l'autre.

---

# 15. Unicode : le piège des « caractères »

Unicode est l'une des principales sources d'erreurs en regex modernes.

## 15.1. Octet, code point et grapheme cluster

Nous devons distinguer :

- l'octet ;
- le code point Unicode ;
- la séquence de code points ;
- le **grapheme cluster**, qui correspond davantage au caractère perçu par l'utilisateur.

Par exemple, un `é` peut être représenté :

1. par le code point précomposé `U+00E9` ;
2. par `e` + accent combinant.

Visuellement, le résultat peut être identique.

## 15.2. Normalisation

Avant certaines comparaisons, normalisons le texte.

Python :

```python
import unicodedata

text = unicodedata.normalize("NFC", text)
```

JavaScript :

```javascript
const normalized = text.normalize("NFC");
```

## 15.3. Propriétés Unicode

PCRE2 et JavaScript moderne proposent des classes basées sur des propriétés Unicode.

JavaScript :

```javascript
/^\p{Letter}+$/u
```

Avec le mode `v`, les classes Unicode deviennent encore plus puissantes :

```javascript
/[\p{Letter}&&\p{Script=Greek}]/v
```

Le flag `v` permet notamment :

- intersection ;
- soustraction ;
- ensembles Unicode plus expressifs.

Il est largement disponible dans les navigateurs modernes depuis 2023.

## 15.4. Une propriété Unicode ne signifie pas grapheme cluster

Même le mode JavaScript `v` ne transforme pas automatiquement un emoji composé en un seul caractère regex.

Pour découper correctement du texte destiné à des humains, des outils spécialisés de segmentation Unicode peuvent être nécessaires.

---

# 16. Substitution

Les regex deviennent particulièrement utiles lorsqu'elles sont combinées à une **fonction de remplacement**.

## 16.1. Python

```python
import re

text = "Launay, Michael"
result = re.sub(r"(\w+),\s+(\w+)", r"\2 \1", text)
print(result)
```

## 16.2. Avec groupes nommés

```python
pattern = r"(?P<family>\w+),\s+(?P<given>\w+)"
replacement = r"\g<given> \g<family>"
```

## 16.3. Fonction de remplacement

Pour une transformation non triviale :

```python
import re

text = "prix: 10, prix: 25"

result = re.sub(
    r"\d+",
    lambda m: str(int(m.group()) * 2),
    text,
)

assert result == "prix: 20, prix: 50"
```

C'est souvent plus lisible que de forcer toute la logique dans la regex.

## 16.4. JavaScript

```javascript
const input = "Launay, Michael";
const output = input.replace(
  /(?<family>\w+),\s+(?<given>\w+)/,
  "$<given> $<family>",
);
```

---

# 17. Découpage et extraction

## 17.1. `split`

Python :

```python
import re

parts = re.split(r"\s*[,;]\s*", "alpha, beta;gamma")
assert parts == ["alpha", "beta", "gamma"]
```

Attention : si le motif contient des **groupes capturants**, certaines APIs incluent les séparateurs capturés dans le résultat.

## 17.2. `findall` et groupes

Python :

```python
re.findall(r"\d+", "A12 B34")
```

donne :

```python
["12", "34"]
```

Mais :

```python
re.findall(r"([A-Z])(\d+)", "A12 B34")
```

donne une liste de tuples.

Pour une API uniforme et les positions, préférons souvent `finditer()` :

```python
for match in re.finditer(r"(?P<name>[A-Z])(?P<id>\d+)", text):
    print(match.groupdict(), match.span())
```

---

# 18. Python 3.14 et le module `re`

Voir également [[Python]].

## 18.1. Compiler un motif

```python
import re

USER_ID = re.compile(r"[a-z][a-z0-9_-]{2,31}", re.ASCII)

if USER_ID.fullmatch(value):
    ...
```

Cela :

- donne un nom métier au motif ;
- centralise les flags ;
- améliore la lisibilité ;
- évite de répéter le motif.

Le module `re` maintient également un cache interne des motifs récents ; compiler explicitement reste surtout une bonne pratique de conception lorsque le motif est réutilisé.

## 18.2. Raw strings

Préférons :

```python
r"\d+\.\d+"
```

à :

```python
"\\d+\\.\\d+"
```

## 18.3. `match`, `search`, `fullmatch`

```python
re.match(pattern, text)      # début de chaîne
re.search(pattern, text)     # n'importe où
re.fullmatch(pattern, text)  # toute la chaîne
```

Évitons d'utiliser `match()` lorsqu'en réalité nous voulons `search()`.

## 18.4. Groupes nommés

```python
m = re.fullmatch(
    r"(?P<scheme>https?)://(?P<host>[^/]+)(?P<path>/.*)?",
    url,
)
```

## 18.5. Motifs lisibles avec `re.VERBOSE`

Pour une regex longue :

```python
DATE = re.compile(
    r"""
    (?P<year>\d{4})
    -
    (?P<month>\d{2})
    -
    (?P<day>\d{2})
    """,
    re.VERBOSE,
)
```

Cette approche permet :

- indentation ;
- commentaires ;
- séparation visuelle des groupes.

> [!warning]
> En mode `VERBOSE`, les espaces non échappés hors classes sont généralement ignorés. Si un espace littéral est requis, écrivons-le explicitement.

## 18.6. Groupes atomiques et quantificateurs possessifs

Depuis Python 3.11 :

```regex
(?>...)
*+
++
?+
{m,n}+
```

sont disponibles.

## 18.7. Ancre `\z`

Depuis Python 3.14 :

```regex
\z
```

représente la fin absolue de la chaîne.

## 18.8. Motifs dynamiques

Pour chercher une valeur utilisateur **comme texte littéral** :

```python
pattern = re.compile(re.escape(user_input))
```

Ne concaténons pas directement une entrée non fiable dans une regex lorsqu'elle n'est pas censée fournir de syntaxe regex.

---

# 19. JavaScript moderne

Voir également [[Javascript]].

## 19.1. Littéral et constructeur

Littéral :

```javascript
const re = /foo\d+/i;
```

Dynamique :

```javascript
const re = new RegExp(pattern, "giu");
```

## 19.2. Groupes nommés

```javascript
const re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
```

## 19.3. `matchAll`

```javascript
const re = /(?<name>[A-Z])(?<id>\d+)/g;

for (const match of text.matchAll(re)) {
  console.log(match.groups);
}
```

## 19.4. Flag `d`

```javascript
const re = /(?<word>\w+)/dg;
```

permet d'obtenir les indices exacts des captures via `match.indices`.

## 19.5. Flag `v`

Le mode :

```javascript
/.../v
```

est l'évolution moderne du mode Unicode `u` pour les ensembles Unicode avancés.

Exemple :

```javascript
const greekLetter = /[\p{Letter}&&\p{Script=Greek}]/v;
```

## 19.6. `RegExp.escape()`

Lorsque nous transformons du texte dynamique en motif littéral :

```javascript
const literal = RegExp.escape(userText);
const re = new RegExp(`^${literal}$`);
```

Cette API est devenue disponible sur les navigateurs modernes en 2025.

---

# 20. POSIX, GNU grep, sed et awk

Les outils Unix utilisent des syntaxes historiques qu'il ne faut pas confondre avec PCRE2.

## 20.1. GNU grep

GNU `grep` propose notamment :

```bash
grep -G 'pattern' file   # BRE, défaut
grep -E 'pattern' file   # ERE
grep -F 'text' file      # chaînes fixes
grep -P 'pattern' file   # PCRE, si disponible
```

> [!tip]
> Si nous cherchons une chaîne fixe, préférons `grep -F`. Il n'y a alors aucune syntaxe regex à interpréter.

## 20.2. BRE vs ERE

En simplifiant :

- BRE est la syntaxe historique de base ;
- ERE offre une notation plus naturelle pour `+`, `?`, `|` et les groupes.

Avec GNU grep, BRE et ERE fournissent essentiellement la même puissance de correspondance, mais avec une notation différente.

## 20.3. Exemples grep

```bash
grep -E '^[0-9]+$' data.txt
```

```bash
grep -Eo 'https?://[^[:space:]]+' logs.txt
```

```bash
grep -F 'a+b(c)' file.txt
```

## 20.4. `sed`

GNU `sed` utilise BRE par défaut.

```bash
sed 's/foo/bar/g' file.txt
```

Avec ERE :

```bash
sed -E 's/([A-Za-z]+),[[:space:]]*([A-Za-z]+)/\2 \1/g' file.txt
```

## 20.5. `awk`

`awk` utilise une syntaxe de regex de type ERE :

```bash
awk '$0 ~ /^[0-9]+$/ { print $0 }' file.txt
```

## 20.6. Shell et quoting

Préférons généralement les **quotes simples** autour d'une regex shell :

```bash
grep -E '^[A-Z][A-Za-z0-9_-]+$' file
```

Sinon le shell peut interpréter :

- `$` ;
- `*` ;
- `?` ;
- `[` ;
- backticks ;
- substitutions.

---

# 21. Vim et Neovim

Vim possède sa propre syntaxe regex.

Nous ne devons donc pas copier aveuglément une regex PCRE dans Vim.

## 21.1. Recherche

```vim
/foo
```

Début de ligne :

```vim
/^foo
```

Fin :

```vim
/foo$/
```

## 21.2. Substitution

```vim
:%s/foo/bar/g
```

Le `%` signifie ici l'ensemble du fichier.

## 21.3. Groupes dans la syntaxe historique

```vim
:%s/^\([^;]*\);\([^;]*\)/\2;\1/
```

## 21.4. Very magic

Pour se rapprocher d'une syntaxe regex moderne :

```vim
\v(pattern|other)+
```

Exemple :

```vim
:%s/\v^([^;]+);([^;]+)/\2;\1/
```

Le préfixe `\v` active le mode **very magic**.

## 21.5. Modifier la portion remplacée avec `\zs` et `\ze`

Vim possède des extensions très utiles :

```vim
/foo\zsbar
```

recherche `foobar`, mais considère que le match remplaçable commence à `bar`.

De même :

```vim
foo\zebar
```

fait terminer la correspondance avant `bar`.

Ces constructions sont **spécifiques à Vim**.

---

# 22. PCRE2

PCRE2 est l'un des moteurs les plus riches et les plus utilisés comme bibliothèque embarquée.

Au 29 août 2026 :

- **10.47** est la dernière release stable ;
- 10.48 est en développement.

## 22.1. Fonctionnalités fréquentes

PCRE2 peut notamment fournir :

- groupes nommés ;
- lookarounds ;
- groupes atomiques ;
- quantificateurs possessifs ;
- propriétés Unicode ;
- `\R` pour diverses fins de ligne ;
- `\X` pour les séquences graphemiques étendues ;
- `\K` pour réinitialiser le début du match ;
- sous-routines et récursion ;
- conditionnels.

Toutes ces constructions réduisent la portabilité.

## 22.2. UTF et UCP

Il faut distinguer :

- décodage UTF ;
- propriétés Unicode utilisées par les classes comme `\w`.

Selon l'API et le logiciel qui embarque PCRE2, ces options peuvent devoir être activées explicitement.

## 22.3. Plus puissant ne signifie pas toujours meilleur

Un moteur riche peut être utile pour :

- transformation complexe ;
- parsing lexical avancé ;
- migration ponctuelle de données.

Mais dans du code applicatif maintenu pendant des années, une regex plus simple + quelques lignes de code sont souvent préférables à une construction PCRE2 extrêmement sophistiquée.

---

# 23. RE2 et les moteurs à temps linéaire

Google **RE2** a été conçu avec la sécurité comme objectif central.

Sa garantie importante est un temps de correspondance **linéaire dans la taille de l'entrée**.

Pour préserver cette propriété, RE2 ne supporte notamment pas :

- les backreferences ;
- les lookarounds généralisés.

## 23.1. Compromis

Moteur à backtracking :

```text
+ fonctionnalités très riches
+ souvent extrêmement rapide sur les cas simples
- peut explorer énormément de chemins
- risque de catastrophic backtracking
```

RE2 :

```text
+ comportement temporel prévisible
+ adapté aux motifs potentiellement non fiables
- syntaxe volontairement moins expressive
```

## 23.2. Le bon moteur dépend du contexte

Pour une regex codée par un développeur et soigneusement testée, Python `re` ou PCRE2 peuvent convenir.

Pour un système qui permet à des utilisateurs d'envoyer leurs propres motifs, un moteur avec garanties de ressources peut être beaucoup plus approprié.

---

# 24. Comment fonctionne un moteur à backtracking ?

Considérons :

```regex
^(a+)+$
```

et une entrée constituée de nombreux `a`, suivis d'un caractère qui fait échouer la correspondance :

```text
aaaaaaaaaaaaaaaaaaaaaaaaaX
```

Un moteur à backtracking peut devoir essayer de nombreuses répartitions des `a` entre :

- le `a+` intérieur ;
- la répétition extérieure `+`.

Le nombre de chemins possibles peut croître extrêmement vite.

C'est le **catastrophic backtracking**.

## 24.1. Pourquoi le cas qui réussit peut masquer le problème

Sur :

```text
aaaaaaaaaaaaaaaaaaaaaaaaa
```

le motif peut réussir rapidement.

C'est souvent l'entrée **presque valide mais finalement invalide** qui déclenche l'explosion.

Nous devons donc tester les cas négatifs longs.

---

# 25. ReDoS — Regular Expression Denial of Service

Une attaque **ReDoS** exploite une regex dont le temps de traitement devient énorme pour certaines entrées.

Cela peut :

- monopoliser un thread ;
- saturer un serveur ;
- retarder une file de traitements ;
- provoquer un déni de service.

Voir également [[Sécurité avancée sous Linux]] pour les principes généraux de défense en profondeur.

## 25.1. Motifs suspects

Nous devons être particulièrement prudents avec :

### Quantificateurs imbriqués ambiguës

```regex
(a+)+
```

### Alternatives qui se recouvrent

```regex
(a|aa)+
```

### Wildcards répétés suivis d'une condition tardive

```regex
^.*.*X$
```

### Motifs où plusieurs chemins reconnaissent la même portion

Plus le moteur possède de façons équivalentes de consommer le même préfixe, plus le backtracking peut devenir problématique.

## 25.2. Réduire le risque

1. simplifier le motif ;
2. éviter les quantificateurs imbriqués ambigus ;
3. rendre les alternatives mutuellement plus distinctes ;
4. utiliser des groupes atomiques ou quantificateurs possessifs lorsque leur sémantique convient ;
5. limiter la taille des entrées ;
6. appliquer des timeouts lorsqu'une API le permet ;
7. utiliser un moteur à temps borné/linéaire si les motifs sont non fiables ;
8. tester les entrées adversariales.

## 25.3. Entrée utilisateur comme motif

Il y a deux cas très différents.

### L'utilisateur fournit du texte à rechercher

Échappons-le :

```python
re.escape(user_text)
```

### L'utilisateur fournit volontairement une regex

Alors la regex devient du **code déclaratif non fiable**.

Nous devons envisager :

- moteur sûr ;
- taille maximale du motif ;
- taille maximale de l'entrée ;
- budget de temps ;
- budget mémoire ;
- restrictions de syntaxe ;
- audit et journalisation.

---

# 26. Regex et validation

Une regex valide une **forme syntaxique**, pas forcément une valeur métier.

## 26.1. Date

```regex
\d{4}-\d{2}-\d{2}
```

accepte :

```text
2026-99-99
```

Pour une vraie date, faisons :

1. validation grossière de la forme si nécessaire ;
2. parsing avec une bibliothèque de dates ;
3. validation métier.

Python :

```python
from datetime import date

year, month, day = map(int, text.split("-"))
value = date(year, month, day)
```

## 26.2. Adresse e-mail

Une regex simplifiée peut suffire pour un formulaire :

```regex
^[^\s@]+@[^\s@]+\.[^\s@]+$
```

Mais elle n'implémente pas l'ensemble des RFC relatives aux adresses e-mail.

Pour déterminer si une adresse est réellement utilisable :

- normalisons selon les besoins ;
- envoyons une confirmation ;
- ne prétendons pas qu'une regex simplifiée prouve l'existence de la boîte.

## 26.3. URL

Évitons de reproduire toute la grammaire URL dans une regex.

Python :

```python
from urllib.parse import urlsplit

parts = urlsplit(url)
```

JavaScript :

```javascript
const parsed = new URL(url);
```

La regex peut ensuite servir à valider une politique locale plus petite.

---

# 27. Quand ne pas utiliser une regex ?

## 27.1. HTML et XML complexes

Utilisons un parseur HTML/XML.

Voir [[HTML]].

## 27.2. JSON

Utilisons :

```python
json.loads(...)
```

et non une regex récursive improvisée.

## 27.3. CSV

Un champ CSV peut contenir :

- le séparateur ;
- des guillemets échappés ;
- des retours à la ligne.

Utilisons un parseur CSV.

## 27.4. Code source

Une regex est utile pour :

- recherche exploratoire ;
- transformations simples ;
- extraction approximative.

Pour une refactorisation syntaxiquement correcte, préférons :

- AST ;
- parser ;
- Language Server ;
- outil de refactoring.

## 27.5. Données structurées

Règle pratique :

> Si le format possède déjà un parseur fiable, commençons par utiliser ce parseur.

---

# 28. Méthode de conception d'une regex

Une bonne regex se construit, elle ne se « devine » pas.

## Étape 1 — écrire des exemples positifs

```text
alice@example.org
bob.smith@example.net
```

## Étape 2 — écrire des exemples négatifs

```text
alice
@example.org
alice@
```

## Étape 3 — décider si le match doit être partiel ou complet

C'est un choix d'API, pas seulement de syntaxe.

## Étape 4 — construire le motif du général au précis

Commencer par :

```regex
.+@.+
```

puis réduire progressivement les ambiguïtés.

## Étape 5 — nommer les parties significatives

```regex
(?P<local>... )@(?P<domain>...)
```

## Étape 6 — limiter les quantificateurs

Préférons, lorsque le format le permet :

```regex
[^,]*
```

à :

```regex
.*
```

si nous savons que la virgule est le séparateur.

## Étape 7 — tester les cas limites

- chaîne vide ;
- longueur 1 ;
- longueur maximale ;
- caractères Unicode ;
- caractères de contrôle ;
- entrée presque valide ;
- entrée très longue.

## Étape 8 — mesurer

Une regex de production critique mérite un benchmark.

## Étape 9 — documenter le moteur

```text
Moteur : Python 3.14 re
```

## Étape 10 — documenter l'intention

Expliquons **ce que la regex cherche à garantir**, plutôt que de seulement recopier le motif.

---

# 29. Regex lisibles et maintenables

## 29.1. Donner un nom métier

Mauvais :

```python
if re.fullmatch(r"[A-Z]{2}-\d{6}", value):
    ...
```

Mieux :

```python
ORDER_ID_PATTERN = re.compile(r"[A-Z]{2}-\d{6}", re.ASCII)
```

## 29.2. Utiliser `VERBOSE` pour les gros motifs

```python
LOG_LINE = re.compile(
    r"""
    ^
    (?P<timestamp>\S+)
    \s+
    (?P<level>DEBUG|INFO|WARNING|ERROR|CRITICAL)
    \s+
    (?P<message>.*)
    $
    """,
    re.VERBOSE,
)
```

## 29.3. Préférer les classes structurées au joker

Mieux :

```regex
[^/]+
```

que :

```regex
.*?
```

si nous savons précisément que `/` délimite le champ.

## 29.4. Une regex n'a pas à tout faire

Très souvent :

```text
regex simple
    ↓
extraction de quelques champs
    ↓
validation Python/JavaScript explicite
```

est plus robuste qu'une unique regex de 600 caractères.

---

# 30. Tests automatisés

Une regex importante doit être testée comme du code.

## 30.1. Pytest paramétré

```python
import re
import pytest

USERNAME = re.compile(r"[a-z][a-z0-9_-]{2,31}", re.ASCII)


@pytest.mark.parametrize(
    "value",
    [
        "alice",
        "bob_42",
        "user-name",
    ],
)
def test_valid_username(value: str) -> None:
    assert USERNAME.fullmatch(value)


@pytest.mark.parametrize(
    "value",
    [
        "ab",
        "42alice",
        "alice!",
        "éloise",
    ],
)
def test_invalid_username(value: str) -> None:
    assert not USERNAME.fullmatch(value)
```

## 30.2. Tester la performance

```python
import re
import time

pattern = re.compile(r"^(a+)+$")
text = "a" * 25 + "X"

start = time.perf_counter()
pattern.fullmatch(text)
elapsed = time.perf_counter() - start

print(elapsed)
```

> [!warning]
> Ne lançons pas volontairement des motifs pathologiques avec des tailles énormes sur une machine de production.

## 30.3. Property-based testing

Hypothesis peut générer de nombreux cas automatiquement.

```python
from hypothesis import given, strategies as st

@given(st.text())
def test_parser_never_crashes(text: str) -> None:
    parse_input(text)
```

Cela est particulièrement utile autour des regex servant de frontière d'entrée.

---

# 31. Debugging

## 31.1. Réduire le motif

Si une regex échoue :

1. supprimer la moitié du motif ;
2. vérifier que la partie restante fonctionne ;
3. réintroduire les éléments un à un.

## 31.2. Tester sur une entrée minimale

Plutôt que 5 Mo de logs :

```text
2026-08-29 ERROR erreur de connexion
```

## 31.3. Rendre visibles les caractères invisibles

Python :

```python
print(repr(text))
```

Cela révèle :

- `\n` ;
- `\r` ;
- tabulations ;
- espaces inhabituels.

## 31.4. Vérifier la normalisation Unicode

Deux textes visuellement identiques peuvent avoir des représentations différentes.

## 31.5. Utiliser un testeur en ligne avec prudence

Des outils comme regex101 sont très utiles, mais :

- sélectionnons la **bonne saveur** ;
- ne collons pas de secrets ni de données personnelles ;
- confirmons ensuite le comportement dans le vrai runtime.

---

# 32. Exemples pratiques

## 32.1. Extraire une adresse IPv4 sans la valider sémantiquement

```regex
\b(?:\d{1,3}\.){3}\d{1,3}\b
```

Cette regex trouve également :

```text
999.999.999.999
```

Elle est utile comme **extracteur candidat**, mais pas comme validateur IPv4 complet.

En Python, utilisons ensuite :

```python
from ipaddress import ip_address

address = ip_address(candidate)
```

## 32.2. Extraire les niveaux de logs

```regex
\b(?:DEBUG|INFO|WARNING|ERROR|CRITICAL)\b
```

## 32.3. Renommer une convention

Transformer :

```text
user_name
```

en `user-name` :

```python
result = re.sub(r"_+", "-", value)
```

## 32.4. Nettoyer les espaces

```python
result = re.sub(r"\s+", " ", text).strip()
```

Attention : cette transformation peut aussi convertir des retours à la ligne en espaces. Vérifions que cela correspond bien au besoin.

## 32.5. Extraire une paire `clé=valeur`

```regex
(?P<key>[A-Za-z_][A-Za-z0-9_]*)=(?P<value>[^\s]+)
```

Pour un vrai format `.env`, les règles de quoting peuvent être plus complexes : utilisons alors un parseur dédié.

## 32.6. Version simplifiée `major.minor.patch`

```regex
(?P<major>0|[1-9]\d*)\.(?P<minor>0|[1-9]\d*)\.(?P<patch>0|[1-9]\d*)
```

Cela ne prétend pas implémenter toute la spécification SemVer avec prerelease et build metadata.

---

# 33. Regex, sécurité et données sensibles

## 33.1. Regex injection

Supposons :

```python
pattern = re.compile("^" + user_value + "$")
```

Si `user_value` vaut :

```text
.*
```

l'utilisateur a modifié le sens du motif.

Si nous voulions un texte littéral :

```python
pattern = re.compile("^" + re.escape(user_value) + "$")
```

## 33.2. Secrets dans les testeurs en ligne

Ne collons pas dans un service tiers :

- tokens ;
- logs clients ;
- mots de passe ;
- clés privées ;
- données personnelles ;
- dumps de production.

## 33.3. Regex de masquage

Une regex de redaction doit être testée **contre les faux négatifs**, pas seulement contre les faux positifs.

Masquer 90 % d'un token et en oublier 10 % peut être une fuite de sécurité.

## 33.4. Validation côté serveur

Une regex HTML ou JavaScript côté navigateur améliore l'UX mais ne remplace pas la validation côté serveur.

Voir [[HTML]] et [[Javascript]].

---

# 34. Portabilité entre moteurs

Une regex portable utilise un sous-ensemble volontairement limité.

## 34.1. Très portable

```text
littéraux
.
[abc]
[^abc]
^ $
* + ?
{m,n}
(...)
|
```

Même ici, les détails peuvent varier.

## 34.2. Moins portable

```text
\d \w \s
lookahead/lookbehind
groupes nommés
backreferences nommées
atomic groups
possessive quantifiers
Unicode properties
inline flags
```

## 34.3. Très spécifique

Exemples :

```text
Vim \zs / \ze
PCRE2 \K
PCRE2 récursion
Python (?P<name>...)
JavaScript (?<name>...)
JavaScript mode v
```

## 34.4. Matrice simplifiée

| Fonction | Python 3.14 | JavaScript moderne | PCRE2 | RE2 | POSIX ERE |
|---|---:|---:|---:|---:|---:|
| alternance | ✅ | ✅ | ✅ | ✅ | ✅ |
| groupes capturants | ✅ | ✅ | ✅ | ✅ | ✅ |
| groupes nommés | ✅ | ✅ | ✅ | ✅ selon API | ❌ standard |
| backreferences | ✅ | ✅ | ✅ | ❌ | selon implémentation/POSIX |
| lookahead | ✅ | ✅ | ✅ | ❌ | ❌ |
| lookbehind | ✅ avec contraintes | ✅ | ✅ | ❌ | ❌ |
| quantificateurs possessifs | ✅ | ❌ standard | ✅ | ❌ | ❌ |
| groupe atomique | ✅ | ❌ standard | ✅ | ❌ | ❌ |
| propriétés Unicode | limité via classes Python | ✅ `u`/`v` | ✅ | ✅ | dépend locale/implémentation |

Cette table est volontairement synthétique : consultons la documentation du moteur pour les détails.

---

# 35. Anti-patterns

## 35.1. Copier une regex sans comprendre son moteur

Une regex Stack Overflow marquée « PCRE » ne fonctionnera pas forcément en Python ou JavaScript.

## 35.2. Utiliser `.*` partout

```regex
^(.*),(.*),(.*)$
```

est souvent plus ambigu et coûteux que :

```regex
^([^,]*),([^,]*),([^,]*)$
```

pour un format très simple sans quoting.

## 35.3. Valider tout un format complexe en une regex

Le résultat devient :

- illisible ;
- difficile à tester ;
- fragile ;
- potentiellement lent.

## 35.4. Croire que « regex courte = regex rapide »

Le motif :

```regex
^(a+)+$
```

est très court et pourtant pathologique sur certains moteurs.

## 35.5. Oublier les entrées invalides

Une regex peut être rapide sur tous les cas positifs et catastrophique sur un cas négatif.

## 35.6. Employer des regex là où une comparaison suffit

Mauvais :

```python
re.fullmatch(r"admin", role)
```

Mieux :

```python
role == "admin"
```

## 35.7. Utiliser une regex pour une liste fixe

Mauvais :

```regex
^(GET|POST|PUT|PATCH|DELETE)$
```

si le code fait ensuite beaucoup de logique métier.

Souvent plus clair :

```python
method in {"GET", "POST", "PUT", "PATCH", "DELETE"}
```

---

# 36. Choisir entre regex et code explicite

Une bonne règle d'architecture est :

```text
motif local et textuel
    ↓
regex

structure récursive / syntaxe formelle complexe
    ↓
parseur

règle métier
    ↓
code explicite
```

Exemple de validation d'un identifiant :

```python
USERNAME = re.compile(r"[a-z][a-z0-9_-]{2,31}", re.ASCII)
```

Puis règle métier :

```python
if username in RESERVED_USERNAMES:
    raise ValueError("nom réservé")
```

La regex ne doit pas connaître tout le métier.

---

# 37. TP 1 — Bases

Pour chaque regex, donner les chaînes reconnues et non reconnues :

```regex
ab*c
ab+c
ab?c
ab{2,4}c
```

Puis tester avec Python.

Objectif : comprendre précisément le quantificateur et sa portée.

---

# 38. TP 2 — Classes et ancres

Écrire une regex correspondant à un identifiant :

- commence par une lettre ASCII minuscule ;
- contient lettres minuscules, chiffres, `_` ou `-` ;
- longueur totale de 3 à 32 caractères.

Écrire au moins :

- 8 cas valides ;
- 8 cas invalides.

Tester avec `re.fullmatch()`.

---

# 39. TP 3 — Extraction de logs

À partir de :

```text
2026-08-29T19:42:17Z ERROR user=alice request_id=abc123 timeout
```

extraire par groupes nommés :

- timestamp ;
- niveau ;
- utilisateur ;
- request ID ;
- message.

Ne pas chercher à valider complètement ISO 8601 dans la regex.

---

# 40. TP 4 — Substitution

Transformer :

```text
Launay;Michael
Becue;Gregoire
```

en :

```text
Michael Launay
Gregoire Becue
```

Le faire :

1. en Vim ;
2. avec `sed -E` ;
3. avec Python `re.sub()`.

Comparer les syntaxes.

---

# 41. TP 5 — Python et groupes nommés

Parser des identifiants :

```text
FR-2026-000001
BE-2026-000002
```

avec :

- `country` ;
- `year` ;
- `sequence`.

Puis retourner une `dataclass` Python.

---

# 42. TP 6 — Unicode

Tester avec :

```text
é
écrit sous forme décomposée
Γειά
東京
٣
👩‍💻
```

Comparer :

- `\w` en Python ;
- `[A-Za-z0-9_]` ;
- `\p{Letter}` en JavaScript avec `u` ou `v`.

Normaliser en NFC avant comparaison.

---

# 43. TP 7 — Lookarounds

Extraire uniquement les nombres :

```text
EUR 10
USD 20
EUR 30
```

qui sont précédés par `EUR`.

Écrire :

1. une solution avec lookbehind si le moteur le permet ;
2. une solution portable utilisant une capture.

Comparer leur lisibilité.

---

# 44. TP 8 — ReDoS

Étudier sans utiliser d'entrée déraisonnablement grande :

```regex
^(a+)+$
```

avec :

```text
aaaa...
```

puis :

```text
aaaa...X
```

Mesurer l'évolution du temps pour des tailles progressivement croissantes.

Refactorer le motif.

Documenter pourquoi le cas invalidant est le plus dangereux.

---

# 45. TP 9 — GNU grep

Dans un fichier de logs :

1. trouver toutes les lignes contenant littéralement `a+b(c)` avec `grep -F` ;
2. extraire les codes HTTP avec `grep -E` ;
3. comparer BRE et ERE ;
4. tester `grep -P` si PCRE est disponible.

---

# 46. TP 10 — JavaScript moderne

Écrire un script Node.js qui :

1. utilise un groupe nommé ;
2. utilise `matchAll()` ;
3. récupère les indices avec le flag `d` ;
4. utilise `RegExp.escape()` pour une chaîne utilisateur ;
5. teste une propriété Unicode avec le mode `v`.

---

# 47. TP 11 — Regex vs parseur

Choisir pour chaque cas :

- regex ;
- bibliothèque de parsing ;
- code explicite.

Cas :

1. extraire un code postal dans un log ;
2. parser JSON ;
3. renommer des variables simples ;
4. analyser un document HTML arbitraire ;
5. valider une vraie date ;
6. extraire un identifiant fixe ;
7. parser du CSV avec champs multilignes ;
8. rechercher un motif dans un dépôt Git.

Justifier chaque choix.

---

# 48. TP 12 — Audit d'une regex de production

Choisir une regex existante dans un projet et documenter :

1. moteur ;
2. intention ;
3. cas positifs ;
4. cas négatifs ;
5. Unicode ;
6. groupes ;
7. complexité probable ;
8. risque ReDoS ;
9. possibilités de simplification ;
10. tests manquants.

Produire ensuite une version améliorée et ses tests.

---

# 49. Projet final — Mini-outil d'analyse de logs

Créer un outil Python capable de lire un fichier de logs et d'extraire des événements.

## 49.1. Exemple d'entrée

```text
2026-08-29T20:00:01Z INFO user=alice action=login request_id=r001
2026-08-29T20:01:14Z ERROR user=bob action=checkout request_id=r002 code=PAYMENT_TIMEOUT
```

## 49.2. Contraintes

Le projet doit :

- utiliser une regex compilée ;
- employer des groupes nommés ;
- séparer parsing syntaxique et validation métier ;
- posséder des tests pytest ;
- traiter les lignes invalides sans planter ;
- mesurer les lignes trop longues ;
- ne pas afficher de secrets accidentellement ;
- documenter le moteur regex ;
- documenter la complexité du motif ;
- inclure un benchmark simple ;
- fournir une sortie JSON structurée.

## 49.3. Architecture suggérée

```text
logs
  ↓
lecture ligne par ligne
  ↓
regex d'extraction
  ↓
objet Event
  ↓
validation métier
  ↓
JSON / métriques / alertes
```

## 49.4. Extension

Comparer les performances et possibilités avec :

- Python `re` ;
- un moteur RE2 accessible depuis l'écosystème choisi ;
- éventuellement PCRE2.

---

# 50. Checklist d'une regex de production

Avant de fusionner une regex importante, vérifier :

- [ ] le moteur et sa version sont connus ;
- [ ] l'intention du motif est documentée ;
- [ ] recherche partielle ou full match est un choix explicite ;
- [ ] les groupes significatifs sont nommés ;
- [ ] les entrées Unicode ont été considérées ;
- [ ] les cas vides et limites sont testés ;
- [ ] les cas invalides longs sont testés ;
- [ ] aucun `.*` ambigu n'est présent sans justification ;
- [ ] les quantificateurs imbriqués ont été audités ;
- [ ] les entrées utilisateur sont échappées si elles doivent être littérales ;
- [ ] un utilisateur ne peut pas imposer une regex arbitraire sans limites ;
- [ ] les données sensibles ne sont pas envoyées dans un testeur externe ;
- [ ] la regex n'essaie pas de remplacer un vrai parseur ;
- [ ] les performances ont été mesurées si le chemin est critique ;
- [ ] les tests accompagnent le motif.

---

# 51. Aide-mémoire

## Bases

```text
.           un caractère quelconque selon le mode
^           début
$           fin selon le mode
[abc]       un caractère parmi a, b, c
[^abc]      un caractère sauf a, b, c
```

## Quantificateurs

```text
?           0 ou 1
*           0 ou plusieurs
+           1 ou plusieurs
{n}         exactement n
{n,m}       de n à m
*? +? ??    lazy dans les moteurs compatibles
*+ ++ ?+    possessif dans les moteurs compatibles
```

## Groupes

```text
(...)               capturant
(?:...)             non capturant
(?P<name>...)        nommé Python
(?<name>...)         nommé JavaScript/PCRE2
```

## Lookarounds

```text
(?=...)      lookahead positif
(?!...)      lookahead négatif
(?<=...)     lookbehind positif
(?<!...)     lookbehind négatif
```

## Python

```python
re.search()
re.match()
re.fullmatch()
re.finditer()
re.sub()
re.split()
re.compile()
re.escape()
```

## GNU grep

```bash
grep -G   # BRE
grep -E   # ERE
grep -F   # chaîne fixe
grep -P   # PCRE si disponible
```

---

# 52. Glossaire

**Ancre**
Assertion portant sur une position, par exemple début ou fin de chaîne.

**Backreference**
Référence au contenu capturé précédemment par un groupe.

**Backtracking**
Stratégie où le moteur revient sur des choix précédents afin d'essayer une autre voie.

**BRE**
Basic Regular Expressions POSIX.

**Capture**
Sous-chaîne mémorisée par un groupe capturant.

**Catastrophic backtracking**
Explosion du nombre de chemins explorés par un moteur à backtracking.

**DFA**
Deterministic Finite Automaton.

**ERE**
Extended Regular Expressions POSIX.

**Flavor / saveur**
Dialecte regex d'un moteur ou d'un outil.

**Greedy**
Quantificateur cherchant à consommer autant que possible tout en permettant le succès global.

**Lazy**
Quantificateur cherchant à consommer le minimum permettant le succès.

**Lookaround**
Assertion qui teste du contexte sans le consommer dans le match.

**NFA**
Nondeterministic Finite Automaton.

**PCRE2**
Bibliothèque « Perl-Compatible Regular Expressions », distincte du moteur Perl lui-même.

**Possessive**
Quantificateur qui ne redonne pas ses caractères au moteur lors du backtracking.

**RE2**
Moteur de Google privilégiant la sûreté et une complexité linéaire, au prix de certaines fonctionnalités.

**ReDoS**
Regular Expression Denial of Service.

**Regex**
Expression régulière au sens pratique ; selon le moteur, peut inclure des extensions allant au-delà des langages réguliers théoriques.

---

# 53. Références

Sources techniques principales :

- Python 3.14 — module `re` : <https://docs.python.org/3.14/library/re.html>
- GNU grep — Regular Expressions : <https://www.gnu.org/software/grep/manual/html_node/Regular-Expressions.html>
- PCRE2 : <https://www.pcre.org/>
- PCRE2 GitHub : <https://github.com/PCRE2Project/pcre2>
- JavaScript RegExp — MDN : <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Regular_expressions>
- JavaScript `RegExp.escape()` — MDN : <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/escape>
- Google RE2 : <https://github.com/google/re2>
- OWASP — Regular expression Denial of Service : <https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS>
- Unicode Standard : <https://www.unicode.org/standard/standard.html>
- Vim help — pattern : `:help pattern`

---

# 54. Conclusion

Les expressions régulières sont un outil extrêmement efficace lorsqu'elles sont utilisées dans leur domaine : **reconnaître et transformer des motifs textuels locaux**.

La compétence importante n'est pas de mémoriser une regex spectaculaire de plusieurs centaines de caractères. Elle consiste à savoir :

1. identifier le moteur ;
2. construire le motif progressivement ;
3. comprendre sa sémantique ;
4. tester les cas positifs et négatifs ;
5. considérer Unicode ;
6. mesurer les performances ;
7. éviter ReDoS ;
8. savoir quand passer à un parseur ou à du code explicite.

Une bonne regex doit rester **compréhensible, testable et proportionnée au problème qu'elle résout**.
