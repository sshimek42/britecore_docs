# BriteCore Data Lineage Reference

This page is the canonical cross-project lineage summary for how BriteCore data is represented across the API, the logical SQL/reporting layer, the raw CSV exports, and the staged service layer.

It is intentionally a synthesis document. The source repos remain the authoritative implementation layer, but this page ties the pieces together into one practical lineage map.

## Executive summary

BriteCore data is best understood as a layered system rather than a single physical schema.

The working model is:

```text
Operational BriteCore platform
    -> live API and report metadata access
    -> logical reporting catalog (`m_*`, `v_*`)
    -> raw CSV export reconstruction
    -> staged service normalization (`raw.*` -> `core.*` -> `reporting.*`)
    -> downstream reporting outputs and integrations
```

The most important architectural truth is that each layer answers a different question:

- API layer: what is live and operational right now?
- reporting layer: what are the curated business semantics and fact/dimension views?
- CSV layer: how are business relationships reconstructed from exported data?
- service layer: how are multiple source signals normalized into a single reporting shape?

## The core lineage model

The raw export and logical model both cluster around a small set of identifiers that behave like the backbone of the business graph:

- `policyId`
- `revisionId`
- `claimId`
- `itemId`
- `contactId`
- `propertyId`

These IDs are the connective tissue across the ecosystem. They are not “just keys”; they are the object links that let us follow a policy from its current state to historical revisions, claim events, property details, item coverages, and contact relationships.

## How the data moves across each layer

### 1) API layer

Canonical repo: `britecore_sdk`

This is the live system access layer. It is how code interacts with the BriteCore product surface in real time.

Typical responsibilities:

- authenticated calls to BriteCore endpoints
- environment and config management
- automation around live data retrieval
- fetch of report metadata and execution parameters

Important point:

The API is not just a raw database tunnel. It is also the path for report metadata discovery, parameter retrieval, and access to live operational objects that may later show up in logical reporting or exported datasets.

### 2) Logical SQL/reporting layer

Canonical repos: `britecore_mcp`, `britecore_sql_reports`, `PolicyReporting`

This is the curated analytics surface. The reporting docs consistently frame the model as fact tables and dimensions, while `britecore_sql_reports` owns the reusable report SQL and report-writing assets.

- `m_*` objects are usually fact-heavy or metric-rich structures
- `v_*` objects are usually lookup, bridge, or dimension-oriented views
- `v_logical_catalog` is the entry point for discovering the reporting model

This layer is intentionally semantic. It is built for analyst consumption, not just for schema exposure.

The practical pattern is:

1. identify the business grain
2. find the relevant fact table
3. attach the necessary dimension or lookup views
4. validate row counts and join fan-out before trusting the result

### 3) Raw CSV export layer

Canonical repo: `britecore-csv-loader`

The CSV layer is the biggest clue as to the underlying business graph. It reconstructs relationships across policy and claim data using a collection of object IDs and revision snapshots.

Key observations from the raw relationship docs:

- the export model is policy-centric but operationally revision-centric
- `revisionId` is often the most important anchor across datasets
- `policyId` is the policy anchor
- `claimId` links claim activity and claim events
- `itemId` and `propertyId` connect coverage and risk details
- `contactId` connects people, parties, and insured roles

This is not a flat dump; it is a business-graph reconstruction layer.

### 4) Staged service layer

Canonical repo: `PolicyReporting`

This layer makes the staging model explicit:

```text
raw.*
    -> core.*
    -> reporting.*
```

This is the clearest evidence that BriteCore data is designed for staged normalization and reporting assembly rather than one-step direct querying.

The pattern indicates:

- raw source files are landed first
- normalized core tables are created next
- final reporting views are assembled last
- reporting logic is separated by transformation stage

## Entity-by-entity lineage

### Policy lineage

A policy usually lives across multiple surfaces:

```text
API data
    -> policy metadata and current state
    -> policy snapshots / revision history
    -> logical reporting table(s) for policy facts and dimensions
    -> raw CSV policy and revision files
    -> normalized core policy model
    -> reporting policy views
```

Key identifiers:

- `policyId` — stable policy anchor
- `revisionId` — revision state or policy cycle anchor

Typical business meaning:

- current policy record and status
- historical policy versions or amendments
- policy-level coverage, premium, and exposure context

### Revision lineage

`revisionId` is the central organizing idea in the raw export model.

Why it matters:

- policy history is often versioned around revisions
- item and coverage changes may be tied to a specific revision
- claims and transactions may be evaluated relative to the policy revision state

