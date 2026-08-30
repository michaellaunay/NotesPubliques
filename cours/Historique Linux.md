---
schema_version: 1
uid: "01M02EX5B6S3VA583WXXYGA4NH"
titre: "Histoire d'UNIX, GNU et Linux"
aliases:
  - "Historique Linux"
  - "Histoire de Linux"
  - "Histoire de GNU/Linux"
  - "Histoire UNIX et BSD"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - histoire-informatique
  - unix
  - bsd
  - gnu-linux
  - noyau-linux
  - logiciel-libre
  - open-source
resume: "Cours historique et conceptuel sur les racines d'UNIX, BSD, GNU et Linux, la naissance du logiciel libre et de l'open source, l'émergence des distributions, le modèle de développement du noyau et l'évolution de l'écosystème jusqu'en 2026."
niveau: intermediaire
prerequis:
  - "[[GNULinux]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-04-22
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Histoire d'UNIX, GNU et Linux

> [!abstract] Objectifs
> Comprendre **d'où vient Linux**, pourquoi il ressemble à UNIX sans être UNIX, ce que GNU a apporté, comment BSD s'insère dans cette histoire et pourquoi l'écosystème actuel est le résultat de plusieurs lignées techniques, juridiques et communautaires qui se croisent depuis plus d'un demi-siècle.

Voir aussi : [[GNULinux]], [[Les distributions Linux]], [[git]], [[Docker]], [[Les namespaces Linux]], [[proc]], [[Initialisation système et des services]].

> [!important] Idée centrale
> Linux n'est pas apparu comme un système d'exploitation complet sorti de nulle part en 1991. Il hérite d'idées issues d'UNIX, s'appuie sur de nombreux composants du projet GNU, se développe en parallèle de la famille BSD et devient progressivement le noyau d'un immense écosystème allant des serveurs aux téléphones Android, en passant par le cloud, les supercalculateurs et les systèmes embarqués.

---
# 1. Les mots à ne pas confondre

L'histoire devient beaucoup plus simple lorsque nous séparons quelques notions souvent mélangées.

## 1.1 UNIX

**UNIX** désigne historiquement la famille issue du système créé à Bell Labs à partir de 1969.

Aujourd'hui, `UNIX®` est également une marque enregistrée de **The Open Group**. Un système ne peut utiliser officiellement cette marque que s'il satisfait au programme de certification correspondant à la **Single UNIX Specification**.

Ainsi :

```text
origine historique UNIX
        │
        ├── nombreuses lignées historiques
        │
        └── standardisation POSIX / Single UNIX Specification
                                  │
                                  └── certification UNIX®
```

Un système peut donc être très proche d'UNIX sans être un produit certifié UNIX®.

## 1.2 Unix-like

On appelle **Unix-like**, ou « de type Unix », un système qui reprend une grande partie des concepts et interfaces d'UNIX sans nécessairement descendre directement du code historique d'AT&T ni posséder la certification UNIX®.

GNU/Linux est généralement décrit comme un système de type Unix.

## 1.3 Linux

Au sens strict, **Linux est le noyau** initié par Linus Torvalds en 1991.

Le noyau gère notamment :

- les processus ;
- la mémoire ;
- les pilotes ;
- les systèmes de fichiers ;
- la pile réseau ;
- l'ordonnancement ;
- l'isolation ;
- les appels système.

Il ne constitue pas à lui seul l'ensemble de l'environnement utilisateur.

## 1.4 GNU

**GNU** est un projet lancé par Richard Stallman en 1983 afin de construire un système d'exploitation libre compatible avec UNIX.

GNU fournit notamment, historiquement ou aujourd'hui :

- GCC ;
- GNU Binutils ;
- GNU Bash ;
- GNU Coreutils ;
- glibc ;
- GDB ;
- GNU Make ;
- Emacs ;
- de nombreux autres outils.

## 1.5 GNU/Linux

L'expression **GNU/Linux** insiste sur le fait qu'un grand nombre de distributions associent :

```text
noyau Linux
    +
outils et bibliothèques GNU
    +
autres logiciels libres ou propriétaires
    +
gestionnaire de paquets
    +
configuration et intégration de la distribution
```

La Free Software Foundation recommande le terme GNU/Linux pour ces systèmes.

Dans l'usage courant, le mot **Linux** est également utilisé pour désigner l'ensemble de la distribution. Les deux usages existent ; il faut simplement savoir à quel niveau nous parlons.

## 1.6 BSD

**BSD**, pour *Berkeley Software Distribution*, désigne la lignée issue des travaux de l'Université de Californie à Berkeley sur UNIX.

Cette lignée donnera notamment :

- FreeBSD ;
- NetBSD ;
- OpenBSD ;
- DragonFly BSD ;
- une partie importante des fondations de Darwin, donc de macOS et iOS.

## 1.7 POSIX

**POSIX** normalise des interfaces de systèmes de type Unix afin de favoriser la portabilité des programmes.

POSIX ne désigne donc ni un noyau ni une distribution.

Un programme écrit en respectant les interfaces normalisées a davantage de chances d'être portable entre plusieurs systèmes de type Unix.

> [!note]
> En 2026, la référence moderne est la famille de spécifications POSIX issue notamment d'IEEE Std 1003.1 et des Base Specifications de The Open Group. La version Issue 8 correspond à l'édition 2024.

---
# 2. Avant UNIX : le besoin de temps partagé

## 2.1 Les ordinateurs des années 1960

Les premiers grands ordinateurs sont coûteux et leur utilisation est très différente de celle d'une machine moderne.

Il est fréquent de soumettre un travail à une machine puis d'attendre le résultat.

Une idée devient alors essentielle : le **temps partagé** (*time sharing*).

L'objectif est de permettre à plusieurs utilisateurs d'interagir avec une même machine en donnant l'impression que chacun dispose de son propre environnement de travail.

## 2.2 CTSS et Multics

Le MIT développe notamment CTSS, puis participe avec General Electric et Bell Labs au projet **Multics** (*Multiplexed Information and Computing Service*).

Multics est ambitieux :

- multi-utilisateur ;
- interactif ;
- temps partagé ;
- protection mémoire ;
- système de fichiers hiérarchique ;
- sécurité et partage de ressources.

Bell Labs se retire du projet en 1969.

Ce retrait ne signifie pas que Multics est un échec total. De nombreuses idées survivront et influenceront fortement les systèmes ultérieurs.

## 2.3 L'équipe de Bell Labs

Parmi les chercheurs concernés figurent notamment :

- Ken Thompson ;
- Dennis Ritchie ;
- Doug McIlroy ;
- Joe Ossanna.

Ils souhaitent conserver un environnement de programmation interactif et convivial, mais sur un système beaucoup plus simple.

C'est dans ce contexte qu'apparaît UNIX.

---
# 3. 1969-1973 : la naissance d'UNIX

## 3.1 1969 : les premiers travaux

En 1969, Ken Thompson développe sur un **DEC PDP-7** les premières briques du système qui deviendra UNIX.

Il travaille notamment sur :

- un système de fichiers ;
- une notion de processus ;
- un interpréteur de commandes ;
- un éditeur ;
- un assembleur ;
- plusieurs utilitaires.

L'idée importante n'est pas seulement d'obtenir un système fonctionnel : il faut créer un environnement dans lequel il est agréable de programmer.

## 3.2 De Unics à Unix

Le nom est traditionnellement associé à un jeu de mots avec **Multics**. Brian Kernighan est souvent crédité pour le nom `Unics`, ensuite devenu `Unix`.

La graphie `UNIX` s'imposera ensuite largement.

## 3.3 Le PDP-11

À partir de 1970, UNIX est porté sur la famille **PDP-11**.

Ce portage est important car le système cesse progressivement d'être lié au PDP-7 initial.

## 3.4 Le langage C

