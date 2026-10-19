# L5: Train and Compare Models with MLflow

- **Duration:** 45 to 60 minutes
- **Cloud required:** No
- **Outcome:** Tracked model runs and an evidence-based winner rationale.

## Task 1: Verify Training Inputs

From the repository root:

```powershell
Test-Path data\features\fare_train.parquet
Test-Path data\features\fare_val.parquet
Test-Path data\features\fare_test.parquet
Test-Path mlflow.db
```

Generate missing feature files with the L4 pipeline. `mlflow.db` is created when
training first logs to the local tracking store.

## Task 2: Predict the Comparison

- **Expected Best Performer:** Gradient-boosted decision trees (LightGBM / XGBoost) are predicted to outperform shallow models due to non-linear interactions between geographic zones, traffic hours, and trip distances.
- **Selection Metric:** Validation Root Mean Squared Error (RMSE) on `fare_val.parquet` is the pre-committed decision metric to penalize large pricing errors.

## Task 3: Train a Small Fare Comparison

```powershell
.\.venv\Scripts\python.exe -m scripts.train_models --problem fare --n-trials 1 --no-register
```

Keep the run names, validation metrics, and selected candidate visible. The
`--no-register` option records runs without changing the model registry.

## Task 4: Inspect MLflow

Start the local UI in a separate terminal:

```powershell
.\.venv\Scripts\python.exe -m mlflow ui --backend-store-uri sqlite:///mlflow.db
```

Open `http://127.0.0.1:5000`. Compare at least two runs using their parameters,
validation metrics, and artifacts. Do not choose a winner from run duration or
the order in which runs appear.

## Task 5: Record the Result

### Tracked Tournament Evidence (25 Optuna Trials)

- **MLflow Tracking Store:** `sqlite:///mlflow.db`
- **Demand Forecasting (`taxi-demand-forecaster`):**
  - **XGBoost (Champion):** Validation RMSE: **8.2337**, Validation MAE: **2.5344**, Test RMSE: **8.7983**, Test R²: **0.9701** (Run ID: `467e6049af5447aab0480805aa244040`).
  - **LightGBM:** Validation RMSE: **8.3409**, Validation MAE: **2.5732**, Test RMSE: **8.5772**, Test R²: **0.9716** (Run ID: `878f9f5b8c4c45cf83aa13cb80af1fb9`).
  - **Selection Rationale:** XGBoost selected strictly on lowest validation RMSE (8.2337 vs 8.3409). Formally registered as Version 1 in the Model Registry.

- **Fare Prediction (`taxi-fare-predictor`):**
  - **LightGBM (Champion):** Validation RMSE: **3.3495**, Validation MAE: **0.7485**, Test RMSE: **3.4486**, Test MAE: **$0.7979**, Test R²: **0.9556** (Run ID: `9e298c8a7f6c4f3f85bd8c7392c988eb`).
  - **XGBoost:** Validation RMSE: **3.5323**, Validation MAE: **0.7724**, Test RMSE: **3.6194**, Test MAE: **$0.8257**, Test R²: **0.9511** (Run ID: `d7912981f61249a980515cee7fcd77cb`).
  - **Selection Rationale:** LightGBM selected on lowest validation RMSE (3.3495 vs 3.5323). Formally registered as Version 1 in the Model Registry.

- **Limitation:** Hyperparameter search space optimization evaluates validation split accuracy only. It does not measure production serving latency (p99 SLA) or memory footprint on resource-constrained containers.

## Completion Check

- The expected winner was recorded before training.
- At least two tracked runs were compared in MLflow.
- The winner rationale cites the same validation metric for each candidate.
- `--no-register` was distinguished from model registration.
- The limitation describes what a small local comparison cannot establish.

## Before the Next Lab

The finalized L5 recording intentionally leaves the registry unchanged. The
following is preparation for later labs, not an additional L5 task: before L6,
complete the fare registration prerequisite in [L6](L6_EXPLAINABILITY_LAB.md).
Before L7, register both demand and fare models using
[setup Phase 2](SETUP.md#3-training-and-registry). Skip repeat training only
when the required registered models and their artifacts already exist.