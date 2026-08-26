# SQL Reporting Cookbook

This page captures practical SQL patterns for BriteCore-style reporting work. It is meant to be a hands-on companion to the higher-level reporting guidance in this docs set.

## Reporting principles

Before writing SQL, keep these principles in mind:

- define the business question and target grain first
- anchor on the fact table that matches the subject
- join only the necessary dimension or bridge views
- check duplicate rows and fan-out after each enrichment step
- validate assumptions against the logical catalog and, when necessary, raw export relationships

## 1. Start with a valid anchor query

A strong reporting query usually begins with a clear anchor table and a precise filter.

```sql
SELECT
    policy_id,
    revision_id,
    effective_date,
    policy_status
FROM m_inforce_policies
WHERE effective_date >= DATEADD(month, -12, CAST(GETDATE() AS date));
```

This gives you a clean base dataset before adding context.

## 2. Use `v_logical_catalog` as the discovery layer

When you are unsure which table to start from, inspect the catalog first.

```sql
SELECT TOP 200
    table_name,
    schema_name,
    logical_type,
    description
FROM v_logical_catalog
WHERE table_name LIKE '%claim%'
   OR table_name LIKE '%policy%'
ORDER BY table_name;
```

This is the easiest way to find:

- likely fact tables
- likely bridge and lookup tables
- candidate join keys
- field descriptions

## 3. Check the grain before and after joins

This is the most common mistake in reporting work: joining views that are valid individually but explode row counts when combined.

```sql
SELECT
    COUNT(*) AS row_count,
    COUNT(DISTINCT policy_id) AS distinct_policies
FROM m_inforce_policies;
```

Then compare after a join:

```sql
SELECT
    COUNT(*) AS row_count,
    COUNT(DISTINCT a.policy_id) AS distinct_policies
FROM m_inforce_policies a
LEFT JOIN v_policy_types b
    ON a.policy_type_id = b.policy_type_id;
```

If the row count jumps but distinct policy count stays flat, the join is probably introducing duplicates or a one-to-many relationship that should be reduced or aggregated.

## 4. Use bridge and lookup views intentionally

Bridge and lookup tables are useful, but they can also be the reason a query starts multiplying records.

```sql
SELECT
    c.claim_id,
    c.claim_number,
    d.claim_date,
    p.peril_name
FROM v_claims c
LEFT JOIN v_claims_dates d
    ON c.claim_id = d.claim_id
LEFT JOIN v_claims_perils p
    ON c.claim_id = p.claim_id;
```

This pattern is valid when the business question is truly claim-centric; it becomes risky when the join is being used for a single fact without aggregation.

## 5. Pattern for dimension enrichment

A common and safe approach is to enrich the anchor query with the minimum dimension set needed for the report.

```sql
SELECT
    a.policy_id,
    a.revision_id,
    b.policy_type_name,
    i.insured_name,
    p.property_city
FROM m_inforce_policies a
LEFT JOIN v_policy_types b
    ON a.policy_type_id = b.policy_type_id
LEFT JOIN v_insureds i
    ON a.insured_id = i.insured_id
LEFT JOIN v_properties p
    ON a.property_id = p.property_id;
```

This pattern is usually easier to reason about and maintain than large multi-table joins from the start.

## 6. Reporting by domain

### Policy reporting

```sql
SELECT
    p.policy_id,
    p.policy_number,
    p.effective_date,
    t.policy_type_name,
    i.insured_name
FROM m_inforce_policies p
LEFT JOIN v_policy_types t
    ON p.policy_type_id = t.policy_type_id
LEFT JOIN v_insureds i
    ON p.insured_id = i.insured_id
WHERE p.effective_date >= DATEADD(year, -1, CAST(GETDATE() AS date));
```

### Claims reporting

```sql
SELECT
    c.claim_id,
    c.claim_number,
    cd.claim_date,
    cp.peril_name,
    cp.loss_amount
FROM v_claims c
LEFT JOIN v_claims_dates cd
    ON c.claim_id = cd.claim_id
LEFT JOIN v_claims_perils cp
    ON c.claim_id = cp.claim_id
WHERE cd.claim_date >= DATEADD(year, -1, CAST(GETDATE() AS date));
```

