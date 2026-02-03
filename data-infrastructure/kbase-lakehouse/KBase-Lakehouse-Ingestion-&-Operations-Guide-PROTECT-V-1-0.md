# KBase Lakehouse Ingestion & Operations Guide (PROTECT)

**Audience:** PROTECT / ASMA technical contributors

**Purpose:**
This document captures the *current, validated* operating model for working within the KBase Lakehouse environment, based on hands-on walkthroughs with the KBase team and a successful GenomeDepot pilot ingestion. It is intentionally precise and reflects **what actually works today**, not hypothetical future architecture.

---

## Quick Start (Happy Path)

If you just want to ingest a new dataset end-to-end, follow this sequence:

1. Create a new source folder under:
   ```
   tenant-general-warehouse/protect/datasets/<dataset>-source/
   ```
2. Upload raw data files (e.g. TSVs) into that folder using the MinIO web UI.
3. Create a dataset-specific ingestion config:
   ```
   protect-<dataset>-config.json
   ```
   and place it under:
   ```
   <dataset>-source/config-json/
   ```
4. Copy an existing canonical ingestion notebook and update only the `cfg_path` to point at the new config file.
5. Run the notebook in JupyterHub and confirm ingestion completes successfully.
6. Query the resulting tables under the Spark SQL namespace:
   ```
   protect_<dataset>
   ```

---

## 1. System Overview (Mental Model)

## Vocabulary & Terms

| Term | Meaning |
|------|--------|
| Dataset | A logical collection of related tables ingested together |
| Source folder | The `<dataset>-source/` directory in the general warehouse |
| Bronze | Raw, user-managed files stored in MinIO |
| Silver | Managed Delta tables produced by ingestion |
| Namespace | Spark SQL schema named `protect_<dataset>` |
| Canonical ingestion notebook | The dataset-specific notebook used to run ingestion |



The KBase Lakehouse consists of **three distinct layers** that must not be conflated:

1. **Object Storage (Raw / Bronze Layer)**
   - Backed by **MinIO** (S3-compatible)
   - Used for *raw, user-managed files* (TSV, CSV, JSON, etc.)
   - Referred to in paths as: `tenant-general-warehouse`

2. **Managed Storage (Silver Layer)**
   - Backed by **Delta Lake / Parquet**
   - Fully managed by the ingestion pipeline
   - ACID-compliant, queryable via Spark SQL
   - Referred to in paths as: `tenant-sql-warehouse`

3. **Compute & Query Layer**
   - **Spark** for ingestion and analytics
   - **JupyterHub** as the execution environment

> **Critical rule:** Users manually manage data in the *general warehouse* only. The *SQL warehouse* is write-only for the ingestion pipeline and must never be edited by hand.

---

## 2. Access & Authentication

### 2.1 SSH Tunnel (Required for MinIO UI Access)

Users must establish a SOCKS proxy tunnel into the KBase network.

**Canonical command (example):**
```bash
ssh -f -D 1338 ac.<username>@login1.berkeley.kbase.us "/bin/sleep infinity"
```

- Creates a local SOCKS5 proxy on `localhost:1338`
- Runs in the background (`-f`)
- Persists indefinitely (`sleep infinity`)

Each user substitutes their own username.

---

### 2.2 Browser Proxy Configuration

The SSH tunnel must be paired with a browser proxy (e.g. **SwitchyOmega**).

**Required settings:**
- Proxy type: **SOCKS5**
- Host: `127.0.0.1`
- Port: `1338`

The browser must have the proxy enabled when accessing MinIO URLs.

---

## 3. MinIO Object Storage Layout

### 3.1 Bucket & Top-Level Structure

Users access the MinIO UI and navigate to the `cdm-lake` bucket.

Two key directories exist:

```
cdm-lake/
├── tenant-general-warehouse/
└── tenant-sql-warehouse/
```

Only `tenant-general-warehouse` is user-managed.

---

### 3.2 PROTECT Dataset Convention (General Warehouse)

All PROTECT datasets live under:

```
tenant-general-warehouse/protect/datasets/
```

Each dataset gets its own *source folder*:

```
<dataset>-source/
├── *.tsv
└── config-json/
    └── protect-<dataset>-config.json
```

**Example (GenomeDepot):**
```
genomedepot-source/
├── browser_strain.tsv
├── browser_sample.tsv
├── browser_taxon.tsv
├── browser_genome.tsv
├── browser_gene_SAMPLED.tsv
├── browser_annotation_SAMPLED.tsv
└── config-json/
    └── protect-genomedepot-config.json
```

> Filenames and paths are **case-sensitive** and must match the ingestion config exactly.

---

## 4. Ingestion Configuration Files

### 4.1 One Config File per Dataset (Design Decision)

Each dataset has its own ingestion configuration file.

