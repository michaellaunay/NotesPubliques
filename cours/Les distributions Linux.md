---
schema_version: 1
uid: "01M02EX5BAG2AX7CXR366J8BBE"
titre: "Les distributions Linux"
aliases:
  - "Distributions GNU/Linux"
  - "Choisir une distribution Linux"
  - "Familles de distributions Linux"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - gnu-linux
  - distributions
  - administration-systeme
  - gestion-paquets
resume: "Cours de référence pour comprendre ce qu'est une distribution Linux, ses composants, les grandes familles Debian, Fedora/RHEL, SUSE, Arch, Gentoo, Alpine et NixOS, les modèles de publication et les critères permettant de choisir une distribution adaptée à un poste, un serveur, un conteneur ou un système spécialisé."
niveau: debutant
prerequis:
  - "[[GNULinux]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-04-24
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Les distributions Linux

> [!abstract] Objectif
> Comprendre **ce qui différencie réellement deux distributions Linux**, reconnaître les principales familles, lire un cycle de support, choisir un modèle de mise à jour et sélectionner une distribution cohérente avec un usage concret plutôt qu'à partir d'un classement de popularité.

Voir aussi : [[GNULinux]], [[Historique Linux]], [[Installation Ubuntu]], [[Initialisation système et des services]], [[Sécurité avancée sous Linux]], [[Docker]], [[WSL2]].

> [!important] Idée centrale
> Une distribution Linux n'est pas seulement « Linux avec un bureau différent ». Elle assemble un **noyau**, un espace utilisateur, une bibliothèque C, un gestionnaire de paquets, des dépôts, une politique de sécurité, un installateur, un système d'initialisation, des choix de configuration et surtout une **politique de maintenance**.

# 1. Linux, GNU/Linux et distribution : ne pas confondre

## 1.1 Linux est d'abord un noyau

Linux fournit notamment :

- l'ordonnancement des processus ;
- la mémoire virtuelle ;
- les pilotes ;
- les systèmes de fichiers ;
- la pile réseau ;
- les namespaces ;
- les cgroups ;
- les interfaces système exposées aux programmes.

Un noyau seul ne constitue pas un système utilisable par une personne.

## 1.2 Une distribution assemble un système

Une distribution classique ajoute autour du noyau :

```text
applications
    │
outils utilisateur et bibliothèques
    │
gestionnaire de paquets + dépôts
    │
services + init + politiques de sécurité
    │
libc + espace utilisateur de base
    │
noyau Linux
    │
matériel
```

Les distributions décident ensuite :

- quelles versions empaqueter ;
- quels correctifs rétroporter ;
- combien de temps maintenir une version ;
- quand introduire des ruptures ;
- quels dépôts activer ;
- comment publier les mises à jour de sécurité ;
- quelles architectures supporter ;
- quels composants installer par défaut.

## 1.3 BSD n'est pas une « distribution Linux »

FreeBSD, OpenBSD et NetBSD sont des systèmes de type Unix complets possédant leur propre noyau et leur propre espace utilisateur de base.

On peut donc parler de **distribution de logiciels** au sens générique, mais pas de « distribution Linux » pour un système BSD.

> [!warning] Formulation à éviter
> « Une distribution est un ensemble de logiciels fourni avec un noyau Linux ou BSD » mélange deux familles différentes. Dans ce cours, **distribution Linux** signifie un système construit autour du noyau Linux.

# 2. Ce qui caractérise réellement une distribution

Comparer deux distributions demande davantage que comparer leurs fonds d'écran.

## 2.1 Le gestionnaire de paquets

Le gestionnaire de paquets sait :

- installer ;
- mettre à jour ;
- supprimer ;
- résoudre les dépendances ;
- vérifier des signatures ;
- interroger les métadonnées ;
- gérer des dépôts.

Exemples :

| Famille | Format / technologie | Outil de haut niveau courant |
|---|---|---|
| Debian / Ubuntu | `.deb`, `dpkg` | `apt` |
| Fedora / RHEL | RPM | `dnf` |
| openSUSE | RPM | `zypper` |
| Arch | paquet `pkg.tar.zst` | `pacman` |
| Alpine | `.apk` | `apk` |
| Gentoo | ebuilds / Portage | `emerge` |
| NixOS | store Nix | `nix`, configuration NixOS |

## 2.2 Les dépôts

Un dépôt n'est pas un simple dossier de fichiers.

Il contient généralement :

- les paquets ;
- les métadonnées ;
- les dépendances ;
- les signatures ;
- parfois plusieurs canaux ou branches ;
- des politiques de promotion entre développement, test et stable.

Une distribution est donc également une **chaîne d'intégration continue à très grande échelle**.

## 2.3 La politique de correctifs

Deux distributions peuvent annoncer la même version majeure d'un logiciel tout en fournissant des niveaux de sécurité différents.

Une distribution stable peut conserver un numéro de version ancien et rétroporter un correctif :

```text
version upstream récente
        │
        ├── nouvelles fonctions
        ├── changements de comportement
        └── correctifs

version distribution stable
        │
        └── certains correctifs rétroportés
            sans adopter toutes les nouveautés
```

> [!important]
> Juger la sécurité d'un paquet uniquement à partir de son numéro de version upstream est souvent une erreur sur Debian, Ubuntu LTS ou RHEL.

## 2.4 Le cycle de publication

Trois grands modèles apparaissent souvent.

### Version stable périodique

Exemples : Debian stable, Ubuntu, Fedora, RHEL, openSUSE Leap, Alpine stable, NixOS stable.

On installe une version identifiée puis on la maintient pendant un temps défini.

### Rolling release

Exemples : Arch Linux, openSUSE Tumbleweed, Gentoo.

Le système reçoit continuellement de nouvelles versions de paquets.

### Image ou système atomique

Exemples : Fedora Atomic Desktops, Fedora CoreOS, Ubuntu Core, Flatcar, certaines plateformes Kubernetes.

L'unité de mise à jour tend à devenir une **image système cohérente**, avec rollback plus facile et moins de mutations directes de la base.

# 3. Stable ne signifie pas « vieux » et rolling ne signifie pas « instable »

## 3.1 Deux sens du mot stable

Le mot peut désigner :

1. un logiciel qui plante peu ;
2. une plate-forme dont les interfaces et versions changent peu.

Une distribution d'entreprise cherche surtout le second sens.

## 3.2 Le compromis fondamental

```text
nouveauté ─────────────────────────────►

RHEL/Debian stable     Fedora      Arch/Tumbleweed
    stabilité API      compromis      fraîcheur
```

Cette représentation reste simplifiée : chaque projet a son propre modèle de tests et de maintenance.

