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
  - smtp
  - securite
resume: "Cours complet sur Postfix : architecture d'un MTA, SMTP et submission, configuration de main.cf et master.cf, relais, domaines locaux et virtuels, SASL, TLS, LMTP/Dovecot, SPF/DKIM/DMARC, sécurité, files d'attente, observabilité, délivrabilité et diagnostic en production."
niveau: intermediaire
prerequis:
  - "[[GNULinux]]"
  - "[[Les protocoles de communications]]"
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-05-30
date_modification: 2026-08-30
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Refonte
> Cours développé le 31 août 2026 (d'environ 5 787 à 9 472 mots  remplacement complet du texte précédent)  vérifié le même jour : schéma  titres  liens et affirmations datées contrôlés. La version précédente reste dans l'historique git du dépôt. Les éléments spécifiques de l'ancienne version sont conservés en annexe  en fin de note.

# Postfix

> [!abstract] Idée centrale
> **Postfix est un MTA** : il reçoit, met en file d'attente, route et transmet des messages. Il ne faut pas confondre ce rôle avec celui d'un serveur IMAP, d'un webmail, d'un filtre antispam ou d'un gestionnaire d'identités. Une architecture de messagerie fiable vient précisément de cette séparation des responsabilités.

Ce cours part d'un serveur GNU/Linux administré en ligne de commande et progresse jusqu'à une architecture de production avec réception SMTP, soumission authentifiée, TLS, DKIM, SPF, DMARC, Dovecot/LMTP, domaines virtuels, audit et dépannage.

Les exemples utilisent des noms réservés comme `example.com` et `mail.example.com`. Nous devons les remplacer par nos propres domaines et adresses.

> [!warning] Sécurité et version
> Au 30 août 2026, la branche stable amont est **Postfix 3.11** et les branches maintenues par le projet sont 3.8 à 3.11. Des correctifs de sécurité importants ont encore été publiés en 2026. En production, nous utilisons les paquets de sécurité maintenus par notre distribution et nous ne laissons pas un ancien Postfix simplement parce qu'« il fonctionne encore ».

# Plan du cours

1. Positionner Postfix dans une architecture de messagerie
2. Comprendre une transaction SMTP
3. Comprendre l'architecture interne de Postfix
4. Installer Postfix et identifier sa version réelle
5. Maîtriser `main.cf`, `master.cf` et `postconf`
6. Construire l'identité DNS et SMTP du serveur
7. Comprendre les classes de domaines Postfix
8. Construire un serveur SMTP minimal sans open relay
9. Distinguer port 25 et soumission utilisateur
10. Ajouter SASL avec Dovecot
11. Configurer un smarthost ou relayhost authentifié
12. Maîtriser TLS entrant et sortant
13. Comprendre les politiques TLS avancées
14. Stockage local, Maildir, Dovecot et LMTP
15. Héberger des domaines virtuels
16. Maîtriser aliases, canonical maps et transport maps
17. Comprendre les tables Postfix et la migration hors Berkeley DB
18. SPF : autorisation de l'infrastructure d'envoi
19. DKIM : signature cryptographique des messages
20. DMARC : alignement, politique et rapports
21. Filtres, Milters et services de politique
22. Sécuriser un serveur exposé à Internet
23. Comprendre et manipuler la file d'attente
24. Logs, observabilité et commandes d'audit
25. Diagnostiquer un problème de réception ou d'envoi
26. Diagnostiquer la délivrabilité
27. Cas particulier : blocages de réputation Microsoft
28. Architectures types
29. Laboratoires pratiques
30. Checklist de mise en production

---

# 1. Positionner Postfix dans une architecture de messagerie

## 1.1. Les rôles principaux

Une architecture de courrier électronique contient plusieurs rôles distincts.

| Rôle | Signification | Exemple |
|---|---|---|
| MUA | Mail User Agent | Thunderbird, Outlook, application mobile |
| MSA | Mail Submission Agent | service SMTP authentifié sur 587/465 |
| MTA | Mail Transfer Agent | Postfix |
| MDA/LDA | Mail Delivery Agent / Local Delivery Agent | `local(8)`, Dovecot LDA |
| LMTP server | Livraison finale via LMTP | Dovecot LMTP |
| IMAP server | Consultation et synchronisation des boîtes | Dovecot |
| Webmail | Interface web | Roundcube, SnappyMail, etc. |
| Filtre | Antispam, antivirus, DKIM, DMARC | Rspamd, OpenDKIM, OpenDMARC, Milter |

Postfix peut jouer plusieurs rôles SMTP :

- **MTA entrant** sur le port 25 ;
- **MTA sortant** vers les MX distants ;
- **MSA** lorsqu'un service `submission` est configuré ;
- **relais interne** ou passerelle SMTP ;
- **client SMTP** vers un smarthost ;
- **routeur** vers Dovecot, un autre MTA ou une application.

Il ne fournit pas nativement l'expérience complète d'une boîte de messagerie moderne. Pour lire les messages sur plusieurs appareils, nous associons généralement Postfix à Dovecot.

## 1.2. Transport, soumission et consultation

Ne mélangeons pas les protocoles :

| Usage | Protocole | Port courant |
|---|---|---:|
| MTA vers MTA | SMTP | 25 |
| Client vers serveur d'envoi | Message Submission + STARTTLS | 587 |
| Client vers serveur d'envoi | Submission avec TLS implicite | 465 |
| Consultation | IMAP + STARTTLS | 143 |
| Consultation | IMAP avec TLS implicite | 993 |
| Ancienne récupération | POP3 + STARTTLS | 110 |
| Ancienne récupération | POP3 avec TLS implicite | 995 |
| Webmail | HTTPS | 443 |

Le **port 25** doit rester compatible avec les autres serveurs SMTP d'Internet. Nous ne devons donc pas imposer sur ce port les mêmes contraintes qu'à nos utilisateurs authentifiés.

Le **port 587** sert à la soumission par un utilisateur ou une application authentifiée. C'est ici que nous pouvons imposer TLS et SASL.

## 1.3. Mbox et Maildir

Deux formats historiques de stockage local sont fréquents.

### Mbox

Une boîte est principalement stockée dans un fichier contenant plusieurs messages. Le verrouillage et les accès concurrents sont donc importants.

### Maildir

Chaque message est un fichier séparé dans une arborescence contenant notamment :

```text
Maildir/
├── cur/
├── new/
└── tmp/
```

Maildir évite le gros fichier partagé de mbox et s'intègre bien avec Dovecot. Il reste toutefois un **format de stockage**, pas un protocole.

---

# 2. Comprendre une transaction SMTP

## 2.1. Enveloppe SMTP et contenu du message

Un e-mail possède deux niveaux qu'il faut distinguer :

1. **l'enveloppe SMTP** : `MAIL FROM`, `RCPT TO` ;
2. **le message** transmis après `DATA` : en-têtes `From:`, `To:`, `Subject:`, corps MIME, pièces jointes, etc.

Exemple simplifié :

```text
S: 220 mail.example.net ESMTP Postfix
C: EHLO mail.example.com
S: 250-mail.example.net
S: 250-STARTTLS
S: 250 SIZE 52428800
C: MAIL FROM:<bounce@example.com>
S: 250 2.1.0 Ok
C: RCPT TO:<alice@example.net>
S: 250 2.1.5 Ok
C: DATA
S: 354 End data with <CR><LF>.<CR><LF>
C: From: Service <news@example.com>
C: To: Alice <alice@example.net>
C: Subject: Test
C:
C: Bonjour
C: .
S: 250 2.0.0 Ok: queued as 3F4A212345
C: QUIT
```

Le domaine évalué par SPF est généralement celui du **MAIL FROM** et non nécessairement celui du champ visible `From:`. DMARC, lui, vérifie l'alignement avec le domaine du champ `From:` visible.

## 2.2. Codes SMTP

La première classe du code indique la nature de la réponse :

- `2xx` : succès ;
- `3xx` : étape intermédiaire ;
- `4xx` : erreur temporaire, le client doit réessayer ;
- `5xx` : erreur permanente pour cette transaction.

Les codes étendus comme `5.7.1` ou `4.7.1` ajoutent une information plus précise.

> [!tip] Diagnostic
> La **commande SMTP qui précède le rejet** est capitale. Un rejet à `RCPT TO` n'a pas la même cause qu'un rejet après `DATA`. Si un serveur refuse à `MAIL FROM`, il n'a pas encore reçu la signature DKIM située dans les en-têtes du message.

## 2.3. SMTP n'est pas un protocole « temps réel garanti »

Lorsque le serveur distant est indisponible, Postfix ne perd pas immédiatement le message. Il le place généralement en file `deferred` et réessaie selon sa politique de backoff jusqu'à expiration de la durée maximale de mise en file.

C'est une propriété centrale de l'e-mail : **store and forward**.

---

# 3. Architecture interne de Postfix

Postfix n'est pas un unique démon monolithique. Il sépare les rôles en plusieurs processus avec des privilèges et responsabilités distincts.

## 3.1. Chemin d'un message reçu par SMTP

```text
Internet
   |
   v
smtpd
   |
   v
cleanup ----> filtres / Milters / réécritures
   |
   v
incoming queue
   |
   v
qmgr
   |
   +--------> smtp -------> serveur SMTP distant
   |
   +--------> local ------> compte UNIX / alias
   |
   +--------> virtual ----> boîte virtuelle
   |
   +--------> lmtp -------> Dovecot / service LMTP
   |
   +--------> pipe -------> programme externe
```

## 3.2. Injection locale

Une commande compatible Sendmail ou un programme local peut injecter un message sans passer par `smtpd` :

```text
application
   |
   v
sendmail(1)
   |
   v
maildrop queue
   |
   v
pickup
   |
   v
cleanup -> qmgr -> transport
```

Cette différence explique pourquoi une configuration DKIM utilise souvent à la fois :

```ini
smtpd_milters = ...
non_smtpd_milters = ...
```

Le second couvre les messages qui n'entrent pas par `smtpd`.

## 3.3. Files principales

Nous rencontrerons notamment :

- `maildrop` : messages injectés localement en attente de `pickup` ;
- `incoming` : messages fraîchement acceptés ;
- `active` : messages actuellement pris en charge par `qmgr` ;
- `deferred` : livraison temporairement impossible ;
- `hold` : messages mis volontairement en attente ;
- `corrupt` : éléments de file détectés comme invalides.

Nous ne modifions jamais les fichiers de la queue à la main.

## 3.4. `master(8)`

Le processus `master` démarre et supervise les services Postfix décrits dans `master.cf`. Il peut, par exemple, lancer un `smtpd` pour le port 25 avec une politique et un autre `smtpd` pour le port 587 avec des options différentes.

---

# 4. Installer Postfix et identifier sa version réelle

## 4.1. Installation sur Debian/Ubuntu

```bash
sudo apt update
sudo apt install postfix
```

Des outils utiles :

```bash
sudo apt install swaks dnsutils openssl mailutils
```

Les paquets disponibles dépendent de la distribution ; ne copions pas aveuglément une commande d'installation provenant d'un tutoriel pour une autre version de Debian, Ubuntu, Fedora ou RHEL.

## 4.2. Version installée

```bash
postconf mail_version
```

Exemple :

```text
mail_version = 3.11.6
```

Vérifions également la version du paquet et sa provenance :

```bash
dpkg-query -W postfix
apt-cache policy postfix
```

Sur une autre famille de distributions, utilisons le gestionnaire de paquets adapté.

## 4.3. Branches et maintenance

En août 2026, l'amont Postfix maintient les branches 3.8 à 3.11 et 3.11 est la stable courante. La version de notre distribution peut différer car un éditeur peut **backporter** des correctifs de sécurité sans changer vers la dernière version mineure amont.

Nous devons donc répondre à deux questions différentes :

1. quelle version amont est installée ?
2. le paquet de notre distribution reçoit-il encore les correctifs de sécurité ?

## 4.4. `compatibility_level`

Postfix possède un mécanisme de compatibilité qui permet à une nouvelle version de conserver temporairement certains anciens comportements.

```bash
postconf compatibility_level
```

Lorsque les logs indiquent qu'un paramètre utilise un « backwards-compatible default », nous devons :

1. comprendre le changement ;
2. rendre explicite la valeur souhaitée si nécessaire ;
3. tester ;
4. seulement ensuite relever le niveau de compatibilité approprié.

Nous ne modifions pas `compatibility_level` au hasard pour faire disparaître un avertissement.

---

# 5. `main.cf`, `master.cf` et `postconf`

## 5.1. Les deux fichiers essentiels

Le répertoire courant se récupère avec :

```bash
postconf config_directory
```

Généralement :

```text
/etc/postfix
```

Nous trouvons principalement :

- `/etc/postfix/main.cf` : paramètres globaux ;
- `/etc/postfix/master.cf` : services, sockets, ports et surcharges par service.

## 5.2. Lire la configuration sans deviner

Afficher uniquement les valeurs explicitement configurées :

```bash
postconf -n
```

Afficher une valeur effective :

```bash
postconf myhostname mydomain myorigin mydestination
```

Afficher la valeur par défaut compilée :

```bash
postconf -d smtp_tls_security_level
```

Lister les types de tables disponibles :

```bash
postconf -m
```

Lister les backends SASL serveur et client :

```bash
postconf -a
postconf -A
```

Inspecter `master.cf` :

```bash
postconf -M
```

À partir de Postfix 3.11, plusieurs outils savent également produire du JSON, notamment `postconf -j`, ce qui simplifie les audits automatisés.

## 5.3. Modifier avec `postconf -e`

Nous pouvons modifier un paramètre sans éditer manuellement le fichier :

```bash
sudo postconf -e 'myhostname = mail.example.com'
```

Pour supprimer une surcharge et revenir au défaut :

```bash
sudo postconf -X nom_du_parametre
```

Avant de faire de l'automatisation, vérifions la documentation de la version réellement installée :

```bash
man postconf
```

## 5.4. Valider et recharger

```bash
sudo postfix check
sudo postfix reload
```

Un `reload` suffit pour la majorité des changements. Un redémarrage complet n'est pas notre réflexe par défaut.

Vérifions ensuite :

```bash
systemctl status postfix --no-pager
journalctl -u postfix -n 100 --no-pager
```

---

# 6. Construire l'identité DNS et SMTP du serveur

Prenons :

```text
Domaine       : example.com
Nom du MTA    : mail.example.com
IPv4          : 192.0.2.25
IPv6          : 2001:db8::25
```

Les préfixes ci-dessus sont réservés à la documentation.

## 6.1. `myhostname`

Le serveur doit posséder un FQDN stable :

```ini
myhostname = mail.example.com
```

Vérifions :

```bash
hostnamectl
hostname -f
postconf myhostname
```

## 6.2. DNS direct

```dns
mail.example.com.  IN A     192.0.2.25
mail.example.com.  IN AAAA  2001:db8::25
```

Si nous publions une adresse IPv6, le serveur doit réellement pouvoir émettre correctement en IPv6. Une IPv6 cassée peut dégrader les livraisons.

## 6.3. MX

```dns
example.com. IN MX 10 mail.example.com.
```

La cible d'un MX doit être un nom d'hôte résolvable par A/AAAA. Évitons de placer un alias CNAME comme cible MX.

Test :

```bash
dig MX example.com +short
getent ahosts mail.example.com
```

## 6.4. PTR / reverse DNS

Le reverse est généralement configuré chez l'hébergeur de l'adresse IP :

```text
192.0.2.25 -> PTR -> mail.example.com
```

Puis le nom doit revenir vers l'adresse publique :

```text
mail.example.com -> A -> 192.0.2.25
```

Test :

```bash
dig -x 192.0.2.25 +short
dig A mail.example.com +short
```

Un PTR cohérent n'est pas une preuve de légitimité, mais son absence ou son incohérence est un signal négatif fréquent pour la délivrabilité.

## 6.5. EHLO/HELO

Le serveur émet normalement son `myhostname` :

```bash
postconf smtp_helo_name myhostname
```

N'écrasons pas `smtp_helo_name` sans raison. L'identité présentée doit être cohérente avec notre architecture et notre DNS.

## 6.6. Vérifier la vue DNS publique

`/etc/hosts`, un split DNS ou `systemd-resolved` peuvent faire apparaître une adresse différente localement.

Comparons :

```bash
getent ahosts mail.example.com
dig mail.example.com A +short
dig @1.1.1.1 mail.example.com A +short
```

Pour la réputation Internet, c'est la vue publique observée par les destinataires qui compte.

---

# 7. Les classes de domaines Postfix

Une grande partie des erreurs de configuration vient d'une mauvaise compréhension de la classe d'un domaine.

## 7.1. Domaines locaux : `mydestination`

```ini
mydestination = $myhostname, localhost.$mydomain, localhost, example.com
```

Ces domaines sont des destinations **locales**. La livraison passe typiquement par `local(8)` vers des comptes UNIX et leurs aliases.

## 7.2. Domaines d'alias virtuels

```ini
virtual_alias_domains = example.org
virtual_alias_maps = lmdb:/etc/postfix/virtual
```

Ils ne possèdent pas directement de boîte Postfix ; chaque adresse est réécrite vers une autre adresse.

Exemple :

```text
contact@example.org   alice@example.com
```

## 7.3. Domaines de boîtes virtuelles

```ini
virtual_mailbox_domains = example.net
```

Ils hébergent des boîtes qui ne correspondent pas nécessairement à des comptes UNIX. La livraison se fait souvent vers Dovecot en LMTP.

## 7.4. Domaines relais

```ini
relay_domains = partner.example
```

Le serveur accepte les messages pour ces domaines puis les transmet ailleurs. Il n'est pas la destination finale.

## 7.5. Règle fondamentale

> [!danger] Classes exclusives
> Un domaine de boîtes virtuelles ne doit pas aussi être placé dans `mydestination` ou `virtual_alias_domains`. Nous déterminons d'abord le **rôle du serveur pour ce domaine**, puis nous choisissons la classe correspondante.

Cette classification permet ensuite à Postfix de choisir correctement le transport et le next-hop.

---

# 8. Serveur SMTP minimal sans open relay

## 8.1. Exemple de base

Voici une base pédagogique pour un serveur qui reçoit `example.com` localement :

```ini
# Identité
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain

# Interfaces
inet_interfaces = all
inet_protocols = all

# Domaines locaux
mydestination = $myhostname, $mydomain, localhost.$mydomain, localhost

# Réseaux autorisés à relayer sans SASL
mynetworks = 127.0.0.0/8 [::1]/128

# Pas de smarthost global
relayhost =

# Anti-open-relay
smtpd_relay_restrictions =
    permit_mynetworks,
    permit_sasl_authenticated,
    defer_unauth_destination

# TLS SMTP public : opportuniste
smtpd_tls_security_level = may
smtp_tls_security_level = may
```

> [!warning] `mynetworks`
> Toute machine incluse dans `mynetworks` peut généralement obtenir des privilèges de relais. N'ajoutons pas un LAN complet, un réseau Docker, un VPN ou un cloud CIDR sans comprendre l'effet exact.

## 8.2. `smtpd_relay_restrictions`

La protection du relais doit être facile à auditer :

```ini
smtpd_relay_restrictions =
    permit_mynetworks,
    permit_sasl_authenticated,
    defer_unauth_destination
```

Cela signifie en substance :

1. autoriser nos réseaux explicitement de confiance ;
2. autoriser les utilisateurs SASL authentifiés ;
3. sinon, n'accepter que les destinataires dont ce serveur est responsable.

Nous ne remplaçons pas cette logique par une grande suite obscure de restrictions copiées depuis un forum.

## 8.3. Tester l'absence d'open relay

Depuis une machine extérieure non autorisée :

```bash
swaks \
  --server mail.example.com \
  --from attacker@example.net \
  --to victim@example.org \
  --quit-after RCPT
```

Si `example.org` est un domaine tiers et que nous ne sommes pas authentifiés, le serveur doit refuser le relais.

Nous testons aussi un destinataire réellement local pour vérifier que la réception n'a pas été cassée.

---

# 9. Port 25 et soumission utilisateur

## 9.1. Pourquoi séparer les services

Le port 25 public :

- accepte des serveurs inconnus ;
- ne peut pas imposer une authentification utilisateur ;
- utilise généralement TLS opportuniste ;
- applique des politiques anti-abus orientées Internet.

Le port 587 :

- cible nos utilisateurs/applications ;
- exige SASL ;
- peut imposer TLS ;
- autorise le relais après authentification ;
- peut appliquer des politiques d'expéditeur différentes.

## 9.2. Exemple `master.cf` pour 587

```text
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_tls_auth_only=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

L'exemple doit être adapté à notre distribution et à notre architecture. Vérifions la ligne effective avec :

```bash
postconf -M submission/inet
```

## 9.3. Port 465

Le port 465 utilise TLS implicite. Selon le `master.cf` fourni par la distribution, le service peut être nommé `submissions` ou historiquement `smtps`.

Le point essentiel est :

```text
-o smtpd_tls_wrappermode=yes
```

Ne configurons pas `wrappermode` sur le port 25.

## 9.4. Vérifier les ports

```bash
ss -ltnp | grep -E ':(25|465|587)\b'
```

Puis :

```bash
openssl s_client -starttls smtp -connect mail.example.com:587 -servername mail.example.com
```

Pour 465 :

```bash
openssl s_client -connect mail.example.com:465 -servername mail.example.com
```

---

# 10. SASL avec Dovecot

Postfix n'implémente pas lui-même le mécanisme d'authentification SASL ; il s'appuie sur un backend, notamment Dovecot ou Cyrus SASL côté serveur.

## 10.1. Socket Dovecot

Dans Dovecot, un exemple classique est :

```text
service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}
```

Puis dans Postfix :

```ini
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_security_options = noanonymous
```

La notation `private/auth` est relative au `queue_directory`, généralement `/var/spool/postfix`, et fonctionne aussi lorsque certains services sont chrootés.

## 10.2. N'activer AUTH que là où il faut

Notre préférence est d'activer :

```ini
smtpd_sasl_auth_enable = yes
```

comme surcharge du service `submission`, plutôt que d'offrir AUTH sans distinction sur tous les `smtpd`.

## 10.3. Tester

```bash
swaks \
  --server mail.example.com:587 \
  --tls \
  --auth LOGIN \
  --auth-user alice@example.com \
  --auth-password 'MOT_DE_PASSE_DE_TEST' \
  --from alice@example.com \
  --to recipient@example.net
