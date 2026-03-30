# PROTECT Data Integration Pipeline — Active Issues & Ongoing Work

**Last updated:** March 30, 2026
**Maintainer:** Spencer Long (Arkin Lab / LBNL)
**Related pipeline:** PROTECT Data Integration Pipeline v1.0
**Server path:** `/usr2/people/protect/Arkin_Lab/protect_data/protect_data_integration_pipeline/`

This document is a living record of open items, pending decisions, and things being actively watched for this pipeline. It is updated as issues are resolved or new ones arise. It is written for two audiences — a plain-language summary first, followed by technical detail.

---

## Current Status at a Glance

| # | Issue | Owner | Blocking analysis? | Status |
|---|---|---|---|---|
| 1 | Vishant mouthrinse sample registration (PRO*M/PRO*MHD) | Vishant Gandhi | ✅ Yes | ⏳ Waiting |
| 2 | Patient 107 / PRO240 patient_id confirmation | Sun-Young Kim | ✅ Yes | 📋 Requested |
| 3 | PRO76 → PRO76M reassignment (42 isolates, Jira CCS-47) | Sun-Young Kim | ⚠️ Partial | ⏳ Waiting |
| 4 | Race/ethnicity free-text → dropdown conversion (Jira CCS-48) | Dahen Ibarra Munoz | ❌ No | ⏳ Waiting |
| 5 | 5 unregistered tracking sheet samples (PRO35, PRO159M, PRO160M, PRO161M, PRO196M) | Sun-Young Kim | ⚠️ Partial | 📋 Requested |
| 6 | Emma's MIND mapping file not on server (`ko_kegg_uniref_pathway_manuallycurated.tsv`) | Emma Rooholfada | ✅ Yes (blocks MIND re-run) | 📋 Requested |

---

## Issue 1: Vishant Mouthrinse Sample Registration (PRO*M / PRO*MHD)

### Plain language

Vishant Gandhi (UCSD) has collected mouthrinse samples from PROTECT patients that are not yet registered in the ASMA system. These samples are identified by the PRO*M and PRO*MHD naming conventions. Until they are formally registered, the integration pipeline cannot link isolates or multi-omics data from these samples to the correct patient records. This blocks downstream analysis for any isolates derived from unregistered mouthrinse samples.

### Technical detail

Affected sample ID patterns: `PRO*M` (mouth rinse) and `PRO*MHD` (healthy donor mouth rinse). These samples must be entered into the ASMA isolate/sample registration system by Vishant before they will appear in the ASMA linkage table. Once registered, Stage 0 of the pipeline will pick them up automatically on the next run. No pipeline changes are required — registration is the blocking action.

---

## Issue 2: Patient 107 / PRO240 patient_id Confirmation

### Plain language

There is an ambiguity in the mapping between internal PROTECT patient identifiers and sample IDs involving patient 107 and sample PRO240. It is unclear whether PRO240 belongs to patient 107 or to a different patient. Until this is confirmed, any isolates linked to PRO240 may be assigned to the wrong patient in the integrated output, which would corrupt downstream per-patient analysis for those records.

### Technical detail

The conflict surfaces during Stage 0 linkage construction when the ASMA linkage table is joined against the Samples tracking sheet. The `patient_id` field for PRO240 does not cleanly resolve to a single patient. Sun-Young Kim needs to verify her lab records and confirm the correct `patient_id` assignment for PRO240. Once confirmed, the linkage table should be updated accordingly and the pipeline re-run to propagate the correction.

---

## Issue 3: PRO76 → PRO76M Reassignment (42 Isolates)

### Plain language

42 bacterial isolates are currently recorded in ASMA as belonging to PRO76 (a frozen home-collected sample) when they should belong to PRO76M (a fresh mouthrinse sample from the same patient). Vishant Gandhi confirmed the plates were prepared from PRO76M. This is a scientifically meaningful distinction: frozen vs. fresh mouthrinse samples have different collection types and isolation contexts. Until corrected, these 42 isolates are misclassified in all pipeline outputs.

