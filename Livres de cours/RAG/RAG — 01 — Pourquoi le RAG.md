---
schema_version: 1
uid: 01M1BQ6275NGN11SW7EYZKET7F
titre: "RAG — 01 — Pourquoi le RAG"
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
resume: "Chapitre 1 sur 12 du livre « RAG » : Pourquoi le RAG ?. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « RAG » — chapitre 1/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — Sommaire|← Sommaire]] · [[RAG — 02 — Les embeddings représenter le sens sous forme de vecteur|02 — Les embeddings représenter le sens sous forme de vecteur →]]

# Chapitre 1 — Pourquoi le RAG ?
## 1.1. Introduction

Dans ce premier chapitre, nous allons poser le problème qui justifie l’existence du **RAG**, pour _Retrieval-Augmented Generation_. Avant de parler d’embeddings, de bases vectorielles, de Graph RAG ou d’Agentic RAG, nous devons comprendre pourquoi un LLM seul ne suffit pas toujours.

Depuis l’arrivée des grands modèles de langage, nous avons pris l’habitude de dialoguer avec des systèmes capables de produire des réponses très fluides, de résumer des textes, d’écrire du code, d’expliquer des concepts complexes ou encore de reformuler des documents. Ces modèles donnent parfois l’impression de “savoir” énormément de choses.

Pourtant, lorsqu’on veut utiliser un LLM dans un contexte professionnel, technique ou documentaire, une difficulté apparaît très vite : le modèle ne connaît pas nécessairement nos données.

Il ne connaît pas notre documentation interne, nos procédures métier, nos contrats, nos tickets, nos décisions d’architecture, nos dépôts Git privés, nos comptes rendus de réunion ou nos bases de connaissances internes. Même lorsqu’il connaît un sujet général, il ne connaît pas forcément la version exacte, récente et validée dont nous avons besoin.

Le RAG répond à cette difficulté en ajoutant une étape de recherche documentaire avant la génération de la réponse.

L’idée centrale est simple :

```text
Avant de demander au LLM de répondre, nous allons chercher les documents pertinents,
puis nous allons les lui fournir comme contexte.
```

Le LLM n’est donc plus seulement utilisé comme une mémoire générale. Il devient un moteur de génération capable de s’appuyer sur des sources externes contrôlées.

---

## 1.2. Les limites d’un LLM utilisé seul

Pour comprendre le RAG, nous devons d’abord comprendre les limites d’un LLM sans accès documentaire externe.

Un LLM est un modèle statistique entraîné sur de très grands volumes de texte. Il apprend des régularités linguistiques, des structures de raisonnement, des formes de discours, des connaissances générales et des relations fréquentes entre concepts.

Mais cette connaissance a plusieurs limites.

### 1.2.1. Le modèle ne connaît pas nos documents internes

Un modèle généraliste peut connaître Python, Linux, Kubernetes, l’histoire de l’informatique ou les grands principes du droit. Mais il ne connaît pas automatiquement :

```text
notre documentation interne ;
notre code privé ;
notre wiki d’équipe ;
nos tickets Jira ;
nos contrats ;
nos procédures ;
nos décisions d’architecture ;
nos incidents de production ;
nos règles métier spécifiques.
```

Si nous lui demandons :

```text
Quelle est la procédure interne pour redémarrer le service de facturation ?
```

il peut produire une réponse plausible, mais cette réponse ne sera pas nécessairement celle de notre organisation.

Il peut dire :

```text
Vérifiez les logs, redémarrez le service, puis contrôlez les métriques.
```

Cette réponse est raisonnable, mais elle n’est pas forcément conforme à la procédure réelle. Peut-être que le redémarrage nécessite une validation préalable. Peut-être qu’il faut prévenir une équipe. Peut-être qu’il existe un script précis à utiliser. Peut-être que le service ne doit jamais être redémarré directement en production.

Le LLM seul ne peut pas deviner cela de manière fiable.

### 1.2.2. Le modèle peut être obsolète

Un LLM est entraîné à un moment donné. Même si certains systèmes peuvent être connectés à des outils externes, le modèle lui-même ne contient pas automatiquement toutes les informations récentes.

