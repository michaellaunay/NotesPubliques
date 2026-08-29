---
schema_version: 1
uid: "01M02JG1VGEBQ5WM5P4BK3V5RD"
titre: "SPARQL et RDF"
aliases:
  - "RDF"
  - "SPARQL"
  - "Web sémantique"
  - "Linked Data"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - web-semantique
  - rdf
  - sparql
  - donnees-liees
  - python
resume: "Fiche sur RDF et SPARQL en 2026 : modèle de triplets, IRI, littéraux, vocabulaires (RDFS, FOAF, Dublin Core, schema.org), syntaxes Turtle, JSON-LD et N-Triples, formes de requêtes SELECT, ASK, CONSTRUCT, UPDATE avec exemples exécutés en rdflib, validation SHACL, interrogation de Wikidata et data.bnf.fr, entrepôts (Jena/Fuseki, Oxigraph, GraphDB, Virtuoso), état de RDF 1.2 et SPARQL 1.2 au W3C, et liens avec Solid."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-03-10
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# SPARQL et RDF

> [!abstract] Objectif
> Comprendre le modèle de données **RDF** — des graphes de triplets sujet-prédicat-objet identifiés par des IRI — et savoir l'interroger avec **SPARQL** : écrire et exécuter des requêtes sur un graphe local avec rdflib, valider des données avec SHACL, interroger des points d'accès publics (Wikidata, data.bnf.fr), choisir un entrepôt de triplets, et situer les évolutions RDF 1.2 et SPARQL 1.2 en cours au W3C.

> [!info] État de la fiche
> Réécrite le 29 août 2026 : la version de 2023 était une définition sans exemple. Les requêtes des chapitres 3 et 4 ont été exécutées avec rdflib 7.6 et pyshacl 0.40 ; l'état des spécifications a été vérifié sur w3.org (RDF 1.2 Concepts en *Candidate Recommendation* depuis avril 2026, SPARQL 1.2 en *Working Draft*).

Voir aussi : [[SOLID|Solid (Social Linked Data)]] (pods et données liées), [[LibGen]] (requêtes data.bnf.fr et Wikidata sur le domaine public), [[Python]], [[HTTP]].

# 1. RDF : des graphes de triplets

**RDF** (*Resource Description Framework*, W3C) représente toute information par des **triplets** : un sujet, un prédicat, un objet.

```text
ex:ada    foaf:name      "Ada Lovelace" .
ex:notes  dct:creator    ex:ada .
ex:notes  dct:title      "Notes on the Analytical Engine"@en .
```

- Sujets et prédicats sont des **IRI** (identifiants globaux, souvent des URL déréférençables) ; les objets sont des IRI ou des **littéraux** typés (`"1843"^^xsd:gYear`) ou étiquetés par une langue (`"…"@en`) ; les **nœuds anonymes** (*blank nodes*) désignent des ressources sans identifiant.
- L'ensemble des triplets forme un **graphe** ; plusieurs graphes nommés forment un **jeu de données** (*dataset*).
- Les prédicats viennent de **vocabulaires** partagés : **RDFS** (classes, sous-classes, domaines), **OWL** (ontologies, raisonnement), **FOAF** (personnes), **Dublin Core** (`dct:title`, `dct:creator`), **schema.org** (le vocabulaire des moteurs de recherche), **SKOS** (thésaurus). Réutiliser un vocabulaire existant est la première règle des données liées.

Ce modèle sert les **données liées** (*Linked Data*) : identifier chaque chose par une IRI, la décrire en RDF, lier vers d'autres jeux de données. Wikidata, data.bnf.fr, DBpedia, les référentiels scientifiques et les pods [[SOLID|Solid]] fonctionnent ainsi ; schema.org en JSON-LD dans les pages web alimente les moteurs de recherche.

## 1.1 Syntaxes

| Syntaxe | Usage |
|---|---|
| **Turtle** (`.ttl`) | lisible, préfixes, la syntaxe pour écrire et enseigner |
| **N-Triples** / **N-Quads** | un triplet par ligne, sans préfixe ; échanges massifs, différences Git |
| **JSON-LD** | RDF en JSON, intégrable dans HTML ; le format de schema.org |
| **RDF/XML** | historique, encore rencontré dans les exports |
| **TriG** | Turtle avec graphes nommés |

