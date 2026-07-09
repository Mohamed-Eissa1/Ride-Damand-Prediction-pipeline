# Real-Time Ride Demand Prediction Pipeline

A streaming data engineering and machine learning system that predicts
short-term ride demand per geographic zone — architected after
production ride-hailing platforms like Uber and Careem, built as the
DEPI AI & Data Engineering track graduation project.

> Full technical documentation (architecture, database design, API
> reference, security, testing strategy, etc.) lives in
> `docs/Ride_Demand_Prediction_Pipeline_Technical_Documentation.docx` —
> that document is the single source of truth for this project. This
> README is an operational quick-start; if anything here ever
> conflicts with the technical documentation, the documentation wins.

## Architecture at a glance

```
Ride Request Producer -> Kafka -> Spark Structured Streaming -> ML Scoring -> PostgreSQL -> FastAPI -> Streamlit Dashboard
```

| Layer | Technology | Folder |
|---|---|---|
| Ingestion | Apache Kafka | `producer/` |
| Stream processing | PySpark Structured Streaming | `streaming/` |
| Machine learning | scikit-learn | `ml/` |
| Persistence | PostgreSQL | `db/` |
| API | FastAPI | `backend/` |
| Dashboard | Streamlit | `dashboard/` |

## Prerequisites

- Docker and Docker Compose
- ~4 GB free RAM for the full stack (Kafka + Spark + Postgres + services)

## Quick start

```bash
git clone https://github.com/<org>/ride-demand-pipeline.git
cd ride-demand-pipeline
cp .env.example .env
# edit .env and set real values for POSTGRES_PASSWORD and API_KEY

docker compose up --build
```

Once every container is healthy:

- API docs (Swagger UI): http://localhost:8000/docs
- API health check: http://localhost:8000/v1/health
- Dashboard: http://localhost:8501

The Postgres schema and demo zone data are created automatically on
first startup from `db/migrations/`.

## Training the model

The ML scoring service requires a trained model artifact before it can
produce predictions. Once the pipeline has been running long enough to
accumulate `zone_features` rows:

```bash
docker compose run --rm ml-scoring python -m ml.train
```

This writes a versioned artifact to `ml/artifacts/`, which the scoring
service picks up automatically.

## Running tests locally

```bash
pip install -r backend/requirements.txt -r ml/requirements.txt -r producer/requirements.txt
pip install pytest

# Fast, no external services required:
pytest tests/unit -v

# Requires the docker-compose stack to already be running:
pytest tests/integration -v
```

## Project structure

```
ride-demand-pipeline/
├── docker-compose.yml
├── .env.example
├── .github/workflows/ci.yml
├── common/              # shared config + logging, used by every service
├── producer/            # Kafka producer (simulator) + debug consumer
├── streaming/           # PySpark Structured Streaming job + feature engineering
├── ml/                  # model training, scoring service, model wrapper
├── backend/             # FastAPI application
├── dashboard/           # Streamlit operational dashboard
├── db/migrations/       # SQL schema, auto-applied on first Postgres startup
├── tests/unit/          # fast tests, no external services needed
├── tests/integration/   # tests run against the live docker-compose stack
└── docs/                # full technical documentation
```

## Team & ownership

| Member | Owns |
|---|---|
| Mohamed Eissa | Kafka, DevOps, CI/CD |
| Mohamed Ahmed Abd El Azim | Spark streaming, feature engineering |
| Mohamed Rezk Rouzik Attia | Machine learning |
| Mohamed Emad Hamdy Mohamed | Backend API, dashboard |

## Further reading

See `docs/Ride_Demand_Prediction_Pipeline_Technical_Documentation.docx`
for business context, full architecture diagrams, database ER diagram,
API reference with request/response examples, security model, and the
testing/deployment strategy.
