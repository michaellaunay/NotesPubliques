---
schema_version: 1
uid: 01M1BQ6277AA85JYNS41HGDN2F
titre: "RAG — 04 — Construire un RAG standard"
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
resume: "Chapitre 4 sur 12 du livre « RAG » : Construire un RAG standard. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « RAG » — chapitre 4/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 03 — Similarité cosinus et recherche vectorielle|← 03 — Similarité cosinus et recherche vectorielle]] · [[RAG — 05 — Prompt augmenté et génération sourcée|05 — Prompt augmenté et génération sourcée →]]

# Chapitre 4 — Construire un RAG standard
## 4.1. Introduction

Dans les chapitres précédents, nous avons posé les bases du RAG.

Nous avons vu pourquoi un LLM seul ne suffit pas toujours lorsqu’il doit répondre à partir de documents spécifiques, récents ou vérifiables.

Nous avons ensuite étudié les **embeddings**, qui permettent de transformer du texte en vecteurs numériques.

Puis nous avons vu comment la **similarité cosinus** permet de comparer ces vecteurs pour retrouver les passages les plus proches d’une question.

Dans ce chapitre, nous allons assembler ces éléments pour construire un **RAG standard complet**.

Nous allons passer d’une vision conceptuelle :

```text
question → recherche → contexte → réponse
```

à une architecture plus concrète :

```text
documents bruts
        ↓
extraction
        ↓
nettoyage
        ↓
chunking
        ↓
embeddings
        ↓
index vectoriel
        ↓
retrieval
        ↓
prompt augmenté
        ↓
réponse du LLM
```

L’objectif de ce chapitre est que nous soyons capables d’expliquer chaque étape d’un pipeline RAG standard, de comprendre les choix techniques associés, et d’identifier les principales sources d’erreur.

---

## 4.2. Vue d’ensemble d’un RAG standard

Un RAG standard repose généralement sur deux grandes phases :

```text
1. La phase d’indexation
2. La phase de requête
```

La phase d’indexation prépare les documents avant toute question utilisateur.

La phase de requête est déclenchée lorsqu’un utilisateur pose une question.

Nous pouvons représenter l’ensemble ainsi :

```text
PHASE D’INDEXATION

Documents
   ↓
Extraction du texte
   ↓
Nettoyage
   ↓
Découpage en chunks
   ↓
Calcul des embeddings
   ↓
Stockage dans un index vectoriel


PHASE DE REQUÊTE

Question utilisateur
   ↓
Embedding de la question
   ↓
Recherche des chunks proches
   ↓
Construction du prompt augmenté
   ↓
Appel au LLM
   ↓
Réponse sourcée
```

Cette séparation est importante.

L’indexation peut être coûteuse, mais elle n’est pas faite à chaque question. Elle est réalisée lors de l’ajout ou de la mise à jour des documents.

La phase de requête, elle, doit être rapide, car elle intervient pendant l’interaction avec l’utilisateur.

---

## 4.3. Phase d’indexation : préparer la mémoire documentaire

La phase d’indexation consiste à transformer des documents bruts en une base interrogeable par recherche vectorielle.

Nous partons de documents tels que :

```text
PDF
Markdown
HTML
documents Word
wikis
tickets
README
pages de documentation
contrats
rapports
fichiers de configuration
dépôts Git
```

Ces documents ne sont pas directement utilisables par un RAG. Nous devons les convertir en morceaux de texte propres, structurés et indexables.

---

## 4.4. Étape 1 : collecte des documents

La première étape consiste à identifier les sources documentaires.

Dans un projet réel, les documents peuvent être dispersés dans plusieurs systèmes :

```text
GitLab ou GitHub
Confluence
Notion
SharePoint
Google Drive
wiki interne
base de tickets
CMS
répertoire de fichiers
base SQL
emails
```

Nous devons donc commencer par définir le périmètre documentaire.

Questions à se poser :

```text
Quels documents voulons-nous rendre interrogeables ?
Qui est propriétaire de ces documents ?
À quelle fréquence changent-ils ?
Existe-t-il plusieurs versions ?
Certains documents sont-ils confidentiels ?
Quels formats devons-nous gérer ?
```

Cette étape est souvent sous-estimée. Pourtant, un RAG est fortement dépendant de la qualité et de la cohérence de ses sources.

Un mauvais périmètre documentaire produit un mauvais assistant.

---

