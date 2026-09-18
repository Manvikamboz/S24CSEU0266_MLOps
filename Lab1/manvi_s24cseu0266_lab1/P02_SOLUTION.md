# Practical 02 Solution Guide --- Your First Honest Model

**Student Name:** Manvi Kamboj  
**Enrollment No:** s24cseu0266  
**Course:** SCSE3040 Machine Learning Operations  
**Institution:** Bennett University  
**Session:** 2026-27  
**Module:** P02-first-model  

---

## 1. Overview & Core MLOps Concepts

**Practical 02** transitions from environment setup to building and evaluating a machine learning model honestly. Key concepts include:

1. **Features ($X$) vs. Target ($y$):**
   - Features ($X$): Input signals available prior to prediction (`distance_km`, `prep_time_min`, `traffic_level`, `rain`).
   - Target ($y$): Outcome variable to be predicted (`delivery_min`).
2. **Train/Test Splitting (`train_test_split`):**
   - Evaluates generalization performance on unseen test data, avoiding data leakage and overfitting.
3. **Establishing Baselines:**
   - A model's error metric is meaningless without context. A naive baseline (e.g. predicting the historical mean or median delivery time for every order) provides the minimum benchmark that any machine learning model must outperform.
4. **Evaluation Metrics (MAE vs RMSE):**
   - **MAE (Mean Absolute Error):** Measures average absolute magnitude of errors in interpretable units (minutes):
     $$\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |y_i - \hat{y}_i|$$
   - **RMSE (Root Mean Squared Error):** Penalizes larger errors more heavily:
     $$\text{RMSE} = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2}$$
5. **Single-Instance Inference:**
   - Packaging trained model parameters into a prediction function to serve real-time predictions for individual orders.

---

## 2. Detailed Walkthrough & Step-by-Step Explanation

### Step 1 & 2: Dataset Loading & Feature Selection
- Load `delivery_times.csv` (600 rows, 5 columns).
- Define $X = [\text{distance\_km}, \text{prep\_time\_min}, \text{traffic\_level}, \text{rain}]$ and $y = \text{delivery\_min}$.

### Step 3: Train/Test Split (80/20, Seed 42)
- Split into `X_train` (480 samples), `X_test` (120 samples), `y_train` (480 labels), `y_test` (120 labels).

### Step 4: Naive Mean Baseline Benchmark
- Compute `baseline_pred = y_train.mean()` ($\approx 32.55$ minutes).
- Measure `baseline_mae` on `y_test`: $\approx 6.23$ minutes.

### Step 5 & 6: Training & Scoring Linear Regression Model
- Fit `LinearRegression()` on `X_train` and `y_train`.
- Predict on `X_test` and calculate model MAE: $\approx 2.01$ minutes.
- **Improvement:** The model reduces prediction error by $4.22$ minutes ($\sim 67.7\%$ error reduction compared to guessing the average).

### Step 7: Model Coefficient Interpretability
- Extracted linear coefficients:
  - Base Intercept: $\sim 6.0$ minutes
  - `distance_km`: $+3.10$ min / km
  - `prep_time_min`: $+0.65$ min / prep minute
  - `traffic_level`: $+4.19$ min / traffic step
  - `rain`: $+5.52$ min if raining

### Step 8: Single Order Inference
- Pass sample input (5.0 km, 20 min prep, traffic level 2, no rain) $\rightarrow$ Output predicted time: $\sim 42.9$ minutes.

---

## 3. Practical Tasks & Code Implementations

### Task T1 --- Work Out the Baseline Yourself

- **Task Requirement:** Compute a median baseline (`y_train.median()`), score its MAE on `y_test` as `T1_median_mae`, and store `"mean"` or `"median"` in `T1_which_is_better` depending on which yields lower MAE.
- **Python Implementation:**
```python
# T1 solution: predict median for every test order
median_val = y_train.median()
median_pred = np.full(len(y_test), median_val)
T1_median_mae = mean_absolute_error(y_test, median_pred)

# Compare with mean baseline MAE
T1_which_is_better = "mean" if baseline_mae < T1_median_mae else "median"

print("median baseline MAE:", T1_median_mae)
print("better baseline    :", T1_which_is_better)
```
- **Output:**
  - `T1_median_mae`: $\approx 6.248$ minutes
  - `T1_which_is_better`: `"mean"` (mean MAE $\approx 6.234$ min vs median MAE $\approx 6.248$ min)

---

### Task T2 --- Does the Split Change the Answer?

- **Task Requirement:** Split dataset into 70% train / 30% test with `random_state=7`, fit a fresh `LinearRegression` model (`model2`), and calculate test MAE (`T2_mae`).
- **Python Implementation:**
```python
# T2 solution: 70/30 split with random_state=7
X_tr2, X_te2, y_tr2, y_te2 = train_test_split(
    X, y, test_size=0.3, random_state=7
)

model2 = LinearRegression()
model2.fit(X_tr2, y_tr2)

T2_mae = mean_absolute_error(y_te2, model2.predict(X_te2))
print("70/30 split, seed 7 -> MAE", T2_mae)
```
- **Output:**
  - `T2_mae`: $\approx 1.956$ minutes
  - **Insight:** MAE remains consistent ($\sim 1.96$ vs $\sim 2.01$), confirming model stability across different dataset splits.

---

### Task T3 --- One Order In, One Number Out

- **Task Requirement:** Implement `predict_minutes(distance_km, prep_time_min, traffic_level, rain)` returning single rounded delivery time prediction. Predict for 3 km, 15 min prep, traffic level 1, in the rain (`T3_rainy_order`).
- **Python Implementation:**
```python
def predict_minutes(distance_km, prep_time_min, traffic_level, rain):
    df_single = pd.DataFrame(
        [
            {
                "distance_km": distance_km,
                "prep_time_min": prep_time_min,
                "traffic_level": traffic_level,
                "rain": rain,
            }
        ]
    )
    pred = model.predict(df_single)[0]
    return round(float(pred), 1)


T3_rainy_order = predict_minutes(3.0, 15, 1, 1)
print("rainy 3 km order ->", T3_rainy_order, "minutes")
```
- **Output:**
  - `T3_rainy_order`: `34.9` minutes

---

## 4. Self-Check Verification Results

| Check Label | Result | Description |
|---|---|---|
| `T1` | **PASS** | `T1_median_mae` correctly measures median baseline MAE |
| `T1` | **PASS** | `T1_which_is_better` correctly identifies baseline with lower MAE (`"mean"`) |
| `T2` | **PASS** | New test set contains exactly 30% (180) of the orders |
| `T2` | **PASS** | `model2` is a trained LinearRegression model |
| `T2` | **PASS** | `T2_mae` accurately measures `model2` MAE on new test split |
| `T3` | **PASS** | `predict_minutes()` returns a single float value |
| `T3` | **PASS** | Predictions match trained scikit-learn model outputs |
| `T3` | **PASS** | Confirms rain flag increases predicted delivery time |
| `T3` | **PASS** | `T3_rainy_order` correctly holds 3 km rainy prediction |

**Status:** `9 of 9 checks passed (10/10 marks)`

---

## 5. Final Submission Statement

> **Model Performance Summary:**  
> The trained Linear Regression model achieved a test MAE of **2.01 minutes**, dramatically outperforming both the naive mean baseline (**6.23 minutes**) and median baseline (**6.25 minutes**) by **4.22 minutes**, representing a **67.7% reduction in prediction error**.