Cela pose problème pour :

```text
les versions logicielles ;
les API récentes ;
les réglementations ;
les prix ;
les procédures modifiées ;
les incidents récents ;
les documents mis à jour ;
les décisions prises après l’entraînement du modèle.
```

Supposons que nous demandions :

```text
Quelle est la procédure actuelle d’onboarding technique ?
```

Si cette procédure a été modifiée la semaine dernière, un LLM seul ne peut pas la connaître, sauf si cette information lui est fournie.

Le RAG permet justement de connecter le modèle à des documents récents et contrôlés.

### 1.2.3. Le modèle peut halluciner

Une hallucination désigne une réponse produite par un modèle qui semble crédible, mais qui est fausse, inventée ou insuffisamment fondée.

Le problème n’est pas seulement que le modèle se trompe. Le problème est qu’il peut se tromper avec beaucoup d’assurance.

Par exemple, si nous demandons :

```text
Quelle clause du contrat indique le délai de paiement ?
```

un LLM seul peut inventer une référence :

```text
Cette information se trouve à l’article 7.2 du contrat.
```

Même si l’article 7.2 n’existe pas.

Dans un contexte juridique, administratif, technique ou médical, ce type d’erreur peut être très problématique.

Le RAG ne supprime pas totalement les hallucinations, mais il permet de réduire fortement le risque en demandant au modèle de répondre à partir de sources explicites.

### 1.2.4. Le modèle ne cite pas naturellement ses sources

Un LLM peut produire une réponse correcte sans dire d’où vient l’information. Or, dans beaucoup de contextes, nous ne voulons pas seulement une réponse : nous voulons pouvoir la vérifier.

Dans un système professionnel, nous avons souvent besoin de savoir :

```text
quel document a été utilisé ;
quelle section justifie la réponse ;
quelle version du document a été consultée ;
si la réponse est directement soutenue par la source ;
si le modèle extrapole.
```

Sans mécanisme de recherche et de citation, le LLM agit comme une boîte noire.

Avec le RAG, nous pouvons associer chaque réponse aux passages documentaires utilisés.

### 1.2.5. Le modèle ne sait pas toujours dire “je ne sais pas”

Un bon système documentaire doit être capable de dire :

```text
Je n’ai pas trouvé l’information dans les documents disponibles.
```

Un LLM seul peut être tenté de produire tout de même une réponse plausible. Cela vient du fait que son objectif principal est de générer une continuation linguistique probable, pas de garantir que l’information provient d’une source validée.

Dans un système RAG, nous pouvons imposer une règle plus stricte :

```text
Si les documents récupérés ne permettent pas de répondre, nous devons le dire explicitement.
```

Cela transforme l’usage du LLM : nous ne lui demandons plus seulement de répondre, nous lui demandons de répondre sous contrainte documentaire.

---

## 1.3. L’idée fondamentale du RAG

Le RAG repose sur une idée simple :

```text
Nous allons chercher les informations pertinentes avant de générer la réponse.
```

Le terme **RAG** signifie :

```text
Retrieval-Augmented Generation
```

On peut le traduire par :

```text
génération augmentée par récupération d’information
```

Le système combine donc deux grandes fonctions :

```text
retrieval : retrouver les bons documents ou passages ;
generation : produire une réponse en langage naturel.
```

Un pipeline RAG minimal ressemble à ceci :

```text
Question utilisateur
        ↓
Recherche de passages pertinents
        ↓
Construction d’un contexte
        ↓
Appel au LLM avec ce contexte
        ↓
Réponse générée à partir des documents
```

La différence avec un LLM seul est donc majeure.

Sans RAG :

```text
Utilisateur → LLM → Réponse
```

Avec RAG :

```text
Utilisateur → Recherche documentaire → Contexte → LLM → Réponse sourcée
```

Le LLM ne travaille plus uniquement avec sa connaissance interne. Il travaille avec une mémoire externe fournie au moment de la requête.

---

## 1.4. Exemple simple

Imaginons une documentation interne contenant le passage suivant :

