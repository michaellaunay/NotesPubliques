---
schema_version: 1
uid: 01M02EX5ATSDQA63DZAW1X8FXN
titre: Bases de données relationnelles
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - bases-de-donnees
  - sql
  - modelisation
resume: "Cours complet sur les bases de données relationnelles : modèle relationnel, SQL moderne, modélisation, normalisation, contraintes, transactions, concurrence, index, optimisation, sécurité, migrations, sauvegarde et exploitation."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-01-02
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: true
---

# Bases de données relationnelles

> [!abstract] Objectif
> Comprendre le **modèle relationnel**, savoir concevoir un schéma cohérent, écrire du SQL robuste, maîtriser les transactions et la concurrence, diagnostiquer les performances et exploiter une base de données en production.

Ce cours utilise principalement **PostgreSQL** pour les exemples avancés, car il expose très clairement les concepts relationnels et transactionnels. Les différences importantes avec **MySQL** et **SQLite** sont signalées lorsqu'elles ont un impact sur le comportement.

Au 29 août 2026, les repères utilisés dans ce cours sont :

- **SQL:2023** — série ISO/IEC 9075:2023 ;
- **PostgreSQL 18.6** comme version stable de référence ; PostgreSQL 19 est encore en bêta ;
- **MySQL 8.4 LTS** pour la branche à support long et **MySQL 9.x** pour la branche Innovation ;
- **SQLite 3.53.4** pour les exemples embarqués.

Une base relationnelle n'est pas simplement « un ensemble de tableaux ». Le modèle relationnel définit des **relations**, des **attributs**, des **domaines**, des **clés**, des **contraintes** et des opérations permettant de produire de nouvelles relations à partir d'autres relations.

---

# Sommaire

1. Principes et histoire du modèle relationnel
2. Relation, tuple, attribut, domaine et NULL
3. Clés et contraintes d'intégrité
4. Modélisation conceptuelle et logique
5. Normalisation et dépendances fonctionnelles
6. DDL : créer et faire évoluer un schéma
7. DML : INSERT, UPDATE et DELETE
8. SELECT : filtrer, trier et projeter
9. Jointures
10. Agrégats, GROUP BY et HAVING
11. Sous-requêtes et CTE
12. Fonctions fenêtre
13. Ensembles et logique SQL
14. Vues et vues matérialisées
15. Transactions et propriétés ACID
16. Niveaux d'isolation, anomalies et MVCC
17. Verrous, concurrence et deadlocks
18. Index
19. Plans d'exécution et optimisation
20. Sécurité et contrôle d'accès
21. SQL dynamique et injection SQL
22. Migrations de schéma
23. Sauvegarde, restauration et reprise après incident
24. Réplication, haute disponibilité et partitionnement
25. Observabilité et maintenance
26. PostgreSQL, MySQL et SQLite : différences utiles
27. Python et bases relationnelles
28. ORM et SQL explicite
29. Architecture applicative et bonnes pratiques
30. Travaux pratiques et projet final

---

# 1. Principes et histoire du modèle relationnel

## 1.1 Pourquoi une base de données ?

Une base de données répond à plusieurs besoins qui deviennent rapidement difficiles à gérer avec de simples fichiers :

- stocker durablement des données ;
- garantir leur cohérence ;
- effectuer des recherches efficaces ;
- permettre les accès concurrents ;
- contrôler les droits ;
- restaurer les données après une panne ;
- faire évoluer le schéma sans perdre l'historique métier.

Un **SGBD** — système de gestion de base de données — fournit les mécanismes permettant d'assurer ces propriétés.

## 1.2 Edgar F. Codd et le modèle relationnel

En 1970, Edgar F. Codd publie *A Relational Model of Data for Large Shared Data Banks*. L'idée fondamentale est de séparer :

- la **représentation logique** des données ;
- leur **organisation physique** sur disque.

L'application travaille avec des relations et des opérations logiques plutôt qu'avec les détails de stockage.

Cette séparation reste essentielle aujourd'hui : une requête SQL décrit **ce que l'on souhaite obtenir**, tandis que l'optimiseur du SGBD choisit **comment** l'obtenir.

## 1.3 SQL n'est pas exactement le modèle relationnel

Le SQL réel diffère du modèle relationnel théorique sur plusieurs points :

- les résultats SQL peuvent contenir des doublons ;
- SQL utilise `NULL` ;
- l'ordre peut être observable avec `ORDER BY` ;
- les SGBD offrent de nombreux types et extensions non prévus par le modèle initial.

Il reste néanmoins profondément inspiré de l'algèbre relationnelle.

## 1.4 Relationnel et NoSQL ne s'opposent pas simplement

Le raccourci suivant est trompeur :

> relationnel = données structurées ; NoSQL = données non structurées.

Un SGBDR moderne peut stocker JSON, tableaux, géométrie, texte intégral ou vecteurs. Le vrai choix porte plutôt sur :

- les contraintes d'intégrité ;
- le modèle d'accès ;
- les garanties transactionnelles ;
- les besoins de distribution ;
- les performances attendues ;
- l'écosystème opérationnel.

PostgreSQL, par exemple, peut combiner colonnes relationnelles classiques et documents JSONB.

---

# 2. Relation, tuple, attribut, domaine et NULL

## 2.1 Vocabulaire

Dans le modèle relationnel :

| Théorie relationnelle | SQL courant |
|---|---|
| Relation | Table |
| Tuple | Ligne |
| Attribut | Colonne |
| Domaine | Ensemble de valeurs possibles / type |

Une relation représente un **ensemble de faits du même type**.

Exemple :

```text
client(id, nom, email, date_creation)
```

## 2.2 L'ordre n'est pas une propriété d'une table

Sans `ORDER BY`, le SGBD ne garantit pas l'ordre des lignes.

Cette requête :

```sql
SELECT id, nom
FROM client;
```

ne promet aucun ordre stable.

Pour obtenir un ordre déterministe :

```sql
SELECT id, nom
FROM client
ORDER BY nom, id;
```

Le second critère `id` sert ici à départager les noms identiques.

## 2.3 Types de données

Quelques familles courantes :

- entiers : `SMALLINT`, `INTEGER`, `BIGINT` ;
- nombres exacts : `NUMERIC(p, s)` / `DECIMAL(p, s)` ;
- nombres flottants : `REAL`, `DOUBLE PRECISION` ;
- texte : `CHAR`, `VARCHAR`, `TEXT` ;
- booléens : `BOOLEAN` ;
- dates : `DATE` ;
- heures : `TIME` ;
- instants : `TIMESTAMP` ;
- données binaires ;
- UUID ;
- JSON ;
- tableaux et types spécifiques selon le SGBD.

Pour de l'argent, on préfère généralement un **type décimal exact** à un flottant binaire.

```sql
prix NUMERIC(12, 2)
```

## 2.4 `NULL` signifie « valeur inconnue ou absente »

`NULL` n'est ni :

- zéro ;
- une chaîne vide ;
- `false` ;
- la chaîne `'NULL'`.

La comparaison suivante est incorrecte :

```sql
WHERE email = NULL
```

Il faut écrire :

```sql
WHERE email IS NULL
```

ou :

```sql
WHERE email IS NOT NULL
```

## 2.5 Logique à trois valeurs

Avec `NULL`, SQL utilise une logique à trois valeurs :

- vrai ;
- faux ;
- inconnu.

Cela peut produire des résultats surprenants avec `NOT IN`.

