# Data Infrastructure
This directory documents how PROTECT data are centrally stored, organized, and governed in the current lab server environment, including organizational conventions and rationale for centralization.

## Data Lake / Storage

- **[KBase Lakehouse](./kbase-lakehouse/)** — Ingestion and operations guide for the KBase Lakehouse environment (MinIO object storage, Delta Lake managed storage, Spark SQL querying)

## Integration Pipeline

- **[PROTECT Data Integration Pipeline](./integration-pipeline/)** — Production pipeline unifying all four PROTECT data streams (isolate records, clinical metadata, isolation provenance, multi-omics) into analysis-ready linked tables (v1.0)