## 4.5. Étape 2 : extraction du texte

Une fois les documents collectés, nous devons en extraire le texte.

Cette étape est simple pour certains formats, mais complexe pour d’autres.

### 4.5.1. Markdown

Le Markdown est généralement très favorable au RAG.

Il contient déjà une structure claire :

```markdown
# Titre

## Section

Texte explicatif

- liste
- élément
- élément
```

Les titres peuvent être conservés comme métadonnées ou ajoutés au contenu des chunks.

Exemple :

```text
Document : guide-deploiement.md
Section : Déploiement en production
```

Ces informations aideront ensuite le système à citer correctement les sources.

### 4.5.2. HTML

Le HTML peut être exploitable, mais il contient souvent beaucoup de bruit :

```text
menus
barres de navigation
pieds de page
scripts
publicités
liens répétitifs
éléments visuels inutiles
```

Il faut donc extraire le contenu principal et supprimer ce qui n’apporte pas d’information.

### 4.5.3. PDF

Le PDF est plus difficile.

Un PDF peut contenir :

```text
texte sélectionnable
images scannées
tableaux
colonnes
notes de bas de page
schémas
en-têtes et pieds de page
mise en page complexe
```

Si le PDF est un scan, il faut utiliser de l’OCR.

Si le PDF contient des tableaux ou des schémas, une extraction texte naïve peut perdre une partie importante de l’information.

Exemple de problème :

```text
Colonne 1 : Service
Colonne 2 : Cluster
Colonne 3 : Responsable
```

Une mauvaise extraction peut mélanger les colonnes et produire un texte incompréhensible.

### 4.5.4. Documents bureautiques

Les fichiers Word, LibreOffice ou Google Docs peuvent contenir :

```text
titres
commentaires
tableaux
images
notes
historique de révision
styles
```

Nous devons décider ce qui doit être conservé.

Dans un RAG documentaire, les commentaires ou métadonnées peuvent parfois être aussi importants que le texte principal.

### 4.5.5. Code source

Pour du code, l’extraction ne consiste pas seulement à lire du texte.

Nous devons préserver :

```text
chemin du fichier
nom des fonctions
classes
imports
types
commentaires
structure du projet
langage
historique Git éventuellement
```

Un chunk de code sans son chemin ou sans le nom de la classe peut perdre beaucoup de sens.

### 4.5.6. Outils de conversion vers le texte et le Markdown

Écrire soi-même l'extraction pour chaque format est rarement rentable : des outils spécialisés produisent directement du Markdown ou du JSON structuré à partir des formats courants. État en août 2026 (versions vérifiées sur PyPI) :

| Outil | Formats | Licence | Points forts | Limites |
|---|---|---|---|---|
| **MarkItDown** 0.1 (Microsoft) | PDF, Word, PowerPoint, Excel, HTML, images, audio, ZIP, EPUB | MIT | une commande, sortie Markdown, serveur MCP, description des images par un LLM | PDF lu par pdfminer : les tableaux sont aplatis, la mise en page perdue |
| **Docling** 2.x (IBM) | PDF, Word, PowerPoint, HTML, images | MIT | modèles de mise en page, tableaux reconstitués, OCR intégrable, export Markdown et JSON avec hiérarchie | lent, modèles à télécharger |
| **marker** 2.x | PDF, EPUB, images | GPL | très bonne fidélité sur les livres et articles, formules | licence GPL, GPU conseillé |
| **pymupdf4llm** | PDF | AGPL ou licence commerciale | rapide, tableaux, Markdown avec titres | licence contraignante pour un produit |
| **Unstructured** 0.27 | une vingtaine de formats | Apache 2.0 (bibliothèque) | éléments typés (titre, paragraphe, tableau) prêts pour le chunking | dépendances lourdes, service commercial en parallèle |
| **Apache Tika** | plus de mille formats | Apache 2.0 | extraction et métadonnées, serveur REST ; Java | texte brut, pas de structure |

Pour le Web, on ne colle pas du HTML brut dans l'index :

| Outil | Rôle | Licence |
|---|---|---|
| **FireCrawl** | explore un site (JavaScript rendu, pagination) et renvoie Markdown ou JSON propre par page ; disponible en service ou auto-hébergé | AGPL (auto-hébergement) |
| **Crawl4AI** | explorateur asynchrone fondé sur Playwright, sorties Markdown adaptées aux LLM, extraction structurée | Apache 2.0 |
| **trafilatura** | extrait le contenu principal d'une page (sans menus ni publicités), avec métadonnées et date | Apache 2.0 |

