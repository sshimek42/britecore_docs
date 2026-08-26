# Join Chains and Lineage Cheat Sheet

This page is the practical “how do I actually connect the data together?” guide.

The goal is to show the common relationship chains that recur across BriteCore policies, claims, revisions, contacts, and property data. These are the chains you can usually trust as a working model even when the raw export is fragmented.

## Core rules

1. Start from the best business key you already have.
2. Find the stable internal id that anchors the entity.
3. Use that id to fan out through the relationship or bridge tables.
4. Join from the bridge row to the master table for the related entity.
5. Keep the grain in mind: bridge rows multiply because they represent relationships, not copies of the main record.

## Policy and revision lineage

### Policy → revision → claim

This is one of the most important and most reusable chains.

```text
policy_number
  -> policy_id / revision_id
  -> revision_id
  -> claims
```

Typical SQL pattern:

```sql
SELECT r.revision_id, r.policy_number, c.claim_id, c.claim_number
FROM v_revisions r
LEFT JOIN v_claims c
  ON c.revision_id = r.revision_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

This is the basic version of the policy-to-claim lineage.

### Policy → revision → properties

```text
policy_number
  -> revision_id
  -> properties
```

Typical SQL pattern:

```sql
SELECT r.revision_id, r.policy_number, p.property_id, p.property_name
FROM v_revisions r
LEFT JOIN v_properties p
  ON p.revision_id = r.revision_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

### Policy → revision → contacts

```text
policy_number
  -> revision_id
  -> v_revisions_contacts
  -> v_contacts
```

Typical SQL pattern:

```sql
SELECT r.revision_id, r.policy_number, rc.contact_id, c.contact_name, c.roles
FROM v_revisions r
LEFT JOIN v_revisions_contacts rc
  ON r.revision_id = rc.revision_id
LEFT JOIN v_contacts c
  ON rc.contact_id = c.contact_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

This is a classic bridge-table pattern: one revision can link to multiple contacts, and the bridge table expands the result set accordingly.

## Claim lineage

### Claim → claim details

```text
claim_number
  -> claim_id
  -> claim detail tables
```

Typical SQL pattern:

```sql
SELECT c.claim_id, c.claim_number, c.revision_id, c.policy_number
FROM v_claims c
WHERE c.claim_number = 'YOUR_CLAIM_NUMBER';
```

Then fan out:

```sql
SELECT *
FROM v_claims_contacts
WHERE claim_id = 'YOUR_CLAIM_ID';
```

```sql
SELECT *
FROM v_claim_payments
WHERE claim_id = 'YOUR_CLAIM_ID';
```

```sql
SELECT *
FROM v_claim_items
WHERE claim_id = 'YOUR_CLAIM_ID';
```

### Claim → contacts

```text
claim_id
  -> v_claims_contacts
  -> v_contacts
```

Typical SQL pattern:

```sql
SELECT cc.claim_id, cc.contact_id, c.contact_name, c.contact_type
FROM v_claims_contacts cc
LEFT JOIN v_contacts c
  ON cc.contact_id = c.contact_id
WHERE cc.claim_id = 'YOUR_CLAIM_ID';
```

### Claim → payments

```text
claim_id
  -> v_claim_payments
  -> payment detail tables / item amount tables
```

Typical SQL pattern:

```sql
SELECT cp.claim_id, cp.claim_payment_id, cp.amount, cp.transaction_date
FROM v_claim_payments cp
WHERE cp.claim_id = 'YOUR_CLAIM_ID';
```

If payment item detail is needed:

```sql
SELECT cpi.claim_payment_id, cpi.claim_item_id, cpi.loss_paid, cpi.legal_paid
FROM v_claim_payments_item_amounts cpi
WHERE cpi.claim_payment_id = 'YOUR_PAYMENT_ID';
```

## Contact lineage

### Contact → addresses / phones / emails

```text
contact_id
  -> addresses
  -> phones
  -> emails
```

Typical SQL pattern:

```sql
SELECT c.contact_id, c.contact_name, a.address_line_1, a.address_city, a.address_state
FROM v_contacts c
LEFT JOIN v_addresses a
  ON a.contact_id = c.contact_id
