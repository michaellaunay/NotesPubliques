---
schema_version: 1
uid: 01M1BQ624D10C9382AQCX77BKZ
titre: "Les namespaces Linux — 07 — Namespace network"
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
  - namespaces
  - conteneurisation
resume: "Chapitre 7 sur 16 du livre « Les namespaces Linux » : Namespace network. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 7/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 06 — Namespace mount|← 06 — Namespace mount]] · [[Les namespaces Linux — 08 — Namespace IPC|08 — Namespace IPC →]]

# Chapitre 7 — Namespace network
## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace network, ou namespace réseau.

Le namespace réseau permet à un processus de disposer de sa propre pile réseau. Cela signifie qu’il peut avoir ses propres interfaces, ses propres adresses IP, ses propres routes, ses propres sockets, ses propres ports en écoute et ses propres règles de filtrage réseau.

C’est un mécanisme fondamental des conteneurs. Sans namespace réseau, les conteneurs partageraient directement les interfaces et les ports de l’hôte.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace réseau ;
    
- créer un namespace réseau avec `unshare` ;
    
- créer un namespace réseau nommé avec `ip netns` ;
    
- comprendre pourquoi l’interface `lo` est isolée ;
    
- connecter un namespace réseau à l’hôte avec une paire `veth` ;
    
- configurer des adresses IP et des routes ;
    
- comprendre le rôle d’un bridge Linux ;
    
- comprendre le principe du NAT ;
    
- faire le lien avec Docker bridge ;
    
- faire le lien avec Kubernetes et les plugins CNI ;
    
- identifier les risques de sécurité liés au partage du réseau de l’hôte.
    

---

## 7.1. Pourquoi isoler le réseau ?

## 7.1.1. Le problème de départ

Sur une machine Linux classique, les processus utilisent la pile réseau de l’hôte.

Ils partagent :

```text
interfaces réseau
adresses IP
routes
ports en écoute
sockets
règles de pare-feu
table ARP
configuration IPv6
```

Cela signifie que deux processus qui veulent écouter sur le même port de la même adresse ne peuvent pas le faire en même temps.

Exemple :

```bash
python3 -m http.server 8080
```

Si un autre processus essaie d’écouter aussi sur `0.0.0.0:8080`, il obtient généralement une erreur :

```text
Address already in use
```

Avec des namespaces réseau séparés, deux processus peuvent écouter sur le même port, car ils n’appartiennent pas à la même pile réseau.

---

## 7.1.2. Ce que nous voulons obtenir

Dans un environnement conteneurisé, nous voulons souvent que chaque conteneur ait :

- ses propres interfaces ;
    
- sa propre adresse IP ;
    
- sa propre table de routage ;
    
- ses propres ports ;
    
- ses propres sockets ;
    
- ses propres règles réseau ;
    
- éventuellement sa propre pile loopback.
    

Ainsi, un conteneur peut écouter sur le port `80` sans empêcher un autre conteneur d’écouter lui aussi sur son propre port `80`.

Nous obtenons cette isolation grâce au namespace network.

---

## 7.2. Qu’est-ce que le namespace network ?

## 7.2.1. Définition

Un namespace network isole la pile réseau visible par un groupe de processus.

Il isole notamment :

```text
interfaces réseau
adresses IP
routes
sockets TCP/UDP
ports en écoute
règles netfilter/nftables selon contexte
table ARP/voisinage
/proc/net
/sys/class/net
```

Un processus dans un namespace réseau ne voit pas forcément les interfaces réseau de l’hôte.

Il voit seulement les interfaces qui appartiennent à son namespace.

---

## 7.2.2. Exemple conceptuel

Sur l’hôte, nous avons :

```text
lo
eth0
docker0
wlan0
```

Dans un namespace réseau isolé, nous pouvons avoir seulement :

```text
lo
```

Puis, si nous ajoutons une interface virtuelle, nous pouvons avoir :

```text
lo
veth-ns
```

Ce namespace a alors sa propre mini-pile réseau.

---

## 7.2.3. Même port, namespaces différents

Deux processus peuvent écouter sur le même port s’ils sont dans deux namespaces réseau différents.

Exemple conceptuel :

```text
Namespace réseau A :
  nginx écoute sur 0.0.0.0:80

Namespace réseau B :
  nginx écoute aussi sur 0.0.0.0:80
```

Il n’y a pas de conflit, car les sockets appartiennent à deux piles réseau séparées.

C’est l’un des principes qui rendent les conteneurs pratiques.

---

## 7.3. Observer le namespace réseau courant

## 7.3.1. Avec `/proc/<PID>/ns/net`

Nous pouvons observer le namespace réseau de notre shell courant :

```bash
readlink /proc/$$/ns/net
```

Exemple :

```text
net:
```

Nous pouvons comparer avec le PID 1 :

```bash
readlink /proc/1/ns/net
```

Si les valeurs sont identiques, notre shell partage le namespace réseau du PID 1.

---

## 7.3.2. Observer les interfaces

Nous affichons les interfaces réseau visibles :

```bash
ip addr
```

ou :

```bash
ip link
```

Nous pouvons aussi regarder :

```bash
ls /sys/class/net
```

