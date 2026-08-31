---
schema_version: 1
uid: 01M1BQ627EE21EVJTW9GFZEV0K
titre: "RAG — 11 — RAG multimodal et documents complexes"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - intelligence-artificielle
  - rag
  - recherche-vectorielle
  - llm
  - embeddings
resume: "Chapitre 11 sur 12 du livre « RAG » : RAG multimodal et documents complexes. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2026-06-03
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « RAG » — chapitre 11/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 10 — Agentic RAG agents, outils et orchestration dynamique|← 10 — Agentic RAG agents, outils et orchestration dynamique]] · [[RAG — 12 — Production, sécurité et observabilité d’un système RAG|12 — Production, sécurité et observabilité d’un système RAG →]]

# Chapitre 11 — RAG multimodal et documents complexes
## 11.1. Introduction

Jusqu’ici, nous avons principalement raisonné sur des documents textuels simples :

```text
Markdown ;
documentation technique ;
paragraphes ;
chunks textuels ;
questions en langage naturel.
```

Dans ce cadre, un pipeline RAG standard peut fonctionner correctement :

```text
document texte
   ↓
chunking
   ↓
embedding
   ↓
retrieval
   ↓
prompt augmenté
   ↓
réponse sourcée
```

Mais dans les cas réels, les documents ne sont pas toujours de simples textes bien structurés.

Nous rencontrons souvent :

```text
PDF ;
scans ;
tableaux ;
images ;
graphiques ;
schémas ;
captures d’écran ;
présentations ;
documents administratifs ;
rapports techniques ;
formulaires ;
factures ;
contrats mis en page ;
pages web complexes ;
documents avec colonnes ;
documents semi-structurés.
```

Ces documents posent un problème important : l’information utile n’est pas toujours contenue dans du texte linéaire facile à extraire.

Par exemple, une procédure peut être dans un tableau. Une architecture peut être représentée sous forme de schéma. Une valeur importante peut être dans une cellule. Une information essentielle peut être portée par la position visuelle d’un élément.

Le **RAG multimodal** vise à traiter ces situations.

L’objectif de ce chapitre est de comprendre comment adapter un système RAG à des documents qui ne sont pas seulement du texte brut.

---

## 11.2. Pourquoi les documents complexes posent problème

Un RAG classique suppose souvent que nous pouvons extraire un texte propre du document.

Exemple idéal :

```text
# Déploiement

Le service checkout est déployé sur cluster-3.
Il utilise l’API payments.
```

Ce texte est facile à découper, indexer et citer.

Mais un PDF réel peut ressembler à ceci :

```text
page avec deux colonnes ;
tableau de correspondance service / cluster ;
schéma d’architecture ;
notes de bas de page ;
en-tête répété ;
pied de page ;
légende ;
image intégrée ;
texte scanné.
```

Si nous appliquons une extraction naïve, nous pouvons obtenir un texte comme :

```text
checkout cluster-3 payments API production owner Jean monitoring yes
billing cluster-4 invoices API staging owner Léa monitoring no
```

Ce texte est difficile à comprendre.

Le RAG peut alors échouer pour une raison simple :

```text
l’information existe dans le document,
mais elle est mal extraite ou mal représentée.
```

Un mauvais pipeline d’ingestion produit un mauvais retrieval.

---

## 11.3. Les limites du texte linéaire

Un document visuel contient souvent de l’information dans sa structure.

Prenons un tableau :

|Service|Cluster|Environnement|Responsable|
|---|---|---|---|
|checkout|cluster-3|production|équipe payments|
|billing|cluster-4|production|équipe finance|

Si nous extrayons seulement les mots dans l’ordre approximatif, nous pouvons obtenir :

```text
Service Cluster Environnement Responsable checkout cluster-3 production équipe payments billing cluster-4 production équipe finance
```

Ce texte contient les mots, mais il perd la structure.

Or la structure est essentielle.

Nous voulons conserver les relations :

