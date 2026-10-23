## Projet : Régression d'Image vers LAI
Aperçu
Ce projet vise à prédire le LAI (Leaf Area Index, indice de surface foliaire) à partir d'images de couvert végétal, en utilisant des techniques de régression supervisée par apprentissage automatique / profond.
Le notebook regression-lai.ipynb constitue le point d'entrée principal et contient l'ensemble du pipeline : chargement des données, exploration, prétraitement, entraînement du modèle et évaluation.

### Structure du projet
text
Data/
└── images/
    └── <nom_image>.jpg    # ex. : A2_1_par_57_Lataule_0613.jpg
Le notebook charge une image depuis ../Data/images/ et affiche ses dimensions (~2328 × 1534 pixels), ce qui suggère des images aériennes/parcellaires de haute résolution.

### Objectif scientifique
Le LAI est un indicateur clé en agronomie et en télédétection (suivi de la biomasse, de la croissance, du stress hydrique). L'objectif est de construire un modèle capable d'estimer cette variable directement à partir des pixels, sans mesure destructrice.

### Pipeline général du notebook
#### Chargement et inspection des données
Lecture des images (PIL / OpenCV / matplotlib).
Vérification des dimensions, canaux et du parcours des fichiers.
#### Prétraitement
Redimensionnement (les images brutes sont très grandes).
Normalisation des valeurs de pixels.
Éventuellement augmentation de données et extraction de patches.

### Modélisation
Approche de régression (CNN ou modèle pré-entraîné type ResNet / EfficientNet avec sortie scalaire).
Fonction de perte adaptée (MSE, MAE, Huber).
Découpage train / validation / test.

### Évaluation
Métriques : R², RMSE, MAE.
Visualisation prédit vs. observé.
Analyse des résidus.

### Prérequis
text
python >= 3.9
numpy
pandas
matplotlib
scikit-learn
torch (ou tensorflow/keras)
opencv-python / pillow
jupyter

###Installation :
bash
pip install -r requirements.txt
# ou
pip install numpy pandas matplotlib scikit-learn torch torchvision opencv-python pillow jupyter
Utilisation
bash
jupyter notebook regression-lai.ipynb
Puis exécuter les cellules dans l'ordre. Adapter le chemin des images (../Data/images/) selon votre arborescence locale.


