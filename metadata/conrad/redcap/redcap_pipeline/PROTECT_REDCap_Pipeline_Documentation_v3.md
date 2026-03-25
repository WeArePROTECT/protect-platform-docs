# PROTECT Data Integration Pipeline — Documentation

**Project:** PROTECT CF Study  
**Authors:** Spencer Long (Berkeley / Arkin Lab)  
**Last updated:** March 18, 2026 (v3)  
**Pipeline version:** protect_pipeline_v3.py  
**Notebook version:** PROTECT_Data_Integration_v4.ipynb  
**Status:** Production — validated and running

---

## Changelog from v2 → v3

| Change | Details |
|---|---|
| **4th input source added** | `APL_metadata_20260303.xlsx` — the authoritative record of fresh vs frozen isolation origin |
| **Frozen→isolation workflow understood** | Fz/Home Fz samples can legitimately have ASMA isolates; confirmed by Sun-Young Kim and Vishant Gandhi (March 17, 2026) |
| **`fz_sample_has_isolates` DQ flag removed** | Was flagging valid biology as an anomaly — completely removed |
| **`isolation_source_type` column added** | New master merged column classifying each isolate as `fresh_only`, `frozen_only`, `fresh_and_frozen`, `mouth_rinse`, or `not_in_apl` |
| **5 additional APL columns added** | `apl_has_frozen`, `apl_sampling_types`, `apl_n_plates`, `apl_media`, `apl_earliest_cp1` |
| **Master merged: 70 → 76 columns** | 6 new APL-derived columns |
| **SampleType routing rule revised** | Fz/Home Fz samples are no longer treated as categorically excluded from ASMA |
| **LINKAGE_PATH updated to v3** | `patient_sputum_asma_gold_linkage_table_v3.csv` |
| **PRO76 vs PRO76M — resolved** | Both are confirmed distinct samples. PRO76 (Home Fz, frozen) → 42 isolates; PRO76M (Fs, mouth rinse) → 25 isolates. Pipeline correct as-is. Confirmed by Sun-Young Kim (March 18, 2026) |

---

## 1. Project Context

The PROTECT study is a longitudinal cystic fibrosis (CF) research study collecting sputum samples from CF patients at UC San Diego (Conrad Lab). The goal of this pipeline is to build complete data lineage from patient clinical metadata all the way down to individual bacterial isolate genomes, producing analysis-ready outputs suitable for manuscript preparation and ARPA-H reporting.

**Teams:**
- **Conrad Lab (UCSD):** Dahen Ibarra Munoz (CRC), Praveen Akuthota (clinician), Jenna Mielke — own the clinical site, manage REDCap, collect and ship samples. Vishant Gandhi (UCSD) — involved in frozen sputum plating and PA-targeted isolation.
- **Arkin Lab (Berkeley/LBNL):** Spencer Long, Adam Arkin (PI), Ezgi Booth (coordinator), Jake Hilzinger (staff scientist), Sun-Young Kim (wet lab) — own the ASMA platform, process isolates, run bioinformatics

**Data flow:**
```
Conrad Lab (UCSD)          →   PROTECT Server   →   Berkeley (Arkin Lab)
  Clinic visits                                        ASMA isolate platform
  REDCap clinical DB                                   KBase data lakehouse
  Samples shipped to Berkeley
  APL_metadata (Berkeley wet lab)
```

---

## 2. Input Datasets

The pipeline requires **four** input files. All must be present before running.

### 2A. REDCap Raw Export
**File pattern:** `PROTECT_RedCapDataExport_<DATE>__raw_data_.csv`  
**Owner:** Dahen Ibarra Munoz (UCSD / Conrad Lab)  
**Delivery cadence:** Monthly export to PROTECT server (~15–18 new samples per delivery). Full dataset (~280 samples) expected by end of May 2026. Data is entered patient-by-patient — each delivery will contain complete data for a subset of patients, not the next N samples in numerical order.  
**Shape:** 9 patients × 693 columns as of March 2, 2026 export  
**Format:** Wide — one row per patient, up to 15 repeating visit instrument blocks

**Block column naming convention — critical:**
- Block 1 (first visit): bare field names — `sampleid`, `day_of_assessment`, `fev1_pp`, etc.
- Blocks 2–15: `_vN` suffix — `sampleid_v2`, `day_of_assessment_v2`, `fev1_pp_v2`, etc.

