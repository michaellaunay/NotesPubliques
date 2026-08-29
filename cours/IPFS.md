---
schema_version: 1
uid: "01M02EX5B6GHDXC0F77CJ5RR8Y"
titre: "IPFS"
aliases:
  - "InterPlanetary File System"
  - "Kubo"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - reseaux
  - decentralisation
  - stockage
  - ipfs
resume: "Cours complet sur IPFS : adressage par contenu, CID, Merkle DAG, UnixFS et IPLD, réseau libp2p/DHT/Bitswap, installation et exploitation d'un nœud Kubo, épinglage, MFS, CAR, IPNS et DNSLink, passerelles vérifiables, réseaux privés et clusters, sécurité, écosystème, limites et travaux pratiques."
niveau: intermediaire
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-05-07
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# IPFS

> [!abstract] Objectif
> Comprendre ce qu'apporte réellement l'**adressage par contenu** — CID, blocs, Merkle DAG — et comment le réseau IPFS retrouve et échange ces blocs (libp2p, DHT, Bitswap, passerelles), puis savoir installer, exploiter et sécuriser un nœud **Kubo** : ajouter, épingler, publier sous un nom mutable (IPNS, DNSLink), servir par passerelle, transporter hors ligne (CAR) et choisir entre nœud personnel, réseau privé, cluster et service d'épinglage.

> [!info] État du cours
> Vérifié en août 2026 sur **Kubo 0.42.x** (juin 2026), l'implémentation de référence en Go, et sur la documentation officielle (docs.ipfs.tech, specs.ipfs.tech). Les commandes indiquées sont celles de Kubo ; les concepts valent pour toute implémentation (Helia en JavaScript notamment).

IPFS (*InterPlanetary File System*) est un ensemble de protocoles ouverts pour **nommer, retrouver et échanger des données par leur contenu** plutôt que par leur emplacement. Là où le Web classique demande « le fichier qui se trouve sur ce serveur, à ce chemin », IPFS demande « les blocs dont voici l'empreinte », et n'importe quel nœud qui les possède peut les fournir.

Voir aussi : [[git]] (un autre système adressé par le contenu), [[HTTP]], [[Les protocoles de communications]], [[Docker]], [[Sécurité avancée sous Linux]].

# Sommaire

1. Introduction : décentralisation et adressage par contenu
2. Le modèle de données : CID, blocs, Merkle DAG, UnixFS et IPLD
3. Le réseau : libp2p, DHT Amino, IPNI, routage délégué et Bitswap
4. Installer Kubo et lancer un nœud
5. Ajouter, retrouver et épingler du contenu
6. MFS, CAR et manipulations avancées
7. Noms mutables : IPNS et DNSLink
8. Passerelles HTTP et récupération vérifiable
9. Réseaux privés, clusters et services d'épinglage
10. Sécurité, vie privée et exploitation
11. Cas d'utilisation et écosystème
12. Limites et défis
13. Travaux pratiques
14. Aide-mémoire
15. Sources et ressources
16. Conclusion

# 1. Introduction : décentralisation et adressage par contenu

## 1.1 Adressage par emplacement et adressage par contenu

Une URL HTTP désigne un **emplacement** : un hôte, un chemin, et implicitement une autorité qui décide du contenu servi à cet endroit. Si le serveur disparaît, change d'avis ou est censuré, le lien meurt ou ment.

Un identifiant IPFS, le **CID** (*Content IDentifier*), désigne un **contenu** : il est calculé à partir des octets eux-mêmes. Trois conséquences en découlent :

- **vérifiabilité** : quiconque reçoit les données peut recalculer le CID et détecter toute altération ;
- **indépendance de la source** : un ami, un miroir, une passerelle ou une clé USB peuvent tous fournir le même contenu sous le même identifiant ;
- **immuabilité** : modifier le contenu produit un autre CID ; les noms mutables (IPNS, DNSLink) sont une couche séparée.

```text
Web classique  : https://exemple.org/rapport.pdf   -> "ce que ce serveur sert ici, aujourd'hui"
IPFS           : /ipfs/bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi
                                                   -> "exactement ces octets, d'où qu'ils viennent"
```

## 1.2 Ce qu'IPFS est, et ce qu'il n'est pas

IPFS **est** :

- un format d'identifiants (CID) et de structures de données (Merkle DAG, UnixFS, IPLD) ;
- un ensemble de protocoles réseau pour trouver qui possède un bloc et l'échanger (libp2p, DHT, Bitswap, passerelles) ;
- des implémentations : **Kubo** (Go, référence), **Helia** (JavaScript), des bibliothèques (Boxo en Go) et des outils (IPFS Desktop, IPFS Companion, ipfs-cluster).

IPFS **n'est pas** :

- un stockage automatique et gratuit : un contenu n'existe sur le réseau que tant qu'au moins un nœud le conserve (**épinglage**) ;
- un système chiffré : tout ce qui est ajouté au réseau public est lisible par quiconque connaît le CID ;
- une blockchain : aucun consensus global, aucune monnaie ; Filecoin, réseau distinct, ajoute les incitations économiques à la persistance ;
- un moyen d'effacer : on ne peut retirer que sa propre copie.

## 1.3 Un peu d'histoire

- 2014 : Juan Benet publie le *whitepaper* IPFS chez Protocol Labs ; première version publique en 2015.
- 2015-2021 : go-ipfs devient l'implémentation de référence ; naissance de libp2p, d'IPLD et de Filecoin (2020).
- 2022 : go-ipfs est renommé **Kubo** ; js-ipfs cède la place à **Helia** (2023).
- 2023-2024 : la DHT publique prend le nom d'**Amino** ; les passerelles *trustless* et le routage délégué HTTP sont spécifiés ; Cloudflare ferme sa passerelle publique.
- 2025-2026 : Kubo est maintenu par l'équipe **Shipyard**, l'IPFS Foundation coordonne les spécifications ; IPIP-499 fixe des **profils de CID** reproductibles entre implémentations (Kubo 0.40, février 2026) ; `ipfs update` intégré (0.41), export partiel de CAR (0.42).

