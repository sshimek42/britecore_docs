# NAIC

## Role

Regulatory extraction and reporting project for NAIC-form outputs built from BriteCore and MIPS-derived policy data.

## Bridge references

For the ecosystem context, start with:

- [Source-of-Truth Matrix](../source_of_truth_matrix.md)
- [Data Flow and Grain Guide](../data_flow_and_grain_guide.md)
- [Repo Selection Guide](../repo_selection_guide.md)

## What the project is for

`NAIC` is the final translation layer for regulatory reporting. It is strongest at:

- translating source policy data into NAIC-aligned reporting fields
- mapping BriteCore values to NAIC form and coverage logic
- config-driven report generation across companies, states, and reporting years
- validating whether source data is ready for a regulatory run

## Best-known documentation entry points

- `C:\PythonProjects\data\NAIC\README.md`
- `C:\PythonProjects\data\NAIC\docs\BRITECORE_INTEGRATION_GUIDE.md`
- `C:\PythonProjects\data\NAIC\docs\BRITECORE_DATA_MAPPING.md`
- `C:\PythonProjects\data\NAIC\docs\BRITECORE_TO_NAIC_SOURCE_JOIN_DIAGRAM.md`

## Typical use cases

Use the `NAIC` project when:

- the task is specifically about NAIC reporting output
- a BriteCore or MIPS dataset must be translated into form-coded regulatory fields
- you are validating source readiness for a reporting run or audit trail

## Relationship to other projects

- relies on the warehouse and source semantics assembled by `policy-data-warehouse`
- can be informed by the raw lineage work in `britecore-csv-loader`
- is distinct from `PolicyReporting`, which is the broader cross-source service layer rather than a single external reporting schema

## Important boundary

This project is not the canonical source for raw export reconstruction or warehouse logic. It is the canonical source for regulatory translation and report generation logic once the underlying data is known to be valid.
