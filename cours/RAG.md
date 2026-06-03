# Plan du cours

# Cours : Comprendre, concevoir et évaluer un système RAG

## Objectif général du cours

Dans ce cours, nous allons comprendre comment un système **RAG**, pour _Retrieval-Augmented Generation_, permet d’augmenter un LLM avec des connaissances externes. Nous verrons pourquoi cette approche est devenue centrale dans les applications d’IA générative, en particulier lorsqu’on veut exploiter des documents internes, des bases de connaissances, des archives métier ou des données qui ne sont pas présentes dans les paramètres du modèle.

Nous partirons des fondements : embeddings, similarité cosinus, recherche vectorielle, découpage documentaire, puis nous irons vers des architectures plus avancées : RAG hybride, reranking, Graph RAG, Agentic RAG, multimodal RAG et évaluation.

---

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

### 21..5. Le modèle ne sait pas toujours dire “je ne sais pas”

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

# Chapitre 2 — Les embeddings : représenter le sens sous forme de vecteur

## 2.1. Introduction

Dans le chapitre précédent, nous avons vu pourquoi le RAG existe. Nous avons compris qu’un LLM seul ne suffit pas toujours lorsqu’il doit répondre à partir de documents spécifiques, récents ou vérifiables.

Nous avons aussi vu qu’un système RAG repose sur une idée simple :

```text
Nous cherchons d’abord les passages pertinents,
puis nous demandons au LLM de répondre à partir de ces passages.
```

Mais une question centrale apparaît immédiatement :

```text
Comment retrouver automatiquement les passages pertinents dans une grande base documentaire ?
```

Si l’utilisateur demande :

```text
Comment réinitialiser mes identifiants ?
```

il est possible que le document pertinent contienne plutôt :

```text
Procédure de changement du mot de passe utilisateur.
```

Les mots ne sont pas exactement les mêmes, mais le sens est proche.

Une recherche classique par mots-clés peut rater certains résultats utiles. C’est là qu’interviennent les **embeddings**.

Un embedding permet de représenter un texte sous forme de vecteur numérique, de façon à ce que des textes proches en sens soient proches dans l’espace vectoriel.

Dans ce chapitre, nous allons comprendre :

```text
ce qu’est un embedding ;
pourquoi on transforme le texte en vecteurs ;
comment ces vecteurs permettent la recherche sémantique ;
quelle est la différence entre recherche lexicale et recherche vectorielle ;
quelles sont les limites des embeddings.
```

---

## 2.2. Pourquoi transformer du texte en nombres ?

Un ordinateur ne comprend pas directement le sens d’une phrase. Pour pouvoir comparer, classer ou rechercher des textes, nous devons les représenter dans une forme mathématique.

Un texte brut est une suite de caractères :

```text
"Le service checkout utilise l’API payments."
```

Pour effectuer des calculs, nous devons transformer cette phrase en nombres.

Un embedding est précisément cette transformation :

```text
texte → vecteur de nombres réels
```

Par exemple :

```text
"Le service checkout utilise l’API payments."
→ [0.12, -0.34, 0.87, 0.05, ..., -0.21]
```

Ce vecteur peut contenir plusieurs centaines ou plusieurs milliers de dimensions.

L’objectif n’est pas que chaque dimension soit facilement interprétable par un humain. L’objectif est que la position du vecteur dans l’espace encode une partie du sens du texte.

Autrement dit, nous voulons que deux textes proches en sens soient représentés par deux vecteurs proches.

---

## 2.3. Intuition générale d’un embedding

Prenons trois phrases :

```text
A. Le chien dort sur le canapé.
B. Un animal domestique se repose sur le sofa.
C. PostgreSQL permet de gérer des transactions SQL.
```

Pour nous, les phrases A et B sont proches.

Elles ne partagent pas exactement les mêmes mots :

```text
chien ≠ animal domestique
dort ≠ se repose
canapé ≠ sofa
```

Mais elles décrivent une situation similaire.

La phrase C, elle, parle d’un sujet complètement différent.

Un bon modèle d’embedding devrait donc produire quelque chose comme :

```text
embedding(A) proche de embedding(B)
embedding(A) éloigné de embedding(C)
embedding(B) éloigné de embedding(C)
```

En simplifiant, nous pouvons imaginer un espace en deux dimensions :

```text
                         informatique
                              ↑
                              |
                              |        PostgreSQL
                              |             x
                              |
                              |
                              |
chien / canapé       x   x
                              |
------------------------------→ animal / domestique
```

Dans la réalité, l’espace n’a pas deux dimensions, mais souvent plusieurs centaines ou milliers. Cependant, l’idée reste la même : les textes sont placés dans un espace où la proximité géométrique doit refléter une proximité de sens.

---

## 2.4. Embedding de mot, de phrase et de document

Il existe plusieurs niveaux d’embeddings.

### 2.4.1. Embedding de mot

Historiquement, beaucoup de modèles représentaient chaque mot par un vecteur.

Exemple :

```text
chien → vecteur
chat → vecteur
voiture → vecteur
ordinateur → vecteur
```

Un bon modèle place les mots proches dans des zones proches.

Par exemple :

```text
chien
chat
animal
vétérinaire
```

devraient être relativement proches.

Tandis que :

```text
SQL
PostgreSQL
transaction
index
```

devraient être proches entre eux, mais éloignés du vocabulaire animalier.

### 2.4.2. Embedding de phrase

Pour un RAG, nous voulons souvent représenter des phrases ou des paragraphes entiers.

Exemple :

```text
"Le cluster-3 sera en maintenance vendredi."
→ vecteur
```

Cela permet de comparer une question utilisateur avec un passage documentaire.

### 2.4.3. Embedding de document

Nous pouvons aussi créer un embedding pour un document entier.

Mais dans un système RAG, nous évitons souvent d’indexer seulement des documents entiers, car ils sont trop longs et trop généraux.

Si un document de vingt pages contient une seule information pertinente, l’embedding global du document risque de diluer cette information.

C’est pourquoi nous découpons généralement les documents en morceaux plus petits, appelés **chunks**.

Nous y reviendrons dans un chapitre dédié.

---

## 2.5. Recherche lexicale et recherche sémantique

Pour comprendre l’intérêt des embeddings, comparons deux manières de chercher de l’information.

### 2.5.1. Recherche lexicale

La recherche lexicale repose sur les mots présents dans la requête.

Si nous cherchons :

```text
mot de passe
```

le moteur cherche des documents contenant :

```text
mot
passe
mot de passe
```

Cette approche est simple et très efficace dans beaucoup de cas.

Elle fonctionne particulièrement bien pour :

```text
noms exacts ;
identifiants ;
codes d’erreur ;
noms de fichiers ;
noms de classes ;
références ;
numéros de tickets ;
termes rares.
```

Exemple :

```text
Erreur : ERR_MODULE_NOT_FOUND
```

Dans ce cas, une recherche lexicale est souvent excellente, car nous voulons retrouver exactement cette chaîne.

### 2.5.2. Recherche sémantique

La recherche sémantique ne cherche pas seulement les mêmes mots. Elle cherche des textes proches en sens.

Exemple :

```text
Question :
Comment changer mon mot de passe ?

Document :
Procédure de réinitialisation des identifiants utilisateur.
```

Une recherche lexicale peut avoir du mal, car les mots ne sont pas identiques.

Une recherche sémantique peut comprendre que :

```text
changer mot de passe
≈ réinitialisation des identifiants
```

Elle peut donc retrouver le bon document.

### 2.5.3. Les deux approches sont complémentaires

Il ne faut pas opposer naïvement recherche lexicale et recherche sémantique.

La recherche vectorielle n’est pas toujours meilleure.

Exemple :

```text
Question :
Que signifie l’erreur P2025 dans Prisma ?
```

Ici, le terme `P2025` est très important. Une recherche lexicale ou hybride sera souvent plus fiable qu’une recherche purement sémantique.

Autre exemple :

```text
Question :
Où est utilisé le fichier docker-compose.prod.yml ?
```

Le nom exact du fichier doit être retrouvé.

En production, beaucoup de systèmes RAG sérieux utilisent une recherche hybride :

```text
recherche lexicale
+
recherche vectorielle
+
filtrage par métadonnées
+
reranking
```

---

## 2.6. Comment un modèle produit-il un embedding ?

Un modèle d’embedding est un modèle entraîné pour transformer du texte en vecteur.

Nous pouvons le voir comme une fonction :

```text
f(texte) = vecteur
```

Par exemple :

```text
f("Le cluster-3 est en maintenance vendredi")
=
[0.18, -0.22, 0.74, ..., 0.09]
```

Le modèle a appris, pendant son entraînement, à placer des textes proches dans des régions proches de l’espace vectoriel.

Il ne s’agit pas d’un dictionnaire manuel. Le modèle apprend statistiquement à partir de grands volumes de textes.

### 2.6.1. Exemple conceptuel

Supposons que le modèle ait vu beaucoup de textes où les mots suivants apparaissent dans des contextes similaires :

```text
mot de passe
password
identifiants
authentification
connexion
réinitialisation
compte utilisateur
```

Il apprend progressivement que ces concepts sont liés.

Donc une question comme :

```text
Je n’arrive plus à me connecter à mon compte.
```

peut être rapprochée d’un document intitulé :

```text
Procédure de réinitialisation du mot de passe.
```

Même si la question ne contient pas explicitement “mot de passe”.

### 2.6.2. Les dimensions ne sont pas directement lisibles

Un embedding peut ressembler à ceci :

```text
[0.018, -0.104, 0.582, -0.331, ..., 0.047]
```

Nous ne pouvons généralement pas dire :

```text
la dimension 12 représente la sécurité ;
la dimension 53 représente Kubernetes ;
la dimension 118 représente le droit.
```

Les dimensions sont des représentations distribuées. Le sens est encodé dans des combinaisons de dimensions, pas dans une seule coordonnée facilement lisible.

C’est une différence importante avec des systèmes symboliques où chaque relation est explicitement nommée.

---

## 2.7. Exemple dans un système RAG

Prenons un mini corpus documentaire :

```text
Doc 1 :
La procédure de réinitialisation du mot de passe nécessite une validation par le responsable sécurité.

Doc 2 :
Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.

Doc 3 :
Les factures sont payables sous trente jours après émission.

Doc 4 :
L’API payments est utilisée par le service checkout.
```

Pendant l’indexation, nous transformons chaque document ou chunk en vecteur :

```text
embedding(Doc 1)
embedding(Doc 2)
embedding(Doc 3)
embedding(Doc 4)
```

Puis un utilisateur demande :

```text
Comment changer un mot de passe administrateur ?
```

Nous transformons aussi cette question en vecteur :

```text
embedding(question)
```

Ensuite, nous comparons l’embedding de la question avec les embeddings des documents.

Le document 1 devrait être le plus proche, car il parle de réinitialisation de mot de passe.

Le système récupère donc ce passage et le transmet au LLM.

Pipeline :

```text
Question utilisateur
        ↓
embedding de la question
        ↓
comparaison avec les embeddings des chunks
        ↓
récupération des chunks les plus proches
        ↓
injection dans le prompt
        ↓
réponse du LLM
```

---

## 2.8. Les embeddings ne sont pas des résumés

Il est important de ne pas confondre embedding et résumé.

Un résumé est lisible par un humain :

```text
Ce document explique la procédure de réinitialisation d’un mot de passe.
```

Un embedding ne l’est pas :

```text
[0.18, -0.22, 0.74, ..., 0.09]
```

Un embedding est une représentation mathématique destinée à être comparée avec d’autres représentations mathématiques.

Nous ne lisons pas un embedding. Nous le comparons.

C’est exactement ce qui le rend utile pour la recherche à grande échelle.

---

## 2.9. Les embeddings ne sont pas une preuve de vérité

Un embedding indique une proximité sémantique, pas une vérité.

Si deux textes sont proches dans l’espace vectoriel, cela signifie qu’ils parlent probablement de sujets proches. Cela ne signifie pas que l’un répond correctement à l’autre.

Exemple :

```text
Question :
Est-ce que le service checkout est affecté par la maintenance de vendredi ?

Chunk récupéré :
Le service checkout gère la validation des paniers.
```

Ce chunk est proche de la question, car il parle du service checkout.

Mais il ne répond pas à la question.

Il manque les informations sur :

