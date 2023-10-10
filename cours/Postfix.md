# Présentation

Postfix est un serveur de courrier électronique, que l'on appelle aussi Mail Transfer Agent (MTA). Sa mission est de recevoir les emails, de les stocker dans la boîte de réception du destinataire ou de les relayer vers un autre serveur de courrier si la boîte du destinataire n'est pas sur notre machine.

# Notions de MTA, MDA, MUA

Le MTA, pour "Mail Transfer Agent", est le logiciel qui transfère les emails d'un serveur à un autre. Postfix est un exemple de MTA.

Nous avons aussi le MDA (Mail Delivery Agent) qui s'occupe de la livraison des emails au destinataire final. Dovecot est un exemple couramment utilisé de MDA.

Enfin, il y a le MUA (Mail User Agent), qui est le client de messagerie que l'utilisateur final utilise pour lire et écrire des emails. Thunderbird et Outlook sont des exemples de MUA.

# Les Boîtes aux Lettres

Une boîte aux lettres est un emplacement de stockage pour les e-mails. Elle peut être composée de dossiers et sous-dossiers, permettant une organisation logique des messages.

## Les types de BAL

Il existe principalement deux types de BAL: Mbox et Maildir.

- **Mbox**: Tous les messages sont stockés dans un fichier unique. L'accès concurrent peut causer des problèmes, et la corruption du fichier est possible.
- **Maildir**: Chaque message est stocké dans un fichier séparé, éliminant ainsi les problèmes de verrouillage et de corruption.
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

Il existe plusieurs variantes du format Mbox, telles que:

- **mboxo**: Le format original, sans verrouillage.
- **mboxrd**: Permet des modifications sûres du fichier.
- **mboxcl**: Supporte le verrouillage du fichier pour un accès concurrent.
- **mboxcl2**: Une variante de mboxcl avec un mécanisme de verrouillage différent.

### Avantages et Inconvénients

#### Avantages

- **Simple à Implémenter**: La gestion du fichier est simple et directe.
- **Compatibilité**: Supporté par de nombreux clients et serveurs de messagerie.

#### Inconvénients

- **Verrouillage**: L'accès concurrent au fichier nécessite un mécanisme de verrouillage, ce qui peut être complexe.
- **Risque de Corruption**: Si le verrouillage n'est pas correctement géré, le fichier peut être corrompu.
- **Performance**: Peut devenir lent avec de grandes boîtes aux lettres.
- 
## Maildir

Maildir est un format pour stocker les messages électroniques où chaque message est conservé dans un fichier séparé, et chaque boîte de réception est un répertoire. Ceci offre plusieurs avantages, notamment l'absence de verrouillage lors de l'accès aux messages.

### Structure

Maildir utilise une structure de répertoire particulière. Chaque message est un fichier unique dans l'un des trois sous-répertoires:

- `new`: Pour les messages non lus.
- `cur`: Pour les messages qui ont été lus ou sont en cours de lecture.
- `tmp`: Utilisé temporairement lors de la livraison ou l'accès aux messages.

### Avantages

- **Pas de verrouillage nécessaire**: Plusieurs processus peuvent accéder simultanément aux messages.
- **Robustesse**: Moins susceptible à la corruption comparé au format Mbox.
- **Flexibilité**: Facilité de gestion, de sauvegarde, et de manipulation.

# Les protocoles de relevé du courrier IMAP et POP3

Les protocoles de relevé du courrier électronique sont des standards utilisés pour accéder, récupérer et gérer les e-mails depuis un serveur de messagerie vers des clients de messagerie. Voici les principaux protocoles de relevé du courrier:

## IMAP (Internet Message Access Protocol)

- **Version**: IMAP4 est la version la plus récente.
- **Fonction**: Permet de lire les e-mails directement sur le serveur.
- **Gestion des dossiers**: Possibilité de créer, supprimer et manipuler des dossiers sur le serveur.
- **Recherche avancée**: Recherche de messages selon divers critères.
- **Synchronisation**: Les messages restent sur le serveur, permettant la synchronisation entre différents appareils. Ainsi, les changements effectués par un client sont répercutés sur tous les autres clients.
- **Port**: Utilise généralement le port 143, ou le port 993 pour IMAP sur SSL.

