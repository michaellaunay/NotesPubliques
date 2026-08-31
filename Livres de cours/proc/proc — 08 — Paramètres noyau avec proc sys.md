---
schema_version: 1
uid: 01M1BQ629TB3T1ATY8YV4ZC370
titre: "proc — 08 — Paramètres noyau avec proc sys"
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
resume: "Chapitre 8 sur 11 du livre « proc » : Paramètres noyau avec /proc/sys. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 8/11
> [[proc — Sommaire|Sommaire]] · [[proc — 07 — Réseau et proc|← 07 — Réseau et proc]] · [[proc — 09 — Sécurité, permissions et isolation|09 — Sécurité, permissions et isolation →]]

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

### Exercice 1 — Explorer `/proc/sys`

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


### Exercice 2 — Correspondance `/proc/sys` et `sysctl`

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
    


### Exercice 3 — Lire des paramètres système

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
    


### Exercice 4 — Modifier temporairement un paramètre non critique

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
    


### Exercice 5 — Comprendre l’erreur avec `sudo echo`

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
    


### Exercice 6 — Inotify

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

---
> [!info] Livre « proc » — chapitre 8/11
> [[proc — Sommaire|Sommaire]] · [[proc — 07 — Réseau et proc|← 07 — Réseau et proc]] · [[proc — 09 — Sécurité, permissions et isolation|09 — Sécurité, permissions et isolation →]]