```text
les dépendances du service checkout ;
l’API payments ;
le cluster-3 ;
la maintenance de vendredi.
```

Nous devons donc retenir une limite essentielle :

```text
similarité sémantique ≠ pertinence suffisante pour répondre
```

Un passage peut être proche mais incomplet.

À l’inverse, un passage peut être indispensable mais peu proche lexicalement ou sémantiquement de la question.

---

## 2.10. Le problème des faits intermédiaires

Reprenons l’exemple vu dans le chapitre précédent :

```text
Fait 1 :
Le service checkout utilise l’API payments.

Fait 2 :
L’API payments tourne sur le cluster-3.

Fait 3 :
Le cluster-3 sera en maintenance vendredi.
```

Question :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Le système vectoriel peut facilement retrouver le fait 1, car il contient :

```text
checkout
```

Il peut aussi retrouver le fait 3, car il contient :

```text
maintenance
vendredi
```

Mais il peut rater le fait 2 :

```text
L’API payments tourne sur le cluster-3.
```

Pourquoi ?

Parce que ce passage ne contient ni :

```text
checkout
maintenance
vendredi
affecté
```

Pourtant, il est indispensable pour construire la chaîne de raisonnement :

```text
checkout
→ payments API
→ cluster-3
→ maintenance vendredi
```

C’est une limite classique du RAG standard.

La recherche vectorielle retrouve des textes proches de la question. Elle ne garantit pas qu’elle retrouve tous les liens nécessaires au raisonnement.

C’est une des raisons pour lesquelles nous étudierons plus tard :

```text
Graph RAG ;
retrieval multi-hop ;
query expansion ;
agentic retrieval ;
recherche hybride.
```

---

## 2.11. Embeddings et granularité documentaire

Un embedding dépend fortement de ce que nous lui donnons à encoder.

Comparons trois niveaux.

### 2.11.1. Phrase seule

```text
L’API payments tourne sur le cluster-3.
```

L’embedding sera précis, mais le contexte est limité.

### 2.11.2. Paragraphe

```text
Le service checkout utilise l’API payments pour déclencher les paiements.
L’API payments tourne sur le cluster-3.
Le cluster-3 sera en maintenance vendredi.
```

L’embedding contient davantage de contexte et permet peut-être de retrouver plus facilement la relation.

### 2.11.3. Document entier

Si nous encodons un document entier de vingt pages, l’information spécifique risque d’être diluée.

Le vecteur représentera une moyenne sémantique approximative de nombreux sujets.

C’est pourquoi le **chunking** est fondamental.

Nous devons choisir une granularité qui préserve assez de contexte sans produire des morceaux trop gros.

---

## 2.12. Embeddings et métadonnées

Un embedding représente le contenu textuel, mais un bon système RAG ne stocke pas seulement un vecteur.

Il stocke aussi des métadonnées.

Par exemple :

```text
texte du chunk ;
embedding ;
document source ;
titre du document ;
section ;
page ;
date de mise à jour ;
auteur ;
type de document ;
droits d’accès ;
URL ;
version ;
tags métier.
```

Ces métadonnées sont essentielles.

Elles permettent :

```text
de citer les sources ;
de filtrer les résultats ;
de respecter les droits d’accès ;
de favoriser les documents récents ;
de limiter la recherche à un périmètre ;
d’expliquer la réponse.
```

Exemple :

```text
Question :
Quelle est la procédure actuelle de déploiement en production ?

Filtre :
type_document = "runbook"
environnement = "production"
date_version > 2025-01-01
```

Sans métadonnées, le système peut retrouver un ancien document obsolète mais sémantiquement très proche.

---

## 2.13. Modèles d’embedding généralistes et spécialisés

Tous les modèles d’embedding ne se valent pas pour tous les usages.

### 2.13.1. Modèle généraliste

Un modèle généraliste fonctionne correctement sur beaucoup de sujets :

```text
questions générales ;
documentation courante ;
FAQ ;
articles ;
emails ;
contenus métiers variés.
```

Il est souvent suffisant pour un premier prototype.

### 2.13.2. Modèle spécialisé

Certains domaines nécessitent des embeddings plus adaptés :

```text
biomédecine ;
droit ;
code source ;
mathématiques ;
documentation technique ;
langues spécifiques ;
documents multilingues.
```

Par exemple, un modèle généraliste peut être moins performant pour comparer des extraits de code, des logs techniques ou des clauses juridiques complexes.

### 2.13.3. Modèle multilingue

Si le corpus contient plusieurs langues, nous devons vérifier que le modèle gère correctement le multilingue.

Exemple :

```text
Question en français :
Comment réinitialiser mon mot de passe ?

Document en anglais :
Password reset procedure for administrator accounts.
```

Un bon modèle multilingue peut rapprocher ces deux textes.

Un modèle monolingue ou mal adapté peut échouer.

---

## 2.14. Embeddings pour du code source

Dans un contexte informatique, nous pouvons aussi créer des embeddings pour du code.

Exemple :

```typescript
async function resetPassword(userId: string) {
  await securityService.validateReset(userId);
  return authService.resetPassword(userId);
}
```

Une question pourrait être :

```text
Où est validée la réinitialisation du mot de passe ?
```

Un embedding de code peut aider à retrouver cette fonction.

Cependant, le code pose des problèmes spécifiques :

```text
les noms exacts sont très importants ;
la structure syntaxique compte ;
les appels de fonctions créent des dépendances ;
les types donnent du sens ;
les imports sont importants ;
le graphe d’appel peut être plus pertinent que la proximité sémantique.
```

Pour le code, une approche purement vectorielle doit souvent être complétée par :

```text
recherche lexicale ;
analyse statique ;
graphe d’appels ;
indexation des symboles ;
LSP (Language Server Protocol);
AST (Abstract Syntaxt Tree);
métadonnées Git.
```

C’est un bon exemple de limite des embeddings seuls.

Ici :
**LSP** signifie **Language Server Protocol**.

C’est un protocole utilisé par les IDE comme VS Code, IntelliJ, Cursor, etc., pour interroger un “serveur de langage” capable de comprendre un langage de programmation.

Par exemple, grâce au LSP, on peut savoir :

- où une fonction est définie ;
- où elle est utilisée ;
- quel est le type d’une variable ;
- quelles erreurs de compilation existent ;
- quels symboles existent dans un fichier ;
- quelles méthodes appartiennent à une classe ;
- quelle documentation est associée à une fonction.

Dans un RAG pour du code, le LSP permet donc d’avoir une compréhension plus “IDE-like” du projet.

Exemple :

```
userService.createUser(...)
```

Avec le LSP, le système peut retrouver directement la définition de `createUser`, même si elle est dans un autre fichier.

**AST** signifie **Abstract Syntax Tree**, en français **arbre syntaxique abstrait**.

C’est une représentation structurée du code sous forme d’arbre.

Par exemple, ce code :

```
function add(a: number, b: number) {  return a + b;}
```

peut être transformé en une structure du genre :

```
FunctionDeclaration ├── name: add ├── parameters │    ├── a: number │    └── b: number └── body      └── ReturnStatement           └── BinaryExpression: a + b
```

L’AST ne voit pas le code comme du texte brut, mais comme une structure logique.

Dans un RAG pour du code, l’AST sert à découper intelligemment le code :

- par fonction ;
- par classe ;
- par méthode ;
- par import/export ;
- par bloc logique ;
- par dépendances entre éléments.

C’est beaucoup mieux que de découper naïvement tous les 500 tokens, car une fonction reste entière et cohérente.

---

## 2.15. Embeddings et bases vectorielles

Les embeddings sont souvent stockés dans une base vectorielle.

Une base vectorielle permet de chercher rapidement les vecteurs les plus proches d’un vecteur de requête.

Pour chaque chunk, nous stockons :

```text
id du chunk ;
texte ;
embedding ;
métadonnées.
```

Quand l’utilisateur pose une question, nous calculons :

```text
embedding(question)
```

Puis nous cherchons :

```text
les k vecteurs les plus proches
```

C’est ce qu’on appelle souvent une recherche **top-k**.

Exemple :

```text
top_k = 5
```

Le système récupère les cinq chunks les plus proches.

Mais le choix de `k` est important.

Si `k` est trop petit, nous risquons de manquer un passage important.

Si `k` est trop grand, nous donnons trop de bruit au LLM.

---

## 2.16. Notion de score de similarité

La base vectorielle renvoie généralement des résultats avec un score.

Exemple :

```text
Chunk 1 : score 0.89
Chunk 2 : score 0.83
Chunk 3 : score 0.76
Chunk 4 : score 0.52
Chunk 5 : score 0.49
```

Ces scores indiquent une proximité entre la question et les chunks.

Mais attention : un score n’est pas universel.

Un score de 0.82 peut être excellent avec un modèle et moyen avec un autre.

Nous devons donc éviter les règles naïves du type :

```text
si score > 0.8 alors le document est forcément pertinent
```

En pratique, nous devons calibrer les seuils sur nos données.

---

## 2.17. Prétraitement du texte

Avant de créer les embeddings, nous devons parfois nettoyer le texte.

Exemples de nettoyage utile :

```text
supprimer le HTML inutile ;
corriger les problèmes d’encodage ;
retirer les menus répétés ;
supprimer les pieds de page répétitifs ;
normaliser les espaces ;
conserver les titres ;
conserver les tableaux sous une forme lisible ;
préserver les métadonnées importantes.
```

Mais il faut éviter de nettoyer trop brutalement.

Dans les anciens pipelines NLP, on supprimait souvent :

```text
stop words ;
ponctuation ;
accents ;
majuscules ;
formes grammaticales.
```

Avec les modèles modernes, ce n’est pas toujours souhaitable.

La ponctuation, les titres, les négations ou les formulations exactes peuvent avoir du sens.

Exemple :

```text
Le service doit être redémarré.
Le service ne doit pas être redémarré.
```

Si nous supprimons trop d’éléments, nous risquons de perdre une information critique.

---

## 2.18. Les limites générales des embeddings

Les embeddings sont puissants, mais ils ont plusieurs limites.

### 2.18.1. Ils capturent une proximité, pas un raisonnement

Un embedding rapproche des textes similaires, mais il ne construit pas nécessairement une chaîne logique.

### 2.18.2. Ils peuvent rater les termes exacts

Un identifiant rare, un code d’erreur ou un nom de fichier peut être mal traité par la recherche sémantique.

Exemple :

```text
ERR_MODULE_NOT_FOUND
P2025
docker-compose.prod.yml
UserRepositoryImpl
```

Pour ces cas, la recherche lexicale est souvent indispensable.

### 2.18.3. Ils peuvent rapprocher des textes trop généraux

Deux textes peuvent sembler proches parce qu’ils parlent du même domaine, sans que l’un réponde vraiment à l’autre.

Exemple :

```text
Question :
Comment corriger l’erreur Prisma P2025 ?

Chunk :
Prisma est un ORM pour TypeScript permettant d’interagir avec une base de données.
```

Le chunk parle de Prisma, mais il ne répond pas à la question.

### 2.18.4. Ils dépendent fortement du modèle

Changer de modèle d’embedding peut changer les résultats du retrieval.

Il faut donc considérer le modèle d’embedding comme un composant critique du système.

### 2.18.5. Ils vieillissent avec l’index

Si nous changeons de modèle d’embedding, nous devons généralement réindexer les documents.

On ne mélange pas proprement des vecteurs produits par plusieurs modèles différents dans le même espace.

---

## 2.19. Expérience pédagogique simple

Pour bien comprendre les embeddings, nous pouvons proposer une petite expérience.

Nous prenons les phrases suivantes :

```text
1. Comment réinitialiser mon mot de passe ?
2. Procédure de changement des identifiants utilisateur.
3. Le cluster Kubernetes sera redémarré vendredi.
4. PostgreSQL utilise des transactions ACID.
5. J’ai oublié mon password administrateur.
6. Les factures sont payables sous trente jours.
```

Nous calculons les embeddings de chaque phrase.

Puis nous cherchons les phrases les plus proches de :

```text
Je n’arrive plus à me connecter à mon compte.
```

Nous devrions obtenir en tête :

```text
1. Comment réinitialiser mon mot de passe ?
2. Procédure de changement des identifiants utilisateur.
3. J’ai oublié mon password administrateur.
```

