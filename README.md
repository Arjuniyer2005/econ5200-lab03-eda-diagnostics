# econ5200-lab03-eda-diagnostics
## Pipeline Health Check — EDA, Corruption & Distribution Shift

**Objective:** Diagnose and repair planted data-quality failures in a country-level panel using exploratory analysis alone, then quantify training-to-inference distribution shift with the Population Stability Index (PSI) and package the checks into reusable utilities and an interactive monitoring dashboard.

### Methodology
- **Diagnosis-first EDA:** Profiled a 260-row country panel with summary statistics, distribution plots, and range checks to locate five planted corruptions before applying any fixes:
  - negative GDP values
  - life expectancy recorded in months instead of years
  - duplicate rows
  - percentage fields mixing 0–1 and 0–100 scales
  - a GDP unit mismatch
- **Remediation:** Resolved each issue with a targeted, documented fix [FILL: one line per fix, e.g. "life expectancy ÷ 12"], reducing the panel from 260 to 230 rows.
- **Distribution-shift measurement:** Computed PSI between a 138-row training split and a 92-row inference split, with GDP in the inference split scaled by 1.3 to simulate shift. Bin edges were taken from [FILL: training sample / pooled sample, per `eda_utils.detect_distribution_shift`], with epsilon smoothing to keep empty bins finite.
- **Tool comparison:** Benchmarked manual EDA against ydata-profiling and recorded which corruptions each approach surfaced or missed.
- **Reusable utilities:** Wrote `eda_utils.py` with three functions:
  - `check_impossible_values(df, constraints)`: rule-based range validation
  - `detect_distribution_shift(train_col, inference_col, n_bins)`: returns PSI and an interpretation
  - `eda_summary(df)`: per-frame profiling
- **Monitoring dashboard:** Built an ipywidgets/plotly dashboard with four panels:
  - train-vs-inference overlays (histogram and ECDF)
  - a PSI threshold slider with a permutation-based noise baseline
  - editable min/max constraints wired to `check_impossible_values`
  - a dirty-vs-clean comparison showing dtype, null, non-numeric, range, and duplicate deltas

### Key Findings
- **Data quality:** All five planted corruptions were identified through EDA alone and corrected; 30 rows were removed in cleaning (260 → 230).
- **Distribution shift:** GDP PSI = [FILL: value], classified as [FILL: stable / moderate / major] under the conventional 0.10 / 0.25 thresholds.
- **PSI calibration:** At these sample sizes (138 vs. 92 rows, 10 bins), the expected PSI under *no* shift is roughly 9 × (1/138 + 1/92) ≈ 0.16. The 0.10 threshold alone would therefore flag stable columns, so PSI values were read against a permutation-derived noise floor rather than fixed cutoffs.
- **Manual EDA vs. automated profiling:** ydata-profiling [FILL: what it caught / missed]; manual EDA [FILL: what it caught / missed]. The main gap was domain knowledge: a profiler reports ranges but cannot know that a value is physically impossible.
- **Limits of PSI:** PSI is a per-column, numeric-only check. It cannot detect:
  - duplicates
  - categorical inconsistencies
  - values that are wrong but still plausible
  - corruptions present equally in both samples
  - joint (multivariate) shift
  
  Because the training and inference splits come from the cleaned frame, PSI detects only the injected GDP shift, not the upstream corruptions. This supports treating PSI as one layer of pipeline monitoring, not a substitute for validation rules.
