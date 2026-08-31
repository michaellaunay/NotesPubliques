---
schema_version: 1
uid: 01M1BQ629SZ6SB6FC38WPQH9A1
titre: "proc — 07 — Réseau et proc"
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
resume: "Chapitre 7 sur 11 du livre « proc » : Réseau et /proc. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 7/11
> [[proc — Sommaire|Sommaire]] · [[proc — 06 — Mémoire d’un processus|← 06 — Mémoire d’un processus]] · [[proc — 08 — Paramètres noyau avec proc sys|08 — Paramètres noyau avec proc sys →]]

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

### Exercice 1 — Observer les interfaces réseau

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
    


### Exercice 2 — Observer les sockets TCP

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
    


### Exercice 3 — Lancer un serveur local et le retrouver

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


### Exercice 4 — Observer les sockets Unix

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
    


### Exercice 5 — Lire la table ARP

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
    


### Exercice 6 — Lire les routes

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

---
> [!info] Livre « proc » — chapitre 7/11
> [[proc — Sommaire|Sommaire]] · [[proc — 06 — Mémoire d’un processus|← 06 — Mémoire d’un processus]] · [[proc — 08 — Paramètres noyau avec proc sys|08 — Paramètres noyau avec proc sys →]]
