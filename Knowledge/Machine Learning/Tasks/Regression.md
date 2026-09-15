---
note_kind: index
aliases:
  - regressor
  - regressors
  - regression task
up: "[[Machine Learning]]"
---

Every method that predicts a continuous numerical value for an input: a house price, a temperature, next month's sales. A [[Supervised Learning]] task. Géron's terms: univariate when there is one feature, multiple when several.

## Methods

- [[Linear Regression]]: weighted sum of features, least squares. The baseline and the chapter's worked example.

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

