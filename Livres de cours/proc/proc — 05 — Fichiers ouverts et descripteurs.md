---
schema_version: 1
uid: 01M1BQ629QNYTEHFJ9Q3HNGAB8
titre: "proc — 05 — Fichiers ouverts et descripteurs"
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
resume: "Chapitre 5 sur 11 du livre « proc » : Fichiers ouverts et descripteurs. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 5/11
> [[proc — Sommaire|Sommaire]] · [[proc — 04 — proc et les processus|← 04 — proc et les processus]] · [[proc — 06 — Mémoire d’un processus|06 — Mémoire d’un processus →]]

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

### Exercice 1 — Observer les descripteurs du shell courant

Nous exécutons :

```bash
ls -l /proc/$$/fd
```

Nous répondons aux questions :

1. Vers quoi pointent les descripteurs `0`, `1` et `2` ?
    
2. Sont-ils attachés à un terminal ?
    
3. Quel est le chemin du terminal ?
    
4. Que se passe-t-il si nous redirigeons la sortie d’une commande vers un fichier ?
    


### Exercice 2 — Observer les redirections

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


### Exercice 3 — Fichier supprimé mais ouvert

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

### Exercice 4 — Compter les descripteurs ouverts

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
    


### Exercice 5 — Comparer `/proc` et `lsof`

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


### 5.19. Ce que nous devons retenir

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

---
> [!info] Livre « proc » — chapitre 5/11
> [[proc — Sommaire|Sommaire]] · [[proc — 04 — proc et les processus|← 04 — proc et les processus]] · [[proc — 06 — Mémoire d’un processus|06 — Mémoire d’un processus →]]
