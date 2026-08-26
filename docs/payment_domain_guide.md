# Payment and Billing Domain Guide

This page is the practical guide for understanding payment and billing data across BriteCore CSV, API, and report-layer sources.

## What payment data looks like

Payment data is usually split into:

- payment header records
- payment item or line-detail records
- policy billing records or account history records
- claim payment records and claim payment detail

This means payment data is often a key example of a relationship-heavy domain where the master record is split across multiple files or views.

## Core identity

The key ids depend on the domain:

- `claim_id` for claim payment linkage
- `policy_id` or `revision_id` for policy billing logic
- `claim_payment_id` for claim payment detail
- payment id or transaction id depending on the export

Typical patterns:

```text
claim_id -> claim payment rows -> item amounts
policy_id / revision_id -> billing and payment context
```

## Best starting tables

### CSV

- `claim_payments.csv` or equivalent claim-payment export
- `claims_payments_item_amounts.csv` or equivalent payment detail split
- `policy_payments.csv` or policy billing files
- `account_history.csv` or accounting history records

### API

- claim payment endpoints under the claim domain
- policy billing endpoints under the policy domain
- general payment and accounting wrappers when present

### SQL/report

- `v_claim_payments`
- `v_claim_payments_item_amounts`
- `v_payments` or billing-related view(s)
- policy billing and accounting views as applicable

## Typical claim payment trace pattern

### Find the payment rows on a claim

```sql
SELECT cp.claim_id,
       cp.claim_payment_id,
       cp.amount,
       cp.transaction_date,
       cp.voided,
       cp.historical
FROM v_claim_payments cp
WHERE cp.claim_id = 'YOUR_CLAIM_ID';
```

### Drill into item detail from each payment

```sql
SELECT cpi.claim_payment_id,
       cpi.claim_item_id,
       cpi.line_item_name,
       cpi.loss_paid,
       cpi.legal_paid,
       cpi.adjusting_paid
FROM v_claim_payments_item_amounts cpi
WHERE cpi.claim_payment_id = 'YOUR_PAYMENT_ID';
```

## Policy billing pattern

Policy billing is usually tied to the policy or revision lifecycle rather than claim detail.

Typical pattern:

```sql
SELECT p.policy_id,
       p.policy_number,
       r.revision_id,
       r.billing_schedule,
       bp.amount,
       bp.transaction_date
FROM v_revisions r
LEFT JOIN v_policy_payments bp
  ON bp.revision_id = r.revision_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

## CSV reconstruction pattern

A payment reconstruction usually requires multiple files, such as:

```text
claims_payments.csv
  -> claims_payments_item_amounts.csv
```

or:

```text
policy_payments.csv
  -> account_history.csv
```

This is the raw exported form of a relationship-heavy billing domain.

## API shape

The API typically gives the payment or billing object in a more nested and structured form, while the report layer flattens the details for analysis.

This is often easier for a live operational workflow, but not always as easy as SQL for aggregated reporting.

## Report shape

The reporting layer usually flattens common payment fields so they are easier to summarize and compare, even when the source data is split across multiple files or endpoints.

## Key fields to watch

Important payment fields include:

- `claim_id`
- `policy_id`
- `revision_id`
- `claim_payment_id`
- `amount`
- `transaction_date`
- `voided`
- `voided_date`
- `historical`
- classification fields or line item labels

## Common gotchas

### 1. Payment headers and item detail are often separate tables

A payment row may not contain all of the detail you need; the item detail may live elsewhere.

### 2. Policy billing and claim payment are different domains

Do not assume the same payment table covers both policy premium and claim payout logic.

### 3. Historical and voided rows can make counts misleading

Use historical or voided flags intentionally when answering business questions.

## Recommended working model

For payment work, the best pattern is:

1. start with the claim or policy context
2. resolve the relevant payment rows
3. drill into item or billing detail using the payment id
4. decide whether you need the operational payment object or the report-level summary

That sequence avoids conflating header-level payment rows with line-item or accounting detail.

## Payment quick reference

```text
claim_id -> claim payment rows -> item amounts
policy_id / revision_id -> billing rows -> account history
```

This is the most reliable default understanding of payment and billing lineage in the BriteCore ecosystem.
