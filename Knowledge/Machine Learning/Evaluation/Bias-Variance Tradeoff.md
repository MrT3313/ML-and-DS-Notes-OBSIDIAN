---
note_kind: concept
aliases:
  - bias variance tradeoff
  - bias-variance trade-off
  - bias/variance tradeoff
  - bias variance decomposition
  - bias variance
up: "[[Generalization]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

The expected error a model makes on a new instance splits into three parts with three different causes: 
- **bias**, being wrong on average because the model family cannot represent the truth; 
- **variance**, being wrong because the model is oversensitive to which particular training set it happened to be handed; and 
- **irreducible error**, the noise in the data itself, which no model of any kind removes. 

One word, two quantities: the bias here is a property of a whole model family, how far its average prediction sits from the truth, and it is unrelated to the *bias term* $\theta_0$ that [[Linear Regression]] adds to a weighted sum. Raising the bias term changes one model's intercept; raising the bias changes which models the family can express at all.

The tradeoff is that the usual ways of lowering one raise the other, so the sum has a minimum somewhere in the middle rather than at either extreme.

A distinction worth holding onto: bias and variance are properties of an *estimation procedure* across the training sets it might have seen, not of the one fitted model in front of you. You never observe them directly on a single fit. What you observe is their sum.

## Formal statement

Fix an input $\mathbf{x}$ and suppose the labels are generated as $y = f(\mathbf{x}) + \varepsilon$ with $\mathbb{E}[\varepsilon] = 0$ and $\operatorname{Var}(\varepsilon) = \sigma^{2}$. Let $\hat{h}_{D}$ be the model that the learning procedure produces from a random training set $D$. Under squared-error loss the expected error decomposes exactly:

$$\mathbb{E}_{D,\varepsilon}\Big[\big(y - \hat{h}_{D}(\mathbf{x})\big)^{2}\Big] = \underbrace{\Big(\mathbb{E}_{D}\big[\hat{h}_{D}(\mathbf{x})\big] - f(\mathbf{x})\Big)^{2}}_{\text{Bias}^{2}} \;+\; \underbrace{\mathbb{E}_{D}\Big[\big(\hat{h}_{D}(\mathbf{x}) - \mathbb{E}_{D}[\hat{h}_{D}(\mathbf{x})]\big)^{2}\Big]}_{\text{Variance}} \;+\; \underbrace{\sigma^{2}}_{\text{irreducible}}$$

Three readings that the prose version loses. The bias term enters **squared**, not raw, which is why the quantity being added is $\text{Bias}^{2}$ and why a bias of $-3$ and a bias of $+3$ cost the same. All three terms are non-negative, so $\sigma^{2}$ is a hard floor: no procedure, however good, gets expected error below it, and a model reported as achieving less than the noise level is measuring something other than what it claims. And this is an identity, derived from nothing but $\mathbb{E}[\varepsilon] = 0$ and independence of $\varepsilon$ from $D$, so it holds for every estimator without qualification, not just well-behaved ones.

Total [[Generalization]] error is this quantity averaged over the input distribution.

### How each term responds to more data

Only the variance term shrinks with $m$, and it does so at a rate of $1/m$ that [[Insufficient Training Data]] writes down for least squares. The squared bias does not move at all, because it depends on the model family and not on the sample size, and $\sigma^{2}$ is a property of the data-generating process. This one asymmetry is the whole reason a [[Learning Curve]] can answer "would more data help": more data buys down exactly one of the three terms, so a model whose error is mostly bias has nothing to gain from it.

### Squared error only

The clean three-way identity is specific to squared-error loss. Under the 0-1 loss that [[Accuracy]] scores a classifier by, no decomposition of this form holds; the proposals that exist are different in kind and do not give three non-negative terms that add up.

So for a classifier, "high bias" and "high variance" stay useful as a vocabulary for the two failure shapes, and the arithmetic does not carry over: there are no three numbers to add up, and an error rate read off a [[Confusion Matrix]] does not decompose into them.

### How much of a tradeoff

The familiar picture, bias falling and variance rising as capacity increases, their sum tracing a U with the best model at the bottom, is a strong and useful tendency rather than a theorem. One qualification keeps it honest.

Parameter count is a bad proxy for the capacity that matters. [[Regularization]] lowers variance while leaving the parameter count untouched; [[Ensemble Learning|averaging]] several models lowers the variance term specifically while leaving any bias the members share exactly intact. Complex models tend to be high-variance, but the converse fails, and a high-variance model is not thereby complex.

## Where it is used

This is the decomposition of the error that [[Generalization]] defines, and it is what gives the two named failures a common cause rather than leaving them as a pair of symptoms. [[Overfitting]] is the variance-dominated regime, where the fitted model tracks its particular training set too closely; [[Underfitting]] is the bias-dominated regime, where the family is wrong before any data arrives. Because the terms respond to different interventions, naming the regime is what makes the next step non-arbitrary.

The [[Learning Curve]] is how you find out which regime you are in without ever computing the terms, since the gap between its two curves stands in for variance and the height of the plateau they converge to stands in for bias plus noise. [[Regularization]] is the dial that spends bias to buy variance, and degree in [[Polynomial Regression]] is the same dial pointed the other way, which is why a degree-$10$ fit on $100$ noisy points is the standing illustration. [[Ensemble Learning]] attacks the variance term alone, which is precisely why it is the answer for an overfitting base learner and no answer at all for an underfitting one. And [[Insufficient Training Data]] is the case where variance dominates for a reason that has nothing to do with the model: at small $m$ the $1/m$ term is simply large, and it is the one term more data fixes.
