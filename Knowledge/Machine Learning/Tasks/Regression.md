---
note_kind: index
aliases:
  - regressor
  - regressors
  - regression task
up: "[[Machine Learning]]"
---

Every method that predicts a continuous numerical value for an input: a house price, a temperature, next month's sales. A [[Supervised Learning]] task. Géron's terms run along two independent axes. Univariate versus multivariate counts the outputs, one predicted value per instance or several at once; simple versus multiple counts the inputs, one feature or many. Chapter 2's housing problem is both multiple and univariate: many features per district, a single median house value out.

## Methods

- [[Linear Regression]]: weighted sum of features, least squares. The baseline, and the worked example on [[California Housing]].

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

