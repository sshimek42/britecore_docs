# Access Mode Decision Matrix

This page is the short, practical guide for choosing how to work with BriteCore data.

## The short version

Use the access mode that best matches your need for:

- source fidelity
- freshness
- structure
- reporting readiness
- offline access

## Decision matrix

| Need | Best access mode | Why |
|---|---|---|
| Point-in-time snapshot and offline work | CSV | Exported files are stored snapshots and are easy to keep locally |
| Real-time or near-real-time object lookup | API | Returns a live structured entity payload with nested objects |
| Fast reporting or new ad hoc report logic | SQL / report views | The data is already shaped into a query-friendly reporting layer |
| Quick business question with minimal reconstruction | SQL / report views | In many cases, the website query layer is easier than stitching together many CSVs or walking multiple API calls |
| Entity lineage and raw-source validation | CSV | Shows the base entity and raw fields before formatting/aggregation |
| Entity detail with nested structure | API | Gives a more object-like representation than flat raw files |
| Report-building with minimal ETL | SQL / report views | Pulls from preformatted reporting views and aggregates |

## Reading the tradeoff correctly

### CSV

Best for:

- offline access
- point-in-time snapshots
- raw field inspection
- source-truth validation
- preserved extraction data

Tradeoffs:

- more fragmented and less structured than API objects
- not real-time
- often requires joins across multiple files or IDs

### API

Best for:

- live object queries
- nested structure with child collections
- direct entity retrieval without needing to reconstruct every join manually

Tradeoffs:

- not as raw as CSV exports
- not always as report-ready as SQL/report views
- requires live connectivity and object traversal

### SQL / report views

Best for:

- preformatted reporting
- repeated business reporting
- fast report creation from existing views
- less custom shaping work than raw CSV or API exploration

Tradeoffs:

- less source fidelity than CSV
- not as naturally object-structured as the API
- can hide some raw-system nuance if used without understanding the source layer

## When to choose each one

### Choose CSV when:

- you need a saved historical extraction
- you are validating raw-source field names or row grain
- you need a low-level, source-faithful record set
- you want offline investigation without depending on live services

### Choose API when:

- you need the current live entity state
- the object model is more useful than a flat export table
- you want nested collections like addresses, phones, or roles without building them from several CSVs

### Choose SQL / report views when:

- you are building or modifying a report
- you want the fastest route to aggregated business logic
- you need a query-friendly layer already designed for reporting

## Important conceptual rule

These are not mutually exclusive stages. They are different access patterns for the same underlying business data.

A team may choose:

- CSV only
- API only
- SQL only
- or mixed mode depending on the question and environment

The key is not whether one is “more correct,” but which mode best matches the task and the available data access.
