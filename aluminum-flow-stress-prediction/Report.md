# Capstone Final Report: Aluminum Plate Flow Stress Prediction

*AlumaTech’s plate stretching process requires accurate **flow stress** predictions to prevent equipment overload and ensure product quality. This project built a data-driven model to **predict flow stress** (and corresponding pull force) for new aluminum plate products **before production**, allowing engineers to assess feasibility and optimize operations in advance[1](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_preprocessing.html).*

**Key Model Metrics:**  
- **Historical Records Analyzed:** 486k+ plate stretches (2013–2023)[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html)  
- **Model Accuracy (R²):** ≈97.7% of variance in flow stress explained[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html)  
- **Avg Prediction Error (RMSE):** ~1.4k psi on test data[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html)  
- **Safety Coverage:** ~99% of cases where actual stress ≤ model’s worst-case prediction[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html)  

## 1. Introduction: Problem Statement and Context

AlumaTech lacked a reliable method to **forecast the pull forces** needed to stretch new aluminum plate configurations. Uncertain estimates led to two risks: **underestimation** risked machine overloading and damage, while **overestimation** caused inefficient scheduling and resource use[1](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_preprocessing.html). 

**Context:** Aluminum plates (often several inches thick and many feet long) are plastically stretched ~2% to relieve internal stresses. The required force (reflected by the plate’s flow stress) depends on alloy and geometry. Without accurate predictions, AlumaTech had to rely on trial runs or conservative assumptions. This hinders launching new plate products and can result in **equipment downtime** or **quality issues** if the process is not well-controlled[1](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_preprocessing.html).

**Objective:** Develop a model that, given a plate’s material and dimensions, **predicts the flow stress** needed. Success means AlumaTech can:
- **Evaluate new designs upfront** – determine if a proposed plate can be stretched with existing equipment (avoiding infeasible projects)[1](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_preprocessing.html).
- **Optimize scheduling and maintenance** – plan high-force jobs with proper precautions, reducing unplanned downtime.
- **Improve consistency** – apply the right force first time, improving yield and uniformity in plate properties[1](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_preprocessing.html).

**Stakeholders:** The **Engineering team** will use these predictions in product development and process planning. **Operations/Maintenance** will schedule stretches and plan preventive maintenance based on predicted forces. **Quality Assurance** will benefit from more consistent stretching parameters, leading to fewer defects[1](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_preprocessing.html). Ultimately, accurate predictions enable data-driven decisions in production and investment (e.g. knowing when a larger stretcher may be needed for future products).

## 2. Data Sources and Collection

We leveraged AlumaTech’s historical **stretch operation logs**. The dataset combines records from 2013–2023, totaling **486,396 stretch records after cleaning**[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html). Each record contains:
- **Alloy** – the aluminum alloy and temper code (categorical).
- **Thickness (Gauge)** – plate thickness in inches (numeric).
- **Width** – plate width in inches (numeric).
- **Length** – plate length in inches (numeric).
- **Stretch Target** – the intended elongation percent (e.g. 2.0%).
- **Max Stretch Force** – peak force applied (in pounds-force).
- **Flow Stress** – tensile stress corresponding to max force, in psi (Force ÷ cross-sectional area)[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html).
- **Timestamps and IDs** – date/time of stretch, job/lot identifiers, etc.
- **Other process data** – prestretch force, yield point, etc. (which occur during stretching).

These data were exported from AlumaTech’s systems (as CSV files per year) and then merged. Table 1 summarizes key features in the final modeling dataset:

| **Feature**              | **Description**                       | **Range / Values**                          |
|--------------------------|---------------------------------------|---------------------------------------------|
| **Alloy**                | Aluminum alloy code                   | 18 categories (e.g. XR7AT0K50, XVKKT7H51)[4](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/README.md) |
| **Thickness (in)**       | Plate thickness (gauge)               | 0.1 – 10.3 inches (mostly 2–4 in)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf)        |
| **Width (in)**           | Plate width                           | 12 – 105 inches (mostly 56–60 in)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf)      |
| **Length (in)**          | Plate length                          | 50 – 525 inches (varies widely)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf)       |
| **Stretch Target (%)**   | Intended elongation percentage        | 0.4% – 5.8% (typically ~2.0%)[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html)         |
| **Flow Stress (psi)**    | Resulting stress during stretch       | ~5,000 – 55,000 psi (bi-modal distribution)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf) |

