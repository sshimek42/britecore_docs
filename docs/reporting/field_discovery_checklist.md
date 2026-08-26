# Field Discovery Checklist

Use this checklist when trying to identify the correct report table, view, or join path for a business question.

## 1. Define the business object

Clarify which object the question is really about:

- policy
- claim
- insured
- contact
- premium
- revision
- item or property
- agency or role

If the object is ambiguous, the analytics work will usually drift into unnecessary joins.

## 2. Define the metric

Document the metric or result you actually need, such as:

- today’s in-force policies
- claim count by peril
- premium by policy type
- revision history by policy
- insured contact count by relationship type

Good metric definitions usually reveal the right fact table immediately.

## 3. Define the grain

Write the target grain in one sentence.

Examples:

- one row per active policy
- one row per claim
- one row per premium term
- one row per revision per policy

If this is not clear, stop and define it before writing joins.

## 4. Identify the likely fact source

Look for the dataset that best matches the business event or metric.

Common starting points:

- `m_*` tables for performance or summary-style reporting
- `v_claims` and related claim views for claim work
- `m_inforce_policies` for active policy questions
- `m_premium_*` tables for premium logic

## 5. Discover the logical catalog

Use `v_logical_catalog` as the primary discovery source.

Check for:

- candidate fact views
- candidate dimension views
- likely join keys
- field descriptions and semantics

This usually reduces wasted time more than any other single step.

## 6. Check dimension candidates

Once you have the anchor fact, list the dimension views that provide context, such as:

- policy type
- insured
- contact
- property
- agency
- date or status dimensions

Do not add them all at once. Add only the ones required to answer the report.

## 7. Validate join logic

Before finalizing the SQL, confirm:

- the join key is valid
- the relationship is one-to-one or one-to-many as expected
- the join is not multiplying rows unexpectedly
- aggregate logic is applied if the relationship is many-to-one

## 8. Validate against raw exports when needed

Use `britecore-csv-loader` when:

- the logical view relationships are unclear
- the grain seems unstable
- the report is based on export data or revision lineage
- the final output should match raw export assumptions

## 9. Confirm the final narrative

A good reporting explanation should be describable in one paragraph:

- what the fact table is
- what the anchor object is
- which dimensions were joined
- what the row grain is
- what the final output shows

## 10. Final quality check

Before considering the work complete, verify:

- the table or view choice matches the business object
- the grain is explicit
- join fan-out is checked
- raw-export validation was done where required
- the logic is explainable to another analyst

## Example review prompt

```text
What is the business object?
What is the metric?
What is the expected row grain?
What is the fact table?
Which dimensions are required?
What is the likely join key?
Is there any many-to-many relationship?
Does the result need raw export validation?
```

## Recommended references

- [Reporting Logical Layer](logical_layer.md)
- [Join Starters](join_starters.md)
- [Decision Matrix](../decision_matrix.md)
- [SQL Reporting Cookbook](sql_reporting_cookbook.md)