> ⚠️ **The suffix is `_vN`, not `_N`.** An earlier notebook version incorrectly searched for `_2`, `_3` etc. and silently missed all visits beyond block 1. This has been fixed since v2.

**Key columns:**

| Column | Description |
|---|---|
| `record_id` | REDCap internal record ID |
| `subject_id` | Numeric patient ID — matches Samples sheet and ASMA |
| `sampleid` | Numeric sample ID for block 1; `sampleid_vN` for blocks 2–15 |
| `day_of_assessment` | Study day of the clinic visit |
| `assessment_of_pex` | Pulmonary exacerbation status (numeric code) |
| `antibiotic_status` | Antibiotic treatment status (numeric code) |
| `cftr_modulator_status` | CFTR modulator medication (numeric code) |
| `fev1_l`, `fev1_pp` | FEV1 in liters and percent predicted |
| `fvc_l`, `fvc_pp` | FVC in liters and percent predicted |
| `inhaled_tobramycin_tobi` | Cycling antibiotic (numeric code) |
| `sex_at_birth`, `race`, `ethnicity` | Demographics |

### 2B. ASMA Linkage Table
**File:** `patient_sputum_asma_gold_linkage_table_v3.csv` (updated from v2)  
**Owner:** Berkeley / Arkin Lab (Sun-Young Kim, wet lab)  
**Shape:** ~5,019 rows × 9 columns  
**Grain:** One row per bacterial isolate cultured from a PROTECT sputum sample

**Key columns:**

| Column | Description |
|---|---|
| `ASMA_id` | Unique isolate ID (e.g. `ASMA-1`) |
| `patient_id` | Numeric patient ID — matches REDCap `subject_id` |
| `sample_id` | PRO-format sample ID (e.g. `PRO1`, `PRO7M`) |
| `patient_type` | `adult`, `pediatric`, or `healthy_donor` |
| `sampling_method` | `sputum`, `oral_rinse`, etc. |
| `sampling_site` | e.g. `lower_respiratory_tract`, `oral_cavity` |
| `isolation_media` | e.g. `BHI_broth` |
| `stock_plate` | Freezer box name |
| `stock_well` | Well position within box |

**ASMA sample ID suffix conventions:**

| Suffix | Meaning |
|---|---|
| None (e.g. `PRO7`) | Sputum/swab, adult |
| `M` (e.g. `PRO7M`) | Mouth rinse, adult |
| `P` (e.g. `PRO7P`) | Sputum/swab, pediatric |
| `MP` (e.g. `PRO7MP`) | Mouth rinse, pediatric |
| `HD*` | Healthy donor — excluded from all joins by default |

### 2C. PROTECT Samples Sheet
**File:** `PROTECT_Samples_-_Sheet1__1_.csv`  
**Owner:** Berkeley / Arkin Lab  
**Shape:** 241 rows × 12 columns  
**Grain:** One row per physical sample collected

This is the official bridge between REDCap and ASMA. It contains both `SampleNumber` (matches REDCap `sampleid`) and `SampleID` (the PRO-format ID used in ASMA), confirming the numeric-to-PRO mapping.

**SampleType routing codes:**

| Code | Meaning | Typical Destination | Appears in ASMA? |
|---|---|---|---|
| `Fs` | Fresh sample | Berkeley isolation | ✅ Yes |
| `Fz` | Frozen sample | Omics pipeline primarily | ⚠️ Sometimes — see below |
| `Fs-Fz` | Split — both | Berkeley isolation + omics | ✅ Yes (isolation portion) |
| `Home Fz` | Home-collected, frozen | Omics pipeline primarily | ⚠️ Sometimes — see below |

> ⚠️ **Important update from v2:** Fz and Home Fz samples can legitimately appear in ASMA. Frozen sputum was deliberately plated for isolation on multiple occasions (confirmed March 17, 2026 by Sun-Young Kim and Vishant Gandhi). See Section 2D and Section 9 for full context. The Samples sheet SampleType correctly reflects the physical format — it does not constrain whether the sample was also processed for isolation. Use the `isolation_source_type` column in the master merged output (derived from APL_metadata) to determine actual isolation origin.

