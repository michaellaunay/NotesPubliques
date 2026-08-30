---
schema_version: 1
uid: 01M02EX5B00TK1CA0QPJ8VS0KM
titre: "Getting Things Done (GTD)"
aliases:
  - GTD
  - Getting Things Done
  - Méthode GTD
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
  - organisation-personnelle
themes:
  - productivite
  - gestion-du-temps
  - gtd
  - organisation
  - gestion-des-taches
  - revue-hebdomadaire
resume: "Cours pratique sur Getting Things Done (GTD) : capture, clarification, organisation, réflexion et engagement, listes de projets et prochaines actions, revue hebdomadaire, horizons de focus, planification naturelle et mise en œuvre dans un système papier ou numérique."
niveau: debutant
prerequis: []
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2023-06-27
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Getting Things Done (GTD)

> [!abstract] Objectif
> Comprendre et mettre en pratique la méthode **Getting Things Done (GTD)** de David Allen sans la réduire à une simple liste de tâches. À la fin de ce cours, on doit savoir capturer ce qui sollicite l'attention, clarifier chaque élément, organiser des rappels fiables, maintenir le système par des revues régulières et choisir quoi faire avec davantage de confiance.

> [!important] Idée centrale
> GTD cherche moins à « gérer le temps » qu'à **gérer ses engagements et son attention**. Le calendrier ne peut pas contenir plus d'heures ; en revanche, on peut améliorer la manière dont on collecte, décide, organise et revoit ce à quoi on s'est engagé.

