---
note_kind: concept
aliases:
  - data-intensive
  - data intensive
  - data-intensive application
  - data-intensive applications
  - compute-intensive
  - compute intensive
  - compute-intensive application
up: "[[Data Systems]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

An application is **data-intensive** when data management is one of the primary challenges in developing it. Four things are what make it the primary challenge: 
- storing and processing large data volumes
- managing changes to data
- ensuring consistency in the face of failures and concurrency
- making services highly available. 
The label is a statement about where the difficulty lives, not about how much data there happens to be, and any one of the four is enough to earn it.

## VS

An application is **compute-intensive** when parallelizing a very large computation is one of the primary challenges. The work is one big calculation that has to be split across many cores, and the hard part is the splitting rather than the moving and keeping of the data.

The two are a pair of answers to one question, which is what the scarce resource is. Neither answer is about volume. A compute-intensive job can read a small input and then run for a week, and a data-intensive service can do almost no arithmetic per request while struggling to serve a petabyte reliably.

They are also not exclusive, and the interesting systems are usually both. Model training on a scientific cluster is compute-intensive in the arithmetic and data-intensive in getting the training set to the cores and keeping the checkpoints consistent. (NIST SP 800-223)[https://csrc.nist.gov/pubs/sp/800/223/final] describes present-day [[High-Performance Computing]] installations whose parallel file systems are measured in petabytes and can reach exabytes, sitting next to compute nodes carrying GPUs for modelling and machine learning, which is both challenges in one machine room.

## Formal statement

**Not further quantitative at this depth.**

The distinction classifies where the difficulty in a project lies, so there is no quantity to write down for it.

## Where it is used

This is the axis the whole of [[Data Systems]] is organized around. The components such a system is assembled from, a database, a cache, a search index, stream processing and batch processing, are each a standard answer to part of one of these four challenges, and so far only the cache has any treatment, inside [[Derived Data]]. [[Distributed System]] is what you reach for when the answers stop fitting on one machine, which is why volume, availability and consistency keep reappearing there as reasons to spread work across a network. The two shapes the work takes are [[Online Transaction Processing]], where the difficulty is many small concurrent reads and writes of current state, and [[Online Analytical Processing]], where it is scanning enormous histories, and the split between them exists because one machine tuned for the first is bad at the second.