## 1.4 Pourquoi la décentralisation compte

- **Résilience** : pas de point unique de défaillance ni de panne d'hébergeur.
- **Intégrité** : les données sont vérifiées à la réception, pas supposées correctes parce qu'elles viennent du « bon » serveur.
- **Localité** : un contenu populaire est servi par les nœuds proches qui l'ont déjà ; un campus, une salle de TP ou une constellation de satellites peuvent partager sans remonter à l'origine.
- **Résistance à la censure et à la disparition** : un identifiant ne dépend d'aucun nom de domaine ni d'aucune entreprise.

Ces avantages ont un prix : la persistance, la vie privée et la performance ne sont plus des propriétés automatiques, mais des choix d'exploitation (chapitres 9, 10 et 12).

# 2. Le modèle de données : CID, blocs, Merkle DAG, UnixFS et IPLD

## 2.1 Anatomie d'un CID

Un CID est un identifiant auto-descriptif composé de quatre parties :

```text
CIDv1 = <multibase><version><multicodec><multihash>
          |           |        |           |
          |           |        |           +-- fonction de hachage + longueur + empreinte (sha2-256 par défaut)
          |           |        +-- format du bloc : dag-pb (UnixFS), raw, dag-cbor, dag-json...
          |           +-- version du CID : 1
          +-- encodage de la chaîne : base32 (b...), base58btc (z...), base36 (k...)
```

Deux versions coexistent :

| | CIDv0 | CIDv1 |
|---|---|---|
| Aspect | `QmXoypizj...` (base58btc, 46 caractères) | `bafybeigdyrzt...` (base32, insensible à la casse) |
| Codec | implicitement dag-pb | explicite : `bafybei…` dag-pb, `bafkrei…` raw, `bafyrei…` dag-cbor |
| Hachage | sha2-256 uniquement | tout multihash (sha2-256, blake3...) |
| Usage | historique, encore très répandu | recommandé, obligatoire pour les passerelles par sous-domaine |

Un CIDv0 se convertit sans perte en CIDv1 ; l'inverse n'est possible que pour dag-pb + sha2-256 :

```bash
ipfs cid format -v 1 -b base32 QmXoypizjW3WknFiJnKLwHCnL72vedxjQkDDP1mXWo6uco
ipfs cid inspect bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi   # Kubo >= 0.41
```

> [!important]
> Le CID identifie une **structure de blocs**, pas seulement des octets : le même fichier découpé avec d'autres paramètres (taille des morceaux, feuilles brutes ou non, version de CID) donne un CID différent. C'est ce que règlent les profils de CID (section 2.5).

## 2.2 Blocs et Merkle DAG

IPFS ne manipule pas des fichiers mais des **blocs** de taille bornée (256 Kio à 1 Mio en pratique, 2 Mio au maximum pour Bitswap). Un fichier volumineux est découpé en blocs-feuilles, puis des blocs intermédiaires listent les CID de leurs enfants : on obtient un **Merkle DAG** (graphe orienté acyclique dont chaque nœud contient les empreintes de ses descendants).

```text
                    racine (CID du fichier)
                   /        |         \
             bloc 1       bloc 2      bloc 3        <- morceaux de 1 Mio (raw)
```

Propriétés qui découlent de cette structure :

- la racine authentifie l'ensemble : vérifier un fichier de 10 Go revient à vérifier une chaîne de hachages ;
- deux fichiers qui partagent des morceaux identiques partagent des blocs (**déduplication**) ;
- un bloc peut être téléchargé depuis n'importe quel pair et vérifié isolément, ce qui autorise le téléchargement parallèle ;
- un répertoire est lui-même un bloc listant les CID de ses entrées : le CID d'un dossier change dès qu'un fichier change (comme un `tree` dans [[git]]).

## 2.3 UnixFS : fichiers et répertoires

**UnixFS** est le format (encodé en dag-pb) qui représente fichiers, répertoires, liens symboliques et métadonnées au-dessus des blocs. Un répertoire trop grand pour tenir dans un bloc est automatiquement transformé en **HAMT** (*Hash Array Mapped Trie*) réparti sur plusieurs blocs (« sharding »).

```bash
ipfs ls bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi   # entrées d'un répertoire
ipfs dag stat <cid>                                                  # nombre de blocs et taille totale
ipfs block stat <cid>                                                 # taille d'un bloc
```

## 2.4 IPLD : au-delà des fichiers

**IPLD** (*InterPlanetary Linked Data*) généralise l'idée : n'importe quelle structure de données (objet JSON, enregistrement, arbre) peut être encodée en **dag-cbor** ou **dag-json**, contenir des liens vers d'autres CID et être parcourue par chemin.

```bash
echo '{"titre": "Cours IPFS", "auteur": "Michaël", "chapitres": 16}' | ipfs dag put
# bafyreib...

ipfs dag get bafyreib.../titre
```

`ipfs dag get` permet de suivre un chemin à travers des liens : `bafyrei.../chapitres/0/titre`. C'est la brique utilisée par les métadonnées de NFT, les bases de données décentralisées et les formats d'archive vérifiables.

## 2.5 Profils de CID (IPIP-499)

Jusqu'en 2025, Kubo, Helia et d'autres outils produisaient des CID différents pour le même fichier. **IPIP-499** définit deux profils qui fixent tous les paramètres de construction du DAG :

| Profil | Contenu |
|---|---|
| `unixfs-v1-2025` (recommandé) | CIDv1, feuilles brutes, sha2-256, morceaux de 1 Mio, 1024 liens par nœud, HAMT à 256 branches |
| `unixfs-v0-2015` (alias `legacy-cid-v0`) | CIDv0, sans feuilles brutes, morceaux de 256 Kio : reproduit les CID historiques |

