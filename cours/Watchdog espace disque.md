Pour créer un script bash qui envoie un e-mail lorsque l'espace disque est utilisé à 70% ou plus, suivons les étapes ci-dessous:

1.  Ouvrons notre éditeur de texte préféré et créons un nouveau fichier nommé `check_disk_usage.sh`.
    
2.  Copions et collons le contenu suivant dans le fichier `check_disk_usage.sh` :
```bash
#!/bin/bash

# Paramètres
THRESHOLD=70
EMAIL="michaellaunay@ecreall.com"
HOSTNAME=$(hostname)

# Récupérer l'utilisation du disque
DISK_USAGE=$(df --output=pcent / | tail -1 | tr -dc '0-9')

# Vérifier si l'utilisation du disque est supérieure ou égale au seuil
if [ "$DISK_USAGE" -ge "$THRESHOLD" ]; then
  # Envoyer un e-mail d'alerte
  SUBJECT="[$HOSTNAME] Alerte d'espace disque: $DISK_USAGE% utilisé"
  MESSAGE="L'espace disque sur le serveur $HOSTNAME est utilisé à $DISK_USAGE%, ce qui dépasse le seuil d'alerte de $THRESHOLD%."
  echo "$MESSAGE" | mail -s "$SUBJECT" "$EMAIL"
fi

```

Rendons le script exécutable en exécutant la commande suivante :
```bash
chmod +x check_disk_usage.sh
```

Configurons le cron pour exécuter ce script. Ouvrons l'éditeur de la table de cron avec la commande suivante :
```bash
crontab -e
```

Ajoutons la ligne suivante pour exécuter le script toutes les heures (nous pouvons ajuster la fréquence en fonction de nos besoins) :
```bash
0 * * * * /chemin/vers/check_disk_usage.sh
```

N'oublions pas de remplacer `/chemin/vers/` par le chemin absolu vers le fichier `check_disk_usage.sh`.

6.  Sauvegardons et quittons l'éditeur. Le cron est maintenant configuré pour exécuter le script `check_disk_usage.sh` toutes les heures.

Avec cette configuration, un e-mail sera envoyé à `michaellaunay@ecreall.com` lorsque l'utilisation du disque dépasse 70%