```

Évitons de mettre un vrai mot de passe dans l'historique du shell. Pour des tests sérieux, utilisons une méthode de saisie ou un secret temporaire approprié.

## 10.4. Contrôler l'expéditeur authentifié

L'authentification ne signifie pas forcément qu'un utilisateur doit pouvoir prétendre être n'importe quelle adresse.

Pour une plateforme multi-utilisateur, nous pouvons utiliser des mécanismes comme :

```ini
smtpd_sender_login_maps = ...
smtpd_sender_restrictions = reject_sender_login_mismatch
```

La table doit refléter la politique réelle des identités autorisées.

---

# 11. Utiliser un relayhost authentifié

Un serveur n'envoie pas forcément directement vers les MX. Il peut transmettre tout le courrier à un smarthost.

## 11.1. Configuration

```ini
relayhost = [smtp.provider.example]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = lmdb:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = encrypt
smtp_tls_CApath = /etc/ssl/certs
```

Dans `/etc/postfix/sasl_passwd` :

```text
[smtp.provider.example]:587 login@example.com:mot-de-passe
```

Puis :

```bash
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap lmdb:/etc/postfix/sasl_passwd
sudo postfix check
sudo postfix reload
```

> [!note] Type de table
> `lmdb:` n'est qu'un exemple moderne. Vérifions d'abord `postconf -m`. Sur des installations plus anciennes, nous rencontrerons souvent `hash:`. Postfix 3.11 accompagne la migration des environnements où Berkeley DB disparaît.

## 11.2. Crochets autour du relayhost

```ini
relayhost = [smtp.provider.example]:587
```

Les crochets demandent d'utiliser directement ce nom d'hôte comme next-hop au lieu d'effectuer la recherche MX normale du domaine.

## 11.3. Ne pas mettre `smtp_tls_security_level = encrypt` partout

Pour un smarthost contrôlé, imposer TLS est logique. Pour la livraison **directe vers l'ensemble d'Internet**, `encrypt` global peut empêcher la livraison vers des domaines dont le serveur distant n'offre pas correctement STARTTLS.

La valeur courante pour l'Internet public reste :

```ini
smtp_tls_security_level = may
```

avec des politiques plus strictes ciblées là où nous pouvons les garantir.

---

# 12. TLS entrant et sortant

## 12.1. TLS entrant sur le port 25

Exemple :

```ini
smtpd_tls_security_level = may
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.example.com/fullchain.pem
smtpd_tls_key_file = /etc/letsencrypt/live/mail.example.com/privkey.pem
```

Le certificat doit correspondre au nom présenté par le serveur.

Test :

```bash
openssl s_client \
  -starttls smtp \
  -connect mail.example.com:25 \
  -servername mail.example.com
