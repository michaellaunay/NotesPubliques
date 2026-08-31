---
schema_version: 1
uid: 01M1BQ629VZ3RN8RCDSAG7FYA6
titre: "proc — 09 — Sécurité, permissions et isolation"
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
resume: "Chapitre 9 sur 11 du livre « proc » : Sécurité, permissions et isolation. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-18."
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

> [!info] Livre « proc » — chapitre 9/11
> [[proc — Sommaire|Sommaire]] · [[proc — 08 — Paramètres noyau avec proc sys|← 08 — Paramètres noyau avec proc sys]] · [[proc — 10 — proc et les outils système|10 — proc et les outils système →]]

# Chapitre 9 — Sécurité, permissions et isolation
## Objectifs du chapitre

Dans ce chapitre, nous étudions `/proc` sous l’angle de la sécurité.

Jusqu’ici, nous avons utilisé `/proc` comme outil d’observation et de diagnostic. Nous avons lu des informations sur le système, les processus, la mémoire, les fichiers ouverts, le réseau et les paramètres noyau.

Mais cette richesse a une contrepartie : `/proc` expose beaucoup d’informations. Certaines sont anodines, d’autres peuvent être sensibles.

À la fin de ce chapitre, nous savons :

- identifier les informations sensibles exposées par `/proc` ;
    
- comprendre les permissions appliquées aux fichiers de `/proc` ;
    
- expliquer les risques liés à `cmdline`, `environ`, `fd`, `maps` et `mem` ;
    
- comprendre l’option de montage `hidepid` ;
    
- analyser les risques sur un serveur multi-utilisateurs ;
    
- comprendre le comportement de `/proc` dans les conteneurs ;
    
- faire le lien avec les namespaces Linux ;
    
- appliquer de bonnes pratiques de durcissement.
    


## 9.1. Pourquoi `/proc` pose des questions de sécurité

### 9.1.1. Une interface très informative

`/proc` est précieux parce qu’il rend visibles de nombreuses informations internes du système.

Nous pouvons y trouver :

```text
processus actifs
lignes de commande
variables d’environnement
fichiers ouverts
sockets réseau
mappings mémoire
paramètres noyau
statistiques système
informations CPU
informations mémoire
```

Cette visibilité est très utile pour l’administration système.

Mais du point de vue sécurité, une information utile à l’administrateur peut aussi être utile à un attaquant.


### 9.1.2. Le problème de l’énumération

Sur un système mal protégé, un utilisateur local peut parfois observer :

```bash
ps aux
ls /proc
cat /proc/<PID>/cmdline
tr '\0' '\n' < /proc/<PID>/environ
ls -l /proc/<PID>/fd
```

Cela peut lui permettre de découvrir :

- quels services tournent ;
    
- quels utilisateurs sont connectés ;
    
- quelles applications sont lancées ;
    
- quels fichiers sont ouverts ;
    
- quels ports ou sockets sont utilisés ;
    
- quels chemins applicatifs sont présents ;
    
- parfois des secrets mal protégés.
    

Nous comprenons donc que `/proc` peut faciliter la reconnaissance locale.


## 9.2. Informations sensibles exposées par `/proc`

### 9.2.1. La ligne de commande : `cmdline`

Le fichier :

```bash
/proc/<PID>/cmdline
```

expose la ligne de commande utilisée pour lancer un processus.

