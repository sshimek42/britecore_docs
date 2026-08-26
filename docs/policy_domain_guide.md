# Policy and Revision Domain Guide

This page is the practical reference for working with policy and revision data across CSV, API, and reporting layers.

## What a policy/revision is

A policy is the contract or coverage record, while a revision is usually the term-level or life-cycle change state associated with that policy.

In practice this means:

- the policy gives the business identity (`policy_number`, policy id)
- the revision gives the term-level state and lifecycle detail (`revision_id`, effective date, expiration date, status)
- claims, property records, contacts, and agencies often all fan out from the revision or policy context

## Core identity

The most important identities are:

- `policy_number` — the human-facing business key
- `policy_id` — internal policy identity
- `revision_id` — internal revision or term anchor
- `policy_term_id` — term-specific identity in some report layers

The typical pattern is:

```text
policy_number -> policy_id / revision_id -> revision context -> claims / properties / contacts
```

## Best starting tables

### CSV

- `policies.csv` — policy master row
- `policy_terms.csv` — term or policy-term row
- `revisions.csv` — revision lifecycle detail
- `properties.csv` — property/risk records attached to a revision
- `mortgagees.csv` — mortgagee or interest records attached to a property/revision

### API

- `policies.retrieve_policy(policy_number=...)`
- `policies.retrieve_policy_ids(policy_number=...)`
- `policies.retrieve_policy_terms(...)`
- `policies.retrieve_revision_details(revision_id=...)`
- `policies.retrieve_risks(...)` and related risk/property endpoints

### SQL/report

- `v_revisions` — revision and term lifecycle view
- `v_properties` — property records at revision and risk grain
- `v_revisions_contacts` — revision-to-contact relationship rows
- `v_revisions_agencies` — revision-to-agency relationship rows
- `v_claims` — claim rows attached to the revision context

## Typical trace pattern

### Find the policy and revision

```sql
SELECT r.revision_id,
       r.policy_number,
       r.policy_status,
       r.term_effective_date,
       r.term_expiration_date,
       r.policy_type_id
FROM v_revisions r
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

### Find the claims tied to the revision

```sql
SELECT c.claim_id,
       c.claim_number,
       c.claim_status,
       c.loss_date
FROM v_claims c
WHERE c.revision_id = 'YOUR_REVISION_ID';
```

### Find the properties tied to the revision

```sql
SELECT p.property_id,
       p.property_name,
       p.risk_city,
       p.risk_state,
       p.risk_zip
FROM v_properties p
WHERE p.revision_id = 'YOUR_REVISION_ID';
```

### Find the contacts tied to the revision

```sql
SELECT rc.revision_id,
       rc.contact_id,
       c.contact_name,
       c.contact_type,
       c.roles
FROM v_revisions_contacts rc
LEFT JOIN v_contacts c
  ON c.contact_id = rc.contact_id
WHERE rc.revision_id = 'YOUR_REVISION_ID';
```

## CSV pattern

The raw CSV export usually separates the policy from the term and revision details, and then spreads child data across additional files.

A policy reconstruction usually looks like:

```text
policies.csv
  -> policy_terms.csv
  -> revisions.csv
  -> properties.csv
  -> mortgagees.csv
  -> addresses.csv / phones.csv / emails.csv
```

This is more fragmented than the API or report layer, but it is often the most faithful representation of the raw system.

## API pattern

The API tends to keep the policy and its related objects in a more structured object model. The wrapper signatures are especially useful because they show exactly which ids are returned and which ids can be used next.

Important examples:

- `retrieve_policy(policy_number=...)`
- `retrieve_policy_ids(policy_number=...)`
- `retrieve_policy_terms(...)`
- `retrieve_revision_details(revision_id=...)`

This is the cleanest way to understand the identity chain when working in the live system.

## Reporting pattern

The reporting layer is usually the simplest place to answer business questions because it is flattened and pre-joined.

The most important reporting anchor is:

- `v_revisions.revision_id`

That column is the central link to claims, contacts, properties, and related details.

## Key fields to watch

Important fields include:

- `policy_number`
- `policy_id`
- `revision_id`
- `policy_term_id`
- `policy_status`
- `revision_state`
- `term_effective_date`
- `term_expiration_date`
- `billing_schedule`
- `cancelDate`
- `renewalStatus`

## Common gotchas

### 1. A policy can have multiple revisions

Do not assume one row per policy. Revision history creates multiple rows for a single policy.

### 2. The current status is not always the same as the historical revision state

A current policy record and a historical revision snapshot are different grains and should not be merged without a clear filter.

### 3. The policy number is not always the key used in all detail tables

Detail tables often use `revision_id` or `policy_id` rather than `policy_number` directly.

### 4. CSV versus API versus SQL are all valid but different

The CSV is fragmented, the API is nested, and the report view is flattened.

## Recommended working model

For policy work, the order is usually:

1. find the policy by `policy_number`
2. resolve the revision id
3. join to claims, properties, and contacts by `revision_id`
4. only then move to more detailed child tables as needed

This will keep the lineage readable and avoid mixing policy-level identities with revision-level or claim-level keys.

## Policy quick reference

```text
policy_number -> revision_id -> claims / properties / contacts / agencies
```

This is the most effective default pattern for policy/revision reconstruction across the ecosystem.
