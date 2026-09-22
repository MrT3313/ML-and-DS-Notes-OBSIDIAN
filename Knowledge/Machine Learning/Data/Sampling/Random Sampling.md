---
note_kind: method
aliases:
  - random sampling
  - random split
  - simple random sampling
  - uniform sampling
  - shuffle split
  - train_test_split
  - hash split
  - ID hash split
up: "[[Sampling]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Draw a subset from a population with every unit equally likely to be selected: fix a fraction, and every member of the population carries that same chance of ending up in the draw, so the inclusion probability is one number rather than a per-unit quantity ([[Sampling]]). No structure is imposed on the draw, which is what makes it the easiest form of sampling to implement and the one reached for by default. Carving a [[Testing Set]] out of the available data is its most frequent application, and the same draw serves anywhere else a subset has to stand in for the whole. It is the right default when instances are independent and identically distributed and $m$ is large enough that chance alone is unlikely to skew the sample; when a variable that matters is unevenly distributed, when a category is rare, or when $m$ is small, use [[Stratified Sampling]] instead. The three implementations below differ only in how membership is decided.

## Algorithm

Permutation route, where $r$ is the test ratio.

1. Draw a uniform permutation $\pi$ of $\{1, \dots, m\}$.
2. Send the first $\lfloor m \cdot r \rfloor$ positions to the test set.
3. Send the rest to the [[Training Set]].

Hash route.

1. Give each instance a stable identifier $\text{id}^{(i)}$.
2. Send it to the test set exactly when

   $$\text{crc32}\big(\text{id}^{(i)}\big) < r \cdot 2^{32}$$

3. Send every other instance to the training set.

CRC32 spreads identifiers roughly uniformly over $[0, 2^{32})$, so the fraction below the threshold is approximately $r$. Membership depends on the identifier alone: not on $m$, not on row order, not on a [[Random Seed]].

That is why the hash route exists. A seeded permutation is reproducible only while the dataset is frozen: append rows and $m$ changes, so the permutation changes and rows that trained last run are tested this run. Refresh a few times and the model has effectively seen everything, which is [[Data Snooping Bias]] by the back door. Hashing pins each row to one side forever, and new rows join the test set at rate $r$ unprompted.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| test ratio | $r$ | 0.25 in `train_test_split`, 0.2 by convention | test estimate less noisy, training set smaller so the fitted model is worse | 0.2 at moderate $m$, shrink it as $m$ grows since a fixed count is enough to pin the metric down |
| identifier column | - | none, the hash route needs one | - | pick a column that is unique, immutable, and never reassigned |
| `shuffle` | - | `True` | - | set `False` only for ordered data you intend to split by position |

`random_state` is left out on purpose: it fixes which rows the shuffle draws into the test set, so it changes the split that comes back, but it is pinned rather than tuned. Fix one integer for the life of the project, see [[Random Seed]].

## Failure modes

- Data that is not independent and identically distributed. A time series split at random puts future rows into training and leaks them into the past. Grouped rows (several readings per patient) and duplicates do the same, splitting a group across train and test.
- Small $m$, where a fair draw is unrepresentative by luck. Sampling 1000 people from a population that is 51.1% female lands outside 48.5% to 53.5% about 10.7% of the time: sampling noise, not a bug, producing [[Nonrepresentative Training Data]].
- A rare category that does not appear at all. The previous bullet is about a share coming back wrong; this one is about a share coming back zero, and a class the sample never contains is a class the model is never shown, so it fits as though the category did not exist. For a population of $N$ units of which a fraction $p$ carry the rare class, a sample of size $n = fN$ drawn without replacement contains none of them with probability

  $$P(\text{none}) = \frac{\binom{N(1-p)}{n}}{\binom{N}{n}} \approx (1-f)^{Np} \approx e^{-fNp}$$

  where $f$ is the sampling fraction, which is the test ratio $r$ when the draw is a split. A class holding 0.01% of a population of $N = 10^{6}$ has $Np = 100$ members, so a 1% sample misses it with probability $(0.99)^{100} = 0.3660$, which agrees with the exact hypergeometric value to four figures: roughly one draw in three, which is not a corner case. What decides this is $fNp$, the expected number of rare-class members drawn, and not the rarity or the sampling fraction on its own, since the same $p$ in a population of $N = 10^{8}$ gives $fNp = 10^{4}$ and a miss probability indistinguishable from zero. That is [[Class Imbalance]] biting at the sampling stage rather than at the loss, and [[Stratified Sampling]] removes it by construction, because a draw taken from inside the rare group cannot come back empty.
- The unstable split: a permutation reshuffles silently whenever the data is refreshed, reordered, or filtered, even with the seed fixed.
- `reset_index()` as the identifier column is safe only if new rows are appended at the end and none is ever deleted or reordered. Otherwise the index is reassigned and rows migrate across the split; a natural immutable key is better.
- `crc32(np.int64(identifier))` truncates toward zero, so a float identifier built from coordinates collides past the decimal point.

## Implementation

NumPy 2.x, using the modern `Generator` rather than legacy global state:

```python
import numpy as np

def shuffle_and_split_data(*, data, test_ratio=0.2, seed=None):
    rng = np.random.default_rng(seed)
    shuffled_indices = rng.permutation(len(data))
    test_set_size = int(len(data) * test_ratio)
    test_indices = shuffled_indices[:test_set_size]
    train_indices = shuffled_indices[test_set_size:]
    return data.iloc[train_indices], data.iloc[test_indices]

train_set, test_set = shuffle_and_split_data(data=housing, seed=42)
```

Python standard library `zlib`, the stable hash split:

```python
from zlib import crc32

def is_id_in_test_set(*, identifier, test_ratio=0.2):
    return crc32(np.int64(identifier)) < test_ratio * 2**32

def split_data_with_id_hash(*, data, id_column, test_ratio=0.2):
    ids = data[id_column]
    in_test_set = ids.apply(lambda id_: is_id_in_test_set(identifier=id_,
                                                          test_ratio=test_ratio))
    return data.loc[~in_test_set], data.loc[in_test_set]

housing_with_id = housing.reset_index()  # adds an `index` column
train_set, test_set = split_data_with_id_hash(data=housing_with_id, id_column="index")
```

scikit-learn 1.6:

```python
from sklearn.model_selection import train_test_split

train_set, test_set = train_test_split(housing, test_size=0.2, random_state=42)
```
