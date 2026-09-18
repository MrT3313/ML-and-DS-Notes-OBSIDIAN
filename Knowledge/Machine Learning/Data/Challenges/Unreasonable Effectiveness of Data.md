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

The unreasonable effectiveness of data is the empirical finding that very different algorithms perform almost equally well once they are given enough data, and that a simple algorithm with a lot of data often beats a sophisticated one with little. The phrase is the title of Halevy, Norvig, and Pereira (2009); the evidence Géron shows is Banko and Brill (2001): a plot of accuracy against training-set size on a natural-language disambiguation task, where the curves for four algorithms rise together and converge as $m$ goes from $10^5$ to $10^9$ words.

## Formal statement

**Not further quantitative at this depth.**

This is an empirical finding, not a law, and nothing stands above the discharge because the finding has no formula: it is the observation that four curves converge, made on one task. What would make it quantitative is a per-algorithm [[Learning Curve]] with a stated rate at which the gap between algorithms closes with $m$, and no source in the vault supplies one.

## Where it is used

It reframes [[Insufficient Training Data]]: for many tasks, collecting more data is a better investment than a better model. Géron's caveat, which I keep: small and medium datasets are still the common case, and there algorithm choice matters. The size-indexed [[Learning Curve]] is how the claim is tested on your own data, since a validation error still falling at the largest $m$ available says more data is the better buy. It also bounds the stakes of [[Model Selection]]: at large $m$ the candidates converge and the choice between them matters less, while at small $m$ it is where the caveat bites.
