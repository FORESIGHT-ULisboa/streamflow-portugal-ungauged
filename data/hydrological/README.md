# Hydrological Data

## Source

Daily streamflow records were retrieved from the **Sistema Nacional de Recursos Hídricos (SNIRH)**, operated by the Portuguese Environment Agency (Agência Portuguesa do Ambiente, APA):

> Portuguese Environment Agency (2024). *Sistema Nacional de Recursos Hídricos (SNIRH): Daily Streamflow Records* [Dataset]. https://snirh.apambiente.pt/ (accessed 21 March 2024).

Note: Only accessible to users located in Portugal. The presented dataset is already pre-processed and quality-controlled, and is provided here for research purposes only. Please contact APA for any questions regarding the original data.

## Coverage

| Property | Value |
|---|---|
| Variable | Average daily streamflow (m³ s⁻¹) |
| Period | 1980–2024 |
| Number of catchments | 53 unregulated (natural) catchments, mainland Portugal |
| Minimum record per catchment | 10 years (not necessarily continuous) |
| Spatial extent | 37°N–42°N, 7°W–9.5°W (WGS84) |

## File format

One CSV file per station, named `<STATION_CODE>.csv` (e.g., `03J_01H.csv`).

| Column | Units | Description |
|---|---|---|
| `Event date` | YYYY-MM-DD | Calendar date (local time) |
| `Q` | m³ s⁻¹ | Average daily streamflow |
| `Q/A` | 10⁻³ m³ s⁻¹ km⁻² | Specific streamflow (Q normalized by drainage area × 10³) |

## Quality control

A conservative qualitative protocol was applied to every time series. An entire hydrological year (1 Oct – 30 Sep) was removed if any of the following anomalies were detected within it:

- Unexpected step changes or spikes inconsistent with observed patterns in the series.
- Consecutive years with blocks of null values (instrument failure or data gap not attributable to natural low flow).
- Unexplained offsets during the low-flow season, potentially indicating sensor drift or rating-curve miscalibration.

Missing values resulting from QC removal were **not interpolated**. Seven additional catchments identified during post-hoc reassessment are excluded from the main results (see supplementary material in the paper).

## Normalization

To facilitate cross-catchment comparison and to prevent drainage-area-driven dominance during TFT training, streamflow was converted to specific streamflow (Eq. 2 in the paper):

```
q = (Q / A) × 10³
```

where `Q` is in m³ s⁻¹ and `A` is the catchment drainage area in km². The factor 10³ normalizes to a reference area of 1000 km².

## Catchment list and cluster assignment

The 53 catchments were divided into four geographical clusters (A–D) used for the leave-one-group-out cross-validation. Cluster assignment and basic catchment statistics are compiled in [`../geomorphological/catchment_descriptors.csv`](../geomorphological/catchment_descriptors.csv).