```text
checkout → cluster-3 ;
checkout → production ;
checkout → équipe payments ;
billing → cluster-4 ;
billing → production ;
billing → équipe finance.
```

Même problème avec un schéma :

```text
frontend → API → payments API → database
```

Si l’extraction ignore les flèches, nous perdons les relations.

Le RAG multimodal cherche donc à préserver non seulement le texte, mais aussi :

```text
la structure ;
la mise en page ;
les relations spatiales ;
les tableaux ;
les légendes ;
les images ;
les diagrammes ;
les liens visuels.
```

---

## 11.4. Qu’est-ce que le RAG multimodal ?

Un RAG multimodal est un système RAG capable de traiter plusieurs types de contenus :

```text
texte ;
image ;
tableau ;
schéma ;
graphique ;
page scannée ;
audio ou vidéo éventuellement ;
document semi-structuré.
```

Dans un RAG textuel classique, nous faisons :

```text
texte → embedding texte → recherche → réponse
```

Dans un RAG multimodal, nous pouvons faire :

```text
image → embedding image ;
page PDF → représentation visuelle ;
tableau → structure tabulaire ;
schéma → description ou graphe ;
texte + image → embedding multimodal ;
```

Le système peut ensuite répondre à des questions comme :

```text
Que montre ce schéma d’architecture ?
Quel service dépend de Redis dans ce diagramme ?
Quelle valeur est indiquée dans le tableau ?
Quel est le total de cette facture ?
Quels risques sont visibles dans cette matrice ?
Quelle tendance montre ce graphique ?
```

---

## 11.5. Les types de documents complexes

### 11.5.1. PDF textuels

Ce sont des PDF contenant du texte sélectionnable.

Ils sont plus simples à traiter, mais la mise en page peut poser problème.

Difficultés :

```text
colonnes ;
en-têtes et pieds de page ;
tableaux ;
titres mal détectés ;
ordre de lecture incorrect ;
notes de bas de page ;
pages mélangées.
```

### 11.5.2. PDF scannés

Ce sont des images de pages.

Il faut utiliser de l’OCR pour extraire le texte.

Difficultés :

```text
qualité du scan ;
rotation ;
bruit ;
tampons ;
écriture manuscrite ;
colonnes ;
tableaux ;
erreurs de reconnaissance.
```

### 11.5.3. Tableaux

Les tableaux contiennent une information structurée.

Difficultés :

```text
cellules fusionnées ;
colonnes mal détectées ;
lignes coupées sur plusieurs pages ;
unités ;
totaux ;
notes ;
hiérarchie de colonnes.
```

### 11.5.4. Graphiques

Les graphiques peuvent contenir :

```text
courbes ;
histogrammes ;
camemberts ;
axes ;
légendes ;
valeurs ;
tendances.
```

Difficulté principale :

```text
le sens dépend de la lecture visuelle.
```

### 11.5.5. Schémas

Les schémas d’architecture, organigrammes et diagrammes de flux contiennent des relations.

Difficultés :

```text
flèches ;
boîtes ;
groupes ;
zones ;
couleurs ;
icônes ;
relations implicites.
```

### 11.5.6. Captures d’écran

Elles peuvent contenir :

```text
interfaces ;
messages d’erreur ;
boutons ;
menus ;
logs ;
états visuels.
```

Le texte extrait ne suffit pas toujours : il faut comprendre la disposition.

---

## 11.6. OCR : reconnaissance optique de caractères

L’OCR, pour _Optical Character Recognition_, consiste à convertir une image contenant du texte en texte exploitable.

Exemple :

```text
image scannée d’une facture
   ↓
OCR
   ↓
texte : "Montant total : 1 240,50 €"
```

L’OCR est souvent nécessaire pour :

```text
PDF scannés ;
photos de documents ;
captures d’écran ;
formulaires papier ;
archives.
```

Mais l’OCR peut produire des erreurs.

Exemples :

```text
0 confondu avec O ;
1 confondu avec l ;
€ mal reconnu ;
colonnes mélangées ;
accents perdus ;
noms propres déformés ;
lignes inversées.
```

