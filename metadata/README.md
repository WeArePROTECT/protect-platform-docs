# Metadata

This directory documents **metadata structure, standards, and governance**
for data used across PROTECT / ASMA platforms and tooling.

It serves as a centralized index for:
- lab-provided metadata packages
- metadata delivery conventions (e.g., periodic drops, system-based exports)
- metadata system design/specification documentation
- non-sensitive schema documentation used to track structure over time

This directory is intentionally **extensible**. As additional labs and data
sources are onboarded, each will be documented in its own subdirectory using
a consistent pattern.

---

## Directory Structure

metadata/
├── README.md
└── conrad/
├── clinical_docs/
├── monthly_drops/
├── redcap/
├── schema_snapshots/
└── README.md


---

## Current Metadata Sources

### Conrad Lab

The only fully documented metadata source at present is the **Conrad Lab**.
All Conrad-specific documentation is contained within:

metadata/conrad/


See `metadata/conrad/README.md` for details on:
- metadata delivery cadence
- REDCap system design and specifications
- clinical metadata handling documentation
- schema snapshots documenting structure evolution

Future metadata sources should be added as **peer directories** alongside
`conrad/`.

---

## Sensitivity and Publishing Guidance

This repository is intended to remain safe to share.

As a general rule:
- Do **not** commit raw metadata exports containing sensitive content
- Prefer documenting metadata via:
  - READMEs
  - design/specification documents
  - non-sensitive schema snapshots (e.g., column headers only)
- If sensitivity is unclear, treat the file as sensitive and exclude it
  until reviewed

---

## Adding New Metadata Sources

When onboarding a new metadata source or lab:

1. Create a new subdirectory under `metadata/` (e.g., `metadata/<lab_name>/`)
2. Add a `README.md` describing:
   - ownership
   - delivery mechanism
   - cadence
   - sensitivity considerations
3. Document structure using non-sensitive schema snapshots where applicable
4. Keep documentation concise, structured, and link-first
