---
note_kind: concept
aliases:
  - feature extraction
  - feature transformation
  - feature encoding
  - feature selection
  - attribute combination
  - derived features
up: "[[Feature]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

Feature engineering is the work of deciding what the [[Model]] actually sees: deriving new columns, reshaping existing ones, and discarding the rest so the signal in the raw data reaches the learning algorithm in a form it can exploit. It runs before fitting, and it is part of the predictor rather than part of the data. It is also the part of the work that needs to know what the columns mean, since which ratio is worth building and which column is a proxy for the answer are questions about the domain and not about the array.

## Formal statement

It is a map applied to every instance before the hypothesis sees it,

$$\phi: \mathbb{R}^{n} \to \mathbb{R}^{d}, \qquad \hat{y}^{(i)} = h\big(\phi(\mathbf{x}^{(i)})\big)$$

so the fitted object is the composition $h \circ \phi$, not $h$ alone. Two consequences. When $d > n$, a $\phi$ that adds ratios, products, or basis functions lets a *linear* $h$ express a nonlinear function of $\mathbf{x}$, which is why a linear model on good features routinely beats a flexible model on raw ones. And because $\phi$ lives inside the predictor, it must be the identical map at training and at inference: every parameter it carries (a median, a scale, a category vocabulary) is estimated on the [[Training Set]] alone and reapplied unchanged, which is what [[Pipeline]] enforces.

## VS

The honest contrast between engineered and learned features is about where $\phi$ comes from, and it falls straight out of the composition above.

**Engineered.** $\phi$ is written down by a person out of domain knowledge and fixed before the fit. The optimizer chooses $h$ and nothing else; $\phi$ carries fitted constants such as a median or a vocabulary, but its *form* was specified rather than searched for.

**Learned.** $\phi$ carries parameters of its own, and they are fitted by the same objective that fits $h$, so the thing being optimized is the composition end to end rather than $h$ sitting on top of a map somebody chose. [[Embedding]] is the instance this vault holds: a table of coordinates trained by the gradient that trains everything downstream of it.

The term the literature settled on for the second is **representation learning**, with Bengio, Courville and Vincent (*IEEE TPAMI* 35(8), 2013) the standing reference. "Feature learning" is a genuine synonym rather than a loose one, and that same paper uses both, carrying "feature learning" in its own index terms and its abstract; what separates them is currency rather than meaning, and "representation learning" is the more general of the two and the one that names the field and its conference. Their statement of why any of it is worth doing is the sharpest version of this note's own subject: the design of the preprocessing that produces a usable representation is where the effort actually goes, it is labour-intensive, and making learning algorithms less dependent on it is the goal. Neither name is claimed as an alias here, because a note defining either would have to define it against a model family this vault has no note for.

The flat version of the claim, that with deep learning you no longer handcraft features, is a statement about the state of a field and it needs a date on it. [[DMLS Ch05 Feature Engineering|Huyen]] is writing in 2022, and **as of September 2026 the claim holds in some settings and not in others.** Where the input is a raw signal, an image, audio, a span of text, a fitted $\phi$ has displaced the hand-written one almost entirely, and the practical choice is which pretrained map to adopt rather than whether to write one. Where the input is a table of columns assembled from a warehouse, hand-specified $\phi$ still does most of the work and gradient-boosted trees over engineered columns is still a strong default. So the split is by input type rather than by year, and "deep learning removes feature engineering" overstates the first case and is simply wrong about the second.

## Where it is used

**Attribute combination** builds a feature as a ratio or product of existing ones, and it is the branch [[HOML Ch02 End-to-End Machine Learning Project|HOML chapter 2]] spends its effort on. A district's `total_rooms` is nearly useless alone, because districts differ wildly in size and the count therefore measures population more than housing. Dividing that out fixes it: against `median_house_value`, `total_bedrooms` correlates at $0.055$ while `bedrooms_ratio` (bedrooms over rooms) correlates at $-0.256$, far stronger and opposite in sign, since a district whose homes are mostly bedrooms is a district of small homes. [[Correlation]] is how you check whether a candidate earned its place, and after the fit [[Feature Importance]] is how you check which of the engineered columns actually paid.

**Transformation** reshapes a column to suit the model's assumptions, through [[Feature Scaling]] or [[Feature Distribution Transformation]]. **Encoding** turns categories into numbers, through [[One-Hot Encoding]] or [[Ordinal Encoding]], and through [[Feature Hashing]] where the level set keeps growing in production and a fitted vocabulary would go stale. **Crossing** builds one feature out of two or more so that an additive model can express an interaction it otherwise cannot, which is [[Feature Crossing]]. **Selection** drops what does not pay, the practical answer to [[Irrelevant Features]] and the neighbour of [[Dimensionality Reduction]]. **Extraction** pulls structured signal out of a raw one, ex a `day_of_week` from a timestamp, and its deeper case is [[Embedding]], where the extracted representation is fitted against a downstream objective rather than specified.

One caution that belongs to this whole list rather than to any one branch: a feature computed *across rows* leaks. A ratio of two columns in the same row uses nothing but that row and is safe anywhere, while a group statistic or a target-encoded mean computed over the full dataset carries held-out labels into training, which is [[Data Leakage]] rather than anything an analyst chose to look at. The fix is the same one the composition above already demands, fitting the statistic inside the [[Pipeline]] on the training fold alone.

The substrate this work runs on is part of the choice. A conformed relational schema reached through SQL is an awkward place to do it rather than an impossible one, since a warehouse can fit some model families in place without the data leaving it, and it stops being an option at all once the input is a photograph or free text rather than columns. That is much of why the reshaping that happens before a fit tends to move to the files of a [[Data Lake]], where a reader imposes whatever structure the work needs.
