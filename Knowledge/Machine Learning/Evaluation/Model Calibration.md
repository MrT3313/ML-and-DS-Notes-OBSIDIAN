---
note_kind: concept
aliases:
  - model calibration
  - calibration
  - calibrated
  - calibrated probabilities
  - miscalibration
  - miscalibrated
  - calibration curve
  - reliability diagram
  - Platt scaling
  - isotonic regression
  - expected calibration error
  - ECE
  - Brier score
  - brier_score_loss
  - CalibratedClassifierCV
  - calibration_curve
  - temperature scaling
up: "[[Performance Measure]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

A model is calibrated when the probabilities it reports are the frequencies its predictions actually come true at: among the instances it scores at $0.8$, about eight in ten should turn out to be positives. Measuring it is a matter of counting, and the simplest method is exactly that: count the number of times the model outputs the probability $X$, count the frequency $Y$ of that prediction coming true, and plot $X$ against $Y$.

## Formal statement

### Perfect calibration is a condition on a whole function

Write $\hat{p}$ for the probability the model reports for the positive class. The model is perfectly calibrated when

$$P\big(Y = 1 \mid \hat{p} = p\big) = p \qquad \text{for all } p \in [0, 1]$$

This is what the counting procedure above estimates, and reading it carefully is most of the subject. It is a condition on the conditional distribution of the outcome given the reported number, so it has to hold at every $p$ the model ever emits, not on average and not at one operating point. It says nothing whatever about how often the model is right: it constrains only the relationship between the number it prints and the frequency behind that number.

On a [[Multiclass Classification]] target the same condition is written for the class the model actually commits to, which is the weaker and more commonly measured form,

$$P\big(Y = \hat{y} \mid \hat{p}_{\max} = p\big) = p, \qquad \hat{p}_{\max} = \max_k \hat{p}_k$$

and it is weaker because it constrains only the winning class's number. The strong form asks for $P(Y = k \mid \hat{p}_k = p) = p$ for every one of the $K$ classes at once, which a model can fail while the winning class's number behaves.

### The reliability diagram is that conditional, estimated by binning

$\hat{p}$ is continuous, so $\hat{p} = p$ is an event of probability zero and the conditional above cannot be counted directly: no two instances share a score exactly. Binning is the estimator. Partition $[0, 1]$ into $M$ bins $B_1, \dots, B_M$, let $n_b = |B_b|$ be the count landing in bin $b$ out of $n$ instances, and take two averages per bin. The first is the mean probability claimed inside the bin,

$$\text{conf}_b = \frac{1}{n_b} \sum_{i \in B_b} \hat{p}^{(i)}$$

and the second is the frequency that claim came true at, which is written $\text{acc}_b$ and has two readings depending on which form of the condition is being checked:

$$\text{acc}_b = \frac{1}{n_b} \sum_{i \in B_b} y^{(i)} \quad \text{(binary, binning } \hat{p} \text{)}, \qquad \text{acc}_b = \frac{1}{n_b} \sum_{i \in B_b} \mathbb{1}\!\left[\, \hat{y}^{(i)} = y^{(i)} \,\right] \quad \text{(binning } \hat{p}_{\max} \text{)}$$

The first is the fraction of positives in the bin and the second is the fraction of committed predictions that were right, and they are the same quantity read against the two forms of the condition above. Plotting $\text{acc}_b$ against $\text{conf}_b$ is the counting procedure with a bin standing in for a single value of $p$, and that plot is the **reliability diagram**, also called the calibration curve. Perfect calibration is the diagonal $\text{acc}_b = \text{conf}_b$. A curve below the diagonal means the model claims more than it delivers, which is overconfidence; a curve above it means underconfidence.

Two things the diagram hides and the numbers do not. A bin holding few instances gives an $\text{acc}_b$ estimated from few trials, so the vertical position of a point near either end of the range is frequently noise rather than miscalibration. And the diagram says nothing about how many instances sit in each bin, so a badly deviating point may be carrying twenty instances out of sixty thousand.

### Expected calibration error

Collapsing the diagram into one number is a weighted average of the vertical gaps, weighted by how much of the data each bin holds:

