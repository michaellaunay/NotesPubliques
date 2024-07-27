Ici, je vais développer l'idée de faire générer des modèles UML et BPMN par un modèle de langage (LLM) à partir des prompts saisis, afin de les rendre actionnables par transformation successive en code Python. L'objectif est d'obtenir un code qui reflète le texte saisi, permettant ainsi d'exécuter ce code sur des données d'entrée pour produire des données de sortie qui seront comparées à des résultats attendus.

- **Le sujet** : Automatisation de la génération de modèles UML et BPMN via un modèle de langage, leur transformation en code Python exécutable, et la validation de ce code par comparaison des résultats obtenus aux résultats attendus.

- **La preuve de concept** : Création d'un prototype démontrant la capacité du LLM à générer des modèles UML et BPMN corrects à partir de descriptions textuelles, et la transformation de ces modèles en code Python fonctionnel.

- **Le développement** : Mise en œuvre complète du processus, incluant l'amélioration de la précision des modèles générés, l'optimisation des transformations en code Python, et l'établissement d'un cadre de validation robuste pour comparer les résultats produits par le code aux attentes.

# Sujet
Génération de modèles hybrides UML/BPMN pour représenter le monde et ses connaissances, suivie de l'exécution de simulations basées sur ces modèles. Ce processus inclut la mesure de l'écart entre les modèles et la réalité, la génération de nouvelles versions des modèles, de nouvelles simulations, de nouvelles mesures d'écart et de nouvelles versions, en un cycle continu d'amélioration. En somme, il s'agit d'appliquer la roue de Deming aux modèles.

# Comment exécuter un modèle ?

Un modèle est une structure écrite dans un langage respectant une syntaxe et une grammaire précises, et qui véhicule une sémantique. Son objectif est de décrire un aspect de la réalité en la simplifiant, ce qui permet de la manipuler pour produire du sens, formuler des hypothèses et comparer des patterns de modèles. 

## Les défis de l'exécution de modèles

L'exécution d'un modèle présente plusieurs défis, principalement en raison des nombreux implicites :

- **Données d'entrée** : Les informations initiales nécessaires pour alimenter le modèle.
- **Données de référence** : Les informations standards utilisées pour valider les résultats du modèle.
- **Implicites du modèle** : Les éléments non explicitement décrits dans le modèle.
- **Culture et interprétation** : La culture et les connaissances du lecteur influencent l'interprétation et la complétion du modèle, ce qui peut générer des différences significatives par rapport à l'intention initiale.

## Transformation et exécution

Un modèle en soi n'est pas exécutable. Ce sont les transformations successives du modèle qui aboutissent à un squelette de code. Ce code doit ensuite être complété par les développeurs, qui ajoutent leur propre culture et interprétation. Cette phase de développement peut introduire des écarts entre le code final et l'interprétation initiale des experts du domaine modélisé.

### Conclusion

Pour minimiser les écarts et améliorer la fidélité de l'exécution du modèle, il est crucial d'avoir des processus de transformation et de validation rigoureux, ainsi qu'une communication claire entre les experts du domaine et les développeurs.

# Techniques

L'ensemble des connaissances et modèles est au format Markdown (Obsidian), Mermaid/PlantUML/S2, et Excalidraw. Le code est écrit en Python ou C++/Rust, et les workflows sont implémentés via des plugins Obsidian. Une transformation de Wikipedia en Markdown fournit une base de connaissances de base, complétée par l'incorporation de livres transformés en Markdown. La langue utilisée pourrait être l'Esperanto.

L'apprentissage des modèles de langage (LLM) se fait par une boucle de rétroaction, où la sortie est comparée à un résultat attendu et une correction des poids est propagée par rétropropagation du gradient.

Pour que les modèles puissent être utilisés dans une simulation, il faut pouvoir les exécuter. Or, généralement, on procède par transformation de modèles pour ensuite générer du code ou du texte (ou méta-modèle selon le niveau auquel on se situe). Dans le cas de la génération de code, on ne peut tester celui-ci que si la génération est à 100 %, ce qui n'est généralement ni le cas ni l'objectif puisque l'intervention humaine est nécessaire. Sans cette intervention, les modèles deviennent très détaillés ou alors les transformations portent les détails manquants aux modèles. Par définition, un modèle est une simplification de la réalité et ne peut pas être aussi précis que la réalité modélisée sans en être aussi complexe (cf. Théorème de l'incomplétude de Turing). 

## Approche de la simulation

La solution est d'accepter une imprécision dans les simulations, ce qui rejoint l'imperfection humaine. Il sera donc déterminant de mettre au point une méthode d'appréciation des écarts. Il faudra aussi trouver une méthode pour obtenir des jeux de tests, voire les générer. Tout cela ressemble à ce qui a déjà été fait dans le cadre des LLM, mais avec une autre finalité. 

## Génération et évaluation des modèles

La sortie sera des modèles UML et BPMN au format texte. On pourra garder en partie l'architecture et les techniques des LLM pour la génération des modèles, mais l'entraînement se fera sur la capacité à transformer un texte en modèle. L'écart entre le modèle généré et la réalité sera évalué par les résultats de la simulation des modèles ou de leur transformation. 

## Objectifs spécifiques

- **Modèles Markdown** : Utilisation de formats comme Obsidian pour structurer les connaissances.
- **Langages de description** : Utilisation de Mermaid, PlantUML et S2 pour la représentation graphique des modèles.
- **Outils graphiques** : Intégration d'Excalidraw pour des diagrammes plus interactifs.
- **Technologies de développement** : Mise en œuvre en Python, C++ et Rust.
- **Enrichissement des connaissances** : Transformation de contenus Wikipedia et de livres en Markdown pour une base de connaissances étendue.
- **Langue de modélisation** : Usage de l'Esperanto pour la standardisation linguistique.

## Méthode d'apprentissage et de validation

L'apprentissage des modèles de langage se fait via une boucle de rétroaction, où chaque sortie est comparée à un attendu. La rétropropagation du gradient permet de corriger les poids des modèles. Pour les simulations, l'exécution des modèles passe par des transformations successives jusqu'à obtenir un code opérationnel. L'acceptation de l'imprécision est cruciale, et une méthode rigoureuse d'appréciation des écarts doit être développée. 

L'entraînement des LLM pour la génération de modèles UML et BPMN devra se concentrer sur la transformation textuelle en modèles structurés. Les écarts entre le modèle généré et la réalité seront mesurés par les résultats des simulations, permettant d'affiner continuellement la précision et l'utilité des modèles produits.