Dans un système RAG, ces erreurs peuvent avoir des conséquences importantes.

Si l’OCR lit :

```text
cluster-8
```

au lieu de :

```text
cluster-3
```

le retrieval et la réponse seront faux.

Nous devons donc traiter l’OCR comme une étape critique, pas comme une simple formalité.

---

## 11.7. Layout : préserver la mise en page

Le **layout** désigne la structure visuelle du document.

Exemples :

```text
titres ;
paragraphes ;
colonnes ;
tableaux ;
encadrés ;
figures ;
légendes ;
notes ;
ordre de lecture ;
position sur la page.
```

Un bon pipeline RAG sur PDF doit essayer de préserver cette structure.

### 11.7.1. Exemple de mauvaise extraction

Document visuel :

|Service|Cluster|
|---|---|
|checkout|cluster-3|
|billing|cluster-4|

Extraction naïve :

```text
Service checkout billing Cluster cluster-3 cluster-4
```

Cette extraction peut laisser croire que :

```text
checkout → billing ;
Cluster → cluster-3 ;
```

ce qui est incohérent.

### 11.7.2. Extraction structurée

Meilleure représentation :

```text
Tableau : Déploiement des services

Ligne 1 :
Service = checkout
Cluster = cluster-3

Ligne 2 :
Service = billing
Cluster = cluster-4
```

Cette représentation est beaucoup plus utile pour un RAG.

---

## 11.8. Représenter les tableaux

Les tableaux doivent souvent être traités différemment du texte libre.

Nous pouvons les représenter en Markdown :

```markdown
| Service | Cluster | Environnement |
|---|---|---|
| checkout | cluster-3 | production |
| billing | cluster-4 | production |
```

Ou en lignes textuelles :

```text
Dans le tableau "Déploiement des services" :
- checkout est déployé sur cluster-3 en production.
- billing est déployé sur cluster-4 en production.
```

Ou en JSON :

```json
[
  {
    "service": "checkout",
    "cluster": "cluster-3",
    "environment": "production"
  },
  {
    "service": "billing",
    "cluster": "cluster-4",
    "environment": "production"
  }
]
```

Le bon format dépend du type de question attendu.

Pour un LLM, les lignes textuelles explicites sont souvent très efficaces.

Pour une API ou une analyse structurée, JSON peut être préférable.

---

## 11.9. Tableaux et chunking

Chunker un tableau est délicat.

Si le tableau est petit, nous pouvons le garder entier.

Si le tableau est grand, il faut le découper.

Mais il faut éviter de séparer les lignes de leurs en-têtes.

Mauvais chunk :

```text
checkout cluster-3 production
billing cluster-4 production
```

Bon chunk :

```text
Tableau : Déploiement des services
Colonnes : Service, Cluster, Environnement

Ligne :
Service = checkout
Cluster = cluster-3
Environnement = production
```

Pour les tableaux longs, nous pouvons créer un chunk par ligne ou par groupe de lignes, en répétant les en-têtes.

Cela augmente un peu la redondance, mais préserve le sens.

---

## 11.10. Représenter les schémas

Un schéma peut être converti en description textuelle.

Exemple de schéma :

```text
frontend → API → payments API → MariaDB
                  ↓
                 Redis
```

Description textuelle :

```text
Le frontend appelle l’API principale.
L’API principale appelle payments API.
payments API écrit dans MariaDB.
payments API utilise Redis.
```

Nous pouvons aussi le représenter sous forme de graphe :

```text
frontend --calls--> API
API --calls--> payments API
payments API --writes_to--> MariaDB
payments API --uses--> Redis
```

Pour un RAG technique, cette représentation est très utile.

Elle peut alimenter à la fois :

```text
un index textuel ;
un graphe de connaissances ;
un Graph RAG.
```

---

## 11.11. Schémas et Graph RAG

Les schémas d’architecture sont naturellement compatibles avec Graph RAG.

