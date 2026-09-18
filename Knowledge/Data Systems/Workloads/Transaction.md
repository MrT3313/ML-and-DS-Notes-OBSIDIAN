---
note_kind: concept
aliases:
  - transaction
  - transactions
  - database transaction
up: "[[Online Transaction Processing]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

A transaction is a group of reads and writes that form a logical unit. The application draws the boundary around the operations that belong to one action, and the database treats what is inside that boundary as one thing rather than as a sequence of unrelated statements.

Which guarantees the database attaches to that boundary, and what happens when two transactions overlap, is the subject of DDIA chapter 8 and is not settled here.

## Formal statement

**Not further quantitative at this depth.**

A transaction is a structuring device, a bracket the application puts around a group of operations, so there is no quantity attached to it here. The guarantees that attach to the bracket, which are what would make this section precise, arrive with DDIA chapter 8.

## Where it is used

[[Online Transaction Processing]] is named for this unit, and the name is accurate: an OLTP request is normally one transaction, opened when a user action arrives and closed when the system is done responding to it. That is the granularity at which an operational system counts its throughput and at which a failure is retried.

It is also what makes an operational database usable as a [[System of Record]]. Trusting one copy as the authoritative one requires knowing that a half-applied update is not a state the data can be left in, and the transaction boundary is where that assurance is defined. A [[Point Query]] inside a transaction is read under those guarantees rather than as a bare lookup.
