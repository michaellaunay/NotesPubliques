---
schema_version: 1
uid: 01M02EX5ASSYXAGZVA14CDPYP3
titre: "Apache HTTP Server"
aliases:
  - Apache
  - httpd
  - Apache2
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - administration-systeme
  - web
  - apache
  - http
  - tls
  - reverse-proxy
resume: "Cours pratique sur Apache HTTP Server 2.4 : architecture, installation Debian/Ubuntu, VirtualHost, autorisation, TLS/ACME, HTTP/2, PHP-FPM, reverse proxy, journalisation, performances, sécurité et dépannage."
niveau: intermediaire
prerequis:
  - "[[GNULinux]]"
  - "[[Initialisation système et des services]]"
  - "[[Sécurité avancée sous Linux]]"
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-06-02
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---
# Apache HTTP Server

> [!abstract] Objectif
> Comprendre suffisamment Apache HTTP Server pour **concevoir, déployer, sécuriser et diagnostiquer** un serveur web moderne. Le but n'est pas de mémoriser une collection de directives, mais de savoir où s'applique chaque configuration, comment vérifier son effet et comment éviter les erreurs historiques encore très présentes dans les anciens tutoriels.

Voir aussi : [[GNULinux]], [[Initialisation système et des services]], [[Sécurité avancée sous Linux]], [[Docker]], [[Postfix]].

> [!important] État en août 2026
> La branche stable amont est **Apache HTTP Server 2.4**. La version amont recommandée au 31 août 2026 est **2.4.68**, publiée le 8 juin 2026. Sur Debian ou Ubuntu, on installe normalement le paquet maintenu par la distribution plutôt que de compiler la dernière version amont : les distributions peuvent rétroporter des correctifs de sécurité sans changer le numéro de version de la même manière qu'en amont.

# 1. Apache, Apache HTTP Server et `httpd`

Le projet officiel s'appelle **Apache HTTP Server**. Son exécutable est historiquement nommé `httpd` sur de nombreuses distributions.

Sur Debian et Ubuntu, le paquet et le service utilisent principalement le nom :

```text
apache2
```

On rencontre donc les formulations suivantes :

```text
Apache HTTP Server   nom du logiciel
httpd                nom générique/historique du démon
apache2              paquet, binaire et service Debian/Ubuntu
```

Apache est avant tout un serveur HTTP extensible par modules. Il peut notamment :

- servir des fichiers statiques ;
- terminer TLS pour HTTPS ;
- héberger plusieurs sites avec des VirtualHost ;
- agir comme reverse proxy ;
- répartir des requêtes entre plusieurs backends ;
- servir des applications FastCGI, WSGI ou CGI ;
- appliquer de l'authentification et de l'autorisation ;
- compresser et mettre en cache des réponses ;
- produire des journaux d'accès et d'erreur détaillés ;
- exposer HTTP/2 avec `mod_http2`.

Apache n'est pas nécessairement le meilleur choix dans tous les projets. Nginx, Caddy, HAProxy, Envoy ou un reverse proxy fourni par une plate-forme cloud peuvent être plus adaptés selon les objectifs. Le bon critère est l'architecture à construire, pas la popularité de l'outil.

# 2. Modèle mental d'une requête

Une requête simplifiée suit une chaîne de traitement de ce type :

```text
client
  │
  │ TCP 80/443
  ▼
Apache
  │
  ├── choix IP/port
  ├── choix du VirtualHost
  ├── TLS éventuel
  ├── traduction URL → ressource/backend
  ├── authentification
  ├── autorisation
  ├── filtres / réécriture / en-têtes
  ├── contenu local ou proxy
  └── journalisation
  │
  ▼
réponse HTTP
```

Pour diagnostiquer un problème, il faut savoir **à quelle étape** il apparaît.

Exemples :

- erreur DNS : Apache n'est peut-être jamais atteint ;
- erreur TLS : le VirtualHost HTTPS ou le certificat est en cause ;
- `403` : l'autorisation ou les permissions du système de fichiers sont probables ;
- `404` : le mapping URL/chemin ou l'application peut être en cause ;
- `502`/`503` : le backend d'un reverse proxy est souvent indisponible ;
- `500` : la configuration, un module ou l'application a échoué.

# 3. Version installée et version amont

Afficher la version du binaire :

```bash
apache2 -v
```

Ou :

```bash
apache2ctl -v
```

Afficher les paramètres de compilation :

```bash
apache2ctl -V
```

Afficher les modules chargés :

```bash
apache2ctl -M
```

> [!warning] Ne pas comparer naïvement des numéros de version
> Un paquet Ubuntu ou Debian peut conserver un numéro de branche donné tout en recevant des correctifs rétroportés. Pour savoir si une vulnérabilité est corrigée, consulter les avis de sécurité et le changelog du paquet de la distribution, pas seulement la page de version amont.

Informations Debian/Ubuntu :

```bash
dpkg -l apache2 apache2-bin
apt policy apache2
apt changelog apache2
```

# 4. Installation sur Debian et Ubuntu

Mettre à jour l'index des paquets :

```bash
sudo apt update
```

Installer Apache :

```bash
sudo apt install apache2
```

Vérifier le service :

```bash
systemctl status apache2 --no-pager
```

Vérifier l'écoute réseau :

```bash
sudo ss -ltnp | grep -E ':(80|443)\b'
```

Tester localement :

```bash
curl -I http://127.0.0.1/
```

Les ports 80/TCP et 443/TCP doivent également être autorisés par le pare-feu lorsqu'ils doivent être accessibles depuis le réseau.

# 5. Arborescence Debian/Ubuntu

La configuration Debian/Ubuntu est organisée en fragments :

```text
/etc/apache2/
├── apache2.conf
├── envvars
├── ports.conf
├── conf-available/
├── conf-enabled/
├── mods-available/
├── mods-enabled/
├── sites-available/
└── sites-enabled/
```

Rôle principal :

| Emplacement | Rôle |
|---|---|
| `/etc/apache2/apache2.conf` | configuration globale principale |
| `/etc/apache2/ports.conf` | ports d'écoute |
| `sites-available/` | VirtualHost disponibles |
| `sites-enabled/` | VirtualHost activés par liens symboliques |
| `mods-available/` | modules disponibles et leur configuration |
| `mods-enabled/` | modules activés |
| `conf-available/` | fragments globaux disponibles |
| `conf-enabled/` | fragments globaux activés |

Cette organisation est **spécifique aux paquets Debian/Ubuntu**. Une documentation amont utilisant `httpd.conf` ou une distribution RPM peut présenter une arborescence différente.

# 6. Commandes Debian/Ubuntu à connaître

Activer un site :

```bash
sudo a2ensite example.com.conf
```

Désactiver un site :

```bash
sudo a2dissite example.com.conf
```

Activer un module :

```bash
sudo a2enmod ssl
```

Désactiver un module :

```bash
sudo a2dismod autoindex
```

Activer une configuration globale :

```bash
sudo a2enconf security
```

Désactiver une configuration :

```bash
sudo a2disconf nom
```

Ces commandes manipulent principalement des liens symboliques entre les répertoires `*-available` et `*-enabled`.

# 7. Toujours tester avant de recharger

Tester la syntaxe :

```bash
sudo apache2ctl configtest
```

Résultat attendu :

```text
Syntax OK
```

Lister les VirtualHost compris par Apache :

```bash
sudo apache2ctl -S
```

Lister les modules :

```bash
sudo apache2ctl -M
```

Puis effectuer un rechargement gracieux :

```bash
sudo systemctl reload apache2
```

Ou :

```bash
sudo apache2ctl graceful
```

> [!tip] Workflow d'administration
> Préférer systématiquement : **modifier → `configtest` → inspecter le diff → reload → tester → lire les logs**.

Un `restart` arrête puis redémarre le service. Un `reload`/`graceful` cherche à appliquer la configuration sans interrompre brutalement les connexions en cours.

# 8. Ports et directive `Listen`

Le fichier Debian/Ubuntu habituel est :

```text
/etc/apache2/ports.conf
```

Exemple :

```apache
Listen 80

<IfModule ssl_module>
    Listen 443
</IfModule>
```

`Listen` indique **où Apache accepte des connexions**. Un bloc `<VirtualHost *:443>` n'ouvre pas à lui seul le port 443 si aucun `Listen 443` n'existe.

Vérifier :

```bash
sudo ss -ltnp
```

# 9. VirtualHost : héberger plusieurs sites

Apache peut sélectionner un site en fonction du couple adresse/port puis du nom HTTP demandé.

Exemple minimal :

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    DocumentRoot /srv/www/example.com/public

    ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined
</VirtualHost>
```

Un `ServerName` explicite est fortement recommandé.

Le premier VirtualHost chargé pour une adresse/port sert généralement de repli lorsqu'aucun nom ne correspond. Sur Debian/Ubuntu, le site `000-default.conf` est nommé ainsi pour être chargé tôt.

Inspecter l'ordre réel :

```bash
sudo apache2ctl -S
```

# 10. Résolution de nom pour un laboratoire local

Pour un test sans DNS public, utiliser une adresse de documentation, par exemple :

```text
192.0.2.10 example.test
```

Dans `/etc/hosts` du client :

```text
192.0.2.10 example.test
```

Puis tester :

```bash
curl -v http://example.test/
```

Pour tester un VirtualHost sans modifier `/etc/hosts` :

```bash
curl -v -H 'Host: example.test' http://192.0.2.10/
```

Pour HTTPS, le nom participe également à SNI et à la validation du certificat ; un simple en-tête `Host` ne suffit donc pas toujours. `curl --resolve` est souvent plus utile :

```bash
curl --resolve example.com:443:192.0.2.10 https://example.com/
```

# 11. Racine documentaire et permissions

Créer une arborescence :

```bash
sudo install -d -o root -g www-data -m 0750 /srv/www/example.com
sudo install -d -o root -g www-data -m 0750 /srv/www/example.com/public
```

Créer un fichier :

```bash
printf '%s\n' '<h1>example.com</h1>' | \
  sudo tee /srv/www/example.com/public/index.html >/dev/null
```

Le démon doit pouvoir **traverser** tous les répertoires parents et lire les fichiers servis.

Examiner chaque composant :

```bash
namei -l /srv/www/example.com/public/index.html
```

> [!warning] Éviter `chmod -R 777`
> Une erreur `403` ne se corrige pas en donnant des droits d'écriture universels. Identifier précisément le compte du processus, les permissions POSIX, ACL éventuelles et les mécanismes de confinement.

# 12. Bloc `<Directory>` et autorisation Apache 2.4

Sur Apache 2.4 :

```apache
<Directory /srv/www/example.com/public>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

`Require all granted` autorise l'accès à cette ressource au niveau Apache.

L'ancienne syntaxe :

```apache
Order allow,deny
Allow from all
```

appartient au modèle Apache 2.2. Le module `mod_access_compat` peut conserver une compatibilité dans certains environnements, mais **la configuration moderne doit utiliser `Require`**.

Exemples :

```apache
Require all granted
```

```apache
Require all denied
```

```apache
Require ip 192.0.2.0/24
```

Plusieurs conditions :

```apache
<RequireAny>
    Require ip 192.0.2.0/24
    Require ip 2001:db8::/32
</RequireAny>
```

# 13. `Options` : ne pas activer plus que nécessaire

Exemple raisonnable pour un répertoire statique :

```apache
Options -Indexes +FollowSymLinks
```

`Indexes` permet la génération automatique d'une liste de fichiers lorsqu'aucun index n'est trouvé. Cela peut divulguer des fichiers inattendus et ne doit être activé que volontairement.

Désactiver explicitement :

```apache
Options -Indexes
```

`ExecCGI` ne doit être activé que là où l'exécution CGI est réellement nécessaire.

# 14. `.htaccess` : utile mais pas par défaut

`AllowOverride` détermine si Apache recherche et accepte des fichiers `.htaccess`.

Pour un serveur que l'on administre soi-même :

```apache
AllowOverride None
```

est généralement préférable :

- configuration centralisée ;
- moins de lectures de fichiers dans l'arborescence ;
- revue plus simple ;
- moins de capacités déléguées à un utilisateur du contenu.

Si une application impose `.htaccess`, limiter les catégories autorisées plutôt que d'utiliser aveuglément :

```apache
AllowOverride All
```

Consulter la documentation de l'application et d'Apache pour choisir précisément les overrides nécessaires.

# 15. Modules

Apache est modulaire.

Lister les modules chargés :

```bash
apache2ctl -M
```

Exemples courants :

```text
ssl            TLS
http2          HTTP/2
headers        modification d'en-têtes
rewrite        réécriture d'URL
proxy          infrastructure proxy
proxy_http     backend HTTP/1.1
proxy_fcgi     FastCGI
proxy_wstunnel WebSocket historique/spécifique
status         état interne du serveur
deflate        compression gzip/deflate
brotli         compression Brotli si disponible
```

Activer plusieurs modules :

```bash
sudo a2enmod ssl headers proxy proxy_http
sudo apache2ctl configtest
sudo systemctl reload apache2
```

Principe de sécurité : **désactiver les fonctions inutiles**, en particulier les modules rarement utilisés qui augmentent la surface d'attaque.

# 16. MPM : comment Apache gère les connexions

Les **Multi-Processing Modules** déterminent le modèle de concurrence.

Les MPM classiques de la branche 2.4 sont notamment :

- `mpm_event` ;
- `mpm_worker` ;
- `mpm_prefork`.

Afficher le MPM actif :

```bash
apache2ctl -V | grep -i 'Server MPM'
```

Sur un serveur moderne utilisant PHP-FPM ou faisant surtout du reverse proxy, **`event` est généralement le meilleur point de départ**.

`prefork` reste nécessaire dans certains scénarios historiques avec des modules non thread-safe, mais ne doit pas être choisi uniquement par habitude.

