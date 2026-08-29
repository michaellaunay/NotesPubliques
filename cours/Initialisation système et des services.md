---
schema_version: 1
uid: "01M02EX5B7ES318ZHXECH7A9JH"
titre: "Initialisation système et des services"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - gnu-linux
  - demarrage
  - systemd
resume: "Cours complet sur le démarrage GNU/Linux et la gestion moderne des services : UEFI/BIOS, GRUB 2, noyau et initramfs, PID 1, unités systemd, dépendances, targets, journal, timers, activation, cgroups, durcissement et diagnostic."
niveau: intermediaire
prerequis:
  - "[[GNULinux]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-04-28
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---

# Initialisation système et gestion des services sous GNU/Linux

> [!note] État du cours
> Ce cours décrit principalement **systemd**, qui est le gestionnaire de système et de services par défaut de nombreuses distributions GNU/Linux modernes. Les exemples visent une distribution récente de type Debian/Ubuntu, Fedora ou Arch Linux. Les versions réellement livrées par une distribution peuvent être plus anciennes que la version amont.

> [!important] Version de référence
> Au 29 août 2026, **systemd 261.2** est la dernière version stable publiée en amont ; systemd 262 est encore en phase de release candidate. Il faut donc toujours vérifier la version présente sur la machine avant d'utiliser une directive récente :
>
> ```bash
> systemd --version
> ```

## Objectifs

À la fin de ce cours, on doit être capable de :

- expliquer toute la chaîne de démarrage d'une machine Linux ;
- distinguer BIOS, UEFI, chargeur d'amorçage, noyau, initramfs et PID 1 ;
- comprendre le rôle de GRUB 2 sans employer la terminologie obsolète de GRUB Legacy ;
- identifier le système d'initialisation réellement utilisé ;
- comprendre le modèle d'unités de systemd ;
- créer et maintenir une unité `.service` robuste ;
- raisonner correctement sur les dépendances et l'ordre de démarrage ;
- utiliser targets, timers, sockets, paths, mounts et automounts ;
- administrer les services système et utilisateur ;
- exploiter `journalctl` et les outils de diagnostic ;
- limiter CPU, mémoire et processus grâce aux cgroups ;
- durcir un service avec les mécanismes de sandboxing de systemd ;
- analyser un démarrage lent ou défaillant ;
- distinguer systemd des anciens SysVinit et Upstart.

## Sommaire

1. La chaîne de démarrage d'un système Linux
2. BIOS, UEFI et Secure Boot
3. GRUB 2
4. Le noyau et l'initramfs
5. PID 1 et les systèmes d'initialisation
6. Architecture de systemd
7. Les unités systemd
8. Où sont stockées les unités ?
9. Administrer les services avec `systemctl`
10. Écrire une unité `.service`
11. Types de services et supervision
12. Dépendances et ordre de démarrage
13. Targets et anciens runlevels
14. Journalisation avec `journalctl`
15. Timers systemd
16. Activation par socket et par chemin
17. Montages, automounts et générateurs
18. Services utilisateur
19. cgroups, slices et gestion des ressources
20. Identité, répertoires et credentials
21. Durcissement des services
22. Analyse des performances de démarrage
23. Diagnostic et récupération
24. Unités transitoires avec `systemd-run`
25. Concevoir un service de production
26. SysVinit, Upstart et autres init systems
27. Travaux pratiques
28. Checklist et commandes de référence

---

# 1. La chaîne de démarrage d'un système Linux

Le démarrage moderne d'un système GNU/Linux est une **chaîne de responsabilités**. Il ne faut pas confondre les différents composants :

```text
Mise sous tension
      │
      ▼
Firmware BIOS ou UEFI
      │
      ▼
Chargeur d'amorçage
GRUB / systemd-boot / autre
      │
      ▼
Noyau Linux + ligne de commande
      │
      ▼
initramfs / early userspace
      │
      ▼
Montage du vrai système racine
      │
      ▼
PID 1 : généralement systemd
      │
      ▼
Unités, services, montages, sockets…
      │
      ▼
Sessions utilisateurs / environnement graphique
```

Chaque étape prépare l'étape suivante.

## 1.1 Firmware

Le firmware initialise suffisamment le matériel pour pouvoir rechercher une cible d'amorçage.

Selon la machine, il s'agit principalement de :

- **BIOS** sur les anciennes machines PC ;
- **UEFI** sur la majorité des machines modernes.

Le firmware n'est pas le noyau Linux et ne lance généralement pas directement les services du système.

## 1.2 Chargeur d'amorçage

Le chargeur d'amorçage sélectionne et charge généralement :

- le noyau Linux ;
- l'initramfs ;
- la ligne de commande passée au noyau.

GRUB 2 est un chargeur fréquent, mais il n'est pas obligatoire.

## 1.3 Noyau Linux

Une fois exécuté, le noyau :

- initialise le processeur et la mémoire ;
- initialise ses sous-systèmes ;
- détecte les périphériques accessibles ;
- charge les pilotes déjà intégrés ;
- utilise l'initramfs si nécessaire ;
- monte ou fait monter le vrai système de fichiers racine ;
- lance le premier processus de l'espace utilisateur.

Ce premier processus reçoit le **PID 1**.

## 1.4 PID 0 et PID 1

Une simplification fréquente consiste à dire que « le PID 0 est l'ordonnanceur ». Ce n'est pas une bonne description.

Le PID 0 correspond au **swapper/idle task** créé par le noyau. L'ordonnanceur est un sous-système du noyau ; ce n'est pas simplement « le processus PID 0 ».

Le **PID 1**, en revanche, est bien le premier processus de l'espace utilisateur. Sur un système systemd :

```bash
ps -p 1 -o pid,comm,args
```

Exemple :

```text
PID COMMAND         COMMAND
  1 systemd         /sbin/init
```

Le chemin `/sbin/init` peut être un lien symbolique vers systemd.

---

# 2. BIOS, UEFI et Secure Boot

# 2.1 BIOS classique

Sur une machine BIOS classique, le firmware lit traditionnellement le premier secteur amorçable d'un disque.

Historiquement, le **MBR** fait 512 octets et contient :

- du code d'amorçage ;
- la table de partitions MBR ;
- une signature.

Cette contrainte de taille explique les architectures historiques en plusieurs étapes.

# 2.2 UEFI

UEFI ne fonctionne pas comme un BIOS qui exécute simplement le code contenu dans le MBR.

Une installation UEFI utilise généralement une **EFI System Partition (ESP)**, formatée en FAT.

On peut la repérer avec :

```bash
findmnt /boot/efi
lsblk -f
```

Les exécutables EFI se trouvent typiquement sous :

```text
/boot/efi/EFI/
```

Par exemple :

```text
EFI/ubuntu/shimx64.efi
EFI/ubuntu/grubx64.efi
```

Le nom exact dépend de la distribution et de l'architecture.

# 2.3 Entrées UEFI

Sous Linux, on peut inspecter les entrées de démarrage UEFI avec :

```bash
efibootmgr -v
```

La commande nécessite généralement les privilèges administrateur pour certaines opérations de modification.

# 2.4 Secure Boot

**UEFI Secure Boot** vérifie une chaîne de confiance cryptographique avant d'exécuter certains composants du démarrage.

Une chaîne fréquente sur Ubuntu est :

```text
UEFI Secure Boot
   ↓
shim signé
   ↓
GRUB signé
   ↓
noyau signé
   ↓
modules selon politique de signature
```

