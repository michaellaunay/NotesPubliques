---
schema_version: 1
uid: "01M02EX5C2TKYX380BZWEXC6Q6"
titre: "Postfix"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - messagerie
  - postfix
resume: "Cours sur le serveur de courrier Postfix : rôles de MTA, MDA et MUA, formats de boîtes aux lettres Mbox et Maildir, protocoles IMAP et POP3, configuration et sécurisation."
niveau: intermediaire
prerequis:
  - "[[GNULinux]]"
  - "[[Les protocoles de communications]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-05-30
date_modification: 2026-08-13
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Présentation

Postfix est un serveur de courrier électronique, que l'on appelle aussi **Mail Transfer Agent (MTA)**. Sa mission principale est de recevoir des e-mails par SMTP et de les relayer vers un autre serveur de courrier ou vers un mécanisme de livraison locale lorsque le destinataire est hébergé sur notre machine.

Postfix ne fournit pas, à lui seul, l'accès aux boîtes aux lettres par IMAP ou POP3. Pour cela, nous utiliserons généralement un logiciel comme Dovecot.

# Notions de MTA, MDA, MUA

Le MTA, pour **Mail Transfer Agent**, est le logiciel qui transfère les e-mails d'un serveur à un autre en utilisant principalement SMTP. Postfix est un exemple de MTA.

Nous avons aussi le **MDA (Mail Delivery Agent)** ou **LDA (Local Delivery Agent)**, qui s'occupe de la livraison finale du message dans la boîte aux lettres de l'utilisateur. Postfix possède son propre agent de livraison locale (`local(8)`). Dovecot peut également assurer cette livraison, notamment via son LDA ou LMTP, en plus de son rôle principal de serveur IMAP/POP3.

Enfin, il y a le **MUA (Mail User Agent)**, qui est le client de messagerie utilisé par l'utilisateur final pour lire et écrire ses e-mails. Thunderbird, Evolution et Outlook sont des exemples de MUA.

# Les Boîtes aux Lettres

Une boîte aux lettres est un emplacement de stockage pour les e-mails. Elle peut être composée de dossiers et sous-dossiers, permettant une organisation logique des messages.

## Les types de BAL

Il existe principalement deux formats historiques de BAL : Mbox et Maildir.

- **Mbox** : Tous les messages sont stockés dans un fichier unique. L'accès concurrent peut causer des problèmes, et la corruption du fichier est possible.
- **Maildir** : Chaque message est stocké dans un fichier séparé, ce qui limite fortement les problèmes de verrouillage et réduit les risques de corruption globale de la boîte.
## Mbox

Le format Mbox est un des formats les plus anciens pour stocker les e-mails sur un système de fichiers. Il stocke tous les messages d'une boîte aux lettres dans un seul fichier.

### Structure de Mbox

Dans le format Mbox, tous les messages d'une boîte aux lettres sont concaténés dans un fichier unique. Chaque message commence par une ligne spéciale qui contient le caractère "From" suivi d'un espace et de l'adresse de l'expéditeur.

Exemple :

```
From user@example.com Tue Jan 1 00:00:00 2021
Subject: Exemple 1
...
From user2@example.com Tue Jan 2 00:00:00 2021
Subject: Exemple 2
...
```

### Variantes de Mbox

Il existe plusieurs variantes du format Mbox, telles que :

- **mboxo** : Le format original, sans verrouillage.
- **mboxrd** : Permet des modifications sûres du fichier.
- **mboxcl** : Supporte le verrouillage du fichier pour un accès concurrent.
- **mboxcl2** : Une variante de mboxcl avec un mécanisme de verrouillage différent.

### Avantages et Inconvénients

#### Avantages

- **Simple à implémenter** : La gestion du fichier est simple et directe.
- **Compatibilité** : Supporté par de nombreux clients et serveurs de messagerie.

#### Inconvénients

- **Verrouillage** : L'accès concurrent au fichier nécessite un mécanisme de verrouillage, ce qui peut être complexe.
- **Risque de corruption** : Si le verrouillage n'est pas correctement géré, le fichier peut être corrompu.
- **Performance** : Peut devenir lent avec de grandes boîtes aux lettres.
## Maildir

Maildir est un format pour stocker les messages électroniques où chaque message est conservé dans un fichier séparé, et chaque boîte de réception est un répertoire. Ceci offre plusieurs avantages, notamment l'absence de verrouillage lors de l'accès aux messages.

### Structure

Maildir utilise une structure de répertoire particulière. Chaque message est un fichier unique dans l'un des trois sous-répertoires :

- `new` : Pour les messages non lus.
- `cur` : Pour les messages qui ont été lus ou sont en cours de lecture.
- `tmp` : Utilisé temporairement lors de la livraison ou l'accès aux messages.

### Avantages

- **Pas de verrouillage nécessaire** : Plusieurs processus peuvent accéder simultanément aux messages.
- **Robustesse** : Moins susceptible à la corruption comparé au format Mbox.
- **Flexibilité** : Facilité de gestion, de sauvegarde, et de manipulation.

# Les protocoles de relevé du courrier IMAP et POP3

Les protocoles de relevé du courrier électronique permettent à un client de messagerie d'accéder aux messages stockés sur un serveur. Ils ne doivent pas être confondus avec SMTP, qui sert principalement au transport et à la soumission des e-mails.

## IMAP (Internet Message Access Protocol)

- **Version courante** : IMAP4rev1, avec des évolutions plus récentes du protocole.
- **Fonction** : Permet de consulter et manipuler les e-mails directement sur le serveur.
- **Gestion des dossiers** : Possibilité de créer, supprimer et manipuler des dossiers sur le serveur.
- **Recherche avancée** : Recherche de messages selon différents critères.
- **Synchronisation** : Les messages restent normalement sur le serveur, ce qui permet la synchronisation entre plusieurs appareils.
- **Ports** : 143 pour IMAP, généralement avec STARTTLS ; 993 pour IMAP avec TLS implicite.