IMAP permet aux clients de messagerie de voir les messages stockés sur un serveur de messagerie. Contrairement à POP3, IMAP permet de multiples clients et des opérations simultanées sur le serveur.

## POP3 (Post Office Protocol 3)

POP3 est un protocole plus simple pour récupérer les messages depuis un serveur de messagerie. Il télécharge les messages sur le client et les supprime du serveur.

- **Fonction**: Télécharge les e-mails du serveur vers l'appareil local, où ils sont lus.
- **Simplicité**: Facile à mettre en œuvre et à utiliser.
- **Téléchargement de messages**: Les messages sont téléchargés et stockés localement.
- **Pas de synchronisation**: Les actions effectuées sur un client ne sont pas répercutées sur les autres clients. Une fois téléchargés, les e-mails sont supprimés du serveur (sauf si configuré autrement).
- **Port**: Utilise généralement le port 110, ou le port 995 pour POP3 sur SSL.

## Webmail

- **Fonction**: Permet d'accéder aux e-mails via un navigateur web.
- **Synchronisation**: Comme IMAP, les messages sont généralement stockés et gérés sur le serveur.
- **Port**: Dépend du protocole web utilisé (HTTP ou HTTPS), généralement 80 ou 443.

### Exchange ActiveSync (EAS)

- **Fonction**: Protocole propriétaire de Microsoft permettant la synchronisation des e-mails, des calendriers et des contacts.
- **Synchronisation**: Fonctionne de manière similaire à IMAP pour la synchronisation des e-mails.
- **Utilisé principalement avec**: Microsoft Exchange, Office 365.

## **NNTP (Network News Transfer Protocol)**

- **Fonction**: Bien que principalement utilisé pour les forums de discussion en ligne, NNTP peut également être utilisé pour récupérer des e-mails.
- **Port**: Utilise généralement le port 119, ou le port 563 pour NNTP sur SSL.

## Comparaison entre IMAP et POP3

- **Stockage**: IMAP stocke les messages sur le serveur; POP3 les télécharge sur le client.
- **Synchronisation**: IMAP synchronise les messages entre clients; POP3 ne le fait pas.
- **Utilisation des ressources**: IMAP peut être plus gourmand en ressources en raison de la synchronisation et de la gestion du serveur.

Chaque protocole a ses propres avantages et inconvénients, et le choix dépendra des besoins spécifiques tels que la nécessité de synchronisation entre différents appareils, la conservation des e-mails sur le serveur, ou des préférences en matière de sécurité et de confidentialité.

# Installation

Pour installer Postfix, nous utilisons la commande suivante :

```bash
apt install postfix
```

Pendant l'installation, nous précisons le nom de l'hôte, en utilisant le Fully Qualified Domain Name (FQDN). Il est important que le reverse DNS ait été correctement configuré, sinon nos emails risquent d'être considérés comme du spam dès la connexion de notre serveur.

Nous installerons également SPF, DKIM, et DMARC pour améliorer la sécurité et la délivrabilité de nos emails.

Si nous utilisons Postfix pour envoyer des emails, nous installons les outils nécessaires pour la signature DKIM et pour tester l'envoi d'emails :

```bash
apt install opendkim opendkim-tools
apt install mailutils
```

Si Postfix gère également la réception des emails, nous installons un outil supplémentaire pour vérifier les enregistrements SPF des emails entrants :

```bash
apt install postfix-policyd-spf-python
```

Nous modifions ensuite le fichier /etc/postfix/master.cf pour lancer le démon d'analyse des emails reçus.

# Configuration

Le fichier de configuration principal de Postfix est /etc/postfix/main.cf.

Selon que notre serveur est la destination finale des emails ou simplement un relais, nous définissons ou non la variable "mydestination".

Voici un exemple de fichier de configuration pour un serveur qui est la destination finale des emails :
```bash
cat /etc/postfix/main.cf
```

```
biff = no
append_dot_mydomain = no
readme_directory = no
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
home_mailbox = Maildir/
mail_spool_directory = /var/spool/mail/
myhostname = monserveur.masociete.com
mydomain = masociete.com
myorigin = $mydomain
header_checks = regexp:/etc/postfix/header_checks
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
relayhost =
mydestination = masociete.com
```

# Configuration de spf

Ajouter une entrée spf à vos entrées DNS sur notre Registrar.

