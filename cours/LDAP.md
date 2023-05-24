# Présentation
LDAP est l'acronyme de Lightweight Directory Access Protocol. Il s'agit d'un protocole standard de l'industrie pour accéder et maintenir des services d'annuaire distribués sur Internet.
Un annuaire, est une sorte de base de données optimisée pour la lecture, la recherche et la navigation, plutôt que pour les opérations d'écriture et de modification.

# OpenLDAP
OpenLDAP est une implémentation libre et ouverte du protocole LDAP. Il est utilisé pour développer des applications de gestion d'annuaires. OpenLDAP permet de centraliser les informations des utilisateurs, telles que les identifiants, les coordonnées et les autorisations d'accès. Cela facilite grandement l'administration des systèmes et des applications.

# Structure de l'annuaire LDAP
Un annuaire LDAP est structuré comme un arbre, avec un nœud racine appelé Base Distinguished Name (ou Base DN). Sous cette racine, nous pouvons avoir plusieurs branches représentant des organisations, des unités organisationnelles, des individus, etc.

# Opérations LDAP
Les opérations les plus courantes en LDAP sont : 
1. **Recherche (Search)** : Chercher et retrouver des entrées dans l'annuaire.
2. **Ajout (Add)** : Ajouter des entrées à l'annuaire.
3. **Suppression (Delete)** : Supprimer des entrées de l'annuaire.
4. **Modification (Modify)** : Modifier des entrées existantes dans l'annuaire.

Ces opérations sont effectuées à l'aide des outils et commandes fournis par OpenLDAP, tels que `ldapsearch`, `ldapadd`, `ldapdelete`, et `ldapmodify`.

# Le format LDIF
LDIF signifie "LDAP Data Interchange Format". Il s'agit d'un format standard utilisé pour représenter les données LDAP sous forme de texte ASCII. Les fichiers LDIF sont utilisés pour l'échange de données entre différents serveurs LDAP, pour l'importation et l'exportation de données, et pour la gestion des modifications des données.

Un fichier LDIF est une collection d'enregistrements séparés par des lignes vides. Chaque enregistrement représente soit une entrée LDAP, soit une modification d'une entrée existante. 

Une entrée typique pourrait ressembler à ceci :

```ldif
dn: cn=John Doe,dc=example,dc=com
objectClass: inetOrgPerson
cn: John Doe
sn: Doe
mail: john.doe@example.com
```

## Les mots clés du format LDIF

- `dn`: "Distinguished Name". C'est l'identifiant unique de l'entrée dans l'annuaire. Il est généralement composé de plusieurs parties, telles que `cn` (Common Name), `dc` (Domain Component), `ou` (Organizational Unit), etc.
  
- `objectClass`: C'est le type d'objet de l'entrée. Il définit les attributs que l'entrée peut ou doit avoir. Les classes d'objet courantes comprennent `top`, `person`, `inetOrgPerson`, `organizationalPerson`, etc.

- `cn`: "Common Name". C'est souvent le nom de l'utilisateur ou de l'objet.
  
- `sn`: "Surname". Il représente généralement le nom de famille d'une personne.
  
- `mail`: L'adresse e-mail de l'utilisateur.

Dans le cadre de modifications, les fichiers LDIF peuvent également inclure les instructions suivantes :

- `add`: Ajoute un nouvel attribut à une entrée existante.
- `delete`: Supprime un attribut d'une entrée existante.
- `replace`: Remplace la valeur d'un attribut existant.
- `changetype`: Spécifie le type de modification à effectuer.

Un exemple de modification pourrait être :

```ldif
dn: cn=John Doe,dc=example,dc=com
changetype: modify
replace: mail
mail: john.newmail@example.com
```
Ceci remplacera l'adresse e-mail de l'utilisateur "John Doe". 

# Les classes d'objets
Les classes d'objets en LDAP sont définies dans les schémas LDAP. Un schéma LDAP est une collection de définitions et de règles concernant les types d'informations qui peuvent être stockées dans l'annuaire.

