---
note_kind: concept
aliases:
  - sample
  - instance
up: "[[Training Set]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---
## Definition
A [[Training Instance]] is one row of it: one feature vector, and in supervised learning one label.

## Formal statement

An instance is one row of the design matrix: $\mathbf{x}^{(i)} \in \mathbb{R}^{n}$, the vector of its $n$ feature values, paired in supervised learning with its label $y^{(i)}$. The training set is the collection of $m$ such pairs,

$$D_{\text{train}} = \{(\mathbf{x}^{(i)}, y^{(i)})\}_{i=1}^{m}$$

and $\mathbf{X} \in \mathbb{R}^{m \times n}$ stacks the instances as rows, so row $i$ of $\mathbf{X}$ is $(\mathbf{x}^{(i)})^{T}$.

## Where it is used

The [[Training Set]] is the collection of $m$ instances the system learns from, and each column running across those instances is a [[Feature]]. A [[Model]] reads one instance at a time and returns one prediction for it, $\hat{y}^{(i)} = h(\mathbf{x}^{(i)})$. [[Stochastic Gradient Descent]] computes each update on a single instance drawn at random, which is what makes the cost of one step independent of $m$. [[Random Sampling]] decides, instance by instance, which side of the train-test split a row lands on.