# 2. SPARQL : interroger les graphes

**SPARQL** (*SPARQL Protocol and RDF Query Language*) décrit des **motifs de triplets** avec des variables ; le moteur cherche toutes les affectations qui les satisfont. Quatre formes de requête :

- `SELECT` : un tableau de résultats (variables liées) ;
- `ASK` : vrai ou faux ;
- `CONSTRUCT` : un nouveau graphe RDF construit à partir des résultats ;
- `DESCRIBE` : les triplets décrivant une ressource.

Clauses courantes : `WHERE` (motifs), `OPTIONAL` (motif facultatif, comme une jointure externe), `FILTER` (conditions), `UNION`, `MINUS`, `GROUP BY` avec `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `ORDER BY`, `LIMIT`/`OFFSET`, chemins de propriétés (`foaf:knows+`, `rdfs:subClassOf*`), `BIND`, `VALUES`, et `SERVICE` pour interroger un autre point d'accès dans la même requête (requête **fédérée**). **SPARQL Update** (`INSERT DATA`, `DELETE`, `LOAD`) modifie les graphes ; le **protocole SPARQL** transporte requêtes et résultats en HTTP (résultats en JSON, XML, CSV, TSV).

# 3. Pratique avec rdflib

```bash
python -m pip install rdflib pyshacl
```

## 3.1 Charger un graphe

```python
from rdflib import Graph

TURTLE = """
@prefix ex:   <http://exemple.org/biblio/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix dct:  <http://purl.org/dc/terms/> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .

ex:ada   a foaf:Person ; foaf:name "Ada Lovelace" ; ex:naissance "1815-12-10"^^xsd:date ; ex:deces "1852-11-27"^^xsd:date .
ex:grace a foaf:Person ; foaf:name "Grace Hopper" ; ex:naissance "1906-12-09"^^xsd:date ; ex:deces "1992-01-01"^^xsd:date .
ex:linus a foaf:Person ; foaf:name "Linus Torvalds" ; ex:naissance "1969-12-28"^^xsd:date .

ex:notes a ex:Livre ; dct:title "Notes on the Analytical Engine"@en ; dct:creator ex:ada ; dct:date "1843"^^xsd:gYear .
ex:a0    a ex:Livre ; dct:title "The A-0 compiler"@en ; dct:creator ex:grace ; dct:date "1952"^^xsd:gYear .
ex:jfp   a ex:Livre ; dct:title "Just for Fun"@en ; dct:creator ex:linus ; dct:date "2001"^^xsd:gYear .

ex:ada foaf:knows ex:grace .
"""

g = Graph()
g.parse(data=TURTLE, format="turtle")
print(len(g), "triplets")          # 24 triplets
```

`g.parse("fichier.ttl")` ou `g.parse("https://…")` chargent un fichier ou une ressource en ligne ; le format se déduit de l'extension ou du type MIME.

## 3.2 SELECT, OPTIONAL, FILTER, ORDER BY

```python
PREFIXES = """
PREFIX ex:   <http://exemple.org/biblio/>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX dct:  <http://purl.org/dc/terms/>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>
"""

livres_anciens = PREFIXES + """
SELECT ?nom ?titre ?annee ?deces WHERE {
  ?livre a ex:Livre ; dct:title ?titre ; dct:creator ?auteur ; dct:date ?annee .
  ?auteur foaf:name ?nom .
  OPTIONAL { ?auteur ex:deces ?deces }
  FILTER (xsd:integer(STR(?annee)) < 2000)
}
ORDER BY ?annee
"""
for ligne in g.query(livres_anciens):
    print(ligne.nom, "|", ligne.titre, "|", ligne.annee, "|", ligne.deces)