```text
La procédure de réinitialisation d’un mot de passe administrateur nécessite
une validation par le responsable sécurité. Une fois la demande validée,
l’administrateur peut utiliser le script reset-admin-password.sh.
```

Un utilisateur demande :

```text
Comment réinitialiser un mot de passe administrateur ?
```

Un LLM seul pourrait répondre :

```text
Vous pouvez utiliser l’interface d’administration ou exécuter une commande
de réinitialisation.
```

Cette réponse est générique. Elle peut être partiellement correcte, mais elle ne respecte pas forcément la procédure interne.

Un système RAG va d’abord chercher dans la documentation. Il récupère le passage pertinent, puis le fournit au LLM.

Le prompt transmis au modèle peut ressembler à ceci :

```text
Réponds à la question en utilisant uniquement le contexte ci-dessous.
Si le contexte ne suffit pas, dis-le.

Contexte :
La procédure de réinitialisation d’un mot de passe administrateur nécessite
une validation par le responsable sécurité. Une fois la demande validée,
l’administrateur peut utiliser le script reset-admin-password.sh.

Question :
Comment réinitialiser un mot de passe administrateur ?
```

Le modèle peut alors répondre :

```text
Pour réinitialiser un mot de passe administrateur, nous devons d’abord obtenir
une validation du responsable sécurité. Une fois cette validation obtenue,
l’administrateur peut utiliser le script reset-admin-password.sh.
```

La réponse est plus fiable, car elle est fondée sur un document précis.

---

## 1.5. Le RAG comme mémoire externe

Une manière utile de comprendre le RAG est de distinguer deux types de mémoire.

### 1.5.1. La mémoire paramétrique du modèle

La mémoire paramétrique correspond aux connaissances encodées dans les poids du modèle pendant son entraînement.

C’est ce que le modèle “sait” déjà avant qu’on lui fournisse un contexte.

Par exemple, un LLM peut probablement expliquer :

```text
ce qu’est une base de données relationnelle ;
ce qu’est une API REST ;
ce qu’est Docker ;
ce qu’est Kubernetes ;
ce qu’est une fonction de hash ;
ce qu’est une classe en Java.
```

Ces connaissances sont générales et proviennent de son entraînement.

### 1.5.2. La mémoire externe du RAG

La mémoire externe correspond aux informations que nous allons chercher au moment de la question.

Elle peut venir de :

```text
documents Markdown ;
PDF ;
pages HTML ;
wikis ;
tickets ;
bases SQL ;
fichiers de configuration ;
documentation technique ;
contrats ;
rapports ;
dépôts Git ;
emails ;
comptes rendus ;
bases métier.
```

Le RAG permet donc de combiner :

```text
connaissance générale du LLM
+
connaissance spécifique récupérée dans nos sources
```

C’est cette combinaison qui rend le système intéressant.

Le LLM apporte sa capacité à comprendre la question, à reformuler, à synthétiser et à produire une réponse claire.

Le système de retrieval apporte les informations précises, récentes et contrôlées.

---

## 1.6. Pourquoi ne pas simplement fine-tuner le modèle ?

Une question naturelle est la suivante :

```text
Pourquoi ne pas entraîner ou fine-tuner directement le modèle sur nos documents ?
```

Le fine-tuning peut être utile, mais il ne remplace pas le RAG dans beaucoup de situations.

### 1.6.1. Le fine-tuning modifie le comportement du modèle

Le fine-tuning sert surtout à adapter un modèle à :

```text
un style de réponse ;
un format de sortie ;
une tâche spécifique ;
un vocabulaire métier ;
une manière de raisonner ;
des exemples de comportement attendu.
```

Il est très utile si nous voulons que le modèle réponde toujours dans un certain format, classe des documents, produise du JSON structuré ou imite un style spécifique.

Mais il n’est pas idéal pour stocker une grande base documentaire mouvante.

### 1.6.2. Les documents changent souvent

Dans une entreprise ou un projet logiciel, les documents évoluent :

```text
les procédures changent ;
les contrats sont mis à jour ;
les versions logicielles évoluent ;
les incidents sont résolus ;
les décisions d’architecture sont remplacées ;
les informations obsolètes doivent disparaître.
```