Voir aussi : [[Obsidian]], [[Projet Encadré]], [[Les méthodes agiles]], [[Les méthodes agiles#5. Kanban|Kanban]], [[MindMap sous Obsidian]].

# 1. Ce qu'est GTD — et ce que GTD n'est pas

**Getting Things Done** est une méthode de productivité personnelle popularisée par David Allen. Elle repose sur un principe simple : l'esprit humain est performant pour **penser**, beaucoup moins pour conserver en permanence une longue liste d'engagements implicites.

Un système GTD vise donc à externaliser les rappels dans un système suffisamment fiable pour que l'on n'ait pas besoin de « garder les choses en tête ».

GTD n'est pas :

- un agenda rempli de tâches arbitrairement datées ;
- une méthode imposant une application particulière ;
- une matrice de priorités unique ;
- une méthode de gestion de projet complète pour une équipe ;
- une injonction à être constamment productif ;
- une simple todo list.

GTD est plutôt un **workflow de décision** :

```text
ce qui attire mon attention
          │
          ▼
       capturer
          │
          ▼
       clarifier
          │
          ▼
       organiser
          │
          ▼
       réfléchir
          │
          ▼
       s'engager / agir
```

La terminologie officielle actuelle résume les cinq étapes par :

1. **Capture** ;
2. **Clarify** ;
3. **Organize** ;
4. **Reflect** ;
5. **Engage**.

En français, on peut les traduire par :

1. **capturer** ;
2. **clarifier** ;
3. **organiser** ;
4. **réfléchir / revoir** ;
5. **s'engager dans l'action**.

> [!warning] Correction d'une confusion fréquente
> Dans GTD, **Engage** ne signifie pas simplement « s'engager à utiliser GTD ». C'est l'étape où l'on choisit et exécute l'action appropriée à partir d'un système à jour.

# 2. Le problème des « boucles ouvertes »

Une grande partie du stress organisationnel vient d'engagements mal définis :

```text
penser au dossier fiscal
réparer le vélo
organiser les vacances
répondre à Julie
changer de serveur
```

Ces formulations ont deux défauts :

- elles ne définissent pas toujours le résultat attendu ;
- elles ne définissent pas l'action physique suivante.

Par exemple :

```text
« refaire le site web »
```

n'est pas une action directement exécutable.

Une formulation plus exploitable pourrait être :

```text
Projet : publier la nouvelle version du site vitrine
Prochaine action : ouvrir le dépôt et lister les pages encore absentes
```

GTD cherche à transformer les engagements vagues en décisions explicites.

# 3. Les cinq étapes du workflow GTD

## 3.1 Capture — collecter ce qui a son attention

Capturer signifie déposer dans un système externe tout ce qui attire l'attention et mérite potentiellement une décision.

Exemples :

- une idée ;
- un courriel à traiter ;
- une promesse faite à quelqu'un ;
- une facture ;
- un document papier ;
- un rappel mental ;
- une tâche professionnelle ;
- un achat à envisager ;
- une question à rechercher ;
- un projet personnel.

L'objectif n'est pas de décider immédiatement de tout. L'objectif est de **ne pas perdre l'entrée**.

### 3.1.1 Une inbox n'est pas une liste de tâches

Une boîte d'entrée contient des éléments **non clarifiés**.

Par exemple :

```text
Inbox
├── facture EDF
├── idée conférence sécurité
├── appeler Paul
├── capture d'écran d'une erreur
└── renouvellement certificat
```

Après clarification, ces éléments peuvent devenir :

```text
Next Actions
Projects
Waiting For
Calendar
Someday/Maybe
Reference
Trash
```

Une inbox doit donc être vue comme une **zone de transit**, pas comme un lieu de stockage permanent.

### 3.1.2 Limiter le nombre de boîtes d'entrée

On peut avoir plusieurs points de capture :

- boîte mail ;
- carnet ;
- application de notes ;
- messagerie professionnelle ;
- bac physique ;
- dictaphone ;
- note mobile.

Mais plus leur nombre augmente, plus la revue devient difficile.

Une bonne règle pratique est :

> avoir autant de points de capture que nécessaire, mais aussi peu que possible.

### 3.1.3 Capturer vite

Une capture utile doit être :

- rapide ;
- sans friction ;
- disponible au moment où l'idée apparaît ;
- suffisamment claire pour être comprise plus tard.

Mauvais exemple :

```text
serveur
```

Meilleur exemple :

```text
Vérifier pourquoi le serveur d'intégration manque d'espace disque
```

On ne cherche pas encore à résoudre le problème, seulement à conserver une entrée compréhensible.

## 3.2 Clarify — déterminer ce que signifie chaque élément

La clarification est le cœur décisionnel de GTD.

Pour chaque entrée :

```text
Qu'est-ce que c'est ?
        │
        ▼
Est-ce actionnable ?
```

Il ne faut pas parcourir l'inbox en se demandant uniquement :

> « Est-ce important ? »

Il faut déterminer ce que l'élément **signifie concrètement**.

### 3.2.1 Si l'élément n'est pas actionnable

Un élément non actionnable peut devenir :

#### Déchet

Il ne présente plus de valeur.

```text
Trash
```

#### Référence

Il est utile de le conserver, mais aucune action n'est actuellement requise.

```text
Reference
```

Exemples :

- documentation technique ;
- facture déjà réglée ;
- contrat archivé ;
- documentation d'un appareil ;
- article utile.

#### À reconsidérer plus tard

L'élément pourrait devenir pertinent ultérieurement.

```text
Someday / Maybe
```

Exemples :

- apprendre le japonais ;
- construire un NAS ;
- visiter l'Islande ;
- essayer un nouveau framework.

Le point important est qu'un élément « Someday/Maybe » n'est **pas un engagement actif**.

### 3.2.2 Si l'élément est actionnable

Il faut déterminer :

1. le **résultat attendu** ;
2. la **prochaine action**.

#### Résultat attendu

Un projet décrit un résultat à obtenir.

Exemple :

```text
Serveur mail migré vers la nouvelle machine
```

#### Prochaine action

La prochaine action doit être la première action physique ou observable qui fait avancer le sujet.

Mauvais :

```text
travailler sur la migration
```

Meilleur :

```text
se connecter au serveur source et exporter la configuration Postfix
```

### 3.2.3 La règle des deux minutes

Si l'action prend environ deux minutes ou moins, GTD recommande généralement de la faire immédiatement lorsque cela est raisonnable.

Exemples :

```text
répondre « oui » à une invitation
classer un PDF
envoyer une adresse
confirmer une date
```

Pourquoi ?

Parce que le coût administratif de :

```text
noter → organiser → revoir → retrouver → exécuter
```

peut devenir supérieur au coût de l'action elle-même.

> [!note]
> « Deux minutes » n'est pas une loi physique. C'est un **seuil heuristique**. Dans une réunion, au volant ou au milieu d'une opération critique, on ne doit évidemment pas interrompre n'importe quoi pour appliquer mécaniquement la règle.

### 3.2.4 Déléguer

Si l'action doit être réalisée par quelqu'un d'autre :

```text
Waiting For
```

Il est utile d'enregistrer :

- la personne ;
- l'objet attendu ;
- la date de la demande ;
- éventuellement une date réelle d'échéance.

Exemple :

```text
[2026-08-31] Alice — retour sur le devis du serveur
```

### 3.2.5 Différer

Si une action doit être faite plus tard par soi-même, elle rejoint généralement :

- une liste de prochaines actions ;
- ou le calendrier si elle doit réellement avoir lieu à une date ou une heure déterminée.

# 4. Qu'est-ce qu'un projet dans GTD ?

Dans le vocabulaire GTD, un **projet** est un résultat souhaité nécessitant plus d'une action.

Cela peut donc être beaucoup plus petit qu'un « projet » au sens classique de la gestion de projet.

Exemples :

```text
Renouveler mon passeport
Installer la nouvelle imprimante
Organiser le repas d'anniversaire
Migrer le dépôt GitHub
Faire réparer la moto
```

Chacun demande plusieurs étapes.

Un projet GTD devrait idéalement être formulé comme un **résultat terminé**.

Moins bon :

```text
Assurance moto
```

Meilleur :

```text
Nouvelle assurance moto souscrite et ancienne résiliée
```

# 5. La liste Projects

La liste des projets est un **index des résultats ouverts**.

Elle n'a pas besoin de contenir toutes les étapes détaillées.

Exemple :

```markdown
# Projects

- [ ] Serveur de sauvegarde opérationnel
- [ ] Déclaration fiscale 2026 déposée
- [ ] Appartement de Toulouse aménagé
- [ ] Documentation API v2 publiée
```

À la revue, on vérifie notamment :

> Chaque projet actif possède-t-il au moins une prochaine action clairement définie ?

C'est un test simple mais puissant.

# 6. Organize — construire les bons rappels

Une fois l'élément clarifié, on range **le rappel**, pas nécessairement toute l'information, à l'endroit approprié.

Un système GTD typique contient :

```text
Calendar
Next Actions
Projects
Waiting For
Someday/Maybe
Reference
Checklists
```

## 6.1 Calendar — le « paysage dur »

Le calendrier doit surtout contenir ce qui est **réellement lié à une date ou une heure**.

Exemples :

- rendez-vous à 14 h ;
- train le 5 septembre ;
- date limite légale ;
- événement prévu ce jour-là ;
- appel convenu à une heure précise.

Éviter de transformer le calendrier en liste de souhaits :

```text
mardi : lire documentation
mercredi : nettoyer bureau
jeudi : trier photos
```

si ces actions pourraient en réalité être faites à d'autres moments.

Sinon, le calendrier devient une liste de promesses régulièrement non tenues.

## 6.2 Next Actions

Une liste de prochaines actions contient des actions **déjà clarifiées et exécutables**.

Exemples :

```text
- appeler le garage pour demander un devis
- lancer pytest sur la branche migration
- écrire à Claire pour confirmer la réservation
- mesurer l'espace disponible dans le rack
```

Une bonne prochaine action commence souvent par un verbe concret.

## 6.3 Organiser par contexte

Historiquement, GTD recommande de regrouper les actions par contexte, c'est-à-dire par environnement ou ressource nécessaire.

Exemples classiques :

```text
@ordinateur
@téléphone
@maison
@bureau
@courses
```

Aujourd'hui, les contextes peuvent être adaptés :

```text
@deep-work
@ssh
@atelier
@toulouse
@roubaix
@mobile
@hors-ligne
```

> [!tip]
> Les contextes doivent réduire les choix, pas créer une taxonomie parfaite. Si une personne travaille presque toujours sur ordinateur, `@ordinateur` peut devenir inutilement large.

## 6.4 Waiting For

La liste **Waiting For** contient les choses dont l'avancement dépend de quelqu'un ou quelque chose d'autre.

Exemple :

```markdown
# Waiting For

- [ ] 2026-08-28 — retour de Paul sur le contrat
- [ ] 2026-08-30 — remboursement fournisseur
- [ ] 2026-08-31 — validation DNS par l'hébergeur
```

Cette liste évite de devoir garder en mémoire :

> « Il faut que je pense à relancer… »

## 6.5 Someday / Maybe

Cette liste contient des idées intéressantes sans engagement actuel.

Exemples :

```text
Construire un clavier mécanique
Faire une formation Rust embarqué
Tester NixOS
Visiter Tallinn
Créer une bibliothèque personnelle numérisée
```

La liste doit être revue périodiquement car un « peut-être » peut devenir :

- un projet actif ;
- une référence ;
- ou un élément à supprimer.

## 6.6 Reference

Le matériel de référence répond à :

> « Je veux peut-être retrouver cette information, mais je n'ai pas d'action à faire dessus maintenant. »

Une bonne archive doit être facile à :

- ranger ;
- retrouver ;
- supprimer lorsqu'elle devient inutile.

Le classement peut être :

- alphabétique ;
- thématique ;
- par projet ;
- par client ;
- par type documentaire.

## 6.7 Checklists

Une checklist est utile pour les procédures répétitives :

```text
Avant un déplacement
- ordinateur
- chargeur
- casque
- billets
- portefeuille
```

ou :

```text
Déploiement production
- tests verts
- sauvegarde
- migration DB validée
- variables d'environnement
- monitoring actif
- rollback documenté
```

La checklist évite de réinventer une procédure à chaque fois.

# 7. Ne pas confondre tâche, prochaine action et projet

| Élément | Exemple | Sens |
|---|---|---|
| idée vague | `serveur mail` | pas encore clarifié |
| projet | `nouveau serveur mail en production` | résultat demandant plusieurs actions |
| prochaine action | `exporter main.cf du serveur actuel` | action immédiatement exécutable |
| calendrier | `bascule DNS lundi 22 h` | action liée à une date/heure réelle |
| waiting for | `retour hébergeur sur le PTR` | dépend d'un tiers |

# 8. Reflect — maintenir le système digne de confiance

Un système n'est utile que s'il reste à jour.

Une liste parfaite créée une fois puis abandonnée ne constitue pas un système GTD.

La réflexion se fait à plusieurs fréquences :

```text
plusieurs fois par jour → calendrier / actions utiles maintenant
quotidiennement          → inbox et engagements proches
hebdomadairement         → vue complète du système
périodiquement           → horizons plus élevés
```

# 9. La Weekly Review

La **revue hebdomadaire** est l'un des mécanismes essentiels de GTD.

Son but n'est pas seulement de « planifier la semaine ».

Elle sert à :

1. retrouver un système clair ;
2. remettre toutes les listes à jour ;
3. prendre du recul ;
4. faire apparaître les décisions oubliées ;
5. retrouver confiance dans le système.

La séquence officielle couramment présentée comporte onze étapes réparties en trois groupes.

## 9.1 Get Clear — retrouver de la clarté

### 1. Rassembler les papiers et éléments épars

Ramener les entrées dispersées vers les inbox.

### 2. Traiter les inbox jusqu'à zéro

« Zéro » signifie :

> zéro élément **non décidé** dans l'inbox.

Cela ne signifie pas zéro courriel dans toute la boîte mail historique.

### 3. Vider sa tête

Capturer les engagements encore présents mentalement :

```text
je dois penser à…
il faudrait…
je n'ai pas répondu à…
je devrais vérifier…
```

## 9.2 Get Current — remettre le système à jour

### 4. Revoir les listes d'actions

Supprimer ce qui est terminé ou devenu inutile.

### 5. Revoir le calendrier passé

Le passé peut révéler :

- une action non suivie ;
- des notes à classer ;
- une relance ;
- un nouveau projet.

### 6. Revoir le calendrier à venir

Anticiper :

- rendez-vous ;
- déplacements ;
- échéances ;
- préparation nécessaire.

### 7. Revoir Waiting For

Relancer lorsque nécessaire.

### 8. Revoir la liste des projets

Pour chaque projet :

```text
est-il toujours actif ?
le résultat est-il clair ?
y a-t-il une prochaine action ?
```

### 9. Revoir les checklists pertinentes

Certaines checklists ne sont utiles qu'à certaines périodes.

## 9.3 Get Creative — retrouver des possibilités

### 10. Revoir Someday / Maybe

Demander :

```text
est-ce le moment de l'activer ?
est-ce encore intéressant ?
faut-il le supprimer ?
```

### 11. Être créatif

Une fois le système nettoyé, de nouvelles idées peuvent apparaître.

C'est souvent une meilleure situation pour réfléchir que lorsque l'esprit est saturé de rappels non clarifiés.

# 10. Engage — choisir quoi faire

Une fois le système capturé, clarifié, organisé et revu, il faut agir.

La question devient :

> « Parmi les actions réellement disponibles, laquelle est appropriée maintenant ? »

GTD utilise plusieurs critères.

## 10.1 Contexte

Qu'est-il possible de faire ici ?

Exemple : sans ordinateur, inutile de consulter une liste de tâches nécessitant un shell SSH.

## 10.2 Temps disponible

Une fenêtre de dix minutes et une plage de trois heures ne permettent pas le même travail.

## 10.3 Énergie disponible

Certaines tâches demandent :

- concentration ;
- créativité ;
- interaction sociale ;
- travail physique ;
- faible énergie.

Il peut être utile de distinguer :

```text
@deep-work
@low-energy
```

## 10.4 Priorité

Une fois filtrées les actions faisables compte tenu du contexte, du temps et de l'énergie, il reste à choisir celle qui a le plus de valeur.

GTD ne remplace donc pas le jugement sur les priorités ; il cherche à faire en sorte que ce jugement s'effectue sur un inventaire plus fiable.

# 11. La nature triple du travail

GTD distingue trois formes de travail.

## 11.1 Faire du travail prédéfini

Exécuter une action déjà clarifiée :

```text
Next Actions → choisir → faire
```

## 11.2 Faire le travail qui apparaît

Certaines choses nouvelles exigent une réaction immédiate :

- incident production ;
- appel urgent ;
- personne qui demande de l'aide ;
- panne matérielle.

Ce n'est pas nécessairement mauvais.

Le problème apparaît lorsque **tout** est traité comme une interruption urgente.

## 11.3 Définir son travail

Traiter les nouvelles entrées :

```text
Capture → Clarify → Organize
```

Une personne qui ne réserve jamais de temps à cette activité finit avec des inbox saturées et des engagements ambigus.

# 12. Les Horizons of Focus

GTD ne se limite pas aux actions immédiates.

La méthode décrit plusieurs niveaux de perspective.

Une représentation pratique est :

```text
Purpose & Principles
        ↓
Vision
        ↓
Goals & Objectives
        ↓
Areas of Focus & Accountability
        ↓
Projects
        ↓
Next Actions / Calendar
```

## 12.1 Purpose and Principles

Questions :

```text
Pourquoi est-ce que je fais cela ?
Quelles valeurs ou règles comptent pour moi ?
```

## 12.2 Vision

À quoi ressemble la situation souhaitée à plus long terme ?

## 12.3 Goals and Objectives

Quels résultats plus importants veut-on obtenir à moyen terme ?

## 12.4 Areas of Focus and Accountability

Ce sont des domaines à maintenir, pas nécessairement des résultats à terminer.

Exemples :

```text
santé
finances
famille
enseignement
infrastructure
sécurité
formation
```

Une zone de responsabilité n'est jamais « terminée » au même sens qu'un projet.

## 12.5 Projects

Résultats à terminer nécessitant plusieurs actions.

## 12.6 Ground — Calendar / Actions

Le niveau opérationnel : ce qui peut être fait maintenant.

# 13. Pourquoi les horizons comptent

Supposons que la liste d'actions propose :

```text
A. optimiser un script de sauvegarde
B. préparer un dossier de candidature
C. comparer deux fournisseurs
```

Les trois sont faisables.

Pour choisir, il faut parfois monter en altitude :

```text
objectif : obtenir une certification cette année
zone de responsabilité : sécurité de l'infrastructure
vision : réduire la dépendance à un fournisseur
```

Les priorités deviennent plus intelligibles lorsque les actions sont reliées aux niveaux supérieurs.

# 14. Le Natural Planning Model

Pour un projet plus complexe, GTD propose un modèle de planification dit « naturel ».

Il suit cinq mouvements :

1. définir le **purpose and principles** ;
2. visualiser le **résultat souhaité** ;
3. faire un **brainstorming** ;
4. **organiser** les idées ;
5. déterminer les **next actions**.

## 14.1 Purpose and principles

Question :

```text
Pourquoi faisons-nous ce projet ?
Quelles contraintes ou valeurs doivent être respectées ?
```

Exemple :

```text
Projet : migrer le serveur mail
Purpose : réduire le risque lié au matériel vieillissant
Principles : interruption < 15 min, aucun secret dans Git, rollback possible
```

## 14.2 Outcome visioning

Décrire à quoi ressemble le succès.

```text
Le nouveau serveur reçoit et envoie les messages,
SPF/DKIM/DMARC passent,
les files sont vides,
le monitoring fonctionne,
l'ancien serveur peut être arrêté.
```

## 14.3 Brainstorming

Lister les idées sans chercher à les ordonner immédiatement :

```text
DNS
Postfix
DKIM
backups
certificats
monitoring
rollback
tests Gmail
PTR
firewall
```

## 14.4 Organizing

Regrouper et ordonner :

```text
Préparation
├── sauvegarde
├── inventaire DNS
└── provisionnement

Configuration
├── Postfix
├── DKIM
└── TLS

Validation
├── tests entrant
├── tests sortant
└── monitoring
```

## 14.5 Next actions

Transformer le plan en mouvements concrets :

```text
Créer la VM Ubuntu 26.04
Exporter postconf -n
Lister les enregistrements DNS actuels
```

# 15. GTD et gestion de projet

GTD répond surtout à :

```text
Quel est le résultat ?
Quelle est la prochaine action ?
Que dois-je revoir ?
```

Un outil de gestion de projet peut en plus gérer :

- dépendances ;
- budgets ;
- ressources ;
- chemin critique ;
- jalons ;
- risques ;
- reporting ;
- rôles d'équipe.

Ils ne sont donc pas concurrents.

On peut avoir :

```text
GTD personnel
     │
     ├── prochaine action : mettre à jour le ticket #184
     │
     ▼
Gestion de projet d'équipe
     │
     └── ticket #184 → sprint / milestone / owner
```

# 16. GTD et Kanban

Kanban visualise généralement le **flux de travail** :

```text
To Do → Doing → Done
```

GTD clarifie plutôt la nature des engagements :

```text
Inbox
Projects
Next Actions
Waiting For
Someday/Maybe
Calendar
```

Une combinaison possible :

```text
GTD → décider ce qu'est l'élément
Kanban → visualiser l'avancement de certains projets/actions
```

# 17. GTD et time blocking

Le time blocking réserve des plages de temps :

```text
09:00–11:00 développement
14:00–15:00 administratif
```

GTD peut l'alimenter, mais il faut éviter de transformer toutes les prochaines actions en faux rendez-vous calendaires.

Une bonne combinaison :

```text
Calendrier : contraintes dures + blocs intentionnels importants
Next Actions : inventaire flexible
```

# 18. GTD et matrice d'Eisenhower

La matrice urgent / important aide à analyser la priorité.

GTD résout un problème différent :

- capturer ;
- clarifier ;
- organiser ;
- maintenir l'inventaire.

Une action peut être correctement clarifiée en GTD puis priorisée avec d'autres méthodes.

# 19. Construire un système GTD minimal

Il n'est pas nécessaire de commencer avec vingt listes.

Un système minimal peut contenir :

```text
Inbox
Calendar
Projects
Next Actions
Waiting For
Someday/Maybe
Reference
```

## 19.1 Version papier

On peut utiliser :

- un carnet ;
- un agenda ;
- des feuilles ;
- un classeur ;
- des dossiers de référence.

La méthode ne dépend pas du numérique.

## 19.2 Version numérique

On peut utiliser :

- une application de tâches ;
- Obsidian ;
- un gestionnaire de projet ;
- un agenda ;
- un gestionnaire de courriels ;
- une combinaison de plusieurs outils.

Le bon outil est celui que l'on **consulte réellement**.

# 20. Implémentation simple dans Obsidian

Une organisation possible :

```text
GTD/
├── Inbox.md
├── Projects.md
├── Next Actions.md
├── Waiting For.md
├── Someday Maybe.md
├── Areas.md
├── Reviews/
└── Reference/
```

## 20.1 Inbox.md

```markdown
# Inbox

- [ ] vérifier contrat assurance
- [ ] idée : sauvegarde hors site
- [ ] demander à Paul les photos
```

## 20.2 Projects.md

```markdown
# Projects

- [ ] Nouveau serveur mail opérationnel
- [ ] Déclaration fiscale déposée
- [ ] Bibliothèque Obsidian réorganisée
```

## 20.3 Next Actions.md

```markdown
# Next Actions

## @ordinateur
- [ ] exporter la configuration Postfix

## @téléphone
- [ ] appeler le garage pour le devis

## @maison
- [ ] mesurer l'emplacement de l'étagère
```

## 20.4 Waiting For.md

```markdown
# Waiting For

- [ ] 2026-08-31 — Alice — retour devis
- [ ] 2026-08-31 — hébergeur — validation PTR
```

## 20.5 Someday Maybe.md

```markdown
# Someday / Maybe

- apprendre Rust embarqué
- construire un homelab ARM
- visiter Helsinki
```

# 21. Notes par projet dans Obsidian

Chaque projet peut avoir sa propre note :

```markdown
---
type: projet
statut: actif
resultat: "Serveur mail migré et ancien serveur arrêté"
---

# Migration serveur mail

## Résultat attendu

Le nouveau serveur traite le courrier de production.

## Prochaine action

- [ ] exporter `postconf -n`

## Support

- [[Postfix]]
- [[Sécurité avancée sous Linux]]

## Notes

...
```

La liste `Projects.md` peut alors être un index de liens vers les projets.

# 22. Métadonnées utiles sans sur-concevoir

Exemple :

```yaml
type: action
projet: migration-mail
contexte: ssh
energie: haute
```

Il faut toutefois éviter de passer plus de temps à enrichir les métadonnées qu'à faire les actions.

> [!warning]
> Un système GTD doit réduire la friction cognitive. Une ontologie trop complexe peut recréer le problème qu'elle était censée résoudre.

# 23. Une inbox dans Obsidian avec capture rapide

On peut conserver une note très simple :

```markdown
- [ ] {{date}} {{time}} — texte de la capture
```

ou une note quotidienne :

```markdown
## Inbox

- [ ] idée à clarifier
```

L'important n'est pas le plugin utilisé, mais le fait que chaque entrée soit ensuite **clarifiée**.

# 24. Les tâches datées : attention aux fausses échéances

Exemple :

```text
Lire documentation — due: 2026-09-02
```

Si le 2 septembre n'a aucune signification réelle, cette date sert seulement à faire remonter artificiellement la tâche.

Après quelques semaines :

```text
20 tâches en retard
```

Le système perd sa crédibilité.

Réserver les vraies échéances aux contraintes réelles.

# 25. Tickler / rappels futurs

Certaines informations doivent réapparaître à une date donnée sans être des actions à faire aujourd'hui.

Exemple :

```text
15 novembre → revoir les offres d'assurance avant renouvellement
```

Ce rôle peut être joué par :

- un calendrier ;
- une tâche différée ;
- un système de rappel ;
- historiquement, un « tickler file » physique.

# 26. Traiter les courriels avec GTD

La boîte mail est une inbox, pas nécessairement un gestionnaire de tâches.

Pour chaque courriel :

```text
Qu'est-ce que c'est ?
Est-ce actionnable ?
```

Puis :

```text
non actionnable
├── supprimer
├── archiver en référence
└── reconsidérer plus tard

actionnable
├── faire maintenant si très court
├── déléguer → Waiting For
└── différer → Next Actions / Calendar
```

## 26.1 Ne pas utiliser « non lu » comme unique rappel

Le statut non lu mélange :

- nouveaux messages ;
- tâches ;
- référence ;
- rappels ;
- messages importants.

Il vaut mieux transformer explicitement le message en action ou rappel lorsque nécessaire.

## 26.2 Inbox Zero n'est pas une religion

Le but n'est pas une esthétique de boîte vide.

L'objectif est de réduire les **éléments non décidés**.

Une archive peut contenir des milliers de messages sans poser de problème si elle n'est pas confondue avec l'inbox active.

# 27. Messageries instantanées

Slack, Teams, Discord, Matrix, SMS et autres messageries génèrent également des engagements.

Si un message implique :

```text
« peux-tu m'envoyer le rapport demain ? »
```

il ne devrait pas dépendre uniquement du fait que le message reste visible dans l'historique.

Créer un rappel dans le système de confiance.

# 28. GTD en équipe

GTD est surtout connu comme méthode personnelle, mais ses mécanismes sont transposables à une équipe :

```text
Capture    → quels sujets ont l'attention de l'équipe ?
Clarify    → quel résultat et quelle prochaine action ?
Organize   → où suit-on le projet ? qui est responsable ?
Reflect    → quelles revues régulières ?
Engage     → que fait l'équipe maintenant ?
```

Une équipe doit toutefois ajouter explicitement :

- propriétaires ;
- responsabilités ;
- dépendances ;
- décisions ;
- règles de communication.

# 29. Waiting For en équipe

Une liste d'attente personnelle peut devenir :

```text
Sujet | Responsable | Attendu | Date demande | Échéance réelle
```

Exemple :

```text
DNS | Hébergeur | PTR configuré | 31/08 | 02/09
```

Cela réduit les relances improvisées.

# 30. GTD et IA générative

Une IA peut aider à certains endroits du workflow, mais elle ne doit pas devenir une nouvelle inbox opaque.

Usages possibles :

- reformuler une capture vague ;
- proposer une prochaine action ;
- résumer une réunion ;
- extraire des engagements d'un texte ;
- regrouper des notes ;
- préparer une checklist ;
- suggérer les projets implicites d'un ensemble d'actions.

Exemple :

```text
Capture : "problème serveur client"
```

Une IA peut poser ou aider à poser :

```text
Quel résultat souhaites-tu ?
Quelle est la première action observable ?
```

## 30.1 L'IA ne décide pas de l'engagement à votre place

Un modèle peut proposer :

```text
« envoyer le contrat aujourd'hui »
```

mais il ne sait pas nécessairement :

- si l'engagement existe réellement ;
- si le contrat est validé ;
- si l'échéance est réelle ;
- si l'action est prioritaire.

La clarification finale reste une responsabilité humaine.

## 30.2 Confidentialité

Envoyer à un service externe :

- courriels ;
- contrats ;
- notes de santé ;
- informations clients ;
- secrets professionnels ;

peut être inapproprié.

Il faut appliquer les règles de confidentialité et de protection des données pertinentes.

# 31. Automatiser sans perdre le contrôle

Exemple utile :

```text
courriel marqué « action »
        ↓
création d'une entrée dans Inbox
```

Mais éviter :

```text
chaque message
        ↓
création automatique d'une tâche
```

sinon le système produit du bruit plutôt que de la clarté.

# 32. Le rôle d'un système de confiance

Un système devient « digne de confiance » lorsque l'on sait :

1. que les entrées importantes y arrivent ;
2. qu'elles seront clarifiées ;
3. que les rappels se trouvent au bon endroit ;
4. que le système est revu assez souvent.

Si l'on continue à penser :

```text
« je dois quand même m'en souvenir moi-même »
```

alors une partie de la confiance manque.

# 33. Symptômes d'un système GTD qui ne fonctionne plus

- inbox jamais vidée ;
- dizaines de listes presque identiques ;
- projets sans prochaine action ;
- actions vagues ;
- échéances artificielles ;
- centaines de tâches en retard ;
- `Someday/Maybe` jamais revu ;
- `Waiting For` oublié ;
- calendrier utilisé comme fourre-tout ;
- revue hebdomadaire évitée parce qu'elle est devenue trop longue ;
- système si complexe qu'on ne le maintient plus.

# 34. Anti-pattern : tout capturer, rien clarifier

Résultat :

```text
Inbox = 742 éléments
```

Le système devient simplement une nouvelle mémoire externe anxiogène.

La capture n'a de valeur que si elle est suivie de clarification.

# 35. Anti-pattern : multiplier les outils

Exemple :

```text
Todoist → tâches perso
Obsidian → projets
mail → rappels
Slack → actions équipe
Google Keep → idées
papier → appels
```

Cela peut fonctionner, mais seulement si l'on sait précisément :

- ce qui va où ;
- quelles inbox doivent être revues ;
- où se trouve la source de vérité.

Sinon, consolider.

# 36. Anti-pattern : utiliser les projets comme actions

```text
- [ ] refaire la cuisine
- [ ] lancer nouveau site
- [ ] organiser mariage
```

Ces éléments sont trop grands pour être choisis comme action immédiate.

Il faut leur associer une prochaine action.

# 37. Anti-pattern : sur-découper

À l'inverse :

```text
- ouvrir navigateur
- ouvrir onglet
- rechercher site banque
- cliquer connexion
```

est rarement utile pour une action ordinaire.

Le niveau de granularité doit rendre l'action **évidente**, sans transformer chaque geste en micro-tâche.

# 38. Anti-pattern : utiliser la priorité pour éviter la clarification

Attribuer :

```text
P1
P2
P3
```

à :

```text
serveur
impôts
site
```

ne résout pas l'ambiguïté.

Il faut d'abord déterminer :

```text
quel résultat ?
quelle prochaine action ?
```

# 39. Anti-pattern : faire la revue comme une séance de production

Pendant une revue, une action découverte peut parfois être faite immédiatement si elle est minuscule.

Mais si chaque action devient une heure de travail, la revue n'aboutit jamais.

Le but principal est de :

```text
mettre le système à jour
```

pas d'exécuter tous les projets.

# 40. Reprendre GTD après abandon

Ne pas essayer de réparer six mois de retard en une seule séance parfaite.

Séquence pragmatique :

```text
1. choisir une inbox principale
2. capturer ce qui est critique
3. clarifier les engagements immédiats
4. reconstruire Projects
5. reconstruire Next Actions
6. reconstruire Waiting For
7. revoir le calendrier
8. remettre une revue régulière
```

Puis nettoyer l'ancien système progressivement.

# 41. Mise en place en une heure

Une première installation légère peut être :

## 0–10 min — créer les listes

```text
Inbox
Projects
Next Actions
Waiting For
Someday/Maybe
```

## 10–25 min — mind sweep

Noter tout ce qui attire l'attention.

## 25–45 min — clarifier

Traiter les éléments les plus importants.

## 45–55 min — relier projets et actions

Chaque projet actif doit avoir une prochaine action.

## 55–60 min — programmer la revue

Choisir un créneau réaliste.

# 42. Exemple complet

Capture :

```text
contrôle technique moto
```

Clarification :

```text
Est-ce actionnable ? oui
Résultat : contrôle technique effectué et documents archivés
Plus d'une action ? oui → projet
Prochaine action : vérifier la date limite sur la carte grise / documents applicables
```

Organisation :

```text
Projects
- Contrôle technique moto effectué

Next Actions @ordinateur
- Vérifier les obligations et la date applicable
```

Après recherche :

```text
Next Actions @téléphone
- Appeler le centre X pour réserver
```

Après appel :

```text
Calendar
- 12/09 14:30 — contrôle technique
```

Après rendez-vous :

```text
Project → Done
Reference → facture / rapport
```

# 43. Exemple technique complet

Capture :

```text
certificat TLS expire bientôt
```

Clarification :

```text
Résultat : certificat renouvelé automatiquement et alerte validée
Projet : oui
Prochaine action : exécuter certbot certificates sur le serveur
```

Organisation :

```text
Projects
- Renouvellement TLS fiabilisé

Next Actions @ssh
- `sudo certbot certificates`
```

Découverte :

```text
renouvellement automatique cassé
```

Nouvelle action :

```text
- inspecter `systemctl status certbot.timer`
```

Puis :

```text
Waiting For
- fournisseur DNS — autorisation API
```

Enfin :

```text
Calendar
- date réelle d'expiration du certificat
```

La séparation entre projet, action et attente rend la situation beaucoup plus lisible.

# 44. Revue quotidienne légère

Une routine possible :

```text
matin
├── calendrier
├── contraintes du jour
└── prochaines actions utiles

fin de journée
├── capturer les engagements apparus
├── traiter les éléments urgents de l'inbox
└── vérifier le lendemain
```

Ce n'est pas un remplacement de la revue hebdomadaire.

# 45. Revue mensuelle ou trimestrielle

Les horizons supérieurs ne nécessitent pas forcément une revue hebdomadaire détaillée.

Une revue plus large peut demander :

```text
Mes zones de responsabilité sont-elles équilibrées ?
Quels objectifs ont changé ?
Quels projets n'ont plus de raison d'être ?
Qu'est-ce qui monopolise mon attention ?
Quels engagements dois-je renégocier ?
```

# 46. Choisir la fréquence de revue

« Hebdomadaire » est un rythme recommandé, mais le principe fondamental est :

> revoir assez souvent pour que le système reste fiable.

Une personne avec peu d'engagements peut avoir des besoins différents d'un responsable de plusieurs projets actifs.

L'important est la **cohérence**.

# 47. GTD et procrastination

GTD ne supprime pas toute procrastination.

Mais il peut réduire certaines causes :

## Ambiguïté

```text
« faire le dossier »
```

→ définir la prochaine action.

## Projet trop grand

```text
« écrire mémoire »
```

→ déterminer le premier mouvement concret.

## Surcharge de choix

100 tâches visibles en même temps

→ filtrer par contexte, temps et énergie.

## Manque de confiance

système obsolète

→ refaire une revue.

Il existe toutefois d'autres causes : fatigue, anxiété, manque d'intérêt, conflit de valeurs, difficulté émotionnelle, etc. GTD n'est pas une solution universelle à ces problèmes.

# 48. GTD et repos

Un système de productivité sain doit également contenir ou protéger :

- sommeil ;
- récupération ;
- loisirs ;
- temps non structuré ;
- relations ;
- santé.

Le but n'est pas de remplir chaque minute d'une « prochaine action ».

# 49. GTD et charge de travail excessive

Un système bien organisé peut révéler une réalité inconfortable :

```text
le volume d'engagements dépasse la capacité disponible
```

La bonne réponse n'est pas toujours d'optimiser davantage.

Il peut être nécessaire de :

- supprimer ;
- déléguer ;
- reporter ;
- renégocier ;
- refuser certains engagements.

GTD aide aussi à rendre cette surcharge visible.

# 50. Mesurer la qualité d'un système GTD

De mauvais indicateurs seraient :

```text
nombre maximal de tâches terminées
nombre de captures
nombre de tags
```

Des questions plus utiles :

- Puis-je retrouver mes engagements ?
- Mes projets ont-ils des prochaines actions ?
- Est-ce que je consulte mon calendrier avec confiance ?
- Est-ce que Waiting For empêche les oublis ?
- Les inbox restent-elles traitables ?
- Est-ce que je peux cesser d'y penser lorsque je ne travaille pas dessus ?

# 51. Checklist de clarification

Pour chaque entrée :

```text
[ ] Qu'est-ce que c'est ?
[ ] Est-ce actionnable ?
```

Si non :

```text
[ ] supprimer ?
[ ] référence ?
[ ] someday/maybe ?
```

Si oui :

```text
[ ] quel résultat ?
[ ] est-ce un projet ?
[ ] quelle prochaine action ?
[ ] moins de deux minutes ?
[ ] déléguer ?
[ ] date/heure réellement nécessaire ?
[ ] sinon, dans quelle liste d'actions ?
```

# 52. Checklist d'un projet sain

```text
[ ] résultat formulé clairement
[ ] toujours pertinent
[ ] propriétaire connu
[ ] prochaine action définie
[ ] support / notes retrouvables
[ ] échéance réelle enregistrée si nécessaire
[ ] dépendances suivies
[ ] revu régulièrement
```

# 53. Checklist d'une Weekly Review

```text
GET CLEAR
[ ] rassembler les éléments épars
[ ] clarifier les inbox
[ ] vider la tête

GET CURRENT
[ ] revoir Next Actions
[ ] revoir calendrier passé
[ ] revoir calendrier futur
[ ] revoir Waiting For
[ ] revoir Projects
[ ] revoir les checklists utiles

GET CREATIVE
[ ] revoir Someday/Maybe
[ ] laisser apparaître de nouvelles idées
```

# 54. Questions de diagnostic

Si un système semble lourd, demander :

```text
Ai-je trop de points de capture ?
Mes inbox sont-elles réellement clarifiées ?
Mes projets sont-ils formulés comme des résultats ?
Mes actions sont-elles réellement exécutables ?
Est-ce que j'abuse des dates d'échéance ?
Ai-je trop de contextes ?
Est-ce que Waiting For est à jour ?
Ma revue est-elle trop longue ? pourquoi ?
Ai-je des engagements que je devrais abandonner ?
```

# 55. Exemple de structure complète

```text
GTD/
├── 00 Inbox.md
├── 10 Next Actions.md
├── 20 Projects.md
├── 30 Waiting For.md
├── 40 Someday Maybe.md
├── 50 Areas.md
├── 60 Goals.md
├── 70 Checklists/
├── 80 Reviews/
└── 90 Reference/
```

L'ordre numérique est facultatif. Il peut simplement faciliter la navigation.

# 56. Exemple de frontmatter pour une action

```yaml
---
type: action
statut: a-faire
projet: migration-mail
contexte: ssh
energie: moyenne
---
```

Mais une simple ligne :

```markdown
- [ ] Exporter `postconf -n` #ssh
```

peut être largement suffisante.

# 57. Exemple de frontmatter pour un projet

```yaml
---
type: projet
statut: actif
resultat: "Nouveau serveur mail en production"
area: infrastructure
---
```

Puis :

```markdown
## Prochaine action

- [ ] Exporter la configuration actuelle
```

# 58. Liens entre GTD et PARA

PARA classe l'information en :

```text
Projects
Areas
Resources
Archives
```

GTD gère davantage le **workflow d'engagements et d'actions**.

Les deux peuvent se compléter :

```text
GTD
├── Projects
├── Next Actions
├── Waiting For
└── Someday/Maybe

PARA
├── Projects
├── Areas
├── Resources
└── Archives
```

Par exemple, un projet GTD peut avoir son dossier documentaire dans `Projects/`, alors que sa prochaine action reste dans `Next Actions`.

# 59. Ne pas confondre Projects et Areas

Projet :

```text
Publier la documentation v2
```

→ peut être terminé.

Area :

```text
Documentation produit
```

→ doit être maintenue dans le temps.

Cette distinction évite des projets éternels comme :

```text
« Santé »
« Finances »
« Infrastructure »
```

# 60. Méthode de migration depuis une todo list classique

Supposons :

```text
- impôts
- voiture
- Pierre
- serveur
- vacances
```

Étape 1 : clarifier.

```text
impôts → déclaration 2026 déposée
voiture → entretien annuel effectué
Pierre → répondre à sa proposition
serveur → nouvelle VM de sauvegarde en production
vacances → séjour septembre réservé
```

Étape 2 : déterminer les projets.

```text
Projects
- déclaration 2026 déposée
- entretien voiture effectué
- VM de sauvegarde en production
- séjour septembre réservé
```

Étape 3 : déterminer les prochaines actions.

```text
Next Actions
- télécharger le relevé fiscal
- appeler le garage
- répondre à Pierre
- vérifier le volume de sauvegardes actuel
- demander les dates disponibles aux participants
```

La différence de clarté est considérable.

# 61. Méthode de traitement d'une inbox importante

Lorsque l'inbox contient des centaines d'éléments :

1. ne pas commencer par réorganiser l'application ;
2. prendre un élément ;
3. décider ce qu'il signifie ;
4. décider où va son rappel ;
5. passer au suivant.

Éviter le « cherry picking » permanent :

```text
je traite uniquement les éléments faciles
```

car les éléments ambigus restent indéfiniment.

# 62. Archive vs Someday/Maybe

Référence :

```text
« Je veux conserver cette information. »
```

Someday/Maybe :

```text
« Je pourrais vouloir agir là-dessus un jour. »
```

Exemple :

```text
PDF sur l'impression 3D → Reference
Construire une imprimante 3D → Someday/Maybe
```

# 63. Incubation et échéances futures

Certains éléments ne doivent pas être vus aujourd'hui mais doivent revenir plus tard.

Exemple :

```text
Revoir l'abonnement en décembre
```

On peut utiliser :

- calendrier ;
- rappel différé ;
- tickler ;
- tâche avec date de réapparition.

Cela évite de surcharger les listes d'actions présentes.

# 64. Projet bloqué

Un projet peut être bloqué parce que :

- on attend quelqu'un ;
- une ressource manque ;
- une décision n'a pas été prise ;
- la prochaine action est mal définie.

Exemple :

```text
Projet : installer fibre secondaire
```

Si rien n'est faisable avant le retour de l'opérateur :

```text
Waiting For
- opérateur — proposition de date
```

La revue de projet doit rendre ce blocage visible.

# 65. Projet en pause

Si un projet n'est plus actif maintenant :

```text
Projects → Someday/Maybe
```

Il vaut mieux une liste courte de projets **réellement actifs** qu'une liste de 80 projets dont 60 ne sont pas engagés.

# 66. Définir « done »

Un projet devient plus clair lorsque l'état terminé est observable.

Vague :

```text
sécurité serveur
```

Clair :

```text
serveur audité, correctifs critiques appliqués et rapport archivé
```

La formulation du résultat réduit les ambiguïtés.

# 67. Next action verb

Les verbes utiles sont souvent :

```text
appeler
écrire
chercher
mesurer
lire
comparer
tester
ouvrir
réserver
commander
compiler
exécuter
vérifier
```

Les formulations moins actionnables :

```text
réfléchir à
avancer sur
s'occuper de
travailler sur
voir pour
```

Elles peuvent être valides si « réfléchir » est réellement l'action, mais elles cachent souvent une décision non faite.

# 68. Quand « réfléchir » est une vraie action

Exemple valide :

```text
Réfléchir 30 min aux risques de migration et produire une liste
```

Ici, on sait :

- ce que l'on fait ;
- pendant combien de temps environ ;
- quel résultat en sort.

# 69. Time / energy estimates

On peut enrichir les actions :

```text
@ordinateur 15m low-energy
@deep-work 90m high-energy
```

Mais seulement si ces données aident réellement à choisir.

Un système trop annoté demande trop de maintenance.

# 70. Une règle de conception : chaque donnée doit servir une décision

Avant d'ajouter un champ :

```text
priorité
énergie
durée
contexte
projet
zone
tag
responsable
```

poser :

> « Quelle décision ce champ m'aidera-t-il réellement à prendre ? »

S'il n'a aucun usage concret, ne pas le maintenir.

# 71. Revue de système

Tous les quelques mois, revoir non seulement les tâches, mais **l'outil lui-même** :

```text
Quelles listes ne sont jamais consultées ?
Quels tags ne servent plus ?
Quelles automatisations génèrent du bruit ?
Quels doublons existent ?
Quels points de capture peuvent être supprimés ?
```

# 72. GTD et recherche plein texte

Les outils modernes rendent parfois inutile une classification extrêmement fine.

Si une référence peut être retrouvée rapidement par :

```text
recherche plein texte
liens
métadonnées simples
```

il n'est pas nécessaire de créer des dizaines de dossiers.

# 73. GTD et RAG personnel

Dans une base documentaire augmentée par recherche sémantique, GTD garde une fonction différente :

```text
RAG → retrouver et synthétiser de l'information
GTD → suivre des engagements et prochaines actions
```

Ne pas transformer le corpus RAG en gestionnaire implicite des tâches.

Une réponse du système :

```text
« Le contrat prévoit un renouvellement en novembre »
```

n'est pas encore un rappel GTD.

Il faut décider :

```text
Créer un rappel en octobre pour revoir le contrat
```

# 74. GTD et calendrier automatique

Une IA ou une application peut proposer de planifier automatiquement les tâches.

Avantages :

- estimation de capacité ;
- visibilité temporelle ;
- aide au focus.

Risques :

- calendriers irréalistes ;
- cascade de replanifications ;
- confusion entre intention et contrainte réelle.

Conserver une distinction entre :

```text
engagement horaire réel
```

et :

```text
préférence de travail
```

# 75. Exemple de revue de projet

Projet :

```text
Documentation API v2 publiée
```

Revue :

```text
Résultat toujours voulu ? oui
Échéance réelle ? 15 septembre
Dernière action terminée ? schéma OpenAPI mis à jour
Prochaine action ? relire la section authentification
Waiting For ? retour sécurité sur OAuth
Support à revoir ? ticket #318, notes atelier
```

Le projet redevient immédiatement actionnable.

# 76. Exemple de Weekly Review en 30 minutes

Ce n'est qu'un exemple :

```text
00–05  collecter les éléments épars
05–10  traiter les inbox urgentes
10–15  revoir calendrier passé/futur
15–20  revoir Waiting For
20–25  revoir Projects et Next Actions
25–28  revoir Someday/Maybe
28–30  capturer idées / décisions
```

Si le backlog est énorme, la première revue prendra probablement plus longtemps.

# 77. Si la revue dure trois heures chaque semaine

Causes possibles :

- inbox jamais traitées entre les revues ;
- trop de projets actifs ;
- système trop fragmenté ;
- références mélangées aux actions ;
- la revue devient une séance de réalisation ;
- trop de métadonnées à maintenir.

La solution n'est pas forcément « être plus discipliné ». Il faut parfois simplifier le système.

# 78. GTD pour les études

Exemples :

```text
Projects
- Rapport de stage rendu
- Examen réseau préparé
- Dossier alternance finalisé

Next Actions
- télécharger le sujet du rapport
- relire chapitre TCP
- demander l'attestation RH
```

Area :

```text
Formation universitaire
```

Référence :

```text
cours PDF
bibliographie
annales
```

# 79. GTD pour l'administration système

Projects :

```text
Postfix migré
Sauvegarde restaurable validée
Ubuntu 26.04 déployé
```

Next Actions :

```text
@ssh
- vérifier `postconf -n`
- exécuter test de restauration
- lire les erreurs `journalctl -p err`
```

Waiting For :

```text
hébergeur — PTR
client — créneau de maintenance
```

Calendar :

```text
fenêtre de maintenance réelle
expiration certificat
```

# 80. GTD pour un développeur

Projects :

```text
Version 2.4 publiée
Bug #431 corrigé
Migration PostgreSQL terminée
```

Actions :

```text
ouvrir le test qui reproduit #431
comparer les plans EXPLAIN avant/après index
préparer changelog 2.4
```

Le backlog GitHub reste une source de travail d'équipe ; la liste GTD personnelle ne doit pas nécessairement en recopier chaque ticket.

# 81. GTD pour une activité indépendante

Areas :

```text
clients
comptabilité
commercial
infrastructure
formation
```

Projects :

```text
Facture X encaissée
Déclaration TVA déposée
Proposition client Y envoyée
```

Waiting For devient particulièrement important pour :

- paiements ;
- bons de commande ;
- validations ;
- remboursements ;
- réponses administratives.

# 82. GTD et obligations réglementaires

Une échéance légale doit être enregistrée comme contrainte réelle.

Mais l'action nécessaire doit apparaître **avant** la date limite.

Exemple :

```text
Calendar
31/10 — échéance déclaration

Next Actions
- télécharger les pièces nécessaires
```

Attendre le jour de l'échéance pour voir l'action serait trop tard.

# 83. Projet avec deadline

Pour un projet :

```text
Rapport rendu le 15 septembre
```

on peut avoir :

```text
Calendar
15/09 — deadline réelle

Projects
Rapport final remis

Next Actions
Relire la section 2
```

Le calendrier contient la contrainte ; la liste d'actions contient ce que l'on peut faire maintenant.

# 84. Incidents et urgences

Lors d'un incident production :

```text
Engage → le travail qui apparaît devient prioritaire
```

Après l'incident :

```text
Capture → actions de suivi
Clarify → projets / next actions
Organize → tickets / rappels
Reflect → revue postmortem
```

GTD n'empêche pas de réagir à une urgence réelle.

# 85. GTD et interruption permanente

Si toute la journée est consacrée à « work as it shows up » :

```text
messages → réaction
mail → réaction
appel → réaction
notification → réaction
```

les projets importants mais non urgents progressent peu.

Le système doit permettre de revenir au travail prédéfini lorsque l'interruption est terminée.

# 86. Réduire les notifications

Une notification transforme potentiellement une entrée en interruption avant même sa clarification.

Réduire les notifications non essentielles peut protéger l'étape **Engage** contre une capture forcée permanente.

# 87. Un système GTD n'a pas besoin d'être joli

Il doit être :

- fiable ;
- rapide ;
- compréhensible ;
- suffisamment complet ;
- facile à revoir.

Une interface magnifique avec des engagements obsolètes reste un mauvais système.

# 88. Progression d'apprentissage recommandée

## Niveau 1 — contrôle

Maîtriser :

```text
Capture
Clarify
Organize
```

## Niveau 2 — fiabilité

Ajouter :

```text
Weekly Review
Waiting For
Someday/Maybe
```

## Niveau 3 — perspective

Relier :

```text
Projects
Areas
Goals
Vision
Purpose
```

## Niveau 4 — adaptation

Simplifier les outils et automatiser les points répétitifs sans perdre la clarté.

# 89. Exercices

## Exercice 1 — mind sweep

Pendant dix minutes, écrire tout ce qui attire l'attention sans chercher à organiser.

Puis compter :

- actions ;
- projets ;
- références ;
- someday/maybe.

## Exercice 2 — transformer les formulations vagues

Transformer :

```text
banque
serveur
vacances
rapport
moto
```

en résultats + prochaines actions.

## Exercice 3 — nettoyer le calendrier

Pour chaque tâche datée :

```text
La date est-elle réellement contrainte ?
```

Si non, déplacer vers une liste appropriée.

## Exercice 4 — construire Waiting For

Lister toutes les choses actuellement attendues d'autres personnes ou organisations.

## Exercice 5 — Weekly Review

Faire une revue complète en utilisant la checklist du chapitre 53.

# 90. Questions de compréhension

1. Pourquoi une inbox n'est-elle pas une liste de tâches ?
2. Quelle différence entre projet et prochaine action ?
3. Quand utiliser le calendrier ?
4. À quoi sert Waiting For ?
5. Pourquoi une Weekly Review est-elle nécessaire ?
6. Quelle différence entre Projects et Areas ?
7. Pourquoi une échéance artificielle peut-elle réduire la confiance dans le système ?
8. Quelle différence entre GTD et Kanban ?
9. Comment une IA peut-elle aider sans décider des engagements à votre place ?
10. Pourquoi Someday/Maybe n'est-il pas un backlog de projets actifs ?

# 91. Réponses synthétiques

1. L'inbox contient des éléments non clarifiés ; une liste d'actions contient des décisions déjà prises.
2. Un projet demande plusieurs actions ; la prochaine action est le mouvement concret immédiatement exécutable.
3. Pour les événements et contraintes réellement liés à une date ou heure, pas comme liste de souhaits.
4. À suivre ce qui dépend d'un tiers et éviter les oublis de relance.
5. Parce qu'un système se périme ; la revue le remet à jour et restaure la confiance.
6. Un projet se termine ; une area doit être maintenue.
7. Parce qu'une accumulation de faux retards transforme le système en source de bruit.
8. GTD clarifie les engagements personnels ; Kanban visualise surtout le flux de travail.
9. En aidant à reformuler, résumer et proposer ; la validation de l'engagement reste humaine.
10. Parce qu'un élément Someday/Maybe n'est pas actuellement engagé.

# 92. Résumé opérationnel

```text
CAPTURE
→ sortir de sa tête ce qui attire l'attention

CLARIFY
→ qu'est-ce que c'est ?
→ actionnable ?
→ résultat ?
→ prochaine action ?

ORGANIZE
→ Calendar
→ Next Actions
→ Projects
→ Waiting For
→ Someday/Maybe
→ Reference

REFLECT
→ revoir assez souvent
→ Weekly Review

ENGAGE
→ choisir selon contexte, temps, énergie et priorité
```

# 93. Les trois questions les plus utiles

Lorsqu'un sujet est flou :

```text
1. Quel résultat est-ce que je veux obtenir ?
2. Quelle est la prochaine action concrète ?
3. Où dois-je revoir ce rappel au bon moment ?
```

Ces trois questions capturent une grande partie de la valeur pratique de GTD.

# 94. Sources et ressources

Sources primaires et ressources de référence :

- David Allen Company — **What is GTD?** : <https://gettingthingsdone.com/what-is-gtd/>
- David Allen Company — ressources sur les cinq étapes : <https://gettingthingsdone.com/five-Steps/>
- David Allen Company — **Choosing what to do**, avec les Horizons of Focus et la Three-fold Nature of Work : <https://gettingthingsdone.com/2023/01/choosing-what-to-do/>
- David Allen Company — ressources et guides : <https://gettingthingsdone.com/insights/>
- David Allen Company — **The GTD Weekly Review**, séquence des onze étapes : <https://gettingthingsdone.com/2009/05/the-gtd-weekly-review/>
- David Allen, *Getting Things Done: The Art of Stress-Free Productivity*.

> [!note] À propos des marques
> **Getting Things Done**, **GTD**, **Weekly Review**, **Horizons of Focus** et certains noms de modèles associés sont des marques ou désignations utilisées par la David Allen Company. Ce cours en présente les concepts à des fins pédagogiques ; il ne remplace pas les ouvrages, formations ou guides officiels.

# 95. Conclusion

GTD fonctionne lorsque le système permet de répondre rapidement à quatre questions :

```text
Qu'est-ce qui a mon attention ?
Qu'est-ce que cela signifie ?
Où se trouve le bon rappel ?
Quelle action est appropriée maintenant ?
```

Le bénéfice principal n'est pas de posséder une todo list plus sophistiquée. C'est de transformer des engagements flous en un inventaire **décidé, organisé, revu et actionnable**.
