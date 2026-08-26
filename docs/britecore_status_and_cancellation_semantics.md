# BriteCore Status and Cancellation Semantics

This page captures a common cross-project source-of-truth fact: BriteCore status and cancellation semantics are not the same as MIPS normalized status semantics.

## The core distinction

In BriteCore, policy status and cancellation reason are stored in separate fields.

Key fields observed in the warehouse and source docs:

- `policyStatus`
- `policyCancellationReason`

These are important because they are not just a single coded status. They often represent:

- current policy state (for example Active, Canceled, Expired)
- a descriptive reason for termination or pending cancellation

## BriteCore pattern

Typical BriteCore behavior:

- `policyStatus` = text-based lifecycle state
- `policyCancellationReason` = descriptive text such as “Company Request” or “Non-Payment of Premium”

This means the raw source value is not a standardized MIPS code set. It is descriptive and often more human-readable than a reporting code.

## MIPS pattern

MIPS generally uses a different normalized model, such as:

- an active/cancelled indicator
- a standardized reason code family
- a separate policy status code that may be more compact than the BriteCore descriptive text

The practical takeaway is that downstream reporting should not collapse BriteCore and MIPS into the same status vocabulary unless the business requirement explicitly asks for a standardized cross-source interpretation.

## Warehouse interpretation

The warehouse docs note that BriteCore raw values are preserved in landing tables and the reporting layer can normalize later. This is the right pattern:

- preserve raw source semantics in `raw.*`
- derive normalized view values in `core.*` or reporting layers when necessary
- keep descriptive cancellation reasons available for audit and business reporting

## Why this matters

This issue repeatedly causes confusion because a team can look at a policy and see:

- BriteCore says “Canceled” with descriptive reason text
- MIPS says a different standardized status/reason pair

Those are not necessarily the same business view. The correct interpretation depends on the layer and the reporting question.

## Best practice

For cross-source reporting or reconciliation:

1. preserve the raw source text in the landed record
2. normalize to a reporting-friendly status where needed
3. keep reason text separate from the status code
4. do not overwrite the descriptive cancellation reason with a normalized code unless the reporting deliverable requires it

## Recommended guidance for docs and reporting

When documenting or building a report, explicitly distinguish:

- raw BriteCore status text
- raw BriteCore cancellation reason text
- normalized reporting status
- cross-source standardized status mapping

This avoids the common bug where a raw descriptive reason is incorrectly treated as a standard code.

## Reference concept

In other words:

- status tells you the lifecycle state
- cancellation reason tells you why the lifecycle changed
- normalization tells you how downstream reporting interprets those facts

Those are related, but not interchangeable.