IMAP permet donc à plusieurs clients de partager le même état de boîte aux lettres : messages lus, dossiers, suppressions, etc.

## POP3 (Post Office Protocol 3)

POP3 est un protocole plus simple destiné à récupérer les messages depuis un serveur de messagerie.

- **Fonction** : Télécharge les e-mails du serveur vers l'appareil local.
- **Simplicité** : Facile à mettre en œuvre et à utiliser.
- **Téléchargement de messages** : Les messages sont généralement stockés localement.
- **Synchronisation limitée** : POP3 ne synchronise pas l'organisation complète de la boîte comme IMAP. Les messages peuvent être supprimés du serveur après téléchargement ou conservés selon la configuration du client.
- **Ports** : 110 pour POP3, généralement avec STARTTLS ; 995 pour POP3 avec TLS implicite.

## Webmail

- **Fonction** : Permet d'accéder aux e-mails via un navigateur web.
- **Synchronisation** : Le webmail travaille généralement sur les messages stockés côté serveur, souvent via IMAP ou une API interne.
- **Port** : HTTPS utilise généralement le port 443.

### Exchange ActiveSync (EAS)

- **Fonction** : Protocole de synchronisation historiquement associé à Microsoft Exchange, permettant de synchroniser e-mails, calendriers et contacts.
- **Utilisation** : Principalement dans les environnements Exchange et certains services compatibles.

## À propos de NNTP

**NNTP (Network News Transfer Protocol)** n'est pas un protocole de relevé du courrier électronique. Il sert aux groupes de discussion Usenet et aux serveurs de news. Il utilise généralement le port 119, ou 563 avec TLS implicite.

## Comparaison entre IMAP et POP3

- **Stockage** : IMAP travaille principalement sur les messages conservés sur le serveur ; POP3 est historiquement orienté téléchargement local.
- **Synchronisation** : IMAP synchronise l'état de la boîte entre plusieurs clients ; POP3 offre une synchronisation beaucoup plus limitée.
- **Utilisation des ressources** : IMAP nécessite davantage de gestion côté serveur, mais il est généralement mieux adapté à l'utilisation moderne sur plusieurs appareils.

Chaque protocole a ses avantages et inconvénients. Aujourd'hui, IMAP est généralement le choix le plus adapté lorsqu'un utilisateur consulte sa boîte depuis plusieurs machines.

# SMTP et les ports utilisés

SMTP sert à transporter ou soumettre les messages. Nous rencontrons principalement :

- **25/TCP** : SMTP entre serveurs (MTA vers MTA). C'est le port utilisé par Postfix lorsqu'il livre directement un message à un MX distant ;
- **587/TCP** : port de soumission (`submission`) utilisé par les clients authentifiés ;
- **465/TCP** : soumission SMTP avec TLS implicite (`submissions`), aujourd'hui également utilisé par de nombreux clients ;
- **STARTTLS** : permet de commencer une connexion en clair puis de négocier TLS, notamment sur 25 et 587.

Il faut distinguer **relais SMTP** et **soumission authentifiée** : un serveur public ne doit pas accepter de relayer arbitrairement les messages d'un client non authentifié vers un domaine tiers.

# Installation

Pour installer Postfix, nous utilisons la commande suivante :

```bash
apt install postfix
```

Pendant l'installation, nous précisons le nom de l'hôte en utilisant son **Fully Qualified Domain Name (FQDN)**, par exemple `mail.masociete.com`.

Pour un serveur de messagerie public, il est important que le DNS direct et le reverse DNS soient cohérents :

```text
IP publique -> PTR -> mail.masociete.com
mail.masociete.com -> A/AAAA -> IP publique
```

Une incohérence n'empêche pas toujours SMTP de fonctionner, mais elle dégrade fortement la délivrabilité auprès de nombreux fournisseurs.

Nous configurerons également **SPF, DKIM et DMARC** pour améliorer l'authentification et la délivrabilité de nos e-mails. SPF et DMARC sont principalement des enregistrements DNS ; DKIM nécessite en plus un mécanisme de signature sur le serveur.

Si nous utilisons Postfix pour envoyer des e-mails, installons OpenDKIM et quelques outils de test :

```bash
apt install opendkim opendkim-tools mailutils
```

Si Postfix gère également la réception des e-mails et que nous souhaitons contrôler SPF sur les messages entrants :

```bash
apt install postfix-policyd-spf-python
```

Nous pouvons également installer `swaks`, très pratique pour tester SMTP :

```bash
apt install swaks
```

DMARC n'a pas besoin d'un démon local pour que **nos messages sortants** soient conformes : il faut publier l'enregistrement DNS et faire en sorte que SPF et/ou DKIM soient correctement alignés. Si nous souhaitons en revanche vérifier DMARC sur les messages entrants, un filtre spécifique comme OpenDMARC peut être ajouté.

# Configuration

Le fichier de configuration principal de Postfix est `/etc/postfix/main.cf`.

Selon que notre serveur est la destination finale des e-mails ou simplement un relais, nous définissons ou non la variable `mydestination`.

Voici un exemple simplifié de fichier de configuration pour un serveur qui est la destination finale des e-mails de `masociete.com` :

```bash
cat /etc/postfix/main.cf
```

```ini
biff = no
append_dot_mydomain = no
readme_directory = no

home_mailbox = Maildir/
mail_spool_directory = /var/spool/mail/

myhostname = mail.masociete.com
mydomain = masociete.com
myorigin = $mydomain
mydestination = $myhostname, $mydomain, localhost.localdomain, localhost
mynetworks = 127.0.0.0/8 [::1]/128
relayhost =

alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases

# Ne pas transformer le serveur en open relay.
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination

# TLS entrant : utiliser en production un vrai certificat correspondant au FQDN.
smtpd_tls_security_level = may
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.masociete.com/fullchain.pem
smtpd_tls_key_file = /etc/letsencrypt/live/mail.masociete.com/privkey.pem

# TLS sortant opportuniste.
smtp_tls_security_level = may
```

