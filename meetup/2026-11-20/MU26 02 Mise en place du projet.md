---
titre: MU26 02 Mise en place du projet
type: cours
statut: brouillon
para: ressource
domaines:
  - enseignement
themes:
  - genie-logiciel
  - organisation-de-projet
  - intelligence-artificielle
  - agents-ia
  - claude-code
  - git
  - meetup
resume: "Mise en place du projet Kunereti à partir d'un dépôt vide : arborescence des livrables d'analyse, fichier de consignes AGENTS.md partagé par Claude Code et Codex, permissions, et premiers prompts de cadrage."
niveau: intermediaire
auteurs:
  - Michaël Launay
langue: fr
confidentialite: publique
rag: true
schema_version: 1
uid: 01M7WTJM22FT8TTX6G5BT06ENP
date_creation: 2026-11-20
publication:
  - notes-publiques
date_modification: 2026-11-20
metadata_verifiees: true
---
# Objectif

Avant la première question d'analyse, on installe ce qui fera tenir le projet dans la durée : un dépôt, une arborescence où chaque livrable a sa place, et un fichier de consignes que les deux agents relisent à chaque session. C'est un quart d'heure qui évite de répéter les mêmes règles à chaque prompt.

> [!note] Prompts reconstitués
> Les prompts de cette note ont été imaginés pour le meetup. Les règles qu'ils installent sont celles qui s'appliquent réellement dans ce coffre (voir `AGENTS.md` à sa racine).

# Le dépôt vide

```bash
mkdir kunereti && cd kunereti
git init
mkdir -p docs
$EDITOR docs/brief.md      # le brief de MU26 01
git add . && git commit -m "docs(brief): déposer le brief de départ"
```

# Premier prompt : proposer sans rien créer

On lance Claude Code en mode plan : il lit et propose, il n'écrit pas.

**Moi**

```text
Lis docs/brief.md. Nous allons mener la phase d'analyse de ce projet
selon la méthode 2TUP : élicitation, glossaire métier, scénarios
regroupés en cas d'utilisation, puis QQOQCP et matrice CRUD pour
trouver les objets et les données, et enfin des croquis d'écrans.

Propose-moi :
1. une arborescence de dossiers où chaque livrable a une place ;
2. un fichier AGENTS.md qui fixe les règles de travail.

Ne crée rien. Ne propose aucune solution technique pour le logiciel
lui-même : nous n'en sommes pas là.
```

**Claude Code** (résumé) propose l'arborescence suivante, acceptée après deux retouches (le dossier `decisions/` et le caractère généré de `suivi.md` et `crud.md`).

```text
kunereti/
├── AGENTS.md              consignes communes aux deux agents
├── CLAUDE.md              une ligne : @AGENTS.md
├── docs/
│   ├── brief.md           le désir de départ, jamais modifié
│   ├── elicitation/
│   │   ├── journal.md     questions, réponses, dates
│   │   ├── besoin.md      le besoin reformulé
│   │   └── hypotheses.md  ce qui a été supposé, à trancher
│   ├── analyse/
│   │   ├── glossaire.md
│   │   ├── analyse.md     acteurs, cas d'utilisation, liste des scénarios
│   │   ├── scenarios/     un fichier par scénario
│   │   ├── suivi.md       GÉNÉRÉ : termes du glossaire × scénarios
│   │   ├── qqoqcp/        une fiche par scénario
│   │   ├── crud.md        GÉNÉRÉ : objets × scénarios
│   │   └── croquis/       un croquis Excalidraw par écran
│   └── decisions/         une fiche courte par décision prise
└── scripts/               ce qui génère suivi.md et crud.md
```

Dans ce coffre, les mêmes livrables portent d'autres noms : [[KR Glossaire métier]], [[KR Analyse]], les fiches `KR Scénario NN …` et les croquis de `kunereti/kunereti_images/`.

# Le fichier de consignes

Un seul fichier fait foi. Codex lit `AGENTS.md` ; Claude Code lit `CLAUDE.md`, qui se contente de l'importer. Les deux agents travaillent ainsi sous les mêmes règles, et on ne maintient qu'un texte.

```md
# Consignes pour les agents

Kunereti est en phase d'analyse. Il n'y a pas de code et il ne doit pas
y en avoir avant la fin de l'analyse, hormis dans scripts/.

## Où sont les choses
- Le besoin : docs/elicitation/besoin.md. Le brief ne fait plus foi.
- Les mots : docs/analyse/glossaire.md.
- Les scénarios : docs/analyse/scenarios/, gabarit dans analyse.md.

## Règles
- N'emploie dans un scénario que des termes du glossaire. Si un terme
  manque, propose-le au glossaire ; ne l'invente pas en passant.
- Quand tu dois supposer quelque chose, écris-le dans
  docs/elicitation/hypotheses.md avec le fichier concerné.
- Ne modifie jamais brief.md ni journal.md (on y ajoute, on n'y réécrit pas).
- suivi.md et crud.md sont générés : modifie le script, pas le fichier.
- Tout fichier que tu écris porte `verifie: false` dans son en-tête.
  Seul un humain le passe à true.
- Écris en français, phrases courtes, sans jargon technique dans les
  scénarios : ils doivent être lisibles par un membre du collectif.

## Vérifier
- `make suivi` et `make crud` doivent passer sans erreur.

## Commits
- Messages en français : `type(scope): description à l'infinitif`.
- Ne pousse jamais sur main. Laisse un diff à relire.
```

Ce fichier est court exprès. Chaque ligne répond à une erreur que l'agent ferait sinon : inventer un mot, corriger le brief pour qu'il colle à la solution, retoucher à la main un tableau généré, se déclarer lui-même relu.

# Permissions

| Agent | Mode au départ | Écrit dans | Commandes |
| --- | --- | --- | --- |
| Claude Code, auteur | Plan, puis écriture avec confirmation | `docs/`, `scripts/` | `git status`, `git diff`, `make suivi`, `make crud` |
| Codex, contre-lecteur | Lecture seule | Rien : il rend un rapport | Aucune |

Le contre-lecteur en lecture seule est un choix de méthode autant que de sécurité : s'il pouvait corriger, on ne saurait plus ce qu'il a trouvé.

# Second prompt : faire relire les consignes

Les consignes sont le premier livrable ; on les fait donc relire comme les autres.

**Moi**, à Codex

```text
Lis AGENTS.md et docs/brief.md. Tu es le contre-lecteur de ce projet.
Dis-moi :
- quelles consignes sont ambiguës ou contradictoires ;
- quelle erreur prévisible d'un agent aucune consigne n'empêche ;
- ce que tu retirerais parce que c'est inutile.
Ne réécris pas le fichier. Rends une liste.
```

**Codex** (résumé) relève que rien ne dit qui tranche une hypothèse, ni ce que fait l'agent quand deux réponses du journal se contredisent. On ajoute deux lignes : les hypothèses sont tranchées par le porteur du projet, et la réponse la plus récente du journal l'emporte, l'agent signalant la contradiction.

# Ce qu'on a à la fin

Un dépôt de trois fichiers, deux commits, et des règles que l'on n'aura plus à redire. La suite : [[MU26 03 Élicitation]].

Retour au support : [[MU26 00 Développer avec Claude Code et Codex]].