# Ada Lovelace | Notes on the Analytical Engine | 1843 | 1852-11-27
# Grace Hopper | The A-0 compiler | 1952 | 1992-01-01
```

`OPTIONAL` conserve les auteurs sans date de décès (la variable reste non liée). Le `FILTER` convertit l'année en entier : la comparaison directe de littéraux `xsd:gYear` n'est pas évaluée par rdflib (la requête renvoie zéro ligne sans erreur), un piège classique de portabilité entre moteurs.

## 3.3 Agrégation, ASK, CONSTRUCT

```python
par_auteur = PREFIXES + """
SELECT ?nom (COUNT(?livre) AS ?nb) WHERE {
  ?livre dct:creator ?auteur .
  ?auteur foaf:name ?nom .
}
GROUP BY ?nom
ORDER BY DESC(?nb)
"""
for ligne in g.query(par_auteur):
    print(ligne.nom, ligne.nb)

connaissance = g.query(PREFIXES + "ASK { ex:ada foaf:knows ex:grace }")
print(bool(connaissance))          # True

derive = g.query(PREFIXES + """
CONSTRUCT { ?auteur ex:aEcrit ?titre }
WHERE { ?livre dct:creator ?auteur ; dct:title ?titre }
""").graph
print(derive.serialize(format="turtle"))
```

`CONSTRUCT` produit un graphe : c'est le moyen de **transformer** des données RDF (changer de vocabulaire, dériver des relations) et de les enregistrer.

## 3.4 Mettre à jour et sérialiser

```python
g.update(PREFIXES + """
INSERT DATA { ex:notes dct:language "en" }
""")

g.serialize(destination="biblio.ttl", format="turtle")
print(g.serialize(format="json-ld", indent=2)[:200])
print(g.serialize(format="nt").splitlines()[0])
```

## 3.5 Valider avec SHACL

**SHACL** (*Shapes Constraint Language*, W3C 2017) décrit les contraintes qu'un graphe doit respecter — l'équivalent d'un schéma, absent du modèle RDF lui-même.

```python
from pyshacl import validate

FORMES = """
@prefix sh:  <http://www.w3.org/ns/shacl#> .
@prefix ex:  <http://exemple.org/biblio/> .
@prefix dct: <http://purl.org/dc/terms/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

ex:LivreShape a sh:NodeShape ;
  sh:targetClass ex:Livre ;
  sh:property [ sh:path dct:title ;   sh:minCount 1 ; sh:maxCount 1 ] ;
  sh:property [ sh:path dct:creator ; sh:minCount 1 ; sh:class foaf:Person ] .
"""
conforme, _, rapport = validate(g, shacl_graph=Graph().parse(data=FORMES, format="turtle"))
print(conforme)                    # False si un ex:Livre n'a ni titre ni créateur
print(rapport)
```

Sur un livre ajouté sans titre ni auteur, le rapport signale `Less than 1 values on ex:mystere->dct:title` et `…->dct:creator`.

# 4. Interroger des points d'accès publics

Le protocole SPARQL est du HTTP : `GET` ou `POST` sur l'URL du point d'accès avec le paramètre `query`, en demandant `application/sparql-results+json`. La bibliothèque **SPARQLWrapper** enveloppe cela ; `requests` suffit aussi.

```python
from SPARQLWrapper import SPARQLWrapper, JSON

sparql = SPARQLWrapper("https://query.wikidata.org/sparql", agent="cours-rdf/1.0 (contact@example.org)")
sparql.setQuery("""
SELECT ?langage ?langageLabel ?annee WHERE {
  ?langage wdt:P31 wd:Q9143 ;                # instance de : langage de programmation
           wdt:P571 ?creation .
  BIND (YEAR(?creation) AS ?annee)
  FILTER (?annee < 1960)
  SERVICE wikibase:label { bd:serviceParam wikibase:language "fr,en" . }
}
ORDER BY ?annee
""")
sparql.setReturnFormat(JSON)
for ligne in sparql.query().convert()["results"]["bindings"]:
    print(ligne["annee"]["value"], ligne["langageLabel"]["value"])
