---
schema_version: 1
uid: "01M02JG1VEB0H5X199SABRMKM1"
titre: "National Emergency Library"
aliases:
  - "Hachette v. Internet Archive"
  - "Prêt numérique contrôlé"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - droit-d-auteur
  - bibliotheques
  - internet-archive
  - pret-numerique
resume: "Fiche sur la National Emergency Library d'Internet Archive (2020) et ses suites : principe du prêt numérique contrôlé, mécanisme réel de prêt et verrous, chronologie complète du procès Hachette v. Internet Archive jusqu'au retrait de 500 000 titres (2024), portée en Europe et en France (PNB), comparaison avec le prêt sous licence, et enseignements pour les corpus d'IA."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-03-29
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# National Emergency Library

> [!abstract] Objectif
> Comprendre ce qu'a été la National Emergency Library d'Internet Archive au printemps 2020, sur quelle théorie juridique — le prêt numérique contrôlé — elle reposait, comment fonctionnait réellement le prêt, et ce que le procès *Hachette v. Internet Archive* a décidé jusqu'à son terme en décembre 2024 ; puis situer ces décisions américaines par rapport au droit européen et au prêt numérique en bibliothèque française.

> [!info] État de la fiche
> Complétée et corrigée le 29 août 2026. La version de 2023, issue d'un échange avec ChatGPT, s'arrêtait à la décision de première instance et comportait plusieurs erreurs de fait — sur l'origine des fichiers (Internet Archive numérise lui-même les livres, c'est le cœur du litige) et sur l'absence de verrou de prêt. Les dates et chiffres ci-dessous proviennent des décisions publiées et des communiqués des parties.