Par exemple, le registrar de **ecreall.com** est OVH et il faut ajouter une entrée SPF et la remplir comme suit :
```
  Sous-domaine []**.ecreall.com** #Préciser le sous-domaine, ici il n'y en a pas, donc nous laissons vide
  TTL [Par défaut] #Mais pour les tests [personnalisés] nous pouvons alors mettre une valeur faible en secondes comme [60]
  Autoriser l'IP de **ecreall.com** à envoyer des emails ? [v]oui # nous autoriserons les adresses ip que nous souhaitons.
  Autoriser les serveurs MX à envoyer des emails [v]oui #si MX est notre serveur
  Autoriser tous les serveurs dont le nom se termine par **ecreall.com** [v]Non # permet de gérer les sous-domaines
  D'autres serveurs ? # Mettre les autres adresses ou noms autorisés à envoyer.
```
Sous Gandi il ne faut surtout pas utiliser le champ spf qui est documenté comme obsolète, à la place il faut utiliser une entrée TXT

Il faut alors mettre "v=spf1 a mx ip4:51.159.31.17 -all" dans le champ.

La valeur des champs spf est expliquée par Google ici https://support.google.com/a/answer/33786

Pour tester :

```bash
nslookup -type=txt ecreall.com
```
    
Ce qui donne :

```
ecreall.com	text = "v=spf1 a mx ip4:45.80.23.242 ip6:2a01:cb0c:7d:4d00:1ac0:4dff:fe09:fe47 -all"
```

# Configuration de opendkim

```bash
vim /etc/opendkim.conf 
```

Ajoutez ou mettez les variables à :

```
  Domain                ecreall.com
  KeyFile               /etc/dkimkeys/dkim.private
  Selector              dkim
  UserID                opendkim
  Socket inet:8892@localhost
```

Le champ \"Domain\" indique quels vont être les mails signés avec la clé contenue dans le fichier \"Keyfile\" Le champ \"Selector\" indique quelle clé dans le fichier utiliser pour ce domaine.
UsserID indique l'utilisateur du démon, attention le fichier de la clé privée doit
pouvoir être lu par cet utilisateur.
Socket indique la socket qui sera utilisée par postfix pour se connecter et signer les mails transmis.

Génération de la clé de signature des mails :
```bash
cd /etc/dkimkeys/
opendkim-genkey -t -s dkim -d ecreall.com
chown root:opendkim dkim.private
chmod 660 dkim.private
```

L'attribut `-s dkim` permet de préciser de signer différemment chaque mail selon son domaine dans le cas où le serveur gère plusieurs domaines.

On a alors :
```bash
ls -lh
```
```bash
total 12K
-rw-rw---- 1 root opendkim 1,7K févr. 11 16:09 dkim.private
-rw------- 1 root root      508 févr. 11 16:09 dkim.txt
-rw-r--r-- 1 root root      664 déc.  27  2019 README.PrivateKeys
```

Vérifier la clé :
```bash
opendkim-testkey -d ecreall.com -s dkim -vvv
```

```
opendkim-testkey: using default configfile /etc/opendkim.conf
opendkim-testkey: /etc/dkimkeys/dkim.private: WARNING: unsafe permissions
opendkim-testkey: key loaded from /etc/dkimkeys/dkim.private
opendkim-testkey: checking key 'dkim._domainkey.ecreall.com'
opendkim-testkey: key not secure
opendkim-testkey: key OK
```

La ligne \"opendkim-testkey: key not secure\" est due au fait que DNSSEC n'a pas été activé sur le dns.

Afficher le contenu de dkim.txt

```bash
cat /etc/dkimkeys/dkim.txt
```

```
dkim._domainkey	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; t=y; "
	  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAu16oOGtsGwvwF0p4+Oa3ZLS7/9TjKagcXGpiBKKrT0fJrZfKhsp43xv2NsH1SM56aBycxxrvECasx8cOQtHJ8ajEgIKAjmWW1C3ISJCNhebyxr+YuH2UlfH5IZByonVi0qJIW2oaZRJMusXr7yM4u1j/oKbLGLWcxLvEr9BMuPDiJHyadKVHhVmNjqfDA8F5Q1vXsno5e8rvPO"
	  "UAhkcr8+TF2/J/kyhy6JyfW84TgxnIcU0mCFATAtDd871gZWqvOiUltku1aXteotSUonsaLDWpcmcwgyAgG7c/sycOPmA+rHvuiZ2/HPNzgW1/U55E2ijrTXzy+43zTAzzcdg/GwIDAQAB" )  ; ----- DKIM key dkim for ecreall.com

```
Configurer notre registrar :