> [!important] MPM et PHP
> L'ancien modèle `mod_php` a longtemps conduit à utiliser `mpm_prefork`. Pour de nouveaux déploiements, PHP-FPM via FastCGI permet généralement de conserver `mpm_event` et de séparer le cycle de vie PHP de celui d'Apache.

# 17. Configuration MPM

Les paramètres disponibles dépendent du MPM.

Exemples de notions :

```text
ServerLimit
StartServers
ThreadsPerChild
MinSpareThreads
MaxSpareThreads
MaxRequestWorkers
MaxConnectionsPerChild
```

Ne pas copier des valeurs de blog sans mesurer : la bonne capacité dépend notamment de :

- la RAM ;
- la consommation par processus/thread ;
- la durée des requêtes ;
- la charge statique ou dynamique ;
- la latence des backends ;
- les limites de descripteurs de fichiers.

Observer d'abord :

```bash
ps -o pid,ppid,rss,cmd -C apache2
systemctl status apache2
```

# 18. HTTPS : ce que TLS apporte

HTTPS est HTTP transporté dans une connexion protégée par TLS.

TLS apporte principalement :

- confidentialité ;
- intégrité ;
- authentification du serveur par certificat ;
- éventuellement authentification du client avec mTLS.

Un certificat contient une **clé publique** et des informations d'identité/signature. La **clé privée reste dans un fichier distinct et secret**.

L'ancienne formulation « certificat X.509 sous forme d'une clé privée et d'une clé privée » est donc incorrecte.

# 19. Activer TLS

Sur Debian/Ubuntu :

```bash
sudo a2enmod ssl
sudo apache2ctl configtest
sudo systemctl reload apache2
```

Un VirtualHost HTTPS minimal :

```apache
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /srv/www/example.com/public

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    <Directory /srv/www/example.com/public>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined
</VirtualHost>
```

Ne jamais publier la clé privée dans un dépôt Git.

# 20. ACME et Let's Encrypt

Pour un service Internet public, l'automatisation ACME est généralement préférable à la génération manuelle de certificats.

La documentation Ubuntu 2026 recommande Certbot comme client ACME pour Let's Encrypt.

Installation Ubuntu documentée :

```bash
sudo snap install --classic certbot
```

Obtention/configuration via le plugin Apache :

```bash
sudo certbot --apache -d example.com -d www.example.com
```

Le domaine doit être réellement contrôlé et le challenge ACME doit être accessible selon la méthode utilisée.

Tester le renouvellement :

```bash
sudo certbot renew --dry-run
```

> [!warning] Certificat autosigné
> Un certificat autosigné peut convenir à un laboratoire ou à une PKI interne maîtrisée. Il n'est pas un substitut transparent à une chaîne de confiance publique pour un site Internet destiné aux navigateurs ordinaires.

# 21. Ne pas recopier d'anciennes suites cryptographiques

Une ancienne fiche proposait notamment :

```apache
SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP
```

Cette configuration doit être **supprimée** : elle fait référence à SSLv2, RC4, classes `LOW`, export et autres choix historiques incompatibles avec une politique TLS moderne.

Pour une installation actuelle :

1. utiliser une version Apache/OpenSSL maintenue ;
2. conserver les valeurs sûres fournies par la distribution lorsque l'on n'a pas de besoin particulier ;
3. si l'on personnalise, tester avec les exigences réelles de compatibilité ;
4. privilégier TLS 1.2 et TLS 1.3 pour un service moderne lorsque le parc client le permet ;
5. surveiller les recommandations de la distribution et des autorités de sécurité.

Exemple explicite, à adapter au contexte :

```apache
SSLProtocol -all +TLSv1.2 +TLSv1.3
```

La gestion des suites TLS 1.3 dépend d'OpenSSL et n'est pas identique à celle des anciennes suites configurées par `SSLCipherSuite`.

# 22. Rediriger HTTP vers HTTPS

Un VirtualHost clair et séparé :

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    Redirect permanent / https://example.com/
</VirtualHost>
```

Le module utilisé pour `Redirect` est `mod_alias`, chargé dans les installations courantes.

Pour des règles complexes, `mod_rewrite` peut être nécessaire, mais il ne faut pas utiliser un moteur de réécriture lorsque `Redirect` suffit.

# 23. HSTS

HSTS demande au navigateur de réutiliser HTTPS pour un domaine pendant une durée donnée.

Activer `headers` :

```bash
sudo a2enmod headers
```

Exemple **uniquement dans le VirtualHost HTTPS** :

```apache
Header always set Strict-Transport-Security "max-age=31536000"
```

`includeSubDomains` et `preload` ont des conséquences plus fortes : ne pas les ajouter sans vérifier que **tous** les sous-domaines sont durablement disponibles en HTTPS.

> [!warning] HSTS est persistant côté navigateur
> Une erreur de configuration HSTS peut rendre un site ou ses sous-domaines difficilement accessibles jusqu'à expiration de la politique. Commencer avec une durée courte lors d'un déploiement contrôlé.

# 24. En-têtes de sécurité

Avec `mod_headers`, on peut ajouter des politiques HTTP.

Exemple de point de départ :

```apache
Header always set X-Content-Type-Options "nosniff"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
```

Une **Content-Security-Policy** doit être conçue pour l'application réelle. Il est dangereux de recopier une CSP complexe sans comprendre les scripts, styles, frames et origines dont l'application dépend.

Les en-têtes suivants ne sont pas des substituts à la correction de vulnérabilités applicatives :

- CSP ;
- HSTS ;
- `X-Content-Type-Options` ;
- `Referrer-Policy`.

Ils forment une défense complémentaire.

# 25. HTTP/2

Apache 2.4 fournit HTTP/2 via `mod_http2`.

Activer :

```bash
sudo a2enmod http2
```

Dans le VirtualHost TLS :

```apache
Protocols h2 http/1.1
```

Tester :

```bash
curl -I --http2 https://example.com/
```

Ou voir le protocole négocié :

```bash
curl -sv --http2 https://example.com/ -o /dev/null
```

Pour les déploiements à forte concurrence HTTP/2, `mpm_event` est généralement préférable à `prefork`.

> [!note] HTTP/3
> Ne pas déduire de la présence de HTTP/2 qu'Apache 2.4 expose automatiquement HTTP/3/QUIC. La prise en charge, les modules et la maturité doivent être vérifiés pour la version réellement déployée avant toute affirmation.

# 26. Servir PHP : préférer PHP-FPM pour les nouveaux déploiements

Installer les composants adaptés à la version PHP de la distribution, par exemple :

```bash
sudo apt install php-fpm
```

Le nom exact du socket dépend de la version installée :

```bash
ls -l /run/php/
```

Activer les modules nécessaires :

```bash
sudo a2enmod proxy_fcgi setenvif
```

Sur Debian/Ubuntu, un helper de configuration PHP-FPM peut être installé par le paquet PHP concerné. Une configuration conceptuelle ressemble à :

```apache
<FilesMatch \.php$>
    SetHandler "proxy:unix:/run/php/php-fpm.sock|fcgi://localhost/"
</FilesMatch>
```

Ne pas recopier littéralement `php-fpm.sock` si ce socket n'existe pas. Vérifier le fichier de pool FPM et `/run/php/`.

Avantages de séparer Apache et PHP-FPM :

- Apache peut conserver un MPM adapté à la concurrence ;
- PHP possède ses propres pools et limites ;
- l'identité Unix et les ressources PHP peuvent être gérées séparément ;
- un redémarrage PHP n'impose pas nécessairement un redémarrage complet du serveur web.

# 27. Reverse proxy HTTP

Apache peut exposer un service qui écoute seulement en local ou sur un réseau privé.

Architecture :

```text
Internet
   │
   │ HTTPS
   ▼
Apache :443
   │
   │ HTTP local
   ▼
application :8000
```

Activer les modules :

```bash
sudo a2enmod proxy proxy_http
```

Exemple :

```apache
<VirtualHost *:443>
    ServerName app.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/app.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/app.example.com/privkey.pem

    ProxyRequests Off
    ProxyPass        / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/

    ErrorLog ${APACHE_LOG_DIR}/app.example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/app.example.com-access.log combined
</VirtualHost>
```

`ProxyPassReverse` réécrit notamment certains en-têtes de redirection émis par le backend afin qu'ils correspondent à l'URL publique.

> [!warning] `ProxyRequests On`
> `ProxyRequests On` active le rôle de **forward proxy**. Ce n'est pas nécessaire pour un reverse proxy et un forward proxy ouvert sur Internet peut être gravement abusé. Pour un reverse proxy ordinaire : `ProxyRequests Off`.

# 28. Transmettre correctement le schéma et l'adresse du client

Une application derrière un reverse proxy doit souvent connaître :

- le nom public ;
- le schéma public `https` ;
- l'adresse du client ;
- éventuellement le port externe.

Apache ajoute/complète généralement `X-Forwarded-For` dans le contexte proxy. On rencontre aussi :

```apache
RequestHeader set X-Forwarded-Proto "https"
```

Mais la confiance dans les en-têtes `X-Forwarded-*` doit être définie côté application. Une application ne doit pas accepter aveuglément une valeur fournie directement par un client non fiable.

Pour des architectures multi-proxy, documenter explicitement la chaîne de confiance.

# 29. `ProxyPreserveHost`

Par défaut, Apache peut utiliser le nom du backend pour l'en-tête `Host` envoyé au backend.

Certaines applications ont besoin du nom public original :

```apache
ProxyPreserveHost On
```

Ce choix n'est pas universel. Il faut comprendre comment l'application route les requêtes et construit ses URL absolues.

# 30. Reverse proxy vers un socket Unix

Un backend local peut écouter sur un socket Unix plutôt que sur un port TCP.

Exemple conceptuel :

```apache
ProxyPass / unix:/run/myapp/app.sock|http://localhost/
ProxyPassReverse / http://localhost/
```

Le compte Apache doit pouvoir accéder au socket et à ses répertoires parents.

Les permissions du socket doivent être organisées par le service backend, pas corrigées manuellement après chaque démarrage.

# 31. WebSocket

Les versions modernes d'Apache peuvent proxyfier les montées de protocole nécessaires à WebSocket dans de nombreux scénarios via `mod_proxy_http`. `mod_proxy_wstunnel` existe également pour ce rôle historique/spécifique.

Avant d'ajouter des règles compliquées :

1. vérifier la version Apache ;
2. consulter la documentation `mod_proxy_http`/`mod_proxy_wstunnel` ;
3. vérifier le chemin exact WebSocket ;
4. tester l'en-tête `Upgrade` ;
5. examiner les logs côté Apache et backend.

Éviter les règles de réécriture copiées de tutoriels anciens lorsqu'un simple `ProxyPass` moderne suffit.

# 32. HTTP/2 vers un backend

Le module `mod_proxy_http2` permet des connexions proxy HTTP/2 avec les schémas `h2://` et `h2c://`.

Exemple :

```apache
ProxyPass /app h2://backend.example.net/
ProxyPassReverse /app https://backend.example.net/
```

Cela ne signifie pas que HTTP/2 doit être utilisé partout entre reverse proxy et backend. Sur une machine locale ou un petit réseau privé, HTTP/1.1 avec keep-alive est souvent suffisant. Mesurer avant de complexifier.

# 33. Équilibrage de charge

Apache peut regrouper plusieurs backends :

```apache
<Proxy "balancer://appcluster">
    BalancerMember "http://10.0.0.11:8000"
    BalancerMember "http://10.0.0.12:8000"
</Proxy>

ProxyPass        / balancer://appcluster/
ProxyPassReverse / balancer://appcluster/
```

Modules typiques :

```bash
sudo a2enmod proxy proxy_http proxy_balancer lbmethod_byrequests
```

Le choix du mode d'équilibrage, des health checks, de l'affinité et du failover dépend de l'application.

Une plate-forme spécialisée comme HAProxy, Envoy ou un load balancer cloud peut devenir préférable pour les architectures complexes.

# 34. Timeouts du proxy

Une requête longue ne doit pas conduire à augmenter aveuglément tous les timeouts du serveur.

On peut agir localement :

```apache
ProxyPass /api http://127.0.0.1:8000/api timeout=30
```

Ou utiliser :

```apache
ProxyTimeout 30
```

Questions à poser avant d'augmenter un timeout :

- le backend est-il lent ou bloqué ?
- l'opération devrait-elle devenir asynchrone ?
- une base de données attend-elle un verrou ?
- une dépendance externe est-elle indisponible ?
- le client doit-il réellement conserver une connexion HTTP ouverte ?

# 35. Réécriture d'URL avec `mod_rewrite`

Activer :

```bash
sudo a2enmod rewrite
```

Exemple simple :

```apache
RewriteEngine On
RewriteRule ^/old/(.*)$ /new/$1 [R=301,L]
```

Pour une redirection simple, préférer toutefois :

```apache
Redirect permanent /old/ /new/
```

`mod_rewrite` devient utile lorsque la condition dépend de plusieurs attributs de la requête.

Inspecter avec un niveau de trace temporaire et prudent :

```apache
LogLevel warn rewrite:trace3
```

Ne pas laisser une trace très verbeuse inutilement en production.

# 36. `Alias`

`Alias` permet de mapper une URL vers un répertoire qui n'est pas sous `DocumentRoot`.

```apache
Alias /downloads/ /srv/downloads/

<Directory /srv/downloads>
    Options -Indexes
    AllowOverride None
    Require all granted
</Directory>
```

Le mapping URL et l'autorisation filesystem sont deux sujets distincts. Ne pas oublier le bloc `<Directory>` approprié.

# 37. Fichiers d'index

La directive `DirectoryIndex` définit les fichiers recherchés :

```apache
DirectoryIndex index.html index.php
```

Ne pas ajouter une longue liste de noms historiques « au cas où ».

