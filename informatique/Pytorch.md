#Expérimentation #Informatique #IA 
# Définition
PyTorch est une bibliothèque open-source pour le traitement des données et le développement de modèles d'apprentissage automatique en Python. Elle est largement utilisée dans le domaine de l'apprentissage profond.

PyTorch a été développé par Facebook en 2016 et est basé sur la bibliothèque Torch, qui est utilisée dans l'industrie depuis des années pour la recherche et la production d'applications d'apprentissage automatique.

L'une des principales caractéristiques de PyTorch est sa capacité à créer des graphes de calcul dynamiques. Cela signifie que les graphes sont créés à mesure que le code est exécuté, ce qui permet aux utilisateurs de modifier le graphique en temps réel. Cette fonctionnalité est utile pour le développement et le débogage de modèles, ainsi que pour les modèles qui ont des architectures variables.

En outre, PyTorch fournit également une prise en charge intégrée pour l'accélération matérielle, tel que les GPU (unités de traitement graphique), qui permettent une accélération significative du temps de formation des modèles.

PyTorch fournit également une grande variété de modules pour la construction de réseaux de neurones, la gestion des données, le calcul de gradients, la visualisation de données et bien plus encore. PyTorch est très flexible et facile à utiliser, ce qui en fait une bibliothèque populaire pour les chercheurs et les développeurs d'apprentissage automatique.

# Utilisation
J'ai numérisé les pages d'un livre, et j'ai ainsi obtenu une centaine d'images jpg appelées imgXXX.jpg où XXX est le numéro de l'image, j'ai corrigé ces images en découpant les bords, recadrant le texte et en supprimant certains défaut.
J'ai ainsi obtenu une centaine d'autres images appelées img_corr_XXX.jpg (XX est le numéro de l'image).
Nous allons faire apprendre ce travail de correction et d'amélioration à un modèle pytorch pour pouvoir automatiser ce travail dans un script python.
# Démarche
1.  Collecte et préparation des données : Divisez vos images corrigées en deux ensembles, l'un pour l'entraînement et l'autre pour la validation. Les images doivent être stockées dans des dossiers séparés. Ensuite, créez un fichier CSV avec les noms des fichiers de chaque ensemble et les annotations (c'est-à-dire les images originales correspondantes).
    
2.  Créez un jeu de données personnalisé : Utilisez la classe de jeu de données personnalisée de PyTorch pour charger vos images, appliquer les transformations de prétraitement et retourner un échantillon de données.
    
3.  Créez un modèle : Vous pouvez utiliser une architecture de réseau de neurones convolutifs (CNN) pour apprendre à améliorer vos images. Vous pouvez également utiliser des modèles pré-entraînés tels que ResNet ou VGG, ou entraîner un modèle de réseau de neurones à partir de zéro.
    
4.  Définissez une fonction de perte : La fonction de perte mesure l'écart entre les prédictions du modèle et les annotations réelles. La fonction de perte appropriée dépendra de votre tâche spécifique. Pour l'amélioration d'image, vous pouvez utiliser une fonction de perte comme la différence moyenne des carrés (MSE) ou le coefficient de corrélation de Pearson (PCC).
    
5.  Entraînez le modèle : Utilisez le jeu de données d'entraînement et de validation pour entraîner votre modèle. Vous pouvez utiliser des techniques telles que l'augmentation des données pour augmenter la taille du jeu de données et améliorer les performances du modèle.
    
6.  Évaluez le modèle : Une fois que vous avez entraîné le modèle, évaluez sa performance sur l'ensemble de validation. Vous pouvez utiliser des mesures telles que la précision, le rappel et le score F1 pour évaluer les performances de votre modèle.
    
7.  Utilisez le modèle pour prédire : Une fois que vous êtes satisfait des performances de votre modèle, vous pouvez l'utiliser pour prédire les améliorations à apporter à de nouvelles images en temps réel.
    

En résumé, préparer les données, créer un jeu de données personnalisé, définir un modèle, définir une fonction de perte, entraîner le modèle, évaluer le modèle et utiliser le modèle pour prédire. Cela nécessite une bonne compréhension de PyTorch et des concepts d'apprentissage automatique.

# Le code
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
from torchvision import transforms
from PIL import Image
import pandas as pd
import os

# Créer une classe de jeu de données personnalisée
class ImageDataset(Dataset):
    def __init__(self, csv_file, root_dir, transform=None):
        self.annotations = pd.read_csv(csv_file)
        self.root_dir = root_dir
        self.transform = transform

    def __len__(self):
        return len(self.annotations)

    def __getitem__(self, index):
        img_path = os.path.join(self.root_dir, self.annotations.iloc[index, 0])
        image = Image.open(img_path).convert('RGB')
        if self.transform:
            image = self.transform(image)
        return (image, image)

