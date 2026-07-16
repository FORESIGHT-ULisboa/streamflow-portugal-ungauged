# Results

This folder contains all model outputs and performance metrics produced for the paper:

> **Francisco, R. and Matos, J. P.** — *Streamflow prediction in natural ungauged catchments using Temporal Fusion Transformers*, Hydrology and Earth System Sciences (HESS), 2026.

---

## Structure

```
results/
├── hbv/
│   ├── metrics/
│   │   ├── for the entire time series (in ungauged)/
│   │   │   └── hbv_metrics_Q_A.csv          # HBV metrics over the full record (compare with TFT ungauged)
│   │   └── for test split in specialization/
│   │       └── hbv_metrics_Q_A.csv          # HBV metrics over the 20 % test split (compare with TFT specialization)
│   └── prediction/                          # HBV calibration outputs — one subfolder per catchment (54)
│       ├── <STATION>/
│       │   ├── best_parameters.txt          # CMA-ES calibration summary and optimal parameter set
│       │   ├── <STATION>_HBV_simulation.csv # Daily simulated streamflow (raw Q, m³ s⁻¹)
│       │   └── <STATION>_optimal_forcing.txt
│       └── README.md                        # Detailed HBV file formats and calibration settings
└── tft/
    ├── metrics/
    │   ├── ungauged/
    │   │   └── config_<N>/                  # N = 0 … 13
    │   │       └── metrics_Q_A.xlsx         # One file per configuration (14 total)
    │   └── specialization/
    │       └── config_<N>/<STATION>/version_<V>/
    │           └── metrics_Q_A.xlsx         # One file per config × station × retraining version (504 total)
    └── prediction/
        ├── ungauged/
        │   └── config_<N>/
        │       └── <STATION>.csv            # One file per predicted station (51–53 per config, 736 total)
        └── specialization/
            └── config_<N>/<STATION>/version_<V>/
                └── <STATION>.csv            # One file per config × station × retraining version (504 total)
```

---

## Contents at a glance

| Folder | Experiment | Contents |
|---|---|---|
| [`hbv/prediction/`](hbv/prediction/) | HBV benchmark calibration | Simulation, optimal forcing, and best parameters for 54 catchments — see its [README](hbv/prediction/README.md) |
| [`hbv/metrics/`](hbv/metrics/) | HBV benchmark evaluation | Per-station NSE, KGE, and CRPS — one CSV over the full time series, one over the test split |
| [`tft/metrics/ungauged/`](tft/metrics/ungauged/) | TFT ungauged (Section 4.1) | One `metrics_Q_A.xlsx` per configuration (14 total) |
| [`tft/metrics/specialization/`](tft/metrics/specialization/) | TFT specialization (Section 4.2) | One `metrics_Q_A.xlsx` per retraining version, per station, per configuration (504 total) |
| [`tft/prediction/ungauged/`](tft/prediction/ungauged/) | TFT ungauged (Section 4.1) | Quantile predictions over the full record, one CSV per station per configuration (736 total) |
| [`tft/prediction/specialization/`](tft/prediction/specialization/) | TFT specialization (Section 4.2) | Quantile predictions per retraining version (504 total) |

Each of the 14 TFT configurations (`config_0` … `config_13`) holds out a different subset of stations. Specialization retraining was run for 3–4 target stations per configuration (52 stations in total), each with up to 10 independent retraining versions (`version_0` … `version_9`).

---

## File formats

### TFT prediction CSVs (`tft/prediction/**/<STATION>.csv`)

Daily observed and predicted specific streamflow:

| Column | Description |
|---|---|
| `date` | Calendar date (YYYY-MM-DD) |
| `observed` | Observed specific streamflow q |
| `split` | Temporal partition: `Training`, `Validation`, `Test` (specialization only), or blank |
| `p_0.01` … `p_0.99` | Predicted quantiles (0.01, 0.05, 0.1, 0.25, 0.4, 0.5, 0.6, 0.75, 0.9, 0.95, 0.99) |

Quantile columns are blank on days without a prediction. In ungauged mode the predictions cover the full available record; in specialization mode the evaluation uses the `Test` split.

### HBV metrics CSVs (`hbv/metrics/**/hbv_metrics_Q_A.csv`)

Semicolon-separated, one row per station: `Group` (SNIRH code with slash), `NSE`, `KGE`, `CRPS`.

### HBV calibration outputs (`hbv/prediction/`)

Documented in detail in [`hbv/prediction/README.md`](hbv/prediction/README.md) — parameter definitions, simulation CSV format, forcing file layout, and calibration settings.

---

## Key conventions

**Station codes.** SNIRH codes use a slash (e.g. `03J/01H`). Folder and file names replace the slash with an underscore (e.g. `03J_01H`).

**Leadtime.** All `metrics_Q_A.xlsx` files retain only the **leadtime = 3 days** columns. Climatology values in those columns are taken from leadtime = 1 day because Climatology is a single-step benchmark whose value does not depend on the forecast horizon.

**Units.** Streamflow is expressed as specific streamflow: q = Q / A × 10³, in units of 10⁻³ m³ s⁻¹ km⁻². The HBV simulation CSV stores raw Q in m³ s⁻¹; apply the normalisation before comparing with TFT outputs.

---

## Relation to the paper

`hbv/prediction/` — Section 2.3.1. HBV was calibrated with all available local data per catchment, so it constitutes an **upper-bound benchmark** relative to the ungauged TFT, which has no access to local streamflow during training.

`tft/metrics/ungauged/` and `hbv/metrics/for the entire time series (in ungauged)/` — Section 4.1. Performance over the full available time series for both TFT (in strictly ungauged mode) and HBV.

`tft/metrics/specialization/` and `hbv/metrics/for test split in specialization/` — Section 4.2. Performance over the 20 % test split for TFT after retraining with limited local data, alongside HBV on the same test split.

`tft/prediction/` — underlying daily quantile predictions from which all TFT metrics were computed.
