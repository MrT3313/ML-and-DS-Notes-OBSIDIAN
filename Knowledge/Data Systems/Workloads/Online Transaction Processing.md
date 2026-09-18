---
note_kind: concept
aliases:
  - OLTP
  - online transaction processing
  - transaction processing
  - operational system
  - operational systems
  - OLTP system
  - OLTP systems
  - OLTP workload
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

Online transaction processing is the workload an operational system carries: a large number of small requests, each touching very few records, each arriving because a user or a service acting for a user just did something, and each answered while the caller is waiting for it. An *operational system* is the backend service and the data infrastructure under it, where data is read, created, and modified in a database based on actions performed by users. Backend engineers are the people who build and run it.

A request typically looks up a small number of records by a key, which is a [[Point Query]], and then inserts, updates, or deletes records based on the user's input. The queries themselves are mostly a fixed set baked into the application code. One-off custom queries do get written, but for maintenance or troubleshooting rather than as the normal traffic, which means the access paths an OLTP database has to serve well are known in advance and can be indexed for.

## VS

The axis is what a single request does to the data. OLTP reads and writes individual records, by key, in their current state. [[Online Analytical Processing]] reads an enormous number of records and hands back a summary of them rather than the records themselves. Everything else in the table below follows from those two read patterns, including the storage layouts each side wants, which is why one database is rarely good at both.

| Property            | Operational system (OLTP)                              | Analytical system (OLAP)                       |
| ------------------- | ------------------------------------------------------ | ---------------------------------------------- |
| Main read pattern   | [[Point Query\|point queries]] (fetch individual records by key) | aggregate over a large number of records       |
| Main write pattern  | create, update, and delete individual records          | bulk import ([[Extract-Transform-Load\|ETL or ELT]]) or event stream |
| Human user example  | end user of a web or mobile application                | internal analyst for decision support          |
| Machine use example | checking if an action is authorized                    | detecting fraud and abuse patterns             |
| Type of queries     | fixed, predefined by the application                   | arbitrary, ad-hoc exploration by analysts      |
| Query volume        | lots of small queries                                  | few queries, each of them complex              |
| Data represents     | latest state of the data (current point in time)       | history of events that happened over time      |
| Dataset size        | gigabytes (GB) to terabytes (TB)                       | terabytes (TB) to petabytes (PB)               |

"Online" in this name means interactive and live: the request is answered while somebody waits for the answer, as opposed to being queued and run later in a batch. That is the sense the term was coined in, back when the alternative was submitting a job and collecting the output afterwards, and it survives in benchmark definitions, where a transaction is described as either executed online or queued for deferred execution. It is a different word from the "online" in [[Online Learning]], which names a model that updates its parameters one instance or mini-batch at a time. Neither is the everyday networked sense of the word.

## Formal statement

The size band in the table above is an order-of-magnitude comparison taken from [[DDIA Ch01 Trade-Offs in Data Systems Architecture|DDIA chapter 1]], not a threshold. A system does not stop being OLTP when it crosses a petabyte; the band says only that an operational dataset holds current state while an analytical one accumulates history, so the two grow at different rates.

The latency requirement is implied by "online" and by nothing stronger: a human or a service is blocked until the answer comes back. The term fixes no particular number of milliseconds.

**Not further quantitative at this depth.**

OLTP names the shape of the traffic a system receives, not a quantity, so there is no formula to write.

## Where it is used

[[Point Query]] is the read pattern that defines the workload, and [[Transaction]] is the unit of work the name is built on, a group of reads and writes committed or abandoned together. An operational database is normally the [[System of Record]] for whatever it holds, meaning the authoritative copy that everything else is derived from, which is why a write here is a write to the truth rather than to a cache.

[[Online Analytical Processing]] is the other half of the axis, and [[Data Warehouse]] is the thing built specifically so that analysts stop querying the operational system directly: expensive scans over an OLTP database degrade it for the users it exists to serve. [[Hybrid Transactional-Analytical Processing]] is the attempt to put both workloads back into one system.
