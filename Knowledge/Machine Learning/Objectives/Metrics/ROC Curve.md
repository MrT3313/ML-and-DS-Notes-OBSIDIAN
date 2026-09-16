---
note_kind: concept
aliases:
  - ROC
  - ROC curve
  - receiver operating characteristic
  - receiver operating characteristic curve
  - AUC
  - ROC AUC
  - AUROC
  - area under the curve
  - area under the ROC curve
  - roc_curve
  - roc_auc_score
  - false positive rate
  - FPR
  - true negative rate
  - TNR
  - specificity
  - fall-out
  - false negative rate
  - FNR
  - miss rate
up: "[[Performance Measure]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

The receiver operating characteristic curve is a plot used to evaluate and illustrate the performance of a binary classifier as its discrimination threshold is varied. It puts the true positive rate on the vertical axis and the false positive rate on the horizontal axis, and each point on the curve is one threshold setting. Like the precision/recall curve it describes a whole family of classifiers, one per cut point, rather than a single model at a single operating point.

## Formal statement

### The four rates

All four are ratios of the cells of a [[Confusion Matrix]], and each is conditioned on one row of it, meaning on one actual class. That is what makes them insensitive to how many instances of the other class exist.

- **True positive rate (TPR), also sensitivity, also [[Recall]].** The proportion of actual positive cases correctly identified as positive. It measures the model's ability to detect positive instances. $$TPR = \frac{TP}{TP + FN}$$
- **False positive rate (FPR), also fall-out.** The proportion of actual negative cases incorrectly classified as positive. It measures how often a false alarm is triggered among actual negatives. $$FPR = \frac{FP}{FP + TN}$$
- **True negative rate (TNR), also specificity.** The proportion of actual negative cases correctly identified as negative. It measures the model's ability to correctly rule out negative instances. $$TNR = \frac{TN}{TN + FP}$$
- **False negative rate (FNR), also miss rate.** The proportion of actual positive cases incorrectly classified as negative. It measures the rate at which real positives are missed. $$FNR = \frac{FN}{TP + FN}$$

Two identities follow by inspection, and they are why only two of the four are ever plotted. The positives split into $TP$ and $FN$ and the negatives split into $TN$ and $FP$, so each pair sums over the same denominator:

$$TPR + FNR = \frac{TP + FN}{TP + FN} = 1, \qquad TNR + FPR = \frac{TN + FP}{TN + FP} = 1$$

Hence $FNR = 1 - TPR$ and $TNR = 1 - FPR$. The ROC axes are the two independent numbers; the other two are their complements and carry nothing new. Specificity is the one that trips people, since a "specificity" axis and a "$1 -$ specificity" axis are the same plot with the $x$ direction reversed.

### The curve

With the threshold $t$ from the [[Precision-Recall Tradeoff]] and $\hat{y}(\mathbf{x}; t) = \mathbb{1}[s(\mathbf{x}) > t]$, every rate above becomes a function of $t$, and the ROC curve is the parametric path traced as $t$ sweeps from $+\infty$ down to $-\infty$:

$$\mathrm{ROC} = \left\{ \left( FPR(t),\; TPR(t) \right) : t \in \mathbb{R} \right\}$$

At $t = +\infty$ nothing is predicted positive, so $TP = FP = 0$ and the path starts at the origin $(0, 0)$. At $t = -\infty$ everything is predicted positive, so $FN = TN = 0$ and it ends at $(1, 1)$. Both $TP(t)$ and $FP(t)$ are non-increasing in $t$, so lowering the threshold moves the point up and to the right monotonically in both coordinates: the curve is non-decreasing, and it never doubles back. Unlike precision, neither axis can dip, which is the reason ROC curves look smooth where the precision-against-threshold plot looks jagged.

The diagonal $TPR = FPR$ is the reference. A classifier that ignores its input and calls an instance positive with probability $p$ regardless has $TPR = p$ and $FPR = p$, so it sits on the diagonal at $(p, p)$ whatever $p$ is, and sweeping $p$ traces the whole line. A useful classifier bows above it toward the top left corner, where $FPR = 0$ and $TPR = 1$ is the perfect point.

### Area under the curve

The single number summarizing the curve is the area under it,

$$\mathrm{AUC} = \int_{0}^{1} TPR \, d(FPR)$$

which for the discrete curve scikit-learn returns is computed by the trapezoidal rule over the plotted vertices. The diagonal cuts the unit square in half, so a random classifier scores $\mathrm{AUC} = 0.5$ and a perfect one scores $\mathrm{AUC} = 1.0$. Below $0.5$ means the ranking is worse than chance, which is usually a sign the score is inverted rather than a sign of a genuinely anti-predictive model.

