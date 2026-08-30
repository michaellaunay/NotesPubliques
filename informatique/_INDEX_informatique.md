---
schema_version: 1
uid: "01M05DHAZ5WYP5S4MM0Z7QA5HW"
titre: "Index informatique"
type: index
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
resume: "Index thématique généré des notes du dossier informatique."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2026-08-16
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: false
metadata_verifiees: false
---

# informatique

Index régénéré par `make index` — ne pas éditer à la main. Vue graphique : [[_INDEX_informatique.excalidraw]].

## Intelligence artificielle

- [[Copilot]] — Fiche de situation sur GitHub Copilot en 2026 : composants (complétion, chat, mode agent, agent cloud, CLI, revue de code), offres et facturation en crédits IA, activation dans VS Code, modèles et clés personnelles, personnalisation par fichiers d'instructions, bonnes pratiques et alternatives ; renvoie au cours Visual Studio Code pour le détail.
- [[Forge de Mistral]] — Note de veille sur Forge, l'orientation annoncée par Mistral AI en mars 2026 pour l'usage de l'IA en entreprise.
- [[LLM en local]] — Procédure pour faire tourner un grand modèle de langage sur sa propre machine en 2026 : vocabulaire (poids ouverts, quantification, GGUF, contexte), dimensionnement mémoire, choix d'un modèle et de sa licence, Ollama, llama.cpp, LM Studio et vLLM, API compatible OpenAI, sorties structurées, branchement d'un client ou d'un RAG, mesure, sécurité et dépannage.
- [[NotebookLM]] — Fiche sur NotebookLM, renommé Gemini Notebook en juillet 2026 : un RAG géré par Google, strictement ancré sur les documents fournis et citant ses sources, avec ses formats de restitution, ses quotas, ses limites structurelles et ce qu'il implique en matière de confidentialité.
- [[Outils IA]] — Panorama daté (août 2026) des outils d'IA générative par besoin — assistants, code, images, vidéo, voix et musique, transcription, recherche documentaire, agents, exécution locale — avec le sort des outils listés en 2023, les points de vigilance (droit d'auteur, AI Act, détecteurs, coût, données) et des renvois vers les notes détaillées du coffre.

## Numérisation et documentation

- [[Chaîne complète de numérisation OCR Markdown traduction RAG local]] — Conception d'une chaîne complète et locale de numérisation, OCR, conversion Markdown, traduction et indexation RAG pour constituer une bibliothèque personnelle interrogeable.
- [[National Emergency Library]] — Fiche sur la National Emergency Library d'Internet Archive (2020) et ses suites : principe du prêt numérique contrôlé, mécanisme réel de prêt et verrous, chronologie complète du procès Hachette v. Internet Archive jusqu'au retrait de 500 000 titres (2024), portée en Europe et en France (PNB), comparaison avec le prêt sous licence, et enseignements pour les corpus d'IA.
- [[Taille des bibliothèques]] — Recueil de chiffres sourcés (2025) et de repères historiques sur la taille des grandes bibliothèques, d'Assurbanipal à la BnF, à la Bibliothèque du Congrès et à Europeana, avec les bibliothèques numériques « sulfureuses ».

## Systèmes et administration

- [[F-Droid et Termux]] — Procédure pour installer un environnement de développement Linux sur Android en 2026 : F-Droid (magasin libre, dépôts, alternatives), Termux (sources officielles F-Droid et GitHub, différences de la version Google Play, versions 0.118.3 et 0.119 bêta), installation pas à pas, premiers réglages (pkg, stockage, SSH, Python, git), dépôt TUR et code-server pour VS Code, limites d'Android 12 à 16 et vérification des développeurs annoncée par Google.

## Réseaux et sécurité

- [[Chiffrement côté navigateur]] — Idée de chiffrement de bout en bout côté navigateur pour transmettre une pièce d'identité à des vérificateurs sans l'exposer au portail : modèle de menace, briques de 2026 (Web Crypto API, age, libsodium, OpenPGP.js), architecture hybride multi-destinataires avec exemples testés, limites (XSS, perte de clés, métadonnées), cadre RGPD et alternative par attestations d'identité (portefeuille européen).
- [[Crash github et publication de clé]] — Retour d'expérience sur une migration de serveur ratée ayant conduit à la publication accidentelle d'une clé sur GitHub.

## Programmation

