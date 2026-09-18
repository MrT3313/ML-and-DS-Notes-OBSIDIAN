---
note_kind: concept
aliases:
  - point query
  - point queries
  - point lookup
  - point lookups
  - key lookup
up: "[[Online Transaction Processing]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

A point query looks up a small number of records by a key, matching one exact key value rather than sweeping a range. It is the read an operational system does most: fetch the row for this user, this order, this session, and then act on it.

The key does not have to be the primary key. An exact-match lookup through any unique index qualifies, and databases treat it as the same kind of access: TiDB, for one, uses its `Point_Get` operator for a predicate on a primary key and for a predicate on a unique key alike. An equality predicate on a non-unique indexed column is the same access pattern too and simply returns more than one row, which is why the useful phrasing is "a small number of records by a key" rather than "a single row".

## Formal statement

A point query is an equality lookup, in the same family as the key-value `Get` that storage engines expose, and it is opposed to a range scan, the family of the iterator that seeks once and then walks forward.

The operational property is a complexity result. Against a B-tree index of fanout $B$ over $n$ records, the lookup touches one node per level and the number of levels is the depth of the tree, so its cost is

$$O(\log_B n)$$

against $O(n)$ for a full scan of the same table. The work is set by the depth of the index, not by how many records the table holds. Because $B$ is large, a few hundred keys to a page, the depth grows slowly enough that the lookup stays roughly as cheap on a billion rows as on a thousand. A scan has no such guarantee: its cost tracks how much it reads, so it grows with the table.

## Where it is used

It is the read pattern that defines [[Online Transaction Processing]], the first row of that note's comparison table and the thing every other OLTP property follows from. Indexes exist so that this lookup does not degrade into a scan, which is why an operational schema is shaped around the keys the application actually looks records up by.

A [[Data Warehouse]] is the contrast case: an analytical query deliberately does not look records up, it scans and aggregates them, and a warehouse is laid out for that instead. A storage layout tuned for cheap point lookups is the wrong layout for cheap scans, which is most of the reason the two workloads end up in different systems. Within an operational system the lookup is normally not issued alone but inside a [[Transaction]], grouped with the writes that depend on what it returned.