Vérifier quel module fournit la directive et quelle configuration de distribution est déjà active.

# 38. Types MIME

Apache associe les extensions à des types MIME.

Une mauvaise valeur `Content-Type` peut casser un navigateur ou créer des risques lorsqu'elle est combinée à du contenu utilisateur.

Inspecter :

```bash
curl -I https://example.com/app.js
```

Réponse attendue, par exemple :

```text
Content-Type: text/javascript
```

ou une valeur JavaScript compatible selon le serveur/application.

Éviter d'utiliser arbitrairement `application/octet-stream` pour tout contenu web.

# 39. Compression

Pour les contenus textuels, Apache peut compresser les réponses.

Gzip/deflate :

```bash
sudo a2enmod deflate
```

Brotli lorsque le module est fourni :

```bash
sudo a2enmod brotli
```

La compression consomme du CPU et ne profite pas de la même manière à tous les formats.

Ne pas recompresser inutilement :

- JPEG ;
- PNG ;
- WebP ;
- AVIF ;
- ZIP ;
- archives déjà compressées.

Tester :

```bash
curl -I -H 'Accept-Encoding: br, gzip' https://example.com/style.css
```

# 40. Cache navigateur

Pour des actifs versionnés par leur nom :

```text
app.8f31c2.js
style.a91be0.css
```

on peut définir une longue durée de cache.

Exemple avec `mod_expires` :

```bash
sudo a2enmod expires
```

```apache
ExpiresActive On
ExpiresByType text/css "access plus 1 year"
ExpiresByType application/javascript "access plus 1 year"
```

Cela n'est correct que si les URL changent lorsque le contenu change.

Pour un `index.html` mutable, une politique plus courte ou une revalidation peut être préférable.

# 41. ETag et validation conditionnelle

HTTP fournit des mécanismes comme :

```text
ETag
Last-Modified
If-None-Match
If-Modified-Since
```

Ils permettent une réponse :

```text
304 Not Modified
```

sans renvoyer le corps complet.

Ne pas chercher à désactiver ETag par réflexe à cause de vieux tutoriels liés à des fermes de serveurs. Comprendre d'abord comment la ressource est servie et quel cache est utilisé.

# 42. Authentification Basic

Pour une petite zone d'administration protégée par TLS :

```bash
sudo htpasswd -c /etc/apache2/htpasswd-admin alice
```

Configuration :

```apache
<Location /admin>
    AuthType Basic
    AuthName "Administration"
    AuthUserFile /etc/apache2/htpasswd-admin
    Require valid-user
</Location>
```

> [!warning] Basic n'est pas du chiffrement
> HTTP Basic transmet un secret encodé dans chaque requête authentifiée. Utiliser **HTTPS** et protéger strictement le fichier de mots de passe.

Pour une application moderne multi-utilisateur, un fournisseur d'identité OIDC/SAML ou l'authentification applicative peut être plus adapté.

# 43. Autoriser une plage IP

Exemple :

```apache
<Location /internal>
    Require ip 192.0.2.0/24
</Location>
```

Avec IPv6 :

```apache
<Location /internal>
    <RequireAny>
        Require ip 192.0.2.0/24
        Require ip 2001:db8::/32
    </RequireAny>
</Location>
```

Derrière un reverse proxy intermédiaire, Apache peut voir l'IP du proxy au lieu du client. Dans ce cas, il faut configurer proprement `mod_remoteip` et **ne faire confiance qu'aux proxys connus**.

# 44. `mod_remoteip`

Activer :

```bash
sudo a2enmod remoteip
```

Le module peut remplacer l'adresse client interne par l'adresse transmise par un proxy de confiance.

Exemple conceptuel :

```apache
RemoteIPHeader X-Forwarded-For
RemoteIPTrustedProxy 10.0.0.0/8
```

Ne jamais faire confiance à `X-Forwarded-For` depuis n'importe quelle origine : sinon le client peut forger son adresse et contourner journalisation ou contrôles basés sur IP.

# 45. Journaux d'accès

Configuration typique :

```apache
CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined
```

Le format `combined` contient notamment :

- IP du client ;
- date ;
- requête ;
- code HTTP ;
- taille ;
- referer ;
- user-agent.

Suivre :

```bash
sudo tail -f /var/log/apache2/example.com-access.log
```

Ne pas oublier la protection des données : les logs peuvent contenir des adresses IP, chemins, paramètres, identifiants ou autres données sensibles.

# 46. Journaux d'erreur

Exemple :

```apache
ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
```

Consulter :

```bash
sudo tail -n 100 /var/log/apache2/example.com-error.log
```

Avec systemd :

```bash
journalctl -u apache2 -n 100 --no-pager
```

Le journal systemd est particulièrement utile pour :

- un échec au démarrage ;
- un module impossible à charger ;
- un port déjà occupé ;
- une erreur de permission ;
- une erreur détectée par systemd avant l'ouverture des logs du site.

# 47. Niveaux de logs

Le niveau global peut être réglé avec `LogLevel`.

Exemple :

```apache
LogLevel warn
```

On peut augmenter un module temporairement :

```apache
LogLevel warn proxy:debug
```

Ou :

```apache
LogLevel warn rewrite:trace3
```

> [!warning] Données sensibles
> Les niveaux `debug`/`trace` peuvent produire énormément de données et parfois exposer des informations que l'on ne souhaite pas conserver. Les utiliser de façon ciblée et temporaire.

# 48. Log rotation

Sur Debian/Ubuntu, `logrotate` gère habituellement les fichiers Apache.

Inspecter :

```bash
cat /etc/logrotate.d/apache2
```

Ne jamais supprimer brutalement un fichier de log énorme en pensant récupérer immédiatement l'espace : un processus peut encore garder le descripteur ouvert.

Diagnostiquer :

```bash
sudo lsof +L1
```

Voir aussi [[Watchdog espace disque]].

# 49. `mod_status`

`mod_status` expose des informations sur l'activité du serveur.

Activer :

```bash
sudo a2enmod status
```

Limiter strictement l'accès :

```apache
<Location /server-status>
    SetHandler server-status
    Require local
</Location>
```

Puis :

```bash
curl http://127.0.0.1/server-status?auto
```

Ne pas exposer inutilement `server-status` au monde : il révèle des informations opérationnelles.

# 50. Métriques et supervision

Les signaux utiles incluent :

```text
requêtes/s
latence
codes 2xx/3xx/4xx/5xx
connexions actives
workers occupés/libres
CPU
RAM
fichiers ouverts
saturation backend
expiration certificats
espace disque des logs
```

`mod_status` peut alimenter un exporter ou une plate-forme de supervision.

Une alerte doit répondre à une question opérationnelle. Exemple :

```text
> 5 % de 5xx pendant 10 minutes
```

est souvent plus utile que :

```text
Apache utilise 30 % de CPU
```

sans contexte.

# 51. Codes HTTP utiles au diagnostic

| Code | Interprétation typique |
|---:|---|
| `200` | succès |
| `301`/`308` | redirection permanente |
| `302`/`307` | redirection temporaire |
| `304` | cache valide, corps non renvoyé |
| `400` | requête invalide |
| `401` | authentification requise/échouée |
| `403` | accès refusé |
| `404` | ressource non trouvée |
| `405` | méthode refusée |
| `413` | corps trop grand |
| `429` | trop de requêtes, généralement application/proxy |
| `500` | erreur interne |
| `502` | mauvaise réponse d'un backend |
| `503` | service indisponible |
| `504` | timeout de backend/gateway |

Le code n'identifie pas à lui seul la cause. Lire les logs et reproduire avec `curl -v`.

# 52. `curl` comme outil de diagnostic

En-têtes seulement :

```bash
curl -I https://example.com/
```

Verbose :

```bash
curl -v https://example.com/
```

Forcer une résolution DNS :

```bash
curl --resolve example.com:443:192.0.2.10 https://example.com/
```

Afficher seulement le statut :

```bash
curl -sS -o /dev/null -w '%{http_code}\n' https://example.com/
```

Mesurer quelques temps :

```bash
curl -sS -o /dev/null \
  -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} total=%{time_total}\n' \
  https://example.com/
```

# 53. Diagnostic TLS avec OpenSSL

Afficher le certificat présenté :

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null
```

Extraire des informations :

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

Le paramètre `-servername` est essentiel lorsque plusieurs certificats sont sélectionnés par SNI.

# 54. SNI

SNI permet au client d'indiquer le nom demandé pendant le handshake TLS.

Cela permet d'héberger plusieurs sites HTTPS sur la même adresse IP :

```text
203.0.113.10
 ├── a.example.com → certificat A
 └── b.example.com → certificat B
```

Les clients web modernes prennent en charge SNI.

Vérifier la sélection :

```bash
openssl s_client -connect 203.0.113.10:443 -servername a.example.com </dev/null
```

# 55. mTLS

TLS peut aussi authentifier un client avec un certificat.

Conceptuellement :

```apache
SSLVerifyClient require
SSLVerifyDepth 2
SSLCACertificateFile /etc/apache2/client-ca.pem
```

mTLS peut être pertinent pour :

- API machine-à-machine ;
- administration très contrôlée ;
- infrastructures internes ;
- intégrations B2B.

Il ajoute toutefois une gestion de cycle de vie des certificats clients. Ne pas le déployer sans procédure d'émission, révocation, rotation et récupération.

# 56. OCSP stapling

Apache/mod_ssl peut présenter une réponse OCSP agrafée au handshake afin que le client n'interroge pas nécessairement lui-même le répondeur de l'autorité.

La configuration dépend de la chaîne de certificats et des capacités de l'environnement OpenSSL.

Avant d'activer :

- vérifier la documentation `mod_ssl` de la version installée ;
- vérifier que l'autorité fournit OCSP ;
- surveiller les erreurs de stapling ;
- ne pas confondre stapling et validation complète de la PKI.

# 57. Désactiver les informations inutiles

La configuration Debian/Ubuntu fournit généralement un fragment `security.conf`.

Directives classiques :

```apache
ServerTokens Prod
ServerSignature Off
```

Elles limitent certaines informations affichées dans les réponses/pages d'erreur.

> [!important] Ce n'est pas une protection principale
> Masquer une version n'empêche pas une attaque. L'objectif principal reste : correctifs de sécurité, surface minimale, isolation, configuration correcte et surveillance.

# 58. Méthodes HTTP

Une application peut vouloir limiter des méthodes sur une ressource.

Exemple :

```apache
<LimitExcept GET POST>
    Require all denied
</LimitExcept>
```

Mais attention : bloquer arbitrairement `OPTIONS`, `HEAD`, WebDAV ou d'autres méthodes peut casser un protocole ou une API.

Définir la politique selon le besoin fonctionnel réel.

# 59. Limiter la taille des requêtes

Apache dispose de limites comme `LimitRequestBody`.

Exemple :

```apache
LimitRequestBody 10485760
```

Ici environ 10 MiB.

Une chaîne reverse proxy doit être cohérente :

```text
client
 ↓
CDN/WAF
 ↓
Apache
 ↓
application
```

Si chaque couche a une limite différente, l'utilisateur peut obtenir des erreurs différentes selon le chemin.

# 60. Timeouts et attaques lentes

Des clients peuvent maintenir des connexions lentes et consommer des ressources.

Apache fournit des mécanismes comme `mod_reqtimeout`.

Vérifier sa présence :

```bash
apache2ctl -M | grep reqtimeout
```

La configuration distribuée par Debian/Ubuntu fournit généralement des valeurs raisonnables.

Ne pas désactiver une protection contre les requêtes lentes uniquement pour faire fonctionner un client défaillant sans analyser la cause.

# 61. KeepAlive

HTTP/1.x peut réutiliser une connexion TCP pour plusieurs requêtes.

Directives :

```apache
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
```

Les valeurs exactes doivent être adaptées à la charge.

Avec `mpm_event`, les connexions keep-alive inactives sont gérées plus efficacement qu'avec des architectures historiques basées sur un processus par connexion.

# 62. Nombre maximal de workers

`MaxRequestWorkers` limite le nombre de requêtes traitées simultanément par les workers.

Une valeur trop faible :

```text
files d'attente
latence
503 éventuels
```

Une valeur trop élevée :

```text
pression mémoire
swap
OOM
latence globale
```

Dimensionner à partir des mesures, puis charger progressivement.

# 63. Descripteurs de fichiers

Chaque connexion, log ou socket consomme des ressources noyau.

Inspecter :

```bash
cat /proc/$(pgrep -o apache2)/limits
```

Voir le nombre de fichiers ouverts :

```bash
sudo lsof -p "$(pgrep -o apache2)" | wc -l
```

Une erreur de fichiers ouverts épuisés peut provoquer des symptômes qui ressemblent à une panne réseau ou HTTP/2.

# 64. Sécurité des fichiers de configuration

Les fichiers Apache peuvent contenir :

- chemins de clés privées ;
- mots de passe de proxy ;
- secrets Basic ;
- noms internes ;
- adresses backend ;
- jetons si la configuration est mal conçue.

Éviter de stocker des secrets en clair dans un dépôt public.

Pour les clés TLS :

```bash
sudo stat /etc/letsencrypt/live/example.com/privkey.pem
```

Le fichier doit être accessible uniquement aux comptes nécessaires.

# 65. Ne pas servir les secrets du projet

Une erreur classique consiste à définir :

```apache
DocumentRoot /srv/myapp
```

alors que ce répertoire contient aussi :

```text
.env
.git/
config/
backups/
private/
```

Préférer une racine publique dédiée :

```text
/srv/myapp/
├── app/
├── config/
├── secrets/
└── public/       ← DocumentRoot
```

Puis :

```apache
DocumentRoot /srv/myapp/public
```

# 66. Protection explicite des fichiers cachés

Même avec une bonne racine publique, on peut ajouter une défense en profondeur adaptée au projet.

Exemple :

```apache
<FilesMatch "^\.ht">
    Require all denied
