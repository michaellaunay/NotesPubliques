---
schema_version: 1
uid: "01M02JG1VDDT6M215A2BNWDQGZ"
titre: "LibGen"
aliases:
  - "Library Genesis"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - droit-d-auteur
  - donnees-ouvertes
  - sparql
  - bibliotheques
resume: "Fiche sur Library Genesis (LibGen) et son statut en 2026 — procès des éditeurs, saisies de domaines, rôle dans les litiges sur l'entraînement des IA — puis reformulation de l'exercice de 2023 « part d'œuvres françaises sous droits » en exercice de données ouvertes : règles du domaine public en France, requêtes SPARQL vérifiées sur data.bnf.fr et Wikidata, script Python."
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
# LibGen

> [!abstract] Objectif
> Situer Library Genesis en 2026 — ce qu'il fut, ce que les tribunaux en ont fait, pourquoi il est devenu central dans les litiges sur l'entraînement des modèles d'IA — et transformer la question posée en 2023 (« quelle part des œuvres françaises d'un tel catalogue est encore sous droits ? ») en un exercice réalisable et licite avec les **données ouvertes** de la BnF et de Wikidata.

> [!info] État de la fiche
> Réécrite le 29 août 2026. La version de 2023 décrivait un site en pleine activité et proposait deux scripts : l'un interrogeait une API `libgen.rs` qui n'existe plus (domaine saisi), l'autre une propriété `isPublicDomain` que data.bnf.fr n'a jamais eue. Les requêtes ci-dessous ont été vérifiées syntaxiquement avec rdflib ; le script Python a été exécuté sur des cas de test.

Voir aussi : [[Droits d'auteur]], [[National Emergency Library]], [[SPARQL et RDF]], [[Taille des bibliothèques]], [[Python]].

# 1. Library Genesis : de 2008 à 2026

**Library Genesis** est une bibliothèque clandestine née en 2008 dans la mouvance russe des bibliothèques numériques, qui a mis en accès libre des millions de livres et d'articles scientifiques, sans autorisation des ayants droit. Sa base de métadonnées (auteur, titre, ISBN, langue, sujet) était publiée en clair et alimentait de nombreux miroirs.

Chronologie récente :

- **septembre 2023** : cinq éditeurs de manuels (Cengage, Macmillan Learning, McGraw Hill, Elsevier, Pearson) assignent LibGen devant un tribunal fédéral de New York ;
- **septembre 2024** : jugement par défaut, 30 millions de dollars de dommages et injonction permanente ; les registraires et services techniques sont enjoints de couper les domaines ;
- **décembre 2024** : saisies de noms de domaine ; le principal développeur est signalé inactif depuis août 2024, les miroirs se fragmentent ;
- **2025-2026** : le projet subsiste sous forme de copies dispersées et de sites successeurs ; en France, les fournisseurs d'accès bloquent ces sites sur décision de justice, comme Sci-Hub et Z-Library.

Le nom est resté au cœur de l'actualité pour une autre raison : les procédures engagées par des auteurs contre les éditeurs de modèles de langage ont établi que des corpus issus de LibGen avaient servi à l'entraînement. Dans *Kadrey v. Meta* (2025), les pièces ont montré l'usage de ces fichiers ; dans *Bartz v. Anthropic* (2025), le juge a distingué l'entraînement, jugé relever du *fair use*, de la constitution d'une bibliothèque de livres piratés, jugée illicite — l'affaire s'est conclue par un règlement de 1,5 milliard de dollars. Pour un cours de [[Droits d'auteur]], LibGen est donc passé du statut de « site pirate » à celui de cas d'école sur la provenance des données d'entraînement.

# 2. Ce que la question de 2023 supposait, et pourquoi la reformuler

La note d'origine voulait mesurer la part d'œuvres françaises « illégales » dans LibGen en croisant son catalogue avec data.bnf.fr. Trois obstacles :

1. le catalogue n'est plus accessible par une API stable, et le récupérer suppose de fréquenter des miroirs bloqués ou éphémères ;
2. le statut « domaine public » n'existe pas comme donnée toute faite : il se **calcule** à partir de la date de décès des auteurs et des règles de durée ;
3. la question n'a pas besoin de LibGen : n'importe quel lot d'ISBN (catalogue d'une bibliothèque, liste d'un cours, échantillon du dépôt légal) permet le même calcul et le même raisonnement.

