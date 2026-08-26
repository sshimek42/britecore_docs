# Property and Risk Domain Guide

This page is the practical guide for working with property or risk records in the BriteCore ecosystem.

## What a property is

A property is the insured location or risk record associated with a policy revision. It can also be joined to mortgagee or interest records and to any claim or policy detail that was tied to the property at the time of coverage or loss.

## Core identity

The key identifiers are usually:

- `property_id` — internal risk/property id
- `revision_id` — links the property back to the policy term
- `property_name` — readable property label
- `risk_*` fields in CSV exports

The common pattern is:

```text
policy_number -> revision_id -> property_id -> mortgagees / property items
```

## Best starting tables

### CSV

- `properties.csv` — property master rows
- `mortgagees.csv` — mortgagee or interest detail rows
- `property_items.csv` — item-level risk or property detail rows
- `revision_items.csv` — revision-level item records

### API

- `policies.retrieve_risks(...)`
- `policies.retrieve_risk_details(risk_id=...)`
- policy property/risk endpoints used during policy revision and underwriting workflows

### SQL/report

- `v_properties` — property master view
- `v_mortgagees` — mortgagee relationship details
- `v_property_items` — property item detail rows
- `v_revisions` — policy-term anchor for the property

## Typical property trace pattern

### Find the property rows on a revision

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

### Pull mortgagee or interest details

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

### Pull item detail on that property or revision

```sql
SELECT pi.property_id,
       pi.property_item_id,
       pi.item_name,
       pi.coverage_name
FROM v_property_items pi
WHERE pi.property_id = 'YOUR_PROPERTY_ID';
```

## CSV shape

The raw CSV export is often more fragmented than the API or report layer. The property master row is usually in `properties.csv`, while mortgagees or property items are split into separate files.

This is normal and means a full property reconstruction often takes multiple files and joins.

## API shape

The API is closer to a nested object pattern, with property fields and related child records returned in a structured object or via dedicated risk endpoints.

This is useful when you want the live risk object and its associated metadata without reassembling a set of CSV files.

## Report shape

The reporting layer usually flattens the property and its key attributes into a denormalized row with a stable `property_id` and `revision_id` for easier join logic.

This is often the easiest place to answer a business question when the required fields are already present in the view.

## Key fields to watch

Important property fields include:

- `property_id`
- `revision_id`
- `property_name`
- `risk_address_line_1`
- `risk_city`
- `risk_state`
- `risk_zip`
- `risk_county`
- `risk_latitude`
- `risk_longitude`
- inspection or valuation fields as applicable

## Common gotchas

### 1. Property data is often tied to revision, not just policy

If you are joining from a policy number, you usually need the revision first.

### 2. Mortgagee records are often not on the master property row

They are often separate contact or relationship records tied by property id or revision id.

### 3. Property detail may be split across item tables

Do not assume a single property file contains all property-level detail.

## Recommended working model

For property work, the reliable pattern is:

1. find the policy or revision
2. resolve the `revision_id`
3. join to property rows by `revision_id`
4. fan out to item detail or mortgagee records by `property_id`

This preserves the correct grain and avoids mixing revision-level data with property-level data.

## Property quick reference

```text
policy_number -> revision_id -> property_id -> mortgagees / items / location detail
```

This is the best default understanding of property lineage in the BriteCore ecosystem.