Préférer souvent :

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM blacklist b
    WHERE b.email = c.email
)
```

à un `NOT IN` lorsque la sous-requête peut retourner des `NULL`.

---

# 3. Clés et contraintes d'intégrité

## 3.1 Clé candidate

Une **clé candidate** est un ensemble minimal d'attributs capable d'identifier un tuple de manière unique.

Exemples possibles pour un utilisateur :

- `id` ;
- `email`, si l'unicité métier est réellement garantie.

La clé primaire est l'une des clés candidates choisie comme identifiant principal.

## 3.2 Clé primaire

```sql
CREATE TABLE client (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nom TEXT NOT NULL
);
```

Une clé primaire implique :

- unicité ;
- absence de `NULL`.

## 3.3 Clé naturelle ou clé artificielle ?

### Clé naturelle

Exemple : ISBN d'un livre.

Avantages :

- possède déjà un sens métier ;
- évite parfois une colonne supplémentaire.

Inconvénients :

- peut changer ;
- peut être longue ;
- peut dépendre d'une règle métier externe.

### Clé artificielle

Exemple : entier généré ou UUID.

```sql
id UUID PRIMARY KEY
```

Elle ne remplace pas les contraintes métier : si `email` doit être unique, il faut **toujours** poser cette contrainte.

```sql
CONSTRAINT client_email_unique UNIQUE (email)
```

## 3.4 Clés étrangères

```sql
CREATE TABLE commande (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    client_id BIGINT NOT NULL,
    CONSTRAINT commande_client_fk
        FOREIGN KEY (client_id)
        REFERENCES client(id)
);
```

La clé étrangère garantit l'**intégrité référentielle**.

## 3.5 Actions référentielles

Exemples :

```sql
ON DELETE CASCADE
ON DELETE SET NULL
ON DELETE RESTRICT
```

`CASCADE` n'est pas « mieux » : il faut choisir le comportement correspondant au métier.

Une cascade mal pensée peut supprimer une grande quantité de données.

## 3.6 `CHECK`

```sql
CREATE TABLE produit (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nom TEXT NOT NULL,
    prix NUMERIC(12, 2) NOT NULL,
    stock INTEGER NOT NULL,
    CONSTRAINT produit_prix_positif CHECK (prix >= 0),
    CONSTRAINT produit_stock_positif CHECK (stock >= 0)
);
```

## 3.7 Contraintes de domaine

Quand une règle peut être garantie par la base, il est souvent préférable de la placer dans la base plutôt que de compter uniquement sur le code applicatif.

Exemple :

```sql
CREATE TABLE reservation (
    debut TIMESTAMP NOT NULL,
    fin TIMESTAMP NOT NULL,
    CHECK (fin > debut)
);
```

La validation côté application reste utile pour l'expérience utilisateur, mais elle ne remplace pas l'intégrité côté base.

---

# 4. Modélisation conceptuelle et logique

## 4.1 Partir du métier

Avant d'écrire du SQL, il faut identifier :

- les entités ;
- leurs attributs ;
- leurs identifiants ;
- leurs relations ;
- leurs cardinalités ;
- les invariants métier.

Exemple de domaine : une bibliothèque.

Entités :

- auteur ;
- livre ;
- exemplaire ;
- adhérent ;
- emprunt.

## 4.2 Relation un-à-plusieurs

Un client peut posséder plusieurs commandes.

```text
client 1 ---- N commande
```

La clé étrangère est placée du côté `N` :

```sql
commande.client_id REFERENCES client(id)
```

## 4.3 Relation plusieurs-à-plusieurs

Un livre peut avoir plusieurs auteurs, et un auteur plusieurs livres.

On crée une table d'association :

```sql
CREATE TABLE livre_auteur (
    livre_id BIGINT NOT NULL REFERENCES livre(id),
    auteur_id BIGINT NOT NULL REFERENCES auteur(id),
    position SMALLINT,
    PRIMARY KEY (livre_id, auteur_id)
);
```

## 4.4 Relation un-à-un

Une relation 1:1 est souvent matérialisée par une clé étrangère `UNIQUE`.

```sql
CREATE TABLE profil (
    utilisateur_id BIGINT PRIMARY KEY
        REFERENCES utilisateur(id),
    biographie TEXT
);
```

## 4.5 Valeur multivaluée : éviter les listes concaténées

Mauvais :

```text
tags = "python,sql,linux"
```

Une représentation relationnelle utilise normalement une table dédiée :

```text
article
article_tag
tag
```

Exception : un type tableau/JSON peut être pertinent si la donnée est réellement traitée comme une valeur atomique par le métier.

## 4.6 Modèle physique

Le passage au modèle physique ajoute des choix comme :

- types exacts ;
- index ;
- partitionnement ;
- conventions de noms ;
- stockage JSON ;
- stratégie d'identifiants.

Ces décisions ne doivent pas polluer trop tôt la modélisation métier.

---

# 5. Normalisation et dépendances fonctionnelles

## 5.1 Pourquoi normaliser ?

La normalisation cherche à éviter :

- anomalies d'insertion ;
- anomalies de mise à jour ;
- anomalies de suppression ;
- redondances non maîtrisées.

Exemple mal conçu :

```text
commande(
    commande_id,
    client_id,
    client_nom,
    client_email,
    produit_id,
    produit_nom,
    quantite
)
```

Le nom du client serait répété pour chaque ligne de commande.

## 5.2 Dépendance fonctionnelle

On écrit :

```text
A -> B
```

si une valeur de `A` détermine une seule valeur de `B`.

Exemple :

```text
client_id -> nom, email
```

## 5.3 Première forme normale — 1NF

Une table doit avoir des valeurs atomiques au regard du modèle choisi et ne pas contenir de groupes répétitifs.

Mauvais exemple conceptuel :

```text
client(id, telephone1, telephone2, telephone3)
```

On préfère :

```text
client(id, ...)
telephone(id, client_id, numero, type)
```

## 5.4 Deuxième forme normale — 2NF

Une table en 1NF est en 2NF si chaque attribut non-clé dépend de **toute** la clé candidate et non d'une partie seulement.

Le problème apparaît surtout avec les clés composées.

## 5.5 Troisième forme normale — 3NF

Une table est en 3NF si les attributs non-clés ne dépendent pas transitivement d'une clé via un autre attribut non-clé.

Mauvais exemple :

```text
employe(id, departement_id, departement_nom)
```

Si :

```text
id -> departement_id
departement_id -> departement_nom
```

alors `departement_nom` appartient plutôt à la table `departement`.

## 5.6 BCNF

La forme normale de Boyce-Codd impose que tout déterminant d'une dépendance fonctionnelle non triviale soit une super-clé.

Elle est plus stricte que la 3NF.

## 5.7 Dénormalisation

La dénormalisation est une **optimisation consciente**, pas une excuse pour éviter la modélisation.

Exemples possibles :

- compteur pré-calculé ;
- vue matérialisée ;
- table analytique ;
- duplication contrôlée dans un système distribué.

Avant de dénormaliser :

1. mesurer ;
2. identifier le vrai goulet d'étranglement ;
3. documenter la source de vérité ;
4. définir comment la copie reste cohérente.

---

# 6. DDL : créer et faire évoluer un schéma

DDL signifie **Data Definition Language**.

## 6.1 `CREATE TABLE`

Exemple PostgreSQL :

```sql
CREATE TABLE client (
    id BIGINT GENERATED ALWAYS AS IDENTITY,
    email TEXT NOT NULL,
    nom TEXT NOT NULL,
    actif BOOLEAN NOT NULL DEFAULT TRUE,
    cree_le TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT client_pk PRIMARY KEY (id),
    CONSTRAINT client_email_unique UNIQUE (email)
);
```

## 6.2 `IDENTITY` plutôt que les mécanismes historiques

Le standard SQL fournit les colonnes identity :

```sql
GENERATED ALWAYS AS IDENTITY
```

ou :

```sql
GENERATED BY DEFAULT AS IDENTITY
```

Dans un nouveau schéma PostgreSQL, elles sont généralement préférables au pseudo-type historique `SERIAL`.

## 6.3 `ALTER TABLE`

```sql
ALTER TABLE client
ADD COLUMN telephone TEXT;
```

```sql
ALTER TABLE client
ADD CONSTRAINT client_telephone_unique UNIQUE (telephone);
```

## 6.4 Renommage

```sql
ALTER TABLE client
RENAME COLUMN telephone TO telephone_principal;
```

## 6.5 Suppression

```sql
ALTER TABLE client
DROP COLUMN telephone_principal;
```

Une suppression de colonne est une opération potentiellement destructive et doit être déployée avec une stratégie de migration.

## 6.6 `DROP TABLE`

```sql
DROP TABLE archive_temporaire;
```

`DROP` est du DDL : il supprime l'objet lui-même.

## 6.7 Schémas SQL

Dans PostgreSQL :

```sql
CREATE SCHEMA comptabilite;
```

Puis :

```sql
CREATE TABLE comptabilite.facture (...);
```

Un schéma permet notamment :

- d'organiser les objets ;
- d'éviter les collisions de noms ;
- de segmenter les permissions.

---

# 7. DML : INSERT, UPDATE et DELETE

DML signifie **Data Manipulation Language**.

## 7.1 `INSERT`

Toujours préférer les colonnes explicites :

```sql
INSERT INTO client (email, nom)
VALUES ('ada@example.org', 'Ada Lovelace');
```

Plutôt que :

```sql
INSERT INTO client
VALUES (...);
```

qui dépend fortement de l'ordre physique des colonnes.

## 7.2 Insérer plusieurs lignes

```sql
INSERT INTO produit (nom, prix)
VALUES
    ('Clavier', 79.90),
    ('Souris', 39.90),
    ('Écran', 349.00);
