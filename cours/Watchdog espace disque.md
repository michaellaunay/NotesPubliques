---
schema_version: 1
uid: "01M02EX5CBWW4RPBXNRV2M8Q6D"
titre: "Watchdog espace disque"
aliases:
  - "Surveillance espace disque Linux"
  - "Disk watchdog"
  - "Supervision capacité disque"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - supervision
  - bash
  - systemd
  - stockage
resume: "Cours pratique sur la supervision de la capacité des systèmes de fichiers Linux : blocs, espace disponible, inodes, script Bash robuste, journalisation, systemd.timer, diagnostic des causes de saturation, alertes et intégration Prometheus/node_exporter."
niveau: intermediaire
prerequis:
  - "[[GNULinux]]"
  - "[[Initialisation système et des services]]"
  - "[[Sécurité avancée sous Linux]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-05-10
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre GNU/Linux
> Ce cours est repris dans le livre [[GNU Linux — Sommaire]] — chapitre [[GNU Linux — 12 — Disques, systèmes de fichiers, RAID et LVM|12]].

> [!info] Refonte
> Cours développé le 31 août 2026 (d'environ 306 à 6 976 mots  remplacement complet du texte précédent)  vérifié le même jour : schéma  titres  liens et affirmations datées contrôlés. La version précédente reste dans l'historique git du dépôt. Les éléments spécifiques de l'ancienne version sont conservés en annexe  en fin de note.

# Surveiller l'espace disque sous GNU/Linux

> [!abstract] Objectif
> Savoir détecter **avant la panne** qu'un système de fichiers manque de capacité, distinguer une saturation en **blocs** d'une saturation en **inodes**, automatiser les contrôles proprement avec `systemd`, diagnostiquer la cause d'une croissance anormale et choisir quand passer d'un simple watchdog local à une vraie plate-forme de supervision.

Voir aussi : [[GNULinux]], [[Initialisation système et des services]], [[Sécurité avancée sous Linux]], [[Postfix]], [[Docker]], [[proc]].

> [!important] Idée centrale
> « Le disque est plein » est une formulation trop vague. Un service peut cesser d'écrire parce que :
>
> - le système de fichiers n'a presque plus de blocs disponibles ;
> - il n'a plus d'inodes disponibles ;
> - un quota est atteint ;
> - le pool sous-jacent LVM/ZFS/Btrfs/thin-provisioning est saturé ;
> - un fichier supprimé est encore ouvert par un processus ;
> - le stockage physique est en erreur alors que la capacité logique semble correcte.
>
> Un watchdog utile doit donc surveiller **plus qu'un pourcentage**.

# 1. Pourquoi surveiller la capacité

Un système de fichiers saturé peut provoquer des effets très différents selon son rôle :

- échec des écritures de logs ;
- impossibilité de créer des fichiers temporaires ;
- échec d'une base de données ;
- rejet de courriels dans une file d'attente ;
- échec d'une mise à jour de paquets ;
- corruption applicative si le logiciel gère mal `ENOSPC` ;
- impossibilité d'ouvrir une session ;
- blocage d'un conteneur ;
- indisponibilité d'un service ;
- impossibilité de faire tourner les journaux ou les sauvegardes.

Sur un serveur, l'objectif n'est donc pas d'être averti **quand le volume est déjà plein**, mais suffisamment tôt pour pouvoir agir.

## 1.1 Capacité et santé sont deux sujets différents

La **capacité** répond à la question :

> Combien d'espace logique reste-t-il dans le système de fichiers ?

La **santé** répond plutôt à :

> Le SSD, le NVMe, le disque, le RAID ou le pool de stockage fonctionnent-ils correctement ?

`df` surveille essentiellement le premier sujet. Il ne remplace pas :

- SMART ;
- les informations NVMe ;
- la supervision RAID ;
- la supervision LVM thin ;
- les outils spécifiques à ZFS ou Btrfs ;
- les métriques d'une baie SAN ou d'un stockage cloud.

# 2. Les notions indispensables

## 2.1 Système de fichiers et point de montage

Un disque physique n'est pas nécessairement équivalent à un système de fichiers.

Une machine peut contenir :

```text
SSD /dev/nvme0n1
└── partition /dev/nvme0n1p3
    └── volume physique LVM
        └── groupe de volumes vg0
            ├── volume logique root
            │   └── ext4 monté sur /
            └── volume logique var
                └── ext4 monté sur /var
```

Pour voir les périphériques :

```bash
lsblk
```

Pour voir les systèmes de fichiers et leurs points de montage :

```bash
findmnt
```

Pour une vue enrichie :

```bash
lsblk -f
```

> [!tip]
> Pour une alerte de **capacité**, on raisonne en général par **système de fichiers monté**, et non seulement par disque physique.

## 2.2 Blocs

Les données d'un fichier occupent des blocs dans un système de fichiers.

Quand l'espace disponible en blocs devient insuffisant, les nouvelles écritures finissent par échouer.

La commande de base est :

```bash
df -h
```

Exemple :

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/vg0/root    40G   28G   10G  74% /
/dev/vg0/var     20G   16G  3.0G  85% /var
```

`-h` affiche des unités lisibles par un humain.

## 2.3 Espace disponible et blocs réservés

Sur certains systèmes de fichiers, notamment ext4, une partie de l'espace peut être réservée à l'administration ou au bon fonctionnement du système.

Il est donc normal que :

```text
Used + Avail
```

ne corresponde pas toujours exactement à :

```text
Size
```

Pour la supervision d'un service non privilégié, la colonne **Avail** est particulièrement importante : elle représente l'espace que les processus ordinaires peuvent réellement utiliser.

Afficher les valeurs en octets :

```bash
df -B1 /
```

Afficher uniquement les champs utiles :

```bash
df -B1 --output=size,used,avail,pcent,target /
```

## 2.4 Inodes

Un système de fichiers peut encore disposer de gigaoctets libres tout en étant incapable de créer un nouveau fichier s'il n'a plus d'inodes.

Un inode contient les métadonnées nécessaires à la représentation d'un objet du système de fichiers.

Afficher leur consommation :

```bash
df -i
```

Exemple :

```text
Filesystem      Inodes   IUsed   IFree IUse% Mounted on
/dev/vg0/root   2621440  180000 2441440    7% /
/dev/vg0/var    1310720 1240000   70720   95% /var
```

Ici `/var` est presque en panne **même si ses blocs ne sont pas encore pleins**.

Afficher directement les champs d'inodes avec GNU `df` :

```bash
df --output=itotal,iused,iavail,ipcent,target /var
```

> [!warning]
> Un watchdog qui ne surveille que `Use%` peut manquer une saturation en inodes.

# 3. `df`, `du` et `stat` ne répondent pas à la même question

## 3.1 `df`

`df` demande au système de fichiers son état global.

```bash
df -h /
```

Question traitée :

> Quelle est la capacité du système de fichiers contenant `/` ?

## 3.2 `du`

`du` additionne l'espace attribué à des fichiers visibles dans une arborescence.

```bash
du -sh /var/log
```

Question traitée :

> Combien d'espace occupent les fichiers accessibles sous `/var/log` ?

Les deux résultats peuvent différer.

## 3.3 `stat`

`stat` peut afficher des informations sur un fichier ou un système de fichiers :

```bash
stat /
```

ou :

```bash
stat -f /
```

Il est utile dans des scripts spécialisés, mais `df` reste généralement plus lisible pour un contrôle de capacité.

# 4. Pourquoi un seuil unique de 70 % est insuffisant

L'ancienne version de ce cours envoyait un courriel à partir de 70 % d'utilisation.

Cette règle est trop simpliste.

Considérons deux volumes :

| Volume | Capacité | Utilisation | Libre approximatif |
|---|---:|---:|---:|
| A | 10 Gio | 80 % | 2 Gio |
| B | 10 Tio | 80 % | 2 Tio |

Le même pourcentage ne représente pas la même urgence.

Inversement, un petit volume critique peut nécessiter une alerte bien avant 90 %.

## 4.1 Utiliser plusieurs critères

Une politique raisonnable combine par exemple :

- pourcentage de blocs utilisés ;
- pourcentage d'inodes utilisés ;
- espace absolu encore disponible ;
- rôle du système de fichiers ;
- vitesse de croissance ;
- durée estimée avant saturation.

Exemple pédagogique :

```text
warning : blocs >= 80 %
critical: blocs >= 90 %
warning : inodes >= 80 %
critical: inodes >= 90 %
critical: moins de 1 Gio disponible
```

Ces valeurs **ne sont pas universelles**.

## 4.2 Adapter les seuils au rôle

Pour `/var/lib/postgresql`, une croissance rapide peut imposer davantage de marge.

Pour un volume d'archives de plusieurs dizaines de téraoctets, un seuil uniquement exprimé en pourcentage peut générer des alertes trop tôt.

Pour `/boot`, un seuil absolu de 1 Gio serait absurde si le volume total ne fait que quelques gigaoctets.

> [!important]
> Les seuils sont une **politique d'exploitation**, pas une vérité mathématique.

# 5. Contrôles manuels rapides

## 5.1 Système de fichiers racine

```bash
df -h /
```

## 5.2 Tous les systèmes de fichiers

```bash
df -h
```

## 5.3 Systèmes de fichiers locaux seulement

```bash
df -hl
```

Cela évite par exemple de bloquer ou ralentir une commande à cause de certains montages réseau.

## 5.4 Inodes

```bash
df -ih
```

## 5.5 Type de système de fichiers

```bash
df -hT
```

ou :

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

## 5.6 Vue synthétique d'un point précis

```bash
df -h /var

df -i /var
```

# 6. Écrire un watchdog Bash robuste

Nous allons construire un script qui :

1. accepte un ou plusieurs chemins à surveiller ;
2. demande à `df` le système de fichiers correspondant ;
3. contrôle les blocs ;
4. contrôle les inodes ;
5. contrôle l'espace libre absolu ;
6. écrit dans le journal système ;
7. renvoie un code d'état exploitable par `systemd` ou un autre superviseur.

## 6.1 Codes de retour

Nous utiliserons :

```text
0 = OK
1 = WARNING
2 = CRITICAL ou erreur de contrôle
```

Cette convention est simple et proche de nombreux systèmes de supervision.

## 6.2 Le script

Créer :

```text
/usr/local/sbin/disk-watchdog
```

avec :

```bash
#!/usr/bin/env bash
set -uo pipefail

WARN_PERCENT="${WARN_PERCENT:-80}"
CRIT_PERCENT="${CRIT_PERCENT:-90}"
WARN_INODE_PERCENT="${WARN_INODE_PERCENT:-80}"
CRIT_INODE_PERCENT="${CRIT_INODE_PERCENT:-90}"
MIN_FREE_MIB="${MIN_FREE_MIB:-1024}"
TAG="${TAG:-disk-watchdog}"

is_uint() {
    [[ "$1" =~ ^[0-9]+$ ]]
}

validate_thresholds() {
    local value
    for value in \
        "$WARN_PERCENT" \
        "$CRIT_PERCENT" \
        "$WARN_INODE_PERCENT" \
        "$CRIT_INODE_PERCENT" \
        "$MIN_FREE_MIB"
    do
        if ! is_uint "$value"; then
            printf 'Configuration invalide : %q n est pas un entier positif\n' "$value" >&2
            exit 2
        fi
    done

    if (( WARN_PERCENT > CRIT_PERCENT )); then
        echo "WARN_PERCENT doit être <= CRIT_PERCENT" >&2
        exit 2
    fi

    if (( WARN_INODE_PERCENT > CRIT_INODE_PERCENT )); then
        echo "WARN_INODE_PERCENT doit être <= CRIT_INODE_PERCENT" >&2
        exit 2
    fi
}

emit() {
    local priority="$1"
    shift
    printf '%s\n' "$*"
    logger --priority "user.${priority}" --tag "$TAG" -- "$*"
}

validate_thresholds

if (( $# == 0 )); then
    set -- /
fi

host="$(hostname -f 2>/dev/null || hostname)"
global_status=0

for path in "$@"; do
    if [[ ! -e "$path" ]]; then
        emit err "host=$host state=CRITICAL path=$path reason=path-not-found"
        global_status=2
        continue
    fi

    if ! block_line="$(
        df -B1 --output=size,used,avail,pcent,target -- "$path" | tail -n 1
    )"; then
        emit err "host=$host state=CRITICAL path=$path reason=df-failed"
        global_status=2
        continue
    fi

    read -r size used avail pcent target <<< "$block_line"
    block_percent="${pcent%%%}"

    if ! inode_line="$(
        df --output=itotal,iused,iavail,ipcent,target -- "$path" | tail -n 1
    )"; then
        emit err "host=$host state=CRITICAL path=$path reason=df-inode-failed"
        global_status=2
        continue
    fi

    read -r itotal iused iavail ipcent inode_target <<< "$inode_line"

    if [[ "$ipcent" == "-" ]]; then
        inode_percent=-1
    else
        inode_percent="${ipcent%%%}"
    fi

    if ! is_uint "$size" || \
       ! is_uint "$used" || \
       ! is_uint "$avail" || \
       ! is_uint "$block_percent"; then
        emit err "host=$host state=CRITICAL path=$path reason=unexpected-df-output"
        global_status=2
        continue
    fi

    free_mib=$(( avail / 1024 / 1024 ))
    local_status=0
    state="OK"
    priority="info"

    if (( block_percent >= CRIT_PERCENT )); then
        local_status=2
    fi

    if (( inode_percent >= 0 && inode_percent >= CRIT_INODE_PERCENT )); then
        local_status=2
    fi

    if (( MIN_FREE_MIB > 0 && free_mib < MIN_FREE_MIB )); then
        local_status=2
    fi

    if (( local_status < 2 && block_percent >= WARN_PERCENT )); then
        local_status=1
    fi

    if (( local_status < 2 && inode_percent >= 0 && inode_percent >= WARN_INODE_PERCENT )); then
        local_status=1
    fi

    case "$local_status" in
        0)
            state="OK"
            priority="info"
            ;;
        1)
            state="WARNING"
            priority="warning"
            ;;
        2)
            state="CRITICAL"
            priority="crit"
            ;;
    esac

    emit "$priority" \
        "host=$host state=$state target=$target blocks=${block_percent}% free_mib=$free_mib inodes=${inode_percent}%"

    if (( local_status > global_status )); then
        global_status="$local_status"
    fi
