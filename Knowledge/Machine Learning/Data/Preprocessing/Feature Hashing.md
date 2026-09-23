---
note_kind: method
aliases:
  - feature hashing
  - hashing trick
  - hash trick
  - hash space
  - FeatureHasher
  - hashed feature
  - hashed features
up: "[[Feature Engineering]]"
sources:
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## What it does and when

Categories get treated as static, and in production they are not. An encoder learns its level set at fit time, and that set is a snapshot: new levels keep arriving afterwards, and how many will eventually exist is not knowable at the moment the column width has to be fixed. Amazon's brand field is the case at scale, already past two million brands with new ones minted daily; a seller registering a brand this afternoon is a level no fitted encoder has ever seen.

Two conventional answers are already in hand, and neither one is an answer. `handle_unknown="ignore"` on [[One-Hot Encoding]] keeps a fitted [[Pipeline]] alive on a level absent at fit time, but it does so by mapping every such level onto the same all-zero block, so the ten thousandth new brand and the first one are the same input. `min_frequency` and `max_categories` fold the rare tail into one shared infrequent column, which bounds the width but answers a different question: it is about levels that are seen and are rare, not about levels that are not seen at all. [[Ordinal Encoding]] has the same shape of non-answer in `handle_unknown="use_encoded_value"`, one sentinel code for everything new.

Feature hashing inverts the order the other encoders work in. Instead of learning the levels and deriving the width from them, fix the width first, then send each category through a hash function to decide which column it lands in. Nothing is fitted, so nothing needs refitting when the level set moves, and the feature matrix has the same number of columns on the day a brand is added as on the day before. Reach for it when the level set is genuinely unbounded, when it is large enough that $K$ one-hot columns is not affordable, or when the schema itself has to stay still, which is why it belongs in a [[Continual Learning]] setting: an update that changes the model's parameters without changing its input width is a far cheaper thing to ship than one that changes both.

The technique reached wide use through Vowpal Wabbit, a fast out-of-core learning system whose first public version carrying hashing, caching and online learning appeared in 2007, with John Langford the primary figure; the project describes itself as sponsored by Microsoft Research and, previously, Yahoo! Research, which is to say it began at Yahoo! and moved to Microsoft afterwards rather than the other way round. [[DMLS Ch05 Feature Engineering|DMLS chapter 5]] records, as of 2022, a reputation for the trick as hacky and looked down on in academic settings while being widely used in industry. That is a report of opinion at a date and nothing below depends on it.

## Algorithm or formula

Let $\mathcal{C}$ be the level set of a categorical [[Feature]], $b$ the size of the hash space in bits, and

$$h : \mathcal{C} \to \{0, 1, \dots, 2^{b} - 1\}$$

a hash function. Category $c$ is encoded by the index $h(c)$. The output width is $2^{b}$ and is chosen by picking $b$, so it is fixed before $\mathcal{C}$ is known and does not move when $\mathcal{C}$ does. This is the whole structural difference from [[One-Hot Encoding]], whose width is $K = |\mathcal{C}|$ and therefore a quantity the data gets to decide. With $b = 18$ the space holds $2^{18} = 262{,}144$ buckets and every category, including ones never seen at fit time, receives an index between $0$ and $262{,}143$.

### How often collisions happen and what they cost

Two categories can be handed the same index, and then the model cannot tell them apart. How often that happens is the birthday calculation. Write $K$ for the number of distinct levels and $M = 2^{b}$ for the number of buckets, and treat a good hash as placing each level uniformly and independently over the buckets.

A named pair shares a bucket with probability $1/M$. There are $\binom{K}{2}$ pairs, and expectation is linear whether or not the pair events are independent, so

$$\mathbb{E}[\text{colliding pairs}] = \binom{K}{2}\frac{1}{M} = \frac{K(K-1)}{2 \cdot 2^{b}} \approx \frac{K^{2}}{2^{b+1}}$$

For one particular level, it escapes each of the other $K - 1$ levels independently with probability $1 - 1/M$, so it escapes all of them with probability $(1 - 1/M)^{K-1}$ and therefore

$$p_{\text{collide}} = 1 - \left(1 - 2^{-b}\right)^{K-1} \approx 1 - e^{-(K-1)/2^{b}}$$

where the approximation is $(1-x)^{t} \approx e^{-xt}$ for small $x$. Summing the Bernoulli indicator over levels gives $K \, p_{\text{collide}}$ levels expected to share their bucket with at least one other, which is the quantity a measured collision rate reports.