```

## 12.2. TLS sur 587

La soumission est un contexte différent :

```text
-o smtpd_tls_security_level=encrypt
```

Ici, le client est sous notre contrôle et nous pouvons exiger TLS avant AUTH.

## 12.3. TLS sortant opportuniste

```ini
smtp_tls_security_level = may
```

Le client Postfix essaie STARTTLS lorsqu'il est disponible mais peut livrer en clair si le serveur distant ne l'offre pas.

Ce comportement est intentionnel pour la compatibilité SMTP sur Internet. **TLS opportuniste chiffre sans authentifier nécessairement le serveur distant.**

## 12.4. Ne pas réinventer les listes de chiffrements sans besoin

Avec un Postfix, OpenSSL et une distribution maintenus, les défauts modernes sont souvent plus sûrs qu'une vieille liste de `!SSLv3`, `!RC4`, `HIGH:!aNULL` copiée il y a dix ans.

Nous ne fixons explicitement protocoles et chiffrements que lorsque :

- la politique de sécurité l'exige ;
- nous comprenons l'impact d'interopérabilité ;
- nous testons avec la bibliothèque TLS réellement installée.

---

# 13. Politiques TLS avancées

## 13.1. `smtp_tls_policy_maps`

Nous pouvons appliquer une politique par destination :

```ini
smtp_tls_policy_maps = lmdb:/etc/postfix/tls_policy
```

Exemples conceptuels :

```text
example.net              encrypt
[mail.partner.test]:25   secure match=nexthop
```

Puis :

```bash
sudo postmap lmdb:/etc/postfix/tls_policy
sudo postfix reload
```

Les niveaux courants incluent notamment :

- `none` ;
- `may` ;
- `encrypt` ;
- `dane` ;
- `dane-only` ;
- `fingerprint` ;
- `verify` ;
- `secure`.

Ils n'ont pas tous la même sémantique d'authentification. Lisons `TLS_README` avant de choisir un niveau autre que `may` ou `encrypt`.

## 13.2. DANE

Postfix sait utiliser DANE/TLSA pour authentifier les serveurs distants lorsque la chaîne DNS est validée par DNSSEC.

Prérequis typiques :

```ini
smtp_dns_support_level = dnssec
smtp_tls_security_level = dane
```

mais le résultat n'est sûr que si le résolveur utilisé valide réellement DNSSEC et si la configuration système est correcte.

DANE ne se résume donc pas à publier un enregistrement TLSA.

## 13.3. MTA-STS

MTA-STS fournit une politique HTTPS indiquant quels MX sont autorisés et que TLS doit être utilisé. Postfix peut être intégré à des **plugins/résolveurs MTA-STS externes** via ses mécanismes de politique TLS.

Postfix 3.11 a renforcé l'intégration avec ces plugins grâce au contrôle des motifs MX de la politique STS.

Ne confondons pas :

- DANE : ancré dans DNSSEC/TLSA ;
- MTA-STS : politique publiée via DNS + HTTPS ;
- TLS-RPT : rapports sur les échecs TLS ;
- TLS opportuniste : simple tentative STARTTLS sans politique externe forte.

## 13.4. REQUIRETLS

Postfix 3.11 apporte le support SMTP de **REQUIRETLS** (RFC 8689). Cette extension permet d'exprimer qu'un message doit être transmis avec les garanties TLS définies par le protocole.

REQUIRETLS est une fonctionnalité de transport de bout en bout entre relais compatibles ; elle ne remplace ni DKIM, ni DMARC, ni le chiffrement du contenu comme OpenPGP ou S/MIME.

---

# 14. Stockage local, Dovecot et LMTP

## 14.1. Livraison locale simple en Maildir

Pour des comptes UNIX :

```ini
home_mailbox = Maildir/
```

Postfix livre alors typiquement dans :

```text
/home/alice/Maildir/
```

Nous pouvons vérifier :

```bash
postconf home_mailbox mailbox_command
```

## 14.2. Dovecot pour IMAP

Dovecot expose les boîtes aux clients et peut utiliser Maildir.

Une ancienne configuration Dovecot 2.x peut contenir :

```text
mail_location = maildir:~/Maildir
```

La syntaxe exacte dépend de la version Dovecot. Nous consultons sa documentation plutôt que de supposer que cette ligne reste identique dans toutes les générations.

## 14.3. Pourquoi LMTP est intéressant

Au lieu de faire écrire directement Postfix dans le stockage, nous pouvons déléguer la livraison finale à Dovecot :

```text
Postfix -> LMTP -> Dovecot -> stockage
```

Avantages :

- Dovecot gère quotas et règles de livraison ;
- une seule couche connaît précisément le backend de boîtes ;
- Postfix reste centré sur SMTP et le routage.

## 14.4. Exemple de socket LMTP

Côté Dovecot :

```text
service lmtp {
  unix_listener /var/spool/postfix/private/dovecot-lmtp {
    mode = 0600
    user = postfix
    group = postfix
  }
}
```

Côté Postfix :

```ini
virtual_transport = lmtp:unix:private/dovecot-lmtp
```

Les permissions et chemins exacts dépendent de notre modèle d'utilisateurs et de notre distribution.

---

# 15. Héberger des domaines virtuels

## 15.1. Pourquoi des utilisateurs virtuels

Sur un hébergeur de courrier, il n'est généralement pas souhaitable de créer un compte UNIX pour chaque adresse e-mail.

Nous utilisons alors :

- `virtual_mailbox_domains` ;
- `virtual_mailbox_maps` ;
- éventuellement `virtual_alias_maps` ;
- un transport LMTP vers Dovecot ;
- une source d'identité LDAP, SQL ou fichier selon l'échelle.

## 15.2. Exemple conceptuel

```ini
virtual_mailbox_domains = example.net, example.org
virtual_mailbox_maps = lmdb:/etc/postfix/vmailbox
virtual_alias_maps = lmdb:/etc/postfix/virtual
virtual_transport = lmtp:unix:private/dovecot-lmtp
```

`/etc/postfix/vmailbox` sert ici à indiquer les destinataires connus :

```text
alice@example.net    OK
bob@example.org      OK
```

Le résultat exact de la table dépend du transport utilisé. Avec un service externe de stockage, elle sert souvent surtout à valider l'existence du destinataire.

## 15.3. Rejeter les destinataires inconnus pendant SMTP

Nous préférons :

```text
client -> RCPT TO inconnu -> 550 utilisateur inconnu
```

plutôt que :

```text
client -> message accepté -> bounce généré plus tard
```

Le second comportement favorise le **backscatter**, surtout lorsque l'expéditeur de l'enveloppe a été usurpé.

## 15.4. Catch-all

Une adresse catch-all paraît pratique :

```text
@example.net    catchall@example.net
```

mais elle reçoit également tout le spam envoyé à des adresses inexistantes. Dans la majorité des déploiements modernes, nous préférons des destinataires explicitement valides.

---

# 16. Aliases, canonical maps et transport maps

## 16.1. Aliases locaux

`/etc/aliases` :

```text
postmaster: alice
abuse: alice
root: alice
```

Puis :

```bash
sudo newaliases
```

`postmaster` doit être opérationnelle sur un domaine de messagerie. `abuse` est également fortement recommandée pour l'exploitation et les procédures de réputation.

## 16.2. Virtual aliases

```ini
virtual_alias_maps = lmdb:/etc/postfix/virtual
```

```text
contact@example.com   alice@example.net
sales@example.com     bob@example.net, carol@example.net
```

Après modification d'une table indexée :

```bash
sudo postmap lmdb:/etc/postfix/virtual
```

## 16.3. Canonical maps

Les tables canonical réécrivent des adresses :

```ini
sender_canonical_maps = regexp:/etc/postfix/sender_canonical
recipient_canonical_maps = regexp:/etc/postfix/recipient_canonical
```

Une table `regexp:` est lue directement ; il n'y a pas de base indexée à générer avec `postmap`.

Exemple :

```text
/^root@.*$/   admin@example.com
```

Les réécritures globales peuvent avoir des effets subtils sur l'enveloppe, les bounces et les signatures. Nous les utilisons avec parcimonie.

## 16.4. Transport maps

```ini
transport_maps = lmdb:/etc/postfix/transport
```

Exemple :

```text
partner.example    smtp:[mail.partner.example]:25
legacy.example     relay:[gateway.internal]:2525
```

Puis :

```bash
sudo postmap lmdb:/etc/postfix/transport
sudo postfix reload
```

Une `transport_map` permet d'outrepasser le transport et/ou le next-hop déterminé par la classe du domaine.

---

# 17. Tables Postfix et migration hors Berkeley DB

## 17.1. Types de tables

Postfix peut utiliser différents backends :

```bash
postconf -m
```

Nous pouvons rencontrer :

```text
btree
cdb
cidr
environ
fail
hash
inline
internal
ldap
lmdb
mysql
pgsql
pcre
proxy
regexp
socketmap
static
texthash
unix
```

La liste dépend de la compilation et des paquets installés.

## 17.2. Table source et base indexée

Avec un backend indexé :

```bash
sudo postmap lmdb:/etc/postfix/virtual
```

puis interrogeons :

```bash
postmap -q alice@example.com lmdb:/etc/postfix/virtual
```

Pour une table `regexp:` ou `pcre:`, nous ne lançons normalement pas `postmap` pour produire un fichier de base.

## 17.3. Fin progressive de Berkeley DB

Postfix 3.11 répond au retrait de Berkeley DB dans certaines distributions. Dans ces environnements, les maps `hash:` et `btree:` peuvent ne plus être disponibles et doivent migrer vers `lmdb:` ou `cdb:` selon le besoin.

Avant une migration :

```bash
postconf -m
postconf default_database_type
postconf -n | grep -E '(^|[ =])(hash|btree):'
```

L'amont fournit `NON_BERKELEYDB_README` pour guider cette migration.

> [!danger] Portabilité
> Ne construisons pas une automatisation qui suppose que `hash:` existe partout. Demandons à la machine ce qu'elle supporte.

## 17.4. Permissions des secrets

Une map contenant des mots de passe SMTP, clés API ou identifiants doit être protégée :

```bash
sudo chown root:root /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd
```

Pensons également au fichier de base généré par `postmap`.

---

# 18. SPF

## 18.1. Ce que SPF authentifie

SPF autorise des infrastructures d'envoi pour un domaine utilisé dans l'enveloppe SMTP, principalement `MAIL FROM`.

Exemple DNS :

```dns
example.com. IN TXT "v=spf1 ip4:192.0.2.25 ip6:2001:db8::25 -all"
```

Nous publions SPF sous forme de **TXT**. L'ancien type DNS `SPF` n'est plus utilisé.

## 18.2. Mécanismes courants

- `ip4:` ;
- `ip6:` ;
- `a` ;
- `mx` ;
- `include:` ;
- `exists:` ;
- `redirect=` ;
- `all`.

Qualificatifs :

- `+` pass ;
- `-` fail ;
- `~` softfail ;
- `?` neutral.

## 18.3. Limite de recherches DNS

SPF limite le nombre de mécanismes/modificateurs qui provoquent des recherches DNS. Une politique constituée d'une cascade d'`include:` peut donc devenir invalide même si sa chaîne semble syntaxiquement correcte.

Nous ne devons pas seulement faire :

```bash
dig TXT example.com +short
```

Nous devons aussi comprendre ce que résolvent réellement `include`, `a` et `mx`.

## 18.4. Un seul enregistrement SPF

Deux TXT différents commençant par `v=spf1` pour le même nom ne représentent pas deux politiques cumulatives : cela produit une erreur SPF.

Consolidons toutes les sources autorisées dans une seule politique.

## 18.5. SPF entrant avec Postfix

Postfix sait déléguer une décision à un **policy service**. Selon la distribution, nous pouvons utiliser un démon SPF fourni séparément, puis l'appeler avec une restriction comme :

```ini
check_policy_service unix:private/policyd-spf
```

Le chemin et le nom du service dépendent du paquet choisi.

Postfix n'a pas vocation à réimplémenter dans `smtpd` tous les standards d'authentification ; il expose au contraire des interfaces de policy delegation et Milter.

---

# 19. DKIM

## 19.1. Principe

DKIM ajoute une signature au message :

```text
DKIM-Signature: v=1; a=rsa-sha256; d=example.com; s=2026a; ...
```

Le destinataire récupère la clé publique sous :

```text
2026a._domainkey.example.com
```

Le serveur d'envoi conserve la clé privée.

Le **selector** permet :

- de faire tourner les clés ;
- d'utiliser plusieurs prestataires ;
- de séparer des flux ;
- de remplacer une clé sans modifier le principe du domaine DKIM.

## 19.2. OpenDKIM comme Milter

Une intégration courante reste :

```text
Postfix -> Milter -> OpenDKIM
```

D'autres stacks, comme Rspamd, peuvent assurer la signature et/ou la vérification DKIM. Le point Postfix à retenir est l'interface Milter.

## 19.3. Générer une clé OpenDKIM

Exemple :

```bash
sudo install -d -m 0750 -o root -g opendkim /etc/opendkim/keys/example.com
cd /etc/opendkim/keys/example.com
sudo opendkim-genkey -b 2048 -s 2026a -d example.com
sudo chown root:opendkim 2026a.private
sudo chmod 0640 2026a.private
```

Nous publions le contenu DNS proposé par le fichier `.txt`, en l'adaptant à l'interface de notre fournisseur DNS.

## 19.4. KeyTable et SigningTable

Exemple multi-domaines :

```text
# /etc/opendkim/KeyTable
2026a._domainkey.example.com example.com:2026a:/etc/opendkim/keys/example.com/2026a.private
```

```text
# /etc/opendkim/SigningTable
*@example.com 2026a._domainkey.example.com
```

Et une liste d'hôtes internes appropriée selon l'architecture.

## 19.5. Relier Postfix au Milter

Exemple socket local :

```ini
smtpd_milters = inet:127.0.0.1:8891
non_smtpd_milters = inet:127.0.0.1:8891
milter_protocol = 6
```

Pour `milter_default_action`, choisissons consciemment la stratégie :

- `accept` : fail-open, le mail continue sans le filtre ;
- `tempfail` : une panne du filtre provoque un échec temporaire ;
- les versions modernes de Postfix proposent aussi une action `shutdown`, devenue le défaut dans Postfix 3.11 afin d'améliorer la gestion de certaines erreurs Milter.

Pour une signature DKIM indispensable aux messages sortants, envoyer silencieusement sans signature peut être moins souhaitable qu'un `4xx` temporaire.

## 19.6. Tester la clé

```bash
opendkim-testkey -d example.com -s 2026a -vvv
```

Puis :

```bash
dig TXT 2026a._domainkey.example.com +short
```

Le message `key OK` confirme que la clé retrouvée dans le DNS correspond au test.

## 19.7. Rotation

Une rotation simple :

1. générer le selector `2026b` ;
2. publier sa clé publique ;
3. attendre sa propagation ;
4. basculer la signature vers `2026b` ;
5. vérifier les messages réels ;
6. conserver l'ancienne clé DNS assez longtemps pour vérifier les messages déjà en transit ;
7. retirer l'ancien selector plus tard.

---

# 20. DMARC

## 20.1. Ce que DMARC ajoute

DMARC relie SPF et DKIM au domaine visible dans l'en-tête :

```text
From: Service <newsletter@example.com>
```

Pour réussir DMARC, il faut qu'au moins un mécanisme réussisse **et soit aligné** avec ce domaine :

```text
DKIM PASS + alignement
OU
SPF PASS + alignement
```

## 20.2. Exemple

```dns
_dmarc.example.com. IN TXT "v=DMARC1; p=none; rua=mailto:dmarc@example.com"
```

Une progression prudente :

```text
p=none -> observation
p=quarantine -> enforcement partiel
p=reject -> politique stricte
```

Ne passons pas à `reject` avant d'avoir inventorié les vraies sources d'envoi : application métier, outil marketing, facturation, supervision, tickets, imprimantes, services cloud, etc.

## 20.3. Alignement strict ou relaxé

- `adkim=r` : alignement DKIM relaxé ;
- `adkim=s` : strict ;
- `aspf=r` : alignement SPF relaxé ;
- `aspf=s` : strict.

Le mode strict n'est pas automatiquement « meilleur » : il peut casser des flux légitimes si notre architecture utilise des sous-domaines ou prestataires différents.

## 20.4. Rapports

`rua=` indique l'adresse de rapports agrégés :

```text
rua=mailto:dmarc-reports@example.com
```

Les rapports sont surtout utiles lorsqu'ils sont agrégés et analysés automatiquement.

## 20.5. DMARC entrant

Vérifier DMARC sur les messages reçus requiert un composant externe, par exemple un Milter ou une plateforme antispam. Publier notre propre enregistrement DMARC ne suffit pas à vérifier celui des autres.

---

# 21. Filtres, Milters et policy services

## 21.1. Trois familles d'intégration

Postfix peut déléguer à plusieurs types de mécanismes :

1. **restrictions SMTP** dans `smtpd_*_restrictions` ;
2. **policy service** : décision externe légère basée sur les attributs SMTP ;
3. **Milter / content filter** : inspection plus riche des événements et du contenu.

## 21.2. Policy service

Le service reçoit des attributs comme :

- IP client ;
- HELO ;
- sender ;
- recipient ;
- informations SASL ;
- informations TLS.

Il renvoie une décision Postfix : accept, reject, defer, dunno, etc.

Cas d'usage :

- SPF ;
- quotas ou politiques d'envoi ;
- règles métier ;
- greylisting.

## 21.3. Milter

Le protocole Milter permet à un service externe d'intervenir à plusieurs étapes SMTP et sur le contenu.

Cas :

- DKIM ;
- DMARC ;
- antispam ;
- antivirus ;
- ajout/suppression d'en-têtes.

## 21.4. `header_checks` et `body_checks`

Postfix propose aussi des contrôles par motifs :

```ini
header_checks = regexp:/etc/postfix/header_checks
```

Ils conviennent à des règles simples et déterministes, mais ne remplacent pas un moteur MIME, un antivirus ou un antispam moderne.

Une expression régulière appliquée au flux brut n'a pas la même compréhension qu'un parseur MIME.

---

# 22. Sécuriser un serveur exposé à Internet

## 22.1. Mettre à jour avant de durcir

En 2026, plusieurs correctifs Postfix ont concerné des dénis de service, des lectures mémoire et des contournements de politique. Une configuration sophistiquée sur une branche non maintenue n'est pas une stratégie de sécurité.

Première ligne de défense :

```bash
sudo apt update
sudo apt upgrade
```

avec une politique de mises à jour adaptée au système de production.

## 22.2. SMTP smuggling et fins de ligne

Les attaques de **SMTP smuggling** exploitent des divergences d'interprétation de fins de ligne et de fin de DATA entre relais.

Les versions modernes incluent des protections, mais nous devons vérifier que notre version a les correctifs appropriés. L'amont documente notamment :

```ini
smtpd_forbid_bare_newline = normalize
```

sur les versions concernées, avec des exclusions éventuelles pour des clients explicitement de confiance.

Sur des versions récentes, plusieurs protections ont évolué vers des défauts plus sûrs. Nous ne recopions pas un workaround 2023 sans vérifier ce que fait notre version 2026.

## 22.3. Ne pas exposer des commandes de confiance

Les mécanismes comme `XCLIENT` ou `XFORWARD` servent dans des architectures de proxy/MTA de confiance. Nous ne les rendons pas accessibles à Internet sans restriction.

## 22.4. `postscreen`

`postscreen` peut filtrer des connexions SMTP sur le **port 25** avant de mobiliser un vrai `smtpd`.

Il peut notamment combiner :

- tests de comportement ;
- listes DNS ;
- cache de clients ;
- limitation de certaines charges automatisées.

Il ne doit pas être placé devant le service de soumission authentifiée : le trafic utilisateur sur 587/465 a un autre modèle de confiance.

## 22.5. DNSBL

Une DNSBL peut être utilisée via des restrictions Postfix, mais une seule liste mal choisie peut provoquer des faux positifs massifs.

Nous devons connaître :

- la politique de la liste ;
- son mécanisme de retrait ;
- le sens de ses codes de réponse ;
- son statut opérationnel ;
- les implications juridiques et métier.

## 22.6. Limites et protections de ressources

Exemples de paramètres à connaître :

```bash
postconf \
  message_size_limit \
  mailbox_size_limit \
  smtpd_client_connection_count_limit \
  smtpd_client_connection_rate_limit \
  smtpd_recipient_limit