done

exit "$global_status"
```

Le rendre exécutable :

```bash
sudo chmod 0755 /usr/local/sbin/disk-watchdog
```

## 6.3 Pourquoi `set -e` n'est pas utilisé ici

Le script utilise :

```bash
set -uo pipefail
```

mais pas `set -e`.

Le watchdog veut être capable :

- de traiter plusieurs chemins ;
- d'enregistrer une erreur sur l'un d'eux ;
- puis de continuer à contrôler les suivants.

L'usage aveugle de `set -e` peut rendre ce type de script moins prévisible.

## 6.4 Pourquoi ne pas parser `df -h`

`df -h` est conçu pour la lecture humaine.

Le script demande explicitement :

```bash
df -B1 --output=size,used,avail,pcent,target
```

Les unités sont alors maîtrisées et les colonnes demandées explicitement.

> [!note]
> `--output` est une extension de GNU Coreutils. Ce script vise donc un système GNU/Linux classique. Pour un script strictement POSIX ou destiné aussi à BSD/macOS, il faut adapter la stratégie de parsing.

# 7. Configuration sans modifier le script

Les seuils peuvent être fournis dans l'environnement.

Créer par exemple :

```text
/etc/default/disk-watchdog
```

avec :

```bash
WARN_PERCENT=80
CRIT_PERCENT=90
WARN_INODE_PERCENT=80
CRIT_INODE_PERCENT=90
MIN_FREE_MIB=1024
TAG=disk-watchdog
```

Protéger le fichier si l'on y place ultérieurement des informations sensibles :

```bash
sudo chown root:root /etc/default/disk-watchdog
sudo chmod 0644 /etc/default/disk-watchdog
```

Ici le fichier ne contient pas de secret.

## 7.1 Tester manuellement

```bash
sudo env \
    WARN_PERCENT=80 \
    CRIT_PERCENT=90 \
    WARN_INODE_PERCENT=80 \
    CRIT_INODE_PERCENT=90 \
    MIN_FREE_MIB=1024 \
    /usr/local/sbin/disk-watchdog / /var