Exemple avec MarkItDown, qui suffit pour une documentation bureautique :

```bash
python -m pip install "markitdown[all]"
markitdown rapport.pdf -o rapport.md
```

```python
from markitdown import MarkItDown

resultat = MarkItDown().convert("rapport.pdf")
print(resultat.text_content[:200])
```

Sur un PDF contenant un tableau, MarkItDown restitue le texte des paragraphes mais **une cellule par ligne** pour le tableau : dès que les tableaux comptent, Docling ou marker s'imposent. Pour les scans et les livres, la chaîne complète (prétraitement, OCR, normalisation) est décrite dans [[Chaîne complète de numérisation OCR Markdown traduction RAG local]] ; les bibliothèques bas niveau (pypdf, pdfplumber, PyMuPDF) sont comparées dans [[PyPdf]].

Règle de choix : **MarkItDown** pour convertir vite et large, **Docling** quand la structure (tableaux, titres, ordre de lecture) conditionne la qualité des chunks, **trafilatura** ou **Crawl4AI** pour un site public, **FireCrawl** quand le site dépend du JavaScript et que l'on accepte un service ou un déploiement AGPL.

---

## 4.6. Étape 3 : nettoyage et normalisation

Après l’extraction, nous devons nettoyer le texte.

Le nettoyage vise à supprimer le bruit sans détruire l’information utile.

Exemples de bruit à supprimer :

```text
menus répétés
pieds de page identiques
numéros de page inutiles
bannières
éléments de navigation
scripts
fragments HTML
espaces excessifs
caractères corrompus
```

Mais nous devons faire attention : un nettoyage trop agressif peut supprimer des informations importantes.

Par exemple, il ne faut pas supprimer automatiquement toutes les petites phrases, car une phrase courte peut être essentielle :

```text
Ne jamais redémarrer ce service en production.
```

De même, les négations doivent absolument être conservées.

Comparer :

```text
Le service peut être redémarré.
Le service ne peut pas être redémarré.
```

Une normalisation mal faite peut inverser le sens.

---

## 4.7. Étape 4 : enrichissement par métadonnées

Un chunk ne doit pas être stocké seul.

Nous devons l’accompagner de métadonnées.

Exemples de métadonnées utiles :

```text
id du document
titre du document
source
URL
chemin du fichier
section
numéro de page
date de création
date de modification
version
auteur
type de document
langue
droits d’accès
tags métier
environnement concerné
```

Ces métadonnées servent à plusieurs choses :

```text
citer les sources ;
filtrer les résultats ;
respecter les droits ;
favoriser les documents récents ;
déboguer le retrieval ;
évaluer le système ;
reconstruire le contexte.
```

Exemple :

```json
{
  "document_id": "runbook-checkout-prod",
  "title": "Runbook Checkout Production",
  "section": "Redémarrage du service",
  "environment": "production",
  "updated_at": "2026-05-12",
  "permissions": ["sre", "team-checkout"]
}
```

Sans métadonnées, le système peut retrouver le bon texte mais être incapable de dire d’où il vient.

---

## 4.8. Étape 5 : découpage en chunks

Le **chunking** consiste à découper les documents en morceaux plus petits.

Pourquoi découper ?

Parce qu’un document entier est souvent trop long pour être injecté dans le prompt.

Parce qu’un embedding de document entier dilue l’information.

Parce qu’une recherche vectorielle fonctionne mieux sur des unités de sens relativement ciblées.

Exemple :

```text
Document complet : 30 pages
        ↓
chunks de 300 à 800 tokens
        ↓
chaque chunk est indexé séparément
```

Le chunking est une des étapes les plus importantes d’un RAG.

Un mauvais chunking peut rendre le système médiocre, même avec un excellent LLM.

---

## 4.9. Stratégies de chunking

Il existe plusieurs stratégies.

### 4.9.1. Chunking à taille fixe

On découpe le texte tous les N tokens ou caractères.

Exemple :

```text
chunk_size = 500 tokens
overlap = 50 tokens
```

Avantage :

```text
simple à implémenter
prévisible
rapide
```

Limite :

```text
peut couper une idée au milieu
peut séparer un titre de son contenu
peut casser une procédure
```