### 2D. APL Metadata *(new in v3)*
**File:** `APL_metadata_20260303.xlsx`  
**Owner:** Berkeley / Arkin Lab (Sun-Young Kim, wet lab)  
**Shape:** 358 rows × 13 columns  
**Grain:** One row per APL isolation plate

This is the **authoritative record of fresh vs frozen isolation origin** for ASMA isolates. The Samples sheet records the physical format of each sample as collected; APL_metadata records what Berkeley actually did with it in the lab.

**Key columns:**

| Column | Description |
|---|---|
| `UCSD_sample_format` | PRO-format sample ID — join key to master merged `pro_sample_id` |
| `sampling_site_type` | `F`=fresh sputum, `FZ`=frozen sputum, `M`=mouth rinse adult, `MP`=mouth rinse pediatric, `P`=pediatric sputum, `FP`=fresh pediatric |
| `APL_id` | Unique APL plate identifier |
| `media` | Isolation media used (e.g. `BHI_broth`, `BHI-Blood_agar`) |
| `CP1_generation_date` | Date of first colony pick |
| `lab` | Receiving lab (e.g. `Zengler`) |

**Why frozen sputum was plated — three distinct events (Sun-Young Kim, March 17, 2026):**

1. **Initial ASMA diversity sweep** — To capture taxonomic diversity that fresh-only sampling might miss, the team requested previously frozen samples originally intended for omics. This produced isolates for PRO8, PRO15, PRO22, PRO23, PRO30, and PRO76. UCSD did not record this in REDCap or the Samples sheet as it was handled ad hoc.

2. **Supplemental diversity isolation** — For some PRO IDs where fresh sample diversity appeared insufficient, additional isolations were attempted from the stored frozen material (e.g. PRO17, PRO66, PRO84, PRO85, PRO102). These appear as `FZ` entries in APL_metadata alongside `F` entries for the same PRO ID.

3. **PA-targeted isolation (ongoing)** — Vishant Gandhi (UCSD) identified PA-positive frozen samples via omics, plated them at UCSD, and shipped to Berkeley for further isolation. His confirmed frozen list: PRO8, 9, 16, 19, 21, 22, 23, 24, 33, 42, 60, 73, 78, 81, 82, 90, 101, 106, 123, 124, 125, 127, 131, 139. Future linkage table updates should be expected from this group.

**Join grain:** APL_metadata has multiple rows per PRO ID (one per plate). The pipeline summarises to one row per PRO ID before joining into the master merged table.

---

## 3. Confirmed ID Mapping

Validated computationally (0 mismatches) and confirmed by Dahen on March 5, 2026:

```
REDCap sampleid N  ==  Samples SampleNumber N  ==  ASMA sample_id PRO{N}
REDCap subject_id  ==  Samples PatientNumber   ==  ASMA patient_id
```

The pipeline re-validates this on every run (Section 6 / crosswalk validation) and prints a clear error if any mismatch is detected.

---

## 4. Pipeline Architecture

```
INPUTS
  REDCap raw CSV          (wide: 1 row/patient, up to 693 cols)
  ASMA linkage table      (1 row/isolate)
  PROTECT Samples sheet   (1 row/sample, 241 rows)
  APL metadata            (1 row/plate → summarised to 1 row/PRO ID)  ← NEW in v3

PIPELINE — protect_pipeline_v3.py / PROTECT_Data_Integration_v4.ipynb
  Step 1    Load raw data + load & summarise APL_metadata           [UPDATED in v3]
  Step 2    Reshape REDCap wide → long    (9 patients × 15 blocks → 27 visit rows)
  Step 3    Filter to rows with data; split multi-sample IDs
  Step 4    Decode numeric codes → human-readable labels
  Step 5    Standardize free-text fields + interim race/ethnicity normalization
  Step 6    Add derived columns (BMI, FEV1/FVC ratio, antibiotic counts)
  Step 7    Data quality flagging
  Step 8    Save clean REDCap output   →  PROTECT_REDCap_<date>_pipeline_v3.csv
  Step 9    Crosswalk validation
  Step 10   Build master merged table                                [UPDATED in v3]
              → Join Samples + REDCap + ASMA linkage + APL_metadata
              → Add isolation_source_type column
              → Remove fz_sample_has_isolates flag
            →  PROTECT_clinical_isolate_linked_<date>_pipeline_v3.csv

OUTPUTS
  PROTECT_REDCap_<date>_pipeline_v3.csv       (silver layer — 1 row/sample visit, decoded + flagged)
  PROTECT_clinical_isolate_linked_<date>_pipeline_v3.csv      (gold layer — 1 row/isolate, all 4 sources joined)
```

