# Datasets

In the KBase Lakehouse context, a **dataset** is a logical collection of related tables that are ingested together into a single Spark SQL namespace (`protect_<dataset>`).

Each dataset:
- Has its own source folder in MinIO (`<dataset>-source/`)
- Has its own ingestion configuration file (`protect-<dataset>-config.json`)
- Has its own canonical ingestion notebook
- Produces tables under the `protect_<dataset>` namespace in the SQL warehouse

## Dataset-Specific Documentation

- [GenomeDepot](./genomedepot/) — GenomeDepot ingestion roadmap and validation checklist
