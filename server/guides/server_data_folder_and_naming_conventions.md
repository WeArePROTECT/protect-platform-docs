# PROTECT / ASMA Data Folder & Naming Conventions Guide

If you have any questions about folder creation, naming conventions, or data uploads, please contact: 📧 spencerlong@berkeley.edu

---

## 1. Goals

- Ensure data is organized, reproducible, and accessible across teams.
- Maintain a consistent folder structure for all labs.
- Use `snake_case` naming for files and folders to avoid confusion and ensure compatibility across systems.
- Clearly separate raw, processed, and metadata files.

---

## 2. Directory Structure

At the top level, each lab has its own folder:

```
/project_root/
    Zengler_Lab/
    Conrad_Lab/
    Nizet_Lab/
    Arkin_Lab/
```

Within each lab folder, keep a standard layout:

```
/lab_name/
    raw_data/        # Raw sequencing or experimental outputs
    processed_data/  # Cleaned, analyzed, or derived data
    metadata/        # Associated metadata files
    README.md        # Short explanation of contents
```

This ensures that anyone entering the directory immediately understands where to look.

---

## 3. File & Folder Naming Conventions

### General Rules

- Always use `snake_case` (e.g., `lung_sample_01.fastq.gz`)
- Never use spaces, special characters, or capital letters.
- Be descriptive but concise — include identifiers like lab, sample type, and date.
- Use standardized suffixes (`_R1`, `_R2` for paired FASTQs, `_v1`, `_v2` for versions).
- Always include a `README.md` in each folder describing the contents.

### Examples

#### Genetic Data (Raw FASTQ)

```
/Zengler_Lab/raw_data/
    patient01_lung_R1.fastq.gz
    patient01_lung_R2.fastq.gz
    patient02_oral_R1.fastq.gz
    patient02_oral_R2.fastq.gz
```

#### Processed Data (Trimmed & Filtered)

```
/Zengler_Lab/processed_data/
    patient01_lung_trimmed_R1.fastq.gz
    patient01_lung_trimmed_R2.fastq.gz
    patient01_lung_host_removed.sam
    patient01_lung_genome_coverage.tsv
```

#### Metadata

```
/Zengler_Lab/metadata/
    patient_metadata.csv
    sample_metadata.csv
    sequencing_protocols.md
```

#### Isolate Genomes

```
/Nizet_Lab/raw_data/
    isolate_asma_001_R1.fastq.gz
    isolate_asma_001_R2.fastq.gz

/Nizet_Lab/processed_data/
    isolate_asma_001_assembly.fasta
    isolate_asma_001_prokka.gff
    isolate_asma_001_amr_report.tsv
```

---

## 4. When to Create a New Folder

- New data type (e.g., adding histology images → create `histology/`).
- Major workflow step (e.g., `raw_data` → `processed_data`).
- Collaborator-specific data (if multiple subgroups within a lab).

Each folder should contain:
- Data files
- A `README.md` describing file formats, protocols, and tools used

---

## 5. Data Scientist's Mindset

When uploading data, ask:

1. Is this raw or processed? → Place accordingly.
2. Does this file name clearly identify sample, type, and version?
3. Can someone else understand this folder without asking me? (if not, add a README).
4. Will this break scripts if parsed programmatically? (stick to `snake_case`, consistent delimiters).

---

✅ **Following these conventions will make our data traceable, reproducible, and analysis-ready, while avoiding messy folders and naming inconsistencies.**