Secure Boot n'est pas un chiffrement du disque.

Il vise principalement à empêcher l'exécution de code d'amorçage non autorisé.

Pour vérifier son état sur certaines distributions :

```bash
mokutil --sb-state
```

---

# 3. GRUB 2

**GRUB** signifie **GRand Unified Bootloader**.

GRUB 2 sait notamment :

- présenter un menu de démarrage ;
- charger plusieurs noyaux ;
- transmettre une ligne de commande au noyau ;
- lire de nombreux systèmes de fichiers ;
- démarrer depuis BIOS ou UEFI ;
- charger dynamiquement des modules ;
- lire LVM et diverses configurations RAID selon les modules disponibles.

# 3.1 Ne pas confondre GRUB Legacy et GRUB 2

L'ancien cours décrivait GRUB 2 avec :

```text
stage 1 → stage 1.5 → stage 2
```

Cette terminologie est celle de **GRUB Legacy**.

Dans GRUB 2 :

- `boot.img` joue un rôle proche de l'ancien stage 1 en mode BIOS ;
- `core.img` contient le cœur nécessaire à l'amorçage ;
- les fonctionnalités supplémentaires sont chargées via des fichiers `*.mod` ;
- il n'existe pas un unique fichier « stage 2 » équivalent à celui de GRUB Legacy.

En UEFI, GRUB est exécuté sous forme d'application EFI.

# 3.2 Fichiers importants

Sur Debian/Ubuntu :

```text
/etc/default/grub
/etc/grub.d/
/boot/grub/grub.cfg
```

Le fichier :

```text
/boot/grub/grub.cfg
```

est **généré** et ne doit généralement pas être modifié directement.

Pour modifier durablement la configuration :

1. modifier `/etc/default/grub` ou les scripts appropriés de `/etc/grub.d/` ;
2. régénérer la configuration.

Sur Debian/Ubuntu :

```bash
sudo update-grub
```

Cette commande est un wrapper autour des outils GRUB appropriés.

# 3.3 Examiner la configuration

```bash
grep -v '^#' /etc/default/grub
```

Pour connaître les entrées générées :

```bash
grep '^menuentry ' /boot/grub/grub.cfg
```

Le format exact peut varier.

# 3.4 Modifier temporairement la ligne du noyau

Depuis le menu GRUB :

1. sélectionner l'entrée ;
2. appuyer sur `e` ;
3. repérer la ligne `linux` ;
4. modifier temporairement les paramètres ;
5. démarrer avec `Ctrl+X` ou `F10` selon l'interface.

La modification ne persiste pas après le redémarrage.

# 3.5 Paramètres du noyau utiles

Exemples :

```text
root=UUID=...
ro
rw
quiet
systemd.unit=rescue.target
systemd.unit=emergency.target
init=/bin/bash
```

> [!warning]
> `init=/bin/bash` contourne une grande partie de l'initialisation habituelle et doit être utilisé uniquement pour un dépannage maîtrisé, généralement avec accès physique et après compréhension de l'état des systèmes de fichiers.

Pour voir la ligne réellement reçue par le noyau :

```bash
cat /proc/cmdline
```

# 3.6 Identifier le noyau et l'initramfs

```bash
uname -r
ls -lh /boot/vmlinuz-* /boot/initrd.img-* 2>/dev/null
```

Sur d'autres distributions, les noms et emplacements peuvent varier.

---

# 4. Le noyau et l'initramfs

# 4.1 Pourquoi un initramfs ?

Le noyau ne peut pas toujours monter immédiatement le vrai système racine.

Il peut avoir besoin de :

- charger un module de stockage ;
- assembler un RAID ;
- activer LVM ;
- déverrouiller un volume LUKS ;
- découvrir le périphérique racine ;
- démarrer depuis le réseau.

L'**initramfs** fournit alors un petit espace utilisateur temporaire en RAM.

# 4.2 initrd et initramfs

Les termes sont parfois utilisés de manière interchangeable dans les noms de fichiers, mais techniquement :

- un ancien **initrd** était une image de disque RAM ;
- un **initramfs** est une archive décompressée dans un système de fichiers RAM initial du noyau.

Un fichier nommé `initrd.img-*` sur Debian/Ubuntu peut donc contenir un initramfs moderne.

# 4.3 Inspecter un initramfs

Sur Debian/Ubuntu :

```bash
lsinitramfs /boot/initrd.img-"$(uname -r)" | less
```

Pour le régénérer :

```bash
sudo update-initramfs -u
```

Sur Fedora/RHEL, l'outil courant est plutôt `dracut`.

# 4.4 Passage au vrai système racine

Après préparation du stockage, l'early userspace bascule vers le vrai `/` puis lance le vrai PID 1.

Le détail dépend de la distribution et de l'initramfs utilisé.

---

# 5. PID 1 et les systèmes d'initialisation

Le PID 1 possède un rôle particulier sous Linux.

Il doit notamment :

- initialiser l'espace utilisateur ;
- lancer ou orchestrer les services ;
- adopter les processus orphelins dans certains cas ;
- récolter les processus terminés qui lui sont rattachés ;
- gérer l'arrêt et le redémarrage du système.

# 5.1 Identifier PID 1

```bash
ps -p 1 -o comm=
```

ou :

```bash
readlink -f /sbin/init
```

# 5.2 Quelques init systems

On rencontre notamment :

- systemd ;
- SysVinit ;
- OpenRC ;
- runit ;
- s6 ;
- historiquement Upstart.

Ce cours se concentre sur systemd.

# 5.3 Correction historique : Debian et Upstart

Debian **n'a pas utilisé Upstart comme système d'initialisation par défaut**.

Le parcours principal de Debian a été :

```text
SysVinit → systemd
```

systemd est l'init par défaut de Debian depuis **Debian 8 Jessie**.

Upstart a été utilisé principalement par Ubuntu pendant plusieurs années avant le passage d'Ubuntu à systemd avec Ubuntu 15.04.

# 5.4 Compatibilité SysV et systemd moderne

Historiquement, systemd savait convertir automatiquement certains scripts SysV `/etc/init.d/*` en unités transitoires.

Cependant, **systemd 260 a supprimé en amont le support des scripts de service System V**.

Cela ne signifie pas que tous les systèmes Linux ont immédiatement perdu la compatibilité :

- les distributions peuvent livrer des versions antérieures ;
- elles peuvent conserver des mécanismes de compatibilité propres ;
- les logiciels bien maintenus doivent désormais fournir des unités systemd natives lorsque systemd est la cible.

---

# 6. Architecture de systemd

systemd n'est pas seulement un programme qui « lance des scripts au démarrage ».

Il forme un ensemble de composants spécialisés.

Parmi eux :

- `systemd` : gestionnaire système / PID 1 ;
- `systemctl` : interface d'administration ;
- `journald` : collecte de journaux ;
- `logind` : sessions et sièges ;
- `udevd` : événements périphériques ;
- `tmpfiles` : création/nettoyage de fichiers temporaires ;
- `sysusers` : création déclarative d'utilisateurs/groupes système ;
- `networkd` : gestion réseau sur certaines installations ;
- `resolved` : résolution de noms sur certaines installations ;
- `timesyncd` : synchronisation simple de l'heure sur certaines installations ;
- `systemd-run` : unités transitoires.

Toutes les distributions n'activent pas tous ces composants.

# 6.1 systemd et D-Bus

systemd expose une API via **D-Bus**.