```

## 7.3 `RETURNING` avec PostgreSQL

```sql
INSERT INTO client (email, nom)
VALUES ('grace@example.org', 'Grace Hopper')
RETURNING id, cree_le;
```

Cela évite une requête supplémentaire pour retrouver l'identifiant généré.

## 7.4 `UPDATE`

```sql
UPDATE produit
SET prix = prix * 1.02
WHERE categorie_id = 12;
```

Avant un `UPDATE` important, il est utile de tester le même prédicat avec :

```sql
SELECT *
FROM produit
WHERE categorie_id = 12;
```

## 7.5 `DELETE`

```sql
DELETE FROM session_web
WHERE expire_le < CURRENT_TIMESTAMP;
```

Attention :

```sql
DELETE FROM session_web;
```

supprime toutes les lignes.

## 7.6 Upsert

PostgreSQL :

```sql
INSERT INTO compteur (cle, valeur)
VALUES ('visites', 1)
ON CONFLICT (cle)
DO UPDATE
SET valeur = compteur.valeur + 1;
```

La syntaxe varie selon les SGBD.

---

# 8. SELECT : filtrer, trier et projeter

## 8.1 Projection

Éviter `SELECT *` dans le code applicatif lorsque seules quelques colonnes sont nécessaires.

```sql
SELECT id, nom, email
FROM client;
```

Avantages :

- contrat plus explicite ;
- moins de données transférées ;
- meilleure stabilité lors de l'ajout de colonnes.

## 8.2 `WHERE`

```sql
SELECT id, nom
FROM client
WHERE actif = TRUE;
```

## 8.3 `AND`, `OR`, `NOT`

```sql
SELECT *
FROM produit
WHERE actif
  AND prix BETWEEN 10 AND 100
  AND stock > 0;
```

Parenthéser les expressions complexes :

```sql
WHERE actif
  AND (categorie = 'livre' OR categorie = 'magazine')
```

## 8.4 `LIKE`

```sql
WHERE nom LIKE 'Ada%'
```

PostgreSQL propose aussi `ILIKE` pour une comparaison insensible à la casse selon sa sémantique propre.

## 8.5 `IN`

```sql
WHERE statut IN ('nouveau', 'paye', 'expedie')
```

## 8.6 `ORDER BY`

```sql
SELECT id, nom, cree_le
FROM client
ORDER BY cree_le DESC, id DESC;
```

## 8.7 Pagination : `LIMIT/OFFSET`

```sql
SELECT id, nom
FROM client
ORDER BY id
LIMIT 50 OFFSET 100;
```

Cette pagination devient coûteuse pour de grands offsets et peut être instable lorsque les données changent.

## 8.8 Pagination par curseur

```sql
SELECT id, nom
FROM client
WHERE id > :dernier_id
ORDER BY id
LIMIT 50;
```

Cette stratégie est souvent appelée **keyset pagination**.

---

# 9. Jointures

## 9.1 `INNER JOIN`

```sql
SELECT
    c.id,
    c.nom,
    co.id AS commande_id
FROM client c
JOIN commande co
    ON co.client_id = c.id;
```

Une ligne n'apparaît que si une correspondance existe des deux côtés.

## 9.2 `LEFT JOIN`

```sql
SELECT
    c.id,
    c.nom,
    COUNT(co.id) AS nb_commandes
FROM client c
LEFT JOIN commande co
    ON co.client_id = c.id
GROUP BY c.id, c.nom;
```

Les clients sans commande restent présents.

## 9.3 Piège avec `LEFT JOIN` et `WHERE`

Cette requête :

```sql
SELECT c.id, co.id
FROM client c
LEFT JOIN commande co
  ON co.client_id = c.id
WHERE co.statut = 'paye';
```

élimine les lignes où `co` est `NULL`, et se comporte donc souvent comme un `INNER JOIN` pour ce prédicat.

Si le filtre appartient à la relation jointe :

```sql
SELECT c.id, co.id
FROM client c
LEFT JOIN commande co
  ON co.client_id = c.id
 AND co.statut = 'paye';
```

## 9.4 `FULL OUTER JOIN`

```sql
SELECT ...
FROM a
FULL OUTER JOIN b ON ...;
```

Conserve les lignes non appariées des deux côtés.

## 9.5 `CROSS JOIN`

Produit cartésien :

```sql
SELECT *
FROM couleur
CROSS JOIN taille;
```

À utiliser consciemment : le nombre de lignes est multiplié.

## 9.6 Self join

Pour une hiérarchie simple :

```sql
CREATE TABLE employe (
    id BIGINT PRIMARY KEY,
    nom TEXT NOT NULL,
    manager_id BIGINT REFERENCES employe(id)
);
```

Puis :

```sql
SELECT
    e.nom AS employe,
    m.nom AS manager
FROM employe e
LEFT JOIN employe m
    ON m.id = e.manager_id;
```

---

# 10. Agrégats, GROUP BY et HAVING

## 10.1 Agrégats courants

```sql
COUNT(*)
COUNT(colonne)
SUM(colonne)
AVG(colonne)
MIN(colonne)
MAX(colonne)
```

`COUNT(*)` compte les lignes. `COUNT(colonne)` ignore les `NULL`.

## 10.2 `GROUP BY`

```sql
SELECT
    client_id,
    COUNT(*) AS nb_commandes,
    SUM(total) AS chiffre_affaires
FROM commande
GROUP BY client_id;
```

## 10.3 `HAVING`

`WHERE` filtre **avant** l'agrégation. `HAVING` filtre les groupes **après** l'agrégation.

```sql
SELECT client_id, SUM(total) AS total
FROM commande
WHERE statut = 'paye'
GROUP BY client_id
HAVING SUM(total) >= 1000;
```

## 10.4 Agrégation conditionnelle

Standard SQL :

```sql
SELECT
    COUNT(*) AS total,
    SUM(CASE WHEN statut = 'paye' THEN 1 ELSE 0 END) AS payees
FROM commande;
```

PostgreSQL propose aussi `FILTER` :

```sql
SELECT
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE statut = 'paye') AS payees
FROM commande;
```

---

# 11. Sous-requêtes et CTE

## 11.1 Sous-requête scalaire

```sql
SELECT nom
FROM produit
WHERE prix > (
    SELECT AVG(prix)
    FROM produit
);
```

## 11.2 `EXISTS`

```sql
SELECT c.id, c.nom
FROM client c
WHERE EXISTS (
    SELECT 1
    FROM commande co
    WHERE co.client_id = c.id
      AND co.statut = 'impaye'
);
```

Pour tester l'existence, `EXISTS` exprime souvent mieux l'intention qu'une jointure suivie de `DISTINCT`.

## 11.3 CTE

```sql
WITH ventes AS (
    SELECT client_id, SUM(total) AS total
    FROM commande
    WHERE statut = 'paye'
    GROUP BY client_id
)
SELECT c.nom, v.total
FROM ventes v
JOIN client c ON c.id = v.client_id
WHERE v.total >= 1000;
```

## 11.4 CTE récursive

```sql
WITH RECURSIVE hierarchie AS (
    SELECT id, nom, manager_id, 0 AS niveau
    FROM employe
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.id, e.nom, e.manager_id, h.niveau + 1
    FROM employe e
    JOIN hierarchie h
      ON e.manager_id = h.id
)
SELECT *
FROM hierarchie
ORDER BY niveau, id;
```

Les CTE récursives permettent notamment de parcourir :

- arbres ;
- hiérarchies ;
- graphes simples.

---

# 12. Fonctions fenêtre

Les fonctions fenêtre calculent une valeur sur un ensemble de lignes **sans réduire** cet ensemble comme le ferait `GROUP BY`.

## 12.1 Numérotation

```sql
SELECT
    client_id,
    id,
    total,
    ROW_NUMBER() OVER (
        PARTITION BY client_id
        ORDER BY total DESC
    ) AS rang
FROM commande;
```

## 12.2 Classement

```sql
RANK()
DENSE_RANK()
ROW_NUMBER()
```

ont des comportements différents en cas d'égalité.

## 12.3 Somme cumulée

```sql
SELECT
    date_commande,
    total,
    SUM(total) OVER (
        ORDER BY date_commande, id
    ) AS cumul
FROM commande;
```

## 12.4 `LAG` et `LEAD`

```sql
SELECT
    mois,
    chiffre_affaires,
    LAG(chiffre_affaires) OVER (ORDER BY mois) AS mois_precedent
