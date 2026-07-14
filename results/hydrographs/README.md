# Results — Dynamic Hydrographs

This directory contains pre-generated interactive Plotly hydrographs (`.html`) for all study catchments.

## How to use

Simply open any `.html` file in a web browser (Chrome, Firefox, Safari, Edge). No server, Python installation, or internet connection is required — Plotly is bundled via CDN link.

Each hydrograph includes:

- **Observed streamflow** (black line)
- **TFT median prediction** (blue dashed line)
- **Full quantile envelope** (shaded bands: 1–99%, 5–95%, 10–90%, 25–75%, 40–60%)
- **HBV deterministic prediction** (orange line)
- **Daily precipitation** (inverted bar chart on secondary axis)

Interactive features:
- Zoom and pan with mouse
- Hover for per-day values
- Toggle traces on/off by clicking the legend

## Directory structure

```
hydrographs/
├── ungauged/       # One .html per catchment — TFT ungauged (full time series)
└── specialized/    # One .html per catchment — TFT specialized (20% test portion)
```

## Regenerating hydrographs

To regenerate from model outputs (requires pre-computed `.p` files in `results/model_outputs/`):

```bash
jupyter nbconvert --to notebook --execute notebooks/05_dynamic_hydrographs.ipynb
```

## File naming

Files are named by SNIRH station code (e.g., `03J01H.html`). The station code, river name, and cluster are described in [`../../data/geomorphological/README.md`](../../data/geomorphological/README.md).
