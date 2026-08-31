---
schema_version: 1
uid: 01M1BQ62BADPYFH34DA2M9TV7V
titre: "Les CNN et RNN — Compléments 2026"
type: cours
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - intelligence-artificielle
  - apprentissage-profond
  - reseaux-de-neurones
  - cnn
  - rnn
resume: "Compléments apportés au livre « Les CNN et RNN » : sections de la version condensée du cours [[Les CNN et RNN]] (31 août 2026) dont le sujet est absent de la version longue."
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

> [!info] Livre « Les CNN et RNN »
> [[Les CNN et RNN — Sommaire|Sommaire]] · [[Les CNN et RNN — 08 — Les modèles Seq2Seq|← 08 — Les modèles Seq2Seq]] · [[Les CNN et RNN — Sommaire|Sommaire →]]

# Compléments 2026

> [!info] Origine
> Les sections ci-dessous proviennent de la version condensée et actualisée du cours [[Les CNN et RNN]] (31 août 2026). Elles traitent de sujets absents de la version longue et n'ont pas été fondues dans les chapitres ; pour les versions logicielles et l'état de l'art du moment, la version condensée fait foi.

## Chapitre 12 — Évaluation, diagnostic, robustesse et exploitation

---

### Ne pas juger uniquement la loss train

Il faut suivre :

```text
train loss
validation loss
métrique métier
```

Une baisse continue de train loss avec une validation qui se dégrade indique probablement de l'overfitting.

### Classification

Mesures possibles :

- accuracy ;
- precision ;
- recall ;
- F1 ;
- macro F1 ;
- ROC-AUC selon le cas ;
- PR-AUC pour classes rares ;
- matrice de confusion.

### Segmentation

Mesures typiques :

- IoU/Jaccard ;
- Dice ;
- métriques par classe.

### Séries temporelles

Selon le problème :

- MAE ;
- RMSE ;
- MAPE avec prudence ;
- métriques de détection d'événements ;
- latence et coût mémoire.

### Debugger un CNN

Checklist :

1. shapes correctes ?
2. plage des valeurs d'entrée correcte ?
3. normalisation compatible avec les poids pré-entraînés ?
4. labels corrects ?
5. loss compatible avec les logits ?
6. `train()`/`eval()` corrects ?
7. gradients non nuls/non NaN ?
8. augmentation trop agressive ?
9. fuite train/test ?

### Debugger un RNN

1. ordre des dimensions correct ?
2. `batch_first` cohérent ?
3. padding traité ?
4. longueur réelle des séquences connue ?
5. état réinitialisé ou conservé volontairement ?
6. état détaché au bon moment ?
7. gradient clipping si nécessaire ?
8. bidirectionnalité compatible avec la causalité ?
9. métrique évaluée sans fuite du futur ?

### Overfit un mini-batch

Excellent test de plomberie : essayer de surapprendre volontairement un tout petit sous-ensemble.

Si le modèle ne peut pas mémoriser quelques exemples :

- bug de données ;
- mauvaise loss ;
- modèle mal connecté ;
- learning rate inadéquat ;
- gradients cassés.

---

---

### Distribution shift

Un excellent score de validation n'implique pas un bon fonctionnement après changement de :

- caméra ;
- saison ;
- appareil ;
- population ;
- fréquence d'échantillonnage ;
- protocole de collecte.

### Adversarial examples

Les modèles de vision peuvent être sensibles à de petites perturbations spécialement conçues.

Ne pas confondre robustesse à une augmentation naturelle et robustesse adversariale.

### Fuites temporelles

Pour une série temporelle, un split aléatoire peut fuiter du futur vers le passé.

Préférer selon le problème :

```text
train = passé
validation = futur proche
test = futur encore plus récent
```

### Données sensibles

Images, audio et séries de capteurs peuvent contenir :

- biométrie ;
- géolocalisation ;
- données médicales ;
- habitudes ;
- secrets industriels.

Voir [[Règlement Général sur la Protection des Données (RGPD)]].

### Modèles pré-entraînés

Un checkpoint externe est un artefact logiciel.

Bonnes pratiques :

- provenance ;
- hash/digest ;
- licence ;
- version ;
- format sûr ;
- éviter les désérialisations arbitraires non fiables.

### Mesurer le système entier

En production, mesurer aussi :

- latence p50/p95/p99 ;
- mémoire ;
- consommation ;
- débit ;
- taux d'erreur ;
- dérive ;
- qualité par sous-population.

Un modèle légèrement moins précis mais beaucoup plus petit peut être le meilleur système.

---

---
