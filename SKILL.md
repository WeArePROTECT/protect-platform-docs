# SKILL.md — Documentation Context for AI Assistants

This file is machine-readable context for an AI assistant contributing to the `protect-platform-docs` repository. Use this file to produce documents that are consistent with the existing style, structure, placement rules, and conventions of this repo.

---

## 1. Repo Purpose & Audience

**What this repo is:**
`protect-platform-docs` is the authoritative documentation hub for the PROTECT / ASMA data infrastructure. It indexes, organizes, and links documentation across GitHub repos, shared drives, and external protocol repositories. It does not duplicate documentation that lives authoritatively elsewhere — it links to it and provides context.

**Primary audiences:**
- New PROTECT team members onboarding to the platform
- Technical contributors adding data, pipelines, or tools
- Collaborators and external reviewers evaluating the system
- AI assistants generating or editing documentation in this repo

**What PROTECT / ASMA is:**
A multi-team research data infrastructure effort supporting cystic fibrosis research. It collects clinical metadata (Conrad Lab / UCSD) and bacterial isolate data (Arkin Lab / Berkeley/LBNL), organizes it in a centralized server environment, ingests into a KBase Lakehouse (MinIO + Delta Lake + Spark), and provides visibility through platforms (GenomeDepot, ASMA Prototype) and tools (File Browser, Isolate Table Viewer, Taxonomy Treemap Viewer).

**Key teams and roles:**
- Conrad Lab (UCSD): Dahen Ibarra Munoz (CRC), Praveen Akuthota (clinician) — clinical data, REDCap
- Arkin Lab (Berkeley/LBNL): Spencer Long (data infrastructure), Adam Arkin (PI), Sun-Young Kim (wet lab), Jake Hilzinger (staff scientist)

**What belongs here vs. elsewhere:**
- Belongs here: READMEs, operational notes, design/spec docs, non-sensitive schema snapshots, pipeline data references, known issues logs
- Does NOT belong here: sensitive data exports, raw clinical metadata CSVs, secrets or credentials, large binary files (except images used in documentation)

---

## 2. Directory Structure & Conventions

```
protect-platform-docs/
├── README.md                         # Repo root index — links to all sections
├── SKILL.md                          # This file
├── architecture/                     # High-level architecture descriptions and diagrams
│   └── README.md
├── data-infrastructure/              # Data organization, naming, and ingestion systems
│   ├── README.md
│   └── kbase-lakehouse/              # KBase MinIO + Delta Lake + Spark documentation
│       ├── README.md                 # Summary guide (mirrors the full guide structure)
│       ├── KBase-Lakehouse-Ingestion-&-Operations-Guide-PROTECT-V-1-0.md  # Full technical guide
│       ├── configs/
│       │   └── README.md             # Config file naming and storage conventions
│       ├── datasets/
│       │   ├── README.md
│       │   └── genomedepot/
│       │       └── README.md         # Dataset-specific ingestion roadmap
│       └── notebooks/
│           └── README.md             # Notebook patterns and conventions
├── metadata/                         # Metadata schemas, governance, and delivery docs
│   ├── README.md
│   └── conrad/                       # Conrad Lab metadata (the only fully documented source)
│       ├── README.md
│       ├── clinical_docs/            # Clinical data handling decisions (.docx files + README)
│       ├── monthly_drops/            # Monthly export cadence and structure docs
│       ├── redcap/                   # REDCap design/spec docs (non-sensitive)
│       │   ├── README.md
│       │   ├── protect_redcap_metadata_readme.md
│       │   ├── redcap_file_package_readme.md
│       │   ├── REDCap Field Specification Version 7.10.2025.xlsx - Field Specification.csv
│       │   └── redcap_pipeline/      # Pipeline data reference and full technical doc
│       │       ├── README.md         # Pipeline data reference (plain + technical)
│       │       └── PROTECT_redcap_Pipeline_Documentation_v3.md
│       └── schema_snapshots/         # Column-header-only snapshots (non-sensitive)
├── operations/                       # Operational notes, known issues, active work
│   ├── README.md
│   └── conrad/                       # Conrad-specific operational items
│       └── redcap_pipeline_active_issues.md
├── platforms/                        # Platform documentation and source repo links
│   ├── README.md
│   ├── genomedepot/
│   │   └── README.md
│   └── asma-prototype/
│       └── README.md
├── references/                       # Index of external docs (GitHub, Drive, protocols.io)
│   └── README.md
├── server/                           # Server environment, onboarding, usage guides
│   ├── README.md
│   └── guides/
│       ├── README.md
│       ├── niya_mfa_setup.md
│       ├── server_data_folder_and_naming_conventions.md
│       ├── server_data_transfer_guide.md
│       ├── server_etiquette_and_resource_monitoring.md
│       ├── server_filezilla_access.md
│       └── originals/                # Original .docx source files (archived)
└── tools/                            # Visibility and audit tools documentation
    ├── README.md
    ├── how-tools-fit-together.md
    ├── file-browser/
    │   ├── README.md
    │   └── images/
    ├── isolate-table-viewer/
    │   ├── README.md
    │   └── images/
    └── isolate-taxonomy-treemap-viewer/
        ├── README.md
        └── images/
```