Ce type d’exercice permet de comprendre concrètement que la recherche ne dépend pas uniquement des mots exacts, mais aussi de la proximité sémantique.

---

## 2.20. Mini-TP proposé

### Objectif

Nous voulons construire une première expérience de recherche sémantique.

### Données

Nous créons un petit corpus :

```text
doc_1 : La procédure de réinitialisation du mot de passe nécessite une validation sécurité.
doc_2 : Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.
doc_3 : Les factures doivent être payées sous trente jours.
doc_4 : Le service checkout utilise l’API payments.
doc_5 : L’API payments est déployée sur le cluster-3.
```

### Étapes

Nous allons :

```text
1. Choisir un modèle d’embedding.
2. Vectoriser chaque document.
3. Vectoriser une question utilisateur.
4. Calculer la similarité entre la question et chaque document.
5. Trier les documents par score.
6. Observer les résultats.
```

### Questions à tester

```text
Comment changer un mot de passe ?
Le service checkout sera-t-il touché vendredi ?
Quand faut-il payer une facture ?
Quel service utilise l’API payments ?
```

### Analyse attendue

Nous devrons observer que certaines questions sont faciles :

```text
Quand faut-il payer une facture ?
```

devrait retrouver :

```text
Les factures doivent être payées sous trente jours.
```

Mais la question :

```text
Le service checkout sera-t-il touché vendredi ?
```

peut être plus difficile, car elle nécessite plusieurs documents :

```text
checkout utilise payments
payments est sur cluster-3
cluster-3 est en maintenance vendredi
```

Cette expérience prépare le terrain pour comprendre les limites du RAG standard.

---

## 2.21. Synthèse du chapitre

Dans ce chapitre, nous avons introduit les embeddings.

Nous avons vu qu’un embedding est une représentation vectorielle d’un texte.

Nous pouvons résumer ainsi :

```text
Un embedding transforme un texte en vecteur numérique.
Deux textes proches en sens doivent avoir des vecteurs proches.
```

Nous avons compris que les embeddings permettent la recherche sémantique, c’est-à-dire la recherche par proximité de sens plutôt que seulement par mots-clés.

Nous avons aussi vu que les embeddings sont au cœur du RAG standard :

```text
documents → chunks → embeddings → index vectoriel
question → embedding → recherche des chunks proches
```

Mais nous avons insisté sur leurs limites :

```text
ils ne prouvent pas la vérité ;
ils ne garantissent pas le raisonnement ;
ils peuvent rater les faits intermédiaires ;
ils peuvent être moins bons sur les identifiants exacts ;
ils dépendent du modèle utilisé ;
ils doivent être complétés par des métadonnées, du lexical, du reranking ou du graphe.
```

Le message principal de ce chapitre est donc le suivant :

```text
Les embeddings sont le mécanisme qui permet au RAG de retrouver des passages proches en sens.
Mais retrouver un passage proche ne signifie pas toujours retrouver le passage nécessaire pour répondre correctement.
```

Dans le prochain chapitre, nous étudierons plus précisément la **similarité cosinus** et la recherche vectorielle, c’est-à-dire la manière dont nous comparons mathématiquement ces embeddings.

---

# Chapitre 3 — Similarité cosinus et recherche vectorielle

## 3.1. Introduction

Dans le chapitre précédent, nous avons étudié les **embeddings**. Nous avons vu qu’un embedding transforme un texte en vecteur numérique, de façon à placer des textes proches en sens dans des zones proches d’un espace vectoriel.

Nous savons maintenant représenter une phrase, un paragraphe ou un morceau de document sous forme de vecteur.

Mais une nouvelle question apparaît :

```text
Comment mesurer concrètement la proximité entre deux embeddings ?
```

Si nous avons :

```text
embedding(question)
embedding(chunk_1)
embedding(chunk_2)
embedding(chunk_3)
```

nous devons être capables de dire quel chunk est le plus proche de la question.

C’est là qu’intervient la **similarité cosinus**.

Dans ce chapitre, nous allons comprendre :

```text
ce qu’est la similarité cosinus ;
pourquoi elle est très utilisée en NLP et en RAG ;
comment elle permet de comparer deux embeddings ;
comment fonctionne une recherche vectorielle ;
ce qu’est une recherche top-k ;
quelles sont les limites des scores de similarité.
```

Nous allons donc passer de l’idée générale :

```text
deux textes proches ont des vecteurs proches
```

à un mécanisme concret :

```text
nous calculons un score de proximité entre vecteurs,
puis nous récupérons les passages les mieux classés.
```

---

## 3.2. Rappel : un texte devient un vecteur

Prenons une question utilisateur :

```text
Comment réinitialiser mon mot de passe ?
```

Un modèle d’embedding transforme cette question en vecteur :

```text
q = [0.12, -0.45, 0.88, ..., 0.03]
```

Dans notre base documentaire, nous avons plusieurs chunks :

```text
chunk_1 :
La procédure de réinitialisation du mot de passe nécessite une validation sécurité.

chunk_2 :
Le cluster-3 sera en maintenance vendredi soir.

chunk_3 :
Les factures sont payables sous trente jours.

chunk_4 :
L’API payments est utilisée par le service checkout.
```

Chaque chunk est lui aussi transformé en vecteur :

```text
v1 = embedding(chunk_1)
v2 = embedding(chunk_2)
v3 = embedding(chunk_3)
v4 = embedding(chunk_4)
```

Le problème devient alors mathématique :

```text
Quel vecteur vi est le plus proche du vecteur q ?
```

Si le modèle d’embedding fonctionne correctement, nous nous attendons à ce que :

```text
v1
```

soit le plus proche de :

```text
q
```

car `chunk_1` parle bien de réinitialisation de mot de passe.

---

## 3.3. Intuition géométrique

Pour comprendre la similarité cosinus, imaginons d’abord un espace à deux dimensions.

Chaque texte est représenté par un point ou un vecteur dans cet espace.

```text
                         sécurité / compte
                               ↑
                               |
          mot de passe         |      reset identifiants
                x              |             x
                               |
                               |
                               |
                               |
facture x                      |
                               |
-------------------------------→ infrastructure
                               |
                               |        cluster maintenance
                               |               x
```

Dans cet exemple, les textes sur le mot de passe sont proches les uns des autres. Les textes sur les factures ou l’infrastructure sont plus éloignés.

En réalité, les embeddings ne vivent pas dans un espace à deux dimensions, mais dans un espace à plusieurs centaines ou milliers de dimensions.

Cependant, l’intuition reste identique :

```text
nous voulons mesurer la proximité géométrique entre vecteurs.
```

---

## 3.4. Distance euclidienne ou similarité cosinus ?

Pour comparer deux vecteurs, nous pourrions utiliser la distance euclidienne.

La distance euclidienne mesure la distance directe entre deux points.

Mais en NLP, on utilise très souvent la **similarité cosinus**, qui mesure plutôt l’orientation des vecteurs.

Pourquoi ?

Parce que dans beaucoup de cas, ce qui nous intéresse n’est pas principalement la longueur du vecteur, mais sa direction dans l’espace sémantique.

Deux vecteurs peuvent avoir des longueurs différentes, mais pointer dans une direction similaire.

Exemple simplifié :

```text
u = [1, 1]
v = [10, 10]
```

Ces deux vecteurs n’ont pas la même longueur, mais ils pointent exactement dans la même direction.

La similarité cosinus considère qu’ils sont très similaires.

C’est utile car nous voulons surtout savoir si deux textes portent le même type de sens, pas si leurs vecteurs ont exactement la même amplitude.

---

## 3.5. Définition de la similarité cosinus

La similarité cosinus entre deux vecteurs `u` et `v` est définie par :

```text
cos(u, v) = (u · v) / (||u|| × ||v||)
```

où :

```text
u · v
```

est le produit scalaire entre les deux vecteurs, et :

```text
||u||
||v||
```

sont les normes des vecteurs.

Le produit scalaire mesure à quel point deux vecteurs vont dans la même direction.

La norme mesure la longueur d’un vecteur.

La division par les normes permet de normaliser le résultat.

---

## 3.6. Interprétation du score

La similarité cosinus donne une valeur comprise entre `-1` et `1`.

En simplifiant :

```text
1    → vecteurs dans la même direction
0    → vecteurs orthogonaux, donc peu liés
-1   → vecteurs dans des directions opposées
```

Dans beaucoup de systèmes d’embeddings modernes, nous rencontrons surtout des scores entre `0` et `1`.

Nous pouvons interpréter grossièrement :

```text
score proche de 1    → textes très proches
score autour de 0.5  → proximité moyenne ou incertaine
score proche de 0    → textes peu liés
```

Mais attention : ces seuils ne sont pas universels.

Un score de `0.82` peut être excellent avec un modèle et seulement moyen avec un autre. Il faut calibrer les seuils sur un corpus réel.

---

## 3.7. Exemple simple avec des vecteurs courts

Prenons deux vecteurs en deux dimensions :

```text
u = [1, 0]
v = [1, 0]
```

Ils pointent dans la même direction.

Leur similarité cosinus vaut :

```text
cos(u, v) = 1
```

Prenons maintenant :

```text
u = [1, 0]
v = [0, 1]
```

Ils sont perpendiculaires.

Leur similarité cosinus vaut :

```text
cos(u, v) = 0
```

Enfin :

```text
u = [1, 0]
v = [-1, 0]
```

Ils pointent dans des directions opposées.

Leur similarité cosinus vaut :

```text
cos(u, v) = -1
```

Dans un système RAG, les vecteurs sont beaucoup plus grands, mais le principe reste le même.

---

## 3.8. Exemple textuel

Imaginons une question :

```text
Question :
Comment réinitialiser mon mot de passe ?
```

Et trois chunks :

```text
Chunk A :
La procédure de changement du mot de passe utilisateur est décrite dans le guide sécurité.

Chunk B :
Le cluster Kubernetes sera mis à jour vendredi soir.

Chunk C :
Les factures doivent être réglées sous trente jours.
```

Après calcul des embeddings, nous pouvons obtenir des scores fictifs :

```text
similarité(question, chunk A) = 0.91
similarité(question, chunk B) = 0.34
similarité(question, chunk C) = 0.18
```

Le système récupère donc le chunk A.

C’est le mécanisme de base d’un RAG standard.

---

## 3.9. De la comparaison simple à la recherche vectorielle

Comparer une question avec trois chunks est facile.

Mais dans un vrai système, nous pouvons avoir :

```text
10 000 chunks ;
100 000 chunks ;
1 million de chunks ;
100 millions de chunks.
```

Nous ne voulons pas recalculer naïvement toutes les comparaisons de manière coûteuse à chaque question.

Nous utilisons donc une **base vectorielle** ou un **index vectoriel**.

Son rôle est de stocker les embeddings et de retrouver rapidement les vecteurs les plus proches d’une requête.

Pipeline :

```text
Documents
   ↓
Chunks
   ↓
Embeddings
   ↓
Index vectoriel
```

Lors d’une question :

```text
Question
   ↓
Embedding de la question
   ↓
Recherche dans l’index vectoriel
   ↓
Chunks les plus proches
```

La recherche vectorielle est donc le mécanisme qui permet de retrouver rapidement les documents pertinents à grande échelle.

---

## 3.10. Recherche top-k

Un retriever vectoriel renvoie souvent les `k` résultats les plus proches.

C’est ce qu’on appelle une recherche **top-k**.

Exemple :

```text
top_k = 5
```

Le système retourne les cinq chunks ayant les meilleurs scores.

Exemple fictif :

```text
1. chunk_17   score = 0.91
2. chunk_42   score = 0.87
3. chunk_08   score = 0.79
4. chunk_95   score = 0.72
5. chunk_11   score = 0.69
```

Ces chunks seront ensuite injectés dans le prompt du LLM.

Le choix de `k` est important.

Si `k` est trop petit, nous risquons de manquer une information nécessaire.

Si `k` est trop grand, nous risquons de donner trop de bruit au modèle.

---

## 3.11. Le compromis du top-k

Prenons une question simple :

```text
Quel est le délai de paiement des factures ?
```

Si le bon chunk est très proche, un `top_k = 3` peut suffire.

Mais pour une question complexe :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

il faut peut-être récupérer plusieurs passages :

```text
checkout utilise payments API ;
payments API tourne sur cluster-3 ;
cluster-3 est en maintenance vendredi.
```

