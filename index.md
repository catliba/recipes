# Recipes Project

Who doesn't want to eat healthy and to eat good?

---

## 1  Introduction

Who doesn't love food? Who doesn't love living longer? This project aims to solve both problems. Say goodbye to highly processed non nutritional artificial foods and say hello to a lifelong skill you are better off with than without - that is cooking. Not just cooking anything however. Cooking only the best, by general consesus and of course, the healthiest. Our goal is to predict whether a recipe is “healthy” using only its nutrition facts—and what patterns link nutritional content with popularity (user ratings)? Our dataset is from Food.com and here is a bit more information about the data we are working with:

| Column           | Description                                                                                                                                                                        |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | Name of the recipe.                                                                                                                                                                |
| `id`             | Unique identifier for the recipe.                                                                                                                                                  |
| `minutes`        | Number of minutes required to prepare the recipe.                                                                                                                                  |
| `contributor_id` | User ID of the person who submitted the recipe.                                                                                                                                    |
| `submitted`      | Date the recipe was submitted to Food.com.                                                                                                                                         |
| `tags`           | List of tags describing the recipe (e.g., “vegan”, “healthy”, “gluten-free”).                                                                                                      |
| `nutrition`      | Nutrition facts as a list: `[calories, total fat (%DV), sugar (%DV), sodium (%DV), protein (%DV), saturated fat (%DV), carbohydrates (%DV)]`. **%DV** means “percent daily value.” |
| `n_steps`        | Number of steps in the recipe instructions.                                                                                                                                        |
| `steps`          | List of the recipe preparation steps, in order.                                                                                                                                    |
| `description`    | User-provided description of the recipe.                                                                                                                                           |

| Column      | Description         |
| ----------- | ------------------- |
| `user_id`   | User ID             |
| `recipe_id` | Recipe ID           |
| `date`      | Date of interaction |
| `rating`    | Rating given        |
| `review`    | Review text         |


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

Next, we explore the data we are working with.

Here, we see the distribution of ratings, which are mostly left skewed. This suggests that many people are lenient in the ratings.

<iframe
  src="imgs/Screenshot 2025-06-06 221035.png"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Here, we see most healthly food clustered around the bottom left. This makes sense as healthy foods are low in total fat (y axis) and sugar.

<iframe
  src="imgs\Screenshot 2025-06-06 121448.png"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot helps visualize whether most recipes are high or low in protein relative to their calories, a key “healthiness” dimension. Most recipes are carb- or fat-heavy relative to protein. This follows the Western diet.

<iframe
  src="imgs/Screenshot 2025-06-06 220800.png"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Lastly, our pivot table, which we see that the ratings are pretty much identical. However, there are missing values in the `avg_rating` column, so we will look into this next. (More on this in the next section)

<iframe
  src="imgs/Screenshot 2025-06-06 215829.png"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## 3  Assessment of Missingness

The `avg_rating` column have 2k+ missing values by far the most in our dataset so I will by analyzing that. I believe that the NaN values are MAR as the more popular recipes will be seen more often so as a result they will be more popular and rated and discussed more by people. Also, the recency of a recipe posting can affect the NaN as the newer the recipe, the less reviews and ratings it will have (say the recipe was just released). In our data cleaning step, we prepared a column that calculates how new the recipe is. Let's take a look.

<iframe
  src="imgs/Screenshot 2025-06-06 222632.png"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The blue bars show the null distribution of Δ generated by shuffling recipe‑age values; the red line is the actual Δ (–0.75 yr). The red line, being all the way to the left suggests dependence on recipe age.

We also looked a another column, protein which shows no association with missingness. This makes sense as protein of a recipe should not affect how many ratings it gets.

| Predictor | Δ (mean | missing − mean | observed) | Permutation p‑value | Interpretation |
|-----------|---------------------------------------|---------------------|----------------|
| recipe_age_years | – 0.75 years | < 1 × 10⁻⁴ | Dependent: newer recipes are far more likely to have no rating. |
| protein_z | + 0.03 σ | 0.17 | Independent: protein content shows no link to missingness. |


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