**Nesting depth pattern:**
- Top-level folders represent major system domains (metadata, operations, platforms, tools, server, data-infrastructure, architecture, references).
- Each top-level folder has a `README.md` serving as its index.
- Subdirectories are used when a domain has multiple distinct sub-topics or data sources (e.g., `metadata/conrad/redcap/redcap_pipeline/`).
- Maximum observed depth: 5 levels (e.g., `metadata/conrad/redcap/redcap_pipeline/PROTECT_redcap_Pipeline_Documentation_v3.md`).
- `images/` subdirectories are used within tool documentation folders for screenshots.
- `originals/` subdirectories archive source `.docx` files alongside converted Markdown guides.

---

## 3. Naming Conventions

### Folder names
- Use `kebab-case` for all folder names (e.g., `kbase-lakehouse`, `asma-prototype`, `file-browser`, `isolate-table-viewer`, `redcap_pipeline` is an exception — see below).
- Exception: `redcap_pipeline` uses `snake_case`. This is an inconsistency and new folders should use `kebab-case`.
- Lab/team names are `lowercase-kebab` (e.g., `conrad`, not `Conrad_Lab`).
- Exception: The server `originals/` subfolder name uses all lowercase.

### File names (Markdown docs)
- Most Markdown docs use `snake_case` (e.g., `server_data_transfer_guide.md`, `redcap_pipeline_active_issues.md`, `niya_mfa_setup.md`).
- Folder-level index files are always named `README.md` (capitalized, no version suffix).
- Long-form technical reference docs may use `Title-Case-With-Hyphens` for versioned documents (e.g., `KBase-Lakehouse-Ingestion-&-Operations-Guide-PROTECT-V-1-0.md`, `PROTECT_redcap_Pipeline_Documentation_v3.md`). These mixed conventions are an inconsistency.
- Version suffixes on non-README files use lowercase `v` followed by a number (e.g., `_v3`, `-V-1-0`). The `V-1-0` form (with hyphens and capital V) is used in the KBase guide title; `_v3` (with underscore and lowercase v) is used in pipeline docs. Prefer `_v3` style for new files.

### Non-Markdown files
- `.docx` originals in `server/guides/originals/` use `Title_Case_With_Underscores_And_Version` (e.g., `PROTECT_Server_Data_Transfer_Guide_V_1_1.docx`). These are archived originals — do not create new `.docx` files.
- Schema snapshot `.txt` files use `snake_case` with ISO dates: `conrad_protect_metadata_columns_2025-08-27.txt`.
- Images use `snake_case` (e.g., `directory_snapshot.png`, `file_browser_preview.png`).
- One `.csv` file in `metadata/conrad/redcap/` has spaces in the name (`REDCap Field Specification Version 7.10.2025.xlsx - Field Specification.csv`) — this is a historical artifact and should not be replicated.

### Naming patterns to use for new files
- New Markdown guides: `snake_case.md`
- New README files: `README.md`
- New operational logs: `snake_case_active_issues.md` or `snake_case_log.md`
- New versioned technical reference docs: `DESCRIPTOR_v{N}.md` (e.g., `protect_redcap_pipeline_v4.md`)

---

## 4. Document Structure Template

Most substantive documents in this repo follow one of two templates:

### Template A: README / Index Document

Use for: `README.md` files that serve as folder indexes.

```markdown
# [Section Title]

[One paragraph describing the purpose of this section and what is documented here.]

---

## [Subsection or Contents List]

- **[Link to child doc or subfolder]** — one-line description
- **[Link to child doc or subfolder]** — one-line description

---

## [Additional context section, if needed]

[Brief content. Keep short. Use bullet points and links rather than prose.]
```

### Template B: Operational / Technical Document

Use for: pipeline documentation, platform docs, tool docs, operational issue logs.

