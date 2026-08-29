---
schema_version: 1
uid: 01M02EX5BRSK0FWK4QAD4RXRFB
titre: Logique
type: cours
statut: actif
para: ressource
domaines:
- enseignement
themes:
- mathematiques
- logique
- informatique-theorique
resume: 'Cours complet de logique appliquée à l’informatique : logique propositionnelle et du premier ordre, méthodes de preuve, SAT/SMT, logique temporelle et modale, logique floue, programmation logique, vérification formelle et assistants de preuve.'
niveau: intermediaire
auteurs:
- Michaël Launay
langue: fr
date_creation: 2023-06-13
date_modification: 2026-08-29
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: true
---

# Logique

La **logique** étudie les formes de raisonnement valides. En informatique, elle ne sert pas seulement à « raisonner correctement » : elle fournit un langage mathématique pour spécifier des programmes, exprimer des contraintes, vérifier des systèmes, interroger des données, construire des circuits, automatiser des preuves et formaliser le comportement d'agents.

Ce cours part de la logique propositionnelle et progresse jusqu'aux outils de vérification formelle modernes.

> [!important]
> Trois niveaux doivent toujours être distingués :
>
> 1. **syntaxe** : quelles expressions sont bien formées ;
> 2. **sémantique** : ce que ces expressions signifient dans un modèle ;
> 3. **preuve** : comment dériver formellement une conclusion à partir de prémisses.

---

# Sommaire

1. Fondements de la logique
2. Logique propositionnelle
3. Tables de vérité et équivalences
4. Formes normales CNF et DNF
5. Déduction et méthodes de preuve
6. Logique des prédicats du premier ordre
7. Théorie des ensembles, relations et fonctions
8. Induction et récursion
9. Algèbre de Boole et informatique numérique
10. SAT : satisfiabilité propositionnelle
11. SMT : satisfiabilité modulo théories
12. Programmation logique : clauses de Horn, Prolog et Datalog
13. Logique et bases de données
14. Logique de Hoare et vérification de programmes
15. Logique modale
16. Logiques temporelles LTL et CTL
17. Model checking
18. Logique intuitionniste et correspondance de Curry-Howard
19. Assistants de preuve
20. Logique floue
21. Logiques multivaluées, probabilistes et paraconsistantes
22. Logiques de description et Web sémantique
23. Logiques épistémiques, déontiques et dynamiques
24. Logique et intelligence artificielle
25. Limites de la formalisation
26. Logique informelle et erreurs de raisonnement
27. Méthode pratique de modélisation logique
28. Travaux pratiques
29. Projet final
30. Checklist, glossaire et références

---

# 1. Fondements de la logique

## 1.1 Proposition, argument et inférence

Une **proposition** est un énoncé auquel on peut attribuer une valeur de vérité dans un cadre donné.

Exemples :

- « 2 est pair » ;
- « le serveur répond » ;
- « l'utilisateur possède le rôle administrateur ».

Ne sont pas des propositions :

- une question : « le serveur répond-il ? » ;
- un ordre : « redémarre le serveur » ;
- une expression contenant une variable libre sans contexte : « x > 3 ».

Un **argument** est constitué de prémisses et d'une conclusion.

```text
P1 : Tous les humains sont mortels.
P2 : Socrate est humain.
C  : Socrate est mortel.
```

Une **inférence** est le passage des prémisses vers une conclusion.

## 1.2 Vérité et validité

Il faut distinguer :

- la **vérité** d'une proposition ;
- la **validité** d'un raisonnement.

Un argument est valide lorsque, **si ses prémisses sont vraies, sa conclusion ne peut pas être fausse**.

Un argument peut donc être formellement valide tout en ayant des prémisses fausses.

```text
Tous les poissons savent programmer.
Nemo est un poisson.
Donc Nemo sait programmer.
```

La forme logique est valide ; la première prémisse est fausse.

## 1.3 Syntaxe, sémantique et théorie de la preuve

### Syntaxe

La syntaxe définit :

- les symboles autorisés ;
- les règles permettant de construire des formules bien formées.

### Sémantique

La sémantique associe un sens aux formules.

En logique propositionnelle, cela revient principalement à attribuer `vrai` ou `faux` aux variables propositionnelles.

### Théorie de la preuve

La théorie de la preuve définit les règles formelles autorisées pour transformer des prémisses en conclusions.

Exemple :

```text
P
P → Q
─────
Q
```

Il s'agit du **modus ponens**.

## 1.4 Langage objet et métalangage

Le **langage objet** est la logique que nous étudions.

Exemple :

```text
P ∧ Q
```

Le **métalangage** est le langage que nous utilisons pour parler de cette logique.

Exemple :

> « La formule `P ∧ Q` est vraie lorsque `P` et `Q` sont vraies. »

Cette distinction devient importante en logique mathématique et en théorie des langages.

## 1.5 Conséquence sémantique

On écrit :

```text
Γ ⊨ φ
```

pour signifier :

> dans tout modèle où toutes les formules de Γ sont vraies, φ est également vraie.

On écrit :

```text
Γ ⊢ φ
```

pour indiquer que `φ` est **dérivable formellement** de `Γ` dans un système de preuve donné.

Ces deux relations ne sont pas identiques par définition :

- `⊨` relève de la sémantique ;
- `⊢` relève de la preuve formelle.

Deux propriétés fondamentales d'un calcul logique sont :

- **correction** (*soundness*) : `Γ ⊢ φ` implique `Γ ⊨ φ` ;
- **complétude** (*completeness*) : `Γ ⊨ φ` implique `Γ ⊢ φ`.

---

# 2. Logique propositionnelle

## 2.1 Variables propositionnelles

Nous utiliserons généralement :

```text
P, Q, R, ...
```

Chaque variable vaut :

```text
Vrai
```

ou :

```text
Faux
```

On les note aussi :

```text
1 / 0
⊤ / ⊥
T / F
```

## 2.2 Connecteurs fondamentaux

### Négation

```text
¬P
```

signifie « non P ».

| P | ¬P |
|---|---|
| V | F |
| F | V |

### Conjonction

```text
P ∧ Q
```

signifie « P et Q ».

| P | Q | P ∧ Q |
|---|---|---|
| V | V | V |
| V | F | F |
| F | V | F |
| F | F | F |

### Disjonction inclusive

```text
P ∨ Q
```

signifie « P ou Q, éventuellement les deux ».

| P | Q | P ∨ Q |
|---|---|---|
| V | V | V |
| V | F | V |
| F | V | V |
| F | F | F |

### Ou exclusif

```text
P ⊕ Q
```