The interpretation that makes AUC worth reporting is a ranking statement rather than a geometric one. Draw one instance at random from the positives and one at random from the negatives. Then

$$\mathrm{AUC} = \Pr\!\left[\, s(\mathbf{X}^{+}) > s(\mathbf{X}^{-}) \,\right] + \tfrac{1}{2} \Pr\!\left[\, s(\mathbf{X}^{+}) = s(\mathbf{X}^{-}) \,\right]$$

the probability that the classifier ranks the positive above the negative, with ties counted half. This is Fawcett's result, the reference scikit-learn itself cites for `roc_auc_score` (Fawcett, T. 2006, *An introduction to ROC analysis*). Two consequences: AUC depends only on the *order* the scores induce, not on their values, so any strictly increasing recalibration of $s$ leaves AUC unchanged; and AUC says nothing about whether any particular threshold is any good, since it averages over all of them at once.

Two classifiers on the [[MNIST]] 5-versus-rest target: the SGD classifier scores $\mathrm{AUC} = 0.9605$ and the random forest scores $\mathrm{AUC} = 0.9983$.

### In scikit-learn

The two classifiers are scored through different methods, and the reason is which response each model actually produces. The SGD classifier exposes `decision_function`, an unbounded signed score, while the random forest exposes `predict_proba`, from which the positive class column is taken. Either works, because AUC only reads the ordering.

scikit-learn 1.6:

```python
import matplotlib.pyplot as plt
from sklearn.model_selection import cross_val_predict
from sklearn.metrics import roc_curve, roc_auc_score

fpr, tpr, thresholds = roc_curve(y_train_5, y_scores)

plt.plot(fpr, tpr, label="ROC curve")
plt.plot([0, 1], [0, 1], "k:", label="Random classifier's ROC curve")

# mark where the 90% precision threshold lands on this curve
idx_for_threshold_at_90 = (thresholds <= threshold_for_90_precision).argmax()
tpr_90, fpr_90 = tpr[idx_for_threshold_at_90], fpr[idx_for_threshold_at_90]
plt.plot([fpr_90], [tpr_90], "ko", label="Threshold for 90% precision")

roc_auc_score(y_train_5, y_scores)                    # 0.9604938554008616

y_probas_forest = cross_val_predict(forest_clf, X_train, y_train_5, cv=3,
                                    method="predict_proba")
y_scores_forest = y_probas_forest[:, 1]
roc_auc_score(y_train_5, y_scores_forest)             # 0.9983436731328145
```

The 1.6 signature is `roc_curve(y_true, y_score, *, pos_label=None, sample_weight=None, drop_intermediate=True)`, returning `(fpr, tpr, thresholds)`. The `thresholds` array is **decreasing**, not increasing, which is the opposite of what `precision_recall_curve` returns and is what makes `(thresholds <= threshold_for_90_precision).argmax()` correct: the boolean array is `False` while the thresholds are still above the target and flips to `True` at the first threshold at or below it, and `argmax` picks out that first `True`. Since 1.3, `thresholds[0]` is `np.inf`, the threshold at which no instance is predicted positive, which is what pins the curve's first vertex to $(0, 0)$; before 1.3 it was an arbitrary `max(y_score) + 1`. Code that compared against `thresholds[0]` numerically changed behaviour at that release.