Dennis Ritchie développe le langage **C** à partir de travaux antérieurs sur B et NB.

Le lien entre C et UNIX devient fondamental.

En **1973**, UNIX est réécrit en grande partie en C.

Cette décision change profondément son avenir.

Un système écrit presque entièrement en assembleur est très dépendant d'une architecture matérielle. Un système écrit principalement en C est beaucoup plus facilement portable.

```text
assembleur spécifique à une machine
              │
              ▼
        portage coûteux

             versus

     grande partie en C
              │
              ▼
        portage facilité
```

> [!important]
> La portabilité d'UNIX constitue l'une des raisons majeures de son influence. Le couple **UNIX + C** devient un modèle extrêmement puissant pour l'enseignement, la recherche et l'industrie.

---
# 4. La philosophie Unix

L'histoire d'UNIX n'est pas seulement une suite de versions. Le système popularise également une manière de concevoir les outils.

## 4.1 Des outils simples et composables

Une idée fréquemment résumée sous le nom de « philosophie Unix » consiste à construire de petits programmes réalisant correctement une tâche et pouvant être combinés.

Exemple :

```bash
cat access.log | grep ' 500 ' | sort | uniq -c
```

Une forme plus idiomatique est :

```bash
grep ' 500 ' access.log | sort | uniq -c
```

Chaque programme joue un rôle précis et le shell permet de construire un traitement plus complexe.

## 4.2 Tout est fichier ?

La formule « tout est fichier » est pratique mais approximative.

L'idée plus utile est qu'UNIX expose de nombreuses ressources au travers d'interfaces homogènes inspirées des fichiers et des descripteurs de fichiers.

Sous Linux, nous retrouvons cette logique avec :

- fichiers ordinaires ;
- terminaux ;
- tubes ;
- sockets ;
- périphériques ;
- pseudo-systèmes de fichiers tels que `/proc` et `/sys`.

## 4.3 Les tubes

Les **pipes** deviennent un élément emblématique d'UNIX.

Exemple :

```bash
ps aux | grep nginx
```

La sortie d'un programme devient l'entrée d'un autre.

Cette composabilité reste au cœur de l'administration GNU/Linux moderne.

---
# 5. Les universités et l'essor de BSD

## 5.1 Pourquoi UNIX arrive dans les universités

AT&T est alors soumis à des contraintes réglementaires liées à son activité de télécommunications.

UNIX est fourni à différentes institutions, notamment universitaires, avec son code source dans le cadre de licences de l'époque.

> [!warning]
> Il est incorrect de présenter cette diffusion comme une « première licence open source ». Le concept moderne d'open source et sa définition apparaîtront bien plus tard. Les conditions juridiques des licences UNIX d'AT&T ne correspondent pas à la définition moderne d'une licence open source.

## 5.2 Berkeley

L'Université de Californie à Berkeley devient l'un des centres majeurs de développement autour d'UNIX.

Bill Joy et d'autres chercheurs y développent la **Berkeley Software Distribution**.

En **1977**, la première BSD est distribuée.

## 5.3 Contributions majeures de BSD

BSD participe à la diffusion ou au développement de nombreuses technologies importantes, notamment :

- la pile TCP/IP ;
- les sockets Berkeley ;
- `vi` ;
- le shell C ;
- de nombreux outils système ;
- des améliorations des systèmes de fichiers et du réseau.

## 5.4 4.2BSD et TCP/IP

**4.2BSD**, publiée en 1983, joue un rôle majeur dans la diffusion de TCP/IP dans les environnements universitaires et de recherche.

Le financement de la DARPA contribue fortement à ces travaux réseau.

Les **Berkeley sockets** deviennent une interface de programmation réseau extrêmement influente.

Nous retrouvons aujourd'hui encore le modèle :

```c
socket()
bind()
listen()
accept()
connect()
send()
recv()
```

## 5.5 Deux familles se dessinent

Dans les années 1980, nous pouvons schématiquement distinguer :

```text
                    UNIX historique
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
        System V                    BSD
          AT&T                 UC Berkeley
```

La réalité est plus complexe : les différentes branches s'influencent mutuellement et des constructeurs intègrent des éléments des deux mondes.

---
# 6. UNIX devient un enjeu industriel

## 6.1 System III et System V

AT&T commercialise différentes générations de son UNIX, notamment :

- System III ;
- System V ;
- System V Release 2 ;
- System V Release 3 ;
- System V Release 4.

**System V Release 4 (SVR4)** rassemble d'ailleurs des technologies venues de plusieurs lignées Unix.

## 6.2 Les UNIX des constructeurs

De nombreux constructeurs développent leur propre variante :

- SunOS puis Solaris chez Sun Microsystems ;
- AIX chez IBM ;
- HP-UX chez Hewlett-Packard ;
- IRIX chez Silicon Graphics ;
- Ultrix chez DEC ;
- Xenix chez Microsoft/SCO ;
- UnixWare ;
- d'autres encore.

Le succès d'UNIX entraîne donc aussi une fragmentation.

## 6.3 Les « Unix wars »

À la fin des années 1980 et au début des années 1990, les acteurs industriels cherchent à imposer ou normaliser différentes variantes.

Pour un développeur, le problème devient évident :

```text
application UNIX
      │
      ├── fonctionne sur système A
      ├── nécessite des adaptations sur B
      └── se comporte différemment sur C
```

Cette fragmentation renforce l'intérêt des normes de portabilité.

---
# 7. POSIX et la standardisation

## 7.1 Pourquoi standardiser ?

L'objectif de POSIX est de définir des interfaces communes à différents systèmes de type Unix.

Cela concerne notamment :

- appels système ;
- bibliothèque C ;
- shell ;
- utilitaires ;
- processus ;
- signaux ;
- fichiers ;
- threads ;
- interfaces réseau selon les ensembles de normes concernés.

## 7.2 Portabilité du code

Un programme qui utilise uniquement des interfaces POSIX standardisées limite sa dépendance à un système particulier.

Exemple conceptuel :

```text
programme C POSIX
      │
      ├── GNU/Linux
      ├── BSD
      ├── macOS
      └── UNIX certifié
```

Cela ne garantit pas qu'un programme fonctionnera sans aucune adaptation, mais fournit une base commune importante.

## 7.3 UNIX est aujourd'hui une certification

The Open Group gère aujourd'hui la marque UNIX®.

Seuls les produits conformes et certifiés selon le programme applicable peuvent utiliser officiellement la marque.

Il faut donc distinguer :

| Terme | Sens |
|---|---|
| UNIX historique | famille issue de Bell Labs |
| UNIX® | marque et certification actuelle |
| Unix-like | système ressemblant conceptuellement à UNIX |
| POSIX | famille de standards d'interfaces |
| Linux | noyau de type Unix, non dérivé du code UNIX historique |

---
# 8. 1983 : naissance du projet GNU

## 8.1 Le contexte

Au début des années 1980, Richard Stallman constate que l'environnement logiciel qu'il connaissait au MIT devient de plus en plus propriétaire.

L'accès au code source et le droit de modifier ou partager un programme ne sont plus considérés comme allant de soi.

## 8.2 L'annonce du projet GNU

Le **27 septembre 1983**, Stallman annonce le projet **GNU**.

GNU signifie :

```text
GNU's Not Unix
```

C'est un **acronyme récursif**.

Le développement du projet commence effectivement en 1984.

## 8.3 Pourquoi être compatible avec UNIX ?

GNU ne cherche pas à reproduire juridiquement UNIX.

Le projet cherche à construire un **système libre compatible avec les concepts et interfaces Unix**.

Le choix est pragmatique :

- UNIX est déjà bien connu ;
- son architecture a fait ses preuves ;
- de nombreux programmeurs connaissent ses interfaces ;
- la compatibilité facilite la migration.

## 8.4 GNU n'est pas seulement une collection d'outils

