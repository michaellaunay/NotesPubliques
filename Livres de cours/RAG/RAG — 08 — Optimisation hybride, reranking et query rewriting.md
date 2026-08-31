---
schema_version: 1
uid: 01M1BQ627B226WCMXWVCCG7BA0
titre: "RAG — 08 — Optimisation hybride, reranking et query rewriting"
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
resume: "Chapitre 8 sur 12 du livre « RAG » : Optimisation : hybride, reranking et query rewriting. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « RAG » — chapitre 8/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 07 — Évaluer un système RAG|← 07 — Évaluer un système RAG]] · [[RAG — 09 — Graph RAG entités, relations et requêtes multi-hop|09 — Graph RAG entités, relations et requêtes multi-hop →]]

# Chapitre 8 — Optimisation : hybride, reranking et query rewriting
## 8.1. Introduction

Dans les chapitres précédents, nous avons construit un RAG standard, puis nous avons appris à évaluer sa qualité.

Nous avons vu que le système peut échouer à plusieurs endroits :

```text
mauvais documents récupérés ;
documents utiles récupérés trop bas dans le classement ;
documents obsolètes ;
chunks trop petits ;
chunks trop grands ;
contexte trop bruité ;
réponse générée trop confiante ;
sources mal citées.
```

Dans ce chapitre, nous allons nous concentrer sur les optimisations qui permettent d’améliorer le retrieval et la qualité du contexte donné au LLM.

Nous allons étudier plusieurs techniques importantes :

```text
recherche hybride ;
fusion des résultats ;
reranking ;
query rewriting ;
query expansion ;
multi-query retrieval ;
HyDE ;
contextual retrieval ;
parent-child retrieval ;
optimisation par métadonnées ;
choix du modèle d’embedding ;
optimisation du chunking.
```

L’objectif n’est pas de tout utiliser systématiquement.

L’objectif est de comprendre quel problème chaque technique résout, et dans quels cas elle devient utile.

---

## 8.2. Pourquoi optimiser un RAG ?

Un RAG naïf peut fonctionner sur une démonstration simple.

Exemple :

```text
Question :
Quel est le délai de paiement ?

Document :
Les factures sont payables sous trente jours.
```

Une recherche vectorielle simple suffit probablement.

Mais dans un vrai corpus, nous avons souvent :

```text
des documents longs ;
des doublons ;
des versions obsolètes ;
des PDF mal extraits ;
des termes techniques ;
des identifiants exacts ;
des noms de fichiers ;
des questions ambiguës ;
des informations réparties sur plusieurs documents ;
des sources contradictoires ;
des droits d’accès ;
des documents multilingues.
```

Dans ce contexte, un simple pipeline :

```text
embedding(question) → top_k chunks → LLM
```

est souvent insuffisant.

Optimiser un RAG consiste donc à améliorer :

```text
la probabilité de retrouver les bons passages ;
le classement des passages ;
la qualité du contexte final ;
la robustesse aux questions mal formulées ;
la capacité à gérer les termes exacts ;
la capacité à réduire le bruit ;
la capacité à éviter les hallucinations.
```

---

## 8.3. Les trois grands leviers d’optimisation

Nous pouvons classer les optimisations en trois grandes familles.

### 8.3.1. Optimiser la requête

Nous modifions ou enrichissons la question avant la recherche.

Exemples :

```text
query rewriting ;
query expansion ;
multi-query retrieval ;
HyDE.
```

Objectif :

```text
formuler une meilleure requête pour le moteur de recherche.
```

### 8.3.2. Optimiser la recherche

Nous améliorons la manière dont les documents sont récupérés.

Exemples :

```text
recherche hybride ;
filtres par métadonnées ;
parent-child retrieval ;
contextual retrieval ;
retrieval multi-hop.
```

Objectif :

```text
récupérer des candidats plus pertinents.
```

### 8.3.3. Optimiser le classement

Nous améliorons l’ordre des résultats avant de construire le prompt.

Exemples :

```text
reranking ;
fusion de scores ;
déduplication ;
diversification.
```

Objectif :

```text
donner au LLM les meilleurs passages dans le meilleur ordre.
```

---

## 8.4. Recherche vectorielle seule : rappel des limites

La recherche vectorielle retrouve les passages proches en sens.

Elle est très utile lorsque l’utilisateur formule une question en langage naturel.

Exemple :

```text
Question :
Je n’arrive plus à me connecter à mon compte.

Document :
Procédure de réinitialisation du mot de passe utilisateur.
```

La recherche vectorielle peut rapprocher ces deux textes.

Mais elle a plusieurs limites.

### 8.4.1. Elle peut rater les termes exacts