### 4.9.2. Chunking par paragraphes

On découpe selon les paragraphes.

Avantage :

```text
respecte mieux la structure naturelle du texte
```

Limite :

```text
certains paragraphes sont trop courts
certains paragraphes sont trop longs
```

### 4.9.3. Chunking par sections

On utilise les titres pour découper.

Exemple :

```markdown
## Installation
## Configuration
## Déploiement
## Dépannage
```

Avantage :

```text
très adapté aux documentations structurées
```

Limite :

```text
nécessite des documents bien structurés
```

### 4.9.4. Chunking avec overlap

L’overlap consiste à répéter une partie du chunk précédent dans le chunk suivant.

Exemple :

```text
chunk 1 : tokens 0 à 500
chunk 2 : tokens 450 à 950
chunk 3 : tokens 900 à 1400
```

L’objectif est d’éviter de perdre une information située à la frontière entre deux chunks.

Avantage :

```text
réduit le risque de couper une idée importante
```

Limite :

```text
augmente le volume indexé
peut créer des résultats redondants
```

### 4.9.5. Chunking hiérarchique

On conserve plusieurs niveaux :

```text
document
section
sous-section
paragraphe
chunk
```

Cela permet de récupérer un chunk précis, puis éventuellement de remonter à sa section ou à son document parent.

C’est souvent une stratégie robuste pour des systèmes avancés.

---

## 4.10. Quelle taille de chunk choisir ?

Il n’existe pas de taille universelle.

Un bon chunk doit être :

```text
assez petit pour être spécifique ;
assez grand pour garder le contexte ;
assez cohérent pour être compréhensible seul.
```

Si le chunk est trop petit :

```text
il manque du contexte ;
il peut devenir ambigu ;
il peut ne pas contenir la réponse complète.
```

Exemple trop petit :

```text
Il doit être validé par le responsable.
```

Sans contexte, nous ne savons pas ce que “il” désigne.

Si le chunk est trop grand :

```text
il contient trop de sujets différents ;
son embedding devient moins précis ;
il ajoute du bruit dans le prompt ;
il consomme trop de tokens.
```

En pratique, pour du texte documentaire, on rencontre souvent des tailles de quelques centaines de tokens, mais cela dépend fortement du corpus.

Le bon choix se fait par expérimentation et évaluation.

---

## 4.11. Ajouter du contexte aux chunks

Un chunk isolé peut perdre son contexte.

Exemple :

```text
Le service ne doit pas être redémarré pendant cette période.
```

Quel service ? Quelle période ?

Pour éviter cela, nous pouvons enrichir le chunk avec son contexte hiérarchique :

```text
Document : Runbook Checkout Production
Section : Maintenance du service checkout
Contenu :
Le service ne doit pas être redémarré pendant cette période.
```

Le texte indexé peut alors devenir :

```text
Runbook Checkout Production — Maintenance du service checkout.
Le service ne doit pas être redémarré pendant cette période.
```

Cela améliore souvent le retrieval.

Mais il faut doser : ajouter trop de contexte répétitif peut aussi polluer les embeddings.

---

## 4.12. Étape 6 : calcul des embeddings

Une fois les chunks créés, nous calculons un embedding pour chacun.

Pipeline :

```text
chunk texte
        ↓
modèle d’embedding
        ↓
vecteur
```

Exemple :

```text
"La procédure de réinitialisation du mot de passe nécessite une validation sécurité."
        ↓
[0.12, -0.44, 0.89, ..., 0.05]
```

Tous les chunks doivent être vectorisés avec le même modèle d’embedding.

Si nous changeons de modèle, nous devons généralement recalculer tous les embeddings.

Il ne faut pas mélanger dans le même index des vecteurs produits par des modèles différents, car ils ne vivent pas dans le même espace vectoriel.

---

## 4.13. Étape 7 : stockage dans l’index

Chaque chunk est stocké dans un index.

Un enregistrement typique contient :

```json
{
  "chunk_id": "doc42_chunk_007",
  "text": "La procédure de réinitialisation du mot de passe nécessite une validation sécurité.",
  "embedding": [0.12, -0.44, 0.89],
  "metadata": {
    "document_id": "doc42",
    "title": "Guide sécurité",
    "section": "Réinitialisation des mots de passe",
    "page": 4,
    "updated_at": "2026-05-20"
  }
}
```