`systemctl` n'est donc pas simplement un wrapper qui exécute des scripts shell : il dialogue avec le gestionnaire systemd.

# 6.2 systemd et cgroups

systemd organise les processus dans la hiérarchie des **control groups**.

Cela permet notamment :

- de suivre tous les descendants d'un service ;
- de les arrêter comme un groupe ;
- de limiter CPU et mémoire ;
- d'organiser services, sessions et conteneurs.

Pour observer la hiérarchie :

```bash
systemd-cgls
```

# 6.3 Activation à la demande

Un service ne doit pas nécessairement démarrer dès le boot.

systemd sait déclencher du travail par :

- socket ;
- timer ;
- changement de chemin ;
- périphérique ;
- dépendance ;
- requête D-Bus ;
- montage.

---

# 7. Les unités systemd

L'objet central de systemd est l'**unité** (*unit*).

Une unité possède un nom avec un suffixe indiquant son type.

# 7.1 Principaux types

| Suffixe | Rôle |
|---|---|
| `.service` | processus/service |
| `.socket` | socket d'activation |
| `.timer` | déclenchement temporel |
| `.path` | surveillance d'un chemin |
| `.mount` | montage |
| `.automount` | automontage |
| `.target` | point de regroupement/synchronisation |
| `.device` | périphérique exposé à systemd |
| `.swap` | espace swap |
| `.slice` | groupe de ressources cgroup |
| `.scope` | processus externes enregistrés dans systemd |

Il existe d'autres types plus spécialisés.

# 7.2 Voir les unités chargées

```bash
systemctl list-units
```

Uniquement les services :

```bash
systemctl list-units --type=service
```

Unités en échec :

```bash
systemctl --failed
```

# 7.3 Unités chargées et fichiers d'unités

Ne pas confondre :

```bash
systemctl list-units
```

et :

```bash
systemctl list-unit-files
```

La première commande affiche surtout l'état du gestionnaire ; la seconde s'intéresse aux fichiers d'unités installés et à leur état d'activation.

---

# 8. Où sont stockées les unités ?

Les chemins exacts varient légèrement selon les distributions.

Une organisation fréquente est :

```text
/usr/lib/systemd/system/   unités fournies par les paquets
/lib/systemd/system/       chemin historique ou équivalent selon distribution
/run/systemd/system/       unités/configuration transitoires
/etc/systemd/system/       configuration administrateur locale
```

# 8.1 Règle essentielle

Il ne faut généralement **pas modifier directement** l'unité fournie sous `/usr/lib/systemd/system` ou `/lib/systemd/system`.

Une mise à jour du paquet pourrait écraser la modification.

On crée plutôt :

- une unité locale dans `/etc/systemd/system/` ;
- ou un **drop-in**.

# 8.2 Voir l'unité effective

```bash
systemctl cat ssh.service
```

Le nom du service SSH peut être `ssh.service` ou `sshd.service` selon la distribution.

# 8.3 Modifier avec un drop-in

```bash
sudo systemctl edit exemple.service
```

Cela crée typiquement :

```text
/etc/systemd/system/exemple.service.d/override.conf
```

Exemple :

```ini
[Service]
Restart=on-failure
RestartSec=5s
```

Puis :

```bash
sudo systemctl daemon-reload
sudo systemctl restart exemple.service
```

`systemctl edit` lance généralement le `daemon-reload` nécessaire lorsqu'il termine avec succès, mais comprendre explicitement cette étape reste important.

# 8.4 Voir les propriétés calculées

```bash
systemctl show exemple.service
```

Exemples ciblés :

```bash
systemctl show exemple.service -p FragmentPath -p DropInPaths
systemctl show exemple.service -p ActiveState -p SubState
```

---

# 9. Administrer les services avec systemctl

# 9.1 État

```bash
systemctl status nginx.service
```

Le suffixe `.service` peut souvent être omis :

```bash
systemctl status nginx
```

# 9.2 Démarrer et arrêter

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
```

# 9.3 Recharger la configuration du daemon

Si le programme sait relire sa configuration sans redémarrer :

```bash
sudo systemctl reload nginx
```

Cela n'a rien à voir avec :

```bash
sudo systemctl daemon-reload
```

`daemon-reload` demande **à systemd** de relire les fichiers d'unités.

# 9.4 Différence entre start et enable

`start` agit maintenant :

```bash
sudo systemctl start nginx
```

`enable` prépare l'activation pour les boots ou targets futurs :

```bash
sudo systemctl enable nginx
```

Les deux opérations sont indépendantes.

Pour faire les deux :

```bash
sudo systemctl enable --now nginx
```

# 9.5 Désactiver

```bash
sudo systemctl disable nginx
```

ou désactiver et arrêter :

```bash
sudo systemctl disable --now nginx
```

# 9.6 Mask

`mask` est plus fort que `disable`.

```bash
sudo systemctl mask exemple.service
```

Une unité masquée ne peut normalement pas être démarrée, même par dépendance.

Pour annuler :

```bash
sudo systemctl unmask exemple.service
```

# 9.7 Tester les états dans un script

```bash
systemctl is-active --quiet nginx.service
systemctl is-enabled --quiet nginx.service
systemctl is-failed --quiet nginx.service
```

Le code de retour est exploitable par un script shell.

# 9.8 Presets

Les **presets** expriment la politique par défaut d'une distribution ou d'un administrateur sur l'activation des unités.

```bash
systemctl preset exemple.service
```

Il ne faut pas confondre `preset` avec `enable`.

---

# 10. Écrire une unité .service

Prenons une application fictive :

```text
/usr/local/bin/demo-worker
```

# 10.1 Exemple simple

Créer :

```text
/etc/systemd/system/demo-worker.service
```

avec :

```ini
[Unit]
Description=Worker de démonstration
After=network.target

[Service]
Type=exec
ExecStart=/usr/local/bin/demo-worker
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Puis :

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now demo-worker.service
```

# 10.2 Les sections principales

Une unité service contient souvent :

```ini
[Unit]
...

[Service]
...

