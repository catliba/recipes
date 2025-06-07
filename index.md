# Healthy‑Recipes Project

A short tagline (one sentence).

---

## 1  Introduction
* **Motivation** – Why predict whether a recipe is “healthy”?  
* **Dataset** – Food.com recipes + interactions (2008 – 2018, ≈ 230 k recipes).  
* **Key takeaway** – \<one‑line headline of your findings\>.

---

## 2  Data Cleaning & Exploratory Data Analysis
* **Parsing “nutrition”**: converted 7‑item strings → numeric cols.  
* **Merged ratings**: `avg_rating` from interactions.  
* **EDA highlights**  
  * Histogram of calories (Fig 1) – right‑skewed, median ≈ 410 kcal.  
  * Histogram of avg ratings (Fig 2) – compressed between 4.2 – 4.8.  
* **Interpretation** – explain what shapes suggest about bias / next steps.

*(Insert two interactive Plotly histograms here.)*

---

## 3  Assessment of Missingness
* **Column examined**: `avg_rating`.  
* **Permutation test** → missingness depends on `recipe_age_years` (p < 0.001)  
  but **not** on `protein_PDV` (p ≈ 0.19).  
* **Conclusion** – missingness is MAR w.r.t. age → must account for age.

---

## 4  Hypothesis Testing
**H₀**: Healthy and not‑healthy recipes receive the same mean rating.  
**H₁**: Healthy recipes are rated higher.

* Test stat: Δ = mean(healthy) − mean(not).  
* 10 000‑shuffle permutation p ≈ 0.08 → fail to reject H₀.  
* **Interpretation** – no clear evidence that users prefer healthier dishes.

---

## 5  Framing a Prediction Problem
* **Task** – classify whether a recipe is “healthy” (tag‑based definition) using only nutrition facts.  
* **Target** – `is_healthy` (1/0).  
* **Features** – 7 raw %‑DV metrics + engineered sugar/protein density.

---

## 6  Baseline Model
* **Model** – logistic regression (linear).  
* **Preprocessing** – StandardScaler for numeric, One‑Hot for binary flag.  
* **Metrics (20 % test)** – accuracy = 0.71, ROC AUC = 0.77.  
* **Limitations** – ignores non‑linear interactions.

---

## 7  Final Model
* **Model** – GradientBoostingClassifier.  
* **New features** – `protein_per_100kcal`, `sugar_per_100kcal`; Quantile transform on heavy‑tailed ratios.  
* **Hyper‑parameter search** – `n_estimators`, `learning_rate`, `max_depth`, `subsample` (RandomizedSearchCV, 20 iters).  
* **Metrics (20 % test)** – accuracy = 0.79, ROC AUC = 0.86.  
* **Feature importance** – plot top contributors (protein density, sugar_PDV, calories).

*(Embed the importance bar chart and ROC curve.)*

---

## 8  Fairness Analysis
**Groups compared** – Quick (≤ 30 min) vs. Not‑quick (> 30 min) recipes.  
* **Metric** – accuracy.  
* Observed Δ = +0.012 (quick higher).  
* 10 000‑shuffle p = 0.18 → fail to reject H₀ ≈ no accuracy disparity.  
* **Caveat** – other axes (e.g., cuisine type) may warrant future analysis.

---

## References
* Food.com Recipes and Interactions Dataset  
* Scikit‑learn documentation  
* A Few Citing Papers / Blogs
