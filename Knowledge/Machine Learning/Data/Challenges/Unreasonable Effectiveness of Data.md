---
note_kind: concept
aliases:
  - data beats algorithms
  - more data beats better algorithms
up: "[[Training Set]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

The unreasonable effectiveness of data is the empirical finding that very different algorithms perform almost equally well once they are given enough data, and that a simple algorithm with a lot of data often beats a sophisticated one with little. The phrase is the title of Halevy, Norvig, and Pereira (2009); the evidence Géron shows is Banko and Brill (2001).

## Formal statement

The evidence is a plot of accuracy against training-set size on a natural-language disambiguation task, where the curves for four algorithms rise together and converge as $m$ goes from $10^5$ to $10^9$ words.

**Not further quantitative at this depth.**

## Where it is used

It reframes [[Insufficient Training Data]]: for many tasks, collecting more data is a better investment than a better model. Géron's caveat, which I keep: small and medium datasets are still the common case, and there algorithm choice matters.