est vrai lorsque **exactement une** des deux propositions est vraie.

| P | Q | P ⊕ Q |
|---|---|---|
| V | V | F |
| V | F | V |
| F | V | V |
| F | F | F |

### Implication

```text
P → Q
```

se lit :

> si P, alors Q.

Elle n'est fausse que lorsque `P` est vraie et `Q` fausse.

| P | Q | P → Q |
|---|---|---|
| V | V | V |
| V | F | F |
| F | V | V |
| F | F | V |

Cela surprend parfois : une implication dont l'antécédent est faux est vraie dans la logique classique.

### Équivalence

```text
P ↔ Q
```

est vraie lorsque `P` et `Q` ont la même valeur de vérité.

| P | Q | P ↔ Q |
|---|---|---|
| V | V | V |
| V | F | F |
| F | V | F |
| F | F | V |

## 2.3 Priorité des opérateurs

Une convention courante est :

1. `¬` ;
2. `∧` ;
3. `∨` ;
4. `→` ;
5. `↔`.

Il est néanmoins préférable d'utiliser explicitement des parenthèses lorsqu'une formule devient complexe.

## 2.4 Arbre syntaxique

Pour :

```text
(P ∧ Q) → ¬R
```

on peut représenter la structure comme :

```text
        →
       / \
      ∧   ¬
     / \   \
    P   Q   R
```

L'arbre syntaxique permet de distinguer la structure de l'expression de sa simple représentation textuelle.

---

# 3. Tables de vérité et équivalences

## 3.1 Construire une table complète

Pour deux variables, il faut :

```text
2² = 4
```

lignes.

Pour `n` variables :

```text
2ⁿ
```

valuations possibles.

Exemple pour :

```text
(P ∧ Q) → P
```

| P | Q | P ∧ Q | (P ∧ Q) → P |
|---|---|---|---|
| V | V | V | V |
| V | F | F | V |
| F | V | F | V |
| F | F | F | V |

La formule est toujours vraie : c'est une tautologie.

## 3.2 Tautologie

Une **tautologie** est vraie pour toutes les valuations.

Exemple :

```text
P ∨ ¬P
```

## 3.3 Contradiction

Une **contradiction** est fausse pour toutes les valuations.

```text
P ∧ ¬P
```

## 3.4 Contingence

Une **contingence** est vraie dans certaines valuations et fausse dans d'autres.

```text
P ∧ Q
```

## 3.5 Satisfiabilité

Une formule est **satisfiable** s'il existe au moins une valuation qui la rend vraie.

```text
P ∧ Q
```

est satisfiable.

```text
P ∧ ¬P
```

est insatisfiable.

## 3.6 Lois fondamentales

### Double négation

```text
¬¬P ≡ P
```

### Idempotence

```text
P ∧ P ≡ P
P ∨ P ≡ P
```

### Commutativité

```text
P ∧ Q ≡ Q ∧ P
P ∨ Q ≡ Q ∨ P
```

### Associativité

```text
(P ∧ Q) ∧ R ≡ P ∧ (Q ∧ R)
(P ∨ Q) ∨ R ≡ P ∨ (Q ∨ R)
```

### Distributivité

```text
P ∧ (Q ∨ R) ≡ (P ∧ Q) ∨ (P ∧ R)
P ∨ (Q ∧ R) ≡ (P ∨ Q) ∧ (P ∨ R)
```

### Absorption

```text
P ∨ (P ∧ Q) ≡ P
P ∧ (P ∨ Q) ≡ P
```

### Lois de De Morgan

```text
¬(P ∧ Q) ≡ ¬P ∨ ¬Q
¬(P ∨ Q) ≡ ¬P ∧ ¬Q
```

### Élimination de l'implication

```text
P → Q ≡ ¬P ∨ Q
```

### Contraposée

```text
P → Q ≡ ¬Q → ¬P
```

### Biconditionnelle

```text
P ↔ Q ≡ (P → Q) ∧ (Q → P)
```

---

# 4. Formes normales CNF et DNF

Les formes normales permettent de représenter une formule selon une structure standard.

## 4.1 Littéral

Un **littéral** est :

- une variable : `P` ;
- ou sa négation : `¬P`.

## 4.2 Clause

Une clause est souvent une disjonction de littéraux :

```text
P ∨ ¬Q ∨ R
```

## 4.3 Forme normale conjonctive — CNF

Une formule en **CNF** est une conjonction de clauses.

```text
(P ∨ Q) ∧ (¬P ∨ R) ∧ (¬Q ∨ ¬R)
```

La CNF joue un rôle central dans SAT.

## 4.4 Forme normale disjonctive — DNF

Une formule en **DNF** est une disjonction de conjonctions de littéraux.

```text
(P ∧ Q) ∨ (¬P ∧ R)
```

## 4.5 Transformation vers CNF

Méthode pédagogique :

1. éliminer `↔` ;
2. éliminer `→` ;
3. pousser les négations vers les variables avec De Morgan ;
4. distribuer `∨` sur `∧` ;
5. simplifier.

Exemple :

```text
P → (Q ∧ R)
```

devient :

```text
¬P ∨ (Q ∧ R)
```

puis :

```text
(¬P ∨ Q) ∧ (¬P ∨ R)
```

## 4.6 Transformation de Tseitin

Pour les solveurs SAT, distribuer naïvement les formules peut provoquer une explosion de taille.

La **transformation de Tseitin** introduit de nouvelles variables correspondant à des sous-formules et produit une CNF de taille linéaire par rapport à la formule initiale.

Cette CNF n'est pas nécessairement équivalente au sens strict sur toutes les nouvelles variables, mais elle est **équisatisfiable** avec la formule de départ.

---

# 5. Déduction et méthodes de preuve

## 5.1 Modus ponens

```text
P
P → Q
─────
Q
```

## 5.2 Modus tollens

```text
P → Q
¬Q
─────
¬P
```

## 5.3 Syllogisme hypothétique

```text
P → Q
Q → R
─────
P → R
```

## 5.4 Élimination de la conjonction

```text
P ∧ Q
─────
P
```

et :

```text
P ∧ Q
─────
Q
```

## 5.5 Introduction de la conjonction

```text
P
Q
─────
P ∧ Q
```

## 5.6 Preuve directe

Pour prouver :

```text
P → Q
```

on suppose `P` et on dérive `Q`.

## 5.7 Preuve par contraposée

Pour démontrer :

```text
P → Q
```

on peut démontrer :

```text
¬Q → ¬P
```

## 5.8 Preuve par contradiction

On suppose la négation de la conclusion et on montre qu'elle conduit à une contradiction.