Dans le contexte de LDAP, un schéma définit plusieurs types d'informations :

- Les **classes d'objets**, qui déterminent les types d'objets que nous pouvons créer dans l'annuaire.
- Les **attributs** disponibles pour chaque classe d'objet.
- Les règles concernant les attributs requis et optionnels pour chaque classe d'objet.

Les classes d'objets comme `top`, `person`, `inetOrgPerson`, `organizationalPerson`, etc., sont définies dans des fichiers de schéma standard inclus avec la plupart des distributions de serveurs LDAP, y compris OpenLDAP.

Par exemple, dans le cas d'OpenLDAP, ces définitions de classes d'objets se trouvent généralement dans des fichiers de schéma dans le répertoire `/etc/ldap/schema/` ou `/etc/openldap/schema/` selon notre installation.

Chaque fichier de schéma contient des définitions pour une ou plusieurs classes d'objets et attributs. Ces définitions incluent le nom de la classe d'objet, la description, l'OID (Object Identifier), la liste des attributs obligatoires (MUST) et la liste des attributs optionnels (MAY).

Par exemple, la définition de la classe d'objet `person` peut ressembler à ceci :
```schema
objectclass ( 2.5.6.6 NAME 'person'
    DESC 'RFC2256: a person'
    SUP top STRUCTURAL
    MUST ( sn $ cn )
    MAY ( userPassword $ telephoneNumber $ seeAlso $ description ) )
```

Dans cet exemple, `sn` et `cn` sont des attributs obligatoires, tandis que `userPassword`, `telephoneNumber`, `seeAlso`, et `description` sont des attributs optionnels pour la classe d'objet `person`.

# Les RFC de LDAP et LDIF
LDAP et LDIF sont définis par plusieurs documents de la série RFC (Request for Comments), qui est une série de notes techniques et d'organisations qui décrit les différents aspects des technologies Internet.
Ces documents sont le cœur des standards pour LDAP et LDIF, mais il y a beaucoup d'autres documents RFC qui décrivent des extensions et des améliorations à ces protocoles.
Voici quelques-uns des plus importants pour LDAP et LDIF :

## RFC pour LDAP (Lightweight Directory Access Protocol)

1. RFC 4510 : "Lightweight Directory Access Protocol (LDAP): Technical Specification Road Map". C'est le document qui résume les différents documents techniques qui définissent LDAP. Lien : [https://tools.ietf.org/html/rfc4510](https://tools.ietf.org/html/rfc4510)

