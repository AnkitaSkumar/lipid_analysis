# Lipid Residue Analysis of Indus Valley Civilisation Pottery

Data cleaning and machine learning pipeline for analyzing lipid (fatty acid) residues extracted from Indus Valley Civilisation (IVC) pottery sherds.


## Table of Contents

- [Background](#background)
- [What This Notebook Does](#what-this-notebook-does)
- [Requirements](#requirements)
- [Data](#data)
- [Usage](#usage)
- [Outputs](#outputs)
- [Notes](#notes)

## Background

Archaeological lipid residue analysis measures fatty acid biomarkers preserved in pottery to infer what foods (animal fats, plant products, aquatic resources) were processed in a vessel. A key indicator is the **P/S ratio** (palmitic-to-stearic acid ratio), used to distinguish ruminant fats, non-ruminant fats, and plant-derived residues.

This project cleans a raw dataset of pottery sherd lipid measurements, engineers a categorical target from the P/S ratio, and applies regression and clustering models to explore what the chemical signatures reveal.

## What This Notebook Does

### 1. Load & Inspect Data
Reads `data.csv` and lists all available features.

### 2. Feature Selection
- Selects a small set of categorical features (`Cholesterol_derivatives`, `longchain_Alkanes`, `Alcohols`, `Sulphur`) based on relevance identified in prior research, dropping `Branched` due to excessive missing values.
- Screens ~20 candidate numerical fatty-acid concentration columns and keeps only those with at least 120 valid (non-NaN) values out of 124 total samples.

### 3. Target Construction
Bins the continuous `PS_ratio` into four categorical labels:

| Label | Condition          | Interpretation      |
|-------|---------------------|----------------------|
| `d`   | PS_ratio < 1         | Ruminant fats        |
| `r`   | 1 ≤ PS_ratio < 1.2    | Ruminant fats        |
| `nr`  | 1.2 ≤ PS_ratio ≤ 2    | Non-ruminant fats     |
| `p`   | PS_ratio > 2          | Plant-derived         |

### 4. Data Type Cleanup
Coerces numerical columns to proper numeric dtype and converts categorical `Present`/`Absent` values to boolean.

### 5. Correlation Analysis
Plots a heatmap of feature correlations and drops one feature from each perfectly correlated pair (`C16conc_ug_vial`, `C18conc_ug_vial`).

### 6. Regression Modeling
Trains and evaluates two regressors to predict `PS_ratio` from fatty acid concentrations:

- `RandomForestRegressor`
- `XGBRegressor`

Each model is evaluated with a train/test split, 5-fold cross-validation (MAE, RMSE), and both impurity-based and permutation feature importance.

### 7. Clustering
Applies K-Means (k=4) to the cleaned feature set — both on raw scaled features and after PCA (retaining 95% variance) — comparing resulting clusters against the true PS_ratio-derived class labels via a confusion matrix.

## Requirements

- Python 3.11
- `numpy`, `pandas`, `seaborn`, `matplotlib`
- `scikit-learn`
- `xgboost`

Install with:

```bash
pip install numpy pandas seaborn matplotlib scikit-learn xgboost
```

## Data

The notebook expects a `data.csv` file in the same directory, containing pottery sherd sample records with fatty acid concentration columns, categorical lipid markers, and a `PS_ratio` column. **This file is not included in this repository** and must be supplied separately.

## Usage

Clone the repository and run the notebook top to bottom:

```bash
git clone <your-repo-url>
cd <your-repo-name>
jupyter notebook indusValleyDataCleaningUpdated.ipynb
```

> Cells are organized sequentially — feature selection and cleaning must run before the modeling and clustering sections, since later cells depend on `FinalDataFrameForML` built earlier in the notebook.

## Outputs

- Printed diagnostics on missing values and feature selection thresholds
- Correlation heatmaps (before/after removing redundant features)
- Actual-vs-predicted scatter plots for both regressors
- Cross-validated MAE/RMSE scores
- Ranked feature importances (impurity-based and permutation-based)
- Confusion matrices comparing K-Means clusters to PS_ratio-derived classes (raw features and PCA-reduced features)

## Notes

- The classification thresholds for the P/S ratio bins were chosen based on domain interpretation in the associated research and can be adjusted in the target-construction cell if needed.
- The numerical feature inclusion threshold (120 out of 124 samples) can be tuned via the `thresholdNumReals` argument to `reliableNumericalFeatureSelector`.

