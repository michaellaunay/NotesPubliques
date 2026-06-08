# Formation Claude Code — 5 couches et 25 concepts fondamentaux à connaître

## Introduction

Claude Code n’est pas seulement une interface de chat pour programmer. C’est un **agent de développement** qui travaille dans le terminal, lit le projet, exécute des commandes, modifie des fichiers, vérifie son travail et peut être intégré dans des workflows plus larges : IDE, GitHub, CI/CD, MCP, subagents, skills, hooks et automatisation.

L’idée centrale à retenir est la suivante : **Claude Code transforme un modèle de langage en agent opérationnel**, capable d’agir dans un environnement de développement réel.

Les 25 concepts peuvent être regroupés en 5 couches :

1. L’architecture
    
2. Le système de contrôle
    
3. L’extension des capacités
    
4. L’automatisation
    
5. La maîtrise stratégique
    

---

# I. L’architecture

## I.1. L’écosystème Anthropic

Anthropic propose plusieurs façons d’utiliser Claude :

- **Claude.ai** : interface web conversationnelle, orientée réflexion, rédaction, analyse, documents, usage général.
    
- **Claude Desktop / applications** : usage bureautique, intégrations locales, connecteurs, environnement personnel.
    
- **Claude API** : intégration applicative, paiement à l’usage, contrôle fin des modèles, coûts, sorties structurées.
    
- **Claude Code** : agent de développement exécuté dans le terminal, connecté au code, à Git, aux fichiers, aux commandes shell et aux outils externes.
    

Claude Code s’inscrit donc dans une logique différente de Claude.ai : il ne se contente pas de répondre, il peut **agir sur un projet**.

## I.2. Claude Code vs Claude.ai

Claude.ai est surtout un assistant conversationnel. Claude Code est un agent local de développement.

La différence principale est le **rapport à l’environnement** :

|Claude.ai|Claude Code|
|---|---|
|Discussion dans une interface web|Travail dans le terminal|
|Analyse de fichiers fournis|Lecture directe du dépôt local|
|Réponses textuelles|Actions concrètes : lire, modifier, tester|
|Usage généraliste|Usage orienté code, projet, Git, CI|
|Peu ou pas d’accès système direct|Accès encadré au shell, fichiers, outils|

Claude Code est donc particulièrement adapté à :

- comprendre un codebase existant ;
    
- corriger un bug ;
    
- refactorer ;
    
- écrire ou mettre à jour des tests ;
    
- produire de la documentation technique ;
    
- analyser un diff Git ;
    
- automatiser des tâches de revue ou de maintenance.
    

## I.3. Le workflow agentique

Le fonctionnement de Claude Code repose sur une boucle agentique :

1. **Explorer**  
    Claude lit les fichiers, cherche dans le projet, comprend l’architecture, inspecte Git, identifie les dépendances.
    
2. **Planifier**  
    Il propose une stratégie, décompose la tâche, identifie les risques et les fichiers concernés.
    
3. **Coder / agir**  
    Il modifie les fichiers, exécute des commandes, lance des tests, applique des corrections.
    
4. **Vérifier**  
    Il compile, teste, relit le diff, corrige les erreurs et recommence si nécessaire.
    

La documentation officielle décrit cette boucle comme trois grandes phases : **gather context, take action, verify results**. Claude Code peut enchaîner ces phases plusieurs fois jusqu’à obtenir un résultat satisfaisant.

## I.4. Le système de permissions

Claude Code peut lire, modifier et exécuter des commandes, mais ces capacités sont encadrées.

Le système de permissions sert à décider :

- quels fichiers Claude peut lire ;
    
- quels fichiers Claude peut modifier ;
    
- quelles commandes il peut exécuter ;
    
- quels outils sont autorisés ;
    
- quels MCP servers sont accessibles ;
    
- quelles actions demandent une validation humaine.
    

Principe de base : **plus Claude est autonome, plus les permissions doivent être strictes**.

Il faut éviter de donner des permissions larges dans un environnement sensible. En particulier :

- ne jamais autoriser aveuglément les commandes destructives ;
    