FROM vente_mensuelle;
```

Très utile pour :

- séries temporelles ;
- comparaisons ;
- détection de changements.

---

# 13. Ensembles et logique SQL

## 13.1 `UNION`

```sql
SELECT email FROM client_fr
UNION
SELECT email FROM client_be;
```

`UNION` élimine les doublons.

## 13.2 `UNION ALL`

```sql
SELECT email FROM client_fr
UNION ALL
SELECT email FROM client_be;
```

Conserve les doublons et évite le coût de déduplication lorsque celle-ci n'est pas nécessaire.

## 13.3 `INTERSECT`

```sql
SELECT email FROM newsletter
INTERSECT
SELECT email FROM client;
```

## 13.4 `EXCEPT`

```sql
SELECT email FROM newsletter
EXCEPT
SELECT email FROM client;
```

## 13.5 `DISTINCT`

```sql
SELECT DISTINCT pays
FROM client;
```

Ne pas utiliser `DISTINCT` comme pansement automatique pour corriger une jointure qui multiplie les lignes : comprendre d'abord pourquoi les doublons existent.

---

# 14. Vues et vues matérialisées

## 14.1 Vue

```sql
CREATE VIEW client_actif AS
SELECT id, nom, email
FROM client
WHERE actif;
```

Une vue stocke essentiellement une définition de requête.

## 14.2 Pourquoi utiliser une vue ?

- encapsuler une requête ;
- présenter une API SQL stable ;
- masquer certaines colonnes ;
- centraliser une logique de lecture.

## 14.3 Vue matérialisée

PostgreSQL :

```sql
CREATE MATERIALIZED VIEW ca_mensuel AS
SELECT
    date_trunc('month', date_commande) AS mois,
    SUM(total) AS total
FROM commande
WHERE statut = 'paye'
GROUP BY 1;
```

Puis :

```sql
REFRESH MATERIALIZED VIEW ca_mensuel;
```

Contrairement à une vue classique, les résultats sont stockés physiquement et peuvent devenir obsolètes entre deux rafraîchissements.

---

# 15. Transactions et propriétés ACID

## 15.1 Définition

Une transaction regroupe plusieurs opérations en une unité logique.

```sql
BEGIN;

UPDATE compte
SET solde = solde - 100
WHERE id = 1;

UPDATE compte
SET solde = solde + 100
WHERE id = 2;

COMMIT;
```

Si une étape échoue :

```sql
ROLLBACK;
```

## 15.2 Atomicité

Une transaction est appliquée entièrement ou pas du tout.

## 15.3 Cohérence — Consistency

La transaction fait passer la base d'un état valide à un autre état valide, au regard des contraintes définies.

ACID ne peut cependant pas inventer des règles métier absentes du schéma et du code.

## 15.4 Isolation

Des transactions concurrentes doivent être suffisamment isolées pour produire un comportement acceptable.

## 15.5 Durabilité

Après `COMMIT`, les données validées doivent survivre aux défaillances couvertes par les garanties du SGBD et de sa configuration.

## 15.6 Savepoints

```sql
BEGIN;

INSERT INTO commande (...) VALUES (...);
SAVEPOINT commande_creee;

INSERT INTO ligne_commande (...) VALUES (...);

-- En cas de problème :
ROLLBACK TO SAVEPOINT commande_creee;

COMMIT;
```

---

# 16. Niveaux d'isolation, anomalies et MVCC

C'est un chapitre essentiel pour comprendre les erreurs qui apparaissent uniquement en production sous charge.

## 16.1 Anomalies classiques

### Dirty read

Une transaction lit une modification non validée d'une autre transaction.

### Non-repeatable read

Deux lectures successives de la même ligne dans une transaction produisent des valeurs différentes à cause d'un `UPDATE` concurrent validé.

### Phantom read

Une même requête par prédicat voit apparaître ou disparaître des lignes à cause d'une transaction concurrente.

### Lost update

Deux transactions lisent une valeur, calculent chacune une nouvelle valeur, puis l'une écrase la modification de l'autre.

### Write skew

Deux transactions modifient des lignes différentes tout en violant ensemble un invariant global.

## 16.2 Niveaux SQL

Les niveaux standard sont :

1. `READ UNCOMMITTED` ;
2. `READ COMMITTED` ;
3. `REPEATABLE READ` ;
4. `SERIALIZABLE`.

Le comportement exact doit être vérifié dans la documentation du SGBD.

PostgreSQL, par exemple, traite `READ UNCOMMITTED` comme `READ COMMITTED`.

## 16.3 MVCC

PostgreSQL utilise le **Multi-Version Concurrency Control**.

Chaque transaction observe un snapshot correspondant à des versions visibles des lignes.

L'idée fondamentale est que lecture et écriture peuvent souvent progresser sans qu'une simple lecture doive verrouiller les lignes comme dans un modèle de verrouillage pessimiste généralisé.

## 16.4 `READ COMMITTED`

C'est le niveau par défaut de PostgreSQL.

Chaque instruction reçoit son propre snapshot au début de l'instruction.

Deux `SELECT` successifs dans la même transaction peuvent donc observer des états différents.

## 16.5 `REPEATABLE READ`

Le snapshot reste stable pendant la transaction, sous réserve de la sémantique précise du SGBD.

## 16.6 `SERIALIZABLE`

Le résultat doit être équivalent à une exécution séquentielle valide des transactions.

Dans PostgreSQL, Serializable Snapshot Isolation peut détecter des conflits et provoquer une **erreur de sérialisation**.

Une application utilisant `SERIALIZABLE` doit donc être capable de **réessayer une transaction complète**.

Pseudo-code :

```python
for tentative in range(MAX_RETRIES):
    try:
        executer_transaction()
        break
    except SerializationFailure:
        attendre_avec_jitter()
```

---

# 17. Verrous, concurrence et deadlocks

## 17.1 Verrouillage explicite de ligne

```sql
SELECT id, solde
FROM compte
WHERE id = 42
FOR UPDATE;
```

Cette instruction indique que la ligne va être modifiée et empêche certaines modifications concurrentes incompatibles jusqu'à la fin de la transaction.

## 17.2 `FOR NO KEY UPDATE`, `FOR SHARE`, etc.

PostgreSQL propose plusieurs forces de verrouillage de lignes. Il faut sélectionner la plus faible compatible avec le besoin.

## 17.3 `SKIP LOCKED`

Pour une file de travaux :

```sql
SELECT id
FROM job
WHERE statut = 'en_attente'
ORDER BY id
FOR UPDATE SKIP LOCKED
LIMIT 1;
```

Plusieurs workers peuvent ainsi prendre des travaux sans attendre les mêmes lignes.

Cette technique nécessite néanmoins une conception attentive de la file et de la reprise après panne.

## 17.4 Deadlock

Transaction A :

```text
verrouille ligne 1
attend ligne 2
```

Transaction B :

```text
verrouille ligne 2
attend ligne 1
```

Le SGBD détecte généralement le cycle et annule une transaction.

## 17.5 Éviter les deadlocks

- verrouiller les ressources dans un ordre stable ;
- garder les transactions courtes ;
- éviter les interactions utilisateur au milieu d'une transaction ;
- prévoir le retry des erreurs transitoires.

## 17.6 Optimistic locking

Une technique applicative consiste à utiliser un numéro de version :

```sql
UPDATE document
SET contenu = :contenu,
    version = version + 1
WHERE id = :id
  AND version = :version_lue;