Exemple :

```text
ERR_MODULE_NOT_FOUND
P2025
docker-compose.prod.yml
Article L.221-1
```

Ces termes doivent souvent être retrouvés exactement.

### 8.4.2. Elle peut récupérer des passages proches mais inutiles

Question :

```text
Comment corriger Prisma P2025 ?
```

Chunk récupéré :

```text
Prisma est un ORM moderne pour Node.js et TypeScript.
```

Le chunk est proche du sujet, mais il ne répond pas à la question.

### 8.4.3. Elle peut rater les faits intermédiaires

Question :

```text
Le service checkout sera-t-il affecté vendredi ?
```

Faits nécessaires :

```text
checkout utilise payments API ;
payments API tourne sur cluster-3 ;
cluster-3 est en maintenance vendredi.
```

Le fait intermédiaire peut être peu proche de la question.

### 8.4.4. Elle dépend fortement du chunking

Si l’information est coupée entre plusieurs chunks, le retrieval peut échouer.

---

## 8.5. Recherche lexicale

La recherche lexicale repose sur les mots exacts de la requête.

Elle reste très importante dans un RAG.

Elle est particulièrement utile pour :

```text
noms de fichiers ;
identifiants ;
codes d’erreur ;
noms de fonctions ;
classes ;
références juridiques ;
numéros ;
acronymes ;
termes rares.
```

Exemple :

```text
Question :
Pourquoi ai-je ERR_MODULE_NOT_FOUND après le build ?
```

Une recherche lexicale retrouve directement les documents contenant :

```text
ERR_MODULE_NOT_FOUND
```

Même si l’embedding du modèle ne représente pas bien ce terme.

### 8.5.1. BM25

BM25 est un algorithme classique de recherche lexicale.

Il favorise les documents qui contiennent les termes de la requête, en tenant compte :

```text
de la fréquence du terme dans le document ;
de la rareté du terme dans le corpus ;
de la longueur du document.
```

BM25 est souvent très fort pour les termes exacts.

### 8.5.2. Limite de la recherche lexicale

La recherche lexicale échoue lorsque les mots sont différents mais le sens proche.

Exemple :

```text
Question :
Je n’arrive plus à me connecter.

Document :
Réinitialisation du mot de passe utilisateur.
```

Le mot “connecter” ne correspond pas forcément à “réinitialisation du mot de passe”.

C’est pourquoi nous combinons souvent lexical et vectoriel.

---

## 8.6. Recherche hybride

La recherche hybride combine :

```text
recherche vectorielle ;
recherche lexicale.
```

L’idée est de bénéficier des forces des deux approches.

```text
Vectoriel → reformulation, synonymes, proximité conceptuelle.
Lexical → termes exacts, codes, identifiants, noms propres.
```

Pipeline :

```text
Question
   ↓
Recherche vectorielle
   ↓
Résultats A

Question
   ↓
Recherche lexicale
   ↓
Résultats B

Fusion A + B
   ↓
Candidats
   ↓
Reranking éventuel
   ↓
Contexte final
```

### 8.6.1. Exemple

Question :

```text
Pourquoi mon application TypeScript échoue avec ERR_MODULE_NOT_FOUND ?
```

La recherche lexicale retrouve :

```text
ERR_MODULE_NOT_FOUND ;
TypeScript ;
build.
```

La recherche vectorielle retrouve :

```text
imports cassés ;
extensions .js manquantes ;
ESM ;
chemins relatifs après compilation ;
tsconfig ;
répertoire dist.
```

La combinaison donne un meilleur ensemble de candidats.

---

## 8.7. Fusion des résultats

Fusionner recherche vectorielle et lexicale n’est pas trivial.

Les scores ne sont pas comparables directement.

Un score BM25 et un score cosinus n’ont pas la même signification.

Nous avons plusieurs stratégies.

### 8.7.1. Union simple

Nous prenons les meilleurs résultats de chaque moteur.

Exemple :

```text
top 10 vectoriel
+
top 10 lexical
=
jusqu’à 20 candidats
```

Puis nous supprimons les doublons.

Avantage :

```text
simple ;
augmente le rappel.
```

Limite :

```text
ne classe pas finement ;
peut garder beaucoup de bruit.
```

### 8.7.2. Pondération des scores

Nous normalisons les scores, puis nous calculons :

```text
score_final = α × score_vectoriel + β × score_lexical
```

Exemple :

```text
score_final = 0.6 × score_vectoriel + 0.4 × score_lexical
```

Avantage :

```text
permet d’ajuster l’importance des deux moteurs.
```

Limite :

```text
nécessite calibration ;
dépend fortement du corpus.
```