Le certificat `ssl-cert-snakeoil.pem` fourni par Debian/Ubuntu est pratique pour les tests locaux, mais nous ne devons pas l'utiliser comme certificat public de production.

Attention à `mydestination` : cette variable liste les domaines pour lesquels **ce serveur est la destination finale**. Un domaine utilisé uniquement comme adresse d'expéditeur ne doit pas être ajouté à `mydestination` pour cette seule raison.

# Configuration de SPF

SPF (**Sender Policy Framework**) permet de publier dans le DNS la liste des serveurs autorisés à envoyer des e-mails pour un domaine donné.

Aujourd'hui, SPF se publie dans un enregistrement **TXT**. L'ancien type DNS `SPF` est obsolète et ne doit plus être utilisé.

Exemple :

```text
v=spf1 a mx ip4:51.159.31.17 -all
```

Quelques mécanismes courants :

- `a` : autorise les adresses IP correspondant au champ A/AAAA du domaine ;
- `mx` : autorise les adresses IP des serveurs MX du domaine ;
- `ip4:...` : autorise explicitement une IPv4 ;
- `ip6:...` : autorise explicitement une IPv6 ;
- `include:...` : inclut la politique SPF d'un autre domaine ;
- `-all` : indique que les autres sources ne sont pas autorisées.

Évitons de publier plusieurs enregistrements SPF pour le même domaine : il doit y avoir une seule politique SPF TXT, qui regroupe toutes les sources légitimes.

Pour tester :

```bash
nslookup -type=txt ecreall.com
```

ou, plus pratique :

```bash
dig TXT ecreall.com +short
```

Ce qui peut donner :

```text
ecreall.com. TXT "v=spf1 a mx ip4:45.80.23.242 ip6:2a01:cb0c:7d:4d00:1ac0:4dff:fe09:fe47 -all"
```

Pour vérifier qu'une IP est réellement couverte, ne nous contentons pas de lire la chaîne SPF : vérifions également les A/AAAA et MX utilisés par les mécanismes `a` et `mx`.

# Configuration de OpenDKIM

OpenDKIM signe les messages sortants en ajoutant un en-tête `DKIM-Signature`. Le serveur destinataire récupère ensuite la clé publique dans le DNS afin de vérifier cette signature.

## Cas à un seul domaine

Éditons :

```bash
vim /etc/opendkim.conf
```

Ajoutons ou adaptons les variables suivantes :

```ini
Syslog              yes
Canonicalization    relaxed/simple
Domain              ecreall.com
KeyFile             /etc/dkimkeys/dkim.private
Selector            dkim
UserID               opendkim
Socket               inet:8891@localhost
```

- `Domain` indique le domaine à signer.
- `KeyFile` indique le fichier contenant la clé privée.
- `Selector` définit le **sélecteur DKIM**. Il sert notamment à construire le nom DNS `dkim._domainkey.ecreall.com`.
- `UserID` indique l'utilisateur du démon ; la clé privée doit être lisible par cet utilisateur.
- `Socket` indique comment Postfix communique avec OpenDKIM.

Générons la paire de clés :

```bash
mkdir -p /etc/dkimkeys
cd /etc/dkimkeys/
opendkim-genkey -b 2048 -s dkim -d ecreall.com
chown root:opendkim dkim.private
chmod 640 dkim.private
```

`opendkim-genkey` génère :

- `dkim.private` : clé privée utilisée par OpenDKIM ;
- `dkim.txt` : enregistrement DNS TXT contenant la clé publique.

Le sélecteur `dkim` permet de publier plusieurs clés dans le temps ou d'utiliser des clés différentes selon les domaines. Ce n'est pas une clé « située dans le fichier » : le sélecteur fait partie de l'identifiant DNS de la clé.

Vérifions la clé :

```bash
opendkim-testkey -d ecreall.com -s dkim -vvv
```

Nous cherchons notamment :

```text
opendkim-testkey: key loaded from /etc/dkimkeys/dkim.private
opendkim-testkey: checking key 'dkim._domainkey.ecreall.com'
opendkim-testkey: key OK
```

Si OpenDKIM affiche :

```text
WARNING: unsafe permissions
```

les permissions de la clé ou de son répertoire sont trop permissives. Vérifions-les :

```bash
ls -ld /etc/dkimkeys
ls -l /etc/dkimkeys/dkim.private
id opendkim
```

Une configuration typique est :

```bash
chown root:opendkim /etc/dkimkeys/dkim.private
chmod 640 /etc/dkimkeys/dkim.private
chmod 750 /etc/dkimkeys
```

La mention `key secure` / `key not secure` concerne la validation DNSSEC de la réponse DNS lors du test. Elle ne signifie pas que la clé RSA est « forte » ou « faible ». Le résultat essentiel pour la correspondance entre la clé privée et la clé publique est `key OK`.

Affichons le contenu généré pour le DNS :

```bash
cat /etc/dkimkeys/dkim.txt
```

Nous obtenons un enregistrement du type :

```text
dkim._domainkey IN TXT ( "v=DKIM1; h=sha256; k=rsa;"
                         "p=MIIBI..." )
```

Nous publions ensuite la valeur dans la zone DNS sous :

```text
dkim._domainkey.ecreall.com
```

Pour vérifier la publication depuis un résolveur public :

```bash
dig @1.1.1.1 TXT dkim._domainkey.ecreall.com +short
```

### Relier Postfix à OpenDKIM

Dans `/etc/postfix/main.cf` :

