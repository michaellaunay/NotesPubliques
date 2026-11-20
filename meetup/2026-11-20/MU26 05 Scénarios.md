---
titre: MU26 05 Scénarios
type: cours
statut: brouillon
para: ressource
domaines:
- enseignement
themes:
- genie-logiciel
- analyse
- 2tup
- scenarios
- intelligence-artificielle
- agents-ia
- claude-code
- meetup
resume: 'Écriture des scénarios de Kunereti avec Claude Code et Codex : acteurs et personnages, liste des scénarios concrets, gabarit, écriture en parallèle par des sous-agents, liste des hypothèses que l''écriture fait apparaître et contre-lecture croisée.'
niveau: intermediaire
auteurs:
- Michaël Launay
langue: fr
confidentialite: publique
rag: true
schema_version: 1
uid: 01M7WTQC6XGFQD10YE75TM47C7
date_creation: 2026-11-20
publication: []
date_modification: 2026-11-20
metadata_verifiees: false
---
# Remarque générale

Dans 2TUP, la phase d'analyse repose sur les cas d'utilisation. Un **scénario concret** raconte une histoire précise : telle personne, dans telle situation, fait ceci, et le système répond cela. Les scénarios se regroupent ensuite en **cas d'utilisation**. Les écrire revient à rédiger par avance le manuel d'utilisation.

En 2024, ChatGPT avait proposé sept scénarios pour un script de publication ([[MU Analyse par chat GPT]]). En 2026, l'analyse de Kunereti en compte 51, répartis en six cas d'utilisation ([[KR Analyse]]). Ce changement d'échelle n'est pas un progrès en soi : 51 scénarios écrits par un agent sont 51 textes à relire. La note montre comment les produire sans perdre la main, et surtout ce qu'ils servent à trouver.

> [!note] Prompts reconstitués
> Les prompts sont imaginés pour le meetup. La liste, le gabarit, les personnages et les hypothèses cités sont ceux de l'analyse réelle.

# Étape 1 : les acteurs et les personnages

**Moi**

```text
À partir de besoin.md et du glossaire, dresse la table des acteurs :
humains, agents d'IA et systèmes. Pour chacun : sa nature et ce qu'il
fait. Un même humain peut tenir plusieurs rôles : liste les rôles,
pas les personnes.

Propose ensuite une communauté fictive et six personnages nommés, que
nous garderons dans tous les scénarios, avec une proposition qu'ils
écriront ensemble du début à la fin.
```

La table réelle compte dix-sept acteurs, dont cinq agents d'IA (curateur, assistant d'amendement, synthétiseur, traducteur, veilleur) et trois systèmes (l'application, le vérificateur, la forge). La communauté est la Coopérative des Tilleuls ; Driss la fonde, Chloé la rejoint, Alice et Bob co-écrivent la proposition « Des vélos partagés », Fatou ouvre un groupe concurrent, Émile siège au comité de pilotage.

Les personnages ne sont pas un ornement. Un scénario qui dit « l'utilisateur vote » laisse tout dans le vague ; un scénario qui dit « Bob a voté à temps, mais son bulletin ne figure pas dans la liste figée à la clôture » oblige à dire ce que le système fait. Et garder les mêmes personnages d'un scénario à l'autre fait apparaître les incohérences : si Chloé est candidate au scénario 2, elle ne peut pas voter au scénario 5 sans avoir été admise au scénario 3.

# Étape 2 : la liste

**Moi**

```text
Donne-moi la liste des scénarios concrets au sens de 2TUP. Pour
chacun : un titre à l'infinitif, l'acteur principal, trois lignes.
Regroupe-les en cas d'utilisation.

Je veux :
- le parcours nominal complet d'une proposition, de la première
  contribution à l'avis final ;
- pour chaque cas d'utilisation, au moins un scénario où ça se passe
  mal : une fraude, une panne, une contestation ;
- un scénario par agent d'IA, où l'agent est l'acteur principal.

N'écris pas encore les scénarios. Signale les scénarios que tu
hésites à inclure, et pourquoi.
```

Demander la liste avant les textes permet de la discuter tant qu'elle ne coûte rien. La demande de scénarios « où ça se passe mal » est celle qui rapporte le plus : laissé à lui-même, un agent écrit des parcours où tout fonctionne.

Les six cas d'utilisation obtenus :

1. Faire vivre la communauté (fonder, rejoindre, admettre, élire, quitter).
2. Recueillir les contributions et tenir les idées.
3. Co-écrire une proposition en groupe de travail.
4. Discuter.
5. Soutenir et décider.
6. Garantir la confiance sans serveur.

Le sixième n'aurait pas existé sans la consigne sur les fraudes et les pannes. Il contient les scénarios les plus utiles pour la suite : [[KR Scénario 33 Refuser une fusion illégale]], [[KR Scénario 34 Contester un scrutin]], [[KR Scénario 36 Neutraliser une tentative d'injection]].

# Étape 3 : le gabarit, sur un seul scénario

On écrit d'abord un scénario, seul, et on le corrige jusqu'à ce qu'il serve de modèle.

**Moi**