L'objectif initial est bien un **système d'exploitation complet**.

Un tel système nécessite :

```text
noyau
+ bibliothèques
+ compilateur
+ shell
+ éditeur
+ assembleur
+ linker
+ debugger
+ utilitaires
+ documentation
+ applications
```

De nombreux composants seront terminés bien avant le noyau GNU lui-même.

---
# 9. 1985 : Free Software Foundation et logiciel libre

## 9.1 Création de la FSF

La **Free Software Foundation (FSF)** est fondée en octobre 1985 afin de soutenir le développement de GNU et la défense du logiciel libre.

## 9.2 Libre ne signifie pas gratuit

Le mot anglais `free` est ambigu.

Dans **free software**, il renvoie à la **liberté**, pas nécessairement au prix.

Un logiciel libre peut être vendu.

## 9.3 Les quatre libertés

La définition de la FSF repose sur quatre libertés fondamentales, numérotées de 0 à 3 :

- exécuter le programme pour tous les usages ;
- étudier son fonctionnement et le modifier ;
- redistribuer des copies ;
- redistribuer des versions modifiées.

L'accès au code source est une condition nécessaire aux libertés d'étude et de modification.

## 9.4 Copyleft

Le **copyleft** utilise le droit d'auteur pour imposer que certaines libertés soient conservées lors de la redistribution d'œuvres dérivées.

Il ne signifie pas « absence de copyright ».

Au contraire :

```text
droit d'auteur
      │
      ▼
licence accordant des libertés
      │
      ▼
condition de conserver ces libertés
```

---
# 10. La GNU General Public License

## 10.1 GPLv1

La **GNU GPL version 1** est publiée en **février 1989**.

Elle formalise un mécanisme de copyleft destiné à empêcher qu'un logiciel redistribué perde les libertés accordées par la licence.

## 10.2 GPLv2

La **GPL version 2** est publiée en **juin 1991**.

C'est la licence sous laquelle le noyau Linux est encore distribué aujourd'hui, avec la formulation spécifique du projet Linux concernant la version 2.

## 10.3 GPLv3

La **GPL version 3** est publiée en 2007.

Elle traite notamment de problématiques apparues depuis la GPLv2 :

- brevets ;
- dispositifs empêchant l'exécution de versions modifiées ;
- compatibilités et précisions juridiques internationales.

Le noyau Linux n'a pas migré vers la GPLv3.

## 10.4 Licence libre et licence permissive

Le monde BSD utilise traditionnellement des licences plus permissives.

Schématiquement :

| Famille | Idée générale |
|---|---|
| GPL / copyleft | les versions redistribuées doivent préserver certaines libertés |
| BSD / MIT / Apache | réutilisation plus permissive, y compris dans certains logiciels propriétaires |

Cette différence de philosophie juridique aura une influence considérable sur les écosystèmes.

---
# 11. GNU à la fin des années 1980

À la fin des années 1980, le projet GNU dispose déjà de nombreuses briques majeures :

- GCC ;
- Emacs ;
- GDB ;
- GNU Make ;
- Bash ;
- Binutils ;
- différentes bibliothèques et commandes.

Le principal composant encore manquant pour fournir le système GNU prévu est un noyau suffisamment opérationnel.

## 11.1 GNU Hurd

Le projet GNU développe **GNU Hurd**, construit autour du micro-noyau Mach.

Il s'agit d'une architecture très différente du noyau Linux.

```text
GNU Hurd
   │
   ├── micro-noyau Mach
   └── ensemble de serveurs Hurd
```

Le développement est techniquement ambitieux mais progresse plus lentement que prévu.

> [!important]
> **Mach appartient à l'histoire de GNU Hurd et de plusieurs systèmes issus de cette lignée ; ce n'est pas le noyau de NetBSD.** L'ancienne version de cette note mélangeait ces histoires.

---
# 12. BSD se libère progressivement du code AT&T

## 12.1 Networking Release 1

En 1989, Berkeley publie **Networking Release 1**, souvent appelée `Net/1`, qui rassemble du code réseau distribuable sous une licence BSD.

Cela ne correspond pas à la naissance de NetBSD.

## 12.2 Networking Release 2

En 1991 apparaît **Net/2**, qui contient davantage de code BSD libéré des dépendances propriétaires historiques.

## 12.3 386BSD

William et Lynne Jolitz travaillent au portage de BSD sur l'architecture Intel 386.

**386BSD** devient une étape essentielle entre les travaux de Berkeley et les BSD libres modernes.

## 12.4 Le procès USL contre BSDi

Au début des années 1990, Unix System Laboratories, filiale d'AT&T, engage une procédure judiciaire liée à la présence supposée de code UNIX protégé dans BSD.

Le litige crée une grande incertitude autour de l'écosystème BSD.

Un règlement intervient en 1994 et permet à la lignée **4.4BSD-Lite** d'offrir une base juridiquement assainie.

## 12.5 Naissance de NetBSD

Le projet **NetBSD** est fondé en **1993**.

NetBSD met historiquement l'accent sur :

- la portabilité ;
- la qualité du code ;
- le support de nombreuses architectures.

La version NetBSD 0.8 paraît en avril 1993.

## 12.6 Naissance de FreeBSD

**FreeBSD** apparaît également en 1993, avec un fort intérêt pour les machines compatibles PC x86 puis pour les serveurs et infrastructures réseau.

## 12.7 OpenBSD

**OpenBSD** naît en 1995 à partir de NetBSD.

Le projet met particulièrement en avant :

- audit du code ;
- sécurité ;
- cryptographie ;
- documentation ;
- conception prudente par défaut.

---
# 13. 1991 : Linus Torvalds commence Linux

## 13.1 Le contexte matériel

En 1991, les ordinateurs personnels compatibles x86 deviennent beaucoup plus accessibles.

Linus Torvalds, étudiant à l'Université d'Helsinki, souhaite mieux exploiter son ordinateur équipé d'un processeur Intel 80386.

Il utilise notamment MINIX, système pédagogique créé par Andrew Tanenbaum, mais commence à développer son propre noyau.

## 13.2 L'annonce du 25 août 1991

Le **25 août 1991**, Linus Torvalds publie sur le groupe de discussion `comp.os.minix` un message annonçant son projet de système libre pour les compatibles 386/486.

Le projet est alors encore expérimental.

## 13.3 Linux 0.01

La première version publique du code, Linux 0.01, apparaît en septembre 1991.

À ce stade, Linux est loin d'être un système complet.

## 13.4 Pourquoi le nom Linux ?

Torvalds avait envisagé le nom `Freax`.

Le répertoire FTP utilisé pour publier le projet fut nommé `linux`, contraction évidente autour de Linus et Unix.

Le nom s'est imposé.

## 13.5 Monolithique ne veut pas dire simple

Linux adopte un **noyau monolithique modulaire**.

Cela signifie que de nombreux services fondamentaux fonctionnent dans l'espace noyau :

- ordonnanceur ;
- mémoire ;
- réseau ;
- systèmes de fichiers ;
- pilotes.

Des modules chargeables peuvent cependant être ajoutés dynamiquement.

Il faut distinguer cela d'une architecture micro-noyau telle que Mach.

---
# 14. 1992 : Linux devient logiciel libre

Les toutes premières versions de Linux utilisent une licence propre qui interdit notamment certaines formes de redistribution commerciale.

En **1992**, Linux adopte la **GNU GPL version 2**.

Cette décision est déterminante.

Elle permet une combinaison beaucoup plus naturelle avec l'écosystème GNU et favorise la contribution collaborative.

```text
           outils GNU déjà disponibles
           GCC / Bash / libc / etc.
                     │
                     │
                     ▼
                noyau Linux
                     │
                     ▼
          système de type Unix utilisable
```

La FSF présente cette combinaison comme le système **GNU/Linux**.

---
# 15. Linux et GNU : deux histoires qui se rejoignent