```ini
milter_protocol = 6
milter_default_action = accept
smtpd_milters = inet:localhost:8891
non_smtpd_milters = inet:localhost:8891
```

`non_smtpd_milters` est utile pour les messages injectés autrement que par `smtpd`, par exemple via la commande locale `sendmail`.

Avec `milter_default_action = accept`, Postfix continue à traiter le message si le Milter est indisponible. C'est tolérant aux pannes, mais cela peut conduire à envoyer des messages **sans signature DKIM**. Si nous voulons éviter absolument les envois non signés, nous pouvons préférer une politique de type `tempfail`, en tenant compte du fait qu'une panne du Milter bloquera alors temporairement les messages concernés.

Après modification :

```bash
systemctl restart opendkim
postfix reload
```

Vérifions que le démon est bien réellement lancé :

```bash
systemctl status opendkim --no-pager
ss -ltnp | grep 8891
ps aux | grep '[o]pendkim'
```

## Configurer plusieurs domaines ou sous-domaines dans DKIM

Pour que Postfix et OpenDKIM signent également les e-mails envoyés depuis `@access.logikascium.com` et `@alirpunkto.logikascium.com`, configurons OpenDKIM avec une `KeyTable` et une `SigningTable`.

### Génération des clés DKIM pour chaque domaine

Pour chaque domaine, il faut générer une paire de clés (privée et publique) DKIM.

**Pour `access.logikascium.com` :**

```bash
sudo mkdir -p /etc/dkimkeys/access.logikascium.com
cd /etc/dkimkeys/access.logikascium.com
sudo opendkim-genkey -s dkim -d access.logikascium.com
sudo chown opendkim:opendkim dkim.private
sudo chmod 600 dkim.private
```

**Pour `alirpunkto.logikascium.com` :**

```bash
sudo mkdir -p /etc/dkimkeys/alirpunkto.logikascium.com
cd /etc/dkimkeys/alirpunkto.logikascium.com
sudo opendkim-genkey -s dkim -d alirpunkto.logikascium.com
sudo chown opendkim:opendkim dkim.private
sudo chmod 600 dkim.private
```

### Publication des clés publiques dans le DNS

Pour chaque domaine, ajouter un enregistrement TXT dans le DNS avec la clé publique.

**Étapes :**

1. **Récupérer la clé publique générée :**

   ```bash
   cat /etc/dkimkeys/access.logikascium.com/dkim.txt
   ```

   Faisons de même pour `alirpunkto.logikascium.com`.

2. **Ajouter l'enregistrement DNS :**

   - **Nom de l'enregistrement :** `dkim._domainkey.access.logikascium.com`
   - **Type :** `TXT`
   - **Valeur :** Contenu du fichier `dkim.txt` correspondant.

### Configuration d'OpenDKIM pour gérer plusieurs domaines

Modifions le fichier `/etc/opendkim.conf` pour utiliser `KeyTable` et `SigningTable`.

**Éditons `/etc/opendkim.conf` et apportons les modifications suivantes :**

1. **Commenter ou supprimer les lignes existantes pour `Domain`, `Selector` et `KeyFile` :**

```ini
   # Domain    publicpolicies.logikascium.com
   # Selector  dkim
   # KeyFile   /etc/dkimkeys/dkim.private
```

1. **Ajouter les lignes suivantes pour utiliser `KeyTable` et `SigningTable` :**

```ini
KeyTable           file:/etc/opendkim/KeyTable
SigningTable       refile:/etc/opendkim/SigningTable
```

### Création des fichiers KeyTable et SigningTable

**Créer le fichier `/etc/opendkim/KeyTable` :**

```bash
sudo vim /etc/opendkim/KeyTable
```

**Contenu du fichier :**

```ini
dkim._domainkey.publicpolicies.logikascium.com publicpolicies.logikascium.com:dkim:/etc/dkimkeys/publicpolicies.logikascium.com/dkim.private
dkim._domainkey.access.logikascium.com access.logikascium.com:dkim:/etc/dkimkeys/access.logikascium.com/dkim.private
dkim._domainkey.alirpunkto.logikascium.com alirpunkto.logikascium.com:dkim:/etc/dkimkeys/alirpunkto.logikascium.com/dkim.private
```

**Créer le fichier `/etc/opendkim/SigningTable` :**

```bash
sudo vim /etc/opendkim/SigningTable
```

**Contenu du fichier :**

```ini
*@publicpolicies.logikascium.com dkim._domainkey.publicpolicies.logikascium.com
*@access.logikascium.com dkim._domainkey.access.logikascium.com
*@alirpunkto.logikascium.com dkim._domainkey.alirpunkto.logikascium.com
```

**Optionnel :** Pour demander à OpenDKIM de refuser les clés privées jugées insuffisamment protégées, nous pouvons ajouter :

```ini
RequireSafeKeys     yes
```

### Vérification des permissions

Assurons-nous que les fichiers de configuration sont lisibles par l'utilisateur `opendkim` :

```bash
sudo chown opendkim:opendkim /etc/opendkim/KeyTable /etc/opendkim/SigningTable
sudo chmod 640 /etc/opendkim/KeyTable /etc/opendkim/SigningTable
```

### Redémarrage d'OpenDKIM et de Postfix

Après avoir effectué les modifications, redémarrons les services pour appliquer la nouvelle configuration :

```bash
sudo systemctl restart opendkim
sudo systemctl restart postfix
```

### Vérification d'OpenDKIM

Vérifions que OpenDKIM est actif et écoute sur le port configuré (par exemple, 8891) :

```bash
sudo systemctl status opendkim
sudo ss -ltnp | grep -E '8891|opendkim'
```

### Test de l'envoi d'e-mails depuis les nouveaux domaines

Envoyons des e-mails depuis les adresses `@access.logikascium.com` et `@alirpunkto.logikascium.com` vers une adresse externe (par exemple, Gmail) pour vérifier que les e-mails sont correctement signés.

