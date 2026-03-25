# Data Science Portfolio

A collection of machine learning and data analysis projects demonstrating expertise in predictive modeling, time series analysis, and industrial applications.

## Projects

### [Aluminum Flow Stress Prediction](./aluminum-flow-stress-prediction)
**Industrial Manufacturing ML** | R² = 0.978

Predictive model for aluminum plate stretching operations using 486K+ historical records. Developed dual-model approach: Random Forest for mean prediction and LightGBM quantile regression for safety bounds (~99% coverage).

**Tech:** Python, scikit-learn, LightGBM, pandas

---

### [3D Rail Alignment Analysis](./3d-rail-alignment-analysis)
**Geospatial Engineering** | 769K+ point cloud

Machine learning pipeline for overhead crane rail alignment using laser scanning. Combines Random Forest classification with GMM clustering to evaluate CMAA industry compliance tolerances.

**Tech:** Python, scikit-learn, CloudCompare, 3D point cloud processing

---

### [SaaS User Adoption Prediction](./saas-user-adoption-prediction)
**Product Analytics** | AUC = 0.917

Classification model predicting which users will adopt a SaaS product (3+ logins in 7 days). Identified key drivers: organization size, invitation status, and creation source.

**Tech:** Python, scikit-learn, pandas, matplotlib

---

### [Ride-Sharing Demand & Retention](./rideshare-demand-retention)
**Transportation Analytics** | AUC = 0.825

Two-part analysis: (1) Time series demand pattern analysis with anomaly detection, (2) Rider retention prediction identifying churn drivers including surge pricing and early engagement.

**Tech:** Python, scikit-learn, time series analysis, JSON parsing

---

## Skills Demonstrated

- **Machine Learning**: Random Forest, Gradient Boosting, Logistic Regression, Clustering (GMM, DBSCAN, KMeans)
- **Data Engineering**: Large-scale data processing (500K+ records), feature engineering, preprocessing pipelines
- **Domain Expertise**: Manufacturing, geospatial/3D analysis, product analytics, transportation
- **Evaluation**: ROC-AUC, precision-recall, cross-validation, quantile regression for safety bounds