$$\text{ECE} = \sum_{b=1}^{M} \frac{n_b}{n} \Big\lvert \, \text{acc}_b - \text{conf}_b \, \Big\rvert$$

Zero exactly when every bin sits on the diagonal. The variant that reports the worst bin rather than the average is the maximum calibration error, $\max_b \lvert \text{acc}_b - \text{conf}_b \rvert$, and it is the right one when a single badly wrong region is the thing that matters.

$M$ is a choice and not a property of the model, which is the part that is easy to forget once a single number is on the slide. Too few bins average opposite errors inside one bin and report them as calibration that is not there. Too many leave each bin with too few instances for its frequency to mean anything, so the estimate inflates on noise. The binning strategy is a second choice on top of it, equal-width bins against equal-count bins, and the two give different numbers on the same predictions. The estimator is not neutral either. Kumar, Liang and Ma (NeurIPS 2019) find that models put through a continuous recalibration map, Platt scaling and temperature scaling among them, are less calibrated than the binned numbers report, and that the usual plugin estimator cannot say by how much. So a small ECE is weaker evidence than it looks, and reporting $M$ and the binning strategy alongside the number is what makes the number reproducible at all.

One more property worth stating, because it is the trap this measure sets. A model that ignores its input and reports the base rate for every instance has $\text{acc}_b = \text{conf}_b$ in its single occupied bin and therefore $\text{ECE} = 0$. Calibration alone certifies nothing about usefulness.

### The Brier score, and what Murphy's decomposition separates

The Brier score (Brier 1950) is the mean squared error taken on probabilities rather than on predicted values, with the outcome coded $y \in \{0, 1\}$:

$$\text{BS} = \frac{1}{n} \sum_{i=1}^{n} \Big( \hat{p}^{(i)} - y^{(i)} \Big)^{2}$$

It lies in $[0, 1]$ with zero the best attainable, and like [[Log Loss]] it is a strictly proper scoring rule, so it is minimized uniquely by reporting the true probabilities. Unlike log loss it is bounded, so a confidently wrong prediction costs at most $1$ rather than diverging.

What makes it more than a second loss is that it splits. Murphy (1973) partitions it over the same bins the reliability diagram uses, writing $\bar{o}_b$ for the observed frequency of positives in bin $b$, which is $\text{acc}_b$ under its binary reading, and $\bar{o}$ for the overall base rate:

$$\text{BS} = \underbrace{\frac{1}{n}\sum_{b} n_b \big( \text{conf}_b - \bar{o}_b \big)^{2}}_{\text{reliability}} \; - \; \underbrace{\frac{1}{n}\sum_{b} n_b \big( \bar{o}_b - \bar{o} \big)^{2}}_{\text{resolution}} \; + \; \underbrace{\bar{o}\,(1 - \bar{o})}_{\text{uncertainty}}$$

Read the three terms by what each one measures. **Reliability** is the squared version of the ECE gap, so it is miscalibration and you want it near zero. **Resolution** is how far the bins' outcome frequencies spread away from the base rate, so it measures whether the model separates instances at all, and it enters with a minus sign, meaning you want it large. **Uncertainty** is fixed by the labels alone, is the Brier score of always predicting the base rate, and no model can move it.

The partition is exact when the forecasts take finitely many distinct values and each group holds one value, which is the case Murphy states it for. Under binning it inherits the bin count the same way the ECE does, so the three terms shift with $M$ even though their sum does not.

The decomposition is the formal statement of the point the next section makes in words: a single Brier score mixes calibration and discrimination into one number, and only the partition tells you which of the two a model is losing on.

### Calibration and discrimination are different properties

**Discrimination** is whether the score orders instances correctly, which is the only thing a threshold sweep can see. **Calibration** is whether the numbers it prints are frequencies, which is the only thing this note measures. Neither implies the other, and both directions of the failure are easy to construct.

