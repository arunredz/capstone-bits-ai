# L7: Call and Package the Prediction API

- **Duration:** 45 to 60 minutes
- **Cloud required:** No
- **Docker required:** Optional; a local Python fallback is provided
- **Outcome:** Valid and invalid API evidence plus a verified container or local runtime.

## Prerequisites

From the repository root, confirm that the project environment and serving
artifacts exist:

```powershell
.\.venv\Scripts\python.exe --version
Test-Path mlflow.db
Test-Path mlruns
Test-Path data\processed\demand_hourly.parquet
```

Path existence alone does not prove that models are registered. In the MLflow
UI, confirm both `taxi-demand-forecaster` and `taxi-fare-predictor` have registered
versions with accessible artifacts. L5's `--no-register` run does not satisfy
this requirement, and L6 may have registered only fare.

If inputs or registered models are missing, complete the pipeline and
model-training steps in [SETUP.md](SETUP.md). For a bounded teaching run that
registers both models, use:

```powershell
.\.venv\Scripts\python.exe -m scripts.train_models --n-trials 1
```

If you plan to use Route A, create and verify the portable registry required by
the Docker image:

```powershell
.\.venv\Scripts\python.exe scripts\make_mlflow_portable.py `
	--src mlflow.db `
	--dest mlflow.container.db `
	--new-base "file:///app/mlruns"
Test-Path mlflow.container.db   # should return True
```

Repeat this command after retraining so the container uses the latest registry.
Route B does not require `mlflow.container.db`.

## Task 1: Predict the Runtime Result

Record the expected health status, one valid prediction result, and the status
code expected from an invalid request before starting the service.

## Task 2: Start the API

Choose one route.

### Route A: Docker Compose

Confirm Docker is available, then build and start the API and MLflow UI:

```powershell
docker version
docker compose -f docker/docker-compose.yml up --build
```

If `docker version` reports that the command is unavailable, cannot connect to
the Docker daemon, or does not include Compose, skip the second command and use
Route B. Docker is optional for L7; record that you used the local fallback.

Keep this terminal open. The API is available at `http://localhost:8000` and
the MLflow UI at `http://localhost:5000`.

### Route B: Local Python

Use this route when Docker is unavailable:

```powershell
.\.venv\Scripts\python.exe -m uvicorn src.serving.api:app --host 127.0.0.1 --port 8000
```

Keep this terminal open. This route starts the API at `http://localhost:8000`;
it does not start the MLflow UI, which is not required to complete L7.

## Task 3: Verify Health and Documentation

In another PowerShell terminal:

```powershell
Invoke-RestMethod http://localhost:8000/health
Start-Process http://localhost:8000/docs
```

Record the health response and identify the fare and demand prediction
endpoints in Swagger UI.

## Task 4: Exercise Valid and Invalid Requests

Use the request examples in `docs/API_EXAMPLES.md` to send:

1. One valid fare request.
2. One valid demand request.
3. One invalid request with a missing or out-of-range field.

Record the prediction fields from the valid responses. For the invalid request,
record the HTTP status and the validation message without changing server code.

## Task 5: Inspect the Packaging Boundary

Open `docker/Dockerfile` and identify:

- The builder and runtime stages.
- The copied source, model registry, model artifacts, and demand history.
- Port `8000`.
- The Uvicorn startup command.

Then open `docker/docker-compose.yml` and identify:

- The `api` and `mlflow-ui` services and their port mappings.
- The development `mlflow.db`, model-artifact, and data mounts that override
	the corresponding files baked into the image.
- The API `/health` container check.

If you used Route B, inspect these files as packaging evidence but state that
you did not execute the image build, Compose wiring, or container health check.

Explain why Docker packages the API and its dependencies but does not train the
models or prove that the service is deployed to AWS.

## Task 6: Stop the Runtime

For Docker Compose, press `Ctrl+C`, then run:

```powershell
docker compose -f docker/docker-compose.yml down
docker compose -f docker/docker-compose.yml ps   # should show no running services
```

For local Python, press `Ctrl+C` in the Uvicorn terminal.

## Result Record

### Runtime route and evidence

```text
Route B: Local Python Uvicorn.
- Service launched via: uvicorn src.serving.api:app --host 0.0.0.0 --port 8000
- Health verification: GET http://127.0.0.1:8000/health returned HTTP 200 with application status "healthy" and loaded registered model metadata.
- Interactive API documentation verified via Swagger UI on http://127.0.0.1:8000/docs.
```

### Valid response finding

```text
- Real-Time Fare Inference:
  POST /predict/fare with payload {"pickup_datetime": "2024-01-15T18:30:00", "pu_location_id": 132, "do_location_id": 230, "trip_distance": 17.2, "passenger_count": 1}
  returned HTTP 200 with predicted_fare: $70.28, duration_estimated: true. The prediction aligns with the official TLC JFK-Manhattan flat rate ($70.00 base).
- Batch Demand Forecast:
  Executed python -m scripts.batch_forecast generating a 24-hour recursive multi-step demand forecast across all 253 NYC zones, successfully persisting 6,072 prediction rows to predictions.db.
```

### Invalid response finding

```text
POST /predict/fare with an invalid payload (e.g., negative trip_distance or malformed datetime string) returned HTTP 422 Unprocessable Entity with structured Pydantic validation error details. The request was rejected cleanly at the schema validation boundary without raising unhandled server exceptions or terminating the background process.
```

### Packaging explanation and limitation

```text
The project defines containerized packaging via docker/Dockerfile (multi-stage Python 3.11 build copying model artifacts, portable registry mlflow.container.db, and demand history) and docker/docker-compose.yml (mounting runtime storage and mapping ports 8000 and 5000). While containerization guarantees portable and reproducible local execution, running the service locally does not prove AWS cloud infrastructure, ALB listener routing, IAM roles, or ECS Fargate auto-recovery.
```

## Completion Check

- The expected result was recorded before starting the service.
- Health and Swagger documentation were inspected.
- Valid fare and demand requests returned interpretable responses.
- An invalid request returned an explained validation error.
- Docker was exercised, or its absence and the local fallback were recorded.
- The result distinguishes local packaging from AWS deployment.