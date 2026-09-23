---
note_kind: concept
aliases:
  - selective prediction
  - selective classification
  - abstention
  - abstain
  - reject option
  - classification with rejection
  - coverage-risk tradeoff
  - selective risk
  - certainty threshold
  - confidence threshold
  - confidence measurement
up: "[[Model Calibration]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

Selective prediction attaches a usefulness threshold to each individual prediction rather than to the model as a whole: the reading is taken on one sample at a time, and it answers whether this particular answer is worth showing. If you only want to show the predictions the model is certain about, you do it through a certainty threshold, and everything below the threshold is not answered at all.

That last clause is the whole of what is new here. A decision threshold picks *which class*; a certainty threshold picks *whether to answer*. [[Precision-Recall Tradeoff]] owns the first, where moving the cut point trades one kind of error for the other and every instance still receives a label. This note owns the second, where the model declines to produce a label for some instances and the errors it would have made on them are neither traded nor absorbed, they simply do not happen.

One word needs separating before anything else. The per-instance reading here is a model's certainty that *this* prediction is right, and it is not the quantity [[Association Rule Learning]] calls confidence, which is $\text{confidence}(A \Rightarrow B) = P(B \mid A)$, a property of a rule over a set of transactions with no model and no prediction in it; `confidence` is also the frontmatter key every concept and method note in this vault carries to record how well checked the note is. A reader who arrived looking for either of those two is in the wrong note and should follow the link.

## Formal statement

### A selective predictor is a pair

The object is not one function but two. Write $h : \mathcal{X} \to \mathcal{Y}$ for the ordinary predictor and $g : \mathcal{X} \to \{0, 1\}$ for a **selection function**, and the pair predicts or abstains:

$$(h, g)(\mathbf{x}) = \begin{cases} h(\mathbf{x}) & \text{if } g(\mathbf{x}) = 1 \\[2pt] \text{abstain} & \text{if } g(\mathbf{x}) = 0 \end{cases}$$

In practice $g$ is never fitted directly. It is a **confidence function** $\kappa : \mathcal{X} \to \mathbb{R}$ cut at a threshold,

$$g(\mathbf{x}) = \mathbb{1}\!\left[\, \kappa(\mathbf{x}) \ge \tau \,\right]$$

and $\tau$ is written here rather than $t$ deliberately, because $t$ is already the decision threshold and the two are applied to different functions to different ends. $t$ cuts a score that ranks the *classes* against each other and decides which one wins. $\tau$ cuts a score that ranks *predictions* by how likely each is to be right and decides whether $h$ speaks. Both can be set on the same model at once, and neither substitutes for the other: tightening $t$ changes which label comes back, tightening $\tau$ reduces how many come back.

The usual $\kappa$ for a model with a posterior is the largest class probability,

$$\kappa(\mathbf{x}) = \max_{k} \hat{p}_k(\mathbf{x})$$

which is Hendrycks and Gimpel's (ICLR 2017) baseline for detecting the model's own misclassifications and its out-of-distribution inputs, and is the same quantity [[Active Learning]]'s least confident measure reads, to the opposite purpose. Geifman and El-Yaniv (NeurIPS 2017) are the reference for the pair and for building $g$ to hit a stated risk level.

### Coverage against selective risk

Two quantities, and the arrangement trades exactly these. **Coverage** is the fraction of inputs answered and **selective risk** is the error rate among the answered:

$$\phi(h, g) = \mathbb{E}\big[\, g(\mathbf{X}) \,\big], \qquad R(h, g) = \frac{\mathbb{E}\big[\, \ell\big(h(\mathbf{X}), Y\big)\, g(\mathbf{X}) \,\big]}{\phi(h, g)}$$

and over a held-out set of $m$ instances they are counted as

$$\hat{\phi} = \frac{1}{m}\sum_{i=1}^{m} g\big(\mathbf{x}^{(i)}\big), \qquad \hat{R} = \frac{\sum_{i=1}^{m} \ell\big(h(\mathbf{x}^{(i)}), y^{(i)}\big)\, g\big(\mathbf{x}^{(i)}\big)}{\sum_{i=1}^{m} g\big(\mathbf{x}^{(i)}\big)}$$

The denominator is the whole point. An abstention is counted as neither a hit nor a miss, and dividing by the answered count rather than by $m$ is what keeps the risk comparable as coverage falls. Without it, abstaining would look like improvement automatically, since the numerator alone can only shrink. At $\phi = 1$ the selection function accepts everything and $R$ is the model's ordinary risk, which is how the two quantities connect back to every other measure in this vault. Under $0/1$ loss $1 - R$ is accuracy computed on the answered subset only.

### The accuracy-coverage curve