---

## 5. Output Schemas

### Output 1: `PROTECT_REDCap_<date>_pipeline_v3.csv` — Silver Layer
**Grain:** One row per sample visit  
**Current shape:** 27 rows × 58 columns  
**Unchanged from v2.**

Column groups in order:

| Group | Key Columns |
|---|---|
| Identity | `record_id`, `subject_id`, `sampleid`, `pro_sample_id`, `visit_block`, `multi_sample_flag`, `original_sampleid_group`, `day_of_assessment`, `dq_flags` |
| Demographics | `sex_at_birth`, `race`, `ethnicity`, `lung_function_age_years`, `ht_cm`, `weight_kg`, `bmi` |
| Clinical | `assessment_of_pex`, `antibiotic_status`, `cftr_modulator_status` |
| Lung function | `fev1_l`, `fev1_pp`, `fvc_l`, `fvc_pp`, `fev1_fvc_ratio` |
| Antibiotic summary | `n_active_antibiotics`, `any_iv_antibiotics`, `n_inhaled_cycling_on` |
| Antibiotic detail | Inhaled cycling cols + 22 binary abx columns |

### Output 2: `PROTECT_clinical_isolate_linked_<date>_pipeline_v3.csv` — Gold Layer
**Grain:** One row per bacterial isolate (Fs/Fz isolation samples); one row per sample (omics-only samples with no isolates)  
**Current shape:** 4,405 rows × 76 columns (was 70 in v2 — 6 APL columns added)  

**Coverage flags:**

| Column | Meaning |
|---|---|
| `has_redcap_metadata` | True if REDCap has a clinical record for this sample |
| `has_isolates` | True if ASMA has isolates banked for this sample |
| `fully_linked` | True if both — the analytically gold-standard rows |
| `sent_to_isolation` | True if SampleType is Fs or Fs-Fz |

**New APL-derived columns (v3):**

| Column | Values | Meaning |
|---|---|---|
| `isolation_source_type` | `fresh_only`, `frozen_only`, `fresh_and_frozen`, `mouth_rinse`, `not_in_apl` | How the isolate was derived — the key distinction for analysis |
| `apl_has_frozen` | `True` / `False` | Whether any plate for this PRO ID used frozen sputum |
| `apl_sampling_types` | Pipe-separated string (e.g. `F\|FZ`) | All sampling types seen across plates for this PRO ID |
| `apl_n_plates` | Integer | Total APL plates processed for this PRO ID |
| `apl_media` | Pipe-separated string | All isolation media used |
| `apl_earliest_cp1` | Date | Date of earliest CP1 generation |

**Current `isolation_source_type` breakdown (March 2026):**

| Value | Row count | Description |
|---|---|---|
| `fresh_only` | 2,476 | Isolated exclusively from fresh sputum |
| `mouth_rinse` | 1,210 | Derived from mouth rinse samples |
| `fresh_and_frozen` | 362 | PRO ID has both fresh and frozen plates (e.g. PRO102, PRO17) |
| `frozen_only` | 212 | Isolated exclusively from frozen sputum (PRO8, 15, 22, 23, 30, 76) |
| `not_in_apl` | 145 | Sample not yet processed through APL |

**Current coverage (March 2026):**

| Status | Count |
|---|---|
| Fully linked (REDCap + isolates) | 643 isolates |
| Has REDCap only (no isolates yet) | 15 |
| Has isolates only (no REDCap yet) | 3,616 |
| Samples sheet only (neither yet) | 131 |

---

## 6. Code Maps (Confirmed March 6, 2026)

All confirmed from `PROTECT_DataDictionary_2026-03-06.csv`. Unchanged from v2.

**Two encoding types exist — do not mix them:**