```

Nous adaptons les valeurs à la charge réelle. Une limite n'est pas un substitut à un reverse proxy SMTP, un antispam ou une protection réseau lorsque le volume devient hostile.

## 22.7. Principe de moindre confiance

- `mynetworks` minimal ;
- secrets lisibles uniquement par les comptes nécessaires ;
- sockets UNIX privilégiés pour les échanges locaux sensibles ;
- pas de mot de passe dans `main.cf` si une map protégée suffit ;
- pas de service d'administration exposé par inadvertance ;
- firewall cohérent avec les ports réellement nécessaires ;
- surveillance de la queue et des logs.

---

# 23. File d'attente

## 23.1. Afficher la queue

```bash
postqueue -p
```

ou :

```bash
mailq
```

Postfix sait également exposer la queue en JSON :

```bash
postqueue -j
```

Très pratique avec `jq` :

```bash
postqueue -j | jq .
```

## 23.2. Inspecter un message

Si l'ID est `3F4A212345` :

```bash
postcat -q 3F4A212345
```

Nous pouvons y voir :

- enveloppe ;
- en-têtes ;
- contenu ;
- métadonnées de queue.

Ne publions pas ce résultat brut sans retirer adresses, identifiants ou contenu confidentiel.

## 23.3. Forcer une tentative

```bash
sudo postqueue -f
```

Cela demande une nouvelle tentative, mais ne corrige évidemment pas la cause d'un échec récurrent.

## 23.4. Mettre en hold

```bash
sudo postsuper -h 3F4A212345
```

Libérer :

```bash
sudo postsuper -H 3F4A212345
```

## 23.5. Requeue

```bash
sudo postsuper -r 3F4A212345
```

La remise en file fait repasser le message par des étapes de traitement ; utilisons-la consciemment après modification de configuration.

## 23.6. Supprimer

```bash
sudo postsuper -d 3F4A212345
```

> [!danger] Destruction
> Les variantes avec `ALL` peuvent supprimer un très grand nombre de messages. Nous listons et sauvegardons les IDs concernés avant toute opération de masse.

## 23.7. `qshape`

`qshape` permet d'observer l'âge et la distribution des messages par destination. Une queue qui grossit n'est pas seulement un nombre : sa **forme** indique si un domaine particulier, une résolution DNS, une connectivité ou une réputation provoque l'accumulation.

---

# 24. Logs, observabilité et audit

## 24.1. Où sont les logs ?

Selon la distribution :

```bash
journalctl -u postfix
```

et/ou :

```bash
/var/log/mail.log
```

Ne supposons pas que `/var/log/mail.log` existe : certaines installations utilisent surtout journald ou une configuration rsyslog différente.

## 24.2. Suivre en direct

```bash
journalctl -fu postfix
```

ou :

```bash
tail -f /var/log/mail.log
```

## 24.3. Suivre un Queue ID

```bash
grep '3F4A212345' /var/log/mail.log
```

Un même message peut produire plusieurs lignes : réception, cleanup, qmgr, tentative SMTP, délai, succès, suppression de queue.

## 24.4. Audit de base

```bash
postconf -n
postconf -M
postconf -m
postfix check
ss -ltnp | grep -E ':(25|465|587)\b'
postqueue -p
```

## 24.5. Paramètres essentiels à afficher ensemble

```bash
postconf \
  mail_version \
  compatibility_level \
  myhostname \
  mydomain \
  myorigin \
  mydestination \
  mynetworks \
  relay_domains \
  virtual_alias_domains \
  virtual_mailbox_domains \
  relayhost \
  inet_interfaces \
  inet_protocols \
  smtpd_relay_restrictions