# Créer une classe de modèle personnalisée
class ImageModel(nn.Module):
    def __init__(self):
        super(ImageModel, self).__init__()
        self.conv1 = nn.Conv2d(3, 64, kernel_size=3, padding=1)
        self.bn1 = nn.BatchNorm2d(64)
        self.conv2 = nn.Conv2d(64, 64, kernel_size=3, padding=1)
        self.bn2 = nn.BatchNorm2d(64)
        self.conv3 = nn.Conv2d(64, 3, kernel_size=3, padding=1)

    def forward(self, x):
        x = self.conv1(x)
        x = self.bn1(x)
        x = nn.functional.relu(x)
        x = self.conv2(x)
        x = self.bn2(x)
        x = nn.functional.relu(x)
        x = self.conv3(x)
        return x

# Définir les hyperparamètres et les options d'entraînement
batch_size = 16
learning_rate = 0.001
num_epochs = 10
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# Charger les données d'entraînement et de validation
train_transform = transforms.Compose([
    transforms.RandomHorizontalFlip(),
    transforms.RandomRotation(10),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])
train_dataset = ImageDataset('train.csv', 'images/', train_transform)
train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)

val_transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])
val_dataset = ImageDataset('val.csv', 'images/', val_transform)
val_loader = DataLoader(val_dataset, batch_size=batch_size, shuffle=False)

# Instancier le modèle et le déplacer sur le GPU si disponible
model = ImageModel().to(device)

# Définir la fonction de perte et l'optimiseur
criterion = nn.MSELoss()
optimizer = optim.Adam(model.parameters(), lr=learning_rate)

# Entraîner le modèle
for epoch in range(num_epochs):
    running_loss = 0.0
    for i, data in enumerate(train_loader, 0):
        inputs, labels = data
        inputs, labels = inputs.to(device), labels.to(device)
        optimizer.zero_grad()
        outputs = model(inputs)
       
    loss = criterion(outputs, labels)
    loss.backward()
    optimizer.step()
    running_loss += loss.item()
    
# Évaluer le modèle sur l'ensemble de validation
with torch.no_grad():
    val_loss = 0.0
    for j, val_data in enumerate(val_loader, 0):
        val_inputs, val_labels = val_data
        val_inputs, val_labels = val_inputs.to(device), val_labels.to(device)
        val_outputs = model(val_inputs)
        val_loss += criterion(val_outputs, val_labels).item()
    val_loss /= (j + 1)

# Afficher les statistiques d'entraînement pour l'époque
print(f"Epoch {epoch+1} - Training Loss: {running_loss/len(train_loader):.4f} - Validation Loss: {val_loss:.4f}")
# Enregistrer le modèle entraîné
torch.save(model.state_dict(), 'image_model.pth')
```


Ce code utilise PyTorch pour créer une classe de jeu de données personnalisée qui charge les images et les transforme, ainsi qu'une classe de modèle personnalisée qui améliore les images. Il entraîne ensuite le modèle en utilisant les données d'entraînement et évalue ses performances sur les données de validation. Enfin, il enregistre le modèle entraîné dans un fichier pour une utilisation ultérieure.

Ce code est un exemple et doit être adapté en fonction de votre cas d'utilisation spécifique. Vous devrez notamment adapter les transformations d'images et les hyperparamètres pour obtenir les meilleurs résultats possibles.

# Utilisation
Exemple de code qui charge le modèle entraîné à partir du fichier `image_model.pth` et l'utilise pour améliorer une nouvelle collection d'images :
```python
import os
from PIL import Image
import torch
from torchvision import transforms

# Charger le modèle entraîné
model = ImageModel()
model.load_state_dict(torch.load('image_model.pth'))
model.eval()

# Transformer les images avec les mêmes transformations utilisées pour l'entraînement
transform = transforms.Compose([
    transforms.Resize((256, 256)),
    transforms.CenterCrop((224, 224)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

# Parcourir les images et les améliorer avec le modèle
for i in range(1, 101):
    # Charger l'image
    image_path = f"img{i:03}.jpg"
    image = Image.open(image_path).convert("RGB")
    
    # Appliquer les transformations
    image = transform(image).unsqueeze(0)  # Ajouter une dimension pour le batch
    
    # Utiliser le modèle pour améliorer l'image
    with torch.no_grad():
        improved_image = model(image).squeeze(0)
    
    # Enregistrer l'image améliorée
    improved_image = improved_image.permute(1, 2, 0)  # Permuter les dimensions pour les sauvegarder en tant qu'image
    improved_image = (improved_image * 0.225 + 0.406).clamp(0, 1) * 255  # Inverser la normalisation et la mise à l'échelle
    improved_image = improved_image.cpu().numpy().astype('uint8')  # Convertir en tableau numpy
    improved_image = Image.fromarray(improved_image)
    improved_image_path = f"img_corr_{i:03}.jpg"
    improved_image.save(improved_image_path)

```
