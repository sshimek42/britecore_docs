# BriteCore Entity Relationship Matrix

This page captures the core business-object relationships that repeatedly show up in the BriteCore reporting layer and raw export reconstruction work.

It is based primarily on the `britecore_mcp` logical catalog, the `britecore_sql_reports` report-table documentation framework, and the raw CSV relationship notes from the CSV loader project. It is meant to be a practical reference for analysts and engineers, not a physical DDL specification.

## Source of truth and warning

The best evidence for these relationships comes from:

- `britecore_sql_reports` / `scripts/report_docs/report_table_doc_framework.md`
- `britecore_sql_reports` / `scripts/report_docs/report_join_starters.md`
- `britecore_sql_reports` / `scripts/report_docs/report_table_v_logical_catalog.md`
- `britecore_sql_reports` / `scripts/report_docs/report_table_v_claims.md`
- `britecore_sql_reports` / `scripts/report_docs/report_table_v_contacts.md`
- `britecore-csv-loader` relationship overview for CSV reconstruction

Important caveat:

- this is a logical relationship matrix, not enforced database schema metadata
- the logical catalog tells us what the reporting layer expects to join on, but it does not guarantee physical keys, nullability, indexes, or exact DDL
- CSV and logical docs often agree on the same object IDs, but the real-world relationship structure still needs row-count and grain validation before production use

## Core identity map

The most important IDs across the ecosystem are:

- `policyId` / `policy_id`
- `revisionId` / `revision_id`
- `claimId` / `claim_id`
- `itemId` / `item_id`
- `propertyId` / `property_id`
- `contactId` / `contact_id`

These are the working backbone of the BriteCore graph. In practice, the most useful relationship chains are:

```text
policy -> revision -> item/property
policy -> claim -> item/property/contact
claim -> claim contacts / claim payments / claim items
contact -> claim participation / insured role / agency relationship
```

## Relationship matrix

| Entity | Primary key(s) | Usually connects to | Common join/path | Grain | Notes |
|---|---|---|---|---|---|
| `Policy` | `policy_id`, `policy_number` | revision, claim, insured, property, agency | `policy_id -> revision_id -> claim_id` | one row per active or historical policy record in the logical view | The policy object is the anchor for broader operational history; the reporting layer usually enriches it with revision, insured, and agent context |
| `Revision` | `revision_id` | policy, item, property, claim, agency, premium history | `revision_id -> item_id`, `revision_id -> property_id`, `revision_id -> claim_id` | one row per revision state or term state | This is the most important raw-export anchor. The CSV work repeatedly points to revision-centric behavior rather than a purely policy-table model |
| `Claim` | `claim_id`, `claim_number` | policy, revision, property, claim contacts, claim items, claim payments, disputes, perils | `claim_id -> revision_id -> policy_id`, `claim_id -> property_id`, `claim_id -> contact_id` | one row per claim header | `v_claims` is the primary claim header view; downstream claim detail tables are bridge/detail views |
| `Item` | `item_id` | policy revision, property, claim items, premium/coverage contexts | `item_id -> revision_id`, `item_id -> policy_id`, `item_id -> claim_id` | one row per insured item or risk item | Often represents the coverage-level object under a policy version; important for exposure and claim drill-through |
| `Property` | `property_id` | revision, item, claim, county, address | `property_id -> revision_id`, `property_id -> item_id`, `property_id -> claim_id` | one row per property or risk location | Shows up in both policy and claim lineage; often used for location and exposure context |
| `Contact` | `contact_id` | claim, insured, agency, address, roles, relationships | `contact_id -> claim_id`, `contact_id -> policy_id`, `contact_id -> agency_id` | one row per contact or party record | `v_contacts` is the canonical enterprise contact universe; role data is denormalized and may require parsing |
| `Agency / Producer` | `agency_id`, `producer_number` | policy revision, commission details, contacts | `agency_id -> contact_id`, `revision_id -> agency_id` | one row per agency or producer record | Often surfaced through contact roles and revision agency context; used heavily in billing and commission logic |
| `Claim Contact` | `claim_id + contact_id` | claim, contact, insured, adjuster, claimant | `claim_id -> v_claims_contacts -> contact_id` | one row per claim-party participation | This is a bridge table-like view; it can multiply rows if used without understanding grain |
| `Claim Item` | `claim_id + item_id` | claim, item, policy revision | `claim_id -> v_claim_items -> item_id` | one row per claim-item relationship | Great for claim coverage detail, but easy to join too aggressively if the viewer expects claim-level grain |
| `Claim Payment` | `claim_id + payment_id` | claim, payment ledger, policy/revision context | `claim_id -> v_claim_payments` | one row per claim payment transaction or payment summary key | Good example of a detail table that is not the same grain as the claim header |
| `Address / County` | `address_id`, `county_id` | property, claim, contact, policy location data | `county_id -> v_counties`, `contact_id -> v_addresses` | one row per address or county reference | Usually enrichment tables or lookup tables, not the business anchor itself |