### 8.7.3. Reciprocal Rank Fusion

La **Reciprocal Rank Fusion**, souvent abrégée RRF, fusionne les classements plutôt que les scores.

Elle donne un bonus aux documents bien classés dans chaque liste.

L’idée simplifiée est :

```text
un document classé haut par plusieurs méthodes doit être favorisé.
```

Avantage :

```text
robuste ;
évite de comparer directement des scores incompatibles.
```

Limite :

```text
ne remplace pas un vrai reranker si la précision doit être fine.
```

---

## 8.8. Reranking

Le reranking est une des optimisations les plus importantes.

Le principe :

```text
1. Nous récupérons largement des candidats.
2. Nous les reclassons plus finement.
```

Pipeline :

```text
Question
   ↓
Retriever rapide
   ↓
30 à 100 chunks candidats
   ↓
Reranker
   ↓
5 à 10 chunks finaux
   ↓
LLM
```

### 8.8.1. Pourquoi le reranking est utile

Un retriever vectoriel calcule une proximité globale.

Un reranker peut comparer plus précisément :

```text
la question complète ;
le passage candidat ;
la relation entre les deux.
```

Il peut distinguer :

```text
un passage qui parle du même sujet ;
un passage qui répond réellement à la question.
```

Exemple :

```text
Question :
Comment corriger l’erreur Prisma P2025 ?

Chunk A :
Prisma est un ORM moderne pour TypeScript.

Chunk B :
L’erreur P2025 survient quand un enregistrement attendu n’est pas trouvé.

Chunk C :
Pour corriger P2025 dans notre projet, vérifier d’abord l’existence de l’id.
```

Le retriever peut trouver A, B et C.

Le reranker doit classer C et B avant A.

### 8.8.2. Cross-encoder vs bi-encoder

Les embeddings classiques sont souvent produits par un bi-encoder.

Principe :

```text
question → vecteur
document → vecteur
similarité(question, document)
```

C’est rapide, car les documents sont pré-encodés.

Un cross-encoder, utilisé comme reranker, lit la question et le document ensemble.

Principe :

```text
(question, document) → score de pertinence
```

Il est souvent plus précis, mais plus coûteux.

Nous utilisons donc généralement :

```text
bi-encoder pour récupérer rapidement ;
cross-encoder pour reranker les candidats.
```

---

## 8.9. Query rewriting

Le query rewriting consiste à reformuler la question utilisateur avant la recherche.

Les utilisateurs ne formulent pas toujours leurs questions de façon optimale.

Exemple :

```text
Ça casse vendredi ?
```

Cette question est compréhensible dans un contexte conversationnel, mais mauvaise pour un moteur de recherche.

Le système peut la reformuler en :

```text
Quels services seront impactés par la maintenance prévue vendredi ?
```

### 8.9.1. Objectif

Le but n’est pas de changer le sens.

Le but est de rendre la requête plus explicite.

Exemples :

```text
"Je ne peux plus me connecter"
→ "problème de connexion compte utilisateur réinitialisation mot de passe"

"Le build casse en prod"
→ "erreur de build en environnement production logs déploiement"

"Ça impacte checkout ?"
→ "impact sur le service checkout dépendances infrastructure maintenance"
```

### 8.9.2. Risques

Le rewriting peut déformer la question.

Exemple :

```text
Question originale :
Le service sera-t-il ralenti ?

Mauvaise reformulation :
Le service sera-t-il indisponible ?
```

“Ralenti” et “indisponible” ne sont pas équivalents.

Nous devons préserver :

```text
les entités ;
la temporalité ;
le niveau d’incertitude ;
les contraintes ;
la portée ;
le type de réponse attendu.
```

### 8.9.3. Bon prompt de rewriting

Un prompt de rewriting peut être :

```text
Reformule la question pour améliorer la recherche documentaire.
Ne change pas le sens.
Conserve les noms propres, dates, codes d’erreur et contraintes.
Retourne uniquement la requête reformulée.
```

---

## 8.10. Query expansion

La query expansion ajoute des termes liés à la question.

Exemple :

```text
Question :
Comment changer mon mot de passe ?
```

Expansion :

```text
réinitialisation ;
identifiants ;
connexion ;
authentification ;
password ;
compte utilisateur.
```

Cela aide notamment la recherche lexicale.

### 8.10.1. Exemple technique

Question :

```text
Erreur module introuvable après build.
```

Expansion :

```text
ERR_MODULE_NOT_FOUND ;
module not found ;
import ;
ESM ;
CommonJS ;
TypeScript ;
tsconfig ;
dist ;
extension .js.
```

### 8.10.2. Risque

Une expansion trop large ajoute du bruit.