- limiter l’accès aux fichiers contenant des secrets ;
    
- éviter le mode sans validation sur un environnement de production ;
    
- contrôler les serveurs MCP activés ;
    
- garder une validation humaine pour les actions à risque.
    

## I.5. Les trois modes d’opération

On peut distinguer trois grands modes d’usage.

### 1. Mode conversationnel

Claude sert à comprendre, expliquer, diagnostiquer.

Exemples :

```text
Explique-moi l’architecture de ce projet.
Quelle est la responsabilité de ce service ?
Pourquoi ce test échoue-t-il ?
```

Dans ce mode, Claude agit peu ou pas. Il lit, analyse et répond.

### 2. Mode collaboratif

Claude travaille avec l’humain, mais l’humain garde le contrôle.

Exemples :

```text
Analyse le bug, propose un plan, puis attends ma validation avant de modifier les fichiers.
Refactore ce module, mais montre-moi d’abord les fichiers impactés.
```

C’est souvent le meilleur mode pour les tâches complexes.

### 3. Mode délégation / autonomie

Claude reçoit une tâche complète et l’exécute.

Exemples :

```text
Corrige ce bug, ajoute un test de non-régression, lance la suite de tests et résume le diff.
```

Ce mode est puissant, mais il nécessite :

- un périmètre clair ;
    
- des tests ;
    
- des permissions maîtrisées ;
    
- un environnement non critique ;
    
- une relecture humaine finale.
    

---

# II. Le système de contrôle

## II.6. CLAUDE.md : la mémoire projet

Le fichier `CLAUDE.md` est un des leviers les plus importants de Claude Code.

Il sert à donner à Claude des instructions persistantes :

- architecture du projet ;
    
- conventions de code ;
    
- commandes de build ;
    
- commandes de test ;
    
- règles de nommage ;
    
- pièges connus ;
    
- choix techniques ;
    
- workflow de contribution ;
    
- règles de sécurité.
    

Exemple :

```markdown
# CLAUDE.md

## Projet
Application Node.js / TypeScript avec backend Express et frontend SolidJS.

## Commandes importantes
- Installer : `bun install`
- Lancer les tests : `bun test`
- Vérifier les types : `bun run typecheck`
- Linter : `bun run lint`

## Règles
- Ne jamais modifier les migrations Prisma sans demander confirmation.
- Toujours lancer les tests unitaires après une modification métier.
- Ne jamais écrire de secrets dans le code.
```

Claude Code charge ces fichiers au démarrage de chaque session. Ils consomment donc du contexte : il faut les garder courts, structurés et utiles.

## II.7. Les emplacements de configuration

Les fichiers `CLAUDE.md` peuvent exister à plusieurs niveaux :

|Niveau|Emplacement typique|Usage|
|---|---|---|
|Organisation|`/etc/claude-code/CLAUDE.md` sous Linux/WSL|Règles globales imposées par l’entreprise|
|Utilisateur|`~/.claude/CLAUDE.md`|Préférences personnelles|
|Projet|`./CLAUDE.md` ou `./.claude/CLAUDE.md`|Conventions partagées du dépôt|
|Local projet|`./CLAUDE.local.md`|Préférences personnelles propres au projet, normalement gitignorées|

Le projet peut être initialisé avec :

```bash
/init
```

Cette commande analyse le dépôt et génère ou améliore un fichier `CLAUDE.md`.

Point important : `/clear` ne génère pas `CLAUDE.md`. `/clear` sert à repartir avec une conversation vide.

## II.8. Les commandes de contrôle essentielles

Quelques commandes importantes :

```text
/init
```

Génère ou améliore le fichier `CLAUDE.md`.

```text
/status
```

Affiche l’état courant : compte, modèle, informations de session.

```text
/model
```

Permet de choisir ou vérifier le modèle utilisé.

```text
/compact
```

Résume la conversation pour libérer du contexte tout en gardant l’essentiel.

```text
/clear
```

Démarre une nouvelle conversation avec un contexte vide. L’ancienne session reste récupérable.

```text
/context
```

Visualise l’usage de la fenêtre de contexte.

