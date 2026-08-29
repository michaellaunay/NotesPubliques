---
schema_version: 1
uid: "01M02JG1VADWJNVTA96XN6WX5J"
titre: "AirFlow"
aliases:
  - "Apache Airflow"
  - "Airflow"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - orchestration
  - donnees
  - airflow
resume: "Fiche sur Apache Airflow 3 (2026) : ce qu'orchestre Airflow et quand l'employer, ce qui a changé avec la version 3 (Task SDK, API d'exécution, interface React, versionnage des Dags, actifs, ordonnancement événementiel), concepts, architecture, installation, exemple de Dag TaskFlow exécuté, bonnes pratiques et migration depuis Airflow 2."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-09-06
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Apache Airflow

> [!abstract] Objectif
> Comprendre ce qu'est Apache Airflow — un orchestrateur de traitements décrits en Python sous forme de graphes de tâches — et savoir l'utiliser dans sa version 3 : concepts (Dag, tâche, exécution, actif), architecture, installation d'un environnement de développement, écriture et test d'un Dag avec le Task SDK, ordonnancement par le temps ou par les données, bonnes pratiques et points de migration depuis Airflow 2.

> [!info] État de la fiche
> Vérifiée le 29 août 2026 avec **Airflow 3.3.1** installé dans un environnement virtuel ; l'exemple de Dag du chapitre 6 a été exécuté avec `airflow dags test`. La fiche de 2023 décrivait Airflow 2 ; Airflow 3.0 (22 avril 2025) a changé l'interface, l'architecture et l'API d'écriture des Dags.

Voir aussi : [[Python]], [[Docker]], [[Data Mining en Python]], [[RAG]], [[Initialisation système et des services]].

# 1. Ce qu'est Airflow, et quand l'employer

