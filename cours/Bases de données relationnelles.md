Objectif: Comprendre les bases de données relationnelles

Plan :

1.  Introduction aux bases de données relationnelles
    
    -   Définition et concept de bases de données relationnelles
    -   Avantages et inconvénients des bases de données relationnelles
    -   Historique et évolution des bases de données relationnelles
2.  Modèle de données relationnelles
    
    -   Structure de données : tables, colonnes, lignes
    -   Clés primaires et clés étrangères
    -   Types de données et contraintes de colonne
    -   Normalisation et dénormalisation des données
3.  Langage de manipulation de données (DML)
    
    -   Structures de contrôle de flux : SELECT, INSERT, UPDATE, DELETE
    -   Jointures et sous-requêtes
    -   Fonctions et agrégats
4.  Langage de définition de données (DDL)
    
    -   Création et modification de tables et de colonnes
    -   Index et vues
5.  Gestion de transactions et de verrous
    
    -   Gestion des transactions avec COMMIT et ROLLBACK
    -   Verrous de tables et de lignes
6.  Sécurité des bases de données
    
    -   Utilisateurs et droits d'accès
    -   Chiffrement des données
7.  Optimisation des requêtes
    
    -   Index et optimisation des requêtes
    -   Analyse des plans d'exécution
8.  Outils de gestion de bases de données
    
    -   SGBD populaires (MySQL, PostgreSQL, Oracle, etc.)
    -   Outils de gestion de bases de données (phpMyAdmin, PgAdmin, etc.)

Pour les 16 heures de travaux pratiques, nous feront des exercices pratiques et des projets qui mettent en pratique les concepts enseignés pendant les heures théoriques. 

## Introduction aux bases de données relationnelles

Un système de gestion de base de données (SGBD) est un logiciel conçu pour stocker, organiser et accéder de manière efficace aux données d'une base de données. Un SGBD est utilisé pour gérer les données d'une organisation et les rendre accessibles à différentes applications et utilisateurs.

L'histoire de la gestion de bases de données remonte à plusieurs décennies, avec l'apparition de premiers systèmes de gestion de bases de données dans les années 1960. A cette époque, les bases de données étaient principalement utilisées pour le stockage de données structurées, telles que les informations sur les employés ou les produits d'une entreprise.

Avec l'explosion de la quantité de données numériques au cours des dernières décennies, la gestion de bases de données est devenue de plus en plus complexe et importante. Les SGBD ont évolué pour devenir des outils de gestion de données de grande envergure, capable de gérer des bases de données de toutes tailles et de tous types.

Il existe deux grands types de SGBD : les SGBD relationnels et les SGBD NoSQL.

Les SGBD relationnels, tels que MySQL, Oracle et Microsoft SQL Server, utilisent le modèle de données relationnel pour stocker et accéder aux données. Dans ce modèle, les données sont stockées dans des tables et les relations entre les données sont définies par des clés étrangères. Les SGBD relationnels sont particulièrement adaptés pour les bases de données de grande envergure et offrent une structure de données rigoureuse et normalisée.

Les SGBD NoSQL, comme MongoDB, Cassandra et Redis, utilisent des modèles de données non relationnels pour stocker et accéder aux données. Ces modèles de données sont plus flexibles et adaptables que le modèle relationnel, ce qui les rend particulièrement adaptés pour les bases de données de grande envergure et pour les données de type non structurées, telles que les données de log ou de médias sociaux. Cependant, ils sont moins adaptés pour les bases de données nécessitant une structure de données rigoureuse et normalisée.

-   Définition et concept de bases de données relationnelles
    
    -   Une base de données relationnelle est un système de gestion de base de données (SGBD) qui utilise le modèle de données relationnel pour stocker et accéder aux données. Le modèle de données relationnel consiste en un ensemble de tables de données organisées de manière à refléter les relations entre les différentes données.
