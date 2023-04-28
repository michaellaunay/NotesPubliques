# La séquence de boot

La séquence de boot est une étape clé dans le processus de démarrage de l'ordinateur. Lorsque l'ordinateur est allumé, le bios (basic input output system) recherche le périphérique d'amorçage en suivant une séquence de boot préétablie.

Le bios lance ensuite le chargeur (loader) qui se trouve dans le MBR (Master Boot Record) et qui ne fait que 512 octets ou sur la "partition système EFI" (EFI System Partition - ESP) dans les systèmes utilisant l'UEFI (windows > 10).

Grub2 est utilisé par Ubuntu depuis la version 9.10 (2009), par Debian depuis la version 6.0 (2011).

Si le chargeur est GRUB 2, alors le MBR ne contient que la première partie de GRUB, appelée "stage 1", qui a pour rôle de charger "stage 1.5" et "stage 2".

"Stage 1.5" contient le code d'accès aux différents systèmes de fichiers. Il est contenu dans les premiers secteurs et peut avoir une taille de 33ko. Ensuite, "stage 2" est chargé, il peut être n'importe où sur le disque.

Une fois "stage 2" chargé, il propose à l'utilisateur de choisir le système d'exploitation ou la version du noyau à démarrer à travers un menu. L'utilisateur peut également modifier la configuration de démarrage en éditant le menu (en appuyant sur "e").

Ensuite, "stage 2" charge l'image du système d'exploitation choisi. Ces fichiers se trouvent dans "/boot". Le noyau en version minimaliste en version compressé est dans le fichier "vmlinuz-\*" (Virtual Memory Linux Compressed). Grub charge aussi en mémoire le fichier "initrd.img-\*" qui contient le système de fichiers tempraire pour permettre au noyau Linux de trouver les pilotes nécessaires (modules).

Le menu de démarrage passe quelques options à l'image du système, notamment le nom du périphérique contenant la racine, par exemple "root=/dev/sda1". Une fois le noyau lancé, il exécute "/sbin/init" avec 1 comme PID, à savoir que le PID 0 est celui de l'ordonnanceur (scheduler) qui permet de paralléliser et gérer les tâches.