Sweeping $\tau$ from below the smallest score to above the largest traces the path $\{(\phi(\tau), R(\tau))\}$, the risk-coverage curve, and plotted as $1 - R$ against $\phi$ it is the accuracy-coverage curve. Coverage is monotone in the threshold for the same containment reason recall is monotone in the decision threshold: for $\tau' > \tau$ the acceptance condition is strictly harder, so

$$\{\, \mathbf{x} : \kappa(\mathbf{x}) \ge \tau' \,\} \subseteq \{\, \mathbf{x} : \kappa(\mathbf{x}) \ge \tau \,\}$$

and raising $\tau$ can only remove instances from the answered set. The curve is read the way a threshold sweep is always read: pick the coverage the application can live with and read off the risk, or fix the risk the application requires and read off how much of the traffic the model can keep.

### Why it is a tradeoff and not a free lunch

The claim that abstaining lowers risk is not free, and it has a one-line proof that also shows exactly what it depends on. Sort the instances by decreasing $\kappa$ and write

$$e_i = P\big(h(\mathbf{x}^{(i)}) \ne y^{(i)} \mid \mathbf{x}^{(i)}\big)$$

for the true probability that the $i$-th prediction is wrong. Assume $\kappa$ ranks correctly, meaning that descending $\kappa$ implies non-decreasing $e$, so $e_1 \le e_2 \le \dots \le e_m$. Answering the top $c$ gives a selective risk that is a running mean,

$$R_c = \frac{1}{c}\sum_{i=1}^{c} e_i$$

and the step from $c$ to $c+1$ is

$$R_{c+1} - R_c = \frac{e_{c+1} - R_c}{c + 1} \; \ge \; 0$$

since $e_{c+1} \ge e_i$ for every $i \le c$ and therefore $e_{c+1} \ge R_c$. So the risk-coverage curve is non-decreasing in coverage: every instance added back is at least as likely to be wrong as the average of those already answered. Risk is bought with coverage and coverage is bought with risk, and the exchange is forced rather than chosen.

What the inequality does not say is how *fast* risk falls as coverage falls. That rate is a property of $\kappa$ and of nothing else, so it is the thing to measure when comparing two confidence functions on the same predictor, usually as the area under the risk-coverage curve.

### The whole arrangement rests on the score

The derivation above used exactly one assumption, that descending $\kappa$ implies non-decreasing $e$, and it is the assumption that fails in practice. Two distinct things can go wrong and they cost different things.

**A score that does not rank loses the monotonicity itself.** If $\kappa$ orders instances in a way unrelated to how likely they are to be wrong, the inequality does not hold, the curve flattens or wanders, and the rule throws away correct answers while keeping wrong ones. There is no sweep to read and no threshold that helps, because the thing being thresholded carries no information about the quantity of interest. This is the same point [[Active Learning]]'s failure modes make from the other end: the loop that *buys* the least confident instances and the rule that *refuses* them are reading the same number, so a number that means nothing breaks both.

**A score that ranks but is not calibrated loses the units.** Ranking is the weaker requirement and it buys only the shape of the curve. The threshold is still a bare knob: $\tau = 0.9$ names no error rate, and the only way to set it is to sweep and read the curve on held-out data. Under [[Model Calibration]] it names one, and the bound is immediate. Perfect calibration on the committed class means $\mathbb{E}[e \mid \kappa = p] = 1 - p$, so

$$R = \mathbb{E}\big[\, 1 - \kappa \;\big|\; \kappa \ge \tau \,\big] \; \le \; 1 - \tau$$

and setting $\tau = 0.95$ is then a statement that the answered predictions are wrong at most five percent of the time, checkable and falsifiable rather than a hope. A certainty threshold is worth exactly as much as the calibration behind it, and with no calibration it is worth a curve and no number.

### Chow's rule, where the costs are known

The threshold is not arbitrary when the cost of an error, of an abstention and of a correct answer are all known and the posterior is available. Chow (1970) gives the optimal rule for that case: abstain when the largest posterior falls below $1 - d$, with

$$d = \frac{C_r - C_c}{C_e - C_c}$$

where $C_e$, $C_r$ and $C_c$ are the costs of an error, a rejection and a correct answer. So the certainty threshold is $\tau = 1 - d$ read on the maximum posterior, and the rule that minimizes total cost is the one this note has been describing all along. Two readings follow. Rejection is worth nothing at all once $1 - d \le 1/K$, since the largest of $K$ posteriors is always at least $1/K$ and the rule would then never fire, which puts a ceiling on how expensive an abstention can be before you should simply guess. And the rule is stated on the *true* posterior, so an uncalibrated score fed into it rejects at a rate nobody asked for, which is the previous section's point arriving from the cost side.

### Conformal prediction, the other route to a guarantee

Thresholding a confidence score gives a tradeoff curve, and under calibration it gives the bound above. What it does not give is a finite-sample guarantee that holds without assuming the model's probabilities are right. Conformal prediction is the named alternative that does. Instead of one label or an abstention it returns a *set* of labels, built by comparing each new instance's score against the scores on a held-out calibration split, and the set contains the true label with at least a user-chosen probability, for any model and any distribution, assuming only that the calibration data and the new instance are exchangeable. Abstention becomes a property of the set rather than a separate decision: a singleton is a committed answer, a set of several labels is the model refusing to narrow further, and an empty set is a refusal outright. Angelopoulos, Bates, Malik and Jordan's construction for image classifiers appeared as a preprint in September 2020 and at ICLR 2021, and Angelopoulos and Bates' 2021 introduction is the standing tutorial. It is a different formalism from the pair $(h, g)$ above rather than a tuning of it, and this note does not carry it.

### In scikit-learn

**scikit-learn 1.6 has no selective prediction API.** There is no selection function, no reject option, and no metric in `sklearn.metrics` conditioned on an answered subset, so neither coverage nor selective risk can be asked for by name. The nearest things in the library are two, and the first is a near miss worth stating because it is the distinction this note rests on. `FixedThresholdClassifier` and `TunedThresholdClassifierCV` set the decision threshold $t$, so they change which class `predict` returns and never whether it returns one; a classifier wrapped in either still answers every row. The second is `CalibratedClassifierCV`, which supplies the score the rule needs rather than the rule.

The rule itself is a few lines over `predict_proba`, and what makes it honest is that the scores are out-of-fold and the model has been calibrated first. scikit-learn 1.6 and NumPy 2.x:

```python
import numpy as np
from sklearn.calibration import CalibratedClassifierCV
from sklearn.model_selection import cross_val_predict

calibrated = CalibratedClassifierCV(clf, method="sigmoid", cv=5)
probs = cross_val_predict(calibrated, X_train, y_train, cv=5,
                          method="predict_proba")

y_true = np.asarray(y_train)
classes = np.unique(y_true)                  # the column order of probs
y_pred = classes[probs.argmax(axis=1)]
kappa = probs.max(axis=1)                    # the confidence function

tau = 0.90                                   # the certainty threshold
answered = kappa >= tau                      # the selection function g
coverage = answered.mean()
selective_risk = (y_pred[answered] != y_true[answered]).mean()
```

The whole curve is the running mean from the monotonicity argument, computed directly rather than by looping over thresholds:

```python
order = np.argsort(-kappa)                   # most confident first
wrong = (y_pred != y_true)[order]
answered_count = np.arange(1, len(order) + 1)

coverages = answered_count / len(order)
risks = np.cumsum(wrong) / answered_count    # selective risk at each coverage
```

`risks[c - 1]` is $R_c$ and `coverages` is $c / m$, so plotting the second against the first is the risk-coverage curve, and plotting `1 - risks` against `coverages` gives the accuracy-coverage curve. Ties in `kappa` are broken by `argsort`'s ordering rather than by anything meaningful, which matters only when many instances share a score exactly, as they do for a small tree ensemble whose probabilities are vote shares.

Where the guarantee route lives as a library, checked on 23 September 2026: MAPIE is the conformal prediction package built against the scikit-learn interface, at version 1.5.0 released August 2026, requiring scikit-learn 1.4 or later, and covering classification, regression and risk control. It is the nearest thing to an off-the-shelf abstention rule with a stated coverage level, and it returns prediction sets rather than the pair above.

## Where it is used

[[Model Calibration]] is what the whole arrangement stands on, in the two separate ways set out above: a score that ranks correctly buys the monotone curve, and a score that is calibrated buys the bound $R \le 1 - \tau$, which is what turns the threshold from a knob into a stated error rate.

[[Precision-Recall Tradeoff]] is the neighbouring sweep and the contrast that defines this note. Both cut a continuous response at a threshold and both are read as a curve, and they differ in what the cut decides: there it is which class is predicted, with every instance still receiving a label and the two error types traded against each other, and here it is whether a prediction is issued at all, with the instances below the cut leaving the confusion matrix entirely. A model can carry both thresholds at once and they are tuned against different requirements.

[[Active Learning]] ranks instances by the same score to the opposite purpose: uncertainty sampling buys labels for the instances the model is least sure about, because those are the ones that move the [[Decision Boundary]], while this note refuses to answer on exactly those instances. The two are the same measurement spent in opposite directions, and they share a failure: both are meaningless on a model whose posterior does not track how often it is right.

[[Association Rule Learning]] is where the other sense of the word lives, a property of a rule over transactions rather than a reading on a prediction, and it is linked here so that a reader who arrived on the wrong sense leaves immediately. [[Performance Measure]] is the category the pair $(\phi, R)$ joins, and it joins as a pair rather than as a number, in the same way the two threshold sweeps do: quoting a selective risk without its coverage is the same selection as quoting a precision without its recall.
