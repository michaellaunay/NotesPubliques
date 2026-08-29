---
schema_version: 1
uid: "01M15D1T00ATFJAZ4D1MA56X13"
titre: "Outils IA"
aliases:
  - "IA pour coder"
  - "IA pour créer des Images"
  - "IAs"
  - "Outils d'IA générative"
type: fiche
statut: actif
para: ressource
domaines:
  - veille
themes:
  - informatique
  - intelligence-artificielle
  - generation-d-images
  - assistance-au-developpement
  - llm
  - epistemologie
resume: "Panorama daté (août 2026) des outils d'IA générative par besoin — assistants, code, images, vidéo, voix et musique, transcription, recherche documentaire, agents, exécution locale — avec le sort des outils listés en 2023, les points de vigilance (droit d'auteur, AI Act, détecteurs, coût, données) et des renvois vers les notes détaillées du coffre."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2022-12-29
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Outils IA

> [!abstract] Objectif
> Tenir en une fiche un panorama **daté** des outils d'IA générative utiles à un développeur, un enseignant ou un créateur de contenus, classés par besoin plutôt que par nom de produit, avec ce qu'il faut vérifier avant de les adopter. Les notes détaillées du coffre — [[LLM]], [[LLM en local]], [[Copilot]], [[Travailler avec Claude]], [[NotebookLM]], [[Forge de Mistral]], [[RAG]] — traitent chaque sujet en profondeur ; cette fiche est la carte.

> [!info] État de la fiche
> Établie le 29 août 2026 en fusionnant trois listes de liens de 2022-2023 (« IA pour coder », « IA pour créer des Images », « IAs »), dont la plupart des entrées avaient disparu, changé de nom ou été dépassées. Les noms de modèles changent tous les trimestres : cette fiche cite des familles et des éditeurs, et signale ce qui doit être revérifié.

