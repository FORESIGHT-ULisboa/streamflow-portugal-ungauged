# Streamflow Prediction in Natural Ungauged Catchments using Temporal Fusion Transformers

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.10](https://img.shields.io/badge/python-3.10-blue.svg)](https://www.python.org/)
[![HESS](https://img.shields.io/badge/journal-HESS-green)](https://doi.org/XXXX)

**`streamflow-portugal-ungauged`** is the official code and data repository for the paper:

> **Francisco, R. and Matos, J. P.** — *Streamflow prediction in natural ungauged catchments using Temporal Fusion Transformers*, Hydrology and Earth System Sciences (HESS), 2026.

This repository contains the full experimental pipeline — data, environment setup, model training, evaluation, and results — to reproduce all findings reported in the paper, including **ungauged prediction** (leave-one-group-out cross-validation) and **model specialization** (transfer-learning fine-tuning with limited local data). The HBV hydrological model calibrated for each catchment is included as a benchmark.

> **New here? Start with the notebooks** in [`notebooks/`](notebooks/):  
> [`00_data_overview`](notebooks/00_data_overview.ipynb) · [`01_hbv_benchmark`](notebooks/01_hbv_benchmark.ipynb) · [`02_tft_ungauged`](notebooks/02_tft_ungauged.ipynb) · [`03_tft_specialization`](notebooks/03_tft_specialization.ipynb) · [`04_performance_tables`](notebooks/04_performance_tables.ipynb) · [`05_dynamic_hydrographs`](notebooks/05_dynamic_hydrographs.ipynb)

---

## Table of contents

- [Overview](#overview)
- [Repository structure](#repository-structure)
- [Installation](#installation)
- [Data](#data)
- [Running the experiments](#running-the-experiments)
- [Results](#results)
- [Citation](#citation)
- [License](#license)

---

## Overview

Streamflow prediction in ungauged basins remains one of the central challenges in operational hydrology. This work evaluates the ability of **Temporal Fusion Transformers (TFT, Lim et al., 2021)** to generalize hydrological knowledge across **53 unregulated catchments in mainland Portugal** under strictly ungauged conditions, and further explores how limited local observations can be used to specialize a pre-trained regional model.

Key results on the test subsets:

| Model | Median NSE | Median KGE | Median CRPS (×10³ m³ s⁻¹ km⁻²) |
|---|---|---|---|
| HBV (individually calibrated — upper bound) | 0.46 | 0.44 | 5.51 |
| **TFT Ungauged** | **0.50** | **0.48** | **3.75** |
| **TFT Specialized** | **0.62** | **0.65** | **3.20** |

> **Note:** HBV is calibrated with full local data and therefore represents an unfair, upper-bound benchmark for the ungauged TFT. Despite this, the TFT ungauged model performs comparably or better on the majority of catchments.

---

## Repository structure

```
streamflow-portugal-ungauged/
├── environment/
│   └── environment.yml             # Conda environment (Python 3.10 + all dependencies)
├── data/
│   ├── hydrological/               # Daily streamflow from SNIRH (53 catchments, 1980–2024)
│   │   └── README.md               # Data provenance and quality-control description
│   ├── meteorological/             # ERA5-Land precipitation and temperature (1980–2024)
│   │   └── README.md
│   └── geomorphological/           # Catchment descriptors (area, elevation, land use, …)
│       └── README.md
├── notebooks/
│   ├── 00_data_overview.ipynb      # Exploratory data analysis; catchment map; statistics
│   ├── 01_hbv_benchmark.ipynb      # HBV calibration with CMA-ES; benchmark metrics
│   ├── 02_tft_ungauged.ipynb       # Leave-one-group-out TFT training and evaluation
│   ├── 03_tft_specialization.ipynb # Transfer-learning retraining with limited local data
│   ├── 04_performance_tables.ipynb # Aggregate metric tables (NSE, KGE, CRPS, α, π_rel)
│   └── 05_dynamic_hydrographs.ipynb# Interactive Plotly hydrographs (saved as .html)
├── results/
│   ├── tables/
│   │   ├── ungauged_metrics.csv    # Per-catchment metrics for TFT ungauged and HBV
│   │   └── specialized_metrics.csv # Per-catchment metrics for TFT specialized
│   └── hydrographs/                # Pre-generated interactive hydrograph .html files
│       ├── ungauged/               # One .html per catchment (ungauged setting)
│       └── specialized/            # One .html per catchment (after specialization)
├── AGENTS.md                       # Conventions for AI coding agents
├── CLAUDE.md                       # → points to AGENTS.md
├── LICENSE
└── README.md
```

---

## Installation

### 1 · Clone the repository

```bash
git clone https://github.com/FORESIGHT-ULisboa/streamflow-portugal-ungauged.git
cd streamflow-portugal-ungauged
```

### 2 · Create the conda environment

All dependencies — PyTorch, PyTorch Forecasting, eWaterCycle/HBV, xarray, and the full scientific stack — are declared in [`environment/environment.yml`](environment/environment.yml).

```bash
conda env create -f environment/environment.yml
conda activate streamflow_portugal_ungauged
```

> **GPU support:** by default the environment installs CPU-only PyTorch. To use a GPU, remove the `cpuonly` line from `environment.yml` and replace it with the appropriate `pytorch-cuda` package for your driver version (see [pytorch.org](https://pytorch.org/get-started/locally/)).

### 3 · Register the Jupyter kernel

```bash
python -m ipykernel install --user \
    --name streamflow_portugal_ungauged \
    --display-name "streamflow_portugal_ungauged"
```

### 4 · Open the notebooks

Open directly in VS Code and select the **streamflow_portugal_ungauged** kernel, or launch JupyterLab:

```bash
jupyter lab notebooks/
```

---

## Data

All data used in this study are publicly available. The sections below describe each source and the preprocessing applied before model inputs were assembled.

### Hydrological data

| Property | Value |
|---|---|
| Source | [SNIRH — Sistema Nacional de Recursos Hídricos](https://snirh.apambiente.pt/) (Portuguese Environment Agency, 2024) |
| Variable | Average daily streamflow (m³ s⁻¹) |
| Period | 1980–2024 |
| Catchments | 53 unregulated catchments in mainland Portugal |
| Minimum record | 10 years of daily data per catchment (not necessarily continuous) |
| Quality control | Hydrological years containing anomalies were removed conservatively (see paper §3.2.1) |

Raw files are stored in `data/hydrological/` as one CSV per catchment, named by station code (e.g., `03J01H.csv`). See [`data/hydrological/README.md`](data/hydrological/README.md) for the column schema and the quality-control log.

Streamflow was normalized to specific streamflow (m³ s⁻¹ km⁻²) × 10³ to facilitate cross-catchment comparison and to prevent area-driven dominance during training (Eq. 2 in the paper).

### Meteorological data

| Property | Value |
|---|---|
| Source | [ERA5-Land](https://doi.org/10.5194/essd-13-4349-2021) — ECMWF Copernicus Climate Data Store |
| Variables | Hourly 2 m air temperature (K); total precipitation (m) |
| Spatial resolution | ≈ 9 km (0.1° × 0.1° grid) |
| Period | 1980–2024 |
| Aggregation | Spatially averaged over each catchment polygon; converted to daily totals / means in local time (UTC+1 / UTC+2 with DST) |

Processed files are stored in `data/meteorological/` as one CSV per catchment. See [`data/meteorological/README.md`](data/meteorological/README.md). No additional quality control was applied to ERA5-Land (see paper §3.2.2).

### Geomorphological data

| Descriptor | Source | Notes |
|---|---|---|
| Drainage area (km²) | Derived from DEM | — |
| Centroid coordinates (WGS84) | Derived from catchment polygon | Latitude and longitude used as static real inputs |
| Mean elevation (m) | [Copernicus DEM GLO-30](https://doi.org/10.5270/ESA-c5d3d65) | 30 m resolution |
| Main land use | [CORINE Land Cover 2018](https://doi.org/10.2909/71c95a07-e296-44fc-b22b-415f42acfdf0) | Static categorical input |
| Gravelius compactness index | Computed from polygon | Shape descriptor; 1 = perfect circle |
| Average annual temperature (°C) | ERA5-Land (1980–2024 mean) | Static real input |
| Average annual precipitation (mm) | ERA5-Land (1980–2024 mean) | Static real input |

All descriptors are compiled in `data/geomorphological/catchment_descriptors.csv`. See [`data/geomorphological/README.md`](data/geomorphological/README.md).

---

## Running the experiments

After installation and data preparation, the experiments can be reproduced in order via the notebooks:

| Notebook | Description | Estimated runtime |
|---|---|---|
| `00_data_overview` | Data exploration, cluster map, flow duration curves | < 5 min |
| `01_hbv_benchmark` | HBV calibration (CMA-ES, 4 000 evaluations per catchment) | ~6 h (CPU) |
| `02_tft_ungauged` | Leave-one-group-out TFT training (14 configurations × 10 runs) | ~12 h (GPU) / ~48 h (CPU) |
| `03_tft_specialization` | Specialization retraining (6 data fractions × 10 runs × 46 catchments) | ~8 h (GPU) |
| `04_performance_tables` | Aggregate NSE / KGE / CRPS / α / π_rel tables and figures | < 5 min |
| `05_dynamic_hydrographs` | Interactive Plotly hydrographs exported as `.html` | < 10 min |

Pre-computed results for all experiments are available in [`results/`](results/) so you can run notebooks `04` and `05` directly without re-training.

---

## Results

### Performance tables

Aggregate metrics for the test subsets are available as CSV files in [`results/tables/`](results/tables/):

- [`ungauged_metrics.csv`](results/tables/ungauged_metrics.csv) — NSE, KGE, CRPS, α, π_rel for every catchment under the TFT ungauged and HBV configurations.
- [`specialized_metrics.csv`](results/tables/specialized_metrics.csv) — same metrics for TFT specialized at multiple data fractions (0%, 1%, 5%, 20%, 30%, 40%, 50%, 60%).

Column schema for both files:

| Column | Description |
|---|---|
| `station` | SNIRH station code |
| `model` | `HBV`, `TFT_ungauged`, or `TFT_specialized` |
| `training_fraction` | Fraction of local data used for specialization (NaN for ungauged/HBV) |
| `NSE` | Nash–Sutcliffe efficiency |
| `KGE` | Kling–Gupta efficiency |
| `CRPS` | Continuous ranked probability score (×10³ m³ s⁻¹ km⁻²) |
| `alpha` | Reliability (α) |
| `pi_rel` | Relative resolution (π_rel) |

### Dynamic hydrographs

Pre-generated interactive hydrographs (Plotly `.html`) are stored in [`results/hydrographs/`](results/hydrographs/). Each file contains the observed streamflow, the TFT median prediction, the full quantile envelope (1–99%, 5–95%, 10–90%, 25–75%, 40–60%), and the HBV deterministic prediction. Open any `.html` file directly in a browser — no server required.

To regenerate or customise the hydrographs, run [`05_dynamic_hydrographs.ipynb`](notebooks/05_dynamic_hydrographs.ipynb).

---

## Citation

If you use this code or data in your work, please cite:

```bibtex
@article{francisco2026tft_ungauged,
  author  = {Francisco, Rafael and Matos, Jos{\'e} Pedro},
  title   = {Streamflow prediction in natural ungauged catchments using Temporal Fusion Transformers},
  journal = {Hydrology and Earth System Sciences},
  year    = {2026},
  doi     = {XXXX},
}
```

The TFT architecture is described in:

```bibtex
@article{lim2021tft,
  author  = {Lim, Bryan and Ar{\i}k, Sercan {\"O}. and Loeff, Nicolas and Pfister, Tom},
  title   = {Temporal Fusion Transformers for interpretable multi-horizon time series forecasting},
  journal = {International Journal of Forecasting},
  volume  = {37},
  pages   = {1748--1764},
  year    = {2021},
  doi     = {10.1016/j.ijforecast.2021.03.012},
}
```

---

## Funding

This research was funded by the Fundação para a Ciência e a Tecnologia, I.P. (FCT) under Grant 2025.00562.BD, through the funding project PREDICT (1801P.01563) and the project UID/6438/2025 of the research unit CERIS.

---

## License

[MIT](LICENSE) — see the file for details.

The hydrological data originate from SNIRH (Portuguese Environment Agency) and are shared under their respective terms of use. The ERA5-Land dataset is provided by ECMWF under the [Copernicus licence](https://cds.climate.copernicus.eu/api/v2/terms/static/licence-to-use-copernicus-products.pdf). The CORINE Land Cover and Copernicus DEM data are distributed under open-access terms by the European Environment Agency and the European Space Agency, respectively.