```text
/mcp
```

Gère ou inspecte les serveurs MCP connectés.

```text
/permissions
```

Configure les règles d’autorisation.

```text
/agents
```

Gère les subagents disponibles.

```text
/resume
```

Reprend une session précédente.

## II.9. Le prompt vérifiable

Un bon prompt Claude Code doit être **spécifique, contextualisé et vérifiable**.

Mauvais prompt :

```text
Corrige les permissions.
```

Meilleur prompt :

```text
FICHIER : permissions.py
LIGNE : autour de 42
COMPORTEMENT ACTUEL : un visiteur anonyme obtient un accès en lecture.
COMPORTEMENT ATTENDU : un visiteur anonyme ne doit avoir aucun droit.
CONTRAINTE : ajoute un test de non-régression.
VÉRIFICATION : lance pytest sur le module concerné.
```

Un prompt efficace précise :

- le fichier ou le périmètre ;
    
- le comportement actuel ;
    
- le comportement attendu ;
    
- les contraintes ;
    
- la méthode de vérification ;
    
- ce qu’il ne faut pas faire.
    

Pour les tâches complexes, il faut demander explicitement :

```text
Réfléchis d’abord, explore le code, propose un plan, puis attends ma validation avant de modifier les fichiers.
```

## II.10. La gestion du contexte

La fenêtre de contexte est une ressource limitée, même lorsqu’elle est très grande.

Bonnes pratiques :

- utiliser `/clear` entre deux tâches sans rapport ;
    
- utiliser `/compact` dans une longue session ;
    
- utiliser `/compact <instruction>` pour préciser ce qu’il faut conserver ;
    
- pointer les fichiers utiles avec `@fichier` plutôt que laisser Claude chercher trop largement ;
    
- éviter de coller des logs énormes sans tri ;
    
- créer des sessions thématiques ;
    
- reprendre une session avec `/resume` ou `claude -r <session>`.
    

Exemple :

```text
@src/auth/permissions.py
@tests/test_permissions.py

Analyse uniquement ces deux fichiers et explique pourquoi le test échoue.
```

L’objectif est de ne pas transformer Claude en moteur de recherche désordonné. Plus le contexte est propre, meilleure est la réponse.

---

# III. L’extension des capacités

## III.11. MCP : Model Context Protocol

MCP permet de connecter Claude Code à des outils ou sources externes :

- bases de données ;
    
- GitHub ;
    
- systèmes de fichiers ;
    
- outils internes ;
    
- documentation ;
    
- API métiers ;
    
- navigateurs ;
    
- services SaaS.
    

Un serveur MCP expose des outils que Claude peut appeler.

Exemples d’usages :

- lire une issue GitHub ;
    
- interroger une base documentaire ;
    
- récupérer un ticket Jira ;
    
- lire une table de base de données ;
    
- appeler un outil interne de déploiement.
    

Mais chaque serveur MCP est aussi un point d’entrée de sécurité. Il faut donc n’activer que les serveurs utiles et de confiance.

## III.12. Les subagents

Les subagents permettent de déléguer des tâches spécialisées à des agents séparés, avec leur propre contexte.

Exemples de subagents :

- `security-reviewer` ;
    
- `code-reviewer` ;
    
- `test-writer` ;
    
- `documentation-writer` ;
    
- `migration-planner` ;
    
- `frontend-reviewer` ;
    
- `database-auditor`.
    

Avantages :

- spécialisation ;
    
- contexte isolé ;
    
- tâches parallélisables ;
    
- meilleure qualité sur les revues ;
    
- réduction de la pollution du contexte principal.
    

Exemple de logique :

```text
Demande au subagent security-reviewer d’analyser ce diff uniquement du point de vue sécurité.
```

Les subagents sont particulièrement utiles pour la revue, l’audit, la documentation et les tâches longues.

## III.13. Les skills

Une skill est une capacité réutilisable décrite dans un fichier `SKILL.md`, souvent placé dans :

```text
.claude/skills/nom-de-la-skill/SKILL.md
```

Elle contient des instructions spécialisées que Claude peut charger uniquement quand elles sont utiles.