</FilesMatch>
```

Les paquets Apache possèdent déjà certaines protections de ce type.

Pour `.git`, `.env` ou des backups, la meilleure défense reste de **ne pas les placer sous la racine publique**.

# 67. Symlinks

`FollowSymLinks` permet à Apache de suivre des liens symboliques.

Un lien mal placé peut faire sortir la résolution de la racine prévue.

Avant d'activer largement :

- comprendre qui peut créer des liens ;
- protéger les répertoires cibles ;
- éviter que des utilisateurs non fiables contrôlent l'arborescence servie ;
- vérifier les permissions Unix.

# 68. CGI

CGI lance ou sollicite un programme pour générer une réponse.

C'est un mécanisme historique mais encore utilisable.

Ne pas activer `ExecCGI` globalement.

Limiter à un répertoire spécifique si nécessaire :

```apache
<Directory /srv/www/cgi-bin>
    Options +ExecCGI -Indexes
    Require all granted
</Directory>
```

Pour les applications modernes persistantes, FastCGI, WSGI ou un serveur applicatif derrière reverse proxy sont généralement préférables.

# 69. WSGI

Pour Python, `mod_wsgi` peut héberger une application compatible WSGI.

Sur Ubuntu :

```bash
sudo apt install libapache2-mod-wsgi-py3
```

Une autre architecture très courante consiste à lancer Gunicorn/uWSGI/etc. comme service séparé puis à utiliser Apache en reverse proxy.

Avantages du service séparé :

- responsabilités distinctes ;
- déploiement applicatif indépendant ;
- isolation des environnements Python ;
- diagnostic plus clair.

Voir les exigences de l'application avant de choisir.

# 70. Ne pas exécuter l'application en root

Apache démarre avec les privilèges nécessaires à certaines opérations puis ses workers utilisent un compte non privilégié selon la configuration de la distribution.

Les backends doivent eux aussi fonctionner avec des comptes dédiés et les droits minimaux.

Exemple systemd :

```ini
[Service]
User=myapp
Group=myapp
```

Le fait qu'Apache soit en frontal ne justifie jamais d'exécuter l'application web en root.

# 71. Reverse proxy et SSRF

Un reverse proxy mal configuré peut devenir un moyen d'atteindre des destinations internes inattendues.

Éviter de construire dynamiquement une destination proxy à partir d'une entrée client non validée.

Risque conceptuel :

```text
GET /proxy?url=http://169.254.169.254/...
```

ou accès à :

```text
127.0.0.1
réseau d'administration
services cloud metadata
sockets internes
```

La destination d'un reverse proxy doit normalement être définie par configuration et limitée à des backends connus.

# 72. Forward proxy : risque d'open proxy

Un forward proxy autorise un client à demander à Apache d'aller chercher une destination arbitraire.

Configuration dangereuse si ouverte :

```apache
ProxyRequests On
```

Un open proxy peut servir :

- à masquer des attaques ;
- à contourner des restrictions ;
- à consommer votre bande passante ;
- à faire apparaître votre adresse IP comme source d'abus.

Si le besoin est uniquement un reverse proxy :

```apache
ProxyRequests Off
```

# 73. Accès à l'administration du balancer

`balancer-manager` peut modifier certains paramètres d'équilibrage.

S'il est activé, le limiter strictement :

```apache
<Location /balancer-manager>
    SetHandler balancer-manager
    Require local
</Location>
```

Ne pas exposer cette interface publiquement sans authentification et politique réseau adaptées.

# 74. CORS

CORS est une politique appliquée par les navigateurs pour certains accès cross-origin.

Apache peut ajouter les en-têtes :

```apache
Header always set Access-Control-Allow-Origin "https://frontend.example.com"
```

Mais CORS est souvent mieux défini par l'application, car la décision dépend :

- de l'origine ;
- de la méthode ;
- des credentials ;
- des en-têtes ;
- de la ressource.

> [!warning] `*` + credentials
> Les règles CORS ont des contraintes précises. Ne pas utiliser `Access-Control-Allow-Origin: *` comme solution universelle, surtout pour des requêtes authentifiées.

# 75. CSP

Une Content-Security-Policy peut réduire l'impact de certaines injections côté navigateur.

Commencer éventuellement en mode rapport :

```apache
Header always set Content-Security-Policy-Report-Only "default-src 'self'"
```

Puis observer et ajuster.

Une politique réaliste peut nécessiter :

- `script-src` ;
- `style-src` ;
- `img-src` ;
- `connect-src` ;
- `frame-ancestors` ;
- nonces ou hashes.

Éviter de rendre la politique inutile avec un ensemble très permissif uniquement pour faire disparaître les erreurs console.

# 76. Cookies et reverse proxy

Apache peut réécrire certains attributs de cookies d'un backend avec des directives `ProxyPassReverseCookie*`.

Exemples de besoins :

```text
backend: internal.example
public: app.example.com
```

ou :

```text
backend path: /
public path: /app/
```

La version 2.4.68 contient justement des correctifs de sécurité touchant certaines fonctions proxy/cookie. Cela illustre pourquoi les modules réellement utilisés doivent rester à jour.

# 77. Mises à jour de sécurité

Sur Debian/Ubuntu :

```bash
sudo apt update
apt list --upgradable
```

Version candidate :

```bash
apt policy apache2
```

Après une mise à jour :

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
```

Selon la nature de la mise à jour, un redémarrage complet du processus peut être requis pour charger les nouveaux binaires/bibliothèques.

Vérifier :

```bash
sudo needrestart
```

si l'outil est installé.

# 78. Sauvegarder la configuration

Les éléments à sauvegarder comprennent typiquement :

```text
/etc/apache2/
/etc/letsencrypt/       selon stratégie de sauvegarde
fichiers applicatifs
configuration systemd des backends
secrets via un mécanisme approprié
```

La clé privée TLS impose des exigences particulières de confidentialité.

Une sauvegarde de configuration n'est utile que si la restauration est testée.

# 79. Configuration sous Git

On peut versionner une **copie contrôlée** de la configuration, sans secrets.

Exemple :

```text
infra/
└── apache/
    ├── sites-available/
    ├── conf-available/
    └── README.md
```

Pipeline possible :

```text
commit
  ↓
review
  ↓
test dans une VM/conteneur
  ↓
apache2ctl configtest
  ↓
déploiement
  ↓
reload
  ↓
smoke tests
```

Ne pas modifier `/etc/apache2` par un `git pull` non contrôlé en production.

# 80. Idempotence et gestion de configuration

Pour plusieurs serveurs, utiliser un outil de configuration :

- Ansible ;
- Salt ;
- Puppet ;
- Chef ;
- image système ;
- autre mécanisme déclaratif.

Exemple logique Ansible :

```text
installer apache2
activer modules
copier VirtualHost
valider config
notifier reload
```

Le rechargement doit être déclenché seulement lorsque la configuration change et **après validation**.

# 81. Apache dans un conteneur

Un conteneur change l'organisation des fichiers et le cycle de vie, mais pas les concepts HTTP.

Bonnes pratiques :

- image versionnée ;
- configuration injectée explicitement ;
- processus Apache au premier plan ;
- logs accessibles au runtime ;
- secrets fournis par le mécanisme de secrets de la plate-forme ;
- filesystem aussi immuable que possible ;
- healthcheck pertinent.

Ne pas utiliser un conteneur comme justification pour ignorer les correctifs Apache/OpenSSL.

# 82. Apache derrière un autre reverse proxy

Architecture fréquente :

```text
Internet
  ↓
CDN / LB / WAF
  ↓
Apache
  ↓
application
```

Questions indispensables :

- qui termine TLS ?
- qui ajoute `X-Forwarded-For` ou `Forwarded` ?
- à quels proxys Apache fait-il confiance ?
- comment l'application retrouve-t-elle le schéma public ?
- où la limite de taille est-elle appliquée ?
- où se fait la redirection HTTPS ?
- qui produit l'identifiant de corrélation ?

Les doubles redirections et les boucles HTTP↔HTTPS viennent souvent d'une chaîne de proxy mal modélisée.

# 83. En-tête standard `Forwarded`

HTTP définit aussi l'en-tête standard :

```text
Forwarded: for=192.0.2.60;proto=https;host=example.com
```

Dans la pratique, les en-têtes `X-Forwarded-*` restent très répandus.

Le point important n'est pas seulement le nom de l'en-tête, mais **la frontière de confiance** : une valeur provenant d'un proxy connu peut avoir un sens différent de la même valeur injectée par un client Internet.

# 84. Identification et corrélation des requêtes

Pour diagnostiquer une architecture distribuée, utiliser un identifiant de requête/corrélation.

Exemple conceptuel :

```text
X-Request-ID: 9d3f...
```

Le même identifiant doit apparaître dans :

```text
Apache
backend
traces
logs applicatifs
```

Il devient alors possible de suivre une requête de bout en bout.

Ne pas traiter automatiquement un identifiant fourni par un client comme digne de confiance sans stratégie de validation/génération.

# 85. `graceful` et déploiement sans coupure brutale

Après validation :

```bash
sudo apache2ctl graceful
```

Apache demande aux anciens workers de terminer leurs connexions en cours et démarre de nouveaux workers avec la configuration rechargée.

Vérifier ensuite :

```bash
systemctl status apache2 --no-pager
journalctl -u apache2 --since '-2 min' --no-pager
curl -fsS https://example.com/ >/dev/null
```

Un reload réussi au niveau systemd ne prouve pas que l'application backend répond correctement : effectuer des smoke tests.

# 86. Procédure de modification sûre

Avant :

```bash
sudo cp -a /etc/apache2/sites-available/example.com.conf \
  /etc/apache2/sites-available/example.com.conf.bak
```

Modifier, puis :

```bash
sudo apache2ctl configtest
sudo apache2ctl -S
sudo systemctl reload apache2
```

Tester :

```bash
curl -fsSI https://example.com/
```

Consulter :

```bash
journalctl -u apache2 --since '-5 min' --no-pager
```

Si la configuration est gérée sous Git/Ansible, préférer le rollback de la source de vérité à l'accumulation de fichiers `.bak`.

# 87. Erreur : `Address already in use`

Symptôme : Apache ne démarre pas et signale que le port est déjà utilisé.

Chercher :

```bash
sudo ss -ltnp | grep -E ':(80|443)\b'
```

Ou :

```bash
sudo lsof -iTCP:80 -sTCP:LISTEN
sudo lsof -iTCP:443 -sTCP:LISTEN
```

Causes possibles :

- Nginx déjà lancé ;
- ancien Apache hors systemd ;
- application qui écoute directement sur 443 ;
- doublon `Listen`/instance ;
- conteneur publiant le même port.

# 88. Erreur `403 Forbidden`

Checklist :

```bash
sudo apache2ctl -S
namei -l /srv/www/example.com/public/index.html
sudo tail -n 100 /var/log/apache2/example.com-error.log
```

Vérifier :

1. le bon VirtualHost ;
2. `DocumentRoot` ;
3. `<Directory>` correspondant ;
4. `Require all granted` ;
5. droits de traversée des répertoires ;
6. AppArmor/SELinux selon distribution ;
7. ACL ;
8. absence d'une règle plus restrictive.

Ne pas commencer par `chmod 777`.

# 89. Erreur `404 Not Found`

Tester le chemin demandé :

```bash
curl -v https://example.com/chemin
```

Inspecter :

```bash
sudo apache2ctl -S
```

Puis distinguer :

```text
404 produit par Apache
vs
404 produit par le backend
```

Un reverse proxy peut parfaitement transmettre un `404` généré par l'application.

Ajouter temporairement un en-tête ou comparer les logs peut aider à identifier la couche.

# 90. Erreur `500 Internal Server Error`

Lire immédiatement :

```bash
sudo tail -n 100 /var/log/apache2/example.com-error.log
```

Causes typiques :

- `.htaccess` invalide ;
- directive non autorisée dans ce contexte ;
- erreur CGI ;
- erreur PHP ;
- module absent ;
- permissions ;
- application backend.

Si Apache ne démarre plus :

```bash
sudo apache2ctl configtest
journalctl -u apache2 -n 100 --no-pager
```

# 91. Erreur `502 Bad Gateway`

Pour un reverse proxy :

```bash
curl -v http://127.0.0.1:8000/
```

Si le backend utilise un socket :

```bash
sudo ss -lxnp
ls -l /run/myapp/
```

Puis vérifier :

```bash
sudo tail -n 100 /var/log/apache2/app.example.com-error.log
```

Causes fréquentes :

- backend arrêté ;
- mauvais port ;
- mauvais socket ;
- permission socket ;
- backend ferme la connexion ;
- TLS attendu d'un côté mais HTTP utilisé de l'autre.

# 92. Erreur `503 Service Unavailable`

Possibilités :

- tous les backends sont indisponibles ;
- pool de workers saturé ;
- application en maintenance ;
- limitation par module ;
- service FastCGI indisponible.

Regarder simultanément :

```bash
systemctl status apache2
systemctl status myapp
curl -v http://127.0.0.1:8000/
```

et les logs des deux services.

# 93. Erreur `504 Gateway Timeout`

Apache atteint généralement le backend mais n'obtient pas une réponse dans le délai prévu.

Ne pas se limiter à :

```text
augmenter ProxyTimeout
```

Mesurer :

- latence applicative ;
- appels DB ;
- appels externes ;
- files d'attente ;
- deadlocks ;
- saturation CPU/RAM ;
- nombre de workers backend.

# 94. Mauvais certificat présenté

Tester :

```bash
openssl s_client -connect 192.0.2.10:443 -servername example.com </dev/null
```

Puis :

```bash
sudo apache2ctl -S
```

Causes :