Exemple :

```text
connexion
```

peut renvoyer à :

```text
connexion utilisateur ;
connexion SSH ;
connexion base de données ;
connexion réseau ;
connexion OAuth.
```

Il faut donc garder une expansion contextualisée.

---

## 8.11. Multi-query retrieval

Le multi-query retrieval consiste à générer plusieurs requêtes différentes à partir d’une question.

Exemple :

Question :

```text
Le checkout sera-t-il affecté vendredi ?
```

Requêtes générées :

```text
dépendances du service checkout ;
infrastructure de l’API payments ;
services impactés par la maintenance de vendredi ;
cluster en maintenance vendredi checkout payments ;
```

Chaque requête explore un angle différent.

On fusionne ensuite les résultats.

### 8.11.1. Avantage

Le multi-query augmente le rappel.

Il est utile pour :

```text
questions larges ;
questions multi-hop ;
questions ambiguës ;
corpus hétérogènes ;
documents avec vocabulaires différents.
```

### 8.11.2. Limite

Il augmente :

```text
la latence ;
le coût ;
le bruit ;
la complexité de fusion.
```

Il faut donc l’utiliser quand la question le justifie.

---

## 8.12. HyDE : Hypothetical Document Embeddings

HyDE signifie **Hypothetical Document Embeddings**.

L’idée est originale :

```text
au lieu d’encoder directement la question,
nous demandons au LLM de générer une réponse hypothétique,
puis nous encodons cette réponse hypothétique pour rechercher des documents proches.
```

### 8.12.1. Exemple

Question :

```text
Pourquoi mon build TypeScript échoue avec un module introuvable ?
```

Réponse hypothétique générée :

```text
Le problème peut venir d’un import relatif qui ne contient pas l’extension .js
après compilation en ESM, ou d’un mauvais chemin dans le répertoire dist.
```

Nous encodons cette réponse hypothétique.

Puis nous cherchons des documents proches.

### 8.12.2. Pourquoi cela peut marcher

Une question est parfois courte ou vague.

Une réponse hypothétique contient plus de vocabulaire documentaire.

Elle peut être plus proche des passages pertinents.

### 8.12.3. Risque

Le document hypothétique peut être faux ou orienter la recherche dans une mauvaise direction.

HyDE peut améliorer le rappel, mais il faut l’évaluer soigneusement.

---

## 8.13. Contextual retrieval

Le contextual retrieval consiste à enrichir les chunks avant indexation.

Un chunk brut peut être ambigu :

```text
Il ne doit pas être redémarré pendant cette période.
```

Nous pouvons ajouter le contexte :

```text
Document : Runbook Checkout Production
Section : Maintenance du service checkout
Chunk :
Il ne doit pas être redémarré pendant cette période.
```

Le texte indexé devient plus compréhensible :

```text
Dans le Runbook Checkout Production, section Maintenance du service checkout :
le service checkout ne doit pas être redémarré pendant cette période.
```

### 8.13.1. Avantage

Le chunk devient plus autonome.

L’embedding représente mieux son sens.

### 8.13.2. Limite

Si nous ajoutons trop d’informations répétitives, nous polluons les embeddings.

Il faut enrichir avec les métadonnées utiles, pas recopier tout le document.

---

## 8.14. Parent-child retrieval

Le parent-child retrieval sépare :

```text
les petits chunks utilisés pour la recherche ;
les grands blocs utilisés pour la génération.
```

Principe :

```text
child chunk → indexé pour retrieval précis
parent chunk → fourni au LLM pour contexte large
```

### 8.14.1. Exemple

Document :

```text
Section : Redémarrage de checkout
- conditions préalables ;
- validation sécurité ;
- commande ;
- vérification post-redémarrage.
```

Nous indexons les paragraphes individuellement.

Si le paragraphe sur la commande est retrouvé, nous fournissons toute la section au LLM.

### 8.14.2. Avantage

Nous combinons :

```text
précision de recherche ;
richesse du contexte.
```

### 8.14.3. Limite

Le parent peut être trop long.

Il faut donc contrôler la taille du bloc parent.

---

## 8.15. Window retrieval

Le window retrieval consiste à récupérer un chunk ainsi que ses voisins.

Exemple :

```text
chunk_12 retrouvé
```

Nous ajoutons :

```text
chunk_11 ;
chunk_13.
```

C’est utile quand l’information est répartie localement.

### 8.15.1. Exemple

```text
chunk_11 :
Avant toute réinitialisation, une validation sécurité est obligatoire.

chunk_12 :
Après validation, exécuter reset-admin-password.sh.

chunk_13 :
L’opération doit être journalisée dans le registre des accès.
```

