---
author : Michaël Launay <michaellaunay@ecreall.com>
version : 2.0.0
licence : Cette création est mise à disposition selon le Contrat Paternité 2.0 France disponible en ligne <http://creativecommons.org/licenses/by/2.0/fr/> 
content : GNU/Linux french training based on https://github.com/michaellaunay/CoursGNULinux

---

# Objectif

Cette formation a pour but de fournir les bases indispensables à l'utilisation et à l'administration des systèmes GNU/Linux.

Cette formation privilégie la distribution Ubuntu.

# Introduction

En 1991, l'étudiant finlandais Linus Torvalds publie sur internet l'intégralité du code source d'un noyau Unix qu'il a écrit en C et en assembleur et qui fonctionne sur PC AT 386(486).

Depuis cette date GNU/Linux ne cesse d'évoluer. Il occupe en 2020 moins de 3,61% du marché mondial des systèmes d'exploitation pour ordinateur personnel, plus de 60% des serveurs web, prés de 75% du Cloud et plus de 80% des smartphones (Android étant basé sur GNU/Linux) et est en autre utilisé en France par la Gendarmerie (Ubuntu) et par l'Assemblée Nationale (Ubuntu), dans la Freebox, par l'entreprise Google (Android) et la fondation Wikipedia (serveur Ubuntu).

# Histoire

Voir [[Historique Linux]]

# Propriété intellectuelle et licence 

