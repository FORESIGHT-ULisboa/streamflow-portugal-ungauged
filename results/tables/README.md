# Results — Performance Tables

This directory contains pre-computed performance metrics for all experiments reported in the paper.

## Files

### `ungauged_metrics.csv`

Per-catchment metrics evaluated over the **full time series** for each test catchment under the ungauged setting. Corresponds to Fig. 5 and Fig. 6 in the paper.

| Column | Description |
|---|---|
| `station` | SNIRH station code |
| `model` | `HBV` or `TFT_ungauged` |
| `NSE` | Nash–Sutcliffe efficiency |
| `KGE` | Kling–Gupta efficiency |
| `CRPS` | Continuous ranked probability score (×10³ m³ s⁻¹ km⁻²) |
| `alpha` | Reliability (α) — TFT only |
| `pi_rel` | Relative resolution (π_rel) — TFT only |

### `specialized_metrics.csv`

Per-catchment metrics evaluated over the **20% test portion** for the ungauged, specialized, and HBV configurations. Corresponds to Fig. 11 and Fig. 13 in the paper.

| Column | Description |
|---|---|
| `station` | SNIRH station code |
| `model` | `HBV`, `TFT_ungauged`, or `TFT_specialized` |
| `training_fraction` | Fraction of local data used for specialization (NaN for HBV/ungauged) |
| `NSE` | Nash–Sutcliffe efficiency |
| `KGE` | Kling–Gupta efficiency |
| `CRPS` | Continuous ranked probability score (×10³ m³ s⁻¹ km⁻²) |
| `alpha` | Reliability (α) — TFT only |
| `pi_rel` | Relative resolution (π_rel) — TFT only |

## Median summary (test subsets)

| Model | Median NSE | Median KGE | Median CRPS |
|---|---|---|---|
| HBV (full time series) | 0.46 | 0.44 | 5.51 |
| TFT Ungauged (full time series) | 0.50 | 0.48 | 3.75 |
| HBV (20% test portion) | 0.48 | 0.40 | 5.16 |
| TFT Ungauged (20% test portion) | 0.48 | 0.44 | 3.59 |
| TFT Specialized (20% test portion) | 0.62 | 0.65 | 3.20 |

> **Note on the two test configurations:** results over the full time series (experiment 1) and the 20% test portion (experiment 2) are not directly comparable because the latter excludes the training/validation portions used for specialization.
