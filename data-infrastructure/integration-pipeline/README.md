# PROTECT Data Integration Pipeline

**Maintainer:** Spencer Long (Arkin Lab / LBNL)
**Version:** v1.1 (Stage 4 added 2026-05-06)
**Server path:** `/usr2/people/protect/Arkin_Lab/protect_data/protect_data_integration_pipeline/`

This directory documents the PROTECT Data Integration Pipeline — the production system that unifies the PROTECT data streams (isolate records, clinical metadata, isolation provenance, multi-omics, and Conrad sample metadata) into a set of analysis-ready linked tables.

---

## Data Streams

| Stream | Source | Owner | Stages affected |
|---|---|---|---|
| Bacterial isolate records | Arkin Lab (ASMA) | Sun-Young Kim / Spencer Long | 0, 2, 3 |
| Clinical metadata (REDCap) | Conrad Lab | Dahen Ibarra Munoz | 1, 2, 3 |
| Isolation provenance (APL) | Arkin Lab | Sun-Young Kim | 2 |
| Multi-omics (metaG/metaRS/MIND) | Zengler Lab | Emma Rooholfada | 0, 3 |
| Conrad sample metadata (Sample Data + Micro Data sheets) | Conrad Lab | Dahen Ibarra Munoz | 4 |

---

## How to Run

- **Entry point:** `protect_pipeline_run.py` at pipeline root on server
- **Config:** edit `config/run_config_<DATE>.yaml` — update input file paths for new monthly data
- **Command:** `python protect_pipeline_run.py --config config/run_config_<DATE>.yaml`
- **Selective stages:** use `--stages 0 1 2 3 4` flags. Stage 4 is a leaf stage and can be re-run independently when only the Conrad metadata changes (e.g., `--stages 4`)
- Outputs land in a dated subdirectory; QA report and run manifest are always produced

---

## Output Files

| Output | Description |
|---|---|
| `protect_isolate_sample_patient_linkage_v{N}.csv` | Foundational linkage table — isolate → sample → patient |
| `protect_redcap_clinical_clean_<DATE>.csv` | Long-format, decoded, DQ-flagged clinical data |
| `protect_clinical_isolate_sample_patient_merged_<DATE>.csv` | Clinical + isolate + sample join |
| `protect_multiomics_isolate_sample_patient_integration_<DATE>.csv` | Full integration including metaG/metaRS/MIND/PA |
| `protect_conrad_sample_data_clean_<DATE>.csv` | Bronze cleaned Conrad sample-level metadata (1 row per sample) |
| `protect_conrad_micro_data_clean_<DATE>.csv` | Bronze cleaned Conrad clinical microbiology cultures (1 row per culture) |
| `protect_multiomics_isolate_sample_patient_conrad_integration_<DATE>.csv` | Multi-omics integration table with Conrad sample-level data merged in (`_conrad`-suffixed columns) |
| `protect_pipeline_qa_report_<DATE>.md` | All QA check results |
| `protect_pipeline_run_log_<DATE>.md` | Run manifest (inputs, outputs, run metadata) |

---

## Full Documentation

The full technical documentation — including stage-by-stage logic, data stream details, QA philosophy, known issues, and version history — lives on the server at the path above and is not reproduced here to avoid dual-maintenance.

Any related reference docs added to this repo in the future will be placed in the [`data-infrastructure/integration-pipeline/`](./) subdirectory.

---

## Related Documentation

- [`data-infrastructure/README.md`](../README.md)
- [`metadata/conrad/redcap/redcap_pipeline/README.md`](../../metadata/conrad/redcap/redcap_pipeline/README.md) — Stage 1 historical context
- [`data-infrastructure/kbase-lakehouse/README.md`](../kbase-lakehouse/README.md) — future ingestion target for pipeline outputs
- [`operations/integration-pipeline/integration_pipeline_active_issues.md`](../../operations/integration-pipeline/integration_pipeline_active_issues.md) — active issues
