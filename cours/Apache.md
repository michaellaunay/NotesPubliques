# Présentation

Apache est un serveur web pouvant être utilisé comme proxy, cache, etc.

Il supporte le protocole https et est donc utilisé pour servir les
applications web à sécuriser.

# Installation

La commande **apt install apache2** permet d'utiliser une version
récente d'Apache.

# Configuration

Nous allons créer un petit site *www.monsite.com* et nous allons voir
comment le sécuriser.

Dans un premier temps nous allons ajouter sur le poste client les entrées www.monsite.com pour réaliser des tests sans passer par le DNS.

Ajout de ssl.monsite.com /etc/hosts :

    192.168.0.7 www.monsite.com

Puis sur le serveur nous allons activer les modules utilisés pour la sécurisation :

    root@monserveur:~# a2enmod ssl

Cette commande créer 2 liens dans /etc/apache2/mods-enabled pointant vers ../mods-available/ssl.conf et ../mods-available/ssl.load.

Pour ajouter un site, il suffit de créer un fichier de configuration dans */etc/apache2/sites-available* puis de l'activer :

    root@monserveur:~# vim /etc/apache2/sites-available/www.monsite.com

      <VirtualHost *:443>
        ServerAdmin michaellaunay@ecreall.com
        ServerName www.monsite.com
        SSLEngine on
        SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP
        SSLCertificateFile /etc/apache2/ssl/server.crt
        SSLCertificateKeyFile /etc/apache2/ssl/server.key
        <Directory /var/www/>
                      Options Indexes FollowSymLinks MultiViews
                      AllowOverride None
                      Order allow,deny
                      allow from all
        </Directory>
        DocumentRoot /var/www
      </VirtualHost>

      <VirtualHost *:80>
        ServerAdmin michaellaunay@ecreall.com
        ServerName www.monsite.com
        <Directory /var/www/>
                      Options Indexes FollowSymLinks MultiViews
                      AllowOverride None
                      Order allow,deny
                      allow from all
        </Directory>
        DocumentRoot /var/www
      </VirtualHost>

Puis d'activer le site :

    root@monserveur:~# a2ensite www.monsite.com
    root@monserveur:~# /etc/init.d/apache2 restart

# Sécurisation

La sécurisation se fait en ajoutant un certificat X 509 sous forme d'une clé privée et d'une clé privée.

Nous verrons au chapitre X 509 l'usage d'une clé autosignée.

# Traçage

La configuration des logs permet de surveiller les accès aux sites.

On constatera que pour un site mis en ligne sur internet les tentatives d'intrusions sont importantes.