**Vérifions les en-têtes de l'e-mail reçu :**

- Rechercher le champ `DKIM-Signature`.
- Assurons-nous que le domaine (`d=`) correspond à l'adresse d'expéditeur.
- Vérifions que le sélecteur (`s=`) est correct.

### Vérification des enregistrements DNS DKIM

Assurons-nous que les enregistrements DNS pour les clés publiques sont correctement publiés et accessibles.

**Utilisons la commande `dig` pour vérifier :**

```bash
dig TXT dkim._domainkey.access.logikascium.com +short
dig TXT dkim._domainkey.alirpunkto.logikascium.com +short
```

### Configuration de SPF et DMARC

Pour chaque domaine réellement utilisé dans le champ `From:`, vérifions également SPF et DMARC.

Exemple SPF :

```text
v=spf1 mx ip4:VOTRE_IP -all
```

Exemple DMARC de démarrage :

```text
v=DMARC1; p=none; rua=mailto:dmarc@votredomaine.com
```

Une fois les rapports analysés et les sources légitimes correctement authentifiées, la politique peut être renforcée progressivement vers `quarantine` puis `reject`.

### Surveillance des logs pour détecter d'éventuelles erreurs

Consulter régulièrement les logs de Postfix et OpenDKIM pour s'assurer qu'il n'y a pas d'erreurs.

**Logs de Postfix :**

```bash
sudo tail -f /var/log/mail.log
```

**Logs d'OpenDKIM :**

```bash
sudo grep -i opendkim /var/log/mail.log
sudo journalctl -u opendkim -n 100 --no-pager
```

### Vérifier la configuration de Postfix pour les multiples domaines

Assurons-nous que Postfix est configuré pour gérer l'envoi d'e-mails depuis ces domaines. Vérifions les paramètres suivants dans `/etc/postfix/main.cf` :

- **mydestination** : Doit inclure uniquement les domaines pour lesquels ce serveur est une destination finale. Le simple fait de signer ou d'envoyer depuis un domaine ne justifie pas de l'ajouter ici.

```ini
mydestination = $myhostname, logikascium.com, localhost.localdomain, localhost
```

- **relay_domains** : Si nous relayons des e-mails pour ces domaines.


### Points supplémentaires

- **Synchronisation de l'heure** : Assurons-nous que notre serveur a l'heure correcte. Utilisons `systemd-timesyncd`, `chrony` ou un autre service NTP si nécessaire.
- **Sécurité** : Assurons-nous que nos clés privées sont sécurisées et que seules les personnes autorisées y ont accès.
- **Documentation** : Conservons une documentation de notre configuration pour faciliter la maintenance future.


## Visualisation d'une clé DKIM

Pour visualiser les informations d'une clé DKIM à partir du fichier de la clé privée, nous pouvons extraire la clé publique correspondante. La clé publique est publiée dans les enregistrements DNS pour permettre aux serveurs de réception de vérifier les signatures DKIM des e-mails.

### Vérifier que OpenSSL est installé

Vérifions qu'OpenSSL est installé sur le système :

```bash
openssl version
```

Si OpenSSL n'est pas installé, alors l'installer avec :

```bash
sudo apt-get install openssl
```

---

### Extraire la clé publique de la clé privée

Supposons que notre fichier de clé privée s'appelle `dkim.private` et est situé dans `/etc/dkimkeys/`.

Exécutons la commande suivante pour extraire la clé publique :

```bash
openssl rsa -in /etc/dkimkeys/dkim.private -pubout -out /tmp/dkim_public.pem
```

- **Explication :**
  - `-in /etc/dkimkeys/dkim.private` : spécifie le fichier de clé privée en entrée.
  - `-pubout` : indique à OpenSSL d'extraire la clé publique.
  - `-out /tmp/dkim_public.pem` : spécifie le fichier de sortie pour la clé publique.

### Afficher la clé publique

Pour voir la clé publique extraite :

```bash
cat /tmp/dkim_public.pem
```