-   Avantages et inconvénients des bases de données relationnelles
    
    -   Avantages :
        -   Structure de données rigoureuse et normalisée
        -   Facilité de mise à jour et d'interrogation des données
        -   Sécurité des données grâce à des contrôles de vérification et de gestion des transactions
        -   Possibilité de bases ACID [wikipedia](https://fr.wikipedia.org/wiki/Propri%C3%A9t%C3%A9s_ACID)
    -   Inconvénients :
        -   Coût de mise en place et de maintenance relativement élevé
        -   Temps d'exécution des requêtes potentiellement long en cas de bases de données de grande taille
-   Historique et évolution des bases de données relationnelles
    
    -   Le modèle de données relationnel a été développé par Edgar F. Codd dans les années 1970. Il a été largement adopté dans les années 1980 grâce à l'apparition de SGBD relationnels tels que Oracle et SQL Server. Depuis, il est devenu l'un des modèles de données les plus populaires et est largement utilisé dans de nombreux domaines, notamment la gestion de bases de données de grande envergure pour les entreprises et les organisations.

### Qu'est le modèle de données relationnel ?

Le modèle de données relationnel est un modèle de stockage de données utilisé par les systèmes de gestion de base de données relationnels (SGBDR) pour représenter les données sous forme de tables de données organisées de manière à refléter les relations entre elles.

Dans le modèle de données relationnel, les données sont organisées dans des tables, chaque table étant composée de colonnes et de lignes. Chaque colonne représente un champ de données (par exemple, "nom" ou "adresse") et chaque ligne représente une entrée de données (par exemple, un employé ou un produit). Les tables sont liées entre elles par des clés étrangères, qui sont des champs utilisés pour établir une relation entre les données de différentes tables.

Le modèle de données relationnel est basé sur les mathématiques de l'algèbre relationnelle, qui a été développée par Edgar F. Codd dans les années 1970. Il a été largement adopté dans les années 1980 grâce à l'apparition de SGBDR tels que Oracle et SQL Server, et est aujourd'hui l'un des modèles de données les plus populaires et largement utilisés dans de nombreux domaines.

Le modèle de données relationnel présente plusieurs avantages, notamment une structure de données rigoureuse et normalisée, une facilité de mise à jour et d'interrogation des données et une sécurité des données grâce à des contrôles de vérification et de gestion des transactions. Cependant, il a également quelques inconvénients, tels que des coûts de mise en place et de maintenance relativement élevés et des temps d'exécution des requêtes potentiellement longs en cas de bases de données de grande taille.

## Modèle de données relationnelles

-   Structure de données : tables, colonnes, lignes
    
    -   Dans le modèle de données relationnel, les données sont organisées dans des tables. Une table est composée de colonnes et de lignes. Chaque colonne représente un champ de données (par exemple, "nom" ou "adresse") et chaque ligne représente une entrée de données (par exemple, un employé ou un produit).
-   Clés primaires et clés étrangères
    
    -   Une clé primaire est un champ ou un ensemble de champs unique dans une table qui est utilisé pour identifier de manière unique chaque ligne de la table.
    -   Une clé étrangère est un champ dans une table qui est utilisé pour établir une relation avec une autre table. La clé étrangère est liée à la clé primaire de l'autre table, permettant ainsi de relier les données de deux tables entre elles.
-   Types de données et contraintes de colonne
    
    -   Chaque colonne d'une table peut avoir un type de données spécifique (par exemple, nombre entier, chaîne de caractères, date, etc.). Il est important de définir le type de données approprié pour chaque colonne afin de s'assurer que les données stockées dans la colonne sont valides et conformes aux attentes.
    -   Les contraintes de colonne sont des règles qui s'appliquent aux données d'une colonne dans une table. Elles peuvent être utilisées pour s'assurer que les données sont valides et cohérentes (par exemple, une colonne "âge" ne peut contenir que des valeurs entières positives).
-   Normalisation et dénormalisation des données
    
    -   La normalisation des données consiste à structurer les données de manière à minimiser les redondances et les dépendances inutiles entre les différentes tables de la base de données. La normalisation des données permet d'améliorer la qualité des données et de rendre la base de données plus facile à maintenir et à mettre à jour.
    -   La dénormalisation des données consiste à ajouter de la redondance aux données dans le but de performance. La dénormalisation des données peut être utilisée pour améliorer les performances des requêtes sur les données, mais peut entraîner une augmentation de la taille de la base de données et une diminution de la qualité des données.

## Langage de manipulation de données (DML)

-   Structures de contrôle de flux : SELECT, INSERT, UPDATE, DELETE
    
    -   SELECT : La clause SELECT permet de sélectionner les données à partir d'une ou plusieurs tables de la base de données. Elle peut être utilisée avec différentes clauses pour filtrer et trier les données sélectionnées.
    -   INSERT : La clause INSERT permet d'ajouter de nouvelles lignes dans une table de la base de données.
    -   UPDATE : La clause UPDATE permet de mettre à jour les données d'une table de la base de données.
    -   DELETE : La clause DELETE permet de supprimer des lignes d'une table de la base de données.
-   Jointures et sous-requêtes
    
    -   Les jointures permettent de combiner les données de plusieurs tables de la base de données en une seule requête. Il existe plusieurs types de jointures (inner join, outer join, etc.), chacun ayant ses propres règles de combinaison des données.
    -   Les sous-requêtes sont des requêtes imbriquées dans une autre requête. Elles peuvent être utilisées pour filtrer les données ou pour obtenir des données à utiliser dans la requête principale.
-   Fonctions et agrégats
    
    -   Les fonctions permettent d'appliquer une transformation aux données sélectionnées (par exemple, calculer la moyenne d'un champ de données).
    -   Les agrégats permettent de calculer des valeurs sur l'ensemble des données sélectionnées (par exemple, calculer la somme de tous les enregistrements d'un champ de données).