## 15.1 Ce que fournit Linux

Linux fournit principalement le noyau :

```text
matériel
   │
   ▼
Linux
   │
   ├── processus
   ├── mémoire
   ├── pilotes
   ├── réseau
   └── systèmes de fichiers
```

## 15.2 Ce que fournissent GNU et les autres projets

Un système utilisable nécessite également :

```text
shell
compilateur
libc
commandes utilisateur
init
éditeur
serveurs
bibliothèques
interface graphique
applications
```

Beaucoup de ces composants viennent de GNU, mais pas tous.

Une distribution moderne peut par exemple utiliser :

- systemd plutôt qu'un init GNU ;
- musl plutôt que glibc ;
- BusyBox plutôt que GNU Coreutils ;
- LLVM/Clang en complément ou à la place de GCC pour certains usages ;
- Wayland, GNOME, KDE, Firefox, OpenSSH et de nombreux projets indépendants.

## 15.3 Tous les systèmes Linux ne sont donc pas GNU/Linux

Android utilise le noyau Linux, mais son espace utilisateur n'est pas un système GNU classique.

De même, certains systèmes embarqués utilisent Linux avec BusyBox et musl.

Ainsi :

```text
Linux != obligatoirement GNU/Linux
```

---
# 16. 1993-1994 : les premières grandes distributions

Un noyau et une collection d'outils ne suffisent pas pour la plupart des utilisateurs.

Il faut :

- installer le système ;
- choisir les logiciels ;
- gérer les dépendances ;
- appliquer des mises à jour ;
- fournir une configuration cohérente.

C'est le rôle des **distributions**.

## 16.1 SLS

**Softlanding Linux System (SLS)** est l'une des premières distributions populaires au début des années 1990.

Elle influence directement plusieurs projets ultérieurs.

## 16.2 Slackware

**Slackware** apparaît en 1993 sous l'impulsion de Patrick Volkerding.

Elle devient l'une des plus anciennes distributions encore maintenues.

## 16.3 Debian

**Debian** est annoncé par Ian Murdock en 1993.

Le projet se distingue par :

- une organisation communautaire ;
- le Debian Social Contract ;
- les Debian Free Software Guidelines ;
- le gestionnaire de paquets `dpkg` puis APT ;
- un grand nombre d'architectures et de paquets.

## 16.4 Red Hat

Red Hat apparaît dans la première moitié des années 1990 et joue un rôle majeur dans la professionnalisation de Linux en entreprise.

La famille RPM donnera naissance ou influencera de nombreux systèmes, notamment Fedora et Red Hat Enterprise Linux.

## 16.5 SUSE

SUSE devient également un acteur historique majeur de l'écosystème Linux européen et professionnel.

---
# 17. 1994 : Linux 1.0

La version **Linux 1.0** est publiée en mars 1994.

L'archive officielle de kernel.org date `linux-1.0.tar.*` du **13 mars 1994**.

Cette version marque un passage symbolique : Linux n'est plus seulement un petit projet expérimental.

Le noyau continue ensuite à évoluer très rapidement.

## 17.1 Linux 1.x

Les premières séries consolident notamment :

- architecture x86 ;
- systèmes de fichiers ;
- réseau ;
- pilotes ;
- multitâche ;
- support matériel.

## 17.2 Linux 2.0

Linux 2.0 paraît en 1996.

Il apporte notamment un support important du **SMP**, permettant d'utiliser plusieurs processeurs.

La série 2.x durera longtemps.

---
# 18. 1996 : Tux et l'identité du projet

Le manchot **Tux** devient la mascotte de Linux dans les années 1990.

Il contribue à donner au projet une identité visuelle forte et très différente de l'image traditionnelle des UNIX commerciaux.

Cette culture communautaire joue un rôle important dans la diffusion du projet : groupes d'utilisateurs, conférences, HOWTO, listes de diffusion et distributions communautaires structurent progressivement l'écosystème.

---
# 19. 1998 : « open source »

## 19.1 Naissance du terme

En février 1998, après l'annonce de l'ouverture du code de Netscape, une réunion à Palo Alto cherche une expression susceptible de mieux présenter le modèle de développement collaboratif aux entreprises.

Le terme **open source**, proposé par Christine Peterson, est retenu.

## 19.2 Création de l'OSI

L'**Open Source Initiative (OSI)** est fondée en 1998, notamment par Eric Raymond et Bruce Perens.

L'OSI devient la gardienne de l'**Open Source Definition** et du processus d'approbation des licences open source.

## 19.3 Logiciel libre et open source ne sont pas de parfaits synonymes

Les deux mouvements se recouvrent largement sur les logiciels et licences qu'ils considèrent acceptables, mais leur discours n'est pas identique.

La FSF insiste principalement sur les **libertés de l'utilisateur**.

Le mouvement open source met historiquement davantage en avant :

- la méthode de développement ;
- la collaboration ;
- la qualité ;
- l'efficacité ;
- les avantages économiques et techniques.

L'expression **FOSS** (*Free and Open Source Software*) permet parfois d'englober les deux traditions.

---
# 20. Linux devient un système d'entreprise

## 20.1 Le changement d'échelle

À la fin des années 1990 et au début des années 2000, Linux entre massivement dans :

- les centres de données ;
- les serveurs Web ;
- les bases de données ;
- le calcul scientifique ;
- les télécommunications ;
- l'embarqué.

## 20.2 Les grands acteurs industriels

Des entreprises comme IBM, Intel, HP, Red Hat, SUSE et beaucoup d'autres investissent dans l'écosystème.

La participation d'entreprises n'est pas contraire au logiciel libre.

Un modèle courant devient :

```text
code libre
   +
contributions communautaires
   +
contributions industrielles
   +
services / support / matériel / cloud
```

## 20.3 Le mythe « communautaire contre entreprise »

Le noyau Linux moderne est à la fois :

- communautaire ;
- développé publiquement ;
- fortement soutenu par des entreprises ;
- maintenu par une hiérarchie de responsables techniques.

Ces catégories ne s'excluent pas.

---
# 21. 2005 : Git naît des besoins du noyau Linux

## 21.1 BitKeeper

De 2002 à 2005, le développement du noyau utilise BitKeeper, un système de gestion de versions distribué propriétaire mis à disposition du projet sous certaines conditions.

Lorsque cette relation prend fin en 2005, le projet a besoin très rapidement d'un nouvel outil.

## 21.2 Création de Git

Linus Torvalds commence alors **Git**.

Les objectifs initiaux sont adaptés au développement du noyau :

- vitesse ;
- intégrité ;
- développement distribué ;
- nombreuses branches parallèles ;
- très grand nombre de changements.

Git dépassera ensuite très largement le cadre du noyau Linux et deviendra l'un des outils centraux du développement logiciel moderne.

Voir : [[git]].

---
# 22. 2007 : Linux Foundation

En 2007, l'**Open Source Development Labs (OSDL)** fusionne avec la **Free Standards Group** pour former la **Linux Foundation**.

La fondation fournit notamment :

- une structure neutre pour différents projets ;
- des moyens financiers et administratifs ;
- des infrastructures ;
- des événements ;
- des travaux autour des standards et de la chaîne logicielle.

Il ne faut pas confondre :

| Organisation | Rôle historique principal |
|---|---|
| FSF | libertés du logiciel et projet GNU |
| OSI | définition et promotion de l'open source |
| Linux Foundation | soutien à Linux et hébergement d'écosystèmes collaboratifs |
| The Open Group | standards ouverts, POSIX/SUS et marque UNIX® |

---
# 23. Linux conquiert l'embarqué et les téléphones

## 23.1 Pourquoi Linux convient à l'embarqué

Linux possède plusieurs caractéristiques intéressantes :

- code source disponible ;
- architecture portable ;
- grande variété de pilotes ;
- possibilité d'adaptation ;
- communauté importante ;
- choix d'espaces utilisateur légers.

