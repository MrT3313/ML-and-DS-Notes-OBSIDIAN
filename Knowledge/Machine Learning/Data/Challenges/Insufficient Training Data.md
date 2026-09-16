---
note_kind: concept
aliases:
  - insufficient data
  - small data
  - too little data
up: "[[Training Set]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Insufficient training data is the challenge of having too few instances for the model to separate real patterns from noise. Simple problems may need thousands of examples; image and speech tasks need millions unless a pretrained model can be reused.

## Formal statement

With $m$ instances and a model of capacity roughly $d$ parameters, the generalization gap of [[Generalization]] shrinks as $m$ grows for fixed $d$; when $m$ is small relative to $d$ the model can fit noise, which is [[Overfitting]]. A precise bound is chapter 4 material.

## Where it is used

It is the first of the chapter's data-side challenges, and the [[Unreasonable Effectiveness of Data]] is the empirical counterpart: past a certain size, data beats algorithm choice. The remedies are more data, a simpler model, or transfer from a model trained elsewhere.
