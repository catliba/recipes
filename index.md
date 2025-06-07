# Recipes Project

Who doesn't want to eat healthy and to eat good?

---

## 1  Introduction

Who doesn't love food? Who doesn't love living longer? This project aims to solve both problems. Say goodbye to highly processed non nutritional artificial foods and say hello to a lifelong skill you are better off with than without - that is cooking. Not just cooking anything however. Cooking only the best, by general consesus and of course, the healthiest. Our goal is to predict whether a recipe is “healthy” using only its nutrition facts—and what patterns link nutritional content with popularity (user ratings)? Our dataset is from Food.com and here is a bit more information about the data we are working with:
| Column           | Description                                                                                                                                                                                              |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | Recipe name                                                                                                                                                                                              |
| `id`             | Recipe ID                                                                                                                                                                                                |
| `minutes`        | Minutes to prepare recipe                                                                                                                                                                                |
| `contributor_id` | User ID who submitted this recipe                                                                                                                                                                        |
| `submitted`      | Date recipe was submitted                                                                                                                                                                                |
| `tags`           | Food.com tags for recipe                                                                                                                                                                                 |
| `nutrition`      | Nutrition information in the form `[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]`; **PDV** stands for “percentage of daily value.” |
| `n_steps`        | Number of steps in recipe                                                                                                                                                                                |
| `steps`          | Text for recipe steps, in order                                                                                                                                                                          |
| `description`    | User‑provided description                                                                                                                                                                                |

There are 83782 unique recipes! We will be using primarily the following columns: `tags`, `nutrition`, `minutes`, and `submitted`.  In addition, we are using another dataset that contains user ratings for the recipes. 

---

## 2  Data Cleaning & Exploratory Data Analysis

After combining the average user rating for the recipes onto our main dataframe, we next parse the nutritional value and create more specified columns from it. Specficially, we have `[calories', 'total fat', 'sugar', 'sodium', 'protein', 'saturated fat', 'carbohydrates']` as our extra columns which we will be utilizing later on in our predictions. Next, I needed to determine which recipes are considered 'healthy.' The `tags` column helps categorize each recipe. Looking and parsing through each of the unique tags in the tags column, I've selected the following to be considered as a healthy recipe: `healthy_tags = ["vegan", "vegetarian", "gluten-free", "healthy", "healthy-2", "high-calcium", "high-fiber", 'high-in-something-diabetic-friendly', 'high-protein']`. Any recipe that contains one of these tags automatically get marked as healthy.I realize that this may not encapsulate all of the healthy recipes, but it is a good start to label our data. Next, we standardize the nutritional values, calculate the age of the recipe. After doing all that, I end with the follow dataframe:

<iframe
  src="imgs/Screenshot 2025-06-06 214441.png"
  width="800"
  height="600"
  frameborder="0"
></iframe>


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
