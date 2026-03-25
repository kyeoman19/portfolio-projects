# SaaS User Adoption Prediction

## Problem

When users sign up for a SaaS product, only a fraction become genuinely engaged. This project tackles a core product-analytics question: **which users will adopt the platform after signing up?** Adoption is defined as logging in 3 or more times within any 7-day window — a behavioral threshold that separates casual sign-ups from users who are integrating the product into their workflow. Accurately predicting adoption enables targeted onboarding interventions, smarter resource allocation, and data-driven product improvements.

## Data & Feature Engineering

The analysis draws on two datasets covering 12,000 users: a user-metadata table (sign-up source, organization, invitation status, marketing preferences) and a login-timestamp log. A custom sliding-window algorithm scans each user's login history to label them as adopted or not, producing a class-imbalanced target with a ~14% adoption rate. Engineered features include organization-size quartiles, a binary invitation flag, and creation-year indicators — all fed through a scikit-learn pipeline with median imputation, standard scaling for numerics, and one-hot encoding for categorical sources.

## Modeling & Results

Two classifiers were compared — L2-regularized Logistic Regression and a 500-tree Random Forest with balanced class weights — evaluated on a stratified 75/25 train-test split.

| Metric | Logistic Regression | Random Forest |
|---|---|---|
| ROC-AUC | 0.862 | **0.917** |
| PR-AUC | 0.706 | **0.825** |
| Accuracy | 82.1% | **94.0%** |
| Precision (adopters) | 42.1% | **90.6%** |
| 5-Fold CV AUC | 0.879 ± 0.011 | **0.935 ± 0.005** |

The Random Forest is the stronger model across every metric, with highly stable cross-validation performance (±0.005 AUC) indicating minimal overfitting.

## Key Findings

Permutation-importance analysis reveals the strongest adoption drivers. **Organization size** is the most influential feature — users from smaller organizations adopt at nearly twice the rate of those from the largest (17.2% vs. 9.3%). **Invitation status** provides a meaningful lift: invited users adopt at 14.7% compared to 12.8% for organic sign-ups, underscoring the value of referral programs. **Creation source** matters too, with org-invite and guest-invite channels outperforming self-signup and personal-project origins. Finally, **last-session recency** is a strong signal — users who disengage early almost never convert later.

## Business Recommendations

These findings translate into four actionable strategies. First, **segment the onboarding experience** — users from large organizations are highest-risk and would benefit from tailored walkthroughs or dedicated support. Second, **double down on invitation and referral programs**, since socially-driven sign-ups consistently show higher engagement. Third, **trigger early-engagement nudges** within the first week (push notifications, feature tours, check-in emails), because the data shows a narrow window in which adoption is decided. Fourth, **monitor organization-level health** as a leading indicator; declining engagement at large accounts may warrant proactive outreach before churn sets in.

## Tools & Techniques

Python (pandas, NumPy, scikit-learn, matplotlib, seaborn) · Logistic Regression & Random Forest classification · Stratified cross-validation · Permutation importance analysis · Custom sliding-window adoption labeling
