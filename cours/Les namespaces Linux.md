# Objectifs généraux du cours

À la fin du cours, nous savons expliquer le rôle des namespaces Linux, comprendre leur place dans l’isolation des processus, manipuler les principaux types de namespaces, et faire le lien avec les conteneurs, Docker, Kubernetes, systemd, `/proc`, les cgroups et la sécurité.

# Chapitre 1 — Introduction aux namespaces Linux

## Objectifs du chapitre

Dans ce premier chapitre, nous posons les bases du cours sur les namespaces Linux.

Nous cherchons à comprendre pourquoi Linux a introduit des mécanismes d’isolation, en quoi ces mécanismes diffèrent de la virtualisation classique, et pourquoi ils sont devenus essentiels avec les conteneurs.

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est un namespace Linux ;
    
- comprendre pourquoi nous isolons des processus ;
    
- distinguer isolation par namespaces et virtualisation par machine virtuelle ;
    
- expliquer le lien entre namespaces et conteneurs ;
    
- comprendre pourquoi un conteneur n’est pas une machine virtuelle ;
    
- identifier les principaux types de namespaces Linux.
    

---

# 1.1. Pourquoi isoler des processus ?

## 1.1.1. Le problème de départ

Sur un système Linux classique, tous les processus partagent le même noyau.

Ils ne voient pas nécessairement les mêmes droits, mais ils vivent dans le même environnement global :

- même table globale des processus ;
    
- même pile réseau ;
    
- même hostname ;
    
- même arborescence de montages ;
    
- mêmes mécanismes IPC ;
    
- même noyau ;
    
- mêmes ressources matérielles sous-jacentes.
    

Cela pose une question importante :

```text
Comment faire croire à un groupe de processus qu’il vit dans son propre système, sans lancer une machine virtuelle complète ?
```

C’est précisément l’un des rôles des namespaces.

---

## 1.1.2. Exemple simple : la vue des processus

Sur une machine Linux classique, nous pouvons afficher les processus :

```bash
ps aux
```

Nous voyons alors de nombreux processus du système.

Mais dans un conteneur Docker, si nous exécutons :

```bash
ps aux
```

nous pouvons ne voir qu’un ou quelques processus.

Pourtant, le conteneur tourne sur le même noyau que l’hôte.

Cela signifie que le noyau présente une vue différente selon le contexte du processus.

Cette idée est fondamentale :

```text
Un namespace ne crée pas forcément une nouvelle ressource.
Il crée une vue isolée d’une ressource existante.
```

---

## 1.1.3. Isoler pour organiser

Nous isolons des processus pour plusieurs raisons :

- éviter qu’un processus voie tous les autres processus ;
    
- donner un hostname différent à un environnement ;
    
- séparer les piles réseau ;
    
- donner une arborescence de fichiers différente ;
    
- limiter les interactions entre applications ;
    
- construire des environnements reproductibles ;
    
- renforcer la sécurité ;
    
- faire fonctionner plusieurs services avec des dépendances différentes ;
    
- exécuter des applications dans des conteneurs.
    

Les namespaces sont donc à la fois un outil d’isolation, d’organisation et de construction d’environnements applicatifs.

---

# 1.2. Qu’est-ce qu’un namespace Linux ?

## 1.2.1. Définition générale

Un namespace Linux est un mécanisme du noyau qui donne à un processus et à ses descendants une vue isolée d’une ressource système.

Autrement dit, deux processus peuvent utiliser le même noyau, mais ne pas voir la même chose.

Par exemple :

- dans un namespace PID, un processus peut voir son propre arbre de processus ;
    
- dans un namespace réseau, un processus peut voir ses propres interfaces réseau ;
    
- dans un namespace UTS, un processus peut voir un hostname différent ;
    
- dans un namespace mount, un processus peut voir une arborescence de montages différente.
    

Nous pouvons résumer ainsi :

```text
Les namespaces isolent ce que les processus voient.
```

---

## 1.2.2. Un processus appartient à plusieurs namespaces

Un processus Linux n’appartient pas à un seul namespace, mais à plusieurs namespaces en même temps.

Nous pouvons observer cela avec :

```bash
ls -l /proc/$$/ns
```

Exemple de sortie :

```text
cgroup -> cgroup:[4026531835]
ipc    -> ipc:[4026531839]
mnt    -> mnt:[4026531841]
net    -> net:[4026531840]
pid    -> pid:[4026531836]
time   -> time:[4026531834]
user   -> user:[4026531837]
uts    -> uts:[4026531838]
```

Chaque ligne représente un type de namespace auquel appartient notre shell courant.

Le nombre entre crochets identifie une instance de namespace.

Deux processus qui ont le même identifiant pour un type donné partagent ce namespace.

---

## 1.2.3. Les namespaces ne sont pas des machines virtuelles

Un namespace ne lance pas un nouveau noyau.

Il ne virtualise pas tout le matériel.

Il ne crée pas une machine complète.

Il isole certaines vues du système pour certains processus.

C’est une différence majeure avec les machines virtuelles.

Dans une machine virtuelle, nous avons généralement :

```text
application
système invité
noyau invité
matériel virtuel
hyperviseur
noyau hôte
matériel réel
```

Avec des namespaces, nous avons plutôt :

```text
application isolée
namespaces
noyau Linux commun
matériel réel
```

Les conteneurs sont donc plus légers que les machines virtuelles, mais ils partagent le noyau avec l’hôte.

---

# 1.3. Virtualisation classique et isolation par namespaces

## 1.3.1. Machine virtuelle

Une machine virtuelle exécute un système d’exploitation invité complet.

Par exemple, sur un hôte Linux, nous pouvons lancer :

- une VM Ubuntu ;
    
- une VM Debian ;
    
- une VM Windows ;
    
- une VM FreeBSD.
    

Chaque VM possède son propre noyau invité.

Cela apporte une isolation forte, mais avec un coût plus important :

- consommation mémoire plus élevée ;
    
- démarrage plus lent ;
    
- image système plus lourde ;
    
- couche d’hypervision ;
    
- administration plus proche d’une machine complète.
    

---

## 1.3.2. Conteneur avec namespaces

Un conteneur Linux n’exécute pas son propre noyau.

Il utilise le noyau de l’hôte, mais avec des vues isolées.

Par exemple, un conteneur peut avoir :

- son propre PID 1 ;
    
- son propre hostname ;
    
- ses propres interfaces réseau ;
    
- son propre système de fichiers apparent ;
    
- ses propres montages ;
    
- ses propres utilisateurs mappés ;
    
- ses propres limites de ressources via les cgroups.
    

Mais il partage toujours le noyau avec l’hôte.

Nous pouvons donc dire :

```text
La VM virtualise une machine.
Le conteneur isole des processus.
```

---

## 1.3.3. Comparaison synthétique

|Point comparé|Machine virtuelle|Conteneur avec namespaces|
|---|---|---|
|Noyau|noyau invité séparé|noyau hôte partagé|
|Démarrage|plus lent|plus rapide|
|Isolation|généralement plus forte|dépend de la configuration|
|Poids|plus lourd|plus léger|
|Usage|systèmes complets|applications et services|
|Sécurité|frontière plus nette|noyau partagé, vigilance nécessaire|
|Exemples|KVM, VMware, VirtualBox|Docker, Podman, LXC, Kubernetes|

Nous ne devons pas opposer naïvement les deux approches.

Elles répondent à des besoins différents.

---

# 1.4. Lien entre namespaces et conteneurs

## 1.4.1. Un conteneur n’est pas une magie Docker

Docker, Podman ou Kubernetes ne créent pas l’isolation à partir de rien.

Ils orchestrent plusieurs mécanismes du noyau Linux, notamment :

- namespaces ;
    
- cgroups ;
    
- capabilities ;
    
- seccomp ;
    
- AppArmor ou SELinux ;
    
- systèmes de fichiers en couches ;
    
- bridges réseau ;
    
- règles de pare-feu ;
    
- runtime OCI comme `runc`.
    

Les namespaces constituent l’un des fondements techniques des conteneurs.

---

## 1.4.2. Ce que Docker automatise

Quand nous lançons :

```bash
docker run --rm -it alpine sh
```

Docker prépare beaucoup de choses pour nous.

Il peut notamment :

- créer un namespace PID ;
    
- créer un namespace mount ;
    
- créer un namespace UTS ;
    
- créer un namespace network ;
    
- créer un namespace IPC ;
    
- configurer des cgroups ;
    
- monter un root filesystem ;
    
- configurer `/proc` ;
    
- créer une interface réseau virtuelle ;
    
- appliquer des capabilities ;
    
- lancer le processus initial.
    

Nous pourrions faire une partie de cela à la main avec `unshare`, `mount`, `ip`, `chroot` ou `pivot_root`, mais Docker automatise le tout.

---

## 1.4.3. Pourquoi étudier les namespaces si Docker existe ?

Nous étudions les namespaces parce qu’un bon ingénieur ne doit pas utiliser les conteneurs comme une boîte noire.

Comprendre les namespaces permet de répondre à des questions importantes :

- pourquoi le PID 1 d’un conteneur est-il spécial ?
    
- pourquoi deux conteneurs ne voient-ils pas les mêmes interfaces réseau ?
    
- pourquoi un conteneur peut-il voir un hostname différent ?
    
- pourquoi `/proc` dans un conteneur ne montre pas tous les processus de l’hôte ?
    
- pourquoi `hostNetwork`, `hostPID` ou `privileged` changent fortement l’isolation ?
    
- pourquoi un conteneur rootless peut fonctionner ?
    
- pourquoi les cgroups ne sont pas la même chose que les namespaces ?
    

---

# 1.5. Limites : un conteneur n’est pas une machine virtuelle

## 1.5.1. Le noyau est partagé

Le point central est le suivant :

```text
Les conteneurs Linux partagent le noyau de l’hôte.
```

Cela signifie que si le noyau est vulnérable, l’isolation peut être affaiblie.

Cela signifie aussi qu’un conteneur Linux ne peut pas exécuter directement un noyau différent de celui de l’hôte.

Par exemple, un conteneur Ubuntu sur un hôte Debian utilise toujours le noyau de l’hôte Debian.

Il peut avoir des bibliothèques Ubuntu dans son root filesystem, mais pas un noyau Ubuntu séparé.

---

## 1.5.2. Isolation forte ou isolation légère ?

Les namespaces fournissent une isolation très utile, mais ce n’est pas la même barrière qu’une machine virtuelle.

Un conteneur mal configuré peut être dangereux.

Exemples de configurations sensibles :

```bash
docker run --privileged ...
docker run -v /:/host ...
docker run -v /proc:/host/proc ...
docker run --pid=host ...
docker run --network=host ...
```

Ces options réduisent fortement l’isolation.

Nous devons donc éviter l’idée :

```text
C’est dans un conteneur, donc c’est forcément isolé.
```

La vraie question est :

```text
Quels namespaces sont isolés ?
Quelles capabilities sont accordées ?
Quels montages sont exposés ?
Quels cgroups limitent les ressources ?
Quelles politiques de sécurité sont actives ?
```

---

## 1.5.3. Les namespaces isolent la vue, pas toujours la consommation

Les namespaces répondent surtout à la question :

```text
Que voit le processus ?
```

Mais ils ne répondent pas entièrement à la question :

```text
Combien de ressources peut-il consommer ?
```

Pour limiter les ressources, nous avons besoin des cgroups.

Exemple :

- le namespace PID peut limiter la vue des processus ;
    
- mais le cgroup `pids` peut limiter le nombre de processus créables ;
    
- le namespace network isole la pile réseau ;
    
- mais les limites de débit ou politiques réseau demandent d’autres mécanismes ;
    
- le namespace mount isole les montages ;
    
- mais les quotas disque demandent encore d’autres mécanismes.
    

Nous retenons :

```text
Namespaces = isolation des vues.
Cgroups = limitation et comptabilité des ressources.
```

---

# 1.6. Vue d’ensemble des principaux namespaces

## 1.6.1. Namespace UTS

Le namespace UTS isole :

- le hostname ;
    
- le domainname.
    

Il permet à un conteneur d’avoir son propre nom de machine.

Exemple :

```bash
unshare --uts bash
hostname test-namespace
hostname
```

Dans ce shell isolé, le hostname peut être différent de celui de l’hôte.

---

## 1.6.2. Namespace PID

Le namespace PID isole la numérotation des processus.

Dans un namespace PID, un processus peut voir son propre PID 1.

Exemple :

```bash
unshare --fork --pid --mount-proc bash
ps aux
cat /proc/1/comm
```

Ce namespace est fondamental pour les conteneurs.

Il permet qu’un processus soit PID 1 dans le conteneur, même s’il a un autre PID sur l’hôte.

---

## 1.6.3. Namespace mount

Le namespace mount isole les points de montage.

Il permet à un processus de voir une arborescence de fichiers différente.

Exemple :

```bash
unshare --mount bash
mount -t tmpfs tmpfs /tmp/test-mnt
```

Le montage peut exister dans le namespace courant sans apparaître dans l’environnement parent, selon la propagation des montages.

Ce namespace est essentiel pour donner à un conteneur son propre root filesystem.

---

## 1.6.4. Namespace network

Le namespace network isole la pile réseau.

Il fournit une vue séparée de :

- interfaces réseau ;
    
- adresses IP ;
    
- routes ;
    
- règles de firewall ;
    
- sockets ;
    
- ports en écoute ;
    
- table ARP ;
    
- loopback.
    

Exemple :

```bash
unshare --net bash
ip addr
```

Dans un nouveau namespace réseau, nous voyons généralement seulement l’interface loopback, souvent désactivée au départ.

---

## 1.6.5. Namespace IPC

Le namespace IPC isole certains mécanismes de communication inter-processus.

Il concerne notamment :

- files de messages System V ;
    
- sémaphores ;
    
- mémoire partagée System V ;
    
- certaines ressources IPC POSIX.
    

Exemple :

```bash
ipcs
unshare --ipc bash
ipcs
```

Nous pouvons voir une vue différente des ressources IPC.

---

## 1.6.6. Namespace user

Le namespace user isole les identifiants utilisateurs et groupes.

Il permet de mapper des UID/GID différents entre l’intérieur du namespace et l’hôte.

C’est ce qui permet, par exemple, d’être `root` dans un namespace sans être réellement `root` sur l’hôte.

Exemple :

```bash
unshare --user --map-root-user bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Nous pouvons obtenir :

```text
uid=0(root) gid=0(root)
```

dans le namespace, alors que notre utilisateur réel sur l’hôte reste non privilégié.

Ce mécanisme est central pour les conteneurs rootless.

---

## 1.6.7. Namespace cgroup

Le namespace cgroup isole la vue des cgroups.

Il ne crée pas les limites de ressources lui-même.

Il permet plutôt à un processus de voir une racine cgroup différente.

Nous devons bien distinguer :

```text
cgroups : limitent et mesurent les ressources.
cgroup namespace : isole la vue des cgroups.
```

---

## 1.6.8. Namespace time

Le namespace time permet d’isoler certains offsets d’horloges, notamment pour :

- `CLOCK_MONOTONIC` ;
    
- `CLOCK_BOOTTIME`.
    

Il ne permet pas simplement de changer l’heure réelle globale du système pour un conteneur.

Il sert surtout à des cas plus spécialisés :

- tests ;
    
- checkpoint/restore ;
    
- environnements isolés avancés.
    

Nous pouvons observer :

```bash
ls -l /proc/$$/ns/time
cat /proc/self/timens_offsets
```

---

# 1.7. Première observation avec `/proc`

## 1.7.1. Observer les namespaces du shell courant

Nous pouvons commencer sans créer de namespace.

Nous observons les namespaces de notre shell :

```bash
ls -l /proc/$$/ns
```

Sortie possible :

```text
cgroup -> cgroup:[4026531835]
ipc    -> ipc:[4026531839]
mnt    -> mnt:[4026531841]
net    -> net:[4026531840]
pid    -> pid:[4026531836]
time   -> time:[4026531834]
user   -> user:[4026531837]
uts    -> uts:[4026531838]
```

Chaque lien symbolique représente un namespace.

---

## 1.7.2. Comparer avec le PID 1

Nous pouvons comparer avec le processus 1 :

```bash
ls -l /proc/1/ns
```

Si notre shell est sur l’hôte, il partage probablement plusieurs namespaces avec le PID 1.

Nous pouvons comparer précisément :

```bash
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
```

Si les deux valeurs sont identiques, les deux processus sont dans le même namespace PID.

Si elles sont différentes, ils sont dans deux namespaces PID distincts.

---

## 1.7.3. Observer avec `lsns`

La commande `lsns` liste les namespaces du système :

```bash
lsns
```

Nous pouvons filtrer :

```bash
lsns -t pid
lsns -t net
lsns -t mnt
```

`lsns` permet de voir :

- le type de namespace ;
    
- son identifiant ;
    
- le nombre de processus ;
    
- un PID associé ;
    
- la commande associée.
    

C’est un outil très utile pour comprendre l’état des namespaces sur une machine.

---

# 1.8. Exemple intuitif : isoler le hostname

## 1.8.1. Situation initiale

Nous regardons le hostname de la machine :

```bash
hostname
```

Exemple :

```text
machine-hote
```

Si nous modifions le hostname sans namespace, nous modifions le hostname global du système.

Mais avec un namespace UTS, nous pouvons changer le hostname seulement dans une vue isolée.

---

## 1.8.2. Créer un namespace UTS

Nous lançons :

```bash
unshare --uts bash
```

Nous sommes maintenant dans un shell qui possède un namespace UTS distinct.

Nous modifions le hostname :

```bash
hostname namespace-test
hostname
```

Nous voyons :

```text
namespace-test
```

Nous quittons :

```bash
exit
```

Puis nous relisons :

```bash
hostname
```

Nous retrouvons le hostname de l’hôte.

Cette démonstration montre le principe des namespaces : une ressource semble différente depuis l’intérieur du namespace, sans changer la vue extérieure.

---

# 1.9. Les namespaces comme fondation des conteneurs

## 1.9.1. Conteneur minimal conceptuel

Un conteneur combine plusieurs isolations.

Un conteneur typique peut utiliser :

```text
UTS namespace   → hostname isolé
PID namespace   → arbre de processus isolé
mount namespace → système de fichiers isolé
net namespace   → réseau isolé
IPC namespace   → IPC isolé
user namespace  → utilisateurs mappés
cgroups         → ressources limitées
capabilities    → privilèges réduits
seccomp         → appels système filtrés
```

Docker, Podman ou Kubernetes automatisent cette combinaison.

---

## 1.9.2. Ce que nous allons faire dans le cours

Dans ce cours, nous n’allons pas seulement utiliser Docker.

Nous allons comprendre les briques de base.

Nous allons manipuler directement :

```bash
unshare
nsenter
lsns
ip netns
mount
ps
ip
```

Puis nous ferons le lien avec :

- Docker ;
    
- Kubernetes ;
    
- `/proc` ;
    
- cgroups ;
    
- systemd ;
    
- sécurité.
    

L’objectif est de pouvoir diagnostiquer des situations réelles, par exemple :

- un conteneur ne voit pas le bon réseau ;
    
- un processus est PID 1 et ne gère pas les signaux ;
    
- `/proc` ne montre pas les bons processus ;
    
- un conteneur rootless a des permissions inattendues ;
    
- un pod Kubernetes partage le réseau entre plusieurs containers ;
    
- un montage apparaît dans l’hôte alors qu’il devait rester isolé.
    

---

# 1.10. Vocabulaire essentiel

## 1.10.1. Namespace

Un namespace est une vue isolée d’une ressource noyau.

Exemple :

```text
namespace PID → vue isolée des processus
namespace net → vue isolée du réseau
```

---

## 1.10.2. Processus

Un processus est une instance d’un programme en cours d’exécution.

Un processus appartient à plusieurs namespaces.

---

## 1.10.3. Namespace parent et enfant

Certains namespaces peuvent être hiérarchiques, notamment le namespace PID et le namespace user.

Un processus dans un namespace parent peut parfois voir des processus du namespace enfant, mais l’inverse n’est pas vrai.

---

## 1.10.4. `unshare`

`unshare` permet de lancer un programme dans un ou plusieurs nouveaux namespaces.

Exemple :

```bash
unshare --uts bash
```

---

## 1.10.5. `nsenter`

`nsenter` permet d’entrer dans les namespaces d’un processus existant.

Exemple :

```bash
nsenter --target <PID> --net bash
```

---

## 1.10.6. `lsns`

`lsns` permet de lister les namespaces présents sur le système.

Exemple :

```bash
lsns -t net
```

---

# 1.11. Points d’attention dès le début

## 1.11.1. Nous devons savoir où nous sommes

Quand nous observons un système, nous devons toujours nous demander :

```text
Sommes-nous sur l’hôte ?
Dans un conteneur ?
Dans un namespace particulier ?
Dans un chroot ?
Dans un pod Kubernetes ?
```

La réponse change l’interprétation des commandes.

---

## 1.11.2. `/proc` dépend des namespaces

Le contenu de `/proc` dépend du contexte.

Exemples :

- `/proc` dans un namespace PID ne montre pas forcément tous les processus de l’hôte ;
    
- `/proc/net` dépend du namespace réseau ;
    
- `/proc/mounts` dépend du namespace de montage ;
    
- `/proc/<PID>/uid_map` dépend du namespace user.
    

Nous faisons donc un lien direct avec le cours sur `/proc`.

---

## 1.11.3. Isolation ne veut pas dire sécurité absolue

Les namespaces contribuent à la sécurité, mais ils ne suffisent pas seuls.

Nous devons aussi considérer :

- cgroups ;
    
- capabilities ;
    
- seccomp ;
    
- AppArmor ;
    
- SELinux ;
    
- droits Unix ;
    
- montages ;
    
- noyau partagé ;
    
- mises à jour de sécurité.
    

Un conteneur mal configuré peut être beaucoup moins isolé qu’on ne l’imagine.

---

# 1.12. Exercices

## Exercice 1 — Observer les namespaces du shell courant

Nous exécutons :

```bash
ls -l /proc/$$/ns
```

Nous répondons :

1. Quels types de namespaces voyons-nous ?
    
2. Quel est l’identifiant du namespace PID ?
    
3. Quel est l’identifiant du namespace réseau ?
    
4. Ces identifiants sont-ils les mêmes que ceux de `/proc/1/ns` ?
    

---

## Exercice 2 — Comparer deux processus

Nous exécutons :

```bash
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
readlink /proc/$$/ns/net
readlink /proc/1/ns/net
```

Nous répondons :

1. Le shell courant partage-t-il le namespace PID avec le PID 1 ?
    
2. Partage-t-il le namespace réseau ?
    
3. Que signifie une valeur identique ?
    
4. Que signifie une valeur différente ?
    

---

## Exercice 3 — Utiliser `lsns`

Nous exécutons :

```bash
lsns
lsns -t pid
lsns -t net
lsns -t mnt
```

Nous répondons :

1. Combien de namespaces PID voyons-nous ?
    
2. Combien de namespaces réseau voyons-nous ?
    
3. Quels processus semblent associés à ces namespaces ?
    
4. Voyons-nous des namespaces liés à des conteneurs ?
    

---

## Exercice 4 — Créer un namespace UTS

Nous exécutons :

```bash
hostname
unshare --uts bash
hostname namespace-test
hostname
exit
hostname
```

Nous répondons :

1. Le hostname change-t-il dans le shell isolé ?
    
2. Le hostname de l’hôte change-t-il après `exit` ?
    
3. Quel namespace avons-nous isolé ?
    
4. Pourquoi cet exemple illustre-t-il bien le concept de namespace ?
    

---

## Exercice 5 — Réfléchir aux conteneurs

Nous lançons un conteneur si Docker ou Podman est disponible :

```bash
docker run --rm -it alpine sh
```

Dans le conteneur :

```sh
hostname
ps
ip addr
ls -l /proc/1/ns
```

Sur l’hôte, nous comparons avec :

```bash
hostname
ps aux
ip addr
lsns
```

Nous répondons :

1. Le conteneur voit-il les mêmes processus que l’hôte ?
    
2. Voit-il les mêmes interfaces réseau ?
    
3. A-t-il le même hostname ?
    
4. Quels namespaces semblent isolés ?
    

---

# 1.13. Ce que nous devons retenir

Nous retenons les points suivants :

1. Les namespaces sont des mécanismes du noyau Linux.
    
2. Ils isolent la vue qu’un processus a de certaines ressources.
    
3. Ils ne créent pas une machine virtuelle complète.
    
4. Un processus appartient à plusieurs namespaces en même temps.
    
5. Les conteneurs reposent largement sur les namespaces.
    
6. Les namespaces isolent les vues, tandis que les cgroups limitent les ressources.
    
7. Un conteneur partage le noyau avec l’hôte.
    
8. L’isolation dépend de la configuration réelle.
    
9. `/proc/<PID>/ns` permet d’observer les namespaces d’un processus.
    
10. `unshare`, `nsenter` et `lsns` sont les outils fondamentaux pour manipuler et observer les namespaces.
    
11. Comprendre les namespaces est indispensable pour comprendre Docker, Podman, Kubernetes et les conteneurs rootless.
    

---

# Conclusion du chapitre 1

Nous avons introduit les namespaces Linux comme un mécanisme fondamental d’isolation.

Nous avons vu que les namespaces permettent à des processus de voir des réalités différentes tout en partageant le même noyau. Cette idée est au cœur des conteneurs modernes.

Nous avons aussi posé une distinction essentielle : les namespaces ne sont pas des machines virtuelles. Ils isolent certaines vues du système, mais ils ne remplacent ni les cgroups, ni les mécanismes de sécurité, ni une isolation complète par hyperviseur.

Dans le chapitre suivant, nous étudions l’architecture générale des namespaces : comment un processus appartient à plusieurs namespaces, comment nous les observons avec `/proc/<PID>/ns`, comment ils sont créés avec `clone` ou `unshare`, et comment nous pouvons entrer dans un namespace existant avec `setns` ou `nsenter`.
```

---

# Chapitre 2 — Architecture générale des namespaces

- Notion d’espace de noms
    
- Un processus appartient à plusieurs namespaces
    
- Héritage des namespaces lors de `fork`
    
- Changement de namespace avec `setns`
    
- Création avec `clone` et `unshare`
    
- Observation des namespaces via `/proc/<PID>/ns`
    
- Identifiants symboliques des namespaces
    
- Durée de vie d’un namespace
    
- Namespace sans processus mais maintenu par un bind mount
    

Commandes principales :

```bash
ls -l /proc/$$/ns
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
```

---

# Chapitre 2 — Architecture générale des namespaces

## Objectifs du chapitre

Dans ce chapitre, nous étudions la structure interne des namespaces Linux.

Dans le chapitre précédent, nous avons défini un namespace comme une vue isolée d’une ressource noyau. Nous allons maintenant comprendre comment ces vues sont attachées aux processus, comment les observer, comment les créer, comment y entrer, et comment elles vivent ou disparaissent.

À la fin de ce chapitre, nous savons :

- expliquer qu’un processus appartient simultanément à plusieurs namespaces ;
    
- observer les namespaces d’un processus via `/proc/<PID>/ns` ;
    
- comparer les namespaces de deux processus ;
    
- comprendre le rôle des appels système `clone`, `unshare` et `setns` ;
    
- utiliser les commandes `unshare`, `nsenter` et `lsns` ;
    
- comprendre la durée de vie d’un namespace ;
    
- expliquer comment un namespace peut rester vivant même sans processus actif apparent ;
    
- faire le lien entre namespaces, processus, `/proc` et conteneurs.
    

---

# 2.1. Un processus appartient à plusieurs namespaces

## 2.1.1. Une appartenance multiple

Un processus Linux n’est pas “dans un namespace” au singulier.

Il appartient simultanément à plusieurs namespaces, chacun correspondant à une catégorie d’isolation.

Nous pouvons observer cela avec :

```bash
ls -l /proc/$$/ns
```

Sortie possible :

```text
cgroup -> cgroup:[4026531835]
ipc    -> ipc:[4026531839]
mnt    -> mnt:[4026531841]
net    -> net:[4026531840]
pid    -> pid:[4026531836]
time   -> time:[4026531834]
user   -> user:[4026531837]
uts    -> uts:[4026531838]
```

Chaque ligne correspond à un namespace différent.

Nous avons donc, pour le shell courant :

- un namespace `mnt` ;
    
- un namespace `pid` ;
    
- un namespace `net` ;
    
- un namespace `uts` ;
    
- un namespace `ipc` ;
    
- un namespace `user` ;
    
- un namespace `cgroup` ;
    
- un namespace `time`.
    

---

## 2.1.2. Même processus, plusieurs vues

Chaque namespace contrôle une vue différente.

Par exemple :

```text
mnt   → vue des montages
pid   → vue des processus
net   → vue réseau
uts   → hostname et domainname
ipc   → ressources IPC
user  → UID, GID et capabilities
cgroup → vue des cgroups
time  → offsets de certaines horloges
```

Nous pouvons donc avoir deux processus qui partagent leur namespace réseau, mais pas leur namespace PID.

Ou inversement, deux processus peuvent partager leur namespace PID, mais avoir des namespaces mount différents.

C’est cette combinaison qui permet de construire des environnements très variés.

---

## 2.1.3. Exemple conceptuel

Imaginons trois processus :

```text
Processus A
  mnt: 1
  pid: 1
  net: 1
  uts: 1

Processus B
  mnt: 2
  pid: 2
  net: 1
  uts: 2

Processus C
  mnt: 2
  pid: 2
  net: 3
  uts: 2
```

Nous pouvons dire :

- B et C partagent le même namespace mount ;
    
- B et C partagent le même namespace PID ;
    
- A et B partagent le même namespace réseau ;
    
- C a un namespace réseau différent ;
    
- B et C ont le même hostname isolé ;
    
- A a une autre vue UTS.
    

Cette granularité est fondamentale.

Un conteneur n’est donc pas seulement “un namespace”, mais une combinaison de plusieurs namespaces.

---

# 2.2. Observer les namespaces avec `/proc/<PID>/ns`

## 2.2.1. Le répertoire `ns`

Chaque processus possède un répertoire :

```bash
/proc/<PID>/ns
```

Pour le shell courant :

```bash
ls -l /proc/$$/ns
```

Nous obtenons des liens symboliques spéciaux.

Exemple :

```text
mnt -> mnt:[4026531841]
pid -> pid:[4026531836]
net -> net:[4026531840]
uts -> uts:[4026531838]
```

Ces liens ne sont pas des fichiers ordinaires. Ils représentent les namespaces auxquels appartient le processus.

---

## 2.2.2. Lire un namespace précis

Nous pouvons lire la cible d’un namespace avec `readlink`.

Exemple :

```bash
readlink /proc/$$/ns/pid
```

Sortie possible :

```text
pid:[4026531836]
```

Pour le namespace réseau :

```bash
readlink /proc/$$/ns/net
```

Sortie possible :

```text
net:[4026531840]
```

Le nombre entre crochets identifie l’instance du namespace.

---

## 2.2.3. Comparer deux processus

Nous pouvons comparer le shell courant et le PID 1 :

```bash
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
```

Si les deux valeurs sont identiques :

```text
pid:[4026531836]
pid:[4026531836]
```

les deux processus partagent le même namespace PID.

Si elles sont différentes :

```text
pid:[4026532501]
pid:[4026531836]
```

les deux processus ne sont pas dans le même namespace PID.

Même principe pour le réseau :

```bash
readlink /proc/$$/ns/net
readlink /proc/1/ns/net
```

---

## 2.2.4. Comparaison complète

Nous pouvons comparer tous les namespaces de deux processus avec un petit script :

```bash
pid1=$$
pid2=1

for ns in cgroup ipc mnt net pid time user uts; do
    a=$(readlink "/proc/$pid1/ns/$ns" 2>/dev/null || echo "absent")
    b=$(readlink "/proc/$pid2/ns/$ns" 2>/dev/null || echo "absent")

    if [ "$a" = "$b" ]; then
        printf "%-8s partagé     %s\n" "$ns" "$a"
    else
        printf "%-8s différent   %s / %s\n" "$ns" "$a" "$b"
    fi
done
```

Ce type de comparaison est très utile quand nous analysons des conteneurs.

---

# 2.3. Identifiants symboliques des namespaces

## 2.3.1. Que signifie `pid:[4026531836]` ?

Quand nous voyons :

```text
pid:[4026531836]
```

nous voyons une représentation symbolique d’un namespace PID.

Le nombre n’est pas un PID.

C’est un identifiant interne exposé par le noyau pour reconnaître une instance de namespace.

Deux processus qui affichent exactement la même valeur pour un type de namespace donné partagent ce namespace.

---

## 2.3.2. Attention aux comparaisons

Nous comparons toujours les namespaces par type.

Par exemple :

```text
pid:[4026531836]
net:[4026531836]
```

Même si les nombres étaient identiques, cela ne signifierait pas que ce sont le même objet, car les types sont différents.

Nous comparons donc :

```text
pid avec pid
net avec net
mnt avec mnt
```

et pas un type avec un autre.

---

# 2.4. Création de namespaces : `clone` et `unshare`

## 2.4.1. Deux idées différentes

Pour créer ou modifier l’appartenance d’un processus à des namespaces, Linux fournit notamment deux mécanismes :

```text
clone   → créer un nouveau processus, éventuellement dans de nouveaux namespaces
unshare → détacher le processus courant d’un ou plusieurs namespaces
```

En pratique, nous utilisons souvent la commande `unshare`, mais il faut comprendre les idées sous-jacentes.

---

## 2.4.2. `clone`

L’appel système `clone` permet de créer un nouveau processus ou thread avec un contrôle fin sur ce qui est partagé avec le parent.

Avec certains flags, `clone` peut créer un nouveau namespace.

Exemples conceptuels de flags :

```text
CLONE_NEWNS     → nouveau namespace mount
CLONE_NEWPID    → nouveau namespace PID
CLONE_NEWNET    → nouveau namespace réseau
CLONE_NEWUTS    → nouveau namespace UTS
CLONE_NEWIPC    → nouveau namespace IPC
CLONE_NEWUSER   → nouveau namespace user
CLONE_NEWCGROUP → nouveau namespace cgroup
CLONE_NEWTIME   → nouveau namespace time
```

Nous n’appelons généralement pas `clone` directement en shell. Ce sont les runtimes de conteneurs, bibliothèques système ou programmes spécialisés qui l’utilisent.

---

## 2.4.3. `unshare`

`unshare` permet à un processus de ne plus partager certains namespaces avec son parent.

La commande `unshare` est l’outil pédagogique principal.

Exemple :

```bash
unshare --uts bash
```

Nous lançons un nouveau shell avec un namespace UTS distinct.

Exemple plus complet :

```bash
unshare --fork --pid --mount-proc bash
```

Nous lançons un shell dans un nouveau namespace PID, avec un `/proc` monté pour refléter cette vue.

---

## 2.4.4. Pourquoi `--fork` est important avec le namespace PID

Le namespace PID a une particularité : le processus qui demande la création du nouveau namespace PID ne devient pas lui-même PID 1 dans ce namespace.

Ce sont ses enfants qui entrent dans le nouveau namespace PID.

C’est pour cela que nous utilisons souvent :

```bash
unshare --fork --pid --mount-proc bash
```

Ici :

- `unshare` crée le nouveau namespace PID ;
    
- `--fork` lance un processus enfant dans ce namespace ;
    
- ce processus enfant devient PID 1 dans le nouveau namespace ;
    
- `--mount-proc` monte `/proc` pour afficher la bonne vue des processus.
    

Sans `--fork`, le comportement peut surprendre.

---

# 2.5. Entrer dans un namespace existant : `setns` et `nsenter`

## 2.5.1. Le principe de `setns`

L’appel système `setns` permet à un processus d’entrer dans un namespace existant, en utilisant un descripteur vers un fichier de namespace, par exemple :

```text
/proc/<PID>/ns/net
```

C’est ce que font des outils comme `nsenter`.

---

## 2.5.2. Utiliser `nsenter`

La commande `nsenter` permet d’entrer dans les namespaces d’un processus cible.

Exemple :

```bash
sudo nsenter --target <PID> --net bash
```

Nous lançons un shell dans le namespace réseau du processus `<PID>`.

Nous pouvons entrer dans plusieurs namespaces à la fois :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid bash
```

Cela est très utile pour diagnostiquer un conteneur depuis l’hôte.

---

## 2.5.3. Exemple avec un processus cible

Nous lançons un shell isolé :

```bash
unshare --uts bash
```

Dans ce shell, nous changeons le hostname :

```bash
hostname ns-test
echo $$
sleep 1000
```

Depuis un autre terminal, nous pouvons entrer dans le namespace UTS du processus :

```bash
sudo nsenter --target <PID> --uts bash
hostname
```

Nous devons voir :

```text
ns-test
```

Cela montre que nous sommes entrés dans le namespace UTS du processus cible.

---

# 2.6. Lister les namespaces avec `lsns`

## 2.6.1. Utilisation de base

La commande `lsns` liste les namespaces visibles :

```bash
lsns
```

Sortie typique :

```text
        NS TYPE   NPROCS   PID USER COMMAND
4026531834 time      120     1 root /sbin/init
4026531835 cgroup    120     1 root /sbin/init
4026531836 pid       120     1 root /sbin/init
4026531837 user      120     1 root /sbin/init
4026531838 uts       120     1 root /sbin/init
4026531839 ipc       120     1 root /sbin/init
4026531840 net       120     1 root /sbin/init
4026531841 mnt       120     1 root /sbin/init
```

Les colonnes indiquent notamment :

- identifiant du namespace ;
    
- type ;
    
- nombre de processus ;
    
- PID associé ;
    
- utilisateur ;
    
- commande.
    

---

## 2.6.2. Filtrer par type

Nous pouvons filtrer par type.

Namespaces PID :

```bash
lsns -t pid
```

Namespaces réseau :

```bash
lsns -t net
```

Namespaces mount :

```bash
lsns -t mnt
```

Namespaces UTS :

```bash
lsns -t uts
```

Cela permet de voir rapidement si des conteneurs ou environnements isolés existent sur la machine.

---

## 2.6.3. Interpréter `NPROCS`

La colonne `NPROCS` indique le nombre de processus associés au namespace.

Mais nous devons rester prudents :

- certains namespaces peuvent être visibles grâce à des références particulières ;
    
- un namespace peut être maintenu vivant par un bind mount ;
    
- la visibilité dépend des permissions ;
    
- un processus peut disparaître pendant l’observation.
    

`lsns` donne une vue très utile, mais ce n’est pas une vérité absolue déconnectée du contexte.

---

# 2.7. Héritage des namespaces

## 2.7.1. Héritage lors de la création d’un processus

Quand un processus crée un enfant, celui-ci hérite normalement des namespaces de son parent.

Exemple :

```bash
bash
```

Le nouveau shell partage les mêmes namespaces que le shell parent, sauf si nous demandons explicitement à en créer de nouveaux.

Nous pouvons vérifier :

```bash
readlink /proc/$$/ns/uts
bash -c 'readlink /proc/$$/ns/uts'
```

Les deux valeurs sont généralement identiques.

---

## 2.7.2. Création explicite d’un nouveau namespace

Avec `unshare`, nous créons une rupture.

Exemple :

```bash
readlink /proc/$$/ns/uts
unshare --uts bash -c 'readlink /proc/$$/ns/uts'
```

Les deux valeurs doivent être différentes.

Cela signifie que le processus lancé par `unshare` appartient à un nouveau namespace UTS.

---

## 2.7.3. Héritage dans un conteneur

Dans un conteneur, le processus principal est lancé dans un ensemble de namespaces.

Ses processus enfants héritent ensuite de ces namespaces.

C’est pourquoi tous les processus d’un même conteneur partagent généralement :

- le même namespace PID ;
    
- le même namespace mount ;
    
- le même namespace UTS ;
    
- le même namespace network ;
    
- le même namespace IPC.
    

Sauf configuration particulière.

---

# 2.8. Durée de vie d’un namespace

## 2.8.1. Quand un namespace disparaît-il ?

Un namespace reste vivant tant qu’il existe au moins une référence vers lui.

La référence la plus évidente est un processus qui appartient à ce namespace.

Quand le dernier processus d’un namespace disparaît, le namespace peut être détruit.

Mais ce n’est pas la seule forme de référence.

---

## 2.8.2. Namespace maintenu par un fichier ouvert

Les entrées de `/proc/<PID>/ns/*` peuvent être ouvertes comme des fichiers.

Si un processus garde un descripteur ouvert vers un namespace, il peut maintenir ce namespace vivant.

C’est une notion importante pour des outils avancés.

---

## 2.8.3. Namespace maintenu par bind mount

Un namespace peut être maintenu vivant avec un bind mount vers son fichier de namespace.

Exemple conceptuel :

```bash
touch /tmp/ns-net
mount --bind /proc/<PID>/ns/net /tmp/ns-net
```

Même si le processus initial disparaît, le namespace peut rester référencé par le bind mount.

Nous pouvons ensuite entrer dans ce namespace via ce chemin, selon les outils et les permissions.

C’est notamment une idée utilisée par certains mécanismes de gestion de namespaces réseau.

---

# 2.9. Namespaces nommés avec `ip netns`

## 2.9.1. Cas particulier des namespaces réseau

La commande `ip netns` fournit une interface pratique pour gérer des namespaces réseau nommés.

Créer un namespace réseau :

```bash
sudo ip netns add ns1
```

Lister :

```bash
ip netns list
```

Exécuter une commande dans ce namespace :

```bash
sudo ip netns exec ns1 ip addr
```

Supprimer :

```bash
sudo ip netns delete ns1
```

---

## 2.9.2. Où sont stockées les références ?

Les namespaces réseau nommés sont généralement référencés sous :

```bash
/var/run/netns
```

ou :

```bash
/run/netns
```

Nous pouvons voir :

```bash
ls -l /run/netns
```

Ces fichiers sont des références vers des namespaces réseau.

Cela permet au namespace de survivre même sans processus actif permanent à l’intérieur.

---

## 2.9.3. Pourquoi c’est utile ?

Les namespaces réseau nommés sont très utiles pour :

- créer des laboratoires réseau ;
    
- tester des routes ;
    
- simuler plusieurs machines ;
    
- comprendre Docker ou Kubernetes ;
    
- manipuler des paires `veth` ;
    
- créer des topologies réseau locales.
    

Nous les étudions plus en détail dans le chapitre consacré au namespace réseau.

---

# 2.10. Les namespaces et `/proc`

## 2.10.1. `/proc` comme outil d’inspection

`/proc` est l’interface principale pour observer les namespaces d’un processus.

Nous utilisons :

```bash
/proc/<PID>/ns
```

mais aussi :

```bash
/proc/<PID>/status
/proc/<PID>/uid_map
/proc/<PID>/gid_map
/proc/<PID>/cgroup
/proc/<PID>/mountinfo
```

Selon le type de namespace, différentes parties de `/proc` deviennent utiles.

---

## 2.10.2. PID namespace et `/proc`

Dans un namespace PID, `/proc` doit généralement être remonté pour refléter correctement les processus visibles.

Exemple :

```bash
unshare --fork --pid bash
ps aux
```

peut donner une vue confuse si `/proc` reste celui de l’hôte.

La forme plus correcte est :

```bash
unshare --fork --pid --mount-proc bash
ps aux
```

Nous voyons alors une vue cohérente du namespace PID.

---

## 2.10.3. Network namespace et `/proc/net`

Le contenu de :

```bash
/proc/net
```

dépend du namespace réseau courant.

Exemple :

```bash
readlink /proc/$$/ns/net
cat /proc/net/dev
```

Dans un autre namespace réseau, nous pouvons voir d’autres interfaces.

---

## 2.10.4. User namespace et UID/GID maps

Pour un user namespace, nous regardons :

```bash
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Ces fichiers montrent comment les identifiants utilisateurs et groupes sont mappés entre l’intérieur du namespace et l’extérieur.

Exemple :

```text
         0       1000          1
```

Cela signifie que l’UID `0` à l’intérieur correspond à l’UID `1000` à l’extérieur, pour une plage de taille `1`.

---

# 2.11. Relation avec les conteneurs

## 2.11.1. Un conteneur est un assemblage de namespaces

Un conteneur typique combine plusieurs namespaces :

```text
mnt
pid
net
uts
ipc
user parfois
cgroup
time parfois
```

Le runtime crée ces namespaces, configure les montages, les cgroups, les capabilities et lance le processus initial.

---

## 2.11.2. Observer un conteneur depuis l’hôte

Nous pouvons récupérer le PID d’un conteneur, puis observer ses namespaces.

Avec Docker, par exemple :

```bash
docker inspect --format '{{.State.Pid}}' <container>
```

Puis :

```bash
sudo ls -l /proc/<PID>/ns
```

Nous pouvons comparer avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous voyons quels namespaces sont isolés.

---

## 2.11.3. Entrer dans les namespaces d’un conteneur

Depuis l’hôte :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid bash
```

Nous entrons dans plusieurs namespaces du conteneur.

C’est une technique utile pour diagnostiquer un conteneur même si aucun shell n’est disponible dans son image.

---

# 2.12. Pièges classiques

## 2.12.1. Confondre PID hôte et PID conteneur

Un processus peut avoir un PID sur l’hôte et un autre PID dans un namespace PID.

Dans le conteneur :

```bash
echo $$
```

peut afficher :

```text
1
```

Sur l’hôte, le même processus peut être :

```text
24531
```

Nous devons toujours préciser depuis quel namespace nous observons.

---

## 2.12.2. Oublier de remonter `/proc`

Quand nous créons un namespace PID à la main, nous devons monter `/proc` correctement.

Sinon, `ps` et `/proc` peuvent ne pas refléter la vue attendue.

Bonne pratique pédagogique :

```bash
unshare --fork --pid --mount-proc bash
```

---

## 2.12.3. Croire qu’un namespace limite les ressources

Un namespace isole une vue.

Il ne limite pas automatiquement la consommation de ressources.

Pour les ressources, nous devons utiliser les cgroups.

Nous retenons :

```text
Namespaces = isolation des vues.
Cgroups = contrôle et comptabilité des ressources.
```

---

## 2.12.4. Confondre user namespace et privilèges réels

Dans un user namespace, nous pouvons être `root` à l’intérieur.

Mais cela ne signifie pas que nous avons les privilèges root complets sur l’hôte.

Exemple :

```bash
unshare --user --map-root-user bash
id
```

Nous pouvons voir :

```text
uid=0(root)
```

Mais cette racine est limitée au namespace.

C’est une distinction centrale pour les conteneurs rootless.

---

# 2.13. Exercices

## Exercice 1 — Observer les namespaces du shell

Nous exécutons :

```bash
ls -l /proc/$$/ns
```

Nous répondons :

1. Quels types de namespaces voyons-nous ?
    
2. Quel est l’identifiant du namespace PID ?
    
3. Quel est l’identifiant du namespace réseau ?
    
4. Quel est l’identifiant du namespace mount ?
    
5. Tous les namespaces ont-ils le même identifiant ?
    

---

## Exercice 2 — Comparer le shell courant et le PID 1

Nous exécutons :

```bash
for ns in cgroup ipc mnt net pid time user uts; do
    echo "$ns"
    readlink "/proc/$$/ns/$ns" 2>/dev/null
    readlink "/proc/1/ns/$ns" 2>/dev/null
    echo
done
```

Nous répondons :

1. Quels namespaces sont partagés ?
    
2. Quels namespaces sont différents ?
    
3. Sommes-nous probablement sur l’hôte ou dans un environnement isolé ?
    
4. Que signifie une différence sur `pid` ?
    
5. Que signifie une différence sur `net` ?
    

---

## Exercice 3 — Créer un namespace UTS

Nous exécutons :

```bash
readlink /proc/$$/ns/uts
unshare --uts bash
readlink /proc/$$/ns/uts
hostname ns-test
hostname
exit
hostname
```

Nous répondons :

1. L’identifiant du namespace UTS a-t-il changé ?
    
2. Le hostname a-t-il changé dans le namespace ?
    
3. Le hostname de l’hôte est-il modifié après `exit` ?
    

---

## Exercice 4 — Créer un namespace PID correctement

Nous exécutons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
echo $$
ps aux
cat /proc/1/comm
ls -l /proc/$$/ns/pid
```

Nous répondons :

1. Quel est le PID du shell dans le namespace ?
    
2. Que montre `ps aux` ?
    
3. Quel est le processus PID 1 ?
    
4. Pourquoi avons-nous utilisé `--mount-proc` ?
    

Nous quittons avec :

```bash
exit
```

---

## Exercice 5 — Lister avec `lsns`

Nous exécutons :

```bash
lsns
lsns -t pid
lsns -t net
lsns -t uts
```

Nous répondons :

1. Combien de namespaces PID voyons-nous ?
    
2. Combien de namespaces réseau voyons-nous ?
    
3. Quels processus sont associés ?
    
4. Voyons-nous des namespaces qui pourraient appartenir à des conteneurs ?
    

---

## Exercice 6 — Créer un namespace réseau nommé

Nous exécutons :

```bash
sudo ip netns add ns-test
ip netns list
ls -l /run/netns
sudo ip netns exec ns-test ip addr
sudo ip netns delete ns-test
```

Nous répondons :

1. Le namespace apparaît-il dans `ip netns list` ?
    
2. Quelle interface voyons-nous à l’intérieur ?
    
3. Où la référence du namespace est-elle stockée ?
    
4. Que se passe-t-il après suppression ?
    

---

# 2.14. Ce que nous devons retenir

Nous retenons les points suivants :

1. Un processus appartient simultanément à plusieurs namespaces.
    
2. Les namespaces d’un processus sont visibles dans `/proc/<PID>/ns`.
    
3. Deux processus partagent un namespace si le lien symbolique correspondant a la même valeur.
    
4. Les namespaces s’héritent normalement lors de la création de processus.
    
5. `clone` peut créer un processus dans de nouveaux namespaces.
    
6. `unshare` permet de détacher un processus de certains namespaces.
    
7. `setns` permet d’entrer dans un namespace existant.
    
8. `nsenter` est l’outil pratique qui exploite cette logique.
    
9. `lsns` permet de lister les namespaces visibles sur le système.
    
10. Un namespace vit tant qu’il existe une référence vers lui.
    
11. Un namespace peut être maintenu vivant par un processus, un fichier ouvert ou un bind mount.
    
12. Les namespaces réseau nommés avec `ip netns` reposent sur des références dans `/run/netns`.
    
13. `/proc` est essentiel pour inspecter les namespaces.
    
14. Dans un namespace PID, il faut remonter `/proc` pour obtenir une vue cohérente.
    
15. Les namespaces isolent les vues, mais ne limitent pas automatiquement les ressources.
    

---

# Conclusion du chapitre 2

Nous avons étudié l’architecture générale des namespaces Linux.

Nous savons maintenant qu’un processus appartient à plusieurs namespaces en même temps, et que nous pouvons observer ces appartenances avec `/proc/<PID>/ns`. Nous avons compris que les namespaces sont hérités par défaut, mais qu’ils peuvent être créés ou modifiés avec des mécanismes comme `clone`, `unshare` et `setns`.

Nous avons également introduit les outils pratiques `unshare`, `nsenter`, `lsns` et `ip netns`, qui nous permettent de créer, lister et rejoindre des namespaces.

Dans le chapitre suivant, nous détaillons précisément les outils de manipulation des namespaces. Nous verrons comment les utiliser proprement, quelles options connaître, et comment éviter les erreurs classiques lors des manipulations manuelles.

# Chapitre 3 — Les outils de manipulation

- `unshare`
    
- `nsenter`
    
- `lsns`
    
- `ip netns`
    
- `mount`
    
- `findmnt`
    
- `ps`
    
- `ip`
    
- `hostname`
    
- `capsh`
    
- `newuidmap` et `newgidmap`
    

Objectif pratique :

```bash
unshare --fork --pid --mount-proc bash
nsenter --target <PID> --pid --mount --uts --ipc --net bash
lsns
```

Nous apprenons à créer, entrer et inspecter des namespaces sans passer par Docker.

---

# Chapitre 4 — Namespace UTS

- Rôle du namespace UTS
    
- Isolation du hostname
    
- Isolation du domainname
    
- Différence entre hostname global et hostname dans un conteneur
    
- Expérience avec `unshare --uts`
    
- Lien avec Docker `--hostname`
    

Commandes :

```bash
hostname
unshare --uts bash
hostname conteneur-test
hostname
exit
hostname
```

Objectif : comprendre qu’un processus peut voir un hostname différent de celui de l’hôte.

---

# Chapitre 5 — Namespace PID

- Rôle du namespace PID
    
- PID vus depuis l’hôte et depuis le namespace
    
- Le processus PID 1 dans un namespace
    
- Responsabilités particulières du PID 1
    
- Gestion des signaux
    
- Processus zombies et réapage
    
- `/proc` dans un namespace PID
    
- Pourquoi un conteneur a souvent son propre PID 1
    

Commandes :

```bash
unshare --fork --pid --mount-proc bash
ps aux
echo $$
cat /proc/1/comm
```

Points essentiels :

- un même processus peut avoir plusieurs PID selon le namespace ;
    
- `/proc` doit être remonté pour refléter correctement le namespace PID ;
    
- le PID 1 a un comportement particulier vis-à-vis des signaux et des zombies.
    

---

# Chapitre 6 — Namespace mount

- Rôle du namespace de montage
    
- Isolation des points de montage
    
- Propagation des montages
    
- `private`, `shared`, `slave`, `unbindable`
    
- `pivot_root`
    
- `chroot` vs namespace mount
    
- Montage de `/proc`
    
- Montage de `/sys`
    
- Bind mounts
    
- Root filesystem d’un conteneur
    

Commandes :

```bash
unshare --mount bash
mount --make-rprivate /
mkdir /tmp/test-mnt
mount -t tmpfs tmpfs /tmp/test-mnt
findmnt /tmp/test-mnt
exit
findmnt /tmp/test-mnt
```

Objectif : comprendre qu’un processus peut voir une arborescence de montages différente de celle de l’hôte.

---

# Chapitre 7 — Namespace network

- Rôle du namespace réseau
    
- Interfaces réseau isolées
    
- Loopback propre au namespace
    
- Tables de routage séparées
    
- Firewall et règles réseau par namespace
    
- Sockets isolées
    
- Création d’une paire `veth`
    
- Pont réseau Linux
    
- NAT
    
- Lien avec Docker bridge
    
- Lien avec Kubernetes CNI
    

Commandes :

```bash
unshare --net bash
ip addr
ip link set lo up
ip addr
```

TP plus avancé :

```bash
ip netns add ns1
ip link add veth-host type veth peer name veth-ns
ip link set veth-ns netns ns1
ip addr add 10.0.0.1/24 dev veth-host
ip link set veth-host up
ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
ip netns exec ns1 ip link set veth-ns up
ip netns exec ns1 ip link set lo up
ping 10.0.0.2
```

---

# Chapitre 8 — Namespace IPC

- Rôle du namespace IPC
    
- Isolation des mécanismes System V IPC
    
- Files de messages
    
- Sémaphores
    
- Mémoire partagée
    
- IPC POSIX
    
- Cas d’usage en conteneurs
    
- Risques de partage IPC avec l’hôte
    
- Option Docker `--ipc`
    

Commandes :

```bash
ipcs
unshare --ipc bash
ipcs
```

Objectif : comprendre que plusieurs processus peuvent être isolés au niveau de leurs mécanismes de communication inter-processus.

---

# Chapitre 9 — Namespace user

- Rôle du namespace user
    
- Mapping UID/GID
    
- Être root dans un namespace sans être root sur l’hôte
    
- `/proc/<PID>/uid_map`
    
- `/proc/<PID>/gid_map`
    
- Capabilities dans un user namespace
    
- Rootless containers
    
- Sécurité et risques
    
- `newuidmap`, `newgidmap`
    
- `/etc/subuid` et `/etc/subgid`
    

Commandes :

```bash
unshare --user --map-root-user bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Point central :

```text
Nous pouvons être UID 0 dans le namespace tout en restant un utilisateur non privilégié sur l’hôte.
```

C’est un mécanisme clé des conteneurs rootless.

---

# Chapitre 10 — Namespace cgroup

- Rôle du namespace cgroup
    
- Isolation de la vue des cgroups
    
- Différence entre cgroup namespace et mécanisme cgroups
    
- `/proc/self/cgroup`
    
- Intérêt pour les conteneurs
    
- Masquage partiel de l’organisation de l’hôte
    
- Lien avec systemd et Kubernetes
    

Commandes :

```bash
cat /proc/self/cgroup
lsns -t cgroup
```

Objectif : comprendre que le namespace cgroup isole la vue, mais que les limites de ressources sont définies par les cgroups eux-mêmes.

---

# Chapitre 11 — Namespace time

- Rôle du namespace time
    
- Isolation partielle des horloges
    
- `CLOCK_MONOTONIC`
    
- `CLOCK_BOOTTIME`
    
- Différence avec l’heure réelle du système
    
- Cas d’usage : tests, checkpoint/restore, environnements isolés
    
- Limites du namespace time
    

Commandes exploratoires :

```bash
ls -l /proc/$$/ns/time
cat /proc/self/timens_offsets
```

Objectif : comprendre que le time namespace ne permet pas simplement de changer l’heure système globale, mais d’ajuster certaines horloges vues par les processus.

---

# Chapitre 12 — Namespaces et `/proc`

- `/proc/<PID>/ns`
    
- Lire les namespaces d’un processus
    
- Comparer deux processus
    
- PID namespace et `/proc`
    
- Network namespace et `/proc/net`
    
- Mount namespace et `/proc/mounts`
    
- User namespace et UID/GID maps
    
- Sécurité des informations exposées dans `/proc`
    

Commandes :

```bash
ls -l /proc/<PID>/ns
readlink /proc/<PID>/ns/net
readlink /proc/<PID>/ns/pid
cat /proc/<PID>/uid_map
cat /proc/<PID>/gid_map
```

Ce chapitre fait le lien direct avec le cours sur `/proc`.

---

# Chapitre 13 — Namespaces et cgroups

- Différence entre isolation et limitation
    
- Namespaces : ce que le processus voit
    
- Cgroups : ce que le processus peut consommer
    
- CPU, mémoire, I/O, pids
    
- cgroups v1 vs cgroups v2
    
- systemd comme gestionnaire de cgroups
    
- Kubernetes requests et limits
    
- OOMKilled
    
- Exemple de conteneur avec namespace isolé mais ressources limitées
    

Synthèse :

```text
Namespaces = isolation des vues.
Cgroups = contrôle des ressources.
Capabilities = contrôle des privilèges.
Seccomp/AppArmor/SELinux = réduction de la surface d’attaque.
```

---

# Chapitre 14 — Namespaces, capabilities et sécurité

- Pourquoi les namespaces ne suffisent pas
    
- Capabilities Linux
    
- `CAP_SYS_ADMIN`
    
- Capabilities dans un user namespace
    
- Seccomp
    
- AppArmor
    
- SELinux
    
- Risques des conteneurs privilégiés
    
- Montage dangereux de `/proc`, `/sys`, `/dev`
    
- Évasion de conteneur : principes généraux sans exploitation
    
- Bonnes pratiques de durcissement
    

Commandes :

```bash
capsh --print
grep Cap /proc/$$/status
```

Objectif : comprendre que l’isolation repose sur plusieurs couches, pas uniquement sur les namespaces.

---

# Chapitre 15 — Des namespaces aux conteneurs

- Comment Docker combine les namespaces
    
- Ce que fait un runtime OCI
    
- Image, root filesystem et mount namespace
    
- Processus principal et PID namespace
    
- Réseau bridge et network namespace
    
- Volumes et bind mounts
    
- Variables d’environnement
    
- Entrypoint et PID 1
    
- Différence entre image et conteneur
    
- Rôle de `runc`, containerd, Docker
    

Démonstration :

```bash
docker run --rm -it alpine sh
ps
hostname
ip addr
mount
cat /proc/1/cgroup
```

Puis comparaison depuis l’hôte :

```bash
docker inspect <container>
lsns
ps aux | grep <process>
```

---

# Chapitre 16 — Namespaces dans Kubernetes

- Pod comme unité de partage de certains namespaces
    
- Containers d’un même pod
    
- Partage du network namespace dans un pod
    
- Pause container
    
- PID namespace dans Kubernetes
    
- Volumes et mount namespaces
    
- CNI et network namespaces
    
- SecurityContext
    
- Privileged containers
    
- hostNetwork, hostPID, hostIPC
    
- Debug avec ephemeral containers
    

Points importants :

```text
Dans un pod Kubernetes, plusieurs containers partagent généralement le même namespace réseau.
Ils ont donc la même IP et peuvent communiquer via localhost.
```

---

# Chapitre 17 — Travaux pratiques

## TP 1 — Observer les namespaces d’un processus

- Lister `/proc/$$/ns`
    
- Comparer avec `/proc/1/ns`
    
- Utiliser `lsns`
    
- Identifier les namespaces partagés ou distincts
    

## TP 2 — Créer un namespace UTS

- Créer un shell isolé
    
- Modifier le hostname
    
- Vérifier que l’hôte n’est pas modifié
    

## TP 3 — Créer un namespace PID

- Créer un PID namespace
    
- Monter `/proc`
    
- Observer le PID 1
    
- Comprendre les limites du PID 1
    

## TP 4 — Créer un namespace réseau

- Créer deux namespaces réseau
    
- Connecter avec une paire `veth`
    
- Configurer IP et routes
    
- Tester avec `ping`
    

## TP 5 — Créer un environnement pseudo-conteneur

- Namespace mount
    
- Namespace UTS
    
- Namespace PID
    
- Montage de `/proc`
    
- Root filesystem minimal
    
- Lancement d’un shell isolé
    

## TP 6 — User namespace rootless

- Créer un user namespace
    
- Mapper l’utilisateur courant vers root
    
- Observer `uid_map`
    
- Tester les limites de privilèges
    

---

# Chapitre 18 — Étude de cas : construire un mini-conteneur

Nous construisons progressivement un mini-conteneur à la main, sans Docker.

Étapes :

1. Préparer un root filesystem minimal.
    
2. Créer un namespace UTS.
    
3. Créer un namespace PID.
    
4. Créer un namespace mount.
    
5. Monter `/proc`.
    
6. Isoler le réseau.
    
7. Ajouter une paire `veth`.
    
8. Configurer une route.
    
9. Ajouter éventuellement un user namespace.
    
10. Lancer un processus isolé.
    
11. Comparer avec un vrai conteneur Docker.
    

Objectif : comprendre concrètement ce qu’un runtime de conteneur automatise.

---

# Chapitre 19 — Limites des namespaces

- Les namespaces isolent les vues, pas tout le système
    
- Le noyau reste partagé
    
- Risques liés aux capabilities
    
- Risques liés aux montages
    
- Risques liés aux périphériques
    
- Risques liés à `/proc` et `/sys`
    
- Complexité des user namespaces
    
- Problèmes de performance et d’observabilité
    
- Debug plus difficile
    
- Écart entre isolation conteneur et isolation VM
    
- Cas où une VM reste préférable
    

---

# Chapitre 20 — Évaluation proposée

## Partie théorique

Questions possibles :

- Qu’est-ce qu’un namespace Linux ?
    
- Quelle différence entre namespace et cgroup ?
    
- Pourquoi le PID 1 est-il particulier dans un namespace PID ?
    
- Comment un processus peut-il être root dans un user namespace sans être root sur l’hôte ?
    
- Pourquoi un pod Kubernetes partage-t-il souvent le namespace réseau ?
    
- Quels risques présente un conteneur privilégié ?
    

## Partie pratique

Sujet possible :

```text
Nous créons un environnement isolé à l’aide de unshare et nsenter.
Nous devons isoler le hostname, les PID, les montages et le réseau.
Nous devons ensuite expliquer quelles parties sont réellement isolées et lesquelles ne le sont pas.
```

## Mini-projet

Construire un script `mini-container.sh` capable de :

- créer un namespace UTS ;
    
- créer un namespace PID ;
    
- créer un namespace mount ;
    
- monter `/proc` ;
    
- lancer un shell dans un root filesystem minimal ;
    
- afficher les namespaces actifs ;
    
- nettoyer correctement à la fin.
    

---

# Progression pédagogique proposée

## Partie I — Concepts fondamentaux

- Chapitres 1 à 3
    
- Objectif : comprendre ce que sont les namespaces et comment les observer.
    

## Partie II — Étude des namespaces un par un

- Chapitres 4 à 11
    
- Objectif : comprendre UTS, PID, mount, network, IPC, user, cgroup et time.
    

## Partie III — Intégration système

- Chapitres 12 à 16
    
- Objectif : relier namespaces, `/proc`, cgroups, sécurité, Docker et Kubernetes.
    

## Partie IV — Travaux pratiques et mini-conteneur

- Chapitres 17 à 20
    
- Objectif : manipuler directement les namespaces et construire une compréhension concrète des conteneurs.

# Chapitre 3 — Les outils de manipulation des namespaces

## Objectifs du chapitre

Dans ce chapitre, nous étudions les outils pratiques qui permettent de créer, observer et rejoindre des namespaces Linux.

Dans le chapitre précédent, nous avons vu qu’un processus appartient simultanément à plusieurs namespaces, observables dans :

```bash
/proc/<PID>/ns
```

Nous allons maintenant manipuler ces namespaces concrètement avec des commandes système.

À la fin de ce chapitre, nous savons :

- utiliser `unshare` pour créer de nouveaux namespaces ;
    
- utiliser `nsenter` pour entrer dans les namespaces d’un processus existant ;
    
- utiliser `lsns` pour lister les namespaces actifs ;
    
- utiliser `ip netns` pour manipuler des namespaces réseau nommés ;
    
- comprendre le rôle de `/proc/<PID>/ns` ;
    
- éviter les pièges classiques, notamment avec le namespace PID et `/proc` ;
    
- faire le lien entre ces outils et les conteneurs.
    

---

# 3.1. Vue d’ensemble des outils

## 3.1.1. Pourquoi avons-nous besoin d’outils ?

Les namespaces sont des objets du noyau. Nous ne les manipulons pas directement comme des fichiers classiques.

Pour les utiliser, nous avons besoin d’outils qui appellent les bons mécanismes noyau :

- `clone` pour créer un nouveau processus dans un ou plusieurs nouveaux namespaces ;
    
- `unshare` pour détacher un processus de certains namespaces ;
    
- `setns` pour rejoindre un namespace existant.
    

En pratique, nous utilisons surtout des commandes déjà disponibles sur la plupart des distributions Linux.

---

## 3.1.2. Les outils principaux

Nous allons étudier les outils suivants :

|Outil|Rôle|
|---|---|
|`unshare`|créer un shell ou une commande dans de nouveaux namespaces|
|`nsenter`|entrer dans les namespaces d’un processus existant|
|`lsns`|lister les namespaces visibles|
|`ip netns`|créer et gérer des namespaces réseau nommés|
|`ip`|configurer les interfaces, routes et namespaces réseau|
|`mount`|manipuler les montages dans un namespace mount|
|`findmnt`|observer les points de montage|
|`ps`|observer les processus dans un namespace PID|
|`capsh`|observer les capabilities|
|`newuidmap` / `newgidmap`|configurer des mappings UID/GID pour les user namespaces|

Ces outils ne sont pas tous au même niveau : certains créent des namespaces, d’autres les observent ou les configurent.

---

# 3.2. `unshare`

## 3.2.1. Rôle de `unshare`

La commande `unshare` lance un programme après avoir créé un ou plusieurs nouveaux namespaces.

Sa forme générale est :

```bash
unshare [options] commande
```

Exemple :

```bash
unshare --uts bash
```

Cette commande lance un shell `bash` dans un nouveau namespace UTS.

Dans ce shell, nous pouvons changer le hostname sans changer celui de l’hôte.

---

## 3.2.2. Options principales

Les options les plus importantes sont :

|Option|Namespace créé|
|---|---|
|`--uts`|namespace UTS|
|`--pid`|namespace PID|
|`--mount`|namespace mount|
|`--net`|namespace network|
|`--ipc`|namespace IPC|
|`--user`|namespace user|
|`--cgroup`|namespace cgroup|
|`--time`|namespace time|
|`--fork`|lance la commande dans un processus enfant|
|`--mount-proc`|monte `/proc` dans le nouveau namespace|
|`--map-root-user`|mappe l’utilisateur courant comme root dans le user namespace|

Nous utilisons souvent plusieurs options ensemble.

---

## 3.2.3. Créer un namespace UTS

Exemple simple :

```bash
hostname
unshare --uts bash
hostname ns-demo
hostname
exit
hostname
```

Interprétation :

1. Nous affichons le hostname de l’hôte.
    
2. Nous lançons un shell dans un nouveau namespace UTS.
    
3. Nous changeons le hostname dans ce namespace.
    
4. Nous vérifions que le hostname a changé dans le shell isolé.
    
5. Nous quittons.
    
6. Nous vérifions que le hostname de l’hôte n’a pas changé.
    

Le namespace UTS est donc un excellent premier exemple, car il est simple et visible.

---

## 3.2.4. Créer un namespace PID

Le namespace PID demande une attention particulière.

Commande correcte :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell obtenu :

```bash
echo $$
ps aux
cat /proc/1/comm
```

Nous observons généralement que le shell lancé est PID 1 dans le nouveau namespace.

Pourquoi utilisons-nous `--fork` ?

Parce que le nouveau namespace PID s’applique aux enfants du processus qui le crée. Le processus `unshare` lui-même ne devient pas directement PID 1 dans ce nouveau namespace.

Pourquoi utilisons-nous `--mount-proc` ?

Parce que `/proc` doit refléter le namespace PID courant. Sans remontage propre de `/proc`, les commandes comme `ps` peuvent afficher une vue incohérente.

Nous retenons :

```text
Pour manipuler un namespace PID proprement avec un shell :
unshare --fork --pid --mount-proc bash
```

---

## 3.2.5. Créer un namespace mount

Nous pouvons créer un namespace de montage :

```bash
unshare --mount bash
```

Dans ce shell, nous pouvons monter un système de fichiers temporaire :

```bash
mkdir -p /tmp/ns-mount-test
mount -t tmpfs tmpfs /tmp/ns-mount-test
findmnt /tmp/ns-mount-test
```

Puis nous quittons :

```bash
exit
```

Depuis l’hôte :

```bash
findmnt /tmp/ns-mount-test
```

Selon la propagation des montages, le montage peut ne pas être visible hors du namespace.

Cependant, sur certaines distributions, la propagation des montages peut créer des surprises. C’est pourquoi nous étudierons les notions `shared`, `private`, `slave` et `unbindable` dans le chapitre sur le namespace mount.

---

## 3.2.6. Créer un namespace réseau

Nous lançons :

```bash
unshare --net bash
```

Dans ce shell :

```bash
ip addr
```

Nous voyons généralement seulement l’interface loopback :

```text
lo
```

Elle peut être désactivée. Nous pouvons l’activer :

```bash
ip link set lo up
ip addr
```

Ce namespace réseau est isolé :

- il a ses propres interfaces ;
    
- il a ses propres routes ;
    
- il a ses propres sockets ;
    
- il a sa propre table réseau.
    

Mais il n’a pas automatiquement accès au réseau extérieur. Pour cela, nous devons configurer une paire `veth`, des routes et éventuellement du NAT.

---

## 3.2.7. Créer un namespace user

Nous pouvons créer un user namespace :

```bash
unshare --user --map-root-user bash
```

Dans ce shell :

```bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Nous pouvons voir :

```text
uid=0(root) gid=0(root)
```

Mais cela ne veut pas dire que nous sommes root sur l’hôte.

Nous sommes root dans le namespace utilisateur.

Le mapping peut ressembler à :

```text
         0       1000          1
```

Cela signifie :

```text
UID 0 dans le namespace → UID 1000 sur l’hôte
taille de plage : 1
```

Ce mécanisme est essentiel pour les conteneurs rootless.

---

## 3.2.8. Combiner plusieurs namespaces

Nous pouvons combiner plusieurs isolations :

```bash
unshare --fork --pid --mount --uts --ipc --mount-proc bash
```

Dans ce shell, nous pouvons vérifier :

```bash
ls -l /proc/$$/ns
hostname
ps aux
```

Nous avons un environnement déjà plus proche d’un conteneur, même s’il manque encore :

- un vrai root filesystem isolé ;
    
- une configuration réseau ;
    
- des cgroups ;
    
- une gestion fine des capabilities ;
    
- des restrictions seccomp ;
    
- une politique de sécurité.
    

---

# 3.3. `nsenter`

## 3.3.1. Rôle de `nsenter`

`nsenter` permet d’entrer dans les namespaces d’un processus existant.

La forme générale est :

```bash
nsenter --target <PID> [options] commande
```

Exemple :

```bash
sudo nsenter --target 1234 --net bash
```

Cette commande lance un shell dans le namespace réseau du processus `1234`.

---

## 3.3.2. Options principales

|Option|Effet|
|---|---|
|`--mount`|entrer dans le namespace mount|
|`--uts`|entrer dans le namespace UTS|
|`--ipc`|entrer dans le namespace IPC|
|`--net`|entrer dans le namespace réseau|
|`--pid`|entrer dans le namespace PID|
|`--user`|entrer dans le namespace user|
|`--cgroup`|entrer dans le namespace cgroup|
|`--time`|entrer dans le namespace time|
|`--target <PID>`|processus dont nous voulons rejoindre les namespaces|

Nous pouvons entrer dans plusieurs namespaces en même temps.

---

## 3.3.3. Entrer dans un namespace réseau

Supposons qu’un processus `1234` soit dans un namespace réseau isolé.

Nous pouvons faire :

```bash
sudo nsenter --target 1234 --net bash
```

Dans ce shell :

```bash
ip addr
ip route
ss -tulpen
```

Nous observons la pile réseau vue par le processus cible.

C’est très utile pour diagnostiquer un conteneur ou un service isolé.

---

## 3.3.4. Entrer dans plusieurs namespaces

Pour entrer dans un environnement plus complet :

```bash
sudo nsenter --target 1234 --mount --uts --ipc --net --pid bash
```

Dans ce shell, nous voyons une partie de l’environnement du processus cible.

Attention : entrer dans le namespace PID avec `nsenter` ne change pas toujours immédiatement la perception de tous les PID pour le shell déjà lancé. Le comportement exact dépend de l’utilisation de `--fork` et du processus enfant lancé. En pratique, pour une expérience plus cohérente, nous pouvons utiliser :

```bash
sudo nsenter --target 1234 --mount --uts --ipc --net --pid --fork bash
```

---

## 3.3.5. Utilisation avec les conteneurs Docker

Pour trouver le PID principal d’un conteneur Docker :

```bash
docker inspect --format '{{.State.Pid}}' <nom_ou_id_conteneur>
```

Puis :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid --fork bash
```

Cela permet d’entrer dans l’environnement du conteneur, même si l’image ne contient pas forcément les outils de debug habituels.

Nous utilisons cette technique avec prudence, car nous intervenons depuis l’hôte dans l’environnement du conteneur.

---

# 3.4. `lsns`

## 3.4.1. Rôle de `lsns`

`lsns` liste les namespaces visibles sur le système.

Commande simple :

```bash
lsns
```

Sortie typique :

```text
        NS TYPE   NPROCS   PID USER COMMAND
4026531834 time      120     1 root /sbin/init
4026531835 cgroup    120     1 root /sbin/init
4026531836 pid       120     1 root /sbin/init
4026531837 user      120     1 root /sbin/init
4026531838 uts       120     1 root /sbin/init
4026531839 ipc       120     1 root /sbin/init
4026531840 net       120     1 root /sbin/init
4026531841 mnt       120     1 root /sbin/init
```

---

## 3.4.2. Colonnes principales

|Colonne|Signification|
|---|---|
|`NS`|identifiant du namespace|
|`TYPE`|type de namespace|
|`NPROCS`|nombre de processus associés|
|`PID`|un PID associé au namespace|
|`USER`|utilisateur du processus affiché|
|`COMMAND`|commande associée|

`lsns` permet de repérer rapidement les namespaces présents, notamment ceux créés par des conteneurs.

---

## 3.4.3. Filtrer par type

Nous pouvons filtrer par type :

```bash
lsns -t pid
lsns -t net
lsns -t mnt
lsns -t uts
lsns -t user
```

Exemple :

```bash
lsns -t net
```

permet de lister les namespaces réseau.

C’est très utile lorsque nous voulons comprendre combien d’environnements réseau isolés existent sur une machine.

---

## 3.4.4. Voir les namespaces d’un processus précis

Nous pouvons combiner `lsns` avec `/proc`.

Pour un PID donné :

```bash
ls -l /proc/<PID>/ns
```

Puis comparer avec :

```bash
lsns
```

L’objectif est de comprendre où se trouve le processus dans l’ensemble des namespaces du système.

---

# 3.5. `ip netns`

## 3.5.1. Rôle de `ip netns`

`ip netns` est spécialisé dans les namespaces réseau.

Il permet de créer des namespaces réseau nommés.

Créer un namespace :

```bash
sudo ip netns add ns1
```

Lister :

```bash
ip netns list
```

Exécuter une commande dans le namespace :

```bash
sudo ip netns exec ns1 ip addr
```

Supprimer :

```bash
sudo ip netns delete ns1
```

---

## 3.5.2. Où sont stockés les namespaces nommés ?

Les références sont généralement visibles dans :

```bash
/run/netns
```

Nous pouvons regarder :

```bash
ls -l /run/netns
```

Ces fichiers servent de références au namespace réseau.

Tant que cette référence existe, le namespace peut rester vivant.

---

## 3.5.3. Créer un namespace réseau et activer loopback

Nous créons :

```bash
sudo ip netns add ns1
```

Nous observons :

```bash
sudo ip netns exec ns1 ip addr
```

Nous activons loopback :

```bash
sudo ip netns exec ns1 ip link set lo up
```

Nous vérifions :

```bash
sudo ip netns exec ns1 ip addr
```

Puis nous supprimons :

```bash
sudo ip netns delete ns1
```

Cet exercice montre qu’un namespace réseau commence souvent avec une pile réseau minimale.

---

## 3.5.4. Différence entre `unshare --net` et `ip netns`

`unshare --net` crée un namespace réseau pour un processus lancé.

`ip netns add` crée un namespace réseau nommé, avec une référence persistante dans `/run/netns`.

Nous pouvons résumer :

```text
unshare --net  → pratique pour lancer un shell ou une commande isolée
ip netns       → pratique pour créer des namespaces réseau nommés et réutilisables
```

Pour des laboratoires réseau, `ip netns` est souvent plus confortable.

---

# 3.6. `ip` pour configurer le réseau des namespaces

## 3.6.1. Pourquoi `ip` est indispensable ?

Créer un namespace réseau ne suffit pas.

Il faut souvent configurer :

- interfaces ;
    
- adresses IP ;
    
- routes ;
    
- liens virtuels ;
    
- ponts ;
    
- NAT ;
    
- règles de firewall.
    

L’outil principal est :

```bash
ip
```

---

## 3.6.2. Créer une paire `veth`

Une paire `veth` est comme un câble virtuel entre deux interfaces.

Nous créons :

```bash
sudo ip link add veth-host type veth peer name veth-ns
```

Nous obtenons deux interfaces liées :

```text
veth-host <── câble virtuel ──> veth-ns
```

Ce qui entre d’un côté ressort de l’autre.

---

## 3.6.3. Placer une interface dans un namespace

Nous créons un namespace :

```bash
sudo ip netns add ns1
```

Nous plaçons une extrémité dedans :

```bash
sudo ip link set veth-ns netns ns1
```

Nous configurons l’hôte :

```bash
sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up
```

Nous configurons le namespace :

```bash
sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
sudo ip netns exec ns1 ip link set lo up
```

Nous testons :

```bash
ping -c 3 10.0.0.2
```

Depuis le namespace :

```bash
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

Nous avons créé une communication réseau entre l’hôte et un namespace isolé.

---

# 3.7. `mount` et `findmnt`

## 3.7.1. Rôle de `mount`

Le namespace mount contrôle la vue des points de montage.

Nous utilisons `mount` pour créer ou modifier des montages.

Exemple :

```bash
unshare --mount bash
mkdir -p /tmp/mnt-test
mount -t tmpfs tmpfs /tmp/mnt-test
mount | grep mnt-test
```

---

## 3.7.2. Rôle de `findmnt`

`findmnt` permet d’observer les montages de manière plus lisible :

```bash
findmnt
findmnt /tmp/mnt-test
findmnt /proc
```

Nous l’utilisons souvent pour vérifier si un montage existe dans le namespace courant.

---

## 3.7.3. Propagation des montages

Un piège classique est la propagation des montages.

Selon la configuration, un montage créé dans un namespace peut être propagé ailleurs.

Pour éviter cela en TP, nous pouvons rendre les montages privés :

```bash
mount --make-rprivate /
```

Nous détaillerons ce point dans le chapitre sur le namespace mount.

Pour l’instant, nous retenons que le namespace mount ne se comprend pas correctement sans la notion de propagation.

---

# 3.8. `ps` et le namespace PID

## 3.8.1. Pourquoi `ps` peut être trompeur

`ps` lit les informations de `/proc`.

Si nous créons un namespace PID sans remonter `/proc`, `ps` peut afficher les processus de l’hôte au lieu de ceux du namespace.

Mauvais exemple pédagogique :

```bash
unshare --fork --pid bash
ps aux
```

Meilleur exemple :

```bash
unshare --fork --pid --mount-proc bash
ps aux
```

Nous voyons une liste cohérente avec le namespace PID.

---

## 3.8.2. Vérifier le PID 1

Dans le shell isolé :

```bash
echo $$
cat /proc/1/comm
ps aux
```

Nous devons comprendre que le PID 1 du namespace peut être notre shell.

Ce point est central pour comprendre les conteneurs.

Dans un conteneur, le processus principal est souvent PID 1 dans son namespace PID.

---

# 3.9. `capsh` et les capabilities

## 3.9.1. Pourquoi parler des capabilities dans un chapitre sur les outils ?

Les namespaces ne suffisent pas à comprendre les privilèges.

Un processus peut être dans un namespace isolé, mais ses permissions dépendent aussi des capabilities.

Nous pouvons afficher les capabilities avec :

```bash
capsh --print
```

Si `capsh` n’est pas installé, il peut être fourni par un paquet comme `libcap2-bin` sur Debian/Ubuntu.

---

## 3.9.2. Lire les capabilities dans `/proc`

Nous pouvons aussi lire :

```bash
grep Cap /proc/$$/status
```

Sortie possible :

```text
CapInh:  0000000000000000
CapPrm:  000001ffffffffff
CapEff:  000001ffffffffff
CapBnd:  000001ffffffffff
CapAmb:  0000000000000000
```

Ces valeurs sont en hexadécimal.

Elles décrivent les capabilities héritables, permises, effectives, bornées et ambiantes.

Nous approfondirons ce point dans le chapitre sur sécurité et capabilities.

---

# 3.10. `newuidmap` et `newgidmap`

## 3.10.1. Pourquoi ces outils existent-ils ?

Les user namespaces utilisent des mappings UID/GID.

Pour des mappings plus larges que le simple utilisateur courant, nous avons besoin de plages subordonnées définies dans :

```bash
/etc/subuid
/etc/subgid
```

Les outils :

```bash
newuidmap
newgidmap
```

permettent de configurer ces mappings de manière contrôlée.

Ils sont utilisés par des outils comme Podman rootless, Buildah ou certains runtimes rootless.

---

## 3.10.2. Observer les mappings

Dans un user namespace :

```bash
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Exemple simple :

```text
         0       1000          1
```

Exemple avec une plage plus large :

```text
         0     100000      65536
```

Cela signifie que les UID internes du namespace sont mappés vers une plage d’UID subordonnés sur l’hôte.

---

## 3.10.3. Pourquoi c’est important ?

Ce mécanisme permet d’avoir `root` dans le conteneur sans donner les privilèges root réels sur l’hôte.

C’est essentiel pour les conteneurs rootless.

Mais c’est aussi une source de complexité :

- permissions de fichiers ;
    
- volumes montés ;
    
- UID/GID inattendus ;
    
- problèmes avec les bind mounts ;
    
- interaction avec les systèmes de fichiers réseau.
    

Nous l’étudierons plus précisément dans le chapitre sur le user namespace.

---

# 3.11. Chaîne d’outils typique pour diagnostiquer un conteneur

## 3.11.1. Trouver le processus

Avec Docker :

```bash
docker ps
docker inspect --format '{{.State.Pid}}' <container>
```

Avec un processus quelconque :

```bash
ps aux | grep <nom>
```

---

## 3.11.2. Observer ses namespaces

```bash
sudo ls -l /proc/<PID>/ns
```

Comparer avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous voyons quels namespaces sont différents.

---

## 3.11.3. Entrer dans ses namespaces

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid --fork bash
```

Puis diagnostiquer :

```bash
hostname
ps aux
ip addr
mount
cat /proc/1/cgroup
```

Nous retrouvons une vue proche de celle du conteneur.

---

## 3.11.4. Vérifier les limites de ressources

Les namespaces ne suffisent pas.

Nous regardons aussi les cgroups :

```bash
cat /proc/1/cgroup
```

Sur cgroups v2 :

```bash
cat /sys/fs/cgroup/memory.current 2>/dev/null
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/cpu.max 2>/dev/null
```

Nous faisons le lien avec le cours sur les cgroups.

---

# 3.12. Pièges classiques avec les outils

## 3.12.1. Oublier les droits

Certaines commandes nécessitent des droits administrateur :

```bash
unshare --net bash
ip link add ...
nsenter --target <PID> --net bash
mount -t tmpfs ...
```

Selon la distribution et la configuration, un utilisateur non privilégié peut créer certains namespaces, notamment user namespaces, mais pas tous.

---

## 3.12.2. Oublier `--fork` avec PID namespace

Commande à éviter pour débuter :

```bash
unshare --pid bash
```

Commande recommandée :

```bash
unshare --fork --pid --mount-proc bash
```

Le namespace PID a une sémantique particulière : les enfants entrent dans le nouveau namespace.

---

## 3.12.3. Oublier de monter `/proc`

Sans `/proc` cohérent, les outils comme `ps` peuvent mentir ou afficher la vue de l’hôte.

Nous utilisons :

```bash
--mount-proc
```

avec `unshare`.

---

## 3.12.4. Confondre namespace réseau nommé et processus vivant

Un namespace réseau créé par :

```bash
sudo ip netns add ns1
```

peut exister grâce à une référence dans `/run/netns`, même si aucun processus ne tourne dedans.

C’est différent d’un namespace créé uniquement pour un processus avec `unshare --net`.

---

## 3.12.5. Croire que `nsenter` crée une isolation

`nsenter` ne crée pas un nouveau namespace.

Il entre dans un namespace existant.

Pour créer, nous utilisons plutôt :

```bash
unshare
ip netns add
```

Pour rejoindre, nous utilisons :

```bash
nsenter
ip netns exec
```

---

# 3.13. Exercices

## Exercice 1 — Créer un namespace UTS avec `unshare`

Nous exécutons :

```bash
hostname
readlink /proc/$$/ns/uts
unshare --uts bash
hostname ns-outil
hostname
readlink /proc/$$/ns/uts
exit
hostname
readlink /proc/$$/ns/uts
```

Nous répondons :

1. L’identifiant du namespace UTS change-t-il ?
    
2. Le hostname change-t-il dans le shell isolé ?
    
3. Le hostname de l’hôte est-il modifié après `exit` ?
    
4. Quel outil avons-nous utilisé ?
    

---

## Exercice 2 — Créer un namespace PID correctement

Nous exécutons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
echo $$
cat /proc/1/comm
ps aux
ls -l /proc/$$/ns/pid
```

Nous répondons :

1. Quel est le PID du shell ?
    
2. Quel est le processus PID 1 ?
    
3. Pourquoi `--fork` est-il nécessaire ?
    
4. Pourquoi `--mount-proc` est-il important ?
    

Nous quittons :

```bash
exit
```

---

## Exercice 3 — Lister les namespaces avec `lsns`

Nous exécutons :

```bash
lsns
lsns -t pid
lsns -t net
lsns -t uts
lsns -t mnt
```

Nous répondons :

1. Quels types de namespaces voyons-nous ?
    
2. Combien de namespaces réseau existent ?
    
3. Certains namespaces semblent-ils liés à des conteneurs ?
    
4. Que signifie la colonne `NPROCS` ?
    

---

## Exercice 4 — Créer un namespace réseau nommé

Nous exécutons :

```bash
sudo ip netns add ns1
ip netns list
ls -l /run/netns
sudo ip netns exec ns1 ip addr
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns1 ip addr
sudo ip netns delete ns1
```

Nous répondons :

1. Où apparaît le namespace nommé ?
    
2. Quelle interface est visible au départ ?
    
3. Pourquoi activons-nous `lo` ?
    
4. Que fait `ip netns exec` ?
    

---

## Exercice 5 — Créer une paire `veth`

Nous exécutons :

```bash
sudo ip netns add ns1
sudo ip link add veth-host type veth peer name veth-ns
sudo ip link set veth-ns netns ns1

sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up

sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
sudo ip netns exec ns1 ip link set lo up

ping -c 3 10.0.0.2
sudo ip netns exec ns1 ping -c 3 10.0.0.1
```

Nettoyage :

```bash
sudo ip netns delete ns1
sudo ip link delete veth-host 2>/dev/null
```

Nous répondons :

1. À quoi sert une paire `veth` ?
    
2. Quelle interface est côté hôte ?
    
3. Quelle interface est côté namespace ?
    
4. Pourquoi pouvons-nous faire un ping entre les deux ?
    

---

## Exercice 6 — Entrer dans les namespaces d’un processus

Dans un premier terminal :

```bash
unshare --uts bash
hostname nsenter-demo
echo $$
sleep 1000
```

Dans un second terminal, avec le PID du shell isolé :

```bash
sudo nsenter --target <PID> --uts bash
hostname
exit
```

Nous répondons :

1. Quel hostname voyons-nous après `nsenter` ?
    
2. Avons-nous créé un nouveau namespace ou rejoint un namespace existant ?
    
3. Quelle option de `nsenter` avons-nous utilisée ?
    

---

# 3.14. Ce que nous devons retenir

Nous retenons les points suivants :

1. `unshare` sert à créer de nouveaux namespaces pour une commande.
    
2. `nsenter` sert à entrer dans les namespaces d’un processus existant.
    
3. `lsns` sert à lister les namespaces visibles.
    
4. `ip netns` sert à manipuler des namespaces réseau nommés.
    
5. `ip` est indispensable pour configurer les interfaces et routes dans les namespaces réseau.
    
6. `mount` et `findmnt` sont essentiels pour comprendre les namespaces mount.
    
7. `ps` dépend de `/proc`, donc `/proc` doit être cohérent avec le namespace PID.
    
8. `capsh` et `/proc/<PID>/status` permettent d’observer les capabilities.
    
9. `newuidmap` et `newgidmap` servent aux mappings avancés des user namespaces.
    
10. Le namespace PID nécessite souvent `--fork`.
    
11. Pour un namespace PID, nous utilisons souvent `--mount-proc`.
    
12. `nsenter` ne crée pas une isolation, il rejoint une isolation existante.
    
13. Les namespaces réseau nommés persistent grâce à des références dans `/run/netns`.
    
14. Les outils de namespaces sont puissants, mais demandent une grande prudence.
    

---

# Conclusion du chapitre 3

Nous avons étudié les principaux outils qui permettent de manipuler les namespaces Linux.

Nous savons maintenant créer des namespaces avec `unshare`, les observer avec `lsns` et `/proc/<PID>/ns`, les rejoindre avec `nsenter`, et manipuler les namespaces réseau avec `ip netns`.

Nous avons aussi vu plusieurs pièges importants : le rôle de `--fork` avec le namespace PID, la nécessité de remonter `/proc`, la différence entre créer et rejoindre un namespace, et la différence entre namespace réseau temporaire et namespace réseau nommé.

Dans le chapitre suivant, nous commençons l’étude détaillée des namespaces un par un avec le namespace UTS, qui isole le hostname et le domainname.

# Chapitre 4 — Namespace UTS

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace UTS.

Le namespace UTS est l’un des namespaces les plus simples à comprendre. Il permet d’isoler deux informations du système :

- le hostname ;
    
- le domainname, aussi appelé nom de domaine NIS dans certains contextes historiques.
    

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace UTS ;
    
- comprendre pourquoi le hostname peut être différent dans un conteneur ;
    
- créer un namespace UTS avec `unshare` ;
    
- modifier le hostname dans un namespace isolé ;
    
- comparer les namespaces UTS de deux processus ;
    
- faire le lien avec Docker, Podman et Kubernetes ;
    
- comprendre les limites de cette isolation.
    

---

# 4.1. Qu’est-ce que le namespace UTS ?

## 4.1.1. Définition

Le namespace UTS isole l’identité système visible par un processus.

Concrètement, il isole principalement :

```text
hostname
domainname
```

Le hostname correspond au nom de la machine tel qu’il est vu par les processus.

Nous pouvons le lire avec :

```bash
hostname
```

ou :

```bash
cat /proc/sys/kernel/hostname
```

Sur une machine classique, cela peut donner :

```text
serveur-prod-01
```

Dans un conteneur, cela peut donner autre chose, par exemple :

```text
b7f3c2a91d4e
```

Cela ne signifie pas que le conteneur a son propre noyau. Cela signifie qu’il a une vue UTS différente.

---

## 4.1.2. Origine du nom UTS

UTS vient historiquement de “UNIX Time-Sharing System”.

Aujourd’hui, dans le contexte des namespaces Linux, nous retenons surtout que le namespace UTS isole le nom de la machine.

Il ne faut pas le confondre avec le namespace time, qui concerne certaines horloges.

Le namespace UTS ne sert pas à gérer le temps. Il sert à isoler l’identité système visible.

---

# 4.2. Le hostname

## 4.2.1. Lire le hostname

Nous pouvons lire le hostname avec :

```bash
hostname
```

Exemple :

```text
machine-hote
```

Nous pouvons aussi lire :

```bash
cat /proc/sys/kernel/hostname
```

Ces deux commandes donnent normalement le hostname actif du namespace UTS courant.

Cela signifie que si nous sommes dans un namespace UTS isolé, nous voyons le hostname de ce namespace, pas forcément celui de l’hôte.

---

## 4.2.2. Modifier le hostname

Sur un système classique, le hostname peut être modifié avec :

```bash
sudo hostname nouveau-nom
```

ou, sur un système avec systemd :

```bash
sudo hostnamectl set-hostname nouveau-nom
```

Mais dans un namespace UTS isolé, modifier le hostname ne modifie que la vue du namespace.

C’est précisément ce que nous voulons expérimenter.

---

## 4.2.3. Hostname actif et configuration persistante

Nous distinguons deux choses :

```text
hostname actif
configuration persistante du hostname
```

Le hostname actif peut être lu dans :

```bash
cat /proc/sys/kernel/hostname
```

La configuration persistante dépend de la distribution. Sur beaucoup de distributions, elle se trouve dans :

```bash
/etc/hostname
```

Mais dans un conteneur, cette distinction peut être différente, car le hostname est souvent injecté au lancement par le runtime.

Nous retenons :

```text
Le namespace UTS isole le hostname actif vu par les processus.
Il ne crée pas à lui seul une configuration persistante complète.
```

---

# 4.3. Le domainname

## 4.3.1. Lire le domainname

Le namespace UTS isole aussi le domainname.

Nous pouvons le lire avec :

```bash
domainname
```

ou :

```bash
cat /proc/sys/kernel/domainname
```

Sur beaucoup de machines modernes, nous pouvons obtenir :

```text
(none)
```

ou une valeur vide selon les outils et distributions.

Le domainname ici est historiquement lié au domaine NIS/YP, et non nécessairement au nom DNS complet de la machine.

---

## 4.3.2. À ne pas confondre avec DNS

Le domainname du namespace UTS n’est pas forcément le domaine DNS utilisé pour résoudre les noms.

Par exemple, une machine peut avoir :

```text
hostname : serveur1
domainname UTS : (none)
nom DNS complet : serveur1.example.org
```

La résolution DNS dépend plutôt de fichiers et services comme :

```text
/etc/hosts
/etc/resolv.conf
systemd-resolved
DNS externe
```

Le namespace UTS ne suffit donc pas à isoler toute la configuration de résolution de noms.

---

# 4.4. Observer le namespace UTS d’un processus

## 4.4.1. Avec `/proc/<PID>/ns/uts`

Comme les autres namespaces, le namespace UTS d’un processus est visible dans :

```bash
/proc/<PID>/ns/uts
```

Pour notre shell courant :

```bash
readlink /proc/$$/ns/uts
```

Sortie possible :

```text
uts:[4026531838]
```

Pour le PID 1 :

```bash
readlink /proc/1/ns/uts
```

Si les deux valeurs sont identiques, notre shell et le PID 1 partagent le même namespace UTS.

---

## 4.4.2. Comparer deux processus

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/uts)"
echo "PID 1         : $(readlink /proc/1/ns/uts)"
```

Exemple :

```text
Shell courant : uts:[4026531838]
PID 1         : uts:[4026531838]
```

Cela signifie que les deux processus partagent le même namespace UTS.

Si nous voyons :

```text
Shell courant : uts:[4026532500]
PID 1         : uts:[4026531838]
```

cela signifie que le shell courant est dans un namespace UTS différent de celui du PID 1.

---

# 4.5. Créer un namespace UTS avec `unshare`

## 4.5.1. Expérience simple

Nous commençons par afficher le hostname de l’hôte :

```bash
hostname
readlink /proc/$$/ns/uts
```

Exemple :

```text
machine-hote
uts:[4026531838]
```

Nous créons ensuite un nouveau namespace UTS :

```bash
unshare --uts bash
```

Dans ce nouveau shell :

```bash
readlink /proc/$$/ns/uts
hostname
```

Nous devons voir un identifiant UTS différent, mais le hostname initial peut encore être identique.

Le namespace est différent, mais il a été initialisé avec une copie de la valeur existante.

---

## 4.5.2. Modifier le hostname dans le namespace

Dans le shell isolé :

```bash
hostname ns-uts-demo
hostname
cat /proc/sys/kernel/hostname
```

Nous obtenons :

```text
ns-uts-demo
```

Nous quittons :

```bash
exit
```

Puis, sur l’hôte :

```bash
hostname
cat /proc/sys/kernel/hostname
```

Nous retrouvons le hostname de l’hôte.

Conclusion :

```text
Le hostname a changé dans le namespace UTS isolé.
Le hostname de l’hôte n’a pas été modifié.
```

---

## 4.5.3. Pourquoi cela fonctionne-t-il ?

Quand nous faisons :

```bash
unshare --uts bash
```

le nouveau shell ne partage plus le namespace UTS de son parent.

Il reçoit une nouvelle instance de namespace UTS.

Cette instance contient initialement les mêmes valeurs, mais elles peuvent ensuite évoluer séparément.

Nous avons donc :

```text
avant unshare :
shell et hôte partagent le même namespace UTS

après unshare :
shell isolé et hôte ont deux namespaces UTS différents
```

---

# 4.6. Permissions nécessaires

## 4.6.1. Création d’un namespace UTS

Selon la distribution et la configuration du noyau, créer un namespace UTS peut nécessiter des privilèges.

La commande suivante peut échouer :

```bash
unshare --uts bash
```

Erreur possible :

```text
unshare: unshare failed: Operation not permitted
```

Dans ce cas, nous pouvons avoir besoin de droits administrateur :

```bash
sudo unshare --uts bash
```

Ou bien nous pouvons utiliser un user namespace combiné, selon la configuration :

```bash
unshare --user --map-root-user --uts bash
```

---

## 4.6.2. Modifier le hostname

Modifier le hostname demande normalement la capacité :

```text
CAP_SYS_ADMIN
```

dans le namespace utilisateur qui gouverne le namespace UTS.

Cela signifie qu’un utilisateur non privilégié ne peut pas toujours modifier le hostname directement.

Mais dans un user namespace avec mapping root :

```bash
unshare --user --map-root-user --uts bash
```

nous pouvons parfois être `root` dans le namespace et disposer des capabilities nécessaires à l’intérieur de ce contexte.

Nous vérifions :

```bash
id
hostname ns-demo
```

Si cela fonctionne, le changement reste limité au namespace UTS créé.

---

# 4.7. Lien avec Docker et Podman

## 4.7.1. Hostname dans un conteneur Docker

Quand nous lançons un conteneur :

```bash
docker run --rm alpine hostname
```

Docker donne souvent au conteneur un hostname dérivé de son identifiant.

Exemple :

```text
b7f3c2a91d4e
```

Ce hostname n’est pas celui de l’hôte.

Cela s’explique par l’utilisation d’un namespace UTS isolé.

---

## 4.7.2. Choisir le hostname d’un conteneur

Avec Docker :

```bash
docker run --rm --hostname mon-conteneur alpine hostname
```

Sortie :

```text
mon-conteneur
```

Nous pouvons aussi entrer dans un shell :

```bash
docker run --rm -it --hostname mon-conteneur alpine sh
```

Puis :

```sh
hostname
cat /proc/sys/kernel/hostname
```

Nous voyons le hostname du conteneur.

---

## 4.7.3. Partager le namespace UTS de l’hôte

Docker permet de ne pas isoler le namespace UTS avec certaines options avancées, selon le runtime et la configuration.

Le principe à retenir est le suivant :

```text
Si le conteneur partage le namespace UTS de l’hôte,
changer le hostname dans le conteneur peut affecter l’hôte.
```

C’est une configuration à éviter sauf besoin très particulier.

En pratique, les conteneurs standards ont leur propre namespace UTS.

---

## 4.7.4. Podman rootless

Podman peut créer des conteneurs rootless.

Dans ce cas, l’isolation UTS fonctionne en combinaison avec un user namespace.

Nous retrouvons les mêmes principes :

- le conteneur a un hostname isolé ;
    
- l’utilisateur peut être root dans le conteneur ;
    
- les privilèges réels sur l’hôte restent limités.
    

---

# 4.8. Lien avec Kubernetes

## 4.8.1. Hostname dans un pod

Dans Kubernetes, chaque pod dispose généralement de son propre hostname.

Dans un conteneur d’un pod, nous pouvons faire :

```sh
hostname
```

Le résultat est souvent lié au nom du pod.

Kubernetes configure cette identité au lancement du pod.

---

## 4.8.2. Containers d’un même pod

Dans Kubernetes, plusieurs containers d’un même pod partagent certains namespaces, notamment souvent le namespace réseau.

Pour le namespace UTS, la gestion dépend du runtime et de la configuration du pod, mais du point de vue applicatif, les containers d’un même pod partagent généralement une identité cohérente liée au pod.

Nous retenons surtout que Kubernetes ne crée pas une machine virtuelle par pod. Il organise des processus avec des namespaces, des cgroups et d’autres mécanismes du noyau.

---

## 4.8.3. Hostname, DNS et service discovery

Dans Kubernetes, le hostname seul ne suffit pas à comprendre la découverte de services.

Nous devons distinguer :

- hostname du pod ;
    
- nom du pod ;
    
- nom du service Kubernetes ;
    
- DNS interne du cluster ;
    
- namespace Kubernetes ;
    
- domaine cluster, souvent `cluster.local`.
    

Le namespace UTS isole le hostname, mais Kubernetes ajoute une couche complète de DNS et d’orchestration.

---

# 4.9. Lien avec `/proc`

## 4.9.1. Lire le hostname via `/proc`

Le hostname actif du namespace UTS courant est visible dans :

```bash
cat /proc/sys/kernel/hostname
```

Dans un namespace UTS isolé :

```bash
unshare --uts bash
hostname ns-proc-demo
cat /proc/sys/kernel/hostname
```

Nous voyons :

```text
ns-proc-demo
```

Après `exit`, sur l’hôte :

```bash
cat /proc/sys/kernel/hostname
```

Nous retrouvons le hostname de l’hôte.

---

## 4.9.2. Lire le namespace UTS via `/proc/<PID>/ns`

Nous pouvons voir le namespace UTS courant :

```bash
readlink /proc/$$/ns/uts
```

Dans deux terminaux différents, nous pouvons comparer les valeurs.

Dans un namespace UTS créé avec `unshare`, l’identifiant change.

---

## 4.9.3. `/proc/sys/kernel/hostname` dépend du namespace

C’est un point important.

Le chemin :

```bash
/proc/sys/kernel/hostname
```

est le même, mais son contenu dépend du namespace UTS du processus qui le lit.

Nous pouvons donc avoir :

```text
même chemin
contenu différent selon le namespace
```

C’est une excellente illustration de l’idée de namespace.

---

# 4.10. Expérience complète guidée

## 4.10.1. Terminal 1 : créer un namespace UTS

Nous lançons :

```bash
echo "Avant :"
hostname
readlink /proc/$$/ns/uts

unshare --uts bash
```

Dans le shell isolé :

```bash
echo "Dans le namespace :"
hostname
readlink /proc/$$/ns/uts

hostname ns-formation
hostname
cat /proc/sys/kernel/hostname

echo $$
sleep 1000
```

Nous gardons ce shell ouvert.

---

## 4.10.2. Terminal 2 : observer depuis l’hôte

Dans un second terminal, nous cherchons le processus :

```bash
ps aux | grep sleep
```

Ou bien nous utilisons le PID affiché dans le terminal 1.

Nous comparons :

```bash
readlink /proc/<PID>/ns/uts
readlink /proc/1/ns/uts
```

Nous pouvons aussi entrer dans le namespace UTS :

```bash
sudo nsenter --target <PID> --uts bash
hostname
```

Nous devons voir :

```text
ns-formation
```

Nous quittons le shell `nsenter` :

```bash
exit
```

---

## 4.10.3. Nettoyage

Dans le terminal 1, nous arrêtons :

```bash
exit
```

ou nous tuons le processus `sleep` si nécessaire :

```bash
kill <PID>
```

Quand le dernier processus du namespace disparaît, le namespace UTS disparaît également, sauf s’il est maintenu par une autre référence.

---

# 4.11. Limites du namespace UTS

## 4.11.1. Il n’isole pas le réseau

Changer le hostname ne crée pas une nouvelle pile réseau.

Pour isoler les interfaces, les routes, les ports et les sockets, nous avons besoin du namespace réseau.

Le namespace UTS ne fait que modifier l’identité système visible.

---

## 4.11.2. Il n’isole pas le système de fichiers

Le namespace UTS ne change pas l’arborescence de fichiers.

Même si le hostname est différent, les processus peuvent encore voir les mêmes fichiers si le namespace mount n’est pas isolé.

Pour isoler les montages, nous avons besoin du namespace mount.

---

## 4.11.3. Il n’isole pas les processus

Le namespace UTS ne modifie pas la table des processus visible.

Pour isoler les PID, nous avons besoin du namespace PID.

Exemple :

```bash
unshare --uts bash
ps aux
```

Nous voyons encore les processus du même namespace PID que le parent.

---

## 4.11.4. Il ne limite aucune ressource

Le namespace UTS ne limite pas :

- CPU ;
    
- mémoire ;
    
- nombre de processus ;
    
- I/O disque ;
    
- réseau.
    

Pour limiter les ressources, nous avons besoin des cgroups.

---

# 4.12. Cas d’usage du namespace UTS

## 4.12.1. Donner une identité à un conteneur

Le cas d’usage principal est simple : donner un hostname propre à un environnement isolé.

Un conteneur peut ainsi croire qu’il s’appelle :

```text
app-frontend-1
```

alors que l’hôte s’appelle :

```text
node-prod-42
```

Cela aide les applications, logs et outils système à avoir une identité cohérente.

---

## 4.12.2. Tests et environnements temporaires

Nous pouvons tester des applications qui lisent le hostname sans modifier l’hôte.

Exemple :

```bash
unshare --uts bash
hostname test-env
./application
```

L’application voit `test-env`, mais l’hôte n’est pas modifié.

---

## 4.12.3. Formation et démonstration

Le namespace UTS est idéal pour introduire les namespaces, car :

- il est simple ;
    
- il produit un effet visible immédiatement ;
    
- il ne demande pas de configuration réseau ;
    
- il illustre bien la notion de vue isolée.
    

---

# 4.13. Pièges classiques

## 4.13.1. Croire que le hostname DNS change automatiquement

Changer le hostname avec :

```bash
hostname ns-demo
```

ne modifie pas forcément :

- `/etc/hosts` ;
    
- `/etc/resolv.conf` ;
    
- les entrées DNS ;
    
- les services Kubernetes ;
    
- les certificats TLS ;
    
- les configurations applicatives.
    

Le hostname système est une chose. La résolution DNS en est une autre.

---

## 4.13.2. Croire que le conteneur a son propre noyau

Un hostname différent ne signifie pas que nous avons un noyau différent.

Dans un conteneur :

```sh
hostname
uname -a
```

Le hostname peut être propre au conteneur, mais `uname -a` affiche le noyau de l’hôte.

Nous retenons :

```text
Le namespace UTS isole l’identité visible.
Il ne virtualise pas le noyau.
```

---

## 4.13.3. Oublier les permissions

Si :

```bash
unshare --uts bash
```

ou :

```bash
hostname ns-demo
```

échoue avec :

```text
Operation not permitted
```

cela indique un problème de permissions ou de capabilities.

Nous pouvons tester avec :

```bash
sudo unshare --uts bash
```

ou, selon la configuration :

```bash
unshare --user --map-root-user --uts bash
```

---

# 4.14. Exercices

## Exercice 1 — Observer le namespace UTS courant

Nous exécutons :

```bash
hostname
cat /proc/sys/kernel/hostname
readlink /proc/$$/ns/uts
readlink /proc/1/ns/uts
```

Nous répondons :

1. Quel est le hostname courant ?
    
2. Le contenu de `/proc/sys/kernel/hostname` est-il identique ?
    
3. Le shell courant partage-t-il le namespace UTS avec le PID 1 ?
    
4. Que signifie une valeur identique ?
    

---

## Exercice 2 — Créer un namespace UTS

Nous exécutons :

```bash
hostname
readlink /proc/$$/ns/uts

unshare --uts bash
readlink /proc/$$/ns/uts
hostname ns-test
hostname
cat /proc/sys/kernel/hostname
exit

hostname
readlink /proc/$$/ns/uts
```

Nous répondons :

1. L’identifiant du namespace UTS change-t-il dans le shell isolé ?
    
2. Le hostname change-t-il dans le shell isolé ?
    
3. Le hostname de l’hôte change-t-il après `exit` ?
    
4. Pourquoi ?
    

---

## Exercice 3 — Comparer deux processus

Dans un terminal :

```bash
unshare --uts bash
hostname ns-comparaison
echo $$
sleep 1000
```

Dans un second terminal :

```bash
readlink /proc/<PID>/ns/uts
readlink /proc/1/ns/uts
```

Nous répondons :

1. Les identifiants sont-ils identiques ?
    
2. Que signifie la différence ?
    
3. Le processus isolé partage-t-il encore les autres namespaces ?
    
4. Comment pouvons-nous vérifier les autres namespaces ?
    

---

## Exercice 4 — Entrer dans un namespace UTS existant

Avec le PID du processus précédent :

```bash
sudo nsenter --target <PID> --uts bash
hostname
readlink /proc/$$/ns/uts
exit
```

Nous répondons :

1. Quel hostname voyons-nous ?
    
2. Avons-nous créé un nouveau namespace ou rejoint un namespace existant ?
    
3. Quel outil avons-nous utilisé ?
    
4. Quelle option de `nsenter` est nécessaire ?
    

---

## Exercice 5 — Comparaison avec Docker

Si Docker est disponible :

```bash
docker run --rm alpine hostname
docker run --rm --hostname cours-uts alpine hostname
```

Nous répondons :

1. Quel hostname Docker donne-t-il par défaut ?
    
2. Que change l’option `--hostname` ?
    
3. Quel namespace explique ce comportement ?
    
4. Le noyau est-il différent de celui de l’hôte ?
    

---

## Exercice 6 — Vérifier que UTS n’isole pas les processus

Nous exécutons :

```bash
unshare --uts bash
hostname ns-mais-pas-pid
ps aux | head
exit
```

Nous répondons :

1. Le hostname est-il isolé ?
    
2. La liste des processus est-elle isolée ?
    
3. Quel namespace faudrait-il ajouter pour isoler les PID ?
    
4. Quelle commande utiliserions-nous ?
    

---

# 4.15. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace UTS isole le hostname et le domainname.
    
2. Il ne crée pas un nouveau noyau.
    
3. Il ne modifie pas à lui seul le réseau, les processus ou les montages.
    
4. Le hostname actif est visible avec `hostname` et `/proc/sys/kernel/hostname`.
    
5. Le namespace UTS d’un processus est visible avec `/proc/<PID>/ns/uts`.
    
6. Deux processus partagent le même namespace UTS si `readlink /proc/<PID>/ns/uts` donne la même valeur.
    
7. `unshare --uts bash` permet de créer un nouveau namespace UTS.
    
8. Modifier le hostname dans ce namespace ne modifie pas celui de l’hôte.
    
9. Docker et Podman utilisent le namespace UTS pour donner un hostname propre aux conteneurs.
    
10. Kubernetes configure également une identité de pod, mais ajoute une couche DNS et orchestration.
    
11. Changer le hostname ne change pas automatiquement la résolution DNS.
    
12. Les permissions et capabilities peuvent empêcher la création ou la modification du namespace UTS.
    
13. Le namespace UTS est simple, mais il illustre parfaitement l’idée de vue isolée.
    

---

# Conclusion du chapitre 4

Nous avons étudié le namespace UTS, l’un des namespaces les plus simples et les plus pédagogiques.

Nous savons maintenant qu’il permet d’isoler le hostname et le domainname d’un groupe de processus. Nous avons vu comment créer un namespace UTS avec `unshare`, comment observer son identifiant avec `/proc/<PID>/ns/uts`, comment modifier le hostname dans une vue isolée, et comment entrer dans ce namespace avec `nsenter`.

Nous retenons surtout que le namespace UTS ne crée pas une machine virtuelle. Il ne change ni le noyau, ni la pile réseau, ni les processus visibles, ni les ressources disponibles. Il isole seulement une partie de l’identité système.

Dans le chapitre suivant, nous étudions le namespace PID, beaucoup plus central pour les conteneurs, car il permet à un processus d’être PID 1 dans son propre environnement.

# Chapitre 5 — Namespace PID

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace PID.

Le namespace PID est l’un des namespaces les plus importants pour comprendre les conteneurs. Il permet d’isoler la numérotation et la visibilité des processus. Grâce à lui, un processus peut être vu comme `PID 1` à l’intérieur d’un environnement isolé, tout en ayant un autre PID sur l’hôte.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace PID ;
    
- comprendre qu’un processus peut avoir plusieurs PID selon le namespace d’observation ;
    
- créer un namespace PID avec `unshare` ;
    
- comprendre pourquoi `--fork` est nécessaire ;
    
- comprendre pourquoi nous devons remonter `/proc` ;
    
- expliquer le rôle particulier du PID 1 ;
    
- identifier les problèmes liés aux signaux et aux processus zombies ;
    
- faire le lien avec Docker, Podman et Kubernetes.
    

---

# 5.1. Pourquoi isoler les PID ?

## 5.1.1. Le problème de la visibilité des processus

Sur une machine Linux classique, nous pouvons lister les processus avec :

```bash
ps aux
```

Nous voyons alors les processus du système :

```text
root           1  ...  /sbin/init
root         512  ...  systemd-journald
user        2042  ...  bash
user        2101  ...  firefox
```

Dans un conteneur, nous ne voulons généralement pas que l’application voie tous les processus de l’hôte.

Nous voulons plutôt qu’elle voie seulement les processus de son environnement.

Le namespace PID permet cette isolation.

---

## 5.1.2. Le principe

Le namespace PID isole la vue des identifiants de processus.

Cela signifie qu’un processus peut avoir :

- un PID dans le namespace de l’hôte ;
    
- un autre PID dans un namespace enfant ;
    
- éventuellement encore un autre PID dans un namespace plus profond.
    

Nous pouvons donc avoir une situation comme :

```text
Vue depuis l’hôte :
PID 24531 -> processus node

Vue depuis le conteneur :
PID 1 -> même processus node
```

C’est le même processus réel, mais il est vu avec des identifiants différents selon le namespace depuis lequel nous l’observons.

---

# 5.2. Observer le namespace PID courant

## 5.2.1. Avec `/proc/<PID>/ns/pid`

Nous pouvons observer le namespace PID de notre shell courant :

```bash
readlink /proc/$$/ns/pid
```

Exemple :

```text
pid:[4026531836]
```

Nous pouvons comparer avec le processus 1 :

```bash
readlink /proc/1/ns/pid
```

Si les valeurs sont identiques, notre shell et le PID 1 partagent le même namespace PID.

---

## 5.2.2. Comparer plusieurs processus

Nous pouvons comparer deux processus :

```bash
pid_a=$$
pid_b=1

readlink /proc/$pid_a/ns/pid
readlink /proc/$pid_b/ns/pid
```

Même valeur :

```text
pid:[4026531836]
pid:[4026531836]
```

Les processus partagent le même namespace PID.

Valeurs différentes :

```text
pid:[4026532501]
pid:[4026531836]
```

Les processus sont dans des namespaces PID différents.

---

# 5.3. Créer un namespace PID avec `unshare`

## 5.3.1. Première commande correcte

Pour créer un namespace PID de manière pédagogique, nous utilisons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell obtenu, nous exécutons :

```bash
echo $$
ps aux
cat /proc/1/comm
readlink /proc/$$/ns/pid
```

Nous devons observer que notre shell est vu comme `PID 1` dans ce nouveau namespace.

---

## 5.3.2. Pourquoi utiliser `--fork` ?

Le namespace PID a une particularité importante.

Quand un processus crée un nouveau namespace PID, ce n’est pas ce processus lui-même qui devient PID 1 dans le nouveau namespace. Ce sont ses enfants qui entrent dans ce nouveau namespace.

C’est pour cela que nous utilisons :

```bash
--fork
```

Avec :

```bash
unshare --fork --pid bash
```

`unshare` crée un enfant, et cet enfant devient le premier processus du nouveau namespace PID.

Nous retenons :

```text
Pour créer un namespace PID utilisable en shell, nous utilisons presque toujours --fork.
```

---

## 5.3.3. Pourquoi utiliser `--mount-proc` ?

Les commandes comme `ps` lisent les informations dans `/proc`.

Si nous créons un nouveau namespace PID sans remonter `/proc`, nous pouvons obtenir une vue incohérente.

Exemple à éviter pour débuter :

```bash
unshare --fork --pid bash
ps aux
```

Le shell est dans un nouveau namespace PID, mais `/proc` peut encore refléter l’ancien environnement de montage.

La forme correcte est :

```bash
unshare --fork --pid --mount-proc bash
```

`--mount-proc` monte un `/proc` adapté au nouveau namespace PID.

Nous retenons :

```text
Namespace PID isolé + /proc non remonté = diagnostic souvent trompeur.
```

---

# 5.4. Le PID 1 dans un namespace PID

## 5.4.1. Un processus particulier

Dans chaque namespace PID, il existe un processus vu comme `PID 1`.

Ce processus a un rôle particulier.

Sur un système classique, le PID 1 est souvent :

```text
systemd
```

ou un autre système d’initialisation.

Dans un conteneur, le PID 1 peut être :

```text
node
python
nginx
bash
tini
```

Nous pouvons observer cela avec :

```bash
cat /proc/1/comm
```

---

## 5.4.2. Pourquoi le PID 1 est important ?

Le PID 1 a des responsabilités particulières :

- il reçoit certains signaux différemment ;
    
- il doit récolter les processus enfants terminés ;
    
- il devient le parent adoptif de certains processus orphelins ;
    
- il influence l’arrêt propre d’un conteneur ;
    
- il peut accumuler des zombies s’il ne gère pas correctement ses enfants.
    

Dans un conteneur, si notre application est directement PID 1, elle doit gérer correctement ces aspects.

---

## 5.4.3. Exemple simple

Nous lançons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
echo $$
cat /proc/1/comm
ps -ef
```

Nous pouvons voir que `bash` est PID 1.

Exemple :

```text
UID        PID  PPID  CMD
root         1     0  bash
root        10     1  ps -ef
```

Ici, `bash` est le premier processus du namespace.

---

# 5.5. Processus zombies et rôle du PID 1

## 5.5.1. Qu’est-ce qu’un processus zombie ?

Un processus zombie est un processus terminé dont le parent n’a pas encore récupéré le code de retour.

Nous pouvons en voir avec :

```bash
ps -eo pid,ppid,state,comm | awk '$3=="Z"'
```

Un zombie ne consomme presque plus de mémoire, mais il occupe encore une entrée dans la table des processus.

---

## 5.5.2. Pourquoi cela concerne les conteneurs ?

Dans un conteneur, si le processus PID 1 lance des enfants et ne les récolte pas correctement, les zombies peuvent s’accumuler.

Cela arrive notamment quand une application n’est pas conçue pour jouer le rôle de processus d’init.

Exemple conceptuel :

```text
PID 1 : application
  ├── enfant A terminé mais non récolté -> zombie
  ├── enfant B terminé mais non récolté -> zombie
  └── enfant C terminé mais non récolté -> zombie
```

Dans un système classique, `systemd` ou un init joue ce rôle. Dans un conteneur minimal, ce rôle peut manquer.

---

## 5.5.3. Utiliser un init minimal

Pour éviter ce problème, nous utilisons parfois un init minimal comme :

```text
tini
dumb-init
```

Avec Docker, nous pouvons utiliser :

```bash
docker run --init ...
```

Docker ajoute alors un petit processus d’init comme PID 1 du conteneur.

Ce processus s’occupe notamment de transmettre les signaux et de récolter les processus enfants.

---

# 5.6. Signaux et PID 1

## 5.6.1. Les signaux Unix

Les signaux permettent de demander à un processus de réagir.

Exemples :

|Signal|Rôle courant|
|---|---|
|`SIGTERM`|demander un arrêt propre|
|`SIGKILL`|tuer immédiatement|
|`SIGINT`|interruption clavier|
|`SIGHUP`|rechargement ou fermeture terminal|
|`SIGCHLD`|notification de terminaison d’un enfant|

Nous envoyons un signal avec :

```bash
kill -TERM <PID>
```

ou :

```bash
kill <PID>
```

qui envoie par défaut `SIGTERM`.

---

## 5.6.2. PID 1 et gestion des signaux

Le PID 1 a un comportement particulier : certains signaux qui tueraient normalement un processus peuvent être ignorés si le processus PID 1 n’a pas explicitement prévu de gestionnaire.

Dans un conteneur, cela peut provoquer un problème :

```text
docker stop envoie SIGTERM
l’application PID 1 ne le gère pas correctement
le conteneur ne s’arrête pas proprement
Docker finit par envoyer SIGKILL
```

Nous devons donc comprendre que le rôle de PID 1 n’est pas neutre.

---

## 5.6.3. Conséquence pour les applications

Une application lancée comme PID 1 doit :

- gérer `SIGTERM` ;
    
- arrêter proprement ses workers ;
    
- fermer ses sockets ;
    
- vider ses buffers ;
    
- attendre ses enfants ;
    
- éviter les zombies.
    

Si elle ne le fait pas, nous devons utiliser un init minimal ou ajuster l’entrypoint.

---

# 5.7. PID namespace et hiérarchie

## 5.7.1. Namespaces PID imbriqués

Les namespaces PID peuvent être hiérarchiques.

Un namespace PID peut avoir un namespace enfant.

Un processus dans le namespace parent peut voir les processus du namespace enfant, mais l’inverse n’est pas vrai.

Exemple conceptuel :

```text
Namespace hôte
  PID 24531 -> processus visible comme PID 1 dans le conteneur

Namespace conteneur
  PID 1 -> même processus
```

Depuis l’hôte, nous voyons le processus avec son PID hôte.

Depuis le conteneur, nous le voyons avec son PID conteneur.

---

## 5.7.2. Observer les PID avec `NSpid`

Dans `/proc/<PID>/status`, nous pouvons parfois voir une ligne :

```bash
grep NSpid /proc/<PID>/status
```

Exemple :

```text
NSpid:  24531  1
```

Cela signifie que le processus a plusieurs identifiants selon les niveaux de namespaces PID.

Ici :

- `24531` est le PID dans le namespace parent ;
    
- `1` est le PID dans le namespace enfant.
    

Cette ligne est très utile pour comprendre les conteneurs.

---

# 5.8. Lien avec `/proc`

## 5.8.1. `/proc` dépend du namespace PID

Dans un namespace PID, `/proc` doit refléter la vue des processus de ce namespace.

Nous testons :

```bash
unshare --fork --pid --mount-proc bash
```

Puis :

```bash
ls /proc | grep -E '^[0-9]+$'
ps aux
```

Nous voyons seulement les processus visibles dans le namespace PID.

---

## 5.8.2. Sans `--mount-proc`

Si nous lançons :

```bash
unshare --fork --pid bash
```

puis :

```bash
ps aux
```

la sortie peut être incohérente, car `/proc` n’a pas forcément été remonté.

C’est un piège très courant.

Nous retenons :

```text
Le namespace PID contrôle la numérotation et la visibilité.
Mais les outils comme ps dépendent de /proc.
Il faut donc que /proc corresponde au namespace PID courant.
```

---

## 5.8.3. Comparer `/proc/1` dedans et dehors

Dans le namespace :

```bash
cat /proc/1/comm
readlink /proc/1/ns/pid
```

Sur l’hôte :

```bash
cat /proc/1/comm
readlink /proc/1/ns/pid
```

Nous pouvons voir deux réalités différentes.

Dans le namespace, `/proc/1` désigne le PID 1 du namespace.

Sur l’hôte, `/proc/1` désigne le PID 1 de l’hôte.

---

# 5.9. PID namespace et Docker

## 5.9.1. Observer un conteneur

Nous lançons un conteneur :

```bash
docker run --rm -d --name pid-demo alpine sleep 1000
```

Nous récupérons son PID côté hôte :

```bash
docker inspect --format '{{.State.Pid}}' pid-demo
```

Exemple :

```text
24531
```

Sur l’hôte :

```bash
ps -p 24531 -o pid,ppid,comm,args
```

Nous voyons le processus avec son PID hôte.

Dans le conteneur :

```bash
docker exec pid-demo ps
```

Nous voyons souvent :

```text
PID   USER     TIME  COMMAND
1     root     0:00  sleep 1000
```

Le même processus est donc vu comme PID `24531` depuis l’hôte et PID `1` dans le conteneur.

---

## 5.9.2. Observer les namespaces

Sur l’hôte :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' pid-demo)

sudo readlink /proc/$pid/ns/pid
readlink /proc/1/ns/pid
```

Si les valeurs diffèrent, le conteneur possède son propre namespace PID.

Nous pouvons aussi regarder :

```bash
sudo grep NSpid /proc/$pid/status
```

Exemple :

```text
NSpid:  24531  1
```

---

## 5.9.3. Partager le namespace PID de l’hôte

Docker permet de lancer un conteneur avec le namespace PID de l’hôte :

```bash
docker run --rm -it --pid=host alpine sh
```

Dans ce conteneur :

```sh
ps aux
```

Nous pouvons voir les processus de l’hôte.

Cette option réduit fortement l’isolation.

Nous l’utilisons seulement pour des besoins particuliers de diagnostic ou d’administration.

---

# 5.10. PID namespace et Kubernetes

## 5.10.1. PID namespace dans un pod

Dans Kubernetes, les conteneurs d’un pod ont généralement une isolation de processus par rapport à l’hôte.

Selon la configuration, les conteneurs d’un même pod peuvent ou non partager le namespace PID.

Par défaut, chaque conteneur a souvent sa propre vue de processus, mais Kubernetes permet des options comme le partage du namespace de processus du pod.

---

## 5.10.2. `shareProcessNamespace`

Kubernetes propose une option :

```yaml
shareProcessNamespace: true
```

Cela permet aux conteneurs d’un même pod de voir les processus les uns des autres.

C’est utile pour certains sidecars de diagnostic ou de supervision.

Exemple conceptuel :

```text
Pod
  container app
  container debug

Avec shareProcessNamespace: true
  debug peut voir les processus de app
```

---

## 5.10.3. `hostPID`

Kubernetes permet aussi :

```yaml
hostPID: true
```

Dans ce cas, le pod partage le namespace PID de l’hôte.

C’est une option très sensible.

Elle peut être utile pour certains agents système, mais elle réduit fortement l’isolation.

Nous retenons :

```text
hostPID: true donne une visibilité sur les processus de l’hôte.
Nous ne l’utilisons pas sans raison forte.
```

---

# 5.11. Entrer dans un namespace PID avec `nsenter`

## 5.11.1. Depuis l’hôte vers un processus isolé

Si nous avons le PID hôte d’un processus dans un namespace PID, nous pouvons entrer dans ce namespace :

```bash
sudo nsenter --target <PID> --pid --fork bash
```

Mais en pratique, pour obtenir une vue cohérente, nous entrons souvent aussi dans le namespace mount :

```bash
sudo nsenter --target <PID> --mount --pid --fork bash
```

Puis :

```bash
ps aux
cat /proc/1/comm
```

---

## 5.11.2. Entrer dans plusieurs namespaces d’un conteneur

Pour diagnostiquer un conteneur depuis l’hôte :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid --fork bash
```

Nous obtenons une vue très proche de celle du conteneur.

Nous pouvons ensuite exécuter :

```bash
ps aux
hostname
ip addr
mount
```

C’est souvent plus puissant qu’un simple `docker exec`, notamment si l’image ne contient pas de shell ou d’outils de diagnostic.

---

# 5.12. Expérience complète guidée

## 5.12.1. Créer un namespace PID

Nous lançons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans ce shell :

```bash
echo "PID du shell : $$"
cat /proc/1/comm
ps -ef
readlink /proc/$$/ns/pid
```

Nous observons que notre shell est PID 1.

---

## 5.12.2. Lancer un processus enfant

Dans le namespace :

```bash
sleep 1000 &
ps -ef
```

Nous pouvons voir :

```text
UID   PID  PPID  CMD
root    1     0  bash
root   12     1  sleep 1000
root   13     1  ps -ef
```

Le processus `sleep` a un PID dans le namespace.

---

## 5.12.3. Observer depuis l’hôte

Dans un autre terminal sur l’hôte :

```bash
ps aux | grep sleep
```

Nous trouvons le PID hôte de `sleep`.

Puis :

```bash
grep NSpid /proc/<PID_HOTE>/status
```

Nous pouvons voir plusieurs PID :

```text
NSpid:  30142  12
```

Cela signifie que le processus est vu comme `30142` depuis l’hôte et `12` dans le namespace enfant.

---

## 5.12.4. Nettoyage

Dans le shell isolé :

```bash
kill %1
exit
```

Quand le PID 1 du namespace se termine, les autres processus du namespace sont généralement terminés également.

---

# 5.13. Pièges classiques

## 5.13.1. Oublier `--fork`

Commande problématique :

```bash
unshare --pid bash
```

Nous risquons de ne pas obtenir le comportement attendu.

Commande recommandée :

```bash
unshare --fork --pid --mount-proc bash
```

---

## 5.13.2. Oublier `--mount-proc`

Commande incomplète :

```bash
unshare --fork --pid bash
```

Nous créons bien un namespace PID, mais les outils qui lisent `/proc` peuvent afficher une vue incorrecte.

Commande recommandée :

```bash
unshare --fork --pid --mount-proc bash
```

---

## 5.13.3. Croire que PID 1 est un processus ordinaire

Dans un namespace PID, le PID 1 doit gérer correctement :

- signaux ;
    
- enfants ;
    
- zombies ;
    
- arrêt propre.
    

Une application lancée directement comme PID 1 peut se comporter différemment que lorsqu’elle est lancée sous un shell ou un superviseur.

---

## 5.13.4. Confondre PID hôte et PID interne

Quand nous lisons un log ou un message d’erreur, nous devons savoir depuis quel namespace le PID est exprimé.

Un PID affiché dans le conteneur peut ne pas exister avec le même numéro sur l’hôte.

Nous utilisons :

```bash
grep NSpid /proc/<PID>/status
```

pour comprendre les correspondances.

---

# 5.14. Exercices

## Exercice 1 — Observer le namespace PID courant

Nous exécutons :

```bash
readlink /proc/$$/ns/pid
readlink /proc/1/ns/pid
ps -ef | head
```

Nous répondons :

1. Le shell courant partage-t-il le namespace PID avec le PID 1 ?
    
2. Que voyons-nous avec `ps` ?
    
3. Sommes-nous probablement sur l’hôte ou dans un environnement isolé ?
    

---

## Exercice 2 — Créer un namespace PID

Nous exécutons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
echo $$
cat /proc/1/comm
ps -ef
readlink /proc/$$/ns/pid
```

Nous répondons :

1. Quel est le PID du shell ?
    
2. Quel est le processus PID 1 ?
    
3. Quels processus sont visibles ?
    
4. Pourquoi avons-nous utilisé `--fork` ?
    
5. Pourquoi avons-nous utilisé `--mount-proc` ?
    

Nous quittons :

```bash
exit
```

---

## Exercice 3 — Observer les PID depuis l’hôte

Dans un premier terminal :

```bash
unshare --fork --pid --mount-proc bash
sleep 1000
```

Dans un second terminal sur l’hôte :

```bash
ps aux | grep sleep
grep NSpid /proc/<PID_HOTE>/status
```

Nous répondons :

1. Quel est le PID du processus sur l’hôte ?
    
2. Quel est son PID dans le namespace ?
    
3. Que signifie la ligne `NSpid` ?
    
4. Pourquoi un même processus peut-il avoir plusieurs PID ?
    

---

## Exercice 4 — Tester `/proc` sans `--mount-proc`

Nous exécutons :

```bash
unshare --fork --pid bash
```

Puis :

```bash
ps -ef | head
ls /proc | grep -E '^[0-9]+$' | head
```

Nous comparons avec :

```bash
unshare --fork --pid --mount-proc bash
```

Nous répondons :

1. Quelle différence observons-nous ?
    
2. Pourquoi `/proc` est-il important ?
    
3. Pourquoi `ps` peut-il être trompeur ?
    

---

## Exercice 5 — Observer un conteneur Docker

Si Docker est disponible :

```bash
docker run --rm -d --name pid-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' pid-demo)

ps -p "$pid" -o pid,ppid,comm,args
sudo grep NSpid /proc/$pid/status

docker exec pid-demo ps
docker rm -f pid-demo
```

Nous répondons :

1. Quel est le PID du processus sur l’hôte ?
    
2. Quel est son PID dans le conteneur ?
    
3. Quel fichier permet de voir les deux ?
    
4. Quel namespace explique cette différence ?
    

---

## Exercice 6 — Comprendre le rôle du PID 1

Nous lançons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
echo $$
sleep 1000 &
ps -ef
kill -TERM 1
```

Nous observons le comportement.

Nous répondons :

1. Le PID 1 réagit-il comme un processus ordinaire ?
    
2. Que se passe-t-il si le PID 1 se termine ?
    
3. Pourquoi ce comportement est-il important pour les conteneurs ?
    

---

# 5.15. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace PID isole la numérotation et la visibilité des processus.
    
2. Un même processus peut avoir plusieurs PID selon le namespace d’observation.
    
3. `/proc/<PID>/ns/pid` permet d’identifier le namespace PID d’un processus.
    
4. `unshare --fork --pid --mount-proc bash` est la commande pédagogique de base.
    
5. `--fork` est nécessaire car les enfants entrent dans le nouveau namespace PID.
    
6. `--mount-proc` est nécessaire pour que `/proc` reflète correctement le namespace PID.
    
7. Le PID 1 d’un namespace a un rôle particulier.
    
8. Le PID 1 doit gérer les signaux et récolter les processus enfants.
    
9. Une application conteneurisée lancée directement comme PID 1 peut avoir des comportements inattendus.
    
10. La ligne `NSpid` dans `/proc/<PID>/status` permet de voir les PID selon les namespaces.
    
11. Docker, Podman et Kubernetes utilisent le namespace PID pour isoler les processus.
    
12. Les options comme `--pid=host` ou `hostPID: true` réduisent fortement l’isolation.
    
13. Le namespace PID isole la vue, mais ne limite pas à lui seul le nombre de processus. Pour cela, nous utilisons les cgroups.
    

---

# Conclusion du chapitre 5

Nous avons étudié le namespace PID, un mécanisme central de l’isolation Linux.

Nous savons maintenant qu’un processus peut être vu avec des PID différents selon le namespace depuis lequel nous l’observons. Nous avons compris pourquoi un conteneur peut avoir son propre PID 1, pourquoi `/proc` doit être remonté dans un namespace PID, et pourquoi le processus PID 1 a des responsabilités particulières.

Nous avons aussi vu que cette isolation peut être réduite ou supprimée avec des options comme `--pid=host` ou `hostPID: true`, ce qui doit être réservé à des cas particuliers.

Dans le chapitre suivant, nous étudions le namespace mount, qui permet à un processus de voir une arborescence de montages différente de celle de l’hôte.

# Chapitre 6 — Namespace mount

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace mount, aussi appelé namespace de montage.

Le namespace mount permet à un processus de voir une arborescence de points de montage différente de celle de l’hôte. C’est un mécanisme fondamental des conteneurs : il permet de donner à une application l’impression qu’elle possède son propre système de fichiers, alors qu’elle partage toujours le noyau avec l’hôte.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace mount ;
    
- créer un namespace mount avec `unshare` ;
    
- comprendre la différence entre arborescence de fichiers et points de montage ;
    
- manipuler des montages isolés ;
    
- comprendre la propagation des montages ;
    
- distinguer `shared`, `private`, `slave` et `unbindable` ;
    
- expliquer le rôle des bind mounts ;
    
- comprendre la différence entre `chroot`, namespace mount et `pivot_root` ;
    
- comprendre comment un conteneur obtient son root filesystem ;
    
- identifier les pièges de sécurité liés aux montages.
    

---

# 6.1. Pourquoi isoler les montages ?

## 6.1.1. Le problème de départ

Sur un système Linux classique, les processus voient une arborescence de fichiers commune.

Nous pouvons afficher les montages avec :

```bash
findmnt
```

ou :

```bash
mount
```

Nous voyons par exemple :

```text
/
├─ /proc
├─ /sys
├─ /dev
├─ /run
├─ /home
└─ /mnt/data
```

Si tous les processus partagent exactement la même vue des montages, il devient difficile de créer un environnement isolé.

Un conteneur a besoin d’une vue différente :

```text
/
├─ /bin
├─ /etc
├─ /lib
├─ /proc
├─ /tmp
└─ /app
```

Cette racine n’est pas forcément la vraie racine de l’hôte. C’est une racine apparente construite pour le processus isolé.

Le namespace mount permet précisément cela.

---

## 6.1.2. Ce que le namespace mount isole

Le namespace mount isole la table des montages visible par un processus.

Cela signifie qu’un processus peut avoir :

- ses propres montages ;
    
- ses propres bind mounts ;
    
- son propre `/proc` ;
    
- son propre `/sys` ;
    
- son propre `/tmp` ;
    
- une racine apparente différente ;
    
- des volumes montés à des emplacements spécifiques ;
    
- des montages en lecture seule ou lecture-écriture selon les besoins.
    

Nous retenons :

```text
Le namespace mount isole la vue des points de montage.
```

Il ne copie pas automatiquement tous les fichiers. Il isole la manière dont les systèmes de fichiers sont montés et visibles.

---

# 6.2. Arborescence de fichiers et points de montage

## 6.2.1. L’arborescence de fichiers

Sous Linux, tout est présenté dans une arborescence unique qui commence à :

```text
/
```

Nous pouvons naviguer avec :

```bash
cd /
ls
```

Nous voyons par exemple :

```text
bin
boot
dev
etc
home
proc
sys
tmp
usr
var
```

Mais cette arborescence est composée de plusieurs systèmes de fichiers montés à différents endroits.

---

## 6.2.2. Le point de montage

Un point de montage est un répertoire sur lequel nous attachons un système de fichiers.

Exemple :

```bash
mount -t tmpfs tmpfs /tmp/test-mount
```

Ici :

```text
/tmp/test-mount
```

devient un point de montage.

Le contenu visible à cet endroit ne vient plus du répertoire ordinaire initial, mais du système de fichiers monté.

Nous pouvons vérifier :

```bash
findmnt /tmp/test-mount
```

---

## 6.2.3. Le rôle du namespace mount

Sans namespace mount, un nouveau montage peut être visible par les autres processus de l’hôte.

Avec un namespace mount isolé, nous pouvons monter quelque chose dans notre environnement sans le rendre visible ailleurs, à condition de gérer correctement la propagation des montages.

C’est ce qui permet à un conteneur d’avoir ses propres montages sans polluer l’hôte.

---

# 6.3. Observer le namespace mount courant

## 6.3.1. Avec `/proc/<PID>/ns/mnt`

Comme les autres namespaces, le namespace mount d’un processus est visible dans :

```bash
/proc/<PID>/ns/mnt
```

Pour notre shell courant :

```bash
readlink /proc/$$/ns/mnt
```

Exemple :

```text
mnt:[4026531841]
```

Pour le PID 1 :

```bash
readlink /proc/1/ns/mnt
```

Si les deux valeurs sont identiques, notre shell partage le même namespace mount que le PID 1.

---

## 6.3.2. Comparer deux processus

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/mnt)"
echo "PID 1         : $(readlink /proc/1/ns/mnt)"
```

Même valeur :

```text
Shell courant : mnt:[4026531841]
PID 1         : mnt:[4026531841]
```

Les deux processus partagent le même namespace mount.

Valeurs différentes :

```text
Shell courant : mnt:[4026533000]
PID 1         : mnt:[4026531841]
```

Les deux processus ont des vues de montages différentes.

---

## 6.3.3. Observer les montages visibles

Nous pouvons observer les montages du processus courant avec :

```bash
findmnt
```

ou :

```bash
cat /proc/self/mounts
```

ou encore :

```bash
cat /proc/self/mountinfo
```

Différence importante :

```text
/proc/self/mounts     → vue simplifiée des montages
/proc/self/mountinfo  → vue plus complète, avec propagation et identifiants
```

Pour l’étude du namespace mount, `mountinfo` devient très utile.

---

# 6.4. Créer un namespace mount avec `unshare`

## 6.4.1. Première expérience

Nous créons un namespace mount :

```bash
unshare --mount bash
```

Dans le shell isolé, nous vérifions :

```bash
readlink /proc/$$/ns/mnt
findmnt | head
```

Dans un autre terminal, nous pouvons comparer :

```bash
readlink /proc/<PID_DU_SHELL_ISOLE>/ns/mnt
readlink /proc/1/ns/mnt
```

Nous devons voir que le shell isolé possède un namespace mount différent.

---

## 6.4.2. Monter un tmpfs dans le namespace

Dans le shell isolé :

```bash
mkdir -p /tmp/ns-mount-demo
mount -t tmpfs tmpfs /tmp/ns-mount-demo
findmnt /tmp/ns-mount-demo
```

Nous créons un fichier :

```bash
echo "visible dans le namespace" > /tmp/ns-mount-demo/test.txt
cat /tmp/ns-mount-demo/test.txt
```

Nous avons monté un système de fichiers temporaire dans le namespace courant.

---

## 6.4.3. Vérifier depuis l’hôte

Depuis un autre terminal sur l’hôte :

```bash
findmnt /tmp/ns-mount-demo
ls /tmp/ns-mount-demo
```

Selon la propagation des montages de votre système, le montage peut être visible ou non depuis l’hôte.

C’est un point essentiel : créer un namespace mount ne suffit pas toujours à éviter la propagation. Il faut comprendre la propagation des montages.

---

# 6.5. Le piège de la propagation des montages

## 6.5.1. Pourquoi ce piège existe ?

Sur beaucoup de distributions modernes, certains montages sont configurés en mode `shared`.

Cela signifie qu’un montage créé dans un namespace peut être propagé vers un autre namespace qui partage le même groupe de propagation.

Nous pourrions croire :

```text
J’ai créé un namespace mount, donc mes montages sont automatiquement invisibles ailleurs.
```

Ce n’est pas toujours vrai.

La propagation des montages peut transmettre certains événements de montage entre namespaces.

---

## 6.5.2. Vérifier la propagation

Nous pouvons inspecter :

```bash
findmnt -o TARGET,PROPAGATION /
```

Sortie possible :

```text
TARGET PROPAGATION
/      shared
```

ou :

```text
TARGET PROPAGATION
/      private
```

Nous pouvons aussi regarder :

```bash
cat /proc/self/mountinfo | head
```

Les mentions comme :

```text
shared:1
master:2
```

indiquent des relations de propagation.

---

## 6.5.3. Rendre les montages privés

Dans un namespace de test, nous pouvons rendre l’arborescence privée :

```bash
mount --make-rprivate /
```

Ensuite, les nouveaux montages faits dans ce namespace ne se propagent normalement plus vers l’extérieur.

Commande pédagogique recommandée :

```bash
unshare --mount bash
mount --make-rprivate /
```

Puis :

```bash
mkdir -p /tmp/ns-mount-demo
mount -t tmpfs tmpfs /tmp/ns-mount-demo
findmnt /tmp/ns-mount-demo
```

Depuis l’hôte, le montage ne doit normalement pas apparaître.

Nous retenons :

```text
Pour un TP propre sur les namespaces mount, nous rendons souvent / récursivement privé avec mount --make-rprivate /.
```

---

# 6.6. Les types de propagation

## 6.6.1. Vue d’ensemble

Linux définit plusieurs modes de propagation des montages :

```text
shared
private
slave
unbindable
```

Ces modes déterminent comment les événements de montage sont propagés entre namespaces.

C’est une notion avancée, mais indispensable pour comprendre Docker, Kubernetes et systemd.

---

## 6.6.2. `shared`

Un montage `shared` propage les événements de montage à d’autres montages du même groupe de propagation.

Si nous montons un système de fichiers sous un point `shared`, ce montage peut apparaître dans d’autres namespaces liés.

Nous pouvons voir ce mode avec :

```bash
findmnt -o TARGET,PROPAGATION /
```

Sortie :

```text
/ shared
```

Ce mode est pratique pour certains usages système, mais peut surprendre dans un contexte d’isolation.

---

## 6.6.3. `private`

Un montage `private` ne propage pas les événements et ne reçoit pas les événements des autres.

Nous pouvons rendre un montage privé :

```bash
mount --make-private /chemin
```

Ou récursivement :

```bash
mount --make-rprivate /chemin
```

Pour un conteneur, nous voulons souvent une base privée afin d’éviter que les montages internes fuient vers l’hôte.

---

## 6.6.4. `slave`

Un montage `slave` reçoit les événements de son maître, mais ne renvoie pas ses propres événements vers lui.

Nous pouvons le voir comme une propagation à sens unique.

Cela peut être utile lorsqu’un environnement isolé doit voir certains montages de l’hôte, mais ne doit pas propager ses propres montages vers l’hôte.

Commande :

```bash
mount --make-slave /chemin
```

Récursif :

```bash
mount --make-rslave /chemin
```

---

## 6.6.5. `unbindable`

Un montage `unbindable` ne peut pas être bind-mounté.

C’est utile pour éviter certaines copies ou propagations de montages sensibles.

Commande :

```bash
mount --make-unbindable /chemin
```

Récursif :

```bash
mount --make-runbindable /chemin
```

Ce mode est plus spécialisé, mais nous devons savoir qu’il existe.

---

## 6.6.6. Synthèse

|Mode|Reçoit des événements|Propage ses événements|Usage typique|
|---|--:|--:|---|
|`shared`|oui|oui|propagation entre environnements liés|
|`private`|non|non|isolation forte des montages|
|`slave`|oui|non|recevoir de l’hôte sans propager vers lui|
|`unbindable`|non|non|empêcher certains bind mounts|

Nous retenons surtout :

```text
shared peut propager.
private isole.
slave reçoit sans renvoyer.
unbindable empêche certains bind mounts.
```

---

# 6.7. Bind mounts

## 6.7.1. Définition

Un bind mount permet de remonter un chemin existant à un autre endroit de l’arborescence.

Exemple :

```bash
mkdir -p /tmp/source /tmp/destination
echo "test" > /tmp/source/fichier.txt

mount --bind /tmp/source /tmp/destination
ls /tmp/destination
```

Nous voyons :

```text
fichier.txt
```

Le même contenu est visible à deux emplacements.

---

## 6.7.2. Utilité dans les conteneurs

Les bind mounts sont essentiels pour les conteneurs.

Quand nous faisons avec Docker :

```bash
docker run -v /home/user/app:/app image
```

Docker crée un bind mount entre :

```text
/home/user/app sur l’hôte
/app dans le conteneur
```

Cela permet de rendre un dossier de l’hôte visible dans le conteneur.

---

## 6.7.3. Bind mount en lecture seule

Nous pouvons remonter un bind mount en lecture seule.

Exemple :

```bash
mount --bind /tmp/source /tmp/destination
mount -o remount,ro,bind /tmp/destination
```

Nous vérifions :

```bash
findmnt /tmp/destination
```

Dans les conteneurs, cela correspond à des montages de type :

```bash
docker run -v /host/path:/container/path:ro image
```

Le mode lecture seule réduit les risques de modification de l’hôte.

---

## 6.7.4. Risques des bind mounts

Un bind mount peut exposer des chemins sensibles.

Exemples dangereux :

```bash
docker run -v /:/host image
docker run -v /var/run/docker.sock:/var/run/docker.sock image
docker run -v /proc:/host/proc image
docker run -v /sys:/host/sys image
```

Ces montages peuvent donner au conteneur une visibilité ou un contrôle excessif sur l’hôte.

Nous retenons :

```text
Un bind mount est une passerelle directe entre deux vues de l’arborescence.
Il doit être choisi avec prudence.
```

---

# 6.8. Monter `/proc` dans un namespace mount

## 6.8.1. Pourquoi monter `/proc` ?

Dans un namespace PID, les outils comme `ps` dépendent de `/proc`.

Si nous construisons un environnement isolé, nous devons souvent monter un `/proc` adapté.

Exemple :

```bash
unshare --fork --pid --mount bash
mount -t proc proc /proc
ps aux
```

Mais attention : monter directement sur `/proc` dans un environnement non préparé peut être risqué ou perturber la vue du shell.

C’est pourquoi `unshare` propose :

```bash
unshare --fork --pid --mount-proc bash
```

---

## 6.8.2. `/proc` dans un conteneur

Dans un conteneur, `/proc` est généralement monté pour refléter le namespace PID du conteneur.

Ainsi, dans le conteneur :

```bash
ps aux
ls /proc
cat /proc/1/comm
```

nous voyons la vue des processus du conteneur, pas forcément celle de l’hôte.

C’est une combinaison entre :

```text
namespace PID
namespace mount
montage de procfs
```

---

## 6.8.3. Risques liés à `/proc`

Monter le `/proc` de l’hôte dans un conteneur est dangereux.

Exemple à éviter :

```bash
docker run -v /proc:/host/proc image
```

Selon les droits du conteneur, cela peut exposer :

- processus de l’hôte ;
    
- informations sensibles ;
    
- variables d’environnement ;
    
- descripteurs ;
    
- paramètres noyau ;
    
- informations réseau.
    

Nous devons toujours distinguer :

```text
/proc du conteneur
/proc de l’hôte
```

---

# 6.9. Monter `/sys` et `/dev`

## 6.9.1. `/sys`

`/sys` expose `sysfs`, qui donne accès à des informations sur :

- périphériques ;
    
- pilotes ;
    
- bus ;
    
- modules noyau ;
    
- classes de périphériques ;
    
- paramètres matériels.
    

Dans un conteneur, exposer `/sys` trop largement peut être dangereux.

Montage à éviter sans besoin fort :

```bash
docker run -v /sys:/sys image
```

Un conteneur standard devrait avoir une vue limitée et contrôlée.

---

## 6.9.2. `/dev`

`/dev` contient les fichiers de périphériques.

Exemples :

```text
/dev/null
/dev/zero
/dev/random
/dev/sda
/dev/tty
/dev/kvm
/dev/dri
```

Donner accès à certains périphériques peut donner beaucoup de pouvoir.

Exemples sensibles :

```text
/dev/sda     → disque brut
/dev/kvm     → virtualisation
/dev/mem     → mémoire physique, si présent
/dev/docker  → selon les cas
```

Les conteneurs doivent avoir un `/dev` minimal et contrôlé.

---

## 6.9.3. Conteneurs privilégiés

Un conteneur lancé avec :

```bash
docker run --privileged ...
```

peut recevoir beaucoup plus de droits, de capabilities et d’accès aux périphériques.

Cela réduit fortement l’isolation.

Nous retenons :

```text
Le namespace mount aide à isoler l’arborescence, mais les montages exposés déterminent une grande partie de la sécurité réelle.
```

---

# 6.10. `chroot` vs namespace mount

## 6.10.1. Qu’est-ce que `chroot` ?

`chroot` change la racine apparente d’un processus.

Exemple :

```bash
chroot /chemin/nouvelle-racine /bin/sh
```

Le processus voit alors :

```text
/chemin/nouvelle-racine
```

comme `/`.

Historiquement, `chroot` a été utilisé pour isoler partiellement des environnements.

---

## 6.10.2. Limites de `chroot`

`chroot` seul n’est pas un mécanisme d’isolation complet.

Il n’isole pas :

- les PID ;
    
- le réseau ;
    
- les IPC ;
    
- le hostname ;
    
- les cgroups ;
    
- les capabilities ;
    
- les montages au sens complet ;
    
- le noyau.
    

De plus, avec des privilèges suffisants, il existe des manières de s’échapper d’un `chroot` mal configuré.

Nous retenons :

```text
chroot change une racine apparente.
Il ne constitue pas à lui seul un conteneur sécurisé.
```

---

## 6.10.3. Namespace mount et `chroot`

Un namespace mount est plus puissant car il permet de construire une table de montages propre.

Nous pouvons combiner :

```text
namespace mount
bind mounts
montage de /proc
montage de /dev minimal
chroot ou pivot_root
```

C’est cette combinaison qui permet de construire un environnement proche d’un conteneur.

---

# 6.11. `pivot_root`

## 6.11.1. Rôle de `pivot_root`

`pivot_root` permet de changer la racine d’un processus en remplaçant l’ancienne racine par une nouvelle, tout en plaçant l’ancienne racine dans un sous-répertoire de la nouvelle.

C’est un mécanisme utilisé dans la construction de systèmes isolés et de conteneurs.

Schéma conceptuel :

```text
Avant :
/                  → racine de l’hôte
/newroot           → futur rootfs

Après pivot_root :
/                  → ancien /newroot
/oldroot           → ancienne racine de l’hôte
```

Ensuite, on peut démonter `/oldroot` pour couper l’accès à l’ancienne racine.

---

## 6.11.2. Pourquoi ne pas utiliser seulement `chroot` ?

`pivot_root` est plus adapté à la construction d’un environnement complet car il agit au niveau des montages.

Il permet de remplacer réellement la racine dans le namespace mount.

Les runtimes de conteneurs utilisent plutôt des mécanismes de ce type, avec une préparation soigneuse des montages.

---

## 6.11.3. Attention

`pivot_root` est plus complexe à manipuler que `chroot`.

Il impose des contraintes :

- nouvelle racine sur un point de montage ;
    
- ancien root placé sous la nouvelle racine ;
    
- gestion correcte des répertoires courants ;
    
- démontage propre de l’ancien root ;
    
- permissions nécessaires.
    

Nous l’étudions surtout pour comprendre les conteneurs, pas comme commande de base à utiliser quotidiennement.

---

# 6.12. Root filesystem d’un conteneur

## 6.12.1. Qu’est-ce qu’un root filesystem ?

Le root filesystem, ou rootfs, est l’arborescence de fichiers vue comme `/` par le conteneur.

Il contient par exemple :

```text
/bin
/etc
/lib
/usr
/var
/tmp
/app
```

Dans Docker, il provient de l’image du conteneur.

Exemple :

```bash
docker run --rm alpine ls /
```

Sortie typique :

```text
bin
dev
etc
home
lib
media
mnt
proc
root
run
sbin
sys
tmp
usr
var
```

Ce n’est pas la racine de l’hôte. C’est la racine de l’image Alpine, montée dans un namespace.

---

## 6.12.2. Images en couches

Docker et les runtimes modernes utilisent souvent des systèmes de fichiers en couches.

Exemples :

```text
overlayfs
aufs anciennement
btrfs
zfs
devicemapper anciennement
```

Le principe courant avec OverlayFS :

```text
couches image en lecture seule
+
couche writable du conteneur
=
rootfs final visible dans le conteneur
```

Le namespace mount permet ensuite de présenter ce rootfs comme `/` au processus du conteneur.

---

## 6.12.3. Volumes et bind mounts

Les volumes Docker ou Kubernetes sont ajoutés comme montages supplémentaires dans le namespace mount du conteneur.

Exemples :

```bash
docker run -v /data:/data image
```

ou en Kubernetes :

```yaml
volumeMounts:
  - name: data
    mountPath: /data
```

Le conteneur voit `/data`, mais la source peut venir :

- d’un dossier de l’hôte ;
    
- d’un volume Docker ;
    
- d’un volume Kubernetes ;
    
- d’un PVC ;
    
- d’un secret ;
    
- d’une ConfigMap ;
    
- d’un tmpfs ;
    
- d’un stockage réseau.
    

---

# 6.13. Construire une mini-isolation de fichiers

## 6.13.1. Préparer un rootfs minimal

Pour un TP avancé, nous pouvons utiliser un rootfs minimal, par exemple avec BusyBox.

Sur Debian/Ubuntu, une approche simple est d’utiliser un répertoire de test :

```bash
mkdir -p /tmp/rootfs-demo/{bin,proc,sys,dev,tmp}
```

Nous pouvons copier un binaire statique comme BusyBox si disponible :

```bash
cp /bin/busybox /tmp/rootfs-demo/bin/
```

Puis créer quelques liens :

```bash
ln -s busybox /tmp/rootfs-demo/bin/sh
ln -s busybox /tmp/rootfs-demo/bin/ls
ln -s busybox /tmp/rootfs-demo/bin/mount
ln -s busybox /tmp/rootfs-demo/bin/ps
```

Cette partie dépend fortement de la distribution et des binaires disponibles.

---

## 6.13.2. Entrer avec un namespace mount et `chroot`

Exemple conceptuel :

```bash
sudo unshare --mount --uts --fork bash
mount --make-rprivate /
chroot /tmp/rootfs-demo /bin/sh
```

Dans ce shell, nous voyons une racine différente.

Mais ce n’est pas encore un conteneur complet :

- pas forcément de namespace PID ;
    
- pas forcément de réseau isolé ;
    
- `/proc` pas forcément monté ;
    
- `/dev` minimal absent ;
    
- pas de cgroups ;
    
- pas de politique de sécurité.
    

---

## 6.13.3. Ajouter `/proc`

Dans un environnement isolé bien préparé, nous pouvons monter :

```bash
mount -t proc proc /proc
```

Puis :

```bash
ps
```

Mais nous devons être prudents : monter `/proc` doit être fait dans le bon namespace, avec le bon PID namespace si nous voulons une vue cohérente.

---

# 6.14. Namespace mount et Docker

## 6.14.1. Ce que Docker fait

Quand nous lançons :

```bash
docker run --rm -it alpine sh
```

Docker prépare un namespace mount dans lequel :

- le rootfs de l’image Alpine est visible comme `/` ;
    
- `/proc` est monté ;
    
- `/sys` peut être monté avec restrictions ;
    
- `/dev` est préparé ;
    
- certains fichiers comme `/etc/resolv.conf`, `/etc/hosts`, `/etc/hostname` sont montés ou générés ;
    
- les volumes demandés sont montés ;
    
- certains chemins peuvent être en lecture seule selon la configuration.
    

Tout cela repose largement sur le namespace mount.

---

## 6.14.2. Observer les montages d’un conteneur

Dans un conteneur :

```sh
mount
cat /proc/self/mountinfo
findmnt
```

Depuis l’hôte, nous pouvons trouver le PID du conteneur :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' <container>)
```

Puis observer :

```bash
sudo readlink /proc/$pid/ns/mnt
sudo cat /proc/$pid/mountinfo | head
```

Nous pouvons aussi entrer dans son namespace mount :

```bash
sudo nsenter --target "$pid" --mount bash
```

Puis :

```bash
findmnt
```

---

## 6.14.3. Bind mounts Docker

Exemple :

```bash
docker run --rm -it -v "$PWD":/app alpine sh
```

Dans le conteneur :

```sh
ls /app
```

Nous voyons le contenu du répertoire courant de l’hôte.

C’est un bind mount.

Si nous voulons le rendre en lecture seule :

```bash
docker run --rm -it -v "$PWD":/app:ro alpine sh
```

Dans le conteneur, une tentative d’écriture dans `/app` doit échouer.

---

# 6.15. Namespace mount et Kubernetes

## 6.15.1. Volumes Kubernetes

Kubernetes utilise aussi les montages.

Dans un pod, nous pouvons définir :

```yaml
volumes:
  - name: config
    configMap:
      name: app-config

containers:
  - name: app
    image: example/app
    volumeMounts:
      - name: config
        mountPath: /etc/app/config
```

Le conteneur voit un montage à :

```text
/etc/app/config
```

Ce montage est préparé par Kubernetes et le runtime.

---

## 6.15.2. Secrets et ConfigMaps

Les Secrets et ConfigMaps Kubernetes sont souvent exposés sous forme de fichiers montés dans le conteneur.

Exemple :

```text
/etc/secrets/password
/etc/config/application.yaml
```

C’est pratique, mais cela signifie que les secrets existent dans l’arborescence de fichiers du conteneur.

Nous devons gérer :

- permissions ;
    
- accès applicatif ;
    
- rotation ;
    
- logs ;
    
- sauvegardes ;
    
- exposition accidentelle.
    

---

## 6.15.3. `hostPath`

Kubernetes permet de monter un chemin de l’hôte dans un pod avec `hostPath`.

Exemple :

```yaml
volumes:
  - name: host-logs
    hostPath:
      path: /var/log
```

C’est puissant, mais risqué.

Un `hostPath` mal choisi peut exposer l’hôte au conteneur.

Nous retenons :

```text
hostPath est l’équivalent Kubernetes d’un bind mount sensible depuis l’hôte.
```

---

# 6.16. Sécurité du namespace mount

## 6.16.1. Le namespace mount ne suffit pas

Un namespace mount peut isoler la vue des montages, mais la sécurité dépend de ce qui est monté.

Si nous montons toute la racine de l’hôte dans le conteneur :

```bash
docker run -v /:/host image
```

l’isolation du rootfs est largement compromise.

Même si le processus a une racine différente, il peut accéder à `/host`.

---

## 6.16.2. Montages sensibles

Chemins sensibles :

```text
/
/proc
/sys
/dev
/run
/var/run/docker.sock
/etc
/root
/home
/var/lib/docker
/var/lib/kubelet
```

Nous ne les montons pas dans un conteneur sans justification forte.

---

## 6.16.3. Lecture seule

Quand un montage est nécessaire, nous privilégions souvent la lecture seule :

```bash
docker run -v /host/path:/container/path:ro image
```

En Kubernetes :

```yaml
volumeMounts:
  - name: data
    mountPath: /data
    readOnly: true
```

Cela ne supprime pas tous les risques, mais réduit les possibilités de modification.

---

## 6.16.4. Root filesystem en lecture seule

Pour durcir un conteneur, nous pouvons rendre le root filesystem en lecture seule.

Docker :

```bash
docker run --read-only image
```

Kubernetes :

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

Il faut alors prévoir des emplacements temporaires explicitement montés, par exemple `/tmp` ou `/run`.

---

# 6.17. Diagnostic avec le namespace mount

## 6.17.1. Quel montage voit ce processus ?

Pour un PID donné :

```bash
cat /proc/<PID>/mountinfo
```

ou :

```bash
sudo nsenter --target <PID> --mount findmnt
```

Nous pouvons alors comprendre :

- quel rootfs est vu ;
    
- quels volumes sont montés ;
    
- quels chemins viennent de l’hôte ;
    
- quels montages sont en lecture seule ;
    
- si `/proc`, `/sys`, `/dev` sont présents ;
    
- quelles propagations sont configurées.
    

---

## 6.17.2. Pourquoi mon fichier n’est-il pas visible ?

Question fréquente :

```text
J’ai créé un fichier sur l’hôte, pourquoi le conteneur ne le voit-il pas ?
```

Causes possibles :

- le chemin n’est pas monté dans le conteneur ;
    
- un autre montage masque le répertoire ;
    
- le conteneur a un rootfs différent ;
    
- le fichier est créé dans une couche writable différente ;
    
- mauvais chemin hôte ;
    
- permissions insuffisantes ;
    
- montage en lecture seule ;
    
- propagation non adaptée.
    

Le namespace mount est souvent au cœur de ce diagnostic.

---

## 6.17.3. Pourquoi mon montage apparaît sur l’hôte ?

Cause fréquente :

```text
propagation shared
```

Nous vérifions :

```bash
findmnt -o TARGET,PROPAGATION /
```

ou dans le namespace concerné :

```bash
findmnt -o TARGET,PROPAGATION
```

Nous pouvons corriger dans un TP avec :

```bash
mount --make-rprivate /
```

Dans un système réel, nous devons comprendre l’impact avant de changer la propagation.

---

# 6.18. Expérience complète guidée

## 6.18.1. Créer un namespace mount isolé

Nous lançons :

```bash
unshare --mount bash
```

Dans le shell :

```bash
echo "Namespace mount : $(readlink /proc/$$/ns/mnt)"
findmnt -o TARGET,PROPAGATION / | cat
```

Nous rendons la propagation privée pour le TP :

```bash
mount --make-rprivate /
```

---

## 6.18.2. Créer un montage visible seulement dans le namespace

Dans le shell isolé :

```bash
mkdir -p /tmp/mnt-ns-demo
mount -t tmpfs tmpfs /tmp/mnt-ns-demo
echo "bonjour depuis le namespace mount" > /tmp/mnt-ns-demo/message.txt
cat /tmp/mnt-ns-demo/message.txt
findmnt /tmp/mnt-ns-demo
```

Dans un autre terminal sur l’hôte :

```bash
findmnt /tmp/mnt-ns-demo
ls /tmp/mnt-ns-demo
```

Normalement, le montage ne doit pas être visible comme montage depuis l’hôte, si la propagation a été rendue privée correctement.

---

## 6.18.3. Observer depuis l’hôte avec `nsenter`

Nous récupérons le PID du shell isolé :

```bash
echo $$
```

Depuis l’hôte :

```bash
sudo nsenter --target <PID> --mount findmnt /tmp/mnt-ns-demo
sudo nsenter --target <PID> --mount cat /tmp/mnt-ns-demo/message.txt
```

Nous voyons le montage depuis le namespace du processus.

---

## 6.18.4. Nettoyage

Dans le shell isolé :

```bash
umount /tmp/mnt-ns-demo
exit
```

Si nécessaire depuis l’hôte :

```bash
sudo umount /tmp/mnt-ns-demo 2>/dev/null
```

---

# 6.19. Pièges classiques

## 6.19.1. Croire que `unshare --mount` isole tout automatiquement

`unshare --mount` crée un nouveau namespace mount, mais la propagation peut encore produire des effets inattendus.

Pour un TP propre :

```bash
mount --make-rprivate /
```

---

## 6.19.2. Confondre fichiers et montages

Deux processus peuvent voir le même fichier sur disque, mais avec des montages différents.

Inversement, un même chemin peut pointer vers des contenus différents selon le namespace mount.

Exemple :

```text
/tmp/data dans le namespace A → tmpfs A
/tmp/data dans le namespace B → répertoire réel de l’hôte
```

Même chemin, réalité différente.

---

## 6.19.3. Monter par-dessus un répertoire existant

Si nous montons un système de fichiers sur un répertoire qui contient déjà des fichiers, ces fichiers ne sont pas supprimés, mais ils sont masqués tant que le montage est actif.

Exemple :

```bash
mkdir -p /tmp/demo
echo "avant" > /tmp/demo/fichier.txt
mount -t tmpfs tmpfs /tmp/demo
ls /tmp/demo
```

Le fichier `fichier.txt` semble disparaître, mais il réapparaît après :

```bash
umount /tmp/demo
ls /tmp/demo
```

Nous devons comprendre cette notion de masquage.

---

## 6.19.4. Confondre `chroot` et conteneur

`chroot` ne suffit pas à créer un conteneur.

Il faut au minimum réfléchir à :

- namespace mount ;
    
- namespace PID ;
    
- namespace UTS ;
    
- namespace réseau ;
    
- user namespace ;
    
- cgroups ;
    
- capabilities ;
    
- `/proc`, `/sys`, `/dev` ;
    
- sécurité des montages.
    

---

## 6.19.5. Exposer le socket Docker

Le montage suivant est extrêmement sensible :

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

Un processus dans le conteneur peut alors contrôler Docker sur l’hôte, ce qui revient souvent à obtenir un pouvoir très important sur la machine.

Nous évitons ce montage sauf besoin très précis et parfaitement compris.

---

# 6.20. Exercices

## Exercice 1 — Observer le namespace mount courant

Nous exécutons :

```bash
readlink /proc/$$/ns/mnt
readlink /proc/1/ns/mnt
findmnt | head
cat /proc/self/mountinfo | head
```

Nous répondons :

1. Le shell partage-t-il le namespace mount du PID 1 ?
    
2. Quels sont les premiers montages visibles ?
    
3. Quelle différence voyons-nous entre `findmnt` et `mountinfo` ?
    

---

## Exercice 2 — Créer un montage isolé

Nous exécutons :

```bash
unshare --mount bash
mount --make-rprivate /
mkdir -p /tmp/ns-mount-demo
mount -t tmpfs tmpfs /tmp/ns-mount-demo
echo "test namespace mount" > /tmp/ns-mount-demo/test.txt
findmnt /tmp/ns-mount-demo
```

Depuis l’hôte :

```bash
findmnt /tmp/ns-mount-demo
ls /tmp/ns-mount-demo
```

Nous répondons :

1. Le montage est-il visible depuis l’hôte ?
    
2. Le fichier est-il visible depuis l’hôte ?
    
3. Pourquoi avons-nous utilisé `mount --make-rprivate /` ?
    

---

## Exercice 3 — Observer la propagation

Nous exécutons :

```bash
findmnt -o TARGET,PROPAGATION /
cat /proc/self/mountinfo | grep -E 'shared|master' | head
```

Nous répondons :

1. La racine est-elle `shared`, `private` ou autre ?
    
2. Voyons-nous des groupes `shared` ?
    
3. Pourquoi cela peut-il influencer les namespaces mount ?
    

---

## Exercice 4 — Tester un bind mount

Nous exécutons :

```bash
mkdir -p /tmp/source-bind /tmp/dest-bind
echo "contenu bind" > /tmp/source-bind/fichier.txt

unshare --mount bash
mount --make-rprivate /
mount --bind /tmp/source-bind /tmp/dest-bind
cat /tmp/dest-bind/fichier.txt
findmnt /tmp/dest-bind
umount /tmp/dest-bind
exit
```

Nous répondons :

1. Que fait `mount --bind` ?
    
2. Le contenu est-il copié ?
    
3. Pourquoi les bind mounts sont-ils importants pour Docker ?
    

---

## Exercice 5 — Masquage par montage

Nous exécutons :

```bash
mkdir -p /tmp/mask-demo
echo "avant montage" > /tmp/mask-demo/fichier.txt
ls /tmp/mask-demo

unshare --mount bash
mount --make-rprivate /
mount -t tmpfs tmpfs /tmp/mask-demo
ls /tmp/mask-demo
echo "dans tmpfs" > /tmp/mask-demo/autre.txt
ls /tmp/mask-demo
umount /tmp/mask-demo
ls /tmp/mask-demo
exit
```

Nous répondons :

1. Le fichier initial a-t-il été supprimé ?
    
2. Pourquoi disparaît-il pendant le montage ?
    
3. Pourquoi réapparaît-il après `umount` ?
    

---

## Exercice 6 — Comparer `chroot` et namespace mount

Nous réfléchissons à la commande :

```bash
chroot /tmp/rootfs /bin/sh
```

Nous répondons :

1. Que change `chroot` ?
    
2. Que ne change-t-il pas ?
    
3. Pourquoi `chroot` seul ne suffit-il pas pour créer un conteneur ?
    
4. Quel rôle joue le namespace mount en plus ?
    

---

## Exercice 7 — Observer un conteneur Docker

Si Docker est disponible :

```bash
docker run --rm -d --name mnt-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' mnt-demo)

sudo readlink /proc/$pid/ns/mnt
readlink /proc/1/ns/mnt

sudo nsenter --target "$pid" --mount findmnt | head
sudo cat /proc/$pid/mountinfo | head

docker rm -f mnt-demo
```

Nous répondons :

1. Le conteneur partage-t-il le namespace mount de l’hôte ?
    
2. Quels montages voyons-nous dans le conteneur ?
    
3. Que représente le rootfs du conteneur ?
    
4. Pourquoi Docker utilise-t-il un namespace mount ?
    

---

# 6.21. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace mount isole la vue des points de montage.
    
2. Il permet à un processus de voir une arborescence de montages différente de celle de l’hôte.
    
3. Le namespace mount est visible avec `/proc/<PID>/ns/mnt`.
    
4. Les montages visibles sont consultables avec `findmnt`, `/proc/self/mounts` et `/proc/self/mountinfo`.
    
5. `unshare --mount bash` crée un nouveau namespace mount.
    
6. La propagation des montages peut produire des effets inattendus.
    
7. `mount --make-rprivate /` est souvent utile en TP pour éviter les propagations.
    
8. Les modes principaux sont `shared`, `private`, `slave` et `unbindable`.
    
9. Un bind mount remonte un chemin existant ailleurs dans l’arborescence.
    
10. Les bind mounts sont essentiels pour les volumes Docker et Kubernetes.
    
11. Monter `/proc` correctement est indispensable dans un namespace PID.
    
12. Monter `/proc`, `/sys` ou `/dev` de l’hôte dans un conteneur peut être dangereux.
    
13. `chroot` change la racine apparente, mais ne suffit pas à créer une isolation complète.
    
14. `pivot_root` est plus adapté à la construction d’un vrai rootfs isolé.
    
15. Le root filesystem d’un conteneur est présenté grâce au namespace mount.
    
16. La sécurité dépend autant de ce qui est monté que du namespace lui-même.
    

---

# Conclusion du chapitre 6

Nous avons étudié le namespace mount, qui est l’un des mécanismes fondamentaux des conteneurs Linux.

Nous savons maintenant qu’il permet d’isoler la vue des montages, de présenter un root filesystem spécifique à un processus, de monter `/proc`, `/sys`, `/dev`, des volumes ou des bind mounts, et de construire une arborescence différente de celle de l’hôte.

Nous avons aussi vu que ce namespace demande une grande prudence. La propagation des montages peut surprendre, les bind mounts peuvent exposer l’hôte, et des montages sensibles comme `/proc`, `/sys`, `/dev` ou le socket Docker peuvent réduire fortement l’isolation.

Nous retenons surtout que le namespace mount ne se limite pas à “changer des fichiers visibles”. Il définit une partie essentielle de la frontière entre l’hôte et l’environnement isolé.

Dans le chapitre suivant, nous étudions le namespace network, qui permet à un processus d’avoir sa propre pile réseau, ses propres interfaces, ses routes, ses ports et ses sockets.

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

# 7.1. Pourquoi isoler le réseau ?

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

# 7.2. Qu’est-ce que le namespace network ?

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

# 7.3. Observer le namespace réseau courant

## 7.3.1. Avec `/proc/<PID>/ns/net`

Nous pouvons observer le namespace réseau de notre shell courant :

```bash
readlink /proc/$$/ns/net
```

Exemple :

```text
net:[4026531840]
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

# 7.4. Créer un namespace réseau avec `unshare`

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

# 7.5. Namespace réseau nommé avec `ip netns`

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

# 7.6. Connecter un namespace réseau à l’hôte avec une paire `veth`

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

# 7.7. Ajouter une route par défaut

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

# 7.8. Activer le forwarding IPv4

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

# 7.9. NAT avec nftables ou iptables

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

# 7.10. DNS dans un namespace réseau

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

# 7.11. Bridge Linux

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

# 7.12. Lien avec Docker bridge

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

# 7.13. Lien avec Kubernetes et CNI

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

# 7.14. `/proc/net` et namespace réseau

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

# 7.15. Pare-feu et règles réseau

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

# 7.16. Sécurité du namespace réseau

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

# 7.17. Diagnostic réseau avec namespaces

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

# 7.18. Expérience complète guidée

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

# 7.19. Pièges classiques

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

# 7.20. Exercices

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

# 7.21. Ce que nous devons retenir

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

# Conclusion du chapitre 7

Nous avons étudié le namespace network, qui est l’un des namespaces les plus importants pour les conteneurs et l’isolation applicative.

Nous savons maintenant créer un namespace réseau, activer son loopback, le connecter à l’hôte avec une paire `veth`, configurer des adresses IP, ajouter des routes, comprendre le forwarding, le NAT et le rôle d’un bridge Linux.

Nous avons aussi relié ces notions aux environnements réels : Docker utilise des bridges, des paires `veth` et du NAT ; Kubernetes confie la configuration réseau aux plugins CNI et fait généralement partager le namespace réseau aux conteneurs d’un même pod.

Nous retenons surtout qu’un problème réseau dans un conteneur ne se diagnostique pas comme un problème réseau classique. Nous devons toujours nous demander dans quel namespace nous observons les interfaces, les routes, les sockets et les ports.

Dans le chapitre suivant, nous étudions le namespace IPC, qui isole certains mécanismes de communication inter-processus comme les files de messages, les sémaphores et la mémoire partagée.

# Chapitre 8 — Namespace IPC

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace IPC.

Le namespace IPC permet d’isoler certains mécanismes de communication inter-processus. IPC signifie _Inter-Process Communication_, c’est-à-dire communication entre processus.

Sous Linux, plusieurs mécanismes permettent à des processus de communiquer localement :

- files de messages System V ;
    
- sémaphores System V ;
    
- mémoire partagée System V ;
    
- mémoire partagée POSIX ;
    
- files de messages POSIX ;
    
- certains objets IPC nommés.
    

Le namespace IPC permet de séparer ces ressources entre groupes de processus. C’est utile pour éviter qu’un processus dans un conteneur voie ou manipule des ressources IPC appartenant à l’hôte ou à un autre conteneur.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace IPC ;
    
- comprendre ce que sont les mécanismes IPC ;
    
- observer les ressources IPC avec `ipcs` ;
    
- créer un namespace IPC avec `unshare`;
    
- comparer les ressources IPC entre l’hôte et un namespace isolé ;
    
- comprendre le lien avec Docker, Podman et Kubernetes ;
    
- identifier les risques liés au partage du namespace IPC avec l’hôte.
    

---

# 8.1. Pourquoi isoler les communications inter-processus ?

## 8.1.1. Le problème de départ

Sur un système Linux, plusieurs processus peuvent communiquer entre eux sans passer par le réseau.

Ils peuvent utiliser des mécanismes IPC pour :

- partager une zone mémoire ;
    
- synchroniser des accès concurrents ;
    
- envoyer des messages ;
    
- coordonner plusieurs processus d’une même application ;
    
- communiquer entre services locaux.
    

Cela peut être très utile, mais cela pose une question d’isolation.

Si tous les processus voient les mêmes ressources IPC, alors un processus non souhaité pourrait potentiellement :

- voir l’existence de ressources IPC d’une autre application ;
    
- interférer avec des sémaphores ;
    
- lire ou modifier une mémoire partagée mal protégée ;
    
- envoyer des messages à une file IPC ;
    
- provoquer des conflits de noms ;
    
- perturber une application.
    

Le namespace IPC permet de donner à un groupe de processus sa propre vue des ressources IPC.

---

## 8.1.2. Exemple conceptuel

Sans namespace IPC isolé :

```text
Hôte
├── application A
│   └── mémoire partagée id=123
├── application B
│   └── sémaphore id=456
└── conteneur C
    └── voit potentiellement certaines ressources IPC globales
```

Avec namespace IPC isolé :

```text
Namespace IPC de l’hôte
├── application A
└── application B

Namespace IPC du conteneur C
└── ressources IPC propres au conteneur
```

Le conteneur ne voit pas les ressources IPC de l’hôte.

---

# 8.2. Qu’est-ce que l’IPC ?

## 8.2.1. Définition générale

IPC signifie _Inter-Process Communication_.

Nous parlons d’IPC lorsqu’un processus communique avec un autre processus sur la même machine.

Il existe plusieurs familles de mécanismes IPC :

```text
pipes
sockets Unix
signaux
files de messages
sémaphores
mémoire partagée
futex
eventfd
```

Tous ces mécanismes ne sont pas isolés de la même façon par le namespace IPC.

Dans ce chapitre, nous nous concentrons surtout sur les IPC System V et POSIX, car ce sont eux qui sont traditionnellement associés au namespace IPC.

---

## 8.2.2. IPC System V

Les IPC System V sont des mécanismes historiques d’Unix.

Ils comprennent :

- files de messages ;
    
- sémaphores ;
    
- segments de mémoire partagée.
    

Nous les observons avec :

```bash
ipcs
```

Sortie typique :

```text
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
```

---

## 8.2.3. IPC POSIX

Les IPC POSIX sont des mécanismes plus modernes ou différents, par exemple :

- mémoire partagée POSIX ;
    
- sémaphores POSIX ;
    
- files de messages POSIX.
    

Ils peuvent apparaître dans des emplacements comme :

```text
/dev/shm
```

ou être gérés par le noyau avec des objets nommés.

Le namespace IPC peut également influencer la visibilité de certaines ressources POSIX IPC.

---

# 8.3. Observer le namespace IPC courant

## 8.3.1. Avec `/proc/<PID>/ns/ipc`

Nous pouvons observer le namespace IPC de notre shell courant :

```bash
readlink /proc/$$/ns/ipc
```

Exemple :

```text
ipc:[4026531839]
```

Nous comparons avec le PID 1 :

```bash
readlink /proc/1/ns/ipc
```

Si les deux valeurs sont identiques, notre shell partage le namespace IPC du PID 1.

---

## 8.3.2. Comparer deux processus

Nous pouvons comparer deux processus :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/ipc)"
echo "PID 1         : $(readlink /proc/1/ns/ipc)"
```

Même valeur :

```text
Shell courant : ipc:[4026531839]
PID 1         : ipc:[4026531839]
```

Les deux processus partagent le même namespace IPC.

Valeurs différentes :

```text
Shell courant : ipc:[4026533000]
PID 1         : ipc:[4026531839]
```

Les deux processus ont des vues IPC différentes.

---

# 8.4. Observer les ressources IPC avec `ipcs`

## 8.4.1. La commande `ipcs`

La commande principale pour observer les ressources IPC System V est :

```bash
ipcs
```

Elle affiche :

- files de messages ;
    
- segments de mémoire partagée ;
    
- tableaux de sémaphores.
    

Nous pouvons aussi filtrer :

```bash
ipcs -q
ipcs -m
ipcs -s
```

Avec :

|Option|Ressource|
|---|---|
|`-q`|files de messages|
|`-m`|mémoire partagée|
|`-s`|sémaphores|

---

## 8.4.2. Exemple de sortie

```text
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x12345678 0          user       600        0            0

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x87654321 1          user       600        4096       1

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x11111111 2          user       600        1
```

Nous voyons que chaque ressource possède :

- une clé ;
    
- un identifiant ;
    
- un propriétaire ;
    
- des permissions ;
    
- des informations propres au type de ressource.
    

---

## 8.4.3. Pourquoi `ipcs` dépend du namespace IPC

La commande `ipcs` affiche les ressources IPC visibles dans le namespace IPC courant.

Donc :

```bash
ipcs
```

sur l’hôte et :

```bash
unshare --ipc bash
ipcs
```

dans un namespace isolé peuvent afficher des résultats différents.

C’est le même outil, mais pas la même vue.

---

# 8.5. Créer un namespace IPC avec `unshare`

## 8.5.1. Expérience simple

Nous créons un nouveau namespace IPC :

```bash
unshare --ipc bash
```

Dans ce shell, nous observons :

```bash
readlink /proc/$$/ns/ipc
ipcs
```

Nous pouvons comparer avec l’hôte dans un autre terminal :

```bash
readlink /proc/1/ns/ipc
ipcs
```

Le namespace IPC est différent.

Les ressources IPC visibles peuvent aussi être différentes.

---

## 8.5.2. Avec les droits nécessaires

Selon la configuration du système, la commande peut échouer :

```bash
unshare --ipc bash
```

Erreur possible :

```text
unshare: unshare failed: Operation not permitted
```

Dans ce cas, nous pouvons utiliser :

```bash
sudo unshare --ipc bash
```

Ou, selon la configuration des user namespaces :

```bash
unshare --user --map-root-user --ipc bash
```

Nous retenons que la création de certains namespaces dépend des permissions et des capacités disponibles.

---

# 8.6. Créer une ressource IPC pour observer l’isolation

## 8.6.1. Problème pratique

Observer l’isolation IPC est moins spectaculaire que changer le hostname ou créer une interface réseau.

Si aucune ressource IPC n’existe, `ipcs` peut afficher une sortie vide dans l’hôte et dans le namespace isolé.

Nous avons donc besoin de créer une ressource IPC de test.

---

## 8.6.2. Créer une mémoire partagée avec Python

Nous pouvons créer une mémoire partagée POSIX avec Python 3 :

```bash
python3 - <<'PY'
from multiprocessing import shared_memory
import time

shm = shared_memory.SharedMemory(create=True, size=1024, name="demo_ipc_namespace")
print("Mémoire partagée créée :", shm.name)
print("PID:", __import__("os").getpid())
try:
    time.sleep(300)
finally:
    shm.close()
    shm.unlink()
PY
```

Selon le système, cette ressource peut être visible via `/dev/shm`.

Dans un autre terminal :

```bash
ls -l /dev/shm | grep demo_ipc_namespace
```

Attention : l’observation dépend aussi du namespace mount, car `/dev/shm` est un montage.

Cela montre que les namespaces peuvent interagir : IPC et mount peuvent tous les deux influencer ce que nous voyons.

---

## 8.6.3. Créer une ressource System V

Les ressources System V sont plus directement observables avec `ipcs`.

Selon les outils installés, nous pouvons utiliser des programmes dédiés ou écrire un petit programme C/Python. Pour un cours général, nous pouvons retenir l’idée suivante :

```text
Créer une ressource IPC System V dans un namespace IPC
→ elle apparaît dans ipcs dans ce namespace
→ elle n’apparaît pas dans ipcs depuis un autre namespace IPC
```

L’objectif pédagogique n’est pas ici d’apprendre toute l’API System V, mais de comprendre l’isolation.

---

# 8.7. Mémoire partagée et `/dev/shm`

## 8.7.1. Rôle de `/dev/shm`

Sur beaucoup de systèmes Linux, `/dev/shm` est un `tmpfs` utilisé pour la mémoire partagée POSIX.

Nous pouvons observer :

```bash
findmnt /dev/shm
```

Sortie possible :

```text
TARGET   SOURCE FSTYPE OPTIONS
/dev/shm tmpfs  tmpfs  rw,nosuid,nodev
```

Les objets de mémoire partagée POSIX peuvent apparaître comme des fichiers dans `/dev/shm`.

---

## 8.7.2. Interaction entre IPC namespace et mount namespace

Il faut être prudent : `/dev/shm` dépend aussi des montages.

Deux processus peuvent avoir :

- le même namespace IPC mais des namespaces mount différents ;
    
- des namespaces IPC différents mais un `/dev/shm` monté de manière surprenante ;
    
- un `/dev/shm` partagé par erreur entre conteneurs.
    

Dans les conteneurs, `/dev/shm` est souvent monté spécifiquement.

Docker permet par exemple de contrôler sa taille avec :

```bash
docker run --shm-size=256m ...
```

Nous voyons donc que le diagnostic IPC ne se limite pas au namespace IPC seul.

---

## 8.7.3. `/dev/shm` dans Docker

Dans un conteneur Docker :

```bash
df -h /dev/shm
mount | grep shm
```

Nous pouvons voir un `tmpfs` dédié.

Par défaut, Docker donne souvent une taille limitée à `/dev/shm`.

Cela peut provoquer des erreurs pour certaines applications utilisant beaucoup de mémoire partagée, par exemple :

- navigateurs ;
    
- bases de données ;
    
- calcul scientifique ;
    
- traitement vidéo ;
    
- tests headless ;
    
- Chrome/Chromium en conteneur.
    

---

# 8.8. Sémaphores

## 8.8.1. Qu’est-ce qu’un sémaphore ?

Un sémaphore est un mécanisme de synchronisation.

Il permet à plusieurs processus de coordonner l’accès à une ressource partagée.

Exemple conceptuel :

```text
Un seul processus à la fois peut écrire dans une zone critique.
Le sémaphore sert à contrôler l’accès.
```

Les sémaphores peuvent être utilisés par des bases de données, des services système ou des applications multi-processus.

---

## 8.8.2. Observer les sémaphores System V

Nous utilisons :

```bash
ipcs -s
```

Sortie possible :

```text
------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x00000000 3          user       600        1
```

Dans un namespace IPC isolé, nous ne voyons que les sémaphores de ce namespace.

---

# 8.9. Files de messages

## 8.9.1. Rôle des files de messages

Les files de messages IPC permettent à des processus d’échanger des messages structurés via le noyau.

Elles sont moins utilisées dans beaucoup d’applications modernes que les sockets Unix ou les files applicatives, mais elles existent toujours.

---

## 8.9.2. Observer les files de messages

Nous utilisons :

```bash
ipcs -q
```

Sortie possible :

```text
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
```

Si aucune file n’existe, la table est vide.

Dans un namespace IPC isolé, les files de messages de l’hôte ne sont pas visibles.

---

# 8.10. Lien avec Docker

## 8.10.1. IPC namespace par défaut

Par défaut, Docker crée généralement un namespace IPC propre au conteneur.

Nous pouvons tester :

```bash
docker run --rm -d --name ipc-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' ipc-demo)

sudo readlink /proc/$pid/ns/ipc
readlink /proc/1/ns/ipc
```

Si les valeurs sont différentes, le conteneur a son propre namespace IPC.

Nous nettoyons :

```bash
docker rm -f ipc-demo
```

---

## 8.10.2. Option `--ipc=host`

Docker permet de partager le namespace IPC de l’hôte :

```bash
docker run --rm -it --ipc=host alpine sh
```

Dans ce cas, le conteneur voit les ressources IPC de l’hôte.

C’est une option sensible.

Elle peut être utile pour certaines applications très spécifiques, mais elle réduit l’isolation.

Nous retenons :

```text
--ipc=host signifie que le conteneur partage le namespace IPC de l’hôte.
```

---

## 8.10.3. Option `--ipc=container:<id>`

Docker permet aussi de partager le namespace IPC d’un autre conteneur :

```bash
docker run --ipc=container:<container> ...
```

Cela peut permettre à deux conteneurs de partager explicitement des ressources IPC.

C’est rarement nécessaire pour des applications modernes, mais cela existe.

---

## 8.10.4. Taille de `/dev/shm`

Docker permet de définir la taille de `/dev/shm` :

```bash
docker run --shm-size=512m image
```

C’est important pour certaines applications.

Si une application échoue avec des erreurs liées à la mémoire partagée, nous devons penser à vérifier :

```bash
df -h /dev/shm
```

dans le conteneur.

---

# 8.11. Lien avec Kubernetes

## 8.11.1. IPC dans Kubernetes

Dans Kubernetes, les pods ont généralement leurs propres namespaces.

Le partage exact du namespace IPC dépend de la configuration du runtime et des options du pod.

Kubernetes permet notamment une option sensible :

```yaml
hostIPC: true
```

Cela fait partager au pod le namespace IPC de l’hôte.

---

## 8.11.2. `hostIPC: true`

Exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostipc-demo
spec:
  hostIPC: true
  containers:
    - name: app
      image: alpine
      command: ["sleep", "3600"]
```

Avec `hostIPC: true`, le pod peut voir les ressources IPC de l’hôte.

C’est une option très sensible.

Nous ne l’utilisons que pour des cas d’administration ou de supervision très spécifiques.

---

## 8.11.3. Mémoire partagée dans Kubernetes

Certains workloads nécessitent davantage de mémoire partagée.

Kubernetes peut utiliser un volume `emptyDir` en mémoire :

```yaml
volumes:
  - name: dshm
    emptyDir:
      medium: Memory
      sizeLimit: 1Gi

containers:
  - name: app
    image: example/app
    volumeMounts:
      - mountPath: /dev/shm
        name: dshm
```

Cela permet d’augmenter ou de contrôler `/dev/shm` dans le conteneur.

---

# 8.12. Namespace IPC et sécurité

## 8.12.1. Ce que le namespace IPC protège

Le namespace IPC évite qu’un processus voie ou manipule directement certaines ressources IPC d’un autre environnement.

Il protège notamment contre :

- observation de segments de mémoire partagée ;
    
- interaction avec des sémaphores ;
    
- collisions de ressources IPC ;
    
- accès involontaire à des files de messages ;
    
- perturbation entre applications.
    

---

## 8.12.2. Ce qu’il ne protège pas seul

Le namespace IPC ne suffit pas à sécuriser un conteneur.

Nous devons aussi considérer :

- user namespace ;
    
- mount namespace ;
    
- permissions Unix ;
    
- capabilities ;
    
- seccomp ;
    
- AppArmor ou SELinux ;
    
- exposition de `/dev/shm` ;
    
- options Docker/Kubernetes ;
    
- volumes montés ;
    
- droits sur les objets IPC.
    

Comme souvent, l’isolation réelle repose sur plusieurs couches.

---

## 8.12.3. Risques de `--ipc=host` et `hostIPC`

Les options suivantes réduisent fortement l’isolation :

```bash
docker run --ipc=host ...
```

et :

```yaml
hostIPC: true
```

Elles peuvent permettre à un processus conteneurisé de voir ou d’interagir avec des ressources IPC de l’hôte.

Nous les évitons sauf cas justifié.

---

# 8.13. Diagnostic IPC

## 8.13.1. Questions à se poser

Lorsqu’une application a un problème lié à l’IPC, nous demandons :

```text
Dans quel namespace IPC tourne-t-elle ?
Voit-elle les ressources IPC attendues ?
Utilise-t-elle System V IPC ou POSIX IPC ?
Utilise-t-elle /dev/shm ?
La taille de /dev/shm est-elle suffisante ?
Les permissions sont-elles correctes ?
Le conteneur partage-t-il l’IPC de l’hôte ?
Un autre processus doit-il partager le même namespace IPC ?
```

---

## 8.13.2. Commandes utiles

Observer le namespace IPC :

```bash
readlink /proc/<PID>/ns/ipc
```

Observer les ressources IPC System V :

```bash
ipcs
ipcs -m
ipcs -s
ipcs -q
```

Observer `/dev/shm` :

```bash
df -h /dev/shm
ls -lh /dev/shm
findmnt /dev/shm
```

Entrer dans le namespace IPC d’un processus :

```bash
sudo nsenter --target <PID> --ipc bash
```

Ou avec d’autres namespaces utiles :

```bash
sudo nsenter --target <PID> --ipc --mount --pid --fork bash
```

---

## 8.13.3. Exemple : application Chrome en conteneur

Certaines applications comme Chrome/Chromium en mode headless peuvent utiliser beaucoup `/dev/shm`.

Symptôme possible :

```text
crash du navigateur
erreurs de mémoire partagée
problèmes aléatoires en tests end-to-end
```

Diagnostic :

```bash
df -h /dev/shm
```

Solution possible avec Docker :

```bash
docker run --shm-size=1g ...
```

Solution possible avec Kubernetes :

```yaml
emptyDir:
  medium: Memory
```

Ici, le problème n’est pas seulement le namespace IPC, mais aussi le montage et la taille de `/dev/shm`.

---

# 8.14. Expérience complète guidée

## 8.14.1. Comparer l’IPC de l’hôte et d’un namespace isolé

Sur l’hôte :

```bash
readlink /proc/$$/ns/ipc
ipcs
```

Nous créons un namespace IPC :

```bash
unshare --ipc bash
```

Dans le shell isolé :

```bash
readlink /proc/$$/ns/ipc
ipcs
```

Nous comparons les identifiants et les ressources visibles.

---

## 8.14.2. Entrer dans le namespace IPC depuis un second terminal

Dans le shell isolé :

```bash
echo $$
sleep 1000
```

Dans un second terminal :

```bash
sudo nsenter --target <PID> --ipc bash
readlink /proc/$$/ns/ipc
ipcs
```

Nous voyons le même namespace IPC que le processus cible.

Nous quittons :

```bash
exit
```

---

## 8.14.3. Nettoyage

Dans le shell isolé :

```bash
exit
```

Si un processus `sleep` a été lancé :

```bash
kill <PID>
```

Quand le dernier processus du namespace IPC disparaît, le namespace est détruit, sauf s’il est maintenu par une autre référence.

---

# 8.15. Pièges classiques

## 8.15.1. Croire que toutes les communications locales passent par le namespace IPC

Le namespace IPC n’isole pas tous les mécanismes de communication.

Par exemple :

- les sockets réseau relèvent du namespace network ;
    
- les sockets Unix peuvent dépendre du système de fichiers et des montages ;
    
- les pipes anonymes dépendent surtout des relations entre processus ;
    
- les signaux dépendent des permissions et du namespace PID ;
    
- les fichiers partagés dépendent du namespace mount et des permissions.
    

Nous devons donc identifier le mécanisme de communication exact.

---

## 8.15.2. Confondre `/dev/shm` et namespace IPC

`/dev/shm` est un montage `tmpfs`.

Il est souvent utilisé pour la mémoire partagée POSIX, mais son comportement dépend aussi du namespace mount.

Un problème de `/dev/shm` peut être lié :

- à sa taille ;
    
- à son montage ;
    
- aux permissions ;
    
- au namespace IPC ;
    
- au runtime de conteneur.
    

---

## 8.15.3. Utiliser `--ipc=host` pour contourner un problème

Il peut être tentant de résoudre un problème en lançant :

```bash
docker run --ipc=host ...
```

Mais cela réduit l’isolation.

Nous devons d’abord comprendre le problème :

- manque de taille `/dev/shm` ?
    
- besoin réel de partager des IPC entre processus ?
    
- mauvaise configuration de volume ?
    
- problème de permissions ?
    

Souvent, une solution plus propre existe.

---

## 8.15.4. Oublier que les ressources IPC peuvent persister

Certaines ressources IPC System V peuvent persister après la fin d’un processus si elles ne sont pas supprimées correctement.

Nous pouvons les voir avec :

```bash
ipcs
```

Et les supprimer avec :

```bash
ipcrm
```

Par exemple :

```bash
ipcrm -m <shmid>
ipcrm -s <semid>
ipcrm -q <msqid>
```

Nous utilisons `ipcrm` avec prudence, car supprimer une ressource IPC utilisée par une application peut la perturber.

---

# 8.16. Exercices

## Exercice 1 — Observer le namespace IPC courant

Nous exécutons :

```bash
readlink /proc/$$/ns/ipc
readlink /proc/1/ns/ipc
ipcs
```

Nous répondons :

1. Le shell courant partage-t-il le namespace IPC du PID 1 ?
    
2. Quelles ressources IPC voyons-nous ?
    
3. `ipcs` affiche-t-il des messages, de la mémoire partagée ou des sémaphores ?
    

---

## Exercice 2 — Créer un namespace IPC

Nous exécutons :

```bash
unshare --ipc bash
readlink /proc/$$/ns/ipc
ipcs
exit
```

Nous répondons :

1. L’identifiant du namespace IPC a-t-il changé ?
    
2. Les ressources visibles sont-elles les mêmes que sur l’hôte ?
    
3. Pourquoi ce namespace est-il utile pour les conteneurs ?
    

---

## Exercice 3 — Entrer dans un namespace IPC

Dans un premier terminal :

```bash
unshare --ipc bash
echo $$
sleep 1000
```

Dans un second terminal :

```bash
sudo nsenter --target <PID> --ipc bash
readlink /proc/$$/ns/ipc
ipcs
exit
```

Nous répondons :

1. Avons-nous créé un nouveau namespace ou rejoint un namespace existant ?
    
2. Quelle option de `nsenter` avons-nous utilisée ?
    
3. Les identifiants IPC sont-ils identiques ?
    

---

## Exercice 4 — Observer `/dev/shm`

Nous exécutons :

```bash
df -h /dev/shm
findmnt /dev/shm
ls -lh /dev/shm
```

Nous répondons :

1. Quel est le type de système de fichiers de `/dev/shm` ?
    
2. Quelle est sa taille ?
    
3. Pourquoi `/dev/shm` est-il important pour certaines applications ?
    
4. Pourquoi son comportement dépend-il aussi du namespace mount ?
    

---

## Exercice 5 — Observer Docker et IPC

Si Docker est disponible :

```bash
docker run --rm -d --name ipc-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' ipc-demo)

sudo readlink /proc/$pid/ns/ipc
readlink /proc/1/ns/ipc

docker rm -f ipc-demo
```

Nous répondons :

1. Le conteneur partage-t-il le namespace IPC de l’hôte ?
    
2. Comment le vérifions-nous ?
    
3. Que ferait l’option `--ipc=host` ?
    

---

## Exercice 6 — Taille de `/dev/shm` dans Docker

Si Docker est disponible :

```bash
docker run --rm alpine df -h /dev/shm
docker run --rm --shm-size=512m alpine df -h /dev/shm
```

Nous répondons :

1. Quelle est la taille par défaut ?
    
2. Que change `--shm-size` ?
    
3. Dans quels cas cela peut-il être nécessaire ?
    

---

## Exercice 7 — Réfléchir à `hostIPC`

Nous lisons cette configuration Kubernetes :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostipc-demo
spec:
  hostIPC: true
  containers:
    - name: app
      image: alpine
      command: ["sleep", "3600"]
```

Nous répondons :

1. Que signifie `hostIPC: true` ?
    
2. Pourquoi est-ce sensible ?
    
3. Dans quels cas cela pourrait-il être justifié ?
    
4. Quelle alternative chercherions-nous avant d’utiliser cette option ?
    

---

# 8.17. Ce que nous devons retenir

Nous retenons les points suivants :

1. IPC signifie communication inter-processus.
    
2. Le namespace IPC isole certaines ressources IPC entre groupes de processus.
    
3. Il concerne notamment les IPC System V : files de messages, sémaphores et mémoire partagée.
    
4. Il influence aussi certains mécanismes POSIX IPC.
    
5. Le namespace IPC d’un processus est visible avec `/proc/<PID>/ns/ipc`.
    
6. La commande `ipcs` affiche les ressources IPC visibles dans le namespace courant.
    
7. `unshare --ipc bash` permet de créer un nouveau namespace IPC.
    
8. `nsenter --target <PID> --ipc bash` permet d’entrer dans le namespace IPC d’un processus.
    
9. `/dev/shm` est souvent utilisé pour la mémoire partagée POSIX, mais dépend aussi du namespace mount.
    
10. Docker crée généralement un namespace IPC isolé par conteneur.
    
11. `--ipc=host` fait partager le namespace IPC de l’hôte et réduit l’isolation.
    
12. Kubernetes propose `hostIPC: true`, option sensible à éviter sans justification forte.
    
13. Les problèmes IPC peuvent venir du namespace IPC, mais aussi de `/dev/shm`, des permissions, des montages ou de la taille disponible.
    
14. Certaines ressources IPC peuvent persister et nécessiter un nettoyage avec `ipcrm`.
    
15. Le namespace IPC est une couche d’isolation utile, mais il ne remplace pas les autres mécanismes de sécurité.
    

---

# Conclusion du chapitre 8

Nous avons étudié le namespace IPC, qui isole certains mécanismes de communication inter-processus.

Nous savons maintenant observer le namespace IPC d’un processus, utiliser `ipcs` pour voir les ressources IPC System V, créer un namespace IPC avec `unshare`, et entrer dans un namespace existant avec `nsenter`.

Nous avons aussi compris que l’IPC ne doit pas être analysé isolément. Dans les environnements conteneurisés, les problèmes de mémoire partagée impliquent souvent `/dev/shm`, donc aussi le namespace mount et la configuration du runtime.

Nous retenons surtout que le namespace IPC évite que des processus isolés voient ou manipulent les ressources IPC de l’hôte ou d’un autre conteneur. Comme toujours avec les namespaces, cette isolation est utile, mais elle ne suffit pas à elle seule : elle doit être combinée avec les permissions, les cgroups, les capabilities et les politiques de sécurité.

Dans le chapitre suivant, nous étudions le namespace user, l’un des namespaces les plus importants et les plus subtils, car il permet d’être `root` dans un namespace sans être réellement `root` sur l’hôte.

# Chapitre 9 — Namespace user

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace user.

C’est l’un des namespaces les plus importants, mais aussi l’un des plus subtils. Il permet d’isoler les identifiants utilisateurs et groupes, c’est-à-dire les UID et les GID.

Son idée centrale est la suivante :

```text
Nous pouvons être root dans un namespace sans être root sur l’hôte.
```

Cela paraît paradoxal au premier abord, mais c’est un mécanisme fondamental pour les conteneurs rootless et pour la réduction des privilèges.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace user ;
    
- comprendre les mappings UID/GID ;
    
- lire `/proc/<PID>/uid_map` et `/proc/<PID>/gid_map` ;
    
- créer un user namespace avec `unshare` ;
    
- comprendre ce que signifie être `root` dans un namespace ;
    
- expliquer le lien avec les capabilities ;
    
- comprendre le rôle de `/etc/subuid` et `/etc/subgid` ;
    
- comprendre le rôle de `newuidmap` et `newgidmap` ;
    
- faire le lien avec Docker, Podman et les conteneurs rootless ;
    
- identifier les limites et risques de sécurité.
    

---

# 9.1. Pourquoi isoler les utilisateurs ?

## 9.1.1. Le problème de départ

Sur un système Linux classique, les utilisateurs sont identifiés par des UID et des GID.

Nous pouvons afficher notre identité avec :

```bash
id
```

Exemple :

```text
uid=1000(user) gid=1000(user) groupes=1000(user),27(sudo)
```

L’utilisateur `root` possède généralement l’UID `0`.

```text
uid=0(root)
```

Sur l’hôte, l’UID `0` est extrêmement puissant. Il peut administrer le système, modifier des fichiers sensibles, gérer des processus, monter des systèmes de fichiers, configurer le réseau, charger certains paramètres, etc.

Le problème est donc le suivant :

```text
Comment permettre à un processus d’être root dans un environnement isolé,
sans lui donner les droits root sur toute la machine ?
```

Le namespace user répond à cette question.

---

## 9.1.2. L’idée fondamentale

Un namespace user permet de créer une correspondance entre les UID/GID vus à l’intérieur du namespace et les UID/GID réels à l’extérieur.

Exemple conceptuel :

```text
Dans le namespace :
UID 0 → root

Sur l’hôte :
UID 0 du namespace correspond à UID 1000
```

Le processus se croit root dans son namespace, mais le noyau sait qu’à l’extérieur, il correspond à un utilisateur non privilégié.

C’est le mécanisme de mapping UID/GID.

---

# 9.2. Comprendre UID, GID et root

## 9.2.1. UID

L’UID identifie un utilisateur.

Exemples courants :

```text
UID 0     → root
UID 1000  → premier utilisateur classique sur beaucoup de distributions
UID 33    → www-data sur Debian/Ubuntu
```

Nous pouvons voir l’UID courant avec :

```bash
id -u
```

---

## 9.2.2. GID

Le GID identifie un groupe.

Nous pouvons voir notre groupe principal avec :

```bash
id -g
```

Et tous nos groupes avec :

```bash
id
```

Les groupes permettent de donner des permissions à plusieurs utilisateurs.

---

## 9.2.3. Root n’est pas seulement un nom

`root` n’est pas magique parce qu’il s’appelle `root`.

Il est spécial parce que son UID est `0`.

Nous pouvons le vérifier :

```bash
id root
```

Sortie typique :

```text
uid=0(root) gid=0(root) groupes=0(root)
```

Le namespace user permet justement de faire apparaître un UID `0` à l’intérieur, tout en le mappant vers un UID non-root à l’extérieur.

---

# 9.3. Observer le namespace user courant

## 9.3.1. Avec `/proc/<PID>/ns/user`

Nous observons le namespace user de notre shell courant :

```bash
readlink /proc/$$/ns/user
```

Exemple :

```text
user:[4026531837]
```

Nous comparons avec le PID 1 :

```bash
readlink /proc/1/ns/user
```

Si les deux valeurs sont identiques, notre shell partage le même namespace user que le PID 1.

---

## 9.3.2. Lire les mappings UID/GID

Les mappings du processus courant sont visibles dans :

```bash
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sur un système classique, dans le namespace initial, nous pouvons voir :

```text
         0          0 4294967295
```

Cela signifie :

```text
UID interne 0 correspond à UID externe 0
pour une très grande plage d’UID.
```

Autrement dit, dans le namespace user initial, les identifiants sont les mêmes dedans et dehors.

---

# 9.4. Format de `uid_map` et `gid_map`

## 9.4.1. Structure d’une ligne

Une ligne de `uid_map` ou `gid_map` a trois colonnes :

```text
ID_dans_le_namespace   ID_dans_le_parent   taille_de_la_plage
```

Exemple :

```text
         0       1000          1
```

Nous lisons :

```text
UID 0 dans le namespace
correspond à UID 1000 dans le namespace parent
pour une plage de taille 1.
```

Cela signifie que seul l’UID `0` interne est mappé vers l’UID `1000` externe.

---

## 9.4.2. Exemple avec une plage plus large

Autre exemple :

```text
         0     100000      65536
```

Nous lisons :

```text
UID 0 à 65535 dans le namespace
correspondent à UID 100000 à 165535 dans le parent.
```

Cela permet à un conteneur d’avoir de nombreux utilisateurs internes sans utiliser directement les vrais UID de l’hôte.

---

## 9.4.3. Pourquoi les mappings sont essentiels

Sans mapping, le noyau ne sait pas comment traduire les identifiants internes vers l’extérieur.

Un fichier créé par un processus dans le namespace doit avoir un propriétaire réel sur l’hôte.

Le mapping répond à cette question :

```text
Quand le processus dit “je suis UID 0”, quel UID réel utilise-t-on sur l’hôte ?
```

C’est une question centrale pour les permissions de fichiers, les bind mounts et les conteneurs rootless.

---

# 9.5. Créer un user namespace simple

## 9.5.1. Avec `unshare --user`

Nous pouvons créer un user namespace :

```bash
unshare --user bash
```

Dans ce shell :

```bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Selon la configuration, nous pouvons voir une situation où les mappings sont vides ou limités.

Créer un user namespace brut n’est pas toujours très pratique.

Nous utilisons souvent une option plus pédagogique :

```bash
unshare --user --map-root-user bash
```

---

## 9.5.2. Avec `--map-root-user`

Nous lançons :

```bash
unshare --user --map-root-user bash
```

Dans le shell :

```bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sortie possible :

```text
uid=0(root) gid=0(root) groupes=0(root)
```

Puis :

```text
         0       1000          1
```

Cela signifie que nous sommes root à l’intérieur du namespace, mais que cet UID root est mappé vers notre UID réel sur l’hôte.

---

## 9.5.3. Vérifier que nous ne sommes pas root sur l’hôte

Dans le namespace :

```bash
id
```

Nous voyons :

```text
uid=0(root)
```

Mais si nous essayons d’écrire dans un fichier sensible de l’hôte, par exemple :

```bash
touch /root/test-userns
```

nous obtenons généralement :

```text
Permission denied
```

Cela montre que notre root interne n’est pas root global sur l’hôte.

Nous sommes root dans le namespace, pas dans le système entier.

---

# 9.6. Root dans le namespace, utilisateur normal sur l’hôte

## 9.6.1. Exemple avec un fichier

Sur l’hôte, nous créons un dossier de test :

```bash
mkdir -p /tmp/userns-demo
```

Nous lançons :

```bash
unshare --user --map-root-user bash
```

Dans le namespace :

```bash
id
touch /tmp/userns-demo/fichier.txt
ls -ln /tmp/userns-demo/fichier.txt
```

Nous pouvons voir à l’intérieur :

```text
-rw-r--r-- 1 0 0 ... fichier.txt
```

Depuis l’hôte, dans un autre terminal :

```bash
ls -ln /tmp/userns-demo/fichier.txt
```

Nous pouvons voir :

```text
-rw-r--r-- 1 1000 1000 ... fichier.txt
```

Le même fichier a une interprétation différente selon le mapping.

---

## 9.6.2. Même fichier, deux lectures différentes

Dans le namespace :

```text
propriétaire : UID 0
```

Sur l’hôte :

```text
propriétaire : UID 1000
```

Ce n’est pas une contradiction.

C’est la conséquence du mapping UID.

Nous retenons :

```text
Le namespace user modifie la traduction des identifiants.
Il ne change pas magiquement les permissions réelles de l’hôte.
```

---

# 9.7. User namespace et capabilities

## 9.7.1. Rappel sur les capabilities

Sous Linux, les privilèges de root sont divisés en capacités appelées capabilities.

Exemples :

```text
CAP_SYS_ADMIN
CAP_NET_ADMIN
CAP_CHOWN
CAP_SETUID
CAP_SETGID
CAP_KILL
CAP_DAC_OVERRIDE
```

Nous pouvons voir les capabilities du shell courant avec :

```bash
grep Cap /proc/$$/status
```

ou :

```bash
capsh --print
```

si l’outil est installé.

---

## 9.7.2. Capabilities dans un user namespace

Quand nous sommes root dans un user namespace, nous pouvons avoir des capabilities à l’intérieur de ce namespace.

Mais ces capabilities ne valent pas nécessairement sur tout l’hôte.

Exemple :

```bash
unshare --user --map-root-user bash
id
grep Cap /proc/$$/status
```

Nous pouvons voir des capabilities effectives dans le namespace.

Mais cela ne veut pas dire que nous pouvons tout faire sur l’hôte.

---

## 9.7.3. Exemple : monter un système de fichiers

Dans certains cas, être root dans un user namespace peut permettre certaines opérations qui seraient interdites autrement, par exemple monter certains types de systèmes de fichiers autorisés dans un namespace mount combiné.

Exemple pédagogique :

```bash
unshare --user --map-root-user --mount bash
mkdir -p /tmp/userns-mnt
mount -t tmpfs tmpfs /tmp/userns-mnt
```

Selon la distribution et la configuration du noyau, cela peut fonctionner.

Mais cela ne signifie pas que nous avons tous les droits root de l’hôte.

La portée des capabilities est limitée par le namespace user et par les règles du noyau.

---

## 9.7.4. `CAP_SYS_ADMIN` : une capacité très large

`CAP_SYS_ADMIN` est souvent décrite comme une capability très puissante.

Elle couvre de nombreuses opérations système.

Dans un user namespace, avoir `CAP_SYS_ADMIN` à l’intérieur permet certaines actions dans les namespaces associés, mais pas nécessairement sur l’hôte.

Nous devons donc toujours demander :

```text
Dans quel user namespace cette capability est-elle effective ?
Sur quelles ressources s’applique-t-elle ?
Le namespace propriétaire de la ressource est-il le même ?
```

---

# 9.8. Mappings GID et difficulté avec les groupes

## 9.8.1. Pourquoi le GID est parfois plus complexe

Le mapping des groupes peut être plus délicat que celui des utilisateurs.

Linux empêche certaines manipulations de groupes non autorisées, notamment pour éviter qu’un utilisateur obtienne des droits de groupe qu’il ne possède pas réellement.

C’est pour cela que l’écriture dans `gid_map` est encadrée.

---

## 9.8.2. `setgroups`

Nous pouvons voir un fichier :

```bash
cat /proc/$$/setgroups
```

Dans certains cas, avant d’écrire dans `gid_map`, il faut désactiver `setgroups` :

```bash
echo deny > /proc/<PID>/setgroups
```

Cette règle évite qu’un processus non privilégié manipule des groupes de manière dangereuse.

Les outils comme `unshare --map-root-user` masquent souvent cette complexité pour les cas simples.

---

# 9.9. `/etc/subuid` et `/etc/subgid`

## 9.9.1. Pourquoi avons-nous besoin de plages subordonnées ?

Pour un conteneur rootless complet, il ne suffit pas toujours de mapper uniquement :

```text
UID 0 interne → UID 1000 externe
```

Un conteneur peut contenir plusieurs utilisateurs internes :

```text
root
www-data
postgres
redis
app
```

Nous avons donc besoin d’une plage d’UID/GID externes réservés à l’utilisateur.

C’est le rôle de :

```bash
/etc/subuid
/etc/subgid
```

---

## 9.9.2. Exemple de `/etc/subuid`

Nous pouvons lire :

```bash
cat /etc/subuid
```

Exemple :

```text
user:100000:65536
```

Cela signifie :

```text
L’utilisateur user peut utiliser la plage d’UID subordonnés
de 100000 à 165535
pour ses user namespaces.
```

Même principe pour :

```bash
cat /etc/subgid
```

---

## 9.9.3. Mapping typique rootless

Un conteneur rootless peut utiliser un mapping comme :

```text
         0     100000      65536
```

Cela signifie :

```text
UID 0 dans le conteneur      → UID 100000 sur l’hôte
UID 1 dans le conteneur      → UID 100001 sur l’hôte
...
UID 65535 dans le conteneur  → UID 165535 sur l’hôte
```

Ainsi, les utilisateurs internes du conteneur correspondent à des UID non privilégiés sur l’hôte.

---

# 9.10. `newuidmap` et `newgidmap`

## 9.10.1. Rôle de ces outils

Les outils :

```bash
newuidmap
newgidmap
```

permettent de configurer les mappings UID/GID d’un processus de manière contrôlée.

Ils vérifient notamment que l’utilisateur a le droit d’utiliser les plages indiquées dans :

```text
/etc/subuid
/etc/subgid
```

Ces outils sont utilisés par des systèmes comme :

- Podman rootless ;
    
- Buildah ;
    
- certains runtimes OCI rootless ;
    
- outils de sandboxing.
    

---

## 9.10.2. Pourquoi ne pas écrire directement dans `uid_map` ?

Techniquement, les mappings sont exposés dans :

```bash
/proc/<PID>/uid_map
/proc/<PID>/gid_map
```

Mais l’écriture directe est soumise à des règles strictes.

Pour des mappings avancés, nous utilisons `newuidmap` et `newgidmap` afin d’éviter de donner des privilèges excessifs.

---

## 9.10.3. Exemple conceptuel

Pour un processus cible `<PID>`, un outil rootless peut configurer :

```bash
newuidmap <PID> 0 100000 65536
newgidmap <PID> 0 100000 65536
```

Cela signifie :

```text
Dans le namespace du processus <PID>,
les UID/GID 0 à 65535 sont mappés vers 100000 à 165535 sur l’hôte.
```

En pratique, nous ne faisons pas toujours cela à la main. Les outils de conteneurs le font pour nous.

---

# 9.11. User namespace et conteneurs rootless

## 9.11.1. Qu’est-ce qu’un conteneur rootless ?

Un conteneur rootless est un conteneur lancé sans privilèges root sur l’hôte.

L’utilisateur qui lance le conteneur est un utilisateur normal.

À l’intérieur du conteneur, le processus peut voir :

```text
uid=0(root)
```

Mais sur l’hôte, il correspond à un UID non privilégié ou à une plage subordonnée.

---

## 9.11.2. Pourquoi c’est important ?

Les conteneurs rootless réduisent les risques.

Si un processus s’échappe partiellement de son conteneur, il n’arrive pas directement avec les droits root de l’hôte.

Cela ne rend pas le système invulnérable, mais cela réduit fortement certains impacts.

Nous retenons :

```text
Rootless ne veut pas dire sans risque.
Rootless veut dire sans root réel sur l’hôte.
```

---

## 9.11.3. Podman rootless

Podman est souvent utilisé en mode rootless.

Quand nous lançons un conteneur rootless, Podman utilise notamment :

- user namespace ;
    
- mappings UID/GID ;
    
- cgroups adaptés ;
    
- réseau rootless ;
    
- stockage adapté à l’utilisateur ;
    
- runtime OCI.
    

Nous pouvons inspecter les mappings depuis l’hôte en retrouvant le PID du processus et en lisant :

```bash
cat /proc/<PID>/uid_map
cat /proc/<PID>/gid_map
```

---

## 9.11.4. Docker rootless

Docker propose aussi un mode rootless.

Le principe reste similaire : le démon et les conteneurs fonctionnent sans privilèges root globaux.

Là encore, les user namespaces et les plages `/etc/subuid` et `/etc/subgid` sont essentiels.

---

# 9.12. User namespace et fichiers montés

## 9.12.1. Le problème des bind mounts

Les user namespaces peuvent créer des surprises avec les volumes et bind mounts.

Exemple :

```text
Dans le conteneur :
un fichier appartient à root:root

Sur l’hôte :
le même fichier appartient à 100000:100000
```

Ou inversement :

```text
Sur l’hôte :
fichier appartient à UID 1000

Dans le conteneur :
ce fichier peut apparaître comme nobody ou comme un UID non mappé
```

Cela dépend du mapping.

---

## 9.12.2. UID non mappés

Si un fichier appartient à un UID qui n’est pas mappé dans le namespace, il peut apparaître avec un identifiant spécial ou inattendu.

Dans certains cas, il peut apparaître comme :

```text
nobody
```

ou comme un UID numérique non résolu.

Cela peut provoquer des erreurs de permissions dans les conteneurs rootless.

---

## 9.12.3. Diagnostic

Pour diagnostiquer, nous utilisons :

Dans le conteneur ou namespace :

```bash
id
ls -ln /chemin
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sur l’hôte :

```bash
ls -ln /chemin
```

Nous comparons les UID/GID numériques.

Il faut raisonner numériquement, pas seulement avec les noms d’utilisateurs.

---

# 9.13. User namespace et sécurité

## 9.13.1. Ce que le user namespace protège

Le user namespace permet de réduire les privilèges réels sur l’hôte.

Il permet notamment :

- d’avoir root dans le conteneur sans root sur l’hôte ;
    
- de limiter les effets d’une compromission ;
    
- de lancer des conteneurs sans démon root ;
    
- de déléguer certaines isolations à des utilisateurs non privilégiés.
    

C’est une brique importante de sécurité.

---

## 9.13.2. Ce qu’il ne protège pas seul

Le user namespace ne suffit pas.

Nous devons aussi considérer :

- namespaces mount, PID, network, IPC ;
    
- cgroups ;
    
- capabilities ;
    
- seccomp ;
    
- AppArmor ou SELinux ;
    
- permissions de fichiers ;
    
- montages exposés ;
    
- vulnérabilités noyau ;
    
- configuration du runtime.
    

Un conteneur rootless avec un mauvais bind mount peut encore exposer des données sensibles.

---

## 9.13.3. Risques historiques et surface d’attaque

Les user namespaces augmentent la quantité de code noyau accessible à des utilisateurs non privilégiés.

Certaines distributions ont longtemps désactivé ou restreint les user namespaces non privilégiés pour des raisons de sécurité.

Nous devons comprendre l’équilibre :

```text
Avantage : conteneurs rootless et isolation plus fine.
Risque : surface d’attaque noyau exposée aux utilisateurs non privilégiés.
```

La bonne décision dépend du contexte :

- poste développeur ;
    
- serveur mutualisé ;
    
- cluster Kubernetes ;
    
- environnement de production sensible ;
    
- machine personnelle ;
    
- système durci.
    

---

# 9.14. Interaction avec les autres namespaces

## 9.14.1. User namespace comme socle de privilèges

Le user namespace est particulier, car il influence les droits sur les autres namespaces.

Créer certains namespaces ou effectuer certaines opérations peut être autorisé si nous avons les capabilities nécessaires dans le user namespace approprié.

Par exemple, nous pouvons combiner :

```bash
unshare --user --map-root-user --mount bash
```

Puis effectuer certains montages autorisés.

---

## 9.14.2. Avec le namespace mount

Le user namespace permet parfois à un utilisateur non privilégié de créer un namespace mount et d’effectuer certains montages limités.

Exemple :

```bash
unshare --user --map-root-user --mount bash
mount -t tmpfs tmpfs /tmp/test
```

Selon la configuration, cela peut fonctionner sans `sudo`.

---

## 9.14.3. Avec le namespace network

Créer un namespace réseau peut nécessiter des privilèges.

Mais combiné à un user namespace, certains environnements permettent :

```bash
unshare --user --map-root-user --net bash
```

Nous sommes alors root dans le user namespace et pouvons avoir certaines capabilities dans le namespace réseau créé.

Cependant, connecter ce namespace au réseau extérieur demande souvent des privilèges ou des outils spécifiques.

C’est pourquoi les conteneurs rootless utilisent des solutions particulières pour le réseau.

---

# 9.15. Expérience complète guidée

## 9.15.1. Créer un user namespace root interne

Nous lançons :

```bash
unshare --user --map-root-user bash
```

Dans le shell :

```bash
echo "Identité dans le namespace :"
id

echo "Namespace user :"
readlink /proc/$$/ns/user

echo "Mapping UID :"
cat /proc/$$/uid_map

echo "Mapping GID :"
cat /proc/$$/gid_map
```

Nous observons que nous sommes root dans le namespace.

---

## 9.15.2. Tester les limites de ce root

Toujours dans le namespace :

```bash
touch /tmp/test-userns
ls -ln /tmp/test-userns
```

Puis :

```bash
touch /root/test-userns
```

Nous devons généralement obtenir :

```text
Permission denied
```

Nous sommes root dans le namespace, mais pas root sur l’hôte.

---

## 9.15.3. Comparer depuis l’hôte

Dans un autre terminal :

```bash
ls -ln /tmp/test-userns
```

Nous observons le propriétaire réel du fichier sur l’hôte.

Si le mapping est :

```text
0 1000 1
```

le fichier créé comme root dans le namespace apparaît souvent comme appartenant à l’UID `1000` sur l’hôte.

---

## 9.15.4. Quitter

Nous quittons :

```bash
exit
```

Quand le dernier processus du namespace disparaît, le namespace user disparaît aussi, sauf s’il est maintenu par une référence.

---

# 9.16. Cas pratique : comprendre un problème de permissions rootless

## 9.16.1. Situation

Nous lançons un conteneur rootless avec un volume monté depuis l’hôte.

L’application dans le conteneur affiche :

```text
Permission denied
```

Pourtant, dans le conteneur, elle semble tourner comme root.

Pourquoi ?

---

## 9.16.2. Analyse

Nous devons vérifier :

Dans le conteneur :

```bash
id
ls -ln /data
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sur l’hôte :

```bash
ls -ln /chemin/hote
cat /etc/subuid
cat /etc/subgid
```

Nous comparons les UID/GID numériques.

---

## 9.16.3. Explication possible

Le fichier sur l’hôte appartient à :

```text
UID 1000
```

Mais dans le conteneur rootless, l’UID interne attendu correspond peut-être à :

```text
UID 100000 sur l’hôte
```

Le conteneur voit donc un propriétaire qui ne correspond pas à l’utilisateur interne attendu.

Nous devons ajuster :

- les permissions ;
    
- le propriétaire ;
    
- le mapping ;
    
- le mode de montage ;
    
- la configuration du runtime ;
    
- ou l’utilisateur utilisé dans le conteneur.
    

---

# 9.17. Pièges classiques

## 9.17.1. Croire que root dans le namespace est root sur l’hôte

C’est l’erreur principale.

Nous devons toujours distinguer :

```text
root interne
root externe
```

Le root interne a des droits dans le namespace, pas nécessairement sur tout l’hôte.

---

## 9.17.2. Raisonner avec les noms au lieu des UID numériques

Les noms comme `root`, `user`, `www-data` viennent de fichiers comme :

```text
/etc/passwd
/etc/group
```

Mais le noyau raisonne surtout avec des nombres.

Nous devons donc utiliser :

```bash
ls -ln
id -u
id -g
```

pour comprendre réellement les permissions.

---

## 9.17.3. Oublier `/etc/subuid` et `/etc/subgid`

Les conteneurs rootless ont besoin de plages subordonnées.

Si elles sont absentes ou incorrectes, nous pouvons rencontrer :

- erreurs de lancement ;
    
- mappings incomplets ;
    
- problèmes de permissions ;
    
- impossibilité d’utiliser certains utilisateurs internes.
    

---

## 9.17.4. Confondre user namespace et cgroups

Le user namespace concerne les identifiants et les privilèges.

Les cgroups concernent les ressources.

Être root dans un user namespace ne signifie pas que nous pouvons consommer CPU et mémoire sans limite.

Pour cela, nous devons regarder les cgroups.

---

## 9.17.5. Oublier les capabilities

Même si nous sommes UID 0 dans un namespace, les opérations autorisées dépendent aussi des capabilities.

Nous devons vérifier :

```bash
grep Cap /proc/$$/status
```

ou :

```bash
capsh --print
```

---

# 9.18. Exercices

## Exercice 1 — Observer le namespace user courant

Nous exécutons :

```bash
id
readlink /proc/$$/ns/user
readlink /proc/1/ns/user
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Nous répondons :

1. Quel est notre UID courant ?
    
2. Partageons-nous le namespace user du PID 1 ?
    
3. Quel mapping UID voyons-nous ?
    
4. Sommes-nous dans le namespace user initial ?
    

---

## Exercice 2 — Créer un user namespace avec root interne

Nous exécutons :

```bash
unshare --user --map-root-user bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
exit
```

Nous répondons :

1. Quel UID voyons-nous dans le namespace ?
    
2. Vers quel UID externe est-il mappé ?
    
3. Sommes-nous root sur l’hôte ?
    
4. Comment le savons-nous ?
    

---

## Exercice 3 — Créer un fichier et comparer les propriétaires

Sur l’hôte :

```bash
mkdir -p /tmp/userns-demo
```

Dans un user namespace :

```bash
unshare --user --map-root-user bash
touch /tmp/userns-demo/test.txt
ls -ln /tmp/userns-demo/test.txt
exit
```

Sur l’hôte :

```bash
ls -ln /tmp/userns-demo/test.txt
```

Nous répondons :

1. Quel propriétaire voyons-nous dans le namespace ?
    
2. Quel propriétaire voyons-nous sur l’hôte ?
    
3. Pourquoi les valeurs diffèrent-elles ?
    
4. Quel rôle joue `uid_map` ?
    

---

## Exercice 4 — Observer les capabilities

Nous exécutons sur l’hôte :

```bash
grep Cap /proc/$$/status
```

Puis dans un user namespace :

```bash
unshare --user --map-root-user bash
grep Cap /proc/$$/status
exit
```

Nous répondons :

1. Les capabilities changent-elles ?
    
2. Dans quel contexte ces capabilities sont-elles valables ?
    
3. Pourquoi UID 0 ne suffit-il pas à comprendre les droits ?
    

---

## Exercice 5 — Lire `/etc/subuid` et `/etc/subgid`

Nous exécutons :

```bash
cat /etc/subuid
cat /etc/subgid
```

Nous répondons :

1. Notre utilisateur possède-t-il une plage subordonnée ?
    
2. Quelle est la taille de cette plage ?
    
3. Pourquoi cette plage est-elle utile pour les conteneurs rootless ?
    
4. Que se passerait-il sans plage suffisante ?
    

---

## Exercice 6 — Combiner user namespace et mount namespace

Nous testons :

```bash
unshare --user --map-root-user --mount bash
mkdir -p /tmp/userns-mount-demo
mount -t tmpfs tmpfs /tmp/userns-mount-demo
findmnt /tmp/userns-mount-demo
umount /tmp/userns-mount-demo
exit
```

Nous répondons :

1. Le montage fonctionne-t-il sans `sudo` ?
    
2. Pourquoi le user namespace peut-il autoriser certaines opérations ?
    
3. Cette autorisation vaut-elle sur tout l’hôte ?
    
4. Quelles limites restent présentes ?
    

---

# 9.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace user isole les UID et GID.
    
2. Il permet d’être root dans un namespace sans être root sur l’hôte.
    
3. Le namespace user d’un processus est visible avec `/proc/<PID>/ns/user`.
    
4. Les mappings UID/GID sont visibles dans `/proc/<PID>/uid_map` et `/proc/<PID>/gid_map`.
    
5. Une ligne de mapping contient : ID interne, ID externe, taille de plage.
    
6. `unshare --user --map-root-user bash` permet de créer un user namespace simple.
    
7. Root interne ne signifie pas root externe.
    
8. Les fichiers peuvent apparaître avec des propriétaires différents selon le namespace.
    
9. Nous devons raisonner avec les UID/GID numériques.
    
10. Les capabilities sont interprétées dans le contexte d’un user namespace.
    
11. `/etc/subuid` et `/etc/subgid` définissent les plages subordonnées disponibles.
    
12. `newuidmap` et `newgidmap` servent à configurer des mappings avancés.
    
13. Les conteneurs rootless reposent fortement sur les user namespaces.
    
14. Les bind mounts peuvent créer des problèmes de permissions avec les mappings.
    
15. Le user namespace réduit certains risques, mais ne remplace pas les autres mécanismes de sécurité.
    
16. Les user namespaces augmentent aussi la surface d’attaque noyau accessible aux utilisateurs non privilégiés.
    
17. Nous devons toujours analyser le mapping, les capabilities, les montages et les cgroups ensemble.
    

---

# Conclusion du chapitre 9

Nous avons étudié le namespace user, l’un des mécanismes les plus puissants et les plus subtils de Linux.

Nous savons maintenant qu’il permet d’isoler les identifiants utilisateurs et groupes, de créer un environnement où un processus se voit comme `root`, tout en restant mappé vers un utilisateur non privilégié sur l’hôte.

Nous avons compris le rôle central des fichiers `uid_map` et `gid_map`, des plages `/etc/subuid` et `/etc/subgid`, ainsi que des outils `newuidmap` et `newgidmap`.

Nous retenons surtout que le namespace user est une brique essentielle des conteneurs rootless. Il améliore la sécurité en évitant de donner les droits root réels à un conteneur, mais il introduit aussi une complexité importante autour des permissions, des fichiers montés, des capabilities et des mappings.

Dans le chapitre suivant, nous étudions le namespace cgroup, qui isole la vue des cgroups, et nous clarifions la différence entre le namespace cgroup et les cgroups eux-mêmes.

# Chapitre 10 — Namespace cgroup

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace cgroup.

Ce namespace est souvent mal compris, parce que son nom ressemble beaucoup aux cgroups eux-mêmes. Pourtant, nous devons distinguer deux choses :

```text
Les cgroups limitent, mesurent et organisent les ressources.

Le namespace cgroup isole la vue que les processus ont de leur position dans l’arborescence des cgroups.
```

Autrement dit, le namespace cgroup ne limite pas directement la mémoire, le CPU ou les processus. Il masque ou transforme la manière dont un processus voit son rattachement aux cgroups.

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est le namespace cgroup ;
    
- distinguer cgroup namespace et cgroups ;
    
- lire `/proc/self/cgroup` ;
    
- comprendre pourquoi cette vue est importante dans les conteneurs ;
    
- faire le lien avec Docker, Podman, systemd et Kubernetes ;
    
- comprendre les limites du namespace cgroup ;
    
- diagnostiquer la position d’un processus dans les cgroups.
    

---

# 10.1. Rappel : que sont les cgroups ?

## 10.1.1. Définition générale

Les cgroups, ou _control groups_, sont un mécanisme du noyau Linux qui permet de regrouper des processus afin de :

- limiter leur consommation de ressources ;
    
- mesurer leur consommation ;
    
- organiser les processus par service, utilisateur, conteneur ou pod ;
    
- appliquer des politiques de contrôle.
    

Les cgroups peuvent contrôler ou mesurer plusieurs types de ressources :

```text
CPU
mémoire
I/O disque
nombre de processus
périphériques
pression de ressources
```

Nous utilisons les cgroups dès que nous lançons des services systemd, des conteneurs Docker, des pods Kubernetes ou des environnements avec limites de ressources.

---

## 10.1.2. Exemple simple

Un processus peut être placé dans un cgroup qui limite sa mémoire à 512 Mio.

Dans ce cas, même si la machine possède 32 Gio de RAM, le processus ne peut pas dépasser la limite imposée par son cgroup.

S’il dépasse cette limite, il peut être tué par le noyau avec un événement de type OOM.

Nous voyons donc que les cgroups répondent à une question différente des namespaces :

```text
Namespaces : que voit le processus ?
Cgroups    : que peut consommer le processus ?
```

---

# 10.2. Pourquoi un namespace cgroup ?

## 10.2.1. Le problème de la vue des cgroups

Sans namespace cgroup, un processus peut voir son chemin complet dans l’arborescence des cgroups de l’hôte.

Nous pouvons lire :

```bash
cat /proc/self/cgroup
```

Exemple possible sur l’hôte :

```text
0::/user.slice/user-1000.slice/session-3.scope
```

Dans un conteneur, sans isolation de cette vue, nous pourrions voir un chemin très révélateur :

```text
0::/system.slice/docker-8c5f2a9c4e7d9f.scope
```

ou encore :

```text
0::/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod123456.slice/cri-containerd-abcdef.scope
```

Cette information révèle l’organisation interne de l’hôte ou du cluster.

Le namespace cgroup permet de présenter une vue plus propre et plus isolée.

---

## 10.2.2. Ce que le namespace cgroup isole

Le namespace cgroup isole principalement la vue des chemins cgroups visibles dans :

```bash
/proc/<PID>/cgroup
/proc/self/cgroup
```

Dans un namespace cgroup isolé, un processus peut voir sa position comme :

```text
0::/
```

alors que, depuis l’hôte, le même processus est réellement placé plus profondément dans l’arborescence :

```text
0::/system.slice/docker-xxxx.scope
```

Le namespace cgroup ne déplace pas forcément le processus. Il change la racine apparente depuis laquelle le processus voit son rattachement.

---

## 10.2.3. Ce que le namespace cgroup ne fait pas

Le namespace cgroup ne définit pas directement les limites de ressources.

Il ne dit pas :

```text
Ce processus a droit à 512 Mio de RAM.
Ce processus a droit à 2 CPU.
Ce processus peut créer 100 processus au maximum.
```

Ces limites sont définies par les cgroups eux-mêmes, via les contrôleurs cgroup.

Le namespace cgroup répond plutôt à :

```text
À quoi ressemble l’arborescence cgroup depuis le point de vue du processus ?
```

---

# 10.3. Observer le namespace cgroup courant

## 10.3.1. Avec `/proc/<PID>/ns/cgroup`

Comme les autres namespaces, le namespace cgroup d’un processus est visible dans :

```bash
/proc/<PID>/ns/cgroup
```

Pour notre shell courant :

```bash
readlink /proc/$$/ns/cgroup
```

Exemple :

```text
cgroup:[4026531835]
```

Nous pouvons comparer avec le PID 1 :

```bash
readlink /proc/1/ns/cgroup
```

Si les valeurs sont identiques, notre shell partage le même namespace cgroup que le PID 1.

---

## 10.3.2. Comparer deux processus

Nous pouvons comparer :

```bash
echo "Shell courant : $(readlink /proc/$$/ns/cgroup)"
echo "PID 1         : $(readlink /proc/1/ns/cgroup)"
```

Même valeur :

```text
Shell courant : cgroup:[4026531835]
PID 1         : cgroup:[4026531835]
```

Les deux processus partagent le même namespace cgroup.

Valeurs différentes :

```text
Shell courant : cgroup:[4026533000]
PID 1         : cgroup:[4026531835]
```

Les deux processus ont une vue cgroup différente.

---

# 10.4. Lire `/proc/self/cgroup`

## 10.4.1. Commande de base

Nous lisons :

```bash
cat /proc/self/cgroup
```

Sur un système utilisant cgroups v2, nous pouvons voir :

```text
0::/user.slice/user-1000.slice/session-3.scope
```

Le format cgroups v2 est souvent :

```text
0::/chemin/du/cgroup
```

La partie importante est le chemin après le second `:`.

---

## 10.4.2. Format avec cgroups v1

Sur un système avec cgroups v1 ou un mode hybride, nous pouvons voir plusieurs lignes :

```text
12:memory:/user.slice
11:cpu,cpuacct:/user.slice
10:pids:/user.slice
9:devices:/user.slice
```

Le format général est :

```text
hiérarchie:contrôleurs:chemin
```

Exemple :

```text
11:cpu,cpuacct:/docker/abcdef
```

Cela signifie que, pour les contrôleurs `cpu` et `cpuacct`, le processus est dans le cgroup `/docker/abcdef`.

---

## 10.4.3. Interprétation

`/proc/self/cgroup` nous dit où le processus courant se situe dans les hiérarchies cgroup.

Mais attention :

```text
Il nous donne une vue.
Cette vue peut être modifiée par le namespace cgroup.
```

Depuis l’intérieur d’un conteneur, nous pouvons voir :

```text
0::/
```

Alors que depuis l’hôte, le même processus est dans un chemin plus détaillé.

---

# 10.5. Différence entre cgroups et namespace cgroup

## 10.5.1. Les cgroups contrôlent les ressources

Les cgroups sont actifs même sans namespace cgroup isolé.

Ils peuvent définir :

```text
memory.max
cpu.max
pids.max
io.max
```

Exemples avec cgroups v2 :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.max
```

Ces fichiers décrivent les limites et consommations du cgroup courant.

---

## 10.5.2. Le namespace cgroup contrôle la vue

Le namespace cgroup ne crée pas ces limites.

Il influence plutôt la manière dont le processus voit son chemin dans `/proc/self/cgroup`.

Nous pouvons résumer :

|Mécanisme|Rôle|
|---|---|
|cgroups|contrôlent et mesurent les ressources|
|namespace cgroup|isole la vue des chemins cgroups|
|`/proc/self/cgroup`|affiche la position du processus dans les cgroups|
|`/sys/fs/cgroup`|expose les contrôleurs et fichiers de limites|

---

## 10.5.3. Analogie

Nous pouvons faire une analogie avec le namespace mount.

Le namespace mount ne crée pas les fichiers. Il change la manière dont les montages sont visibles.

De la même façon, le namespace cgroup ne crée pas les limites. Il change la manière dont l’arborescence cgroup est visible.

---

# 10.6. cgroups v1 et cgroups v2

## 10.6.1. Pourquoi cette distinction compte ?

Pour comprendre le namespace cgroup, nous devons savoir que Linux a connu deux grandes versions de cgroups :

```text
cgroups v1
cgroups v2
```

Les deux modèles organisent les ressources différemment.

Les systèmes modernes utilisent de plus en plus cgroups v2, notamment avec systemd, Docker récent et Kubernetes récent.

---

## 10.6.2. cgroups v1

Avec cgroups v1, chaque contrôleur peut avoir sa propre hiérarchie.

Nous pouvons avoir :

```text
memory → une hiérarchie
cpu    → une autre hiérarchie
pids   → une autre hiérarchie
```

Cela rend l’analyse parfois complexe.

Dans `/proc/self/cgroup`, nous voyons souvent plusieurs lignes.

Exemple :

```text
12:memory:/docker/abcdef
11:cpu,cpuacct:/docker/abcdef
10:pids:/docker/abcdef
```

---

## 10.6.3. cgroups v2

Avec cgroups v2, nous avons une hiérarchie unifiée.

Dans `/proc/self/cgroup`, nous voyons souvent une seule ligne :

```text
0::/user.slice/user-1000.slice/session-3.scope
```

Les fichiers de contrôle se trouvent dans la hiérarchie unifiée, souvent montée sur :

```bash
/sys/fs/cgroup
```

Exemples :

```bash
cat /sys/fs/cgroup/cgroup.controllers
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/cpu.max
```

---

# 10.7. Observer les limites cgroups v2

## 10.7.1. Mémoire

Nous pouvons lire la mémoire consommée par le cgroup courant :

```bash
cat /sys/fs/cgroup/memory.current
```

La valeur est en octets.

Nous pouvons lire la limite :

```bash
cat /sys/fs/cgroup/memory.max
```

Sortie possible :

```text
max
```

Cela signifie qu’il n’y a pas de limite explicite à ce niveau.

Ou bien :

```text
536870912
```

Cela correspond à 512 Mio.

---

## 10.7.2. CPU

Nous lisons :

```bash
cat /sys/fs/cgroup/cpu.max
```

Sortie possible :

```text
max 100000
```

Cela signifie qu’il n’y a pas de quota CPU explicite.

Autre exemple :

```text
200000 100000
```

Cela signifie que le cgroup peut utiliser 200000 microsecondes de CPU par période de 100000 microsecondes, soit environ 2 CPU.

---

## 10.7.3. Nombre de processus

Nous lisons :

```bash
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Exemple :

```text
pids.current → 42
pids.max     → 512
```

Cela signifie que le cgroup contient actuellement 42 processus ou threads, avec une limite à 512.

---

# 10.8. Créer un namespace cgroup avec `unshare`

## 10.8.1. Expérience simple

Nous pouvons créer un namespace cgroup avec :

```bash
unshare --cgroup bash
```

Dans le shell :

```bash
readlink /proc/$$/ns/cgroup
cat /proc/self/cgroup
```

Nous comparons avec l’hôte dans un autre terminal :

```bash
readlink /proc/<PID_DU_SHELL>/ns/cgroup
readlink /proc/1/ns/cgroup
```

Nous devons voir que le namespace cgroup peut être différent.

---

## 10.8.2. Permissions

Selon la distribution et la configuration, cette commande peut échouer :

```bash
unshare --cgroup bash
```

Erreur possible :

```text
unshare: unshare failed: Operation not permitted
```

Nous pouvons tester avec :

```bash
sudo unshare --cgroup bash
```

Ou dans certains cas :

```bash
unshare --user --map-root-user --cgroup bash
```

Les permissions dépendent fortement du système, de la version du noyau et des règles de sécurité.

---

## 10.8.3. Ce que nous observons

Dans un nouveau namespace cgroup, le contenu de :

```bash
cat /proc/self/cgroup
```

peut devenir plus court ou présenter une racine différente.

Mais nous devons bien comprendre :

```text
Nous avons isolé la vue.
Nous n’avons pas nécessairement créé une nouvelle limite mémoire ou CPU.
```

---

# 10.9. Lien avec systemd

## 10.9.1. systemd organise les processus en cgroups

Sur beaucoup de distributions modernes, systemd organise les services et sessions dans des cgroups.

Exemples :

```text
system.slice
user.slice
machine.slice
```

Nous pouvons voir l’état d’un service :

```bash
systemctl status ssh
```

ou :

```bash
systemctl status nginx
```

systemd affiche souvent le cgroup du service.

---

## 10.9.2. `systemd-cgls`

Nous pouvons afficher l’arborescence cgroup avec :

```bash
systemd-cgls
```

Sortie conceptuelle :

```text
Control group /:
├─user.slice
│ └─user-1000.slice
│   └─session-3.scope
└─system.slice
  ├─ssh.service
  └─docker.service
```

Cette vue est très utile pour comprendre où se trouvent les processus.

---

## 10.9.3. `systemd-cgtop`

Nous pouvons voir une vue dynamique avec :

```bash
systemd-cgtop
```

Cela montre la consommation par cgroup :

- CPU ;
    
- mémoire ;
    
- tâches ;
    
- I/O selon configuration.
    

C’est une vue plus orientée services que `top`.

---

## 10.9.4. Limites systemd

systemd peut définir des limites de ressources pour un service.

Exemple dans une unité systemd :

```ini
[Service]
MemoryMax=512M
CPUQuota=200%
TasksMax=512
```

Ces paramètres sont appliqués via les cgroups.

Le namespace cgroup peut masquer une partie du chemin vu par le processus, mais les limites restent effectives.

---

# 10.10. Lien avec Docker

## 10.10.1. Docker et les cgroups

Docker utilise les cgroups pour limiter les ressources des conteneurs.

Exemples :

```bash
docker run --memory=512m image
docker run --cpus=2 image
docker run --pids-limit=256 image
```

Ces options configurent des cgroups.

Elles ne créent pas seulement des namespaces.

---

## 10.10.2. Observer depuis le conteneur

Dans un conteneur :

```bash
cat /proc/self/cgroup
```

Nous pouvons voir une vue simplifiée.

Sur cgroups v2, nous voyons parfois :

```text
0::/
```

Cela signifie que, dans son namespace cgroup, le conteneur voit son cgroup comme la racine.

Depuis l’hôte, le même processus peut être dans un chemin beaucoup plus long.

---

## 10.10.3. Observer depuis l’hôte

Nous lançons un conteneur :

```bash
docker run --rm -d --name cgroup-demo --memory=512m alpine sleep 1000
```

Nous récupérons son PID :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' cgroup-demo)
```

Nous observons :

```bash
sudo readlink /proc/$pid/ns/cgroup
readlink /proc/1/ns/cgroup

sudo cat /proc/$pid/cgroup
```

Nous pouvons comparer avec :

```bash
docker exec cgroup-demo cat /proc/self/cgroup
```

Nous voyons que la vue depuis le conteneur et la vue depuis l’hôte peuvent être différentes.

---

## 10.10.4. Nettoyage

```bash
docker rm -f cgroup-demo
```

---

# 10.11. Lien avec Kubernetes

## 10.11.1. Pods et cgroups

Kubernetes utilise les cgroups pour appliquer les `requests` et `limits`.

Exemple :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Ces valeurs se traduisent au niveau du nœud par une configuration de cgroups.

---

## 10.11.2. OOMKilled

Si un conteneur dépasse sa limite mémoire, Kubernetes peut indiquer :

```text
OOMKilled
```

Mais l’événement vient du noyau et des cgroups : le processus dépasse la limite mémoire de son cgroup.

Nous pouvons diagnostiquer avec :

```bash
kubectl describe pod <pod>
kubectl top pod
kubectl logs <pod>
```

Et, sur le nœud, avec les fichiers cgroup si nous avons accès au système.

---

## 10.11.3. Chemins cgroup Kubernetes

Depuis l’hôte, un processus de pod peut apparaître dans un chemin du type :

```text
/kubepods.slice/kubepods-burstable.slice/...
```

ou autre selon :

- runtime utilisé ;
    
- systemd ou cgroupfs ;
    
- cgroups v1 ou v2 ;
    
- distribution ;
    
- configuration du cluster.
    

Le namespace cgroup peut masquer cette complexité depuis l’intérieur du conteneur.

---

# 10.12. Namespace cgroup et `/proc`

## 10.12.1. `/proc/<PID>/cgroup`

Pour un processus donné :

```bash
cat /proc/<PID>/cgroup
```

Nous voyons sa position cgroup depuis notre perspective.

Dans certains cas, nous devons comparer :

```bash
cat /proc/<PID>/cgroup
sudo nsenter --target <PID> --cgroup cat /proc/self/cgroup
```

Cela permet de comprendre si la vue change avec le namespace cgroup.

---

## 10.12.2. `/proc/<PID>/ns/cgroup`

Nous lisons :

```bash
readlink /proc/<PID>/ns/cgroup
```

Cela nous dit dans quel namespace cgroup se trouve le processus.

Deux processus qui ont le même lien symbolique partagent la même vue cgroup.

---

## 10.12.3. `/sys/fs/cgroup`

Le répertoire :

```bash
/sys/fs/cgroup
```

expose les fichiers de contrôle cgroup.

Il dépend aussi du namespace mount et de ce qui est monté.

Dans un conteneur, la vue de `/sys/fs/cgroup` peut être :

- complète ;
    
- partielle ;
    
- en lecture seule ;
    
- masquée ;
    
- adaptée au cgroup du conteneur.
    

Nous devons donc croiser trois choses :

```text
/proc/self/cgroup        → position vue par le processus
/proc/self/ns/cgroup     → namespace cgroup
/sys/fs/cgroup           → interface de contrôle montée
```

---

# 10.13. Cas pratique : comprendre une limite mémoire

## 10.13.1. Lancement d’un conteneur limité

Nous lançons :

```bash
docker run --rm -it --memory=256m alpine sh
```

Dans le conteneur :

```sh
cat /proc/self/cgroup
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/memory.current 2>/dev/null
```

Sur cgroups v2, nous pouvons voir :

```text
memory.max     → 268435456
memory.current → valeur courante
```

Cela correspond à 256 Mio.

---

## 10.13.2. Interprétation

Même si le conteneur voit parfois une vue cgroup simplifiée, la limite est bien appliquée par le noyau.

Nous distinguons donc :

```text
Vue cgroup dans /proc/self/cgroup
Limite effective dans les fichiers cgroup
```

Le namespace cgroup peut changer la première, mais pas supprimer la seconde.

---

# 10.14. Cas pratique : comparer hôte et conteneur

## 10.14.1. Côté hôte

Nous lançons :

```bash
docker run --rm -d --name cgroup-demo --memory=512m alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' cgroup-demo)
```

Sur l’hôte :

```bash
echo "Namespace cgroup du conteneur :"
sudo readlink /proc/$pid/ns/cgroup

echo "Namespace cgroup de l'hôte :"
readlink /proc/1/ns/cgroup

echo "Vue cgroup depuis l'hôte :"
sudo cat /proc/$pid/cgroup
```

---

## 10.14.2. Côté conteneur

Nous exécutons :

```bash
docker exec cgroup-demo sh -c '
echo "Vue depuis le conteneur :"
cat /proc/self/cgroup
echo
echo "Limite mémoire éventuelle :"
cat /sys/fs/cgroup/memory.max 2>/dev/null || true
'
```

Nous comparons les résultats.

Nous pouvons observer que :

- le chemin cgroup vu depuis l’hôte est plus détaillé ;
    
- le chemin vu depuis le conteneur peut être simplifié ;
    
- la limite mémoire reste visible ou appliquée selon le montage de `/sys/fs/cgroup`.
    

---

## 10.14.3. Nettoyage

```bash
docker rm -f cgroup-demo
```

---

# 10.15. Pièges classiques

## 10.15.1. Confondre cgroup namespace et cgroups

C’est le piège principal.

Nous devons retenir :

```text
Le namespace cgroup isole la vue.
Les cgroups contrôlent les ressources.
```

Créer un namespace cgroup ne limite pas automatiquement la mémoire, le CPU ou les processus.

---

## 10.15.2. Croire que `/proc/self/cgroup` donne toujours le chemin réel de l’hôte

Dans un conteneur, `/proc/self/cgroup` peut être volontairement simplifié.

Nous ne devons pas supposer que ce chemin correspond au chemin complet de l’hôte.

Pour le voir, nous devons observer depuis l’hôte.

---

## 10.15.3. Croire que `/sys/fs/cgroup` est toujours complet

Dans un conteneur, `/sys/fs/cgroup` peut être :

- absent ;
    
- partiellement monté ;
    
- en lecture seule ;
    
- limité au cgroup du conteneur ;
    
- différent selon cgroups v1 ou v2.
    

Nous devons donc toujours vérifier :

```bash
findmnt /sys/fs/cgroup
cat /proc/self/mountinfo | grep cgroup
```

---

## 10.15.4. Confondre limites Kubernetes et namespace cgroup

Dans Kubernetes, les limites CPU/mémoire viennent des cgroups.

Le namespace cgroup peut masquer le chemin exact.

Mais si un pod est `OOMKilled`, c’est bien une limite cgroup qui est en jeu, pas le namespace cgroup lui-même.

---

# 10.16. Diagnostic cgroup

## 10.16.1. Questions à se poser

Lorsque nous diagnostiquons un problème cgroup, nous demandons :

```text
Dans quel cgroup est le processus ?
Sommes-nous dans cgroups v1 ou v2 ?
Quelle est la limite mémoire ?
Quelle est la consommation mémoire courante ?
Quelle est la limite CPU ?
Quelle est la limite pids ?
La vue est-elle celle de l’hôte ou celle du conteneur ?
Le namespace cgroup masque-t-il le chemin réel ?
/sys/fs/cgroup est-il monté correctement ?
systemd organise-t-il ce processus ?
Kubernetes impose-t-il une limite ?
```

---

## 10.16.2. Commandes utiles

Pour le namespace :

```bash
readlink /proc/<PID>/ns/cgroup
```

Pour la position cgroup :

```bash
cat /proc/<PID>/cgroup
cat /proc/self/cgroup
```

Pour cgroups v2 :

```bash
cat /sys/fs/cgroup/cgroup.controllers
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Pour systemd :

```bash
systemd-cgls
systemd-cgtop
systemctl status <service>
```

Pour Docker :

```bash
docker stats
docker inspect <container>
```

Pour Kubernetes :

```bash
kubectl top pod
kubectl describe pod
kubectl get events
```

---

# 10.17. Sécurité et namespace cgroup

## 10.17.1. Pourquoi masquer la vue cgroup ?

Masquer ou simplifier les chemins cgroups peut éviter de révéler :

- organisation de l’hôte ;
    
- noms de services ;
    
- identifiants de conteneurs ;
    
- structure Kubernetes ;
    
- chemins systemd ;
    
- informations sur le runtime.
    

Ce n’est pas une barrière de sécurité absolue, mais cela réduit l’exposition d’informations internes.

---

## 10.17.2. Écriture dans `/sys/fs/cgroup`

Permettre à un conteneur d’écrire librement dans `/sys/fs/cgroup` peut être dangereux.

Un processus pourrait tenter de :

- modifier ses limites ;
    
- déplacer des processus ;
    
- créer des sous-cgroups ;
    
- perturber l’organisation attendue.
    

C’est pourquoi `/sys/fs/cgroup` est souvent monté en lecture seule ou limité dans les conteneurs.

---

## 10.17.3. Le namespace cgroup ne remplace pas les politiques de sécurité

Nous devons toujours combiner :

```text
namespaces
cgroups
capabilities
seccomp
AppArmor ou SELinux
montages contrôlés
utilisateur non-root
root filesystem en lecture seule si possible
```

Le namespace cgroup est une couche parmi d’autres.

---

# 10.18. Exercices

## Exercice 1 — Observer le namespace cgroup courant

Nous exécutons :

```bash
readlink /proc/$$/ns/cgroup
readlink /proc/1/ns/cgroup
cat /proc/self/cgroup
```

Nous répondons :

1. Le shell courant partage-t-il le namespace cgroup du PID 1 ?
    
2. Que contient `/proc/self/cgroup` ?
    
3. Sommes-nous probablement dans le namespace cgroup initial ?
    

---

## Exercice 2 — Identifier cgroups v1 ou v2

Nous exécutons :

```bash
findmnt /sys/fs/cgroup
cat /proc/filesystems | grep cgroup
cat /sys/fs/cgroup/cgroup.controllers 2>/dev/null
```

Nous répondons :

1. Le système utilise-t-il cgroups v2 ?
    
2. `/sys/fs/cgroup` est-il une hiérarchie unifiée ?
    
3. Quels contrôleurs sont disponibles ?
    

---

## Exercice 3 — Lire les limites cgroups v2

Si cgroups v2 est disponible :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Nous répondons :

1. Quelle mémoire est consommée ?
    
2. Existe-t-il une limite mémoire ?
    
3. Quelle est la limite CPU ?
    
4. Quelle est la limite de processus ?
    

---

## Exercice 4 — Créer un namespace cgroup

Nous exécutons :

```bash
unshare --cgroup bash
readlink /proc/$$/ns/cgroup
cat /proc/self/cgroup
exit
```

Si la commande échoue, nous essayons en environnement de test :

```bash
sudo unshare --cgroup bash
```

Nous répondons :

1. L’identifiant du namespace cgroup a-t-il changé ?
    
2. Le contenu de `/proc/self/cgroup` a-t-il changé ?
    
3. Avons-nous créé une limite mémoire ou CPU ?
    
4. Pourquoi ?
    

---

## Exercice 5 — Observer un service systemd

Nous choisissons un service :

```bash
systemctl status ssh
```

ou :

```bash
systemctl status nginx
```

Puis :

```bash
systemd-cgls
systemd-cgtop
```

Nous répondons :

1. Dans quel cgroup le service apparaît-il ?
    
2. systemd organise-t-il les processus par service ?
    
3. Quelle différence faisons-nous entre service systemd et processus Linux ?
    

---

## Exercice 6 — Observer un conteneur Docker

Si Docker est disponible :

```bash
docker run --rm -d --name cgroup-demo --memory=512m alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' cgroup-demo)

sudo readlink /proc/$pid/ns/cgroup
readlink /proc/1/ns/cgroup

sudo cat /proc/$pid/cgroup
docker exec cgroup-demo cat /proc/self/cgroup
docker exec cgroup-demo sh -c 'cat /sys/fs/cgroup/memory.max 2>/dev/null || true'

docker rm -f cgroup-demo
```

Nous répondons :

1. Le conteneur partage-t-il le namespace cgroup de l’hôte ?
    
2. La vue de `/proc/self/cgroup` est-elle identique dedans et dehors ?
    
3. La limite mémoire est-elle visible ?
    
4. Quelle différence faisons-nous entre vue et limite ?
    

---

## Exercice 7 — Analyse Kubernetes

Nous étudions cette configuration :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Nous répondons :

1. Quelle limite mémoire est demandée ?
    
2. Quelle limite CPU est demandée ?
    
3. Quel mécanisme Linux applique ces limites ?
    
4. Quel rôle joue éventuellement le namespace cgroup ?
    
5. Que signifie un pod `OOMKilled` ?
    

---

# 10.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. Les cgroups contrôlent et mesurent les ressources.
    
2. Le namespace cgroup isole la vue des chemins cgroups.
    
3. Le namespace cgroup ne limite pas directement CPU, mémoire ou pids.
    
4. `/proc/<PID>/ns/cgroup` identifie le namespace cgroup d’un processus.
    
5. `/proc/self/cgroup` affiche la position cgroup vue par le processus.
    
6. Avec cgroups v2, nous avons souvent une hiérarchie unifiée.
    
7. Avec cgroups v1, nous pouvons avoir plusieurs hiérarchies par contrôleur.
    
8. `/sys/fs/cgroup` expose les fichiers de contrôle et de mesure.
    
9. systemd organise les services dans des cgroups.
    
10. Docker utilise les cgroups pour appliquer `--memory`, `--cpus` et `--pids-limit`.
    
11. Kubernetes traduit `requests` et `limits` en configuration cgroup sur les nœuds.
    
12. `OOMKilled` est lié au dépassement d’une limite mémoire cgroup.
    
13. Le namespace cgroup peut masquer le chemin réel de l’hôte.
    
14. Dans un conteneur, `/proc/self/cgroup` et `/sys/fs/cgroup` peuvent présenter une vue simplifiée ou limitée.
    
15. Nous devons toujours distinguer vue cgroup, limites effectives et montage de `/sys/fs/cgroup`.
    
16. Le namespace cgroup est utile pour l’isolation d’information, mais il ne remplace pas les cgroups eux-mêmes.
    

---

# Conclusion du chapitre 10

Nous avons étudié le namespace cgroup et clarifié sa relation avec les cgroups.

Nous savons maintenant que les cgroups sont le mécanisme qui contrôle et mesure les ressources, tandis que le namespace cgroup isole la vue qu’un processus a de son rattachement dans l’arborescence cgroup.

Cette distinction est essentielle. Quand un conteneur est limité à 512 Mio de RAM, ce n’est pas le namespace cgroup qui impose cette limite : ce sont les cgroups. Le namespace cgroup peut seulement faire apparaître le chemin du conteneur comme une racine plus propre ou plus isolée dans `/proc/self/cgroup`.

Nous avons également relié ce mécanisme à systemd, Docker et Kubernetes. Nous comprenons désormais pourquoi un service systemd, un conteneur Docker ou un pod Kubernetes apparaissent dans des cgroups, et pourquoi la vue observée depuis l’intérieur d’un conteneur peut différer de celle observée depuis l’hôte.

Dans le chapitre suivant, nous étudions le namespace time, qui permet d’isoler certains offsets d’horloges comme `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`.

# Chapitre 11 — Namespace time

## Objectifs du chapitre

Dans ce chapitre, nous étudions le namespace time.

Le namespace time est plus récent et plus spécialisé que les namespaces UTS, PID, mount, network, IPC ou user. Il ne sert pas à donner une pile réseau ou une table de processus séparée. Il permet d’isoler certains décalages temporels vus par les processus.

Nous devons toutefois faire attention à une idée importante :

```text
Le namespace time ne permet pas simplement de changer l’heure globale du système.
```

Il concerne surtout certaines horloges monotones utilisées par les programmes pour mesurer des durées, des délais ou le temps écoulé depuis le démarrage.

À la fin de ce chapitre, nous savons :

- expliquer le rôle du namespace time ;
    
- distinguer heure réelle et horloges monotones ;
    
- comprendre les horloges `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME` ;
    
- observer le namespace time avec `/proc/<PID>/ns/time` ;
    
- lire `/proc/self/timens_offsets` ;
    
- comprendre les cas d’usage : tests, checkpoint/restore, conteneurs avancés ;
    
- identifier les limites du namespace time.
    

---

# 11.1. Pourquoi isoler le temps ?

## 11.1.1. Le temps comme dépendance système

Les programmes utilisent le temps en permanence.

Ils l’utilisent pour :

- mesurer une durée ;
    
- déclencher un timeout ;
    
- planifier une tâche ;
    
- calculer un délai d’expiration ;
    
- horodater un événement ;
    
- mesurer un temps de démarrage ;
    
- vérifier une période de validité ;
    
- gérer des caches ;
    
- calculer une latence.
    

Exemples :

```text
attendre 5 secondes
expirer une session après 30 minutes
mesurer une latence réseau
déterminer depuis combien de temps le système tourne
relancer un service après un délai
```

Dans un environnement isolé, il peut être utile qu’un groupe de processus voie certains temps de manière différente.

---

## 11.1.2. Exemple de problème

Imaginons que nous voulions restaurer un processus qui a été figé puis relancé plus tard.

Le processus peut avoir enregistré des timers internes basés sur le temps monotone.

Si, lors de la restauration, le temps monotone vu par le processus a trop avancé, il peut croire que :

- tous ses timeouts ont expiré ;
    
- ses connexions sont trop anciennes ;
    
- ses tâches programmées sont en retard ;
    
- ses mesures internes sont incohérentes.
    

Le namespace time aide certains outils, notamment de checkpoint/restore, à fournir une vue temporelle plus cohérente.

---

# 11.2. Les différents types de temps sous Linux

## 11.2.1. L’heure réelle

L’heure réelle correspond à l’heure du calendrier.

Nous pouvons l’afficher avec :

```bash
date
```

Exemple :

```text
dim. 24 mai 2026 22:35:12 CEST
```

Cette heure peut changer si :

- nous la réglons manuellement ;
    
- NTP la corrige ;
    
- la machine change de fuseau horaire ;
    
- l’horloge matérielle est ajustée.
    

Cette heure correspond à ce que les programmes utilisent souvent pour les dates humaines, les logs ou les certificats.

Le namespace time ne sert pas principalement à isoler cette heure réelle.

---

## 11.2.2. Le temps monotone

Le temps monotone est une horloge qui ne recule pas.

Elle sert à mesurer des durées.

Par exemple, un programme peut dire :

```text
Je démarre un chronomètre maintenant.
Je relis l’horloge plus tard.
La différence me donne une durée.
```

Cette horloge est importante parce qu’elle n’est pas affectée de la même manière que l’heure réelle par les changements de date ou les corrections NTP.

Sous Linux, une horloge importante est :

```text
CLOCK_MONOTONIC
```

---

## 11.2.3. Le temps depuis le démarrage

Une autre notion importante est le temps depuis le démarrage du système.

Nous pouvons lire :

```bash
cat /proc/uptime
```

Exemple :

```text
12345.67 89012.34
```

Le premier nombre indique le temps écoulé depuis le démarrage, en secondes.

Cette notion est proche des horloges monotones, mais plusieurs horloges existent avec des détails différents.

---

## 11.2.4. `CLOCK_BOOTTIME`

`CLOCK_BOOTTIME` ressemble à `CLOCK_MONOTONIC`, mais elle inclut aussi le temps passé en suspension.

Elle est utile pour certains timers qui doivent tenir compte du temps réel écoulé même si la machine a été suspendue.

Nous retenons :

```text
CLOCK_MONOTONIC mesure un temps monotone.
CLOCK_BOOTTIME inclut aussi le temps passé en suspension.
```

Le namespace time permet d’appliquer des offsets à certaines de ces horloges.

---

# 11.3. Qu’est-ce que le namespace time ?

## 11.3.1. Définition

Le namespace time permet d’isoler des offsets appliqués à certaines horloges vues par les processus.

Il concerne principalement :

```text
CLOCK_MONOTONIC
CLOCK_BOOTTIME
```

Cela signifie qu’un processus dans un namespace time peut voir une valeur monotone décalée par rapport à celle de l’hôte.

---

## 11.3.2. Ce qu’il ne fait pas

Le namespace time ne permet pas simplement de faire :

```text
Dans ce conteneur, nous sommes le 1er janvier 2030.
Sur l’hôte, nous sommes le 24 mai 2026.
```

Pour l’heure réelle, les choses sont plus complexes et ne sont pas l’objectif principal du namespace time.

Nous ne devons donc pas le confondre avec :

- le fuseau horaire ;
    
- la commande `date`;
    
- le réglage NTP ;
    
- l’horloge matérielle ;
    
- les variables `TZ`;
    
- les bibliothèques qui simulent le temps.
    

---

## 11.3.3. Pourquoi parler d’offset ?

Le namespace time ne crée pas une horloge entièrement indépendante.

Il applique un décalage, c’est-à-dire un offset, à certaines horloges.

Nous pouvons représenter cela ainsi :

```text
temps monotone vu dans le namespace
=
temps monotone de l’hôte
+
offset du namespace
```

L’offset peut être différent selon le namespace time.

---

# 11.4. Observer le namespace time courant

## 11.4.1. Avec `/proc/<PID>/ns/time`

Comme les autres namespaces, le namespace time d’un processus est visible dans :

```bash
/proc/<PID>/ns/time
```

Pour notre shell courant :

```bash
readlink /proc/$$/ns/time
```

Exemple :

```text
time:[4026531834]
```

Nous comparons avec le PID 1 :

```bash
readlink /proc/1/ns/time
```

Si les deux valeurs sont identiques, notre shell partage le même namespace time que le PID 1.

---

## 11.4.2. Namespace time pour les enfants

Sur les noyaux récents, nous pouvons aussi voir :

```bash
ls -l /proc/$$/ns/time_for_children
```

Exemple :

```text
time_for_children -> time:[4026531834]
```

Cette entrée indique le namespace time qui sera utilisé pour les processus enfants.

Ce détail est important parce que la création et la configuration du namespace time impliquent souvent les futurs enfants du processus.

---

# 11.5. Lire `/proc/self/timens_offsets`

## 11.5.1. Le fichier `timens_offsets`

Le fichier principal pour observer les offsets du namespace time est :

```bash
/proc/self/timens_offsets
```

Nous pouvons lire :

```bash
cat /proc/self/timens_offsets
```

Sortie possible :

```text
monotonic           0         0
boottime            0         0
```

Les lignes correspondent aux horloges concernées.

---

## 11.5.2. Interprétation

Une ligne ressemble à :

```text
monotonic           0         0
```

Elle indique un offset pour l’horloge monotone.

Les colonnes numériques correspondent au décalage en secondes et nanosecondes.

Par exemple, conceptuellement :

```text
monotonic           3600      0
```

signifierait que l’horloge monotone est décalée d’une heure pour les processus du namespace concerné.

---

## 11.5.3. Lire les offsets du processus courant

Nous exécutons :

```bash
echo "Namespace time : $(readlink /proc/$$/ns/time)"
cat /proc/self/timens_offsets
```

Nous obtenons généralement des offsets nuls dans un shell normal.

---

# 11.6. Créer un namespace time

## 11.6.1. Avec `unshare --time`

Nous pouvons tenter :

```bash
unshare --time bash
```

Dans le shell :

```bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
```

Selon le noyau, les permissions et la version de `util-linux`, cette commande peut fonctionner ou échouer.

Erreur possible :

```text
unshare: unshare failed: Operation not permitted
```

Dans ce cas, nous devons tester dans un environnement adapté ou avec des privilèges :

```bash
sudo unshare --time bash
```

---

## 11.6.2. Avec user namespace

Sur certaines configurations, nous pouvons combiner avec un user namespace :

```bash
unshare --user --map-root-user --time bash
```

Puis :

```bash
id
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
```

Le comportement dépend fortement du noyau et de la configuration de sécurité.

Nous devons donc être plus prudents qu’avec les namespaces UTS ou PID.

---

## 11.6.3. Pourquoi c’est moins visible ?

Changer le hostname dans un namespace UTS est immédiatement visible.

Créer un namespace PID est immédiatement visible avec `ps`.

Créer un namespace réseau est visible avec `ip addr`.

Le namespace time est plus subtil, car il ne change pas forcément une commande simple comme :

```bash
date
```

Son effet concerne certaines horloges utilisées par les programmes, pas forcément l’heure affichée à l’utilisateur.

---

# 11.7. Modifier les offsets temporels

## 11.7.1. Écriture dans `timens_offsets`

Dans certains cas, les offsets peuvent être configurés en écrivant dans :

```bash
/proc/<PID>/timens_offsets
```

ou :

```bash
/proc/self/timens_offsets
```

Exemple conceptuel :

```bash
echo "monotonic 3600 0" > /proc/self/timens_offsets
```

ou :

```bash
echo "boottime 3600 0" > /proc/self/timens_offsets
```

Cela demande les permissions adéquates et doit être fait au bon moment.

---

## 11.7.2. Attention au moment de configuration

Le namespace time a une contrainte importante : les offsets doivent généralement être configurés avant que le processus cible ou ses enfants n’utilisent réellement ce namespace de manière définitive.

En pratique, les outils spécialisés gèrent ce détail.

Pour un cours de niveau Master 2, nous devons surtout comprendre le principe :

```text
Le namespace time applique des offsets à certaines horloges.
La configuration est plus délicate que pour un simple hostname.
```

---

## 11.7.3. Pourquoi nous ne faisons pas toujours la manipulation à la main

Selon le système, manipuler manuellement `timens_offsets` peut être difficile pour plusieurs raisons :

- noyau trop ancien ;
    
- commande `unshare` sans support complet ;
    
- permissions insuffisantes ;
    
- restrictions de sécurité ;
    
- moment d’écriture incorrect ;
    
- absence de besoin réel dans les environnements courants.
    

Nous étudions donc surtout le mécanisme et ses cas d’usage.

---

# 11.8. Mesurer les horloges en pratique

## 11.8.1. Avec Python

Nous pouvons observer plusieurs notions temporelles avec Python :

```bash
python3 - <<'PY'
import time

print("time.time()        :", time.time())
print("time.monotonic()   :", time.monotonic())
print("time.perf_counter():", time.perf_counter())
PY
```

Interprétation :

- `time.time()` correspond à l’heure réelle Unix ;
    
- `time.monotonic()` utilise une horloge monotone ;
    
- `time.perf_counter()` sert à mesurer des performances et durées.
    

Le namespace time vise surtout les horloges monotones et boottime, pas simplement `time.time()`.

---

## 11.8.2. Avec `/proc/uptime`

Nous lisons :

```bash
cat /proc/uptime
```

Puis :

```bash
sleep 2
cat /proc/uptime
```

Nous voyons que le premier nombre augmente.

Dans certains contextes avec namespace time, les valeurs liées au temps depuis le démarrage peuvent être influencées par les offsets.

---

## 11.8.3. Comparaison hôte et namespace

Nous pouvons comparer :

Sur l’hôte :

```bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
cat /proc/uptime
```

Dans un namespace time :

```bash
unshare --time bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
cat /proc/uptime
```

Selon les offsets configurés, les valeurs peuvent être identiques ou décalées.

---

# 11.9. Cas d’usage : checkpoint/restore

## 11.9.1. Qu’est-ce que checkpoint/restore ?

Le checkpoint/restore consiste à figer l’état d’un processus, puis à le restaurer plus tard.

Un outil connu dans cet écosystème est CRIU, _Checkpoint/Restore In Userspace_.

L’objectif est de pouvoir :

- migrer un processus ;
    
- sauvegarder son état ;
    
- restaurer un conteneur ;
    
- déplacer une charge ;
    
- suspendre puis reprendre une application.
    

---

## 11.9.2. Pourquoi le temps pose problème ?

Un processus peut contenir des timers internes.

Exemple :

```text
timer A expire dans 10 secondes
connexion B expire dans 30 secondes
cache C expire dans 5 minutes
```

Si nous figeons le processus et le restaurons longtemps après, le temps monotone vu par le processus peut ne plus correspondre à ce qu’il attend.

Le namespace time permet d’ajuster la vue de certaines horloges pour rendre la restauration plus cohérente.

---

## 11.9.3. Conteneurs migrables

Dans des scénarios avancés, nous pouvons vouloir migrer un conteneur d’un hôte à un autre.

Pour cela, il faut reconstruire une vue cohérente de plusieurs ressources :

- processus ;
    
- fichiers ;
    
- réseau ;
    
- mémoire ;
    
- cgroups ;
    
- temps.
    

Le namespace time ajoute une brique utile à cette cohérence.

---

# 11.10. Cas d’usage : tests

## 11.10.1. Tester du code dépendant du temps

Beaucoup de logiciels dépendent du temps.

Exemples :

- expiration de session ;
    
- retry avec backoff ;
    
- timeout réseau ;
    
- expiration de cache ;
    
- calcul de durée ;
    
- ordonnanceur de tâches ;
    
- certificats ;
    
- tokens ;
    
- licences.
    

Pour tester ces comportements, nous avons parfois besoin de simuler un temps différent.

---

## 11.10.2. Limite du namespace time pour les tests applicatifs

Le namespace time ne remplace pas toujours les bibliothèques de test temporel.

Par exemple, si nous voulons tester :

```text
Nous sommes le 1er janvier 2030.
```

le namespace time n’est pas forcément l’outil adapté.

Nous pouvons utiliser :

- injection d’horloge dans le code ;
    
- bibliothèques de mock du temps ;
    
- variables de configuration ;
    
- simulateurs ;
    
- outils comme `libfaketime` dans certains cas.
    

Le namespace time est plus bas niveau et concerne surtout certaines horloges noyau.

---

# 11.11. Cas d’usage : conteneurs avancés

## 11.11.1. Pourquoi les conteneurs peuvent en bénéficier

Les conteneurs classiques n’ont pas toujours besoin du namespace time.

Mais dans des contextes avancés, il peut être utile pour :

- migration de conteneurs ;
    
- tests système ;
    
- environnements de simulation ;
    
- outils de checkpoint/restore ;
    
- workloads sensibles aux durées ;
    
- isolation temporelle partielle.
    

---

## 11.11.2. Pourquoi ce n’est pas aussi courant que les autres namespaces

Les namespaces suivants sont très courants dans les conteneurs :

```text
mnt
pid
net
uts
ipc
user
```

Le namespace time est plus spécialisé.

Beaucoup d’utilisateurs de Docker ou Kubernetes n’y pensent jamais directement.

Il reste néanmoins important pour comprendre l’évolution des mécanismes d’isolation Linux.

---

# 11.12. Lien avec Docker, Podman et Kubernetes

## 11.12.1. Docker et Podman

Les runtimes de conteneurs peuvent utiliser le namespace time si le noyau et la configuration le permettent.

Mais dans la plupart des usages classiques, nous rencontrons moins directement ce namespace que les namespaces PID, mount ou network.

Pour observer un conteneur :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' <container>)
sudo readlink /proc/$pid/ns/time
readlink /proc/1/ns/time
```

Si les valeurs diffèrent, le conteneur utilise un namespace time distinct.

---

## 11.12.2. Kubernetes

Dans Kubernetes, le namespace time n’est pas généralement le premier mécanisme que nous configurons.

Les paramètres les plus visibles sont plutôt :

- `hostNetwork`;
    
- `hostPID`;
    
- `hostIPC`;
    
- `securityContext`;
    
- cgroups via requests/limits ;
    
- volumes ;
    
- capabilities ;
    
- seccomp.
    

Le namespace time peut intervenir dans des scénarios runtime plus avancés, mais il n’est pas aussi exposé dans la configuration Kubernetes courante.

---

# 11.13. Sécurité du namespace time

## 11.13.1. Pourquoi isoler le temps peut être sensible

Le temps influence beaucoup de comportements :

- expiration de tokens ;
    
- délais de sécurité ;
    
- logs ;
    
- timeouts ;
    
- mécanismes anti-rejeu ;
    
- supervision ;
    
- mesure de performance ;
    
- ordonnancement.
    

Une vue temporelle incorrecte peut perturber une application.

---

## 11.13.2. Le namespace time n’est pas une barrière de sécurité suffisante

Comme les autres namespaces, il ne suffit pas seul.

Nous devons le combiner avec :

- user namespace ;
    
- mount namespace ;
    
- cgroups ;
    
- capabilities ;
    
- seccomp ;
    
- AppArmor ou SELinux ;
    
- configuration correcte du runtime.
    

---

## 11.13.3. Attention aux logs et à l’observabilité

Si différents environnements voient des temps différents, les logs peuvent devenir plus difficiles à corréler.

Exemple :

```text
Hôte : événement à 10:00
Processus isolé : durée monotone décalée
Outil de monitoring : métrique interprétée différemment
```

Nous devons savoir quelles horloges sont utilisées par nos outils.

---

# 11.14. Différence avec le fuseau horaire

## 11.14.1. Fuseau horaire

Le fuseau horaire est souvent contrôlé par :

```text
/etc/localtime
variable TZ
bibliothèques libc
configuration applicative
```

Exemple :

```bash
TZ=UTC date
TZ=Europe/Paris date
```

Cela change l’affichage local de l’heure, pas le namespace time.

---

## 11.14.2. Ne pas confondre

Nous distinguons :

|Mécanisme|Rôle|
|---|---|
|Fuseau horaire|change l’affichage local de l’heure|
|`date` système|affiche ou règle l’heure réelle|
|NTP|synchronise l’heure réelle|
|namespace time|applique des offsets à certaines horloges monotonic/boottime|
|mock applicatif|simule le temps dans le code|

Cette distinction est essentielle pour éviter de choisir le mauvais outil.

---

# 11.15. Diagnostic autour du namespace time

## 11.15.1. Questions à se poser

Lorsque nous analysons un problème lié au temps, nous demandons :

```text
L’application utilise-t-elle l’heure réelle ou une horloge monotone ?
Le problème concerne-t-il une date humaine ou une durée ?
Le processus est-il dans un namespace time différent ?
Quels offsets sont configurés ?
Le fuseau horaire est-il en cause ?
NTP corrige-t-il l’heure ?
Sommes-nous dans un conteneur ?
L’application utilise-t-elle des mocks ou une bibliothèque temporelle ?
```

---

## 11.15.2. Commandes utiles

Observer le namespace time :

```bash
readlink /proc/$$/ns/time
readlink /proc/1/ns/time
```

Observer les offsets :

```bash
cat /proc/self/timens_offsets
```

Observer l’heure réelle :

```bash
date
timedatectl
```

Observer l’uptime :

```bash
cat /proc/uptime
uptime
```

Tester différentes horloges en Python :

```bash
python3 - <<'PY'
import time
print("time.time()     =", time.time())
print("time.monotonic()=", time.monotonic())
PY
```

---

# 11.16. Expérience guidée

## 11.16.1. Observer le namespace time actuel

Nous exécutons :

```bash
echo "Namespace time courant :"
readlink /proc/$$/ns/time

echo "Namespace time du PID 1 :"
readlink /proc/1/ns/time

echo "Offsets temporels :"
cat /proc/self/timens_offsets
```

Nous répondons :

```text
Partageons-nous le même namespace time que le PID 1 ?
Les offsets sont-ils nuls ?
```

---

## 11.16.2. Créer un namespace time

Nous testons :

```bash
unshare --time bash
```

Dans le shell :

```bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
date
cat /proc/uptime
```

Puis nous quittons :

```bash
exit
```

Si la commande échoue, nous notons l’erreur et nous testons éventuellement dans un environnement autorisé :

```bash
sudo unshare --time bash
```

---

## 11.16.3. Comparer avec Python

Dans le shell normal :

```bash
python3 - <<'PY'
import time
print("time.time()     =", time.time())
print("time.monotonic()=", time.monotonic())
PY
```

Dans le namespace time :

```bash
unshare --time bash
python3 - <<'PY'
import time
print("time.time()     =", time.time())
print("time.monotonic()=", time.monotonic())
PY
exit
```

Si aucun offset n’est configuré, les différences ne seront pas spectaculaires.

L’intérêt est surtout de comprendre quelles horloges existent et lesquelles peuvent être concernées.

---

# 11.17. Limites du namespace time

## 11.17.1. Il est moins général qu’on l’imagine

Le namespace time ne permet pas de tout simuler.

Il ne remplace pas :

- une horloge système indépendante ;
    
- une machine virtuelle ;
    
- un mock applicatif ;
    
- une configuration de fuseau horaire ;
    
- NTP ;
    
- une simulation complète du calendrier.
    

Il concerne surtout certains offsets d’horloges noyau.

---

## 11.17.2. Il dépend du noyau

Le namespace time n’est disponible que sur des noyaux suffisamment récents.

Sur des systèmes anciens, il peut être absent.

Nous vérifions :

```bash
ls -l /proc/$$/ns/time
```

Si ce fichier n’existe pas, le système ne supporte probablement pas le namespace time.

---

## 11.17.3. Il dépend des permissions

Même si le noyau le supporte, la création ou la modification des offsets peut être interdite.

Nous pouvons obtenir :

```text
Operation not permitted
```

C’est normal sur des systèmes durcis ou non configurés pour cela.

---

## 11.17.4. Il est peu utilisé dans les usages courants

Un développeur Docker classique manipule souvent :

- mount ;
    
- PID ;
    
- network ;
    
- UTS ;
    
- IPC ;
    
- user.
    

Il manipule beaucoup plus rarement le namespace time.

Cela ne le rend pas inutile, mais il reste plus spécialisé.

---

# 11.18. Pièges classiques

## 11.18.1. Croire que `date` change

Créer un namespace time ne signifie pas nécessairement que :

```bash
date
```

affiche une date différente.

Pour modifier l’affichage de `date`, nous parlons plutôt :

- heure réelle ;
    
- fuseau horaire ;
    
- variable `TZ`;
    
- configuration système ;
    
- mocks applicatifs.
    

---

## 11.18.2. Confondre durée et date

Le namespace time est plus pertinent pour des durées et des horloges monotones que pour des dates humaines.

Nous devons distinguer :

```text
date humaine : 24 mai 2026
durée : 12,5 secondes
temps monotone : compteur qui ne recule pas
```

---

## 11.18.3. Ignorer l’impact sur les logs

Si une application mélange plusieurs sources de temps, les logs et mesures peuvent devenir difficiles à interpréter.

Nous devons toujours savoir quelle horloge est utilisée.

---

# 11.19. Exercices

## Exercice 1 — Observer le namespace time

Nous exécutons :

```bash
readlink /proc/$$/ns/time
readlink /proc/1/ns/time
cat /proc/self/timens_offsets
```

Nous répondons :

1. Le shell partage-t-il le namespace time du PID 1 ?
    
2. Quels offsets voyons-nous ?
    
3. Les offsets sont-ils nuls ?
    
4. Que signifient les lignes `monotonic` et `boottime` ?
    

---

## Exercice 2 — Comparer date et temps monotone

Nous exécutons :

```bash
date
cat /proc/uptime
python3 - <<'PY'
import time
print("time.time()     =", time.time())
print("time.monotonic()=", time.monotonic())
PY
```

Nous répondons :

1. Quelle commande affiche l’heure réelle ?
    
2. Quelle valeur représente une durée depuis le démarrage ?
    
3. Quelle fonction Python utilise une horloge monotone ?
    
4. Pourquoi utilisons-nous une horloge monotone pour mesurer une durée ?
    

---

## Exercice 3 — Créer un namespace time

Nous testons :

```bash
unshare --time bash
readlink /proc/$$/ns/time
cat /proc/self/timens_offsets
exit
```

Si cela échoue :

```bash
sudo unshare --time bash
```

Nous répondons :

1. Le namespace time a-t-il changé ?
    
2. La commande demande-t-elle des privilèges ?
    
3. Les offsets sont-ils modifiés par défaut ?
    
4. Pourquoi l’effet est-il moins visible qu’avec le namespace UTS ?
    

---

## Exercice 4 — Distinguer fuseau horaire et namespace time

Nous exécutons :

```bash
date
TZ=UTC date
TZ=Europe/Paris date
readlink /proc/$$/ns/time
```

Nous répondons :

1. Que change la variable `TZ` ?
    
2. Change-t-elle le namespace time ?
    
3. Change-t-elle l’heure réelle du système ?
    
4. Pourquoi faut-il distinguer fuseau horaire et namespace time ?
    

---

## Exercice 5 — Réflexion checkpoint/restore

Nous considérons un processus figé puis restauré deux heures plus tard.

Nous répondons :

1. Quels timers internes peuvent être perturbés ?
    
2. Pourquoi une horloge monotone peut poser problème ?
    
3. Comment un namespace time peut-il aider ?
    
4. Pourquoi cela concerne-t-il surtout des usages avancés ?
    

---

# 11.20. Ce que nous devons retenir

Nous retenons les points suivants :

1. Le namespace time isole des offsets appliqués à certaines horloges.
    
2. Il concerne principalement `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`.
    
3. Il ne permet pas simplement de changer l’heure réelle du système.
    
4. L’heure réelle s’observe avec `date` et dépend d’autres mécanismes.
    
5. Le fuseau horaire est différent du namespace time.
    
6. Le namespace time d’un processus est visible avec `/proc/<PID>/ns/time`.
    
7. Les offsets sont visibles avec `/proc/self/timens_offsets`.
    
8. Les offsets sont exprimés en secondes et nanosecondes.
    
9. La manipulation du namespace time dépend du noyau et des permissions.
    
10. Le namespace time est particulièrement utile pour checkpoint/restore.
    
11. Il peut aussi servir à certains tests système avancés.
    
12. Il est moins utilisé dans les conteneurs courants que les namespaces PID, mount ou network.
    
13. Pour tester des dates applicatives, un mock d’horloge est souvent plus adapté.
    
14. Nous devons toujours distinguer date, durée, uptime, horloge monotone et fuseau horaire.
    
15. Le namespace time est une brique spécialisée d’isolation, pas une virtualisation complète du temps.
    

---

# Conclusion du chapitre 11

Nous avons étudié le namespace time, un namespace plus spécialisé que ceux vus précédemment.

Nous savons maintenant qu’il ne sert pas simplement à changer la date visible avec `date`, mais à appliquer des offsets à certaines horloges utilisées pour mesurer des durées, notamment `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`.

Nous avons compris pourquoi ce mécanisme est utile pour des scénarios avancés comme le checkpoint/restore, la migration de conteneurs ou certains tests système. Nous avons aussi vu ses limites : il dépend du noyau, des permissions, et ne remplace pas les mécanismes applicatifs de simulation du temps.

Dans le chapitre suivant, nous faisons le lien entre les namespaces et `/proc`, afin de comprendre comment `/proc/<PID>/ns`, `/proc/net`, `/proc/mounts`, `/proc/self/cgroup`, `uid_map` et `gid_map` reflètent les différentes vues isolées du système.

# Chapitre 12 — Namespaces et `/proc`

## Objectifs du chapitre

Dans ce chapitre, nous faisons le lien entre deux notions centrales de Linux :

```text
les namespaces
/proc
```

Nous avons déjà vu que les namespaces isolent différentes vues du système : processus, réseau, montages, hostname, IPC, utilisateurs, cgroups et temps.

Nous avons aussi étudié `/proc` comme interface d’observation du noyau.

Nous allons maintenant comprendre que `/proc` est l’un des meilleurs moyens d’observer les namespaces, mais aussi que son contenu dépend souvent du namespace depuis lequel nous le lisons.

À la fin de ce chapitre, nous savons :

- observer les namespaces d’un processus avec `/proc/<PID>/ns` ;
    
- comparer les namespaces de deux processus ;
    
- comprendre pourquoi `/proc` change selon le namespace courant ;
    
- relier le namespace PID à `/proc/<PID>` ;
    
- relier le namespace network à `/proc/net` ;
    
- relier le namespace mount à `/proc/mounts` et `/proc/self/mountinfo` ;
    
- relier le namespace user à `uid_map` et `gid_map` ;
    
- relier le namespace cgroup à `/proc/self/cgroup` ;
    
- relier le namespace time à `/proc/self/timens_offsets` ;
    
- comprendre les implications de sécurité de `/proc`.
    

---

# 12.1. `/proc` comme interface d’observation des namespaces

## 12.1.1. Rappel sur `/proc`

`/proc` est un pseudo-système de fichiers.

Il ne contient pas des fichiers classiques stockés sur disque. Il expose des informations produites dynamiquement par le noyau.

Exemples :

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/loadavg
cat /proc/self/status
```

Dans le contexte des namespaces, `/proc` est particulièrement important, car il permet d’observer :

- les namespaces auxquels appartient un processus ;
    
- la vue des processus ;
    
- la vue réseau ;
    
- la vue des montages ;
    
- les mappings utilisateurs ;
    
- les cgroups ;
    
- certains offsets temporels.
    

---

## 12.1.2. Même chemin, contenu différent

Une idée importante revient souvent :

```text
Le chemin peut être identique, mais le contenu peut changer selon le namespace.
```

Exemples :

```bash
cat /proc/net/dev
cat /proc/sys/kernel/hostname
cat /proc/self/cgroup
cat /proc/mounts
```

Ces chemins peuvent exister dans plusieurs environnements, mais afficher des informations différentes selon le processus qui les lit.

Nous devons donc toujours poser la question :

```text
Depuis quel namespace lisons-nous ce fichier ?
```

---

# 12.2. `/proc/<PID>/ns`

## 12.2.1. Le répertoire principal

Chaque processus possède un répertoire :

```bash
/proc/<PID>/ns
```

Pour le shell courant :

```bash
ls -l /proc/$$/ns
```

Sortie possible :

```text
cgroup -> cgroup:[4026531835]
ipc    -> ipc:[4026531839]
mnt    -> mnt:[4026531841]
net    -> net:[4026531840]
pid    -> pid:[4026531836]
time   -> time:[4026531834]
user   -> user:[4026531837]
uts    -> uts:[4026531838]
```

Chaque lien symbolique représente le namespace du processus pour un type donné.

---

## 12.2.2. Lire un namespace précis

Nous pouvons lire un namespace avec `readlink`.

Exemples :

```bash
readlink /proc/$$/ns/pid
readlink /proc/$$/ns/net
readlink /proc/$$/ns/mnt
readlink /proc/$$/ns/user
```

Sorties possibles :

```text
pid:[4026531836]
net:[4026531840]
mnt:[4026531841]
user:[4026531837]
```

Le nombre entre crochets identifie une instance de namespace.

Deux processus partagent un namespace donné si la valeur est identique pour ce type de namespace.

---

## 12.2.3. Comparer avec le PID 1

Nous comparons notre shell avec le PID 1 :

```bash
for ns in cgroup ipc mnt net pid time user uts; do
    printf "%-8s shell=%-24s pid1=%s\n" \
        "$ns" \
        "$(readlink /proc/$$/ns/$ns 2>/dev/null)" \
        "$(readlink /proc/1/ns/$ns 2>/dev/null)"
done
```

Exemple :

```text
cgroup  shell=cgroup:[4026531835]   pid1=cgroup:[4026531835]
ipc     shell=ipc:[4026531839]      pid1=ipc:[4026531839]
mnt     shell=mnt:[4026531841]      pid1=mnt:[4026531841]
net     shell=net:[4026531840]      pid1=net:[4026531840]
pid     shell=pid:[4026531836]      pid1=pid:[4026531836]
time    shell=time:[4026531834]     pid1=time:[4026531834]
user    shell=user:[4026531837]     pid1=user:[4026531837]
uts     shell=uts:[4026531838]      pid1=uts:[4026531838]
```

Si toutes les valeurs sont identiques, nous sommes probablement dans le même ensemble de namespaces que le PID 1 du système courant.

Si certaines valeurs diffèrent, nous sommes dans un environnement partiellement isolé.

---

# 12.3. Comparer deux processus quelconques

## 12.3.1. Script de comparaison

Nous pouvons écrire une fonction simple :

```bash
compare_ns() {
    pid_a="$1"
    pid_b="$2"

    for ns in cgroup ipc mnt net pid time user uts; do
        a=$(readlink "/proc/$pid_a/ns/$ns" 2>/dev/null || echo "absent")
        b=$(readlink "/proc/$pid_b/ns/$ns" 2>/dev/null || echo "absent")

        if [ "$a" = "$b" ]; then
            printf "%-8s partagé   %s\n" "$ns" "$a"
        else
            printf "%-8s différent %s / %s\n" "$ns" "$a" "$b"
        fi
    done
}
```

Utilisation :

```bash
compare_ns $$ 1
```

Ou pour comparer deux processus :

```bash
compare_ns <PID_A> <PID_B>
```

---

## 12.3.2. Interpréter les différences

Si seul `uts` diffère, les processus peuvent avoir un hostname différent.

Si `net` diffère, ils ne voient pas la même pile réseau.

Si `pid` diffère, ils ne voient pas la même table de processus.

Si `mnt` diffère, ils ne voient pas la même table de montages.

Si `user` diffère, ils n’interprètent pas forcément les UID/GID de la même manière.

Nous pouvons donc diagnostiquer rapidement quel type d’isolation est en place.

---

# 12.4. Namespace PID et `/proc`

## 12.4.1. `/proc` comme vue des processus

Le namespace PID influence la vue des processus.

Dans un namespace PID isolé, `/proc` doit refléter les processus visibles dans ce namespace.

Nous créons un namespace PID propre :

```bash
unshare --fork --pid --mount-proc bash
```

Dans ce shell :

```bash
ps -ef
ls /proc | grep -E '^[0-9]+$'
cat /proc/1/comm
```

Nous voyons une table de processus réduite.

---

## 12.4.2. Pourquoi `--mount-proc` est essentiel

Si nous créons un namespace PID sans remonter `/proc` :

```bash
unshare --fork --pid bash
```

puis :

```bash
ps -ef
```

nous pouvons obtenir une vue incohérente.

Le processus est bien dans un nouveau namespace PID, mais `/proc` peut encore être celui du namespace mount parent.

Nous retenons :

```text
Namespace PID isolé + /proc non adapté = observation trompeuse.
```

C’est pour cela que nous utilisons :

```bash
unshare --fork --pid --mount-proc bash
```

---

## 12.4.3. Observer les PID imbriqués avec `NSpid`

Depuis l’hôte, nous pouvons observer un processus dans un namespace PID enfant :

```bash
grep NSpid /proc/<PID_HOTE>/status
```

Exemple :

```text
NSpid:  24531  1
```

Cela signifie que le même processus est vu :

- comme PID `24531` dans le namespace parent ;
    
- comme PID `1` dans le namespace enfant.
    

Cette ligne est très utile pour comprendre les conteneurs.

---

# 12.5. Namespace network et `/proc/net`

## 12.5.1. `/proc/net` dépend du namespace réseau

Le répertoire :

```bash
/proc/net
```

donne des informations sur la pile réseau.

Mais son contenu dépend du namespace réseau du processus qui lit ces fichiers.

Exemples :

```bash
cat /proc/net/dev
cat /proc/net/tcp
cat /proc/net/udp
cat /proc/net/unix
```

Dans deux namespaces réseau différents, ces fichiers peuvent afficher des informations différentes.

---

## 12.5.2. Démonstration avec `ip netns`

Nous créons un namespace réseau :

```bash
sudo ip netns add ns1
sudo ip netns exec ns1 ip link set lo up
```

Nous comparons :

```bash
cat /proc/net/dev
sudo ip netns exec ns1 cat /proc/net/dev
```

Sur l’hôte, nous pouvons voir :

```text
lo
eth0
docker0
wlan0
```

Dans `ns1`, nous pouvons voir seulement :

```text
lo
```

Même chemin, contenu différent.

---

## 12.5.3. Comparer les sockets

Sur l’hôte :

```bash
ss -tulpen
```

Dans le namespace :

```bash
sudo ip netns exec ns1 ss -tulpen
```

Les ports en écoute ne sont pas les mêmes.

Un serveur qui écoute dans `ns1` n’apparaît pas nécessairement dans la vue réseau de l’hôte comme un socket local classique.

---

## 12.5.4. Nettoyage

```bash
sudo ip netns delete ns1
```

---

# 12.6. Namespace UTS et `/proc/sys/kernel/hostname`

## 12.6.1. Le hostname via `/proc`

Le hostname actif est visible dans :

```bash
cat /proc/sys/kernel/hostname
```

Mais ce fichier dépend du namespace UTS.

Nous testons :

```bash
hostname
cat /proc/sys/kernel/hostname
```

Puis :

```bash
unshare --uts bash
hostname ns-proc-uts
hostname
cat /proc/sys/kernel/hostname
exit
```

Après `exit` :

```bash
hostname
cat /proc/sys/kernel/hostname
```

Nous retrouvons le hostname de l’hôte.

---

## 12.6.2. Même fichier, valeur différente

Dans le namespace UTS isolé :

```text
/proc/sys/kernel/hostname → ns-proc-uts
```

Sur l’hôte :

```text
/proc/sys/kernel/hostname → machine-hote
```

Le chemin est identique, mais la valeur dépend du namespace UTS.

C’est une excellente illustration du rôle de `/proc`.

---

# 12.7. Namespace mount et `/proc/mounts`

## 12.7.1. Observer les montages

La vue des montages est visible avec :

```bash
cat /proc/self/mounts
cat /proc/self/mountinfo
findmnt
```

Ces informations dépendent du namespace mount.

Un processus dans un namespace mount isolé peut voir des montages que l’hôte ne voit pas, ou inversement.

---

## 12.7.2. Démonstration

Nous créons un namespace mount :

```bash
unshare --mount bash
mount --make-rprivate /
mkdir -p /tmp/proc-mnt-demo
mount -t tmpfs tmpfs /tmp/proc-mnt-demo
```

Dans le shell isolé :

```bash
findmnt /tmp/proc-mnt-demo
grep proc-mnt-demo /proc/self/mountinfo
```

Depuis l’hôte :

```bash
findmnt /tmp/proc-mnt-demo
grep proc-mnt-demo /proc/self/mountinfo
```

Le montage ne doit normalement pas apparaître depuis l’hôte si la propagation a été rendue privée.

---

## 12.7.3. `/proc/<PID>/mountinfo`

Depuis l’hôte, nous pouvons lire les montages vus par un processus :

```bash
cat /proc/<PID>/mountinfo
```

C’est très utile pour analyser un conteneur.

Nous pouvons aussi entrer dans son namespace mount :

```bash
sudo nsenter --target <PID> --mount findmnt
```

Nous comparons alors :

```text
vue des montages depuis l’hôte
vue des montages depuis le namespace du processus
```

---

## 12.7.4. Nettoyage

Dans le namespace isolé :

```bash
umount /tmp/proc-mnt-demo
exit
```

---

# 12.8. Namespace user et UID/GID maps

## 12.8.1. Fichiers principaux

Le namespace user est observable avec :

```bash
readlink /proc/$$/ns/user
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Ces fichiers indiquent comment les UID et GID sont traduits entre l’intérieur et l’extérieur du namespace.

---

## 12.8.2. Exemple avec `--map-root-user`

Nous lançons :

```bash
unshare --user --map-root-user bash
```

Dans le shell :

```bash
id
readlink /proc/$$/ns/user
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sortie possible :

```text
uid=0(root) gid=0(root)
```

Puis :

```text
         0       1000          1
```

Cela signifie que l’UID `0` dans le namespace correspond à l’UID `1000` dans le namespace parent.

---

## 12.8.3. Diagnostiquer les permissions

Quand un fichier a des propriétaires inattendus dans un conteneur rootless, nous devons comparer :

Dans le conteneur ou namespace :

```bash
id
ls -ln /chemin
cat /proc/$$/uid_map
cat /proc/$$/gid_map
```

Sur l’hôte :

```bash
ls -ln /chemin
```

Nous devons raisonner avec les UID/GID numériques.

Les noms comme `root`, `www-data` ou `user` peuvent être trompeurs, car ils dépendent de `/etc/passwd` et `/etc/group` dans la vue courante.

---

# 12.9. Namespace cgroup et `/proc/self/cgroup`

## 12.9.1. Fichiers importants

Pour le namespace cgroup :

```bash
readlink /proc/$$/ns/cgroup
cat /proc/self/cgroup
```

Le fichier :

```bash
/proc/self/cgroup
```

indique la position du processus dans l’arborescence cgroup telle qu’elle est vue depuis le namespace cgroup courant.

---

## 12.9.2. Vue simplifiée dans les conteneurs

Dans un conteneur, nous pouvons voir :

```text
0::/
```

Depuis l’hôte, pour le même processus, nous pouvons voir :

```text
0::/system.slice/docker-abcdef.scope
```

ou, en Kubernetes :

```text
0::/kubepods.slice/kubepods-burstable.slice/...
```

Le namespace cgroup peut donc masquer une partie du chemin réel de l’hôte.

---

## 12.9.3. Ne pas confondre vue et limite

Le namespace cgroup ne limite pas directement les ressources.

Pour voir les limites effectives en cgroups v2, nous lisons plutôt :

```bash
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.max
```

Nous retenons :

```text
/proc/self/cgroup montre une position vue.
/sys/fs/cgroup expose les contrôles et mesures.
```

---

# 12.10. Namespace IPC et `/proc`

## 12.10.1. Observer le namespace IPC

Nous lisons :

```bash
readlink /proc/$$/ns/ipc
```

Pour comparer avec le PID 1 :

```bash
readlink /proc/1/ns/ipc
```

Le namespace IPC influence la visibilité de certaines ressources IPC.

---

## 12.10.2. `ipcs`

La commande :

```bash
ipcs
```

affiche les ressources IPC System V visibles dans le namespace IPC courant.

Nous pouvons comparer :

```bash
ipcs
unshare --ipc bash
ipcs
exit
```

Les ressources visibles peuvent différer.

---

## 12.10.3. `/dev/shm`

Pour la mémoire partagée POSIX, nous regardons souvent :

```bash
df -h /dev/shm
ls -lh /dev/shm
findmnt /dev/shm
```

Mais `/dev/shm` dépend aussi du namespace mount.

C’est un bon exemple d’interaction entre namespaces :

```text
IPC namespace → ressources IPC visibles
mount namespace → montage /dev/shm visible
```

---

# 12.11. Namespace time et `/proc/self/timens_offsets`

## 12.11.1. Observer le namespace time

Nous lisons :

```bash
readlink /proc/$$/ns/time
readlink /proc/1/ns/time
```

Nous pouvons aussi voir :

```bash
ls -l /proc/$$/ns/time_for_children
```

selon le noyau.

---

## 12.11.2. Lire les offsets

Le fichier principal est :

```bash
cat /proc/self/timens_offsets
```

Sortie possible :

```text
monotonic           0         0
boottime            0         0
```

Ces valeurs indiquent les offsets appliqués à certaines horloges.

---

## 12.11.3. Attention à l’interprétation

Le namespace time ne signifie pas que :

```bash
date
```

affiche forcément une autre date.

Il concerne surtout des offsets pour certaines horloges monotones ou liées au boottime.

Nous devons donc distinguer :

- heure réelle ;
    
- fuseau horaire ;
    
- uptime ;
    
- temps monotone ;
    
- offsets de namespace time.
    

---

# 12.12. `/proc/self`, `/proc/thread-self` et namespaces

## 12.12.1. `/proc/self`

Le lien :

```bash
/proc/self
```

pointe toujours vers le processus courant.

Exemple :

```bash
readlink /proc/self
```

Sortie possible :

```text
12345
```

Dans un namespace PID, ce PID est interprété selon la vue courante.

`/proc/self` est donc très pratique pour écrire des commandes indépendantes du PID numérique.

---

## 12.12.2. `/proc/thread-self`

Le lien :

```bash
/proc/thread-self
```

pointe vers le thread courant.

Il est utile pour les programmes multi-threads.

Dans les manipulations de namespaces de base, nous utilisons surtout `/proc/self`, mais il est important de savoir que `/proc` peut aussi exposer la granularité des threads.

---

# 12.13. Entrer dans les namespaces d’un processus avec `/proc`

## 12.13.1. `nsenter` utilise les références de `/proc`

La commande `nsenter` s’appuie sur les namespaces d’un processus cible.

Exemple :

```bash
sudo nsenter --target <PID> --net bash
```

Elle rejoint le namespace réseau exposé par :

```bash
/proc/<PID>/ns/net
```

Même principe pour :

```bash
sudo nsenter --target <PID> --mount --uts --ipc --net --pid --fork bash
```

---

## 12.13.2. Entrer dans plusieurs namespaces

Pour diagnostiquer un conteneur depuis l’hôte, nous pouvons faire :

```bash
pid=<PID_DU_CONTENEUR>

sudo nsenter --target "$pid" \
    --mount \
    --uts \
    --ipc \
    --net \
    --pid \
    --fork \
    bash
```

Puis :

```bash
hostname
ps -ef
ip addr
findmnt
cat /proc/self/cgroup
```

Nous observons une vue proche de celle du conteneur.

---

## 12.13.3. Pourquoi `--fork` avec `--pid`

Quand nous entrons dans un namespace PID avec `nsenter`, nous utilisons souvent :

```bash
--fork
```

Cela permet de lancer un enfant dans le namespace PID cible et d’obtenir une vue plus cohérente.

Sans cela, certaines observations peuvent être surprenantes.

---

# 12.14. Analyse d’un conteneur avec `/proc`

## 12.14.1. Récupérer le PID hôte

Avec Docker :

```bash
docker inspect --format '{{.State.Pid}}' <container>
```

Exemple :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' mon-conteneur)
```

---

## 12.14.2. Observer les namespaces du conteneur

Depuis l’hôte :

```bash
sudo ls -l /proc/$pid/ns
```

Nous comparons avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous pouvons identifier quels namespaces sont isolés.

---

## 12.14.3. Observer la vue PID

```bash
sudo grep NSpid /proc/$pid/status
```

Exemple :

```text
NSpid:  24531  1
```

Nous comprenons le PID hôte et le PID interne.

---

## 12.14.4. Observer les montages

```bash
sudo cat /proc/$pid/mountinfo | head
```

Ou :

```bash
sudo nsenter --target "$pid" --mount findmnt | head
```

Nous voyons le rootfs, `/proc`, `/sys`, `/dev`, les volumes et les bind mounts.

---

## 12.14.5. Observer le réseau

```bash
sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route
sudo nsenter --target "$pid" --net ss -tulpen
```

Nous observons les interfaces, les routes et les sockets du conteneur.

---

## 12.14.6. Observer les cgroups

```bash
sudo cat /proc/$pid/cgroup
```

Et depuis le conteneur :

```bash
docker exec <container> cat /proc/self/cgroup
```

Nous comparons la vue de l’hôte et la vue interne.

---

# 12.15. Sécurité de `/proc` avec les namespaces

## 12.15.1. Informations sensibles

`/proc` peut exposer des informations sensibles :

```text
/proc/<PID>/cmdline
/proc/<PID>/environ
/proc/<PID>/fd
/proc/<PID>/maps
/proc/<PID>/mountinfo
/proc/<PID>/cgroup
/proc/net/*
```

Ces fichiers peuvent révéler :

- arguments de lancement ;
    
- tokens passés par erreur en ligne de commande ;
    
- variables d’environnement ;
    
- fichiers ouverts ;
    
- chemins internes ;
    
- sockets ;
    
- structure de déploiement ;
    
- informations cgroup ou conteneur.
    

---

## 12.15.2. Exemples dangereux

Secrets en ligne de commande :

```bash
app --token secret123
```

Visible potentiellement dans :

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

Secrets en environnement :

```bash
DATABASE_URL=postgres://user:password@host/db
```

Visible potentiellement dans :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

Fichiers ouverts :

```bash
ls -l /proc/<PID>/fd
```

peut révéler des fichiers sensibles.

---

## 12.15.3. `hidepid`

Sur un serveur multi-utilisateurs, nous pouvons limiter la visibilité de `/proc` avec l’option de montage `hidepid`.

Vérifier :

```bash
findmnt /proc
```

Exemple de configuration :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

`hidepid=2` masque davantage les processus des autres utilisateurs.

Cela ne remplace pas les autres protections, mais réduit l’exposition.

---

## 12.15.4. `/proc` dans les conteneurs

Dans un conteneur, `/proc` doit être monté avec prudence.

Monter le `/proc` de l’hôte dans un conteneur est dangereux :

```bash
docker run -v /proc:/host/proc image
```

Cela peut exposer des informations de l’hôte.

Nous préférons laisser le runtime monter un `/proc` adapté au namespace PID du conteneur.

---

# 12.16. Interactions entre namespaces

## 12.16.1. Aucun namespace ne suffit seul

Nous avons vu que plusieurs vues de `/proc` dépendent de plusieurs namespaces à la fois.

Exemple :

```text
ps dépend de /proc
/proc dépend du namespace mount
la vue des PID dépend du namespace PID
```

Donc, pour que `ps` soit cohérent dans un conteneur, il faut :

- un namespace PID ;
    
- un namespace mount adapté ;
    
- un montage correct de procfs.
    

---

## 12.16.2. Exemple avec `/dev/shm`

Pour `/dev/shm` :

```text
namespace IPC → ressources IPC
namespace mount → montage /dev/shm
cgroups → limite mémoire éventuelle
```

Un problème de mémoire partagée en conteneur peut donc venir de plusieurs couches.

---

## 12.16.3. Exemple avec le réseau

Pour le réseau :

```text
namespace network → interfaces, routes, sockets
namespace mount → fichiers de configuration visibles
/proc/net → vue réseau
cgroups → limites éventuelles
capabilities → droit de configurer le réseau
```

Un diagnostic correct doit croiser les informations.

---

# 12.17. Méthode de diagnostic avec `/proc`

## 12.17.1. Étape 1 : identifier le processus

Nous commençons par trouver le PID :

```bash
ps aux | grep <nom>
```

ou, pour Docker :

```bash
docker inspect --format '{{.State.Pid}}' <container>
```

---

## 12.17.2. Étape 2 : observer les namespaces

```bash
sudo ls -l /proc/<PID>/ns
```

Puis comparer avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous identifions les namespaces partagés et différents.

---

## 12.17.3. Étape 3 : observer chaque vue utile

Pour les PID :

```bash
sudo grep NSpid /proc/<PID>/status
```

Pour les montages :

```bash
sudo cat /proc/<PID>/mountinfo | head
```

Pour le réseau :

```bash
sudo nsenter --target <PID> --net ip addr
sudo nsenter --target <PID> --net ss -tulpen
```

Pour les utilisateurs :

```bash
sudo cat /proc/<PID>/uid_map
sudo cat /proc/<PID>/gid_map
```

Pour les cgroups :

```bash
sudo cat /proc/<PID>/cgroup
```

---

## 12.17.4. Étape 4 : entrer dans le contexte si nécessaire

```bash
sudo nsenter --target <PID> \
    --mount \
    --uts \
    --ipc \
    --net \
    --pid \
    --fork \
    bash
```

Puis :

```bash
hostname
ps -ef
ip addr
findmnt
cat /proc/self/cgroup
id
```

Nous obtenons une vue proche de celle du processus cible.

---

# 12.18. Exercices

## Exercice 1 — Observer tous les namespaces du shell

Nous exécutons :

```bash
ls -l /proc/$$/ns
```

Nous répondons :

1. Quels namespaces voyons-nous ?
    
2. Quel est l’identifiant du namespace PID ?
    
3. Quel est l’identifiant du namespace réseau ?
    
4. Quel est l’identifiant du namespace mount ?
    
5. Ces identifiants sont-ils les mêmes que ceux du PID 1 ?
    

---

## Exercice 2 — Comparer deux processus

Nous lançons :

```bash
unshare --uts bash
hostname ns-test
echo $$
sleep 1000
```

Dans un autre terminal :

```bash
for ns in cgroup ipc mnt net pid time user uts; do
    echo "$ns"
    readlink /proc/<PID>/ns/$ns 2>/dev/null
    readlink /proc/1/ns/$ns 2>/dev/null
    echo
done
```

Nous répondons :

1. Quel namespace diffère ?
    
2. Pourquoi ?
    
3. Les autres namespaces sont-ils partagés ?
    
4. Comment isoler aussi le namespace PID ?
    
5. Comment isoler aussi le namespace réseau ?
    

---

## Exercice 3 — PID namespace et `/proc`

Nous exécutons :

```bash
unshare --fork --pid --mount-proc bash
```

Dans le shell :

```bash
ps -ef
cat /proc/1/comm
ls /proc | grep -E '^[0-9]+$'
```

Nous répondons :

1. Quels processus voyons-nous ?
    
2. Pourquoi `/proc/1` ne désigne-t-il pas le PID 1 de l’hôte ?
    
3. Pourquoi `--mount-proc` est-il nécessaire ?
    

---

## Exercice 4 — Network namespace et `/proc/net`

Nous exécutons :

```bash
sudo ip netns add ns1
sudo ip netns exec ns1 ip link set lo up

cat /proc/net/dev
sudo ip netns exec ns1 cat /proc/net/dev

sudo ip netns delete ns1
```

Nous répondons :

1. Les interfaces visibles sont-elles les mêmes ?
    
2. Pourquoi `/proc/net/dev` change-t-il ?
    
3. Quel namespace influence ce fichier ?
    

---

## Exercice 5 — Mount namespace et `/proc/self/mountinfo`

Nous exécutons :

```bash
unshare --mount bash
mount --make-rprivate /
mkdir -p /tmp/ns-proc-mount
mount -t tmpfs tmpfs /tmp/ns-proc-mount
grep ns-proc-mount /proc/self/mountinfo
```

Depuis l’hôte :

```bash
grep ns-proc-mount /proc/self/mountinfo
```

Nous répondons :

1. Le montage est-il visible dans les deux contextes ?
    
2. Quel namespace explique cette différence ?
    
3. Pourquoi la propagation des montages compte-t-elle ?
    

---

## Exercice 6 — User namespace et mappings

Nous exécutons :

```bash
unshare --user --map-root-user bash
id
cat /proc/$$/uid_map
cat /proc/$$/gid_map
exit
```

Nous répondons :

1. Quel UID voyons-nous dans le namespace ?
    
2. Vers quel UID externe est-il mappé ?
    
3. Pourquoi cela ne donne-t-il pas les droits root sur l’hôte ?
    

---

## Exercice 7 — Cgroup namespace et `/proc/self/cgroup`

Nous exécutons :

```bash
cat /proc/self/cgroup
readlink /proc/$$/ns/cgroup

unshare --cgroup bash
cat /proc/self/cgroup
readlink /proc/$$/ns/cgroup
exit
```

Nous répondons :

1. Le namespace cgroup a-t-il changé ?
    
2. La vue de `/proc/self/cgroup` a-t-elle changé ?
    
3. Cela crée-t-il une limite mémoire ou CPU ?
    
4. Pourquoi faut-il distinguer cgroup namespace et cgroups ?
    

---

## Exercice 8 — Analyse d’un conteneur Docker

Si Docker est disponible :

```bash
docker run --rm -d --name proc-ns-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' proc-ns-demo)

sudo ls -l /proc/$pid/ns
sudo grep NSpid /proc/$pid/status
sudo cat /proc/$pid/cgroup
sudo cat /proc/$pid/mountinfo | head
sudo nsenter --target "$pid" --net ip addr

docker rm -f proc-ns-demo
```

Nous répondons :

1. Quels namespaces sont différents de l’hôte ?
    
2. Quel est le PID du processus dans le conteneur ?
    
3. Quelle vue cgroup voyons-nous depuis l’hôte ?
    
4. Quelles interfaces réseau voit le conteneur ?
    
5. Pourquoi `/proc` est-il indispensable au diagnostic ?
    

---

# 12.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc` est l’interface principale pour observer les namespaces d’un processus.
    
2. Les namespaces sont visibles dans `/proc/<PID>/ns`.
    
3. Deux processus partagent un namespace si le lien symbolique correspondant est identique.
    
4. Le contenu de certains fichiers `/proc` dépend du namespace courant.
    
5. Le namespace PID influence la vue des processus dans `/proc`.
    
6. Le namespace network influence `/proc/net`.
    
7. Le namespace UTS influence `/proc/sys/kernel/hostname`.
    
8. Le namespace mount influence `/proc/self/mounts` et `/proc/self/mountinfo`.
    
9. Le namespace user est observable avec `uid_map` et `gid_map`.
    
10. Le namespace cgroup influence la vue de `/proc/self/cgroup`.
    
11. Le namespace time expose ses offsets dans `/proc/self/timens_offsets`.
    
12. `nsenter` s’appuie sur les namespaces exposés via `/proc/<PID>/ns`.
    
13. Dans un conteneur, `/proc` doit être monté correctement pour refléter la bonne vue.
    
14. `/proc` peut exposer des informations sensibles.
    
15. La sécurité de `/proc` dépend des permissions, des options de montage, des namespaces et des politiques de sécurité.
    
16. Un diagnostic sérieux croise toujours namespaces, `/proc`, cgroups, montages, capabilities et contexte d’exécution.
    

---

# Conclusion du chapitre 12

Nous avons relié les namespaces à `/proc`.

Nous savons maintenant que `/proc` n’est pas seulement une source d’informations système générales. C’est aussi une interface centrale pour observer, comparer et comprendre les namespaces d’un processus.

Nous avons vu que `/proc/<PID>/ns` permet d’identifier les namespaces, que `/proc/net` dépend du namespace réseau, que `/proc/self/mountinfo` dépend du namespace mount, que `uid_map` et `gid_map` expliquent les traductions du namespace user, et que `/proc/self/cgroup` peut être influencé par le namespace cgroup.

Nous retenons surtout que `/proc` est contextuel : ce que nous lisons dépend souvent du processus qui lit et du namespace dans lequel il se trouve.

Dans le chapitre suivant, nous étudions plus précisément la relation entre namespaces et cgroups, afin de clarifier la différence entre isolation des vues et limitation des ressources.

# Chapitre 13 — Namespaces et cgroups

## Objectifs du chapitre

Dans ce chapitre, nous clarifions la relation entre deux mécanismes fondamentaux de Linux :

```text
namespaces
cgroups
```

Ces deux mécanismes sont souvent associés aux conteneurs, mais ils ne jouent pas le même rôle.

Nous devons retenir dès le départ :

```text
Les namespaces isolent ce que les processus voient.
Les cgroups contrôlent ce que les processus consomment.
```

À la fin de ce chapitre, nous savons :

- distinguer clairement namespaces et cgroups ;
    
- expliquer pourquoi les deux sont nécessaires aux conteneurs ;
    
- comprendre les ressources contrôlées par les cgroups ;
    
- lire les limites CPU, mémoire et processus avec cgroups v2 ;
    
- comprendre la différence entre cgroups v1 et cgroups v2 ;
    
- faire le lien avec systemd ;
    
- faire le lien avec Docker, Podman et Kubernetes ;
    
- expliquer ce qu’est un `OOMKilled` ;
    
- diagnostiquer une limite de ressources dans un conteneur.
    

---

# 13.1. Pourquoi comparer namespaces et cgroups ?

## 13.1.1. Deux mécanismes souvent confondus

Quand nous parlons de conteneurs, nous disons souvent :

```text
Docker isole les processus.
Kubernetes limite les ressources.
Un conteneur a son propre réseau.
Un pod a une limite mémoire.
```

Derrière ces phrases, plusieurs mécanismes Linux interviennent.

Les namespaces permettent de créer des vues isolées :

```text
vue des processus
vue réseau
vue des montages
vue des utilisateurs
vue du hostname
vue des IPC
```

Les cgroups permettent de contrôler les ressources :

```text
CPU
mémoire
I/O
nombre de processus
périphériques
```

Un conteneur complet utilise donc les deux.

---

## 13.1.2. Exemple simple

Imaginons un processus dans un conteneur.

Grâce aux namespaces, il peut voir :

```text
PID 1
hostname app-1
interface eth0
racine / propre au conteneur
seulement ses propres processus
```

Mais sans cgroups, il pourrait potentiellement consommer trop de ressources :

```text
toute la mémoire disponible
trop de CPU
trop de processus
trop d'I/O disque
```

Les namespaces ne suffisent donc pas.

Inversement, avec seulement des cgroups, nous pourrions limiter la mémoire et le CPU, mais le processus verrait encore l’environnement de l’hôte.

Nous avons besoin des deux.

---

# 13.2. Synthèse fondamentale

## 13.2.1. Namespaces : isolation des vues

Les namespaces répondent à la question :

```text
Que voit le processus ?
```

Exemples :

|Namespace|Ce qu’il isole|
|---|---|
|`pid`|les identifiants et la vue des processus|
|`net`|interfaces, routes, sockets, ports|
|`mnt`|points de montage|
|`uts`|hostname et domainname|
|`ipc`|ressources IPC|
|`user`|UID, GID et capabilities|
|`cgroup`|vue des chemins cgroups|
|`time`|offsets de certaines horloges|

Les namespaces changent donc la perception du système.

---

## 13.2.2. Cgroups : contrôle des ressources

Les cgroups répondent à la question :

```text
Que peut consommer le processus ?
```

Ils permettent notamment de contrôler :

|Ressource|Exemple de contrôle|
|---|---|
|CPU|quota, poids, répartition|
|mémoire|limite maximale, consommation actuelle|
|I/O|limitation ou observation des accès disque|
|pids|nombre maximal de processus/threads|
|devices|accès à certains périphériques|
|pressure|observation de la pression CPU, mémoire, I/O|

Les cgroups ne changent pas forcément ce que le processus voit. Ils imposent des règles de consommation.

---

## 13.2.3. Résumé

Nous retenons :

```text
Namespaces = isolation des vues.
Cgroups = contrôle et comptabilité des ressources.
Capabilities = contrôle fin des privilèges.
Seccomp/AppArmor/SELinux = réduction de la surface d’attaque.
```

Un conteneur sérieux combine ces mécanismes.

---

# 13.3. Exemple : namespace PID sans cgroup pids

## 13.3.1. Situation

Nous créons un namespace PID :

```bash
unshare --fork --pid --mount-proc bash
```

Dans ce namespace, nous voyons peu de processus :

```bash
ps -ef
```

Nous pouvons avoir :

```text
UID   PID  PPID  CMD
user    1     0  bash
user    7     1  ps -ef
```

La vue est isolée.

---

## 13.3.2. Limite

Mais cela ne limite pas automatiquement le nombre de processus que nous pouvons créer.

Un processus mal écrit ou malveillant peut lancer de nombreux enfants :

```bash
:(){ :|:& };:
```

Cette commande est une fork bomb. Nous ne l’exécutons pas en vrai sur une machine non protégée.

Le namespace PID isole la vue des processus, mais ne limite pas le nombre de processus.

Pour limiter le nombre de processus, nous utilisons le contrôleur cgroup `pids`.

---

## 13.3.3. Contrôleur `pids`

Avec cgroups v2, nous pouvons lire :

```bash
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Exemple :

```text
pids.current = 42
pids.max     = 512
```

Cela signifie que le cgroup contient actuellement 42 processus ou threads, avec une limite de 512.

Nous retenons :

```text
Namespace PID : ce que nous voyons.
Cgroup pids  : combien de processus nous pouvons créer.
```

---

# 13.4. Exemple : namespace network sans limite réseau

## 13.4.1. Situation

Nous créons un namespace réseau :

```bash
sudo ip netns add ns1
```

Nous lui donnons une interface `veth`, une IP et une route.

Le namespace réseau isole :

```text
interfaces
routes
sockets
ports
loopback
```

Un processus dans `ns1` ne voit pas directement les interfaces de l’hôte.

---

## 13.4.2. Limite

Le namespace réseau n’impose pas automatiquement :

- une limite de débit ;
    
- une limite de nombre de connexions ;
    
- une politique de qualité de service ;
    
- une interdiction d’émettre trop de trafic ;
    
- une règle de filtrage applicative.
    

Pour cela, nous avons besoin d’autres mécanismes :

```text
tc
nftables
iptables
eBPF
CNI Kubernetes
politiques réseau
cgroups selon les ressources concernées
```

Le namespace réseau isole la pile réseau. Il ne définit pas à lui seul toute la politique réseau.

---

# 13.5. Exemple : namespace mount sans quota disque

## 13.5.1. Situation

Le namespace mount permet à un processus de voir une arborescence de fichiers différente.

Dans un conteneur, nous pouvons avoir :

```text
/
├── bin
├── etc
├── app
├── proc
└── tmp
```

Cette racine n’est pas la racine réelle de l’hôte.

---

## 13.5.2. Limite

Le namespace mount ne limite pas automatiquement :

- la quantité de disque utilisée ;
    
- le nombre de fichiers créés ;
    
- les I/O disque ;
    
- la vitesse d’écriture ;
    
- l’espace consommé dans un volume.
    

Pour cela, nous avons besoin d’autres mécanismes :

```text
quotas filesystem
cgroups I/O
limites runtime
limites Kubernetes ephemeral-storage
politiques de stockage
```

Nous retenons :

```text
Namespace mount : quelle arborescence nous voyons.
Limites disque/I/O : combien nous pouvons écrire ou à quelle vitesse.
```

---

# 13.6. Les ressources contrôlées par les cgroups

## 13.6.1. CPU

Les cgroups peuvent contrôler l’usage CPU.

Avec cgroups v2 :

```bash
cat /sys/fs/cgroup/cpu.max
```

Sorties possibles :

```text
max 100000
```

Cela signifie qu’il n’y a pas de quota CPU explicite.

Autre sortie :

```text
200000 100000
```

Cela signifie que le cgroup peut utiliser 200000 microsecondes de CPU toutes les 100000 microsecondes.

Nous interprétons :

```text
200000 / 100000 = 2 CPU
```

---

## 13.6.2. Mémoire

Les cgroups peuvent mesurer et limiter la mémoire.

Avec cgroups v2 :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
```

Exemple :

```text
memory.current = 120000000
memory.max     = 536870912
```

Cela signifie :

```text
consommation actuelle : environ 120 Mo
limite : 512 Mio
```

Si `memory.max` contient :

```text
max
```

cela signifie qu’aucune limite stricte n’est définie à ce niveau.

---

## 13.6.3. Nombre de processus

Le contrôleur `pids` limite le nombre de processus et threads.

```bash
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Exemple :

```text
pids.current = 30
pids.max     = 256
```

Cela évite notamment qu’un processus crée un nombre illimité d’enfants.

---

## 13.6.4. I/O

Les cgroups peuvent aussi contrôler ou mesurer les I/O.

Avec cgroups v2, nous pouvons voir des fichiers comme :

```bash
cat /sys/fs/cgroup/io.stat
cat /sys/fs/cgroup/io.max
```

`io.stat` donne des statistiques d’I/O par périphérique.

`io.max` peut permettre de limiter certaines opérations selon les périphériques.

L’usage précis dépend du noyau, du contrôleur activé et du stockage utilisé.

---

## 13.6.5. Pression de ressources

Linux expose aussi des informations de pression, appelées PSI, _Pressure Stall Information_.

Exemples :

```bash
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io
```

Et parfois dans les cgroups :

```bash
cat /sys/fs/cgroup/cpu.pressure
cat /sys/fs/cgroup/memory.pressure
cat /sys/fs/cgroup/io.pressure
```

Ces informations indiquent si des tâches attendent des ressources.

Elles sont utiles pour diagnostiquer une saturation.

---

# 13.7. cgroups v1 et cgroups v2

## 13.7.1. Pourquoi deux versions ?

Les cgroups ont évolué.

Historiquement, Linux utilisait cgroups v1.

Aujourd’hui, les systèmes modernes utilisent de plus en plus cgroups v2.

Nous devons comprendre les deux, car nous pouvons encore rencontrer les deux modèles.

---

## 13.7.2. cgroups v1

Avec cgroups v1, chaque contrôleur peut avoir sa propre hiérarchie.

Exemple :

```text
memory
cpu,cpuacct
pids
blkio
devices
```

Dans `/proc/self/cgroup`, nous pouvons voir plusieurs lignes :

```text
12:memory:/user.slice
11:cpu,cpuacct:/user.slice
10:pids:/user.slice
```

Cela peut rendre l’analyse plus complexe.

---

## 13.7.3. cgroups v2

Avec cgroups v2, nous avons une hiérarchie unifiée.

Dans `/proc/self/cgroup`, nous voyons souvent une seule ligne :

```text
0::/user.slice/user-1000.slice/session-3.scope
```

Les fichiers de contrôle sont généralement dans :

```bash
/sys/fs/cgroup
```

Exemples :

```bash
cat /sys/fs/cgroup/cgroup.controllers
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/cpu.max
```

---

## 13.7.4. Vérifier la version utilisée

Nous pouvons vérifier le montage :

```bash
findmnt /sys/fs/cgroup
```

Si nous voyons :

```text
FSTYPE cgroup2
```

nous utilisons cgroups v2.

Nous pouvons aussi tester :

```bash
test -f /sys/fs/cgroup/cgroup.controllers && echo "cgroups v2"
```

---

# 13.8. systemd comme gestionnaire de cgroups

## 13.8.1. systemd organise les processus

Sur beaucoup de distributions modernes, systemd organise les processus en cgroups.

Nous pouvons voir l’arborescence :

```bash
systemd-cgls
```

Exemple conceptuel :

```text
Control group /:
├─user.slice
│ └─user-1000.slice
│   └─session-3.scope
└─system.slice
  ├─ssh.service
  ├─docker.service
  └─nginx.service
```

systemd ne se contente pas de lancer des services. Il les place dans des groupes de contrôle.

---

## 13.8.2. Observer la consommation

Nous pouvons utiliser :

```bash
systemd-cgtop
```

Cela donne une vue dynamique de la consommation des cgroups :

```text
Control Group           Tasks   %CPU   Memory
/                         250    8.0    2.3G
/system.slice              80    2.0    800M
/user.slice               120    5.5    1.2G
```

C’est une vue orientée groupes de processus, souvent plus pertinente qu’un simple `top`.

---

## 13.8.3. Limiter un service systemd

Dans une unité systemd, nous pouvons définir :

```ini
[Service]
MemoryMax=512M
CPUQuota=200%
TasksMax=512
```

Ces paramètres se traduisent en configuration cgroup.

Le service reste un ensemble de processus Linux, mais systemd organise et limite cet ensemble.

---

## 13.8.4. Lien avec les namespaces

systemd peut aussi lancer des services avec certaines formes d’isolation.

Exemples de directives :

```ini
PrivateTmp=true
PrivateNetwork=true
ProtectSystem=strict
ReadWritePaths=/var/lib/app
```

Certaines utilisent des namespaces ou des mécanismes associés.

Nous voyons donc que systemd combine aussi plusieurs briques du noyau.

---

# 13.9. Docker, Podman et les cgroups

## 13.9.1. Options Docker courantes

Docker utilise les cgroups pour appliquer des limites.

Exemples :

```bash
docker run --memory=512m image
docker run --cpus=2 image
docker run --pids-limit=256 image
```

Ces options ne relèvent pas principalement des namespaces.

Elles configurent des cgroups.

---

## 13.9.2. Exemple mémoire

Nous lançons :

```bash
docker run --rm -it --memory=256m alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/memory.current 2>/dev/null
```

Avec cgroups v2, nous pouvons voir :

```text
268435456
```

Cela correspond à 256 Mio.

---

## 13.9.3. Exemple CPU

Nous lançons :

```bash
docker run --rm -it --cpus=2 alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/cpu.max 2>/dev/null
```

Nous pouvons voir une valeur proche de :

```text
200000 100000
```

Cela correspond à environ 2 CPU.

---

## 13.9.4. Exemple pids

Nous lançons :

```bash
docker run --rm -it --pids-limit=128 alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/pids.max 2>/dev/null
cat /sys/fs/cgroup/pids.current 2>/dev/null
```

Nous voyons la limite de processus.

---

## 13.9.5. Podman rootless

Podman rootless utilise aussi les cgroups, mais le fonctionnement dépend de :

- cgroups v2 ;
    
- systemd utilisateur ;
    
- droits de l’utilisateur ;
    
- configuration de la distribution ;
    
- runtime OCI ;
    
- version de Podman.
    

Nous devons donc diagnostiquer avec :

```bash
podman info
cat /proc/self/cgroup
systemd-cgls --user
```

selon le contexte.

---

# 13.10. Kubernetes requests et limits

## 13.10.1. Requests

Dans Kubernetes, une `request` indique la quantité de ressources demandée pour planifier le pod.

Exemple :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
```

Cela signifie :

```text
Nous demandons au scheduler de prévoir 256 Mio de mémoire.
Nous demandons 0,5 CPU.
```

La request influence la planification.

---

## 13.10.2. Limits

Une `limit` définit une limite maximale.

Exemple :

```yaml
resources:
  limits:
    memory: "512Mi"
    cpu: "1"
```

Cela signifie :

```text
Le conteneur ne doit pas dépasser 512 Mio de mémoire.
Le conteneur peut utiliser jusqu’à 1 CPU.
```

Ces limites sont appliquées via les cgroups sur le nœud.

---

## 13.10.3. Exemple complet

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Interprétation :

```text
Le pod demande 256 Mio et 0,5 CPU pour la planification.
Le pod est limité à 512 Mio et 1 CPU à l’exécution.
```

Nous devons distinguer planification et limite réelle.

---

## 13.10.4. Qualité de service Kubernetes

Kubernetes classe les pods en QoS selon les requests et limits :

```text
Guaranteed
Burstable
BestEffort
```

Résumé :

|Classe|Condition générale|
|---|---|
|`Guaranteed`|requests = limits pour CPU et mémoire sur tous les conteneurs|
|`Burstable`|au moins une request ou limit, mais pas Guaranteed|
|`BestEffort`|aucune request ni limit|

Cette classe influence notamment les décisions d’éviction en cas de pression sur le nœud.

---

# 13.11. OOMKilled

## 13.11.1. Définition

`OOMKilled` signifie que le processus a été tué à cause d’un dépassement mémoire.

OOM signifie :

```text
Out Of Memory
```

Dans Kubernetes, nous pouvons voir :

```bash
kubectl describe pod <pod>
```

avec un état :

```text
Reason: OOMKilled
Exit Code: 137
```

Cela indique généralement que le conteneur a dépassé sa limite mémoire.

---

## 13.11.2. Ce qui se passe côté Linux

Si un processus dépasse la limite `memory.max` de son cgroup, le noyau peut tuer un processus dans ce cgroup.

C’est une action du noyau liée au contrôleur mémoire des cgroups.

Ce n’est pas un namespace qui tue le processus.

Nous retenons :

```text
OOMKilled = limite mémoire cgroup dépassée.
```

---

## 13.11.3. Diagnostiquer un OOMKilled

Nous vérifions :

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod
kubectl get events
```

Sur le nœud, si nous avons accès :

```bash
dmesg | grep -i oom
journalctl -k | grep -i oom
```

Et dans certains cas :

```bash
cat /sys/fs/cgroup/memory.events
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
```

---

## 13.11.4. Solutions possibles

Selon le cas :

- augmenter la limite mémoire ;
    
- réduire la consommation applicative ;
    
- corriger une fuite mémoire ;
    
- ajuster le nombre de workers ;
    
- limiter les caches ;
    
- améliorer les paramètres JVM, Node.js, Python, etc. ;
    
- définir des requests plus réalistes ;
    
- surveiller la mémoire avec Prometheus.
    

Nous ne répondons pas simplement “ajoutons de la mémoire” sans analyser le comportement.

---

# 13.12. Exemple complet : conteneur limité en mémoire

## 13.12.1. Lancer le conteneur

Nous lançons :

```bash
docker run --rm -it --name mem-demo --memory=256m alpine sh
```

Dans le conteneur :

```sh
cat /proc/self/cgroup
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/memory.current 2>/dev/null
```

Nous observons la limite.

---

## 13.12.2. Depuis l’hôte

Dans un autre terminal :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' mem-demo)
sudo cat /proc/$pid/cgroup
sudo readlink /proc/$pid/ns/cgroup
```

Nous pouvons comparer :

```text
vue depuis l’hôte
vue depuis le conteneur
```

Le namespace cgroup peut changer la vue du chemin, mais la limite est effective.

---

## 13.12.3. Nettoyage

Dans le conteneur :

```sh
exit
```

Ou depuis l’hôte :

```bash
docker rm -f mem-demo
```

---

# 13.13. Exemple complet : limite de processus

## 13.13.1. Lancer un conteneur avec limite pids

Nous lançons :

```bash
docker run --rm -it --pids-limit=64 alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/pids.max 2>/dev/null
cat /sys/fs/cgroup/pids.current 2>/dev/null
```

Nous voyons la limite.

---

## 13.13.2. Pourquoi c’est utile ?

Cela protège contre :

- fork bomb ;
    
- bugs créant trop de threads ;
    
- runaway process ;
    
- mauvaise configuration de workers ;
    
- épuisement de la table des processus.
    

Le namespace PID ne suffit pas à cela.

---

# 13.14. Exemple complet : CPU limité

## 13.14.1. Lancer un conteneur avec CPU limité

```bash
docker run --rm -it --cpus=1 alpine sh
```

Dans le conteneur :

```sh
cat /sys/fs/cgroup/cpu.max 2>/dev/null
```

Nous voyons généralement un quota correspondant à 1 CPU.

---

## 13.14.2. Interprétation

Avec cgroups v2 :

```text
100000 100000
```

signifie environ 1 CPU.

```text
50000 100000
```

signifie environ 0,5 CPU.

```text
200000 100000
```

signifie environ 2 CPU.

Nous devons toutefois tenir compte du scheduler, de la charge globale et de la configuration du runtime.

---

# 13.15. Observabilité des cgroups

## 13.15.1. Avec les outils système

Nous pouvons observer les cgroups avec :

```bash
systemd-cgls
systemd-cgtop
cat /proc/self/cgroup
cat /sys/fs/cgroup/*
```

Pour Docker :

```bash
docker stats
```

Pour Kubernetes :

```bash
kubectl top pod
kubectl top node
```

---

## 13.15.2. Avec Prometheus

Dans un cluster Kubernetes, les métriques cgroups sont souvent collectées via :

```text
kubelet
cAdvisor
node-exporter
Prometheus
```

Prometheus permet ensuite d’historiser :

- CPU consommé ;
    
- mémoire consommée ;
    
- redémarrages ;
    
- OOMKilled ;
    
- limites ;
    
- requests ;
    
- saturation nœud.
    

---

## 13.15.3. Pourquoi `/proc` et `/sys` ne suffisent pas en production

`/proc` et `/sys/fs/cgroup` donnent une vue locale et instantanée.

En production, nous avons besoin :

- d’historique ;
    
- de graphes ;
    
- d’alertes ;
    
- d’agrégation multi-nœuds ;
    
- de corrélation avec les logs ;
    
- de lien avec les déploiements ;
    
- de métriques applicatives.
    

Les cgroups fournissent les compteurs bruts. L’observabilité les transforme en information exploitable.

---

# 13.16. Sécurité et isolation

## 13.16.1. Pourquoi les cgroups renforcent la sécurité

Les cgroups limitent l’impact d’un processus qui consomme trop.

Exemples :

- une fuite mémoire ne doit pas tuer tout le nœud ;
    
- une fork bomb doit être limitée ;
    
- un service CPU-bound ne doit pas monopoliser toute la machine ;
    
- un conteneur ne doit pas saturer les I/O disque ;
    
- un pod ne doit pas faire tomber les autres pods.
    

Les cgroups participent donc à la stabilité et à la sécurité opérationnelle.

---

## 13.16.2. Ce que les cgroups ne font pas

Les cgroups ne masquent pas les processus.

Ils ne donnent pas un hostname différent.

Ils ne créent pas de pile réseau séparée.

Ils ne changent pas les UID/GID.

Ils ne remplacent pas les permissions Unix.

Ils ne filtrent pas les appels système.

Ils ne remplacent pas AppArmor, SELinux ou seccomp.

Nous devons donc les combiner avec les namespaces et les mécanismes de sécurité.

---

## 13.16.3. Mauvaise configuration

Un conteneur sans limites peut poser problème.

Exemple Kubernetes :

```yaml
resources: {}
```

Cela peut produire un pod `BestEffort`.

En cas de pression mémoire, ce pod est plus facilement évincé.

À l’inverse, une limite trop basse peut provoquer des `OOMKilled` répétés.

Nous devons donc dimensionner correctement.

---

# 13.17. Méthode de diagnostic : ressource saturée dans un conteneur

## 13.17.1. Étape 1 : identifier le symptôme

Nous commençons par identifier :

```text
CPU élevé
mémoire élevée
OOMKilled
trop de processus
latence I/O
pod évincé
conteneur redémarre
```

---

## 13.17.2. Étape 2 : vérifier les limites

Dans Docker :

```bash
docker inspect <container>
docker stats
```

Dans Kubernetes :

```bash
kubectl describe pod <pod>
kubectl top pod
kubectl get events
```

Dans le conteneur ou sur le nœud :

```bash
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.max
cat /sys/fs/cgroup/pids.current
```

---

## 13.17.3. Étape 3 : distinguer vue et limite

Nous ne confondons pas :

```text
/proc/self/cgroup
```

avec :

```text
/sys/fs/cgroup/memory.max
/sys/fs/cgroup/cpu.max
/sys/fs/cgroup/pids.max
```

Le premier montre une position ou une vue.

Les seconds montrent des limites ou mesures.

---

## 13.17.4. Étape 4 : relier à l’application

Nous analysons ensuite l’application :

- nombre de workers ;
    
- taille des caches ;
    
- fuite mémoire ;
    
- threads ;
    
- batchs trop gros ;
    
- requêtes lentes ;
    
- configuration JVM/Node/Python ;
    
- pool de connexions ;
    
- limites applicatives.
    

Les cgroups nous disent qu’une limite est atteinte. Ils ne nous disent pas toujours pourquoi l’application consomme autant.

---

# 13.18. Travaux pratiques

## TP 1 — Distinguer namespaces et cgroups

Nous exécutons :

```bash
readlink /proc/$$/ns/pid
readlink /proc/$$/ns/net
readlink /proc/$$/ns/mnt
cat /proc/self/cgroup
```

Nous répondons :

1. Quels namespaces voyons-nous ?
    
2. Que nous dit `/proc/self/cgroup` ?
    
3. Est-ce que les namespaces affichent des limites de ressources ?
    
4. Est-ce que `/proc/self/cgroup` isole les processus ?
    

---

## TP 2 — Identifier cgroups v1 ou v2

Nous exécutons :

```bash
findmnt /sys/fs/cgroup
test -f /sys/fs/cgroup/cgroup.controllers && echo "cgroups v2"
cat /proc/self/cgroup
```

Nous répondons :

1. Le système utilise-t-il cgroups v2 ?
    
2. Combien de lignes contient `/proc/self/cgroup` ?
    
3. Que signifie `0::/` en cgroups v2 ?
    

---

## TP 3 — Lire les limites cgroups v2

Si cgroups v2 est disponible :

```bash
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/pids.current
cat /sys/fs/cgroup/pids.max
```

Nous répondons :

1. Quelle est la consommation mémoire actuelle ?
    
2. Quelle est la limite mémoire ?
    
3. Quelle est la limite CPU ?
    
4. Quelle est la limite de processus ?
    

---

## TP 4 — Observer systemd

Nous exécutons :

```bash
systemd-cgls
systemd-cgtop
systemctl status ssh
```

ou un autre service disponible.

Nous répondons :

1. Où apparaît le service dans l’arborescence cgroup ?
    
2. Quelle consommation voyons-nous ?
    
3. Pourquoi systemd utilise-t-il les cgroups ?
    

---

## TP 5 — Docker avec limite mémoire

Si Docker est disponible :

```bash
docker run --rm -d --name mem-demo --memory=256m alpine sleep 1000
docker stats --no-stream mem-demo
docker exec mem-demo sh -c 'cat /sys/fs/cgroup/memory.max 2>/dev/null; cat /sys/fs/cgroup/memory.current 2>/dev/null'
docker rm -f mem-demo
```

Nous répondons :

1. Quelle limite mémoire est appliquée ?
    
2. Où la lisons-nous ?
    
3. Est-ce un namespace ou un cgroup qui applique cette limite ?
    

---

## TP 6 — Docker avec limite pids

Si Docker est disponible :

```bash
docker run --rm -d --name pids-demo --pids-limit=64 alpine sleep 1000
docker exec pids-demo sh -c 'cat /sys/fs/cgroup/pids.max 2>/dev/null; cat /sys/fs/cgroup/pids.current 2>/dev/null'
docker rm -f pids-demo
```

Nous répondons :

1. Quelle limite de processus est appliquée ?
    
2. Pourquoi cette limite est-elle utile ?
    
3. Pourquoi le namespace PID ne suffit-il pas ?
    

---

## TP 7 — Kubernetes requests et limits

Nous étudions :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Nous répondons :

1. Quelle quantité de mémoire est demandée ?
    
2. Quelle quantité de mémoire est limitée ?
    
3. Quelle quantité de CPU est demandée ?
    
4. Quelle quantité de CPU est limitée ?
    
5. Quel mécanisme Linux applique ces limites ?
    
6. Que signifie `OOMKilled` ?
    

---

# 13.19. Pièges classiques

## 13.19.1. Croire que les namespaces limitent les ressources

Un namespace PID ne limite pas le nombre de processus.

Un namespace mount ne limite pas l’espace disque.

Un namespace network ne limite pas automatiquement le débit.

Un namespace user ne limite pas la mémoire.

Les namespaces isolent des vues.

---

## 13.19.2. Croire que les cgroups isolent les vues

Les cgroups limitent ou mesurent les ressources, mais ne créent pas à eux seuls :

- un hostname séparé ;
    
- une pile réseau séparée ;
    
- une table de processus séparée ;
    
- un root filesystem séparé.
    

Pour cela, nous avons besoin des namespaces.

---

## 13.19.3. Confondre cgroup namespace et cgroups

Le namespace cgroup isole la vue des chemins cgroups.

Les cgroups appliquent les limites.

Nous ne devons pas dire :

```text
Le namespace cgroup limite la mémoire.
```

Nous devons dire :

```text
Le cgroup limite la mémoire.
Le namespace cgroup peut masquer le chemin cgroup vu par le processus.
```

---

## 13.19.4. Interpréter naïvement `/proc/meminfo` dans un conteneur

Dans un conteneur, `/proc/meminfo` peut parfois refléter la mémoire du nœud plutôt que la limite réelle du cgroup, selon les environnements.

Pour connaître la limite, nous regardons plutôt :

```bash
cat /sys/fs/cgroup/memory.max
```

avec cgroups v2.

---

## 13.19.5. Oublier la QoS Kubernetes

Un pod sans requests ni limits peut être plus vulnérable aux évictions.

Un pod avec une limite trop basse peut être `OOMKilled`.

Un pod avec des requests trop hautes peut être difficile à planifier.

Les cgroups ne remplacent pas le dimensionnement applicatif.

---

# 13.20. Ce que nous devons retenir

Nous retenons les points suivants :

1. Les namespaces isolent les vues du système.
    
2. Les cgroups contrôlent et mesurent les ressources.
    
3. Les deux mécanismes sont complémentaires.
    
4. Un conteneur utilise généralement namespaces et cgroups.
    
5. Le namespace PID ne limite pas le nombre de processus.
    
6. Le contrôleur cgroup `pids` limite le nombre de processus ou threads.
    
7. Le namespace mount ne limite pas l’espace disque.
    
8. Les cgroups et quotas contrôlent les ressources.
    
9. Le namespace network n’impose pas seul une politique réseau complète.
    
10. Les cgroups peuvent limiter CPU, mémoire, I/O et pids.
    
11. cgroups v1 utilise souvent plusieurs hiérarchies.
    
12. cgroups v2 utilise une hiérarchie unifiée.
    
13. systemd organise les services en cgroups.
    
14. Docker et Podman utilisent les cgroups pour appliquer les limites de ressources.
    
15. Kubernetes traduit requests et limits en configuration cgroup.
    
16. `OOMKilled` correspond généralement à un dépassement de limite mémoire cgroup.
    
17. Le namespace cgroup isole la vue des chemins cgroups, pas les ressources elles-mêmes.
    
18. En diagnostic, nous devons distinguer vue, limite, consommation et contexte d’exécution.
    

---

# Conclusion du chapitre 13

Nous avons étudié la relation entre namespaces et cgroups.

Nous savons maintenant que ces deux mécanismes sont complémentaires, mais profondément différents. Les namespaces répondent à la question “que voit le processus ?”, tandis que les cgroups répondent à la question “que peut consommer le processus ?”.

Cette distinction est essentielle pour comprendre les conteneurs. Un conteneur a besoin de namespaces pour voir son propre environnement, mais il a aussi besoin de cgroups pour ne pas consommer toutes les ressources de l’hôte.

Nous avons aussi relié ces notions à systemd, Docker, Podman et Kubernetes. Nous savons désormais interpréter une limite mémoire, une limite CPU, une limite de processus, un `OOMKilled`, ainsi que la différence entre une vue cgroup simplifiée et une limite réellement appliquée.

Dans le chapitre suivant, nous étudions les capabilities et la sécurité, afin de comprendre pourquoi les namespaces et les cgroups ne suffisent pas à eux seuls pour construire une isolation robuste.

# Chapitre 14 — Namespaces, capabilities et sécurité

## Objectifs du chapitre

Dans ce chapitre, nous étudions la sécurité autour des namespaces Linux.

Jusqu’ici, nous avons vu que les namespaces permettent d’isoler différentes vues du système :

```text
PID
réseau
montages
utilisateurs
IPC
hostname
cgroups
temps
```

Nous avons aussi vu que les cgroups permettent de limiter les ressources.

Mais cela ne suffit pas à garantir une isolation robuste.

Un conteneur ou un processus isolé peut encore être dangereux si :

- il possède trop de privilèges ;
    
- il dispose de capabilities trop larges ;
    
- il accède à des montages sensibles ;
    
- il partage certains namespaces avec l’hôte ;
    
- il peut appeler trop d’appels système ;
    
- il n’est pas limité par AppArmor, SELinux ou seccomp ;
    
- il tourne en mode privilégié ;
    
- il a accès à `/proc`, `/sys`, `/dev` ou au socket Docker de l’hôte.
    

À la fin de ce chapitre, nous savons :

- expliquer pourquoi les namespaces ne suffisent pas à sécuriser un conteneur ;
    
- comprendre le rôle des capabilities Linux ;
    
- expliquer pourquoi `CAP_SYS_ADMIN` est particulièrement sensible ;
    
- observer les capabilities d’un processus ;
    
- comprendre le rôle de seccomp ;
    
- comprendre le rôle d’AppArmor et de SELinux ;
    
- identifier les risques des conteneurs privilégiés ;
    
- identifier les montages dangereux ;
    
- proposer des bonnes pratiques de durcissement.
    

---

# 14.1. Pourquoi les namespaces ne suffisent pas

## 14.1.1. Les namespaces isolent des vues

Nous avons vu que les namespaces permettent de modifier ce qu’un processus voit.

Par exemple :

```text
namespace PID     → le processus ne voit pas tous les PID
namespace network → le processus ne voit pas toutes les interfaces réseau
namespace mount   → le processus ne voit pas tous les montages
namespace UTS     → le processus voit un hostname isolé
namespace IPC     → le processus voit des IPC isolées
namespace user    → le processus peut voir des UID/GID mappés
```

Mais voir moins de choses ne signifie pas forcément être parfaitement sécurisé.

---

## 14.1.2. Le noyau reste partagé

Dans un conteneur Linux classique, le noyau est partagé avec l’hôte.

Cela signifie que les processus de l’hôte et les processus du conteneur utilisent le même noyau.

Nous devons donc retenir :

```text
Un conteneur Linux n’est pas une machine virtuelle.
Il ne possède pas son propre noyau.
```

Si un processus dans un conteneur exploite une vulnérabilité du noyau, il peut potentiellement sortir de l’isolation prévue.

C’est une différence fondamentale avec une machine virtuelle, où le noyau invité est séparé du noyau hôte par un hyperviseur.

---

## 14.1.3. L’isolation dépend de la configuration réelle

Nous ne devons jamais dire simplement :

```text
C’est dans un conteneur, donc c’est sécurisé.
```

Nous devons plutôt demander :

```text
Quels namespaces sont isolés ?
Quels namespaces sont partagés avec l’hôte ?
Quelles capabilities sont accordées ?
Quels montages sont exposés ?
Quels cgroups limitent les ressources ?
Quel profil seccomp est actif ?
Quel profil AppArmor ou SELinux est actif ?
Le conteneur tourne-t-il en root ?
Le conteneur est-il privilégié ?
```

La sécurité réelle dépend de l’ensemble de ces réponses.

---

# 14.2. Rappel : les trois grandes dimensions

## 14.2.1. Ce que le processus voit

C’est le rôle principal des namespaces.

```text
Namespaces = isolation des vues
```

Exemples :

- vue des processus ;
    
- vue réseau ;
    
- vue des montages ;
    
- vue des utilisateurs ;
    
- vue des IPC ;
    
- vue du hostname.
    

---

## 14.2.2. Ce que le processus consomme

C’est le rôle principal des cgroups.

```text
Cgroups = contrôle des ressources
```

Exemples :

- mémoire maximale ;
    
- quota CPU ;
    
- nombre de processus ;
    
- I/O ;
    
- accès à certains périphériques ;
    
- pression de ressources.
    

---

## 14.2.3. Ce que le processus a le droit de faire

C’est le rôle de plusieurs mécanismes combinés :

```text
capabilities
seccomp
AppArmor
SELinux
permissions Unix
montages
droits sur les périphériques
configuration du runtime
```

C’est cette troisième dimension qui nous intéresse particulièrement dans ce chapitre.

---

# 14.3. Les capabilities Linux

## 14.3.1. Pourquoi les capabilities existent-elles ?

Historiquement, Linux distinguait principalement :

```text
root
non-root
```

Le compte `root`, UID `0`, avait presque tous les privilèges.

Cette approche est simple, mais trop grossière.

Un programme peut avoir besoin d’un privilège précis, par exemple :

- ouvrir un port inférieur à 1024 ;
    
- changer le propriétaire d’un fichier ;
    
- configurer une interface réseau ;
    
- monter un système de fichiers ;
    
- envoyer un signal à un autre processus ;
    
- modifier certaines limites système.
    

Nous ne voulons pas forcément lui donner tous les droits root pour cela.

Les capabilities découpent donc les privilèges de root en unités plus fines.

---

## 14.3.2. Définition

Une capability est un droit spécifique accordé à un processus.

Nous pouvons résumer :

```text
Avant :
root a tous les privilèges.

Avec les capabilities :
un processus peut recevoir seulement certains privilèges précis.
```

Exemples de capabilities :

|Capability|Rôle simplifié|
|---|---|
|`CAP_CHOWN`|changer le propriétaire d’un fichier|
|`CAP_DAC_OVERRIDE`|contourner certaines permissions fichiers|
|`CAP_NET_BIND_SERVICE`|écouter sur un port inférieur à 1024|
|`CAP_NET_ADMIN`|administrer le réseau|
|`CAP_SYS_ADMIN`|nombreuses opérations système sensibles|
|`CAP_SYS_PTRACE`|tracer ou inspecter d’autres processus|
|`CAP_KILL`|envoyer des signaux à certains processus|
|`CAP_SETUID`|modifier l’UID d’un processus|
|`CAP_SETGID`|modifier le GID d’un processus|
|`CAP_MKNOD`|créer certains fichiers de périphériques|

---

# 14.4. Observer les capabilities d’un processus

## 14.4.1. Avec `/proc/<PID>/status`

Nous pouvons observer les capabilities du shell courant :

```bash
grep Cap /proc/$$/status
```

Sortie possible :

```text
CapInh:  0000000000000000
CapPrm:  000001ffffffffff
CapEff:  000001ffffffffff
CapBnd:  000001ffffffffff
CapAmb:  0000000000000000
```

Ces valeurs sont en hexadécimal.

Elles représentent différents ensembles de capabilities.

---

## 14.4.2. Les principaux ensembles

Nous distinguons notamment :

|Champ|Signification|
|---|---|
|`CapInh`|capabilities héritables|
|`CapPrm`|capabilities permises|
|`CapEff`|capabilities effectives|
|`CapBnd`|capabilities maximales possibles après réduction|
|`CapAmb`|capabilities ambiantes transmissibles à certains exécutables|

Le champ le plus directement important pour l’action courante est souvent :

```text
CapEff
```

car il représente les capabilities effectivement actives.

---

## 14.4.3. Avec `capsh`

L’outil `capsh` rend la sortie plus lisible.

Commande :

```bash
capsh --print
```

Sur Debian/Ubuntu, il peut être fourni par :

```bash
sudo apt install libcap2-bin
```

Sortie typique :

```text
Current: cap_chown,cap_dac_override,cap_fowner,...
Bounding set = cap_chown,cap_dac_override,...
Ambient set =
```

Nous utilisons `capsh` pour comprendre rapidement quels droits possède un processus.

---

## 14.4.4. Décoder les capabilities

Nous pouvons aussi utiliser :

```bash
capsh --decode=<valeur_hexadécimale>
```

Exemple :

```bash
capsh --decode=0000000000000400
```

Cela permet de transformer une valeur hexadécimale de `/proc/<PID>/status` en noms de capabilities.

---

# 14.5. Capabilities et conteneurs

## 14.5.1. Pourquoi les conteneurs ont des capabilities réduites

Un conteneur lancé en root ne reçoit généralement pas toutes les capabilities de root.

Docker, Podman ou Kubernetes réduisent souvent la liste par défaut.

Cela signifie que dans un conteneur :

```bash
id
```

peut afficher :

```text
uid=0(root)
```

mais cela ne signifie pas que le processus possède toutes les capabilities possibles.

Nous devons distinguer :

```text
UID 0
capabilities effectives
namespaces
montages
profils de sécurité
```

---

## 14.5.2. Observer dans Docker

Dans un conteneur Docker :

```bash
docker run --rm -it alpine sh
```

Nous pouvons exécuter :

```sh
grep Cap /proc/$$/status
```

Ou, si `capsh` est disponible dans l’image :

```sh
capsh --print
```

Nous pouvons comparer avec l’hôte.

---

## 14.5.3. Ajouter une capability

Docker permet d’ajouter une capability :

```bash
docker run --cap-add=NET_ADMIN image
```

Cela donne au conteneur davantage de droits réseau.

Exemple : il peut être capable de modifier certaines interfaces ou règles réseau dans son namespace réseau.

Mais si le conteneur partage le namespace réseau de l’hôte, cette capability devient beaucoup plus dangereuse.

---

## 14.5.4. Retirer une capability

Docker permet aussi de retirer des capabilities :

```bash
docker run --cap-drop=ALL image
```

Puis de n’ajouter que ce qui est strictement nécessaire :

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image
```

Cette approche est plus sûre.

Nous appliquons le principe du moindre privilège.

---

# 14.6. `CAP_SYS_ADMIN`, la capability dangereuse

## 14.6.1. Pourquoi elle est particulière

`CAP_SYS_ADMIN` est souvent considérée comme une capability très large.

Elle donne accès à de nombreuses opérations système sensibles.

Elle intervient notamment dans plusieurs opérations liées à :

- montages ;
    
- namespaces ;
    
- certains paramètres noyau ;
    
- certains systèmes de fichiers ;
    
- opérations avancées d’administration ;
    
- manipulation de ressources globales selon contexte.
    

Elle est parfois surnommée de manière informelle “le nouveau root”, parce qu’elle couvre énormément d’actions.

---

## 14.6.2. Pourquoi éviter `CAP_SYS_ADMIN`

Si nous donnons `CAP_SYS_ADMIN` à un conteneur, nous augmentons fortement les risques.

Exemple :

```bash
docker run --cap-add=SYS_ADMIN image
```

Cette option peut être nécessaire dans certains cas très spécifiques, mais elle doit toujours être justifiée.

Avant de l’utiliser, nous demandons :

```text
Quelle opération exacte nécessite CAP_SYS_ADMIN ?
Existe-t-il une capability plus précise ?
Pouvons-nous éviter ce besoin ?
Le conteneur peut-il tourner autrement ?
Le montage ou le périphérique exposé est-il vraiment nécessaire ?
```

---

## 14.6.3. Avec un user namespace

Dans un user namespace, `CAP_SYS_ADMIN` peut être limitée au contexte du namespace.

Mais cela ne la rend pas automatiquement inoffensive.

Nous devons toujours nous demander :

```text
Dans quel user namespace cette capability est-elle effective ?
Quelles ressources appartiennent à ce user namespace ?
Quels montages sont exposés ?
Quels autres namespaces sont partagés ?
```

---

# 14.7. Seccomp

## 14.7.1. Qu’est-ce que seccomp ?

Seccomp signifie _secure computing mode_.

C’est un mécanisme du noyau Linux permettant de filtrer les appels système qu’un processus peut effectuer.

Un appel système est une demande faite au noyau.

Exemples :

```text
open
read
write
clone
mount
ptrace
ioctl
execve
chmod
setns
```

Si un processus n’a pas besoin de certains appels système dangereux, nous pouvons les bloquer.

---

## 14.7.2. Pourquoi seccomp est utile

Même si un processus est dans des namespaces, il parle toujours au même noyau que l’hôte.

Limiter les appels système réduit donc la surface d’attaque.

Exemple :

```text
Si l’application n’a pas besoin de ptrace, nous pouvons bloquer ptrace.
Si elle n’a pas besoin de mount, nous pouvons bloquer mount.
Si elle n’a pas besoin de certains appels très spécialisés, nous les bloquons.
```

Seccomp ne remplace pas les namespaces, mais il les complète.

---

## 14.7.3. Seccomp avec Docker

Docker applique généralement un profil seccomp par défaut.

Nous pouvons désactiver seccomp avec :

```bash
docker run --security-opt seccomp=unconfined image
```

Mais c’est une option sensible.

Elle réduit la protection.

Nous l’évitons sauf diagnostic ou besoin très spécifique.

---

## 14.7.4. Profils seccomp personnalisés

Il est possible de fournir un profil seccomp personnalisé.

Conceptuellement :

```bash
docker run --security-opt seccomp=profil.json image
```

Le profil décrit quels appels système sont autorisés ou interdits.

En production, nous pouvons durcir une application en construisant un profil adapté à son comportement réel.

---

# 14.8. AppArmor

## 14.8.1. Rôle d’AppArmor

AppArmor est un mécanisme de contrôle d’accès obligatoire.

Il permet de définir ce qu’un programme a le droit de faire, notamment :

- quels fichiers il peut lire ;
    
- quels fichiers il peut écrire ;
    
- quelles capacités il peut utiliser ;
    
- quels profils il suit.
    

AppArmor fonctionne avec des profils attachés aux programmes ou conteneurs.

---

## 14.8.2. Observer AppArmor

Sur certaines distributions, nous pouvons voir l’état d’AppArmor avec :

```bash
sudo aa-status
```

Ou vérifier le profil d’un processus :

```bash
cat /proc/<PID>/attr/current
```

Exemple possible :

```text
docker-default (enforce)
```

Cela signifie que le processus est confiné par le profil AppArmor `docker-default`.

---

## 14.8.3. Docker et AppArmor

Docker peut utiliser un profil AppArmor par défaut.

Nous pouvons désactiver ce profil avec :

```bash
docker run --security-opt apparmor=unconfined image
```

C’est une option sensible, car elle retire une couche de sécurité.

Nous l’évitons en production sauf besoin très justifié.

---

# 14.9. SELinux

## 14.9.1. Rôle de SELinux

SELinux est un autre mécanisme de contrôle d’accès obligatoire.

Il est très utilisé dans certaines distributions, notamment les familles Red Hat, Fedora, CentOS Stream, Rocky Linux ou AlmaLinux.

SELinux applique des politiques de sécurité basées sur des contextes.

---

## 14.9.2. Observer SELinux

Nous pouvons vérifier l’état :

```bash
getenforce
```

Sorties possibles :

```text
Enforcing
Permissive
Disabled
```

Nous pouvons voir les contextes de fichiers avec :

```bash
ls -Z
```

Et les contextes de processus avec :

```bash
ps -eZ
```

---

## 14.9.3. SELinux et conteneurs

Avec SELinux, les conteneurs peuvent être confinés par des labels spécifiques.

Un conteneur peut se voir interdire l’accès à certains fichiers même si les permissions Unix semblent l’autoriser.

C’est une source fréquente de confusion, mais aussi une couche de sécurité puissante.

Nous devons donc penser à SELinux lorsqu’un accès fichier échoue sans raison apparente sur une distribution qui l’utilise.

---

# 14.10. Conteneurs privilégiés

## 14.10.1. Qu’est-ce qu’un conteneur privilégié ?

Avec Docker :

```bash
docker run --privileged image
```

Avec Kubernetes :

```yaml
securityContext:
  privileged: true
```

Un conteneur privilégié reçoit beaucoup plus de droits que normalement.

Il peut obtenir :

- davantage de capabilities ;
    
- accès plus large à `/dev` ;
    
- restrictions AppArmor/seccomp amoindries selon configuration ;
    
- capacité à effectuer des opérations proches de l’administration hôte ;
    
- accès à des périphériques sensibles.
    

---

## 14.10.2. Pourquoi c’est dangereux

Un conteneur privilégié réduit fortement l’isolation.

Dans certains cas, il peut devenir très proche d’un processus root sur l’hôte.

Nous ne devons donc pas utiliser `--privileged` comme solution rapide à un problème de permission.

Mauvaise démarche :

```text
Ça ne marche pas, donc nous lançons en privileged.
```

Bonne démarche :

```text
Quel droit précis manque ?
Quelle capability manque ?
Quel fichier ou périphérique doit être exposé ?
Pouvons-nous limiter l’accès ?
Pouvons-nous changer l’architecture ?
```

---

## 14.10.3. Cas où cela peut être justifié

Il existe des cas spécifiques :

- agents système ;
    
- outils d’administration de nœud ;
    
- conteneurs de build très particuliers ;
    
- environnements de test isolés ;
    
- outils qui manipulent des périphériques ;
    
- certains composants bas niveau.
    

Mais même dans ces cas, nous devons préférer une permission ciblée à un privilège global.

---

# 14.11. Montages dangereux

## 14.11.1. Pourquoi les montages sont critiques

Le namespace mount donne au conteneur une vue de fichiers.

Mais si nous montons des chemins sensibles de l’hôte, nous créons une passerelle entre l’environnement isolé et l’hôte.

Les montages peuvent donc annuler une partie importante de l’isolation.

---

## 14.11.2. Monter `/`

Montage dangereux :

```bash
docker run -v /:/host image
```

Le conteneur voit toute la racine de l’hôte sous `/host`.

Même si le rootfs du conteneur est isolé, l’hôte est exposé.

Si le conteneur peut écrire, il peut modifier des fichiers de l’hôte.

---

## 14.11.3. Monter `/proc`

Montage dangereux :

```bash
docker run -v /proc:/host/proc image
```

Cela peut exposer :

- processus de l’hôte ;
    
- informations sensibles ;
    
- arguments de ligne de commande ;
    
- variables d’environnement ;
    
- fichiers ouverts ;
    
- informations réseau ;
    
- paramètres noyau.
    

Nous évitons ce montage sauf cas de supervision ou diagnostic très contrôlé.

---

## 14.11.4. Monter `/sys`

Montage dangereux :

```bash
docker run -v /sys:/host/sys image
```

`/sys` expose des objets noyau, des périphériques, des paramètres matériels et des informations sensibles.

Un accès en écriture à `/sys` peut être particulièrement dangereux.

---

## 14.11.5. Monter `/dev`

Montage dangereux :

```bash
docker run -v /dev:/host/dev image
```

Ou donner trop de périphériques.

Certains fichiers de `/dev` permettent d’interagir avec :

- disques ;
    
- GPU ;
    
- terminaux ;
    
- interfaces bas niveau ;
    
- KVM ;
    
- périphériques USB ;
    
- périphériques système.
    

Nous préférons exposer seulement les périphériques nécessaires.

---

## 14.11.6. Monter le socket Docker

Montage très sensible :

```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock image
```

Un processus dans le conteneur peut alors parler au démon Docker de l’hôte.

Dans beaucoup de configurations, cela revient à donner un contrôle très important sur la machine.

Pourquoi ?

Parce qu’un processus qui contrôle Docker peut souvent lancer un autre conteneur avec des montages sensibles, voire avec `--privileged`.

Nous considérons donc ce montage comme quasi équivalent à un accès administratif sur l’hôte.

---

# 14.12. Namespaces partagés avec l’hôte

## 14.12.1. `--pid=host`

Docker :

```bash
docker run --pid=host image
```

Kubernetes :

```yaml
hostPID: true
```

Le conteneur partage le namespace PID de l’hôte.

Conséquences :

- il voit les processus de l’hôte ;
    
- il peut potentiellement interagir avec eux selon ses droits ;
    
- il réduit l’isolation ;
    
- il expose plus d’informations.
    

---

## 14.12.2. `--network=host`

Docker :

```bash
docker run --network=host image
```

Kubernetes :

```yaml
hostNetwork: true
```

Le conteneur partage la pile réseau de l’hôte.

Conséquences :

- il voit les interfaces de l’hôte ;
    
- il partage les ports de l’hôte ;
    
- il peut écouter directement sur les adresses de l’hôte ;
    
- les conflits de ports sont possibles ;
    
- l’isolation réseau est réduite.
    

---

## 14.12.3. `--ipc=host`

Docker :

```bash
docker run --ipc=host image
```

Kubernetes :

```yaml
hostIPC: true
```

Le conteneur partage les ressources IPC de l’hôte.

Conséquences :

- il peut voir certaines IPC de l’hôte ;
    
- il peut interagir avec des ressources de mémoire partagée ou sémaphores selon permissions ;
    
- il réduit l’isolation.
    

---

## 14.12.4. Bilan

Ces options peuvent être nécessaires pour certains agents système, mais elles doivent être considérées comme sensibles.

Nous posons toujours la question :

```text
Avons-nous vraiment besoin de partager ce namespace avec l’hôte ?
Existe-t-il une alternative plus ciblée ?
```

---

# 14.13. User namespace et sécurité

## 14.13.1. Root dans le conteneur, non-root sur l’hôte

Le user namespace permet de mapper l’UID `0` interne vers un UID non privilégié sur l’hôte.

Exemple :

```text
Dans le conteneur :
UID 0

Sur l’hôte :
UID 100000
```

Cela réduit l’impact potentiel d’un conteneur compromis.

---

## 14.13.2. Rootless containers

Les conteneurs rootless utilisent cette logique.

Ils permettent à un utilisateur non-root de lancer des conteneurs.

Avantages :

- pas de démon root obligatoire ;
    
- réduction de l’impact d’une compromission ;
    
- meilleure isolation des droits hôte ;
    
- utile sur postes développeurs et certains serveurs.
    

Limites :

- réseau parfois plus complexe ;
    
- performances ou fonctionnalités différentes ;
    
- problèmes de permissions avec bind mounts ;
    
- dépendance à `/etc/subuid` et `/etc/subgid`;
    
- support variable selon environnements.
    

---

## 14.13.3. Limite de sécurité

Le user namespace réduit certains risques, mais ne les supprime pas.

Il reste des risques liés :

- au noyau partagé ;
    
- aux montages ;
    
- aux capabilities dans le namespace ;
    
- aux erreurs de configuration ;
    
- aux vulnérabilités du runtime ;
    
- aux fichiers exposés depuis l’hôte.
    

Nous devons l’utiliser comme une couche, pas comme une solution unique.

---

# 14.14. Seccomp, capabilities et namespaces : exemple combiné

## 14.14.1. Cas d’un serveur web

Nous avons un serveur web conteneurisé qui écoute sur le port 8080.

Il n’a pas besoin de :

- modifier les interfaces réseau ;
    
- monter des systèmes de fichiers ;
    
- tracer d’autres processus ;
    
- créer des périphériques ;
    
- changer l’heure système ;
    
- charger des modules noyau.
    

Nous pouvons donc réduire fortement ses privilèges.

---

## 14.14.2. Exemple Docker durci

Exemple indicatif :

```bash
docker run --rm \
  --read-only \
  --cap-drop=ALL \
  --security-opt no-new-privileges:true \
  --pids-limit=256 \
  --memory=512m \
  --cpus=1 \
  -p 8080:8080 \
  image-web
```

Ici, nous combinons :

- root filesystem en lecture seule ;
    
- suppression des capabilities ;
    
- interdiction d’acquérir de nouveaux privilèges ;
    
- limite de processus ;
    
- limite mémoire ;
    
- limite CPU ;
    
- exposition explicite du port.
    

Selon l’application, il faut ajouter des volumes temporaires pour `/tmp` ou `/run`.

---

## 14.14.3. Exemple Kubernetes durci

Exemple indicatif :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-secure
spec:
  containers:
    - name: web
      image: image-web
      ports:
        - containerPort: 8080
      securityContext:
        runAsNonRoot: true
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
        seccompProfile:
          type: RuntimeDefault
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "1"
```

Ce n’est pas une recette universelle, mais une bonne base de réflexion.

---

# 14.15. `no-new-privileges`

## 14.15.1. Principe

`no-new-privileges` empêche un processus et ses enfants d’obtenir de nouveaux privilèges via certains mécanismes, par exemple des exécutables setuid.

Avec Docker :

```bash
docker run --security-opt no-new-privileges:true image
```

Dans Kubernetes :

```yaml
securityContext:
  allowPrivilegeEscalation: false
```

Ce mécanisme réduit les possibilités d’élévation de privilèges.

---

## 14.15.2. Pourquoi c’est utile

Même si un fichier setuid est présent dans l’image, le processus ne doit pas pouvoir s’en servir pour acquérir davantage de droits.

C’est une protection simple et efficace pour beaucoup de workloads applicatifs.

---

# 14.16. Périphériques et `/dev`

## 14.16.1. Pourquoi les périphériques sont sensibles

Les fichiers dans `/dev` ne sont pas des fichiers ordinaires.

Ils donnent accès à des périphériques ou interfaces noyau.

Exemples :

```text
/dev/null
/dev/random
/dev/tty
/dev/sda
/dev/kvm
/dev/dri
/dev/fuse
```

Certains sont inoffensifs ou nécessaires. D’autres sont très sensibles.

---

## 14.16.2. Exposer un périphérique précis

Docker permet :

```bash
docker run --device=/dev/dri image
```

Kubernetes permet des mécanismes autour des device plugins, notamment pour les GPU.

Nous préférons exposer un périphérique précis plutôt que tout `/dev`.

---

## 14.16.3. Risque de `/dev/kvm`

Donner accès à `/dev/kvm` permet d’utiliser l’accélération de virtualisation.

Cela peut être nécessaire pour certains cas, mais c’est sensible.

Nous devons comprendre l’impact avant de l’exposer.

---

# 14.17. Sécurité de `/proc`

## 14.17.1. Informations visibles

Nous avons déjà vu que `/proc` peut exposer :

```text
/proc/<PID>/cmdline
/proc/<PID>/environ
/proc/<PID>/fd
/proc/<PID>/maps
/proc/<PID>/mountinfo
/proc/net/*
```

Ces fichiers peuvent révéler des secrets ou des détails d’architecture.

---

## 14.17.2. Secrets en ligne de commande

Mauvaise pratique :

```bash
app --password supersecret
```

Le secret peut apparaître dans :

```bash
ps aux
tr '\0' ' ' < /proc/<PID>/cmdline
```

Nous évitons les secrets en arguments.

---

## 14.17.3. Secrets en variables d’environnement

Les variables d’environnement peuvent être visibles dans :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

selon les permissions et le contexte.

Les secrets en variables d’environnement sont pratiques, mais pas parfaitement invisibles.

Nous devons limiter les accès à `/proc`, utiliser des mécanismes de secrets adaptés et éviter de trop exposer les processus.

---

## 14.17.4. Option `hidepid`

Sur un serveur multi-utilisateurs, nous pouvons monter `/proc` avec :

```text
hidepid=2
```

Exemple dans `/etc/fstab` :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

Cela réduit la visibilité des processus appartenant à d’autres utilisateurs.

---

# 14.18. Bonnes pratiques de durcissement

## 14.18.1. Principe du moindre privilège

Nous donnons au processus uniquement ce dont il a besoin.

Nous évitons :

```text
--privileged
--network=host
--pid=host
--ipc=host
--cap-add=SYS_ADMIN
montage de /
montage de /proc
montage de /sys
montage du socket Docker
```

sauf justification forte.

---

## 14.18.2. Réduire les capabilities

Bonne approche :

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image
```

si l’application a seulement besoin d’écouter sur un port bas.

En Kubernetes :

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE
```

Nous ajoutons seulement les capabilities nécessaires.

---

## 14.18.3. Utiliser un utilisateur non-root

Dockerfile :

```dockerfile
USER 10001
```

Kubernetes :

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
```

Même si les namespaces isolent l’environnement, exécuter l’application en non-root réduit les risques.

---

## 14.18.4. Root filesystem en lecture seule

Docker :

```bash
docker run --read-only image
```

Kubernetes :

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

Nous ajoutons ensuite uniquement les chemins nécessaires en écriture, par exemple :

```text
/tmp
/run
/var/cache/app
```

avec des volumes adaptés.

---

## 14.18.5. Activer seccomp par défaut

En Kubernetes :

```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault
```

Nous évitons :

```yaml
seccompProfile:
  type: Unconfined
```

sauf besoin très spécifique.

---

## 14.18.6. Limiter les ressources

Nous définissons des limites raisonnables :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Nous utilisons aussi une limite de processus lorsque c’est possible.

---

## 14.18.7. Éviter les montages sensibles

Nous évitons de monter :

```text
/
/proc
/sys
/dev
/var/run/docker.sock
/var/lib/docker
/var/lib/kubelet
/etc
/root
```

Si un montage est nécessaire, nous préférons :

- chemin précis ;
    
- lecture seule ;
    
- permissions minimales ;
    
- utilisateur non-root ;
    
- politique AppArmor/SELinux adaptée.
    

---

# 14.19. Étude de cas : conteneur trop permissif

## 14.19.1. Configuration dangereuse

Exemple :

```bash
docker run --rm -it \
  --privileged \
  --network=host \
  --pid=host \
  -v /:/host \
  image
```

Cette configuration cumule plusieurs risques.

Le conteneur :

- est privilégié ;
    
- partage le réseau de l’hôte ;
    
- partage les PID de l’hôte ;
    
- voit toute la racine de l’hôte sous `/host`.
    

Ce n’est presque plus une isolation applicative.

---

## 14.19.2. Analyse

Nous identifions les problèmes :

|Élément|Risque|
|---|---|
|`--privileged`|donne trop de droits|
|`--network=host`|supprime l’isolation réseau|
|`--pid=host`|expose les processus de l’hôte|
|`-v /:/host`|expose tout le système de fichiers|
|absence de limites|risque de consommation excessive|
|root par défaut|impact plus important en cas de compromission|

---

## 14.19.3. Version plus raisonnable

Selon le besoin réel, nous cherchons une version réduite :

```bash
docker run --rm \
  --cap-drop=ALL \
  --security-opt no-new-privileges:true \
  --read-only \
  --memory=512m \
  --cpus=1 \
  -v /chemin/necessaire:/data:ro \
  image
```

Nous ne montons que le chemin nécessaire, en lecture seule.

Nous n’utilisons pas `--privileged`.

Nous ne partageons pas les namespaces de l’hôte sauf besoin démontré.

---

# 14.20. Exercices

## Exercice 1 — Observer les capabilities

Nous exécutons :

```bash
grep Cap /proc/$$/status
```

Puis, si disponible :

```bash
capsh --print
```

Nous répondons :

1. Quelles capabilities effectives voyons-nous ?
    
2. Quelle différence faisons-nous entre `CapPrm` et `CapEff` ?
    
3. Pourquoi les capabilities sont-elles plus fines que le simple couple root/non-root ?
    

---

## Exercice 2 — Comparer root hôte et root conteneur

Si Docker est disponible :

```bash
docker run --rm alpine sh -c 'id; grep Cap /proc/$$/status'
```

Nous répondons :

1. Le processus est-il UID 0 dans le conteneur ?
    
2. A-t-il forcément toutes les capabilities de root ?
    
3. Pourquoi devons-nous distinguer UID et capabilities ?
    

---

## Exercice 3 — Ajouter et retirer des capabilities

Nous comparons :

```bash
docker run --rm alpine sh -c 'grep Cap /proc/$$/status'
docker run --rm --cap-drop=ALL alpine sh -c 'grep Cap /proc/$$/status'
docker run --rm --cap-drop=ALL --cap-add=NET_BIND_SERVICE alpine sh -c 'grep Cap /proc/$$/status'
```

Nous répondons :

1. Que change `--cap-drop=ALL` ?
    
2. Que change `--cap-add=NET_BIND_SERVICE` ?
    
3. Pourquoi cette approche est-elle préférable à `--privileged` ?
    

---

## Exercice 4 — Observer AppArmor ou SELinux

Sur une distribution avec AppArmor :

```bash
cat /proc/$$/attr/current
sudo aa-status
```

Sur une distribution avec SELinux :

```bash
getenforce
ps -eZ | head
ls -Z | head
```

Nous répondons :

1. Quel mécanisme est actif ?
    
2. Le processus courant est-il confiné ?
    
3. Pourquoi ces mécanismes complètent-ils les namespaces ?
    

---

## Exercice 5 — Analyser une configuration Docker dangereuse

Nous analysons :

```bash
docker run --privileged --network=host --pid=host -v /:/host image
```

Nous répondons :

1. Quels namespaces sont partagés avec l’hôte ?
    
2. Quels montages sont dangereux ?
    
3. Pourquoi `--privileged` augmente-t-il le risque ?
    
4. Comment proposer une configuration plus sûre ?
    

---

## Exercice 6 — Sécurité de `/proc`

Nous lançons un processus avec un argument fictif :

```bash
sleep 300 --fake-secret 2>/dev/null
```

Cette commande n’est pas forcément acceptée par `sleep` selon les systèmes. Nous pouvons plutôt lancer :

```bash
bash -c 'sleep 300' secret-fictif &
```

Puis :

```bash
ps aux | grep secret
tr '\0' ' ' < /proc/<PID>/cmdline
```

Nous répondons :

1. Les arguments sont-ils visibles ?
    
2. Pourquoi ne devons-nous pas passer de secrets en ligne de commande ?
    
3. Quels autres fichiers de `/proc` peuvent révéler des informations ?
    

---

## Exercice 7 — Kubernetes securityContext

Nous analysons :

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
  seccompProfile:
    type: RuntimeDefault
```

Nous répondons :

1. Quel est le rôle de `runAsNonRoot` ?
    
2. Quel est le rôle de `allowPrivilegeEscalation: false` ?
    
3. Quel est le rôle de `readOnlyRootFilesystem` ?
    
4. Pourquoi supprimons-nous toutes les capabilities ?
    
5. Que fait `RuntimeDefault` pour seccomp ?
    

---

# 14.21. Ce que nous devons retenir

Nous retenons les points suivants :

1. Les namespaces isolent des vues, mais ne suffisent pas à sécuriser un système.
    
2. Les conteneurs partagent le noyau de l’hôte.
    
3. Les capabilities découpent les privilèges root en droits plus fins.
    
4. Être UID 0 ne signifie pas posséder toutes les capabilities.
    
5. `CAP_SYS_ADMIN` est particulièrement sensible.
    
6. `--privileged` réduit fortement l’isolation.
    
7. Seccomp limite les appels système disponibles.
    
8. AppArmor et SELinux ajoutent des politiques de contrôle d’accès obligatoire.
    
9. Les montages de `/`, `/proc`, `/sys`, `/dev` et du socket Docker sont très sensibles.
    
10. Partager `pid`, `network` ou `ipc` avec l’hôte réduit fortement l’isolation.
    
11. Le user namespace permet de réduire l’impact de root dans le conteneur.
    
12. Les conteneurs rootless améliorent la sécurité, mais ne suppriment pas tous les risques.
    
13. Le principe du moindre privilège doit guider la configuration.
    
14. Nous préférons supprimer toutes les capabilities puis ajouter seulement celles nécessaires.
    
15. Nous évitons `--privileged` sauf cas exceptionnel.
    
16. Nous utilisons des limites cgroups pour réduire l’impact opérationnel.
    
17. Nous devons penser ensemble : namespaces, cgroups, capabilities, seccomp, AppArmor/SELinux, montages et utilisateur d’exécution.
    

---

# Conclusion du chapitre 14

Nous avons étudié la sécurité autour des namespaces.

Nous savons maintenant que les namespaces et les cgroups sont nécessaires, mais pas suffisants. Les namespaces isolent les vues, les cgroups limitent les ressources, mais les droits réels d’un processus dépendent aussi des capabilities, de seccomp, d’AppArmor, de SELinux, des montages, des périphériques exposés et du mode d’exécution.

Nous avons compris pourquoi un conteneur privilégié, un montage de `/`, un accès au socket Docker, `--network=host`, `--pid=host` ou `CAP_SYS_ADMIN` peuvent réduire fortement l’isolation.

Nous retenons surtout une méthode professionnelle : nous ne donnons jamais des privilèges “pour que ça marche”. Nous identifions le besoin exact, puis nous accordons le minimum nécessaire.

Dans le chapitre suivant, nous étudions comment Docker et les runtimes de conteneurs assemblent concrètement namespaces, cgroups, capabilities, montages et processus pour construire un conteneur.

# Chapitre 15 — Des namespaces aux conteneurs

## Objectifs du chapitre

Dans ce chapitre, nous relions les namespaces Linux au fonctionnement concret des conteneurs.

Jusqu’ici, nous avons étudié les briques séparément :

```text
namespace PID
namespace mount
namespace network
namespace UTS
namespace IPC
namespace user
namespace cgroup
namespace time
cgroups
capabilities
seccomp
AppArmor / SELinux
```

Nous allons maintenant comprendre comment un runtime de conteneur assemble ces mécanismes pour lancer une application dans un environnement isolé.

À la fin de ce chapitre, nous savons :

- expliquer ce qu’est réellement un conteneur Linux ;
    
- comprendre comment Docker combine les namespaces ;
    
- distinguer image, root filesystem et conteneur ;
    
- comprendre le rôle du runtime OCI ;
    
- expliquer le rôle de `runc`, `containerd` et Docker ;
    
- comprendre l’importance de l’entrypoint et du PID 1 ;
    
- comprendre les volumes et bind mounts ;
    
- comprendre le réseau bridge Docker ;
    
- observer un conteneur depuis l’hôte avec `/proc`, `lsns` et `nsenter`.
    

---

# 15.1. Un conteneur n’est pas une machine virtuelle

## 15.1.1. Rappel fondamental

Un conteneur Linux n’exécute pas son propre noyau.

Il utilise le noyau de l’hôte, mais avec des vues isolées.

Nous devons retenir :

```text
Machine virtuelle : noyau invité séparé.
Conteneur Linux   : noyau hôte partagé.
```

Dans une machine virtuelle, nous avons généralement :

```text
application
système invité
noyau invité
matériel virtuel
hyperviseur
noyau hôte
matériel réel
```

Dans un conteneur, nous avons plutôt :

```text
application
rootfs isolé
namespaces
cgroups
noyau Linux hôte
matériel réel
```

---

## 15.1.2. Pourquoi les conteneurs sont plus légers

Les conteneurs sont plus légers parce qu’ils ne démarrent pas un système d’exploitation complet avec son propre noyau.

Ils lancent simplement un ou plusieurs processus sur le noyau de l’hôte.

Ce qui donne l’impression d’un système séparé vient de la combinaison :

```text
namespaces
cgroups
root filesystem
capabilities
seccomp
profils de sécurité
configuration réseau
```

Nous ne devons donc pas voir Docker comme une technologie magique, mais comme un orchestrateur de mécanismes Linux.

---

# 15.2. Qu’est-ce qu’un conteneur Linux ?

## 15.2.1. Définition pratique

Un conteneur Linux est un ou plusieurs processus lancés avec :

- des namespaces isolés ;
    
- des limites cgroups ;
    
- un root filesystem spécifique ;
    
- des capabilities contrôlées ;
    
- des montages préparés ;
    
- une configuration réseau ;
    
- une politique de sécurité ;
    
- une commande de démarrage.
    

Autrement dit :

```text
Un conteneur est un processus isolé et configuré.
```

Il ne faut pas le confondre avec l’image.

---

## 15.2.2. Image et conteneur

Une image est un modèle immuable ou quasi immuable contenant :

- fichiers ;
    
- bibliothèques ;
    
- binaires ;
    
- métadonnées ;
    
- configuration par défaut ;
    
- entrypoint ;
    
- commande par défaut.
    

Un conteneur est une instance en cours d’exécution créée à partir d’une image.

Nous pouvons résumer :

```text
Image      = modèle de fichiers et de configuration.
Conteneur  = processus lancé à partir de cette image.
```

Exemple :

```bash
docker pull alpine
docker run --rm -it alpine sh
```

`alpine` est l’image.

Le shell lancé est un processus dans un conteneur créé à partir de cette image.

---

# 15.3. Ce que fait Docker au lancement d’un conteneur

## 15.3.1. Commande simple

Quand nous lançons :

```bash
docker run --rm -it alpine sh
```

Docker effectue beaucoup d’opérations.

Il ne fait pas seulement :

```text
lancer /bin/sh
```

Il prépare un environnement d’exécution complet.

---

## 15.3.2. Étapes conceptuelles

Docker ou son runtime prépare notamment :

1. le root filesystem issu de l’image ;
    
2. un namespace mount ;
    
3. un namespace PID ;
    
4. un namespace network ;
    
5. un namespace UTS ;
    
6. un namespace IPC ;
    
7. parfois un namespace user ;
    
8. une configuration cgroup ;
    
9. des capabilities ;
    
10. un profil seccomp ;
    
11. éventuellement un profil AppArmor ou SELinux ;
    
12. les fichiers `/etc/hosts`, `/etc/resolv.conf`, `/etc/hostname` ;
    
13. les volumes et bind mounts ;
    
14. l’interface réseau virtuelle ;
    
15. le processus initial.
    

Nous voyons que Docker assemble les chapitres précédents.

---

# 15.4. Les namespaces utilisés par un conteneur

## 15.4.1. Namespace PID

Le conteneur a souvent son propre namespace PID.

Dans le conteneur :

```sh
ps
```

nous pouvons voir :

```text
PID   USER     TIME  COMMAND
1     root     0:00  sh
```

Mais sur l’hôte, le même processus a un autre PID.

Nous pouvons vérifier :

```bash
docker run --rm -d --name pid-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' pid-demo)

ps -p "$pid" -o pid,ppid,comm,args
sudo grep NSpid /proc/$pid/status

docker exec pid-demo ps
docker rm -f pid-demo
```

Le namespace PID permet au processus principal d’être PID 1 dans le conteneur.

---

## 15.4.2. Namespace mount

Le conteneur a son propre namespace mount.

Il voit comme `/` le root filesystem de l’image.

Dans le conteneur :

```sh
ls /
```

nous voyons la racine de l’image, par exemple :

```text
bin
dev
etc
home
lib
proc
root
sys
tmp
usr
var
```

Ce n’est pas la racine de l’hôte.

Docker prépare cette vue avec :

- rootfs de l’image ;
    
- OverlayFS ou autre backend ;
    
- montage de `/proc` ;
    
- montage de `/sys` ;
    
- préparation de `/dev` ;
    
- volumes ;
    
- bind mounts.
    

---

## 15.4.3. Namespace network

Le conteneur a généralement son propre namespace réseau.

Dans le conteneur :

```sh
ip addr
```

nous voyons souvent :

```text
lo
eth0
```

`eth0` est souvent une extrémité d’une paire `veth`.

L’autre extrémité est côté hôte, connectée au bridge Docker, souvent `docker0`.

---

## 15.4.4. Namespace UTS

Docker donne souvent un hostname propre au conteneur.

Exemple :

```bash
docker run --rm alpine hostname
```

Sortie possible :

```text
a1b2c3d4e5f6
```

Nous pouvons définir le hostname :

```bash
docker run --rm --hostname mon-conteneur alpine hostname
```

Le namespace UTS permet cette isolation du hostname.

---

## 15.4.5. Namespace IPC

Docker crée généralement un namespace IPC propre au conteneur.

Nous pouvons comparer :

```bash
docker run --rm -d --name ipc-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' ipc-demo)

sudo readlink /proc/$pid/ns/ipc
readlink /proc/1/ns/ipc

docker rm -f ipc-demo
```

Si les valeurs diffèrent, le conteneur a son propre namespace IPC.

---

## 15.4.6. Namespace user

Le namespace user dépend de la configuration.

Dans un conteneur Docker classique, le conteneur peut être root à l’intérieur, mais pas nécessairement avec un mapping user namespace complet.

Avec les conteneurs rootless, ou certaines options de remapping, l’UID `0` interne peut correspondre à un UID non privilégié sur l’hôte.

Nous observons avec :

```bash
cat /proc/<PID>/uid_map
cat /proc/<PID>/gid_map
```

---

## 15.4.7. Namespace cgroup

Le namespace cgroup peut permettre au conteneur de voir une vue simplifiée de son rattachement cgroup.

Dans le conteneur :

```sh
cat /proc/self/cgroup
```

nous pouvons voir une vue plus simple que depuis l’hôte.

Mais les limites réelles sont appliquées par les cgroups eux-mêmes.

---

# 15.5. Le root filesystem du conteneur

## 15.5.1. Définition

Le root filesystem, ou rootfs, est l’arborescence vue comme `/` par le processus du conteneur.

Il contient les fichiers nécessaires à l’exécution de l’application :

```text
/bin
/etc
/lib
/usr
/app
/var
/tmp
```

Dans Docker, ce rootfs provient de l’image.

---

## 15.5.2. Images en couches

Les images Docker sont composées de couches.

Exemple conceptuel :

```text
couche 1 : base Alpine
couche 2 : installation de paquets
couche 3 : copie de l’application
couche 4 : configuration
```

Ces couches sont généralement en lecture seule.

Quand nous lançons un conteneur, Docker ajoute une couche writable propre au conteneur.

Nous obtenons :

```text
couches image en lecture seule
+
couche writable du conteneur
=
rootfs visible dans le conteneur
```

---

## 15.5.3. OverlayFS

Sur beaucoup de systèmes, Docker utilise OverlayFS.

Nous pouvons conceptualiser :

```text
lowerdir : couches en lecture seule
upperdir : couche writable du conteneur
merged   : vue finale présentée au conteneur
```

Le namespace mount présente ensuite `merged` comme racine `/` du conteneur.

---

## 15.5.4. Conséquence pratique

Si nous créons un fichier dans un conteneur :

```sh
echo test > /tmp/fichier
```

ce fichier est écrit dans la couche writable du conteneur.

Si nous supprimons le conteneur, cette couche disparaît, sauf si le fichier est dans un volume.

Nous retenons :

```text
Le conteneur est éphémère.
Les volumes servent à conserver des données.
```

---

# 15.6. Volumes et bind mounts

## 15.6.1. Pourquoi les volumes existent

Un conteneur est souvent jetable.

Mais certaines données doivent persister :

- base de données ;
    
- fichiers uploadés ;
    
- cache durable ;
    
- configuration ;
    
- logs selon architecture ;
    
- artefacts produits.
    

Nous utilisons alors des volumes ou bind mounts.

---

## 15.6.2. Bind mount

Un bind mount monte un chemin de l’hôte dans le conteneur.

Exemple :

```bash
docker run --rm -it -v "$PWD":/app alpine sh
```

Dans le conteneur :

```sh
ls /app
```

Nous voyons le répertoire courant de l’hôte.

Le conteneur et l’hôte accèdent au même contenu.

---

## 15.6.3. Volume Docker

Un volume Docker est géré par Docker.

Exemple :

```bash
docker volume create data-demo
docker run --rm -it -v data-demo:/data alpine sh
```

Les données sont stockées dans l’espace géré par Docker.

Nous pouvons lister les volumes :

```bash
docker volume ls
```

Et inspecter :

```bash
docker volume inspect data-demo
```

---

## 15.6.4. Lecture seule

Nous pouvons monter en lecture seule :

```bash
docker run --rm -it -v "$PWD":/app:ro alpine sh
```

Dans le conteneur, l’écriture dans `/app` doit échouer.

C’est une bonne pratique lorsque le conteneur n’a besoin que de lire des fichiers.

---

## 15.6.5. Risques

Les montages peuvent casser l’isolation.

Montages dangereux :

```bash
-v /:/host
-v /proc:/host/proc
-v /sys:/host/sys
-v /dev:/host/dev
-v /var/run/docker.sock:/var/run/docker.sock
```

Nous ne les utilisons pas sans justification forte.

---

# 15.7. Le réseau Docker bridge

## 15.7.1. Bridge par défaut

Docker crée souvent un bridge nommé :

```text
docker0
```

Nous pouvons l’observer :

```bash
ip addr show docker0
```

Les conteneurs connectés au réseau bridge par défaut reçoivent une interface `eth0` dans leur namespace réseau.

Côté hôte, une paire `veth` connecte le conteneur au bridge.

---

## 15.7.2. Schéma conceptuel

```text
conteneur
  eth0
   |
 paire veth
   |
hôte
  vethXXXX
   |
 docker0
   |
 réseau extérieur via routage/NAT
```

Le namespace réseau isole la pile réseau du conteneur.

Le bridge permet la communication entre conteneurs et avec l’hôte.

Le NAT permet souvent la sortie vers Internet.

---

## 15.7.3. Ports publiés

Quand nous écrivons :

```bash
docker run -p 8080:80 image
```

nous demandons :

```text
port 8080 de l’hôte → port 80 du conteneur
```

Le conteneur peut écouter sur `80` dans son namespace réseau.

L’hôte expose `8080` et redirige vers le conteneur.

Nous distinguons donc :

```text
port interne du conteneur
port exposé sur l’hôte
```

---

## 15.7.4. Mode host network

Avec :

```bash
docker run --network=host image
```

le conteneur partage le namespace réseau de l’hôte.

Il n’a plus d’isolation réseau classique.

Conséquences :

- mêmes interfaces que l’hôte ;
    
- mêmes ports ;
    
- conflits possibles ;
    
- exposition plus directe ;
    
- diagnostic différent.
    

Nous utilisons ce mode avec prudence.

---

# 15.8. Entrypoint, command et PID 1

## 15.8.1. Le processus initial

Un conteneur démarre avec un processus initial.

Ce processus devient généralement PID 1 dans le namespace PID du conteneur.

Il peut venir de :

```text
ENTRYPOINT
CMD
commande passée à docker run
```

Exemple :

```bash
docker run --rm alpine sleep 1000
```

Ici, `sleep 1000` devient le processus principal du conteneur.

---

## 15.8.2. `ENTRYPOINT` et `CMD`

Dans un Dockerfile :

```dockerfile
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

La commande finale est :

```text
python3 app.py
```

Autre exemple :

```dockerfile
ENTRYPOINT ["./server"]
CMD ["--port", "8080"]
```

`ENTRYPOINT` définit souvent le binaire principal.

`CMD` définit les arguments par défaut.

---

## 15.8.3. PID 1 et signaux

Le processus PID 1 doit gérer correctement les signaux.

Quand nous faisons :

```bash
docker stop <container>
```

Docker envoie d’abord généralement `SIGTERM`.

Si le processus ne s’arrête pas, Docker finit par envoyer `SIGKILL`.

Une application mal conçue comme PID 1 peut ignorer ou mal gérer `SIGTERM`.

---

## 15.8.4. Processus zombies

Le PID 1 doit aussi récolter les processus enfants terminés.

Si l’application lance des sous-processus et ne les récolte pas, des zombies peuvent s’accumuler.

Nous pouvons utiliser un init minimal :

```bash
docker run --init image
```

Docker ajoute alors un petit init, souvent `tini`, qui aide à gérer les signaux et les zombies.

---

# 15.9. Rôle de `runc`, containerd et Docker

## 15.9.1. Docker n’est pas seul

Quand nous utilisons Docker, plusieurs composants interviennent.

Schéma simplifié :

```text
docker CLI
   |
dockerd
   |
containerd
   |
runc
   |
noyau Linux
```

Chaque composant a un rôle.

---

## 15.9.2. Docker CLI

La commande :

```bash
docker run ...
```

est envoyée au démon Docker.

Le client Docker n’est qu’une interface de commande.

---

## 15.9.3. `dockerd`

Le démon Docker gère :

- images ;
    
- conteneurs ;
    
- réseaux ;
    
- volumes ;
    
- API Docker ;
    
- intégration avec containerd ;
    
- configuration globale Docker.
    

---

## 15.9.4. `containerd`

`containerd` est un runtime de plus bas niveau qui gère le cycle de vie des conteneurs :

- création ;
    
- démarrage ;
    
- arrêt ;
    
- supervision ;
    
- images ;
    
- snapshots ;
    
- interaction avec le runtime OCI.
    

Kubernetes utilise souvent `containerd` directement comme runtime CRI.

---

## 15.9.5. `runc`

`runc` est un runtime OCI bas niveau.

Il est responsable de créer concrètement le conteneur au niveau Linux :

- namespaces ;
    
- cgroups ;
    
- capabilities ;
    
- montages ;
    
- rootfs ;
    
- processus initial.
    

`runc` applique une configuration conforme à la spécification OCI.

---

# 15.10. OCI : Open Container Initiative

## 15.10.1. Pourquoi une spécification ?

Avant la standardisation, chaque outil pouvait définir ses propres formats et comportements.

L’OCI, Open Container Initiative, définit notamment :

- une spécification d’image ;
    
- une spécification de runtime ;
    
- un format de configuration.
    

Cela permet à différents outils d’être compatibles.

---

## 15.10.2. Runtime OCI

Un runtime OCI reçoit une configuration et lance le conteneur selon cette configuration.

Cette configuration décrit notamment :

- commande à exécuter ;
    
- environnement ;
    
- rootfs ;
    
- montages ;
    
- namespaces ;
    
- capabilities ;
    
- cgroups ;
    
- utilisateur ;
    
- limites ;
    
- hooks éventuels.
    

`runc` est l’implémentation de référence historique.

---

# 15.11. Observer un conteneur depuis l’hôte

## 15.11.1. Lancer un conteneur de test

Nous lançons :

```bash
docker run --rm -d --name ns-demo alpine sleep 1000
```

Nous récupérons le PID hôte :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' ns-demo)
echo "$pid"
```

---

## 15.11.2. Observer ses namespaces

Depuis l’hôte :

```bash
sudo ls -l /proc/$pid/ns
```

Nous comparons avec l’hôte :

```bash
ls -l /proc/1/ns
```

Nous pouvons voir quels namespaces sont différents.

---

## 15.11.3. Observer les PID

```bash
sudo grep NSpid /proc/$pid/status
```

Exemple :

```text
NSpid:  24531  1
```

Le même processus est PID `24531` sur l’hôte et PID `1` dans le conteneur.

---

## 15.11.4. Observer les cgroups

```bash
sudo cat /proc/$pid/cgroup
```

Et dans le conteneur :

```bash
docker exec ns-demo cat /proc/self/cgroup
```

Nous comparons la vue depuis l’hôte et la vue interne.

---

## 15.11.5. Observer les montages

```bash
sudo cat /proc/$pid/mountinfo | head
```

Ou :

```bash
sudo nsenter --target "$pid" --mount findmnt | head
```

Nous voyons les montages du conteneur.

---

## 15.11.6. Observer le réseau

```bash
sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route
```

Si `ip` n’est pas disponible dans le conteneur, nous pouvons l’utiliser depuis l’hôte avec `nsenter`.

---

## 15.11.7. Nettoyage

```bash
docker rm -f ns-demo
```

---

# 15.12. `docker exec` et `nsenter`

## 15.12.1. `docker exec`

Quand nous utilisons :

```bash
docker exec -it ns-demo sh
```

Docker lance un nouveau processus dans le conteneur existant.

Ce nouveau processus rejoint les namespaces du conteneur.

Il partage donc généralement :

- namespace PID ;
    
- namespace mount ;
    
- namespace network ;
    
- namespace UTS ;
    
- namespace IPC ;
    
- cgroups du conteneur.
    

---

## 15.12.2. `nsenter`

Avec `nsenter`, nous pouvons entrer dans les namespaces du processus principal depuis l’hôte :

```bash
sudo nsenter --target "$pid" \
  --mount \
  --uts \
  --ipc \
  --net \
  --pid \
  --fork \
  sh
```

Cela est utile lorsque :

- l’image ne contient pas de shell ;
    
- `docker exec` ne fonctionne pas ;
    
- nous voulons utiliser des outils de l’hôte ;
    
- nous voulons diagnostiquer un namespace précis.
    

---

## 15.12.3. Différence pratique

`docker exec` passe par Docker.

`nsenter` passe directement par les namespaces exposés dans `/proc/<PID>/ns`.

Les deux approches sont utiles, mais elles n’ont pas exactement le même niveau d’abstraction.

---

# 15.13. Variables d’environnement

## 15.13.1. Définition

Un conteneur reçoit souvent des variables d’environnement.

Exemple :

```bash
docker run --rm -e APP_ENV=production alpine env
```

Elles peuvent configurer :

- environnement ;
    
- URL de base de données ;
    
- chemins ;
    
- options applicatives ;
    
- mode debug ;
    
- secrets parfois, avec prudence.
    

---

## 15.13.2. Observation

Dans un conteneur :

```sh
env
```

Depuis l’hôte, selon les permissions :

```bash
sudo tr '\0' '\n' < /proc/$pid/environ
```

Nous devons nous souvenir que les variables d’environnement peuvent être visibles via `/proc`.

---

## 15.13.3. Prudence avec les secrets

Passer des secrets en variables d’environnement est courant, mais pas parfait.

Risques :

- visibilité via `/proc` ;
    
- fuite dans des dumps ;
    
- affichage dans logs ;
    
- exposition par erreur dans une page debug ;
    
- transmission à des processus enfants.
    

En production, nous préférons des mécanismes de secrets adaptés et des permissions strictes.

---

# 15.14. Capabilities et sécurité dans un conteneur

## 15.14.1. Root dans le conteneur

Dans un conteneur :

```sh
id
```

peut afficher :

```text
uid=0(root) gid=0(root)
```

Mais cela ne signifie pas que le processus a tous les droits sur l’hôte.

Nous devons vérifier :

```sh
grep Cap /proc/$$/status
```

---

## 15.14.2. Réduire les capabilities

Bonne pratique :

```bash
docker run --cap-drop=ALL image
```

Puis ajouter seulement ce qui est nécessaire :

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image
```

Nous appliquons le principe du moindre privilège.

---

## 15.14.3. Éviter `--privileged`

`--privileged` donne beaucoup trop de droits.

```bash
docker run --privileged image
```

Nous l’évitons sauf cas très spécifique.

Nous préférons identifier le besoin précis :

```text
capability manquante ?
périphérique précis ?
montage précis ?
profil seccomp trop strict ?
```

---

# 15.15. Seccomp, AppArmor et SELinux dans les conteneurs

## 15.15.1. Seccomp

Docker applique généralement un profil seccomp par défaut.

Nous évitons :

```bash
docker run --security-opt seccomp=unconfined image
```

sauf diagnostic ou besoin justifié.

---

## 15.15.2. AppArmor

Sur Ubuntu/Debian, Docker peut utiliser un profil AppArmor comme :

```text
docker-default
```

Nous pouvons observer :

```bash
cat /proc/<PID>/attr/current
```

---

## 15.15.3. SELinux

Sur Fedora, RHEL, Rocky, AlmaLinux ou CentOS Stream, SELinux peut confiner les conteneurs.

Un problème de permission peut venir de SELinux même si les permissions Unix semblent correctes.

Nous vérifions :

```bash
getenforce
ls -Z
ps -eZ
```

---

# 15.16. Construire mentalement un conteneur

## 15.16.1. Les briques

Pour construire un conteneur, nous devons assembler :

```text
rootfs
namespace mount
namespace PID
namespace UTS
namespace IPC
namespace network
namespace user éventuellement
cgroups
capabilities
seccomp
AppArmor/SELinux
processus initial
```

Ce n’est pas une seule technologie.

C’est une composition.

---

## 15.16.2. Mini-conteneur conceptuel

À la main, nous pourrions faire :

```bash
unshare --fork --pid --mount --uts --ipc bash
```

Puis :

```bash
mount --make-rprivate /
mount -t proc proc /proc
hostname mini-container
```

Mais il manquerait encore :

- un rootfs propre ;
    
- un pivot_root ou chroot propre ;
    
- un réseau isolé configuré ;
    
- des cgroups ;
    
- des capabilities réduites ;
    
- une politique seccomp ;
    
- une gestion correcte du PID 1 ;
    
- un nettoyage fiable.
    

Docker automatise tout cela.

---

# 15.17. Exemple d’analyse complète d’un conteneur

## 15.17.1. Lancement

Nous lançons :

```bash
docker run --rm -d \
  --name analyse-demo \
  --hostname demo \
  --memory=256m \
  --pids-limit=128 \
  alpine sleep 1000
```

Nous récupérons le PID :

```bash
pid=$(docker inspect --format '{{.State.Pid}}' analyse-demo)
```

---

## 15.17.2. Namespaces

```bash
sudo ls -l /proc/$pid/ns
```

Nous comparons avec :

```bash
ls -l /proc/1/ns
```

Nous identifions les namespaces différents.

---

## 15.17.3. PID

```bash
sudo grep NSpid /proc/$pid/status
docker exec analyse-demo ps
```

Nous observons le PID hôte et le PID interne.

---

## 15.17.4. Hostname

```bash
docker exec analyse-demo hostname
sudo nsenter --target "$pid" --uts hostname
```

Nous vérifions le namespace UTS.

---

## 15.17.5. Réseau

```bash
sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route
```

Nous observons la pile réseau du conteneur.

---

## 15.17.6. Montages

```bash
sudo nsenter --target "$pid" --mount findmnt | head
sudo cat /proc/$pid/mountinfo | head
```

Nous observons le rootfs et les montages spéciaux.

---

## 15.17.7. Cgroups

```bash
docker exec analyse-demo sh -c 'cat /proc/self/cgroup'
docker exec analyse-demo sh -c 'cat /sys/fs/cgroup/memory.max 2>/dev/null || true'
docker exec analyse-demo sh -c 'cat /sys/fs/cgroup/pids.max 2>/dev/null || true'
```

Nous vérifions les limites.

---

## 15.17.8. Capabilities

```bash
docker exec analyse-demo sh -c 'grep Cap /proc/$$/status'
```

Nous observons les capabilities.

---

## 15.17.9. Nettoyage

```bash
docker rm -f analyse-demo
```

---

# 15.18. Pièges classiques

## 15.18.1. Confondre image et conteneur

Une image n’est pas un processus.

Un conteneur est une instance lancée à partir d’une image.

Nous pouvons avoir plusieurs conteneurs basés sur la même image.

---

## 15.18.2. Croire que le conteneur a son propre noyau

Dans un conteneur :

```sh
uname -a
```

affiche le noyau de l’hôte.

Le conteneur n’a pas son propre noyau.

---

## 15.18.3. Oublier le rôle du PID 1

Si l’application est PID 1, elle doit gérer :

- signaux ;
    
- arrêt propre ;
    
- processus enfants ;
    
- zombies.
    

Sinon, nous utilisons un init minimal :

```bash
docker run --init ...
```

---

## 15.18.4. Confondre `EXPOSE` et publication de port

Dans un Dockerfile :

```dockerfile
EXPOSE 8080
```

documente le port prévu.

Mais cela ne publie pas automatiquement le port sur l’hôte.

Pour publier :

```bash
docker run -p 8080:8080 image
```

---

## 15.18.5. Croire qu’un volume est une copie

Un bind mount n’est pas une copie.

Si nous montons :

```bash
-v "$PWD":/app
```

le conteneur et l’hôte voient le même contenu.

Une modification dans le conteneur peut modifier l’hôte.

---

## 15.18.6. Utiliser `--privileged` pour résoudre trop vite

Si quelque chose ne marche pas, nous ne lançons pas directement :

```bash
docker run --privileged ...
```

Nous identifions le mécanisme en cause :

- permissions fichiers ;
    
- UID/GID ;
    
- capability manquante ;
    
- seccomp ;
    
- AppArmor/SELinux ;
    
- montage absent ;
    
- périphérique nécessaire ;
    
- cgroup trop restrictif.
    

---

# 15.19. Exercices

## Exercice 1 — Observer les namespaces d’un conteneur

Nous lançons :

```bash
docker run --rm -d --name ns-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' ns-demo)

sudo ls -l /proc/$pid/ns
ls -l /proc/1/ns

docker rm -f ns-demo
```

Nous répondons :

1. Quels namespaces sont différents de ceux de l’hôte ?
    
2. Quels namespaces sont éventuellement partagés ?
    
3. Pourquoi un conteneur a-t-il besoin de plusieurs namespaces ?
    

---

## Exercice 2 — Observer le PID interne et externe

Nous lançons :

```bash
docker run --rm -d --name pid-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' pid-demo)

ps -p "$pid" -o pid,ppid,comm,args
sudo grep NSpid /proc/$pid/status
docker exec pid-demo ps

docker rm -f pid-demo
```

Nous répondons :

1. Quel est le PID du processus sur l’hôte ?
    
2. Quel est son PID dans le conteneur ?
    
3. Pourquoi les deux valeurs diffèrent-elles ?
    

---

## Exercice 3 — Observer le réseau du conteneur

Nous lançons :

```bash
docker run --rm -d --name net-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' net-demo)

sudo nsenter --target "$pid" --net ip addr
sudo nsenter --target "$pid" --net ip route

docker rm -f net-demo
```

Nous répondons :

1. Quelles interfaces le conteneur voit-il ?
    
2. Quelle est sa route par défaut ?
    
3. Quel rôle joue le bridge Docker ?
    

---

## Exercice 4 — Observer les montages du conteneur

Nous lançons :

```bash
docker run --rm -d --name mnt-demo alpine sleep 1000
pid=$(docker inspect --format '{{.State.Pid}}' mnt-demo)

sudo nsenter --target "$pid" --mount findmnt | head
sudo cat /proc/$pid/mountinfo | head

docker rm -f mnt-demo
```

Nous répondons :

1. Que représente `/` dans le conteneur ?
    
2. Voyons-nous `/proc`, `/sys` et `/dev` ?
    
3. Pourquoi le namespace mount est-il fondamental ?
    

---

## Exercice 5 — Comparer volume et couche writable

Nous lançons :

```bash
mkdir -p /tmp/docker-volume-test
echo "depuis hote" > /tmp/docker-volume-test/fichier.txt

docker run --rm -it -v /tmp/docker-volume-test:/data alpine sh
```

Dans le conteneur :

```sh
cat /data/fichier.txt
echo "depuis conteneur" > /data/autre.txt
exit
```

Sur l’hôte :

```bash
ls -l /tmp/docker-volume-test
cat /tmp/docker-volume-test/autre.txt
```

Nous répondons :

1. Le fichier créé dans le conteneur est-il visible sur l’hôte ?
    
2. Pourquoi ?
    
3. Quelle différence avec un fichier créé hors volume dans le conteneur ?
    

---

## Exercice 6 — Observer les limites cgroups

Nous lançons :

```bash
docker run --rm -d \
  --name limits-demo \
  --memory=256m \
  --pids-limit=64 \
  alpine sleep 1000

docker exec limits-demo sh -c 'cat /sys/fs/cgroup/memory.max 2>/dev/null || true'
docker exec limits-demo sh -c 'cat /sys/fs/cgroup/pids.max 2>/dev/null || true'

docker rm -f limits-demo
```

Nous répondons :

1. Quelle limite mémoire est visible ?
    
2. Quelle limite de processus est visible ?
    
3. Quel mécanisme applique ces limites ?
    

---

## Exercice 7 — Analyser les capabilities

Nous comparons :

```bash
docker run --rm alpine sh -c 'grep Cap /proc/$$/status'
docker run --rm --cap-drop=ALL alpine sh -c 'grep Cap /proc/$$/status'
```

Nous répondons :

1. Les capabilities changent-elles ?
    
2. Pourquoi est-ce important ?
    
3. Pourquoi être root dans le conteneur ne suffit-il pas à comprendre les droits ?
    

---

# 15.20. Ce que nous devons retenir

Nous retenons les points suivants :

1. Un conteneur Linux est un processus isolé, pas une machine virtuelle.
    
2. Le conteneur partage le noyau de l’hôte.
    
3. Docker assemble des namespaces, cgroups, capabilities, montages et politiques de sécurité.
    
4. Une image est un modèle, un conteneur est une instance en cours d’exécution.
    
5. Le rootfs d’un conteneur vient des couches de l’image et d’une couche writable.
    
6. Le namespace mount présente ce rootfs comme `/`.
    
7. Le namespace PID permet au processus principal d’être PID 1 dans le conteneur.
    
8. Le PID 1 doit gérer les signaux et les processus enfants.
    
9. Le namespace network donne au conteneur sa propre pile réseau.
    
10. Le réseau bridge Docker repose sur des paires `veth`, un bridge et souvent du NAT.
    
11. Les volumes et bind mounts ajoutent des montages dans le conteneur.
    
12. Les bind mounts peuvent exposer l’hôte.
    
13. Les cgroups appliquent les limites CPU, mémoire et pids.
    
14. Les capabilities contrôlent les privilèges fins.
    
15. Seccomp, AppArmor et SELinux ajoutent des couches de sécurité.
    
16. `runc` crée concrètement les namespaces, montages, cgroups et processus.
    
17. `containerd` gère le cycle de vie des conteneurs à plus bas niveau.
    
18. Docker fournit une interface plus haut niveau.
    
19. `docker exec` lance un processus dans les namespaces du conteneur.
    
20. `nsenter` permet d’entrer directement dans les namespaces depuis l’hôte.
    
21. Pour diagnostiquer un conteneur, nous croisons `/proc`, `nsenter`, cgroups, montages, réseau et capabilities.
    

---

# Conclusion du chapitre 15

Nous avons relié les namespaces au fonctionnement concret des conteneurs.

Nous savons maintenant qu’un conteneur n’est pas une machine virtuelle, mais un processus lancé dans un environnement soigneusement construit. Docker, Podman, containerd ou `runc` assemblent des namespaces, des cgroups, des montages, des capabilities et des politiques de sécurité pour produire cette isolation.

Nous avons aussi compris la différence entre image, root filesystem et conteneur. L’image fournit les fichiers, le namespace mount présente ces fichiers comme racine, le namespace PID donne une vue de processus isolée, le namespace network fournit une pile réseau séparée, et les cgroups limitent les ressources.

Nous retenons surtout qu’un conteneur est une composition de mécanismes Linux. Pour diagnostiquer ou sécuriser un conteneur, nous devons donc raisonner couche par couche : processus, montages, réseau, utilisateurs, cgroups, capabilities et sécurité.

Dans le chapitre suivant, nous étudions les namespaces dans Kubernetes, où ces mêmes mécanismes sont utilisés à l’échelle de pods, de nœuds, de workloads et de clusters.

# Chapitre 16 — Namespaces dans Kubernetes

## Objectifs du chapitre

Dans ce chapitre, nous étudions l’usage des namespaces Linux dans Kubernetes.

Nous avons déjà compris que Docker, Podman et les runtimes OCI utilisent les namespaces pour isoler des processus. Kubernetes reprend ces mêmes mécanismes, mais à une échelle supérieure : il ne gère pas seulement des conteneurs isolés, il orchestre des pods sur un ensemble de nœuds.

Le point central est le suivant :

```text
Dans Kubernetes, l’unité fondamentale n’est pas le conteneur seul.
L’unité fondamentale d’exécution est le pod.
```

À la fin de ce chapitre, nous savons :

- expliquer le rôle du pod comme unité d’isolation ;
    
- comprendre quels namespaces sont partagés entre conteneurs d’un même pod ;
    
- comprendre le rôle du pause container ;
    
- expliquer pourquoi les conteneurs d’un même pod partagent souvent le réseau ;
    
- comprendre les options `hostNetwork`, `hostPID` et `hostIPC` ;
    
- faire le lien entre CNI et namespaces réseau ;
    
- comprendre le rôle du `securityContext` ;
    
- utiliser les ephemeral containers pour le debug ;
    
- relier Kubernetes aux notions de namespaces, cgroups, capabilities et sécurité.
    

---

# 16.1. Rappel : Kubernetes orchestre des conteneurs

## 16.1.1. Kubernetes ne remplace pas le noyau Linux

Kubernetes n’invente pas une nouvelle forme magique d’isolation.

Il s’appuie sur les mécanismes Linux que nous avons étudiés :

```text
namespaces
cgroups
capabilities
seccomp
AppArmor ou SELinux
mounts
réseau virtuel
runtimes de conteneurs
```

Kubernetes demande au runtime de conteneurs de créer les environnements d’exécution.

Selon les installations, ce runtime peut être par exemple :

```text
containerd
CRI-O
```

Ces runtimes utilisent ensuite des runtimes OCI comme `runc` ou d’autres implémentations compatibles.

---

## 16.1.2. Kubernetes travaille au niveau du pod

Dans Docker seul, nous pensons souvent en termes de conteneur.

Dans Kubernetes, nous pensons d’abord en termes de pod.

Un pod peut contenir :

- un seul conteneur ;
    
- plusieurs conteneurs applicatifs ;
    
- un conteneur principal et un ou plusieurs sidecars ;
    
- des init containers ;
    
- des ephemeral containers pour le debug.
    

Le pod est donc une enveloppe logique qui regroupe un ou plusieurs conteneurs partageant certaines ressources.

---

# 16.2. Qu’est-ce qu’un pod ?

## 16.2.1. Définition

Un pod est la plus petite unité déployable dans Kubernetes.

Il représente un groupe de conteneurs qui doivent fonctionner ensemble sur le même nœud.

Ces conteneurs partagent généralement :

```text
un namespace réseau
une adresse IP de pod
certains volumes
une durée de vie commune
un contexte de planification
```

Ils ne sont pas répartis sur plusieurs machines : tous les conteneurs d’un même pod s’exécutent sur le même nœud.

---

## 16.2.2. Exemple simple de pod à un conteneur

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-simple
spec:
  containers:
    - name: app
      image: nginx:alpine
      ports:
        - containerPort: 80
```

Ce pod contient un seul conteneur.

Même dans ce cas simple, Kubernetes crée un environnement réseau, des montages, des cgroups et des namespaces autour de ce conteneur.

---

## 16.2.3. Exemple de pod avec sidecar

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-sidecar
spec:
  containers:
    - name: app
      image: mon-app:latest
      ports:
        - containerPort: 8080

    - name: sidecar-logs
      image: fluent-bit:latest
```

Ici, le pod contient deux conteneurs :

```text
app
sidecar-logs
```

Ils vivent ensemble et peuvent partager certains volumes et le réseau du pod.

---

# 16.3. Namespaces Linux et namespaces Kubernetes

## 16.3.1. Attention à l’ambiguïté du mot namespace

Kubernetes utilise aussi le mot `namespace`.

Mais un namespace Kubernetes n’est pas un namespace Linux.

Nous devons distinguer :

|Terme|Signification|
|---|---|
|Namespace Linux|mécanisme noyau d’isolation des processus|
|Namespace Kubernetes|espace logique pour organiser les objets Kubernetes|

Un namespace Kubernetes sert à organiser des objets comme :

```text
pods
services
deployments
configmaps
secrets
roles
```

Exemples :

```bash
kubectl get namespaces
kubectl get pods -n default
kubectl get pods -n kube-system
```

Cela n’a pas le même sens que :

```bash
readlink /proc/$$/ns/net
```

---

## 16.3.2. Exemple de namespace Kubernetes

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

Ce namespace Kubernetes organise les ressources nommées `production`.

Il ne crée pas directement un namespace Linux.

Un pod dans le namespace Kubernetes `production` aura néanmoins ses propres namespaces Linux au moment de son exécution sur un nœud.

---

## 16.3.3. Règle de vocabulaire

Nous retenons :

```text
Namespace Kubernetes = organisation logique du cluster.
Namespace Linux = isolation noyau d’un processus.
```

Dans ce chapitre, quand nous parlons de namespaces sans précision, nous parlons des namespaces Linux utilisés par les pods et conteneurs.

---

# 16.4. Les namespaces Linux dans un pod

## 16.4.1. Vue d’ensemble

Un pod Kubernetes utilise plusieurs namespaces Linux.

Selon la configuration, il peut utiliser :

```text
network namespace
PID namespace
mount namespace
UTS namespace
IPC namespace
user namespace
cgroup namespace
time namespace selon runtime et noyau
```

Mais tous ne sont pas nécessairement partagés de la même manière entre les conteneurs du pod.

---

## 16.4.2. Partage classique

Dans un pod classique :

|Namespace Linux|Comportement typique|
|---|---|
|Network|partagé entre les conteneurs du pod|
|Mount|propre à chaque conteneur, avec volumes communs montés|
|PID|souvent séparé par conteneur, sauf option de partage|
|IPC|généralement isolé du host, modalités selon runtime|
|UTS|identité liée au pod/conteneur selon runtime|
|User|dépend de la configuration et du support|
|Cgroup|organisé par pod et conteneur|
|Time|rarement manipulé directement|

Le point le plus important est le partage du namespace réseau.

---

# 16.5. Le namespace réseau du pod

## 16.5.1. Une IP par pod

Dans Kubernetes, chaque pod reçoit généralement sa propre adresse IP.

Cette IP appartient au pod, pas à chaque conteneur individuellement.

Exemple :

```bash
kubectl get pod -o wide
```

Sortie possible :

```text
NAME        READY   STATUS    IP           NODE
web-abc     1/1     Running   10.244.1.23  node-1
```

L’adresse `10.244.1.23` est l’IP du pod.

---

## 16.5.2. Les conteneurs d’un même pod partagent le réseau

Si un pod contient deux conteneurs, ils partagent généralement le même namespace réseau.

Cela signifie qu’ils partagent :

```text
la même IP
les mêmes interfaces
les mêmes routes
le même loopback
les mêmes ports
la même pile réseau
```

Conséquence importante :

```text
Deux conteneurs d’un même pod peuvent communiquer via localhost.
```

---

## 16.5.3. Exemple : app et sidecar

Imaginons un pod :

```text
Pod
├── app écoute sur 127.0.0.1:8080
└── sidecar proxy
```

Le sidecar peut joindre l’application avec :

```text
http://127.0.0.1:8080
```

car il partage le même namespace réseau.

C’est la base de nombreux modèles sidecar :

- proxy service mesh ;
    
- collecteur de logs ;
    
- agent de supervision ;
    
- proxy de sécurité ;
    
- adaptateur réseau local.
    

---

## 16.5.4. Conséquence sur les ports

Comme les conteneurs d’un pod partagent le même namespace réseau, ils ne peuvent pas écouter sur le même port et la même adresse en même temps.

Exemple problématique :

```text
container A écoute sur 0.0.0.0:8080
container B essaie aussi d’écouter sur 0.0.0.0:8080
```

Le second échoue généralement avec :

```text
Address already in use
```

C’est normal : dans le pod, le port appartient au namespace réseau partagé.

---

# 16.6. Le pause container

## 16.6.1. Pourquoi existe-t-il ?

Kubernetes utilise généralement un conteneur spécial appelé `pause container`, ou conteneur sandbox.

Son rôle est de maintenir certains namespaces du pod, en particulier le namespace réseau.

Conceptuellement :

```text
pause container
  détient le namespace réseau du pod

container app
  rejoint ce namespace réseau

container sidecar
  rejoint ce même namespace réseau
```

Le pause container est très minimal. Il ne fait presque rien, mais il sert d’ancrage aux namespaces du pod.

---

## 16.6.2. Pourquoi ne pas attacher le réseau au conteneur applicatif ?

Si le namespace réseau était attaché uniquement au premier conteneur applicatif, que se passerait-il si ce conteneur redémarrait ?

Kubernetes veut que le pod conserve une identité réseau stable pendant la durée de vie du pod.

Le pause container aide à maintenir cette stabilité.

Ainsi, les autres conteneurs peuvent redémarrer sans nécessairement recréer toute la sandbox réseau du pod.

---

## 16.6.3. Observer le pause container

Sur un nœud, avec les outils du runtime, nous pouvons parfois voir des conteneurs pause.

Avec `crictl`, par exemple :

```bash
sudo crictl ps -a
```

Nous pouvons voir des images ou noms liés à `pause`.

Selon le runtime et la distribution Kubernetes, les détails peuvent varier.

---

# 16.7. CNI et namespace réseau

## 16.7.1. Qu’est-ce que CNI ?

CNI signifie Container Network Interface.

C’est une spécification qui permet aux runtimes de déléguer la configuration réseau des pods à des plugins.

Kubernetes ne configure pas directement toute la connectivité réseau. Il appelle un plugin CNI.

Exemples de solutions CNI :

```text
Calico
Cilium
Flannel
Canal
Antrea
Weave Net
```

---

## 16.7.2. Ce que fait un plugin CNI

Un plugin CNI configure généralement :

- le namespace réseau du pod ;
    
- une interface dans le pod, souvent `eth0` ;
    
- une paire `veth` ;
    
- les routes ;
    
- la connectivité entre nœuds ;
    
- les règles de pare-feu ;
    
- éventuellement eBPF ;
    
- les NetworkPolicies.
    

Schéma conceptuel :

```text
pod network namespace
  eth0
   |
veth
   |
nœud
   |
CNI / bridge / routage / eBPF / overlay
   |
réseau cluster
```

---

## 16.7.3. Observer depuis un pod

Dans un pod :

```bash
ip addr
ip route
cat /proc/net/dev
```

Nous voyons la pile réseau du pod.

Mais dans une image minimale, `ip` peut manquer. Dans ce cas, nous pouvons utiliser un conteneur de debug ou entrer depuis le nœud avec `nsenter`.

---

## 16.7.4. NetworkPolicy

Les NetworkPolicies Kubernetes permettent de définir quelles communications sont autorisées.

Elles ne sont pas appliquées par les namespaces Linux seuls.

Elles dépendent du plugin CNI.

Nous retenons :

```text
Namespace réseau = isolation de la pile réseau.
CNI = configuration de la connectivité.
NetworkPolicy = politique de communication, si le CNI la supporte.
```

---

# 16.8. PID namespace dans Kubernetes

## 16.8.1. Vue par défaut

Par défaut, les processus d’un conteneur sont généralement isolés des autres conteneurs et de l’hôte au niveau PID.

Dans un conteneur :

```bash
ps
```

nous voyons souvent seulement les processus du conteneur.

Mais Kubernetes peut configurer le partage de processus au niveau du pod.

---

## 16.8.2. `shareProcessNamespace`

Kubernetes permet :

```yaml
spec:
  shareProcessNamespace: true
```

Cela permet aux conteneurs d’un même pod de voir leurs processus respectifs.

Exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pid-shared-demo
spec:
  shareProcessNamespace: true
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "sleep 3600"]

    - name: debug
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
```

Dans le conteneur `debug`, nous pouvons voir les processus du conteneur `app`.

---

## 16.8.3. Pourquoi partager les processus ?

Cela peut être utile pour :

- debug ;
    
- sidecar de supervision ;
    
- agents de diagnostic ;
    
- collecte de traces ;
    
- gestion de processus entre conteneurs du même pod.
    

Mais cela réduit l’isolation entre conteneurs du pod.

Nous l’activons seulement si le besoin est clair.

---

## 16.8.4. `hostPID`

Kubernetes permet aussi :

```yaml
spec:
  hostPID: true
```

Dans ce cas, le pod partage le namespace PID de l’hôte.

C’est beaucoup plus sensible.

Conséquences :

- le pod voit les processus du nœud ;
    
- il peut exposer des informations sensibles ;
    
- il peut interagir avec certains processus selon permissions ;
    
- il réduit fortement l’isolation.
    

Nous évitons `hostPID: true` sauf cas spécifique, par exemple certains agents système.

---

# 16.9. Namespace network partagé avec l’hôte : `hostNetwork`

## 16.9.1. Principe

Kubernetes permet :

```yaml
spec:
  hostNetwork: true
```

Dans ce cas, le pod partage le namespace réseau de l’hôte.

Il ne reçoit pas une IP de pod classique de la même manière.

Il utilise directement la pile réseau du nœud.

---

## 16.9.2. Conséquences

Avec `hostNetwork: true` :

- le pod voit les interfaces réseau du nœud ;
    
- il partage les ports du nœud ;
    
- il peut écouter directement sur les adresses du nœud ;
    
- il peut entrer en conflit avec d’autres services du nœud ;
    
- il contourne une partie de l’isolation réseau habituelle ;
    
- le diagnostic réseau change.
    

Exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostnetwork-demo
spec:
  hostNetwork: true
  containers:
    - name: app
      image: nginx:alpine
```

---

## 16.9.3. Cas d’usage possibles

`hostNetwork` peut être utilisé pour :

- agents réseau ;
    
- composants de monitoring bas niveau ;
    
- certains DaemonSets ;
    
- composants CNI ;
    
- outils qui doivent écouter directement sur le réseau du nœud.
    

Mais nous ne l’utilisons pas pour une application standard sans raison forte.

---

# 16.10. Namespace IPC dans Kubernetes

## 16.10.1. Isolation IPC

Les pods utilisent généralement une isolation IPC par rapport à l’hôte.

Les ressources IPC de l’hôte ne sont pas directement visibles depuis les conteneurs ordinaires.

---

## 16.10.2. `hostIPC`

Kubernetes permet :

```yaml
spec:
  hostIPC: true
```

Dans ce cas, le pod partage le namespace IPC de l’hôte.

C’est sensible.

Conséquences :

- visibilité sur certaines ressources IPC de l’hôte ;
    
- interaction possible avec des segments de mémoire partagée, sémaphores ou files de messages selon permissions ;
    
- réduction de l’isolation.
    

Nous l’évitons sauf besoin système très spécifique.

---

## 16.10.3. `/dev/shm`

Certaines applications ont besoin d’un `/dev/shm` plus grand.

Nous pouvons utiliser un volume `emptyDir` en mémoire :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shm-demo
spec:
  volumes:
    - name: dshm
      emptyDir:
        medium: Memory
        sizeLimit: 1Gi
  containers:
    - name: app
      image: mon-app:latest
      volumeMounts:
        - name: dshm
          mountPath: /dev/shm
```

Cela évite d’utiliser `hostIPC` pour un simple problème de mémoire partagée.

---

# 16.11. Mount namespaces et volumes Kubernetes

## 16.11.1. Chaque conteneur a sa vue de montages

Chaque conteneur d’un pod a généralement son propre namespace mount.

Cela signifie que chaque conteneur peut avoir une vue de fichiers adaptée.

Mais Kubernetes peut monter les mêmes volumes dans plusieurs conteneurs.

---

## 16.11.2. Exemple de volume partagé

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-shared-demo
spec:
  volumes:
    - name: shared-data
      emptyDir: {}
  containers:
    - name: producer
      image: busybox
      command: ["sh", "-c", "echo hello > /data/message.txt && sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data

    - name: consumer
      image: busybox
      command: ["sh", "-c", "cat /data/message.txt; sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data
```

Les deux conteneurs ont chacun leur namespace mount, mais ils voient le même volume à `/data`.

---

## 16.11.3. Types de volumes

Kubernetes peut monter différents types de volumes :

```text
emptyDir
configMap
secret
persistentVolumeClaim
hostPath
projected
downwardAPI
CSI volumes
```

Chaque volume est intégré dans la vue de montage du conteneur.

---

## 16.11.4. `hostPath`

`hostPath` monte un chemin du nœud dans le pod.

Exemple :

```yaml
volumes:
  - name: host-logs
    hostPath:
      path: /var/log
```

C’est puissant, mais sensible.

Risques :

- exposition de fichiers du nœud ;
    
- modification accidentelle ;
    
- dépendance au nœud ;
    
- réduction de portabilité ;
    
- risque de sécurité si le chemin est critique.
    

Nous évitons `hostPath` sauf besoin bien justifié.

---

# 16.12. Cgroups dans Kubernetes

## 16.12.1. Requests et limits

Kubernetes utilise les cgroups pour appliquer les limites.

Exemple :

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

Nous interprétons :

```text
request memory = mémoire demandée pour la planification
limit memory   = mémoire maximale autorisée
request cpu    = CPU demandé
limit cpu      = CPU maximal autorisé
```

---

## 16.12.2. Organisation par pod et conteneur

Sur le nœud, les processus des pods sont organisés dans une hiérarchie cgroup.

Nous pouvons voir des chemins liés à :

```text
kubepods
burstable
besteffort
guaranteed
pod UID
container ID
```

Les détails dépendent de :

- cgroups v1 ou v2 ;
    
- systemd ou cgroupfs ;
    
- runtime ;
    
- distribution ;
    
- configuration du cluster.
    

---

## 16.12.3. OOMKilled

Si un conteneur dépasse sa limite mémoire, le noyau peut tuer le processus.

Kubernetes affiche alors souvent :

```text
OOMKilled
```

Nous diagnostiquons avec :

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod
kubectl get events
```

Le mécanisme bas niveau est le contrôleur mémoire des cgroups.

---

# 16.13. SecurityContext

## 16.13.1. Rôle

Le `securityContext` permet de configurer des paramètres de sécurité au niveau du pod ou du conteneur.

Il peut définir notamment :

- utilisateur d’exécution ;
    
- groupe ;
    
- capabilities ;
    
- mode privilégié ;
    
- seccomp ;
    
- root filesystem en lecture seule ;
    
- élévation de privilèges ;
    
- SELinux ;
    
- AppArmor selon configuration.
    

---

## 16.13.2. Exemple de conteneur durci

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-demo
spec:
  containers:
    - name: app
      image: nginx:alpine
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
        seccompProfile:
          type: RuntimeDefault
```

Cette configuration applique plusieurs bonnes pratiques :

```text
non-root
pas d’élévation de privilèges
rootfs en lecture seule
capabilities supprimées
seccomp par défaut du runtime
```

---

## 16.13.3. `privileged`

Kubernetes permet :

```yaml
securityContext:
  privileged: true
```

C’est très sensible.

Un conteneur privilégié reçoit des droits très larges.

Nous ne l’utilisons pas pour contourner rapidement un problème de permission. Nous cherchons d’abord le besoin exact :

```text
capability manquante
périphérique nécessaire
volume nécessaire
profil seccomp trop restrictif
AppArmor ou SELinux
```

---

## 16.13.4. Capabilities

Nous pouvons ajouter ou retirer des capabilities :

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE
```

Nous préférons supprimer toutes les capabilities puis ajouter uniquement celles nécessaires.

---

# 16.14. User namespaces dans Kubernetes

## 16.14.1. Pourquoi c’est utile

Le user namespace permet de mapper l’UID `0` dans le conteneur vers un UID non privilégié sur l’hôte.

Cela réduit l’impact potentiel d’une compromission.

Dans Kubernetes, l’usage des user namespaces dépend du runtime, de la version du cluster, de la configuration des nœuds et des fonctionnalités activées.

---

## 16.14.2. Différence avec `runAsUser`

`runAsUser` change l’utilisateur utilisé dans le conteneur.

Exemple :

```yaml
securityContext:
  runAsUser: 10001
```

Mais cela ne crée pas nécessairement un user namespace.

Nous devons distinguer :

```text
runAsUser    = UID utilisé par le processus dans le conteneur
user namespace = mapping UID/GID entre conteneur et hôte
```

Les deux mécanismes peuvent se compléter, mais ils ne sont pas identiques.

---

## 16.14.3. Problèmes avec les volumes

Les user namespaces peuvent compliquer les permissions sur les volumes.

Un fichier peut apparaître avec un UID différent selon :

- le mapping UID/GID ;
    
- le type de volume ;
    
- le système de fichiers ;
    
- le runtime ;
    
- la politique de sécurité ;
    
- l’utilisateur d’exécution.
    

Nous diagnostiquons avec :

```bash
id
ls -ln /chemin
cat /proc/self/uid_map
cat /proc/self/gid_map
```

---

# 16.15. Ephemeral containers pour le debug

## 16.15.1. Problème des images minimales

Les images de production sont souvent minimales.

Elles peuvent ne pas contenir :

```text
sh
bash
ip
ss
ps
curl
dig
tcpdump
strace
```

Cela complique le diagnostic.

---

## 16.15.2. Principe des ephemeral containers

Les ephemeral containers permettent d’ajouter temporairement un conteneur de debug dans un pod existant.

Nous pouvons utiliser :

```bash
kubectl debug -it <pod> --image=busybox --target=<container>
```

Selon le cluster et les permissions, cela ajoute un conteneur de debug qui peut partager certains namespaces avec le pod ou cibler un conteneur existant.

---

## 16.15.3. Intérêt

Cela permet de diagnostiquer :

- réseau du pod ;
    
- DNS ;
    
- montages ;
    
- processus selon configuration ;
    
- fichiers visibles ;
    
- environnement d’exécution.
    

Nous évitons ainsi de mettre des outils de debug dans les images de production.

---

## 16.15.4. Limites

Les ephemeral containers :

- nécessitent des droits Kubernetes ;
    
- ne sont pas faits pour exécuter une charge durable ;
    
- peuvent ne pas partager tous les namespaces selon configuration ;
    
- doivent respecter les politiques de sécurité du cluster ;
    
- ne résolvent pas tous les problèmes d’observabilité.
    

---

# 16.16. Debug réseau dans Kubernetes

## 16.16.1. Questions à se poser

Quand un pod ne communique pas, nous demandons :

```text
Le pod a-t-il une IP ?
L’interface eth0 est-elle présente ?
La route par défaut existe-t-elle ?
Le DNS fonctionne-t-il ?
Le service Kubernetes pointe-t-il vers les bons endpoints ?
La NetworkPolicy autorise-t-elle le trafic ?
Le CNI fonctionne-t-il ?
Le problème vient-il du pod, du service, du nœud ou du cluster ?
```

---

## 16.16.2. Commandes utiles

```bash
kubectl get pod -o wide
kubectl describe pod <pod>
kubectl get svc
kubectl get endpoints
kubectl exec -it <pod> -- ip addr
kubectl exec -it <pod> -- ip route
kubectl exec -it <pod> -- nslookup kubernetes.default
```

Si l’image ne contient pas les outils :

```bash
kubectl debug -it <pod> --image=nicolaka/netshoot
```

Selon les politiques du cluster, l’image de debug peut être différente.

---

## 16.16.3. Comprendre `localhost`

Dans un pod multi-conteneurs :

```text
localhost désigne le namespace réseau partagé du pod.
```

Donc si un conteneur `app` écoute sur `127.0.0.1:8080`, un sidecar du même pod peut le joindre avec :

```text
http://127.0.0.1:8080
```

Mais un autre pod ne peut pas utiliser ce `localhost`. Il doit utiliser l’IP du pod ou un Service Kubernetes.

---

# 16.17. Debug PID dans Kubernetes

## 16.17.1. Voir les processus

Dans un conteneur :

```bash
kubectl exec -it <pod> -- ps -ef
```

Nous voyons les processus visibles dans le namespace PID du conteneur ou du pod selon configuration.

---

## 16.17.2. Avec `shareProcessNamespace`

Si le pod a :

```yaml
shareProcessNamespace: true
```

un conteneur de debug peut voir les processus des autres conteneurs du pod.

Cela est utile pour :

- observer un processus ;
    
- envoyer un signal ;
    
- lire `/proc/<PID>` selon permissions ;
    
- diagnostiquer des zombies ;
    
- analyser le PID 1.
    

---

## 16.17.3. Avec `hostPID`

Si le pod a :

```yaml
hostPID: true
```

il voit les processus du nœud.

C’est utile pour certains agents système, mais sensible.

Nous devons éviter cette option pour les applications standards.

---

# 16.18. Debug des montages dans Kubernetes

## 16.18.1. Problèmes fréquents

Les problèmes de montage peuvent venir de :

- volume absent ;
    
- mauvais `mountPath` ;
    
- secret ou configMap non monté ;
    
- permissions incorrectes ;
    
- `readOnly` activé ;
    
- `subPath` mal utilisé ;
    
- PVC non attaché ;
    
- `hostPath` dépendant du nœud ;
    
- root filesystem en lecture seule.
    

---

## 16.18.2. Commandes utiles

```bash
kubectl describe pod <pod>
kubectl exec -it <pod> -- mount
kubectl exec -it <pod> -- findmnt
kubectl exec -it <pod> -- ls -la /chemin
kubectl exec -it <pod> -- id
kubectl exec -it <pod> -- ls -ln /chemin
```

Si `findmnt` n’est pas disponible, nous pouvons utiliser :

```bash
kubectl exec -it <pod> -- cat /proc/self/mountinfo
```

---

## 16.18.3. `readOnlyRootFilesystem`

Si nous activons :

```yaml
readOnlyRootFilesystem: true
```

l’application ne peut plus écrire partout.

Nous devons prévoir des volumes pour les chemins nécessaires :

```yaml
volumeMounts:
  - name: tmp
    mountPath: /tmp
volumes:
  - name: tmp
    emptyDir: {}
```

Nous devons donc connaître les besoins d’écriture de l’application.

---

# 16.19. Options sensibles dans Kubernetes

## 16.19.1. `hostNetwork`

```yaml
hostNetwork: true
```

Risque :

```text
partage du réseau du nœud
conflits de ports
exposition directe
réduction de l’isolation réseau
```

---

## 16.19.2. `hostPID`

```yaml
hostPID: true
```

Risque :

```text
visibilité sur les processus du nœud
fuite d’informations
interactions possibles selon permissions
```

---

## 16.19.3. `hostIPC`

```yaml
hostIPC: true
```

Risque :

```text
partage des IPC du nœud
exposition possible de mémoire partagée ou sémaphores
```

---

## 16.19.4. `privileged`

```yaml
securityContext:
  privileged: true
```

Risque :

```text
capabilities étendues
accès plus large aux périphériques
affaiblissement de seccomp/AppArmor/SELinux selon configuration
```

---

## 16.19.5. `hostPath`

```yaml
hostPath:
  path: /
```

Risque :

```text
exposition du système de fichiers du nœud
modifications accidentelles ou malveillantes
dépendance forte au nœud
```

Nous traitons ces options comme des exceptions, pas comme des paramètres ordinaires.

---

# 16.20. Méthode d’analyse d’un pod

## 16.20.1. Étape 1 : identifier le pod et le nœud

```bash
kubectl get pod <pod> -o wide
kubectl describe pod <pod>
```

Nous regardons :

- nœud ;
    
- IP du pod ;
    
- conteneurs ;
    
- événements ;
    
- volumes ;
    
- securityContext ;
    
- ressources ;
    
- redémarrages.
    

---

## 16.20.2. Étape 2 : observer depuis le pod

```bash
kubectl exec -it <pod> -- hostname
kubectl exec -it <pod> -- id
kubectl exec -it <pod> -- cat /proc/self/cgroup
kubectl exec -it <pod> -- ls -l /proc/self/ns
kubectl exec -it <pod> -- ps -ef
kubectl exec -it <pod> -- cat /proc/self/mountinfo
```

Selon l’image, certaines commandes peuvent manquer.

---

## 16.20.3. Étape 3 : utiliser un conteneur de debug

```bash
kubectl debug -it <pod> --image=busybox
```

ou une image plus riche :

```bash
kubectl debug -it <pod> --image=nicolaka/netshoot
```

Nous utilisons ce conteneur pour vérifier :

- DNS ;
    
- réseau ;
    
- routes ;
    
- ports ;
    
- montages ;
    
- processus selon partage ;
    
- accès aux services.
    

---

## 16.20.4. Étape 4 : analyser les options sensibles

Nous cherchons dans le manifeste :

```yaml
hostNetwork: true
hostPID: true
hostIPC: true
securityContext:
  privileged: true
hostPath:
```

Si ces options sont présentes, nous évaluons leur justification et leur impact.

---

# 16.21. Exemple complet de pod multi-conteneurs

## 16.21.1. Manifeste

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-demo
spec:
  volumes:
    - name: shared
      emptyDir: {}

  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo hello > /shared/message.txt; nc -lk -p 8080"]
      volumeMounts:
        - name: shared
          mountPath: /shared

    - name: sidecar
      image: busybox
      command: ["sh", "-c", "while true; do cat /shared/message.txt; sleep 10; done"]
      volumeMounts:
        - name: shared
          mountPath: /shared
```

Ce pod illustre :

- volume partagé ;
    
- namespace réseau partagé ;
    
- conteneurs distincts ;
    
- durée de vie commune.
    

---

## 16.21.2. Observations

Nous pouvons tester :

```bash
kubectl apply -f pod.yaml
kubectl get pod multi-container-demo -o wide
kubectl exec -it multi-container-demo -c app -- hostname
kubectl exec -it multi-container-demo -c sidecar -- cat /shared/message.txt
```

Pour tester le réseau partagé, si les outils sont disponibles :

```bash
kubectl exec -it multi-container-demo -c sidecar -- wget -qO- http://127.0.0.1:8080
```

Nous observons que le sidecar rejoint le service exposé localement par `app`, car ils partagent le namespace réseau du pod.

---

# 16.22. Travaux pratiques

## TP 1 — Observer les namespaces d’un pod

Dans un pod disposant de `ls` :

```bash
kubectl exec -it <pod> -- ls -l /proc/self/ns
```

Nous répondons :

1. Quels namespaces voyons-nous ?
    
2. Le pod voit-il un namespace réseau ?
    
3. Le namespace PID est-il partagé avec les autres conteneurs ?
    
4. Que pouvons-nous déduire de `/proc/self/cgroup` ?
    

---

## TP 2 — Vérifier le partage réseau dans un pod

Nous créons un pod avec deux conteneurs :

- `app` écoute sur `127.0.0.1:8080` ;
    
- `debug` tente de joindre `127.0.0.1:8080`.
    

Nous répondons :

1. Pourquoi `localhost` fonctionne-t-il entre deux conteneurs du même pod ?
    
2. Pourquoi cela ne fonctionnerait-il pas depuis un autre pod ?
    
3. Quel namespace explique ce comportement ?
    

---

## TP 3 — Tester `shareProcessNamespace`

Nous créons un pod avec :

```yaml
shareProcessNamespace: true
```

Puis nous exécutons :

```bash
kubectl exec -it <pod> -c debug -- ps -ef
```

Nous répondons :

1. Voyons-nous les processus de l’autre conteneur ?
    
2. Quel namespace est partagé ?
    
3. Quels sont les avantages et risques ?
    

---

## TP 4 — Observer les volumes

Nous créons un volume `emptyDir` monté dans deux conteneurs.

Nous répondons :

1. Le fichier créé par un conteneur est-il visible par l’autre ?
    
2. Les conteneurs partagent-ils nécessairement le même namespace mount ?
    
3. Comment Kubernetes rend-il le volume visible dans les deux conteneurs ?
    

---

## TP 5 — Analyser un `securityContext`

Nous analysons :

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
  seccompProfile:
    type: RuntimeDefault
```

Nous répondons :

1. Quel risque réduit chaque option ?
    
2. Quelle option concerne les capabilities ?
    
3. Quelle option concerne seccomp ?
    
4. Que devons-nous prévoir si le rootfs est en lecture seule ?
    

---

## TP 6 — Identifier les options sensibles

Nous analysons un manifeste contenant :

```yaml
hostNetwork: true
hostPID: true
hostIPC: true
securityContext:
  privileged: true
volumes:
  - name: host
    hostPath:
      path: /
```

Nous répondons :

1. Quelles isolations sont réduites ?
    
2. Quels risques apparaissent ?
    
3. Quelles alternatives plus ciblées proposons-nous ?
    

---

# 16.23. Pièges classiques

## 16.23.1. Confondre namespace Kubernetes et namespace Linux

Un namespace Kubernetes comme `production` n’est pas un namespace Linux.

Il organise les objets du cluster.

Les namespaces Linux isolent les processus sur les nœuds.

---

## 16.23.2. Croire qu’un conteneur d’un pod a sa propre IP

Dans Kubernetes, l’IP est généralement celle du pod.

Les conteneurs du même pod partagent le namespace réseau.

Ils partagent donc la même IP.

---

## 16.23.3. Oublier les conflits de ports dans un pod

Deux conteneurs d’un même pod ne peuvent pas écouter sur le même port et la même adresse en même temps.

Ils partagent le même namespace réseau.

---

## 16.23.4. Utiliser `hostNetwork` pour résoudre trop vite un problème réseau

Si un pod n’arrive pas à communiquer, nous ne devons pas passer immédiatement à :

```yaml
hostNetwork: true
```

Nous devons d’abord diagnostiquer :

- CNI ;
    
- DNS ;
    
- Service ;
    
- Endpoints ;
    
- NetworkPolicy ;
    
- routes ;
    
- ports ;
    
- application.
    

---

## 16.23.5. Utiliser `privileged` pour résoudre un problème de permission

Même logique :

```yaml
privileged: true
```

ne doit pas être une solution réflexe.

Nous cherchons le besoin précis :

- capability ;
    
- volume ;
    
- user ;
    
- groupe ;
    
- périphérique ;
    
- profil seccomp ;
    
- SELinux/AppArmor.
    

---

# 16.24. Ce que nous devons retenir

Nous retenons les points suivants :

1. Kubernetes utilise les namespaces Linux via les runtimes de conteneurs.
    
2. L’unité fondamentale d’exécution Kubernetes est le pod.
    
3. Un namespace Kubernetes n’est pas un namespace Linux.
    
4. Les conteneurs d’un même pod partagent généralement le namespace réseau.
    
5. Ils partagent donc la même IP et le même `localhost`.
    
6. Le pause container sert à maintenir certains namespaces du pod.
    
7. Le CNI configure le namespace réseau du pod et la connectivité cluster.
    
8. Les NetworkPolicies dépendent du support du plugin CNI.
    
9. Le namespace PID peut être partagé avec `shareProcessNamespace`.
    
10. `hostPID`, `hostNetwork` et `hostIPC` réduisent fortement l’isolation.
    
11. Les volumes Kubernetes sont montés dans les namespaces mount des conteneurs.
    
12. `hostPath` est puissant mais sensible.
    
13. Les cgroups appliquent les requests et limits.
    
14. `OOMKilled` vient généralement d’un dépassement de limite mémoire cgroup.
    
15. `securityContext` permet de configurer utilisateur, capabilities, seccomp et autres paramètres de sécurité.
    
16. Les ephemeral containers facilitent le debug sans modifier l’image applicative.
    
17. Pour diagnostiquer un pod, nous devons croiser namespaces, CNI, volumes, cgroups, securityContext et événements Kubernetes.
    

---

# Conclusion du chapitre 16

Nous avons étudié l’usage des namespaces Linux dans Kubernetes.

Nous savons maintenant que Kubernetes ne remplace pas les mécanismes Linux : il les orchestre. Le pod est l’unité centrale, et plusieurs conteneurs d’un même pod partagent généralement le même namespace réseau, donc la même IP et le même `localhost`.

Nous avons aussi compris le rôle du pause container, qui sert d’ancrage aux namespaces du pod, ainsi que le rôle du CNI, qui configure la connectivité réseau.

Nous avons relié les namespaces aux options Kubernetes sensibles comme `hostNetwork`, `hostPID`, `hostIPC`, `privileged` et `hostPath`. Ces options peuvent être nécessaires dans certains cas système, mais elles réduisent fortement l’isolation et doivent donc être justifiées.

Nous retenons surtout que diagnostiquer Kubernetes demande de raisonner à plusieurs niveaux : pod, conteneur, namespace Linux, cgroups, volumes, réseau CNI, sécurité et événements du cluster.

Dans le chapitre suivant, nous passons aux travaux pratiques pour manipuler directement les namespaces et consolider l’ensemble du cours.