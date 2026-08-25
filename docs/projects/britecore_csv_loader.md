# britecore-csv-loader

## Role

Raw CSV loading and relationship-discovery toolkit for BriteCore exports.

## What the project does

Based on the local `README.md` and `CSV_RELATIONSHIPS_OVERVIEW.md`, `britecore-csv-loader` provides:

- `RawCsvInterface` for loading raw datasets by file name
- automatic relationship discovery across `*Id` fields
- revision-centric linking from `revisions.csv` to related exported data
- policy snapshots and linked-entity extraction
- CLI-based inspection and a dedicated claims extractor workflow

## Best-known documentation entry points

- `C:\PythonProjects\data\britecore-csv-loader\README.md`
- `C:\PythonProjects\data\britecore-csv-loader\CSV_RELATIONSHIPS_OVERVIEW.md`
- `C:\PythonProjects\data\britecore-csv-loader\docs\RELATIONSHIP_DIAGRAMS.md`

## What it is strongest at

- loading raw BriteCore CSV exports
- discovering `*Id`-based relationships automatically
- revision-centric and policy-centric snapshot assembly
- CLI-based inspection and claims extraction workflows

## Key local documentation themes

- low-memory mode and DuckDB resource tuning
- raw CSV relationship diagrams and ER-style walkthroughs
- policyholder, revision, property, item, claim, and contact linkage patterns
- claims-extractor workflows for reporting-year outputs

## Recommended use cases

Use `britecore-csv-loader` first when:

- you only have raw exported CSVs and need to reconstruct relationships
- you want to inspect revision-centric linkage outside the logical SQL layer
- you need raw-export parity checks against report-layer views
- you want a lightweight CLI/data-interface for claim or policy snapshot analysis

## Relationship to other projects

- complements `britecore_mcp` by documenting the raw-export layer beneath logical SQL reporting
- complements `PolicyReporting` where raw CSVs are a source interface
- is separate from `britecore_sdk`, which focuses on live API access rather than exported datasets

## Particularly valuable documentation artifact

`CSV_RELATIONSHIPS_OVERVIEW.md` is a high-value source for understanding practical joins and relationship frequency in raw CSV exports.

It is especially useful as a bridge between raw-export datasets and the more curated logical-layer docs in `britecore_mcp`.