- [[AirFlow]] — Fiche sur Apache Airflow 3 (2026) : ce qu'orchestre Airflow et quand l'employer, ce qui a changé avec la version 3 (Task SDK, API d'exécution, interface React, versionnage des Dags, actifs, ordonnancement événementiel), concepts, architecture, installation, exemple de Dag TaskFlow exécuté, bonnes pratiques et migration depuis Airflow 2.
- [[Modèles climatiques]] — Fiche sur les modèles climatiques vus de l'informatique (2026) : chronologie corrigée des modèles et des machines (ENIAC, Manabe, Cray-1, exascale), fonctionnement d'un modèle de climat, projets d'intercomparaison CMIP6 et CMIP7 (Assessment Fast Track pour le GIEC AR7), modèles d'apprentissage automatique (AIFS opérationnel à l'ECMWF), accès aux données CMIP en Python avec xarray et intake-esm (exemple exécuté), et repères sur la recherche française.
- [[PyPdf]] — Extraire du texte, des métadonnées, des tableaux et des structures JSON depuis des PDF en Python en 2026 : pypdf (successeur de PyPDF2), pdfplumber, PyMuPDF, pypdfium2, docling et marker ; distinguer PDF natif et PDF scanné ; exemples testés, pièges et licences.
- [[Python GMail]] — Procédure pour lire, archiver et envoyer des courriels Gmail depuis Python en 2026 : mot de passe d'application avec IMAP/SMTP ou OAuth 2.0 avec l'API Gmail, recherche par objet et par date, sauvegarde des messages et des pièces jointes, gestion des secrets et pièges courants.

## Développement web

- [[Migration Plone 5.2 python 2.7 vers 3]] — Traduction et adaptation de la documentation de la communauté Plone sur le débogage de la ZODB (zodbverify, fsoids, objets cassés, POSKeyError) et la migration de Plone 5.2 de Python 2.7 vers Python 3, complétée en 2026 par l'état des versions (fin de vie de Plone 5.2, Plone 6.2), les outils courants et le chemin de migration recommandé vers Plone 6.
- [[Présentation Nova-Ideo]] — Présentation de Nova-Ideo, logiciel libre de démocratie participative issu de la thèse CIFRE d'Amen Souissi chez Ecréall (2013), de sa méthode de génération de portails à partir des processus métier, de ses déploiements (KuneAgi pour la coopérative Cosmopolitical, Engie), et de sa situation en 2026 : reprise par Logikascium et modernisation en cours.
- [[Ressources pour le web]] — Ressources gratuites ou libres pour habiller un site ou une maquette (état août 2026) : logos de substitution, illustrations, icônes, polices, couleurs, retouche d'images, maquettes d'appareils et cartes ; licences à vérifier, liens raccourcis de 2023 résolus ou signalés.
- [[Solutions de VR]] — Panorama daté (août 2026) des solutions de réalité virtuelle et augmentée sur le web : standard WebXR et navigateurs compatibles, moteurs three.js, Babylon.js, A-Frame et PlayCanvas, mondes sociaux (Hubs Community Edition, Hyperfy, Voxels), formats glTF et USDZ, casques, et sort des liens transmis en 2023.

## Génie logiciel

- [[DevOps Project Manager]] — Courte fiche sur le rôle de DevOps Project Manager et son périmètre.
- [[UML pour Visual code]] — Procédure pour modéliser en UML dans Visual Studio Code en 2026 sans écrire d'extension : diagrammes textuels PlantUML et Mermaid avec leurs extensions et leur rendu (Java, Graphviz ou serveur PlantUML), diagrammes dessinés avec Draw.io et Excalidraw, génération de diagrammes de classes depuis du code Python avec pyreverse (exemple exécuté), et passage au modèle avec pyecore.

## Données et web sémantique

- [[LibGen]] — Fiche sur Library Genesis (LibGen) et son statut en 2026 — procès des éditeurs, saisies de domaines, rôle dans les litiges sur l'entraînement des IA — puis reformulation de l'exercice de 2023 « part d'œuvres françaises sous droits » en exercice de données ouvertes : règles du domaine public en France, requêtes SPARQL vérifiées sur data.bnf.fr et Wikidata, script Python.
- [[SPARQL et RDF]] — Fiche sur RDF et SPARQL en 2026 : modèle de triplets, IRI, littéraux, vocabulaires (RDFS, FOAF, Dublin Core, schema.org), syntaxes Turtle, JSON-LD et N-Triples, formes de requêtes SELECT, ASK, CONSTRUCT, UPDATE avec exemples exécutés en rdflib, validation SHACL, interrogation de Wikidata et data.bnf.fr, entrepôts (Jena/Fuseki, Oxigraph, GraphDB, Virtuoso), état de RDF 1.2 et SPARQL 1.2 au W3C, et liens avec Solid.

## Histoire et matériel informatique

- [[Microsoft]] — Traduction d'un article sur le rôle des parents de Bill Gates dans l'histoire de Microsoft.
- [[Émulation jeux d'arcade]] — Capture d'une référence vidéo sur les techniques d'émulation des jeux d'arcade.

## Langues et linguistique

- [[Analyse sémantique]] — Transcription d'un échange avec ChatGPT sur l'analyse sémantique.
