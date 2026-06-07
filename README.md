# Real-Time Data Streaming Pipeline

## Overview

This repository is my end-to-end data engineering project built around a real-time streaming pipeline. It collects random user data from an external API, streams it through Kafka, processes it with Spark, and stores results in Cassandra. Apache Airflow orchestrates the workflow, and the entire stack runs in Docker containers.

## Architecture

The project includes the following components:

- **Apache Airflow**: Orchestrates the pipeline and runs the data ingestion task.
- **Kafka + Zookeeper**: Handles streaming of generated user records.
- **Schema Registry + Control Center**: Provides Kafka schema and monitoring support.
- **Spark**: Intended for streaming consumption and processing.
- **Cassandra**: Target storage for processed user records.
- **PostgreSQL**: Airflow metadata database.

## What I Built

- A DAG (`dags/kafka_stream.py`) that fetches data from `https://randomuser.me/api/`
- A Kafka producer that writes cleaned JSON messages to topic `users_created`
- A Spark consumer implementation in `spark_stream.py` designed to read Kafka and write to Cassandra
- A Docker Compose setup that brings up Airflow, Kafka, Spark, Cassandra, and supporting services

## Folder Structure

- `docker-compose.yml` — service definitions for Airflow, Kafka, Spark, Cassandra, Postgres, and observability
- `dags/kafka_stream.py` — Airflow DAG that streams API data into Kafka
- `spark_stream.py` — Spark streaming consumer for Kafka and Cassandra
- `script/entrypoint.sh` — Airflow container startup script
- `requirements.txt` — Python dependencies for Airflow and Kafka

## How to Run Locally

1. Open the project directory:
    ```bash
    cd c:\Users\Shiva\e2e-data-engineering
    ```

2. Start the stack:
    ```bash
    docker compose up -d
    ```

3. Confirm services are running:
    ```bash
    docker compose ps
    ```

4. Open Airflow:
    - `http://localhost:8080`

## How to Validate Data Flow

### Check Airflow

- View DAGs in Airflow UI
- Trigger `user_automation` manually if needed
- Inspect task logs in Airflow UI or via container logs

### Check Kafka

```bash
docker compose exec -T broker bash -lc "kafka-topics --bootstrap-server broker:29092 --list"
```

If the topic exists, consume a few messages:

```bash
docker compose exec -T broker bash -lc "kafka-console-consumer --bootstrap-server broker:29092 --topic users_created --from-beginning --timeout-ms 10000 --max-messages 5"
```

### Check Cassandra

```bash
docker compose exec -T cassandra_db bash -lc "cqlsh -u cassandra -p cassandra -e \"SELECT * FROM spark_streams.created_users LIMIT 5;\""
```

## Notes

- The Airflow DAG writes messages to Kafka but the Spark consumer is expected to be started separately for Cassandra ingestion.
- `dags/kafka_stream.py` formats API responses and sends them to Kafka.
- If you need to debug, check the Airflow task logs and Kafka topic list first.

## Requirements

- Docker
- Docker Compose
- Python (for local development only if editing DAGs or scripts)

## Improvements I Can Make

- Add full Spark execution in Docker Compose
- Add validation and retry logic in the producer DAG
- Add a complete end-to-end test for Kafka-to-Cassandra ingestion

---

This is my working end-to-end streaming project, built to demonstrate real-time ingestion, orchestration, and storage with Docker and modern data engineering tools.