```

## 24.6. TLS

```bash
postconf -n | grep -E '(^|_)tls|tls_'
```

Puis tests actifs :

```bash
openssl s_client -starttls smtp -connect mail.example.com:25 -servername mail.example.com
```

## 24.7. DNS

```bash
dig MX example.com +short
dig A mail.example.com +short
dig AAAA mail.example.com +short
dig -x 192.0.2.25 +short
dig TXT example.com +short
dig TXT _dmarc.example.com +short
dig TXT 2026a._domainkey.example.com +short
```

## 24.8. SMTP avec `swaks`

Test minimal :

```bash
swaks --server mail.example.com --to postmaster@example.com
```

Tester STARTTLS :

```bash
swaks --server mail.example.com --tls --to postmaster@example.com
```

Tester uniquement jusqu'à RCPT :

```bash
swaks --server mail.example.com --to postmaster@example.com --quit-after RCPT
```

---

# 25. Méthode de diagnostic

Le dépannage SMTP devient beaucoup plus simple avec une méthode fixe.

## 25.1. Étape 1 — définir le sens du flux

Question :

```text
entrant ? sortant ? soumission ? livraison locale ? relayhost ?
```

Sans cela, nous risquons d'analyser `smtpd_*` alors que le problème est dans le client `smtp(8)`.

## 25.2. Étape 2 — récupérer les faits

```bash
postconf mail_version
postconf -n
postconf -M
postqueue -p
journalctl -u postfix -n 200 --no-pager
```

## 25.3. Étape 3 — repérer le Queue ID

Dans les logs :

```text
postfix/qmgr[...]: 3F4A212345: from=<...>, size=..., nrcpt=1
```

Puis :

```bash
journalctl -u postfix | grep 3F4A212345
```

## 25.4. Étape 4 — lire le code SMTP distant

Exemple :

```text
status=deferred (host mx.example.net[203.0.113.50] said:
451 4.7.1 Try again later)
```

Le texte entre parenthèses contient souvent la cause exploitable.

## 25.5. Étape 5 — vérifier la couche concernée

### DNS

```bash
dig MX destination.example
```

### TCP

```bash
nc -vz mx.destination.example 25
```

### SMTP/TLS

```bash
openssl s_client -starttls smtp -connect mx.destination.example:25 -servername mx.destination.example
```

### Authentification sortante vers smarthost

```bash
postconf relayhost smtp_sasl_auth_enable smtp_sasl_password_maps smtp_tls_security_level
```

### Routage local

```bash
postconf mydestination virtual_mailbox_domains virtual_alias_domains relay_domains transport_maps
```

## 25.6. Étape 6 — corriger puis tester un seul cas

Évitons simultanément :

- de changer DNS ;
- de changer TLS ;
- de changer DKIM ;
- de vider la queue ;
- de modifier les restrictions.

Sinon nous ne saurons pas quelle modification a réellement résolu le problème.

---

# 26. Délivrabilité

La **délivrabilité** n'est pas la même chose que la validité SMTP.

Un message peut être techniquement accepté puis classé en spam. Inversement, un serveur peut être parfaitement configuré mais bloqué pour mauvaise réputation historique de son IP.

## 26.1. Checklist d'identité

- FQDN stable ;
- A/AAAA corrects ;
- PTR cohérent ;
- MX correct ;
- HELO/EHLO cohérent ;
- SPF valide ;
- DKIM valide ;
- DMARC aligné ;
- TLS fonctionnel ;
- `postmaster` et `abuse` opérationnelles.

## 26.2. Réputation

Les fournisseurs évaluent également :

- historique de l'IP ;
- volume et variation du volume ;
- plaintes utilisateurs ;
- taux de destinataires invalides ;
- contenu et URLs ;
- comportement face aux bounces ;
- listes de blocage ;
- réputation du domaine ;
- qualité des consentements pour les envois de masse.

Aucune ligne de `main.cf` ne peut « réparer » instantanément une réputation dégradée.

## 26.3. Vérifier un message réellement reçu

Dans Gmail, Outlook ou un autre fournisseur, inspectons les en-têtes complets.

Nous recherchons notamment :

```text
Authentication-Results:
    spf=pass ...
    dkim=pass ...
    dmarc=pass ...