# 4. Upstream, downstream et dérivées

## 4.1 Upstream

Un projet upstream est une source située en amont dans la chaîne de développement.

Exemple :

```text
Linux kernel
Python
OpenSSL
GNOME
KDE
PostgreSQL
```

Les distributions prennent les versions upstream, les intègrent, parfois les corrigent et construisent des paquets.

## 4.2 Downstream

Une distribution dérivée réutilise tout ou partie d'une autre distribution.

Exemple simplifié :

```text
Debian
  └── Ubuntu
        ├── Linux Mint (édition Ubuntu)
        └── nombreuses dérivées
```

Mais « dérivée » ne signifie pas « copie sans valeur ». Une dérivée peut changer :

- l'installateur ;
- les dépôts ;
- le bureau ;
- les cycles de support ;
- les correctifs ;
- les outils d'administration ;
- les services commerciaux.

# 5. La famille Debian

## 5.1 Debian

Debian est une distribution communautaire généraliste à forte culture de stabilité et d'architecture multi-plateforme.

En août 2026 :

- la stable est **Debian 13 « trixie »** ;
- Debian 13.6 a été publiée le 11 juillet 2026 ;
- `forky` est la branche testing ;
- `sid` reste la branche unstable.

Le projet annonce pour Debian 13 un cycle d'environ cinq ans : support Debian complet puis LTS.

### Commandes de base

```bash
sudo apt update
sudo apt upgrade
```

Recherche :

```bash
apt search nginx
```

Informations :

```bash
apt show nginx
```

Paquets installés :

```bash
apt list --installed
```

Outil bas niveau :

```bash
dpkg -l
```

### Forces typiques

- documentation abondante ;
- stabilité ;
- grand nombre de paquets ;
- architectures multiples ;
- excellente base serveur ;
- gouvernance communautaire ;
- politiques de paquets très documentées.

### Points d'attention

- versions parfois moins récentes dans stable ;
- besoin de comprendre les branches avant de mélanger des dépôts ;
- éviter de transformer une stable en mélange non maîtrisé stable/testing/unstable.

> [!warning]
> Ajouter un dépôt prévu pour une autre version Debian ou une autre distribution peut rendre la résolution de dépendances incohérente.

# 6. Ubuntu et ses variantes

Ubuntu est dérivée de Debian mais possède :

- ses propres dépôts ;
- son propre calendrier ;
- ses propres images ;
- des choix d'intégration Canonical ;
- un support LTS distinct.

Au 31 août 2026, la LTS courante est **Ubuntu 26.04.1 LTS « Resolute Raccoon »**, publiée le 27 août 2026.

## 6.1 LTS et versions intermédiaires

Les versions LTS sont destinées aux installations qui recherchent un horizon de support long.

Les versions intermédiaires suivent un cycle plus rapide.

> [!tip]
> Pour un serveur qu'on ne souhaite pas réinstaller fréquemment, une LTS est généralement plus cohérente qu'une version intermédiaire.

## 6.2 Ubuntu Desktop et Ubuntu Server

Ce ne sont pas deux distributions radicalement différentes.

Elles partagent les mêmes fondations et dépôts, mais leurs images et ensembles de paquets par défaut diffèrent.

Un serveur peut recevoir une interface graphique ; un poste peut héberger des services serveur.

## 6.3 Les flavours officielles

Ubuntu publie plusieurs variantes de bureau, par exemple :

- Kubuntu ;
- Xubuntu ;
- Lubuntu ;
- Ubuntu Studio ;
- Ubuntu Cinnamon ;
- Ubuntu Budgie ;
- Edubuntu.

Le noyau et la base restent Ubuntu ; l'environnement et certains choix par défaut changent.

# 7. Linux Mint

Linux Mint vise principalement le poste de travail.

La variante principale est construite sur Ubuntu LTS ; LMDE utilise Debian comme base.

Ses points distinctifs comprennent :

- Cinnamon ;
- des outils graphiques intégrés ;
- une expérience desktop conservatrice ;
- une transition généralement douce pour des utilisateurs venant de Windows.

> [!note]
> Il est plus utile de choisir Mint pour son expérience de bureau que de la considérer comme une plate-forme serveur distincte.

# 8. Fedora : une distribution communautaire en amont de RHEL

Fedora est une distribution communautaire à cycle rapide, sponsorisée par Red Hat mais gouvernée par le Fedora Project.

**Fedora Linux 44** est sortie le 28 avril 2026.

Elle sert notamment de terrain d'intégration à des technologies qui pourront ensuite rejoindre l'écosystème Enterprise Linux.

## 8.1 Corriger une idée fréquente

La relation n'est pas :

```text
RHEL → Fedora
```

Le flux est plutôt :

```text
projets upstream
      │
      ▼
Fedora / Rawhide / ELN
      │
      ▼
CentOS Stream
      │
      ▼
RHEL
```

Le schéma réel est plus complexe, mais il montre correctement le sens général du développement.

## 8.2 Gestion de paquets

```bash
sudo dnf upgrade --refresh
```

Installer :

```bash
sudo dnf install podman
```

Rechercher :

```bash
dnf search podman
```

## 8.3 Fedora Workstation

Fedora Workstation vise le poste de développement et le desktop moderne.

Elle adopte rapidement :

- nouveaux noyaux ;
- GNOME récent ;
- toolchains récentes ;
- nouveautés systemd ;
- PipeWire ;
- Wayland ;
- technologies de conteneurs.

## 8.4 Fedora Atomic Desktops

Les Atomic Desktops proposent une base atomique avec variantes telles que :

- Silverblue ;
- Kinoite ;
- Sway Atomic ;
- Budgie Atomic ;
- COSMIC Atomic.

L'approche encourage :

- base système transactionnelle ;
- applications Flatpak ;
- développement isolé dans des conteneurs/toolboxes ;
- rollback du système.

# 9. RHEL : Enterprise Linux

Red Hat Enterprise Linux est une distribution commerciale orientée entreprises.

En août 2026 :

- **RHEL 10.2** est disponible depuis le 19 mai 2026 ;
- RHEL 9.8 est également maintenue dans la branche 9.

RHEL privilégie :

- cycles de vie longs ;
- certification matérielle et logicielle ;
- compatibilité ;
- support éditeur ;
- correctifs rétroportés ;
- SELinux ;
- outils d'administration et de conformité.

> [!important]
> Dans un environnement d'entreprise, la valeur de RHEL ne se résume pas au contenu des RPM. Le contrat de support, les certifications et les engagements de cycle de vie font partie du produit.

# 10. CentOS : comprendre le changement de modèle

## 10.1 CentOS Linux historique

Historiquement, CentOS Linux reconstruisait les sources de RHEL et produisait une distribution située **après RHEL**.