- Se connecter à la zone DNS de son domaine, par exemple pour Ecreall dans OVH, nous ajoutons une entrée DKIM en remplissant les champs comme suit ::

```
      Sous-domaine [dkim._domainkey]
      Version [v]
      Algorithme (hash) -256 [v]
      Clé Publique [MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1YRDtepyDeIgVolfFz4bRgacdE0hxGFhB+9XTXmZbYcPc0iyDaGJivpd7TYAZ2zRBG+wU6s8viK9mxA/JLDTklhdbnD2oQOjBA1g7bcqqo/F3gHbApaz/M2DrQ4y5HEaHTjm/bsCLzbO7v3buTuhxu6mpVp5m/q+uX7o2LB1GkTw/DbqE2j3tHx5N5sojX6dZxvk+V9nyInArY4ni3uWrH3Y8aLSK7+QHyZVJAVGiT6jdRDdEERQlo2CTZj6UQu3jGtic+GZCU8Hp/SJWQj/xrx/ygZEJ0z294fLEIOGgUw66vRl6iVE9NJCaavTqBxlfgX7QNOa/9bqnR9uDI2flwIDAQAB
      ]

      Type de service []

      Mode test [v] # Permet de demander aux serveurs recevant nos mails de ne pas tenir compte de DKIM tant qu'on a pas fini

      sous-domaine [v] La clé publique n'est pas valide pour les sous-domaines
```

Quand tout est au point, ne pas oublier d'éditer l'entrée pour enlever le mode de test.

### dmarc

L'ajout de DMARC se fait par simple ajoute d'une entrée de type DMARC ou TXT pour le sous-domaine \_dmarc avec les valeurs suivantes :

```
v=DMARC1;p=quarantine;pct=100;rua=mailto:michaellaunay+dmarc@ecreall.com;sp=quarantine;aspf=s;
```

Où V est la version du protocole, p est la politique à appliquer pour les messages reçus soi-disant de notre domaine, mais qui échoue, None (ne rien faire) ou Quarantine
marquer douteux, Reject rejeter.

pct le pourcentage à traiter.

rua l'uri de la ressource à prévenir en cas d'usurpation.

sp la politique des sous domainkeys.

aspf s'il faut suivre spf à la lettre.

# Installer Dovecot

Dovecot est un serveur IMAP et POP3 conçu pour être léger et facile à configurer.
Nous avons configuré postfix pour utiliser le format Maildir et allons voir comment configurer Dovecot pour ce format.
## Installation de Dovecot

Installons Dovecot avec la commande :

```bash
sudo apt-get install dovecot-imapd
```

## Configuration de Dovecot avec Maildir

Éditons le fichier de configuration principale `/etc/dovecot/dovecot.conf` et assurons-nous que les lignes suivantes sont présentes:

```txt
mail_location = maildir:~/Maildir
```

Cela indique à Dovecot d'utiliser le format Maildir pour le stockage des e-mails.

# Tester la configuration

Aller sur le site [appmaildev.com](https://appmaildev.com)
```bash
echo test | mail -s "Test postfix" -aFROM:michaellaunay@ecreall.com test-1fc42c52@appmaildev.com
```


# Créer un serveur postfix de test

Pour développer il peut être intéressant d'installer un postfix et rediriger tout le traffic dans des boites locales.
```bash
sudo apt install postfix mailutils
# Choisir l'installation "site" et positioner un nom de domaine
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

Générer la table de correspondances et redémarrer.
```bash
postmap /etc/postfix/recipient_canonical
systemctl restart postfix
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
smtpd_recipient_restrictions = reject #Bloque tout envoi
sender_canonical_maps = regexp:/etc/postfix/sender_canonical #modifie l'émetteur
sender_bcc_maps = regexp:/etc/postfix/sender_bcc #Modifie le CC
always_bcc = destinataire_origine@example.com #Ajoute toujours un CC
```
