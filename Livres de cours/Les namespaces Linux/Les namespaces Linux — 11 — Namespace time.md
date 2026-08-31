---
schema_version: 1
uid: 01M1BQ624JP9XAPDTPKST5Y42H
titre: "Les namespaces Linux — 11 — Namespace time"
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
resume: "Chapitre 11 sur 16 du livre « Les namespaces Linux » : Namespace time. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 11/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 10 — Namespace cgroup|← 10 — Namespace cgroup]] · [[Les namespaces Linux — 12 — Namespaces et proc|12 — Namespaces et proc →]]

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

## 11.1. Pourquoi isoler le temps ?

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

## 11.2. Les différents types de temps sous Linux

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

## 11.3. Qu’est-ce que le namespace time ?

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

## 11.4. Observer le namespace time courant

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
time:
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
time_for_children -> time:
```

Cette entrée indique le namespace time qui sera utilisé pour les processus enfants.

Ce détail est important parce que la création et la configuration du namespace time impliquent souvent les futurs enfants du processus.

---

## 11.5. Lire `/proc/self/timens_offsets`

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

## 11.6. Créer un namespace time

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

## 11.7. Modifier les offsets temporels

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

## 11.8. Mesurer les horloges en pratique

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

## 11.9. Cas d’usage : checkpoint/restore

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

## 11.10. Cas d’usage : tests

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

## 11.11. Cas d’usage : conteneurs avancés

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

## 11.12. Lien avec Docker, Podman et Kubernetes

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

## 11.13. Sécurité du namespace time

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

## 11.14. Différence avec le fuseau horaire

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

## 11.15. Diagnostic autour du namespace time

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

## 11.16. Expérience guidée

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

## 11.17. Limites du namespace time

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

## 11.18. Pièges classiques

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

## 11.19. Exercices

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

## 11.20. Ce que nous devons retenir

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

## Conclusion du chapitre 11

Nous avons étudié le namespace time, un namespace plus spécialisé que ceux vus précédemment.

Nous savons maintenant qu’il ne sert pas simplement à changer la date visible avec `date`, mais à appliquer des offsets à certaines horloges utilisées pour mesurer des durées, notamment `CLOCK_MONOTONIC` et `CLOCK_BOOTTIME`.

Nous avons compris pourquoi ce mécanisme est utile pour des scénarios avancés comme le checkpoint/restore, la migration de conteneurs ou certains tests système. Nous avons aussi vu ses limites : il dépend du noyau, des permissions, et ne remplace pas les mécanismes applicatifs de simulation du temps.

Dans le chapitre suivant, nous faisons le lien entre les namespaces et `/proc`, afin de comprendre comment `/proc/<PID>/ns`, `/proc/net`, `/proc/mounts`, `/proc/self/cgroup`, `uid_map` et `gid_map` reflètent les différentes vues isolées du système.

---
> [!info] Livre « Les namespaces Linux » — chapitre 11/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 10 — Namespace cgroup|← 10 — Namespace cgroup]] · [[Les namespaces Linux — 12 — Namespaces et proc|12 — Namespaces et proc →]]
