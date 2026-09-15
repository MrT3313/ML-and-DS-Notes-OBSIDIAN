---
note_kind: concept
aliases:
  - association rules
  - market basket analysis
  - frequent itemset mining
  - Apriori
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

It is an [[Unsupervised Learning]] task.

## Definition

Association rule learning finds items or attribute values that occur together in a large dataset more often than chance, and states them as rules. The standard setting is market basket analysis: customers who buy barbecue sauce and chips tend to buy steak.

## Formal statement

For itemsets $A$ and $B$ over transactions $D$, the rule $A \Rightarrow B$ has

$$\text{support}(A \Rightarrow B) = P(A \cup B), \qquad \text{confidence}(A \Rightarrow B) = P(B \mid A)$$

Rules are kept when both exceed chosen thresholds. Apriori prunes the search using the fact that a subset of a frequent itemset is itself frequent.

## Where it is used

Retail placement and recommendation.
