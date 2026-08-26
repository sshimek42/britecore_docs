# britecore-csv-loader

## Role

Raw CSV loading and relationship-discovery toolkit for BriteCore exports.

## Bridge references

For the ecosystem context, start with:

- [Data Flow and Grain Guide](../data_flow_and_grain_guide.md)
- [Lineage Guide](../lineage_guide.md)
- [Repo Selection Guide](../repo_selection_guide.md)

## What the project is for

`britecore-csv-loader` is the project to use when the work starts from raw extracted CSVs and the main problem is reconstructing the actual relationship model. It is strongest at:

- loading raw CSV datasets by filename
- auto-discovering `*Id` linkage between exported files
- building revision-centric policy snapshots
- validating row-grain and join assumptions before SQL/report logic is finalized

## Best-known documentation entry points

- `C:\PythonProjects\data\britecore-csv-loader\README.md`
- `C:\PythonProjects\data\britecore-csv-loader\CSV_RELATIONSHIPS_OVERVIEW.md`
- `C:\PythonProjects\data\britecore-csv-loader\docs\RELATIONSHIP_DIAGRAMS.md`

## Typical use cases

Use `britecore-csv-loader` when:

- you only have raw export CSVs and need to reconstruct the data model
- you are validating how `revisionId`, `policyId`, `claimId`, and related IDs connect across files
- you need a quick snapshot or raw-export parity check before a warehouse or report layer is designed

## Relationship to other projects

- is the raw-source layer beneath `policy-data-warehouse`
- informs the reporting design that lands in `PolicyReporting`
- is distinct from `britecore_sdk`, which is built for live API access rather than exported datasets

## Important boundary

This project is not the canonical home for the warehouse schema or report logic. It is the canonical home for raw-export structure and lineage reconstruction.

