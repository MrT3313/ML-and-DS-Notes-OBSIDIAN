---
note_kind: concept
aliases:
  - recall
  - sensitivity
  - true positive rate
  - TPR
  - hit rate
  - recall_score
  - recall score
  - detection rate
up: "[[Confusion Matrix]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

Recall answers the mirror question to precision: out of all the actual positive instances in the data, how many did the model correctly catch? It is a statement about coverage of the positive class, and it says nothing about how many false alarms were raised getting there.

The same quantity travels under several names. **Sensitivity** is the medical and diagnostic-testing name, and **true positive rate** (TPR) is the signal-detection and [[ROC Curve]] name. **Hit rate** is that same ratio in the vocabulary of signal detection theory, where a signal-present trial answered yes is a *hit* and one answered no is a *miss*, so the hit rate is hits over hits plus misses, which is $\frac{\text{TP}}{\text{TP} + \text{FN}}$ with the cells renamed (Georgeson, *Sensitivity and Bias: an introduction to Signal Detection Theory*, University of Birmingham, after Green and Swets 1974 and Macmillan and Creelman 1991). They are not related quantities or special cases; they are the identical ratio, and which word gets used is a matter of which field is speaking.

## Formal statement

$$\text{Recall} = \frac{\text{TP}}{\text{TP} + \text{FN}}$$

Both cells come from the actual-positive row of a [[Confusion Matrix]], $C_{11}$ over $C_{10} + C_{11}$, so recall is the fraction of that row sitting on the diagonal. The denominator is the number of instances that really are positive, which is fixed by the data and cannot be changed by the model. That fixed denominator is what makes recall well behaved under thresholding and is the structural difference between it and precision.

On the [[MNIST]] 5-detector,

$$C = \begin{bmatrix} 53892 & 687 \\ 1891 & 3530 \end{bmatrix}, \qquad \text{Recall} = \frac{3530}{3530 + 1891} = \frac{3530}{5421} = 0.6512$$

so the model finds about $65\%$ of the fives and walks past $1891$ of them, on a target where it reports roughly $0.957$ [[Accuracy]].

Recall is undefined when there are no actual positives, since $\text{TP} + \text{FN} = 0$ makes the ratio $0/0$. This is a property of the evaluation set and not of the model: a fold or a subgroup that happens to contain no positive instance has no recall to report, no matter what the classifier does. scikit-learn's `zero_division` decides what comes back in that case.

The complement is the miss rate, or false negative rate,

$$\text{FNR} = 1 - \text{Recall} = \frac{\text{FN}}{\text{TP} + \text{FN}}$$

which is the axis a detection error tradeoff curve plots where a [[ROC Curve]] plots recall itself.

### The relationship to precision, stated carefully

The claim that increasing recall reduces precision holds under one condition and fails in general.

Fix a single trained model with a scoring function $s(\mathbf{x})$ and a threshold $t$, predicting positive when $s(\mathbf{x}) \ge t$. Lowering $t$ grows the predicted-positive set, so TP can only rise or stay. The denominator $\text{TP} + \text{FN}$ is fixed by the data, so recall is **monotonically non-increasing in $t$** and buying recall means lowering the cut point. Every instance that lowering the threshold admits is either a true positive or a false positive, and the false positives enter precision's denominator without entering its numerator, so precision **tends** to fall. It is not strictly monotone: a step that admits only true positives raises precision, which is why a precision-versus-threshold curve wobbles while the recall curve descends cleanly. The honest version is therefore: for one fixed model, recall is bought by lowering the threshold and the bill is paid in precision, with no cut point that improves both.

Across different models the claim is false, and the MNIST numbers say so. The [[Stochastic Gradient Descent Classifier]] reaches recall $0.6512$ at precision $0.8371$; a random forest on the same target and the same folds reaches recall $0.8725$ at precision $0.9897$, higher on both at once. Recall and precision are not functions of each other, and a better model moves the whole curve outward rather than sliding along it. The threshold sweep, the curve and the operating points belong to [[Precision-Recall Tradeoff]].

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.metrics import recall_score
from sklearn.model_selection import cross_val_predict

y_train_pred = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3)
recall_score(y_train_5, y_train_pred)             # 0.6511713705958311
```

The signature is `recall_score(y_true, y_pred, *, labels=None, pos_label=1, average='binary', sample_weight=None, zero_division='warn')`, identical in shape to `precision_score`, which is expected since both are computed by the same underlying routine and differ only in which total they divide by. With a boolean target the defaults report the class `pos_label=1`, meaning `True`. The user guide gives the same ratio, `tp / (tp + fn)`, describes it as the classifier's ability to find all the positive samples, and notes in passing that recall is sometimes called sensitivity. When `tp + fn == 0` the function returns $0$ and raises `UndefinedMetricWarning`, unless `zero_division` overrides that.

`roc_curve` returns this same quantity as its `tpr` array, one value per threshold, which is the concrete sense in which recall and true positive rate are one thing with two names.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `average` | which aggregation over classes | `'binary'` | `'binary'` reports only `pos_label`; `'micro'` pools every class's TP and FN into one global ratio, which for a single-label target equals accuracy; `'macro'` averages the per-class recalls unweighted, so a rare class weighs as much as a common one; `'weighted'` averages them weighted by support; `'samples'` averages per instance and is multilabel only; `None` returns the per-class vector unaveraged | `'binary'` for two classes, `'macro'` when catching the rare class matters as much as the common one, `'weighted'` for a population-tracking number, `None` to read each class |
| `pos_label` | the class treated as positive | `1` | changing it swaps which class is being recalled, exchanging TP with TN and FN with FP, so the result is a different measurement rather than a rescaling of the same one | state it explicitly whenever the positive class is not `1` or `True` |
| `zero_division` | the value returned when $\text{TP} + \text{FN} = 0$ | `'warn'` | `'warn'` returns $0$ and warns; `0.0` returns $0$ quietly; `1.0` returns $1$, crediting perfect recall on a class with no instances; `np.nan` drops the undefined class from an average rather than counting it as $0$ | keep the default so the warning surfaces an empty positive class; use `np.nan` with macro averaging so a class absent from a fold does not pull the mean down |
| `labels` | which classes are included, and in what order | `None` | naming a subset restricts the average to those classes, so a class left out stops contributing at all and the averaged number can move either way | set it whenever `average` is not `'binary'` and a class may be missing from a fold, so the set averaged over stays fixed |
| `sample_weight` | per-instance weights $w_i$ | `None` | each confusion matrix cell becomes a weighted sum rather than a count, so $\text{TP}$ and $\text{FN}$ are reweighted before the ratio is formed | leave unset unless the instances genuinely are not equally important |

## Where it is used

It is one of the two ratios read off a [[Confusion Matrix]], specifically its actual-positive row, and the measure to lead with in [[Classification]] whenever a missed positive is the expensive error, since it is the only one of the four cells' ratios that counts false negatives against the model.

[[Precision]] is the quantity it trades against for one fixed model, and neither is informative without the other, because recall alone is driven to $1$ by predicting positive for everything. [[F1 Score]] is their harmonic mean, which stays low unless both are high. [[Precision-Recall Tradeoff]] is the threshold sweep that moves both and the home of the curve. [[ROC Curve]] plots recall as its vertical axis under the name true positive rate, against the false positive rate on the horizontal, which is the same model viewed against the negative class instead of against its own positive predictions.