Si la question porte sur la procédure complète, récupérer seulement `chunk_12` est insuffisant.

Avec une fenêtre, nous obtenons les conditions et les suites.

### 8.15.2. Limite

La fenêtre augmente la taille du contexte.

Il faut éviter de récupérer trop large.

---

## 8.16. Diversification des résultats

Un retriever peut retourner plusieurs chunks redondants.

Exemple :

```text
chunk 1 : cluster-3 en maintenance vendredi ;
chunk 2 : maintenance vendredi sur cluster-3 ;
chunk 3 : vendredi soir maintenance cluster-3.
```

Ces trois chunks disent presque la même chose.

Pour une question multi-hop, nous préférons récupérer des informations complémentaires :

```text
checkout utilise payments API ;
payments API tourne sur cluster-3 ;
cluster-3 est en maintenance vendredi.
```

La diversification cherche à maximiser :

```text
pertinence ;
non-redondance ;
couverture des aspects de la question.
```

### 8.16.1. Stratégies

Nous pouvons diversifier par :

```text
document ;
section ;
entité ;
type de source ;
cluster de similarité ;
étape de raisonnement.
```

---

## 8.17. Déduplication

La déduplication supprime les résultats trop similaires.

Elle est utile lorsque :

```text
les documents ont été exportés plusieurs fois ;
plusieurs versions coexistent ;
l’overlap crée des chunks proches ;
les mêmes paragraphes apparaissent dans plusieurs pages ;
des documents sont copiés entre espaces.
```

Méthodes possibles :

```text
hash exact ;
similarité textuelle ;
similarité d’embedding ;
regroupement par document source ;
détection de versions.
```

Un bon RAG doit éviter de remplir le prompt avec dix variantes du même passage.

---

## 8.18. Optimisation des métadonnées

Les métadonnées ne servent pas seulement à citer.

Elles peuvent fortement améliorer le retrieval.

Exemples :

```text
service = checkout ;
environment = production ;
type = runbook ;
status = validé ;
date = 2026-05-12 ;
language = fr ;
permissions = team-sre ;
version = 3.2.
```

Une question comme :

```text
Comment redémarrer checkout en production ?
```

peut appliquer :

```text
service = checkout
environment = production
type = runbook
status = validé
```

Cela réduit le bruit et évite les documents obsolètes.

### 8.18.1. Métadonnées explicites vs implicites

Certaines métadonnées existent déjà :

```text
chemin du fichier ;
date de modification ;
auteur ;
URL.
```

D’autres peuvent être extraites :

```text
service concerné ;
environnement ;
type de procédure ;
entités mentionnées ;
niveau de confidentialité.
```

Nous pouvons utiliser un LLM ou des règles pour enrichir les documents avec ces métadonnées.

---

## 8.19. Optimisation du chunking

Le chunking influence fortement le retrieval.

### 8.19.1. Chunks trop petits

Problème :

```text
"Après validation, exécuter le script."
```

Ce chunk est incompréhensible seul.

### 8.19.2. Chunks trop grands

Problème :

```text
un chunk de 3000 tokens contenant installation, configuration, erreurs,
déploiement et monitoring.
```

L’embedding devient trop général.

### 8.19.3. Bon chunking

Un bon chunk doit être :

```text
sémantiquement cohérent ;
assez autonome ;
pas trop long ;
associé à des métadonnées ;
lié à son parent documentaire.
```

### 8.19.4. Chunking adapté au type de document

Pour du Markdown :

```text
découpage par titres et sous-titres.
```

Pour du code :

```text
découpage par fonction, classe ou fichier.
```

Pour un PDF administratif :

```text
découpage par section, page et tableau.
```

Pour une FAQ :

```text
découpage par paire question-réponse.
```

Le chunking ne doit pas être uniforme pour tous les corpus.

---

## 8.20. Optimisation du modèle d’embedding

Le choix du modèle d’embedding est un paramètre important.

Nous devons vérifier :

```text
langue du corpus ;
domaine technique ;
capacité multilingue ;
qualité sur les textes courts ;
qualité sur les textes longs ;
qualité sur le code ;
coût ;
latence ;
dimension des vecteurs ;
licence ;
possibilité d’hébergement local.
```

### 8.20.1. Modèle généraliste

Suffisant pour :

```text
FAQ ;
documentation simple ;
contenus généralistes ;
prototype.
```

### 8.20.2. Modèle spécialisé

Utile pour :

```text
code source ;
droit ;
biomédical ;
finance ;
documentation très technique ;
corpus multilingue.
```

### 8.20.3. Changement de modèle

Si nous changeons de modèle d’embedding, nous devons généralement réindexer tout le corpus.