Nous l’affichons de manière lisible avec :

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
echo
```

Le problème apparaît lorsque des secrets sont passés en arguments.

Exemples de mauvaises pratiques :

```bash
mysql -u admin -pMonMotDePasse
curl -H "Authorization: Bearer TOKEN_SECRET" https://api.example.org
python app.py --password secret
backup --s3-secret-key ABCDEF
```

Ces informations peuvent être visibles dans `/proc/<PID>/cmdline`, mais aussi dans des outils comme `ps`.

Nous retenons :

```text
Nous ne passons pas de secrets en arguments de ligne de commande.
```


### 9.2.2. Les variables d’environnement : `environ`

Le fichier :

```bash
/proc/<PID>/environ
```

contient les variables d’environnement du processus.

Nous les lisons avec :

```bash
tr '\0' '\n' < /proc/<PID>/environ
```

Les variables d’environnement peuvent contenir :

```text
DATABASE_URL
POSTGRES_PASSWORD
MYSQL_PASSWORD
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
GITHUB_TOKEN
JWT_SECRET
API_KEY
```

Elles sont pratiques, mais elles ne sont pas automatiquement secrètes.

Si les permissions permettent leur lecture, un autre utilisateur ou un processus compromis peut récupérer ces valeurs.

Nous retenons :

```text
Les variables d’environnement sont une convention pratique de configuration.
Elles ne remplacent pas un vrai mécanisme de gestion de secrets.
```


### 9.2.3. Les fichiers ouverts : `fd`

Le dossier :

```bash
/proc/<PID>/fd
```

expose les descripteurs ouverts.

Il peut révéler :

- fichiers de configuration ;
    
- fichiers temporaires ;
    
- logs ;
    
- sockets ;
    
- bases SQLite ;
    
- fichiers supprimés mais encore ouverts ;
    
- répertoires de travail ;
    
- chemins internes d’une application.
    

Exemple :

```bash
ls -l /proc/<PID>/fd
```

Sortie possible :

```text
0 -> /dev/null
1 -> /var/log/app/access.log
2 -> /var/log/app/error.log
3 -> /etc/app/secret.conf
4 -> socket:[123456]
5 -> /tmp/session-data
```

Cela peut donner beaucoup d’informations sur l’architecture d’un service.


### 9.2.4. La cartographie mémoire : `maps`

Le fichier :

```bash
/proc/<PID>/maps
```

expose les mappings mémoire du processus.

Il peut révéler :

- bibliothèques chargées ;
    
- chemins exacts des binaires ;
    
- versions de bibliothèques ;
    
- présence de modules natifs ;
    
- fichiers mappés ;
    
- exécutable supprimé mais encore chargé ;
    
- structure générale de l’espace mémoire.
    

Exemple :

```bash
cat /proc/<PID>/maps | head
```

Du point de vue d’un attaquant, ces informations peuvent aider à comprendre l’environnement d’exécution.

Les protections modernes, comme l’ASLR, limitent l’exploitation directe, mais l’exposition d’informations mémoire reste sensible.


### 9.2.5. La mémoire brute : `mem`

Le fichier :

```bash
/proc/<PID>/mem
```

représente la mémoire virtuelle du processus.

Son accès est fortement contrôlé.

Lire ou modifier ce fichier peut permettre :

- d’extraire des secrets en mémoire ;
    
- d’observer des données applicatives ;
    
- de modifier le comportement d’un processus ;
    
- de contourner certaines protections si les permissions sont insuffisantes.
    

Nous ne l’utilisons pas dans un usage courant.

Nous retenons :

```text
/proc/<PID>/mem est extrêmement sensible.
Son accès doit être strictement contrôlé.
```


### 9.2.6. Les liens `cwd`, `exe` et `root`

Les liens suivants peuvent aussi exposer des informations :

```bash
/proc/<PID>/cwd
/proc/<PID>/exe
/proc/<PID>/root
```

Ils permettent de savoir :

- d’où le processus travaille ;
    
- quel binaire il exécute ;
    
- quelle racine de système de fichiers il voit ;
    
- s’il tourne dans un conteneur ou un chroot ;
    
- s’il utilise un exécutable supprimé.
    

Exemple :

```bash
ls -l /proc/<PID>/exe
```

Sortie possible :

```text
/proc/1234/exe -> /opt/app/releases/2026-05-01/app-server
```

Ce type d’information peut révéler la stratégie de déploiement ou les chemins internes.


## 9.3. Permissions dans `/proc`

### 9.3.1. Observer les permissions

Nous pouvons afficher les permissions d’un processus :

```bash
ls -ld /proc/<PID>
ls -l /proc/<PID> | head
```

Certaines entrées sont lisibles par tous, d’autres uniquement par le propriétaire du processus ou par `root`.

Exemple :

```bash
ls -l /proc/$$/environ
```

Nous observons les droits Unix classiques :

```text
-r-------- 1 user user 0 mai 19 10:00 environ
```

Le fichier semble avoir une taille `0`, mais il peut produire du contenu lors de la lecture.


### 9.3.2. Le rôle de l’UID

Les règles d’accès dépendent notamment :

- de l’utilisateur réel du processus ;
    
- de l’utilisateur qui lit ;
    
- des permissions du fichier ;
    
- des capacités Linux ;
    
- des options de montage de `/proc` ;
    
- de mécanismes de sécurité comme Yama, AppArmor ou SELinux.
    

Un utilisateur peut généralement lire plus facilement les informations de ses propres processus que celles des processus d’un autre utilisateur.


### 9.3.3. Tester avec deux utilisateurs

Sur une machine de test, nous pouvons créer deux utilisateurs :

```bash
sudo adduser alice
sudo adduser bob
```

Nous lançons un processus sous `alice` :

```bash
sudo -u alice sleep 1000 &
pid=$!
```

Puis nous essayons de lire depuis un autre utilisateur :

```bash
sudo -u bob cat /proc/$pid/status
sudo -u bob tr '\0' '\n' < /proc/$pid/environ
sudo -u bob ls -l /proc/$pid/fd
```

Nous observons ce qui est autorisé ou refusé.

Cet exercice montre que `/proc` applique des restrictions, mais que le niveau exact dépend de la configuration du système.


## 9.4. Le mécanisme `ptrace` et Yama

### 9.4.1. Pourquoi parler de `ptrace` ?

Sous Linux, certains accès à `/proc/<PID>` sont liés aux règles de traçage et de debugging.

Le mécanisme `ptrace` permet à un processus d’en observer ou contrôler un autre, par exemple pour un débogueur comme `gdb`.

Certaines restrictions de `/proc` suivent une logique proche : si un processus n’a pas le droit d’en tracer un autre, il n’a pas forcément le droit de lire certaines informations sensibles.


### 9.4.2. Le paramètre `ptrace_scope`

Sur certaines distributions, le module de sécurité Yama expose :

```bash
cat /proc/sys/kernel/yama/ptrace_scope
```

Valeurs courantes :

|Valeur|Effet général|
|--:|---|
|`0`|comportement classique plus permissif|
|`1`|restrictions supplémentaires, souvent parent-enfant|
|`2`|seul un processus avec privilège peut tracer|
|`3`|traçage désactivé de manière stricte jusqu’au redémarrage|

La valeur exacte disponible dépend de la configuration du noyau.


### 9.4.3. Effet sur le debugging

Si `ptrace_scope` est restrictif, un utilisateur peut rencontrer des difficultés à attacher `gdb` à un processus qui ne descend pas directement de son shell.

Exemple :

```bash
gdb -p <PID>
```

peut échouer avec une erreur de permission.

Nous comprenons donc que les protections autour de `/proc` et du debugging sont liées.


## 9.5. Option de montage `hidepid`

### 9.5.1. Le problème sur les serveurs multi-utilisateurs

Sur un serveur partagé, nous voulons parfois éviter qu’un utilisateur voie les processus des autres utilisateurs.

Sans restriction particulière, un utilisateur peut souvent lister les PID :

```bash
ls /proc | grep -E '^[0-9]+$'
```

et voir certaines informations via :

```bash
ps aux
```

Cela peut révéler :

- noms d’utilisateurs ;
    
- commandes lancées ;
    
- services personnels ;
    
- scripts ;
    
- chemins de travail ;
    
- arguments de ligne de commande.
    

L’option `hidepid` permet de réduire cette visibilité.


### 9.5.2. Les valeurs de `hidepid`

`hidepid` est une option de montage de `procfs`.

Les valeurs classiques sont :

|Valeur|Effet|
|--:|---|
|`hidepid=0`|comportement classique, visibilité large|
|`hidepid=1`|les utilisateurs voient les PID, mais pas les détails sensibles des processus des autres|
|`hidepid=2`|les processus des autres utilisateurs sont masqués|
|`hidepid=invisible`|forme équivalente ou moderne selon systèmes pour masquer davantage|
|`hidepid=ptraceable`|visibilité alignée sur les processus traçables, selon noyau|

Dans un cours général, nous retenons surtout `0`, `1` et `2`.


### 9.5.3. Vérifier le montage actuel

Nous vérifions les options de montage de `/proc` :

```bash
findmnt /proc
```

ou :

```bash
mount | grep ' /proc '
```

Sortie possible :

```text
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
```

Si `hidepid` est actif, nous pouvons voir :

```text
hidepid=2
```

dans les options.


### 9.5.4. Monter `/proc` avec `hidepid`

Sur une machine de test, nous pouvons remonter `/proc` avec :

```bash
sudo mount -o remount,hidepid=2 /proc
```

Puis nous testons avec un utilisateur non privilégié :

```bash
ls /proc
ps aux
```

Nous observons que les processus des autres utilisateurs deviennent moins visibles.

Pour revenir au comportement classique :

```bash
sudo mount -o remount,hidepid=0 /proc
```

Nous faisons cela uniquement dans un environnement de test ou avec une bonne compréhension des effets.


### 9.5.5. Configuration persistante

Une configuration persistante peut se faire via `/etc/fstab`.

Exemple indicatif :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

Selon les distributions, systemd ou d’autres mécanismes peuvent influencer le montage de `/proc`.

Nous devons donc tester et vérifier après redémarrage :

```bash
findmnt /proc
```


### 9.5.6. Groupe autorisé

Dans certains environnements, nous voulons masquer les processus aux utilisateurs ordinaires, mais garder une visibilité pour un groupe d’administrateurs.

Des options de montage peuvent permettre d’indiquer un groupe autorisé, par exemple avec `gid`.

Le principe est :

```text
les utilisateurs ordinaires voient moins d’informations ;
les membres d’un groupe dédié gardent une visibilité d’administration.
```

La configuration exacte dépend de la distribution et de la version du noyau.


## 9.6. `/proc` dans les conteneurs

### 9.6.1. Pourquoi les conteneurs changent la lecture de `/proc`

Un conteneur utilise des mécanismes d’isolation du noyau Linux, notamment les namespaces.

Un processus dans un conteneur peut voir :

- une liste de processus différente ;
    
- une pile réseau différente ;
    
- des montages différents ;
    
- une racine différente ;
    
- des limites cgroups différentes.
    

Cela signifie que `/proc` vu depuis un conteneur n’est pas forcément le même que `/proc` vu depuis l’hôte.


### 9.6.2. Namespace PID

Le namespace PID contrôle la vue des processus.

Dans un conteneur, le processus principal peut avoir le PID `1` dans le conteneur, mais un autre PID sur l’hôte.

Dans le conteneur :

```bash
cat /proc/1/comm
```

peut afficher :

```text
node
```

ou :

```text
python
```

ou :

```text
bash
```

Sur l’hôte, le même processus peut avoir le PID `24531`.

Nous retenons :

```text
Le PID dépend du namespace depuis lequel nous observons.
```


### 9.6.3. Namespace réseau

Nous avons vu au chapitre 7 que :

```bash
/proc/net
```

dépend du namespace réseau.

Dans un conteneur, nous pouvons voir :

```bash
cat /proc/net/dev
```

et obtenir seulement :

```text
lo
eth0
```

alors que l’hôte possède :

```text
lo
eno1
docker0
br-...
veth...
wg0
```

La vue réseau du conteneur est isolée.


### 9.6.4. Namespace de montage

Le namespace de montage contrôle les points de montage visibles.

Dans un conteneur :

```bash
cat /proc/mounts
```

peut montrer une arborescence différente de celle de l’hôte.

Le lien :

```bash
/proc/<PID>/root
```

est particulièrement intéressant.

Depuis l’hôte, si nous regardons un processus de conteneur :

```bash
sudo ls -l /proc/<PID>/root
```

nous pouvons accéder à la racine vue par ce processus.

C’est puissant pour le diagnostic, mais sensible du point de vue sécurité.


## 9.7. Risques des conteneurs privilégiés

### 9.7.1. Le danger d’un `/proc` trop permissif

Dans un conteneur, monter `/proc` de manière trop permissive peut exposer des informations de l’hôte ou permettre des actions dangereuses.

Un conteneur privilégié peut avoir des capacités étendues.

Si nous combinons :

```text
conteneur privilégié
montage sensible de /proc
accès à /sys
accès à /dev
capabilities élevées
```

nous réduisons fortement l’isolation.


### 9.7.2. Exemple de mauvaise pratique

Un montage dangereux pourrait ressembler à :

```bash
docker run --privileged -v /proc:/host/proc image
```

Le conteneur peut alors observer des informations de l’hôte via `/host/proc`.

Selon les autres droits accordés, cela peut devenir très risqué.

Nous évitons ce type de montage sauf besoin exceptionnel et environnement maîtrisé.


### 9.7.3. Paramètres noyau depuis un conteneur

Certains fichiers de `/proc/sys` peuvent être visibles dans un conteneur.

Mais le noyau est partagé entre l’hôte et les conteneurs.

Modifier un paramètre noyau depuis un conteneur peut donc affecter l’hôte ou d’autres conteneurs, selon le paramètre et le namespace.

C’est pourquoi les runtimes de conteneurs limitent généralement les accès en écriture.

Nous retenons :

```text
Un conteneur partage le noyau avec l’hôte.
Les paramètres noyau ne sont pas des paramètres privés applicatifs ordinaires.
```


## 9.8. `/proc/sys` et sécurité

### 9.8.1. Paramètres de durcissement

Certains paramètres de `/proc/sys` ont un rôle de sécurité.

Exemples :

```bash
cat /proc/sys/kernel/randomize_va_space
cat /proc/sys/kernel/dmesg_restrict
cat /proc/sys/kernel/kptr_restrict
cat /proc/sys/fs/protected_symlinks
cat /proc/sys/fs/protected_hardlinks
```

Ils influencent :

- la randomisation mémoire ;
    
- l’accès aux logs noyau ;
    
- l’exposition des pointeurs noyau ;
    
- la protection contre certaines attaques par liens symboliques ;
    
- la protection contre certains hard links abusifs.
    


### 9.8.2. ASLR : `randomize_va_space`

Nous lisons :

```bash
cat /proc/sys/kernel/randomize_va_space
```

Valeurs courantes :

```text
0 : désactivé
1 : randomisation partielle
2 : randomisation plus complète
```

En production, nous gardons généralement l’ASLR activé.

Désactiver l’ASLR facilite certains tests bas niveau, mais affaiblit la sécurité.


### 9.8.3. Restriction de `dmesg`

Nous lisons :

```bash
cat /proc/sys/kernel/dmesg_restrict
```

Si la valeur est `1`, l’accès non privilégié à `dmesg` est restreint.

Cela limite l’exposition d’informations noyau qui pourraient aider un attaquant.


### 9.8.4. Restriction des pointeurs noyau

Nous lisons :

```bash
cat /proc/sys/kernel/kptr_restrict
```

Ce paramètre limite l’exposition d’adresses noyau dans certaines interfaces.

C’est important car les adresses noyau peuvent faciliter des attaques exploitant des vulnérabilités bas niveau.


## 9.9. Secrets et bonnes pratiques applicatives

### 9.9.1. Ne pas mettre de secrets dans les arguments

Nous évitons :

```bash
app --password secret
app --token abcdef
curl -H "Authorization: Bearer secret" ...
```

Nous préférons :

- gestionnaire de secrets ;
    
- fichier de configuration avec permissions strictes ;
    
- agent local ;
    
- socket sécurisée ;
    
- injection contrôlée par l’orchestrateur ;
    
- variable d’environnement seulement si le risque est compris et accepté.
    


### 9.9.2. Variables d’environnement : pratique mais exposable

Les variables d’environnement sont très utilisées dans Docker, Kubernetes, systemd et les applications cloud-native.

Mais elles peuvent être visibles :

- dans `/proc/<PID>/environ` ;
    
- dans certains dumps ;
    
- dans des interfaces de debug ;
    
- dans des logs si l’application les affiche ;
    
- dans des erreurs de configuration.
    

Nous ne devons pas croire qu’une variable d’environnement est forcément protégée.


### 9.9.3. Fichiers de secrets

Un fichier de secret peut être préférable si :

- les permissions sont strictes ;
    
- le propriétaire est correct ;
    
- le fichier n’est pas logué ;
    
- le fichier n’est pas inclus dans une image Docker ;
    
- le fichier n’est pas commité dans Git ;
    
- le secret peut être monté temporairement par l’orchestrateur.
    

Exemple de permissions :

```bash
chmod 600 /etc/app/secret.conf
chown appuser:appuser /etc/app/secret.conf
```

Mais là encore, si le processus ouvre ce fichier, il peut apparaître dans :

```bash
/proc/<PID>/fd
```

Nous ne supprimons pas le risque, nous le réduisons et nous le contrôlons.


## 9.10. Serveurs multi-utilisateurs

### 9.10.1. Risques spécifiques

Sur un serveur utilisé par plusieurs personnes, `/proc` peut exposer des informations entre utilisateurs.

Exemples :

- un étudiant voit les processus d’un autre étudiant ;
    
- un utilisateur voit une commande contenant un token ;
    
- un utilisateur identifie un service vulnérable ;
    
- un utilisateur observe les chemins de travail d’un autre ;
    
- un utilisateur détecte les ports locaux ouverts.
    

Ces risques sont importants sur :

- serveurs universitaires ;
    
- machines de calcul partagées ;
    
- serveurs de formation ;
    
- bastions SSH ;
    
- plateformes mutualisées.
    


### 9.10.2. Mesures possibles

Nous pouvons renforcer :

```text
hidepid=2
Yama ptrace_scope
permissions strictes
pas de secrets en arguments
limites systemd
isolation par conteneur ou VM
journalisation contrôlée
groupes d’administration dédiés
```

Aucune mesure unique ne suffit.

Nous construisons une défense en profondeur.


## 9.11. Audit rapide de sécurité autour de `/proc`

### 9.11.1. Vérifier les options de montage

```bash
findmnt /proc
```

Nous cherchons notamment :

```text
hidepid
nosuid
nodev
noexec
```

Options courantes :

```text
rw,nosuid,nodev,noexec,relatime
```


### 9.11.2. Vérifier les paramètres sensibles

```bash
sysctl kernel.randomize_va_space
sysctl kernel.dmesg_restrict
sysctl kernel.kptr_restrict
sysctl fs.protected_symlinks
sysctl fs.protected_hardlinks
```

Nous cherchons des valeurs cohérentes avec un système durci.


### 9.11.3. Chercher des secrets en arguments

En environnement de test ou d’audit autorisé :

```bash
ps aux | grep -Ei 'password|passwd|token|secret|key'
```

Nous pouvons aussi inspecter `/proc/<PID>/cmdline`.

L’objectif est de détecter les mauvaises pratiques.


### 9.11.4. Chercher des environnements sensibles

Avec les droits nécessaires et dans un cadre autorisé :

```bash
sudo sh -c '
for env in /proc/[0-9]*/environ; do
    pid=$(echo "$env" | cut -d/ -f3)
    tr "\0" "\n" < "$env" 2>/dev/null |
    grep -Ei "password|passwd|token|secret|key" |
    sed "s/^/PID=$pid /"
