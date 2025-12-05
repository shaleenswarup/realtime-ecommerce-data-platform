# Realtime E-Commerce Data Platform

A production-style, end-to-end data engineering project that simulates an e-commerce data platform:

- Real-time order event ingestion via Kafka  
- Bronze/Silver data lake layers on S3-compatible storage (MinIO)  
- Daily batch processing into a Postgres analytics warehouse  
- Airflow for orchestration  
- Config management, tests, and Docker-based local infra

## Architecture

**Components:**

1. **Ingestion Service (Python)**  
   - Simulates e-commerce order events  
   - Publishes JSON messages to a Kafka topic (`orders`)

2. **Streaming Job (Spark Structured Streaming)**  
   - Consumes the `orders` topic  
   - Writes raw JSON payloads to MinIO as the **Bronze** layer (partitioned by date)

3. **Batch Job (Spark Batch)**  
   - Runs daily (triggered by Airflow)  
   - Reads Bronze data from MinIO  
   - Cleans/transforms into **Silver** Parquet tables  
   - Optionally aggregates and loads into a **Gold** table in Postgres

4. **Airflow**  
   - DAG: `bronze_to_silver_dag`  
   - Schedules the daily batch job  
   - Example schedule: every day at 02:00

5. **Postgres**  
   - Acts as an analytics warehouse  
   - Contains `fact_orders` table populated from Silver data

6. **MinIO**  
   - Local S3-compatible object store  
   - Buckets:  
     - `ecommerce-bronze`  
     - `ecommerce-silver`

## Tech Stack

- Python 3.11  
- Kafka & Zookeeper  
- Spark 3 (Structured Streaming)  
- MinIO (S3 compatible)  
- PostgreSQL  
- Airflow  
- Docker & Docker Compose  
- Pytest

## Getting Started

### 1. Clone and set up

```bash
git clone https://github.com/<your-username>/realtime-ecommerce-data-platform.git
cd realtime-ecommerce-data-platform
cp .env.example .env
```

Edit `.env` with any overrides you want (ports, credentials, etc).

### 2. Start infrastructure

```bash
docker compose up -d
```

This will start:
- Kafka + Zookeeper
- MinIO
- Postgres
- Airflow (webserver + scheduler)
- Spark base image (for jobs)

### 3. Run the ingestion service

```bash
poetry install  # or: pip install -r airflow/requirements.txt plus src requirements
poetry run python -m src.ingestion_service.producer
```

This script will continuously publish simulated order events into Kafka.

### 4. Run the streaming job

In another terminal:

```bash
poetry run python -m src.streaming_jobs.orders_to_bronze
```

This Spark Structured Streaming job will:

- Consume `orders` topic from Kafka  
- Write raw JSON data to MinIO (`ecommerce-bronze` bucket), partitioned by date.

### 5. Airflow UI

Open Airflow:

- URL: http://localhost:8080
- Default user/pass (set in `.env` or Airflow config)

Unpause the DAG `bronze_to_silver_dag` and trigger it.  
This will run the daily batch job that:

- Reads Bronze data from MinIO  
- Writes cleaned Silver Parquet data  
- Optionally loads aggregates into Postgres.

### 6. Running tests

```bash
poetry run pytest
```

Currently includes:

- `test_config.py` – ensures config is loaded correctly.

You can add more tests for:
- Schema validation
- Data quality checks
- Idempotency of batch jobs

## Project Highlights (for your resume / interviews)

- **End-to-end pipeline**: Ingestion → Streaming → Data Lake (Bronze/Silver) → Warehouse → Orchestration  
- **Production patterns**:  
  - Config-driven (12-factor style)  
  - Dockerized infra  
  - Airflow DAGs and scheduling  
  - Testable, modular Python code  
- **Extensible**:  
  - Add more Kafka topics (clickstream, inventory)  
  - Add more DAGs (backfill, anomaly detection)  
  - Plug in dbt or Great Expectations for advanced modeling/quality checks

## Folder Structure

See the main README section or run:

```bash
tree -L 3
```

to explore the repo.

## License

MIT (or your preferred license).