### Technical detail

In the current integrated output:
- 42 isolates are assigned to `pro_sample_id = PRO76`
- PRO76 has `sample_collection_type = Home Fz` and `isolation_source_type = frozen_only`
- Vishant's isolation records confirm plates were from PRO76M (`sample_collection_type = Fs`, fresh mouthrinse)

**What needs to happen:** Sun-Young to review her notes against Vishant's isolation sheet and correct the ASMA linkage table entry. Once the linkage table is updated, re-running Stage 0 will automatically reclassify those 42 isolates from `frozen_only` → `mouth_rinse`.

**Jira:** CCS-47 (Waiting On Others)

---

## Issue 4: Race/Ethnicity Free-Text → Dropdown Conversion

### Plain language

In the current REDCap export, patients' race and ethnicity fields were entered as free-text rather than from a standardized dropdown. The pipeline applies an interim normalization mapping to convert known text values to standard labels, but this is a workaround. Dahen Ibarra Munoz plans to convert these fields in REDCap to use proper dropdown menus, after which the data will arrive in a standardized numeric format automatically. All rows using the workaround are DQ-flagged in the output.

### Technical detail

`race` and `ethnicity` currently arrive as free-text strings rather than numeric codes. The pipeline's interim normalization (`RACE_FREE_TEXT_MAP` / `ETHNICITY_FREE_TEXT_MAP`) maps known values to canonical labels. Unrecognized values surface as `FREE_TEXT_NEEDS_REVIEW: <value>`. When Dahen completes the REDCap conversion, verify on the next delivery that these fields contain numeric codes, then remove the interim normalization block and associated DQ flag from the pipeline.

**Jira:** CCS-48 (Waiting On Others — Dahen)

---

## Issue 5: 5 Unregistered Tracking Sheet Samples

### Plain language

Five sample IDs appear in the pipeline inputs but cannot be found in the PROTECT Samples tracking sheet: PRO35, PRO159M, PRO160M, PRO161M, and PRO196M. Without a tracking sheet entry, the pipeline cannot resolve the `SampleNumber → SampleID` mapping for these samples, leaving them with incomplete linkage records. Isolates from these samples are present in the ASMA system but cannot be fully joined to patient records until the samples are registered.

### Technical detail

These five IDs are flagged during Stage 0 when the pipeline attempts to look up `SampleNumber` in the Protect Samples tracking sheet (Google Sheets source; local export path configured in `run_config`). No match is found for PRO35, PRO159M, PRO160M, PRO161M, PRO196M. Sun-Young Kim needs to add these to the tracking sheet. Once added, re-running Stage 0 will resolve the linkage automatically.

---

## Issue 6: Emma's MIND Mapping File Missing from Server

### Plain language

The MIND (metagenomic and transcriptomic integration) stage of the pipeline requires a curated mapping file (`ko_kegg_uniref_pathway_manuallycurated.tsv`) that Emma Rooholfada (Zengler Lab) maintains. This file is not currently on the server, which means the MIND data integration stage cannot be re-run. Until Emma uploads this file to the expected server path, any re-run of the full pipeline will produce incomplete multi-omics integration output.

### Technical detail

The pipeline's Stage 3 multi-omics integration step requires:
- `ko_kegg_uniref_pathway_manuallycurated.tsv` at the path configured in `run_config` under `mind_mapping_file`

Emma needs to place this file on the PROTECT server at the configured path. Once available, no pipeline code changes are needed — Stage 3 will read it automatically on the next run.

---

## Resolved Items

| Date | Issue | Resolution |
|---|---|---|
| March 25, 2026 | Stage 2 APL join schema mismatch | Sun-Young's new APL file had a different schema (`sampling_site_type` column missing). Resolved by rewriting `_summarise_apl()` to support both legacy and current schema. ✅ Resolved |
