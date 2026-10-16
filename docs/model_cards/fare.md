# Model Card — taxi-fare-predictor

_Generated: 2026-10-16 15:11 UTC_

## Model details

- **Problem:** fare
- **Algorithm:** LGBMRegressor
- **Target:** `fare_amount`
- **Registry version:** n/a

## Intended use

Operational forecasting of fare_amount to support fleet positioning and rider fare estimates. Not intended for individual pricing decisions or any use with legal/financial consequences for individuals.

## Evaluation — overall (test split)

| Metric | Value |
|--------|-------|
| rmse | 3.4378 |
| mae | 0.8011 |
| mape | 8.9865 |
| r2 | 0.9559 |

## Evaluation — subgroup fairness

### By borough

Disparity ratio (worst/best RMSE): **7.42** (threshold 1.5) — ⚠️ **flagged**. Worst: `Unknown`, best: `Manhattan`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| Unknown | 815 | 16.873 | 3.276 | 13.19 |
| Bronx | 580 | 10.259 | 5.945 | 537.76 |
| Brooklyn | 1946 | 8.741 | 4.602 | 17.43 |
| Queens | 23747 | 7.941 | 2.529 | 15.86 |
| Manhattan | 245640 | 2.274 | 0.583 | 6.99 |

### By time_of_day

Disparity ratio (worst/best RMSE): **1.85** (threshold 1.5) — ⚠️ **flagged**. Worst: `overnight`, best: `evening`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| overnight | 8572 | 5.628 | 1.352 | 6.49 |
| afternoon | 101213 | 3.628 | 0.817 | 9.28 |
| morning | 68332 | 3.302 | 0.873 | 8.48 |
| evening | 94618 | 3.040 | 0.682 | 9.26 |

### By day_type

Disparity ratio (worst/best RMSE): **1.31** (threshold 1.5) — ok. Worst: `weekend`, best: `weekday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| weekend | 3150 | 4.472 | 1.114 | 4.79 |
| weekday | 269585 | 3.424 | 0.797 | 9.04 |

### By holiday

Disparity ratio (worst/best RMSE): **1.00** (threshold 1.5) — ok. Worst: `non-holiday`, best: `non-holiday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| non-holiday | 272735 | 3.438 | 0.801 | 8.99 |

### By airport

Disparity ratio (worst/best RMSE): **2.58** (threshold 1.5) — ⚠️ **flagged**. Worst: `airport`, best: `non-airport`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| airport | 20957 | 7.409 | 2.161 | 15.55 |
| non-airport | 251778 | 2.869 | 0.688 | 8.44 |

## Top features (SHAP)

| Feature | Mean |SHAP| |
|---------|-----------|
| trip_distance | 8.1685 |
| trip_duration_min | 3.4610 |
| DOLocationID | 0.5085 |
| PULocationID | 0.2040 |
| passenger_count | 0.1041 |
| hour_cos | 0.0513 |
| hour_sin | 0.0352 |
| hour | 0.0334 |
| dow_sin | 0.0137 |
| day_of_week | 0.0122 |

## Limitations

- Trained on a single month of NYC Yellow Taxi data; may not generalise to other periods, boroughs served sparsely, or green/for-hire vehicles.
- Predictions degrade under distribution shift (see drift monitoring).

## Ethical considerations

- No demographic or protected attributes are used or available; fairness is assessed as geographic/temporal service equity, not demographic parity.
- Large borough-level disparities could translate into unequal service quality across neighbourhoods and should be monitored.
