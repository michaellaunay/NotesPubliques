---
schema_version: 1
uid: "01M02JG1VBMRPJMSQVR0F74J75"
titre: "Chiffrement côté navigateur"
aliases:
  - "Chiffrement de bout en bout dans le navigateur"
type: idee
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - securite
  - cryptographie
  - vie-privee
  - identite-numerique
resume: "Idée de chiffrement de bout en bout côté navigateur pour transmettre une pièce d'identité à des vérificateurs sans l'exposer au portail : modèle de menace, briques de 2026 (Web Crypto API, age, libsodium, OpenPGP.js), architecture hybride multi-destinataires avec exemples testés, limites (XSS, perte de clés, métadonnées), cadre RGPD et alternative par attestations d'identité (portefeuille européen)."
maturite: a-experimenter
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-08-01
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Chiffrement côté navigateur

> [!abstract] Objectif
> Permettre à un candidat de transmettre une pièce d'identité à quelques **vérificateurs** d'un portail associatif sans que le portail, ses administrateurs, ses sauvegardes ni un courriel intercepté puissent la lire : chiffrement **de bout en bout** effectué dans le navigateur du candidat, pour les seules clés publiques des vérificateurs. Cette note fixe le modèle de menace, les briques utilisables en 2026, une architecture, ses limites et ce qu'il faudrait expérimenter avant de la retenir.

> [!info] État de l'idée
> Reprise le 29 août 2026. Les bibliothèques citées en 2023 sont à écarter : **CryptoJS** n'est plus maintenue (dernière version 4.2.0, octobre 2023, avec avertissement de fin de vie), **sjcl** est arrêtée depuis des années, **forge** n'est plus qu'en maintenance et n'apporte rien que le navigateur n'offre nativement. Les deux exemples ci-dessous ont été exécutés avec Node 22 (même API Web Crypto que les navigateurs) et `age-encryption` 0.3.

Voir aussi : [[OAuth OpenID]], [[Identité numérique européenne]], [[Règlement Général sur la Protection des Données (RGPD)]], [[Sécurité avec Python]], [[Javascript]].

# 1. Le problème

Le processus initial — le candidat téléverse ou envoie par courriel une numérisation de sa pièce d'identité aux membres vérificateurs — a trois défauts :

- la pièce transite et se conserve **en clair** (boîtes mail, serveur, sauvegardes, journaux) ;
- le candidat découvre l'**adresse des vérificateurs**, alors qu'on ne sait pas encore qui il est : possibilité de pression sur eux ;
- l'association diffuse des données personnelles des vérificateurs sans nécessité.

