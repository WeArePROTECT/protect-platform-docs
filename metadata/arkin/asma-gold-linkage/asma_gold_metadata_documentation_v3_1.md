# ASMA Isolate Gold Metadata Table — Build Documentation

**Version:** v3.1
**Status:** Gold — All 29 QA checks passed
**Last Updated:** 2026-03-19
**Maintainer:** Spencer Long (Arkin Lab)
**Notebook:** `patient_sputum_asma_isolate_gold_metadata_v2_2.ipynb`

---

## 1. Purpose

This document describes the construction of the **ASMA isolate-level gold metadata table** — a single CSV that harmonizes CF and healthy donor isolates into an analysis-ready join layer.

The table bridges:

- Isolate-level datasets (taxonomy, WGS, phenotype, AMR/VF) keyed by `ASMA_id`
- Patient-level datasets keyed by `patient_id`
- Sample-level datasets keyed by `sample_id`

A core design goal is to represent sampling context in two orthogonal fields — `sampling_method` (collection technique) and `sampling_site` (anatomical origin) — to prevent silent mixing of samples from different biological compartments in downstream analysis.

---

## 2. Deliverable Outputs

| Output | Description |
|---|---|
| `patient_sputum_asma_gold_linkage_table_v3_1.csv` | Gold metadata table — 5,019 rows, 10 columns |
| `asma_gold_linkage_table_reference.md` | Data reference doc: schema, value glossary, usage examples |
| `patient_sputum_asma_isolate_gold_metadata_v2_2.ipynb` | Build notebook with embedded QC suite |

---

## 3. Source Data and Provenance

### 3.1 Sun-Young Kim — CF Isolate Metadata

**Contributor:** Sun-Young Kim (Arkin Lab)
**Role:** Canonical CF isolate ↔ patient ↔ sample linkage, plus isolate storage metadata

Source files:

| File | Description |
|---|---|
| `ASMA_metadata_SK_20260303.xlsx` | Primary canonical workbook. Combines plate-level CP1 metadata with patient and sample linkage. |
| `ASMA_metadata_SK_20260319.xlsx` | Updated workbook adding a `Notes` column. Structurally identical to the 20260303 file — same 4,990 rows, all original columns unchanged. Source of `sk_notes`. |
| `APL_metadata_20260303.xlsx` | APL plate identifier resolution table. Maps `APL:#` identifiers to `sample_id` and media type. |
| [Plate-CP1-metadata.gsheet](https://docs.google.com/spreadsheets/d/1uwGFBpR6V537Sb1finQrRh7wb0mc2L0p9yzBtp1yQX0/edit?usp=sharing) | Upstream source linking isolate-level plate IDs to sputum IDs. |
| [Protect Samples.gsheet](https://docs.google.com/spreadsheets/d/1IBG7-rSybWBHJIw2Cd9sabrfvIXU3KVUQqyt-j9_i1Y/edit?usp=sharing) | Maps `SampleNumber` to `SampleID` (= `sample_id` in this table). |

**Identifier linkage logic:**

- For ASMA IDs isolated from `PL:#` plates: `sputum_id` in Plate-CP1-metadata → `SampleNumber` in Protect Samples → `SampleID` = `sample_id`.
- For ASMA IDs isolated from `APL:#` plates: `UCSD_sample_format` in `APL_metadata_20260303.xlsx` → `SampleID` in Protect Samples → `sample_id`.

**`APL:#` temporary identifiers:** `APL:#` plate IDs are assigned by the Arkin Lab. They are unique per agar plate (sample × media combination) and are not persistent identifiers. Downstream users should resolve `APL:#` to `sample_id` + media type via `APL_metadata_20260303.xlsx` and should not treat `APL:#` as a stable primary key.

### 3.2 Jake Hilzinger — Healthy Donor Isolate Metadata

**Contributor:** Jake Hilzinger (Arkin Lab, staff scientist)
**Role:** HD isolate-to-patient/site mapping and sampling site annotation
**Source:** [20251109_healthy-asma-isolates](https://docs.google.com/spreadsheets/d/1V_7KwJUCEVcVb4GWSIysyFFq-mdP-FN85mbSQodDo08/edit)

Jake's sheet provided: `ASMA_id` for HD isolates, HD patient identifiers, anatomical sampling site (`tongue`/`throat`), and stock plate/well metadata. ASMA IDs were matched to isolate, patient, and site data using R.

**Isolate naming convention:** `HD{plate}.{isolate}` — e.g., `HD1.1` = plate 1, isolate 1; `HD1.1.2` = plate 1, isolate 1 from re-streak, isolate 2 from re-streak. Each patient had four plates.

**Known caveat:** Freezer stocks for HD patient P4 were lost. Genomes exist but physical stocks are unavailable.

---

## 4. Schema

See `asma_gold_linkage_table_reference.md` for the full column reference. Summary:

| Column | Type | Description |
|---|---|---|
| `ASMA_id` | `string` | Primary key — unique isolate identifier |
| `patient_id` | `integer` | Numeric patient ID — interpret with `patient_type` |
| `sample_id` | `string` | Sample ID for linking to sample-level metadata |
| `patient_type` | `enum` | `adult` \| `pediatric` \| `healthy_donor` |
| `sampling_method` | `enum` | `sputum` \| `oral_rinse` \| `oral_swab` |
| `sampling_site` | `enum` | `lower_respiratory_tract` \| `oral_cavity` \| `tongue` \| `throat` |
| `isolation_media` | `string` | Growth medium at isolation; null for `PL:#` CF rows (expected) |
| `stock_plate` | `string` | Freezer stock plate/box identifier |
| `stock_well` | `string` | Well position on stock plate/box |
| `sk_notes` | `string` | Provenance/availability notes from SK workbook; null for most rows |

---

## 5. Key Design Decisions

### 5.1 Separate Method from Site

Sampling context is encoded in two columns — `sampling_method` and `sampling_site` — rather than one. This enables downstream analyses to filter on anatomical origin independently of collection technique, and prevents silent merging of samples from different compartments.

### 5.2 Locked Sampling Harmonization Rules

```
sputum      →  sampling_site = lower_respiratory_tract
oral_rinse  →  sampling_site = oral_cavity
oral_swab   →  sampling_site = tongue OR throat  (HD only)
```

CF rows must not contain `oral_swab`. HD rows must contain `oral_swab`. These constraints are enforced by QA checks.

### 5.3 Terminology: `mouth_rinse` → `oral_rinse`

Standardized on `oral_rinse` to align with terminology guidance (per Alex). The corresponding site remains `oral_cavity`.

### 5.4 Deterministic HD Sample ID Construction

HD entries may not have a CP1-style `sample_id`. When missing, sample IDs are constructed deterministically:

```
Format:    HD{patient_id}_{sampling_site}
Examples:  HD1_tongue, HD1_throat
```

Controlled by notebook flag: `CONSTRUCT_HD_SAMPLE_IDS_IF_MISSING = True/False`. When disabled, missing HD sample IDs are flagged in QC.

---

## 6. Build Process Summary

The build notebook (`patient_sputum_asma_isolate_gold_metadata_v2_2.ipynb`) executes these steps:

1. Load source datasets (SK workbook + Jake HD sheet)
2. Standardize column names to the target schema
3. Normalize `patient_type` values to canonical forms
4. Derive `sampling_method` and `sampling_site` using locked harmonization rules
5. Construct HD `sample_id` when missing (if flag enabled)
6. Assemble final metadata table: concatenate CF and HD frames, sort by ASMA numeric index, merge `sk_notes` from `ASMA_metadata_SK_20260319.xlsx` on `ASMA_id`, enforce `FINAL_COLS` column order
7. Run QA suite (29 checks) and generate QC report
8. Write final CSV output

---

## 7. QA Validation Suite (29 Checks)

All 29 checks passed on the v3.1 output.

### Identity Integrity
- Row count (expected: 5,019)
- Unique `ASMA_id` count
- Duplicate `ASMA_id` detection (expected: 0)

### Missingness
- Required columns populated: `ASMA_id`, `patient_id`, `sample_id`, `patient_type`, `sampling_method`, `sampling_site`
- `sk_notes` null count (expected: ~4,784 — null for rows with no provenance annotation)
- `isolation_media` null rate for `PL:#` rows (expected null — not an error)

### Mapping Integrity
- Each `ASMA_id` maps to exactly one `patient_id`
- Each `ASMA_id` maps to exactly one `sample_id`
- Duplicate triplet check: (`ASMA_id`, `patient_id`, `sample_id`) duplicates (expected: 0)

### Cohort Rules
- CF rows must not contain `oral_swab`
- HD rows must contain `oral_swab`
- `sputum` rows must have `sampling_site = lower_respiratory_tract`
- `oral_rinse` rows must have `sampling_site = oral_cavity`
- HD site must be `tongue` or `throat`

### Vocabulary Control
- `sampling_method` values restricted to allowed set
- `sampling_site` values restricted to allowed set
- `patient_type` values restricted to allowed set

### HD Validation
- HD `sample_id` format: `^HD\d+_(tongue|throat)$`
- HD `ASMA_id` coverage against Jake's sheet
- HD `patient_id` agreement with Jake's sheet
- HD site agreement with Jake's sheet

### Plate Corrections
- 287 stock plate corrections applied and validated

### Numeric Type Check
- `patient_id` stored as integer (not string)

---

## 8. Version History

| Version | Date | Summary |
|---|---|---|
| v1 | 2026-02 | Initial gold metadata table integrating CF and HD cohorts. |
| v2 | 2026-02 | Updated SK metadata workbook, plate corrections, improved QC checks. |
| v3 | 2026-03-13 | Debugged notebook after regression introduced during rebuild. Surgical repair applied; v2 baseline dependency enforced explicitly. |
| v3.1 / v2.2 | 2026-03-19 | Added `sk_notes` column from `ASMA_metadata_SK_20260319.xlsx`. No new rows. |

---

## 9. Regression Event and Repair (2026-03-13)

During a rebuild of the metadata table, several regressions were detected:

- `sample_id` suffix stripping
- `sampling_method` misclassification
- `sampling_site` misassignment
- Missing `isolation_media` values
- `stock_well` formatting inconsistencies
- `patient_id` exported as string instead of integer

**Root cause:** Several notebook transformations unintentionally overrode validated metadata fields during a partial rebuild.

**Repair strategy:** Rather than rewriting the notebook, only the cells responsible for regressions were modified. Validated metadata fields were restored from the v2 baseline. Assertions were added to detect future regressions. This preserved the notebook's historical structure and readability.

---

## 10. Notes Column Addition (v3.1 — 2026-03-19)

On 2026-03-19, Sun-Young Kim provided `ASMA_metadata_SK_20260319.xlsx`, which adds a `Notes` column to the SK canonical workbook. The file is structurally identical to the 20260303 version — 4,990 rows, all original columns unchanged.

Two note types are applied:

| Note Type | Rows | Affected IDs |
|---|---|---|
| Frozen sputum / diversity sweep | 212 | PRO8, PRO15, PRO22, PRO23, PRO30, PRO76 |
| BSL-3 destruction (CP2PL47) | 23 | ASMA-3482 – ASMA-3504 |

The diversity sweep note resolves a previously open provenance item: samples were collected during an early diversity sweep and were not logged by UCSD at the time of collection.

The BSL-3 destruction note documents that all 23 isolates on plate `20250923_CP:2_PL:47` were destroyed following identification of BSL-3 organisms ASMA-3484 and ASMA-3485. Physical stocks no longer exist.

> **Critical rule:** Any analysis drawing on CP2PL47 isolates (ASMA-3482 – ASMA-3504) must exclude these rows. Physical stocks do not exist.

The column is named `sk_notes` (not `Notes`) to attribute its origin clearly and distinguish it from future notes columns from other contributors.

**Implementation:** The notes merge is embedded in the Section 8 — Assemble Final Metadata Table cell in the build notebook. `SK_NOTES_PATH` is defined in Section 2 — Configuration alongside other input paths.

---

## 11. Known Issues and Caveats

### Expected Data Gaps (Not Errors)

- `isolation_media` is null for all `PL:#`-type CF rows. Media was not recorded for these isolates at the time of isolation. This is documented expected missingness and must not be flagged as an error.
- `sk_notes` is null for 4,784 of 5,019 rows. Only rows with explicit provenance or availability annotations are populated.

### Confirmed Contamination Cases

Two isolates have documented Rothia contamination with re-isolation in progress:

- `ASMA-1449` — stock `20250530_CP:2_PL:20`, well A4
- `ASMA-1614` — stock `20250530_CP:2_PL:21`, well H4

If additional contamination is detected, document using: `ASMA_id`, `stock_plate`, `stock_well`, `notes`, `re-isolation status`.

### Plate Corrections Applied in v2

`ASMA_metadata_SK_20260219` contained incomplete `stock_plate` names for three plates (`CP2PL54`, `CP2PL55`, `CP2PL56`) and a typo on `20251230_CPL2_PL:57`. These are corrected in `ASMA_metadata_SK_20260303.xlsx` and in the v2 build notebook. 287 total stock plate corrections were validated.

### Pandas 3.x Compatibility

`isolation_media` must be initialized as `pd.array(dtype=object)` and APL media assignment must use `.astype(object).values` to avoid Arrow dtype compatibility errors in pandas 3.x.

---

## 12. Reproducibility

**Notebook:** `patient_sputum_asma_isolate_gold_metadata_v2_2.ipynb`

**Required inputs:**

```
patient_sputum_asma_gold_linkage_table_v2.csv     ← v2 baseline (validation anchor)
ASMA_metadata_SK_20260303.xlsx
ASMA_metadata_SK_20260319.xlsx                    ← source of sk_notes column
APL_metadata_20260303.xlsx
healthy donor mapping sheet (Jake)
```

All source files are stored in the Lakehouse under the ASMA metadata directory. Running the notebook end-to-end rebuilds the gold table and regenerates the QC report.

---

## 13. Related Documentation

| Document | Description |
|---|---|
| `asma_gold_linkage_table_reference.md` | Data reference: schema, value glossary, usage examples, known caveats |
| `patient_sputum_asma_gold_linkage_table_v3_1.csv` | The gold output table |
| `patient_sputum_asma_isolate_gold_metadata_v2_2.ipynb` | Build and QA notebook (Lakehouse) |
