Dans le série des exercices pour mon cours de [[Python]] 
Je propose de faire un algorithme de compression qui se base sur la suite de décimale de PI.
Ainsi nous pourrions chercher les morceaux de données à compresser dans une table des décimales de PI.
La sortie aurait alors le format suivant :
Longueur total de la donnée décompressée
Nombre de tuples
Nombre de décimales en base 256 (octet) requis (le nombre de décimales de pi à calculer si on n'a pas une table).
Liste de tuples (index, longueur)

Pour décompresser générer les décimales de pi en base 256 (octet),
Puis pour chaque tuple copier la sous chaîne des décimales d'index et de longueur donnés par le tuple.

Pour les 10⁶ premières décimales, il existe 8 nombres de 5 digits n'ayant pas de représentation ['14523', '17125', '22801', '33394', '36173', '39648', '40527', '96710'].

Voir [Une formule ÉTRANGE | Axel Arno](https://youtu.be/MxcRztpXKFQ?si=rSVm8nF9l8mhjVQI) et [Une formule STUPÉFIANTE | Axel Arno](https://youtu.be/nZRKNth6OSA?si=Wo_n8xfmoHB0c3XL)
