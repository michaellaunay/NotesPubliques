---
schema_version: 1
uid: 01M1BQ629RHK7DNZ4CMSZEVWVB
titre: "proc — 06 — Mémoire d’un processus"
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
resume: "Chapitre 6 sur 11 du livre « proc » : Mémoire d’un processus. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 6/11
> [[proc — Sommaire|Sommaire]] · [[proc — 05 — Fichiers ouverts et descripteurs|← 05 — Fichiers ouverts et descripteurs]] · [[proc — 07 — Réseau et proc|07 — Réseau et proc →]]

# Chapitre 6 — Mémoire d’un processus
## Objectifs du chapitre

Dans ce chapitre, nous étudions la mémoire d’un processus à travers `/proc`.

Nous avons vu précédemment qu’un processus possède une ligne de commande, un état, des fichiers ouverts, un environnement et des descripteurs. Nous allons maintenant nous intéresser à son espace mémoire.

Sous Linux, chaque processus possède une mémoire virtuelle. Cette mémoire virtuelle ne correspond pas directement à la quantité de RAM réellement consommée. Elle est composée de zones mémoire, appelées mappings, qui peuvent représenter :

- le code exécutable du programme ;
    
- les bibliothèques partagées ;
    
- le tas ;
    
- la pile ;
    
- des fichiers mappés en mémoire ;
    
- de la mémoire anonyme ;
    
- des zones partagées ;
    
- des zones réservées mais pas encore réellement utilisées.
    

À la fin de ce chapitre, nous savons :

- expliquer la différence entre mémoire virtuelle et mémoire résidente ;
    
- lire `/proc/<PID>/maps` ;
    
- identifier la pile, le tas et les bibliothèques partagées ;
    
- comprendre les permissions mémoire ;
    
- lire `/proc/<PID>/smaps` et `/proc/<PID>/smaps_rollup` ;
    
- distinguer RSS, PSS, mémoire privée et mémoire partagée ;
    
- utiliser `/proc/<PID>/limits` pour comprendre les limites de ressources ;
    
- construire un premier diagnostic mémoire sur un processus.


## 6.1. Mémoire virtuelle et mémoire physique

### 6.1.1. Pourquoi la mémoire d’un processus est difficile à comprendre ?

Lorsque nous disons qu’un processus “consomme de la mémoire”, nous simplifions fortement la réalité.

Sous Linux, un processus ne voit pas directement la RAM physique. Il manipule un espace d’adressage virtuel.

Cet espace virtuel est ensuite traduit par le noyau et le matériel, notamment par la MMU, en pages mémoire physiques.

Nous devons donc distinguer plusieurs notions :

|Notion|Signification|
|---|---|
|Mémoire virtuelle|espace d’adressage vu par le processus|
|Mémoire résidente|partie réellement présente en RAM|
|Mémoire partagée|mémoire utilisée par plusieurs processus|
|Mémoire privée|mémoire propre à un processus|
|Mémoire anonyme|mémoire non directement associée à un fichier|
|Mémoire mappée|mémoire associée à un fichier ou à une ressource|

Cette distinction est essentielle pour éviter de mauvaises conclusions.


### 6.1.2. Exemple classique : `VmSize` n’est pas la RAM consommée

Dans `/proc/<PID>/status`, nous pouvons lire :

```bash
grep -E '^(VmSize|VmRSS):' /proc/<PID>/status
```

Exemple :

```text
VmSize:   1250000 kB
VmRSS:      85000 kB
```

Nous pourrions croire que le processus consomme 1,25 Go de RAM. Ce serait faux.

`VmSize` représente l’espace mémoire virtuel total du processus.

`VmRSS` représente la mémoire résidente, c’est-à-dire la partie actuellement présente en RAM.

Nous retenons :

```text
VmSize élevé ≠ consommation RAM élevée.
VmRSS est souvent plus proche de la mémoire réellement présente en RAM.
```

Mais même `VmRSS` doit être interprété avec prudence, car il inclut de la mémoire partagée.


## 6.2. Préparer un processus d’exemple

### 6.2.1. Lancer un processus simple

Pour observer la mémoire, nous lançons un processus simple :

```bash
sleep 1000 &
pid=$!
echo "$pid"
```

Puis nous observons ses informations mémoire :

```bash
grep -E '^(Name|State|VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
```

Nous obtenons une première vue.

Cependant, `sleep` est très simple. Pour observer plus de mappings, nous pouvons utiliser un programme plus riche, par exemple notre shell :

```bash
pid=$$
```

ou un processus Python :

```bash
python3 -c 'import time; data = bytearray(100 * 1024 * 1024); time.sleep(1000)' &
pid=$!
```

Ce processus alloue environ 100 Mio de mémoire et reste actif.


### 6.2.2. Nettoyage à la fin

Lorsque nous lançons un processus de test en arrière-plan, nous devons penser à le terminer :

```bash
kill $pid
```

Si nécessaire :

```bash
kill -9 $pid
```

Nous évitons de laisser des processus de test tourner inutilement.


## 6.3. `/proc/<PID>/maps` : cartographie mémoire

### 6.3.1. Lire `maps`

Le fichier principal pour comprendre l’organisation mémoire d’un processus est :

```bash
cat /proc/<PID>/maps
```

Exemple :

```bash
cat /proc/$pid/maps | head
```

Sortie possible :