- `ServerName` absent ou incorrect ;
- mauvais VirtualHost chargé en premier ;
- mauvais chemin de certificat ;
- SNI non fourni pendant le test ;
- proxy/CDN externe termine TLS avant Apache.

# 95. Boucle de redirection HTTPS

Symptôme :

```text
ERR_TOO_MANY_REDIRECTS
```

Architecture typique :

```text
client --HTTPS--> load balancer --HTTP--> Apache/app
```

Si l'application croit recevoir HTTP, elle redirige vers HTTPS ; le load balancer revient en HTTP interne ; boucle.

Solution : transmettre et **faire confiance correctement** au schéma public, par exemple via `X-Forwarded-Proto`, avec configuration cohérente du framework.

# 96. HTTP/2 non négocié

Vérifier :

```bash
apache2ctl -M | grep http2
```

Puis :

```bash
apache2ctl -S
```

Dans le VirtualHost :

```apache
Protocols h2 http/1.1
```

Tester :

```bash
curl -sv --http2 https://example.com/ -o /dev/null
```

Vérifier également la bibliothèque TLS/ALPN et la présence éventuelle d'un proxy intermédiaire.

# 97. Apache démarre mais le mauvais site répond

Commande centrale :

```bash
sudo apache2ctl -S
```

Elle révèle :

- adresses/ports ;
- ordre des VirtualHost ;
- `ServerName` ;
- fichiers sources de configuration.

Lister les sites actifs :

```bash
ls -l /etc/apache2/sites-enabled/
```

Tester le nom exact avec :

```bash
curl --resolve example.com:80:192.0.2.10 http://example.com/
```

# 98. DNS ≠ Apache

Si :

```bash
dig example.com
```

ne pointe pas vers le serveur attendu, modifier Apache ne corrigera pas le DNS.

Chaîne à vérifier :

```text
DNS
 ↓
routage / firewall / NAT
 ↓
port 80/443
 ↓
Apache
 ↓
VirtualHost
 ↓
contenu/backend
```

Tester chaque couche séparément.

# 99. Pare-feu

Avec UFW :

```bash
sudo ufw status verbose
```

Ouvrir explicitement ce qui est nécessaire :

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Si le serveur n'a pas à exposer HTTP clair, il peut être pertinent de n'ouvrir que 443, sauf besoins ACME/redirection. Cela dépend de l'architecture.

Ne pas oublier :

- firewall cloud ;
- security groups ;
- NAT ;
- box/routeur ;
- pare-feu local.

# 100. AppArmor / SELinux

Selon la distribution et les profils actifs, les permissions POSIX ne sont pas la seule politique.

Ubuntu utilise largement AppArmor.

État :

```bash
sudo aa-status
```

Sur une distribution SELinux :

```bash
getenforce
```

Ne pas désactiver globalement un mécanisme de confinement pour « voir si ça marche » sans analyser les logs de refus.

# 101. Résolution des chemins et `realpath`

Lorsqu'un alias ou lien symbolique est complexe :

```bash
realpath /srv/www/example.com/public
namei -l /srv/www/example.com/public
```

Cela permet de vérifier :

- cible réelle ;
- droits des parents ;
- montage ;
- lien cassé.

# 102. Test de configuration dans un pipeline

Dans une image ou VM de CI, charger la configuration puis :

```bash
apache2ctl configtest
```

On peut aussi vérifier :

```bash
apache2ctl -S
apache2ctl -M
```

Et exécuter des tests HTTP :

```bash
curl -fsS http://127.0.0.1/health
```

Pour TLS :

```bash
curl -fkSs https://127.0.0.1/health
```

`-k` ne doit servir qu'au laboratoire avec certificat volontairement non fiable.

# 103. Liveness et readiness

Dans un orchestrateur, distinguer :

```text
liveness  → le processus doit-il être redémarré ?
readiness → peut-il recevoir du trafic maintenant ?
```

Tester uniquement la page statique Apache peut donner :

```text
Apache OK
backend mort
```

Une readiness utile doit éventuellement tester la capacité réelle de servir l'application, sans pour autant déclencher une opération lourde ou destructive.

# 104. Pages d'erreur

Apache peut personnaliser les réponses :

```apache
ErrorDocument 404 /errors/404.html
```

Ne pas faire une page d'erreur dépendante d'un backend qui peut être la cause de la panne.

Une page `503` locale au reverse proxy peut être utile lors d'une indisponibilité applicative.

Éviter d'exposer :

- stack traces ;
- chemins internes ;
- versions précises ;
- secrets ;
- requêtes SQL.

# 105. Maintenance

Un mode maintenance propre peut être géré :

- par l'application ;
- par un fichier/flag consulté par `mod_rewrite` ;
- par le load balancer ;
- par le système de déploiement.

Objectifs :

- retourner un `503`, pas un faux `200` ;
- éventuellement fournir `Retry-After` ;
- laisser passer les health checks nécessaires ;
- permettre l'administration depuis un réseau sûr.

# 106. `503` avec `Retry-After`

Exemple conceptuel :

```apache
Header always set Retry-After "300" env=maintenance
```

Un `Retry-After` informe le client d'une durée ou date approximative avant nouvelle tentative.

À coordonner avec la manière dont le mode maintenance est déclenché.

# 107. Reverse proxy et chemins

Attention aux slashs :

```apache
ProxyPass /app/ http://127.0.0.1:8000/
```

n'a pas exactement la même sémantique qu'une combinaison incohérente de chemins avec/sans `/` final.

Toujours tester :

```text
/app
/app/
/app/foo
```

et les redirections produites par le backend.

# 108. Ne pas mélanger URL et filesystem

Ces contextes n'adressent pas la même chose :

```apache
<Directory /srv/www/example.com/public>
```

→ chemin disque.

```apache
<Location /admin>
```

→ espace d'URL.

```apache
<FilesMatch "\.php$">
```

→ nom de fichier.

Choisir le bon contexte évite des règles de sécurité appliquées au mauvais endroit.

# 109. Ordre des sections de configuration

Apache fusionne des directives provenant de contextes différents selon des règles documentées.

Des comportements surprenants viennent souvent du cumul de :

```text
<Directory>
.htaccess
<Files>
<Location>
VirtualHost
```

Lorsqu'une autorisation semble incohérente, simplifier temporairement la configuration et consulter les règles de fusion de sections de la documentation Apache.

# 110. Variables d'environnement Apache

On rencontre :

```text
${APACHE_LOG_DIR}
```

Sur Debian/Ubuntu, certaines variables sont définies dans :

```text
/etc/apache2/envvars
```

Ne pas confondre :

- variables shell/systemd ;
- variables de configuration Apache ;
- variables CGI ;
- variables d'environnement d'une application backend.

# 111. `SetEnvIf` et décisions conditionnelles

`mod_setenvif` permet de définir des variables selon des attributs de requête.

Exemple conceptuel :

```apache
SetEnvIf Request_URI "^/health$" healthcheck
```

Ces variables peuvent ensuite influencer :

- logs ;
- en-têtes ;
- autorisation selon modules/contexte.

Ne pas construire un moteur métier complexe dans la configuration Apache si cette logique appartient à l'application.

# 112. Exclure les health checks des logs

Pour réduire le bruit :

```apache
SetEnvIf Request_URI "^/health$" dontlog
CustomLog ${APACHE_LOG_DIR}/access.log combined env=!dontlog
```

Cela peut être utile si un load balancer interroge la route toutes les secondes.

Mais garder une métrique séparée des health checks pour savoir si les sondes échouent.

# 113. Journaliser le temps de traitement

Un format de log personnalisé peut inclure la durée d'une requête.

Exemple conceptuel :

```apache
LogFormat "%h %l %u %t \"%r\" %>s %b %D" timed
CustomLog ${APACHE_LOG_DIR}/access.log timed
```

`%D` permet d'observer le temps de service en microsecondes dans la syntaxe historique d'Apache.

Vérifier la documentation de `mod_log_config` pour les champs exacts de la version utilisée.

# 114. Logs JSON

Pour intégration avec Loki/ELK/OpenSearch, un format structuré peut être utile.

Mais JSON exige un échappement correct des valeurs.

Ne pas construire naïvement :

```text
{"ua":"<User-Agent brut>"}
```

si le champ peut contenir des caractères qui cassent le JSON.

Utiliser les capacités de logging disponibles dans la version ou un agent de collecte capable de parser un format sûr et stable.

# 115. Vie privée des logs

Éviter de journaliser :

- mots de passe ;
- jetons `Authorization` ;
- cookies de session ;
- secrets de query string ;
- données personnelles non nécessaires.

Une URL comme :

```text
/reset?token=SECRET
```

peut finir dans des logs, historiques et outils d'analyse.

Concevoir les applications pour transmettre les secrets dans des emplacements appropriés et minimiser les journaux.

# 116. Rotation des certificats

ACME automatise l'émission et le renouvellement, mais il faut surveiller l'échec du mécanisme.

Contrôle :

```bash
systemctl list-timers | grep -i certbot
```

ou, selon installation :

```bash
systemctl status snap.certbot.renew.timer
```

La commande exacte dépend du mode d'installation.

Alerter plusieurs jours avant expiration est une défense complémentaire.

# 117. Vérifier l'expiration d'un certificat

Local :

```bash
openssl x509 -in /etc/letsencrypt/live/example.com/cert.pem -noout -dates
```

Distant :

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null \
  | openssl x509 -noout -enddate
```

Pour une supervision, calculer le nombre de jours restants et alerter bien avant l'expiration.

# 118. Chaîne de certificats

Un serveur doit présenter une chaîne appropriée jusqu'à une autorité reconnue par le client.

Avec Let's Encrypt, Certbot configure généralement :

```text
fullchain.pem
privkey.pem
```

`fullchain.pem` contient le certificat du serveur et les intermédiaires nécessaires.

Une chaîne incomplète peut fonctionner sur certains postes qui disposent déjà d'un intermédiaire en cache, mais échouer ailleurs.

# 119. Clé privée et permissions

Ne jamais :

```bash
chmod 644 /etc/ssl/private/example.key
```

simplement pour contourner une erreur.

Identifier quel processus doit lire la clé et utiliser la stratégie prévue par la distribution/ACME.

Ne pas copier la clé privée dans `/var/www`.

# 120. Rotation de logs et reload gracieux

Lorsqu'un fichier de log est renommé par `logrotate`, Apache doit rouvrir ses fichiers.

Les paquets de distribution intègrent normalement cette opération.

Ne pas inventer un cron parallèle qui :

```text
mv access.log access.old
```

sans signaler Apache, au risque de conserver le descripteur de l'ancien fichier.

# 121. Tests de charge

Outils possibles :

- `ab` pour des tests simples historiques ;
- `wrk` ;
- `hey` ;
- `k6` ;
- outils applicatifs spécialisés.

Ne pas lancer un benchmark agressif contre un système de production sans autorisation.

Mesurer au minimum :

```text
débit
p50/p95/p99
5xx
CPU
RAM
workers
backend
DB
```

# 122. Benchmark : ne pas confondre débit et qualité

Un serveur qui répond :

```text
100 000 req/s
```

sur un fichier de 20 octets ne prédit pas la capacité d'une application réelle.

Tester des scénarios représentatifs :

- TLS ;
- fichiers statiques ;
- API ;
- cache chaud/froid ;
- backend lent ;
- uploads ;
- concurrence réaliste.

# 123. Cache reverse proxy

`mod_cache` permet à Apache de mettre en cache des réponses.

Mais la mise en cache d'une application authentifiée peut exposer des données si la clé de cache ne distingue pas correctement les utilisateurs ou variantes.

Avant d'activer :

- comprendre `Cache-Control` ;
- `Vary` ;
- cookies ;
- authentification ;
- invalidation ;
- contenu privé/public.

Pour les cas complexes, un CDN ou cache dédié peut être plus simple à maîtriser.

# 124. `Cache-Control`

Réponse typique pour une ressource publique versionnée :

```text
Cache-Control: public, max-age=31536000, immutable
```

Pour une page utilisateur privée :

```text
Cache-Control: private, no-store
```

Ces valeurs dépendent de la sémantique applicative. Apache ne peut pas deviner correctement si une réponse contient une donnée personnelle.

# 125. Reverse proxy et streaming

Pour SSE, téléchargements ou réponses en flux, vérifier :

- buffering ;
- timeouts ;
- compression ;
- comportement du backend ;
- proxy intermédiaire ;
- idle timeout du load balancer.

Une requête longue n'est pas nécessairement « lente » au sens d'une panne : elle peut être un flux intentionnel.

# 126. Uploads volumineux

Chaîne de limites :

```text
navigateur
 ↓
CDN/LB
 ↓
Apache
 ↓
framework
 ↓
serveur applicatif
 ↓
stockage
```

Évaluer :

- limite du corps ;
- durée ;
- espace temporaire ;
- quotas ;
- antivirus si nécessaire ;
- extension/type MIME ;
- nom de fichier ;
- stockage hors `DocumentRoot`.

# 127. Fichiers envoyés par les utilisateurs

Ne jamais supposer qu'une extension :

```text
.jpg
```

garantit que le contenu est une image sûre.

Pour des uploads :

- générer un nom serveur ;
- stocker hors racine exécutable ;
- contrôler le contenu ;
- fixer le type de réponse ;
- interdire exécution CGI/PHP ;
- appliquer des quotas ;
- servir via un domaine séparé si le modèle de menace l'exige.

# 128. Séparer contenu statique et application

Architecture :

```text
/srv/myapp/public/static/
```

peut être servi directement par Apache, tandis que :

```text
/api/
```

est proxyfié vers l'application.

Exemple :

```apache
Alias /static/ /srv/myapp/static/

<Directory /srv/myapp/static>
    Require all granted
</Directory>

