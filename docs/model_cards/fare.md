# Model Card — taxi-fare-predictor

_Generated: 2026-10-19 13:33 UTC_

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
| rmse | 3.4486 |
| mae | 0.7979 |
| mape | 8.7346 |
| r2 | 0.9556 |

## Evaluation — subgroup fairness

### By borough

Disparity ratio (worst/best RMSE): **7.47** (threshold 1.5) — ⚠️ **flagged**. Worst: `Unknown`, best: `Manhattan`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| Unknown | 815 | 17.024 | 3.255 | 13.43 |
| Bronx | 580 | 10.269 | 6.039 | 473.08 |
| Brooklyn | 1946 | 8.915 | 4.757 | 17.76 |
| Queens | 23747 | 7.958 | 2.546 | 15.42 |
| Manhattan | 245640 | 2.278 | 0.576 | 6.90 |

### By time_of_day

Disparity ratio (worst/best RMSE): **1.86** (threshold 1.5) — ⚠️ **flagged**. Worst: `overnight`, best: `evening`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| overnight | 8572 | 5.639 | 1.337 | 6.39 |
| afternoon | 101213 | 3.645 | 0.816 | 9.20 |
| morning | 68332 | 3.332 | 0.877 | 8.47 |
| evening | 94618 | 3.028 | 0.672 | 8.64 |

### By day_type

Disparity ratio (worst/best RMSE): **1.31** (threshold 1.5) — ok. Worst: `weekend`, best: `weekday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| weekend | 3150 | 4.503 | 1.115 | 4.89 |
| weekday | 269585 | 3.434 | 0.794 | 8.78 |

### By holiday

Disparity ratio (worst/best RMSE): **1.00** (threshold 1.5) — ok. Worst: `non-holiday`, best: `non-holiday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| non-holiday | 272735 | 3.449 | 0.798 | 8.73 |

### By airport

Disparity ratio (worst/best RMSE): **2.56** (threshold 1.5) — ⚠️ **flagged**. Worst: `airport`, best: `non-airport`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| airport | 20957 | 7.397 | 2.159 | 14.99 |
| non-airport | 251778 | 2.886 | 0.685 | 8.21 |

## Top features (SHAP)

| Feature | Mean |SHAP| |
|---------|-----------|
| trip_distance | 8.1162 |
| trip_duration_min | 3.8163 |
| DOLocationID | 0.4761 |
| PULocationID | 0.1624 |
| passenger_count | 0.1073 |
| hour_cos | 0.0474 |
| hour | 0.0388 |
| hour_sin | 0.0298 |
| day_of_week | 0.0145 |
| dow_sin | 0.0143 |

## Limitations

- Trained on a single month of NYC Yellow Taxi data; may not generalise to other periods, boroughs served sparsely, or green/for-hire vehicles.
- Predictions degrade under distribution shift (see drift monitoring).

## Ethical considerations

- No demographic or protected attributes are used or available; fairness is assessed as geographic/temporal service equity, not demographic parity.
- Large borough-level disparities could translate into unequal service quality across neighbourhoods and should be monitored.
