Je scanne de nombreux vieux livres au format jpg puis les concatenne dans des pdf.
Les images doivent être retravaillées pour recouper et recentrer les texte des pages.
Je souhaite utiliser le ML en python .
Voici un exemple de code en Python utilisant le module OpenCV pour recouper et recentrer les pages d'un livre scanné au format PDF :
```python
import cv2
import numpy as np
from pdf2image import convert_from_path

# Convertir la première page du PDF en image
pages = convert_from_path('chemin/vers/votre/fichier.pdf', 500)
img = np.array(pages[0])

# Convertir l'image en niveaux de gris
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Appliquer un flou gaussien pour réduire le bruit
blurred = cv2.GaussianBlur(gray, (5, 5), 0)

# Appliquer un seuillage adaptatif pour binariser l'image
thresh = cv2.adaptiveThreshold(blurred, 255, cv2.ADAPTIVE_THRESH_MEAN_C, cv2.THRESH_BINARY_INV, 11, 4)

# Rechercher les contours des objets dans l'image
contours, hierarchy = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

# Trouver le plus grand contour (le contour de la page)
largest_contour = max(contours, key=cv2.contourArea)

# Trouver les coins de la page
rect = cv2.minAreaRect(largest_contour)
box = cv2.boxPoints(rect)
box = np.int0(box)

# Calculer la largeur et la hauteur de la page
width = int(rect[1][0])
height = int(rect[1][1])

# Définir les coins de la page cible
target_corners = np.array([[0, 0], [width - 1, 0], [width - 1, height - 1], [0, height - 1]], dtype=np.float32)

# Trouver la matrice de transformation perspective
M = cv2.getPerspectiveTransform(box.astype(np.float32), target_corners)

# Appliquer la transformation perspective pour recadrer la page
warped = cv2.warpPerspective(img, M, (width, height))

# Enregistrer l'image recadrée
cv2.imwrite('chemin/vers/votre/image_recadree.jpg', warped)
```

Ce code lit le premier page d'un fichier PDF, convertit l'image en niveaux de gris, applique un flou gaussien pour réduire le bruit, applique un seuillage adaptatif pour binariser l'image, trouve les contours des objets dans l'image, trouve le plus grand contour (le contour de la page), trouve les coins de la page, calcule la largeur et la hauteur de la page, définit les coins de la page cible, trouve la matrice de transformation perspective et applique la transformation perspective pour recadrer la page. Enfin, l'image recadrée est enregistrée dans un fichier.

Je souhaite reconnaitre les lignes de textes d'une image au format jpg et pour chaque ligne de texte sa position dans l'image et la valeur de la chaine reconnue. Un ligne est une suite de mots n'ayant pas plus de 8 espaces entre deux mots. Pour reconnaître les lignes de texte d'une image au format jpg, il faut utiliser une bibliothèque de traitement d'image comme OpenCV et une bibliothèque de reconnaissance de texte comme Pytesseract.

Voici le code Python pour reconnaître les lignes de texte d'une image au format jpg, pour chaque ligne de texte sa position dans l'image et la valeur de la chaîne reconnue :

```python
import cv2
import pytesseract

# Charger l'image
image = cv2.imread("image.jpg")

# Convertir l'image en noir et blanc
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

# Appliquer un flou gaussien pour réduire le bruit de l'image
blur = cv2.GaussianBlur(gray, (5, 5), 0)

# Appliquer une détection de contours pour trouver les contours dans l'image
edged = cv2.Canny(blur, 30, 150)

# Trouver les contours dans l'image
contours, _ = cv2.findContours(edged.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

# Initialiser la liste des lignes de texte
lines = []

# Pour chaque contour, trouver la position et la valeur de la chaîne reconnue
for contour in contours:
    # Calculer les coordonnées du contour
    x, y, w, h = cv2.boundingRect(contour)
    
    # Ignorer les contours qui sont trop petits
    if w < 50 or h < 10:
        continue
        
    # Extraire la région d'intérêt de l'image
    roi = gray[y:y + h, x:x + w]
    
    # Appliquer la reconnaissance optique de caractères (OCR) pour reconnaître le texte
    text = pytesseract.image_to_string(roi, lang='fra', config='--psm 6')
    
    # Ajouter la ligne de texte et sa position à la liste
    lines.append((text, (x, y, x + w, y + h)))

# Trier les lignes de texte par ordre de position verticale
lines = sorted(lines, key=lambda x: x[1][1])

# Définir le nombre maximum d'espaces autorisés entre les mots
max_spaces = 8

# Regrouper les lignes de texte en lignes de texte contiguës
grouped_lines = []
for line in lines:
    if not grouped_lines:
        # Ajouter la première ligne
       #@TODO

```

Pytesseract peut retourner les coordonnées des boîtes englobantes des caractères et des mots dans l'image. Pour ce faire, vous devez utiliser le paramètre `output_type` et lui attribuer la valeur `Output.DICT` pour que Pytesseract renvoie un dictionnaire contenant des informations détaillées sur la reconnaissance optique de caractères.

Voici un exemple de code qui utilise Pytesseract pour extraire les lignes de texte et leurs positions, ainsi que les coordonnées des boîtes englobantes de chaque mot de chaque ligne :
```python
import pytesseract
from PIL import Image

# Ouvrir l'image avec PIL
image = Image.open("image.jpg")

# Utiliser Pytesseract pour reconnaître le texte dans l'image
data = pytesseract.image_to_data(image, output_type=pytesseract.Output.DICT)

# Itérer sur chaque ligne pour trouver la position et la valeur
for i, text in enumerate(data["text"]):
    # Supprimer les espaces de début et de fin de texte
    text = text.strip()
    # Vérifier si le texte n'est pas vide
    if len(text) > 0:
        # Vérifier si le texte correspond à une ligne valide
        if len(text.split()) > 1 and all(len(word) <= 8 for word in text.split()):
            # Récupérer les coordonnées de la boîte englobante de la ligne
            left = data["left"][i]
            top = data["top"][i]
            width = data["width"][i]
            height = data["height"][i]
            # Imprimer la position et la valeur de la ligne
            print(f"Ligne {i + 1}: {text}")
            print(f"Position : ({left}, {top})")
            print(f"Dimensions : {width} x {height}")
            # Itérer sur chaque mot dans la ligne pour trouver ses coordonnées de boîte englobante
            for j, word in enumerate(text.split()):
                # Récupérer les coordonnées de la boîte englobante du mot
                word_left = data["left"][i] + data["left"][data["word_num"][i]+j]
                word_top = data["top"][i] + data["top"][data["word_num"][i]+j]
                word_width = data["width"][data["word_num"][i]+j]
                word_height = data["height"][data["word_num"][i]+j]
                # Imprimer la position et la valeur du mot
                print(f"Mot {j + 1}: {word}")
                print(f"Position : ({word_left}, {word_top})")
                print(f"Dimensions : {word_width} x {word_height}")

```
Ce code utilise la fonction `pytesseract.image_to_data()` pour récupérer les informations détaillées de la reconnaissance optique de caractères. Les informations sont stockées dans un dictionnaire Python avec des clés telles que `text`, `left`, `top`, `width`, `height` et `word_num`. Les coordonnées des boîtes englobantes des mots sont calculées en ajoutant les coordonnées du mot à celles de la ligne. Notez que les coordonnées sont exprimées en pixels, en partant du coin supérieur gauche de l'image.