---
schema_version: 1
uid: "01M02JG1VRNRTCN3RZ84CDDMPD"
titre: "MU 04 Cahier des charges"
type: document
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - genie-logiciel
  - cahier-des-charges
  - markdown
  - obsidian
  - meetup
resume: "Cahier des charges de l'étude de cas de publication des notes Markdown."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-03-23
date_modification: 2024-04-03
confidentialite: privee
publication: []
rag: true
metadata_verifiees: false
---
# Cahier des charges
Michaël Launay possède de nombreuses notes écrites en [[Markdown]] (le même format utilisé par ChatGPT pour ses réponses aux prompts que nous lui soumettons via le web) avec le logiciel de prise de notes [[Obsidian]].
Ces notes sont dans un répertoire sur son ordinateur appelé Notes.
Un sous-ensemble des notes de ce répertoire est archivé dans un dépôt de code source public et gratuit appelé GitHub (un outil proposé par Microsoft où l'on peut partager son code, mais aussi tout type de documents tant que l'ensemble fait moins de 2Go, cela avec l'outil [[git]]). Les notes de ce dépôt sont donc accessibles depuis le web et peuvent être consultées par tous, c'est d'ailleurs parmi elles, dans le répertoire "cours", que se trouvent tous les cours de Michaël Launay.
L'URL est https://github.com/michaellaunay/NotesPubliques.
Comme on peut le voir, ce dépôt n'est pas très "glamour", même si `GitHub` affiche les notes avec un minimum de mise en forme.
Nous souhaitons donc écrire un script Python qui permet de convertir ces notes en HTML et génère un site propre, "responsive".
Bien sûr, le script doit garder le mécanisme de liens propre à Obsidian, et gérer les différents types de ressources (image, PDF, etc.).
Nous souhaitons aussi avoir une charte graphique cohérente.