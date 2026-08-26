# Lineage Guide

This page captures the most important lineage concepts used across raw exports, warehouse loads, and reporting layers.

## Core identity fields

The raw BriteCore ecosystem is heavily keyed off identity fields and revision-based relationships.

Most common identifiers:

- `revisionId`
- `policyId`
- `claimId`
- `propertyId`
- `itemId`
- `contactId`

These are not just arbitrary IDs. They form the practical lineage chain for understanding how a policy, claim, or property is related across multiple exported files.

## Why this matters

When you are working with exported CSVs, a single policy may be spread across:

- revision rows
- policy rows
- property rows
- contact rows
- claim rows
- item rows

The relationship is often not direct in a single file. Instead, it is stitched together via shared IDs and the revision frame.

## Typical lineage pattern

```text
revision row
    |
    +--> policyId
    +--> propertyId
    +--> claimId(s)
    +--> itemId(s)
    +--> contactId(s)

This produces a policy snapshot or claim-linked view when the IDs are resolved together.
```

## Important conceptual distinction

### Revision-based lineage

A revision is often the best starting point when you ask:

- what changed over time?
- what was the policy state at a point in time?
- what rows belong to the same policy revision snapshot?

### Policy-centric lineage

A policy is often the best starting point when you ask:

- what is the current or reporting-period policy state?
- what properties and claims belong to this policy?
- how do I aggregate by policy for reporting?

### Claim-centric lineage

A claim is often the best starting point when you ask:

- what happened in a claim record?
- what policy or property is attached?
- how do I reconcile claim severity or claim counts?

## Practical use rule

Start from the question, not the table.

- If the question is about lifecycle change, start with revision lineage.
- If the question is about current policy semantics, start with policy lineage.
- If the question is about loss events, start with claim lineage.

## Warehouse and reporting implications

The warehouse and reporting layers are designed to preserve these distinctions while creating a more queryable model.

The goal is not to flatten everything into one giant table. The goal is to make the lineage explicit enough that:

- joins are auditable
- grain is known
- current-state and historical-state questions are separable

## Common lineage bug

A frequent mistake is treating a raw export row as if it were the final reporting record. In BriteCore, the raw rows often represent source facts, not report-level narrative output. The lineage logic must bridge the source row to the normalized or aggregated reporting layer.

## Best practice for any investigation

When tracing a fact, always ask:

1. Which record family does it start in?
2. Which shared ID connects it to the next family?
3. Is the target question revision-based, policy-based, or claim-based?
4. Is the final output a raw fact, warehouse fact, or reporting aggregate?

This is the fastest way to avoid false joins and inconsistent reporting grain.
