# Le système de fichiers `/proc` sous GNU/Linux

## Objectifs du cours

À la fin de ce cours, nous savons expliquer le rôle de `/proc`, lire les informations principales exposées par le noyau Linux, comprendre le lien entre processus, noyau et système de fichiers virtuel, et utiliser `/proc` pour diagnostiquer un système en production.

Nous travaillons avec une approche à la fois théorique et pratique, adaptée à un niveau Master 2 en informatique.

----------------
# Chapitre 1 — Introduction générale à `/proc`

## Objectifs du chapitre

Dans ce premier chapitre, nous posons les bases nécessaires pour comprendre le rôle de `/proc` dans un système GNU/Linux.

Nous cherchons à répondre à une question simple : pourquoi le noyau Linux expose-t-il une partie de son état interne sous la forme de fichiers lisibles dans un répertoire ?

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est `/proc` ;
    
- distinguer un système de fichiers réel d’un système de fichiers virtuel ;
    
- comprendre pourquoi `/proc` est utile pour observer le système ;
    
- situer `/proc` par rapport à `/sys`, `/dev` et `/run` ;
    
- comprendre pourquoi `/proc` est central pour l’administration, le diagnostic et la sécurité.
    


## 1.1. Pourquoi étudier `/proc` ?

### 1.1.1. Une interface directe avec le noyau

Lorsque nous utilisons GNU/Linux, nous manipulons en permanence des commandes comme :

```bash
ps
top
htop
free
uptime
lsof
ss
```

Ces commandes semblent nous donner directement des informations sur les processus, la mémoire, le processeur, les fichiers ouverts ou le réseau.

En réalité, beaucoup de ces outils s’appuient sur des informations fournies par le noyau. Une partie importante de ces informations est exposée via `/proc`.

Nous pouvons donc voir `/proc` comme une interface d’observation du noyau.

Par exemple :

```bash
cat /proc/cpuinfo
```

nous donne des informations sur le processeur.

```bash
cat /proc/meminfo
```

nous donne des informations sur l’état de la mémoire.

```bash
cat /proc/uptime
```

nous donne le temps écoulé depuis le démarrage du système.

```bash
ls /proc
```

nous montre notamment une série de dossiers numériques correspondant aux processus actifs.

Nous comprenons donc que `/proc` n’est pas un simple répertoire documentaire. C’est une vue dynamique de l’état du système.


### 1.1.2. `/proc` n’est pas stocké sur le disque

Un point essentiel doit être compris dès le départ : les fichiers de `/proc` ne sont généralement pas des fichiers ordinaires.

Lorsque nous lisons :

```bash
cat /proc/meminfo
```

nous ne lisons pas un fichier texte enregistré sur le disque dur ou le SSD.

Nous demandons au noyau de générer, à cet instant précis, une représentation textuelle de certaines informations internes liées à la mémoire.

Autrement dit, le contenu est produit à la demande.

C’est pour cette raison que beaucoup de fichiers dans `/proc` semblent avoir une taille nulle :

```bash
ls -lh /proc/meminfo
```

Nous pouvons obtenir une sortie ressemblant à ceci :

```text
-r--r--r-- 1 root root 0 mai 16 14:10 /proc/meminfo
```

Le fichier affiche une taille de `0`, mais il contient pourtant des données lorsque nous le lisons.

Cela peut sembler contradictoire, mais c’est logique : le noyau ne connaît pas nécessairement à l’avance la taille exacte du contenu qui sera généré. Le fichier n’existe pas comme un fichier classique sur disque.


### 1.1.3. Un système de fichiers virtuel

`/proc` est ce que nous appelons un système de fichiers virtuel, ou pseudo-système de fichiers.

Un système de fichiers classique organise des données persistantes sur un support de stockage :

- disque dur ;
    
- SSD ;
    
- clé USB ;
    
- carte SD ;
    
- volume réseau.
    

Un système de fichiers virtuel, lui, expose des informations comme si elles étaient des fichiers, mais ces informations ne sont pas forcément stockées sur un périphérique physique.

Dans le cas de `/proc`, les données proviennent principalement :

- du noyau ;
    
- de la table des processus ;
    
- de la gestion mémoire ;
    
- de la pile réseau ;
    
- des paramètres internes du système.
    

Nous pouvons vérifier que `/proc` est monté comme un système de fichiers particulier :

```bash
findmnt /proc
```

Sortie possible :

```text
TARGET SOURCE FSTYPE OPTIONS
/proc  proc   proc   rw,nosuid,nodev,noexec,relatime
```

Le type de système de fichiers est ici `proc`.

Nous pouvons également utiliser :

```bash
mount | grep proc
```

Nous observons alors que `/proc` est monté explicitement par le système.


### 1.1.4. La philosophie Unix : “tout est fichier”

L’une des idées historiques des systèmes Unix est de représenter de nombreuses ressources sous forme de fichiers.

Cette approche permet d’utiliser des outils simples et génériques :

```bash
cat
less
grep
awk
sed
wc
head
tail
```

Avec `/proc`, nous pouvons lire des informations système avec les mêmes outils que pour des fichiers texte ordinaires.

Par exemple :

```bash
grep MemAvailable /proc/meminfo
```

ou :

```bash
grep "model name" /proc/cpuinfo
```

ou encore :

```bash
cat /proc/sys/kernel/hostname
```

Cette simplicité est très puissante.

Nous n’avons pas besoin d’une API complexe pour obtenir certaines informations. Nous pouvons lire un fichier.

Mais cette simplicité a aussi des limites : les formats ne sont pas toujours faciles à parser, certains fichiers sont très bas niveau, et leur interprétation demande une bonne connaissance du noyau Linux.


## 1.2. À quoi sert concrètement `/proc` ?

### 1.2.1. Observer les processus

La fonction historiquement centrale de `/proc` est l’observation des processus.

Chaque processus actif possède un répertoire dans `/proc`, nommé avec son PID.

Par exemple :

```bash
ls /proc/1
```

permet d’observer le processus numéro `1`, qui est généralement le processus d’initialisation du système, souvent `systemd` sur les distributions modernes.

Nous pouvons aussi observer le shell courant :

```bash
ls /proc/$$
```

`$$` contient le PID du shell courant.

Dans un répertoire de processus, nous trouvons des fichiers comme :

```text
cmdline
status
stat
environ
fd
maps
limits
cwd
exe
root
```

Ces fichiers permettent de savoir :

- quelle commande a lancé le processus ;
    
- dans quel état il se trouve ;
    
- quels fichiers il a ouverts ;
    
- quelles variables d’environnement il possède ;
    
- quelles bibliothèques sont chargées en mémoire ;
    
- quelles limites de ressources s’appliquent à lui.
    

Nous comprenons déjà pourquoi `/proc` est indispensable pour le diagnostic système.


### 1.2.2. Comprendre l’état de la mémoire

Linux utilise la mémoire de manière complexe.

Une machine peut afficher très peu de mémoire “libre”, tout en étant en bonne santé, parce que le noyau utilise la mémoire disponible comme cache disque.

C’est pour cela que `/proc/meminfo` est important.

Nous y trouvons des indicateurs comme :

```text
MemTotal
MemFree
MemAvailable
Buffers
Cached
SwapTotal
SwapFree
Dirty
Writeback
```

La différence entre `MemFree` et `MemAvailable` est particulièrement importante.

`MemFree` indique la mémoire totalement inutilisée.

`MemAvailable` estime la mémoire réellement disponible pour de nouvelles applications, en tenant compte de ce qui peut être libéré par le noyau.

Nous voyons donc que `/proc` ne sert pas seulement à afficher des chiffres. Il nous oblige à comprendre le modèle de gestion mémoire de Linux.


### 1.2.3. Observer la charge système

Nous pouvons lire :

```bash
cat /proc/loadavg
```

Sortie possible :

```text
0.35 0.42 0.38 2/842 15321
```

Les trois premières valeurs correspondent à la charge moyenne sur :

- 1 minute ;
    
- 5 minutes ;
    
- 15 minutes.
    

Ces données sont utilisées par la commande :

```bash
uptime
```

Mais `/proc/loadavg` nous montre directement la source brute.

Nous pouvons aussi lire :

```bash
cat /proc/uptime
```

Sortie possible :

```text
18243.51 72103.88
```

La première valeur indique le temps depuis le démarrage du système.

La seconde indique le temps passé en idle, cumulé sur les processeurs.

Là encore, `/proc` nous donne accès à des informations bas niveau que les outils classiques reformattent ensuite.


### 1.2.4. Lire et modifier certains paramètres noyau

Une partie très importante de `/proc` se trouve dans :

```bash
/proc/sys
```

Ce répertoire expose des paramètres du noyau.

Par exemple :

```bash
cat /proc/sys/kernel/hostname
```

affiche le nom d’hôte de la machine.

```bash
cat /proc/sys/net/ipv4/ip_forward
```

indique si le routage IPv4 est activé.

```bash
cat /proc/sys/vm/swappiness
```

affiche un paramètre influençant l’usage du swap.

Certains fichiers peuvent être modifiés :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

Cette commande active temporairement le forwarding IPv4.

Nous voyons ici que `/proc` n’est pas seulement une interface de lecture. Dans certains cas, c’est aussi une interface de configuration du noyau.

Cependant, nous devons être prudents : modifier un paramètre noyau sans comprendre son effet peut dégrader les performances, casser le réseau ou affaiblir la sécurité.


## 1.3. `/proc` comme outil pédagogique

### 1.3.1. Comprendre Linux en profondeur

Étudier `/proc` permet de dépasser l’usage superficiel des commandes.

Lorsque nous utilisons :

```bash
ps aux
```

nous voyons une liste de processus.

Mais lorsque nous explorons :

```bash
/proc/<PID>/status
/proc/<PID>/cmdline
/proc/<PID>/fd
/proc/<PID>/maps
```

nous comprenons beaucoup mieux ce qu’est réellement un processus du point de vue du noyau.

Nous voyons qu’un processus n’est pas seulement “un programme qui tourne”.

C’est un ensemble de ressources :

- un PID ;
    
- un utilisateur propriétaire ;
    
- un état d’exécution ;
    
- un espace mémoire ;
    
- des descripteurs de fichiers ;
    
- des variables d’environnement ;
    
- des limites de ressources ;
    
- un répertoire courant ;
    
- un exécutable associé ;
    
- des signaux ;
    
- des threads.
    

`/proc` rend ces notions visibles.


### 1.3.2. Faire le lien entre théorie et pratique

Dans un cours de systèmes d’exploitation, nous étudions souvent des notions abstraites :

- processus ;
    
- threads ;
    
- mémoire virtuelle ;
    
- scheduling ;
    
- appels système ;
    
- fichiers ;
    
- permissions ;
    
- signaux ;
    
- sockets ;
    
- espaces de noms.
    

`/proc` nous permet de relier ces notions à des observations concrètes.

Par exemple, lorsque nous étudions la mémoire virtuelle, nous pouvons lire :

```bash
cat /proc/<PID>/maps
```

Nous voyons alors les zones mémoire du processus.

Lorsque nous étudions les fichiers ouverts, nous pouvons lire :

```bash
ls -l /proc/<PID>/fd
```

Lorsque nous étudions les limites de ressources, nous pouvons lire :

```bash
cat /proc/<PID>/limits
```

Nous passons donc d’une théorie abstraite à une analyse expérimentale du système.


## 1.4. `/proc` pour l’administration système

### 1.4.1. Diagnostiquer un incident

Dans un contexte professionnel, `/proc` est utile pour analyser des incidents.

Exemples de situations :

- un serveur manque de mémoire ;
    
- un processus consomme trop de CPU ;
    
- un service garde trop de fichiers ouverts ;
    
- un fichier supprimé occupe encore de l’espace disque ;
    
- une application fuit de la mémoire ;
    
- un conteneur voit trop d’informations du système hôte ;
    
- un paramètre noyau est mal configuré ;
    
- une machine a une charge anormalement élevée.
    

Dans ces situations, `/proc` permet de vérifier les faits.

Nous ne sommes pas obligés de nous limiter aux outils haut niveau. Nous pouvons inspecter directement les données exposées par le noyau.


### 1.4.2. Exemple simple : identifier les fichiers ouverts

Supposons qu’un processus ait le PID `1234`.

Nous pouvons lister ses fichiers ouverts :

```bash
ls -l /proc/1234/fd
```

Nous pouvons voir des sorties ressemblant à :

```text
0 -> /dev/null
1 -> /var/log/app.log
2 -> /var/log/app-error.log
3 -> socket:[123456]
4 -> /tmp/data.txt
```

Nous comprenons alors que le processus écrit peut-être dans un fichier de log, utilise une socket réseau et garde un fichier temporaire ouvert.

Cette information peut être essentielle pour comprendre un comportement anormal.


### 1.4.3. Exemple simple : retrouver un fichier supprimé mais encore ouvert

Un cas classique en administration système est le suivant :

1. un gros fichier de log est supprimé ;
    
2. l’espace disque ne se libère pas ;
    
3. le processus continue d’écrire dans le fichier supprimé ;
    
4. le fichier n’apparaît plus dans le système de fichiers classique ;
    
5. mais il existe encore tant qu’un descripteur de fichier reste ouvert.
    

Nous pouvons détecter cela avec :

```bash
ls -l /proc/<PID>/fd | grep deleted
```

Nous obtenons par exemple :

```text
5 -> /var/log/app.log (deleted)
```

Cela signifie que le processus garde encore une référence vers ce fichier.

Nous comprenons alors pourquoi l’espace disque reste occupé.


## 1.5. `/proc` pour la sécurité

### 1.5.1. Une source d’information sensible

`/proc` expose beaucoup d’informations.

Certaines peuvent être sensibles :

```bash
cat /proc/<PID>/cmdline
```

peut afficher des arguments de ligne de commande.

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

peut afficher les variables d’environnement.

Or, certaines applications passent parfois des secrets dans les arguments ou dans l’environnement :

- tokens API ;
    
- mots de passe ;
    
- clés d’accès ;
    
- chaînes de connexion ;
    
- secrets applicatifs.
    

C’est une mauvaise pratique, mais elle existe.

Nous devons donc comprendre que `/proc` peut devenir une surface d’exposition.


### 1.5.2. Permissions et cloisonnement

Les permissions Unix limitent une partie des accès.

Un utilisateur ne peut pas toujours lire toutes les informations des processus appartenant à un autre utilisateur.

Mais selon la configuration du système, certaines informations restent visibles.

Sur un serveur multi-utilisateurs, cela peut poser problème.

Linux permet de renforcer la protection avec des options de montage de `/proc`, par exemple `hidepid`.

Nous approfondissons ce point dans un chapitre ultérieur, mais nous retenons déjà que `/proc` n’est pas neutre du point de vue sécurité.


### 1.5.3. Conteneurs et `/proc`

Dans Docker, Kubernetes ou d’autres systèmes de conteneurisation, `/proc` joue un rôle important.

Un conteneur possède généralement sa propre vue des processus grâce aux namespaces.

Cependant, si `/proc` est mal monté, ou si le conteneur est trop privilégié, il peut révéler des informations sur l’hôte.

Nous devons donc apprendre à poser les bonnes questions :

- Quel `/proc` le conteneur voit-il ?
    
- Est-ce le `/proc` de l’hôte ou une vue isolée ?
    
- Le conteneur peut-il voir les processus hors de son namespace ?
    
- Peut-il modifier des paramètres noyau ?
    
- A-t-il accès à des fichiers sensibles ?
    

Pour un ingénieur système, DevOps ou sécurité, ces questions sont essentielles.


## 1.6. Différence entre `/proc`, `/sys`, `/dev` et `/run`

### 1.6.1. Pourquoi comparer ces répertoires ?

Un étudiant peut facilement confondre `/proc`, `/sys`, `/dev` et `/run`, car ces répertoires ne se comportent pas comme de simples dossiers utilisateurs.

Ils sont souvent peuplés dynamiquement et contiennent des informations liées au système.

Nous devons donc clarifier leur rôle dès le début.


### 1.6.2. `/proc`

`/proc` expose principalement :

- les processus ;
    
- certaines informations noyau ;
    
- des statistiques système ;
    
- des paramètres modifiables via `/proc/sys`.
    

Exemples :

```bash
/proc/cpuinfo
/proc/meminfo
/proc/loadavg
/proc/uptime
/proc/<PID>
/proc/sys
```

Nous pouvons résumer ainsi :

```text
/proc = vue dynamique sur les processus et certains états du noyau
```


### 1.6.3. `/sys`

`/sys` est lié à `sysfs`.

Il expose une vue structurée du matériel, des périphériques, des pilotes et du modèle interne des objets du noyau.

Exemples :

```bash
/sys/class/net
/sys/block
/sys/devices
/sys/bus
/sys/module
```

Nous y trouvons par exemple des informations sur :

- les interfaces réseau ;
    
- les disques ;
    
- les périphériques PCI ;
    
- les modules noyau ;
    
- les pilotes ;
    
- l’alimentation ;
    
- certains paramètres matériels.
    

Nous pouvons résumer ainsi :

```text
/sys = vue structurée sur les périphériques, pilotes et objets noyau
```

La différence avec `/proc` est importante : `/proc` est historiquement centré sur les processus et certaines informations globales, tandis que `/sys` expose plus proprement le modèle matériel et les objets du noyau.


### 1.6.4. `/dev`

`/dev` contient des fichiers spéciaux représentant des périphériques.

Exemples :

```bash
/dev/sda
/dev/null
/dev/zero
/dev/random
/dev/tty
/dev/input
```

Ces fichiers ne sont pas des fichiers ordinaires.

Ils servent d’interface entre l’espace utilisateur et les périphériques ou pseudo-périphériques.

Par exemple :

```bash
echo "test" > /dev/null
```

envoie du texte vers un périphérique spécial qui ignore tout ce qu’on lui écrit.

Nous pouvons résumer ainsi :

```text
/dev = interface sous forme de fichiers vers les périphériques
```


### 1.6.5. `/run`

`/run` contient des données temporaires d’exécution.

Il est généralement vidé au redémarrage.

On y trouve souvent :

- des fichiers PID ;
    
- des sockets Unix ;
    
- des informations de services ;
    
- des données runtime de `systemd`, NetworkManager, Docker, etc.
    

Exemples :

```bash
/run/systemd
/run/user
/run/sshd.pid
/run/docker.sock
```

Nous pouvons résumer ainsi :

```text
/run = état temporaire des services depuis le démarrage
```


## 1.7. Comparaison synthétique

|Répertoire|Type|Rôle principal|Exemple|
|---|---|---|---|
|`/proc`|pseudo-système de fichiers|Processus et état noyau|`/proc/meminfo`, `/proc/<PID>`|
|`/sys`|pseudo-système de fichiers|Périphériques, pilotes, objets noyau|`/sys/class/net`|
|`/dev`|fichiers spéciaux|Accès aux périphériques|`/dev/sda`, `/dev/null`|
|`/run`|système temporaire|Données d’exécution des services|`/run/docker.sock`|

Nous retenons donc :

```text
/proc observe les processus et certains états noyau.
/sys décrit le matériel et les objets noyau.
/dev donne accès aux périphériques.
/run stocke l’état temporaire des services.
```


## 1.8. Première exploration guidée

Nous terminons le chapitre par une exploration simple.

### 1.8.1. Vérifier que `/proc` est monté

```bash
findmnt /proc
```

Nous identifions :

- le point de montage ;
    
- le type `proc` ;
    
- les options de montage.
    


### 1.8.2. Observer le contenu de `/proc`

```bash
ls /proc
```

Nous distinguons :

- les répertoires numériques ;
    
- les fichiers d’information globale ;
    
- les sous-répertoires comme `sys`, `net`, `self`, `thread-self`.
    

Les répertoires numériques correspondent aux PID des processus actifs.


### 1.8.3. Observer le processus courant

```bash
ls /proc/$$
```

Nous obtenons des informations sur notre shell courant.

Nous pouvons lire :

```bash
cat /proc/$$/comm
```

ou :

```bash
cat /proc/$$/status
```

Nous observons que notre shell est un processus comme les autres, exposé par `/proc`.


### 1.8.4. Lire quelques informations système

```bash
cat /proc/version
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/loadavg
cat /proc/uptime
```

Nous ne cherchons pas encore à tout interpréter.

L’objectif est d’observer la diversité des informations disponibles.


## 1.9. Points d’attention dès le début

### 1.9.1. Ne pas modifier sans comprendre

Lire `/proc` est généralement sans risque.

Modifier certains fichiers, notamment dans `/proc/sys`, peut avoir un impact direct sur le système.

Nous devons donc distinguer :

```bash
cat /proc/...
```

qui lit une information, de :

```bash
echo valeur | sudo tee /proc/...
```

qui modifie un paramètre.

Dans un environnement pédagogique, nous privilégions d’abord la lecture.


### 1.9.2. Attention aux permissions

Certaines commandes peuvent échouer :

```bash
cat /proc/<PID>/environ
```

peut retourner :

```text
Permission denied
```

Cela ne signifie pas que `/proc` ne fonctionne pas.

Cela signifie que le noyau applique une politique de permissions.

Nous devons apprendre à interpréter les erreurs autant que les sorties réussies.


### 1.9.3. Attention aux environnements conteneurisés

Dans un conteneur, `/proc` peut ne pas refléter toute la machine hôte.

Certaines informations peuvent être masquées, filtrées ou isolées.

Lorsque nous analysons `/proc`, nous devons toujours savoir dans quel contexte nous nous trouvons :

- machine physique ;
    
- machine virtuelle ;
    
- conteneur Docker ;
    
- pod Kubernetes ;
    
- environnement WSL ;
    
- système embarqué.
    

Le même fichier `/proc/meminfo` peut ne pas avoir exactement la même signification selon le contexte.


## Conclusion du chapitre 1

Nous retenons que `/proc` est une interface fondamentale entre l’espace utilisateur et le noyau Linux.

Ce n’est pas un simple dossier, mais un pseudo-système de fichiers généré dynamiquement.

Nous l’utilisons pour observer :

- les processus ;
    
- la mémoire ;
    
- le CPU ;
    
- la charge système ;
    
- certains paramètres noyau ;
    
- certains aspects réseau ;
    
- les ressources ouvertes par les programmes.
    

Nous comprenons aussi que `/proc` est à la fois :

- un outil pédagogique ;
    
- un outil d’administration ;
    
- un outil de diagnostic ;
    
- un point sensible pour la sécurité.
    

Dans les chapitres suivants, nous allons progressivement entrer dans le détail : d’abord l’architecture interne de `/proc`, puis les fichiers globaux du système, avant d’analyser finement les processus via `/proc/<PID>`.

# Chapitre 2 — Architecture interne de `/proc`

## Objectifs du chapitre

Dans ce chapitre, nous étudions la nature interne de `/proc`.

Nous avons vu dans le chapitre précédent que `/proc` n’est pas un répertoire ordinaire. Nous allons maintenant comprendre plus précisément comment il s’insère dans l’architecture Linux.

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est un pseudo-système de fichiers ;
    
- comprendre le rôle du Virtual File System, ou VFS ;
    
- identifier le type de montage de `/proc` ;
    
- expliquer pourquoi les fichiers de `/proc` affichent souvent une taille de `0` ;
    
- comprendre comment le noyau génère dynamiquement le contenu des fichiers ;
    
- distinguer les fichiers globaux de `/proc` et les répertoires associés aux processus ;
    
- comprendre les notions de `/proc/self` et `/proc/thread-self`.
    


## 2.1. `/proc` comme pseudo-système de fichiers

### 2.1.1. Rappel : un système de fichiers classique

Avant de comprendre `/proc`, nous rappelons ce qu’est un système de fichiers classique.

Un système de fichiers classique organise des données persistantes sur un support de stockage.

Par exemple :

```text
ext4
xfs
btrfs
zfs
vfat
ntfs
```

Ces systèmes de fichiers permettent de stocker :

- des fichiers ;
    
- des répertoires ;
    
- des permissions ;
    
- des dates ;
    
- des liens symboliques ;
    
- des métadonnées ;
    
- parfois des snapshots ou de la compression.
    

Quand nous écrivons un fichier dans `/home`, `/var` ou `/opt`, les données sont généralement enregistrées sur un disque ou un volume réseau.

Exemple :

```bash
echo "bonjour" > /tmp/exemple.txt
cat /tmp/exemple.txt
```

Ici, le contenu existe réellement dans un fichier, même si `/tmp` peut parfois être monté en mémoire selon la configuration.


### 2.1.2. Un pseudo-système de fichiers

`/proc` fonctionne différemment.

Il ne sert pas principalement à stocker des données persistantes. Il expose des informations internes du noyau sous une forme compatible avec l’interface fichier.

Nous parlons donc de pseudo-système de fichiers ou de système de fichiers virtuel.

Cela signifie que :

- les fichiers ne correspondent pas nécessairement à des blocs sur disque ;
    
- les contenus sont souvent calculés au moment de la lecture ;
    
- certaines entrées représentent des structures internes du noyau ;
    
- certains fichiers peuvent être modifiés pour changer des paramètres noyau ;
    
- l’arborescence évolue dynamiquement selon l’état du système.
    

Cette idée est essentielle : `/proc` ressemble à un répertoire, se parcourt comme un répertoire, mais ne se comporte pas comme un espace de stockage classique.


### 2.1.3. Pourquoi utiliser une interface fichier ?

Nous pouvons nous demander pourquoi Linux expose ces informations sous forme de fichiers au lieu d’utiliser uniquement des appels système spécialisés.

La réponse tient à plusieurs avantages :

1. Nous pouvons utiliser les outils Unix classiques.
    
2. Nous pouvons inspecter le système avec `cat`, `grep`, `awk`, `sed`, `less`.
    
3. Les scripts shell peuvent accéder facilement aux informations.
    
4. Les programmes peuvent lire ces informations sans bibliothèque complexe.
    
5. Le modèle reste cohérent avec la philosophie Unix.
    

Par exemple :

```bash
grep MemAvailable /proc/meminfo
```

permet d’obtenir rapidement la mémoire disponible.

Nous n’avons pas besoin d’écrire un programme C utilisant une API spécifique du noyau.

Cette simplicité explique en grande partie le succès de `/proc`.


## 2.2. Le rôle du VFS dans Linux

### 2.2.1. Qu’est-ce que le VFS ?

Linux utilise une couche d’abstraction appelée VFS, pour Virtual File System.

Le VFS fournit une interface commune pour manipuler des systèmes de fichiers très différents.

Quand un programme appelle :

```c
open()
read()
write()
close()
```

il ne sait pas forcément s’il lit un fichier sur un disque ext4, un fichier réseau NFS, un périphérique dans `/dev`, ou une entrée virtuelle dans `/proc`.

Le VFS masque ces différences et fournit une interface uniforme.

Nous pouvons le représenter ainsi :

```text
Programme utilisateur
        |
        v
Appels système : open, read, write, close
        |
        v
VFS Linux
        |
        +--> ext4, xfs, btrfs...
        +--> procfs
        +--> sysfs
        +--> tmpfs
        +--> devtmpfs
        +--> nfs
```

`/proc` est donc intégré au système via le VFS.


### 2.2.2. Ce que le VFS permet pour `/proc`

Grâce au VFS, un fichier virtuel de `/proc` peut être lu comme un fichier ordinaire.

Par exemple :

```bash
cat /proc/uptime
```

revient conceptuellement à :

1. ouvrir le fichier `/proc/uptime` ;
    
2. demander au noyau son contenu ;
    
3. lire les octets générés ;
    
4. afficher le résultat.
    

Mais derrière cette opération, il n’y a pas de lecture de blocs sur disque.

Le noyau appelle une fonction associée à cette entrée de procfs, qui produit le texte correspondant à l’état courant du système.


### 2.2.3. Même interface, comportement différent

Le point important est le suivant : l’interface est la même, mais le comportement interne change.

Nous pouvons utiliser :

```bash
cat /etc/hostname
cat /proc/sys/kernel/hostname
```

Ces deux commandes peuvent afficher le nom de la machine.

Mais elles ne lisent pas la même chose.

`/etc/hostname` est un fichier de configuration persistant.

`/proc/sys/kernel/hostname` reflète une valeur noyau active à l’instant où nous la lisons.

Si nous modifions temporairement le nom d’hôte via le noyau, le contenu actif peut changer sans que le fichier `/etc/hostname` soit immédiatement modifié.

Nous voyons donc que deux fichiers textuels peuvent avoir des natures très différentes.


## 2.3. Montage de `/proc`

### 2.3.1. Vérifier le montage

Sur un système GNU/Linux classique, `/proc` est monté automatiquement au démarrage.

Nous pouvons le vérifier avec :

```bash
findmnt /proc
```

Sortie possible :

```text
TARGET SOURCE FSTYPE OPTIONS
/proc  proc   proc   rw,nosuid,nodev,noexec,relatime
```

Nous observons plusieurs éléments :

- `TARGET` indique le point de montage ;
    
- `SOURCE` indique la source logique ;
    
- `FSTYPE` indique le type de système de fichiers ;
    
- `OPTIONS` indique les options de montage.
    

Ici, le type est `proc`.


### 2.3.2. Vérifier avec `mount`

Nous pouvons aussi utiliser :

```bash
mount | grep ' /proc '
```

Sortie possible :

```text
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
```

Nous retrouvons les mêmes informations sous une autre forme.

Le mot important est :

```text
type proc
```

Cela nous indique que `/proc` est monté avec le système de fichiers `procfs`.


### 2.3.3. Signification des options courantes

Les options affichées peuvent varier selon les distributions.

Nous rencontrons souvent :

```text
rw,nosuid,nodev,noexec,relatime
```

Nous les interprétons ainsi :

|Option|Signification|
|---|---|
|`rw`|le système de fichiers est monté en lecture-écriture|
|`nosuid`|les bits setuid/setgid ne sont pas pris en compte|
|`nodev`|les fichiers spéciaux de périphériques ne sont pas interprétés|
|`noexec`|l’exécution directe de fichiers depuis ce montage est interdite|
|`relatime`|les dates d’accès sont mises à jour de manière optimisée|

Même si `/proc` est monté en `rw`, cela ne signifie pas que tout est modifiable.

Beaucoup de fichiers sont en lecture seule.

Certains fichiers, notamment sous `/proc/sys`, sont modifiables si nous avons les droits nécessaires.


### 2.3.4. Monter `/proc` manuellement

Dans certains contextes particuliers, nous pouvons avoir besoin de monter `/proc` manuellement.

Cela peut arriver dans :

- un environnement `chroot` ;
    
- une image système minimale ;
    
- un conteneur ;
    
- une phase de réparation système ;
    
- un initramfs.
    

La commande est généralement :

```bash
sudo mount -t proc proc /proc
```

Nous précisons :

- `-t proc` : le type de système de fichiers ;
    
- `proc` : la source logique ;
    
- `/proc` : le point de montage.
    

Dans un `chroot`, nous pouvons monter `/proc` ainsi :

```bash
sudo mount -t proc proc /mnt/system/proc
```

Cela permet aux outils exécutés dans le `chroot` d’accéder à une vue de procfs.


### 2.3.5. Démontage de `/proc`

En temps normal, nous ne démontons pas `/proc` sur un système actif.

Mais dans un environnement de test ou de réparation, nous pouvons faire :

```bash
sudo umount /mnt/system/proc
```

Nous devons être prudents : beaucoup d’outils système dépendent de `/proc`.

Sur un système démarré normalement, démonter `/proc` peut casser des commandes ou des services.


## 2.4. Les fichiers de taille apparente `0`

### 2.4.1. Observation

Nous pouvons observer la taille apparente de certains fichiers :

```bash
ls -lh /proc/meminfo /proc/cpuinfo /proc/uptime
```

Sortie possible :

```text
-r--r--r-- 1 root root 0 mai 16 14:30 /proc/cpuinfo
-r--r--r-- 1 root root 0 mai 16 14:30 /proc/meminfo
-r--r--r-- 1 root root 0 mai 16 14:30 /proc/uptime
```

La taille affichée est `0`.

Pourtant :

```bash
cat /proc/meminfo
```

affiche bien du contenu.

Nous devons donc distinguer :

- la taille affichée par les métadonnées du fichier ;
    
- le contenu réellement produit lors de la lecture.
    


### 2.4.2. Pourquoi la taille est souvent nulle ?

Dans un système de fichiers classique, la taille du fichier correspond au nombre d’octets stockés.

Dans `/proc`, le contenu est généré à la demande.

Le noyau ne stocke pas nécessairement une version textuelle du fichier.

Il peut donc annoncer une taille de `0`, même si la lecture produit du texte.

Nous pouvons le comprendre ainsi :

```text
Fichier classique :
contenu stocké -> taille connue -> lecture

Fichier /proc :
état noyau -> génération à la lecture -> taille pas forcément connue
```

Ce comportement peut surprendre lorsque nous écrivons des scripts ou des programmes qui supposent qu’une taille de `0` signifie “fichier vide”.

Dans `/proc`, cette hypothèse est fausse.


### 2.4.3. Conséquence pour les programmes

Un programme mal écrit pourrait faire :

1. vérifier la taille du fichier ;
    
2. voir `0` ;
    
3. conclure qu’il n’y a rien à lire ;
    
4. ne pas appeler `read()`.
    

Cela fonctionnerait mal avec `/proc`.

La bonne approche consiste à lire le fichier jusqu’à la fin, sans se fier uniquement à sa taille apparente.

En shell, les commandes comme `cat`, `grep` ou `awk` fonctionnent correctement parce qu’elles lisent effectivement le flux.

Exemple :

```bash
awk '/MemAvailable/ { print $2 " " $3 }' /proc/meminfo
```


## 2.5. Génération dynamique des contenus

### 2.5.1. Une lecture correspond à un état instantané

Lorsque nous lisons un fichier dans `/proc`, nous obtenons une vue de l’état du système à un moment donné.

Par exemple :

```bash
cat /proc/uptime
```

Si nous relançons la commande une seconde plus tard, le résultat change.

```bash
cat /proc/loadavg
```

peut également changer d’une lecture à l’autre.

Nous devons donc comprendre que `/proc` est vivant : il reflète l’état courant du système.


### 2.5.2. Exemple avec `/proc/uptime`

Exécutons :

```bash
cat /proc/uptime
sleep 1
cat /proc/uptime
```

Nous pouvons obtenir :

```text
25420.15 98120.42
25421.16 98124.37
```

La première valeur augmente d’environ une seconde.

La seconde valeur représente le temps idle cumulé des processeurs, elle peut augmenter plus vite que le temps réel si plusieurs CPU sont inactifs en parallèle.

Nous voyons ici que le contenu n’est pas statique.


### 2.5.3. Exemple avec les processus

Les répertoires numériques de `/proc` changent constamment.

Si un processus démarre, un nouveau répertoire apparaît.

Si un processus se termine, son répertoire disparaît.

Nous pouvons observer cela avec :

```bash
ls /proc | grep -E '^[0-9]+$' | sort -n | tail
```

Puis lancer un processus :

```bash
sleep 1000 &
```

Nous récupérons son PID :

```bash
echo $!
```

Puis nous observons :

```bash
ls /proc/$!
```

Quand le processus se termine, le répertoire `/proc/<PID>` disparaît.


### 2.5.4. Conséquence : les erreurs de course

Comme `/proc` reflète un système vivant, nous pouvons rencontrer des erreurs de course.

Par exemple :

```bash
for pid in /proc/[0-9]*; do
    cat "$pid/status" > /dev/null
done
```

Un processus peut disparaître entre le moment où le shell liste `/proc/[0-9]*` et le moment où nous essayons de lire son fichier `status`.

Nous pouvons alors obtenir :

```text
No such file or directory
```

Ce n’est pas forcément une erreur grave. Cela signifie simplement que le processus s’est terminé entre deux opérations.

Dans un script robuste, nous devons anticiper ce cas.

Exemple :

```bash
for pid in /proc/[0-9]*; do
    [ -r "$pid/status" ] || continue
    cat "$pid/status" > /dev/null 2>&1 || continue
done
```

Nous apprenons ainsi une règle importante : avec `/proc`, l’état peut changer pendant que nous l’observons.


## 2.6. Structure générale de `/proc`

### 2.6.1. Vue d’ensemble

Lorsque nous faisons :

```bash
ls /proc
```

nous voyons généralement plusieurs types d’entrées :

1. des répertoires numériques ;
    
2. des fichiers globaux ;
    
3. des sous-répertoires spécialisés ;
    
4. des liens symboliques particuliers.
    

Exemple simplifié :

```text
1
42
137
cpuinfo
meminfo
uptime
loadavg
version
cmdline
filesystems
mounts
net
sys
self
thread-self
```

Nous devons apprendre à distinguer ces catégories.


### 2.6.2. Les répertoires numériques

Les répertoires numériques correspondent aux processus actifs.

Par exemple :

```text
/proc/1
/proc/1234
/proc/5678
```

Chaque nombre est un PID.

Nous pouvons vérifier le PID du shell courant :

```bash
echo $$
```

Puis :

```bash
ls /proc/$$
```

Nous observons le répertoire associé au processus du shell.


### 2.6.3. Les fichiers globaux

Les fichiers globaux donnent des informations sur l’ensemble du système.

Exemples :

```text
/proc/cpuinfo
/proc/meminfo
/proc/uptime
/proc/loadavg
/proc/version
/proc/filesystems
/proc/modules
/proc/swaps
/proc/partitions
```

Nous les étudions plus en détail au chapitre suivant.

Pour l’instant, nous retenons que ces fichiers ne sont pas liés à un processus particulier.

Ils décrivent un état global du noyau ou de la machine.


### 2.6.4. Les sous-répertoires spécialisés

Certains sous-répertoires regroupent des informations par domaine.

Par exemple :

```text
/proc/net
/proc/sys
/proc/irq
/proc/bus
/proc/fs
```

`/proc/net` expose des informations réseau.

`/proc/sys` expose des paramètres noyau lisibles et parfois modifiables.

`/proc/fs` expose des informations liées à certains systèmes de fichiers.

`/proc/irq` expose des informations liées aux interruptions matérielles.


### 2.6.5. Les liens symboliques particuliers

Nous trouvons aussi des liens symboliques spéciaux :

```bash
ls -l /proc/self
ls -l /proc/thread-self
```

Sortie possible :

```text
/proc/self -> 12345
/proc/thread-self -> 12345/task/12345
```

`/proc/self` pointe vers le processus courant, du point de vue du processus qui accède à ce lien.

Cela signifie que la cible peut changer selon le programme qui lit `/proc/self`.


## 2.7. `/proc/self`

### 2.7.1. Principe

`/proc/self` est un lien symbolique dynamique vers le répertoire `/proc/<PID>` du processus courant.

Si notre shell lit ce lien, il pointe vers le PID du shell.

Mais si `ls` lit ce lien, il peut pointer vers le PID du processus `ls`.

C’est un point subtil.

Exemple :

```bash
ls -l /proc/self
```

La commande `ls` affiche le lien du point de vue du processus `ls`, pas forcément du shell.


### 2.7.2. Comparaison avec `$$`

Dans un shell, nous pouvons faire :

```bash
echo $$
readlink /proc/self
```

Selon la commande utilisée, nous pouvons observer une différence.

`$$` est le PID du shell.

`/proc/self` est évalué par le processus qui accède à ce chemin.

Exemple :

```bash
echo "PID du shell : $$"
echo "PID vu via /proc/self : $(readlink /proc/self)"
```

Dans cette commande, `readlink` s’exécute dans un processus séparé. `/proc/self` peut donc pointer vers `readlink`, et non vers le shell.

Pour observer le shell, nous utilisons plutôt :

```bash
ls /proc/$$
```


### 2.7.3. Utilité de `/proc/self`

`/proc/self` est très utile pour les programmes.

Un programme peut accéder à ses propres informations sans connaître explicitement son PID.

Par exemple :

```bash
cat /proc/self/status
```

affiche le statut du processus `cat`, car c’est lui qui lit le fichier.

Dans un programme C, Python ou Rust, `/proc/self` permet d’inspecter :

- ses propres descripteurs de fichiers ;
    
- son environnement ;
    
- ses limites ;
    
- sa mémoire ;
    
- son exécutable ;
    
- son répertoire courant.
    

Exemple :

```bash
ls -l /proc/self/fd
```

Nous voyons les fichiers ouverts par le processus `ls`.


## 2.8. `/proc/thread-self`

### 2.8.1. Processus et threads dans `/proc`

Linux représente les threads d’un processus dans :

```text
/proc/<PID>/task/<TID>
```

Chaque thread possède un identifiant appelé TID.

Pour un processus mono-thread, le PID principal et le TID principal sont souvent identiques.

Pour un processus multi-thread, nous avons plusieurs entrées dans :

```bash
ls /proc/<PID>/task
```


### 2.8.2. Rôle de `/proc/thread-self`

`/proc/thread-self` pointe vers le thread courant.

Exemple :

```bash
ls -l /proc/thread-self
```

Sortie possible :

```text
/proc/thread-self -> 12345/task/12345
```

Dans un programme multi-thread, ce lien permet à chaque thread d’accéder à son propre répertoire dans `/proc`.

Cela est plus précis que `/proc/self`, qui désigne le processus.


### 2.8.3. Pourquoi c’est important ?

Certains diagnostics nécessitent une observation au niveau des threads.

Par exemple :

- état d’un thread ;
    
- pile d’un thread ;
    
- scheduling ;
    
- affinité CPU ;
    
- statistiques fines d’exécution.
    

Nous n’entrons pas encore dans le détail, mais nous retenons que `/proc` ne représente pas seulement les processus : il permet aussi d’observer les threads.


## 2.9. Types d’entrées dans `/proc`

### 2.9.1. Fichiers en lecture seule

Beaucoup d’entrées sont en lecture seule.

Exemple :

```bash
ls -l /proc/cpuinfo
```

Sortie possible :

```text
-r--r--r-- 1 root root 0 mai 16 15:00 /proc/cpuinfo
```

Nous pouvons lire :

```bash
cat /proc/cpuinfo
```

mais nous ne pouvons pas modifier ce fichier.


### 2.9.2. Fichiers modifiables

Certains fichiers sont modifiables, souvent sous `/proc/sys`.

Exemple :

```bash
ls -l /proc/sys/vm/swappiness
```

Sortie possible :

```text
-rw-r--r-- 1 root root 0 mai 16 15:00 /proc/sys/vm/swappiness
```

Nous pouvons lire :

```bash
cat /proc/sys/vm/swappiness
```

Et avec les droits administrateur, nous pouvons écrire :

```bash
echo 10 | sudo tee /proc/sys/vm/swappiness
```

Nous devons comprendre que cette écriture n’écrit pas un fichier sur disque. Elle demande au noyau de modifier un paramètre actif.


### 2.9.3. Répertoires

Certains éléments sont des répertoires :

```bash
ls -ld /proc/net /proc/sys /proc/1
```

Ils organisent les informations par domaine.


### 2.9.4. Liens symboliques

Certains éléments sont des liens symboliques :

```bash
ls -l /proc/self
ls -l /proc/$$/cwd
ls -l /proc/$$/exe
ls -l /proc/$$/root
```

Ces liens peuvent pointer vers :

- un répertoire de processus ;
    
- le répertoire courant d’un processus ;
    
- l’exécutable lancé ;
    
- la racine vue par le processus.
    

Ces liens sont très utiles, mais ils peuvent aussi être sensibles du point de vue sécurité.


### 2.9.5. Fichiers spéciaux et contenus particuliers

Certaines entrées ont un comportement spécial.

Par exemple :

```bash
/proc/kcore
```

peut représenter une vue de la mémoire du noyau.

Nous n’utilisons pas ce fichier dans un cours introductif sans précaution.

Certaines entrées peuvent être très volumineuses, difficiles à lire ou réservées à des usages avancés.

Nous devons donc éviter l’idée suivante :

```text
Tout ce qui est dans /proc peut être lu sans risque ni coût.
```

Ce n’est pas vrai.


## 2.10. Lecture de `/proc` avec les outils Unix

### 2.10.1. Utiliser `cat`

La commande la plus simple est :

```bash
cat /proc/uptime
cat /proc/loadavg
cat /proc/version
```

Elle convient aux fichiers courts.


### 2.10.2. Utiliser `less`

Pour les fichiers plus longs :

```bash
less /proc/cpuinfo
less /proc/meminfo
```

Pour quitter `less`, nous appuyons sur `q`.


### 2.10.3. Utiliser `grep`

Nous pouvons extraire une information précise :

```bash
grep MemAvailable /proc/meminfo
grep "model name" /proc/cpuinfo
grep "^Threads:" /proc/$$/status
```


### 2.10.4. Utiliser `awk`

`awk` permet de transformer les informations.

Exemple :

```bash
awk '/MemAvailable/ { print $2 " " $3 }' /proc/meminfo
```

Nous pouvons calculer une valeur en Mio :

```bash
awk '/MemAvailable/ { print int($2/1024) " MiB" }' /proc/meminfo
```


### 2.10.5. Utiliser `watch`

Pour observer une valeur dans le temps :

```bash
watch -n 1 'cat /proc/loadavg'
```

ou :

```bash
watch -n 1 'grep MemAvailable /proc/meminfo'
```

Nous voyons alors que `/proc` fournit une vue dynamique.


## 2.11. Bonnes pratiques de parsing

### 2.11.1. Ne pas supposer un format trop rigide

Les fichiers de `/proc` sont souvent textuels, mais ils ne sont pas tous conçus comme des API stables pour tous les usages.

Un script trop fragile peut casser si :

- une ligne change ;
    
- un champ est absent ;
    
- un noyau expose une information différemment ;
    
- le système est dans un conteneur ;
    
- une distribution applique des patchs particuliers.
    

Nous devons parser prudemment.


### 2.11.2. Préférer les clés explicites

Dans `/proc/meminfo`, nous avons des lignes de type :

```text
MemTotal:       16234568 kB
MemAvailable:   9345672 kB
```

Il est préférable de chercher la clé :

```bash
awk '/^MemAvailable:/ { print $2 }' /proc/meminfo
```

plutôt que de supposer que `MemAvailable` sera toujours à une ligne précise.


### 2.11.3. Gérer les erreurs

Un processus peut disparaître pendant que nous le lisons.

Un fichier peut être inaccessible.

Une information peut être absente.

Un script robuste doit donc gérer :

- `Permission denied` ;
    
- `No such file or directory` ;
    
- fichier vide ;
    
- format inattendu ;
    
- processus terminé ;
    
- environnement conteneurisé.
    

Exemple :

```bash
pid=1234

if [ -r "/proc/$pid/status" ]; then
    cat "/proc/$pid/status"
else
    echo "Processus inaccessible ou terminé"
fi
```


## 2.12. Mini-démonstration : explorer `/proc` proprement

Nous pouvons proposer une première exploration guidée.

### 2.12.1. Identifier le montage

```bash
findmnt /proc
```

Nous notons le type `proc`.


### 2.12.2. Voir les grandes catégories

```bash
ls /proc | head
ls /proc | grep -E '^[0-9]+$' | head
ls /proc | grep -E '^[a-zA-Z_ -]+$' | head
```

Nous distinguons les PID et les fichiers globaux.


### 2.12.3. Compter les processus visibles

```bash
ls /proc | grep -E '^[0-9]+$' | wc -l
```

Nous obtenons le nombre de processus visibles dans notre namespace PID.

Nous précisons que dans un conteneur, ce nombre peut être très différent de celui de l’hôte.


### 2.12.4. Observer notre shell

```bash
echo $$
cat /proc/$$/comm
cat /proc/$$/status | head
```

Nous identifions notre shell comme un processus.


### 2.12.5. Observer les fichiers ouverts du shell

```bash
ls -l /proc/$$/fd
```

Nous voyons au minimum les descripteurs :

```text
0
1
2
```

Nous pouvons discuter de l’entrée standard, de la sortie standard et de la sortie d’erreur.


## 2.13. Ce que nous devons retenir

Nous retenons les idées suivantes :

1. `/proc` est un pseudo-système de fichiers.
    
2. Il est intégré à Linux via le VFS.
    
3. Il est généralement monté automatiquement au démarrage.
    
4. Son type de montage est `proc`.
    
5. Ses fichiers ne sont généralement pas stockés sur disque.
    
6. Leur contenu est souvent généré dynamiquement lors de la lecture.
    
7. Les tailles affichées à `0` ne signifient pas que les fichiers sont vides.
    
8. Les répertoires numériques correspondent aux processus actifs.
    
9. `/proc/self` désigne le processus courant du point de vue du lecteur.
    
10. `/proc/thread-self` désigne le thread courant.
    
11. Certains fichiers sont lisibles, d’autres modifiables, d’autres sensibles.
    
12. Les scripts qui lisent `/proc` doivent être robustes face aux changements dynamiques.
    


## 2.14. Exercices

## Exercice 1 — Identifier la nature de `/proc`

Nous exécutons :

```bash
findmnt /proc
mount | grep ' /proc '
```

Nous répondons aux questions :

1. Quel est le type du système de fichiers ?
    
2. Quelles sont les options de montage ?
    
3. Que signifient `nosuid`, `nodev` et `noexec` ?
    
4. Pourquoi `/proc` n’est-il pas un système de fichiers classique ?
    


## Exercice 2 — Observer la taille apparente des fichiers

Nous exécutons :

```bash
ls -lh /proc/cpuinfo /proc/meminfo /proc/uptime
wc -c /proc/cpuinfo /proc/meminfo /proc/uptime
```

Nous comparons la taille affichée par `ls` et le nombre d’octets lus par `wc`.

Nous expliquons la différence.


## Exercice 3 — Observer le caractère dynamique

Nous exécutons :

```bash
cat /proc/uptime
sleep 2
cat /proc/uptime
```

Puis :

```bash
cat /proc/loadavg
sleep 2
cat /proc/loadavg
```

Nous expliquons pourquoi les valeurs changent.


## Exercice 4 — Observer un processus temporaire

Nous lançons :

```bash
sleep 60 &
pid=$!
echo $pid
ls /proc/$pid
cat /proc/$pid/status | head
```

Puis nous terminons le processus :

```bash
kill $pid
```

Nous vérifions :

```bash
ls /proc/$pid
```

Nous expliquons pourquoi le répertoire a disparu.


## Exercice 5 — Comprendre `/proc/self`

Nous exécutons :

```bash
echo $$
readlink /proc/self
bash -c 'echo "PID bash interne: $$"; readlink /proc/self'
```

Nous expliquons pourquoi `/proc/self` ne pointe pas toujours vers le PID que nous imaginons au départ.


## Exercice 6 — Écrire un mini-script robuste

Nous écrivons un script Bash qui liste les processus visibles et affiche leur nom.

Version simple :

```bash
for dir in /proc/[0-9]*; do
    pid=${dir#/proc/}
    if [ -r "$dir/comm" ]; then
        name=$(cat "$dir/comm" 2>/dev/null) || continue
        echo "$pid $name"
    fi
done
```

Nous expliquons pourquoi nous redirigeons les erreurs et pourquoi nous vérifions la lisibilité du fichier.


## Conclusion du chapitre 2

Nous comprenons maintenant l’architecture générale de `/proc`.

`/proc` n’est pas un dossier ordinaire, mais une interface dynamique fournie par le noyau via le VFS. Il nous permet de lire des informations système en utilisant les outils classiques du monde Unix, tout en manipulant en réalité des données générées à partir de l’état courant du noyau.

Cette architecture explique plusieurs comportements importants : fichiers de taille apparente nulle, contenu changeant, disparition possible des répertoires de processus, liens dynamiques comme `/proc/self`, et nécessité d’écrire des scripts robustes.

Dans le chapitre suivant, nous étudions les fichiers globaux de `/proc`, notamment `/proc/cpuinfo`, `/proc/meminfo`, `/proc/uptime`, `/proc/loadavg` et `/proc/version`, afin de comprendre comment Linux expose l’état général du système.

# Chapitre 3 — Informations globales sur le système

## Objectifs du chapitre

Dans ce chapitre, nous étudions les fichiers globaux de `/proc`.

Contrairement aux répertoires `/proc/<PID>`, qui décrivent un processus précis, les fichiers globaux exposent des informations sur l’ensemble du système : processeur, mémoire, noyau, charge, temps de fonctionnement, systèmes de fichiers, swap, modules, interruptions, disques ou encore ligne de commande du noyau.

À la fin de ce chapitre, nous savons :

- lire les principales informations système depuis `/proc` ;
    
- interpréter `/proc/cpuinfo` ;
    
- comprendre `/proc/meminfo` ;
    
- lire la version du noyau ;
    
- interpréter `/proc/uptime` et `/proc/loadavg` ;
    
- distinguer mémoire libre, mémoire disponible, cache et swap ;
    
- comprendre les limites de ces fichiers dans les machines virtuelles et les conteneurs ;
    
- utiliser ces informations pour établir un premier diagnostic système.
    


## 3.1. Vue d’ensemble des fichiers globaux

### 3.1.1. Les fichiers globaux les plus utiles

À la racine de `/proc`, nous trouvons de nombreux fichiers qui donnent une vue générale du système.

Nous rencontrons notamment :

```text
/proc/cpuinfo
/proc/meminfo
/proc/version
/proc/uptime
/proc/loadavg
/proc/cmdline
/proc/filesystems
/proc/mounts
/proc/swaps
/proc/partitions
/proc/modules
/proc/interrupts
/proc/stat
```

Ces fichiers ne décrivent pas un processus particulier. Ils exposent une partie de l’état global du noyau.

Nous pouvons commencer par les lister :

```bash
ls -lh /proc/cpuinfo /proc/meminfo /proc/version /proc/uptime /proc/loadavg
```

Nous observons encore une fois que beaucoup de ces fichiers affichent une taille de `0`, car leur contenu est généré dynamiquement.


### 3.1.2. Première lecture rapide

Nous pouvons obtenir un premier aperçu du système avec :

```bash
cat /proc/version
cat /proc/cpuinfo | head
cat /proc/meminfo | head
cat /proc/uptime
cat /proc/loadavg
```

Ces quelques commandes permettent déjà de répondre à des questions importantes :

- quelle version du noyau est utilisée ?
    
- quel processeur est visible ?
    
- combien de mémoire est disponible ?
    
- depuis combien de temps le système tourne-t-il ?
    
- quelle est la charge moyenne du système ?
    

Dans un diagnostic réel, ces informations constituent souvent la première couche d’observation.


## 3.2. Informations CPU avec `/proc/cpuinfo`

### 3.2.1. Lire `/proc/cpuinfo`

Le fichier `/proc/cpuinfo` expose des informations sur les processeurs logiques visibles par le noyau.

Nous le lisons avec :

```bash
cat /proc/cpuinfo
```

Sur une machine classique, nous obtenons un bloc d’informations par processeur logique.

Un extrait peut ressembler à ceci :

```text
processor   : 0
vendor_id   : GenuineIntel
cpu family  : 6
model       : 154
model name  : Intel(R) Core(TM) i7-12700H
stepping    : 3
microcode   : 0xffffffff
cpu MHz     : 2300.000
cache size  : 24576 KB
physical id : 0
siblings    : 20
core id     : 0
cpu cores   : 14
apicid      : 0
flags       : fpu vme de pse tsc msr pae mce cx8 apic ...
```

Nous devons apprendre à distinguer les informations réellement utiles des détails très bas niveau.


### 3.2.2. Nombre de CPU logiques

Chaque entrée `processor` correspond à un CPU logique vu par Linux.

Pour les compter :

```bash
grep -c '^processor' /proc/cpuinfo
```

Exemple :

```text
16
```

Cela signifie que Linux voit 16 processeurs logiques.

Un processeur logique n’est pas forcément un cœur physique. Il peut s’agir d’un thread matériel, par exemple avec l’Hyper-Threading chez Intel ou SMT chez AMD.

Nous pouvons aussi utiliser :

```bash
nproc
```

Mais ici, notre objectif est de comprendre la source d’information brute.


### 3.2.3. Modèle du processeur

Pour afficher le nom du modèle :

```bash
grep 'model name' /proc/cpuinfo | head -1
```

Exemple :

```text
model name  : AMD Ryzen 9 5900X 12-Core Processor
```

Nous utilisons `head -1` car le nom est répété pour chaque processeur logique.

Nous pouvons extraire uniquement la valeur :

```bash
awk -F': ' '/model name/ { print $2; exit }' /proc/cpuinfo
```


### 3.2.4. Cœurs physiques et threads logiques

Dans `/proc/cpuinfo`, plusieurs champs peuvent nous aider :

```text
physical id
core id
cpu cores
siblings
processor
```

Nous devons distinguer :

- `processor` : identifiant du CPU logique ;
    
- `physical id` : identifiant du socket physique ;
    
- `core id` : identifiant du cœur dans un socket ;
    
- `cpu cores` : nombre de cœurs physiques par socket ;
    
- `siblings` : nombre de threads logiques par socket.
    

Exemple simplifié :

```text
1 socket physique
8 cœurs physiques
16 processeurs logiques
```

Cela signifie probablement que chaque cœur expose deux threads matériels.

Nous pouvons comparer avec :

```bash
lscpu
```

`lscpu` donne une vue plus lisible, mais s’appuie sur les informations fournies par le noyau.


### 3.2.5. Flags CPU

Le champ `flags` est très important.

Exemple :

```bash
grep -m1 '^flags' /proc/cpuinfo
```

Il peut contenir des indicateurs comme :

```text
sse
sse2
avx
avx2
avx512f
aes
vmx
svm
nx
lm
```

Ces flags indiquent des fonctionnalités supportées par le processeur.

Quelques exemples :

|Flag|Signification|
|---|---|
|`sse`, `sse2`, `avx`, `avx2`|extensions SIMD pour calcul vectoriel|
|`aes`|accélération matérielle AES|
|`vmx`|virtualisation Intel VT-x|
|`svm`|virtualisation AMD-V|
|`nx`|bit No-eXecute pour la protection mémoire|
|`lm`|long mode, support 64 bits|

Ces informations peuvent être utiles pour :

- vérifier si une machine supporte la virtualisation ;
    
- vérifier la disponibilité d’instructions cryptographiques ;
    
- comprendre pourquoi un binaire optimisé ne fonctionne pas ;
    
- diagnostiquer un problème de performance lié aux instructions CPU.
    


### 3.2.6. Limites de `/proc/cpuinfo`

Nous devons rester prudents.

Dans une machine virtuelle, `/proc/cpuinfo` décrit le processeur virtuel exposé par l’hyperviseur, pas nécessairement le processeur physique complet.

Dans un conteneur, `/proc/cpuinfo` peut montrer plus de CPU que ceux réellement alloués par les limites cgroups.

Par exemple, dans Docker ou Kubernetes, une application peut voir 16 CPU dans `/proc/cpuinfo`, mais être limitée à 2 CPU par cgroup.

Nous retenons donc :

```text
/proc/cpuinfo indique les CPU visibles, pas toujours les ressources réellement garanties.
```

Pour les conteneurs modernes, nous devons aussi regarder les cgroups, par exemple dans `/sys/fs/cgroup`.


## 3.3. Mémoire système avec `/proc/meminfo`

### 3.3.1. Lire `/proc/meminfo`

Le fichier `/proc/meminfo` expose l’état détaillé de la mémoire.

Nous le lisons avec :

```bash
cat /proc/meminfo
```

Extrait possible :

```text
MemTotal:       16234568 kB
MemFree:         1234567 kB
MemAvailable:   9345672 kB
Buffers:          234567 kB
Cached:          4567890 kB
SwapCached:            0 kB
Active:          5432100 kB
Inactive:        3210000 kB
SwapTotal:       2097148 kB
SwapFree:        2097148 kB
Dirty:             12345 kB
Writeback:             0 kB
```

Ce fichier est beaucoup plus riche que la simple sortie de `free`.


### 3.3.2. `MemTotal`

`MemTotal` indique la quantité totale de mémoire RAM visible par le noyau.

Exemple :

```bash
grep '^MemTotal:' /proc/meminfo
```

Sortie :

```text
MemTotal:       16234568 kB
```

Nous pouvons convertir en Gio :

```bash
awk '/^MemTotal:/ { printf "%.2f GiB\n", $2/1024/1024 }' /proc/meminfo
```

Nous faisons attention : la valeur est exprimée en kilo-octets selon l’affichage de `/proc/meminfo`.


### 3.3.3. `MemFree`

`MemFree` indique la mémoire complètement inutilisée.

```bash
grep '^MemFree:' /proc/meminfo
```

Mais cette valeur ne suffit pas pour savoir si le système manque de mémoire.

Linux utilise volontairement la mémoire disponible pour le cache disque. Une mémoire “non libre” peut donc être facilement récupérable.

Nous devons éviter l’erreur classique :

```text
MemFree faible ≠ système forcément saturé
```


### 3.3.4. `MemAvailable`

`MemAvailable` est souvent l’indicateur le plus utile pour estimer la mémoire encore disponible pour lancer de nouveaux programmes.

```bash
grep '^MemAvailable:' /proc/meminfo
```

Il tient compte de la mémoire libre, mais aussi de certaines zones de cache récupérables.

Nous pouvons afficher la valeur en Mio :

```bash
awk '/^MemAvailable:/ { printf "%.0f MiB\n", $2/1024 }' /proc/meminfo
```

Dans un diagnostic simple, nous privilégions souvent `MemAvailable` à `MemFree`.


### 3.3.5. `Buffers` et `Cached`

Linux utilise la RAM pour accélérer les accès disque.

Deux champs importants sont :

```bash
grep -E '^(Buffers|Cached):' /proc/meminfo
```

- `Buffers` correspond à certaines métadonnées et buffers liés aux blocs ;
    
- `Cached` correspond au cache de fichiers.
    

Cette mémoire peut souvent être libérée si une application en a besoin.

C’est pourquoi une machine Linux peut afficher peu de mémoire libre tout en ayant beaucoup de mémoire disponible.


### 3.3.6. `SwapTotal` et `SwapFree`

Le swap est une extension de la mémoire sur disque.

Nous regardons :

```bash
grep -E '^(SwapTotal|SwapFree):' /proc/meminfo
```

Exemple :

```text
SwapTotal:       2097148 kB
SwapFree:        1048576 kB
```

Nous pouvons calculer le swap utilisé :

```bash
awk '
/^SwapTotal:/ { total=$2 }
/^SwapFree:/ { free=$2 }
END { printf "Swap utilisé : %.0f MiB\n", (total-free)/1024 }
' /proc/meminfo
```

Un peu de swap utilisé n’est pas forcément grave. En revanche, une utilisation massive et active du swap peut indiquer une pression mémoire importante.


### 3.3.7. `Dirty` et `Writeback`

Deux champs sont utiles pour comprendre les écritures disque :

```bash
grep -E '^(Dirty|Writeback):' /proc/meminfo
```

- `Dirty` : mémoire contenant des données modifiées mais pas encore écrites sur disque ;
    
- `Writeback` : mémoire en cours d’écriture vers le disque.
    

Si `Dirty` est très élevé, cela peut indiquer que le système accumule des écritures en attente.

Dans certains incidents, nous pouvons observer une forte latence disque associée à une quantité importante de pages dirty.


### 3.3.8. Comparaison avec `free`

La commande :

```bash
free -h
```

présente une synthèse lisible de la mémoire.

Mais elle s’appuie en grande partie sur les informations de `/proc/meminfo`.

Nous pouvons comparer :

```bash
free -h
cat /proc/meminfo | head
```

Notre objectif n’est pas de remplacer `free`, mais de comprendre ce qu’il résume.


### 3.3.9. Limites dans les conteneurs

Dans certains environnements conteneurisés, `/proc/meminfo` peut présenter la mémoire de l’hôte, tandis que le conteneur est limité par cgroup.

Cela dépend de la version du noyau, du runtime, de la configuration et du système de cgroups.

Nous devons donc être prudents dans Docker et Kubernetes.

Une application peut lire :

```text
MemTotal: 64 Go
```

alors que le conteneur est limité à :

```text
2 Go
```

Pour diagnostiquer un conteneur, nous devons regarder aussi les limites cgroups dans `/sys/fs/cgroup`.


## 3.4. Version du noyau avec `/proc/version`

### 3.4.1. Lire la version du noyau

Le fichier `/proc/version` donne des informations sur le noyau courant.

```bash
cat /proc/version
```

Exemple :

```text
Linux version 6.8.0-31-generic (buildd@lcy02-amd64-...) (x86_64-linux-gnu-gcc-13 ...) #31-Ubuntu SMP PREEMPT_DYNAMIC ...
```

Nous pouvons y trouver :

- la version du noyau ;
    
- le nom de construction ;
    
- le compilateur utilisé ;
    
- la date ou les métadonnées de compilation ;
    
- des informations de distribution.
    


### 3.4.2. Comparaison avec `uname`

La commande classique est :

```bash
uname -a
```

Exemple :

```text
Linux machine 6.8.0-31-generic #31-Ubuntu SMP PREEMPT_DYNAMIC x86_64 GNU/Linux
```

Nous pouvons aussi utiliser :

```bash
uname -r
```

pour obtenir uniquement la version du noyau :

```text
6.8.0-31-generic
```

`/proc/version` donne souvent plus de détails sur la compilation.


### 3.4.3. À ne pas confondre avec la version de distribution

Le noyau Linux et la distribution sont deux choses différentes.

Pour connaître la distribution, nous utilisons plutôt :

```bash
cat /etc/os-release
```

Exemple :

```text
NAME="Ubuntu"
VERSION="24.04 LTS (Noble Numbat)"
```

Nous retenons :

```text
/proc/version → version du noyau
/etc/os-release → version de la distribution
```

Cette distinction est importante en support système.


## 3.5. Temps de fonctionnement avec `/proc/uptime`

### 3.5.1. Lire `/proc/uptime`

Le fichier `/proc/uptime` contient deux valeurs :

```bash
cat /proc/uptime
```

Exemple :

```text
25420.15 98120.42
```

La première valeur représente le temps écoulé depuis le démarrage du système, en secondes.

La seconde représente le temps passé en idle par les CPU, cumulé.


### 3.5.2. Interpréter la première valeur

La première valeur est l’uptime.

Nous pouvons la convertir grossièrement :

```bash
awk '{ printf "Uptime : %.2f heures\n", $1/3600 }' /proc/uptime
```

Ou en jours :

```bash
awk '{ printf "Uptime : %.2f jours\n", $1/86400 }' /proc/uptime
```

Cette information permet de savoir depuis combien de temps le système n’a pas redémarré.


### 3.5.3. Interpréter la seconde valeur

La seconde valeur correspond au temps idle cumulé sur tous les CPU.

Sur une machine avec plusieurs CPU logiques, cette valeur peut augmenter plus vite que le temps réel.

Exemple :

- si le système a 4 CPU logiques ;
    
- si les 4 CPU restent inactifs pendant 1 seconde ;
    
- le temps idle cumulé augmente d’environ 4 secondes.
    

Nous devons donc éviter de comparer naïvement la seconde valeur à la première.


### 3.5.4. Comparaison avec `uptime`

La commande :

```bash
uptime
```

affiche une sortie plus lisible :

```text
15:42:01 up 2 days,  3:12,  2 users,  load average: 0.24, 0.32, 0.28
```

Elle combine notamment des informations issues de `/proc/uptime` et `/proc/loadavg`.


## 3.6. Charge système avec `/proc/loadavg`

### 3.6.1. Lire `/proc/loadavg`

Nous lisons :

```bash
cat /proc/loadavg
```

Exemple :

```text
0.35 0.42 0.38 2/842 15321
```

Ce fichier contient cinq informations :

```text
load1 load5 load15 running/total last_pid
```

Dans l’exemple :

- `0.35` : charge moyenne sur 1 minute ;
    
- `0.42` : charge moyenne sur 5 minutes ;
    
- `0.38` : charge moyenne sur 15 minutes ;
    
- `2/842` : 2 tâches exécutables sur 842 tâches totales ;
    
- `15321` : dernier PID attribué.
    


### 3.6.2. Que signifie la charge moyenne ?

La charge moyenne mesure le nombre moyen de tâches :

- en cours d’exécution ;
    
- prêtes à s’exécuter ;
    
- parfois bloquées en attente non interruptible, notamment sur des entrées/sorties.
    

Ce n’est pas simplement un pourcentage CPU.

Une charge de `4.00` n’a pas le même sens sur une machine à 2 CPU logiques et sur une machine à 16 CPU logiques.


### 3.6.3. Interprétation selon le nombre de CPU

Supposons une machine avec 4 CPU logiques.

Une règle simplifiée :

|Load average|Interprétation approximative|
|---|---|
|`0.50`|machine peu chargée|
|`2.00`|charge modérée|
|`4.00`|CPU pleinement utilisés|
|`8.00`|file d’attente importante|

Mais cette interprétation dépend fortement du contexte.

Pour connaître le nombre de CPU logiques :

```bash
nproc
```

ou :

```bash
grep -c '^processor' /proc/cpuinfo
```


### 3.6.4. Les trois valeurs : 1, 5 et 15 minutes

Les trois valeurs permettent de voir la tendance.

Exemple :

```text
8.00 2.00 1.00
```

La charge est montée récemment.

Exemple inverse :

```text
1.00 4.00 8.00
```

La charge était élevée avant, mais elle redescend.

Nous devons donc lire les trois valeurs ensemble.


### 3.6.5. Charge CPU ou attente I/O ?

Une charge élevée ne signifie pas toujours que le CPU est saturé.

Des processus bloqués en attente disque peuvent faire monter la charge.

Pour différencier les causes, nous devons compléter avec :

```bash
top
htop
vmstat 1
iostat
pidstat
```

Mais `/proc/loadavg` nous donne le premier signal.


## 3.7. Ligne de commande du noyau avec `/proc/cmdline`

### 3.7.1. Lire `/proc/cmdline`

Le fichier `/proc/cmdline` contient les paramètres passés au noyau au démarrage.

```bash
cat /proc/cmdline
```

Exemple :

```text
BOOT_IMAGE=/boot/vmlinuz-6.8.0-31-generic root=UUID=... ro quiet splash
```

Nous y trouvons souvent :

- l’image noyau démarrée ;
    
- la partition racine ;
    
- des options de démarrage ;
    
- des paramètres de sécurité ;
    
- des paramètres de debug ;
    
- des options liées au matériel.
    


### 3.7.2. Utilité pratique

Ce fichier est utile pour vérifier :

- si le système démarre en mode normal ou debug ;
    
- quelle racine est utilisée ;
    
- si certaines mitigations CPU sont activées ou désactivées ;
    
- si des paramètres réseau, console ou mémoire ont été passés ;
    
- si des options spécifiques ont été configurées dans GRUB.
    

Exemple :

```bash
cat /proc/cmdline | tr ' ' '\n'
```

Cette commande affiche un paramètre par ligne.


## 3.8. Systèmes de fichiers avec `/proc/filesystems`

### 3.8.1. Lire les systèmes de fichiers supportés

Nous lisons :

```bash
cat /proc/filesystems
```

Exemple :

```text
nodev   sysfs
nodev   tmpfs
nodev   proc
        ext4
        xfs
        btrfs
nodev   cgroup
nodev   devpts
```

Ce fichier indique les systèmes de fichiers connus par le noyau courant.


### 3.8.2. Signification de `nodev`

La mention `nodev` indique que le système de fichiers ne correspond pas à un périphérique bloc classique.

Par exemple :

```text
nodev proc
nodev sysfs
nodev tmpfs
```

`proc`, `sysfs` et `tmpfs` ne sont pas montés depuis une partition disque classique.


### 3.8.3. Utilité pratique

Ce fichier peut nous aider à vérifier si un type de système de fichiers est supporté.

Par exemple :

```bash
grep ext4 /proc/filesystems
grep btrfs /proc/filesystems
grep nfs /proc/filesystems
```

Si un système de fichiers n’apparaît pas, il peut être absent, non chargé ou disponible via un module non encore chargé.


## 3.9. Points de montage avec `/proc/mounts`

### 3.9.1. Lire `/proc/mounts`

Le fichier `/proc/mounts` expose les points de montage vus par le processus courant.

```bash
cat /proc/mounts
```

Il peut être long. Nous pouvons le rendre plus lisible avec :

```bash
column -t /proc/mounts | less
```

ou utiliser directement :

```bash
findmnt
```


### 3.9.2. Différence avec `/etc/fstab`

`/etc/fstab` décrit les montages configurés de manière persistante.

`/proc/mounts` décrit les montages réellement actifs.

Nous retenons :

```text
/etc/fstab → intention de configuration
/proc/mounts → état réel du système
```

Il est possible qu’un montage actif ne soit pas dans `/etc/fstab`, et inversement qu’un montage dans `/etc/fstab` ne soit pas actuellement monté.


### 3.9.3. Lien avec `/proc/self/mounts`

Sur les systèmes modernes, `/proc/mounts` est souvent lié à :

```bash
/proc/self/mounts
```

Nous pouvons vérifier :

```bash
ls -l /proc/mounts
```

Cela signifie que les montages sont vus du point de vue du processus courant, ce qui est important avec les namespaces de montage dans les conteneurs.


## 3.10. Swap avec `/proc/swaps`

### 3.10.1. Lire les espaces de swap

Nous lisons :

```bash
cat /proc/swaps
```

Exemple :

```text
Filename                Type        Size       Used    Priority
/swapfile               file        2097148    0       -2
```

Ce fichier indique :

- le fichier ou périphérique de swap ;
    
- son type ;
    
- sa taille ;
    
- la quantité utilisée ;
    
- sa priorité.
    


### 3.10.2. Comparaison avec `swapon`

La commande suivante donne une vue plus lisible :

```bash
swapon --show
```

Mais `/proc/swaps` est la source directe exposée par le noyau.


### 3.10.3. Diagnostic rapide

Si une machine est lente, nous pouvons vérifier :

```bash
cat /proc/swaps
grep -E '^(SwapTotal|SwapFree):' /proc/meminfo
```

Si beaucoup de swap est utilisé et que la mémoire disponible est faible, nous pouvons suspecter une pression mémoire.

Mais nous devons compléter avec d’autres outils pour savoir si le système est en train de swapper activement.


## 3.11. Partitions avec `/proc/partitions`

### 3.11.1. Lire les partitions connues

Nous lisons :

```bash
cat /proc/partitions
```

Exemple :

```text
major minor  #blocks  name

   8        0  976762584 sda
   8        1     524288 sda1
   8        2  976236544 sda2
 259        0 1000204632 nvme0n1
 259        1     524288 nvme0n1p1
```

Nous voyons les périphériques bloc et leurs partitions.


### 3.11.2. Limites

Ce fichier donne une vue brute.

Pour une analyse plus lisible, nous utilisons souvent :

```bash
lsblk
blkid
fdisk -l
```

Mais `/proc/partitions` reste utile pour comprendre ce que le noyau voit.


## 3.12. Modules noyau avec `/proc/modules`

### 3.12.1. Lire les modules chargés

Nous lisons :

```bash
cat /proc/modules
```

Extrait possible :

```text
iwlwifi 598016 1 iwlmvm, Live 0x0000000000000000
nvidia 56848384 125, Live 0x0000000000000000
vboxdrv 655360 3 vboxnetadp,vboxnetflt,vboxpci, Live 0x0000000000000000
```

Chaque ligne décrit un module noyau chargé.


### 3.12.2. Comparaison avec `lsmod`

La commande :

```bash
lsmod
```

affiche les modules de manière plus lisible.

En réalité, `lsmod` lit généralement `/proc/modules`.

Nous pouvons comparer :

```bash
lsmod | head
cat /proc/modules | head
```


### 3.12.3. Utilité pratique

Nous utilisons ces informations pour vérifier :

- si un pilote est chargé ;
    
- si un module réseau est présent ;
    
- si un module de virtualisation est actif ;
    
- si un module GPU est chargé ;
    
- si un module de système de fichiers est disponible.
    

Exemple :

```bash
grep -E 'nvidia|amdgpu|i915' /proc/modules
```


## 3.13. Interruptions avec `/proc/interrupts`

### 3.13.1. Lire les interruptions

Le fichier `/proc/interrupts` expose les interruptions matérielles et leur répartition par CPU.

```bash
cat /proc/interrupts
```

Extrait simplifié :

```text
           CPU0       CPU1       CPU2       CPU3
  0:         12          0          0          0   IO-APIC   2-edge      timer
  1:        120          0          0          0   IO-APIC   1-edge      i8042
 16:      10234       9876      11234       9543   IO-APIC  16-fasteoi   eth0
```

Nous voyons combien d’interruptions chaque CPU a traité.


### 3.13.2. Utilité avancée

Ce fichier devient utile lorsque nous analysons :

- une carte réseau très sollicitée ;
    
- une répartition déséquilibrée des interruptions ;
    
- une latence système ;
    
- un problème de pilote ;
    
- une machine haute performance.
    

Dans un cours introductif à `/proc`, nous ne détaillons pas encore l’optimisation IRQ, mais nous retenons que `/proc/interrupts` donne une vue précieuse sur l’activité matérielle.


## 3.14. Statistiques globales avec `/proc/stat`

### 3.14.1. Lire `/proc/stat`

Le fichier `/proc/stat` contient de nombreuses statistiques globales.

```bash
cat /proc/stat | head
```

Exemple :

```text
cpu  123456 789 34567 9876543 1234 0 567 0 0 0
cpu0 12345 67 3456 987654 123 0 56 0 0 0
cpu1 12346 68 3457 987655 124 0 57 0 0 0
intr 123456789 ...
ctxt 987654321
btime 1715860000
processes 123456
procs_running 2
procs_blocked 0
```

Ce fichier est utilisé par de nombreux outils de monitoring.


### 3.14.2. Champs utiles

Nous pouvons y trouver :

- temps CPU par mode ;
    
- nombre total d’interruptions ;
    
- nombre de changements de contexte ;
    
- heure de boot ;
    
- nombre de processus créés depuis le démarrage ;
    
- processus en cours d’exécution ;
    
- processus bloqués.
    

Exemple :

```bash
grep '^btime' /proc/stat
```

`btime` donne l’heure de démarrage du système sous forme de timestamp Unix.

Nous pouvons la convertir :

```bash
date -d "@$(awk '/^btime/ {print $2}' /proc/stat)"
```


### 3.14.3. Pourquoi `/proc/stat` est plus difficile

Contrairement à `/proc/meminfo`, `/proc/stat` est moins lisible directement.

Les champs CPU nécessitent une interprétation précise.

Nous ne les utilisons pas naïvement.

Pour comprendre l’utilisation CPU, nous préférons souvent :

```bash
top
mpstat
pidstat
vmstat
```

Mais ces outils reposent en partie sur ces statistiques.


## 3.15. Construire une première synthèse système

### 3.15.1. Objectif

Nous voulons maintenant construire une commande qui produit une synthèse simple à partir de `/proc`.

Nous cherchons à afficher :

- version du noyau ;
    
- uptime ;
    
- CPU logiques ;
    
- mémoire totale ;
    
- mémoire disponible ;
    
- load average ;
    
- nombre de processus visibles.
    


### 3.15.2. Script Bash simple

Nous pouvons écrire :

```bash
##!/usr/bin/env bash

kernel=$(uname -r)
cpu_count=$(grep -c '^processor' /proc/cpuinfo)
mem_total=$(awk '/^MemTotal:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo)
mem_available=$(awk '/^MemAvailable:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo)
uptime_days=$(awk '{ printf "%.2f jours", $1/86400 }' /proc/uptime)
loadavg=$(awk '{ print $1 " " $2 " " $3 }' /proc/loadavg)
process_count=$(ls /proc | grep -E '^[0-9]+$' | wc -l)

echo "Kernel             : $kernel"
echo "CPU logiques       : $cpu_count"
echo "Mémoire totale     : $mem_total"
echo "Mémoire disponible : $mem_available"
echo "Uptime             : $uptime_days"
echo "Load average       : $loadavg"
echo "Processus visibles : $process_count"
```

Nous pouvons le sauvegarder :

```bash
nano proc-summary.sh
chmod +x proc-summary.sh
./proc-summary.sh
```


### 3.15.3. Discussion sur la robustesse

Ce script est volontairement simple.

Dans un environnement de production, nous devons améliorer plusieurs points :

- gérer les erreurs de lecture ;
    
- éviter de supposer que tous les champs existent ;
    
- tenir compte des conteneurs ;
    
- éviter certains parsings trop fragiles ;
    
- gérer les différences de noyau ;
    
- produire une sortie machine-readable, par exemple JSON.
    

Mais ce script montre déjà que `/proc` permet de construire rapidement un outil d’observation système.


## 3.16. Étude de cas : première analyse d’un serveur

### 3.16.1. Situation

Nous nous connectons à un serveur qui semble lent.

Nous ne savons pas encore si le problème vient :

- du CPU ;
    
- de la mémoire ;
    
- du swap ;
    
- du disque ;
    
- du réseau ;
    
- d’un processus particulier.
    

Nous utilisons uniquement quelques fichiers globaux de `/proc` pour une première analyse.


### 3.16.2. Étape 1 : version et uptime

```bash
cat /proc/version
cat /proc/uptime
```

Nous cherchons à savoir :

- si le noyau est ancien ;
    
- si la machine vient de redémarrer ;
    
- si l’uptime est très long ;
    
- si un redémarrage récent peut expliquer un incident.
    


### 3.16.3. Étape 2 : CPU visible

```bash
grep -c '^processor' /proc/cpuinfo
grep 'model name' /proc/cpuinfo | head -1
```

Nous identifions la capacité CPU apparente.

Si la charge moyenne est de `8` sur une machine à `2` CPU logiques, la situation est plus préoccupante que sur une machine à `32` CPU logiques.


### 3.16.4. Étape 3 : charge moyenne

```bash
cat /proc/loadavg
```

Nous interprétons les trois valeurs.

Exemple :

```text
12.00 10.50 8.20
```

Si la machine a 4 CPU logiques, la charge est probablement élevée.

Mais nous ne concluons pas encore que le CPU est seul responsable.


### 3.16.5. Étape 4 : mémoire disponible

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Cached|SwapTotal|SwapFree):' /proc/meminfo
```

Nous observons :

- mémoire totale ;
    
- mémoire réellement disponible ;
    
- cache ;
    
- swap total ;
    
- swap libre.
    

Si `MemAvailable` est très bas et que le swap est largement utilisé, nous suspectons une pression mémoire.


### 3.16.6. Étape 5 : hypothèse initiale

À partir de ces informations, nous pouvons formuler une première hypothèse :

```text
La machine présente une charge élevée, avec peu de mémoire disponible et une utilisation importante du swap. Le ralentissement peut être lié à une pression mémoire provoquant des attentes I/O.
```

Ou au contraire :

```text
La charge est faible, la mémoire disponible est correcte, et le swap n’est pas utilisé. Le ralentissement perçu vient probablement d’un service spécifique, du réseau ou d’un stockage distant.
```

Nous ne cherchons pas encore à conclure définitivement. Nous construisons une première lecture du système.


## 3.17. Limites des informations globales

### 3.17.1. Une vue globale ne suffit pas toujours

Les fichiers globaux nous donnent une vision d’ensemble, mais ils ne disent pas toujours quel processus est responsable.

Par exemple :

```bash
cat /proc/loadavg
```

peut indiquer une forte charge, mais ne donne pas directement le processus responsable.

```bash
cat /proc/meminfo
```

peut indiquer une pression mémoire, mais ne dit pas immédiatement quelle application consomme le plus.

Pour aller plus loin, nous devons ensuite explorer :

```text
/proc/<PID>
```

C’est l’objet des chapitres suivants.


### 3.17.2. Attention aux moyennes

Les moyennes peuvent masquer des pics.

La charge moyenne sur 15 minutes peut être faible alors qu’un pic violent vient de se produire.

Inversement, la charge sur 15 minutes peut rester élevée alors que le problème est déjà terminé.

Nous devons toujours croiser :

- 1 minute ;
    
- 5 minutes ;
    
- 15 minutes ;
    
- observation en temps réel ;
    
- logs ;
    
- métriques applicatives.
    


### 3.17.3. Attention aux environnements virtualisés

Dans une machine virtuelle ou un conteneur, les fichiers globaux peuvent ne pas représenter exactement les ressources physiques.

Nous devons identifier le contexte :

```bash
systemd-detect-virt
```

ou observer les cgroups :

```bash
cat /proc/1/cgroup
```

Dans Kubernetes, la lecture brute de `/proc` peut être trompeuse si nous oublions les limites CPU et mémoire du pod.


## 3.18. Exercices

## Exercice 1 — Identifier le CPU

Nous exécutons :

```bash
grep -c '^processor' /proc/cpuinfo
grep 'model name' /proc/cpuinfo | head -1
grep -m1 '^flags' /proc/cpuinfo
```

Nous répondons aux questions :

1. Combien de CPU logiques sont visibles ?
    
2. Quel est le modèle du processeur ?
    
3. Le processeur expose-t-il `vmx` ou `svm` ?
    
4. Le processeur expose-t-il `aes` ?
    
5. Quelle différence faisons-nous entre cœur physique et CPU logique ?
    


## Exercice 2 — Comprendre la mémoire

Nous exécutons :

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree):' /proc/meminfo
```

Nous répondons aux questions :

1. Quelle est la mémoire totale ?
    
2. Quelle est la mémoire libre ?
    
3. Quelle est la mémoire disponible ?
    
4. Pourquoi `MemFree` peut-il être beaucoup plus faible que `MemAvailable` ?
    
5. Le swap est-il activé ?
    
6. Quelle quantité de swap est utilisée ?
    


## Exercice 3 — Uptime et charge

Nous exécutons :

```bash
cat /proc/uptime
cat /proc/loadavg
nproc
```

Nous répondons aux questions :

1. Depuis combien de temps la machine est-elle démarrée ?
    
2. Quelle est la charge moyenne sur 1, 5 et 15 minutes ?
    
3. Cette charge est-elle élevée par rapport au nombre de CPU logiques ?
    
4. La charge semble-t-elle monter ou descendre ?
    


## Exercice 4 — Comparer fichiers bruts et commandes haut niveau

Nous comparons :

```bash
cat /proc/meminfo | head
free -h
```

Puis :

```bash
cat /proc/loadavg
uptime
```

Puis :

```bash
cat /proc/modules | head
lsmod | head
```

Nous expliquons ce que les commandes haut niveau ajoutent par rapport aux fichiers bruts.


## Exercice 5 — Construire une synthèse système

Nous écrivons un script qui affiche :

```text
Kernel
CPU logiques
Mémoire totale
Mémoire disponible
Swap utilisé
Uptime
Load average
Nombre de processus visibles
```

Nous utilisons uniquement `/proc` autant que possible.


## Conclusion du chapitre 3

Nous savons maintenant lire les principales informations globales exposées par `/proc`.

Nous avons étudié :

- `/proc/cpuinfo` pour le processeur ;
    
- `/proc/meminfo` pour la mémoire ;
    
- `/proc/version` pour le noyau ;
    
- `/proc/uptime` pour le temps de fonctionnement ;
    
- `/proc/loadavg` pour la charge système ;
    
- `/proc/cmdline` pour les paramètres de démarrage ;
    
- `/proc/filesystems` pour les systèmes de fichiers supportés ;
    
- `/proc/mounts` pour les montages actifs ;
    
- `/proc/swaps` pour le swap ;
    
- `/proc/partitions` pour les périphériques bloc ;
    
- `/proc/modules` pour les modules noyau ;
    
- `/proc/interrupts` et `/proc/stat` pour des statistiques plus avancées.
    

Nous retenons surtout que ces fichiers donnent une première lecture de l’état du système. Ils sont indispensables pour diagnostiquer rapidement une machine, mais ils doivent être interprétés avec prudence, surtout dans les environnements virtualisés ou conteneurisés.

Dans le chapitre suivant, nous passons de la vue globale à la vue par processus avec les répertoires `/proc/<PID>`.

# Chapitre 4 — `/proc` et les processus

## Objectifs du chapitre

Dans ce chapitre, nous passons de la vision globale du système à l’observation détaillée des processus.

Nous avons vu que `/proc` expose des informations générales sur le noyau, la mémoire, le CPU ou la charge système. Nous étudions maintenant les répertoires de la forme :

```bash
/proc/<PID>
```

Chaque processus actif possède son propre répertoire dans `/proc`. Ce répertoire nous permet d’observer son état, sa ligne de commande, ses variables d’environnement, son utilisateur, ses threads, ses signaux, ses limites, ses fichiers ouverts et son contexte d’exécution.

À la fin de ce chapitre, nous savons :

- identifier les processus visibles dans `/proc` ;
    
- comprendre la structure générale de `/proc/<PID>` ;
    
- lire les fichiers `cmdline`, `comm`, `status` et `stat` ;
    
- interpréter les principaux états d’un processus ;
    
- lire l’environnement d’un processus avec `environ` ;
    
- comprendre les liens `cwd`, `exe` et `root` ;
    
- faire le lien entre `/proc` et les commandes `ps`, `top` et `htop` ;
    
- écrire des commandes simples pour analyser un processus.
    


## 4.1. Les répertoires par PID

### 4.1.1. Rappel : qu’est-ce qu’un PID ?

Un processus est une instance d’un programme en cours d’exécution.

Chaque processus possède un identifiant numérique appelé PID, pour Process Identifier.

Nous pouvons afficher le PID du shell courant avec :

```bash
echo $$
```

Exemple :

```text
18542
```

Cela signifie que notre shell a pour PID `18542`.

Dans `/proc`, nous pouvons alors observer :

```bash
ls /proc/18542
```

ou directement :

```bash
ls /proc/$$
```

Nous constatons que chaque processus actif possède un répertoire nommé par son PID.


### 4.1.2. Lister les processus visibles via `/proc`

Pour lister les répertoires numériques de `/proc`, nous utilisons :

```bash
ls /proc | grep -E '^[0-9]+$'
```

Chaque ligne correspond à un processus visible dans notre namespace PID.

Pour compter ces processus :

```bash
ls /proc | grep -E '^[0-9]+$' | wc -l
```

Nous obtenons une estimation du nombre de processus visibles.

Nous parlons d’estimation car l’état change pendant que nous observons le système : des processus peuvent apparaître ou disparaître entre deux commandes.


### 4.1.3. Observer le processus `1`

Le processus `1` occupe une place particulière.

Nous pouvons l’observer avec :

```bash
cat /proc/1/comm
```

Sur une distribution moderne avec systemd, nous obtenons souvent :

```text
systemd
```

Mais dans un conteneur, nous pouvons obtenir autre chose, par exemple :

```text
bash
node
python
tini
```

Le PID `1` est le premier processus visible dans le namespace courant.

Sur un système classique, il correspond au processus d’initialisation. Dans un conteneur, il correspond souvent au processus principal du conteneur.

Nous retenons donc :

```text
/proc/1 ne signifie pas toujours “le vrai PID 1 de la machine physique”.
Il signifie “le PID 1 dans le namespace PID courant”.
```


## 4.2. Structure générale de `/proc/<PID>`

### 4.2.1. Explorer un processus

Nous choisissons d’abord un processus simple.

Nous lançons :

```bash
sleep 1000 &
```

Puis nous récupérons son PID :

```bash
pid=$!
echo "$pid"
```

Nous explorons ensuite son répertoire :

```bash
ls -l /proc/$pid
```

Nous trouvons de nombreuses entrées, par exemple :

```text
cmdline
comm
cwd
environ
exe
fd
limits
maps
root
stat
status
task
```

Chaque entrée expose une information différente.


### 4.2.2. Les catégories principales

Nous pouvons regrouper les entrées de `/proc/<PID>` en plusieurs catégories.

|Entrée|Rôle|
|---|---|
|`cmdline`|ligne de commande complète|
|`comm`|nom court du processus|
|`status`|état lisible du processus|
|`stat`|statistiques compactes bas niveau|
|`environ`|variables d’environnement|
|`cwd`|répertoire courant|
|`exe`|exécutable associé|
|`root`|racine vue par le processus|
|`fd`|descripteurs de fichiers ouverts|
|`maps`|cartographie mémoire|
|`limits`|limites de ressources|
|`task`|threads du processus|

Dans ce chapitre, nous nous concentrons surtout sur `cmdline`, `comm`, `status`, `stat`, `environ`, `cwd`, `exe` et `root`.

Les entrées `fd`, `maps`, `smaps` et `limits` sont approfondies dans les chapitres suivants.


## 4.3. `cmdline` : la ligne de commande du processus

### 4.3.1. Lire `/proc/<PID>/cmdline`

Le fichier `cmdline` contient la ligne de commande utilisée pour lancer le processus.

Exemple :

```bash
cat /proc/$pid/cmdline
```

Pour notre processus `sleep`, la sortie peut sembler étrange :

```text
sleep1000
```

En réalité, les arguments ne sont pas séparés par des espaces, mais par des caractères nuls `\0`.

C’est pour cela que l’affichage brut avec `cat` n’est pas toujours lisible.


### 4.3.2. Affichage lisible de `cmdline`

Nous pouvons remplacer les caractères nuls par des espaces :

```bash
tr '\0' ' ' < /proc/$pid/cmdline
echo
```

Sortie attendue :

```text
sleep 1000
```

Nous pouvons aussi afficher un argument par ligne :

```bash
tr '\0' '\n' < /proc/$pid/cmdline
```

Sortie :

```text
sleep
1000
```

Cette représentation est plus fidèle : chaque ligne correspond à un argument.


### 4.3.3. Cas des processus noyau

Certains processus ont une ligne de commande vide.

Par exemple, certains threads noyau peuvent apparaître avec une `cmdline` vide.

Nous pouvons rencontrer :

```bash
cat /proc/<PID>/cmdline
```

qui ne produit aucune sortie.

Cela ne signifie pas nécessairement que le processus n’existe pas. Cela peut signifier qu’il ne possède pas de ligne de commande utilisateur classique.


### 4.3.4. Risque de sécurité

La ligne de commande peut contenir des informations sensibles.

Exemples de mauvaises pratiques :

```bash
mysql -u user -pMonMotDePasse
curl -H "Authorization: Bearer TOKEN_SECRET" https://example.org
python app.py --api-key SECRET
```

Si ces secrets sont passés en arguments, ils peuvent se retrouver visibles dans :

```bash
/proc/<PID>/cmdline
```

et parfois dans les sorties de :

```bash
ps aux
```

Nous retenons une règle importante :

```text
Nous évitons de passer des secrets en arguments de ligne de commande.
```

Nous privilégions des mécanismes plus sûrs : fichiers de secrets avec permissions strictes, gestionnaires de secrets, variables d’environnement maîtrisées ou sockets protégées selon le contexte.


## 4.4. `comm` : le nom court du processus

### 4.4.1. Lire `/proc/<PID>/comm`

Le fichier `comm` contient le nom court du processus.

```bash
cat /proc/$pid/comm
```

Exemple :

```text
sleep
```

Ce nom est généralement court et ne contient pas toute la ligne de commande.


### 4.4.2. Différence entre `comm` et `cmdline`

Nous comparons :

```bash
cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline
echo
```

Sortie possible :

```text
sleep
sleep 1000
```

`comm` donne le nom du processus.

`cmdline` donne la commande complète avec ses arguments.

La différence est importante.

Un processus Python peut avoir :

```text
comm = python
cmdline = python manage.py runserver 0.0.0.0:8000
```

Dans un diagnostic, `cmdline` est souvent plus utile pour identifier précisément le rôle du processus.


## 4.5. `status` : état lisible du processus

### 4.5.1. Lire `/proc/<PID>/status`

Le fichier `status` est l’un des fichiers les plus utiles pour lire rapidement les informations d’un processus.

```bash
cat /proc/$pid/status
```

Extrait possible :

```text
Name:   sleep
Umask:  0022
State:  S (sleeping)
Tgid:   18590
Pid:    18590
PPid:   18420
Uid:    1000    1000    1000    1000
Gid:    1000    1000    1000    1000
FDSize: 64
Threads:        1
VmPeak:     5788 kB
VmSize:     5788 kB
VmRSS:       912 kB
SigQ:   0/30995
SigPnd: 0000000000000000
SigBlk: 0000000000000000
SigIgn: 0000000000000000
SigCgt: 0000000000000000
```

Ce fichier est plus lisible que `stat`, car les champs sont nommés.


### 4.5.2. Les champs d’identification

Nous trouvons notamment :

```text
Name
State
Tgid
Pid
PPid
Uid
Gid
Threads
```

Nous les interprétons ainsi :

|Champ|Signification|
|---|---|
|`Name`|nom court du processus|
|`State`|état courant|
|`Tgid`|identifiant du groupe de threads|
|`Pid`|identifiant du processus ou thread|
|`PPid`|PID du processus parent|
|`Uid`|identifiants utilisateur|
|`Gid`|identifiants groupe|
|`Threads`|nombre de threads|

Le champ `PPid` est particulièrement utile pour comprendre qui a lancé un processus.


### 4.5.3. Lire seulement les champs utiles

Nous pouvons extraire les lignes principales :

```bash
grep -E '^(Name|State|Pid|PPid|Uid|Gid|Threads):' /proc/$pid/status
```

Exemple :

```text
Name:   sleep
State:  S (sleeping)
Pid:    18590
PPid:   18420
Uid:    1000    1000    1000    1000
Gid:    1000    1000    1000    1000
Threads:        1
```

Cette commande est utile pour un premier diagnostic.


### 4.5.4. Les champs mémoire dans `status`

Le fichier `status` contient aussi des informations mémoire :

```bash
grep -E '^(VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
```

Nous pouvons obtenir :

```text
VmPeak:     5788 kB
VmSize:     5788 kB
VmRSS:       912 kB
RssAnon:      92 kB
RssFile:     820 kB
RssShmem:      0 kB
```

Nous retenons seulement une idée générale pour l’instant :

- `VmSize` représente la taille de l’espace mémoire virtuel ;
    
- `VmRSS` représente la mémoire résidente en RAM ;
    
- `RssAnon`, `RssFile` et `RssShmem` détaillent une partie de cette mémoire.
    

Nous approfondissons ces notions dans le chapitre sur la mémoire des processus.


## 4.6. Les états des processus

### 4.6.1. Lire l’état dans `status`

L’état du processus se lit dans :

```bash
grep '^State:' /proc/$pid/status
```

Exemple :

```text
State:  S (sleeping)
```

Le processus `sleep` est naturellement en état sleeping, car il attend l’écoulement du temps.


### 4.6.2. Principaux états

Nous rencontrons principalement les états suivants :

|Code|Nom|Signification|
|---|---|---|
|`R`|running|processus en cours d’exécution ou prêt à s’exécuter|
|`S`|sleeping|sommeil interruptible, attente normale|
|`D`|uninterruptible sleep|attente non interruptible, souvent I/O|
|`Z`|zombie|processus terminé mais non récolté par son parent|
|`T`|stopped|processus arrêté, par exemple avec `SIGSTOP`|
|`t`|tracing stop|processus arrêté car tracé ou debuggué|
|`I`|idle|thread noyau idle|


### 4.6.3. État `R`

Un processus en état `R` est soit en train de s’exécuter sur un CPU, soit prêt à s’exécuter.

Il est dans la file des tâches exécutables.

Exemple d’observation :

```bash
ps -eo pid,state,comm | awk '$2=="R"'
```

Cet état est normal si le processus travaille réellement.


### 4.6.4. État `S`

L’état `S` est très courant.

Il signifie que le processus dort, mais peut être réveillé par un signal ou un événement.

Exemples :

- un shell qui attend une commande ;
    
- un serveur qui attend une connexion ;
    
- un processus `sleep` ;
    
- une application qui attend une réponse réseau.
    

Cet état n’est pas inquiétant en soi.


### 4.6.5. État `D`

L’état `D` signifie un sommeil non interruptible.

Il apparaît souvent lorsqu’un processus attend une opération d’entrée/sortie noyau : disque, réseau bas niveau, système de fichiers distant, périphérique bloqué.

Un processus en état `D` peut être difficile à tuer, car il attend que le noyau termine l’opération.

Si nous observons beaucoup de processus en état `D`, nous suspectons souvent :

- un disque lent ou bloqué ;
    
- un montage NFS problématique ;
    
- un périphérique défaillant ;
    
- une saturation I/O ;
    
- un bug pilote ;
    
- un problème de stockage distant.
    

Nous pouvons chercher ces processus avec :

```bash
ps -eo pid,state,comm | awk '$2=="D"'
```


### 4.6.6. État `Z`

Un processus zombie est un processus terminé dont le parent n’a pas encore récupéré le code de sortie.

Il ne consomme presque plus de mémoire utile, mais il occupe encore une entrée dans la table des processus.

Nous pouvons les lister avec :

```bash
ps -eo pid,ppid,state,comm | awk '$3=="Z"'
```

Dans `/proc`, un zombie possède encore un répertoire `/proc/<PID>`, mais beaucoup d’informations ne sont plus disponibles comme pour un processus vivant.

Un zombie isolé n’est pas forcément grave. Beaucoup de zombies persistants peuvent indiquer un bug dans le processus parent.


### 4.6.7. État `T`

L’état `T` indique qu’un processus est arrêté.

Nous pouvons produire cet état avec :

```bash
sleep 1000 &
pid=$!
kill -STOP $pid
grep '^State:' /proc/$pid/status
```

Sortie possible :

```text
State:  T (stopped)
```

Nous pouvons le reprendre avec :

```bash
kill -CONT $pid
grep '^State:' /proc/$pid/status
```

Puis le terminer :

```bash
kill $pid
```

Nous voyons ici le lien entre signaux Unix et états de processus.


## 4.7. `stat` : statistiques compactes bas niveau

### 4.7.1. Lire `/proc/<PID>/stat`

Le fichier `stat` contient de nombreuses informations compactes sur le processus.

```bash
cat /proc/$pid/stat
```

Exemple simplifié :

```text
18590 (sleep) S 18420 18590 18420 34816 18590 4194304 ...
```

Ce fichier est moins lisible que `status`, mais il est utilisé par de nombreux outils système.


### 4.7.2. Les premiers champs

Les premiers champs sont particulièrement importants :

|Position|Champ|Signification|
|---|---|---|
|1|`pid`|PID|
|2|`comm`|nom du processus entre parenthèses|
|3|`state`|état|
|4|`ppid`|PID du parent|
|5|`pgrp`|groupe de processus|
|6|`session`|session|

Pour les extraire rapidement, nous pouvons faire :

```bash
awk '{ print "pid=" $1, "comm=" $2, "state=" $3, "ppid=" $4 }' /proc/$pid/stat
```

Exemple :

```text
pid=18590 comm=(sleep) state=S ppid=18420
```


### 4.7.3. Attention au parsing de `stat`

Le parsing de `/proc/<PID>/stat` est piégeux.

Le champ `comm` est entre parenthèses et peut contenir des espaces ou des caractères particuliers.

Une méthode naïve qui découpe simplement sur les espaces peut être incorrecte dans certains cas.

Pour un cours d’introduction, nous retenons :

```text
Nous utilisons /proc/<PID>/status pour la lecture humaine.
Nous utilisons /proc/<PID>/stat avec prudence pour les outils bas niveau.
```


## 4.8. `environ` : environnement du processus

### 4.8.1. Lire `/proc/<PID>/environ`

Le fichier `environ` contient les variables d’environnement du processus.

```bash
cat /proc/$pid/environ
```

Comme pour `cmdline`, les entrées sont séparées par des caractères nuls.

Nous les rendons lisibles avec :

```bash
tr '\0' '\n' < /proc/$pid/environ
```

Exemple :

```text
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOME=/home/gregoire
LANG=fr_FR.UTF-8
SHELL=/bin/bash
```


### 4.8.2. À quoi servent les variables d’environnement ?

Les variables d’environnement servent à transmettre des paramètres au processus.

Exemples courants :

```text
PATH
HOME
USER
LANG
SHELL
PWD
JAVA_HOME
NODE_ENV
PYTHONPATH
DATABASE_URL
```

Elles influencent le comportement des programmes.

Par exemple :

```bash
NODE_ENV=production node server.js
```

ou :

```bash
LANG=C sort fichier.txt
```


### 4.8.3. Implications de sécurité

`environ` peut contenir des secrets.

Exemples :

```text
DATABASE_URL=postgresql://user:password@host/db
AWS_SECRET_ACCESS_KEY=...
GITHUB_TOKEN=...
JWT_SECRET=...
```

Si un utilisateur non autorisé peut lire l’environnement d’un processus, il peut récupérer des informations sensibles.

C’est pourquoi les permissions, les options de montage de `/proc` et les pratiques de gestion des secrets sont importantes.

Nous retenons :

```text
Les variables d’environnement sont pratiques, mais elles ne sont pas magiquement secrètes.
```


### 4.8.4. Permissions

La lecture de `environ` peut échouer :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

Erreur possible :

```text
Permission denied
```

Cela dépend :

- de l’utilisateur propriétaire du processus ;
    
- des permissions ;
    
- de la configuration du noyau ;
    
- des options de montage de `/proc` ;
    
- de mécanismes de sécurité comme Yama, AppArmor ou SELinux selon le système.
    

Nous devons interpréter cette erreur comme une protection normale, pas comme un dysfonctionnement.


## 4.9. `cwd`, `exe` et `root`

### 4.9.1. Présentation

Dans `/proc/<PID>`, nous trouvons trois liens symboliques très importants :

```bash
ls -l /proc/$pid/cwd
ls -l /proc/$pid/exe
ls -l /proc/$pid/root
```

Ils nous renseignent sur le contexte d’exécution du processus.

|Lien|Signification|
|---|---|
|`cwd`|répertoire de travail courant|
|`exe`|exécutable associé au processus|
|`root`|racine du système de fichiers vue par le processus|


### 4.9.2. `cwd` : répertoire courant

Le lien `cwd` pointe vers le répertoire courant du processus.

Exemple :

```bash
ls -l /proc/$$/cwd
```

Sortie possible :

```text
/proc/18542/cwd -> /home/gregoire
```

Si nous changeons de répertoire :

```bash
cd /tmp
ls -l /proc/$$/cwd
```

Nous voyons que `cwd` change.

Cette information est utile lorsqu’un processus semble utiliser des chemins relatifs ou écrire dans un emplacement inattendu.


### 4.9.3. `exe` : exécutable lancé

Le lien `exe` pointe vers le binaire exécuté.

```bash
ls -l /proc/$pid/exe
```

Exemple :

```text
/proc/18590/exe -> /usr/bin/sleep
```

C’est très utile pour savoir quel binaire réel est exécuté.

Deux processus peuvent avoir le même nom court, mais ne pas venir du même chemin.

Exemple :

```text
/usr/bin/python3
/home/user/venv/bin/python
/opt/app/custom-python
```

Le lien `exe` aide à lever cette ambiguïté.


### 4.9.4. Cas d’un exécutable supprimé

Il arrive qu’un processus continue d’exécuter un binaire qui a été supprimé ou remplacé sur disque.

Nous pouvons alors voir :

```text
/proc/<PID>/exe -> /usr/bin/app (deleted)
```

Cela signifie que le processus utilise encore l’ancien exécutable chargé en mémoire.

Ce cas est fréquent après une mise à jour : un service doit être redémarré pour utiliser la nouvelle version du binaire.


### 4.9.5. `root` : racine vue par le processus

Le lien `root` pointe vers la racine du système de fichiers vue par le processus.

```bash
ls -l /proc/$pid/root
```

Sur un système normal, nous obtenons souvent :

```text
/proc/18590/root -> /
```

Mais dans un `chroot` ou un conteneur, ce lien peut donner une vue différente.

Ce lien est très important pour comprendre l’isolation.

Un processus peut croire que sa racine est `/`, alors que cette racine correspond en réalité à un sous-arbre spécifique du système hôte.


### 4.9.6. Lien avec les conteneurs

Dans un conteneur, `root` permet de comprendre quelle arborescence le processus voit.

Depuis l’hôte, si nous avons les permissions nécessaires, nous pouvons parfois inspecter :

```bash
ls /proc/<PID_CONTENEUR>/root
```

Cela donne accès à la racine du conteneur telle qu’elle est vue par le processus.

Cette capacité est puissante pour le diagnostic, mais elle doit être comprise du point de vue sécurité.


## 4.10. `task` : les threads du processus

### 4.10.1. Observer les threads

Le dossier `task` contient les threads du processus.

```bash
ls /proc/$pid/task
```

Pour un processus simple comme `sleep`, nous obtenons généralement un seul identifiant :

```text
18590
```

Pour un processus multi-thread, nous pouvons obtenir plusieurs entrées :

```text
20001
20002
20003
20004
```

Chaque entrée correspond à un thread.


### 4.10.2. Processus et threads dans Linux

Linux représente les threads comme des tâches partageant certaines ressources.

Chaque thread possède un identifiant appelé TID.

Dans `/proc`, nous trouvons :

```text
/proc/<PID>/task/<TID>
```

Chaque thread possède son propre fichier `status`, `stat`, etc.

Exemple :

```bash
for t in /proc/$pid/task/*; do
    echo "Thread ${t##*/}"
    grep '^State:' "$t/status"
done
```


### 4.10.3. Utilité

Cette observation est utile pour :

- analyser une application multi-thread ;
    
- comprendre le nombre réel de threads ;
    
- observer des threads bloqués ;
    
- diagnostiquer une saturation ;
    
- vérifier le comportement d’un serveur Java, Node.js, Python, PostgreSQL ou autre.
    

Nous approfondissons cette dimension lorsque nous parlons du scheduling, des performances et de la mémoire.


## 4.11. Lien entre `/proc` et `ps`

### 4.11.1. Observer avec `ps`

La commande `ps` affiche les processus.

Exemple :

```bash
ps -p $pid -o pid,ppid,state,comm,args
```

Sortie possible :

```text
    PID    PPID S COMMAND         COMMAND
  18590   18420 S sleep           sleep 1000
```

Nous pouvons retrouver ces informations dans `/proc`.

|Information `ps`|Source possible dans `/proc`|
|---|---|
|PID|nom du répertoire `/proc/<PID>`|
|PPID|`/proc/<PID>/status` ou `stat`|
|état|`/proc/<PID>/status` ou `stat`|
|commande|`/proc/<PID>/comm`|
|arguments|`/proc/<PID>/cmdline`|


### 4.11.2. Reproduire une mini-version de `ps`

Nous pouvons écrire une commande simple :

```bash
for dir in /proc/[0-9]*; do
    pid=${dir#/proc/}
    [ -r "$dir/status" ] || continue

    name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    state=$(awk '/^State:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    ppid=$(awk '/^PPid:/ {print $2}' "$dir/status" 2>/dev/null) || continue

    printf "%8s %8s %2s %s\n" "$pid" "$ppid" "$state" "$name"
done | head
```

Sortie possible :

```text
       1        0  S systemd
     532        1  S systemd-journal
     610        1  S NetworkManager
    18542     1840 S bash
    18590    18542 S sleep
```

Nous venons de reproduire une version très simplifiée de `ps`.


### 4.11.3. Pourquoi `ps` est plus robuste

Notre script est pédagogique, mais `ps` est beaucoup plus robuste.

Il gère :

- les processus qui disparaissent pendant la lecture ;
    
- les formats précis ;
    
- les permissions ;
    
- les threads ;
    
- les options de tri ;
    
- les colonnes ;
    
- les différences de noyau ;
    
- les performances.
    

L’objectif n’est donc pas de remplacer `ps`, mais de comprendre d’où viennent ses informations.


## 4.12. Lien entre `/proc`, `top` et `htop`

### 4.12.1. `top`

La commande :

```bash
top
```

affiche dynamiquement :

- les processus ;
    
- leur consommation CPU ;
    
- leur consommation mémoire ;
    
- leur état ;
    
- la charge système ;
    
- le nombre de tâches ;
    
- le swap ;
    
- la mémoire.
    

Une grande partie de ces informations provient de `/proc`, notamment :

```text
/proc/stat
/proc/meminfo
/proc/loadavg
/proc/<PID>/stat
/proc/<PID>/status
```


### 4.12.2. `htop`

`htop` offre une interface interactive plus confortable.

Il présente :

- une barre CPU ;
    
- une barre mémoire ;
    
- une liste des processus ;
    
- les arbres de processus ;
    
- les threads ;
    
- les commandes complètes ;
    
- les signaux envoyables depuis l’interface.
    

Mais conceptuellement, il reste lié aux informations exposées par le noyau, souvent via `/proc`.


### 4.12.3. Observer les lectures avec `strace`

Nous pouvons observer ce que lit une commande avec `strace`.

Exemple :

```bash
strace -e openat ps -p $pid -o pid,ppid,state,comm,args
```

Nous pouvons voir des ouvertures de fichiers sous `/proc`.

Cette observation montre que les outils haut niveau ne sont pas magiques : ils lisent, agrègent et formattent des informations.


## 4.13. Cas pratiques

### 4.13.1. Identifier précisément un processus

Nous voulons savoir ce qu’est un processus.

Nous utilisons :

```bash
pid=1234

cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Uid|Gid|Threads):' /proc/$pid/status
ls -l /proc/$pid/exe
ls -l /proc/$pid/cwd
```

Nous obtenons :

- son nom court ;
    
- sa commande complète ;
    
- son état ;
    
- son parent ;
    
- son utilisateur ;
    
- son nombre de threads ;
    
- son exécutable ;
    
- son répertoire courant.
    

C’est une base de diagnostic très solide.


### 4.13.2. Trouver le parent d’un processus

Nous lisons :

```bash
grep '^PPid:' /proc/$pid/status
```

Exemple :

```text
PPid:   18420
```

Puis :

```bash
cat /proc/18420/comm
tr '\0' ' ' < /proc/18420/cmdline; echo
```

Nous comprenons alors qui a lancé le processus.

Nous pouvons aussi utiliser :

```bash
pstree -p
```

mais l’objectif est ici de comprendre la source dans `/proc`.


### 4.13.3. Identifier un processus bloqué

Nous cherchons les processus en état `D` :

```bash
for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue
    state=$(awk '/^State:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    if [ "$state" = "D" ]; then
        pid=${dir#/proc/}
        name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null)
        echo "$pid $state $name"
    fi
done
```

Si nous en trouvons beaucoup, nous suspectons une attente I/O ou un problème noyau/périphérique.


### 4.13.4. Voir les processus d’un utilisateur

Nous pouvons lire les UID dans `status`.

Exemple :

```bash
for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue
    pid=${dir#/proc/}
    uid=$(awk '/^Uid:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    printf "%8s %8s %s\n" "$pid" "$uid" "$name"
done | head
```

Le premier UID affiché est l’UID réel du processus.

Pour convertir un UID en nom d’utilisateur :

```bash
getent passwd 1000
```


## 4.14. Pièges et limites

### 4.14.1. Le processus peut disparaître

Entre deux lectures, un processus peut se terminer.

Exemple :

```bash
cat /proc/$pid/status
```

peut échouer avec :

```text
No such file or directory
```

Cela signifie souvent que le processus n’existe plus.

Un script robuste doit l’accepter.


### 4.14.2. Les permissions peuvent bloquer la lecture

Certains fichiers peuvent être illisibles :

```bash
cat /proc/$pid/environ
```

peut produire :

```text
Permission denied
```

C’est normal si nous n’avons pas les droits nécessaires.

Nous devons distinguer :

- processus inexistant ;
    
- processus existant mais inaccessible ;
    
- fichier absent ;
    
- fichier lisible mais vide.
    


### 4.14.3. Les conteneurs changent la vue

Dans un conteneur, les PID visibles peuvent être différents de ceux de l’hôte.

Un processus peut avoir :

- un PID dans le conteneur ;
    
- un autre PID sur l’hôte.
    

Le répertoire `/proc` reflète le namespace PID du processus qui le lit.

Nous devons donc toujours demander :

```text
Depuis quel contexte observons-nous /proc ?
Depuis l’hôte ?
Depuis un conteneur ?
Depuis un chroot ?
Depuis un namespace particulier ?
```


### 4.14.4. Ne pas confondre nom et exécutable

Le nom affiché dans `comm` ne suffit pas toujours.

Un processus peut s’appeler :

```text
python
node
java
bash
```

mais ce nom ne dit pas quelle application est réellement lancée.

Nous devons regarder :

```bash
/proc/<PID>/cmdline
/proc/<PID>/exe
/proc/<PID>/cwd
```

pour comprendre précisément le processus.


## 4.15. Exercices

## Exercice 1 — Observer le shell courant

Nous exécutons :

```bash
echo $$
cat /proc/$$/comm
tr '\0' ' ' < /proc/$$/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Uid|Gid|Threads):' /proc/$$/status
ls -l /proc/$$/cwd
ls -l /proc/$$/exe
ls -l /proc/$$/root
```

Nous répondons aux questions :

1. Quel est le PID du shell ?
    
2. Quel est son nom court ?
    
3. Quelle est sa ligne de commande ?
    
4. Quel est son processus parent ?
    
5. Quel est son répertoire courant ?
    
6. Quel binaire exécute-t-il ?
    


## Exercice 2 — Lancer et analyser un processus

Nous lançons :

```bash
sleep 300 &
pid=$!
```

Nous analysons :

```bash
cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Threads):' /proc/$pid/status
ls -l /proc/$pid/exe
```

Puis nous terminons :

```bash
kill $pid
```

Nous vérifions :

```bash
ls /proc/$pid
```

Nous expliquons pourquoi le répertoire disparaît.


## Exercice 3 — Observer les états

Nous lançons :

```bash
sleep 1000 &
pid=$!
```

Nous lisons son état :

```bash
grep '^State:' /proc/$pid/status
```

Puis nous l’arrêtons :

```bash
kill -STOP $pid
grep '^State:' /proc/$pid/status
```

Nous le reprenons :

```bash
kill -CONT $pid
grep '^State:' /proc/$pid/status
```

Puis nous le terminons :

```bash
kill $pid
```

Nous expliquons les états observés.


## Exercice 4 — Lire les variables d’environnement

Nous observons l’environnement du shell courant :

```bash
tr '\0' '\n' < /proc/$$/environ | head
```

Nous répondons aux questions :

1. Quelles variables reconnaissons-nous ?
    
2. À quoi servent `PATH`, `HOME`, `LANG` ?
    
3. Pourquoi peut-il être dangereux d’y stocker des secrets ?
    
4. Pourquoi cette lecture peut-elle être refusée pour certains processus ?
    


## Exercice 5 — Mini-version de `ps`

Nous écrivons un script `mini-ps.sh` :

```bash
##!/usr/bin/env bash

printf "%8s %8s %-2s %s\n" "PID" "PPID" "S" "NAME"

for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue

    pid=${dir#/proc/}

    name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    state=$(awk '/^State:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    ppid=$(awk '/^PPid:/ {print $2}' "$dir/status" 2>/dev/null) || continue

    printf "%8s %8s %-2s %s\n" "$pid" "$ppid" "$state" "$name"
done
```

Nous le rendons exécutable :

```bash
chmod +x mini-ps.sh
./mini-ps.sh | head
```

Nous comparons avec :

```bash
ps -eo pid,ppid,state,comm | head
```


## 4.16. Ce que nous devons retenir

Nous retenons les points suivants :

1. Chaque processus actif possède un répertoire `/proc/<PID>`.
    
2. Les répertoires numériques de `/proc` représentent les processus visibles dans le namespace courant.
    
3. `/proc/1` représente le PID 1 du namespace courant, pas forcément celui de l’hôte physique.
    
4. `cmdline` donne la ligne de commande complète, avec des arguments séparés par `\0`.
    
5. `comm` donne le nom court du processus.
    
6. `status` donne une vue lisible et très utile du processus.
    
7. `stat` donne une vue compacte et bas niveau, mais plus difficile à parser correctement.
    
8. Les états `R`, `S`, `D`, `Z` et `T` sont essentiels pour le diagnostic.
    
9. `environ` expose les variables d’environnement et peut contenir des informations sensibles.
    
10. `cwd`, `exe` et `root` décrivent le contexte d’exécution du processus.
    
11. `task` permet d’observer les threads.
    
12. Les outils `ps`, `top` et `htop` s’appuient en partie sur ces informations.
    
13. Les scripts qui lisent `/proc/<PID>` doivent gérer les processus qui disparaissent et les permissions refusées.
    
14. Les conteneurs et namespaces modifient la vue que nous avons des processus.
    


## Conclusion du chapitre 4

Nous savons maintenant utiliser `/proc` pour analyser les processus.

Nous passons d’une vision globale du système à une vision détaillée par processus. Cette capacité est fondamentale pour l’administration système, le diagnostic applicatif, la cybersécurité et la compréhension fine de Linux.

Nous savons identifier un processus, lire sa commande, comprendre son état, retrouver son parent, observer son environnement, vérifier son exécutable et inspecter son contexte d’exécution.

Dans le chapitre suivant, nous approfondissons un aspect central du diagnostic : les fichiers ouverts et les descripteurs via `/proc/<PID>/fd`.

# Chapitre 5 — Fichiers ouverts et descripteurs

## Objectifs du chapitre

Dans ce chapitre, nous étudions les fichiers ouverts par les processus à travers :

```bash
/proc/<PID>/fd
```

Nous avons vu au chapitre précédent qu’un processus possède un répertoire dédié dans `/proc`. Nous allons maintenant analyser l’une des parties les plus utiles de ce répertoire : les descripteurs de fichiers.

Sous GNU/Linux, un processus ne manipule pas directement “un fichier” au sens abstrait. Il manipule des descripteurs de fichiers, c’est-à-dire de petits identifiants numériques qui pointent vers des ressources ouvertes.

Ces ressources peuvent être :

- des fichiers classiques ;
    
- des sockets réseau ;
    
- des pipes ;
    
- des terminaux ;
    
- des périphériques ;
    
- des fichiers supprimés mais encore ouverts ;
    
- des sockets Unix ;
    
- des pseudo-fichiers.
    

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est un descripteur de fichier ;
    
- comprendre le rôle de `0`, `1` et `2` ;
    
- explorer `/proc/<PID>/fd` ;
    
- identifier les fichiers ouverts par un processus ;
    
- repérer les sockets, pipes et périphériques ;
    
- comprendre pourquoi un fichier supprimé peut encore occuper de l’espace disque ;
    
- utiliser `/proc/<PID>/fd` pour diagnostiquer des incidents ;
    
- faire le lien avec `lsof`.
    


## 5.1. Notion de descripteur de fichier

### 5.1.1. Qu’est-ce qu’un descripteur de fichier ?

Un descripteur de fichier est un entier utilisé par un processus pour désigner une ressource ouverte.

Lorsqu’un programme ouvre un fichier avec un appel système comme :

```c
open()
```

le noyau retourne un entier.

Exemple conceptuel :

```text
open("/var/log/app.log") → 3
```

Le processus utilise ensuite ce nombre pour lire ou écrire :

```c
read(3, ...)
write(3, ...)
close(3)
```

Le processus ne manipule donc pas directement le chemin `/var/log/app.log` à chaque opération. Il manipule le descripteur `3`.


### 5.1.2. Pourquoi cette abstraction est importante ?

Cette abstraction permet au noyau d’unifier plusieurs types de ressources.

Le même mécanisme peut représenter :

```text
fichier classique
socket TCP
socket Unix
pipe
terminal
périphérique
pseudo-fichier
```

C’est une application directe de la philosophie Unix : beaucoup de ressources peuvent être manipulées comme des fichiers.

Ainsi, pour un processus, écrire dans un fichier, écrire dans un terminal ou écrire dans une socket repose sur une logique proche : il écrit dans un descripteur.


### 5.1.3. Exemple simple

Nous lançons un processus :

```bash
sleep 1000 &
pid=$!
```

Puis nous observons ses descripteurs :

```bash
ls -l /proc/$pid/fd
```

Sortie possible :

```text
total 0
lrwx------ 1 user user 64 mai 16 16:00 0 -> /dev/pts/2
lrwx------ 1 user user 64 mai 16 16:00 1 -> /dev/pts/2
lrwx------ 1 user user 64 mai 16 16:00 2 -> /dev/pts/2
```

Nous voyons trois descripteurs ouverts : `0`, `1` et `2`.


## 5.2. Les descripteurs standards : `0`, `1`, `2`

### 5.2.1. Présentation

Par convention, chaque processus possède généralement trois descripteurs ouverts au démarrage :

|Descripteur|Nom|Rôle|
|--:|---|---|
|`0`|entrée standard|lire les données d’entrée|
|`1`|sortie standard|écrire la sortie normale|
|`2`|sortie d’erreur|écrire les messages d’erreur|

Nous les appelons souvent :

```text
stdin
stdout
stderr
```


### 5.2.2. Entrée standard : `0`

Le descripteur `0` correspond à l’entrée standard.

Pour un programme interactif, elle pointe souvent vers le terminal.

Exemple :

```bash
ls -l /proc/$$/fd/0
```

Sortie possible :

```text
0 -> /dev/pts/2
```

Cela signifie que notre shell lit son entrée depuis le terminal `/dev/pts/2`.


### 5.2.3. Sortie standard : `1`

Le descripteur `1` correspond à la sortie standard.

Exemple :

```bash
ls -l /proc/$$/fd/1
```

Sortie possible :

```text
1 -> /dev/pts/2
```

Lorsque nous exécutons :

```bash
echo "bonjour"
```

le texte est écrit dans le descripteur `1`.


### 5.2.4. Sortie d’erreur : `2`

Le descripteur `2` correspond à la sortie d’erreur.

Exemple :

```bash
ls -l /proc/$$/fd/2
```

Sortie possible :

```text
2 -> /dev/pts/2
```

Les erreurs sont écrites séparément de la sortie normale.

C’est ce qui permet de rediriger la sortie standard et la sortie d’erreur différemment.


### 5.2.5. Redirections shell

Nous pouvons observer l’effet des redirections.

Exemple :

```bash
bash -c 'echo $$; sleep 1000' > /tmp/sortie.txt 2> /tmp/erreur.txt &
pid=$!
```

Puis :

```bash
ls -l /proc/$pid/fd
```

Nous pouvons voir :

```text
0 -> /dev/pts/2
1 -> /tmp/sortie.txt
2 -> /tmp/erreur.txt
```

Nous comprenons alors que les redirections shell ne sont pas magiques : elles changent les fichiers associés aux descripteurs du processus.


## 5.3. Le dossier `/proc/<PID>/fd`

### 5.3.1. Rôle du dossier `fd`

Le dossier :

```bash
/proc/<PID>/fd
```

contient un lien symbolique par descripteur ouvert.

Chaque lien porte le numéro du descripteur.

Exemple :

```bash
ls -l /proc/$pid/fd
```

Sortie possible :

```text
0 -> /dev/null
1 -> /var/log/app.log
2 -> /var/log/app-error.log
3 -> socket:[123456]
4 -> /tmp/data.txt
5 -> pipe:[654321]
```

Nous pouvons ainsi voir ce que le processus garde ouvert.


### 5.3.2. Lire un lien de descripteur

Pour lire la cible d’un descripteur :

```bash
readlink /proc/$pid/fd/1
```

Exemple :

```text
/var/log/app.log
```

Ou pour tous les descripteurs :

```bash
for fd in /proc/$pid/fd/*; do
    printf "%s -> %s\n" "${fd##*/}" "$(readlink "$fd")"
done
```

Cette commande affiche chaque numéro de descripteur avec sa cible.


### 5.3.3. Permissions

L’accès à `/proc/<PID>/fd` dépend des permissions.

Si le processus appartient à un autre utilisateur, nous pouvons rencontrer :

```text
Permission denied
```

ou ne pas voir toutes les informations.

Avec les droits administrateur, nous avons généralement une meilleure visibilité :

```bash
sudo ls -l /proc/<PID>/fd
```

Nous devons toutefois éviter d’utiliser `sudo` sans raison : l’objectif est de comprendre les permissions normales du système.


### 5.3.4. Processus qui disparaît

Comme toujours avec `/proc`, le processus peut se terminer pendant notre observation.

Une commande comme :

```bash
ls -l /proc/$pid/fd
```

peut échouer avec :

```text
No such file or directory
```

Cela signifie souvent que le processus n’existe plus.

Dans un script, nous devons gérer ce cas.


## 5.4. Types de ressources visibles dans `fd`

### 5.4.1. Fichiers classiques

Un descripteur peut pointer vers un fichier classique :

```text
4 -> /home/user/data.txt
```

Cela signifie que le processus a ouvert ce fichier.

Il peut l’avoir ouvert :

- en lecture ;
    
- en écriture ;
    
- en lecture-écriture ;
    
- avec un verrou ;
    
- en mode append ;
    
- via une bibliothèque.
    

Le lien dans `/proc/<PID>/fd` ne suffit pas toujours à connaître le mode exact d’ouverture. Pour cela, nous pouvons regarder `fdinfo`.


### 5.4.2. Sockets réseau

Un descripteur peut pointer vers une socket :

```text
3 -> socket:[123456]
```

Cela signifie que le processus possède une socket ouverte.

Le nombre entre crochets est un identifiant interne, souvent un inode de socket.

Nous pouvons chercher cette socket dans `/proc/net/tcp`, `/proc/net/udp` ou avec des outils comme :

```bash
ss -p
```

ou :

```bash
sudo lsof -i -P -n
```

Nous détaillons davantage le réseau dans le chapitre 7.


### 5.4.3. Pipes

Un descripteur peut pointer vers un pipe :

```text
5 -> pipe:[654321]
```

Un pipe permet de faire communiquer deux processus.

Exemple :

```bash
cat fichier.txt | grep erreur
```

Ici, la sortie de `cat` est reliée à l’entrée de `grep` par un pipe.

Dans `/proc/<PID>/fd`, nous pouvons voir ces connexions sous forme de `pipe:[...]`.


### 5.4.4. Terminaux

Un descripteur peut pointer vers un terminal :

```text
0 -> /dev/pts/2
1 -> /dev/pts/2
2 -> /dev/pts/2
```

`/dev/pts/2` représente ici un pseudo-terminal, souvent utilisé par un terminal graphique ou une connexion SSH.

Cela signifie que le processus est attaché à une session interactive.


### 5.4.5. Périphériques

Un descripteur peut pointer vers un périphérique :

```text
0 -> /dev/null
1 -> /dev/null
2 -> /dev/null
```

ou :

```text
3 -> /dev/ttyUSB0
```

ou encore :

```text
4 -> /dev/random
```

Cela peut être très utile pour diagnostiquer des services, des programmes embarqués, des outils série ou des programmes qui lisent des sources pseudo-aléatoires.


### 5.4.6. Fichiers anonymes et ressources spéciales

Nous pouvons aussi voir des ressources particulières :

```text
anon_inode:[eventfd]
anon_inode:[timerfd]
anon_inode:[eventpoll]
```

Ces entrées correspondent à des objets noyau anonymes utilisés pour des mécanismes avancés :

- événements ;
    
- timers ;
    
- epoll ;
    
- signalfd ;
    
- io_uring selon les cas.
    

Nous ne les approfondissons pas ici, mais nous devons savoir les reconnaître.


## 5.5. Le dossier `/proc/<PID>/fdinfo`

### 5.5.1. Rôle de `fdinfo`

À côté de `fd`, nous trouvons souvent :

```bash
/proc/<PID>/fdinfo
```

Ce dossier contient un fichier par descripteur.

Exemple :

```bash
ls /proc/$pid/fdinfo
```

Sortie possible :

```text
0  1  2  3
```

Pour lire les informations d’un descripteur :

```bash
cat /proc/$pid/fdinfo/3
```


### 5.5.2. Exemple de contenu

Sortie possible :

```text
pos:    128
flags:  0100002
mnt_id: 35
ino:    1234567
```

Nous trouvons notamment :

|Champ|Signification|
|---|---|
|`pos`|position courante dans le fichier|
|`flags`|flags d’ouverture|
|`mnt_id`|identifiant du montage|
|`ino`|inode associé|

Pour un fichier classique, `pos` indique où le processus se trouve dans le fichier.


### 5.5.3. Utilité de `pos`

Supposons qu’un processus lise progressivement un fichier.

Nous pouvons observer la position :

```bash
watch -n 1 "cat /proc/$pid/fdinfo/3"
```

Si `pos` augmente, le processus avance dans sa lecture ou son écriture.

Cela peut aider à savoir si un programme progresse ou reste bloqué.


### 5.5.4. Interpréter les flags

Les flags sont affichés sous forme numérique, souvent en octal.

Ils correspondent aux flags d’ouverture du fichier, par exemple :

- lecture seule ;
    
- écriture seule ;
    
- lecture-écriture ;
    
- append ;
    
- non-blocking ;
    
- close-on-exec.
    

L’interprétation exacte demande de connaître les constantes du noyau et de la libc.

Pour un diagnostic avancé, nous préférons souvent utiliser :

```bash
lsof
```

ou :

```bash
strace
```

mais `fdinfo` nous donne la source brute.


## 5.6. Cas classique : fichier supprimé mais encore ouvert

### 5.6.1. Le problème

Un problème très fréquent en administration système est le suivant :

1. un fichier volumineux est supprimé ;
    
2. `ls` ne le montre plus ;
    
3. pourtant l’espace disque n’est pas libéré ;
    
4. le disque reste plein ;
    
5. un processus garde encore le fichier ouvert.
    

Cela arrive souvent avec des fichiers de logs.

Exemple :

```text
/var/log/app.log
```

est supprimé, mais le service continue d’écrire dedans.


### 5.6.2. Pourquoi l’espace n’est pas libéré ?

Sous Linux, supprimer un fichier avec :

```bash
rm fichier.log
```

supprime le nom du fichier dans le répertoire.

Mais si un processus garde le fichier ouvert, les données restent associées à l’inode tant que le descripteur n’est pas fermé.

Le fichier n’a plus de nom visible dans l’arborescence, mais il existe encore tant qu’un processus le référence.

Nous pouvons résumer :

```text
Nom supprimé ≠ données immédiatement libérées si le fichier reste ouvert.
```

L’espace est libéré quand le dernier descripteur vers ce fichier est fermé.


### 5.6.3. Reproduire le cas en TP

Nous créons un fichier :

```bash
dd if=/dev/zero of=/tmp/gros-fichier.log bs=1M count=100
```

Nous lançons un processus qui garde le fichier ouvert :

```bash
tail -f /tmp/gros-fichier.log &
pid=$!
```

Nous supprimons le fichier :

```bash
rm /tmp/gros-fichier.log
```

Le fichier n’apparaît plus :

```bash
ls -lh /tmp/gros-fichier.log
```

Mais le processus le garde ouvert.

Nous observons :

```bash
ls -l /proc/$pid/fd | grep deleted
```

Sortie possible :

```text
3 -> /tmp/gros-fichier.log (deleted)
```


### 5.6.4. Retrouver les fichiers supprimés encore ouverts

Pour chercher globalement :

```bash
sudo find /proc/*/fd -lname '*deleted*' 2>/dev/null
```

Ou avec affichage plus lisible :

```bash
sudo sh -c '
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    case "$target" in
        *"(deleted)"*) echo "$fd -> $target" ;;
    esac
done
'
```

Nous pouvons ensuite identifier le PID à partir du chemin :

```text
/proc/1234/fd/3 -> /var/log/app.log (deleted)
```

Ici, le PID est `1234` et le descripteur est `3`.


### 5.6.5. Libérer l’espace

La solution propre consiste généralement à redémarrer ou recharger le service concerné.

Exemple :

```bash
sudo systemctl restart mon-service
```

ou :

```bash
sudo systemctl reload mon-service
```

selon le service.

Lorsque le processus ferme le descripteur, l’espace disque est libéré.

Dans certains cas, nous pouvons tronquer le descripteur directement, mais cette pratique doit être utilisée avec prudence :

```bash
sudo truncate -s 0 /proc/<PID>/fd/<FD>
```

Nous ne l’utilisons que si nous comprenons parfaitement ce que nous faisons, car nous modifions la ressource encore ouverte par le processus.


## 5.7. Diagnostic : “quel fichier ce processus écrit-il ?”

### 5.7.1. Observer les descripteurs

Supposons qu’un processus écrive beaucoup sur disque.

Nous voulons identifier ses fichiers ouverts.

Nous commençons par :

```bash
ls -l /proc/<PID>/fd
```

Nous cherchons des fichiers classiques :

```text
1 -> /var/log/app.log
4 -> /data/output.dat
5 -> /tmp/cache.tmp
```


### 5.7.2. Observer les positions avec `fdinfo`

Pour un fichier donné, nous regardons :

```bash
cat /proc/<PID>/fdinfo/<FD>
```

Puis nous observons dans le temps :

```bash
watch -n 1 'cat /proc/<PID>/fdinfo/<FD>'
```

Si `pos` augmente rapidement, le processus lit ou écrit dans ce fichier.


### 5.7.3. Croiser avec la taille du fichier

Nous pouvons regarder la taille du fichier :

```bash
ls -lh /proc/<PID>/fd/<FD>
```

ou directement la cible :

```bash
readlink /proc/<PID>/fd/<FD>
```

Puis :

```bash
ls -lh "$(readlink /proc/<PID>/fd/<FD>)"
```

Attention : cela ne fonctionne pas si le fichier est supprimé ou si le chemin contient des cas particuliers.


## 5.8. Diagnostic : “pourquoi mon service ne démarre-t-il pas ?”

### 5.8.1. Descripteurs et logs

Un service peut échouer parce qu’il ne peut pas ouvrir :

- un fichier de configuration ;
    
- un fichier de log ;
    
- une socket ;
    
- un certificat ;
    
- un périphérique ;
    
- un fichier temporaire.
    

Lorsque le service est encore actif, `/proc/<PID>/fd` peut nous montrer ce qu’il a réussi à ouvrir.

Exemple :

```bash
sudo ls -l /proc/<PID>/fd
```

Nous pouvons vérifier si le fichier attendu est bien ouvert.


### 5.8.2. Complément avec `strace`

Si le processus échoue immédiatement, `/proc/<PID>/fd` peut ne pas suffire, car le processus disparaît trop vite.

Dans ce cas, nous utilisons plutôt :

```bash
strace -e openat,close,read,write -f commande
```

`strace` permet d’observer les ouvertures de fichiers en temps réel.

Mais `/proc/<PID>/fd` reste utile pour les processus vivants.


## 5.9. Diagnostic : trop de fichiers ouverts

### 5.9.1. Symptôme

Un service peut échouer avec une erreur du type :

```text
Too many open files
```

Cela signifie qu’il a atteint une limite de descripteurs ouverts.

Nous pouvons compter les descripteurs ouverts :

```bash
ls /proc/<PID>/fd | wc -l
```

Exemple :

```text
1024
```


### 5.9.2. Lire les limites du processus

Nous regardons :

```bash
cat /proc/<PID>/limits
```

Nous cherchons :

```text
Max open files
```

Exemple :

```text
Max open files            1024                 1048576              files
```

Nous interprétons :

- limite souple ;
    
- limite dure ;
    
- unité.
    

Nous étudierons `limits` plus en détail au chapitre suivant, mais il est déjà utile ici.


### 5.9.3. Trouver une fuite de descripteurs

Si le nombre de descripteurs augmente continuellement, nous suspectons une fuite de descripteurs.

Nous pouvons observer :

```bash
watch -n 1 "ls /proc/<PID>/fd | wc -l"
```

Si le nombre monte sans redescendre, l’application ouvre des fichiers, sockets ou pipes sans les fermer correctement.

Nous pouvons inspecter :

```bash
ls -l /proc/<PID>/fd
```

pour voir quel type de ressource s’accumule.

Exemples :

```text
socket:[...]
pipe:[...]
/tmp/...
```

Cela oriente le diagnostic.


## 5.10. Sockets et correspondance avec le réseau

### 5.10.1. Identifier une socket

Dans `fd`, nous pouvons voir :

```text
7 -> socket:[4026532761]
```

Ce numéro correspond à une socket.

Pour savoir à quelle connexion elle correspond, nous utilisons généralement :

```bash
ss -tulpen
```

ou :

```bash
sudo ss -p
```


### 5.10.2. Utiliser `lsof`

Nous pouvons aussi utiliser :

```bash
sudo lsof -p <PID>
```

ou pour le réseau :

```bash
sudo lsof -Pan -p <PID> -i
```

Cela nous donne une vue plus lisible des sockets.

Mais il est important de comprendre que ces outils agrègent des informations que le noyau expose notamment via `/proc`.


## 5.11. Comparaison avec `lsof`

### 5.11.1. Rôle de `lsof`

`lsof` signifie “list open files”.

Il liste les fichiers ouverts par les processus.

Exemple :

```bash
sudo lsof -p <PID>
```

Sortie possible :

```text
COMMAND PID USER FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   932 root cwd  DIR  259,2     4096  128 /
nginx   932 root txt  REG  259,2  1260032  456 /usr/sbin/nginx
nginx   932 root 1w   REG  259,2   123456  789 /var/log/nginx/access.log
nginx   932 root 2w   REG  259,2    23456  790 /var/log/nginx/error.log
nginx   932 root 6u  IPv4 123456      0t0  TCP *:80 (LISTEN)
```


### 5.11.2. Ce que `lsof` apporte

`lsof` apporte une lecture plus riche :

- nom de commande ;
    
- PID ;
    
- utilisateur ;
    
- numéro de descripteur ;
    
- mode d’ouverture ;
    
- type de ressource ;
    
- périphérique ;
    
- inode ;
    
- nom lisible ;
    
- sockets réseau interprétées.
    

`/proc/<PID>/fd` est plus brut, mais disponible sur presque tous les systèmes Linux sans installer d’outil supplémentaire.


### 5.11.3. Quand utiliser `/proc` plutôt que `lsof` ?

Nous utilisons directement `/proc` lorsque :

- `lsof` n’est pas installé ;
    
- nous voulons comprendre la source brute ;
    
- nous écrivons un script léger ;
    
- nous diagnostiquons dans un conteneur minimal ;
    
- nous voulons observer un descripteur précis ;
    
- nous voulons accéder à un fichier supprimé mais encore ouvert.
    

Nous utilisons `lsof` lorsque :

- nous voulons une vue synthétique ;
    
- nous voulons filtrer rapidement ;
    
- nous voulons interpréter les sockets ;
    
- nous voulons une sortie plus humaine ;
    
- nous faisons un diagnostic interactif.
    

Les deux approches sont complémentaires.


## 5.12. Accéder au contenu via `/proc/<PID>/fd/<FD>`

### 5.12.1. Principe

Un lien comme :

```bash
/proc/<PID>/fd/3
```

peut parfois être utilisé comme chemin d’accès au fichier ouvert.

Exemple :

```bash
cat /proc/<PID>/fd/3
```

Cela lit le contenu du fichier associé au descripteur `3`, si nous avons les droits nécessaires.


### 5.12.2. Cas du fichier supprimé

C’est particulièrement utile lorsqu’un fichier a été supprimé mais reste ouvert.

Si nous avons :

```text
/proc/1234/fd/3 -> /tmp/data.log (deleted)
```

nous pouvons parfois récupérer son contenu :

```bash
cp /proc/1234/fd/3 /tmp/data-recupered.log
```

Cela peut sauver un fichier supprimé accidentellement, tant que le processus le garde ouvert.


### 5.12.3. Précautions

Nous devons être prudents.

Lire un descripteur peut avoir des effets selon le type de ressource :

- lire un pipe peut consommer les données ;
    
- lire une socket peut perturber le protocole ;
    
- lire un terminal peut interférer avec l’entrée ;
    
- écrire dans un descripteur peut modifier le comportement du processus.
    

Nous ne manipulons donc pas aveuglément `/proc/<PID>/fd/<FD>` en production.


## 5.13. Redirections et héritage des descripteurs

### 5.13.1. Héritage à la création d’un processus

Lorsqu’un processus crée un enfant avec `fork()` puis `exec()`, les descripteurs ouverts peuvent être hérités.

C’est pourquoi un service peut garder ouvert un fichier ou une socket qui vient de son parent.

Exemple :

```bash
bash -c 'sleep 1000' > /tmp/out.txt &
pid=$!
ls -l /proc/$pid/fd
```

Nous voyons que `sleep` hérite de la sortie standard redirigée vers `/tmp/out.txt`.


### 5.13.2. Close-on-exec

Pour éviter d’hériter de descripteurs non nécessaires, les programmes peuvent utiliser le flag `close-on-exec`.

Ce flag indique que le descripteur doit être fermé lors d’un `exec()`.

En diagnostic, une accumulation de descripteurs hérités peut expliquer des comportements inattendus :

- fichier impossible à démonter ;
    
- socket gardée ouverte ;
    
- log supprimé mais non libéré ;
    
- service enfant gardant des ressources du parent.
    


## 5.14. Relation entre fichiers ouverts et démontage

### 5.14.1. Point de montage occupé

Un montage peut refuser de se démonter :

```bash
sudo umount /mnt/data
```

Erreur possible :

```text
target is busy
```

Cela signifie souvent qu’un processus utilise encore un fichier ou un répertoire sur ce montage.


### 5.14.2. Identifier les processus concernés

Nous pouvons utiliser :

```bash
sudo lsof +f -- /mnt/data
```

ou explorer `/proc` :

```bash
sudo sh -c '
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    case "$target" in
        /mnt/data/*) echo "$fd -> $target" ;;
    esac
done
'
```

Nous pouvons aussi vérifier les répertoires courants :

```bash
sudo ls -l /proc/[0-9]*/cwd 2>/dev/null | grep '/mnt/data'
```

Un processus peut empêcher le démontage simplement parce que son répertoire courant se trouve dans le point de montage.


## 5.15. Cas pratique complet : disque plein à cause d’un log supprimé

### 5.15.1. Situation

Nous avons un serveur avec une partition pleine.

La commande :

```bash
df -h
```

indique que `/var` est à 100 %.

Nous cherchons les gros fichiers :

```bash
sudo du -h /var | sort -h | tail
```

Mais nous ne trouvons pas de fichier assez gros pour expliquer l’usage disque.

Nous suspectons alors un fichier supprimé mais encore ouvert.


### 5.15.2. Recherche dans `/proc`

Nous cherchons les descripteurs supprimés :

```bash
sudo sh -c '
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    case "$target" in
        *"(deleted)"*) echo "$fd -> $target" ;;
    esac
done
'
```

Sortie possible :

```text
/proc/2451/fd/7 -> /var/log/app/app.log (deleted)
```

Nous identifions :

```text
PID = 2451
FD  = 7
fichier = /var/log/app/app.log
```


### 5.15.3. Identifier le processus

Nous lisons :

```bash
cat /proc/2451/comm
tr '\0' ' ' < /proc/2451/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Uid):' /proc/2451/status
```

Nous trouvons par exemple :

```text
app-server
/usr/bin/app-server --config /etc/app/config.yml
```

Nous savons maintenant quel service garde le log ouvert.


### 5.15.4. Correction

La correction propre consiste à faire rouvrir les logs par le service.

Selon le service :

```bash
sudo systemctl reload app-server
```

ou :

```bash
sudo systemctl restart app-server
```

Puis nous vérifions :

```bash
df -h /var
```

et :

```bash
sudo ls -l /proc/2451/fd 2>/dev/null | grep deleted
```

Si le processus a été redémarré, son ancien PID n’existe plus et l’espace doit être libéré.


## 5.16. Scripts utiles

### 5.16.1. Lister les fichiers ouverts par un PID

```bash
pid="$1"

if [ -z "$pid" ]; then
    echo "Usage: $0 <PID>" >&2
    exit 1
fi

if [ ! -d "/proc/$pid/fd" ]; then
    echo "Processus introuvable ou inaccessible : $pid" >&2
    exit 1
fi

for fd in /proc/$pid/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    printf "%5s -> %s\n" "${fd##*/}" "$target"
done
```

Nous pouvons sauvegarder ce script sous :

```bash
list-fd.sh
```

Puis :

```bash
chmod +x list-fd.sh
./list-fd.sh 1234
```


### 5.16.2. Compter les descripteurs ouverts par processus

```bash
for dir in /proc/[0-9]*; do
    pid=${dir#/proc/}
    [ -d "$dir/fd" ] || continue

    count=$(ls "$dir/fd" 2>/dev/null | wc -l) || continue
    name=$(cat "$dir/comm" 2>/dev/null || echo "?")

    printf "%8s %8s %s\n" "$count" "$pid" "$name"
done | sort -n | tail
```

Cette commande permet de trouver les processus ayant le plus de descripteurs ouverts.


### 5.16.3. Trouver les fichiers supprimés encore ouverts

```bash
sudo sh -c '
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    case "$target" in
        *"(deleted)"*)
            pid=$(echo "$fd" | cut -d/ -f3)
            comm=$(cat "/proc/$pid/comm" 2>/dev/null || echo "?")
            echo "PID=$pid COMM=$comm FD=${fd##*/} TARGET=$target"
            ;;
    esac
done
'
```

Sortie possible :

```text
PID=2451 COMM=app-server FD=7 TARGET=/var/log/app/app.log (deleted)
```


## 5.17. Pièges et limites

### 5.17.1. Un lien `fd` n’est pas toujours un fichier classique

Dans `/proc/<PID>/fd`, nous pouvons voir :

```text
socket:[...]
pipe:[...]
anon_inode:[...]
```

Nous ne devons pas supposer que chaque descripteur pointe vers un chemin de fichier classique.


### 5.17.2. Lire un descripteur peut avoir un effet

Accéder à :

```bash
/proc/<PID>/fd/<FD>
```

n’est pas toujours neutre.

Pour un fichier classique, lire est souvent sans danger.

Pour une socket, un pipe ou un terminal, lire peut consommer des données ou perturber le processus.

Nous devons donc être prudents.


### 5.17.3. Les permissions peuvent masquer des informations

Selon les droits, nous pouvons ne pas voir certains descripteurs.

C’est normal.

Nous devons éviter de conclure trop vite qu’un processus n’a rien ouvert si nous n’avons pas les permissions nécessaires.


### 5.17.4. Les processus changent pendant l’observation

Un processus peut ouvrir ou fermer des fichiers pendant que nous lisons `/proc/<PID>/fd`.

Un descripteur peut disparaître entre :

```bash
ls /proc/<PID>/fd
```

et :

```bash
readlink /proc/<PID>/fd/<FD>
```

Les scripts robustes doivent tolérer ces erreurs.


## 5.18. Exercices

## Exercice 1 — Observer les descripteurs du shell courant

Nous exécutons :

```bash
ls -l /proc/$$/fd
```

Nous répondons aux questions :

1. Vers quoi pointent les descripteurs `0`, `1` et `2` ?
    
2. Sont-ils attachés à un terminal ?
    
3. Quel est le chemin du terminal ?
    
4. Que se passe-t-il si nous redirigeons la sortie d’une commande vers un fichier ?
    


## Exercice 2 — Observer les redirections

Nous lançons :

```bash
bash -c 'echo $$; sleep 300' > /tmp/out.txt 2> /tmp/err.txt &
pid=$!
```

Puis :

```bash
ls -l /proc/$pid/fd
```

Nous identifions :

1. l’entrée standard ;
    
2. la sortie standard ;
    
3. la sortie d’erreur ;
    
4. les fichiers associés.
    

Nous terminons ensuite :

```bash
kill $pid
```


## Exercice 3 — Fichier supprimé mais ouvert

Nous créons un fichier :

```bash
dd if=/dev/zero of=/tmp/test-deleted.log bs=1M count=50
```

Nous le gardons ouvert :

```bash
tail -f /tmp/test-deleted.log &
pid=$!
```

Nous le supprimons :

```bash
rm /tmp/test-deleted.log
```

Nous observons :

```bash
ls -l /proc/$pid/fd | grep deleted
```

Nous répondons :

1. Pourquoi le fichier n’apparaît-il plus dans `/tmp` ?
    
2. Pourquoi l’espace peut-il rester occupé ?
    
3. Que se passe-t-il quand nous arrêtons `tail` ?
    

Nous terminons :

```bash
kill $pid
```


## Exercice 4 — Compter les descripteurs ouverts

Nous choisissons un processus actif, par exemple le shell :

```bash
pid=$$
```

Puis :

```bash
ls /proc/$pid/fd | wc -l
```

Nous comparons avec un processus plus complexe, par exemple un navigateur ou un serveur.

Nous discutons :

1. Pourquoi certains processus ont-ils beaucoup de descripteurs ?
    
2. Quels types de ressources voyons-nous ?
    
3. Comment détecter une fuite de descripteurs ?
    


## Exercice 5 — Comparer `/proc` et `lsof`

Nous exécutons :

```bash
ls -l /proc/$$/fd
```

Puis :

```bash
lsof -p $$
```

Si `lsof` n’est pas installé :

```bash
sudo apt install lsof
```

Nous comparons :

1. quelles informations sont identiques ;
    
2. quelles informations sont plus lisibles avec `lsof` ;
    
3. ce que `/proc` permet de voir directement ;
    
4. pourquoi `lsof` est pratique en production.
    


## 5.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. Un processus manipule des ressources ouvertes via des descripteurs de fichiers.
    
2. Les descripteurs `0`, `1` et `2` correspondent à l’entrée standard, la sortie standard et la sortie d’erreur.
    
3. `/proc/<PID>/fd` expose les descripteurs ouverts d’un processus.
    
4. Les liens dans `fd` peuvent pointer vers des fichiers, sockets, pipes, terminaux, périphériques ou objets noyau.
    
5. `/proc/<PID>/fdinfo` donne des informations complémentaires comme la position et les flags.
    
6. Un fichier supprimé peut continuer à occuper de l’espace disque s’il reste ouvert par un processus.
    
7. `/proc/<PID>/fd` est très utile pour diagnostiquer les logs supprimés, les fuites de descripteurs, les sockets ouvertes et les points de montage occupés.
    
8. `lsof` fournit une vue plus lisible, mais `/proc` permet de comprendre la source brute.
    
9. Lire ou écrire via `/proc/<PID>/fd/<FD>` peut avoir des effets et doit être fait avec prudence.
    
10. Les scripts doivent gérer les permissions, les processus qui disparaissent et les ressources non classiques.
    


## Conclusion du chapitre 5

Nous savons maintenant analyser les fichiers ouverts par un processus à travers `/proc/<PID>/fd`.

Cette compétence est essentielle pour le diagnostic système. Elle permet de comprendre pourquoi un disque reste plein, pourquoi un service garde une socket ouverte, pourquoi un démontage échoue, pourquoi un processus atteint la limite “Too many open files”, ou encore pourquoi un fichier supprimé reste récupérable.

Nous avons aussi vu que les descripteurs de fichiers dépassent largement les fichiers classiques : ils représentent une interface générale vers de nombreuses ressources du noyau.

Dans le chapitre suivant, nous étudions la mémoire d’un processus avec `/proc/<PID>/maps`, `/proc/<PID>/smaps`, `/proc/<PID>/smaps_rollup` et `/proc/<PID>/limits`.

# Chapitre 6 — Mémoire d’un processus

## Objectifs du chapitre

Dans ce chapitre, nous étudions la mémoire d’un processus à travers `/proc`.

Nous avons vu précédemment qu’un processus possède une ligne de commande, un état, des fichiers ouverts, un environnement et des descripteurs. Nous allons maintenant nous intéresser à son espace mémoire.

Sous Linux, chaque processus possède une mémoire virtuelle. Cette mémoire virtuelle ne correspond pas directement à la quantité de RAM réellement consommée. Elle est composée de zones mémoire, appelées mappings, qui peuvent représenter :

- le code exécutable du programme ;
    
- les bibliothèques partagées ;
    
- le tas ;
    
- la pile ;
    
- des fichiers mappés en mémoire ;
    
- de la mémoire anonyme ;
    
- des zones partagées ;
    
- des zones réservées mais pas encore réellement utilisées.
    

À la fin de ce chapitre, nous savons :

- expliquer la différence entre mémoire virtuelle et mémoire résidente ;
    
- lire `/proc/<PID>/maps` ;
    
- identifier la pile, le tas et les bibliothèques partagées ;
    
- comprendre les permissions mémoire ;
    
- lire `/proc/<PID>/smaps` et `/proc/<PID>/smaps_rollup` ;
    
- distinguer RSS, PSS, mémoire privée et mémoire partagée ;
    
- utiliser `/proc/<PID>/limits` pour comprendre les limites de ressources ;
    
- construire un premier diagnostic mémoire sur un processus.
    


## 6.1. Mémoire virtuelle et mémoire physique

### 6.1.1. Pourquoi la mémoire d’un processus est difficile à comprendre ?

Lorsque nous disons qu’un processus “consomme de la mémoire”, nous simplifions fortement la réalité.

Sous Linux, un processus ne voit pas directement la RAM physique. Il manipule un espace d’adressage virtuel.

Cet espace virtuel est ensuite traduit par le noyau et le matériel, notamment par la MMU, en pages mémoire physiques.

Nous devons donc distinguer plusieurs notions :

|Notion|Signification|
|---|---|
|Mémoire virtuelle|espace d’adressage vu par le processus|
|Mémoire résidente|partie réellement présente en RAM|
|Mémoire partagée|mémoire utilisée par plusieurs processus|
|Mémoire privée|mémoire propre à un processus|
|Mémoire anonyme|mémoire non directement associée à un fichier|
|Mémoire mappée|mémoire associée à un fichier ou à une ressource|

Cette distinction est essentielle pour éviter de mauvaises conclusions.


### 6.1.2. Exemple classique : `VmSize` n’est pas la RAM consommée

Dans `/proc/<PID>/status`, nous pouvons lire :

```bash
grep -E '^(VmSize|VmRSS):' /proc/<PID>/status
```

Exemple :

```text
VmSize:   1250000 kB
VmRSS:      85000 kB
```

Nous pourrions croire que le processus consomme 1,25 Go de RAM. Ce serait faux.

`VmSize` représente l’espace mémoire virtuel total du processus.

`VmRSS` représente la mémoire résidente, c’est-à-dire la partie actuellement présente en RAM.

Nous retenons :

```text
VmSize élevé ≠ consommation RAM élevée.
VmRSS est souvent plus proche de la mémoire réellement présente en RAM.
```

Mais même `VmRSS` doit être interprété avec prudence, car il inclut de la mémoire partagée.


## 6.2. Préparer un processus d’exemple

### 6.2.1. Lancer un processus simple

Pour observer la mémoire, nous lançons un processus simple :

```bash
sleep 1000 &
pid=$!
echo "$pid"
```

Puis nous observons ses informations mémoire :

```bash
grep -E '^(Name|State|VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
```

Nous obtenons une première vue.

Cependant, `sleep` est très simple. Pour observer plus de mappings, nous pouvons utiliser un programme plus riche, par exemple notre shell :

```bash
pid=$$
```

ou un processus Python :

```bash
python3 -c 'import time; data = bytearray(100 * 1024 * 1024); time.sleep(1000)' &
pid=$!
```

Ce processus alloue environ 100 Mio de mémoire et reste actif.


### 6.2.2. Nettoyage à la fin

Lorsque nous lançons un processus de test en arrière-plan, nous devons penser à le terminer :

```bash
kill $pid
```

Si nécessaire :

```bash
kill -9 $pid
```

Nous évitons de laisser des processus de test tourner inutilement.


## 6.3. `/proc/<PID>/maps` : cartographie mémoire

### 6.3.1. Lire `maps`

Le fichier principal pour comprendre l’organisation mémoire d’un processus est :

```bash
cat /proc/<PID>/maps
```

Exemple :

```bash
cat /proc/$pid/maps | head
```

Sortie possible :

```text
55b1f3b2d000-55b1f3b2f000 r--p 00000000 08:02 393324 /usr/bin/sleep
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
55b1f3b33000-55b1f3b35000 r--p 00006000 08:02 393324 /usr/bin/sleep
55b1f3b35000-55b1f3b36000 r--p 00007000 08:02 393324 /usr/bin/sleep
55b1f3b36000-55b1f3b37000 rw-p 00008000 08:02 393324 /usr/bin/sleep
55b1f4f4f000-55b1f4f70000 rw-p 00000000 00:00 0      [heap]
7f83d5c00000-7f83d5c21000 rw-p 00000000 00:00 0
7f83d5c21000-7f83d6000000 ---p 00000000 00:00 0
```

Chaque ligne décrit une zone mémoire du processus.


### 6.3.2. Structure d’une ligne de `maps`

Une ligne de `/proc/<PID>/maps` suit généralement cette structure :

```text
adresse-début-adresse-fin permissions offset périphérique inode chemin
```

Exemple :

```text
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
```

Nous pouvons découper :

|Champ|Exemple|Signification|
|---|---|---|
|Plage d’adresses|`55b1f3b2f000-55b1f3b33000`|zone mémoire virtuelle|
|Permissions|`r-xp`|droits de lecture, écriture, exécution, partage|
|Offset|`00002000`|décalage dans le fichier mappé|
|Périphérique|`08:02`|périphérique contenant le fichier|
|Inode|`393324`|inode du fichier|
|Chemin|`/usr/bin/sleep`|fichier ou zone spéciale|


### 6.3.3. Plages d’adresses

La première colonne indique la plage d’adresses virtuelles.

Exemple :

```text
55b1f3b2f000-55b1f3b33000
```

Cela signifie que la zone commence à l’adresse virtuelle :

```text
55b1f3b2f000
```

et se termine à :

```text
55b1f3b33000
```

Nous pouvons calculer la taille de cette zone :

```text
0x55b1f3b33000 - 0x55b1f3b2f000 = 0x4000
```

Soit 16 Kio.

Nous pouvons automatiser ce calcul, mais l’idée principale est que `maps` expose des plages d’adresses virtuelles, pas directement des adresses physiques.


### 6.3.4. Permissions mémoire

La deuxième colonne indique les permissions.

Exemples :

```text
r--p
r-xp
rw-p
---p
r--s
rw-s
```

Nous interprétons les caractères :

|Caractère|Signification|
|---|---|
|`r`|lecture autorisée|
|`w`|écriture autorisée|
|`x`|exécution autorisée|
|`-`|permission absente|
|`p`|mapping privé|
|`s`|mapping partagé|

Exemple :

```text
r-xp
```

signifie :

```text
lisible, exécutable, non modifiable, privé
```

C’est typiquement une zone de code.

Exemple :

```text
rw-p
```

signifie :

```text
lisible, modifiable, non exécutable, privé
```

C’est typiquement une zone de données ou de tas.


## 6.4. Les zones mémoire importantes

### 6.4.1. L’exécutable principal

Dans `maps`, nous voyons souvent plusieurs lignes associées à l’exécutable.

Exemple :

```text
55b1f3b2d000-55b1f3b2f000 r--p ... /usr/bin/sleep
55b1f3b2f000-55b1f3b33000 r-xp ... /usr/bin/sleep
55b1f3b33000-55b1f3b35000 r--p ... /usr/bin/sleep
55b1f3b36000-55b1f3b37000 rw-p ... /usr/bin/sleep
```

Nous observons que le même fichier peut être mappé plusieurs fois avec des permissions différentes.

Cela correspond aux différentes sections du binaire :

- code exécutable ;
    
- données en lecture seule ;
    
- données modifiables ;
    
- métadonnées.
    

Nous pouvons faire le lien avec les formats exécutables comme ELF.


### 6.4.2. Le tas : `[heap]`

Le tas est une zone mémoire utilisée pour les allocations dynamiques.

Dans `maps`, il apparaît souvent ainsi :

```text
55b1f4f4f000-55b1f4f70000 rw-p 00000000 00:00 0 [heap]
```

Le tas est utilisé par des fonctions comme :

```c
malloc()
calloc()
realloc()
free()
```

En Python, JavaScript, Java, Ruby ou Go, le runtime gère lui-même les allocations, mais repose finalement sur des mécanismes mémoire du système.

Nous pouvons chercher le tas :

```bash
grep '\[heap\]' /proc/$pid/maps
```

Un processus peut avoir un tas plus ou moins grand selon son comportement.


### 6.4.3. La pile : `[stack]`

La pile du thread principal apparaît souvent ainsi :

```text
7ffcc7b4d000-7ffcc7b6e000 rw-p 00000000 00:00 0 [stack]
```

Nous pouvons la chercher :

```bash
grep '\[stack\]' /proc/$pid/maps
```

La pile contient notamment :

- les variables locales ;
    
- les adresses de retour ;
    
- les frames d’appel ;
    
- certaines données temporaires.
    

Chaque thread peut avoir sa propre pile.

Dans les processus multi-thread, nous pouvons observer les stacks par thread dans :

```bash
/proc/<PID>/task/<TID>/maps
```


### 6.4.4. Bibliothèques partagées

Les bibliothèques partagées apparaissent avec leur chemin.

Exemples :

```text
/usr/lib/x86_64-linux-gnu/libc.so.6
/usr/lib/x86_64-linux-gnu/libpthread.so.0
/usr/lib/x86_64-linux-gnu/libm.so.6
```

Nous pouvons les lister :

```bash
awk '{print $6}' /proc/$pid/maps 2>/dev/null | grep '^/' | sort -u
```

Cela nous permet de voir les bibliothèques chargées par le processus.

C’est utile pour diagnostiquer :

- une mauvaise version de bibliothèque ;
    
- un conflit de dépendances ;
    
- un binaire qui charge une bibliothèque inattendue ;
    
- une application qui utilise beaucoup de modules natifs.
    


### 6.4.5. Mémoire anonyme

Certaines lignes n’ont pas de chemin.

Exemple :

```text
7f83d5c00000-7f83d5c21000 rw-p 00000000 00:00 0
```

Nous parlons souvent de mémoire anonyme.

Elle n’est pas directement associée à un fichier.

Elle peut correspondre à :

- des allocations dynamiques ;
    
- des zones réservées par le runtime ;
    
- des piles de threads ;
    
- des buffers ;
    
- des zones internes de bibliothèques.
    

Une forte quantité de mémoire anonyme peut être normale pour certaines applications, mais elle peut aussi signaler une fuite mémoire.


### 6.4.6. Mappings de fichiers

Un processus peut mapper un fichier en mémoire.

Cela peut être fait avec l’appel système :

```c
mmap()
```

Un fichier mappé peut être lu comme s’il était en mémoire.

Cela sert notamment pour :

- charger un exécutable ;
    
- charger des bibliothèques partagées ;
    
- accéder efficacement à de gros fichiers ;
    
- partager de la mémoire entre processus ;
    
- implémenter certaines bases de données ;
    
- manipuler des fichiers volumineux.
    

Dans `maps`, ces mappings apparaissent avec le chemin du fichier.


## 6.5. Comprendre `maps` par l’exemple

### 6.5.1. Afficher une vue simplifiée

Nous pouvons afficher les zones mémoire avec leurs permissions et chemins :

```bash
awk '{print $1, $2, $6}' /proc/$pid/maps | head -30
```

Attention : toutes les lignes n’ont pas forcément de sixième champ.

Nous pouvons écrire une version plus lisible :

```bash
awk '
{
  path = $6
  if (path == "") path = "[anonymous]"
  print $1, $2, path
}
' /proc/$pid/maps | head -30
```


### 6.5.2. Lister les zones exécutables

Les zones exécutables ont la permission `x`.

```bash
awk '$2 ~ /x/ { print }' /proc/$pid/maps
```

Nous y trouvons généralement :

- le binaire principal ;
    
- les bibliothèques partagées ;
    
- le linker dynamique ;
    
- parfois des zones JIT pour certains runtimes.
    

Pour des environnements comme JavaScript, Java, .NET ou certains moteurs de base de données, la présence de mémoire exécutable générée dynamiquement peut être normale.


### 6.5.3. Lister les zones modifiables

Les zones en écriture ont la permission `w`.

```bash
awk '$2 ~ /w/ { print }' /proc/$pid/maps | head
```

Nous y trouvons souvent :

- le tas ;
    
- la pile ;
    
- des zones anonymes ;
    
- des zones de données modifiables ;
    
- certaines zones partagées.
    


### 6.5.4. Chercher les fichiers supprimés mappés

Comme pour les descripteurs, un processus peut encore mapper un fichier supprimé.

Nous pouvons chercher :

```bash
grep deleted /proc/$pid/maps
```

Exemple :

```text
7f1234000000-7f1234200000 r-xp ... /usr/lib/libold.so (deleted)
```

Cela peut arriver après une mise à jour de bibliothèque.

Le processus continue d’utiliser l’ancienne version tant qu’il n’est pas redémarré.


## 6.6. `/proc/<PID>/smaps` : statistiques détaillées par mapping

### 6.6.1. Lire `smaps`

Le fichier :

```bash
/proc/<PID>/smaps
```

donne des statistiques détaillées pour chaque zone mémoire.

Nous pouvons lire :

```bash
cat /proc/$pid/smaps | head -60
```

La sortie ressemble à `maps`, mais chaque mapping est suivi de nombreuses lignes.

Extrait :

```text
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
Size:                  16 kB
KernelPageSize:         4 kB
MMUPageSize:            4 kB
Rss:                   16 kB
Pss:                    4 kB
Shared_Clean:          16 kB
Shared_Dirty:           0 kB
Private_Clean:          0 kB
Private_Dirty:          0 kB
Referenced:            16 kB
Anonymous:              0 kB
Swap:                   0 kB
```

`smaps` est beaucoup plus détaillé que `maps`.


### 6.6.2. Coût de lecture de `smaps`

Lire `smaps` peut coûter plus cher que lire `maps`.

Sur un gros processus avec beaucoup de mappings, `smaps` peut être volumineux et demander un travail réel au noyau.

Nous évitons donc de lire `smaps` en boucle très fréquente en production sans raison.

Pour une synthèse plus légère, nous préférons parfois :

```bash
/proc/<PID>/smaps_rollup
```

que nous étudions ensuite.


### 6.6.3. Champs importants de `smaps`

Les champs les plus utiles sont :

|Champ|Signification|
|---|---|
|`Size`|taille virtuelle du mapping|
|`Rss`|mémoire résidente en RAM|
|`Pss`|part proportionnelle de mémoire|
|`Shared_Clean`|mémoire partagée propre|
|`Shared_Dirty`|mémoire partagée modifiée|
|`Private_Clean`|mémoire privée propre|
|`Private_Dirty`|mémoire privée modifiée|
|`Referenced`|mémoire récemment référencée|
|`Anonymous`|mémoire anonyme|
|`Swap`|mémoire de ce mapping partie en swap|

Nous n’avons pas besoin de tous les champs pour un premier diagnostic, mais nous devons comprendre les principaux.


## 6.7. RSS, PSS, mémoire partagée et mémoire privée

### 6.7.1. RSS : Resident Set Size

Le RSS représente la mémoire du processus actuellement présente en RAM.

Dans `status`, nous avons :

```bash
grep '^VmRSS:' /proc/$pid/status
```

Dans `smaps`, chaque mapping a une ligne :

```text
Rss: 16 kB
```

Le RSS est utile, mais il peut être trompeur.

Pourquoi ? Parce qu’il compte la mémoire partagée dans chaque processus qui l’utilise.


### 6.7.2. Exemple de limite du RSS

Supposons qu’une bibliothèque partagée de 10 Mio soit utilisée par 20 processus.

Le RSS de chaque processus peut inclure ces 10 Mio.

Si nous additionnons naïvement les RSS des 20 processus, nous comptons potentiellement plusieurs fois la même mémoire.

Nous obtenons alors une surestimation.

C’est pourquoi le RSS est utile pour observer un processus, mais dangereux pour additionner la mémoire d’un ensemble de processus.


### 6.7.3. PSS : Proportional Set Size

Le PSS répartit la mémoire partagée entre les processus qui l’utilisent.

Si une page mémoire est partagée par 4 processus, chaque processus se voit attribuer un quart de cette page dans son PSS.

Cela donne une estimation plus juste de la contribution mémoire réelle d’un processus.

Dans `smaps`, nous voyons :

```text
Pss: 4 kB
```

Pour additionner la mémoire de plusieurs processus, le PSS est souvent plus pertinent que le RSS.


### 6.7.4. Mémoire privée

La mémoire privée est propre à un processus.

Dans `smaps`, elle apparaît notamment sous :

```text
Private_Clean
Private_Dirty
```

- `Private_Clean` : mémoire privée propre, potentiellement relisible depuis un fichier ;
    
- `Private_Dirty` : mémoire privée modifiée, réellement propre au processus.
    

La mémoire `Private_Dirty` est souvent intéressante pour détecter la mémoire réellement consommée par une application.


### 6.7.5. Mémoire partagée

La mémoire partagée apparaît sous :

```text
Shared_Clean
Shared_Dirty
```

Elle peut correspondre à :

- bibliothèques partagées ;
    
- pages partagées entre processus ;
    
- mémoire partagée explicite ;
    
- fichiers mappés partagés.
    

Une forte mémoire partagée n’est pas forcément problématique.

Au contraire, elle peut être un mécanisme d’efficacité.


## 6.8. `/proc/<PID>/smaps_rollup` : synthèse mémoire

### 6.8.1. Lire `smaps_rollup`

Sur les noyaux modernes, nous pouvons lire :

```bash
cat /proc/$pid/smaps_rollup
```

Exemple :

```text
55b1f3b2d000-7ffcc7b6e000 ---p 00000000 00:00 0 [rollup]
Rss:                1216 kB
Pss:                 180 kB
Pss_Dirty:           104 kB
Shared_Clean:       1112 kB
Shared_Dirty:          0 kB
Private_Clean:         0 kB
Private_Dirty:       104 kB
Referenced:         1216 kB
Anonymous:           104 kB
Swap:                  0 kB
```

Ce fichier agrège les informations de `smaps` pour l’ensemble du processus.

Il est très pratique pour obtenir une vue synthétique.


### 6.8.2. Extraire les champs clés

Nous pouvons lire seulement quelques champs :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Swap):' /proc/$pid/smaps_rollup
```

Exemple :

```text
Rss:                1216 kB
Pss:                 180 kB
Shared_Clean:       1112 kB
Private_Dirty:       104 kB
Swap:                  0 kB
```

Ces informations donnent une vision plus fine que `VmRSS`.


### 6.8.3. Quand préférons-nous `smaps_rollup` ?

Nous préférons `smaps_rollup` lorsque :

- nous voulons une synthèse mémoire du processus ;
    
- nous ne voulons pas analyser mapping par mapping ;
    
- nous voulons une estimation plus fine que `status` ;
    
- nous voulons observer `Pss` ;
    
- nous cherchons une contribution mémoire plus réaliste.
    

Nous préférons `smaps` lorsque :

- nous voulons comprendre quelle zone mémoire consomme ;
    
- nous cherchons une bibliothèque ou un mapping particulier ;
    
- nous analysons une fuite mémoire fine ;
    
- nous voulons distinguer précisément les zones anonymes, fichiers mappés, heap, stack.
    


## 6.9. Calculer une synthèse mémoire avec `awk`

### 6.9.1. Additionner les RSS dans `smaps`

Nous pouvons additionner les lignes `Rss` :

```bash
awk '/^Rss:/ { total += $2 } END { printf "RSS total: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Cela doit être proche de `VmRSS`.


### 6.9.2. Additionner les PSS

```bash
awk '/^Pss:/ { total += $2 } END { printf "PSS total: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Le PSS est souvent plus intéressant pour comprendre la part réelle du processus.


### 6.9.3. Additionner la mémoire privée dirty

```bash
awk '/^Private_Dirty:/ { total += $2 } END { printf "Private_Dirty: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Cette valeur est utile pour savoir quelle mémoire modifiée appartient vraiment au processus.


### 6.9.4. Regrouper par fichier mappé

Nous pouvons produire une vue approximative par chemin :

```bash
awk '
/^[0-9a-f]+-[0-9a-f]+/ {
    path=$6
    if (path=="") path="[anonymous]"
}
/^Rss:/ {
    rss[path]+=$2
}
END {
    for (p in rss) {
        printf "%10.2f MiB  %s\n", rss[p]/1024, p
    }
}
' /proc/$pid/smaps | sort -n | tail
```

Cela permet d’identifier les chemins ou catégories qui contribuent le plus au RSS.

Nous restons prudents : ce script est pédagogique et peut être amélioré.


## 6.10. `/proc/<PID>/limits` : limites de ressources

### 6.10.1. Lire les limites

Le fichier :

```bash
cat /proc/<PID>/limits
```

affiche les limites de ressources du processus.

Exemple :

```text
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             30995                30995                processes
Max open files            1024                 1048576              files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
```

Ces limites correspondent aux limites de ressources applicables au processus.


### 6.10.2. Limite souple et limite dure

Nous distinguons :

|Type|Signification|
|---|---|
|Soft limit|limite actuellement appliquée|
|Hard limit|limite maximale que le processus peut généralement se fixer sans privilèges supplémentaires|

Un processus peut souvent augmenter sa limite souple jusqu’à la limite dure.

Pour aller au-delà de la limite dure, il faut généralement des droits supplémentaires.


### 6.10.3. Limites importantes pour la mémoire

Plusieurs limites peuvent impacter la mémoire :

|Limite|Rôle|
|---|---|
|`Max data size`|taille maximale du segment de données|
|`Max stack size`|taille maximale de pile|
|`Max resident set`|limite RSS, rarement utilisée comme contrainte stricte moderne|
|`Max locked memory`|mémoire verrouillable en RAM|
|`Max address space`|taille maximale de l’espace d’adressage virtuel|

La limite la plus visible dans de nombreux cas est `Max stack size`.

Exemple :

```bash
grep 'Max stack size' /proc/$pid/limits
```

Sortie possible :

```text
Max stack size            8388608              unlimited            bytes
```

Cela correspond à une pile de 8 Mio par défaut dans beaucoup de configurations.


### 6.10.4. Lien avec `ulimit`

Dans le shell, nous pouvons afficher les limites avec :

```bash
ulimit -a
```

Les processus enfants héritent généralement des limites du shell ou du service qui les lance.

Exemple :

```bash
ulimit -n
```

affiche la limite du nombre de fichiers ouverts.

Nous faisons le lien avec le chapitre précédent :

```text
Une erreur “Too many open files” est souvent liée à la limite Max open files.
```


### 6.10.5. Limites systemd

Pour un service lancé par systemd, les limites peuvent être définies dans l’unité.

Exemples :

```ini
LimitNOFILE=65536
LimitNPROC=4096
LimitSTACK=8M
```

Après modification, nous rechargeons systemd :

```bash
sudo systemctl daemon-reload
sudo systemctl restart mon-service
```

Puis nous vérifions dans :

```bash
cat /proc/<PID>/limits
```

Nous ne nous contentons pas de lire le fichier de configuration : nous vérifions la limite réellement appliquée au processus.


## 6.11. Diagnostiquer une fuite mémoire

### 6.11.1. Symptôme

Une fuite mémoire se manifeste souvent par une croissance continue de la mémoire consommée par un processus.

Nous pouvons observer :

```bash
watch -n 2 "grep -E '^(VmSize|VmRSS|RssAnon):' /proc/$pid/status"
```

Si `VmRSS` ou `RssAnon` augmente continuellement sans redescendre, nous suspectons une fuite ou une accumulation de données.


### 6.11.2. Observer `smaps_rollup`

Nous pouvons aussi observer :

```bash
watch -n 2 "grep -E '^(Rss|Pss|Private_Dirty|Anonymous|Swap):' /proc/$pid/smaps_rollup"
```

Les champs intéressants sont :

- `Rss` ;
    
- `Pss` ;
    
- `Private_Dirty` ;
    
- `Anonymous` ;
    
- `Swap`.
    

Si `Private_Dirty` et `Anonymous` augmentent continuellement, cela peut indiquer que le processus accumule de la mémoire propre.


### 6.11.3. Différencier fuite et cache applicatif

Une croissance mémoire n’est pas toujours une fuite.

Elle peut correspondre à :

- un cache applicatif volontaire ;
    
- un warm-up de runtime ;
    
- un JIT ;
    
- un pool de connexions ;
    
- un chargement de données ;
    
- une file d’attente interne ;
    
- une montée en charge normale.
    

Nous devons croiser avec :

- les logs applicatifs ;
    
- les métriques métier ;
    
- le trafic ;
    
- le nombre de requêtes ;
    
- les tailles de files ;
    
- les métriques du runtime.
    

`/proc` donne des symptômes, pas toujours la cause applicative exacte.


## 6.12. Cas particulier des runtimes modernes

### 6.12.1. Java

Un processus Java peut réserver un espace mémoire virtuel important.

Nous pouvons voir un `VmSize` très élevé sans que la RAM soit réellement utilisée.

Il faut croiser avec :

```bash
jcmd
jstat
jmap
```

et les options de JVM :

```text
-Xms
-Xmx
-XX:MaxRAMPercentage
```

`/proc` nous donne la vue noyau, mais la JVM a sa propre gestion mémoire.


### 6.12.2. Node.js

Node.js utilise le moteur V8.

La mémoire observée dans `/proc` inclut :

- le heap JavaScript ;
    
- les buffers natifs ;
    
- les bibliothèques ;
    
- le code JIT ;
    
- les allocations C++ natives ;
    
- les modules natifs.
    

Nous ne pouvons pas conclure uniquement avec `VmRSS`.

Nous devons parfois compléter avec :

```bash
node --inspect
process.memoryUsage()
```

ou des dumps mémoire.


### 6.12.3. Python

Python peut conserver de la mémoire via son allocateur interne.

Après libération d’objets Python, la mémoire peut ne pas être immédiatement rendue au système.

Nous pouvons donc observer un RSS stable ou élevé même si l’application a libéré des objets.

Nous complétons avec :

- `tracemalloc` ;
    
- `objgraph` ;
    
- profils mémoire ;
    
- métriques applicatives.
    


### 6.12.4. Go

Go utilise son propre garbage collector.

Le runtime peut réserver de la mémoire, la réutiliser, et ne pas la rendre immédiatement au système.

Nous complétons avec :

- `pprof` ;
    
- métriques runtime ;
    
- `GOMEMLIMIT` ;
    
- `GOGC`.
    

Encore une fois, `/proc` est indispensable, mais il ne remplace pas les outils spécifiques du runtime.


## 6.13. Mémoire et conteneurs

### 6.13.1. Limite du point de vue `/proc`

Dans un conteneur, `/proc/<PID>/status`, `maps` et `smaps` décrivent le processus.

Mais la limite mémoire réelle peut être imposée par les cgroups.

Un processus peut voir beaucoup de mémoire système dans `/proc/meminfo`, mais être limité à 512 Mio par son conteneur.

Nous devons donc croiser avec :

```bash
cat /proc/1/cgroup
```

et les fichiers de cgroups dans :

```bash
/sys/fs/cgroup
```


### 6.13.2. OOM dans les conteneurs

Lorsqu’un conteneur dépasse sa limite mémoire, il peut être tué par le mécanisme OOM.

Depuis l’application, nous pouvons voir :

- une croissance de RSS ;
    
- une pression mémoire ;
    
- puis une terminaison brutale.
    

Dans Kubernetes, cela apparaît souvent comme :

```text
OOMKilled
```

Pour diagnostiquer, nous croisons :

- `/proc/<PID>/status` ;
    
- `/proc/<PID>/smaps_rollup` ;
    
- les métriques cgroups ;
    
- les événements Kubernetes ;
    
- les logs du pod.
    


## 6.14. Sécurité et mémoire

### 6.14.1. Informations sensibles dans la mémoire

La mémoire d’un processus peut contenir :

- mots de passe ;
    
- tokens ;
    
- clés privées ;
    
- secrets applicatifs ;
    
- données utilisateurs ;
    
- contenu de requêtes ;
    
- données temporaires.
    

Certains fichiers de `/proc` peuvent exposer des informations sensibles si les permissions sont trop ouvertes.


### 6.14.2. `/proc/<PID>/mem`

Il existe un fichier :

```bash
/proc/<PID>/mem
```

qui représente la mémoire du processus.

Nous ne l’utilisons pas dans ce cours comme outil de lecture directe.

Son accès est fortement contrôlé, car il permettrait de lire ou modifier la mémoire d’un processus.

Nous retenons simplement :

```text
/proc/<PID>/mem est très sensible et ne doit pas être manipulé sans cadre précis.
```


### 6.14.3. Dumps mémoire

Un core dump peut contenir toute la mémoire d’un processus au moment d’un crash.

Cela peut aider au debugging, mais cela peut aussi exposer des secrets.

La limite liée aux core dumps apparaît dans :

```bash
grep 'Max core file size' /proc/$pid/limits
```

Si la limite est `0`, les core dumps sont désactivés pour ce processus.

Nous devons traiter les dumps mémoire comme des données sensibles.


## 6.15. Cas pratique complet : analyser la mémoire d’un processus

### 6.15.1. Situation

Nous avons un processus applicatif suspecté de consommer trop de mémoire.

Nous avons son PID :

```bash
pid=1234
```

Nous voulons établir un premier diagnostic.


### 6.15.2. Identifier le processus

Nous commençons par :

```bash
cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline
echo
grep -E '^(Name|State|Pid|PPid|Threads):' /proc/$pid/status
```

Nous vérifions que nous analysons le bon processus.


### 6.15.3. Lire la mémoire globale du processus

Nous lisons :

```bash
grep -E '^(VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
```

Nous interprétons :

- `VmSize` : espace virtuel ;
    
- `VmRSS` : mémoire résidente ;
    
- `RssAnon` : mémoire anonyme ;
    
- `RssFile` : mémoire liée à des fichiers ;
    
- `RssShmem` : mémoire partagée.
    


### 6.15.4. Lire la synthèse fine

Si disponible :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Anonymous|Swap):' /proc/$pid/smaps_rollup
```

Nous obtenons une vision plus fine.

Nous cherchons notamment :

- PSS élevé ;
    
- Private_Dirty élevé ;
    
- Anonymous élevé ;
    
- Swap non nul.
    


### 6.15.5. Identifier les mappings importants

Nous regardons les plus gros contributeurs :

```bash
awk '
/^[0-9a-f]+-[0-9a-f]+/ {
    path=$6
    if (path=="") path="[anonymous]"
}
/^Rss:/ {
    rss[path]+=$2
}
END {
    for (p in rss) {
        printf "%10.2f MiB  %s\n", rss[p]/1024, p
    }
}
' /proc/$pid/smaps | sort -n | tail -20
```

Nous cherchons :

- mémoire anonyme importante ;
    
- gros fichiers mappés ;
    
- bibliothèques inattendues ;
    
- segments heap importants ;
    
- fichiers supprimés encore mappés.
    


### 6.15.6. Observer l’évolution

Nous observons dans le temps :

```bash
watch -n 5 "grep -E '^(VmRSS|RssAnon):' /proc/$pid/status"
```

ou :

```bash
watch -n 5 "grep -E '^(Rss|Pss|Private_Dirty|Anonymous|Swap):' /proc/$pid/smaps_rollup"
```

Si la mémoire augmente continuellement, nous formulons une hypothèse de fuite ou d’accumulation.


### 6.15.7. Formuler une conclusion

Exemple de conclusion prudente :

```text
Nous observons que le processus possède un VmRSS élevé, mais qu’une partie importante est partagée. Le PSS est plus modéré. La mémoire privée dirty reste stable. Nous ne concluons donc pas à une fuite mémoire immédiate.
```

Autre exemple :

```text
Nous observons une croissance continue de RssAnon et Private_Dirty. Cette évolution suggère une accumulation de mémoire propre au processus. Nous devons compléter l’analyse avec les outils du runtime applicatif.
```


## 6.16. Scripts utiles

### 6.16.1. Synthèse mémoire d’un PID

```bash
##!/usr/bin/env bash

pid="$1"

if [ -z "$pid" ]; then
    echo "Usage: $0 <PID>" >&2
    exit 1
fi

if [ ! -d "/proc/$pid" ]; then
    echo "Processus introuvable : $pid" >&2
    exit 1
fi

echo "Processus"
echo "---------"
cat "/proc/$pid/comm" 2>/dev/null || true
tr '\0' ' ' < "/proc/$pid/cmdline" 2>/dev/null
echo
echo

echo "Status mémoire"
echo "--------------"
grep -E '^(VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' "/proc/$pid/status" 2>/dev/null || true
echo

if [ -r "/proc/$pid/smaps_rollup" ]; then
    echo "smaps_rollup"
    echo "-------------"
    grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Anonymous|Swap):' "/proc/$pid/smaps_rollup"
else
    echo "smaps_rollup indisponible"
fi
```

Nous pouvons le sauvegarder sous :

```bash
mem-summary.sh
```

Puis :

```bash
chmod +x mem-summary.sh
./mem-summary.sh 1234
```


### 6.16.2. Surveiller la mémoire d’un processus

```bash
watch -n 2 "grep -E '^(VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/<PID>/status"
```

Nous remplaçons `<PID>` par le PID réel.


### 6.16.3. Afficher les bibliothèques chargées

```bash
awk '{print $6}' /proc/<PID>/maps 2>/dev/null | grep '^/' | sort -u
```

Cette commande liste les fichiers mappés depuis le disque.


### 6.16.4. Chercher les fichiers mappés supprimés

```bash
grep deleted /proc/<PID>/maps
```

Si nous trouvons des bibliothèques ou fichiers supprimés, nous pouvons envisager de redémarrer le processus pour qu’il recharge les versions actuelles.


## 6.17. Pièges et limites

### 6.17.1. Ne pas confondre virtuel et physique

Un très grand `VmSize` peut être normal.

Certains programmes réservent beaucoup d’espace virtuel sans utiliser réellement autant de RAM.

Nous regardons aussi :

- `VmRSS` ;
    
- `RssAnon` ;
    
- `Pss` ;
    
- `Private_Dirty`.
    


### 6.17.2. Ne pas additionner naïvement les RSS

Additionner les RSS de plusieurs processus peut surestimer la mémoire utilisée, car la mémoire partagée est comptée plusieurs fois.

Pour une estimation plus juste, nous préférons le PSS.


### 6.17.3. `smaps` peut être coûteux

Lire `smaps` trop fréquemment sur beaucoup de processus peut coûter cher.

Pour une supervision régulière, nous préférons des outils adaptés ou des métriques déjà agrégées.


### 6.17.4. Les runtimes ont leur propre logique

Java, Go, Python, Node.js et d’autres runtimes gèrent leur mémoire de manière spécifique.

`/proc` donne la vue du noyau, mais pas toujours la cause applicative.

Nous complétons avec les outils du langage.


### 6.17.5. Les conteneurs ajoutent une couche

Dans un conteneur, nous devons croiser `/proc` avec les cgroups.

Sinon, nous risquons de mal interpréter les limites mémoire.


## 6.18. Exercices

## Exercice 1 — Observer `maps` sur le shell courant

Nous exécutons :

```bash
cat /proc/$$/maps | head -30
```

Nous répondons :

1. Quelles sont les permissions observées ?
    
2. Voyons-nous le binaire du shell ?
    
3. Voyons-nous des bibliothèques partagées ?
    
4. Voyons-nous `[heap]` ?
    
5. Voyons-nous `[stack]` ?
    


## Exercice 2 — Comparer `VmSize` et `VmRSS`

Nous exécutons :

```bash
grep -E '^(VmSize|VmRSS):' /proc/$$/status
```

Nous répondons :

1. Quelle valeur est la plus élevée ?
    
2. Pourquoi ?
    
3. Laquelle est plus proche de la RAM réellement utilisée ?
    
4. Pourquoi même `VmRSS` reste imparfait ?
    


## Exercice 3 — Créer un processus qui alloue de la mémoire

Nous lançons :

```bash
python3 -c 'import time; data = bytearray(200 * 1024 * 1024); time.sleep(300)' &
pid=$!
```

Nous observons :

```bash
grep -E '^(VmSize|VmRSS|RssAnon):' /proc/$pid/status
```

Puis :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Anonymous):' /proc/$pid/smaps_rollup
```

Nous expliquons pourquoi `RssAnon` et `Private_Dirty` augmentent.

Nous terminons :

```bash
kill $pid
```


## Exercice 4 — Lister les bibliothèques chargées

Nous choisissons un processus :

```bash
pid=$$
```

Puis :

```bash
awk '{print $6}' /proc/$pid/maps 2>/dev/null | grep '^/' | sort -u
```

Nous répondons :

1. Quelles bibliothèques voyons-nous ?
    
2. Voyons-nous `libc` ?
    
3. Voyons-nous le linker dynamique ?
    
4. Pourquoi un processus charge-t-il plusieurs bibliothèques ?
    


## Exercice 5 — Observer les limites

Nous exécutons :

```bash
cat /proc/$$/limits
```

Nous répondons :

1. Quelle est la limite de pile ?
    
2. Quelle est la limite du nombre de fichiers ouverts ?
    
3. Les core dumps sont-ils autorisés ?
    
4. Quelle est la différence entre limite souple et limite dure ?
    


## Exercice 6 — Détecter une croissance mémoire

Nous lançons un programme Python qui alloue progressivement :

```bash
python3 - <<'PY' &
import time
data = []
while True:
    data.append(bytearray(10 * 1024 * 1024))
    time.sleep(2)
PY
pid=$!
```

Nous observons :

```bash
watch -n 2 "grep -E '^(VmRSS|RssAnon):' /proc/$pid/status"
```

Nous répondons :

1. Quelles valeurs augmentent ?
    
2. Pourquoi ?
    
3. En quoi cela ressemble-t-il à une fuite mémoire ?
    
4. Quelles informations supplémentaires faudrait-il pour conclure ?
    

Nous terminons :

```bash
kill $pid
```


## 6.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. Un processus possède un espace mémoire virtuel.
    
2. La mémoire virtuelle ne correspond pas directement à la RAM consommée.
    
3. `/proc/<PID>/maps` décrit les zones mémoire du processus.
    
4. Les permissions `r`, `w`, `x`, `p`, `s` décrivent les droits et le type de mapping.
    
5. `[heap]` correspond au tas et `[stack]` à la pile.
    
6. Les bibliothèques partagées apparaissent comme des fichiers mappés.
    
7. La mémoire anonyme est souvent liée aux allocations dynamiques.
    
8. `/proc/<PID>/smaps` donne des statistiques détaillées par mapping.
    
9. `/proc/<PID>/smaps_rollup` donne une synthèse pratique.
    
10. Le RSS mesure la mémoire résidente, mais peut compter de la mémoire partagée plusieurs fois.
    
11. Le PSS répartit la mémoire partagée et donne une estimation souvent plus juste.
    
12. `Private_Dirty` est utile pour estimer la mémoire réellement propre au processus.
    
13. `/proc/<PID>/limits` expose les limites de ressources du processus.
    
14. Une croissance mémoire n’est pas toujours une fuite : nous devons croiser avec le contexte applicatif.
    
15. Dans les conteneurs, nous devons compléter `/proc` avec les cgroups.
    


## Conclusion du chapitre 6

Nous savons maintenant analyser la mémoire d’un processus avec `/proc`.

Nous avons appris à lire la cartographie mémoire avec `maps`, à interpréter les permissions des mappings, à identifier le tas, la pile, les bibliothèques partagées et la mémoire anonyme. Nous avons aussi étudié `smaps` et `smaps_rollup`, qui permettent d’aller au-delà de `VmRSS` pour mieux comprendre la mémoire réellement utilisée par un processus.

Nous retenons surtout qu’une analyse mémoire sérieuse demande de distinguer mémoire virtuelle, mémoire résidente, mémoire partagée, mémoire privée et mémoire proportionnelle. Sans ces distinctions, nous risquons de diagnostiquer à tort une fuite mémoire ou une surconsommation.

Dans le chapitre suivant, nous étudions le réseau à travers `/proc/net`, notamment les interfaces, les sockets TCP, UDP et Unix.

# Chapitre 7 — Réseau et `/proc`

## Objectifs du chapitre

Dans ce chapitre, nous étudions les informations réseau exposées par `/proc`.

Nous avons déjà vu que `/proc` permet d’observer les processus, leurs fichiers ouverts et leur mémoire. Nous allons maintenant nous intéresser à la pile réseau du noyau Linux.

Historiquement, plusieurs informations réseau sont exposées dans :

```bash
/proc/net
```

Nous y trouvons notamment :

```text
/proc/net/dev
/proc/net/tcp
/proc/net/udp
/proc/net/unix
/proc/net/route
/proc/net/arp
/proc/net/snmp
```

Ces fichiers sont souvent moins lisibles que les commandes modernes comme `ip`, `ss` ou `netstat`, mais ils permettent de comprendre comment le noyau expose l’état réseau.

À la fin de ce chapitre, nous savons :

- lire les statistiques des interfaces réseau avec `/proc/net/dev` ;
    
- comprendre les sockets TCP avec `/proc/net/tcp` ;
    
- comprendre les sockets UDP avec `/proc/net/udp` ;
    
- observer les sockets Unix avec `/proc/net/unix` ;
    
- faire le lien entre les sockets vues dans `/proc/<PID>/fd` et `/proc/net` ;
    
- comprendre pourquoi les fichiers réseau de `/proc` sont difficiles à lire directement ;
    
- comparer `/proc/net` avec `ss`, `ip addr`, `ip route` et `lsof`.
    


## 7.1. Vue d’ensemble de `/proc/net`

### 7.1.1. Explorer `/proc/net`

Nous commençons par lister le contenu de `/proc/net` :

```bash
ls /proc/net
```

Nous pouvons obtenir de nombreuses entrées :

```text
arp
dev
if_inet6
netlink
netstat
packet
protocols
route
snmp
tcp
tcp6
udp
udp6
unix
wireless
```

Toutes ces entrées ne nous intéressent pas au même niveau.

Pour ce cours, nous nous concentrons principalement sur :

```text
dev
tcp
udp
unix
route
arp
snmp
```


### 7.1.2. `/proc/net` est lié au namespace réseau

Un point important : `/proc/net` ne représente pas toujours le réseau global de la machine physique.

Dans les systèmes modernes, `/proc/net` dépend du namespace réseau du processus qui le lit.

Dans un conteneur Docker ou un pod Kubernetes, un processus peut avoir sa propre pile réseau isolée. Dans ce cas, `/proc/net` affiche la vue réseau du conteneur, pas forcément celle de l’hôte.

Nous retenons :

```text
/proc/net reflète le namespace réseau courant.
```

Cela est essentiel pour comprendre les différences entre :

```bash
cat /proc/net/dev
```

exécuté sur l’hôte, et la même commande exécutée dans un conteneur.


### 7.1.3. Comparaison avec les commandes modernes

Les fichiers de `/proc/net` sont souvent bruts.

En pratique, nous utilisons plutôt :

```bash
ip addr
ip link
ip route
ss -tulpen
ss -x
```

Mais ces commandes s’appuient sur des informations du noyau et fournissent une vue plus lisible.

Nous étudions `/proc/net` non pas pour remplacer ces outils, mais pour comprendre la source bas niveau.


## 7.2. Interfaces réseau avec `/proc/net/dev`

### 7.2.1. Lire `/proc/net/dev`

Le fichier :

```bash
cat /proc/net/dev
```

affiche les statistiques des interfaces réseau.

Exemple :

```text
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
    lo: 1234567    1200    0    0    0     0          0         0 1234567    1200    0    0    0     0       0          0
  eth0: 987654321  85432   0    3    0     0          0       120 456789123  62341   0    0    0     0       0          0
```

Nous trouvons une ligne par interface réseau.


### 7.2.2. Colonnes principales

Chaque interface possède deux blocs de statistiques :

```text
Receive  → trafic reçu
Transmit → trafic envoyé
```

Les colonnes importantes sont :

|Colonne|Signification|
|---|---|
|`bytes`|nombre d’octets reçus ou envoyés|
|`packets`|nombre de paquets reçus ou envoyés|
|`errs`|erreurs|
|`drop`|paquets abandonnés|
|`fifo`|erreurs de file FIFO|
|`frame`|erreurs de trame|
|`multicast`|paquets multicast reçus|
|`colls`|collisions, surtout historique Ethernet|
|`carrier`|erreurs de porteuse|

Dans un diagnostic simple, nous regardons surtout :

- `bytes` ;
    
- `packets` ;
    
- `errs` ;
    
- `drop`.
    


### 7.2.3. Identifier les interfaces

Nous pouvons afficher seulement les noms d’interfaces :

```bash
awk -F: 'NR>2 {gsub(/ /,"",$1); print $1}' /proc/net/dev
```

Sortie possible :

```text
lo
eth0
wlan0
docker0
br-123456
vethabcd
```

Nous pouvons rencontrer :

|Interface|Rôle courant|
|---|---|
|`lo`|loopback local|
|`eth0`|interface Ethernet classique|
|`wlan0`|interface Wi-Fi|
|`docker0`|pont réseau Docker|
|`br-*`|bridge Linux|
|`veth*`|paire virtuelle, souvent pour conteneurs|
|`tun0`|tunnel VPN|
|`wg0`|interface WireGuard|


### 7.2.4. Observer le trafic dans le temps

`/proc/net/dev` contient des compteurs cumulés depuis l’activation de l’interface.

Pour voir le trafic évoluer :

```bash
watch -n 1 'cat /proc/net/dev'
```

Nous pouvons aussi filtrer une interface :

```bash
watch -n 1 "grep eth0 /proc/net/dev"
```

Mais la lecture brute est peu pratique.

Nous pouvons écrire une commande plus simple :

```bash
awk -F'[: ]+' '/eth0:/ {print "RX bytes=" $3, "TX bytes=" $11}' /proc/net/dev
```

Attention : les positions peuvent être délicates à cause des espaces. Nous devons tester nos commandes.


### 7.2.5. Calculer un débit approximatif

Nous pouvons mesurer les octets reçus et envoyés à deux instants.

Exemple simple :

```bash
iface=eth0

rx1=$(awk -F'[: ]+' -v i="$iface" '$2==i {print $3}' /proc/net/dev)
tx1=$(awk -F'[: ]+' -v i="$iface" '$2==i {print $11}' /proc/net/dev)

sleep 1

rx2=$(awk -F'[: ]+' -v i="$iface" '$2==i {print $3}' /proc/net/dev)
tx2=$(awk -F'[: ]+' -v i="$iface" '$2==i {print $11}' /proc/net/dev)

echo "RX: $((rx2-rx1)) octets/s"
echo "TX: $((tx2-tx1)) octets/s"
```

Ce script donne un débit approximatif sur une seconde.

En production, nous utilisons plutôt des outils comme :

```bash
sar
iftop
nload
bmon
ip -s link
```

Mais l’exercice permet de comprendre le principe.


### 7.2.6. Comparaison avec `ip -s link`

La commande moderne équivalente est :

```bash
ip -s link
```

Elle affiche les statistiques par interface de manière plus lisible.

Nous pouvons comparer :

```bash
cat /proc/net/dev
ip -s link
```

Nous retenons :

```text
/proc/net/dev donne les compteurs bruts.
/ip -s link donne une présentation plus lisible.
```


## 7.3. Sockets TCP avec `/proc/net/tcp`

### 7.3.1. Lire `/proc/net/tcp`

Le fichier :

```bash
cat /proc/net/tcp
```

expose les sockets TCP IPv4 du namespace réseau courant.

Exemple :

```text
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode
   0: 0100007F:1F90 00000000:0000 0A 00000000:00000000 00:00000000 00000000 1000        0 123456 1 0000000000000000 100 0 0 10 0
```

Ce format est compact et peu humain.

Nous devons apprendre à décoder les champs principaux.


### 7.3.2. Colonnes principales

Les colonnes importantes sont :

|Colonne|Signification|
|---|---|
|`local_address`|adresse IP locale et port local|
|`rem_address`|adresse IP distante et port distant|
|`st`|état TCP|
|`tx_queue`|file d’envoi|
|`rx_queue`|file de réception|
|`uid`|UID propriétaire|
|`inode`|inode de la socket|

Les champs les plus utiles pour un premier diagnostic sont :

```text
local_address
rem_address
st
uid
inode
```


### 7.3.3. Format des adresses IP et ports

Dans `/proc/net/tcp`, les adresses IPv4 sont affichées en hexadécimal, avec les octets en ordre little-endian.

Exemple :

```text
0100007F:1F90
```

Le port est :

```text
1F90
```

en hexadécimal.

Convertissons le port :

```bash
printf "%d\n" 0x1F90
```

Résultat :

```text
8080
```

L’adresse :

```text
0100007F
```

correspond à :

```text
127.0.0.1
```

car les octets sont inversés :

```text
01 00 00 7F → 127.0.0.1
```


### 7.3.4. Décoder une adresse IPv4

Nous pouvons écrire une petite fonction Bash :

```bash
hex_to_ip() {
    local hex="$1"
    printf "%d.%d.%d.%d" \
        "0x${hex:6:2}" \
        "0x${hex:4:2}" \
        "0x${hex:2:2}" \
        "0x${hex:0:2}"
}
```

Exemple :

```bash
hex_to_ip 0100007F
```

Sortie :

```text
127.0.0.1
```

Pour séparer adresse et port :

```bash
entry="0100007F:1F90"
ip_hex=${entry%:*}
port_hex=${entry#*:}

hex_to_ip "$ip_hex"
printf ":%d\n" "0x$port_hex"
```


### 7.3.5. États TCP

La colonne `st` indique l’état de la socket TCP sous forme hexadécimale.

Quelques états courants :

|Code|État TCP|
|---|---|
|`01`|`ESTABLISHED`|
|`02`|`SYN_SENT`|
|`03`|`SYN_RECV`|
|`04`|`FIN_WAIT1`|
|`05`|`FIN_WAIT2`|
|`06`|`TIME_WAIT`|
|`07`|`CLOSE`|
|`08`|`CLOSE_WAIT`|
|`09`|`LAST_ACK`|
|`0A`|`LISTEN`|
|`0B`|`CLOSING`|

Exemple :

```text
st = 0A
```

signifie :

```text
LISTEN
```

Donc la socket écoute des connexions entrantes.


### 7.3.6. Identifier les sockets en écoute

Nous pouvons afficher les lignes TCP en état `LISTEN` :

```bash
awk 'NR>1 && $4=="0A" {print}' /proc/net/tcp
```

Cela donne les sockets TCP IPv4 en écoute, mais le format reste difficile.

La commande moderne équivalente est :

```bash
ss -ltn
```

ou avec les processus :

```bash
sudo ss -ltnp
```


### 7.3.7. Comparaison avec `ss`

Nous comparons :

```bash
cat /proc/net/tcp
ss -tan
```

`ss` affiche les informations de manière lisible :

```text
State      Recv-Q Send-Q Local Address:Port   Peer Address:Port
LISTEN     0      4096   127.0.0.1:8080      0.0.0.0:*
ESTAB      0      0      192.168.1.10:443    192.168.1.20:51322
```

Nous retenons :

```text
/proc/net/tcp expose les données brutes.
ss les décode et les présente clairement.
```


## 7.4. Faire le lien entre socket et processus

### 7.4.1. Rappel : sockets dans `/proc/<PID>/fd`

Dans le chapitre 5, nous avons vu qu’un processus peut avoir un descripteur de type socket :

```bash
ls -l /proc/<PID>/fd
```

Exemple :

```text
7 -> socket:[123456]
```

Le nombre `123456` est l’inode de la socket.

Dans `/proc/net/tcp`, nous avons aussi une colonne `inode`.

Nous pouvons donc relier une ligne de `/proc/net/tcp` à un processus.


### 7.4.2. Trouver la ligne TCP correspondant à une socket

Si nous avons :

```text
socket:[123456]
```

nous cherchons dans `/proc/net/tcp` :

```bash
awk '$10=="123456" {print}' /proc/net/tcp
```

Selon les fichiers, la colonne de l’inode est souvent la dixième, mais nous devons vérifier l’en-tête.

Exemple :

```bash
head -1 /proc/net/tcp
```


### 7.4.3. Trouver le processus à partir d’un inode de socket

À l’inverse, si nous avons une socket dans `/proc/net/tcp` avec l’inode `123456`, nous pouvons chercher quel processus la possède :

```bash
sudo sh -c '
inode=123456
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    if [ "$target" = "socket:[$inode]" ]; then
        pid=$(echo "$fd" | cut -d/ -f3)
        comm=$(cat "/proc/$pid/comm" 2>/dev/null || echo "?")
        echo "PID=$pid COMM=$comm FD=${fd##*/}"
    fi
done
'
```

Sortie possible :

```text
PID=2451 COMM=nginx FD=7
```

Nous venons de faire manuellement ce que `ss -p` ou `lsof -i` font de manière plus lisible.


### 7.4.4. Pourquoi cette correspondance est importante ?

Cette correspondance permet de répondre à des questions pratiques :

- quel processus écoute sur ce port ?
    
- quel processus possède cette connexion TCP ?
    
- pourquoi un port est-il déjà utilisé ?
    
- quel service garde une connexion ouverte ?
    
- quel processus établit des connexions suspectes ?
    

Dans la pratique, nous utilisons souvent :

```bash
sudo ss -tulpen
```

Mais comprendre le lien inode-socket-processus est fondamental.


## 7.5. Sockets UDP avec `/proc/net/udp`

### 7.5.1. Lire `/proc/net/udp`

Le fichier :

```bash
cat /proc/net/udp
```

expose les sockets UDP IPv4.

Exemple :

```text
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode
  10: 00000000:0035 00000000:0000 07 00000000:00000000 00:00000000 00000000     0        0 22222 2 0000000000000000 0
```

Ici, le port local :

```text
0035
```

correspond à :

```bash
printf "%d\n" 0x0035
```

Résultat :

```text
53
```

Il peut donc s’agir d’un service DNS.


### 7.5.2. Différence entre TCP et UDP

TCP est orienté connexion.

Il possède des états comme :

```text
LISTEN
ESTABLISHED
TIME_WAIT
CLOSE_WAIT
```

UDP est sans connexion.

Il n’a pas la même logique d’état que TCP.

Dans `/proc/net/udp`, nous voyons surtout :

- adresse locale ;
    
- port local ;
    
- adresse distante éventuelle ;
    
- files d’attente ;
    
- UID ;
    
- inode.
    


### 7.5.3. Commandes modernes équivalentes

Pour les sockets UDP :

```bash
ss -uan
```

Pour les sockets UDP en écoute :

```bash
ss -lun
```

Avec les processus :

```bash
sudo ss -lunp
```

Nous comparons :

```bash
cat /proc/net/udp
sudo ss -lunp
```

`ss` est plus lisible, mais `/proc/net/udp` montre la représentation brute.


## 7.6. IPv6 : `/proc/net/tcp6` et `/proc/net/udp6`

### 7.6.1. Lire les sockets IPv6

Les sockets TCP IPv6 sont dans :

```bash
cat /proc/net/tcp6
```

Les sockets UDP IPv6 sont dans :

```bash
cat /proc/net/udp6
```

Le format ressemble à `/proc/net/tcp` et `/proc/net/udp`, mais les adresses sont plus longues.


### 7.6.2. Lisibilité encore plus faible

Les adresses IPv6 sont encodées en hexadécimal sur 128 bits.

Exemple :

```text
00000000000000000000000000000000:1F90
```

Cela peut représenter :

```text
[::]:8080
```

Nous n’allons pas écrire un décodeur IPv6 complet ici.

En pratique, nous utilisons :

```bash
ss -tulpen
```

qui affiche TCP et UDP, IPv4 et IPv6, de manière lisible.


## 7.7. Sockets Unix avec `/proc/net/unix`

### 7.7.1. Qu’est-ce qu’une socket Unix ?

Une socket Unix est un mécanisme de communication inter-processus local à la machine.

Contrairement à une socket TCP, elle n’utilise pas une adresse IP et un port.

Elle peut être associée à un chemin de fichier, par exemple :

```text
/run/docker.sock
/var/run/postgresql/.s.PGSQL.5432
/run/systemd/private
```

Elle peut aussi être anonyme.


### 7.7.2. Lire `/proc/net/unix`

Nous lisons :

```bash
cat /proc/net/unix
```

Exemple :

```text
Num       RefCount Protocol Flags    Type St Inode Path
00000000  00000002 00000000 00010000 0001 01 12345 /run/systemd/private
00000000  00000002 00000000 00010000 0001 01 67890 /run/docker.sock
00000000  00000003 00000000 00000000 0001 03 54321
```

Nous trouvons notamment :

|Colonne|Signification|
|---|---|
|`RefCount`|compteur de références|
|`Flags`|flags|
|`Type`|type de socket|
|`St`|état|
|`Inode`|inode|
|`Path`|chemin éventuel|


### 7.7.3. Sockets Unix nommées et anonymes

Une socket Unix peut avoir un chemin :

```text
/run/docker.sock
```

ou ne pas en avoir.

Une socket sans chemin peut être utilisée pour une communication interne entre processus apparentés.

Nous pouvons chercher une socket précise :

```bash
grep docker /proc/net/unix
```

ou :

```bash
grep postgres /proc/net/unix
```


### 7.7.4. Comparaison avec `ss -x`

La commande moderne est :

```bash
ss -x
```

Pour les sockets Unix en écoute :

```bash
ss -xl
```

Avec les processus :

```bash
sudo ss -xlp
```

Nous comparons :

```bash
cat /proc/net/unix
sudo ss -xlp
```

`ss` permet de voir plus facilement quel processus possède quelle socket Unix.


## 7.8. Table ARP avec `/proc/net/arp`

### 7.8.1. Rôle d’ARP

ARP signifie Address Resolution Protocol.

Il permet d’associer une adresse IPv4 locale à une adresse MAC sur un réseau local.

Par exemple :

```text
192.168.1.1 → aa:bb:cc:dd:ee:ff
```


### 7.8.2. Lire `/proc/net/arp`

Nous lisons :

```bash
cat /proc/net/arp
```

Exemple :

```text
IP address       HW type     Flags       HW address            Mask     Device
192.168.1.1      0x1         0x2         aa:bb:cc:dd:ee:ff     *        eth0
192.168.1.20     0x1         0x2         11:22:33:44:55:66     *        eth0
```

Nous voyons :

- adresse IP ;
    
- adresse MAC ;
    
- interface ;
    
- flags.
    


### 7.8.3. Commandes modernes équivalentes

Nous utilisons généralement :

```bash
ip neigh
```

ou :

```bash
arp -n
```

si l’outil `arp` est installé.

Nous comparons :

```bash
cat /proc/net/arp
ip neigh
```


## 7.9. Routes avec `/proc/net/route`

### 7.9.1. Lire `/proc/net/route`

Le fichier :

```bash
cat /proc/net/route
```

affiche la table de routage IPv4.

Exemple :

```text
Iface Destination Gateway     Flags RefCnt Use Metric Mask        MTU Window IRTT
eth0  00000000    0101A8C0    0003  0      0   100    00000000    0   0      0
eth0  0001A8C0    00000000    0001  0      0   100    00FFFFFF    0   0      0
```

Le format est encore une fois en hexadécimal little-endian.


### 7.9.2. Route par défaut

La destination :

```text
00000000
```

avec un masque :

```text
00000000
```

correspond généralement à la route par défaut.

Dans l’exemple :

```text
Gateway = 0101A8C0
```

se décode en :

```text
192.168.1.1
```


### 7.9.3. Commande moderne équivalente

En pratique, nous utilisons :

```bash
ip route
```

Exemple :

```text
default via 192.168.1.1 dev eth0 proto dhcp metric 100
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10 metric 100
```

Nous retenons :

```text
/proc/net/route donne la table brute.
/ip route donne une vue lisible et complète.
```


## 7.10. Statistiques TCP/IP avec `/proc/net/snmp` et `/proc/net/netstat`

### 7.10.1. Lire `/proc/net/snmp`

Le fichier :

```bash
cat /proc/net/snmp
```

contient des statistiques réseau cumulées.

Nous pouvons y voir des sections comme :

```text
Ip:
Icmp:
Tcp:
Udp:
```

Exemple simplifié :

```text
Tcp: RtoAlgorithm RtoMin RtoMax MaxConn ActiveOpens PassiveOpens AttemptFails EstabResets CurrEstab InSegs OutSegs RetransSegs
Tcp: 1 200 120000 -1 123 45 2 1 4 123456 120000 12
```

La première ligne donne les noms des champs.

La seconde donne les valeurs.


### 7.10.2. Interprétation

Ces statistiques permettent d’observer :

- connexions TCP actives ;
    
- connexions passives ;
    
- segments reçus ;
    
- segments envoyés ;
    
- retransmissions ;
    
- erreurs ;
    
- datagrammes UDP ;
    
- messages ICMP.
    

Nous pouvons extraire les retransmissions TCP :

```bash
awk '
/^Tcp:/ {
    if (!headers_seen) {
        for (i=1; i<=NF; i++) h[i]=$i
        headers_seen=1
    } else {
        for (i=1; i<=NF; i++) v[h[i]]=$i
        print "RetransSegs=" v["RetransSegs"]
    }
}
' /proc/net/snmp
```

Ce type de parsing est pédagogique, mais en production nous utilisons plutôt des outils de monitoring.


### 7.10.3. `/proc/net/netstat`

Le fichier :

```bash
cat /proc/net/netstat
```

expose d’autres statistiques, souvent plus détaillées.

Nous pouvons y trouver des compteurs TCP étendus, selon les versions du noyau.

Exemple de section :

```text
TcpExt:
IpExt:
```

Ces informations sont utiles pour du diagnostic réseau avancé, mais leur interprétation demande de bonnes connaissances TCP/IP.


## 7.11. Limites de lisibilité de `/proc/net`

### 7.11.1. Formats bruts

Les fichiers de `/proc/net` sont souvent :

- compacts ;
    
- encodés en hexadécimal ;
    
- peu documentés dans leur format exact pour un usage humain ;
    
- plus adaptés aux outils qu’à la lecture directe.
    

Exemple :

```text
0100007F:1F90
```

est moins lisible que :

```text
127.0.0.1:8080
```


### 7.11.2. Pourquoi conserver ces fichiers ?

Même s’ils sont peu lisibles, ces fichiers restent utiles :

- ils exposent une interface simple ;
    
- ils sont accessibles avec des outils basiques ;
    
- ils permettent de comprendre ce que voient les outils système ;
    
- ils fonctionnent dans des environnements minimaux ;
    
- ils aident à diagnostiquer sans dépendre d’outils installés.
    


### 7.11.3. Outils à privilégier en pratique

En pratique, nous utilisons souvent :

```bash
ip addr
ip link
ip -s link
ip route
ip neigh
ss -tulpen
ss -xlp
lsof -i
```

Nous gardons `/proc/net` pour :

- l’apprentissage ;
    
- le debugging bas niveau ;
    
- les scripts très légers ;
    
- les conteneurs minimaux ;
    
- la compréhension des sources d’information.
    


## 7.12. Cas pratique : quel processus écoute sur un port ?

### 7.12.1. Avec les outils modernes

Supposons que nous voulons savoir quel processus écoute sur le port `8080`.

Nous utilisons directement :

```bash
sudo ss -ltnp 'sport = :8080'
```

Sortie possible :

```text
State  Recv-Q Send-Q Local Address:Port Peer Address:Port Process
LISTEN 0      4096   127.0.0.1:8080    0.0.0.0:*         users:(("node",pid=2451,fd=18))
```

Nous obtenons :

- le processus ;
    
- son PID ;
    
- le descripteur ;
    
- l’adresse locale ;
    
- le port.
    


### 7.12.2. Avec `/proc/net/tcp`

Nous pouvons faire le même raisonnement à la main.

Le port `8080` en hexadécimal est :

```bash
printf "%04X\n" 8080
```

Résultat :

```text
1F90
```

Nous cherchons les sockets en écoute sur ce port :

```bash
awk '$2 ~ /:1F90$/ && $4=="0A" {print}' /proc/net/tcp
```

Nous récupérons l’inode, par exemple :

```text
123456
```

Puis nous cherchons le processus propriétaire :

```bash
sudo sh -c '
inode=123456
for fd in /proc/[0-9]*/fd/*; do
    target=$(readlink "$fd" 2>/dev/null) || continue
    if [ "$target" = "socket:[$inode]" ]; then
        pid=$(echo "$fd" | cut -d/ -f3)
        comm=$(cat "/proc/$pid/comm" 2>/dev/null || echo "?")
        echo "PID=$pid COMM=$comm FD=${fd##*/}"
    fi
done
'
```

Nous retrouvons le processus.

Cette méthode est plus longue, mais elle montre la relation entre :

```text
/proc/net/tcp
/proc/<PID>/fd
```


## 7.13. Cas pratique : analyser rapidement l’activité réseau

### 7.13.1. Objectif

Nous voulons faire une première analyse réseau d’une machine.

Nous cherchons à répondre :

- quelles interfaces existent ?
    
- quelles interfaces échangent du trafic ?
    
- quelles sockets TCP écoutent ?
    
- quelles connexions sont établies ?
    
- quelles sockets Unix importantes existent ?
    
- quelle route par défaut est utilisée ?
    


### 7.13.2. Interfaces

Nous commençons par :

```bash
cat /proc/net/dev
```

Puis nous utilisons :

```bash
ip -s link
```

Nous identifions :

- interface principale ;
    
- loopback ;
    
- interfaces Docker ou bridge ;
    
- interfaces VPN ;
    
- erreurs ou drops.
    


### 7.13.3. Ports TCP en écoute

Nous utilisons :

```bash
ss -ltnp
```

Puis nous comparons avec :

```bash
awk 'NR>1 && $4=="0A" {print}' /proc/net/tcp
```

Nous observons que `ss` est plus lisible.


### 7.13.4. Connexions TCP établies

Nous utilisons :

```bash
ss -tan state established
```

Dans `/proc/net/tcp`, l’état `ESTABLISHED` correspond à :

```text
01
```

Nous pouvons donc faire :

```bash
awk 'NR>1 && $4=="01" {print}' /proc/net/tcp
```


### 7.13.5. Sockets Unix

Nous regardons :

```bash
cat /proc/net/unix | head
```

Puis :

```bash
sudo ss -xlp
```

Nous cherchons notamment :

- Docker ;
    
- PostgreSQL ;
    
- systemd ;
    
- services locaux ;
    
- sockets applicatives.
    


### 7.13.6. Route par défaut

Nous utilisons :

```bash
ip route
```

Puis nous observons la source brute :

```bash
cat /proc/net/route
```

Nous identifions la route par défaut.


## 7.14. Scripts pédagogiques

### 7.14.1. Afficher les interfaces depuis `/proc/net/dev`

```bash
##!/usr/bin/env bash

printf "%-15s %15s %15s %10s %10s\n" "Interface" "RX bytes" "TX bytes" "RX drop" "TX drop"

awk -F'[: ]+' '
NR > 2 {
    iface=$2
    rx_bytes=$3
    rx_drop=$6
    tx_bytes=$11
    tx_drop=$14
    printf "%-15s %15s %15s %10s %10s\n", iface, rx_bytes, tx_bytes, rx_drop, tx_drop
}
' /proc/net/dev
```

Nous pouvons sauvegarder ce script sous :

```bash
netdev-summary.sh
```

Puis :

```bash
chmod +x netdev-summary.sh
./netdev-summary.sh
```


### 7.14.2. Décoder les états TCP principaux

```bash
tcp_state_name() {
    case "$1" in
        1) echo "ESTABLISHED" ;;
        2) echo "SYN_SENT" ;;
        3) echo "SYN_RECV" ;;
        4) echo "FIN_WAIT1" ;;
        5) echo "FIN_WAIT2" ;;
        6) echo "TIME_WAIT" ;;
        7) echo "CLOSE" ;;
        8) echo "CLOSE_WAIT" ;;
        9) echo "LAST_ACK" ;;
        0A) echo "LISTEN" ;;
        0B) echo "CLOSING" ;;
        *) echo "UNKNOWN" ;;
    esac
}
```

Nous pouvons utiliser cette fonction dans un script plus complet.


### 7.14.3. Décoder une adresse IPv4 de `/proc/net/tcp`

```bash
hex_to_ip() {
    local hex="$1"
    printf "%d.%d.%d.%d" \
        "0x${hex:6:2}" \
        "0x${hex:4:2}" \
        "0x${hex:2:2}" \
        "0x${hex:0:2}"
}

decode_addr_port() {
    local value="$1"
    local ip_hex="${value%:*}"
    local port_hex="${value#*:}"

    hex_to_ip "$ip_hex"
    printf ":%d" "0x$port_hex"
}
```

Exemple :

```bash
decode_addr_port 0100007F:1F90
echo
```

Sortie :

```text
127.0.0.1:8080
```


### 7.14.4. Mini-lecture lisible de `/proc/net/tcp`

```bash
##!/usr/bin/env bash

hex_to_ip() {
    local hex="$1"
    printf "%d.%d.%d.%d" \
        "0x${hex:6:2}" \
        "0x${hex:4:2}" \
        "0x${hex:2:2}" \
        "0x${hex:0:2}"
}

decode_addr_port() {
    local value="$1"
    local ip_hex="${value%:*}"
    local port_hex="${value#*:}"

    hex_to_ip "$ip_hex"
    printf ":%d" "0x$port_hex"
}

tcp_state_name() {
    case "$1" in
        1) echo "ESTABLISHED" ;;
        2) echo "SYN_SENT" ;;
        3) echo "SYN_RECV" ;;
        4) echo "FIN_WAIT1" ;;
        5) echo "FIN_WAIT2" ;;
        6) echo "TIME_WAIT" ;;
        7) echo "CLOSE" ;;
        8) echo "CLOSE_WAIT" ;;
        9) echo "LAST_ACK" ;;
        0A) echo "LISTEN" ;;
        0B) echo "CLOSING" ;;
        *) echo "UNKNOWN" ;;
    esac
}

printf "%-22s %-22s %-15s %-10s %-10s\n" "Local" "Remote" "State" "UID" "Inode"

tail -n +2 /proc/net/tcp | while read -r sl local remote st rest; do
    uid=$(echo "$rest" | awk '{print $4}')
    inode=$(echo "$rest" | awk '{print $6}')

    printf "%-22s %-22s %-15s %-10s %-10s\n" \
        "$(decode_addr_port "$local")" \
        "$(decode_addr_port "$remote")" \
        "$(tcp_state_name "$st")" \
        "$uid" \
        "$inode"
done
```

Nous obtenons une version pédagogique simplifiée de `ss`.


## 7.15. Pièges et limites

### 7.15.1. Les formats sont peu lisibles

Les fichiers comme `/proc/net/tcp` et `/proc/net/route` sont difficiles à lire à la main.

Ils utilisent :

- hexadécimal ;
    
- little-endian ;
    
- codes numériques ;
    
- colonnes compactes.
    

Nous devons accepter que ces fichiers sont plutôt conçus pour les outils que pour les humains.


### 7.15.2. Les namespaces changent la vue

Dans un conteneur, `/proc/net` peut être différent de celui de l’hôte.

Si nous diagnostiquons un problème réseau Kubernetes, nous devons savoir si nous sommes :

- dans le pod ;
    
- dans le nœud ;
    
- dans un namespace réseau spécifique ;
    
- dans un conteneur privilégié ;
    
- dans un environnement CNI particulier.
    


### 7.15.3. Les permissions peuvent limiter l’association aux processus

Lire `/proc/net/tcp` est souvent possible, mais retrouver le processus via `/proc/<PID>/fd` peut nécessiter des droits plus élevés.

C’est pourquoi :

```bash
ss -tulpen
```

sans `sudo` peut parfois masquer le nom du processus, tandis que :

```bash
sudo ss -tulpen
```

donne plus d’informations.


### 7.15.4. Les sockets peuvent disparaître

Une connexion TCP peut se fermer pendant notre analyse.

Une socket peut apparaître puis disparaître rapidement.

Un script robuste doit tolérer :

```text
No such file or directory
Permission denied
```

lorsqu’il parcourt `/proc`.


### 7.15.5. `/proc/net` ne remplace pas une analyse réseau complète

`/proc/net` donne une vue locale du noyau.

Pour une analyse complète, nous pouvons aussi utiliser :

- captures réseau avec `tcpdump` ou Wireshark ;
    
- métriques applicatives ;
    
- logs de proxy ou pare-feu ;
    
- observabilité Kubernetes ;
    
- statistiques de switch ou routeur ;
    
- outils cloud ;
    
- tests actifs avec `curl`, `dig`, `ping`, `mtr`.
    


## 7.16. Exercices

## Exercice 1 — Observer les interfaces réseau

Nous exécutons :

```bash
cat /proc/net/dev
```

Puis :

```bash
ip -s link
```

Nous répondons :

1. Quelles interfaces réseau voyons-nous ?
    
2. Quelle interface semble être l’interface principale ?
    
3. Combien d’octets ont été reçus et envoyés ?
    
4. Y a-t-il des erreurs ou des paquets abandonnés ?
    
5. Les informations sont-elles plus lisibles avec `ip -s link` ?
    


## Exercice 2 — Observer les sockets TCP

Nous exécutons :

```bash
cat /proc/net/tcp
```

Puis :

```bash
ss -tan
```

Nous répondons :

1. Quels états TCP voyons-nous ?
    
2. Voyons-nous des sockets en `LISTEN` ?
    
3. Voyons-nous des connexions `ESTABLISHED` ?
    
4. Pourquoi `/proc/net/tcp` est-il difficile à lire directement ?
    


## Exercice 3 — Lancer un serveur local et le retrouver

Nous lançons un serveur HTTP simple :

```bash
python3 -m http.server 8080 &
pid=$!
```

Nous vérifions avec :

```bash
ss -ltnp 'sport = :8080'
```

Puis nous cherchons dans `/proc/net/tcp` :

```bash
printf "%04X\n" 8080
awk '$2 ~ /:1F90$/ && $4=="0A" {print}' /proc/net/tcp
```

Nous répondons :

1. Quel port est utilisé ?
    
2. Quel est son encodage hexadécimal ?
    
3. Quel est l’état TCP ?
    
4. Quel inode de socket voyons-nous ?
    
5. Comment retrouvons-nous le processus propriétaire ?
    

Nous terminons :

```bash
kill $pid
```


## Exercice 4 — Observer les sockets Unix

Nous exécutons :

```bash
cat /proc/net/unix | head
```

Puis :

```bash
ss -x | head
```

Nous cherchons :

```bash
grep docker /proc/net/unix
grep systemd /proc/net/unix
```

Nous répondons :

1. Quelles sockets Unix nommées voyons-nous ?
    
2. Certaines sockets sont-elles anonymes ?
    
3. Pourquoi les sockets Unix sont-elles utiles ?
    
4. Pourquoi `ss -x` est-il plus lisible ?
    


## Exercice 5 — Lire la table ARP

Nous exécutons :

```bash
cat /proc/net/arp
```

Puis :

```bash
ip neigh
```

Nous répondons :

1. Quelles adresses IP locales sont connues ?
    
2. Quelles adresses MAC sont associées ?
    
3. Sur quelle interface ?
    
4. Que signifie le voisinage réseau ?
    


## Exercice 6 — Lire les routes

Nous exécutons :

```bash
cat /proc/net/route
```

Puis :

```bash
ip route
```

Nous répondons :

1. Quelle est la route par défaut ?
    
2. Quelle interface l’utilise ?
    
3. Pourquoi `/proc/net/route` est-il moins lisible ?
    
4. Pourquoi `ip route` est-il préférable en diagnostic courant ?
    


## 7.17. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc/net` expose une partie de l’état réseau du noyau.
    
2. `/proc/net/dev` donne les statistiques des interfaces réseau.
    
3. `/proc/net/tcp` expose les sockets TCP IPv4.
    
4. `/proc/net/udp` expose les sockets UDP IPv4.
    
5. `/proc/net/tcp6` et `/proc/net/udp6` exposent les sockets IPv6.
    
6. `/proc/net/unix` expose les sockets Unix locales.
    
7. Les adresses et ports dans `/proc/net/tcp` sont encodés en hexadécimal.
    
8. Les états TCP sont représentés par des codes comme `01` pour `ESTABLISHED` et `0A` pour `LISTEN`.
    
9. Le lien entre `/proc/net/tcp` et `/proc/<PID>/fd` se fait grâce à l’inode de socket.
    
10. Les commandes `ss`, `ip` et `lsof` offrent une vue plus lisible.
    
11. `/proc/net` dépend du namespace réseau courant.
    
12. Dans les conteneurs, la vue réseau peut différer de celle de l’hôte.
    
13. Les sockets peuvent apparaître et disparaître rapidement.
    
14. `/proc/net` est utile pour comprendre, mais ne remplace pas une analyse réseau complète.
    


## Conclusion du chapitre 7

Nous savons maintenant utiliser `/proc` pour observer une partie de l’état réseau d’un système Linux.

Nous avons étudié les interfaces réseau avec `/proc/net/dev`, les sockets TCP et UDP avec `/proc/net/tcp` et `/proc/net/udp`, les sockets Unix avec `/proc/net/unix`, ainsi que les informations liées à ARP, aux routes et aux statistiques TCP/IP.

Nous avons surtout compris que les fichiers de `/proc/net` sont des représentations brutes : ils sont précieux pour apprendre et diagnostiquer à bas niveau, mais nous utilisons en pratique des outils comme `ss`, `ip`, `lsof` ou `tcpdump` pour obtenir une vue plus lisible et plus complète.

Dans le chapitre suivant, nous étudions `/proc/sys`, qui permet non seulement de lire certains paramètres du noyau, mais aussi de les modifier temporairement.

# Chapitre 8 — Paramètres noyau avec `/proc/sys`

## Objectifs du chapitre

Dans ce chapitre, nous étudions une partie particulière de `/proc` :

```bash
/proc/sys
```

Jusqu’ici, nous avons surtout utilisé `/proc` pour observer l’état du système : processus, mémoire, fichiers ouverts, réseau. Avec `/proc/sys`, nous passons à une dimension plus sensible : certains fichiers permettent non seulement de lire des paramètres du noyau, mais aussi de les modifier.

À la fin de ce chapitre, nous savons :

- comprendre le rôle de `/proc/sys` ;
    
- lire des paramètres noyau ;
    
- identifier les grands domaines : `kernel`, `vm`, `net`, `fs`, `user`, `debug` ;
    
- modifier temporairement un paramètre noyau ;
    
- utiliser la commande `sysctl` ;
    
- rendre une configuration persistante ;
    
- comprendre les risques liés à la modification de paramètres noyau.
    


## 8.1. Présentation générale de `/proc/sys`

### 8.1.1. Une interface de configuration du noyau

Le répertoire `/proc/sys` expose des paramètres internes du noyau Linux.

Nous pouvons le parcourir avec :

```bash
ls /proc/sys
```

Sortie possible :

```text
abi
debug
dev
fs
kernel
net
sunrpc
user
vm
```

Ces sous-répertoires organisent les paramètres par domaine.

Contrairement à beaucoup de fichiers de `/proc`, certains fichiers de `/proc/sys` sont modifiables. Quand nous écrivons dedans, nous ne modifions pas un fichier texte ordinaire : nous demandons au noyau de changer un paramètre actif.

Exemple :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Si la valeur est :

```text
0
```

le routage IPv4 n’est pas activé.

Si nous écrivons :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

nous activons temporairement le forwarding IPv4.


### 8.1.2. Attention : nous agissons sur le noyau

Nous devons être très prudents.

Lire un paramètre est généralement sans risque :

```bash
cat /proc/sys/vm/swappiness
```

Modifier un paramètre peut avoir des conséquences immédiates :

```bash
echo 10 | sudo tee /proc/sys/vm/swappiness
```

Selon le paramètre, nous pouvons affecter :

- le réseau ;
    
- la mémoire ;
    
- la sécurité ;
    
- les limites système ;
    
- le comportement des fichiers ;
    
- les performances ;
    
- la stabilité.
    

Nous retenons donc :

```text
/proc/sys est une interface de configuration active du noyau.
Nous ne modifions pas ces paramètres sans comprendre leur effet.
```


## 8.2. Organisation de `/proc/sys`

### 8.2.1. Les grands domaines

Les répertoires principaux correspondent à de grands domaines du noyau.

|Répertoire|Rôle général|
|---|---|
|`/proc/sys/kernel`|paramètres généraux du noyau|
|`/proc/sys/vm`|gestion mémoire virtuelle|
|`/proc/sys/net`|pile réseau|
|`/proc/sys/fs`|systèmes de fichiers et limites associées|
|`/proc/sys/user`|limites liées aux namespaces et ressources utilisateur|
|`/proc/sys/debug`|paramètres de debug|
|`/proc/sys/dev`|certains périphériques|
|`/proc/sys/abi`|compatibilité ABI|
|`/proc/sys/sunrpc`|paramètres liés à SunRPC/NFS|

Nous pouvons explorer :

```bash
find /proc/sys -maxdepth 2 -type f | head
```

ou :

```bash
tree -L 2 /proc/sys 2>/dev/null
```

si `tree` est installé.


### 8.2.2. Fichiers simples, effets complexes

Beaucoup de paramètres sont représentés par des fichiers contenant une valeur simple.

Exemple :

```bash
cat /proc/sys/kernel/hostname
```

Sortie :

```text
machine-test
```

Autre exemple :

```bash
cat /proc/sys/vm/swappiness
```

Sortie :

```text
60
```

La valeur est simple, mais son interprétation peut être complexe.

Nous devons donc distinguer :

```text
facilité de lecture ≠ simplicité du comportement noyau
```


## 8.3. Lecture des paramètres noyau

### 8.3.1. Lire le nom d’hôte actif

Nous pouvons lire le nom d’hôte actif :

```bash
cat /proc/sys/kernel/hostname
```

Ce nom correspond au hostname courant du noyau.

Nous pouvons comparer avec :

```bash
hostname
```

Le fichier `/proc/sys/kernel/hostname` donne l’état actif, tandis que des fichiers comme `/etc/hostname` servent à la configuration persistante selon la distribution.


### 8.3.2. Lire la valeur de `swappiness`

Le paramètre `swappiness` influence la tendance du noyau à utiliser le swap.

```bash
cat /proc/sys/vm/swappiness
```

Valeur possible :

```text
60
```

Une valeur plus basse tend à réduire l’usage du swap, sans l’interdire totalement.

Nous ne devons pas simplifier abusivement :

```text
swappiness faible ≠ swap désactivé
swappiness élevé ≠ système forcément lent
```

C’est un paramètre d’arbitrage, pas un interrupteur absolu.


### 8.3.3. Lire le forwarding IPv4

Nous lisons :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Valeurs possibles :

```text
0
```

ou :

```text
1
```

Interprétation :

|Valeur|Signification|
|---|---|
|`0`|le noyau ne route pas les paquets IPv4 entre interfaces|
|`1`|le noyau peut router les paquets IPv4 entre interfaces|

Ce paramètre est important pour :

- routeurs Linux ;
    
- machines faisant du NAT ;
    
- VPN ;
    
- conteneurs ;
    
- Kubernetes ;
    
- passerelles réseau.
    


### 8.3.4. Lire les limites de fichiers

Nous pouvons lire :

```bash
cat /proc/sys/fs/file-max
```

Ce paramètre indique le nombre maximal de fichiers ouverts à l’échelle du système.

Nous pouvons aussi observer l’état courant avec :

```bash
cat /proc/sys/fs/file-nr
```

Sortie possible :

```text
12032   0   9223372036854775807
```

Selon le noyau, ces champs indiquent notamment le nombre de handles de fichiers alloués et la limite maximale.

Nous faisons le lien avec le chapitre 5 : un processus a ses propres descripteurs, mais le système possède aussi des limites globales.


## 8.4. Correspondance entre `/proc/sys` et `sysctl`

### 8.4.1. La commande `sysctl`

La commande `sysctl` permet de lire et modifier les paramètres exposés via `/proc/sys`.

Exemple :

```bash
sysctl vm.swappiness
```

Sortie :

```text
vm.swappiness = 60
```

Ce paramètre correspond au fichier :

```bash
/proc/sys/vm/swappiness
```

La correspondance est simple :

```text
/proc/sys/vm/swappiness
↔
vm.swappiness
```

Les `/` deviennent des `.`.


### 8.4.2. Exemples de correspondance

|Fichier `/proc/sys`|Nom `sysctl`|
|---|---|
|`/proc/sys/kernel/hostname`|`kernel.hostname`|
|`/proc/sys/vm/swappiness`|`vm.swappiness`|
|`/proc/sys/net/ipv4/ip_forward`|`net.ipv4.ip_forward`|
|`/proc/sys/fs/file-max`|`fs.file-max`|
|`/proc/sys/kernel/pid_max`|`kernel.pid_max`|

Nous pouvons donc lire :

```bash
cat /proc/sys/kernel/pid_max
```

ou :

```bash
sysctl kernel.pid_max
```

Les deux donnent la même information sous une forme différente.


### 8.4.3. Lister les paramètres disponibles

Nous pouvons lister tous les paramètres connus par `sysctl` :

```bash
sysctl -a
```

Cette commande peut produire beaucoup de lignes.

Nous filtrons souvent avec `grep` :

```bash
sysctl -a | grep swappiness
sysctl -a | grep ip_forward
sysctl -a | grep '^net.ipv4'
```

Nous pouvons aussi explorer directement `/proc/sys`.


## 8.5. Modifier temporairement un paramètre

### 8.5.1. Modification par écriture directe

Certains paramètres peuvent être modifiés en écrivant dans le fichier correspondant.

Exemple avec `ip_forward` :

```bash
cat /proc/sys/net/ipv4/ip_forward
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward
```

Nous modifions ici l’état actif du noyau.

Cette modification est temporaire : elle est perdue au redémarrage, sauf si elle est aussi configurée de manière persistante.


### 8.5.2. Modification avec `sysctl -w`

La méthode équivalente avec `sysctl` est :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Sortie possible :

```text
net.ipv4.ip_forward = 1
```

Pour `swappiness` :

```bash
sudo sysctl -w vm.swappiness=10
```

Cela modifie la valeur active.


### 8.5.3. Pourquoi préférer `sysctl` ?

Nous pouvons modifier directement `/proc/sys`, mais `sysctl` présente plusieurs avantages :

- syntaxe standard ;
    
- affichage clair ;
    
- meilleure intégration avec les fichiers de configuration ;
    
- habitudes administrateur ;
    
- moins de risque d’écrire dans le mauvais fichier ;
    
- possibilité de charger une configuration complète.
    

Nous utilisons donc souvent `sysctl` en administration système.


## 8.6. Rendre une configuration persistante

### 8.6.1. Le problème des modifications temporaires

Si nous faisons :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

la modification est active immédiatement.

Mais après redémarrage, elle peut disparaître.

Pour rendre une configuration persistante, nous devons l’écrire dans un fichier de configuration.


### 8.6.2. `/etc/sysctl.conf`

Historiquement, nous pouvons utiliser :

```bash
/etc/sysctl.conf
```

Exemple :

```conf
net.ipv4.ip_forward = 1
vm.swappiness = 10
```

Puis nous appliquons :

```bash
sudo sysctl -p
```

Cette commande charge par défaut `/etc/sysctl.conf`.


### 8.6.3. `/etc/sysctl.d/*.conf`

Sur les distributions modernes, nous utilisons souvent :

```bash
/etc/sysctl.d/
```

Exemple :

```bash
sudo nano /etc/sysctl.d/99-custom.conf
```

Contenu :

```conf
net.ipv4.ip_forward = 1
vm.swappiness = 10
```

Puis :

```bash
sudo sysctl --system
```

Cette commande charge les configurations depuis plusieurs emplacements, dont `/etc/sysctl.d`.

Nous préférons souvent créer un fichier dédié plutôt que modifier directement `/etc/sysctl.conf`, car c’est plus propre et plus maintenable.


### 8.6.4. Vérifier la valeur réellement appliquée

Après configuration, nous vérifions toujours :

```bash
sysctl net.ipv4.ip_forward
sysctl vm.swappiness
```

ou :

```bash
cat /proc/sys/net/ipv4/ip_forward
cat /proc/sys/vm/swappiness
```

Nous ne supposons pas que la configuration est appliquée : nous la vérifions.


## 8.7. Domaine `kernel`

### 8.7.1. Présentation

Le répertoire :

```bash
/proc/sys/kernel
```

contient des paramètres généraux du noyau.

Nous pouvons lister :

```bash
ls /proc/sys/kernel | head
```

Exemples de paramètres :

```text
hostname
domainname
pid_max
threads-max
panic
randomize_va_space
dmesg_restrict
kptr_restrict
```


### 8.7.2. `kernel.hostname`

Nous lisons :

```bash
cat /proc/sys/kernel/hostname
```

ou :

```bash
sysctl kernel.hostname
```

Nous pouvons temporairement modifier le hostname actif :

```bash
echo "machine-test" | sudo tee /proc/sys/kernel/hostname
```

ou :

```bash
sudo sysctl -w kernel.hostname=machine-test
```

En pratique, pour modifier durablement le nom d’hôte, nous utilisons plutôt les outils de la distribution, par exemple :

```bash
sudo hostnamectl set-hostname machine-test
```


### 8.7.3. `kernel.pid_max`

Nous lisons :

```bash
cat /proc/sys/kernel/pid_max
```

Ce paramètre indique la valeur maximale des PID attribuables.

Exemple :

```text
4194304
```

Le noyau attribue des PID jusqu’à cette limite, puis réutilise des valeurs disponibles.

Nous ne modifions pas ce paramètre sans raison précise.


### 8.7.4. `kernel.threads-max`

Nous lisons :

```bash
cat /proc/sys/kernel/threads-max
```

Ce paramètre limite le nombre total de threads que le système peut créer.

Il est lié à la capacité mémoire et aux contraintes du noyau.

Nous faisons le lien avec :

```bash
cat /proc/<PID>/status | grep Threads
```

et avec les limites de processus utilisateur.


### 8.7.5. `kernel.randomize_va_space`

Nous lisons :

```bash
cat /proc/sys/kernel/randomize_va_space
```

Ce paramètre concerne l’ASLR, Address Space Layout Randomization.

Valeurs courantes :

|Valeur|Signification approximative|
|---|---|
|`0`|désactivation|
|`1`|randomisation partielle|
|`2`|randomisation plus complète|

L’ASLR est une mesure de sécurité qui rend plus difficile l’exploitation de certaines vulnérabilités mémoire.

Nous ne désactivons pas ce paramètre en production.


### 8.7.6. `kernel.dmesg_restrict` et `kernel.kptr_restrict`

Ces paramètres concernent l’exposition d’informations sensibles.

```bash
cat /proc/sys/kernel/dmesg_restrict
cat /proc/sys/kernel/kptr_restrict
```

Ils peuvent limiter :

- l’accès aux logs noyau ;
    
- l’exposition d’adresses noyau ;
    
- certaines informations utiles à un attaquant.
    

Nous les étudions aussi dans le chapitre sécurité.


## 8.8. Domaine `vm`

### 8.8.1. Présentation

Le répertoire :

```bash
/proc/sys/vm
```

contient des paramètres liés à la mémoire virtuelle.

Nous pouvons lister :

```bash
ls /proc/sys/vm | head
```

Exemples :

```text
swappiness
dirty_ratio
dirty_background_ratio
overcommit_memory
overcommit_ratio
drop_caches
max_map_count
```

Ces paramètres peuvent avoir un impact important sur les performances.


### 8.8.2. `vm.swappiness`

Nous avons déjà vu :

```bash
cat /proc/sys/vm/swappiness
```

La valeur indique la tendance du noyau à déplacer des pages mémoire vers le swap.

Exemples de modification temporaire :

```bash
sudo sysctl -w vm.swappiness=10
```

ou :

```bash
echo 10 | sudo tee /proc/sys/vm/swappiness
```

Nous devons éviter les recettes universelles du type “mettre toujours 1” ou “mettre toujours 10”.

La bonne valeur dépend :

- du type de charge ;
    
- de la quantité de RAM ;
    
- de la présence de swap ;
    
- du type de stockage ;
    
- des exigences de latence ;
    
- du comportement applicatif.
    


### 8.8.3. `vm.dirty_ratio` et `vm.dirty_background_ratio`

Ces paramètres contrôlent le comportement des pages mémoire modifiées non encore écrites sur disque.

Nous lisons :

```bash
cat /proc/sys/vm/dirty_ratio
cat /proc/sys/vm/dirty_background_ratio
```

Idée générale :

- `dirty_background_ratio` : seuil à partir duquel le noyau commence à écrire en arrière-plan ;
    
- `dirty_ratio` : seuil plus élevé à partir duquel les processus peuvent être ralentis pour forcer l’écriture.
    

Ces paramètres peuvent influencer les performances d’écriture disque.

Nous les modifions avec prudence, surtout sur des serveurs de bases de données ou de stockage.


### 8.8.4. `vm.overcommit_memory`

Nous lisons :

```bash
cat /proc/sys/vm/overcommit_memory
```

Ce paramètre contrôle la stratégie d’allocation mémoire virtuelle.

Valeurs courantes :

|Valeur|Comportement|
|---|---|
|`0`|heuristique noyau|
|`1`|overcommit toujours autorisé|
|`2`|overcommit strict selon une limite calculée|

Ce paramètre est important pour certaines applications sensibles à l’allocation mémoire, notamment bases de données, calcul scientifique ou systèmes embarqués.


### 8.8.5. `vm.max_map_count`

Nous lisons :

```bash
cat /proc/sys/vm/max_map_count
```

Ce paramètre limite le nombre de mappings mémoire qu’un processus peut avoir.

Il est connu dans le monde Elasticsearch, OpenSearch et certaines bases de données, car ces systèmes peuvent nécessiter une valeur élevée.

Modification temporaire :

```bash
sudo sysctl -w vm.max_map_count=262144
```

Configuration persistante :

```conf
vm.max_map_count = 262144
```

dans un fichier de `/etc/sysctl.d/`.


### 8.8.6. `vm.drop_caches`

Le fichier :

```bash
/proc/sys/vm/drop_caches
```

permet de demander au noyau de libérer certains caches.

Exemple souvent vu :

```bash
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

Nous devons être très prudents avec ce paramètre.

Il ne sert pas à “réparer” la mémoire dans un usage normal. Linux utilise le cache pour améliorer les performances. Vider les caches peut au contraire ralentir le système.

Nous l’utilisons uniquement dans des contextes de test ou de diagnostic précis.


## 8.9. Domaine `net`

### 8.9.1. Présentation

Le répertoire :

```bash
/proc/sys/net
```

contient les paramètres de la pile réseau.

Nous pouvons explorer :

```bash
ls /proc/sys/net
```

Sortie possible :

```text
core
ipv4
ipv6
netfilter
unix
```

Les paramètres réseau sont nombreux et sensibles.


### 8.9.2. `net.ipv4.ip_forward`

Nous lisons :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Modification temporaire :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Configuration persistante :

```conf
net.ipv4.ip_forward = 1
```

Ce paramètre est nécessaire si la machine doit router des paquets IPv4 entre interfaces.

Exemples :

- passerelle Linux ;
    
- routeur ;
    
- serveur VPN ;
    
- NAT ;
    
- certaines configurations Docker ou Kubernetes.
    


### 8.9.3. Paramètres ICMP

Exemple :

```bash
cat /proc/sys/net/ipv4/icmp_echo_ignore_all
```

Si nous mettons :

```bash
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1
```

la machine ignore les requêtes ping ICMP echo.

Nous ne le faisons pas sans raison, car `ping` est utile pour le diagnostic.


### 8.9.4. Paramètres TCP

Nous trouvons de nombreux paramètres TCP :

```bash
ls /proc/sys/net/ipv4/tcp_*
```

Exemples :

```text
tcp_fin_timeout
tcp_keepalive_time
tcp_syncookies
tcp_max_syn_backlog
```

Ces paramètres peuvent influencer :

- la résistance à certaines attaques ;
    
- la gestion des connexions ;
    
- les timeouts ;
    
- les performances réseau ;
    
- le comportement serveur sous forte charge.
    

Nous ne modifions pas ces paramètres avec des recettes copiées sans analyse.


### 8.9.5. `net.ipv4.tcp_syncookies`

Nous lisons :

```bash
cat /proc/sys/net/ipv4/tcp_syncookies
```

Les SYN cookies sont une protection contre certaines attaques SYN flood.

Valeurs courantes :

```text
0
1
```

Nous gardons généralement cette protection activée sauf cas très particulier.


### 8.9.6. IPv6

Les paramètres IPv6 se trouvent dans :

```bash
/proc/sys/net/ipv6
```

Exemple :

```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```

Nous pouvons aussi voir des paramètres par interface :

```bash
ls /proc/sys/net/ipv6/conf
```

Sortie possible :

```text
all
default
eth0
lo
```

Cela signifie que certaines options peuvent être globales ou spécifiques à une interface.


## 8.10. Domaine `fs`

### 8.10.1. Présentation

Le répertoire :

```bash
/proc/sys/fs
```

contient des paramètres liés aux fichiers, aux inodes, aux descripteurs, aux montages et à certains comportements de systèmes de fichiers.

Nous pouvons lister :

```bash
ls /proc/sys/fs | head
```

Exemples :

```text
file-max
file-nr
inode-nr
inotify
protected_hardlinks
protected_symlinks
```


### 8.10.2. `fs.file-max`

Nous lisons :

```bash
cat /proc/sys/fs/file-max
```

Ce paramètre indique la limite globale du nombre de fichiers ouverts au niveau du système.

Nous le distinguons de :

```bash
cat /proc/<PID>/limits
```

qui donne les limites d’un processus particulier.


### 8.10.3. `fs.file-nr`

Nous lisons :

```bash
cat /proc/sys/fs/file-nr
```

Sortie possible :

```text
12032   0   9223372036854775807
```

Ce fichier donne des informations sur les handles de fichiers au niveau système.

Nous l’utilisons pour détecter une pression globale sur les fichiers ouverts.


### 8.10.4. Inotify

Les paramètres inotify sont dans :

```bash
/proc/sys/fs/inotify
```

Exemples :

```bash
cat /proc/sys/fs/inotify/max_user_watches
cat /proc/sys/fs/inotify/max_user_instances
cat /proc/sys/fs/inotify/max_queued_events
```

Ces paramètres sont importants pour :

- IDE ;
    
- outils de développement ;
    
- watchers JavaScript ;
    
- Docker bind mounts ;
    
- synchronisation de fichiers ;
    
- outils comme `webpack`, `vite`, `watchexec`.
    

Si un outil dit :

```text
ENOSPC: System limit for number of file watchers reached
```

nous devons souvent regarder :

```bash
cat /proc/sys/fs/inotify/max_user_watches
```

Modification temporaire :

```bash
sudo sysctl -w fs.inotify.max_user_watches=524288
```

Configuration persistante :

```conf
fs.inotify.max_user_watches = 524288
```


### 8.10.5. `protected_symlinks` et `protected_hardlinks`

Nous lisons :

```bash
cat /proc/sys/fs/protected_symlinks
cat /proc/sys/fs/protected_hardlinks
```

Ces paramètres renforcent la sécurité contre certaines attaques utilisant des liens symboliques ou hard links, notamment dans des répertoires partagés comme `/tmp`.

Nous les laissons généralement activés.


## 8.11. Domaine `user`

### 8.11.1. Présentation

Le répertoire :

```bash
/proc/sys/user
```

contient des limites liées aux namespaces et ressources utilisateur.

Nous pouvons lister :

```bash
ls /proc/sys/user
```

Exemples possibles :

```text
max_user_namespaces
max_pid_namespaces
max_net_namespaces
max_mnt_namespaces
```

Ces paramètres sont importants pour les conteneurs et les mécanismes d’isolation.


### 8.11.2. `user.max_user_namespaces`

Nous lisons :

```bash
cat /proc/sys/user/max_user_namespaces
```

Ce paramètre limite le nombre de namespaces utilisateur.

Il peut influencer :

- Docker rootless ;
    
- Podman ;
    
- certains sandboxes ;
    
- outils d’isolation ;
    
- navigateurs ;
    
- environnements de build.
    

Une valeur trop basse peut empêcher certains outils de fonctionner.

Une valeur trop permissive peut augmenter la surface d’attaque si le système n’est pas correctement durci.


## 8.12. Domaine `debug`

### 8.12.1. Présentation

Le répertoire :

```bash
/proc/sys/debug
```

expose certains paramètres de debug selon le noyau et la configuration.

Tous les systèmes n’ont pas les mêmes entrées.

Nous l’explorons avec :

```bash
ls /proc/sys/debug
```

Nous ne modifions généralement pas ces paramètres dans un cours introductif sans objectif précis.


## 8.13. Permissions et erreurs fréquentes

### 8.13.1. Permission refusée

Si nous essayons de modifier un paramètre sans privilèges :

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

nous pouvons obtenir :

```text
Permission denied
```

Même avec `sudo`, cette forme peut échouer :

```bash
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

Pourquoi ? Parce que la redirection `>` est effectuée par le shell courant, avant ou en dehors de l’effet attendu de `sudo`.


### 8.13.2. Utiliser `tee`

La bonne forme est :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

Ici, `tee` s’exécute avec les droits administrateur et écrit dans le fichier.

Nous pouvons masquer l’affichage si nécessaire :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward > /dev/null
```


### 8.13.3. Utiliser `sysctl -w`

La forme recommandée est souvent :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Elle évite le piège de la redirection shell.


### 8.13.4. Fichier en lecture seule

Certains paramètres ne sont pas modifiables.

Nous pouvons voir des permissions en lecture seule :

```bash
ls -l /proc/sys/kernel/ostype
```

Si nous essayons d’écrire dedans, le noyau refuse.

Ce n’est pas une erreur du système : tous les paramètres exposés ne sont pas configurables.


## 8.14. Cas pratique 1 : activer temporairement le routage IPv4

### 8.14.1. Situation

Nous voulons transformer temporairement une machine Linux en routeur IPv4 entre deux interfaces.

Nous vérifions d’abord :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Si la valeur est :

```text
0
```

le forwarding est désactivé.


### 8.14.2. Activation temporaire

Nous activons :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

ou :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

Nous vérifions :

```bash
cat /proc/sys/net/ipv4/ip_forward
```


### 8.14.3. Persistance

Nous créons :

```bash
sudo nano /etc/sysctl.d/99-routing.conf
```

Contenu :

```conf
net.ipv4.ip_forward = 1
```

Puis :

```bash
sudo sysctl --system
```

Nous vérifions :

```bash
sysctl net.ipv4.ip_forward
```


### 8.14.4. Attention : forwarding ne suffit pas

Activer `ip_forward` ne suffit pas toujours.

Pour faire réellement du routage ou du NAT, nous devons aussi configurer :

- les routes ;
    
- le pare-feu ;
    
- éventuellement NAT avec `nftables` ou `iptables` ;
    
- les règles de filtrage ;
    
- les interfaces réseau.
    

Nous retenons donc :

```text
ip_forward autorise le noyau à router.
Il ne configure pas automatiquement toute la politique réseau.
```


## 8.15. Cas pratique 2 : corriger une limite inotify

### 8.15.1. Situation

Nous lançons un outil de développement, par exemple un serveur Vite, Webpack, un IDE ou un watcher, et nous obtenons une erreur :

```text
ENOSPC: System limit for number of file watchers reached
```

Nous suspectons une limite inotify.


### 8.15.2. Lire la valeur actuelle

Nous lisons :

```bash
cat /proc/sys/fs/inotify/max_user_watches
```

ou :

```bash
sysctl fs.inotify.max_user_watches
```

Exemple :

```text
fs.inotify.max_user_watches = 8192
```

Cette valeur peut être trop basse pour de gros projets.


### 8.15.3. Modification temporaire

Nous pouvons tester :

```bash
sudo sysctl -w fs.inotify.max_user_watches=524288
```

Nous relançons ensuite l’outil concerné.


### 8.15.4. Configuration persistante

Nous créons :

```bash
sudo nano /etc/sysctl.d/99-inotify.conf
```

Contenu :

```conf
fs.inotify.max_user_watches = 524288
```

Puis :

```bash
sudo sysctl --system
```

Nous vérifions :

```bash
sysctl fs.inotify.max_user_watches
```


## 8.16. Cas pratique 3 : observer sans casser la mémoire

### 8.16.1. Mauvaise pratique fréquente

Nous voyons parfois cette commande proposée :

```bash
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

Elle force le noyau à libérer certains caches.

Le problème est que beaucoup d’utilisateurs interprètent mal le cache Linux.

Ils voient une mémoire “utilisée” élevée et pensent qu’il faut “nettoyer” la RAM.

En réalité, le cache est utile : il accélère les accès disque et peut être libéré si une application a besoin de mémoire.


### 8.16.2. Bonne lecture

Nous regardons plutôt :

```bash
free -h
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached):' /proc/meminfo
```

Nous privilégions `MemAvailable` pour comprendre la mémoire réellement disponible.

Nous ne vidons pas les caches pour “améliorer” le système.


### 8.16.3. Quand utiliser `drop_caches` ?

Nous pouvons l’utiliser dans des contextes de benchmark ou de test contrôlé, par exemple pour mesurer des lectures disque sans effet du cache.

Mais nous ne l’utilisons pas comme routine d’administration.


## 8.17. Bonnes pratiques

### 8.17.1. Lire avant de modifier

Avant toute modification, nous lisons la valeur actuelle :

```bash
sysctl vm.swappiness
```

ou :

```bash
cat /proc/sys/vm/swappiness
```

Nous gardons une trace de la valeur initiale.


### 8.17.2. Modifier temporairement avant de rendre persistant

Nous testons d’abord temporairement :

```bash
sudo sysctl -w parametre=valeur
```

Puis nous observons les effets.

Si le comportement est correct, nous rendons la configuration persistante dans `/etc/sysctl.d`.


### 8.17.3. Documenter la raison

Dans un fichier de configuration, nous ajoutons un commentaire.

Exemple :

```conf
## Augmente la limite inotify pour les projets de développement avec beaucoup de fichiers.
fs.inotify.max_user_watches = 524288
```

Un futur administrateur doit comprendre pourquoi la valeur a été changée.


### 8.17.4. Éviter les recettes universelles

Nous évitons :

```text
Mets swappiness à 1, c’est toujours mieux.
Désactive IPv6, ça règle tout.
Vide les caches, Linux ira plus vite.
Augmente tous les buffers TCP au maximum.
```

Ces recettes peuvent dégrader le système.

Nous modifions un paramètre parce que nous avons :

- un problème identifié ;
    
- une hypothèse ;
    
- une mesure avant ;
    
- un test ;
    
- une mesure après ;
    
- une documentation.
    


## 8.18. Exercices

## Exercice 1 — Explorer `/proc/sys`

Nous exécutons :

```bash
ls /proc/sys
find /proc/sys -maxdepth 2 -type f | head -30
```

Nous répondons :

1. Quels grands domaines voyons-nous ?
    
2. Quels paramètres semblent liés au noyau ?
    
3. Quels paramètres semblent liés à la mémoire ?
    
4. Quels paramètres semblent liés au réseau ?
    


## Exercice 2 — Correspondance `/proc/sys` et `sysctl`

Nous comparons :

```bash
cat /proc/sys/vm/swappiness
sysctl vm.swappiness
```

Puis :

```bash
cat /proc/sys/net/ipv4/ip_forward
sysctl net.ipv4.ip_forward
```

Nous répondons :

1. Quelle correspondance voyons-nous entre chemin et nom `sysctl` ?
    
2. Pourquoi `sysctl` est-il plus pratique ?
    
3. Les valeurs sont-elles identiques ?
    


## Exercice 3 — Lire des paramètres système

Nous exécutons :

```bash
sysctl kernel.hostname
sysctl kernel.pid_max
sysctl kernel.threads-max
sysctl vm.swappiness
sysctl fs.file-max
```

Nous répondons :

1. Quel est le hostname actif ?
    
2. Quelle est la valeur maximale des PID ?
    
3. Quelle est la limite globale de threads ?
    
4. Quelle est la valeur de swappiness ?
    
5. Quelle est la limite globale de fichiers ouverts ?
    


## Exercice 4 — Modifier temporairement un paramètre non critique

Nous choisissons un paramètre raisonnable à modifier en environnement de test, par exemple `vm.swappiness`.

Nous lisons :

```bash
sysctl vm.swappiness
```

Nous modifions temporairement :

```bash
sudo sysctl -w vm.swappiness=10
```

Nous vérifions :

```bash
cat /proc/sys/vm/swappiness
```

Puis nous remettons la valeur initiale.

Nous répondons :

1. La modification est-elle immédiate ?
    
2. Est-elle persistante ?
    
3. Comment la rendre persistante ?
    
4. Pourquoi devons-nous noter la valeur initiale ?
    


## Exercice 5 — Comprendre l’erreur avec `sudo echo`

Nous testons en environnement adapté :

```bash
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

Puis :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

Nous répondons :

1. Pourquoi la première commande peut-elle échouer ?
    
2. Pourquoi la seconde fonctionne-t-elle ?
    
3. En quoi `sysctl -w` évite-t-il ce piège ?
    


## Exercice 6 — Inotify

Nous lisons :

```bash
sysctl fs.inotify.max_user_watches
sysctl fs.inotify.max_user_instances
```

Nous répondons :

1. À quoi sert inotify ?
    
2. Quels outils peuvent consommer beaucoup de watchers ?
    
3. Quelle erreur apparaît souvent quand la limite est trop basse ?
    
4. Comment tester une nouvelle valeur temporairement ?
    
5. Comment la rendre persistante proprement ?
    


## 8.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc/sys` expose des paramètres actifs du noyau.
    
2. Certains paramètres sont seulement lisibles, d’autres sont modifiables.
    
3. Écrire dans `/proc/sys` modifie le comportement actif du noyau.
    
4. Les modifications directes sont généralement temporaires.
    
5. `sysctl` fournit une interface pratique pour lire et modifier ces paramètres.
    
6. La correspondance est simple : `/proc/sys/net/ipv4/ip_forward` devient `net.ipv4.ip_forward`.
    
7. Les configurations persistantes se placent dans `/etc/sysctl.conf` ou, de préférence, dans `/etc/sysctl.d/*.conf`.
    
8. Nous devons toujours vérifier la valeur réellement appliquée.
    
9. Les domaines principaux sont `kernel`, `vm`, `net`, `fs`, `user`, `debug`.
    
10. Les paramètres réseau et mémoire peuvent avoir des effets importants sur les performances et la sécurité.
    
11. Nous ne modifions pas un paramètre noyau par recette universelle.
    
12. Nous documentons les changements persistants.
    
13. Nous testons temporairement avant de rendre une modification permanente.
    
14. Nous sommes particulièrement prudents avec `drop_caches`, `overcommit_memory`, les paramètres TCP et les paramètres de sécurité.
    


## Conclusion du chapitre 8

Nous savons maintenant utiliser `/proc/sys` pour lire et modifier certains paramètres actifs du noyau Linux.

Cette partie de `/proc` est différente des fichiers purement informatifs : elle constitue une interface de configuration. Elle est donc très puissante, mais aussi potentiellement dangereuse.

Nous avons appris à lire les paramètres directement dans `/proc/sys`, à utiliser `sysctl`, à faire la correspondance entre chemins et noms de paramètres, à appliquer une modification temporaire, puis à rendre une configuration persistante avec `/etc/sysctl.d`.

Nous retenons surtout une règle professionnelle : nous ne modifions un paramètre noyau qu’après avoir compris le problème, mesuré l’état initial, testé prudemment, vérifié l’effet et documenté la décision.

Dans le chapitre suivant, nous étudions les aspects sécurité, permissions et isolation de `/proc`, notamment les informations sensibles, l’option `hidepid` et le comportement dans les conteneurs.

# Chapitre 9 — Sécurité, permissions et isolation

## Objectifs du chapitre

Dans ce chapitre, nous étudions `/proc` sous l’angle de la sécurité.

Jusqu’ici, nous avons utilisé `/proc` comme outil d’observation et de diagnostic. Nous avons lu des informations sur le système, les processus, la mémoire, les fichiers ouverts, le réseau et les paramètres noyau.

Mais cette richesse a une contrepartie : `/proc` expose beaucoup d’informations. Certaines sont anodines, d’autres peuvent être sensibles.

À la fin de ce chapitre, nous savons :

- identifier les informations sensibles exposées par `/proc` ;
    
- comprendre les permissions appliquées aux fichiers de `/proc` ;
    
- expliquer les risques liés à `cmdline`, `environ`, `fd`, `maps` et `mem` ;
    
- comprendre l’option de montage `hidepid` ;
    
- analyser les risques sur un serveur multi-utilisateurs ;
    
- comprendre le comportement de `/proc` dans les conteneurs ;
    
- faire le lien avec les namespaces Linux ;
    
- appliquer de bonnes pratiques de durcissement.
    


## 9.1. Pourquoi `/proc` pose des questions de sécurité

### 9.1.1. Une interface très informative

`/proc` est précieux parce qu’il rend visibles de nombreuses informations internes du système.

Nous pouvons y trouver :

```text
processus actifs
lignes de commande
variables d’environnement
fichiers ouverts
sockets réseau
mappings mémoire
paramètres noyau
statistiques système
informations CPU
informations mémoire
```

Cette visibilité est très utile pour l’administration système.

Mais du point de vue sécurité, une information utile à l’administrateur peut aussi être utile à un attaquant.


### 9.1.2. Le problème de l’énumération

Sur un système mal protégé, un utilisateur local peut parfois observer :

```bash
ps aux
ls /proc
cat /proc/<PID>/cmdline
tr '\0' '\n' < /proc/<PID>/environ
ls -l /proc/<PID>/fd
```

Cela peut lui permettre de découvrir :

- quels services tournent ;
    
- quels utilisateurs sont connectés ;
    
- quelles applications sont lancées ;
    
- quels fichiers sont ouverts ;
    
- quels ports ou sockets sont utilisés ;
    
- quels chemins applicatifs sont présents ;
    
- parfois des secrets mal protégés.
    

Nous comprenons donc que `/proc` peut faciliter la reconnaissance locale.


## 9.2. Informations sensibles exposées par `/proc`

### 9.2.1. La ligne de commande : `cmdline`

Le fichier :

```bash
/proc/<PID>/cmdline
```

expose la ligne de commande utilisée pour lancer un processus.

Nous l’affichons de manière lisible avec :

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
echo
```

Le problème apparaît lorsque des secrets sont passés en arguments.

Exemples de mauvaises pratiques :

```bash
mysql -u admin -pMonMotDePasse
curl -H "Authorization: Bearer TOKEN_SECRET" https://api.example.org
python app.py --password secret
backup --s3-secret-key ABCDEF
```

Ces informations peuvent être visibles dans `/proc/<PID>/cmdline`, mais aussi dans des outils comme `ps`.

Nous retenons :

```text
Nous ne passons pas de secrets en arguments de ligne de commande.
```


### 9.2.2. Les variables d’environnement : `environ`

Le fichier :

```bash
/proc/<PID>/environ
```

contient les variables d’environnement du processus.

Nous les lisons avec :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

Les variables d’environnement peuvent contenir :

```text
DATABASE_URL
POSTGRES_PASSWORD
MYSQL_PASSWORD
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
GITHUB_TOKEN
JWT_SECRET
API_KEY
```

Elles sont pratiques, mais elles ne sont pas automatiquement secrètes.

Si les permissions permettent leur lecture, un autre utilisateur ou un processus compromis peut récupérer ces valeurs.

Nous retenons :

```text
Les variables d’environnement sont une convention pratique de configuration.
Elles ne remplacent pas un vrai mécanisme de gestion de secrets.
```


### 9.2.3. Les fichiers ouverts : `fd`

Le dossier :

```bash
/proc/<PID>/fd
```

expose les descripteurs ouverts.

Il peut révéler :

- fichiers de configuration ;
    
- fichiers temporaires ;
    
- logs ;
    
- sockets ;
    
- bases SQLite ;
    
- fichiers supprimés mais encore ouverts ;
    
- répertoires de travail ;
    
- chemins internes d’une application.
    

Exemple :

```bash
ls -l /proc/<PID>/fd
```

Sortie possible :

```text
0 -> /dev/null
1 -> /var/log/app/access.log
2 -> /var/log/app/error.log
3 -> /etc/app/secret.conf
4 -> socket:[123456]
5 -> /tmp/session-data
```

Cela peut donner beaucoup d’informations sur l’architecture d’un service.


### 9.2.4. La cartographie mémoire : `maps`

Le fichier :

```bash
/proc/<PID>/maps
```

expose les mappings mémoire du processus.

Il peut révéler :

- bibliothèques chargées ;
    
- chemins exacts des binaires ;
    
- versions de bibliothèques ;
    
- présence de modules natifs ;
    
- fichiers mappés ;
    
- exécutable supprimé mais encore chargé ;
    
- structure générale de l’espace mémoire.
    

Exemple :

```bash
cat /proc/<PID>/maps | head
```

Du point de vue d’un attaquant, ces informations peuvent aider à comprendre l’environnement d’exécution.

Les protections modernes, comme l’ASLR, limitent l’exploitation directe, mais l’exposition d’informations mémoire reste sensible.


### 9.2.5. La mémoire brute : `mem`

Le fichier :

```bash
/proc/<PID>/mem
```

représente la mémoire virtuelle du processus.

Son accès est fortement contrôlé.

Lire ou modifier ce fichier peut permettre :

- d’extraire des secrets en mémoire ;
    
- d’observer des données applicatives ;
    
- de modifier le comportement d’un processus ;
    
- de contourner certaines protections si les permissions sont insuffisantes.
    

Nous ne l’utilisons pas dans un usage courant.

Nous retenons :

```text
/proc/<PID>/mem est extrêmement sensible.
Son accès doit être strictement contrôlé.
```


### 9.2.6. Les liens `cwd`, `exe` et `root`

Les liens suivants peuvent aussi exposer des informations :

```bash
/proc/<PID>/cwd
/proc/<PID>/exe
/proc/<PID>/root
```

Ils permettent de savoir :

- d’où le processus travaille ;
    
- quel binaire il exécute ;
    
- quelle racine de système de fichiers il voit ;
    
- s’il tourne dans un conteneur ou un chroot ;
    
- s’il utilise un exécutable supprimé.
    

Exemple :

```bash
ls -l /proc/<PID>/exe
```

Sortie possible :

```text
/proc/1234/exe -> /opt/app/releases/2026-05-01/app-server
```

Ce type d’information peut révéler la stratégie de déploiement ou les chemins internes.


## 9.3. Permissions dans `/proc`

### 9.3.1. Observer les permissions

Nous pouvons afficher les permissions d’un processus :

```bash
ls -ld /proc/<PID>
ls -l /proc/<PID> | head
```

Certaines entrées sont lisibles par tous, d’autres uniquement par le propriétaire du processus ou par `root`.

Exemple :

```bash
ls -l /proc/$$/environ
```

Nous observons les droits Unix classiques :

```text
-r-------- 1 user user 0 mai 19 10:00 environ
```

Le fichier semble avoir une taille `0`, mais il peut produire du contenu lors de la lecture.


### 9.3.2. Le rôle de l’UID

Les règles d’accès dépendent notamment :

- de l’utilisateur réel du processus ;
    
- de l’utilisateur qui lit ;
    
- des permissions du fichier ;
    
- des capacités Linux ;
    
- des options de montage de `/proc` ;
    
- de mécanismes de sécurité comme Yama, AppArmor ou SELinux.
    

Un utilisateur peut généralement lire plus facilement les informations de ses propres processus que celles des processus d’un autre utilisateur.


### 9.3.3. Tester avec deux utilisateurs

Sur une machine de test, nous pouvons créer deux utilisateurs :

```bash
sudo adduser alice
sudo adduser bob
```

Nous lançons un processus sous `alice` :

```bash
sudo -u alice sleep 1000 &
pid=$!
```

Puis nous essayons de lire depuis un autre utilisateur :

```bash
sudo -u bob cat /proc/$pid/status
sudo -u bob tr '\0' '\n' < /proc/$pid/environ
sudo -u bob ls -l /proc/$pid/fd
```

Nous observons ce qui est autorisé ou refusé.

Cet exercice montre que `/proc` applique des restrictions, mais que le niveau exact dépend de la configuration du système.


## 9.4. Le mécanisme `ptrace` et Yama

### 9.4.1. Pourquoi parler de `ptrace` ?

Sous Linux, certains accès à `/proc/<PID>` sont liés aux règles de traçage et de debugging.

Le mécanisme `ptrace` permet à un processus d’en observer ou contrôler un autre, par exemple pour un débogueur comme `gdb`.

Certaines restrictions de `/proc` suivent une logique proche : si un processus n’a pas le droit d’en tracer un autre, il n’a pas forcément le droit de lire certaines informations sensibles.


### 9.4.2. Le paramètre `ptrace_scope`

Sur certaines distributions, le module de sécurité Yama expose :

```bash
cat /proc/sys/kernel/yama/ptrace_scope
```

Valeurs courantes :

|Valeur|Effet général|
|--:|---|
|`0`|comportement classique plus permissif|
|`1`|restrictions supplémentaires, souvent parent-enfant|
|`2`|seul un processus avec privilège peut tracer|
|`3`|traçage désactivé de manière stricte jusqu’au redémarrage|

La valeur exacte disponible dépend de la configuration du noyau.


### 9.4.3. Effet sur le debugging

Si `ptrace_scope` est restrictif, un utilisateur peut rencontrer des difficultés à attacher `gdb` à un processus qui ne descend pas directement de son shell.

Exemple :

```bash
gdb -p <PID>
```

peut échouer avec une erreur de permission.

Nous comprenons donc que les protections autour de `/proc` et du debugging sont liées.


## 9.5. Option de montage `hidepid`

### 9.5.1. Le problème sur les serveurs multi-utilisateurs

Sur un serveur partagé, nous voulons parfois éviter qu’un utilisateur voie les processus des autres utilisateurs.

Sans restriction particulière, un utilisateur peut souvent lister les PID :

```bash
ls /proc | grep -E '^[0-9]+$'
```

et voir certaines informations via :

```bash
ps aux
```

Cela peut révéler :

- noms d’utilisateurs ;
    
- commandes lancées ;
    
- services personnels ;
    
- scripts ;
    
- chemins de travail ;
    
- arguments de ligne de commande.
    

L’option `hidepid` permet de réduire cette visibilité.


### 9.5.2. Les valeurs de `hidepid`

`hidepid` est une option de montage de `procfs`.

Les valeurs classiques sont :

|Valeur|Effet|
|--:|---|
|`hidepid=0`|comportement classique, visibilité large|
|`hidepid=1`|les utilisateurs voient les PID, mais pas les détails sensibles des processus des autres|
|`hidepid=2`|les processus des autres utilisateurs sont masqués|
|`hidepid=invisible`|forme équivalente ou moderne selon systèmes pour masquer davantage|
|`hidepid=ptraceable`|visibilité alignée sur les processus traçables, selon noyau|

Dans un cours général, nous retenons surtout `0`, `1` et `2`.


### 9.5.3. Vérifier le montage actuel

Nous vérifions les options de montage de `/proc` :

```bash
findmnt /proc
```

ou :

```bash
mount | grep ' /proc '
```

Sortie possible :

```text
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
```

Si `hidepid` est actif, nous pouvons voir :

```text
hidepid=2
```

dans les options.


### 9.5.4. Monter `/proc` avec `hidepid`

Sur une machine de test, nous pouvons remonter `/proc` avec :

```bash
sudo mount -o remount,hidepid=2 /proc
```

Puis nous testons avec un utilisateur non privilégié :

```bash
ls /proc
ps aux
```

Nous observons que les processus des autres utilisateurs deviennent moins visibles.

Pour revenir au comportement classique :

```bash
sudo mount -o remount,hidepid=0 /proc
```

Nous faisons cela uniquement dans un environnement de test ou avec une bonne compréhension des effets.


### 9.5.5. Configuration persistante

Une configuration persistante peut se faire via `/etc/fstab`.

Exemple indicatif :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

Selon les distributions, systemd ou d’autres mécanismes peuvent influencer le montage de `/proc`.

Nous devons donc tester et vérifier après redémarrage :

```bash
findmnt /proc
```


### 9.5.6. Groupe autorisé

Dans certains environnements, nous voulons masquer les processus aux utilisateurs ordinaires, mais garder une visibilité pour un groupe d’administrateurs.

Des options de montage peuvent permettre d’indiquer un groupe autorisé, par exemple avec `gid`.

Le principe est :

```text
les utilisateurs ordinaires voient moins d’informations ;
les membres d’un groupe dédié gardent une visibilité d’administration.
```

La configuration exacte dépend de la distribution et de la version du noyau.


## 9.6. `/proc` dans les conteneurs

### 9.6.1. Pourquoi les conteneurs changent la lecture de `/proc`

Un conteneur utilise des mécanismes d’isolation du noyau Linux, notamment les namespaces.

Un processus dans un conteneur peut voir :

- une liste de processus différente ;
    
- une pile réseau différente ;
    
- des montages différents ;
    
- une racine différente ;
    
- des limites cgroups différentes.
    

Cela signifie que `/proc` vu depuis un conteneur n’est pas forcément le même que `/proc` vu depuis l’hôte.


### 9.6.2. Namespace PID

Le namespace PID contrôle la vue des processus.

Dans un conteneur, le processus principal peut avoir le PID `1` dans le conteneur, mais un autre PID sur l’hôte.

Dans le conteneur :

```bash
cat /proc/1/comm
```

peut afficher :

```text
node
```

ou :

```text
python
```

ou :

```text
bash
```

Sur l’hôte, le même processus peut avoir le PID `24531`.

Nous retenons :

```text
Le PID dépend du namespace depuis lequel nous observons.
```


### 9.6.3. Namespace réseau

Nous avons vu au chapitre 7 que :

```bash
/proc/net
```

dépend du namespace réseau.

Dans un conteneur, nous pouvons voir :

```bash
cat /proc/net/dev
```

et obtenir seulement :

```text
lo
eth0
```

alors que l’hôte possède :

```text
lo
eno1
docker0
br-...
veth...
wg0
```

La vue réseau du conteneur est isolée.


### 9.6.4. Namespace de montage

Le namespace de montage contrôle les points de montage visibles.

Dans un conteneur :

```bash
cat /proc/mounts
```

peut montrer une arborescence différente de celle de l’hôte.

Le lien :

```bash
/proc/<PID>/root
```

est particulièrement intéressant.

Depuis l’hôte, si nous regardons un processus de conteneur :

```bash
sudo ls -l /proc/<PID>/root
```

nous pouvons accéder à la racine vue par ce processus.

C’est puissant pour le diagnostic, mais sensible du point de vue sécurité.


## 9.7. Risques des conteneurs privilégiés

### 9.7.1. Le danger d’un `/proc` trop permissif

Dans un conteneur, monter `/proc` de manière trop permissive peut exposer des informations de l’hôte ou permettre des actions dangereuses.

Un conteneur privilégié peut avoir des capacités étendues.

Si nous combinons :

```text
conteneur privilégié
montage sensible de /proc
accès à /sys
accès à /dev
capabilities élevées
```

nous réduisons fortement l’isolation.


### 9.7.2. Exemple de mauvaise pratique

Un montage dangereux pourrait ressembler à :

```bash
docker run --privileged -v /proc:/host/proc image
```

Le conteneur peut alors observer des informations de l’hôte via `/host/proc`.

Selon les autres droits accordés, cela peut devenir très risqué.

Nous évitons ce type de montage sauf besoin exceptionnel et environnement maîtrisé.


### 9.7.3. Paramètres noyau depuis un conteneur

Certains fichiers de `/proc/sys` peuvent être visibles dans un conteneur.

Mais le noyau est partagé entre l’hôte et les conteneurs.

Modifier un paramètre noyau depuis un conteneur peut donc affecter l’hôte ou d’autres conteneurs, selon le paramètre et le namespace.

C’est pourquoi les runtimes de conteneurs limitent généralement les accès en écriture.

Nous retenons :

```text
Un conteneur partage le noyau avec l’hôte.
Les paramètres noyau ne sont pas des paramètres privés applicatifs ordinaires.
```


## 9.8. `/proc/sys` et sécurité

### 9.8.1. Paramètres de durcissement

Certains paramètres de `/proc/sys` ont un rôle de sécurité.

Exemples :

```bash
cat /proc/sys/kernel/randomize_va_space
cat /proc/sys/kernel/dmesg_restrict
cat /proc/sys/kernel/kptr_restrict
cat /proc/sys/fs/protected_symlinks
cat /proc/sys/fs/protected_hardlinks
```

Ils influencent :

- la randomisation mémoire ;
    
- l’accès aux logs noyau ;
    
- l’exposition des pointeurs noyau ;
    
- la protection contre certaines attaques par liens symboliques ;
    
- la protection contre certains hard links abusifs.
    


### 9.8.2. ASLR : `randomize_va_space`

Nous lisons :

```bash
cat /proc/sys/kernel/randomize_va_space
```

Valeurs courantes :

```text
0 : désactivé
1 : randomisation partielle
2 : randomisation plus complète
```

En production, nous gardons généralement l’ASLR activé.

Désactiver l’ASLR facilite certains tests bas niveau, mais affaiblit la sécurité.


### 9.8.3. Restriction de `dmesg`

Nous lisons :

```bash
cat /proc/sys/kernel/dmesg_restrict
```

Si la valeur est `1`, l’accès non privilégié à `dmesg` est restreint.

Cela limite l’exposition d’informations noyau qui pourraient aider un attaquant.


### 9.8.4. Restriction des pointeurs noyau

Nous lisons :

```bash
cat /proc/sys/kernel/kptr_restrict
```

Ce paramètre limite l’exposition d’adresses noyau dans certaines interfaces.

C’est important car les adresses noyau peuvent faciliter des attaques exploitant des vulnérabilités bas niveau.


## 9.9. Secrets et bonnes pratiques applicatives

### 9.9.1. Ne pas mettre de secrets dans les arguments

Nous évitons :

```bash
app --password secret
app --token abcdef
curl -H "Authorization: Bearer secret" ...
```

Nous préférons :

- gestionnaire de secrets ;
    
- fichier de configuration avec permissions strictes ;
    
- agent local ;
    
- socket sécurisée ;
    
- injection contrôlée par l’orchestrateur ;
    
- variable d’environnement seulement si le risque est compris et accepté.
    


### 9.9.2. Variables d’environnement : pratique mais exposable

Les variables d’environnement sont très utilisées dans Docker, Kubernetes, systemd et les applications cloud-native.

Mais elles peuvent être visibles :

- dans `/proc/<PID>/environ` ;
    
- dans certains dumps ;
    
- dans des interfaces de debug ;
    
- dans des logs si l’application les affiche ;
    
- dans des erreurs de configuration.
    

Nous ne devons pas croire qu’une variable d’environnement est forcément protégée.


### 9.9.3. Fichiers de secrets

Un fichier de secret peut être préférable si :

- les permissions sont strictes ;
    
- le propriétaire est correct ;
    
- le fichier n’est pas logué ;
    
- le fichier n’est pas inclus dans une image Docker ;
    
- le fichier n’est pas commité dans Git ;
    
- le secret peut être monté temporairement par l’orchestrateur.
    

Exemple de permissions :

```bash
chmod 600 /etc/app/secret.conf
chown appuser:appuser /etc/app/secret.conf
```

Mais là encore, si le processus ouvre ce fichier, il peut apparaître dans :

```bash
/proc/<PID>/fd
```

Nous ne supprimons pas le risque, nous le réduisons et nous le contrôlons.


## 9.10. Serveurs multi-utilisateurs

### 9.10.1. Risques spécifiques

Sur un serveur utilisé par plusieurs personnes, `/proc` peut exposer des informations entre utilisateurs.

Exemples :

- un étudiant voit les processus d’un autre étudiant ;
    
- un utilisateur voit une commande contenant un token ;
    
- un utilisateur identifie un service vulnérable ;
    
- un utilisateur observe les chemins de travail d’un autre ;
    
- un utilisateur détecte les ports locaux ouverts.
    

Ces risques sont importants sur :

- serveurs universitaires ;
    
- machines de calcul partagées ;
    
- serveurs de formation ;
    
- bastions SSH ;
    
- plateformes mutualisées.
    


### 9.10.2. Mesures possibles

Nous pouvons renforcer :

```text
hidepid=2
Yama ptrace_scope
permissions strictes
pas de secrets en arguments
limites systemd
isolation par conteneur ou VM
journalisation contrôlée
groupes d’administration dédiés
```

Aucune mesure unique ne suffit.

Nous construisons une défense en profondeur.


## 9.11. Audit rapide de sécurité autour de `/proc`

### 9.11.1. Vérifier les options de montage

```bash
findmnt /proc
```

Nous cherchons notamment :

```text
hidepid
nosuid
nodev
noexec
```

Options courantes :

```text
rw,nosuid,nodev,noexec,relatime
```


### 9.11.2. Vérifier les paramètres sensibles

```bash
sysctl kernel.randomize_va_space
sysctl kernel.dmesg_restrict
sysctl kernel.kptr_restrict
sysctl fs.protected_symlinks
sysctl fs.protected_hardlinks
```

Nous cherchons des valeurs cohérentes avec un système durci.


### 9.11.3. Chercher des secrets en arguments

En environnement de test ou d’audit autorisé :

```bash
ps aux | grep -Ei 'password|passwd|token|secret|key'
```

Nous pouvons aussi inspecter `/proc/<PID>/cmdline`.

L’objectif est de détecter les mauvaises pratiques.


### 9.11.4. Chercher des environnements sensibles

Avec les droits nécessaires et dans un cadre autorisé :

```bash
sudo sh -c '
for env in /proc/[0-9]*/environ; do
    pid=$(echo "$env" | cut -d/ -f3)
    tr "\0" "\n" < "$env" 2>/dev/null |
    grep -Ei "password|passwd|token|secret|key" |
    sed "s/^/PID=$pid /"
done
'
```

Nous n’exécutons pas ce type de commande sans autorisation explicite, car elle peut exposer des secrets.


## 9.12. Cas pratique : durcir un serveur partagé

### 9.12.1. Situation

Nous administrons un serveur Linux utilisé par plusieurs développeurs.

Chaque utilisateur peut se connecter en SSH.

Nous voulons éviter qu’un utilisateur puisse observer trop facilement les processus et arguments des autres.


### 9.12.2. Diagnostic initial

Nous vérifions :

```bash
findmnt /proc
```

Puis, depuis un utilisateur non privilégié :

```bash
ps aux | head
ls /proc | grep -E '^[0-9]+$' | head
```

Nous observons la visibilité actuelle.


### 9.12.3. Mesure principale : `hidepid=2`

Nous testons temporairement :

```bash
sudo mount -o remount,hidepid=2 /proc
```

Puis nous vérifions avec un utilisateur non privilégié :

```bash
ps aux
ls /proc
```

Nous observons que les processus des autres utilisateurs sont masqués ou moins visibles.


### 9.12.4. Persistance

Nous configurons `/etc/fstab` avec prudence.

Exemple :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

Puis nous testons au redémarrage ou via un remount contrôlé.

Nous vérifions :

```bash
findmnt /proc
```


### 9.12.5. Compléments

Nous vérifions aussi :

```bash
sysctl kernel.yama.ptrace_scope
sysctl kernel.dmesg_restrict
sysctl kernel.kptr_restrict
```

Nous sensibilisons les utilisateurs :

```text
pas de mots de passe en ligne de commande ;
pas de tokens dans les scripts visibles ;
pas de secrets dans les logs ;
permissions strictes sur les fichiers de configuration.
```


## 9.13. Cas pratique : analyser un conteneur

### 9.13.1. Situation

Nous sommes dans un conteneur et nous voulons comprendre ce que `/proc` représente.

Nous exécutons :

```bash
cat /proc/1/comm
ls /proc | grep -E '^[0-9]+$' | wc -l
cat /proc/net/dev
cat /proc/self/mounts | head
cat /proc/1/cgroup
```

Nous observons :

- le PID 1 du conteneur ;
    
- le nombre de processus visibles ;
    
- les interfaces réseau vues ;
    
- les montages visibles ;
    
- les cgroups.
    


### 9.13.2. Comparaison avec l’hôte

Depuis l’hôte, nous comparons :

```bash
ps aux
ip addr
findmnt
```

Nous constatons que les vues diffèrent.

Nous retenons :

```text
/proc n’est pas une vérité absolue indépendante du contexte.
C’est une vue dépendante des namespaces et des montages.
```


### 9.13.3. Vérifier les montages sensibles

Dans un conteneur, nous inspectons :

```bash
mount | grep proc
findmnt /proc
```

Nous cherchons à savoir si `/proc` est monté normalement, en lecture seule, ou avec des restrictions spécifiques.

Selon le runtime, certaines parties de `/proc` peuvent être masquées ou rendues non modifiables.


## 9.14. Bonnes pratiques de sécurité

### 9.14.1. Pour les développeurs

Nous appliquons les règles suivantes :

```text
ne pas passer de secrets en arguments ;
éviter d’imprimer l’environnement dans les logs ;
ne pas stocker de secrets dans des fichiers lisibles par tous ;
limiter les permissions des fichiers de configuration ;
prévoir la rotation des secrets ;
utiliser un gestionnaire de secrets quand c’est possible.
```


### 9.14.2. Pour les administrateurs système

Nous vérifions :

```text
options de montage de /proc ;
paramètres sysctl de sécurité ;
permissions des services ;
limites systemd ;
séparation des utilisateurs ;
journalisation ;
durcissement SSH ;
conteneurs non privilégiés ;
absence de montages dangereux.
```


### 9.14.3. Pour les environnements conteneurisés

Nous évitons :

```text
--privileged sans nécessité ;
montage de /proc de l’hôte ;
montage de /sys de l’hôte ;
accès large à /dev ;
capabilities inutiles ;
secrets en variables d’environnement si l’exposition est inacceptable.
```

Nous préférons :

```text
capabilities minimales ;
root filesystem en lecture seule si possible ;
utilisateur non-root ;
seccomp ;
AppArmor ou SELinux ;
secrets gérés par l’orchestrateur ;
politiques réseau ;
limites cgroups.
```


## 9.15. Pièges et limites

### 9.15.1. Masquer `/proc` ne suffit pas

`hidepid` réduit la visibilité, mais ne remplace pas :

- une bonne gestion des permissions ;
    
- une séparation correcte des utilisateurs ;
    
- une politique de secrets ;
    
- un durcissement système ;
    
- une supervision ;
    
- des mises à jour de sécurité.
    

C’est une couche parmi d’autres.


### 9.15.2. Trop restreindre peut casser des outils

Certaines restrictions peuvent perturber :

- outils de monitoring ;
    
- agents de supervision ;
    
- outils de debug ;
    
- scripts d’administration ;
    
- services qui inspectent les processus.
    

Avant de durcir, nous devons tester.


### 9.15.3. Les conteneurs ne sont pas des machines virtuelles

Un conteneur partage le noyau avec l’hôte.

Même si `/proc` est isolé par namespaces, une mauvaise configuration peut affaiblir l’isolation.

Nous ne devons pas considérer un conteneur privilégié comme une barrière de sécurité forte.


### 9.15.4. Les secrets finissent souvent en mémoire

Même si nous évitons les arguments et limitons les variables d’environnement, un secret utilisé par une application finit souvent en mémoire.

Nous devons donc contrôler :

- qui peut déboguer le processus ;
    
- qui peut lire ses dumps mémoire ;
    
- qui peut accéder à `/proc/<PID>/mem` ;
    
- qui peut lancer des outils de diagnostic ;
    
- comment les core dumps sont gérés.
    


## 9.16. Exercices

## Exercice 1 — Observer les permissions

Nous exécutons :

```bash
ls -ld /proc/$$
ls -l /proc/$$/cmdline
ls -l /proc/$$/environ
ls -l /proc/$$/fd
ls -l /proc/$$/maps
```

Nous répondons :

1. Quels fichiers sont lisibles par l’utilisateur courant ?
    
2. Quels fichiers sont sensibles ?
    
3. Pourquoi `environ` est-il plus sensible que `status` ?
    
4. Pourquoi `fd` peut-il révéler des informations privées ?
    


## Exercice 2 — Observer les arguments d’un processus

Nous lançons volontairement un processus de test :

```bash
sleep 300 --test-secret-example 2>/dev/null &
pid=$!
```

Selon l’implémentation de `sleep`, cette commande peut échouer si l’option est invalide. Nous pouvons plutôt utiliser :

```bash
python3 -c 'import time; time.sleep(300)' --token FAUX_SECRET_DE_TEST &
pid=$!
```

Puis :

```bash
tr '\0' ' ' < /proc/$pid/cmdline
echo
```

Nous répondons :

1. Les arguments sont-ils visibles ?
    
2. Pourquoi ne faut-il jamais mettre un vrai secret ici ?
    
3. Quelle commande haut niveau peut aussi afficher ces arguments ?
    

Nous terminons :

```bash
kill $pid
```


## Exercice 3 — Variables d’environnement

Nous lançons :

```bash
TEST_SECRET=FAUX_SECRET_DE_TEST sleep 300 &
pid=$!
```

Puis :

```bash
tr '\0' '\n' < /proc/$pid/environ | grep TEST_SECRET
```

Nous répondons :

1. La variable est-elle visible ?
    
2. Qui peut la lire ?
    
3. Pourquoi les variables d’environnement ne sont-elles pas un coffre-fort ?
    
4. Quelles alternatives pouvons-nous envisager ?
    

Nous terminons :

```bash
kill $pid
```


## Exercice 4 — Vérifier `hidepid`

Nous exécutons :

```bash
findmnt /proc
mount | grep ' /proc '
```

Nous répondons :

1. L’option `hidepid` est-elle active ?
    
2. Si oui, quelle valeur ?
    
3. Que changerait `hidepid=2` ?
    
4. Quels outils pourraient être impactés ?
    


## Exercice 5 — Paramètres de sécurité `sysctl`

Nous exécutons :

```bash
sysctl kernel.randomize_va_space
sysctl kernel.dmesg_restrict
sysctl kernel.kptr_restrict
sysctl fs.protected_symlinks
sysctl fs.protected_hardlinks
```

Nous répondons :

1. L’ASLR est-il activé ?
    
2. L’accès à `dmesg` est-il restreint ?
    
3. Les pointeurs noyau sont-ils restreints ?
    
4. Les protections sur symlinks et hardlinks sont-elles activées ?
    
5. Pourquoi ces paramètres réduisent-ils l’exposition d’informations ?
    


## Exercice 6 — `/proc` dans un conteneur

Dans un conteneur de test, nous exécutons :

```bash
cat /proc/1/comm
ls /proc | grep -E '^[0-9]+$'
cat /proc/net/dev
cat /proc/self/mounts | head
cat /proc/1/cgroup
```

Nous répondons :

1. Quel est le PID 1 dans le conteneur ?
    
2. Combien de processus voyons-nous ?
    
3. Quelles interfaces réseau voyons-nous ?
    
4. La vue correspond-elle à l’hôte ?
    
5. Quels namespaces semblent influencer ces résultats ?
    


## 9.17. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc` expose des informations puissantes et parfois sensibles.
    
2. `cmdline` peut révéler des secrets passés en arguments.
    
3. `environ` peut révéler des secrets passés en variables d’environnement.
    
4. `fd` peut révéler les fichiers, sockets et ressources ouvertes.
    
5. `maps` peut révéler la structure mémoire et les bibliothèques chargées.
    
6. `/proc/<PID>/mem` est extrêmement sensible.
    
7. Les permissions dépendent de l’UID, des droits, des capacités et des mécanismes de sécurité.
    
8. `ptrace_scope` peut restreindre le debugging et certains accès inter-processus.
    
9. `hidepid` permet de limiter la visibilité des processus entre utilisateurs.
    
10. `/proc/net` dépend du namespace réseau courant.
    
11. `/proc/1` dépend du namespace PID courant.
    
12. Les conteneurs ne sont pas des VM : ils partagent le noyau avec l’hôte.
    
13. Les conteneurs privilégiés et les montages de `/proc` de l’hôte sont dangereux.
    
14. Les paramètres de `/proc/sys` peuvent renforcer ou affaiblir la sécurité.
    
15. Le durcissement doit être testé, documenté et adapté au contexte.
    
16. La sécurité de `/proc` repose sur une défense en profondeur.
    


## Conclusion du chapitre 9

Nous savons maintenant analyser `/proc` comme une surface d’observation sensible.

`/proc` est indispensable pour comprendre et administrer Linux, mais il peut aussi révéler trop d’informations si le système est mal configuré ou si les applications manipulent mal leurs secrets.

Nous avons étudié les risques liés aux lignes de commande, aux variables d’environnement, aux fichiers ouverts, aux mappings mémoire, aux paramètres noyau et aux conteneurs. Nous avons aussi vu plusieurs mécanismes de protection : permissions Unix, restrictions `ptrace`, option `hidepid`, paramètres `sysctl`, namespaces et bonnes pratiques applicatives.

Nous retenons surtout une idée : `/proc` n’est pas dangereux en soi, mais il rend visibles des informations qui doivent être maîtrisées. Un bon administrateur sait l’utiliser pour diagnostiquer, mais aussi le restreindre lorsque le contexte l’exige.

Dans le chapitre suivant, nous étudions le lien entre `/proc` et les outils système comme `ps`, `top`, `htop`, `free`, `uptime`, `lsof`, `ss`, `vmstat` et `strace`.

# Chapitre 10 — `/proc` et les outils système

## Objectifs du chapitre

Dans ce chapitre, nous étudions le lien entre `/proc` et les outils système classiques.

Depuis le début du cours, nous lisons directement des fichiers comme :

```bash
/proc/cpuinfo
/proc/meminfo
/proc/loadavg
/proc/<PID>/status
/proc/<PID>/fd
/proc/net/tcp
```

Mais, en pratique, nous utilisons souvent des outils plus confortables :

```bash
ps
top
htop
free
uptime
vmstat
lsof
ss
pidstat
strace
```

Ces outils ne remplacent pas `/proc` : ils s’appuient souvent sur lui, ou sur des interfaces noyau proches, pour produire une vue plus lisible.

À la fin de ce chapitre, nous savons :

- comprendre le rôle de `/proc` comme source d’information pour les outils système ;
    
- relier `ps`, `top`, `htop`, `free`, `uptime`, `lsof`, `ss` et `vmstat` aux données noyau ;
    
- utiliser `strace` pour observer les fichiers ouverts par un outil ;
    
- construire un mini-outil système en Bash ;
    
- comprendre les limites de l’observation par `/proc`.
    


## 10.1. Pourquoi les outils système existent-ils ?

### 10.1.1. `/proc` est complet, mais brut

`/proc` expose énormément d’informations.

Mais ces informations sont souvent :

- nombreuses ;
    
- dispersées ;
    
- peu formatées ;
    
- parfois difficiles à parser ;
    
- parfois encodées ;
    
- dépendantes du noyau ;
    
- difficiles à interpréter directement.
    

Par exemple :

```bash
cat /proc/loadavg
```

donne :

```text
0.35 0.42 0.38 2/842 15321
```

C’est compact, mais peu explicite pour un débutant.

La commande :

```bash
uptime
```

affiche une information plus lisible :

```text
14:32:01 up 3 days,  2:15,  2 users,  load average: 0.35, 0.42, 0.38
```

Nous comprenons donc que les outils système ajoutent :

- du formatage ;
    
- de l’interprétation ;
    
- des calculs ;
    
- des filtres ;
    
- du tri ;
    
- une interface interactive ;
    
- parfois une agrégation de plusieurs sources.
    


### 10.1.2. Les outils comme couches d’abstraction

Nous pouvons représenter la relation ainsi :

```text
Noyau Linux
   |
   v
/proc, /sys, netlink, appels système, cgroups
   |
   v
outils système : ps, top, free, ss, lsof, vmstat...
   |
   v
administrateur, développeur, supervision
```

`/proc` est donc une interface bas niveau.

Les outils système sont des couches de lecture et de présentation.

Nous devons savoir utiliser les deux niveaux :

- les outils pour aller vite ;
    
- `/proc` pour comprendre, vérifier ou diagnostiquer finement.
    


## 10.2. `ps` et `/proc`

### 10.2.1. Rôle de `ps`

La commande `ps` affiche une photographie des processus.

Exemple :

```bash
ps aux
```

ou :

```bash
ps -eo pid,ppid,state,user,comm,args
```

Elle affiche notamment :

- PID ;
    
- utilisateur ;
    
- état ;
    
- consommation CPU ;
    
- consommation mémoire ;
    
- commande ;
    
- arguments ;
    
- parent ;
    
- terminal ;
    
- temps CPU.
    

Une grande partie de ces informations peut être retrouvée dans `/proc/<PID>`.


### 10.2.2. Correspondance entre `ps` et `/proc`

|Information affichée par `ps`|Source possible dans `/proc`|
|---|---|
|PID|nom du répertoire `/proc/<PID>`|
|PPID|`/proc/<PID>/status` ou `/proc/<PID>/stat`|
|état|`/proc/<PID>/status` ou `/proc/<PID>/stat`|
|commande courte|`/proc/<PID>/comm`|
|arguments|`/proc/<PID>/cmdline`|
|utilisateur|`/proc/<PID>/status`|
|mémoire RSS|`/proc/<PID>/status` ou `/proc/<PID>/statm`|
|temps CPU|`/proc/<PID>/stat`|
|terminal|`/proc/<PID>/stat` et informations associées|

`ps` lit donc des informations brutes, les assemble, puis les affiche sous forme tabulaire.


### 10.2.3. Reproduire une petite partie de `ps`

Nous pouvons écrire une mini-version très simplifiée :

```bash
printf "%8s %8s %-2s %-20s %s\n" "PID" "PPID" "S" "NAME" "CMDLINE"

for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue

    pid=${dir#/proc/}

    name=$(awk '/^Name:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    state=$(awk '/^State:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    ppid=$(awk '/^PPid:/ {print $2}' "$dir/status" 2>/dev/null) || continue
    cmd=$(tr '\0' ' ' < "$dir/cmdline" 2>/dev/null)

    printf "%8s %8s %-2s %-20s %s\n" "$pid" "$ppid" "$state" "$name" "$cmd"
done | head
```

Ce script montre que `ps` n’est pas magique.

Il fait cependant beaucoup plus que ce script : il gère les erreurs, les permissions, les threads, les formats, les tris et les colonnes complexes.


### 10.2.4. Pourquoi `ps` est plus fiable qu’un script naïf

Notre script pédagogique peut échouer dans plusieurs cas :

- un processus disparaît pendant la lecture ;
    
- un fichier devient inaccessible ;
    
- une ligne de commande contient des caractères particuliers ;
    
- un utilisateur n’a pas les permissions ;
    
- un processus noyau n’a pas de `cmdline` ;
    
- un champ attendu est absent ;
    
- un processus change d’état pendant l’analyse.
    

`ps` gère beaucoup de ces cas.

Nous retenons :

```text
Lire /proc nous aide à comprendre.
Utiliser ps nous aide à travailler efficacement.
```


## 10.3. `top`, `htop` et `/proc`

### 10.3.1. Rôle de `top`

La commande :

```bash
top
```

donne une vue dynamique du système.

Elle affiche notamment :

- charge moyenne ;
    
- nombre de tâches ;
    
- CPU utilisé ;
    
- mémoire ;
    
- swap ;
    
- liste des processus ;
    
- consommation CPU par processus ;
    
- consommation mémoire par processus ;
    
- état des processus.
    

Pour construire cette vue, `top` lit plusieurs sources d’information, notamment des fichiers sous `/proc`.


### 10.3.2. Informations globales utilisées

`top` peut exploiter des informations similaires à :

```bash
/proc/stat
/proc/meminfo
/proc/loadavg
/proc/uptime
```

Ces fichiers permettent de connaître :

- les temps CPU cumulés ;
    
- la mémoire totale et disponible ;
    
- la charge moyenne ;
    
- l’uptime ;
    
- le nombre de processus.
    


### 10.3.3. Informations par processus utilisées

Pour chaque processus, `top` peut lire des informations proches de :

```bash
/proc/<PID>/stat
/proc/<PID>/status
/proc/<PID>/cmdline
```

Il peut ensuite calculer :

- le pourcentage CPU ;
    
- la mémoire résidente ;
    
- l’état ;
    
- le temps CPU cumulé ;
    
- la commande ;
    
- l’utilisateur.
    

Le pourcentage CPU est calculé à partir d’une différence entre deux observations dans le temps.


### 10.3.4. Pourquoi `top` observe dans le temps

Un fichier comme `/proc/<PID>/stat` contient des compteurs cumulés.

Pour calculer une consommation CPU instantanée, il faut :

1. lire une première valeur ;
    
2. attendre un intervalle ;
    
3. lire une deuxième valeur ;
    
4. calculer la différence ;
    
5. rapporter cette différence au temps écoulé et au nombre de CPU.
    

C’est ce que font les outils interactifs.

Nous pouvons donc comprendre pourquoi une mesure CPU n’est pas simplement une valeur statique lue dans un fichier.


### 10.3.5. `htop`

`htop` est une interface plus confortable que `top`.

Il ajoute :

- navigation interactive ;
    
- arbre des processus ;
    
- recherche ;
    
- tri visuel ;
    
- barres CPU et mémoire ;
    
- affichage des threads ;
    
- envoi de signaux ;
    
- personnalisation des colonnes.
    

Mais conceptuellement, `htop` repose sur les mêmes grandes idées : lire l’état du noyau, le reformater, l’actualiser régulièrement.


## 10.4. `free` et `/proc/meminfo`

### 10.4.1. Rôle de `free`

La commande :

```bash
free -h
```

affiche une synthèse mémoire :

```text
               total        used        free      shared  buff/cache   available
Mem:            15Gi       4.2Gi       1.1Gi       320Mi       9.7Gi        10Gi
Swap:          2.0Gi          0B       2.0Gi
```

Elle rend lisible ce que `/proc/meminfo` expose en détail.


### 10.4.2. Source principale : `/proc/meminfo`

Nous pouvons comparer :

```bash
cat /proc/meminfo | head -20
free -h
```

`free` utilise notamment des champs comme :

```text
MemTotal
MemFree
MemAvailable
Buffers
Cached
SwapTotal
SwapFree
Shmem
```

Il les présente sous forme synthétique.


### 10.4.3. Interprétation correcte

Le point fondamental reste celui vu au chapitre 3 :

```text
MemFree faible ne signifie pas forcément manque de mémoire.
MemAvailable est souvent plus utile pour évaluer la mémoire réellement disponible.
```

`free` nous aide à éviter une mauvaise lecture en présentant une colonne `available`.

Mais nous devons savoir que cette colonne provient d’une interprétation des données noyau.


## 10.5. `uptime` et `/proc/loadavg`

### 10.5.1. Rôle de `uptime`

La commande :

```bash
uptime
```

affiche :

- l’heure courante ;
    
- le temps depuis le démarrage ;
    
- le nombre d’utilisateurs connectés ;
    
- la charge moyenne à 1, 5 et 15 minutes.
    

Exemple :

```text
14:42:10 up 5 days,  3:12,  2 users,  load average: 0.25, 0.31, 0.29
```


### 10.5.2. Sources liées dans `/proc`

Nous pouvons comparer :

```bash
cat /proc/uptime
cat /proc/loadavg
uptime
```

`/proc/uptime` donne le temps depuis le démarrage.

`/proc/loadavg` donne les charges moyennes.

`uptime` reformate ces informations et les combine avec d’autres informations système.


### 10.5.3. Lire la tendance de charge

Nous utilisons `uptime` pour aller vite, mais nous devons interpréter correctement les trois valeurs :

```text
load average: 8.00, 4.00, 2.00
```

La charge monte.

```text
load average: 2.00, 4.00, 8.00
```

La charge descend.

Nous devons aussi comparer ces valeurs au nombre de CPU logiques :

```bash
nproc
```

ou :

```bash
grep -c '^processor' /proc/cpuinfo
```


## 10.6. `vmstat` et `/proc`

### 10.6.1. Rôle de `vmstat`

La commande :

```bash
vmstat 1
```

affiche périodiquement des statistiques sur :

- processus en attente ;
    
- mémoire ;
    
- swap ;
    
- entrées/sorties ;
    
- interruptions ;
    
- changements de contexte ;
    
- CPU.
    

Exemple :

```text
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 900000 120000 8000000  0    0     1     2  200  500  5  2 92  1  0
```


### 10.6.2. Sources d’information

`vmstat` agrège des informations qui peuvent venir de sources comme :

```text
/proc/stat
/proc/meminfo
/proc/vmstat
/proc/diskstats
```

Selon les versions et les implémentations, l’outil peut utiliser plusieurs interfaces noyau.


### 10.6.3. Lire `/proc/vmstat`

Nous pouvons observer :

```bash
cat /proc/vmstat | head
```

Ce fichier contient de nombreux compteurs liés à la mémoire virtuelle.

Exemples de champs possibles :

```text
pgpgin
pgpgout
pswpin
pswpout
pgfault
pgmajfault
```

Ces compteurs sont cumulés.

Un outil comme `vmstat` calcule des différences entre deux lectures pour produire des valeurs par seconde.


### 10.6.4. Ce que `vmstat` apporte

`vmstat` est très utile pour savoir si une machine souffre de :

- pression mémoire ;
    
- swap actif ;
    
- attente I/O ;
    
- activité disque ;
    
- charge CPU ;
    
- nombreux changements de contexte ;
    
- processus bloqués.
    

`/proc` contient les compteurs.

`vmstat` les met en forme et les rend exploitables en temps réel.


## 10.7. `lsof` et `/proc/<PID>/fd`

### 10.7.1. Rôle de `lsof`

La commande :

```bash
lsof
```

signifie “list open files”.

Elle affiche les fichiers ouverts par les processus.

Exemple :

```bash
sudo lsof -p <PID>
```

Elle peut afficher :

```text
COMMAND PID USER FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   932 root cwd  DIR  259,2     4096  128 /
nginx   932 root txt  REG  259,2  1260032  456 /usr/sbin/nginx
nginx   932 root 1w   REG  259,2   123456  789 /var/log/nginx/access.log
nginx   932 root 2w   REG  259,2    23456  790 /var/log/nginx/error.log
nginx   932 root 6u  IPv4 123456      0t0  TCP *:80 (LISTEN)
```


### 10.7.2. Lien avec `/proc/<PID>/fd`

Nous avons vu au chapitre 5 que :

```bash
ls -l /proc/<PID>/fd
```

affiche les descripteurs ouverts.

`lsof` va plus loin :

- il identifie le type de ressource ;
    
- il indique les modes d’ouverture ;
    
- il résout des noms ;
    
- il associe des sockets réseau ;
    
- il affiche l’utilisateur ;
    
- il permet de filtrer.
    


### 10.7.3. Exemples utiles

Voir les fichiers ouverts par un processus :

```bash
sudo lsof -p <PID>
```

Voir les processus qui utilisent un fichier :

```bash
sudo lsof /chemin/du/fichier
```

Voir les processus qui utilisent un répertoire ou un montage :

```bash
sudo lsof +D /mnt/data
```

Voir les sockets réseau :

```bash
sudo lsof -i -P -n
```

Voir les processus écoutant sur un port :

```bash
sudo lsof -i :8080 -P -n
```


### 10.7.4. Quand préférons-nous `/proc` ?

Nous utilisons directement `/proc` lorsque :

- `lsof` n’est pas installé ;
    
- l’environnement est minimal ;
    
- nous voulons inspecter un descripteur précis ;
    
- nous voulons comprendre la source brute ;
    
- nous cherchons un fichier supprimé encore ouvert ;
    
- nous écrivons un script très léger.
    

Nous utilisons `lsof` lorsque nous voulons une vue lisible et rapide.


## 10.8. `ss` et `/proc/net`

### 10.8.1. Rôle de `ss`

La commande :

```bash
ss
```

permet d’observer les sockets.

Elle remplace largement l’ancien outil `netstat`.

Exemples :

```bash
ss -tan
ss -ltn
sudo ss -tulpen
ss -x
```

Elle affiche :

- sockets TCP ;
    
- sockets UDP ;
    
- sockets Unix ;
    
- ports en écoute ;
    
- connexions établies ;
    
- processus associés ;
    
- files d’attente.
    


### 10.8.2. Comparaison avec `/proc/net`

Nous pouvons comparer :

```bash
cat /proc/net/tcp
ss -tan
```

`/proc/net/tcp` affiche des adresses hexadécimales et des codes d’état.

`ss` affiche :

```text
State      Recv-Q Send-Q Local Address:Port   Peer Address:Port
LISTEN     0      4096   127.0.0.1:8080      0.0.0.0:*
ESTAB      0      0      192.168.1.10:443    192.168.1.20:51322
```

Nous voyons que `ss` décode et présente les informations.


### 10.8.3. Voir les ports en écoute

Commande pratique :

```bash
sudo ss -tulpen
```

Options :

|Option|Rôle|
|---|---|
|`-t`|TCP|
|`-u`|UDP|
|`-l`|sockets en écoute|
|`-p`|processus|
|`-e`|informations étendues|
|`-n`|pas de résolution DNS/service|

Cette commande est souvent la première à utiliser pour savoir quels services écoutent sur une machine.


### 10.8.4. Pourquoi `ss` est préférable à la lecture brute

Lire `/proc/net/tcp` directement est utile pédagogiquement.

Mais en production, `ss` est préférable parce qu’il :

- décode les adresses ;
    
- affiche les ports lisiblement ;
    
- traduit les états TCP ;
    
- associe les processus ;
    
- gère IPv4 et IPv6 ;
    
- affiche TCP, UDP et Unix ;
    
- fournit des filtres puissants.
    


## 10.9. `pidstat` et les statistiques par processus

### 10.9.1. Rôle de `pidstat`

La commande :

```bash
pidstat
```

fait partie du paquet `sysstat`.

Elle permet d’observer l’activité des processus dans le temps.

Exemples :

```bash
pidstat 1
pidstat -r 1
pidstat -d 1
pidstat -u 1
```

Elle peut afficher :

- CPU par processus ;
    
- mémoire ;
    
- fautes de page ;
    
- I/O ;
    
- threads ;
    
- changements de contexte.
    


### 10.9.2. Lien avec `/proc`

`pidstat` exploite des informations provenant de `/proc`, par exemple :

```text
/proc/<PID>/stat
/proc/<PID>/status
/proc/<PID>/io
/proc/stat
```

Comme `top`, il calcule des différences entre lectures successives.


### 10.9.3. Utilité pratique

`pidstat` est très utile lorsqu’un problème est intermittent.

Exemple :

```bash
pidstat -u -r -d 1
```

Nous pouvons observer en temps réel :

- quel processus consomme du CPU ;
    
- quel processus fait des lectures/écritures ;
    
- quel processus voit sa mémoire augmenter ;
    
- quel processus change de comportement.
    

`/proc` fournit les compteurs.

`pidstat` produit une observation temporelle claire.


## 10.10. `strace` pour voir les accès à `/proc`

### 10.10.1. Rôle de `strace`

`strace` permet d’observer les appels système effectués par un programme.

Nous pouvons l’utiliser pour voir quels fichiers un outil ouvre.

Exemple :

```bash
strace -e openat ps -p $$
```

Nous voyons des appels comme :

```text
openat(AT_FDCWD, "/proc/12345/stat", O_RDONLY) = 3
openat(AT_FDCWD, "/proc/12345/status", O_RDONLY) = 3
openat(AT_FDCWD, "/proc/12345/cmdline", O_RDONLY) = 3
```

Cela montre que `ps` lit bien des informations dans `/proc`.


### 10.10.2. Observer `free`

Nous pouvons tester :

```bash
strace -e openat free -h
```

Nous pouvons voir une ouverture de :

```text
/proc/meminfo
```

Cela confirme le lien entre `free` et `/proc/meminfo`.


### 10.10.3. Observer `uptime`

Nous pouvons tester :

```bash
strace -e openat uptime
```

Nous pouvons voir des accès à des fichiers comme :

```text
/proc/uptime
/proc/loadavg
```

selon l’implémentation.


### 10.10.4. Observer `ss`

Nous pouvons utiliser :

```bash
strace -e openat ss -tan
```

Selon les versions, `ss` peut utiliser netlink plutôt que lire uniquement `/proc/net/tcp`.

C’est un point important : tous les outils modernes ne lisent pas exclusivement `/proc`.

Ils peuvent utiliser :

- `/proc` ;
    
- netlink ;
    
- appels système ;
    
- `/sys` ;
    
- cgroups ;
    
- bibliothèques système.
    

Nous retenons donc :

```text
/proc est une source majeure, mais pas la seule interface noyau.
```


## 10.11. Construire un mini-outil système en Bash

### 10.11.1. Objectif

Nous voulons construire un petit outil qui affiche une synthèse système à partir de `/proc`.

Il doit afficher :

- hostname ;
    
- version du noyau ;
    
- uptime ;
    
- charge moyenne ;
    
- CPU logiques ;
    
- mémoire totale ;
    
- mémoire disponible ;
    
- swap utilisé ;
    
- nombre de processus ;
    
- nombre de processus par état.
    


### 10.11.2. Première version simple

Nous créons un fichier :

```bash
nano mini-proc-status.sh
```

Contenu :

```bash
##!/usr/bin/env bash

hostname=$(cat /proc/sys/kernel/hostname 2>/dev/null || hostname)
kernel=$(uname -r)
cpu_count=$(grep -c '^processor' /proc/cpuinfo 2>/dev/null)

uptime_days=$(awk '{ printf "%.2f jours", $1/86400 }' /proc/uptime)
loadavg=$(awk '{ print $1 " " $2 " " $3 }' /proc/loadavg)

mem_total=$(awk '/^MemTotal:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo)
mem_available=$(awk '/^MemAvailable:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo)

swap_used=$(awk '
/^SwapTotal:/ { total=$2 }
/^SwapFree:/ { free=$2 }
END { printf "%.1f MiB", (total-free)/1024 }
' /proc/meminfo)

process_count=$(find /proc -maxdepth 1 -regex '/proc/[0-9]+' 2>/dev/null | wc -l)

echo "Hostname           : $hostname"
echo "Kernel             : $kernel"
echo "CPU logiques       : $cpu_count"
echo "Uptime             : $uptime_days"
echo "Load average       : $loadavg"
echo "Mémoire totale     : $mem_total"
echo "Mémoire disponible : $mem_available"
echo "Swap utilisé       : $swap_used"
echo "Processus visibles : $process_count"
```

Nous rendons le script exécutable :

```bash
chmod +x mini-proc-status.sh
```

Puis nous exécutons :

```bash
./mini-proc-status.sh
```


### 10.11.3. Ajouter les états des processus

Nous pouvons ajouter un comptage des états.

```bash
echo
echo "États des processus"
echo "-------------------"

for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue
    awk '/^State:/ { print $2 }' "$dir/status" 2>/dev/null
done | sort | uniq -c
```

Sortie possible :

```text
      2 D
      3 R
    180 S
      1 Z
```

Nous pouvons intégrer cela au script.


### 10.11.4. Version plus complète

```bash
##!/usr/bin/env bash

set -u

read_file() {
    local file="$1"
    [ -r "$file" ] && cat "$file"
}

hostname=$(read_file /proc/sys/kernel/hostname || hostname)
kernel=$(uname -r)
cpu_count=$(grep -c '^processor' /proc/cpuinfo 2>/dev/null || echo "?")

uptime_days=$(awk '{ printf "%.2f jours", $1/86400 }' /proc/uptime 2>/dev/null || echo "?")
loadavg=$(awk '{ print $1 " " $2 " " $3 }' /proc/loadavg 2>/dev/null || echo "?")

mem_total=$(awk '/^MemTotal:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo 2>/dev/null || echo "?")
mem_available=$(awk '/^MemAvailable:/ { printf "%.1f GiB", $2/1024/1024 }' /proc/meminfo 2>/dev/null || echo "?")

swap_used=$(awk '
/^SwapTotal:/ { total=$2 }
/^SwapFree:/ { free=$2 }
END {
    if (total == "") print "?"
    else printf "%.1f MiB", (total-free)/1024
}
' /proc/meminfo 2>/dev/null)

process_count=$(find /proc -maxdepth 1 -regex '/proc/[0-9]+' 2>/dev/null | wc -l)

echo "Synthèse système"
echo "================"
echo "Hostname           : $hostname"
echo "Kernel             : $kernel"
echo "CPU logiques       : $cpu_count"
echo "Uptime             : $uptime_days"
echo "Load average       : $loadavg"
echo "Mémoire totale     : $mem_total"
echo "Mémoire disponible : $mem_available"
echo "Swap utilisé       : $swap_used"
echo "Processus visibles : $process_count"

echo
echo "États des processus"
echo "-------------------"

for dir in /proc/[0-9]*; do
    [ -r "$dir/status" ] || continue
    awk '/^State:/ { print $2 }' "$dir/status" 2>/dev/null
done | sort | uniq -c
```

Ce script reste simple, mais il montre comment bâtir un outil d’observation avec `/proc`.


## 10.12. Construire un mini-outil en Python

### 10.12.1. Objectif

Nous pouvons aussi écrire un outil Python.

Python est plus adapté si nous voulons :

- structurer les données ;
    
- gérer les erreurs proprement ;
    
- produire du JSON ;
    
- construire une API ;
    
- intégrer l’outil à une supervision ;
    
- faire des calculs plus complexes.
    


### 10.12.2. Exemple simple

```python
##!/usr/bin/env python3

from pathlib import Path
import os
import json

PROC = Path("/proc")


def read_text(path: Path) -> str | None:
    try:
        return path.read_text().strip()
    except (FileNotFoundError, PermissionError, ProcessLookupError):
        return None


def parse_meminfo() -> dict[str, int]:
    data: dict[str, int] = {}
    content = read_text(PROC / "meminfo")
    if content is None:
        return data

    for line in content.splitlines():
        key, value = line.split(":", 1)
        parts = value.strip().split()
        if parts:
            data[key] = int(parts[0])
    return data


def count_processes() -> int:
    return sum(1 for p in PROC.iterdir() if p.name.isdigit())


def process_states() -> dict[str, int]:
    states: dict[str, int] = {}

    for p in PROC.iterdir():
        if not p.name.isdigit():
            continue

        status = read_text(p / "status")
        if status is None:
            continue

        for line in status.splitlines():
            if line.startswith("State:"):
                state = line.split()[1]
                states[state] = states.get(state, 0) + 1
                break

    return states


def main() -> None:
    mem = parse_meminfo()

    uptime_content = read_text(PROC / "uptime")
    uptime_seconds = None
    if uptime_content:
        uptime_seconds = float(uptime_content.split()[0])

    loadavg = read_text(PROC / "loadavg")

    result = {
        "hostname": read_text(PROC / "sys/kernel/hostname"),
        "kernel": os.uname().release,
        "cpu_logical": sum(
            1
            for line in (read_text(PROC / "cpuinfo") or "").splitlines()
            if line.startswith("processor")
        ),
        "uptime_seconds": uptime_seconds,
        "loadavg": loadavg.split()[:3] if loadavg else None,
        "memory": {
            "MemTotal_kB": mem.get("MemTotal"),
            "MemAvailable_kB": mem.get("MemAvailable"),
            "SwapTotal_kB": mem.get("SwapTotal"),
            "SwapFree_kB": mem.get("SwapFree"),
        },
        "process_count": count_processes(),
        "process_states": process_states(),
    }

    print(json.dumps(result, indent=2, ensure_ascii=False))


if __name__ == "__main__":
    main()
```

Nous pouvons l’enregistrer sous :

```bash
mini_proc_status.py
```

Puis :

```bash
chmod +x mini_proc_status.py
./mini_proc_status.py
```


### 10.12.3. Pourquoi gérer les exceptions ?

Lorsque nous parcourons `/proc`, plusieurs erreurs sont normales :

- le processus disparaît ;
    
- le fichier n’est pas lisible ;
    
- le fichier n’existe plus ;
    
- les permissions refusent l’accès ;
    
- le format diffère.
    

C’est pourquoi nous gérons :

```python
FileNotFoundError
PermissionError
ProcessLookupError
```

Un outil robuste ne considère pas ces cas comme des catastrophes. Il les traite comme des événements normaux lors de l’observation d’un système vivant.


## 10.13. Comparaison des outils

### 10.13.1. Tableau synthétique

|Besoin|Fichier `/proc`|Outil pratique|
|---|---|---|
|Voir les processus|`/proc/<PID>`|`ps`, `top`, `htop`|
|Voir la mémoire|`/proc/meminfo`|`free`, `vmstat`|
|Voir la charge|`/proc/loadavg`|`uptime`, `top`|
|Voir l’uptime|`/proc/uptime`|`uptime`|
|Voir les fichiers ouverts|`/proc/<PID>/fd`|`lsof`|
|Voir les sockets|`/proc/net/*`|`ss`, `lsof -i`|
|Voir les paramètres noyau|`/proc/sys`|`sysctl`|
|Voir les stats CPU|`/proc/stat`|`top`, `mpstat`, `vmstat`|
|Voir les I/O disque|`/proc/diskstats`, `/proc/<PID>/io`|`iostat`, `pidstat -d`|
|Observer les appels système|appels système|`strace`|


### 10.13.2. Quand lire directement `/proc` ?

Nous lisons directement `/proc` lorsque :

- nous voulons comprendre le fonctionnement interne ;
    
- nous sommes dans un environnement minimal ;
    
- un outil haut niveau n’est pas disponible ;
    
- nous voulons écrire un script simple ;
    
- nous voulons vérifier la source brute ;
    
- nous voulons diagnostiquer un cas très précis ;
    
- nous enseignons les systèmes d’exploitation.
    


### 10.13.3. Quand utiliser les outils ?

Nous utilisons les outils lorsque :

- nous voulons aller vite ;
    
- nous avons besoin d’une présentation lisible ;
    
- nous voulons trier ou filtrer ;
    
- nous voulons une vue temps réel ;
    
- nous voulons éviter les erreurs de parsing ;
    
- nous voulons une sortie connue par les administrateurs ;
    
- nous sommes en production.
    

Les deux approches se complètent.


## 10.14. Limites des outils fondés sur `/proc`

### 10.14.1. Les outils voient une photographie

`ps` donne une photographie à un instant donné.

Un processus peut apparaître puis disparaître entre deux observations.

Un état peut changer très rapidement.

Nous devons éviter de surinterpréter une seule lecture.


### 10.14.2. Les outils calculent des moyennes

`top`, `pidstat` ou `vmstat` calculent des valeurs sur des intervalles.

Le résultat dépend :

- de la durée d’échantillonnage ;
    
- de la fréquence ;
    
- du nombre de CPU ;
    
- de la charge ;
    
- du contexte virtualisé ou non.
    

Une mesure à une seconde peut ne pas refléter un pic très court.


### 10.14.3. Les conteneurs changent la vue

Dans un conteneur, les outils lisent souvent `/proc` depuis le point de vue du conteneur.

Cela signifie que :

- `ps` voit les processus du namespace PID ;
    
- `ss` voit le namespace réseau ;
    
- `free` peut parfois être trompeur selon la configuration ;
    
- `top` peut ne pas afficher correctement les limites cgroups ;
    
- les PID diffèrent entre hôte et conteneur.
    

Nous devons donc toujours demander :

```text
Depuis quel contexte l’outil est-il exécuté ?
```


### 10.14.4. Certains outils utilisent d’autres interfaces

Tous les outils ne lisent pas exclusivement `/proc`.

Certains utilisent aussi :

- netlink ;
    
- `/sys`;
    
- cgroups ;
    
- perf events ;
    
- eBPF ;
    
- appels système ;
    
- bibliothèques système.
    

C’est particulièrement vrai pour les outils réseau modernes ou les outils de performance avancés.


## 10.15. Cas pratique : diagnostic rapide avec outils et `/proc`

### 10.15.1. Situation

Nous nous connectons à un serveur lent.

Nous voulons faire un diagnostic rapide.

Nous combinons outils haut niveau et vérification dans `/proc`.


### 10.15.2. Étape 1 : charge et uptime

Outil :

```bash
uptime
```

Source brute :

```bash
cat /proc/loadavg
cat /proc/uptime
```

Nous interprétons la charge par rapport au nombre de CPU :

```bash
nproc
```


### 10.15.3. Étape 2 : mémoire

Outil :

```bash
free -h
```

Source brute :

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree):' /proc/meminfo
```

Nous cherchons :

- mémoire disponible ;
    
- swap utilisé ;
    
- cache ;
    
- pression mémoire potentielle.
    


### 10.15.4. Étape 3 : processus consommateurs

Outil :

```bash
top
```

ou :

```bash
ps -eo pid,ppid,state,%cpu,%mem,comm,args --sort=-%cpu | head
```

Source brute pour un PID suspect :

```bash
cat /proc/<PID>/comm
tr '\0' ' ' < /proc/<PID>/cmdline; echo
grep -E '^(Name|State|Pid|PPid|Threads|VmRSS):' /proc/<PID>/status
```


### 10.15.5. Étape 4 : fichiers ouverts

Outil :

```bash
sudo lsof -p <PID>
```

Source brute :

```bash
sudo ls -l /proc/<PID>/fd
```

Nous cherchons :

- fichiers logs ;
    
- sockets ;
    
- fichiers supprimés ;
    
- beaucoup de descripteurs ;
    
- ressources inattendues.
    


### 10.15.6. Étape 5 : réseau

Outil :

```bash
sudo ss -tulpen
```

Source brute :

```bash
cat /proc/net/tcp
cat /proc/net/udp
cat /proc/net/unix
```

Nous utilisons directement `ss` pour la lisibilité.

Nous utilisons `/proc/net` pour comprendre la représentation brute ou diagnostiquer un environnement minimal.


### 10.15.7. Étape 6 : hypothèse

À partir de ces éléments, nous formulons une hypothèse prudente.

Exemple :

```text
Nous observons une charge élevée, peu de CPU réellement utilisé, plusieurs processus en état D, et une attente I/O élevée dans vmstat. Nous suspectons un problème de stockage ou de système de fichiers plutôt qu’un calcul CPU.
```

Autre exemple :

```text
Nous observons une croissance continue du RSS d’un processus applicatif, sans augmentation proportionnelle du trafic. Nous suspectons une fuite mémoire ou une accumulation de cache applicatif non maîtrisée.
```


## 10.16. Exercices

## Exercice 1 — Relier `ps` à `/proc`

Nous exécutons :

```bash
ps -p $$ -o pid,ppid,state,comm,args
```

Puis :

```bash
cat /proc/$$/comm
tr '\0' ' ' < /proc/$$/cmdline; echo
grep -E '^(State|Pid|PPid):' /proc/$$/status
```

Nous répondons :

1. Quelles informations de `ps` retrouvons-nous dans `/proc` ?
    
2. Quelle information est plus lisible avec `ps` ?
    
3. Quelle information est plus brute dans `/proc` ?
    


## Exercice 2 — Relier `free` à `/proc/meminfo`

Nous exécutons :

```bash
free -h
```

Puis :

```bash
grep -E '^(MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree):' /proc/meminfo
```

Nous répondons :

1. Quels champs de `/proc/meminfo` apparaissent dans `free` ?
    
2. Pourquoi `available` est-il important ?
    
3. Pourquoi `free` est-il plus lisible ?
    


## Exercice 3 — Relier `uptime` à `/proc`

Nous exécutons :

```bash
uptime
cat /proc/uptime
cat /proc/loadavg
```

Nous répondons :

1. Où retrouvons-nous le temps depuis le démarrage ?
    
2. Où retrouvons-nous la charge moyenne ?
    
3. Que signifient les trois valeurs de load average ?
    


## Exercice 4 — Observer un outil avec `strace`

Nous exécutons :

```bash
strace -e openat free -h
```

Puis :

```bash
strace -e openat uptime
```

Nous répondons :

1. Quels fichiers sous `/proc` sont ouverts ?
    
2. Que prouve cette observation ?
    
3. Pourquoi certains outils peuvent-ils aussi utiliser d’autres interfaces ?
    


## Exercice 5 — Comparer `/proc/<PID>/fd` et `lsof`

Nous lançons :

```bash
sleep 300 &
pid=$!
```

Puis :

```bash
ls -l /proc/$pid/fd
lsof -p $pid
```

Nous répondons :

1. Quelles informations sont visibles dans les deux cas ?
    
2. Que rajoute `lsof` ?
    
3. Pourquoi `/proc/<PID>/fd` reste utile ?
    
4. Pourquoi `lsof` est plus pratique en production ?
    

Nous terminons :

```bash
kill $pid
```


## Exercice 6 — Comparer `/proc/net/tcp` et `ss`

Nous lançons un serveur local :

```bash
python3 -m http.server 8080 &
pid=$!
```

Puis :

```bash
cat /proc/net/tcp
ss -ltnp 'sport = :8080'
```

Nous répondons :

1. Quel format est le plus lisible ?
    
2. Comment le port 8080 apparaît-il dans `/proc/net/tcp` ?
    
3. Que rajoute `ss` ?
    
4. Pourquoi est-il utile de connaître les deux niveaux ?
    

Nous terminons :

```bash
kill $pid
```


## Exercice 7 — Construire un mini-outil

Nous écrivons un script Bash ou Python qui affiche :

```text
hostname
kernel
uptime
load average
CPU logiques
mémoire totale
mémoire disponible
swap utilisé
nombre de processus
états des processus
```

Nous utilisons `/proc` autant que possible.

Nous comparons ensuite notre sortie avec :

```bash
hostname
uname -r
uptime
free -h
ps
```


## 10.17. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc` expose des informations brutes du noyau.
    
2. Les outils système rendent ces informations plus lisibles.
    
3. `ps` s’appuie largement sur `/proc/<PID>`.
    
4. `top` et `htop` observent le système dans le temps.
    
5. `free` synthétise `/proc/meminfo`.
    
6. `uptime` reformate `/proc/uptime` et `/proc/loadavg`.
    
7. `vmstat` exploite des compteurs système cumulés.
    
8. `lsof` enrichit la lecture de `/proc/<PID>/fd`.
    
9. `ss` rend lisibles les informations réseau, parfois via d’autres interfaces noyau modernes.
    
10. `pidstat` observe les processus dans le temps à partir de compteurs.
    
11. `strace` permet de vérifier quels fichiers et appels système un outil utilise.
    
12. Un outil haut niveau peut utiliser `/proc`, mais aussi netlink, `/sys`, cgroups ou d’autres interfaces.
    
13. Lire `/proc` nous aide à comprendre ce que font les outils.
    
14. Utiliser les outils nous aide à diagnostiquer plus vite.
    
15. Dans les conteneurs, les outils héritent des limites de visibilité imposées par les namespaces et cgroups.
    


## Conclusion du chapitre 10

Nous savons maintenant relier `/proc` aux outils système classiques.

Nous comprenons que des commandes comme `ps`, `top`, `free`, `uptime`, `lsof`, `ss`, `vmstat` ou `pidstat` ne sortent pas leurs informations de nulle part : elles lisent, agrègent, calculent et présentent des données fournies par le noyau.

Cette compréhension est importante pour un niveau Master 2. Elle nous permet de ne pas utiliser les outils comme des boîtes noires. Nous savons revenir aux sources brutes, vérifier une information, comprendre une incohérence et écrire nos propres outils d’observation.

Dans le chapitre suivant, nous passons à une série de travaux pratiques structurés pour consolider l’ensemble du cours sur `/proc`.

------
---------------------------
----------


# 12. Étude de cas : diagnostic d’un serveur

Nous simulons un incident de production :

- charge système élevée
    
- mémoire presque saturée
    
- processus qui consomme trop de fichiers ouverts
    
- suspicion de fuite mémoire
    
- logs supprimés mais espace disque toujours occupé
    

Nous utilisons `/proc` pour enquêter.

Nous construisons une démarche :

1. Nous identifions les processus actifs.
    
2. Nous observons la charge système.
    
3. Nous analysons la mémoire globale.
    
4. Nous inspectons le processus suspect.
    
5. Nous regardons ses fichiers ouverts.
    
6. Nous examinons sa mémoire.
    
7. Nous formulons une hypothèse.
    
8. Nous proposons une action corrective.
    


# Chapitre 13 — Limites de `/proc`

## Objectifs du chapitre

Dans ce chapitre, nous prenons du recul.

Depuis le début du cours, nous utilisons `/proc` comme une interface puissante pour observer le système Linux : processus, mémoire, fichiers ouverts, réseau, paramètres noyau, sécurité et outils système.

Nous devons maintenant comprendre une idée essentielle : `/proc` est indispensable, mais il n’est pas suffisant.

À la fin de ce chapitre, nous savons :

- identifier les limites techniques de `/proc` ;
    
- comprendre pourquoi certains fichiers sont difficiles à parser ;
    
- expliquer les différences selon les noyaux, distributions et environnements ;
    
- éviter les pièges dans les conteneurs ;
    
- comprendre les risques de sécurité liés à une exposition excessive ;
    
- comparer `/proc` avec des approches modernes comme `/sys`, les cgroups, eBPF, `perf`, systemd, Prometheus et Kubernetes.
    


## 13.1. `/proc` est puissant, mais pas parfait

### 13.1.1. Une interface historique

`/proc` est une interface ancienne et centrale dans Linux.

Elle a été conçue pour exposer simplement certaines informations internes du noyau sous forme de fichiers.

Cette idée est très pratique :

```bash
cat /proc/meminfo
cat /proc/cpuinfo
cat /proc/loadavg
cat /proc/<PID>/status
```

Nous pouvons observer beaucoup d’informations sans bibliothèque complexe, simplement avec les outils Unix classiques.

Cependant, cette simplicité apparente masque plusieurs limites.

`/proc` n’est pas une API moderne uniformisée. C’est une collection d’interfaces textuelles, parfois historiques, parfois très bas niveau, parfois difficiles à interpréter correctement.


### 13.1.2. Une interface d’observation, pas une solution complète d’observabilité

`/proc` nous donne des données locales.

Mais une supervision moderne doit souvent répondre à des questions plus larges :

- que se passe-t-il sur plusieurs machines ?
    
- comment évolue la charge sur plusieurs heures ?
    
- quel service est responsable d’une dégradation ?
    
- quelle requête applicative provoque la saturation ?
    
- quel pod Kubernetes consomme trop ?
    
- quelle version de l’application était déployée au moment de l’incident ?
    
- quelles alertes devons-nous déclencher ?
    

`/proc` ne répond pas seul à ces questions.

Il fournit des signaux bruts. Nous devons ensuite les collecter, agréger, historiser, corréler et interpréter.


## 13.2. Formats parfois difficiles à parser

### 13.2.1. Des fichiers textuels, mais pas toujours simples

Un avantage de `/proc` est que beaucoup de fichiers sont textuels.

Mais “textuel” ne veut pas forcément dire “facile à parser”.

Exemple simple :

```bash
cat /proc/meminfo
```

Sortie typique :

```text
MemTotal:       16234568 kB
MemFree:         1234567 kB
MemAvailable:   9345672 kB
Buffers:          234567 kB
Cached:          4567890 kB
```

Ce fichier est relativement facile à lire.

Mais d’autres fichiers sont beaucoup plus difficiles.

Exemple :

```bash
cat /proc/net/tcp
```

Sortie typique :

```text
sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt uid timeout inode
0: 0100007F:1F90 00000000:0000 0A 00000000:00000000 00:00000000 00000000 1000 0 123456
```

Ici, les adresses sont encodées en hexadécimal, les octets sont inversés, les états TCP sont codés par nombres, et les colonnes demandent une connaissance précise du format.


### 13.2.2. Exemple : `/proc/<PID>/stat`

Le fichier :

```bash
cat /proc/<PID>/stat
```

est compact, mais difficile à parser correctement.

Exemple :

```text
18590 (mon processus) S 18420 18590 18420 34816 18590 ...
```

Le deuxième champ, `comm`, est entre parenthèses et peut contenir des espaces ou des caractères particuliers.

Un découpage naïf avec `awk '{print $1, $2, $3}'` peut produire une interprétation incorrecte si le nom du processus contient des espaces.

Nous devons donc être prudents.

Pour une lecture humaine, nous préférons souvent :

```bash
cat /proc/<PID>/status
```

qui est plus verbeux mais plus clair.


### 13.2.3. Exemple : `/proc/net/tcp`

Dans `/proc/net/tcp`, une adresse comme :

```text
0100007F:1F90
```

correspond à :

```text
127.0.0.1:8080
```

Mais cela suppose de savoir que :

- l’adresse IPv4 est en hexadécimal ;
    
- l’ordre des octets est inversé ;
    
- le port est en hexadécimal ;
    
- l’état TCP est aussi codé.
    

C’est pour cela qu’en pratique nous utilisons :

```bash
ss -tan
```

ou :

```bash
sudo ss -tulpen
```

Ces commandes décodent les informations pour nous.


### 13.2.4. Scripts fragiles

Un script qui lit `/proc` peut être fragile s’il suppose :

- qu’une ligne existe toujours ;
    
- qu’un champ est toujours à la même position ;
    
- qu’un fichier est toujours lisible ;
    
- qu’un processus reste vivant pendant la lecture ;
    
- que le format est identique sur toutes les versions de noyau ;
    
- que l’environnement n’est pas conteneurisé.
    

Exemple fragile :

```bash
cat /proc/$pid/status | grep VmRSS | awk '{print $2}'
```

Version un peu plus propre :

```bash
awk '/^VmRSS:/ { print $2 }' "/proc/$pid/status" 2>/dev/null
```

Mais même cette version doit gérer le cas où le processus disparaît.


### 13.2.5. Bonne pratique

Lorsque nous écrivons un outil basé sur `/proc`, nous devons :

- chercher les clés explicitement ;
    
- gérer les erreurs ;
    
- accepter les champs absents ;
    
- éviter les hypothèses trop fortes ;
    
- tester sur plusieurs systèmes ;
    
- documenter les versions de noyau ciblées ;
    
- préférer les outils spécialisés quand le parsing devient trop complexe.
    

Nous devons voir `/proc` comme une source brute, pas toujours comme une API stable de haut niveau.


## 13.3. Informations très bas niveau

### 13.3.1. `/proc` expose des faits, pas toujours leur signification

`/proc` donne des informations proches du noyau.

Mais ces informations demandent souvent une interprétation.

Exemple :

```bash
cat /proc/loadavg
```

Sortie :

```text
8.00 4.00 2.00 3/842 15321
```

Cette sortie ne dit pas directement :

```text
Votre serveur est lent parce que le disque est saturé.
```

Elle donne une charge moyenne.

À nous de comprendre si cette charge vient :

- du CPU ;
    
- d’attentes I/O ;
    
- de processus bloqués ;
    
- de contention ;
    
- d’un pic temporaire ;
    
- d’un problème de stockage ;
    
- d’une limite de conteneur.
    


### 13.3.2. Exemple avec la mémoire

`/proc/meminfo` donne beaucoup d’informations :

```bash
cat /proc/meminfo
```

Mais interpréter correctement la mémoire Linux demande de comprendre :

- la mémoire libre ;
    
- la mémoire disponible ;
    
- le cache ;
    
- les buffers ;
    
- le swap ;
    
- les pages dirty ;
    
- les pages writeback ;
    
- les allocations noyau ;
    
- les cgroups éventuels.
    

Un débutant peut voir :

```text
MemFree: très faible
```

et conclure :

```text
Le système n’a plus de mémoire.
```

Alors qu’en réalité :

```text
MemAvailable peut être élevé, car Linux utilise la mémoire libre comme cache.
```

Nous devons donc toujours expliquer les métriques, pas seulement les afficher.


### 13.3.3. Exemple avec la mémoire d’un processus

Un processus peut avoir :

```text
VmSize:  10 Go
VmRSS:   500 Mo
Pss:     200 Mo
```

Sans contexte, ces chiffres sont difficiles à interpréter.

`VmSize` peut être très élevé parce que le processus réserve beaucoup d’espace virtuel.

`VmRSS` indique la mémoire résidente, mais compte aussi la mémoire partagée.

`Pss` donne une estimation plus juste de la contribution réelle du processus.

Nous voyons donc que `/proc` expose des niveaux de détail qui nécessitent une vraie culture système.


## 13.4. Lisibilité limitée pour certains fichiers

### 13.4.1. Tous les fichiers de `/proc` ne sont pas faits pour être lus directement

Certains fichiers sont lisibles facilement :

```bash
cat /proc/uptime
cat /proc/loadavg
cat /proc/meminfo
```

D’autres sont beaucoup moins accessibles :

```bash
cat /proc/stat
cat /proc/vmstat
cat /proc/net/tcp
cat /proc/interrupts
cat /proc/<PID>/stat
cat /proc/<PID>/smaps
```

Ils contiennent des données utiles, mais pas toujours adaptées à une lecture directe.


### 13.4.2. Exemple : `/proc/stat`

`/proc/stat` contient des compteurs CPU cumulés :

```text
cpu  123456 789 34567 9876543 1234 0 567 0 0 0
cpu0 12345 67 3456 987654 123 0 56 0 0 0
```

Ces valeurs ne sont pas des pourcentages instantanés.

Pour obtenir une utilisation CPU, nous devons :

1. lire les compteurs à un instant `t1` ;
    
2. attendre ;
    
3. lire les compteurs à un instant `t2` ;
    
4. calculer les différences ;
    
5. convertir en proportions.
    

C’est ce que font des outils comme :

```bash
top
mpstat
vmstat
```


### 13.4.3. Exemple : `/proc/<PID>/smaps`

`smaps` est très détaillé :

```bash
cat /proc/<PID>/smaps
```

Il peut contenir des milliers de lignes.

Il permet une analyse fine de la mémoire, mais il est difficile à lire directement.

Pour une première synthèse, nous préférons :

```bash
cat /proc/<PID>/smaps_rollup
```

Ou bien :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Anonymous|Swap):' /proc/<PID>/smaps_rollup
```

Nous choisissons donc le bon niveau d’information selon le besoin.


## 13.5. Différences entre systèmes Linux, versions de noyau et distributions

### 13.5.1. `/proc` dépend du noyau

`/proc` est fourni par le noyau Linux.

Son contenu peut varier selon :

- la version du noyau ;
    
- les options de compilation ;
    
- les modules chargés ;
    
- l’architecture matérielle ;
    
- la distribution ;
    
- les patchs appliqués ;
    
- l’environnement d’exécution.
    

Un fichier présent sur une machine peut être absent sur une autre.

Un champ présent dans un noyau récent peut ne pas exister dans un noyau ancien.


### 13.5.2. Exemple : `smaps_rollup`

Le fichier :

```bash
/proc/<PID>/smaps_rollup
```

est très pratique.

Mais il n’existe pas forcément sur tous les noyaux anciens.

Un script robuste doit donc vérifier :

```bash
if [ -r "/proc/$pid/smaps_rollup" ]; then
    cat "/proc/$pid/smaps_rollup"
else
    echo "smaps_rollup indisponible"
fi
```

Nous ne devons pas supposer que toutes les machines exposent exactement les mêmes fichiers.


### 13.5.3. Exemple : différences selon l’architecture

`/proc/cpuinfo` ne présente pas exactement les mêmes champs selon l’architecture.

Sur x86, nous voyons souvent :

```text
model name
cpu cores
siblings
flags
```

Sur ARM, les champs peuvent être différents.

Un script qui cherche uniquement :

```bash
grep "model name" /proc/cpuinfo
```

peut fonctionner sur un PC x86, mais pas forcément sur une machine ARM ou embarquée.


### 13.5.4. Exemple : distributions et sécurité

Certaines distributions activent des paramètres de sécurité plus stricts.

Par exemple :

```bash
cat /proc/sys/kernel/yama/ptrace_scope
```

peut exister sur certaines distributions et pas sur d’autres.

Des options de montage comme `hidepid` peuvent aussi être configurées différemment.

Nous devons donc éviter les conclusions universelles.


## 13.6. Pièges dans les conteneurs

### 13.6.1. `/proc` dépend du contexte

Dans un conteneur, `/proc` ne donne pas forcément la même vue que sur l’hôte.

La vue dépend notamment :

- du namespace PID ;
    
- du namespace réseau ;
    
- du namespace de montage ;
    
- des cgroups ;
    
- des montages masqués ;
    
- des permissions ;
    
- du runtime de conteneur.
    

Nous devons toujours demander :

```text
Où sommes-nous en train de lire /proc ?
Sur l’hôte ?
Dans un conteneur ?
Dans un pod Kubernetes ?
Dans un chroot ?
Dans un namespace spécifique ?
```


### 13.6.2. Exemple : PID 1

Sur une machine classique :

```bash
cat /proc/1/comm
```

peut afficher :

```text
systemd
```

Dans un conteneur, la même commande peut afficher :

```text
node
python
bash
tini
```

Le PID 1 est alors le PID 1 du namespace du conteneur, pas forcément celui de l’hôte.

Nous devons donc éviter de conclure trop vite.


### 13.6.3. Exemple : réseau

Dans un conteneur :

```bash
cat /proc/net/dev
```

peut afficher seulement :

```text
lo
eth0
```

Alors que l’hôte possède :

```text
lo
eno1
docker0
br-...
veth...
wg0
```

`/proc/net` dépend du namespace réseau courant.


### 13.6.4. Exemple : mémoire

Dans certains environnements, `/proc/meminfo` peut donner une vue qui ne correspond pas directement aux limites mémoire du conteneur.

Un conteneur peut être limité à 512 Mio par cgroup, tandis que `/proc/meminfo` semble indiquer plusieurs dizaines de Go disponibles selon la configuration.

Pour analyser un conteneur, nous devons croiser `/proc` avec les cgroups :

```bash
cat /proc/1/cgroup
ls /sys/fs/cgroup
```

Dans Kubernetes, nous devons aussi regarder :

```bash
kubectl describe pod
kubectl top pod
kubectl top node
```


### 13.6.5. Exemple : CPU

De même, `/proc/cpuinfo` peut afficher tous les CPU visibles du nœud, alors que le conteneur est limité à une fraction de CPU.

Un programme peut donc croire qu’il dispose de 16 CPU, alors que Kubernetes lui a attribué une limite de 2 CPU.

Les runtimes modernes essaient parfois de mieux tenir compte des cgroups, mais ce n’est pas universel.

Nous retenons :

```text
Dans un conteneur, /proc donne une vue utile, mais parfois trompeuse.
Les cgroups donnent les limites réellement imposées.
```


## 13.7. Risques de sécurité si `/proc` est mal exposé

### 13.7.1. `/proc` peut révéler trop d’informations

Nous avons vu au chapitre 9 que `/proc` peut exposer :

- arguments de ligne de commande ;
    
- variables d’environnement ;
    
- fichiers ouverts ;
    
- chemins internes ;
    
- bibliothèques chargées ;
    
- sockets ;
    
- informations réseau ;
    
- paramètres noyau ;
    
- structure mémoire.
    

Si ces informations sont accessibles à des utilisateurs non autorisés, elles peuvent faciliter une attaque.


### 13.7.2. Secrets dans `cmdline`

Exemple à éviter :

```bash
backup --password SuperSecret
```

Le secret peut être visible dans :

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

ou :

```bash
ps aux
```


### 13.7.3. Secrets dans `environ`

Exemple :

```bash
DATABASE_URL=postgresql://user:password@host/db
```

Cette variable peut être visible dans :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

selon les permissions.


### 13.7.4. Fichiers ouverts

`/proc/<PID>/fd` peut révéler :

```text
/etc/app/secret.conf
/var/lib/app/private.db
/run/docker.sock
/tmp/session-token
```

Cela peut aider un attaquant à comprendre où chercher.


### 13.7.5. Mauvais montage dans un conteneur

Un conteneur avec un montage trop large peut exposer le `/proc` de l’hôte :

```bash
docker run --privileged -v /proc:/host/proc image
```

Cette pratique est dangereuse.

Elle peut donner au conteneur une visibilité excessive sur l’hôte.

Nous n’utilisons ce type de montage que dans des cas très spécifiques, fortement contrôlés, par exemple certains agents de supervision ou de sécurité, et jamais par défaut.


## 13.8. Comparaison avec `/sys` ou `sysfs`

### 13.8.1. Rôle de `/sys`

`/sys` est le point de montage de `sysfs`.

Il expose une vue structurée des objets du noyau :

```bash
ls /sys
```

Nous y trouvons notamment :

```text
block
bus
class
devices
firmware
fs
kernel
module
power
```

`/sys` est plus orienté vers :

- matériel ;
    
- périphériques ;
    
- pilotes ;
    
- bus ;
    
- classes de périphériques ;
    
- modules noyau ;
    
- attributs structurés d’objets noyau.
    


### 13.8.2. Différence avec `/proc`

Nous pouvons résumer :

```text
/proc : processus, état global, paramètres historiques du noyau
/sys  : modèle matériel, périphériques, objets noyau structurés
```

Exemples :

```bash
cat /proc/cpuinfo
ls /sys/class/net
ls /sys/block
ls /sys/bus/pci/devices
```

Pour les interfaces réseau, `/proc/net/dev` donne des compteurs, tandis que `/sys/class/net` expose des attributs structurés par interface.


### 13.8.3. Pourquoi `/sys` est souvent préférable pour le matériel

Pour écrire un outil qui inspecte les périphériques, `/sys` est souvent plus propre.

Exemple :

```bash
ls /sys/class/net/eth0
```

Nous pouvons y trouver des fichiers comme :

```text
address
carrier
mtu
operstate
speed
statistics
```

C’est plus structuré que certaines sorties historiques de `/proc`.


## 13.9. Comparaison avec les cgroups

### 13.9.1. Rôle des cgroups

Les cgroups, ou control groups, permettent de limiter, mesurer et organiser l’usage des ressources.

Ils concernent notamment :

- CPU ;
    
- mémoire ;
    
- I/O ;
    
- processus ;
    
- pids ;
    
- périphériques ;
    
- pression de ressources.
    

Les cgroups sont fondamentaux pour :

- Docker ;
    
- Kubernetes ;
    
- systemd ;
    
- isolation de services ;
    
- limites de ressources.
    


### 13.9.2. Pourquoi `/proc` ne suffit pas dans les conteneurs

`/proc` décrit souvent le processus et le système visible.

Mais les cgroups décrivent les limites et consommations imposées à un groupe de processus.

Exemple :

```text
/proc/meminfo peut donner une vue large de la mémoire.
/sys/fs/cgroup peut donner la limite mémoire réelle du conteneur.
```

Pour un diagnostic de conteneur, nous devons donc regarder les deux.


### 13.9.3. Exemple avec cgroups v2

Sur un système avec cgroups v2, nous pouvons trouver :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
```

Ces fichiers indiquent notamment :

- la mémoire actuellement consommée par le cgroup ;
    
- la limite mémoire ;
    
- la limite CPU.
    

Cela complète `/proc`.


### 13.9.4. systemd et cgroups

systemd utilise les cgroups pour organiser les services.

Nous pouvons inspecter :

```bash
systemctl status nginx
systemd-cgtop
systemd-cgls
```

Ces outils donnent une vue orientée services, plus utile que de simples PID dans certains contextes.


## 13.10. Comparaison avec eBPF

### 13.10.1. Qu’est-ce qu’eBPF ?

eBPF permet d’exécuter de petits programmes vérifiés dans le noyau, de manière contrôlée, pour observer ou filtrer des événements.

Il permet une observabilité beaucoup plus dynamique que la simple lecture de fichiers dans `/proc`.

Avec eBPF, nous pouvons observer :

- appels système ;
    
- latences ;
    
- paquets réseau ;
    
- événements TCP ;
    
- allocations ;
    
- scheduling ;
    
- ouvertures de fichiers ;
    
- événements de sécurité ;
    
- performances applicatives.
    


### 13.10.2. Ce que `/proc` ne sait pas faire

`/proc` donne surtout des états ou des compteurs.

Il est moins adapté pour répondre à des questions comme :

```text
Quel processus ouvre ce fichier en ce moment ?
Quelle fonction noyau provoque cette latence ?
Quelle requête réseau déclenche ces retransmissions ?
Quel appel système échoue le plus souvent ?
Quelle pile d’appel provoque cette consommation CPU ?
```

eBPF permet de capturer des événements en temps réel et parfois de remonter des stacks.


### 13.10.3. Outils eBPF

Des outils comme ceux de BCC, bpftrace ou des solutions modernes d’observabilité utilisent eBPF.

Exemples de types d’outils :

```text
execsnoop
opensnoop
tcpconnect
tcplife
biolatency
profile
```

Ils permettent de voir des événements que `/proc` ne montre pas directement.


### 13.10.4. Limites d’eBPF

eBPF est puissant, mais plus complexe.

Il demande :

- un noyau compatible ;
    
- des permissions ;
    
- une bonne compréhension du système ;
    
- une attention aux coûts d’observation ;
    
- une maîtrise des outils.
    

Nous ne remplaçons donc pas `/proc` par eBPF. Nous utilisons eBPF lorsque nous avons besoin d’une observation dynamique plus fine.


## 13.11. Comparaison avec `perf`

### 13.11.1. Rôle de `perf`

`perf` est un outil d’analyse de performance bas niveau.

Il permet d’observer :

- consommation CPU ;
    
- événements matériels ;
    
- cycles CPU ;
    
- cache misses ;
    
- branches ;
    
- scheduling ;
    
- stacks d’appels ;
    
- hotspots dans le code.
    

Exemples :

```bash
perf top
perf record
perf report
```


### 13.11.2. Ce que `perf` apporte par rapport à `/proc`

`/proc` peut nous dire :

```text
Ce processus consomme beaucoup de CPU.
```

Mais `perf` peut nous aider à répondre :

```text
Quelle fonction consomme le CPU ?
Quelle pile d’appel revient le plus souvent ?
Le problème vient-il du cache CPU ?
Le programme passe-t-il son temps dans le noyau ou en espace utilisateur ?
```

`perf` descend donc à un niveau d’analyse de performance plus précis.


### 13.11.3. Quand utiliser `perf`

Nous utilisons `perf` lorsque :

- un processus consomme beaucoup de CPU ;
    
- nous devons identifier les fonctions coûteuses ;
    
- nous analysons une régression de performance ;
    
- nous faisons du profilage système ;
    
- nous devons comprendre une latence bas niveau.
    

`/proc` nous aide à détecter un symptôme. `perf` nous aide à expliquer certaines causes profondes.


## 13.12. Comparaison avec systemd

### 13.12.1. systemd comme vue orientée services

`/proc` expose des processus.

systemd expose des unités de service.

Exemple :

```bash
systemctl status nginx
```

donne une vue métier d’administration :

```text
service actif ou non
PID principal
logs récents
cgroup associé
redémarrages
état de l’unité
configuration
```

C’est souvent plus utile que de chercher uniquement dans `/proc`.


### 13.12.2. Exemple

Avec `/proc`, nous pouvons voir :

```bash
cat /proc/1234/comm
cat /proc/1234/status
```

Avec systemd, nous pouvons voir :

```bash
systemctl status nginx
journalctl -u nginx
```

systemd relie le processus à un service, à ses logs, à ses limites et à sa configuration.


### 13.12.3. systemd et limites

systemd peut définir des limites :

```ini
LimitNOFILE=65536
MemoryMax=1G
CPUQuota=200%
```

Ces limites se retrouvent ensuite dans `/proc` ou les cgroups.

Nous devons donc comprendre les deux niveaux :

```text
systemd configure et organise.
/proc et cgroups exposent l’état réel.
```


## 13.13. Comparaison avec Prometheus

### 13.13.1. `/proc` n’historise pas

`/proc` donne une vue instantanée.

Si nous voulons savoir ce qui s’est passé hier à 3 h du matin, `/proc` ne peut pas nous le dire.

Il ne garde pas l’historique des métriques.

Prometheus, au contraire, collecte et stocke des séries temporelles.


### 13.13.2. Node Exporter

Dans un système Linux supervisé, Prometheus utilise souvent un agent comme Node Exporter.

Node Exporter collecte des métriques depuis :

- `/proc` ;
    
- `/sys` ;
    
- statistiques noyau ;
    
- fichiers système ;
    
- parfois d’autres sources.
    

Il expose ensuite ces métriques dans un format exploitable par Prometheus.


### 13.13.3. Ce que Prometheus apporte

Prometheus permet :

- historique ;
    
- graphes ;
    
- alertes ;
    
- agrégation ;
    
- requêtes PromQL ;
    
- comparaison entre machines ;
    
- détection de tendances ;
    
- corrélation temporelle.
    

Exemple de questions que Prometheus peut traiter :

```text
La mémoire disponible baisse-t-elle depuis 6 heures ?
Combien de serveurs ont une charge supérieure à 80 % ?
Ce pic de latence correspond-il à un pic CPU ?
Le nombre de fichiers ouverts augmente-t-il progressivement ?
```

`/proc` donne les valeurs locales. Prometheus les transforme en observabilité exploitable dans le temps.


## 13.14. Comparaison avec Kubernetes

### 13.14.1. Kubernetes ajoute une couche d’abstraction

Dans Kubernetes, nous ne raisonnons pas seulement en processus.

Nous raisonnons en :

- pods ;
    
- containers ;
    
- nodes ;
    
- deployments ;
    
- replicasets ;
    
- services ;
    
- namespaces Kubernetes ;
    
- limites et requests ;
    
- événements ;
    
- probes ;
    
- volumes ;
    
- CNI ;
    
- logs.
    

`/proc` reste présent, mais il est seulement une couche locale.


### 13.14.2. Exemple de problème mémoire

Dans `/proc`, nous pouvons voir qu’un processus consomme de la mémoire.

Mais dans Kubernetes, nous voulons aussi savoir :

```text
Quel pod ?
Quel container ?
Quelle limite mémoire ?
Quelle request ?
Le pod a-t-il été OOMKilled ?
Sur quel nœud ?
Depuis quand ?
Y a-t-il eu un redémarrage ?
```

Nous utilisons alors :

```bash
kubectl top pod
kubectl describe pod
kubectl logs
kubectl get events
```


### 13.14.3. Exemple de problème réseau

`/proc/net/tcp` peut montrer des sockets.

Mais dans Kubernetes, un problème réseau peut venir :

- du pod ;
    
- du service ;
    
- du DNS cluster ;
    
- du CNI ;
    
- d’une NetworkPolicy ;
    
- d’un ingress ;
    
- d’un sidecar ;
    
- d’un proxy ;
    
- d’un problème de nœud.
    

`/proc` est utile dans le conteneur ou sur le nœud, mais il ne suffit pas à comprendre toute la chaîne.


### 13.14.4. Observabilité Kubernetes

Nous complétons avec :

```text
metrics-server
Prometheus
Grafana
Loki
Tempo
OpenTelemetry
kubectl describe
kubectl logs
kubectl top
events Kubernetes
traces applicatives
```

Kubernetes demande une observabilité multi-couches.


## 13.15. Synthèse comparative

|Besoin|`/proc` suffit-il ?|Approche complémentaire|
|---|--:|---|
|Voir la mémoire globale instantanée|Oui|`free`, Prometheus|
|Voir la mémoire d’un processus|Partiellement|`smaps`, runtime, profiler|
|Voir les fichiers ouverts|Oui|`lsof`|
|Voir les sockets|Partiellement|`ss`, `tcpdump`, eBPF|
|Voir les périphériques|Partiellement|`/sys`, `udevadm`, `lsblk`|
|Voir les limites d’un conteneur|Non, pas toujours|cgroups|
|Voir l’historique|Non|Prometheus|
|Voir les causes profondes CPU|Non|`perf`, profilers|
|Voir les événements temps réel|Non|eBPF, auditd, logs|
|Voir l’état d’un service|Partiellement|systemd|
|Voir l’état d’un pod|Non|Kubernetes|


## 13.16. Méthode professionnelle : choisir le bon outil

### 13.16.1. Partir du symptôme

Nous ne choisissons pas un outil au hasard.

Nous partons du symptôme :

```text
CPU élevé
mémoire élevée
disque plein
port occupé
service inaccessible
conteneur OOMKilled
latence réseau
fuite de fichiers ouverts
processus bloqué
```

Puis nous choisissons le niveau d’analyse.


### 13.16.2. Exemple : disque plein

Première analyse :

```bash
df -h
du -h
```

Si les fichiers visibles n’expliquent pas l’espace :

```bash
sudo lsof | grep deleted
```

ou directement :

```bash
sudo find /proc/*/fd -lname '*deleted*' 2>/dev/null
```

Ici, `/proc` est très pertinent.


### 13.16.3. Exemple : CPU élevé

Première analyse :

```bash
top
ps -eo pid,%cpu,comm,args --sort=-%cpu | head
```

Puis :

```bash
cat /proc/<PID>/status
cat /proc/<PID>/stat
```

Si nous devons comprendre quelle fonction consomme le CPU :

```bash
perf top
perf record
```

Ici, `/proc` identifie le processus, mais `perf` explique mieux la cause.


### 13.16.4. Exemple : conteneur tué

Première analyse Kubernetes :

```bash
kubectl describe pod
kubectl logs
kubectl get events
```

Analyse dans le conteneur ou sur le nœud :

```bash
cat /proc/<PID>/status
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
```

Ici, `/proc` seul ne suffit pas. Les cgroups et Kubernetes sont indispensables.


## 13.17. Bonnes pratiques

### 13.17.1. Utiliser `/proc` comme source de vérité locale

`/proc` est excellent pour vérifier un état local :

```bash
cat /proc/<PID>/status
ls -l /proc/<PID>/fd
cat /proc/meminfo
cat /proc/loadavg
```

Nous l’utilisons pour comprendre ce que le noyau voit.


### 13.17.2. Ne pas tout parser soi-même

Si un outil robuste existe, nous l’utilisons.

Exemples :

```bash
ps
top
free
ss
lsof
vmstat
pidstat
systemctl
```

Nous lisons `/proc` directement lorsque nous voulons comprendre, vérifier, automatiser légèrement ou diagnostiquer un cas précis.


### 13.17.3. Tester les scripts sur plusieurs environnements

Un script basé sur `/proc` doit être testé sur :

- machine physique ;
    
- machine virtuelle ;
    
- conteneur ;
    
- noyau ancien ;
    
- noyau récent ;
    
- distribution différente ;
    
- architecture différente si nécessaire.
    


### 13.17.4. Gérer les erreurs normalement

Avec `/proc`, les erreurs sont normales :

```text
Permission denied
No such file or directory
fichier absent
champ absent
processus terminé
```

Un outil robuste doit les gérer sans s’effondrer.


### 13.17.5. Penser sécurité

Nous évitons :

- secrets en ligne de commande ;
    
- secrets en variables d’environnement sans analyse ;
    
- montage du `/proc` de l’hôte dans un conteneur ;
    
- accès non nécessaire à `/proc/<PID>/mem` ;
    
- visibilité excessive sur serveurs multi-utilisateurs.
    

Nous utilisons si nécessaire :

- `hidepid` ;
    
- restrictions `ptrace` ;
    
- permissions strictes ;
    
- cgroups ;
    
- conteneurs non privilégiés ;
    
- politiques de sécurité.
    


## 13.18. Exercices

## Exercice 1 — Identifier une limite de lisibilité

Nous comparons :

```bash
cat /proc/net/tcp
ss -tan
```

Nous répondons :

1. Quelle sortie est la plus lisible ?
    
2. Comment sont représentés les ports dans `/proc/net/tcp` ?
    
3. Comment sont représentés les états TCP ?
    
4. Pourquoi `ss` est-il préférable en diagnostic courant ?
    


## Exercice 2 — Tester la robustesse d’un script

Nous écrivons un script qui parcourt :

```bash
/proc/[0-9]*/status
```

Puis nous le lançons pendant que des processus démarrent et s’arrêtent.

Nous devons gérer :

- processus disparu ;
    
- permission refusée ;
    
- fichier absent.
    

Objectif : produire une sortie sans crash ni messages d’erreur inutiles.


## Exercice 3 — Comparer `/proc` et `/sys`

Nous exécutons :

```bash
cat /proc/net/dev
ls /sys/class/net
ls /sys/class/net/$(ip route | awk '/default/ {print $5; exit}')
```

Nous répondons :

1. Quelles informations trouvons-nous dans `/proc/net/dev` ?
    
2. Quelles informations trouvons-nous dans `/sys/class/net` ?
    
3. Quelle interface est la plus structurée pour décrire le matériel ?
    
4. Quelle interface est la plus directe pour lire les compteurs réseau ?
    


## Exercice 4 — Lire les limites cgroups

Dans un conteneur ou sur une machine utilisant cgroups v2, nous observons :

```bash
cat /proc/1/cgroup
cat /sys/fs/cgroup/memory.current 2>/dev/null
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/cpu.max 2>/dev/null
```

Nous répondons :

1. Quelle mémoire le processus utilise-t-il ?
    
2. Quelle limite mémoire est appliquée ?
    
3. Quelle limite CPU est appliquée ?
    
4. Pourquoi `/proc/meminfo` peut-il être insuffisant ?
    


## Exercice 5 — Comparer service et processus

Nous choisissons un service systemd, par exemple `ssh` ou `nginx`.

Nous exécutons :

```bash
systemctl status ssh
```

ou :

```bash
systemctl status nginx
```

Puis nous récupérons son PID et nous observons :

```bash
cat /proc/<PID>/status
cat /proc/<PID>/limits
ls -l /proc/<PID>/fd
```

Nous répondons :

1. Que donne systemd que `/proc` ne donne pas directement ?
    
2. Que donne `/proc` que systemd ne détaille pas ?
    
3. Pourquoi les deux vues sont-elles complémentaires ?
    


## Exercice 6 — Observer l’absence d’historique

Nous lisons :

```bash
cat /proc/loadavg
```

Puis nous attendons quelques minutes et nous relisons.

Nous répondons :

1. `/proc` permet-il de savoir quelle était la charge il y a une heure ?
    
2. Quel outil ou système faudrait-il pour historiser ces valeurs ?
    
3. Pourquoi Prometheus ou un autre système de métriques devient-il nécessaire en production ?
    


## 13.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc` est une interface puissante, mais brute.
    
2. Certains fichiers sont faciles à lire, d’autres sont difficiles à parser.
    
3. Les formats ne doivent pas être supposés universels.
    
4. Les informations de `/proc` sont souvent bas niveau et demandent une interprétation.
    
5. `/proc` donne surtout une vue locale et instantanée.
    
6. Les versions de noyau, distributions et architectures peuvent modifier les fichiers disponibles.
    
7. Dans les conteneurs, `/proc` peut être partiel ou trompeur.
    
8. Les cgroups sont indispensables pour comprendre les limites réelles des conteneurs.
    
9. Une mauvaise exposition de `/proc` peut créer des risques de sécurité.
    
10. `/sys` complète `/proc` pour les périphériques et objets noyau.
    
11. eBPF permet une observation dynamique d’événements que `/proc` ne montre pas directement.
    
12. `perf` est plus adapté à l’analyse fine des performances CPU.
    
13. systemd donne une vue orientée services.
    
14. Prometheus ajoute l’historique, l’agrégation et l’alerte.
    
15. Kubernetes ajoute une couche d’analyse orientée pods, containers, événements et orchestration.
    
16. Nous devons choisir l’outil selon le symptôme, le contexte et le niveau d’analyse nécessaire.
    


## Conclusion du chapitre 13

Nous avons étudié les limites de `/proc`.

Nous savons maintenant que `/proc` est une interface fondamentale pour comprendre Linux, mais qu’elle ne constitue pas à elle seule une solution complète d’administration, de diagnostic ou d’observabilité.

`/proc` nous donne une vue locale, instantanée et souvent bas niveau du système. Cette vue est précieuse, mais elle doit être complétée par d’autres interfaces et outils : `/sys` pour les périphériques, les cgroups pour les limites de ressources, eBPF pour les événements dynamiques, `perf` pour le profilage, systemd pour la vue service, Prometheus pour l’historisation, et Kubernetes pour l’orchestration.

Nous retenons donc une approche professionnelle : nous utilisons `/proc` comme fondation, mais nous ne nous y limitons pas. Nous choisissons l’outil adapté au problème, au contexte d’exécution et au niveau de précision attendu.


# 14. Évaluation proposée

## Partie théorique

Nous vérifions que nous savons expliquer :

- ce qu’est un pseudo-système de fichiers
    
- pourquoi `/proc` n’est pas stocké sur disque
    
- le rôle de `/proc/<PID>`
    
- la différence entre `/proc` et `/sys`
    
- les implications de sécurité de `/proc`
    
- le rôle de `/proc/sys`
    

## Partie pratique

Nous demandons aux étudiants de produire un mini-rapport de diagnostic à partir d’une machine Linux.

Le rapport doit contenir :

- état CPU
    
- état mémoire
    
- uptime
    
- processus principaux
    
- fichiers ouverts par un processus donné
    
- limites de ressources
    
- paramètres noyau observés
    
- conclusion technique
    

## Projet court

Nous développons un outil simple, en Bash ou Python, qui lit `/proc` et produit un tableau de synthèse du système.

Exemple de sortie attendue :

```text
Hostname        : serveur-test
Kernel          : Linux 6.x
Uptime          : 3 jours, 4 heures
CPU logiques    : 8
Mémoire totale  : 16 Go
Mémoire dispo   : 9 Go
Processus       : 243
Load average    : 0.42 0.51 0.48
```


# Conclusion du cours

Nous retenons que `/proc` est une interface fondamentale entre l’espace utilisateur et le noyau Linux.

Nous l’utilisons pour comprendre les processus, la mémoire, le réseau, les paramètres noyau et l’état global du système.

Nous voyons aussi que `/proc` est à la fois un outil pédagogique, un outil d’administration et un outil de diagnostic avancé. Pour un informaticien de niveau Master 2, sa maîtrise permet de mieux comprendre Linux en profondeur, au-delà des commandes haut niveau.