## 23.2 Android

Android utilise un noyau Linux adapté à ses besoins.

Aujourd'hui, l'Android Open Source Project base ses **Android Common Kernels** sur des noyaux Linux LTS en amont auxquels sont ajoutées des modifications spécifiques.

Android démontre une distinction essentielle :

```text
Android
   ├── noyau Linux
   └── espace utilisateur Android
          ≠ environnement GNU/Linux classique
```

Ainsi, des milliards d'appareils utilisent Linux sans présenter à l'utilisateur un bureau ou un shell GNU/Linux traditionnel.

---
# 24. Linux dans la virtualisation et le cloud

## 24.1 KVM

Le support de **KVM** transforme Linux en hyperviseur via les extensions de virtualisation matérielle.

Linux peut alors être à la fois :

- système hôte ;
- noyau d'hyperviseur ;
- système invité.

## 24.2 Namespaces et cgroups

Les **namespaces** isolent différentes vues des ressources :

- PID ;
- réseau ;
- montages ;
- utilisateurs ;
- IPC ;
- UTS ;
- cgroup ;
- temps.

Les **cgroups** permettent de contrôler et mesurer les ressources.

Ces mécanismes deviennent les fondations des conteneurs Linux modernes.

Voir : [[Les namespaces Linux]] et [[Docker]].

## 24.3 Conteneurs

Docker popularise à partir de 2013 une ergonomie de construction et distribution d'images de conteneurs.

Mais Docker n'invente pas l'isolation Linux.

Il assemble et rend accessibles différentes fonctionnalités déjà présentes dans le noyau et l'espace utilisateur.

## 24.4 Kubernetes

Kubernetes pousse ensuite à une très grande échelle l'orchestration des conteneurs.

Linux devient ainsi l'une des principales fondations du cloud moderne.

---
# 25. Comment le noyau Linux est développé

## 25.1 Un projet distribué

Le développement ne consiste pas à envoyer toutes les modifications directement à Linus Torvalds.

Le projet est organisé autour de sous-systèmes et de mainteneurs :

```text
contributeur
    │
    ▼
mainteneur d'un pilote ou composant
    │
    ▼
mainteneur de sous-système
    │
    ▼
arbre d'intégration
    │
    ▼
mainline de Linus Torvalds
```

Le schéma réel varie selon le sous-système mais cette hiérarchie explique la capacité du projet à absorber un très grand volume de changements.

## 25.2 Les patches

Historiquement, les contributions au noyau utilisent fortement :

- Git ;
- les listes de diffusion ;
- les patches envoyés par courrier électronique ;
- la revue par les pairs ;
- les arbres Git des mainteneurs.

## 25.3 La merge window

Après la publication d'une version stable principale, une période d'environ deux semaines est traditionnellement utilisée pour intégrer les changements destinés à la version suivante.

Puis viennent les **release candidates** :

```text
v7.x-rc1
v7.x-rc2
v7.x-rc3
...
```

Une nouvelle version mainline est généralement publiée toutes les **9 à 10 semaines**.

---
# 26. Mainline, stable et longterm

Kernel.org distingue plusieurs catégories importantes.

## 26.1 Mainline

La branche **mainline** est maintenue par Linus Torvalds.

C'est là que sont intégrées les nouvelles fonctionnalités majeures.

## 26.2 Stable

Après publication d'une version mainline, des correctifs importants peuvent être rétroportés vers une branche **stable**.

Exemple :

```text
7.2
 ├── 7.2.1
 ├── 7.2.2
 └── ...
```

## 26.3 Longterm / LTS

Certaines versions sont maintenues plus longtemps.

Elles sont particulièrement importantes pour :

- distributions d'entreprise ;
- Android ;
- systèmes embarqués ;
- infrastructures nécessitant une longue stabilité d'ABI ou de support.

## 26.4 Le noyau de notre distribution n'est pas forcément le dernier noyau kernel.org

Une distribution stable peut utiliser une version LTS plus ancienne tout en appliquant de nombreux correctifs.

Ainsi :

```text
numéro inférieur
     ≠
moins sécurisé ou inutilisable
```

Il faut regarder la politique de maintenance de la distribution.

---
# 27. Comprendre les numéros de version Linux

## 27.1 Les anciennes séries

Pendant longtemps, la numérotation distinguait notamment des séries stables et de développement avec le second nombre pair ou impair.

Cette convention appartient à l'histoire ancienne du projet et ne décrit plus le modèle actuel.

## 27.2 Les grands changements de numéro

Quelques jalons :

| Version | Année | Remarque |
|---|---:|---|
| 1.0 | 1994 | première version 1.0 |
| 2.0 | 1996 | importante maturation, SMP notamment |
| 2.6 | 2003 | série qui durera de nombreuses années |
| 3.0 | 2011 | changement de numérotation |
| 4.0 | 2015 | nouveau numéro majeur |
| 5.0 | 2019 | nouveau numéro majeur |
| 6.0 | 2022 | nouveau numéro majeur |
| 7.0 | 2026 | nouveau numéro majeur |

> [!warning]
> Le passage de 6.x à 7.x ne signifie pas que Linux a été entièrement réécrit. Les changements de numéro majeur servent aussi à conserver une numérotation maniable.

## 27.3 Situation au 31 août 2026

Au moment de la mise à jour de ce cours :

- `7.2` est la branche **mainline** publiée le 16 août 2026 ;
- `7.2.2` est une version **stable** publiée le 28 août 2026 ;
- plusieurs branches longterm 6.x et 5.x restent maintenues.

Nous vérifions toujours l'état actuel avec :

```text
https://kernel.org/
```

---
# 28. Les distributions : un écosystème, pas une seule plate-forme

## 28.1 Une distribution choisit et intègre

Une distribution décide notamment :

- version du noyau ;
- système d'initialisation ;
- libc ;
- gestionnaire de paquets ;
- versions des bibliothèques ;
- politique de sécurité ;
- environnement de bureau ;
- calendrier de publication ;
- durée de support.

## 28.2 Familles majeures

Nous pouvons schématiquement représenter quelques grandes familles :

```text
Debian
 ├── Ubuntu
 │    ├── Linux Mint (selon édition)
 │    └── nombreuses dérivées
 └── autres dérivées

Red Hat / Fedora
 ├── Fedora
 ├── RHEL
 └── dérivées compatibles

SUSE
 ├── openSUSE
 └── SUSE Linux Enterprise

Arch Linux
 └── nombreuses dérivées

Gentoo
Slackware
NixOS
Alpine Linux
...
```

Cette représentation est volontairement simplifiée.

Voir : [[Les distributions Linux]].

---
# 29. Linux, desktop et serveurs

## 29.1 Serveurs

Linux devient extrêmement présent sur les serveurs grâce notamment à :

- stabilité ;
- automatisation ;
- réseau ;
- outils de développement ;
- virtualisation ;
- conteneurs ;
- disponibilité du code ;
- diversité des distributions.

## 29.2 Desktop

Sur le poste de travail, plusieurs environnements se développent :

- GNOME ;
- KDE Plasma ;
- Xfce ;
- Cinnamon ;
- LXQt ;
- autres gestionnaires de fenêtres et shells.

## 29.3 GNOME et KDE

KDE apparaît dans les années 1990 avec Qt.

À l'époque, la licence de Qt suscite des inquiétudes dans une partie de la communauté du logiciel libre.

Le projet **GNOME** est lancé en 1997 afin de créer un environnement de bureau entièrement libre autour de GTK.

Qt adoptera ensuite des licences libres appropriées, et KDE comme GNOME deviendront deux piliers durables du desktop Linux.

---
# 30. X11, Wayland et l'évolution de l'affichage

L'affichage graphique sous les systèmes Unix-like a longtemps reposé principalement sur le **X Window System**, notamment X11.