Exemples :

```text
.claude/skills/code-review/SKILL.md
.claude/skills/security-audit/SKILL.md
.claude/skills/release-note/SKILL.md
.claude/skills/migration-plone/SKILL.md
```

Différence importante :

- `CLAUDE.md` : contexte général chargé au démarrage ;
    
- `SKILL.md` : procédure spécialisée chargée à la demande.
    

Il vaut mieux mettre dans une skill les procédures longues, rares ou spécialisées.

## III.14. Les commandes personnalisées

Les commandes personnalisées permettent de transformer une procédure répétitive en commande réutilisable.

Historiquement, elles peuvent être placées dans :

```text
.claude/commands/
```

Aujourd’hui, elles sont proches du mécanisme des skills. Une commande ou skill peut être appelée avec `/nom`.

Exemple :

```text
/review-api
/generate-release-note
/audit-security
```

Cela permet de capitaliser les bons prompts d’équipe.

Exemple de commande personnalisée :

```markdown
# .claude/commands/review-api.md

Analyse le diff courant du point de vue API :
- breaking changes ;
- compatibilité clients ;
- validation des entrées ;
- erreurs HTTP ;
- sécurité ;
- tests manquants.

Produis une liste priorisée.
```

Utilisation :

```text
/review-api
```

## III.15. Les entrées et sorties

Claude Code peut recevoir du contexte par plusieurs canaux.

### Entrées

1. **Prompt direct**
    

```bash
claude "Explique ce projet"
```

2. **Pipe shell**
    

```bash
git diff | claude -p "Fais une revue du diff"
```

3. **Références de fichiers**
    

```text
@src/auth.ts
@tests/auth.test.ts
```

4. **MCP**
    

Claude interroge des outils externes connectés.

5. **Git / terminal / projet**
    

Claude peut utiliser l’état du dépôt, les fichiers et les commandes disponibles.

### Sorties

Claude peut produire :

- texte ;
    
- patchs ;
    
- fichiers modifiés ;
    
- plans ;
    
- rapports ;
    
- JSON ;
    
- JSON structuré selon un schéma ;
    
- commentaires de revue ;
    
- résumés de session.
    

Pour l’automatisation, il est préférable de demander une sortie structurée.

Exemple :

```text
Réponds en JSON avec les champs :
- status
- files_changed
- tests_run
- risks
- next_steps
```

---

# IV. L’automatisation

## IV.16. Le mode headless

Le mode headless consiste à utiliser Claude Code sans interaction continue, souvent avec :

```bash
claude -p "Analyse ce diff et retourne un rapport JSON"
```

Exemple :

```bash
git diff | claude -p "Fais une revue de code et retourne uniquement du JSON"
```

Ce mode est puissant pour :

- scripts ;
    
- CI ;
    
- revue automatique ;
    
- génération de rapports ;
    
- migration de code ;
    
- analyse de logs.
    

Mais il est dangereux si on lui donne trop de permissions.

Exemple plus prudent :

```bash
git diff | claude -p "Analyse ce diff sans modifier les fichiers"
```

## IV.17. Permissions et outils autorisés

En automatisation, il faut limiter les outils.

Exemple de logique :

```bash
claude -p "Analyse le projet" \
  --allowedTools Read,Grep,Glob
```

Cela permet d’autoriser uniquement :

- lecture de fichiers ;
    
- recherche ;
    
- exploration.
    

À éviter en production :

- autoriser les éditions sans validation ;
    
- autoriser les commandes destructives ;
    
- autoriser un accès non contrôlé aux secrets ;
    
- donner accès à tout le système de fichiers ;
    
- connecter tous les serveurs MCP par défaut.
    

Principe : **en headless, le moindre privilège est obligatoire**.

## IV.18. Hooks

Les hooks permettent d’automatiser des actions déterministes à certains moments du cycle de vie de Claude Code.

Exemples :

- formatter automatiquement après modification ;
    
- bloquer une commande dangereuse ;
    
- refuser l’accès à certains fichiers ;
    
- notifier quand Claude attend une action ;
    
- injecter du contexte après une compaction ;
    
