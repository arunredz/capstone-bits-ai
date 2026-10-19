# L3: Explore the Data and Trace Its Lineage

- **Cloud required:** No; initial source download requires internet.
- **Outcome:** Three findings about cleaning, demand, and feature/split behavior,
  each grounded in the executed notebook or persisted pipeline evidence.
- **Recording companion:** Follow the finalized L3 route below. L4 revisits the
  persistence and consistency checks; it does not replace this exploratory walkthrough.

## 1. Read the Complete Route

Open the [lineage diagram](data-lineage-route.png). Trace source, import,
cleaning, the demand/fare branch, chronological splits, and downstream consumers.
Distinguish transformation arrows from reference/evidence arrows.

Keep the three row meanings explicit:

- Raw trips: one published completed trip.
- Demand: one observed pickup zone during one hour, including inserted zero counts.
- Fare: one accepted completed trip after cleaning, targeting `fare_amount`.

## 2. Verify the Official Source

Open the [NYC TLC Trip Record Data page](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).
Identify Yellow Taxi trip Parquet and the taxi-zone lookup CSV. The recording
uses January 2024; this is the configured teaching snapshot, not a claim that it
is the latest publication. Zones are broad labels, not identities or GPS traces.

## 3. Connect Source to Code

Inspect [configuration](../config/config.yaml),
[ingestion](../src/data/ingestion.py), and
[the pipeline entry point](../scripts/run_pipeline.py), in that order. Record
the configured period, source URLs, filenames, and local destinations. Separate
download/load responsibility from validation and feature engineering.

## 4. Run and Open the Notebook

Complete [environment setup](SETUP.md) first. With raw inputs already present,
the recorded persistence command is:

```powershell
.\.venv\Scripts\python.exe -m scripts.run_pipeline --skip-download
```

If a raw file is missing, acquire it using the recorded fallback:

```powershell
.\.venv\Scripts\python.exe -m scripts.run_pipeline
```

Open [the exploration notebook](../notebooks/01_explore_dataset.ipynb) and select
the project `.venv` kernel. The recording preparation also rebuilds this notebook
with `python -m scripts.build_explore_notebook`. That command overwrites the
generated notebook, so use it only when intentionally regenerating it, not after
adding work you need to retain. No rebuild is required merely to open the notebook.

## 5. Execute the Notebook in Order

Run one code cell at a time and interpret its output before continuing. Follow
the recorded sequence, using section names if cell positions have changed:

| Notebook segment | Evidence to inspect |
| --- | --- |
| Setup and source inventory | Project root, interpreter, provider, files, and source period |
| Raw inputs and schema | Trip preview, required fields, types, and zone lookup |
| Cleaning | Unsuitable values, ordered rules, retained/removed counts and reasons |
| Demand aggregation | Unique zone-hour keys, dense dimensions, zero rows, hourly profile, busiest zones |
| Demand features | Calendar inputs, past-only lags/rolling statistics, and history warm-up |
| Fare features | One clean trip per row, distance/duration/passengers/zones/calendar, and fare-distance plot |
| Chronological splits | All six train/validation/test outputs and their time boundaries |
| Artifacts and lineage checks | Saved paths, row-count relationships, unique keys, and assertions |

The notebook calculates stages in memory to explain them. The pipeline writes
the official persisted outputs. Do not claim a saved file was refreshed merely
because its in-memory equivalent was recalculated.

## 6. Reconcile Persisted Evidence

Check the two raw inputs, clean trips, hourly demand, pipeline summary, both
complete feature tables, and all six split files: thirteen expected artifacts
in the recorded route. File existence proves persistence, not correctness.

Run the notebook's final assertions. Confirm cleaning did not add trips, fare
rows match clean rows, demand zone-hour keys are unique, and split times move
forward. Inspect the lag/rolling code separately to establish past-only inputs.

## 7. Present Three Findings

### Finding 1: Data Cleaning and Filtering
- **Evidence / Artifact:** `data/processed/pipeline_summary.json` and `trips_clean.parquet` (configured period: January 2024).
- **Observed Metrics:** Raw input of 2,964,624 records cleaned to 2,727,349 valid trips (92.00% retained; 237,275 dropped).
- **Largest Removal Reason:** `passenger_count_out_of_range` accounted for 167,879 dropped rows (70.8% of all removals, corresponding to records with 0 or >6 passengers), followed by `trip_duration_out_of_range` (36,999 rows) and `fare_out_of_range` (32,321 rows).
- **Limitation:** Filtering relies on heuristic bounds (1 to 6 passengers, $2.50 to $500.00 fare); trips recorded with 0 passengers may represent package deliveries rather than corrupt meter events.

### Finding 2: Demand Grid Densification
- **Evidence / Artifact:** `data/processed/demand_hourly.parquet`.
- **Observed Metrics:** 253 taxi pickup zones evaluated across 744 hours (31 days) yield a dense temporal grid of 188,232 zone-hour cells. Zero-count rows are explicitly inserted to prevent missing-hour sparsity. Busiest zones: Zone 132 (JFK Airport) and Zone 230/237 (Midtown Manhattan).
- **Limitation:** Geographic zone polygons represent broad administrative areas rather than precise street-level coordinates or driver hailing locations.

### Finding 3: Feature Lineage and Temporal Splits
- **Evidence / Artifact:** `data/features/` split files and `pipeline_summary.json`.
- **Observed Metrics:** Fare features preserve all 2,727,349 clean trips (100% preservation). Demand features require a 168-hour (7-day) warm-up buffer for lag feature generation (`lag_168`), resulting in 145,728 valid feature rows ($253 \times 576$ hours). Chronological splits strictly preserve temporal causality: Train (Jan 1–24, 80%), Validation (Jan 25–28, 10%), and Test (Jan 29–31, 10%).
- **Limitation:** The single-month split evaluates mid-winter post-holiday travel patterns and does not establish model stability across summer seasons or severe multi-day blizzard conditions.

## Completion Check

- Source and configuration are identified before interpreting results.
- All notebook sections were executed or a specific failed/skipped step is recorded.
- The three row meanings and the two feature branches remain distinct.
- Pipeline outputs and notebook calculations were reconciled.
- Three findings cite observed evidence.
- Limitations cover one-month Yellow Taxi scope, broad geography, no direct
  demographic attributes, project cleaning assumptions, and no currentness guarantee.