```markdown
# [Document Title]

**[Metadata block — use bold key: value pairs]**
- Audience, Purpose, Status, Last Updated, Maintainer

---

## Plain Language Summary / What It Is

[2–5 sentences for non-technical readers. Always comes first.]

---

## [Technical Section 1]

[Detail. Use numbered sections for long docs. Use tables, code blocks, bullet lists.]

---

## [Technical Section 2]

...

---

## Related Documentation

[Table or list of related docs with links and one-line descriptions.]
```

### Required sections by document type

| Document type | Required sections |
|---|---|
| Platform README | What it is, What it provides, Where it is hosted, Source repos, Current status |
| Tool README | What it is, What it shows (and does not show), Where to access it, How it works, Source repo |
| Pipeline data reference | Plain language summary, File locations, Output file specs, How pipeline works, Related docs |
| Operations/issues doc | Status table at a glance, Per-issue plain language + technical detail, Resolved items |
| Server guide | Prerequisites/what you need, Numbered step-by-step instructions, Quick reminders |
| Data infrastructure README | Purpose, Contents/directory map, Usage or adding new sources |

---

## 5. Writing Style & Tone Guide

**Tone:** Precise, practical, direct. Not casual, not bureaucratic.

**Voice:** Second person ("you") for procedural guides. Third person for descriptive and reference docs.

**Precision rules:**
- Qualify maturity state with standard language: "initial / v1", "prototype / initial", "iterative", "in progress", "production — validated and running". Never use "finalized" or "complete".
- When something is explicitly deferred, say so: "This is intentionally not addressed in the current phase."
- Use "must", "should", "may" with care. Use "must" for safety-critical rules (e.g., never editing the SQL warehouse by hand).

**What to avoid:**
- Do not use emojis in new technical sections (existing server guides have emoji but this is a legacy style from earlier docs).
- Do not pad with filler sentences ("This document will...").
- Do not use passive voice where active is clearer.
- Do not duplicate content that already lives authoritatively elsewhere — link instead.

**Tables:**
- Use Markdown tables for: vocabulary glossaries, column reference lists, issue trackers, input file descriptions, status summaries.
- Keep table rows short. Move long explanations to sections below the table.

**Code blocks:**
- Use triple-backtick fenced code blocks for all commands, paths, file trees, and code snippets.
- Always specify the language for syntax highlighting (bash, python, json, etc.) when applicable.

**Callouts:**
- Use `> **Critical rule:** ...` for must-not-violate constraints.
- Use `> **Note:** ...` for important clarifications that are not hard rules.

**Length:**
- READMEs: short (under 80 lines where possible). They are indexes, not guides.
- Technical reference docs: as long as needed to be complete and unambiguous.
- Operational issue docs: one plain-language paragraph + one technical paragraph per issue.

---

## 6. Audience & Depth Guidelines

**For new team members (onboarding context):** Write in plain language first. Assume no prior knowledge of the PROTECT system. Explain what a thing is before explaining how it works.

**For technical contributors (engineers, data scientists):** Include exact file paths, command syntax, schema definitions, and code. Be explicit about constraints (e.g., case sensitivity, naming rules, write modes).

**For reviewers and auditors:** Emphasize what the system does and does not expose, what data is sensitive vs. non-sensitive, and current status/maturity level.

**The two-half rule:**
Many documents in this repo (especially operations and pipeline docs) follow a "plain language first, technical detail second" structure within each section. Apply this pattern whenever a topic has both a conceptual component and an implementation component.

---

## 7. Placement Rules (When Adding New Docs)