**A model can rank perfectly and be badly calibrated.** Take any scoring function and halve every probability it reports. The map $p \mapsto p/2$ is strictly increasing, so every ordering is preserved, every reading the [[ROC Curve]] takes is unchanged, since AUC depends only on the order the scores induce, and every point on the [[Precision-Recall Tradeoff]]'s sweep survives with a relabelled threshold. The reliability diagram, meanwhile, now runs at twice the diagonal's height, $\text{acc}_b \approx 2\,\text{conf}_b$, which is systematic underconfidence, and the ECE is large. Nothing got worse in the sense those two notes measure, and the probabilities are now lies.

**A model can be perfectly calibrated and useless at ranking.** The constant predictor that reports the base rate $\pi$ for every instance satisfies $P(Y = 1 \mid \hat{p} = \pi) = \pi$ exactly. Its ECE is zero, its Brier reliability term is zero, and its AUC is $0.5$. In the decomposition it loses everything on resolution, which is exactly the term that measures the thing it does not do.

The practical form of the split is which repair is available. A model that discriminates but is miscalibrated can be fixed after the fact by a monotone map applied to its scores, and the three such maps are below. A model that does not discriminate cannot be fixed that way at all, because no monotone map creates an ordering that was not already there.

### Recalibration fits a map on held-out scores

All three methods leave the fitted model alone and fit a map $g$ from its scores to probabilities. The map is fitted by minimizing a proper scoring rule against the labels, and the scores it is fitted on must be scores the model did not fit on.

That last requirement is not a detail. A model has partly memorized its own training rows, so its in-sample scores are more often correct than its out-of-sample scores at the same value. A map fitted on them learns a distortion that does not exist on fresh data and pushes the output toward $0$ and $1$, which is the direction that makes the calibrated model worse than the uncalibrated one. [[Cross-Validation]] is where honest scores come from, one out-of-fold score per training instance.

### Platt scaling

Fit a one-dimensional [[Logistic Regression]] on the score $s$, two parameters and nothing else:

$$\hat{p} = \sigma(a s + b), \qquad \sigma(z) = \frac{1}{1 + e^{-z}}$$

$a$ and $b$ are chosen to minimize the [[Log Loss]] of the mapped probabilities against the held-out labels. Platt (1999) writes the same family as $1/(1 + \exp(As + B))$, which is $\sigma(-(As+B))$, the identical two-parameter map with the signs of the pair flipped.

One detail of his fit is worth carrying because it is what the library actually implements. The targets are not $1$ and $0$ but smoothed toward the interior,

$$t_{+} = \frac{N_{+} + 1}{N_{+} + 2}, \qquad t_{-} = \frac{1}{N_{-} + 2}$$

with $N_{+}$ and $N_{-}$ the positive and negative counts in the calibration set. Platt derives them from an uninformative prior on the true probability rather than from a heuristic, and the effect is to bound the fit away from the step function that hard $0$ and $1$ targets produce on separable scores, where $a$ would otherwise run off toward infinity. The smoothing matters more the smaller the calibration set is, which is the regime where this map is used in the first place.

Two consequences follow from it having two parameters. It needs very little data, so it is the right default on a small calibration set. And it can only undo a distortion of sigmoid shape, so a model miscalibrated some other way comes back still miscalibrated, with the residual invisible unless the reliability diagram is plotted again after the map. Being strictly monotone whenever $a > 0$, it cannot change the ranking, so AUC is exactly preserved.

### Isotonic regression

The non-parametric alternative. Fit the non-decreasing function $g$ that minimizes squared error against the labels,

$$\min_{g} \sum_{i=1}^{n} \Big( y^{(i)} - g\big(s^{(i)}\big) \Big)^{2} \quad \text{subject to} \quad s^{(i)} \le s^{(j)} \implies g\big(s^{(i)}\big) \le g\big(s^{(j)}\big)$$

The solution is a step function and is computed by the pool adjacent violators algorithm, which is linear in $n$ once the scores are sorted. Zadrozny and Elkan introduced the binned estimator for this purpose (ICML 2001) and then isotonic regression itself along with the reduction from multiclass to a set of binary calibrations (KDD 2002).

The shape constraint is the only assumption, so it corrects any monotone distortion rather than only a sigmoid one, and that flexibility is paid for in data. With one step per pooled block it has far more effective parameters than Platt scaling and it overfits a small calibration set badly, the usual figure being that it wants something in the low thousands of calibration instances before it is preferable. It is also non-decreasing rather than strictly increasing, so it ties scores together where it flattens, and those ties can change the ranking: AUC is not guaranteed to survive it.

