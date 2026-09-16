---
note_kind: concept
aliases:
  - ensemble
  - ensembles
  - ensemble methods
  - base learner
  - weak learner
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## Definition

An ensemble combines the predictions of several models, called base learners or weak learners, into one prediction that is better than any of them alone. The gain comes from disagreement: if the members made the same mistakes there would be nothing to pool, so diversity among base learners is the resource being spent, and every ensemble method is a scheme for manufacturing it.

The chapter's evidence is a random forest, which is an ensemble of decision trees each fitted on a resampled subset of the data and a random subset of the [[Feature|features]]. On the same problem the single tree cross-validates at about $66{,}900$ RMSE and the linear model at about $69{,}900$, while the forest reaches about $47{,}000$, all under 10-fold cross-validation.

Four families produce the diversity in different ways. *Bagging* fits the same algorithm to bootstrap resamples and averages. *Boosting* fits members in sequence, each one concentrating on what its predecessors got wrong. *Stacking* trains a further model to combine the members' outputs. *Voting* simply pools the predictions of unrelated algorithms. The mechanisms, and the reasons to prefer one over another, are the subject of a later chapter; what follows is only the argument for why any of them works.

## Formal statement

Take $M$ base learners whose predictions are individually unbiased, each with error variance $\sigma^{2}$, with average pairwise correlation $\rho$ between their errors. Averaging them gives

$$\operatorname{Var}\left(\frac{1}{M}\sum_{j=1}^{M} \hat{y}_j\right) = \rho\sigma^{2} + \frac{1-\rho}{M}\,\sigma^{2}$$

Two readings follow. Only the second term shrinks with $M$, so adding members past a point buys almost nothing. And as $M \to \infty$ the variance does not go to zero but to $\rho\sigma^{2}$: the correlation between the members is a floor no amount of committee can get under. Independent members ($\rho = 0$) recover the familiar $\sigma^{2}/M$; identical members ($\rho = 1$) give $\sigma^{2}$, which is to say the ensemble is one model copied $M$ times.

## Where it is used

Because averaging attacks variance rather than bias, ensembling is the standard answer to a low-bias, high-variance base learner: a fully grown decision tree fits its training data perfectly and generalizes badly ([[Overfitting]]), and bagging many such trees keeps the flexibility while cancelling much of the noise. It does little for a member that is systematically wrong in the same direction everywhere ([[Underfitting]]), since the formula assumes unbiased members and averaging preserves a shared bias intact. Bias comes down only through a different mechanism, the sequential fitting boosting uses, where each member is trained against what its predecessors got wrong.

In practice the ensemble enters as one more candidate in [[Model Selection]], scored by [[Cross-Validation]] exactly like any single model, and tuned by [[Randomized Search]] over both its own count of members and the [[Hyperparameter|hyperparameters]] of its base learner. Its cost is interpretability and compute: $M$ models to fit, store, and call for every prediction.
