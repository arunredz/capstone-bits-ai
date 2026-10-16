# Model Card — taxi-demand-forecaster

_Generated: 2026-10-16 15:11 UTC_

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
| rmse | 9.0454 |
| mae | 2.5852 |
| mape | 40.6921 |
| r2 | 0.9684 |

## Evaluation — subgroup fairness

### By borough

Disparity ratio (worst/best RMSE): **267.80** (threshold 1.5) — ⚠️ **flagged**. Worst: `Manhattan`, best: `Staten Island`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| Manhattan | 3862 | 16.381 | 7.938 | 30.50 |
| Queens | 3971 | 6.193 | 1.161 | 52.64 |
| Unknown | 116 | 3.336 | 2.100 | 48.73 |
| Brooklyn | 3457 | 0.740 | 0.437 | 54.17 |
| Bronx | 2476 | 0.443 | 0.254 | 64.09 |
| EWR | 58 | 0.268 | 0.088 | 98.26 |
| Staten Island | 633 | 0.061 | 0.027 | 94.19 |

### By time_of_day

Disparity ratio (worst/best RMSE): **2.91** (threshold 1.5) — ⚠️ **flagged**. Worst: `evening`, best: `overnight`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| evening | 4554 | 12.502 | 3.451 | 38.22 |
| afternoon | 3947 | 8.642 | 3.004 | 36.45 |
| morning | 3036 | 6.542 | 2.430 | 39.30 |
| overnight | 3036 | 4.291 | 0.898 | 58.51 |

### By day_type

Disparity ratio (worst/best RMSE): **1.00** (threshold 1.5) — ok. Worst: `weekday`, best: `weekday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| weekday | 14573 | 9.045 | 2.585 | 40.69 |

### By holiday

Disparity ratio (worst/best RMSE): **1.00** (threshold 1.5) — ok. Worst: `non-holiday`, best: `non-holiday`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| non-holiday | 14573 | 9.045 | 2.585 | 40.69 |

### By airport

Disparity ratio (worst/best RMSE): **4.21** (threshold 1.5) — ⚠️ **flagged**. Worst: `airport`, best: `non-airport`.

| Group | n | RMSE | MAE | MAPE |
|-------|---|------|-----|------|
| airport | 116 | 35.741 | 25.630 | 37.04 |
| non-airport | 14457 | 8.499 | 2.400 | 40.77 |

## Top features (SHAP)

| Feature | Mean |SHAP| |
|---------|-----------|
| lag_1 | 16.2821 |
| lag_168 | 6.6261 |
| lag_24 | 2.8352 |
| hour_cos | 0.7233 |
| rolling_std_3 | 0.3249 |
| rolling_mean_168 | 0.3043 |
| hour | 0.3036 |
| lag_2 | 0.2930 |
| lag_3 | 0.2925 |
| rolling_mean_24 | 0.2724 |

## Limitations

- Trained on a single month of NYC Yellow Taxi data; may not generalise to other periods, boroughs served sparsely, or green/for-hire vehicles.
- Predictions degrade under distribution shift (see drift monitoring).

## Ethical considerations

- No demographic or protected attributes are used or available; fairness is assessed as geographic/temporal service equity, not demographic parity.
- Large borough-level disparities could translate into unequal service quality across neighbourhoods and should be monitored.
