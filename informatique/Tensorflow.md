#Expérimentation #Informatique #IA 
TensorFlow est une bibliothèque open-source d'apprentissage automatique développée par Google. Elle permet de construire des modèles d'apprentissage profond (deep learning) pour résoudre des problèmes complexes, tels que la reconnaissance d'images, la classification de textes, la prédiction de séries temporelles, la détection d'objets et bien d'autres.

TensorFlow fournit une interface de programmation flexible pour la création de modèles d'apprentissage automatique, avec une grande variété de modules pour la construction de réseaux de neurones, la gestion des données, la visualisation de données et bien plus encore. Elle utilise des graphes de calcul statiques pour optimiser les calculs et permet une accélération matérielle à travers des unités de traitement graphique (GPU) ou des unités de traitement tensorielles (TPU).

TensorFlow est devenu l'une des bibliothèques les plus populaires pour l'apprentissage automatique en raison de sa flexibilité, de sa performance et de son écosystème en constante évolution. De nombreuses entreprises et organisations l'utilisent pour la recherche et la production d'applications d'apprentissage automatique.

# Apprentisage du modèle
On se place dans le même cas que pythorch (série d'images brutes et série d'images corrigées) [[Pytorch]].
On utilise TensorFlow pour entraîner un modèle similaire pour améliorer des images et l'utiliser pour améliorer de nouvelles images.
Exemple de code qui utilise TensorFlow 2 pour accomplir cela :
```python
import os
import numpy as np
from PIL import Image
import tensorflow as tf
from tensorflow.keras import layers, models

# Créer le modèle
model = models.Sequential([
    layers.InputLayer(input_shape=(224, 224, 3)),
    layers.experimental.preprocessing.Rescaling(1./255),
    layers.Conv2D(32, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),
    layers.Conv2D(64, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),
    layers.Conv2D(128, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),
    layers.Flatten(),
    layers.Dense(128, activation='relu'),
    layers.Dense(3, activation='sigmoid')
])

# Compiler le modèle
model.compile(optimizer='adam', loss='mse')

# Charger les données d'entraînement et de validation
train_images = []
train_labels = []
val_images = []
val_labels = []
for i in range(1, 81):
    # Charger l'image originale
    image_path = f"img{i:03}.jpg"
    image = Image.open(image_path).convert("RGB")
    image = np.array(image.resize((224, 224)))  # Redimensionner l'image à la taille d'entrée du modèle
    train_images.append(image)
    
    # Charger l'image corrigée
    image_path = f"img_corr_{i:03}.jpg"
    image = Image.open(image_path).convert("RGB")
    image = np.array(image.resize((224, 224)))  # Redimensionner l'image à la taille d'entrée du modèle
    train_labels.append(image)
for i in range(81, 101):
    # Charger l'image originale
    image_path = f"img{i:03}.jpg"
    image = Image.open(image_path).convert("RGB")
    image = np.array(image.resize((224, 224)))  # Redimensionner l'image à la taille d'entrée du modèle
    val_images.append(image)
    
    # Charger l'image corrigée
    image_path = f"img_corr_{i:03}.jpg"
    image = Image.open(image_path).convert("RGB")
    image = np.array(image.resize((224, 224)))  # Redimensionner l'image à la taille d'entrée du modèle
    val_labels.append(image)
train_images = np.array(train_images)
train_labels = np.array(train_labels)
val_images = np.array(val_images)
val_labels = np.array(val_labels)

# Entraîner le modèle
model.fit(train_images, train_labels, validation_data=(val_images, val_labels), epochs=10)

# Utiliser le modèle pour améliorer de nouvelles images
for i in range(101, 121):
    # Charger l'image
    image_path = f"img{i:03}.jpg"
    image = Image.open(image_path).convert("RGB")
    image = np.array(image.resize((224, 224)))  # Redimensionner l'image à la taille d'entrée du modèle
    
    # Utiliser le modèle pour améliorer l'image
    improved_image = model.predict(np.expand_dims(image, 0))[0]

    improved_image = Image.fromarray(np.uint8(improved_image * 255))  # Convertir l'image améliorée en format Image
    improved_image.save(f"improved_img_{i:03}.jpg")  # Enregistrer l'image améliorée

```
Ce code suit un processus similaire au code PyTorch précédent :

-   Créer le modèle avec les couches appropriées.
-   Compiler le modèle avec une fonction de perte appropriée.
-   Charger les données d'entraînement et de validation sous forme de tableau NumPy.
-   Entraîner le modèle avec les données d'entraînement.
-   Charger de nouvelles images et utiliser le modèle pour améliorer chacune d'elles.
-   Enregistrer chaque image améliorée dans un fichier.

La principale différence est la syntaxe utilisée pour créer et entraîner le modèle, ainsi que pour charger les données. Cependant, le processus est similaire et le résultat est le même : un modèle qui peut améliorer des images et qui peut être utilisé pour améliorer de nouvelles images.

# Utilisation
Exemple de code qui utilise le modèle TensorFlow pour améliorer une collection d'images :
```python
import numpy as np
import tensorflow as tf
from PIL import Image

# Charger les modèles
generator = tf.keras.models.load_model("generator.h5")

# Parcourir toutes les images à améliorer
for i in range(1, 101):
    # Charger l'image à améliorer
    image = Image.open(f"img{i:03}.jpg")
    image = np.array(image) / 255.0  # Convertir l'image en tableau NumPy et normaliser les valeurs de pixel

    # Améliorer l'image
    improved_image = generator.predict(np.expand_dims(image, axis=0))[0]

    # Convertir l'image améliorée en format Image et l'enregistrer
    improved_image = Image.fromarray(np.uint8(improved_image * 255))
    improved_image.save(f"improved_img_{i:03}.jpg")

```