[Install]
...
```

`[Unit]` décrit notamment les relations avec les autres unités.

`[Service]` décrit le processus et son environnement.

`[Install]` indique comment l'unité est rattachée lors d'un `enable`.

# 10.3 ExecStart n'est pas un shell

C'est une règle fondamentale.

Ceci n'est généralement pas interprété comme dans Bash :

```ini
ExecStart=/usr/bin/echo bonjour > /tmp/message
```

La redirection `>` n'est pas automatiquement interprétée par un shell.

Si un shell est réellement nécessaire :

```ini
ExecStart=/bin/sh -c 'echo bonjour > /tmp/message'
```

Mais il vaut mieux éviter d'ajouter un shell si le programme peut être lancé directement.

# 10.4 Plusieurs commandes

Pour les services qui l'acceptent :

```ini
ExecStartPre=/usr/local/bin/demo-check-config
ExecStart=/usr/local/bin/demo-worker
ExecReload=/bin/kill -HUP $MAINPID
ExecStopPost=/usr/local/bin/demo-cleanup
```

Les règles dépendent du `Type=` du service.

# 10.5 Variables d'environnement

Directement :

```ini
Environment=APP_MODE=production
```

ou depuis un fichier :

```ini
EnvironmentFile=/etc/demo-worker/environment
```

> [!warning]
> Les variables d'environnement ne sont pas une bonne solution pour tous les secrets. Elles peuvent être exposées dans divers contextes. Pour des secrets, voir plus loin les **credentials systemd** et les mécanismes dédiés de l'application.

# 10.6 Utilisateur de service

```ini
User=demo
Group=demo
```

Un daemon réseau n'a pas besoin d'être root simplement parce qu'il tourne en arrière-plan.

# 10.7 Répertoires gérés par systemd

```ini
RuntimeDirectory=demo-worker
StateDirectory=demo-worker
CacheDirectory=demo-worker
LogsDirectory=demo-worker
```

Cela évite souvent des scripts `mkdir/chown` fragiles.

---

# 11. Types de services et supervision

`Type=` indique à systemd comment déterminer que le service a démarré.

# 11.1 Type=exec

Pour de nombreux nouveaux services simples :

```ini
Type=exec
ExecStart=/usr/local/bin/demo-worker
```

`Type=exec` attend que l'exécution du binaire ait réussi avant de considérer la phase de lancement comme franchie.

# 11.2 Type=simple

```ini
Type=simple
ExecStart=/usr/local/bin/demo-worker
```

Le processus configuré devient le processus principal et systemd ne réalise pas une synchronisation supplémentaire de disponibilité.

# 11.3 Type=oneshot

Pour une tâche qui s'exécute puis se termine :

```ini
[Service]
Type=oneshot
ExecStart=/usr/local/sbin/prepare-demo
```

Avec :

```ini
RemainAfterExit=yes
```

l'unité peut rester considérée active après la fin du processus.

# 11.4 Type=notify

Une application intégrée à `sd_notify()` peut informer systemd lorsqu'elle est réellement prête :

```ini
Type=notify
NotifyAccess=main
```

C'est préférable à des délais arbitraires lorsque l'application sait émettre une notification de readiness.

# 11.5 Type=forking

Utilisé pour certains daemons historiques qui se détachent eux-mêmes :

```ini
Type=forking
PIDFile=/run/ancien-daemon.pid
```

Pour un nouveau programme, il est généralement plus simple de **rester au premier plan** et de laisser systemd superviser le processus.

# 11.6 Redémarrage automatique

Pour un service long :

```ini
Restart=on-failure
RestartSec=3s
```

Il faut éviter une boucle de redémarrage incontrôlée.

Les limites de démarrage peuvent être réglées, par exemple :

```ini
[Unit]
StartLimitIntervalSec=60s
StartLimitBurst=5
```

# 11.7 Watchdog

Les applications compatibles peuvent utiliser le watchdog systemd :

```ini
WatchdogSec=30s
```

Le processus doit envoyer les notifications de watchdog attendues.

Le simple ajout de `WatchdogSec=` ne transforme pas une application incompatible en application surveillée correctement.

---

# 12. Dépendances et ordre de démarrage

C'est l'une des parties les plus mal comprises de systemd.

> [!important]
> **Dépendance** et **ordre** sont deux notions orthogonales.

# 12.1 Wants et Requires

```ini
Wants=postgresql.service
```

exprime une dépendance relativement faible.

```ini
Requires=postgresql.service
```

exprime une dépendance plus forte.

Mais cela ne signifie pas à lui seul : « démarre obligatoirement après PostgreSQL ».

# 12.2 After et Before

L'ordre s'exprime séparément :

```ini
After=postgresql.service
```

Pour demander et ordonner :

```ini
[Unit]
Requires=postgresql.service
After=postgresql.service
```

# 12.3 Pourquoi cette séparation ?

Elle permet à systemd de démarrer en parallèle les unités qui n'ont pas de contrainte d'ordre entre elles.

# 12.4 Autres relations utiles

Quelques directives importantes :

- `Wants=` ;
- `Requires=` ;
- `Requisite=` ;
- `BindsTo=` ;
- `PartOf=` ;
- `Conflicts=` ;
- `Before=` ;
- `After=`.

Elles n'ont pas toutes la même sémantique.

# 12.5 PartOf

Exemple :

```ini
[Unit]
PartOf=demo.target
```

Cela peut permettre de propager certaines opérations d'arrêt/redémarrage depuis l'unité indiquée.

# 12.6 Réseau : network.target n'est pas “Internet prêt”

Un piège fréquent :

```ini
After=network.target
```

ne garantit pas que :

- une adresse IP est configurée ;
- DNS fonctionne ;
- Internet est joignable ;
- un serveur distant est prêt.

Pour un service qui exige réellement un réseau configuré, on peut avoir besoin de :

```ini
Wants=network-online.target
After=network-online.target
```

mais cette synchronisation peut ralentir le boot et doit être utilisée uniquement si le service en a réellement besoin.

Un serveur qui **fournit** un service réseau n'a généralement pas besoin d'attendre `network-online.target`.

# 12.7 Voir les dépendances

```bash
systemctl list-dependencies demo-worker.service
```

Sens inverse :

```bash
systemctl list-dependencies --reverse demo-worker.service
```

---

# 13. Targets et anciens runlevels

Une `.target` sert principalement à **regrouper et synchroniser** des unités.

Exemples :

- `basic.target` ;
- `multi-user.target` ;
- `graphical.target` ;
- `rescue.target` ;
- `emergency.target` ;
- `shutdown.target`.

# 13.1 Target par défaut

```bash
systemctl get-default
```

Changer :

```bash
sudo systemctl set-default multi-user.target
```

ou :

```bash
sudo systemctl set-default graphical.target
```

# 13.2 Changer la target courante

```bash
sudo systemctl isolate multi-user.target
```

> [!warning]
> `isolate` peut arrêter les unités qui ne font pas partie de la nouvelle target. Il ne faut pas l'utiliser à distance sans comprendre les conséquences, notamment sur le réseau et la session SSH.

# 13.3 Runlevels SysV

Historiquement, SysVinit utilise des niveaux numériques.

La sémantique exacte varie selon les distributions.

Une représentation classique est :

| Runlevel | Signification courante |
|---|---|
| 0 | arrêt |
| 1 | mode mono-utilisateur / secours |
| 2–4 | modes multi-utilisateurs selon distribution |
| 5 | souvent graphique sur certaines distributions |
| 6 | redémarrage |

Il est faux de considérer les niveaux 2, 3 et 5 comme universellement identiques sur tous les Unix/Linux.

# 13.4 Aliases de compatibilité systemd

systemd fournit historiquement des aliases tels que :

```text
runlevel3.target → multi-user.target
runlevel5.target → graphical.target
```

Le modèle natif à utiliser reste cependant celui des **targets nommées**.

# 13.5 Modes rescue et emergency

Pour basculer en mode secours :

```bash
sudo systemctl rescue
```

Le mode emergency est encore plus minimal :

```bash
sudo systemctl emergency
```

Ces commandes perturbent fortement le système courant.

---

# 14. Journalisation avec journalctl

`systemd-journald` collecte de nombreux événements :

- stdout/stderr des services ;
- messages du noyau ;
- messages syslog transférés ;
- métadonnées sur les unités et processus.

# 14.1 Logs d'un service

```bash
journalctl -u nginx.service
```

Suivre en temps réel :

```bash
journalctl -fu nginx.service
```

# 14.2 Boot courant

```bash
journalctl -b
```

Boot précédent :

```bash
journalctl -b -1
```

Lister les boots connus :

```bash
journalctl --list-boots
```

# 14.3 Messages du noyau

```bash
journalctl -k
```

Équivalent limité au boot courant :

```bash
journalctl -k -b
```

# 14.4 Priorité

Uniquement `err` et plus grave :

```bash
journalctl -p err
```

Depuis aujourd'hui :

```bash
journalctl --since today
```

# 14.5 Logs structurés

Le journal conserve des champs comme :

- `_SYSTEMD_UNIT` ;
- `_PID` ;
- `_UID` ;
- `_COMM` ;
- `PRIORITY`.

Exemple :

```bash
journalctl _SYSTEMD_UNIT=sshd.service
```

# 14.6 Persistance

Selon la configuration, le journal peut être :

- volatile dans `/run/log/journal` ;
- persistant sous `/var/log/journal`.

Pour connaître l'utilisation disque :

```bash
journalctl --disk-usage
```

Nettoyage contrôlé :

```bash
sudo journalctl --vacuum-time=30d
```

Il faut définir une politique adaptée au besoin d'audit et à l'espace disque.

---

# 15. Timers systemd

Les timers peuvent remplacer de nombreux usages de cron tout en bénéficiant :

- des dépendances systemd ;
- du journal ;
- de la supervision ;
- des cgroups ;
- de la persistance optionnelle ;
- d'un délai aléatoire contrôlé.

# 15.1 Service associé

`/etc/systemd/system/demo-backup.service` :

```ini
[Unit]
Description=Sauvegarde de démonstration

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/demo-backup
```

# 15.2 Timer

`/etc/systemd/system/demo-backup.timer` :

```ini
[Unit]
Description=Lance la sauvegarde quotidienne

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=15m