Nous devons aussi refaire l’évaluation, car le classement des résultats peut changer.

---

## 8.21. Optimisation par type de question

Toutes les questions ne doivent pas utiliser le même pipeline.

Nous pouvons classer les questions.

### 8.21.1. Question factuelle simple

Exemple :

```text
Quel est le délai de paiement ?
```

Pipeline possible :

```text
recherche vectorielle simple ;
top_k faible ;
réponse courte.
```

### 8.21.2. Question avec identifiant exact

Exemple :

```text
Que signifie ERR_MODULE_NOT_FOUND ?
```

Pipeline possible :

```text
recherche lexicale prioritaire ;
hybride ;
reranking.
```

### 8.21.3. Question multi-hop

Exemple :

```text
Le checkout sera-t-il affecté vendredi ?
```

Pipeline possible :

```text
multi-query ;
retrieval large ;
reranking ;
diversification ;
éventuellement Graph RAG.
```

### 8.21.4. Question large

Exemple :

```text
Quels sont les risques de cette migration ?
```

Pipeline possible :

```text
retrieval large ;
regroupement par thèmes ;
synthèse structurée ;
citations multiples.
```

### 8.21.5. Question impossible

Exemple :

```text
Quel est le numéro personnel du responsable sécurité ?
```

Pipeline attendu :

```text
retrieval ;
vérification de suffisance ;
refus de répondre si absence de source.
```

Un système avancé peut donc router les questions vers différentes stratégies.

---

## 8.22. Optimisation du contexte final

Même après retrieval et reranking, nous devons construire un bon contexte final.

Critères :

```text
éviter les doublons ;
préserver les sources ;
ordonner les passages ;
garder les informations complémentaires ;
limiter la taille ;
inclure les métadonnées utiles ;
préserver les citations.
```

### 8.22.1. Ordre logique

Pour une question multi-hop :

```text
checkout utilise payments ;
payments tourne sur cluster-3 ;
cluster-3 est en maintenance.
```

Cet ordre aide le LLM à raisonner.

### 8.22.2. Ordre par autorité

Pour une question réglementaire :

```text
texte officiel ;
procédure interne ;
commentaire ;
cas pratique.
```

### 8.22.3. Ordre chronologique

Pour une question d’incident :

```text
événement initial ;
logs ;
correction ;
post-mortem.
```

L’ordre du contexte influence la réponse.

---

## 8.23. Optimisation contre les hallucinations

Les optimisations de retrieval réduisent les hallucinations en fournissant un meilleur contexte.

Mais il faut aussi agir sur la génération.

Stratégies :

```text
prompt strict ;
citations obligatoires ;
réponse "je ne sais pas" autorisée ;
séparation faits / déductions / limites ;
validation post-génération ;
détection des affirmations non sourcées.
```

Exemple de format utile :

```text
Faits établis par les sources :
- ...
Déduction :
- ...
Limites :
- ...
Sources :
- ...
```

Ce format réduit les conclusions trop fortes.

---

## 8.24. Optimisation coût / latence

Chaque optimisation a un coût.

Exemple :

```text
query rewriting → appel LLM supplémentaire ;
multi-query → plusieurs recherches ;
reranking → modèle supplémentaire ;
HyDE → génération + embedding ;
retrieval large → plus de documents à traiter ;
gros contexte → génération plus coûteuse.
```

Nous devons donc arbitrer.

### 8.24.1. Exemple de pipeline léger

Pour questions simples :

```text
vector search top_k=5
→ prompt
→ LLM
```

### 8.24.2. Exemple de pipeline robuste

Pour questions complexes :

```text
classification de la question
→ query rewriting
→ recherche hybride
→ top 50 candidats
→ reranking
→ diversification
→ contexte final
→ LLM
→ validation
```

Le pipeline robuste est meilleur, mais plus cher et plus lent.

### 8.24.3. Routage adaptatif

Une solution consiste à adapter le pipeline :

```text
question simple → pipeline rapide ;
question complexe → pipeline avancé ;
question sensible → pipeline avec validation ;
question impossible → vérification stricte de suffisance.
```

---

## 8.25. Exemple complet d’optimisation

Question :

```text
Le checkout sera-t-il impacté par la maintenance de vendredi ?
```

Corpus :

```text
Doc 1 :
Le service checkout utilise l’API payments.

Doc 2 :
L’API payments est déployée sur le cluster-3.

Doc 3 :
Le cluster-3 sera en maintenance vendredi soir.

Doc 4 :
Le service checkout expose une API REST pour les paniers.

Doc 5 :
Ancienne documentation : checkout était déployé sur cluster-1.

Doc 6 :
Le cluster-7 sera mis à jour lundi.
```

