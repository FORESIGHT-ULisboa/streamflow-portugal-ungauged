# AGENTS.md — Conventions for AI coding agents

This file records project conventions and constraints for AI coding assistants (GitHub Copilot, Claude Code, etc.). All agents operating on this repository **must read and follow this file before making any changes**.

---

## Project purpose

This repository reproduces the experiments of Francisco and Matos (2026) on streamflow prediction in ungauged catchments using Temporal Fusion Transformers. The code covers data preprocessing, HBV calibration, TFT training, and performance evaluation.

---

## Python style

- Python 3.10. Type hints everywhere (PEP 484).
- `black` formatting, line length 100.
- `ruff` for linting (rules: `E`, `F`, `I`, `UP`).
- Docstrings follow NumPy style.
- All public functions must have a docstring.

---

## Repository layout rules

- **Do not commit** large binary files (models, `.pkl`, `.nc` rasters). Use Git LFS or provide download instructions.
- **Do not commit** raw ERA5-Land `.nc` files — provide the preprocessed CSVs only.
- **Do not commit** secrets, API keys, or credentials.
- Notebooks go in `notebooks/`. Each notebook must be self-contained and reproducible when run top-to-bottom with a clean kernel.
- Results (tables, hydrographs) go in `results/`. Pre-computed outputs are versioned; regeneration scripts are in notebooks.

---

## Hydrological conventions

- Streamflow is always stored and processed as **specific streamflow** `q` (m³ s⁻¹ km⁻²) × 10³ unless explicitly stated otherwise.
- The **hydrological year** in Portugal runs from 1 October to 30 September.
- Performance metrics follow Table 3 of the paper: NSE (optimal = 1), KGE (optimal = 1), CRPS (optimal = 0), α reliability (optimal = 1), π_rel (optimal = +∞).
- When reporting results, **always state whether they correspond to the full time series or the 20% test portion**, as these are systematically different (see §4.1 vs §4.2 of the paper).

---

## Cross-validation and data-leakage rules

- The leave-one-group-out split is **spatial**: test catchments are withheld entirely from training.
- Within calibration catchments, the temporal split uses **strides of 3 months** aligned to the hydrological year. Stride length must not be reduced below 3 months (risk of autocorrelation leakage).
- Standardization statistics (mean, std) are computed **exclusively from training catchments** within each fold. Never use statistics from test or validation catchments.
- The test set is **never touched** during hyperparameter selection. Only the spatial validation group (100% validation catchments) is used for model selection.

---

## Naming conventions

| Entity | Convention | Example |
|---|---|---|
| Station code | SNIRH format | `03J01H` |
| Model variants | Snake-case | `tft_ungauged`, `tft_specialized`, `hbv` |
| Configuration index | 1-based integer θ | `config_01` … `config_14` |
| Run index | 1-based integer | `run_01` … `run_10` |
| Best model per config | `m{θ}` | `m1`, `m14` |

---

## Testing

- Unit tests live in `tests/` (not yet populated — contributions welcome).
- Run with `pytest tests/ -v`.
- Before opening a PR, run `black . && ruff check .` and fix all issues.

---

## Questions

Open an issue or contact rafael.francisco@tecnico.ulisboa.pt.