L’index permet ensuite de chercher rapidement les chunks dont l’embedding est proche de celui de la question.

Selon les architectures, cet index peut être :

```text
une base vectorielle dédiée ;
une extension vectorielle dans une base SQL ;
un moteur de recherche hybride ;
un index local ;
un service cloud.
```

Le choix technique dépend du volume, de la latence attendue, des contraintes de sécurité et de l’écosystème existant.

---

## 4.14. Phase de requête : répondre à une question

Une fois l’index construit, nous pouvons traiter les questions utilisateurs.

La phase de requête commence par une question :

```text
Comment réinitialiser un mot de passe administrateur ?
```

Le système doit :

```text
vectoriser la question ;
chercher les chunks proches ;
filtrer les résultats ;
construire un prompt ;
appeler le LLM ;
retourner une réponse.
```

---

## 4.15. Étape 1 : embedding de la question

La question est transformée en embedding avec le même modèle que les chunks.

```text
question
        ↓
modèle d’embedding
        ↓
vecteur de requête
```

Exemple :

```text
"Comment réinitialiser un mot de passe administrateur ?"
        ↓
[0.18, -0.31, 0.76, ..., 0.02]
```

Nous insistons : le modèle utilisé pour la question doit être le même que celui utilisé pour les documents.

---

## 4.16. Étape 2 : retrieval

Le retriever compare l’embedding de la question aux embeddings des chunks.

Il récupère les chunks les plus proches.

Exemple :

```text
Question :
Comment réinitialiser un mot de passe administrateur ?

Résultats :
1. Procédure de réinitialisation du mot de passe administrateur — score 0.91
2. Politique de sécurité des comptes privilégiés — score 0.83
3. Guide utilisateur sur la connexion — score 0.76
```

Ces résultats deviennent le contexte documentaire potentiel.

---

## 4.17. Étape 3 : filtres et métadonnées

Avant d’envoyer les chunks au LLM, nous pouvons appliquer des filtres.

Exemples :

```text
ne garder que les documents récents ;
ne garder que les documents validés ;
ne garder que les documents accessibles à l’utilisateur ;
ne garder que les documents de production ;
ne garder que les documents dans la bonne langue.
```

Exemple :

```text
Question :
Comment redémarrer checkout en production ?

Filtres :
environment = production
type = runbook
permissions incluent l’utilisateur
```

Cela évite des erreurs graves, comme répondre avec une procédure de test pour un environnement de production.

---

## 4.18. Étape 4 : sélection du contexte

Nous devons choisir quels chunks seront injectés dans le prompt.

Nous ne pouvons pas toujours injecter tous les résultats, car le contexte du LLM est limité.

Nous devons donc arbitrer entre :

```text
quantité d’information ;
pertinence ;
redondance ;
coût ;
taille du prompt ;
clarté du contexte.
```

Un bon contexte doit être :

```text
suffisant pour répondre ;
pas trop long ;
pas trop redondant ;
traçable ;
cohérent.
```

Si plusieurs chunks sont très proches mais identiques, il peut être utile de dédupliquer.

Si un chunk dépend de son paragraphe précédent, il peut être utile de récupérer aussi le voisinage documentaire.

---

## 4.19. Étape 5 : construction du prompt augmenté

Le prompt augmenté est le prompt envoyé au LLM avec les passages récupérés.

Exemple simple :

```text
Tu es un assistant documentaire.

Réponds à la question uniquement à partir du contexte fourni.
Si le contexte ne permet pas de répondre, dis :
"Je ne trouve pas cette information dans les documents fournis."

Contexte :
[Source 1]
La procédure de réinitialisation du mot de passe administrateur nécessite
une validation par le responsable sécurité.

[Source 2]
Après validation, l’administrateur peut utiliser le script reset-admin-password.sh.

Question :
Comment réinitialiser un mot de passe administrateur ?
```

Le LLM peut alors répondre :

```text
Pour réinitialiser un mot de passe administrateur, nous devons d’abord obtenir
une validation du responsable sécurité. Une fois cette validation obtenue,
l’administrateur peut utiliser le script reset-admin-password.sh.
```

Le prompt est essentiel.

Il doit contraindre le modèle à utiliser les sources, à ne pas inventer et à signaler les informations manquantes.

---

## 4.20. Étape 6 : génération de la réponse

Le LLM reçoit :

```text
la question ;
le contexte documentaire ;
les consignes de réponse.
```

