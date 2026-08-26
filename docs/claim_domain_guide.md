# Claim Domain Guide

This page is the practical reference for working with BriteCore claims across CSV, API, and report-view layers.

## What a claim is

A claim is the operational record for an insured loss or claim event. In practice, a claim usually has:

- a claim header (`claim_id`, `claim_number`, status, type, dates)
- a policy or revision lineage (`policy_number`, `revision_id`)
- one or more parties (`contacts`)
- associated financial activity (`payments`, item amounts)
- additional event detail (`dates`, `items`, `perils`, `disputes`, `changes`)

## Core identity

The important identities for a claim are:

- `claim_id` — internal identity for claim detail and bridge tables
- `claim_number` — human-facing claim key
- `revision_id` — policy-term lineage anchor
- `policy_number` — policy business key

The pattern is usually:

```text
claim_number -> claim_id -> claim detail tables
policy_number -> revision_id -> claim linkage
```

## Best starting tables

### CSV

- `claims.csv` — claim master row
- `claims_contacts.csv` — claim party linkage
- `claims_payments.csv` — payment rows
- `claims_payment_item_amounts.csv` or equivalent item detail
- `claims_perils.csv` — perils / coverage context
- `claims_changes.csv` — lifecycle or state change trail

### API

- `claims.get_claim(claim_id=...)`
- `claims.get_claim(claim_number=...)`
- `claims.get_claim_contacts(claim_id=...)`
- `claims.get_claim_payments(claim_id=...)`
- relationship-specific claim endpoints under the claim domain

### SQL/report

- `v_claims` — claim master view
- `v_claims_contacts` — claim-to-contact relationship rows
- `v_claim_payments` — claim payment rows
- `v_claim_items` — item-level claim detail
- `v_claims_dates` — dates and claim timeline fields
- `v_claims_perils` — peril coverage detail

## Common claim trace pattern

```sql
SELECT c.claim_id,
       c.claim_number,
       c.policy_number,
       c.revision_id,
       c.claim_status,
       c.loss_date
FROM v_claims c
WHERE c.claim_number = 'YOUR_CLAIM_NUMBER';
```

Then fan out:

```sql
SELECT cc.claim_id, cc.contact_id, ct.contact_name, ct.contact_type
FROM v_claims_contacts cc
LEFT JOIN v_contacts ct
  ON ct.contact_id = cc.contact_id
WHERE cc.claim_id = 'YOUR_CLAIM_ID';
```

```sql
SELECT cp.claim_id, cp.claim_payment_id, cp.amount, cp.transaction_date
FROM v_claim_payments cp
WHERE cp.claim_id = 'YOUR_CLAIM_ID';
```

## Typical data shape differences

### CSV shape

The raw export usually looks like a normalized set of files:

- one row per claim in `claims.csv`
- separate rows in bridge/detail tables for related parties and payment events

This means a full claim reconstruction usually takes multiple files.

### API shape

The API often returns the claim as a nested object, then attaches sibling collections via relationship-specific API calls. This is typically much easier to read as a single object graph.

### SQL/report shape

The reporting layer usually collapses those relationship details into flattened views, making it easier to query and aggregate.

## Key fields to watch

Important claim fields include:

- `claim_id`
- `claim_number`
- `claim_status`
- `claim_type`
- `loss_date`
- `date_reported`
- `policy_number`
- `revision_id`
- `catastrophe_id`
- `loss_location_address_*`

Especially important for lineage:

- `revision_id` links the claim back to the policy term
- `claim_id` is the central join key for claim-detail tables
- `policy_number` ties the claim to the policy identity at a business level

## Common gotchas

### 1. The claim number is not enough for all joins

The claim number is the business key. The relationship tables usually use the internal `claim_id`.

### 2. A claim may have many contact rows

Do not assume one claim-to-contact row. A claim can have multiple associated parties.

### 3. Payment detail may be split across multiple tables

Payment header rows and payment item-detail rows often live in different tables or views.

### 4. Claim status and claim event dates are not always the same concept

Use status for lifecycle state and date fields for event timing.

## Recommended working model

For a claim workflow, the order is usually:

1. find the claim by `claim_number` or `claim_id`
2. confirm the claim master row
3. join to the related contact bridge table
4. join to the payment rows
5. if needed, join to item detail or peril/date tables

That sequence keeps the grain clear and avoids confusing the relationship layer with the base claim record.

## Claim quick reference

```text
claim_number -> claim_id -> claim detail / bridge tables
        -> revision_id -> policy term context
        -> policy_number -> policy record context
```

This is the most reliable mental model for claim-related work.