Un schéma contient souvent :

```text
composants ;
flèches ;
dépendances ;
groupes ;
zones réseau ;
environnements ;
bases de données ;
files ;
APIs.
```

Nous pouvons convertir le schéma en triplets :

```text
service A --calls--> service B ;
service B --writes_to--> database C ;
service C --runs_on--> cluster D.
```

Exemple :

```text
checkout --uses--> payments API
payments API --runs_on--> cluster-3
payments API --writes_to--> MariaDB
```

Cela permet ensuite de répondre à :

```text
Quels services dépendent de payments API ?
Quels composants sont affectés si MariaDB tombe ?
Quels flux traversent la zone DMZ ?
Quels services tournent sur cluster-3 ?
```

Un RAG multimodal peut donc être une étape d’entrée vers un Graph RAG.

---

## 11.12. Images et embeddings multimodaux

Pour les images, nous pouvons utiliser des embeddings multimodaux.

Un modèle multimodal peut représenter dans un même espace :

```text
du texte ;
des images ;
des captures ;
des pages de document.
```

Cela permet de poser une question textuelle et de retrouver une image pertinente.

Exemple :

```text
Question :
schéma de l’architecture payments

Résultat :
image contenant le diagramme payments-api → MariaDB → Redis.
```

Ou inversement :

```text
image d’un schéma
   ↓
embedding image
   ↓
recherche de textes liés
```

Les embeddings multimodaux sont utiles lorsque l’image elle-même porte l’information.

Mais ils ne remplacent pas toujours l’extraction structurée.

Pour un tableau ou un schéma technique, il est souvent préférable d’extraire explicitement les relations.

---

## 11.13. Deux stratégies pour les documents visuels

Pour traiter une page complexe, nous avons généralement deux stratégies.

### 11.13.1. Stratégie 1 : convertir en texte structuré

Nous transformons la page en une représentation textuelle exploitable.

Exemple :

```text
Page 7 — Schéma d’architecture

Le frontend appelle l’API principale.
L’API principale appelle payments API.
payments API utilise Redis pour les files.
payments API écrit dans MariaDB.
```

Avantage :

```text
compatible avec un RAG textuel classique ;
facile à citer ;
facile à relire ;
peut alimenter un graphe.
```

Limite :

```text
risque de perdre des informations visuelles ;
qualité dépendante du modèle d’extraction.
```

### 11.13.2. Stratégie 2 : indexer l’image ou la page directement

Nous calculons un embedding de la page ou de l’image.

Avantage :

```text
préserve l’information visuelle ;
utile pour retrouver des schémas ou captures.
```

Limite :

```text
moins précis pour extraire des valeurs ;
plus difficile à citer précisément ;
peut nécessiter un modèle multimodal à la génération.
```

En pratique, nous combinons souvent les deux.

---

## 11.14. Pipeline RAG pour PDF complexe

Un pipeline réaliste peut ressembler à ceci :

```text
PDF
   ↓
découpage en pages
   ↓
détection du layout
   ↓
extraction du texte
   ↓
OCR si nécessaire
   ↓
extraction des tableaux
   ↓
description des images et schémas
   ↓
normalisation
   ↓
création de chunks structurés
   ↓
embeddings texte et/ou multimodaux
   ↓
index vectoriel + métadonnées
   ↓
retrieval
   ↓
réponse avec citations page/section
```

Ce pipeline est plus complexe qu’un RAG sur Markdown, mais il est nécessaire pour des documents réels.

---

## 11.15. Métadonnées spécifiques aux documents complexes

Pour les PDF et documents visuels, les métadonnées sont encore plus importantes.

Exemples :

```text
document_id ;
titre ;
page ;
position sur la page ;
type de bloc ;
section ;
tableau ;
figure ;
légende ;
coordonnées ;
qualité OCR ;
source image ;
date ;
version ;
droits d’accès.
```

Exemple :

