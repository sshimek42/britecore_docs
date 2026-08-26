# Bridge Table Guide

This page explains the difference between base entity tables and bridge tables in the BriteCore reporting and API ecosystem.

## Why bridge tables exist

Many BriteCore relationships are not one-to-one. A policy may have many contacts, a claim may have many claimants and adjusters, and a revision may be linked to several agencies or secondary parties.

When the same record can appear under more than one related object, the reporting layer usually splits that relationship into a bridge table or relationship view.

The same pattern appears in both reporting SQL and CSV export logic:

- base tables hold the main entity record
- bridge tables hold the linkage rows
- each bridge row usually represents one relationship instance

This is why a direct join from a master table can sometimes look too small or too broad: the relationship itself has its own grain.

## Base tables vs bridge tables

### Base tables

Base tables are the primary record holders.

Examples:

- `v_contacts`
- `v_claims`
- `v_policies` or `v_revisions`
- `v_properties`

These tables usually represent one row per core business object.

A base table often has a primary entity id such as:

- `contact_id`
- `claim_id`
- `revision_id`
- `property_id`

### Bridge tables

Bridge tables hold the many-to-many or one-to-many relationship rows.

Examples:

- `v_claims_contacts`
- `v_revisions_contacts`
- `v_revisions_agencies`
- `v_claims_payments`

These tables usually do not represent a new object in the same way as a base entity. Instead, they connect two objects together and often repeat the parent id on multiple rows.

Typical key pattern:

- `claim_id` + `contact_id`
- `revision_id` + `contact_id`
- `revision_id` + `agency_id`

That means a bridge table can have multiple rows for the same base entity if it has multiple related records.

## Common pattern in the BriteCore schema

The dominant pattern is:

1. find the master entity id
2. use that id to join into the bridge relationship table
3. then join from the bridge table to the related entity table

Example:

```sql
SELECT c.claim_id, c.claim_number, cc.contact_id, ct.contact_name
FROM v_claims c
LEFT JOIN v_claims_contacts cc
  ON c.claim_id = cc.claim_id
LEFT JOIN v_contacts ct
  ON cc.contact_id = ct.contact_id
WHERE c.claim_number = 'YOUR_CLAIM_NUMBER';
```

This is normal and expected. A claim can have many related contacts, and the bridge table expands the relationship into one row per relationship.

## Why bridge rows multiply

Bridge tables are often intentionally denormalized for reporting convenience, but that also means they can look repetitive.

You may see:

- one claim with several associated contacts
- one revision with several linked agencies
- one policy revision with multiple named insured or interest records
- one claim with several payment or exposure rows

This is not a bug. It is the actual grain of the relationship.

A bridge-table row should be read as:

- “this parent is linked to this child”
- not “this is a second copy of the parent object”

## Practical examples

### Claim-to-contact bridge

`v_claims_contacts` is a classic bridge relationship table.

It usually connects:

- `claim_id`
- `contact_id`
- maybe role flags or relationship type

Example:

```sql
SELECT *
FROM v_claims_contacts
WHERE claim_id = 'YOUR_CLAIM_ID';
```

This returns the list of contacts associated with the claim. The next step is usually joining to `v_contacts` to get the contact master fields.

### Revision-to-contact bridge

`v_revisions_contacts` follows the same pattern, but at the revision layer.

This is often the key path when a record is anchored in a policy revision rather than a claim or contact record.

```sql
SELECT r.revision_id, r.policy_number, rc.contact_id, c.contact_name
FROM v_revisions r
LEFT JOIN v_revisions_contacts rc
  ON r.revision_id = rc.revision_id
LEFT JOIN v_contacts c
  ON rc.contact_id = c.contact_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

### Revision-to-agency bridge

`v_revisions_agencies` is another good example of a relationship view that is not a base entity row.

Its purpose is to connect:

- a policy revision
- to one or more agency records

This is not a duplicate agency table; it is the agency relationship layer.

## CSV analog

The CSV export follows the same design concept, although the files are more fragmented.

Examples:

- `claims.csv` = master claim rows
- `claims_contacts.csv` = claim-to-contact bridge rows
- `policies.csv` = policy master rows
- `policy_terms.csv` = revision/term rows
- `addresses.csv` / `phones.csv` / `emails.csv` = child object tables that are often flattened later into API or SQL views

This is the same pattern as report views, just expressed in raw export files rather than a database view layer.

## How to work with bridge tables safely

When working with a bridge-table-heavy dataset, use this pattern:

1. Start with the base entity id you already know.
2. Check whether the relationship is stored in a bridge table.
3. Join the bridge table to the related base table.
4. Keep the grain in mind: bridge rows can multiply the result set.
5. Use the base id and relationship ids to validate whether you are looking at a relationship row or a master record.

## Quick rule of thumb

- Base table = one row per entity
- Bridge table = one row per relationship
- If the same parent id repeats and the result set expands, you are probably looking at a bridge or link table

## Common gotchas

### 1. “Why are there multiple rows for one contact?”

Because the same contact may be linked to multiple revisions, claims, agencies, or roles.

### 2. “Is this a duplicate?”

Maybe not. It may be a valid bridge row representing a separate relationship instance.

### 3. “Why do the CSVs and SQL views look different?”

Because the raw CSV layer is not always pre-aggregated like the report layer. The bridge layer may be split across many files and then flattened later.

## Recommended documentation principle

When a table name includes a relationship phrase such as:

- `claims_contacts`
- `revisions_contacts`
- `revisions_agencies`

treat it as a relationship layer, not as a new master entity table.

This is especially important when building join logic or validating whether a result set is showing duplicated relationship rows rather than duplicate master records.
