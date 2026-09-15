---
note_kind: concept
aliases:
  - sampling bias
  - selection bias
  - unrepresentative data
  - sampling noise
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Training data is nonrepresentative when it does not reflect the population the model will meet, so the model learns the sample's quirks as if they were the world's. There are two causes: sampling noise, when the sample is too small to be representative by chance, and sampling bias, when the collection method systematically over- or under-represents part of the population.

## Formal statement

The sample is drawn from $q(\mathbf{x}, y) \ne p(\mathbf{x}, y)$, where $p$ is the population. A model fit to $q$ minimizes $\mathbb{E}_q[\ell]$, not $\mathbb{E}_p[\ell]$, and the gap between them is not visible from within the sample. Not further quantitative at this depth.

## Where it is used

Géron's example: the 1936 Literary Digest poll sampled from phone books and club rosters, so it reached the wealthy and predicted the wrong winner. In the vault this is the collection-side cause of [[Data Mismatch]]; the remedy is a sampling procedure that matches production, which chapter 2's stratified split addresses.