Enfin, "systemd" démarre les démons (nom usuel les applications d'arrière plan) qui vont, par exemple, configurer les services réseau et l'environnement de l'utilisateur.

Systemd positionne l'heure du système et sa timezone, le nom de la machine (hostname), vérifie l'état des disques, monte les systèmes de fichiers, configure les interfaces réseau, le systeme de filtre réseau (packet filter), nettoye les vieux fichiers dans /tmp.

L'ensemble des services démarrés par systemd détermine le "run level".

0 - arrêt
1 -  simple utilisateur (single)
2 - multi-utilisateurs sans réseau
3 - multi-utilisateurs avec réseau
5 - mode graphique
6 - redémarrage

On peut savoir dans quel "run level" est le noyau avec la commande :
```bash
michaellaunay@leojag:~$ runlevel
N 5
```

On peut voir le script utilisé par Grub qui décompresse le noyau :
```bash
cat /usr/src/linux-headers-*-generic/scripts/extract-vmlinux
```

Lien :
https://youtu.be/H0T1yMpKiHY
https://fr.wikipedia.org/wiki/Ordonnancement_dans_les_syst%C3%A8mes_d%27exploitation

# GRUB 2

**GRUB** est l'acronyme de GRand Unified Bootloader.
C'est un programme GNU de démarrage permettant de choisir le système d'exploitation qui sera chargé.

Le fichier */boot/grub.cfg* crée le menu de démarrage au boot, et impose un choix par défaut.

La mise à jour du noyau entraine la modification du fichier */boot/grub/grub.cfg* qui ne doit pas être modifier manuellement.

## Modifications du démarrage du noyau
Pour customiser le bootloader, il faut éditer le fichier **/etc/default/grub** puis lancer **update-grub**.

## Modifications temporaires du démarrage du noyau
Lors du démarrage avec GRUB 2, il est possible de passer des paramètres en utilisant l'éditeur de ligne de commande de GRUB :

1.  Au démarrage de l'ordinateur, lorsque le menu GRUB s'affiche, sélectionner l'entrée de menu à démarrer, mais au lieu d'appuyer sur la touche Entrée pour démarrer, appuyer sur la touche "e" pour éditer les options de démarrage de cette entrée.
    
2.  L'éditeur de ligne de commande de GRUB apparaîtra, affichant les options de démarrage pour l'entrée sélectionnée. Il faut alors rechercher la ligne qui commence par "linux" ou "linuxefi" (pour les systèmes UEFI) et placer le curseur à la fin de cette ligne.
    
3.  À cet emplacement, il faut alors ajouter les paramètres de démarrage souhaités, comme le paramètre "nomodeset" pour résoudre des problèmes d'affichage qu'i faut ajouter à la fin de la ligne. Les paramètres doivent être séparer par des espaces.
    
4.  Une fois paramètres de démarrage ajoutés, appuyez sur Ctrl + X ou F10.

Ces modifications ne sont que temporaires et ne seront pas persistantes entre les redémarrages.

Pour connaitre les paramètres possibles :
```bash
man bootparam
```
Voir la doc : <https://doc.ubuntu-fr.org/grub-pc>

## Les arguments ou paramètres du kernel
Les principales options d'amorçage du noyau sont :
> -   root=/dev/sda1 indique la partition système,
> -   ro Indique que le système de fichier doit être monté en lecture seul,
> -   init permet de démarrer un autre processus à la place d'*init*,
> -   single ou emergency permet de passer en mode simple utilisateur et permet par exemple de pouvoir modifier le mot de passe root,
> -   quiet démarre en mode silencieux,
> -   nosmp n'utilise qu'un seul processeur,
> -   noht pas d'hyper threading,
> -   noapic pas de détection des interruptions,
> -   nolapic aucune gestion des interruptions,
> -   apm=off\|on désactive ou active la gestion de l'alimentation en énergie,
> -   noresume ne réveille pas une hibernation.

# Systemd vs InitV vs upstart

Une fois que Grub a terminé le stage 2, il passe la main au premier processus de Linux pour initialiser le noyau et dépendra de la distriubtion utilisée. Ce processus d'initialisation porte le numéro de processus (PID) 1 et est responsable de l'initialisation du système et du lancement de tous les autres processus.

Systemd, InitV et Upstart sont des systèmes d'initialisation utilisés dans les distributions Linux pour démarrer le système et gérer les processus.
Systemd est le plus utilisé.
Voici une explication de chacun d'entre eux :

## Systemd
Systemd est devenu le système d'initialisation par défaut dans de nombreuses distributions Linux, notamment Fedora à partir de la version 15, CentOS/RHEL à partir de la version 7, Ubuntu à partir de la version 15.04, et d'autres. Il offre une gestion avancée des services, une parallélisation du démarrage améliorée et une journalisation centralisée.

Systemd propose:
1.  Gestion des services : Systemd gère les services système en utilisant des "unités". Une unité est une description d'un service, d'un point de montage, d'un socket réseau, d'un appareil matériel, etc. Chaque service est représenté par une unité systemd, qui spécifie comment le service doit être géré, y compris le démarrage, l'arrêt, le redémarrage, etc.
    
2.  Parallélisation du démarrage : Systemd vise à accélérer le processus de démarrage en permettant le démarrage parallèle des services. Au lieu d'exécuter les services séquentiellement, systemd peut démarrer plusieurs services en parallèle, ce qui réduit le temps de démarrage global du système.
    
3.  Dépendances et ordre de démarrage : Systemd gère les dépendances entre les services de manière plus explicite. Chaque unité systemd peut spécifier les unités dont elle dépend et celles dont elle est dépendante. Cela permet à systemd de gérer l'ordre de démarrage des services en s'assurant que les dépendances sont satisfaites avant le démarrage des services.
    
4.  Journalisation avancée : Systemd intègre un journal de logs centralisé appelé le "journal systemd" (systemd journal). Le journal systemd collecte les logs système et fournit des fonctionnalités avancées de recherche et de filtrage des logs.
    
5.  Gestion des cgroups : Systemd utilise les cgroups (groupes de contrôle) du noyau Linux pour gérer les ressources système des processus. Il permet une gestion plus fine des ressources telles que la mémoire, le processeur, les entrées/sorties, etc., pour les services individuels.
    
6.  Commandes d'administration : Systemd fournit un ensemble de commandes d'administration pour interagir avec le système, telles que systemctl pour gérer les services et consulter l'état du système, journalctl pour accéder aux journaux système, et d'autres commandes pour gérer les unités systemd et les fonctionnalités spécifiques.

## InitV
InitV signifie Init system V qui a été introduit pour la première fois dans UNIX System V en 1983, d'où son nom et reste utilisé sur encore quelque distribution.

Ce système est remplacé par **upstart** sur Debain depuis la version 11.

Au démarrage, le noyau lance *init*.

L'ancien système était paramétrable via le fichier */etc/inittab* qui est remplacé par la notion de *job*.

**init** lit le répertoire */etc/event.d* qui contient les jobs à lancer.

Chaque job réalise des actions en fonction du niveau d'exécution du noyau.

**init** se charge de maintenir les jobs opérationnels.

La commande **initctl** permet de communiquer avec **init**.

La liste des jobs démarrés est donnés par **initctl list** : :

    root@luciole:~# initctl list
    control-alt-delete (stop) waiting
    last-good-boot (stop) waiting
    logd (stop) waiting
    rc-default (stop) waiting
    rc0 (stop) waiting
    rc1 (stop) waiting
    rc2 (stop) waiting
    rc3 (stop) waiting
    rc4 (stop) waiting
    rc5 (stop) waiting
    rc6 (stop) waiting
    rcS (stop) waiting
    rcS-sulogin (stop) waiting
    sulogin (stop) waiting
    tty1 (start) running, process 10354
    tty2 (start) running, process 8436
    tty3 (start) running, process 8437
    tty4 (start) running, process 8429
    tty5 (start) running, process 8430
    tty6 (start) running, process 8439

Le lancement d'un job se fait par *initctl start rc0*, son arrêt par *initctl stop rc0*.

Liens :

> -   <http://www.digitbooks.fr/articles/2-upstart.html>

# Les "target units" de SystemD
Sous Systemd, le concept de niveaux de démarrage traditionnel (runlevels) de SysVInit est remplacé par les cibles de démarrage (target units). Les cibles de démarrage sont des ensembles prédéfinis d'unités (services, points de montage, etc.) qui doivent être activées lors du démarrage du système.

Voici comment fonctionnent les niveaux de démarrage dans systemd :

1.  Cibles de démarrage : Au lieu d'avoir des niveaux de démarrage numérotés comme dans les systèmes d'initialisation anciens/traditionnels, systemd utilise des noms de cibles de démarrage. Par exemple, la cible de démarrage par défaut est appelée "default.target" (anciennement "runlevel5.target" dans SysVinit).
    
2.  Liaisons symboliques : Les cibles de démarrage sont associées à des liaisons symboliques. Par exemple, la liaison symbolique "default.target" pointe vers la cible réelle qui sera activée lors du démarrage. Les liaisons symboliques peuvent être modifiées pour changer la cible de démarrage par défaut.
    
3.  Activer et désactiver les cibles : Pour activer une cible de démarrage spécifique, vous pouvez utiliser la commande "systemctl set-default" suivie du nom de la cible. Par exemple, "systemctl set-default graphical.target" définirait la cible "graphical.target" comme cible de démarrage par défaut.
    
    Vous pouvez également activer ou désactiver une cible de démarrage spécifique pour une session de démarrage en utilisant la commande "systemctl isolate" suivie du nom de la cible. Par exemple, "systemctl isolate multi-user.target" active la cible "multi-user.target" pour la session de démarrage actuelle.
    
4.  Dépendances entre les cibles : Systemd gère les dépendances entre les cibles de démarrage à l'aide des directives "Requires" et "Wants". Les cibles peuvent déclarer leurs dépendances sur d'autres cibles, de sorte que les unités requises seront également activées lors du démarrage.
    
5.  Services liés aux cibles : Chaque cible de démarrage peut inclure des services qui doivent être activés lors du démarrage de cette cible. Par exemple, la cible "graphical.target" peut inclure les services responsables de l'environnement de bureau et de l'affichage graphique.
    

En résumé, systemd utilise les cibles de démarrage pour définir les ensembles prédéfinis d'unités à activer lors du démarrage du système. Les liaisons symboliques et les commandes "systemctl" permettent de gérer les cibles de démarrage par défaut et d'activer/désactiver des cibles spécifiques. Les dépendances entre les cibles et les services associés sont également gérés pour assurer l'ordre approprié du démarrage des unités.
# Les niveaux d'exécution de SysV (Run level)

Signification des niveaux pour Unix utilisant encore SysV : :

    Niveau S : Initialisation du système (le système de fichier est en read only),
    Niveau 0 : Extinction,
    Niveau 1 : Mode mono utilisateur,
    Niveau 2 et 5 : Mode multi-utilisateur avec réseau avec démarrage du serveur X,
    Niveau 6 : Reboot.

## Le système de démarrage des services

Lors du lancement d'*init* par le noyau, celui-ci transmet l'information de niveau à exécuter.

Ainsi *init* lance les jobs en leur précisant le niveau demandé.

Le job *rc5* lance */etc/init.d/rc 5*, qui à son tour va lancer les scripts contenus dans */etc/rc5.d* selon l'ordre lexicographique.

Les répertoires */etc/rcX.d* contiennent des liens vers les scripts de */etc/init.d*.

Les scripts contenus dans */etc/init.d* permettent de démarrer, arrêter, ou connaître le statut des démons.

## Changement du niveau d'exécution

Il est possible de changer le niveau d'exécution.

Exemple pour arrêter la machine :

> **initctl emit runlevel 0**