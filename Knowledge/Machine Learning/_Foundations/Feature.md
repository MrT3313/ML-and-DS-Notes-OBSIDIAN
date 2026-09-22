---
note_kind: concept
aliases:
  - features
  - attribute
  - input variable
  - predictor variable
  - covariate
up: "[[Training Instance]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
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

### Static and dynamic features

A feature computed by a scheduled job over historical data at rest is a *static feature*, and [[Batch Processing]] is the mode that produces it: the cadence can be slow because the quantity being computed moves slowly, which is the whole reason that mode is affordable here. A feature computed from the current state of a running system, off a transport as records arrive, is a *streaming feature*, also called a *dynamic feature*, and [[Stream Processing]] is the mode that produces it. Which of a model's inputs fall on which side is decided per feature rather than once for the model. [[DMLS]] chapter 5 is titled Feature Engineering and is where this belongs at length; what is here is the vocabulary and the pointer.
