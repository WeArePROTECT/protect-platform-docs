# PROTECT REDCap Pipeline — Data Reference

**Pipeline version:** v3 (`protect_pipeline_v3.py`)  
**Notebook version:** v4 (`PROTECT_Data_Integration_v4.ipynb`)  
**Last updated:** March 17, 2026  
**Maintainer:** Spencer Long — Berkeley / Arkin Lab  
**Status:** Production — validated and running

---

## What This Is

This document describes two data files produced by the PROTECT REDCap integration pipeline. These files are the primary analytical outputs linking patient clinical data from the Conrad Lab (UCSD) with bacterial isolate data from the Arkin Lab (Berkeley). They are the starting point for any analysis involving PROTECT patient samples.

A plain-language summary is given first, followed by full technical detail for engineers and data scientists.

---

## Plain Language Summary

The PROTECT study collects sputum samples from cystic fibrosis (CF) patients at UC San Diego. At each clinic visit, clinical information about the patient is recorded — things like their lung function, what medications they are on, and how stable their disease is. That data lives in a system called REDCap, owned and managed by the Conrad Lab at UCSD.

On the Berkeley side, bacteria are isolated from those same sputum samples and catalogued in a system called ASMA. Each isolate gets its own unique ID and is stored for genome sequencing and downstream analysis.

The pipeline described here takes those two streams of data and joins them together — so that for any given bacterial isolate, you can also see the full clinical picture of the patient and visit it came from. The pipeline also cleans, decodes, and validates the raw REDCap data as part of this process.

The result is two files:

- **`PROTECT_REDCap_3_2_2026_pipeline_v3.csv`** — one row per patient sample visit, with clinical data decoded into human-readable form and validated
- **`PROTECT_clinical_isolate_linked_3_2_2026_pipeline_v3.csv`** — one row per bacterial isolate, with the isolate's full clinical and sample context attached

The naming convention encodes provenance directly: `3_2_2026` is the date of the raw REDCap export the file was derived from, and `pipeline_v3` identifies the processing logic version.

---

## File Locations (PROTECT Server)

```
Conrad_Lab/metadata/RedCapDataExports/
├── raw/
│   └── PROTECT_RedCapDataExport_3.2.2026 (raw data).csv       ← untouched source
├── pipeline_outputs/
│   ├── 2026-03-17/
│   │   ├── PROTECT_REDCap_3_2_2026_pipeline_v3.csv
│   │   └── PROTECT_clinical_isolate_linked_3_2_2026_pipeline_v3.csv
│   └── latest/                                                  ← always current run
│       ├── PROTECT_REDCap_3_2_2026_pipeline_v3.csv
│       └── PROTECT_clinical_isolate_linked_3_2_2026_pipeline_v3.csv
└── reference/
    ├── PROTECT_DataDictionary_2026-03-06.csv
    └── PROTECT_Samples_-_Sheet1__1_.csv
```

**Naming convention:** `PROTECT_<descriptor>_<raw_data_date>_<pipeline_version>.csv`
- `<raw_data_date>` = date of the REDCap export the file was derived from (not the run date)
- `<pipeline_version>` = version of the processing pipeline used

**`latest/`** always contains a copy of the most recent pipeline run. Point notebooks and downstream scripts here for a stable path that does not need updating between deliveries.

---

## Output File 1: `PROTECT_REDCap_3_2_2026_pipeline_v3.csv`

### What it is

One row per patient sample visit. This is the silver-layer clinical dataset — REDCap data that has been decoded, validated, and enriched with derived columns. It represents the cleanest available view of the clinical metadata as recorded by the Conrad Lab at each patient visit.

**Current shape:** 27 rows × 58 columns  
**Patients:** 9 (as of March 2, 2026 export)  
**Source:** REDCap raw export → pipeline Steps 1–8

### Column groups

| Group | Key columns | Description |
|---|---|---|
| Identity | `record_id`, `subject_id`, `sampleid`, `pro_sample_id`, `visit_block` | Patient and sample identifiers; `pro_sample_id` is the PRO-format ID (e.g. `PRO7`) used across all datasets |
| Quality | `dq_flags`, `multi_sample_flag` | Pipe-separated data quality flags; see DQ Flags section below |
| Demographics | `sex_at_birth`, `race`, `ethnicity`, `lung_function_age_years`, `ht_cm`, `weight_kg`, `bmi` | Patient demographics; `bmi` is pipeline-derived |
| Clinical status | `day_of_assessment`, `assessment_of_pex`, `antibiotic_status`, `cftr_modulator_status` | Visit-level clinical state, fully decoded from REDCap numeric codes |
| Lung function | `fev1_l`, `fev1_pp`, `fvc_l`, `fvc_pp`, `fev1_fvc_ratio` | Spirometry results; `fev1_fvc_ratio` is pipeline-derived |
| Antibiotic summary | `n_active_antibiotics`, `any_iv_antibiotics`, `n_inhaled_cycling_on` | Pipeline-derived antibiotic burden counts |
| Antibiotic detail | `inhaled_tobramycin_tobi`, `azithromycin_oral`, `amikaciniv`, (+ 19 more) | Individual antibiotic fields, decoded |