Voir [[Droits d'auteur]]

# Qu'est ce que GNU/Linux ?

GNU/Linux est un système d'exploitation libre et open-source, qui est utilisé sur des ordinateurs, des serveurs, des smartphones et d'autres dispositifs électroniques. Il est composé de deux éléments principaux : le système d'exploitation GNU et le noyau Linux.

## Qu'est-ce qu'un noyau ?

Pour définir le noyau, nous pouvons nous baser sur les services qu'il fournit :

    Abstraction du matériel (fourniture d'interface)
    Gestion des interruptions
    Gestion des tâches et autres logiciels
    Gestion des utilisateurs
    Gestion des droits d'accès

Historiquement on distingue les micro-noyaux des noyaux monolithiques.
Cette séparation vient de ce que le noyau est censé gérer (kernel space) et donc de ce qui est de la responsabilité des utilisateurs (user space). Dans les faits aujourd'hui même les noyaux monolithiques comme Linux sont modulaires et ne charge les modules que si nécessaire pendant l'utilisation.

[[Les distributions Linux]]

[[Installation Ubuntu]]

# Utilisation de GNU/Linux

Présentation interactive du système d'exploitation:
> -   le bureau,
> -   les fenêtres d'application,
> -   le tableau de bord.

Administration graphique du système:
> -   Configuration du réseau (Système (Flèche descendante de la barre de menus, à droite) \> Wifi ou Filaire (non) connecté ou Administration (Roue dentée) \> Wifi ou Réseau)
> -   Synaptic (Pour l'installer <https://doc.ubuntu-fr.org/synaptic> ): l'installation de logiciels (Système \> Administration \>
>     Gestionnaire de paquets Synaptic)
> -   configuration des dépôts (Rechercher depuis le menu Activité -\> Logiciels & mises à jour)
> -   personnalisations basiques <https://doc.ubuntu-fr.org/personnalisation_basique>
> -   la configuration de Gnome (installer gnome-tweaks )
> -   les applets
> -   la résolution graphique
> -   les bureaux virtuels
> -   les services (Système \> Administration \> Services)

Les logiciels d'administration ne sont que des surcouches graphiques (front-end) qui appellent les commandes en ligne, par conséquent leurs possibilités sont moindres.

## L'aide et la communauté

### L'aide en ligne

En mode graphique, les applications possèdent un onglet \"Aide\" permettant d'ouvrir un navigateur sur l'aide en ligne. Cette aide est généralement accessible par la touche `F1. @TODO

![Aide en ligne d'Ubuntu (appelée avec F1)](AideEnLigneUbuntu.jpg)

Dans un shell, la plupart des commandes unix acceptent l'option `-h` ou `--help` ou `--usage` :
```bash
apropos --help
```
Ce qui affiche :
```
Usage: apropos [OPTION...] KEYWORD...
Project-Id-Version: man-db 2.3.90
Report-Msgid-Bugs-To: Colin Watson <cjwatson@debian.org>
POT-Creation-Date: 2008-05-05 02:09+0100
PO-Revision-Date: 2008-08-19 20:37+0000
Last-Translator: Laurent Pelecq <laurent.pelecq@soleil.org>
Language-Team: French <traduc@traduc.org>
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Launchpad-Export-Date: 2008-11-09 09:58+0000
X-Generator: Launchpad (build Unknown)

  -d, --debug                emit debugging messages
  -v, --verbose              print verbose warning messages
  -e, --exact                search each keyword for exact match
  -r, --regex                interpret each keyword as a regex
  -w, --wildcard             the keyword(s) contain wildcards
  -a, --and                  require all keywords to match
  -l, --long                 do not trim output to terminal width
  -C, --config-file=FICHIER  use this user configuration file
  -L, --locale=LOCALE        define the locale for this search
  -m, --systems=SYSTEM       use manual pages from other systems
  -M, --manpath=CHEMIN       set search path for manual pages to PATH
  -s, --section=SECTION      search only this section
  -?, --help                 give this help list
	  --usage                give a short usage message
  -V, --version              print program version

Mandatory or optional arguments to long options are also mandatory or optional
for any corresponding short options.

The --regex option is enabled by default.

Report bugs to cjwatson@debian.org.
```
Pour trouver une commande il suffit de faire `apropos MotClé` qui affichera toutes les commandes comportant MotClé dans sa description courte.
Toutefois la base des commandes peut avoir besoin d'être régénérée par `makewhatis`.

`whatis NomDeCommande` affichera la description courte de NomDeCommande.

### Les man pages

Les applications et commandes possèdent toutes un manuel accessible en ligne de commande via la commande man.

Ce manuel est généralement traduit dans la langue de l'utilisateur :
```bash
man man
```
Ce qui affiche :
```
MAN(1)            Utilitaires de l’afficheur des pages de Manuel
MAN(1)

NOM
	   man - Interface de consultation des manuels de référence en ligne

SYNOPSIS
	   man  [-c|-w|-tZ] [-H[navigateur]] [-T[périphérique]] [-adhu7V] [-i|-I]
	   [-m système[,...]] [-L langue] [-p chaîne] [-C fichier] [-M chemin]
	   [-P afficheur] [-r invite] [-S liste] [-e extension] [[section] page ...] ...
	   man -l [-7] [-tZ] [-H[navigateur]] [-T[périphérique]] [-p chaîne]
	   [-P afficheur] [-r invite] fichier ...
	   man -k [apropos options] expression_rationnelle ...
	   man -f [whatis options] page ...

DESCRIPTION
   man est le programme de visualisation des pages de manuel.  Chacun  des  arguments  page,  indiqué dans la ligne de commande de man, porte, en principe, le nom d’un programme, d’un utilitaire ou d’une fonction. La page de manuel  Correspondant à chaque argument est alors trouvée et affichée. Si une section est précisée alors man limite  la  recherche  à  cette  section.  Par  défaut,  il recherche dans toutes les sections disponibles, suivant un ordre prédéfini. Il n’affiche que la première page de manuel trouvée, même si  d’autres  pages  de manuel existent dans d’autres sections.

	   Le  tableau  ci-dessous  indique le numéro des sections de manuel ainsi que le
	   type de pages qu’elles contiennent.

	   1   Programmes exécutables ou commandes de l’interpréteur de  commandes (shell) ;
	   2   Appels système (Fonctions fournies par le noyau) ;
	   3   Appels  de  bibliothèque  (fonctions  fournies  par  les  bibliothèques des programmes) ;
	   4   Fichiers spéciaux (situés généralement dans /dev) ;
	   5   Formats des fichiers et conventions. Par exemple /etc/passwd ;
	   6   Jeux ;
	   7   Divers (y compris les macropaquets et les  conventions).   Par exemple, man(7), groff(7) ;
	   8   Commandes  de  gestion  du  système (généralement réservées au superutilisateur) ;
	   9   Sous-programmes du noyau [hors standard].
```

Une page de manuel est constituée de plusieurs parties.
Elles peuvent être libellées NOM, SYNOPSIS,  DESCRIPTION,  OPTIONS,  FICHIERS, VOIR AUSSI, BOGUES et AUTEUR.

Pour chercher les pages associées à un mot clé:
```bash
man -k manual
```

```
apropos (1)          - search the manual page names and descriptions
catman (8)           - create or update the pre-formatted manual pages
esdcompat (1)        - manual page for pulseaudio esd wrapper 0.9.5
grub-reboot (8)      - manual page for grub-reboot 0.01
man (1)              - an interface to the on-line reference manuals
manconv (1)          - convert manual page from one encoding to another
mandb (8)            - create or update the manual page index caches
manpath (1)          - determine search path for manual pages
missing (7)          - missing manual pages
pulseaudio (1)       - manual page for pulseaudio 0.9.5
readahead-list (8)   - manual page for readahead-list: 0.20050517.0220
readahead-watch (8)  - manual page for readahead-watch: 0.20050517.0220
update-apt-xapian-index (8) - manual page for update-apt-xapian-index 0.15
w3mman (1)           - an interface to the on-line reference manuals by w3m(1)
whatis (1)           - display manual page descriptions
whereis (1)          - locate the binary, source, and manual page files for a command
xman (1)             - Manual page display program for the X Window System
```

### Les sites

Le site officiel de Linux <http://www.linux.org>

Un site dédié à Linux (Linux Entre Amis) : <http://www.lea-linux.org>

Une présentation de Linux <http://fr.wikipedia.org/wiki/Linux>

La communauté ubuntu française <http://www.ubuntu-fr.org/>

### Les forums

Le forum de la communauté Ubuntu <http://ubuntuforums.org/>

Le forum de la communauté Debian française <http://forum.debian-fr.org>

### Les LUGs

Un LUG est un groupe d'utilisateurs de Linux (Linux User Group) réuni généralement au sein d'une association loi 1901.

Dans la région lilloise on compte essentiellement Chtinux <http://www.chtinux.org/> anciennement Campux et CLX <http://clx.asso.fr/spip>

Les LUGs réalisent la promotion de Linux est des logiciels libres. Ils organisent des manifestations telles que des install party.

# Shell & Commandes

## Les terminaux (tty)

Historiquement, un terminal est une interface homme-machine minimale issue des technologies de communication de la fin XIX et du début XX siècle, le Télétype marque déposée en 1906 est l'ancêtre des claviers numériques des premiers ordinateurs.

L'abréviation tty de Télétype a été utilisée pour décrire l'interface série de communication utilisée au début d'Unix.
Par usage c'est le terme qui décrit l'interface de saisie et d'affichage avec l'humain.
On trouve aussi l'appellation de terminal ou console.

La commande tty affiche le pseudo fichier associé à la saisie.

Dans l'environnement graphique XWindows on trouve des logiciels émulant les terminaux, on les appelle alors des terminaux virtuels (ex: xterm).

Les terminaux ne sont en charge que de la récupération des touches frappées, de leur transformation en lettre, et de l'affichage de celle-ci. L'interprétation de ce qui est saisi est dévolue au shell.

Les six premiers terminaux sont accessibles par la combinaison de touche `Ctrl+Alt+F[1-6]`.

Le terminal graphique est accessible `Ctrl+Alt+F7`

## La ligne de commande

Sous Unix la CLI (Command Line Interface) est la méthode privilégiée pour transmettre au système les ordres à exécuter.

## Les différents shell

Le shell est un logiciel qui interprète séquentiellement les commandes saisies dans un terminal ou stockées dans un fichier (script) ou provenant d'un pseudo fichier.

La syntaxe et la sémantique de cette interprétation dépendent du shell employé.

Historiquement la première version est **sh** (1977 écrit par Stephen Bourne) qui évolua en **csh**, **ksh** et **bash** (Bourne again shell) le plus répandu.

Bash est l'interpréteur de commande par défaut des Unix libres et de Mac OS X.

Pour connaître la version de bash en cours d'utilisation:
```bash
echo $BASH
```

```
    /bin/bash
```

```bash
echo $BASH_VERSION
```

```
5.1.16(1)-release
```

Pour modifier le shell par défaut associé à un utilisateur il faut modifier `/etc/passwd` avec la commande `usermod -s /bin/bash login` :
```bash
grep michael /etc/passwd
```

```
michaellaunay:x:1000:1000:Michael Launay,,,:/home/michaellaunay:/bin/bash
```

```bash
sudo usermod -s /bin/sh michaellaunay
grep michael /etc/passwd
```

```
michaellaunay:x:1000:1000:Michael Launay,,,:/home/michaellaunay:/bin/sh
```
   
Pour créer un compte qui pourra se connecter sans avoir de shell (utilisation de tunnel) :
```bash
usermod -s /bin/false prestataire
```

Détails sur le format du fichier passwd
```bash
man 5 passwd
```

```
PASSWD(5)                                                                              Formats et conversions de fich                                                                              PASSWD(5)

NOM
       passwd - fichier des mots de passe

DESCRIPTION
       /etc/passwd contains one line for each user account, with seven fields delimited by colons (« : »). These fields are:

       •   nom de connexion de l'utilisateur (« login »)

       •   un mot de passe chiffré optionnel

       •   l'identifiant numérique de l'utilisateur

       •   l'identifiant numérique du groupe de l'utilisateur

       •   le nom complet de l'utilisateur ou un champ de commentaires

       •   le répertoire personnel de l'utilisateur

       •   l'interpréteur de commandes de l'utilisateur (optionnel)
...
```

Plus d'information : `man bash`

### Lien
[Bourne-Again-shell](http://fr.wikipedia.org/wiki/Bourne-Again_shell)

## Les fichiers de ressources et de configuration de bash

Lors de son démarrage, le shell bash détermine comment il a été appelé, soit de façon interactive, soit pour exécuter un script, soit en tant que shell de login. En fonction de la nature de son lancement, il exécutera différents fichiers pour se configurer.

Lorsqu'un shell interactif est lancé en ouverture de session (c'est-à-dire, un shell interactif de login), il exécute les scripts suivants :
```
/etc/profile
~/.bash_profile #le ~ désigne le répertoire "home" de l'utilisateur
~/.bash_login #si ~/.bash_profile n'existe pas
~/.profile #si ~/.bash_login
```

Pour un shell interactif non-login, les scripts suivants sont exécutés :
```
/etc/bash.bashrc
~/.bashrc
```

Pour que les modifications de ces scripts soient prises en compte immédiatement dans le shell courant, nous devons utiliser la commande `source`.

Lorsqu'un script est exécuté en mode non interactif, le shell tente alors d'exécuter le fichier désigné par la variable `BASH_ENV`, si celle-ci existe, pour connaitre son contenu nous pouvons faire :
```bash
echo $BASH_ENV
```
Qui par défaut n'affichera rien, car elle est non définie, mais si nous affectons à `BASH_ENV` le chemin d'un script, nous modifierons l'exécution de toutes les scripts lancées après.
Un petit exemple :
```bash
echo "echo \# Débogage" > /tmp/trace.sh # Le script trace.sh contientra la commande `echo  \#Débogage`
echo "set -x" >> /tmp/trace.sh # Nous ajoutons la commande d'affichage des lignes de commandes
chmod +x /tmp/trace.sh   # Nous rendons exécutable ce fichier
/tmp/trace.sh            # Nous exécutons ce fichier pour le tester
```
Nous venons de créer un script exécutable `/tmp/trace.sh` qui à son exécution affiche et positionne l'affichage des lignes de commandes avant  exécution :
```
# Débogage
```
Nous allons maintenant affecter le chemin de cette commande à la variable `BASH_ENV` et constater les modifications.
```bash
echo $BASH_ENV           # Nous affichons le contenu de la variable BASH_ENV
BASH_ENV='/tmp/trace.sh' # Nous affectons la chaîne /tmp/trace.sh à la variable BASH_ENV
export BASH_ENV # maintenant BASH_ENV sera accessible à toute commande exécutée depuis le shell courant
echo "echo cuicui" > /tmp/oiseau.sh
bash /tmp/oiseau.sh # Nous exécute oiseau.sh avec bash, mais nous aurions pu faire `chmod +x` dessus pour le rendre directement exécutable.
```
affiche :
```
# Débogage
++ echo cuicui
cuicui
```
Où nous voyons que les commandes du fichier `/tmp/oiseau.sh` avant leur exécution.

## Les variables d'environnement

Les variables d'environnement sont accessibles en consultation avec la commande `env` :
```bash
env
```
Affiche les variables d'environnement et leur valeur :
```
SHELL=/bin/bash
TERM=xterm
HISTSIZE=1000
USERNAME=michaellaunay
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/home/michaellaunay
EDITOR=vim
LANG=fr_FR.UTF-8
HOME=/home/michaellaunay
LOGNAME=michaellaunay
DISPLAY=:0.0
OLDPWD=/home/michaellaunay
```

Signification des variables d'environnement :
```
BASH      Le nom du fichier bash
DISPLAY   Le numéro de serveur et de session d'affichage
EDITOR    L'éditeur à utiliser par défaut
HISTSIZE  La taille du fichier historique
HOSTNAME  Le nom de la machine
HOME      Le répertoire personnel de l'utilisateur
LANG      La langue de l'utilisateur et l'encodage utilisé pour afficher cette langue
LOGNAME   Le nom d'utilisateur lors de l'ouverture de la session
MAIL      Le chemin vers la boite mail de l'utilisateur
OLDPWD    Le répertoire où nous étions avant le dernier cd
PATH      Le chemin vers les exécutables
PS1       Permet de constituer l'invite de commande
PS2       Symbole affiché sur les lignes de commande débordant sur plusieurs lignes
PROMPT_COMMAND Le nom d'une commande à exécuter à chaque commande
PWD       Le chemin actuel
SHELL     Le shell de l'utilisateur
TERM      Le type de terminal
USERNAME  Le nom d'utilisateur
```

Pour accéder au contenu d'une variable, il suffit de la référencer en la précédent de **\$**:
```bash
echo $HOME
```

```
/home/michaellaunay
```

Pour voir l'ensemble des définitions réalisées dans un shell (variable et fonction) il suffit de taper **set**.

Pour voir les lignes exécutées dans un script `set -x` en début de script.

## Les caractères spéciaux

Les caractères suivants permettent de déclencher des comportements particuliers :
```
# Mise en commentaire
> Indirection vers un fichier
< Indirection depuis un fichier
| Pipe
? Un caractère ou pas
. Un caractère
* Une chaîne de caractère
$ Référencement d'une variable
\ Échappement
/ Séparateur
[ Début d'un ensemble ou d'un test
] Fin d'un ensemble ou d'un test
( Sous shell ou évaluation
) Fin de sous shell ou d'évaluation
: Séparateur de groupe
; Fin de commande
^ Inversion ou début
@ Adresse
` Début ou fin d'interprétation
~ Désigne le répertoire personnel
```

Si nous voulons les utiliser pour nommer par exemple un fichier sans que le comportement particulier soit déclenché nous avons l'obligation de les échapper avec **\\** ou de les mettre entre apostrophes **'** ou guillemets **"**:
```
\# ou '#' ou "#"
\> ou '>' ou ">"
\< ou '<' ou "<"
\| ou '|' ou "|"
\? ou '?' ou "?"
\. ou '.' ou "."
\* ou '*' ou "*"
\$ ou '$' ou "$"
\\ ou '' ou "\"
\/ ou '/' ou "/"
\[ ou '[' ou "["
\] ou ']' ou "]"
\( ou '(' ou "("
\) ou ')' ou ")"
\: ou ':' ou ":"
\; ou ';' ou ";"
\^ ou '^' ou "^"
```

### Exemple

Nous allons créer un fichier ayant pour nom `[*]^[*]`
```bash
echo lunettes > /tmp/\[\*\]\^\["*"']'
ls /tmp
```
Nous voyons que le répertoire `/tmp` contient bien :
```
[*]^[*]
```
Si nous affichons le contenu de ce fichier avec la commande suivantes :
```bash
cat /tmp/\[\*\]\^\[\*\]
```
Alors nous avons :
```
lunettes
```

## Variables spéciales

En plus des variables d'environnement vue précédemment nous avons :
```
$? Qui fait référence au code de retour de la dernière commande exécuté.
$$ Le pid du programme en cours d'exécution.
$! Le pid de la dernière commande lancée en tâche de fond.
$# Le nombre de paramètres.
$0 Le nom du programme en cours d'exécution.
$1 Le premier paramètre passé.
$2 Le second paramètre passé.
...
$9 Le neuvième paramètre.
${10} Le dixième paramètre
...
${99} Le quatrevingt dixneuvième paramètre
$*, $@ # L'ensemble des paramètres
```

## Création, affectation de variable

Pour créer une variable ou en modifier sa valeur, il suffit de la définir :
```bash
VAR='Bonjour tout le monde'
```
Ce qui crée la variable "VAR" et lui affecte la valeur 'Bonjour tout le monde'.
Que nous pouvons afficher avec la commande suivante :
```bash
echo $VAR
```
Ce qui affiche :
```
Bonjour tout le monde
```
Modifions sa valeur et affichons la :
```bash
VAR=Salut
echo $VAR
```
Nous obtenons:
```
Salut
```
Ajoutons une chaîne de caractères à cette variable et affichons la :
```bash
VAR=$VAR' à tous' # Attention à coller l'apostrophe au nom de la variable sans quoi il y aura une évaluation/exécution de la chaîne de caractères.
echo $VAR
```

```
Salut à tous
```

Exemple avec la variable d'environnement PATH qui indique au système où chercher les exécutables lorsqu'on tape une commande dans le terminal.
```bash
PATH=/home/michaellaunay/MesScripts:$PATH
echo $PATH
```
Dans cet exemple, nous avons modifié la valeur de la variable PATH, en ajoutant le chemin '/home/michaellaunay/MesScripts' au début de la valeur actuelle de PATH.
Son affichage avec la commande `echo` produit :
```
/home/michaellaunay/MesScripts:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Pour supprimer une variable, on peut utiliser **unset** :
```bash
unset BASH_ENV
```

## Protection des espaces des valeurs des variables

Attention bash passe les mots d'une variable en les séparant d'un unique espace. Ainsi :
```bash
VARIABLE="  commence par 2 espaces puis en contient cinq     et finit par deux  "
echo DEBUT${VARIABLE}FIN #L'abscence de guillemet forcent le bash à passer en paramettre chacuns des mots séparé par un unique espace.
```
Ce qui affiche :
```
DEBUT commence par 2 espaces puis en contient cinq et finit par deux FIN
```

```bash
echo DEBUT"$VARIABLE"FIN #Les guillemets protègent les espaces !
```
```
DEBUT  commence par 2 espaces puis en contient cinq      et finit par deux  FIN
```

On peut aussi échapper les espaces avec des antislashs lors de l'affectation de la variable.

## Évaluation d'une variable

Une variable peut contenir une commande. Son simple appel suffit à provoquer son évaluation :
```bash
MON_SCRIPT="/tmp/ma_cmd.sh"
echo "echo Hello" > $MON_SCRIPT
chmod +x $MON_SCRIPT
$MON_SCRIPT #exécute la commande contenue dans la variable MON_SCRIPT
```
Qui affiche
```
Hello
```
Alors que :
```bash
cat $MON_SCRIPT #Affiche ce qu'il y a dans le script /tmp/ma_cmd.sh
```
Affiche le contenu du fichier :
```
#! /bin/bash
echo Hello
```

## Export de variable

Toute variable créée dans un shell n'est accessible que dans celui-ci.

Pour la rendre accessible aux commandes et scripts appelés après l'affectation il faut l'exporter.
Exemple :
```bash
echo "echo \$SALUTATION" > /tmp/cmd.sh
chmod +x /tmp/cmd.sh
/tmp/cmd.sh
```
N'affichera rien tant que la variable SALUTATION n'aura pas été définie **et** exportée :
```bash
SALUTATION=coucou
echo $SALUTATION
```
Qui affiche bien le contenu de salutation depuis le shell courant :
```
coucou
```
Mais si on exécute le script `/tmp/cmd.sh` alors SALUTATION est encore vide pour lui
```bash
/tmp/cmd.sh
```
Qui n'affiche rien, sauf si l'on fait un `export` comme ci dessous :
```bash
export SALUTATION
/tmp/cmd.sh
```
Nous obtenons enfin :
```
coucou
```

## Les tests d'expressions et fichier, opérateurs de contrôle

La commande **test** permet de tester une expression et de retourner 0 si le test est vrai et 1 s'il est faux :
```bash
test 1 = 1
echo $?
```
Nous obtenons **ZÉRO** ! 
En shell cela signifie qu'il n'y a pas d'erreur, la logique est donc inversée par rapport au langage de programmation classique :
```
0
```
Par contre :
```bash
test 1 = 2
echo $?
```
Donne un résultat différent de ZÉRO ce qui indique une erreur et généralement le numéro de l'erreur.
```
1
```

Nous pouvons aussi remplacer la commande `test` par des crochets, mais il faut alors encadrer les crochets par des espaces :
```bash
[ 1 = 2 ]
echo $?
```
**Attention** les espaces avant et après les crochets sont indispensables (sauf en début ou fin de ligne) pour permettre à l'interpréteur de commandes d'identifier un test et délimiter ce qui est testé.
Ici le résultat est bien différent de 0 donc qu'il n'y a pas d'égalité.
```
1
```

Les options de test sont très nombreuses. N'hésitons pas à faire un `man test`.

Avec **test** et **if** il est possible d'exécuter conditionnellement des commandes :
```bash
VAR=2
if [ $VAR = 2 ]; then echo Vrai; else echo Faux;fi
```
Affiche la chaîne de caractères correspondant au résultat de l'évaluation. Ici :
```
Vrai
```
Autre forme d'écriture possible (notons le remplacement des `;` par des sauts de lignes) :
```bash
VAR=$HOME
if [ -w $VAR ]
 then echo écriture possible dans $VAR
 else echo écriture impossible dans $VAR
fi
```
Qui donne le résultat suivant :
```
écriture possible dans /home/michaellaunay
```

## Exécution d'opérations arithmétiques

La construction **$[ nombre1 opérateur nombre2 ]** permet de réaliser le calcul d'expression sur des entiers :
```bash
echo $[ 10 - 1 ]
```
Encore une fois notons bien les espaces avant les crochets !
Qui donne bien le résultat attendu :
```
9
```

## La création d'une variable et sa modification

Lors de la création et la modification d'une variable, nous utilisons l'opérateur égal, tout comme dans la plupart des langages de programmation :

```bash
CMPT=0 # Cette ligne est équivalente à la suivante
let CMPT=0
# Que nous affichons
echo $CMPT
```

Ce qui affiche :

```
0
```

Nous pouvons augmenter la valeur de `CMPT` de 1 :

```bash
let CMPT+=1
echo $CMPT
```

Et nous obtenons :

```
1
```

Si nous augmentons `CMPT` :

```bash
let CMPT+=1 # équivalent à `CMPT=$[ CMPT + 1 ]`
CMPT=$[ CMPT + 1 ] # nous incrémentons à nouveau 
echo $CMPT
```

Nous obtenons :

```
3
```

## La boucle while et until

**`While`** permet d'exécuter des commandes tant que la condition est satisfaite alors que **`until`** exécute des commandes tant que la condition échoue.

Exemple :
```bash
VAR=4
while [ $VAR -gt 0 ]
 do
  echo itération $VAR;
  VAR=$[ $VAR - 1 ]
 done
```
 Ce qui donne :
```
itération 4
itération 3
itération 2
itération 1
```

## La boucle for

Pour chaque élément d'un ensemble, nous exécutons une commande :
```bash
NORD="Lille Roubaix"
CENTRE="Paris Chartres"
SUD="Nice Marseille"
for ville in $NORD $CENTRE $SUD
 do
  echo Visiter $ville
 done
```
 
```
Visiter Lille
Visiter Roubaix
Visiter Paris
Visiter Chartres
Visiter Nice
Visiter Marseille
```

## Le choix multiple (case)

Permet de réaliser un branchement. Ne pas oublier les deux points-virgules à la fin d'un cas :
```bash
VAR=Lille
case $VAR in
 'lille' | 'Lille' | 'LILLE' )
   echo J\'y habite # Remarquons l'échapement de l'apostrophe
 ;;
 'paris' | 'Paris' | 'PARIS' )
   echo J\'y ai habité
 ;;
 * )
   echo Je ne connais pas
 ;;
 esac
```
Ce qui donne :
```
J'y habite
```

## Les opérateurs && et \|\|

L'opérateur **&&** permet d'exécuter la commande suivante si la commande précédente réussit (retourne 0) :
```bash
grep refusée /var/log/user.log > /tmp/connexion.txt && vim /tmp/connexion.txt
```
L'opérateur **\|\|** permet d'exécuter la commande suivante si la commande précédente a échoué (retour de 1) :
```bash
grep refusée /var/log/user.log > /dev/null || echo tout va bien
```

## La commande trap

Elle permet de positionner une fonction qui sera exécutée lors de la réception d'un signal (man 7 signal) :
``` bash
trap "echo Fin d'exécution" EXIT
trap "echo Interruption violente Ctrl-c" SIGINT
trap "echo Fin demandée" SIGTERM
trap "echo Reprise d'exécution" SIGCONT
trap "echo Signal USR" SIGUSR1 SIGUSR2
```

## La commande sed

Elle permet de faire des traitements sur les lignes d'un flux.
Par exemple elle permet de trouver un motif et de le remplacer.
Nous la rencontrons dans de nombreux scripts.

Par exemple dans la ligne suivante :
```bash
ls -1 | xargs -i echo mv {} {} | sed -e "s/Ubuntu22.04_//2" | bash
```

`ls -1` affiche le contenu du répertoire courant, une ligne par fichier.

Le résultat est envoyé à **`xargs`** qui pour chaque ligne va créer une chaîne de caractères `mv contenu_ligne contenu_ligne`

Le résultat est envoyé à **`sed`** qui supprime la seconde occurrence de la chaîne \"Ubuntu22.04\" qu'il rencontre.

Le résultat est exécuté par bash en transformant la chaîne de caractères reçue en ligne de commande.

Ici **`sed`** permet de renommer les fichiers de type Ubuntu22.04\_00\_EssayerOuInstaller.png en 00\_EssayerOuInstaller.png.

À cette ligne complexe, nous préférerons renommer de façon plus élégante et rapide avec la ligne de commande :

```bash
for filename in *; do mv $filename ${filename/Ubuntu22.04_/}; done
```

Pour plus d'information sur **sed** voir
<https://www.commentcamarche.net/faq/9536-sed-introduction-a-sed-part-i>

## L'expansion de paramètre

Liste des Filtres pour l'expansion de paramètre du Shell
<https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html>

Ici `${parameter}`` sera remplacé par la valeur de parameter
```bash
CMPT=$(( 1 + 20 / 2 ))
echo ${CMPT}
```
 Réalise l'opération puis affecte CMPT. Pour les opérations possibles voir https://www.gnu.org/software/bash/manual/html_node/Shell-Arithmetic.html#Shell-Arithmetic
 Le résultat est :
```
11
```

Pour créer des tableaux :
```bash
name[1]='un'
name[2]='deux'
name[3]='trois'
echo ${name[2]}
```
 Qui est équivalent à 'declare -n name' voir https://www.gnu.org/software/bash/manual/html_node/Arrays.html#Arrays
 Le résultat est :
```
deux
```
Équivalent à :
```bash
name=('zero' 'un' 'deux' 'trois')
echo ${name[1]}
```
Et a pour résultat :
```
un
```
Pour supprimer une entrée :
```bash
unset name[0]
echo ${name[0]}
```
Qui affiche une entrée vide
```
```
Alors :
```bash
echo ${name[1]}
```
Affiche encore :
```
un
```

## Les scripts

Un script est un fichier qui contient une suite de commandes.

La première ligne permet d'indiquer le shell dans lequel doit être exécuté le script :

```bash
#!/bin/bash
echo "C'est du bash"
```

Cette ligne s'appelle le [shebang](http://en.wikipedia.org/wiki/Shebang_(Unix)).

## Les fonctions

Une fonction est une portion de code nommée, réutilisable qui a accès à toutes les variables du script ou du shell d'où elle est appelée :
```bash
function carré() {
	echo $[ $1 * $1]
}
```
Que nous appelons comme cela :
```bash
carré 3
```
Et qui donne :
```
9
```

## Lecture des saisies clavier

La commande **`read`** permet de lire la saisie clavier et de l'affecter avec une variable :

```bash
read VAR
```
Qui attend de l'utilisateur la saisie d'une chaîne de caractères terminée par `Entrée`, par exemple :
```
coucou
```
Nous pouvons alors consulter la saisie qui a été stockée dans la variable :
```bash
echo $VAR
```

```
coucou
```

## Exercice

Réalisez une calculatrice demandant la saisie de la première opérande puis de l'opérateur (symbole ou littéral), puis de la seconde opérande.
Affichez le résultat puis exécutez à nouveau tant que le signal SIGUSR1 n'est pas reçu.

## Les sous-programmes

Dans un shell on peut appeler un script directement en passant son nom si celui-ci est exécutable ou en le faisant interpréter par le shell pour lequel il a été écrit.

Lorsqu'on exécute un ensemble de commandes encadré par des parenthèses alors le shell courant démarre un sous shell pour exécuter les commandes :

```bash
VAR=0
(VAR=$[ $VAR + 1]; echo $VAR)  
```
Un sous shell a été lancé et dans ce shell le calcul a été fait "localement" :
```
1
```
Mais cela n'a pas affecté la variable du shell parent :
```bash
echo $VAR
```
Et nous voyons que la valeur est restée à 0 :
```
0
```
Il est également possible de forcer l'exécution de commande en utilisant **\`** :
```bash
echo Quelle est la `date` ?
```
Ce qui affiche la chaîne de caractères "date" :
```
Quelle est la date ?
``` 
Mais nous pouvons lancer la commande `date`  en utilisant **\`**:
 ```bash
echo Aujourd\'hui nous sommes le `date`
```
Ce qui affiche :
```
Aujourd'hui nous sommes le jeu. 06 juil. 2023 16:54:23 CEST
```

## La complétion de commande

En appuyant sur la touche tab le shell affiche toutes les commandes ayant pour préfixe les lettres déjà saisies sur la ligne de commande.

## Historique des commandes

Les commandes saisies dans un shell sont enregistrées dans le fichier \~/.bash\_history

Il est possible d'accéder aux anciennes commandes en utilisant les flèches.

## Les commandes de base pour se déplacer dans l'arborescence

Voici les commandes les plus utilisées

### ls

ls Permet d'afficher les informations d'un fichier ou d'un répertoire, sans paramètre elle affiche le contenu du répertoire courant.

```bash
ls UnChemin # Affiche le contenu de UnChemin si c'est un répertoire, sinon affiche le nom de UnChemin
ls -lah # Affiche les détails, les fichiers cachés, et utilise des unités informatiques
ls -F # Affiche un / derrière le nom des répertoires 
info ls # Permet de connaître le sens des colonnes des options de `ls`, par exemple le chiffre de la seconde colonne de l'option -l est le nombre de hard links.
```

Ainsi :
```bash
ls -lh
```
Affiche :
```
total 2,1G
drwxr-xr-x   2 root root  12K avril 19 16:40 bin
drwxr-xr-x   4 root root 4,0K avril  8 06:51 boot
drwxr-xr-x   2 root root 4,0K mai   16  2019 cdrom
drwxr-xr-x  19 root root 4,6K avril 18 22:11 dev
drwxr-xr-x 158 root root  12K avril 15 06:43 etc
drwxr-xr-x   5 root root 4,0K août  22  2019 home
lrwxrwxrwx   1 root root   32 janv.  6 18:48 initrd.img -> boot/initrd.img-5.0.0-38-generic
lrwxrwxrwx   1 root root   32 janv.  6 18:48 initrd.img.old -> boot/initrd.img-5.0.0-37-generic
drwxr-xr-x  21 root root 4,0K mars   5 06:28 lib
drwxr-xr-x   2 root root 4,0K mars   5 06:28 lib32
drwxr-xr-x   2 root root 4,0K mars   5 06:28 lib64
drwx------   2 root root  16K mai   16  2019 lost+found
drwxr-xr-x   3 root root 4,0K juin  24  2019 media
drwxr-xr-x   2 root root 4,0K févr. 10  2019 mnt
drwxr-xr-x   5 root root 4,0K août  26  2019 opt
dr-xr-xr-x 354 root root    0 avril 18 22:11 proc
drwx------   8 root root 4,0K mars  25 10:16 root
drwxr-xr-x  39 root root 1,1K avril 19 10:15 run
drwxr-xr-x   2 root root  12K avril 19 16:40 sbin
drwxr-xr-x  17 root root 4,0K mars  22 22:44 snap
drwxr-xr-x   2 root root 4,0K févr. 10  2019 srv
-rw-------   1 root root 2,0G mai   16  2019 swapfile
dr-xr-xr-x  13 root root    0 avril 18 22:11 sys
drwxrwxrwt  24 root root 4,0K avril 19 17:20 tmp
drwxr-xr-x  14 root root 4,0K août   1  2019 usr
drwxr-xr-x  15 root root 4,0K juin  17  2019 var
lrwxrwxrwx   1 root root   29 janv.  6 18:48 vmlinuz -> boot/vmlinuz-5.0.0-38-generic
lrwxrwxrwx   1 root root   29 janv.  6 18:48 vmlinuz.old -> boot/vmlinuz-5.0.0-37-generic
```

### pwd

**`pwd`**  Affiche le chemin du répertoire courant

Si l'on exécute la commande `pwd` pour voir quel est le répertoire courant :
```bash
pwd
```
Et qu'il affiche
```
/home/michaellaunay/Documents/Notes/cours
```
Alors c'est que nous sommes dans ce répertoire.

### cd

**`cd`** Permet de déplacer le répertoire courant si elle n'a pas de paramètre et dans le répertoire donné en paramètre sinon.

```bash
cd # équivalent à cd $HOME
pwd
```
Nous changeons de répertoire en allant dans notre `home` , `pwd` affiche alors :
```
/home/michaellaunay
```

## Les jokers pour les noms de fichiers

Les caractères spéciaux sont très utilisés pour désigner des noms de fichiers :
```
* # Désigne toute chaîne contiguë de caractères
? # Désigne un caractère
[...] # Permet de désigner des ensembles de caractères [4-69] accepte 4, 5, 6, et 9, [[] accepte [ identique à \[A
[^...] # Permet de désigner des ensembles à exclure
{1..9} #Permet de désigner et créer tous les fichiers nommés de 1 à 9
```
Par exemple :
```bash
touch /tmp/fich{0..9} # Créer des fichiers vides ayant pour nom fich0 à fich9
ls /tmp/fich[5-9]
```
Affichera :
```
/tmp/fich5  /tmp/fich6  /tmp/fich7  /tmp/fich8  /tmp/fich9
```

## Les chemins relatifs

Un **chemin relatif** est un chemin qui permet de se déplacer jusqu'au fichier cible à partir du chemin courant :
```bash
cd ~ # identique à cd $HOME ou cd
ls -l ../../etc/passwd
```
Affichera :
```
-rw-r--r-- 1 root root 1583 2009-04-02 11:35 ../../etc/passwd
```

Le symbole **.** indique le répertoire courant alors que les symboles **..** indiquent le parent.

## Les chemins absolus

Un **chemin absolu** est un chemin qui commence à la racine **/** de l'arborescence et énonce tous les sous-répertoires jusqu'à la cible :

```bash
ls -l /etc/passwd
```
Affichera
```
-rw-r--r-- 1 root root 1583 2009-04-02 11:35 /etc/passwd
```


## Création / suppression de répertoire

### mkdir

La commande **`mkdir`** permet de créer des répertoires :
```bash
mkdir NomRep # Crée le répertoire NomRep.
mkdir -p Rep1/Rep2/Rep3 # Crée Rep3 et l'arborescence Rep1/Rep2 si nécessaire.
```

### rmdir

La commande **`rmdir`** permet de supprimer un répertoire vide, on peut aussi le faire avec **rm -r** dans le cas d'un répertoire non vide.

## Lecture de fichier

### cat

La commande **`cat`** permet d'afficher le contenu d'un fichier.

### strings

La commande **`strings`** permet de n'afficher que les chaînes de caractères d'un fichier binaire.

## Rechercher des fichiers

### find

La commande **`find`** permet de réaliser des recherches basées sur les informations d'un fichier (nom, date de création, de modification, etc.) :

```bash
find Documents/ecreall -name "*pdf" -ctime -2
```
Recherche à partir du répertoire Documents/ecreall tous les fichiers finissant par pdf, créés depuis moins de 2 jours. Cela affiche par exemple
```
Documents/ecreall/Cours/CoursGNULinux/CoursGNULinux.pdf
```

### grep

La commande **grep** permet de réaliser des recherches basées sur la présence d'une chaîne ou d'une expression régulière dans le contenu d'un fichier.

### locate

La commande **locate** permet de trouver un fichier si le chemin a été renseigné dans la base de données mise à jour par le super utilisateur avec **updatedb** ou **slocate -u**.

## Archivage / Compression

### zip

**`zip`**, **`unzip`** permet de compresser et décompresser les fichiers aux format zip.

### gzip

**`gzip`** permet de compresser et décompresser les fichiers au format gzip.

### tar

**`tar`** avec les options **cf** permet d'archiver une arborescence en conservant les informations de propriétaire, les dates de création, les permissions d'accès. Avec les options **xf**, permet d'extraire une archive.

**`tar cfz`** permet de combiner **tar** et **gzip** en une commande.
L'option **`--listed-incremental=nom_fichier.list`** permet d'enregistrer un snapshot des fichiers archivés en vue de permettre des `tar` incrémentaux. C.f.
<https://doc.ubuntu-fr.org/tar#utilisation_en_archivage_incrementiel>
Attention il est indispensable que la première archive soit lancée avec cette option pour que l'incrémentation soit possible !

## Autres commandes

### mv

**`mv`** permet de déplacer un fichier ou une arborescence.

### tail

**`tail`** permet de n'afficher que les dernières lignes d'un fichier, l'option -f permet d'afficher le contenu au fur et à mesure de son arrivé dans le flux.

### tee

**`tee`** permet d'écrire le contenu de la sortie standard dans un fichier tout en laissant ce contenu dans la sortie standard ce qui permet dans un pipe d'avoir une capture du contenu sans casser le pipe.

### ln

**`ln`** permet de créer des liens. Ainsi **`ln -s Source Destination`** permet de créer un lien symbolique.

### cp

**`cp`** permet de copier un fichier dans un autre. **`cp -r Rep1 Rep2`** copie toute l'arborescence Rep1 vers Rep2.

### script

**`script NOM_Fichier`** permet d'enregistrer la session (les interactions en ligne de commande) vers un fichier, ce qui permet de l'auditer voire de la rejouer.

L'option -t permet d'enregistrer les dates des échanges vers le flux d'erreur. L'enregistrement sera arrêté par la commande **`exit`**. 

**`scriptreplay`** Permet de rejouer la session.

```bash
NOM=`date +%Y%m%d%H%m%S`_upgrade_ubuntu
script -a ~/$NOM.script -t 2> ~/$NOM.time
```

## Les noms de fichiers

Linux est sensible à la casse (majuscules vs minuscules).

Depuis 2007, l'ensemble du système utilise [utf-8](http://fr.wikipedia.org/wiki/Utf-8) comme encodage par défaut, 
en conséquence tous les caractères accentués peuvent être utilisés pour nommer les fichiers.

Les caractères spéciaux et les espaces peuvent être utilisés à la condition d'être échappés.

La taille des noms ne doit pas excéder 255 octets.

Si nous utilisons des caractères accentués ou asiatiques, le nombre de caractères maximal est inférieur à 255, car il faut 2 à 4 octets pour représenter un caractère autre que ASCII en
[utf-8](http://fr.wikipedia.org/wiki/Utf-8).

Tout fichier ou répertoire commençant par un **.** sera caché et accessible uniquement avec l'option **-a** de **ls**.

## Les attributs des fichiers

Les attributs de fichier permettent de gérer les permissions d'accès en lecture, écriture, exécution, traversée et également de connaître la nature du fichier.

Ainsi :

```bash
ls -lh
```
```
total 24K
lrwxrwxrwx   1 michaellaunay users   11 2009-03-01 21:23 unLienSymbolique -> unFichier
drwxr-x--- 139 michaellaunay users  12K 2009-04-30 09:12 unSousRep
drwx------   2 michaellaunay michaellaunay  16K 2009-03-01 21:21 lost+found
-rw-r-----   1 michaellaunay amis  32K 2009-04-02 11:35 unFichier
michaellaunay@luciole:~/Documents/ecreall/Cours/CoursGNULinux$ ls -l /bin/mount
-rwsr-xr-x 1 root root 98440 2008-09-25 15:08 /bin/mount
michaellaunay@luciole:~/Documents/ecreall/Cours/CoursGNULinux$ ls -l
drwxrwxrwt  19 root root  4096 2009-05-03 11:10 tmp
```


*lrwxrwxrwx 1 michaellaunay users 11 2009-03-01 21:23* est la liste des
attributs qui doit être décomposée comme ceci :

première lettre :

      l indique que le fichier est un lien symbolique (un raccourci).
      d indique que le fichier est un répertoire
      - indique que le fichier est un fichier ordinaire
      c périphérique de type caractère
      b périphérique de type bloc
      s socket
      p fifo

premier groupe de 3 lettres :

`r--` indique que le propriétaire a le droit de lecture
`-w-` indique que le propriétaire a le droit d'écriture
`--x` indique soit que le propriétaire a le droit d'exécuter si le fichier est ordinaire soit que le propriétaire a le droit de traverser si le fichier est un répertoire
`--s` (SUID) indique qu'un utilisateur qui exécute le fichier usurpe les droits du propriétaire pour tous les accès effectués par l'exécutable.
  Le propriétaire a les droits d'exécuter ou de traverser (--x est positionné, mais est caché).
`--S`(SUID) indique qu'un utilisateur qui exécute le fichier usurpe les droits du propriétaire.
  Le propriétaire n'a pas les droits d'exécuter ou de traverser (--x n'est pas positionné).

second groupe de 3 lettres :

Même signification que précédemment, mais pour les groupes et sauf pour le SUID.
`--s` (SGID) indique qu'un utilisateur appartenant au groupe qui exécute le fichier usurpe les droits du groupe et que le groupe a les droits d'exécution.
`--S` (SGID) indique qu'un utilisateur appartenant au groupe qui exécute le fichier usurpe les droits du groupe, mais que le groupe n'a pas les droits d'exécuter ou de traverser.

troisième groupe de 3 lettres :

même signification que précédemment mais pour tous les autres utilisateurs et sauf SGID
`--t` (Sticky bit) Indique que les utilisateurs ont le droit de modifier le contenu du fichier ou du répertoire, mais pas de le supprimer.
    Les utilisateurs ont le droit d'exécution ou de traverser.
`--T` (Sticky bit) Idem mais les utilisateurs n'ont pas le droit d'exécuter ou de traverser.

Le fichier unFichier a pour propriétaire *michaellaunay* (owner) et pour groupe *amis* (owning group).

Les notions de permission et de groupe seront détaillées ci-après.

La taille du fichier unFichier est de 32ko.

La date est celle de dernière modification. La date du dernier accès est accessible avec la commande `ls -u -l`.

Les permissions d'un lien ne sont pas utilisées, car ceux sont celles de la cible qui sont vérifiées.

Si les permissions sont suivies d'un + alors des ACL sont positionnées.

## Les types de fichiers

Outre les fichiers normaux, les répertoires et les liens, il existe de nombreux fichiers spéciaux sous Unix.

En effet la philosophie d'Unix est de vouloir que tout soit fichier :

- Les périphériques sont manipulés comme s'ils étaient des fichiers.
- Les piles (fifo, lifo), les pipes nommées, sockets sont manipulés comme des fichiers.
- Les caractéristiques du système sont traduites à travers une arborescence.
 - Le noyau lui-même est adressé à travers une arborescence qui permet de connaître son état et de le modifier.
-  Les processus sont eux-mêmes manipulés à travers une arborescence de fichiers.

## /dev

Contient les fichiers de périphériques physiques ou virtuels :

    /dev/sda    # Premier disk scsi ou sata ou usb
    /dev/sda1   # Première partition de /dev/sda
    /dev/sdb    # Second périphérique scsi ou sata ou usb
    /dev/cdrom  # Lien vers le périphérique gérant le cdrom
    /dev/null   # Utile pour se débarrasser du contenu d'un flux
    /dev/zero   # Générateur d'octet nul
    /dev/random # Générateur aléatoire

Exemple:
```bash
find /usr -name "*.pdf" 2> /dev/null
```

    /usr/share/doc/shared-mime-info/shared-mime-info-spec.pdf
    /usr/share/example-content/case_ubuntu_johnshopkins_v2.pdf
    /usr/share/example-content/case_howard_county_library.pdf
    /usr/share/example-content/case_oxford_archaeology.pdf
    /usr/share/example-content/case_ubuntu_locatrix_v1.pdf
    /usr/share/example-content/case_Skegness.pdf
    /usr/share/example-content/case_Contact.pdf
    /usr/share/example-content/case_OaklandUniversity.pdf
    /usr/share/example-content/case_KRUU.pdf
    /usr/share/example-content/case_Wellcome.pdf
    /usr/share/evolution/2.24/help/quickref/fr/quickref.pdf
    /usr/share/gnome/help/system-admin-guide/C/system-admin-guide.pdf
    /usr/share/gnome/help/gnome-access-guide/C/gnome-access-guide.pdf
    /usr/share/gnome/help/user-guide/C/user-guide.pdf

Dans ce cas tous les messages d'erreur ont été envoyés à la poubelle.

## /sys

**sysfs** est une arborescence virtuelle résidant en mémoire qui exporte des informations sur les périphériques.

Cette arborescence offre plusieurs types de classement, une même information peut donc être trouvée de différente manière.

Les commandes telles que **lsusb** ou **lspci** vont chercher les informations dont elles ont besoin dans cette arborescence.

**/sys/class/** montre les périphériques regroupés en classes :
```bash
ls /sys/class/
```

```
atm        firmware       ieee1394_protocol  pci_bus        scsi_disk     usb_host
backlight  gpio           ieee80211          pcmcia_socket  scsi_generic  vc
bdi        graphics       input              power_supply   scsi_host     video_output
block      hidraw         leds               ppdev          sound         vtconsole
bluetooth  hwmon          mem                printer        spi_master
dma        ieee1394       misc               rfkill         thermal
dmi        ieee1394_host  mmc_host           rtc            tty
drm        ieee1394_node  net                scsi_device    usb_endpoint
```

```bash
cat /sys/class/thermal/cooling_device0/type
```

```
Processor
```

```bash
cat /sys/class/thermal/cooling_device0/cur_state
```

```
0
```

## /proc

La philosophie Unix est que "tout est fichier". Cette approche offre une grande flexibilité et une cohérence dans la gestion des ressources du système, qu'il s'agisse de périphériques matériels, de sockets réseau ou de processus en cours d'exécution.
Cette abstraction permet aux développeurs de traiter différents types de ressources avec une API cohérente, rendant le système plus facile à comprendre et à programmer.

**`procfs`** est l'arborescence virtuelle résidant en mémoire qui exporte des informations sur le noyau.

C'est dans cette arborescence que des commandes comme **ps** vont chercher des informations sur les processus.

### Avantages

1. **Uniformité**: L'API est constante pour tous les types de fichiers, ce qui facilite le développement.
2. **Simplicité**: Les opérations sur les fichiers sont simples et bien comprises, ce qui réduit la complexité du système.
3. **Composabilité**: Les fichiers peuvent être utilisés en conjonction avec une variété d'outils de ligne de commande et de bibliothèques.

### Le système de fichiers /proc

Le répertoire `/proc` est un système de fichiers spécial qui agit comme une interface vers les structures de données du noyau. Il ne contient pas de fichiers au sens traditionnel, mais plutôt une représentation en mode texte des états internes du système.

### Structure de /proc

Chaque processus en cours d'exécution sur le système a un répertoire correspondant dans `/proc`, nommé d'après son identifiant de processus (PID). À l'intérieur de ce répertoire, nous trouverons divers fichiers et sous-répertoires contenant des informations sur le processus.

Exemple:

```bash
/proc/
|-- [PID]/        # Répertoire pour le processus avec PID donné
|   |-- cmdline   # La ligne de commande qui a lancé le processus
|   |-- environ   # Les variables d'environnement pour ce processus
|   |-- mem       # La mémoire virtuelle du processus
|   |-- status    # Diverses informations sur le statut du processus
|-- cpuinfo       # Informations sur les processeurs CPU
|-- meminfo       # Informations sur la mémoire
|-- ...
```

Exemple :
```bash
cat /proc/cpuinfo
```

```
processor : 0
vendor_id : GenuineIntel
cpu family    : 6
model     : 15
model name    : Intel(R) Core(TM)2 Duo CPU     L7500  @ 1.60GHz
stepping  : 11
cpu MHz       : 800.000
cache size    : 4096 KB
physical id   : 0
siblings  : 2
core id       : 0
cpu cores : 2
apicid        : 0
initial apicid    : 0
fpu       : yes
fpu_exception : yes
cpuid level   : 10
wp        : yes
flags     : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi
		  mmx fxsr sse sse2 ss ht tm pbe syscall nx lm constant_tsc arch_perfmon pebs bts rep_good
		  nopl pni monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr lahf_lm ida
bogomips  : 3191.95
clflush size  : 64
cache_alignment   : 64
address sizes : 36 bits physical, 48 bits virtual
power management:
```

`/proc` permet en tant que root et selon l'état du processus observé d'analyser ses ressources et sa mémoire.

Ainsi il est possible de récupérer le contenu de la mémoire du processus arrêté.
## Accès à la mémoire et aux variables d'exécution d'un processus

Il est à noter que l'accès à ces informations sensibles est restreint au superutilisateur `root`. Voici comment on peut lire la mémoire d'un processus:

### Exemple: Lecture de la mémoire d'un processus

1. Tout d'abord, localisons le PID du processus cible. Par exemple, pour un processus nommé `example_process` :

    ```bash
    pidof example_process
    ```

    Supposons que le PID soit `1234`.

2. Ensuite, utilisons `cat` ou un éditeur hexadécimal pour lire le fichier `mem` du processus. Nous devons être `root` pour faire cela.

    ```bash
    sudo cat /proc/1234/mem
    ```

    ou

    ```bash
    sudo xxd /proc/1234/mem
    ```

3. Pour lire les variables d'environnement du processus :

    ```bash
    sudo cat /proc/1234/environ
    ```

Ces commandes donneront accès à des informations très détaillées sur le processus. À utiliser avec précaution et responsabilité.
### Liens

Voir
<https://unix.stackexchange.com/questions/6301/how-do-i-read-from-proc-pid-mem-under-linux>
et
<https://unix.stackexchange.com/questions/6267/how-to-re-load-all-running-applications-from-swap-space-into-ram/6271#6271>
## Enchaînement et parallélisation des commandes

Toute commande doit être vue comme une boîte noire ayant une entrée standard (`stdin`), une sortie standard (`stdout`) et une sortie d'erreur standard qui permet aussi d'afficher des informations (`stderr`).

Par défaut l'entrée standard est la saisie clavier et les sorties sont l'écran.

## Les flux standards

Les flux standards `stdin`, `stdout` et `stderr` sont numérotés respectivement 0, 1 et 2.

En conséquence nous pouvons utiliser ces numéros pour les désigner lors des redirections.

### Input, Output

La notion d'input (entrée) et d'output (sortie) est relative à la commande, ainsi dans un pipe entre deux commandes l'entrée de la seconde commande et en fait la sortie de la première. Le système crée un flux entre les deux commandes nourri par la première et consommé par la seconde.

## Les redirections

Les redirections vont permettre d'indiquer que faire des entrées et sorties standards.

Les redirections de fichier :

    >, 1>  # Stocke la sortie standard dans un fichier
    2>     # Stocke la sortie des erreurs dans un fichier
    &>     # Stocke les sorties dans un seul fichier
    >&     # Idem
    >>     # Concatène la sortie standard à la fin d'un fichier
    <      # Utilise un fichier en entrée
    |      # pipe, décrit ci-après

## Les pipes

Le pipe permet d'enchaîner les commandes, l'entrée d'une commande est alors le résultat de la commande précédente.

L'intérêt est de pouvoir créer des comportements complexes à partir de commandes simples. Cette association peut à son tour être manipulée comme une boite noire et être insérée dans un pipe plus complexe.

Exemple:

```bash
netstat -anp |grep 'tcp\|udp' | awk '{print $5}' | sed s/::ffff:// | cut -d: -f1 | sort | uniq -c | sort -n
```

### alias/unalias

La commande intégrée alias permet de redéfinir des commandes :
```bash
alias rm="echo 'ça va couper' && rm"
```

La commande **unalias** supprime les alias.

### screen

La commande **screen** est un multiplexeur de terminaux il permet de gérer plusieurs shell et de ce déconnecter sans tuer les shell dont les commandes ne sont pas encore finies.

L'intérêt est de pouvoir réaliser des tâches d'administration longues sans devoir rester connecté, ou si le réseau n'est pas fiable de ne pas perdre le travail accompli en reprenant là ou la connexion s'est rompue.

Les options de bases :

```bash
screen -dmS Nom
screen -r Nom # Permet de se rattacher au terminal Nom
```

Pour se détacher `Crtl+a` `Ctrl+d`
Pour un nouveau `Ctrl-a` `Ctrl-c`
Pour passer de l'un à l'autre : `Ctrl-a` `Ctrl-n`
Voir `man screen`

La commande **`screen`** est très utilisée avec `ssh`, elle permet de conserver le **`tty`** ouvert lors des déconnexions et donc de reprendre là où nous en étions. Il suffit de la relancer avec l'option `-r` pour rattacher une session précédente, de même en début de session nous pouvons faire `Ctrl+a` `esc` pour enregistrer les lignes et donc avoir la scroll bar.

## ssh

La commande **ssh** permet de se connecter à distance sur une machine Unix ceci de façon chiffrée. Elle permet aussi d'ouvrir des tunnels chiffrés.

L'ouverture d'un tunnel entre 2 machines est de la forme :
```bash
ssh -L ${PORT_SOURCE}:${nom_machine_dest}:${PORT_DEST} ${USER}@${DEST}
```

où \${PORT\_SOURCE} est le numéro de port d'entrée du tunnel sur la machine où nous sommes, \${nom\_machine\_dest} est soit localhost soit le nom de la machine destination soit une adresse du réseau privé derrière le serveur destination, \${PORT\_DEST} est le numéro du port de sortie du tunnel sur la machine cible \${USER} est le nom d'utilisateur.
\${DEST} est le nom complet du serveur de destination.

Nous pouvons ajouter l'option -i avec un nom de fichier clé à utiliser

Exemple :
```bash
ssh -l 9880:localhost:80 michaellaunay@plateforme.test.com
```

Me permet d'ouvrir un tunnel entre ma machine et le serveur plateforme en utilisant mon  compte michaellaunay.

Une fois mon mot de passe ou ma clé acceptée je me retrouve sur la machine distante et un tunnel est ouvert entre ma machine locale et plateforme.

Si j'ouvre un navigateur sur ma machine et que je mets comme adresse <http://localhost:9880>, la communication est chiffrée et envoyée sur plateforme où elle ressort sur le port 80 ce qui me permet d'accéder au serveur web de plateforme1 sans que quiconque ne sache ce que je fais.

Compréhension de ssh :

> -   <http://fr.wikipedia.org/wiki/Ssh>
> -   <http://web.archive.org/web/20110907084212/http://www.unixgarden.com/index.php/administration-systeme/principes-et-utilisation-de-ssh>
> -   <https://youtu.be/pLJC96zfwrE>

Si la clé d'une machine à laquelle nous nous connectons habituellement a changé (cas d'une réinstallation), nous pouvons être amené à supprimer son entrée dans le fichier *\~/.ssh/known\_hosts*.

Le plus simple est alors d'utiliser la commande **`ssh-keygen -R NomDeLaMachineDistante`**.

L'installation du deamon **`apt-get install ssh`**

Pour sécuriser les connexions **ssh**, il faut éditer
*/etc/ssh/sshd\_config* et mettre l'option *PermitRootLogin=no* et
ajouter en fin de fichier *AllowUsers idUtilisateurAutorise*.

Nous pouvons aussi limiter les adresses pouvant se connecter via le paramètre
*ListenAddress* et les ports avec *PermitOpen host:port*.

Il est possible de créer des sections de configuration par utilisateur :
```
Match User michaellaunay
X11Forwarding yes
MAtch All
```
Pour ne permettre la connexion que par clé nous positionnons
\"ChallengeResponseAuthentication no\"

Voir vidéo <https://youtu.be/qrS1rSFb-1w>

Créer une clé:

```bash
ssh-keygen
```
L'option -i permet de préciser le nom du fichier de destination

Ajouter sa clé public à un serveur distant :
```bash
ssh-copy-id user@Serveur_Distant
```
L'option -i permet de préciser le nom du fichier clé à utiliser

Pour limiter les connexions par clé à seulement certaines adresses, éditer le fichier \~/.ssh/authorized\_keys ajouter devant la clé 'from=\"192.168.0.1\" '

Supprimer la clé d'un serveur distant :

```
ssh-keygen -R NomServeurDistant
```

Nous pouvons utiliser **`tar`** et **`ssh`** pour faire des archives à travers un flux sécurisé :

```bash
tar cf - RepertoireSource | ssh user@ServeurSauvegarde "cat > nom_archive.tar"
```


La restauration se fera alors comme suit :

```bash
ssh user@ServeurSauvegarde "cat nom_archive.tar" | tar xf
```


Utiliser un Agent ssh

Saisir à chaque fois sa clé ou son mot de passe peut être fastidieux. Nous avons alors la possibilité d'utiliser un agent ssh.

Vérifiez qu'il est déjà en train de tourner :

```bash
ps -p $SSH_AGENT_PID # s'il fonctionne la variable d'environnement contient son PID
```

Le lancer sinon :

```bash
eval `ssh-agent`
```

Pour ajouter des clés :

```bash
ssh-add
```

Pour se connecter et continuer à utiliser les clés de l'agent sur la destination (forward agent): :

```bash
ssh -A ...
```

Pour rebondir (embarque l'agent sur les dernières versions de ssh) :

```bash
ssh -J 192.168.0.3,192.168.0.1 usedest@destination.ecreall.com
```
Enchaîne les rebonds sur les adresses séparées par la virgule.

Nous pouvons aussi paramétrer des rebonds en éditant `~/.ssh/config` :

```
Host machine_intermediaire_ou_alias
Hostname adresse_ip_ou_nom
User nom_utilisateur
IdentityFile chemin_vers_cle_intermediaire
Host serveur_dest_alias
Hostname adresse_ip_ou_nom
User nom_utilisateur
ProxyJump machine_intermediaire_ou_alias
```
Voir [XAVKI](https://youtu.be/vpbD7xA2wac)

Créer un tunnel entre deux machines en tâche de fond :

```bash
ssh -fNL port_local_sortant:adresse_rebond:port_entrant_distant user@Serveur_Distant
```
`-f` pour mettre en fond `-N` pour ne pas exécuter de commande.

# iptables et ufw

La commande **iptables** permet de consulter et modifier les règles du firewall.

Le service **ufw** est un \"firewall\" pré-configurer que nous pouvons facilement compléter.

Pour l'installer il suffit de faire :

```bash
apt install ufw
```

Pour connaître la liste des applications pouvant être autorisées par ufw à passer le firewall :

```bash
ufw app list
```

Applications disponibles :
```
Apache
Apache Full
Apache Secure
Bind9
Dovecot IMAP
Dovecot Secure IMAP
OpenLDAP LDAP
OpenLDAP LDAPS
OpenSSH
Postfix
Postfix SMTPS
Postfix Submission
```

Nous pourrons alors : soit autoriser les ports manuellement, soit autoriser les ports utilisés par une application.

```bash
ufw allow OpenSSH
```

Modification du firewall pour permettre en entrée http, https, smtp :

```bash
vim /etc/ufw/ufw.conf
```
Et mettre `ENABLED=yes` si elle n'est pas déjà positionnée.

```bash
ufw allow ssh # 22/tcp Ouvre le port ssh à tous (que nous pouvons restreindre à certaines adresses)
ufw allow http # 80/tcp Ouverture de http
ufw allow https # 443/tcp Ouverture de https
ufw allow smtp # 25/tcp Ouverture de smtp (envoi des courriels)
ufw enable # Rend actif ufw
```

Ces commandes permettent aussi de gérer ipv6

Vérification :

```bash
ufw status
```

```
État : actif

Vers                       Action      De
----                       ------      --
22/tcp                     ALLOW       Anywhere                  
25/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
25/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6) 
```
Permettre le lancement au démarrrage:

```bash
systemctl enable ufw
```

# Un peu de configuration pour l'utilisation en ligne

Beaucoup de paramètre par défaut peuvent être modifier dans /etc

## Configurer vim and bash

Dé-commenter `syntax on` dans `/etc/vim/vimrc`, ce qui permet d'avoir la coloration syntaxique.

Pour avoir la recherche dans l'historique des commandes en saisissant les premières lettres de la commande éditer `/etc/inputrc` et supprimer le guillemet de tête:
```
"\e[5~": history-search-backward
"\e[6~": history-search-forward
```

Pour faire de vim l'éditeur par défaut:
```bash
echo "export EDITOR=vim" > /etc/profile.d/editor.sh
```

Pour augmenter le nombre de ligne dans l'historique des commandes, créer `/etc/profile.d/history.sh` en mettant :
```
# https://wiki.ubuntu.com/Spec/EnhancedBash
shopt -s histappend
PROMPT_COMMAND="history -a; $PROMPT_COMMAND"
export HISTSIZE=1000
export HISTFILESIZE=1000
export GREP_OPTIONS='--color=auto'
```

# Gestion des permissions et droits d'accès

## Concepts

Tous les utilisateurs ont un compte qui permet de les identifier.

Les programmes fonctionnant en tâche de fond (services) sont lancés depuis des utilisateurs créés spécialement pour eux. Ainsi, le serveur html **apache** est lancé depuis le compte **www-data**.

Les utilisateurs peuvent appartenir à des groupes ce qui permet de donner des droits à un ensemble d'utilisateurs très facilement.

Tout fichier appartient à un utilisateur et à un groupe.

La gestion des droits d'accès et d'exécution se résume alors à gérer les types d'accès en fonction du propriétaire, du groupe, et du reste des utilisateurs.

Comme vu précédemment la commande **ls -l** permet d'afficher les attributs d'un fichier et donc ses permissions.

À la création d'un fichier, les droits sont automatiquement positionnés en fonction de la valeur par défaut du système et de **umask** :

```bash
umask
```
    
```
0022
```

```bash
touch test1 #Permet de créer un le fichier test1 s'il n'existe pas ou de mettre à jour sa date de modification à maintenant.
ls -lh test1
```
Nous voyons alors 
```
-rw-r--r-- 1 michaellaunay michaellaunay 0 avril  5 12:17 
```

```bash
umask 027
touch test2
ls -lh test2
```

```
-rw-r----- 1 michaellaunay michaellaunay 0 avril  5 12:18 test2
```
Le propriétaire est alors le créateur, et le groupe est généralement le groupe par défaut de l'utilisateur sauf dans le cas ou le répertoire porte le SGID alors le groupe est celui du répertoire.

## Changer le propriétaire ou le groupe propriétaire

La commande **`chown`** permet de changer le propriétaire et le groupe d'un fichier :
```bash
ls -l /tmp/MonFichier
```
Affiche
```
-rw-rw-rw- 1 michaellaunay michaellaunay 0 2009-05-03 19:08 /tmp/MonFichier
```
Pour changer le propriétaire :
```bash
chown root:users /tmp/MonFichier
ls -l /tmp/MonFichier
```
Ce qui affiche
```
-rw-rw-rw- 1 root users 0 2009-05-03 19:08 /tmp/MonFichier
```

Toutefois pour des raisons de sécurité (gestion des quotas : attaque sushi) la commande peut être réservée au super utilisateur.

Nous disposons aussi de la commande **`chgrp`** qui permet de changer le groupe d'un fichier.

## Valeurs symboliques et octales des permissions

Les tableaux suivants donnent les équivalents symboliques octaux des permissions.

  ---------------------- ------------ --------
  DROIT                  LETTRE       VALEUR

  Lecture                r (read)     4

  Écriture               w (write)    2

  Exécution / Traversé   x (execute)  1
  ---------------------- ------------ --------

Ainsi les permissions *rwx* sont équivalentes à *7* et *rwxr-xr\--* donne *754*.

  ------------ ---------------------------- --------
  DROIT        LETTRE                       VALEUR

  SUID         s si le propriétaire a *x* S 4
               si non                       

  SGID         s si le groupe a *x* S sinon 2

  Sticky Bit   t si les autres ont *x* T    1
               sinon                        
  ------------ ---------------------------- --------

Ainsi *rwsr-sr-t* est équivalent à *7755*.

Si nous avons un S ou un T en majuscule, cela signifie que les droits d'exécution n'ont pas été positionnés.

Ceci n'a pas de sens dans le cas général et indique une suppression du droit d'exécution avec oubli du SUID ou GUID ou Sticky Bit.

Sauf avec l'usage des ACLs, où un utilisateur particulier peut avoir le droit d'exécution et redonne du sens à S ou T.

### Changer les permissions sur les fichiers

La commande **chmod** permet de modifier les droits des fichiers.

#### Mode chiffré

Exemple :

```bash
ls -l MonFichier
```

```
-rw-r--r-- 1 michaellaunay michaellaunay 0 2009-05-03 19:40 MonFichier
```

```bash
chmod 754 MonFichier
ls -l MonFichier  
```

```
-rwxr-xr-- 1 michaellaunay michaellaunay 0 2009-05-03 19:40 MonFichier
```

#### Notation relative (aux droits existants)

Exemple :
```bash
ls -l MonFichier
```

```
-rwxr-xr-- 1 michaellaunay michaellaunay 0 2009-05-03 19:40 MonFichier
```

```
chmod u+s,g-x,o-r MonFichier
ls -l MonFichier
```

```
-rwsr----- 1 michaellaunay michaellaunay 0 2009-05-03 19:40 MonFichier
```

Attention aux modifications contradictoires :

```bash
echo coucou > /tmp/hello
ls -l /tmp/hello
```

```
-rw-r--r-- 1 michaellaunay michaellaunay 7 2009-05-07 09:45 /tmp/hello
```

```bash
sudo chmod u-w,o+w /tmp/hello
ls -l /tmp/hello
```

```
-r--r--rw- 1 michaellaunay michaellaunay 7 2009-05-07 09:45 /tmp/hello
```

```bash
echo bonjour >> /tmp/hello
```

```
bash: /tmp/hello: Permission non accordée
```


### Notation absolue

Exemple :

```bash
ls -l MonFichier
``` 

```bash
-rwsr----- 1 michaellaunay michaellaunay 0 2009-05-03 19:40 MonFichier
```

```bash
chmod u=rx,g=rx,o=rx MonFichier
ls -l MonFichier
```

```
-r-xr-xr-x 1 michaellaunay michaellaunay 0 2009-05-03 19:40 MonFichier
```

### Umask

Par défaut le système applique les droits 0666 pour un fichier et 0777 pour les répertoires auxquels il applique encore le filtre **umask** qui par défaut vaut 0022, les droits sont alors 0644 (rw-r\--r\--) pour un fichier et 0755 (rwx-rx-rx) pour un répertoire.

Il est possible de changer la valeur du masque de permissions en appelant **umask nouvellevaleur**.

# ACL

Le mécanisme de gestion des droits Unix couvre 95% des usages.

Il reste donc certains cas non couverts comme le fait d'attribuer les droits de modification d'un fichier à un utilisateur sans avoir à demander à l'administrateur de devoir créer un groupe (ce qui manque un peu de souplesse).

Nous pouvons aussi vouloir associer de nouveaux attributs aux fichiers pour par exemple gérer des informations de sécurités.

À l'inverse il est très difficile de restreindre les droits d'un utilisateur d'un groupe donné pour un seul fichier.

C'est pour répondre ce besoin qu'ont été implémentées les Access Control List

Les ACLs reposent sur le mécanisme des attributs étendus.

Pour les rendre disponibles, il faut que la partition soit montée avec les options *acl* et *user\_xattr* (modifier en conséquence */etc/fstab*).

Les fonctions d'accès aux *acl* sont **getfacl**, **setfacl**, **getfattr**, **setfattr**.

Voir aussi les man pages de *acl* et *attr(5)*.

## Attributs étendus

Les attributs étendus permettent de gérer simplement les métadonnées associées à un fichier.

Ce sont ces attributs étendus qui recevront les informations liées aux ACLs.

Pour installer le paquet : **`apt install attr`**

Ajouter l'option `user_xattr` aux partitions dans `*/etc/fstab*`.

Puis utiliser **`setfattr`** pour positionner les attributs et **`getfattr`** pour les afficher :
```bash
echo test > MonFichier
setfattr -n user.description -v 'Contient des données de test' MonFichier
ls -l MonFichier
```

```
-rw-r--r-- 1 michaellaunay michaellaunay 5 2009-05-05 08:17 MonFichier
```

```bash
getfattr -d MonFichier
```

```
#file: MonFichier
user.description="Contient des donn\305\251es de test"
```

Remarque : La présence d'attributs étendus n'est pas signalée par *ls*.

## Affectation des ACL

Pour vérifier que les ACLs peuvent être activées :
```bash
grep -i acl /boot/config-`uname -r`
```

```
CONFIG_EXT2_FS_POSIX_ACL=y
CONFIG_EXT3_FS_POSIX_ACL=y
CONFIG_EXT4DEV_FS_POSIX_ACL=y
CONFIG_FS_POSIX_ACL=y
CONFIG_GENERIC_ACL=y
CONFIG_JFS_POSIX_ACL=y
CONFIG_NFSD_V2_ACL=y
CONFIG_NFSD_V3_ACL=y
CONFIG_NFS_ACL_SUPPORT=m
CONFIG_NFS_V3_ACL=y
CONFIG_REISERFS_FS_POSIX_ACL=y
CONFIG_TMPFS_POSIX_ACL=y
CONFIG_XFS_POSIX_ACL=y
```

Pour installer les ACL si besoin *apt-get install acl*.

Puis rendre la partition compatible avec les ACL (édition de fstab).

Exemple de changement de permissions :

```bash
sudo -i
mkdir /tmp/MYDIR
chacl u::rwx,u:michaellaunay:rwx,g::---,o::---,m::rwx /tmp/MYDIR
ls -l /tmp
```

```
drwx------+ 2 root     root    4096 2009-05-04 22:37 MYDIR
```

```bash
su - michaellaunay
touch /tmp/MYDIR/MonFichier
ls -l /tmp/MYDIR/
```
    
```
-rw-r--r-- 1 michaellaunay michaellaunay 0 2009-05-04 22:50 /tmp/MYDIR/
```

```bash
setfacl -m isabelle:r /tmp/MYDIR/MonFichier
setfacl -m g:users:- /tmp/MYDIR/MonFichier
getfacl /tmp/MYDIR/MonFichier
```

```
getfacl: Removing leading '/' from absolute path names
# file: tmp/MYDIR/MonFichier
# owner: michaellaunay
# group: michaellaunay
user::rw-
user:isabelle:r--
group::r--
group:users:---
mask::r--
other::r--
```

# Les processus

## Définition

Un processus est l'instance d'un programme en cours de fonctionnement.

Une application est constituée de un à plusieurs processus qui collaborent à la réalisation du travail demandé.

Chaque processus s'exécute en parallèle des autres.

Un processus correspond à un fichier exécutable.

Les processus utilisent des bibliothèques qui peuvent être statiques ou dynamiques selon qu'elles sont dans le code de l'application ou non.

L'extension des bibliothèques dynamiques est *.so* (shared object).

Un processus est lancé par un autre processus, ainsi il existe une relation père-fils entre les processus.

Le processus ancêtre de tous les autres est *init* qui est lancé lors du démarrage par le noyau.

Son *PID* est 1.

**`Alt+F2`** est un raccourci clavier permettant d'appeler le lanceur.

## Attributs d'un processus

`PID` : Identifiant du processus (Process Identification),

`PPID` : Identifiant du processus père (Parent Process Identification),

`PGID` : Identifiant du groupe de processus qui permet de connaître l'application à laquelle appartient le processus,

`UID` : Le compte utilisateur ayant lancé le processus,

`GIDs` : Les groupes de l'utilisateur ayant lancé le processus,

`TTY` : Terminal où a été lancé le processus,

`NICE` : Priorité appliquée pour le scheduling,

`CMD` : La commande à l'origine du processus.

## Cycle de vie d'un processus

Un processus est dans un état qui peut être \"created\", \"ready\", \"running\", \"sleeping\", \"idle\" (en attente de signal), \"Terminated\" = \"zombie\"

*Created* correspond à l'état du processus au moment de sa création lorsque les variables ne sont pas encore renseignées.

*Ready* le processus est en mémoire, les variables sont renseignées.

*Running* le processus est en cours d'exécution.

*Sleeping* le processus a été préempté.

*Idle* le processus attend un signal.

*Zombie* le processus a fini de s'exécuter, le code de retour attend sa lecture.

Voir : <http://en.wikipedia.org/wiki/Process_states>

## Les différentes sortes de processus

Nous distinguons les processus classiques des démons qui sont les services unix.

Les démons ou démons fonctionnent en arrière-plan ils ont en général pour père le processus 1.

Les démons sont lancés et arrêtés à partir des scripts contenus dans **/etc/init.d**.

## Envoi de signaux aux processus

L'envoi de signaux au processus se fait par la commande **kill** ou **pkill**.

Les processus peuvent établir entre eux une communication événementielle basée sur les signaux.

Seuls les signaux **9** **SIGKILL**, et **SIGSTOP** ne peuvent être attrapés.

## Les commandes liées à la gestion des processus

### free

La commande **`free`** affiche les ressources mémoires consommées.

### fuser

La commande **`fuser`** liste les processus accédant à un fichier.

### ldd

La commande **`ldd`** affiche la liste des bibliothèques utilisées par un exécutable.

### lsof

La commande **`lsof`** affiche les fichiers ouverts par un processus **`lsof -p PID`**.

### nice

La commande **`nice`** et **`renice`** permette de modifier la priorité d'exécution.

### pgrep

La commande **`pgrep`** recherche un processus par son nom.

### ps

La commande **`ps`** affiche les processus en cours.

### pstree

La commande **`pstree`** affiche l'arborescence des processus.

### top

La commande **`top`** affiche la liste de processus classés par consommation décroissante.

### uptime

La commande **`uptime`** affiche les informations de temps de fonctionnement, du nombre d'utilisateurs connectés, de la charge.

## Arrière-plan / Avant-plan / Détachement

Pour lancer un processus en arrière-plan, nous pouvons soit terminer la ligne de commande qui le lance avec **``&``**, soit le lancer, faire **`Ctrl+z`** puis **`bg`**.

Lors du **`Ctrl+z`** la commande **`fg`** ramène le processus au premier plan.

La commande **`jobs`** permet de lister les processus suspendus, nous pouvons alors les rattacher avec **fg num\_job**.

Les processus dont le père meure sans attendre le statut de ses enfants sont raccrochés à *init*.

## Modification des priorités

Les processus ont des priorités fixées entre -20 (la plus haute) et +19.

Le *scheduler* gère l'ordre d'exécution des processus en fonction de cette priorité.

Par défaut un processus est lancé avec la priorité +10.

Seul l'administrateur peut donner des priorités négatives aux processus.

La commande **`nice[COMMAND [ARG]]`** permet de lancer une commande en lui donnant la priorité *p* si nous passons l'option *-n p*.

La commande **`renice`** permet de modifier la priorité d'un processus.

# Planification de tâches

Sous unix deux démons sont chargés de la planification des tâches : **atd** qui permet de programmer une tâche différée et **crond** qui permet de programmer les tâches répétitives.

## La commande crontab

**`crond`** est un service qui peut être programmé grâce à la commande **`crontab`**.

**`crontab -l`** liste les commandes déjà programmées pour l'utilisateur courant.

**`crontab -e`** permet d'éditer le fichier des commandes programmées pour l'utilisateur courant.

L'éditeur utilisé par **`crontab -e`** est celui désigné par la variable *EDITOR*.

## Le fichier crontab système

Il est possible d'éditer directement le fichier `/etc/crontab` ou ceux contenus dans `/var/spool/cron/crontabs/\${USER}`

Le format du fichier est le même que lors de l'édition avec *`crontab -e`*:

  ---------- -------- ------------- -------- ------------------- ----------
  Minutes    Heures   Jour du mois  Mois     Jour de la semaine  Commande

  (0-60)     (0-24)   (1-31)        (1-12)   (0-6)(0 pour dimanche)    un script
  ---------- -------- ------------- -------- ------------------- ----------

Le joker \*\*\*\*\* permet d'indiquer que toutes les valeurs sont acceptées.

Pour les fichiers *cron* du système, une colonne *Utilisateur* s'intercale juste avant celle de la *commande*. Elle permet alors d'indiquer sous quel utilisateur doit être lancée la commande.

Exemple :

```bash
sudo crontab -l
```

```
# m h  dom mon dow   command
00 4 * * * /usr/bin/webalizer -c /etc/webalizer/www_ecreall.conf
10 4 * * * /usr/bin/webalizer -c /etc/webalizer/ssl_ecreall.conf
* * * * * /root/load.sh update
0 * * * * /root/load.sh graph > /dev/null
```

```bash
sudo cat /etc/crontab
```

```
    # /etc/crontab: system-wide crontab
    # Unlike any other crontab you don't have to run the `crontab'
    # command to install the new version when you edit this file
    # and files in /etc/cron.d. These files also have username fields,
    # that none of the other crontabs do.

    SHELL=/bin/sh
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

    # m h dom mon dow user    command
    17 *  * * *   root    cd / && run-parts --report /etc/cron.hourly
    25 6  * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
    47 6  * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
    52 6  1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
    #
```

Sur la plupart des distributions *`/etc/crontab`* lance les scripts contenus dans *`/etc/cron.hourly`*, *`/etc/cron.daily`*, *`/etc/cron.weekly`*, *`/etc/cron.monthly`*. Pour ajouter une tâche, il suffit d'ajouter un script au répertoire désiré.

Les fichiers *`/etc/cron.allow`* et *`/etc/cron.deny`* permettent s'ils existent de nommer les utilisateurs pouvant programmer des tâches.

## La commande at

**`at`** permet de lancer une commande à une heure donnée, la commande utilise le démon **`atd`**

**`atq`** permet de voir la liste des commandes en attente d'exécution.

**`atrm`** Permet de supprimer une commande programmée.

Exemple :
```bash
apt install mailutils # Pour avoir la commande mail
at 6:45; mail -s "Debout" michaellaunay@ecreall.com < reveil.msg
```

Les fichiers *`/etc/at.allow`* et *`at.deny`* permettent comme pour `cron` de lister les utilisateurs pouvant ou non lancer **`at`**.

# Les utilisateurs et les groupes

Unix est un système multi-utilisateurs.

Tout fichier est associé à un propriétaire et à un groupe.

La gestion des droits dépend des notions d'utilisateur, de propriétaire, de groupe, et des autres.

Ceci suppose :

> -   l'existence d'une base des comptes utilisateurs,
> -   l'existence d'une base des groupes d'utilisateurs,
> -   que tout fichier possède un utilisateur propriétaire et un groupe
>     propriétaire,
> -   que tout processus hérite des droits de l'utilisateur l'ayant
>     lancé et par conséquent de l'ensemble des droits des groupes de
>     l'utilisateur,
> -   que l'utilisateur *root* a tous les droits pour pouvoir gérer le
>     système.

## Qu'est qu'un utilisateur ?

Chaque utilisateur d'un système Unix est associé à un identifiant unique qui lui permet de s'authentifier et d'accéder à son compte.

Ainsi au *login* le système demande à l'utilisateur son mot de passe.

Lorsque la connexion réussit, le système associe à l'utilisateur l'**UID** (**User IDentification**) correspondant à son identifiant.

Le système associe également à l'utilisateur un **GID** (**Groupe IDentification**) qui est le groupe principal de l'utilisateur.

Ces numéros seront utilisés pour gérer les permissions des fichiers. Les commandes comme *ls* feront alors la correspondance entre les numéros et les noms d'utilisateurs et de groupes.

Un **UID** est associé à un répertoire personnel et à un shell.

L'UID 0 désigne l'utilisateur **root**

Un utilisateur peut ne pas être une personne physique, mais être l'utilisateur d'exécution d'un démon.

En conséquence, les **UID** des personnes physiques commencent généralement à partir de 1000.

## Qu'est qu'un groupe ?

Les groupes permettent de créer des ensembles d'utilisateurs afin de gérer collectivement les permissions.

Généralement la création d'un utilisateur engendre la création de son groupe principal ayant pour identifiant de groupe le même identifiant et pour **GID** la même valeur que l'**UID**.

## Gestion des comptes

### Ajouter un utilisateur

La création d'un nouvel utilisateur peut être faite à l'aide des commandes **`useradd`** ou **`adduser`** la seconde étant préférable, car interactive.

## Supprimer un utilisateur

La commande **`userdel`** permet de supprimer un utilisateur.

Avec l'option *`-r`* cette commande supprimera en plus le répertoire personnel de l'utilisateur.

## Désactiver un compte utilisateur

L'une des façons les plus propres d'interdire la connexion à un utilisateur est de lui associer le shell **`nologin`** :

```bash
sudo usermod -s /usr/sbin/nologin indesirable
```

## Changer le mot de passe d'un utilisateur

La commande **`passwd`** permet sous *root* de changer le mot de passe d'un utilisateur.

Si nous sommes un utilisateur, la commande demandera de saisir l'ancien mot de passe.

La commande **`chpasswd`** permet de scripter les changements de mots de passe.

## Afficher des informations d'un utilisateur

La commande **id** permet d'afficher les informations de l'utilisateur :

```bash
id
```

```
uid=1000(michaellaunay) gid=1000(michaellaunay) groupes=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),107(fuse),111(lpadmin),112(admin),1000(michaellaunay)
```


La commande **`groups`** permet d'afficher les informations de groupe :

```bash
groups
```

```
michaellaunay adm dialout cdrom plugdev lpadmin admin sambashare
```

La commande **`who`** permet de savoir qui est connecté sur la machine.

La commande **`whoami`** permet de savoir sous quelle identité nous sommes connecté.

## Modifier les informations d'un utilisateur

La commande **`usermod`** permet de modifier toutes les propriétés d'un utilisateur.

Faire *`man usermod`*

La commande **`chsh`** permet de changer le shell de connexion.

La commande **`chfn`** permet de changer la description d'un utilisateur.

## Changer d'identité

La commande **`su`** permet de changer d'identité. **su -** permettra en plus d'utiliser l'environnement de l'utilisateur.

La commande **`sudo`** permet d'exécuter un script ou une commande en tant que *root*. **sudo -i** permet sous *Ubuntu* de se connecter en tant que *root*.

## Gestion des groupes

### Créer un groupe

Comme pour l'utilisateur nous pouvons utiliser **`addgroup`** ou **`groupadd`**

### Afficher des informations sur les groupes

La commande **`groups`** déjà vu affiche les informations d'appartenance.

#### Ajouter un utilisateur à un groupe

Nous utilisons la commande **usermod** de la façon suivante :

```bash
sudo usermod -a -G cdrom,dev michaellaunay
```

#### Ajouter un utilisateur au groupe des administrateurs

usermod -a -G sudo identifiant\_de\_l\_utilisateur

#### Changer de groupe principal

La commande **newgrp** permet de changer de groupe principal.

#### Modifier un groupe

Pour modifier un groupe nous pouvons utiliser la commande **groupmod**.

#### Supprimer un groupe

La commande **groupdel** permet de supprimer un groupe.

#### Le fichier /etc/passwd

Le fichier */etc/passwd* contient la définition de tous les comptes.

#### Le fichier /etc/shadow

Le fichier */etc/shadow* contient les mots de passe des comptes définis dans */etc/passwd*

#### Le fichier /etc/group

Le fichier */etc/group* contient la définition de tous les groupes.

# LDAP
[[LDAP]]
# Versionner les changements de configuration
Modifier les fichiers de configuration du répertoire **/etc** peut vite conduire a des erreurs.
Le mieux est de versionner les mofications et d'utiliser `etckeeper`
Il s'agit d'une collection de outils destinés à maintenir tout le répertoire /etc (où sont généralement stockés les fichiers de configuration) sous un système de contrôle de versions, comme Git ou Mercurial.

Il va nous aider à savoir qui a fait quoi et quand et à revenir à une version précédente de la configuration si nécessaire. `etckeeper` peut automatiquement effectuer des commits lors de l'installation de nouveaux paquets, et il peut aussi être utilisé manuellement pour suivre les changements.

Nous pouvons utiliser la commande suivante :
```bash
sudo apt-get install etckeeper
```

Une fois installé, il faut initialiser le répertoire /etc avec la commande suivante :
```bash
sudo etckeeper init
```

Et finalement, faire un premier commit :
```bash
sudo etckeeper commit "Initial commit"
```

Maintenant, chaque fois que nous apportons des modifications à un fichier dans /etc, nous pouvons faire un *commit* avec `etckeeper commit "<message>"`. Et chaque fois que nous installerons ou mettrons à jour des paquets avec apt, etckeeper enregistrera automatiquement les modifications.

# Syslog
**Syslog** est le système chargé d'enregistrer les fichiers journaux.

Le démon **klogd** consigne les événements de type message du noyau, authentification, connexion alors que **syslogd** enregistre les messages d'envoi ou réception de courrier, ceux d'erreur, etc.

Les fichiers de messages se trouvent dans */var/log*

**syslogd** est configuré avec le fichier */etc/syslog.conf*. Ce fichier permet d'indiquer les sources de messages et les destinations associées (fichier, tty, application ou syslog d'une autre machine).
Pour être prise en compte, la modification du fichier de conf doit être suivie par un *kill 1 \$PID\_SYSLOG*.

Toutefois de nombreux programmes n'utilisent pas *syslog* comme *CUPS*, *Samba*, etc.

Afin d'éviter que la taille des fichiers de logs n'explose la capacité du disque les fichiers sont compressés de façon régulière par **logrotate** dont la configuration est modifiable en éditant
**/etc/logrotate**

La commande **dmesg** permet d'afficher les messages du noyau.

### Modification de la configuration de logrotate

Logrotate possède une configuration par défaut contenue dans \"/etc/logrotate.conf\" puis un répertoire avec les configurations des services pour compléter ou remplacer la configuration par défaut.

Il est fréquent que pour des raisons légales, nous devons garder un ou deux ans de logs selon la nature des utilisateurs et des services. Souvent nous gardons 104 semaines de connexions et 52 semaines de navigation et 14 semaines pour les autres services.

Pour modifier la conf par défaut à 14 semaines nous éditons \"/etc/logrotate.conf\" :

> -   Remplacer \"rotate 4\" par \"rotate 14\" pour garder 3 mois de log
>     par défaut
> -   Décommenter \"compress\" pour compresser les anciens fichiers log
> -   Ajouter delaycompress chaque vieux fichier de log
> -   Limiter la taille d'un fichier de log à 100M

Nous devons donc avoir dans /etc/logrotate.conf :
```
rotate 14
compress
delaycompress
size 100M
```   

Puis nous changeons \"rotate X\" à \"rotate 104\" dans les fichiers des services concernés se trouvant dans le répertoire \"/etc/logrotate.d/\".
Par exemple, nous allons modifier \"/etc/logrotate.d/apache2\" pour mettre rotate à 104, et modifier le fichier \"/etc/logrotate.d/rsyslog\" modifier les logs des services d'authentifications.

# Périphérique disque et système de fichiers

### Les disques durs sous Linux

L'organisation physique d'un disque est constituée de plateaux superposés divisés en pistes concentriques.

Chaque piste contenant un certain nombre de secteurs de 512 octets.

L'ensemble des pistes des différents plateaux accessibles sans nouveau déplacement des têtes de lecture constitue un cylindre.

Voire <http://fr.wikipedia.org/wiki/Disque_dur>.

Le nommage des périphériques disques dépend de leur nature et de la façon dont ils sont gérés par le bios.

Ainsi sda, sdb correspondent à des disques durs alors que nvme0n1p7 correspond à un disque ssd monté sur le connecteur nvme.

Depuis 2013 les nommages des périphériques suivent les règles *ifname* voir :
<https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/7/html/networking_guide/sec-understanding_the_device_renaming_procedure>

### Les concepts

Un disque est \"découpé\" en partitions.

Le premier secteur contient le MBR (Master Boot Record) qui décrit la table des 4 premières partitions et contient également le code du chargeur primaire (primary loader).

L'une des quatre partitions primaires peut être du type étendu et contenir des partitions logiques qui sont alors chaînées entre elles.

La taille des partitions est donnée en nombre de cylindres, ce qui fixe le nombre de secteurs de la partition.

La partition possède un type qui fixe son usage, par exemple 83 pour Linux, 82 pour le swap, 5 pour une partition étendue, etc.

### Les systèmes de fichiers

Un système de fichier est une structure de données permettant de stocker et organiser les informations dans des fichiers.

Le système de fichier est généralement stocké dans une partition, mais il peut l'être sur un disque amovible (USB) ou dans un fichier.

### Partitions

La numérotation des partitions est réalisée en accolant au nom du périphérique le numéro de la partition. Ainsi :

    sda est le premier disque complet
    sda1 est la première partition
    sda4 est généralement la partition étendue
    sda5 est la première partition logique
    sdb est le second disque

### Utilitaires de partitionnement

Historiquement la commande permettant de créer les partitions était **fdisk**, mais elle est limitée à des partitions de taille inférieure à 2To.

Elle est remplacée par la commande **parted** et par sa version graphique **gparted** qui permet de créer et retailler des partitions déjà existantes, mais il faut descendre le paquet.

La commande **partprobe** permet d'avertir le système que l'nous avons modifié la table des partitions.

### Arborescence standard et organisation du FHS

Le Filesystem Hierachy Standard est l'organisation standard Unix utilisée par Linux

Tout est fichier. Les périphériques (scanner, imprimante, etc) sont manipulés sous forme de fichier dans lequel nous alons lire et écrire.

#### Arborescence de /

/ est la racine, elle a pour contenu :

    * /bin contient les exécutables du système d'exploitation,
    * /boot les fichiers de démarrage,
    * /dev les périphériques sous forme de fichiers pouvant être lus ou écrits,
    * /etc les fichier de configuration et ceux nécessaires au démarrage,
    * /home les répertoires des utilisateurs,
    * /lib le bibliothèques partagées et les modules du noyau dans le sous-répertoire modules,
    * /mnt les dossier des points de montage temporaires,
    * /proc les états du noyau,
    * /root le répertoire du super utilisateur root,
    * /sbin les exécutables du super utilisateur,
    * /sys contient les caractéristiques et informations sur les périphériques comme le nom du fabriquant, les bus connectés,
    * /tmp les fichiers temporaires liés à l'exécution des applications ou services, ils sont effacés au reboot,
    * /usr les ressources du système non essentielles (Unix Système Ressources)
    * /var les fichiers tels que les based de données, les pages html, les mails, les logs

-   Pour une description plus complète C.f. <http://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard>

#### Arborescence de /usr

**/usr** contient :

    * /usr/bin/ Binaires de l'utilisateur,
    * /usr/include/ Entêtes des bibliothèques partagées,
    * /usr/lib/ Bibliothèques partagées des logiciels utilisateurs,
    * /usr/local/ Hiérarchie pour les données de locales.
    * /usr/sbin/ Binaires pour l'administrateur,
    * /usr/share/ Fichiers indépendants de la plateforme (non binaires),
    * /usr/src/ Les sources du noyau,

#### Arborescence de /var

**/var** contient:

> -   /var/lib/ Données persistantes telles les bases de données,
> -   /var/lock Les fichiers de verrous,
> -   /var/log Les fichiers de Log
> -   /var/mail Les mails si la configuration précise qu'ils doivent être stockés ici
> -   /var/run Les informations d'exécution des daemons
> -   /var/spool Les queues de traitement (mail, impression, etc)
> -   /var/tmp Les fichiers temporaires à préserver des reboots

### Formater une partition

La commande **mkfs** permet de formater une partition. Le type système de fichier est alors choisi à ce moment ex: **mkfs.ext3**.

### Monter / Démonter une partition

Un système de fichier est accessible après son *montage* soit au démarrage, soit à l'aide de la commande **mount**.

Le démontage se fait à l'aide de la commande **umount**.

La commande **sshfs** permet de monter un disque distant à travers le protocole **ssh**. Pour démonter un système de fichier monté avec sshfs : **fusermount -u point\_de\_montage**.

### Les tables de montage : /etc/fstab

Le fichier **/etc/fstab** contient les montages à réaliser au démarrage ou pour lesquels *root* a autorisé un montage manuel.

### Tables systèmes, inodes

Un file system est composé de différentes tables système :

> -   Le super-bloc contenant les informations de taille, d'état de montage.
> -   La table des *inodes* (nœud d'index) qui fait correspondre à chaque fichier un numéro d'identification unique et qui possède les informations des droits d'accès, de propriété.
> -   Les répertoires qui sont une table de correspondance *inode* vers nom de fichier.

La commande **ls -i** permet d'afficher l'inode d'un fichier.

Remarques :

> -   Les *inodes* ne contiennent pas le nom du fichier.
> -   Un *file system* est limité en *inodes*

### Journalisation

La journalisation permet d'enregistrer les manipulations réalisées sur les fichiers et l'arborescence.

Pour certains systèmes de fichier, elle enregistre en plus les différences, ce qui permet de revenir à un état précédent.

Les systèmes simples permettent néanmoins de revenir au dernier état cohérent en cas de plantage du système.

Toutefois, les coupures de courant peuvent aboutir à des états incohérents, ceci à cause du cache en écriture des disques durs.

Au redémarrage de la machine, la journalisation va permettre d'accélérer le diagnostic des disques.

**ext3** est un système journalisé de manière simple.

### Récupération de fichiers

La journalisation sur les partitions de type ext3 et plus comme la ext4 permet de récupérer les fichiers supprimés. Pour cela il faut arrêter le système, le booter sur un système externe comme une clé usb bootable (live usb). Devenir root, installer sur le système externe extundelete :

    apt install extundelete

Repérer la partition contenant les fichiers à restaurer avec la commande lsblk. Puis appeler la commande extundelete :

    sudo extundelete /dev/sda6 --restore-directory /home/michaellaunay/Maildir/cur/ --after 1636239600

Où \--restore-directory permet de dire dans quel répertoire nous cherchons à récupérer les  fichiers. Et \--after permet de dire à partir de quelle date de suppression nous souhaitons récupérer les fichiers. La date est en seconde à partir du 1er janvier 1970 (nous pouvons utiliser datetime).

### Contrôle des systèmes de fichiers

La commande **`df`** permet de lister les montages réalisés.

La commande **du** permet de calculer la taille d'une arborescence.

La commande **`fsck`** permet de vérifier l'état d'une partition.
ATTENTION ! Il ne faut pas l'utiliser sur des partitions montées.

La commande **`e2label`** permet d'affecter un nom à un *file system*.

La commande **`hdparm`** permet avec l'option *-t* de connaître les performances d'un disque.

### Montage des périphériques amovibles

La commande **`lsusb`** permet de voir les périphériques USB connectés.

La commande **`lspci`** permet de voir les périphériques PCI connectés.

Lorsqu'un périphérique de type blocs ou caractères est détecté par le noyau, un périphérique correspondant est ajouté dans *`/dev`* par le démon **`udevd`** du système **`udev`**.

Le système **`udev`** a pour rôle de gérer l'unicité des noms pour les périphériques et de maintenir *`/dev`* en cohérence avec les périphériques présents.

Les fichiers de configuration de **`udev`** sont placés dans *`/etc/udev`*.
Il est possible de définir des règles dans *`/etc/udev/rules.d`* qui seront évaluées dans l'ordre lexicographique.

Le démon HAL (Hardware Abstraction Layer) **`hald`** est notifié par **`udev`** de l'ajout d'un périphérique (règle *`/etc/udev/rules.d/90-hal.rules`*).

**`HAL`** identifie alors le type des périphériques connectés, du système de fichiers, et en fonction des informations comme *VendorId* ou *ProductId* d'associer le contenu avec un type d'application.

La base de données des périphériques est située dans le répertoire *`/usr/share/hal/fdi/`* :
```bash
grep -rl ipod /usr/share/hal/fdi/*
```

```
/usr/share/hal/fdi/information/10freedesktop/10-usb-music-players.fdi
```


Une fois le périphérique complètement identifié, **HAL** envoie un message sur le bus de communication des applications **D-Bus**.

Les applications de l'environnement graphique vont alors monter le périphérique et le rendre accessible à l'utilisateur.

Sous **gnome** le comportement peut être modifié via le gestionnaire de fichiers **nautilus** dans Edition-Préférences-Supports\*\*.

Exemple des noms persistants donnés par udev: :
```bash
ls -lR /dev/disk
```

```
/dev/disk:
total 0
drwxr-xr-x 2 root root 260 avril 18 22:11 by-id
drwxr-xr-x 2 root root  80 avril 18 22:11 by-partlabel
drwxr-xr-x 2 root root 100 avril 18 22:11 by-partuuid
drwxr-xr-x 2 root root 160 avril 18 22:11 by-path
drwxr-xr-x 2 root root 100 avril 18 22:11 by-uuid

/dev/disk/by-id:
total 0
lrwxrwxrwx 1 root root  9 avril 18 22:11 ata-ST8000VN0022-2EL112_ZA1F9KQR -> ../../sda
lrwxrwxrwx 1 root root 10 avril 18 22:11 ata-ST8000VN0022-2EL112_ZA1F9KQR-part1 -> ../../sda1
lrwxrwxrwx 1 root root 13 avril 18 22:11 nvme-eui.0025385491b00f2f -> ../../nvme0n1
lrwxrwxrwx 1 root root 15 avril 18 22:11 nvme-eui.0025385491b00f2f-part1 -> ../../nvme0n1p1
lrwxrwxrwx 1 root root 15 avril 18 22:11 nvme-eui.0025385491b00f2f-part2 -> ../../nvme0n1p2
lrwxrwxrwx 1 root root 13 avril 18 22:11 nvme-Samsung_SSD_970_EVO_Plus_1TB_S4EWNF0M403887L -> ../../nvme0n1
lrwxrwxrwx 1 root root 15 avril 18 22:11 nvme-Samsung_SSD_970_EVO_Plus_1TB_S4EWNF0M403887L-part1 -> ../../nvme0n1p1
lrwxrwxrwx 1 root root 15 avril 18 22:11 nvme-Samsung_SSD_970_EVO_Plus_1TB_S4EWNF0M403887L-part2 -> ../../nvme0n1p2
lrwxrwxrwx 1 root root  9 avril 18 22:11 usb-Generic_STORAGE_DEVICE-0:0 -> ../../sdb
lrwxrwxrwx 1 root root  9 avril 18 22:11 wwn-0x5000c500b5c4d662 -> ../../sda
lrwxrwxrwx 1 root root 10 avril 18 22:11 wwn-0x5000c500b5c4d662-part1 -> ../../sda1

/dev/disk/by-partlabel:
total 0
lrwxrwxrwx 1 root root 15 avril 18 22:11 'EFI\x20System\x20Partition' -> ../../nvme0n1p1
lrwxrwxrwx 1 root root 10 avril 18 22:11  Sauvegardes -> ../../sda1

/dev/disk/by-partuuid:
total 0
lrwxrwxrwx 1 root root 15 avril 18 22:11 54795468-79c2-48ce-9c7e-f6bd6aea6914 -> ../../nvme0n1p2
lrwxrwxrwx 1 root root 15 avril 18 22:11 e9a9f238-ec93-4777-82e8-cbea7c8cf986 -> ../../nvme0n1p1
lrwxrwxrwx 1 root root 10 avril 18 22:11 eeb50a63-ec05-4e34-b673-b90bc5ea2cfa -> ../../sda1

/dev/disk/by-path:
total 0
lrwxrwxrwx 1 root root  9 avril 18 22:11 pci-0000:00:14.0-usb-0:1:1.0-scsi-0:0:0:0 -> ../../sda
lrwxrwxrwx 1 root root 10 avril 18 22:11 pci-0000:00:14.0-usb-0:1:1.0-scsi-0:0:0:0-part1 -> ../../sda1
lrwxrwxrwx 1 root root  9 avril 18 22:11 pci-0000:00:14.0-usb-0:5:1.0-scsi-0:0:0:0 -> ../../sdb
lrwxrwxrwx 1 root root 13 avril 18 22:11 pci-0000:03:00.0-nvme-1 -> ../../nvme0n1
lrwxrwxrwx 1 root root 15 avril 18 22:11 pci-0000:03:00.0-nvme-1-part1 -> ../../nvme0n1p1
lrwxrwxrwx 1 root root 15 avril 18 22:11 pci-0000:03:00.0-nvme-1-part2 -> ../../nvme0n1p2

/dev/disk/by-uuid:
total 0
lrwxrwxrwx 1 root root 10 avril 18 22:11 0a2768a9-34ac-4944-83a2-2e10ed4c48a5 -> ../../sda1
lrwxrwxrwx 1 root root 15 avril 18 22:11 aad5ec2d-1dce-4a2b-8b07-2cd2bef72410 -> ../../nvme0n1p2
lrwxrwxrwx 1 root root 15 avril 18 22:11 C3AC-80CA -> ../../nvme0n1p1
```
Voir :

> <https://doc.ubuntu-fr.org/udev>
> <https://web.archive.org/web/20100417131709/www.unixgarden.com/index.php/programmation/decouvertes-et-experimentation-avec-d-bus>

# Utilitaire smartd

Smart signifie **Self-Monitoring, Analysis and Reporting Technology** c'est une technologie d'auto surveillance mise en œuvre par certains disques durs.

Le but de cette technologie est de permettre le suivi de l'usure \"normale\" des disques internes.

Lorsque les disques utilisés supportent les informations d'état **smart**, il devient possible de vérifier leur état, mais aussi de lancer un démon qui nous alerte en cas de risque.

Le contrôle se fait au détriment d'une légère perte de performance.

Installation :

```bash
sudo apt-get install smartmontools
```

Le démon smartd est alors installé et peu prévenir l'administrateur par mail lorsque les informations d'état des disques atteindront les seuils d'alertes.

La vérification se fait au démarrage et est modifiable en éditant */etc/smartd.conf*.