```

Si aucune ligne n'est modifiée, un concurrent a gagné la course.

---

# 18. Index

Un index accélère certaines lectures au prix :

- d'espace disque ;
- de coût lors des écritures ;
- de maintenance ;
- de complexité supplémentaire.

## 18.1 B-tree

C'est l'index généraliste le plus courant.

```sql
CREATE INDEX commande_client_idx
ON commande (client_id);
```

Il est adapté à de nombreux prédicats d'égalité et d'ordre.

## 18.2 Index unique

```sql
CREATE UNIQUE INDEX client_email_uq
ON client (email);
```

Pour une règle métier d'unicité, une contrainte `UNIQUE` est souvent plus expressive.

## 18.3 Index multicolonne

```sql
CREATE INDEX commande_client_date_idx
ON commande (client_id, date_commande DESC);
```

L'ordre des colonnes est important.

Un index `(client_id, date_commande)` n'est pas équivalent à `(date_commande, client_id)`.

## 18.4 Index partiel — PostgreSQL

```sql
CREATE INDEX commande_impayee_idx
ON commande (client_id)
WHERE statut = 'impaye';
```

Très utile lorsqu'une petite fraction des lignes correspond au prédicat intéressant.

## 18.5 Index d'expression — PostgreSQL

```sql
CREATE INDEX client_email_lower_idx
ON client (lower(email));
```

Une requête compatible :

```sql
SELECT id
FROM client
WHERE lower(email) = lower(:email);
```

## 18.6 `INCLUDE` — PostgreSQL

```sql
CREATE INDEX commande_client_idx
ON commande (client_id)
INCLUDE (total, statut);
```

Les colonnes incluses peuvent permettre certains index-only scans sans participer à la clé de recherche.

## 18.7 Autres familles PostgreSQL

- B-tree ;
- Hash ;
- GiST ;
- SP-GiST ;
- GIN ;
- BRIN.

Exemples :

- GIN : recherche dans JSONB, tableaux, full text ;
- GiST : géométrie, ranges, extensions ;
- BRIN : très grosses tables dont les valeurs sont corrélées à l'ordre physique.

## 18.8 Trop d'index est un problème

Chaque `INSERT`, `DELETE` et souvent `UPDATE` doit entretenir les index concernés.

Avant d'ajouter un index :

1. identifier une requête importante ;
2. mesurer son plan ;
3. concevoir l'index ;
4. re-mesurer ;
5. vérifier le coût sur les écritures et le stockage.

---

# 19. Plans d'exécution et optimisation

## 19.1 `EXPLAIN`

PostgreSQL :

```sql
EXPLAIN
SELECT *
FROM commande
WHERE client_id = 42;
```

Le plan peut montrer :

- `Seq Scan` ;
- `Index Scan` ;
- `Index Only Scan` ;
- `Bitmap Heap Scan` ;
- différentes stratégies de jointure.

## 19.2 `EXPLAIN ANALYZE`

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT ...;
```

`ANALYZE` **exécute réellement la requête**.

Attention avec :

```sql
DELETE
UPDATE
INSERT
```

Une technique de test peut être :

```sql
BEGIN;
EXPLAIN (ANALYZE, BUFFERS)
DELETE FROM ...;
ROLLBACK;
```

à condition de comprendre les effets annexes éventuels.

## 19.3 Estimation du nombre de lignes

L'optimiseur choisit son plan à partir de statistiques.

Un indice essentiel est l'écart entre :

```text
rows estimées
```

et :

```text
actual rows
```

Un gros écart peut signaler :

- statistiques insuffisantes ;
- données très corrélées ;
- distribution atypique ;
- prédicat difficile à estimer.

## 19.4 Sargabilité

Une condition « sargable » permet d'utiliser efficacement un index.

Exemple potentiellement défavorable :

```sql
WHERE EXTRACT(YEAR FROM date_commande) = 2026
```

On préfère souvent une plage :

```sql
WHERE date_commande >= DATE '2026-01-01'
  AND date_commande <  DATE '2027-01-01'
```

## 19.5 Éviter N+1

Un ORM peut produire :

```text
1 requête pour les commandes
+ 1 requête par client
```

Pour 1000 commandes, cela peut faire 1001 requêtes.

Solutions :

- jointure ;
- eager loading ;
- batch loading ;
- requête spécialisée.

## 19.6 Optimiser dans le bon ordre

1. vérifier la correction du modèle ;
2. mesurer la requête ;
3. examiner le plan ;
4. réduire les données inutiles ;
5. vérifier les index ;
6. revoir la requête ;
7. seulement ensuite envisager cache ou dénormalisation.

---

# 20. Sécurité et contrôle d'accès

## 20.1 Principe du moindre privilège

Une application ne devrait pas se connecter avec un superutilisateur.

Exemple conceptuel :

```sql
CREATE ROLE app LOGIN PASSWORD '...';
GRANT CONNECT ON DATABASE boutique TO app;
GRANT USAGE ON SCHEMA public TO app;
GRANT SELECT, INSERT, UPDATE, DELETE
ON TABLE client, commande, ligne_commande
TO app;
```

Le détail varie selon le SGBD.

## 20.2 Séparer les rôles

Exemples :

- rôle de migration ;
- rôle applicatif ;
- rôle lecture seule ;
- rôle backup ;
- rôle administrateur.

Cela limite l'impact d'un secret compromis.

## 20.3 Secrets

Ne jamais placer un mot de passe de production :

- dans Git ;
- dans une image Docker ;
- en clair dans un notebook partagé ;
- dans une URL qui sera journalisée.

Utiliser un gestionnaire de secrets ou un mécanisme de credentials adapté.

## 20.4 Chiffrement en transit

Utiliser TLS pour les connexions réseau lorsque le réseau n'est pas entièrement de confiance.

Surtout : **valider le certificat**.

Désactiver la validation TLS transforme le chiffrement en protection très incomplète contre un attaquant actif.

## 20.5 Chiffrement au repos

Peut être assuré par plusieurs couches :

- disque/LUKS ;
- stockage cloud chiffré ;
- chiffrement natif du SGBD ;
- chiffrement applicatif de certaines colonnes.

Le bon niveau dépend du modèle de menace.

## 20.6 Row-Level Security — PostgreSQL

```sql
ALTER TABLE document ENABLE ROW LEVEL SECURITY;
```

Puis une politique peut limiter les lignes visibles selon l'identité ou le contexte de session.

RLS ne dispense pas de comprendre les rôles et les privilèges.

---

# 21. SQL dynamique et injection SQL

## 21.1 Le problème

Mauvais code Python :

```python
sql = f"SELECT * FROM utilisateur WHERE email = '{email}'"
cursor.execute(sql)
```

Si `email` contient du SQL, la requête change de structure.

## 21.2 Paramétrer les valeurs

Avec l'API `sqlite3` :

```python
cursor.execute(
    "SELECT id, email FROM utilisateur WHERE email = ?",
    (email,),
)
```

Avec un driver PostgreSQL, la syntaxe de placeholder dépend du driver utilisé.

L'idée reste la même :

> les **valeurs** sont transmises séparément du texte SQL.

## 21.3 Paramètres et identifiants

Un paramètre ne représente généralement pas un nom de table ou de colonne.

Mauvais raisonnement :

```sql
SELECT * FROM ?
```

Pour un nom de colonne dynamique :

- utiliser une liste blanche ;
- utiliser l'API de composition SQL du driver.

## 21.4 `ORDER BY` dynamique

Exemple de liste blanche :

```python
colonnes_autorisees = {
    "nom": "nom",
    "date": "cree_le",
}

colonne = colonnes_autorisees[tri_demande]
```

Puis construire uniquement à partir d'identifiants prédéfinis.

---

# 22. Migrations de schéma

## 22.1 Le schéma est du code

Une modification de schéma doit être :

- versionnée ;
- relue ;
- testée ;
- reproductible ;
- accompagnée d'une stratégie de retour ou de récupération.

Outils courants :

- Alembic ;
- Flyway ;
- Liquibase ;
- migrations des frameworks.

## 22.2 Migration compatible avec plusieurs versions d'application

Pour un déploiement sans interruption, une migration doit souvent suivre la stratégie :

```text
expand -> migrate -> contract
```

### Expand

Ajouter une nouvelle structure compatible avec l'ancien code.

### Migrate

Copier ou transformer les données.

### Contract

Une fois tout le code migré, supprimer l'ancienne structure.

## 22.3 Exemple : renommer une colonne sans interruption

Au lieu d'un renommage brutal :

1. ajouter la nouvelle colonne ;
2. déployer un code écrivant les deux colonnes ;
3. backfill ;
4. faire lire la nouvelle colonne ;
5. arrêter l'écriture dans l'ancienne ;
6. supprimer l'ancienne dans une migration ultérieure.

## 22.4 Migrations longues

Une opération DDL peut :

- verrouiller une table ;
- réécrire une table ;
- générer beaucoup de WAL/binlog ;
- saturer les I/O.

Toujours tester les migrations importantes sur un volume réaliste.

---

# 23. Sauvegarde, restauration et reprise après incident

## 23.1 Une sauvegarde non restaurée n'est pas une sauvegarde vérifiée

Il faut tester régulièrement la restauration.

## 23.2 Sauvegarde logique PostgreSQL

```bash
pg_dump -Fc -d boutique -f boutique.dump
```

Restauration :

```bash
createdb boutique_restoree
pg_restore -d boutique_restoree boutique.dump
```

## 23.3 Sauvegarde du cluster

Pour des scénarios de taille importante et de reprise précise, les mécanismes physiques et l'archivage du WAL deviennent importants.

## 23.4 RPO et RTO

### RPO — Recovery Point Objective

