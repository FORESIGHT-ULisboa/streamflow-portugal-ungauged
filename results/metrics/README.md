# Metrics

Performance metrics for both the **ungauged** and **specialization** TFT experiments, together with HBV reference values. All `metrics_Q_A.xlsx` files contain only the **leadtime = 3 days** columns; see the notes below on Climatology.

---

## Structure

```
metrics/
├── ungauged/
│   ├── config_0/
│   │   └── metrics_Q_A.xlsx          # TFT ungauged + Persistence + Climatology
│   ├── config_1/
│   │   └── metrics_Q_A.xlsx
│   ├── ...                           (14 configs, one file each)
│   └── hbv metrics for the entire time series/
│       └── hbv_metrics_Q_A.csv       # HBV NSE, KGE, CRPS — full record
│
└── specialization/
    ├── config_0/
    │   ├── 04O_02H/
    │   │   ├── version_0/
    │   │   │   └── metrics_Q_A.xlsx  # TFT after retraining — run 0
    │   │   ├── version_1/
    │   │   │   └── metrics_Q_A.xlsx
    │   │   └── ...                   (10 versions per station)
    │   ├── 12G_02H/
    │   │   └── ...
    │   └── ...                       (4 stations per config on average)
    ├── config_1/
    │   └── ...
    ├── ...                           (14 configs; 52 stations; 504 files total)
    └── hbv metrics for the test split/
        └── hbv_metrics_Q_A.csv       # HBV NSE, KGE, CRPS — test split only
```

---

## `ungauged/`

Each `config_N/metrics_Q_A.xlsx` contains the evaluation of the best TFT ungauged model for that leave-one-group-out configuration (θ = 0…13) against Persistence and Climatology benchmarks. Metrics cover Training, Validation, and Test subsets across all catchments active in that configuration.

The best model per configuration was selected as the run with the highest mean NSE on the spatial validation group (Eq. 11 in the paper). Ten independent training runs were performed per configuration (Table 1 and Section 2.5.1).

`hbv metrics for the entire time series/hbv_metrics_Q_A.csv` — semicolon-separated summary of HBV performance over the **full available record** for all 56 catchments. Columns: `Group`, `NSE`, `KGE`, `CRPS`.

---

## `specialization/`

Each `config_N/<STATION>/version_K/metrics_Q_A.xlsx` contains metrics for one independent retraining run (K = 0…9) of the best ungauged model for configuration N, fine-tuned on local data from the target station. The test split is 20 % of the available time series, fixed across all versions.

`hbv metrics for the test split/hbv_metrics_Q_A.csv` — same format as the ungauged HBV summary, but restricted to the **test split** used in the specialization experiment.

---

## `metrics_Q_A.xlsx` — file format

Each workbook has a single sheet (`metrics_3days`) with the following layout:

**Header — rows 1–2:**

| Row | Content |
|---|---|
| 1 | Catchment group name (e.g. `03J/01H`) — merged across its Subset columns |
| 2 | Subset: `Training`, `Validation`, or `Test` |

**Index — columns A–B:**

| Column | Content |
|---|---|
| A | Metric name (forward-filled across model rows within the same metric) |
| B | Model: `TFT`, `Persistence`, or `Climatology` |

**Data — column C onward:** one column per (catchment, subset) combination, values for leadtime = 3 days.

---

## Metrics

| Metric | Optimal | Units | Description | Reference |
|---|---|---|---|---|
| `CRPS` | 0 | 10⁻³ m³ s⁻¹ km⁻² | Continuous Ranked Probability Score | Hersbach (2000) |
| `KGE` | 1 | — | Kling–Gupta efficiency | Gupta et al. (2009) |
| `MAE` | 0 | 10⁻³ m³ s⁻¹ km⁻² | Mean Absolute Error | — |
| `NSE` | 1 | — | Nash–Sutcliffe efficiency | Nash and Sutcliffe (1970) |
| `quantile_loss` | 0 | — | Pinball loss averaged over quantile levels | Koenker and Bassett (1978) |
| `relative_bias` | 0 | — | Relative mean bias | — |
| `reliability` | 1 | — | PIT-based reliability index α | Renard et al. (2010) |
| `resolution_relative` | +∞ | — | Relative resolution π_rel | Renard et al. (2010) |

---

## Models

| Model | Description |
|---|---|
| `TFT` | Temporal Fusion Transformer — ungauged or after specialization retraining |
| `Persistence` | Naive benchmark: forecast equals the last observed value |
| `Climatology` | Long-term daily mean for the calendar day (single-step; see note) |

---

## Note on Climatology and leadtime

Climatology produces the same forecast regardless of lead time (the long-term daily mean). Its values in the 3-day columns are taken from its leadtime = 1 day results, which are equivalent in meaning. This substitution is applied by `collect_metrics.py` and `collect_retraining.py` and does not affect interpretation.

---

## Selecting the best specialization version

For the paper, the best version per catchment is the run with the highest NSE on the **Validation** subset. The `best_version` dictionary in `collect_metrics.py` records this selection for the ungauged experiment; the analogous selection for specialization is made per catchment from the 10 available versions in each station folder.

---

## References

- Gupta, H. V., Kling, H., Yilmaz, K. K., and Martinez, G. F. (2009). Decomposition of the mean squared error and NSE performance criteria. *Journal of Hydrology*, 377, 80–91.
- Hersbach, H. (2000). Decomposition of the Continuous Ranked Probability Score for ensemble prediction systems. *Weather and Forecasting*, 15, 559–570.
- Koenker, R. and Bassett, G. J. (1978). Regression quantiles. *Econometrica*, 46, 33–50.
- Nash, J. E. and Sutcliffe, J. V. (1970). River flow forecasting through conceptual models. *Journal of Hydrology*, 10, 282–290.
- Renard, B., Kavetski, D., Kuczera, G., Thyer, M., and Franks, S. W. (2010). Understanding predictive uncertainty in hydrologic modeling. *Water Resources Research*, 46.
