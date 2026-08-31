---
schema_version: 1
uid: 01M1BQ629YVJBC168AGANB1RR2
titre: "proc — 11 — Limites de proc"
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
resume: "Chapitre 11 sur 11 du livre « proc » : Limites de /proc. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 11/11
> [[proc — Sommaire|Sommaire]] · [[proc — 10 — proc et les outils système|← 10 — proc et les outils système]] · [[proc — Compléments 2026|Compléments 2026 →]]

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

---
> [!info] Livre « proc » — chapitre 11/11
> [[proc — Sommaire|Sommaire]] · [[proc — 10 — proc et les outils système|← 10 — proc et les outils système]] · [[proc — Compléments 2026|Compléments 2026 →]]
