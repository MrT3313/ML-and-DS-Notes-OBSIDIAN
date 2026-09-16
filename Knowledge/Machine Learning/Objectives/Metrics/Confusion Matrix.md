---
note_kind: concept
aliases:
  - confusion matrix
  - confusion matrices
  - error matrix
  - contingency table
  - confusion_matrix
  - ConfusionMatrixDisplay
  - true positive
  - true negative
  - false positive
  - false negative
  - TP
  - TN
  - FP
  - FN
  - Type I error
  - Type II error
  - type 1 error
  - type 2 error
  - false alarm
  - missed detection
up: "[[Performance Measure]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

A confusion matrix is a table that counts how a classifier's predicted labels line up against the true labels, one cell per (actual class, predicted class) pair, over every instance in an evaluation set. It is the full record of a classifier's behaviour on that set, and every scalar [[Performance Measure]] in this folder is a ratio computed from its cells.

## Formal statement

For a target with $K$ classes, the confusion matrix is the $K \times K$ array

$$C_{ij} = \left|\left\{\, n : y^{(n)} = i \ \text{ and } \ \hat{y}^{(n)} = j \,\right\}\right|$$

the number of instances whose true class is $i$ and whose predicted class is $j$. Rows are actual, columns are predicted. This is the scikit-learn convention, stated in its user guide as "entry $i, j$ in a confusion matrix is the number of observations actually in group $i$, but predicted to be in group $j$", with the explicit warning that other references transpose it. Reading a matrix with the axes swapped turns precision into recall silently, so the convention has to be checked, never assumed.

Three structural facts follow directly from the counting:

$$\sum_{i}\sum_{j} C_{ij} = m, \qquad \sum_{j} C_{ij} = m_i, \qquad \operatorname{tr}(C) = \sum_{i} C_{ii}$$

Every instance lands in exactly one cell, so the whole matrix sums to the evaluation set size $m$. Row $i$ sums to $m_i$, the support of class $i$, which is fixed by the data and not by the model. The diagonal holds the correct predictions and everything off it is an error, so the trace is the number of correct predictions and $m - \operatorname{tr}(C)$ is the number of mistakes. The two scalars that summarise the whole table fall straight out of that:

$$\text{Accuracy} = \frac{\operatorname{tr}(C)}{\sum_{i}\sum_{j} C_{ij}} = \frac{\operatorname{tr}(C)}{m}, \qquad \text{error rate} = 1 - \frac{\operatorname{tr}(C)}{m}$$

A perfect classifier puts every instance on the diagonal: $C_{ij} = 0$ for $i \neq j$, so $C$ is a **diagonal** matrix whose diagonal entries are the class counts $m_i$, not a matrix of ones. The [[MNIST]] 5-detector makes this concrete. Pretending the predictions were perfect gives

$$C_{\text{perfect}} = \begin{bmatrix} 54579 & 0 \\ 0 & 5421 \end{bmatrix}$$

the 54579 non-fives and the 5421 fives in the 60000-instance training set, each sitting on its own diagonal cell. An identity matrix would claim one instance per class, which is a different statement and almost never true.

### The binary case and its four named cells

With two classes, scikit-learn orders the labels in sorted order, so the negative class is row and column $0$ and the positive class is row and column $1$:

$$C = \begin{bmatrix} \text{TN} & \text{FP} \\ \text{FN} & \text{TP} \end{bmatrix}$$

| cell | actual | predicted | name | also called |
|---|---|---|---|---|
| $C_{00}$ | negative | negative | true negative (TN) | correct rejection |
| $C_{01}$ | negative | positive | false positive (FP) | Type I error, false alarm |
| $C_{10}$ | positive | negative | false negative (FN) | Type II error, missed detection |
| $C_{11}$ | positive | positive | true positive (TP) | hit |

The Type I and Type II naming is imported from hypothesis testing, where the null hypothesis is "this instance is negative". A Type I error rejects a true null, which is calling a negative instance positive, so Type I is the false positive. A Type II error fails to reject a false null, which is calling a positive instance negative, so Type II is the false negative (Banerjee et al., *Industrial Psychiatry Journal* 18(2), 2009). The plain-language readings follow the same asymmetry: a false positive is an alarm with nothing behind it, a false negative is a real thing the model walked past.

Which of the two classes is "positive" is a modelling choice and not a property of the data, so all four names move when that choice moves. [[Binary Classification]] carries that argument.

That same 5-detector, a [[Stochastic Gradient Descent Classifier]] evaluated on out-of-fold predictions, returns

$$C = \begin{bmatrix} 53892 & 687 \\ 1891 & 3530 \end{bmatrix}$$

so 3530 of the 5421 fives were caught, 1891 were missed, and 687 other digits were wrongly called fives. Those four numbers are the whole input to its precision, recall and $F_1$ scores.

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.metrics import confusion_matrix
from sklearn.model_selection import cross_val_predict