*Table 1: Key features in the modeling dataset (post-cleaning). The **target** variable is flow stress.* 

This dataset covers a wide variety of plate sizes and alloy types that AlumaTech has processed, providing a strong basis for training a comprehensive model.

## 3. Data Preprocessing and Cleaning

Before analysis, we performed extensive **data cleaning and wrangling**:

- **Duplicate Removal:** Using a composite key (timestamp + job + lot + piece), we identified duplicate logs of the same stretch. We found 177 duplicates (e.g., a record repeated twice) and removed them, keeping the entry with the highest recorded force as the authoritative record[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html). After this, each stretch operation is unique (486,573 down to 486,396 records).

- **Invalid Data Handling:** We fixed or removed clearly erroneous values:
  - Entries with **zero values** for thickness or width (which is impossible) were dropped or corrected using alternate fields. For example, one stretch had `Width=0` in one column but a valid width in another field; we substituted the valid value[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html).
  - An alloy labeled “#CALC!” (a data entry error) was removed[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html).
  - After cleaning, no record had missing or nonsensical plate dimensions.

- **Outlier Removal:** We filtered out a few extreme outliers in **flow stress**. A handful of stretches showed flow stress above **60,000 psi**, which exceeds the strength limits of the alloys used and likely indicated bad data or exceptional conditions. We removed these (<0.01% of data) to prevent skewing the model[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf). The highest flow stress in the final set is ~55 ksi, consistent with known material capabilities.

- **Rare Alloy Filtering:** Alloys with very few records provide insufficient data to model. We dropped alloy categories with **<500 samples**, which eliminated 8 out of 26 alloys (mostly experimental or uncommon ones)[4](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/README.md). These contributed negligible data (<0.2%). The final dataset covers **18 alloys** — the core set used in production — each with at least several hundred records, and many with tens of thousands.

- **Feature Selection:** We retained features known **prior** to stretching for the model inputs: **alloy, thickness, width, length, stretch_target**. The **output** is flow stress. Other logged fields (prestretch force, yield point, etc.) were not used as they occur during the operation (not predictive). This yields a clean table of input features and target.

After cleaning, we had **486,396 records** with reliable data[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html). We then prepared the data for modeling:
- We split it into a **training set (80%)** and **test set (20%)**, stratifying by alloy so each alloy type appears proportionally in both sets[4](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/README.md).
- We set up a preprocessing pipeline to one-hot encode the **Alloy** (creating 18 binary dummy features) and standard-scale the numerical features (**Thickness, Width, Length, Stretch Target**)[4](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/README.md). This numeric scaling (zero mean, unit variance) helps some models (like regression) and doesn’t affect tree-based models.

The data was now ready for exploratory analysis and modeling.
## 4. Exploratory Data Analysis (EDA)

We analyzed the cleaned data to understand the relationships and inform our modeling:

- **Flow Stress Distribution:** The distribution of flow stress is **bimodal** (see **Figure 1**). We observe one large cluster of stretches with flow stress around **15–20 ksi**, and another around **35–45 ksi**[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf). This suggests two broad groups of conditions (likely reflecting different alloys). After removing a few outliers above 60 ksi, the range is roughly 5–55 ksi. The bimodal nature indicates a categorical factor (alloy) might be splitting the data into high vs low regimes.
  [Graph: **Figure 1 – Distribution of Flow Stress.** Histogram of flow stress (psi) for all operations, showing a lower peak ~18 ksi and an upper peak ~40 ksi.]

