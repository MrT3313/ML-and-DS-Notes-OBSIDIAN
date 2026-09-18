---
note_kind: concept
aliases:
  - semi-supervised
  - partially labeled learning
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---
## Definition

Semi-supervised learning trains on a small labeled set together with a large unlabeled one. The unlabeled data reveals the shape of the distribution; the few labels name the parts of it.

## Formal statement

$D = D_L \cup D_U$ with $|D_L| \ll |D_U|$, where $D_L$ carries labels and $D_U$ does not. Most algorithms combine an unsupervised step on all of $D$ with a supervised step on $D_L$.

**Not further quantitative at this depth.**

Semi-supervised learning is a paradigm, and the decomposition of $D$ above is the whole of its mathematics at this depth: it fixes what the data looks like, not what any algorithm does with it. A later chapter would add the objective that ties the two parts together, ex a [[Clustering]] step on all of $D$ followed by a label-propagation rule from $D_L$.

## Where it is used

It sits between [[Supervised Learning]] and [[Unsupervised Learning]] on the supervision axis and is the practical answer to label cost. Typical pipelines run [[Clustering]] first and then propagate the few labels through the clusters.


