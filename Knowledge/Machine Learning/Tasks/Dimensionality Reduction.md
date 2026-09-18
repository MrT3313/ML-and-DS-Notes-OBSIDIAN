---
note_kind: index
aliases:
  - dimension reduction
  - feature extraction methods
  - manifold learning
up: "[[Machine Learning]]"
---

Every method that reduces the number of features while keeping as much of the information as possible: to remove noise, speed up training, or visualize high-dimensional data in two or three dimensions. An [[Unsupervised Learning]] task and one form of feature extraction under [[Irrelevant Features]]. Géron's example: merge a car's mileage and age into one wear feature.

## Methods

None written yet. Chapter 8 brings PCA, random projection, LLE, and t-SNE for visualization.

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

All methods; the curse of dimensionality that motivates them; when reduction hurts.
