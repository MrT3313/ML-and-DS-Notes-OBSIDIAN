---
note_kind: concept
aliases:
  - data beats algorithms
  - more data beats better algorithms
up: "[[Training Set]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

The unreasonable effectiveness of data is the empirical finding that very different algorithms perform almost equally well once they are given enough data, and that a simple algorithm with a lot of data often beats a sophisticated one with little. The phrase is the title of Halevy, Norvig, and Pereira (2009); the evidence Géron shows is Banko and Brill (2001): a plot of accuracy against training-set size on a natural-language disambiguation task, where the curves for four algorithms rise together and converge as $m$ goes from $10^5$ to $10^9$ words.

### Mind against data and compute

The finding is contested rather than settled. Against "Data" stands "Mind", meaning inductive biases or intelligent architectural design, and "Data" is itself grouped with computation, since more data tends to require more computation, so what gets traded against architecture is data and the compute needed to consume it. Nobody disputes that finite data is necessary; what is contested is whether it is sufficient. Judea Pearl presses the mind side hardest, titling the introduction to *The Book of Why* (2018) "Mind over Data", and Christopher Manning puts the milder version, that structure is what lets a system learn more from less data. An *inductive bias* is any basis a learner has for preferring one generalization over another beyond bare consistency with the training instances, and Mitchell (1980) makes it a necessity rather than a taste: a learner carrying none cannot classify anything it has not already seen, so it cannot beat rote lookup. This is not the *bias* of the [[Bias-Variance Tradeoff]], which is a statistical quantity inside a decomposition of expected error, and a reader who arrived wanting that one wants that note instead.

## Formal statement

**Not further quantitative at this depth.**

This is an empirical finding, not a law, and nothing stands above the discharge because the finding has no formula: it is the observation that four curves converge, made on one task. What would make it quantitative is a per-algorithm [[Learning Curve]] with a stated rate at which the gap between algorithms closes with $m$, and no source in the vault supplies one.

## Where it is used

It reframes [[Insufficient Training Data]]: for many tasks, collecting more data is a better investment than a better model. Géron's caveat, which I keep: small and medium datasets are still the common case, and there algorithm choice matters. The size-indexed [[Learning Curve]] is how the claim is tested on your own data, since a validation error still falling at the largest $m$ available says more data is the better buy. It also bounds the stakes of [[Model Selection]]: at large $m$ the candidates converge and the choice between them matters less, while at small $m$ it is where the caveat bites.
