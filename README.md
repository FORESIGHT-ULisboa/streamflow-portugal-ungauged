<p align="center">
  <img src="images/foresight.png" alt="FORESIGHT" width="320">
</p>


# Streamflow Prediction in Natural Ungauged Catchments using Temporal Fusion Transformers

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-yellow.svg)](LICENSE)
[![Python 3.8](https://img.shields.io/badge/python-3.8-blue.svg)](https://www.python.org/)

**`streamflow-portugal-ungauged`** is the official code and data repository for the paper:

> **Francisco, R. and Matos, J. P.** — *Streamflow prediction in natural ungauged catchments using Temporal Fusion Transformers*, 2026

This repository contains the input data, conda environment, pre-computed model outputs — TFT predictions and metrics alongside the HBV benchmark — and a visualization notebook to inspect all findings reported in the paper, including **ungauged prediction** (leave-one-group-out cross-validation) and **model specialization** (transfer-learning fine-tuning with limited local data).

---

## Table of contents

- [Overview](#overview)
- [Repository structure](#repository-structure)
- [Installation](#installation)
- [Data](#data)
- [Results](#results)
- [Visualizing predictions](#visualizing-predictions)
- [Citation](#citation)
- [Funding](#funding)
- [License](#license)

---

## Overview

Streamflow prediction in ungauged catchments remains one of the central challenges in operational hydrology. This work evaluates the ability of **Temporal Fusion Transformers (TFT, Lim et al., 2021)** to generalize hydrological knowledge across **53 unregulated catchments in mainland Portugal** under strictly ungauged conditions, and further explores how limited local observations can be used to specialize a pre-trained regional model.

Key results on the test subsets, for catchments in complete ungauged conditions (100% of the time series for testing), are summarized in the table below. The TFT model outperforms the HBV benchmark on the majority of catchments, despite HBV being calibrated with full local data and therefore representing an unfair upper-bound benchmark.

| Model | Median NSE | Median KGE | Median CRPS (×10³ m³ s⁻¹ km⁻²) |
|---|---|---|---|
| HBV (individually calibrated — upper bound) | 0.46 | 0.44 | 5.51 |
| **TFT Ungauged** | **0.50** | **0.48** | **3.75** |

When limited local data is available, the TFT model can be specialized via transfer learning. The table below shows the median performance of the TFT model after specialization with 60% of data for training, 20% for validation, and 20% for testing. The TFT specialized model outperforms the ungauged TFT on all metrics. The showed results and metrics correpond to the 20% subset used for testing.

| Model | Median NSE | Median KGE | Median CRPS (×10³ m³ s⁻¹ km⁻²) |
|---|---|---|---|
| HBV (individually calibrated — upper bound) | 0.48 | 0.40 | 5.16 |
| **TFT Ungauged** | **0.48** | **0.44** | **3.59** |
| **TFT Specialized** | **0.62** | **0.65** | **3.20** |

> **Note:** HBV is calibrated with full local data and therefore represents an unfair, upper-bound benchmark for the ungauged TFT. Despite this, the TFT ungauged model performs comparably or better on the majority of catchments.

---

## Repository structure

```
streamflow-portugal-ungauged/
├── data/
│   ├── hydrological/               # Daily streamflow from SNIRH — one CSV per catchment + README
│   ├── meteorological/             # ERA5-Land daily precipitation and temperature — one CSV per catchment + README
│   ├── geomorphological/           # catchment_descriptors.csv (area, elevation, land use, …) + README
│   └── shapefile/                  # Catchment boundary polygons (catchments.shp)
├── notebooks/
│   └── 00_prediction_plots.ipynb   # Interactive plots: TFT (ungauged/specialized) vs observations and HBV
├── results/
│   ├── hbv/
│   │   ├── metrics/                # HBV NSE/KGE/CRPS — full record and test-split variants
│   │   └── prediction/             # Calibration outputs, one subfolder per catchment + README
│   ├── tft/
│   │   ├── metrics/                # metrics_Q_A.xlsx per configuration (ungauged/specialization)
│   │   └── prediction/             # Daily quantile predictions (ungauged/specialization)
│   └── README.md                   # Full description of the results structure and file formats
├── images/
├── environment.yml                 # Conda environment (Python 3.8, PyTorch + CUDA 11.8, full stack)
├── AGENTS.md                       # Conventions for AI coding agents
├── CLAUDE.md                       # → points to AGENTS.md
├── LICENSE                         # GNU GPL v3
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

All dependencies — PyTorch, PyTorch Forecasting, CMA-ES, xarray/GDAL, and the full scientific stack — are declared in [`environment.yml`](environment.yml). The environment is named `environment`.

```bash
conda env create -f environment.yml
conda activate environment
```

> **GPU support:** the environment installs CUDA 11.8-enabled PyTorch by default. On a CPU-only machine, remove the `cuda`, `cudatoolkit`, and `pytorch-cuda` entries from `environment.yml` before creating the environment (see [pytorch.org](https://pytorch.org/get-started/locally/) for alternatives matching your setup).

### 3 · Open the notebook

Open [`notebooks/00_prediction_plots.ipynb`](notebooks/00_prediction_plots.ipynb) directly in VS Code and select the `environment` kernel, or launch JupyterLab:

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

Raw files are stored in `data/hydrological/` as one CSV per catchment, named by station code (e.g., `03J_01H.csv`). See [`data/hydrological/README.md`](data/hydrological/README.md) for the column schema and the quality-control log.

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

### Catchment boundaries

The delineated catchment polygons used for the spatial aggregation of ERA5-Land and for the map figures are provided as a shapefile in [`data/shapefile/`](data/shapefile/) (`catchments.shp`, WGS84).

---

## Results

All model outputs and performance metrics are pre-computed and included in [`results/`](results/) — no retraining is needed to inspect or reuse the paper's findings. See [`results/README.md`](results/README.md) for the complete description of the folder layout, file formats, and naming conventions.

| Folder | Contents |
|---|---|
| [`results/hbv/prediction/`](results/hbv/prediction/) | HBV calibration outputs per catchment (simulation, optimal forcing, best parameters) — detailed in its own [README](results/hbv/prediction/README.md) |
| [`results/hbv/metrics/`](results/hbv/metrics/) | HBV NSE, KGE, and CRPS per station — over the full record (ungauged comparison) and over the 20 % test split (specialization comparison) |
| [`results/tft/metrics/`](results/tft/metrics/) | TFT `metrics_Q_A.xlsx` files — one per configuration (ungauged) and one per configuration × station × retraining version (specialization) |
| [`results/tft/prediction/`](results/tft/prediction/) | Daily TFT quantile predictions (0.01–0.99) with observations and split labels, one CSV per station |

Key conventions (full details in [`results/README.md`](results/README.md)):

- Station codes follow SNIRH with the slash replaced by an underscore in file names (`03J/01H` → `03J_01H`).
- All `metrics_Q_A.xlsx` files retain only the **leadtime = 3 days** columns.
- Streamflow is expressed as specific streamflow: q = Q / A × 10³, in 10⁻³ m³ s⁻¹ km⁻².

---

## Visualizing predictions

The notebook [`00_prediction_plots.ipynb`](notebooks/00_prediction_plots.ipynb) builds interactive plots of the TFT streamflow predictions against observations and the HBV benchmark for any station. Choose the **experiment** (`ungauged` or `specialized`) and the **station** (SNIRH code) in the notebook, and it renders the observed series, the TFT median prediction with its quantile envelope, and the HBV deterministic simulation from the pre-computed files in [`results/`](results/).

---

## Citation

If you use this code or data in your work, please cite:

```bibtex
@article{francisco2026tft_ungauged,
  author  = {Francisco, Rafael and Matos, Jos{\'e} Pedro},
  title   = {Streamflow prediction in natural ungauged catchments using Temporal Fusion Transformers},
  journal = {xxxx},
  year    = {xxxx},
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

This research was funded by the Portuguese Foundation for Science and Technology (Fundação para a Ciência e a Tecnologia, I.P. - FCT, https://ror.org/00snfqn5816) under Grant 2025.00562.BD with DOI identifier https://doi.org/10.54499/2025.00562.BD, through the funding project PREDICT (LISBOA2030-FEDER-00856400) and the project UID/6438/2025 of the research unit CERIS.

---

## License

This repository is licensed under the **[GNU General Public License v3.0](LICENSE)** — you are free to use, modify, and redistribute the code, provided derivative works remain under the same license. See the [LICENSE](LICENSE) file for the full terms.

The hydrological data originate from SNIRH (Portuguese Environment Agency) and are shared under their respective terms of use. The ERA5-Land dataset is provided by ECMWF under the [Copernicus licence](https://cds.climate.copernicus.eu/api/v2/terms/static/licence-to-use-copernicus-products.pdf). The CORINE Land Cover and Copernicus DEM data are distributed under open-access terms by the European Environment Agency and the European Space Agency, respectively.
