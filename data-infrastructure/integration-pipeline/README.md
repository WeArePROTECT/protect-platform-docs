# PROTECT Data Integration Pipeline

**Maintainer:** Spencer Long (Arkin Lab / LBNL)
**Version:** v1.0
**Server path:** `/usr2/people/protect/Arkin_Lab/protect_data/protect_data_integration_pipeline/`

This directory documents the PROTECT Data Integration Pipeline — the production system that unifies all four PROTECT data streams (isolate records, clinical metadata, isolation provenance, and multi-omics) into a set of analysis-ready linked tables.

---

## Data Streams

| Stream | Source | Owner |
|---|---|---|
| Bacterial isolate records | Arkin Lab (ASMA) | Sun-Young Kim / Spencer Long |
| Clinical metadata | Conrad Lab (REDCap) | Dahen Ibarra Munoz |
| Isolation provenance (APL) | Arkin Lab | Sun-Young Kim |
| Multi-omics (metaG/metaRS/MIND) | Zengler Lab | Emma Rooholfada |

---

## How to Run

- **Entry point:** `protect_pipeline_run.py` at pipeline root on server
- **Config:** edit `config/run_config_<DATE>.yaml` — update input file paths for new monthly data
- **Command:** `python protect_pipeline_run.py --config config/run_config_<DATE>.yaml`
- **Selective stages:** use `--stages 0 1 2 3` flags
- Outputs land in a dated subdirectory; QA report and run manifest are always produced

---

## Output Files

| Output | Description |
|---|---|
| `protect_isolate_sample_patient_linkage_v{N}.csv` | Foundational linkage table — isolate → sample → patient |
| `protect_redcap_clinical_clean_<DATE>.csv` | Long-format, decoded, DQ-flagged clinical data |
| `protect_clinical_isolate_sample_patient_merged_<DATE>.csv` | Clinical + isolate + sample join |
| `protect_multiomics_isolate_sample_patient_integration_<DATE>.csv` | Full integration including metaG/metaRS/MIND/PA |
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