- **Influence of Alloy:** **Alloy type** is indeed the primary determinant of flow stress. Different alloys have vastly different strength characteristics. In the data, each alloy’s flow stress distribution is narrow, but positioned at a different level. For example, alloy *XVKKT7H55* (a high-strength alloy) has a median flow stress around 45 ksi, whereas *XR6AT0K61* (a softer alloy) centers around 15 ksi[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf). Figure 2 illustrates these differences by alloy. This confirms the model must incorporate alloy as a key input (e.g., via categorical encoding) to allow distinct predictions per alloy.

  [Graph: **Figure 2 – Flow Stress by Alloy.** Boxplots of flow stress for each alloy category. High-strength alloys (right side) require much higher flow stress than lower-strength alloys (left side).]

- **Influence of Thickness:** Within a given alloy, plate **thickness** (gauge) shows a mild effect on flow stress. Interestingly, thicker plates tended to require slightly *lower* flow stress for the same percent stretch in many cases. For instance, in one common alloy, 4-inch thick plates had ~2–3 ksi lower flow stress than 2-inch thick plates of the same alloy[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf). This could be due to metallurgical factors (thicker plates may have different temper or a more favorable stress state). Overall, the correlation between thickness and flow stress across all data was weak (Pearson r ≈ -0.11)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf), but within alloy subgroups the trend is noticeable enough that including thickness in the model adds accuracy.

  [Graph: **Figure 3 – Flow Stress vs. Thickness.** Example scatter for a single alloy, showing flow stress (psi) vs plate thickness. A slight downward trend indicates thicker plates need somewhat less stress to stretch for this alloy.]

- **Influence of Width and Length:** **Width** and **length** showed little independent correlation with flow stress. Width mainly scales the total force required but not the stress (since stress is normalized by cross-sectional area). The correlation of width with flow stress was near zero (r ≈ 0.18)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf). **Length** had a slight negative correlation (r ≈ -0.15)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf) — longer plates might have marginally lower flow stress, possibly due to strain distribution or machine grip factors. However, these effects are minor. We still include width and length in the model, allowing the model to learn any subtle effects or interactions (for example, extremely narrow plates might behave differently), but we expect their contribution to be small.

- **Influence of Stretch Target:** The majority of stretches targeted about **2.0% elongation** (this was the standard). A small subset had different targets (e.g. 1.5%, 2.5%, etc.)[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html). A higher stretch target can cause slightly higher flow stress due to material strain hardening. Our data showed a slight trend: stretches with 2.5% target ended with a bit higher flow stress than comparable plates stretched 2.0%. However, because target % rarely deviated, its overall correlation was low (r ≈ -0.26, influenced by the fact that some high-strength alloys used lower targets for safety)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf). We include stretch_target in the model to adjust predictions in those few cases where it differs from 2.0%.

- **Combined Effects:** A correlation heatmap of numeric features confirmed these observations: flow stress correlated strongly with **process outputs** like prestretch force (0.55) and yield point (0.55) – not available beforehand – and very weakly with thickness, width, length individually[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf). This highlights that **alloy (categorical)** is the major splitter (not captured in simple correlation) and that a multivariate model is needed to untangle the effects of alloy + dimensions together.

  [Graph: **Figure 4 – Correlation Heatmap (numeric features).** Flow stress has minimal direct correlation with thickness, width, or length individually[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf), indicating the need for a model that accounts for alloy categories and interactions.]

**EDA Summary:** The data revealed that **plate alloy** sets the baseline flow stress level, with **thickness** providing a secondary adjustment (thicker plates slightly easier to stretch in many cases). Width, length, and small changes in target percent have negligible influence. These insights guided our modeling: we ensured alloy is a central feature and anticipated largely linear effects for thickness, etc., with potential small interactions. The observations also set expectations for the model – e.g., we expected high accuracy because the process is quite deterministic per alloy and thickness.

## 5. Modeling Approach

Based on the problem and EDA findings, we developed two main types of models:
- A model to predict the **mean expected flow stress** for a plate (for typical planning).
- A model to predict a **conservative high-end flow stress** (to ensure safety margins).

