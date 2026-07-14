# Geomorphological Data

## Overview

Catchment descriptors were computed for each of the 53 study catchments. They are used as **static inputs** to the TFT model (Table 4 in the paper) and provide the geomorphological context needed for spatial generalization under ungauged conditions.

## File

All descriptors are compiled in **`catchment_descriptors.csv`** (one row per catchment).

## Column schema

| Column | Type | Units | Source | Description |
|---|---|---|---|---|
| `station` | str | — | SNIRH | Station code (e.g., `03J/01H`) |
| `cluster` | str (A–D) | — | This study | Geographic cluster for leave-one-group-out CV |
| `area` | float | km² | DEM-derived | Drainage area |
| `gravelius_index` | float | — | Computed | Gravelius compactness index (perimeter / (2√(π·A))) |
| `centroid_lat` | float | ° (WGS84) | DEM-derived | Catchment centroid latitude |
| `centroid_lon` | float | ° (WGS84) | DEM-derived | Catchment centroid longitude |
| `mean_elev` | float | m | Copernicus DEM GLO-30 | Mean elevation |
| `main_land_use` | str (categorical) | — | CORINE 2018 | Dominant land-cover class |
| `avg_annual_tp` | float | mm | ERA5-Land | Mean annual precipitation (1980–2024) |
| `avg_annual_t2m` | float | °C | ERA5-Land | Mean annual temperature (1980–2024) |

## Data sources

| Descriptor | Source | Reference |
|---|---|---|
| Drainage area, centroid, elevation | Copernicus DEM GLO-30 | European Space Agency (2019). https://doi.org/10.5270/ESA-c5d3d65 |
| Main land use | CORINE Land Cover 2018 | European Environment Agency (2018). https://doi.org/10.2909/71c95a07-e296-44fc-b22b-415f42acfdf0 |
| Gravelius compactness index | Computed from polygon | Gravelius (1914) |
| Average annual climate | ERA5-Land (1980–2024 mean) | Muñoz Sabater et al. (2021) |

## Notes on the Gravelius compactness index

The Gravelius index *G* is defined as:

```
G = P / (2 √(π · A))
```

where *P* is the catchment perimeter (km) and *A* is the drainage area (km²). A value of 1 corresponds to a perfectly circular catchment; values > 1 indicate more elongated or irregular shapes. It is used here as a compact shape descriptor and static real input to the TFT.

## CORINE land-use classes

The dominant land-use class was assigned as the CORINE 2018 Level-1 class covering the largest fraction of the catchment area:

| Code | Class |
|---|---|
| 111 | Continuous urban fabric |
| 112 | Discontinuous urban fabric |
| 121 | Industrial or commercial units |
| 122 | Road and rail networks and associated land |
| 123 | Port areas |
| 124 | Airports |
| 131 | Mineral extraction sites |
| 132 | Dump sites |
| 133 | Construction sites |
| 141 | Green urban areas |
| 142 | Sport and leisure facilities |
| 211 | Non-irrigated arable land |
| 212 | Permanently irrigated land |
| 213 | Rice fields |
| 221 | Vineyards |
| 222 | Fruit trees and berry plantations |
| 223 | Olive groves |
| 231 | Pastures |
| 241 | Annual crops associated with permanent crops |
| 242 | Complex cultivation patterns |
| 243 | Land principally occupied by agriculture, with significant areas of natural vegetation |
| 244 | Agro-forestry areas |
| 311 | Broad-leaved forest |
| 312 | Coniferous forest |
| 313 | Mixed forest |
| 321 | Natural grasslands |
| 322 | Moors and heathland |
| 323 | Sclerophyllous vegetation |
| 324 | Transitional woodland-shrub |
| 331 | Beaches, dunes, sands |
| 332 | Bare rocks |
| 333 | Sparsely vegetated areas |
| 334 | Burnt areas |
| 335 | Glaciers and perpetual snow |
| 411 | Inland marshes |
| 412 | Peat bogs |
| 421 | Salt marshes |
| 422 | Salines |
| 423 | Intertidal flats |
| 511 | Water courses |
| 512 | Water bodies |
| 521 | Coastal lagoons |
| 522 | Estuaries |
| 523 | Sea and ocean |
| 999 | NODATA |
| 990 | UNCLASSIFIED LAND SURFACE |
| 995 | UNCLASSIFIED WATER BODIES |
