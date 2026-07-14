# HBV Per-Station Results

This folder contains calibration and simulation outputs from the HBV model,
organized by gauging station, for the ungauged streamflow prediction work
(`streamflow-portugal-ungauged`).

## Folder structure

```
results/hbv/
├── <station_name_1>/
│   ├── <station_name_1>_HBV_simulation.csv
│   ├── best_parameters.txt
│   └── <station_name_1>_optimal_forcing.txt
├── <station_name_2>/
│   ├── ...
└── ...
```

Each subfolder corresponds to one gauging station. Station names follow the
same naming convention used elsewhere in this repository

## File descriptions

### `*_HBV_simulation.csv`
HBV simulated in m³/s for the station. The file contains a time series of daily streamflow values over the simulation period.

### `best_parameters.txt`
The parameter set selected as optimal during HBV calibration for this station,
along with the calibration criterion used to select it.

### `*_optimal_forcing.txt`
The forcing (meteorological input) dataset identified as best suited for
driving the HBV model at this station (already with multiplication factors for evapotranspiration and precipitation applied).