Si les erreurs disque n'ont pu être corrigées, il faut fortement songer à changer de disque\...

Attention à l'usage de *smart* avec le *RAID* qui pose problème avec certains contrôleurs.

Pour savoir si smart est activé sur le disque :

```bash
sudo smartctl -i /dev/sda
```

```bash
smartctl version 5.38 [x86_64-unknown-linux-gnu] Copyright (C) 2002-8 Bruce Allen
Home page is http://smartmontools.sourceforge.net/


=== START OF INFORMATION SECTION ===
Device Model:     FUJITSU MHW2160BH PL
Serial Number:    K10FT7A25Y3B
Firmware Version: 0084001E
User Capacity:    160 041 885 696 bytes
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   8
ATA Standard is:  ATA-8-ACS revision 3b
Local Time is:    Sun May 24 18:33:45 2009 CEST
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

```bash
smartctl -A /dev/sda
```

```
smartctl version 5.38 [x86_64-unknown-linux-gnu] Copyright (C) 2002-8 Bruce Allen
Home page is http://smartmontools.sourceforge.net/
```
Pour l'activer si nécessaire :
```bash
sudo smartctl -s /dev/sda
```

Consultation de l'état d'un disque :
```
=== START OF READ SMART DATA SECTION ===
SMART Attributes Data Structure revision number: 16
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x000f   100   100   046    Pre-fail  Always       -       182781
  2 Throughput_Performance  0x0005   100   100   030    Pre-fail  Offline      -       39256064
  3 Spin_Up_Time            0x0003   100   100   025    Pre-fail  Always       -       1
  4 Start_Stop_Count        0x0032   099   099   000    Old_age   Always       -       603
  5 Reallocated_Sector_Ct   0x0033   100   100   024    Pre-fail  Always       -       8589934592000
  7 Seek_Error_Rate         0x000f   100   100   047    Pre-fail  Always       -       1690
  8 Seek_Time_Performance   0x0005   100   100   019    Pre-fail  Offline      -       0
  9 Power_On_Hours          0x0032   080   080   000    Old_age   Always       -       10494
 10 Spin_Retry_Count        0x0013   100   100   020    Pre-fail  Always       -       0
 12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always       -       564