Nous obtiendrons quelque chose comme :

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAr9ZL2Ck...
...suite de la clé...
-----END PUBLIC KEY-----
```

### Préparer la clé publique pour l'enregistrement DNS DKIM

Les enregistrements DNS DKIM nécessitent que la clé publique soit sur une seule ligne, sans les en-têtes et pieds de page. Pour cela utilisons la commande suivante :

```bash
openssl rsa -in /etc/dkimkeys/dkim.private -pubout -outform PEM | grep -v '^-.*KEY-----' | tr -d '\n'
```

- **Explication :**
  - `grep -v '^-.*KEY-----'` : supprime les lignes contenant les en-têtes et pieds de page.
  - `tr -d '\n'` : supprime les sauts de ligne pour obtenir une clé sur une seule ligne.

### Construire l'enregistrement DNS DKIM

Avec la clé publique préparée, nous allons ajouter un enregistrement TXT DKIM :

```
v=DKIM1; k=rsa; p=VOTRE_CLÉ_PUBLIQUE
```

- Remplacez `VOTRE_CLÉ_PUBLIQUE` par la clé publique obtenue à l'étape précédente.

### Vérifier les détails de la clé

Pour voir plus de détails sur la clé (par exemple, le modulus et l'exposant public), exécutons :

```bash
openssl rsa -in /etc/dkimkeys/dkim.private -text -noout
```


#### Vérification de l'enregistrement DKIM dans le DNS

Après avoir ajouté l'enregistrement DNS :

```bash
dig TXT dkim._domainkey.logikascium.com +short
```

#### Vérification finale

Après avoir configuré l'enregistrement DNS et redémarré les services si nécessaire, envoyons un e-mail de test à une adresse externe (par exemple, Gmail) et vérifions les en-têtes pour nous assurer que le champ `DKIM-Signature` est présent (sur gmail afficher l'original du mail).

Nous pouvons aussi utiliser des outils en ligne pour tester la configuration DKIM, comme :

- **Mail Tester** : [www.mail-tester.com](https://www.mail-tester.com/)
- **DKIM Core Validator** : [dkimcore.org/tools/keycheck.html](https://dkimcore.org/tools/keycheck.html)

### DMARC

DMARC (**Domain-based Message Authentication, Reporting and Conformance**) permet au propriétaire d'un domaine d'indiquer aux serveurs destinataires ce qu'ils doivent faire lorsque l'authentification échoue. Il permet également de recevoir des rapports agrégés.

L'enregistrement est publié en TXT sous `_dmarc.domaine`.

Exemple :

```text
v=DMARC1; p=quarantine; pct=100; rua=mailto:michaellaunay+dmarc@ecreall.com; sp=quarantine; aspf=s; adkim=s;
```

- `v=DMARC1` : version du protocole ;
- `p=none|quarantine|reject` : politique appliquée au domaine principal ;
- `pct=100` : pourcentage des messages auxquels appliquer la politique ;
- `rua=mailto:...` : adresse de réception des rapports agrégés ;
- `sp=...` : politique appliquée aux sous-domaines ;
- `aspf=r|s` : alignement SPF relaxed ou strict ;
- `adkim=r|s` : alignement DKIM relaxed ou strict.

Pour que DMARC passe, il faut qu'au moins **SPF ou DKIM réussisse avec un domaine aligné sur le domaine visible dans le champ `From:`**.

Pour tester la publication :

```bash
dig TXT _dmarc.ecreall.com +short
```

# Audit et diagnostic d'un serveur Postfix

Lorsqu'un serveur fonctionne mais qu'un fournisseur refuse nos messages, il faut distinguer :

- une erreur de configuration Postfix ;
- une erreur DNS ;
- un échec SPF/DKIM/DMARC ;
- un problème TLS ;
- un compte compromis ou un open relay ;
- une mauvaise réputation de l'IP ou de sa plage réseau.

Voici une procédure d'audit reproductible.

## Vérifier la configuration Postfix active

`postconf -n` n'affiche que les paramètres qui diffèrent des valeurs par défaut :

```bash
postconf -n
```

Pour contrôler rapidement les paramètres critiques :

```bash
postconf \
    myhostname \
    mydomain \
    myorigin \
    mydestination \
    mynetworks \
    smtpd_relay_restrictions \
    smtpd_recipient_restrictions \
    smtp_helo_name
```

Vérifions également la cohérence syntaxique de Postfix :

```bash
postfix check
```

Pour éviter un open relay, nous cherchons typiquement une politique de ce genre :

```ini
mynetworks = 127.0.0.0/8 [::1]/128
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination
```

Depuis une **machine extérieure** au serveur, nous pouvons tester le refus de relais avec `swaks` :

```bash
swaks \
    --server mail.masociete.com \
    --from externe@example.net \
    --to victime@example.org \
    --quit-after RCPT
```

Si `example.org` n'est pas un domaine local et que nous ne sommes pas authentifiés, Postfix doit refuser le relais, par exemple avec `Relay access denied`.

## Vérifier les services et les ports

```bash
systemctl status postfix --no-pager
ss -ltnp | grep -E ':(25|465|587)\b'
```

Pour OpenDKIM :

```bash
systemctl status opendkim --no-pager
ss -ltnp | grep 8891
ps aux | grep '[o]pendkim'
```

Attention : `systemctl status` peut parfois afficher `active (exited)` pour un ancien script SysV sans qu'un démon soit réellement actif. Vérifions donc toujours le processus et le socket.

Nous pouvons également vérifier l'état réel des paquets Debian/Ubuntu :

```bash
dpkg -l opendkim opendkim-tools
command -v opendkim
```

Dans la sortie de `dpkg -l` :

```text
ii = paquet installé
rc = paquet supprimé, fichiers de configuration conservés
```

C'est un cas important : les fichiers `/etc/opendkim.conf` peuvent être présents alors que le démon `opendkim` n'est plus installé.

## Vérifier la file d'attente

```bash
postqueue -p
```

ou :

```bash
mailq
```

Une file vide donne :

```text
Mail queue is empty
```

Une accumulation de messages `deferred` indique qu'il faut analyser les réponses des serveurs distants et les logs.

## Auditer les envois dans les logs

Afficher les derniers messages effectivement envoyés :

```bash
grep 'status=sent' /var/log/mail.log | tail -100
```

Afficher les erreurs et reports :

```bash
grep -E 'status=(bounced|deferred)' /var/log/mail.log | tail -100
```

Afficher les authentifications SMTP SASL :

```bash
grep 'sasl_username=' /var/log/mail.log | tail -100
```

Une absence de résultat ne prouve pas qu'aucun compte n'a jamais été compromis, mais un volume inattendu ou des utilisateurs inconnus doivent nous alerter.

Compter les envois SMTP sortants par heure :

```bash
grep 'postfix/smtp.*status=sent' /var/log/mail.log \
    | awk '{print substr($1,1,13)}' \
    | sort \
    | uniq -c
