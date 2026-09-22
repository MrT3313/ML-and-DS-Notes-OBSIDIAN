---
note_kind: concept
aliases:
  - sampling bias
  - selection bias
  - unrepresentative data
  - sampling noise
up: "[[Training Set]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## Definition

Training data is nonrepresentative when it does not reflect the population the model will meet, so the model learns the sample's quirks as if they were the world's. There are two causes: sampling noise, when the sample is too small to be representative by chance, and sampling bias, when the collection method systematically over- or under-represents part of the population.

## Formal statement

The sample is drawn from $q(\mathbf{x}, y) \ne p(\mathbf{x}, y)$, where $p$ is the population. A model fit to $q$ minimizes $\mathbb{E}_q[\ell]$, not $\mathbb{E}_p[\ell]$, and the gap between them is not visible from within the sample.

**Not further quantitative at this depth.**

Nonrepresentative data is a data pathology, and the mismatch $q \ne p$ above is the whole of its mathematics at this depth: it names the gap without measuring it, and no bound on $\mathbb{E}_p[\ell] - \mathbb{E}_q[\ell]$ is stated here. What a later chapter adds is not a measure of the gap but a sampling procedure that keeps $q$ close to $p$ by construction, which is [[Stratified Sampling]].

## Where it is used

Géron's example: the 1936 Literary Digest poll sampled from phone books and club rosters, so it reached the wealthy and predicted the wrong winner. The same failure at modern scale is the pretraining corpus of a language model, assembled out of whatever text could actually be gathered, Wikipedia and Common Crawl and forum text from Reddit among the usual constituents, rather than out of text representative of everything the model will later be asked about; whatever that reachable slice of the web over-represents, the model over-represents with it.

[[Nonprobability Sampling]] is the family of procedures whose use is the commonest way this condition is produced, convenience, snowball, judgement and quota being its members, and the two notes divide as procedure against outcome: that note owns how the rows get picked, this one owns the resulting mismatch between the sample and the population. [[Sampling]] is the general subject both sit under.

In the vault this is the collection-side cause of [[Data Mismatch]]; the remedy is a sampling procedure that matches production, which [[Stratified Sampling]] gives by construction for whatever variable drives the target. The same requirement falls on the [[Testing Set]]: a held-out slice that misrepresents the population estimates generalization error for the wrong population. And a sample that was representative when collected stops being so once the population moves, which is the mechanism behind [[Model Rot]].
