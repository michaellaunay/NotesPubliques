---
schema_version: 1
uid: "01M02EX5B7KS75F157R5W7JZXC"
titre: "Installation Ubuntu"
type: procedure
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - gnu-linux
  - ubuntu
  - installation
resume: "Installer proprement Ubuntu Desktop ou Server : choix de version, vérification de l'image, UEFI/Secure Boot, partitionnement, LVM/LUKS, réseau, SSH, post-installation, automatisation Subiquity/autoinstall et dépannage."
niveau: debutant
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-04-30
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre GNU/Linux
> Ce cours est repris dans le livre [[GNU Linux — Sommaire]] — chapitre [[GNU Linux — 04 — Installer Ubuntu|4]].

> [!info] Refonte
> Cours développé le 31 août 2026 (d'environ 1 373 à 7 194 mots  remplacement complet du texte précédent)  vérifié le même jour : schéma  titres  liens et affirmations datées contrôlés. La version précédente reste dans l'historique git du dépôt. Élément non repris : la procédure pas à pas d'installation d'Ubuntu 22.04 (étapes 01 à 08 et 04 bis)  obsolète  n'est pas reprise.

# Installation d'Ubuntu

> [!abstract] Objectif
> Savoir installer **Ubuntu Desktop** ou **Ubuntu Server** de manière reproductible et sûre, depuis le choix de la version jusqu'aux vérifications post-installation. Le cours couvre le démarrage UEFI, Secure Boot, GPT, LVM, LUKS, le réseau avec Netplan, SSH, les mises à jour et l'installation automatisée avec **Subiquity / autoinstall**.

Voir aussi : [[GNULinux]], [[Les distributions Linux]], [[Initialisation système et des services]], [[Sécurité avancée sous Linux]], [[Les protocoles de communications]], [[proc]], [[Les namespaces Linux]].

> [!important] État du cours au 31 août 2026
> La dernière version LTS publiée est **Ubuntu 26.04.1 LTS « Resolute Raccoon »**, publiée le 27 août 2026. Ubuntu 24.04 LTS reste également supportée. Pour un nouveau serveur ou poste destiné à durer, une **LTS** est généralement le meilleur choix. Les versions intermédiaires sortent tous les six mois mais ne reçoivent que neuf mois de mises à jour.

---

# 1. Ce que signifie « installer Ubuntu »

Installer un système GNU/Linux ne consiste pas seulement à copier des fichiers sur un disque. L'installation prépare plusieurs couches :

```text
Firmware UEFI
   ↓
chargeur d'amorçage
   ↓
noyau Linux + initramfs
   ↓
système de fichiers racine /
   ↓
systemd et services
   ↓
session utilisateur / services serveur
```

Pendant l'installation, il faut donc prendre des décisions sur :

- la version d'Ubuntu ;
- l'architecture CPU ;
- le mode de démarrage UEFI ;
- le partitionnement ;
- le chiffrement ;
- les comptes utilisateurs ;
- le réseau ;
- l'accès SSH pour un serveur ;
- les mises à jour ;
- éventuellement les paquets et services à installer.

La qualité de ces décisions influence directement :

- la sécurité ;
- la maintenabilité ;
- la facilité de sauvegarde ;
- la capacité de restauration ;
- la facilité de montée de version ;
- les performances ;
- la durée de vie de l'installation.

---

# 2. Desktop, Server, Cloud, Core : choisir la bonne édition

## 2.1 Ubuntu Desktop

Ubuntu Desktop fournit notamment :

- une interface graphique GNOME ;
- NetworkManager ;
- un navigateur ;
- des applications graphiques ;
- les composants nécessaires à une station de travail.

Il convient à :

- un ordinateur personnel ;
- un poste de développement ;
- un poste pédagogique ;
- une machine nécessitant une interface graphique locale.

Une interface graphique n'est pas intrinsèquement « interdite » sur un serveur. En revanche, sur un serveur administré à distance, installer une pile graphique inutile :

- augmente le nombre de paquets ;
- augmente la quantité de mises à jour ;
- augmente la surface d'attaque ;
- consomme davantage de mémoire et d'espace disque ;
- ajoute des composants à diagnostiquer.

La bonne règle est donc :

> [!tip]
> **N'installez que les composants dont vous avez réellement besoin.**

## 2.2 Ubuntu Server

Ubuntu Server fournit une base plus minimale, pensée pour l'administration distante et les services.

Cas classiques :

- serveur web ;
- serveur de bases de données ;
- serveur de messagerie ;
- serveur DNS ;
- hyperviseur ou hôte de conteneurs ;
- serveur applicatif ;
- nœud de calcul ;
- machine virtuelle.

L'installateur serveur repose sur **Subiquity**.

## 2.3 Images cloud

Pour une VM dans un cloud ou une infrastructure automatisée, il est souvent préférable d'utiliser une **Ubuntu Cloud Image** plutôt que d'installer manuellement depuis une ISO.

Ces images sont prévues pour être initialisées avec **cloud-init**.

## 2.4 Ubuntu Core

Ubuntu Core est une édition transactionnelle basée sur les snaps, principalement pensée pour les appliances et l'IoT. Ce n'est pas une variante d'Ubuntu Server installée de la même manière.

---

# 3. Choisir la version

## 3.1 Convention de version

Ubuntu utilise le format :

```text
AA.MM
```

Exemples :

```text
24.04 → avril 2024
26.04 → avril 2026
26.10 → octobre 2026
```

Les versions `.04` des années paires sont normalement des **LTS**.

## 3.2 LTS ou version intermédiaire ?

| Type | Cadence | Support standard | Usage conseillé |
|---|---:|---:|---|
| LTS | 2 ans | 5 ans | production, serveurs, postes durables |
| intermédiaire | 6 mois | 9 mois | test, matériel très récent, besoin de nouveautés |

Pour un serveur de production :

```text
LTS > version intermédiaire
```

sauf besoin technique identifié.

## 3.3 Ubuntu Pro

Ubuntu Pro ajoute notamment :

- une couverture de sécurité étendue ;
- ESM ;
- Livepatch ;
- des options de conformité et de durcissement.

Il ne faut toutefois pas confondre :

```text
LTS                = cycle de maintenance standard
Ubuntu Pro         = services de sécurité/support supplémentaires
unattended-upgrades = mécanisme d'installation automatique de mises à jour
Livepatch          = correctifs de noyau applicables sans redémarrage dans certains cas
```

---

# 4. Architecture matérielle

## 4.1 amd64

C'est l'architecture habituelle des PC et serveurs Intel/AMD 64 bits.

Malgré son nom, `amd64` fonctionne aussi sur les processeurs Intel x86-64.

## 4.2 arm64

`arm64` correspond à AArch64.

On la rencontre notamment sur :

- serveurs ARM ;
- cartes de développement ;
- certaines machines embarquées ;
- certaines plateformes cloud.

## 4.3 Vérifier l'architecture sur un Linux existant

```bash
uname -m
```

Exemples :

```text
x86_64  → amd64
aarch64 → arm64
```

Pour l'architecture des paquets Debian :

```bash
dpkg --print-architecture
```

---

# 5. Préparer l'installation

Avant toute modification de disque :

- sauvegarder les données ;
- vérifier que la sauvegarde est lisible ;
- repérer le bon disque cible ;
- noter la configuration réseau si elle est statique ;
- conserver les clés de chiffrement ou mots de passe nécessaires ;
- vérifier si BitLocker est utilisé en cas de dual boot Windows ;
- vérifier le mode UEFI ;
- télécharger l'ISO depuis une source officielle.

> [!danger]
> Une erreur de sélection de disque pendant le partitionnement peut détruire immédiatement les données d'un autre disque.

## 5.1 Inventorier une machine Linux existante

Avant réinstallation :

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,MODEL
```

```bash
findmnt
```

```bash
ip -br address
```

```bash
ip route
```

```bash
sudo efibootmgr -v
```

```bash
lspci -nn
```

```bash
sudo dmidecode -t system
```

## 5.2 Sauvegarder la liste des paquets

Sur un Ubuntu existant :

```bash
apt-mark showmanual > packages-manual.txt
```

Cette liste n'est pas une sauvegarde complète du système, mais elle aide à reconstruire un poste.

---

# 6. Télécharger et vérifier l'image ISO

Télécharger les images depuis les sites Ubuntu officiels.

Serveur :

<https://ubuntu.com/download/server>

Desktop :

<https://ubuntu.com/download/desktop>

## 6.1 Pourquoi vérifier une ISO ?

Deux propriétés sont distinctes :

**Intégrité** : le fichier téléchargé est-il identique au fichier publié ?

**Authenticité** : les sommes de contrôle proviennent-elles réellement du projet Ubuntu ?

Une simple somme SHA-256 vérifie l'intégrité si l'on fait déjà confiance à la somme publiée. Une signature OpenPGP permet en plus de vérifier l'origine du fichier contenant les sommes.

## 6.2 SHA-256

Exemple :

```bash
sha256sum ubuntu-26.04.1-live-server-amd64.iso
```

Comparer le résultat au fichier officiel `SHA256SUMS` correspondant à la version téléchargée.

Si l'on dispose du fichier `SHA256SUMS` :

```bash
sha256sum -c SHA256SUMS 2>&1 | grep 'OK$'
```

Le tutoriel officiel de vérification est disponible ici :

<https://ubuntu.com/tutorials/how-to-verify-ubuntu>

> [!important]
> Ne recopiez pas une empreinte trouvée sur un forum ou un site tiers pour « vérifier » une image téléchargée depuis ce même site tiers.

---

# 7. Créer la clé USB d'installation

Copier le fichier ISO dans le système de fichiers de la clé **ne suffit pas**. Il faut écrire l'image comme média amorçable.

## 7.1 Depuis Ubuntu

L'outil graphique **Créateur de disque de démarrage** peut être utilisé.

En ligne de commande, `dd` fonctionne aussi, mais exige une extrême prudence :

```bash
lsblk
```

Repérer la clé, par exemple `/dev/sdX`, puis :

```bash
sudo dd if=ubuntu.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

Puis :

```bash
sync
```

> [!danger]
> La destination doit être le **disque** de la clé (`/dev/sdX`), pas une partition (`/dev/sdX1`). Une erreur dans `of=` peut écraser un disque système.

## 7.2 Depuis Windows ou macOS

Utiliser un outil d'écriture d'image compatible, par exemple celui indiqué dans la documentation Ubuntu.

Documentation actuelle :

<https://ubuntu.com/desktop/docs/en/latest/how-to/create-a-bootable-usb-stick/>

---

# 8. BIOS historique et UEFI moderne

Le terme « BIOS » est encore souvent utilisé de manière générique, mais la majorité des machines modernes utilisent **UEFI**.

## 8.1 UEFI

UEFI remplace progressivement le BIOS historique et sait notamment :

- démarrer depuis une partition système EFI ;
- gérer des entrées de démarrage ;
- utiliser Secure Boot ;
- fonctionner naturellement avec GPT.

## 8.2 Partition EFI

Sur une installation UEFI, on trouve normalement une partition **ESP** :

```text
EFI System Partition
FAT32
montée sur /boot/efi
```

Elle contient les chargeurs EFI.

## 8.3 Vérifier si Linux a démarré en UEFI

```bash
if [ -d /sys/firmware/efi ]; then
    echo UEFI
else
    echo BIOS/Legacy
fi
```

## 8.4 Menu de démarrage

Selon le constructeur, le menu de boot peut être accessible avec :

```text
F12
F10
F9
Esc
```

Il est généralement préférable d'utiliser le **Boot Menu** ponctuel plutôt que de modifier définitivement l'ordre de démarrage.

---

# 9. Secure Boot

Secure Boot permet au firmware UEFI de vérifier les composants de la chaîne de démarrage avant de leur transférer l'exécution.

Ubuntu prend en charge Secure Boot sur les configurations compatibles.

Il n'est donc généralement **pas nécessaire de désactiver Secure Boot** pour installer Ubuntu.

Des modules noyau tiers peuvent toutefois nécessiter une signature ou un mécanisme d'enrôlement de clé, par exemple avec MOK.

Vérifier l'état :

```bash
mokutil --sb-state
```

Résultat possible :

```text
SecureBoot enabled
```

> [!warning]
> Désactiver Secure Boot uniquement pour contourner un problème sans comprendre sa cause réduit une protection utile de la chaîne de démarrage.

---

# 10. GPT et MBR

## 10.1 GPT

Pour une machine UEFI moderne, préférer **GPT**.

GPT :

- est moderne ;
- ne possède pas la limite historique de quatre partitions primaires de MBR ;
- fonctionne bien avec de grands disques ;
- est le choix naturel avec UEFI.

## 10.2 MBR / msdos

MBR reste utile pour certains matériels ou scénarios anciens.

Pour une nouvelle installation moderne :

```text
UEFI + GPT
```

constitue le choix par défaut raisonnable.

---

# 11. Comprendre les noms de disques

Les noms dépendent de la technologie.

SATA/SCSI/USB :

```text
/dev/sda
/dev/sdb
```

NVMe :

```text
/dev/nvme0n1
/dev/nvme1n1
```

Partitions NVMe :

```text
/dev/nvme0n1p1
/dev/nvme0n1p2
```

Afficher clairement les disques :

```bash
lsblk -d -o NAME,SIZE,MODEL,SERIAL,TRAN
```

Puis les partitions :

```bash
lsblk -f
```

---

# 12. Partitionnement : principes

Le partitionnement n'est pas un concours visant à créer le plus de partitions possible.

Chaque découpage doit répondre à un objectif :

- isolation ;
- quotas ou capacité ;
- chiffrement ;
- snapshots ;
- restauration ;
- séparation d'un workload ;
- contraintes de sécurité.

## 12.1 Installation simple d'un poste

Pour la majorité des postes :

```text
ESP
/
swapfile éventuel
```

peut suffire.

Une partition `/home` séparée est possible, mais elle ne remplace jamais une sauvegarde.

## 12.2 Serveur

Sur un serveur, séparer certains volumes peut être pertinent lorsque l'on veut empêcher un service de saturer la racine.

Exemples :

```text
/var
/var/log
/var/lib
/var/lib/docker
/var/lib/postgresql
/srv
/tmp
```

Mais la séparation doit être conçue selon l'usage du serveur.

> [!note]
> Contrairement à une ancienne affirmation de ce cours, **Ubuntu peut parfaitement utiliser un `/var` séparé**. Ce montage est classique sur des serveurs. Il faut simplement que la configuration de stockage et le démarrage soient cohérents.

---

# 13. LVM

LVM signifie **Logical Volume Manager**.

Il ajoute une couche de gestion entre les périphériques physiques et les systèmes de fichiers.

```text
disque / partition
      ↓
PV = Physical Volume
      ↓
VG = Volume Group
      ↓
LV = Logical Volume
      ↓
système de fichiers
```

Exemple :

```text
/dev/nvme0n1p3
        ↓
       PV
        ↓
     vgubuntu
       /   \
      /     \
 lv_root   lv_var
```

## 13.1 Avantages

- redimensionnement plus souple ;
- création de volumes logiques ;
- snapshots LVM ;
- agrégation de plusieurs PV dans un VG ;
- organisation plus flexible du stockage serveur.

## 13.2 Commandes principales

```bash
pvs
```

```bash
vgs
```

```bash
lvs
```

Vue détaillée :

```bash
sudo lvs -a -o +devices
```

---

# 14. Chiffrement avec LUKS

Pour chiffrer un volume Linux, Ubuntu utilise couramment **LUKS** via `cryptsetup`.

Pile typique :

```text
disque
  ↓
partition
  ↓
LUKS
  ↓
LVM
  ↓
volumes logiques
  ↓
ext4 / autre système de fichiers
```

Le chiffrement protège surtout les données **au repos** :

- vol du disque ;
- vol de la machine ;
- accès physique au support éteint.

Il ne protège pas contre un attaquant ayant déjà pris le contrôle d'un système démarré et déverrouillé.

## 14.1 Voir les volumes chiffrés

```bash
lsblk -f
```

```bash
sudo cryptsetup status <nom_du_volume>
```

## 14.2 Sauvegarder les informations critiques

Une phrase secrète oubliée ou des métadonnées LUKS irrécupérables peuvent rendre les données définitivement inaccessibles.

La sauvegarde reste obligatoire, même avec du RAID, du LVM et du chiffrement.

---

# 15. Chiffrement adossé au TPM

Les versions modernes de l'installateur peuvent proposer, sur certains matériels et scénarios, un chiffrement utilisant le **TPM**.

Dans la syntaxe `autoinstall` actuelle, Subiquity permet notamment le layout `hybrid` avec chiffrement :

```yaml
autoinstall:
  version: 1
  storage:
    layout:
      name: hybrid
      encrypted: true
```

Ce mécanisme ne doit pas être confondu avec un simple LUKS déverrouillé par une phrase secrète.

> [!warning]
> Le support exact du chiffrement TPM dépend de la version d'Ubuntu, de l'installateur, du firmware, de Secure Boot et du matériel. Vérifier la documentation de la version réellement déployée avant de standardiser une procédure automatisée.

---

# 16. Swap : corriger une règle devenue trompeuse

Le swap peut prendre la forme :

- d'une partition ;
- d'un volume logique ;
- d'un fichier de swap.

Ubuntu Desktop utilise couramment un **swapfile**.

## 16.1 Le swap n'est pas une extension équivalente à la RAM

Le swap est énormément plus lent que la mémoire vive.

Il permet notamment :

- d'éviter certaines pénuries brutales de mémoire ;
- de déplacer des pages mémoire peu utilisées ;
- éventuellement de servir à l'hibernation.

## 16.2 Faut-il swap = RAM ?

Non, pas comme règle générale.

La taille dépend :

- de la quantité de RAM ;
- du workload ;
- du risque d'OOM ;
- de l'hibernation ;
- de la taille des images mémoire qu'il faut pouvoir stocker.

L'hibernation impose des contraintes particulières et ne se résume pas à « créer une partition swap égale à la RAM ».

## 16.3 Voir le swap

```bash
swapon --show
```

```bash
free -h
```

---

# 17. Systèmes de fichiers

Le choix par défaut raisonnable pour une installation Ubuntu classique reste souvent **ext4**.

Afficher les systèmes de fichiers :

```bash
lsblk -f
```

```bash
findmnt
```

Afficher l'espace :

```bash
df -hT
```

> [!tip]
> Ne choisissez pas un système de fichiers exotique uniquement pour une fonctionnalité séduisante. Vérifiez les outils de réparation, de sauvegarde, de monitoring et l'expérience de l'équipe qui administrera la machine.

---

# 18. Installation d'Ubuntu Desktop

L'interface graphique exacte de l'installateur évolue d'une version à l'autre. Il est plus robuste de comprendre les décisions que de mémoriser la position des boutons.

## 18.1 Démarrer la session live

Après démarrage sur la clé USB, Ubuntu Desktop permet généralement de :

- tester Ubuntu ;
- installer Ubuntu.

Le mode live est utile pour :

- tester le Wi-Fi ;
- tester l'affichage ;
- vérifier le clavier ;
- accéder à des fichiers ;
- diagnostiquer un système qui ne démarre plus.

## 18.2 Choisir langue et clavier

Tester les caractères spécifiques avant installation :

```text
@  €  œ  |  \  {  }  [  ]
```

Une disposition erronée peut être particulièrement gênante au moment de saisir une phrase secrète de chiffrement.

## 18.3 Réseau

Une connexion réseau permet notamment :

- de mettre à jour l'installateur ;
- de récupérer certains paquets ;
- d'obtenir des mises à jour pendant l'installation.

Une installation hors ligne reste possible dans de nombreux scénarios.

## 18.4 Type d'installation

Les libellés changent selon les versions, mais les décisions sont généralement :

- installation standard ou plus minimale ;
- pilotes/logiciels tiers ;
- installation à côté d'un autre OS ;
- effacement du disque ;
- partitionnement manuel ;
- chiffrement selon les possibilités de la version.

## 18.5 Résumé avant écriture

Lire attentivement le récapitulatif.

Contrôler notamment :

- le bon disque ;
- les partitions supprimées ;
- les partitions créées ;
- le mode de chiffrement ;
- la partition EFI ;
- la destination du chargeur de démarrage.

Une fois le partitionnement validé, les modifications destructives commencent.

---

# 19. Dual boot avec Windows

Le dual boot nécessite davantage de prudence.

Avant modification :

1. sauvegarder Windows ;
2. sauvegarder la clé de récupération BitLocker ;
3. vérifier l'état de BitLocker ;
4. libérer de l'espace depuis Windows si possible ;
5. conserver le démarrage UEFI ;
6. ne pas supprimer l'ESP Windows par erreur.

## 19.1 Ne pas mélanger les modes de démarrage

Éviter :

```text
Windows installé en UEFI
Ubuntu démarré/installé en mode Legacy
```

Préférer :

```text
Windows UEFI
Ubuntu UEFI
```

## 19.2 Fast Startup

Le démarrage rapide de Windows peut laisser certains systèmes de fichiers NTFS dans un état qui rend leur montage depuis Linux problématique.

Si les deux systèmes doivent accéder à un même volume NTFS, comprendre cette interaction avant de forcer un montage en écriture.

---

# 20. Installation d'Ubuntu Server avec Subiquity

Subiquity est l'installateur moderne utilisé par Ubuntu Server et sert aussi de backend à l'installateur Desktop moderne.

Documentation :

<https://canonical-subiquity.readthedocs-hosted.com/>

## 20.1 Flux général

Une installation serveur interactive comporte typiquement :

1. langue ;
2. clavier ;
3. mise à jour éventuelle de l'installateur ;
4. réseau ;
5. proxy éventuel ;
6. miroir Ubuntu ;
7. stockage ;
8. profil utilisateur ;
9. Ubuntu Pro éventuel ;
10. SSH ;
11. éventuels composants supplémentaires ;
12. installation ;
13. redémarrage.

## 20.2 Profil administrateur

Ubuntu ne demande normalement pas d'utiliser directement le compte root pour l'administration quotidienne.

Le premier utilisateur peut recevoir les droits `sudo`.

Exemple :

```bash
sudo apt update
```

et non :

```bash
su -
```

comme réflexe systématique.

---

# 21. SSH pendant l'installation serveur

Pour un serveur distant, installer OpenSSH pendant l'installation est souvent pertinent.

Subiquity sait :

- installer `openssh-server` ;
- importer ou déclarer des clés publiques ;
- autoriser ou non l'authentification par mot de passe.

Après installation :

```bash
systemctl status ssh
```

Ports en écoute :

```bash
sudo ss -lntp
```

Connexion :

```bash
ssh utilisateur@serveur.example.net
```

## 21.1 Clé publique

Sur le poste client :

```bash
ssh-keygen -t ed25519
```

Puis installer la clé publique :

```bash
ssh-copy-id utilisateur@serveur.example.net
```

Pour un déploiement automatisé, il est préférable d'injecter les clés publiques dès l'installation plutôt que de commencer avec un mot de passe faible temporaire.

---

# 22. Réseau : Netplan

Ubuntu utilise **Netplan** comme couche déclarative de configuration réseau.

Fichiers :

```text
/etc/netplan/*.yaml
```

Netplan génère ensuite la configuration du backend :

- `systemd-networkd`, courant sur Server ;
- `NetworkManager`, courant sur Desktop.

## 22.1 Voir l'état

```bash
ip -br address
```

```bash
ip route
```

```bash
netplan status -a
```

## 22.2 DHCP

Exemple :

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: true
```

Valider prudemment :

```bash
sudo netplan try
```

Puis :

```bash
sudo netplan apply
```

> [!important]
> Sur une machine distante, `netplan try` est préférable pour un changement risqué : une mauvaise configuration peut couper votre propre accès SSH.

## 22.3 IPv4 statique

Exemple :

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      addresses:
        - 192.0.2.10/24
      routes:
        - to: default
          via: 192.0.2.1
      nameservers:
        addresses:
          - 192.0.2.53
          - 2001:db8::53
```

Les réseaux `192.0.2.0/24` et `2001:db8::/32` sont utilisés ici comme valeurs de documentation : ne les recopiez pas comme configuration réelle.

## 22.4 DNS

Ubuntu utilise couramment `systemd-resolved`.

Voir l'état :

```bash
resolvectl status
```

Ne remplacez pas arbitrairement `/etc/resolv.conf` par un fichier statique sans comprendre Netplan et `systemd-resolved`.

---

# 23. Nom d'hôte

Afficher :

```bash
hostnamectl
```

Changer :

```bash
sudo hostnamectl hostname serveur01.example.net
```

Vérifier :

```bash
hostname
hostname -f
```

Pour un serveur exposé sur Internet, le nom d'hôte, le DNS direct et parfois le reverse DNS peuvent être importants pour certains services, notamment la messagerie.

---

# 24. Première connexion : contrôles indispensables

Après le premier démarrage :

```bash
cat /etc/os-release
```

```bash
uname -a
```

```bash
lsblk -f
```

```bash
findmnt
```

```bash
ip -br address
```

```bash
ip route
```

```bash
systemctl --failed
```

```bash
journalctl -p err -b
```

Vérifier aussi le temps :

```bash
timedatectl
```

---

# 25. Mettre le système à jour

Après installation :

```bash
sudo apt update
```

Puis :

```bash
sudo apt upgrade
```

Si nécessaire :

```bash
sudo apt full-upgrade
```

Différence simplifiée :

- `apt update` rafraîchit les index ;
- `apt upgrade` met à niveau les paquets sans supprimer automatiquement des paquets installés ;
- `apt full-upgrade` peut modifier les dépendances plus largement.

Vérifier si un redémarrage est demandé :

```bash
if [ -f /var/run/reboot-required ]; then
    cat /var/run/reboot-required
fi
```

---

# 26. Mises à jour automatiques

Ubuntu installe par défaut le paquet `unattended-upgrades` dans les configurations standard concernées.

Vérifier :

```bash
dpkg -l unattended-upgrades
```

Configuration principale :

```text
/etc/apt/apt.conf.d/20auto-upgrades
/etc/apt/apt.conf.d/50unattended-upgrades
```

Logs :

```text
/var/log/unattended-upgrades/
```

Vérifier les timers APT :

```bash
systemctl list-timers 'apt-*'
```

> [!important]
> Automatiser les correctifs de sécurité ne dispense pas de superviser les échecs, les redémarrages requis et les régressions applicatives.

---

# 27. Paquets : `apt`, snaps et dépôts

Ubuntu utilise principalement des paquets Debian `.deb` via APT, et peut également utiliser des snaps.

## 27.1 Rechercher

```bash
apt search nginx
```

## 27.2 Installer

```bash
sudo apt install nginx
```

## 27.3 Supprimer

```bash
sudo apt remove nginx
```

## 27.4 Purger la configuration du paquet

```bash
sudo apt purge nginx
```

## 27.5 Nettoyer des dépendances devenues inutiles

```bash
sudo apt autoremove
```

Ne lancez pas `autoremove` sans lire la liste des paquets proposés à la suppression.

---

# 28. Pilotes et microcodes

Sur Desktop, l'outil graphique de pilotes additionnels peut proposer certains pilotes propriétaires.

En ligne de commande :

```bash
ubuntu-drivers devices
```

Pour examiner le matériel :

```bash
lspci -k
```

Le microcode CPU peut être livré sous forme de paquets Ubuntu et mis à jour indépendamment du firmware UEFI.

---

# 29. Firmware

Sur de nombreux PC, `fwupd` permet de distribuer des mises à jour de firmware via LVFS lorsque le constructeur les publie.

Voir les périphériques :

```bash
fwupdmgr get-devices
```

Rafraîchir les métadonnées :

```bash
sudo fwupdmgr refresh
```

Voir les mises à jour :

```bash
fwupdmgr get-updates
```

> [!warning]
> Une mise à jour de firmware est une opération différente d'une mise à jour de paquet Linux. Elle peut imposer un redémarrage et doit être planifiée avec prudence sur un serveur.

---

# 30. Pare-feu

Ubuntu fournit **UFW** comme interface simple de configuration du pare-feu.

État :

```bash
sudo ufw status verbose
```

Avant d'activer UFW sur un serveur accessible uniquement en SSH :

```bash
sudo ufw allow OpenSSH
```

Puis :

```bash
sudo ufw enable
```

Vérifier :

```bash
sudo ufw status numbered
```

> [!danger]
> Sur un serveur distant, une erreur de règle de pare-feu peut couper votre accès d'administration.

---

# 31. Synchronisation de l'heure

Une heure correcte est essentielle pour :

- TLS ;
- journaux ;
- Kerberos ;
- signatures ;
- certificats ;
- corrélation d'événements ;
- bases de données et applications distribuées.

Vérifier :

```bash
timedatectl
```

Sur Ubuntu Server moderne, **chrony** est couramment utilisé par défaut pour la synchronisation réseau.

```bash
systemctl status chrony
```

```bash
chronyc tracking
```

---

# 32. Journaux et diagnostic

Le premier outil de diagnostic d'un système moderne utilisant systemd est souvent `journalctl`.

Erreurs du boot courant :

```bash
journalctl -p err -b
```

Noyau :

```bash
journalctl -k -b
```

Un service :

```bash
journalctl -u ssh -b
```

Suivre en direct :

```bash
journalctl -fu ssh
```

Services en échec :

```bash
systemctl --failed
```

---

# 33. Vérifier le stockage après installation

Vue synthétique :

```bash
lsblk -o NAME,SIZE,FSTYPE,FSVER,MOUNTPOINTS,UUID
```

Montages :

```bash
findmnt
```

Espace :

```bash
df -hT
```

Inodes :

```bash
df -ih
```

LVM :

```bash
sudo pvs
sudo vgs
sudo lvs
```

RAID logiciel :

```bash
cat /proc/mdstat
```

SMART/NVMe, selon le matériel :

```bash
sudo smartctl -a /dev/sda
```

ou :

```bash
sudo nvme smart-log /dev/nvme0
```

---

# 34. `/etc/fstab`

Les montages persistants sont généralement déclarés dans :

```text
/etc/fstab
```

Préférer les UUID aux noms `/dev/sdX` susceptibles de changer :

```bash
blkid
```

Exemple :

```text
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /srv ext4 defaults 0 2
```

Après modification :

```bash
sudo systemctl daemon-reload
sudo mount -a
```

Puis vérifier :

```bash
findmnt
```

> [!danger]
> Une erreur dans `/etc/fstab` peut perturber le démarrage. Tester `mount -a` avant de redémarrer.

---

# 35. Autoinstall : remplacer les anciens preseeds

Les anciens installateurs Debian/Ubuntu utilisaient largement des fichiers **preseed**.

Avec Subiquity, l'automatisation moderne utilise **autoinstall**, en YAML.

```text
preseed / d-i → ancien mécanisme
Subiquity autoinstall → mécanisme moderne Ubuntu Server et Desktop pris en charge
```

Documentation :

<https://canonical-subiquity.readthedocs-hosted.com/en/latest/intro-to-autoinstall.html>

---

# 36. Structure minimale d'un fichier autoinstall

Exemple :

```yaml
#cloud-config
autoinstall:
  version: 1
  locale: fr_FR.UTF-8
  keyboard:
    layout: fr
  identity:
    hostname: serveur01
    username: admin
    password: "$6$HASH_ICI"
  ssh:
    install-server: true
    allow-pw: false
    authorized-keys:
      - "ssh-ed25519 AAAA... cle-admin"
```

> [!danger]
> Ne stockez jamais un mot de passe en clair dans un dépôt Git. Le champ `password` attend un hash compatible avec la configuration concernée. Les secrets d'installation doivent être gérés comme des secrets.

## 36.1 Générer un hash de mot de passe

Avec `openssl` :

```bash
openssl passwd -6
```

Le résultat ressemble à :

```text
$6$...
```

Pour une infrastructure automatisée, une authentification SSH par clé et une stratégie de gestion de secrets sont préférables à un mot de passe partagé.

---

# 37. Autoinstall : stockage

## 37.1 Layout direct

```yaml
autoinstall:
  version: 1
  storage:
    layout:
      name: direct
```

## 37.2 LVM

Le layout LVM est pris en charge directement :

```yaml
autoinstall:
  version: 1
  storage:
    layout:
      name: lvm
```

## 37.3 LVM chiffré par phrase secrète

```yaml
autoinstall:
  version: 1
  storage:
    layout:
      name: lvm
      password: "SECRET_LUKS"
```

> [!danger]
> Cet exemple illustre la syntaxe. Ne placez pas une vraie phrase secrète directement dans un dépôt public ou dans une image ISO diffusée.

## 37.4 Configuration détaillée

Pour les besoins avancés, la section `storage.config` permet de décrire précisément :

- disques ;
- partitions ;
- RAID ;
- LVM ;
- formats ;
- montages.

Avant généralisation, valider l'autoinstall dans une VM jetable.

---

# 38. Autoinstall : réseau

Exemple DHCP :

```yaml
#cloud-config
autoinstall:
  version: 1
  network:
    version: 2
    ethernets:
      enp1s0:
        dhcp4: true
```

Pour une configuration physique hétérogène, éviter de supposer que toutes les machines utiliseront exactement `enp1s0`.

Netplan sait faire correspondre une interface avec `match`, par exemple selon son adresse MAC ou d'autres propriétés.

---

# 39. Autoinstall : paquets et commandes tardives

Installer des paquets :

```yaml
autoinstall:
  version: 1
  packages:
    - vim
    - curl
    - git
```

Des `late-commands` peuvent exécuter des opérations vers la fin de l'installation.

Cependant :

> [!tip]
> Plus une installation automatisée contient de logique shell ad hoc, plus elle devient difficile à tester. Utilisez autoinstall pour poser une base reproductible, puis un outil de configuration adapté pour le reste lorsque l'infrastructure grandit.

Exemples d'outils complémentaires :

- cloud-init ;
- Ansible ;
- Salt ;
- Puppet ;
- scripts idempotents maîtrisés.

---

# 40. Fournir la configuration autoinstall

La méthode recommandée passe généralement par **cloud-init**, souvent avec la datasource **NoCloud**.

Le fichier cloud-config contient :

```yaml
#cloud-config
autoinstall:
  version: 1
```

Il peut être distribué via une datasource appropriée.

Subiquity accepte aussi un fichier `autoinstall.yaml` placé sur le média selon les règles documentées.

Documentation :

<https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/providing-autoinstall.html>

> [!note]
> Depuis Ubuntu 24.04, le format avec la clé de premier niveau `autoinstall:` est également accepté sur le média d'installation selon la documentation Subiquity.

---

# 41. Relation entre autoinstall et cloud-init

Ils ne jouent pas le même rôle.

```text
cloud-init dans le système d'installation
            ↓
peut fournir la configuration autoinstall
            ↓
Subiquity installe le système cible
            ↓
cloud-init peut aussi configurer le système cible au premier boot
```

Dans un fichier :

```yaml
#cloud-config

autoinstall:
  version: 1
  user-data:
    # directives cloud-init appliquées au système cible
```

Les directives cloud-init placées au mauvais niveau peuvent s'appliquer au système éphémère de l'installateur au lieu du système cible.

---

# 42. Récupérer la configuration générée par une installation

Après une installation réalisée avec Subiquity, une base de configuration reproductible peut être trouvée dans les logs de l'installateur, notamment :

```text
/var/log/installer/autoinstall-user-data
```

C'est très utile pour :

1. faire une installation interactive correcte ;
2. récupérer la configuration ;
3. la nettoyer ;
4. remplacer les valeurs spécifiques ;
5. la valider ;
6. la tester en VM ;
7. seulement ensuite l'utiliser pour le déploiement automatisé.

---

# 43. Tester une installation automatisée dans une VM

Ne testez pas d'abord une configuration destructrice sur une machine physique importante.

Une VM permet de vérifier :

- le partitionnement ;
- le démarrage ;
- cloud-init ;
- les utilisateurs ;
- SSH ;
- les paquets ;
- les commandes de fin d'installation.

Exemple de workflow :

```text
ISO Ubuntu
  +
autoinstall YAML
  ↓
VM jetable
  ↓
validation
  ↓
recréation complète
  ↓
validation de l'idempotence / reproductibilité
  ↓
matériel ou production
```

---

# 44. Installation sur machine distante

Une installation à distance est plus risquée car une mauvaise configuration réseau ou boot peut rendre la machine inaccessible.

Technologies possibles :

- console IPMI ;
- iDRAC ;
- iLO ;
- KVM over IP ;
- console série ;
- PXE/iPXE ;
- média virtuel BMC ;
- autoinstall.

Avant une réinstallation distante, disposer idéalement d'un accès **hors bande** indépendant du système installé.

> [!danger]
> SSH n'est pas un accès hors bande : si le noyau, le réseau ou le système de fichiers racine ne démarre plus, SSH ne vous sauvera pas.

---

# 45. Installation par PXE : principe

Pour déployer de nombreuses machines :

```text
UEFI
 ↓
DHCP
 ↓
PXE / HTTPBoot / iPXE
 ↓
noyau + initrd installateur
 ↓
Subiquity
 ↓
autoinstall
 ↓
système installé
```

Le détail dépend fortement de l'infrastructure réseau et dépasse le cadre d'une installation manuelle débutante.

L'idée essentielle est de séparer :

- le mécanisme de boot réseau ;
- l'image d'installation ;
- la configuration autoinstall ;
- les secrets ;
- la configuration post-installation.

---

# 46. Installation dans une VM

Une VM Ubuntu n'a généralement pas besoin des mêmes choix qu'une machine physique.

Bonnes pratiques :

- utiliser des périphériques virtio lorsque la plateforme les prend en charge ;
- éviter un partitionnement inutilement complexe ;
- utiliser les images cloud si l'hyperviseur et l'automatisation s'y prêtent ;
- installer l'agent invité adapté à la plateforme ;
- ne pas traiter un snapshot comme une sauvegarde indépendante.

## 46.1 QEMU/KVM

Identifier la virtualisation :

```bash
systemd-detect-virt
```

Exemple :

```text
kvm
```

---

# 47. Après installation : sécurité minimale d'un serveur

Checklist :

- mises à jour installées ;
- heure synchronisée ;
- utilisateur nominatif ;
- accès root direct évité ;
- SSH par clé ;
- authentification par mot de passe désactivée lorsque le contexte le permet ;
- pare-feu ;
- seuls les ports nécessaires sont ouverts ;
- sauvegardes configurées ;
- supervision ;
- logs collectés ;
- stratégie de redémarrage pour les mises à jour ;
- secrets hors des fichiers publics ;
- documentation du système.

Lister les ports :

```bash
sudo ss -lntup
```

Lister les services activés :

```bash
systemctl list-unit-files --type=service --state=enabled
```

Voir les processus :

```bash
ps auxf
```

---

# 48. Sauvegarde : ne pas confondre les mécanismes

Ces technologies ne sont pas équivalentes :

```text
RAID       ≠ sauvegarde
LVM        ≠ sauvegarde
snapshot   ≠ sauvegarde indépendante
chiffrement ≠ sauvegarde
réplication ≠ forcément sauvegarde
```

Une vraie stratégie de sauvegarde doit répondre à :

- quoi sauvegarder ?
- où ?
- à quelle fréquence ?
- combien de versions ?
- quelle copie hors ligne/hors site ?
- comment restaurer ?
- quand la restauration a-t-elle été testée pour la dernière fois ?

---

# 49. Montée de version

Une installation LTS n'est pas figée pour toujours.

Avant un `do-release-upgrade` :

1. lire les notes de version ;
2. sauvegarder ;
3. tester la restauration ;
4. mettre la version actuelle complètement à jour ;
5. vérifier les dépôts tiers ;
6. vérifier l'espace disque ;
7. prévoir une console de secours ;
8. tester les applications critiques.

Commande :

```bash
sudo do-release-upgrade
```

> [!important]
> La disponibilité d'une montée de version automatique peut être volontairement différée après la publication d'une nouvelle LTS. Par exemple, après la publication de 26.04.1, Canonical a annoncé que l'offre automatique de migration depuis 24.04 serait décalée de quelques semaines afin d'intégrer certains correctifs de régression.

---

# 50. Dépannage : la machine ne démarre pas sur la clé

Vérifier :

1. l'image a-t-elle réellement été écrite et non copiée ?
2. la clé apparaît-elle dans le menu UEFI ?
3. le média correspond-il à l'architecture ?
4. la somme SHA-256 est-elle correcte ?
5. la clé fonctionne-t-elle sur une autre machine ?
6. un autre port USB fonctionne-t-il ?
7. le firmware est-il à jour ?
8. un réglage de boot USB bloque-t-il le démarrage ?

Éviter de désactiver immédiatement toutes les protections UEFI au hasard.

---

# 51. Dépannage : l'installateur ne voit pas le disque

Causes possibles :

- contrôleur RAID ;
- mode Intel RST/VMD ;
- pilote manquant ;
- périphérique défectueux ;
- configuration firmware ;
- disque NVMe non détecté ;
- disque déjà utilisé par une configuration particulière.

Depuis un shell :

```bash
lsblk
```

```bash
lspci -nnk
```

```bash
dmesg | grep -Ei 'nvme|ata|scsi|raid|vmd'
```

> [!warning]
> Changer un contrôleur de stockage de RAID/RST vers AHCI sur une machine Windows existante peut empêcher Windows de démarrer si la transition n'a pas été préparée. Ne modifiez pas ce réglage à l'aveugle.

---

# 52. Dépannage : pas de réseau

Identifier les interfaces :

```bash
ip -br link
```

```bash
ip -br address
```

Pilote :

```bash
lspci -k
```

Route :

```bash
ip route
```

DNS :

```bash
resolvectl status
```

Tester par couche :

```bash
ping -c 3 192.0.2.1
```

Puis une IP externe réellement joignable dans votre environnement, puis un nom DNS.

Ne concluez pas « Internet est en panne » si seul le DNS est cassé.

---

# 53. Dépannage : le système démarre en emergency mode

Commencer par lire l'erreur :

```bash
journalctl -xb
```

Vérifier les unités en échec :

```bash
systemctl --failed
```

Les causes fréquentes après modification manuelle incluent :

- erreur dans `/etc/fstab` ;
- UUID incorrect ;
- disque absent ;
- système de fichiers endommagé ;
- volume chiffré non déverrouillé ;
- dépendance de montage.

Vérifier :

```bash
blkid
```

```bash
cat /etc/fstab
```

```bash
findmnt --verify
```

---

# 54. Dépannage : GRUB / UEFI

Inspecter les entrées UEFI :

```bash
sudo efibootmgr -v
```

Vérifier la partition EFI :

```bash
findmnt /boot/efi
```

```bash
ls -l /boot/efi/EFI/
```

En cas de réparation, identifier d'abord :

- si le système a été installé en UEFI ;
- quelle est l'ESP ;
- où se trouve la racine ;
- si le disque est chiffré ;
- si LVM est utilisé.

Une commande de « réparation GRUB » copiée au hasard peut aggraver une configuration dual boot.

---

# 55. Dépannage : vérifier le boot courant

Temps de démarrage :

```bash
systemd-analyze
```

Unités lentes :

```bash
systemd-analyze blame
```

Chaîne critique :

```bash
systemd-analyze critical-chain
```

Boot précédent :

```bash
journalctl -b -1
```

Boot courant :

```bash
journalctl -b
```

---

# 56. Commandes à connaître après une installation

## Version

```bash
cat /etc/os-release
```

## Noyau

```bash
uname -r
```

## CPU

```bash
lscpu
```

## RAM

```bash
free -h
```

## Disques

```bash
lsblk -f
```

## Espace

```bash
df -hT
```

## Montages

```bash
findmnt
```

## Réseau

```bash
ip -br a
```

## Routes

```bash
ip route
```

## DNS

```bash
resolvectl status
```

## Ports

```bash
sudo ss -lntup
```

## Services en échec

```bash
systemctl --failed
```

## Erreurs du boot

```bash
journalctl -p err -b
```

## Mises à jour

```bash
sudo apt update && apt list --upgradable
```

---

# 57. Une installation de production reproductible

Une installation professionnelle devrait pouvoir être reconstruite sans dépendre de la mémoire de l'administrateur.

Mauvais modèle :

```text
installer manuellement
cliquer un peu partout
modifier des fichiers à la main
oublier ce qui a été fait
```

Meilleur modèle :

```text
image officielle vérifiée
        ↓
autoinstall versionné
        ↓
secrets injectés séparément
        ↓
configuration post-installation automatisée
        ↓
tests
        ↓
supervision
        ↓
sauvegardes testées
```

Ce modèle rend l'infrastructure :

- explicable ;
- testable ;
- reproductible ;
- auditable ;
- plus facile à restaurer.

---

# 58. Travaux pratiques — installation Desktop

## Objectif

Installer une Ubuntu Desktop LTS dans une VM ou sur une machine de laboratoire.

## Étapes

1. télécharger l'image officielle ;
2. vérifier son SHA-256 ;
3. démarrer en UEFI ;
4. tester la session live ;
5. identifier les disques ;
6. installer Ubuntu ;
7. appliquer les mises à jour ;
8. relever les informations système ;
9. vérifier Secure Boot ;
10. produire un compte rendu.

## Commandes à inclure dans le compte rendu

```bash
cat /etc/os-release
uname -r
lsblk -f
findmnt
ip -br a
ip route
mokutil --sb-state
systemctl --failed
```

---

# 59. Travaux pratiques — Ubuntu Server

## Objectif

Installer une Ubuntu Server LTS et l'administrer exclusivement à distance.

## Contraintes

- UEFI ;
- GPT ;
- LVM ;
- SSH ;
- clé Ed25519 ;
- pas de mot de passe SSH une fois la clé validée ;
- pare-feu UFW ;
- mises à jour ;
- réseau documenté ;
- sauvegarde définie.

## Vérifications

```bash
hostnamectl
```

```bash
lsblk -f
```

```bash
sudo pvs && sudo vgs && sudo lvs
```

```bash
ip -br a
```

```bash
sudo ss -lntup
```

```bash
sudo ufw status verbose
```

```bash
systemctl --failed
```

```bash
journalctl -p err -b
```

---

# 60. Travaux pratiques — autoinstall

## Objectif

Transformer une installation interactive en installation reproductible.

1. réaliser une installation serveur de référence ;
2. récupérer :

```text
/var/log/installer/autoinstall-user-data
```

3. supprimer les données spécifiques ;
4. remplacer les secrets ;
5. injecter une clé SSH ;
6. configurer le stockage ;
7. tester dans une VM neuve ;
8. vérifier que SSH fonctionne ;
9. détruire la VM ;
10. recommencer l'installation sans intervention humaine.

Le TP n'est validé que si la machine est **recréable**, pas simplement si « l'installation a marché une fois ».

---

# 61. Checklist finale

Avant de considérer une installation comme terminée :

- [ ] image officielle obtenue ;
- [ ] intégrité de l'image vérifiée ;
- [ ] sauvegarde préalable effectuée ;
- [ ] UEFI utilisé lorsque le matériel le permet ;
- [ ] Secure Boot laissé actif sauf justification documentée ;
- [ ] bon disque sélectionné ;
- [ ] partitionnement documenté ;
- [ ] chiffrement utilisé lorsque le risque le justifie ;
- [ ] phrase secrète / clé de récupération sauvegardée de façon sûre ;
- [ ] réseau fonctionnel ;
- [ ] DNS fonctionnel ;
- [ ] heure synchronisée ;
- [ ] mises à jour installées ;
- [ ] SSH par clé configuré si serveur ;
- [ ] ports en écoute vérifiés ;
- [ ] pare-feu configuré ;
- [ ] `systemctl --failed` vérifié ;
- [ ] erreurs du journal vérifiées ;
- [ ] sauvegardes configurées ;
- [ ] restauration prévue/testable ;
- [ ] configuration reproductible ou documentée.

---

# 62. Erreurs fréquentes à éviter

> [!failure] Copier l'ISO sur la clé
> Une ISO doit être écrite comme image amorçable.

> [!failure] Désactiver Secure Boot par réflexe
> Ubuntu sait normalement fonctionner avec Secure Boot. Chercher d'abord la cause réelle du problème.

> [!failure] Suivre un tutoriel BIOS/MBR ancien sur un PC UEFI
> Comprendre le mode de démarrage avant le partitionnement.

> [!failure] Créer `/home` et croire que l'on a une sauvegarde
> Une partition séparée reste sur le même disque et peut être perdue avec lui.

> [!failure] Appliquer « swap = RAM » partout
> La taille du swap dépend du besoin réel, notamment de l'hibernation.

> [!failure] Désactiver le mot de passe SSH avant d'avoir testé la clé
> Toujours conserver une session ou une console de secours pendant le changement.

> [!failure] Modifier Netplan directement sur un serveur distant sans filet
> Utiliser `netplan try` et conserver un accès hors bande pour les systèmes critiques.

> [!failure] Mettre des secrets dans `autoinstall.yaml` versionné
> Séparer configuration et secrets.

> [!failure] Considérer un snapshot comme une sauvegarde
> Tester une restauration indépendante.

---

# 63. Résumé

Pour une nouvelle installation standard en 2026 :

```text
Ubuntu LTS
  +
image officielle vérifiée
  +
UEFI / GPT
  +
Secure Boot si compatible
  +
partitionnement simple et justifié
  +
LUKS si chiffrement nécessaire
  +
Netplan
  +
mises à jour automatiques supervisées
  +
SSH par clé sur serveur
  +
sauvegardes testées
```

Pour plusieurs machines :

```text
Subiquity + autoinstall + cloud-init
              ↓
configuration post-installation automatisée
              ↓
tests et supervision
```

L'objectif n'est pas simplement d'obtenir une machine qui démarre. Une bonne installation doit être **sûre, compréhensible, maintenable et reproductible**.

---

# 64. Références officielles

Ubuntu — cycle de publication et support :

<https://ubuntu.com/about/release-cycle>

Ubuntu Server 26.04.1 LTS :

<https://ubuntu.com/download/server>

Documentation Ubuntu Server :

<https://ubuntu.com/server/docs/>

Documentation de l'installation / Subiquity :

<https://canonical-subiquity.readthedocs-hosted.com/>

Installation serveur de base :

<https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/basic-server-installation.html>

Configuration du stockage :

<https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/configure-storage.html>

Introduction à autoinstall :

<https://canonical-subiquity.readthedocs-hosted.com/en/latest/intro-to-autoinstall.html>

Référence autoinstall :

<https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html>

Fournir la configuration autoinstall :

<https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/providing-autoinstall.html>

Relation cloud-init / autoinstall :

<https://canonical-subiquity.readthedocs-hosted.com/en/latest/explanation/cloudinit-autoinstall-interaction.html>

Netplan dans Ubuntu Server :

<https://ubuntu.com/server/docs/explanation/networking/about-netplan/>

Mises à jour automatiques :

<https://ubuntu.com/server/docs/how-to/software/automatic-updates/>

Conseils de sécurité Ubuntu Server :

<https://ubuntu.com/server/docs/explanation/security/security_suggestions/>

Vérifier une image Ubuntu :

<https://ubuntu.com/tutorials/how-to-verify-ubuntu>

Créer une clé USB amorçable :

<https://ubuntu.com/desktop/docs/en/latest/how-to/create-a-bootable-usb-stick/>
