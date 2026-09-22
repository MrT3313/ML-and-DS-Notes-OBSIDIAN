---
note_kind: concept
aliases:
  - transaction
  - transactions
  - database transaction
up: "[[Online Transaction Processing]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

A transaction is a group of reads and writes that form a logical unit. The application draws the boundary around the operations that belong to one action, and the database treats what is inside that boundary as one thing rather than as a sequence of unrelated statements.

The guarantees a database may attach to that boundary are [[ACID]], and they are named and bounded there rather than here. What happens when two transactions overlap, meaning which isolation level an engine actually runs at and which anomalies that level still permits, is the subject of [[DDIA]] chapter 8 and is not settled in either note.

## Formal statement

The boundary is what any guarantee quantifies over, and that is what the structure buys. For a group of operations $o_1, \dots, o_n$ enclosed in one transaction, either every $o_i$ takes effect or none of them does, and there is no third result that the caller or a later reader can observe.

So a transaction ends in exactly one of two ways.

- It **commits**, and every operation inside the boundary stands together.
- It **aborts**, and the database is left as it was when the boundary opened, with any step already applied rolled back.

Nothing between those two is an end state. A transaction that has not committed or aborted is in flight rather than partly done, which is the property the whole device exists to provide, and the caller is told which of the two it got rather than having to infer it from what it can read afterwards. An abort has two possible origins and the outcome is the same either way: the application asks for one after finding bad input or a violated rule, or the database imposes one after a deadlock or a timeout.

Which guarantees ride on that boundary, and what each of them does and does not promise, is [[ACID]]. Concurrency is the half still open: the contract above fixes what one transaction looks like from outside, and says nothing about what two of them do to each other, which is where [[DDIA]] chapter 8 goes.

## Where it is used

[[Online Transaction Processing]] is named for this unit, and the name is accurate: an OLTP request is normally one transaction, opened when a user action arrives and closed when the system is done responding to it. That is the granularity at which an operational system counts its throughput and at which a failure is retried.

It is also what makes an operational database usable as a [[System of Record]]. Trusting one copy as the authoritative one requires knowing that a half-applied update is not a state the data can be left in, and the transaction boundary is where that assurance is defined. A [[Point Query]] inside a transaction is read under those guarantees rather than as a bare lookup.