**Examples:**
- `protect-genomedepot-config.json`
- `protect-isolate-config.json`
- `protect-phenotype-config.json`

**Rationale:**
- Each config defines exactly one `dataset`
- Each config maps to exactly one SQL namespace (`protect_<dataset>`)
- Avoids brittle conventions and accidental cross-dataset overwrites
- Aligns with the ingestion tooling as implemented

---

### 4.2 What the Config Controls

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

---

## 5. Canonical Ingestion Notebooks

### 5.1 One Notebook per Dataset

Each dataset has a **canonical ingestion notebook** stored in JupyterHub.

**Example names:**
- `protect_genomedepot_ingest.ipynb`
- `protect_isolate_ingest.ipynb`

The notebooks are nearly identical; the only dataset-specific input is the config path.

---

### 5.2 Standard Notebook Structure

A canonical ingestion notebook consists of four logical steps:

1. **Initialize runtime clients**
   - Spark session
   - MinIO client

2. **Define the config path**
   - `s3a://cdm-lake/tenant-general-warehouse/.../config-json/...`

3. **Validate & summarize config**
   - Confirms dataset name, table count, paths

4. **Run ingestion**
   - Materializes Delta tables
   - Returns a structured ingestion report

The notebook does **not** upload data.

---

## 6. SQL Warehouse Output

### 6.1 Namespace & Storage Layout

For a dataset named `<dataset>`, ingestion creates:

- **Namespace (schema):** `protect_<dataset>`
- **Storage path:**
  ```
  tenant-sql-warehouse/protect/protect_<dataset>.db/
  ```

Each table is stored as a Delta table:

```
browser_strain/
├── part-*.parquet
└── _delta_log/
```

---

### 6.2 Querying

Tables can be queried using Spark SQL:

```python
spark.sql("SELECT * FROM protect_genomedepot.browser_strain")
```

For small result sets, data may be converted to Pandas. For larger analyses, Spark or pandas-on-Spark should be used.

---

## 7. Ingestion Reports & Validation

Each ingestion run returns a structured **report object** containing:
- Rows read / written / rejected
- Extra columns dropped
- Per-table elapsed time
- Quarantine paths
- Error summaries

This report should be treated as:
- An ingestion audit trail
- A lightweight QC artifact

---

## 8. Schema Alignment & Precision Notes

- `inferSchema` is disabled by default
- Schemas are enforced via `schema_sql`
- Extra columns in TSVs **may be silently dropped** (with warnings)

**Best practice:**
- Validate TSV headers against `schema_sql` before ingestion
- Inspect ingestion reports for `extra_columns_dropped`

---

## 9. Data Upload Process (Current Practice)

### 9.1 Current (v1) Approach

Data files are uploaded **manually** using the MinIO web interface.

Workflow:
1. Establish SSH tunnel + browser proxy
2. Log into MinIO UI
3. Navigate to:
   ```
   tenant-general-warehouse/protect/datasets/<dataset>-source/
   ```
4. Upload files directly
5. Run ingestion notebook

There is no dedicated upload notebook or required CLI step.

---

### 9.2 Future Scaling Considerations (Explicitly Deferred)

For very large datasets (e.g. TB-scale FASTQs, BAMs):
- Manual web uploads may become impractical
- CLI-based upload methods may be evaluated (e.g. MinIO client, sync tools)

This does **not** affect:
- Dataset structure
- Config design
- Ingestion notebooks
- SQL warehouse layout

---

## 10. GenomeDepot as the Reference Dataset

GenomeDepot serves as the **reference implementation** for PROTECT Lakehouse ingestion:
- Clean relational structure
- Explicit sampling strategy
- Stable schemas
- Successful end-to-end ingestion

Future datasets should follow the same pattern unless there is a clear reason to diverge.

---

## Out of Scope (Current Phase)

The following are intentionally **not** addressed in the current Lakehouse operating model:

- Automated or CLI-based bulk data uploads
- Streaming or incremental ingestion
- Row-level updates, merges, or upserts
- Formal APIs for cross-dataset joins
- Workflow orchestration (e.g. Airflow, scheduling)
- Performance tuning or cost optimization

These topics may be revisited in later phases but are explicitly deferred to keep the current system simple, auditable, and correct.


GenomeDepot serves as the **reference implementation** for PROTECT Lakehouse ingestion:
- Clean relational structure
- Explicit sampling strategy
- Stable schemas
- Successful end-to-end ingestion

Future datasets should follow the same pattern unless there is a clear reason to diverge.

---

## 11. Summary

This document reflects the **current, validated operating model** for the PROTECT Lakehouse environment:
- Manual uploads via MinIO UI
- One dataset per config
- One canonical notebook per dataset
- Automated, reproducible ingestion into Delta Lake

This foundation is intentionally simple, explicit, and scalable.

