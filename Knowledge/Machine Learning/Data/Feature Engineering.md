---
note_kind: concept
aliases:
  - feature extraction
  - feature transformation
  - feature encoding
  - feature selection
  - attribute combination
  - feature crosses
  - derived features
up: "[[Feature]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

Feature engineering is the work of deciding what the [[Model]] actually sees: deriving new columns, reshaping existing ones, and discarding the rest so the signal in the raw data reaches the learning algorithm in a form it can exploit. It runs before fitting, and it is part of the predictor rather than part of the data.

## Formal statement

It is a map applied to every instance before the hypothesis sees it,

$$\phi: \mathbb{R}^{n} \to \mathbb{R}^{d}, \qquad \hat{y}^{(i)} = h\big(\phi(\mathbf{x}^{(i)})\big)$$

so the fitted object is the composition $h \circ \phi$, not $h$ alone. Two consequences. When $d > n$, a $\phi$ that adds ratios, products, or basis functions lets a *linear* $h$ express a nonlinear function of $\mathbf{x}$, which is why a linear model on good features routinely beats a flexible model on raw ones. And because $\phi$ lives inside the predictor, it must be the identical map at training and at inference: every parameter it carries (a median, a scale, a category vocabulary) is estimated on the [[Training Set]] alone and reapplied unchanged, which is what [[Pipeline]] enforces.

## Where it is used

**Attribute combination** builds a feature as a ratio or product of existing ones, and it is where Chapter 2 spends its effort. A district's `total_rooms` is nearly useless alone, because districts differ wildly in size and the count therefore measures population more than housing. Dividing that out fixes it: against `median_house_value`, `total_bedrooms` correlates at $0.055$ while `bedrooms_ratio` (bedrooms over rooms) correlates at $-0.256$, far stronger and opposite in sign, since a district whose homes are mostly bedrooms is a district of small homes. [[Correlation]] is how you check whether a candidate earned its place.

**Transformation** reshapes a column to suit the model's assumptions, through [[Feature Scaling]] or [[Feature Distribution Transformation]]. **Encoding** turns categories into numbers, through [[One-Hot Encoding]] or [[Ordinal Encoding]]. **Selection** drops what does not pay, the practical answer to [[Irrelevant Features]] and the neighbour of [[Dimensionality Reduction]]. **Extraction** pulls structured signal out of a raw one, ex. a `day_of_week` from a timestamp, or an embedding from text.

The substrate this work runs on is part of the choice. A conformed relational schema reached through SQL is an awkward place to do it rather than an impossible one, since a warehouse can fit some model families in place without the data leaving it, and it stops being an option at all once the input is a photograph or free text rather than columns. That is much of why the reshaping that happens before a fit tends to move to the files of a [[Data Lake]], where a reader imposes whatever structure the work needs.

### Feature importance closes the loop

After fitting, the importances say which engineered features paid off. On the tuned housing forest, three of the top five are engineered ratios, behind only log-transformed median income. `get_feature_names_out()` on the preprocessing step is what makes that readable, since the importances are otherwise an anonymous array of column positions.

scikit-learn 1.6:

```python
final_model = rnd_search.best_estimator_        # preprocessing + random_forest
importances = final_model["random_forest"].feature_importances_
sorted(zip(importances,
           final_model["preprocessing"].get_feature_names_out()),
       reverse=True)
```

Read that ranking as a hint. Impurity-based `feature_importances_` is biased toward high-cardinality and continuous features and is computed on training data, so it can reward a column that does nothing on unseen rows; `sklearn.inspection.permutation_importance` measures the score drop when a column is shuffled, runs on held-out data, and is the more trustworthy read.

One caution: a feature computed across rows leaks. A ratio of two columns in the same row uses nothing but that row, but a group statistic or target-encoded mean fitted on the full dataset lets held-out labels into training, which is [[Data Snooping Bias]] in a preprocessing costume.
