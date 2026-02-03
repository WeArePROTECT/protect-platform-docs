# GenomeDepot Ingestion Roadmap

GenomeDepot serves as the **reference implementation** for PROTECT Lakehouse ingestion.

## Scope Tiers

- **Tier A:** Core relational tables (strains, samples, taxa, genomes)
- **Tier B:** Gene-level annotations (sampled for performance)
- **Tier C:** Extended annotations and metadata (future expansion)

## Required Artifacts

For each ingestion run, the following artifacts must be prepared:

1. **Exports:** TSV files exported from GenomeDepot database
   - `browser_strain.tsv`
   - `browser_sample.tsv`
   - `browser_taxon.tsv`
   - `browser_genome.tsv`
   - `browser_gene_SAMPLED.tsv`
   - `browser_annotation_SAMPLED.tsv`

2. **Config:** `protect-genomedepot-config.json`
   - Located in: `genomedepot-source/config-json/`
   - Defines table schemas and source paths

3. **Notebook:** Canonical ingestion notebook
   - Executed in JupyterHub
   - References the config path

## Validation Checklist

After ingestion, validate:

- [ ] Row counts match expected values from source exports
- [ ] Foreign key relationships (joins) resolve correctly
- [ ] Schema alignment confirmed (no unexpected column drops)
- [ ] Ingestion report reviewed for warnings or errors
- [ ] Sample queries return expected results

---

*This roadmap will be expanded with detailed validation procedures and schema evolution notes as the ingestion process matures.*
