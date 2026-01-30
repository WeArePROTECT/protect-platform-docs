# PROTECT REDCap Metadata Overview

**Last updated:** January 26, 2026

---

## 1. Purpose of This Document

This README documents the finalized understanding of the **PROTECT clinical and sample metadata architecture** following the introduction of a new REDCap database developed by the Conrad/UCSD team (Dahen Ibarra Munoz).

It is intended to serve as a **single source of truth** for:
- What metadata is expected from the new REDCap database
- What metadata already exists on the PROTECT servers
- How these datasets relate to one another
- What data is intentionally excluded or restricted
- How downstream systems (e.g., ASMA, GenomeDepot, analytics) should interact with these data sources

This document reflects a detailed comparative analysis of:
- The originally planned (but never implemented) REDCap schema (2025)
- The newly implemented REDCap prototype (2026)
- Existing PROTECT server-based metadata
- Clarifications provided via email from the Conrad team

---

## 2. High-Level Architectural Model

The current metadata architecture intentionally separates **clinical metadata** from **sample-handling and experimental metadata**.

- **REDCap** serves as the *clinical abstraction layer*
- **PROTECT server metadata** serves as the *sample handling and wet-lab provenance layer*
- Downstream systems rely on **both**, joined via shared identifiers

This separation is deliberate and should be treated as a stable design decision.

---

## 3. Metadata Available on the PROTECT Server (Non-REDCap)

The following columns are already collected and maintained via PROTECT server-based metadata files. These fields are **not included in REDCap by design** and should continue to be sourced from the server.

### 3.1 Server Metadata Columns

- `sample_id`
- `subject_id`
- `cup_weight`
- `patient_status`
- `collection_method`
- `collection_day`
- `fresh_or_frozen`
- `diagnosis`

### 3.2 Scope and Use

These fields capture:
- Sample collection context
- Wet-lab handling conditions
- Experimental provenance

They are considered the **canonical source** for all sample-handling and laboratory-context metadata and are required for reproducibility, experimental interpretation, and ASMA ingestion.

---

## 4. Metadata Included in the New REDCap Database (Clinical Metadata)

The new REDCap database is scoped to **clinical and EHR-adjacent data only**. It is not intended to store experimental, wet-lab, or sample-processing information.

### 4.1 REDCap Core Identifiers

- `sample_id`
- `subject_id`

These identifiers serve as the **join keys** between REDCap and server metadata.

### 4.2 Clinical and Demographic Fields

REDCap is expected to include the following categories of data:

- Clinical status (e.g., stable vs acute / pulmonary exacerbation context)
- Antibiotic exposure (oral, inhaled, IV; categorized)
- Pulmonary function measurements (FEV1, FVC, % predicted)
- Demographics (sex at birth, race, ethnicity)
- Anthropometrics (height, weight)
- Clinical assessment timing and visit context

### 4.3 Design Notes

- Some clinical fields have been **merged or renamed** relative to earlier planning documents
- REDCap exports may be provided as labels-only or raw variable names depending on configuration
- REDCap should be treated as a **read-only upstream clinical source** for downstream systems

---

## 5. UCSD-Only Data (Not Shared with PROTECT)

Certain clinical or administrative data elements used internally by the UCSD team are:

- Explicitly excluded from REDCap
- Not shared via PROTECT server metadata
- Not available to the UC Berkeley team
- Not intended for downstream analysis or ASMA ingestion

If a field does **not** appear in either:
- a REDCap export, or
- the PROTECT server metadata

it should be assumed to be **UCSD-internal and out of scope** for PROTECT analyses.

---

## 6. Access, Security, and Governance Considerations

### 6.1 REDCap Access

REDCap access may be restricted to specific individuals. Depending on workflow preferences, PROTECT may:
- Receive recurring REDCap exports via the server, and/or
- Grant limited REDCap access to designated data integrators

### 6.2 Server Access

Once REDCap-derived data is placed on the PROTECT server:
- Access controls should be explicitly defined
- Some fields may require restricted handling depending on sensitivity

### 6.3 Public or External Sharing (Future)

Any future public-facing or external use of REDCap-derived data (e.g., via ASMA) should:
- Use aggregated or de-identified representations
- Respect any clinical privacy constraints
- Be explicitly reviewed prior to release

---

## 7. Downstream System Guidance

Downstream systems (ASMA, GenomeDepot, analytics pipelines) should assume:

- **Clinical context** comes from REDCap
- **Sample handling and experimental context** comes from server metadata
- Joins are performed using `sample_id` and `subject_id`

No downstream system should assume REDCap contains complete metadata for experimental interpretation.

---

## 8. Summary

- REDCap is a clinical metadata system, not a full metadata hub
- Server metadata remains essential and authoritative for sample handling
- UCSD-only data is intentionally excluded from PROTECT
- This separation is stable and intentional

Questions or proposed changes to this model should be discussed prior to implementation in downstream systems.

---

**End of document**