```bash
ipfs config profile apply unixfs-v1-2025
ipfs config Import      # affiche les paramètres d'import en vigueur
```

> [!tip]
> Si des CID doivent rester identiques pendant des années (références publiées, archives, contrats), appliquer explicitement un profil : les valeurs implicites de `ipfs add` peuvent changer d'une version de Kubo à l'autre.

# 3. Le réseau : libp2p, DHT Amino, IPNI, routage délégué et Bitswap

Retrouver un contenu se fait en deux temps : **qui** possède les blocs (routage de contenu), puis **comment** les obtenir (échange de blocs).

## 3.1 libp2p

**libp2p** est la bibliothèque réseau modulaire née d'IPFS, aujourd'hui utilisée bien au-delà (Ethereum, Filecoin, Polkadot...). Elle fournit :

- une identité par nœud, le **PeerID**, dérivé d'une clé publique ;
- des **multiadresses** auto-descriptives : `/ip4/203.0.113.7/udp/4001/quic-v1/p2p/12D3KooW...` ;
- plusieurs transports (TCP, QUIC, WebSocket, WebTransport, WebRTC) et le chiffrement des connexions (Noise, TLS) ;
- la traversée de NAT : AutoNAT pour se savoir joignable, relais circuit v2 et perçage de trous (DCUtR) ;
- un **gestionnaire de ressources** qui borne connexions, flux et mémoire.

```bash
ipfs id                      # PeerID et adresses annoncées
ipfs swarm peers | wc -l     # pairs connectés
ipfs swarm addrs local
```

## 3.2 La DHT Amino

La **DHT** (*Distributed Hash Table*, algorithme Kademlia) publique d'IPFS s'appelle **Amino**. Elle stocke trois types d'enregistrements :

- **provider records** : « le pair P possède le CID C » ;
- **peer records** : les adresses d'un PeerID ;
- **IPNS records** : la valeur signée d'un nom IPNS.

Un nœud **fournit** (*provide*) régulièrement ses contenus : les enregistrements expirent (48 h) et doivent être réannoncés. Depuis Kubo 0.39, l'annonce se fait par balayage (*sweep*) pour tenir sur des dizaines de millions de CID ; la stratégie `Provide.Strategy` (`all`, `pinned`, `roots`, `mfs`) choisit ce qui est annoncé.

```bash
ipfs routing findprovs <cid>       # qui fournit ce CID ? (remplace l'ancien ipfs dht findprovs)
ipfs routing findpeer <peerid>
ipfs provide stat                  # état de la file d'annonces
```

Un nœud derrière un NAT ou peu fiable peut se limiter au mode **client** (`Routing.Type=autoclient`) : il interroge la DHT sans y servir d'enregistrements.

## 3.3 IPNI et routage délégué

