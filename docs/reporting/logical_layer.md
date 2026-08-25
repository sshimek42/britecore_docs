# Reporting Logical Layer

This page consolidates the current mental model for the BriteCore reporting layer.

## Facts vs dimensions

A practical reporting model is:

- **Facts** = the core business records or measurements you want to analyze
- **Dimensions** = descriptive context such as names, dates, policy types, addresses, roles, or agencies

In most cases:
- start with the dataset that matches the business question
- define the target grain first
- join additional views only when you need more detail

## `m_*` vs `v_*`

### `m_*` views
- usually pre-computed and date-driven
- often the best fact-table anchors for performance-sensitive reporting
- commonly used for accounting, premium, in-force, and profitability style reporting

### `v_*` views
- logical views, dimensions, bridges, lookups, and audit/enrichment layers
- often the best starting point for entity-centric reporting such as claims, contacts, files, and revisions
- also used to resolve ids into human-readable attributes

## Current authoritative discovery source

Use:

```sql
SELECT *
FROM v_logical_catalog;
```

This remains the best source for:
- current object inventory
- field names
- logical types
- embedded field descriptions
- join/discovery hints

## Common join starter patterns

### Policies
- start with `m_inforce_policies`
- enrich with `v_policy_types`, `v_revisions`, `v_insureds`, `v_properties`

### Claims
- start with `v_claims`
- enrich with `v_claims_dates`, `v_claims_perils`, `v_claims_contacts`, `v_claim_payments`, `v_claim_items`

### Premiums
- start with `m_premium_terms` or `m_premium_transactions`
- enrich with `v_return_premium`, `v_account_history`, `v_revisions`, `v_policy_types`

### Audit / lineage
- start with `v_revisions`
- enrich with `v_revision_items`, `v_revisions_contacts`, `v_revisions_agencies`

For a more practical guide, see the join starters page.