### 8.25.1. Recherche vectorielle naïve

Résultats :

```text
Doc 1
Doc 4
Doc 3
```

Problème :

```text
Doc 2 manque.
Doc 4 est peu utile.
```

### 8.25.2. Query rewriting

Question reformulée :

```text
Quels composants et dépendances du service checkout sont concernés par la maintenance de vendredi ?
```

Cela peut aider à retrouver les dépendances.

### 8.25.3. Multi-query

Requêtes :

```text
dépendances du service checkout ;
déploiement de payments API ;
maintenance vendredi cluster ;
impact checkout payments cluster.
```

Les résultats récupèrent :

```text
Doc 1
Doc 2
Doc 3
```

### 8.25.4. Métadonnées

Nous filtrons :

```text
status = actif
```

Doc 5 est exclu.

### 8.25.5. Reranking

Le reranker classe :

```text
Doc 1 : nécessaire
Doc 2 : nécessaire
Doc 3 : nécessaire
Doc 4 : secondaire
Doc 6 : hors sujet
```

### 8.25.6. Contexte final

Nous injectons :

```text
Source 1 : checkout utilise payments API.
Source 2 : payments API est déployée sur cluster-3.
Source 3 : cluster-3 sera en maintenance vendredi soir.
```

Réponse attendue :

```text
Le service checkout peut être impacté par la maintenance de vendredi, car il
utilise l’API payments, qui est déployée sur le cluster-3, et ce cluster sera
en maintenance vendredi soir. Les sources ne précisent pas s’il existe une
redondance ou un mécanisme de failover.
```

---

## 8. 26. Pseudo-code d’un pipeline optimisé

```python
def optimized_rag(question, user):
    # 1. Classification de la question
    question_type = classify_question(question)

    # 2. Rewriting si nécessaire
    if question_type in ["ambiguous", "multi_hop", "broad"]:
        rewritten_queries = rewrite_or_expand(question)
    else:
        rewritten_queries = [question]

    # 3. Recherche hybride sur chaque requête
    candidates = []

    for query in rewritten_queries:
        vector_results = vector_search(
            query=query,
            top_k=30,
            filters={"permissions": user.permissions}
        )

        lexical_results = lexical_search(
            query=query,
            top_k=30,
            filters={"permissions": user.permissions}
        )

        candidates.extend(vector_results)
        candidates.extend(lexical_results)

    # 4. Fusion et déduplication
    candidates = merge_and_deduplicate(candidates)

    # 5. Reranking
    ranked = rerank(
        question=question,
        documents=candidates
    )

    # 6. Diversification
    final_chunks = diversify(
        ranked,
        max_chunks=6
    )

    # 7. Construction du prompt
    prompt = build_grounded_prompt(
        question=question,
        chunks=final_chunks
    )

    # 8. Génération
    answer = llm.generate(prompt)

    # 9. Validation optionnelle
    checked_answer = validate_grounding(
        question=question,
        chunks=final_chunks,
        answer=answer
    )

    return checked_answer
```

Ce pseudo-code illustre une architecture plus réaliste qu’un RAG minimal.

---

## 8.27. Mini-TP : comparer un RAG naïf et un RAG optimisé

### Objectif

Nous voulons comparer la qualité de plusieurs pipelines.

### Corpus

```text
doc_1 : Le service checkout utilise l’API payments.
doc_2 : L’API payments est déployée sur le cluster-3.
doc_3 : Le cluster-3 sera en maintenance vendredi soir.
doc_4 : Le service checkout expose une API REST pour les paniers.
doc_5 : Ancienne documentation : checkout était déployé sur cluster-1.
doc_6 : Le cluster-7 sera mis à jour lundi.
doc_7 : L’erreur ERR_MODULE_NOT_FOUND apparaît lorsque Node.js ne trouve pas un module importé.
doc_8 : En ESM, les imports relatifs compilés doivent souvent inclure l’extension .js.
```

### Questions

```text
Le checkout sera-t-il affecté vendredi ?
Pourquoi ai-je ERR_MODULE_NOT_FOUND après build TypeScript ?
Je n’arrive plus à me connecter, que faire ?
```

### Pipelines à comparer

```text
Pipeline A :
recherche vectorielle simple top_k=3

Pipeline B :
recherche vectorielle top_k=10 + reranking

Pipeline C :
recherche hybride + reranking

Pipeline D :
query rewriting + hybride + reranking + métadonnées
```

### Analyse attendue

Nous comparons :

```text
sources attendues récupérées ;
bruit ;
qualité de la réponse ;
citations ;
latence ;
complexité.
```

