# PROTECT REDCap Pipeline — Active Issues & Ongoing Work

**Last updated:** March 17, 2026  
**Maintainer:** Spencer Long — Berkeley / Arkin Lab  
**Related pipeline:** `protect_pipeline_v3.py` / `PROTECT_Data_Integration_v4.ipynb`  
**Jira project:** CCS (Berkeley Tasks)

This document is a living record of open items, pending decisions, and things being actively watched for this dataset. It is updated as issues are resolved or new ones arise. It is written for two audiences — a plain-language summary first, followed by technical detail.

---

## Current Status at a Glance

| # | Issue | Owner | Blocking analysis? | Status |
|---|---|---|---|---|
| 1 | PRO76 vs PRO76M — isolate reassignment needed | Sun-Young Kim | Partial — affects 42 isolates | ⏳ Waiting |
| 2 | Race/ethnicity dropdowns — verify on next REDCap export | Spencer Long | No — rows are flagged | ⏳ Next delivery |
| 3 | `cftr_modulator_status` missing for patients 534 and 538 | Dahen (UCSD) | No — but gaps clinical picture | ⏳ Next delivery |
| 4 | REDCap direct export access for Spencer | Dahen (UCSD) | No — workaround in place | ⏳ Waiting |
| 5 | Vishant's remaining frozen plates — not yet in ASMA | Vishant Gandhi / Sun-Young Kim | No — pipeline handles it | 👀 Monitoring |
| 6 | Sun-Young formal metadata for ad hoc frozen isolations | Sun-Young Kim | No | 📋 Requested |

---

## Issue 1: PRO76 vs PRO76M — Isolate Reassignment

### Plain language

The PROTECT study tracks samples by ID. Sample PRO76 is a frozen home-collected sample; PRO76M is a fresh mouth rinse sample from the same patient. These are two distinct physical samples.

Vishant Gandhi (UCSD) confirmed that the bacterial plates he prepared for this patient came from PRO76M — the fresh mouth rinse — not from PRO76 (the frozen sample). However, Berkeley's ASMA database currently records those 42 bacterial isolates as belonging to PRO76. This means 42 isolates are currently linked to the wrong sample. Until this is corrected, those isolates appear to come from a frozen sample when they actually came from a fresh mouth rinse — a scientifically meaningful distinction.

### Technical detail

In `PROTECT_clinical_isolate_linked_3_2_2026_pipeline_v3.csv`:
- 42 isolates are assigned to `pro_sample_id = PRO76`
- PRO76 has `sample_collection_type = Home Fz` and `isolation_source_type = frozen_only`
- Vishant's isolation records show plates were from PRO76M (`sample_collection_type = Fs`, a fresh mouth rinse)
- Sun-Young's lab notes recorded these as PRO76

**What needs to happen:** Sun-Young to review her notes against Vishant's isolation sheet and correct the ASMA linkage table entry if the plates are confirmed as PRO76M. Once the linkage table is updated, re-running the pipeline will automatically reclassify those 42 isolates from `frozen_only` → `mouth_rinse`.

**Jira:** CCS-47 (Waiting On Others)

---

## Issue 2: Race/Ethnicity Fields — Pending REDCap Dropdown Conversion

### Plain language

In the current dataset, patients' race and ethnicity were entered as free-text rather than from a standardized dropdown menu. The pipeline has a temporary fix that converts known text entries (e.g. "white" → "White") to standard values, but this is a workaround. Dahen (UCSD) plans to convert these fields in REDCap to use a proper dropdown menu, after which the data will arrive in a standardized numeric format automatically.

All rows affected by the workaround are flagged in the data so they are easy to identify.

### Technical detail

As of the March 2, 2026 export, `race` and `ethnicity` arrive as free-text strings rather than numeric codes. The pipeline's interim normalization (`RACE_FREE_TEXT_MAP` / `ETHNICITY_FREE_TEXT_MAP` in Section 1 of the notebook) maps known values to canonical labels. Unrecognized values surface as `FREE_TEXT_NEEDS_REVIEW: <value>`. 22 of 27 rows in the current output carry the `race_ethnicity_interim_normalized|verify_on_next_export` DQ flag.

**What needs to happen on next delivery:**
1. Open the new REDCap CSV and check a few rows — do `race` and `ethnicity` now contain numbers (1, 2, 3...) instead of text like `'white'`?
2. If yes: remove `RACE_FREE_TEXT_MAP`, `ETHNICITY_FREE_TEXT_MAP`, the interim normalization block (Step 3.5), and the associated DQ flag from the notebook. The fields will decode automatically through the existing code maps.
3. If still free-text: check for any new unrecognized values and add them to the maps before re-running.

**Jira:** CCS-48 (To Do — triggered on next delivery)

---

## Issue 3: `cftr_modulator_status` Missing for Patients 534 and 538

### Plain language

CFTR modulators are a critical class of medications for cystic fibrosis patients. For two patients (534 and 538), this field is completely blank across all of their clinic visits in the current dataset — 10 visits total. This is not expected: even if a patient is not on a modulator, the field should say "None." This appears to be a data entry gap in REDCap that Dahen needs to fill in.

This does not block analysis but leaves an incomplete clinical picture for two patients who together have over 200 bacterial isolates in the collection.

