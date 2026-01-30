# Metadata

This directory documents PROTECT metadata standards and lab-provided metadata packages used across PROTECT / ASMA platforms and tooling.

The current focus is **Conrad Lab metadata**, including:
- recurring metadata drop conventions (structure + cadence)
- REDCap design/specification documentation (future-facing; design artifacts)
- clinical metadata handling documentation
- non-sensitive schema snapshots (column headers only)

---

## Current Structure

metadata/
├── README.md
└── conrad/
├── clinical_docs/
├── monthly_drops/
├── redcap/
└── schema_snapshots/


---

## Conrad Metadata
### `conrad/clinical_docs/`
Clinical metadata handling documentation provided by the Conrad team.  
This folder captures the most recent guidance available and may include multiple versions of decision documents.

### `conrad/monthly_drops/`
Documentation describing how Conrad metadata is delivered over time (often event-driven, frequently monthly when new samples are added), and what constitutes the "current" export structure.

### `conrad/redcap/`
REDCap documentation for a **Conrad-owned REDCap system currently in construction**.  
Files here are design/spec artifacts describing field structure and planned access patterns (not live exports).

### `conrad/schema_snapshots/`
Non-sensitive schema snapshots (column headers only) extracted from Conrad metadata exports to document structure evolution over time without storing sensitive content.

---

## Sensitivity and Publishing Guidance

This repository is intended to remain safe to share. As a rule:
- Do **not** commit raw metadata exports containing sensitive content.
- Prefer documenting structure via **schema snapshots** (column headers) and written READMEs.
- If a file's sensitivity is unclear, treat it as sensitive by default and exclude it until reviewed.

---

## How to Add New Metadata Documentation

1. Add new **REDCap design/spec** materials to `conrad/redcap/` and update `conrad/redcap/README.md`.
2. When a new metadata drop arrives, update `conrad/monthly_drops/README.md` if cadence or structure changes.
3. Add/refresh a new **schema snapshot** (column headers only) in `conrad/schema_snapshots/` for structure tracking.
4. Keep documentation short, structured, and evergreen.