Si nous fine-tunons un modèle sur ces données, il faudra le réentraîner à chaque évolution importante. Cela peut être coûteux, lent et difficile à contrôler.

Avec le RAG, nous pouvons simplement mettre à jour l’index documentaire.

### 1.6.3. Le fine-tuning ne cite pas naturellement les sources

Même si un modèle a été fine-tuné sur des documents internes, il ne sait pas forcément dire précisément d’où vient une information.

Il peut avoir absorbé une formulation, mais il ne fournit pas automatiquement :

```text
le document source ;
la page ;
le paragraphe ;
la version ;
la date de mise à jour.
```

Le RAG est plus adapté lorsqu’on a besoin de traçabilité.

### 1.6.4. Le RAG est souvent plus simple à auditer

Dans un système RAG, nous pouvons inspecter :

```text
la question ;
les documents récupérés ;
les scores de similarité ;
les sources utilisées ;
le prompt final ;
la réponse générée.
```

Nous pouvons donc mieux comprendre pourquoi le système a répondu ainsi.

Dans un modèle fine-tuné, une partie de la connaissance est cachée dans les poids du modèle, ce qui rend l’explication plus difficile.

### 1.6.5. Fine-tuning et RAG sont complémentaires

Il ne faut pas opposer radicalement fine-tuning et RAG.

En pratique, nous pouvons utiliser :

```text
RAG pour fournir les connaissances ;
fine-tuning pour adapter le comportement du modèle.
```

Par exemple, nous pouvons fine-tuner un modèle pour qu’il réponde toujours dans un format métier, tout en utilisant le RAG pour lui fournir les documents pertinents.

---

## 1.7. Les cas d’usage typiques du RAG

Le RAG est particulièrement utile lorsque nous avons un corpus documentaire important et que nous voulons interagir avec lui en langage naturel.

### 1.7.1. Assistant de documentation technique

Un cas fréquent consiste à créer un assistant capable de répondre à des questions sur une documentation technique.

Exemple :

```text
Comment configurer l’authentification OAuth ?
Quelle variable d’environnement contrôle le timeout Redis ?
Quel service consomme cette file BullMQ ?
Comment déployer l’application en environnement de test ?
```

Le RAG permet de retrouver les bons passages dans les README, les fichiers de configuration, les tickets ou les documents d’architecture.

### 1.7.2. Assistant juridique ou administratif

Le RAG peut aussi aider à naviguer dans des corpus juridiques ou administratifs.

Exemples :

```text
Que dit ce contrat sur les pénalités de retard ?
Quelle procédure suivre pour demander une aide financière ?
Quels documents sont nécessaires pour ce dossier ?
Quelle clause encadre la confidentialité ?
```

Dans ce type de cas, les sources sont très importantes. Il ne suffit pas d’avoir une réponse plausible : nous devons pouvoir vérifier le texte exact.

### 1.7.3. Support client

Un système RAG peut être utilisé pour aider un service support.

Il peut répondre à partir :

```text
d’une base de connaissances ;
d’anciens tickets ;
de procédures internes ;
de guides produits ;
de FAQ ;
de notes de version.
```

Il peut proposer une réponse à un client, mais aussi indiquer quelle source justifie cette réponse.

### 1.7.4. Recherche dans des archives internes

Dans beaucoup d’organisations, l’information existe déjà, mais elle est difficile à trouver.

Elle est dispersée entre :

```text
emails ;
documents partagés ;
wikis ;
PDF ;
présentations ;
tickets ;
comptes rendus ;
dépôts Git.
```

Le RAG peut servir d’interface en langage naturel pour explorer cette mémoire documentaire.

### 1.7.5. Aide au développement logiciel

Pour des équipes informatiques, le RAG peut être appliqué à :

```text
documentation d’API ;
code source ;
issues ;
pull requests ;
décisions d’architecture ;
logs ;
guides de déploiement ;
runbooks d’exploitation.
```

Exemple de question :

```text
Quels modules utilisent encore l’ancien client Redis ?
```

