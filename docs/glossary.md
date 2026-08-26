# Glossary

This glossary defines the core terms used across the BriteCore reporting and documentation stack.

## Fact table

A fact table is the dataset that captures the actual business event or measure being analyzed.

Examples:

- premium transactions
- policy in-force records
- claim payments
- revision metadata
- claim event data

Fact tables are usually the strongest anchors for a metric-based or policy-based report.

## Dimension table

A dimension table provides descriptive context around a fact, such as:

- policy type
- insured name
- property information
- contact metadata
- agency or role details

Dimensions usually enrich a fact table without changing the primary business event being measured.

## Grain

The grain is the unit of a row in the final output.

Examples:

- one row = one policy
- one row = one claim
- one row = one premium term
- one row = one revision

If the grain is wrong, the report may look valid but still be logically incorrect.

## `m_*` views

`m_*` views are usually materialized or performance-oriented data structures.

Common usage:

- fact tables for in-force, premium, or accounting-style reporting
- precomputed reporting layers for known business questions
- analytic surfaces intended for a stable, repeatable grain

## `v_*` views

`v_*` views are usually logical or curated view layers.

Common usage:

- dimensions
- lookup tables
- bridge tables
- claim, contact, policy, and revision context views
- human-readable business views built atop raw or upstream sources

## `v_logical_catalog`

`v_logical_catalog` is the central discovery surface for reporting metadata.

It is commonly used to answer:

- which view contains a field
- which views are fact vs dimension-like
- what the field description says
- which objects are likely to be used together

## Revision

A revision is a versioned or time-aware record representing an update or state change in a policy, claim, or related object.

Revisions are important when:

- a record changes over time
- history must be compared or audited
- policy or claim lineage matters

## Raw export

A raw export is an exported dataset from a source system, often in CSV form.

These exports are useful when:

- the logical SQL layer is not enough
- the relationship model must be reconstructed from files
- lineage validation is required
- the export structure exposes relationships that are hard to infer from the curated views

## Source of truth

A source of truth is the canonical repository or system that owns a given fact, rule, or implementation detail.

Rule of thumb:

- logical metadata often lives in the report layer repo
- implementation logic lives in the owning project repo
- `britecore_docs` organizes and summarizes, rather than replacing the real implementation

## Join fan-out

Join fan-out happens when one row joins to multiple matching rows and the result set grows beyond the intended grain.

Typical causes:

- many-to-one joins used incorrectly
- bridge tables joined without aggregation
- event tables joined to a system of multiple related records

## Bridge table

A bridge table connects one business entity to multiple related entities.

Examples:

- claim to peril association
- contact to role or relationship mapping
- policy to revision or parties table

Bridge tables are often useful for mapping but dangerous when joined without a clear aggregation strategy.

## Lookup table

A lookup table resolves coded IDs into human-readable values.

Examples:

- policy type codes
- agency codes
- state or jurisdiction mapping
- role codes

These are usually safe to join when the business question is about description and classification.

## Data lineage

Data lineage describes how data moves from raw source to processed/reporting view.

In this stack, lineage often matters for:

- export-to-view validation
- claim or policy history review
- debugging mismatched counts
- explaining where a metric originated

## Canonical documentation

Canonical documentation is the authoritative doc set for a repo or system.

The operating principle in this repo is:

- keep detailed implementation material in the original repo
- keep shared summaries here
- link to the canonical source whenever possible

## Reporting layer

The reporting layer is the set of curated views, tables, and documentation used to understand and analyze BriteCore reporting objects.

This layer includes:

- fact tables
- logical views
- bridge tables
- report documents and join guidance

## Service layer

A service layer combines source systems into a higher-order reporting or orchestration interface.

In this stack, `PolicyReporting` is the clearest example of a cross-source service boundary.
