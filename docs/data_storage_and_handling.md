# BriteCore Data Storage and Handling

This page summarizes how BriteCore data moves between the live API, the logical discovery layer, the SQL reporting project, and the raw CSV export model. It is a cross-project synthesis of the local source repos and should be read as a practical architecture summary, not as a replacement for the canonical implementation docs in each repo.

## Executive summary

The local ecosystem suggests a layered model rather than a single monolithic store:

1. The live BriteCore application stores and serves operational data through the API and report execution layer.
2. The logical discovery layer (`britecore_mcp`) exposes report metadata, `m_*`/`v_*` discovery, and join guidance.
3. The SQL reporting project (`britecore_sql_reports`) owns the report SQL definitions, report-table docs, and reusable report assets.
4. The raw export layer extracts CSV snapshots from the same underlying domain and organizes them around keys such as `policyId`, `revisionId`, `claimId`, `itemId`, `contactId`, and `propertyId`.
5. The service layer (`PolicyReporting`) normalizes multiple inputs into a more query-friendly model for cross-source reporting.

The important design insight is that these are not competing copies of the same data. They are different access surfaces over the same business domain, each with a different purpose, grain, and update rhythm.

## 1) Source-of-truth model by access surface

### A. Live API layer

Canonical repo: `britecore_sdk`

This is the API-first access layer. It is responsible for:

- authenticated access to BriteCore endpoints
- configuration and environment-based connection management
- Python client wrappers for operational or reporting calls
- live retrieval of data from the BriteCore service

The SDK is not the same thing as the reporting catalog. It gives you access to the system as a live API surface, which means:

- you can query operational endpoints and report definitions
- you can retrieve metadata about SQL reports and their tokens
- you can integrate automation or validation scripts against the live system

According to `britecore_mcp` docs, this is also how report metadata can be discovered programmatically:

- `retrieve_sql_reports()` lists report definitions
- `retrieve_report(report_id=...)` fetches the report payload and tokens
- the raw payload exposes report parameters such as `StartDate`, `EndDate`, and other SQL runner inputs

This matters because the built-in SQL reporting layer is not just a database view; it is also a report metadata system with execution parameters and report definitions.

### B. Built-in SQL reporting layer

Canonical repos: `britecore_mcp`, `britecore_sql_reports`, `PolicyReporting`

The built-in SQL/reporting layer is best understood through the logical catalog model:

- `m_*` views are usually fact-heavy, time- or balance-oriented structures
- `v_*` views are usually dimensions, bridges, lookups, and enrichment views
- `v_logical_catalog` is the best starting point for discovering this layer

The `britecore_mcp` docs explicitly say the logical layer is the primary discovery mechanism and that the practical reporting model is facts vs dimensions. The SQL reporting project then carries the reusable report definitions and report-writing assets. Together, they form the SQL/reporting story without forcing MCP to own every report artifact.

The logic is:

- identify the business object and grain
- start with the fact table that matches the business measure
- enrich with dimension views only as needed
- validate row counts and distinct keys to avoid fan-out

This is a curated, analytical surface, not the same as the raw transaction tables in a normalized relational model.

### C. CSV export / raw relationship layer

Canonical repo: `britecore-csv-loader`

The raw export model is the clearest example of how BriteCore data is shaped for downstream reconstruction and analysis. The key document, `CSV_RELATIONSHIPS_OVERVIEW.md`, makes several critical observations:

- the export model is policy-centric but operationally revision-centric
- the most reused keys are `revisionId`, `policyId`, `claimId`, `itemId`, `contactId`, and `propertyId`
- multiple raw CSVs join through these IDs rather than a single normalized relational PK/FK model
- `revisions.csv` acts as an anchor for a large portion of the policy and item lineage

This implies that the exported CSV data is not merely a flat dump; it is a reconstruction of business relationships across policy, claim, property, item, and contact objects.

The raw export view is especially useful for:

- tracing policy lineage across revisions
- validating row grain assumptions before joining report tables
- reconstructing the object graph behind a claim or policy snapshot
- comparing raw export relationships to the logical reporting layer

### D. Service-layer normalization (`PolicyReporting`)

Canonical repo: `PolicyReporting`

This project sits above the raw source interfaces and provides a reporting-oriented service boundary. The README describes a stack of:

- raw parsers
- source interfaces
- DAL layer
- service layer
- future output renderers / API layer / UI

This is a very important architectural clue: BriteCore data is not consumed only through one final query layer. Instead, data is staged and normalized across several layers:

- raw source files or exports are landed into `raw.*` tables
- normalized data is loaded into `core.*` tables
- reporting views are built in `reporting.*`
- service logic assembles higher-level cross-source answer sets

This is the clearest evidence that the system is intentionally layered for performance, normalization, and reporting flexibility.

## 2) The data flow as a practical end-to-end pipeline

A likely end-to-end model looks like this:

```text
BriteCore operational system
        |
        +--> API endpoints / report definitions (via britecore_sdk)
        |
        +--> SQL report runners / logical reporting layer (via britecore_mcp + britecore_sql_reports + v_logical_catalog)
        |
        +--> CSV exports and raw relationship snapshots (via britecore-csv-loader)
        |
        +--> Cross-source service assembly (via PolicyReporting)
```

More concretely:

```text
Operational BriteCore data
    -> API access / live retrieval
    -> report definition discovery and execution
    -> logical views (`m_*`, `v_*`)
    -> raw CSV snapshots keyed by revision/policy/claim/item/contact ids
    -> normalized SQL loading into core/reporting schemas
    -> cross-source reporting/service output
```

## 3) What the repo evidence says about actual storage patterns

### Key evidence from `britecore-csv-loader`

The raw CSV relationship overview identifies the dominant relationship model:

- `revisionId` is the highest-value link field across many datasets
- `policyId` is the primary policy anchor
- `claimId` threads claim-related data together
- `itemId` and `propertyId` connect coverage and risk details
- `contactId` connects contact and party relationships

This suggests that BriteCore data is heavily modeled around revisioned, object-linked snapshots rather than a simple warehouse-style star schema. The raw export layer is not a direct physical database mirror; it is a practical flattened relational reconstruction of business entities and links.

### Key evidence from `britecore_mcp`

The docs across `britecore_mcp` and `britecore_sql_reports` make the reporting model explicit:

- `m_*` objects are usually fact tables
- `v_*` objects are usually dimensions, lookup tables, and bridges
- the target grain should be identified before joins
- `v_logical_catalog` is the authoritative discovery surface

This is a strong signal that the SQL/reporting stack is intentionally curated and semantic, not just raw storage.

### Key evidence from `PolicyReporting`

The project separates raw landing, core schema load, and reporting views:

- `raw.*` is for landing source datasets
- `core.*` is for normalized canonical source data
- `reporting.*` contains final reporting views

This is the clearest representation of a staged data pipeline. It says the service reporting layer is designed around explicit transformation stages rather than direct ad hoc queries over live source exports.

## 4) Important insights discovered

### Insight 1: The reporting layer is intentionally semantic, not just raw storage

The logical layer is not meant to be direct physical storage. It is a curated reporting abstraction designed around facts and dimensions. This explains why there are so many `v_*` views and why `v_logical_catalog` is central.

### Insight 2: Revision is the central organizing concept in the raw model

The CSV export analysis makes `revisionId` the dominant key across many datasets. This suggests that policy and claim event history are organized around revision timelines and revision-scoped entity snapshots. In other words, the raw data is highly time-aware and event-driven.

### Insight 3: The system is layered by purpose, not by a single schema

The data flow looks like this:

- operational / API layer
- reporting layer / logical catalog
- raw export reconstruction layer
- service normalization layer

Each layer answers a different question and uses a different grain. The challenge is not to force them into a single schema; it is to understand when each layer should be used.

### Insight 4: Join fan-out is a major risk area

`britecore_mcp` explicitly warns about grain and join fan-out. This is especially important when working with revision, claim, contact, and policy relationship tables. A raw export or logical view may look valid in isolation but become incorrect when joined too aggressively.

### Insight 5: CSVs are a reconstruction surface, not a “lesser” copy

The raw CSV model is essential for lineage validation and for understanding how relationships are assembled in practice. It should not be dismissed as just a dump. It is often the best place to validate the assumptions behind the curated reporting layer.

## 5) Suggested canonical reading order for a detailed data-model study

If the goal is to document BriteCore data handling end-to-end, this is the order I would use:

1. `britecore_sdk` architecture and API docs
   - for the live system access layer
2. `britecore_mcp` logical catalog docs
   - for the reporting semantics and fact-vs-dimension model
3. `britecore_sql_reports` report-table framework and report docs
   - for report SQL definitions and worked reporting examples
4. `britecore-csv-loader` `CSV_RELATIONSHIPS_OVERVIEW.md`
   - for the raw relationship model and worked examples
5. `PolicyReporting` README
   - for the layered raw/core/reporting service model
5. This doc, plus any final cross-project narrative summary

## 6) Best working mental model

A useful summary model is:

- API access = live operational access
- logical reporting = curated analytics surface
- CSV exports = raw relationship reconstruction layer
- SQL loading/service layer = normalized staging and reporting service boundary

When these are viewed together, the BriteCore data architecture looks less like a single schema and more like a deliberately layered domain model built to serve both analyst exploration and operational reporting.

## 7) Recommended next doc to generate

The next high-value document to add would be a dedicated “BriteCore data lineage map” that ties together:

- key entities (`policy`, `revision`, `claim`, `item`, `contact`, `property`)
- the primary keys used in each layer
- the common join paths used in the raw CSV layer
- the reporting views used in the logical layer
- the normalized stages used in `PolicyReporting`

That would turn this summary into a complete cross-project lineage guide.