L’objectif est de voir qu’un pipeline plus complexe n’est pas toujours nécessaire, mais devient utile sur les questions difficiles.

---

## 8.28. Mini-TP : query rewriting

### Objectif

Nous voulons apprendre à reformuler une question sans changer son sens.

### Questions originales

```text
Ça casse vendredi ?
Le build plante après compile.
Je peux plus me connecter.
C’est quoi P2025 ?
```

### Reformulations attendues

```text
Quels services ou composants seront impactés par la maintenance prévue vendredi ?

Pourquoi le build échoue-t-il après la compilation ?

Quelle procédure suivre en cas de problème de connexion au compte utilisateur ?

Que signifie l’erreur Prisma P2025 et comment la corriger ?
```

### Analyse

Nous vérifions que la reformulation conserve :

```text
les entités ;
l’intention ;
la temporalité ;
les termes exacts ;
le niveau d’incertitude.
```

---

## 8.29. Mini-TP : reranking manuel

### Objectif

Nous voulons comprendre ce qu’un reranker doit faire.

### Question

```text
Comment corriger l’erreur Prisma P2025 ?
```

### Chunks candidats

```text
A :
Prisma est un ORM pour TypeScript et Node.js.

B :
L’erreur P2025 signifie qu’un enregistrement attendu n’a pas été trouvé.

C :
Pour corriger P2025, vérifier que l’id existe avant update ou delete.

D :
PostgreSQL supporte les transactions ACID.

E :
Notre API utilise Prisma pour accéder à MariaDB.
```

### Classement attendu

```text
1. C
2. B
3. E
4. A
5. D
```

### Discussion

Nous distinguons :

```text
passage qui répond directement ;
passage qui explique le problème ;
passage contextuel ;
passage général ;
passage hors sujet.
```

---

## 8.30. Questions de compréhension

À la fin de ce chapitre, nous devons pouvoir répondre aux questions suivantes :

```text
Pourquoi la recherche vectorielle seule est-elle parfois insuffisante ?
Dans quels cas la recherche lexicale est-elle indispensable ?
Qu’est-ce qu’une recherche hybride ?
Pourquoi les scores BM25 et vectoriels ne sont-ils pas directement comparables ?
À quoi sert Reciprocal Rank Fusion ?
Qu’est-ce qu’un reranker ?
Quelle différence entre bi-encoder et cross-encoder ?
Qu’est-ce que le query rewriting ?
Quelle différence entre query rewriting et query expansion ?
Qu’est-ce que le multi-query retrieval ?
Qu’est-ce que HyDE ?
Pourquoi enrichir les chunks avec du contexte ?
Qu’est-ce que le parent-child retrieval ?
Pourquoi diversifier les résultats ?
Pourquoi optimiser les métadonnées ?
Pourquoi faut-il adapter le pipeline au type de question ?
Comment arbitrer entre qualité, coût et latence ?
```

---

## 8.31. Synthèse du chapitre

Dans ce chapitre, nous avons étudié les principales optimisations d’un système RAG.

Nous avons vu qu’un RAG naïf peut fonctionner sur des cas simples, mais qu’il atteint vite ses limites dans des contextes réels.

Nous avons étudié les principales techniques d’optimisation :

```text
recherche lexicale ;
recherche hybride ;
fusion des résultats ;
reranking ;
query rewriting ;
query expansion ;
multi-query retrieval ;
HyDE ;
contextual retrieval ;
parent-child retrieval ;
window retrieval ;
diversification ;
déduplication ;
optimisation des métadonnées ;
optimisation du chunking ;
routage selon le type de question.
```

Nous avons aussi vu que chaque optimisation a un coût :

```text
plus de latence ;
plus d’appels modèle ;
plus de complexité ;
plus de paramètres à évaluer ;
plus de risques de bruit.
```

Le message principal du chapitre est donc le suivant :

```text
Optimiser un RAG ne consiste pas à ajouter toutes les techniques disponibles.
Cela consiste à identifier le type d’échec observé, puis à appliquer
l’optimisation adaptée : meilleure requête, meilleure recherche,
meilleur classement ou meilleur contexte.
```

Dans le chapitre suivant, nous aborderons le **Graph RAG**. Nous verrons comment représenter explicitement les entités et les relations pour répondre à des questions multi-hop que le RAG vectoriel standard gère difficilement.

---

---
> [!info] Livre « RAG » — chapitre 8/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 07 — Évaluer un système RAG|← 07 — Évaluer un système RAG]] · [[RAG — 09 — Graph RAG entités, relations et requêtes multi-hop|09 — Graph RAG entités, relations et requêtes multi-hop →]]
