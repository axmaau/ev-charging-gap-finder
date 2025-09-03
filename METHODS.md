# METHODS

## Purpose
Quantify EV fast-charging coverage and corridor gaps in the U.S. using open data, then surface a ranked list of places where new sites add the most value.

## Data Sources
- DOE Alternative Fuels Data Center (AFDC) – station locations/attributes.
- NREL EVI-Pro Lite – state-level charging need estimates.
- U.S. Census / ACS – county population (normalization).
- FHWA Alternative Fuel Corridors (AFC) – corridor polylines.

## Scope & Filters
- Geography: United States (50 states + DC).
- Stations: `fuel_type = ELEC`, `access = public`, `status = E` (available).
- Time: snapshot at ingest time; repo stores daily snapshots in `data/bronze/`.

## Pipeline Overview
**Bronze (raw snapshots)**  
- AFDC API JSON saved as-is; ACS CSV; AFC GeoJSON.
  
**Staging (standardized)**  
- Parse connector types → flags (`is_ccs`, `is_nacs`, `is_chademo`).
- Compute `dc_fast_ports = ev_dc_fast_num`, `level2_ports = ev_level2_evse_num`.
- Geocode to county FIPS via point-in-polygon (EPSG:4326 → common CRS).
- Deduplicate by `id` and (lat, lon); keep latest `last_updated`.

**Marts (analysis-ready)**  
- `county_coverage`: aggregates by county; computes per-capita metrics.
- `corridor_gaps_legacy`: measures along-corridor distances between nearest DCFC sites; flags segments >50 miles or >1 mile off-corridor (legacy benchmark).
- `corridor_gaps_gradient`: reports distance bands (e.g., 50/75/100 mi) for flexible 2025 guidance.
- `state_need_vs_have`: joins EVI-Pro Lite “needed DCFC” vs AFDC “existing DCFC”.

## Metric Definitions
- **DC fast ports per 100k**:  
  `ports_per_100k = (dc_fast_ports / population) * 100000`
- **DC fast share**:  
  `dc_fast_share = dc_fast_ports / (dc_fast_ports + level2_ports)`
- **Legacy corridor gap (pass/fail)**: segment fails if **adjacent DCFC sites along the corridor are >50 miles apart** or the nearest site lies **>1 mile** from the corridor centerline.
- **Distance gradient view**: report the computed inter-site distances; color-band at 50/75/100 mi thresholds (context for newer guidance).
- **Need vs Have**:  
  `gap = (EVI-Pro Lite required DCFC) − (AFDC existing DCFC)` at state level.
- **(Optional) Gap Index** (example weighting—tunable):  
  Normalize per-capita shortage (z-score), corridor distance (min-max), and need ratio;  
  `gap_index = 0.5*z_per_capita_shortage + 0.3*scaled_corridor_distance + 0.2*need_ratio`.

## Data-Quality Checks
- Freshness: AFDC `last_updated` mean ≤ 7 days.
- Valid ranges: lat/lon within U.S. bounds; ports ≥ 0.
- Keys: not-null station `id`, unique county FIPS.
- Reconciliation: AFDC `station_counts` vs ingested row counts.

## Geo Methods
- CRS: ingest in EPSG:4326; compute distances on the projected CRS appropriate for each corridor segment or via geodesic haversine along polyline.
- Corridor proximity: buffer corridor centerline by 1 mile for legacy test; nearest-neighbor search for DCFC stations.

## Limitations
- AFDC power values and connector metadata can lag reality.
- EVI-Pro Lite provides state-level needs; downscaling below the state level introduces uncertainty.
- Corridor shapefiles update periodically; distances reflect snapshot date.
- Availability/reliability is not modeled beyond `status = E`.

## Reproducibility
- `Makefile` targets orchestrate ingest → staging → marts.
- dbt models/tests define transformations and constraints.
- CI runs lint/tests and (optionally) `dbt build` on PRs.

## Versioning
- Record data snapshot date in release notes.
- Breaking changes to metric formulas are version-tagged.