```

Afficher le code de retour :

```bash
echo $?
```

# 8. Journaliser plutôt que coder l'envoi d'un mail dans le cœur du script

L'ancienne version faisait directement :

```bash
echo "$MESSAGE" | mail -s "$SUBJECT" "$EMAIL"
```

Cela mélangeait plusieurs responsabilités :

```text
mesurer
+ décider de l'état
+ formater une alerte
+ envoyer un courriel
```

Il est préférable de séparer :

```text
watchdog
   │
   ├── mesure
   ├── décision
   └── journalisation + code de retour
                  │
                  ▼
       mécanisme de notification
```

Le script utilise :

```bash
logger
```

Les messages sont donc récupérables dans le journal :

```bash
journalctl -t disk-watchdog
```

Derniers messages :

```bash
journalctl -t disk-watchdog -n 50 --no-pager
```

Suivi en temps réel :

```bash
journalctl -f -t disk-watchdog
```

# 9. Exécuter le watchdog avec systemd

Pour une machine moderne utilisant systemd, un `.service` associé à un `.timer` est souvent plus intéressant qu'une tâche cron :

- journalisation intégrée ;
- statut visible ;
- dépendances explicites ;
- contrôle des privilèges ;
- durcissement de l'unité ;
- historique via `journalctl` ;
- inspection avec `systemctl list-timers`.

## 9.1 Service

Créer :

```text
/etc/systemd/system/disk-watchdog.service
```

```ini
[Unit]
Description=Surveillance de la capacité des systèmes de fichiers
Documentation=man:df(1)

[Service]
Type=oneshot
EnvironmentFile=-/etc/default/disk-watchdog
ExecStart=/usr/local/sbin/disk-watchdog / /var

# WARNING=1 est considéré comme un résultat acceptable.
# CRITICAL=2 continue à faire échouer l'unité.
SuccessExitStatus=1

NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=read-only
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
RestrictSUIDSGID=yes
LockPersonality=yes
MemoryDenyWriteExecute=yes
RestrictAddressFamilies=AF_UNIX
```

> [!note]
> Adapter la ligne `ExecStart` aux points de montage qui ont réellement du sens sur la machine.

## 9.2 Timer monotone

Créer :

```text
/etc/systemd/system/disk-watchdog.timer
```

```ini
[Unit]
Description=Contrôle périodique de l'espace disque

[Timer]
OnBootSec=5min
OnUnitActiveSec=15min
AccuracySec=1min
RandomizedDelaySec=30s
Unit=disk-watchdog.service

[Install]
WantedBy=timers.target
```

Ce timer :

- lance un premier contrôle quelques minutes après le démarrage ;
- relance ensuite le service environ toutes les quinze minutes ;
- ajoute un petit décalage afin d'éviter que de nombreuses machines ne lancent exactement la même tâche simultanément.

## 9.3 Vérifier les unités

Avant activation :

```bash
sudo systemd-analyze verify \
    /etc/systemd/system/disk-watchdog.service \
    /etc/systemd/system/disk-watchdog.timer
