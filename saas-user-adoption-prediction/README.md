# User Adoption Prediction for SaaS Platform

**Predicting which users will become active adopters using engagement data**

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-orange.svg)
![Classification](https://img.shields.io/badge/Task-Classification-yellow.svg)

## Overview

This project builds a predictive model to identify which users will "adopt" a SaaS product after signing up. User adoption is defined as having **3 or more logins within any 7-day window** - a signal of genuine engagement versus one-time signups.

Understanding adoption drivers enables:
- **Targeted onboarding** for at-risk user segments
- **Resource allocation** toward high-potential users
- **Product improvements** based on adoption correlates

## Key Results

| Model | ROC-AUC | PR-AUC | Accuracy |
|-------|---------|--------|----------|
| **Random Forest** | **0.917** | **0.825** | **94.0%** |
| Logistic Regression | 0.862 | 0.706 | 82.1% |

The Random Forest model achieves **91.7% AUC** with strong precision (90.6%) for identifying adopters.

### Cross-Validation Performance
- **5-fold CV ROC-AUC**: 0.935 ± 0.005 (highly stable)

## Dataset

Two data sources totaling **12,000 users**:

| File | Contents |
|------|----------|
| `takehome_users.csv` | User metadata (12,000 records) |
| `takehome_user_engagement.csv` | Login timestamps (5.8 MB) |

### Features Used
- **Invitation status** - Was user invited by existing user?
- **Organization size** - Quartile groupings
- **Creation source** - ORG_INVITE, GUEST_INVITE, SIGNUP, etc.
- **Marketing preferences** - Opted-in vs. opted-out
- **Creation timing** - Year of signup

### Target Variable
- **Adopted** (binary): 3+ logins within any 7-day period
- Baseline adoption rate: ~14%

## Key Findings

### Adoption Drivers (by importance)
1. **Organization size** - Smaller orgs have higher adoption (17.2% vs 9.3% for largest)
2. **Invitation status** - Invited users adopt at higher rates (14.7% vs 12.8%)
3. **Creation source** - Referral channels outperform organic signup
4. **Last session timing** - Recency correlates with adoption

### Business Insights
- Focus onboarding resources on users from **large organizations** (high risk)
- **Invitation/referral programs** drive meaningful adoption lift
- Users who don't engage within first week rarely adopt later

## Technical Approach

### Data Pipeline
1. **Adoption calculation**: Custom algorithm scanning login timestamps for 7-day windows with 3+ logins
2. **Feature engineering**: Organization size quartiles, binary flags, source encoding
3. **Train-test split**: 80/20 with stratification by adoption status

### Models Compared
- **Logistic Regression** (L2 regularized) - Interpretable baseline
- **Random Forest** (500 trees, balanced class weights) - Best performer

### Evaluation Metrics
- **ROC-AUC** - Primary metric for imbalanced classification
- **PR-AUC** - Precision-recall tradeoff for minority class
- **Permutation importance** - Feature contribution analysis

## Project Structure

```
├── user_adoption_analysis.ipynb   # Main analysis notebook
├── adoption_charts.png             # Adoption rate visualizations
├── rf_roc_curve.png               # Random Forest ROC curve
├── lr_roc_curve.png               # Logistic Regression ROC curve
├── rf_feature_importance.png      # RF feature rankings
├── rf_permutation_importance.png  # Permutation-based importance
├── lr_feature_importance_coeff_*.png  # LR coefficient analysis
├── takehome_users.csv             # User metadata
└── takehome_user_engagement.csv   # Login timestamps
```

## Visualizations

The analysis includes:
- Adoption rates by user segment (stacked bar charts)
- ROC curves comparing model performance
- Feature importance rankings (both coefficient-based and permutation)
- Precision-recall analysis

## Technologies Used

- **Python 3.8+**
- **pandas** / **NumPy** - Data manipulation
- **scikit-learn** - Logistic Regression, Random Forest, evaluation metrics
- **matplotlib** / **seaborn** - Visualization

## Recommendations

Based on the model insights:

1. **Segment onboarding** - Create specialized flows for large-org users
2. **Incentivize invitations** - Referral-based signups show higher adoption
3. **Early engagement triggers** - Users need 3+ sessions in week 1 to adopt
4. **Monitor org size** - Largest organizations require additional support
