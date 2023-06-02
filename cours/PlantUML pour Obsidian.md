[[Obsidian]] possède un module permettant de visiualiser les diagrammes écrit avec la syntaxe de PlantUML.

Il existe de nombreux outil de texte vers diagrammes comme le montre [[Outils de modélisation textuels]].

L'utilisation de PlantUML pour modéliser des processus métiers en BPMN est une approche intéressante et assez répandue. PlantUML est un langage de modélisation graphique qui permet de créer des diagrammes à l'aide de texte structuré. Il est souvent utilisé pour générer des diagrammes UML, mais il peut également être utilisé pour représenter des diagrammes BPMN.

Le fait d'intégrer PlantUML dans Obsidian à l'aide du module dédié "PlantUML" nous permet de créer et de visualiser nos diagrammes BPMN directement dans notre environnement de prise de notes. Cela facilite la documentation et la communication des processus métiers au sein de notre organisation.

Il est important de noter que PlantUML n'est pas une solution spécifiquement conçue pour le BPMN. Bien qu'il permette de représenter des diagrammes BPMN, il manque certaines fonctionnalités avancées spécifiques au BPMN que l'on retrouve dans des outils dédiés tels que Bizagi, Signavio, ou Camunda.

Voici quelques retours connus de l'utilisation de PlantUML pour le BPMN :

1. **Simplicité et facilité d'utilisation** : PlantUML utilise une syntaxe simple et basée sur du texte, ce qui facilite la création des diagrammes BPMN. Si vous êtes à l'aise avec la syntaxe, vous pouvez rapidement modéliser vos processus métiers.

2. **Intégration avec Obsidian** : Si vous utilisez Obsidian comme votre principal outil de prise de notes, l'intégration de PlantUML peut vous permettre de centraliser vos diagrammes BPMN avec le reste de votre documentation.

3. **Limitations de fonctionnalités** : PlantUML peut ne pas offrir toutes les fonctionnalités avancées nécessaires pour modéliser certains aspects complexes du BPMN. Par exemple, des éléments spécifiques tels que les événements de message ou les règles de temporisation peuvent être plus difficiles à représenter.

4. **Rendu graphique** : Le rendu graphique des diagrammes BPMN dans PlantUML peut être plus rudimentaire par rapport aux outils dédiés. La qualité visuelle des diagrammes peut ne pas être aussi élevée que celle obtenue avec des outils spécialisés.

En résumé, l'utilisation de PlantUML pour modéliser des processus métiers en BPMN dans Obsidian peut être une solution pratique et efficace, en particulier si nous recherchons une approche basée sur du texte et une intégration avec notre environnement de prise de notes. Cependant, il est important de garder à l'esprit les limitations et les fonctionnalités spécifiques au BPMN qui pourraient être manquantes.

# Installation
Pour installer un serveur PlantUML sur Ubuntu 22.04, nous pouvons suivre les étapes ci-dessous :

## Mise à jour du système
Avant d'installer le serveur PlantUML, il est recommandé de mettre à jour notre système Ubuntu. Ouvrons un terminal et exécutons les commandes suivantes :

```
sudo apt update
sudo apt upgrade
```

## Installer Java

PlantUML nécessite Java pour s'exécuter. Nous pouvons installer OpenJDK en exécutant la commande suivante :

```
sudo apt install openjdk-11-jre
```

## Télécharger le serveur PlantUML

Pour télécharger le serveur PlantUML, Nous pouvons utiliser l'outil wget. Exécutons la commande suivante pour télécharger le fichier JAR du serveur :

```
wget https://sourceforge.net/projects/plantuml/files/plantuml.jar/download -O plantuml.jar
```

## Démarrer le serveur PlantUML

Maintenant que nous avons téléchargé le fichier JAR du serveur PlantUML, nous pouvons le démarrer en utilisant la commande suivante :

```
java -jar plantuml.jar -tsvg -p 9898
```

Le paramètre `-tsvg` spécifie que les diagrammes PlantUML doivent être générés au format SVG. Nous pouvons également utiliser d'autres formats d'image tels que PNG, JPG, etc.

L'option `-p` ou `--port` permet de spécifier le port sur lequel le serveur PlantUML doit écouter les requêtes. Nous utilisons le port 9898.

Le serveur PlantUML doit maintenant être en cours d'exécution sur notre machine Ubuntu. Nous pouvons y accéder en utilisant l'adresse http://localhost:9898 dans notre navigateur.

Remarque : Si nous souhaitons exécuter le serveur PlantUML en arrière-plan, nous pouvons utiliser l'outil `nohup` de la manière suivante :

```
nohup java -jar plantuml.jar -tsvg -p 9898 > plantuml.log &
```

Cela exécutera le serveur en arrière-plan et redirigera les journaux vers le fichier `plantuml.log`.

# Modeleurs graphiques générant du PlantUML
Il existe des modeleurs UML et BPMN qui peuvent générer la description des diagrammes en PlantUML. Ces outils nous permettent de créer des diagrammes UML et BPMN de manière visuelle et intuitive, puis de générer automatiquement le code PlantUML correspondant.

Voici quelques exemples d'outils qui offrent cette fonctionnalité :

1. **Visual Paradigm** : Visual Paradigm est un puissant outil de modélisation UML et BPMN qui prend en charge la génération de code PlantUML. Nous pouvons créer des diagrammes en utilisant l'interface graphique conviviale de Visual Paradigm, puis générer automatiquement le code PlantUML correspondant pour chaque diagramme.

2. **Astah** : Astah est un autre outil populaire pour la modélisation UML et la création de diagrammes BPMN. Il prend également en charge la génération de code PlantUML. Nous pouvons concevoir nos diagrammes dans Astah et exporter le code PlantUML pour les utiliser dans vos projets.

3. **Modelio** : Modelio est un environnement de modélisation open source qui prend en charge l'édition et la génération de code PlantUML pour les diagrammes UML. Il offre une variété de fonctionnalités de modélisation et permet la génération de code PlantUML pour les classes, les cas d'utilisation, les diagrammes de séquence, etc.

Ces outils nous permettent de travailler de manière visuelle et de bénéficier des avantages de la modélisation UML et BPMN, tout en générant automatiquement le code PlantUML correspondant. Cela peut être utile si vous préférez travailler dans un environnement graphique tout en utilisant PlantUML pour générer les diagrammes.