**Features:** All models use the same input features: **Alloy, Thickness, Width, Length, Stretch Target**. Alloy was one-hot encoded into 18 binary features, and other features were numeric (scaled). The target is **flow stress** (psi).

We tried the following approaches:

**(a) Linear Regression (Baseline):** First, we fit a multiple linear regression model. This assumes flow stress = _constant_ + (coefficients × features). Given our EDA showed mostly linear patterns (within alloy groups), we expected a linear model to capture the bulk of variance. It’s also easily interpretable (each alloy gets its own intercept, and each feature a slope).

- We included the one-hot alloy variables to allow a different intercept for each alloy (essentially a separate baseline for each alloy).
- We tried adding regularization (Ridge, Lasso) to see if it improved generalization, but the best alpha for both was ~0 (no regularization), indicating the linear model was not overfitting and coefficients were stable[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).

**(b) Random Forest Regression:** To capture any non-linear interactions or complex dependencies, we trained a Random Forest ensemble. A random forest can automatically model interactions (e.g., it can learn different thickness effects for different alloys by splitting on alloy first in some trees). We tuned hyperparameters using cross-validation:
- Performed a randomized grid search over number of trees (50–200), max tree depth (10–30), min samples per split/leaf (2–10).
- The best model used ~150 trees, max depth ~30, and minimal leaf size ~2[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html). This configuration balances bias and variance given our large dataset.

**(c) LightGBM Quantile Regression:** To meet the need of **worst-case scenario prediction**, we trained a gradient boosting model (LightGBM) to predict the **99th percentile** of flow stress instead of the mean. This model is trained with a quantile loss (pinball loss at α=0.99) so that its predictions are higher than about 99% of the training examples. In other words, it aims to slightly overshoot the actual flow stress most of the time, giving an upper bound.
- We tuned LightGBM with random search (30 trials) varying number of leaves, learning rate, max depth, and estimators[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).
- The chosen model had ~300 trees, max depth 10, and learning rate ~0.05[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).

**Performance Metrics:** For the regression models (linear and RF), we used **Root Mean Squared Error (RMSE)** and **Mean Absolute Error (MAE)** to evaluate accuracy, as well as **R²** to see proportion of variance explained. RMSE was key since large errors (especially underestimates) are problematic. For the quantile model, we focused on **coverage** (did it achieve ~99% of actuals below predicted) and looked at the size of overestimation (MAE) rather than R².

**Train/Test Split:** We trained on 80% of data and reserved 20% for testing. All results reported are on the **test set**, which was not used in training or model tuning (ensuring a fair evaluation on unseen data).

## 6. Model Performance and Results

All models were evaluated on the test set of ~97k operations. **Table 2** compares their performance:

| **Model**                 | **Test RMSE (psi)** | **Test MAE (psi)** | **R² (Test)** | **Notes**                                  |
|---------------------------|--------------------:|-------------------:|-------------:|--------------------------------------------|
| Linear Regression (OLS)   | 1,445               | 937                | 0.9756[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html) | Baseline linear fit (alloy dummies + values) |
| Ridge Regression (α=0.001)| 1,445               | 937                | 0.9756[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html) | Same as OLS (found virtually no regularization) |
| Lasso Regression (α=0.001)| 1,445               | 937                | 0.9756[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html) | Same as OLS (no meaningful regularization) |
| **Random Forest (tuned)** | **1,387**           | **903**            | **0.9775**[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html) | Non-linear model, slight error reduction     |
| **LightGBM (99% Quantile)** | 4,882               | 3,837              | ~0.72 (pseudo)[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html) | Predicts 99th percentile (intentional high bias) |

*Table 2: Model performance on the test set. The forest model gives the best mean prediction. The quantile model’s errors appear large because it deliberately over-predicts to cover worst cases.* 