```

Cela valide la chaîne telle que le destinataire l'a réellement observée.

## 26.4. Attention à IPv6

Si Postfix choisit une sortie IPv6 mais que :

- le PTR IPv6 manque ;
- SPF n'autorise pas l'IPv6 ;
- le réseau IPv6 est instable ;
- la réputation IPv6 est mauvaise ;

nous pouvons obtenir un comportement différent de l'IPv4.

Ne désactivons pas IPv6 comme premier réflexe : corrigeons d'abord l'identité et la connectivité. Si l'infrastructure ne sait réellement pas l'opérer, la politique réseau doit être cohérente avec ce choix.

## 26.5. Forwarding et DMARC

Le transfert de courrier peut casser SPF car le serveur de forwarding devient la nouvelle IP d'envoi. DKIM peut survivre si le message n'est pas modifié de façon incompatible avec la signature.

ARC peut transmettre des résultats d'authentification à travers certains intermédiaires, mais n'est pas un « passe-droit DMARC » universel.

---

# 27. Cas pratique : blocage Microsoft S3140

Exemple de réponse :

```text
550 5.7.1 Unfortunately, messages from [192.0.2.25] weren't sent.
Please contact your Internet service provider since part of their network
is on our block list (S3140).
```

Si le rejet arrive en réponse à `MAIL FROM`, le serveur distant n'a pas encore reçu le contenu du message transmis après `DATA`.

Il n'a donc pas encore pu évaluer la signature DKIM de ce message.

Cela ne veut pas dire que DKIM est inutile ; cela veut dire que **la cause immédiate de ce rejet précis se situe plus tôt dans la transaction**.

Ordre de diagnostic :

1. vérifier qu'il n'y a pas de compromission ou d'open relay ;
2. vérifier l'IP réellement utilisée ;
3. vérifier PTR, A/AAAA et HELO ;
4. vérifier SPF ;
5. vérifier DKIM et DMARC avec un fournisseur qui accepte le mail ;
6. vérifier la réputation et les outils Microsoft ;
7. suivre la procédure Microsoft de Sender Support lorsque nécessaire ;
8. contacter l'hébergeur si le blocage vise un bloc réseau dont nous ne contrôlons pas la réputation globale.

Microsoft SNDS est utile pour les IP que nous administrons :

```text
https://substrate.office.com/ip-domain-management-snds/SNDS
```

Les URLs des portails fournisseurs changent parfois ; si un ancien lien redirige ou disparaît, recherchons le portail officiel actuel plutôt que d'utiliser un clone tiers.

---

# 28. Architectures types

## 28.1. Machine locale qui envoie via un smarthost

Objectif : aucune réception Internet.

```text
application -> Postfix local -> smtp.provider.example:587 -> Internet
```

Principes :

```ini
inet_interfaces = loopback-only
mydestination =
relayhost = [smtp.provider.example]:587
smtp_sasl_auth_enable = yes
smtp_tls_security_level = encrypt
```

Très adapté à une application qui veut une queue locale fiable sans exploiter un MTA public complet.

## 28.2. MTA public de réception

```text
Internet :25 -> Postfix -> filtre -> Dovecot LMTP -> boîtes
```

À ajouter :

- MX/PTR ;
- destinataires connus ;
- TLS ;
- antispam ;
- SPF/DKIM/DMARC entrant selon politique ;
- monitoring de queue.

## 28.3. Serveur complet utilisateurs

```text
                    +--> Dovecot IMAP :993 --> MUA
                    |
Internet :25 -> Postfix -> LMTP -> stockage
                    ^
                    |
MUA :587/465 -> Postfix submission + SASL
```

La réception entre MTA et la soumission utilisateur restent deux services distincts.

## 28.4. Relay interne

```text
applications internes -> Postfix relay -> Internet / smarthost
```

Ici, `mynetworks` peut autoriser des subnets internes **si et seulement si** la frontière réseau est réellement maîtrisée. Sur un cloud ou un réseau partagé, l'authentification explicite est souvent préférable.

## 28.5. Multi-domaines virtuels

```text
Internet
  |
  v
Postfix
  |  virtual_mailbox_domains
  |  virtual_mailbox_maps -> SQL/LDAP
  v
Dovecot LMTP
  |
  v
stockage de boîtes virtuelles
```

Cette architecture évite un compte UNIX par boîte.

---

# 29. Laboratoires pratiques

## TP 1 — Identifier une installation

Objectif : ne rien modifier.

```bash
postconf mail_version
postconf compatibility_level
postconf -n
postconf -M
postconf -m
postconf -a
postconf -A
postqueue -p
ss -ltnp | grep -E ':(25|465|587)\b'
```

Questions :

1. quel service écoute sur 25 ?
2. existe-t-il un service 587 ?
3. quels domaines sont locaux ?
4. un relayhost est-il configuré ?
5. quel backend de map est disponible ?
6. SASL Dovecot est-il supporté ?

## TP 2 — Serveur local uniquement

Installons Postfix sur une VM de laboratoire et limitons l'écoute :

```ini
inet_interfaces = loopback-only
mydestination = $myhostname, localhost.$mydomain, localhost
```

Puis :

```bash
sudo postfix check
sudo postfix reload
ss -ltnp | grep ':25'
```

Injectons localement :

```bash
printf 'Sujet de test\n' | mail -s 'Postfix local' root
```

Observons la queue et les logs.

## TP 3 — Tester le protocole

Avec `swaks` :

```bash
swaks --server 127.0.0.1 --to root@localhost
```

Puis détaillons dans les logs :

```bash
journalctl -u postfix -n 100 --no-pager
```

Repérons le Queue ID.

## TP 4 — Créer un alias

```text
# /etc/aliases
postmaster: root
abuse: root
```

```bash
sudo newaliases
```

Test :

```bash
swaks --server 127.0.0.1 --to postmaster@localhost
```

## TP 5 — Table virtuelle

Choisissons un backend réellement disponible :

```bash
postconf -m | grep -E '^(lmdb|hash)$'
```

Exemple LMDB :

```text
# /etc/postfix/virtual
contact@example.test alice@localhost
```

```bash
sudo postmap lmdb:/etc/postfix/virtual
postmap -q contact@example.test lmdb:/etc/postfix/virtual
```

Nous n'activons pas le domaine en production dans ce TP ; le but est de comprendre le fonctionnement d'une map.

## TP 6 — Audit DNS d'un domaine réel

```bash
dig MX example.com +short
dig A mail.example.com +short
dig AAAA mail.example.com +short
dig -x IP +short
dig TXT example.com +short
dig TXT _dmarc.example.com +short
```

Construisons ensuite un tableau :

| Élément | Attendu | Observé | OK ? |
|---|---|---|---|
| MX | mail.example.com | ... | ... |
| A | IP publique | ... | ... |
| PTR | mail.example.com | ... | ... |
| SPF | source autorisée | ... | ... |
| DKIM | selector présent | ... | ... |
| DMARC | politique publiée | ... | ... |

## TP 7 — Audit TLS

```bash
openssl s_client \
  -starttls smtp \
  -connect mail.example.com:25 \
  -servername mail.example.com
