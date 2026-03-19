# ASMA Isolate Gold Linkage Table — Data Reference

**Status:** Gold — All 29 QA checks passed
**Version:** v3.1 (2026-03-19)
**Maintainer:** Spencer Long (Arkin Lab)
**File:** `patient_sputum_asma_gold_linkage_table_v3_1.csv`

---

## What This Table Is

The ASMA Isolate Gold Linkage Table is the canonical join layer for the ASMA/PROTECT project. It maps every isolate identifier (`ASMA_id`) to its associated patient and sample identifiers, enabling downstream datasets — taxonomy, WGS, phenotype, clinical metadata — to be joined correctly without ambiguity about cohort membership, sampling method, or anatomical origin.

It covers two cohorts: cystic fibrosis (CF) patients from the Conrad Lab (UCSD) and healthy donors (HD) from the Arkin Lab (Berkeley/LBNL). All 5,019 isolates across both cohorts are represented in a single, analysis-ready table.

---

## Dataset Summary

| Metric | Value |
|---|---|
| Total rows | 5,019 |
| Unique `ASMA_id` values | 5,019 |
| Duplicate `ASMA_id` values | 0 |
| Columns | 10 |

### Cohort Breakdown

| Patient Type | Row Count |
|---|---|
| adult (CF) | 3,789 |
| pediatric (CF) | 470 |
| healthy_donor | 760 |
| **Total** | **5,019** |

### Sampling Method Breakdown

| Sampling Method | Row Count |
|---|---|
| sputum | 3,050 |
| oral_rinse | 1,209 |
| oral_swab (HD only) | 760 |

### Sampling Site Breakdown

| Sampling Site | Row Count |
|---|---|
| lower_respiratory_tract | 3,050 |
| oral_cavity | 1,209 |
| tongue | 389 |
| throat | 371 |

---

## Schema

### Core Identifiers

| Column | Type | Description |
|---|---|---|
| `ASMA_id` | `string` | Primary key. Unique isolate identifier used across all ASMA/PROTECT isolate datasets. Format: `ASMA-{integer}`. |
| `patient_id` | `integer` | Numeric patient identifier. Must be interpreted in the context of `patient_type` — CF and HD patient numbers are assigned independently and do not overlap, but should not be compared directly without the `patient_type` qualifier. |
| `sample_id` | `string` | Sample identifier linking to sample-level metadata. For CF rows, derived from CP1 sputum IDs. For HD rows, constructed as `HD{patient_id}_{sampling_site}` when no CP1-style ID exists. |

### Cohort Classification

| Column | Type | Allowed Values |
|---|---|---|
| `patient_type` | `enum` | `adult` \| `pediatric` \| `healthy_donor` |

### Sampling Metadata

These two columns together define the sampling context. They are kept separate to prevent silent mixing of samples from different anatomical compartments.

| Column | Type | Allowed Values | Notes |
|---|---|---|---|
| `sampling_method` | `enum` | `sputum` \| `oral_rinse` \| `oral_swab` | CF rows use `sputum` or `oral_rinse`. HD rows always use `oral_swab`. |
| `sampling_site` | `enum` | `lower_respiratory_tract` \| `oral_cavity` \| `tongue` \| `throat` | Derived deterministically from `sampling_method` for CF; taken from HD source data for HD rows. |

**Sampling harmonization rules (locked):**

```
sputum      →  lower_respiratory_tract
oral_rinse  →  oral_cavity
oral_swab   →  tongue OR throat  (HD only, site taken from Jake's sheet)
```

### Isolation and Storage Metadata

| Column | Type | Description | Nullability |
|---|---|---|---|
| `isolation_media` | `string` | Growth medium used during initial isolation (e.g., `BHI_broth`, `R2A_agar`). | Null for `PL:#`-type CF rows where media information was not recorded. This is expected and not an error. |
| `stock_plate` | `string` | Freezer stock plate/box identifier. | Populated for all isolates with physical stocks. |
| `stock_well` | `string` | Well position within the stock plate/box. | Populated for all isolates with physical stocks. |
| `sk_notes` | `string` | Provenance and availability notes contributed by Sun-Young Kim. Null for the majority of rows. | Null for 4,784 of 5,019 rows — this is expected. |

---

## Column Value Reference

### `patient_type`

| Value | Cohort | Source |
|---|---|---|
| `adult` | CF adult patients | Sun-Young Kim (Arkin Lab) |
| `pediatric` | CF pediatric patients | Sun-Young Kim (Arkin Lab) |
| `healthy_donor` | Healthy donors | Jake Hilzinger (Arkin Lab) |