Read off what controls it: the exponent is $(K-1)/2^{b}$, so the load factor $K/2^{b}$ decides everything and neither $K$ nor $b$ means anything alone. Two million brands into $b = 18$ is a load of $7.6$ and $p_{\text{collide}} = 0.9995$, which is to say the space is hopeless for that column no matter how good the hash. The same two million into scikit-learn's default $b = 20$ still collides at $0.85$; $b = 25$ brings it to $0.058$. Doubling the space halves the load and, while the load is small, roughly halves the collision rate, which is why $b$ is chosen in whole bits rather than tuned finely.

What a collision costs, as against how often one happens, is measured rather than derived. Lucas Bernardi's Booking.com engineering post of January 2018 fits a logistic regression under Vowpal Wabbit on three datasets, Booking.com's own (roughly 200k features and 250M examples over four classes) plus the Criteo and Avazu click datasets, and plots held-out log loss against the measured proportion of colliding features. Quoting it: "even for 50% colliding features, the performance lost is much less than half percent". Two riders travel with that number. The metric is log loss on a held-out set, so the half percent is half a percent of a log loss and not of accuracy or of anything else, and the same post reports the effect as dataset dependent, with Booking.com's own data hurt more than twice as much as Criteo's at that same 50%. It is a company engineering post rather than a paper, three datasets under one learner, and it is worth exactly that much.

Two mitigations exist and they are not equals. Widening the space is the direct one, and its effect is exactly the formula above. The other is usually stated loosely, as choosing a hash that sends similar categories to nearby values, and its precise form is locality-sensitive hashing (Indyk and Motwani, STOC 1998), which promises something about collision probability rather than about nearness of outputs. A family $\mathcal{H}$ is $(r, cr, P_1, P_2)$-sensitive with respect to a named metric $d$ when, for $h$ drawn from $\mathcal{H}$,

$$d(\mathbf{u}, \mathbf{v}) \le r \implies \Pr_{h}[h(\mathbf{u}) = h(\mathbf{v})] \ge P_1, \qquad d(\mathbf{u}, \mathbf{v}) \ge cr \implies \Pr_{h}[h(\mathbf{u}) = h(\mathbf{v})] \le P_2$$

with $P_1 > P_2$. Collisions are made likelier between close points and rarer between distant ones, in expectation over the draw, and no individual pair is promised anything. It is also a family per metric rather than one function, MinHash for Jaccard, random hyperplanes for cosine, $p$-stable projections for $\ell_p$, so using it on categories means first having a metric on the categories, which is the step that is usually missing. Where that metric does exist, a collision costs less, because the two levels sharing an index were alike to begin with.

### The sign hash

Weinberger, Dasgupta, Attenberg, Langford and Smola (ICML 2009) give the construction that makes collisions cancel rather than accumulate. Alongside $h$, draw a second hash $\xi : \mathcal{C} \to \{-1, +1\}$, and define the hashed vector coordinate by coordinate as

$$\phi_i(\mathbf{x}) = \sum_{j \,:\, h(j) = i} \xi(j) \, x_j$$

with a one-hot input the special case where exactly one $x_j$ is $1$. Their Lemma 2 is that the resulting hash kernel is unbiased,

