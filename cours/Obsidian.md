#Informatique #Cours
# Introduction

Obsidian est une application de prise de notes et de gestion des connaissances qui favorise la création de liens entre les idées et les informations. Contrairement à d'autres applications de prise de notes, Obsidian stocke toutes nos notes sur notre ordinateur local sous la forme de fichiers Markdown, ce qui offre une grande flexibilité et un contrôle total sur nos données.

# Historique et philosophie d'Obsidian

Obsidian a été créé par les développeurs de Dynalist, une application de liste de tâches populaire. Ils ont développé Obsidian avec l'idée de créer un "deuxième cerveau" numérique, où l'on peut stocker toutes ses idées et connaissances de manière interconnectée. Ils croient fermement à l'importance de la possession des données par l'utilisateur et c'est pourquoi Obsidian stocke toutes les notes localement dans le Markdown en texte brut.

# Cas d'utilisation d'Obsidian

Obsidian peut être utilisé pour une multitude de cas d'utilisation :
- Notes de cours;
- Planification de projets;
- Suivi des tâches;
- Rédaction de documents;
- Création d'un système de gestion des connaissances personnelles;
Obsidian est un outil polyvalent qui peut s'adapter à de nombreux besoins.

# L'interface utilisateur d'Obsidian

L'interface d'Obsidian est épurée et centrée sur le contenu. Elle comporte un menu de navigation latéral, une zone de rédaction et une zone de visualisation. Elle offre également une visualisation graphique qui permet de voir les connexions entre nos notes. Obsidian offre aussi une grande capacité de personnalisation, nous permettant d'adapter l'interface à nos besoins et à notre style de travail.

# Installation et configuration d'Obsidian

Pour commencer avec Obsidian, nous téléchargeons d'abord l'application à partir de son site web officiel. Une fois le fichier téléchargé, nous l'ouvrons et suivons les instructions pour l'installer sur notre ordinateur. Une fois l'installation terminée, nous lançons Obsidian et arrivons à l'écran d'accueil. Ici, nous avons la possibilité de configurer Obsidian en fonction de nos besoins. Nous pouvons choisir le thème, l'organisation des panneaux, et beaucoup d'autres paramètres à partir du menu des préférences.

# Installation d'Obsidian sur GNU/Linux

Il existe plusieurs façons d'installer Obsidian sur notre ordinateur. Voici deux méthodes couramment utilisées pour l'installation sur un système Linux.

## 1ère façon: Installation via Snap

La première méthode consiste à télécharger Obsidian à partir de son site web officiel, puis à utiliser Snap pour installer le fichier téléchargé.

