---
schema_version: 1
uid: "01M02JG1WNP9VCE6511XMZFB5P"
titre: "Tutorial panoramique"
type: idee
statut: actif
para: ressource
domaines:
  - communication
themes:
  - video
  - panoramique
  - pedagogie
  - multimedia
resume: "Idée d'avril 2023 : détourner la vidéo panoramique à 360 degrés pour des tutoriels à embranchements — approfondir en se déplaçant, audio contextuel selon le point de vue, arbre décisionnel — avec la réponse de ChatGPT de l'époque ; réalisable aujourd'hui dans le navigateur avec WebXR et A-Frame."
maturite: a-experimenter
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-04-16
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Idée, pas procédure
> Note d'avril 2023 requalifiée de `procedure` en `idee` le 31 août 2026 (audit du dossier réflexions, point 10) : l'idée tient dans mes trois questions ci-dessous, le rappel et la réponse qui les entourent sont de ChatGPT et sont conservés tels quels. Ce que ChatGPT décrivait en généralités se fait aujourd'hui dans le navigateur : A-Frame, présenté dans [[Solutions de VR]], offre nativement la sphère vidéo à 360 degrés, le son positionnel et les zones cliquables — les trois briques des trois questions (https://aframe.io/docs/).

# Rappel sur le panoramique
Les vidéos que l'on peut explorer en modifiant son point de vue sont appelées des vidéos à 360 degrés ou vidéos panoramiques à 360 degrés. Ces vidéos sont tournées à l'aide de caméras spéciales équipées de plusieurs objectifs qui capturent l'environnement dans toutes les directions. Lorsque la vidéo est visualisée sur un écran d'ordinateur ou un appareil mobile, il est possible de faire pivoter la vue à l'aide d'un curseur ou d'un mouvement du doigt pour changer l'angle de vision. Cela permet à l'utilisateur de regarder la vidéo de tous les côtés et de se sentir immergé dans l'environnement capturé.

# Détournement à des fins pédagogiques
Si l'on regarde la vidéo de présentation de la bibliothèque des Médicis à Florence https://youtu.be/vCKsck7doXU (voir [[Taille des bibliothèques]]), on voit qu'elle est panoramique et que l'on peut déplacer son point de vue tout en continuant à écouter la guide. Si j'ai bien compris on génère ce genre de fichier via un appareil photo panoramique.
Mes questions sont donc les suivantes :
1) pourrait-on utiliser la panoramique pour faire des vidéos de tutoriels où en se déplaçant on pourrait ainsi approfondir certains points du tutorial ou revenir sur des points que l'on n'a pas compris ?
2) Est-il alors possible de changer l'audio selon le point de vue que l'on a pris ? Peut-on rendre contextuel ce que l'on voit ?
3) Et ainsi avoir un arbre décisionnel de ce que l'on voit selon certaines valeurs de variables ?
Bref je souhaiterais détourner la panoramique pour faire de la pédagogie.

*Réponse de ChatGPT (avril 2023) :*

Oui, il est tout à fait possible d'utiliser des vidéos panoramiques pour des tutoriels ou des cours en ligne interactifs. En utilisant un équipement de prise de vue panoramique, on peut capturer des vidéos qui permettent aux spectateurs de déplacer leur point de vue dans la vidéo, et donc d'explorer les différents aspects de la leçon.

En ce qui concerne l'audio, il est possible d'utiliser des techniques de spatialisation du son pour faire en sorte que l'audio s'ajuste en fonction de l'emplacement du spectateur dans la vidéo. Cela permettrait à l'utilisateur d'entendre les commentaires ou les explications de l'instructeur en fonction de l'endroit où il se trouve dans la vidéo.

Il est également possible de rendre le contenu contextuel en utilisant des techniques d'annotation vidéo pour ajouter des informations supplémentaires sur l'environnement de la vidéo. Les annotations vidéo sont des zones cliquables de la vidéo qui peuvent contenir des textes, des images ou des liens, permettant ainsi de fournir des informations supplémentaires sur ce qui est visible à l'écran.

Enfin, il est possible d'utiliser des techniques d'analyse vidéo pour créer un arbre décisionnel basé sur les choix de l'utilisateur dans la vidéo. Cela peut permettre à l'utilisateur de naviguer dans la vidéo en fonction de ses propres choix et de ses propres intérêts, créant ainsi une expérience d'apprentissage plus personnalisée et interactive.

Tout cela nécessite un certain niveau de compétence technique pour la production, mais c'est tout à fait possible et de plus en plus courant pour des applications pédagogiques ou de formation en ligne.