La **variance** est une mesure statistique qui quantifie la dispersion d'un ensemble de données par rapport à sa moyenne. Autrement dit, elle indique à quel point les valeurs d'un ensemble de données diffèrent en moyenne de la moyenne de cet ensemble.

### Calcul de la variance
La variance \( \sigma^2 \) d'un ensemble de données se calcule en plusieurs étapes :

1. **Calcul de la moyenne** (\( \mu \)) :
   $$
   \mu = \frac{1}{n} \sum_{i=1}^{n} x_i
   $$
   où \( x_i \) représente chaque valeur des données, et \( n \) est le nombre total de données.

2. **Différence entre chaque valeur et la moyenne** :
   On soustrait la moyenne à chaque valeur du jeu de données, soit \( x_i - \mu \).

3. **Carré de ces différences** :
   On élève chaque différence au carré pour éliminer les valeurs négatives.

4. **Moyenne des carrés des différences** :
   Enfin, on fait la moyenne des carrés des différences :
   $$
   \sigma^2 = \frac{1}{n} \sum_{i=1}^{n} (x_i - \mu)^2
   $$

### Interprétation
- Une variance **faible** signifie que les valeurs sont proches de la moyenne, donc peu dispersées.
- Une variance **élevée** indique que les valeurs sont largement dispersées autour de la moyenne.

### Exemple :
Si l’on prend les données suivantes : \( 2, 4, 6, 8, 10 \),
- La moyenne est \( \mu = 6 \).
- Les écarts par rapport à la moyenne sont \( -4, -2, 0, 2, 4 \).
- Les carrés de ces écarts sont \( 16, 4, 0, 4, 16 \).
- La variance est la moyenne de ces carrés, soit \( \frac{16 + 4 + 0 + 4 + 16}{5} = 8 \).

La variance peut aussi servir à mesurer la volatilité dans des contextes comme la finance (mesure des fluctuations de prix) ou l’évaluation des erreurs dans les prévisions.