```text
55b1f3b2d000-55b1f3b2f000 r--p 00000000 08:02 393324 /usr/bin/sleep
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
55b1f3b33000-55b1f3b35000 r--p 00006000 08:02 393324 /usr/bin/sleep
55b1f3b35000-55b1f3b36000 r--p 00007000 08:02 393324 /usr/bin/sleep
55b1f3b36000-55b1f3b37000 rw-p 00008000 08:02 393324 /usr/bin/sleep
55b1f4f4f000-55b1f4f70000 rw-p 00000000 00:00 0      [heap]
7f83d5c00000-7f83d5c21000 rw-p 00000000 00:00 0
7f83d5c21000-7f83d6000000 ---p 00000000 00:00 0
```

Chaque ligne décrit une zone mémoire du processus.


### 6.3.2. Structure d’une ligne de `maps`

Une ligne de `/proc/<PID>/maps` suit généralement cette structure :

```text
adresse-début-adresse-fin permissions offset périphérique inode chemin
```

Exemple :

```text
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
```

Nous pouvons découper :

|Champ|Exemple|Signification|
|---|---|---|
|Plage d’adresses|`55b1f3b2f000-55b1f3b33000`|zone mémoire virtuelle|
|Permissions|`r-xp`|droits de lecture, écriture, exécution, partage|
|Offset|`00002000`|décalage dans le fichier mappé|
|Périphérique|`08:02`|périphérique contenant le fichier|
|Inode|`393324`|inode du fichier|
|Chemin|`/usr/bin/sleep`|fichier ou zone spéciale|


### 6.3.3. Plages d’adresses

La première colonne indique la plage d’adresses virtuelles.

Exemple :

```text
55b1f3b2f000-55b1f3b33000
```

Cela signifie que la zone commence à l’adresse virtuelle :

```text
55b1f3b2f000
```

et se termine à :

```text
55b1f3b33000
```

Nous pouvons calculer la taille de cette zone :

```text
0x55b1f3b33000 - 0x55b1f3b2f000 = 0x4000
```

Soit 16 Kio.

Nous pouvons automatiser ce calcul, mais l’idée principale est que `maps` expose des plages d’adresses virtuelles, pas directement des adresses physiques.


### 6.3.4. Permissions mémoire

La deuxième colonne indique les permissions.

Exemples :

```text
r--p
r-xp
rw-p
---p
r--s
rw-s
```

Nous interprétons les caractères :

|Caractère|Signification|
|---|---|
|`r`|lecture autorisée|
|`w`|écriture autorisée|
|`x`|exécution autorisée|
|`-`|permission absente|
|`p`|mapping privé|
|`s`|mapping partagé|

Exemple :

```text
r-xp
```

signifie :

```text
lisible, exécutable, non modifiable, privé
```

C’est typiquement une zone de code.

Exemple :

```text
rw-p
```

signifie :

```text
lisible, modifiable, non exécutable, privé
```

C’est typiquement une zone de données ou de tas.


## 6.4. Les zones mémoire importantes

### 6.4.1. L’exécutable principal

Dans `maps`, nous voyons souvent plusieurs lignes associées à l’exécutable.

Exemple :

```text
55b1f3b2d000-55b1f3b2f000 r--p ... /usr/bin/sleep
55b1f3b2f000-55b1f3b33000 r-xp ... /usr/bin/sleep
55b1f3b33000-55b1f3b35000 r--p ... /usr/bin/sleep
55b1f3b36000-55b1f3b37000 rw-p ... /usr/bin/sleep
```

Nous observons que le même fichier peut être mappé plusieurs fois avec des permissions différentes.

Cela correspond aux différentes sections du binaire :

- code exécutable ;
    
- données en lecture seule ;
    
- données modifiables ;
    
- métadonnées.
    

Nous pouvons faire le lien avec les formats exécutables comme ELF.


### 6.4.2. Le tas : `[heap]`

Le tas est une zone mémoire utilisée pour les allocations dynamiques.

Dans `maps`, il apparaît souvent ainsi :

```text
55b1f4f4f000-55b1f4f70000 rw-p 00000000 00:00 0 [heap]
```

Le tas est utilisé par des fonctions comme :

```c
malloc()
calloc()
realloc()
free()
```

En Python, JavaScript, Java, Ruby ou Go, le runtime gère lui-même les allocations, mais repose finalement sur des mécanismes mémoire du système.

Nous pouvons chercher le tas :

```bash
grep '\[heap\]' /proc/$pid/maps
```

Un processus peut avoir un tas plus ou moins grand selon son comportement.


### 6.4.3. La pile : `[stack]`

La pile du thread principal apparaît souvent ainsi :

```text
7ffcc7b4d000-7ffcc7b6e000 rw-p 00000000 00:00 0 [stack]
```

Nous pouvons la chercher :

```bash
grep '\[stack\]' /proc/$pid/maps
```

La pile contient notamment :

- les variables locales ;
    
- les adresses de retour ;
    
- les frames d’appel ;
    
- certaines données temporaires.
    

Chaque thread peut avoir sa propre pile.

Dans les processus multi-thread, nous pouvons observer les stacks par thread dans :

```bash
/proc/<PID>/task/<TID>/maps
```


### 6.4.4. Bibliothèques partagées

Les bibliothèques partagées apparaissent avec leur chemin.

Exemples :

```text
/usr/lib/x86_64-linux-gnu/libc.so.6
/usr/lib/x86_64-linux-gnu/libpthread.so.0
/usr/lib/x86_64-linux-gnu/libm.so.6
```

Nous pouvons les lister :