Il génère ensuite une réponse.

Mais le LLM ne doit pas être considéré comme une source. La source est le document récupéré.

Le LLM est un moteur de formulation et de synthèse.

C’est une distinction importante :

```text
Les documents apportent l’information.
Le LLM formule la réponse.
```

Si le contexte est insuffisant, le modèle doit le dire.

---

## 4.21. Étape 7 : réponse sourcée

Dans un RAG professionnel, la réponse doit idéalement inclure des sources.

Exemple :

```text
Pour réinitialiser un mot de passe administrateur, nous devons d’abord obtenir
une validation du responsable sécurité. Après validation, le script
reset-admin-password.sh peut être utilisé.

Sources :
- Guide sécurité, section "Réinitialisation des comptes administrateurs"
- Runbook comptes privilégiés, page 4
```

Les citations permettent :

```text
de vérifier la réponse ;
de corriger les erreurs ;
de renforcer la confiance ;
d’auditer le système ;
de retrouver le document original.
```

Une réponse sans source peut être utile, mais elle est moins fiable dans un contexte professionnel.

---

## 4.22. Cas où le contexte ne permet pas de répondre

Un bon RAG doit savoir ne pas répondre.

Exemple :

```text
Question :
Quel est le numéro de téléphone du responsable sécurité ?
```

Si les documents récupérés ne contiennent pas cette information, le système doit répondre :

```text
Je ne trouve pas cette information dans les documents disponibles.
```

Il ne doit pas inventer un numéro.

Pour cela, le prompt peut contenir une instruction stricte :

```text
Si le contexte ne contient pas la réponse, indique explicitement que l’information
n’est pas disponible dans les documents fournis.
```

Mais l’instruction ne suffit pas toujours. Il faut aussi évaluer le comportement du modèle.

---

## 4.23. Exemple complet de bout en bout

Supposons que nous ayons trois documents.

```text
Doc 1 :
La procédure de réinitialisation d’un mot de passe administrateur nécessite
une validation par le responsable sécurité.

Doc 2 :
Après validation, l’administrateur peut utiliser le script
reset-admin-password.sh.

Doc 3 :
Les factures sont payables sous trente jours.
```

### 4.23.1. Phase d’indexation

Nous découpons les documents :

```text
chunk_1 = Doc 1
chunk_2 = Doc 2
chunk_3 = Doc 3
```

Nous calculons les embeddings :

```text
embedding(chunk_1)
embedding(chunk_2)
embedding(chunk_3)
```

Nous stockons :

```text
chunk_id
texte
embedding
métadonnées
```

### 4.23.2. Phase de requête

Question :

```text
Comment réinitialiser un mot de passe administrateur ?
```

Nous calculons :

```text
embedding(question)
```

Le retriever retourne :

```text
chunk_1 : score 0.91
chunk_2 : score 0.84
chunk_3 : score 0.21
```

Nous gardons `chunk_1` et `chunk_2`.

Nous construisons le prompt :

```text
Contexte :
[chunk_1]
La procédure de réinitialisation d’un mot de passe administrateur nécessite
une validation par le responsable sécurité.

[chunk_2]
Après validation, l’administrateur peut utiliser le script
reset-admin-password.sh.

Question :
Comment réinitialiser un mot de passe administrateur ?
```

Réponse attendue :

```text
Pour réinitialiser un mot de passe administrateur, nous devons d’abord obtenir
une validation du responsable sécurité. Après cette validation, l’administrateur
peut utiliser le script reset-admin-password.sh.
```

---

## 4.24. Exemple d’échec : mauvais retrieval

Prenons maintenant une question :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Documents :

```text
Doc 1 :
Le service checkout utilise l’API payments.

Doc 2 :
L’API payments est déployée sur le cluster-3.

Doc 3 :
Le cluster-3 sera en maintenance vendredi soir.

Doc 4 :
Le service checkout expose une API REST pour le panier.
```

Si le retriever récupère seulement :

```text
Doc 1
Doc 4
```

le LLM sait que checkout existe et qu’il utilise payments, mais il ne sait pas que payments tourne sur cluster-3, ni que cluster-3 est en maintenance.

Il ne peut pas répondre correctement.

Si le retriever récupère :

```text
Doc 1
Doc 2
Doc 3
```

alors le LLM peut conclure :