- auditer les changements de configuration.
    

Les hooks sont plus fiables que de simples instructions dans `CLAUDE.md`, car ils s’exécutent de manière déterministe.

Exemple d’usage :

```text
Bloquer toute lecture de fichiers .env, secrets.json, id_rsa, *.pem.
```

## IV.19. CI/CD

Claude Code peut être intégré dans des workflows CI/CD.

Exemple de cas d’usage :

- revue automatique de Pull Request ;
    
- analyse de dette technique ;
    
- génération de release notes ;
    
- triage d’issues ;
    
- vérification de conventions ;
    
- analyse de tests échoués.
    

Exemple de workflow GitHub Actions :

```yaml
name: Claude Review

on:
  pull_request:

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Claude review
        run: |
          git diff origin/main...HEAD | claude -p "Review this PR and return risks, bugs and missing tests."
```

Dans un vrai contexte d’entreprise, il faut gérer :

- l’authentification ;
    
- les secrets ;
    
- les permissions ;
    
- le coût ;
    
- les logs ;
    
- les données envoyées au modèle ;
    
- les règles de conformité.
    

## IV.20. Patterns multiagents et chaînage de sessions

Claude Code peut être utilisé avec plusieurs sessions ou agents en parallèle.

Patterns possibles :

### 1. Analyse parallèle

Un agent analyse la sécurité, un autre les tests, un autre la documentation.

### 2. Worktree par tâche

Chaque agent travaille dans un worktree Git isolé.

### 3. Revue croisée

Un agent code, un autre relit.

### 4. Chaînage de session

Une session produit un résumé structuré, repris ensuite par une autre session.

Exemple :

```text
Résume cette session en JSON avec :
- objectif
- décisions prises
- fichiers modifiés
- tests lancés
- risques restants
- prochaine action recommandée
```

Puis :

```bash
claude -r "auth-refactor" "Continue à partir du résumé et termine les tests."
```

---

# V. La maîtrise stratégique

## V.21. L’économie des tokens

Les tokens sont l’unité de coût et de contexte.

Ils sont consommés par :

- le prompt ;
    
- les fichiers lus ;
    
- `CLAUDE.md` ;
    
- l’historique de conversation ;
    
- les sorties du modèle ;
    
- les appels outils ;
    
- les documents MCP ;
    
- les logs collés ;
    
- les résumés et vérifications.
    

Optimiser les tokens signifie :

- réduire le contexte inutile ;
    
- pointer les bons fichiers ;
    
- compacter régulièrement ;
    
- séparer les tâches ;
    
- éviter les logs bruts énormes ;
    
- utiliser des skills plutôt que surcharger `CLAUDE.md` ;
    
- utiliser le prompt caching quand c’est pertinent ;
    
- utiliser le Batch API pour les traitements non urgents.
    

Le prompt caching réduit le coût et la latence en réutilisant des portions répétées du prompt. Les lectures de cache coûtent une fraction du prix d’entrée standard. Le Batch API peut aussi réduire les coûts pour les traitements asynchrones ou non urgents.

## V.22. Le choix du modèle

Les familles de modèles Claude sont généralement organisées en trois niveaux :

### Haiku

Modèle rapide et économique.

À utiliser pour :

- tâches simples ;
    
- classification ;
    
- extraction ;
    
- reformulation légère ;
    
- scripts répétitifs ;
    
- analyse peu risquée.
    

### Sonnet

Modèle équilibré.

À utiliser pour :

- la majorité des tâches de code ;
    
- refactoring courant ;
    
- debug ;
    
- tests ;
    
- documentation ;
    
- revue standard.
    

### Opus

Modèle le plus puissant.

À utiliser pour :

- architecture complexe ;
    
- audit critique ;
    
- analyse de sécurité ;
    
- gros refactoring ;
    
- raisonnement difficile ;
    
- arbitrages techniques.
    

Règle pratique :

```text
Haiku pour le volume.
Sonnet pour le quotidien.
Opus pour les décisions difficiles.
```

## V.23. Sécurité

La sécurité est centrale avec Claude Code, car l’outil peut agir sur un environnement réel.