### Détails et exemples
    *SELECT* : La clause SELECT permet de sélectionner les données à partir d'une ou plusieurs tables de la base de données. Elle peut être utilisée avec différentes clauses pour filtrer et trier les données sélectionnées.

Exemple de requête SELECT simple qui sélectionne tous les enregistrements de la table "employees" :
```sql
SELECT * FROM employees;
```

Exemple de requête SELECT avec la clause WHERE qui sélectionne uniquement les enregistrements de la table "employees" dont le champ "department" est égal à "sales" :
```sql
SELECT * FROM employees WHERE department = 'sales';
```

Exemple de requête SELECT avec la clause ORDER BY qui sélectionne tous les enregistrements de la table "employees" et les trie par ordre alphabétique croissant sur le champ "last_name" :
```sql
SELECT * FROM employees ORDER BY last_name ASC;
```

    *INSERT* : La clause INSERT permet d'ajouter de nouvelles lignes dans une table de la base de données.

Exemple de requête INSERT qui ajoute un nouvel enregistrement dans la table "employees" :
```sql
INSERT INTO employees (first_name, last_name, department, salary)
VALUES ('John', 'Doe', 'marketing', 50000);
```


*UPDATE* : La clause UPDATE permet de mettre à jour les données d'une table de la base de données.


Exemple de requête UPDATE qui modifie le salaire de l'employé ayant l'ID 1 dans la table "employees" :
```sql
UPDATE employees
SET salary = 55000
WHERE id = 1;
```
*DELETE* : La clause DELETE permet de supprimer des lignes d'une table de la base de données.

Exemple de requête DELETE qui supprime l'enregistrement de l'employé ayant l'ID 1 dans la table "employees" :
```sql
DELETE FROM employees WHERE id = 1;
```

Voici maintenant des exemples de jointures et de sous-requêtes :

    *Jointures* : Les jointures permettent de combiner les données de plusieurs tables de la base de données en une seule requête. Il existe plusieurs types de jointures (inner join, outer join, etc.), chacun ayant ses propres règles de combinaison des données.