```json
{
  "document_id": "rapport_infra_2026",
  "page": 12,
  "block_type": "table",
  "table_title": "Déploiement des services",
  "rows": 24,
  "source": "PDF original",
  "ocr_confidence": 0.96
}
```

Ces métadonnées permettent :

```text
de citer précisément ;
de revenir à la page originale ;
de filtrer par type de contenu ;
de vérifier la qualité de l’OCR ;
de distinguer texte, tableau et figure.
```

---

## 11.16. Citations dans un RAG multimodal

Citer une source textuelle est relativement simple :

```text
Document X, section Y.
```

Citer une source multimodale demande plus de précision.

Exemples :

```text
Rapport infrastructure 2026, page 12, tableau "Déploiement des services".
```

ou :

```text
Schéma d’architecture, page 7, figure 2.
```

ou :

```text
Capture d’écran, section "Erreur de build", message affiché en haut de page.
```

Une bonne citation doit permettre à l’utilisateur de retrouver l’information dans le document original.

Dans un contexte professionnel, c’est essentiel.

---

## 11.17. RAG multimodal et hallucinations

Les documents complexes augmentent les risques d’erreur.

Exemples :

```text
un chiffre mal lu par OCR ;
une colonne associée à la mauvaise ligne ;
une flèche de schéma mal interprétée ;
une légende ignorée ;
une valeur extrapolée depuis un graphique ;
une image décrite trop librement ;
une capture d’écran mal contextualisée.
```

Le modèle peut produire une réponse convaincante mais fausse.

Nous devons donc être prudents.

Stratégies :

```text
conserver les sources originales ;
citer page/table/figure ;
indiquer les incertitudes d’OCR ;
éviter les conclusions trop fortes ;
valider les extractions critiques ;
utiliser des formats structurés ;
mettre en place une revue humaine pour les cas sensibles.
```

---

## 11.18. Exemple : tableau de services

Document :

|Service|Dépendance|Environnement|
|---|---|---|
|checkout|payments API|production|
|payments API|MariaDB|production|
|notification|Redis|production|

Question :

```text
Quels services peuvent être affectés si Redis tombe ?
```

Si nous représentons le tableau sous forme brute :

```text
checkout payments API production payments API MariaDB production notification Redis production
```

le RAG peut se tromper.

Meilleure représentation :

```text
Dans le tableau des dépendances :
- le service checkout dépend de payments API en production ;
- le service payments API dépend de MariaDB en production ;
- le service notification dépend de Redis en production.
```

Réponse attendue :

```text
Le service notification peut être affecté si Redis tombe, car le tableau indique
que notification dépend de Redis en production.
```

Nous voyons ici que la qualité de la représentation du tableau est déterminante.

---

## 11.19. Exemple : schéma d’architecture

Schéma :

```text
client web → API gateway → checkout → payments API → MariaDB
                                      ↓
                                    Redis
```

Question :

```text
Quels composants sont impliqués dans un paiement ?
```

Description structurée :

```text
Le client web appelle l’API gateway.
L’API gateway appelle checkout.
checkout appelle payments API.
payments API écrit dans MariaDB.
payments API utilise Redis.
```

Réponse :

```text
Les composants impliqués sont le client web, l’API gateway, le service checkout,
payments API, MariaDB et Redis. Le flux principal va du client web vers l’API gateway,
puis vers checkout, puis vers payments API, qui utilise MariaDB et Redis.
```

Sans extraction du schéma, un RAG textuel classique ne pourrait pas répondre correctement.

---

## 11.20. Exemple : facture ou document administratif

Document scanné :

```text
Montant total : 1 240,50 €
Date d’échéance : 30/06/2026
Référence : FAC-2026-041
```

Question :

```text
Quelle est la date d’échéance de la facture ?
```

Pipeline :

```text
OCR
   ↓
extraction structurée
   ↓
indexation avec métadonnées
   ↓
retrieval
   ↓
réponse sourcée
```

Réponse :

```text
La date d’échéance indiquée sur la facture est le 30/06/2026.
```

Mais si l’OCR lit :

```text
30/08/2026
```