et :

```bash
cat /proc/net/dev
```

Ces chemins dépendent du namespace réseau courant.

Dans un namespace réseau différent, leur contenu peut changer.

---

## 7.3.3. Observer les routes

Nous affichons la table de routage :

```bash
ip route
```

Dans le namespace réseau de l’hôte, nous voyons souvent une route par défaut :

```text
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10
```

Dans un namespace réseau nouvellement créé, nous pouvons ne voir aucune route.

---

## 7.3.4. Observer les sockets

Nous pouvons afficher les sockets TCP et UDP :

```bash
ss -tulpen
```

ou :

```bash
ss -tan
```

Le résultat dépend du namespace réseau courant.

Un port en écoute dans l’hôte n’apparaît pas forcément dans un namespace réseau isolé.

---

## 7.4. Créer un namespace réseau avec `unshare`

## 7.4.1. Première expérience

Nous créons un nouveau namespace réseau :

```bash
unshare --net bash
```

Dans ce shell, nous observons :

```bash
readlink /proc/$$/ns/net
ip addr
ip route
ss -tulpen
```

Nous constatons généralement que :

- le namespace réseau est différent ;
    
- seule l’interface `lo` existe ;
    
- `lo` peut être désactivée ;
    
- aucune route par défaut n’est configurée ;
    
- aucune connexion réseau extérieure n’est disponible.
    

---

## 7.4.2. Activer l’interface loopback

Dans un namespace réseau neuf, l’interface loopback existe, mais elle peut être désactivée.

Nous vérifions :

```bash
ip link
```

Nous pouvons voir :

```text
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
```

Nous l’activons :

```bash
ip link set lo up
```

Puis :

```bash
ip addr
```

Nous voyons généralement :

```text
lo    127.0.0.1/8
```

Le loopback est maintenant actif dans ce namespace.

---

## 7.4.3. Tester le loopback

Nous pouvons lancer un serveur local :

```bash
python3 -m http.server 8080 &
```

Puis tester :

```bash
curl http://127.0.0.1:8080
```

Cela fonctionne dans le namespace si `lo` est actif.

Mais depuis l’hôte, ce serveur n’est pas accessible directement sur `127.0.0.1:8080`, car le loopback du namespace est séparé du loopback de l’hôte.

Nous retenons :

```text
Chaque namespace réseau possède son propre loopback.
127.0.0.1 ne désigne pas toujours la même pile réseau.
```

---

## 7.5. Namespace réseau nommé avec `ip netns`

## 7.5.1. Pourquoi utiliser `ip netns` ?

`unshare --net` est pratique pour lancer un shell isolé.

Mais pour construire des laboratoires réseau, nous préférons souvent créer des namespaces réseau nommés.

Nous utilisons :

```bash
sudo ip netns add ns1
```

Puis :

```bash
ip netns list
```

Sortie :

```text
ns1
```

---

## 7.5.2. Exécuter une commande dans un namespace nommé

Nous exécutons :

```bash
sudo ip netns exec ns1 ip addr
```

Nous pouvons activer `lo` :

```bash
sudo ip netns exec ns1 ip link set lo up
```

Puis vérifier :

```bash
sudo ip netns exec ns1 ip addr
```

---

## 7.5.3. Où est stockée la référence ?

Les namespaces réseau nommés sont généralement référencés dans :

```bash
/run/netns
```

Nous pouvons vérifier :

```bash
ls -l /run/netns
```

Le fichier `ns1` n’est pas un fichier ordinaire au sens classique. Il sert de référence vers le namespace réseau.

Tant que cette référence existe, le namespace peut rester vivant même sans processus permanent à l’intérieur.

---

## 7.5.4. Supprimer le namespace

Nous supprimons :

```bash
sudo ip netns delete ns1
```

Puis :

```bash
ip netns list
```

Le namespace disparaît si plus aucune référence ne le maintient vivant.

---

## 7.6. Connecter un namespace réseau à l’hôte avec une paire `veth`

## 7.6.1. Principe d’une paire `veth`

Une paire `veth` fonctionne comme un câble Ethernet virtuel.

Elle possède deux extrémités :

```text
veth-host  <── lien virtuel ──>  veth-ns
```

Ce qui entre par une extrémité ressort par l’autre.

Nous plaçons une extrémité dans le namespace de l’hôte et l’autre dans le namespace isolé.

---

## 7.6.2. Créer le namespace

Nous créons un namespace réseau nommé :

```bash
sudo ip netns add ns1
```

Nous activons le loopback :

```bash
sudo ip netns exec ns1 ip link set lo up
```

---

## 7.6.3. Créer la paire `veth`

Nous créons la paire :

```bash
sudo ip link add veth-host type veth peer name veth-ns
```

Nous vérifions côté hôte :

```bash
ip link show veth-host
ip link show veth-ns
```

Pour l’instant, les deux interfaces sont dans le namespace réseau de l’hôte.

---

## 7.6.4. Déplacer une extrémité dans le namespace

Nous déplaçons `veth-ns` dans `ns1` :

```bash
sudo ip link set veth-ns netns ns1
```

Maintenant :

- `veth-host` reste côté hôte ;
    