```bash
awk '{print $6}' /proc/$pid/maps 2>/dev/null | grep '^/' | sort -u
```

Cela nous permet de voir les bibliothèques chargées par le processus.

C’est utile pour diagnostiquer :

- une mauvaise version de bibliothèque ;
    
- un conflit de dépendances ;
    
- un binaire qui charge une bibliothèque inattendue ;
    
- une application qui utilise beaucoup de modules natifs.
    


### 6.4.5. Mémoire anonyme

Certaines lignes n’ont pas de chemin.

Exemple :

```text
7f83d5c00000-7f83d5c21000 rw-p 00000000 00:00 0
```

Nous parlons souvent de mémoire anonyme.

Elle n’est pas directement associée à un fichier.

Elle peut correspondre à :

- des allocations dynamiques ;
    
- des zones réservées par le runtime ;
    
- des piles de threads ;
    
- des buffers ;
    
- des zones internes de bibliothèques.
    

Une forte quantité de mémoire anonyme peut être normale pour certaines applications, mais elle peut aussi signaler une fuite mémoire.


### 6.4.6. Mappings de fichiers

Un processus peut mapper un fichier en mémoire.

Cela peut être fait avec l’appel système :

```c
mmap()
```

Un fichier mappé peut être lu comme s’il était en mémoire.

Cela sert notamment pour :

- charger un exécutable ;
    
- charger des bibliothèques partagées ;
    
- accéder efficacement à de gros fichiers ;
    
- partager de la mémoire entre processus ;
    
- implémenter certaines bases de données ;
    
- manipuler des fichiers volumineux.
    

Dans `maps`, ces mappings apparaissent avec le chemin du fichier.


## 6.5. Comprendre `maps` par l’exemple

### 6.5.1. Afficher une vue simplifiée

Nous pouvons afficher les zones mémoire avec leurs permissions et chemins :

```bash
awk '{print $1, $2, $6}' /proc/$pid/maps | head -30
```

Attention : toutes les lignes n’ont pas forcément de sixième champ.

Nous pouvons écrire une version plus lisible :

```bash
awk '
{
  path = $6
  if (path == "") path = "[anonymous]"
  print $1, $2, path
}
' /proc/$pid/maps | head -30
```


### 6.5.2. Lister les zones exécutables

Les zones exécutables ont la permission `x`.

```bash
awk '$2 ~ /x/ { print }' /proc/$pid/maps
```

Nous y trouvons généralement :

- le binaire principal ;
    
- les bibliothèques partagées ;
    
- le linker dynamique ;
    
- parfois des zones JIT pour certains runtimes.
    

Pour des environnements comme JavaScript, Java, .NET ou certains moteurs de base de données, la présence de mémoire exécutable générée dynamiquement peut être normale.


### 6.5.3. Lister les zones modifiables

Les zones en écriture ont la permission `w`.

```bash
awk '$2 ~ /w/ { print }' /proc/$pid/maps | head
```

Nous y trouvons souvent :

- le tas ;
    
- la pile ;
    
- des zones anonymes ;
    
- des zones de données modifiables ;
    
- certaines zones partagées.
    


### 6.5.4. Chercher les fichiers supprimés mappés

Comme pour les descripteurs, un processus peut encore mapper un fichier supprimé.

Nous pouvons chercher :

```bash
grep deleted /proc/$pid/maps
```

Exemple :

```text
7f1234000000-7f1234200000 r-xp ... /usr/lib/libold.so (deleted)
```

Cela peut arriver après une mise à jour de bibliothèque.

Le processus continue d’utiliser l’ancienne version tant qu’il n’est pas redémarré.


## 6.6. `/proc/<PID>/smaps` : statistiques détaillées par mapping

### 6.6.1. Lire `smaps`

Le fichier :

```bash
/proc/<PID>/smaps
```

donne des statistiques détaillées pour chaque zone mémoire.

Nous pouvons lire :

```bash
cat /proc/$pid/smaps | head -60
```

La sortie ressemble à `maps`, mais chaque mapping est suivi de nombreuses lignes.

Extrait :

```text
55b1f3b2f000-55b1f3b33000 r-xp 00002000 08:02 393324 /usr/bin/sleep
Size:                  16 kB
KernelPageSize:         4 kB
MMUPageSize:            4 kB
Rss:                   16 kB
Pss:                    4 kB
Shared_Clean:          16 kB
Shared_Dirty:           0 kB
Private_Clean:          0 kB
Private_Dirty:          0 kB
Referenced:            16 kB
Anonymous:              0 kB
Swap:                   0 kB
```

`smaps` est beaucoup plus détaillé que `maps`.


### 6.6.2. Coût de lecture de `smaps`

Lire `smaps` peut coûter plus cher que lire `maps`.

Sur un gros processus avec beaucoup de mappings, `smaps` peut être volumineux et demander un travail réel au noyau.

Nous évitons donc de lire `smaps` en boucle très fréquente en production sans raison.

Pour une synthèse plus légère, nous préférons parfois :

```bash
/proc/<PID>/smaps_rollup
```

que nous étudions ensuite.


### 6.6.3. Champs importants de `smaps`

Les champs les plus utiles sont :

|Champ|Signification|
|---|---|
|`Size`|taille virtuelle du mapping|
|`Rss`|mémoire résidente en RAM|
|`Pss`|part proportionnelle de mémoire|
|`Shared_Clean`|mémoire partagée propre|
|`Shared_Dirty`|mémoire partagée modifiée|
|`Private_Clean`|mémoire privée propre|
|`Private_Dirty`|mémoire privée modifiée|
|`Referenced`|mémoire récemment référencée|
|`Anonymous`|mémoire anonyme|
|`Swap`|mémoire de ce mapping partie en swap|