Ce modèle est terminé.

Il ne faut plus écrire en 2026 :

> « Pour un serveur : CentOS »

sans préciser ce dont on parle.

## 10.2 CentOS Stream

CentOS Stream est une distribution construite par les ingénieurs RHEL et sert de branche majeure à partir de laquelle les versions mineures RHEL sont créées.

**CentOS Stream 10** est la génération actuelle et doit être maintenue approximativement jusqu'en 2030.

Le flux simplifié est :

```text
Fedora
  │
  ▼
CentOS Stream
  │
  ▼
RHEL
```

CentOS Stream permet notamment :

- de voir ce qui arrive dans le prochain RHEL mineur ;
- de contribuer avant l'intégration RHEL ;
- de développer des SIG et des composants Enterprise Linux.

# 11. Rocky Linux et AlmaLinux

Rocky Linux et AlmaLinux fournissent des systèmes Enterprise Linux communautaires compatibles avec l'écosystème RHEL.

En août 2026 :

- Rocky Linux 10.2 est la version courante de la branche 10 ;
- AlmaLinux OS 10.2 est également disponible.

## 11.1 Pourquoi les choisir ?

Cas typiques :

- infrastructure cherchant une compatibilité Enterprise Linux ;
- logiciels certifiés ou conçus pour l'écosystème RHEL ;
- besoin d'une base communautaire ;
- migration depuis l'ancien CentOS Linux.

## 11.2 Ne pas les confondre avec CentOS Stream

```text
CentOS Stream : en amont immédiat de RHEL mineur
Rocky/Alma    : distributions Enterprise Linux compatibles
```

Leurs objectifs de projet et politiques de construction ne sont pas identiques.

# 12. La famille SUSE

## 12.1 openSUSE Tumbleweed

Tumbleweed est une rolling release.

Elle vise des paquets récents, intégrés et testés en continu.

Elle convient bien :

- aux postes techniques ;
- aux développeurs ;
- aux utilisateurs désirant une rolling release intégrée ;
- aux personnes appréciant l'écosystème YaST/zypper.

## 12.2 openSUSE Leap

Leap suit un modèle de publication stable.

**Leap 16.0** est la génération actuelle en 2026.

## 12.3 SUSE Linux Enterprise

SUSE Linux Enterprise est l'offre entreprise commerciale.

Il ne faut donc pas écrire simplement « SUSE » pour désigner indistinctement openSUSE et le produit entreprise.

## 12.4 Zypper

Mise à jour des métadonnées :

```bash
sudo zypper refresh
```

Mise à jour :

```bash
sudo zypper update
```

Installer :

```bash
sudo zypper install nginx
```

Rechercher :

```bash
zypper search nginx
```

# 13. Arch Linux

Arch Linux est une distribution indépendante, minimaliste au sens de ses choix, orientée vers les utilisateurs qui veulent comprendre et contrôler leur système.

Elle suit une **rolling release**.

## 13.1 Installation minimale

L'installation de base fournit peu de décisions à la place de l'administrateur.

Cela signifie :

- plus de contrôle ;
- davantage de lecture de documentation ;
- plus de responsabilité lors des mises à jour.

## 13.2 Pacman

Synchroniser et mettre à jour tout le système :

```bash
sudo pacman -Syu
```

Installer :

```bash
sudo pacman -S nginx
```

Rechercher :

```bash
pacman -Ss nginx
```

## 13.3 Ne pas faire de mise à jour partielle

Arch est conçue pour garder le système cohérent en mettant à jour l'ensemble.

Éviter les manipulations qui mettent un paquet à jour sans synchroniser les dépendances attendues.

## 13.4 AUR

L'Arch User Repository contient des `PKGBUILD` communautaires.

> [!warning]
> L'AUR n'est pas le dépôt officiel de paquets binaires Arch. Un `PKGBUILD` est du code de construction fourni par la communauté : il faut le lire et comprendre ce qu'il exécute.

# 14. EndeavourOS et Manjaro

Ces distributions rendent l'écosystème Arch plus accessible, mais ne doivent pas être décrites comme « Arch avec un installateur » sans nuance.

Elles prennent leurs propres décisions concernant :

- dépôts ;
- calendrier ;
- outils ;
- configuration ;
- expérience utilisateur.

Un problème sur une dérivée n'est donc pas toujours reproductible sur Arch elle-même.

# 15. Gentoo

Gentoo est une distribution rolling très configurable, construite autour de Portage.

## 15.1 Portage et ebuilds

Portage décrit comment construire et installer les paquets.

Synchronisation :

```bash
sudo emerge --sync
```

Mise à jour du monde :

```bash
sudo emerge --ask --verbose --update --deep --newuse @world
```

## 15.2 USE flags

Les USE flags permettent d'activer ou désactiver certaines fonctionnalités lors de la construction.

Exemple conceptuel :

```text
package A
  ├── support X
  ├── support Y
  └── support Z
```

L'utilisateur peut choisir certaines fonctionnalités selon sa politique locale.

## 15.3 Compilation : éviter le mythe

Gentoo ne doit pas être résumé à :

> « on compile tout donc c'est plus rapide »

L'intérêt principal est plutôt :

- contrôle ;
- personnalisation ;
- compréhension du système ;
- politiques de fonctionnalités ;
- flexibilité des versions et profils.

Les gains de performance de compilation spécifique sont souvent beaucoup moins importants que les différences d'architecture logicielle ou d'algorithme.

## 15.4 Init

Gentoo peut être utilisé avec OpenRC ou systemd selon le profil choisi.

# 16. Alpine Linux

Alpine Linux vise un système simple, petit et orienté sécurité.

**Alpine 3.24** est la branche stable publiée en juin 2026.

## 16.1 Deux différences majeures

Alpine utilise traditionnellement :

- **musl** plutôt que glibc ;
- **BusyBox** pour de nombreux outils de base ;
- **OpenRC** comme init par défaut.

Ces choix réduisent l'empreinte mais ont des conséquences de compatibilité.

## 16.2 APK

```bash
sudo apk update
sudo apk upgrade
```

Installer :

```bash
sudo apk add nginx
```

## 16.3 Alpine et les conteneurs

Alpine est populaire pour les images de conteneurs petites.

Mais « plus petite » ne signifie pas automatiquement « meilleure ».

Avant de choisir Alpine pour une image :

- vérifier la compatibilité musl ;
- mesurer le temps de build ;
- vérifier les wheels Python disponibles ;
- évaluer le besoin de compiler des dépendances natives ;
- comparer avec une image `*-slim` basée sur Debian.