y_train_pred = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3)
cm = confusion_matrix(y_train_5, y_train_pred)
```

The signature is `confusion_matrix(y_true, y_pred, *, labels=None, sample_weight=None, normalize=None)`. The true labels come first, and swapping the two positional arguments transposes the result, which is the single easiest way to misread the output. In the binary case the four cells unpack in row-major order, `tn, fp, fn, tp = cm.ravel()`.

The predictions are built with `cross_val_predict` rather than `predict`, because each sample belongs to exactly one test fold and its prediction is computed by an estimator fitted on the other folds, so no instance is scored by a model that saw it during fitting; see [[Cross-Validation]]. Calling `predict` on the training set instead would produce a matrix describing memorisation rather than [[Generalization]].

The display class draws the same matrix and takes the same arguments:

scikit-learn 1.6:

```python
from sklearn.metrics import ConfusionMatrixDisplay

ConfusionMatrixDisplay.from_predictions(y_train, y_train_pred)
```

`normalize` is the one argument on the display that changes the numbers rather than their appearance:

scikit-learn 1.6:

```python
ConfusionMatrixDisplay.from_predictions(y_train, y_train_pred,
                                        normalize="true", values_format=".0%")
```

`normalize="true"` divides each row by its own total, turning the counts into per-class rates that sum to $1$ along each row:

$$\tilde{C}_{ij} = \frac{C_{ij}}{\sum_{k} C_{ik}} = \frac{C_{ij}}{m_i}$$

That is what makes a matrix readable when the classes differ in size. In raw counts a common class dominates the colour scale and a rare class with a terrible error rate stays invisible, because the comparison being made is between absolute counts drawn from different denominators. Row normalisation puts every class on the same denominator, so cell $\tilde{C}_{ij}$ reads as "of the instances that really were class $i$, this fraction was called $j$". `normalize` also accepts `"pred"` (divide by column totals, giving the fraction of each predicted class that really belonged to $i$) and `"all"` (divide by $m$), and defaults to `None`, which leaves counts alone. `values_format` is purely the cell text format string, `".0%"` for whole percentages, and changes nothing that is computed.

Choosing which of those axes to read, emptying the diagonal so that only the mistakes are displayed, and turning what the picture shows into a decision about what to fix, are a procedure rather than an argument, and that procedure is [[Error Analysis]]. It is built entirely on this object and it is where the ten-class digit findings live.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `labels` | the ordered class set | `None`, meaning every label seen in `y_true` or `y_pred`, sorted | supplying it fixes the row and column order and can add classes absent from the data (all-zero rows) or drop classes present in it, which changes the matrix shape and every total computed from it | set it whenever the sorted order is not the reading order you want, or when an evaluation fold might be missing a class that the full target has |
| `normalize` | $\tilde{C}$ above | `None` | `"true"` divides by row totals, `"pred"` by column totals, `"all"` by $m$; the entries stop being counts and become rates, and the integer sum $m$ becomes $1$ per row, per column, or overall | `"true"` to compare per-class error rates under [[Class Imbalance]], `None` when the raw counts are the point |
| `sample_weight` | $w^{(n)}$ | `None`, equivalent to $w^{(n)} = 1$ for all $n$ | each cell becomes $\sum_{n \in \text{cell}} w^{(n)}$ instead of a count, so raising one instance's weight raises its cell and every ratio computed from it | a weight vector correcting for a sampling design, or the boolean error mask `(y_pred != y_true)`, which weights every correct prediction by $0$ and so empties the diagonal; [[Error Analysis]] is where that use belongs |

`include_values`, `xticks_rotation`, `cmap`, `colorbar`, `ax` and `values_format` on the display class only affect appearance and are not hyperparameters by this test.

## Where it is used

It is the [[Performance Measure]] that reports every kind of mistake separately instead of collapsing them into one number, and the object [[Classification]] is judged by whenever "how often is it right" is too coarse a question. [[Accuracy]] is its trace over its total, [[Precision]] and [[Recall]] are each a ratio of two of its binary cells, and [[F1 Score]] is the harmonic mean of those two, so all four are readings of this one table rather than independent measurements.

[[Class Imbalance]] is what it reveals that a single number hides: a 5-detector scoring about $0.957$ accuracy still misses $1891$ of $5421$ fives, and only the cells say so. [[Cross-Validation]] supplies the out-of-fold predictions it is honestly built from, since a matrix computed from training-set predictions measures fit rather than performance. [[Multiclass Classification]] is the general $K \times K$ case, where the matrix stops being four named cells and acquires structure worth reading cell by cell. Reading it that way, to find which classes a model confuses and to decide what to do about it, is [[Error Analysis]], the one procedure whose whole input is this table.