### Data quality flags (`dq_flags`)

Rows are never dropped. Issues are documented in the `dq_flags` column as pipe-separated strings.

| Flag | What it means |
|---|---|
| `lung_function_restrictive_pattern\|seek_clinical_review` | Patient 545: FEV1% > FVC% — confirmed valid restrictive pattern by clinician |
| `lung_function_spirometry_not_performed` | Patient 534: spirometry not done at this visit (~10% of visits); NaN in lung function fields is expected |
| `multi_sample_visit_metadata_duplicated` | Two sample IDs collected at same visit (e.g. samples 7 and 8); clinical metadata intentionally identical for both rows |
| `race_ethnicity_interim_normalized\|verify_on_next_export` | Race/ethnicity were free-text in this export and normalized by the pipeline's interim map; verify on next delivery whether REDCap dropdown is now live |
| `contains_unknown_code` | A decoded field has a code not in the code map; update CODE_MAPS in pipeline Section 1 |

---

## Output File 2: `PROTECT_clinical_isolate_linked_3_2_2026_pipeline_v3.csv`

### What it is

One row per bacterial isolate. This is the gold-layer analytical dataset — every PROTECT isolate in the ASMA collection joined to its full clinical and sample context. This is the primary file for any analysis connecting bacterial genomics to patient clinical outcomes.

**Current shape:** 4,405 rows × 76 columns  
**Unique patients:** 61  
**Unique samples:** 241  
**Unique isolates:** 4,259 (plus 146 rows for samples with no isolates yet)  
**Source:** PROTECT Samples sheet + REDCap silver layer + ASMA linkage table + APL metadata → pipeline Step 10

### Four data sources joined

| Source | Owner | What it contributes |
|---|---|---|
| PROTECT Samples sheet | Berkeley / Arkin Lab | The anchor — every physical sample ever collected; sample type, routing, collection date |
| REDCap export | Conrad Lab / UCSD | Patient clinical metadata for each visit |
| ASMA linkage table | Berkeley / Arkin Lab (Sun-Young Kim) | The isolate collection — one row per banked bacterial isolate |
| APL metadata | Berkeley / Arkin Lab (Sun-Young Kim) | Whether each isolation event used fresh or frozen sputum — the authoritative fresh/frozen record |

### Coverage flags

| Column | Meaning |
|---|---|
| `has_redcap_metadata` | Clinical data is available for this sample |
| `has_isolates` | Isolates are banked in ASMA for this sample |
| `fully_linked` | Both are present — the analytically gold-standard rows |
| `sent_to_isolation` | Sample type is Fs or Fs-Fz (intended for isolation) |

**Current coverage:**

| Status | Count |
|---|---|
| Fully linked (clinical + isolates) | 643 isolates |
| Clinical data only, no isolates yet | 15 |
| Isolates only, no clinical data yet | 3,616 |
| Samples sheet only (neither yet) | 131 |

### Key columns

| Column | Description |
|---|---|
| `patient_id` | Numeric patient ID — consistent across all four source datasets |
| `pro_sample_id` | PRO-format sample ID (e.g. `PRO7`, `PRO7M`) — join key to ASMA |
| `ASMA_id` | Unique isolate identifier (e.g. `ASMA-1`) |
| `sample_collection_type` | `Fs`=fresh, `Fz`=frozen, `Fs-Fz`=split, `Home Fz`=home-collected frozen |
| `isolation_source_type` | Whether the isolate came from fresh or frozen sputum: `fresh_only`, `frozen_only`, `fresh_and_frozen`, `mouth_rinse`, or `not_in_apl` |
| `apl_has_frozen` | True if any isolation plate for this sample used frozen sputum |
| `assessment_of_pex` | Pulmonary exacerbation status at visit (Stable / Mild / Moderate / Severe / Uncertain) |
| `cftr_modulator_status` | CFTR modulator therapy (Trikafta / Kalydeco / Symdeko / Orkambi / Alyftrek / None / Unknown) |
| `fev1_pp` | FEV1 percent predicted |
| `antibiotic_status` | Antibiotic treatment context (Chronic / Acute / None) |
| `dq_flags` | Pipe-separated data quality flags inherited from REDCap silver layer |

