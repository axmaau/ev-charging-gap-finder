# ev-charging-gap-finder

Analyzes U.S. EV charging infrastructure using open data from the Department of Energy’s AFDC and NREL. Builds a reproducible data pipeline (SQL + dbt) that cleans and models station data, joins with Census population and highway corridor maps, and highlights underserved counties and corridor gaps. Outputs interactive dashboards and a 1-page memo showing where new fast-charging stations would have the biggest impact.


## What’s different from AFDC / EV Hub / EVI-Pro Lite?

**This repo is an open, reproducible pipeline** that merges *supply* (AFDC stations) with *need* (EVI-Pro Lite) and *accessibility* (corridor spacing + per-capita), then ships tested marts and a stakeholder-ready dashboard. Most public maps or reports show pieces; this repo puts them together end-to-end.

### How it differs
- **AFDC Station Locator:** The canonical map of stations, but not a reproducible ETL or per-capita/corridor gap analysis with tests. This project ingests AFDC, normalizes connectors/ports, and builds shareable marts.
- **Atlas EV Hub & similar dashboards:** Great visuals, but not an OSS repo you can clone, run nightly, and extend with your own tests/metrics. Here, the dbt models, tests, CI, and BI files live in one place.
- **EVI-Pro Lite:** Provides *need* estimates, not supply integration. This repo computes **Need vs Have** and a simple **Gap Index** to prioritize geographies.
- **Joint Office corridor maps / legacy 50-mile benchmark:** Official corridor views, not a repo. This project computes both **legacy pass/fail (≈50 mi / 1-mi off-corridor)** and a **graded distance view** aligned with more flexible 2025 guidance.
- **Academic/state NEVI studies:** Insightful PDFs, but not productionized pipelines. Here you get code, tests, and outputs you can re-run.

### What you get here (at a glance)
- **Reproducible stack:** AFDC/EVI-Pro/ACS → dbt models + tests → marts → dashboard + 1-page memo.
- **Per-capita coverage:** County/state DC-fast ports per 100k, DC-fast share, and Top-10 under-served lists.
- **Corridor gaps (two lenses):** Legacy 50-mile compliance **and** distance gradients (50/75/100 mi).
- **Need vs Have:** Join EVI-Pro Lite estimates to current AFDC ports to surface a prioritized **gap index**.
- **Equity-ready (optional):** ACS layers (e.g., renters/no-garage proxy) to contextualize siting.
- **Ops hygiene:** Daily snapshot option, CI hooks, and clear METHODS/NOTICE for data provenance.

> **Why it matters:** Planners and operators don’t just need dots on a map, they need a ranked, defensible list of *where the next few sites unlock the most benefit*. This repo delivers that, with code you can audit and extend.






Short description, data sources, quickstart:

make setup          # create venv, install deps
make ingest         # AFDC/Census/AFC → bronze
make stage          # bronze → staging
make marts          # staging → marts
make docs           # build memo/dbt docs (optional)

**License**
- Code: MIT
- Docs/visuals: CC BY 4.0
- Data: © respective data providers (AFDC, NREL EVI-Pro Lite, Census, FHWA). See NOTICE.

Charging Coverage & Gap Finder
Data: AFDC All Stations API; FHWA AFC lines; Census/ACS; NREL EVI-Pro Lite. 
NREL Developer Network
Data.gov
Census.gov
NREL

Metrics: DC fast ports per 100k residents (county), DC fast share, corridor gap length (legacy 50-mile benchmark + 1-mile offset), state need vs have. 
Congress.gov

Run: make setup && make ingest && dbt build
Outputs: /dash (PBIX/TWBX), /docs/memo.pdf, screenshots in README.