192 Power-Off_Retract_Count 0x0032   100   100   000    Old_age   Always       -       9
193 Load_Cycle_Count        0x0032   079   079   000    Old_age   Always       -       422760
194 Temperature_Celsius     0x0022   100   100   000    Old_age   Always       -       42 (Lifetime Min/Max 14/55)
195 Hardware_ECC_Recovered  0x001a   100   100   000    Old_age   Always       -       166
196 Reallocated_Event_Count 0x0032   100   100   000    Old_age   Always       -       453509120
197 Current_Pending_Sector  0x0012   100   100   000    Old_age   Always       -       0
198 Offline_Uncorrectable   0x0010   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x003e   200   200   000    Old_age   Always       -       0
200 Multi_Zone_Error_Rate   0x000f   100   100   060    Pre-fail  Always       -       10052
203 Run_Out_Cancel          0x0002   100   100   000    Old_age   Always       -       3732309344292
240 Head_Flying_Hours       0x003e   200   200   000    Old_age   Always       -       0
```
Lecture du résultat :

TYPE :
	Old_age : indique qu'un dépassement n'est pas critique, nous avons simplement dépassé la valeur
			  garantie par le constructeur.
	Pre-fail : indique que tout dépassement risque de provoquer une perte du disque.

UPDATED :
	Always : la valeur est maintenue à jour.
	Offline : la valeur est calculée uniquement lors des tests.

VALUE : La valeur actuelle du disque. Une valeur comprise entre 100 et 255 indique généralement une bonne santé du disque.

WORST : La pire valeur enregistrée par le disque.

THRESH : La valeur seuil en dessous de laquelle le disque commence à souffrir.

RAW VALUE : est la conversion de VALUE dans les unités utilisées par le  constructeur.

Il existe plusieurs types de tests pour mettre à jour les valeurs :

offline : le disque ne doit pas être monté.
short : test court sur les performances, les problèmes électriques et lectures/écritures physiques.
long : version longue du précédent.

Lancement d'un test :

```bash
sudo smartctl -t long /dev/sda
```

```
smartctl version 5.38 [x86_64-unknown-linux-gnu] Copyright (C) 2002-8 Bruce Allen
Home page is http://smartmontools.sourceforge.net/