```

Questions :

- le certificat est-il expiré ?
- le SAN contient-il le FQDN ?
- la chaîne est-elle valide ?
- STARTTLS est-il annoncé ?

## TP 8 — Open relay

Depuis une IP qui n'est pas dans `mynetworks` :

```bash
swaks \
  --server mail.example.com \
  --from outsider@example.net \
  --to thirdparty@example.org \
  --quit-after RCPT
```

Le relais non authentifié doit être refusé.

## TP 9 — Queue deferred

Sur un laboratoire, provoquons une destination temporairement injoignable puis :

```bash
postqueue -p
postqueue -j | jq .
postcat -q QUEUE_ID
```

Après correction :

```bash
sudo postqueue -f
```

Observons la transition jusqu'à `status=sent`.

## TP 10 — Soumission authentifiée

Objectif : obtenir :

```text
MUA -> TLS -> AUTH -> Postfix :587 -> queue -> sortie SMTP
```

Vérifions successivement :

1. socket Dovecot SASL ;
2. `postconf -a` ;
3. service `submission` dans `master.cf` ;
4. certificat ;
5. authentification ;
6. autorisation de relais ;
7. identité d'expéditeur ;
8. DKIM sur le message sortant.

---

# 30. Checklist de mise en production

## 30.1. Système

- [ ] OS maintenu ;
- [ ] Postfix sur une branche ou un paquet encore supporté ;
- [ ] mises à jour de sécurité appliquées ;
- [ ] heure système synchronisée ;
- [ ] firewall documenté ;
- [ ] sauvegarde de `/etc/postfix` et des secrets ;
- [ ] procédure de restauration testée.

## 30.2. Identité

- [ ] `myhostname` = FQDN public cohérent ;
- [ ] A/AAAA corrects ;
- [ ] PTR correct ;
- [ ] MX correct ;
- [ ] HELO/EHLO cohérent ;
- [ ] IPv6 opérationnelle ou non publiée.

## 30.3. Relais

- [ ] `mynetworks` minimal ;
- [ ] `smtpd_relay_restrictions` explicite ;
- [ ] test open relay depuis l'extérieur ;
- [ ] destinataires inconnus refusés proprement ;
- [ ] pas de catch-all involontaire.

## 30.4. Submission

- [ ] port 587 et/ou 465 distinct du 25 ;
- [ ] TLS imposé ;
- [ ] SASL actif ;
- [ ] secrets protégés ;
- [ ] limitation de l'identité d'expéditeur si nécessaire ;
- [ ] logs séparables via `syslog_name`.

## 30.5. TLS

- [ ] certificat valide ;
- [ ] renouvellement automatique surveillé ;
- [ ] STARTTLS testé sur 25 ;
- [ ] TLS obligatoire sur submission ;
- [ ] `smtp_tls_security_level` cohérent avec direct-MX ou smarthost ;
- [ ] politiques DANE/MTA-STS testées si utilisées.

## 30.6. Authentification du domaine

- [ ] SPF unique et valide ;
- [ ] nombre de lookups SPF maîtrisé ;
- [ ] DKIM 2048 bits ou politique cryptographique actuelle ;
- [ ] clé privée protégée ;
- [ ] selector documenté et rotation prévue ;
- [ ] DMARC déployé progressivement ;
- [ ] rapports DMARC réellement lus.

## 30.7. Exploitation

- [ ] `postmaster` fonctionne ;
- [ ] `abuse` fonctionne ;
- [ ] monitoring de la queue ;
- [ ] alertes sur volume deferred ;
- [ ] logs centralisés ou conservés suffisamment ;
- [ ] test périodique de délivrabilité ;
- [ ] procédure en cas de compromission ;
- [ ] procédure de rotation des secrets et clés.

---

# Commandes à retenir

## Configuration

```bash
postconf -n
postconf -d nom_parametre
postconf nom_parametre
postconf -M
postconf -m
postconf -a
postconf -A
postfix check
postfix reload
```

## Service

```bash
systemctl status postfix --no-pager
journalctl -u postfix -n 100 --no-pager
journalctl -fu postfix
ss -ltnp | grep -E ':(25|465|587)\b'
```

## Queue

```bash
postqueue -p
postqueue -j
postcat -q QUEUE_ID
postqueue -f
postsuper -h QUEUE_ID
postsuper -H QUEUE_ID
postsuper -r QUEUE_ID
postsuper -d QUEUE_ID
```

## Maps

```bash
postmap lmdb:/etc/postfix/virtual
postmap -q cle lmdb:/etc/postfix/virtual
newaliases
```

## DNS

```bash
dig MX example.com +short
dig A mail.example.com +short
dig AAAA mail.example.com +short
dig -x IP +short
dig TXT example.com +short
dig TXT _dmarc.example.com +short
dig TXT SELECTOR._domainkey.example.com +short
```

## SMTP/TLS

```bash
swaks --server mail.example.com --to postmaster@example.com
openssl s_client -starttls smtp -connect mail.example.com:25 -servername mail.example.com
```

---

# Erreurs classiques

## « Relay access denied »

Causes possibles :

- nous essayons réellement de relayer sans autorisation ;
- le domaine local est dans la mauvaise classe ;
- `virtual_mailbox_domains` ou `relay_domains` manque ;
- l'utilisateur n'est pas authentifié ;
- une map ne contient pas le domaine/destinataire attendu.

## « Mail for ... loops back to myself »

Le serveur considère le domaine ou next-hop comme lui-même mais ne sait pas comment terminer correctement la livraison.

Auditons :

```bash
postconf mydestination virtual_alias_domains virtual_mailbox_domains relay_domains transport_maps relayhost
```

## « SASL authentication failed »

Vérifions :

```bash
postconf -a
postconf smtpd_sasl_type smtpd_sasl_path
ls -l /var/spool/postfix/private/auth
journalctl -u dovecot -n 100 --no-pager
journalctl -u postfix -n 100 --no-pager
```

## « Connection timed out » vers port 25

Possibilités :

- firewall local ;
- security group cloud ;
- opérateur qui bloque 25 ;
- fournisseur cloud qui interdit SMTP sortant ;
- routage IPv6 cassé ;
- destination réellement indisponible.

Testons séparément DNS, IPv4, IPv6 et TCP.

## DKIM `key not secure`

Dans certains outils OpenDKIM, la mention « secure » peut concerner la validation DNSSEC de la réponse, et non la longueur cryptographique de la clé elle-même.

Lisons tout le résultat et vérifions surtout :

```text
key OK
```

ainsi que la politique DNSSEC si nous voulons précisément bénéficier de DNSSEC.

## SPF PASS mais DMARC FAIL

SPF peut réussir sur :

```text
MAIL FROM:<bounce@mailer.vendor.example>
```

alors que le champ visible est :

```text
From: service@example.com
```

Sans alignement, DMARC peut échouer malgré `spf=pass`.

---

# Références

## Documentation Postfix officielle

- Documentation principale : <https://www.postfix.org/documentation.html>
- Paramètres `postconf(5)` : <https://www.postfix.org/postconf.5.html>
- Commande `postconf(1)` : <https://www.postfix.org/postconf.1.html>
- Configuration de base : <https://www.postfix.org/BASIC_CONFIGURATION_README.html>
- SASL : <https://www.postfix.org/SASL_README.html>
- TLS : <https://www.postfix.org/TLS_README.html>
- Milter : <https://www.postfix.org/MILTER_README.html>
- Policy delegation : <https://www.postfix.org/SMTPD_POLICY_README.html>
- Domaines virtuels : <https://www.postfix.org/VIRTUAL_README.html>
- Réécriture et classes d'adresses : <https://www.postfix.org/ADDRESS_REWRITING_README.html>
- Debugging : <https://www.postfix.org/DEBUG_README.html>
- SMTP smuggling : <https://www.postfix.org/smtp-smuggling.html>
- Migration hors Berkeley DB : <https://www.postfix.org/NON_BERKELEYDB_README.html>
- Annonces et versions : <https://www.postfix.org/announcements.html>

## Standards essentiels

- RFC 5321 — SMTP
- RFC 6409 — Message Submission
- RFC 8314 — TLS pour submission et accès
- RFC 7208 — SPF
- RFC 6376 — DKIM
- RFC 7489 — DMARC
- RFC 7672 — DANE pour SMTP
- RFC 8461 — MTA-STS
- RFC 8460 — SMTP TLS Reporting
- RFC 8689 — REQUIRETLS

---

# Conclusion

Postfix devient beaucoup plus simple à administrer lorsque nous raisonnons par couches :

```text
identité DNS
    -> réception ou submission SMTP
        -> politique de relais
            -> queue
                -> routage
                    -> transport SMTP ou livraison LMTP/local
                        -> authentification du domaine
                            -> réputation et délivrabilité
