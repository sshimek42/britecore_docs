# Data Flow and Grain Guide

This page explains the practical path a BriteCore fact takes through the ecosystem and the grain at which each layer operates.

## The high-level flow

```text
Raw BriteCore exports
    |
    +--> britecore-csv-loader
    |       - loads raw CSVs
    |       - discovers relationship IDs
    |       - reconstructs revision snapshots
    |
    +--> policy-data-warehouse
    |       - lands raw data into raw.* tables
    |       - normalizes into core.* tables
    |       - exposes reporting.* views
    |
    +--> PolicyReporting
            - fans out multi-source queries
            - normalizes service-layer objects
            - supports cross-source reporting use cases

Optional downstream products:
    - NAIC regulatory extraction/reporting
    - SQL/reporting outputs
    - API or UI service layers
```

## Important nuance

This is a useful conceptual flow, but not a mandatory chain.

In real work, teams often choose one of these access patterns:

- CSV-only investigation for the least aggregated, most source-faithful view
- API-only investigation for nested domain objects and entity payloads
- SQL/report-only investigation for flattened, business-oriented projections
- mixed mode when a team needs to validate source facts plus reporting output

The right choice depends on access, speed, and the level of aggregation you want built in.

## Grain by layer

### 1) Raw export layer

The raw export model is generally revision-centric.

Typical facts at this layer:

- revision records
- policy records as seen in exported CSVs
- property, claim, item, contact linkage fields
- raw relationship IDs such as `revisionId`, `policyId`, `claimId`, `propertyId`, `itemId`, `contactId`

This layer is best for:

- understanding row-to-row linkage
- validating joins before SQL/report design
- reconstructing policy or claim snapshots from raw files

### 2) Warehouse layer

The warehouse layer turns raw data into a more stable reporting model.

Typical outputs:

- `raw.*` landing tables
- `core.*` normalized tables
- `reporting.*` reusable views

This layer is best for:

- durable operational reporting
- source-safe landed data
- validation against report grain and consistent joins
- shared SQL reporting over standardized records

### 3) Service layer

The service layer assembles more business-facing objects across sources.

Typical outputs:

- cross-source policy queries
- report objects built from multiple systems
- normalized higher-level business answers

This layer is best for:

- multi-source reporting
- source preference logic
- cross-system retrieval and deduplication

### 4) Regulatory or product-specific layer

Regulatory and product-specific layers sit above the source and warehouse layers and translate normalized business facts into external schemas.

Typical outputs:

- NAIC form-coded policy outputs
- report deliverables
- validation and audit artifacts

## Important grain rule

Do not assume the same grain across layers.

Examples:

- a raw CSV row may represent one revision snapshot
- a warehouse row may represent one normalized policy or claim
- a report object may represent one reporting-period, state, form, or company summary

A data point is only safe to use if you know the grain of the row and the join path.

## Common mismatch patterns

### Raw export grain != reporting grain

A raw file can contain many rows per policy or claim. The warehouse and service layers must collapse or aggregate them intentionally.

### Revision snapshots vs current policy state

BriteCore exports are often revision-based. A revision may reflect a policy state at a point in time, not the latest policy state in all cases. It is critical to distinguish:

- event-time or revision-time view
- current-state view
- reporting-period view

### Source-specific semantics vs standardized semantics

A field may have a raw value in BriteCore and a normalized meaning in reporting or regulatory layers. The correct usage depends on the target product.

## Practical rule for troubleshooting

When a result seems wrong, ask the following in order:

1. What layer am I looking at: raw, warehouse, service, or regulatory?
2. What is the grain of the record?
3. What ID is tying rows together?
4. Is the problem source data, join logic, or normalization logic?
5. Is the output trying to represent a current state or a historical revision?

## Recommended document set for this view

- `source_of_truth_matrix.md` — repo and canonical ownership
- `lineage_guide.md` — how IDs and record families connect
- `britecore_status_and_cancellation_semantics.md` — source-specific status logic
- `repo_selection_guide.md` — which project to use for which task

This is the clearest “bridge” knowledge for reducing redundant and ambiguous documentation.