Principes fondamentaux :

1. **Ne jamais mettre de clé API dans le code**
    

Utiliser :

- variables d’environnement ;
    
- gestionnaire de secrets ;
    
- fichiers ignorés par Git ;
    
- coffre type Vault, Doppler, SOPS, etc.
    

2. **Bloquer les fichiers sensibles**
    

À protéger :

```text
.env
.env.*
secrets.json
id_rsa
*.pem
*.key
credentials.*
```

3. **Limiter les commandes dangereuses**
    

À encadrer strictement :

```bash
rm -rf
sudo
chmod -R
chown -R
docker system prune
kubectl delete
terraform destroy
drop database
```

4. **Contrôler les MCP servers**
    

Chaque serveur MCP peut introduire :

- injection de prompt ;
    
- tool poisoning ;
    
- fuite de données ;
    
- accès excessif ;
    
- appels non maîtrisés à des services externes.
    

5. **Garder une validation humaine**
    

Surtout pour :

- production ;
    
- infrastructure ;
    
- base de données ;
    
- sécurité ;
    
- secrets ;
    
- déploiement ;
    
- facturation ;
    
- suppression de données.
    

## V.24. Déployer Claude Code en entreprise

En entreprise, Claude Code devient un outil d’ingénierie logiciel et non plus seulement un assistant individuel.

Les enjeux principaux sont :

- gouvernance ;
    
- SSO / SAML ;
    
- gestion des comptes ;
    
- contrôle des permissions ;
    
- politique MCP ;
    
- audit des logs ;
    
- conformité ;
    
- gestion des coûts ;
    
- séparation des environnements ;
    
- protection des secrets ;
    
- formation des équipes.
    

Une équipe peut mutualiser :

- des `CLAUDE.md` projet ;
    
- des skills ;
    
- des commandes personnalisées ;
    
- des subagents ;
    
- des hooks ;
    
- des workflows CI ;
    
- des règles de sécurité ;
    
- des modèles de prompts.
    

L’objectif est de capitaliser les bonnes pratiques au lieu de laisser chaque développeur réinventer son usage.

## V.25. Vision stratégique

Claude Code préfigure une couche d’exécution agentique entre :

- le développeur ;
    
- le terminal ;
    
- l’IDE ;
    
- Git ;
    
- les tickets ;
    
- les bases de données ;
    
- les outils internes ;
    
- les pipelines CI/CD ;
    
- les documents ;
    
- les navigateurs ;
    
- les systèmes métier.
    

La vision n’est pas seulement “un chatbot qui aide à coder”, mais un **système d’agents connectés aux outils de l’entreprise**.

Le développeur ne disparaît pas. Son rôle évolue :

- il définit l’objectif ;
    
- il cadre le périmètre ;
    
- il choisit les contraintes ;
    
- il valide les décisions ;
    
- il contrôle la sécurité ;
    
- il relit les changements ;
    
- il transforme les bonnes pratiques en automatisations.
    

L’utilisateur débutant demande à Claude de répondre.  
L’utilisateur avancé demande à Claude d’agir.  
L’utilisateur stratégique construit un système de travail autour de Claude.

---

# Conclusion

Les 5 idées essentielles à retenir :

1. **Claude Code est un agent**  
    Il ne se contente pas de répondre : il explore, agit, modifie, teste et vérifie.
    
2. **Le contexte est le levier principal**  
    `CLAUDE.md`, `/compact`, `/clear`, `@fichier`, sessions thématiques et prompts précis déterminent fortement la qualité.
    
3. **L’écosystème est extensible**  
    MCP, subagents, skills, commandes personnalisées et hooks permettent de spécialiser Claude Code.
    
4. **L’automatisation est puissante mais risquée**  
    Headless, CI/CD et multiagents peuvent industrialiser le travail, mais nécessitent permissions, sandbox et validation humaine.
    
5. **La maîtrise stratégique dépasse l’outil**  
    Le vrai enjeu est de construire une méthode : coûts, sécurité, modèles, gouvernance, capitalisation d’équipe et intégration dans l’infrastructure.