```

Rechercher un destinataire ou un domaine particulier :

```bash
grep -i 'hotmail\|outlook' /var/log/mail.log | tail -100
```

Suivre les logs en direct pendant un test :

```bash
tail -f /var/log/mail.log
```

## Vérifier le DNS direct et le reverse DNS

Supposons :

```text
FQDN = delfeno.logikascium.com
IP   = 62.210.205.68
```

Vérifions le PTR :

```bash
dig -x 62.210.205.68 +short
```

Puis le DNS direct depuis des résolveurs publics :

```bash
dig @1.1.1.1 delfeno.logikascium.com A +short
dig @8.8.8.8 delfeno.logikascium.com A +short
```

Nous recherchons une cohérence de type :

```text
62.210.205.68 -> PTR -> delfeno.logikascium.com
delfeno.logikascium.com -> A -> 62.210.205.68
```

### Attention au résolveur local

Sur une machine Linux, la résolution locale peut être influencée par `/etc/hosts`, `systemd-resolved` ou un DNS local.

Par exemple :

```bash
dig delfeno.logikascium.com A +short
```

peut localement renvoyer `127.0.1.1`, alors que les résolveurs publics voient bien l'IP publique.

Dans le doute :

```bash
getent hosts delfeno.logikascium.com
grep -n delfeno /etc/hosts
resolvectl status

dig @1.1.1.1 delfeno.logikascium.com A +short
```

Pour la délivrabilité, c'est avant tout la vue DNS publique qui compte pour le serveur distant.

## Vérifier les MX

```bash
dig @1.1.1.1 logikascium.com MX +short
```

Puis vérifions l'adresse de chaque MX :

```bash
dig @1.1.1.1 delfeno.logikascium.com A +short
```

Il est inutile de publier plusieurs MX de priorités différentes s'ils pointent exactement vers la même machine : cela ne fournit aucune redondance réelle.

## Vérifier SPF

```bash
dig @1.1.1.1 TXT logikascium.com +short
```

Exemple :

```text
"v=spf1 a mx -all"
```

Dans ce cas, il faut aussi vérifier que les mécanismes `a` et `mx` aboutissent réellement à l'IP d'envoi.

## Vérifier DMARC

```bash
dig @1.1.1.1 TXT _dmarc.logikascium.com +short
```

Exemple :

```text
v=DMARC1; p=reject; rua=mailto:dmarc@logikascium.com; aspf=s; adkim=s;
```

## Auditer DKIM et OpenDKIM

Cherchons d'abord le sélecteur et la clé configurés :

```bash
grep -RniE 'Selector|Domain|KeyFile|SigningTable|KeyTable' \
    /etc/opendkim* 2>/dev/null
```

Puis vérifions l'enregistrement DNS correspondant. Si le sélecteur est `dkim` :

```bash
dig @1.1.1.1 TXT dkim._domainkey.logikascium.com +short
```

Vérifions que la clé privée correspond à la clé publique :

```bash
opendkim-testkey -d logikascium.com -s dkim -vvv
```

Résultat recherché :

```text
key OK
```

Contrôlons ensuite les Milters configurés dans Postfix :

```bash
postconf -n | grep -Ei 'milter|dkim'
```

Exemple :

```ini
milter_default_action = accept
milter_protocol = 6
non_smtpd_milters = inet:localhost:8891
smtpd_milters = inet:localhost:8891
```

Enfin, vérifions le daemon et ses logs :

```bash
systemctl status opendkim --no-pager
ss -ltnp | grep 8891
journalctl -u opendkim -n 100 --no-pager
grep -i opendkim /var/log/mail.log | tail -100
```

Lors d'un envoi signé, nous pouvons voir :

```text
DKIM-Signature field added (s=dkim, d=logikascium.com)
```

## Vérifier le certificat SMTP entrant

Pour tester STARTTLS sur le port 25 :

```bash
openssl s_client \
    -starttls smtp \
    -connect delfeno.logikascium.com:25 \
    -servername delfeno.logikascium.com
```

Vérifions notamment le nom du certificat, sa chaîne de confiance et sa date d'expiration.

Dans Postfix :

```bash
postconf smtpd_tls_cert_file smtpd_tls_key_file smtpd_tls_security_level
```

Un certificat `ssl-cert-snakeoil.pem` est un certificat local de test et doit être remplacé sur un serveur public.

## Vérification finale avec un vrai destinataire

Un des meilleurs tests consiste à envoyer un message vers une boîte externe, par exemple Gmail, puis à utiliser **Afficher l'original**.

Nous recherchons :

```text
SPF:   PASS
DKIM:  PASS
DMARC: PASS
```

Et dans les en-têtes :

```text
DKIM-Signature: ... d=logikascium.com; s=dkim; ...
```

Cette méthode valide toute la chaîne réelle : DNS, IP d'envoi, enveloppe SMTP, signature DKIM et alignement DMARC.

## Comprendre à quelle étape SMTP se produit le rejet

Une transaction SMTP simplifiée est :

```text
Client                     Serveur distant
  |                              |
  |------ EHLO ----------------->|
  |<----- 250 -------------------|
  |------ MAIL FROM ------------>|
  |<----- 250 -------------------|
  |------ RCPT TO -------------->|
  |<----- 250 -------------------|
  |------ DATA ----------------->|
  |      en-têtes + corps         |
  |      DKIM-Signature           |
  |------ . -------------------->|