$$\mathbb{E}_{h,\xi}\big[\langle \mathbf{x}, \mathbf{x}' \rangle_{\phi}\big] = \langle \mathbf{x}, \mathbf{x}' \rangle$$

and the reason is visible in the cross terms. Expanding the hashed inner product gives terms $\xi(j)\xi(j') x_j x'_{j'}$ over pairs $j, j'$ landing in the same bucket. For $j \neq j'$ the product $\xi(j)\xi(j')$ is $+1$ and $-1$ with equal probability, so that term has mean zero and a collision adds noise rather than a systematic amount; only the $j = j'$ terms survive, where $\xi(j)^{2} = 1$ and the original inner product is recovered. Without the sign every collision adds a strictly positive cross term, so the distortion is one-directional and grows with the number of collisions. The paper's Theorem 3 goes further and bounds the deviation exponentially, with the variance of the hashed inner product of unit vectors at $O(1/M)$.

A hash also appears in [[Random Sampling]], doing a different job. There it consumes a row's stable identifier and the hashed value is compared against a threshold to decide which side of a split that row falls on, so the output is a membership decision about an instance and that note carries the alias `hash split`. Here the hashed value is a column position for a category, so the output is a coordinate in $\phi(\mathbf{x})$, and the naming for the split use stays over there.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `n_features` | $M = 2^{b}$ | `1048576`, which is $2^{20}$ | wider output, lower load factor $K/M$, so fewer collisions and more parameters for the model to fit | estimate $K$, then pick $b$ so that $1 - e^{-(K-1)/2^{b}}$ sits under the collision rate you will accept, and step in whole bits |
| `alternate_sign` | $\xi$ | `True` | `False` drops the sign hash, so colliding levels add instead of cancelling and the hashed inner product is biased upward, worst at small `n_features` | leave on; turn it off only where the downstream model requires non-negative input |
| `input_type` | | `'dict'` | `'string'` takes an iterable of strings per row with value $1$ implied, `'pair'` takes name and value pairs; the choice decides what is fed to $h$ and therefore what collides | `'string'` for bare category labels, `'dict'` when a numeric value rides along with each name |

`dtype` changes the stored type of the output entries and not which entry is written where, so it is pinned at `np.float64` rather than tuned.

## Failure modes

- Newcomers collide differently from how a catch-all bucket pools them, and this is the distinction worth being precise about. With `handle_unknown="ignore"` on [[One-Hot Encoding]] there is one destination for everything unseen: every new level maps to the same all-zero block, the members are mutually indistinguishable, and whatever the model reads off that block is an average over an arbitrary mixture that keeps growing. With hashing there is no shared destination at all; a new level lands uniformly at random in the whole space and shares its index with roughly $K/M$ existing levels chosen by the hash. The contrast is one pooled bucket for all arrivals against collisions spread uniformly, and it is not a contrast between popular and unpopular levels, since the level a newcomer collides with is drawn without regard to frequency and may itself be as rare as the newcomer.
- The map is one way, so the encoding is not invertible. Given an index there is no way back to the category, `FeatureHasher` defines no `get_feature_names_out()` because it has nothing to return, and a large coefficient or a high permutation importance on column $i$ names a bucket rather than a brand. Any workflow that reads [[Feature Engineering|feature importances]] back to decide what to keep loses its handle here, and recovering it means keeping an external table of which categories were seen hashing to which index, which is the vocabulary the method existed to avoid storing.
- An undersized space fails quietly. Nothing errors when the load factor passes $1$: the matrix is the right shape, the fit runs, and two million brands into $2^{18}$ buckets simply means each column carries about eight brands stacked on top of each other. The failure shows up only if the collision rate is computed, which is a reason to compute it before the fit rather than after a disappointing score.
- One global hash space lets different fields collide with each other. Vowpal Wabbit hashes every feature into a single space, so a country value and a user id can land in the same bucket and share one coefficient. The fix is to hash per field, or to build the key from the field name and the value together so that `brand=nike` and `country=nike` are different strings to $h$.
- Colliding levels are handed identical weights, which produces models that are not wrong so much as absurd on inspection. Lucas Bernardi's Booking.com write-up gives the example of Denmark and Greece sharing a bucket in a hotel price model and therefore contributing exactly the same amount, and the same happens to a pair of binary flags that have nothing to do with each other.

## Implementation

scikit-learn 1.6 ships `FeatureHasher`, with `n_features` the hash space $M$ of the formula above, defaulting to $2^{20} = 1{,}048{,}576$, and `alternate_sign` defaulting to `True`, which applies the sign hash $\xi$ and is documented as approximately conserving the inner product in the hashed space even when `n_features` is small. The underlying $h$ is the signed 32 bit version of MurmurHash3, and `n_features` is not required to be a power of two even though thinking in bits is the natural way to pick it. The estimator is stateless: `fit` only validates the parameters and learns nothing, which is exactly the property the fixed width buys, though calling `fit_transform` is still preferred over `transform` alone so that validation actually runs.

```python
from sklearn.feature_extraction import FeatureHasher

hasher = FeatureHasher(n_features=2**18, input_type="string")
brands = [["nike"], ["adidas"], ["a-brand-registered-this-morning"]]
X = hasher.fit_transform(brands)      # (3, 262144) scipy sparse, nothing fitted
```

Build the key from the field name and the value to keep fields out of each other's buckets, and pass a numeric value alongside where one exists:

```python
rows = [
    {"brand=nike": 1, "country=NL": 1, "nights": 3},
    {"brand=adidas": 1, "country=GR": 1, "nights": 1},
]
hasher = FeatureHasher(n_features=2**20, input_type="dict", alternate_sign=True)
X = hasher.fit_transform(rows)
```

Size the space from the formula rather than by habit, with NumPy 2.x:

```python
import numpy as np

def collision_rate(*, n_levels, n_features):
    """Share of levels expected to share a bucket with at least one other."""
    return 1.0 - (1.0 - 1.0 / n_features) ** (n_levels - 1)

collision_rate(n_levels=2_000_000, n_features=2**18)   # 0.9995
collision_rate(n_levels=2_000_000, n_features=2**20)   # 0.8515
collision_rate(n_levels=2_000_000, n_features=2**25)   # 0.0579

bits = np.arange(18, 28)
rates = [collision_rate(n_levels=2_000_000, n_features=2**b) for b in bits]
```

For raw text rather than a category column, `HashingVectorizer` is the same idea with tokenization in front of it.