Exemple de requête utilisant une jointure INNER JOIN qui combine les données de la table "employees" et de la table "departments" :
```sql
SELECT employees.*, departments.name as department_name FROM employees
INNER JOIN departments ON employees.department_id = departments.id;
```

    *Sous-requêtes* : Les sous-requêtes sont des requêtes imbriquées dans une autre requête. Elles peuvent être utilisées pour filtrer les données ou pour obtenir des données à utiliser dans la requête principale.

Exemple de requête utilisant une sous-requête qui sélectionne les enregistrements de la table "employees" dont le salaire est supérieur à la moyenne des salaires de la table "employees" :
```sql
SELECT * FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);
```

Exemples de fonctions et d'agrégats :

    *Fonctions* : Les fonctions permettent d'appliquer une transformation aux données sélectionnées (par exemple, calculer la moyenne d'un champ de données).

Exemple de requête utilisant la fonction UPPER qui met en majuscules le champ "last_name" de tous les enregistrements de la table "employees" :
```sql
SELECT UPPER(last_name) as last_name FROM employees;
```

    *Agrégats* : Les agrégats permettent de calculer des valeurs sur l'ensemble des données sélectionnées (par exemple, calculer la somme de tous les enregistrements d'un champ de données).

Exemple de requête utilisant l'agrégat SUM qui calcule la somme des salaires de tous les enregistrements de la table "employees" :
```sql
SELECT SUM(salary) as total_salary FROM employees;
```
Langage de définition de données (DDL)

 ###   Création et modification de tables
        La syntaxe de création de table varie selon les SGBDR, mais en général elle permet de spécifier les colonnes de la table, leur type de données et leurs contraintes.
        La syntaxe de modification de table permet de modifier les colonnes existantes ou d'ajouter de nouvelles colonnes à une table existante.

 ###   Vues
        Les vues sont des objets de base de données qui représentent une requête enregistrée. Elles permettent de simplifier la syntaxe des requêtes complexes ou fréquemment utilisées en les encapsulant dans une vue, qui peut être utilisée comme une table.

 ###   Index
        Les index sont des structures de données qui permettent d'accélérer les requêtes sur les données. Ils sont créés sur une ou plusieurs colonnes d'une table et permettent de trier rapidement les données pour faciliter la recherche.

 ###   Contraintes
        Les contraintes sont des règles qui s'appliquent aux données d'une table. Elles peuvent être utilisées pour s'assurer que les données sont valides et cohérentes (par exemple, une colonne "âge" ne peut contenir que des valeurs entières positives).

Exemple de création de table avec le SGBDR MySQL :

```SQL
CREATE TABLE employees (
  id INTEGER PRIMARY KEY AUTO_INCREMENT,
  first_name VARCHAR(255) NOT NULL,
  last_name VARCHAR(255) NOT NULL,
  department VARCHAR(255) NOT NULL,
  salary INTEGER NOT NULL,
  CHECK (salary > 0)
);
```

Exemple de création de vue avec le SGBDR MySQL :
```SQL
CREATE VIEW employee_names AS
SELECT id, first_name, last_name FROM employees;
```

Exemple de création d'index avec le SGBDR MySQL :
```SQL
CREATE INDEX last_name_index ON employees (last_name);
```

Exemple de définition de contraintes avec le SGBDR MySQL :
```SQL
ALTER TABLE employees
ADD FOREIGN KEY (department_id) REFERENCES departments(id)
ON DELETE CASCADE;
```

Cette requête ajoute une contrainte de clé étrangère à la table "employees", liée à la clé primaire de la table "departments". La clause "ON DELETE CASCADE" spécifie que si un enregistrement de la table "departments" est supprimé, tous les enregistrements de la table "employees" liés à cet enregistrement seront également supprimés.

Attention la syntaxe des différents SGBDR peut variée pour la création et la modification de tables, les vues, les index et les contraintes.

## Transactions et isolation des données