[Install]
WantedBy=timers.target
```

Puis :

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now demo-backup.timer
```

# 15.3 Voir les timers

```bash
systemctl list-timers --all
```

# 15.4 Tester une expression OnCalendar

```bash
systemd-analyze calendar 'Mon..Fri 08:00'
```

# 15.5 Persistent=true

Si le timer aurait dû s'exécuter pendant l'arrêt de la machine, `Persistent=true` permet généralement de déclencher la tâche après le prochain démarrage.

Cela concerne les timers calendaires concernés, pas toutes les formes de timer.

# 15.6 RandomizedDelaySec

Sur une flotte de milliers de machines, on évite de lancer une tâche exactement à la même seconde partout :

```ini
RandomizedDelaySec=30m
```

Cela réduit les effets de troupeau (*thundering herd*).

# 15.7 Monotonic timers

Exemples :

```ini
OnBootSec=10min
OnUnitActiveSec=1h
```

Ces timers ne reposent pas uniquement sur l'heure du calendrier.

---

# 16. Activation par socket et par chemin

# 16.1 Socket activation

systemd peut ouvrir une socket avant le daemon.

Le service n'est démarré que lorsqu'une connexion arrive.

Exemple conceptuel :

```ini
[Socket]
ListenStream=127.0.0.1:9000
```

Le service doit être conçu ou configuré pour accepter la socket transmise par systemd.

Cette technique peut :

- paralléliser le boot ;
- lancer un service à la demande ;
- conserver un point d'écoute pendant certains redémarrages.

# 16.2 Voir les sockets

```bash
systemctl list-sockets
```

# 16.3 Path activation

Une unité `.path` peut déclencher une unité lorsqu'un chemin change.

Exemple :

```ini
[Path]
PathExists=/var/lib/demo/trigger
Unit=demo-import.service

[Install]
WantedBy=paths.target
```

Activer :

```bash
sudo systemctl enable --now demo-import.path
```

# 16.4 Limites de Path units

Ce mécanisme ne remplace pas un bus d'événements fiable pour tous les besoins.

Il faut réfléchir à :

- la fréquence des événements ;
- les courses ;
- la persistance ;
- la sémantique attendue en cas de redémarrage.

---

# 17. Montages, automounts et générateurs

systemd transforme de nombreuses ressources système en unités.

# 17.1 Mount units

Un montage `/srv/data` correspond à :

```text
srv-data.mount
```

Le nom résulte d'un échappement de chemin.

Pour le calculer :

```bash
systemd-escape --path --suffix=mount /srv/data
```

# 17.2 /etc/fstab et générateurs

systemd utilise des **générateurs** au démarrage et lors de certains reloads pour produire des unités à partir de sources externes comme `/etc/fstab`.

Ainsi, conserver `/etc/fstab` est parfaitement compatible avec systemd.

# 17.3 Voir les montages systemd

```bash
systemctl list-units --type=mount
```

# 17.4 Automount

Une unité `.automount` permet de retarder le montage jusqu'au premier accès.

Cela peut être utile pour :

- NFS ;
- systèmes de fichiers réseau ;
- ressources rarement utilisées.

# 17.5 Dépendances réseau des montages distants

Les systèmes de fichiers réseau sont traités spécialement et peuvent tirer `network-online.target` selon leur configuration.

Il faut éviter de fabriquer manuellement des dépendances réseau contradictoires lorsqu'une unité de montage en crée déjà.

---

# 18. Services utilisateur

systemd existe aussi au niveau de l'utilisateur.

# 18.1 Gestionnaire utilisateur

Une session peut avoir son propre :

```text
systemd --user
```

Les commandes s'exécutent avec :

```bash
systemctl --user status
```

# 18.2 Emplacement des unités utilisateur

Emplacement courant :

```text
~/.config/systemd/user/
```

Exemple :

```text
~/.config/systemd/user/sync-notes.service
```

# 18.3 Exemple

```ini
[Unit]
Description=Synchronisation des notes

[Service]
Type=oneshot
ExecStart=%h/bin/sync-notes
```

Puis :

```bash
systemctl --user daemon-reload
systemctl --user start sync-notes.service
```

# 18.4 Linger

Normalement, la vie du gestionnaire utilisateur dépend des sessions et de la politique de logind.

Pour certains services utilisateur devant continuer sans session ouverte :

```bash
loginctl enable-linger "$USER"
```

Cette décision a des conséquences en ressources et en sécurité.

# 18.5 Variables de session

Un service `--user` ne récupère pas automatiquement tout l'environnement interactif du shell.

Il faut donc éviter de supposer que :

- `PATH` est identique ;
- un agent SSH est disponible ;
- des variables exportées dans `.bashrc` existent.

---

# 19. cgroups, slices et gestion des ressources

Sur un système moderne, systemd s'appuie sur les **cgroups**, souvent en hiérarchie unifiée cgroup v2.

# 19.1 Voir l'arbre

```bash
systemd-cgls
```

# 19.2 Slices

Les services sont organisés dans des slices.

Exemples :

```text
system.slice
user.slice
machine.slice
```

# 19.3 Limiter la mémoire

```ini
[Service]
MemoryMax=512M
```

# 19.4 Limiter le CPU

```ini
[Service]
CPUQuota=50%
```

# 19.5 Limiter le nombre de tâches

```ini
[Service]
TasksMax=200
```

# 19.6 Appliquer une propriété temporairement

Exemple :

```bash
sudo systemctl set-property demo-worker.service MemoryMax=512M
```

Selon les options utilisées, la modification peut être persistante.

# 19.7 Observer les ressources

```bash
systemd-cgtop
```

# 19.8 Pourquoi limiter ?

Un daemon défaillant ne devrait pas pouvoir :

- consommer toute la RAM ;
- créer un nombre illimité de processus ;
- monopoliser tous les CPU.

La limite systemd complète les contrôles applicatifs et la supervision.

---

# 20. Identité, répertoires et credentials

# 20.1 DynamicUser