> [!example]
> Une image Python Alpine peut devenir plus complexe qu'une image Debian slim si plusieurs dépendances attendent glibc ou nécessitent une compilation C/Rust.

# 17. NixOS

NixOS adopte une approche déclarative et reproductible.

**NixOS 26.05 « Yarara »** est sortie le 30 mai 2026.

## 17.1 Idée fondamentale

Au lieu de modifier progressivement la machine avec une suite de commandes impératives, on décrit un état voulu.

Exemple simplifié :

```nix
{ config, pkgs, ... }:
{
  services.openssh.enable = true;
  environment.systemPackages = with pkgs; [
    git
    vim
  ];
}
```

Puis le système construit une nouvelle génération.

## 17.2 Avantages

- configuration versionnable ;
- rollback ;
- coexistence de versions ;
- reproductibilité ;
- description déclarative du système.

## 17.3 Coût d'apprentissage

NixOS demande d'apprendre :

- le langage Nix ;
- le store `/nix/store` ;
- les dérivations ;
- les modules ;
- flakes selon le workflow ;
- une logique différente des FHS classiques.

# 18. Slackware

Slackware est l'une des plus anciennes distributions Linux encore actives.

La qualifier de « projet un peu fou » n'apporte aucune information technique.

Elle se distingue historiquement par :

- simplicité de conception ;
- proximité avec les logiciels upstream ;
- configuration assez traditionnelle ;
- peu d'automatisation implicite.

Elle intéresse surtout les utilisateurs qui recherchent cette philosophie spécifique ; ce n'est pas une catégorie technique à elle seule.

# 19. Kali Linux

Kali Linux est destinée aux tests d'intrusion et à la sécurité offensive.

Elle n'est pas une recommandation générale pour apprendre Linux au quotidien.

> [!warning]
> Installer Kali comme poste principal « parce qu'elle contient des outils de hacking » est généralement une mauvaise façon d'apprendre l'administration Linux ou la cybersécurité.

Il est souvent plus pertinent :

- d'utiliser une distribution généraliste ;
- d'exécuter Kali dans une VM ;
- ou d'utiliser un environnement de laboratoire isolé.

# 20. Distributions spécialisées

## 20.1 Fedora CoreOS

Système minimal et automatiquement mis à jour, destiné notamment aux charges conteneurisées.

## 20.2 Flatcar Container Linux

Système orienté conteneurs et mises à jour automatiques.

## 20.3 Talos Linux

Système spécialisé Kubernetes, administré par API, qui remet fortement en cause le modèle du serveur généraliste avec shell interactif.

## 20.4 Ubuntu Core

Système transactionnel orienté appareils, edge et IoT, basé sur snaps.

## 20.5 OpenWrt

Distribution Linux pour routeurs et équipements réseau embarqués.

## 20.6 Raspberry Pi OS

Distribution construite pour Raspberry Pi, dérivée de Debian.

# 21. Android, ChromeOS et Linux embarqué

## 21.1 Android utilise le noyau Linux

Android utilise Linux mais ne constitue pas une distribution GNU/Linux classique.

Son espace utilisateur, son framework applicatif et son modèle de sécurité sont différents.

## 21.2 ChromeOS

ChromeOS utilise également Linux dans son architecture, mais son modèle d'OS, de mises à jour et d'applications est spécifique.

## 21.3 Yocto Project

Yocto n'est pas une distribution grand public à installer telle quelle.

C'est un ensemble d'outils permettant de construire des systèmes Linux embarqués personnalisés.

# 22. Le système d'initialisation

Le choix de la distribution influence souvent l'init.

| Distribution | Init courant |
|---|---|
| Debian | systemd |
| Ubuntu | systemd |
| Fedora | systemd |
| RHEL | systemd |
| openSUSE | systemd |
| Arch | systemd |
| Alpine | OpenRC |
| Gentoo | OpenRC ou systemd |
| NixOS | systemd |

Voir [[Initialisation système et des services]].

# 23. libc : glibc et musl

La bibliothèque C est une composante essentielle de l'espace utilisateur.

La majorité des distributions généralistes utilisent **glibc**.

Alpine utilise principalement **musl**.

Conséquences possibles :

- compatibilité binaire ;
- disponibilité de paquets précompilés ;
- comportement de certaines bibliothèques ;
- taille ;
- résolution DNS ;
- compilation de dépendances natives.

# 24. Sécurité : la distribution définit une politique

La sécurité ne dépend pas seulement du noyau.

Une distribution fournit :

- des équipes sécurité ;
- des avis CVE ;
- des dépôts de correctifs ;
- des politiques de compilation ;
- des mécanismes MAC ;
- des réglages de durcissement ;
- un cycle de support.

## 24.1 SELinux

Fedora et RHEL placent SELinux au centre de leur politique de contrôle d'accès obligatoire.

## 24.2 AppArmor

Ubuntu utilise largement AppArmor.

## 24.3 Ne pas désactiver le mécanisme pour « résoudre » un problème

Mauvaise démarche :

```bash
# désactiver SELinux ou AppArmor sans diagnostic
```

Meilleure démarche :

1. lire le journal ;
2. comprendre la règle refusée ;
3. corriger la politique ou le déploiement ;
4. conserver le contrôle de sécurité si possible.

# 25. Les architectures matérielles

Une distribution peut supporter :

- x86-64 ;
- ARM64/AArch64 ;
- RISC-V ;
- IBM POWER ;
- IBM Z ;
- d'autres architectures selon le projet.

Ne pas supposer qu'une distribution disponible sur PC existe officiellement sur toute architecture.

Exemple notable : Debian 13 supporte officiellement `riscv64` à sa sortie.

# 26. Desktop : choisir d'abord l'écosystème

Pour un poste de travail, considérer :

- compatibilité matérielle ;
- GPU ;
- environnement de bureau ;
- fréquence de mise à jour ;
- support des applications ;
- Flatpak ;
- jeux ;
- pilotes propriétaires si nécessaires ;
- Secure Boot ;
- durée de support.

## 26.1 GNOME

Souvent associé à Fedora Workstation et Ubuntu.

## 26.2 KDE Plasma

Disponible sur de nombreuses distributions, notamment Fedora KDE, Kubuntu, openSUSE et Arch.

## 26.3 Le bureau ne définit pas la distribution

On peut installer KDE sur Debian ou GNOME sur Arch.

Le choix d'une distribution ne devrait donc pas reposer uniquement sur une capture d'écran.

# 27. Serveur : les critères changent

Pour un serveur, on privilégie souvent :

- durée de support ;
- cadence de correctifs ;
- stabilité des interfaces ;
- documentation ;
- automatisation ;
- conformité ;
- support éditeur ;
- disponibilité de paquets ;
- compatibilité avec la plate-forme cloud.

