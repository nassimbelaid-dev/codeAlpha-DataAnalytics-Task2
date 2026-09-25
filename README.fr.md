# Dataset Diabète — Analyse Exploratoire des Données (EDA)

## 1. Présentation du projet

Ce projet est consacré à l’analyse exploratoire du jeu de données **Pima Indians Diabetes** (`diabetes.csv`). Il répond spécifiquement à la **Tâche 2 : Analyse Exploratoire des Données (EDA)**, dont l’objectif est d’explorer, de comprendre et d’interroger les données avant toute étape de modélisation.

## 2. Objectif de l’EDA

L’objectif de ce notebook est de :

* Poser des questions pertinentes avant toute analyse.
* Explorer la structure des données (variables, types, dimensions).
* Identifier les tendances, les modèles et les anomalies présents dans les données.
* Formuler et tester des hypothèses simples à l’aide de statistiques et de visualisations.
* Détecter d’éventuels problèmes de qualité des données (valeurs manquantes, doublons, valeurs suspectes) qui seront traités lors d’une analyse ultérieure.

## 3. Description du jeu de données

Le jeu de données `diabetes.csv` (Pima Indians Diabetes Database) contient **768 observations** et **9 variables** :

| Variable                 | Description                                         |
| ------------------------ | --------------------------------------------------- |
| Pregnancies              | Nombre de grossesses                                |
| Glucose                  | Concentration de glucose plasmatique                |
| BloodPressure            | Pression artérielle diastolique (mm Hg)             |
| SkinThickness            | Épaisseur du pli cutané du triceps (mm)             |
| Insulin                  | Insuline sérique à 2 heures (mu U/ml)               |
| BMI                      | Indice de masse corporelle (IMC)                    |
| DiabetesPedigreeFunction | Fonction de pedigree du diabète                     |
| Age                      | Âge du patient (années)                             |
| Outcome                  | Variable cible : 0 = non diabétique, 1 = diabétique |

## 4. Organisation du notebook

Le notebook `diabetes_EDA.ipynb` est organisé en sections claires :

1. Importation des bibliothèques
2. Chargement du jeu de données
3. Questions avant l’analyse
4. Exploration de la structure des données
5. Statistiques descriptives
6. Distribution des classes
7. Analyse des corrélations
8. Visualisation des données
9. Tests d’hypothèses
10. Évaluation de la qualité des données
11. Conclusions finales / Synthèse de l’EDA

## 5. Bibliothèques utilisées

* `pandas` — manipulation et analyse des données
* `numpy` — calculs numériques
* `matplotlib` — visualisation de base
* `seaborn` — visualisation statistique (heatmap, boxplot, countplot)

Voir le fichier `requirements.txt` pour les versions recommandées.

## 6. Étapes de l’analyse

1. Charger le jeu de données et vérifier sa structure (dimensions, types).
2. Calculer les statistiques descriptives (moyenne, écart-type, médiane, minimum, maximum).
3. Étudier la distribution de la variable cible `Outcome`.
4. Analyser les corrélations entre les variables (matrice de covariance/corrélation + heatmap).
5. Visualiser les distributions (histogrammes, matrice de dispersion, boxplots).
6. Formuler et tester trois hypothèses simples (Glucose, BMI et Age par rapport à `Outcome`).
7. Détecter les problèmes de qualité des données (valeurs manquantes, doublons, zéros suspects).
8. Résumer les observations finales (Synthèse de l’EDA).

## 7. Principaux résultats

* Le jeu de données ne contient ni valeurs manquantes explicites ni doublons.
* Plusieurs colonnes (`Glucose`, `BloodPressure`, `SkinThickness`, `Insulin`, `BMI`) contiennent des valeurs nulles qui n’ont pas de sens physiologique et représentent probablement des valeurs manquantes codées comme zéro.
* Le jeu de données est déséquilibré : environ **65 % de patients non diabétiques contre 35 % de patients diabétiques**.
* Les valeurs moyennes de `Glucose`, `BMI` et `Age` sont plus élevées chez les patients diabétiques, ce qui confirme les trois hypothèses testées.
* Aucune corrélation très forte n’a été détectée entre les variables explicatives.

## 8. Comment exécuter le notebook

1. Installer les dépendances :

   ```bash
   pip install -r requirements.txt
   ```

2. Placer le fichier `diabetes.csv` dans le même dossier que le notebook (ou adapter le chemin dans la cellule « Load Dataset »).

3. Lancer Jupyter et exécuter le notebook cellule par cellule :

   ```bash
   jupyter notebook diabetes_EDA.ipynb
   ```

## 9. Structure des fichiers

```text
.
├── diabetes_EDA.ipynb    # Notebook d’analyse exploratoire des données (EDA)
├── diabetes.csv          # Jeu de données (à fournir par l’utilisateur)
├── README.md             # Ce fichier
└── requirements.txt      # Dépendances Python
```

## 10. Auteur

*[Nassim BELAID]*
