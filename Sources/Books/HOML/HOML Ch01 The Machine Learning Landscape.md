---
note_kind: source
medium: book
chapter: 1
up: "[[HOML]]"
url: https://github.com/ageron/handson-ml3/blob/main/01_the_machine_learning_landscape.ipynb
aliases:
  - HOML Chapter 1
  - The Machine Learning Landscape
---

## Thesis

A machine learning system is a program whose performance on a task improves with data, and the whole field can be laid out along three independent axes: how much supervision the data carries, whether learning happens in one batch or as a stream, and whether the system memorizes instances or fits a parametric model. Almost everything that goes wrong is either bad data (too little, unrepresentative, noisy, irrelevant, or mismatched to production) or a bad fit (over- or underfitting), and the only honest estimate of generalization comes from data the model never trained on.

## Extracted

Definition and vocabulary:
- [[Machine Learning]]
- [[Model]]
- [[Feature]]
- [[Training Set]]
- [[Training Instance]]
- [[Hyperparameter]]

Supervision:
- [[Supervised Learning]]
- [[Unsupervised Learning]]
- [[Semi-Supervised Learning]]
- [[Self-Supervised Learning]]

Training regime:
- [[Batch Learning]]
- [[Online Learning]]
- [[Out-of-Core Learning]]

Optimization:
- [[Learning Rate]]

Representation:
- [[Instance-Based Learning]]
- [[Model-Based Learning]]

Supervised tasks:
- [[Classification]]
- [[Regression]]

Unsupervised tasks:
- [[Clustering]]
- [[Dimensionality Reduction]]
- [[Anomaly Detection]]
- [[Novelty Detection]]
- [[Association Rule Learning]]

Models named:
- [[Linear Regression]]
- [[Logistic Regression]]

Objectives:
- [[Performance Measure]]
- [[Utility Function]]
- [[Cost Function]]

Challenges, bad data:
- [[Insufficient Training Data]]
- [[Unreasonable Effectiveness of Data]]
- [[Nonrepresentative Training Data]]
- [[Poor-Quality Data]]
- [[Irrelevant Features]]
- [[Data Mismatch]]

Challenges, bad algorithm:
- [[Overfitting]]
- [[Underfitting]]

Testing and validating:
- [[Generalization]]
- [[Testing Set]]
- [[Holdout Validation]]
- [[Cross-Validation]]
- [[Model Selection]]

Production:
- [[Model Rot]]

## Open questions

- The chapter also covers regularization, the no free lunch theorem, and reinforcement learning. Regularization is now written: [[HOML Ch04 Training Models]] brought [[Regularization]] as the category, with a penalty on the weights and a rule that stops the fit early as its instances. The other two are still absent, and each needs a note in `Generalization/` or `Reinforcement Learning/`.
- "Learning rate" in this chapter means how fast an online system adapts; in chapter 4 it means the gradient descent step size. Is the same number doing both jobs in SGD, or are these two different knobs? Chapter 3 did not settle it: it introduces the [[Stochastic Gradient Descent Classifier]] and fits it, but takes the optimizer that names the estimator entirely as given and never covers gradient descent. [[HOML Ch04 Training Models]] settles it. They are one knob: a squared-error stochastic update on a single instance collapses to an exponential moving average of the targets with smoothing factor $2\eta$, so the step size and the forgetting rate are two readings of the same coefficient, and the effective memory is about $1/(2\eta)$ instances. The derivation, and the two riders that keep it from being glib, are in [[Learning Rate]].
- Is novelty detection a separate task from anomaly detection, or the same task with a clean training set? Chapter 9 should settle it.
- Association rule learning gets no later HOML chapter. Does it stay a single note under `Tasks/`?