Pour certains services qui n'ont pas besoin d'une identité Unix permanente :

```ini
[Service]
DynamicUser=yes
```

systemd attribue dynamiquement une identité de service.

Ce mécanisme fonctionne particulièrement bien avec :

```ini
StateDirectory=demo
CacheDirectory=demo
LogsDirectory=demo
```

# 20.2 RuntimeDirectory

```ini
RuntimeDirectory=demo
```

crée un emplacement approprié sous `/run` pour le service.

# 20.3 Credentials systemd

Les versions modernes de systemd proposent un mécanisme de **credentials** pour transmettre des données sensibles ou de configuration à un service sous forme de fichiers contrôlés.

Exemple :

```ini
[Service]
LoadCredential=api-token:/etc/demo/api-token
ExecStart=/usr/local/bin/demo-worker
```

Le programme reçoit un répertoire de credentials via l'environnement défini par systemd, plutôt qu'un secret directement placé dans la ligne de commande.

Il faut quand même protéger correctement le fichier source et comprendre le modèle de menace.

# 20.4 systemd-creds

`systemd-creds` permet notamment de manipuler des credentials et, selon la plateforme et la configuration, de les chiffrer/sceller.

Voir :

```bash
systemd-creds --help
man systemd-creds
```

Les fonctions disponibles dépendent de la version systemd et du matériel.

---

# 21. Durcissement des services

systemd fournit de nombreuses directives qui exploitent des mécanismes du noyau Linux pour réduire les privilèges d'un service.

> [!important]
> Le durcissement doit être **testé**. Activer aveuglément toutes les options peut casser le service.

# 21.1 NoNewPrivileges

```ini
NoNewPrivileges=yes
```

empêche le processus et ses descendants de gagner de nouveaux privilèges par certains mécanismes comme setuid/setgid et capabilities de fichiers.

# 21.2 PrivateTmp

```ini
PrivateTmp=yes
```

isole les espaces temporaires du service.

# 21.3 ProtectSystem

```ini
ProtectSystem=strict
```

rend de larges parties du système de fichiers en lecture seule ou inaccessibles dans le namespace de montage du service.

On ouvre ensuite précisément les zones d'écriture nécessaires avec les directives prévues à cet effet.

# 21.4 ProtectHome

```ini
ProtectHome=yes
```

réduit l'accès aux répertoires utilisateurs.

# 21.5 PrivateDevices

```ini
PrivateDevices=yes
```

limite l'accès aux périphériques.

# 21.6 Protection du noyau

Exemples :

```ini
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectKernelLogs=yes
ProtectControlGroups=yes
```

# 21.7 Capabilities

Pour un service qui doit simplement écouter sur un port privilégié, il est souvent inutile de lui donner tous les privilèges root.

Exemple :

```ini
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
```

Il faut comprendre les implications des capabilities avant de les attribuer.

# 21.8 RestrictAddressFamilies

Pour un service uniquement IPv4/IPv6 :

```ini
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
```

# 21.9 RestrictNamespaces

```ini
RestrictNamespaces=yes
```

peut empêcher la création de namespaces supplémentaires si le service n'en a pas besoin.

# 21.10 SystemCallFilter

systemd peut limiter les familles d'appels système via seccomp :

```ini
SystemCallFilter=@system-service
```

Il faut vérifier la compatibilité réelle avec le programme.

# 21.11 Exemple durci

```ini
[Unit]
Description=API de démonstration
After=network.target

[Service]
Type=exec
DynamicUser=yes
ExecStart=/usr/local/bin/demo-api
Restart=on-failure
RestartSec=5s

StateDirectory=demo-api
RuntimeDirectory=demo-api

NoNewPrivileges=yes
PrivateTmp=yes
PrivateDevices=yes
ProtectSystem=strict
ProtectHome=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectKernelLogs=yes
ProtectControlGroups=yes
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
RestrictNamespaces=yes
LockPersonality=yes
MemoryDenyWriteExecute=yes

MemoryMax=512M
TasksMax=200

[Install]
WantedBy=multi-user.target
```

`MemoryDenyWriteExecute=yes` peut être incompatible avec certains runtimes JIT. Il faut alors adapter le profil plutôt que désactiver sans analyse l'ensemble du durcissement.

# 21.12 Évaluer le durcissement

```bash
systemd-analyze security demo-api.service
```

Le score obtenu est **un indicateur**, pas une preuve que le service est sécurisé.

Une application vulnérable reste vulnérable même dans une sandbox bien configurée.

# 21.13 Voir le sandboxing effectif

```bash
systemctl show demo-api.service \
  -p NoNewPrivileges \
  -p ProtectSystem \
  -p ProtectHome \
  -p PrivateTmp \
  -p CapabilityBoundingSet
```

---

# 22. Analyse des performances de démarrage

# 22.1 Temps global

```bash
systemd-analyze time
```

Sortie typique :

```text
Startup finished in ... (kernel) + ... (userspace) = ...
```

# 22.2 Services les plus longs

```bash
systemd-analyze blame
```

> [!warning]
> Une unité longue dans `blame` n'est pas forcément la cause du retard global : des unités peuvent s'exécuter en parallèle.

# 22.3 Chemin critique

```bash
systemd-analyze critical-chain
```

Pour une unité :

```bash
systemd-analyze critical-chain graphical.target
```

Le **critical chain** est souvent plus utile que `blame` pour comprendre ce qui bloque réellement une target.

# 22.4 Graphe SVG

```bash
systemd-analyze plot > boot.svg
```

On peut ensuite ouvrir `boot.svg` dans un navigateur.

# 22.5 Vérifier une unité

```bash
systemd-analyze verify /etc/systemd/system/demo-worker.service
```

Cette validation détecte plusieurs erreurs de configuration, mais pas toutes les erreurs fonctionnelles du programme.

---

# 23. Diagnostic et récupération

# 23.1 Méthode de diagnostic

Lorsqu'un service ne démarre pas :

1. observer son état ;
2. lire ses logs ;
3. lire le fichier d'unité effectif ;
4. vérifier les dépendances ;
5. vérifier utilisateur, chemins et permissions ;
6. tester le programme hors systemd si cela a du sens ;
7. vérifier les mécanismes de sandboxing ;
8. vérifier les limites de ressources ;
9. vérifier l'environnement réellement disponible.

# 23.2 État détaillé

```bash
systemctl status demo-worker.service
```

# 23.3 Logs

```bash
journalctl -u demo-worker.service -b --no-pager
```

# 23.4 Configuration effective

```bash
systemctl cat demo-worker.service
```

# 23.5 Propriétés

```bash
systemctl show demo-worker.service
```

# 23.6 Reset failed

Après correction :

```bash
sudo systemctl reset-failed demo-worker.service
```

Cela réinitialise l'état failed et certains compteurs associés ; cela ne corrige pas la cause de la panne.

# 23.7 Processus du service

```bash
systemctl status demo-worker.service
systemd-cgls /system.slice/demo-worker.service
```

# 23.8 Pourquoi “ça marche dans mon shell” mais pas dans systemd ?

Causes fréquentes :

- `PATH` différent ;
- mauvais `WorkingDirectory` ;
- utilisateur différent ;
- droits manquants ;
- variables de shell non chargées ;
- absence de TTY ;
- secret non disponible ;
- sandbox systemd ;
- réseau pas encore disponible ;
- fichier relatif supposé dans le répertoire courant.