### Temperature scaling

The neural network case, and a one-parameter special case of the above. Divide the logits $z_1, \dots, z_K$ by a single positive scalar before the softmax:

$$\hat{p}_k = \frac{e^{z_k / T}}{\sum_{j=1}^{K} e^{z_j / T}}$$

$T$ is fitted on a held-out split by minimizing log loss, and it is the whole of what is fitted. $T > 1$ flattens the distribution toward uniform, $T < 1$ sharpens it toward a one-hot vector, and $T = 1$ is the original model. Because dividing by a positive constant preserves the order of the logits, the $\arg\max$ is untouched and accuracy is exactly unchanged, which is the property that makes it safe to apply to a model already in use.

Guo, Pleiss, Sun and Weinberger (ICML 2017) are the reference for it and for the finding that motivates it: modern networks are systematically miscalibrated in the overconfident direction, and the architecture choices that raised accuracy are implicated in it. Increasing depth, increasing width and adding batch normalization each make calibration worse, and so does reducing weight decay, which is the one that runs the other way, since the [[Regularization]] that hurts accuracy is the thing that was holding calibration together. Their result is that this single parameter is usually enough to repair the damage, which is itself evidence that what those networks suffer is a uniform sharpening rather than a structured distortion.

Which families need repair at all, and in which direction, is Niculescu-Mizil and Caruana's (ICML 2005) subject, and it is the study scikit-learn's own user guide is built on. Maximum-margin methods and boosted trees push their scores toward the middle of the range and produce a sigmoid-shaped reliability curve. Naive Bayes pushes the other way, toward $0$ and $1$, because its independence assumption multiplies evidence it has counted more than once. Bagged trees and random forests avoid the extremes because averaging over variable members pulls predictions away from the ends. Logistic regression is close to calibrated already, which follows from what it minimizes rather than from luck.

### In scikit-learn

scikit-learn 1.6 splits the subject across two modules: `sklearn.calibration` holds the diagram and the recalibrators, `sklearn.metrics` holds the score.

```python
import matplotlib.pyplot as plt
from sklearn.calibration import CalibratedClassifierCV, calibration_curve
from sklearn.metrics import brier_score_loss
from sklearn.model_selection import cross_val_predict

y_prob = cross_val_predict(clf, X_train, y_train, cv=5,
                           method="predict_proba")[:, 1]

# X is the mean claimed probability per bin, Y is the frequency it came true
prob_true, prob_pred = calibration_curve(y_train, y_prob, n_bins=10,
                                         strategy="uniform")
plt.plot(prob_pred, prob_true)          # note the order, see below
plt.plot([0, 1], [0, 1], "k:")          # the diagonal, perfect calibration

brier_score_loss(y_train, y_prob)

calibrated = CalibratedClassifierCV(clf, method="sigmoid", cv=5)
calibrated.fit(X_train, y_train)
```

`calibration_curve(y_true, y_prob, *, pos_label=None, n_bins=5, strategy='uniform')` returns the pair `(prob_true, prob_pred)`, in that order, where `prob_true` is $\text{acc}_b$ and `prob_pred` is $\text{conf}_b$. The return order is the reverse of the plotting order, which is the trap: the horizontal axis is the second returned array. `n_bins` is $M$, defaulting to $5$, which is low enough that a curve plotted at the default can miss a distortion entirely. Empty bins are dropped rather than returned as gaps, so the returned arrays are frequently shorter than `n_bins` and their length is data dependent. `strategy="uniform"` gives equal-width bins and `strategy="quantile"` gives equal-count bins, the second being the one that keeps $n_b$ from collapsing in the sparse tails.

`brier_score_loss(y_true, y_proba=None, *, sample_weight=None, pos_label=None, y_prob='deprecated')` is the 1.6 signature. The second positional parameter is `y_proba`; the older name `y_prob` still exists as a deprecated keyword, deprecated in 1.5 and scheduled for removal in 1.7, so code naming it warns now and breaks later. There is no decomposition in the library: the reliability and resolution terms have to be computed from `calibration_curve`'s output by hand.

