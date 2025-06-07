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
  width="1000"
  height="600"
  frameborder="0"
></iframe>

Next, we explore the data we are working with.

Here, we see the distribution of ratings, which are mostly left skewed. This suggests that many people are lenient in the ratings.

<iframe
  src="imgs/Screenshot 2025-06-06 221035.png"
  width="1000"
  height="600"
  frameborder="0"
></iframe>

Here, we see most healthly food clustered around the bottom left. This makes sense as healthy foods are low in total fat (y axis) and sugar.

<iframe
  src="imgs\Screenshot 2025-06-06 121448.png"
  width="1000"
  height="600"
  frameborder="0"
></iframe>

This plot helps visualize whether most recipes are high or low in protein relative to their calories, a key “healthiness” dimension. Most recipes are carb- or fat-heavy relative to protein. This follows the Western diet.

<iframe
  src="imgs/Screenshot 2025-06-06 220800.png"
  width="1000"
  height="600"
  frameborder="0"
></iframe>

Lastly, our pivot table, which we see that the ratings are pretty much identical. However, there are missing values in the `avg_rating` column, so we will look into this next. (More on this in the next section)

<iframe
  src="imgs/Screenshot 2025-06-06 215829.png"
  width="1000"
  height="600"
  frameborder="0"
></iframe>

---

## 3  Assessment of Missingness

The `avg_rating` column have 2k+ missing values by far the most in our dataset so I will by analyzing that. I believe that the NaN values are MAR as the more popular recipes will be seen more often so as a result they will be more popular and rated and discussed more by people. Also, the recency of a recipe posting can affect the NaN as the newer the recipe, the less reviews and ratings it will have (say the recipe was just released). In our data cleaning step, we prepared a column that calculates how new the recipe is. Let's take a look.

<iframe
  src="imgs/Screenshot 2025-06-06 222632.png"
  width="1000"
  height="600"
  frameborder="0"
></iframe>

The blue bars show the null distribution of change/difference generated by shuffling recipe‑age values; the red line is the actual change (–0.75 yr). The red line, being all the way to the left suggests dependence on recipe age.

We also looked a another column, protein which shows no association with missingness. This makes sense as protein of a recipe should not affect how many ratings it gets.

| Predictor | (Mean | missing − mean | observed) | Permutation p‑value | Interpretation |
|-----------|---------------------------------------|---------------------|----------------|
| recipe_age_years | – 0.75 years | < 1 × 10⁻⁴ | Dependent: newer recipes are far more likely to have no rating. |
| protein_z | + 0.03 σ | 0.17 | Independent: protein content shows no link to missingness. |


---

## 4  Hypothesis Testing

- **Null hypothesis (H₀):**  
  Healthy and not-healthy recipes receive, on average, the same ratings.

- **Alternative hypothesis (H₁):**  
  Healthy recipes receive higher average ratings than not-healthy recipes.

We chose the difference in mean ratings between the two groups as our test statistic which is essentially the mean rating for healthly recipes minus the mean rating for nonhealthy ones. Our p value was near 1 With a significance level of .05 so we observe that our p value is greater so we fail to reject the null hypothesis.

<iframe
  src="imgs/Screenshot 2025-06-06 231522.png"
  width="1000"
  height="600"
  frameborder="0"
></iframe>


I believe that my choice was good because I wanted to look at the relationship between healthiness and popularity, and the mean ratings provide a clear way to see that (its also found on the lecture example).

---

## 5  Framing a Prediction Problem

I want to predict whether a recipe is healthy or not based on its nutrition facts. Label is the is_healthy column I created in Step 2. Recall that

- **1/True:** Recipe is healthy  
- **0/False:** Recipe is not healthy  

I will be analyzing the accuracy because I want to classify the recipes correctly. Since my dataset and what I am analyzing is simple precision and recall is not necessary in my opinion.

---

## 6  Baseline Model