-   Transactions
    
    -   Une transaction est un ensemble de requêtes exécutées de manière atomique, c'est-à-dire que soit toutes les requêtes de la transaction sont exécutées avec succès, soit aucune n'est exécutée si une erreur se produit. Les transactions permettent de s'assurer que les données de la base de données restent cohérentes en cas d'échec d'une requête.
-   Isolation des données
    
    -   L'isolation des données permet de s'assurer que les transactions en cours ne sont pas affectées par les modifications de données effectuées par d'autres transactions. Il existe plusieurs niveaux d'isolation des données qui définissent la manière dont les transactions interagissent entre elles.

Voici un exemple de transaction avec le SGBDR MySQL :
```SQL
START TRANSACTION;  UPDATE accounts SET balance = balance - 100 WHERE id = 1; UPDATE accounts SET balance = balance + 100 WHERE id = 2;
```


Sécurité des bases de données

    Droits d'accès et privilèges
        Les droits d'accès et privilèges permettent de contrôler qui peut accéder aux données et aux objets de la base de données, ainsi que les actions que ces utilisateurs peuvent effectuer (lecture, écriture, suppression, etc.).

    Mots de passe et authentification
        Les mots de passe et l'authentification permettent de sécuriser l'accès aux données et aux objets de la base de données en exigeant que les utilisateurs fournissent des informations d'identification avant de pouvoir accéder aux données.

    Chiffrement
        Le chiffrement des données permet de protéger les données sensibles en les rendant illisibles pour toute personne ne disposant pas de la clé de déchiffrement.

Exemple de gestion des droits d'accès et privilèges avec le SGBDR MySQL :
```SQL
GRANT SELECT, INSERT, UPDATE ON employees TO 'user1';
REVOKE DELETE ON employees FROM 'user1';
```

Cet exemple donne à l'utilisateur "user1" les droits de lecture, d'insertion et de modification sur la table "employees", mais lui retire le droit de suppression.

Exemple de gestion des mots de passe et de l'authentification avec le SGBDR MySQL :
```SQL
CREATE USER 'user2'@'localhost' IDENTIFIED BY 'password';
```

Cet exemple crée un utilisateur nommé "user2" avec le mot de passe "password" et autorise l'authentification uniquement à partir de l'hôte local.

Exemple de chiffrement des données avec le SGBDR MySQL :
```SQL
UPDATE employees SET password = AES_ENCRYPT(password, 'key') WHERE id = 1;
```
Cet exemple chiffre le mot de passe de l'employé ayant l'ID 1 en utilisant l'algorithme AES et la clé de chiffrement "key".

Il est important de noter que la sécurité de votre base de données dépend également de votre environnement d'hébergement et de votre réseau. Il est recommandé de mettre en place des mesures de sécurité supplémentaires, telles que l'authentification à deux facteurs, la chiffrement des communications réseau et la mise à jour régulière des logiciels et des systèmes d'exploitation.

Il est également conseillé de sauvegarder régulièrement votre base de données pour pouvoir la restaurer en cas de perte de données ou de corruption. Il existe plusieurs façons de sauvegarder une base de données, telles que l'utilisation de scripts de sauvegarde, l'export de données vers un fichier externe ou la réplication de la base de données sur un serveur de sauvegarde.

Enfin, il est important de respecter les réglementations et les lois en vigueur concernant la protection des données et la confidentialité. Cela peut inclure la mise en place de politiques de sécurité et de gestion des accès, la déclaration de fuites de données potentielles et la mise en place de mesures de protection pour les données sensibles.

Optimisation des requêtes et des bases de données

    Optimisation des requêtes
        L'optimisation des requêtes permet d'améliorer les performances des requêtes en utilisant les index et en sélectionnant judicieusement les données à récupérer. Il est important de prendre le temps d'analyser les requêtes et de les tester pour s'assurer qu'elles sont aussi efficaces que possible.

    Optimisation de la base de données
        L'optimisation de la base de données permet d'améliorer les performances globales de la base de données en utilisant les index de manière efficace, en organisant les données de manière à minimiser les accès disques et en utilisant les paramètres de configuration optimaux.

