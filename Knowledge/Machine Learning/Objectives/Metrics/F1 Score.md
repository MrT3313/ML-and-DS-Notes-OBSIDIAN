---
note_kind: concept
aliases:
  - F1
  - F1 score
  - F1-score
  - F1 measure
  - F-score
  - F-measure
  - F beta
  - F-beta
  - F-beta score
  - Fbeta
  - harmonic mean of precision and recall
  - f1_score
  - fbeta_score
up: "[[Confusion Matrix]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

The F1 score collapses [[Precision]] and [[Recall]] into a single number by taking their harmonic mean, which is high only when both inputs are high. It is exactly the harmonic mean rather than merely something like one, and the derivation below shows why the familiar $2PR/(P+R)$ formula and the harmonic mean are the same expression written two ways.

## Formal statement

Write $P$ for precision and $R$ for recall. The form usually quoted is

$$F_1 = 2 \cdot \frac{P \cdot R}{P + R}$$

and the harmonic mean of two numbers is the reciprocal of the average of their reciprocals,

$$F_1 = \frac{2}{\frac{1}{P} + \frac{1}{R}}$$

The two are identical: multiply numerator and denominator of the second by $PR$ and $\frac{1}{P} + \frac{1}{R} = \frac{R + P}{PR}$ turns it into $\frac{2PR}{P + R}$. Substituting $P = \frac{TP}{TP + FP}$ and $R = \frac{TP}{TP + FN}$ into the reciprocal sum gives $\frac{1}{P} + \frac{1}{R} = \frac{2 \cdot TP + FP + FN}{TP}$, so F1 reads directly off the four cells of a [[Confusion Matrix]]:

$$F_1 = \frac{2 \cdot TP}{2 \cdot TP + FP + FN} = \frac{TP}{TP + \frac{FN + FP}{2}}$$

A true positive counts once, and each of the two error types counts half. The true-negative cell does not appear at all.

### Why the harmonic mean punishes the smaller input

The harmonic mean gives much more weight to low values, which is worth a worked pair rather than an assertion. Take a classifier with $P = 0.9$ and $R = 0.1$. The arithmetic mean is $\frac{0.9 + 0.1}{2} = 0.5$, which reads as a mediocre but functioning model. The harmonic mean is

$$F_1 = \frac{2 \cdot 0.9 \cdot 0.1}{0.9 + 0.1} = \frac{0.18}{1.0} = 0.18$$

The mechanism is the reciprocals: $1/R = 10$ swamps $1/P \approx 1.11$ in the sum, so the small value sets the answer almost by itself. In general the harmonic mean is bounded above by the smaller of its two inputs times $2$, and it goes to $0$ as either input goes to $0$ no matter how close the other sits to $1$. That is the whole reason to prefer it here: a classifier can be gamed to perfect precision by predicting positive exactly once, and to perfect recall by predicting positive always, and F1 refuses to reward either.

The numbers for the SGD 5-detector on [[MNIST]], from its confusion matrix cells $TP = 3530$, $FP = 687$, $FN = 1891$:

$$P = \frac{3530}{3530 + 687} = 0.8371, \qquad R = \frac{3530}{3530 + 1891} = 0.6512, \qquad F_1 = \frac{3530}{3530 + \frac{1891 + 687}{2}} = 0.7325$$

The random forest on the same target reaches $F_1 = 0.9275$, from $P = 0.9897$ and $R = 0.8725$.

### The $F_\beta$ family

Balancing precision and recall equally is a choice, not a law, and it is the choice that makes F1 wrong for some tasks. The generalization keeps the harmonic structure and adds a weight:

$$F_\beta = (1 + \beta^2) \cdot \frac{P \cdot R}{\beta^2 P + R} = \frac{1 + \beta^2}{\frac{1}{P} + \frac{\beta^2}{R}}$$

The second form is the useful one: it is a *weighted* harmonic mean, carrying weight $1$ on the precision term and weight $\beta^2$ on the recall term. A larger weight on $1/R$ means a small recall contributes more to the denominator, so it hurts the score more. Hence $\beta > 1$ weights recall more heavily and $\beta < 1$ weights precision more heavily, with $\beta = 1$ recovering F1, $\beta \to \infty$ approaching $R$ alone and $\beta \to 0$ approaching $P$ alone. In confusion-matrix cells,

$$F_\beta = \frac{(1 + \beta^2) \cdot TP}{(1 + \beta^2) \cdot TP + FP + \beta^2 \cdot FN}$$

which shows the same thing from the other side: a missed positive ($FN$) is charged $\beta^2$ times what a false alarm ($FP$) is charged. Picking $\beta$ is the principled answer to the observation that F1 is not always appropriate for every task, and it is a better answer than abandoning a single number, because it states the asymmetry as a parameter instead of leaving it implicit.

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.metrics import f1_score, fbeta_score

