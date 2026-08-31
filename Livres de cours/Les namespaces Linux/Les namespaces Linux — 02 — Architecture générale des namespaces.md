---
schema_version: 1
uid: 01M1BQ6248AHPD9BB5T6AMWP3N
titre: "Les namespaces Linux — 02 — Architecture générale des namespaces"
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
resume: "Chapitre 2 sur 16 du livre « Les namespaces Linux » : Architecture générale des namespaces. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 2/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 01 — Introduction aux namespaces Linux|← 01 — Introduction aux namespaces Linux]] · [[Les namespaces Linux — 03 — Les outils de manipulation des namespaces|03 — Les outils de manipulation des namespaces →]]

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

## 2.1. Un processus appartient à plusieurs namespaces

## 2.1.1. Une appartenance multiple

Un processus Linux n’est pas “dans un namespace” au singulier.

Il appartient simultanément à plusieurs namespaces, chacun correspondant à une catégorie d’isolation.

Nous pouvons observer cela avec :

```bash
ls -l /proc/$$/ns
```

Sortie possible :

```text
cgroup -> cgroup:
ipc    -> ipc:
mnt    -> mnt:
net    -> net:
pid    -> pid:
time   -> time:
user   -> user:
uts    -> uts:
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

## 2.2. Observer les namespaces avec `/proc/<PID>/ns`

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
mnt -> mnt:
pid -> pid:
net -> net:
uts -> uts:
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
pid:
```

Pour le namespace réseau :

```bash
readlink /proc/$$/ns/net
```

Sortie possible :

```text
net:
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
pid:
pid:
```

les deux processus partagent le même namespace PID.

Si elles sont différentes :

```text
pid:
pid:
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

## 2.3. Identifiants symboliques des namespaces

## 2.3.1. Que signifie `pid:[4026531836]` ?

Quand nous voyons :

```text
pid:
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
pid:
net:
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

## 2.4. Création de namespaces : `clone` et `unshare`

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

## 2.5. Entrer dans un namespace existant : `setns` et `nsenter`

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

## 2.6. Lister les namespaces avec `lsns`

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

## 2.7. Héritage des namespaces

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

## 2.8. Durée de vie d’un namespace

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

## 2.9. Namespaces nommés avec `ip netns`

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

## 2.10. Les namespaces et `/proc`

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

## 2.11. Relation avec les conteneurs

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

## 2.12. Pièges classiques

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

## 2.13. Exercices

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

## 2.14. Ce que nous devons retenir

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

## Conclusion du chapitre 2

Nous avons étudié l’architecture générale des namespaces Linux.

Nous savons maintenant qu’un processus appartient à plusieurs namespaces en même temps, et que nous pouvons observer ces appartenances avec `/proc/<PID>/ns`. Nous avons compris que les namespaces sont hérités par défaut, mais qu’ils peuvent être créés ou modifiés avec des mécanismes comme `clone`, `unshare` et `setns`.

Nous avons également introduit les outils pratiques `unshare`, `nsenter`, `lsns` et `ip netns`, qui nous permettent de créer, lister et rejoindre des namespaces.

Dans le chapitre suivant, nous détaillons précisément les outils de manipulation des namespaces. Nous verrons comment les utiliser proprement, quelles options connaître, et comment éviter les erreurs classiques lors des manipulations manuelles.

---
> [!info] Livre « Les namespaces Linux » — chapitre 2/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 01 — Introduction aux namespaces Linux|← 01 — Introduction aux namespaces Linux]] · [[Les namespaces Linux — 03 — Les outils de manipulation des namespaces|03 — Les outils de manipulation des namespaces →]]