Typical lineage:

```text
policy revision data
    -> revision metadata and effective-dated state
    -> item and property relationships under that revision
    -> claim or transaction activity referencing the revision context
    -> logical reporting views keyed to policy state or revision state
```

### Claim lineage

Claims often connect to policy and revision lineage without being identical to the policy table.

Typical key path:

```text
claimId
    -> policyId
    -> revisionId
    -> itemId / propertyId
    -> contactId for claimant, insured, or involved party
```

This is why claim data often appears as a separate branch in the raw export model: claims are operationally related to policy history but not simply duplicated within the policy table.

### Item lineage

Items are usually the coverage or risk objects under a policy.

Typical lineage:

```text
itemId
    -> policyId
    -> revisionId
    -> propertyId (when the item is tied to a property-based risk)
    -> claim activity, premium history, or coverage lineage
```

This is an important part of the raw relationship reconstruction because items are often where coverage-level and risk-level facts become visible.

### Property lineage

Property objects often anchor risk or location data.

Typical lineage:

```text
propertyId
    -> policyId / revisionId
    -> itemId references
    -> contactId or party links when the property is tied to insured/owner information
    -> report or export slices about exposure, location, or valuation
```

The property branch is critical for understanding exposure and underwriting context, especially when policy and claim data are used together.

### Contact lineage

Contact data is usually a party graph rather than a single direct transaction table.

Typical lineage:

```text
contactId
    -> policyId / revisionId
    -> claimId
    -> propertyId or itemId when relationship is policy- or exposure-based
    -> logical dimensions or lookups for insureds, agents, brokers, or claim parties
```

This means contacts may be captured in a shared identity layer and then attached to policy, claim, and property contexts as needed.

## Primary join patterns to understand

### Pattern 1: Policy to revision

```text
policyId -> revisionId
```

This is one of the most important relationships because many downstream records are revision-specific.

### Pattern 2: Revision to item/property

```text
revisionId -> itemId -> propertyId
```

This captures the exposure and coverage relationship under a policy version.

### Pattern 3: Policy to claim

```text
policyId -> claimId
```

Simple at a high level, but often complicated by claim lifecycle and claim-specific revision contexts.

### Pattern 4: Claim to policy revision and parties

```text
claimId -> policyId -> revisionId
claimId -> contactId
claimId -> itemId / propertyId
```

This is the path for operational claim analysis.

### Pattern 5: Report layer fact-to-dimension joins

```text
fact table (m_*)
    -> dimension or bridge view (v_*)
    -> lookup or policy/contact/property enrichment
```

This is the curated reporting view in the logical catalog. It is the analyst-facing translation of the raw object graph.

## Where the layers differ

### API layer differences

- live and operational
- not necessarily curated for analysis
- often better for retrieving current state and metadata
- more likely to reflect business process state than warehouse-style final reporting shapes

### Logical reporting layer differences

- curated and semantic
- designed for analysis
- fact vs dimension joins are the key mental model
- better for reporting and business questions

### CSV layer differences

- relationship reconstruction and lineage validation
- flattened and exports-oriented
- excellent for tracing keys and object graph behavior
- especially useful when the reporting layer is too abstract or when the grain is unclear

### Service-layer differences

- staged transformation model
- normalized core data before reporting
- better for cross-source assembly and reusable reporting products
- strongest match for a production reporting stack built over multiple data feeds

## Practical guidance for working with this lineage

When investigating a BriteCore data question, use this sequence:

1. Start from the business question and target grain.
2. Use `britecore_mcp` to find the logical facts and dimensions.
3. Use the API if you need live operational state or report metadata.
4. Use `britecore-csv-loader` to validate relationships and identities.
5. Use `PolicyReporting` when a staged or cross-source reporting product is needed.
6. Only after that should you decide whether the answer belongs in a report, a service, or a raw-data validation step.

## Recommended final mental model

The simplest useful mental model is:

```text
API = live operational access
logical reporting = curated business semantics
CSV exports = raw relationship reconstruction
service normalization = staged reporting assembly
```

When these are viewed together, BriteCore data looks less like a single canonical table set and more like a layered insurance operating model with multiple surfaces for operational work, analytics, and downstream reporting.

## Key insight

The most important architecture fact is not where the physical database lives; it is how the business objects are connected across revision history, policy state, claims, coverage items, property details, and contacts.

That connection model is the true key to understanding BriteCore data storage and handling.
