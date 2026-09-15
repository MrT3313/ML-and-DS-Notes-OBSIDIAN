---
note_kind: index
aliases:
  - outlier detection
  - anomalies
  - outliers
up: "[[Machine Learning]]"
---

Every method that flags instances that deviate strongly from the majority, in a dataset that may itself contain some outliers. An [[Unsupervised Learning]] task. Uses: fraud, defects, and cleaning a dataset before training, which links it to [[Poor-Quality Data]]. Contrast with [[Novelty Detection]], which assumes a clean training set.

## Methods

None written yet. Chapter 9 brings Isolation Forest and one-class SVM; Gaussian mixtures also serve.

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

All methods; the threshold choice; evaluation when anomalies are rare.