la réponse sera fausse.

Pour des données critiques, il faut parfois conserver le scan original et permettre une vérification humaine.

---

## 11.21. Documents semi-structurés

Certains documents ne sont ni complètement libres ni complètement structurés.

Exemples :

```text
formulaires ;
factures ;
bons de commande ;
rapports d’intervention ;
fiches techniques ;
emails avec tableaux ;
tickets support ;
fichiers YAML ;
logs ;
exports CSV.
```

Ces documents peuvent être traités avec des extracteurs spécialisés.

Exemple pour YAML :

```yaml
service: checkout
cluster: cluster-3
environment: production
```

Il serait dommage de transformer cela en texte plat sans préserver la structure.

Représentation utile :

```text
Le service checkout est déployé sur cluster-3 en environnement production.
```

Ou directement :

```json
{
  "service": "checkout",
  "cluster": "cluster-3",
  "environment": "production"
}
```

Le choix dépend de l’usage.

---

## 11.22. RAG sur logs

Les logs sont un cas particulier de document semi-structuré.

Exemple :

```text
2026-06-04T10:15:23Z ERROR notification-service Redis connection timeout
```

Une recherche RAG classique peut aider à interpréter les logs, mais les logs nécessitent souvent :

```text
filtrage temporel ;
agrégation ;
recherche exacte ;
corrélation ;
détection de patterns ;
analyse statistique ;
lien avec incidents ;
lien avec métriques.
```

Un agent ou un pipeline spécialisé peut être plus adapté qu’un simple RAG vectoriel.

Question :

```text
Les erreurs notification-service correspondent-elles à la panne Redis ?
```

Il faut comparer :

```text
horodatages ;
messages d’erreur ;
dépendances ;
incidents ;
métriques.
```

C’est un bon cas pour Agentic RAG.

---

## 11.23. RAG multimodal et multimodalité à la génération

Deux approches sont possibles.

### 11.23.1. Extraction avant génération

Nous transformons d’abord les images, tableaux et schémas en texte structuré.

Puis le LLM répond à partir de ce texte.

Avantage :

```text
plus simple ;
plus auditable ;
compatible avec un LLM textuel ;
citations plus faciles.
```

### 11.23.2. Modèle multimodal à la génération

Nous envoyons directement l’image ou la page au modèle multimodal.

Avantage :

```text
meilleure compréhension visuelle possible ;
utile pour schémas complexes ;
utile quand l’extraction textuelle est insuffisante.
```

Limite :

```text
plus coûteux ;
moins déterministe ;
citations parfois plus difficiles ;
dépend fortement du modèle.
```

En pratique, nous pouvons combiner :

```text
extraction structurée pour indexation ;
modèle multimodal pour vérification ou questions complexes.
```

---

## 11.24. Indexation multimodale

Un système avancé peut stocker plusieurs représentations du même document.

Exemple pour une page PDF :

```text
texte extrait ;
image de la page ;
tableaux extraits ;
description des figures ;
embedding texte ;
embedding image ;
métadonnées de layout ;
liens vers le document original.
```

Lors du retrieval, le système peut chercher :

```text
dans le texte ;
dans les tableaux ;
dans les images ;
dans les descriptions ;
dans les métadonnées.
```

Puis il fusionne les résultats.

Cela ressemble à une recherche hybride, mais appliquée à plusieurs modalités.

---

## 11.25. Exemple d’architecture RAG multimodal

Architecture possible :

```text
Documents bruts
   ↓
Détection du type
   ↓
PDF textuel ? → extraction texte + layout
PDF scan ? → OCR + layout
Tableau ? → extraction tabulaire
Image ? → description + embedding image
Schéma ? → extraction relations + description
   ↓
Normalisation
   ↓
Chunks textuels structurés
   ↓
Embeddings texte / image
   ↓
Index textuel + index multimodal + métadonnées
   ↓
Retriever multimodal
   ↓
Fusion et reranking
   ↓
Prompt avec sources
   ↓
LLM textuel ou multimodal
```