Avec le temps, les limites architecturales et les besoins modernes conduisent au développement de **Wayland**.

Aujourd'hui, de nombreux bureaux Linux utilisent Wayland comme protocole d'affichage principal tout en conservant souvent une compatibilité X11 via XWayland.

Cette évolution illustre une caractéristique générale de Linux :

```text
compatibilité historique
        +
évolution progressive des briques
```

---
# 31. systemd et l'évolution de l'espace utilisateur

Linux ne se résume pas au noyau.

L'espace utilisateur change profondément avec le temps.

Par exemple, de nombreuses distributions adoptent **systemd** pour :

- lancer les services ;
- gérer les dépendances ;
- journaliser ;
- gérer certaines fonctions réseau et système ;
- superviser les unités.

Voir : [[Initialisation système et des services]].

L'histoire de Linux est donc aussi celle d'un écosystème qui remplace progressivement certaines briques tout en en conservant d'autres.

---
# 32. Linux et les architectures matérielles

Linux commence sur x86, mais devient rapidement portable.

Il fonctionne ou a fonctionné sur de nombreuses architectures :

- x86 ;
- x86-64 ;
- ARM ;
- AArch64 ;
- RISC-V ;
- PowerPC ;
- s390x ;
- MIPS selon les générations ;
- SPARC selon les générations ;
- autres architectures historiques.

Cette portabilité est une conséquence lointaine de l'idée qui avait déjà rendu UNIX puissant : séparer autant que possible le système des détails propres à une machine.

---
# 33. Linux dans les supercalculateurs

Linux est devenu la plate-forme dominante du calcul haute performance.

Les raisons sont multiples :

- code modifiable ;
- optimisation possible pour le matériel ;
- réseau hautes performances ;
- ordonnanceurs spécialisés ;
- compatibilité avec les outils scientifiques ;
- contrôle complet du système.

Cet usage rappelle que Linux peut être aussi bien :

```text
un noyau de routeur embarqué
un système Android
un serveur cloud
un poste de développeur
un supercalculateur
```

La même famille technologique couvre des échelles extrêmement différentes.

---
# 34. eBPF : le noyau devient programmable de manière contrôlée

L'une des évolutions majeures du Linux moderne est **eBPF**.

eBPF permet de charger des programmes vérifiés dans le noyau pour différents usages :

- observation ;
- traçage ;
- réseau ;
- sécurité ;
- performance.

Des outils modernes s'appuient sur cette infrastructure pour observer le système avec une granularité qui aurait autrefois nécessité des modifications plus intrusives du noyau.

Cette évolution montre que l'histoire de Linux n'est pas terminée : son architecture continue de changer tout en préservant une grande compatibilité.

---
# 35. Rust dans le noyau Linux

## 35.1 Pourquoi introduire un nouveau langage ?

Le noyau Linux est historiquement écrit très majoritairement en C, avec de l'assembleur pour certaines parties.

La sécurité mémoire devient cependant un sujet de plus en plus important, notamment pour les pilotes.

## 35.2 Linux 6.1

Le support initial de **Rust** est intégré au noyau mainline avec Linux **6.1**.

Il vise à permettre progressivement l'écriture de certains composants dans un langage offrant davantage de garanties de sécurité mémoire.

## 35.3 Ce que cela ne signifie pas

Rust ne remplace pas soudainement C.

Il s'agit d'une évolution progressive :

```text
énorme base historique en C
          +
infrastructure Rust
          +
nouveaux composants ciblés
```

Le noyau Linux de 2026 reste donc profondément lié au langage C, mais son histoire continue d'intégrer de nouvelles approches.

---
# 36. Les relations entre Linux et BSD aujourd'hui

Linux et les BSD sont des systèmes distincts.

Ils partagent cependant :

- beaucoup d'idées Unix ;
- des standards communs ;
- certains logiciels utilisateur ;
- des échanges de concepts et parfois de code selon les licences.

## 36.1 Différence d'organisation

Dans un projet BSD, le **base system** est généralement développé comme un ensemble cohérent comprenant noyau et outils fondamentaux.

Dans l'écosystème Linux, le noyau est un projet indépendant et les distributions assemblent de nombreux projets séparés.

Schématiquement :

```text
FreeBSD
  └── base system cohérent
        ├── noyau
        └── outils système

GNU/Linux
  ├── noyau Linux
  ├── GNU
  ├── systemd ou autre init
  ├── OpenSSH
  ├── environnement graphique
  └── intégration par la distribution
```

Cette différence organisationnelle est très importante pour comprendre les deux mondes.

---
# 37. macOS, Darwin et l'héritage BSD

macOS n'est pas une distribution Linux.

Apple utilise **Darwin**, dont l'architecture combine notamment :

- XNU ;
- Mach ;
- composants issus de BSD ;
- technologies propres à Apple.

Cela explique pourquoi de nombreuses commandes et interfaces paraissent familières à un administrateur Linux ou BSD sans que macOS utilise le noyau Linux.

Il est donc incorrect d'écrire :

```text
macOS = Linux
```

Il est plus juste de le placer dans la grande famille des systèmes Unix-like, avec une filiation technique différente.

---
# 38. Solaris et les autres UNIX commerciaux

## 38.1 SunOS et Solaris

Sun Microsystems, fondée en 1982, développe SunOS puis Solaris.

Solaris devient l'un des UNIX commerciaux les plus influents, notamment dans les serveurs et stations de travail.

## 38.2 Innovations

L'écosystème Solaris popularise ou développe des technologies importantes telles que :

- ZFS ;
- DTrace ;
- zones ;
- SMF.

Ces idées influenceront le reste de l'industrie, y compris l'écosystème Linux.

## 38.3 OpenSolaris et illumos

Sun lance OpenSolaris au milieu des années 2000.

Après le rachat de Sun par Oracle, l'écosystème libre se poursuit notamment autour d'**illumos** et de distributions dérivées.

---
# 39. Plan 9 : l'héritier expérimental de Bell Labs

À la fin des années 1980, des chercheurs de Bell Labs, dont Rob Pike et Ken Thompson, commencent **Plan 9**.

Plan 9 ne devient pas le successeur commercial universel d'UNIX, mais pousse plus loin plusieurs idées :

- ressources représentées via des espaces de noms ;
- forte orientation réseau ;
- protocole 9P ;
- environnement distribué.

Certaines idées de Plan 9 influencent encore des systèmes modernes.

Linux possède par exemple un client de système de fichiers 9P, utilisé dans certains environnements virtualisés ou de partage hôte-invité.

---
# 40. Une chronologie synthétique

| Date | Événement |
|---:|---|
| 1965 | Bell Labs rejoint le projet Multics avec MIT et GE |
| 1969 | Bell Labs quitte Multics ; Ken Thompson commence le système qui deviendra UNIX |
| 1970 | portage vers PDP-11 |
| 1972 | développement du langage C par Dennis Ritchie |
| 1973 | UNIX est réécrit en grande partie en C |
| 1977 | première Berkeley Software Distribution |
| 1983 | 4.2BSD et diffusion majeure de TCP/IP ; annonce du projet GNU |
| 1984 | début effectif du développement GNU |
| 1985 | création de la Free Software Foundation |
| 1989 | GPLv1 ; BSD Networking Release 1 |
| 1991 | GPLv2 ; annonce et premières versions du noyau Linux ; BSD Net/2 |
| 1992 | Linux passe sous GPLv2 ; 386BSD ; début du litige USL/BSDi |
| 1993 | NetBSD, FreeBSD, Slackware et Debian apparaissent |
| 1994 | Linux 1.0 ; 4.4BSD-Lite après assainissement de la lignée BSD |
| 1995 | naissance d'OpenBSD |
| 1996 | Linux 2.0 |
| 1997 | lancement de GNOME |
| 1998 | terme « open source » et création de l'OSI |
| 2005 | Git est créé pour répondre aux besoins du développement Linux |
| 2007 | création de la Linux Foundation ; GPLv3 |
| 2008 | première version commerciale d'Android, basée sur Linux |
| 2011 | Linux 3.0 |
| 2013 | Docker popularise les conteneurs Linux |
| 2015 | Linux 4.0 |
| 2019 | Linux 5.0 |
| 2022 | Linux 6.0 puis support initial de Rust dans 6.1 |
| 2026 | Linux 7.0 puis 7.2 ; branche stable 7.2.2 au 31 août |