```

Bonnes pratiques d'usage : un `User-Agent` identifiable, des requêtes bornées (`LIMIT`), un cache local, et le respect des délais d'attente du service ; les points d'accès publics limitent la durée d'exécution (60 s sur Wikidata). Les préfixes `wd:`, `wdt:`, `wikibase:` et `bd:` sont prédéclarés sur Wikidata ; data.bnf.fr expose ses propres vocabulaires (voir l'exemple dans [[LibGen]]).

# 5. Entrepôts de triplets

| Entrepôt | Langage | Licence | Usage |
|---|---|---|---|
| **rdflib** (en mémoire ou Berkeley DB) | Python | BSD | scripts, enseignement, jusqu'à quelques millions de triplets |
| **Oxigraph** (`pyoxigraph`) | Rust, liaison Python | Apache/MIT | serveur léger et rapide, embarquable, SPARQL 1.1 complet, RDF 1.2 en cours |
| **Apache Jena / Fuseki** | Java | Apache 2.0 | référence libre, serveur SPARQL complet, raisonnement, TDB2 |
| **GraphDB**, **Virtuoso**, **Stardog**, **Blazegraph** | divers | libre ou commercial | production, très grands volumes ; Blazegraph propulse encore Wikidata, en cours de remplacement |

# 6. RDF 1.2 et SPARQL 1.2 : où en est le W3C

Le groupe de travail RDF & SPARQL (issu du groupe RDF-star) révise l'ensemble des recommandations de 2014. État en août 2026 :

- **RDF 1.2 Concepts** et **RDF 1.2 Semantics** : *Candidate Recommendation* depuis le 7 avril 2026, implémentations demandées (Jena et Oxigraph en cours) ; les syntaxes (Turtle 1.2, N-Triples 1.2, JSON-LD) suivent en *Working Draft* ;
- **SPARQL 1.2** : douze documents (Query, Update, Protocol, Federated Query, Service Description, formats de résultats…) en *Working Draft*, Query mis à jour en avril 2026.

Ce qui change : les **termes triplets** (*triple terms*), qui permettent de faire d'un triplet l'objet d'un autre triplet pour exprimer provenance, dates de validité ou degré de confiance sans les lourdes réifications d'autrefois (c'est l'héritage de RDF-star) ; les chaînes **avec direction d'écriture** (`"…"@ar--rtl`) ; et un processus de « standard vivant » pour les évolutions futures. Toute donnée RDF 1.1 reste valide en 1.2.

# 7. Aide-mémoire

| Besoin | Écriture |
|---|---|
| Charger | `Graph().parse("f.ttl")` ; `parse(data=…, format="turtle")` |
| Interroger | `g.query(requete)` ; résultats itérables, attributs nommés |
| Modifier | `g.update("INSERT DATA { … }")` ; `g.add((s, p, o))` |
| Sérialiser | `g.serialize(format="turtle" \| "json-ld" \| "nt")` |
| Valider | `pyshacl.validate(g, shacl_graph=formes)` |
| Point d'accès distant | `SPARQLWrapper(url)` ou `requests.get(url, params={"query": …}, headers={"Accept": "application/sparql-results+json"})` |
| Fédérer | `SERVICE <https://…/sparql> { … }` dans le `WHERE` |
| Chemin de propriétés | `?a foaf:knows+ ?b`, `?c rdfs:subClassOf* ex:Chose` |

# 8. Sources

- RDF 1.2 Concepts (Candidate Recommendation, avril 2026) : <https://www.w3.org/TR/rdf12-concepts/> ; RDF 1.1 Primer : <https://www.w3.org/TR/rdf11-primer/>
- SPARQL 1.2 Query Language (Working Draft) : <https://www.w3.org/TR/sparql12-query/> ; SPARQL 1.1 Query (Recommendation 2013) : <https://www.w3.org/TR/sparql11-query/>
- Groupe de travail RDF & SPARQL : <https://www.w3.org/groups/wg/rdf-star/>
- SHACL : <https://www.w3.org/TR/shacl/>
- rdflib : <https://rdflib.readthedocs.io/> ; pyshacl : <https://github.com/RDFLib/pySHACL> ; SPARQLWrapper : <https://sparqlwrapper.readthedocs.io/>
- Oxigraph : <https://github.com/oxigraph/oxigraph> ; Apache Jena : <https://jena.apache.org/>
- Wikidata Query Service : <https://query.wikidata.org/> ; data.bnf.fr : <https://data.bnf.fr/>
