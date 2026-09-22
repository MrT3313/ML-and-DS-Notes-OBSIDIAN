---
note_kind: concept
aliases:
  - insufficient data
  - small data
  - too little data
up: "[[Training Set]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
  - "[[DMLS Ch01 Overview of Machine Learning Systems]]"
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## Definition

Insufficient training data is the challenge of having too few instances for the model to separate real patterns from noise. Simple problems may need thousands of examples; image and speech tasks need millions unless a pretrained model can be reused.

## Formal statement

Of the three terms a model's error decomposes into under the [[Bias-Variance Tradeoff]], only variance depends on $m$. For a linear model fitted by least squares, averaged over the training inputs, it falls like

$$\operatorname{Var}\big(\hat{h}(\mathbf{x})\big) \approx \frac{\sigma^{2} d}{m}$$

with $d$ the number of fitted parameters and $\sigma^{2}$ the noise in the target. Squared bias and the noise floor $\sigma^{2}$ do not move with $m$ at all.

That is the whole diagnostic. Being short of data is the condition that variance dominates the other two terms, so more instances buy error back at a rate of $1/m$, while a model whose error is mostly bias will not improve however much is added. Which case you are in is read off a [[Learning Curve]], whose shape says whether the variance term is still live or already spent.

Formal sample-complexity results bound the gap of [[Generalization]] in terms of $m$ and a capacity measure, but they are distribution-free and worst-case, so the sizes they return are far too pessimistic to plan against.

## Where it is used

It is the first of the data-side challenges a project has to clear, and the [[Unreasonable Effectiveness of Data]] is its empirical counterpart: past a certain size, data beats algorithm choice. The remedies split by which term is dominant: more instances, whether collected or manufactured by [[Data Augmentation]], against the variance term; a simpler model or stronger [[Regularization]] against it too, by cutting $d$ rather than raising $m$; and [[Transfer Learning]] when neither is available, reusing a model fitted on a task that had cheap and abundant data so the target pays only for adaptation. That last remedy divides further by how many labelled examples of the target classes are left: [[Few-Shot Learning]] when a handful of them exist and the model is adapted from those, and [[Zero-Shot Learning]] when there are none at all and a description of the classes has to stand in for the missing labels. Left unaddressed it shows up as [[Overfitting]], since too few instances relative to capacity leaves the model free to fit noise.

### Four ways to get labels you do not have

Being short of data and being short of *labels* are not the same shortage, and the second has four standing answers. They are told apart by where the label comes from and by how much ground truth each still demands.

| approach | where the label comes from | ground truth still needed |
|---|---|---|
| [[Weak Supervision]] | heuristics that are allowed to be noisy, applied over the unlabelled pool | none in principle, though a small set is worth having to check the heuristics against |
| [[Semi-Supervised Learning]] | structural assumptions about the data, which extend the labels you already have | a small seed set for the assumptions to propagate from |
| [[Transfer Learning]] | a model pretrained on another task, carried over rather than generated | none in the [[Zero-Shot Learning]] limit; some for fine-tuning, though far fewer than a fit from scratch would need |
| [[Active Learning]] | a person, asked about the instances the model gains most from | yes, this one buys ground truth and its whole contribution is spending the budget well |

The four are not mutually exclusive and are often stacked, a pretrained model fine-tuned on weakly labelled data being the common combination. Only the last brings genuinely new information about the target into the system, which is also why it is the only one that costs annotation time; the other three are ways of stretching information already on hand.