**Linear Regression:** Achieved **R² = 0.9756**, explaining ~97.6% of variance in flow stress on test data[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html). Its RMSE (~1445 psi) is only ~3-4% of typical stress values – meaning on average it predicts within ~±1.4 ksi. The MAE was 937 psi[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html), indicating half the time it’s within about 0.94 ksi. This high accuracy from a simple model confirms our intuition: given alloy and dimensions, flow stress is largely a linear function. The linear model’s coefficients were interpretable:
- Each alloy dummy had a strong effect (e.g., hard alloys had large positive intercept adjustments).
- The thickness coefficient was small negative (reflecting the slight decrease in stress for thicker plates).
- Width and length coefficients were near zero.
- Stretch target had a small positive coefficient (since higher % needed slightly more stress).

No alloy’s behavior was completely anomalous; hence the linear model did well. However, a linear model **predicts the average** outcome for each combination. In practice, about half of actual points will be above the linear prediction. We needed a way to guard against those cases (enter the quantile model).

**Random Forest:** The random forest improved slightly over linear, with **R² = 0.9775** and RMSE ~1387 psi[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html). This ~4% reduction in RMSE indicates the forest captured some small non-linear patterns that linear regression missed. For example, the forest might have learned interactions like “For alloy X, very thick plates reach a minimum required stress and don’t drop further” or other subtle trends. The MAE (~903 psi) also improved[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).

Feature importance in the RF model underlined:
- **Alloy** was by far the most important factor (the first splits in many trees were on alloy).
- **Thickness** was the most important numeric feature (the forest consistently used gauge to fine-tune predictions within alloy groups)[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).
- **Stretch Target** had some importance (trees split on target% when it deviated from 2.0).
- **Width and Length** had negligible importance (confirming they seldom affected splits).

In practice, the RF predictions were very close to the linear model’s (since the problem is mostly linear), but with slightly reduced error variance. This makes the RF our best model for **predicting the expected flow stress**.

**LightGBM 99% Quantile:** This model is not judged by R² (which is ~0.72 here) because it’s not trying to fit the average[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html). Instead, it’s tuned to overshoot. On the test set, the quantile model’s predictions were higher than the actual flow stress **about 98-99% of the time**, achieving near the desired coverage. In only ~1-2% of cases did actual slightly exceed prediction, and by small margins (a few hundred psi). As a trade-off, it **overestimates by ~3.8 ksi on average** (MAE)[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html). For example, if the true required stress is 40 ksi, the 99% model might predict ~44 ksi. This model is useful as a *safety bound*: we can be ~99% confident the actual requirement will not exceed its prediction.

The benefit is clear in preventing underestimation. The cost is that its predictions are conservative. However, for AlumaTech’s purposes, it’s better to plan for a slightly higher force than needed than to be caught under-prepared. We consider this model a success in meeting the safety criterion.

In summary, the **Random Forest** gives an excellent point estimate, and the **Quantile LightGBM** gives a high-confidence ceiling. Both models align with the observed data patterns and passed the test-set validation, indicating they generalize well.

## 7. Key Findings and Interpretation

The modeling results lead to several **key findings**:

- **Flow stress can be predicted with high accuracy.** With R² ~0.98, our model explains nearly all the variability in flow stress on unseen plates[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html). This means the stretching process behaves consistently according to the input parameters; there is little random noise. The small test errors (~3% of value) show that data-driven predictions are feasible and reliable for operational use.

- **Alloy and thickness are the dominant drivers.** The model confirmed that alloy differences account for the largest variation in required flow stress. Each alloy effectively has its own baseline flow stress curve. Thickness is the next contributor: within each alloy, a thicker plate tends to lower the stress needed (though modestly). Other factors (width, length) were practically negligible. This finding emphasizes that knowing the material properties (alloy) and thickness is sufficient to estimate forces – which aligns with metallurgical knowledge.

- **The process is largely linear and additive.** The success of the linear model suggests there aren’t strong complex interactions beyond what we expected. This boosts confidence that the model will behave predictably even for intermediate values. It also means we can interpret the model: e.g., switching from Alloy A to Alloy B might always add ~10 ksi to the required stress regardless of dimensions, etc. The random forest captured a few non-linear quirks (maybe small plateau effects or interactions), but those only marginally improved accuracy.

