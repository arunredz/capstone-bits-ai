# Taxi Forecasting

[![CI Quality Gate](https://github.com/arunredz/capstone-bits-ai/actions/workflows/ci.yml/badge.svg)](https://github.com/arunredz/capstone-bits-ai/actions/workflows/ci.yml)

Python application for two NYC taxi prediction tasks:

- **Demand:** pickup count for one pickup zone and hour.
- **Fare:** `fare_amount` for one trip, not the full passenger bill.

The repository includes data processing, XGBoost/LightGBM training, MLflow
tracking, FastAPI serving, recursive batch forecasts, SHAP explanations,
subgroup evaluation, drift detection, a Streamlit dashboard, and optional AWS
infrastructure templates. 


## Quickstart: Clean Reproduction (Golden Path)

To clone and reproduce the entire machine learning pipeline end-to-end in a fresh directory (Ubuntu / WSL 2):

### 1. Clone & Environment Setup

```bash
# Clone repository and enter directory
git clone [https://github.com/arunredz/capstone-bits-ai.git](https://github.com/arunredz/capstone-bits-ai.git)
cd capstone-bits-ai

# Create and activate Python 3.11 virtual environment
python3.11 -m venv .venv
source .venv/bin/activate

# Upgrade pip and install pinned runtime dependencies
pip install --upgrade pip
pip install -r requirements.txt

```

### 2. Run Quality Gates & Unit Tests

```bash
# Verify code formatting and linting
python -m ruff check .

# Run test suite with statement coverage (120/120 tests, >75% coverage threshold)
python -m pytest --cov=src --cov-report=term-missing

```

### 3. Data Engineering Pipeline & Temporal Splitting

```bash
# Ingests NYC TLC Yellow Taxi records (Jan 2024), validates schema, filters corrupt rows,
# densifies the hourly demand grid, and creates 80/10/10 chronological splits
python -m scripts.run_pipeline

```

(Note: Omit `--skip-download` on a fresh machine so it fetches the raw monthly Parquet and zone lookup files into `data/raw/`. If you already have the raw files cached locally, add `--skip-download`).

### 4. Hyperparameter Tuning & MLflow Model Registry

```bash
# Runs Optuna hyperparameter sweeps (25 trials each) across XGBoost and LightGBM
# Logs runs, parameters, metrics, and registers champion models to sqlite:///mlflow.db
python -m scripts.train_models --problem both --n-trials 25

```

### 5. Responsible AI, Subgroup Fairness & Explainability

```bash
# Audits subgroup disparities across Boroughs, Airports, and Time of Day
# Generates global SHAP summary beeswarm plots and publishes Model Cards to docs/model_cards/
python -m scripts.responsible.run_analysis

```

### 6. Model Serving & 24-Hour Batch Forecast

```bash
# Start FastAPI backend server in the background (Port 8000)
uvicorn src.serving.api:app --host 0.0.0.0 --port 8000 &

# Quick smoke test for real-time fare inference (returns HTTP 200 with ~$70 quote)
curl -X POST "[http://127.0.0.1:8000/predict/fare](http://127.0.0.1:8000/predict/fare)" \
     -H "Content-Type: application/json" \
     -d '{
       "pickup_datetime": "2024-01-15T18:30:00",
       "pu_location_id": 132,
       "do_location_id": 230,
       "trip_distance": 17.2,
       "passenger_count": 1
     }'

# Execute 24-hour recursive multi-step demand batch forecast across all 253 zones (writes 6,072 rows to predictions.db)
python -m scripts.batch_forecast

```

### 7. Feature Drift Monitoring

```bash
# Calculates Population Stability Index (PSI) and Kolmogorov-Smirnov statistics against baseline
python -m scripts.monitoring.demo_drift_report --problem fare --mode drift

```

### 8. Launch Operations Dashboard

```bash
# Install dashboard dependencies (Streamlit & Plotly)
pip install -r requirements-dashboard.txt

# Start the Streamlit UI (opens on http://localhost:8501)
streamlit run dashboard/app.py

```

