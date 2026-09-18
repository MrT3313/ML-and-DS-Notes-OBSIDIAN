---
note_kind: concept
aliases:
  - system of record
  - systems of record
  - source of truth
  - authoritative source
  - canonical data
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

A **system of record** holds the authoritative, canonical version of some piece of data. If any other system disagrees with it, the system of record is by definition the correct one.

That last clause is the whole point of the term. It is not a prediction that the authoritative copy is more accurate, and not a claim about which system has better hardware or fresher data. It is a rule decided in advance about who wins a disagreement, so that when two systems report different numbers nobody has to argue about which to believe.

## VS

The counterpart is [[Derived Data]], and the relationship is a direction rather than a ranking. Authority runs one way: a derived copy is computed from the system of record, so a discrepancy between them is a defect in the derived copy and is repaired by recomputing it. The repair never runs backwards, because writing a derived value back over the authoritative one would destroy the only copy that nothing else can reconstruct.

The practical test is loss. Delete a derived copy and you rebuild it from upstream. Delete the system of record and there is nothing upstream to rebuild it from, which is why backup, durability and audit effort concentrate there.

## Formal statement

The rule, as a procedure: if system $A$ is the system of record for a datum $x$ and system $B$ reports a different value for $x$, then $B$ is wrong, and the repair is to recompute $B$ from $A$.

Two invariants have to hold for that procedure to terminate on an answer.

**Uniqueness.** Each datum has exactly one system of record. Two systems both claiming authority over the same field is not a redundant arrangement, it is an unresolvable disagreement, because the rule above then fires in both directions at once.

**Acyclicity.** The authority relation has no cycles. If $B$ is derived from $A$, nothing on which $A$ depends may in turn be derived from $B$, or a stale value can circulate back around and become authoritative by accident.

Which system holds the record is a decision, not a discovery, and it is made per field rather than per system. One service can be the system of record for a user's email address while another is the system of record for that user's billing history, and each is a derived consumer of the other.

## Where it is used

[[Derived Data]] is the other half of the pair, and the tie-break rule above is what makes a derived copy safe to keep: it can be redundant and stale without being dangerous, because it never gets to win. [[Online Transaction Processing]] systems are the usual holders of the record, since the operational database is where users actually create and modify state, and everything else in the organization is reading a copy of what happened there. [[Data Warehouse]] is the standard example on the other side, a read-only copy assembled from several operational systems that is authoritative for nothing, which is exactly why analysts may query it freely.