### Premium reporting

```sql
SELECT
    pt.policy_id,
    pt.premium_term_id,
    pt.premium_amount,
    pt.effective_date,
    vt.policy_type_name
FROM m_premium_terms pt
LEFT JOIN v_policy_types vt
    ON pt.policy_type_id = vt.policy_type_id
WHERE pt.effective_date >= DATEADD(year, -1, CAST(GETDATE() AS date));
```

## 7. Aggregate before you join to large context tables

This is a helpful optimization and a good logic pattern when the report is centered on a fact table.

```sql
WITH claim_summary AS (
    SELECT
        claim_id,
        SUM(loss_amount) AS total_loss_amount
    FROM v_claim_payments
    GROUP BY claim_id
)
SELECT
    c.claim_id,
    c.claim_number,
    cs.total_loss_amount,
    cd.claim_date
FROM v_claims c
LEFT JOIN claim_summary cs
    ON c.claim_id = cs.claim_id
LEFT JOIN v_claims_dates cd
    ON c.claim_id = cd.claim_id;
```

This keeps the summary logic in one place and reduces the chance of a later join inflating the dataset.

## 8. Use window functions for ranked or period-based logic

```sql
SELECT
    claim_id,
    claim_number,
    claim_date,
    ROW_NUMBER() OVER (PARTITION BY claim_id ORDER BY claim_date DESC) AS latest_claim_event_rank
FROM v_claims_dates;
```

This is valuable when working with event tables, revision histories, or multiple claim-related dates.

## 9. Audit and lineage patterns

For audit work, keep the anchor object stable and then inspect adjacent revision history.

```sql
SELECT
    r.revision_id,
    r.revision_number,
    r.updated_at,
    ri.item_type,
    ri.item_id
FROM v_revisions r
LEFT JOIN v_revision_items ri
    ON r.revision_id = ri.revision_id
WHERE r.updated_at >= DATEADD(month, -3, CAST(GETDATE() AS date));
```

This is useful when reasoning through policy changes, claim adjustments, or revision-driven reporting.

## 10. Validation queries for report quality

### Check for duplicate keys

```sql
SELECT
    policy_id,
    COUNT(*) AS row_count
FROM m_inforce_policies
GROUP BY policy_id
HAVING COUNT(*) > 1;
```

### Check join fan-out

```sql
SELECT
    a.policy_id,
    COUNT(*) AS joined_rows
FROM m_inforce_policies a
LEFT JOIN v_revisions r
    ON a.revision_id = r.revision_id
GROUP BY a.policy_id
HAVING COUNT(*) > 1;
```

### Check upstream description coverage

```sql
SELECT
    COUNT(*) AS rows_missing_policy_type
FROM m_inforce_policies a
LEFT JOIN v_policy_types t
    ON a.policy_type_id = t.policy_type_id
WHERE t.policy_type_id IS NULL;
```

These checks help identify whether the problem is in the data, the join key, or the chosen reporting pattern.

## 11. When to switch to raw CSV validation

Use raw CSV validation when:

- the logical layer is still ambiguous
- the rows look wrong even after reasonable joins
- the business rule depends on revision or export lineage
- you need to confirm whether the logical layer matches the raw exported data model

This is especially relevant when working with revisions, claims, policies, and relationship-heavy datasets.

## 12. Reporting quality checklist

Before moving a report to production or shared use, confirm:

- the business question is explicitly stated
- the anchor object is clear and stable
- the row grain is validated
- joins are limited to required enrichment
- duplicates or fan-out are controlled
- the final query is understandable to another analyst
- assumptions are documented for future maintenance

## Reference set

- [Reporting Logical Layer](logical_layer.md)
- [Join Starters](join_starters.md)
- [Reporting Playbook](reporting_playbook.md)
- [Source Registry](../source_registry.md)
