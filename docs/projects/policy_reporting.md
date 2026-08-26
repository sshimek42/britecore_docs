# PolicyReporting

## Role

Cross-source service layer that combines MIPS and BriteCore data under a unified reporting/query API.

## Bridge references

For the ecosystem context, start with:

- [Source-of-Truth Matrix](../source_of_truth_matrix.md)
- [Data Flow and Grain Guide](../data_flow_and_grain_guide.md)
- [Repo Selection Guide](../repo_selection_guide.md)

## What the project is for

`PolicyReporting` is the place to move logic once the problem becomes a shared reporting answer rather than a single-source analysis. It is strongest at:

- fan-out queries across BriteCore and MIPS
- service-level normalization and report-object assembly
- SQL-backed or mixed-source reporting workflows
- future API/UI-facing report services

## Best-known documentation entry points

- `C:\PythonProjects\PolicyReporting\README.md`
- `C:\PythonProjects\PolicyReporting\CLAIMS_REPORTING_GUIDE.md`
- `C:\PythonProjects\PolicyReporting\QUICKSTART.md`
- `C:\PythonProjects\PolicyReporting\INSURANCE_REPORTING_ROADMAP.md`

## Typical use cases

Use `PolicyReporting` when:

- you need one reporting interface across BriteCore and MIPS
- you want either SQL-backed or mixed raw-data reporting
- the question is a shared service-layer answer, not a source-specific data dump

## Relationship to other projects

- sits above raw-source and warehouse layers
- consumes the logical semantics surfaced by `britecore_mcp`
- works with the landed normalization in `policy-data-warehouse`
- complements `britecore-csv-loader` when raw export lineage is still being validated

## Important boundary

This repo is not the canonical home for raw source extraction or the warehouse schema itself. Those remain in the owning repos. `PolicyReporting` is the boundary where the ecosystem becomes a unified reporting product.

