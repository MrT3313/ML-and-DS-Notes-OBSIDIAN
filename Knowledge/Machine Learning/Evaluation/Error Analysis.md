---
note_kind: method
aliases:
  - error analysis
  - analysing errors
  - analyzing errors
  - confusion analysis
  - misclassification analysis
  - error profile
  - reading the errors
up: "[[Confusion Matrix]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## What it does and when

Error analysis is what you do once a model is trained and scored, when the useful question stops being how many mistakes it makes and becomes which ones. Its output is not a number, it is a decision about the next thing to try: which classes need more instances, which need a [[Feature Engineering|feature]] the model currently cannot see, which confusions are an artifact of how the inputs were prepared rather than a fact about the classes. Reach for it when a scalar [[Performance Measure]] has said a model is good enough to keep and not good enough to ship, and reach for it especially on [[Multiclass Classification]], where the $K \times K$ [[Confusion Matrix]] carries structure that any single score flattens away.

The instrument is the confusion matrix throughout, the table every [[Classification]] model is judged off. This note is the protocol for reading it; that note is the object being read.

> [!warning]
> Run this on out-of-fold predictions over the [[Training Set]], never on the [[Testing Set]] and never on predictions the model made about rows it was fitted on. Predictions on fitted rows describe memorisation rather than [[Generalization]], so the errors you would be analysing are not the errors the shipped model will make. And every remedy you adopt after looking is a modelling choice conditioned on whatever you looked at, so looking at the test set turns it into a validation set and its final score into [[Data Snooping Bias]]. Every matrix read below takes the first route, built from `cross_val_predict` on the training set, with the test set left untouched ([[HOML Ch03 Classification|HOML chapter 3]] holds it back until its exercises).

## Algorithm

1. Produce out-of-fold predictions with `cross_val_predict`, so each instance is predicted by an estimator fitted on the folds that excluded it. See [[Cross-Validation]]. Apply the same preprocessing you intend to ship; on [[MNIST]] the pixels are standardized first, because the [[Stochastic Gradient Descent Classifier]] read below is scale sensitive.
2. Display the raw counts. This is the sanity check, not the finding. Counts are dominated by class size and by the diagonal, so a common class shows a large cell whatever its error rate, and everything off the diagonal is squeezed into the dark end of the colour scale.
3. Normalize by row, `normalize="true"`, which divides each row by its own support. Now cell $(i, j)$ reads "of the instances that really were class $i$, this fraction was called $j$", and classes of different sizes become comparable. Under [[Class Imbalance]] this is the step that stops a rare class with a terrible rate from hiding behind a common class with a good one.
4. Mask the correct predictions with `sample_weight = (y_pred != y_true)`. The boolean array weights every correct prediction by $0$ and every mistake by $1$, so the diagonal empties and the picture becomes the distribution of the errors alone. This is the step that makes the structure visible, because the diagonal otherwise carries almost all the mass.
5. Normalize the masked matrix by row and by column, and read both. They answer different questions. The row version answers "when the true class is $i$, what does the model predict instead?". The column version, `normalize="pred"`, answers "when the model predicts $j$ wrongly, what was it really?". A confusion that looks severe one way can look mild the other, and reading only one of them is the standard way to draw a directional conclusion the data does not support.
6. Pull out the individual instances behind the pair that interests you: the true $a$ predicted $a$, true $a$ predicted $b$, true $b$ predicted $a$ and true $b$ predicted $b$ blocks. Aggregates say which pair is confused; only the instances say what about them is confusable.
7. Convert the finding into a next action, and re-run the whole protocol after making the change, since fixing one confusion redistributes the rest.

### Why both axes have to be read

On the ten-digit MNIST target with a standardized-pixel [[Stochastic Gradient Descent Classifier]], the row-normalized full matrix puts class $5$ lowest on the diagonal at about $82$ percent correct, against about $94$, $95$ and $94$ percent for classes $0$, $1$ and $6$. So the model is worst at fives, which the raw counts do not say plainly: the raw diagonal cell for class $5$ is $4444$ against $6400$ for class $1$, and part of that gap is simply that there are fewer fives in the file.

