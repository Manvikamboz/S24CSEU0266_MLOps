# SCSE3040 Machine Learning Operations (MLOps) --- Lab Practicals Comprehensive Solution Guide

**Student Name:** Manvi Kamboj  
**Enrollment No:** s24cseu0266  
**Course:** Machine Learning Operations (SCSE3040)  
**Program:** B.Tech CSE 5th Semester  
**Institution:** Bennett University  
**Session:** 2026-27  
**Repository:** [Bennett-MLOps-Lab/SCSE3040-Lab](https://github.com/Bennett-MLOps-Lab/SCSE3040-Lab)  

---

## 1. Executive Summary & Course Architecture

This repository hosts the practical lab curriculum for SCSE3040. The entire course revolves around building, packaging, serving, scaling, and monitoring a single running production project: a **Food Delivery-Time Predictor**.

### The 13-Practical MLOps Arc

$$\text{Notebook} \longrightarrow \text{Package} \longrightarrow \text{Tracked Experiments} \longrightarrow \text{REST API} \longrightarrow \text{Container} \longrightarrow \text{Cluster} \longrightarrow \text{Logging} \longrightarrow \text{CI/CD} \longrightarrow \text{Monitoring}$$

| Practical | Topic / Title | Core Objective | Status | Marks |
|---|---|---|---|---|
| **P01** | [Your MLOps Workbench](./P01-workbench/SOLUTION.md) | Virtual environments, pinned dependencies (`requirements.txt`), seeds, data hashing & Git setup | **COMPLETED** (7/7 Pass) | 10 |
| **P02** | [Your First Honest Model](./P02-first-model/SOLUTION.md) | Features vs. targets, 80/20 train/test split, naive mean/median baselines, Linear Regression, MAE/RMSE scoring, single-order inference | **COMPLETED** (9/9 Pass) | 10 |
| **P03** | Choosing a Model Honestly | Cross-validation, model comparisons, bias-variance tradeoff | *Upcoming* | 4 |
| **P04** | From Notebook to Package | Converting notebook code into modular Python packages (`src/`) | *Upcoming* | 4 |
| **P05** | Settings in a File, Bugs Caught by a Robot | Config management (`YAML`/`TOML`), automated testing (`pytest`) | *Upcoming* | 4 |
| **P06** | Remembering Every Experiment | Experiment tracking with MLflow (parameters, metrics, artifacts) | *Upcoming* | 4 |
| **P07** | Putting the Model Behind an Address | REST API development with FastAPI / Uvicorn | *Upcoming* | 4 |
| **P08** | Packing the Service into a Container | Docker containerization, Dockerfile optimization, image builds | *Upcoming* | 4 |
| **P09** | Describing the Service to a Cluster | Kubernetes deployment, services, and manifests (`kubectl`) | *Upcoming* | 4 |
| **P10** | Logs You Can Actually Search | Structured logging (`structlog`), centralized log management | *Upcoming* | 4 |
| **P11** | A Robot That Checks Your Work | Continuous Integration (CI) with GitHub Actions | *Upcoming* | 4 |
| **P12** | A Pipeline You Can Re-run | Orchestrated data & training pipelines (DVC / Airflow) | *Upcoming* | 4 |
| **P13** | Watching a Model Get Worse | Data drift, concept drift, model monitoring & alerting | *Upcoming* | 4 |

---

## 2. Environment Setup Guide

Follow these steps (as detailed in `SETUP.md`):

1. **Create Python Virtual Environment:**
   ```bash
   python3 -m venv .venv
   ```
2. **Install Pinned Dependencies:**
   ```bash
   .venv/bin/python -m pip install --upgrade pip
   .venv/bin/python -m pip install -r requirements-lock.txt
   .venv/bin/python -m pip install jupyterlab nbformat ipykernel httpx
   ```
3. **Register Jupyter Kernel:**
   ```bash
   .venv/bin/python -m ipykernel install --sys-prefix --name python3 --display-name "Python 3 (SCSE3040)"
   ```

---

## 3. Practical 01: Your MLOps Workbench

### Core Objectives
1. Verify active Python environment (`Inside .venv : True`).
2. Freeze exact dependency versions (`numpy`, `pandas`, `scikit-learn`).
3. Enforce deterministic randomness using fixed random seeds (`np.random.default_rng(SEED)`).
4. Hash datasets using SHA-256 to ensure data provenance and reproducibility.

### Walkthrough & Step Analysis
- **Step 0-1:** Inspect Python executable path.
- **Step 2-3:** Inspect package versions and export `requirements.txt`.
- **Step 4-5:** Prove that unseeded PRNG calls vary across runs while seeded PRNG calls repeat identically.
- **Step 6-7:** Generate 600 synthetic food delivery rows using seed `42` and hash dataset with `hashlib.sha256`.
- **Step 8-9:** Log run details to `run_info.json` and initialize Git tracking inside `work/`.

### Task Solutions

#### Task T1 --- Seed Manipulation (`seed=7`)
```python
T1_first_three = list(
    np.round(np.random.default_rng(7).uniform(0.5, 12.0, 600), 2)[:3]
)
# Result: [1.29, 9.47, 9.42]
```

#### Task T2 --- Dynamic Requirement Pining (`work/my_requirements.txt`)
```python
from importlib.metadata import version

MY_LIBS = ["numpy", "pandas", "scikit-learn"]
my_lines = "\n".join([f"{name}=={version(name)}" for name in MY_LIBS])
(WORK / "my_requirements.txt").write_text(my_lines, encoding="utf-8")
```

#### Task T3 --- Run Fingerprinting Function (`fingerprint(path)`)
```python
def fingerprint(path):
    df = pd.read_csv(path)
    return {"rows": len(df), "sha256": sha256_of(path), "seed": SEED}


T3_fp = fingerprint(DATA)
```

---

## 4. Practical 02: Your First Honest Model

### Core Objectives
1. Separate feature matrix ($X$) from target vector ($y$).
2. Perform honest 80/20 train/test dataset splitting.
3. Compute naive mean and median baseline scores to benchmark model performance.
4. Fit scikit-learn `LinearRegression` model and score using MAE and RMSE.
5. Extract learned coefficients to interpret real-world feature impact.
6. Deploy a single-instance prediction routine `predict_minutes(...)`.

### Walkthrough & Step Analysis
- **Step 1-2:** Load dataset (600 rows, 5 columns: `distance_km`, `prep_time_min`, `traffic_level`, `rain`, `delivery_min`).
- **Step 3:** Train/Test split (480 training rows, 120 testing rows, `random_state=42`).
- **Step 4:** Compute Mean Baseline MAE: $\approx 6.23$ minutes.
- **Step 5-6:** Fit Linear Regression model. Test MAE: $\approx 2.01$ minutes.
- **Step 7:** Learned feature weights:
  - Distance: $+3.10$ min / km
  - Prep Time: $+0.65$ min / min
  - Traffic: $+4.19$ min / level
  - Rain: $+5.52$ min
- **Step 8:** Predict single order ($5\text{ km}, 20\text{ min prep}, \text{traffic } 2, \text{no rain}$) $\rightarrow 42.9$ minutes.

### Task Solutions

#### Task T1 --- Median Baseline Benchmark
```python
median_pred = np.full(len(y_test), y_train.median())
T1_median_mae = mean_absolute_error(y_test, median_pred)
T1_which_is_better = "mean" if baseline_mae < T1_median_mae else "median"
# T1_median_mae ≈ 6.248 min, T1_which_is_better = "mean"
```

#### Task T2 --- Split Sensitivity Analysis (70/30 Split, Seed 7)
```python
X_tr2, X_te2, y_tr2, y_te2 = train_test_split(
    X, y, test_size=0.3, random_state=7
)
model2 = LinearRegression().fit(X_tr2, y_tr2)
T2_mae = mean_absolute_error(y_te2, model2.predict(X_te2))
# T2_mae ≈ 1.956 min
```

#### Task T3 --- Single Order Predictor (`predict_minutes`)
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
# T3_rainy_order = 34.9 min
```

---

## 5. Summary of Automated Self-Checks

```text
==================================================================
SELF-CHECK   Practical 01 --- Your MLOps Workbench
==================================================================
  [PASS]  T1 | T1_first_three holds three numbers
  [PASS]  T1 | those are the first three distances for seed 7
  [PASS]  T2 | work/my_requirements.txt exists
  [PASS]  T2 | it pins all three libraries with ==
  [PASS]  T3 | fingerprint() returns the three required keys
  [PASS]  T3 | it counts 600 data rows and uses seed 42
  [PASS]  T3 | the checksum matches the file on disk
------------------------------------------------------------------
  7 of 7 checks passed (10/10 Marks)

==================================================================
SELF-CHECK   Practical 02 --- Your First Honest Model
==================================================================
  [PASS]  T1 | T1_median_mae is the MAE of a median baseline
  [PASS]  T1 | T1_which_is_better names the lower-MAE baseline
  [PASS]  T2 | the new test set holds 30% of the orders
  [PASS]  T2 | model2 is a trained LinearRegression
  [PASS]  T2 | T2_mae is that model's MAE on the new test set
  [PASS]  T3 | predict_minutes returns a single number
  [PASS]  T3 | it agrees with the trained model
  [PASS]  T3 | rain makes the same order take longer
  [PASS]  T3 | T3_rainy_order is the rainy 3 km prediction
------------------------------------------------------------------
  9 of 9 checks passed (10/10 Marks)
```

---

## 6. Repository File Layout

```text
.
├── README.md                      # Main course overview
├── SETUP.md                       # Environment setup guide
├── COURSE_LAB_GUIDE.md            # Comprehensive assignment & task guide
├── requirements-lock.txt          # Locked dependencies
├── data/
│   └── delivery_times.csv         # Synthetic dataset (600 rows)
├── P01-workbench/
│   ├── P01.ipynb                  # Executed Practical 01 notebook
│   ├── README.md                  # P01 instructions
│   ├── SOLUTION.md                # Detailed P01 solution walkthrough
│   └── work/
│       ├── my_requirements.txt    # Generated pinned library requirements
│       ├── requirements.txt       # Base requirements
│       └── run_info.json          # Execution provenance JSON metadata
└── P02-first-model/
    ├── P02.ipynb                  # Executed Practical 02 notebook
    ├── README.md                  # P02 instructions
    └── SOLUTION.md                # Detailed P02 solution walkthrough
```
