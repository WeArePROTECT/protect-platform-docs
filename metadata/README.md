# Metadata Standards

This directory contains documentation for standardized metadata schemas used across PROTECT, including current and prior versions, update cadence notes, and links to example files where appropriate.

---

## Purpose of Metadata Standardization in PROTECT

Metadata standardization ensures consistent data organization, discoverability, and interoperability across PROTECT platforms and tools. This directory documents the schemas, conventions, and processes that govern how metadata is structured, versioned, and maintained.

---

## Conrad Metadata

The Conrad team provides recurring metadata exports and maintains the REDCap database system.

### [`conrad/`](conrad/)
- **`redcap/`** — REDCap design and specification documentation (Conrad-owned system, in construction)
- **`monthly_drops/`** — Documentation of Conrad monthly metadata export cadence and process
- **`clinical_docs/`** — Clinical data handling decisions and structure documentation
- **`schema_snapshots/`** — Non-sensitive column snapshots documenting export structure evolution

See the README files in each subdirectory for details.

---

## Directory Structure

### [`schemas/`](schemas/)
- **`current/`** — Active schema definitions and field specifications currently in use
- **`archive/`** — Historical schema versions and deprecated specifications

Schema evolution is handled iteratively. As schemas are updated, previous versions are moved to archive for reference.

### [`cadence/`](cadence/)
Documentation of recurring metadata update processes, including:
- Conrad team monthly metadata export cadence (in progress)
- Update workflows and expectations

### [`examples/`](examples/)
- **`conrad/`** — Example metadata files from Conrad team monthly exports
- **`templates/`** — Blank or fill-in templates for metadata creation

### [`references/`](references/)
- External documentation sources
- Metadata inventory and classification records
- Notes for review and open questions
- Links to authoritative documentation

### [`_holding_sensitive/`](_holding_sensitive/)
Files containing potentially sensitive content that require review before committing. These files are intentionally not committed to the repository.

### [`_ambiguous/`](_ambiguous/)
Files whose classification or destination is unclear and require review.

---

## Current Schemas

### REDCap Field Specification
- **Location:** `conrad/redcap/REDCap Field Specification EDITS Version 1.16.2026.xlsx`
- **Status:** Current (January 2026)
- **Purpose:** Defines the REDCap database field specification for clinical metadata
- **Notes:** Includes color-coded annotations indicating field inclusion/exclusion decisions
- **Authoritative Documentation:** See `conrad/redcap/redcap_file_package_readme.md` (ground truth)

### Conrad Monthly Export Structure
- **Location:** `conrad/schema_snapshots/` (non-sensitive column headers only)
- **Status:** Current structure based on September/December 2025 exports
- **Purpose:** Documents the structure of Conrad monthly metadata exports
- **Notes:** Actual export files are sensitive and held in `_holding_sensitive/conrad_monthly_drops/`

### Server Metadata
- **Status:** Under review
- **Purpose:** Documents server-based metadata fields (sample handling, wet-lab provenance)
- **Notes:** See `conrad/redcap/protect_redcap_metadata_readme.md` for architecture overview

---

## Schema Evolution

Schema evolution is iterative and in progress. When schemas are updated:
1. Previous versions are moved to `schemas/archive/`
2. New versions are placed in `schemas/current/`
3. Changes are documented in reference materials

---

## Conrad Update Cadence

The Conrad team provides monthly metadata exports. See `conrad/monthly_drops/README.md` for details on:
- Source location on THAR
- Update cadence (often monthly, event-driven)
- Export structure evolution
- Sensitive content handling

---

## Examples and Templates

- **Conrad Examples:** See `examples/conrad/` for dated metadata exports from the Conrad team
- **Templates:** See `examples/templates/` for blank or fill-in templates

---

## External and Sensitive References

- **External Sources:** See `references/external_sources.md` for high-level documentation of external sources
- **Sensitive Content:** Files in `_holding_sensitive/` are documented at a non-sensitive level in `references/external_sources.md` but are intentionally not committed

---

## Adding New Metadata Documentation

When adding new metadata documentation:

1. **Schemas:** Place current schemas in `schemas/current/`, archive older versions in `schemas/archive/`
2. **Examples:** Add Conrad team examples to `examples/conrad/`, templates to `examples/templates/`
3. **References:** Add external documentation pointers to `references/`
4. **Cadence:** Document update processes in `cadence/`
5. **Sensitive Content:** If unsure about sensitivity, place in `_holding_sensitive/` and document in `references/external_sources.md`

Keep documentation:
- **Short** and **structured**
- **Reviewer-friendly**
- Use "initial / v1 / iterative / in progress" language rather than "finalized"
- Prefer linking to authoritative existing documentation over duplicating it

---

## Open Questions and Review Items

See `references/notes_for_review.md` for:
- Open questions about schema versioning and processes
- Ambiguities requiring clarification
---

**End of README**