```text
Supposons ¬P
...
Q ∧ ¬Q
Donc P
```

Cette méthode utilise la logique classique.

## 5.9 Preuve par cas

Lorsque :

```text
P ∨ Q
```

on peut démontrer la conclusion séparément dans le cas `P` puis dans le cas `Q`.

## 5.10 Déduction naturelle

La **déduction naturelle** formalise les règles d'introduction et d'élimination des connecteurs.

Par exemple pour `∧` :

```text
P   Q
───── ∧I
P ∧ Q
```

```text
P ∧ Q
───── ∧E₁
P
```

---

# 6. Logique des prédicats du premier ordre

La logique propositionnelle traite les propositions comme des blocs indivisibles.

La logique du premier ordre permet de parler :

- d'objets ;
- de propriétés ;
- de relations ;
- de quantification.

## 6.1 Termes

Un terme peut être :

- une variable : `x` ;
- une constante : `alice` ;
- une application de fonction : `mere(x)`.

## 6.2 Prédicats

Un prédicat exprime une propriété ou une relation.

```text
Humain(x)
```

```text
Aime(x, y)
```

## 6.3 Formules atomiques

```text
Humain(socrate)
```

```text
x = y
```

sont des formules atomiques.

## 6.4 Quantificateur universel

```text
∀x P(x)
```

signifie :

> pour tout x, P(x).

Exemple :

```text
∀x (Humain(x) → Mortel(x))
```

## 6.5 Quantificateur existentiel

```text
∃x P(x)
```

signifie :

> il existe au moins un x tel que P(x).

## 6.6 Variables libres et liées

Dans :

```text
∀x P(x, y)
```

- `x` est liée ;
- `y` est libre.

Une formule sans variable libre est une **phrase** ou formule close.

## 6.7 Portée d'un quantificateur

Comparez :

```text
∀x (P(x) → Q(x))
```

et :

```text
(∀x P(x)) → Q(x)
```

La portée change complètement le sens.

## 6.8 Négation des quantificateurs

```text
¬∀x P(x) ≡ ∃x ¬P(x)
```

```text
¬∃x P(x) ≡ ∀x ¬P(x)
```

Exemple :

> « Tout le monde est connecté » est faux

équivaut à :

> « Il existe au moins une personne qui n'est pas connectée. »

## 6.9 Ordre des quantificateurs

Les deux formules suivantes ne signifient pas la même chose :

```text
∀x ∃y Aime(x, y)
```

> chacun aime quelqu'un.

```text
∃y ∀x Aime(x, y)
```

> il existe quelqu'un que tout le monde aime.

## 6.10 Modèle

Un modèle du premier ordre fournit notamment :

- un domaine d'individus ;
- l'interprétation des constantes ;
- l'interprétation des fonctions ;
- l'interprétation des prédicats.

## 6.11 Limites

La logique du premier ordre est extrêmement expressive, mais il n'existe pas de procédure générale décidant la validité de **toutes** les formules du premier ordre.

Cela contraste avec la logique propositionnelle, dont le problème de satisfiabilité est décidable.

---

# 7. Théorie des ensembles, relations et fonctions

## 7.1 Ensembles

```text
A = {1, 2, 3}
```

## 7.2 Appartenance

```text
2 ∈ A
```

```text
5 ∉ A
```

## 7.3 Sous-ensemble

```text
A ⊆ B
```

signifie :

```text
∀x (x ∈ A → x ∈ B)
```

## 7.4 Union

```text
A ∪ B
```

correspond à :

```text
x ∈ A ∨ x ∈ B
```

## 7.5 Intersection

```text
A ∩ B
```

correspond à :

```text
x ∈ A ∧ x ∈ B
```

## 7.6 Différence

```text
A \ B
```

correspond à :

```text
x ∈ A ∧ x ∉ B
```

## 7.7 Complément

Par rapport à un univers `U` :

```text
Aᶜ = U \ A
```

## 7.8 Produit cartésien

```text
A × B = {(a, b) | a ∈ A ∧ b ∈ B}
```

## 7.9 Relations

Une relation binaire de `A` vers `B` est un sous-ensemble de :

```text
A × B
```

Une relation sur `A` peut être :

- réflexive ;
- symétrique ;
- antisymétrique ;
- transitive.

## 7.10 Relations d'équivalence

Une relation est une équivalence si elle est :

- réflexive ;
- symétrique ;
- transitive.

Elle partitionne l'ensemble en classes d'équivalence.

## 7.11 Ordres partiels

Une relation d'ordre partiel est :

- réflexive ;
- antisymétrique ;
- transitive.

Les systèmes de dépendances sont souvent naturellement représentés comme des ordres partiels.

## 7.12 Fonctions

Une fonction :

```text
f : A → B
```

associe à chaque élément de `A` exactement un élément de `B`.

Propriétés utiles :

- injective ;
- surjective ;
- bijective.

## 7.13 Cardinalité

Pour un ensemble fini :

```text
|A|
```

est son nombre d'éléments.

Pour les ensembles infinis, deux ensembles peuvent avoir la même cardinalité même si l'un semble être « inclus » dans l'autre.

Par exemple les entiers naturels et les entiers pairs sont équipotents.

---

# 8. Induction et récursion

## 8.1 Induction sur les entiers naturels

Pour démontrer :

```text
∀n ∈ ℕ, P(n)
```

on démontre :

1. **initialisation** : `P(0)` ;
2. **hérédité** : `P(n) → P(n+1)`.

## 8.2 Exemple

Démontrons :

```text
1 + 2 + ... + n = n(n+1)/2
```

### Cas initial

Pour `n = 1` :

```text
1 = 1×2/2
```

### Hypothèse

Supposons :

```text
1 + ... + n = n(n+1)/2
```

### Étape

Alors :

```text
1 + ... + n + (n+1)
= n(n+1)/2 + (n+1)
= (n+1)(n+2)/2
```

## 8.3 Induction forte

On suppose :

```text
P(0), ..., P(n)
```

pour prouver :

```text
P(n+1)
```

## 8.4 Induction structurelle

Elle s'applique aux structures récursives :

- arbres ;
- listes ;
- expressions syntaxiques ;
- AST.

Pour une liste :

1. prouver le cas `[]` ;
2. supposer la propriété vraie pour `xs` ;
3. la prouver pour `x :: xs`.

L'induction structurelle est fondamentale dans les preuves sur les compilateurs et les langages.

---

# 9. Algèbre de Boole et informatique numérique

## 9.1 Algèbre de Boole

Une algèbre booléenne manipule notamment :

