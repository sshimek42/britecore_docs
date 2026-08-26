# Query Quickstarts

This page turns the earlier mapping rules into concrete, reusable query patterns.

The idea is simple: start with the business key, resolve it to the internal id, then fan out through the relationship tables or API endpoints.

## 1. Policy trace quickstart

### Find a policy by number and show its claim history

```sql
SELECT r.revision_id,
       r.policy_number,
       r.policy_status,
       r.term_effective_date,
       r.term_expiration_date,
       c.claim_id,
       c.claim_number,
       c.claim_status
FROM v_revisions r
LEFT JOIN v_claims c
  ON c.revision_id = r.revision_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER'
ORDER BY r.revision_id, c.claim_number;
```

### Find the policy contacts tied to the current revision

```sql
SELECT r.revision_id,
       r.policy_number,
       rc.contact_id,
       c.contact_name,
       c.contact_type,
       c.roles
FROM v_revisions r
LEFT JOIN v_revisions_contacts rc
  ON rc.revision_id = r.revision_id
LEFT JOIN v_contacts c
  ON c.contact_id = rc.contact_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

### API equivalent

```python
from britecore_sdk.api.api_calls.v2 import policies

policy = policies.retrieve_policy(policy_number='YOUR_POLICY_NUMBER')
policy_ids = policies.retrieve_policy_ids(policy_number='YOUR_POLICY_NUMBER')
# policy_ids often gives revision_id + other linkage ids
```

## 2. Claim trace quickstart

### Find the claim and attach the linked contacts

```sql
SELECT c.claim_id,
       c.claim_number,
       c.policy_number,
       c.revision_id,
       c.claim_status,
       cc.contact_id,
       ct.contact_name,
       ct.contact_type,
       ct.roles
FROM v_claims c
LEFT JOIN v_claims_contacts cc
  ON cc.claim_id = c.claim_id
LEFT JOIN v_contacts ct
  ON ct.contact_id = cc.contact_id
WHERE c.claim_number = 'YOUR_CLAIM_NUMBER';
```

### Find the claim payments

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

### API equivalent

```python
from britecore_sdk.api.api_calls.v2 import claims

claim = claims.get_claim(claim_number='YOUR_CLAIM_NUMBER')
claim_contacts = claims.get_claim_contacts(claim_id=claim['id'])
claim_payments = claims.get_claim_payments(claim_id=claim['id'])
```

## 3. Contact trace quickstart

### Pull the contact master row and attached addresses, phones, and emails

```sql
SELECT c.contact_id,
       c.contact_name,
       c.contact_type,
       c.roles,
       a.address_line_1,
       a.address_city,
       a.address_state,
       a.address_zip,
       p.phone,
       p.phone_type,
       e.email,
       e.email_type
FROM v_contacts c
LEFT JOIN v_addresses a
  ON a.contact_id = c.contact_id
LEFT JOIN v_phones p
  ON p.contact_id = c.contact_id
LEFT JOIN v_emails e
  ON e.contact_id = c.contact_id
WHERE c.contact_id = 'YOUR_CONTACT_ID';
```

### API equivalent

```python
from britecore_sdk.api.api_calls.v2 import contacts

contact = contacts.get_contact(contact_id='YOUR_CONTACT_ID')
# nested arrays: contact['addresses'], contact['phones'], contact['emails'], contact['roles']
```

## 4. Property and mortgagee trace quickstart

### Find property rows tied to a policy revision

```sql
SELECT r.revision_id,
       r.policy_number,
       p.property_id,
       p.property_name,
       p.risk_city,
       p.risk_state,
       p.risk_zip
FROM v_revisions r
LEFT JOIN v_properties p
  ON p.revision_id = r.revision_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

### Find mortgagees linked to those properties

```sql
SELECT p.property_id,
       p.property_name,
       m.contact_id,
       m.name,
       m.address_line_1,
       m.address_city,
       m.address_state,
       m.address_zip
FROM v_properties p
LEFT JOIN v_mortgagees m
  ON m.property_id = p.property_id
WHERE p.revision_id = 'YOUR_REVISION_ID';
```

## 5. Payment and item detail quickstart

### Claim payment drilldown

```sql
SELECT cp.claim_payment_id,
       cp.claim_id,
       cp.amount,
       cp.transaction_date,
       cpi.claim_item_id,
       cpi.line_item_name,
       cpi.loss_paid,
       cpi.legal_paid
FROM v_claim_payments cp
LEFT JOIN v_claim_payments_item_amounts cpi
  ON cpi.claim_payment_id = cp.claim_payment_id
WHERE cp.claim_id = 'YOUR_CLAIM_ID';
```

This pattern shows how a claim payment can expand into multiple item-detail rows.

## 6. CSV reconstruction quickstart

When starting from CSV exports, use the base entity + child/bridge chain.

### Example: reconstruct a claim

```text
claims.csv
  -> filter to claim_id or claimNumber
  -> join claims_contacts.csv on claimId
  -> join claims_payments.csv on claimId
  -> join claims_payments_item_amounts.csv on claimPaymentId
```

### Example: reconstruct a policy

```text
policies.csv
  -> join policy_terms.csv on policyId
  -> join revisions.csv on revisionId or policyTermId
  -> join properties.csv on revisionId
  -> join mortgagees.csv on propertyId + revisionId
  -> join addresses.csv / phones.csv / emails.csv on contactId
```

This is the raw-export version of the same lineage logic used in SQL and API work.

## 7. Safe query pattern for bridge-heavy data

When a business question is about a parent record, do this:

```sql
SELECT parent_key,
       bridge_key,
       related_key,
       related_name
FROM parent_bridge_table b
LEFT JOIN related_master_table r
  ON r.related_key = b.related_key
WHERE b.parent_key = 'YOUR_PARENT_KEY';
```

This is the core pattern for relationship-heavy data and should be your default approach when a result set seems unexpectedly large.

## 8. Common interpretation rule

If the result set expands unexpectedly:

- check whether you are on a bridge table
- check whether the parent is linked to multiple children
- check whether a relationship record appears once per association
- check whether you are accidentally joining a current-state table to a historical one

## Short version

The strongest general pattern is:

- start with a human-facing key
- resolve to the internal id
- fan out via relationship tables or API endpoints
- join back to the master entity for the fields you need

That pattern works across policy, claim, contact, property, and payment data and is the best practical default for BriteCore work.