```

Si le serveur distant refuse le message **en réponse à `MAIL FROM`**, il n'a pas encore reçu le contenu du message ni l'en-tête `DKIM-Signature`.

C'est essentiel pour le diagnostic : réparer DKIM est nécessaire pour une bonne délivrabilité, mais cela ne peut pas immédiatement corriger un blocage IP appliqué avant `DATA`.

## Cas pratique : blocage Microsoft S3140

Exemple de réponse :

```text
550 5.7.1 Unfortunately, messages from [62.210.205.68] weren't sent.
Please contact your Internet service provider since part of their network
is on our block list (S3140).
```

Si cette réponse arrive au `MAIL FROM`, le filtrage porte d'abord sur l'IP ou la réputation réseau. Le message n'est pas encore transmis à Microsoft, donc la signature DKIM n'a pas encore pu être évaluée.

Dans ce cas :

1. vérifions d'abord que le serveur n'est pas compromis et qu'il n'est pas open relay ;
2. vérifions PTR/A, SPF, DKIM et DMARC ;
3. vérifions les en-têtes d'un message accepté par un autre grand fournisseur ;
4. inscrivons l'IP dans **Microsoft SNDS** afin de consulter les données de réputation disponibles ;
5. suivons ensuite la procédure de support/déblocage indiquée par Microsoft et, si le message mentionne le réseau du fournisseur, contactons également l'hébergeur.

Microsoft SNDS :

```text
https://substrate.office.com/ip-domain-management-snds/snds
```

SNDS donne des informations de réputation sur les IP que nous administrons et permet également d'accéder au programme de remontée des signalements indésirables (JMRP). Pour un problème urgent de délivrabilité, Microsoft demande en revanche de contacter **Sender Support** : SNDS est un outil de diagnostic et de suivi, pas un bouton de déblocage immédiat.

Lors de l'autorisation d'une IP, Microsoft peut proposer des adresses administratives comme `postmaster@domaine`, `abuse@domaine` ou l'adresse abuse de l'opérateur réseau. Choisissons une adresse que nous contrôlons effectivement pour valider la demande.

Le fait qu'un message soit accepté par Microsoft 365 mais refusé par Outlook.com/Hotmail n'est pas contradictoire : les politiques et systèmes de filtrage rencontrés peuvent différer selon le service destinataire.

# Adresses techniques obligatoires ou recommandées

Pour un domaine de messagerie public, prévoyons au minimum des alias administratifs comme `postmaster` et `abuse` :

```bash
vim /etc/aliases
```

Par exemple :

```text
postmaster: michaellaunay
abuse:      michaellaunay
```

Puis reconstruisons la base d'alias :

```bash
newaliases
```

Ces adresses sont également utiles lors des procédures de validation auprès de services de réputation comme Microsoft SNDS.

# Références utiles pour l'audit

- Postfix Milter : https://www.postfix.org/MILTER_README.html
- Paramètres Postfix : https://www.postfix.org/postconf.5.html
- Microsoft SNDS : https://substrate.office.com/ip-domain-management-snds/snds
- OpenDKIM testkey : https://manpages.ubuntu.com/manpages/noble/man8/opendkim-testkey.8.html


# Installer Dovecot

Dovecot est principalement un serveur IMAP et POP3. Il peut également assurer la livraison locale via LDA/LMTP.
Nous avons configuré Postfix pour utiliser le format Maildir et allons voir comment configurer Dovecot pour ce format.
## Installation de Dovecot

Installons Dovecot avec la commande :

```bash
sudo apt-get install dovecot-imapd
```

## Configuration de Dovecot avec Maildir

Selon la version et l'organisation de la configuration Dovecot, ce paramètre se trouve souvent dans `/etc/dovecot/conf.d/10-mail.conf`. Assurons-nous que la configuration contient :

```txt
mail_location = maildir:~/Maildir
```

Cela indique à Dovecot d'utiliser le format Maildir pour le stockage des e-mails.

# Tester la configuration

Nous pouvons utiliser un service de test externe comme [appmaildev.com](https://appmaildev.com), puis envoyer un message vers l'adresse temporaire fournie :
```bash
echo test | mail -s "Test postfix" -aFROM:michaellaunay@ecreall.com test-1fc42c52@appmaildev.com
```


# Créer un serveur postfix de test

Pour développer, il peut être intéressant d'installer un Postfix de test et de rediriger tout le trafic vers une boîte locale.
```bash
sudo apt install postfix mailutils
# Choisir l'installation « Site Internet » et définir un nom de domaine
```

Éditer /etc/postfix/main.cf

```bash
vim /etc/postfix/main.cf
```

Ajouter en fin de fichier :

```
recipient_canonical_maps = regexp:/etc/postfix/recipient_canonical
mydestination = ecreall.com
relayhost =
home_mailbox = Maildir/
mailbox_command =
```

Créer /etc/postfix/recipient_canonical

```bash
vim /etc/postfix/recipient_canonical
```

Ajouter :

```
/.*/    michaellaunay@ecreall.com
```

Le type de table `regexp:` est lu directement par Postfix : il n'est pas nécessaire de lancer `postmap` pour ce fichier. Redémarrons ou rechargeons Postfix :

```bash
postfix reload
```

Tester

```bash
echo "Un super test d'envoi de mail" | mail -s "Test envoi" michaellaunay@gmail.com
ls -lh /home/michaellaunay/Maildir/new/
```

```
total 40K
-rw------- 1 michaellaunay michaellaunay 479 avril 24 20:08 1682359681.V803I12e4dffM19518.leojag
```

```bash
cat /home/michaellaunay/Maildir/new/1682359681.V803I12e4dffM19518.leojag
```

```
Return-Path: <michaellaunay@leojag>
X-Original-To: michaellaunay@gmail.com
Delivered-To: michaellaunay@ecreall.com
Received: by leojag.home (Postfix, from userid 1000)
	id 032DFC0BE9; Mon, 24 Apr 2023 20:08:01 +0200 (CEST)
Subject: Test envoi
To: <michaellaunay@ecreall.com>
User-Agent: mail (GNU Mailutils 3.14)
Date: Mon, 24 Apr 2023 20:08:00 +0200
Message-Id: <20230424180801.032DFC0BE9@leojag.home>
From: Michaël Launay <michaellaunay@leojag>

Un super test d'envoi de mail
```

De la même façon nous pouvons jouer avec les paramètres :

```bash
vim /etc/postfix/main.cf
```

Modifications des mails :
```
smtpd_recipient_restrictions = reject #Rejette tous les destinataires reçus par smtpd
sender_canonical_maps = regexp:/etc/postfix/sender_canonical #modifie l'émetteur
sender_bcc_maps = regexp:/etc/postfix/sender_bcc #Ajoute une copie cachée (BCC) selon l'expéditeur
always_bcc = destinataire_origine@example.com #Ajoute toujours une copie cachée (BCC)
```