`CalibratedClassifierCV(estimator=None, *, method='sigmoid', cv=None, n_jobs=None, ensemble='auto')` is the recalibrating wrapper, and it is an estimator like any other under the [[Scikit-Learn Estimator API]], so it drops into a [[Pipeline]] and is cross-validated as one unit. `method` is `"sigmoid"` for Platt scaling or `"isotonic"`; the default is the right one on small calibration sets for the reason given above. `cv=None` means 5-fold, stratified on a classification target. `ensemble` decides what the wrapper is at prediction time: `True` fits one classifier and one calibrator per fold and averages their probabilities, `False` collects out-of-fold scores through `cross_val_predict`, fits a single calibrator on them, and pairs it with the estimator refitted on all the data. `"auto"`, the 1.6 default, resolves to `False` for a frozen estimator and `True` otherwise.

**The version trap is the already-fitted estimator.** Calibrating a model that has already been fitted used to be `cv="prefit"`. In 1.6 that value is deprecated in favour of wrapping the fitted estimator, and an implementation written from the older recipe warns on this version and stops working on a later one. The 1.6 spelling:

```python
from sklearn.frozen import FrozenEstimator
from sklearn.calibration import CalibratedClassifierCV
from sklearn.model_selection import train_test_split

X_fit, X_calib, y_fit, y_calib = train_test_split(X_train, y_train,
                                                  random_state=42)

base = clf.fit(X_fit, y_fit)
calibrated = CalibratedClassifierCV(FrozenEstimator(base))
calibrated.fit(X_calib, y_calib)
```

`FrozenEstimator` arrived in 1.6 and makes `fit` a no-op on whatever it wraps, so the wrapper trains only the calibrator, on `X_calib`. The held-out requirement is then yours to satisfy by making that split yourself, which is exactly what it was under `cv="prefit"` and is the reason the split appears in the snippet rather than beside it.

## Where it is used

[[Log Loss]] is the training-time counterpart of what this note measures after the fact: it is a strictly proper scoring rule, so fitting against it already rewards honest probabilities, and every measure here is a way of asking, after the fit, how far the result fell short of that. The Brier score is the other strictly proper rule and the one that decomposes, which is why both appear rather than one.

[[Softmax Regression]] is the standing case of probabilities that look calibrated and are not guaranteed to be: its outputs sum to $1$ by construction and its regularization term systematically shrinks the scores and flattens them, without the $\arg\max$ moving, so the model can be accurate and badly calibrated at once. [[Scikit-Learn Estimator API]] states the general version of the same warning at the interface level: `predict_proba` promises one number per class in $[0, 1]$ summing to $1$ across the row, and promises nothing at all about those numbers matching observed frequencies, while `decision_function` does not even promise the range.

[[Active Learning]] is the method whose every uncertainty measure depends on this holding, since least confident, margin and entropy are all functions of the posterior and none of them ranks instances usefully when the posterior is a distortion of the truth. [[Selective Prediction]] depends on it in the same way and for a sharper reason: a certainty threshold is a number with units only if the model's probabilities are frequencies.

[[ROC Curve]] and [[Precision-Recall Tradeoff]] are what calibration is not. Both are sweeps of a threshold over a score, both read only the ordering that score induces, and both are therefore blind to the entire subject of this note, which is the separation set out above. [[Cross-Validation]] is where the held-out scores a recalibration is fitted on come from, through `cross_val_predict` and through the wrapper's own internal folds.

[[Data Leakage]] is the note to check the held-out requirement against, and the answer is that it is not one. Leakage's condition is that a feature or a training row is illegitimate for the target being predicted, and fitting a calibrator on the model's own training rows puts no label information into any feature and touches no held-out row, so it fails that condition; the defect is in-sample optimism, the same one as quoting a training score as a generalization estimate. It becomes a leak in one case only, when the rows the calibrator was fitted on are also the rows the calibrated model is later scored on, and then it is the ordinary contamination of an evaluation set rather than anything specific to calibration.