ProxyPass /api/ http://127.0.0.1:8000/api/
ProxyPassReverse /api/ http://127.0.0.1:8000/api/
```

# 129. Exclure un chemin du proxy

Selon la forme de configuration, un chemin statique doit être exclu avant une règle proxy générale.

Toujours consulter la documentation `ProxyPass` sur l'ordre et le matching, puis tester :

```bash
curl -I https://example.com/static/app.css
curl -I https://example.com/api/health
```

Les règles de proxy doivent rester lisibles : une forêt d'exceptions est souvent un signe que l'espace d'URL doit être repensé.

# 130. Versionner une configuration avec tests

Un dépôt d'infrastructure peut contenir :

```text
apache/
├── README.md
├── sites/
│   ├── example.com.conf
│   └── app.example.com.conf
├── conf/
│   └── hardening.conf
└── tests/
    ├── test-http.sh
    └── test-tls.sh
```

CI :

```text
lint
 ↓
construction image de test
 ↓
apache2ctl configtest
 ↓
start
 ↓
curl smoke tests
 ↓
tests TLS
```

# 131. Validation des en-têtes

Un test simple :

```bash
curl -sSI https://example.com/
```

Vérifier :

```text
HTTP/2 200
content-type
cache-control
strict-transport-security
x-content-type-options
content-security-policy si applicable
```

Ne pas cocher des en-têtes comme une checklist de conformité sans comprendre ce qu'ils protègent.

# 132. Scanner TLS

Des outils comme SSL Labs peuvent fournir un diagnostic externe pour un service public.

Ils ne voient cependant que :

- l'endpoint public ;
- les protocoles/ciphers exposés ;
- le certificat ;
- quelques propriétés HTTP.

Ils ne prouvent pas que l'application, le backend ou le système d'exploitation sont sûrs.

# 133. Scanner la configuration Apache

La revue doit chercher notamment :

```text
ancien contrôle d'accès Apache 2.2
Indexes involontaire
AllowOverride All inutile
ProxyRequests On
TLS obsolète
racine trop large
admin/status publics
secrets dans conf
modules inutiles
logs sensibles
```

Un scanner automatique aide, mais la compréhension de l'architecture reste nécessaire.

# 134. Migration Apache 2.2 → 2.4 : autorisation

Ancien :

```apache
Order allow,deny
Allow from all
```

Moderne :

```apache
Require all granted
```

Ancien :

```apache
Order deny,allow
Deny from all
Allow from 192.0.2.0/24
```

Moderne :

```apache
Require ip 192.0.2.0/24
```

Ne pas mélanger les deux modèles dans le même bloc sans comprendre `mod_access_compat`.

# 135. Migration : TLS

Supprimer les directives historiques qui mentionnent :

```text
SSLv2
SSLv3
RC4
EXPORT
LOW
```

Puis reconstruire la politique en fonction :

- version Apache ;
- OpenSSL ;
- politique distribution ;
- navigateurs/clients ;
- recommandations de sécurité en vigueur.

# 136. Migration : `mod_php` vers PHP-FPM

Ancien modèle :

```text
Apache + mod_php + prefork
```

Modèle moderne fréquent :

```text
Apache event
   ↓ FastCGI
PHP-FPM
```

Plan :

1. installer FPM ;
2. activer `proxy_fcgi` ;
3. configurer le socket/pool ;
4. tester PHP ;
5. basculer le MPM si compatible ;
6. désactiver le module PHP ancien ;
7. mesurer mémoire et performances.

# 137. Migration : `.htaccess` vers VirtualHost

Si l'on contrôle le serveur, déplacer progressivement :

```text
.htaccess
   ↓
bloc <Directory> dans site/conf
```

Avantages :

- validation centralisée ;
- moins de recherche de fichiers ;
- droits de configuration plus clairs ;
- configuration versionnable.

Conserver `.htaccess` lorsqu'une délégation de configuration est réellement requise.

# 138. Apache et systemd

Commandes :

```bash
systemctl status apache2
systemctl start apache2
systemctl stop apache2
systemctl restart apache2
systemctl reload apache2
systemctl enable apache2
```

Voir l'unité :

```bash
systemctl cat apache2
```

Voir le journal du boot courant :

```bash
journalctl -u apache2 -b --no-pager
```

Voir aussi [[Initialisation système et des services]].

# 139. Ne pas modifier directement l'unité du paquet

Pour une surcharge systemd :

```bash
sudo systemctl edit apache2
```

Cela crée un drop-in sous :

```text
/etc/systemd/system/apache2.service.d/
```

Éviter de modifier directement un fichier sous :

```text
/lib/systemd/system/
```

ou :

```text
/usr/lib/systemd/system/
```

car une mise à jour du paquet peut le remplacer.

# 140. Ressources systemd

On peut ajouter des limites au service via un drop-in lorsque le besoin est justifié.

Exemples de familles de directives :

```text
LimitNOFILE=
MemoryMax=
TasksMax=
```

Mais appliquer une limite sans comprendre la charge peut créer une panne artificielle.

Mesurer et tester sous charge avant production.

# 141. Démarrage après réseau

Un reverse proxy vers un service local n'a pas besoin de garantir que le backend est déjà prêt au moment précis où Apache démarre : il peut retourner temporairement 503/502.

Pour une orchestration stricte, il faut coordonner les unités systemd ou utiliser readiness/health checks.

Ne pas ajouter des dépendances `After=network-online.target` à tous les services par réflexe.

# 142. DNS des backends

Avec :

```apache
ProxyPass / http://backend.internal:8000/
```

la résolution DNS du backend et le comportement de réutilisation des connexions doivent être compris.

Dans des environnements où l'adresse backend change souvent, vérifier que le mécanisme Apache choisi correspond à la découverte de service attendue.

Pour Kubernetes, on utilise généralement le Service stable plutôt qu'une IP de Pod.

# 143. IPv6

Écoute :

```bash
sudo ss -ltnp | grep apache2
```

DNS :

```bash
dig A example.com
dig AAAA example.com
```

Un site peut fonctionner en IPv4 et échouer en IPv6 si un enregistrement AAAA pointe vers une machine mal configurée.

Tester explicitement :

```bash
curl -4 -I https://example.com/
curl -6 -I https://example.com/
```

# 144. Dual stack et firewall

Ouvrir 443 en IPv4 ne signifie pas toujours qu'IPv6 suit selon le pare-feu utilisé.

Vérifier :

```bash
sudo nft list ruleset
```

ou l'outil de pare-feu choisi.

Ne pas publier un AAAA avant de confirmer routage, firewall, Apache et certificat sur IPv6.

# 145. DNS CAA

Un enregistrement CAA peut limiter quelles autorités sont autorisées à émettre un certificat pour le domaine.

Exemple conceptuel :

```dns
example.com. CAA 0 issue "letsencrypt.org"
```

Avant d'ajouter une politique CAA stricte, inventorier toutes les autorités utilisées par l'organisation, y compris les services CDN/cloud.

# 146. Certificats wildcard

Un certificat :

```text
*.example.com
```

ne couvre généralement pas :

```text
example.com
```

ni :

```text
a.b.example.com
```

selon les règles de correspondance usuelles.

ACME wildcard nécessite typiquement une validation DNS-01.

# 147. TLS et backend

Si Apache termine TLS puis communique avec un backend local :

```text
client --TLS--> Apache --HTTP--> 127.0.0.1
```

le segment local n'est pas chiffré, mais il reste confiné à la machine si le backend écoute uniquement sur loopback.

Entre machines :

```text
Apache -- réseau --> backend
```

évaluer TLS interne ou un réseau de confiance selon le modèle de menace.

# 148. Vérification TLS du backend

Lorsque le reverse proxy utilise HTTPS vers un backend, il faut comprendre comment Apache vérifie le certificat du backend.

Ne jamais désactiver durablement la vérification uniquement pour faire fonctionner un certificat incorrect.

Installer la CA interne appropriée ou utiliser un certificat correspondant au nom du backend.

# 149. Headers `Host` et sécurité

Le client contrôle l'en-tête `Host` de sa requête HTTP.

Une application ne doit pas l'utiliser sans validation pour :

- construire des liens sensibles ;
- générer des reset password URLs ;
- décider d'un tenant ;
- calculer une origine de confiance.

Apache sélectionne le VirtualHost selon le nom, mais l'application doit aussi avoir sa propre liste d'hôtes autorisés si nécessaire.

# 150. VirtualHost de repli

Le premier VirtualHost d'une adresse/port reçoit les requêtes dont le `Host` ne correspond à aucun autre site.

Pour réduire les comportements inattendus, on peut définir un site de repli qui ne sert aucune application sensible.

Exemple :

```apache
<VirtualHost *:80>
    ServerName default.invalid
    <Location />
        Require all denied
    </Location>
</VirtualHost>
```

À adapter à la stratégie locale et à l'ordre de chargement.

# 151. Ne pas faire confiance à `ServerName` comme contrôle d'accès

`ServerName` sert principalement au routage/identité du VirtualHost.

Ce n'est pas une ACL.

Pour protéger `/admin`, utiliser :

```apache
Require ip ...
```

et/ou une authentification, un VPN, un firewall, etc.

# 152. Séparer frontal public et interface d'administration

Architecture préférable :

```text
Internet → app.example.com
VPN      → admin.internal.example
```

plutôt qu'une interface admin totalement publique protégée uniquement par une URL difficile à deviner.

Une défense en profondeur peut combiner :

- VPN ;
- réseau source ;
- MFA applicatif ;
- authentification forte ;
- logs/audit.

# 153. Sauvegarde et restauration de certificats ACME

Selon l'outil ACME, le répertoire contient plus qu'un simple certificat :

- clés ;
- comptes ACME ;
- historique ;
- liens ;
- paramètres de renouvellement.

Ne pas copier partiellement `/etc/letsencrypt/live` sans comprendre les liens vers `archive` et la structure de Certbot.

Tester la restauration dans une machine isolée.

# 154. Horloge système et TLS

Une date système incorrecte peut provoquer :

- certificats « pas encore valides » ;
- certificats considérés expirés ;
- problèmes de logs ;
- erreurs de jetons applicatifs.

Vérifier :

```bash
timedatectl
```

Voir la synchronisation configurée par la distribution.

# 155. Nom de serveur global

Apache peut avertir s'il ne connaît pas son nom canonique global.

On peut définir un `ServerName` global approprié dans un fragment sous `conf-available` si nécessaire.

Ne pas ajouter :

```apache
ServerName localhost
```

sans réfléchir simplement pour faire disparaître un warning sur une machine qui doit avoir un nom canonique spécifique.

# 156. `apache2ctl -S` est prioritaire

Lorsque plusieurs sites existent, cette commande doit devenir un réflexe :

```bash
sudo apache2ctl -S
```

Elle répond mieux que :

```text
« j'ai modifié le bon fichier, pourquoi ça ne marche pas ? »
```

car elle montre **ce qu'Apache a réellement chargé**.

# 157. `apache2ctl -t -D DUMP_RUN_CFG`

Selon la version, des options de dump permettent d'inspecter certains paramètres calculés.

Exemple :

```bash
sudo apache2ctl -t -D DUMP_RUN_CFG
```

Utiliser aussi :

```bash
sudo apache2ctl -t -D DUMP_VHOSTS
sudo apache2ctl -t -D DUMP_MODULES
```

ou les commandes équivalentes disponibles.

# 158. Configuration conditionnelle `<IfModule>`

Exemple :

```apache
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
</IfModule>
```

Cela évite une erreur si le module est absent, mais peut aussi masquer une configuration importante qui n'est jamais appliquée.

Pour une exigence de sécurité critique, il peut être préférable de **faire échouer explicitement le déploiement** si le module attendu n'est pas chargé.

# 159. `<IfVersion>`

`mod_version` permet des conditions selon la version Apache.

Utile lors de migrations temporaires, mais une configuration qui accumule :

```text
si 2.2
si 2.4.20
si 2.4.50
```

devient difficile à maintenir.

Préférer une base de versions supportées clairement définie.

# 160. Modules tiers

Un module tiers s'exécute dans le processus Apache et peut donc affecter :

- stabilité ;
- sécurité ;
- mémoire ;
- compatibilité lors d'une mise à jour.

Avant installation :

- vérifier maintenance ;
- paquet distribution ;
- politique de sécurité ;
- compatibilité 2.4 ;
- besoin réel.

Un reverse proxy vers un service séparé est parfois plus sûr qu'un module natif non maintenu.

# 161. Ne pas compiler Apache sans raison

Sur une distribution serveur :

```bash
apt install apache2
```

est généralement préférable à :

```text
./configure
make
make install
```

car le paquet fournit :

- mises à jour de sécurité ;
- unité systemd ;
- intégration logrotate ;
- arborescence de configuration ;
- dépendances ;
- conventions de la distribution.

Compiler peut être justifié pour un besoin spécifique, mais crée une responsabilité de maintenance supplémentaire.

# 162. Apache n'est pas un WAF complet

Des modules de filtrage existent, notamment ModSecurity dans certains environnements, mais installer un WAF ne corrige pas :

- injection SQL ;
- XSS ;
- contrôle d'accès applicatif ;
- secrets exposés ;
- dépendances vulnérables.

Un WAF est une couche complémentaire, avec règles, faux positifs, mises à jour et surveillance.

# 163. Rate limiting

Apache possède divers modules et mécanismes de limitation, mais pour des politiques sophistiquées par utilisateur/API, l'application, un API gateway, CDN ou WAF est souvent mieux placé.

Une limitation basée uniquement sur IP peut pénaliser :

- NAT d'entreprise ;
- universités ;
- opérateurs mobiles ;
- proxys partagés.

Concevoir selon l'identité réellement disponible.

# 164. Défense contre les bots

Le `User-Agent` est librement choisi par le client.

Une règle :

```text
bloquer si User-Agent = badbot
```

est facile à contourner.

La lutte contre l'abus combine plutôt :

- quotas ;
- authentification ;
- réputation ;
- analyse comportementale ;
- CDN/WAF ;
- limites applicatives.

# 165. Compression et secrets

La compression de réponses mêlant secrets et entrées contrôlées peut avoir des implications cryptographiques dans certains scénarios historiques/avancés.

Ne pas désactiver toute compression par réflexe, mais éviter de compresser des réponses sensibles lorsque le modèle de menace l'exige et suivre les recommandations de la pile applicative.

# 166. En-têtes hop-by-hop

Dans une chaîne proxy, certains en-têtes ont une portée par connexion et ne doivent pas être transmis comme des en-têtes end-to-end.

Apache et ses modules proxy gèrent les règles du protocole, mais une configuration manuelle d'en-têtes doit respecter cette distinction.

Exemples historiques de concepts hop-by-hop :

```text
Connection
Upgrade
TE
```

Consulter les RFC HTTP actuelles pour les règles précises.

# 167. Canonicalisation des URL

Choisir une forme canonique :

```text
https://example.com/
```

plutôt que de servir simultanément sans stratégie :

```text
http://example.com
https://example.com
https://www.example.com
http://www.example.com
```

Rediriger les variantes vers la forme voulue.

Cela améliore :

- cohérence ;
- cookies ;
- SEO ;
- caches ;
- liens absolus ;
- observabilité.

# 168. Redirection `www` ou non-`www`

Exemple vers le domaine nu :

```apache
<VirtualHost *:80>
    ServerName www.example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

