---
note_kind: concept
aliases:
  - class imbalance
  - imbalanced classes
  - imbalanced data
  - imbalanced dataset
  - imbalanced classification
  - unbalanced classes
  - skewed classes
  - class skew
  - rare positive class
  - rare class
  - majority class
  - minority class
  - class prior
  - imbalance ratio
up: "[[Training Set]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

A training set is class imbalanced when its classes are not present in comparable numbers, so one class dominates whatever the model is optimizing. The model is not being stubborn when it ignores the rare class: predicting the majority really is the cheapest way to lower an error that counts every instance equally, and the rare class contributes too few instances for getting them wrong to cost much.

## Formal statement

With $m$ instances and $m_k$ in class $k$, the class prior is the share each class holds,

$$\pi_k = \frac{m_k}{m}, \qquad \sum_k \pi_k = 1$$

and the imbalance ratio compares the extremes,

$$\text{IR} = \frac{\max_k m_k}{\min_k m_k}$$

$\text{IR} = 1$ is perfect balance and the number grows without bound as the rare class thins out. The consequence that matters is the one a constant predictor demonstrates: always answering with the majority class gives

$$\text{acc}_{\text{majority}} = \max_k \pi_k$$

so the more imbalanced the target, the higher the accuracy available to a model that has learned nothing.

The [[MNIST]] 5-versus-rest target is a worked case. Of the $60{,}000$ training images, $5{,}421$ are fives and $54{,}579$ are not, so

$$\pi_{5} = \frac{5{,}421}{60{,}000} = 0.09035, \qquad \pi_{\text{not }5} = 0.90965, \qquad \text{IR} = \frac{54{,}579}{5{,}421} \approx 10.07$$

A [[Baseline Model]] that never predicts a 5 therefore cross-validates to $0.90965$ accuracy. That is only a tenfold imbalance, mild next to fraud or rare-disease detection where $\text{IR}$ runs to $10^{3}$ or more, and it is already enough to make accuracy unreadable. A [[Stochastic Gradient Descent Classifier]] fitted on that target scores $[0.95035, 0.96035, 0.9604]$ across three folds, and its [[Confusion Matrix]] shows why the number flatters it: $1{,}891$ of the $5{,}421$ fives are missed, a [[Recall]] of $3{,}530 / 5{,}421 = 0.651$, which no reading of the accuracy would have revealed.

## Where it is used

It is a property of the [[Training Set]] you check before choosing how to score anything, by counting the labels. It is the condition under which [[Accuracy]] stops being informative, since the constant predictor already takes $\max_k \pi_k$ of it and the whole remaining range is the part worth arguing about. What you look at instead is the [[Confusion Matrix]], which keeps the two error types apart, and the measures read off it: [[Precision]] and [[Recall]] survive imbalance because both are conditioned on a single row or column of that matrix and so cannot be inflated by a large true-negative count. A [[Baseline Model]] is what makes the problem visible in one line, by putting $\max_k \pi_k$ on the same page as the model's score.

On the data side, [[Stratified Sampling]] is how a split preserves $\pi_k$ in each part rather than leaving it to chance, which matters most when the rare class is small enough that a plain random draw could underrepresent it or miss it outright. The same argument makes stratified folds the default for [[Cross-Validation]] on a classification target. Within the model, `class_weight="balanced"` reweights each instance by $m / (K \, m_k)$, which is the inverse of its class's prior up to a constant, so the rare class regains the influence its count denied it.

## VS

The word "skew" gets used for both of these and they are not the same thing.

- **This note**, class imbalance, is about the *counts of the label*: how many rows carry each class. It is a property of $\mathbf{y}$, it is measured by the priors $\pi_k$, and it breaks accuracy.
- **[[Skewed Data]]** is about the *shape of a numeric feature's distribution*: values piled on one side of the range with a long tail on the other. It is a property of a column of $\mathbf{X}$, it is measured by the third standardized moment $\gamma_1$, and it is fixed with a [[Feature Distribution Transformation]].

A dataset can have either, both, or neither, and neither remedy touches the other problem. On the [[California Housing|housing]] data the word means the second sense throughout. On an [[MNIST]] 5-versus-rest target it means the first.
