# PROTECT Platform Docs

This repository is the authoritative documentation hub for PROTECT / ASMA data infrastructure, metadata standards, and operational governance. It is intended to be the first place a new PROTECT team member starts, and a clear reference point for collaborators and reviewers.

This repository primarily **indexes, organizes, and links** existing documentation maintained across GitHub, shared drives, and external protocol repositories. Documentation here is **iterative** and will evolve as platform capabilities and standards mature.

---

## What PROTECT / ASMA Is (Context)

PROTECT / ASMA is a data infrastructure effort supporting multi-team research workflows. It brings together data and metadata from multiple contributors and makes them consistently organized, discoverable, and reviewable through a shared set of platforms and tools.

---

## What This Repository Documents

This repository documents:

- **Centralized data organization**  
  How PROTECT data are stored in a centralized lab server environment and structured by lab and contributor.

- **Standardized metadata**  
  The current metadata schemas, their versioned evolution, and how schema-consistent updates are incorporated over time (including recurring monthly updates from the Conrad team).

- **Ingestion rules and governance**  
  Conventions and practices that make uploads repeatable and non-ad-hoc: naming conventions, required metadata, onboarding expectations, and documented submission behaviors.

- **Platforms and visibility tools**  
  The primary platforms and viewers that consume standardized data and metadata (prototype and v1 systems), including audit-friendly visibility utilities.

---

## Repository Map

- [`architecture/`](architecture/) — High-level architecture descriptions and diagrams
- [`data-infrastructure/`](data-infrastructure/) — Data organization patterns, naming conventions, and rationale
- [`metadata/`](metadata/) — Metadata schemas, versioning, update cadence, examples/links
- [`platforms/`](platforms/) — Platform documentation and links to source repositories
- [`tools/`](tools/) — Viewers and audit visibility tools (purpose, audience, data exposed)
- [`server/`](server/) — Central server environment, onboarding, and usage expectations
- [`operations/`](operations/) — Operational notes, known limitations, and in-progress items
- [`references/`](references/) — Index of external documentation sources (GitHub/Drive/protocols)

---

## Platforms and Tools Covered

This repository includes documentation and pointers for:

### Platforms
- **[GenomeDepot](platforms/genomedepot/)** — Web-based platform for storing, browsing, and exploring isolate/genome data and annotations
- **[ASMA Prototype](platforms/asma-prototype/)** — R&D prototype and vision testbed for exploring multi-omic and isolate data browsing, linking, and exploration concepts

### Tools
- **[PROTECT File Browser](tools/file-browser/)** — Read-only directory browser providing audit visibility into server folder structure and filenames
- **[Isolate Table Viewer](tools/isolate-table-viewer/)** — Interactive table viewer for isolate taxonomy and metadata with search and filter capabilities
- **[Taxonomy Treemap Viewer](tools/isolate-taxonomy-treemap-viewer/)** — Interactive treemap visualization for exploring hierarchical taxonomy distribution

Each entry in this repository focuses on **what the platform/tool is**, **what data and metadata it consumes or exposes**, and **where its authoritative implementation docs live**.

### Data Infrastructure
- **[KBase Lakehouse](data-infrastructure/kbase-lakehouse/)** — Ingestion and operations guide for the KBase Lakehouse environment (MinIO object storage, Delta Lake managed storage, Spark SQL querying)

---

## Current State and Iterative Evolution

The current execution environment uses a centralized lab server for data storage and contributor workflows. Standards and documentation are maintained with an iterative mindset: initial conventions and schemas are documented, refined through use, and versioned as the program evolves.

---

## Contribution Model

This repository is intended to remain lightweight and readable:

- Prefer **linking to authoritative existing documentation** over duplicating it.
- When adding new material, keep it **short**, **structured**, and **reviewer-friendly**.
- Use “initial / v1 / iterative / in progress” language rather than “finalized.”

If you are adding or updating documentation, start by placing it in the most appropriate directory above and add a pointer to it from the relevant folder README.