=== START OF OFFLINE IMMEDIATE AND SELF-TEST SECTION ===
Sending command: "Execute SMART Extended self-test routine immediately in off-line mode".
Drive command "Execute SMART Extended self-test routine immediately in off-line mode" successful.
Testing has begun.
Please wait 92 minutes for test to complete.
Test will complete after Sun May 24 20:18:57 2009

Use smartctl -X to abort test.
```

Pour voir le résultat il faut soit consulter les *logs* soit :

```bash
sudo smartctl -c /dev/sda
sudo smartctl -l selftest /dev/sda
sudo smartctl -A /dev/sda
```

Configuration de **smartd** :

Le man de **samrtd** donne l'ensemble des informations permettant de configurer le démon qui scrute l'état des disques.

Informations :

> -   <http://fr.wikipedia.org/wiki/Self-Monitoring,_Analysis_and_Reporting_Technology>
> -   <http://linux-attitude.fr/post/Soyez-encore-plus-a-lecoute-de-vos-disques>

# Les montages en raid

Le RAID (Redundant Array of Independant Disk) permet d'augmenter la tolérance aux pannes ou d'avoir un espace de stockage plus rapide ou plus grand que ce que nous obtiendrions avec un seul disque.

La tolérance est obtenue soit par mirroring, soit par calcul de parité.

Les performances sont obtenues par multiplexage des disques, par exemple un mot de 4 octets voit chacun de ses octets écrits en parallèle sur 4 disques différents.

La concaténation permet elle de disposer virtuellement d'un seul disque dont la capacité est la somme de chacun des disques.

Voir : <http://fr.wikipedia.org/wiki/RAID_(informatique)>

## Le RAID 0

Nous cherchons les performances sans tolérance aux pannes.

## Le RAID 1

L'information est dupliquée sur les disques qui sont donc montés en miroir.

## Le RAID 5

Il nécessite au minimum 3 disques.

Les informations sont stockées par bande (strip).

Par exemple pour un système à 3 disques, deux bandes de données et une de parité sont écrites alternativement sur les 3 disques.

Les bandes de parités sont écrites alternativement sur tous les disques façon a accroître la résistance aux pannes.

Elles sont calculées en faisant un ou exclusif des bandes de données précédentes.

## La gestion du RAID logiciel par Linux

La commande **`mdadm`** permet de gérer les disques *RAID*.

La commande **`mdadm \--create`** permet d'initialiser un disque *RAID*.

La commande **`mdadm \--detail /dev/mdX`** affiche l'état d'un *RAID*.

La commande **`mdadm \--deamonise /dev/mdX`** démarre le *RAID*.

Le fichier *`/etc/mdadm/mdadm.conf`* contient la configuration utilisée par `mdadm` au démarrage.

La commande **`mdadm \--detail \--scan \--verbose`** permet de récupérer la configuration et si nécessaire de la stocker dans *`/etc/mdadm/mdadm.conf`* pour le prochain démarrage.

Lien : <http://doc.ubuntu-fr.org/raid_logiciel>

Exemple création d'un disque :

```bash
sudo fdisk /dev/sda #Pour la création de /dev/sda1
sudo fdisk /dev/sdb #Pour la création de /dev/sdb1
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1
sudo mdadm --daemonise /dev/md0
```
 
Sinon en éditant /etc/mdadm/mdadm.conf on peut y ajouter
```
#ARRAY /dev/md0 level=raid1 num-devices=2 devices=/dev/sda1,/dev/sdb1
```

Les périphériques *RAID* apparaissent sous */dev/md\#*.

# LVM

**`LVM`** `Logical Volume Manager` s'intercale entre le noyau et les partitions des disques afin de permettre :

> -   de redimensionner les partitions,
> -   de concaténer les disques,
> -   de réaliser des instantanés du système de fichier.

Sa mise en place doit être faite dès le partitionnement (type 8E).

*`GRUB`* ne fonctionne pas avec *LVM* il faut donc soit utiliser *Lilo* soit réserver LVM à */home*.

Glossaire :

**PV** *Physical Volume* est un disque ou un ensemble de disques utilisés par *LVM*,

**VG** *Volume Group* est un ensemble de PV,

**LV** *Logical Volume* est une partie du *VG* vue par l'utilisateur comme s'il s'agissait d'une partition,

**PD** *Physical Device* est ensemble de partition d'un disque utilisé par *LVM*,

**PE** *Physical Extent* est la plus petite unité de données gérée par LVM (\~= 4Mo).

Les modules de *LVM* doivent être montés avec *modprobe dm\_load*.

La commande **vgscan** permet de lancer *LVM*.

La commande **pvcreate** permet de créer un volume physique.

La commande **vgcreate** permet de regrouper le *PV* dans un groupe de
volume.

La commande **lvcreate** permet de créer le volume logique.

Il ne faut pas oublier de formater un volume logique pour pouvoir s'en servir, nous utilisons alors *mkfs* classiquement.

La commande **lvextend** permet de modifier la taille d'un *LV*.

Pour retailler le système de fichier, nous utiliserons la commande **resize2fs** après avoir démonté le système de fichier.

# Python

Voir [[Python]]

# Initialisation lors du boot

Voir [[Initialisation système et des services]]

# Modules

Le noyau Linux est modulaire.

La gestion de nombreux périphériques n'est pas faite dans le noyau, mais dans des modules qui sont chargés à la demande.

La commande **modprobe** permet de charger un module directement par son nom.

La commande **lsmod** permet de connaître les modules déjà chargés.

La commande **rmmod** permet de supprimer un module du noyau.

Il est possible d'intégrer définitivement les modules au noyau en recompilant celui-ci.

# Configuration réseau et outils TCP/IP

### Introduction

L'unix BSD a été l'une des premières plateformes à supporter la pile protocolaire TCP/IP.

Dans cette tradition GNU/Linux regorge d'outils TCP, UDP, ICMP.

TCP signifie Transmission Control Protocol.

IP : Internet Protocol.

UDP : User Datagram Protocol.

ICMP : Internet Control Message Protocol.

Sous Ubuntu l'installation configure l'interface réseau en client DHCP (Dynamique Host Configuration Protocol).

De nombreux programmes communiquent entre eux sur la machine en utilisant l'interface réseau *loopback* de nom *localhost* et d'adresse *127.0.0.1* et ceci alors même que la connexion ne sort pas de la machine.

Liens :

> -   <http://fr.wikipedia.org/wiki/Tcp>
> -   <http://fr.wikipedia.org/wiki/Internet_Protocol>

### Configuration automatique

Le protocole DHCP permet de demander à un serveur une adresse IP ainsi que les paramètres de connexion tels que le masque de sous réseau, les DNS, la passerelle.

La commande **dhclient**, permet de relancer la négociation avec le serveur.

Infos : <http://fr.wikipedia.org/wiki/DHCP>

### La commande ip

La commande ip permet d'afficher et modifier toutes les interfaces réseau.

`ip addr` : Affiche les adresses ip et toutes les informations.
`ip addr show dev em1` : Affiche les informations pour le périphérique em1 
`ip addr add 192.168.1.1/24 dev em1` : Ajoute l'adresse 192.168.1.1 avec le
masque 24 au périphérique em1.

`ip link` : Gère et affiche toutes les interfaces réseaux.
`ip link show dev em1` : Affiche les informations pour em1.
`ip -s link` : ffiche les interfaces statiques.
`ip link set dev eno12345678 up` : Met en fonctionnement l'interface eno12345678.
`ip link set dev eno12345678 down` : Éteint l'interface eno12345678.

ip route : Affiche et permet la modification de la table de routage.

ip maddr : Affiche et permet la gestion des adresses multicast.
ip maddr show dev em1 : Affiche les informations multicast de em1

ip neigh : Affiche les objets voisins c'est-à-dire la table ARP pour IPv4.
ip neigh show dev em1 : Affiche le cache ARP de l'interface em1

La commande ip permet de consulter et changer l'état ou les paramètres
de tous les types de périphériques réseau.
Elle remplace les commandes ifconfig, iwconfig, ifup/ifdown, route que nous détaillerons ci-après, car elles sont encore proposées par certaines docs.

voir :

<http://cpham.perso.univ-pau.fr/ENSEIGNEMENT/UERHD/DescriptifCmdIP.pdf>

<https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf>

### ifconfig (déprécié)

La commande **ifconfig** permet à la fois de consulter les paramètres réseau, mais également de configurer les interfaces.
Cette commande est aujourd'hui obsolète et remplacée par **ip** que nous détaillerons
ci-après, on peut l'installer avec **apt install net-tools**.
Toutefois elle est beaucoup plus simple, mais moins complète que **ip** .

Exemple de configuration :

```bash
sudo ifconfig eth0 192.168.0.7 netmask 255.255.255.0
```

La configuration ainsi réalisée n'est pas permanent, elle sera perdue au prochain démarrage.

Pour modifier de façon permanente la configuration réseau il faut éditer */etc/network/interfaces*.

**`ifconfig`** est remplacé par la commande **ip addr** ou **ip a**
**`ifconfig eth0 192.168.0.11`** est remplacé par **`ip addr add 192.168.0.11/255.255.255.0 dev enxe4b97aef38eb`**
Les noms comme eth0 sont remplacés par la convention de nommage **ifname** pour éviter le
changement de nom lors du reboot.

## iwconfig (déprécié)

La commande **iwconfig** permet de configurer les cartes wifi.

## ifup/ifdown (déprécié)

La commande **ifup** permet de démarrer une interface réseau en fonction de la configuration indiquée dans */etc/network/interfaces* Remplacée par **ip link set NOM\_PERIPHERIQUE up**

La commande **ifdown** permet de l'arrêter. Remplacée par **ip link set NOM\_PERIPHERIQUE down**

## route (déprécié)

La commande **route** permet de consulter et de fixer l'adresse de la passerelle :

```bash
sudo route add default gw 192.168.0.1
```

Remplacée par **ip route**

En consultation elle est identique à **netstat -nr**

## ip

La commande ip est le couteau suisse de la configuration réseau, son paquet **iproute2** remplace les commandes du paquet **net-tools** :

### Attribuer une adresse

```bash
sudo ip addr add 192.168.0.54/24 dev eth0
```


### Connaître son adresse

```bash
sudo ip -4c addr show #-4 affiche uniquement les IPv4, -c pour l'affichage couleur
```

### Activer une interface réseau

```bash
ip link set eth0 update
```

### Désactiver une interface réseau

```bash
sudo ip link set eth0 down
```

### Supprimer une adresse d'une interface

```bash
sudo ip addr del 192.168.0.54 dev eth0
```


### Ajouter une gateway

```bash
sudo ip route add default via 192.168.0.1
```

## Les interfaces virtuelles

La création d'interfaces virtuelles permet de donner plusieurs adresses IP à une même carte réseau.

Cela permet par exemple de créer une adresse ip fixe pour une entrée DNS tout en la redirigeant via l'interface du datacenter vers une autre ip.

La ligne de commande est du type :

```bash
sudo ip link add link DEVICE name NAME type vlan
```

Voir <https://www.systutorials.com/docs/linux/man/8-ip-link/>

# Fixer le nom de machine

La commande **`hostname`** permet à la fois de consulter et de changer le nom de la machine.

Il est maintenant souhaitable d'utiliser la commande **`hostnamectl`** par exemple comme suit :

```bash
sudo hostnamectl set-hostname luciole
```

En effet de nombreuse machines reçoivent leur nom par la couche réseau lors du boot comme par exemple les images cloud.

Attention il ne s'agit pas du Fully Qualified Domain Name, mais seulement du nom de la machine sans le nom de domaine.

Il est possible aussi de modifier le nom de façon définitive via le fichier /etc/hostname

# Positionner le reverse

Pour ne pas être considéré comme spameur lors de l'envoi de mail il faut positionner le \"reverse\" du serveur sur le même Full Qualified Domain Name (fqdn), c'est-à-dire que si on fait une recherche du nom de la machine à partir de son adresse ip, le résultat doit être le nom de la machine suivi de son domaine.
Pour cela il faut :

    - Vérifier que dans la zone DNS de notre registrar, là où on a enregistré le nom de notre domaine, on a bien un champ A qui corresponde à l'adresse de la machine ;
    - Aller sur l'interface d'administration du serveur (Scaleway, OVH, Gandi, etc) ;
    - Modifier le reverse en donnant le fqdn de la machine.

Pour vérifier, il suffira de comparer l'adresse obtenue avec `dig $FQDN_Du_Serveur` avec `dig -x $IP_Du_Serveur`.

### Démarrage et arrêt du réseau

La commande **/etc/init.d/networking start** permet de démarrer la couche réseau.

### La résolution de nom

La commande **dig** permet de réaliser la résolution de nom.

La commande **`dig -x \$ADRESSE\_IP`** permet de réaliser la résolution inverse.

Le fichier **`/etc/resolv.conf`** est utilisé pour connaître les adresses des DNS.

## La modification du fichier `/etc/hosts`

Le fichier *`/etc/hosts`* contient les adresses et noms des machines connues.

On y trouve au minimum la définition du loopback et de la machine.

Les valeurs qui y sont priment sur la résolution DNS.

Exemple :
```bash
cat /etc/hosts
```

```
127.0.0.1 localhost griffon griffon.ecreall.com
88.191.77.45    griffon.ecreall.com
```

La modification est triviale puisqu'il suffit d'ajouter une ligne \$Adresse \$Nom1 \$Nom2.

# Les outils et commandes de tests réseau

## ping

La commande **ping** permet d'envoyer des paquets ICMP à une machine distante pour tester la connectivité du réseau.

## host

Un autre utilitaire de résolution de nom de domaine.

## Traceroute

La commande **traceroute** permet de connaître les noeuds du réseau nous séparant d'un machine cible.

## netstat

La commande **netstat** permet de connaître le statut des connexions réseaux.

**`netstat -tp`** permet de voir les connexions restées ouvertes et les processus associés.

Exemple :
```bash
sudo netstat -taupe
```

```
Connexions Internet actives (serveurs et établies)
Proto Recv-Q Send-Q Adresse locale          Adresse distante        Etat       User       Inode       PID/Program name
tcp        0      0 luciole.local:34978     ecs.amazonaws.com:www   ESTABLISHED michaellaunay 53652       11400/gvfsd-http
tcp        1      0 luciole.local:35516     ecs.amazonaws.com:www   CLOSE_WAIT  michaellaunay 50142       11400/gvfsd-http
tcp        1      0 luciole.local:50217     ecs.amazonaws.com:www   CLOSE_WAIT  michaellaunay 53380       11400/gvfsd-http
tcp        1      0 luciole.local:35515     ecs.amazonaws.com:www   CLOSE_WAIT  michaellaunay 50135       11400/gvfsd-http
tcp        0      0 luciole.local:34979     ecs.amazonaws.com:www   ESTABLISHED michaellaunay 53668       11400/gvfsd-http