```text
0, 1
AND
OR
NOT
```

Les lois algébriques correspondent étroitement aux équivalences de logique propositionnelle.

## 9.2 Circuits

```text
P ∧ Q
```

peut être implémenté par une porte AND.

```text
¬P
```

par une porte NOT.

## 9.3 NAND et NOR

NAND et NOR sont **fonctionnellement complets** : il est possible de construire toutes les fonctions booléennes uniquement avec l'un de ces types de portes.

## 9.4 Simplification

Les lois de Boole permettent de réduire des circuits.

Exemple :

```text
P ∧ (P ∨ Q)
```

se simplifie en :

```text
P
```

Voir également [[Systèmes numériques]].

---

# 10. SAT : satisfiabilité propositionnelle

## 10.1 Le problème SAT

SAT demande :

> existe-t-il une valuation rendant une formule propositionnelle vraie ?

Exemple :

```text
(P ∨ Q) ∧ (¬P ∨ R)
```

est satisfiable.

## 10.2 SAT et complexité

SAT est historiquement le premier problème démontré **NP-complet**.

Cela ne signifie pas que toutes les instances pratiques sont difficiles : les solveurs modernes résolvent fréquemment des formules comportant des millions de variables et clauses lorsque leur structure est favorable.

## 10.3 DPLL

L'algorithme DPLL repose sur :

- choix d'une variable ;
- propagation unitaire ;
- backtracking ;
- élimination des littéraux purs dans certaines variantes.

## 10.4 CDCL

Les solveurs SAT modernes emploient souvent **CDCL** (*Conflict-Driven Clause Learning*).

Idée : lorsqu'une branche conduit à une contradiction, le solveur analyse le conflit et apprend une nouvelle clause empêchant de répéter la même erreur.

## 10.5 Usages

SAT est utilisé pour :

- vérification matérielle ;
- planification ;
- configuration de produits ;
- ordonnancement ;
- analyse de dépendances ;
- cryptanalyse ;
- vérification logicielle.

---

# 11. SMT : satisfiabilité modulo théories

SAT manipule essentiellement des booléens.

SMT ajoute des **théories** comme :

- arithmétique entière ;
- arithmétique réelle ;
- bit-vectors ;
- tableaux ;
- chaînes ;
- nombres flottants ;
- types algébriques.

## 11.1 Exemple

Nous voulons satisfaire :

```text
x > 5
x < 10
y = x + 2
```

Un solveur SMT peut raisonner directement sur ces contraintes.

## 11.2 Z3 avec Python

```python
from z3 import Int, Solver, sat

x = Int("x")
y = Int("y")

solver = Solver()
solver.add(x > 5)
solver.add(x < 10)
solver.add(y == x + 2)

if solver.check() == sat:
    print(solver.model())
```

## 11.3 Contraintes logicielles

Exemple :

```text
version >= 3
architecture ∈ {amd64, arm64}
feature_x → version >= 5
```

SAT/SMT conviennent très bien à ce type de problème.

## 11.4 SMT-LIB

**SMT-LIB** définit un langage standard pour exprimer des problèmes SMT.

Exemple :

```lisp
(set-logic QF_LIA)
(declare-const x Int)
(assert (> x 5))
(assert (< x 10))
(check-sat)
(get-model)
```

> [!note]
> La version de référence publiée de SMT-LIB reste 2.6 ; il faut distinguer le standard du rythme d'évolution des solveurs qui l'implémentent.

---

# 12. Programmation logique : clauses de Horn, Prolog et Datalog

## 12.1 Clause de Horn

Une clause de Horn contient au plus un littéral positif.

Exemple :

```text
Humain(x) ∧ Humain(y) ∧ Parent(x, y) → Ancetre(x, y)
```

## 12.2 Prolog

Exemple :

```prolog
parent(alice, bob).
parent(bob, claire).

ancestor(X, Y) :- parent(X, Y).
ancestor(X, Y) :- parent(X, Z), ancestor(Z, Y).
```

Requête :

```prolog
?- ancestor(alice, claire).
```

## 12.3 Unification

L'**unification** cherche une substitution permettant de rendre deux termes identiques.

Exemple :

```text
f(x, a)
f(b, y)
```

peuvent être unifiés avec :

```text
x = b
y = a
```

## 12.4 Résolution

La résolution est une règle d'inférence utilisée notamment en démonstration automatique et en programmation logique.

## 12.5 Datalog

Datalog est un sous-langage logique particulièrement adapté :

- aux règles ;
- aux graphes ;
- à l'analyse statique ;
- aux politiques de sécurité ;
- à certaines requêtes récursives.

Contrairement à Prolog général, Datalog restreint généralement les termes afin d'obtenir de bonnes propriétés de terminaison et de calcul.

---

# 13. Logique et bases de données

Voir également [[Bases de données relationnelles]].

## 13.1 Modèle relationnel

Le modèle relationnel utilise notamment :

- ensembles ;
- tuples ;
- relations ;
- logique du premier ordre.

Il ne faut cependant pas confondre le **modèle relationnel mathématique** avec le comportement exact de SQL.

## 13.2 Calcul relationnel

Le calcul relationnel permet d'exprimer ce que l'on veut obtenir plutôt que la procédure permettant de l'obtenir.

## 13.3 SQL et logique à trois valeurs

SQL introduit :

```text
TRUE
FALSE
UNKNOWN
```

à cause notamment de `NULL`.

Exemple :

```sql
SELECT NULL = 5;
```

ne produit pas `FALSE`, mais `UNKNOWN`.

D'où :

```sql
WHERE column = NULL
```

est incorrect pour tester `NULL`.

Il faut utiliser :

```sql
WHERE column IS NULL
```

## 13.4 De Morgan et NULL

En SQL, transformer mécaniquement une expression booléenne selon l'intuition de la logique bivalente peut surprendre lorsque `UNKNOWN` intervient.

Il faut donc comprendre la sémantique SQL, pas seulement connaître les symboles logiques.

---

# 14. Logique de Hoare et vérification de programmes

## 14.1 Triple de Hoare

```text
{P} C {Q}
```

signifie :

> si la précondition `P` est vraie avant l'exécution de `C`, alors, si `C` termine, la postcondition `Q` sera vraie après l'exécution.

## 14.2 Exemple

```text
{x = 1}
y := x + 1
{y = 2}
```

## 14.3 Précondition la plus faible

La **weakest precondition** d'une commande pour une postcondition donnée est la condition minimale nécessaire avant l'exécution pour garantir cette postcondition.

## 14.4 Invariant de boucle

Pour prouver une boucle correcte, il faut souvent identifier un invariant : une propriété vraie :

