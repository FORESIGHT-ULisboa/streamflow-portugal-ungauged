# Results

This folder contains all model outputs and performance metrics produced for the paper:

> **Francisco, R. and Matos, J. P.** — *Streamflow prediction in natural ungauged catchments using Temporal Fusion Transformers*, Hydrology and Earth System Sciences (HESS), 2026.

---

## Structure

```
results/
├── hbv prediction/          # HBV calibration outputs (one subfolder per catchment)
└── metrics/                 # Performance metrics for all experiments
    ├── ungauged/            # TFT ungauged and HBV metrics (full time series)
    └── specialization/      # TFT specialization and HBV metrics (test split)
```

---

## Contents at a glance

| Folder | Experiment | Contents |
|---|---|---|
| [`hbv prediction/`](hbv%20prediction/) | HBV benchmark calibration | Simulation, optimal forcing, and best parameters for 56 catchments |
| [`metrics/ungauged/`](metrics/) | TFT ungauged (Section 4.1) | One `metrics_Q_A.xlsx` per configuration (14 total); HBV summary CSV over the full time series |
| [`metrics/specialization/`](metrics/) | TFT specialization (Section 4.2) | One `metrics_Q_A.xlsx` per retraining version, per station, per configuration (504 files total); HBV summary CSV over the test split |

---

## Key conventions

**Station codes.** SNIRH codes use a slash (e.g. `03J/01H`). Folder and file names replace the slash with an underscore (e.g. `03J_01H`).

**Leadtime.** All `metrics_Q_A.xlsx` files retain only the **leadtime = 3 days** columns. Climatology values in those columns are taken from leadtime = 1 day because Climatology is a single-step benchmark whose value does not depend on the forecast horizon.

**Units.** Streamflow is expressed as specific streamflow: q = Q / A × 10³, in units of 10⁻³ m³ s⁻¹ km⁻². The HBV simulation CSV stores raw Q in m³ s⁻¹; apply the normalisation before comparing with TFT outputs.

---

## Relation to the paper

`hbv prediction/` — Section 2.3.1. HBV was calibrated with all available local data per catchment, so it constitutes an **upper-bound benchmark** relative to the ungauged TFT, which has no access to local streamflow during training.

`metrics/ungauged/` — Section 4.1. Performance over the full available time series for both TFT (in strictly ungauged mode) and HBV.

`metrics/specialization/` — Section 4.2. Performance over the 20 % test split for TFT after retraining with limited local data, alongside HBV on the same test split.