Si nous mettons :

```text
top_k = 1
```

nous récupérerons peut-être seulement :

```text
checkout utilise payments API.
```

et la réponse sera incomplète.

Si nous mettons :

```text
top_k = 20
```

nous récupérerons peut-être les bons passages, mais aussi beaucoup de bruit documentaire.

Le `top_k` est donc un paramètre d’équilibre entre :

```text
rappel : récupérer tous les éléments utiles ;
précision : éviter de récupérer des éléments inutiles.
```

---

## 3.12. Score de similarité et seuil minimal

En plus du top-k, nous pouvons utiliser un seuil minimal.

Par exemple :

```text
ne garder que les chunks dont le score est supérieur à 0.75
```

Cela évite de donner au LLM des documents trop éloignés.

Exemple :

```text
chunk_17   score = 0.91 → conservé
chunk_42   score = 0.87 → conservé
chunk_08   score = 0.79 → conservé
chunk_95   score = 0.52 → rejeté
chunk_11   score = 0.41 → rejeté
```

Mais là encore, attention : un seuil fixe peut être dangereux.

Certaines questions ont naturellement des scores faibles parce qu’elles sont formulées de manière vague ou parce que les documents sont différents lexicalement.

D’autres questions peuvent avoir des scores élevés sur des documents proches mais inutiles.

Nous devons donc tester ces seuils sur des cas réels.

---

## 3.13. Similarité élevée ne signifie pas réponse correcte

C’est une limite fondamentale.

Un chunk peut être très proche de la question, sans contenir la réponse.

Exemple :

```text
Question :
Comment corriger l’erreur Prisma P2025 ?

Chunk récupéré :
Prisma est un ORM moderne pour Node.js et TypeScript.
```

Le chunk parle bien de Prisma, donc il peut être proche.

Mais il ne répond pas à la question.

Autre exemple :

```text
Question :
Le service checkout sera-t-il affecté vendredi ?

Chunk récupéré :
Le service checkout gère la validation des paniers.
```

Le chunk parle bien de checkout, mais il ne dit rien sur vendredi, ni sur les dépendances, ni sur une maintenance.

Nous devons donc distinguer :

```text
similarité sémantique
```

et :

```text
pertinence pour répondre.
```

Ce sont deux notions proches, mais pas identiques.

---

## 3.14. Similarité faible ne signifie pas inutilité

À l’inverse, un chunk peut avoir une similarité modérée ou faible avec la question, tout en étant indispensable.

Reprenons l’exemple :

```text
Question :
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Faits disponibles :

```text
1. Le service checkout utilise l’API payments.
2. L’API payments tourne sur le cluster-3.
3. Le cluster-3 sera en maintenance vendredi.
```

Le fait 2 :

```text
L’API payments tourne sur le cluster-3.
```

peut avoir un score plus faible, parce qu’il ne contient pas les mots :

```text
checkout
maintenance
vendredi
affecté
```

Pourtant, il est indispensable.

La recherche vectorielle standard peut donc échouer sur les questions multi-hop (plusieurs sauts).

Cela ne signifie pas que les embeddings sont inutiles. Cela signifie qu’ils ne suffisent pas toujours.

---

## 3.15. Recherche vectorielle exacte et approximative

Lorsque nous avons peu de vecteurs, nous pouvons comparer la question à tous les chunks.

C’est une recherche exacte.

Mais à grande échelle, cette approche peut être coûteuse.

Les bases vectorielles utilisent souvent des algorithmes de recherche approximative des plus proches voisins, appelés **ANN**, pour _Approximate Nearest Neighbors_.

L’idée est d’accepter parfois une approximation pour gagner beaucoup en performance.

Nous cherchons donc :

```text
les vecteurs probablement les plus proches
```

plutôt que :

```text
les vecteurs mathématiquement les plus proches avec certitude absolue.
```

En pratique, cela permet de faire des recherches rapides dans de très grands corpus.

Mais cela ajoute un compromis :

```text
rapidité
vs
exactitude de la recherche
```

---

## 3.16. Index vectoriel : intuition

Un index vectoriel organise les vecteurs pour éviter de comparer la requête à tous les vecteurs un par un.

Il peut utiliser différentes structures ou techniques, comme :

```text
graphes de proximité ;
quantification ;
partitionnement ;
arbres ;
clustering ;
HNSW ;
IVF ;
PQ.
```

Nous n’avons pas besoin de maîtriser tous ces algorithmes dès maintenant, mais nous devons comprendre l’idée :

```text
l’index prépare l’espace vectoriel pour rendre les recherches rapides.
```

Sans index, chercher dans un million de chunks peut être coûteux.

Avec un index bien construit, la recherche peut être beaucoup plus rapide.

---

## 3.17. Métadonnées et filtrage

Une recherche vectorielle pure peut retourner des chunks proches, mais pas forcément autorisés, récents ou pertinents pour le contexte métier.

Nous devons donc utiliser les métadonnées.

Exemple de métadonnées :

```text
source = "documentation production"
type = "runbook"
date = "2026-05-10"
environment = "prod"
service = "checkout"
permissions = ["team-payments", "sre"]
```

Une recherche peut alors combiner :

```text
similarité vectorielle
+
filtres par métadonnées
```

Exemple :

```text
Question :
Comment redémarrer le service checkout en production ?

Filtre :
environment = "prod"
type = "runbook"
service = "checkout"
```

Cela évite de récupérer un document de test, une ancienne procédure ou une note non validée.

---

## 3.18. Recherche vectorielle et droits d’accès

Dans un système professionnel, les droits d’accès sont critiques.

Un utilisateur ne doit jamais recevoir une réponse fondée sur un document qu’il n’a pas le droit de lire.

Il ne suffit donc pas de faire :

```text
chercher les chunks les plus proches
```

Il faut faire :

```text
chercher les chunks les plus proches parmi ceux que l’utilisateur a le droit de consulter
```

Cela peut être géré :

```text
avant la recherche ;
pendant la recherche ;
après la recherche ;
ou par séparation des index.
```

Mais dans tous les cas, la règle est claire :

```text
le RAG ne doit pas devenir un moyen de contourner les permissions documentaires.
```

---

## 3.19. Recherche vectorielle et documents obsolètes

Un autre problème important est la fraîcheur des documents.

Supposons que nous ayons deux documents :

```text
Document A, 2023 :
Le service checkout tourne sur cluster-1.

Document B, 2026 :
Le service checkout tourne désormais sur cluster-3.
```

Les deux documents peuvent être proches de la question :

```text
Sur quel cluster tourne checkout ?
```

Si nous ne tenons pas compte de la date ou de la version, le système peut récupérer l’ancien document.

Les métadonnées permettent de gérer cela :

```text
date de mise à jour ;
version ;
statut du document ;
document archivé ou actif ;
environnement concerné.
```

Un bon RAG ne doit pas seulement retrouver un document sémantiquement proche. Il doit retrouver le bon document dans le bon contexte.

---

## 3.20. Exemple complet de recherche vectorielle

Nous construisons un mini corpus :

```text
doc_1 :
La procédure de réinitialisation du mot de passe nécessite une validation sécurité.

doc_2 :
Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.

doc_3 :
Les factures doivent être payées sous trente jours.

doc_4 :
Le service checkout utilise l’API payments.

doc_5 :
L’API payments est déployée sur le cluster-3.
```

Question :

```text
Le service checkout sera-t-il touché par la maintenance de vendredi ?
```

Résultats possibles d’une recherche vectorielle :

```text
1. doc_4 : score 0.84
   Le service checkout utilise l’API payments.

2. doc_2 : score 0.81
   Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.

3. doc_5 : score 0.58
   L’API payments est déployée sur le cluster-3.

4. doc_1 : score 0.22
   La procédure de réinitialisation du mot de passe nécessite une validation sécurité.

5. doc_3 : score 0.15
   Les factures doivent être payées sous trente jours.
```

Si nous prenons :

```text
top_k = 2
```

nous récupérons `doc_4` et `doc_2`, mais nous ratons `doc_5`.

Le LLM reçoit alors :

```text
checkout utilise payments API ;
cluster-3 est en maintenance vendredi.
```

Mais il ne reçoit pas le lien :

```text
payments API est déployée sur cluster-3.
```

Il ne peut donc pas conclure de manière rigoureuse.

Si nous prenons :

```text
top_k = 3
```

nous récupérons les trois documents nécessaires.

Cet exemple montre que la configuration du retrieval influence directement la qualité de la réponse.

---

## 3.21. Le rôle du reranking

Une recherche vectorielle est souvent une première étape.

Elle peut récupérer un ensemble assez large de documents :

```text
top_k = 30
```

Puis un modèle plus coûteux mais plus précis peut les reclasser.

C’est le **reranking**.

Pipeline :

```text
Question
   ↓
Recherche vectorielle large
   ↓
30 chunks candidats
   ↓
Reranker
   ↓
5 meilleurs chunks
   ↓
LLM
```

Le retriever vectoriel sert à chercher rapidement.

Le reranker sert à juger plus finement la pertinence de chaque chunk par rapport à la question.

Nous étudierons cette étape plus en détail dans un chapitre ultérieur.

---

## 3.22. Recherche vectorielle vs recherche hybride

La recherche vectorielle est très utile, mais elle n’est pas toujours suffisante.

Exemple :

```text
Question :
Que signifie l’erreur ERR_MODULE_NOT_FOUND ?
```

Le terme exact `ERR_MODULE_NOT_FOUND` est déterminant.

Une recherche lexicale peut être plus fiable qu’une recherche vectorielle.

À l’inverse :

```text
Question :
Je n’arrive plus à me connecter à mon compte, que faire ?
```

La recherche vectorielle peut retrouver un document qui parle de :

```text
réinitialisation du mot de passe
```

même si la question ne contient pas ces mots.

En production, une bonne stratégie est souvent hybride :

```text
score final = score lexical + score vectoriel + score de reranking
```

Cela permet de combiner :

```text
la précision des mots exacts ;
la souplesse de la recherche sémantique ;
la finesse d’un reranker.
```

---

## 3.23. Les erreurs fréquentes dans l’interprétation des scores

### 3.23.1. Croire qu’un score élevé garantit une bonne réponse

Un score élevé indique une proximité, pas une réponse suffisante.

### 3.23.2. Croire qu’un seuil universel existe

Il n’existe pas de seuil magique valable pour tous les modèles et tous les corpus.

### 3.23.3. Comparer les scores de modèles différents

Les scores produits par deux modèles d’embedding différents ne sont pas directement comparables.

### 3.23.4. Oublier les métadonnées

Un document proche mais obsolète, non autorisé ou hors contexte peut être dangereux.

### 3.23.5. Confondre retrieval et raisonnement

La recherche vectorielle sélectionne du contexte. Elle ne fait pas, à elle seule, le raisonnement complet.

---

## 3.24. Mini-TP : calculer une similarité cosinus

### Objectif

Nous voulons comprendre concrètement comment comparer deux vecteurs.

### Données

Nous prenons deux vecteurs simples :

```text
u = [1, 2, 3]
v = [2, 4, 6]
```

Ces deux vecteurs pointent dans la même direction, car :

```text
v = 2u
```

Nous nous attendons donc à une similarité cosinus de `1`.

Puis nous prenons :

```text
u = [1, 0]
v = [0, 1]
```

Ces vecteurs sont orthogonaux. Nous nous attendons à une similarité de `0`.

### Code Python

```python
import numpy as np

def cosine_similarity(u, v):
    u = np.array(u)
    v = np.array(v)

    dot_product = np.dot(u, v)
    norm_u = np.linalg.norm(u)
    norm_v = np.linalg.norm(v)

    return dot_product / (norm_u * norm_v)