Cette architecture est plus coûteuse, mais beaucoup plus robuste sur des documents réels.

---

## 11.26. Évaluation d’un RAG multimodal

Évaluer un RAG multimodal est plus difficile qu’évaluer un RAG textuel.

Nous devons évaluer :

```text
qualité OCR ;
qualité d’extraction des tableaux ;
qualité d’interprétation des schémas ;
qualité des embeddings multimodaux ;
qualité du retrieval ;
qualité des citations page/figure/tableau ;
fidélité de la réponse ;
exactitude des valeurs numériques ;
gestion des incertitudes.
```

### 11.26.1. Questions d’évaluation

```text
La bonne page est-elle retrouvée ?
Le bon tableau est-il retrouvé ?
La bonne cellule est-elle lue ?
La relation dans le schéma est-elle correctement interprétée ?
La réponse cite-t-elle la bonne page ou figure ?
Le système signale-t-il une faible confiance OCR ?
```

### 11.26.2. Cas critiques

Les valeurs numériques doivent être évaluées avec attention.

Exemple :

```text
1 240,50 €
```

ne doit pas devenir :

```text
124,50 €
```

ou :

```text
1 240,00 €
```

Pour les documents administratifs, financiers ou juridiques, ces erreurs peuvent être graves.

---

## 11.27. Bonnes pratiques

### 11.27.1. Ne pas tout aplatir en texte brut

Nous devons préserver la structure quand elle porte du sens.

### 11.27.2. Répéter les en-têtes de tableaux

Chaque chunk de tableau doit rester compréhensible seul.

### 11.27.3. Conserver les références de page

Les citations doivent permettre de revenir au document original.

### 11.27.4. Distinguer texte extrait et interprétation

Une description de schéma produite par un modèle est une interprétation, pas le document original.

### 11.27.5. Indiquer les incertitudes

Si l’OCR est faible, le système doit le signaler.

### 11.27.6. Utiliser des extracteurs spécialisés

Un PDF, un tableau, une image et un fichier YAML ne doivent pas forcément être traités de la même façon.

### 11.27.7. Évaluer par type de document

Un bon score global peut cacher de mauvais résultats sur les tableaux ou les scans.

---

## 11.28. Quand utiliser du multimodal ?

Le RAG multimodal est utile lorsque :

```text
les documents contiennent beaucoup de tableaux ;
les PDF sont scannés ;
les images portent l’information ;
les schémas d’architecture sont importants ;
les graphiques doivent être interprétés ;
la mise en page contient du sens ;
les documents sont semi-structurés ;
les citations doivent pointer vers des pages ou figures.
```

Il est moins nécessaire lorsque :

```text
le corpus est en Markdown propre ;
les documents sont déjà structurés ;
les questions portent uniquement sur du texte ;
les tableaux peuvent être extraits simplement en CSV ;
la complexité visuelle est faible.
```

Comme toujours, nous devons choisir l’architecture selon le besoin réel.

---

## 11.29. Mini-TP : transformer un tableau en chunks

### Objectif

Nous voulons apprendre à représenter un tableau pour un RAG.

### Tableau

|Service|Dépendance|Environnement|
|---|---|---|
|checkout|payments API|production|
|payments API|MariaDB|production|
|notification|Redis|production|

### Travail demandé

Nous produisons des chunks textuels exploitables.

### Solution possible

```text
Chunk 1 :
Dans le tableau "Dépendances des services", le service checkout dépend de
payments API en environnement production.

Chunk 2 :
Dans le tableau "Dépendances des services", le service payments API dépend de
MariaDB en environnement production.

Chunk 3 :
Dans le tableau "Dépendances des services", le service notification dépend de
Redis en environnement production.
```

### Question de test

```text
Quels services dépendent de Redis ?
```

Réponse attendue :

```text
Le service notification dépend de Redis en production.
```

---

## 11.30. Mini-TP : transformer un schéma en graphe

### Schéma textuel