Nous n’avons pas besoin de tous les champs pour un premier diagnostic, mais nous devons comprendre les principaux.


## 6.7. RSS, PSS, mémoire partagée et mémoire privée

### 6.7.1. RSS : Resident Set Size

Le RSS représente la mémoire du processus actuellement présente en RAM.

Dans `status`, nous avons :

```bash
grep '^VmRSS:' /proc/$pid/status
```

Dans `smaps`, chaque mapping a une ligne :

```text
Rss: 16 kB
```

Le RSS est utile, mais il peut être trompeur.

Pourquoi ? Parce qu’il compte la mémoire partagée dans chaque processus qui l’utilise.


### 6.7.2. Exemple de limite du RSS

Supposons qu’une bibliothèque partagée de 10 Mio soit utilisée par 20 processus.

Le RSS de chaque processus peut inclure ces 10 Mio.

Si nous additionnons naïvement les RSS des 20 processus, nous comptons potentiellement plusieurs fois la même mémoire.

Nous obtenons alors une surestimation.

C’est pourquoi le RSS est utile pour observer un processus, mais dangereux pour additionner la mémoire d’un ensemble de processus.


### 6.7.3. PSS : Proportional Set Size

Le PSS répartit la mémoire partagée entre les processus qui l’utilisent.

Si une page mémoire est partagée par 4 processus, chaque processus se voit attribuer un quart de cette page dans son PSS.

Cela donne une estimation plus juste de la contribution mémoire réelle d’un processus.

Dans `smaps`, nous voyons :

```text
Pss: 4 kB
```

Pour additionner la mémoire de plusieurs processus, le PSS est souvent plus pertinent que le RSS.


### 6.7.4. Mémoire privée

La mémoire privée est propre à un processus.

Dans `smaps`, elle apparaît notamment sous :

```text
Private_Clean
Private_Dirty
```

- `Private_Clean` : mémoire privée propre, potentiellement relisible depuis un fichier ;
    
- `Private_Dirty` : mémoire privée modifiée, réellement propre au processus.
    

La mémoire `Private_Dirty` est souvent intéressante pour détecter la mémoire réellement consommée par une application.


### 6.7.5. Mémoire partagée

La mémoire partagée apparaît sous :

```text
Shared_Clean
Shared_Dirty
```

Elle peut correspondre à :

- bibliothèques partagées ;
    
- pages partagées entre processus ;
    
- mémoire partagée explicite ;
    
- fichiers mappés partagés.
    

Une forte mémoire partagée n’est pas forcément problématique.

Au contraire, elle peut être un mécanisme d’efficacité.


## 6.8. `/proc/<PID>/smaps_rollup` : synthèse mémoire

### 6.8.1. Lire `smaps_rollup`

Sur les noyaux modernes, nous pouvons lire :

```bash
cat /proc/$pid/smaps_rollup
```

Exemple :

```text
55b1f3b2d000-7ffcc7b6e000 ---p 00000000 00:00 0 [rollup]
Rss:                1216 kB
Pss:                 180 kB
Pss_Dirty:           104 kB
Shared_Clean:       1112 kB
Shared_Dirty:          0 kB
Private_Clean:         0 kB
Private_Dirty:       104 kB
Referenced:         1216 kB
Anonymous:           104 kB
Swap:                  0 kB
```

Ce fichier agrège les informations de `smaps` pour l’ensemble du processus.

Il est très pratique pour obtenir une vue synthétique.


### 6.8.2. Extraire les champs clés

Nous pouvons lire seulement quelques champs :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Swap):' /proc/$pid/smaps_rollup
```

Exemple :

```text
Rss:                1216 kB
Pss:                 180 kB
Shared_Clean:       1112 kB
Private_Dirty:       104 kB
Swap:                  0 kB
```

Ces informations donnent une vision plus fine que `VmRSS`.


### 6.8.3. Quand préférons-nous `smaps_rollup` ?

Nous préférons `smaps_rollup` lorsque :

- nous voulons une synthèse mémoire du processus ;
    
- nous ne voulons pas analyser mapping par mapping ;
    
- nous voulons une estimation plus fine que `status` ;
    
- nous voulons observer `Pss` ;
    
- nous cherchons une contribution mémoire plus réaliste.
    

Nous préférons `smaps` lorsque :

- nous voulons comprendre quelle zone mémoire consomme ;
    
- nous cherchons une bibliothèque ou un mapping particulier ;
    
- nous analysons une fuite mémoire fine ;
    
- nous voulons distinguer précisément les zones anonymes, fichiers mappés, heap, stack.
    


## 6.9. Calculer une synthèse mémoire avec `awk`

### 6.9.1. Additionner les RSS dans `smaps`

Nous pouvons additionner les lignes `Rss` :

```bash
awk '/^Rss:/ { total += $2 } END { printf "RSS total: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Cela doit être proche de `VmRSS`.


### 6.9.2. Additionner les PSS

