---
schema_version: 1
uid: 01M02JG1VDEEXGCN08Z7BGZ7ZK
titre: LLM en local
aliases:
  - Liens LLM
  - Faire tourner un LLM en local
type: procedure
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - intelligence-artificielle
  - llm
  - llama-cpp
  - ollama
  - inference-locale
resume: "Procédure pour faire tourner un grand modèle de langage sur sa propre machine en 2026 : vocabulaire (poids ouverts, quantification, GGUF, contexte), dimensionnement mémoire, choix d'un modèle et de sa licence, Ollama, llama.cpp, LM Studio et vLLM, API compatible OpenAI, sorties structurées, branchement d'un client ou d'un RAG, mesure, sécurité et dépannage."
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2024-02-29
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# LLM en local

> [!abstract] Objectif
> Faire tourner un grand modèle de langage **sur sa propre machine** sans dépendre d'une API distante : comprendre ce qui fixe la faisabilité (mémoire, quantification, contexte), choisir un modèle et sa licence, l'exécuter avec Ollama ou llama.cpp, l'exposer par une API compatible OpenAI pour y brancher un client, un éditeur ou un [[RAG]], mesurer ce que l'on obtient, et le faire sans exposer sa machine ni ses données.

> [!info] État de la note
> Vérifiée le 29 août 2026 : Ollama 0.33, llama.cpp build b10xxx (`ggml-org/llama.cpp`), Hugging Face `hf` CLI. Cette note remplace la « procédure de Vincent » de février 2024 (modèles TheBloke, binaire `main`, option `LLAMA_CUBLAS`), dont plus aucune commande ne fonctionne telle quelle ; l'ancienne version reste dans l'historique Git.

Voir aussi : [[LLM]] (ce qu'est un modèle de langage), [[RAG]] (l'exploiter sur ses documents), [[Les transformers]], [[Docker]], [[Visual studio code]], [[Travailler avec Claude]].

# Sommaire

1. Pourquoi exécuter un LLM en local
2. Vocabulaire minimal
3. Dimensionner : mémoire, quantification et contexte
4. Choisir un modèle en 2026
5. Voie 1 — Ollama
6. Voie 2 — llama.cpp
7. Voie 3 — LM Studio, vLLM, MLX et autres moteurs
8. Brancher un client : SDK, éditeur, interface Web, RAG
9. Sorties structurées et appels d'outils
10. Mesurer : vitesse et qualité
11. Sécurité, confidentialité, licences
12. Dépannage
13. Aide-mémoire
14. Sources

# 1. Pourquoi exécuter un LLM en local

- **Confidentialité** : les documents, le code et les prompts ne quittent pas la machine — condition souvent nécessaire pour des données de patients, d'élèves, de clients ou un dépôt de code privé ([[Règlement Général sur la Protection des Données (RGPD)]]).
- **Coût et disponibilité** : pas de facturation au jeton, pas de quota, fonctionnement hors ligne ; un poste de travail correctement dimensionné suffit pour le résumé, la classification, la génération de code, un assistant de rédaction ou un RAG d'équipe.
- **Contrôle** : choix du modèle, de sa version, de ses paramètres d'échantillonnage, du gabarit de conversation ; reproductibilité d'une expérience pédagogique.
- **Apprentissage** : rien ne vaut, pour comprendre le cours [[LLM]], de voir un modèle de 4 milliards de paramètres tenir dans 3 Go et répondre à 30 jetons par seconde sur un portable.

En contrepartie : les modèles à poids ouverts restent derrière les meilleurs modèles fermés sur le raisonnement long, le multimodal avancé et la fiabilité des appels d'outils ; le matériel borne la taille du modèle, donc sa qualité ; et « local » ne veut pas dire « sûr » (chapitre 11).

