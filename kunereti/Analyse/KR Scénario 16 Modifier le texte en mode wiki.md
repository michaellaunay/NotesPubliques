---
statut: brouillon
para: projet
domaines:
- recherche
auteurs:
- Michaël Launay
confidentialite: publique
publication:
- notes-publiques
rag: true
titre: 'KR Scénario 16 : Modifier le texte en mode wiki'
type: document
themes:
- genie-logiciel
- scenarios
- git
- markdown
- kuneagi
resume: 'Scénario de co-écriture en mode wiki : chaque modification enregistrée produit une nouvelle version, sans validation préalable.'
schema_version: 1
uid: 01M7AK88XG7C36HJKK7VJD57RM
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Modifier le texte en mode wiki**

## Résumé
Le groupe travaille en mode wiki. Alice rédige la section « Financement » ; chaque modification qu'elle enregistre produit une nouvelle version, sans validation. Le vote de publication portera sur une version précise, désignée par son empreinte : une retouche de dernière minute donne une autre version, sur laquelle personne n'a voté.

## Acteurs en jeu
- **Alice** : participante du groupe.
- **Poste de travail** : Obsidian ou le portail local.
- **Dépôt du groupe**.
- **Vérificateur**.

## Prérequis
- L'itération en cours est en mode wiki.
- Alice est participante du groupe.
- Son poste a synchronisé le dépôt du groupe.

## Étapes du scénario
1. Alice ouvre la section « Financement » sur son poste.
2. Elle rédige. Son travail en cours reste dans sa copie personnelle.
3. Elle enregistre sa modification, avec une phrase qui dit ce qu'elle a changé.
4. Le poste signe la modification et la propose au dépôt du groupe.
5. Le vérificateur contrôle trois choses seulement : le schéma, la signature d'Alice et son appartenance au groupe.
6. La modification est fusionnée ; l'empreinte de la section et celle de la proposition sont mises à jour.
7. Si Bob a modifié la même section entre-temps, le poste d'Alice lui montre les deux versions et lui demande de les réconcilier avant d'envoyer.
8. Les autres participants voient la nouvelle version à leur synchronisation.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant Al as Alice
    participant P as Poste de travail
    participant G as Dépôt du groupe
    participant V as Vérificateur
    Al->>P: Ouvre et modifie la section
    Al->>P: Enregistre la modification
    P->>P: Signe
    P->>G: Propose la modification
    alt Section inchangée depuis la synchronisation
        G->>V: Demande le contrôle
        V-->>G: Schéma, signature, appartenance valides
        G->>G: Fusionne, met à jour les empreintes
    else Section modifiée entre-temps par Bob
        G-->>P: Version plus récente
        P->>Al: Demande de réconcilier les deux versions
    end
```

Ce diagramme illustre le mode qui convient dans la plupart des cas, au démarrage, quand le texte est encore informe. Son défaut de 2017, le poids donné à celui qui modifie le dernier, a disparu : on vote sur une empreinte, pas sur un titre.

Cas d'utilisation : « Co-écrire une proposition en groupe de travail ». Retour à l'analyse : [[KR Analyse]].