```bash
awk '/^Pss:/ { total += $2 } END { printf "PSS total: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Le PSS est souvent plus intéressant pour comprendre la part réelle du processus.


### 6.9.3. Additionner la mémoire privée dirty

```bash
awk '/^Private_Dirty:/ { total += $2 } END { printf "Private_Dirty: %.2f MiB\n", total/1024 }' /proc/$pid/smaps
```

Cette valeur est utile pour savoir quelle mémoire modifiée appartient vraiment au processus.


### 6.9.4. Regrouper par fichier mappé

Nous pouvons produire une vue approximative par chemin :

```bash
awk '
/^[0-9a-f]+-[0-9a-f]+/ {
    path=$6
    if (path=="") path="[anonymous]"
}
/^Rss:/ {
    rss[path]+=$2
}
END {
    for (p in rss) {
        printf "%10.2f MiB  %s\n", rss[p]/1024, p
    }
}
' /proc/$pid/smaps | sort -n | tail
```

Cela permet d’identifier les chemins ou catégories qui contribuent le plus au RSS.

Nous restons prudents : ce script est pédagogique et peut être amélioré.


## 6.10. `/proc/<PID>/limits` : limites de ressources

### 6.10.1. Lire les limites

Le fichier :

```bash
cat /proc/<PID>/limits
```

affiche les limites de ressources du processus.

Exemple :

```text
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             30995                30995                processes
Max open files            1024                 1048576              files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
```

Ces limites correspondent aux limites de ressources applicables au processus.


### 6.10.2. Limite souple et limite dure

Nous distinguons :

|Type|Signification|
|---|---|
|Soft limit|limite actuellement appliquée|
|Hard limit|limite maximale que le processus peut généralement se fixer sans privilèges supplémentaires|

Un processus peut souvent augmenter sa limite souple jusqu’à la limite dure.

Pour aller au-delà de la limite dure, il faut généralement des droits supplémentaires.


### 6.10.3. Limites importantes pour la mémoire

Plusieurs limites peuvent impacter la mémoire :

|Limite|Rôle|
|---|---|
|`Max data size`|taille maximale du segment de données|
|`Max stack size`|taille maximale de pile|
|`Max resident set`|limite RSS, rarement utilisée comme contrainte stricte moderne|
|`Max locked memory`|mémoire verrouillable en RAM|
|`Max address space`|taille maximale de l’espace d’adressage virtuel|

La limite la plus visible dans de nombreux cas est `Max stack size`.

Exemple :

```bash
grep 'Max stack size' /proc/$pid/limits
```

Sortie possible :

```text
Max stack size            8388608              unlimited            bytes
```

Cela correspond à une pile de 8 Mio par défaut dans beaucoup de configurations.


### 6.10.4. Lien avec `ulimit`

Dans le shell, nous pouvons afficher les limites avec :

```bash
ulimit -a
```

Les processus enfants héritent généralement des limites du shell ou du service qui les lance.

Exemple :

```bash
ulimit -n
```

affiche la limite du nombre de fichiers ouverts.

Nous faisons le lien avec le chapitre précédent :

```text
Une erreur “Too many open files” est souvent liée à la limite Max open files.
```


### 6.10.5. Limites systemd

Pour un service lancé par systemd, les limites peuvent être définies dans l’unité.

Exemples :

```ini
LimitNOFILE=65536
LimitNPROC=4096
LimitSTACK=8M
```

Après modification, nous rechargeons systemd :

```bash
sudo systemctl daemon-reload
sudo systemctl restart mon-service
```

Puis nous vérifions dans :

```bash
cat /proc/<PID>/limits
```

Nous ne nous contentons pas de lire le fichier de configuration : nous vérifions la limite réellement appliquée au processus.


## 6.11. Diagnostiquer une fuite mémoire

### 6.11.1. Symptôme

Une fuite mémoire se manifeste souvent par une croissance continue de la mémoire consommée par un processus.

Nous pouvons observer :

```bash
watch -n 2 "grep -E '^(VmSize|VmRSS|RssAnon):' /proc/$pid/status"
```

Si `VmRSS` ou `RssAnon` augmente continuellement sans redescendre, nous suspectons une fuite ou une accumulation de données.


### 6.11.2. Observer `smaps_rollup`

Nous pouvons aussi observer :

```bash
watch -n 2 "grep -E '^(Rss|Pss|Private_Dirty|Anonymous|Swap):' /proc/$pid/smaps_rollup"
```

Les champs intéressants sont :

- `Rss` ;
    
- `Pss` ;
    
- `Private_Dirty` ;
    
- `Anonymous` ;
    
- `Swap`.
    

Si `Private_Dirty` et `Anonymous` augmentent continuellement, cela peut indiquer que le processus accumule de la mémoire propre.


### 6.11.3. Différencier fuite et cache applicatif

Une croissance mémoire n’est pas toujours une fuite.

Elle peut correspondre à :

- un cache applicatif volontaire ;
    
- un warm-up de runtime ;
    
- un JIT ;
    
- un pool de connexions ;
    
- un chargement de données ;
    
- une file d’attente interne ;
    
- une montée en charge normale.
    

Nous devons croiser avec :

- les logs applicatifs ;
    
- les métriques métier ;
    
- le trafic ;
    
- le nombre de requêtes ;
    
- les tailles de files ;
    
- les métriques du runtime.
    

`/proc` donne des symptômes, pas toujours la cause applicative exacte.


## 6.12. Cas particulier des runtimes modernes

### 6.12.1. Java

Un processus Java peut réserver un espace mémoire virtuel important.

Nous pouvons voir un `VmSize` très élevé sans que la RAM soit réellement utilisée.

Il faut croiser avec :

```bash
jcmd
jstat
jmap
```

et les options de JVM :

```text
-Xms
-Xmx
-XX:MaxRAMPercentage
```

`/proc` nous donne la vue noyau, mais la JVM a sa propre gestion mémoire.


### 6.12.2. Node.js

Node.js utilise le moteur V8.

La mémoire observée dans `/proc` inclut :

- le heap JavaScript ;
    
- les buffers natifs ;
    
- les bibliothèques ;
    
- le code JIT ;
    
- les allocations C++ natives ;
    
- les modules natifs.
    

Nous ne pouvons pas conclure uniquement avec `VmRSS`.

Nous devons parfois compléter avec :

```bash
node --inspect
process.memoryUsage()
```

ou des dumps mémoire.


### 6.12.3. Python

Python peut conserver de la mémoire via son allocateur interne.

Après libération d’objets Python, la mémoire peut ne pas être immédiatement rendue au système.

Nous pouvons donc observer un RSS stable ou élevé même si l’application a libéré des objets.

Nous complétons avec :

- `tracemalloc` ;
    
- `objgraph` ;
    
- profils mémoire ;
    
- métriques applicatives.
    


### 6.12.4. Go

Go utilise son propre garbage collector.

Le runtime peut réserver de la mémoire, la réutiliser, et ne pas la rendre immédiatement au système.

Nous complétons avec :

- `pprof` ;
    
- métriques runtime ;
    
- `GOMEMLIMIT` ;
    
- `GOGC`.
    

Encore une fois, `/proc` est indispensable, mais il ne remplace pas les outils spécifiques du runtime.


## 6.13. Mémoire et conteneurs

### 6.13.1. Limite du point de vue `/proc`

Dans un conteneur, `/proc/<PID>/status`, `maps` et `smaps` décrivent le processus.

Mais la limite mémoire réelle peut être imposée par les cgroups.

Un processus peut voir beaucoup de mémoire système dans `/proc/meminfo`, mais être limité à 512 Mio par son conteneur.

Nous devons donc croiser avec :

```bash
cat /proc/1/cgroup
```

et les fichiers de cgroups dans :

```bash
/sys/fs/cgroup
```


### 6.13.2. OOM dans les conteneurs

Lorsqu’un conteneur dépasse sa limite mémoire, il peut être tué par le mécanisme OOM.

Depuis l’application, nous pouvons voir :

- une croissance de RSS ;
    
- une pression mémoire ;
    
- puis une terminaison brutale.
    

Dans Kubernetes, cela apparaît souvent comme :

```text
OOMKilled
```

Pour diagnostiquer, nous croisons :

- `/proc/<PID>/status` ;
    
- `/proc/<PID>/smaps_rollup` ;
    
- les métriques cgroups ;
    
- les événements Kubernetes ;
    
- les logs du pod.
    


## 6.14. Sécurité et mémoire

### 6.14.1. Informations sensibles dans la mémoire

La mémoire d’un processus peut contenir :

- mots de passe ;
    
- tokens ;
    
- clés privées ;
    
- secrets applicatifs ;
    
- données utilisateurs ;
    
- contenu de requêtes ;
    
- données temporaires.
    

Certains fichiers de `/proc` peuvent exposer des informations sensibles si les permissions sont trop ouvertes.


### 6.14.2. `/proc/<PID>/mem`

Il existe un fichier :

```bash
/proc/<PID>/mem
```

qui représente la mémoire du processus.

Nous ne l’utilisons pas dans ce cours comme outil de lecture directe.

Son accès est fortement contrôlé, car il permettrait de lire ou modifier la mémoire d’un processus.

Nous retenons simplement :

```text
/proc/<PID>/mem est très sensible et ne doit pas être manipulé sans cadre précis.
```


### 6.14.3. Dumps mémoire

Un core dump peut contenir toute la mémoire d’un processus au moment d’un crash.

Cela peut aider au debugging, mais cela peut aussi exposer des secrets.

La limite liée aux core dumps apparaît dans :

```bash
grep 'Max core file size' /proc/$pid/limits
```

Si la limite est `0`, les core dumps sont désactivés pour ce processus.

Nous devons traiter les dumps mémoire comme des données sensibles.


## 6.15. Cas pratique complet : analyser la mémoire d’un processus

### 6.15.1. Situation

Nous avons un processus applicatif suspecté de consommer trop de mémoire.

Nous avons son PID :

```bash
pid=1234
```

Nous voulons établir un premier diagnostic.


### 6.15.2. Identifier le processus

Nous commençons par :

```bash
cat /proc/$pid/comm
tr '\0' ' ' < /proc/$pid/cmdline
echo
grep -E '^(Name|State|Pid|PPid|Threads):' /proc/$pid/status
```

Nous vérifions que nous analysons le bon processus.


### 6.15.3. Lire la mémoire globale du processus

Nous lisons :

```bash
grep -E '^(VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/$pid/status
```

Nous interprétons :

- `VmSize` : espace virtuel ;
    
- `VmRSS` : mémoire résidente ;
    
- `RssAnon` : mémoire anonyme ;
    
- `RssFile` : mémoire liée à des fichiers ;
    
- `RssShmem` : mémoire partagée.
    


### 6.15.4. Lire la synthèse fine

Si disponible :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Anonymous|Swap):' /proc/$pid/smaps_rollup
```

