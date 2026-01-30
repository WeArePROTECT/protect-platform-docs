# How PROTECT Tools Fit Together

The PROTECT platform includes several complementary tools that work together to provide comprehensive visibility and exploration capabilities for PROTECT data. Understanding how these tools relate helps reviewers and scientists navigate the ecosystem effectively.

## Starting with Data Discovery

The **PROTECT File Browser** serves as the entry point for understanding what data exists on the PROTECT server. It provides a read-only view of the directory structure, showing folder organization and filenames without exposing file contents. This tool is particularly useful for audit transparency and for locating where specific datasets or lab contributions are stored.

## Exploring Individual Genomes

Once data locations are identified, **GenomeDepot** enables detailed exploration of individual isolate genomes and their annotations. Researchers can browse genome collections, view comprehensive functional annotations, perform BLAST searches, and access interactive genome browsers. GenomeDepot serves as the primary interface for deep-dive analysis of genomic data.

## Understanding Isolate Collections

For researchers who need to understand the composition and characteristics of isolate collections at a higher level, two complementary taxonomy viewers provide different perspectives:

- The **Isolate Table Viewer** offers a searchable, sortable table interface for exploring isolate taxonomy and metadata. It enables filtering and searching across collections to identify isolates of interest based on taxonomic classifications or metadata fields.

- The **Taxonomy Treemap Viewer** provides a visual overview of isolate composition and distribution through an interactive treemap. The hierarchical visualization helps researchers understand the diversity and relative abundance of different taxonomic groups within collections, with drill-down capabilities from high-level groups down to individual isolates.

## Unified Exploration Interface

The **ASMA Prototype** links these tools together into a unified exploration interface. Through the ASMA Prototype, researchers can access the File Browser, GenomeDepot, and taxonomy viewers from a single platform, enabling seamless movement between different views of PROTECT data.

## A Typical Workflow

A reviewer or scientist might start by using the File Browser to understand what data is available, then move to the Taxonomy Treemap Viewer to get a visual sense of collection composition. From there, they might use the Isolate Table Viewer to search and filter for specific isolates of interest, and finally dive into GenomeDepot to explore detailed genomic annotations for selected isolates.

All of these tools share a common characteristic: they are **read-only and exploratory**. They provide visibility and analysis capabilities without allowing modifications to underlying datasets, ensuring data integrity while enabling comprehensive exploration and review.
