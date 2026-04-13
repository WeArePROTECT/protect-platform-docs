# REDCap Pipeline — Active Issues

**Last Updated:** April 12, 2026  
**Maintainer:** Spencer Long (Arkin Lab / LBNL)  
**Related pipeline:** `protect_data_integration_pipeline/` → Stage 1 (`stage1_redcap_clean.py`)  
**Related docs:** `metadata/conrad/redcap/redcap_pipeline/PROTECT_REDCap_Pipeline_Documentation_v3.md`, `PROTECT_Data_Integration_Pipeline_Documentation_v1_0.md`

---

## Status at a Glance

| # | Issue | Owner | Blocking analysis? | Status |
|---|---|---|---|---|
| CCS-47 | PRO76 vs PRO76M isolate reassignment | Sun-Young Kim | No | ⏳ Waiting |

---

## Active Issues

### CCS-47 — PRO76 vs PRO76M Isolate Reassignment ⏳ HIGH
**Owner:** Sun-Young Kim (Berkeley wet lab)  
**Blocking analysis?** No — pipeline runs correctly; this is a labeling question only.

Vishant Gandhi (UCSD) confirmed that the plates corresponding to PRO76 actually came from PRO76M (a fresh Fs mouth rinse sample), not PRO76 (Home Fz frozen). The ASMA linkage table currently assigns those 42 isolates to `PRO76`. This is a discrepancy between Vishant's isolation sheet and Sun-Young's notes. Currently PRO76 is classified as `frozen_only` in `isolation_source_type`. If the ASMA linkage table is corrected to PRO76M, those isolates will automatically reclassify as `mouth_rinse` on the next pipeline run — no code changes needed.

**Resolution path:** Sun-Young Kim confirms the correct assignment → linkage table corrected → pipeline re-run.

---

## Resolved Items

### CCS-57 — Stage 1: Retire RACE_FREE_TEXT_MAP, Decode Race/Ethnicity from Dropdown Codes ✅ Resolved April 12, 2026
`RACE_FREE_TEXT_MAP`, `ETHNICITY_FREE_TEXT_MAP`, and the `_normalize_race_ethnicity()` function removed from `stage1_redcap_clean.py`. Race and ethnicity were already present in `CODE_MAPS` — the free-text path was running redundantly. All three deprecated DQ flags removed. Verified: race and ethnicity decode to human-readable labels, zero deprecated flags in output, zero UNKNOWN_CODE values. Flagged rows dropped from 76 → 29 (only valid DQ flags remain).

### CCS-58 — Config Template Bug: linkage_table Missing for REDCap-Only Runs ✅ Resolved April 12, 2026
Two fixes implemented: (1) `linkage_table` field in `run_config_3_25_26.yaml` is now explicit and populated with the v4_1 path, with a comment explaining when to update it. (2) Runtime guard added in `protect_pipeline_run.py` — raises a clear `ValueError` with an actionable message if Stage 0 is skipped and `linkage_table` is not set, rather than failing silently downstream.

### CCS-48 — Race/Ethnicity REDCap Dropdown Conversion ✅ Resolved April 12, 2026
Dahen (Conrad Lab) completed the REDCap dropdown conversion. Confirmed live in the March 30, 2026 export — all race and ethnicity values now arrive as numeric codes across all visit blocks.

### CCS-51 — cftr_modulator_status NaN for Patients 534 and 538 ✅ Resolved April 12, 2026
Both patients now have `cftr_modulator_status = 6` ("None" — no CFTR modulator) for visits v1–v4 in the March 30, 2026 export. NaN on v5+ is expected (visits not yet completed).

### CCS-49 — REDCap Direct Export Access for Spencer
Administrative item — no pipeline impact. Not tracked here.
