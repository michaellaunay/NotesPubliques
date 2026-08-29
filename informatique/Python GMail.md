---
schema_version: 1
uid: "01M02JG1VFX5DPF0A3NBETC9YN"
titre: "Python GMail"
aliases:
  - "Gmail en Python"
type: procedure
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - python
  - messagerie
  - imap
  - oauth
  - automatisation
resume: "Procédure pour lire, archiver et envoyer des courriels Gmail depuis Python en 2026 : mot de passe d'application avec IMAP/SMTP ou OAuth 2.0 avec l'API Gmail, recherche par objet et par date, sauvegarde des messages et des pièces jointes, gestion des secrets et pièges courants."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-01-27
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Python GMail

> [!abstract] Objectif
> Écrire un script Python qui se connecte à un compte Gmail pour retrouver des messages (par objet, par date, avec pièce jointe), les archiver sur disque et en envoyer, en utilisant l'une des deux voies encore acceptées par Google : un **mot de passe d'application** avec IMAP/SMTP, ou **OAuth 2.0** avec l'API Gmail — sans jamais écrire un secret dans le code.

> [!info] État de la note
> Vérifiée le 29 août 2026 avec Python 3.12+. Elle remplace les exemples de 2023, qui appelaient `imaplib.login()` avec le mot de passe du compte : Google refuse ce mode depuis le 30 mai 2022 pour les comptes personnels et le 30 septembre 2024 pour Google Workspace (fin des « applications moins sécurisées »).

Voir aussi : [[Python]], [[OAuth OpenID]], [[Sécurité avec Python]], [[Postfix]].

# Sommaire

1. Choisir sa voie d'accès
2. Voie 1 — Mot de passe d'application, IMAP et SMTP
3. Voie 2 — OAuth 2.0 et l'API Gmail
4. Gérer les secrets
5. Pièges courants
6. Aide-mémoire
7. Sources

# 1. Choisir sa voie d'accès

| | Mot de passe d'application + IMAP/SMTP | OAuth 2.0 + API Gmail |
|---|---|---|
| Pour qui | un script personnel sur son propre compte | une application distribuée, plusieurs comptes, un compte Workspace |
| Prérequis | validation en deux étapes activée | projet Google Cloud, client OAuth, écran de consentement |
| Bibliothèques | `imaplib`, `smtplib`, `email` (bibliothèque standard) | `google-api-python-client`, `google-auth-oauthlib` |
| Portée des droits | tout le compte mail | limitée aux *scopes* demandés (`gmail.readonly`, `gmail.send`…) |
| Révocation | supprimer le mot de passe d'application | révoquer le jeton ou l'accès de l'application |
| Limites | interdit ou désactivé par certains administrateurs Workspace | jetons de test valables 7 jours tant que l'application n'est pas publiée |

Règle simple : **un script pour soi → mot de passe d'application ; un outil pour d'autres → OAuth**. Dans les deux cas, le secret vit hors du code (chapitre 4).

# 2. Voie 1 — Mot de passe d'application, IMAP et SMTP

## 2.1 Obtenir un mot de passe d'application