- `veth-ns` est dans le namespace `ns1`.
    

Nous vérifions :

```bash
ip link show veth-host
sudo ip netns exec ns1 ip link show veth-ns
```

---

## 7.6.5. Configurer les adresses IP

Côté hôte :

```bash
sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up
```

Côté namespace :

```bash
sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
```

Nous vérifions :

```bash
ip addr show veth-host
sudo ip netns exec ns1 ip addr show veth-ns
```

---

## 7.6.6. Tester la connectivité

Depuis l’hôte :

```bash
ping -c 3 10.0.0.2
```

Depuis le namespace :

```bash
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

Si tout est correct, les deux côtés communiquent.

Nous avons créé un lien réseau entre l’hôte et un namespace isolé.

---

## 7.6.7. Nettoyage

Nous supprimons le namespace :

```bash
sudo ip netns delete ns1
```

Puis, si l’interface côté hôte existe encore :

```bash
sudo ip link delete veth-host 2>/dev/null
```

Quand nous supprimons une extrémité d’une paire `veth`, l’autre disparaît généralement aussi.

---

## 7.7. Ajouter une route par défaut

## 7.7.1. Situation actuelle

Après la configuration précédente, le namespace `ns1` peut joindre l’hôte via `10.0.0.1`.

Mais il ne sait pas forcément joindre Internet.

Dans `ns1`, nous vérifions :

```bash
sudo ip netns exec ns1 ip route
```

Nous voyons probablement :

```text
10.0.0.0/24 dev veth-ns proto kernel scope link src 10.0.0.2
```

Il manque une route par défaut.

---

## 7.7.2. Ajouter une route par défaut

Nous ajoutons dans `ns1` :

```bash
sudo ip netns exec ns1 ip route add default via 10.0.0.1
```

Nous vérifions :

```bash
sudo ip netns exec ns1 ip route
```

Nous voyons :

```text
default via 10.0.0.1 dev veth-ns
10.0.0.0/24 dev veth-ns proto kernel scope link src 10.0.0.2
```

Le namespace envoie maintenant son trafic externe vers l’hôte.

---

## 7.7.3. Pourquoi cela ne suffit pas toujours ?

Même avec une route par défaut, l’accès Internet peut ne pas fonctionner.

Il faut aussi que l’hôte :

- accepte de router les paquets ;
    
- fasse éventuellement du NAT ;
    
- autorise le trafic dans son pare-feu ;
    
- ait une route retour ou traduise les adresses.
    

Nous devons donc configurer le forwarding et le NAT si nous voulons sortir vers Internet.

---

## 7.8. Activer le forwarding IPv4

## 7.8.1. Lire l’état actuel

Sur l’hôte :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Valeurs :

```text
0 → forwarding désactivé
1 → forwarding activé
```

Nous pouvons aussi utiliser :

```bash
sysctl net.ipv4.ip_forward
```

---

## 7.8.2. Activation temporaire

Pour activer temporairement :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

ou :

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

Cette modification est active immédiatement, mais elle n’est pas forcément persistante après redémarrage.

---

## 7.8.3. Persistance

Pour rendre la configuration persistante, nous pouvons créer un fichier :

```bash
sudo nano /etc/sysctl.d/99-forwarding.conf
```

Contenu :

```conf
net.ipv4.ip_forward = 1
```

Puis appliquer :

```bash
sudo sysctl --system
```

Dans un TP, nous pouvons nous contenter d’une activation temporaire.

---

## 7.9. NAT avec nftables ou iptables

## 7.9.1. Pourquoi avons-nous besoin de NAT ?

Notre namespace `ns1` utilise l’adresse :

```text
10.0.0.2
```

Si nous envoyons un paquet vers Internet, la machine distante ne sait probablement pas retourner vers `10.0.0.2`, car c’est une adresse privée de notre laboratoire local.

Nous avons donc besoin de traduire l’adresse source en adresse de l’hôte.

C’est le rôle du NAT, souvent sous forme de masquerading.

---

## 7.9.2. Exemple avec iptables

Supposons que l’interface de sortie de l’hôte soit `eth0`.

Nous pouvons faire :

```bash
sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
```

Puis dans le namespace :

```bash
sudo ip netns exec ns1 ping -c 3 1.1.1.1
```

Si la route, le forwarding et le NAT sont corrects, le ping peut fonctionner.

Pour DNS, il faut aussi une configuration de résolution de noms.

---

## 7.9.3. Exemple avec nftables

Sur les systèmes modernes, `nftables` remplace souvent `iptables`.

Exemple indicatif :

```bash
sudo nft add table ip nat
sudo nft add chain ip nat postrouting '{ type nat hook postrouting priority 100 ; }'
sudo nft add rule ip nat postrouting ip saddr 10.0.0.0/24 oifname "eth0" masquerade
```

La syntaxe exacte dépend de la configuration existante du système.

En cours, nous insistons surtout sur le principe :

```text
route par défaut dans le namespace
+ forwarding sur l’hôte
+ NAT sur l’hôte
= accès possible vers l’extérieur
```

---

## 7.9.4. Nettoyage des règles

En TP, nous devons nettoyer les règles ajoutées.

Avec `iptables`, nous pouvons supprimer la règle si nous la connaissons :

```bash
sudo iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
```

Avec `nftables`, nous devons supprimer la règle ou la table créée.

Nous devons éviter de laisser des règles réseau de test sur une machine de production.

---

## 7.10. DNS dans un namespace réseau

## 7.10.1. Ping IP et résolution DNS

Si ceci fonctionne :

```bash
sudo ip netns exec ns1 ping -c 3 1.1.1.1
```

mais ceci échoue :

```bash
sudo ip netns exec ns1 ping -c 3 example.org
```

le problème est probablement lié au DNS.

Le namespace réseau isole la pile réseau, mais le fichier de configuration DNS dépend aussi de la vue du système de fichiers.

Nous devons regarder :

```bash
cat /etc/resolv.conf
```

dans l’environnement concerné.

---

## 7.10.2. Avec `ip netns exec`

Quand nous utilisons `ip netns exec`, la commande s’exécute dans le namespace réseau, mais elle garde souvent la même vue de fichiers que l’hôte.

Donc `/etc/resolv.conf` peut rester celui de l’hôte.

Cependant, certaines distributions ou outils peuvent gérer des fichiers par namespace, par exemple sous :

```text
/etc/netns/<nom>/resolv.conf
```

Pour un namespace nommé `ns1`, nous pouvons créer :

```bash
sudo mkdir -p /etc/netns/ns1
echo "nameserver 1.1.1.1" | sudo tee /etc/netns/ns1/resolv.conf
```

Puis :

```bash
sudo ip netns exec ns1 ping -c 3 example.org
```

Selon la configuration, `ip netns exec` peut utiliser ce fichier.

---

## 7.11. Bridge Linux

## 7.11.1. Pourquoi utiliser un bridge ?

Si nous avons plusieurs namespaces réseau, nous pouvons connecter chacun à l’hôte avec une paire `veth`.

Mais pour connecter plusieurs namespaces entre eux proprement, nous utilisons souvent un bridge Linux.

Un bridge fonctionne comme un switch virtuel.

Schéma conceptuel :

```text
          bridge br0
          /       \
   veth-ns1     veth-ns2
      |            |
     ns1          ns2