L'exercice devient donc : **pour une liste d'ISBN, déterminer la part des œuvres dont les droits patrimoniaux sont éteints**, à partir de données ouvertes.

# 3. Les règles du domaine public en France

- Les droits patrimoniaux durent **70 ans après la mort de l'auteur** (article L123-1 du Code de la propriété intellectuelle), décomptés à partir du 1er janvier de l'année suivant le décès ; œuvre de collaboration : à partir du décès du dernier coauteur.
- **Prorogations de guerre** : pour les œuvres publiées avant 1948, des extensions liées aux deux guerres mondiales peuvent subsister ; depuis l'arrêt de la Cour de cassation de 2007, elles ne s'appliquent que si le calcul « 50 ans + prorogations » dépasse 70 ans, ce qui concerne surtout les auteurs **morts pour la France** (30 ans de plus).
- Le **droit moral** est perpétuel : une œuvre du domaine public reste attribuée à son auteur et protégée contre la dénaturation.
- Une traduction, une édition critique ou une préface ont leurs propres auteurs et leurs propres durées : un ISBN récent d'un texte ancien peut être sous droits par sa traduction.

En première approximation : **domaine public si `année_courante > année_décès + 70`**, avec un marquage « à vérifier » pour les auteurs morts entre 1900 et 1948 et pour les éditions récentes d'œuvres anciennes.

# 4. Trouver la date de décès d'un auteur à partir d'un ISBN

## 4.1 data.bnf.fr

data.bnf.fr expose le catalogue de la BnF en RDF, avec un point d'accès SPARQL (voir api.bnf.fr pour son adresse et ses conditions). Une manifestation (édition) porte son ISBN ; elle renvoie à l'œuvre, qui renvoie à ses auteurs, dont les dates sont publiées.

```sparql
PREFIX bnf-onto: <http://data.bnf.fr/ontology/bnf-onto/>
PREFIX rdarelationships: <http://rdvocab.info/RDARelationshipsWEMI/>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX bio: <http://vocab.org/bio/0.1/>

SELECT ?titre ?auteur ?naissance ?deces WHERE {
  ?edition bnf-onto:isbn ?isbn ;
           rdarelationships:workManifested ?oeuvre .
  FILTER (REPLACE(?isbn, "-", "") = "9782070457665")
  ?oeuvre dcterms:title ?titre ;
          dcterms:creator ?personne .
  ?personne foaf:name ?auteur .
  OPTIONAL { ?personne bio:birth ?naissance }
  OPTIONAL { ?personne bio:death ?deces }
}
```

Les ISBN sont stockés tels qu'imprimés (souvent avec tirets), d'où la normalisation par `REPLACE`. Les dates `bio:death` sont des chaînes (année ou date complète) : les convertir en entier avant tout calcul.

## 4.2 Wikidata

Wikidata couvre moins finement les éditions françaises mais son point d'accès (<https://query.wikidata.org/>) est stable et documenté. Propriétés utiles : ISBN-13 (`P212`), auteur (`P50`), date de décès (`P570`).

```sparql
SELECT ?edition ?editionLabel ?auteurLabel ?deces WHERE {
  ?edition wdt:P212 "978-2-07-045766-5" .
  ?edition wdt:P629 ?oeuvre .
  ?oeuvre wdt:P50 ?auteur .
  OPTIONAL { ?auteur wdt:P570 ?deces }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "fr,en" . }
}
```

Sur Wikidata, l'ISBN-13 est saisi avec tirets ; l'édition (`P629`) renvoie à l'œuvre, qui porte l'auteur.

# 5. Script : statut d'une liste d'ISBN