- avant la première itération ;
- après chaque itération ;
- et permettant de démontrer la postcondition à la sortie.

## 14.5 Correction partielle et totale

**Correction partielle** :

> si le programme termine, le résultat est correct.

**Correction totale** :

> le programme termine **et** le résultat est correct.

La terminaison exige souvent un **variant**, c'est-à-dire une quantité bien fondée qui décroît à chaque itération.

---

# 15. Logique modale

## 15.1 Modalités

Deux opérateurs classiques :

```text
□P
```

> nécessairement P.

```text
◇P
```

> possiblement P.

Ils sont reliés par :

```text
◇P ≡ ¬□¬P
```

## 15.2 Sémantique de Kripke

Un modèle de Kripke contient :

- un ensemble de mondes possibles ;
- une relation d'accessibilité entre mondes ;
- une valuation des propositions dans chaque monde.

```text
w1 ───> w2
 │
 └────> w3
```

`□P` est vraie dans `w1` si `P` est vraie dans tous les mondes accessibles depuis `w1`.

## 15.3 Systèmes modaux

Différentes propriétés de la relation d'accessibilité donnent différents systèmes modaux.

Exemples :

- K ;
- T ;
- S4 ;
- S5.

## 15.4 Informatique

Les logiques modales servent à raisonner sur :

- connaissances ;
- croyances ;
- permissions ;
- transitions ;
- systèmes distribués ;
- programmes.

---

# 16. Logiques temporelles LTL et CTL

Les logiques temporelles permettent de formuler des propriétés sur l'évolution d'un système.

## 16.1 LTL

**Linear Temporal Logic** raisonne sur des chemins d'exécution linéaires.

Opérateurs principaux :

```text
X P
```

> P au prochain état.

```text
F P
```

> P sera vraie un jour.

```text
G P
```

> P sera toujours vraie.

```text
P U Q
```

> P reste vraie jusqu'à ce que Q devienne vraie.

## 16.2 Propriété de sûreté

```text
G ¬collision
```

> une collision ne doit jamais se produire.

Intuition :

> « quelque chose de mauvais n'arrive jamais ».

## 16.3 Propriété de vivacité

```text
G(request → F response)
```

> chaque requête finira par recevoir une réponse.

Intuition :

> « quelque chose de bon finit par se produire ».

## 16.4 CTL

CTL ajoute des quantificateurs sur les chemins :

- `A` : pour tous les chemins ;
- `E` : il existe un chemin.

Exemples :

```text
AG P
```

> sur tous les chemins, P reste toujours vraie.

```text
EF P
```

> il existe un chemin où P finit par devenir vraie.

## 16.5 LTL vs CTL

LTL et CTL ne sont pas deux syntaxes interchangeables : elles expriment les propriétés temporelles selon des modèles différents.

CTL* généralise les deux familles.

---

# 17. Model checking

## 17.1 Principe

Le **model checking** consiste à :

1. construire un modèle fini ou symbolique d'un système ;
2. exprimer une propriété formelle ;
3. vérifier automatiquement que le modèle satisfait cette propriété.

## 17.2 Contre-exemple

Si une propriété est fausse, un model checker peut souvent fournir une trace d'exécution expliquant l'échec.

```text
état 0
  ↓
état 3
  ↓
état 7
  ↓
violation
```

C'est extrêmement utile pour le diagnostic.

## 17.3 Explosion d'états

Un système composé de nombreux composants concurrents peut produire un nombre gigantesque d'états.

Techniques utilisées :

- représentation symbolique ;
- BDD ;
- SAT/SMT ;
- réduction d'ordre partiel ;
- abstraction.

## 17.4 Outils

Exemples historiques ou actuels :

- SPIN ;
- NuSMV / nuXmv ;
- TLA+ / TLC ;
- UPPAAL ;
- CBMC pour certains usages liés au C.

---

# 18. Logique intuitionniste et correspondance de Curry-Howard

## 18.1 Différence avec la logique classique

La logique intuitionniste ne considère pas qu'une proposition est établie uniquement parce que sa négation conduit à une contradiction.

Le tiers exclu :

```text
P ∨ ¬P
```

n'est pas accepté comme principe général sans preuve constructive de l'une des branches.

## 18.2 Preuve constructive

Pour prouver :

```text
∃x P(x)
```

une preuve constructive doit fournir un témoin `x` et une preuve de `P(x)`.

## 18.3 Curry-Howard

La correspondance de Curry-Howard relie :

```text
proposition  ↔ type
preuve       ↔ programme
```

Par exemple :

```text
P → Q
```

correspond conceptuellement à un type de fonction :

```text
P -> Q
```

Une preuve est alors une valeur de ce type.

## 18.4 Applications

Cette correspondance sous-tend de nombreux assistants de preuve et langages à types dépendants.

---

# 19. Assistants de preuve

## 19.1 Principe

Un assistant de preuve permet d'écrire :

- définitions ;
- propositions ;
- preuves ;
- tactiques d'automatisation.

La preuve finale est vérifiée par un noyau logique.

## 19.2 Lean

Lean 4 est à la fois :

- un assistant de preuve interactif ;
- un langage fonctionnel ;
- un environnement basé sur la théorie des types dépendants.

Au 29 août 2026, la branche stable récente est Lean 4.33.x, tandis que 4.34 est encore en release candidate.

Exemple :

```lean
example (P Q : Prop) (h₁ : P) (h₂ : P → Q) : Q := by
  exact h₂ h₁
```

## 19.3 Tactiques

Une tactique aide à construire un terme de preuve.

Exemple :

```lean
example (P Q : Prop) : P ∧ Q → P := by
  intro h
  exact h.left
```

## 19.4 Autres assistants

Exemples :

- Coq ;
- Isabelle/HOL ;
- Agda ;
- HOL4 ;
- F*.

Ils diffèrent notamment par :

- leur logique ;
- leur théorie des types ;
- leur noyau ;
- leur automatisation ;
- leur écosystème.

## 19.5 Noyau de confiance

Un intérêt majeur est de réduire la **Trusted Computing Base**.

Idéalement, même si une tactique complexe contient un bug, elle ne doit pas pouvoir faire accepter une preuve incorrecte si le noyau vérifie chaque terme final.

> [!warning]
> Un assistant de preuve ne garantit pas que la **spécification** correspond au besoin réel. Il garantit uniquement ce qui a été formalisé et effectivement prouvé.

---

# 20. Logique floue

La **logique floue** ne doit pas être confondue avec la probabilité.

## 20.1 Degré d'appartenance

