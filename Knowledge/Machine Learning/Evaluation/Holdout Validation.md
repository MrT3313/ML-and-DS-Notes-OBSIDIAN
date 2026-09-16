---
note_kind: method
aliases:
  - validation set
  - dev set
  - development set
  - holdout set
  - train-validation-test split
up: "[[Model Selection]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## What it does and when

Holdout validation carves a validation set out of the [[Training Set]], trains candidate models on the remainder, and picks the one that scores best on the validation set. The winner is then retrained on the full training set and evaluated once on the [[Testing Set]]. Use it whenever there is more than one candidate model or [[Hyperparameter]] setting, which is always.

## Algorithm

1. Split $D_{\text{train}}$ into $D_{\text{fit}}$ and $D_{\text{val}}$ with $|D_{\text{val}}| = v \cdot m_{\text{train}}$.
2. For each candidate $c$: fit $h_c$ on $D_{\text{fit}}$, compute $\mathcal{L}(h_c, D_{\text{val}})$.
3. Choose $c^{*} = \arg\min_c \mathcal{L}(h_c, D_{\text{val}})$.
4. Refit $h_{c^{*}}$ on all of $D_{\text{train}}$; report $\mathcal{L}(h_{c^{*}}, D_{\text{test}})$.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| validation fraction | $v$ | 0.2 | estimate less noisy, less data to fit on | larger $m$ allows smaller $v$ |

## Failure modes

- Small $D_{\text{val}}$: the selection is decided by noise, so the wrong candidate wins.
- Large $D_{\text{val}}$: models are fit on much less data than the final one, so the comparison is between weaker cousins of the models actually deployed.
- Repeated use for many candidates overfits the validation set itself; [[Cross-Validation]] softens this at higher compute cost.

## Implementation

scikit-learn 1.6:

```python
from sklearn.model_selection import train_test_split
X_fit, X_val, y_fit, y_val = train_test_split(X_train, y_train, test_size=0.2, random_state=42)
```