> [!note]
> Une chronologie simplifie nécessairement l'histoire. Plusieurs projets commencent avant leur première publication officielle et de nombreuses lignées évoluent en parallèle.

---
# 41. Corriger les idées reçues

## 41.1 « Linux a été créé en 1994 »

Faux.

Linux commence en **1991**.

1994 correspond à la sortie de **Linux 1.0**.

## 41.2 « Linux est une copie du code UNIX »

Faux.

Linux reprend des **concepts et interfaces de type Unix**, mais son noyau a été développé séparément.

## 41.3 « GNU est une distribution Linux »

Faux.

GNU est un projet de système d'exploitation et un vaste ensemble de logiciels.

## 41.4 « Linux est un système d'exploitation complet »

Cela dépend du sens du mot `Linux`.

Techniquement, Linux désigne le noyau.

Dans le langage courant, « Linux » peut désigner une distribution complète.

## 41.5 « NetBSD apparaît en 1989 »

Faux.

1989 correspond notamment à BSD Networking Release 1.

NetBSD est fondé en **1993**.

## 41.6 « NetBSD utilise le noyau Mach »

Faux.

NetBSD possède son propre noyau BSD.

Mach est lié à d'autres lignées, notamment GNU Hurd et l'histoire de Darwin/XNU.

## 41.7 « AT&T distribuait UNIX en open source »

Anachronique et trompeur.

AT&T a distribué le code source d'UNIX dans différents contextes contractuels, mais cela ne correspond pas automatiquement à une licence open source au sens défini à partir de 1998.

## 41.8 « Open source veut dire domaine public »

Faux.

Un logiciel open source reste protégé par le droit d'auteur et distribué sous une licence accordant certains droits.

## 41.9 « GPL veut dire gratuit »

Faux.

La GPL permet la vente du logiciel. Elle impose surtout des obligations concernant les libertés et la redistribution du code dans les situations couvertes par la licence.

## 41.10 « Android est une distribution GNU/Linux classique »

Faux.

Android utilise Linux mais possède son propre espace utilisateur et sa propre pile applicative.

---
# 42. Pourquoi Linux a réussi

Il n'existe pas une cause unique.

## 42.1 Un bon moment historique

Au début des années 1990 :

- les PC x86 se généralisent ;
- Internet facilite la collaboration ;
- GNU fournit déjà beaucoup d'outils ;
- les UNIX commerciaux restent coûteux ou fragmentés ;
- BSD connaît une période d'incertitude juridique.

Linux arrive dans un contexte particulièrement favorable.

## 42.2 Une licence adaptée à la collaboration

La GPLv2 fournit un cadre qui favorise le partage des améliorations lors de redistributions couvertes par ses obligations.

## 42.3 Internet

Le développement distribué devient possible à grande échelle grâce :

- Usenet ;
- FTP ;
- courrier électronique ;
- listes de diffusion ;
- puis Git et les forges.

## 42.4 Une architecture pragmatique

Linux accepte rapidement de nombreux pilotes, systèmes de fichiers et architectures.

Le projet privilégie largement le développement incrémental et l'intégration de contributions concrètes.

## 42.5 Les distributions

Les distributions transforment un noyau et des logiciels en systèmes installables et maintenables.

## 42.6 Le soutien industriel

Les entreprises apportent :

- développeurs ;
- matériel ;
- tests ;
- financement ;
- certification ;
- support ;
- adoption en production.

## 42.7 La capacité d'adaptation

Linux s'adapte successivement à :

- serveurs ;
- clusters ;
- embarqué ;
- smartphones ;
- virtualisation ;
- cloud ;
- conteneurs ;
- architectures nouvelles ;
- exigences de sécurité modernes.

---
# 43. Observer l'histoire sur une machine Linux actuelle

Nous pouvons relier ce cours à notre propre système.

## 43.1 Version du noyau

```bash
uname -r
```

Exemple :

```text
6.8.0-xx-generic
```

ou une version plus récente selon la distribution.

## 43.2 Informations complètes

```bash
uname -a
```

## 43.3 Distribution

```bash
cat /etc/os-release
```

Nous pouvons récupérer :

- nom ;
- version ;
- identifiant ;
- famille éventuelle.

## 43.4 Compilateur utilisé pour le noyau

Selon les informations exposées :

```bash
cat /proc/version
```

Voir : [[proc]].

## 43.5 Architecture

```bash
uname -m
```

Exemples :

```text
x86_64
aarch64
riscv64
```

## 43.6 PID 1

```bash
ps -p 1 -o pid,comm,args
```

Cela permet d'observer quel système d'initialisation est utilisé.

## 43.7 libc

Sur un système glibc :

```bash
ldd --version
```

Une distribution Alpine peut au contraire utiliser musl.

Nous constatons ainsi qu'un système « Linux » est bien l'assemblage de nombreuses briques.

---
# 44. TP 1 — Identifier les différentes couches de notre système

## Objectif

Distinguer noyau, distribution et espace utilisateur.

## Étape 1 — noyau

```bash
uname -srmo
```

Noter :

- nom du noyau ;
- version ;
- architecture.

## Étape 2 — distribution

```bash
cat /etc/os-release
```

## Étape 3 — shell

```bash
printf '%s\n' "$SHELL"
```

Puis :

```bash
bash --version | head -1
```

si Bash est utilisé.

## Étape 4 — libc

```bash
ldd --version 2>&1 | head
```

## Étape 5 — init

```bash
ps -p 1 -o comm=
```

## Étape 6 — conclusion

Construire un schéma de ce type :

```text
Ubuntu 26.04
   │
   ├── Linux <version>
   ├── glibc <version>
   ├── systemd <version>
   ├── Bash <version>
   └── paquets Debian/Ubuntu
```

Nous ne devons plus écrire « Ubuntu est le noyau » ou « systemd est Linux ».

---
# 45. TP 2 — Comparer Linux et un système BSD

Nous pouvons réaliser cet exercice avec une machine virtuelle FreeBSD ou NetBSD.

## 45.1 Sur GNU/Linux

```bash
uname -a
cat /etc/os-release
ps -p 1 -o comm=
```

## 45.2 Sur FreeBSD

```sh
uname -a
freebsd-version
ps -p 1
```

## 45.3 Questions

Comparer :

- noyau ;
- commandes ;
- système d'init ;
- arborescence ;
- gestion des paquets ;
- emplacement de la configuration ;
- documentation ;
- licence du système de base.

## 45.4 Ce que nous devons observer

Les deux systèmes sont de type Unix et partagent de nombreuses habitudes, mais leur développement et leur intégration sont différents.

---
# 46. TP 3 — Lire l'historique du noyau avec Git

> [!warning]
> Le dépôt du noyau complet est volumineux. Pour un poste limité, utilisons un clone partiel ou consultons directement l'interface Web de kernel.org.

## 46.1 Clone partiel

```bash
git clone --filter=blob:none --no-checkout \
  https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

Puis :

```bash
cd linux
git log --oneline --decorate -n 20
```

## 46.2 Tags

```bash
git tag --list 'v7.*' | tail
```

## 46.3 Afficher une version

```bash
git show --no-patch v7.2
```

## 46.4 Comprendre la limite

L'historique Git officiel moderne du noyau ne contient pas nécessairement toute l'histoire originale de 1991 sous la forme d'un historique Git natif, car Git lui-même n'existe que depuis 2005.

Cette observation relie directement deux chapitres de l'histoire :

```text
Linux 1991
   │
   ├── patches / archives / outils antérieurs
   │
   └── Git créé en 2005
            │
            ▼
     historique moderne distribué
