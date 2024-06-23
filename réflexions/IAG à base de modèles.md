L'idée est de faire une thèse pour un leader de l'IA :
- Le sujet
- La preuve de concept
- Le développement

# Sujet
Génération de modèles hybrides UML/BPMN pour représenter le monde et ses connaissances, puis exécution de simulations à partir de ces modèles, mesure de l'écart entre ces modèles et la réalité, génération de nouvelles versions du modèle, nouvelles simulations, nouvelles mesures d'écart, nouvelles versions, et incrémentation. Bref, roue de Deming sur les modèles.

# Techniques
L'ensemble des connaissances et modèles sont au format Markdown (Obsidian), Mermaid/PlantUML/S2, Excalidraw. Tout est en Python ou C++/Rust, les workflows sont implémentés via des plugins Obsidian. Une transformation de Wikipedia en Markdown permet de fournir une base de connaissances de base, elle est complétée par l'incorporation de livres transformés en Markdown. La langue utilisée pourrait être l'Esperanto.

L'apprentissage des LLM se fait par une boucle de rétroaction (la sortie est comparée avec un attendu et une correction des poids est propagée par rétropropagation du gradient).

Pour que les modèles puissent être utilisés dans une simulation, il faut pouvoir les exécuter. Or, généralement, on procède par transformation de modèles pour ensuite générer du code ou du texte (ou méta-modèle selon le niveau auquel on se situe). Dans le cas de la génération de code, on ne peut tester celui-ci que si la génération est à 100 %, ce qui n'est généralement ni le cas ni l'objectif puisque l'intervention humaine est nécessaire, sans quoi les modèles deviennent très détaillés ou alors les transformations portent les détails manquants aux modèles. Par définition, un modèle est une simplification de la réalité et ne peut pas être aussi précis que la réalité modélisée sans en être aussi complexe (cf. Théorème de l'incomplétude de Turing). La solution est d'accepter une imprécision dans les simulations (ce qui rejoint l'imperfection humaine). Il sera donc déterminant de mettre au point une méthode d’appréciation des écarts. Il faudra aussi trouver une méthode pour avoir des jeux de tests, voire les générer. Tout cela ressemble à ce qui a déjà été fait dans le cadre des LLM, mais avec une autre finalité. La sortie sera des modèles UML et BPMN au format texte, on pourra garder en partie l'architecture et les techniques des LLM pour la partie génération des modèles, mais l'entraînement se fera sur la capacité à transformer un texte en modèle. L'écart entre le modèle généré et la réalité sera donné par les résultats de la simulation des modèles ou de leur transformation.