```

Recharger systemd :

```bash
sudo systemctl daemon-reload
```

Activer le timer :

```bash
sudo systemctl enable --now disk-watchdog.timer
```

## 9.4 Contrôler le timer

```bash
systemctl status disk-watchdog.timer
```

```bash
systemctl list-timers disk-watchdog.timer
```

## 9.5 Déclencher manuellement le service

```bash
sudo systemctl start disk-watchdog.service
```

Puis :

```bash
systemctl status disk-watchdog.service
```

et :

```bash
journalctl -u disk-watchdog.service --no-pager
```

# 10. Timer monotone ou `OnCalendar=` ?

Le timer précédent utilise :

```ini
OnBootSec=5min
OnUnitActiveSec=15min
```

Ces délais reposent sur une horloge monotone et conviennent bien à un contrôle périodique simple.

Une alternative calendaire est :

```ini
[Timer]
OnCalendar=*:0/15
Persistent=yes
RandomizedDelaySec=30s
```

Vérifier une expression :

```bash
systemd-analyze calendar '*:0/15'
```

`Persistent=yes` est pertinent avec `OnCalendar=` : si une exécution a été manquée parce que le timer était inactif, systemd peut déclencher le service au retour.

> [!important]
> `Persistent=` n'a d'effet que pour les timers basés sur `OnCalendar=`. Il ne transforme pas un timer monotone en timer persistant.

Pour un watchdog de capacité, rattraper toutes les exécutions manquées n'est généralement pas nécessaire : **un contrôle immédiatement après redémarrage suffit**.

# 11. Et `cron` ?

Cron reste parfaitement utilisable.

Par exemple :

```cron
*/15 * * * * /usr/local/sbin/disk-watchdog / /var
```

Mais sur une machine systemd, le timer apporte une meilleure intégration au reste de l'administration système.

Comparaison simplifiée :

| Besoin | cron | systemd timer |
|---|---:|---:|
| planification périodique | oui | oui |
| journal systemd natif | indirect | oui |
| dépendances d'unités | non | oui |
| durcissement du processus | limité | oui |
| inspection des prochaines exécutions | limitée | `systemctl list-timers` |
| déclenchement relatif au boot | possible mais moins naturel | natif |
| rattrapage calendaire | dépend de l'outil | `Persistent=` |

# 12. Ajouter une notification

Le cœur du watchdog ne devrait pas dépendre d'un seul canal d'alerte.

On peut raccorder :

- un courriel ;
- un serveur de supervision ;
- Prometheus/Alertmanager ;
- un outil d'astreinte ;
- un webhook interne ;
- une plate-forme de logs centralisée.

## 12.1 Notification par courriel locale

Si un MTA ou un relais SMTP est correctement configuré, on peut créer un mécanisme séparé.

Par exemple, ajouter dans `[Unit]` de `disk-watchdog.service` :

```ini
OnFailure=disk-watchdog-alert@%n.service
```

Puis créer :

```text
/etc/systemd/system/disk-watchdog-alert@.service
```

```ini
[Unit]
Description=Alerte après échec de %i

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/disk-watchdog-alert %i
```

Le script d'alerte peut récupérer les derniers journaux de l'unité et les transmettre au canal choisi.

Exemple minimal avec `mail` :

```bash
#!/usr/bin/env bash
set -euo pipefail

unit="${1:?Nom de l'unité manquant}"
recipient="admin@example.net"
host="$(hostname -f 2>/dev/null || hostname)"

journalctl -u "$unit" -n 30 --no-pager \
    | mail -s "[$host] $unit en état critique" "$recipient"
```

> [!warning]
> La présence de la commande `mail` ne signifie pas que le serveur sait réellement envoyer des messages vers Internet. Il faut un MTA local ou un relais correctement configuré. Voir [[Postfix]].

## 12.2 Ne pas mettre de jeton secret dans une ligne de commande

Un webhook avec jeton d'API ne devrait pas être écrit en clair dans :

```ini
ExecStart=/usr/bin/curl https://service.example/token-secret/...
```

Les arguments d'un processus peuvent être observables par d'autres mécanismes d'administration.

Préférer :

- un fichier de configuration protégé ;
- un credential systemd ;
- un gestionnaire de secrets ;
- une intégration native de l'outil de supervision.

# 13. Diagnostiquer un système de fichiers qui se remplit

Une alerte utile doit être suivie d'un diagnostic.

## 13.1 Commencer par identifier le système de fichiers

```bash
findmnt /var
```

ou :

```bash
df -hT /var
```

## 13.2 Identifier les gros répertoires sans changer de système de fichiers

Pour la racine :

```bash
sudo du -x -h --max-depth=1 / 2>/dev/null | sort -h
```

Pour `/var` :

```bash
sudo du -x -h --max-depth=1 /var 2>/dev/null | sort -h
```

`-x` évite de traverser d'autres systèmes de fichiers montés dans l'arborescence.

Puis descendre progressivement :

```bash
sudo du -x -h --max-depth=1 /var/lib 2>/dev/null | sort -h
```

## 13.3 Utiliser `ncdu`

Pour une exploration interactive :

```bash
sudo apt install ncdu
sudo ncdu -x /
```

Ne supprimer aucun fichier dont le rôle n'est pas compris.

# 14. Cas classique : fichier supprimé mais encore ouvert

Sous Unix, supprimer le nom d'un fichier ne libère pas nécessairement immédiatement ses blocs.

Si un processus possède encore un descripteur ouvert sur le fichier, les données restent occupées jusqu'à la fermeture du dernier descripteur.

Symptôme typique :

```text
df : disque presque plein
du : beaucoup moins d'espace visible
```

Chercher les fichiers supprimés encore ouverts :

```bash
sudo lsof +L1
```

ou :

```bash
sudo lsof | grep '(deleted)'
```

Exemple conceptuel :

```text
processus  PID  user  ...  /var/log/app.log (deleted)
```

La bonne correction dépend du service.

Souvent il faut :

- demander au programme de rouvrir ses logs ;
- faire une rotation correcte ;
- redémarrer proprement le service si nécessaire.

> [!danger]
> Ne pas tuer un processus de production uniquement pour récupérer de l'espace sans comprendre les conséquences.

# 15. Les journaux systemd

Afficher la taille occupée par le journal :

```bash
journalctl --disk-usage
```

Nettoyer en conservant une taille maximale :

```bash
sudo journalctl --vacuum-size=500M
```

Nettoyer en conservant une durée :

```bash
sudo journalctl --vacuum-time=14d
```

Ces commandes sont utiles pour une intervention ponctuelle.

La politique durable se configure dans :

```text
/etc/systemd/journald.conf
```

ou de préférence dans un drop-in :

```text
/etc/systemd/journald.conf.d/
```

Par exemple :

```ini
[Journal]
SystemMaxUse=1G
```

Après modification, vérifier la documentation de la version installée :

```bash
man journald.conf
```

# 16. Logs traditionnels et logrotate

Lister les fichiers importants :

```bash
sudo du -ah /var/log | sort -h | tail -n 30
```

Inspecter la configuration :

```bash
ls -l /etc/logrotate.conf /etc/logrotate.d/
```

Forcer uniquement un test en mode debug :

```bash
sudo logrotate --debug /etc/logrotate.conf
```

> [!warning]
> Ne pas confondre le mode `--debug`, qui simule les actions, et une rotation forcée réelle.

# 17. Docker et conteneurs

Les images, couches, volumes et logs de conteneurs peuvent consommer rapidement `/var`.

Vue synthétique :

```bash
docker system df
```

Vue détaillée :

```bash
docker system df -v
```

Lister les volumes :

```bash
docker volume ls
```

> [!danger]
> Éviter d'utiliser aveuglément :
>
> ```bash
> docker system prune -a --volumes
> ```
>
> Cette commande peut supprimer des données ou artefacts encore utiles. Identifier précisément ce qui peut être détruit avant tout nettoyage.

Les logs JSON d'un conteneur peuvent également grossir fortement si aucune rotation n'est configurée.

# 18. Inodes : diagnostiquer une explosion du nombre de fichiers

Si :

```bash
df -i /var
```

montre une saturation, chercher les zones contenant énormément de fichiers.

Une première approche :

```bash
sudo find /var -xdev -type f -printf '%h\n' \
    | sort \
    | uniq -c \
    | sort -n \
    | tail -n 30
