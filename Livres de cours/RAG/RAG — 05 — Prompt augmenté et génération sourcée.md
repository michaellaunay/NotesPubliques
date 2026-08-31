---
schema_version: 1
uid: 01M1BQ627802NSQXJ9Q4ZTX6QR
titre: "RAG — 05 — Prompt augmenté et génération sourcée"
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
resume: "Chapitre 5 sur 12 du livre « RAG » : Prompt augmenté et génération sourcée. Version longue du cours, découpée le 31 août 2026 à partir de l'état du 2026-08-29."
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

> [!info] Livre « RAG » — chapitre 5/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 04 — Construire un RAG standard|← 04 — Construire un RAG standard]] · [[RAG — 06 — Améliorer le retrieval|06 — Améliorer le retrieval →]]

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

---
> [!info] Livre « RAG » — chapitre 5/12
> [[RAG — Sommaire|Sommaire]] · [[RAG — 04 — Construire un RAG standard|← 04 — Construire un RAG standard]] · [[RAG — 06 — Améliorer le retrieval|06 — Améliorer le retrieval →]]
