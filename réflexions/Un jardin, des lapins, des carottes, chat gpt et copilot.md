---
schema_version: 1
uid: 01M02JG1WN1WG24C72V5NJMQJD
titre: Un jardin, des lapins, des carottes, chat gpt et copilot
type: reflexion
statut: a-relire
para: ressource
domaines:
- communication
- enseignement
themes:
- pedagogie
- intelligence-artificielle
- python
resume: Article LinkedIn de mars 2023 — huit heures de cours de Python à Polytech'Lille avec ChatGPT et Copilot, le projet lapins-carottes généré en vingt minutes et la question « Monsieur, à quoi allez-vous servir ? » ; post-scriptum 2026 sur les agents, Copilot et la suite en classe.
auteurs:
- Michaël Launay
langue: fr
date_creation: 2023-03-07
date_modification: 2026-08-31
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Témoignage daté
> Article LinkedIn du 7 mars 2023, conservé tel quel comme témoignage d'époque. Post-scriptum ajouté le 31 août 2026 (audit du dossier réflexions, point 5).

#Article #Linkedin  #Réflexion 
J'ai donné 8h de cours de python à des étudiants de Polytech'Lille, où je leur ai d’abord passé la vidéo de Monsieur Phi https://youtu.be/R2fjRbc9Sa0 ) pour leur permettre de bien situer l'outil, puis nous avons généré du code en utilisant à la fois Copilot (c'est gratuit pour les étudiants) et Chatgpt3.

La première demande que nous avons faite à chatgpt est de nous donner l'url d'inscription gratuite à github pour les étudiants, ce qui leur a permis d'installer le plugin dans [[Visual studio code|Visual Studio Code]] sur le poste de tous les étudiants.

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
https://fr.wikipedia.org/wiki/%C3%89quations_de_pr%C3%A9dation_de_Lotka-Volterra (les boucles de rétroaction entre lapins et carottes évoquées dans [[L'utopie de la Modélisation]])

"""

Ce qui a le plus bluffé les étudiants est qu'il n'aura fallu qu'une vingtaine de minutes en jouant entre Copilot et Chatgpt pour avoir un programme qui s'exécute et fait la plupart des choses demandées.

Bien sûr, le fait de savoir ce que l'on veut et de bien connaître Python permet d'éviter des bogues comme lorsque la notation des paramètres d'une fonction fait appel à un type défini en dessous (Il ne nous a pas proposé de faire une interface EtreVivant). Dans ce cas-là, il suffisait de poser la bonne question à Chatgpt pour savoir qu'il suffisait d'importer Type de typing.

L'une des questions de l'une de mes élèves a alors été, "Monsieur, à quoi allez-vous servir ?" et moi de répondre, "à vous guider et non plus à essayer de vous enfoncer le savoir dans le crâne".

Pour traduire tout le code en anglais cela n'a pris que deux minutes !

Pour la petite histoire, les lapins meurent tous tant que l'on n’introduit pas un renard.

On pourrait faire le même travail à partir de http://loic-steffan.fr/WordPress3/dynamique-homme-nature-handy-modelisation-des-inegalites-et-de-lexploitation-des-ressources-dans-leffondrement-ou-la-soutenabilite-des-societes/ 

# Post-scriptum (31 août 2026)

*Ajouté lors de l'audit du dossier réflexions.*

- **Les outils ont changé de nature.** Le Copilot de 2023 était une complétion de code ; en 2026 le même nom recouvre un mode agent qui édite, lance les commandes et les tests et itère, un agent cloud auquel on assigne une *issue* et qui ouvre une *pull request*, une CLI et une revue de code automatique — voir [[Copilot]]. Il reste gratuit pour les étudiants et les enseignants par GitHub Education. ChatGPT 3 a laissé place à des agents qui écrivent, exécutent et corrigent ([[Travailler avec Claude]]) : les vingt minutes de 2023 tiennent aujourd'hui dans un prompt vérifiable, avec ses critères d'acceptation et ses commandes de test.
- **Ce qui n'a pas changé.** Copilot suggère, il ne prouve rien ; un code généré doit être lu, compris, testé et relu sous l'angle sécurité ([[Copilot]]). Savoir ce que l'on veut et connaître Python — l'interface `EtreVivant` que l'outil n'a pas proposée, le `Type` de `typing` qu'il a fallu lui demander — reste la condition, comme en 2023.
- **La réponse à « Monsieur, à quoi allez-vous servir ? »** a pris forme dans le [[Projet Encadré]] : l'usage de l'IA y est fortement recommandé, à condition d'être entièrement documenté et traçable, et les conversations avec l'outil font l'objet de débats en classe. Guider, c'est désormais lire les prompts avec les étudiants. Le billet compagnon de la même période, sur les détecteurs de texte, est [[Suis-je une IA]].