**Inhaled cycling antibiotics** (`inhaled_tobramycin_tobi`, `inhaled_aztreonam_cayston`, `inhaled_amikacin_arikayce`) use a 4-option cycling scale:

| Code | Label |
|---|---|
| 1 | On Cycle |
| 2 | Off Cycle |
| 3 | Not Taking |
| 4 | Unknown/Unavailable |

**All other antibiotic fields** use binary Yes/No: `1=Yes, 2=No`

**Named field maps:**

| Field | Codes |
|---|---|
| `assessment_of_pex` | 1=Stable, 2=Mild Exacerbation, 3=Moderate Exacerbation, 4=Severe Exacerbation, 5=Uncertain |
| `antibiotic_status` | 1=Chronic, 2=Acute, 3=None |
| `cftr_modulator_status` | 1=Trikafta, 2=Kalydeco, 3=Symdeko, 4=Orkambi, 5=Alyftrek, 6=None, 7=Unknown/Unavailable |
| `sex_at_birth` | 1=Male, 2=Female |
| `race` | 1=American Indian or Alaska Native, 2=Asian, 3=Black or African American, 4=Native Hawaiian or Other Pacific Islander, 5=Other Race or Mixed Race, 6=Unknown, 7=White |
| `ethnicity` | 1=Not Hispanic Latino(a) or Spanish origin, 2=Other Hispanic Latino(a) or Spanish origin, 3=Unreported |

---

## 7. Race/Ethnicity Interim Normalization

As of the March 2, 2026 export, the `race` and `ethnicity` fields arrive as free-text strings rather than numeric dropdown codes. Dahen confirmed on the March 5, 2026 call that these fields will be converted to dropdowns in REDCap (Q7 resolution). Once that conversion is live and a new export is received, the fields will arrive as numeric codes and decode automatically through CODE_MAPS.

**Until then**, the pipeline applies an interim normalization step in Step 5 that maps known free-text values to their canonical equivalents. Any unrecognized value surfaces as `FREE_TEXT_NEEDS_REVIEW: <value>` — never silently converted or dropped. Normalized rows receive a `race_ethnicity_interim_normalized|verify_on_next_export` DQ flag.

**Action required on every new export:** Check whether race/ethnicity now arrive as numeric codes. If yes, retire `RACE_FREE_TEXT_MAP` and `ETHNICITY_FREE_TEXT_MAP` from Section 1 of the notebook — the fields will decode automatically. If still free-text, update the maps with any new values observed.

---

## 8. Data Quality Flags

The `dq_flags` column in both outputs contains pipe-separated flag strings. Rows are **never dropped** — all flagged rows remain in the dataset.

| Flag | Meaning | Action |
|---|---|---|
| `lung_function_restrictive_pattern\|seek_clinical_review` | Patient 545: FEV1% > FVC% at every visit. Confirmed valid by Dahen — restrictive pattern in spirometry reports. | Seek clinical interpretation from Dr. Akuthota before using patient 545's lung function data in analysis |
| `lung_function_spirometry_not_performed` | Patient 534: spirometry not performed at this visit (~10% of visits). NaN in lung function fields is expected. | Treat as missing — do not impute |
| `multi_sample_visit_metadata_duplicated` | Two samples collected at same visit (e.g. `7, 8`). Clinical metadata is intentionally identical for both rows — confirmed by Dahen. | Be aware when aggregating by visit |
| `contains_unknown_code` | A decoded field has a code value not in the code map. Should not appear with current dictionary. | Update CODE_MAPS in Section 1 and re-run |
| `race_ethnicity_interim_normalized\|verify_on_next_export` | Row had free-text race/ethnicity values normalized by interim map. | Verify on next export whether REDCap dropdown conversion is live |
| `race_free_text_unrecognized\|manual_review_required` | Race value could not be mapped — manual review needed. | Investigate and update `RACE_FREE_TEXT_MAP` |
| `ethnicity_free_text_unrecognized\|manual_review_required` | Ethnicity value could not be mapped — manual review needed. | Investigate and update `ETHNICITY_FREE_TEXT_MAP` |

> **Removed in v3:** The `fz_sample_has_isolates|sampletype_may_be_miscoded|needs_lab_verification` flag has been removed. The frozen→isolation workflow has been confirmed as valid biology (see Section 2D). These rows are no longer anomalies and are correctly classified via `isolation_source_type`.