done
'
```

Nous n’exécutons pas ce type de commande sans autorisation explicite, car elle peut exposer des secrets.


## 9.12. Cas pratique : durcir un serveur partagé

### 9.12.1. Situation

Nous administrons un serveur Linux utilisé par plusieurs développeurs.

Chaque utilisateur peut se connecter en SSH.

Nous voulons éviter qu’un utilisateur puisse observer trop facilement les processus et arguments des autres.


### 9.12.2. Diagnostic initial

Nous vérifions :

```bash
findmnt /proc
```

Puis, depuis un utilisateur non privilégié :

```bash
ps aux | head
ls /proc | grep -E '^[0-9]+$' | head
```

Nous observons la visibilité actuelle.


### 9.12.3. Mesure principale : `hidepid=2`

Nous testons temporairement :

```bash
sudo mount -o remount,hidepid=2 /proc
```

Puis nous vérifions avec un utilisateur non privilégié :

```bash
ps aux
ls /proc
```

Nous observons que les processus des autres utilisateurs sont masqués ou moins visibles.


### 9.12.4. Persistance

Nous configurons `/etc/fstab` avec prudence.

Exemple :

```fstab
proc /proc proc defaults,nosuid,nodev,noexec,hidepid=2 0 0
```

Puis nous testons au redémarrage ou via un remount contrôlé.

Nous vérifions :

```bash
findmnt /proc
```


### 9.12.5. Compléments

Nous vérifions aussi :

```bash
sysctl kernel.yama.ptrace_scope
sysctl kernel.dmesg_restrict
sysctl kernel.kptr_restrict
```

Nous sensibilisons les utilisateurs :

```text
pas de mots de passe en ligne de commande ;
pas de tokens dans les scripts visibles ;
pas de secrets dans les logs ;
permissions strictes sur les fichiers de configuration.
```


## 9.13. Cas pratique : analyser un conteneur

### 9.13.1. Situation

Nous sommes dans un conteneur et nous voulons comprendre ce que `/proc` représente.

Nous exécutons :

```bash
cat /proc/1/comm
ls /proc | grep -E '^[0-9]+$' | wc -l
cat /proc/net/dev
cat /proc/self/mounts | head
cat /proc/1/cgroup
```

Nous observons :

- le PID 1 du conteneur ;
    
- le nombre de processus visibles ;
    
- les interfaces réseau vues ;
    
- les montages visibles ;
    
- les cgroups.
    


### 9.13.2. Comparaison avec l’hôte

Depuis l’hôte, nous comparons :

```bash
ps aux
ip addr
findmnt
```

Nous constatons que les vues diffèrent.

Nous retenons :

```text
/proc n’est pas une vérité absolue indépendante du contexte.
C’est une vue dépendante des namespaces et des montages.
```


### 9.13.3. Vérifier les montages sensibles

Dans un conteneur, nous inspectons :

```bash
mount | grep proc
findmnt /proc
```

Nous cherchons à savoir si `/proc` est monté normalement, en lecture seule, ou avec des restrictions spécifiques.

Selon le runtime, certaines parties de `/proc` peuvent être masquées ou rendues non modifiables.


## 9.14. Bonnes pratiques de sécurité

### 9.14.1. Pour les développeurs

Nous appliquons les règles suivantes :

```text
ne pas passer de secrets en arguments ;
éviter d’imprimer l’environnement dans les logs ;
ne pas stocker de secrets dans des fichiers lisibles par tous ;
limiter les permissions des fichiers de configuration ;
prévoir la rotation des secrets ;
utiliser un gestionnaire de secrets quand c’est possible.
```


### 9.14.2. Pour les administrateurs système

Nous vérifions :

```text
options de montage de /proc ;
paramètres sysctl de sécurité ;
permissions des services ;
limites systemd ;
séparation des utilisateurs ;
journalisation ;
durcissement SSH ;
conteneurs non privilégiés ;
absence de montages dangereux.
```


### 9.14.3. Pour les environnements conteneurisés

Nous évitons :

```text
--privileged sans nécessité ;
montage de /proc de l’hôte ;
montage de /sys de l’hôte ;
accès large à /dev ;
capabilities inutiles ;
secrets en variables d’environnement si l’exposition est inacceptable.
```

Nous préférons :

```text
capabilities minimales ;
root filesystem en lecture seule si possible ;
utilisateur non-root ;
seccomp ;
AppArmor ou SELinux ;
secrets gérés par l’orchestrateur ;
politiques réseau ;
limites cgroups.
```


## 9.15. Pièges et limites

### 9.15.1. Masquer `/proc` ne suffit pas

`hidepid` réduit la visibilité, mais ne remplace pas :

- une bonne gestion des permissions ;
    
- une séparation correcte des utilisateurs ;
    
- une politique de secrets ;
    
- un durcissement système ;
    
- une supervision ;
    
- des mises à jour de sécurité.
    

C’est une couche parmi d’autres.


### 9.15.2. Trop restreindre peut casser des outils

Certaines restrictions peuvent perturber :

- outils de monitoring ;
    
- agents de supervision ;
    
- outils de debug ;
    
- scripts d’administration ;
    
- services qui inspectent les processus.
    

Avant de durcir, nous devons tester.


### 9.15.3. Les conteneurs ne sont pas des machines virtuelles

Un conteneur partage le noyau avec l’hôte.

Même si `/proc` est isolé par namespaces, une mauvaise configuration peut affaiblir l’isolation.

Nous ne devons pas considérer un conteneur privilégié comme une barrière de sécurité forte.


### 9.15.4. Les secrets finissent souvent en mémoire

Même si nous évitons les arguments et limitons les variables d’environnement, un secret utilisé par une application finit souvent en mémoire.

Nous devons donc contrôler :

- qui peut déboguer le processus ;
    
- qui peut lire ses dumps mémoire ;
    
- qui peut accéder à `/proc/<PID>/mem` ;
    
- qui peut lancer des outils de diagnostic ;
    
- comment les core dumps sont gérés.
    


## 9.16. Exercices

### Exercice 1 — Observer les permissions

Nous exécutons :

```bash
ls -ld /proc/$$
ls -l /proc/$$/cmdline
ls -l /proc/$$/environ
ls -l /proc/$$/fd
ls -l /proc/$$/maps
```

Nous répondons :

1. Quels fichiers sont lisibles par l’utilisateur courant ?
    
2. Quels fichiers sont sensibles ?
    
3. Pourquoi `environ` est-il plus sensible que `status` ?
    
4. Pourquoi `fd` peut-il révéler des informations privées ?
    


### Exercice 2 — Observer les arguments d’un processus

Nous lançons volontairement un processus de test :

```bash
sleep 300 --test-secret-example 2>/dev/null &
pid=$!
```

Selon l’implémentation de `sleep`, cette commande peut échouer si l’option est invalide. Nous pouvons plutôt utiliser :

```bash
python3 -c 'import time; time.sleep(300)' --token FAUX_SECRET_DE_TEST &
pid=$!
```

Puis :

```bash
tr '\0' ' ' < /proc/$pid/cmdline
echo
```

Nous répondons :

1. Les arguments sont-ils visibles ?
    
2. Pourquoi ne faut-il jamais mettre un vrai secret ici ?
    
3. Quelle commande haut niveau peut aussi afficher ces arguments ?
    

Nous terminons :

```bash
kill $pid
```


### Exercice 3 — Variables d’environnement

Nous lançons :

```bash
TEST_SECRET=FAUX_SECRET_DE_TEST sleep 300 &
pid=$!
```

Puis :

```bash
tr '\0' '\n' < /proc/$pid/environ | grep TEST_SECRET
```

Nous répondons :

1. La variable est-elle visible ?
    
2. Qui peut la lire ?
    
3. Pourquoi les variables d’environnement ne sont-elles pas un coffre-fort ?
    
4. Quelles alternatives pouvons-nous envisager ?
    

Nous terminons :

```bash
kill $pid
```


### Exercice 4 — Vérifier `hidepid`

Nous exécutons :

```bash
findmnt /proc
mount | grep ' /proc '
```

Nous répondons :

1. L’option `hidepid` est-elle active ?
    
2. Si oui, quelle valeur ?
    
3. Que changerait `hidepid=2` ?
    
4. Quels outils pourraient être impactés ?
    


### Exercice 5 — Paramètres de sécurité `sysctl`

Nous exécutons :

```bash
sysctl kernel.randomize_va_space
sysctl kernel.dmesg_restrict
sysctl kernel.kptr_restrict
sysctl fs.protected_symlinks
sysctl fs.protected_hardlinks
```

Nous répondons :

1. L’ASLR est-il activé ?
    
2. L’accès à `dmesg` est-il restreint ?
    
3. Les pointeurs noyau sont-ils restreints ?
    
4. Les protections sur symlinks et hardlinks sont-elles activées ?
    
5. Pourquoi ces paramètres réduisent-ils l’exposition d’informations ?
    


### Exercice 6 — `/proc` dans un conteneur

Dans un conteneur de test, nous exécutons :

```bash
cat /proc/1/comm
ls /proc | grep -E '^[0-9]+$'
cat /proc/net/dev
cat /proc/self/mounts | head
cat /proc/1/cgroup
```

Nous répondons :

1. Quel est le PID 1 dans le conteneur ?
    
2. Combien de processus voyons-nous ?
    
3. Quelles interfaces réseau voyons-nous ?
    
4. La vue correspond-elle à l’hôte ?
    
5. Quels namespaces semblent influencer ces résultats ?
    


## 9.17. Ce que nous devons retenir

Nous retenons les points suivants :

1. `/proc` expose des informations puissantes et parfois sensibles.
    
2. `cmdline` peut révéler des secrets passés en arguments.
    
3. `environ` peut révéler des secrets passés en variables d’environnement.
    
4. `fd` peut révéler les fichiers, sockets et ressources ouvertes.
    
5. `maps` peut révéler la structure mémoire et les bibliothèques chargées.
    
6. `/proc/<PID>/mem` est extrêmement sensible.
    
7. Les permissions dépendent de l’UID, des droits, des capacités et des mécanismes de sécurité.
    
8. `ptrace_scope` peut restreindre le debugging et certains accès inter-processus.
    
9. `hidepid` permet de limiter la visibilité des processus entre utilisateurs.
    
10. `/proc/net` dépend du namespace réseau courant.
    
11. `/proc/1` dépend du namespace PID courant.
    
12. Les conteneurs ne sont pas des VM : ils partagent le noyau avec l’hôte.
    
13. Les conteneurs privilégiés et les montages de `/proc` de l’hôte sont dangereux.
    
14. Les paramètres de `/proc/sys` peuvent renforcer ou affaiblir la sécurité.
    
15. Le durcissement doit être testé, documenté et adapté au contexte.
    
16. La sécurité de `/proc` repose sur une défense en profondeur.
    


## Conclusion du chapitre 9

Nous savons maintenant analyser `/proc` comme une surface d’observation sensible.

`/proc` est indispensable pour comprendre et administrer Linux, mais il peut aussi révéler trop d’informations si le système est mal configuré ou si les applications manipulent mal leurs secrets.

Nous avons étudié les risques liés aux lignes de commande, aux variables d’environnement, aux fichiers ouverts, aux mappings mémoire, aux paramètres noyau et aux conteneurs. Nous avons aussi vu plusieurs mécanismes de protection : permissions Unix, restrictions `ptrace`, option `hidepid`, paramètres `sysctl`, namespaces et bonnes pratiques applicatives.

Nous retenons surtout une idée : `/proc` n’est pas dangereux en soi, mais il rend visibles des informations qui doivent être maîtrisées. Un bon administrateur sait l’utiliser pour diagnostiquer, mais aussi le restreindre lorsque le contexte l’exige.

Dans le chapitre suivant, nous étudions le lien entre `/proc` et les outils système comme `ps`, `top`, `htop`, `free`, `uptime`, `lsof`, `ss`, `vmstat` et `strace`.

---
> [!info] Livre « proc » — chapitre 9/11
> [[proc — Sommaire|Sommaire]] · [[proc — 08 — Paramètres noyau avec proc sys|← 08 — Paramètres noyau avec proc sys]] · [[proc — 10 — proc et les outils système|10 — proc et les outils système →]]
