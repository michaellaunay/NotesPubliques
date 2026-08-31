---
schema_version: 1
uid: 01M1BQ620J01HDQFHFEWPW1V5J
titre: "Les transformers — Compléments 2026"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - intelligence-artificielle
  - apprentissage-profond
  - transformers
  - attention
  - llm
resume: "Compléments apportés au livre « Les transformers » : sections de la version condensée du cours [[Les transformers]] (31 août 2026) dont le sujet est absent de la version longue."
niveau: avance
auteurs:
  - Michaël Launay
langue: fr
date_creation: 2026-06-08
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Livre « Les transformers »
> [[Les transformers — Sommaire|Sommaire]] · [[Les transformers — 30 — Synthèse générale du cours des mécanismes d’attention aux systèmes fondés sur les Transformers|← 30 — Synthèse générale du cours des mécanismes d’attention aux systèmes fondés sur les Transformers]] · [[Les transformers — Sommaire|Sommaire →]]

# Compléments 2026

> [!info] Origine
> Les sections ci-dessous proviennent de la version condensée et actualisée du cours [[Les transformers]] (31 août 2026). Elles traitent de sujets absents de la version longue et n'ont pas été fondues dans les chapitres ; pour les versions logicielles et l'état de l'art du moment, la version condensée fait foi.

## 7.4. Post-Norm et Pre-Norm

Schéma historique post-norm :

```text
x
 ↓
Sublayer
 ↓
+ residual
 ↓
Norm
```

Schéma pre-norm courant :

```text
x ───────────────┐
 ↓               │
Norm             │
 ↓               │
Sublayer         │
 ↓               │
+ ←──────────────┘
```

Pre-norm est souvent plus facile à entraîner à grande profondeur.

## 17.4. FlexAttention

PyTorch propose aussi **FlexAttention** pour exprimer des modifications structurées des scores/masques tout en visant des kernels performants.

C'est utile lorsqu'un simple causal mask n'est pas suffisant.

Dans la documentation PyTorch actuelle, `torch.nn.attention.flex_attention` reste classé comme fonctionnalité **prototype** : son API publique est utilisable, mais il faut éviter de figer une dépendance sur ses options internes de kernel sans tests de compatibilité.

## 23.3. Bloc pre-norm

```python
class TransformerBlock(nn.Module):
    def __init__(self, d_model: int, n_heads: int, d_hidden: int):
        super().__init__()
        self.norm1 = nn.RMSNorm(d_model)
        self.attn = CausalSelfAttention(d_model, n_heads)
        self.norm2 = nn.RMSNorm(d_model)
        self.ffn = SwiGLU(d_model, d_hidden)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = x + self.attn(self.norm1(x))
        x = x + self.ffn(self.norm2(x))
        return x
```

Cette version ne contient volontairement pas :

- RoPE ;
- KV cache ;
- GQA ;
- tensor parallelism ;
- dropout détaillé ;
- optimisations de serving.

Elle sert de squelette pédagogique.

---

---

## 27.3. Pourquoi l'hybrider ?

Pour réduire :

- coût quadratique ;
- KV cache ;
- latence sur longues séquences.

L'objectif n'est pas forcément de « tuer le Transformer », mais de choisir le meilleur opérateur selon la tâche et le matériel.

---

---

## TP 7 — Profiler SDPA

Avec PyTorch :

1. créer des tenseurs FP16/BF16 sur GPU si disponible ;
2. tester plusieurs longueurs ;
3. utiliser `scaled_dot_product_attention` ;
4. profiler temps et mémoire ;
5. comparer avec une implémentation naïve.

Ne conclure qu'à partir du matériel réellement utilisé.

---

## Glossaire

**Attention**
Mécanisme produisant une combinaison de values pondérée par la compatibilité entre queries et keys.

**Self-attention**
Attention où Q, K et V proviennent de la même séquence/représentation.

**Cross-attention**
Attention où les queries et les keys/values proviennent de sources différentes.

**Causal mask**
Masque empêchant une position de consulter le futur dans une génération autoregressive.

**MHA**
Multi-Head Attention.

**MQA**
Multi-Query Attention : plusieurs têtes query partagent une paire K/V.

**GQA**
Grouped-Query Attention : plusieurs groupes de têtes query partagent plusieurs têtes K/V.

**RoPE**
Rotary Position Embedding.

**FFN**
Feed-Forward Network appliqué indépendamment aux positions d'une couche donnée.

**KV cache**
Cache des keys/values déjà calculées pendant une génération autoregressive.

**Prefill**
Traitement initial du contexte avant génération incrémentale.

**Decode**
Phase où de nouveaux tokens sont produits autoregressivement.

**FlashAttention**
Famille d'algorithmes exacts d'attention optimisant les mouvements mémoire et le calcul GPU.

**MoE**
Mixture of Experts.

**TTFT**
Time To First Token.

**SDPA**
Scaled Dot-Product Attention ; également nom de la primitive PyTorch correspondante.

---
