---
note_kind: index
aliases:
  - novelty
  - novel instances
up: "[[Machine Learning]]"
---

Every method that decides whether a new, unseen instance is abnormal, having trained only on a clean reference set with no outliers in it. An [[Unsupervised Learning]] task. It differs from [[Anomaly Detection]] in its training assumption: the training set is assumed pure, so anything unlike it is novel. Géron's example: a dataset with 1 percent Chihuahuas is normal for an anomaly detector but a novelty detector trained without Chihuahuas would flag them.

## Methods

None written yet. Chapter 9's one-class SVM is the usual choice.

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

All methods; whether this deserves its own map or folds into [[Anomaly Detection]] after chapter 9.
