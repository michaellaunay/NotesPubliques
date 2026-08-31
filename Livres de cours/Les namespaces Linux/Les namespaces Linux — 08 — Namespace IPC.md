---
schema_version: 1
uid: 01M1BQ624FFF7064423V8VNAC4
titre: "Les namespaces Linux — 08 — Namespace IPC"
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
resume: "Chapitre 8 sur 16 du livre « Les namespaces Linux » : Namespace IPC. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 8/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 07 — Namespace network|← 07 — Namespace network]] · [[Les namespaces Linux — 09 — Namespace user|09 — Namespace user →]]

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

## 8.1. Pourquoi isoler les communications inter-processus ?

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

## 8.2. Qu’est-ce que l’IPC ?

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

## 8.3. Observer le namespace IPC courant

## 8.3.1. Avec `/proc/<PID>/ns/ipc`

Nous pouvons observer le namespace IPC de notre shell courant :

```bash
readlink /proc/$$/ns/ipc
```

Exemple :

```text
ipc:
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
Shell courant : ipc:
PID 1         : ipc:
```

Les deux processus partagent le même namespace IPC.

Valeurs différentes :

```text
Shell courant : ipc:
PID 1         : ipc:
```

Les deux processus ont des vues IPC différentes.

---

## 8.4. Observer les ressources IPC avec `ipcs`

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

## 8.5. Créer un namespace IPC avec `unshare`

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

## 8.6. Créer une ressource IPC pour observer l’isolation

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

## 8.7. Mémoire partagée et `/dev/shm`

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

## 8.8. Sémaphores

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

## 8.9. Files de messages

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

## 8.10. Lien avec Docker

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

## 8.11. Lien avec Kubernetes

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

## 8.12. Namespace IPC et sécurité

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

## 8.13. Diagnostic IPC

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

## 8.14. Expérience complète guidée

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

## 8.15. Pièges classiques

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

## 8.16. Exercices

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

## 8.17. Ce que nous devons retenir

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

## Conclusion du chapitre 8

Nous avons étudié le namespace IPC, qui isole certains mécanismes de communication inter-processus.

Nous savons maintenant observer le namespace IPC d’un processus, utiliser `ipcs` pour voir les ressources IPC System V, créer un namespace IPC avec `unshare`, et entrer dans un namespace existant avec `nsenter`.

Nous avons aussi compris que l’IPC ne doit pas être analysé isolément. Dans les environnements conteneurisés, les problèmes de mémoire partagée impliquent souvent `/dev/shm`, donc aussi le namespace mount et la configuration du runtime.

Nous retenons surtout que le namespace IPC évite que des processus isolés voient ou manipulent les ressources IPC de l’hôte ou d’un autre conteneur. Comme toujours avec les namespaces, cette isolation est utile, mais elle ne suffit pas à elle seule : elle doit être combinée avec les permissions, les cgroups, les capabilities et les politiques de sécurité.

Dans le chapitre suivant, nous étudions le namespace user, l’un des namespaces les plus importants et les plus subtils, car il permet d’être `root` dans un namespace sans être réellement `root` sur l’hôte.

---
> [!info] Livre « Les namespaces Linux » — chapitre 8/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 07 — Namespace network|← 07 — Namespace network]] · [[Les namespaces Linux — 09 — Namespace user|09 — Namespace user →]]
