# PROTECT Data Integration Pipeline — Active Issues & Ongoing Work

**Last updated:** June 11, 2026
**Maintainer:** Spencer Long (Arkin Lab / LBNL)
**Related pipeline:** PROTECT Data Integration Pipeline v1.1 (Stage 4 added 2026-05-06)
**Server path:** `/usr2/people/protect/Arkin_Lab/protect_data/protect_data_integration_pipeline/`

This document is a living record of open items, pending decisions, and things being actively watched for this pipeline. It is updated as issues are resolved or new ones arise. It is written for two audiences — a plain-language summary first, followed by technical detail.

---

## Current Status at a Glance

| # | Issue | Owner | Blocking analysis? | Status |
|---|---|---|---|---|
| 1 | Vishant mouthrinse sample registration (PRO*M/PRO*MHD) | Vishant Gandhi | ✅ Yes | ⏳ Waiting |
| 2 | Patient 107 / PRO240 patient_id confirmation | Sun-Young Kim | ✅ Yes | 📋 Requested |
| 3 | PRO76 → PRO76M reassignment (42 isolates, Jira CCS-47) | Sun-Young Kim | ⚠️ Partial | ⏳ Waiting |
| 4 | 5 unregistered tracking sheet samples (PRO35, PRO159M, PRO160M, PRO161M, PRO196M) | Sun-Young Kim | ⚠️ Partial | 📋 Requested |
| 5 | Emma's MIND mapping file not on server (`ko_kegg_uniref_pathway_manuallycurated.tsv`) | Emma Rooholfada | ✅ Yes (blocks MIND re-run) | 📋 Requested |
| 6 | Zengler MIND/omics analysis is sputum-only (sample-ID suffixes stripped by convention) | Emma Rooholfada | No | ✅ Resolved |

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

## Issue 4: 5 Unregistered Tracking Sheet Samples

### Plain language

Five sample IDs appear in the pipeline inputs but cannot be found in the PROTECT Samples tracking sheet: PRO35, PRO159M, PRO160M, PRO161M, and PRO196M. Without a tracking sheet entry, the pipeline cannot resolve the `SampleNumber → SampleID` mapping for these samples, leaving them with incomplete linkage records. Isolates from these samples are present in the ASMA system but cannot be fully joined to patient records until the samples are registered.

### Technical detail

These five IDs are flagged during Stage 0 when the pipeline attempts to look up `SampleNumber` in the Protect Samples tracking sheet (Google Sheets source; local export path configured in `run_config`). No match is found for PRO35, PRO159M, PRO160M, PRO161M, PRO196M. Sun-Young Kim needs to add these to the tracking sheet. Once added, re-running Stage 0 will resolve the linkage automatically.

---

## Issue 5: Emma's MIND Mapping File Missing from Server

### Plain language

The MIND (metagenomic and transcriptomic integration) stage of the pipeline requires a curated mapping file (`ko_kegg_uniref_pathway_manuallycurated.tsv`) that Emma Rooholfada (Zengler Lab) maintains. This file is not currently on the server, which means the MIND data integration stage cannot be re-run. Until Emma uploads this file to the expected server path, any re-run of the full pipeline will produce incomplete multi-omics integration output.

### Technical detail

The pipeline's Stage 3 multi-omics integration step requires:
- `ko_kegg_uniref_pathway_manuallycurated.tsv` at the path configured in `run_config` under `mind_mapping_file`

Emma needs to place this file on the PROTECT server at the configured path. Once available, no pipeline code changes are needed — Stage 3 will read it automatically on the next run.

---

## Issue 6: Zengler MIND / Omics Analysis Uses Sputum Samples Only

### Plain language

The Zengler Lab's multi-omics analysis (metaG / metaRS / MIND), maintained by Emma Rooholfada, works with sputum samples only — it does not include mouth-rinse samples. Because of this, Emma intentionally drops the sample-ID suffixes (`M` = mouth rinse, `P` = pediatric, `MP` = mouth rinse + pediatric) in her metadata, so her sample IDs are bare numbers (e.g. `PRO111` rather than `PRO111P`). This is by design, not an error. Anyone joining her omics metadata to the integrated tables should read a bare omics `SampleID` as the sputum sample for that number and should not expect mouth-rinse omics. Confirmed by Emma on 2026-06-11.

### Technical detail

Emma's MIND metadata (`/usr2/people/protect/Zengler_Lab/Emma/analysis3/WoL2_Subset50_metaG_metadata.tsv` and the `metaRS` equivalent, which feed the `analysis_cluster*/MIND[_ratio]/` runs) carries bare-numeric `SampleID`s with suffixes stripped. There is no collision risk inside her dataset: although ~12 sample numbers exist as both a sputum (`PRO###`) and a mouth-rinse (`PRO###M`) for the same patient, only the sputum member is in her omics set, so the bare ID is unambiguous there. Separately, two pediatric sputum samples appeared bare with missing subject IDs — `PRO227` → `PRO227P` (subject 1009) and `PRO233` → `PRO233P` (subject 1000) — and are being re-attributed from the current REDCap export (Emma had built her metadata from an older export). No pipeline change is required. Remaining action: record the sputum-only scope in the omics analysis procedure so the bare-ID convention is explicit for downstream users.

---

## Resolved Items

| Date | Issue | Resolution |
|---|---|---|
| March 25, 2026 | Stage 2 APL join schema mismatch | Sun-Young's new APL file had a different schema (`sampling_site_type` column missing). Resolved by rewriting `_summarise_apl()` to support both legacy and current schema. ✅ Resolved |
| April 12, 2026 | Race/ethnicity free-text → dropdown conversion (Jira CCS-48) | Dahen completed the REDCap conversion. Race and ethnicity now arrive as numeric codes and decode through `CODE_MAPS` correctly. CCS-48 closed. ✅ Resolved |
| May 1, 2026 | Race/ethnicity Stage 1 cleanup (Jira CCS-57) | Removed the inert `no_unrecognized_race_ethnicity` Stage 1 QA check (PR 2). No DQ flags remain related to free-text race/ethnicity. ✅ Resolved |
| May 6, 2026 | Stage 4 Conrad sample metadata integration (PR 3) | New leaf stage `stages/stage4_conrad_integration.py` ingests Dahen's Conrad sample-metadata Excel file (Sample Data + Micro Data sheets). Implements 14 QA checks including the cross-source consistency check `subject_id_consistent_with_redcap` (162 shared samples, 0 disagreements on the 4/30/26 file). Three new outputs (cleaned Sample Data, cleaned Micro Data, Stage-3-merged Conrad integration). Pipeline bumped to v1.1; intake protocols §9 promoted from "(proposed)" to operational. ✅ Resolved |