Voir aussi : [[Droits d'auteur]], [[Hermes Agent]], [[Les transformers]], [[Visual studio code]].

# 1. Assistants généralistes

| Éditeur | Produit | Ce qui le distingue |
|---|---|---|
| OpenAI | ChatGPT (modèles GPT-5.x) | écosystème le plus large : recherche, images, agents, applications tierces |
| Anthropic | Claude (Sonnet/Opus 4.x) | qualité en rédaction et en code, agents de code (Claude Code), fichiers et projets ; voir [[Travailler avec Claude]] |
| Google | Gemini | intégration Workspace, très longs contextes, NotebookLM |
| Mistral AI | Le Chat | acteur européen, modèles à poids ouverts, offre entreprise (voir [[Forge de Mistral]]) |
| Perplexity | Perplexity | recherche Web avec citations |
| Modèles ouverts | Qwen, Gemma, gpt-oss, DeepSeek, Llama… | exécutables sur sa machine : voir [[LLM en local]] |

Le choix se fait sur trois critères qui ne se lisent pas dans les classements : où vont les données (conditions d'utilisation, option d'exclusion de l'entraînement, offre entreprise), le coût réel à l'usage (abonnement contre facturation au jeton), et la possibilité de brancher ses propres documents ([[RAG]]).

# 2. Pour coder

Le sujet a sa fiche : [[Copilot]] pour GitHub Copilot et ses alternatives (Claude Code, Codex, Gemini CLI, Cursor, Cline, Continue), et le cours [[Visual studio code]] pour les instructions, prompt files, agents et MCP. Trois idées de 2022 qui ont abouti autrement :

- **Sketch2Code** (maquette → HTML) n'existe plus ; tous les assistants acceptent aujourd'hui une capture d'écran ou une maquette et produisent le code, et les agents itèrent sur le résultat rendu.
- **Générer les commentaires à partir du code** et **documenter un projet existant** sont devenus des usages standard des agents (fichier d'instructions, tâche « documente ce module »), y compris pour proposer une migration de langage — à valider par des tests, jamais à accepter en bloc.
- **Azure Cognitive Services** s'appelle **Azure AI services** ; les services dédiés (vision, OCR, parole) restent pertinents quand un modèle généraliste est trop cher ou trop lent.

# 3. Images

| Besoin | Outils (2026) | Remarques |
|---|---|---|
| Génération grand public | GPT Image (OpenAI), Midjourney, Gemini/Imagen, Adobe Firefly | Firefly revendique un entraînement sur contenus sous licence ; Midjourney reste l'outil des directions artistiques |
| Modèles à poids ouverts | FLUX (Black Forest Labs), Stable Diffusion 3.5, Qwen-Image | exécutables localement avec ComfyUI ou l'extension **Krita AI Diffusion** ; contrôle fin (ControlNet, LoRA) |
| Retouche et nettoyage | outils intégrés aux précédents, Cleanup.pictures, PicWish | l'« amélioration de photo » de 2023 est devenue une fonction standard des éditeurs |

**DALL·E 2**, cité en 2023, a été retiré par OpenAI ; les liens de l'époque vers `openai.com/dall-e-2` n'existent plus.

# 4. Vidéo, avatars, 3D

- **Génération** : Veo (Google DeepMind), Kling, Runway, Seedance, Pika, et des modèles ouverts comme Wan ; en 2026 l'audio synchronisé et les plans de plus d'une minute sont devenus courants. Sora (OpenAI) a connu plusieurs versions et repositionnements : vérifier son état avant de le citer.
- **Avatars et présentateurs** : HeyGen, Synthesia, D-ID — formation, communication interne, doublage.
- **3D et jeux** : Leonardo (racheté par Canva en 2024), Meshy et les générateurs de textures ; le passage image → modèle 3D est devenu utilisable pour le prototypage.

# 5. Voix, musique, transcription

- **Voix** : ElevenLabs (référence), Murf, Play.ht ; clonage de voix soumis au consentement et, dans l'UE, aux obligations de transparence (chapitre 9).
- **Musique** : Suno (accord avec Warner fin 2025), Udio, Aiva, Soundraw ; lire les licences d'exploitation avant tout usage commercial.
- **Transcription et sous-titres** : Whisper (OpenAI) et ses implémentations locales (`whisper.cpp`, faster-whisper) suffisent à la plupart des besoins, hors ligne ; Checksub, Fireflies, Supernormal pour la transcription et le résumé de réunions en service.

# 6. Recherche documentaire et lecture

- [[NotebookLM]] : questions sur ses propres sources avec citations, résumés audio et vidéo ; la référence pour l'enseignement.
- **Perplexity**, **Elicit**, **Consensus**, **Semantic Scholar** : recherche avec sources ; toujours ouvrir la source citée, une citation peut être inexacte.
- **Traduction** : DeepL, Google, et les LLM eux-mêmes ; les modèles ouverts traduisent correctement en local ([[Chaîne complète de numérisation OCR Markdown traduction RAG local]]).

# 7. Agents et automatisation

Les assistants savent désormais exécuter des tâches : agents de code (Claude Code, Copilot, Codex), agents de navigateur, automatisations avec **n8n** ou **Make**, agents personnels comme [[Hermes Agent]] ou OpenClaw. Règles constantes : droits minimaux, validation humaine avant toute action irréversible, journalisation, méfiance envers le contenu lu (injection de prompt). Les annuaires du type « There's an AI for That » existent toujours ; ils recensent, ils ne trient pas.

# 8. Exécuter en local

Pour la confidentialité, le coût ou la pédagogie : Ollama, LM Studio et llama.cpp pour le texte ([[LLM en local]]), ComfyUI pour l'image et la vidéo, `whisper.cpp` pour la parole. Un poste avec 16 Go de mémoire fait déjà tourner des modèles de 8 à 14 milliards de paramètres.

# 9. Vigilance

- **Droit d'auteur** : les œuvres purement générées ne sont pas protégeables (position constante du Copyright Office américain, confirmée en 2025 ; article Numerama de 2023 sur Midjourney) ; les données d'entraînement font l'objet de procès et d'accords (Suno–Warner, presse) — voir [[Droits d'auteur]].
- **AI Act** : les obligations de transparence s'appliquent depuis le **2 août 2026** : signaler les contenus générés ou manipulés (hypertrucages), marquage lisible par machine, information des personnes qui dialoguent avec une IA.
- **Détecteurs de contenu généré** (Originality et consorts) : peu fiables, faux positifs fréquents ; OpenAI a retiré son propre détecteur en 2023 pour cette raison. Ne pas fonder une sanction sur un score.
- **Données** : tout ce qui est collé dans un service en ligne peut être conservé ; exclure secrets et données personnelles, ou passer en local.
- **Coût** : les offres « illimitées » se transforment en crédits (voir [[Copilot]]) ; mesurer l'usage.
- **Épistémologie** : un modèle de langage produit du texte plausible, pas du vrai ; la conférence « L'intelligence artificielle n'existe pas » (titre du livre de Luc Julia) reste une bonne entrée en matière critique, et le cours [[LLM]] explique le mécanisme.

# 10. Ce qu'il est advenu des outils listés en 2023

| Cité en 2022-2023 | En 2026 |
|---|---|
| Sketch2Code, DALL·E 2, Genmo (alpha), tome.app | disparus ou entièrement repositionnés |
| Runway, Synthesia, D-ID, HeyGen, ElevenLabs, Murf | toujours là, devenus des acteurs installés |
| Leonardo.ai | racheté par Canva (2024) |
| Fliki, Play.ht, PicWish, Rytr, Slides AI, Namelix, Aiva, Soundraw, Checksub, Supernormal, Pollinations, Pixela, Phraser, Qoqo, FormulaBot, Tweet Hunter | existent pour la plupart, sans être devenus des références ; à vérifier au besoin |
| theresanaiforthat | annuaire toujours actif |
| Originality | détecteur ; voir les réserves du chapitre 9 |

# 11. Sources

- OpenAI : <https://openai.com/>, Anthropic : <https://www.anthropic.com/>, Google DeepMind : <https://deepmind.google/>, Mistral AI : <https://mistral.ai/>
- Black Forest Labs (FLUX) : <https://bfl.ai/> ; Stability AI : <https://stability.ai/> ; ComfyUI : <https://www.comfy.org/>
- Krita AI Diffusion : <https://github.com/Acly/krita-ai-diffusion>
- whisper.cpp : <https://github.com/ggml-org/whisper.cpp>
- AI Act, calendrier d'application : <https://artificialintelligenceact.eu/implementation-timeline/>
- Copyright Office américain, rapport sur l'IA et le droit d'auteur : <https://www.copyright.gov/ai/>
- Numerama, images Midjourney et droit d'auteur (2023) : <https://www.numerama.com/tech/1279732-les-images-generees-par-midjourney-ne-peuvent-pas-etre-protegees-par-copyright.html>
- Conférence « L'intelligence artificielle n'existe pas » : <https://youtu.be/yuDBSbng_8o>