```

Permet de voir des connexions en direction du cloud d'Amazon, un *cat /proc/11400/cmd* donne :

```bash
sudo cat /proc/11400/cmd
```

```
/usr/lib/gvfs/gvfsd-http--spawner:1.82/org/gtk/gvfs/exec_spaw/0
```

On voit que la commande est *spawnée* ce qui signifie que si on la *kill* elle sera relancée par le système.

Après quelques recherches, il s'avère que c'est l'application *Rhythmbox* qui va chercher les couvertures des albums écoutés en utilisant le service gvfs.

## tcpdump

La commande **`tcpdump`** permet "d'espionner" ce qui se passe sur nos interfaces réseau.

## nmap

La commande **`nmap`** permet de scanner les ports d'une machine et donc de faire un diagnostic des éventuelles portes d'entrée.

## ngrep

La commande **`ngrep`** permet de n'afficher les paquets réseau qu'à la condition qu'ils contiennent la chaîne cherchée.

## wireshark

Elle permet d'ausculter les paquets réseau comme ceux enregistrés par **`tcpdump`**.

## last

La commande **`last`** permet de connaître les derniers *login* réalisés sur la machine, leur date et adresse d'origine.

# Gestion des paquetages et installation de logiciels sous Ubuntu

## Ubuntu, un système \"Grand Public\"

Ubuntu simplifie extrêmement l'installation et devient donc facile d'accès pour un non linuxien.

## Quelle version choisir et pour quel usage ?

Il existe deux types de versions :

> -   Une version serveur, dépouillée d'interface graphique, permettant d'utiliser RAID et LVM,
> -   Une version pc de bureau, utilisant Gnome.

## Les dépôts

Nous avons vu précédemment la gestion graphique des dépôts.

Nous pouvons éditer le fichier */etc/apt/sources.list* et ajouter des dépôts.

Exemple :

```bash
sudo echo "deb http://packages.medibuntu.org/ karmic free non-free" >> /etc/apt/sources.list*
```

Toutefois il faudra télécharger la clé d'authentification du nouveau dépôt et l'ajouter avec :

```bash
sudo wget -q http://fr.packages.medibuntu.org/medibuntu-key.gpg -O- | sudo apt-key add -
```

Puis mettre à jour le cache avec **apt update**

## Installation de paquets

La commande **`apt install $NOM_PAQUET`** permet d'installer des paquets :

```bash
sudo apt install libdvdread7 mkisofs dvdbackup dvdauthor oggvideotools ffmpeg
sudo apt install libavcodec-58 libavdevice58 libavformat58
```

## Suppression de paquets

Pour supprimer un paquet on utilise **`apt remove $NOM_PAQUET`**.

## Informations sur les paquetages

Pour avoir la liste des paquets installés **`dpkg -l`**.

## Recherche de paquetages

La commande **`apt search $MOT_CLE`** permet de chercher un paquet à partir d'un mot de sa description.

## Mise à jour des paquetages

Pour mettre à jour la distribution :
```bash
sudo apt update
sudo apt upgrade
```

## Installation avec paramétrage

Pour installer un paquet en fournissant les réponses aux questions interactives et donc pourvoir scripter l'installation, il faut utiliser **debconf**.

Par exemple pour connaître les paramètres du paquet **postfix** :

```bash
sudo apt install debconf 
sudo debconf-show postfix
```

```
      postfix/main_cf_conversion_warning: true
      postfix/sqlite_warning:
      postfix/recipient_delim: +
      postfix/root_address:
      postfix/not_configured:
      postfix/compat_conversion_warning: true
      postfix/tlsmgr_upgrade_warning:
      postfix/mailbox_limit: 0
      postfix/newaliases: false
      postfix/kernel_version_warning:
      postfix/retry_upgrade_warning:
      postfix/lmtp_retired_warning: true
      postfix/mydomain_warning:
      postfix/protocols: all
    * postfix/main_mailer_type: Internet Site
      postfix/bad_recipient_delimiter:
      postfix/rfc1035_violation: false
      postfix/relay_restrictions_warning:
      postfix/mynetworks: 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
      postfix/dynamicmaps_conversion_warning:
      postfix/chattr: false
    * postfix/mailname: luciole.ecreall.com
      postfix/procmail: false
      postfix/destinations: $myhostname, luciole.ecreall.com, localhost
      postfix/relayhost:
```

Puis pour fixer les valeurs :

```bash
sudo debconf-set-selections
```

## Interfaces graphiques

Il existe deux types d'interfaces graphiques, une en mode texte
**`aptitude`** et une dans X11 **synaptic**.

## Commande alien

La commande `alien` permet de transformer un paquet *rpm* en paquet debian
et donc de pouvoir l'installer.

# Apache

[[Apache]]

# Postfix
[[Postfix]]

# MySQL
## Présentation

MySQL est une base de données légère facile à mettre en œuvre est très utilisée par les sites web, on lui préfèrera MariaDB (<https://mariadb.com/fr/>) ou mieux PostgresSQL
(<https://www.postgresql.org/>). À noter que MariaDB possède des offres serverless.

Son utilisation est libre, mais si les sources de l'application réalisée ne sont pas en GPL, il faut s'acquitter de l'achat d'une licence commerciale.

## Installation

La commande **apt install mysql-server** permet d'installer le serveur contenant la base de données alors que **apt-get install mysql-client** se contentera d'installer le client.

## Configuration

À l'installation il est fortement recommandé de donner un mot de passe à l'utilisateur root.

Liens :

> -   <http://www-fr.mysql.com/>
> -   <http://fr.wikipedia.org/wiki/Mysql>

# Sécurisation

## Certificat X 509

Les certificats X 509 sont utilisés à la fois pour l'authentification et pour le chiffrage des infrastructures à clés publiques (PKI) par exemple dans le protocole ssl lors des connexions ssh (port 22) ou https (port 443).

Ils sont délivrés par une autorité de certification et sont liés à une adresse électronique ou à une entrée DNS.

Si l'autorité de certification est connue du navigateur, la connexion se fera sans alerter l'utilisateur. Dans le cas contraire, il sera prévenu que le site n'est pas de confiance.

Toutefois, il est possible de disposer des avantages du chiffrement sans passer par une autorité *de confiance*, en utilisant un certificat *auto-signé* :

```bash
sudo make-ssl-cert /usr/share/ssl-cert/ssleay.cnf /etc/ssl/apache2/ssl/ssl.monsite.com.pem
sudo vim /etc/apache2/sites-available/ssl.monsite.com
```
Supprimer SSLCertificateFile et mettre à jour SSLCertificateKeyFile 
SSLCertificateKeyFile /etc/apache2/ssl/ssl.monsite.com.pem

```bash
sudo service apache2 restart
```

Mais les certificats autosignés ont l'inconvénient de ne pas avoir d'autorité connue et donc d'être refusé.

Nous pouvons utiliser les services let's encrypt qui permettent d'avoir un certificat valide reconnu par tous les navigateurs.

Configuration de Let's Encrypt pour générer nos certificats ssl :
```bash
sudo apt install apache2 #Si ce n'est pas fait, vérifier que le port http est ouvert sur ufw !
sudo apt install certbot
sudo certbot certonly --webroot -w /var/www/html -d URL_De_Mon_Site 
```

Vérification de la génération :

```bash
sudo openssl x509 -noout -text -in /etc/letsencrypt/live/URL_De_Mon_Site/fullchain.pem | grep "Not After"
```

```
Not After : Aug  5 11:25:06 2020 GMT
```


Modifier le cron de renouvellement `/etc/cron.d/certbot` et mettre :

```
0 \*/12 \* \* \* root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew --apache
```

Forcer le renouvellement :

Pour forcer le renouvellement de tous les noms de domaines
```bash
sudo certbot renew --force-renewal
```

Pour renouveler un domaine en particulier
```bash
sudo certbot -d michaellaunay.ecreall.com --force-renewal
```


Liens :

> -   <http://doc.ubuntu-fr.org/tutoriel/securiser_apache2_avec_ssl>
> -   <http://fr.wikipedia.org/wiki/X.509>

# postgrey

**postgrey** est un paquet de configuration de *postfix* permettant de différer la réception des mails des serveurs inconnus.

Le but est d'éliminer les spams, car les serveurs de spams ne prennent pas la peine de renvoyer un courrier dont la réception est différée.

# fail2ban

Fail2ban est un démon qui permet de modifier les règles du firewall pour bannir pendant un temps déterminé les adresses IP qui ont échoué plusieurs connexions de suite à l'un des services du serveur.

Le temps d'exclusion, le nombre de tentatives tolérées, les adresses non bannies sont configurables via les fichiers /etc/fail2ban/fail2ban.conf et /etc/fail2ban/jail.conf.

# Docker

[[Docker]]

### git

[[git]]

# Visual code
[[Visual studio code]]

# Site de tests des requêtes html

Pour tester les différentes requêtes du web :
    https://httpbin.org

# Capturer la session en cours

Ubuntu inclut un outil de capture d'écran appelé Screenshot pour l'image est Screencast pour la vidéo, pour le déclencher faire la combinaison de touches :

> Ctrl+Alt+Shift+R

Pour interrompre la session, faire la même combinaison de touches.

Il est possible de le lancer en ligne avec par exemple un délai avant enregistrement, mais il ne prend alors qu'un seul cliché à la fois:

```bash
gnome-screenshot -d 30 -f /tmp/capture.png 
```
Ce qui dans 30s prend une capture et l'enregistre au format png.

Par défaut le temps de capture de screencast est positionné à 30s pour le changer :

```bash
gsettings set org.gnome.settings-daemon.plugins.media-keys max-screencast-length 240
```
Remplaçons 240s par ce que nous voulons

### Formats de fichier de configuration

Selon les logiciels installés, on trouve différents formats de fichier de configuration dont voici les principaux :
    * json
    * xml
    * toml
    * ini
    * yaml

[^1]: <http://en.wikipedia.org/wiki/Comparison_of_operating_systems>