**architecture/**: Place diagrams and high-level conceptual descriptions of how PROTECT systems relate. Do not place implementation details here.

**data-infrastructure/**: Place docs about how data is physically stored, organized, and governed. Sub-topics (e.g., new storage systems beyond KBase Lakehouse) get their own subdirectory.

**metadata/**: Place metadata schema docs, delivery mechanism docs, and non-sensitive structural snapshots. Each new lab or metadata source gets its own subdirectory (`metadata/<lab_name>/`). Never commit sensitive data exports here.

**operations/**: Place living operational notes, known issue logs, and active work tracking. Organize by system or lab (`operations/<lab_or_system>/`). Files here are updated frequently and are expected to be in flux.

**platforms/**: Place platform-level documentation for data-consuming platforms. Each platform gets its own subdirectory. Focus on: what the platform is, what data it consumes, source repo links, current status. Do not reproduce implementation details — link to source repos.

**references/**: Place indexes of external documentation sources (other GitHub repos, Google Drive folders, protocols.io). Do not place full documents here — only pointers with descriptions.

**server/**: Place all server onboarding, access, and usage guides. Markdown guides go in `server/guides/`. Original `.docx` sources go in `server/guides/originals/`.

**tools/**: Place documentation for visibility and audit tools. Each tool gets its own subdirectory with a `README.md` and optionally an `images/` subfolder for screenshots.

**Root level**: Only `README.md`, `SKILL.md`, `.gitignore`, and workspace config files belong at the root.

---

## 8. Operations Folder Conventions

The `operations/` folder holds two distinct types of living records. Keep them separate.

**Structure pattern:**
```
operations/
├── README.md                          # Brief index of what's in this folder
├── TASKS.md                           # Repo-wide open task list (single source of truth)
└── <system_or_lab>/                   # One subdirectory per system or data source
    └── <topic>_active_issues.md       # Living operational issues for that system
```

---

### TASKS.md — Repo-Wide Task Tracker

`TASKS.md` is the single place to track anything that needs to be done in this repo — doc fixes, missing content, structural cleanup, or follow-up items from other work. It is intentionally lightweight.

**Rules:**
- When a task is done, delete it. Do not archive it. The file should only contain open work.
- Keep it short. If TASKS.md is getting long, that means tasks aren't getting done — not that more rows are needed.
- Tasks are grouped by area (e.g., `## Data Infrastructure`, `## Metadata`, `## Server`).
- Each task is one line with a checkbox, a plain-language description, and an optional link to the affected file.
- Do not track Low severity naming/style nitpicks here unless you plan to fix them soon. Fix them opportunistically when you're already in the file.

**When the AI assistant helps create or edit a doc:**
- It should check the new or edited doc against the conventions in this SKILL.md.
- If it finds something that doesn't conform and can't fix it in context, it should tell you — not silently add it to a log you have to maintain.

---

### System-Specific Active Issues Documents

These live in `operations/<system_or_lab>/` and track operational issues tied to a specific data pipeline or delivery — things like data quality problems, pending decisions from collaborators, or recurring delivery issues that affect downstream analysis. These are different from repo housekeeping tasks.

**When to create one:**
- When a system has recurring deliveries, known data quality issues, or pending decisions that affect downstream analysis.
- Link it from the relevant `metadata/` or `data-infrastructure/` README under "Related Documentation."

**Format:**
- Begin with a metadata block: Last Updated, Maintainer, Related pipeline.
- Summary status table at top: # | Issue | Owner | Blocking analysis? | Status
- Status emojis: ⏳ Waiting, 👀 Monitoring, 📋 Requested, ✅ Resolved
- Each issue: plain language summary paragraph + technical detail paragraph.
- End with a "Resolved Items" section (keep last 3–5 resolved items for context, then drop them).

---

## 9. Known Issues & Gaps (at time of skill generation — March 2026)

These are real issues identified during the initial repo analysis. High and Medium severity items have been migrated to `operations/TASKS.md`. Low severity items (mostly naming inconsistencies in legacy files) should be fixed opportunistically — do not create tracking overhead for them.

**High priority (missing or broken content):**
- `architecture/README.md` — empty stub, no architecture content or diagrams
- `references/README.md` — empty stub, no references listed
- `metadata/conrad/redcap/README.md` — references two `.docx` files that are not present in the repo

**Medium priority (incomplete or duplicated content):**
- `operations/README.md` — one sentence, no navigation guidance
- `data-infrastructure/README.md` — one line, missing data organization patterns and rationale
- `server/README.md` — guide index only, no description of the actual server (hostname, OS, storage, admin)
- `metadata/conrad/clinical_docs/README.md` — lists files but doesn't describe their content
- `metadata/conrad/redcap/redcap_pipeline/README.md` — missing Prerequisites section
- `data-infrastructure/kbase-lakehouse/README.md` — near-verbatim duplicate of sections from the full operations guide, creating dual-maintenance burden
- `KBase-Lakehouse-Ingestion-&-Operations-Guide-PROTECT-V-1-0.md` — Section 10 ("GenomeDepot as the Reference Dataset") appears twice
- `metadata/conrad/redcap/REDCap Field Specification...csv` — filename has spaces and violates naming conventions

**Low priority (legacy style, fix opportunistically):**
- Several older server guides use heavy emoji styling inconsistent with newer docs
- `redcap_pipeline/` folder uses `snake_case` instead of `kebab-case`
- Mixed file naming conventions across older long-form docs
- Directory tree diagrams in some READMEs rendered as plain text instead of fenced code blocks