Nous obtenons une vision plus fine.

Nous cherchons notamment :

- PSS élevé ;
    
- Private_Dirty élevé ;
    
- Anonymous élevé ;
    
- Swap non nul.
    


### 6.15.5. Identifier les mappings importants

Nous regardons les plus gros contributeurs :

```bash
awk '
/^[0-9a-f]+-[0-9a-f]+/ {
    path=$6
    if (path=="") path="[anonymous]"
}
/^Rss:/ {
    rss[path]+=$2
}
END {
    for (p in rss) {
        printf "%10.2f MiB  %s\n", rss[p]/1024, p
    }
}
' /proc/$pid/smaps | sort -n | tail -20
```

Nous cherchons :

- mémoire anonyme importante ;
    
- gros fichiers mappés ;
    
- bibliothèques inattendues ;
    
- segments heap importants ;
    
- fichiers supprimés encore mappés.
    


### 6.15.6. Observer l’évolution

Nous observons dans le temps :

```bash
watch -n 5 "grep -E '^(VmRSS|RssAnon):' /proc/$pid/status"
```

ou :

```bash
watch -n 5 "grep -E '^(Rss|Pss|Private_Dirty|Anonymous|Swap):' /proc/$pid/smaps_rollup"
```

Si la mémoire augmente continuellement, nous formulons une hypothèse de fuite ou d’accumulation.


