# Asteroid Hazard Classification

A multi-method classification project predicting whether a near-Earth celestial object poses a hazard to Earth, using NASA NeoWs data (1950–2025) and an ensemble of variable selection and tree-based methods in R.

## Project Overview

This project was completed as part of STA5703 Data Mining Methodology at the University of Central Florida. The goal was to build and compare machine learning models capable of reliably classifying a near-Earth object (NEO) as hazardous or non-hazardous based on its orbital and physical characteristics. Accurate classification matters because both false negatives (missing a real threat) and false positives (misallocating planetary defense resources) carry serious consequences.

The project progressed in two stages: variable selection via logistic regression, ridge, and LASSO, followed by ensemble modeling using Random Forest, Bagging, and Gradient Boosting.

## Dataset

- **Source:** NASA NeoWs 1950–2025 via Kaggle ("Near Earth Comets Classification")
- **Raw:** 375,659 rows, 41 columns
- **After preprocessing:** 35,430 rows, 19 columns (18 predictors, 1 binary response)
- **Response variable:** `Hazardous` (TRUE / FALSE)
- **Class imbalance:** ~93% non-hazardous, ~7% hazardous
- **Split strategy:** 60% training / 20% validation / 20% test (stratified, seed = 5703)

## Preprocessing

- Removed redundant columns: Name, Equinox, and non-kilometer distance/velocity unit columns
- Removed date and timestamp columns with no explanatory power
- Filtered to Earth-orbiting objects only
- Removed duplicate observations by `Neo Reference ID` to prevent data leakage
- Converted `Minimum.Orbit.Intersection` from scientific notation string to numeric
- Converted `Hazardous` to factor for classification compatibility

## Methods

### Stage 1: Variable Selection

| Method | Variables Used | AIC (Training) | AUC (Validation) |
| --- | --- | --- | --- |
| Baseline (Full Logistic) | 18 | 2238.383 | 0.9869 |
| Stepwise (Bidirectional) | 5 | 2220.360 | 0.9868 |
| Best Subset | 6 | 2225.745 | 0.9869 |
| Ridge (lambda.min) | 18 | N/A | 0.9696 |
| LASSO (lambda.min) | 13 | N/A | 0.9870 |

LASSO was selected as the best variable selection model: it matched the highest validation AUC while performing automatic variable selection and reducing overfitting risk. The most consistently important predictors across all methods were Minimum Orbit Intersection, Absolute Magnitude, and Orbit Uncertainty.

### Stage 2: Ensemble Methods

Three ensemble models were trained and tuned, each evaluated on the held-out test set:

- **Random Forest:** Tuned via OOB error plots to select mtry = 4; ROC-optimized classification threshold of 0.528
- **Bagging:** Pure bagging (mtry = p); 100, 300, and 500 tree models compared; 300-tree model selected for optimal AUC and efficiency; classification threshold of 0.025 to maximize sensitivity
- **Gradient Boosting (GBM):** 5-fold cross-validation with internal downsampling to handle class imbalance; grid search over trees (100–600), interaction depth (1–5), and shrinkage (0.001/0.01/0.1); optimal threshold of 0.929 via pROC

## Results

| Model | Accuracy | Sensitivity | Specificity | Kappa | F1 | AUC |
| --- | --- | --- | --- | --- | --- | --- |
| Random Forest | 0.9975 | 0.9840 | 0.9985 | 0.9806 | 0.9820 | 0.9994 |
| Bagging | 0.9663 | 0.9960 | 0.9640 | 0.7884 | 0.8062 | 0.9987 |
| Boosting | 0.9980 | 0.9840 | 0.9991 | 0.9849 | 0.9859 | 0.9996 |

All three models significantly outperformed the best variable selection baseline (LASSO AUC = 0.9870). Boosting achieved the strongest overall balance across metrics, with the highest AUC (0.9996), Kappa (0.9849), and specificity (0.9991). Bagging achieved the highest sensitivity (0.9960) with only 2 false negatives, at the cost of 237 false positives, making it the most conservative choice for hazard detection. Random Forest maintained strong performance across all metrics with the simplest tuning process.

## Project Structure

```
AsteroidHazardClassification/
├── README.md
├── Team3_Hazardous_Celestial_Objects_Full_Analysis.Rmd   # Full analysis pipeline
├── Final_Project_Report.docx                             # Written report
└── Hazardous_Celestial_Objects_Presentation.pptx         # Final presentation
```

## Setup

1. Clone this repo
2. Download the dataset from Kaggle: [Near Earth Comets Classification](https://www.kaggle.com/datasets/seniruepasinghe/near-earth-comets-classification-dataset)
3. Place `nasa_neows_1950_2025.csv` in your working directory
4. Install required R packages:

```r
install.packages(c(
  "dplyr", "ggplot2", "tidyr", "knitr", "kableExtra",
  "caret", "randomForest", "pROC", "glmnet", "leaps",
  "MASS", "rpart", "rpart.plot", "gbm", "scales",
  "tibble", "corrplot", "GGally"
))
```

5. Open and knit `Team3_Hazardous_Celestial_Objects_Full_Analysis.Rmd`

## Team

| Member | Contribution |
| --- | --- |
| Alejandro Arroyo | Data pipeline architecture and preprocessing; consolidated and debugged group members' inconsistent preprocessing approaches into a unified pipeline; integrated Random Forest, Bagging, and GBM models into a single reproducible Rmd; coordinated final model comparison and report |
| Ryujin Bupp | Bagging implementation and tuning |
| Jose Placeres | Random Forest implementation and tuning |
| Fernando Ricaurte | Gradient Boosting implementation and tuning |

## Course

STA5703 Data Mining Methodology. University of Central Florida, Fall 2025.