1. Téléchargons Obsidian à partir de [ce lien](https://help.obsidian.md/Getting+started/Download+and+install+Obsidian#Install+Obsidian+using+Snap).
2. Ouvrons un terminal et naviguons vers l'emplacement du fichier téléchargé.
3. Exécutons la commande `snap install <nom_du_fichier>` pour installer Obsidian. Il faut remplacer `<nom_du_fichier>` par le nom du fichier téléchargé.

Plus d'informations sur l'utilisation de Snap pour installer Obsidian peuvent être trouvées [ici](https://help.obsidian.md/Getting+started/Download+and+install+Obsidian#Install+Obsidian+using+Snap).

## 2ème façon: Installation via Flatpak

La seconde méthode est d'utiliser Flatpak, un système de gestion de paquets pour Linux. Cette méthode nécessite que nous ayons Flatpak installé sur notre système.

Installons Flatpak en exécutant la commande suivante dans un terminal :
```shell
sudo apt install flatpak
```

Mettons à jour les dépôts de Flatpak en exécutant la commande suivante :
```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Cherchons l'ID de l'application Obsidian en exécutant :
```bash
flatpak search Obsidian
```
L'ID devrait être quelque chose comme `md.obsidian.Obsidian`.

Installons Obsidian en exécutant :
```bash
flatpak install md.obsidian.Obsidian
```

Exécutons Obsidian en exécutant 
```bash
flatpak run md.obsidian.Obsidian
```

Pour mettre à jour
```bash
flatpack update
```

Pour désinstaller
```bash
flatpack uninstall md.obsidian.Obsidian
```
Nous pouvons trouver plus d'informations sur l'utilisation de Flatpak pour installer des applications sur [doc.ubuntu-fr.org](https://doc.ubuntu-fr.org/flatpak).

## 3ème façon: Utiliser un logiciel compatible

Sous [[Visual studio code]], il est possible d'installer une extension Open Source qui permet de lire et écrire les notes Obsidian et suivant les liens. Cette extension s'appelle [Mardown Memo](https://marketplace.visualstudio.com/items?itemName=svsool.markdown-memo)

# Création de notre premier vault

Dans Obsidian, les notes sont stockées dans ce qu'on appelle un "vault" (ou "coffre-fort" en français). Pour créer notre premier vault, nous sélectionnons "Nouveau vault" depuis l'écran d'accueil, puis nous lui donnons un nom et choisissons un emplacement sur notre ordinateur pour le stocker. Une fois le vault créé, nous pouvons commencer à y ajouter des notes. Il est possible de réaliser soi-même la fonctionnalité de partage, en gérant soit même un dépôt git.

# Navigation entre les notes

Dans Obsidian, nous avons plusieurs façons de naviguer entre nos notes. Nous pouvons utiliser la barre latérale pour parcourir les notes de notre vault. Nous pouvons également utiliser la barre de recherche en haut pour trouver rapidement une note. En outre, Obsidian offre la possibilité de voir un aperçu d'une note en passant simplement la souris sur un lien, ce qui peut grandement faciliter la navigation entre nos notes.

# Création de liens entre les notes

L'un des aspects les plus puissants d'Obsidian est sa capacité à créer des liens entre les notes. Pour créer un lien vers une autre note, nous écrivons le nom de la note entre crochets doubles, comme ceci: `[[Nom de la note]]`. Une fois que nous avons créé un lien, nous pouvons cliquer dessus pour accéder à la note liée. De plus, Obsidian garde une trace de tous les liens entrants et sortants d'une note, ce qui nous aide à comprendre comment nos idées et nos informations sont interconnectées.

# Découverte du Markdown dans Obsidian

Le langage Markdown est un langage de balisage léger conçu pour être facile à lire et à écrire. Dans Obsidian, toutes nos notes sont écrites en Markdown. Pour créer un titre, par exemple, nous utilisons le symbole '#'. Par exemple, '# Titre' créera un titre de niveau 1, '## Sous-titre' un titre de niveau 2, et ainsi de suite. Pour mettre du texte en gras, nous entourons les mots avec deux astérisques, comme ceci: '**texte en gras**'. Pour créer une liste à puces, nous commençons chaque ligne par un tiret '-'. 

Markdown supporte également les liens hypertextes et les images. Pour créer un lien, nous utilisons la syntaxe suivante: `[texte du lien](URL)`. Pour ajouter une image, nous utilisons une syntaxe similaire: `![texte alternatif](URL de l'image)`.

Pour plus d'information voir [[Markdown]]

# Familiarisation avec les outils de rédaction d'Obsidian

Obsidian fournit une gamme d'outils de rédaction qui rendent la création de notes très fluide. Par exemple, Obsidian propose un mode de prévisualisation qui nous permet de voir comment notre note apparaîtra une fois rendue. Il existe également un mode éditeur qui nous permet de voir et de modifier directement le code Markdown de notre note.

Obsidian offre également une variété d'outils pour formater notre texte. Nous pouvons utiliser les boutons de la barre d'outils pour ajouter des titres, des listes, des citations, et plus encore. Nous pouvons même utiliser des raccourcis clavier pour accélérer le processus de rédaction.

## Organisation des notes avec les tags et les liens

Obsidian offre plusieurs façons d'organiser nos notes. Une façon est d'utiliser des tags. Pour ajouter un tag à une note, nous utilisons le symbole '#'. Par exemple, `#tag` ajoute le tag 'tag' à notre note. Attention! il ne doit pas y avoir d'espace entre le dièse et nom du tag, sinon c'est un titre et non un tag!
Nous pouvons ensuite cliquer sur un tag pour voir toutes les notes qui ont ce tag.

Une autre façon d'organiser nos notes est d'utiliser des liens. Comme mentionné précédemment, nous pouvons créer des liens entre nos notes en utilisant la syntaxe `[[Nom de la note]]`. Obsidian trace ensuite ces liens, ce qui nous permet de voir comment nos notes sont connectées les unes aux autres.

## panneau de recherche

Pour retrouver rapidement une note dans Obsidian, nous pouvons utiliser le panneau de recherche. Pour l'ouvrir, nous appuyons sur Ctrl+P (Cmd+P sur Mac), puis nous tapons les mots clés que nous recherchons. Obsidian affiche alors une liste de notes qui correspondent à notre recherche.

Le panneau de recherche d'Obsidian est très puissant. Il peut rechercher non seulement les titres de notes, mais aussi leur contenu. De plus, il prend en charge la recherche par tags, ce qui signifie que nous pouvons rechercher toutes les notes avec un certain tag en tapant '#tag' dans le panneau de recherche.

IV. Les fonctionnalités avancées d'Obsidian

## visualisation graphique

1. L'un des aspects les plus uniques d'Obsidian est son mode de visualisation graphique. Ce mode nous permet de voir une représentation visuelle de toutes nos notes et des liens entre elles. Chaque note est représentée par un point, et chaque lien par une ligne reliant deux points. Cette fonctionnalité peut nous aider à comprendre comment nos idées sont connectées et à découvrir de nouvelles connexions.

2. Pour accéder au mode de visualisation graphique, nous cliquons sur l'icône en forme de graphe dans la barre latérale. Nous pouvons ensuite zoomer et dézoomer, et cliquer sur les points pour naviguer entre nos notes.

# Étendre Obsidian

Obsidian prend en charge les plugins, ce qui nous permet d'étendre ses fonctionnalités en fonction de nos besoins. Il existe de nombreux plugins disponibles, qui ajoutent des fonctionnalités allant des graphiques à la gestion des tâches.

Pour installer un plugin, nous allons dans le menu des paramètres, dans le sous menu "Modules Complémentaires", puis dans la section des "Modules Complémnetaires", bouton "Parcourir". Nous pouvons alors chercher des plugins et les installer directement à partir de là.

# Options de synchronisation et de sauvegarde

Même si Obsidian stocke toutes nos notes localement, il offre plusieurs options pour synchroniser et sauvegarder nos notes. Nous pouvons, par exemple, stocker notre vault dans un dossier synchronisé avec un service cloud comme Dropbox ou Google Drive.

Obsidian offre également un service de synchronisation intégré, Obsidian Sync. Ce service est payant, mais il offre plusieurs avantages, comme la synchronisation en temps réel, la prise en charge de plusieurs appareils, et la sauvegarde des versions précédentes de nos notes.

# Découverte des thèmes et de la personnalisation d'Obsidian

Obsidian est personnalisable. Nous pouvons, par exemple, choisir entre un thème clair et un thème sombre, et nous pouvons personnaliser la disposition de notre interface utilisateur en déplaçant et en redimensionnant les panneaux.

Obsidian prend également en charge les thèmes personnalisés, que nous pouvons télécharger et installer à partir de la communauté Obsidian. Ces thèmes peuvent changer l'apparence de notre interface utilisateur, de la couleur de fond aux icônes en passant par la typographie.

Pour installer un thème personnalisé, nous allons dans le menu des "Paramètres", puis dans la section "Apparence". Nous pouvons alors chercher des thèmes et les installer directement à partir de là.

# Utiliser sa feuille de style CSS
Obsidian, en plus de sa prise en charge du Markdown, permet d'injecter des balises HTML et du CSS personnalisé dans vos notes, ce qui permet de styliser vos notes de manière beaucoup plus sophistiquée.

## Personnalisation avec votre propre feuille de style CSS

1. Tout d'abord, il faut activer le support du CSS personnalisé dans Obsidian. Pour cela, allez dans les paramètres (`Settings`), puis dans la section `Appearance` et activez l'option `Custom CSS`.

2. Ensuite, vous devrez créer une feuille de style CSS personnalisée. Dans votre coffre-fort Obsidian, créez un nouveau fichier appelé `obsidian.css`. C'est dans ce fichier que vous allez écrire vos styles CSS personnalisés.

3. Vous pouvez maintenant commencer à écrire vos propres règles CSS dans ce fichier. Par exemple, si vous voulez que tous les titres de niveau 1 soient colorés en rouge, vous pouvez ajouter la règle suivante:

    ```css
    h1 {
        color: red;
    }
    ```

4. Les changements que vous apportez à votre fichier `obsidian.css` seront automatiquement appliqués à toutes vos notes.

## Utilisation de balises HTML

Obsidian supporte également l'utilisation de balises HTML dans nos notes. Cela signifie que nous pouvons utiliser des éléments comme `<span>` pour appliquer des styles spécifiques à des parties spécifiques de notre texte. Par exemple, si nous avons défini une classe CSS appelée `Classe1` dans notre fichier `obsidian.css` qui change la couleur du texte en bleu, nous pouvons utiliser cette classe dans une balise `<span>` pour faire apparaître du texte en bleu:

```html
<span class="Classe1">Un texte affiché avec le style de la classe Classe1</span>
```

Cela afficherait le texte "Un texte affiché avec le style de la classe Classe1" en bleu, en supposant que notre classe `Classe1` ressemble à ceci dans notre fichier `obsidian.css`:

```css
.Classe1 {
    color: blue;
}
```

Notons que l'utilisation de HTML et CSS dans Obsidian peut être très puissante, mais elle nécessite également une bonne compréhension de ces langages. Si vous n'êtes pas déjà familiarisé avec eux, il existe de nombreuses ressources en ligne pour vous aider à apprendre, tel [MDN](https://developer.mozilla.org/fr/docs/Web/CSS).

V. Projets pratiques avec Obsidian

# Exemples d'usages
## Développement d'un système de gestion de connaissances personnelles

Avec Obsidian, nous pouvons créer un système de gestion de connaissances personnelles (Personal Knowledge Management, PKM) puissant et flexible. Nous commençons par créer des notes pour chaque idée ou information que nous voulons retenir. Nous pouvons alors lier ces notes entre elles, créant ainsi un réseau interconnecté de connaissances.

Au fur et à mesure que notre réseau de notes grandit, nous commençons à voir des motifs et des connexions qui n'étaient pas évidents auparavant. Cela peut nous aider à développer une compréhension plus profonde et plus nuancée de notre domaine de connaissance.

## Planification et réalisation d'un projet de recherche avec Obsidian

Obsidian peut également être un outil précieux pour la planification et la réalisation de projets de recherche. Nous pouvons créer des notes pour chaque article que nous lisons, chaque idée que nous avons, et chaque hypothèse que nous voulons tester.

Nous pouvons ensuite utiliser les fonctionnalités de liaison et de visualisation d'Obsidian pour comprendre comment ces éléments sont connectés et pour définir la direction de notre recherche. Les plugins d'Obsidian, comme le plugin Kanban, peuvent également nous aider à gérer nos tâches et notre temps efficacement.

## Création d'un journal numérique quotidien

Enfin, Obsidian peut être utilisé pour créer un journal numérique quotidien. Nous pouvons créer une nouvelle note pour chaque jour, et y écrire nos pensées, nos idées, nos expériences et nos réflexions. 

Avec le temps, notre journal devient une précieuse ressource pour l'introspection et la réflexion personnelle. En plus, avec la fonction de recherche d'Obsidian, nous pouvons facilement retrouver et revisiter des entrées passées.

# Autres outils similaires à Obsidian

Il existe de nombreux autres outils de prise de notes et de gestion de connaissances que nous pourrions explorer, comme Roam Research, Notion, et Zettlr. Chacun de ces outils a ses propres forces et faiblesses, et le meilleur outil pour nous dépendra de nos besoins et de notre style de travail.

# Applications d'Obsidian

Obsidian transforme la façon dont nous travaillons et apprenons. Par exemple, dans l'éducation, Obsidian aide les étudiants à développer une compréhension plus profonde et plus intégrée de leur domaine d'étude. Dans la recherche, Obsidian aide les chercheurs à gérer et à synthétiser une grande quantité d'information.

# Proposition de ressources pour
1.  Pour ceux qui souhaitent continuer à explorer Obsidian après ce cours, il existe de nombreuses ressources disponibles. La documentation officielle d'Obsidian est un excellent point de départ : elle couvre toutes les fonctionnalités de base et avancées d'Obsidian.
    
2.  La communauté Obsidian est également une ressource précieuse. Il existe des forums et des groupes de discussion où les utilisateurs d'Obsidian partagent des conseils, des astuces et des idées sur la façon d'utiliser Obsidian.
    
3.  Enfin, il y a de nombreux tutoriels, cours en ligne, et livres qui approfondissent l'utilisation d'Obsidian. Certains se concentrent sur des sujets spécifiques, comme la gestion de connaissances personnelles, tandis que d'autres offrent une vue d'ensemble plus large de toutes les fonctionnalités d'Obsidian.

# Commandes
"ctrl" ou "command" sur mac
ctrl+p : Accès aux commandes que l'on trouvera par autocomplétion de la saisie.
ctrl+o : Ouvrir un fichier
ctrll+n : Créer une note
ctrl+f : Recherche d'une chaîne de caractères
ctrl+maj+f : Ouvre la barre de recherche sur tout Obsidian

## Synatxe
C'est celle de markdown [[Markdown]]
Lien : doubles crochets
Tag : dièse collé au tag

Case à cocher :
- [x] T1
- [ ] T2

## Templates
Créons un dossier par exemple "templates"
Allons dans "Paramètres", puis "Modules principaux", activer "Modèles", paramètrer le module Modèles depuis "Modules principaux -> Modèles", préciser le nom du dossier créer

# Note quotidienne

## Intérêt de la démarche
Les notes quotidiennes, aussi appelées "journal quotidien" dans certains outils de prise de notes, sont un excellent moyen d'organiser nos pensées, nos idées et nos activités au jour le jour. Voici comment nous pouvons les utiliser efficacement.

1. **Nous démarrons chaque jour avec une nouvelle note :** En début de journée, nous ouvrons notre outil de prise de notes et créons une nouvelle note pour la journée. Nous la nommons généralement avec la date du jour pour la retrouver facilement plus tard.

2. **Nous consignons nos pensées et nos idées :** Tout au long de la journée, nous utilisons notre note quotidienne pour enregistrer nos pensées, nos idées, nos observations et nos intuitions. Il n'y a pas de format fixe à suivre - nous écrivons simplement ce qui nous vient à l'esprit.

3. **Nous suivons nos activités :** Nous utilisons également nos notes quotidiennes pour suivre ce que nous faisons chaque jour. Cela peut inclure les tâches que nous avons terminées, les réunions auxquelles nous avons assisté, les personnes que nous avons rencontrées, etc.

4. **Nous réfléchissons à notre journée :** À la fin de la journée, nous aimons prendre un moment pour réfléchir à notre journée. Nous relisons notre note quotidienne et réfléchissons à ce que nous avons appris, ce que nous avons accompli et ce que nous voulons faire différemment demain.

5. **Nous relions nos notes quotidiennes à d'autres notes :** Si nous trouvons que certaines de nos pensées ou idées se rapportent à d'autres notes que nous avons prises, nous créons un lien entre ces notes. Cela nous aide à voir les connexions entre nos idées et à construire un réseau de connaissances interconnectées.

6. **Nous revoyons nos notes quotidiennes :** De temps en temps, nous aimons parcourir nos anciennes notes quotidiennes. Cela nous donne une perspective sur notre développement et notre croissance au fil du temps, et nous aide souvent à trouver des idées ou des intuitions que nous avions oubliées.

En utilisant les notes quotidiennes de cette manière, nous constatons que nous sommes plus conscients de nos pensées et de nos activités, et que nous sommes mieux capables de réfléchir et d'apprendre de nos expériences.

## Mise en place dans Obsidian

Dans Obsidian, le concept de notes quotidiennes est intégré de manière assez profonde. Voici comment nous utilisons cette fonctionnalité :

1. **Nous activons les notes quotidiennes :** Pour commencer, nous nous assurons que la fonctionnalité des notes quotidiennes est activée. Pour ce faire, nous allons dans Paramètres > Plugins de base, puis nous activons le plugin "Notes quotidiennes".

2. **Nous configurons nos préférences :** Une fois le plugin activé, nous cliquons sur l'engrenage à côté de "Notes quotidiennes" pour accéder à ses paramètres. Ici, nous pouvons configurer le format de la date, le répertoire dans lequel les notes quotidiennes seront enregistrées, et d'autres préférences.

3. **Nous créons notre note quotidienne :** Avec le plugin des notes quotidiennes activé, une nouvelle icône apparaît dans la barre latérale. En cliquant sur cette icône (qui ressemble à une feuille de calendrier), nous pouvons créer une nouvelle note pour la journée actuelle.

4. **Nous utilisons la note quotidienne :** Comme nous l'avons expliqué précédemment, nous utilisons cette note pour consigner nos pensées, nos idées, nos activités et nos réflexions tout au long de la journée. Nous pouvons également créer des liens vers d'autres notes en tapant [[ suivi du nom de la note à laquelle nous voulons nous référer.

5. **Nous naviguons entre les notes quotidiennes :** Avec le plugin des notes quotidiennes, il est facile de naviguer entre nos différentes notes quotidiennes. Dans le mode de visualisation, nous voyons des flèches en haut de la note qui nous permettent de passer à la note de la journée précédente ou de la journée suivante.

Avec ces étapes, nous avons une structure solide pour documenter et réfléchir à notre journée dans Obsidian. Cela nous permet de suivre notre progression, de retrouver facilement d'anciennes notes et de voir comment nos idées évoluent au fil du temps.

# Modules
Modélisation textuelle de diagrammes [[PlantUML pour Obsidian]]

# Alias

---
aliases : [Utilisation Obsidian, raccourcis Obsidian]
---

# Liens
Vidéo Thibaut Lopes https://youtu.be/nREnpgixe9U
Vidéo Thibaut Lopes https://youtu.be/GZGvi3ez-HM
Vidéo Bibliothèque UdeM (Omntréal) https://youtu.be/VFpFbD4BXtc
Vidéo LeMindMappeur https://www.youtube.com/watch?v=LPxyEJeVsPg