The error-only matrices are where the structure appears, and the two axes disagree in an instructive way.

- **By row**, column $8$ dominates almost every line: of the errors made on true zeros about $65$ percent are called $8$, on true ones about $62$ percent, on true twos about $51$ percent, on true fives about $55$ percent, on true nines about $44$ percent. Read alone, this says the model pours its mistakes into $8$.
- **By column**, that same column $8$ is unremarkable, running at roughly $5$ to $19$ percent across the classes. The errors landing on $8$ arrive in small shares from every class at once, so there is no single class-to-$8$ pair to go and fix.
- The pairs that are severe on **both** axes are the real confusions. True $3$ called $5$ is about $22$ percent of the threes' errors and about $34$ percent of what gets wrongly called $5$; true $5$ called $3$ is about $17$ percent and about $35$ percent the other way. True $7$ called $9$ is about $36$ and about $37$ percent. True $9$ called $7$ is about $21$ percent of the nines' errors and about $56$ percent of everything wrongly called $7$, the single largest cell in the column-normalized display.

Percentages above are read off the figures saved in the [[HOML Ch03 Classification|HOML chapter 3]] notebook, which rounds to whole percent, so treat them as rounded readings rather than as exact ratios.

The pair worth plotting image by image is $3$ against $5$, in four blocks of twenty five images each. The mechanism that makes those blocks worth looking at is a property of the model rather than of the digits: a linear classifier holds one weight per pixel per class, `coef_` of shape (number of classes, $784$), and scores an image by a weighted sum of its pixels. A stroke that moves a few pixels lands on different weights entirely, so two threes that a person reads as identical can score differently, and a three whose ink happens to fall where the five weights are positive is called a five. Nothing in that model knows that a shape can translate.

### Acting on the finding

The moves error analysis points to, once a confusion is established:

- **More instances of the confused classes**, so the model has examples of the distinction it is failing on. This is the [[Insufficient Training Data]] answer.
- **Engineered features that expose the distinction the pixels do not**, ex counting closed loops, which separates $8$ with two from $6$ with one and $5$ with none. See [[Feature Engineering]].
- **Preprocessing that removes the nuisance variation**, centring and de-skewing the images so a shifted or rotated stroke stops landing on foreign weights. Worth knowing that MNIST is already centred on each digit's centre of mass, so this means going further than the file already goes, not correcting an oversight.
- **Growing the training set with transformed copies of its own images**, which is [[Data Augmentation]]. In the [[HOML Ch03 Classification|HOML chapter 3]] notebook this is exercise 2 and its solution, headed `## 2. Data Augmentation`, which shifts every training image one pixel in each of four directions, retrains, and reports about a half point of accuracy. That notebook's Error Analysis section contains no augmentation prose, so the connection is one it makes across its own sections rather than in a single paragraph.

Uncertain: the notebook's Error Analysis section is code only, carrying no prose at all, so which of the first three remedies the printed [[HOML Ch03 Classification|HOML chapter 3]] text actually proposes and in what words could not be checked against a primary source. The book text is paywalled. Treat the first three bullets as the moves this analysis licenses, not as verified quotations of a source.

## Hyperparameters

None. The arguments that change what the reading says, `normalize` and the error mask passed as `sample_weight`, belong to the confusion matrix the protocol reads rather than to the protocol, and they are tabulated in [[Confusion Matrix]].

## Failure modes

