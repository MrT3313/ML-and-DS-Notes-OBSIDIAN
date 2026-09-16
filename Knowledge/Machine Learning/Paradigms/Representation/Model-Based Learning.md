---
note_kind: concept
aliases:
  - parametric learning
  - eager learning
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

A model-based system uses the training examples to fit a model with adjustable parameters, then predicts from the model alone; the training instances are not consulted at prediction time. The standard illustration fits a line to life satisfaction against GDP per capita and reads predictions off the line.

## Formal statement

Choose a family $h_{\boldsymbol\theta}$, a [[Performance Measure]], and set

$$\boldsymbol\theta^{*} = \arg\min_{\boldsymbol\theta} J(\boldsymbol\theta; D_{\text{train}})$$

Prediction is $h_{\boldsymbol\theta^{*}}(\mathbf{x})$, whose cost is independent of $m$.

## Where it is used

The alternative is [[Instance-Based Learning]]. [[Linear Regression]], [[Logistic Regression]], support vector machines, and neural networks are all model-based. The workflow is a four-step loop: study the data, select a model, train it, apply it to new cases. The family choice is the first [[Model Selection]] decision, and a wrong family gives [[Underfitting]].
