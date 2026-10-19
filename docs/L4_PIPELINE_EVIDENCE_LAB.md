# L4: Build Features and Demand Data

- **Duration:** 35 to 45 minutes
- **Cloud required:** No
- **Outcome:** A verified feature file and pipeline summary with an evidence-based finding.

## Expected Result

Before running the pipeline, record what you expect it to do:

```text
The pipeline will reuse the configured January 2024 source files, clean the
trips, create hourly demand, generate demand and fare features, split both
tables by time, and write a summary.
```

## Task 1: Verify Inputs and Configuration

1. Confirm the repository root and project interpreter.
2. Confirm both January 2024 files exist under `data/raw`.
3. Inspect the configured period, directories, cleaning bounds, split fractions,
   lags, and rolling windows in `config/config.yaml`.

If raw inputs are missing on a fresh checkout, acquire and build them once with
the following command (requires internet), then use Task 2 for the cached rerun:

```powershell
.\.venv\Scripts\python.exe -m scripts.run_pipeline --year 2024 --month 1
```

## Task 2: Run the Pipeline

From the repository root:

```powershell
.\.venv\Scripts\python.exe -m scripts.run_pipeline --skip-download
```

Record whether the command completed and identify the last successful stage.

## Task 3: Inspect Persisted Evidence

Inspect:

- `data/processed/pipeline_summary.json`
- `data/processed/trips_clean.parquet`
- `data/processed/demand_hourly.parquet`
- `data/features/demand_features.parquet`
- `data/features/fare_features.parquet`
- The six demand and fare train, validation, and test files

Record the cleaning counts, demand dimensions, feature shapes, and chronological
date ranges shown by the files.

## Task 4: Run Consistency Checks

Confirm that:

- Clean rows match the pipeline summary.
- Fare-feature rows match clean-trip rows.
- Summed hourly pickup counts match clean-trip rows.
- Demand zone-hour keys are unique.
- Demand and fare feature rows match the pipeline summary.

## Task 5: Explain the Result

### Evidence or output paths

```text
- Pipeline summary: data/processed/pipeline_summary.json
- Cleaned trips: data/processed/trips_clean.parquet (2,727,349 rows)
- Hourly demand: data/processed/demand_hourly.parquet (188,232 rows)
- Demand features: data/features/demand_features.parquet (145,728 rows)
- Fare features: data/features/fare_features.parquet (2,727,349 rows)
- Fare chronological splits: fare_train.parquet (2,181,879 rows), fare_val.parquet (272,735 rows), fare_test.parquet (272,735 rows)
- Demand chronological splits: demand_train.parquet (116,582 rows), demand_val.parquet (14,573 rows), demand_test.parquet (14,573 rows)
```

### Main finding

```text
The automated data pipeline successfully ingested 2,964,624 raw Yellow Taxi trips for January 2024 and applied ordered cleaning filters, retaining 2,727,349 clean trips (92.0% retention rate) and dropping 237,275 corrupt/out-of-range rows. The dominant removal driver was invalid passenger counts (167,879 rows with 0 or >6 passengers), followed by out-of-range trip durations (36,999 rows) and fare bounds (32,321 rows). The hourly demand grid was densified across all 253 zones and 744 hours (188,232 total cells); after accounting for the 168-hour history warm-up required for lag features, 145,728 demand feature rows were generated. Chronological 80/10/10 splitting strictly preserved temporal causality across train (Jan 1-24), validation (Jan 25-28), and test (Jan 29-31) partitions.
```

### Decision or next action

```text
Proceed to Step 4 model training and Optuna hyperparameter tuning using the persisted feature tables, utilizing validation partitions for candidate selection while strictly reserving the test partition for unbiased final evaluation.
```

### Limitation

```text
The analysis and feature lineage reflect a single winter month (January 2024) of NYC Yellow Medallion taxis only. It excludes Green Boro taxis and high-volume for-hire vehicles (Uber/Lyft), and does not account for summer seasonality or severe weather disruptions occurring outside January 2024.
```

## Completion Check

- The expected result was recorded before execution.
- The pipeline command and final result are visible.
- At least one generated feature file and the pipeline summary were inspected.
- The consistency checks passed or any failure was explained.
- The finding cites observed evidence rather than inference alone.
- The next action follows from the evidence.
- The limitation stays within what a local January 2024 run can establish.