WHERE c.contact_id = 'YOUR_CONTACT_ID';
```

```sql
SELECT c.contact_id, c.contact_name, p.phone, p.phone_type
FROM v_contacts c
LEFT JOIN v_phones p
  ON p.contact_id = c.contact_id
WHERE c.contact_id = 'YOUR_CONTACT_ID';
```

```sql
SELECT c.contact_id, c.contact_name, e.email, e.email_type
FROM v_contacts c
LEFT JOIN v_emails e
  ON e.contact_id = c.contact_id
WHERE c.contact_id = 'YOUR_CONTACT_ID';
```

### Contact → role and claim associations

```text
contact_id
  -> v_claims_contacts
  -> claim records
```

```text
contact_id
  -> v_revisions_contacts
  -> revision records
```

This is how a single contact can appear across many claims, roles, and revisions.

## Property and mortgagee lineage

### Revision → property → mortgagee

```text
revision_id
  -> properties
  -> mortgagees (via property + revision linkage)
```

Typical SQL pattern:

```sql
SELECT r.revision_id, r.policy_number, p.property_id, p.property_name, m.contact_id, m.name
FROM v_revisions r
LEFT JOIN v_properties p
  ON p.revision_id = r.revision_id
LEFT JOIN v_mortgagees m
  ON m.property_id = p.property_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

This is a good example of how the same business object can be reconstructed from multiple intermediate tables.

## Claim/payment/item drilldown

```text
claim_id
  -> claim payment rows
  -> claim payment item amount rows
```

This chain explains why payment detail is often split across more than one file or report view.

```sql
SELECT cp.claim_payment_id, cp.claim_id, cp.amount, cp.transaction_date,
       cpi.claim_item_id, cpi.loss_paid, cpi.legal_paid
FROM v_claim_payments cp
LEFT JOIN v_claim_payments_item_amounts cpi
  ON cpi.claim_payment_id = cp.claim_payment_id
WHERE cp.claim_id = 'YOUR_CLAIM_ID';
```

## Common CSV equivalent chains

The CSV exports follow the same chain concept, but the files are more fragmented.

### Example: claim reconstruction

```text
claims.csv
  -> claims_contacts.csv
  -> claims_payments.csv
  -> claims_payments_item_amounts.csv
```

### Example: policy reconstruction

```text
policies.csv
  -> policy_terms.csv
  -> revisions.csv
  -> properties.csv
  -> mortgagees.csv
  -> addresses.csv / phones.csv / emails.csv
```

The raw CSV chain has more steps because the export is intentionally split by concern.

## Safe interpretation of relationship chains

A relationship chain is usually only valid if you maintain the correct grain:

- claim master row = one claim
- claim-contact bridge row = one contact assignment on that claim
- claim payment row = one payment event
- claim payment item amount row = one payment item split

If the result set expands, that is usually because the relationship layer has more than one link row.

## Quick reference

### Base entity to child chain

```text
policy_number -> revision_id -> claims / properties / contacts
claim_number -> claim_id -> claim contacts / payments / items
contact_id -> addresses / phones / emails / roles
```

### Bridge-table pattern

```text
parent_entity_id -> bridge_table -> related_entity_id -> related_entity_master
```

### Reporting-friendly pattern

```text
master table -> relationship bridge -> detail table
```

## Most important gotchas

### 1. Do not assume one-to-one joins everywhere

A single claim or revision can map to multiple contacts, payments, or properties.

### 2. Do not treat bridge rows as duplicates

They often represent relationship events, not duplicate master records.

### 3. Do not expect the same file layout in CSV and API

The CSV layer is more fragmented; the API and report layer are better structured for direct lineage work.

### 4. Keep the business key and internal id separate

The human-facing number (`policy_number`, `claim_number`) usually leads to an internal id (`revision_id`, `claim_id`) that is what the bridge and detail tables use.

## Short version

The ecosystem is easiest to reason about when you think in reverse:

- start at the human-facing key,
- resolve to the internal id,
- fan out through the bridge tables,
- then join to the related master tables.

That is the core lineage pattern that holds across policies, claims, contacts, properties, payments, and related records.
