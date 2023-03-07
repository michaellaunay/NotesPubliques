#Article #Linkedin  #Réflexion 
J'ai donné 8h de cours de python à des étudiants de PolytechLille, où je leur ai d’abord passé la vidéo de Monsieur Phi https://youtu.be/R2fjRbc9Sa0 ) pour leur permettre de bien situer l'outil, puis nous avons généré du code en utilisant à la fois Copilot (c'est gratuit pour les étudiants) et Chatgpt3.

La première demande que nous avons faite à chatgpt est de nous donner l'url d'inscription gratuite à github pour les étudiants, ce qui leur a permis d'installer le plugin dans visualcode sur le poste de tous les étudiants.

Ensuite, je leur ai fait générer le code du projet du premier trimestre qu'ils venaient de me rendre et dont voici le sujet :

"""

Simuler une population de lapins et de carottes dans un jardin.

Les lapins peuvent vivre 4 ans max s'ils ont manqué un repas et 6 ans s'ils sont bien nourris, c'est-à-dire s'ils ont mangé chaque semaine.

Lorsqu'ils ont 1 an, ils peuvent se reproduire s'ils ont un partenaire et donner 2 portées de 6 lapereaux: une en avril et une en juillet

Les carottes sont semées en mars et il y en a 200 en juin.

Il faut simuler l'évolution des populations.

Un lapin mange une carotte par semaine.

Un lapin meurt s'il n'a pas mangé depuis plus de 2 semaines.

Au départ, il n'y a que deux lapins, un mâle et une femelle et 200 carottes

On va simuler l'évolution hebdomadaire sur 6 ans, puis tracer les populations avec matplotlib

Écrire une classe Lapin une classe Carotte et une classe Jardin.

Utiliser les listes, et les fonctions random de math.
https://fr.wikipedia.org/wiki/%C3%89quations_de_pr%C3%A9dation_de_Lotka-Volterra

"""

Ce qui a le plus bluffé les étudiants est qu'il n'aura fallu qu'une vingtaine de minutes en jouant entre Copilot et Chatgpt pour avoir un programme qui s'exécute et fait la plupart des choses demandées.

Bien sûr, le fait de savoir ce que l'on veut et de bien connaitre Python permet d'éviter des bogues comme lorsque la notation des paramètres d'une fonction fait appel à un type défini en dessous (Il ne nous a pas proposé de faire une interface EtreVivant). Dans ce cas là, il suffisait de poser la bonne question à Chatgpt pour savoir qu'il suffisait d'importer Type de typing.

L'une des questions de l'une de mes élèves a alors étée, "Monsieur, à quoi allez-vous servir ?" et moi de répondre, "à vous guider et non plus à essayer de vous enfoncer le savoir dans le crâne".

Pour traduire tout le code en anglais cela n'a pris que deux minutes !

Pour la petite histoire, les lapins meurent tous tant que l'on n’introduit pas un renard.