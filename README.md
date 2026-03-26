# Strategic Humanitarian Aid Allocation (HELP International)

## Overview
This project uses the **HELP International** dataset (`Country-data.csv`) to prepare data for clustering analysis.

It is split across two notebooks:
1. `Analysis.ipynb` (Task 1): Feature engineering (percent-to-absolute conversion) and saving a cleaned dataset.
2. `Preprocessing.ipynb` (Tasks 2-3): EDA, outlier review, winsorization, and standardization for K-Means.

## Dataset
File: `Country-data.csv`

Columns (used for modeling/EDA):
- `country` (identifier)
- `child_mort`
- `exports`, `health`, `imports` (these are percent-based indicators in the raw file)
- `income`, `inflation`, `life_expec`, `total_fer`, `gdpp`

### Data quality (key finding)
After applying the required percent-to-absolute conversion (see Task 1/2 below), the dataset has:
- Shape: **(167, 10)**
- Missing values: **0 total missing entries**

## Task 1 - Feature Engineering (`Analysis.ipynb`)
### What the notebook does
1. Loads `Country-data.csv`.
2. Checks and reports missing values.
3. Creates `df_cleaned = df.copy()`.
4. Converts percent-based indicators into absolute per-capita values using the specified rules:
   - `exports = exports * gdpp / 100`
   - `health  = health  * gdpp / 100`
   - `imports = imports * gdpp / 100`
5. Saves the engineered dataset to:
   - `Country-data-cleaned.csv`

### Why this conversion matters
Clustering and distance-based methods must compare indicators on consistent numerical meaning. Converting from “% of GDP” to “absolute per-capita values” ensures that magnitudes like “10% of GDP” are scaled appropriately by each country’s `gdpp`.

## Tasks 2-3 - EDA + K-Means Preprocessing (`Preprocessing.ipynb`)
### Reproducibility (key finding)
This notebook is designed to be **self-contained**:
- It loads `Country-data.csv` directly.
- It reproduces the Task 1 percent-to-absolute conversion **in memory only**.
- It **does not write** `Country-data-cleaned.csv`.

## Load and Prepare Data (includes validation)
Key in-memory objects:
- `df_prepared`: raw data + percent-to-absolute conversion

Validation performed:
- Confirms expected columns exist
- Converts expected feature columns to numeric
- Checks that required columns have no missing values
- Identifies numeric columns for modeling

It then prints:
- Dataset shape
- Dtypes of required columns
- The list of numeric features used in EDA/scaling

## Task 2 - Exploratory Data Analysis
Key steps and outputs:
1. Summary statistics table:
   - mean, median, std, and coefficient of variation (CV)
2. IQR outlier review:
   - Computes IQR bounds per feature and counts outliers
3. Boxplots:
   - Boxplots for all numeric features
   - Optional `log1p` boxplots for monetary-scale features (controlled by `PLOT_LOG_BOXPLOTS`)
4. Correlation heatmap:
   - Shows pairwise correlations among the feature columns

### Key EDA findings
**Relative variability (CV):**
- Highest CV: `exports` (**2.4222**), followed by `imports` (**2.2329**)
- Lowest CV: `total_fer` (**0.5135**) and `life_expec` (**0.1260**)

**IQR outlier counts (largest to smallest):**
- `gdpp`: 25
- `health`: 24
- `exports`: 18
- `imports`: 12
- `income`: 8
- `inflation`: 5
- `child_mort`: 4
- `life_expec`: 3
- `total_fer`: 1

**Correlations with `child_mort`:**
- Strong negative: `life_expec` (**-0.8867**)
- Moderate negative: `income` (**-0.5243**), `gdpp` (**-0.4830**)
- Strong positive: `total_fer` (**0.8485**)
- Mild positive: `inflation` (**0.2883**)

## Task 3 - Outlier Handling + Standardization for K-Means
### Winsorization/capping (IQR-based)
Key in-memory objects:
- `df_model`: winsorized (capped) numeric features

Approach:
- Uses the same IQR bounds to cap extreme values instead of deleting rows.
- Rationale for K-Means: reduces leverage of extreme values so centroids aren’t dominated by a few extreme countries.

### Standardization
Key in-memory objects:
- `X_scaled`: standardized numeric matrix
- `df_scaled`: standardized features with `country` preserved

Standardization method:
- `StandardScaler` to transform features to approximately:
  - mean ~ 0
  - standard deviation ~ 1

Validation:
- A check confirms the scaled feature means and stds are approximately at the target values.

### Optional exports
Flag:
- `SAVE_OUTPUTS = False` by default

If enabled, the notebook writes:
- `preprocessed_country_data.csv`
- `scaled_features.csv`

Otherwise, it keeps everything in-memory only.