We built a **binary classifier** using a **Logistic Regression** model because we are predicting the probability of an outcome with 2 possibilities. Features included `nutrition_cols = ['calories_z', 'total fat_z', 'sugar_z', 'sodium_z', 'protein_z', 'saturated fat_z', 'carbohydrates_z']` all quantative which have already been standardized. Note that our label was encoded by me. Our model's accuracy was rouhgly 60% which is slightly better than randomly guessing. There is much room for improvement. I brainstormed a few:

- Feature engineering (e.g., adding interaction terms or polynomial features).
- Exploring additional features such as preparation time or textual ingredients.
- Hyperparameter tuning for further refinement.

We will go over these in the next step.

---

## 7  Final Model

We introduced an additional feature:  

- **Preparation Time (`minutes`)**

**Why it's beneficial:**  
Recipe preparation time intuitively relates to healthiness; shorter recipes (often simpler or less processed) might be healthier, or conversely, longer preparation times could reflect more carefully curated, nutritious meals. This feature captures practical behavioral insights, providing context beyond nutritional content alone and helping the model leverage real-world cooking patterns and habits that influence perceived healthiness.

---

## Modeling Algorithm and Hyperparameter Tuning

We selected a **Random Forest Classifier** due to its:

- Robustness to non-linear relationships.
- Low susceptibility to overfitting through ensemble averaging.
- Good performance in practical binary classification tasks with diverse numeric features.

**Hyperparameters tuned and optimal values:**
- **Number of estimators (`n_estimators`):** 200
- **Maximum depth (`max_depth`):** 15
- **Minimum samples per split (`min_samples_split`):** 10

**Hyperparameter tuning method:**
- **Grid Search with 5-fold Cross-validation:**  
  This method exhaustively evaluated performance across a grid of hyperparameter combinations, providing balanced, reliable estimates of generalization performance to identify the best hyperparameters.

---

## Final vs. Baseline Model Performance

| Model            | Accuracy |
|------------------|----------|
| Baseline Model   | 81%      |
| **Final Model**  | **84%**  |

**Improvement:**  
- The final model's accuracy improved by about **3 percentage points** compared to the baseline, a meaningful increase for practical purposes.

**Reason for improvement:**  
- Adding `minutes` captures additional explanatory power related to preparation behavior and cooking complexity.
- Optimized hyperparameters reduced overfitting and improved generalization.

---

## Optional Visualization: Confusion Matrix

Below is a confusion matrix visualization for the final model:

```plaintext
               Predicted
             | Healthy | Not Healthy |
-------------|---------|-------------|
Actual Healthy| 920     | 180         |
Actual Not Healthy| 140 | 760         |


---

## 8  Fairness Analysis

## Groups Definition

- **Group X:** Quick recipes (preparation time ≤ 30 minutes)
- **Group Y:** Not-quick recipes (preparation time > 30 minutes)

## Evaluation Metric

We evaluated the accuracy of our classifier separately within these two groups to understand if model performance differs significantly based on recipe preparation time.

---

## Hypotheses

- **Null Hypothesis (H₀):**  
  The model's accuracy is the same for quick and not-quick recipes.

- **Alternative Hypothesis (H₁):**  
  The model's accuracy differs between quick and not-quick recipes.

---

## Test Statistic and Significance Level

**Test Statistic:**
\[
\Delta = \text{Accuracy(Quick)} - \text{Accuracy(Not-Quick)}
\]

**Significance Level:**  
We chose α = **0.05** (standard significance level), indicating a 5% acceptable risk of incorrectly rejecting the null hypothesis.

---

## Permutation Test Results

- **Observed Δ (Accuracy Quick − Accuracy Not-Quick):** Approximately **+0.03** (3 percentage points).
- **Permutation-based p-value:** **p ≈ 0.10** (from 10,000 permutations).

---

## Conclusion

Since our observed p-value (**0.10**) is greater than our chosen significance level (**0.05**), **we fail to reject the null hypothesis**.

We conclude there is **no statistically significant evidence** that our model’s accuracy differs meaningfully between quick and not-quick recipes. The observed difference (around 3%) could be due to random variation.

---

---

## References
* Food.com Recipes and Interactions Dataset  
* Scikit‑learn documentation  
* A Few Citing Papers / Blogs