---

## 9. Known Open Items

### ~~Open Item A: PRO76 vs PRO76M~~ — RESOLVED ✅ (March 18, 2026)
**Owner:** Sun-Young Kim (Berkeley wet lab)  
**Jira:** CCS-47 (Done)

Sun-Young Kim confirmed via documentation (March 18, 2026) that PRO76 and PRO76M are two genuinely distinct samples for the same patient, both present in the PROTECT Samples sheet (created by Vishant Gandhi):

- **PRO76** (Home Fz, frozen sputum) → 42 ASMA isolates → `isolation_source_type = frozen_only` ✅
- **PRO76M** (Fs, fresh mouth rinse) → 25 ASMA isolates → `isolation_source_type = mouth_rinse` ✅

Sun-Young assigned separate APL IDs to each in APL_metadata, and both counts match the pipeline output exactly. Vishant's earlier comment that "plates from PRO76 should be PRO76M" was likely a confusion — his own Samples sheet lists both as separate entries, which is the authoritative record. **The pipeline requires no changes.**

### Open Item B: Vishant's Remaining Frozen Plates — Pipeline Ready ✅
**Owner:** Vishant Gandhi (UCSD) / Sun-Young Kim (Berkeley)

Vishant provided a list of 24 PRO IDs plated from frozen sputum. Of these, only 3 (PRO8, PRO22, PRO23) currently have ASMA isolates. The remaining 21 (PRO9, 16, 19, 21, 24, 33, 42, 60, 73, 78, 81, 82, 90, 101, 106, 123, 124, 125, 127, 131, 139) were plated but are not yet banked. The pipeline will handle these correctly when they appear in future linkage table updates — no false anomaly flag will trigger.

### Open Item C: Race/Ethnicity Dropdown Conversion 📋 MEDIUM
**Owner:** Dahen (REDCap), Spencer (pipeline verification)  
**Jira:** CCS-48

Pending Dahen completing the REDCap dropdown conversion for race and ethnicity. Once live, verify on the next export that these fields arrive as numeric codes, then retire the interim normalization maps from Section 1 of the notebook.

### Open Item D: Ad Hoc Frozen Isolation Provenance Notes in ASMA_metadata_SK 📋 LOW
**Owner:** Sun-Young Kim (Berkeley wet lab)

The ad hoc frozen isolations (PRO8, PRO15, PRO22, PRO23, PRO30, PRO76) were not formally documented by UCSD at the time they occurred. The biological and methodological context is fully documented in this pipeline documentation (Section 2D) and in the GitHub active issues log. The outstanding question is whether it is also useful to add a brief provenance note at the ASMA record level in `ASMA_metadata_SK` — for example a notes field entry reading "frozen sputum, initial diversity sweep, not recorded by UCSD at time of collection."

Sun-Young has been asked whether `ASMA_metadata_SK` has an appropriate notes or comments field for this purpose. If such a field exists, adding short notes for these six PRO IDs would help anyone reading the ASMA records directly understand the isolation context without needing to cross-reference pipeline documentation. If no suitable field exists, the pipeline documentation is sufficient and no further action is needed.

---

## 10. Monthly Update Procedure

When a new REDCap export arrives:

1. Place the new CSV in the data directory (or update the PROTECT server path)
2. Open `PROTECT_Data_Integration_v4.ipynb` and update `REDCAP_RAW_PATH` in Section 0
3. If a new `APL_metadata` file has been received from Sun-Young, update `APL_METADATA_PATH` too
4. Run all cells top-to-bottom
5. Review the Section 9 summary report output:
   - ✅ Crosswalk validation passes with 0 patient ID mismatches
   - ✅ No `UNKNOWN_CODE_X` values in DQ flags
   - ✅ `race_ethnicity_interim_normalized` flag disappears if REDCap dropdown is now live
   - ✅ Row and patient counts increased as expected
   - ✅ `isolation_source_type` breakdown looks consistent with expectations
   - ✅ No `FREE_TEXT_NEEDS_REVIEW` values in race/ethnicity fields
6. Archive previous output CSVs before overwriting

Alternatively run `python protect_pipeline_v3.py` directly from the command line.

---

## 11. Data Lineage Overview

