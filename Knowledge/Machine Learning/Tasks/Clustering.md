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

None written yet. Chapter 9 brings [[K-Means]], hierarchical clustering, and DBSCAN.

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

## What is missing

All methods; how to choose the number of clusters; how to evaluate a clustering without labels.