```

Chaque namespace est connecté au bridge par une paire `veth`.

---

## 7.11.2. Créer un bridge

Nous créons le bridge :

```bash
sudo ip link add br0 type bridge
sudo ip addr add 10.10.0.1/24 dev br0
sudo ip link set br0 up
```

Le bridge a l’adresse `10.10.0.1`.

---

## 7.11.3. Connecter un namespace au bridge

Nous créons un namespace :

```bash
sudo ip netns add ns1
```

Nous créons une paire `veth` :

```bash
sudo ip link add veth1-host type veth peer name veth1-ns
```

Nous plaçons une extrémité dans le namespace :

```bash
sudo ip link set veth1-ns netns ns1
```

Nous connectons l’autre au bridge :

```bash
sudo ip link set veth1-host master br0
sudo ip link set veth1-host up
```

Nous configurons le namespace :

```bash
sudo ip netns exec ns1 ip addr add 10.10.0.2/24 dev veth1-ns
sudo ip netns exec ns1 ip link set veth1-ns up
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns1 ip route add default via 10.10.0.1
```

---

## 7.11.4. Ajouter un second namespace

Nous répétons avec `ns2` :

```bash
sudo ip netns add ns2
sudo ip link add veth2-host type veth peer name veth2-ns
sudo ip link set veth2-ns netns ns2

sudo ip link set veth2-host master br0
sudo ip link set veth2-host up

sudo ip netns exec ns2 ip addr add 10.10.0.3/24 dev veth2-ns
sudo ip netns exec ns2 ip link set veth2-ns up
sudo ip netns exec ns2 ip link set lo up
sudo ip netns exec ns2 ip route add default via 10.10.0.1
```

Nous testons :

```bash
sudo ip netns exec ns1 ping -c 3 10.10.0.3
sudo ip netns exec ns2 ping -c 3 10.10.0.2
```

Les deux namespaces communiquent via le bridge.

---

## 7.11.5. Nettoyage du bridge

Nous supprimons :

```bash
sudo ip netns delete ns1
sudo ip netns delete ns2
sudo ip link delete br0
```

Si des interfaces restent :

```bash
ip link | grep veth
```

Nous pouvons supprimer les interfaces résiduelles avec prudence.

---

## 7.12. Lien avec Docker bridge

## 7.12.1. Le réseau bridge par défaut

Docker crée souvent un bridge nommé :

```text
docker0
```

Nous pouvons l’observer :

```bash
ip addr show docker0
```

ou :

```bash
ip link show docker0
```

Ce bridge connecte les conteneurs au réseau de l’hôte.

Chaque conteneur reçoit généralement une interface virtuelle connectée à ce bridge.

---

## 7.12.2. Observer un conteneur Docker

Nous lançons :

```bash
docker run --rm -d --name net-demo alpine sleep 1000
```

Nous récupérons le PID hôte :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' net-demo)
```

Nous observons le namespace réseau :

```bash
sudo readlink /proc/$pid/ns/net
readlink /proc/1/ns/net
```

