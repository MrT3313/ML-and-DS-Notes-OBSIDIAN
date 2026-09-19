---
note_kind: index
aliases:
  - cluster analysis
  - clustering algorithms
  - clusters
up: "[[Machine Learning]]"
---

Every method that groups unlabeled instances so that members of a group are more similar to each other than to members of other groups. An [[Unsupervised Learning]] task; the similarity measure is the design choice, and the result is a description of the data, not a predictor. Also the first step in [[Semi-Supervised Learning]] pipelines.

## Methods

None written yet. K-Means, arriving with HOML chapter 9, which also brings hierarchical clustering and DBSCAN.

```base
filters:
  and:
    - file.hasLink(this.file)
    - file.inFolder("Knowledge")
views:
  - type: table
    name: Linked here
    order:
      - file.name
      - note_kind
      - confidence
```