Un ensemble flou associe à un élément un degré :

```text
μ_A(x) ∈ [0, 1]
```

Exemple : pour l'ensemble « températures chaudes » :

```text
μ_chaud(18°C) = 0.1
μ_chaud(25°C) = 0.6
μ_chaud(32°C) = 0.95
```

Cela ne signifie pas :

> « il y a 60 % de chances que 25 °C soit chaud ».

Il s'agit d'un degré d'appartenance à une catégorie vague.

## 20.2 Opérations simples

Une convention courante est :

```text
μ_{A∩B}(x) = min(μ_A(x), μ_B(x))
```

```text
μ_{A∪B}(x) = max(μ_A(x), μ_B(x))
```

```text
μ_{¬A}(x) = 1 - μ_A(x)
```

Ce ne sont pas les seules familles possibles d'opérateurs flous.

## 20.3 Système de contrôle flou

Étapes typiques :

1. fuzzification ;
2. application des règles ;
3. agrégation ;
4. défuzzification.

Exemple de règle :

```text
SI température EST froide
ET humidité EST forte
ALORS chauffage EST élevé
```

## 20.4 Usages

- contrôle industriel ;
- automatisme ;
- systèmes embarqués ;
- décision multicritère ;
- traitement de notions linguistiques graduelles.

---

# 21. Logiques multivaluées, probabilistes et paraconsistantes

## 21.1 Logiques multivaluées

Elles possèdent plus de deux valeurs de vérité.

SQL constitue un exemple pratique célèbre avec :

```text
TRUE
FALSE
UNKNOWN
```

## 21.2 Logique probabiliste

Une probabilité mesure une incertitude sur des événements ou hypothèses.

```text
P(pluie demain) = 0.7
```

Cela n'est pas équivalent à un degré d'appartenance flou.

## 21.3 Logiques paraconsistantes

Dans la logique classique, une contradiction peut conduire au principe d'explosion :

```text
P ∧ ¬P ⊢ Q
```

pour n'importe quel `Q`.

Les logiques paraconsistantes cherchent à permettre un raisonnement utile même lorsque certaines informations sont contradictoires.

Elles peuvent être intéressantes lorsque les données proviennent de sources multiples imparfaites.

---

# 22. Logiques de description et Web sémantique

## 22.1 Objectif

Les **logiques de description** représentent :

- concepts ;
- rôles ;
- individus ;
- relations hiérarchiques.

Elles sont conçues pour offrir un compromis entre :

- expressivité ;
- décidabilité ;
- efficacité du raisonnement.

## 22.2 Exemple conceptuel

```text
Chat ⊆ Mammifère
Mammifère ⊆ Animal
```

Un raisonneur peut déduire :

```text
Chat ⊆ Animal
```

## 22.3 OWL

OWL, utilisé dans le Web sémantique, s'appuie sur des logiques de description pour certaines de ses variantes et profils.

Applications :

- ontologies ;
- connaissances structurées ;
- classification ;
- vérification de cohérence.

---

# 23. Logiques épistémiques, déontiques et dynamiques

## 23.1 Logique épistémique

Elle formalise les connaissances ou croyances.

```text
K_A P
```

peut se lire :

> l'agent A sait que P.

Utilisations :

- protocoles distribués ;
- jeux ;
- systèmes multi-agents ;
- raisonnement sur l'information.

## 23.2 Logique déontique

Elle formalise :

- obligation ;
- permission ;
- interdiction.

Exemple conceptuel :

```text
O(chiffrer_données)
```

> il est obligatoire de chiffrer les données.

Elle peut être utile pour formaliser certaines politiques, mais traduire directement du droit en logique reste délicat.

## 23.3 Logique dynamique

La logique dynamique raisonne sur les effets des programmes ou actions.

Notation typique :

```text
[α]P
```

> après toute exécution de l'action ou du programme α, P est vraie.

---

# 24. Logique et intelligence artificielle

## 24.1 IA symbolique

Historiquement, une grande partie de l'IA reposait sur :

- règles ;
- systèmes experts ;
- planification ;
- représentation de connaissances ;
- démonstration automatique.

## 24.2 Raisonnement et LLM

Un LLM génère des tokens à partir d'un modèle probabiliste appris.

Il ne faut pas confondre :

```text
texte ressemblant à une preuve
```

et :

```text
preuve formellement vérifiée
```

Un modèle peut produire un raisonnement convaincant mais invalide.

## 24.3 LLM + solveur

Une architecture plus robuste peut être :

```text
Utilisateur
   ↓
LLM
   ↓
formalisation de contraintes
   ↓
SAT / SMT / assistant de preuve
   ↓
validation
   ↓
réponse
```

Le modèle sert alors d'interface ou de générateur de conjectures, tandis qu'un outil symbolique vérifie certains résultats.

## 24.4 Agents

Dans un agent IA, la logique peut servir à :

- exprimer les préconditions d'un outil ;
- vérifier des politiques ;
- représenter des dépendances ;
- satisfaire des contraintes de planification ;
- valider un état avant action.

## 24.5 Limite fondamentale

Un système logique n'améliore pas une prémisse fausse.

```text
raisonnement parfait
+
données fausses
=
conclusion potentiellement fausse
```

La qualité de la connaissance et de la formalisation reste donc essentielle.

---

# 25. Limites de la formalisation

## 25.1 Décidabilité

Un problème est **décidable** s'il existe un algorithme qui termine toujours et répond correctement oui/non.

## 25.2 Semi-décidabilité

Un problème peut être reconnaissable sans être décidable : l'algorithme peut terminer lorsque la réponse est oui, mais ne jamais terminer dans certains cas négatifs.

## 25.3 Problème de l'arrêt

Il n'existe pas d'algorithme général permettant de déterminer pour tout programme et toute entrée si le programme terminera.

## 25.4 Théorèmes d'incomplétude de Gödel

Très schématiquement, pour certains systèmes formels suffisamment expressifs et cohérents pour formaliser l'arithmétique :

- il existe des énoncés vrais dans le modèle standard qui ne sont pas démontrables dans le système ;
- un tel système ne peut pas démontrer sa propre cohérence sous les hypothèses usuelles du théorème.

> [!warning]
> Les théorèmes de Gödel ne signifient pas « tout est relatif » ni « on ne peut rien prouver ».

## 25.5 Théorème de complétude du premier ordre

Il ne faut pas confondre l'incomplétude de Gödel avec la **complétude de la logique du premier ordre** de Gödel :

```text
Γ ⊨ φ  implique  Γ ⊢ φ
```

pour une axiomatisation correcte de la logique du premier ordre.