Voir aussi : [[Droits d'auteur]], [[LibGen]], [[Taille des bibliothèques]].

# 1. Les faits

Internet Archive, organisation à but non lucratif de San Francisco, numérise depuis les années 2000 des livres imprimés — les siens et ceux de bibliothèques partenaires — et les prête sous forme numérique dans son **Open Library**. Le principe, baptisé **prêt numérique contrôlé** (*controlled digital lending*, CDL), veut qu'une copie numérique ne circule qu'à raison d'un exemplaire prêté par exemplaire papier possédé (« *owned-to-loaned ratio* »), l'exemplaire papier étant retiré de la circulation.

Le **24 mars 2020**, pendant le confinement, Internet Archive a suspendu cette règle : la **National Emergency Library** a supprimé les listes d'attente et autorisé des emprunts simultanés illimités d'un même titre, pour environ 1,4 million de livres. Devant les protestations d'auteurs et d'éditeurs, le programme a été fermé le **16 juin 2020**, deux semaines avant la date prévue. Le **1er juin 2020**, Hachette, HarperCollins, Penguin Random House et Wiley avaient assigné Internet Archive devant le tribunal fédéral du district sud de New York.

# 2. Comment fonctionnait réellement le prêt

- L'utilisateur crée un compte, emprunte jusqu'à **dix livres** à la fois, pour **quatorze jours** (ou une heure renouvelable en lecture dans le navigateur).
- Le livre se lit dans la liseuse Web d'Internet Archive ou se télécharge sous forme de fichier protégé par un **verrou numérique** (Adobe DRM) qui expire à la fin du prêt : contrairement à ce qu'affirmait la fiche de 2023, il existait bien un mécanisme de retour, automatique.
- Les fichiers sont des **numérisations** réalisées par Internet Archive à partir d'exemplaires papier ; ce ne sont pas des livres numériques d'éditeur. Aucun filigrane nominatif n'était apposé.
- Aucun verrou n'est infaillible : la capture d'écran ou le contournement du DRM restent possibles, comme pour tout service de prêt, sans que l'on dispose de chiffres sur les fuites.

En régime normal (hors NEL), les emprunts d'un titre sont plafonnés au nombre d'exemplaires papier détenus, avec liste d'attente.

# 3. Chronologie du procès *Hachette v. Internet Archive*

| Date | Événement |
|---|---|
| 1er juin 2020 | assignation par les quatre éditeurs, sur un échantillon de 127 ouvrages ; les pièces évoquent 3,6 millions d'œuvres potentiellement sous droits dans la collection |
| 24 mars 2023 | jugement du juge Koeltl : le prêt numérique contrôlé n'est pas un usage loyal (*fair use*) — usage non transformatif, concurrence directe avec le marché des livres numériques sous licence |
| août 2023 | jugement d'accord : injonction permanente sur les titres disponibles en version numérique chez les éditeurs plaignants, indemnisation d'un montant non divulgué |
| 4 septembre 2024 | la cour d'appel du deuxième circuit **confirme** intégralement, dans un arrêt de 64 pages qui écarte la théorie du prêt numérique contrôlé |
| 21 septembre 2024 | Internet Archive annonce avoir retiré du prêt **plus de 500 000 livres** et en prévoit d'autres |
| 3-4 décembre 2024 | Internet Archive renonce à saisir la Cour suprême ; l'affaire est close |

La décision ne condamne pas la numérisation à des fins de conservation, mais la mise en prêt de copies intégrales d'œuvres sous droits sans licence. Elle a été immédiatement invoquée, par les éditeurs comme par les auteurs, dans les procès de 2025 sur les corpus d'entraînement des modèles d'IA (voir [[LibGen]]).

# 4. Et en Europe ?

Le droit américain du *fair use* n'a pas d'équivalent en Europe, où les exceptions sont limitativement énumérées. La Cour de justice de l'Union européenne a jugé en 2016 (*Vereniging Openbare Bibliotheken*) que le prêt d'un livre numérique par une bibliothèque publique peut relever de l'exception de prêt public, à condition de reproduire le modèle du papier : **un exemplaire, un emprunteur à la fois**, copie acquise licitement, expiration à la fin du prêt. Le prêt numérique contrôlé « à la française » est donc concevable, mais sur des fichiers obtenus légalement, pas sur des numérisations d'exemplaires papier.

En pratique, les bibliothèques françaises prêtent des livres numériques via **PNB** (Prêt Numérique en Bibliothèque), dispositif interprofessionnel porté par Dilicom : chaque bibliothèque achète des licences par titre (nombre de prêts simultanés, nombre total de prêts, durée), et les lecteurs empruntent avec des verrous (LCP ou Adobe). L'offre dépend des conditions fixées par chaque éditeur, souvent plus restrictives et plus coûteuses que pour le papier — ce qui explique l'attrait, et le risque, du modèle d'Internet Archive.

# 5. Prêt sous licence : OverDrive, PNB, Kobo

| Service | Modèle | Verrou |
|---|---|---|
| **OverDrive / Libby** (monde anglophone, quelques bibliothèques françaises) | licences négociées avec les éditeurs, exemplaires simultanés limités, listes d'attente | DRM avec expiration automatique |
| **PNB** (France) | licences par titre achetées par la bibliothèque, prêts simultanés et totaux plafonnés | LCP (Readium) ou Adobe |
| **Kobo, Amazon, Apple** | vente ou abonnement de livres numériques à l'unité ; ce n'est pas du prêt | DRM propriétaire ; l'accès dépend du compte |

Aucune bibliothèque numérique n'a accès à « tous les livres » : l'offre est celle que les éditeurs acceptent de licencier, titre par titre. Pour savoir si sa bibliothèque propose PNB, consulter son site ou le portail de la médiathèque départementale.

# 6. Les livres ont-ils fui vers des sites pirates ?

La question posée en 2023 reste sans réponse documentée : aucune étude publique ne relie les numérisations d'Internet Archive aux collections de Library Genesis, dont les fichiers proviennent pour l'essentiel d'autres sources (livres numériques d'éditeurs dépourvus de leur verrou). Ce qui est établi, en revanche, c'est que des corpus issus de LibGen ont servi à entraîner des modèles de langage, ce qui a déplacé le débat de la lecture vers l'entraînement ([[LibGen]], chapitre 1).

# 7. Ce qu'il faut retenir

- Le prêt numérique contrôlé était une **théorie juridique**, pas une exception reconnue ; aux États-Unis elle a été rejetée en 2023 et 2024.
- La NEL n'a pas créé le litige ; elle l'a précipité en supprimant la seule limite qui rendait le modèle défendable.
- En Europe, le prêt numérique en bibliothèque existe, mais sous licence et un exemplaire à la fois.
- Les mêmes questions — provenance des fichiers, copie intégrale, effet sur le marché — structurent les procès de 2025-2026 sur l'intelligence artificielle.

# 8. Sources

- Arrêt du deuxième circuit, *Hachette Book Group v. Internet Archive*, 4 septembre 2024 : <https://law.justia.com/cases/federal/appellate-courts/ca2/23-1260/23-1260-2024-09-04.html>
- Internet Archive, « Lending of Digitized Books » (21 septembre 2024) : <https://blog.archive.org/2024/09/21/lending-of-digitized-books/>
- Association of American Publishers, communiqué de fin de procédure (4 décembre 2024) : <https://publishers.org/news/aap-celebrates-final-victory-in-infringement-case-against-internet-archive/>
- Communia, « Internet Archive loses appeal – what does it mean? » (portée européenne, 2024) : <https://communia-association.org/2024/09/25/internet-archive-loses-appeal-what-does-it-mean/>
- CJUE, affaire C-174/15, *Vereniging Openbare Bibliotheken*, 10 novembre 2016
- Déclaration sur le prêt numérique contrôlé (2018) : <https://controlleddigitallending.org/>
- Next INpact, « Internet Archive perd son procès contre Hachette et compagnie » (mars 2023) : <https://www.nextinpact.com/article/71336/internet-archive-perd-son-proces-contre-hachette-et-compagnie>
- PNB, Prêt Numérique en Bibliothèque : <https://www.dilicom.net/pnb/>
