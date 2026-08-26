# policy-data-warehouse

## Role

Warehouse and reporting layer for landed BriteCore and MIPS data.

## Bridge references

For the ecosystem context, start with:

- [Source-of-Truth Matrix](../source_of_truth_matrix.md)
- [Data Flow and Grain Guide](../data_flow_and_grain_guide.md)
- [BriteCore Status and Cancellation Semantics](../britecore_status_and_cancellation_semantics.md)

## What the project is for

`policy-data-warehouse` is the canonical place for:

- raw landing into `raw.*` tables
- core normalization into reportable tables
- SQL-backed report view construction
- preserving source semantics while creating a stable reporting layer

## Best-known documentation entry points

- `C:\PythonProjects\data\policy-data-warehouse\README.md`
- `C:\PythonProjects\data\policy-data-warehouse\data\docs\BRITECORE_DATA_STRUCTURE.md`
- `C:\PythonProjects\data\policy-data-warehouse\docs\project_notes\WORKFLOW_GUIDE.md`
- `C:\PythonProjects\data\policy-data-warehouse\scripts\README_dev_reset_then_load.md`

## Typical use cases

Use `policy-data-warehouse` when:

- the task requires a landed and queryable warehouse model
- raw values must be preserved and then normalized into report-ready views
- the team is validating report grain, status semantics, or SQL-backed reporting behavior

## Relationship to other projects

- receives raw source facts from `britecore-csv-loader`
- feeds the service-layer queries in `PolicyReporting`
- provides the staging layer for downstream regulatory work in `NAIC`

## Important boundary

This repo is the warehouse boundary, not the source extraction layer and not the final external reporting layer. It is where raw and normalized semantics get separated cleanly.