- **Worst-case scenarios are now quantifiable.** Perhaps the most important outcome: we now have a way to predict not just the expected force, but a **conservative upper bound** for the force. The 99th percentile model provides a stress value that almost no actual stretch will exceed. This addresses AlumaTech’s primary safety concern. Previously, if a new plate was proposed, the uncertainty might force heavy safety margins or risky guesswork. Now, we can say for example, “We expect 38 ksi, but to be safe design for up to 42 ksi.” This separates the concerns of *optimal setting* vs. *worst-case planning*.

- **Data quantity and diversity paid off.** The huge dataset (nearly half a million records) ensured the model is well-trained. We see no sign of overfitting (test error ~ train error) and the model performs well for all alloy types, even those less common, due to stratified splitting and enough examples of each. The data included extreme cases (very thick plates, very high-strength alloys), which means the model can handle a wide range of scenarios. This wide coverage is crucial for deployment – it means for most new inquiries, the conditions will fall within the range the model has seen and learned.

Overall, the project validated that **historical process data can effectively drive predictive modeling** for industrial operations. AlumaTech’s stretching process follows physical laws that our model captured through the proxy of alloy and geometry features. With this model, the company can essentially simulate a stretch in silico and get results instantly, rather than physically testing or relying on heuristics.

## 8. Recommendations for the Client

Based on our findings, we provide the following **recommendations** to AlumaTech on using the model and insights:

1. **Integrate Model into New Product Design Reviews:** Make it standard practice that when a new plate product (new alloy/size) is proposed, the engineering team runs it through the prediction model. If the model indicates a required flow stress near or beyond current equipment limits, **redesign or special planning** is needed. For example, if an alloy-temper combination is predicted to need 60 ksi but the stretcher max is 55 ksi, engineers should adjust the design (thinner plate, different temper) or plan to outsource/upgrade equipment. This feasibility check will save costly trial-and-error and prevent scheduling jobs that cannot be safely executed[1](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_preprocessing.html).

2. **Use Worst-Case Predictions for Scheduling and Maintenance:** Incorporate the model’s **99th percentile prediction** into production scheduling. Identify stretches that have exceptionally high predicted stresses and schedule them strategically (e.g., not all in one shift, avoid stacking them back-to-back). Ensure maintenance personnel know when a high-stress job is coming – they might perform a pre-job inspection or ensure the machine is at optimal condition (proper hydraulic pressure, no ongoing issues). By doing so, AlumaTech can **avoid unexpected breakdowns** and extend equipment life, as the heaviest loads are anticipated and managed.

3. **Optimize Stretch Parameters to Improve Quality:** The model enables a deeper understanding of the process. Quality engineers can analyze cases where actual outcomes deviated from predictions. If some alloy consistently requires slightly more force than expected, it could indicate material property variance – perhaps adjust heat treatment or supplier specs. Moreover, knowing the exact expected stress for each plate, operators can tune the process (e.g., adjust the **stretch speed or gripping method** for plates that need very high stress, to apply force more gradually and uniformly). This will result in **more consistent elongation** and reduce residual stress differences plate-to-plate. Essentially, use the model as a guide for fine-tuning and standardizing the process for each product family.

Beyond immediate use, we recommend AlumaTech engages in **training** sessions for engineers and ops teams on the new tool, so they trust and understand the predictions. Highlight past examples: show that the model would have accurately predicted known tough cases from the history, building confidence in its reliability.

## 9. Ideas for Further Research

To continuously improve and expand the utility of the model, AlumaTech should consider these future steps:

- **Additional Input Factors:** Investigate if incorporating other known variables (if available) could further improve accuracy. For instance, **plate temperature** during stretching, or time since heat treatment (aging), might affect required force. In our data, these were not explicitly tracked. Conducting controlled trials or capturing these process parameters in future can enrich the model. If, say, warm plates stretch easier, adding temperature as an input could adjust predictions for cases where heating is applied.