Voici un exemple d'optimisation de requête avec le SGBDR MySQL :
```SQL
SELECT * FROM employees WHERE salary > 50000 AND department = 'marketing';
```

Pour optimiser cette requête, nous pouvons créer un index sur les colonnes "salary" et "department" de la table "employees" :
```SQL
CREATE INDEX salary_department_index ON employees (salary, department);
```

Cet index permettra au SGBDR de trier rapidement les enregistrements en fonction de leur salaire et de leur département, ce qui améliorera les performances de la requête.

Voici un exemple d'optimisation de base de données avec le SGBDR MySQL :
```SQL
ALTER TABLE employees ENGINE = INNODB;
```
Cette commande modifie le moteur de stockage de la table "employees" en utilisant le moteur InnoDB, qui est conçu pour offrir de bonnes performances et une gestion efficace des transactions.

Il existe de nombreuses autres techniques d'optimisation de requêtes et de bases de données, en fonction du SGBDR utilisé et de la structure des données. Par exemple, il peut être utile de regrouper les données fréquemment utilisées dans la même table ou de dénormaliser les données pour réduire le nombre de jointures nécessaires dans les requêtes.

Il est également recommandé de surveiller les performances de votre base de données en utilisant des outils de monitoring et en analysant les journaux d'erreurs et de performances. Cela vous permettra de détecter les problèmes éventuels et de les résoudre avant qu'ils n'affectent significativement les performances de votre base de données.

Enfin, il est important de continuer à apprendre et à se tenir au courant des nouvelles technologies et des meilleures pratiques en matière d'optimisation de bases de données.

## Outils de gestion de bases de données

Il existe de nombreux outils de gestion de bases de données disponibles pour vous aider à gérer et à interagir avec vos bases de données. Voici quelques exemples d'outils populaires :

### Interfaces de ligne de commande (CLI)
Les interfaces de ligne de commande, comme mysql ou psql, permettent d'exécuter des requêtes SQL et de gérer les bases de données depuis la ligne de commande.

### Interfaces graphiques (GUI)
Les interfaces graphiques, comme MySQL Workbench ou pgAdmin, permettent d'interagir avec les bases de données de manière visuelle et de visualiser les données et les structures de données de manière intuitive.

### Logiciels de développement intégré (IDE)
Les logiciels de développement intégré, comme PyCharm ou Visual Studio, permettent de créer et de déboguer des applications utilisant des bases de données, ainsi que de gérer les bases de données elles-mêmes.

### Remarques
Il est recommandé de choisir l'outil de gestion de bases de données qui convient le mieux à vos besoins et à votre environnement de travail. Si vous êtes un développeur, par exemple, un IDE pourrait être l'outil le plus adapté, alors qu'une interface graphique pourrait être plus adaptée à un utilisateur non technique.

Il est également important de se familiariser avec les différentes fonctionnalités et outils proposés par chaque outil de gestion de bases de données, afin de pouvoir en tirer le meilleur parti. Cela peut inclure la création et la modification de tables, la gestion des utilisateurs et des privilèges, la sauvegarde et la restauration de bases de données, etc.

# Fonctionnement sqllite en python

SQLite est une bibliothèque qui implémente un système de gestion de bases de données relationnelles léger et autonome. Il peut être utilisé avec Python grâce à l'utilisation d'un module appelé sqlite3. Ce module fournit une interface pour se connecter à une base de données SQLite, exécuter des commandes SQL et manipuler les données stockées dans cette base de données.

Pour utiliser SQLite avec Python, vous devez d'abord importer le module sqlite3. Ensuite, vous pouvez utiliser la fonction connect() pour créer une connexion à une base de données existante ou en créer une nouvelle. Vous pouvez ensuite utiliser la méthode cursor() de la connexion pour créer un curseur pour exécuter des commandes SQL.