Choix fréquents :

```text
Debian stable
Ubuntu LTS
RHEL
Rocky Linux
AlmaLinux
SUSE Linux Enterprise
```

CentOS Stream peut être un excellent choix lorsque son positionnement en amont de RHEL correspond au besoin, mais il ne remplace pas automatiquement l'ancien CentOS Linux dans tous les contextes.

# 28. Développement logiciel

Un développeur peut chercher :

- toolchains récentes ;
- conteneurs ;
- IDE ;
- pilotes ;
- bibliothèques ;
- documentation ;
- reproductibilité.

Quelques profils possibles :

| Besoin | Choix possibles |
|---|---|
| nouveautés rapides | Fedora, Arch, Tumbleweed |
| base très stable | Debian stable, Ubuntu LTS |
| environnement RHEL | Fedora/CentOS Stream/RHEL/Rocky/Alma selon cible |
| reproductibilité déclarative | NixOS |
| contrôle source très fin | Gentoo |

# 29. Conteneurs : ne pas choisir comme pour une VM

Une image de conteneur ne contient généralement pas :

- un noyau complet ;
- un boot classique ;
- tous les services d'un serveur ;
- un init complet.

Le noyau est fourni par l'hôte.

Une « image Ubuntu » dans Docker signifie principalement :

```text
espace utilisateur Ubuntu
+
paquets Ubuntu
+
libc Ubuntu
```

et non une VM Ubuntu complète.

# 30. Distribution hôte et distribution conteneur peuvent différer

On peut exécuter :

```text
hôte Fedora
  └── conteneur Debian

hôte Ubuntu
  └── conteneur Alpine

hôte RHEL
  └── conteneur UBI
```

à condition que les architectures et appels système nécessaires soient compatibles avec le noyau hôte.

# 31. Universal Base Image (UBI)

Red Hat fournit des images UBI conçues pour construire des conteneurs compatibles avec l'écosystème RHEL.

Elles peuvent être pertinentes lorsque :

- l'application cible OpenShift/RHEL ;
- un environnement d'entreprise exige cette chaîne ;
- on veut une base RPM bien définie.

# 32. WSL

WSL permet d'installer différentes distributions Linux sous Windows.

Le choix reste important :

- Ubuntu pour la documentation et l'intégration courante ;
- Debian pour une base sobre ;
- d'autres distributions selon les besoins.

Voir [[WSL2]].

# 33. Cloud

Dans le cloud, choisir une distribution implique aussi :

- disponibilité des images officielles ;
- agent cloud ;
- support du fournisseur ;
- noyau optimisé ;
- sécurité ;
- intégration IAM ;
- mise à jour automatique ;
- marketplace et licences.

Une image bricolée depuis une ISO locale n'est pas toujours l'équivalent d'une image cloud officielle.

# 34. Immutable, image-based et atomic

Les distributions modernes explorent des modèles où `/usr` ou la base système est moins mutable.

Objectifs :

- mises à jour atomiques ;
- rollback ;
- réduction du drift ;
- reproductibilité ;
- séparation système/applications.

Exemple conceptuel :

```text
base système versionnée
      │
      ├── Flatpak pour applications GUI
      ├── conteneur de développement
      └── données utilisateur séparées
```

# 35. Flatpak, Snap et AppImage ne remplacent pas la distribution

Ces technologies distribuent des applications, mais la distribution reste responsable de nombreux éléments :

- noyau ;
- boot ;
- drivers ;
- systemd/OpenRC ;
- réseau ;
- sécurité ;
- bibliothèques système ;
- mises à jour de base.

# 36. Comment lire un cycle de support

Ne retenir pas seulement la date de sortie.

Chercher :

- fin du support standard ;
- fin du support sécurité ;
- LTS éventuel ;
- support payant éventuel ;
- durée des versions mineures ;
- politique de montée de version.

# 37. Point release

Une point release comme Debian 13.6 ou Ubuntu 26.04.1 ne signifie pas nécessairement une nouvelle génération complète.

Elle peut fournir :

- médias d'installation mis à jour ;
- correctifs cumulés ;
- prise en charge matérielle améliorée ;
- corrections de bugs.

Toujours lire la politique propre au projet.

# 38. Pourquoi le numéro du noyau ne suffit pas

Deux distributions peuvent utiliser :

```text
Linux 6.12
```

mais avec :

- configurations différentes ;
- patches différents ;
- backports différents ;
- modules activés différents ;
- politiques de support différentes.

Comparer seulement `uname -r` est donc insuffisant.

# 39. Identifier sa distribution

Le standard de fait le plus utile est `/etc/os-release`.

```bash
cat /etc/os-release
```

Exemple de champs :

```text
NAME=
ID=
VERSION_ID=
VERSION_CODENAME=
ID_LIKE=
```

Depuis un script :

```bash
. /etc/os-release
printf '%s %s\n' "$ID" "$VERSION_ID"
```

> [!warning]
> `lsb_release` n'est pas installé partout et le LSB n'est plus la base universelle qu'on pourrait imaginer à partir de son nom.

# 40. Détecter la famille de paquets

```bash
command -v apt
command -v dnf
command -v zypper
command -v pacman
command -v apk
command -v emerge
command -v nix
```

Mais éviter d'écrire des scripts complexes uniquement à partir de cette détection si l'outil de déploiement sait déjà cibler la plate-forme.

# 41. `uname` ne donne pas la distribution

```bash
uname -a
```

renseigne principalement sur le noyau et la machine.

Cela ne permet pas de conclure :

```text
"c'est Ubuntu"
```

Pour cela, lire `/etc/os-release`.

# 42. Installer un paquet : comparaison pratique

## Debian / Ubuntu

```bash
sudo apt update
sudo apt install curl
```

## Fedora / RHEL / Rocky / Alma

```bash
sudo dnf install curl
```

## openSUSE

```bash
sudo zypper install curl
```

## Arch

```bash
sudo pacman -S curl
```

## Alpine

```bash
sudo apk add curl
```

## Gentoo

```bash
sudo emerge --ask net-misc/curl
```

# 43. Rechercher à quel paquet appartient un fichier

## Debian

Fichier installé :

```bash
dpkg -S /usr/bin/ssh
```

## RPM

```bash
rpm -qf /usr/bin/ssh
```

## Arch

```bash
pacman -Qo /usr/bin/ssh
```

Ces commandes sont très utiles en diagnostic.

# 44. Inspecter les paquets installés

## Debian

```bash
dpkg -l
```

## RPM

```bash
rpm -qa
```

## Arch

```bash
pacman -Q
```

## Alpine

```bash
apk info
```

# 45. Dépôts tiers : principale source de dette système