Quantité maximale de données que l'on accepte de perdre.

Exemple :

```text
RPO = 5 minutes
```

### RTO — Recovery Time Objective

Temps maximal acceptable pour rétablir le service.

```text
RTO = 30 minutes
```

La stratégie de backup doit découler de ces objectifs et non l'inverse.

## 23.5 PITR

Le **Point-In-Time Recovery** permet de restaurer un état puis de rejouer le journal jusqu'à un instant précis.

Cas classique :

```text
DELETE accidentel à 14:32:18
```

On peut restaurer jusqu'à juste avant cette opération si la chaîne de sauvegarde et de journaux est complète.

## 23.6 Sauvegarder aussi ce qui entoure la base

- rôles ;
- configuration ;
- secrets nécessaires à la restauration ;
- procédures ;
- extensions ;
- version du SGBD ;
- scripts d'infrastructure.

---

# 24. Réplication, haute disponibilité et partitionnement

## 24.1 Réplication n'est pas sauvegarde

Si un utilisateur exécute :

```sql
DELETE FROM client;
```

la suppression peut être immédiatement répliquée.

La réplication améliore la disponibilité ; elle ne remplace pas une sauvegarde historique.

## 24.2 Réplication physique

Une réplique reproduit l'état physique du serveur primaire selon les mécanismes du SGBD.

Usages :

- haute disponibilité ;
- lecture sur réplique ;
- reprise après panne.

## 24.3 Réplication logique

Réplique des modifications logiques d'objets choisis.

Elle peut être utile pour :

- migrations ;
- sous-ensembles de données ;
- intégrations ;
- certaines mises à niveau.

## 24.4 Haute disponibilité

Un système HA doit traiter :

- détection de panne ;
- élection/promotion ;
- routage des clients ;
- prévention du split-brain ;
- reconnexion ;
- observabilité.

« Deux serveurs PostgreSQL » ne suffisent pas à former une solution HA complète.

## 24.5 Partitionnement

Une grosse table peut être découpée en partitions.

Exemple PostgreSQL :

```sql
CREATE TABLE evenement (
    id BIGINT NOT NULL,
    cree_le TIMESTAMPTZ NOT NULL,
    payload JSONB
) PARTITION BY RANGE (cree_le);
```

Puis :

```sql
CREATE TABLE evenement_2026_08
PARTITION OF evenement
FOR VALUES FROM ('2026-08-01') TO ('2026-09-01');
```

Le partitionnement est utile pour certains volumes et cycles de vie, mais n'accélère pas automatiquement toutes les requêtes.

---

# 25. Observabilité et maintenance

## 25.1 Mesures importantes

- latence p50/p95/p99 ;
- requêtes par seconde ;
- connexions actives ;
- taux de cache ;
- I/O ;
- taille des tables et index ;
- transactions longues ;
- verrous ;
- deadlocks ;
- réplication lag ;
- temps de checkpoint ;
- stockage disponible.

## 25.2 Connexions

Une connexion SGBD a un coût.

Une application Web utilise généralement un **pool de connexions**.

Le dimensionnement n'est pas :

```text
plus de connexions = plus de performances
```

Trop de connexions peuvent augmenter :

- la mémoire ;
- la contention ;
- le coût de scheduling.

## 25.3 Transactions longues

Elles peuvent retenir :

- des verrous ;
- des versions de lignes ;
- des ressources ;
- des journaux.

Une transaction ne doit pas rester ouverte pendant qu'un utilisateur réfléchit devant un formulaire.

## 25.4 PostgreSQL : `VACUUM`

MVCC crée des versions de lignes devenues mortes.

PostgreSQL utilise `VACUUM`, généralement piloté automatiquement par **autovacuum**, pour récupérer la réutilisabilité de cet espace et maintenir certains invariants internes.

Ne pas désactiver autovacuum comme « optimisation » générale.

## 25.5 `ANALYZE`

Met à jour des statistiques utilisées par l'optimiseur.

```sql
ANALYZE commande;
```

Autovacuum peut aussi déclencher l'analyse automatique.

## 25.6 Requêtes lentes

Procédure :

1. collecter la requête et ses paramètres représentatifs ;
2. mesurer ;
3. lire le plan ;
4. regarder les volumes ;
5. vérifier les statistiques ;
6. vérifier les index ;
7. inspecter la concurrence ;
8. optimiser ;
9. re-mesurer.

---

# 26. PostgreSQL, MySQL et SQLite : différences utiles

## 26.1 PostgreSQL

PostgreSQL est un SGBDR client/serveur généraliste très riche.

Atouts courants :

- SQL avancé ;
- MVCC robuste ;
- types extensibles ;
- JSONB ;
- nombreux types d'index ;
- extensions ;
- réplication logique ;
- contraintes avancées.

Au 29 août 2026, la version stable actuelle est **PostgreSQL 18.6**, publiée le 13 août 2026. PostgreSQL 19 est encore en développement/bêta.

PostgreSQL 18 a notamment introduit ou renforcé :

- un sous-système d'I/O asynchrone ;
- les skip scans B-tree ;
- `uuidv7()` ;
- des colonnes générées virtuelles ;
- l'authentification OAuth.

## 26.2 MySQL

MySQL utilise aujourd'hui deux branches de publication :

- **LTS**, pour la stabilité à long terme ;
- **Innovation**, pour l'arrivée régulière de fonctionnalités.

En 2026 :

- MySQL 8.4 est la branche LTS ;
- MySQL 9.x est la branche Innovation.

Il faut donc éviter de choisir une version uniquement parce que son numéro est plus élevé : la politique de cycle de vie doit correspondre au besoin du projet.

## 26.3 SQLite

SQLite est une bibliothèque embarquée : il n'y a pas de serveur séparé obligatoire.

```text
application -> bibliothèque SQLite -> fichier .db
```

C'est idéal pour :

- applications desktop ;
- mobile ;
- fichiers de données applicatifs ;
- tests ;
- petits services mono-instance ;
- outils CLI.

Au 29 août 2026, la version stable est **SQLite 3.53.4**, publiée le 24 juillet 2026.

SQLite 3.53 a notamment corrigé le bug de reset WAL et enrichi `ALTER TABLE`.

## 26.4 SQLite n'est pas « un petit PostgreSQL »

Différences notables :

- typage dynamique avec type affinity ;
- modèle de concurrence spécifique au fichier ;
- jeu de fonctionnalités différent ;
- DDL et fonctions disponibles différents.

Une application testée uniquement avec SQLite peut donc se comporter différemment en production sur PostgreSQL.

## 26.5 Choisir

| Besoin | Choix souvent pertinent |
|---|---|
| Application Web généraliste | PostgreSQL / MySQL |
| Base embarquée locale | SQLite |
| SQL avancé et extensions | PostgreSQL |
| Écosystème MySQL existant | MySQL |
| Tests unitaires simples | SQLite, avec prudence si la prod utilise un autre SGBD |

Le choix doit rester fondé sur des besoins concrets.

---

# 27. Python et bases relationnelles

## 27.1 DB-API et contexte transactionnel

Exemple SQLite moderne :

```python
import sqlite3

with sqlite3.connect("app.db") as connexion:
    connexion.execute("PRAGMA foreign_keys = ON")

    connexion.execute(
        """
        CREATE TABLE IF NOT EXISTS utilisateur (
            id INTEGER PRIMARY KEY,
            email TEXT NOT NULL UNIQUE,
            nom TEXT NOT NULL
        )
        """
    )

    connexion.execute(
        "INSERT INTO utilisateur (email, nom) VALUES (?, ?)",
        ("ada@example.org", "Ada"),
    )
```

Le bloc `with` facilite la gestion du commit/rollback selon le comportement de l'API utilisée.

## 27.2 Activer explicitement les clés étrangères SQLite

Selon la configuration, il est prudent de vérifier :

```python
connexion.execute("PRAGMA foreign_keys = ON")
```

avant de compter sur l'intégrité référentielle.

## 27.3 Lire des lignes

```python
connexion.row_factory = sqlite3.Row

curseur = connexion.execute(
    "SELECT id, email, nom FROM utilisateur ORDER BY id"
)

for ligne in curseur:
    print(ligne["id"], ligne["email"], ligne["nom"])
```

## 27.4 Paramètres

Correct :

```python
connexion.execute(
    "SELECT id FROM utilisateur WHERE email = ?",
    (email,),
)
```

Incorrect :

```python
connexion.execute(
    f"SELECT id FROM utilisateur WHERE email = '{email}'"
)
```

## 27.5 Transactions explicites

