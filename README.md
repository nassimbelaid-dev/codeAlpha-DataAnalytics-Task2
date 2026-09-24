# Diabetes Dataset — Exploratory Data Analysis (EDA)

## 1. Project Overview

This project is focused on the exploratory analysis of the
**Pima Indians Diabetes** dataset (`diabetes.csv`). It specifically addresses **Task 2:
Exploratory Data Analysis (EDA)**, whose goal is to explore, understand and question the
data before any modeling step.

## 2. EDA Objective

The purpose of this notebook is to:

- Ask meaningful questions before any analysis.
- Explore the structure of the data (variables, types, dimensions).
- Identify trends, patterns and anomalies present in the data.
- Formulate and test simple hypotheses using statistics and visualizations.
- Detect potential data quality issues (missing values, duplicates, suspicious values)
  to be addressed in a later analysis.

## 3. Dataset Description

The `diabetes.csv` dataset (Pima Indians Diabetes Database) contains **768 observations**
and **9 variables**:

| Variable | Description |
|---|---|
| Pregnancies | Number of pregnancies |
| Glucose | Plasma glucose concentration |
| BloodPressure | Diastolic blood pressure (mm Hg) |
| SkinThickness | Triceps skin fold thickness (mm) |
| Insulin | 2-Hour serum insulin (mu U/ml) |
| BMI | Body mass index |
| DiabetesPedigreeFunction | Diabetes pedigree function |
| Age | Patient's age (years) |
| Outcome | Target variable: 0 = non-diabetic, 1 = diabetic |

## 4. Notebook Organization

The `diabetes_EDA.ipynb` notebook is organized into clear sections:

1. Import Libraries
2. Load Dataset
3. Questions before the analysis
4. Data Structure Exploration
5. Descriptive Statistics
6. Class Distribution
7. Correlation Analysis
8. Data Visualization
9. Hypothesis Testing
10. Data Quality Assessment
11. Final Conclusions / Final EDA Summary

## 5. Libraries Used

- `pandas` — data manipulation and analysis
- `numpy` — numerical computations
- `matplotlib` — basic visualization
- `seaborn` — statistical visualization (heatmap, boxplot, countplot)

See `requirements.txt` for recommended versions.

## 6. Analysis Steps

1. Load the dataset and check its structure (dimensions, types).
2. Compute descriptive statistics (mean, standard deviation, median, min, max).
3. Study the distribution of the target variable `Outcome`.
4. Analyze correlations between variables (covariance/correlation matrix + heatmap).
5. Visualize distributions (histograms, scatter matrix, boxplots).
6. Formulate and test three simple hypotheses (Glucose, BMI, Age vs Outcome).
7. Detect data quality issues (missing values, duplicates, suspicious zeros).
8. Summarize the final observations (Final EDA Summary).

## 7. Key Findings

- The dataset contains neither explicit missing values nor duplicates.
- Several columns (Glucose, BloodPressure, SkinThickness, Insulin, BMI) contain zero
  values that make no physiological sense and likely represent missing values coded
  as zero.
- The dataset is imbalanced: about 65% non-diabetic patients versus 35% diabetic.
- Glucose, BMI and Age are on average higher in diabetic patients, confirming the
  three tested hypotheses.
- No very strong correlation was detected between the explanatory variables.

## 8. How to Run the Notebook

1. Install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Place the `diabetes.csv` file in the same folder as the notebook (or adapt the
   path in the "Load Dataset" cell).
3. Launch Jupyter and run the notebook cell by cell:
   ```bash
   jupyter notebook diabetes_EDA.ipynb
   ```

## 9. File Structure

```
.
├── diabetes_EDA.ipynb    # Exploratory Data Analysis (EDA) notebook
├── diabetes.csv          # Dataset (to be provided by the user)
├── README.md             # This file
└── requirements.txt      # Python dependencies
```

## 10. Author

_[Nassim BELAID]_
