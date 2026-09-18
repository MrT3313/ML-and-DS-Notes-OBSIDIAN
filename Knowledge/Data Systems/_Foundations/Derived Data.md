---
note_kind: concept
aliases:
  - derived data
  - derived data system
  - derived data systems
  - redundant data
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

**Derived data** is the result of taking existing data from another system and transforming or processing it in some way. A **derived data system** is a system whose whole contents are of that kind, which means it can be recreated from its original source if lost.

Being rebuildable is what the term is for. Derived data is redundant by construction, since every value in it already exists somewhere upstream, and redundancy is usually a thing to avoid. Here it is often essential anyway, because the derived copy is arranged for a job the original cannot do quickly.

### The cache as the worked case

A cache is the smallest example. Data can be served from the cache if it is present, and if the cache does not contain what you need you fall back to the underlying database. Both halves matter. The hit path is why the cache exists, and the miss path is why it is allowed to exist: since every answer is also obtainable from the database, an empty cache is slow rather than wrong, and a cache that is thrown away entirely costs latency rather than data. A copy without a working fallback path is not a cache, it is a second [[System of Record]] that nobody has admitted to keeping.

## VS

[[System of Record]] is the counterpart, and the two are told apart by one question: if this were deleted, could it be rebuilt from something else? If yes, it is derived. If no, it is the record.

Derived is not a synonym for lesser or for optional. Most of what users actually touch is derived, and the derived copy is frequently the faster, more convenient and more heavily queried of the two. What it does not get is authority. A stale or wrong derived value is repaired by recomputing it from upstream, which is why the redundancy is affordable in the first place.

## Formal statement

A derived value $d$ is a function of source state $s$:

$$d = f(s)$$

where $f$ is the transformation the pipeline implements. Two consequences follow directly from that equation, and they are the whole practical content of the term.

Recoverability: because $f$ and $s$ both still exist, any lost $d$ is recomputable by running $f$ again, so the derived store needs no independent backup, only a rerunnable pipeline.

Staleness: what is stored is $f(s)$ as of the last time the pipeline ran, while the current truth is $f(s_{\text{now}})$. The gap between them is the staleness, and it is not an error condition but a parameter of the design, set by how often $f$ is rerun and how much lag the consumer can absorb.

## Where it is used

Analytical systems are usually derived data systems, because they are consumers of data created elsewhere. That is the link into the whole analytics side: a [[Data Warehouse]] holds a read-only copy assembled from the operational databases, and [[Online Analytical Processing]] queries it precisely because nothing there is authoritative and a heavy scan therefore cannot damage anything a user depends on. A [[Data Lake]] is the same idea one step earlier, holding the copy in its raw form so each consumer can apply its own $f$.

[[Extract-Transform-Load]] is how the derived copy is actually produced, and it is where $f$ lives as running code. [[System of Record]] is the upstream end that every derived store points back at, and the rule that it wins any disagreement is what lets a derived copy be redundant without being risky.