## Most common join chains in the logical layer

These are the patterns that appear repeatedly in the MCP docs and are the best starting points for report design.

### 1) Policy reporting

```text
m_inforce_policies
    -> v_revisions via revision_id
    -> v_policy_types via policy_type_id
    -> v_insureds via primary_insured_id
    -> v_properties via revision_id or downstream property linkage
    -> v_revisions_agencies via revision_id
```

This is the classic policy-centric reporting chain. It is useful for in-force counts, premium, and insured population analysis.

### 2) Claim reporting

```text
v_claims
    -> v_claims_dates via claim_id
    -> v_claims_perils via claim_id
    -> v_claims_contacts via claim_id
    -> v_claim_payments via claim_id
    -> v_claim_items via claim_id
    -> v_claim_change_log via claim_id
    -> v_revisions via revision_id
    -> v_counties via loss_location_address_county_id
```

This is the main claim-detail chain and reflects the fact that the claim table is usually the anchor while the rest are bridge/detail satellites.

### 3) Contact / party reporting

```text
v_contacts
    -> v_addresses via contact_id
    -> v_contact_relationships via subject_contact_id / related_contact_id
    -> v_claims_contacts via contact_id
    -> v_commission_payments via payee_contact_id
```

This is the party-centric chain and is particularly useful for agency, insured, claimant, and role analysis.

### 4) Billing and premium reporting

```text
m_premium_terms or m_premium_transactions
    -> v_revisions via revision_id
    -> v_policy_types via policy_type_id
    -> v_insureds via primary_insured_id
    -> v_revisions_agencies via revision_id
```

This is the policy-billing chain and is one of the key places where revision semantics matter.

## The critical grain rule

The MCP docs repeatedly emphasize that grain must be defined before joins. This is the most important rule in the matrix.

A table may look valid to join because it has a shared ID, but the relationship can still be wrong if the destination table expands the row count.

Examples:

- `v_claims_contacts` often adds one row per claim-party relationship; it is not a one-row-per-claim table
- `v_contact_relationships` can create multiple relationship rows per contact
- `v_claim_items` may multiply a claim row if a single claim has multiple covered items
- `v_revisions` can represent several policy-version snapshots, which must be used with careful date or effectivity logic

The right pattern is: define the target grain first, then join only the supporting detail tables that are required to answer the question.

## What the CSV export model confirms

The raw CSV relationship overview is consistent with the logical layer and reinforces a few key points:

- `revisionId` is the most important raw anchor for many policy and policy-history relationships
- `policyId` is the policy anchor
- `claimId` ties claim-related data together
- `itemId` and `propertyId` tie risk and coverage detail together
- `contactId` ties people and parties into the graph

This means the logical reporting model is not arbitrary: it is an abstraction over a real relationship graph that is still visible in the raw CSV exports.

## Recommended interpretation for reporting work

When building a BriteCore report, use this sequence:

1. Start with the fact or anchor table that matches the business question.
2. Confirm the grain of the anchor table.
3. Add only the dimension or bridge tables needed for context.
4. Validate the join by checking row counts and distinct key counts.
5. Use the raw CSV view and `v_logical_catalog` together when the relationship is ambiguous.

## Summary

The core of the BriteCore data model is a relationship graph built around policy, revision, claim, item, property, and contact identities. The logical reporting layer abstracts this graph into fact and dimension tables, while the CSV export layer makes the underlying links visible in a more operational form.

The real key to understanding the system is not any single table, but the pattern of shared IDs and the way they connect across the reporting, raw-export, and service layers.