### `sampling_method`

| Value | Cohort | Description |
|---|---|---|
| `sputum` | CF | Expectorated sputum sample |
| `oral_rinse` | CF | Oral rinse / mouth wash sample |
| `oral_swab` | HD | Oral swab (tongue or throat) |

### `sampling_site`

| Value | Method | Description |
|---|---|---|
| `lower_respiratory_tract` | `sputum` | Sputum from the lower respiratory tract |
| `oral_cavity` | `oral_rinse` | Rinse capturing the oral cavity broadly |
| `tongue` | `oral_swab` | Tongue swab (HD) |
| `throat` | `oral_swab` | Throat swab (HD) |

### `sk_notes` — Applied Values

`sk_notes` is null for most rows. When populated, it contains one of two note types:

| Note | Affected Rows | Affected Sample IDs |
|---|---|---|
| `"frozen sputum, initial diversity sweep, not recorded by UCSD at time of collection"` | 212 | PRO8, PRO15, PRO22, PRO23, PRO30, PRO76 |
| `"The isolate corresponding to 20250923_CP:2_PL:47 was destroyed following the identification of BSL-3 organisms (ASMA-3484 and ASMA-3485)"` | 23 | All isolates in `20250923_CP:2_PL:47` |

> **Critical rule:** Isolates ASMA-3482 through ASMA-3504 (plate `20250923_CP:2_PL:47`) have been destroyed following BSL-3 organism identification. Physical stocks no longer exist. Any analysis drawing on these isolates must exclude these rows.

---

## How to Use This Table

### As a Join Key

This table is the recommended join layer for all ASMA/PROTECT isolate analyses. Join on `ASMA_id` to attach patient-level or sample-level context to any downstream dataset.

```python
import pandas as pd

gold = pd.read_csv("patient_sputum_asma_gold_linkage_table_v3_1.csv")

# Attach gold metadata to a downstream dataset keyed by ASMA_id
downstream = pd.read_csv("your_downstream_table.csv")
merged = downstream.merge(gold, on="ASMA_id", how="left")
```

### Filtering by Cohort

```python
cf_adults    = gold[gold["patient_type"] == "adult"]
cf_pediatric = gold[gold["patient_type"] == "pediatric"]
hd           = gold[gold["patient_type"] == "healthy_donor"]
```

### Filtering by Sampling Context

```python
# Sputum isolates only (lower respiratory tract, CF)
sputum_only = gold[gold["sampling_method"] == "sputum"]

# Exclude destroyed plate
available = gold[gold["sk_notes"].isna() | ~gold["sk_notes"].str.contains("destroyed", na=False)]
```

### Reading `patient_id` Correctly

`patient_id` is an integer and must be interpreted together with `patient_type`. CF and HD patient numbers are independent namespaces — a CF patient_id of `5` and an HD patient_id of `5` are different patients.

---

## Known Data Gaps and Caveats

**`isolation_media` nulls for `PL:#` rows.** Isolation media was not recorded for CF isolates plated on PL-type plates. This is expected and documented. Do not flag these as errors.

**HD stock loss for HD.P4.** Freezer stocks for HD patient P4 were lost. Genomes exist for these isolates but physical stocks are unavailable.

**Confirmed contamination cases.** Two isolates have documented Rothia contamination and are in re-isolation:
- `ASMA-1449` — stock `20250530_CP:2_PL:20`, well A4
- `ASMA-1614` — stock `20250530_CP:2_PL:21`, well H4

**`APL:#` identifiers are temporary.** `APL:#` plate IDs (visible in upstream source data) are Arkin Lab-assigned temporary identifiers and are not stable primary keys. Resolve to `sample_id` + media type via `APL_metadata_20260303.xlsx` if needed.

---

## Related Documentation

| Document | Description |
|---|---|
| `asma_gold_metadata_documentation_v3_1.md` | Full build documentation: source provenance, design decisions, QA suite, version history, regression event record |
| `ASMA_metadata_SK_20260303.xlsx` | Sun-Young Kim's canonical CF isolate metadata workbook (Lakehouse) |
| `ASMA_metadata_SK_20260319.xlsx` | Updated SK workbook adding the `Notes` column (source of `sk_notes`) |
| `APL_metadata_20260303.xlsx` | APL plate identifier resolution table (Lakehouse) |
| `patient_sputum_asma_isolate_gold_metadata_v2_2.ipynb` | Build notebook — reproduces this table from source inputs |