Un RAG simple peut aider, mais selon la complexité, il faudra peut-être combiner recherche vectorielle, recherche lexicale, analyse de graphe et outils spécialisés.

---

## 1.8. Le RAG n’est pas seulement “un chatbot sur des documents”

Une erreur fréquente consiste à réduire le RAG à :

```text
mettre des PDF dans une base vectorielle et poser des questions à un chatbot.
```

Cette vision est trop simplifiée.

Un vrai système RAG est une architecture complète. Il faut traiter plusieurs problèmes :

```text
Comment extraire correctement le texte ?
Comment découper les documents ?
Comment conserver les métadonnées ?
Comment gérer les droits d’accès ?
Comment retrouver les bons passages ?
Comment éviter de récupérer trop de bruit ?
Comment citer les sources ?
Comment évaluer les réponses ?
Comment mettre à jour l’index ?
Comment détecter les hallucinations ?
Comment gérer les documents contradictoires ?
```

La qualité du système ne dépend pas seulement du LLM utilisé. Elle dépend aussi fortement de la qualité du pipeline documentaire.

Un mauvais RAG avec un très bon LLM peut produire de mauvaises réponses.

Un bon RAG nécessite donc une approche d’ingénierie complète.

---

## 1.9. Les grandes étapes d’un système RAG

Nous pouvons résumer un système RAG en deux phases principales :

```text
phase d’indexation ;
phase de requête.
```

### 1.9.1. La phase d’indexation

La phase d’indexation est réalisée avant les questions utilisateurs.

Elle consiste à préparer les documents.

Pipeline typique :

```text
documents bruts
        ↓
extraction du texte
        ↓
nettoyage
        ↓
découpage en chunks
        ↓
création des embeddings
        ↓
stockage dans un index
```

Chaque chunk est généralement stocké avec :

```text
son texte ;
son embedding ;
son document source ;
son titre ;
sa position ;
ses métadonnées ;
ses droits d’accès éventuels.
```

Cette phase est critique, car si les documents sont mal extraits ou mal découpés, le retrieval sera mauvais.

### 1.9.2. La phase de requête

La phase de requête a lieu lorsqu’un utilisateur pose une question.

Pipeline typique :

```text
question utilisateur
        ↓
embedding de la question
        ↓
recherche des chunks proches
        ↓
sélection des passages pertinents
        ↓
construction du prompt
        ↓
génération de la réponse
        ↓
retour avec sources
```

C’est cette phase qui donne l’impression à l’utilisateur de discuter avec sa documentation.

Mais en réalité, le système effectue une recherche avant de générer la réponse.

---

## 1.10. Les composants principaux

Un système RAG minimal comporte plusieurs composants.

### 1.10.1. Les sources documentaires

Ce sont les données que nous voulons rendre interrogeables :

```text
PDF ;
Markdown ;
HTML ;
bases de données ;
wikis ;
tickets ;
dépôts Git ;
documents bureautiques ;
emails ;
rapports.
```

### 1.10.2. Le pipeline d’ingestion

Il transforme les documents bruts en texte exploitable.

Il doit gérer :

```text
l’extraction ;
le nettoyage ;
la normalisation ;
les métadonnées ;
les erreurs ;
les formats complexes.
```

### 1.10.3. Le chunker

Le chunker découpe les documents en morceaux plus petits.

Cette étape est essentielle, car le LLM ne peut pas recevoir toute la base documentaire en contexte.

Il faut donc produire des unités de texte suffisamment petites pour être retrouvées facilement, mais suffisamment grandes pour garder du sens.

### 1.10.4. Le modèle d’embedding

Le modèle d’embedding transforme chaque chunk en vecteur.

Il transforme aussi la question utilisateur en vecteur lors de la recherche.

### 1.10.5. L’index ou la base vectorielle

La base vectorielle stocke les embeddings et permet de retrouver rapidement les vecteurs les plus proches d’une requête.

Elle peut être spécialisée, comme une base vectorielle dédiée, ou intégrée à une base de données existante.

### 1.10.6. Le retriever

