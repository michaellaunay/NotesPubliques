---
schema_version: 1
uid: "01M02JG1VBZZ1DFNE4H1T5ST2A"
titre: "Copilot"
aliases:
  - "GitHub Copilot"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - intelligence-artificielle
  - assistance-au-developpement
  - vscode
  - copilot
resume: "Fiche de situation sur GitHub Copilot en 2026 : composants (complétion, chat, mode agent, agent cloud, CLI, revue de code), offres et facturation en crédits IA, activation dans VS Code, modèles et clés personnelles, personnalisation par fichiers d'instructions, bonnes pratiques et alternatives ; renvoie au cours Visual Studio Code pour le détail."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-01-23
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# GitHub Copilot

> [!abstract] Objectif
> Savoir ce qu'est devenu GitHub Copilot en 2026 — une famille d'outils (complétion, chat, agents, CLI, revue de code) facturés en crédits IA et intégrés à VS Code et à GitHub — comment l'activer, avec quels modèles, et dans quel cadre l'utiliser ; le détail du travail avec les agents (instructions, prompt files, agents personnalisés, MCP, workflows responsables) est dans les chapitres 19 à 24 du cours [[Visual studio code]].

> [!info] État de la fiche
> Vérifiée le 29 août 2026 sur les pages officielles de GitHub. Cette fiche remplace la procédure de 2023 « Ajouter GitHub Copilot à Visual Studio Code », devenue sans objet : l'extension n'a plus à être installée, elle est intégrée à VS Code, et l'usage s'est déplacé de la complétion vers les agents.

Voir aussi : [[Visual studio code]], [[LLM en local]], [[Travailler avec Claude]], [[git]].

# 1. Ce que recouvre « Copilot » en 2026

GitHub Copilot, lancé en 2021 comme complétion de code, désigne aujourd'hui un ensemble d'outils partageant un abonnement et un choix de modèles :

| Composant | Où | Ce qu'il fait |
|---|---|---|
| **Complétion et suggestions d'édition suivante** | éditeur | propose la suite du code ou la modification suivante, acceptée avec `Tab` |
| **Chat** | éditeur, github.com, mobile | questions sur le code, explication, génération, avec le contexte du dépôt |
| **Mode agent** (*agent mode*) | éditeur | planifie et exécute une tâche multi-fichiers : édite, lance des commandes et des tests avec votre autorisation, itère |
| **Agent cloud** (*coding agent*) | github.com | on lui assigne une *issue* ou une tâche ; il travaille dans un environnement GitHub Actions et ouvre une *pull request* à relire |
| **Copilot CLI** | terminal | agent en ligne de commande (`copilot`), planification (`/plan`), exécution encadrée |
| **Revue de code** | pull requests | commentaires automatiques sur un diff |
| **Spaces**, **Spark**, **Copilot Apps** | github.com | contextes partagés, génération d'applications, intégrations |

Ce qui n'a pas changé : Copilot **suggère**, il ne prouve rien. Un code généré doit être lu, compris, testé et relu sous l'angle sécurité avant d'être intégré (cours [[Visual studio code]], § 19.1 et chapitre 24).

# 2. Offres et facturation

Depuis le **1er juin 2026**, GitHub facture l'usage en **crédits IA** (1 crédit = 0,01 $) calculés sur les jetons consommés au tarif du modèle choisi ; l'ancien décompte en « requêtes premium » ne subsiste que pour certains abonnements annuels antérieurs.

| Offre | Prix (par utilisateur et par mois) | Contenu |
|---|---|---|
| **Free** | 0 $ | 2 000 complétions et 50 requêtes de chat par mois, mode agent et CLI limités, choix de modèles restreint |
| **Pro** | 10 $ | complétions illimitées, agent cloud, environ 1 500 crédits mensuels |
| **Pro+** | 39 $ | idem, modèles premium (classe Opus), environ 7 000 crédits |
| **Max** | 100 $ | environ 20 000 crédits ; nouvelles inscriptions suspendues en juin 2026 |
| **Business** / **Enterprise** | 19 $ / 39 $ | crédits mutualisés au niveau de l'organisation, contrôles et journaux d'audit, garanties sur les données |

Points à retenir :

- les **complétions** ne consomment pas de crédits sur les offres payantes ; le chat, le mode agent, l'agent cloud, la CLI et la revue de code en consomment, proportionnellement au modèle et à la longueur de la tâche — une session agentique de plusieurs heures coûte sans commune mesure avec une question de chat ;
- **étudiants, enseignants et mainteneurs de projets libres** bénéficient gratuitement d'une offre supérieure au Free (GitHub Education) ;
- au printemps 2026, GitHub a temporairement suspendu les nouvelles inscriptions individuelles, la demande agentique ayant dépassé la tarification par abonnement ; vérifier la disponibilité et les montants sur la page officielle avant de s'engager, ils changent plusieurs fois par an.