Un bon service utilise des **chemins absolus** et un environnement explicite.

# 23.9 Emergency target depuis GRUB

On peut ajouter temporairement à la ligne du noyau :

```text
systemd.unit=emergency.target
```

ou :

```text
systemd.unit=rescue.target
```

La différence entre rescue et emergency doit être connue avant usage : emergency fournit un environnement encore plus minimal.

# 23.10 Boot qui échoue à cause d'un montage

Examiner :

```bash
systemctl --failed
journalctl -b -p warning
systemctl status local-fs.target
```

Puis vérifier `/etc/fstab`, les UUID et les options.

```bash
findmnt --verify
blkid
```

---

# 24. Unités transitoires avec systemd-run

`systemd-run` permet de lancer un programme dans une unité créée dynamiquement.

# 24.1 Lancer une commande supervisée

```bash
systemd-run --unit=demo-date /usr/bin/date
```

# 24.2 Avec limite mémoire

```bash
systemd-run --unit=demo-job \
  --property=MemoryMax=256M \
  /usr/local/bin/demo-job
```

# 24.3 Scope interactif

Pour exécuter une commande dans un scope :

```bash
systemd-run --scope /usr/bin/bash
```

# 24.4 Timer transitoire

```bash
systemd-run --on-active=10m /usr/local/bin/demo-job
```

Pour des tâches de production permanentes, une vraie paire `.service`/`.timer` versionnée est souvent plus lisible.

---

# 25. Concevoir un service de production

Prenons un daemon Python fictif installé dans :

```text
/opt/myapi/.venv/bin/myapi
```

Il doit :

- écouter sur un port non privilégié ;
- fonctionner sans root ;
- écrire son état sous `/var/lib/myapi` ;
- être redémarré en cas d'échec ;
- disposer de limites de ressources ;
- ne pas accéder aux répertoires personnels.

# 25.1 Unité complète

```ini
[Unit]
Description=API MyAPI
Documentation=https://example.invalid/docs/myapi
Wants=network-online.target
After=network-online.target

[Service]
Type=exec
DynamicUser=yes
ExecStart=/opt/myapi/.venv/bin/myapi --config /etc/myapi/config.toml
Restart=on-failure
RestartSec=5s
TimeoutStartSec=30s
TimeoutStopSec=30s

StateDirectory=myapi
RuntimeDirectory=myapi
CacheDirectory=myapi

NoNewPrivileges=yes
PrivateTmp=yes
PrivateDevices=yes
ProtectSystem=strict
ProtectHome=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectKernelLogs=yes
ProtectControlGroups=yes
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
RestrictNamespaces=yes
LockPersonality=yes

MemoryMax=1G
TasksMax=256

[Install]
WantedBy=multi-user.target
```

# 25.2 Questionner chaque directive

Même dans cet exemple, il faut se demander :

- l'application a-t-elle réellement besoin de `network-online.target` ?
- utilise-t-elle un JIT qui rendrait certaines protections incompatibles ?
- doit-elle lire des certificats ?
- doit-elle écrire ailleurs que dans `StateDirectory` ?
- a-t-elle besoin d'un accès à un device ?
- doit-elle recevoir un credential ?

Une unité de production est un **contrat explicite de dépendances et de privilèges**.

# 25.3 Déploiement

Après installation :

```bash
sudo systemd-analyze verify /etc/systemd/system/myapi.service
sudo systemctl daemon-reload
sudo systemctl enable --now myapi.service
systemctl status myapi.service
```

Puis :

```bash
journalctl -u myapi.service -b
systemd-analyze security myapi.service
```

# 25.4 Mise à jour atomique

Lors d'un déploiement applicatif :

1. préparer la nouvelle version ;
2. vérifier sa configuration ;
3. basculer le chemin/version ;
4. redémarrer ou recharger le service ;
5. vérifier readiness et logs ;
6. garder une stratégie de rollback.

systemd ne remplace pas un mécanisme de déploiement, mais fournit une supervision fiable du processus.

---

# 26. SysVinit, Upstart et autres init systems

Ce chapitre est principalement historique et sert à comprendre de vieux systèmes.

# 26.1 SysVinit

SysVinit s'organise traditionnellement autour de :

- `/etc/inittab` ;
- scripts `/etc/init.d/` ;
- répertoires `/etc/rc?.d/` ;
- runlevels numériques.

Les scripts commencent souvent par des liens `Sxx` ou `Kxx` selon le démarrage ou l'arrêt.

# 26.2 Pourquoi systemd a changé le modèle

Les limites des approches traditionnelles comprennent notamment :

- dépendances parfois implicites ;
- scripts shell complexes ;
- difficulté de supervision de processus qui forkent ;
- parallélisation limitée ou fragile ;
- intégration inégale des logs et ressources.

systemd propose au contraire un graphe d'unités déclaratif et une supervision basée sur les cgroups.

# 26.3 Upstart

Upstart était un init **événementiel** développé initialement par Canonical.

Il a notamment été utilisé par Ubuntu avant systemd.

Concepts historiques :

- jobs ;
- événements ;
- `initctl` ;
- fichiers de configuration Upstart.

Il ne faut pas apprendre Upstart comme solution recommandée pour une nouvelle distribution moderne.

# 26.4 systemd 260 et fin du convertisseur SysV amont

Une évolution importante de 2026 : systemd 260 a retiré le support natif amont des scripts de service SysV.

Conséquence pratique pour un éditeur logiciel :

> fournir une vraie unité `.service` et ne plus supposer qu'un `/etc/init.d/mon-service` sera automatiquement traduit par toutes les versions futures de systemd.

# 26.5 OpenRC, runit et s6

systemd n'est pas l'unique modèle possible.

D'autres systèmes privilégient :

- simplicité ;
- supervision indépendante ;
- portabilité Unix ;
- composition d'outils spécialisés.

Le choix dépend de la distribution et des objectifs techniques.

---

# 27. Travaux pratiques

# TP 1 — Reconstituer son boot

Objectif : identifier toute la chaîne de démarrage de sa machine.

Commandes :

```bash
systemd --version
ps -p 1 -o pid,comm,args
cat /proc/cmdline
uname -r
findmnt /
findmnt /boot/efi 2>/dev/null || true
```

Questions :

1. BIOS ou UEFI ?
2. Quel est PID 1 ?
3. Quel noyau tourne ?
4. Où est la racine ?
5. Quel initramfs correspond au noyau ?

# TP 2 — Examiner GRUB sans rien modifier

```bash
cat /etc/default/grub
ls -l /boot/grub 2>/dev/null
```

Repérer :

- timeout ;
- options du noyau ;
- éventuel mode recovery.

Ne pas modifier `grub.cfg`.

# TP 3 — Explorer les unités

```bash
systemctl list-units --type=service
systemctl list-unit-files --type=service
systemctl --failed
```

Choisir un service et afficher :

```bash
systemctl cat cron.service 2>/dev/null || systemctl cat crond.service
```

# TP 4 — Créer un oneshot

Créer `/usr/local/bin/hello-systemd` :

```bash
#!/bin/sh
set -eu
printf 'Bonjour depuis systemd à %s\n' "$(date -Is)"
```

Puis :

```bash
sudo chmod 0755 /usr/local/bin/hello-systemd
```

Créer :

```text
/etc/systemd/system/hello-systemd.service
```

```ini
[Unit]
Description=TP oneshot systemd

[Service]
Type=oneshot
ExecStart=/usr/local/bin/hello-systemd
```

