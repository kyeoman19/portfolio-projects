# Aluminum Plate Flow Stress Prediction

**Predicting manufacturing forces for aluminum plate stretching operations using machine learning**

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-orange.svg)
![LightGBM](https://img.shields.io/badge/LightGBM-Gradient_Boosting-green.svg)

## Overview

This project develops a predictive model for aluminum plate manufacturing operations. When aluminum plates are plastically stretched (~2% elongation) to relieve internal stresses, the required force depends on the alloy type and plate geometry. Accurate force predictions enable:

- **Feasibility assessment** for new plate designs before production
- **Equipment protection** by preventing overload conditions
- **Optimized scheduling** by anticipating high-force operations
- **Quality improvement** through consistent process parameters

## Key Results

| Model | Test RMSE | Test R² | Use Case |
|-------|-----------|---------|----------|
| **Random Forest** | 1,387 psi | **0.978** | Best mean predictor |
| Linear Regression | 1,445 psi | 0.976 | Interpretable baseline |
| **LightGBM (99th percentile)** | 4,882 psi | 0.72 | Safety bounds (~99% coverage) |

The Random Forest model explains **97.8% of variance** in flow stress, with typical prediction errors of only ~3% of actual values.

## Dataset

- **486,396 stretch operations** spanning 2013-2023
- **18 aluminum alloy types** after filtering rare variants
- **5 input features**: Alloy, Thickness, Width, Length, Stretch Target
- **Target variable**: Flow Stress (psi) - force normalized by cross-sectional area

## Technical Approach

### Data Pipeline
1. **Data Cleaning**: Duplicate removal, invalid value correction, outlier filtering (>60k psi)
2. **Feature Engineering**: One-hot encoding for alloys, standard scaling for numerics
3. **Stratified Split**: 80/20 train-test split stratified by alloy type

### Models Developed
- **Linear Regression** (with Ridge/Lasso regularization testing)
- **Random Forest** with hyperparameter tuning via RandomizedSearchCV
- **LightGBM Quantile Regression** for 99th percentile prediction (safety bounds)

### Key Findings
- **Alloy type** is the dominant predictor - accounts for largest variance
- **Thickness** provides secondary refinement (thicker plates require slightly less stress)
- **Width/Length** have negligible impact on stress (already normalized)
- Process is largely **linear and deterministic** - high accuracy achievable with simple models

## Project Structure

```
├── Notebooks/
│   ├── capstone2_AlumaTech_data_wrangling.ipynb  # Data cleaning & merging
│   ├── capstone2_AlumaTech_EDA.ipynb             # Exploratory analysis
│   ├── capstone2_AlumaTech_preprocessing.ipynb   # Feature engineering
│   └── capstone2_AlumaTech_modeling.ipynb        # Model training & evaluation
├── data/                                          # Raw and processed datasets
├── Images/                                        # Visualizations
├── best_random_forest_model.pkl                   # Trained RF model
├── model_Xth_percentile.pkl                       # Trained quantile model
├── preprocessor.pkl                               # Sklearn preprocessing pipeline
└── Report.md                                      # Detailed technical report
```

## Visualizations

The analysis includes:
- Flow stress distribution histograms (bimodal pattern by alloy strength)
- Boxplots of flow stress by alloy type
- Correlation heatmaps
- Actual vs. predicted scatter plots
- Feature importance rankings

## Business Impact

This model enables the manufacturing team to:
1. **Evaluate new designs upfront** - determine feasibility before production trials
2. **Plan high-force jobs strategically** - schedule maintenance and inspections appropriately
3. **Set accurate process parameters** - apply correct force on first attempt
4. **Quantify safety margins** - 99th percentile model provides equipment protection bounds

## Technologies Used

- **Python 3.8+**
- **pandas** / **NumPy** - Data manipulation
- **scikit-learn** - Preprocessing, Random Forest, model evaluation
- **LightGBM** - Gradient boosting with quantile regression
- **matplotlib** / **seaborn** - Visualization

## Future Enhancements

- Incorporate plate temperature as an additional feature
- Develop alloy-specific sub-models for edge cases
- Build web interface or Excel plugin for shop floor access
- Automated retraining pipeline with new production data