### 6.15.7. Formuler une conclusion

Exemple de conclusion prudente :

```text
Nous observons que le processus possède un VmRSS élevé, mais qu’une partie importante est partagée. Le PSS est plus modéré. La mémoire privée dirty reste stable. Nous ne concluons donc pas à une fuite mémoire immédiate.
```

Autre exemple :

```text
Nous observons une croissance continue de RssAnon et Private_Dirty. Cette évolution suggère une accumulation de mémoire propre au processus. Nous devons compléter l’analyse avec les outils du runtime applicatif.
```


## 6.16. Scripts utiles

### 6.16.1. Synthèse mémoire d’un PID

```bash
##!/usr/bin/env bash

pid="$1"

if [ -z "$pid" ]; then
    echo "Usage: $0 <PID>" >&2
    exit 1
fi

if [ ! -d "/proc/$pid" ]; then
    echo "Processus introuvable : $pid" >&2
    exit 1
fi

echo "Processus"
echo "---------"
cat "/proc/$pid/comm" 2>/dev/null || true
tr '\0' ' ' < "/proc/$pid/cmdline" 2>/dev/null
echo
echo

echo "Status mémoire"
echo "--------------"
grep -E '^(VmPeak|VmSize|VmRSS|RssAnon|RssFile|RssShmem):' "/proc/$pid/status" 2>/dev/null || true
echo

if [ -r "/proc/$pid/smaps_rollup" ]; then
    echo "smaps_rollup"
    echo "-------------"
    grep -E '^(Rss|Pss|Private_Dirty|Shared_Clean|Anonymous|Swap):' "/proc/$pid/smaps_rollup"
else
    echo "smaps_rollup indisponible"
fi
```

Nous pouvons le sauvegarder sous :

```bash
mem-summary.sh
```

Puis :

```bash
chmod +x mem-summary.sh
./mem-summary.sh 1234
```


### 6.16.2. Surveiller la mémoire d’un processus

```bash
watch -n 2 "grep -E '^(VmSize|VmRSS|RssAnon|RssFile|RssShmem):' /proc/<PID>/status"
```

Nous remplaçons `<PID>` par le PID réel.


### 6.16.3. Afficher les bibliothèques chargées

```bash
awk '{print $6}' /proc/<PID>/maps 2>/dev/null | grep '^/' | sort -u
```

Cette commande liste les fichiers mappés depuis le disque.


### 6.16.4. Chercher les fichiers mappés supprimés

```bash
grep deleted /proc/<PID>/maps
```

Si nous trouvons des bibliothèques ou fichiers supprimés, nous pouvons envisager de redémarrer le processus pour qu’il recharge les versions actuelles.


## 6.17. Pièges et limites

### 6.17.1. Ne pas confondre virtuel et physique

Un très grand `VmSize` peut être normal.

Certains programmes réservent beaucoup d’espace virtuel sans utiliser réellement autant de RAM.

Nous regardons aussi :

- `VmRSS` ;
    
- `RssAnon` ;
    
- `Pss` ;
    
- `Private_Dirty`.
    


### 6.17.2. Ne pas additionner naïvement les RSS

Additionner les RSS de plusieurs processus peut surestimer la mémoire utilisée, car la mémoire partagée est comptée plusieurs fois.

Pour une estimation plus juste, nous préférons le PSS.


### 6.17.3. `smaps` peut être coûteux

Lire `smaps` trop fréquemment sur beaucoup de processus peut coûter cher.

Pour une supervision régulière, nous préférons des outils adaptés ou des métriques déjà agrégées.


### 6.17.4. Les runtimes ont leur propre logique

Java, Go, Python, Node.js et d’autres runtimes gèrent leur mémoire de manière spécifique.

`/proc` donne la vue du noyau, mais pas toujours la cause applicative.

Nous complétons avec les outils du langage.


### 6.17.5. Les conteneurs ajoutent une couche

Dans un conteneur, nous devons croiser `/proc` avec les cgroups.

Sinon, nous risquons de mal interpréter les limites mémoire.


## 6.18. Exercices

### Exercice 1 — Observer `maps` sur le shell courant

Nous exécutons :

```bash
cat /proc/$$/maps | head -30
```

Nous répondons :

1. Quelles sont les permissions observées ?
    