```
SOURCE (UCSD / Conrad Lab)
  REDCap database
    ↓ monthly export to PROTECT server
  PROTECT_RedCapDataExport_<DATE>__raw_data_.csv

SOURCE (Berkeley / Arkin Lab)
  ASMA isolate platform
    ↓ maintained by Sun-Young Kim
  patient_sputum_asma_gold_linkage_table_v3.csv
  PROTECT_Samples_-_Sheet1__1_.csv

SOURCE (Berkeley / Arkin Lab — NEW in v3)
  APL isolation tracking
    ↓ maintained by Sun-Young Kim
  APL_metadata_20260303.xlsx

SOURCE (UCSD / Vishant Gandhi — context only)
  Frozen sputum plating records
  (not yet formally in pipeline — informs APL_metadata)

PIPELINE (Berkeley / Spencer Long)
  protect_pipeline_v3.py  /  PROTECT_Data_Integration_v4.ipynb
    ↓ outputs to PROTECT server
  PROTECT_REDCap_3_2_2026_pipeline_v3.csv         (silver — visit-level clinical data)
  PROTECT_clinical_isolate_linked_3_2_2026_pipeline_v3.csv  (gold — isolate-level, all 4 sources joined)
    ↓ loaded into
  KBase data lakehouse             (for genomics and downstream analysis)
```

---

## 12. File Inventory

| File | Role | Owner | Notes |
|---|---|---|---|
| `protect_pipeline_v3.py` | Main pipeline script — run monthly | Berkeley | Update `REDCAP_RAW_PATH` and `APL_METADATA_PATH` at top |
| `PROTECT_Data_Integration_v4.ipynb` | Interactive notebook version | Berkeley | Same logic as pipeline script |
| `PROTECT_REDCap_<date>_pipeline_v3.csv` | Output: silver layer, 1 row/visit | Pipeline output | Date = raw REDCap export date (e.g. `3_2_2026`) |
| `PROTECT_clinical_isolate_linked_<date>_pipeline_v3.csv` | Output: gold layer, 1 row/isolate | Pipeline output | 76 cols; date matches silver layer |
| `PROTECT_RedCapDataExport_*.csv` | Input: raw REDCap export | UCSD / Dahen | Update path each month |
| `patient_sputum_asma_gold_linkage_table_v3.csv` | Input: ASMA isolate records | Berkeley / Sun-Young | Updated from v2 |
| `PROTECT_Samples_-_Sheet1__1_.csv` | Input: sample bridge sheet | Berkeley | |
| `APL_metadata_20260303.xlsx` | Input: fresh vs frozen isolation origin | Berkeley / Sun-Young | **New in v3** — update when Sun-Young releases new version |
| `PROTECT_DataDictionary_2026-03-06.csv` | REDCap code definitions | UCSD / Dahen | Reference — do not modify |
| `PROTECT_Dahen_Call_Questions_v6.docx` | Q&A log with Conrad team | Berkeley / Spencer | Updated March 17, 2026 |
| `PROTECT_Pipeline_Documentation_v3.md` | This document | Berkeley / Spencer | |

---

## 13. Guiding Principles

- **Evidence before assumptions** — never assume ID mappings or routing logic without confirmation. The crosswalk check runs on every execution. The frozen→isolation discovery (Section 2D) is a direct result of applying this principle.
- **Flag, don't drop** — data quality issues are documented in `dq_flags`. No rows are ever removed.
- **Preserve provenance** — raw input files are never overwritten. Outputs have distinct names.
- **Visible failures** — unknown codes render as `UNKNOWN_CODE_X`, not silently as NaN. Unrecognized free-text surfaces as `FREE_TEXT_NEEDS_REVIEW: <value>`.
- **Respect the multi-team dynamic** — UCSD owns REDCap and clinical data; Berkeley owns ASMA and bioinformatics. Questions about REDCap data go to Dahen; questions about isolate records go to Sun-Young; questions about frozen plating go to Vishant.
- **Monthly cadence** — pipeline re-runs with a single path change. All other logic is stable.
- **Assumptions are not facts** — the Fz routing rule was treated as a hard constraint in v2; it was wrong. Any rule derived from pattern observation rather than explicit team confirmation should be documented as an assumption and verified before acting on it.
