---
schema_version: 1
uid: 01M1BQ624GMJ2Q9G8F3EYTAR52
titre: "Les namespaces Linux — 09 — Namespace user"
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
resume: "Chapitre 9 sur 16 du livre « Les namespaces Linux » : Namespace user. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « Les namespaces Linux » — chapitre 9/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 08 — Namespace IPC|← 08 — Namespace IPC]] · [[Les namespaces Linux — 10 — Namespace cgroup|10 — Namespace cgroup →]]

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

## 9.1. Pourquoi isoler les utilisateurs ?

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

## 9.2. Comprendre UID, GID et root

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

## 9.3. Observer le namespace user courant

## 9.3.1. Avec `/proc/<PID>/ns/user`

Nous observons le namespace user de notre shell courant :

```bash
readlink /proc/$$/ns/user
```

Exemple :

```text
user:
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

## 9.4. Format de `uid_map` et `gid_map`

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

## 9.5. Créer un user namespace simple

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

## 9.6. Root dans le namespace, utilisateur normal sur l’hôte

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

## 9.7. User namespace et capabilities

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

## 9.8. Mappings GID et difficulté avec les groupes

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

## 9.9. `/etc/subuid` et `/etc/subgid`

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

## 9.10. `newuidmap` et `newgidmap`

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

## 9.11. User namespace et conteneurs rootless

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

## 9.12. User namespace et fichiers montés

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

## 9.13. User namespace et sécurité

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

## 9.14. Interaction avec les autres namespaces

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

## 9.15. Expérience complète guidée

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

## 9.16. Cas pratique : comprendre un problème de permissions rootless

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

## 9.17. Pièges classiques

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

## 9.18. Exercices

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

## 9.19. Ce que nous devons retenir

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

## Conclusion du chapitre 9

Nous avons étudié le namespace user, l’un des mécanismes les plus puissants et les plus subtils de Linux.

Nous savons maintenant qu’il permet d’isoler les identifiants utilisateurs et groupes, de créer un environnement où un processus se voit comme `root`, tout en restant mappé vers un utilisateur non privilégié sur l’hôte.

Nous avons compris le rôle central des fichiers `uid_map` et `gid_map`, des plages `/etc/subuid` et `/etc/subgid`, ainsi que des outils `newuidmap` et `newgidmap`.

Nous retenons surtout que le namespace user est une brique essentielle des conteneurs rootless. Il améliore la sécurité en évitant de donner les droits root réels à un conteneur, mais il introduit aussi une complexité importante autour des permissions, des fichiers montés, des capabilities et des mappings.

Dans le chapitre suivant, nous étudions le namespace cgroup, qui isole la vue des cgroups, et nous clarifions la différence entre le namespace cgroup et les cgroups eux-mêmes.

---
> [!info] Livre « Les namespaces Linux » — chapitre 9/16
> [[Les namespaces Linux — Sommaire|Sommaire]] · [[Les namespaces Linux — 08 — Namespace IPC|← 08 — Namespace IPC]] · [[Les namespaces Linux — 10 — Namespace cgroup|10 — Namespace cgroup →]]
