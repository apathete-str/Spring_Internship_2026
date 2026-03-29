# Scripts Notebook Workflows

This file summarizes the active notebooks in `scripts/`, their outputs, and their dependencies.

## Current filenames

- `01_eda_univariate_bivariate.ipynb`
- `02_outlier_handling.ipynb`
- `03_preprocessing_pipeline.ipynb`
- `04_noise_generation_gmm.ipynb`

## Dependency note

The notebook numbering matches the current filename convention, but the actual data dependency is:

1. `03_preprocessing_pipeline.ipynb` creates the cleaned and robust-scaled datasets.
2. `02_outlier_handling.ipynb` depends on `data/interim/ILPD_cleaned.csv`.
3. `04_noise_generation_gmm.ipynb` depends on `data/processed/ILPD_robust_scaled_features.csv`.
4. `01_eda_univariate_bivariate.ipynb` reads the raw dataset and can be run independently for exploratory work.

## 1) `01_eda_univariate_bivariate.ipynb`

### Workflow

1. Resolve the repo root and initialize the EDA figure folders.
2. Load the raw ILPD dataset with explicit schema.
3. Apply light preparation for visualization:
   - fill missing `Albumin_Globulin_Ratio`
   - map target labels
   - cast `Gender` as categorical
4. Run univariate EDA:
   - target and gender countplots
   - numeric histograms with KDE
   - numeric boxplots
   - summary statistics
5. Run bivariate EDA:
   - gender vs target countplot
   - feature vs target boxplots
   - selected pairplot
   - correlation heatmap
6. Export the notebook to HTML.

### Output

- `produced_reports/figures/eda/univariate/categorical_distributions.png`
- `produced_reports/figures/eda/univariate/numeric_histograms.png`
- `produced_reports/figures/eda/univariate/numeric_boxplots.png`
- `produced_reports/figures/eda/bivariate/gender_vs_target.png`
- `produced_reports/figures/eda/bivariate/numeric_vs_target_boxplots.png`
- `produced_reports/figures/eda/bivariate/selected_pairplot.png`
- `produced_reports/figures/eda/bivariate/correlation_heatmap.png`
- `produced_reports/01_eda_univariate_bivariate.html`

---

## 2) `02_outlier_handling.ipynb`

### Workflow

1. Load the cleaned dataset from `data/interim/ILPD_cleaned.csv`.
2. Run outlier diagnostics using:
   - IQR
   - Z-score
   - Modified Z-score
3. Generate KDE plots for each numeric feature.
4. Persist the outlier report and the outlier figure set.
5. Export the notebook to HTML.

### Output

- `produced_reports/docs/ILPD_outlier_report.csv`
- `produced_reports/figures/eda/outliers/kde_<feature>.png`
- `produced_reports/02_outlier_handling.html`

---

## 3) `03_preprocessing_pipeline.ipynb`

### Workflow

1. Resolve the repo root and initialize the output directories in `data/` and `produced_reports/docs/`.
2. Load raw ILPD and standardize the working schema.
3. Run data-quality checks:
   - missing-value snapshot
   - invalid and infinite value checks
   - `Gender` encoding
   - duplicate removal
4. Impute `Albumin_and_Globulin_Ratio` using group-wise median by `Gender` with global fallback.
5. Apply clinically bounded capping.
6. Build the robust-scaled feature dataset.
7. Persist cleaned, capped, and scaled datasets plus preprocessing metadata.
8. Export the notebook to HTML.

### Output

- `data/interim/ILPD_cleaned.csv`
- `data/processed/ILPD_clinically_capped.csv`
- `data/processed/ILPD_robust_scaled_features.csv`
- `produced_reports/docs/ILPD_preprocessing_metadata.json`
- `produced_reports/03_preprocessing_pipeline.html`

---

## 4) `04_noise_generation_gmm.ipynb`

### Workflow

1. Resolve the repo root and load `config/noise_generation.toml`.
2. Load the robust-scaled dataset from `data/processed/ILPD_robust_scaled_features.csv`.
3. Group rows by `Result` and `Gender`.
4. Fit a GMM inside each group and sample synthetic rows.
5. Add controlled jitter and clip samples to the observed robust-scaled range for stability.
6. Re-attach grouping columns, add `Synthetic_Flag`, and concatenate original plus synthetic rows.
7. Persist the augmented dataset and the generation summary.
8. Export the notebook to HTML.

### Output

- `data/processed/ILPD_robust_scaled_with_gmm_noise.csv`
- `produced_reports/docs/ILPD_noise_generation_summary.json`
- `produced_reports/04_noise_generation_gmm.html`

## Notes

- `legacy_notebooks/` stores older broad notebooks kept for reference.
- `produced_reports/` is the generated-output area for HTML reports, QA artifacts, metadata, and figures.