Nous entrons dans le namespace réseau du conteneur :

```bash
sudo nsenter --target "$pid" --net ip addr
```

Nous voyons généralement :

```text
lo
eth0
```

`eth0` dans le conteneur est souvent une extrémité de paire `veth`.

---

## 7.12.3. Ports publiés

Quand nous faisons :

```bash
docker run -p 8080:80 image
```

Docker configure une redirection entre :

```text
port 8080 de l’hôte
port 80 du conteneur
```

Cela peut passer par des règles NAT, du proxying ou une combinaison selon la version et la configuration.

Le point important est :

```text
Le conteneur peut écouter sur 80 dans son namespace.
L’hôte expose éventuellement 8080 vers ce conteneur.
```

---

## 7.12.4. Mode host network

Docker permet :

```bash
docker run --network=host image
```

Dans ce cas, le conteneur partage le namespace réseau de l’hôte.

Conséquences :

- il voit les interfaces de l’hôte ;
    
- il partage les ports de l’hôte ;
    
- il n’a pas d’isolation réseau classique ;
    
- un service dans le conteneur peut écouter directement sur les adresses de l’hôte.
    

Cette option est puissante, mais réduit fortement l’isolation.

---

## 7.13. Lien avec Kubernetes et CNI

## 7.13.1. Le modèle réseau Kubernetes

Dans Kubernetes, chaque pod possède généralement sa propre adresse IP.

Tous les conteneurs d’un même pod partagent le même namespace réseau.

Cela signifie que deux conteneurs dans le même pod peuvent communiquer via :

```text
localhost
127.0.0.1
```

Exemple :

```text
Pod
├── container app
└── container sidecar

Les deux partagent le même namespace réseau.
```

Si `app` écoute sur `127.0.0.1:8080`, le sidecar peut souvent l’atteindre sur `localhost:8080`.

---

## 7.13.2. Pause container

Kubernetes utilise généralement un conteneur spécial, souvent appelé pause container, pour maintenir les namespaces du pod.

Le namespace réseau du pod est souvent attaché à ce conteneur pause.

Les autres conteneurs du pod rejoignent ensuite ce namespace réseau.

Conceptuellement :

```text
pause container
  détient le namespace réseau du pod

app container
  rejoint ce namespace réseau

sidecar container
  rejoint ce même namespace réseau
```

Cela explique pourquoi les conteneurs d’un même pod partagent la même IP.

---

## 7.13.3. CNI

Kubernetes délègue la configuration réseau à des plugins CNI, Container Network Interface.

Exemples de solutions CNI :

```text
Calico
Cilium
Flannel
Weave Net
Canal
Antrea
```

Le plugin CNI configure notamment :

- interfaces réseau du pod ;
    
- paires `veth` ;
    
- routes ;
    
- bridges éventuels ;
    
- règles eBPF ou iptables ;
    
- politiques réseau ;
    
- connectivité entre nœuds.
    

Le namespace réseau est la brique noyau. Le CNI est la couche d’intégration Kubernetes.

---

## 7.13.4. hostNetwork

Kubernetes permet :

```yaml
hostNetwork: true
```

Dans ce cas, le pod partage le namespace réseau de l’hôte.

Conséquences :

- le pod voit les interfaces de l’hôte ;
    
- il partage les ports de l’hôte ;
    
- il n’a pas son IP de pod classique ;
    
- les conflits de ports sont possibles avec les services de l’hôte ;
    
- l’isolation réseau est réduite.
    

Nous n’utilisons pas `hostNetwork` sans raison précise.

---

## 7.14. `/proc/net` et namespace réseau

## 7.14.1. `/proc/net` dépend du namespace courant

Nous avons vu dans le cours sur `/proc` que :

```bash
cat /proc/net/dev
cat /proc/net/tcp
cat /proc/net/udp
cat /proc/net/unix
```

affichent des informations réseau.

Ces informations dépendent du namespace réseau courant.

Dans un namespace isolé :

```bash
sudo ip netns exec ns1 cat /proc/net/dev
```

nous ne voyons que les interfaces de `ns1`.

---

## 7.14.2. Comparer hôte et namespace

Sur l’hôte :

```bash
cat /proc/net/dev
```

Dans `ns1` :

```bash
sudo ip netns exec ns1 cat /proc/net/dev
```

Nous comparons les interfaces.

Nous pouvons aussi comparer les sockets :

```bash
ss -tulpen
sudo ip netns exec ns1 ss -tulpen
```

Les ports visibles ne sont pas les mêmes.

---

## 7.14.3. Même chemin, contenu différent

Le chemin :

```text
/proc/net/dev
```

est le même, mais son contenu dépend du namespace réseau du processus qui le lit.

C’est une idée importante :

```text
même chemin
namespace différent
contenu différent
```

Comme pour `/proc/sys/kernel/hostname` avec le namespace UTS, `/proc/net` illustre parfaitement la logique des namespaces.

---

## 7.15. Pare-feu et règles réseau

## 7.15.1. Netfilter et namespaces

Linux utilise netfilter pour le filtrage, le NAT et d’autres traitements réseau.

Les règles peuvent dépendre du namespace réseau.