Deux mesures se complètent : ne jamais mettre candidat et vérificateurs en contact direct (le portail sert d'intermédiaire, les vérificateurs sont désignés par un pseudonyme), et chiffrer la pièce **avant** qu'elle ne quitte le navigateur du candidat, pour les seuls vérificateurs désignés.

## 1.1 Modèle de menace

| Acteur | Ce qu'il peut faire | Ce que le chiffrement de bout en bout change |
|---|---|---|
| Administrateur du portail, hébergeur, sauvegardes | lire la base et les fichiers | ne voit que des octets chiffrés |
| Attaquant ayant volé la base | idem, hors ligne | idem, tant que les clés privées ne sont pas sur le serveur |
| Attaquant sur le réseau | intercepter | rien de plus que TLS ne protège déjà ; le chiffrement protège au repos et contre le serveur |
| Vérificateur malhonnête | lire la pièce qu'on lui a confiée | rien : c'est le destinataire légitime ; la parade est organisationnelle (plusieurs vérificateurs, journal, charte) |
| Attaquant injectant du JavaScript (XSS, dépendance compromise, serveur détourné) | remplacer le code de chiffrement | **tout** : c'est la limite structurelle du chiffrement dans le navigateur (chapitre 5) |

# 2. Les briques en 2026

| Brique | Rôle | Pourquoi |
|---|---|---|
| **Web Crypto API** (`crypto.subtle`) | primitives natives : AES-GCM, X25519/ECDH, HKDF, RSA-OAEP, signatures | aucune dépendance, clés **non exportables** stockables dans IndexedDB, présente dans tous les navigateurs (X25519 depuis Chrome 113, Safari 17, Firefox 130) |
| **age** (`age-encryption`) | format de fichier chiffré à **plusieurs destinataires**, implémentation TypeScript de Filippo Valsorda | interopérable avec l'outil `age` en ligne de commande, clés courtes (`age1…`), API en cinq lignes |
| **libsodium.js** (`libsodium-wrappers`) | boîtes scellées (`crypto_box_seal`), secretbox, signatures | bibliothèque de référence pour construire soi-même, difficile à mal utiliser |
| **OpenPGP.js** 6 | messages OpenPGP, clés existantes des membres | utile si les vérificateurs ont déjà des clés PGP ; format plus lourd |
| **@noble/ciphers**, **@noble/curves** | primitives auditées en JavaScript pur | quand Web Crypto ne suffit pas (courbes ou schémas absents) |

À écarter : CryptoJS, sjcl, forge, tout schéma « maison » (AES-CBC sans authentification, clés dérivées d'un mot de passe sans KDF lente, IV réutilisés).

# 3. Architecture proposée

```text
Vérificateur (une fois)                       Candidat                              Portail
------------------------                      --------                              -------
génère une paire de clés dans son            récupère les clés publiques
navigateur ; publie la clé publique  ─────►  des vérificateurs désignés  ◄────────  liste des vérificateurs (pseudonymes)
garde la clé privée (non exportable,         chiffre le fichier pour ces clés
sauvegardée sous forme de secours)           dépose le paquet chiffré  ───────────►  stocke le paquet, journalise
                                                                                      notifie les vérificateurs
ouvre le paquet avec sa clé privée  ◄────────────────────────────────────────────────  sert le paquet
rend sa décision (validé / refusé)  ──────────────────────────────────────────────►  purge le paquet une fois la décision prise
```

Principes :

1. **Chiffrement hybride** : une clé symétrique aléatoire par document (AES-256-GCM ou XChaCha20-Poly1305), elle-même chiffrée une fois par vérificateur avec sa clé publique — c'est exactement ce que fait `age` avec plusieurs destinataires.
2. **Le portail ne détient aucune clé privée** : il stocke des clés publiques, des paquets chiffrés et des décisions.
3. **Plusieurs vérificateurs par dossier** : la décision est collégiale et un vérificateur indisponible ne bloque pas.
4. **Durée de vie courte** : le paquet est supprimé à la décision ; ne subsistent que la décision, sa date et les pseudonymes des vérificateurs.

## 3.1 Variante recommandée : `age`

```javascript
import * as age from "age-encryption";

// Vérificateur, une fois : identité privée à conserver, destinataire public à publier
const identite = await age.generateIdentity();               // "AGE-SECRET-KEY-1…"
const destinataire = await age.identityToRecipient(identite); // "age1…", publié sur le portail

// Candidat : chiffrer le fichier (Uint8Array lu depuis <input type="file">) pour N vérificateurs
const chiffreur = new age.Encrypter();
for (const cle of clesPubliquesDesVerificateurs) chiffreur.addRecipient(cle);
const paquet = await chiffreur.encrypt(octetsDuFichier);      // à déposer sur le portail

// Vérificateur : ouvrir le paquet
const dechiffreur = new age.Decrypter();
dechiffreur.addIdentity(identite);
const octets = await dechiffreur.decrypt(paquet);             // Uint8Array du fichier d'origine
```

Le paquet produit commence par `age-encryption.org/v1` et se déchiffre aussi avec l'outil `age` sur un poste de travail, ce qui offre une voie de secours hors navigateur. L'identité privée (`AGE-SECRET-KEY-1…`) doit être conservée par le vérificateur dans un gestionnaire de mots de passe : c'est la seule chose à sauvegarder.

## 3.2 Variante sans dépendance : Web Crypto API

Le même schéma peut s'écrire avec les seules primitives du navigateur — X25519 pour l'échange de clés, HKDF pour dériver la clé d'enveloppe, AES-256-GCM pour le contenu et pour les enveloppes. Ce qui suit a été testé et tient en une centaine de lignes ; il vaut surtout comme illustration de ce que `age` fait pour nous.

```javascript
const subtle = crypto.subtle;
const enc = new TextEncoder();

// Vérificateur : paire X25519 longue durée, clé privée non exportable (stockable telle quelle dans IndexedDB)
async function genererPaire() {
  return subtle.generateKey({ name: "X25519" }, false, ["deriveBits"]);
}
async function exporterPublique(paire) {
  return new Uint8Array(await subtle.exportKey("raw", paire.publicKey));   // 32 octets à publier
}

async function hkdf(secretBrut) {
  const base = await subtle.importKey("raw", secretBrut, "HKDF", false, ["deriveBits"]);
  return subtle.deriveBits(
    { name: "HKDF", hash: "SHA-256", salt: new Uint8Array(0), info: enc.encode("piece-identite-v1") },
    base, 256);
}

// Candidat : une clé de contenu, puis une enveloppe par vérificateur
async function chiffrerPour(fichier, clesPubliques) {
  const cle = await subtle.generateKey({ name: "AES-GCM", length: 256 }, true, ["encrypt"]);
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const contenu = new Uint8Array(await subtle.encrypt({ name: "AES-GCM", iv }, cle, fichier));
  const cleBrute = new Uint8Array(await subtle.exportKey("raw", cle));

  const enveloppes = [];
  for (const publique of clesPubliques) {
    const ephemere = await subtle.generateKey({ name: "X25519" }, false, ["deriveBits"]);
    const destinataire = await subtle.importKey("raw", publique, { name: "X25519" }, false, []);
    const secret = await subtle.deriveBits({ name: "X25519", public: destinataire }, ephemere.privateKey, 256);
    const kek = await subtle.importKey("raw", await hkdf(secret), { name: "AES-GCM" }, false, ["encrypt"]);
    const ivEnv = crypto.getRandomValues(new Uint8Array(12));
    enveloppes.push({
      ephemere: new Uint8Array(await subtle.exportKey("raw", ephemere.publicKey)),
      iv: ivEnv,
      cleChiffree: new Uint8Array(await subtle.encrypt({ name: "AES-GCM", iv: ivEnv }, kek, cleBrute)),
    });
  }
  return { iv, contenu, enveloppes };
}

// Vérificateur : essayer chaque enveloppe avec sa clé privée
async function dechiffrer(paquet, paire) {
  for (const env of paquet.enveloppes) {
    try {
      const ephemere = await subtle.importKey("raw", env.ephemere, { name: "X25519" }, false, []);
      const secret = await subtle.deriveBits({ name: "X25519", public: ephemere }, paire.privateKey, 256);
      const kek = await subtle.importKey("raw", await hkdf(secret), { name: "AES-GCM" }, false, ["decrypt"]);
      const cleBrute = await subtle.decrypt({ name: "AES-GCM", iv: env.iv }, kek, env.cleChiffree);
      const cle = await subtle.importKey("raw", cleBrute, { name: "AES-GCM" }, false, ["decrypt"]);
      return new Uint8Array(await subtle.decrypt({ name: "AES-GCM", iv: paquet.iv }, cle, paquet.contenu));
    } catch {
      // enveloppe destinée à un autre vérificateur
    }
  }
  throw new Error("aucune enveloppe pour cette clé");
}
```

Ce schéma est un ECIES simplifié ; il manque, par rapport à `age`, un format normalisé, la protection des métadonnées et une implémentation relue par d'autres. C'est la raison de préférer `age` dès qu'on sort de la démonstration.

# 4. Gestion des clés des vérificateurs

C'est le point dur du chiffrement côté client : **une clé privée perdue rend les dossiers illisibles**.

- **Dans le navigateur** : une `CryptoKey` non exportable stockée dans IndexedDB survit aux rechargements mais pas à un changement de navigateur ni à un nettoyage des données de site ; prévoir un mécanisme de secours.
- **Secours** : identité `age` (ou clé exportée chiffrée par un mot de passe fort via Argon2/PBKDF2) rangée dans le gestionnaire de mots de passe du vérificateur ; jamais sur le portail.
- **Plusieurs appareils** : republier une clé par appareil ; le candidat chiffre pour toutes les clés actives du vérificateur.
- **Passkeys** : l'extension **PRF** de WebAuthn permet de dériver une clé de chiffrement depuis une passkey, donc de retrouver la même clé sur tout appareil synchronisé ; support encore inégal en 2026, à expérimenter.
- **Renouvellement et révocation** : une clé compromise se remplace ; les dossiers en cours sont rechiffrés par le candidat ou clôturés — d'où l'intérêt de dossiers à courte durée de vie.

# 5. Limites à connaître

- **Le code vient du serveur** : si le serveur (ou une dépendance, ou une extension) est compromis, il peut servir un JavaScript qui exfiltre le fichier avant chiffrement. Contre-mesures : politique CSP stricte, intégrité des sous-ressources, pas de CDN tiers, construction reproductible, revue des dépendances — et l'honnêteté de dire que la protection vise le serveur *stocké* et *sauvegardé*, pas un serveur activement malveillant.
- **Métadonnées** : le portail sait qui a déposé quoi, quand, pour qui et la taille du fichier ; à minimiser et à purger.
- **Le poste du vérificateur** : le document est en clair pendant la consultation ; ne pas offrir de téléchargement, afficher dans la page, purger le cache.
- **Ergonomie** : génération et sauvegarde des clés, appareils multiples, reprise après perte — c'est ce qui fait échouer la plupart des projets de chiffrement de bout en bout, pas la cryptographie.

# 6. Cadre juridique

Une numérisation de pièce d'identité est une donnée personnelle à fort risque (usurpation). Le [[Règlement Général sur la Protection des Données (RGPD)]] impose de justifier la finalité (vérifier l'éligibilité d'un candidat), de **minimiser** (une attestation d'identité suffirait-elle ?), de **limiter la conservation** (suppression dès la décision) et de sécuriser (article 32 : le chiffrement de bout en bout est une mesure pertinente, à documenter). Un traitement de pièces d'identité à grande échelle peut justifier une analyse d'impact.

# 7. L'alternative de 2026 : ne plus transmettre de pièce du tout

Le règlement eIDAS 2 impose aux États membres de proposer un **portefeuille européen d'identité numérique** (EUDI Wallet) ; en France, France Identité porte ce rôle. Le candidat pourrait alors présenter une **attestation vérifiable** (identité, âge, résidence) dont le portail contrôle la signature sans jamais recevoir de numérisation : plus de fichier sensible, plus de clés de vérificateurs à gérer. Voir [[Identité numérique européenne]]. Le chiffrement côté navigateur reste utile tant que ces portefeuilles ne sont pas généralisés et pour les documents qu'ils ne couvrent pas.

# 8. À expérimenter

1. Prototype `age` : génération de l'identité côté vérificateur, publication de la clé, chiffrement multi-destinataires et déchiffrement dans la page, sans téléchargement.
2. Parcours de secours : perte de navigateur, second appareil, révocation.
3. CSP stricte et intégrité des sous-ressources sur la page de dépôt ; audit du bundle.
4. Mesure de l'expérience utilisateur avec de vrais vérificateurs.
5. En parallèle, veille sur les attestations du portefeuille européen pour supprimer le besoin.

# 9. Sources

- Web Crypto API : <https://developer.mozilla.org/docs/Web/API/Web_Crypto_API> ; courbes sûres (X25519, Ed25519) : <https://wicg.github.io/webcrypto-secure-curves/>
- age : <https://age-encryption.org/>, bibliothèque TypeScript <https://github.com/FiloSottile/typage>
- libsodium.js : <https://github.com/jedisct1/libsodium.js>
- OpenPGP.js : <https://openpgpjs.org/>
- noble (ciphers, curves) : <https://paulmillr.com/noble/>
- WebAuthn PRF : <https://w3c.github.io/webauthn/#prf-extension>
- CNIL, chiffrement et données personnelles : <https://www.cnil.fr/fr/securite-chiffrer-garantir-lintegrite-ou-signer>
- Portefeuille européen d'identité numérique : <https://ec.europa.eu/digital-building-blocks/sites/display/EUDIGITALIDENTITYWALLET>
