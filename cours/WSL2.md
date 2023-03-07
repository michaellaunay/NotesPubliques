#Cours #Informatique 
# Objectif
WSL2 est un outil fournit avec windows qui permet de faire fonctionner une mahine virtuelle GNU/Linux.
# Description
WSL2 (Windows Subsystem for Linux 2) est une fonctionnalité de Windows 11 qui permet aux utilisateurs d'exécuter des distributions Linux directement sur Windows, sans avoir besoin d'installer un hyperviseur ou une machine virtuelle séparée. Il offre une compatibilité presque totale avec les applications Linux, ce qui permet aux utilisateurs de travailler sur des projets et des applications Linux directement sur leur système Windows.

Pour installer WSL2 sur Windows 11, vous devez suivre ces étapes :

1.  Ouvrez les "Paramètres" de Windows 11.
2.  Cliquez sur "Applications".
3.  Cliquez sur "Programmes et fonctionnalités".
4.  Cliquez sur "Fonctionnalités facultatives".
5.  Cliquez sur "Ajouter une fonctionnalité".
6.  Recherchez "Windows Subsystem for Linux".
7.  Cochez la case à côté de "Windows Subsystem for Linux" et cliquez sur "Installer".
8.  Redémarrez votre ordinateur lorsque l'installation est terminée.

Après avoir installé WSL2, télécharger et installer une distribution Linux à partir du Microsoft Store.
Les distributions Linux populaires, telles que Ubuntu, Debian et Kali Linux, sont disponibles gratuitement sur le Microsoft Store.
Après avoir installé une nouvelle distribution Linux de votre choix, l'ouvrir à partir du menu Démarrer de Windows 11 et commencer à travailler dans l'environnement Linux en ligne de commande.
Il est possible de rajouter l'interface graphique en installant xorg, voir ci-après.
# Limitations de wsl2
WSL2 est disponible sur Windows 11 Familiale. Cependant, pour pouvoir utiliser WSL2 sur Windows 11 Familiale si sa version est suppérieur ou égale à 1903 (mise à jour de mai 2019). Toutes les fonctionnalités de WSL2 ne sont pas disponibles sur Windows 11 Familiale, comme le partage de fichiers avec Linux, qui nécessite une version professionnelle ou entreprise de Windows 11.
# Installation de l'interface graphique
Lorsque vous installez Ubuntu via WSL2, l'interface graphique n'est pas incluse par défaut. Cependant, il est possible d'installer une interface graphique dans Ubuntu WSL2. Voici les étapes à suivre :

- Ouvrez le terminal Ubuntu WSL2.
- Exécutez la commande suivante pour mettre à jour les packages Ubuntu :
``` bash
sudo apt update && sudo apt upgrade
```    
- Exécutez la commande suivante pour installer le serveur X11 :
```bash
sudo apt install xorg
```
- Exécutez la commande suivante pour installer un gestionnaire de fenêtres léger, tel que XFCE4 :
``` bash
sudo apt install xfce4
```   
- Exécutez la commande suivante pour installer un seveur VNC, comme TigerVNC :
```bash
sudo apt install tigervnc-common tigervnc-standalone-server
```
- Exécutez la commande suivante pour configurer un mot de passe VNC :
```bash
vncpasswd
```  
- Exécutez la commande suivante pour lancer le serveur VNC :
```bash
vncserver :1 -geometry 1024x768 -depth 24
```    
    Cette commande lance un serveur VNC sur le port 5901.
- Téléchargez un client VNC pour Windows, tel que RealVNC ou TightVNC.
- Lancez le client VNC et connectez-vous à l'adresse IP de votre machine Ubuntu WSL2, suivie du port 5901 (par exemple, 127.0.0.1:5901).
-  Entrez le mot de passe VNC que vous avez défini précédemment et appuyez sur "Connexion".
L'interface graphique de XFCE4 s'affiche dans la fenêtre du client VNC.
Les performances de l'interface graphique peuvent être affectées en raison de la virtualisation de WSL2 et de la méthode d'accès à l'interface graphique via le réseau VNC.