### `isolation_source_type` breakdown (March 2026)

| Value | Count | Meaning |
|---|---|---|
| `fresh_only` | 2,476 | Isolated from fresh sputum only |
| `mouth_rinse` | 1,210 | Derived from mouth rinse samples |
| `fresh_and_frozen` | 362 | PRO ID has both fresh and frozen isolation plates |
| `frozen_only` | 212 | Isolated exclusively from frozen sputum |
| `not_in_apl` | 145 | Sample not yet processed through APL |

> **Note on frozen sputum:** Fz and Home Fz samples can legitimately have ASMA isolates. Frozen sputum was deliberately plated for isolation on multiple occasions — for taxonomic diversity and PA-targeted isolation. This is valid biology confirmed by the Berkeley wet lab team (March 2026). `isolation_source_type` is the correct column to use when filtering by isolation origin; the `sample_collection_type` column reflects only the physical format of the collected sample.

---

## How the Pipeline Works

### Inputs required

| File | Description | Owner |
|---|---|---|
| `PROTECT_RedCapDataExport_<date>.csv` | Raw REDCap export — wide format, one row per patient | Dahen Ibarra Munoz, UCSD |
| `patient_sputum_asma_gold_linkage_table_v3.csv` | ASMA isolate records — one row per isolate | Sun-Young Kim, Berkeley |
| `PROTECT_Samples_-_Sheet1__1_.csv` | Sample bridge sheet — maps sample numbers to PRO IDs | Berkeley / Arkin Lab |
| `APL_metadata_20260303.xlsx` | Fresh vs frozen isolation origin per sample | Sun-Young Kim, Berkeley |

### Processing steps

1. **Load** all four input files; summarise APL metadata from plate-level to sample-level
2. **Reshape** REDCap from wide format (1 row/patient, up to 693 columns) to long format (1 row/sample visit)
3. **Filter** to rows with actual sample data; split multi-sample visit IDs into separate rows
4. **Decode** REDCap numeric codes into human-readable labels using confirmed code maps from the REDCap data dictionary
5. **Normalize** race/ethnicity free-text (interim — pending REDCap dropdown conversion by UCSD)
6. **Derive** calculated columns: BMI, FEV1/FVC ratio, antibiotic counts
7. **Flag** data quality issues into `dq_flags` column
8. **Save** silver-layer output: `PROTECT_REDCap_<date>_pipeline_v3.csv`
9. **Validate** crosswalk — confirms 0 patient ID mismatches between REDCap and ASMA
10. **Build** gold-layer output: join Samples + REDCap + ASMA + APL; add `isolation_source_type`; save `PROTECT_clinical_isolate_linked_<date>_pipeline_v3.csv`

### Running the pipeline

```bash
# Command line
python protect_pipeline_v3.py

# Or open the notebook and update Section 0, then run all cells:
# PROTECT_Data_Integration_v4.ipynb
# Update REDCAP_RAW_PATH to the new export file
# If new APL_metadata received, update APL_METADATA_PATH too
```

### ID mapping (confirmed)

```
REDCap sampleid N  ==  Samples SampleNumber N  ==  ASMA sample_id PRO{N}
REDCap subject_id  ==  Samples PatientNumber   ==  ASMA patient_id
```

Validated computationally on every pipeline run with a crosswalk check. Any mismatch will surface as an error in the Section 9 summary report.

---

## Delivery Cadence

The Conrad Lab (Dahen Ibarra Munoz) delivers REDCap exports monthly to the PROTECT server. Data is entered patient-by-patient, so each delivery contains complete data for a subset of patients rather than the next N samples in numerical order. Roughly 15–18 new samples per delivery. Full dataset (~280 samples) expected by end of May 2026.

When a new export arrives: update `REDCAP_RAW_PATH` in Section 0 of the notebook → run all cells → review Section 9 summary report → archive previous outputs → copy new outputs to `latest/`.

---

## Related Documentation

| Document | Location | Description |
|---|---|---|
| `PROTECT_Pipeline_Documentation_v3.md` | This folder | Full pipeline technical reference — input schemas, code maps, DQ flags, open items |
| `redcap_pipeline_active_issues.md` | `operations/conrad/` | Running log of open items and current status |
| `PROTECT_Dahen_Call_Questions_v6.docx` | PROTECT server | Q&A log with Conrad team covering all confirmed data decisions |
| `PROTECT_DataDictionary_2026-03-06.csv` | PROTECT server / `reference/` | Authoritative REDCap field definitions — do not modify |