Le retriever est le composant qui sélectionne les documents ou passages pertinents.

Il peut utiliser :

```text
recherche vectorielle ;
recherche lexicale ;
filtres par métadonnées ;
recherche hybride ;
reranking.
```

### 1.10.7. Le générateur

Le générateur est le LLM qui produit la réponse finale à partir du contexte récupéré.

### 1.10.8. Le module d’évaluation et d’observabilité

Dans un système sérieux, nous devons aussi observer et évaluer :

```text
les documents récupérés ;
les scores ;
les sources utilisées ;
la qualité des réponses ;
les hallucinations ;
les retours utilisateurs ;
les temps de réponse ;
les coûts.
```

---

## 1.11. Exemple complet : assistant sur documentation d’infrastructure

Prenons un exemple plus proche d’un contexte informatique.

Nous avons une documentation interne contenant ces informations :

```text
Le service checkout gère la validation des paniers.
Il utilise l’API payments pour déclencher les paiements.
L’API payments est déployée sur le cluster-3.
Le cluster-3 fera l’objet d’une maintenance vendredi soir.
```

Un utilisateur demande :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Un RAG standard va essayer de retrouver les passages les plus proches de la question.

Il peut récupérer :

```text
Le service checkout gère la validation des paniers.
```

et :

```text
Le cluster-3 fera l’objet d’une maintenance vendredi soir.
```

Mais il peut rater :

```text
L’API payments est déployée sur le cluster-3.
```

Or ce passage est indispensable pour relier checkout à cluster-3.

Cet exemple nous permet d’introduire une limite importante du RAG standard : il retrouve des textes proches de la question, mais il ne retrouve pas toujours tous les éléments nécessaires à un raisonnement multi-hop.

Nous verrons plus tard que des approches comme **Graph RAG** permettent de représenter explicitement les relations :

```text
checkout → payments API → cluster-3 → maintenance vendredi
```

Mais pour l’instant, retenons surtout ceci :

```text
Le RAG standard est très utile, mais il n’est pas une solution magique.
```

---

## 1.12. Les bénéfices du RAG

Le RAG apporte plusieurs bénéfices importants.

### 1.12.1. Réponses plus pertinentes

Le modèle peut répondre à partir de documents spécifiques au contexte de l’utilisateur ou de l’organisation.

### 1.12.2. Données plus fraîches

Il suffit de mettre à jour l’index documentaire pour intégrer de nouvelles informations.

### 1.12.3. Réduction des hallucinations

Le modèle est contraint par un contexte documentaire. Cela ne supprime pas totalement les erreurs, mais cela les réduit.

### 1.12.4. Traçabilité

Nous pouvons associer une réponse à des sources.

### 1.12.5. Séparation entre connaissance et comportement

Les documents restent dans une base externe. Le modèle n’a pas besoin d’être réentraîné pour chaque mise à jour documentaire.

### 1.12.6. Adaptabilité

Le même LLM peut être utilisé avec plusieurs bases documentaires différentes.

---

## 1.13. Les limites du RAG

Nous devons aussi être lucides sur les limites.

### 1.13.1. Si le bon document n’est pas récupéré, le LLM ne peut pas répondre correctement

Le retrieval est souvent le maillon critique.

Si le système récupère les mauvais passages, le LLM produira une réponse fondée sur un mauvais contexte.

### 1.13.2. Si les documents sont mauvais, la réponse sera mauvaise

Le RAG ne transforme pas une documentation désorganisée en vérité fiable.

Si les documents sont :

```text
obsolètes ;
contradictoires ;
mal extraits ;
mal structurés ;
incomplets ;
non sourcés ;
```

alors le système aura des difficultés.

### 1.13.3. Le chunking peut casser le sens

Si nous découpons les documents trop finement, nous perdons le contexte.

Si nous les découpons trop largement, nous récupérons trop de bruit.

Le chunking est donc un compromis.

### 1.13.4. Le modèle peut encore halluciner

Même avec les bons documents, le LLM peut :

```text
mal lire un passage ;
surinterpréter ;
ignorer une nuance ;
mélanger deux sources ;
inventer une conclusion.
```