f1_score(y_train_5, y_train_pred)              # 0.7325171197343846

fbeta_score(y_train_5, y_train_pred, beta=2)   # recall weighted 4x
fbeta_score(y_train_5, y_train_pred, beta=0.5) # precision weighted 4x
```

The 1.6 signatures are `f1_score(y_true, y_pred, *, labels=None, pos_label=1, average='binary', sample_weight=None, zero_division='warn')` and `fbeta_score(y_true, y_pred, *, beta, labels=None, pos_label=1, average='binary', sample_weight=None, zero_division='warn')`. Note the `*` sitting before `beta`: it is keyword-only and has no default, so `fbeta_score` cannot be called without stating the weight, which is the right design given that the whole point of the function is that weight.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `average` | which aggregation over labels | `"binary"` | `"binary"` reports the one class named by `pos_label` and is valid only on a binary target; `"macro"` averages the per-label scores unweighted, `"weighted"` averages them weighted by support, `"micro"` pools $TP$, $FP$ and $FN$ across labels before computing one score, `"samples"` scores each instance's label set and averages over instances, `None` skips averaging and returns the vector of per-label scores | leave it at `"binary"` for a two-class target; on a multilabel target it must be set explicitly, `"macro"` when every label matters equally and `"weighted"` when frequent labels should dominate |
| `pos_label` | the class treated as positive | `1` | names which class counts as positive; switching it exchanges $TP$ with $TN$ and $FP$ with $FN$, so the returned number becomes the other class's F1, which is a different value and not its complement | set it whenever the positive class is not labelled `1`, for instance a string or boolean target |
| `zero_division` | the value returned when a denominator is zero | `"warn"` | `"warn"` returns $0.0$ and raises a warning, `0.0` and `1.0` substitute those constants silently, `np.nan` excludes that label from the average entirely | `0.0` to silence the warning while keeping the pessimistic reading, `np.nan` when a label with no instances should not drag a macro average down |
| `beta` (`fbeta_score`) | $\beta$ | none, keyword-only and required | recall is weighted more relative to precision, by a factor of $\beta^2$ on the $1/R$ term of the reciprocal sum | $\beta = 2$ when a miss costs more than a false alarm, $\beta = 0.5$ when a false alarm costs more |

`sample_weight` also changes the returned number, replacing each count by a weighted sum, though the worked numbers above leave it unset. `labels` restricts which labels enter a macro, weighted or micro average, so it changes the number too, and it is only meaningful once `average` has left `"binary"`.

## VS

Against [[Accuracy]], the structural difference is one cell. Accuracy is $\frac{TP + TN}{TP + TN + FP + FN}$ and F1 is $\frac{2 \cdot TP}{2 \cdot TP + FP + FN}$: the true-negative count appears in the first and is absent from the second. That is exactly why F1 survives [[Class Imbalance]] and accuracy does not. A large, easy negative class inflates $TN$ and therefore accuracy, while leaving every term of F1 untouched. The price is that F1 is not symmetric under relabelling which class is positive, so which class you call positive is now a decision that changes the reported number.

Against reporting $P$ and $R$ as a pair, F1 buys comparability and pays with information. Two classifiers can share an F1 of $0.73$ with completely different operating behaviour, one cautious and one trigger-happy, and the single number cannot tell them apart. F1 is the right summary when you need to rank many candidates at once, and the pair is the right report when someone has to live with the model's errors.

## Where it is used

It is the [[Performance Measure]] to reach for once [[Accuracy]] has proved uninformative on an imbalanced target, and it is computed from [[Precision]] and [[Recall]], the two quantities it averages harmonically, both of which are themselves ratios of the cells of a [[Confusion Matrix]] that F1 can also be read off directly.

Its weakness is the [[Precision-Recall Tradeoff]]: F1 rewards classifiers whose precision and recall are similar, so a single balanced number is the wrong summary whenever the task genuinely wants one of them higher than the other. The standing example is a classifier that picks out videos safe for children, where you would rather reject many harmless videos than let one unsafe video through. That is a preference for precision over recall, and F1 will score such a classifier below a balanced one that is worse at the actual job. The fix is $F_\beta$ with $\beta < 1$, not a different metric.

On [[Multilabel Classification]] the `average` argument becomes load-bearing rather than incidental, since a per-label F1 has to be collapsed into a scalar before it can be reported, and the worked multilabel example prints both `"macro"` ($0.9764$) and `"weighted"` ($0.9778$) to show how little they differ when the labels are near balanced.
