# ev-charging-gap-finder

Analyzes U.S. EV charging infrastructure using open data from the Department of Energy’s AFDC and NREL. Builds a reproducible data pipeline (SQL + dbt) that cleans and models station data, joins with Census population and highway corridor maps, and highlights underserved counties and corridor gaps. Outputs interactive dashboards and a 1-page memo showing where new fast-charging stations would have the biggest impact.
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