2. RFC 4511 : "Lightweight Directory Access Protocol (LDAP): The Protocol". Ce document définit le protocole LDAP lui-même. Lien : [https://tools.ietf.org/html/rfc4511](https://tools.ietf.org/html/rfc4511)

3. RFC 4512 : "Lightweight Directory Access Protocol (LDAP): Directory Information Models". Ce document décrit les modèles d'information de l'annuaire utilisés par LDAP. Lien : [https://tools.ietf.org/html/rfc4512](https://tools.ietf.org/html/rfc4512)

4. RFC 4513 : "Lightweight Directory Access Protocol (LDAP): Authentication Methods and Security Mechanisms". Ce document définit les méthodes d'authentification et les mécanismes de sécurité utilisés par LDAP. Lien : [https://tools.ietf.org/html/rfc4513](https://tools.ietf.org/html/rfc4513)

Il existe également d'autres RFCs qui décrivent d'autres aspects de LDAP, comme la recherche de chaînes, la syntaxe des filtres de recherche, etc.

## RFC pour LDIF (LDAP Data Interchange Format)
RFC 2849 : "The LDAP Data Interchange Format (LDIF) - Technical Specification". Ce document définit le format LDIF utilisé pour représenter les données LDAP en tant que texte ASCII. Lien : [https://tools.ietf.org/html/rfc2849](https://tools.ietf.org/html/rfc2849)

# Installation openldap
Ldap est un annuaire qui permet de gérer l'utilisateur d'un service
sans créer un compte unix.

Installation du serveur ldap :
```bash
apt install slapd ldap-utils
```

Modification de la configuration :
```bash
dpkg-reconfigure slapd
# saisie de "ecreall.com" comme domaine
# saisie de "people" comme organization
# saisie du mot de passe (comme celui de l'utilisateur michaellaunay, mais pour LDAP)
```

Attention ! Configurer LTS pour chiffrer les connexions si elles sont extérieures à la machine, car les mots de passe circulent en clair (voir <https://wiki.debian.org/LDAP/OpenLDAPSetup#Enable_TLS.2FSSL>)!

Activer le service au démarrage :

```bash
systemctl enable slapd
```

Rendre \"ldap\" accessible en éditant \"/etc/ldap/ldap.conf\" en ajoutant :
```
BASE    dc=ecreall,dc=com
URI     ldap://127.0.0.1
```

# Ajout d'une Organisation à LDAP
Prenons l'exemple d'Ecréall
Créer un fichier ecreall.ldif contenant :
```
dn: ou=People,dc=ecreall,dc=com
ou: People
objectClass: top
objectClass: organizationalUnit

dn: ou=Group,dc=ecreall,dc=com
ou: Group
objectClass: top
objectClass: organizationalUnit

dn: cn=Gérant,dc=ecreall,dc=com
objectclass: organizationalRole
cn: Gérant
```

```bash
ldapadd -x -D "cn=admin,dc=ecreall,dc=com" -W -f ecreall.ldif
```

Mettre à jour l'index (cache) :
```bash
systemctl stop slapd
slapindex
chown -R openldap:openldap /var/lib/ldap
chown -R openldap:openldap /var/lib/ldap
chmod -R 700 /var/lib/ldap
systemctl start slapd
```

Vérification :
```bash
ldapsearch -x -b 'dc=ecreall,dc=com' '(objectclass=*)'
```

# Ajouter une OrganizationUnit
Créer un fichier \"e-services.ldif\" et y mettre :

```ldif
dn: ou=Etudes,dc=ecreall,dc=com
objectClass: organizationalUnit
ou: Etudes
```
>
```bash
ldapadd -x -D "cn=admin,dc=ecreall,dc=com" -W -f e-services.ldif
```

# Ajouter une personne
## Version non optimisée
Exemple "non optimisé" pour ajouter Michaël Launay, créer un fichier
> \"ldif\_files/michaellaunay.ldif\" :
```ldif
dn: uid=michaellaunay,ou=Études,dc=ecreall,dc=com
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetorgperson
cn: Michael Launay
sn: Launay
gn: Michael
uid: michaellaunay
title: Gerant
mail: michaellaunay@ecreall.com
telephoneNumber: 0320793290
postalAddress: 11 A Avenue de l'Harmonie
postalCode: 59650
l: Villeneuve d ASCQ
```
Il y a une certaine redondance dans ce fichier LDIF, en particulier dans la déclaration des classes d'objet (objectClass).
Voici pourquoi :
1. `inetOrgPerson` est une classe d'objet qui hérite des classes d'objet `organizationalPerson` et `person`. Donc, si nous déclarons `inetOrgPerson`, nous n'avons pas besoin de déclarer également `person` et `organizationalPerson`.

2. De même, `organizationalPerson` hérite de la classe `person`, donc si nous déclarons `organizationalPerson`, nous n'avons pas besoin de déclarer `person`.

3. Enfin, toutes les entrées LDAP doivent inclure la classe `top`, mais celle-ci est généralement incluse automatiquement lorsque nous déclarons une autre classe d'objet. Donc, en pratique, nous n'avons pas besoin de déclarer explicitement `top`.

## Version otpimisée du fichier LDIF
Nous pouvons simplifier le fichier LDIF comme suit :
```ldif
dn: uid=michaellaunay,ou=Études,dc=ecreall,dc=com
objectClass: inetOrgPerson
cn: Michael Launay
sn: Launay
gn: Michael
uid: michaellaunay
title: Gerant
mail: michaellaunay@ecreall.com
telephoneNumber: 0320793290
postalAddress: 11 A Avenue de l'Harmonie
postalCode: 59650
l: Villeneuve d ASCQ
```

De cette façon, l'entrée conserve les mêmes propriétés, mais sans redondance dans le fichier LDIF.

## Ajouter l'entrée à l'annuaire
```bash
ldapadd -x -D "cn=admin,dc=ecreall,dc=com" -W -f ldif_files/michaellaunay.ldif
```

## Enregistrer le mot de passe du nouvel utilisateur :
```bash
ldappasswd -D "cn=admin,dc=ecreall,dc=com" -W "cn=Prenom Nom,ou=People,dc=ecreall,dc=com" -S
```

## Modifier le mot de passe d'un utilisateur existant
Pour modifier le mot de passe d'un utilisateur existant dans LDAP, nous pouvons utiliser la commande `ldappasswd`. Voici la syntaxe générale de cette commande :

```bash
ldappasswd -H ldap://localhost -x -D "cn=admin,dc=ecreall,dc=com" -W -S "uid=utilisateur,ou=People,dc=ecreall,dc=com"
```

Décortiquons cette commande :

- `-H ldap://localhost` spécifie l'URI du serveur LDAP.
- `-x` utilise une authentification simple au lieu de SASL.
- `-D "cn=admin,dc=ecreall,dc=com"` spécifie l'identifiant de l'utilisateur qui a les droits d'administrateur pour effectuer les modifications.
- `-W` demande le mot de passe de l'utilisateur spécifié après `-D`.
- `-S "uid=utilisateur,ou=People,dc=ecreall,dc=com"` spécifie l'utilisateur dont nous voulons changer le mot de passe. Vous devrez remplacer "utilisateur" par le nom d'utilisateur réel.

Après avoir exécuté cette commande, elle vous demandera d'entrer le nouveau mot de passe deux fois pour confirmation. Si tout se passe bien, le mot de passe de l'utilisateur sera changé.

# Stockage des mots de passe dans OpenLDAP
OpenLDAP stocke les mots de passe en utilisant des hachages plutôt qu'en clair, conformément aux exigences de sécurité et de confidentialité, y compris celles du Règlement Général sur la Protection des Données (RGPD).

Lorsqu'un mot de passe est fourni à OpenLDAP (par exemple, lors de la création ou de la modification d'un compte utilisateur), le serveur LDAP ne stocke pas le mot de passe lui-même. Au lieu de cela, il utilise une fonction de hachage pour transformer le mot de passe en une "empreinte" de hachage, qui est ce qui est réellement stocké. Lorsque l'utilisateur se connecte, le mot de passe qu'il fournit est à nouveau haché et comparé à l'empreinte de hachage stockée.

Ce processus de hachage est unidirectionnel, ce qui signifie qu'il n'est pas possible de retrouver le mot de passe original à partir de l'empreinte de hachage. Ainsi, même si quelqu'un parvenait à accéder aux données stockées par le serveur LDAP, il ne pourrait pas retrouver les mots de passe des utilisateurs.

OpenLDAP supporte plusieurs schémas de hachage, y compris SHA-2, SHA-1, et MD5. Il est recommandé d'utiliser le schéma de hachage le plus fort possible pour améliorer la sécurité. À partir de la version 2.4, OpenLDAP utilise par défaut SSHA (Salted SHA), une variante de SHA qui inclut un "sel" pour rendre les attaques par force brute ou par table de hachage plus difficiles.

Le hachage des mots de passe ne dispense pas de l'importance de l'utilisation du chiffrement (par exemple, TLS) pour les connexions au serveur LDAP. Le hachage protège les mots de passe stockés, mais sans chiffrement, les mots de passe peuvent être interceptés en clair lors de leur transmission au serveur.

# Liens
> <https://wiki.debian.org/LDAP/OpenLDAPSetup>
> <https://guide.ubuntu-fr.org/server/openldap-server.html>
> <http://www-sop.inria.fr/members/Laurent.Mirtain/ldap-livre.html>