print(cosine_similarity([1, 2, 3], [2, 4, 6]))
print(cosine_similarity([1, 0], [0, 1]))
print(cosine_similarity([1, 0], [-1, 0]))
```

### Résultat attendu

```text
1.0
0.0
-1.0
```

Ce TP montre que la similarité cosinus mesure bien l’orientation des vecteurs.

---

## 3.25. Mini-TP : simuler une recherche vectorielle

### Objectif

Nous voulons simuler le comportement d’un retriever.

### Corpus

```python
documents = [
    "La procédure de réinitialisation du mot de passe nécessite une validation sécurité.",
    "Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.",
    "Les factures doivent être payées sous trente jours.",
    "Le service checkout utilise l’API payments.",
    "L’API payments est déployée sur le cluster-3."
]
```

Dans un vrai système, nous utiliserions un modèle d’embedding.

Pour un exercice pédagogique, nous pouvons imaginer que nous avons déjà calculé les scores :

```python
scores = [
    0.22,
    0.81,
    0.15,
    0.84,
    0.58
]
```

Question :

```text
Le service checkout sera-t-il touché par la maintenance de vendredi ?
```

Nous voulons trier les documents par score.

### Code Python

```python
documents = [
    "La procédure de réinitialisation du mot de passe nécessite une validation sécurité.",
    "Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.",
    "Les factures doivent être payées sous trente jours.",
    "Le service checkout utilise l’API payments.",
    "L’API payments est déployée sur le cluster-3."
]

scores = [0.22, 0.81, 0.15, 0.84, 0.58]

ranked = sorted(
    zip(documents, scores),
    key=lambda item: item[1],
    reverse=True
)

for doc, score in ranked:
    print(score, "→", doc)
```

### Résultat attendu

```text
0.84 → Le service checkout utilise l’API payments.
0.81 → Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.
0.58 → L’API payments est déployée sur le cluster-3.
0.22 → La procédure de réinitialisation du mot de passe nécessite une validation sécurité.
0.15 → Les factures doivent être payées sous trente jours.
```

### Analyse

Nous voyons que les trois premiers documents permettent de construire la réponse complète.

Mais si nous avions choisi :

```text
top_k = 2
```

nous aurions raté le troisième document, pourtant indispensable.

Ce mini-TP montre concrètement l’importance du choix de `top_k`.

---

## 3.26. Questions de compréhension

À la fin de ce chapitre, nous devons être capables de répondre aux questions suivantes :

```text
Qu’est-ce que la similarité cosinus ?
Pourquoi l’utilise-t-on pour comparer des embeddings ?
Quelle est la différence entre distance et similarité ?
Que signifie un score proche de 1 ?
Pourquoi un score élevé ne garantit-il pas une bonne réponse ?
Qu’est-ce qu’une recherche top-k ?
Pourquoi le choix de k est-il important ?
Pourquoi faut-il utiliser des métadonnées ?
Pourquoi la recherche vectorielle peut-elle rater des faits intermédiaires ?
Pourquoi la recherche hybride est-elle souvent préférable en production ?
```

---

## 3.27. Synthèse du chapitre

Dans ce chapitre, nous avons étudié la similarité cosinus et la recherche vectorielle.

Nous avons vu que la similarité cosinus permet de comparer deux embeddings en mesurant l’orientation de leurs vecteurs.

Nous pouvons résumer ainsi :

```text
embedding(question)
        ↓
comparaison par similarité cosinus
        ↓
classement des chunks
        ↓
récupération des meilleurs résultats
```

Nous avons aussi vu que cette méthode est au cœur du RAG standard.

Cependant, nous avons insisté sur ses limites :

```text
un score élevé ne garantit pas une réponse correcte ;
un score faible ne signifie pas toujours que le passage est inutile ;
les seuils doivent être calibrés ;
le top-k influence fortement la réponse ;
les métadonnées sont indispensables ;
la recherche vectorielle peut rater les faits intermédiaires ;
la recherche hybride est souvent plus robuste.
```

Le message principal du chapitre est donc le suivant :

```text
La similarité cosinus permet de retrouver des textes proches en sens,
mais la proximité sémantique ne suffit pas toujours à garantir la pertinence documentaire.
```

Dans le chapitre suivant, nous construirons un **RAG standard complet** : ingestion, chunking, embeddings, indexation, retrieval, prompt augmenté et génération.

---

Voici le **chapitre 4**.

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

# Chapitre 5 — Prompt augmenté et génération sourcée

## 5.1. Introduction

Dans le chapitre précédent, nous avons construit un pipeline RAG standard complet.

Nous avons vu comment passer de documents bruts à un index vectoriel, puis comment récupérer des chunks pertinents à partir d’une question utilisateur.

Mais récupérer les bons documents ne suffit pas.

Une fois les passages retrouvés, nous devons les transmettre correctement au LLM pour qu’il produise une réponse fiable, claire et vérifiable.

C’est l’objet de ce chapitre.

Nous allons étudier la partie **generation** du RAG :

```text
question utilisateur
        ↓
chunks récupérés
        ↓
construction du prompt augmenté
        ↓
appel au LLM
        ↓
réponse sourcée
```

Nous verrons que cette étape est critique, car un LLM peut encore :

```text
mal utiliser les documents ;
ignorer une source importante ;
surinterpréter un passage ;
mélanger deux sources ;
inventer une information absente ;
répondre avec trop d’assurance ;
ne pas signaler une incertitude.
```

Le rôle du prompt augmenté est donc de cadrer le comportement du modèle.

Nous ne voulons pas seulement une réponse bien écrite. Nous voulons une réponse :

```text
fondée sur les documents ;
traçable ;
vérifiable ;
adaptée à la question ;
capable de reconnaître les informations manquantes.
```

---

## 5.2. Rappel : le LLM n’est pas la source

Dans un système RAG, il est essentiel de distinguer deux rôles.

Les documents fournissent l’information.

Le LLM formule, synthétise et explique.

Autrement dit :

```text
Les sources apportent la connaissance.
Le LLM apporte la capacité de rédaction et de raisonnement linguistique.
```

Cette distinction évite une erreur fréquente : croire que la réponse est fiable parce qu’elle est bien formulée.

Une réponse peut être :

```text
fluide ;
structurée ;
convaincante ;
techniquement plausible ;
```

et pourtant fausse.

Dans un RAG, la qualité de la réponse dépend donc de deux choses :

```text
1. Les bons documents ont-ils été récupérés ?
2. Le LLM les a-t-il utilisés correctement ?
```

Si le retrieval échoue, le modèle ne dispose pas de l’information nécessaire.

Si la génération échoue, le modèle peut mal utiliser une information pourtant disponible.

Nous devons donc apprendre à contrôler cette seconde étape.

---

## 5.3. Qu’est-ce qu’un prompt augmenté ?

Un prompt augmenté est un prompt enrichi par des documents récupérés automatiquement.

Un prompt classique ressemble à ceci :

```text
Question :
Comment réinitialiser un mot de passe administrateur ?
```

Un prompt augmenté ressemble plutôt à ceci :

```text
Tu es un assistant documentaire.

Réponds à la question en utilisant uniquement le contexte fourni.
Si le contexte ne permet pas de répondre, dis-le explicitement.

Contexte :
[Source 1]
La procédure de réinitialisation d’un mot de passe administrateur nécessite
une validation par le responsable sécurité.

[Source 2]
Après validation, l’administrateur peut utiliser le script reset-admin-password.sh.

Question :
Comment réinitialiser un mot de passe administrateur ?
```

Le prompt augmenté contient donc généralement :

```text
une consigne de rôle ;
des règles de réponse ;
un contexte documentaire ;
la question utilisateur ;
parfois un format de sortie attendu ;
parfois des consignes de citation.
```

Le but est de transformer le LLM en assistant contrôlé par les sources.

---

## 5.4. Structure générale d’un prompt RAG

Un prompt RAG robuste peut être structuré en plusieurs blocs.

```text
[Système / rôle]
Tu es un assistant chargé de répondre à partir de documents internes.

[Règles]
- Utilise uniquement les informations présentes dans le contexte.
- Ne complète pas avec des informations non fournies.
- Si la réponse n’est pas disponible, dis-le.
- Cite les sources utilisées.

[Contexte documentaire]
Source 1...
Source 2...
Source 3...

[Question utilisateur]
...

[Format attendu]
Réponse courte, puis sources.
```

Cette structure aide le modèle à séparer :

```text
les consignes ;
les sources ;
la question ;
la forme attendue.
```

En pratique, cette séparation réduit les risques de confusion.

---

## 5.5. Exemple minimal de prompt augmenté

Prenons deux chunks récupérés.

```text
[Source A]
La procédure de réinitialisation d’un mot de passe administrateur nécessite
une validation par le responsable sécurité.

[Source B]
Après validation, l’administrateur peut utiliser le script reset-admin-password.sh.
```

Question :

```text
Comment réinitialiser un mot de passe administrateur ?
```

Prompt :

```text
Réponds à la question uniquement avec les sources fournies.
Si les sources ne permettent pas de répondre, indique que l’information
n’est pas disponible.

Sources :
[Source A]
La procédure de réinitialisation d’un mot de passe administrateur nécessite
une validation par le responsable sécurité.

[Source B]
Après validation, l’administrateur peut utiliser le script reset-admin-password.sh.

Question :
Comment réinitialiser un mot de passe administrateur ?
```

Réponse attendue :

```text
Pour réinitialiser un mot de passe administrateur, nous devons d’abord obtenir
une validation du responsable sécurité. Après cette validation, l’administrateur
peut utiliser le script reset-admin-password.sh.

Sources : Source A, Source B.
```

Cet exemple est simple, mais il contient déjà les principes essentiels :

```text
le modèle doit utiliser les sources ;
il ne doit pas inventer ;
il doit citer les passages utilisés.
```

---

## 5.6. La consigne “utilise uniquement le contexte” ne suffit pas toujours

Une erreur fréquente est de croire qu’il suffit d’écrire :

```text
Réponds uniquement à partir du contexte.
```

Cette instruction est utile, mais elle ne garantit pas totalement le comportement du modèle.

Le LLM peut encore :

```text
compléter avec sa connaissance générale ;
faire une inférence trop forte ;
oublier qu’une information est absente ;
répondre à une question voisine ;
produire une réponse plausible mais non sourcée.
```

Par exemple, si le contexte dit :

```text
Le service checkout utilise l’API payments.
```

et que la question est :

```text
Le service checkout sera-t-il affecté vendredi ?
```

Le modèle ne devrait pas conclure sans autre information.

Il devrait répondre :

```text
Le contexte indique que le service checkout utilise l’API payments,
mais il ne fournit pas d’information sur une maintenance vendredi.
Nous ne pouvons donc pas conclure à partir des sources fournies.
```

Un mauvais comportement serait :

```text
Oui, il sera probablement affecté vendredi.
```

Cette réponse est peut-être plausible, mais elle n’est pas justifiée par le contexte.

---

## 5.7. Répondre quand l’information est absente

Un bon système RAG doit savoir dire :

```text
Je ne trouve pas cette information dans les documents fournis.
```

Ce comportement est essentiel.

Dans beaucoup de cas, une réponse négative ou incertaine est meilleure qu’une réponse inventée.

Exemple :

```text
Contexte :
Le service checkout utilise l’API payments.

Question :
Sur quel cluster tourne l’API payments ?
```

Réponse correcte :

```text
Le contexte indique que le service checkout utilise l’API payments,
mais il ne précise pas sur quel cluster l’API payments est déployée.
```

Réponse incorrecte :

```text
L’API payments tourne probablement sur cluster-3.
```

Même si `cluster-3` est une hypothèse plausible dans notre exemple général, elle n’est pas présente dans le contexte fourni.

Le RAG doit résister à la tentation de compléter.

---

## 5.8. Répondre avec incertitude contrôlée

Il existe plusieurs niveaux de réponse selon la qualité du contexte.

### 5.8.1. Le contexte répond directement

```text
Question :
Quand a lieu la maintenance du cluster-3 ?

Contexte :
Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.

Réponse :
La maintenance du cluster-3 aura lieu vendredi soir à partir de 22 h.
```

### 5.8.2. Le contexte répond indirectement

```text
Question :
Le service checkout sera-t-il affecté vendredi ?

Contexte :
Le service checkout utilise l’API payments.
L’API payments est déployée sur le cluster-3.
Le cluster-3 sera en maintenance vendredi soir.
```

Réponse :

```text
Oui, le service checkout peut être affecté vendredi, car il utilise l’API payments,
qui est déployée sur le cluster-3, et ce cluster sera en maintenance vendredi soir.
```

Ici, la réponse repose sur une chaîne d’informations présentes dans le contexte.

### 5.8.3. Le contexte est insuffisant

```text
Question :
Le service checkout sera-t-il affecté vendredi ?