Pour une logique métier importante, rendre la frontière transactionnelle visible dans le code.

Pseudo-code :

```python
with transaction():
    reserver_stock()
    creer_commande()
    enregistrer_paiement()
```

Ne pas disperser des `commit()` dans les fonctions métiers internes : cela rend l'atomicité difficile à raisonner.

## 27.6 PostgreSQL en Python

Pilotes courants :

- Psycopg ;
- asyncpg pour certains usages asynchrones ;
- drivers transitifs via SQLAlchemy.

Le choix dépend du modèle synchrone/asynchrone et de l'architecture.

---

# 28. ORM et SQL explicite

## 28.1 À quoi sert un ORM ?

Un ORM peut apporter :

- mapping objets ↔ lignes ;
- gestion d'unit of work ;
- génération de SQL ;
- relations ;
- migrations via outils associés.

Exemples :

- SQLAlchemy ;
- Django ORM ;
- Hibernate ;
- Entity Framework.

## 28.2 Un ORM ne supprime pas le besoin de comprendre SQL

Sans connaissances relationnelles, on risque :

- N+1 ;
- requêtes non sargables ;
- transactions trop longues ;
- mauvais index ;
- chargements massifs involontaires ;
- incohérences de concurrence.

## 28.3 SQLAlchemy 2.x : style conceptuel

```python
from sqlalchemy import select

stmt = (
    select(Client)
    .where(Client.actif.is_(True))
    .order_by(Client.nom)
)
```

Il est important de savoir inspecter le SQL produit et le plan côté base.

## 28.4 Quand écrire du SQL explicitement ?

- requêtes analytiques ;
- fonctions fenêtre complexes ;
- bulk operations ;
- optimisation ciblée ;
- usage d'extensions du SGBD ;
- migrations.

ORM et SQL explicite ne sont pas des approches exclusives.

---

# 29. Architecture applicative et bonnes pratiques

## 29.1 La base doit protéger ses invariants

Ne pas dépendre uniquement de :

```text
if valeur_valide:
    INSERT ...
```

Placer aussi dans le schéma :

- `NOT NULL` ;
- `UNIQUE` ;
- foreign keys ;
- `CHECK` ;
- contraintes plus avancées si le SGBD le permet.

## 29.2 Éviter les transactions distribuées inutiles

Une transaction SQL locale est robuste.

Une opération qui doit modifier :

- PostgreSQL ;
- un broker ;
- un service externe ;

ne peut pas simplement supposer qu'un `COMMIT` commun existe.

Un pattern courant est le **Transactional Outbox** :

```text
transaction SQL :
    données métier
    + événement outbox

puis :
    worker publie l'événement
```

## 29.3 Idempotence

Pour un paiement ou une API :

```text
idempotency_key UNIQUE
```

peut permettre à la base de participer à la protection contre les doubles traitements.

## 29.4 Horodatage

Pour des instants absolus côté serveur, PostgreSQL utilise souvent :

```sql
TIMESTAMPTZ
```

Stocker et comparer des instants correctement est préférable à manipuler arbitrairement des chaînes de fuseaux horaires.

## 29.5 Argent

Préférer :

```sql
NUMERIC(12, 2)
```

ou une représentation en unité minimale entière selon le domaine.

Éviter le flottant binaire pour une valeur financière exacte.

## 29.6 Suppression logique

Pattern :

```sql
deleted_at TIMESTAMPTZ NULL
```

Il apporte des avantages d'audit, mais aussi des coûts :

- toutes les requêtes doivent gérer l'état ;
- les contraintes d'unicité deviennent plus complexes ;
- les données continuent d'occuper de l'espace ;
- cela ne remplace pas une vraie politique de rétention.

## 29.7 Audit

Selon le besoin :

- colonnes `created_at`, `updated_at` ;
- table d'événements ;
- logs applicatifs ;
- audit natif/extension ;
- CDC.

Attention au RGPD et à la minimisation des données dans les journaux.

## 29.8 Base par service ?

Dans une architecture microservices, « database per service » signifie surtout **propriété des données**.

Cela ne nécessite pas nécessairement une instance physique distincte par microservice dès le premier jour.

L'enjeu est d'éviter qu'un service modifie directement les tables privées d'un autre.

---

# 30. Travaux pratiques

# TP 1 — Concevoir un modèle relationnel

Concevoir le schéma d'une bibliothèque avec :

- auteurs ;
- livres ;
- exemplaires ;
- adhérents ;
- emprunts.

Contraintes minimales :

- ISBN unique lorsqu'il est présent ;
- un exemplaire appartient à un livre ;
- un emprunt possède une date de début ;
- la date de retour ne peut pas précéder la date de début.

Livrables :

1. diagramme ;
2. tables ;
3. clés ;
4. contraintes ;
5. justification de la normalisation.

---

# TP 2 — DDL et contraintes

Créer le schéma du TP 1 en SQL.

Tester volontairement :

- doublon de clé ;
- clé étrangère inexistante ;
- `NULL` interdit ;
- contrainte `CHECK` violée.

Documenter les erreurs obtenues.

---

# TP 3 — Requêtes SQL

Écrire des requêtes permettant de produire :

- tous les livres disponibles ;
- tous les emprunts actifs ;
- le nombre d'emprunts par adhérent ;
- les cinq ouvrages les plus empruntés ;
- les adhérents sans emprunt ;
- les emprunts en retard.

Utiliser au moins :

- `JOIN` ;
- `LEFT JOIN` ;
- `GROUP BY` ;
- `HAVING` ;
- `EXISTS`.

---

# TP 4 — Normalisation

Partir de la table :

```text
commande(
    commande_id,
    date,
    client_email,
    client_nom,
    produit_1,
    quantite_1,
    produit_2,
    quantite_2,
    ...
)
```

La transformer progressivement jusqu'en 3NF.

Pour chaque transformation :

- identifier l'anomalie ;
- identifier la dépendance fonctionnelle ;
- expliquer la nouvelle table.

---

# TP 5 — Transactions concurrentes

Ouvrir deux sessions SQL.

Reproduire un scénario de réservation de stock :

```text
stock initial = 1
```

Les deux sessions tentent de réserver le dernier article.

Étudier :

- comportement naïf ;
- `SELECT ... FOR UPDATE` ;
- mise à jour conditionnelle :

```sql
UPDATE produit
SET stock = stock - 1
WHERE id = :id
  AND stock > 0;
```

Comparer les stratégies.

---

# TP 6 — Isolation

Avec deux connexions PostgreSQL, expérimenter :

- `READ COMMITTED` ;
- `REPEATABLE READ` ;
- `SERIALIZABLE`.

Observer les snapshots et provoquer au moins une erreur de sérialisation.

Écrire ensuite un mécanisme de retry côté application.

---

# TP 7 — Index et `EXPLAIN`

Créer une table contenant au moins plusieurs centaines de milliers de lignes.

Comparer :

```sql
SELECT ... WHERE client_id = ...;
```

avant et après :

```sql
CREATE INDEX ...;
```

Mesurer :

- temps ;
- type de scan ;
- buffers ;
- estimation vs lignes réelles.

Tester ensuite un index multicolonne.

---

# TP 8 — Injection SQL

Écrire une petite application volontairement vulnérable construisant une requête par concaténation.

Sans exploiter de système externe :

1. démontrer localement pourquoi la structure de la requête peut être altérée ;
2. remplacer par des paramètres ;
3. créer un `ORDER BY` dynamique avec une liste blanche ;
4. ajouter un test automatique.

---

# TP 9 — Migration sans interruption

Situation initiale :

```text
utilisateur.nom_complet
```

Nouvelle cible :

```text
utilisateur.prenom
utilisateur.nom
```

Concevoir une migration `expand -> migrate -> contract` compatible avec deux versions de l'application déployées simultanément.

---

# TP 10 — Sauvegarde et restauration

Avec PostgreSQL :

1. créer une base ;
2. insérer des données ;
3. réaliser un `pg_dump` ;
4. supprimer volontairement la base de test ;
5. restaurer dans une nouvelle base ;
6. vérifier les données et contraintes.

Écrire une procédure reproductible.

---

# TP 11 — SQLite en Python

Créer une application Python locale gérant :

- utilisateurs ;
- projets ;
- tâches.

Contraintes :

- clés étrangères actives ;
- toutes les requêtes paramétrées ;
- au moins une transaction multi-instructions ;
- tests automatisés ;
- sauvegarde du fichier `.db` hors exécution concurrente non maîtrisée.

---

# TP 12 — Audit d'une base existante

Pour une base de démonstration, produire un rapport contenant :