Ce qui a changé depuis 2024 : les dépôts de quantifications communautaires ont tourné (TheBloke est inactif depuis début 2024 ; les éditeurs publient eux-mêmes des GGUF ou passent par `unsloth`, `bartowski`, `ggml-org`), `llama.cpp` a réorganisé ses binaires (`llama-cli`, `llama-server`) et ses options de compilation (`GGML_CUDA`), Ollama est devenu le point d'entrée grand public, les modèles à **mélange d'experts** (MoE) ont rendu accessibles des modèles de 20 à 120 milliards de paramètres sur une seule machine, et le format de conversation est désormais porté par le modèle lui-même (gabarit Jinja embarqué dans le GGUF) au lieu d'être tapé à la main (`[INST] … [/INST]`).

# 2. Vocabulaire minimal

- **Poids ouverts / open source** : « poids ouverts » signifie que les paramètres sont téléchargeables ; les données et le code d'entraînement ne le sont généralement pas. La licence décide de ce que l'on peut en faire (chapitre 11).
- **Paramètres** : la taille du modèle (4B, 8B, 27B, 70B…). Pour un modèle **MoE**, distinguer paramètres **totaux** (ce qu'il faut charger en mémoire) et **actifs** par jeton (ce qui détermine la vitesse) : Gemma 4 26B-A4B, gpt-oss 120B (≈ 5B actifs), Qwen3 30B-A3B.
- **Quantification** : réduction de la précision des poids (16 bits → 8, 6, 5, 4 bits ou moins) pour tenir en mémoire, au prix d'une légère perte de qualité. Les variantes `Q4_K_M`, `Q5_K_M`, `Q6_K`, `Q8_0`, `IQ4_XS` désignent des schémas de quantification différents.
- **GGUF** : le format de fichier de `llama.cpp` (poids quantifiés, vocabulaire, gabarit de conversation, métadonnées), lu aussi par Ollama, LM Studio, Jan, etc.
- **Contexte** : nombre maximal de jetons (prompt + réponse) que le modèle traite ; sa mémoire de travail, le **cache KV**, croît avec lui.
- **Préremplissage et génération** : le traitement du prompt (*prefill*) est parallèle et rapide sur GPU ; la génération est séquentielle, un jeton à la fois, et limitée par la bande passante mémoire.
- **Jetons par seconde** : la mesure de vitesse ; en dessous de 5 à 10 jetons/s l'usage interactif devient pénible.
- **Modèles de raisonnement** : entraînés à « penser » avant de répondre (DeepSeek-R1, Magistral, gpt-oss, Qwen3 en mode *thinking*) ; meilleurs sur les problèmes multi-étapes, plus lents et plus verbeux.

# 3. Dimensionner : mémoire, quantification et contexte

## 3.1 La règle de trois de la mémoire

Un modèle quantifié occupe environ sa taille de fichier en mémoire, plus le cache KV :

```text
mémoire ≈ paramètres × bits / 8   +   cache KV (∝ contexte × couches × dimension)
```

Ordres de grandeur pour `Q4_K_M` (poids seuls, prévoir 1 à 4 Go de plus pour le contexte) :

| Modèle | Fichier Q4_K_M | Machine typique |
|---|---|---|
| 1-4B (Gemma 4 E4B, Qwen3 4B, Llama 3.2 3B) | 1-3 Go | portable sans GPU, Raspberry Pi 5 pour les plus petits |
| 7-9B (Qwen3 8B, Llama 3.1 8B, Mistral 7B) | 5-6 Go | GPU 8 Go, Mac 16 Go |
| 12-14B (Gemma 3 12B, Phi-4) | 8-9 Go | GPU 12-16 Go, Mac 24 Go |
| 24-32B (Mistral Small 3.x, Gemma 3 27B, Qwen3 32B, Qwen3-Coder 30B-A3B) | 15-20 Go | GPU 24 Go (RTX 3090/4090/5090), Mac 32-48 Go |
| 70B (Llama 3.3 70B) | 40-43 Go | 2 GPU 24 Go, Mac 64 Go |
| gpt-oss 120B (MoE, MXFP4) | ≈ 63 Go | Mac 96-128 Go, GPU 80 Go, PC 128 Go de RAM avec déchargement partiel |

Sur un Mac à mémoire unifiée ou un PC de type Ryzen AI MAX, GPU et CPU partagent la même mémoire : c'est ce qui rend les modèles de 70 à 120 milliards de paramètres accessibles hors centre de calcul.

## 3.2 Choisir la quantification

- `Q4_K_M` : le compromis par défaut ; perte de qualité faible pour un modèle ≥ 7B.
- `Q5_K_M` / `Q6_K` : à préférer si la mémoire le permet, surtout pour les petits modèles (≤ 8B), plus sensibles à la quantification.
- `Q8_0` : quasi sans perte, deux fois plus gros que Q4.
- `IQ3_*`, `IQ2_*` : pour faire entrer un gros modèle dans une petite mémoire ; la qualité chute nettement en dessous de 3 bits.
- `MXFP4` : format natif de gpt-oss ; ne pas le re-quantifier.

Règle pratique : **un modèle plus grand en Q4 bat presque toujours un modèle plus petit en Q8**.

## 3.3 Le contexte coûte cher

Le cache KV d'un modèle 8B en 16 bits pèse de l'ordre de 0,5 Go pour 4 000 jetons et plusieurs gigaoctets à 32 000. Trois leviers :

- ne demander que le contexte utile (`-c 8192` plutôt que la valeur maximale du modèle) ;
- quantifier le cache KV en 8 bits (`--cache-type-k q8_0 --cache-type-v q8_0` avec llama.cpp ; 8 bits par défaut dans Ollama depuis 2026) ;
- activer l'attention flash (`-fa on`), qui réduit la mémoire et accélère les longs contextes.

## 3.4 CPU, GPU, ou les deux

`llama.cpp` et Ollama savent répartir les couches entre GPU et CPU (**déchargement**, `-ngl`). Un modèle qui déborde du GPU tourne encore, mais à la vitesse de la RAM système ; mieux vaut alors une quantification plus agressive ou un modèle MoE, dont seuls les experts actifs travaillent à chaque jeton.

# 4. Choisir un modèle en 2026

## 4.1 Familles à poids ouverts

Le paysage change tous les mois ; en août 2026 les familles à connaître sont les suivantes (vérifier la version courante sur Hugging Face ou dans la bibliothèque Ollama) :

| Famille (éditeur) | Tailles typiques | Licence | Usage |
|---|---|---|---|
| **Qwen 3.x** (Alibaba) | 0,6B → 235B, MoE 30B-A3B, variantes Coder, Embedding | Apache 2.0 | polyvalent multilingue, référence par défaut ; excellent en code |
| **Gemma 4 / Gemma 3** (Google) | E2B, E4B, 12B, 27B, MoE 26B-A4B | Apache 2.0 (Gemma 4), conditions Gemma (3) | multimodal, très bon rapport qualité/taille, appels d'outils |
| **gpt-oss** (OpenAI) | 20B, 120B (MoE) | Apache 2.0 | raisonnement ; le 20B tient dans 16 Go |
| **Mistral Small 3.x / Magistral / Devstral** (Mistral AI) | 24B ; Ministral 3-8B | Apache 2.0 | assistant européen, raisonnement (Magistral), agents de code (Devstral) |
| **Llama 3.x / Llama 4** (Meta) | 1B → 70B ; Scout/Maverick (MoE) | licence communautaire Meta | écosystème très large ; restrictions (700 M d'utilisateurs, clauses UE sur le multimodal) |
| **DeepSeek R1 / V4** (DeepSeek) | distillations 1,5B → 70B ; V4 très gros | MIT | raisonnement et mathématiques |
| **GLM-5.x** (Zhipu), **Kimi K2.x** (Moonshot), **Nemotron 3.x** (NVIDIA) | 30B-A3B → très gros | MIT / propres licences | agents et code ; Nemotron 3.5 Lightning vise les agents « toujours actifs » |
| **Phi-4** (Microsoft) | 4B, 14B | MIT | petits modèles denses efficaces |

Modèles d'**embeddings** pour un RAG : `nomic-embed-text`, `bge-m3` (multilingue), `Qwen3-Embedding`. Modèles de **vision** : Gemma 4, Qwen3-VL, Mistral Small 3.x, Llama 4.

## 4.2 Méthode de choix

1. **Matériel d'abord** : la plus grande taille qui tient confortablement (chapitre 3), pas la plus grande qui se charge.
2. **Tâche** : code → Qwen3-Coder ou Devstral ; raisonnement → gpt-oss, Magistral, DeepSeek-R1 ; multilingue et français → Qwen3, Mistral, Gemma ; multimodal → Gemma 4, Qwen3-VL.
3. **Licence** : Apache 2.0 et MIT pour un produit ; lire les conditions Meta et Gemma avant de distribuer.
4. **Tester** sur ses propres prompts avant de conclure (chapitre 10) ; un classement public ne remplace pas dix cas réels.

## 4.3 Où télécharger

- **Bibliothèque Ollama** (`ollama pull nom:tag`) : le plus simple, quantification choisie par la bibliothèque.
- **Hugging Face** : chercher `<modèle> GGUF` ; les dépôts `ggml-org`, `unsloth` et `bartowski` publient des quantifications fiables et documentées. Téléchargement par la CLI officielle :

```bash
python -m pip install -U huggingface_hub
hf download unsloth/Qwen3-8B-GGUF --include "*Q4_K_M*" --local-dir ~/modeles
```

- **Conversion** : seulement si un modèle n'existe qu'en `safetensors` (`convert_hf_to_gguf.py` de llama.cpp, puis `llama-quantize`).

# 5. Voie 1 — Ollama

Ollama enveloppe llama.cpp (et, sur Apple Silicon, un moteur MLX) dans un service local avec une bibliothèque de modèles, un cache et deux API : la sienne et une API compatible OpenAI. C'est le point d'entrée recommandé pour débuter et pour une équipe.

## 5.1 Installer

```bash
curl -fsSL https://ollama.com/install.sh | sh    # Linux ; paquets macOS et Windows sur ollama.com
ollama --version
systemctl status ollama                          # service systemd créé par l'installateur
```

Avec [[Docker]] et un GPU NVIDIA (NVIDIA Container Toolkit installé) :

```bash
docker run -d --gpus all -v ollama:/root/.ollama -p 127.0.0.1:11434:11434 --name ollama ollama/ollama
```

## 5.2 Utiliser

```bash
ollama pull qwen3:8b               # télécharger sans lancer
ollama run gemma4:e4b              # télécharger si besoin, puis dialoguer
ollama run gpt-oss:20b "Explique le cache KV en trois phrases."
ollama ls                          # modèles présents
ollama show qwen3:8b               # architecture, contexte, quantification, licence, gabarit
ollama ps                          # modèles chargés en mémoire, part GPU/CPU
ollama rm gemma3:4b
```

Dans le dialogue, `/set parameter num_ctx 16384` change le contexte, `/set parameter temperature 0.2` l'échantillonnage, `/show info` affiche le modèle, `/bye` quitte. `ollama run --verbose …` affiche les jetons par seconde à chaque réponse.

Depuis la version 0.32 (juillet 2026), la commande `ollama` seule ouvre un agent interactif (chat, code, recherche Web) ; `ollama launch <outil>` configure en une commande des outils tiers (Claude Code, Cline, OpenClaw, Hermes…) pour qu'ils utilisent le modèle local.

## 5.3 L'API

Le service écoute sur `http://127.0.0.1:11434` :

```bash
# API native
curl http://127.0.0.1:11434/api/generate -d '{"model": "qwen3:8b", "prompt": "Bonjour", "stream": false}'

# API compatible OpenAI
curl http://127.0.0.1:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "qwen3:8b", "messages": [{"role": "user", "content": "Bonjour"}]}'
```

Variables utiles (dans `/etc/systemd/system/ollama.service.d/override.conf` ou l'environnement) :

| Variable | Rôle |
|---|---|
| `OLLAMA_HOST=127.0.0.1:11434` | adresse d'écoute ; ne pas mettre `0.0.0.0` sans reverse proxy authentifié |
| `OLLAMA_MODELS=/data/ollama` | répertoire des modèles |
| `OLLAMA_KEEP_ALIVE=30m` | durée de maintien d'un modèle en mémoire |
| `OLLAMA_NUM_PARALLEL=2` | requêtes simultanées par modèle |
| `OLLAMA_MAX_LOADED_MODELS=2` | modèles chargés en même temps |
| `OLLAMA_NO_CLOUD=1` | désactive les modèles « cloud » et les appels vers ollama.com |

## 5.4 Personnaliser avec un Modelfile

```text
FROM qwen3:8b
PARAMETER temperature 0.2
PARAMETER num_ctx 16384
SYSTEM """Tu es un assistant de relecture de code Python. Réponds en français, en citant les lignes."""
```

```bash
ollama create relecteur -f Modelfile
ollama run relecteur
```

Un Modelfile peut aussi partir d'un fichier GGUF local (`FROM ./mon-modele-Q4_K_M.gguf`) ou importer un adaptateur LoRA (`ADAPTER`).

# 6. Voie 2 — llama.cpp

`llama.cpp` (dépôt `ggml-org/llama.cpp`) est le moteur sous-jacent d'Ollama, de LM Studio et de beaucoup d'autres ; l'utiliser directement donne accès à tous les paramètres d'inférence, aux grammaires, au benchmark et à la quantification.

## 6.1 Obtenir les binaires

Binaires précompilés sur la page des *releases* GitHub (CPU, CUDA, Vulkan, Metal), images Docker `ghcr.io/ggml-org/llama.cpp` (`:server`, `:server-cuda`, `:server-vulkan`, `:full-cuda`), ou compilation :

```bash
sudo apt install build-essential cmake git
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp

cmake -B build -DCMAKE_BUILD_TYPE=Release                    # CPU (AVX2/AVX-512 détectés)
cmake -B build -DGGML_CUDA=ON -DCMAKE_BUILD_TYPE=Release     # NVIDIA : CUDA Toolkit requis
cmake -B build -DGGML_VULKAN=ON -DCMAKE_BUILD_TYPE=Release   # GPU AMD/Intel/NVIDIA via Vulkan
cmake -B build -DGGML_HIP=ON -DCMAKE_BUILD_TYPE=Release      # AMD ROCm
cmake --build build --config Release -j "$(nproc)"

ls build/bin/                                                # llama-cli, llama-server, llama-bench, llama-quantize...
```

Sur macOS, Metal est activé par défaut. Épingler un tag de build (`git checkout b10437`) dans un déploiement : les options changent entre builds.

## 6.2 Dialoguer et servir

```bash
# télécharge le GGUF depuis Hugging Face (cache ~/.cache/llama.cpp) et lance le dialogue
./build/bin/llama-cli -hf ggml-org/gemma-3-1b-it-GGUF -ngl 99

# fichier local, tout sur GPU, contexte 16k, attention flash, cache KV 8 bits
./build/bin/llama-cli -m ~/modeles/Qwen3-8B-Q4_K_M.gguf -ngl 99 -c 16384 -fa on \
    --cache-type-k q8_0 --cache-type-v q8_0

# serveur compatible OpenAI (port 8080, interface Web incluse sur /)
./build/bin/llama-server -m ~/modeles/Qwen3-8B-Q4_K_M.gguf -ngl 99 -c 16384 -fa on \
    --host 127.0.0.1 --port 8080 --api-key "$(cat ~/.llama-key)" --jinja -np 2
```

Options à connaître : `-ngl N` (couches sur GPU, `99` = toutes), `-c N` (contexte), `-fa on` (attention flash), `-t N` (fils CPU), `--jinja` (utilise le gabarit de conversation embarqué, nécessaire aux appels d'outils), `-np N` (requêtes parallèles), `--api-key`, `--embeddings` (mode embeddings), `--reasoning-format` (séparer la « pensée » d'un modèle de raisonnement).

```bash
curl http://127.0.0.1:8080/v1/chat/completions \
  -H "Authorization: Bearer $(cat ~/.llama-key)" -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Bonjour"}], "max_tokens": 200}'
```

## 6.3 Benchmarker et quantifier

```bash
./build/bin/llama-bench -m ~/modeles/Qwen3-8B-Q4_K_M.gguf -ngl 99 -p 512 -n 128
./build/bin/llama-quantize ~/modeles/Qwen3-8B-F16.gguf ~/modeles/Qwen3-8B-Q5_K_M.gguf Q5_K_M
```

`llama-bench` sépare la vitesse de préremplissage (`pp512`) et de génération (`tg128`) ; c'est la mesure à reporter quand on compare deux quantifications ou deux machines.

# 7. Voie 3 — LM Studio, vLLM, MLX et autres moteurs

| Outil | Pour qui | Points clés |
|---|---|---|
| **LM Studio** | poste de travail, découverte, Windows/macOS/Linux | interface graphique, recherche de GGUF (et MLX sur Mac), serveur compatible OpenAI sur le port 1234, CLI `lms`, mode sans interface |
| **vLLM** | serveur GPU partagé, production | débit élevé (*PagedAttention*, traitement par lots continu), poids non quantifiés ou AWQ/GPTQ/FP8, API OpenAI, `vllm serve <modèle>` ; nécessite un GPU dédié |
| **MLX** (Apple) | Mac Apple Silicon | moteur natif, souvent nettement plus rapide que llama.cpp sur Mac ; utilisé par Ollama (depuis 0.19) et LM Studio ; `mlx-lm` en Python |
| **SGLang**, **TensorRT-LLM** | production spécialisée | alternatives à vLLM, orientées débit et matériel NVIDIA |
| **Jan**, **GPT4All**, **koboldcpp** | usage personnel | interfaces au-dessus de llama.cpp |
| **`llm`** (Simon Willison) | ligne de commande, scripts | `pipx install llm` puis `llm install llm-ollama` ; journalise prompts et réponses dans SQLite, pratique pour les TP |

Règle simple : **Ollama pour que ça marche, llama.cpp pour contrôler comment ça marche, vLLM quand plusieurs utilisateurs partagent un GPU.**

# 8. Brancher un client : SDK, éditeur, interface Web, RAG

Toutes ces voies exposent la même API compatible OpenAI ; un client n'a besoin que d'une URL de base et d'un nom de modèle.

```python
from openai import OpenAI

client = OpenAI(base_url="http://127.0.0.1:11434/v1", api_key="ollama")  # ou :8080/v1 pour llama-server

reponse = client.chat.completions.create(
    model="qwen3:8b",
    messages=[
        {"role": "system", "content": "Tu réponds en français, brièvement."},
        {"role": "user", "content": "Qu'est-ce que la quantification d'un modèle ?"},
    ],
    temperature=0.2,
)
print(reponse.choices[0].message.content)
```

- **Éditeur** : dans [[Visual studio code]], l'extension Continue et Copilot Chat (fournisseur Ollama) acceptent un modèle local ; réserver les modèles de code (Qwen3-Coder, Devstral) à la complétion.
- **Interface Web** : Open WebUI (`docker run -d -p 127.0.0.1:3000:8080 -v open-webui:/app/backend/data --add-host=host.docker.internal:host-gateway ghcr.io/open-webui/open-webui:main`) offre chat, historique, documents et comptes au-dessus d'Ollama.
- **Obsidian** : les greffons d'assistants acceptent une URL compatible OpenAI ; voir le cours [[Obsidian OSIA Construire son système d'exploitation personnel augmenté par l'IA]].
- **RAG** : un modèle d'embeddings local (`ollama pull nomic-embed-text` puis `/api/embed`, ou `llama-server --embeddings`) et un modèle de génération suffisent pour le pipeline du cours [[RAG]] ; la chaîne documentaire est décrite dans [[Chaîne complète de numérisation OCR Markdown traduction RAG local]].

# 9. Sorties structurées et appels d'outils

Obtenir du JSON valide était, en 2024, l'affaire des grammaires GBNF ; c'est aujourd'hui intégré aux API.

```bash
# Ollama : format JSON contraint par un schéma
curl http://127.0.0.1:11434/api/chat -d '{
  "model": "qwen3:8b",
  "messages": [{"role": "user", "content": "Ada Lovelace, née en 1815, mathématicienne."}],
  "format": {"type": "object", "properties": {"nom": {"type": "string"}, "annee": {"type": "integer"}}, "required": ["nom", "annee"]},
  "stream": false
}'
```

Avec `llama-server`, le champ OpenAI `response_format` (`{"type": "json_schema", "json_schema": {...}}`) est traduit en grammaire ; `llama-cli --grammar-file grammars/json.gbnf` reste disponible pour contraindre n'importe quelle syntaxe.

Les **appels d'outils** (*function calling*) fonctionnent avec les modèles entraînés pour cela (Qwen3, Gemma 4, gpt-oss, Mistral Small, Llama 3.x/4) : passer `tools` dans la requête, lancer `llama-server` avec `--jinja`. Leur fiabilité en local s'est beaucoup améliorée en 2026 mais reste inférieure aux modèles fermés : valider systématiquement les arguments avant d'exécuter quoi que ce soit.

# 10. Mesurer : vitesse et qualité

- **Vitesse** : `llama-bench`, `ollama run --verbose`, ou les champs `eval_count`/`eval_duration` de l'API Ollama. Reporter séparément préremplissage et génération, et préciser contexte, quantification et matériel.
- **Qualité** : constituer un jeu de 10 à 30 prompts représentatifs avec réponses attendues, exécuter à `temperature 0` et `seed` fixe, comparer modèles et quantifications ; `llama-perplexity` pour l'effet d'une quantification sur un texte de référence ; `lm-evaluation-harness` pour les jeux de tests publics.
- **Limites des classements** : les scores publics sont sensibles à la contamination et aux gabarits ; ils orientent une présélection, ils ne remplacent pas le test sur ses propres cas.

# 11. Sécurité, confidentialité, licences

- **Exposition** : lier les serveurs à `127.0.0.1` ; pour un accès réseau, reverse proxy avec TLS et authentification (`--api-key` pour llama-server ; Ollama n'authentifie pas). Un serveur d'inférence ouvert sur Internet est trouvé et utilisé en quelques heures.
- **Local n'immunise pas** contre l'injection de prompt : un document indexé dans un RAG ou une page lue par un agent peut contenir des instructions ; encadrer les appels d'outils (chapitre 9) et ne jamais donner à un agent plus de droits qu'à l'utilisateur.
- **Télémétrie et cloud** : Ollama propose des modèles « cloud » et un agent avec recherche Web ; `OLLAMA_NO_CLOUD=1` les désactive si la politique de l'organisation l'exige. Vérifier de même les options des interfaces graphiques.
- **Licences** : Apache 2.0 (Qwen3, Gemma 4, gpt-oss, Mistral) et MIT (DeepSeek, Phi-4, GLM) autorisent l'usage commercial ; la licence Meta plafonne les usages massifs et exclut certains usages multimodaux dans l'UE ; les conditions Gemma 3 imposent une politique d'usage. Conserver la licence à côté du fichier de modèle.
- **Données d'entraînement et biais** : un modèle à poids ouverts n'est pas auditable pour autant ; documenter modèle, version et quantification dans tout résultat publié (voir aussi [[Droits d'auteur]]).
- **Ressources** : un GPU d'inférence consomme 200 à 450 W en charge ; `ollama ps` et `nvidia-smi` montrent ce qui est chargé ; décharger les modèles inutilisés (`OLLAMA_KEEP_ALIVE`).

# 12. Dépannage

| Symptôme | Cause probable | Piste |
|---|---|---|
| `out of memory` au chargement | modèle ou contexte trop grand | quantification plus petite, `-c` réduit, cache KV 8 bits, `-ngl` partiel |
| Très lent alors qu'un GPU est présent | exécution sur CPU | journal de démarrage : `CUDA`/`Metal` détecté ? pilote et CUDA Toolkit cohérents (`nvidia-smi`), binaire compilé avec le bon backend |
| Réponses incohérentes, balises étranges | mauvais gabarit de conversation | modèle `instruct`/`it` et non `base` ; `--jinja` ; GGUF récent |
| La réponse s'arrête net | contexte atteint | augmenter `-c` / `num_ctx`, réduire le prompt |
| `port already in use` | autre instance | `ollama ps`, `ss -ltnp` |
| Qualité décevante | modèle sous-dimensionné ou trop quantifié | modèle plus grand en Q4 plutôt que plus petit en Q8 ; comparer sur son jeu de prompts |
| Modèle introuvable dans Ollama | tag inexact | `ollama search` sur ollama.com/library, `ollama show` pour les variantes |

# 13. Aide-mémoire

| Besoin | Commande |
|---|---|
| Installer Ollama | `curl -fsSL https://ollama.com/install.sh \| sh` |
| Télécharger, lancer, lister, inspecter | `ollama pull`, `ollama run`, `ollama ls`, `ollama show` |
| Modèles chargés, mémoire GPU | `ollama ps`, `nvidia-smi` |
| API Ollama compatible OpenAI | `http://127.0.0.1:11434/v1` |
| Compiler llama.cpp (NVIDIA) | `cmake -B build -DGGML_CUDA=ON && cmake --build build --config Release -j` |
| Dialoguer depuis Hugging Face | `llama-cli -hf ggml-org/<modèle>-GGUF -ngl 99` |
| Servir | `llama-server -m modele.gguf -ngl 99 -c 16384 -fa on --jinja --api-key …` |
| Mesurer | `llama-bench -m modele.gguf -ngl 99` |
| Quantifier | `llama-quantize entree-F16.gguf sortie-Q5_K_M.gguf Q5_K_M` |
| Télécharger un GGUF | `hf download <depot> --include "*Q4_K_M*" --local-dir ~/modeles` |
| Sortie JSON garantie | `format` (Ollama) / `response_format` (llama-server) |

# 14. Sources

- Ollama : <https://ollama.com/>, bibliothèque <https://ollama.com/library>, dépôt et notes de version <https://github.com/ollama/ollama/releases>
- llama.cpp : <https://github.com/ggml-org/llama.cpp> — `docs/build.md`, `tools/server/README.md`, `grammars/README.md`
- Hugging Face : documentation GGUF <https://huggingface.co/docs/hub/gguf>, CLI `hf` <https://huggingface.co/docs/huggingface_hub/guides/cli>
- LM Studio : <https://lmstudio.ai/docs>
- vLLM : <https://docs.vllm.ai/>
- MLX : <https://github.com/ml-explore/mlx-lm>
- Open WebUI : <https://docs.openwebui.com/>
- `llm` : <https://llm.datasette.io/>
- lm-evaluation-harness : <https://github.com/EleutherAI/lm-evaluation-harness>
- Cartes de modèles et licences : les pages Hugging Face de Qwen, Google (Gemma), OpenAI (gpt-oss), Mistral AI, Meta (Llama), DeepSeek
