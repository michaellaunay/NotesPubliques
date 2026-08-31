---
schema_version: 1
uid: 01M1BQ629J6G3E22XGSSJA2FH6
titre: "proc — 01 — Introduction générale à proc"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - gnu-linux
  - noyau
  - procfs
resume: "Chapitre 1 sur 11 du livre « proc » : Introduction générale à /proc. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2026-05-24
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « proc » — chapitre 1/11
> [[proc — Sommaire|Sommaire]] · [[proc — Sommaire|← Sommaire]] · [[proc — 02 — Architecture interne de proc|02 — Architecture interne de proc →]]

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

---
> [!info] Livre « proc » — chapitre 1/11
> [[proc — Sommaire|Sommaire]] · [[proc — Sommaire|← Sommaire]] · [[proc — 02 — Architecture interne de proc|02 — Architecture interne de proc →]]
