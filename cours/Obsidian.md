#Informatique #Cours
# Installation
## 1er façon

Téléchager Obsidian.

Utiliser snap sur le fichier téléchargé.

links :
https://help.obsidian.md/Getting+started/Download+and+install+Obsidian#Install+Obsidian+using+Snap

## 2ieme façon
Obsidian utilise le format flapack.
Il faut installer flatpack :
	sudo apt install flatpack

Puis mettre à jour les dépôts de flatpack :
	flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

Chercher l'id de l'application à installer :
	flatpack search Obsidian
	... md.obsidian.Obsidian ...

L'installer :
	flatpak install md.obsidian.Obsidian

Puis l'exécuter :
	flatpak run md.obsidian.Obsidian

links :
	https://doc.ubuntu-fr.org/flatpak
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
Alias : 
---
aliases : [Utilisation Obsidian, raccourcis Obsidian]
---

Case à cocher :
- [x] T1
- [ ] T2

## Templates
Créer un dossier par exemple "templates"
Aller dans "Paramètres", puis "Modules principaux", activer "Modèles", paramètrer le module Modèles depuis "Modules principaux -> Modèles", préciser le nom du dossier créer

links:
Vidéo Thibaut Lopes https://youtu.be/nREnpgixe9U
Vidéo Thibaut Lopes https://youtu.be/GZGvi3ez-HM
Vidéo Bibliothèque UdeM (Omntréal) https://youtu.be/VFpFbD4BXtc
Vidéo LeMindMappeur https://www.youtube.com/watch?v=LPxyEJeVsPg