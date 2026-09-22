---
note_kind: concept
aliases:
  - ACID
  - ACID transaction
  - ACID transactions
  - ACID guarantees
  - atomicity
  - durability
  - transaction isolation
  - BASE
up: "[[Transaction]]"
sources:
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---

## Definition

ACID is the set of guarantees a database may attach to a [[Transaction]]: atomicity, consistency, isolation and durability. The transaction draws the boundary and says which operations belong together; ACID says what the database promises about the group inside it. The four are bundled by an acronym rather than by a mechanism, and an engine implements them with four different pieces of machinery.

A transactional database usually offers all four and is not required to. Giving one up is a design decision rather than a defect: something is bought with it, normally availability or scale, and the caller then owes whatever work the database has stopped doing. Systems built that way are sometimes labelled BASE.

### Where the acronym comes from

The four properties were named as one set by Theo Haerder and Andreas Reuter in 1983, in a survey of database recovery, and that paper is where the initials were first put together as a test of an engine's quality. Three of the four are given there as obligations on the system. Consistency is not: the paper defines it by saying that each successful transaction, in its own words, "by definition commits only legal results", so legality is assumed of the transaction program and the paper then spends its length on making sure only complete transactions leave a trace.

A widely repeated claim goes further and says the C was included largely to make the acronym pronounceable. The 1983 paper says nothing of the kind. The remark is attributed to Joe Hellerstein and reaches most readers through [[DDIA]], which relays it as his; no statement by Hellerstein himself stands behind it that this note could find, so it is recorded as a remark of uncertain provenance rather than as a fact about the paper. What the paper does support is the narrower and more useful point, which is the one the table below turns on: the C is the letter whose enforcement the database shares with the application, and keeping that split visible is what stops the word being confused with the distributed sense of consistency that [[DDIA]] chapter 10 is named for.

## VS

BASE is the counterpart acronym, coined by Dan Pritchett in 2008 for what a partitioned system looks like when it chooses availability over consistency: basically available, soft state, eventually consistent. Pritchett puts the two in opposition directly, writing that "where ACID is pessimistic and forces consistency at the end of every operation, BASE is optimistic and accepts that the database consistency will be in a state of flux".

The availability is bought by tolerating partial failure rather than by any new guarantee. Partition users across five database servers and a design in this style keeps an outage on one of them from taking down the other four fifths, which is why the acronym leads with "basically available" rather than with a stronger word.

What BASE is not is an absence of reasoning about correctness. Pritchett's own claim is that it "requires a more in-depth analysis of the operations within a logical transaction than is typically applied to ACID", because the invariants the database was enforcing become the application's to maintain across a queue and a delay. The work moves; it does not disappear. A store described as BASE has therefore made a trade that has to be checked against what the data is for, and it is the wrong choice for anything a [[System of Record]] is expected to settle.

## Formal statement

Each letter, stated as what the database promises and what it does not. The second column is the half that makes the promise checkable, since a guarantee with no stated limit cannot be found false.

| Letter          | What is guaranteed                                                                                                                                                                                                                | What is **not** guaranteed                                                                                                                                                                                                                              |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **A**tomicity   | Every operation inside the boundary takes effect, or none does. A transaction that cannot finish leaves no partial trace: steps already applied are rolled back, and the caller is told which of the two outcomes it got.            | That the transaction succeeds. Atomicity fixes the number of possible outcomes at two and says nothing about which one you get, so noticing an abort and retrying it is the caller's job. It also says nothing about what a concurrent reader sees meanwhile. |
| **C**onsistency | A committing transaction moves the database from one state satisfying the declared rules to another. The rules are the ones actually declared to the engine: uniqueness, foreign keys, check constraints, types.                    | Any invariant the application holds but never declared. The database enforces what it was told and cannot enforce what only the application code knows, which makes the C a joint obligation rather than a service the engine renders on its own.             |
| **I**solation   | Concurrent transactions behave as though they had run one after another, to the degree the isolation level in force provides.                                                                                                      | That the level in force is the strongest one. Most engines default to something weaker than serializable, and each weaker level permits its own named set of anomalies.                                                                                      |
| **D**urability  | Once a transaction has committed, its results survive a later failure of the system: a crash, a power cut, a restart.                                                                                                              | Loss of the storage itself. Surviving a failed disk or a destroyed site needs redundancy of a different kind, a copy of the data plus a log of everything since the copy was taken, which is a separate mechanism from the commit path.                       |

The order of the letters is not the order of dependence. Haerder and Reuter make consistency a precondition of durability rather than a peer of it, on the ground that making results permanent is only worth doing if the results were legal to begin with, so a system that commits garbage durably has satisfied D and failed the transaction anyway.

What happens when two transactions actually overlap is not reached here. The isolation levels, the anomalies each one permits, and what it costs to rule them out, are the subject of [[DDIA]] chapter 8, and until that is read the I in the table above is a promise whose strength is unquantified.

## Where it is used

These guarantees attach to a [[Transaction]] and to nothing smaller: the bracket is what they quantify over, which is why a database that offers ACID still offers no protection to two statements the application forgot to group. [[Online Transaction Processing]] is the workload that normally asks for them, because an operational request is a user's action and a half-applied action is a support ticket rather than a stale number.

They are also what makes an operational database usable as a [[System of Record]]. Trusting one copy as authoritative requires knowing that a half-applied update is not a state the data can be left in and that a committed one will still be there after a restart, which is atomicity and durability doing exactly that work; the other two decide what a concurrent reader is allowed to see while the update is in flight.

[[Hybrid Transactional-Analytical Processing]] is where the guarantees become a design constraint rather than a feature, since running an analytical scan inside a system that owes these promises to its writers is the whole difficulty the category exists to address.
