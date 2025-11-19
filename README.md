# Adherence-Modeling-Capstone
### Goal:
Analyze user log and plan data to develop personas, define and calculate adherence, predict adherence, and come up with qualitative insights

### Authors:
- Karunya Iyappan
- Sam Knisely
- Alec Pixton
- Trenton Ribbens

## Summary of Implications & Next Steps
- Personas can guide **feature design**, **marketing**, and **persona-level strategies** (e.g., reminders, tailored goal suggestions).
- Future work:
  - Refine personas with more data.
  - Analyze how **initial plans** compare to **starting values** at onboarding.
  - Reassess **adherence over time** with larger samples to confirm stability or surface persona differences.

---

## Project Overview
This project analyzes user adherence to health goals, explores underlying structure with PCA, segments users with K-Means clustering, and models adherence using XGBoost. It also includes summary tables and figures for personas and model performance.

---

## Data Cleaning (Data_Cleaning.ipynb)
 - Removed plan data rows that were clear outliers. These rows had entries that were orders of magnitude above realistic goals.
 - Removed columns that were not helpful for analysis.

---

## Defining Adherence
Adherence compares user logs to the goals set in plan data.

**Key rules**
- Only evaluate a user on goals that are **defined** in their plan data.
- A goal is **defined** if its plan value is **non-null and non-zero**  
  *(exception: zero is valid when zero is a sensible goal, e.g., cigarettes or alcohol)*.
- **Adherence** = (# of log values that meet/exceed the goal) ÷ (# of defined goals).
  - Example: 4 goals defined, 2 met ⇒ **50% adherence**.
- What counts as “exceeding” depends on the feature:
  - **Lower is better** (e.g., sugar, alcohol): being **below** the goal counts as exceeding.
  - **Higher is better** (e.g., fruits/vegetables, exercise): being **above** the goal counts as exceeding.

---

## Identifying Key Features with PCA (PCA_Analysis.ipynb)
To reduce dimensionality and highlight structure, we applied Principal Component Analysis (PCA) to the cleaned dataset.

- The first **three principal components** explained ~**63%** of the variance.
- Interpretations:
  1. **PC1**: strong positive loadings for fitness, nutrition, and social goals → **overall health-plan engagement**.
  2. **PC2**: variation in **logging consistency**.
  3. **PC3**: separates users by tendency to **meet vs. exceed** goals.

---

## Clustering Analysis of Users with K-Means (Clustering_Analysis.ipynb)
We ran K-Means on category-level adherence to uncover user personas.

- Categories: **diet**, **lifestyle**, **exercise**, **supplements**.
- **Category adherence** computed using the definition above, then averaged per user.
- **Overall adherence** = average of the four category adherences.
- Best balance of distinctiveness and interpretability was achieved using **only the four category adherences** with **k = 3**.  
  Adding **total log entries** and **overall adherence** produced less distinct clusters.
- **Silhouette score**: **0.314** (range: −1 poor … +1 great).

### Adherence Metrics by Cluster
| Cluster | Diet  | Lifestyle | Exercise | Supplement | Overall |
|:--|--:|--:|--:|--:|--:|
| Light | 0.136 | 0.180 | 0.084 | 0.011 | 0.103 |
| Core  | 0.352 | 0.627 | 0.052 | 0.009 | 0.260 |
| Power | 0.485 | 0.670 | 0.249 | 0.628 | 0.508 |

### Entry and User Counts by Cluster
| Cluster | Log Entries | User Count |
|:--|--:|--:|
| Light | 13.20 | 20 |
| Core  | 28.14 | 14 |
| Power | 29.50 | 10 |

### Personas (Summary)
- **Light Users**: Low adherence across all categories; **far fewer** log entries than others.
- **Core Users**: Moderate adherence in **diet** and **lifestyle**; low overall; primarily track diet and key lifestyle variables (e.g., sleep, water).
- **Power Users**: Use **every** category; low adherence in **exercise**, moderate in **diet** and **supplements**, **high** in **lifestyle**; overall adherence is **~2×** Core Users.  
  Both Core and Power have **>2×** the log entries of Light Users.

From the time-series sample, adherence appeared **fairly stable** (no strong “start high then fade” pattern). As more data arrives, revisit adherence trends and differences by persona.

---

## Predicting Adherence with XGBoost (XGBoost_Model.ipynb)
We modeled adherence (proportion of goals achieved vs. goals set) using **XGBoost regression**.

**Why XGBoost**
- Gradient boosting over decision trees.
- Handles sparse data well.
- Learns optimal directions for splits when values are missing.

**Model setups tested**
1. **All predictors** (all plan + log variables)  
2. **Median-importance pruned** (drop features below median importance)  
3. **SHAP-pruned** (drop based on SHAP importance)

**Result:** The **All Predictors** model performed best and did **not** show notable overfitting under **5-fold cross-validation**.  
- Between folds: **R² SD ≈ 2 percentage points**, **RMSE SD ≈ 0.0038**.

### XGBoost Model Diagnostics
| Model | RMSE | R² |
|:--|--:|--:|
| All Predictors | 0.0322 | 92.95% |
| Median Importance Pruning | 0.0401 | 89.09% |
| SHAP Pruning | 0.0694 | 67.32% |

---
