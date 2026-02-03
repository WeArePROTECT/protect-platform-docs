# Ingestion Configuration Files

Ingestion configuration files define how raw data files are transformed into managed Delta tables in the SQL warehouse.

## Naming Convention

Config files follow this naming pattern:
```
protect-<dataset>-config.json
```

**Examples:**
- `protect-genomedepot-config.json`
- `protect-isolate-config.json`
- `protect-phenotype-config.json`

## Storage Location

Config files are stored in MinIO under the dataset's source folder:

```
tenant-general-warehouse/protect/datasets/<dataset>-source/config-json/protect-<dataset>-config.json
```

## Config Structure

Each config file specifies:
- Tenant name (`protect`)
- Dataset identifier
- Bronze base path (general warehouse)
- Silver base path (SQL warehouse)
- A list of logical tables, each with:
  - Source file path
  - Explicit SQL schema (`schema_sql`)
  - Write mode (typically `overwrite`)

The config is the **single source of truth** for an ingestion run.

## Security Note

> **Do not commit secrets:** Config files may contain paths and identifiers but should not contain authentication credentials or private endpoints. Credentials are managed separately through the JupyterHub environment.