```

---
# 47. TP 4 — Vérifier les versions supportées plutôt que mémoriser un numéro

L'histoire des versions continue après l'écriture de ce cours.

Nous ne devons donc pas apprendre « la dernière version est 7.2.2 » comme une vérité permanente.

## 47.1 Source officielle

Consulter :

```text
https://kernel.org/
```

Identifier :

- mainline ;
- stable ;
- longterm ;
- linux-next.

## 47.2 Sur Ubuntu ou Debian

```bash
apt-cache policy linux-image-generic 2>/dev/null || true
```

## 47.3 Question

Pourquoi notre distribution peut-elle utiliser un noyau 6.x alors que kernel.org affiche une branche 7.x ?

Réponse attendue : la distribution choisit une branche, applique des correctifs, réalise ses propres tests et fournit une politique de support différente de la branche mainline.

---
# 48. TP 5 — Retracer une fonctionnalité moderne

Choisir une fonctionnalité :

- namespaces ;
- cgroups ;
- KVM ;
- eBPF ;
- WireGuard ;
- Rust ;
- io_uring ;
- Btrfs ;
- RISC-V.

Pour cette fonctionnalité, rechercher :

1. la première version du noyau qui l'introduit ;
2. les versions où elle devient réellement exploitable ;
3. les outils utilisateur nécessaires ;
4. une distribution qui l'active ;
5. son usage actuel.

L'objectif est de comprendre qu'une fonctionnalité ne naît pas toujours en une seule version.

Elle peut évoluer selon :

```text
prototype
   ↓
intégration initiale
   ↓
API instable
   ↓
améliorations
   ↓
adoption par les distributions
   ↓
usage industriel massif
```

---
# 49. Lecture historique critique

L'histoire de l'informatique contient beaucoup de récits simplifiés.

## 49.1 Vérifier les dates

Une date peut désigner :

- début d'un projet ;
- première annonce ;
- premier code public ;
- première version stable ;
- première commercialisation.

Dire « Linux date de 1994 » peut donc venir d'une confusion entre la naissance du projet et Linux 1.0.

## 49.2 Vérifier les filiations

Deux systèmes peuvent partager des interfaces sans partager le même code source.

Exemple : Linux reproduit de nombreuses interfaces Unix sans être dérivé du noyau UNIX historique.

## 49.3 Éviter les anachronismes juridiques

Le fait que du code source ait été disponible en 1975 ne signifie pas que sa licence était « open source » selon une définition formalisée en 1998.

## 49.4 Distinguer influence et copie

Une technologie peut :

- inspirer une autre ;
- fournir une API ;
- être réimplémentée ;
- être portée ;
- être réellement copiée sous licence.

Ces situations sont très différentes.

---
# 50. Ce qu'il faut retenir

## 50.1 La filiation intellectuelle

```text
Multics
   │
   ▼
UNIX ───────────────┐
   │                │
   │                ▼
   │               BSD
   │                │
   │                ├── NetBSD
   │                ├── FreeBSD
   │                ├── OpenBSD
   │                └── influence Darwin/macOS
   │
   ├── standardisation POSIX / UNIX®
   │
   └─────────────┐
                 ▼
          idée d'un Unix libre
                 │
                 ▼
                GNU
                 │
                 ├── GCC
                 ├── Bash
                 ├── glibc
                 └── nombreux outils
                         │
                         │
Linux 1991 ──────────────┘
      │
      ▼
distributions GNU/Linux
      │
      ├── serveurs
      ├── desktop
      ├── cloud
      ├── HPC
      └── embarqué

Linux ───────────────────► Android et autres espaces utilisateur non-GNU
```

## 50.2 Les cinq idées essentielles

1. **UNIX naît à Bell Labs en 1969** et devient portable grâce au langage C au début des années 1970.
2. **BSD** prolonge et transforme UNIX dans le monde universitaire, notamment autour du réseau.
3. **GNU**, lancé en 1983, construit les briques d'un système Unix libre et formalise le logiciel libre avec la FSF et la GPL.
4. **Linux**, commencé en 1991 et placé sous GPLv2 en 1992, fournit le noyau qui manquait à de nombreux utilisateurs des outils GNU.
5. Le succès actuel repose sur un **écosystème**, pas sur un seul projet : noyau, GNU, distributions, BSD, standards, entreprises, communautés, cloud, Android et milliers de projets indépendants.

---
# 51. Sources et références

## UNIX et Bell Labs

- Nokia Bell Labs, archive historique, **The Creation of the UNIX Operating System** : <https://www.nokia.com/bell-labs/unix-history/index.html>
- Computer History Museum, chronologie 1969 : <https://www.computerhistory.org/timeline/1969/>

## GNU, FSF et GPL

- Projet GNU, **Annonce initiale du projet GNU** : <https://www.gnu.org/gnu/initial-announcement.fr.html>
- Projet GNU, **Présentation du système GNU** : <https://www.gnu.org/gnu/gnu-history.fr.html>
- GNU GPL v1 : <https://www.gnu.org/licenses/old-licenses/gpl-1.0.html>
- GNU GPL v2 : <https://www.gnu.org/licenses/old-licenses/gpl-2.0.html>
- GNU GPL v3 : <https://www.gnu.org/licenses/gpl-3.0.html>

## BSD

- NetBSD, **The History of the NetBSD Project** : <https://www.netbsd.org/about/history.html>
- NetBSD, documentation BSD historique : <https://www.netbsd.org/docs/bsd/>
- FreeBSD, documentation du projet : <https://www.freebsd.org/docs/>

## Linux

- Linux Kernel Archives : <https://kernel.org/>
- Kernel.org, versions actives : <https://www.kernel.org/releases.html>
- Archive Linux 1.0 : <https://www.kernel.org/pub/linux/kernel/v1.0/>
- Documentation officielle du noyau : <https://docs.kernel.org/>
- Documentation Rust du noyau : <https://docs.kernel.org/rust/>

## Git

- Git SCM, **A Short History of Git** : <https://git-scm.com/book/en/v2/Getting-Started-A-Short-History-of-Git>

## Open source

- Open Source Initiative, **History of the Open Source Initiative** : <https://opensource.org/about/history-of-the-open-source-initiative>
- Open Source Definition : <https://opensource.org/osd>

## UNIX, POSIX et standards

- The Open Group, programme de certification UNIX : <https://www.opengroup.org/certifications/unix>
- The Open Group, UNIX Standard : <https://www.opengroup.org/membership/forums/platform/unix>
- POSIX / Base Specifications Issue 8 : <https://pubs.opengroup.org/onlinepubs/9799919799/>

## Linux Foundation

- Linux Foundation, histoire et fonctionnement : <https://www.linuxfoundation.org/>

## Android

- Android Open Source Project, **Kernel overview** : <https://source.android.com/docs/core/architecture/kernel>

---
# 52. Pour aller plus loin

Après ce cours, nous pouvons approfondir dans l'ordre suivant :

1. [[GNULinux]] — architecture et utilisation d'un système GNU/Linux ;
2. [[Les distributions Linux]] — différences entre familles de distributions ;
3. [[Initialisation système et des services]] — init, systemd et cycle de démarrage ;
4. [[proc]] — interface entre espace utilisateur et informations du noyau ;
5. [[Les namespaces Linux]] — isolation moderne ;
6. [[Docker]] — conteneurs ;
7. [[git]] — outil né des besoins du développement du noyau Linux ;
8. [[Sécurité avancée sous Linux]] — mécanismes de sécurité contemporains.