Chaque namespace réseau possède sa propre pile réseau, et donc sa propre vue de certaines règles et tables.

Nous pouvons tester dans un namespace :

```bash
sudo ip netns exec ns1 nft list ruleset
```

ou, si `iptables` est utilisé :

```bash
sudo ip netns exec ns1 iptables -L
```

Selon les droits et l’outillage disponible, les résultats peuvent varier.

---

## 7.15.2. Pare-feu sur l’hôte

Même si un namespace a ses propres règles, le trafic passe souvent par l’hôte pour sortir.

Le pare-feu de l’hôte peut donc influencer :

- la connectivité du namespace ;
    
- le NAT ;
    
- les communications entre namespaces ;
    
- les communications vers Internet.
    

Dans un diagnostic réseau, nous devons regarder :

```bash
ip route
nft list ruleset
iptables -L -n -v
iptables -t nat -L -n -v
```

selon le système.

---

## 7.16. Sécurité du namespace réseau

## 7.16.1. Ce que le namespace réseau protège

Le namespace réseau limite la visibilité et l’accès aux éléments réseau de l’hôte.

Un processus isolé ne voit pas directement :

- les interfaces de l’hôte ;
    
- les ports en écoute de l’hôte ;
    
- les sockets de l’hôte ;
    
- les routes de l’hôte ;
    
- certains états réseau de l’hôte.
    

C’est une protection importante.

---

## 7.16.2. Ce qu’il ne protège pas seul

Le namespace réseau ne suffit pas à lui seul.

Nous devons aussi considérer :

- capabilities réseau ;
    
- accès à `/proc` et `/sys` ;
    
- montages sensibles ;
    
- règles de pare-feu ;
    
- cgroups ;
    
- politiques Kubernetes ;
    
- seccomp ;
    
- AppArmor ou SELinux.
    

Un conteneur avec `CAP_NET_ADMIN` peut modifier beaucoup de choses dans son namespace réseau.

S’il partage le namespace réseau de l’hôte, `CAP_NET_ADMIN` devient encore plus sensible.

---

## 7.16.3. Options dangereuses

Docker :

```bash
docker run --network=host ...
```

Kubernetes :

```yaml
hostNetwork: true
```

Ces options réduisent l’isolation réseau.

Elles peuvent être justifiées pour certains agents système, mais elles doivent être utilisées avec prudence.

---

## 7.17. Diagnostic réseau avec namespaces

## 7.17.1. Questions à se poser

Lorsqu’un conteneur ou un namespace réseau a un problème, nous posons les questions suivantes :

```text
Dans quel namespace suis-je ?
Quelles interfaces sont visibles ?
L’interface lo est-elle up ?
L’interface veth est-elle up ?
Quelle adresse IP est configurée ?
Quelle route par défaut existe ?
Le forwarding est-il activé ?
Le NAT est-il configuré ?
Le DNS fonctionne-t-il ?
Le pare-feu bloque-t-il ?
Le processus écoute-t-il dans le bon namespace ?
```

---

## 7.17.2. Commandes utiles

Dans le namespace :

```bash
ip addr
ip link
ip route
ss -tulpen
cat /proc/net/dev
cat /etc/resolv.conf
```

Depuis l’hôte vers un namespace nommé :

```bash
sudo ip netns exec ns1 ip addr
sudo ip netns exec ns1 ip route
sudo ip netns exec ns1 ss -tulpen
```

Depuis l’hôte vers un processus :

```bash
sudo nsenter --target <PID> --net ip addr
sudo nsenter --target <PID> --net ip route
sudo nsenter --target <PID> --net ss -tulpen
```

---

## 7.17.3. Exemple : un serveur écoute, mais il est inaccessible

Nous lançons dans `ns1` :

```bash
sudo ip netns exec ns1 python3 -m http.server 8080
```

Depuis l’hôte, nous testons :

```bash
curl http://10.0.0.2:8080
```

Si cela échoue, nous vérifions :

```bash
sudo ip netns exec ns1 ip addr
sudo ip netns exec ns1 ss -ltnp
ip addr show veth-host
ping -c 3 10.0.0.2
```

Nous cherchons :

- le serveur écoute-t-il sur `127.0.0.1` seulement ?
    
- `veth-ns` est-elle up ?
    
- `veth-host` est-elle up ?
    
- les IP sont-elles correctes ?
    
- le pare-feu bloque-t-il ?
    
- la route est-elle correcte ?
    

---

## 7.18. Expérience complète guidée

## 7.18.1. Créer un namespace réseau connecté à l’hôte

Nous exécutons :

```bash
sudo ip netns add ns1
sudo ip netns exec ns1 ip link set lo up

sudo ip link add veth-host type veth peer name veth-ns
sudo ip link set veth-ns netns ns1

sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up

sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
```

Nous testons :

