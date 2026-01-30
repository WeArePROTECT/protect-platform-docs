# PROTECT Isolate Table Viewer

## What it is

The PROTECT Isolate Table Viewer is an interactive web-based tool that displays taxonomic and isolate metadata in a searchable, sortable table format. It provides visibility into PROTECT isolate collections with filtering and exploration capabilities, enabling researchers to browse and analyze isolate taxonomy data without requiring direct access to underlying data files.

## What it shows

The viewer displays:

- **Taxonomic classifications** — Hierarchical taxonomy information for each isolate
- **Isolate metadata** — Associated metadata fields for each isolate entry
- **Interactive table features** — Search, sort, and filter capabilities via DataTables
- **Data freshness indicator** — Display of when the underlying data was last updated

This is a read-only viewer for taxonomy and isolate metadata. It does not support editing or modifying the underlying dataset.

## Data source and ownership

The canonical taxonomy dataset is **maintained by Alex Styer**. The data is stored in a tab-separated values (TSV) format and includes taxonomic classifications and isolate metadata for PROTECT collections.

The viewer automatically reflects updates when the source data is regenerated, ensuring users always see the most current information available.

## How it works (high level)

The Isolate Table Viewer is served through the ASMA FastAPI backend:

- **Data serving** — The ASMA backend reads taxonomy data directly from Alex Styer's maintained directory
- **API endpoints** — FastAPI routes serve the table viewer HTML, underlying TSV data, and associated assets
- **Automatic updates** — When the source taxonomy file is updated, the viewer automatically reflects the changes (subject to HTTP cache headers)
- **Path integration** — The viewer is integrated into the ASMA platform under the `/asma/` route prefix

## Related viewers and endpoints

The Isolate Table Viewer is part of a suite of taxonomy visualization tools:

- **[Isolate Treemap Viewer](https://protect.qb3.berkeley.edu/asma/api/taxonomy/treemap)** — Plotly-based treemap visualization of taxonomic hierarchy
- **[Taxonomy Data (TSV)](https://protect.qb3.berkeley.edu/asma/api/taxonomy/tsv)** — Raw TSV data endpoint for programmatic access

## Where to access it

- **Live table viewer:** https://protect.qb3.berkeley.edu/asma/api/taxonomy/table

## Example table snapshot

Below is an example view of the Isolate Table Viewer interface (search, sort, filters, and export options).

![Isolate Table Viewer preview](images/table_preview.png)

## Integration with ASMA Prototype

The Isolate Table Viewer is accessible from the ASMA Prototype platform, providing integrated access to taxonomy data alongside other PROTECT tools and data exploration features.

---

**Note:** This tool is maintained as part of the ASMA platform infrastructure. For questions about the underlying taxonomy data, contact Alex Styer. For technical questions about the viewer interface, refer to the ASMA Prototype documentation.
