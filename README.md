# Netflix dbt Project

An end-to-end data pipeline that ingests raw Netflix data into AWS S3, stages it in Snowflake via Snowpipe, and transforms it into analytics-ready models using dbt, following a Medallion (Raw → Staging → Dimension/Fact) architecture.

## Architecture

Source Data
│
▼
AWS S3 (raw landing zone)
│
▼
Snowpipe (auto-ingest)
│
▼
Snowflake (RAW schema)
│
▼
dbt Transformations
│
├──► STAGING (cleaned, typed, renamed)
│
└──► MARTS (DIM / FACT models for analytics)


## Tech Stack

- **Storage:** AWS S3 — landing zone for raw source files
- **Ingestion:** Snowpipe — continuous, automated loading from S3 into Snowflake
- **Warehouse:** Snowflake — RAW, STAGING, and MARTS schemas
- **Transformation:** dbt (data build tool) — SQL-based modeling, testing, and documentation
- **Architecture pattern:** Medallion (Raw → Staging → Dimension/Fact)

## Data Flow

1. **Raw ingestion (S3 → Snowflake)**
   Source files are uploaded to an S3 bucket. A Snowpipe configured with an S3 event notification automatically detects new files and loads them into raw tables in Snowflake, with minimal to no transformation at this stage.

2. **Staging layer (`models/staging`)**
   Raw tables are cleaned, renamed, and cast to proper types. This layer standardizes column names and formats but does not yet apply business logic — it's a 1:1 (or close to it) reflection of the source with light cleanup.

3. **Marts layer (`models/marts`)**
   Staging models are joined, aggregated, and modeled into dimension and fact tables suited for analytics and reporting — for example, dimension tables for titles, genres, or dates, and fact tables capturing viewing activity or ratings.

## Project Structure

netflix_dbt_project/
├── models/
│ ├── staging/
│ │ ├── sources.yml # source table definitions (raw Snowflake tables)
│ │ ├── staging.yml # staging model documentation & tests
│ │ └── stg.sql # staging models
│ └── marts/
│ ├── marts.yml # marts documentation & tests
│ ├── dim.sql # dimension models
│ └── fct*.sql # fact models
├── snapshots/ # (if applicable) slowly changing dimension snapshots
├── macros/ # reusable dbt macros
├── seeds/ # static reference/lookup data
├── tests/ # custom data tests
├── dbt_project.yml
└── README.md

## Notes

- Raw data ingestion (S3 → Snowpipe → Snowflake) happens outside of dbt; dbt picks up from the RAW schema onward.
- Environment variables and credentials (Snowflake keys, AWS credentials) are kept out of version control — see `.env.example` for the expected variables if applicable.

## Roadmap / Possible Extensions

- Orchestration (e.g. Airflow) to schedule and monitor dbt runs
- Incremental models for large fact tables
- Additional data quality tests and source freshness checks