Pour HTTPS, le VirtualHost `www.example.com` doit encore disposer d'un certificat valide si le navigateur se connecte en HTTPS avant de recevoir la redirection.

# 169. Redirections et codes 301/308

`301` est historiquement très utilisé.

`308 Permanent Redirect` conserve explicitement la méthode et le corps.

Pour une API `POST`, la sémantique de redirection est importante.

Ne pas choisir un code uniquement par habitude ; vérifier le comportement client voulu.

# 170. `HEAD`

Une requête `HEAD` demande les mêmes métadonnées que `GET` sans corps de réponse.

Tester :

```bash
curl -I https://example.com/
```

Une application/proxy doit gérer correctement `HEAD` ; le bloquer peut casser des checks, crawlers ou caches.

# 171. `OPTIONS`

`OPTIONS` peut être utilisé pour :

- découvrir des capacités ;
- préflight CORS ;
- WebDAV selon contexte.

Ne pas le bloquer arbitrairement si une API web cross-origin en dépend.

# 172. WebDAV

Apache fournit des modules DAV.

WebDAV autorise potentiellement l'écriture distante de ressources.

Si non utilisé :

- ne pas activer les modules/configurations DAV ;
- supprimer les endpoints inutiles.

Les correctifs de sécurité 2.4.68 incluent d'ailleurs un problème touchant `mod_dav_fs`, ce qui illustre l'intérêt de réduire la surface de modules.

# 173. FTP proxy

`mod_proxy_ftp` existe historiquement, mais son usage est aujourd'hui spécialisé.

La version 2.4.68 corrige également plusieurs vulnérabilités dans ce module.

Si l'on n'utilise pas le proxy FTP : ne pas l'activer.

# 174. Inventaire des modules

Créer périodiquement :

```bash
apache2ctl -M | sort
```

Puis classer :

```text
nécessaire
compris
propriétaire/tiers
inutilisé
à supprimer
```

Chaque module supplémentaire est du code chargé dans ou autour du serveur.

# 175. Inventaire des sites

```bash
ls -1 /etc/apache2/sites-enabled/
```

Pour chaque site :

- propriétaire ;
- domaine ;
- destination ;
- certificat ;
- logs ;
- backend ;
- date de fin prévue ;
- criticité.

Les VirtualHost oubliés deviennent souvent des surfaces d'attaque non maintenues.

# 176. Domaines expirés

Un ancien VirtualHost pour un domaine que l'organisation ne contrôle plus peut devenir dangereux si le domaine est racheté ou repointe ailleurs.

Lors d'un retrait de service :

1. retirer DNS ;
2. retirer certificat/ACME ;
3. désactiver le VirtualHost ;
4. supprimer backend ;
5. nettoyer monitoring ;
6. conserver l'archive nécessaire ;
7. documenter la fin de vie.

# 177. Sites temporaires

Pour un environnement de staging :

```text
staging.example.com
```

ne pas supposer que « personne ne connaît l'URL ».

Protéger par :

- VPN ;
- IP allowlist ;
- authentification ;
- identité de test ;
- exclusion des données réelles sensibles.

# 178. Robots et staging

`robots.txt` n'est pas un mécanisme de sécurité.

Il indique une préférence aux crawlers coopératifs.

Un site staging contenant des données confidentielles doit être inaccessible aux utilisateurs non autorisés, pas seulement marqué :

```text
Disallow: /
```

# 179. Secrets dans les URLs

Éviter :

```text
https://example.com/download?api_key=SECRET
```

Une URL peut apparaître dans :

- logs Apache ;
- proxy ;
- analytics ;
- historique navigateur ;
- referer selon politique ;
- screenshots ;
- monitoring.

Préférer des mécanismes d'authentification appropriés.

# 180. Redaction des logs

Si une application existante place des secrets en query string, une stratégie temporaire peut chercher à ne pas journaliser la query complète.

Mais la vraie correction consiste à modifier le protocole applicatif.

Ne pas inventer une regex de masquage fragile comme unique protection.

# 181. Fichiers statiques avec hash

Pipeline frontend :

```text
main.js
  ↓ build
main.6c98d2.js
```

Puis :

```text
Cache-Control: public, max-age=31536000, immutable
```

Le HTML référence le nouveau nom à chaque déploiement.

Cela simplifie fortement le cache navigateur/CDN.

# 182. `sendfile`

Apache peut s'appuyer sur des mécanismes noyau pour servir efficacement des fichiers.

Sur certains filesystems réseau ou configurations particulières, des directives comme `EnableSendfile` peuvent influencer le comportement.

Ne pas modifier les valeurs uniquement à partir d'un benchmark local : vérifier le stockage réel (NFS, overlay, VM, conteneur).

# 183. NFS et fichiers web

Avec NFS :

- latence ;
- cohérence ;
- locks ;
- attributs ;
- `sendfile` ;
- indisponibilité réseau

peuvent changer le comportement d'un serveur statique.

Un stockage objet + CDN peut être plus adapté pour de gros ensembles d'actifs immuables.

# 184. Reverse proxy vers conteneurs

Éviter de proxyfier vers une IP de conteneur éphémère codée en dur :

```apache
ProxyPass / http://172.18.0.5:8000/
```

Préférer un nom/service stable fourni par l'orchestrateur ou publier le backend sur un socket/port stable géré par l'hôte.

# 185. Docker Compose

Architecture possible :

```text
Apache hôte
  ↓ 127.0.0.1:18000
Docker publish
  ↓
app:8000
```

Publier le backend seulement sur loopback :

```text
127.0.0.1:18000:8000
```

plutôt que sur toutes les interfaces si seul Apache doit y accéder.

Voir [[Docker]].

# 186. Backend et health checks

Un endpoint `/health` doit être :

- rapide ;
- non destructif ;
- peu dépendant d'éléments non indispensables ;
- suffisamment représentatif de la disponibilité.

Ne pas faire :

```text
/health → exécute une migration SQL
```

ou une opération coûteuse à chaque seconde.

# 187. Draining lors d'un déploiement

Avec plusieurs backends :

```text
BalancerMember A
BalancerMember B
```

un déploiement propre peut :

1. retirer A du trafic ;
2. attendre les requêtes en cours ;
3. mettre à jour A ;
4. vérifier readiness ;
5. remettre A ;
6. répéter pour B.

Cela évite une coupure globale.

# 188. Sessions et load balancing

Si l'application stocke sa session en mémoire locale, le load balancing peut exiger une affinité (« sticky session »).

Une architecture plus robuste stocke souvent la session dans :

- base de données ;
- Redis ;
- token signé selon design ;
- autre stockage partagé.

Cela facilite le remplacement des backends.

# 189. TLS termination et client IP

Lorsqu'un load balancer externe termine TLS, Apache voit parfois :

```text
source = load balancer
proto = HTTP
```

Il faut alors transmettre de manière sûre :

```text
client IP
proto public
host public
```

et configurer la confiance dans le load balancer.

# 190. PROXY protocol

Certains load balancers transmettent l'adresse client via le PROXY protocol au niveau connexion plutôt que par en-tête HTTP.

La prise en charge dépend des modules/version et de l'architecture.

Ne pas envoyer PROXY protocol vers un port qui n'est pas configuré pour le comprendre : la première ligne de la connexion serait interprétée comme du HTTP/TLS invalide.

# 191. TLS passthrough

Deux modèles :

```text
TLS termination:
client --TLS--> LB --HTTP/TLS--> Apache
```

```text
TLS passthrough:
client --TLS------------------> Apache
       LB ne déchiffre pas
```

Les conséquences diffèrent pour :

- certificats ;
- SNI ;
- inspection WAF ;
- IP client ;
- HTTP routing.

# 192. Apache comme unique frontal ou composant intermédiaire

Ne pas superposer :

```text
Caddy → Nginx → Apache → HAProxy → app
```

sans nécessité claire.

Chaque couche ajoute :

- configuration ;
- timeout ;
- logs ;
- headers ;
- certificats ;
- points de panne.

Documenter la responsabilité de chaque proxy.

# 193. Diagramme d'architecture minimal

```text
              ┌──────────────┐
Internet ────▶│ Apache :443  │
              └──────┬───────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   /static/       /api/        /admin/
   fichiers      backend       backend
                :8000          :9000
```

Le diagramme doit montrer :

- terminaison TLS ;
- réseaux ;
- ports ;
- backends ;
- frontières de confiance.

# 194. Documentation d'un VirtualHost

Pour chaque site, documenter :

```text
Nom public:
Propriétaire:
DNS:
Adresse(s):
Certificat/ACME:
DocumentRoot ou backend:
Compte backend:
Logs:
Monitoring:
Sauvegarde:
Date de revue:
```

Cela réduit fortement le coût de maintenance après plusieurs années.

# 195. Exemple complet : site statique HTTPS

Fichier :

```text
/etc/apache2/sites-available/example.com.conf
```

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    Redirect permanent / https://example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com

    DocumentRoot /srv/www/example.com/public

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    Protocols h2 http/1.1

    <Directory /srv/www/example.com/public>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Strict-Transport-Security "max-age=86400"

    ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined
</VirtualHost>
```

Le `max-age` HSTS volontairement court dans cet exemple permet un premier déploiement prudent. Augmenter après validation de la stratégie HTTPS.

# 196. Activer le site complet

Modules :

```bash
sudo a2enmod ssl http2 headers
```

Site :

```bash
sudo a2ensite example.com.conf
```

Validation :

```bash
sudo apache2ctl configtest
sudo apache2ctl -S
```

Reload :

```bash
sudo systemctl reload apache2
```

Tests :

```bash
curl -I http://example.com/
curl -I --http2 https://example.com/
```

# 197. Exemple complet : reverse proxy HTTPS

```apache
<VirtualHost *:80>
    ServerName app.example.com
    Redirect permanent / https://app.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName app.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/app.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/app.example.com/privkey.pem

    Protocols h2 http/1.1

    ProxyRequests Off
    ProxyPreserveHost On

    RequestHeader set X-Forwarded-Proto "https"

    ProxyPass        / http://127.0.0.1:8000/ timeout=30
    ProxyPassReverse / http://127.0.0.1:8000/

    Header always set X-Content-Type-Options "nosniff"

    ErrorLog ${APACHE_LOG_DIR}/app.example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/app.example.com-access.log combined
</VirtualHost>
```

Modules :

```bash
sudo a2enmod ssl http2 headers proxy proxy_http
```

# 198. Tester le backend avant Apache

Avant :

```bash
curl -v http://127.0.0.1:8000/
```

Puis Apache localement avec résolution forcée :

```bash
curl --resolve app.example.com:443:127.0.0.1 \
  -v https://app.example.com/
```

Puis depuis le réseau extérieur.

Cette progression localise rapidement la couche défaillante.

# 199. Exemple : application sous `/app/`

```apache
ProxyPass        /app/ http://127.0.0.1:8000/
ProxyPassReverse /app/ http://127.0.0.1:8000/
```

L'application doit savoir qu'elle est éventuellement montée sous un préfixe `/app/`.

Sinon elle peut générer des liens :

```text
/static/app.css
```

au lieu de :

```text
/app/static/app.css
```

Le problème appartient alors au contrat entre proxy et application.

# 200. Exemple : refuser l'accès à un répertoire

```apache
<Directory /srv/www/example.com/private>
    Require all denied
</Directory>
```

Mais le meilleur design consiste souvent à ne pas placer `private/` sous une arborescence servie.

# 201. Exemple : administration locale seulement

```apache
<Location /server-status>
    SetHandler server-status
    Require local
</Location>
```

Tester depuis la machine :

```bash
curl http://127.0.0.1/server-status?auto
```

Un tunnel SSH peut permettre une consultation distante sans exposer directement la route au réseau public.

# 202. Exemple : IP + authentification

Exiger les deux :

```apache
<Location /admin>
    AuthType Basic
    AuthName "Admin"
    AuthUserFile /etc/apache2/htpasswd-admin

    <RequireAll>
        Require ip 192.0.2.0/24
        Require valid-user
    </RequireAll>
</Location>
```

L'ordre logique est explicite grâce à `<RequireAll>`.

# 203. Exemple : IP ou utilisateur

```apache
<RequireAny>
    Require ip 192.0.2.0/24
    Require valid-user