```

Cette commande peut être coûteuse sur un très gros système de fichiers.

Autres causes fréquentes :

- cache mal borné ;
- files de messages ;
- millions de petits fichiers de sessions ;
- extraction d'une archive pathologique ;
- spool applicatif ;
- répertoires temporaires non nettoyés.

# 19. Trouver les gros fichiers

Exemple : fichiers de plus de 1 Gio sur le système de fichiers racine :

```bash
sudo find / -xdev -type f -size +1G -printf '%s %p\n' 2>/dev/null \
    | sort -n
```

Version lisible avec `numfmt` :

```bash
sudo find / -xdev -type f -size +1G -printf '%s %p\0' 2>/dev/null \
    | sort -z -n \
    | while IFS= read -r -d '' line; do
        size="${line%% *}"
        path="${line#* }"
        printf '%8s  %s\n' "$(numfmt --to=iec "$size")" "$path"
      done
```

> [!note]
> Sur des serveurs volumineux, un `find` complet peut être coûteux en I/O. Utiliser les outils de supervision, les index disponibles ou cibler d'abord les répertoires suspects.

# 20. Quand `du` et `df` ne concordent pas

Causes possibles :

1. fichiers supprimés mais encore ouverts ;
2. blocs réservés ;
3. fichiers cachés sous un point de montage ;
4. snapshots ;
5. fonctionnement propre au système de fichiers ;
6. sparse files et différences entre taille apparente et blocs réellement alloués ;
7. mécanismes de compression/déduplication ;
8. conteneurs ou namespaces de montage.

## 20.1 Taille apparente et occupation réelle

Afficher la taille logique :

```bash
du -h --apparent-size fichier
```

Comparer aux blocs réellement consommés :

```bash
du -h fichier
```

Un fichier sparse peut avoir une très grande taille apparente sans consommer autant de blocs.

# 21. Surveiller également la couche de stockage

Le watchdog de capacité ne suffit pas.

## 21.1 SMART

Installer :

```bash
sudo apt install smartmontools
```

Identifier les disques :

```bash
lsblk -d -o NAME,MODEL,SERIAL,SIZE,TRAN
```

Interroger un disque compatible :

```bash
sudo smartctl -a /dev/sda
```

Pour un NVMe, selon la pile utilisée :

```bash
sudo smartctl -a /dev/nvme0
```

`smartmontools` fournit aussi `smartd`, destiné à la surveillance régulière.

> [!important]
> SMART n'est pas une garantie qu'un disque ne tombera pas en panne. Il fournit des indicateurs supplémentaires ; il ne remplace ni la redondance ni les sauvegardes.

## 21.2 NVMe CLI

Sur un système utilisant les outils NVMe :

```bash
sudo apt install nvme-cli
```

Puis par exemple :

```bash
sudo nvme list
sudo nvme smart-log /dev/nvme0
```

## 21.3 RAID

Pour un RAID logiciel Linux :

```bash
cat /proc/mdstat
```

et selon le besoin :

```bash
sudo mdadm --detail /dev/md0
```

La capacité d'un système de fichiers peut sembler normale alors qu'un membre RAID est déjà défaillant.

# 22. LVM thin, ZFS et Btrfs

Les systèmes modernes peuvent avoir plusieurs couches de capacité.

## 22.1 LVM

```bash
pvs
vgs
lvs
```

Pour du thin provisioning, surveiller en particulier les pourcentages de données **et de métadonnées** du thin pool.

Une saturation du thin pool est différente d'un simple `df` élevé dans le système de fichiers invité.

## 22.2 Btrfs

`df` n'explique pas à lui seul la répartition interne de Btrfs.

Utiliser également :

```bash
sudo btrfs filesystem usage /
```

et selon les besoins :

```bash
sudo btrfs filesystem df /
```

## 22.3 ZFS

Utiliser les outils ZFS :

```bash
zpool list
zpool status
zfs list
```

Les snapshots peuvent conserver des blocs même après suppression des fichiers courants.

> [!important]
> Sur ces technologies, la bonne alerte combine la vue du système de fichiers avec celle du **pool sous-jacent**.

# 23. Limites d'un watchdog local

Le script Bash convient bien à :

- une machine isolée ;
- un TP ;
- une protection complémentaire ;
- un serveur simple ;
- un contrôle de secours.

Il devient moins adapté lorsque l'on veut :

- superviser des dizaines ou centaines de machines ;
- garder l'historique ;
- visualiser les tendances ;
- prédire la saturation ;
- dédupliquer les alertes ;
- définir des périodes de silence ;
- router les alertes vers plusieurs équipes ;
- calculer des SLO ;
- corréler stockage, CPU, mémoire et applications.

Dans ce cas, il faut un système de métriques et d'alerting.

# 24. Prometheus et node_exporter

`node_exporter` expose des métriques système Linux, notamment celles du collecteur `filesystem`.

Parmi les métriques importantes :

```text
node_filesystem_size_bytes
node_filesystem_free_bytes
node_filesystem_avail_bytes
node_filesystem_files
node_filesystem_files_free
node_filesystem_readonly
```

`avail_bytes` représente l'espace disponible pour les utilisateurs non privilégiés et est généralement plus pertinent que `free_bytes` pour une alerte applicative.

## 24.1 Pourcentage d'espace utilisé

Une expression courante est :

```promql
100 * (
  1 - node_filesystem_avail_bytes
      / node_filesystem_size_bytes
)
```

Il faut filtrer les systèmes de fichiers sans intérêt pour l'alerte.

Exemple conceptuel :

```promql
100 * (
  1 - node_filesystem_avail_bytes{
        fstype!~"tmpfs|overlay|squashfs"
      }
      / node_filesystem_size_bytes{
        fstype!~"tmpfs|overlay|squashfs"
      }
) > 85
```

Les filtres exacts doivent correspondre à l'environnement.

## 24.2 Pourcentage d'inodes consommés

```promql
100 * (
  1 - node_filesystem_files_free
      / node_filesystem_files
)
```

Éviter les séries où le nombre total d'inodes est nul ou non pertinent.

## 24.3 Systèmes de fichiers en lecture seule

```promql
node_filesystem_readonly == 1
```

Un système de fichiers devenu soudainement read-only peut être beaucoup plus critique qu'un simple problème de capacité.

# 25. Exemple de règles Prometheus

Exemple pédagogique :

```yaml
groups:
  - name: filesystem
    rules:
      - alert: FilesystemSpaceWarning
        expr: |
          100 * (
            1 - node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs"}
                / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs"}
          ) > 80
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "Espace disque élevé sur {{ $labels.instance }}"

      - alert: FilesystemSpaceCritical
        expr: |
          100 * (
            1 - node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs"}
                / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs"}
          ) > 90
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Espace disque critique sur {{ $labels.instance }}"
```

`for:` évite de déclencher immédiatement une alerte à cause d'un pic très court.

> [!warning]
> Les labels et les exclusions doivent être adaptés aux métriques réellement produites. Tester les expressions dans Prometheus avant de les utiliser en production.

# 26. Prédire une saturation

Une supervision centralisée peut aller plus loin qu'un seuil fixe.

Prometheus fournit par exemple `predict_linear()`.

Exemple conceptuel :

```promql
predict_linear(
  node_filesystem_avail_bytes[6h],
  24 * 3600
) < 0
```

L'idée est :

> Si la tendance observée sur les six dernières heures continue, l'espace disponible deviendra-t-il négatif dans les prochaines 24 heures ?

Cette prédiction est utile sur une croissance relativement régulière.

Elle peut être trompeuse en présence :

- d'importants fichiers temporaires ;
- de rotation de logs ;
- de sauvegardes cycliques ;
- de snapshots périodiques ;
- d'une charge très irrégulière.

Une alerte prédictive ne remplace donc pas les seuils de sécurité.

# 27. Éviter les pseudo-systèmes de fichiers

Une machine Linux expose de nombreux systèmes de fichiers qui ne correspondent pas à un volume persistant classique :

- `proc` ;
- `sysfs` ;
- `tmpfs` ;
- `devtmpfs` ;
- `cgroup2` ;
- `overlay` selon l'environnement ;
- montages de conteneurs.

Il faut décider lesquels ont un sens opérationnel.

Pour un contrôle manuel limité aux systèmes locaux :

```bash
df -l
```

Pour node_exporter, le collecteur filesystem propose des filtres de types de systèmes de fichiers et de points de montage.

> [!tip]
> Ne recopier pas aveuglément une regex d'exclusion trouvée sur Internet. Les points de montage pertinents dépendent de l'hôte, des conteneurs, de Kubernetes, de l'usage de NFS et de la distribution.

# 28. Conteneurs et namespaces de montage

Dans un conteneur, la vue de `df` peut être différente de celle de l'hôte.

Il faut demander :

> Où s'exécute le watchdog et quel namespace de montage observe-t-il ?

Un watchdog exécuté dans un conteneur ne voit pas automatiquement tous les systèmes de fichiers de l'hôte.

Pour node_exporter lancé en conteneur afin de surveiller l'hôte, la documentation officielle décrit une configuration où la racine de l'hôte est montée en lecture seule et fournie via `--path.rootfs`.

C'est un cas où comprendre les [[Les namespaces Linux|namespaces Linux]] est essentiel.

# 29. Sécurité du watchdog

Un contrôleur de capacité n'a normalement pas besoin de privilèges élevés pour lire `df`.

Le principe à suivre est :

> donner au service uniquement les droits dont il a besoin.

Le service proposé utilise plusieurs protections systemd :

```ini
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=read-only
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
RestrictSUIDSGID=yes
LockPersonality=yes
MemoryDenyWriteExecute=yes
```

Afficher le score de sécurité d'une unité :

```bash
systemd-analyze security disk-watchdog.service
```

Ce score est un **outil d'aide**, pas une preuve de sécurité.

## 29.1 Éviter root si possible

Le script de contrôle lui-même peut généralement tourner avec un utilisateur non privilégié.

Cependant, certaines arborescences ou méthodes de diagnostic (`du`, `lsof`, SMART, RAID) nécessitent des droits supplémentaires.

Ne pas donner ces privilèges au watchdog uniquement parce que les outils de dépannage en ont besoin.

# 30. Ne pas automatiser la suppression de données sans politique claire

Une très mauvaise réponse à une alerte serait :

```bash
rm -rf /var/log/*
```

ou :

```bash
docker system prune -af --volumes
```

Un watchdog ne devrait pas supprimer arbitrairement des données pour faire disparaître l'alerte.

Une action automatique n'est acceptable que si :

- les données sont explicitement jetables ;
- leur durée de conservation est définie ;
- la procédure est testée ;
- les erreurs sont gérées ;
- l'action est observable ;
- elle ne masque pas la cause racine.

Exemples de mécanismes mieux adaptés :

- `logrotate` ;
- `systemd-tmpfiles` ;
- politique de rétention de journald ;
- politique de rétention applicative ;
- expiration de cache documentée.

# 31. Tester sans remplir le vrai disque

Il ne faut pas tester un watchdog en remplissant volontairement `/` sur une machine importante.

Une méthode pédagogique consiste à créer un petit système de fichiers dans un fichier sparse.

## 31.1 Créer une image de test

```bash
truncate -s 256M /tmp/watchdog-test.img
```

Créer un ext4 :

```bash
sudo mkfs.ext4 -F /tmp/watchdog-test.img
```

Créer le point de montage :

```bash
sudo mkdir -p /mnt/watchdog-test
```

Monter :

```bash
sudo mount -o loop /tmp/watchdog-test.img /mnt/watchdog-test
```

Vérifier :

```bash
df -h /mnt/watchdog-test
```

## 31.2 Remplir progressivement

Créer par exemple un fichier de 150 Mio :

```bash
sudo dd if=/dev/zero \
    of=/mnt/watchdog-test/blob.bin \
    bs=1M count=150 status=progress
```

Lancer le watchdog avec un seuil bas :

```bash
sudo env \
    WARN_PERCENT=40 \
    CRIT_PERCENT=60 \
    MIN_FREE_MIB=0 \
    /usr/local/sbin/disk-watchdog /mnt/watchdog-test
```

Observer :

```bash
echo $?
```

et :

```bash
journalctl -t disk-watchdog -n 20 --no-pager
```

## 31.3 Nettoyer le laboratoire

```bash
sudo umount /mnt/watchdog-test
sudo rmdir /mnt/watchdog-test
rm /tmp/watchdog-test.img
```

> [!danger]
> Toujours vérifier le point de montage avant un `rm`, un `dd`, un `mkfs` ou un démontage. Une erreur de périphérique ou de chemin peut détruire des données.

# 32. Tester le timer systemd

Après installation :

```bash
systemctl list-timers disk-watchdog.timer
```

Déclencher le service :

```bash
sudo systemctl start disk-watchdog.service
```

Observer :

```bash
journalctl -u disk-watchdog.service -n 50 --no-pager
```

Vérifier les logs produits par `logger` :

```bash
journalctl -t disk-watchdog -n 50 --no-pager
```

# 33. Dépannage du watchdog

## 33.1 Le script fonctionne à la main mais pas avec systemd

Inspecter :

```bash
systemctl status disk-watchdog.service
```

```bash
journalctl -u disk-watchdog.service -b --no-pager
```

Causes fréquentes :

- chemin incorrect dans `ExecStart=` ;
- script non exécutable ;
- fichier `/etc/default/disk-watchdog` invalide ;
- protection systemd trop restrictive ;
- commande utilisée avec un chemin relatif ;
- dépendance non installée.

## 33.2 Le timer n'apparaît pas

```bash
systemctl status disk-watchdog.timer
```

```bash
systemctl list-timers --all
```

Puis :

```bash
systemctl is-enabled disk-watchdog.timer
```

## 33.3 Les seuils semblent incohérents

Comparer directement :

```bash
df -B1 --output=size,used,avail,pcent,target /
```

et :

```bash
df --output=itotal,iused,iavail,ipcent,target /
```

Vérifier aussi :

```bash
cat /etc/default/disk-watchdog
```

## 33.4 `df` est élevé mais aucun gros fichier n'est visible

Tester :

```bash
sudo lsof +L1
```

Puis examiner :

- snapshots ;
- points de montage ;
- système de fichiers ;
- conteneurs ;
- couche de stockage.

# 34. Méthode de diagnostic en production

Lorsqu'une alerte arrive :

```text
1. Quel système de fichiers est concerné ?
            │
            ▼
2. Blocs, inodes ou espace absolu ?
            │
            ▼
3. La croissance est-elle réelle et récente ?
            │
            ▼
4. Quels répertoires / fichiers l'expliquent ?
            │
            ├── rien de visible -> lsof +L1 / snapshots / montage
            │
            ▼
5. Quelle application produit les données ?
            │
            ▼
6. Peut-on arrêter la croissance sans perte ?
            │
            ▼
7. Nettoyer selon une politique connue
            │
            ▼
8. Corriger la cause et la supervision
```

Ne commencer par supprimer des fichiers qu'après avoir compris le rôle des données.

# 35. Exemple d'exploitation : `/var`

Supposons une alerte :

```text
host=web01 state=CRITICAL target=/var blocks=93% free_mib=620 inodes=17%
```

On apprend immédiatement :

- le problème concerne les blocs ;
- pas les inodes ;
- moins de 1 Gio est disponible ;
- le système de fichiers à examiner est `/var`.

Première étape :

```bash
df -hT /var
```

Puis :

```bash
sudo du -x -h --max-depth=1 /var 2>/dev/null | sort -h
```

Si `/var/log` domine :

```bash
journalctl --disk-usage
sudo du -x -h --max-depth=2 /var/log 2>/dev/null | sort -h | tail
```

Si `/var/lib/docker` domine :

```bash
docker system df -v
```

Si `du` ne retrouve pas l'espace :

```bash
sudo lsof +L1
```

# 36. Capacité versus débit et latence

Un système de fichiers à 40 % peut tout de même être inutilisable si le stockage est saturé en I/O.

La capacité ne dit rien directement sur :

- la latence ;
- les IOPS ;
- le débit ;
- la profondeur de file ;
- le temps d'attente I/O ;
- les erreurs matérielles.

Outils complémentaires :

```bash
iostat
```

```bash
pidstat -d
```

```bash
vmstat
```

Ces outils répondent à d'autres questions et ne doivent pas être confondus avec `df`.

# 37. Quotas

Un utilisateur peut recevoir :

```text
Disk quota exceeded
```

alors que `df` indique encore de l'espace libre.

Selon le système, consulter les quotas avec les outils appropriés, par exemple :

```bash
quota
```

ou pour l'administration :

```bash
repquota
```

Les quotas sont une couche supplémentaire de limitation.

# 38. Supervision de plusieurs points de montage

Exemple :

```bash
/usr/local/sbin/disk-watchdog \
    / \
    /var \
    /srv \
    /home
```

Le script conserve le niveau le plus grave rencontré.

Exemple :

```text
/      -> OK
/var   -> WARNING
/srv   -> CRITICAL
/home  -> OK
```

Le code final sera :

```text
2
```

Il est donc utilisable par un superviseur externe.

# 39. Pourquoi passer des chemins plutôt que des périphériques

La commande :

```bash
df /var
```

retrouve automatiquement le système de fichiers contenant `/var`.

Cela évite de coder en dur :

```text
/dev/sda2
```

ou :

```text
/dev/mapper/vg0-var
```

La configuration reste plus stable si le nom du périphérique change mais que le point de montage reste le même.

# 40. Monitoring et alerting ne sont pas la même chose

**Monitoring** :

```text
collecter + stocker + visualiser
```

**Alerting** :

```text
évaluer une condition + notifier
```

Exemple :

```text
node_exporter
     │
     ▼
Prometheus
     │
     ├── historique
     ├── requêtes
     └── règles
             │
             ▼
        Alertmanager
             │
             ├── mail
             ├── messagerie
             ├── astreinte
             └── silences / routage
```

Coder un `mail` directement dans un script peut suffire pour un laboratoire, mais ne remplace pas ce modèle à l'échelle.

# 41. Checklist de mise en production

Avant de déclarer la surveillance prête :

- [ ] identifier les systèmes de fichiers réellement critiques ;
- [ ] définir warning et critical ;
- [ ] prendre en compte les inodes ;
- [ ] définir une marge absolue si elle a du sens ;
- [ ] tester le script manuellement ;
- [ ] vérifier ses codes de retour ;
- [ ] tester les unités avec `systemd-analyze verify` ;
- [ ] vérifier le timer avec `systemctl list-timers` ;
- [ ] centraliser ou conserver les journaux ;
- [ ] vérifier le canal de notification ;
- [ ] tester réellement une alerte ;
- [ ] documenter la procédure de diagnostic ;
- [ ] surveiller également le stockage physique/pool ;
- [ ] vérifier la politique de rotation/rétention ;
- [ ] éviter toute suppression automatique non maîtrisée.

# 42. TP 1 — Watchdog local

## Objectif

Créer une surveillance de `/` et `/var`.

## Travail demandé

1. installer `/usr/local/sbin/disk-watchdog` ;
2. créer `/etc/default/disk-watchdog` ;
3. lancer le script à la main ;
4. relever les valeurs de blocs et d'inodes ;
5. provoquer un WARNING avec des seuils de test ;
6. vérifier le code de retour ;
7. retrouver le message avec `journalctl`.

## Validation

La commande :

```bash
journalctl -t disk-watchdog
```

doit permettre de retrouver les contrôles.

# 43. TP 2 — systemd timer

## Objectif

Exécuter automatiquement le watchdog.

## Travail demandé

1. créer `disk-watchdog.service` ;
2. créer `disk-watchdog.timer` ;
3. vérifier les fichiers avec `systemd-analyze verify` ;
4. activer le timer ;
5. afficher sa prochaine exécution ;
6. forcer une exécution manuelle ;
7. retrouver les logs ;
8. analyser la sécurité avec `systemd-analyze security`.

# 44. TP 3 — Laboratoire de saturation

## Objectif

Tester sans risquer la partition système.

## Travail demandé

1. créer une image de 256 Mio ;
2. créer un ext4 ;
3. la monter par loop ;
4. observer `df` et `df -i` ;
5. remplir progressivement le volume ;
6. déclencher WARNING puis CRITICAL ;
7. supprimer le fichier de test ;
8. observer le retour à l'état OK ;
9. démonter et supprimer le laboratoire.

# 45. TP 4 — Diagnostic

Le formateur prépare un système de fichiers dont l'espace est consommé par l'une de ces causes :

- gros fichier de log ;
- fichier supprimé encore ouvert ;
- grand nombre de petits fichiers ;
- données Docker ;
- journal systemd volumineux.

L'étudiant doit déterminer :

1. la couche concernée ;
2. blocs ou inodes ;
3. la cause ;
4. une correction sûre ;
5. une prévention durable.

# 46. Questions de révision

1. Quelle différence entre `df` et `du` ?
2. Pourquoi `df -i` est-il important ?
3. Pourquoi 80 % n'a-t-il pas la même signification sur 10 Gio et 10 Tio ?
4. À quoi correspond `Avail` ?
5. Comment trouver un fichier supprimé encore ouvert ?
6. Pourquoi séparer le watchdog du mécanisme d'envoi de mail ?
7. Quel avantage apporte un timer systemd par rapport à cron ?
8. Dans quel cas `Persistent=yes` est-il utile ?
9. Pourquoi `df` ne suffit-il pas pour LVM thin, ZFS ou Btrfs ?
10. Quelle différence entre capacité et santé du stockage ?
11. Quelle métrique node_exporter permet d'obtenir l'espace disponible aux processus non privilégiés ?
12. Pourquoi une suppression automatique de données est-elle dangereuse ?

# 47. Résumé opérationnel

Pour un serveur isolé :

```text
df + df inodes
      │
      ▼
disk-watchdog
      │
      ├── journalctl
      └── code de retour
             │
             ▼
        systemd.timer
             │
             └── notification séparée
```

Pour un parc :

```text
node_exporter
      │
      ▼
Prometheus
      │
      ├── espace
      ├── inodes
      ├── historique
      └── prédiction
             │
             ▼
        Alertmanager
```

À retenir :

> [!summary]
> - Surveiller **les blocs et les inodes**.
> - Regarder aussi l'**espace absolu disponible**.
> - Adapter les seuils au rôle et à la taille du volume.
> - Utiliser un `systemd.timer` pour une intégration locale propre.
> - Journaliser avant de notifier.
> - Diagnostiquer avec `du`, `lsof`, journald et les outils du stockage.
> - Ne pas confondre capacité, performance et santé matérielle.
> - Pour plusieurs machines, préférer une collecte centralisée avec historique et règles d'alerte.
> - Ne jamais automatiser un nettoyage destructif sans politique explicite.

# 48. Références

## GNU Coreutils

Documentation de `df` et de l'utilisation de l'espace :

- GNU Coreutils, `df invocation` ;
- GNU Coreutils, chapitre `File space usage`.

Documentation locale :

```bash
man df
man du
man stat
```

## systemd

```bash
man systemd.timer
man systemd.time
man systemd.service
man systemd.exec
man systemd-analyze
```

Points importants :

- `OnBootSec=` et `OnUnitActiveSec=` pour les timers monotones ;
- `OnCalendar=` pour les calendriers ;
- `Persistent=` pour le rattrapage des événements calendaires manqués ;
- `RandomizedDelaySec=` pour répartir les déclenchements.

## Prometheus

Consulter :

- la documentation Prometheus ;
- le guide officiel de supervision d'un hôte Linux avec node_exporter ;
- la documentation du collecteur `filesystem` de node_exporter.

## Santé du stockage

```bash
man smartctl
man smartd
```

et, pour NVMe :

```bash
man nvme
```

# Annexe — Le script d'origine (2023)

> [!info] Annexe
> Version initiale de cette note : un script minimal `check_disk_usage.sh` lancé par cron, conservé tel quel lors de la refonte du 31 août 2026.

Pour créer un script bash qui envoie un e-mail lorsque l'espace disque est utilisé à 70% ou plus, suivons les étapes ci-dessous:

1.  Ouvrons notre éditeur de texte préféré et créons un nouveau fichier nommé `check_disk_usage.sh`.
    
2.  Copions et collons le contenu suivant dans le fichier `check_disk_usage.sh` :
```bash
#!/bin/bash

# Paramètres
THRESHOLD=70
EMAIL="michaellaunay@ecreall.com"
HOSTNAME=$(hostname)

# Récupérer l'utilisation du disque
DISK_USAGE=$(df --output=pcent / | tail -1 | tr -dc '0-9')

# Vérifier si l'utilisation du disque est supérieure ou égale au seuil
if [ "$DISK_USAGE" -ge "$THRESHOLD" ]; then
  # Envoyer un e-mail d'alerte
  SUBJECT="[$HOSTNAME] Alerte d'espace disque: $DISK_USAGE% utilisé"
  MESSAGE="L'espace disque sur le serveur $HOSTNAME est utilisé à $DISK_USAGE%, ce qui dépasse le seuil d'alerte de $THRESHOLD%."
  echo "$MESSAGE" | mail -s "$SUBJECT" "$EMAIL"
fi

```

Rendons le script exécutable en exécutant la commande suivante :
```bash
chmod +x check_disk_usage.sh
```

Configurons le cron pour exécuter ce script. Ouvrons l'éditeur de la table de cron avec la commande suivante :
```bash
crontab -e
```

Ajoutons la ligne suivante pour exécuter le script toutes les heures (nous pouvons ajuster la fréquence en fonction de nos besoins) :
```bash
0 * * * * /chemin/vers/check_disk_usage.sh
```

N'oublions pas de remplacer `/chemin/vers/` par le chemin absolu vers le fichier `check_disk_usage.sh`.

6.  Sauvegardons et quittons l'éditeur. Le cron est maintenant configuré pour exécuter le script `check_disk_usage.sh` toutes les heures.

Avec cette configuration, un e-mail sera envoyé à `michaellaunay@ecreall.com` lorsque l'utilisation du disque dépasse 70%