```bash
ping -c 3 10.0.0.2
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

---

## 7.18.2. Lancer un service dans le namespace

Nous lançons :

```bash
sudo ip netns exec ns1 python3 -m http.server 8080
```

Dans un autre terminal, depuis l’hôte :

```bash
curl http://10.0.0.2:8080
```

Nous avons un service réseau isolé dans `ns1`, accessible via l’interface `veth`.

---

## 7.18.3. Observer les sockets

Dans `ns1` :

```bash
sudo ip netns exec ns1 ss -ltnp
```

Sur l’hôte :

```bash
ss -ltnp | grep 8080
```

Le service peut être visible différemment selon le contexte.

Nous comprenons ainsi que les sockets appartiennent à un namespace réseau.

---

## 7.18.4. Nettoyage

Nous arrêtons le serveur avec `Ctrl+C`.

Puis :

```bash
sudo ip netns delete ns1
sudo ip link delete veth-host 2>/dev/null
```

Nous vérifions :

```bash
ip netns list
ip link | grep veth
```

---

## 7.19. Pièges classiques

## 7.19.1. Oublier d’activer `lo`

Dans un namespace réseau neuf :

```bash
ip link set lo up
```

est souvent nécessaire.

Sans cela, même `127.0.0.1` peut ne pas fonctionner correctement.

---

## 7.19.2. Oublier de monter ou d’installer les outils

Dans un conteneur minimal, les commandes suivantes peuvent manquer :

```text
ip
ss
ping
curl
iptables
nft
```

Le namespace réseau existe, mais l’image ne contient pas forcément les outils de diagnostic.

Nous pouvons diagnostiquer depuis l’hôte avec `nsenter`.

---

## 7.19.3. Confondre `127.0.0.1` de l’hôte et du conteneur

Dans un conteneur :

```text
127.0.0.1
```

désigne le loopback du conteneur, pas celui de l’hôte.

Dans un pod Kubernetes, deux conteneurs du même pod partagent souvent le même `127.0.0.1`.

Nous devons toujours préciser le namespace réseau.

---

## 7.19.4. Oublier la route par défaut

Un namespace peut avoir une interface et une IP, mais aucune route par défaut.

Nous vérifions :

```bash
ip route
```

ou :

```bash
sudo ip netns exec ns1 ip route
```

---

## 7.19.5. Oublier le NAT

Si le namespace doit accéder à Internet via l’hôte, il faut souvent :

```text
route par défaut
forwarding
NAT
pare-feu adapté
DNS
```

Une IP configurée ne suffit pas.

---

## 7.19.6. Utiliser `hostNetwork` sans comprendre

`--network=host` ou `hostNetwork: true` supprime une partie importante de l’isolation réseau.

Cela peut provoquer :

- conflits de ports ;
    
- exposition de services ;
    
- confusion dans les diagnostics ;
    
- risques de sécurité accrus.
    

---

## 7.20. Exercices

## Exercice 1 — Observer le namespace réseau courant

Nous exécutons :

```bash
readlink /proc/$$/ns/net
readlink /proc/1/ns/net
ip addr
ip route
ss -tulpen
```

Nous répondons :

1. Le shell partage-t-il le namespace réseau du PID 1 ?
    
2. Quelles interfaces voyons-nous ?
    
3. Quelle est la route par défaut ?
    
4. Quels ports sont en écoute ?
    

---

## Exercice 2 — Créer un namespace réseau avec `unshare`

Nous exécutons :

```bash
unshare --net bash
ip addr
ip route
ip link set lo up
ip addr
exit
```

Nous répondons :

1. Quelles interfaces voyons-nous au départ ?
    
2. `lo` est-elle active ?
    
3. Que change `ip link set lo up` ?
    
4. Le namespace a-t-il une route par défaut ?
    

---

## Exercice 3 — Créer un namespace réseau nommé

Nous exécutons :

```bash
sudo ip netns add ns1
ip netns list
sudo ip netns exec ns1 ip addr
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns1 ip addr
sudo ip netns delete ns1
```

Nous répondons :

1. Où apparaît le namespace ?
    
2. Quelle interface est visible ?
    
3. Pourquoi activons-nous `lo` ?
    
4. Que fait `ip netns exec` ?
    

---

## Exercice 4 — Connecter un namespace à l’hôte

Nous exécutons :

```bash
sudo ip netns add ns1
sudo ip netns exec ns1 ip link set lo up

sudo ip link add veth-host type veth peer name veth-ns
sudo ip link set veth-ns netns ns1

sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up

sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up

ping -c 3 10.0.0.2
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

Nettoyage :

```bash
sudo ip netns delete ns1
sudo ip link delete veth-host 2>/dev/null
```

Nous répondons :

1. Quel est le rôle de `veth-host` ?
    
2. Quel est le rôle de `veth-ns` ?
    
3. Pourquoi le ping fonctionne-t-il ?
    
4. Que se passe-t-il quand nous supprimons le namespace ?
    

---

## Exercice 5 — Lancer un service dans un namespace

Nous reprenons la configuration de l’exercice 4.

Dans un terminal :

```bash
sudo ip netns exec ns1 python3 -m http.server 8080
```

Dans un autre terminal :

```bash
curl http://10.0.0.2:8080
sudo ip netns exec ns1 ss -ltnp
ss -ltnp | grep 8080
```

Nous répondons :

1. Dans quel namespace le service écoute-t-il ?
    
2. Quelle adresse utilisons-nous depuis l’hôte ?
    
3. Le port apparaît-il de la même manière dans `ss` côté hôte et côté namespace ?
    
