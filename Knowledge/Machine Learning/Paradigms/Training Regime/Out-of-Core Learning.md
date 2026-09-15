---
note_kind: concept
aliases:
  - out-of-core
  - out-of-memory training
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---
## Definition

Out-of-core learning trains on a dataset too large for main memory by loading it in chunks, running one incremental training step per chunk, and discarding the chunk. It is [[Online Learning]] applied to a fixed dataset rather than a stream.

## Formal statement

Partition $D$ into chunks $C_1, \dots, C_K$ with $|C_k| \cdot \text{bytes per instance} \le \text{RAM}$; for each $k$ run the online update on $C_k$. One pass over all chunks is one epoch. Not further quantitative at this depth.

## Where it is used

It needs an algorithm with an incremental update, which rules out closed-form fits like the normal equation for [[Linear Regression]]. It is the answer when [[Batch Learning]] is impossible for size reasons rather than for freshness reasons.
