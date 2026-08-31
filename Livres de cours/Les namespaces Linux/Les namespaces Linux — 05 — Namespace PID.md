---
schema_version: 1
uid: 01M1BQ624BMR6KC49S8HZ8RCP8
titre: "Les namespaces Linux — 05 — Namespace PID"
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
resume: "Chapitre 5 sur 16 du livre « Les namespaces Linux » : Namespace PID. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 5/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 04 — Namespace UTS|← 04 — Namespace UTS]] · [[Les namespaces Linux — 06 — Namespace mount|06 — Namespace mount →]]

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

## 5.1. Pourquoi isoler les PID ?

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

## 5.2. Observer le namespace PID courant

## 5.2.1. Avec `/proc/<PID>/ns/pid`

Nous pouvons observer le namespace PID de notre shell courant :

```bash
readlink /proc/$$/ns/pid
```

Exemple :

```text
pid:
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
pid:
pid:
```

Les processus partagent le même namespace PID.

Valeurs différentes :

```text
pid:
pid:
```

Les processus sont dans des namespaces PID différents.

---

## 5.3. Créer un namespace PID avec `unshare`

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

## 5.4. Le PID 1 dans un namespace PID

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

## 5.5. Processus zombies et rôle du PID 1

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

## 5.6. Signaux et PID 1

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

## 5.7. PID namespace et hiérarchie

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

## 5.8. Lien avec `/proc`

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

## 5.9. PID namespace et Docker

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

## 5.10. PID namespace et Kubernetes

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

## 5.11. Entrer dans un namespace PID avec `nsenter`

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

## 5.12. Expérience complète guidée

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

## 5.13. Pièges classiques

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

## 5.14. Exercices

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

## 5.15. Ce que nous devons retenir

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

## Conclusion du chapitre 5

Nous avons étudié le namespace PID, un mécanisme central de l’isolation Linux.

Nous savons maintenant qu’un processus peut être vu avec des PID différents selon le namespace depuis lequel nous l’observons. Nous avons compris pourquoi un conteneur peut avoir son propre PID 1, pourquoi `/proc` doit être remonté dans un namespace PID, et pourquoi le processus PID 1 a des responsabilités particulières.

Nous avons aussi vu que cette isolation peut être réduite ou supprimée avec des options comme `--pid=host` ou `hostPID: true`, ce qui doit être réservé à des cas particuliers.

Dans le chapitre suivant, nous étudions le namespace mount, qui permet à un processus de voir une arborescence de montages différente de celle de l’hôte.

---
> [!info] Livre « Les namespaces Linux » — chapitre 5/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 04 — Namespace UTS|← 04 — Namespace UTS]] · [[Les namespaces Linux — 06 — Namespace mount|06 — Namespace mount →]]
