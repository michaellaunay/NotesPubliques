LibGen (Library Genesis) est une bibliothèque en ligne qui fournit un accès gratuit à des millions de livres, articles scientifiques et documents académiques. Elle permet aux utilisateurs de télécharger des livres électroniques, des articles scientifiques, des thèses, des documents de recherche, des manuels et des magazines gratuitement.

Fondée en 2008, LibGen est devenue une plateforme populaire pour les étudiants, les universitaires, les chercheurs et les amateurs de livres du monde entier. La bibliothèque en ligne utilise des sources ouvertes et piratées pour fournir des documents et des livres gratuits à ses utilisateurs. Bien que cette pratique soit controversée et illégale dans de nombreux pays, LibGen a été maintes fois hébergée sur différents domaines et continue de fonctionner.
En France seuls les documents tombés dans le domaines public ont le droit d'être téléchargée sans contrepartie données à l'auteur, généralement la maison d'édition rétribue l'auteur lors de l'achat des livres, mais avec la numérisation la copie des livres illégales des livres .
Nous allons ici créer un scipt qui va donner le pourcentage d'oeuvres fançaises illégales dans libgen.
Pour cela nous devons :
- récupérer le catalogue français libgen
- tester chaque description d'oeuvre dans le site de la bnf pour savoir si l'oeuvre est dans le domaines.
Pour vérifier qu'une oeuvre est dans le domaine public https://data.bnf.fr/
On pêut intérroger le site de la bnf via la syntax sparql
https://api.bnf.fr/fr/sparql-endpoint-de-databnffr
Exemple python pour savoir si un isbn est dans le domaine public
```python
import requests
import json

isbn = "9782070457665"  # Remplacez cet exemple d'ISBN par le vôtre

# Interroge l'API data.bnf.fr avec l'ISBN fourni
response = requests.get(f"https://data.bnf.fr/api/sparql?default-graph-uri=&query=SELECT+%3FisPublicDomain+WHERE+%7B+%3Fwork+<http%3A%2F%2Frdvocab.info%2FRDA%2FworkManifestedAsISBN+%3E+%22{isbn}%22+.%0D%0A++++%3Fwork+<http%3A%2F%2Frdvocab.info%2FRDA%2FworkIsInDomainOfManifestation+%3E+%3Fdomain+.%0D%0A++++%3Fdomain+<http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fissued>%3Fdate+.%0D%0A++++BIND%28%28%28year%28now%28%29%29+-+year%28%3Fdate%29%29+%3E+70%29+AS+%3FisPublicDomain%29.+%7D&format=json")

# Vérifie si la réponse est valide
if response.status_code != 200:
    print("La requête a échoué. Veuillez réessayer plus tard.")
    exit()

# Convertit la réponse JSON en objet Python
data = json.loads(response.text)

# Analyse le résultat de la requête pour déterminer si l'œuvre est dans le domaine public
if len(data["results"]["bindings"]) > 0:
    is_public_domain = data["results"]["bindings"][0]["isPublicDomain"]["value"]
    if is_public_domain == "true":
        print("L'œuvre associée à l'ISBN donné est dans le domaine public.")
    else:
        print("L'œuvre associée à l'ISBN donné n'est pas dans le domaine public.")
else:
    print("Aucune œuvre trouvée pour cet ISBN.")

```
Pour réaliser des statistiques sur le nombre d'oeuvres piratées par libgen, il faut récupérer son catalogue puis passer chaque oeuvre à la base de données data.bnf.fr.
Explication de l'API de libgen http://faq.fyicenter.com/1231_What_Is_Library_Genesis_API.html
Exemple de code https://gist.github.com/hixann/af466e96b1988b67fdde5b2afdb735cf

```python
import requests
import json

# URL de l'API de LibGen pour récupérer la liste des livres
url = "http://libgen.rs/json.php?fields=Topic,Title,Author,Year,Language&q=language%3Afr"

# Récupérer la réponse de l'API
response = requests.get(url)

# Convertir la réponse en objet JSON
json_data = json.loads(response.content)

# Initialiser un dictionnaire pour stocker les livres par thème
books_by_topic = {}

# Parcourir les livres dans la réponse de l'API
for book in json_data:
    # Récupérer le thème du livre
    topic = book.get("Topic")

    # Ajouter le livre à la liste des livres pour ce thème
    if topic in books_by_topic:
        books_by_topic[topic].append(book)
    else:
        books_by_topic[topic] = [book]

# Afficher le nombre de livres par thème
for topic, books in books_by_topic.items():
    print(f"{topic}: {len(books)} livres")
```