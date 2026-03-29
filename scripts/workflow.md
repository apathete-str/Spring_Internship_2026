# Scripts Notebook Workflows

This file summarizes the workflow and outputs of each notebook in `scripts/`.

## 1) `eda_univariate_bivariate.ipynb`

### Workflow (compressed)
1. Resolve repo root and set paths (`data/raw/ILPD.csv`, figure output folders).
2. Load ILPD with explicit schema/column names.
3. Basic prep:
   - fill missing `Albumin_Globulin_Ratio` with median,
   - map target labels (1/2 → disease/no disease),
   - cast `Gender` as categorical.
4. Univariate EDA:
   - target and gender countplots,
   - numeric histograms (+KDE),
   - numeric boxplots,
   - summary stats (`describe`).
5. Bivariate EDA:
   - gender vs target countplot,
   - feature vs target boxplots,
   - selected pairplot,
   - correlation heatmap (including encoded gender/target).

### Output
- Saved figures under:
  - `reports/figures/eda/univariate/`
  - `reports/figures/eda/bivariate/`
- Typical artifacts include:
  - `categorical_distributions.png`
  - `numeric_histograms.png`
  - `numeric_boxplots.png`
  - `gender_vs_target.png`
  - `numeric_vs_target_boxplots.png`
  - `selected_pairplot.png`
  - `correlation_heatmap.png`

---

## 2) `preprocessing_pipeline.ipynb`

### Workflow (compressed)
1. Resolve repo root; initialize output directories in `data/` and `produced_reports/docs/`.
2. Load raw ILPD and standardize schema.
3. Data-quality checks:
   - missing-value snapshot,
   - invalid/infinite checks,
   - `Gender` encoding,
   - duplicate removal.
4. Missing-value treatment:
   - `Albumin_and_Globulin_Ratio` imputed using group-wise median by `Gender` + global fallback.
5. Clinical capping (winsorization-style) using pre-defined plausible limits.
6. Robust scaling for model-ready features (`Result` retained unscaled).
7. Persist datasets and preprocessing metadata manifest.

### Output
- Data artifacts:
  - `data/interim/ILPD_cleaned.csv`
  - `data/processed/ILPD_clinically_capped.csv`
  - `data/processed/ILPD_robust_scaled_features.csv`
- Governance artifact:
  - `produced_reports/docs/ILPD_preprocessing_metadata.json`

---

## 3) `outlier_handling.ipynb`

### Workflow (compressed)
1. Load cleaned dataset (`data/interim/ILPD_cleaned.csv`).
3. Run outlier diagnostics per numeric feature:
   - IQR,
   - Z-score,
   - Modified Z-score.
4. Generate KDE plots for each numeric feature.
4. Persist outlier report and KDE figures.

### Output
- QA report:
  - `produced_reports/docs/ILPD_outlier_report.csv`
- KDE figures:
  - `produced_reports/figures/eda/outliers/kde_<feature>.png`


---

## HTML export

Each notebook includes a final cell that converts itself to HTML and writes output to `produced_reports/html/`.

---

## 4) `noise_generation_gmm.ipynb`

### Workflow (compressed)
1. Load robust-scaled dataset (`data/processed/ILPD_robust_scaled_features.csv`).
2. Load noise hyperparameters from `config/noise_generation.toml` (including `noise_ratio = 0.2`).
3. Follow legacy GMM noise-generation idea:
   - fit per-group GMM (`Result`, `Gender`),
   - sample synthetic candidates,
   - add small Gaussian jitter,
   - enforce structural constraints (binary `Gender`),
   - clip to observed robust feature ranges.
4. Merge original + synthetic rows and tag using `Synthetic_Flag`.
5. Save augmented dataset and generation summary.

### Output
- Augmented robust dataset:
  - `data/processed/ILPD_robust_scaled_with_gmm_noise.csv`
- Noise generation summary:
  - `produced_reports/docs/ILPD_noise_generation_summary.json`
- HTML export:
  - `produced_reports/html/noise_generation_gmm.html`

Noise config file:
- `config/noise_generation.toml` (set `noise.noise_ratio = 0.2` for 20% synthetic rows).
