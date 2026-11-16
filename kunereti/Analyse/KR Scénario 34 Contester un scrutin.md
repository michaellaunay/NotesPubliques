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
titre: 'KR Scénario 34 : Contester un scrutin'
type: document
themes:
- genie-logiciel
- scenarios
- democratie
- securite
- kuneagi
resume: 'Scénario d''erreur : un bulletin est écarté à tort à la clôture ; le votant apporte la preuve, un jury tiré au sort annule le scrutin, qui est rouvert.'
schema_version: 1
uid: 01M7AK890AWQN752E88EM7RBK1
date_creation: 2026-11-13
langue: fr
date_modification: 2026-11-13
metadata_verifiees: false
---
# **Contester un scrutin**

## Résumé
Bob a voté à temps, mais son bulletin ne figure pas dans la liste figée à la clôture. Il n'y a pas d'horloge commune ni d'urne centrale pour trancher. Bob produit ses preuves : son bulletin signé et l'historique de sa copie. Un jury tiré au sort examine la contestation et annule le scrutin, qui est rouvert.

## Acteurs en jeu
- **Bob** : le votant écarté.
- **Jury** : des membres tirés au sort.
- **Dépôt du groupe**.
- **Autorité du groupe**.

## Prérequis
- Un scrutin est clos et dépouillé.
- Bob détient un bulletin signé pour ce scrutin, absent de la liste figée à la clôture.
- La charte fixe le délai de contestation.

## Étapes du scénario
1. Bob constate, en rejouant le dépouillement, que son bulletin n'a pas été compté.
2. Il dépose une contestation signée : son bulletin, et l'historique de sa copie montrant qu'il avait été envoyé avant la clôture.
3. La fusion du résultat est suspendue pendant le délai de contestation.
4. Un jury est tiré au sort de façon vérifiable, hors des participants au scrutin si la taille de la communauté le permet.
5. Le jury examine les preuves : le bulletin est valide, et son absence pouvait changer le résultat.
6. Les jurés déposent leurs avis signés ; la majorité annule.
7. Le scrutin passe à l'état `void`. Un nouveau scrutin est ouvert sur les mêmes candidats, avec le même corps électoral.
8. Si le bulletin n'avait pas pu changer le résultat, le jury aurait pu maintenir le scrutin en le faisant consigner.

## Diagramme de séquence Mermaid
```mermaid
sequenceDiagram
    participant B as Bob
    participant G as Dépôt du groupe
    participant J as Jury tiré au sort
    participant U as Autorité du groupe
    B->>B: Rejoue le dépouillement, bulletin absent
    B->>G: Dépose la contestation et ses preuves
    Note over G: Fusion du résultat suspendue
    G-->>J: Jury désigné par tirage au sort
    J->>G: Examinent bulletin et historique
    J->>G: Déposent leurs avis signés
    alt Contestation fondée
        G->>G: Scrutin à l'état void
        U->>G: Ouvre un nouveau scrutin
    else Contestation rejetée
        U->>G: Fusionne le résultat initial
    end
```

Ce diagramme illustre la réponse de Kunereti à l'absence d'horloge commune : la clôture fige une liste, et celui qui en est exclu à tort dispose de preuves publiables. C'est plus lourd qu'un horodatage central, et c'est le prix de son absence.

Cas d'utilisation : « Garantir la confiance sans serveur ». Retour à l'analyse : [[KR Analyse]].
