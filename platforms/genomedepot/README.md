# GenomeDepot

## What it is

GenomeDepot is a web-based platform for storing, browsing, and exploring isolate/genome data and annotations for PROTECT. It provides interactive access to microbial genomic collections with comprehensive annotation and comparative exploration capabilities. Within PROTECT, GenomeDepot serves as the primary interface for exploring isolate genomes and their associated annotations.

## What it provides

- **Isolate/genome browsing** — Navigate and explore genome collections through a web interface
- **Search and filtering** — Find genomes and features using search and filter capabilities
- **Annotation summaries** — View comprehensive functional annotations for genomes and genes
- **Genome feature views** — Explore genes, proteins, and genomic regions with detailed information
- **Comparative exploration** — Analyze synteny and genomic neighborhoods across genomes
- **Interactive genome browser** — JBrowse integration for visualizing genome features and annotations
- **BLAST search** — Perform nucleotide and protein BLAST searches across genome collections
- **Sequence download** — Access and download genome sequences and annotations

## Where it is hosted (PROTECT instance)

**Viewer-facing URL:** https://protect.qb3.berkeley.edu/genomedepot/

The PROTECT instance is used by PROTECT scientists for genome exploration and analysis, and provides review/audit visibility into genomic data collections.

## Deployment model (high-level)

GenomeDepot runs on the lab server environment as a Podman-based service. The deployment consists of:
- **Web application container** — Django application served by Gunicorn WSGI server
- **Database container** — MySQL relational database for structured storage of genomes, annotations, and metadata

The containers share persistent volumes for static files, genome data, annotations, and reference databases.

## Annotation and analysis components (high-level)

GenomeDepot integrates multiple annotation and analysis tools:

- **EggNOG-mapper** — Functional annotation (KEGG, GO, EC, TC, CAZy, COG)
- **Diamond** — Fast BLAST searches against genome collections
- **POEM** — Operon prediction using machine learning models
- **AMRFinderPlus** — Antimicrobial resistance gene detection
- **Pfam/TIGRFAM** — Protein domain annotation
- **antiSMASH** — Secondary metabolite biosynthetic gene cluster detection
- **JBrowse** — Interactive genome browser for feature visualization

## Source repositories and code

- **GitHub (authoritative):** https://github.com/WeArePROTECT/GenomeDepot
- **THAR codebase path:** /usr2/people/spencerlong/GenomeDepot

## Related documentation

- GenomeDepot source repository README: https://github.com/WeArePROTECT/GenomeDepot/blob/main/README.md
- PROTECT server environment documentation (see `../server/` in this repository)

## Current status

GenomeDepot is in **initial / v1** deployment for PROTECT. The platform is operational and supports genome import, annotation, and browsing workflows. Annotation pipeline capabilities continue to be refined iteratively.
