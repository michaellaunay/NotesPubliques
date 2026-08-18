---
schema_version: 1
uid: 01M02JG1WQ9ZB9VDNT9KP5QF2P
titre: Écart type
aliases:
- Déviation standard
type: fiche
statut: actif
para: ressource
domaines:
- enseignement
themes:
- mathematiques
- statistiques
- dispersion
resume: Définition de l'écart type et méthode de calcul à partir de la variance.
auteurs:
- Michaël Launay
langue: fr
date_creation: 2024-10-12
date_modification: 2026-08-18
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: false
---
L'**écart type** (ou **déviation standard**) est une mesure statistique qui exprime à quel point les valeurs d'un ensemble de données sont dispersées par rapport à la moyenne. Il est étroitement lié à la [[Variance]], car l'écart type est simplement la racine carrée de la variance.

# Formule de l'écart type
L'écart type \( \sigma \) pour un ensemble de données est calculé comme suit :
$$
\sigma = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (x_i - \mu)^2}
$$
- \( x_i \) représente chaque valeur dans l'ensemble de données,
- \( \mu \) est la moyenne des valeurs,
- \( n \) est le nombre total de données.

# Étapes pour le calcul de l'écart type
1. **Calculer la moyenne** (\( \mu \)).
2. **Soustraire la moyenne de chaque valeur** pour obtenir les écarts.
3. **Élever ces écarts au carré**.
4. **Moyenne des carrés des écarts** : ceci donne la variance.
5. **Prendre la racine carrée** de la variance pour obtenir l'écart type.

# Interprétation
L'écart type est plus facile à interpréter que la variance, car il est exprimé dans les mêmes unités que les données d'origine (contrairement à la variance, qui est en unités au carré).

- Un **écart type faible** signifie que les données sont proches de la moyenne, donc peu dispersées.
- Un **écart type élevé** indique que les données sont très dispersées autour de la moyenne.

# Exemple
Si on reprend les données \( 2, 4, 6, 8, 10 \), la variance calculée est \( 8 \). L'écart type sera donc la racine carrée de cette variance :

$$
\sigma = \sqrt{8} \approx 2.83
$$


L'écart type donne une idée plus intuitive de la "distance" typique des valeurs par rapport à la moyenne.

# Utilité de l'écart type
L'écart type est couramment utilisé dans plusieurs domaines pour évaluer la variabilité, comme :
- **Finance** : pour mesurer la volatilité des prix d'actifs.
- **Statistiques** : pour quantifier l'incertitude autour d'une moyenne.
- **Physique** : pour évaluer les écarts par rapport à une valeur théorique.