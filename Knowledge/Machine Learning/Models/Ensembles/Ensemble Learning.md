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
  - "[[HOML Ch03 Classification]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

An ensemble combines the predictions of several models, called base learners or weak learners, into one prediction that is better than any of them alone. The gain comes from disagreement: if the members made the same mistakes there would be nothing to pool, so diversity among base learners is the resource being spent, and every ensemble method is a scheme for manufacturing it.

The standing evidence is a random forest, an ensemble of decision trees each fitted on a resampled subset of the data and a random subset of the [[Feature|features]]. On the California housing regression problem the single tree cross-validates at about $66{,}900$ RMSE and the linear model at about $69{,}900$, while the forest reaches about $47{,}000$, all under 10-fold cross-validation.

Four families produce the diversity in different ways. [[Bagging]] fits the same algorithm to bootstrap resamples and averages. [[Boosting]] fits members in sequence, each one concentrating on what its predecessors got wrong. [[Stacking]] trains a further model to combine the members' outputs. *Voting* simply pools the predictions of unrelated algorithms. Each of the first three carries its own mechanism, its own knobs and its own failure modes in its own note; what follows here is only the argument for why any of them works, which is the part all four share. What is still outstanding is the treatment of the members as models in their own right, decision trees and random forests with their fitting, their splitting criteria and their feature importances, and that arrives with HOML chapters 6 and 7.

## Formal statement

Take $M$ base learners whose predictions are individually unbiased, each with error variance $\sigma^{2}$, with average pairwise correlation $\rho$ between their errors. Averaging them gives

$$\operatorname{Var}\left(\frac{1}{M}\sum_{j=1}^{M} \hat{y}_j\right) = \rho\sigma^{2} + \frac{1-\rho}{M}\,\sigma^{2}$$

Two readings follow. Only the second term shrinks with $M$, so adding members past a point buys almost nothing. And as $M \to \infty$ the variance does not go to zero but to $\rho\sigma^{2}$: the correlation between the members is a floor no amount of committee can get under. Independent members ($\rho = 0$) recover the familiar $\sigma^{2}/M$; identical members ($\rho = 1$) give $\sigma^{2}$, which is to say the ensemble is one model copied $M$ times.

The $\rho$ in that expression is also a rule about what to put in the ensemble, and it is the one piece of practical advice the formula yields directly. Since the floor is set by how much the members' errors agree, the members should be as unlike each other as you can make them, which in practice means picking genuinely different model families rather than refitting one family with different settings. Two gradient-boosted tree models on the same feature matrix are wrong in the same places and their average inherits those mistakes whole; a tree model, a linear model and a nearest-neighbour model are wrong in different places, and the average of those sits on a lower floor. Each of the four families is a scheme for pushing $\rho$ down without pushing $\sigma^{2}$ up, and the choice of which model families to enrol is the part of that you do by hand.

## Where it is used

Because averaging attacks variance rather than bias, ensembling is the standard answer to a low-bias, high-variance base learner: a fully grown decision tree fits its training data perfectly and generalizes badly ([[Overfitting]]), and bagging many such trees keeps the flexibility while cancelling much of the noise. It does little for a member that is systematically wrong in the same direction everywhere ([[Underfitting]]), since the formula assumes unbiased members and averaging preserves a shared bias intact. Bias comes down only through a different mechanism, the sequential fitting boosting uses, where each member is trained against what its predecessors got wrong.

In practice the ensemble enters as one more candidate in [[Model Selection]], scored by [[Cross-Validation]] exactly like any single model, and tuned by [[Randomized Search]] over both its own count of members and the [[Hyperparameter|hyperparameters]] of its base learner. Its cost is interpretability and compute: $M$ models to fit, store, and call for every prediction.

That cost is operational as well as computational, which is what decides whether an ensemble is worth shipping rather than merely worth trying. An ensemble is more complex to deploy and harder to maintain than a single model, since every member is a separate artifact that has to be versioned, monitored and kept in step with the others, and the gain being bought is usually small in absolute terms. What keeps the trade alive is that the gain is not always small in value: where a fraction of a point of performance converts into a large amount of money, the extra machinery pays for itself, and that is the setting where ensembles are standard practice rather than a curiosity.

`RandomForestClassifier` fills the same role in classification, as the second classifier against which a linear one is measured, on one target and one set of folds so that the comparison means something. The forest wins decisively and on every reading at once: a [[ROC Curve|ROC AUC]] of about $0.9983$ against about $0.9605$, and [[Precision]] of about $0.9897$ with [[Recall]] of about $0.8725$ against about $0.8371$ and about $0.6512$. It has to be scored through `predict_proba`, its positive-class column standing in for a score, because it has no `decision_function`; which response method a model owes its caller is the practical edge of the [[Scikit-Learn Estimator API]], and here it decides how an ensemble can be put on a curve at all.
