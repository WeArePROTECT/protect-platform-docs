# ASMA Prototype

## What it is

ASMA Prototype is an R&D prototype and vision testbed for exploring what the full ASMA (Airway Synthetic Microbial Atlas) system could become. It combines a FastAPI backend with a React frontend to demonstrate how multi-omic and isolate data can be browsed, linked, and explored interactively. This is not production software; it serves as a training ground for scientists to test and play with integrated tools and concepts. The prototype is used for internal exploration, demos, and collaborative design discussion.

## What it provides (current)

- **Universal Browser** — Browse across Patients → Samples → Bins → Isolates in a lineage view with search and export capabilities
- **Search and export** — Search across entities and export tables to CSV/JSON formats
- **Interaction network exploration** — Force-layout graph visualization of isolates with edge types (competition, co-occurrence, complementarity) with node-click interactions
- **Formulation builder** — Concept demo for drag-and-drop isolate and prebiotic selection with mock formulation score previews
- **Taxonomy viewers** — Interactive DataTables view and Plotly treemap visualization of taxonomic hierarchy
- **Functional-omics dashboard (stubbed)** — Placeholder interface for viewing bin abundance bars and pathway chips, designed for future integration with real functional-omics pipelines

## Built-in tools and links

The ASMA Prototype UI includes links and integrations to:

- **PROTECT File Viewer:** https://protect.qb3.berkeley.edu/protect/
- **GenomeDepot:** https://protect.qb3.berkeley.edu/genomedepot/
- **PROTECT Taxonomic Data Table:** https://protect.qb3.berkeley.edu/asma/api/taxonomy/table
- **PROTECT Taxonomy Treemap:** https://protect.qb3.berkeley.edu/asma/api/taxonomy/treemap

## Data model at a glance

The prototype demonstrates a hierarchical data model linking:
- **Patients** — Top-level entities
- **Samples** — Associated with patients
- **Bins** — Taxonomic bins derived from samples
- **Isolates** — Individual microbial isolates linked to bins

This structure is designed to demonstrate linking and browsing concepts across the data hierarchy, enabling navigation from high-level patient data down to individual isolate details.

## Source repositories and code

- **GitHub (authoritative):** https://github.com/WeArePROTECT/asma-prototype
- **THAR codebase path:** /usr2/people/spencerlong/asma-prototype

## Related documentation

- ASMA Prototype source repository README: https://github.com/WeArePROTECT/asma-prototype/blob/main/README.md
- ASMA Prototype overview documentation: https://github.com/WeArePROTECT/asma-prototype/blob/main/overview.md

## Current status

ASMA Prototype is in **prototype / initial** stage as an R&D testbed. The platform is functional for demonstration and exploration purposes, with core browsing, network visualization, and formulation concept features implemented. The system continues to evolve iteratively as a vision testbed for future ASMA development.
