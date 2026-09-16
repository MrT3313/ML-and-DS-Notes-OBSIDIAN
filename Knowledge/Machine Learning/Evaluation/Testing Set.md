---
note_kind: concept
aliases:
  - test data
  - holdout test set
  - train-test split
  - test-set
up: "[[Generalization]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## Definition

The test set is a slice of the data, set aside before any training, used exactly once to estimate how the final model will perform on data it has never seen. Its only job is to estimate [[Generalization]] error honestly.

The slice is taken before any exploration at all, not merely before training: plots, summaries, and correlation hunts run on the [[Training Set]] alone, because looking at the held-out rows biases every modelling choice you make afterwards ([[Data Snooping Bias]]).

## Formal statement

Split $D$ into $D_{\text{train}}$ of size $m_{\text{train}}$ and $D_{\text{test}}$ of size $m_{\text{test}}$ with $D_{\text{train}} \cap D_{\text{test}} = \emptyset$; Géron's default is 80/20, with a smaller test fraction when $m$ is large. The estimate is

$$\hat{\mathcal{L}}_{\text{gen}} = \frac{1}{m_{\text{test}}} \sum_{i \in D_{\text{test}}} \ell\big(h(\mathbf{x}^{(i)}), y^{(i)}\big)$$

Two requirements follow on how the split is drawn. It must be stable across runs, which is what a fixed [[Random Seed]] buys and what a hash of a per-row identifier buys more robustly ([[Random Sampling]]). And the test set must be representative of the population on whatever variable drives the target, which for small $m$ or a rare subgroup means drawing it with [[Stratified Sampling]] rather than uniformly.

### Reporting the score with an interval

$\hat{\mathcal{L}}_{\text{gen}}$ is itself an estimate from a finite sample, so a point estimate overstates what you know. With squared errors $e_i^2 = \big(h(\mathbf{x}^{(i)}) - y^{(i)}\big)^2$, build an interval for the mean squared error and take the square root of both endpoints; the root is monotone, so coverage carries over to [[Root Mean Squared Error]] unchanged.

SciPy 1.14, the bootstrap, which resamples the errors and assumes nothing about their shape:

```python
from scipy import stats

def rmse(squared_errors):
    return np.sqrt(np.mean(squared_errors))

confidence = 0.95
squared_errors = (final_predictions - y_test) ** 2
boot_result = stats.bootstrap([squared_errors], rmse,
                              confidence_level=confidence, random_state=42)
rmse_lower, rmse_upper = boot_result.confidence_interval
```

The book's printed alternative is a $t$-interval on the same quantity:

```python
np.sqrt(stats.t.interval(confidence, len(squared_errors) - 1,
                         loc=squared_errors.mean(),
                         scale=stats.sem(squared_errors)))
```

Both are legitimate, but the $t$-interval assumes the squared errors are roughly normal, and squared errors are non-negative and heavily right-skewed, so that assumption usually fails. The bootstrap is the safer default, and it is what the book's companion notebook uses.

## Where it is used

Comparing $\hat{\mathcal{L}}_{\text{gen}}$ with training error diagnoses [[Overfitting]]. It must not be used to choose between models or tune a [[Hyperparameter]]; that is what [[Holdout Validation]] and [[Cross-Validation]] are for, and using the test set for it produces a model tuned to the test set and an optimistic estimate. If the test set does not match production data, see [[Data Mismatch]].

### The single-use rule

The single-use rule has a corollary worth stating bluntly: once the number is in hand, resist tuning to improve it. A model tuned against the test set will not generalize better, and the number you would then report has already lost its meaning. If the score disappoints, the honest move is to go back to [[Model Selection]] on validation data, not to keep drawing from the same well.
