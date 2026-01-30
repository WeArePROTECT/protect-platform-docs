# PROTECT Taxonomy Treemap Viewer

## What it is

The PROTECT Taxonomy Treemap Viewer is an interactive treemap visualization tool for exploring the hierarchical taxonomy distribution of PROTECT isolate collections. It provides a visual representation of taxonomic relationships, enabling researchers to understand the composition and diversity of isolate collections through an intuitive, drill-down interface.

## What it shows (and does not show)

- **Shows:** Hierarchical taxonomy distribution (Domain → Phylum → Class → Order → Family → Genus) with proportional area representation
- **Does NOT:**
  - Allow editing or modifications to the dataset
  - Expose raw file contents or clinical data
  - Provide direct data downloads (data accessible via related TSV endpoint)

## Where to access it

- **Treemap Viewer:** https://protect.qb3.berkeley.edu/asma/api/taxonomy/treemap
- **Related Table Viewer:** https://protect.qb3.berkeley.edu/asma/api/taxonomy/table

## Data source and ownership

The canonical taxonomy dataset is **maintained by Alex Styer**. The ASMA backend serves the published dataset through taxonomy endpoints, ensuring the treemap visualization reflects the most current taxonomic classifications available.

## How it works (high level)

- **Backend endpoint** serves taxonomy data for visualization through the ASMA FastAPI platform
- **Treemap rendering** displays hierarchical distribution with proportional area representation and supports drill-down exploration of taxonomic levels

## Integration with ASMA Prototype

The Taxonomy Treemap Viewer is linked from the ASMA Prototype "tools" menu, enabling quick access to taxonomy exploration alongside other PROTECT data visualization tools.

## Example treemap snapshot

A screenshot of the treemap visualization can be added to `images/` if needed for documentation purposes.

## Related documentation

- **[Isolate Table Viewer](../isolate-table-viewer/README.md)** — Interactive table view of the same taxonomy data
- **[ASMA Prototype Platform](../../platforms/asma-prototype/README.md)** — Overview of the ASMA Prototype platform that hosts this viewer

## Current status

The Taxonomy Treemap Viewer is in **initial / iterative** deployment as part of the ASMA platform. The visualization continues to be refined based on user feedback and taxonomy data updates.