### Technical detail

`cftr_modulator_status` is NaN for all visits of patients 534 (5 visits: samples 6, 24, 73, 78, 79) and 538 (5 visits: samples 10, 31, 56, 60, 61). Every other clinical field for these patients is populated normally. The field uses codes 1–7 (Trikafta / Kalydeco / Symdeko / Orkambi / Alyftrek / None / Unknown). No pipeline changes are needed — once Dahen enters the correct values in REDCap and delivers the next export, the pipeline will decode and populate the field automatically.

**Jira:** CCS-51 (To Do — follow up with Dahen on next delivery)

---

## Issue 4: REDCap Direct Export Access for Spencer

### Plain language

Currently, Spencer receives REDCap data as a CSV file that Dahen manually exports and uploads to the shared server once a month. Dahen offered to look into giving Spencer direct read access to the REDCap database, which would allow Spencer to pull fresh data at any time without waiting. This would be particularly useful given the pace of new data coming in through May 2026. This is pending Dahen confirming the access method.

### Technical detail

Default delivery is monthly CSV exports to the PROTECT server. If direct access is provisioned (web UI export or API token), update `REDCAP_RAW_PATH` in Section 0 of the notebook to the new file location. No other pipeline changes required.

**Jira:** CCS-49 (Waiting On Others — Dahen)

---

## Issue 5: Vishant's Remaining Frozen Plates — Not Yet in ASMA

### Plain language

Vishant Gandhi (UCSD) plated frozen sputum samples from 24 patients for PA-targeted bacterial isolation and shipped them to Berkeley. Of these, only 3 have bacterial isolates recorded in the ASMA system so far. The other 21 patient samples were plated but the isolates have not yet been added to the collection. When they are, they will appear in the ASMA linkage table and will flow into the linked dataset automatically on the next pipeline run. No action needed — just monitoring.

### Technical detail

Vishant's confirmed frozen plate list: PRO8, 9, 16, 19, 21, 22, 23, 24, 33, 42, 60, 73, 78, 81, 82, 90, 101, 106, 123, 124, 125, 127, 131, 139.

Currently in ASMA: PRO8 (26 isolates), PRO22 (40), PRO23 (34).  
Not yet in ASMA: the remaining 21 PRO IDs.

The pipeline will classify these correctly as `frozen_only` via APL_metadata when they appear — no anomaly flag will trigger. Note: Vishant confirmed PRO76 plates were from PRO76M, not PRO76 (see Issue 1).

**No action required.** Monitoring for appearance in future linkage table updates.

---

## Issue 6: Sun-Young Formal Metadata for Ad Hoc Frozen Isolations

### Plain language

The six original samples that first prompted the investigation (PRO8, PRO15, PRO22, PRO23, PRO30, PRO76) were isolated from frozen sputum on an ad hoc basis during the early phase of the PROTECT study to capture greater bacterial diversity. UCSD was not notified at the time, so these events were never recorded in REDCap or the Samples sheet. Sun-Young noted she intends to create formal metadata entries documenting this context. This will make the record complete for future reference and auditing.

### Technical detail

These six PRO IDs currently have `isolation_source_type = frozen_only` in the linked output, which is correct based on APL_metadata. Sun-Young creating formal metadata entries will not change the pipeline outputs — the `isolation_source_type` classification is already correct. The metadata documentation is for completeness of the lab record.

**No pipeline action needed.**

---

## Next REDCap Delivery — Watch List

The next delivery is expected the week of **March 30 – April 3, 2026** (first ~80 samples). When it arrives, check the following before and after running the pipeline:

**Before running:**
- [ ] Is `race`/`ethnicity` now numeric? → Retire interim normalization maps if yes (Issue 2)
- [ ] Update `REDCAP_RAW_PATH` in Section 0 of the notebook
- [ ] Check if Sun-Young has released a new `APL_metadata` file → Update `APL_METADATA_PATH` if so

**After running — Section 9 summary report should show:**
- [ ] Crosswalk validation: 0 patient ID mismatches
- [ ] No `UNKNOWN_CODE_X` values in DQ flags
- [ ] Patient and row counts increased as expected
- [ ] `cftr_modulator_status` NaN count for patients 534/538 decreased (Issue 3)
- [ ] PRO76 reclassified from `frozen_only` → `mouth_rinse` if Sun-Young corrected ASMA linkage table (Issue 1)

---

## Resolved Items

| Date | Issue | Resolution |
|---|---|---|
| March 17, 2026 | Fz/Home Fz samples with ASMA isolates — flagged as anomaly | Confirmed valid biology: frozen sputum was deliberately plated for isolation on multiple occasions. False anomaly flag removed from pipeline. APL_metadata added as 4th source; `isolation_source_type` column added to linked output. |
| March 17, 2026 | All REDCap code maps unconfirmed | All 33 coded fields confirmed against `PROTECT_DataDictionary_2026-03-06.csv`. Zero discrepancies. |
| March 17, 2026 | REDCap `_vN` visit block suffix bug | Early pipeline versions searched for `_2`, `_3` suffixes instead of `_v2`, `_v3`. Fixed — all 15 visit blocks now correctly parsed. |
