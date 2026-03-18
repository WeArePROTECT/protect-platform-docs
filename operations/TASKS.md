# Repo Task List

**Last Updated:** 2026-03-18
**Rule:** When a task is done, delete it. Do not archive it. This file contains only open work.

---

## Architecture

- [ ] Write actual content for `architecture/README.md` — currently an empty stub. Needs: high-level system diagram, description of how PROTECT components relate (THAR → KBase Lakehouse → GenomeDepot/ASMA). [→ architecture/README.md](../architecture/README.md)

---

## References

- [ ] Populate `references/README.md` with actual external references — currently a single-sentence stub. Needs: links to GitHub repos, Google Drive folders, protocols.io, and one-line descriptions for each. [→ references/README.md](../references/README.md)

---

## Operations

- [ ] Expand `operations/README.md` — currently one sentence. Needs: list of contents, description of the two-tier structure (TASKS.md vs. system-specific active issues docs), and navigation guidance. [→ operations/README.md](README.md)

---

## Data Infrastructure

- [ ] Expand `data-infrastructure/README.md` — currently one line and a single link. Needs: description of data organization patterns, naming conventions, and rationale for centralization. [→ data-infrastructure/README.md](../data-infrastructure/README.md)
- [ ] Resolve duplicate sections in KBase Lakehouse README and full operations guide — `data-infrastructure/kbase-lakehouse/README.md` reproduces large sections of the full guide verbatim. Decide: is the README a true summary (rewrite it short) or is it the canonical doc (deprecate the full guide)? [→ data-infrastructure/kbase-lakehouse/README.md](../data-infrastructure/kbase-lakehouse/README.md)
- [ ] Remove duplicate Section 10 ("GenomeDepot as the Reference Dataset") from `KBase-Lakehouse-Ingestion-&-Operations-Guide-PROTECT-V-1-0.md` — appears twice. [→ full guide](../data-infrastructure/kbase-lakehouse/KBase-Lakehouse-Ingestion-&-Operations-Guide-PROTECT-V-1-0.md)

---

## Metadata

- [ ] Locate or recreate `Protect Redcap Metadata Readme.docx` and `Redcap File Package Readme.docx` — referenced in `metadata/conrad/redcap/README.md` but not present in the repo. Either add the files or remove the references. [→ metadata/conrad/redcap/README.md](../metadata/conrad/redcap/README.md)
- [ ] Expand `metadata/conrad/clinical_docs/README.md` — currently lists `.docx` files but provides no overview of what clinical data handling decisions are documented or how they relate to the PROTECT data model. [→ metadata/conrad/clinical_docs/README.md](../metadata/conrad/clinical_docs/README.md)
- [ ] Add Prerequisites section to `metadata/conrad/redcap/redcap_pipeline/README.md` — currently describes inputs and pipeline steps but does not state what access, credentials, or environment a user needs before running. [→ redcap_pipeline/README.md](../metadata/conrad/redcap/redcap_pipeline/README.md)
- [ ] Rename `metadata/conrad/redcap/REDCap Field Specification Version 7.10.2025.xlsx - Field Specification.csv` — filename has spaces, dots in version string, and mixed casing. Rename to `snake_case` convention (e.g., `redcap_field_specification_v7_10_2025.csv`). [→ file](../metadata/conrad/redcap/)

---

## Server

- [ ] Expand `server/README.md` — currently a guide index only. Needs: server name (THAR), OS, storage capacity, administrator contact, and access model. [→ server/README.md](../server/README.md)