Contexte :
Le service checkout utilise l’API payments.
```

Réponse :

```text
Le contexte ne permet pas de conclure. Il indique seulement que le service
checkout utilise l’API payments, mais il ne donne aucune information sur une
maintenance vendredi ni sur l’infrastructure de l’API payments.
```

### 5.8.4. Le contexte est contradictoire

```text
Source A :
L’API payments est déployée sur le cluster-3.

Source B :
L’API payments est déployée sur le cluster-7.
```

Réponse :

```text
Les sources fournissent des informations contradictoires. La source A indique
que l’API payments est déployée sur le cluster-3, tandis que la source B indique
cluster-7. Il faut vérifier quelle source est la plus récente ou la plus fiable.
```

Un bon RAG ne doit pas masquer les contradictions.

---

## 5.9. Les citations

Les citations sont centrales dans un RAG.

Elles permettent de relier la réponse à ses sources.

Une réponse sourcée peut prendre plusieurs formes.

### 5.9.1. Citation simple en fin de phrase

```text
La maintenance du cluster-3 aura lieu vendredi soir à 22 h. [Source 2]
```

### 5.9.2. Sources en fin de réponse

```text
La maintenance du cluster-3 aura lieu vendredi soir à 22 h.

Sources :
- Runbook infrastructure, section "Maintenances planifiées"
```

### 5.9.3. Citation par affirmation

```text
Le service checkout utilise l’API payments [Source 1].
L’API payments est déployée sur le cluster-3 [Source 2].
Le cluster-3 sera en maintenance vendredi soir [Source 3].
```

Cette dernière forme est très utile pour les réponses techniques ou juridiques, car elle permet d’auditer chaque étape.

---

## 5.10. Granularité des citations

Une citation peut pointer vers :

```text
un document ;
une section ;
une page ;
un paragraphe ;
un chunk ;
une ligne ;
une cellule de tableau.
```

Plus la citation est précise, plus elle est utile.

Comparer :

```text
Source : documentation interne
```

avec :

```text
Source : Runbook Checkout Production, section 3.2, page 7
```

La seconde citation est bien plus vérifiable.

Dans un RAG de production, nous devons donc conserver des métadonnées suffisamment précises lors de l’indexation :

```text
document_id ;
titre ;
section ;
page ;
position ;
URL ;
date de version ;
extrait exact.
```

Sans ces métadonnées, les citations seront vagues.

---

## 5.11. Format de sortie

Le prompt peut demander un format de sortie précis.

Exemple simple :

```text
Réponds en trois parties :
1. Réponse courte
2. Justification
3. Sources
```

Exemple de réponse :

```text
Réponse courte :
Oui, le service checkout peut être affecté.

Justification :
Le service checkout utilise l’API payments. L’API payments est déployée sur
le cluster-3. Le cluster-3 sera en maintenance vendredi soir.

Sources :
- Source 1
- Source 2
- Source 3
```

Ce format est utile pour des étudiants, des développeurs ou des utilisateurs métier, car il sépare clairement :

```text
la conclusion ;
le raisonnement ;
les preuves.
```

Pour une API, nous pouvons aussi demander un format JSON.

Exemple :

```json
{
  "answer": "Oui, le service checkout peut être affecté vendredi.",
  "confidence": "high",
  "reasoning_summary": [
    "Le service checkout utilise l’API payments.",
    "L’API payments est déployée sur le cluster-3.",
    "Le cluster-3 sera en maintenance vendredi soir."
  ],
  "sources": ["source_1", "source_2", "source_3"]
}
```

Mais attention : il faut valider le JSON côté application, car le LLM peut produire un JSON mal formé.

---

## 5.12. Prompt pour réponse courte ou longue

Selon l’usage, nous ne voulons pas toujours le même niveau de détail.

### 5.12.1. Réponse courte

Prompt :

```text
Réponds en une phrase, puis cite la source.
```

Réponse :

```text
Oui, le service checkout peut être affecté vendredi, car il dépend de l’API
payments déployée sur le cluster-3, qui sera en maintenance. Source : ...
```

### 5.12.2. Réponse pédagogique

Prompt :

```text
Réponds de manière pédagogique, en expliquant le raisonnement étape par étape.
```

Réponse :

```text
Pour savoir si checkout sera affecté, nous devons suivre la chaîne de dépendance.
D’abord, la source 1 indique que checkout utilise payments API. Ensuite, la source 2
indique que payments API tourne sur cluster-3. Enfin, la source 3 indique que
cluster-3 sera en maintenance vendredi. Nous pouvons donc conclure que checkout
peut être affecté.
```

### 5.12.3. Réponse opérationnelle

Prompt :

```text
Réponds sous forme d’action opérationnelle pour une équipe SRE.
```

Réponse :

```text
Action recommandée :
- considérer checkout comme potentiellement impacté ;
- prévenir l’équipe responsable de payments API ;
- vérifier si une redondance existe hors cluster-3 ;
- surveiller les métriques checkout pendant la maintenance.
```

Le même contexte peut donc produire plusieurs formes de réponse selon le besoin.

---

## 5.13. Gestion des contradictions

Un corpus documentaire réel contient souvent des contradictions.

Exemple :

```text
Source A, 2024 :
Le service checkout utilise l’API payments v1.

Source B, 2026 :
Le service checkout utilise l’API payments v2.
```

Si le modèle reçoit les deux sources, il peut mélanger les informations.

Le prompt doit donc l’aider à gérer les conflits :

```text
Si les sources se contredisent, signale la contradiction.
Ne choisis pas arbitrairement une source sauf si les métadonnées indiquent
clairement laquelle est plus récente ou plus fiable.
```

Réponse attendue :

```text
Les sources ne sont pas cohérentes. La source A indique que checkout utilise
payments v1, tandis que la source B indique payments v2. La source B est plus
récente ; si elle est validée, elle doit probablement être privilégiée.
```

Mais la meilleure solution n’est pas seulement dans le prompt.

Il faut aussi gérer les métadonnées :

```text
date ;
statut ;
version ;
source officielle ou brouillon ;
document archivé ou actif.
```

---

## 5.14. Gestion des sources obsolètes

Une source obsolète peut être sémantiquement très proche de la question.

Exemple :

```text
Document 2023 :
Le service checkout est déployé sur cluster-1.

Document 2026 :
Le service checkout est déployé sur cluster-3.
```

Question :

```text
Sur quel cluster est déployé checkout ?
```

Un retrieval naïf peut récupérer les deux.

Le prompt peut demander :

```text
Privilégie les documents les plus récents lorsque plusieurs sources parlent
du même sujet.
Signale si une source semble obsolète.
```

Réponse possible :

```text
La source la plus récente indique que checkout est déployé sur cluster-3.
Une source plus ancienne mentionne cluster-1, mais elle semble obsolète.
```

Mais là encore, le prompt ne remplace pas une bonne indexation.

Nous devons marquer les documents :

```text
actif ;
archivé ;
brouillon ;
validé ;
obsolète ;
date de version.
```

---

## 5.15. Gestion des droits d’accès

Le prompt ne doit jamais être utilisé comme seul mécanisme de sécurité.

Il serait dangereux de faire :

```text
Voici des documents confidentiels.
Ne les montre pas à l’utilisateur s’il n’a pas le droit.
```

Le modèle pourrait se tromper.

La bonne règle est :

```text
Les documents non autorisés ne doivent jamais entrer dans le prompt.
```

La sécurité doit être appliquée avant la génération, au niveau du retrieval ou du filtrage.

Le prompt ne reçoit donc que les sources que l’utilisateur a le droit de consulter.

Ensuite seulement, le LLM génère la réponse.

---

## 5.16. Réduction des hallucinations

Le prompt augmenté aide à réduire les hallucinations en imposant des règles.

Exemple de consignes utiles :

```text
Utilise uniquement le contexte fourni.
Ne complète pas avec des informations externes.
Si une information manque, dis-le.
Ne donne pas de valeur, date, nom ou référence non présent dans les sources.
Cite les sources utilisées.
Si les sources sont contradictoires, signale-le.
```

Ces règles ne garantissent pas un comportement parfait, mais elles améliorent la fiabilité.

Nous pouvons aussi demander au modèle de séparer :

```text
ce qui est explicitement indiqué ;
ce qui est déduit ;
ce qui est inconnu.
```

Exemple :

```text
Éléments explicitement indiqués :
- checkout utilise payments API.
- payments API est déployée sur cluster-3.
- cluster-3 sera en maintenance vendredi.

Déduction :
- checkout peut être affecté par la maintenance.

Information manquante :
- le contexte ne précise pas s’il existe une redondance hors cluster-3.
```

Ce format est très utile pour éviter les conclusions trop fortes.

---

## 5.17. Déduction contrôlée

Un RAG ne doit pas forcément se limiter à recopier les sources.

Il peut aussi faire des déductions, mais elles doivent être contrôlées.

Exemple :

```text
Source 1 :
Le service checkout utilise l’API payments.

Source 2 :
L’API payments est déployée sur cluster-3.

Source 3 :
Cluster-3 sera en maintenance vendredi.
```

La conclusion :

```text
Le service checkout peut être affecté vendredi.
```

n’est pas écrite explicitement dans une source, mais elle est raisonnablement déduite.

Nous pouvons donc demander au modèle de distinguer :

```text
faits extraits ;
déductions ;
incertitudes.
```

Exemple de réponse :

```text
Faits :
- checkout utilise payments API.
- payments API est déployée sur cluster-3.
- cluster-3 sera en maintenance vendredi.

Déduction :
- checkout peut être affecté par la maintenance.

Limite :
- les sources ne précisent pas s’il existe un mécanisme de redondance ou de failover.
```

Cette manière de répondre est souvent préférable à une réponse trop catégorique.

---

## 5.18. Le problème du contexte trop long

Un autre risque est de fournir trop de contexte au LLM.

Même si les modèles modernes acceptent de grands contextes, un contexte trop long peut poser problème :

```text
coût plus élevé ;
latence plus forte ;
risque de dilution de l’information ;
risque de contradictions ;
risque que le modèle ignore certains passages ;
réponse moins ciblée.
```

Un bon système RAG ne cherche donc pas à donner le plus de documents possible.

Il cherche à donner :

```text
les documents suffisants et pertinents.
```

Il vaut souvent mieux fournir cinq passages bien choisis que cinquante passages approximatifs.

---

## 5.19. Ordonner les sources dans le prompt

L’ordre des sources peut influencer la réponse.

Nous pouvons ordonner les sources selon :

```text
score de pertinence ;
date ;
autorité de la source ;
structure logique ;
ordre chronologique ;
ordre du raisonnement.
```

Pour une question multi-hop, l’ordre logique peut être préférable.

Exemple :

```text
Source 1 : checkout utilise payments API.
Source 2 : payments API est déployée sur cluster-3.
Source 3 : cluster-3 sera en maintenance vendredi.
```

Cet ordre aide le modèle à suivre la chaîne de raisonnement.

Pour une question juridique, on peut préférer :

```text
texte légal ;
document interne ;
commentaire ;
cas particulier.
```

L’ordre n’est donc pas neutre.

---

## 5.20. Déduplication du contexte

Le retrieval peut récupérer plusieurs chunks très proches.

Exemple :

```text
Chunk 1 :
La maintenance du cluster-3 est prévue vendredi soir.

Chunk 2 :
Vendredi soir, le cluster-3 fera l’objet d’une maintenance.

Chunk 3 :
Maintenance cluster-3 vendredi soir.
```

Ces chunks apportent presque la même information.

Les injecter tous dans le prompt gaspille des tokens et peut donner une impression artificielle de confirmation.

Une étape de déduplication peut améliorer le contexte :

```text
garder le passage le plus complet ;
fusionner les passages proches ;
supprimer les redondances ;
préserver les sources distinctes si elles sont importantes.
```

---

## 5.21. Construction d’un contexte synthétique

Parfois, au lieu d’envoyer directement les chunks bruts au LLM final, nous pouvons faire une étape intermédiaire.

Pipeline :

```text
chunks récupérés
        ↓
synthèse contrôlée des faits
        ↓