- **Alloy-Specific Models:** Our global model works well across alloys, but a potential enhancement is to train **specialized sub-models for each alloy group**. This could capture nuances (like slight non-linearity in thickness effect unique to an alloy) more precisely. With so much data per major alloy, a dedicated model might squeeze out a bit more accuracy. We could also explore a hierarchical model (multi-level) that shares information across alloys but allows individual tuning.

- **Physics-Based Model Integration:** Combining this data-driven approach with **physical modeling** could be fruitful. For example, using known stress-strain curves of alloys to inform the model structure, or validating model predictions against finite element analysis (FEA) of the stretching process for extreme cases. A hybrid model could improve confidence especially for extrapolation beyond the observed data range (e.g., a slightly thicker plate than seen before – our model would predict, but FEA could verify if it’s reasonable).

- **Periodic Model Retraining:** The model should be kept up-to-date as new data comes in. The aluminum stretching process or material supply might change over time (new alloys, machine wear, etc.). We recommend establishing a schedule to **retrain the model (e.g., annually)** with the latest stretches. This way, the model will capture any gradual shifts or expansions in operating conditions. Our pipeline can be automated so new records are appended and the model retrains and validates with minimal human intervention.

- **User-Friendly Deployment:** To maximize usage, the model should be accessible via a simple tool. For instance, a **web interface or Excel plugin** where a user inputs Alloy, Thickness, Width, Length, and out comes the predicted stress (expected and worst-case). This removes any friction in using the model. Additionally, integrate the model into the existing MES (Manufacturing Execution System) so that every scheduled job is automatically accompanied by a prediction, flagging any that are unusual.

By pursuing these steps, AlumaTech can ensure the model remains accurate, grows with the business, and is fully adopted on the shop floor and in design reviews. Essentially, the model should become an integral part of the stretching operation’s knowledge base, continuously improved and always consulted.

## 10. Code Finalization and Deliverables

**Finalized Code:** All analysis and modeling code is consolidated in well-documented Jupyter notebooks:
- *Data Wrangling & EDA Notebook* – Shows data cleaning steps (merging files, removing outliers/duplicates) and exploratory charts[2](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_modeling.html)[5](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/Project%20Proposal.pdf).
- *Modeling Notebook* – Shows preprocessing, model training, hyperparameter search, and evaluation on train/test sets[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).
- The notebooks are written with explanatory markdown so that colleagues can understand the workflow. They have been cleaned (removing extraneous code, fixing formatting) for readability.

**Model Artifacts:** We saved the trained models for future use:
- A **preprocessing pipeline object** (`preprocessor.pkl`) that applies the same scaling and encoding as during training[4](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/README.md).
- The final **Random Forest model** (`best_random_forest_model.pkl`) for mean prediction and the **LightGBM model** (`model_99th_percentile.pkl`) for high-percentile prediction[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).
These can be loaded in a Python environment to make predictions on new data instantly.

**Model Metrics File:** We prepared a CSV file summarizing the final model details:
- It lists each model (Linear, Ridge, RF, LightGBM 99%) with columns for features used, key hyperparameters, and performance metrics (RMSE, MAE, R² on test data)[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).
- This provides a concise reference. For example, it shows the RF used 150 trees, depth 30, and achieved R² 0.9775 on test[3](https://coffmaneng-my.sharepoint.com/personal/kevin_yeoman_coffman_com/Documents/Microsoft%20Copilot%20Chat%20Files/capstone2_AlumaTech_EDA.html).
This file can be included in documentation or for audit of the model’s performance.

**Documentation:** In addition to this written report, we created:
- A **Project README** (Markdown) summarizing the project goals, approach, and results in brief, for the repository.
- An **Executive Slide Deck** highlighting the problem, methodology, key findings, and recommendations (with visuals). This is intended for presenting to stakeholders who want a high-level overview.
- Both the README and slides draw from the analysis and include visuals (graphs from EDA, etc.) to communicate findings clearly.

All code and documentation have been uploaded to AlumaTech’s internal project repository. The next steps would be to integrate the model into production and train relevant staff in its use, as described in the recommendations. With these deliverables, AlumaTech has not only a functioning predictive model but also the means to maintain and understand it moving forward.
