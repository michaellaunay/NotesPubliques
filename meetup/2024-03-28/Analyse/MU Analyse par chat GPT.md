# Remarque générale
L'analyse doit être menée conjointement par l'humain et l'IA, l'idéal étant de sélectionner une méthode pour ensuite la mettre en application avec ChatGPT. Il est crucial de vérifier les résultats générés et de corriger d'éventuelles hallucinations ou pertes de contexte, qui surviennent généralement tous les 7 ou 8 prompts.
Nous avons opté ici pour la méthode 2TUP (2 Track Unified Process), qui est une approche orientée objet. La phase d'analyse dans cette méthode repose sur la création de cas d'utilisation, ce qui, en résumé, consiste à illustrer le manuel d'utilisation en décrivant des scénarios d'usage.
Habituellement, dans cette méthode, une première partie concerne  **[l'élicitation](https://hal.inria.fr/inria-00098954/)** , qui a pour but de préciser le besoin. En effet, les clients expriment souvent initialement un désir plutôt qu'un besoin clairement défini (voir [Architecture des logiciels]).
La première étape consiste à identifier les acteurs, les actions, les ressources, les données, les concepts et les idées. Ensuite, on élabore un glossaire métier, suivi de scénarios d'usage que l'on regroupe dans des cas d'utilisation.
Nous examinerons comment intégrer l'IA dans ces tâches.
# Retranscription de l'analyse faite avec Chat GPT
## Génération du glossaire métier

**Moi**
Prend le role d'un développeur python sous ubuntu 24.02, nous allons développer une petite application qui satisfait le cahier des charges suivant : 
```md
# Cahier des charges Michaël Launay possède de nombreuses notes écrites en [[Markdown]] (le même format utilisé par ChatGPT pour ses réponses aux prompts que nous lui soumettons via le web) avec le logiciel de prise de notes [[Obsidian]]. Ces notes sont dans un répertoire sur son ordinateur appelé Notes. Un sous-ensemble des notes de ce répertoire est archivé dans un dépôt de code source public et gratuit appelé GitHub (un outil proposé par Microsoft où l'on peut partager son code, mais aussi tout type de documents tant que l'ensemble fait moins de 2Go, cela avec l'outil [[git]]). Les notes de ce dépôt sont donc accessibles depuis le web et peuvent être consultées par tous, c'est d'ailleurs parmi elles, dans le répertoire "cours", que se trouvent tous les cours de Michaël Launay. L'URL est https://github.com/michaellaunay/NotesPubliques. Comme on peut le voir, ce dépôt n'est pas très "glamour", même si `GitHub` affiche les notes avec un minimum de mise en forme. Nous souhaitons donc écrire un script Python qui permet de convertir ces notes en HTML et génère un site propre, "responsive". Bien sûr, le script doit garder le mécanisme de liens propre à Obsidian, et gérer les différents types de ressources (image, PDF, etc.). Nous souhaitons aussi avoir une charte graphique cohérente.
```
Pour cela nous allons d'abord analyser ce cahier des charges, en réalisant un glossaire métier

**Chat GPT**
Pour développer une application répondant à ce cahier des charges sous Ubuntu 24.02, nous devons d'abord comprendre et définir clairement les termes et concepts clés mentionnés. Voici un glossaire métier pour vous aider à démarrer :

### Glossaire métier

- **Markdown** : Langage de balisage léger utilisé pour formater des documents texte. Markdown permet d'écrire en utilisant un format de texte simple qui peut être converti en HTML et d'autres formats. Il est souvent utilisé pour rédiger des fichiers README, pour la documentation de logiciels, et pour les messages dans des forums en ligne.

- **Obsidian** : Application de prise de notes et de gestion de connaissances qui utilise le format Markdown pour les notes. Obsidian permet de créer des liens entre les notes, simulant ainsi une base de données de connaissances interconnectée.

- **GitHub** : Plateforme d'hébergement de code source qui utilise Git, un système de contrôle de version distribué. GitHub permet aux développeurs de stocker, gérer et suivre les modifications de leur code source. Il supporte également la documentation et d'autres types de fichiers, rendant possible le partage public ou privé de projets complets.

- **Git** : Système de contrôle de version open-source utilisé pour le suivi des modifications dans les fichiers de programmation. Git aide les équipes de développement à coordonner leur travail et à gérer les versions de leur code source.

- **HTML** : Langage de balisage utilisé pour créer des pages web. HTML définit la structure et le layout des pages web en utilisant une variété de balises et d'attributs.

- **Site Responsive** : Site web conçu pour offrir une bonne expérience de visualisation sur une large gamme d'appareils, y compris les ordinateurs de bureau, les tablettes et les smartphones. Cela implique l'utilisation de techniques de conception web flexibles pour s'assurer que le contenu s'adapte automatiquement à la taille de l'écran de l'appareil.

- **Liens propre à Obsidian** : Références ou liens entre notes au sein de l'application Obsidian qui permettent de naviguer facilement d'une note à une autre, formant un réseau de connaissances interconnecté.

- **Ressources (image, PDF, etc.)** : Fichiers autres que les fichiers texte Markdown qui peuvent être inclus dans les notes. Ces ressources peuvent être des images, des documents PDF, des fichiers audio ou vidéo, etc.

- **Charte graphique** : Ensemble de directives qui définissent l'aspect visuel et le style d'un site web ou d'une application. Cela comprend la palette de couleurs, les polices de caractères, les éléments de mise en page et d'autres éléments de design visuel.

Avec ce glossaire en main, nous avons une base solide pour commencer à concevoir et développer le script Python qui transformera les notes de Michaël Launay en un site web propre et responsive, tout en respectant les exigences spécifiques mentionnées dans le cahier des charges.

## Génaration de la liste des scénarios concrets

**Moi**
Donne moi la liste des scénarios concrets au sens méthode 2TUP (UML) que l'on pourrait écrire à partir de ce scénario.

**Chat GPT**
La méthode 2TUP (Two-Track Unified Process) met l'accent sur une approche itérative et incrémentale du développement logiciel, où l'analyse et la conception se déroulent en parallèle, permettant des ajustements en fonction des retours et des évolutions du projet. Les scénarios concrets, souvent formalisés via des diagrammes de cas d'utilisation en UML (Unified Modeling Language), représentent des interactions typiques entre les utilisateurs (acteurs) et le système pour réaliser un objectif spécifique. À partir du cahier des charges donné, voici plusieurs scénarios concrets que l'on pourrait envisager :

### 1. **Création d'une note Markdown dans Obsidian**
- **Acteur** : Michaël Launay
- **Scénario** : Michaël crée une nouvelle note en utilisant le format Markdown dans Obsidian. Il peut y inclure du texte formaté, des liens vers d'autres notes, et des références à des ressources externes (images, PDF, etc.).

### 2. **Conversion de notes Markdown en HTML**
- **Acteur** : Système de conversion (script Python)
- **Scénario** : Le script lit une note Markdown spécifiée par l'utilisateur et la convertit en un fichier HTML respectant une charte graphique prédéfinie, tout en préservant les liens internes et externes.

### 3. **Ajout d'une ressource à une note**
- **Acteur** : Michaël Launay
- **Scénario** : Michaël ajoute une ressource (par exemple, une image ou un document PDF) à une note. Le système doit garantir que cette ressource soit accessible dans la version HTML de la note.

### 4. **Génération du site web complet**
- **Acteur** : Système de conversion (script Python)
- **Scénario** : À partir de l'ensemble des notes Markdown disponibles, le script génère un site web complet, organisé selon la structure des notes et dossiers d'Obsidian, en produisant un résultat responsive et esthétiquement cohérent.

### 5. **Consultation d'une note convertie en HTML**
- **Acteur** : Utilisateur du site web (visiteur)
- **Scénario** : Un utilisateur navigue sur le site généré, consulte différentes notes converties en HTML, et suit les liens entre elles pour explorer le contenu lié.

### 6. **Mise à jour d'une note dans Obsidian et régénération du site web**
- **Acteur** : Michaël Launay
- **Scénario** : Michaël met à jour le contenu d'une note dans Obsidian. Le système détecte cette mise à jour et régénère automatiquement la version HTML de la note ainsi que toutes les parties affectées du site web.

### 7. **Gestion des erreurs de conversion**
- **Acteur** : Système de conversion (script Python)
- **Scénario** : Lors de la conversion d'une note, le système rencontre un formatage Markdown non supporté ou une référence de ressource invalide. Il doit gérer l'erreur en informant l'utilisateur et, si possible, en procédant à une conversion partielle.

Ces scénarios couvrent une gamme d'interactions typiques entre les utilisateurs et le système, reflétant les étapes essentielles pour transformer les notes Markdown de Michaël en un site web attractif et fonctionnel. Ils serviront de base pour la modélisation des cas d'utilisation en UML, guidant ainsi les phases de développement et de test du projet.