</RequireAny>
```

Le choix `AND`/`OR` doit suivre le besoin de sécurité ; ne pas le laisser implicite.

# 204. TP 1 — servir un site statique

Objectifs :

1. installer Apache ;
2. créer `/srv/www/lab.example.test/public` ;
3. créer `index.html` ;
4. créer un VirtualHost port 80 ;
5. ajouter une entrée `/etc/hosts` ;
6. activer le site ;
7. exécuter `configtest` ;
8. recharger ;
9. tester avec `curl` ;
10. examiner les logs.

Contraintes :

```text
Indexes désactivé
AllowOverride None
Require all granted
logs dédiés
```

# 205. TP 1 — validation

Le résultat doit passer :

```bash
apache2ctl configtest
apache2ctl -S
curl -f http://lab.example.test/
```

Créer ensuite un répertoire sans `index.html` :

```text
/srv/www/lab.example.test/public/files/
```

Vérifier que l'accès ne produit **pas** une liste automatique si `Indexes` est désactivé.

# 206. TP 2 — reverse proxy

Lancer un petit backend de laboratoire :

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Configurer Apache :

```apache
ProxyRequests Off
ProxyPass /app/ http://127.0.0.1:8000/
ProxyPassReverse /app/ http://127.0.0.1:8000/
```

Activer :

```bash
sudo a2enmod proxy proxy_http
```

Puis comparer :

```bash
curl http://127.0.0.1:8000/
curl http://lab.example.test/app/
```

# 207. TP 2 — provoquer une panne

Arrêter le backend.

Observer :

```bash
curl -v http://lab.example.test/app/
```

Puis :

```bash
sudo tail -n 30 /var/log/apache2/lab-error.log
```

Identifier le code produit par Apache et le message d'erreur correspondant.

Redémarrer le backend et vérifier le retour à la normale.

# 208. TP 3 — HTTPS de laboratoire

Pour Internet réel, utiliser ACME.

Pour un laboratoire fermé, créer une CA locale ou certificat de test selon le cours PKI choisi.

Configurer :

```apache
SSLEngine on
SSLCertificateFile ...
SSLCertificateKeyFile ...
```

Puis tester :

```bash
openssl s_client -connect lab.example.test:443 -servername lab.example.test
```

et :

```bash
curl -vk https://lab.example.test/
```

`-k` n'est acceptable ici que parce que la confiance locale n'est pas installée.

# 209. TP 4 — HTTP/2

Activer :

```bash
sudo a2enmod http2
```

Configurer :

```apache
Protocols h2 http/1.1
```

Vérifier :

```bash
curl -sv --http2 https://lab.example.test/ -o /dev/null
```

Comparer le protocole négocié avant et après activation.

# 210. TP 5 — droits Unix et `403`

Retirer volontairement le droit de traversée d'un répertoire parent dans un laboratoire.

Observer le `403` et le log.

Diagnostiquer avec :

```bash
namei -l /srv/www/lab.example.test/public/index.html
```

Corriger **le droit minimal nécessaire**.

Ne jamais utiliser `chmod -R 777` dans ce TP.

# 211. TP 6 — mauvais VirtualHost

Créer deux VirtualHost sur le même port.

Utiliser :

```bash
apache2ctl -S
```

Puis tester :

```bash
curl -H 'Host: site-a.example.test' http://192.0.2.10/
curl -H 'Host: site-b.example.test' http://192.0.2.10/
```

Tester enfin un nom inconnu et identifier le VirtualHost par défaut.

# 212. TP 7 — observabilité

Activer `mod_status` en local.

Générer quelques requêtes :

```bash
for i in $(seq 1 50); do curl -s http://lab.example.test/ >/dev/null; done
```

Consulter :

```bash
curl http://127.0.0.1/server-status?auto
```

Identifier les métriques qui seraient utiles pour une alerte.

# 213. TP 8 — logs de latence

Créer un `LogFormat` contenant `%D`.

Effectuer :

```bash
curl http://lab.example.test/
```

Puis :

```bash
tail -n 5 /var/log/apache2/lab-access.log
```

Comparer une ressource statique à une route proxy volontairement plus lente.

# 214. TP 9 — certificat et SNI

Configurer deux VirtualHost HTTPS avec deux noms.

Tester :

```bash
openssl s_client -connect 192.0.2.10:443 -servername a.example.test </dev/null
```

Puis :

```bash
openssl s_client -connect 192.0.2.10:443 -servername b.example.test </dev/null
```

Vérifier que le certificat présenté change comme prévu.

# 215. TP 10 — durcissement

À partir d'une configuration volontairement mauvaise :

```apache
Options Indexes FollowSymLinks ExecCGI
AllowOverride All
ProxyRequests On
ServerSignature On
```

Identifier chaque risque ou coût de maintenance.

Construire la version minimale répondant uniquement au besoin fonctionnel.

# 216. Checklist avant mise en production

## Système

- [ ] distribution maintenue ;
- [ ] mises à jour de sécurité appliquées ;
- [ ] heure synchronisée ;
- [ ] pare-feu vérifié ;
- [ ] sauvegardes définies ;
- [ ] stockage des logs surveillé.

## Apache

- [ ] `apache2ctl configtest` passe ;
- [ ] `apache2ctl -S` est compris ;
- [ ] modules inutiles désactivés ;
- [ ] `ProxyRequests Off` sauf besoin explicite de forward proxy ;
- [ ] `Indexes` désactivé sauf besoin ;
- [ ] `AllowOverride None` lorsque possible ;
- [ ] autorisation Apache 2.4 avec `Require` ;
- [ ] MPM choisi consciemment.

## TLS

- [ ] certificat valide pour tous les noms ;
- [ ] clé privée protégée ;
- [ ] chaîne complète ;
- [ ] renouvellement automatisé ;
- [ ] monitoring d'expiration ;
- [ ] protocoles obsolètes non activés ;
- [ ] HSTS seulement après validation.

## Application / proxy

- [ ] backend non exposé inutilement ;
- [ ] health check ;
- [ ] timeouts cohérents ;
- [ ] headers de proxy et confiance documentés ;
- [ ] uploads hors zone exécutable ;
- [ ] aucune donnée secrète sous `DocumentRoot`.

## Observabilité

- [ ] logs accès/erreur ;
- [ ] rotation ;
- [ ] alertes 5xx ;
- [ ] latence ;
- [ ] saturation workers ;
- [ ] disponibilité backend ;
- [ ] certificat ;
- [ ] disque.

# 217. Erreurs fréquentes

> [!danger] Erreur 1 — recopier une configuration TLS de 2010
> Toute présence de `SSLv2`, `SSLv3`, `RC4`, `EXPORT` ou `LOW` doit déclencher une revue immédiate.

> [!danger] Erreur 2 — utiliser `Order allow,deny`
> La syntaxe moderne Apache 2.4 est fondée sur `Require`.

> [!danger] Erreur 3 — `chmod 777`
> Cela masque le diagnostic et élargit inutilement les droits.

> [!danger] Erreur 4 — `ProxyRequests On`
> Un reverse proxy n'en a pas besoin.

> [!danger] Erreur 5 — `AllowOverride All` partout
> Cela délègue beaucoup de configuration et complique le raisonnement.

> [!danger] Erreur 6 — laisser `Indexes`
> Un répertoire sans index peut alors devenir une liste publique de fichiers.

> [!danger] Erreur 7 — placer `.env` sous `DocumentRoot`
> La séparation structurelle est plus robuste qu'une règle de blocage ajoutée après coup.

> [!danger] Erreur 8 — recharger sans `configtest`
> Une faute de syntaxe peut rendre le service indisponible.

> [!danger] Erreur 9 — diagnostiquer uniquement côté Apache
> Pour un reverse proxy, tester le backend directement.

> [!danger] Erreur 10 — considérer le numéro de version Ubuntu comme identique au suivi amont
> Les distributions rétroportent des correctifs.

# 218. Méthode de diagnostic en 12 commandes

```bash
# 1. Service
systemctl status apache2 --no-pager

# 2. Journal
journalctl -u apache2 -n 100 --no-pager

# 3. Syntaxe
sudo apache2ctl configtest

# 4. VirtualHost
sudo apache2ctl -S

# 5. Modules
sudo apache2ctl -M

# 6. Écoute
sudo ss -ltnp | grep apache2

# 7. Test local HTTP
curl -v http://127.0.0.1/

# 8. Test nom forcé
curl --resolve example.com:443:127.0.0.1 -vk https://example.com/

# 9. DNS
dig example.com A AAAA

# 10. TLS
openssl s_client -connect example.com:443 -servername example.com </dev/null

# 11. Permissions
namei -l /srv/www/example.com/public/index.html

# 12. Backend
curl -v http://127.0.0.1:8000/
```

Adapter les chemins et ports au service réel.

# 219. Commandes de référence

## Paquet

```bash
sudo apt install apache2
apt policy apache2
```

## Service

```bash
systemctl status apache2
sudo systemctl reload apache2
sudo systemctl restart apache2
```

## Validation

```bash
sudo apache2ctl configtest
sudo apache2ctl -S
sudo apache2ctl -M
sudo apache2ctl -V
```

## Sites

```bash
sudo a2ensite example.com.conf
sudo a2dissite example.com.conf
```

## Modules

```bash
sudo a2enmod ssl
sudo a2dismod autoindex
```

## Configurations

```bash
sudo a2enconf nom
sudo a2disconf nom
```

## Réseau

```bash
sudo ss -ltnp
curl -v http://127.0.0.1/
```

## Logs

```bash
journalctl -u apache2
sudo tail -f /var/log/apache2/error.log
```

# 220. Résumé conceptuel

```text
Apache 2.4
│
├── écoute
│   └── Listen 80/443
│
├── routage
│   └── VirtualHost + ServerName
│
├── fichiers
│   └── DocumentRoot + Directory + Require
│
├── modules
│   ├── ssl
│   ├── http2
│   ├── proxy
│   ├── headers
│   └── ...
│
├── TLS
│   ├── certificat
│   ├── clé privée
│   ├── ACME
│   └── protocoles modernes
│
├── dynamique
│   ├── PHP-FPM
│   ├── WSGI
│   └── reverse proxy
│
├── exploitation
│   ├── configtest
│   ├── graceful reload
│   ├── logs
│   └── status/monitoring
│
└── sécurité
    ├── moindre privilège
    ├── modules minimaux
    ├── pas d'Indexes involontaire
    ├── pas d'open proxy
    ├── secrets hors DocumentRoot
    └── mises à jour
```

# 221. Sources officielles

## Apache HTTP Server

- Documentation Apache HTTP Server 2.4 : https://httpd.apache.org/docs/2.4/
- Téléchargement et version stable : https://httpd.apache.org/download.cgi
- Vulnérabilités Apache HTTP Server 2.4 : https://httpd.apache.org/security/vulnerabilities_24.html
- Virtual Hosts : https://httpd.apache.org/docs/2.4/vhosts/
- Access Control : https://httpd.apache.org/docs/2.4/howto/access.html
- `mod_authz_core` : https://httpd.apache.org/docs/2.4/mod/mod_authz_core.html
- `mod_ssl` : https://httpd.apache.org/docs/2.4/mod/mod_ssl.html
- HTTP/2 : https://httpd.apache.org/docs/2.4/howto/http2.html
- `mod_http2` : https://httpd.apache.org/docs/2.4/mod/mod_http2.html
- Reverse Proxy Guide : https://httpd.apache.org/docs/2.4/howto/reverse_proxy.html
- `mod_proxy` : https://httpd.apache.org/docs/2.4/mod/mod_proxy.html
- `mod_proxy_http2` : https://httpd.apache.org/docs/2.4/mod/mod_proxy_http2.html
- `mod_headers` : https://httpd.apache.org/docs/2.4/mod/mod_headers.html
- `mod_rewrite` : https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html
- `mod_log_config` : https://httpd.apache.org/docs/2.4/mod/mod_log_config.html
- `mod_status` : https://httpd.apache.org/docs/2.4/mod/mod_status.html
- Security Tips : https://httpd.apache.org/docs/2.4/misc/security_tips.html

## Ubuntu / Debian

- Ubuntu Server — installation Apache2 : https://ubuntu.com/server/docs/how-to/web-services/install-apache2/
- Ubuntu Server — configuration Apache2 : https://documentation.ubuntu.com/server/how-to/web-services/configure-apache2-settings/
- Ubuntu Server — modules Apache2 : https://ubuntu.com/server/docs/how-to-use-apache2-modules/
- Ubuntu Server — certificats TLS / Certbot : https://ubuntu.com/server/docs/how-to/security/obtain-tls-certificates/
- Debian manpage `apache2ctl` : https://manpages.debian.org/apache2/apache2ctl.8
- Debian manpage `a2ensite` : https://manpages.debian.org/apache2/a2ensite.8

# 222. À retenir

> [!summary]
> 1. Apache 2.4 est un serveur HTTP **modulaire** ; comprendre les modules actifs est essentiel.
> 2. Sur Debian/Ubuntu, travailler avec `sites-available`, `mods-available`, `a2ensite`, `a2enmod` et `apache2ctl`.
> 3. Toujours lancer **`apache2ctl configtest`** avant un reload.
> 4. `apache2ctl -S` est l'outil principal pour comprendre la sélection des VirtualHost.
> 5. En Apache 2.4, utiliser **`Require`**, pas les anciennes directives `Order/Allow/Deny` de 2.2.
> 6. Ne jamais recopier d'anciennes suites TLS utilisant SSLv2, RC4, EXPORT ou LOW.
> 7. Pour un certificat public, automatiser avec **ACME** lorsque possible.
> 8. Pour un backend moderne, Apache fonctionne très bien comme **reverse proxy** ou frontal **PHP-FPM**.
> 9. `ProxyRequests On` n'est pas nécessaire au reverse proxy et peut créer un open proxy dangereux.
> 10. La bonne démarche d'exploitation est : **mesurer → modifier → valider → reload gracieux → tester → observer**.