# 3. Activer Copilot dans VS Code

1. Ouvrir VS Code (version récente : Copilot y est intégré, sans extension à installer) et cliquer sur l'icône Copilot de la barre d'état ou de la barre de titre.
2. Se connecter avec son compte GitHub ; l'offre Free s'active sans carte bancaire.
3. Dans la vue Chat, choisir le **mode** (*Ask*, *Edit*, *Agent*) et le **modèle** dans le sélecteur ; les modèles disponibles dépendent de l'offre.
4. Régler ce qui est actif : `Ctrl+,` puis `github.copilot.enable` pour activer ou désactiver les complétions par langage, et les paramètres de confidentialité (`telemetry`, suggestions correspondant à du code public).
5. Vérifier dans un fichier que les complétions apparaissent, puis, dans le chat, demander une explication d'une fonction du projet ouvert pour tester le contexte.

**Ses propres modèles.** VS Code accepte une clé d'API personnelle (Anthropic, OpenAI, Google, Azure, OpenRouter…) et des modèles locaux servis par Ollama : le chat et le mode agent fonctionnent alors avec le modèle choisi, sans consommer de crédits GitHub — la marche à suivre pour un modèle local est dans [[LLM en local]]. Les complétions, elles, restent sur le service GitHub.

# 4. Personnaliser le comportement

La qualité d'un agent dépend du contexte et des règles qu'on lui donne ; VS Code et GitHub lisent des fichiers versionnés dans le dépôt :

- `.github/copilot-instructions.md` et `AGENTS.md` : conventions du projet, commandes de test, ce qu'il ne faut pas toucher (cours [[Visual studio code]], chapitre 20) ;
- **prompt files** (`.prompt.md`) : tâches réutilisables paramétrées (chapitre 21) ;
- **agents personnalisés** : rôles avec outils et permissions limités (chapitre 22) ;
- **serveurs MCP** : accès à des outils externes, avec les risques d'injection indirecte associés (chapitre 23).

Ces fichiers sont lus aussi par l'agent cloud sur github.com et par Copilot CLI : une seule configuration sert partout.

# 5. Bonnes pratiques et risques

- **Relire tout** : le diff est la frontière de contrôle ; un agent qui exécute des commandes le fait avec vos droits (chapitre 24 du cours VS Code).
- **Secrets** : ne jamais placer un jeton, un mot de passe ou un fichier `.env` dans le contexte ; les entreprises désactivent souvent le chat sur certains dépôts.
- **Données** : sur les offres Business et Enterprise, prompts et suggestions ne sont pas utilisés pour entraîner les modèles ; lire la politique en vigueur pour les offres individuelles et régler la télémétrie.
- **Code public** : activer le filtre des suggestions correspondant à du code public et le rapport de références si la propriété intellectuelle importe ([[Droits d'auteur]]).
- **Coût** : surveiller la consommation de crédits (page d'usage GitHub, indicateur dans VS Code) ; préférer un modèle économique pour les tâches simples et réserver les modèles premium aux tâches longues.
- **Dépendance** : Copilot n'est pas un cours ; imposer, en formation, des phases sans assistant pour que la compréhension précède la génération.

# 6. Alternatives

Le marché s'est élargi : **Claude Code** et le mode agent de [[Travailler avec Claude]], **Codex** d'OpenAI, **Gemini CLI**, les éditeurs Cursor et Windsurf, les extensions Cline et Continue, et les assistants entièrement locaux au-dessus d'Ollama ou llama.cpp ([[LLM en local]]). Copilot garde deux avantages : l'intégration native à GitHub (issues, pull requests, Actions) et la présence dans tous les éditeurs ; il n'est plus, en revanche, le seul assistant sérieux ni forcément le moins cher.

# 7. Aide-mémoire

| Besoin | Où |
|---|---|
| Offres, crédits, disponibilité | <https://github.com/features/copilot/plans> |
| Suivi de la consommation | github.com → *Settings* → *Copilot* → *Usage* |
| Activer / désactiver les complétions | paramètre `github.copilot.enable` |
| Changer de mode ou de modèle | sélecteurs de la vue Chat de VS Code |
| Instructions du projet | `.github/copilot-instructions.md`, `AGENTS.md` |
| Agent cloud | assigner une issue à `@copilot` sur github.com |
| CLI | `npm install -g @github/copilot` puis `copilot` |

# 8. Sources

- GitHub Copilot, offres et tarifs : <https://github.com/features/copilot/plans>
- Documentation GitHub Copilot : <https://docs.github.com/copilot>
- Copilot dans VS Code : <https://code.visualstudio.com/docs/copilot/overview>
- Facturation en crédits IA (juin 2026) : <https://docs.github.com/en/copilot/reference/copilot-billing>