Un dépôt tiers peut être légitime, mais il augmente la surface de confiance.

Questions à poser :

- qui signe les paquets ?
- pour quelle version de distribution ?
- quel rythme de maintenance ?
- que se passe-t-il lors d'une montée de version ?
- ce dépôt remplace-t-il des paquets de base ?

# 46. Ne pas exécuter aveuglément `curl | sh`

Une procédure d'installation moderne doit être évaluée.

Avant :

```bash
curl -fsSL https://example.invalid/install.sh | sh
```

préférer au minimum :

```bash
curl -fsSLO https://example.invalid/install.sh
less install.sh
```

puis vérifier :

- source ;
- signature ou checksum ;
- contenu ;
- privilèges demandés.

# 47. Clés de dépôts

Les anciennes pratiques consistant à ajouter une clé globalement sans séparation de confiance sont à éviter.

Sur les distributions modernes, préférer les mécanismes documentés par la distribution pour :

- keyrings dédiés ;
- `signed-by` côté APT ;
- clés associées à un dépôt précis ;
- rotation et révocation.

# 48. Pourquoi « la meilleure distribution » n'existe pas

Une distribution optimise un ensemble de contraintes.

Exemple :

```text
priorité            choix plausible
────────────────────────────────────────
long support        RHEL / Ubuntu LTS / Debian / SLE
nouveauté desktop   Fedora / Tumbleweed / Arch
minimal conteneur   Alpine / Debian slim / UBI minimal
reproductibilité    NixOS
personnalisation    Gentoo / Arch
apprentissage       Debian / Ubuntu / Fedora / Arch selon objectif
```

# 49. Choisir pour apprendre Linux

Pour apprendre l'administration Linux générale, choisir une distribution :

- bien documentée ;
- assez standard ;
- disposant d'une grande communauté ;
- proche de l'environnement cible.

Exemples :

```text
Debian
Ubuntu
Fedora
```

Arch est excellente pour comprendre le système si l'apprenant accepte une installation plus manuelle.

# 50. Choisir pour apprendre l'entreprise

Si l'objectif est l'écosystème Enterprise Linux :

```text
Fedora → nouveautés et développement
CentOS Stream → préfiguration RHEL
RHEL → produit supporté
Rocky/Alma → variantes communautaires Enterprise Linux
```

# 51. Choisir pour un homelab

Un homelab peut utiliser :

- Debian stable ;
- Ubuntu Server LTS ;
- Rocky/Alma ;
- Fedora Server ;
- NixOS ;
- Proxmox VE pour la couche de virtualisation, qui est elle-même basée sur Debian.

Le bon choix dépend souvent davantage de l'objectif d'apprentissage que d'un classement absolu.

# 52. Choisir pour Kubernetes

Pour un cluster Kubernetes, considérer :

- OS généraliste minimal ;
- système spécialisé immuable ;
- politique de mise à jour ;
- support du runtime ;
- conformité ;
- administration distante.

Exemples :

```text
Ubuntu
RHEL / CoreOS
Flatcar
Talos
```

# 53. Choisir pour un appareil embarqué

Les contraintes changent :

- stockage réduit ;
- RAM réduite ;
- démarrage ;
- mise à jour OTA ;
- rollback ;
- lecture seule ;
- support matériel ;
- durée de vie longue.

On peut préférer :

- Buildroot ;
- Yocto ;
- OpenWrt ;
- Ubuntu Core ;
- distribution constructeur.

# 54. Multi-architecture et cross-compilation

Une distribution moderne ne se limite pas au PC x86-64.

Pour l'embarqué ou le cloud ARM, vérifier :

```bash
uname -m
```

Valeurs fréquentes :

```text
x86_64
aarch64
riscv64
```

Le nom de l'architecture dans le gestionnaire de paquets peut différer.

# 55. Les distributions et les pilotes propriétaires

Les projets ont des politiques différentes concernant :

- NVIDIA ;
- codecs ;
- firmware ;
- brevets ;
- logiciels non libres.

Un choix de distribution peut donc avoir un impact concret sur :

- installation GPU ;
- jeux ;
- calcul CUDA ;
- Wi-Fi ;
- multimédia.

# 56. Secure Boot

Secure Boot n'est pas une caractéristique d'une seule distribution.

La qualité de l'expérience dépend notamment :

- signatures du bootloader ;
- noyau signé ;
- modules tiers ;
- mécanisme MOK éventuel ;
- pilotes DKMS.

Ne pas désactiver Secure Boot par réflexe sans comprendre la cause du problème.

# 57. Wayland et X11

Le serveur d'affichage ne définit pas la distribution.

Les distributions récentes ont largement migré vers Wayland pour les bureaux modernes, avec compatibilité XWayland pour de nombreuses applications X11.

Le choix réel dépend :

- du bureau ;
- du GPU ;
- des logiciels ;
- de la distribution ;
- de la session sélectionnée.

# 58. systemd n'est pas « Linux »

systemd est un ensemble de composants d'espace utilisateur.

Linux peut fonctionner avec d'autres init :

- OpenRC ;
- runit ;
- s6 ;
- init traditionnels.

Les distributions choisissent leur politique.

# 59. GNU n'est pas obligatoire pour utiliser Linux

La plupart des distributions de bureau/serveur utilisent beaucoup d'outils GNU.

Mais Alpine utilise largement BusyBox et Android un espace utilisateur très différent.

D'où l'intérêt de distinguer :

```text
Linux              noyau
GNU/Linux          système Linux avec forte composante GNU
Android            noyau Linux + espace utilisateur Android
Alpine              Linux + musl + BusyBox + écosystème Alpine
```

# 60. Tester une distribution sans réinstaller sa machine

## Machine virtuelle

Utiliser :

- KVM/QEMU ;
- GNOME Boxes ;
- virt-manager ;
- VirtualBox selon environnement.

## Live USB

Permet de tester :

- Wi-Fi ;
- GPU ;
- audio ;
- écran ;
- clavier ;
- environnement de bureau.

## Conteneur

Pour explorer l'espace utilisateur :

```bash
docker run --rm -it debian:13 bash
```

ou :

```bash
podman run --rm -it fedora:44 bash
```

> [!warning]
> Un conteneur ne reproduit pas le boot, le noyau et les pilotes d'une installation native.

# 61. Distrobox et Toolbx

Les conteneurs peuvent fournir des environnements utilisateurs d'autres distributions sur le poste courant.

Exemple conceptuel :

```text
Fedora Atomic Desktop
   ├── applications Flatpak
   ├── toolbox Fedora
   └── distrobox Ubuntu
```

Cela réduit parfois le besoin de changer toute la distribution uniquement pour un outil de développement.

