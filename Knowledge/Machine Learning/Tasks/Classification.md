---
note_kind: index
aliases:
  - classifier
  - classifiers
  - classification task
up: "[[Machine Learning]]"
---

Every method that predicts a discrete class, label, or category for an input: spam or not, cat or dog or car. A [[Supervised Learning]] task; its performance measures arrive in chapter 3.

## Methods

- [[Logistic Regression]]: linear score through a sigmoid, thresholded. The baseline.

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
    sort:
      - property: file.name
        direction: ASC
      - property: confidence
        direction: DESC

```

