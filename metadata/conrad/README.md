# Conrad Metadata

This directory contains documentation for metadata provided and maintained by the Conrad Lab, including:
- design/spec documentation for a Conrad-owned REDCap system (in construction)
- guidance for periodic metadata deliveries ("drops") as new samples/data are added
- clinical metadata handling documentation
- non-sensitive schema snapshots (column headers only) used to document structure evolution over time

---

## Directory Map

metadata/conrad/
├── clinical_docs/
├── monthly_drops/
├── redcap/
└── schema_snapshots/


---

## Contents

### `clinical_docs/`
Clinical metadata handling documentation provided by the Conrad team.

See: `clinical_docs/README.md`

### `monthly_drops/`
Documentation describing how Conrad metadata deliveries occur over time (often event-driven, frequently monthly when new samples are added) and what constitutes the current export structure.

See: `monthly_drops/README.md`

### `redcap/`
Design/spec documentation for a Conrad-owned REDCap database currently in construction. Materials in this folder describe field structure and planned access patterns (design artifacts), not live exports.

See:
- `redcap/README.md`
- `redcap/redcap_file_package_readme.md` (authoritative design package overview)

### `schema_snapshots/`
Non-sensitive "schema snapshots" (column headers only) extracted from Conrad metadata exports. These snapshots are used to document metadata structure evolution without storing sensitive export data.

---

## Sensitivity Guidance

This repository should remain safe to share. Do not commit raw Conrad metadata exports containing sensitive content. Prefer documenting structure using `schema_snapshots/` and the READMEs above.

---

## Updating This Documentation

When new Conrad materials arrive:
1. Place REDCap design/spec updates into `redcap/` and update `redcap/README.md` if needed.
2. Update cadence/structure notes in `monthly_drops/README.md` when delivery patterns or export structure change.
3. Add a new column-header snapshot to `schema_snapshots/` to capture any schema evolution.

Keep updates short, structured, and link-first.