# 62. Sauvegarder avant une migration de distribution

Une migration n'est pas qu'une mise à jour de paquets.

Sauvegarder :

- données ;
- `/etc` utile ;
- bases de données ;
- secrets ;
- clés SSH ;
- liste de paquets ;
- configuration des services ;
- conteneurs/volumes ;
- procédures de restauration.

# 63. Ne pas confondre upgrade et réinstallation

Certaines distributions supportent officiellement une montée de version majeure ; d'autres recommandent ou exigent une réinstallation dans certains scénarios.

Toujours lire les notes de version.

Exemple : Rocky Linux ne présente pas la montée directe vers une nouvelle version majeure comme un simple `dnf upgrade` standard.

# 64. Automatisation et distributions

Ansible, Salt, Puppet et autres outils doivent tenir compte de la famille.

Exemple Ansible :

```yaml
- name: Installer nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

Le module générique délègue au gestionnaire de paquets adapté lorsque cela suffit.

# 65. Éviter les scripts « si Ubuntu alors… » partout

Préférer :

- API d'abstraction ;
- rôles par famille ;
- conteneurs ;
- paquetage natif ;
- détection via facts.

Exemple conceptuel :

```text
Debian family
  ├── Debian
  └── Ubuntu

RedHat family
  ├── Fedora
  ├── RHEL
  ├── Rocky
  └── Alma