1. Activer la **validation en deux étapes** sur le compte (<https://myaccount.google.com/security>) ; sans elle, l'option n'apparaît pas.
2. Ouvrir <https://myaccount.google.com/apppasswords>, nommer l'application (par exemple `script-archivage`) et copier le mot de passe de 16 caractères affiché une seule fois.
3. Vérifier que l'IMAP est activé dans les paramètres Gmail (*Voir tous les paramètres* → *Transfert et POP/IMAP*).

Ce mot de passe donne un accès complet à la messagerie : le traiter comme le mot de passe du compte, un par script, et le supprimer quand le script n'est plus utilisé.

## 2.2 Rechercher et archiver des messages

Le script ci-dessous reprend le besoin d'origine : retrouver les messages dont l'objet contient une chaîne, reçus depuis une date, et les archiver avec leurs pièces jointes.

```python
"""Archive les messages Gmail dont l'objet contient un motif, avec leurs pièces jointes.

Variables d'environnement : GMAIL_USER, GMAIL_APP_PASSWORD.
"""
from __future__ import annotations

import email
import imaplib
import os
import re
from datetime import date
from email.message import EmailMessage
from email.policy import default
from pathlib import Path

IMAP_HOST = "imap.gmail.com"
DOSSIER_SORTIE = Path("emails")


def nom_sur(texte: str, defaut: str = "sans-titre") -> str:
    """Rend un texte utilisable comme nom de fichier."""
    texte = re.sub(r"[^\w.\- ]+", "_", texte, flags=re.UNICODE).strip(" .")
    return texte[:120] or defaut


def archiver(message: EmailMessage, uid: str, dossier: Path) -> Path:
    """Écrit le message en .eml et ses pièces jointes dans un sous-dossier."""
    objet = message.get("Subject", "") or ""
    cible = dossier / f"{uid}_{nom_sur(objet)}"
    cible.mkdir(parents=True, exist_ok=True)
    (cible / "message.eml").write_bytes(message.as_bytes())
    for piece in message.iter_attachments():
        nom = nom_sur(piece.get_filename() or "piece-jointe")
        (cible / nom).write_bytes(piece.get_payload(decode=True))
    return cible


def rechercher(motif: str, depuis: date) -> None:
    utilisateur = os.environ["GMAIL_USER"]
    mot_de_passe = os.environ["GMAIL_APP_PASSWORD"]

    with imaplib.IMAP4_SSL(IMAP_HOST) as boite:
        boite.login(utilisateur, mot_de_passe)
        boite.select("INBOX", readonly=True)          # readonly : ne marque rien comme lu

        # Syntaxe IMAP standard : SUBJECT est une recherche de sous-chaîne,
        # SINCE attend un jour au format 16-Nov-2022 (mois en anglais).
        critere = f'(SUBJECT "{motif}" SINCE "{depuis.strftime("%d-%b-%Y")}")'
        statut, donnees = boite.uid("search", None, critere)
        if statut != "OK":
            raise RuntimeError(f"recherche impossible : {donnees}")

        uids = donnees[0].split()
        print(f"{len(uids)} message(s) trouvé(s)")
        for uid in uids:
            statut, reponse = boite.uid("fetch", uid, "(BODY.PEEK[])")
            brut = reponse[0][1]
            message = email.message_from_bytes(brut, policy=default)
            chemin = archiver(message, uid.decode(), DOSSIER_SORTIE)
            print(f"  {message['Date']}  {message['Subject']!r}  -> {chemin}")


if __name__ == "__main__":
    rechercher("RAHEU", date(2022, 11, 16))
```

Points importants du script :

- `policy=default` donne des objets `EmailMessage` modernes : les en-têtes sont déjà décodés (accents, encodage MIME) et `iter_attachments()` évite de parcourir les parties MIME à la main ;
- `boite.uid(...)` travaille avec des identifiants stables, contrairement aux numéros de séquence renvoyés par `search()` qui changent dès qu'un message est supprimé ;
- `BODY.PEEK[]` récupère le message sans le marquer comme lu ; `readonly=True` protège contre toute modification ;
- les noms de fichiers sont nettoyés (`nom_sur`) : un objet de courriel peut contenir `/`, `:` ou des caractères interdits ;
- Gmail accepte aussi sa propre syntaxe de recherche par l'extension `X-GM-RAW` : `boite.uid("search", None, "X-GM-RAW", '"subject:RAHEU after:2022/11/16 has:attachment"')`.

Exécution :

```bash
python -m venv .venv && . .venv/bin/activate
export GMAIL_USER="prenom.nom@gmail.com"
read -r -s -p "Mot de passe d'application : " GMAIL_APP_PASSWORD && export GMAIL_APP_PASSWORD
python archive_gmail.py
```

## 2.3 Envoyer un message avec SMTP

```python
"""Envoie un courriel par SMTP Gmail avec un mot de passe d'application."""
import os
import smtplib
from email.message import EmailMessage
from pathlib import Path


def envoyer(destinataire: str, objet: str, corps: str, piece_jointe: Path | None = None) -> None:
    message = EmailMessage()
    message["From"] = os.environ["GMAIL_USER"]
    message["To"] = destinataire
    message["Subject"] = objet
    message.set_content(corps)
    if piece_jointe is not None:
        message.add_attachment(
            piece_jointe.read_bytes(),
            maintype="application",
            subtype="octet-stream",
            filename=piece_jointe.name,
        )
    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as smtp:
        smtp.login(os.environ["GMAIL_USER"], os.environ["GMAIL_APP_PASSWORD"])
        smtp.send_message(message)


if __name__ == "__main__":
    envoyer("destinataire@example.org", "Rapport RAHEU", "Bonjour,\n\nci-joint le rapport.\n")
```

`EmailMessage` s'occupe de l'encodage des en-têtes et des pièces jointes ; l'ancien assemblage `MIMEMultipart` + `base64` n'est plus nécessaire.

# 3. Voie 2 — OAuth 2.0 et l'API Gmail

## 3.1 Créer le client OAuth

1. Dans la console Google Cloud (<https://console.cloud.google.com/>), créer ou choisir un projet, puis activer l'**API Gmail** dans la bibliothèque d'API.
2. Dans *Google Auth Platform* (anciennement « Identifiants »), configurer l'**écran de consentement** (type *Externe* pour un compte personnel), déclarer l'adresse de l'utilisateur comme **testeur**, puis créer un **ID client OAuth** de type *Application de bureau*.
3. Télécharger le fichier JSON du client, le renommer `client_secret.json` et le garder hors du dépôt Git.

Tant que l'application reste en statut *Test*, seuls les testeurs déclarés peuvent l'autoriser et le jeton de rafraîchissement expire après sept jours ; publier l'application (et, pour les scopes sensibles Gmail, passer la vérification Google) lève ces limites.

## 3.2 Obtenir et conserver un jeton

```bash
python -m pip install google-api-python-client google-auth-oauthlib
```

```python
"""Obtient (ou rafraîchit) un jeton OAuth 2.0 pour l'API Gmail."""
from pathlib import Path

from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow

SCOPES = [
    "https://www.googleapis.com/auth/gmail.readonly",   # lecture
    "https://www.googleapis.com/auth/gmail.send",       # envoi
]
CLIENT_SECRET = Path("client_secret.json")
TOKEN = Path("token.json")


def obtenir_credentials() -> Credentials:
    creds = Credentials.from_authorized_user_file(TOKEN, SCOPES) if TOKEN.exists() else None
    if creds and creds.valid:
        return creds
    if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
    else:
        flux = InstalledAppFlow.from_client_secrets_file(CLIENT_SECRET, SCOPES)
        creds = flux.run_local_server(port=0)          # ouvre le navigateur une fois
    TOKEN.write_text(creds.to_json())
    return creds
```

La première exécution ouvre le navigateur ; ensuite `token.json` est rafraîchi silencieusement. Ne demander que les scopes nécessaires : `gmail.readonly` suffit pour archiver, `gmail.send` pour envoyer, `gmail.modify` seulement pour étiqueter ou supprimer.

## 3.3 Rechercher et archiver avec l'API

```python
"""Archive via l'API Gmail les messages correspondant à une requête de recherche Gmail."""
import base64
import email
from email.policy import default
from pathlib import Path

from googleapiclient.discovery import build
from googleapiclient.errors import HttpError

from oauth_gmail import obtenir_credentials      # fonction de la section 3.2
from archive_gmail import archiver               # fonction de la section 2.2


def archiver_par_requete(requete: str, dossier: Path = Path("emails")) -> None:
    service = build("gmail", "v1", credentials=obtenir_credentials())
    try:
        page = None
        while True:
            reponse = service.users().messages().list(
                userId="me", q=requete, maxResults=100, pageToken=page
            ).execute()
            for ref in reponse.get("messages", []):
                brut = service.users().messages().get(
                    userId="me", id=ref["id"], format="raw"
                ).execute()["raw"]
                message = email.message_from_bytes(
                    base64.urlsafe_b64decode(brut), policy=default
                )
                print(archiver(message, ref["id"], dossier))
            page = reponse.get("nextPageToken")
            if not page:
                break
    except HttpError as erreur:
        print(f"Erreur API : {erreur}")


if __name__ == "__main__":
    archiver_par_requete("subject:RAHEU after:2022/11/16 has:attachment")
```

La requête `q` utilise exactement la syntaxe de la barre de recherche Gmail (`from:`, `subject:`, `after:`, `before:`, `has:attachment`, `label:`, `filename:`). Le format `raw` renvoie le message MIME complet en base64url, que le module `email` lit comme dans la voie 1 — la fonction `archiver` est donc partagée.

## 3.4 Envoyer avec l'API

```python
"""Envoie un courriel avec l'API Gmail."""
import base64
from email.message import EmailMessage

from googleapiclient.discovery import build

from oauth_gmail import obtenir_credentials


def envoyer_api(destinataire: str, objet: str, corps: str) -> str:
    message = EmailMessage()
    message["To"] = destinataire
    message["Subject"] = objet
    message.set_content(corps)
    service = build("gmail", "v1", credentials=obtenir_credentials())
    reponse = service.users().messages().send(
        userId="me", body={"raw": base64.urlsafe_b64encode(message.as_bytes()).decode()}
    ).execute()
    return reponse["id"]
```

L'en-tête `From` est renseigné par Gmail avec l'adresse du compte autorisé.

# 4. Gérer les secrets

- **Jamais dans le code ni dans Git** : variables d'environnement, fichier `.env` lu par `python-dotenv` et listé dans `.gitignore`, ou trousseau du système avec `keyring` (`keyring.set_password("gmail", utilisateur, mot_de_passe)`).
- `client_secret.json` et `token.json` sont des secrets au même titre : droits `600`, hors dépôt, un jeu par machine.
- Un secret exposé se **révoque** (page des mots de passe d'application, ou *Sécurité* → *Connexions tierces* pour un jeton OAuth) avant toute autre action — voir le retour d'expérience [[Crash github et publication de clé]].
- Sur Workspace, l'administrateur peut interdire les mots de passe d'application et restreindre les applications OAuth non vérifiées : se renseigner avant de coder.

# 5. Pièges courants

| Symptôme | Cause | Remède |
|---|---|---|
| `[AUTHENTICATIONFAILED] Invalid credentials` | mot de passe du compte utilisé, ou 2FA non activée | mot de passe d'application (section 2.1) ou OAuth |
| `SEARCH` ne renvoie rien | date au mauvais format (`16-11-2022`) ou mois localisé | `%d-%b-%Y` en anglais : `16-Nov-2022` ; forcer `locale` C |
| Dossier introuvable | noms Gmail localisés | `boite.list()` pour voir les noms réels (`[Gmail]/Tous les messages`, `[Gmail]/All Mail`) |
| Noms de fichiers illisibles | objet non décodé ou caractères interdits | `policy=default` et fonction `nom_sur` |
| Messages marqués comme lus par le script | `FETCH BODY[]` sans `PEEK` | `BODY.PEEK[]` et `select(..., readonly=True)` |
| Le jeton OAuth expire au bout d'une semaine | application en statut *Test* | publier l'application ou accepter la reconnexion hebdomadaire |
| `insufficientPermissions` | scope manquant | ajouter le scope, supprimer `token.json`, ré-autoriser |
| Erreurs 429 | quota de l'API dépassé | pagination, `maxResults`, attente exponentielle |

# 6. Aide-mémoire

| Besoin | Voie 1 (IMAP/SMTP) | Voie 2 (API Gmail) |
|---|---|---|
| Authentification | mot de passe d'application | `InstalledAppFlow` + `token.json` |
| Connexion | `imaplib.IMAP4_SSL("imap.gmail.com")` | `build("gmail", "v1", credentials=creds)` |
| Recherche | `boite.uid("search", None, '(SUBJECT "x" SINCE "16-Nov-2022")')` | `messages().list(userId="me", q="subject:x after:2022/11/16")` |
| Lecture sans marquer lu | `boite.uid("fetch", uid, "(BODY.PEEK[])")` | `messages().get(id=..., format="raw")` |
| Analyse MIME | `email.message_from_bytes(brut, policy=default)` | idem après `base64.urlsafe_b64decode` |
| Pièces jointes | `message.iter_attachments()` | idem |
| Envoi | `smtplib.SMTP_SSL("smtp.gmail.com", 465)` | `messages().send(body={"raw": ...})` |

# 7. Sources

- Mots de passe d'application : <https://support.google.com/accounts/answer/185833>
- Fin des « applications moins sécurisées » (Workspace, 30 septembre 2024) : <https://support.google.com/a/answer/6260879>
- Extensions IMAP de Gmail (`X-GM-RAW`) : <https://developers.google.com/workspace/gmail/imap/imap-extensions>
- API Gmail, guide Python : <https://developers.google.com/workspace/gmail/api/quickstart/python>
- Scopes de l'API Gmail : <https://developers.google.com/workspace/gmail/api/auth/scopes>
- Bibliothèque standard Python : <https://docs.python.org/3/library/imaplib.html>, <https://docs.python.org/3/library/email.html>
