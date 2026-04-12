# REDCap Pipeline — Active Issues

**Last Updated:** April 12, 2026  
**Maintainer:** Spencer Long (Arkin Lab / LBNL)  
**Related pipeline:** `protect_data_integration_pipeline/` → Stage 1 (`stage1_redcap_clean.py`)  
**Related docs:** `metadata/conrad/redcap/redcap_pipeline/PROTECT_Pipeline_Documentation_v3.md`, `PROTECT_Data_Integration_Pipeline_Documentation_v1_0.md`

---

## Status at a Glance

| # | Issue | Owner | Blocking analysis? | Status |
|---|---|---|---|---|
| CCS-47 | PRO76 vs PRO76M isolate reassignment | Sun-Young Kim | No | ⏳ Waiting |
| CCS-57 | Stage 1: retire RACE_FREE_TEXT_MAP, decode race/ethnicity from dropdown codes | Spencer | No | 📋 To Do |
| CCS-58 | Config template bug: `linkage_table` not set for REDCap-only runs causes silent Stage 2 failure | Spencer | No | 📋 To Do |

---

## Active Issues

### CCS-47 — PRO76 vs PRO76M Isolate Reassignment ⏳ HIGH
**Owner:** Sun-Young Kim (Berkeley wet lab)  
**Blocking analysis?** No — pipeline runs correctly; this is a labeling question only.

Vishant Gandhi (UCSD) confirmed that the plates corresponding to PRO76 actually came from PRO76M (a fresh Fs mouth rinse sample), not PRO76 (Home Fz frozen). The ASMA linkage table currently assigns those 42 isolates to `PRO76`. This is a discrepancy between Vishant's isolation sheet and Sun-Young's notes. Currently PRO76 is classified as `frozen_only` in `isolation_source_type`. If the ASMA linkage table is corrected to PRO76M, those isolates will automatically reclassify as `mouth_rinse` on the next pipeline run — no code changes needed.

**Resolution path:** Sun-Young Kim confirms the correct assignment → linkage table corrected → pipeline re-run.

---

### CCS-57 — Stage 1: Retire RACE_FREE_TEXT_MAP, Decode Race/Ethnicity from Dropdown Codes 📋 HIGH
**Owner:** Spencer Long  
**Blocking analysis?** No — data is decoding correctly. This is a code cleanup and DQ flag noise issue.

The REDCap dropdown conversion for race and ethnicity was confirmed live in the March 30, 2026 delivery (CCS-48 resolved April 12, 2026). All race and ethnicity values now arrive as numeric codes. However, `stage1_redcap_clean.py` still runs the legacy `RACE_FREE_TEXT_MAP` interim normalization path, causing `race_ethnicity_interim_normalized|verify_on_next_export` to appear on every row as a code artifact rather than a real data quality signal.

**Work required:** (1) Add race and ethnicity code mappings to `CODE_MAPS` from `PROTECT_DataDictionary_2026-03-06.csv`. (2) Remove `RACE_FREE_TEXT_MAP`, `ETHNICITY_FREE_TEXT_MAP`, and the interim normalization step in Step 5. (3) Remove the three deprecated DQ flags from the flag logic and from documentation. (4) Verify: re-run Stage 1 against `PROTECT_RawDataExport_2026.03.30_1622.csv`, confirm race/ethnicity decode to labels and no interim flags appear.

---

### CCS-58 — Config Template Bug: linkage_table Missing for REDCap-Only Runs 📋 HIGH
**Owner:** Spencer Long  
**Blocking analysis?** No — workaround is to set `linkage_table` manually in config. But it will silently fail for any operator who doesn't know to do this.

When running Stage 1–3 only (`--stages 1 2 3`), Stage 2 requires `inputs.linkage_table` to be set explicitly in the YAML config. The current config template leaves this field commented out. When absent, the pipeline logs `Stage 0 skipped — using existing linkage: None` and Stage 2 fails downstream with a non-obvious error. Discovered during the April 12, 2026 pipeline run.

**Work required:** Option A — uncomment `linkage_table` in the config template with a clear note that it must be set when Stage 0 is skipped, pointing to the current linkage table version. Option B — add a startup validation guard that raises a clear `ConfigurationError` when Stage 0 is skipped and `linkage_table` is absent. Recommend both.

---

## Resolved Items

### CCS-48 — Race/Ethnicity REDCap Dropdown Conversion ✅ Resolved April 12, 2026
Dahen (Conrad Lab) completed the REDCap dropdown conversion. Confirmed live in the March 30, 2026 export — all race and ethnicity values now arrive as numeric codes across all visit blocks. Code cleanup tracked as CCS-57.

### CCS-51 — cftr_modulator_status NaN for Patients 534 and 538 ✅ Resolved April 12, 2026
Both patients now have `cftr_modulator_status = 6` ("None" — no CFTR modulator) for visits v1–v4 in the March 30, 2026 export. NaN on v5+ is expected (visits not yet completed).

### CCS-49 — REDCap Direct Export Access for Spencer
Administrative item — no pipeline impact. Not tracked here.
