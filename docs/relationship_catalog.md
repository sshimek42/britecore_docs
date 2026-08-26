# Relationship Catalog

This page is the cross-reference for the most common BriteCore relationship patterns.

The purpose is to make the ecosystem easier to reason about by organizing the common parent-child and bridge relationships in one place.

## Relationship model at a glance

The general pattern is:

- master entity table or CSV file holds the primary object
- bridge table holds the relationship rows
- detail table holds child or attached records
- report views flatten the relationship for readable downstream reporting

## Policy-related relationships

### Policy → revision

- `policies.csv` / `v_policies` or policy master identity
- `policy_terms.csv` / `v_revisions` or term-level lifecycle data
- often tied by `policyId` and `policy_term_id` / `revision_id`

Typical relationship:

```text
policy_id -> many revision rows
revision_id -> many claim rows
```

### Revision → claim

- `v_revisions.revision_id` = join key to `v_claims.revision_id`
- `claims.csv` contains the claim master rows
- relationship is usually one-to-many: one revision can have many claims

### Revision → property

- `revisions.csv` / `v_revisions` to `properties.csv` / `v_properties`
- join on `revisionId` or `revision_id`

### Revision → contact

- `v_revisions_contacts` is the bridge table
- join `revision_id` to `contact_id`

### Revision → agency

- `v_revisions_agencies` or similar agency relationship views
- join `revision_id` to `agency_id` or associated contact id

## Claim-related relationships

### Claim → contact

- `claims_contacts.csv` / `v_claims_contacts`
- join `claimId` / `claim_id` to `contactId` / `contact_id`

### Claim → payment

- `claims_payments.csv` / `v_claim_payments`
- join `claimId` / `claim_id`

### Claim → payment item detail

- `claims_payments_item_amounts.csv` / `v_claim_payments_item_amounts`
- join `claimPaymentId` or corresponding payment id

### Claim → item

- `claim_items.csv` / `v_claim_items`
- join `claimId` / `claim_id`

### Claim → peril / dates / disputes / exposures

- `claims_perils.csv`
- `claims_dates.csv`
- `claims_disputes.csv`
- `claim_exposures.csv`
- `claim_properties.csv`

These are all detail or bridge tables keyed to the claim id.

## Contact-related relationships

### Contact → address

- `addresses.csv` / `v_addresses`
- join on `contactId` / `contact_id`

### Contact → phone

- `phones.csv` / `v_phones`
- join on `contactId` / `contact_id`

### Contact → email

- `emails.csv` / `v_emails`
- join on `contactId` / `contact_id`

### Contact → role or association

- may live in `v_roles` or in denormalized text fields in `v_contacts`
- often multiple roles per contact across multiple claims or revisions

## Property-related relationships

### Property → revision

- join `properties.csv` or `v_properties.revision_id` to `v_revisions.revision_id`

### Property → mortgagee

- `mortgagees.csv` / `v_mortgagees`
- join on `propertyId` or `revisionId`

### Property → item

- `property_items.csv` / `v_property_items`
- join to property id or revision id depending on the reporting view

## Financial and billing relationships

### Policy/claim to payment

- policy payments and claim payments are often separate domains
- join via `policyId`, `revisionId`, or `claimId` depending on the detail table

### Commission or agency relationships

- agency and commission records may attach to contacts, revisions, or policy terms, depending on the report
- use the revision or contact id as the anchor for lineage

## Typical bridge-pattern examples

### Example 1: claim to contact

```text
claim_id -> v_claims_contacts -> contact_id -> v_contacts
```

### Example 2: revision to contact

```text
revision_id -> v_revisions_contacts -> contact_id -> v_contacts
```

### Example 3: policy revision to property

```text
revision_id -> v_properties -> property_id
```

### Example 4: claim to payment item detail

```text
claim_id -> v_claim_payments -> claim_payment_id -> v_claim_payments_item_amounts
```

## Relationship catalog summary

| Parent | Relationship table | Child | Join key |
|---|---|---|---|
| Policy / revision | `v_claims` | Claim | `revision_id` |
| Revision | `v_revisions_contacts` | Contact | `revision_id` + `contact_id` |
| Claim | `v_claims_contacts` | Contact | `claim_id` + `contact_id` |
| Claim | `v_claim_payments` | Payment | `claim_id` |
| Claim payment | `v_claim_payments_item_amounts` | Item detail | `claim_payment_id` |
| Revision | `v_properties` | Property | `revision_id` |
| Property | `v_mortgagees` | Mortgagee contact | `property_id` |
| Contact | `v_addresses` | Address | `contact_id` |
| Contact | `v_phones` | Phone | `contact_id` |
| Contact | `v_emails` | Email | `contact_id` |

## Querying advice

When you do not know which table is authoritative for a relationship:

1. identify the master object id
2. look for the bridge table at the same domain level
3. join the bridge to the master table for the related object
4. verify whether the result set is one row per entity or one row per relationship

## Important rule

If the same parent id repeats multiple times in a result set, do not assume the data is duplicated. It may be the correct relationship grain.

This is especially common in bridge-heavy tables like claim-contact, revision-contact, payment detail, and property-interest tables.
