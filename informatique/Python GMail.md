# Python GMail
## Donne moi le code python qui permet de se connecter à son compte messagerie gmail, puis de récupérer dans un dossier tous les emails ayant la chaine "RAHEU" en objet et ayant été envoyer le le 16 novembre 2022.

Voici un exemple de code Python qui permet de se connecter à un compte Gmail, de récupérer les e-mails qui ont "RAHEU" dans l'objet et qui ont été envoyés le 16 novembre 2022, puis de les enregistrer dans un dossier:

```python
import imaplib
import email
import os

# Connexion à Gmail
mail = imaplib.IMAP4_SSL('imap.gmail.com')
mail.login('your_email@gmail.com', 'your_password')
mail.select('inbox')

# Recherche des e-mails
date = '16-NOV-2022'
result, data = mail.search(None, '(SINCE "{}")'.format(date))

# Récupération des e-mails
for num in data[0].split():
    result, data = mail.fetch(num, '(RFC822)')
    raw_email = data[0][1]
    email_message = email.message_from_bytes(raw_email)

    # Vérification de l'objet
    if "RAHEU" in email_message['Subject']:
        # Enregistrement de l'e-mail dans un dossier
        if not os.path.exists("emails"):
            os.makedirs("emails")
        with open(os.path.join("emails", email_message['Subject']), 'w') as f:
            f.write(email_message.get_payload())

# Fermeture de la connexion
mail.close()
mail.logout()
```

Il est important de noter que cette réponse suppose que vous avez activé l'accès à l'API de compte Google pour votre compte Gmail, et que vous avez créé des identifiants pour utiliser l'API de Gmail. Sinon cela ne fonctionnera pas.

## Activer l'API google

Étapes pour activer l'API Gmail et obtenir un jeton d'authentification :