```text
Écris le scénario 2, « Rejoindre une communauté », dans
docs/analyse/scenarios/. Plan imposé :
- Résumé (trois ou quatre phrases)
- Acteurs en jeu
- Prérequis
- Étapes du scénario, numérotées : une action par étape, et à chaque
  fois ce que la personne voit
- Diagramme de séquence Mermaid
- Une phrase qui rattache le scénario à son cas d'utilisation

N'emploie que des termes du glossaire. Chaque fois que tu dois
supposer une règle que ni le journal ni le glossaire ne donnent,
écris l'étape quand même et ajoute la supposition à hypotheses.md.
```

Le résultat est [[KR Scénario 02 Rejoindre une communauté]]. Extrait :

> 2. Elle scanne le QR code. L'application y lit l'adresse du dépôt et l'empreinte de la clé racine.
> 3. L'application clone le dépôt de communauté et vérifie que la notice est bien signée par la clé dont l'empreinte figure dans le QR code. Si ce n'est pas le cas, elle s'arrête et le dit.
> 4. Chloé lit la charte et l'accepte.
> 5. Elle choisit un pseudonyme. L'application génère ses deux paires de clés et les range dans le coffre du téléphone ; elle lui fait noter une phrase de secours.

Le diagramme de séquence n'est pas décoratif : pour le tracer, l'agent doit dire **qui** parle à **qui**, et un acteur oublié dans les étapes y apparaît aussitôt.

# Étape 4 : écrire les autres en parallèle

Les scénarios sont indépendants une fois la liste, le glossaire et le gabarit fixés. C'est le cas d'école de la délégation à des sous-agents.

**Moi**

```text
Le scénario 2 est le modèle. Écris les autres en déléguant à des
sous-agents, un par cas d'utilisation. Chacun reçoit : le glossaire,
besoin.md, la liste des scénarios, le scénario 2 comme gabarit, la
fiche des personnages, et la règle sur les hypothèses.

Quand ils ont fini, relis toi-même l'ensemble et rends-moi :
- les contradictions entre scénarios (même personnage, même objet,
  deux règles) ;
- la liste consolidée des hypothèses, numérotée, avec le scénario
  concerné ;
- les termes employés hors glossaire.
```

Ce que chaque sous-agent reçoit est tout ce qu'il sait : il n'a pas vu l'élicitation. C'est une bonne discipline, parce qu'elle oblige à vérifier que les fichiers suffisent. Si un sous-agent écrit une bêtise, c'est souvent qu'une règle n'était écrite nulle part.

# Ce que l'écriture a fait apparaître

C'est le résultat le plus important de l'étape, et il n'est pas dans les scénarios : il est dans `hypotheses.md`. L'analyse réelle en liste dix-huit. Quelques-unes :

| Scénario | Ce qu'il a fallu supposer pour écrire les étapes |
| --- | --- |
| 2. Rejoindre | Un candidat peut lire les contenus avant d'être admis. À revoir pour une communauté privée |
| 15. Rejoindre un groupe | Un nouveau participant ne vote qu'au scrutin suivant, puisque la liste des votants est figée à l'ouverture |
| 16. Mode wiki | Deux modifications simultanées de la même section sont réconciliées par celui qui envoie en second |
| 19. Voter | Un votant peut remplacer son bulletin jusqu'à la clôture |
| 20. Dépouiller | Si le quorum n'est pas atteint, le scrutin est sans effet et le texte d'origine est conservé |
| 20 et 21 | Des amendements adoptés séparément peuvent donner un texte incohérent : faut-il un vote sur l'ensemble ? |

Aucune de ces questions n'était apparue pendant l'élicitation. On ne les voit qu'en déroulant les étapes une à une. C'est pour cela qu'on écrit des scénarios, et c'est pour cela que la consigne « écris l'étape quand même et note la supposition » vaut mieux que « arrête-toi et demande » : l'agent avance, et l'humain tranche ensuite dix-huit questions d'un coup, en connaissance de cause.

# Étape 5 : la contre-lecture croisée

**Moi**, à Codex

```text
Lis docs/analyse/. Tu es un membre de mauvaise foi de la Coopérative
des Tilleuls. Pour chaque cas d'utilisation, cherche comment tricher,
bloquer ou nuire en respectant la lettre des scénarios. Rends une
liste : ce que tu fais, le scénario qui le permet, le scénario qui
devrait l'empêcher et ne le fait pas.
```

Donner un rôle hostile au contre-lecteur produit de meilleures trouvailles qu'un « vérifie la cohérence ». Parmi ce que ce type de lecture a fait ajouter à l'analyse réelle : la quarantaine après un départ (pour ne pas revenir aussitôt sous une identité vierge), la durée de validité d'une décision (pour ne pas revoter la même exclusion chaque matin), et le quota hebdomadaire de publications.

# Ce qu'il faut retenir

- Des personnages nommés, toujours les mêmes.
- La liste avant les textes ; un scénario modèle avant les cinquante autres.
- Exiger les scénarios d'échec : l'agent ne les propose pas seul.
- Le livrable caché est la liste des hypothèses.
- Les sous-agents ne savent que ce qui est écrit : c'est un test des fichiers.
- 51 scénarios ne se relisent pas à l'œil. Il faut des contrôles : [[MU26 06 Tableau de suivi glossaire-scénarios]] et [[MU26 07 QQOQCP et CRUD]].

Retour au support : [[MU26 00 Développer avec Claude Code et Codex]].