```

Les réflexes les plus importants sont :

1. **savoir quel processus Postfix travaille** (`smtpd`, `smtp`, `qmgr`, `local`, `lmtp`, etc.) ;
2. **savoir dans quelle classe se trouve le domaine** ;
3. **ne jamais confondre port 25 et submission** ;
4. **auditer la configuration effective avec `postconf` plutôt que lire seulement `main.cf`** ;
5. **suivre les Queue IDs dans les logs** ;
6. **maintenir Postfix et sa distribution à jour** ;
7. **considérer SPF, DKIM, DMARC et la réputation comme des couches complémentaires, pas comme une unique option « anti-spam »**.

Avec cette méthode, une panne de messagerie cesse d'être une suite d'essais aléatoires : elle devient un problème de routage, de politique, de protocole ou d'identité que nous pouvons isoler étape par étape.

# Annexe — Configuration de référence : SPF, OpenDKIM multi-domaines, DMARC et audit (ancien cours, 2023-2026)

> [!info] Annexe
> Sections opérationnelles de la version précédente du cours, écrites pour le serveur delfeno (domaines logikascium.com et multiscium.com), conservées telles quelles lors de la refonte du 31 août 2026.

## Configuration de SPF

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

## Configuration de OpenDKIM

OpenDKIM signe les messages sortants en ajoutant un en-tête `DKIM-Signature`. Le serveur destinataire récupère ensuite la clé publique dans le DNS afin de vérifier cette signature.

### Cas à un seul domaine

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

#### Relier Postfix à OpenDKIM

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

### Configurer plusieurs domaines ou sous-domaines dans DKIM

Pour que Postfix et OpenDKIM signent également les e-mails envoyés depuis `@access.logikascium.com` et `@alirpunkto.logikascium.com`, configurons OpenDKIM avec une `KeyTable` et une `SigningTable`.

#### Génération des clés DKIM pour chaque domaine

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

#### Publication des clés publiques dans le DNS

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

#### Configuration d'OpenDKIM pour gérer plusieurs domaines

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

#### Création des fichiers KeyTable et SigningTable

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

#### Vérification des permissions

Assurons-nous que les fichiers de configuration sont lisibles par l'utilisateur `opendkim` :

```bash
sudo chown opendkim:opendkim /etc/opendkim/KeyTable /etc/opendkim/SigningTable
sudo chmod 640 /etc/opendkim/KeyTable /etc/opendkim/SigningTable
```

#### Redémarrage d'OpenDKIM et de Postfix

Après avoir effectué les modifications, redémarrons les services pour appliquer la nouvelle configuration :

```bash
sudo systemctl restart opendkim
sudo systemctl restart postfix
```

#### Vérification d'OpenDKIM

Vérifions que OpenDKIM est actif et écoute sur le port configuré (par exemple, 8891) :

```bash
sudo systemctl status opendkim
sudo ss -ltnp | grep -E '8891|opendkim'
```

#### Test de l'envoi d'e-mails depuis les nouveaux domaines

Envoyons des e-mails depuis les adresses `@access.logikascium.com` et `@alirpunkto.logikascium.com` vers une adresse externe (par exemple, Gmail) pour vérifier que les e-mails sont correctement signés.

**Vérifions les en-têtes de l'e-mail reçu :**

- Rechercher le champ `DKIM-Signature`.
- Assurons-nous que le domaine (`d=`) correspond à l'adresse d'expéditeur.
- Vérifions que le sélecteur (`s=`) est correct.

#### Vérification des enregistrements DNS DKIM

Assurons-nous que les enregistrements DNS pour les clés publiques sont correctement publiés et accessibles.

**Utilisons la commande `dig` pour vérifier :**

```bash
dig TXT dkim._domainkey.access.logikascium.com +short
dig TXT dkim._domainkey.alirpunkto.logikascium.com +short
```

#### Configuration de SPF et DMARC

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

#### Surveillance des logs pour détecter d'éventuelles erreurs

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

#### Vérifier la configuration de Postfix pour les multiples domaines

Assurons-nous que Postfix est configuré pour gérer l'envoi d'e-mails depuis ces domaines. Vérifions les paramètres suivants dans `/etc/postfix/main.cf` :

- **mydestination** : Doit inclure uniquement les domaines pour lesquels ce serveur est une destination finale. Le simple fait de signer ou d'envoyer depuis un domaine ne justifie pas de l'ajouter ici.

```ini
mydestination = $myhostname, logikascium.com, localhost.localdomain, localhost
```

- **relay_domains** : Si nous relayons des e-mails pour ces domaines.


#### Points supplémentaires

- **Synchronisation de l'heure** : Assurons-nous que notre serveur a l'heure correcte. Utilisons `systemd-timesyncd`, `chrony` ou un autre service NTP si nécessaire.
- **Sécurité** : Assurons-nous que nos clés privées sont sécurisées et que seules les personnes autorisées y ont accès.
- **Documentation** : Conservons une documentation de notre configuration pour faciliter la maintenance future.


### Visualisation d'une clé DKIM

Pour visualiser les informations d'une clé DKIM à partir du fichier de la clé privée, nous pouvons extraire la clé publique correspondante. La clé publique est publiée dans les enregistrements DNS pour permettre aux serveurs de réception de vérifier les signatures DKIM des e-mails.

#### Vérifier que OpenSSL est installé

Vérifions qu'OpenSSL est installé sur le système :

```bash
openssl version
```

Si OpenSSL n'est pas installé, alors l'installer avec :

```bash
sudo apt-get install openssl
```

---

#### Extraire la clé publique de la clé privée

Supposons que notre fichier de clé privée s'appelle `dkim.private` et est situé dans `/etc/dkimkeys/`.

Exécutons la commande suivante pour extraire la clé publique :

```bash
openssl rsa -in /etc/dkimkeys/dkim.private -pubout -out /tmp/dkim_public.pem
```

- **Explication :**
  - `-in /etc/dkimkeys/dkim.private` : spécifie le fichier de clé privée en entrée.
  - `-pubout` : indique à OpenSSL d'extraire la clé publique.
  - `-out /tmp/dkim_public.pem` : spécifie le fichier de sortie pour la clé publique.

#### Afficher la clé publique

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

#### Préparer la clé publique pour l'enregistrement DNS DKIM

Les enregistrements DNS DKIM nécessitent que la clé publique soit sur une seule ligne, sans les en-têtes et pieds de page. Pour cela utilisons la commande suivante :

```bash
openssl rsa -in /etc/dkimkeys/dkim.private -pubout -outform PEM | grep -v '^-.*KEY-----' | tr -d '\n'
```

- **Explication :**
  - `grep -v '^-.*KEY-----'` : supprime les lignes contenant les en-têtes et pieds de page.
  - `tr -d '\n'` : supprime les sauts de ligne pour obtenir une clé sur une seule ligne.

#### Construire l'enregistrement DNS DKIM

Avec la clé publique préparée, nous allons ajouter un enregistrement TXT DKIM :

```
v=DKIM1; k=rsa; p=VOTRE_CLÉ_PUBLIQUE
```

- Remplacez `VOTRE_CLÉ_PUBLIQUE` par la clé publique obtenue à l'étape précédente.

#### Vérifier les détails de la clé

Pour voir plus de détails sur la clé (par exemple, le modulus et l'exposant public), exécutons :

```bash
openssl rsa -in /etc/dkimkeys/dkim.private -text -noout
```


##### Vérification de l'enregistrement DKIM dans le DNS

Après avoir ajouté l'enregistrement DNS :

```bash
dig TXT dkim._domainkey.logikascium.com +short
```

##### Vérification finale

Après avoir configuré l'enregistrement DNS et redémarré les services si nécessaire, envoyons un e-mail de test à une adresse externe (par exemple, Gmail) et vérifions les en-têtes pour nous assurer que le champ `DKIM-Signature` est présent (sur gmail afficher l'original du mail).

Nous pouvons aussi utiliser des outils en ligne pour tester la configuration DKIM, comme :

- **Mail Tester** : [www.mail-tester.com](https://www.mail-tester.com/)
- **DKIM Core Validator** : [dkimcore.org/tools/keycheck.html](https://dkimcore.org/tools/keycheck.html)

#### DMARC

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

## Audit et diagnostic d'un serveur Postfix

Lorsqu'un serveur fonctionne mais qu'un fournisseur refuse nos messages, il faut distinguer :

- une erreur de configuration Postfix ;
- une erreur DNS ;
- un échec SPF/DKIM/DMARC ;
- un problème TLS ;
- un compte compromis ou un open relay ;
- une mauvaise réputation de l'IP ou de sa plage réseau.

Voici une procédure d'audit reproductible.

### Vérifier la configuration Postfix active

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

### Vérifier les services et les ports

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

### Vérifier la file d'attente

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

### Auditer les envois dans les logs

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

### Vérifier le DNS direct et le reverse DNS

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

#### Attention au résolveur local

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

### Vérifier les MX

```bash
dig @1.1.1.1 logikascium.com MX +short
```

Puis vérifions l'adresse de chaque MX :

```bash
dig @1.1.1.1 delfeno.logikascium.com A +short
```

Il est inutile de publier plusieurs MX de priorités différentes s'ils pointent exactement vers la même machine : cela ne fournit aucune redondance réelle.

### Vérifier SPF

```bash
dig @1.1.1.1 TXT logikascium.com +short
```

Exemple :

```text
"v=spf1 a mx -all"
```

Dans ce cas, il faut aussi vérifier que les mécanismes `a` et `mx` aboutissent réellement à l'IP d'envoi.

### Vérifier DMARC

```bash
dig @1.1.1.1 TXT _dmarc.logikascium.com +short
```

Exemple :

```text
v=DMARC1; p=reject; rua=mailto:dmarc@logikascium.com; aspf=s; adkim=s;
```

### Auditer DKIM et OpenDKIM

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

### Vérifier le certificat SMTP entrant

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

### Vérification finale avec un vrai destinataire

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

### Comprendre à quelle étape SMTP se produit le rejet

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

### Cas pratique : blocage Microsoft S3140

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

## Adresses techniques obligatoires ou recommandées

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

## Références utiles pour l'audit

- Postfix Milter : https://www.postfix.org/MILTER_README.html
- Paramètres Postfix : https://www.postfix.org/postconf.5.html
- Microsoft SNDS : https://substrate.office.com/ip-domain-management-snds/snds
- OpenDKIM testkey : https://manpages.ubuntu.com/manpages/noble/man8/opendkim-testkey.8.html


## Installer Dovecot

Dovecot est principalement un serveur IMAP et POP3. Il peut également assurer la livraison locale via LDA/LMTP.
Nous avons configuré Postfix pour utiliser le format Maildir et allons voir comment configurer Dovecot pour ce format.
### Installation de Dovecot

Installons Dovecot avec la commande :

```bash
sudo apt-get install dovecot-imapd
```

### Configuration de Dovecot avec Maildir

Selon la version et l'organisation de la configuration Dovecot, ce paramètre se trouve souvent dans `/etc/dovecot/conf.d/10-mail.conf`. Assurons-nous que la configuration contient :

```txt
mail_location = maildir:~/Maildir
```

Cela indique à Dovecot d'utiliser le format Maildir pour le stockage des e-mails.

## Tester la configuration

Nous pouvons utiliser un service de test externe comme [appmaildev.com](https://appmaildev.com), puis envoyer un message vers l'adresse temporaire fournie :
```bash
echo test | mail -s "Test postfix" -aFROM:michaellaunay@ecreall.com test-1fc42c52@appmaildev.com
```


## Créer un serveur postfix de test

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