1.  Allez sur la console de développeur Google : [https://console.developers.google.com/](https://console.developers.google.com/)
2.  Créez un projet ou sélectionnez un projet existant.
3.  Accédez à la page "Bibliothèque" et recherchez "Gmail API".
4.  Cliquez sur "Activer" pour activer l'API Gmail.
5.  Accédez à la page "Identifiants" et créez des identifiants pour l'application. Assurez-vous de sélectionner "Application de bureau" pour le type d'application.
6.  Téléchargez les informations d'identification (fichier JSON) et utilisez-les pour obtenir un jeton d'accès en utilisant la bibliothèque Google Auth de Python.

Voici un exemple de code Python pour envoyer un email en utilisant le jeton d'authentification obtenu :

```python
from google.oauth2.credentials import Credentials
import base64
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from googleapiclient.discovery import build

# Chargement des informations d'identification à partir du fichier JSON
creds = Credentials.from_authorized_user_info(info=json.loads(json_creds))

# Création de l'objet service
service = build('gmail', 'v1', credentials=creds)

# Création de l'email
message = MIMEMultipart()
text = MIMEText('Contenu de l\'email')
message.attach(text)
message['to'] = 'destinataire@example.com'
message['subject'] = 'Sujet de l\'email'

# Encodage de l'email en base64
create_message = {'raw': base64.urlsafe_b64encode(message.as_bytes()).decode()}

# Envoi de l'email
send_message = (service.users().messages().send(userId="me", body=create_message).execute())

print(F'Email envoyé à l\'adresse : {message["to"]} avec le sujet : {message["subject"]}')
```
Il est important de noter que l'utilisation de cette fonctionnalité nécessite une autorisation de l'utilisateur de l'application pour l'accès aux données de l'utilisateur, il est donc important de suivre les instructions de la bibliothèque Google Auth pour obtenir les informations d'identification nécessaires.

## code pour la récupération d'une collection de messages

Voici un exemple de code Python qui utilise la bibliothèque Google API pour récupérer les messages reçus le 16 novembre 2022 qui ont une pièce jointe :

```python
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError

# Fonction pour récupérer les messages
def get_messages_with_attachment(service,date):
    try:
        # Recherche des messages reçus le 16 novembre 2022 qui ont une pièce jointe
        query = "has:attachment after:{}".format(date)
        messages = service.users().messages().list(userId='me',q=query).execute()
        if 'messages' in messages:
            return messages['messages']
        else:
            print('No messages found.')
            return None
    except HttpError as error:
        print(F'An error occurred: {error}')
        return None

# Création de l'objet service
service = build('gmail', 'v1', credentials=creds)

# Appel de la fonction pour récupérer les messages
date = '2022/11/16'
messages = get_messages_with_attachment(service, date)

# Affichage des informations sur les messages
if messages is not None:
    for message in messages:
        print(F'ID: {message["id"]}')
        print(F'Subject: {message["snippet"]}')
```
Il est important de noter que cette fonction utilise la recherche avancée de Gmail pour rechercher les messages qui ont une pièce jointe. Donc vous devez respecter la syntaxe de recherche google pour les requêtes pour que cela fonctionne.

De plus, vous devez vous assurer que les informations d'identification pour accéder à l'API Gmail sont correctement chargées avant d'appeler la fonction `build()`.

## Connexion par mot de passe
```python
import imaplib
import email
import os

# Connexion à Gmail
mail = imaplib.IMAP4_SSL('imap.gmail.com')
mail.login('your_email@gmail.com', 'your_password')
mail.select('inbox')

# Recherche des e-mails
date = '16-NOV-2022'
result, data = mail.search(None, '(SINCE "{}")'.format(date))

# Récupération des e-mails
for num in data[0].split():
    result, data = mail.fetch(num, '(RFC822)')
    raw_email = data[0][1]
    email_message = email.message_from_bytes(raw_email)

    # Vérification de l'objet
    if "RAHEU" in email_message['Subject']:
        # Enregistrement de l'e-mail dans un dossier
        if not os.path.exists("emails"):
            os.makedirs("emails")
        with open(os.path.join("emails", email_message['Subject']), 'w') as f:
            f.write(email_message.get_payload())

# Fermeture de la connexion
mail.close()
mail.logout()
```

## Connexion par token

Voici les étapes pour activer l'API Gmail et obtenir un jeton d'authentification :

1.  Allez sur la console de développeur Google : [https://console.developers.google.com/](https://console.developers.google.com/)
2.  Créez un projet ou sélectionnez un projet existant.
3.  Accédez à la page "Bibliothèque" et recherchez "Gmail API".
4.  Cliquez sur "Activer" pour activer l'API Gmail.
5.  Accédez à la page "Identifiants" et créez des identifiants pour l'application. Assurez-vous de sélectionner "Application de bureau" pour le type d'application.
6.  Téléchargez les informations d'identification (fichier JSON) et utilisez-les pour obtenir un jeton d'accès en utilisant la bibliothèque Google Auth de Python.

Voici un exemple de code Python pour envoyer un email en utilisant le jeton d'authentification obtenu :

```python
from google.oauth2.credentials
import Credentials
import base64
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from googleapiclient.discovery import build 

# Chargement des informations d'identification à partir du fichier JSON
creds = Credentials.from_authorized_user_info(info=json.loads(json_creds))

# Création de l'objet service 
service = build('gmail', 'v1', credentials=creds)

# Création de l'email 
message = MIMEMultipart()
text = MIMEText('Contenu de l\'email')
message.attach(text)
message['to'] = 'destinataire@example.com' message['subject'] = 'Sujet de l\'email'
# Encodage de l'email en base64 
create_message = {'raw': base64.urlsafe_b64encode(message.as_bytes()).decode()}  # Envoi de l'email 
send_message = (service.users().messages().send(userId="me", body=create_message).execute())
print(F'Email envoyé à l\'adresse : {message["to"]} avec le sujet : {message["subject"]}')
```

Il est important de noter que l'utilisation de cette fonctionnalité nécessite une autorisation de l'utilisateur de l'application pour l'accès aux données de l'utilisateur, il est donc important de suivre les instructions de la bibliothèque Google Auth pour obtenir les informations d'identification nécessaires.