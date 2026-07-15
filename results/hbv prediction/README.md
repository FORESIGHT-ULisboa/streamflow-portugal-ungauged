# HBV Prediction

Calibration outputs of the **HBV conceptual rainfall–runoff model** (Bergström, 1976) for 56 unregulated catchments in mainland Portugal. This folder provides the **benchmark** used throughout the paper (Section 2.3.1 and Section 4.1).

HBV was calibrated individually per catchment using **CMA-ES** (Hansen, 2016), maximising NSE over the full available streamflow record. The eWaterCycle package (Hut et al., 2022) was used as the HBV implementation (HBV-bmi, original 1976 formulation).

---

## Structure

One subfolder per catchment, named by SNIRH station code with underscores:

```
hbv prediction/
├── 02G_01H/
│   ├── best_parameters.txt
│   ├── 02G_01H_HBV_simulation.csv
│   └── 02G_01H_optimal_forcing.txt
├── 03I_03H/
│   └── ...
└── ...                          (56 catchments total)
```

---

## File descriptions

### `best_parameters.txt`

Plain-text summary of the CMA-ES calibration run. Example for station `02G_01H`:

```
Optimization Summary for Group: 02G/01H
========================================
Objective function value: NSE
Best metric: 0.2923
Total Iterations: 55
Total Fun Evals: 1540
Execution Time: 00:17:24
Stop Reason: {'tolfun': 0.001}
========================================
Best Parameters:
Imax: 0.0876
Ce: 1.5000
Sumax: 156.2324
Beta: 1.5788
Pmax: 0.0000
Tlag: 0.5739
Kf: 0.2499
Ks: 0.0701
FM: 3.2036
Pmult: 0.8953
ETmult: 1.2734
```

The 11 calibrated parameters and their physical meaning:

| Parameter | Units | Description |
|---|---|---|
| `Imax` | mm | Maximum interception storage |
| `Ce` | — | Evaporation correction factor |
| `Sumax` | mm | Maximum soil moisture storage |
| `Beta` | — | Soil drainage exponent |
| `Pmax` | — | Percolation fraction |
| `Tlag` | days | Routing lag time |
| `Kf` | day⁻¹ | Fast reservoir coefficient |
| `Ks` | day⁻¹ | Slow reservoir coefficient |
| `FM` | mm °C⁻¹ day⁻¹ | Snowmelt factor |
| `Pmult` | — | Precipitation multiplier (applied to ERA5-Land forcing) |
| `ETmult` | — | Potential evapotranspiration multiplier |

---

### `<STATION>_HBV_simulation.csv`

Daily simulated streamflow with the optimal parameter set.

| Column | Units | Description |
|---|---|---|
| `Event date` | YYYY-MM-DD | Calendar date (local time) |
| `0` | m³ s⁻¹ | Simulated raw streamflow Q |

The first simulation day is always one day after the start of the forcing record — day 1 is discarded as spin-up. To convert to specific streamflow for comparison with TFT outputs, apply:

```
q  [10⁻³ m³ s⁻¹ km⁻²]  =  Q  [m³ s⁻¹]  /  A  [km²]  ×  10³
```

where A is the catchment drainage area from `data/geomorphological/catchment_descriptors.csv`.

---

### `<STATION>_optimal_forcing.txt`

Tab-separated daily meteorological forcing with `Pmult` and `ETmult` already applied. No header row.

| Column | Units | Description |
|---|---|---|
| 1 | — | Year |
| 2 | — | Month (1–12) |
| 3 | — | Day (1–31) |
| 4 | mm | Precipitation × Pmult |
| 5 | °C | Mean daily air temperature |
| 6 | mm | Potential evapotranspiration × ETmult |

The raw forcing (before multipliers) comes from ERA5-Land (Muñoz Sabater et al., 2021). PET was computed using the Hargreaves–Samani method (Hargreaves and Samani, 1985; Eq. 3–7 in the paper).

---

## Calibration settings

| Setting | Value |
|---|---|
| Algorithm | CMA-ES (Hansen, 2016), Python `cma` package v4.1.0 |
| Objective | NSE, maximised |
| Max function evaluations | 4 000 |
| Convergence tolerance | 10⁻³ (objective improvement) |
| Calibration data | Full available record per catchment (1980–2024) |
| Spin-up | First day of each simulation discarded |

Because HBV is calibrated with **all available local data**, it represents an unfair but useful **upper-bound benchmark** for the ungauged TFT, which has no access to the target catchment's streamflow during training.

---

## References

- Bergström, S. (1976). *Development and Application of a Conceptual Runoff Model for Scandinavian Catchments*. SMHI Reports RHO 7, Norrköping.
- Hansen, N. (2016). The CMA Evolution Strategy: A Tutorial. https://doi.org/10.48550/arXiv.1604.00772
- Hargreaves, G. H. and Samani, Z. A. (1985). Reference crop evapotranspiration from temperature. *Applied Engineering in Agriculture*, 1, 96–99.
- Hut, R. et al. (2022). The eWaterCycle platform for open and FAIR hydrological collaboration. *Geoscientific Model Development*, 15, 5371–5390.
- Muñoz Sabater, J. et al. (2021). ERA5-Land: a state-of-the-art global reanalysis dataset for land applications. *Earth System Science Data*, 13, 4349–4383.
