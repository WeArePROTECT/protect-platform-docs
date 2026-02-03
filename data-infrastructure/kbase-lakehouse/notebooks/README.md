# Canonical Ingestion Notebooks

Canonical ingestion notebooks are stored in **JupyterHub** and serve as the execution environment for dataset ingestion.

## Pattern

Each dataset has exactly one canonical ingestion notebook. Notebooks follow a standard structure:

1. **Initialize runtime clients** (Spark session, MinIO client)
2. **Define the config path** (points to `protect-<dataset>-config.json` in MinIO)
3. **Validate & summarize config** (dataset name, table count, paths)
4. **Run ingestion** (materializes Delta tables, returns ingestion report)

## Notebook Naming Convention

Notebooks follow the pattern:
```
protect_<dataset>_ingest.ipynb
```

**Examples:**
- `protect_genomedepot_ingest.ipynb`
- `protect_isolate_ingest.ipynb`
- `protect_phenotype_ingest.ipynb`

## Config Path Pattern

Within each notebook, the config path follows this pattern:
```
s3a://cdm-lake/tenant-general-warehouse/protect/datasets/<dataset>-source/config-json/protect-<dataset>-config.json
```

The only dataset-specific change between notebooks is updating this `cfg_path` variable.

---

*Note: Notebooks are not stored in this repository. They are maintained in JupyterHub and referenced here for documentation purposes.*
