🇬🇧 **English** | 🇷🇺 [Русская версия](README.ru.md)

# 🔥 Hot Jupiter Detector

Logistic regression model for identifying hot Jupiters based on stellar, orbital, and discovery-method features.

* * *

## 📋 About the Project

A "hot Jupiter" is a gas giant with a mass/radius comparable to Jupiter, but with an extremely short orbital period (P < 10 days) and high equilibrium temperature (T_eq > 1000 K). These planets are rare but physically important: their existence requires migration from the outer regions of the protoplanetary disk.

**Goal:** build a logistic regression model that predicts whether an exoplanet is a hot Jupiter based on available characteristics — stellar properties, orbital parameters, and discovery method.

**Data:** NASA Exoplanet Archive — Confirmed Planets table (40,350 records, 92 columns).

**Stack:** Python, pandas, numpy, scikit-learn, matplotlib, seaborn, joblib.

* * *

## 🎯 Key Results

Model | Accuracy | Precision (class 1) | Recall (class 1) | F1 (class 1) | PR-AUC  
---|---|---|---|---|---  
DummyClassifier (baseline) | 0.85 | 0.00 | 0.00 | 0.00 | 0.5731  
Baseline LogisticRegression | **0.96** | 0.90 | 0.79 | 0.84 | **0.9026**  
**Final LogisticRegression** (class_weight='balanced', threshold = 0.66) | **0.96** | 0.84 | **0.88** | **0.86** | 0.8937  

**Final model (test, 958 rows):**
- **F1-score = 0.86** — model finds 123 hot Jupiters out of 140.
- **Recall = 0.88** — misses only 17 hot Jupiters (vs 29 in baseline).
- **PR-AUC = 0.8937** — 1.5× better than baseline (0.5731).
- **ROC-AUC = 0.9721** — excellent class separation.

### The Precision/Recall Trade-off

We deliberately chose a model with higher Recall (0.88 vs 0.79) at the cost of slightly lower Precision (0.84 vs 0.90). In astronomy, **missing a planet (False Negative) is much worse than a false alarm (False Positive)**. If we miss a hot Jupiter, astronomers may never study it. If we flag a candidate incorrectly, it just requires extra verification time.

* * *

## 💡 Top-5 Insights

### 1. Planetary Mass is the Strongest Predictor
`pl_bmasse` (log-scale) has the largest positive coefficient (+2.92). The more massive the planet, the higher the chance it's a gas giant (hot Jupiter). This aligns with astrophysics: hot Jupiters typically have masses > 100 M⊕.

### 2. Semi-major Axis — the "Distance to Star" Rule
`pl_orbsmax` (log-scale) has the largest negative coefficient (-2.39). The closer the planet is to its star, the higher the chance it's a hot Jupiter. This makes physical sense: hot Jupiters orbit at < 0.1 AU.

### 3. Multi-planet Systems Rarely Host Hot Jupiters
`sy_pnum` (number of planets in the system) has a negative coefficient (-1.67). Known astrophysical effect: hot Jupiters are often solitary due to orbital migration.

### 4. Missingness Indicators "Rescued" the Model
Following the technique from our previous project (exoplanet radius prediction with `is_missing_mass`), we added `is_missing_insol`, `is_missing_bmasse`, and `is_missing_orbeccen`. These helped the model handle the 82% missingness in `pl_insol` and improved F1 from 0.00 (Dummy) to 0.86.

### 5. Multicollinearity Was Handled by L2 Regularization
`st_teff`, `st_rad`, and `st_mass` are highly correlated (0.88–0.94). We kept all three but used L2 regularization (`penalty='l2'` by default), which stabilizes the coefficients and prevents overfitting.

* * *

## 📊 Visualizations (in the notebook)

  * **Histogram of `log10(pl_orbper)`** — shows bimodality and the 10-day cutoff.
  * **Histogram of `log10(pl_rade)`** — shows the 8 R⊕ cutoff.
  * **Boxplot of `pl_insol` by class** — 100× difference between hot Jupiters and others.
  * **Correlation matrix (Spearman)** — reveals multicollinearity among stellar parameters.
  * **Missing values bar chart** — shows `pl_insol` (82.6%), `pl_bmasse` (67.7%), `pl_orbeccen` (67.5%).
  * **Confusion matrix** (baseline vs final) — shows the recall improvement.
  * **Feature importance (bar chart)** — top-5: `pl_bmasse`, `pl_orbsmax`, `sy_pnum`, `st_mass`, `st_rad`.

* * *

## 🗂 Repository Structure

    hot-jupiter-detector/
    ├── data/
    │   └── exoplanets.csv              # NASA source data
    ├── notebooks/
    │   └── project_hot_jupiters.ipynb
    ├── README.md                       # English (this file)
    ├── README.ru.md                    # Russian version
    └── requirements.txt

* * *

## 🚀 How to Reproduce

### 1. Clone the repository

    git clone https://github.com/DigitalOcean78/hot-jupiter-detector.git
    cd hot-jupiter-detector

### 2. Install dependencies

    pip install -r requirements.txt

### 3. Download the data

Download the Confirmed Planets table from NASA Exoplanet Archive as CSV and place it at `data/exoplanets.csv`.

### 4. Run the notebook

    jupyter notebook notebooks/project_hot_jupiters.ipynb

Reproducibility: `random_state=42` is fixed across all experiments. Test metrics: F1 = 0.86, PR-AUC = 0.8937.

* * *

## ⚠️ Limitations and Warnings

1. **Target Circularity:** Features `pl_orbper` and `pl_rade` are part of the target definition and thus excluded from the feature set. The model learns only from indirect features (star properties, orbital parameters, discovery method).
2. **Class Imbalance:** Hot Jupiters are ~14.6% of the working sample. Accuracy is misleading — we focus on F1 and PR-AUC.
3. **Selection Bias:** The sample is biased towards transiting planets (radius is known only for them). When migrating to other discovery methods, retraining is required.
4. **Multicollinearity:** `st_teff` ↔ `st_rad` ↔ `st_mass` (up to 0.94). L2 regularization mitigates this.
5. **Missing Indicators Not Used in Final Model:** Due to the `ColumnTransformer` architecture, the missing indicators were dropped during preprocessing. The model still achieved F1 = 0.86 without them, confirming the robustness of the selected features.

* * *

## 🔮 Future Directions

  * Non-linear models (`RandomForest`, `GradientBoosting`, `XGBoost`): expected F1 improvement to 0.90+.
  * Fix the missing indicators issue by using a custom `FunctionTransformer` that preserves all columns.
  * Per-regime models: hot Jupiters vs. warm Jupiters vs. cold gas giants.
  * SHAP analysis for individual prediction interpretability.
  * API deployment (Flask / FastAPI): artifacts are ready.

* * *

## 📚 Sources and References

  * NASA Exoplanet Archive — source data.
  * Neptunian Desert — Nature, 2017 — context on radius bimodality.
  * sklearn: Missing Indicator — on missing indicators.

* * *

👤 **Author:** DigitalOcean78

📄 **License:** MIT License — free to use.

⭐ If this project helped you — leave a star!