La tension apparente disparaît lorsqu'on distingue :

- la logique elle-même ;
- les théories particulières exprimées dans cette logique.

## 25.6 Expressivité vs décidabilité

Plus une logique devient expressive, plus le raisonnement automatique peut devenir difficile ou indécidable.

Il existe donc un compromis fréquent :

```text
expressivité
    ↕
complexité du raisonnement
```

C'est l'une des raisons pour lesquelles il existe de nombreuses logiques spécialisées.

---

# 26. Logique informelle et erreurs de raisonnement

La logique formelle ne couvre pas toutes les erreurs argumentatives du langage naturel.

## 26.1 Affirmation du conséquent

```text
Si P alors Q.
Q.
Donc P.
```

Invalide.

Exemple :

```text
S'il pleut, la rue est mouillée.
La rue est mouillée.
Donc il pleut.
```

Il peut exister une autre cause.

## 26.2 Négation de l'antécédent

```text
P → Q
¬P
─────
¬Q
```

Invalide.

## 26.3 Corrélation et causalité

```text
A corrèle avec B
```

n'implique pas :

```text
A cause B
```

## 26.4 Faux dilemme

Présenter seulement deux options alors que d'autres existent.

## 26.5 Généralisation hâtive

Déduire une règle générale à partir d'un échantillon insuffisant.

## 26.6 Appel à l'autorité

La parole d'un expert constitue un élément de preuve possible, pas une démonstration logique absolue.

## 26.7 Ad hominem

Attaquer la personne plutôt que l'argument.

## 26.8 Homme de paille

Déformer l'argument d'un interlocuteur afin d'en réfuter une version plus faible.

## 26.9 Biais et validité

Un argument peut être psychologiquement convaincant sans être formellement valide.

Inversement, un argument valide peut être difficile à accepter intuitivement.

---

# 27. Méthode pratique de modélisation logique

Pour formaliser un problème réel :

## Étape 1 — définir le domaine

Quels objets existent ?

```text
Utilisateur
Serveur
Rôle
Ressource
```

## Étape 2 — identifier les propositions ou prédicats

```text
Authentifié(u)
Admin(u)
PeutLire(u, r)
```

## Étape 3 — séparer faits et règles

Faits :

```text
Admin(alice)
```

Règles :

```text
∀u (Admin(u) → PeutLire(u, rapport))
```

## Étape 4 — choisir la bonne logique

| Besoin | Outil logique possible |
|---|---|
| contraintes booléennes | SAT |
| entiers, bit-vectors, chaînes | SMT |
| objets et relations | logique du premier ordre |
| temps et transitions | LTL / CTL |
| programmes | Hoare / logique dynamique |
| ontologies | logique de description |
| preuves vérifiées | théorie des types / assistant de preuve |
| catégories graduelles | logique floue |

## Étape 5 — préciser les hypothèses

Une formalisation doit expliciter les hypothèses cachées.

Exemple :

```text
Un utilisateur ne possède qu'un rôle.
```

Est-ce réellement vrai ?

## Étape 6 — chercher les contre-exemples

Avant de chercher à prouver une propriété, essayer activement de la falsifier.

## Étape 7 — automatiser lorsque cela apporte de la valeur

Une formule simple peut être vérifiée à la main.

Un système contenant :

- centaines de contraintes ;
- états concurrents ;
- arithmétique ;
- règles croisées ;

bénéficie d'un solveur.

## Étape 8 — valider le modèle métier

Le solveur peut démontrer :

```text
modèle ⊨ propriété
```

mais ne peut pas garantir :

```text
modèle = réalité souhaitée
```

Cette seconde validation reste humaine et métier.

---

# 28. Travaux pratiques

## TP 1 — Tables de vérité

Construire les tables de vérité de :

```text
P → Q
¬Q → ¬P
(P → Q) ↔ (¬Q → ¬P)
```

Montrer que la troisième formule est une tautologie.

---

## TP 2 — Détecter les raisonnements invalides

Pour chacun des raisonnements suivants :

1. formaliser les propositions ;
2. indiquer si l'inférence est valide ;
3. fournir un contre-exemple si elle ne l'est pas.

```text
Si le service est en panne, l'alerte se déclenche.
L'alerte est déclenchée.
Donc le service est en panne.
```

---

## TP 3 — CNF

Transformer en CNF :

```text
(P → Q) ∧ (Q → R)
```

puis :

```text
P → (Q ∧ R)
```

---

## TP 4 — Logique des prédicats

Formaliser :

1. tous les administrateurs sont des utilisateurs ;
2. certains utilisateurs ne sont pas administrateurs ;
3. chaque projet possède au moins un responsable ;
4. il existe un responsable qui gère tous les projets.

Comparer attentivement les formules 3 et 4.

---

## TP 5 — Ensembles et relations

Soit :

```text
A = {1, 2, 3}
B = {2, 3, 4}
```

Calculer :

```text
A ∪ B
A ∩ B
A \ B
B \ A
A × B
```

Puis définir une relation `R` telle que :

```text
x R y  ssi  x < y
```

Étudier réflexivité, symétrie, antisymétrie et transitivité.

---

## TP 6 — SAT de configuration

Nous avons trois options :

```text
A : cache Redis
B : haute disponibilité
C : stockage local
```

Contraintes :

```text
B → A
C → ¬B
A ∨ C
```

Déterminer toutes les configurations satisfaisantes.

---

## TP 7 — Z3

Installer :

```bash
python -m pip install z3-solver
```

Résoudre :

```text
x + y = 20
x > 0
y > 0
x < y
x est pair
```

Écrire un programme Python affichant plusieurs modèles possibles.

---

## TP 8 — SQL et logique à trois valeurs

Créer une table :

```sql
CREATE TABLE test (
    id INTEGER PRIMARY KEY,
    score INTEGER
);
```

Insérer :

```sql
INSERT INTO test VALUES
(1, 10),
(2, NULL),
(3, 20);
```

Comparer :

```sql
WHERE score = NULL
```

```sql
WHERE score IS NULL
```

et expliquer le résultat à l'aide de `UNKNOWN`.

---

## TP 9 — Invariant de boucle

Pour :

```python
def somme(n):
    total = 0
    i = 0
    while i <= n:
        total += i
        i += 1
    return total
```

Proposer un invariant permettant de démontrer :

```text
résultat = n(n+1)/2
```

---

## TP 10 — LTL

Pour un système de file de messages, formaliser :

1. un message accepté finit par être traité ;
2. un message traité ne redevient jamais « non traité » ;
3. le service ne doit jamais traiter un message supprimé.

