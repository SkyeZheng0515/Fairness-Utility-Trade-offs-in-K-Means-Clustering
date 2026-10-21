# Assessing Fairness-Utility Trade-offs in K-Means Clustering: A Systematic Empirical Evaluation

This study provides a systematic empirical evaluation of fairness-aware clustering and benchmarks Standard K-Means and six fairness-aware variants across 8 real-world datasets, expanded into 18 experimental configurations, under two k-selection strategies.

- **Strategy 1** applies a single consensus *k*, selected from Standard K-Means via majority vote across six k-selection heuristics, uniformly to all algorithms for controlled comparison.
- **Strategy 2** lets each algorithm's *k* be chosen by the same majority-vote consensus, revealing the best attainable performance of each method.

Results are evaluated across clustering utility and group fairness metrics, and repeated over **30 seeds (0–29)** for statistical robustness. A supplementary **k-sensitivity study** (k = 2–10) checks how stable each metric is to the choice of k.

---

## Environment (reproducibility)

All reported results, including the Table 9 runtime comparison, were produced with:

- **Python** 3.12.7
- **Packages:** numpy 1.26.4, pandas 2.2.2, scikit-learn 1.5.1, scipy 1.13.1, matplotlib 3.9.2, statsmodels 0.14.2, scikit-posthocs 0.17.1, aif360 0.6.1, ucimlrepo 0.0.7, requests 2.32.3, seaborn 0.13.2, openpyxl 3.1.5
- **OS:** Windows 11 (10.0.26200)
- **CPU:** Intel Core Ultra 9 185H (16 cores / 22 logical processors, 2.30 GHz)
- **RAM:** 32 GB

Runtime figures in Table 9 will scale with hardware and are reported for relative comparison across methods rather than as absolute benchmarks.

---

## Setup

```bash
pip install -r requirements.txt
```

Download the eight datasets and place them under `data/` (Diabetes and Dutch are downloaded automatically by the loaders):

| Dataset | Source |
|---|---|
| Adult Income | https://archive.ics.uci.edu/dataset/2/adult |
| COMPAS | https://github.com/propublica/compas-analysis |
| German Credit | https://archive.ics.uci.edu/dataset/144/statlog+german+credit+data |
| Default Credit Card | https://archive.ics.uci.edu/dataset/350/default+of+credit+card+clients |
| Law School (LSAC) | https://github.com/damtharvey/law-school-dataset |
| Diabetes 130-US | https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008 |
| Dutch Census | https://github.com/tailequy/fairness_dataset/tree/main/Dutch_census |
| MEPS (Panel 20, 2016) | https://meps.ahrq.gov/mepsweb/data_stats/download_data_files_results.jsp?cboDataYear=All&cboDataTypeY=1%2CHousehold+Full+Year+File&buttonYearandDataType=Search&cboPufNumber=All&SearchTitle=Longitudinal |

The first 5 datasets form **Part A** (original datasets); Diabetes / Dutch / MEPS form **Part B** (new datasets), added to test generalization beyond the original benchmark.

> **Note on file paths:** All paths are relative. The repository is organised as `data/` (raw datasets), `notebooks/` (all notebooks) and `results/` (all result files). Each notebook first switches its working directory to `../results`, so result files are read from and written to `results/`, and raw datasets are read from `../data/`. For MEPS, `h192.csv` is read from the aif360 raw-data folder (`aif360/data/raw/meps/`), which is located automatically from the installed aif360 package.

Each base dataset is expanded into multiple sensitive-attribute configurations (gender / race / combined, where the attribute is available):

- **Part A: 11 configurations** — adult, compas, german, credit, law, adult_race, adult_combined, compas_race, compas_combined, law_race, law_combined (german and credit have no race attribute)
- **Part B: 7 configurations** — diabetes_gender, diabetes_race, diabetes_combined, dutch_gender, meps_gender, meps_race, meps_combined
- **Total: 18 configurations across 8 base datasets**

### Algorithms

- K-Means
- Fairlet Decomposition
- Balanced Fair K-means (BFKM)
- Cluster-level Centroid Fairness (CCF)
- Post-Processing Based Nearest Foreign Point (PP-NFP)
- Post-Processing Based Gini (PP-Gini)
- Rawlsian K-Means

---

## Execute

### Step 1 — 30-run repeated experiments (final results, seeds 0–29)

```
Strategy1_PartA_30runs.ipynb
Strategy1_PartB_30runs.ipynb

Strategy2_PartA_ccf_30runs.ipynb
Strategy2_PartB_ccf_30runs.ipynb
Strategy2_PartA_kmf_30runs.ipynb
Strategy2_PartB_kmf_30runs.ipynb
```

These are the final reported results. All Strategy 2 30-run notebooks use **per-algorithm optimal k** (each algorithm's own best k, not a single consensus k) — this is the only k-selection variant carried forward for Strategy 2's 30-run stage.

### Step 2 — k-sensitivity analysis (supplementary)

Sweeps k = 2–10 for every configuration × method and checks how Silhouette / Balance / SPD move with k. Run in order:

```
k_Sensitivity_Analysis.ipynb            # main sweep -> k_sensitivity.pkl
k_Sensitivity_Fairlet_Only.ipynb        # Fairlet is slow, run separately -> k_sensitivity_fairlet.pkl
k_Sensitivity_Plots.ipynb               # final figures: Figure 5a/b/c, Figures S1–S3, Table S5
```

### Step 3 — Compile & normalize results

Reads the 6 pkl files produced in Step 1 directly (no manual Excel assembly):

```
Results_Normalized.ipynb
```

Outputs: normalized (0–1 min-max scaled) result tables in `results/`.

### Step 4 — Final tables, figures & significance tests

```
Results_Visualization.ipynb   # Table5 Win-Tie-Loss, Table6 Heatmap, Table7 MFR
Runtime_Analysis.ipynb        # per-algorithm runtime comparison (Table 9)
Statistical_Tests.ipynb       # Friedman + Wilcoxon post-hoc (Bonferroni), effect sizes, Nemenyi
```

All output files above are written to the same `results/` folder.

---

## Supplementary diagnostics

- **`Encoding_Sensitivity_Check.ipynb`** — LabelEncoder vs. One-Hot encoding comparison on the Adult dataset (Adjusted Rand Index), informing the switch to One-Hot encoding used throughout the pipeline.
- **`Table1_From_Your_Loaders.ipynb`** — recomputes the analytic sample size, post-encoding feature count, and sensitive-group sizes reported in Table 1, directly from the dataset loader functions.
