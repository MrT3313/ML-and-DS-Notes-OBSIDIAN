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

Retail placement and recommendation: which products to shelve together, and what to suggest once a basket already holds an item. It is one of the tasks of [[Unsupervised Learning]], since transactions carry no label and the rules are structure found in the data itself. [[Clustering]] and [[Anomaly Detection]] are its sibling unsupervised tasks, and the contrast is what each one groups: clustering groups instances by similarity, anomaly detection singles out the instances that fit no group, and association rule learning groups attribute values rather than instances, by how often they occur together.