4. Que se passerait-il si le serveur écoutait seulement sur `127.0.0.1` ?
    

---

## Exercice 6 — Créer deux namespaces avec un bridge

Nous créons :

```bash
sudo ip link add br0 type bridge
sudo ip addr add 10.10.0.1/24 dev br0
sudo ip link set br0 up

sudo ip netns add ns1
sudo ip netns add ns2
```

Nous connectons `ns1` :

```bash
sudo ip link add veth1-host type veth peer name veth1-ns
sudo ip link set veth1-ns netns ns1
sudo ip link set veth1-host master br0
sudo ip link set veth1-host up
sudo ip netns exec ns1 ip addr add 10.10.0.2/24 dev veth1-ns
sudo ip netns exec ns1 ip link set veth1-ns up
sudo ip netns exec ns1 ip link set lo up
```

Nous connectons `ns2` :

```bash
sudo ip link add veth2-host type veth peer name veth2-ns
sudo ip link set veth2-ns netns ns2
sudo ip link set veth2-host master br0
sudo ip link set veth2-host up
sudo ip netns exec ns2 ip addr add 10.10.0.3/24 dev veth2-ns
sudo ip netns exec ns2 ip link set veth2-ns up
sudo ip netns exec ns2 ip link set lo up
```

Nous testons :

```bash
sudo ip netns exec ns1 ping -c 3 10.10.0.3
sudo ip netns exec ns2 ping -c 3 10.10.0.2
```

Nettoyage :

```bash
sudo ip netns delete ns1
sudo ip netns delete ns2
sudo ip link delete br0
```

Nous répondons :

1. Quel rôle joue le bridge ?
    
2. Pourquoi l’appelons-nous un switch virtuel ?
    
3. Pourquoi Docker utilise-t-il une approche similaire ?
    
4. Que faudrait-il ajouter pour permettre l’accès Internet ?
    

---

## Exercice 7 — Observer un conteneur Docker

Si Docker est disponible :

```bash
docker run --rm -d --name net-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' net-demo)

sudo readlink /proc/$pid/ns/net
readlink /proc/1/ns/net

sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route

docker rm -f net-demo
```

Nous répondons :

1. Le conteneur partage-t-il le namespace réseau de l’hôte ?
    
2. Quelles interfaces voit-il ?
    
3. Quelle est sa route par défaut ?
    
4. Quel mécanisme relie le conteneur au réseau de l’hôte ?
    

---

## 7.21. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace network isole la pile réseau.
    
2. Il isole les interfaces, adresses IP, routes, sockets et ports.
    
3. `/proc/<PID>/ns/net` permet d’identifier le namespace réseau d’un processus.
    
4. `/proc/net` dépend du namespace réseau courant.
    
5. Un namespace réseau neuf contient souvent seulement `lo`, parfois désactivée.
    
6. Nous devons activer `lo` avec `ip link set lo up`.
    
7. Une paire `veth` permet de connecter deux namespaces réseau.
    
8. `ip netns` permet de créer des namespaces réseau nommés.
    
9. Un bridge Linux fonctionne comme un switch virtuel.
    
10. Pour sortir vers Internet, il faut souvent route par défaut, forwarding, NAT et DNS.
    
11. Docker utilise des namespaces réseau, des paires `veth`, des bridges et des règles NAT.
    
12. Dans Kubernetes, les conteneurs d’un même pod partagent généralement le même namespace réseau.
    
13. Le pause container maintient souvent le namespace réseau du pod.
    
14. Les plugins CNI configurent le réseau des pods.
    
15. `--network=host` et `hostNetwork: true` réduisent fortement l’isolation.
    
16. `127.0.0.1` désigne le loopback du namespace courant, pas forcément celui de l’hôte.
    
17. Le namespace réseau est puissant, mais il doit être compris avec les routes, le pare-feu, le NAT et les politiques de sécurité.
    

---

## Conclusion du chapitre 7

Nous avons étudié le namespace network, qui est l’un des namespaces les plus importants pour les conteneurs et l’isolation applicative.

Nous savons maintenant créer un namespace réseau, activer son loopback, le connecter à l’hôte avec une paire `veth`, configurer des adresses IP, ajouter des routes, comprendre le forwarding, le NAT et le rôle d’un bridge Linux.

Nous avons aussi relié ces notions aux environnements réels : Docker utilise des bridges, des paires `veth` et du NAT ; Kubernetes confie la configuration réseau aux plugins CNI et fait généralement partager le namespace réseau aux conteneurs d’un même pod.

Nous retenons surtout qu’un problème réseau dans un conteneur ne se diagnostique pas comme un problème réseau classique. Nous devons toujours nous demander dans quel namespace nous observons les interfaces, les routes, les sockets et les ports.

Dans le chapitre suivant, nous étudions le namespace IPC, qui isole certains mécanismes de communication inter-processus comme les files de messages, les sémaphores et la mémoire partagée.

---
> [!info] Livre « Les namespaces Linux » — chapitre 7/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 06 — Namespace mount|← 06 — Namespace mount]] · [[Les namespaces Linux — 08 — Namespace IPC|08 — Namespace IPC →]]