Pour les très gros fournisseurs (services d'épinglage, Filecoin), la DHT ne suffit pas : l'**IPNI** (*InterPlanetary Network Indexer*, `cid.contact`) indexe des milliards de CID annoncés par lots. Kubo interroge la DHT **et** un indexeur par défaut (`Routing.Type=auto`).

Le **routage délégué** expose ces recherches par une simple API HTTP (`/routing/v1/providers/<cid>`), qu'un navigateur ou un client léger peut appeler sans participer à la DHT. Un nœud Kubo peut lui-même servir cette API (`Gateway.ExposeRoutingAPI`).

## 3.4 Bitswap et récupération HTTP

**Bitswap** est le protocole d'échange de blocs : un nœud diffuse une liste de souhaits (*wantlist*) à ses pairs, distingue « as-tu ce bloc ? » (*want-have*) de « envoie-le » (*want-block*), reçoit les blocs, les vérifie et les redistribue. Le téléchargement est naturellement parallèle et incrémental.

```bash
ipfs bitswap wantlist
ipfs bitswap stat
ipfs stats bw
```

Depuis 2025, Kubo sait aussi récupérer des blocs **par HTTP** auprès de passerelles *trustless* (chapitre 8), ce qui rapproche IPFS d'un CDN vérifiable et facilite l'accès depuis des environnements où le pair-à-pair est bridé.

# 4. Installer Kubo et lancer un nœud

## 4.1 Installation

Les binaires officiels sont publiés sur <https://dist.ipfs.tech> et sur GitHub :

```bash
# Linux amd64 — vérifier la dernière version sur dist.ipfs.tech
VERSION=v0.42.0
curl -LO "https://dist.ipfs.tech/kubo/${VERSION}/kubo_${VERSION}_linux-amd64.tar.gz"
tar -xzf "kubo_${VERSION}_linux-amd64.tar.gz"
cd kubo && sudo bash install.sh
ipfs version --all
```

Autres méthodes :

- **Docker** : image officielle `ipfs/kubo` (`docker run -d --name ipfs -v ipfs_staging:/export -v ipfs_data:/data/ipfs -p 4001:4001 -p 4001:4001/udp -p 127.0.0.1:5001:5001 -p 127.0.0.1:8080:8080 ipfs/kubo:release`) ;
- **IPFS Desktop** : application graphique (Windows, macOS, Linux) qui embarque Kubo et l'interface Web ;
- paquets communautaires des distributions (souvent en retard d'une ou deux versions) ;
- compilation depuis les sources (`make build`, Go 1.26 requis).

Depuis Kubo 0.41, la mise à jour est intégrée et remplace l'ancien outil `ipfs-update` :

```bash
ipfs update check
ipfs update install      # sauvegarde l'ancien binaire ; ipfs update revert pour revenir en arrière
```

## 4.2 Initialisation

```bash
ipfs init                        # crée ~/.ipfs (variable IPFS_PATH) : clés, config, dépôt de blocs
ipfs init --profile server       # sur une machine avec IP publique : pas de découverte locale, pas d'annonce d'adresses privées
ipfs init --profile lowpower     # portable, Raspberry Pi : moins de connexions et d'annonces
```

Les profils se combinent et s'appliquent aussi après coup avec `ipfs config profile apply`. Le dépôt utilise par défaut le stockage `flatfs` (un fichier par bloc) avec un index LevelDB ; `pebbleds` est disponible pour les gros nœuds.

## 4.3 Démarrer le démon

```bash
ipfs daemon
```

Le démon ouvre trois services :

| Port | Rôle | Exposition |
|---|---|---|
| 4001 TCP et UDP (QUIC) | échanges pair-à-pair (swarm) | Internet |
| 5001 | API RPC (`/api/v0/…`) et interface Web `http://127.0.0.1:5001/webui` | **loopback uniquement** |
| 8080 | passerelle HTTP | loopback par défaut |

> [!danger]
> L'API du port 5001 permet de lire la configuration, publier sous vos clés et supprimer des données : elle ne doit **jamais** être exposée sur une interface publique. Ne modifiez `Addresses.API` qu'en connaissance de cause et préférez un tunnel SSH.

Service systemd minimal (utilisateur dédié `ipfs`, dépôt dans `/var/lib/ipfs`) :

```ini
[Unit]
Description=Kubo IPFS daemon
After=network-online.target
Wants=network-online.target

[Service]
User=ipfs
Environment=IPFS_PATH=/var/lib/ipfs
ExecStart=/usr/local/bin/ipfs daemon --enable-gc --migrate
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

## 4.4 Vérifier et observer

```bash
ipfs id
ipfs swarm peers
ipfs repo stat --human
ipfs config show | less
ipfs config Addresses.Gateway
ipfs stats bw --interval 5s
```

L'interface Web (`/webui`) affiche l'état du nœud, les pairs connectés et permet d'explorer, épingler et parcourir des CID.

# 5. Ajouter, retrouver et épingler du contenu

## 5.1 Ajouter

```bash
echo "Bonjour IPFS" > bonjour.txt
ipfs add bonjour.txt
# added bafkreig... bonjour.txt

ipfs add -r site/            # récursif : chaque fichier reçoit un CID, puis le répertoire
ipfs add -w -r site/         # -w enveloppe le tout dans un répertoire pour conserver le nom
ipfs add -Q --only-hash gros-fichier.iso     # calcule le CID sans rien stocker
ipfs add --cid-version 1 rapport.pdf         # inutile si le profil unixfs-v1-2025 est appliqué
```

Ce qui se passe réellement : le fichier est découpé en blocs, les blocs sont écrits dans le dépôt local, le CID racine est **épinglé** (sauf `--pin=false`) et sera **annoncé** à la DHT par le démon selon `Provide.Strategy`. Rien n'est « envoyé » nulle part : tant que votre nœud est le seul à posséder les blocs, il est la seule source.

## 5.2 Retrouver

```bash
ipfs cat bafkreig...                       # affiche un fichier
ipfs cat bafybeig.../index.html            # chemin dans un répertoire
ipfs get bafybeig... -o copie/             # récupère un fichier ou une arborescence sur disque
ipfs ls bafybeig...
ipfs resolve /ipfs/bafybeig.../images/logo.png    # CID de l'élément désigné par le chemin
```

Si les blocs ne sont pas dans le dépôt local, Kubo interroge le routage (DHT, IPNI) puis Bitswap. Un premier accès à un contenu rare peut prendre plusieurs secondes ; un contenu inexistant ne renverra jamais d'erreur « introuvable » définitive, seulement un délai : penser à `--timeout` (`ipfs --timeout 30s cat <cid>`).

## 5.3 Épingler : la seule garantie de persistance

Tout bloc téléchargé pour lecture entre dans le **cache** du dépôt et sera supprimé par le ramasse-miettes. Seuls les contenus **épinglés** sont conservés.

```bash
ipfs pin add --name "cours-ipfs-2026" bafybeig...   # épinglage récursif (le DAG entier)
ipfs pin ls --type=recursive --names
ipfs pin rm bafybeig...
ipfs pin update <ancien-cid> <nouveau-cid>          # remplace une version par une autre en évitant de retélécharger les blocs communs
ipfs pin verify
```

Types d'épingle : `recursive` (le DAG complet), `direct` (un seul bloc) et `indirect` (bloc conservé parce qu'un ancêtre est épinglé). Les fichiers de la MFS (chapitre 6) sont protégés implicitement.

## 5.4 Ramasse-miettes et quotas

```bash
ipfs repo gc                       # supprime les blocs ni épinglés ni référencés par la MFS
ipfs config Datastore.StorageMax   # 10GB par défaut
ipfs daemon --enable-gc            # GC automatique quand StorageMax est approché (Datastore.GCPeriod)
```

Un nœud dont le dépôt grossit sans limite n'a généralement pas activé le GC, ou épingle par mégarde tout ce qu'il consulte (la passerelle locale n'épingle pas ; `ipfs get`/`ipfs cat` non plus).

# 6. MFS, CAR et manipulations avancées

## 6.1 Le système de fichiers mutable (MFS)

La **MFS** (*Mutable File System*) offre une vue « répertoire personnel » sur des CID : on y crée, copie, déplace des fichiers avec des commandes familières, et Kubo recalcule les CID des répertoires parents à chaque modification. Elle sert de zone de travail avant publication.

```bash
ipfs files mkdir -p /sites/cours
ipfs files cp /ipfs/bafybeig... /sites/cours/index.html   # référence un contenu existant sans le copier
ipfs files write --create --parents /sites/cours/notes.txt notes.txt
ipfs files ls -l /sites/cours
ipfs files stat /sites/cours           # le CID courant du répertoire
ipfs files rm -r /sites/ancien
ipfs files chcid --cid-version 1 /sites   # migrer une arborescence vers CIDv1
```

Le CID renvoyé par `ipfs files stat /sites/cours` est exactement celui que l'on publiera (IPNS, DNSLink) ou que l'on partagera.

## 6.2 Archives CAR : transporter et archiver

Un fichier **CAR** (*Content Addressable aRchive*) contient des blocs et leurs CID : c'est le format d'échange hors ligne, de sauvegarde et de dépôt chez les services d'épinglage ou Filecoin.

```bash
ipfs dag export bafybeig... > cours.car        # tout le DAG
ipfs dag export --local-only bafybeig... > partiel.car   # Kubo >= 0.42 : seulement les blocs présents localement
ipfs dag import cours.car                      # sur une autre machine : les blocs sont importés et la racine épinglée
```

Comme le CID est recalculé à l'import, une archive CAR déposée sur une clé USB, envoyée par courriel ou stockée dans un objet S3 reste vérifiable.

## 6.3 Fournir à la demande et inspecter le dépôt

```bash
ipfs provide once bafybeig...      # annoncer immédiatement un CID (Kubo >= 0.42 ; ipfs routing provide auparavant)
ipfs refs -r bafybeig...           # tous les CID du DAG
ipfs refs local | wc -l            # blocs présents localement
ipfs repo verify
ipfs repo migrate                  # après une mise à jour majeure du format de dépôt
```

# 7. Noms mutables : IPNS et DNSLink

Un CID ne changera jamais ; un site, lui, évolue. Deux mécanismes fournissent un nom **stable** pointant vers un CID **variable**.

## 7.1 IPNS

Un nom IPNS est le hachage d'une **clé publique** ; seul le détenteur de la clé privée peut publier un nouvel enregistrement, signé et daté, qui associe le nom à un chemin `/ipfs/...`.

```bash
ipfs key gen cours              # nouvelle paire de clés, nommée localement "cours"
ipfs key list -l
ipfs name publish --key=cours /ipfs/bafybeig...
# Published to k51qzi5uqu5d...: /ipfs/bafybeig...

ipfs name resolve k51qzi5uqu5d...
ipfs cat /ipns/k51qzi5uqu5d.../index.html
```

Points d'exploitation :

- l'enregistrement a une durée de validité (`--lifetime`, 48 h par défaut) et le démon le **republie** périodiquement (`Ipns.RepublishPeriod`) : un nœud éteint plusieurs jours laisse son nom expirer ;
- `--ttl` contrôle la mise en cache par les résolveurs ; une valeur courte accélère la propagation d'une mise à jour, une valeur longue soulage la DHT ;
- la clé `self` (le PeerID du nœud) est un nom IPNS utilisable directement, mais il est préférable de créer une clé par site pour pouvoir la sauvegarder et la déplacer (`ipfs key export` / `ipfs key import`) ;
- la résolution IPNS par la DHT prend quelques secondes ; les passerelles et les navigateurs la mettent en cache.

## 7.2 DNSLink

**DNSLink** relie un nom de domaine ordinaire à un chemin IPFS par un enregistrement DNS TXT :

```text
_dnslink.cours.exemple.org.  IN TXT  "dnslink=/ipfs/bafybeig..."
```

ou, pour ne mettre à jour que l'IPNS :

```text
_dnslink.cours.exemple.org.  IN TXT  "dnslink=/ipns/k51qzi5uqu5d..."
```

```bash
ipfs resolve /ipns/cours.exemple.org
ipfs dns cours.exemple.org
```

Un domaine DNSLink se consulte sur toute passerelle : `https://ipfs.io/ipns/cours.exemple.org/` ou, sur un domaine servi par sa propre passerelle, directement `https://cours.exemple.org/`. C'est le mécanisme utilisé par la documentation IPFS, par des sites institutionnels et par de nombreux portails Web3.

# 8. Passerelles HTTP et récupération vérifiable

Une **passerelle** est un serveur HTTP qui lit des contenus IPFS pour des clients qui ne parlent pas le protocole (navigateurs, `curl`, scripts).

## 8.1 Passerelle locale

```bash
curl http://127.0.0.1:8080/ipfs/bafkreig.../
curl -H "Accept: application/vnd.ipld.raw" http://127.0.0.1:8080/ipfs/bafkreig...   # bloc brut
```

Deux formes d'URL existent :

- **chemin** : `http://127.0.0.1:8080/ipfs/<cid>/index.html` — simple, mais tous les sites partagent la même **origine** Web, ce qui casse l'isolation des cookies et du stockage ;
- **sous-domaine** : `http://<cidv1>.ipfs.localhost:8080/` — chaque contenu a sa propre origine, comme un site distinct ; c'est la forme recommandée pour les applications Web (et la raison pour laquelle CIDv1 en base32, insensible à la casse, est nécessaire).

`ipfs://<cid>` et `ipns://<nom>` sont compris nativement par Brave et par l'extension **IPFS Companion**, qui redirigent vers le nœud local.

## 8.2 Passerelles publiques et passerelles de confiance

Des passerelles publiques (`ipfs.io`, `dweb.link`, `trustless-gateway.link`, celles des services d'épinglage) servent n'importe quel CID. Elles sont pratiques mais réintroduisent un intermédiaire : disponibilité, journalisation, blocage de contenus, et surtout **confiance** — une passerelle « classique » renvoie des octets que le client n'a pas vérifiés.

Les **passerelles trustless** résolvent ce problème : elles ne renvoient que des blocs ou des archives CAR, que le client vérifie lui-même.

```bash
curl -H "Accept: application/vnd.ipld.car" "https://trustless-gateway.link/ipfs/bafybeig...?dag-scope=all" -o site.car
ipfs dag import site.car        # vérification locale de chaque bloc
```

Côté navigateur, `@helia/verified-fetch` remplace `fetch()` par une version qui récupère les blocs (par passerelles trustless ou pair-à-pair) et vérifie les CID avant de livrer le contenu.

## 8.3 Exposer sa propre passerelle

Servir un site public depuis son nœud est légitime, mais demande quelques réglages :

```bash
ipfs config --json Gateway.NoFetch true                   # ne sert que le contenu local, n'explore pas le réseau à la demande d'inconnus
ipfs config --json Gateway.DeserializedResponses false    # mode trustless uniquement (blocs, CAR)
ipfs config --json Gateway.PublicGateways '{"cours.exemple.org": {"Paths": ["/ipfs", "/ipns"], "UseSubdomains": false}}'
```

Un serveur HTTP frontal (nginx, Caddy) ajoute TLS, cache et journalisation. Une passerelle ouverte à tout le réseau devient vite une cible (contenus illicites, bande passante) : `Gateway.NoFetch` et une liste de refus (section 10.4) sont indispensables.

# 9. Réseaux privés, clusters et services d'épinglage

## 9.1 Réseau privé

Un **réseau privé** IPFS n'échange qu'avec les nœuds qui partagent une clé secrète (`swarm.key`) et ne se connecte pas au réseau public. C'est la configuration adaptée à une entreprise, un laboratoire ou une salle de TP.

```bash
# sur une machine, générer la clé partagée
printf '/key/swarm/psk/1.0.0/\n/base16/\n%s\n' "$(tr -dc 'a-f0-9' < /dev/urandom | head -c 64)" > swarm.key

# sur chaque nœud : copier swarm.key dans $IPFS_PATH, retirer les amorces publiques, ajouter les siennes
cp swarm.key ~/.ipfs/
ipfs bootstrap rm --all
ipfs bootstrap add /ip4/10.0.0.10/tcp/4001/p2p/12D3KooW...
export LIBP2P_FORCE_PNET=1        # refuse de démarrer sans swarm.key
ipfs daemon
```

Les passerelles publiques et les indexeurs n'ont plus aucun rôle : la persistance et la disponibilité reposent entièrement sur les nœuds du réseau privé.

## 9.2 IPFS Cluster

**ipfs-cluster** coordonne un ensemble de nœuds Kubo pour maintenir un **jeu d'épingles répliqué** (`ipfs-cluster-service` sur chaque machine, `ipfs-cluster-ctl` pour administrer, `ipfs-cluster-follow` pour suivre un cluster public en lecture seule).

```bash
ipfs-cluster-ctl peers ls
ipfs-cluster-ctl pin add --replication-min 2 --replication-max 3 bafybeig...
ipfs-cluster-ctl status bafybeig...
```

Le cluster garantit qu'un contenu reste épinglé sur *n* machines même si l'une tombe, et rééquilibre automatiquement.

## 9.3 Services d'épinglage

Un **service d'épinglage** conserve vos contenus sur ses nœuds contre abonnement. L'**API d'épinglage standard** (*Pinning Service API*) permet de piloter Pinata, Filebase, Storacha (ex-web3.storage), 4EVERLAND et d'autres depuis Kubo :

```bash
ipfs pin remote service add pinata https://api.pinata.cloud/psa "$PINATA_TOKEN"
ipfs pin remote add --service=pinata --name="cours-ipfs" bafybeig...
ipfs pin remote ls --service=pinata --status=queued,pinning,pinned
```

Le contenu est alors disponible depuis le service même lorsque votre nœud est éteint. **Filecoin** va plus loin : un fournisseur de stockage s'engage cryptographiquement, contre rémunération, à conserver les données pendant une durée contractuelle — c'est la réponse économique au problème de la persistance.

## 9.4 Quel dispositif choisir ?

| Besoin | Solution |
|---|---|
| Partager un contenu quelques jours, expérimenter | nœud personnel + passerelle publique |
| Site ou documentation stable | nœud dédié (`server`) + DNSLink + service d'épinglage en secours |
| Données internes, TP, laboratoire | réseau privé |
| Haute disponibilité de milliers d'objets | ipfs-cluster (et/ou service d'épinglage) |
| Archivage long terme avec garantie | Filecoin, via un service ou un agrégateur |

# 10. Sécurité, vie privée et exploitation

## 10.1 Ce qui est public

- **Le contenu** : tout ce qui est ajouté au réseau public est lisible par quiconque obtient le CID ; les CID se devinent rarement mais se propagent (passerelles, journaux, dépôts Git). Chiffrer avant d'ajouter est la seule protection.
- **Votre PeerID et vos adresses IP** : tout pair connecté les connaît ; vos annonces DHT révèlent ce que vous fournissez.
- **Ce que vous consultez** : demander un CID sur la DHT et Bitswap est observable par les pairs interrogés ; une passerelle publique voit tout ce qui passe par elle.

Pour un contenu sensible : chiffrer (par exemple avec `age` ou un chiffrement applicatif), utiliser un réseau privé, ou ne pas utiliser IPFS.

## 10.2 Effacement et droit à l'oubli

Retirer une épingle et lancer le GC supprime **votre** copie ; les autres nœuds, passerelles et services qui l'ont récupérée la conservent tant qu'ils le souhaitent. Publier sous IPFS des données personnelles engage donc une responsabilité durable ([[Règlement Général sur la Protection des Données (RGPD)]]).

## 10.3 Durcir un nœud

- garder l'API RPC sur le loopback ; pour l'interface Web depuis une autre machine, utiliser un tunnel SSH (`ssh -L 5001:127.0.0.1:5001 serveur`) ;
- appliquer le profil `server` sur une machine publique ;
- borner les ressources : `Swarm.ResourceMgr` (activé par défaut), `Swarm.ConnMgr.HighWater`, `Datastore.StorageMax` ;
- **AutoTLS** (Kubo ≥ 0.33) obtient automatiquement un certificat pour `*.libp2p.direct` afin d'accepter des connexions WebSocket sécurisées depuis les navigateurs : utile pour un nœud public, inutile derrière un NAT ;
- mettre à jour régulièrement (`ipfs update check`) et surveiller les métriques Prometheus exposées sur `http://127.0.0.1:5001/debug/metrics/prometheus` ;
- exécuter le démon sous un utilisateur dédié, sans droits sur d'autres données (voir [[Sécurité avancée sous Linux]]).

## 10.4 Listes de refus

Le greffon **nopfs**, intégré à Kubo, applique des listes de refus au format `.deny` déposées dans `$IPFS_PATH/denylists/` : le nœud refuse de servir, épingler ou relayer les CID listés. La liste communautaire *Bad Bits* recense les contenus signalés ; un exploitant de passerelle publique doit l'appliquer et documenter sa procédure de signalement.

## 10.5 Sauvegarder un nœud

Un nœud se résume à : ses **clés** (`ipfs key export`, et le fichier `config` qui contient l'identité), sa **liste d'épingles** (`ipfs pin ls --type=recursive --names`) et sa **MFS** (`ipfs files stat /`). Les blocs eux-mêmes se réexportent en CAR ou se retrouvent sur le réseau tant qu'un autre nœud les épingle — d'où l'intérêt d'un second nœud ou d'un service d'épinglage.

# 11. Cas d'utilisation et écosystème

## 11.1 Web décentralisé

Un site statique (documentation, blog, application front-end) se publie en trois commandes : `ipfs add -r`, `ipfs name publish` ou mise à jour DNSLink, et se sert par n'importe quelle passerelle. De nombreuses interfaces Web3 (portefeuilles, DEX) publient ainsi leur front-end, parfois adressé par ENS (`nom.eth` → `contenthash` IPFS), pour qu'aucun hébergeur ne puisse le faire disparaître.

## 11.2 Blockchains et NFT

Une blockchain stocke mal les gros objets ; elle stocke en revanche très bien un **CID**. Les métadonnées et images de NFT sont référencées par des URI `ipfs://bafy...` : l'engagement sur la chaîne authentifie le contenu sans le contenir. La persistance reste à organiser (service d'épinglage, Filecoin) — un NFT dont personne n'épingle l'image n'affiche plus rien.

## 11.3 Données scientifiques et archives

Des jeux de données volumineux (génomique, climat, corpus) sont distribués par CID : citation stable, vérification d'intégrité, téléchargement depuis le miroir le plus proche, mise à jour incrémentale (seuls les blocs modifiés changent). Les archives CAR se déposent dans des entrepôts institutionnels ou sur Filecoin.

## 11.4 Distribution logicielle et paquets

Des distributions Linux, des registres de paquets et des dépôts de modèles expérimentent la distribution par IPFS : déduplication entre versions, cache pair-à-pair sur un campus, vérification native. Le rapprochement avec [[git]] est direct : les deux systèmes adressent par contenu et échangent des objets vérifiables.

## 11.5 L'écosystème en 2026

| Composant | Rôle |
|---|---|
| **Kubo** | nœud de référence (Go), CLI, passerelle, RPC |
| **Helia** et `@helia/verified-fetch` | implémentation JavaScript pour Node.js et navigateurs |
| **Boxo** | bibliothèques Go réutilisables (UnixFS, Bitswap, passerelle) |
| **libp2p** | couche réseau |
| **IPFS Desktop**, **IPFS Companion** | intégration poste de travail et navigateur |
| **ipfs-cluster** | réplication d'épingles |
| **IPNI** (`cid.contact`), **someguy** | indexation et routage délégué |
| **Filecoin** | persistance rémunérée |
| **IPFS Foundation**, **Shipyard** | spécifications (specs.ipfs.tech, IPIP) et maintenance de Kubo |

# 12. Limites et défis

## 12.1 Persistance

Rien ne garantit qu'un CID reste disponible : la moitié des « liens morts » IPFS sont des contenus que plus personne n'épingle. Toute publication sérieuse suppose une stratégie d'épinglage explicite (chapitre 9).

## 12.2 Performance et latence

Une résolution DHT « à froid » prend de une à plusieurs secondes ; un contenu fourni par un seul nœud derrière un NAT peut être lent ou inaccessible. Les remèdes sont connus mais recentralisent partiellement : passerelles, services d'épinglage, indexeurs, récupération HTTP.

## 12.3 Vie privée et modération

Le protocole ne cache ni ce que l'on publie ni ce que l'on consulte. La modération des passerelles publiques (listes de refus, retrait) crée des points de contrôle que le protocole cherchait précisément à éviter : un équilibre encore débattu.

## 12.4 Ressources et expérience utilisateur

Un nœud complet consomme bande passante, connexions et espace disque ; sur un portable ou un mobile, on lui préfère un client léger (routage délégué, passerelles trustless, Helia dans le navigateur). L'intégration native dans les navigateurs reste partielle.

## 12.5 Évolution du protocole

Les commandes et formats évoluent (`ipfs dht` remplacé par `ipfs routing`, profils de CID, nouvelles passerelles) : un déploiement doit suivre les notes de version de Kubo et les IPIP, et fixer explicitement ce dont dépend la reproductibilité des CID.

# 13. Travaux pratiques

## TP 1 — Premier nœud, premier CID

1. Installer Kubo, exécuter `ipfs init --profile unixfs-v1-2025` puis `ipfs daemon`.
2. Ajouter un fichier texte ; noter son CID ; le retrouver par `ipfs cat`, par la passerelle locale et par une passerelle publique.
3. Modifier un caractère, ajouter à nouveau : expliquer pourquoi le CID change et pourquoi l'ancien reste accessible.

## TP 2 — Un contenu, un identifiant

1. Calculer le CID d'un même fichier sur deux machines avec `ipfs add --only-hash` : comparer.
2. Recommencer après `ipfs config profile apply unixfs-v0-2015` sur l'une des deux : expliquer la différence.
3. Convertir un CIDv0 en CIDv1 avec `ipfs cid format` et décomposer le résultat avec `ipfs cid inspect`.

## TP 3 — Explorer un Merkle DAG

1. Ajouter un fichier de 5 Mio et un répertoire de 200 petits fichiers.
2. Avec `ipfs ls`, `ipfs dag get`, `ipfs refs -r` et `ipfs dag stat`, dessiner la structure obtenue et compter les blocs.
3. Modifier un seul fichier du répertoire : quels blocs sont nouveaux ?

## TP 4 — Réseau privé à deux nœuds

1. Avec deux conteneurs `ipfs/kubo` (ou deux machines), créer un réseau privé avec `swarm.key`.
2. Vérifier avec `ipfs swarm peers` que seuls les deux nœuds se voient.
3. Ajouter un contenu sur l'un, le lire sur l'autre ; couper le premier : que se passe-t-il ? Épingler sur le second et recommencer.

## TP 5 — Publier un site avec IPNS

1. Construire un petit site statique dans la MFS (`ipfs files`).
2. Créer une clé, publier le répertoire avec `ipfs name publish --key`.
3. Mettre à jour une page, republier, mesurer le délai de propagation avec `--ttl` court puis long.

## TP 6 — Transport hors ligne

1. Exporter un répertoire en CAR, le transférer par clé USB vers une machine sans réseau, l'importer.
2. Corrompre un octet du fichier CAR et réimporter : observer l'échec de vérification.

## TP 7 — Passerelle et récupération vérifiable

1. Récupérer un contenu public en CAR depuis une passerelle trustless avec `curl`, l'importer localement.
2. Comparer avec un téléchargement « classique » depuis `ipfs.io` : que vérifie-t-on dans chaque cas ?
3. Configurer sa propre passerelle en `Gateway.NoFetch` et la tester.

## TP 8 — Ce que le réseau voit

1. Pendant qu'un camarade télécharge un contenu que vous fournissez, observer `ipfs bitswap ledger`, `ipfs swarm peers` et `ipfs stats bw`.
2. Lister ce que votre nœud annonce à la DHT selon `Provide.Strategy`.
3. Rédiger, en dix lignes, le modèle de menace d'un nœud personnel.

## TP 9 — Service d'épinglage et DNSLink

1. Créer un compte chez un service compatible avec l'API d'épinglage, y épingler le site du TP 5 depuis Kubo.
2. Ajouter un enregistrement `_dnslink` sur un domaine de test, résoudre avec `ipfs resolve` puis par une passerelle.
3. Éteindre son nœud et vérifier que le site reste servi.

## Projet — Portail documentaire décentralisé

Publier une documentation versionnée : profil de CID explicite, MFS comme zone de travail, IPNS + DNSLink pour le nom, épinglage redondant (second nœud ou service), passerelle avec liste de refus, procédure de sauvegarde des clés, et un document expliquant ce qui est public, ce qui est persistant, et pourquoi.

# 14. Aide-mémoire

| Besoin | Commande |
|---|---|
| Initialiser un nœud public | `ipfs init --profile server` |
| Appliquer le profil de CID recommandé | `ipfs config profile apply unixfs-v1-2025` |
| Démarrer | `ipfs daemon --enable-gc` |
| Ajouter un fichier / un dossier | `ipfs add fichier`, `ipfs add -r -w dossier/` |
| Calculer un CID sans stocker | `ipfs add -Q --only-hash fichier` |
| Lire, télécharger, lister | `ipfs cat`, `ipfs get`, `ipfs ls` |
| Inspecter un CID / un DAG | `ipfs cid inspect`, `ipfs dag get`, `ipfs dag stat` |
| Convertir CIDv0 → v1 | `ipfs cid format -v 1 -b base32 <cid>` |
| Épingler, lister, retirer | `ipfs pin add --name`, `ipfs pin ls --names`, `ipfs pin rm` |
| Nettoyer | `ipfs repo gc` |
| Zone de travail mutable | `ipfs files mkdir/cp/write/ls/stat` |
| Exporter / importer un DAG | `ipfs dag export`, `ipfs dag import` |
| Qui fournit un CID ? | `ipfs routing findprovs <cid>` |
| Annoncer immédiatement | `ipfs provide once <cid>` |
| Publier un nom | `ipfs key gen`, `ipfs name publish --key=nom /ipfs/<cid>` |
| Résoudre un nom | `ipfs name resolve`, `ipfs resolve /ipns/domaine` |
| Épinglage distant | `ipfs pin remote service add`, `ipfs pin remote add` |
| Pairs, débit, dépôt | `ipfs swarm peers`, `ipfs stats bw`, `ipfs repo stat --human` |
| Mettre à jour Kubo | `ipfs update check`, `ipfs update install` |

# 15. Sources et ressources

- Documentation IPFS : <https://docs.ipfs.tech/> — concepts, guide de démarrage Kubo, référence des commandes et de la configuration
- Spécifications et IPIP : <https://specs.ipfs.tech/> — CID, UnixFS, passerelles trustless, routage délégué, IPIP-499
- Kubo : <https://github.com/ipfs/kubo> et notes de version <https://github.com/ipfs/kubo/releases>
- Distributions officielles : <https://dist.ipfs.tech/>
- Helia : <https://helia.io/> et <https://github.com/ipfs/helia>
- libp2p : <https://docs.libp2p.io/>
- IPFS Foundation : <https://ipfsfoundation.org/>
- Filecoin : <https://docs.filecoin.io/>
- IPFS Cluster : <https://ipfscluster.io/>
- Forum de la communauté : <https://discuss.ipfs.tech/>

# 16. Conclusion

IPFS déplace la question « où est cette donnée ? » vers « quelle est cette donnée ? ». Le CID rend chaque contenu vérifiable et indépendant de sa source ; le Merkle DAG rend les gros objets déduplicables et parallélisables ; libp2p, la DHT, les indexeurs et Bitswap retrouvent et échangent les blocs ; IPNS et DNSLink redonnent des noms stables ; les passerelles, de préférence *trustless*, ouvrent le tout au Web ordinaire.

Ce que le protocole ne fournit pas — persistance, confidentialité, effacement, performance garantie — doit être **décidé et exploité** : épinglage redondant, chiffrement, réseau privé, service ou Filecoin. Bien utilisé, IPFS est moins un « disque dur mondial » qu'une couche de distribution vérifiable, complémentaire de [[git]] pour le code et de HTTP pour l'accès.