`roc_auc_score(y_true, y_score, *, average='macro', sample_weight=None, max_fpr=None, multi_class='raise', labels=None)` is the 1.6 signature. Three of its arguments change the number that comes back. `average` decides how per-label areas are combined on a multilabel target, `"macro"` unweighted and `"weighted"` by support, with `"micro"` pooling the indicator matrix elementwise first. `multi_class` is inert on a binary target and must be set explicitly to `"ovr"` or `"ovo"` on a multiclass one, since its default `"raise"` refuses to guess; `"ovr"` scores each class against all the others and `"ovo"` averages over all class pairs. `max_fpr` restricts the integral to $[0, \mathrm{max\_fpr}]$ and returns McClish's standardized partial area, which is the right call when only the low-false-alarm end of the curve is operationally reachable, and is unavailable for multiclass.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `pos_label` (`roc_curve`) | the class treated as positive | `None` | names which class the rates are computed about; switching it reflects the whole curve through the point $(0.5, 0.5)$, since $TPR$ and $FPR$ are replaced by $TNR = 1 - FPR$ and $FNR = 1 - TPR$, and the area under the reflected curve is $1 - \mathrm{AUC}$ | required when the labels are not $\{0, 1\}$ or $\{-1, 1\}$, since the function raises rather than guess. `roc_auc_score` has no such argument and reads the label ordering instead |
| `average` (`roc_auc_score`) | how per-label areas are combined | `'macro'` | `'macro'` averages the per-label AUCs unweighted so a rare label counts as much as a common one; `'weighted'` averages them by support; `'micro'` pools the whole label-indicator matrix into one binary problem first; `'samples'` averages per instance; `None` returns the per-label vector | inert on a binary target; `'macro'` when every label matters equally, `'micro'` when the overall instance-level ranking is the point |
| `multi_class` (`roc_auc_score`) | the multiclass decomposition | `'raise'` | `'raise'` refuses to compute anything and forces the choice; `'ovr'` scores each class against all the others, so it inherits the imbalance of a one-versus-rest split; `'ovo'` averages over all $\binom{K}{2}$ class pairs and is insensitive to class priors | `'ovo'` when the classes are imbalanced, `'ovr'` when you want per-class numbers back via `average=None` |
| `max_fpr` (`roc_auc_score`) | the right edge of the integral | `None` | restricts the area to $\int_{0}^{\text{max\_fpr}} TPR \, d(FPR)$ and rescales it by McClish's standardization, so raising it toward $1$ recovers the full AUC and lowering it concentrates on the low-false-alarm end | set it to the largest false alarm rate the application can actually tolerate, rather than reporting an area over thresholds nobody would deploy |
| `drop_intermediate` (`roc_curve`) | whether collinear vertices are discarded | `True` | keeping it on removes points that lie on a straight segment of the curve, shrinking the returned arrays without changing the shape or the area; turning it off returns one vertex per distinct score | leave on for plotting, off when you need the threshold array to line up with every distinct score |

`sample_weight` also changes the numbers, by replacing each cell count with a weighted sum before the rates are taken.

## Where it is used

It is a [[Performance Measure]] for [[Binary Classification]] whose four rates are all ratios of the cells of a [[Confusion Matrix]], and whose vertical axis is [[Recall]] under its other name, so reading a ROC curve is reading recall against the cost of getting it.

It is the second of the two threshold-sweep views, alongside the [[Precision-Recall Tradeoff]], and the rule for choosing between them is easy to garble. Stating it as a preference for the precision/recall curve when the positive class is rare or when you care about false positives and false negatives collapses a comparison into a conjunction, and so states nothing: caring about both kinds of error is the situation everybody is always in. The rule that works is a comparison: prefer the precision/recall curve whenever the positive class is rare, or whenever a false positive costs you more than a false negative does, and otherwise report the ROC curve. The operative word is *than*, and the mechanism behind it is worked through in the section below.

[[Class Imbalance]] is the condition that makes the ROC curve look optimistic, since a large negative class sits in the denominator of the only axis that counts false alarms. [[Ensemble Learning]] supplies the comparison case: a random forest is scored on the same target through `predict_proba` rather than `decision_function`, because a tree ensemble produces a class-probability estimate from the vote share of its members and has no signed decision score to offer, and its curve sits well above the SGD classifier's at $\mathrm{AUC} = 0.9983$ against $0.9605$.

## VS

Against the precision/recall curve, both plot the same threshold sweep of the same model, and they differ in exactly one respect: what sits in the denominator of the horizontal reading. Its consequences are large enough to decide which plot to publish.

The ROC curve's $FPR = \frac{FP}{FP + TN}$ has the full negative count underneath it. The precision used by the PR curve, $P = \frac{TP}{TP + FP}$, has the false positive count in its own denominator and no reference to $TN$ at all. So when negatives vastly outnumber positives, a great many false positives barely move $FPR$ while they gut precision.

The [[MNIST]] 5-versus-rest target makes this concrete. Of the $60{,}000$ training digits, $5{,}421$ are fives and $54{,}579$ are not. At the default threshold the SGD classifier has $TP = 3530$ and $FP = 687$, so

$$FPR = \frac{687}{54{,}579} = 0.0126, \qquad P = \frac{3530}{3530 + 687} = 0.8371$$

Now imagine the false alarms grew tenfold to $6{,}870$ with the same hits. The ROC reading moves to $FPR = \frac{6870}{54{,}579} = 0.1259$, still comfortably near the left edge of the plot and still visually excellent. Precision collapses to $\frac{3530}{3530 + 6870} = 0.3394$. Ten times the false alarms cost the ROC curve about twelve points of a hundred-point axis and cost precision half its value. Someone reading only the ROC curve would not notice that five of every eight flagged instances are now wrong.

This is also why "you care more about false positives than false negatives" points to the same plot as "the positive class is rare". Both conditions make $FP$ the quantity that matters, and precision is the measure that reacts to it while $FPR$ is the measure that hides it in a large denominator. When the classes are near balanced and both error types cost about the same, the negative count is no longer large enough to absorb anything, the two curves say compatible things, and the ROC curve is the conventional report.