```text
frontend → api → checkout → payments API → MariaDB
                          ↓
                         Redis
```

### Travail demandé

Nous extrayons les relations.

### Relations attendues

```text
frontend --calls--> api
api --calls--> checkout
checkout --calls--> payments API
payments API --writes_to--> MariaDB
payments API --uses--> Redis
```

### Question

```text
Quels composants sont impliqués dans le flux de paiement ?
```

### Réponse attendue

```text
Le flux implique frontend, api, checkout, payments API, MariaDB et Redis.
```

---

## 11.31. Mini-TP : détecter les risques d’OCR

### Texte OCR extrait

```text
Montant total : 1240,50 €
Date d’échéance : 30/08/2026
Référence : FAC-2026-O41
```

### Document original attendu

```text
Montant total : 1240,50 €
Date d’échéance : 30/06/2026
Référence : FAC-2026-041
```

### Travail demandé

Nous identifions les erreurs possibles :

```text
30/08/2026 au lieu de 30/06/2026 ;
O41 au lieu de 041.
```

### Discussion

Nous devons comprendre que l’OCR peut introduire des erreurs difficiles à détecter automatiquement.

Pour les valeurs critiques, une vérification humaine ou une validation croisée peut être nécessaire.

---

## 11.32. Questions de compréhension

À la fin de ce chapitre, nous devons pouvoir répondre aux questions suivantes :

```text
Pourquoi les documents complexes posent-ils problème à un RAG textuel ?
Qu’est-ce qu’un RAG multimodal ?
Pourquoi le layout est-il important ?
Quelles erreurs peut produire l’OCR ?
Comment représenter un tableau pour un RAG ?
Pourquoi faut-il répéter les en-têtes dans les chunks de tableau ?
Comment représenter un schéma d’architecture ?
Quel lien existe entre schémas et Graph RAG ?
Quelle différence entre extraction structurée et embedding multimodal ?
Pourquoi les citations sont-elles plus complexes dans un RAG multimodal ?
Quels types de métadonnées faut-il conserver ?
Comment évaluer un RAG multimodal ?
Dans quels cas le multimodal est-il nécessaire ?
```

---

## 11.33. Synthèse du chapitre

Dans ce chapitre, nous avons étudié le RAG multimodal et les documents complexes.

Nous avons vu qu’un RAG textuel classique fonctionne bien lorsque les documents sont propres, linéaires et structurés. Mais il devient insuffisant lorsque l’information est portée par :

```text
tableaux ;
images ;
schémas ;
graphiques ;
scans ;
mise en page ;
formulaires ;
documents semi-structurés.
```

Nous avons compris que l’enjeu principal est de préserver le sens du document lors de l’ingestion.

Cela implique :

```text
OCR ;
analyse de layout ;
extraction de tableaux ;
description de figures ;
représentation de schémas ;
métadonnées précises ;
citations page/table/figure ;
évaluation spécifique.
```

Nous avons aussi vu que le multimodal ne signifie pas forcément envoyer toutes les images au LLM. Souvent, la meilleure stratégie consiste à transformer les contenus visuels en représentations structurées :

```text
tableaux → lignes explicites ;
schémas → relations ;
graphiques → tendances et valeurs ;
PDF → blocs structurés avec pages et sections.
```

Le message principal du chapitre est donc le suivant :

```text
Un RAG sur documents complexes ne doit pas seulement extraire du texte.
Il doit préserver la structure, la position, les relations et les éléments visuels
qui portent l’information.
```

Dans le chapitre suivant, nous étudierons la mise en production d’un système RAG : sécurité, droits d’accès, observabilité, coûts, performances, mises à jour et exploitation.

---

---
> [!info] Livre « RAG » — chapitre 11/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 10 — Agentic RAG agents, outils et orchestration dynamique|← 10 — Agentic RAG agents, outils et orchestration dynamique]] · [[RAG — 12 — Production, sécurité et observabilité d’un système RAG|12 — Production, sécurité et observabilité d’un système RAG →]]