2. Voyons-nous le binaire du shell ?
    
3. Voyons-nous des bibliothèques partagées ?
    
4. Voyons-nous `[heap]` ?
    
5. Voyons-nous `[stack]` ?
    


### Exercice 2 — Comparer `VmSize` et `VmRSS`

Nous exécutons :

```bash
grep -E '^(VmSize|VmRSS):' /proc/$$/status
```

Nous répondons :

1. Quelle valeur est la plus élevée ?
    
2. Pourquoi ?
    
3. Laquelle est plus proche de la RAM réellement utilisée ?
    
4. Pourquoi même `VmRSS` reste imparfait ?
    


### Exercice 3 — Créer un processus qui alloue de la mémoire

Nous lançons :

```bash
python3 -c 'import time; data = bytearray(200 * 1024 * 1024); time.sleep(300)' &
pid=$!
```

Nous observons :

```bash
grep -E '^(VmSize|VmRSS|RssAnon):' /proc/$pid/status
```

Puis :

```bash
grep -E '^(Rss|Pss|Private_Dirty|Anonymous):' /proc/$pid/smaps_rollup
```

Nous expliquons pourquoi `RssAnon` et `Private_Dirty` augmentent.

Nous terminons :

```bash
kill $pid
```


### Exercice 4 — Lister les bibliothèques chargées

Nous choisissons un processus :

```bash
pid=$$
```

Puis :

```bash
awk '{print $6}' /proc/$pid/maps 2>/dev/null | grep '^/' | sort -u
```

Nous répondons :

1. Quelles bibliothèques voyons-nous ?
    
2. Voyons-nous `libc` ?
    
3. Voyons-nous le linker dynamique ?
    
4. Pourquoi un processus charge-t-il plusieurs bibliothèques ?
    


### Exercice 5 — Observer les limites

Nous exécutons :

```bash
cat /proc/$$/limits
```

Nous répondons :

1. Quelle est la limite de pile ?
    
2. Quelle est la limite du nombre de fichiers ouverts ?
    
3. Les core dumps sont-ils autorisés ?
    
4. Quelle est la différence entre limite souple et limite dure ?


### Exercice 6 — Détecter une croissance mémoire

Nous lançons un programme Python qui alloue progressivement :

```bash
python3 - <<'PY' &
import time
data = []
while True:
    data.append(bytearray(10 * 1024 * 1024))
    time.sleep(2)
PY
pid=$!
```

Nous observons :

```bash
watch -n 2 "grep -E '^(VmRSS|RssAnon):' /proc/$pid/status"
```

Nous répondons :

1. Quelles valeurs augmentent ?
    
2. Pourquoi ?
    
3. En quoi cela ressemble-t-il à une fuite mémoire ?
    
4. Quelles informations supplémentaires faudrait-il pour conclure ?
    

Nous terminons :

```bash
kill $pid
```


## 6.19. Ce que nous devons retenir

Nous retenons les points suivants :

1. Un processus possède un espace mémoire virtuel.
    
2. La mémoire virtuelle ne correspond pas directement à la RAM consommée.
    
3. `/proc/<PID>/maps` décrit les zones mémoire du processus.
    
4. Les permissions `r`, `w`, `x`, `p`, `s` décrivent les droits et le type de mapping.
    
5. `[heap]` correspond au tas et `[stack]` à la pile.
    
6. Les bibliothèques partagées apparaissent comme des fichiers mappés.
    
7. La mémoire anonyme est souvent liée aux allocations dynamiques.
    
8. `/proc/<PID>/smaps` donne des statistiques détaillées par mapping.
    
9. `/proc/<PID>/smaps_rollup` donne une synthèse pratique.
    
10. Le RSS mesure la mémoire résidente, mais peut compter de la mémoire partagée plusieurs fois.
    
11. Le PSS répartit la mémoire partagée et donne une estimation souvent plus juste.
    
12. `Private_Dirty` est utile pour estimer la mémoire réellement propre au processus.
    
13. `/proc/<PID>/limits` expose les limites de ressources du processus.
    
14. Une croissance mémoire n’est pas toujours une fuite : nous devons croiser avec le contexte applicatif.
    
15. Dans les conteneurs, nous devons compléter `/proc` avec les cgroups.
    


## Conclusion du chapitre 6

Nous savons maintenant analyser la mémoire d’un processus avec `/proc`.

Nous avons appris à lire la cartographie mémoire avec `maps`, à interpréter les permissions des mappings, à identifier le tas, la pile, les bibliothèques partagées et la mémoire anonyme. Nous avons aussi étudié `smaps` et `smaps_rollup`, qui permettent d’aller au-delà de `VmRSS` pour mieux comprendre la mémoire réellement utilisée par un processus.

Nous retenons surtout qu’une analyse mémoire sérieuse demande de distinguer mémoire virtuelle, mémoire résidente, mémoire partagée, mémoire privée et mémoire proportionnelle. Sans ces distinctions, nous risquons de diagnostiquer à tort une fuite mémoire ou une surconsommation.

Dans le chapitre suivant, nous étudions le réseau à travers `/proc/net`, notamment les interfaces, les sockets TCP, UDP et Unix.

---
> [!info] Livre « proc » — chapitre 6/11
> [[proc — Sommaire|Sommaire]] · [[proc — 05 — Fichiers ouverts et descripteurs|← 05 — Fichiers ouverts et descripteurs]] · [[proc — 07 — Réseau et proc|07 — Réseau et proc →]]