réponse finale
```

Exemple de contexte synthétique :

```text
Faits extraits des sources :
1. checkout utilise payments API.
2. payments API est déployée sur cluster-3.
3. cluster-3 sera en maintenance vendredi soir.
```

Puis le LLM répond à partir de cette synthèse.

Cette approche peut rendre le prompt plus clair, mais elle ajoute un risque : la synthèse intermédiaire peut elle-même être erronée.

Il faut donc conserver les sources originales.

---

## 5.22. Réponse avec niveau de confiance

Certains systèmes demandent au LLM de produire un niveau de confiance.

Exemple :

```text
Confiance : élevée
```

ou :

```text
Confiance : faible
```

Mais il faut être prudent.

Un LLM peut être mal calibré sur sa propre confiance.

Il vaut mieux fonder le niveau de confiance sur des critères observables :

```text
les sources répondent directement ;
plusieurs sources cohérentes confirment ;
les sources sont récentes ;
les sources sont officielles ;
le retrieval a de bons scores ;
il n’y a pas de contradiction ;
l’information n’est pas déduite trop indirectement.
```

Exemple :

```text
Confiance : moyenne.
Raison : les sources permettent de déduire un impact possible, mais elles ne
précisent pas s’il existe un mécanisme de failover.
```

Le niveau de confiance doit donc être justifié.

---

## 5.23. Exemple complet de prompt robuste

Voici un exemple de prompt plus complet pour un RAG documentaire.

```text
Tu es un assistant documentaire technique.

Tu dois répondre à la question de l’utilisateur en t’appuyant uniquement sur
les sources fournies dans le contexte.

Règles :
1. N’utilise pas d’information extérieure au contexte.
2. Si le contexte ne permet pas de répondre, dis-le explicitement.
3. Si une information est déduite, indique qu’il s’agit d’une déduction.
4. Si les sources se contredisent, signale la contradiction.
5. Cite les sources utilisées pour chaque affirmation importante.
6. Ne cite pas une source qui ne soutient pas réellement l’affirmation.

Contexte :
[Source 1 — Runbook Checkout, section Dépendances]
Le service checkout utilise l’API payments pour déclencher les paiements.

[Source 2 — Inventaire infrastructure, section Payments]
L’API payments est déployée sur le cluster-3.

[Source 3 — Planning maintenance, section Cluster-3]
Le cluster-3 sera en maintenance vendredi soir à partir de 22 h.

Question :
Le service checkout sera-t-il affecté par la maintenance de vendredi ?

Format de réponse attendu :
- Réponse courte
- Justification
- Limites ou incertitudes
- Sources
```

Réponse attendue :

```text
Réponse courte :
Oui, le service checkout peut être affecté par la maintenance de vendredi.

Justification :
Le service checkout utilise l’API payments. L’API payments est déployée sur
le cluster-3. Or, le cluster-3 sera en maintenance vendredi soir à partir de
22 h. Nous pouvons donc déduire que checkout peut être impacté.

Limites ou incertitudes :
Les sources ne précisent pas s’il existe une redondance ou un mécanisme de
failover permettant d’éviter l’impact.

Sources :
- Source 1 — dépendance entre checkout et payments API.
- Source 2 — déploiement de payments API sur cluster-3.
- Source 3 — maintenance prévue du cluster-3.
```

Cette réponse est meilleure qu’un simple “oui”, car elle explicite le raisonnement et ses limites.

---

## 5.24. Exemple de mauvaise réponse

Avec le même contexte, une mauvaise réponse serait :

```text
Oui, checkout sera indisponible vendredi à partir de 22 h.
```

Pourquoi est-ce mauvais ?

Parce que les sources permettent de dire :

```text
checkout peut être affecté
```

mais pas forcément :

```text
checkout sera indisponible
```

Il manque des informations sur :

```text
la redondance ;
le failover ;
le mode de maintenance ;
l’impact exact ;
l’architecture de haute disponibilité.
```

Le modèle a transformé une possibilité en certitude.

C’est un exemple classique de surinterprétation.

---

## 5.25. Exemple de refus correct

Contexte :

```text
[Source 1]
Le service checkout utilise l’API payments.
```

Question :

```text
Le service checkout sera-t-il affecté par la maintenance de vendredi ?
```

Réponse correcte :

```text
Le contexte ne permet pas de répondre. Il indique seulement que le service
checkout utilise l’API payments, mais il ne fournit aucune information sur une
maintenance vendredi, ni sur l’infrastructure de l’API payments.
```

Cette réponse est moins spectaculaire, mais elle est plus fiable.

Un bon RAG doit préférer une réponse incomplète honnête à une réponse complète inventée.

---

## 5.26. Pseudo-code de génération RAG

Nous pouvons représenter la génération ainsi :

```python
def build_prompt(question, retrieved_chunks):
    context = ""

    for i, chunk in enumerate(retrieved_chunks, start=1):
        context += f"""
[Source {i}]
Titre : {chunk.metadata["title"]}
Section : {chunk.metadata.get("section", "Non précisée")}
Texte :
{chunk.text}
"""

    prompt = f"""
Tu es un assistant documentaire.

Règles :
- Réponds uniquement avec le contexte fourni.
- Si le contexte ne permet pas de répondre, dis-le.
- Cite les sources utilisées.
- Signale les contradictions éventuelles.

Contexte :
{context}

Question :
{question}

Réponse :
"""

    return prompt
```

Puis :

```python
def answer_question(question, user):
    chunks = retrieve_chunks(question, user)
    prompt = build_prompt(question, chunks)
    answer = llm.generate(prompt)

    return {
        "answer": answer,
        "sources": [chunk.metadata for chunk in chunks]
    }
```

Dans un système réel, nous ajouterions :

```text
validation du format ;
vérification des citations ;
filtrage des sources ;
logs ;
évaluation ;
gestion des erreurs ;
limites de tokens.
```

---

## 5.27. Validation de la réponse

Après génération, nous pouvons ajouter une étape de validation.

Questions de validation :

```text
La réponse utilise-t-elle réellement les sources ?
Chaque affirmation importante est-elle sourcée ?
Le modèle a-t-il inventé une date, un nom ou une valeur ?
La réponse signale-t-elle les incertitudes ?
Les sources citées soutiennent-elles vraiment les affirmations ?
Le format attendu est-il respecté ?
```

Cette validation peut être faite :

```text
par règles ;
par un autre appel LLM ;
par vérification automatique des citations ;
par échantillonnage humain ;
par tests sur un benchmark.
```

Dans des contextes sensibles, la validation humaine reste importante.

---

## 5.28. Les erreurs classiques de génération

### 5.28.1. Surinterprétation

Le modèle conclut plus que ce que les sources permettent.

```text
Source :
Le service peut être affecté.

Réponse incorrecte :
Le service sera indisponible.
```

### 5.28.2. Omission d’une source importante

Le modèle ignore un passage essentiel.

### 5.28.3. Mélange de sources

Le modèle combine deux sources qui ne parlent pas exactement du même périmètre.

### 5.28.4. Citation décorative

Le modèle cite une source, mais cette source ne soutient pas vraiment l’affirmation.

### 5.28.5. Réponse hors contexte

Le modèle utilise sa connaissance générale au lieu du contexte fourni.

### 5.28.6. Refus excessif

Le modèle dit qu’il ne peut pas répondre alors que les sources contiennent l’information.

### 5.28.7. Mauvaise gestion des contradictions

Le modèle choisit arbitrairement une source sans signaler le conflit.

---

## 5.29. Mini-TP : améliorer un prompt RAG

### Objectif

Nous voulons comparer plusieurs prompts et observer leur effet sur la réponse.

### Contexte

```text
[Source 1]
Le service checkout utilise l’API payments.

[Source 2]
L’API payments est déployée sur le cluster-3.

[Source 3]
Le cluster-3 sera en maintenance vendredi soir.
```

Question :

```text
Le service checkout sera-t-il affecté vendredi ?
```

### Prompt 1 : minimal

```text
Réponds à la question avec le contexte.

Contexte :
...

Question :
...
```

### Prompt 2 : plus strict

```text
Réponds uniquement à partir du contexte.
Si tu fais une déduction, indique-le.
Si le contexte ne suffit pas, dis-le.
Cite les sources.
```

### Prompt 3 : structuré

```text
Réponds en quatre parties :
1. Réponse courte
2. Faits extraits des sources
3. Déduction
4. Limites
```

### Travail demandé

Nous comparons les réponses obtenues.

Nous observons :

```text
le modèle cite-t-il les sources ?
fait-il une déduction trop forte ?
signale-t-il les limites ?
répond-il clairement ?
```

L’objectif est de comprendre que la formulation du prompt influence fortement la qualité de la génération.

---

## 5.30. Mini-TP : détecter les hallucinations

### Contexte

```text
[Source 1]
Le service checkout utilise l’API payments.

[Source 2]
L’API payments est déployée sur le cluster-3.
```

Question :

```text
À quelle heure aura lieu la maintenance de vendredi ?
```

### Réponse A

```text
La maintenance aura lieu vendredi à 22 h.
```

### Réponse B

```text
Le contexte ne permet pas de répondre. Il mentionne checkout, payments API et
cluster-3, mais ne donne aucune information sur une maintenance ni sur son horaire.
```

### Analyse attendue

La réponse A est une hallucination, car l’horaire n’est pas dans les sources.

La réponse B est correcte, même si elle ne répond pas directement à la question.

Nous devons apprendre à valoriser les réponses honnêtes.

---

## 5.31. Questions de compréhension

À la fin de ce chapitre, nous devons pouvoir répondre aux questions suivantes :

```text
Qu’est-ce qu’un prompt augmenté ?
Pourquoi le LLM ne doit-il pas être considéré comme la source ?
Pourquoi la consigne "réponds uniquement avec le contexte" ne suffit-elle pas toujours ?
Comment gérer une information absente ?
Comment gérer des sources contradictoires ?
Pourquoi les citations sont-elles importantes ?
Qu’est-ce qu’une citation décorative ?
Pourquoi un contexte trop long peut-il dégrader la réponse ?
Comment distinguer un fait extrait d’une déduction ?
Pourquoi la sécurité ne doit-elle pas être déléguée au prompt ?
Comment valider une réponse générée par un RAG ?
```

---

## 5.32. Synthèse du chapitre

Dans ce chapitre, nous avons étudié la génération dans un système RAG.

Nous avons vu qu’après le retrieval, il faut construire un prompt augmenté qui donne au LLM :

```text
les sources pertinentes ;
la question ;
les règles de réponse ;
le format attendu ;
les contraintes de citation.
```

Nous avons insisté sur une idée centrale :

```text
Le LLM formule la réponse, mais les documents doivent porter l’information.
```

Nous avons aussi vu qu’un bon système RAG doit savoir :

```text
répondre quand les sources suffisent ;
refuser ou nuancer quand elles ne suffisent pas ;
signaler les contradictions ;
séparer faits et déductions ;
citer précisément ;
éviter les hallucinations ;
ne pas transformer une possibilité en certitude.
```

Le message principal du chapitre est donc le suivant :

```text
La qualité d’un RAG ne dépend pas seulement des documents récupérés.
Elle dépend aussi de la manière dont nous présentons ces documents au LLM
et dont nous contrôlons la réponse générée.
```

Dans le prochain chapitre, nous étudierons l’**amélioration d’un système RAG**.

---

# 6. Améliorer le retrieval

## 6.1. Recherche lexicale, vectorielle et hybride

Nous comparerons :

```text
BM25
recherche vectorielle
recherche hybride
```

Nous expliquerons pourquoi la recherche lexicale reste forte sur :

- identifiants ;
    
- noms exacts ;
    
- codes ;
    
- références ;
    
- termes rares ;
    
- erreurs techniques.
    

Et pourquoi la recherche vectorielle est forte sur :

- reformulation ;
    
- synonymes ;
    
- proximité conceptuelle ;
    
- questions naturelles.
    

## 6.2. Reranking

Nous introduirons le reranking comme une seconde étape après le retrieval.

Pipeline :

```text
retrieval large : top 50
        ↓
reranker
        ↓
sélection finale : top 5
```

Nous verrons que le retriever cherche vite et large, tandis que le reranker trie plus finement.

## 6.3. Query rewriting

Nous verrons comment un LLM peut reformuler la question pour améliorer la recherche.

Exemple :

```text
Question utilisateur :
"Est-ce que ça casse vendredi ?"

