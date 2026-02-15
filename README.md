# ETL Toll Data Pipeline

## Overview

An Apache Airflow ETL pipeline that collects road traffic data from multiple toll plazas using different file formats (CSV, TSV, fixed-width), consolidates them into a unified dataset, and applies transformations for analysis.

## Architecture

```
Source (S3)          Extract                    Transform         Load
┌──────────┐    ┌──────────────────┐       ┌──────────────┐   ┌─────────┐
│tolldata  │───>│ CSV  ──> csv_data│──┐    │  Uppercase   │   │Staging  │
│  .tgz    │    │ TSV  ──> tsv_data│──┼───>│  Vehicle     │──>│  Area   │
│          │    │ FW   ──> fw_data │──┘    │  Type        │   │         │
└──────────┘    └──────────────────┘       └──────────────┘   └─────────┘
```

## Pipeline Tasks

| Task | Description |
|------|-------------|
| `download_dataset` | Downloads the toll data archive from IBM Cloud Storage |
| `untar_dataset` | Extracts the compressed archive |
| `extract_data_from_csv` | Extracts vehicle data (Rowid, Timestamp, Vehicle number, Vehicle type) |
| `extract_data_from_tsv` | Extracts toll plaza data (Number of axles, Tollplaza id, Tollplaza code) |
| `extract_data_from_fixed_width` | Extracts payment data (Payment code, Vehicle Code) |
| `consolidate_data` | Merges all extracted data into a single CSV file |
| `transform_data` | Converts Vehicle type field to uppercase |

## Task Dependencies

```
download → untar → ┌─ extract_csv ──────┐
                    ├─ extract_tsv ──────┼─→ consolidate → transform
                    └─ extract_fixed_w ──┘
```

The three extraction tasks run in parallel since they are independent of each other.

## Tech Stack

- **Apache Airflow 2.7** — Workflow orchestration
- **PostgreSQL 13** — Airflow metadata database
- **Docker & Docker Compose** — Containerization
- **Python 3.10** — ETL logic

## Quick Start

### Prerequisites

- Docker & Docker Compose installed on your machine

### Installation

1. Clone the repository:
```bash
git clone https://github.com/YOUR_USERNAME/etl-toll-pipeline.git
cd etl-toll-pipeline
```

2. Create required directories:
```bash
mkdir -p logs plugins
```

3. Initialize Airflow:
```bash
docker compose up airflow-init
```

4. Start the services:
```bash
docker compose up -d
```

5. Access the Airflow Web UI at [http://localhost:8080](http://localhost:8080)
   - Username: `admin`
   - Password: `admin`

6. Unpause the `ETL_toll_data` DAG and trigger it.

### Shutdown

```bash
docker compose down
```

To also remove the database volume:
```bash
docker compose down -v
```

## Project Structure

```
etl-toll-pipeline/
├── dags/
│   └── ETL_toll_data.py     # Airflow DAG definition
├── logs/                     # Airflow logs (gitignored)
├── plugins/                  # Custom Airflow plugins
├── docker-compose.yml        # Docker services configuration
├── .gitignore
└── README.md
```

## Data Sources

The pipeline processes three file formats from toll plaza operators:

| File | Format | Fields Extracted |
|------|--------|-----------------|
| `vehicle-data.csv` | CSV | Rowid, Timestamp, Vehicle number, Vehicle type |
| `tollplaza-data.tsv` | TSV | Number of axles, Tollplaza id, Tollplaza code |
| `payment-data.txt` | Fixed-width | Payment code, Vehicle Code |

## Author

**Christian TEWA** — Data Engineering Student at EFREI Paris