Tester :

```bash
sudo systemctl daemon-reload
sudo systemctl start hello-systemd.service
journalctl -u hello-systemd.service --no-pager
```

# TP 5 — Transformer le TP en timer

Créer :

```text
/etc/systemd/system/hello-systemd.timer
```

```ini
[Unit]
Description=Déclenche hello-systemd

[Timer]
OnBootSec=2min
OnUnitActiveSec=1h

[Install]
WantedBy=timers.target
```

Puis :

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now hello-systemd.timer
systemctl list-timers --all
```

# TP 6 — Comprendre l'ordre et la dépendance

Créer deux oneshots `database-ready.service` et `app-start.service`.

Dans le second :

```ini
[Unit]
Requires=database-ready.service
After=database-ready.service
```

Comparer avec une version qui ne contient que `Requires=`.

Observer :

```bash
systemctl list-dependencies app-start.service
```

# TP 7 — Journal et boot précédent

```bash
journalctl --list-boots
journalctl -b -p warning
journalctl -b -1 -p err
```

Identifier trois événements significatifs et leur unité d'origine.

# TP 8 — Limiter un service

Ajouter temporairement à un service de laboratoire :

```ini
[Service]
MemoryMax=128M
TasksMax=32
```

Puis observer :

```bash
systemctl show hello-systemd.service -p MemoryMax -p TasksMax
```

# TP 9 — Durcir progressivement

Sur un service de laboratoire, tester successivement :

```ini
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
PrivateDevices=yes
```

Après chaque changement :

```bash
sudo systemctl daemon-reload
sudo systemctl restart mon-lab.service
systemctl status mon-lab.service
journalctl -u mon-lab.service -n 30
```

Objectif : comprendre quelle ressource devient inaccessible et pourquoi.

# TP 10 — Analyser le temps de boot

```bash
systemd-analyze time
systemd-analyze blame | head -20
systemd-analyze critical-chain
```

Expliquer pourquoi la première ligne de `blame` n'est pas forcément « le coupable » du boot lent.

# TP 11 — Service utilisateur

Créer :

```text
~/.config/systemd/user/hello-user.service
```

```ini
[Unit]
Description=Service utilisateur de démonstration

[Service]
Type=oneshot
ExecStart=/usr/bin/printf Bonjour-systemd-user
```

Puis :

```bash
systemctl --user daemon-reload
systemctl --user start hello-user.service
systemctl --user status hello-user.service
```

# TP 12 — Audit d'une unité existante

Choisir un vrai service et produire une fiche avec :

- fichier source ;
- drop-ins ;
- utilisateur ;
- `Type=` ;
- dépendances ;
- target d'activation ;
- stratégie de restart ;
- limites CPU/mémoire ;
- mesures de sandboxing ;
- logs ;
- score `systemd-analyze security` ;
- trois améliorations possibles.

---

# 28. Checklist et commandes de référence

# 28.1 Avant de créer un service

- [ ] Le programme peut-il rester au premier plan ?
- [ ] Quel utilisateur doit l'exécuter ?
- [ ] A-t-il vraiment besoin de root ?
- [ ] Quels fichiers doit-il lire ?
- [ ] Où doit-il écrire ?
- [ ] De quel réseau a-t-il réellement besoin ?
- [ ] Doit-il attendre le réseau ou simplement écouter ?
- [ ] Quelle politique de redémarrage est appropriée ?
- [ ] Quels timeouts sont raisonnables ?
- [ ] Quelles limites CPU/RAM/processus faut-il fixer ?
- [ ] Quelles protections systemd sont compatibles ?
- [ ] Comment seront fournis les secrets ?
- [ ] Comment vérifier sa disponibilité ?
- [ ] Comment revenir à la version précédente ?

# 28.2 Commandes quotidiennes

```bash
systemctl status SERVICE
systemctl start SERVICE
systemctl stop SERVICE
systemctl restart SERVICE
systemctl reload SERVICE
systemctl enable SERVICE
systemctl enable --now SERVICE
systemctl disable SERVICE
systemctl mask SERVICE
systemctl unmask SERVICE
systemctl cat SERVICE
systemctl show SERVICE
systemctl --failed
```

# 28.3 Après modification d'une unité

```bash
sudo systemctl daemon-reload
sudo systemd-analyze verify /etc/systemd/system/mon-service.service
sudo systemctl restart mon-service.service
systemctl status mon-service.service
journalctl -u mon-service.service -b
```

# 28.4 Diagnostic du boot

```bash
systemd-analyze time
systemd-analyze blame
systemd-analyze critical-chain
journalctl -b
journalctl -b -1
systemctl --failed
```

# 28.5 Timers

```bash
systemctl list-timers --all
systemd-analyze calendar daily
```

# 28.6 Ressources

```bash
systemd-cgls
systemd-cgtop
systemctl show SERVICE -p MemoryCurrent -p MemoryMax -p TasksCurrent -p TasksMax
```

# 28.7 Sécurité

```bash
systemd-analyze security SERVICE
systemctl show SERVICE -p NoNewPrivileges -p ProtectSystem -p ProtectHome
```

---

# À retenir

1. Le démarrage est une chaîne **firmware → bootloader → noyau → initramfs → PID 1**.
2. La terminologie **stage 1 / 1.5 / 2 appartient à GRUB Legacy**, pas à l'architecture réelle de GRUB 2.
3. systemd est le PID 1 de nombreuses distributions, mais c'est aussi un gestionnaire d'unités, de dépendances, de ressources et d'activation.
4. Une dépendance `Requires=` ou `Wants=` n'implique pas automatiquement un ordre : `After=`/`Before=` est une dimension distincte.
5. `start` et `enable` ne signifient pas la même chose.
6. Les targets sont le modèle natif de systemd ; les runlevels numériques ne sont plus qu'un héritage de compatibilité.
7. Les timers, sockets et paths permettent de démarrer des tâches à la demande plutôt que tout au boot.
8. `journalctl` est l'outil central pour diagnostiquer les services systemd.
9. Les cgroups permettent de suivre et limiter les ressources d'un service.
10. Le sandboxing systemd réduit la surface d'attaque, mais doit être adapté au programme et ne remplace pas la sécurité applicative.
11. Depuis systemd 260, le projet amont ne prend plus directement en charge les scripts de service SysV : une application moderne doit fournir une unité native.

# Références

## systemd

- Documentation systemd : <https://systemd.io/>
- Pages de manuel : <https://www.freedesktop.org/software/systemd/man/latest/>
- `systemd.unit` : <https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html>
- `systemd.service` : <https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html>
- `systemd.exec` : <https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html>
- `systemd.timer` : <https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html>
- `systemd.special` : <https://www.freedesktop.org/software/systemd/man/latest/systemd.special.html>
- `systemd-analyze` : <https://www.freedesktop.org/software/systemd/man/latest/systemd-analyze.html>
- Releases systemd : <https://github.com/systemd/systemd/releases>

## GRUB

- Manuel GNU GRUB 2 : <https://www.gnu.org/software/grub/manual/grub/grub.html>

## Debian

- Debian Wiki — systemd : <https://wiki.debian.org/systemd>
- Debian Wiki — Init : <https://wiki.debian.org/Init>

## Noyau Linux

- Paramètres du noyau : <https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html>

## Cours liés

- [[GNULinux]]
- [[Les namespaces Linux]]
- [[proc]]
- [[Sécurité avancée sous Linux]]
