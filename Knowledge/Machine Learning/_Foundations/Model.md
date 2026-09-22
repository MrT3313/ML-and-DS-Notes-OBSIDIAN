---
note_kind: concept
aliases:
  - hypothesis
  - predictor
  - predictors
  - model parameters
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
confidence: draft
---
## Definition

The [[Model]] is the part of a machine learning system that learns from the [[Training Set]] and then makes predictions. It is a function from a feature vector to an output, plus the parameters that were set during training. A [[Data Model]] is a different thing that claims the same word: the shape data is organized into before anything is fitted on it, settled by a designer in advance rather than estimated from a sample, so a reader who followed "model" here wanting that wants that note instead.

## Formal statement

The model is the hypothesis $h$ with parameters $\boldsymbol\theta$ chosen from data:

$$\hat{y}^i = h_{\boldsymbol\theta}(\mathbf{x}^i)$$

Training picks $\boldsymbol\theta$ to optimize a [[Performance Measure]] on the [[Training Set]].

## Where it is used

[[Model-Based Learning]] is the strategy of fitting such an $h$
- [[Instance-Based Learning]] replaces it with a similarity lookup. 

Choosing between candidate models is [[Model Selection]]
- a model too flexible for its data shows [[Overfitting]], one too rigid shows [[Underfitting]]. 

A deployed model decays through [[Model Rot]]. 

> [!example] Examples
> [[Linear Regression]], [[Logistic Regression]].