Apache Airflow orchestre des **workflows** : des ensembles de tâches liées par des dépendances, planifiées dans le temps ou déclenchées par des événements, exécutées avec reprises sur erreur, journalisées et visualisées. Né chez Airbnb en 2014, projet Apache de premier niveau depuis 2019, il est devenu le standard des chaînes de données (extraction, transformation, chargement, entraînement de modèles, indexation d'un [[RAG]]…).

Un workflow est un **Dag** (graphe orienté acyclique) écrit en Python : le code est versionné, relu, testé comme n'importe quel programme. Airflow n'exécute pas de calcul lourd lui-même ; il **déclenche** des tâches — un script, une requête SQL, un conteneur, un job Spark, un appel d'API — et suit leur état.

Quand un simple `cron` suffit (une commande, une machine, pas de dépendances), Airflow est disproportionné. Il devient utile dès qu'il faut : des dépendances entre étapes, des reprises et des alertes, un historique consultable, plusieurs machines, ou réagir à l'arrivée d'une donnée. Les alternatives de 2026 sont Dagster et Prefect (orientés « actifs » et développeur), n8n pour l'automatisation sans code, et les orchestrateurs des clouds.

# 2. Ce qui a changé avec Airflow 3

| Airflow 2 | Airflow 3 (depuis avril 2025) |
|---|---|
| imports depuis `airflow.models`, `airflow.decorators`… | **Task SDK** : `from airflow.sdk import dag, task, DAG, Asset` — interface stable pour les auteurs de Dags |
| les tâches lisent la base de métadonnées | **API d'exécution** : les tâches ne touchent plus la base ; elles parlent au serveur d'API, ce qui permet de les exécuter à distance, isolées, et bientôt dans d'autres langages (SDK Java et Go expérimentaux en 3.3) |
| `airflow webserver` (Flask) | `airflow api-server` : interface **React** et API REST v2 sur FastAPI |
| un Dag n'a qu'une définition | **versionnage des Dags** : chaque changement de structure est conservé et consultable pour chaque exécution |
| `schedule_interval`, datasets | `schedule=` unique (cron, `timedelta`, timetable, actifs) ; **actifs** (`Asset`, `@asset`) et ordonnancement **événementiel** (files de messages) |
| `execution_date` | `logical_date`, et Dags sans intervalle de données (plusieurs exécutions identiques possibles, utile en apprentissage automatique) |
| SubDAGs | supprimés ; utiliser TaskGroups |
| exécuteur séquentiel par défaut | `LocalExecutor` par défaut, exécuteur **Edge** pour des agents distants |
| un seul paquet | `apache-airflow-core`, `apache-airflow-task-sdk`, fournisseurs (`apache-airflow-providers-*`) versionnés séparément |

Depuis : 3.1 (fin 2025) a ajouté les tâches avec validation humaine et les greffons d'interface React ; 3.2 (2026) le **multi-équipes** dans un même déploiement et le partitionnement des actifs ; 3.3 (juillet 2026) un **magasin d'état** pour les tâches et les actifs, des politiques de reprise personnalisables, le décorateur `@result` et le SDK multi-langages.

# 3. Concepts

- **Dag** : le workflow ; un identifiant, un ordonnancement, une date de début, des tâches et leurs dépendances.
- **Tâche** et **opérateur** : une tâche est l'instanciation d'un opérateur (Python, Bash, SQL, Docker, Kubernetes, HTTP…) ; avec l'API **TaskFlow**, une fonction Python décorée par `@task` est une tâche, et ses valeurs de retour circulent entre tâches par **XCom**.
- **Exécution de Dag** (*Dag run*) et **instance de tâche** : une exécution à une `logical_date` donnée, avec un état par tâche (en attente, en cours, succès, échec, reprise…).
- **Actif** (*asset*) : une donnée produite ou consommée (fichier, table, objet) ; un Dag peut être déclenché par la mise à jour d'un actif plutôt que par l'horloge.
- **Connexions** et **variables** : identifiants et paramètres stockés hors du code, éventuellement dans un coffre externe.
- **Exécuteur** : où tournent les tâches — `LocalExecutor` (même machine), Celery, Kubernetes, Edge.
- **Fournisseurs** (*providers*) : paquets d'opérateurs et de connexions pour chaque système externe.

# 4. Architecture

```text
 auteur ──► fichiers de Dags (dossier ou bundle Git) ──► dag-processor ──► base de métadonnées
                                                                                 ▲
 api-server (interface React, API REST v2, API d'exécution) ◄── scheduler ───────┘
        ▲                                                        │
        │ Task SDK / API d'exécution                             ▼
     tâches (workers locaux, Celery, Kubernetes, agents Edge) ◄── exécuteur
 triggerer : attentes asynchrones (capteurs, événements) sans occuper un worker
```

Cinq services (`api-server`, `scheduler`, `dag-processor`, `triggerer`, workers selon l'exécuteur) autour d'une base PostgreSQL en production — SQLite suffit pour apprendre. `airflow standalone` lance tout en un seul processus.

# 5. Installer un environnement de développement

Airflow s'installe avec le fichier de **contraintes** de sa version, qui fige les dépendances testées (Python 3.10 à 3.13 pour la série 3.3) :

```bash
python -m venv .venv && . .venv/bin/activate
AIRFLOW_VERSION=3.3.1
PYTHON_VERSION="$(python -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')"
pip install "apache-airflow==${AIRFLOW_VERSION}" \
  --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

export AIRFLOW_HOME=~/airflow           # configuration, base SQLite, journaux, dossier dags/
airflow db migrate                        # crée ou met à niveau la base
airflow standalone                        # api-server + scheduler + dag-processor + triggerer ; affiche le mot de passe admin
```

L'interface est alors sur <http://localhost:8080>. Pour un poste ou une équipe, le **Docker Compose** officiel et le **chart Helm** (Kubernetes) sont les voies documentées ; les services gérés (Astronomer, Amazon MWAA, Google Cloud Composer) évitent d'exploiter la plateforme soi-même.

# 6. Écrire et tester un Dag

Exemple exécuté avec Airflow 3.3.1 : trois tâches TaskFlow, un actif produit, planification quotidienne.

```python
"""Exemple Airflow 3 : extraire, transformer, publier — écrit avec le Task SDK."""
from __future__ import annotations

import json
from datetime import datetime
from pathlib import Path

from airflow.sdk import Asset, dag, task

RAPPORT = Asset("file:///tmp/rapports/ventes.json")   # l'actif produit ; d'autres Dags peuvent s'y abonner


@dag(
    dag_id="rapport_quotidien",
    schedule="@daily",                                   # cron, timedelta, Asset ou None
    start_date=datetime(2026, 1, 1),
    catchup=False,
    tags=["cours", "exemple"],
    default_args={"retries": 2},
)
def rapport_quotidien():

    @task
    def extraire() -> list[dict]:
        return [{"produit": "capteur", "qte": 12}, {"produit": "passerelle", "qte": 3}]

    @task
    def transformer(lignes: list[dict]) -> dict:
        return {"total": sum(l["qte"] for l in lignes), "produits": len(lignes)}

    @task(outlets=[RAPPORT])                                # déclare que la tâche met à jour l'actif
    def publier(resume: dict) -> None:
        cible = Path("/tmp/rapports/ventes.json")
        cible.parent.mkdir(parents=True, exist_ok=True)
        cible.write_text(json.dumps(resume, ensure_ascii=False))

    publier(transformer(extraire()))


rapport_quotidien()
```

Le fichier se dépose dans `$AIRFLOW_HOME/dags/`. Les dépendances découlent des appels : `extraire → transformer → publier`, les valeurs de retour transitant par XCom (petites données seulement ; les gros résultats passent par un stockage et l'on échange des chemins).

Tester sans scheduler ni interface :

```bash
python dags/rapport_quotidien.py            # le module s'importe : pas d'erreur de syntaxe ni d'import
airflow dags test rapport_quotidien         # exécute une fois toutes les tâches, journaux à l'écran
airflow dags list-import-errors             # Dags refusés par le dag-processor
```

Un second Dag peut se déclencher **quand l'actif est mis à jour** : `@dag(schedule=[RAPPORT], ...)`.

# 7. Ordonnancer

- **Temps** : `schedule="0 6 * * 1-5"`, `"@hourly"`, `timedelta(hours=6)` ; `start_date` et `catchup` décident si les exécutions passées sont rattrapées ; `airflow dags backfill` pour rejouer une période.
- **Données** : `schedule=[asset_a, asset_b]` (tous mis à jour) ou expressions `asset_a | asset_b` ; partitionnement des actifs depuis 3.2.
- **Événements** : un Dag peut être déclenché par un message (SQS, Kafka…) via les déclencheurs asynchrones et le `triggerer`.
- **À la demande** : `schedule=None`, puis `airflow dags trigger`, l'API REST ou l'interface ; depuis 3.3, `@result` et `GET …/dagRuns/{id}/wait` permettent d'attendre le résultat d'une exécution.

# 8. Bonnes pratiques

- **Tâches idempotentes** : rejouer une exécution doit donner le même résultat ; écrire par partition (date, clé), ne pas ajouter à l'aveugle.
- **Rien de lourd au niveau du fichier de Dag** : il est ré-analysé en permanence ; pas de requête ni de lecture de fichier hors des tâches.
- **Secrets hors du code** : connexions et variables Airflow, ou un backend de secrets (Vault, gestionnaires cloud).
- **Petites tâches, dépendances explicites**, TaskGroups pour structurer, `retries` et `retry_delay` raisonnés, alertes sur échec.
- **Tester** : import du module, `airflow dags test`, tests unitaires des fonctions métier hors Airflow ; `ruff` fournit des règles `AIR` qui détectent les usages dépréciés.
- **Versionner** les Dags dans Git ; Airflow 3 conserve les versions et permet de rejouer une exécution avec la version d'origine ou la dernière.
- **Observer** : durées, files d'attente, journaux ; métriques StatsD/OpenTelemetry.

# 9. Migrer depuis Airflow 2

1. Passer d'abord en 2.10 et corriger tous les avertissements de dépréciation ; l'outil `ruff` (règles `AIR30x`) et `airflow config lint` signalent l'essentiel.
2. Remplacer les imports par `airflow.sdk` ; `schedule_interval` → `schedule` ; `execution_date` → `logical_date` ; supprimer SubDAGs et accès directs à la base depuis les tâches.
3. Renommer les datasets en actifs (`Asset`) ; vérifier les fournisseurs installés et leurs versions.
4. Remplacer le service `webserver` par `api-server` dans les déploiements ; exécuter `airflow db migrate`.
5. Rejouer les Dags critiques avec `dags test` avant la mise en production.

# 10. Aide-mémoire

| Besoin | Commande |
|---|---|
| Environnement complet en local | `airflow standalone` |
| Base de métadonnées | `airflow db migrate` |
| Lister, déclencher, rejouer | `airflow dags list`, `airflow dags trigger <id>`, `airflow dags backfill` |
| Tester un Dag | `airflow dags test <id>` |
| Erreurs d'import | `airflow dags list-import-errors` |
| Tâches d'une exécution | `airflow tasks list <id>`, `airflow tasks test <id> <tâche>` |
| Connexions, variables | `airflow connections add …`, `airflow variables set …` |
| Configuration effective | `airflow config list`, `airflow config get-value core executor` |

# 11. Sources

- Documentation Airflow : <https://airflow.apache.org/docs/apache-airflow/stable/> ; notes de version : <https://airflow.apache.org/docs/apache-airflow/stable/release_notes.html>
- Task SDK : <https://airflow.apache.org/docs/task-sdk/stable/>
- Annonces : Airflow 3.0 (avril 2025) <https://airflow.apache.org/blog/airflow-three-point-oh-is-here/>, Airflow 3.3.0 (juillet 2026) <https://airflow.apache.org/blog/airflow-3.3.0/>
- Guide de migration 2 → 3 : <https://airflow.apache.org/docs/apache-airflow/stable/installation/upgrading_to_airflow3.html>
- Docker Compose et Helm : <https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html>, <https://airflow.apache.org/docs/helm-chart/stable/>
- Devoxx France, « Devenez un chef d'orchestre avec Apache Airflow » (Marwen Ben Dhahbia, Airflow 2) : <https://youtu.be/irAgANuiqc0>