- **Analysing predictions the model was fitted on.** `predict` on the training set shows you a model that has memorized the rows, so the confusions you find are the ones an overfitted model no longer makes and the ones it will make in production go unseen. `cross_val_predict` is the fix, and it is what step 1 of the protocol calls for.
- **Analysing the test set.** Every remedy adopted after looking at it is a choice conditioned on it, so the final score it later produces is optimistic and no correction recovers the honest number. That is [[Data Snooping Bias]] reached by a different road than premature exploration.
- **Reading one normalization axis and concluding in a direction the other does not support.** The MNIST column $8$ above is the standing case: dominant by row, unremarkable by column, so "the model turns everything into eights" survives the first display and dies on the second.
- **Acting on a confusion built from too few instances.** A masked, row-normalized cell reading $50$ percent can rest on two errors out of four. The rate is computed over the errors of that class alone, and for a class the model is mostly right about that denominator is small, so keep the raw count display alongside and check the cell is real before spending a week on it.
- **Treating a confusion as symmetric.** True $9$ called $7$ and true $7$ called $9$ are separate cells with separate denominators and they routinely differ; [[Precision]] is what the column of the matrix reports and [[Recall]] is what the row reports, and the same asymmetry runs through every reading of the table.
- **Stopping at the aggregate.** The matrix names the pair; it says nothing about what the instances of that pair have in common. Only looking at the misclassified inputs themselves turns "threes and fives are confused" into a hypothesis you can act on.

## Implementation

scikit-learn 1.6:

```python
from sklearn.model_selection import cross_val_predict
from sklearn.metrics import ConfusionMatrixDisplay

# out-of-fold predictions, on the training set, after the same scaling the model ships with
y_train_pred = cross_val_predict(sgd_clf, X_train_scaled, y_train, cv=3)

# 1. raw counts: the sanity check
ConfusionMatrixDisplay.from_predictions(y_train, y_train_pred)

# 2. rates within each true class: comparable across class sizes
ConfusionMatrixDisplay.from_predictions(y_train, y_train_pred,
                                        normalize="true", values_format=".0%")

# 3. errors only, as a share of each TRUE class's errors
sample_weight = (y_train_pred != y_train)
ConfusionMatrixDisplay.from_predictions(y_train, y_train_pred,
                                        sample_weight=sample_weight,
                                        normalize="true", values_format=".0%")

# 4. the same errors, as a share of each PREDICTED class's errors
ConfusionMatrixDisplay.from_predictions(y_train, y_train_pred,
                                        sample_weight=sample_weight,
                                        normalize="pred", values_format=".0%")

# 5. the instances behind one confused pair
cl_a, cl_b = '3', '5'
X_aa = X_train[(y_train == cl_a) & (y_train_pred == cl_a)]   # 3 called 3
X_ab = X_train[(y_train == cl_a) & (y_train_pred == cl_b)]   # 3 called 5
X_ba = X_train[(y_train == cl_b) & (y_train_pred == cl_a)]   # 5 called 3
X_bb = X_train[(y_train == cl_b) & (y_train_pred == cl_b)]   # 5 called 5
```

The 1.6 signature is `ConfusionMatrixDisplay.from_predictions(y_true, y_pred, *, labels=None, sample_weight=None, normalize=None, display_labels=None, include_values=True, xticks_rotation='horizontal', values_format=None, cmap='viridis', ax=None, colorbar=True, im_kw=None, text_kw=None)`. True labels come first; swapping the two positional arguments transposes the display and silently exchanges the two readings in step 5.

`normalize` accepts `'true'` (over the true conditions, meaning rows), `'pred'` (over the predicted conditions, meaning columns), `'all'` (over the total number of samples) and `None`, which is the default and leaves counts alone. `sample_weight` is documented only as "Sample weights", and the trick is that a boolean array is a perfectly good weight vector: each cell becomes $\sum_{n \in \text{cell}} w^{(n)}$, so a diagonal cell containing only correct predictions sums weights of $0$ and empties.

The class labels arrive from OpenML as strings, which is why the comparisons in step 5 are written against `'3'` and `'5'` rather than `3` and `5`. See [[MNIST]].

The companion notebook's cells around these calls also set `plt.rc('font', size=...)`, build subplot grids, set titles and call `save_fig(...)`. None of that changes a computed value, and `save_fig` is a helper defined in the companion repository that raises `NameError` if pasted without it.
