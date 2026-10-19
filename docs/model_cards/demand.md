# Model Card — taxi-demand-forecaster

_Generated: 2026-10-19 13:33 UTC_

## Model details

- **Problem:** demand
- **Algorithm:** XGBRegressor
- **Target:** `pickup_count`
- **Registry version:** n/a

## Intended use

Operational forecasting of pickup_count to support fleet positioning and rider fare estimates. Not intended for individual pricing decisions or any use with legal/financial consequences for individuals.

## Evaluation — overall (test split)

| Metric | Value |
|--------|-------|
| rmse | 8.7983 |
| mae | 2.5492 |
| mape | 39.6817 |
| r2 | 0.9701 |

## Evaluation — subgroup fairness

### By borough

Disparity ratio (worst/best RMSE): **191.75** (threshold 1.5) — ⚠️ **flagged**. Worst: `Manhattan`, best: `Staten Island`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| Manhattan | 3862 | 15.901 | 7.740 | 29.86 |
| Queens | 3971 | 6.105 | 1.187 | 51.17 |
| Unknown | 116 | 3.249 | 2.032 | 46.82 |
| Brooklyn | 3457 | 0.739 | 0.454 | 52.59 |
| Bronx | 2476 | 0.445 | 0.280 | 62.62 |
| EWR | 58 | 0.272 | 0.116 | 97.10 |
| Staten Island | 633 | 0.083 | 0.063 | 92.64 |

### By time_of_day

Disparity ratio (worst/best RMSE): **2.76** (threshold 1.5) — ⚠️ **flagged**. Worst: `evening`, best: `overnight`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| evening | 4554 | 12.117 | 3.401 | 36.89 |
| afternoon | 3947 | 8.480 | 2.974 | 35.67 |
| morning | 3036 | 6.215 | 2.341 | 38.74 |
| overnight | 3036 | 4.386 | 0.927 | 56.66 |

### By day_type

Disparity ratio (worst/best RMSE): **1.00** (threshold 1.5) — ok. Worst: `weekday`, best: `weekday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| weekday | 14573 | 8.798 | 2.549 | 39.68 |

### By holiday

Disparity ratio (worst/best RMSE): **1.00** (threshold 1.5) — ok. Worst: `non-holiday`, best: `non-holiday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| non-holiday | 14573 | 8.798 | 2.549 | 39.68 |

### By airport

Disparity ratio (worst/best RMSE): **4.27** (threshold 1.5) — ⚠️ **flagged**. Worst: `airport`, best: `non-airport`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| airport | 116 | 35.209 | 25.705 | 35.10 |
| non-airport | 14457 | 8.251 | 2.363 | 39.78 |

## Top features (SHAP)

| Feature | Mean |SHAP| |
|---------|-----------|
| lag_1 | 13.4726 |
| lag_168 | 7.9506 |
| lag_24 | 4.0912 |
| rolling_mean_3 | 0.7234 |
| hour_cos | 0.7086 |
| lag_3 | 0.4606 |
| rolling_mean_168 | 0.3306 |
| hour | 0.2814 |
| rolling_std_24 | 0.2494 |
| rolling_std_3 | 0.2164 |

## Limitations

- Trained on a single month of NYC Yellow Taxi data; may not generalise to other periods, boroughs served sparsely, or green/for-hire vehicles.
- Predictions degrade under distribution shift (see drift monitoring).

## Ethical considerations

- No demographic or protected attributes are used or available; fairness is assessed as geographic/temporal service equity, not demographic parity.
- Large borough-level disparities could translate into unequal service quality across neighbourhoods and should be monitored.