```python
"""Estime, pour une liste d'ISBN, la part des œuvres dont les droits patrimoniaux sont éteints en France.

Les dates de décès sont obtenues par SPARQL (data.bnf.fr ou Wikidata) ; la règle appliquée est
« 70 ans après le décès », avec un marquage « à vérifier » pour les cas ambigus.
"""
from __future__ import annotations

import re
from dataclasses import dataclass
from datetime import date

import requests

ENDPOINT_BNF = "https://data.bnf.fr/sparql"
DUREE = 70

REQUETE_BNF = """
PREFIX bnf-onto: <http://data.bnf.fr/ontology/bnf-onto/>
PREFIX rdarelationships: <http://rdvocab.info/RDARelationshipsWEMI/>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX bio: <http://vocab.org/bio/0.1/>
SELECT ?titre ?auteur ?deces WHERE {
  ?edition bnf-onto:isbn ?isbn ; rdarelationships:workManifested ?oeuvre .
  FILTER (REPLACE(?isbn, "-", "") = "%s")
  ?oeuvre dcterms:title ?titre ; dcterms:creator ?personne .
  ?personne foaf:name ?auteur .
  OPTIONAL { ?personne bio:death ?deces }
}
"""


@dataclass
class Statut:
    isbn: str
    titre: str | None
    auteurs: list[str]
    deces: list[int | None]
    verdict: str          # "domaine public", "sous droits", "inconnu"
    a_verifier: bool


def annee(valeur: str | None) -> int | None:
    """Extrait une année d'une date data.bnf.fr ('1961', '1961-07-02', '19..')."""
    if not valeur:
        return None
    m = re.match(r"(\d{4})", valeur)
    return int(m.group(1)) if m else None


def verdict(deces: list[int | None], aujourd_hui: date | None = None) -> tuple[str, bool]:
    aujourd_hui = aujourd_hui or date.today()
    if not deces or any(d is None for d in deces):
        return "inconnu", True
    dernier = max(deces)                              # œuvre de collaboration : dernier coauteur
    libre = aujourd_hui.year > dernier + DUREE
    ambigu = 1900 <= dernier <= 1948                  # prorogations de guerre possibles
    return ("domaine public" if libre else "sous droits"), ambigu


def interroger_bnf(isbn: str) -> Statut:
    isbn = isbn.replace("-", "")
    reponse = requests.get(
        ENDPOINT_BNF,
        params={"query": REQUETE_BNF % isbn, "format": "application/sparql-results+json"},
        headers={"Accept": "application/sparql-results+json"},
        timeout=30,
    )
    reponse.raise_for_status()
    lignes = reponse.json()["results"]["bindings"]
    if not lignes:
        return Statut(isbn, None, [], [], "inconnu", True)
    titre = lignes[0]["titre"]["value"]
    auteurs = sorted({l["auteur"]["value"] for l in lignes})
    deces = [annee(l.get("deces", {}).get("value")) for l in lignes]
    v, ambigu = verdict(deces)
    return Statut(isbn, titre, auteurs, deces, v, ambigu)


def statistiques(isbns: list[str]) -> dict[str, int]:
    compte = {"domaine public": 0, "sous droits": 0, "inconnu": 0, "a_verifier": 0}
    for isbn in isbns:
        s = interroger_bnf(isbn)
        compte[s.verdict] += 1
        compte["a_verifier"] += int(s.a_verifier)
        print(f"{s.isbn}  {s.verdict:15s}  {', '.join(s.auteurs) or '?'}  {s.titre or ''}")
    return compte


if __name__ == "__main__":
    print(statistiques(["978-2-07-045766-5", "978-2-07-036002-4"]))
```

La fonction `verdict` est la seule logique métier ; elle se teste sans réseau. Les appels SPARQL sont à espacer (une requête par seconde suffit) et à mettre en cache : un ISBN interrogé une fois n'a pas à l'être deux.

# 6. Ce que l'exercice apprend

- La différence entre **accès** (un fichier est disponible) et **droit** (l'œuvre est libre) ; la disponibilité d'un livre sur un site ne dit rien de son statut.
- Le domaine public est une **date calculée**, pas un attribut : la qualité des données d'autorité (dates de décès, homonymes, œuvres collectives) fait toute la précision du résultat.
- Les limites des catalogues : une édition récente d'un classique est sous droits par sa traduction ou son appareil critique ; les auteurs sans notice d'autorité restent « inconnus ».
- Le même raisonnement s'applique aux **corpus d'entraînement** des modèles d'IA, ce qui explique pourquoi les litiges de 2025 se sont joués sur la provenance des livres plutôt que sur l'entraînement lui-même.

# 7. Sources

- data.bnf.fr : <https://data.bnf.fr/> ; API et point d'accès SPARQL : <https://api.bnf.fr/>
- Wikidata Query Service : <https://query.wikidata.org/>
- Code de la propriété intellectuelle, article L123-1 (durée des droits) : <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000006278936>
- Cour de cassation, 27 février 2007, sur les prorogations de guerre (résumé sur le site de la Cour)
- Library Genesis, article de Wikipédia (chronologie judiciaire) : <https://fr.wikipedia.org/wiki/Library_Genesis>
- *Bartz v. Anthropic* et *Kadrey v. Meta* : décisions de 2025 du district nord de Californie ; couverture par la presse spécialisée