```text
Oui, le service checkout peut être affecté, car il dépend de l’API payments,
qui est déployée sur cluster-3, et cluster-3 est en maintenance vendredi.
```

Cet exemple montre que la qualité du RAG dépend d’abord de la qualité du retrieval.

---

## 4.25. Les paramètres principaux d’un RAG standard

Plusieurs paramètres influencent fortement le comportement du système.

### 4.25.1. Taille des chunks

```text
chunk_size
```

Si les chunks sont trop petits, ils manquent de contexte.

S’ils sont trop grands, ils ajoutent du bruit.

### 4.25.2. Overlap

```text
chunk_overlap
```

L’overlap réduit le risque de couper une information importante à la frontière entre deux chunks.

Mais il augmente la redondance.

### 4.25.3. Nombre de résultats récupérés

```text
top_k
```

Un top-k trop faible peut rater des informations.

Un top-k trop élevé peut noyer le LLM dans du bruit.

### 4.25.4. Seuil de similarité

```text
score_threshold
```

Il permet d’écarter des résultats trop éloignés, mais il doit être calibré.

### 4.25.5. Modèle d’embedding

Le choix du modèle d’embedding influence directement la qualité du retrieval.

### 4.25.6. Modèle génératif

Le choix du LLM influence la qualité de la synthèse, du raisonnement et du respect des consignes.

### 4.25.7. Prompt système

Le prompt détermine en partie le comportement du modèle :

```text
répondre uniquement à partir des sources ;
citer les documents ;
signaler les informations manquantes ;
éviter les hypothèses ;
structurer la réponse.
```

---

## 4.26. Les erreurs classiques lors de la construction d’un RAG

### 4.26.1. Indexer des documents mal extraits

Si le texte extrait est mauvais, les embeddings seront mauvais.

Exemple :

```text
Tableau PDF extrait dans le mauvais ordre.
```

Le RAG ne pourra pas reconstruire correctement l’information.

### 4.26.2. Découper sans respecter la structure

Un chunking aveugle peut séparer une question de sa réponse, un titre de son contenu ou une condition de sa conséquence.

### 4.26.3. Oublier les métadonnées

Sans métadonnées, il devient difficile de citer, filtrer, auditer ou gérer les versions.

### 4.26.4. Utiliser uniquement la recherche vectorielle

La recherche vectorielle peut rater les identifiants exacts, les codes d’erreur ou les noms de fichiers.

### 4.26.5. Mettre trop de contexte

Injecter trop de chunks peut dégrader la réponse.

Le LLM peut se perdre, mélanger les sources ou accorder de l’importance à un passage secondaire.

### 4.26.6. Ne pas gérer les droits

Un RAG sans contrôle d’accès peut exposer des informations confidentielles.

### 4.26.7. Ne pas évaluer

Sans jeu de tests, nous ne savons pas si le système fonctionne réellement.

Un RAG peut sembler impressionnant sur quelques démos, mais échouer sur les cas critiques.

---

## 4.27. Pseudo-code d’un RAG standard

Voici une version simplifiée.

### Indexation

```python
documents = load_documents()

chunks = []

for document in documents:
    text = extract_text(document)
    clean = clean_text(text)
    doc_chunks = split_into_chunks(clean)

    for chunk in doc_chunks:
        embedding = embedding_model.encode(chunk.text)

        vector_index.add(
            id=chunk.id,
            vector=embedding,
            text=chunk.text,
            metadata={
                "source": document.source,
                "title": document.title,
                "section": chunk.section,
                "updated_at": document.updated_at
            }
        )
```

### Requête

```python
def answer_question(question, user):
    question_embedding = embedding_model.encode(question)

    results = vector_index.search(
        vector=question_embedding,
        top_k=5,
        filters={
            "permissions": user.permissions
        }
    )

    context = build_context(results)

    prompt = f"""
    Réponds à la question uniquement avec le contexte fourni.
    Si le contexte ne permet pas de répondre, dis-le.

    Contexte :
    {context}

    Question :
    {question}
    """

    answer = llm.generate(prompt)

    return {
        "answer": answer,
        "sources": [result.metadata for result in results]
    }
```

Ce pseudo-code est volontairement simplifié, mais il montre la structure générale.

---

## 4.28. Variante avec recherche hybride

Dans un système plus robuste, nous pouvons combiner recherche vectorielle et recherche lexicale.