- schéma ;
- contraintes manquantes ;
- index inutiles ou candidats ;
- requêtes lentes ;
- privilèges excessifs ;
- stratégie backup ;
- RPO/RTO ;
- migrations ;
- points de concurrence ;
- recommandations priorisées.

---

# Projet final — Mini système de commande

Construire une application de commande avec :

```text
client
adresse
produit
commande
ligne_commande
paiement
stock
```

## Contraintes fonctionnelles

- email client unique ;
- prix strictement positif ;
- quantité commandée positive ;
- total cohérent ;
- stock jamais négatif ;
- une commande possède un historique de statut ;
- un paiement possède une clé d'idempotence.

## Exigences SQL

Le projet doit utiliser :

- PK/FK ;
- `UNIQUE` ;
- `CHECK` ;
- transactions ;
- au moins une fonction fenêtre ;
- un CTE ;
- un index multicolonne ;
- `EXPLAIN ANALYZE` ;
- une migration ;
- un rôle applicatif non-superuser.

## Concurrence

Deux commandes concurrentes ne doivent pas pouvoir vendre deux fois la dernière unité d'un produit.

La solution doit expliquer clairement la stratégie choisie :

- verrou pessimiste ;
- mise à jour atomique conditionnelle ;
- isolation sérialisable ;
- autre stratégie correctement justifiée.

## Exploitation

Documenter :

- backup ;
- restauration ;
- monitoring ;
- rotation des secrets ;
- migration ;
- test de reprise.

---

# Checklist de conception

Avant de valider un schéma :

- [ ] Chaque table représente un concept clair.
- [ ] Chaque table possède une clé adaptée.
- [ ] Les clés étrangères métier sont présentes.
- [ ] Les colonnes obligatoires utilisent `NOT NULL`.
- [ ] Les règles simples utilisent `CHECK`/`UNIQUE` lorsque pertinent.
- [ ] Les types de données correspondent au domaine.
- [ ] Les valeurs financières ne sont pas stockées dans un flottant binaire sans justification.
- [ ] Les `NULL` ont une signification claire.
- [ ] La normalisation est suffisante.
- [ ] Toute dénormalisation est mesurée et documentée.

# Checklist de requête

- [ ] Les colonnes sont explicites plutôt que `SELECT *` lorsque cela constitue une API applicative.
- [ ] Les valeurs utilisateur sont paramétrées.
- [ ] Les jointures ne multiplient pas involontairement les lignes.
- [ ] Le traitement de `NULL` est correct.
- [ ] L'ordre est explicitement défini si l'application en dépend.
- [ ] La pagination est adaptée au volume.
- [ ] Une requête lente a été mesurée avec un plan d'exécution.

# Checklist transactionnelle

- [ ] La frontière transactionnelle correspond à une opération métier.
- [ ] Les transactions restent courtes.
- [ ] Le niveau d'isolation est compris.
- [ ] Les erreurs de sérialisation/deadlock peuvent être réessayées lorsque nécessaire.
- [ ] Les accès concurrents à un même invariant ont été testés.
- [ ] Aucun appel utilisateur long n'a lieu avec des verrous ouverts.

# Checklist de production

- [ ] L'application n'utilise pas un superutilisateur.
- [ ] TLS est activé lorsque nécessaire et les certificats sont validés.
- [ ] Les secrets ne sont pas dans Git.
- [ ] Les sauvegardes sont automatiques.
- [ ] La restauration est testée.
- [ ] RPO et RTO sont définis.
- [ ] Les migrations sont versionnées.
- [ ] Les requêtes lentes sont observables.
- [ ] Les connexions sont limitées/poolées.
- [ ] La réplication n'est pas confondue avec la sauvegarde.
- [ ] L'espace disque est surveillé.
- [ ] Les versions du SGBD restent supportées et corrigées.

---

# Erreurs fréquentes

## « Une clé primaire suffit pour garantir le métier »

Faux. Une clé primaire garantit uniquement son identité propre.

Si un email doit être unique :

```sql
UNIQUE (email)
```

reste nécessaire.

## « Les contraintes ralentissent la base, je les fais dans Python »

Une application peut avoir plusieurs chemins d'écriture, plusieurs versions ou plusieurs clients. La base est le dernier point commun capable de protéger l'intégrité.

## « `SELECT *` est toujours mauvais »

Pas nécessairement dans une exploration interactive ou certaines opérations internes. Il devient surtout problématique lorsqu'il définit implicitement le contrat de données d'une application.

## « Ajouter un index rend toujours une base plus rapide »

Faux. Il accélère certains accès et ralentit les écritures/maintenance.

## « `SERIALIZABLE` évite toutes les erreurs sans effort »

Faux. Un niveau sérialisable peut volontairement annuler une transaction concurrente : l'application doit savoir gérer le retry.

## « Une réplique est une sauvegarde »

Faux. Les erreurs logiques se répliquent aussi.

## « SQLite ne gère pas les clés étrangères »

SQLite les supporte, mais leur activation et leur configuration doivent être correctement vérifiées dans l'environnement utilisé.

## « NoSQL remplace les bases relationnelles à grande échelle »

Il n'existe pas de règle générale. Les systèmes relationnels modernes fonctionnent à très grande échelle, et le choix dépend du problème.

---

# Glossaire

**ACID**
Atomicité, Cohérence, Isolation, Durabilité.

**B-tree**
Structure d'index généraliste ordonnée.

**Cardinalité**
Nombre de lignes d'un ensemble ou multiplicité d'une relation selon le contexte.

**CTE**
Common Table Expression, sous-requête nommée introduite par `WITH`.

**DDL**
Data Definition Language : définition du schéma.

**DML**
Data Manipulation Language : manipulation des données.

**Foreign key**
Contrainte reliant une valeur à une clé d'une autre relation.

**Index-only scan**
Plan pouvant répondre à une requête essentiellement depuis l'index, sous réserve de la visibilité des tuples et du SGBD.

**MVCC**
Multi-Version Concurrency Control.

**Normalisation**
Processus de structuration réduisant les dépendances et redondances problématiques.

**PITR**
Point-In-Time Recovery.

**RPO**
Recovery Point Objective.

**RTO**
Recovery Time Objective.

**Sargable**
Prédicat permettant à l'optimiseur d'utiliser efficacement une structure d'accès appropriée.

**SGBD / DBMS**
Système de gestion de base de données.

**Tuple**
Ligne dans la terminologie relationnelle.

**WAL**
Write-Ahead Log, journal utilisé notamment pour la durabilité et la réplication dans PostgreSQL.

---

# Références

## Standard SQL

- ISO/IEC 9075-1:2023 — SQL/Framework : <https://www.iso.org/standard/76583.html>

## PostgreSQL

- Documentation PostgreSQL : <https://www.postgresql.org/docs/current/>
- PostgreSQL 18.6 : <https://www.postgresql.org/docs/release/18.6/>
- Contrôle de concurrence/MVCC : <https://www.postgresql.org/docs/current/mvcc.html>
- Index : <https://www.postgresql.org/docs/current/indexes.html>
- `EXPLAIN` : <https://www.postgresql.org/docs/current/using-explain.html>
- Backup : <https://www.postgresql.org/docs/current/backup.html>

## MySQL

- MySQL 8.4 Reference Manual : <https://dev.mysql.com/doc/refman/8.4/en/>
- Modèle LTS / Innovation : <https://dev.mysql.com/doc/refman/8.4/en/mysql-releases.html>

## SQLite

- Documentation : <https://www.sqlite.org/docs.html>
- Historique des versions : <https://www.sqlite.org/changes.html>
- Foreign keys : <https://www.sqlite.org/foreignkeys.html>
- Transactions : <https://www.sqlite.org/lang_transaction.html>

---

# Conclusion

Les bases relationnelles restent un socle majeur des systèmes d'information parce qu'elles combinent :

- un modèle de données rigoureux ;
- des contraintes fortes ;
- un langage déclaratif puissant ;
- des transactions ;
- un contrôle fin de la concurrence ;
- des décennies d'optimisation et d'outillage.

La compétence essentielle n'est pas de mémoriser toutes les variantes de syntaxe SQL. Elle consiste à savoir raisonner sur :

```text
modèle
  -> contraintes
  -> requêtes
  -> transactions
  -> concurrence
  -> performances
  -> exploitation
```

Un bon schéma relationnel rend les états invalides difficiles à représenter, un bon SQL exprime clairement l'intention, et une bonne exploitation considère la sauvegarde, la sécurité et la concurrence comme des propriétés du système dès sa conception.
