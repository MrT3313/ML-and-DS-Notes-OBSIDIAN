---
note_kind: concept
aliases:
  - HTAP
  - hybrid transactional/analytical processing
  - hybrid transactional-analytical processing
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

Some systems offer hybrid transactional/analytical processing, which aims to enable OLTP and analytics in a single system without requiring [[Extract-Transform-Load|ETL]] from one system into another. The pitch is that an aggregate can be computed over data that is current to the last commit, because there is no copy step and therefore no lag between the two halves.

Gartner created the term in early 2014, in a report on breaking the wall between transaction processing and analytics, and vendors adopted it from there rather than the other way round.

What is on offer is usually less of a merger than the name suggests. These systems often still have the same operational and analytical engines inside them, with a shared interface hidden in front. TiDB is the clearest published case: row-oriented storage in TiKV serves the transactional side, column-oriented storage in TiFlash serves the analytical side, the columnar replicas are kept in step asynchronously through Raft, and the query planner picks between the two per query on cost. The split did not go away. It moved below the SQL interface and became somebody else's operational problem instead of yours, which is a real benefit and a smaller one than "a single system" implies.

## Formal statement

**Not further quantitative at this depth.**

HTAP is a product category and an architectural goal rather than a measurable property, so there is no quantity attached to the term itself.

## Where it is used

It is the attempt to merge [[Online Transaction Processing]] and [[Online Analytical Processing]], the two workloads the rest of this folder keeps apart, and its architecture is best read as a comment on why they were kept apart in the first place.

It does not replace a [[Data Warehouse]], and the reasons are the ones that motivated the warehouse to begin with. A warehouse integrates data from many operational systems into one schema, so it answers questions no single operational database can; it keeps history that an operational store overwrites; and it isolates expensive scans from the traffic users are waiting on. Merging the two workloads inside one database addresses none of those three.
