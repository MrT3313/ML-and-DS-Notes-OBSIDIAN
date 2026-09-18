---
note_kind: concept
aliases:
  - data silo
  - data silos
  - silo
  - silos
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

A data silo is data held inside one department's or one system's boundary in a way the rest of the organization cannot reach. The form the problem takes when you try to do analysis is concrete: the data of interest is spread across multiple operational systems, which makes it hard to combine those datasets into a single query.

## Formal statement

The condition a silo names is checkable against a real organization: one real-world entity, a customer or an order, exists in more than one system, under more than one identifier and more than one schema, and no system holds the correspondence between them. Three things follow.

- **Duplicate records.** The same customer is a row in the billing system and a different row in the support system, and neither is wrong in its own system.
- **Metrics that disagree.** Two departments each compute "active users" from the data they can reach, get different numbers, and both are defensible, because the disagreement is in which rows were reachable rather than in the arithmetic.
- **Questions that cannot be asked at all.** Any question whose answer needs a join across two silos has no query that can express it, so it goes unanswered rather than answered wrongly.

The honest limit: nothing dissolves a silo automatically. Consolidating the sources relocates the work of reconciling identifiers and definitions into the pipeline that does the consolidating, where it is at least written down in one place, but it does not make the work disappear.

## Where it is used

It is the first of the four reasons a [[Data Warehouse]] exists, and the one the warehouse answers most directly: the warehouse is a single queryable copy of the data from all of the various operational systems, so the join that no [[Online Transaction Processing]] system could serve becomes an ordinary query there. An [[Extract-Transform-Load]] pipeline is the mechanism that gets it there, and it is the place where the reconciling actually happens, since conforming identifiers and units across sources is the transform step's real job rather than an incidental part of it.
