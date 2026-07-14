# Meteorological Data

## Source

Hourly air temperature and precipitation were retrieved from the **ERA5-Land** reanalysis dataset:

> Muñoz Sabater, J., Dutra, E., Agustí-Panareda, A., et al. (2021). ERA5-Land: a state-of-the-art global reanalysis dataset for land applications. *Earth System Science Data*, 13, 4349–4383. https://doi.org/10.5194/essd-13-4349-2021

ERA5-Land is produced by the European Centre for Medium-Range Weather Forecasts (ECMWF) and distributed through the [Copernicus Climate Data Store (CDS)](https://cds.climate.copernicus.eu/).

## Coverage

| Property | Value |
|---|---|
| Variables retrieved | 2 m air temperature (K, hourly); total precipitation (m, hourly) |
| Spatial resolution | ≈ 9 km (0.1° × 0.1° regular grid) |
| Period | 1980–2024 |
| Spatial aggregation | Catchment-mean (area-weighted average over each catchment polygon) |

## Processing steps

1. **Spatial aggregation:** ERA5-Land gridded data were aggregated to a single catchment-mean value using the catchment polygon boundaries (one value per catchment per hour).
2. **Unit conversion:** temperature converted from K to °C; precipitation from m to mm.
3. **Temporal aggregation:** hourly values were aggregated to daily totals (precipitation) and daily means (temperature).
4. **Time zone:** all hourly records were converted from UTC to local Portuguese time (UTC+1 standard, UTC+2 during DST) before daily aggregation to ensure hydrological consistency with the streamflow records.

No additional quality-control procedures were applied to ERA5-Land, given its demonstrated acceptable performance over Portugal (Almeida and Coelho, 2023; Francisco and Matos, 2024) and the robustness of the TFT as a data-driven model to residual biases.

## File format

One CSV file per station, named `<STATION_CODE>.csv` (e.g., `03J01H.csv`).

| Column | Units | Description |
|---|---|---|
| `date` | YYYY-MM-DD | Calendar date (local time) |
| `precip_mm` | mm | Daily total precipitation |
| `temp_mean_C` | °C | Daily mean 2 m air temperature |
| `temp_max_C` | °C | Daily maximum 2 m air temperature (used by HBV for PET) |
| `temp_min_C` | °C | Daily minimum 2 m air temperature (used by HBV for PET) |

## Static meteorological descriptors

Two static descriptors derived from the full ERA5-Land record (1980–2024) are included in [`../geomorphological/catchment_descriptors.csv`](../geomorphological/catchment_descriptors.csv):

| Descriptor | Description |
|---|---|
| `avg_annual_precip_mm` | Mean annual precipitation averaged over the full record |
| `avg_annual_temp_C` | Mean annual temperature averaged over the full record |

These are used as static real inputs to the TFT (see Table 4 in the paper).