```text
Question
   ↓
Recherche vectorielle
   ↓
Recherche BM25
   ↓
Fusion des résultats
   ↓
Reranking
   ↓
Prompt
   ↓
LLM
```

Cela permet de mieux gérer à la fois :

```text
les reformulations ;
les synonymes ;
les noms exacts ;
les codes techniques ;
les références rares.
```

Nous étudierons cela dans un chapitre d’optimisation.

---

## 4.29. Mini-TP : construire un RAG minimal

### Objectif

Nous voulons construire un petit RAG sur un corpus très simple.

### Corpus

```python
documents = [
    {
        "id": "doc1",
        "text": "La procédure de réinitialisation du mot de passe nécessite une validation sécurité."
    },
    {
        "id": "doc2",
        "text": "Après validation, l’administrateur peut utiliser le script reset-admin-password.sh."
    },
    {
        "id": "doc3",
        "text": "Les factures sont payables sous trente jours après émission."
    },
    {
        "id": "doc4",
        "text": "Le service checkout utilise l’API payments."
    },
    {
        "id": "doc5",
        "text": "L’API payments est déployée sur le cluster-3."
    },
    {
        "id": "doc6",
        "text": "Le cluster-3 sera en maintenance vendredi soir."
    }
]
```

### Questions à tester

```text
Comment réinitialiser un mot de passe administrateur ?
Quand faut-il payer une facture ?
Le service checkout sera-t-il affecté vendredi ?
```

### Travail demandé

Nous devons :

```text
1. Vectoriser les documents.
2. Vectoriser chaque question.
3. Récupérer les top-k documents.
4. Construire un prompt avec les documents récupérés.
5. Générer une réponse.
6. Observer les cas de réussite et d’échec.
```

### Analyse attendue

La question sur le mot de passe devrait fonctionner avec les documents `doc1` et `doc2`.

La question sur les factures devrait fonctionner avec `doc3`.

La question sur checkout est plus intéressante, car elle nécessite :

```text
doc4
doc5
doc6
```

Si le top-k est trop faible, le système peut ne pas récupérer tous les documents nécessaires.

---

## 4.30. Questions de compréhension

À la fin de ce chapitre, nous devons pouvoir répondre aux questions suivantes :

```text
Quelles sont les deux grandes phases d’un RAG ?
Pourquoi l’extraction documentaire est-elle critique ?
Pourquoi le chunking influence-t-il fortement la qualité du RAG ?
À quoi servent les métadonnées ?
Pourquoi faut-il utiliser le même modèle d’embedding pour les documents et les questions ?
Qu’est-ce que le top-k ?
Pourquoi un RAG doit-il parfois refuser de répondre ?
Pourquoi les sources sont-elles importantes ?
Quelles sont les erreurs fréquentes dans un RAG standard ?
Pourquoi la recherche vectorielle seule peut-elle être insuffisante ?
```

---

## 4.31. Synthèse du chapitre

Dans ce chapitre, nous avons construit mentalement un RAG standard complet.

Nous avons vu qu’un système RAG repose sur deux phases :

```text
indexation ;
requête.
```

Pendant l’indexation, nous transformons les documents bruts en chunks vectorisés.

Pendant la requête, nous transformons la question en embedding, nous retrouvons les chunks proches, puis nous les donnons au LLM pour générer une réponse.

Nous avons compris que la qualité du RAG dépend de nombreux choix :

```text
qualité des sources ;
extraction du texte ;
nettoyage ;
chunking ;
métadonnées ;
modèle d’embedding ;
index vectoriel ;
retrieval ;
top-k ;
prompt ;
modèle génératif ;
citations ;
gestion des droits.
```

Le message principal du chapitre est le suivant :

```text
Un RAG standard n’est pas seulement une recherche vectorielle suivie d’un appel LLM.
C’est une chaîne complète de traitement documentaire, de récupération,
de sélection et de génération contrôlée.
```

Dans le chapitre suivant, nous étudierons plus précisément le **prompt augmenté et la génération sourcée**, c’est-à-dire la manière dont nous transmettons les documents récupérés au LLM pour obtenir une réponse fiable, vérifiable et utile.

---

---
> [!info] Livre « RAG » — chapitre 4/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 03 — Similarité cosinus et recherche vectorielle|← 03 — Similarité cosinus et recherche vectorielle]] · [[RAG — 05 — Prompt augmenté et génération sourcée|05 — Prompt augmenté et génération sourcée →]]