```

Mais attention : deux membres d'une même famille peuvent diverger fortement.

# 66. Reproductibilité

Une commande :

```bash
sudo apt install nginx
```

ne garantit pas à elle seule que la même version sera obtenue dans deux ans.

Pour la reproductibilité, envisager :

- snapshot de dépôt ;
- version pinning ;
- conteneurs ;
- images ;
- Nix ;
- SBOM ;
- infrastructure as code.

# 67. SBOM et provenance

Dans les environnements modernes, le choix d'une distribution touche aussi la supply chain.

Questions :

- qui construit le paquet ?
- comment est-il signé ?
- quelle provenance ?
- quel SBOM ?
- quelle politique CVE ?
- quels backports ?

# 68. Support commercial ou communautaire

Le choix n'est pas binaire.

Il existe :

- projets communautaires sans contrat ;
- projets communautaires avec prestataires ;
- distributions commerciales ;
- offres de support facultatives ;
- clouds fournissant leur propre support.

# 69. Évaluer une distribution : grille pratique

Attribuer une note ou un commentaire à :

| Critère | Question |
|---|---|
| cycle | rolling ou versions ? |
| support | combien d'années ? |
| paquets | assez récents ? |
| sécurité | avis et correctifs rapides ? |
| architecture | matériel supporté ? |
| automatisation | outils disponibles ? |
| documentation | qualité et fraîcheur ? |
| communauté | active ? |
| entreprise | certification/support requis ? |
| desktop | GPU, bureau, codecs ? |
| conteneurs | images officielles ? |
| conformité | FIPS/CIS/autres besoins ? |

# 70. Arbre de décision simplifié

```text
Besoin principal ?
│
├── serveur long terme
│   ├── support commercial requis → RHEL / SLE / Ubuntu Pro
│   └── communautaire → Debian / Ubuntu LTS / Rocky / Alma
│
├── desktop développeur récent
│   ├── versions périodiques → Fedora
│   └── rolling → Tumbleweed / Arch
│
├── système déclaratif → NixOS
│
├── personnalisation source → Gentoo
│
├── image minimale → Alpine ou image slim selon compatibilité
│
└── Kubernetes spécialisé → CoreOS / Flatcar / Talos selon architecture
```

# 71. Tableau comparatif synthétique

| Distribution | Modèle | Paquets | Init | Positionnement |
|---|---|---|---|---|
| Debian | stable + testing/unstable | APT/dpkg | systemd | généraliste, serveur |
| Ubuntu | périodique + LTS | APT/dpkg | systemd | desktop, serveur, cloud |
| Fedora | ~6 mois | DNF/RPM | systemd | technologies récentes |
| RHEL | entreprise long terme | DNF/RPM | systemd | entreprise/support |
| CentOS Stream | continu dans branche majeure | DNF/RPM | systemd | amont RHEL mineur |
| Rocky | entreprise communautaire | DNF/RPM | systemd | écosystème EL |
| AlmaLinux | entreprise communautaire | DNF/RPM | systemd | écosystème EL |
| openSUSE Leap | périodique | zypper/RPM | systemd | stable SUSE |
| Tumbleweed | rolling | zypper/RPM | systemd | rolling intégrée |
| Arch | rolling | pacman | systemd | contrôle/minimalisme |
| Gentoo | rolling | Portage | OpenRC/systemd | personnalisation |
| Alpine | stable + edge | apk | OpenRC | minimal/musl |
| NixOS | périodique + unstable | Nix | systemd | déclaratif/reproductible |

# 72. État de quelques versions au 31 août 2026

> [!note]
> Cette table sert de repère temporel. Les numéros deviennent forcément obsolètes ; les concepts du cours sont plus importants.

| Projet | Repère courant au 31/08/2026 |
|---|---|
| Debian | 13.6 « trixie » |
| Ubuntu LTS | 26.04.1 LTS « Resolute Raccoon » |
| Fedora | 44 |
| RHEL | 10.2 |
| CentOS Stream | 10 |
| Rocky Linux | 10.2 |
| AlmaLinux | 10.2 |
| openSUSE Leap | 16.0 |
| Arch Linux | rolling release |
| Gentoo | rolling release |
| Alpine | 3.24 |
| NixOS | 26.05 « Yarara » |

# 73. Erreurs fréquentes

## « Ubuntu Server est un autre Linux »

Non : il s'agit d'une variante d'installation Ubuntu avec un ensemble de paquets différent.

## « Debian est uniquement pour les administrateurs avancés »

Non. Debian propose des installateurs et bureaux complets ; son positionnement n'interdit pas un usage débutant.

## « Gentoo est automatiquement plus rapide »

Non. La compilation et les USE flags donnent surtout du contrôle ; les performances doivent être mesurées.

## « Fedora dérive de RHEL »

Non : Fedora est en amont dans la chaîne de développement Enterprise Linux.

## « CentOS = RHEL gratuit »

Cette formulation décrivait imparfaitement l'ancien CentOS Linux. CentOS Stream a aujourd'hui une position différente.

## « Alpine est toujours la meilleure image Docker »

Non. La petite taille peut être compensée par des problèmes de compatibilité ou de compilation.

## « rolling release = instable »

Non. Rolling décrit un modèle de publication, pas un taux de crash.

## « distribution récente = noyau plus sécurisé »

Pas nécessairement. Les distributions stables rétroportent des correctifs.

# 74. Bonnes pratiques avant de choisir

1. définir l'usage ;
2. vérifier la durée de support ;
3. vérifier le matériel ;
4. vérifier les paquets indispensables ;
5. lire les notes de version ;
6. tester en VM/live ;
7. vérifier la politique de sécurité ;
8. vérifier les méthodes d'upgrade ;
9. éviter les dépôts tiers au début ;
10. documenter le choix.

# 75. TP 1 — Identifier une machine

Exécuter :

```bash
cat /etc/os-release
uname -a
uname -m
```

Puis répondre :

- distribution ?
- version ?
- famille ?
- architecture ?
- noyau ?
- gestionnaire de paquets ?

# 76. TP 2 — Comparer deux conteneurs

Lancer :

```bash
docker run --rm debian:13 cat /etc/os-release
```

Puis :

```bash
docker run --rm alpine:3.24 cat /etc/os-release
```

Comparer :

```bash
docker run --rm debian:13 sh -c 'ldd --version | head -n 1'
```

et :

```bash
docker run --rm alpine:3.24 sh -c 'ldd --version 2>&1 | head -n 1'
```

Objectif : constater la différence glibc/musl.

# 77. TP 3 — Comparer les gestionnaires de paquets

Dans plusieurs VMs ou conteneurs, rechercher `curl` avec :

```text
apt search
dnf search
zypper search
pacman -Ss
apk search
```

Comparer :

- syntaxe ;
- métadonnées ;
- taille ;
- dépendances ;
- version.

# 78. TP 4 — Étudier le cycle de vie

Choisir trois distributions et documenter :

- date de sortie ;
- fin de support ;
- fréquence de nouvelles versions ;
- possibilité d'upgrade ;
- support LTS ;
- politique sécurité.

# 79. TP 5 — Choix argumenté

Proposer une distribution pour :

1. un laptop développeur ;
2. un serveur PostgreSQL ;
3. un cluster Kubernetes ;
4. un Raspberry Pi ;
5. un poste familial ;
6. une image Python de production ;
7. un serveur devant exécuter un logiciel certifié RHEL.

Chaque choix doit être justifié par des critères, pas par la popularité.

# 80. TP 6 — Construire une matrice de décision

Créer un tableau :

```text
critère              poids
──────────────────────────
support long           5
paquets récents         3
simplicité admin        4
support ARM             2
support commercial      5
```

Noter plusieurs distributions et discuter les limites de cette méthode.

# 81. Questions de révision

1. Quelle différence entre Linux et une distribution ?
2. Quel rôle joue une libc ?
3. Pourquoi une version de paquet ancienne peut-elle être corrigée ?
4. Qu'est-ce qu'un backport ?
5. Différence stable/rolling ?
6. Quelle est la position de Fedora par rapport à RHEL ?
7. Quelle est la position de CentOS Stream ?
8. Pourquoi Alpine peut-elle poser des problèmes de compatibilité ?
9. Qu'apporte NixOS ?
10. Pourquoi `/etc/os-release` est-il plus pertinent que `uname` pour identifier la distribution ?
11. Quelle différence entre un OS de conteneur et une VM ?
12. Pourquoi la durée de support est-elle cruciale sur serveur ?
13. Qu'est-ce qu'une distribution atomique ?
14. Pourquoi ne faut-il pas comparer seulement les numéros de noyau ?
15. À quoi servent les dépôts ?

# 82. Checklist serveur

Avant d'installer :

- [ ] version encore supportée ;
- [ ] date de fin de support connue ;
- [ ] architecture supportée ;
- [ ] sauvegarde prévue ;
- [ ] dépôts officiels identifiés ;
- [ ] politique de mises à jour définie ;
- [ ] mécanisme de sécurité conservé ;
- [ ] monitoring prévu ;
- [ ] procédure d'upgrade documentée ;
- [ ] automatisation compatible.

# 83. Checklist poste de travail

- [ ] GPU fonctionnel ;
- [ ] Wi-Fi fonctionnel ;
- [ ] audio ;
- [ ] veille ;
- [ ] Secure Boot ;
- [ ] logiciels métiers ;
- [ ] navigateur ;
- [ ] sauvegarde ;
- [ ] chiffrement disque ;
- [ ] durée de support ;
- [ ] politique de mise à jour.

# 84. Résumé

Une distribution Linux est avant tout une **politique d'intégration et de maintenance** autour du noyau Linux.

Pour la choisir, retenir :

```text
noyau
+ userland
+ libc
+ paquets
+ dépôts
+ init
+ sécurité
+ cycle de publication
+ durée de support
+ gouvernance
+ cas d'usage
= distribution
```

Les grandes familles ne sont pas des niveaux de difficulté figés.

Le choix dépend principalement :

- de la fraîcheur souhaitée ;
- de la stabilité des interfaces ;
- du support ;
- de l'écosystème logiciel ;
- des contraintes matérielles ;
- du modèle d'administration.

# 85. Références principales

Sources consultées et repères officiels au 31 août 2026 :

- Debian Releases : <https://www.debian.org/releases/>
- Debian 13 « trixie » : <https://www.debian.org/releases/trixie/>
- Ubuntu announcements : <https://lists.ubuntu.com/archives/ubuntu-announce/>
- Fedora Project : <https://fedoraproject.org/>
- Fedora Magazine, Fedora Linux 44 : <https://fedoramagazine.org/announcing-fedora-linux-44/>
- RHEL release dates : <https://access.redhat.com/articles/red-hat-enterprise-linux-release-dates>
- CentOS Stream documentation : <https://docs.centos.org/centos-stream-docs/>
- CentOS Stream 10 : <https://www.centos.org/centos10/>
- Rocky Linux releases : <https://docs.rockylinux.org/latest/releases/>
- AlmaLinux release notes : <https://wiki.almalinux.org/release-notes/>
- openSUSE documentation : <https://doc.opensuse.org/>
- Arch Linux : <https://archlinux.org/about/>
- ArchWiki : <https://wiki.archlinux.org/>
- Alpine releases : <https://www.alpinelinux.org/releases/>
- NixOS : <https://nixos.org/>
- NixOS 26.05 : <https://nixos.org/blog/announcements/2026/nixos-2605/>
- Gentoo Wiki / Handbook : <https://wiki.gentoo.org/wiki/Handbook:Main_Page>

> [!tip] Maintenir ce cours
> Les **numéros de version** du chapitre 72 doivent être revérifiés lors d'une mise à jour future. Les chapitres conceptuels — familles, gestion de paquets, upstream/downstream, modèles stable/rolling/atomic — vieillissent beaucoup moins vite.
