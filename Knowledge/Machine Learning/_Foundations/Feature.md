---
note_kind: concept
aliases:
  - features
  - attribute
  - input variable
  - predictor variable
  - covariate
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---
## Definition

A [[Feature]] is one measurable property, characteristic, or attribute of an [[Training Instance|instance]]: a column of the data that the [[Model]] reads to make its prediction. 

> [!note]
> An "attribute" is the data type (mileage) while a "feature" is attribute plus value (mileage = 15,000), though the words are used interchangeably ~[[HOML Ch01 The Machine Learning Landscape|HOML Chapter 1]].

## Formal statement

Instance $i$ is the vector $\mathbf{x}^{(i)} \in \mathbb{R}^{n}$; feature $j$ is its component $x^{(i)}_j$, and the full feature matrix is $\mathbf{X} \in \mathbb{R}^{m \times n}$.

## Where it is used

Features are the input side of every [[Model]]. Features that carry no signal are [[Irrelevant Features]]; reducing $n$ is [[Dimensionality Reduction]]; noisy or missing values are [[Poor-Quality Data]]. In [[Supervised Learning]] the label is the one column that is not a feature.
