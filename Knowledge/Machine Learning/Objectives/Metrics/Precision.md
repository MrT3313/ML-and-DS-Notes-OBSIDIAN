---
note_kind: concept
aliases:
  - precision
  - positive predictive value
  - PPV
  - precision_score
  - precision score
up: "[[Confusion Matrix]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

Precision answers one question about a classifier: out of all the positive predictions it made, how many were actually positive? It is a statement about the model's claims, not about the data, so it says how much a positive prediction can be trusted and says nothing about how many positives were missed.

## Formal statement

$$\text{Precision} = \frac{\text{TP}}{\text{TP} + \text{FP}}$$

Both cells come from the predicted-positive column of a [[Confusion Matrix]], $C_{11}$ over $C_{01} + C_{11}$, so precision is the fraction of that column sitting on the diagonal. The denominator is the number of instances the model called positive, which the model controls completely: predict positive once, correctly, and precision is $1$. That degenerate case is why precision is never reported alone.

On the [[MNIST]] 5-detector the matrix is

$$C = \begin{bmatrix} 53892 & 687 \\ 1891 & 3530 \end{bmatrix}, \qquad \text{Precision} = \frac{3530}{3530 + 687} = \frac{3530}{4217} = 0.8371$$

so when that model says "this is a five" it is right about $84\%$ of the time.

Precision is undefined when the model predicts no positives at all, since $\text{TP} + \text{FP} = 0$ makes the ratio $0/0$. This is not a rare pathology: it is exactly what a constant negative predictor does on an imbalanced target, and it happens routinely inside a single cross-validation fold or for a rare class under macro averaging. scikit-learn's `zero_division` is the parameter that decides what gets returned in that case.

### The relationship to recall, stated carefully

The claim that increasing precision reduces recall is true under one condition and false in general, and the condition is the whole content of the claim.

Fix a single trained model with a scoring function $s(\mathbf{x})$ and predict positive when $s(\mathbf{x}) \ge t$. Raising $t$ shrinks the predicted-positive set, since

$$t' > t \implies \{\, n : s(\mathbf{x}^{(n)}) \ge t' \,\} \subseteq \{\, n : s(\mathbf{x}^{(n)}) \ge t \,\}$$

so TP and FP can only fall or stay. Recall's denominator $\text{TP} + \text{FN}$ is the number of actual positives, which is fixed by the data, so recall is **monotonically non-increasing** in $t$. Precision's denominator shrinks along with its numerator, so precision **tends** upward but is not monotone: dropping one instance can remove a true positive and no false positive, which lowers the ratio. That is why a precision curve plotted against threshold is bumpy while a recall curve only ever descends. So the honest version is: for one fixed model, buying precision by raising the threshold costs recall, and you cannot get both at once by moving the cut point.

Across different models the claim is simply false, and the MNIST numbers demonstrate it. The [[Stochastic Gradient Descent Classifier]] reaches precision $0.8371$ at recall $0.6512$; a random forest on the same target and the same folds reaches precision $0.9897$ at recall $0.8725$, higher on both. Precision and recall are not functions of each other. A better model moves the entire curve outward, and only movement *along* one curve forces the trade. [[Precision-Recall Tradeoff]] carries the threshold sweep and the curve itself.

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.metrics import precision_score
from sklearn.model_selection import cross_val_predict

y_train_pred = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3)
precision_score(y_train_5, y_train_pred)          # 0.8370879772350012
```

The signature is `precision_score(y_true, y_pred, *, labels=None, pos_label=1, average='binary', sample_weight=None, zero_division='warn')`. With a boolean target the defaults report the single class `pos_label=1`, which for booleans is `True`, so the call above scores the five-detection and not the not-a-five detection. The two numbers are different and the default silently picks one.

The user guide states the same ratio, `tp / (tp + fp)`, and describes it as the classifier's ability not to label a negative sample as positive. When `tp + fp == 0` it returns $0$ and raises `UndefinedMetricWarning`, unless `zero_division` says otherwise.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `average` | which aggregation over classes | `'binary'` | `'binary'` reports only `pos_label`; `'micro'` pools all classes' TP and FP into one global ratio; `'macro'` averages the per-class precisions unweighted, so a rare class counts as much as a common one; `'weighted'` averages them weighted by each class's support; `'samples'` averages per instance and is meaningful only for multilabel; `None` returns the per-class vector with no averaging | `'binary'` for a two-class target, `'macro'` when every class matters equally, `'weighted'` when you want a number that tracks the population, `None` when you intend to read each class |
| `pos_label` | the class treated as positive | `1` | changing it swaps which class the binary score describes, turning TP into TN and FP into FN, which generally gives a completely different number rather than a related one | set it explicitly whenever the positive class is not labelled `1` or `True`, and whenever a reader might guess wrong |
| `zero_division` | the value returned when $\text{TP} + \text{FP} = 0$ | `'warn'` | `'warn'` returns $0$ and warns; `0.0` returns $0$ silently; `1.0` returns $1$, which flatters a model that predicted nothing; `np.nan` excludes the undefined class from an average instead of counting it | leave at the default so the warning surfaces the degenerate case; use `np.nan` under macro averaging so an absent class does not drag the mean toward $0$ |
| `labels` | which classes are included, and in what order | `None` | naming a subset restricts the average to those classes, so a class left out stops contributing entirely and the averaged number can move in either direction | set it whenever `average` is not `'binary'` and some class may be absent from a fold, so the set of classes averaged over stays fixed |
| `sample_weight` | per-instance weights $w_i$ | `None` | each confusion matrix cell becomes a weighted sum rather than a count, so $\text{TP}$ and $\text{FP}$ are reweighted before the ratio is taken | leave unset unless the instances genuinely are not equally important |

## Where it is used

It is one of the two ratios read off a [[Confusion Matrix]], specifically its predicted-positive column, and the first measure to report for [[Classification]] whenever a false alarm is expensive, since it is the only one of the four cells' ratios that counts false positives against the model. It is also the vertical axis of the precision/recall curve.

[[Recall]] is the quantity it trades against for one fixed model, and reporting precision without it is the standing failure mode, because precision alone can be driven to $1$ by predicting almost nothing. [[F1 Score]] is their harmonic mean, the single number that refuses to be high unless both are. [[Precision-Recall Tradeoff]] is the threshold sweep that moves both at once and the place the curve and its operating points belong.
