# Record Reconciliation Checklist

This page is the practical validation checklist for confirming that a BriteCore record is being interpreted correctly across the CSV, API, and SQL/report layers.

The purpose is to give a repeatable way to check that a record is really the same business object when the shapes look different.

## Core checklist

### 1. Start with the business key

Identify the human-facing identifier you already have:

- policy number
- claim number
- contact id
- property id
- revision id

Do not start by guessing which internal table to query. Start from the key you already know is valid.

### 2. Resolve to the stable internal id

Next, identify the internal id that the system uses for lineage and bridge tables:

- `revision_id` for policy-term lineage
- `claim_id` for claim-level lineage
- `contact_id` for contact-level lineage
- `property_id` for property/risk lineage

This is the step that makes the bridge tables and detail views usable.

### 3. Confirm the base entity row

Check the master CSV or report table for the business object itself:

- `claims.csv` or `v_claims`
- `contacts.csv` or `v_contacts`
- `policies.csv` or `v_revisions`
- `properties.csv` or `v_properties`

This confirms whether the object exists as a base record and what its raw or flattened fields look like.

### 4. Check the bridge and detail tables

Once you have the internal id, inspect the relationship layer:

- claim to contact
- revision to contact
- claim to payment
- revision to property
- property to mortgagee

If the relationship is not on the master table, it is often on the join or bridge table.

### 5. Compare the same record across layers

The final step is to compare:

- raw CSV row
- API object response
- SQL/report row

What to look for:

- Are the same ids present?
- Are the names and numbers consistent?
- Are the child arrays in the API represented by separate CSV or report tables?
- Is the report row a flattened view of the raw object?

### 6. Check whether the grain changed

This is the step that catches many false alarms.

Examples:

- one master claim but multiple claim-contact rows
- one revision but multiple property rows
- one contact but multiple phone rows
- one payment but multiple payment item rows

If the row count expanded, check bridge or child-table grain rather than assuming duplicate data.

## Reconciliation template

Use this checklist for any record you are trying to reconcile:

### Record identification

- Business key: ______
- Internal id: ______
- Parent/related entity id(s): ______

### Base entity check

- Master CSV file: ______
- Master report view: ______
- Does the base record exist? yes / no

### Relationship check

- Related bridge or detail tables: ______
- Relationship keys present? yes / no
- Does the relationship multiply per row? yes / no

### API check

- API endpoint used: ______
- Response contains nested arrays? yes / no
- Are child collections represented by separate tables? yes / no

### SQL/report check

- SQL view used: ______
- Flattened fields present? yes / no
- Does the row look denormalized or aggregated? yes / no

### Final determination

- Same record across layers? yes / no
- Grain mismatch? yes / no
- Missing relationship between tables? yes / no

## Common validation patterns

### Policy validation

```sql
SELECT r.revision_id, r.policy_number, r.policy_status, c.claim_id, c.claim_number
FROM v_revisions r
LEFT JOIN v_claims c
  ON c.revision_id = r.revision_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

### Claim validation

```sql
SELECT c.claim_id, c.claim_number, c.policy_number, cc.contact_id, ct.contact_name
FROM v_claims c
LEFT JOIN v_claims_contacts cc
  ON cc.claim_id = c.claim_id
LEFT JOIN v_contacts ct
  ON ct.contact_id = cc.contact_id
WHERE c.claim_number = 'YOUR_CLAIM_NUMBER';
```

### Contact validation

```sql
SELECT c.contact_id, c.contact_name, a.address_line_1, p.phone, e.email
FROM v_contacts c
LEFT JOIN v_addresses a
  ON a.contact_id = c.contact_id
LEFT JOIN v_phones p
  ON p.contact_id = c.contact_id
LEFT JOIN v_emails e
  ON e.contact_id = c.contact_id
WHERE c.contact_id = 'YOUR_CONTACT_ID';
```

## Decision rule

If a record appears mismatched across layers, the most common cause is one of these:

- different layer/grain
- bridge table instead of master table
- raw CSV naming differences
- human key vs internal id mismatch
- current-state vs historical-state mix

## Short version

The most reliable reconciliation pattern is:

- business key → internal id → bridge/detail table → master table

If that pattern resolves cleanly, the record is usually being interpreted correctly.
