---
schema_version: 1
uid: 01M1BQ629KJFJGSQ7AN18FZ6K4
titre: "proc — 02 — Architecture interne de proc"
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
resume: "Chapitre 2 sur 11 du livre « proc » : Architecture interne de /proc. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 2/11
> [[proc — Sommaire|Sommaire]] · [[proc — 01 — Introduction générale à proc|← 01 — Introduction générale à proc]] · [[proc — 03 — Informations globales sur le système|03 — Informations globales sur le système →]]

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

### Exercice 1 — Identifier la nature de `/proc`

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

### Exercice 2 — Observer la taille apparente des fichiers

Nous exécutons :

```bash
ls -lh /proc/cpuinfo /proc/meminfo /proc/uptime
wc -c /proc/cpuinfo /proc/meminfo /proc/uptime
```

Nous comparons la taille affichée par `ls` et le nombre d’octets lus par `wc`.

Nous expliquons la différence.


### Exercice 3 — Observer le caractère dynamique

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


### Exercice 4 — Observer un processus temporaire

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


### Exercice 5 — Comprendre `/proc/self`

Nous exécutons :

```bash
echo $$
readlink /proc/self
bash -c 'echo "PID bash interne: $$"; readlink /proc/self'
```

Nous expliquons pourquoi `/proc/self` ne pointe pas toujours vers le PID que nous imaginons au départ.


### Exercice 6 — Écrire un mini-script robuste

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

---
> [!info] Livre « proc » — chapitre 2/11
> [[proc — Sommaire|Sommaire]] · [[proc — 01 — Introduction générale à proc|← 01 — Introduction générale à proc]] · [[proc — 03 — Informations globales sur le système|03 — Informations globales sur le système →]]