Classifier chaque propriété comme sûreté ou vivacité lorsque cela est pertinent.

---

## TP 11 — Lean

Écrire avec Lean :

```text
P ∧ Q → Q ∧ P
```

Puis :

```text
(P → Q) → (Q → R) → (P → R)
```

Sans automatisation excessive : comprendre chaque introduction et élimination.

---

## TP 12 — Audit de raisonnement d'un agent IA

Prendre une réponse générée par un LLM contenant une argumentation technique.

1. isoler les prémisses ;
2. isoler la conclusion ;
3. distinguer faits, hypothèses et inférences ;
4. rechercher les sauts logiques ;
5. formaliser une partie en logique propositionnelle ou SMT ;
6. vérifier la partie formalisable ;
7. documenter ce qui reste non formalisé.

L'objectif est de comprendre la différence entre :

```text
raisonnement plausible
```

et :

```text
raisonnement formellement vérifié
```

---

# 29. Projet final — Vérification d'un système d'autorisation

## Objectif

Concevoir puis vérifier un petit système d'autorisation.

## Domaine

Nous avons :

- des utilisateurs ;
- des ressources ;
- des rôles ;
- des niveaux de confidentialité.

Rôles :

```text
viewer
editor
admin
```

Niveaux :

```text
public
internal
secret
```

## Règles

Exemples :

```text
viewer peut lire public
editor peut modifier public et internal
admin peut tout administrer
```

Mais :

```text
un compte désactivé ne doit effectuer aucune action
```

et :

```text
une ressource secret exige une authentification forte
```

## Travail demandé

### Partie 1 — Modélisation

Définir :

- prédicats ;
- domaines ;
- règles ;
- invariants.

### Partie 2 — Propositionnel

Formaliser une version simplifiée avec des variables booléennes.

### Partie 3 — SMT

Ajouter :

- niveaux numériques ;
- contraintes de rôle ;
- état activé/désactivé.

Utiliser Z3 pour rechercher une configuration violant une propriété.

### Partie 4 — Temporel

Exprimer :

```text
un compte désactivé ne doit jamais effectuer une nouvelle action
```

sous forme de propriété temporelle conceptuelle.

### Partie 5 — Tests

Transformer les invariants en tests automatisés.

### Partie 6 — Rapport

Documenter :

- les hypothèses ;
- ce qui a été prouvé ;
- ce qui n'a pas été prouvé ;
- les limites du modèle.

> [!important]
> Une bonne conclusion n'est pas « le système est sécurisé », mais par exemple :
>
> « Dans le modèle M, sous les hypothèses H, la propriété P est satisfaite. »

---

# 30. Checklist, glossaire et références

## 30.1 Checklist de raisonnement

Avant d'accepter une conclusion :

- [ ] Les prémisses sont-elles explicites ?
- [ ] Les termes sont-ils définis sans ambiguïté ?
- [ ] L'inférence est-elle valide ?
- [ ] Les prémisses sont-elles réellement vraies ?
- [ ] Existe-t-il un contre-exemple ?
- [ ] Les quantificateurs sont-ils dans le bon ordre ?
- [ ] Des hypothèses implicites ont-elles été oubliées ?
- [ ] Le modèle correspond-il au domaine réel ?
- [ ] Utilise-t-on la bonne logique ?
- [ ] Le problème est-il décidable dans le fragment choisi ?

## 30.2 Checklist pour une spécification formelle

- [ ] définir les états ;
- [ ] définir les transitions ;
- [ ] définir les invariants ;
- [ ] distinguer sûreté et vivacité ;
- [ ] préciser les hypothèses d'environnement ;
- [ ] formaliser les erreurs possibles ;
- [ ] rechercher activement des contre-exemples ;
- [ ] conserver une traçabilité entre exigence métier et formule ;
- [ ] versionner la spécification ;
- [ ] valider la formalisation avec les experts métier.

## 30.3 Glossaire

**Axiome**
Proposition adoptée comme point de départ dans un système formel.

**Clause**
Disjonction de littéraux dans le contexte SAT/CNF.

**Complétude**
Propriété reliant conséquence sémantique et dérivabilité syntaxique selon le système considéré.

**Contradiction**
Formule fausse pour toutes les valuations.

**Décidable**
Problème pour lequel un algorithme correct termine toujours.

**DNF**
Forme normale disjonctive.

**CNF**
Forme normale conjonctive.

**Formule**
Expression bien formée d'un langage logique.

**Invariant**
Propriété préservée au cours de l'exécution ou des transitions d'un système.

**Littéral**
Variable propositionnelle ou sa négation.

**Modèle**
Structure donnant une interprétation sémantique aux symboles d'une logique.

**SAT**
Problème de satisfiabilité de formules propositionnelles.

**SMT**
Satisfiabilité modulo théories.

**Satisfiable**
Vraie dans au moins un modèle ou une valuation.

**Tautologie**
Vraie dans toutes les valuations.

**Théorème**
Formule démontrable dans le système considéré.

**Valide**
Vraie dans tous les modèles pertinents, selon le contexte logique.

## 30.4 Références et outils

Pour approfondir :

- Stanford Encyclopedia of Philosophy — entrées sur les différentes logiques ;
- *Introduction to Mathematical Logic* ;
- *Logic in Computer Science* de Huth et Ryan ;
- SMT-LIB — standard d'échange pour solveurs SMT ;
- Z3 — solveur SMT de Microsoft Research ;
- Lean — assistant de preuve et langage ;
- TLA+ — spécification et vérification de systèmes concurrents ;
- SPIN — model checking ;
- SWI-Prolog — programmation logique.

## 30.5 Liens avec les autres cours

- [[Informatique]]
- [[Systèmes numériques]]
- [[Bases de données relationnelles]]
- [[Architecture des logiciels]]
- [[Python]]
- [[Sécurité avec Python]]
- [[LLM]]

---

# Conclusion

La logique informatique ne se réduit pas aux opérateurs `AND`, `OR` et `NOT`.

Elle relie :

```text
raisonnement
    ↓
spécification
    ↓
algorithmes
    ↓
programmes
    ↓
preuves
    ↓
vérification automatique
```

La compétence la plus importante n'est pas de mémoriser toutes les notations, mais de savoir :

1. identifier précisément ce que l'on veut exprimer ;
2. choisir le formalisme adapté ;
3. distinguer vérité, validité et preuve ;
4. expliciter les hypothèses ;
5. chercher les contre-exemples ;
6. connaître les limites de la formalisation ;
7. utiliser les solveurs et assistants de preuve comme **outils de vérification**, pas comme substituts à une bonne modélisation.
