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
  - "[[DMLS Ch04 Training Data]]"
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

### What makes the imbalance bite

$\text{IR}$ alone does not predict how much damage is done, and the claim that it does is the one worth being careful about. Japkowicz and Stephen (*Intelligent Data Analysis* 6(5), 2002, 429 to 449) varied three things independently on artificially generated domains, the imbalance level, the complexity of the target concept, and the overall size of the training set, and concluded that the harm is a joint function of all three together with the learner rather than of the ratio alone. Concept complexity there is the number of disjoint subclusters the concept is built from, and the measure they suggest for carrying it over to a real dataset is $C = \log_2 L$, with $L$ the number of leaves a C4.5 tree grows on that dataset.

What their experiments support, with the conditions attached:

- **A simple enough concept is unharmed at any imbalance level tested.** Class imbalance did not hinder classification of the simple domains, linearly separable ones among them. This is the strongest version of the claim that survives, and it is a statement about the concept being simple, not about the model being linear: a linear model on a concept it cannot represent is not the case being described.
- **Sensitivity rises with concept complexity and falls with training set size.** With every subcluster holding 50 instances, the error stayed below $1\%$ at the highest complexity tested and at every imbalance level, which is the size effect on its own.
- **So the ratio is a proxy rather than a cause.** Splitting a rare class across several subclusters leaves each subcluster with too few instances to be learned, and it is that small-disjunct problem, in complex and small domains, that the imbalance ratio stands in for. Imbalance per se, in a large simple domain, costs nothing.
- **The learner is the fourth factor.** Across that range C5.0 was the most sensitive, a multi-layer perceptron showed a less consistent pattern, and a support vector machine appeared insensitive.

Two conditions travel with all four. The evidence is from artificial domains constructed to move one factor at a time, so none of the numbers transfers to a particular real dataset without being re-measured on it; and "complexity" throughout means complexity of the concept in the sense above, not the number of features and not the capacity of the model.

Binary imbalance is also the easy case: with $K > 2$ there is no single $\pi_{\min}$ to protect, several classes can be rare at once and rare against different rivals, and every remedy has to be given a target for each class rather than a ratio.

## Where it is used

It is a property of the [[Training Set]] you check before choosing how to score anything, by counting the labels. It is the condition under which [[Accuracy]] and its complement the error rate stop being informative, since the constant predictor already takes $\max_k \pi_k$ of it and the whole remaining range is the part worth arguing about. What you look at instead is the [[Confusion Matrix]], which keeps the two error types apart, and the measures read off it: [[Precision]] and [[Recall]] survive imbalance because both are conditioned on a single row or column of that matrix and so cannot be inflated by a large true-negative count. A [[Baseline Model]] is what makes the problem visible in one line, by putting $\max_k \pi_k$ on the same page as the model's score.

At the sharp end it stops being a scoring problem and becomes a coverage one. A class whose $m_k$ is a handful is not a badly weighted class, it is a class the model has barely seen, so the task it poses locally is the one [[Few-Shot Learning]] is named for: that note's own contrast puts its axis at coverage of the target classes rather than at supervision, and severe imbalance is one of the ordinary ways a dataset arrives with full supervision and no coverage. Reading a rare class that way changes what you would do about it, since no reweighting of five instances manufactures a sixth.

On the data side, [[Stratified Sampling]] is how a split preserves $\pi_k$ in each part rather than leaving it to chance, which matters most when the rare class is small enough that a plain random draw could underrepresent it or miss it outright. The same argument makes stratified folds the default for [[Cross-Validation]] on a classification target. Within the model, `class_weight="balanced"` reweights each instance by $m / (K \, m_k)$, which is the inverse of its class's prior up to a constant, so the rare class regains the influence its count denied it; that switch is the one-line entry to [[Loss Reweighting]], which owns the family it belongs to.

The two families of remedy divide on what they are allowed to touch, and naming the division is most of the decision. [[Resampling]] is the data-level answer: it duplicates, synthesises or discards training rows until the class proportions are what you want, changing the data the optimizer sees and leaving the objective exactly as written. [[Loss Reweighting]] is the algorithm-level answer: it leaves every row where it was collected and changes what each row is worth in the objective, so what changes is what the optimizer is rewarded for. A third position is to accept the imbalance and fix the reading of it instead, which is the metric argument above and costs nothing.

Whichever family is chosen, the same rule applies to the measurement: **a model is never evaluated on resampled data.** The rebalancing is applied to the training split after the split, and the [[Testing Set]] and every validation fold keep the class mix the deployment data has, because a score measured at priors you manufactured is a score on a distribution nobody will send you. [[Resampling]] carries the mechanics and the leak it causes when the order is wrong.

### Whether to correct it at all

There is a real position, held mostly about large models, that the imbalance should be left alone. If the rare class is rare in the world, then the training distribution is not a defect to be repaired but the thing the model is supposed to learn, priors included; a model that has learned it will answer with the right base rate unprompted, and rebalancing the data deliberately teaches it a base rate that is false. The argument gets stronger as models get larger, since capacity is what lets a model represent a rare region well enough that its scarcity stops being fatal, which is the same direction the small-disjunct finding above points in. The concession the position makes is the whole of the practical objection: building a model good enough to learn the imbalance rather than collapse onto the majority is hard, and if you are not going to get one, the counts are still what the optimizer will follow. It is a position, not a settled result, and the evidence to settle it is not in this note.

## VS

The word "skew" gets used for both of these and they are not the same thing.

- **This note**, class imbalance, is about the *counts of the label*: how many rows carry each class. It is a property of $\mathbf{y}$, it is measured by the priors $\pi_k$, and it breaks accuracy.
- **[[Skewed Data]]** is about the *shape of a numeric feature's distribution*: values piled on one side of the range with a long tail on the other. It is a property of a column of $\mathbf{X}$, it is measured by the third standardized moment $\gamma_1$, and it is fixed with a [[Feature Distribution Transformation]].

A dataset can have either, both, or neither, and neither remedy touches the other problem. On the [[California Housing|housing]] data the word means the second sense throughout. On an [[MNIST]] 5-versus-rest target it means the first.