Une fois que vous avez un curseur, vous pouvez utiliser les méthodes comme execute() pour exécuter des commandes SQL, fetchone() pour récupérer une ligne de résultat à la fois et fetchall() pour récupérer toutes les lignes de résultat. Enfin, vous devez vous assurer de fermer la connexion à la base de données en utilisant close() pour éviter les fuites de mémoire et des erreurs.

Il est important de noter que SQLite ne supporte pas les utilisateurs multiples en même temps, c'est pourquoi il est utilisé principalement pour les petites application ou des projets de développement local

Donne le code pour créer une table utilisateurs(nom, prénom, age, lieu de naissance) qui possède une clé étrangère vers une table fiches de paies?

Voici un exemple de code pour créer une table "utilisateurs" avec une colonne "id_fiche_paie" qui est une clé étrangère vers une table "fiches_paie" :

```python
import sqlite3
conn = sqlite3.connect("ma_base_de_donnees.db")
cursor = conn.cursor()
# Création de la table utilisateurs avec une clé étrangère id_fiche_paie 
cursor.execute("""CREATE TABLE utilisateurs (
    id INTEGER PRIMARY KEY,
    nom TEXT,
    prenom TEXT,
    age INTEGER,  
    lieu_naissance TEXT,
    id_fiche_paie INTEGER,
    FOREIGN KEY (id_fiche_paie) REFERENCES fiches_paie(id)
)""")
# Création de la table fiches_paie 
cursor.execute("""CREATE TABLE fiches_paie (
    id INTEGER PRIMARY KEY,
    salaire REAL
)""")
conn.commit()

conn.close()
```

Notez que dans cet exemple, j'ai créé une table "fiches_paie" qui possède une clé primaire "id" qui est utilisée comme clé étrangère dans la table "utilisateurs" dans la colonne "id_fiche_paie" Il est important de noter que c'est votre responsabilité de gérer les données pour respecter les contraintes référentielles.

## Exemple
Voici un exemple de code pour insérer des données dans les tables "utilisateurs" et "fiches_paie" que nous avons créées précédemment:

```python
import sqlite3

conn = sqlite3.connect("ma_base_de_donnees.db")
cursor = conn.cursor()

# Insertion d'une fiche de paie
cursor.execute("INSERT INTO fiches_paie (id, salaire) VALUES (1, 6000)")

# Insertion d'un utilisateur lié à la fiche de paie 1
cursor.execute("INSERT INTO utilisateurs (id, nom, prenom, age, lieu_naissance, id_fiche_paie) VALUES (1, 'John', 'Doe', 30, 'Paris', 1)")

# Insertion d'une autre fiche de paie
cursor.execute("INSERT INTO fiches_paie (id, salaire) VALUES (2, 7000)")

# Insertion d'un autre utilisateur lié à la fiche de paie 2
cursor.execute("INSERT INTO utilisateurs (id, nom, prenom, age, lieu_naissance, id_fiche_paie) VALUES (2, 'Jane', 'Doe', 25, 'Lyon', 2)")

conn.commit()
conn.close()

```

Dans cet exemple, nous insérons d'abord une fiche de paie avec un id de 1 et un salaire de 6000, puis nous insérons un utilisateur avec un id de 1, un nom de "John", un prénom de "Doe", un age de 30, un lieu de naissance de "Paris" et une clé étrangère id_fiche_paie qui est liée à la fiche de paie 1. Ensuite nous insérons une autre fiche de paie avec un id de 2 et un salaire de 7000, puis nous insérons un autre utilisateur avec un id de 2, un nom de "Jane", un prénom de "Doe", un age de 25, un lieu de naissance de "Lyon" et une clé étrangère id_fiche_paie qui est liée à la fiche de paie 2. Il est important de noter que dans cet exemple, j'ai utilisé des valeurs d'ID fixes, mais dans un vrai scénario vous utiliserez une séquence d'auto-incrément ou des générateurs de clé pour générer automatiquement les valeurs d'ID.