Requête reformulée :
"impact de la maintenance prévue vendredi sur les services dépendants"
```

Nous discuterons aussi des risques : une mauvaise reformulation peut déformer l’intention.

---

# 7. Évaluation d’un système RAG

## 7.1. Pourquoi évaluer séparément retrieval et génération

Nous expliquerons qu’un système RAG peut échouer à deux endroits :

```text
1. Il ne récupère pas les bons documents.
2. Il récupère les bons documents mais le LLM répond mal.
```

Nous séparerons donc :

- évaluation du retrieval ;
    
- évaluation de la génération ;
    
- évaluation finale de la réponse.
    

## 7.2. Métriques de retrieval

Nous présenterons :

- precision@k ;
    
- recall@k ;
    
- MRR ;
    
- nDCG ;
    
- taux de documents sources retrouvés.
    

Nous relierons cela aux notions classiques vues en machine learning :

```text
vrai positif
faux positif
faux négatif
vrai négatif
précision
rappel
```

## 7.3. Métriques de génération

Nous étudierons :

- fidélité au contexte ;
    
- exactitude ;
    
- complétude ;
    
- utilité ;
    
- citation correcte ;
    
- absence d’hallucination.
    

Nous verrons qu’une réponse peut être bien écrite mais fausse, ou correcte mais insuffisamment sourcée.

## 7.4. Jeux de tests

Nous apprendrons à construire un petit benchmark interne :

```text
questions connues
documents sources attendus
réponses de référence
cas simples
cas ambigus
cas impossibles
cas multi-hop
```

---

# 8. Graph RAG

## 8.1. Pourquoi le RAG standard ne suffit pas toujours

Nous repartirons de l’exemple :

```text
checkout service → payments API → cluster-3 → maintenance vendredi
```

Nous montrerons que le RAG standard peut rater le fait intermédiaire.

## 8.2. Construire un graphe de connaissances

Nous introduirons les notions de :

```text
entité
relation
nœud
arête
triplet
```

Exemple :

```text
checkout service --uses--> payments API
payments API --runs_on--> cluster-3
cluster-3 --has_maintenance--> Friday
```

## 8.3. Extraction d’entités et de relations

Nous verrons comment un LLM peut extraire automatiquement des relations depuis des documents.

Nous discuterons aussi des difficultés :

- doublons d’entités ;
    
- synonymes ;
    
- erreurs d’extraction ;
    
- relations implicites ;
    
- relations contradictoires ;
    
- évolution temporelle des connaissances.
    

## 8.4. Retrieval par parcours de graphe

Nous montrerons que Graph RAG ne cherche pas seulement des chunks proches, mais aussi des chemins relationnels.

```text
entité de départ
        ↓
voisins
        ↓
chemin pertinent
        ↓
sous-graphe
        ↓
contexte injecté au LLM
```

## 8.5. Cas d’usage

Nous identifierons les cas où Graph RAG est pertinent :

- architecture logicielle ;
    
- dépendances entre services ;
    
- analyse de risques ;
    
- documentation d’entreprise ;
    
- historique de décisions ;
    
- corpus scientifiques ;
    
- analyse de réseaux d’acteurs ;
    
- systèmes juridiques ou administratifs complexes.
    

---

# 9. Agentic RAG

## 9.1. Différence entre pipeline fixe et agent

Nous comparerons :

```text
RAG standard :
question → retrieval → génération

Agentic RAG :
question → choix d’action → outil → observation → nouvelle action → réponse
```

Nous verrons que dans un Agentic RAG, le LLM ne reçoit pas seulement du contexte : il peut décider de chercher, interroger une API, reformuler, vérifier, ou appeler plusieurs outils.

## 9.2. Outils utilisables par un agent

Nous listerons les outils possibles :

- base vectorielle ;
    
- moteur BM25 ;
    
- graphe de connaissances ;
    
- base SQL ;
    
- API métier ;
    
- calendrier ;
    
- tickets ;
    
- logs ;
    
- moteur web ;
    
- interpréteur de code.
    

## 9.3. Exemple complet

Question :

```text
Vérifie si la panne Redis d’hier peut expliquer les erreurs Sentry, puis propose une correction.
```

Nous montrerons que l’agent pourrait :

```text
1. Chercher les incidents Redis.
2. Lire les logs applicatifs.
3. Consulter Sentry.
4. Lire les manifests Kubernetes.
5. Identifier les services dépendants.
6. Proposer une hypothèse.
7. Vérifier cette hypothèse.
8. Générer une réponse argumentée.
```

## 9.4. Risques de l’Agentic RAG

Nous insisterons sur les limites :

- coût plus élevé ;
    
- latence ;
    
- comportements moins déterministes ;
    
- appels outils inutiles ;
    
- boucles ;
    
- erreurs de planification ;
    
- difficulté d’évaluation ;
    
- sécurité des outils.
    

Nous verrons qu’en production, on encadre fortement les agents.

---

# 10. RAG multimodal et documents complexes

## 10.1. Le problème des PDF et documents riches

Nous étudierons les limites du RAG sur documents complexes :

- tableaux ;
    
- images ;
    
- schémas ;
    
- scans ;
    
- colonnes ;
    
- notes de bas de page ;
    
- documents administratifs ;
    
- rapports techniques ;
    
- présentations.
    

Nous verrons qu’un simple découpage texte peut perdre beaucoup d’information.

## 10.2. OCR, layout et structure documentaire

Nous introduirons :

- OCR ;
    
- extraction de tableaux ;
    
- conservation des titres ;
    
- hiérarchie documentaire ;
    
- métadonnées ;
    
- liens internes ;
    
- pages et positions.
    

## 10.3. Multimodal RAG

Nous expliquerons que le RAG multimodal peut indexer ou interpréter :

- texte ;
    
- images ;
    
- graphiques ;
    
- tableaux ;
    
- captures d’écran ;
    
- schémas techniques.
    

Nous verrons quand il faut utiliser des embeddings texte, image ou multimodaux.

---

# 11. Architecture de production

## 11.1. Composants principaux

Nous proposerons une architecture réaliste :

```text
connecteurs documentaires
        ↓
pipeline d’ingestion
        ↓
normalisation / nettoyage
        ↓
chunking
        ↓
embedding
        ↓
index vectoriel + index lexical + métadonnées
        ↓
retrieval / reranking
        ↓
LLM
        ↓
réponse sourcée
        ↓
logs / évaluation / feedback
```

## 11.2. Gestion de la fraîcheur des données

Nous verrons comment gérer :

- documents modifiés ;
    
- suppression de documents ;
    
- réindexation ;
    
- versioning ;
    
- droits d’accès ;
    
- données obsolètes.
    

## 11.3. Sécurité et droits d’accès

Nous aborderons un point critique :

> Un système RAG ne doit jamais donner accès à un document que l’utilisateur n’a pas le droit de lire.

Nous verrons donc :

- filtrage par permissions ;
    
- ACL ;
    
- séparation des index ;
    
- contrôle au moment du retrieval ;
    
- journalisation des accès.
    

## 11.4. Observabilité

Nous verrons ce qu’il faut logger :

- question utilisateur ;
    
- requête reformulée ;
    
- chunks récupérés ;
    
- scores ;
    
- sources utilisées ;
    
- réponse générée ;
    
- feedback utilisateur ;
    
- temps de réponse ;
    
- coût.
    

---

# 12. Travaux pratiques proposés

## TP 1 — Construire un mini moteur de recherche sémantique

Nous construirons un petit moteur qui :

```text
charge des documents Markdown
les découpe en chunks
calcule les embeddings
stocke les vecteurs
retrouve les passages proches d’une question
```

Objectif : comprendre le retrieval sans encore générer de réponse.

## TP 2 — Construire un RAG simple

Nous ajouterons un LLM au moteur précédent.

Le système devra :

```text
retrouver les chunks pertinents
les injecter dans un prompt
produire une réponse
citer les sources
```

## TP 3 — Comparer recherche lexicale et vectorielle

Nous construirons des exemples où :

- BM25 fonctionne mieux ;
    
- l’embedding fonctionne mieux ;
    
- l’hybride fonctionne mieux.
    

Nous analyserons les résultats.

## TP 4 — Évaluer le RAG

Nous créerons un jeu de questions/réponses.

Nous mesurerons :

- recall@k ;
    
- pertinence des sources ;
    
- fidélité de la réponse ;
    
- hallucinations.
    

## TP 5 — Mini Graph RAG

Nous construirons un petit graphe à partir de documents techniques.

Exemple :

```text
service A dépend de service B
service B utilise Redis
Redis est déployé sur cluster-1
```

Puis nous poserons des questions multi-hop.

## TP 6 — Mini Agentic RAG

Nous construirons un agent simple capable de choisir entre :

```text
chercher dans les documents
interroger une base structurée
répondre directement
demander une vérification
```

L’objectif ne sera pas de faire un agent “magique”, mais de comprendre les risques d’orchestration.

---

# 13. Projet final possible

Nous proposerons un projet final sous forme de mini-produit RAG.

Par exemple :

```text
Construire un assistant de documentation technique pour un projet logiciel.
```

Le système devra :

- ingérer une documentation ;
    
- découper proprement les documents ;
    
- indexer les chunks ;
    
- répondre aux questions ;
    
- citer les sources ;
    
- gérer les cas où la réponse n’est pas dans les documents ;
    
- proposer une évaluation minimale ;
    
- documenter les limites.
    

Version avancée :

- ajouter BM25 ;
    
- ajouter un reranker ;
    
- ajouter un graphe de dépendances ;
    
- ajouter une interface web ;
    
- ajouter des métriques d’observabilité.
    

---

# 14. Progression pédagogique proposée

## Séance 1 — Introduction aux limites des LLM et au principe du RAG

Nous posons le problème : pourquoi un LLM seul ne suffit pas pour des données métier.

## Séance 2 — Embeddings et similarité cosinus

Nous comprenons comment représenter le texte sous forme de vecteurs et comparer les textes entre eux.

## Séance 3 — Bases vectorielles et retrieval

Nous construisons le cœur du système de recherche.

## Séance 4 — Chunking, métadonnées et ingestion documentaire

Nous voyons pourquoi la qualité du RAG dépend énormément de la préparation des documents.

## Séance 5 — Génération augmentée et prompts sourcés

Nous apprenons à injecter le contexte dans le prompt et à produire des réponses fiables.

## Séance 6 — Évaluation du RAG

Nous séparons l’évaluation du retrieval et celle de la génération.

## Séance 7 — Optimisation : hybride, reranking, query rewriting

Nous améliorons la pertinence des résultats.

## Séance 8 — Graph RAG

Nous abordons les requêtes multi-hop et les graphes de connaissances.

## Séance 9 — Agentic RAG

Nous abordons les agents, les outils et l’orchestration dynamique.

## Séance 10 — RAG multimodal et documents complexes

Nous traitons les PDF, tableaux, images, schémas et documents semi-structurés.

## Séance 11 — Production, sécurité, observabilité

Nous passons d’un prototype à une architecture exploitable.

## Séance 12 — Présentation des projets

Nous évaluons les projets étudiants selon la qualité technique, l’évaluation et la clarté des limites.

---

# 15. Compétences attendues à la fin du cours

À la fin du cours, nous devrons être capables de :

- expliquer clairement ce qu’est un RAG ;
    
- distinguer RAG standard, Graph RAG et Agentic RAG ;
    
- comprendre le rôle des embeddings ;
    
- utiliser la similarité cosinus ;
    
- construire un pipeline d’ingestion ;
    
- choisir une stratégie de chunking ;
    
- interroger une base vectorielle ;
    
- concevoir un prompt augmenté ;
    
- évaluer la qualité d’un système RAG ;
    
- identifier les limites d’un système RAG ;
    
- proposer une architecture de production raisonnable ;
    
- justifier le choix entre RAG standard, Graph RAG et Agentic RAG selon le besoin.
    

---

# 16. Message pédagogique central

Le fil conducteur du cours pourrait être le suivant :

> Nous n’allons pas apprendre à “mettre un chatbot sur des documents”. Nous allons apprendre à construire un système de recherche, de sélection, de justification et de génération contrôlée autour d’un LLM.

C’est cette distinction qui est fondamentale.

Un RAG n’est pas seulement un LLM avec une base vectorielle. C’est une architecture complète où la qualité dépend autant de l’ingestion, du retrieval, de l’évaluation, des métadonnées, de la sécurité et des sources que du modèle génératif lui-même.