Il faut donc évaluer le système.

### 1.13.5. Les droits d’accès sont critiques

Dans un système professionnel, un utilisateur ne doit jamais obtenir une réponse fondée sur un document qu’il n’a pas le droit de consulter.

La sécurité du RAG est donc un sujet central.

---

## 1.14. RAG standard, Graph RAG et Agentic RAG

Nous introduisons dès maintenant trois grandes familles que nous détaillerons plus tard.

### 1.14.1. RAG standard

Le RAG standard cherche les chunks les plus proches de la question.

Il est adapté aux questions factuelles simples.

Exemple :

```text
Quelle est la commande pour redémarrer le service ?
```

### 1.14.2. Graph RAG

Le Graph RAG ajoute une représentation sous forme de graphe :

```text
entités + relations
```

Il est adapté aux questions qui demandent de relier plusieurs informations.

Exemple :

```text
Quels services seront impactés par la maintenance du cluster-3 ?
```

### 1.14.3. Agentic RAG

L’Agentic RAG laisse un agent choisir les outils à utiliser et l’ordre des opérations.

Il est adapté aux tâches dynamiques multi-sources.

Exemple :

```text
Analyse l’incident d’hier, compare les logs avec les tickets ouverts,
puis propose une hypothèse et une action corrective.
```

Ces trois approches ne forment pas une échelle où l’une remplacerait l’autre. Elles correspondent à des types de problèmes différents.

---

## 1.15. Ce que nous allons apprendre dans la suite du cours

Dans les chapitres suivants, nous allons construire progressivement notre compréhension du RAG.

Nous verrons d’abord les embeddings, qui permettent de représenter les textes sous forme de vecteurs.

Puis nous étudierons la similarité cosinus, qui permet de comparer ces vecteurs.

Nous construirons ensuite un pipeline RAG standard :

```text
ingestion ;
chunking ;
embedding ;
indexation ;
retrieval ;
prompt augmenté ;
génération.
```

Nous apprendrons à évaluer ce pipeline, puis à l’améliorer avec :

```text
recherche hybride ;
reranking ;
query rewriting ;
métadonnées ;
Graph RAG ;
Agentic RAG ;
RAG multimodal.
```

Enfin, nous discuterons des contraintes de production :

```text
sécurité ;
droits d’accès ;
observabilité ;
coût ;
latence ;
mise à jour des index ;
qualité documentaire.
```

---

## 1.16. Synthèse du chapitre

Dans ce chapitre, nous avons posé la motivation du RAG.

Nous avons vu qu’un LLM seul est puissant, mais limité lorsqu’il doit répondre à partir de connaissances spécifiques, récentes ou vérifiables.

Nous avons compris que le RAG ajoute une étape de recherche documentaire avant la génération.

Nous pouvons résumer l’idée ainsi :

```text
Un LLM seul génère une réponse à partir de ce qu’il a appris.
Un système RAG cherche d’abord les informations pertinentes,
puis demande au LLM de répondre à partir de ces informations.
```

Le RAG permet donc de construire des assistants plus fiables, plus vérifiables et mieux adaptés aux données métier.

Mais nous avons aussi vu que le RAG n’est pas magique. Sa qualité dépend de nombreux éléments :

```text
qualité des documents ;
qualité de l’extraction ;
qualité du chunking ;
qualité des embeddings ;
qualité du retrieval ;
qualité du prompt ;
qualité de l’évaluation ;
gestion des droits ;
mise à jour des sources.
```

Le message principal de ce chapitre est donc le suivant :

```text
Un RAG n’est pas simplement un chatbot branché sur des documents.
C’est une architecture de recherche, de sélection, de justification
et de génération contrôlée autour d’un LLM.
```

Dans le prochain chapitre, nous entrerons dans le cœur mathématique et technique du RAG : les **embeddings**.

---

---
> [!info] Livre « RAG » — chapitre 1/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — Sommaire|← Sommaire]] · [[RAG — 02 — Les embeddings représenter le sens sous forme de vecteur|02 — Les embeddings représenter le sens sous forme de vecteur →]]
