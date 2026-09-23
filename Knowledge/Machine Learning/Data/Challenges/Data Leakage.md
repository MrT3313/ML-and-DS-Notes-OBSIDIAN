---
note_kind: concept
aliases:
  - data leakage
  - label leakage
  - target leakage
  - feature leakage
  - leaky feature
  - leaky features
  - train-test contamination
  - test set leakage
  - group leakage
up: "[[Feature Engineering]]"
sources:
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

Data leakage is a form of the label reaching the set of features used to predict it, in a shape that will not exist at inference time. The fit then learns a relationship that is genuinely present in the file and genuinely absent in production, so the model scores beautifully on held-out data and fails on arrival. Nothing raises an error and nothing warns, and the leak is usually nonobvious enough that it is found by being surprised rather than by being looked for.

[[Data Snooping Bias]] is a different thing, and the reason to say so here is that neither one contains the other. Snooping is you conditioning your choices on the held-out sample, and choosing a model because it scored well on the [[Testing Set]] puts no label information into any feature at all. Leakage needs no analyst in the room: a marker of how sick the patient was is in the pixels of the scan whether or not anybody ever opens the test rows, which is the case worked through under cause 6 below. The two intersect in exactly two of the six causes, a scaler and an imputer fitted across the split, and those two are counted here, because what is contaminated in them is a fitted transformer rather than a decision anyone made.

## Formal statement

The precise version is an availability condition, and it is stated most carefully by Kaufman, Rosset and Perlich (KDD 2011, extended with Stitelman in *ACM TKDD* 6(4):15, 2012), whose vocabulary the rest of this section uses. Write $v \in \text{legit}\{u\}$ to mean that $v$ is observable for the purpose of inferring $u$. The axiom the whole subject rests on is that a target is never legitimate for itself,

$$y \notin \text{legit}\{y\}$$

and a model leaks when any of its observational inputs is illegitimate for the target it is being asked to predict. That splits into two conditions, because a model has two kinds of input.

- **No leaking feature.** Every feature is legitimate for the target process: $\forall x_j, \; x_j \in \text{legit}\{\mathcal{Y}\}$. This is the case where a column is computed from the answer, or from a proxy recorded only because the answer was known.
- **No leakage in the training examples.** Every feature vector and every label used for training is legitimate with respect to each evaluation target. This is the subtler condition, and it is the one that catches a split done wrongly, since each individual column can be clean while the *set of rows* the model was fitted on still carries information about the rows it will be scored on. Under independent and identically distributed data it reduces to the first condition; it does not reduce when the data is non-stationary.

The timing half is what they call the **no-time-machine requirement**. Let $t_x$ be the moment a feature instance is observed and $t_y$ the moment its related target is required,

$$\text{legit}\{\mathcal{Y}\} \subseteq \big\{\, x \in \mathcal{X} \;\big|\; t_x < t_y \,\big\}$$

an inclusion rather than an equality, since other legitimacy constraints may narrow the set further. Two variants are worth carrying. A buffer $\tau$, giving $t_x < t_y - \tau$, excludes a feature recorded so close to the event that it is really a trigger for it rather than a predictor of it. A memory limit, $t_y - \tau < t_x < t_y$, excludes information that is old enough that the deployed system will no longer hold it.

Stated as something a claim can be found false against: recompute column $j$ from the information available at $t_y$ alone, the way production would have to compute it, and compare the result against the column the training file carries. Agreement on every instance is the claim that the feature is legitimate; a single row of disagreement falsifies it. That check is what turns "this feature looks fine" into something that can be wrong.

What a leak does to the estimate follows from the condition rather than being a separate claim. For a leaky $x_j$ the joint distribution the fit was shown and the joint distribution the deployed model will meet are not the same object,

$$P_{\text{train}}(x_j, y) \ne P_{\text{deploy}}(x_j, y)$$

and the held-out split, drawn from the same contaminated file, inherits the first one. So the test score is not a biased estimate of [[Generalization]] error. It is an honest estimate of a quantity that has no counterpart in production, which is why no correction factor exists and why the size of the gap stays unknown until deployment supplies it.

The two statistical causes make the condition concrete enough to compute. Standardizing a column with

$$\mu_j = \frac{1}{m}\sum_{i=1}^{m} x_j^{(i)}, \qquad \sigma_j^2 = \frac{1}{m}\sum_{i=1}^{m}\big(x_j^{(i)} - \mu_j\big)^2$$

taken over all $m$ rows makes the transform a function of the held-out rows, so every training row's scaled value depends on data the deployed pipeline will never hold, and the fit can tune itself to the spread of the very rows it is about to be examined on. Fitting the same two constants over the $m_{\text{train}}$ rows of the training split alone satisfies the condition, and the held-out rows are then transformed by constants estimated without them, which is exactly the position production is in with a row it has never seen. The same sentence with $\mu_j$ replaced by a median and "scale" replaced by "fill" is the imputation case, and it is the same defect.

## Where it is used

Leakage is not confined to one step. It can be introduced while the data is generated, collected, sampled, split, processed, or while features are being built, which is why finding it is a matter of auditing a chain rather than checking a function.

### The six ways it gets in

1. **A time-correlated split drawn uniformly.** Data is often time-correlated, meaning the moment a row was generated shifts the distribution of its label. Predicting the seventh day of a price series is the clean shape: the honest split trains on days one to six and tests on day seven, and a uniform draw from [[Random Sampling]] scatters day-seven rows through the training set instead, handing the model the future it was asked to forecast. That is the no-time-machine requirement broken by the split rather than by any one column. The remedy is to split by time, which is a different procedure and not random sampling wearing a different hat.
2. **Scaling fitted before the split.** Global statistics computed over the entire file and then used to scale every split leak the mean and variance of the held-out rows into training, as the formulas above show. Split first, fit the scaler on the training split, and apply those constants everywhere: scikit-learn's own guidance is to split before any preprocessing step and never to call `fit` on test data. [[Feature Scaling]] is the family and [[Standardization]] is the member where the leak is easiest to see. Pushing the rule further back still, to before any exploratory work at all, is a habit with a different justification behind it rather than a second leakage result, and the argument for it is [[Data Snooping Bias]]'s.
3. **Imputing from statistics computed across the whole file.** Filling holes with a mean, median or mode computed over all rows is the scaling defect in a second costume, so the statistic comes from the training split alone. [[Missing Value Imputation]] carries the mechanics and already enforces it through the fit and transform split.
4. **Duplicates and near-duplicates left in before the split.** Copies that were never removed put the same instance on both sides of the boundary. They arrive from the collection itself, from merging sources that overlap, and from oversampling, so the check belongs upstream: [[Poor-Quality Data]] names them as a defect of the file, and [[Resampling]] carries the case where an oversampler manufactures them after the fact. The general statement is here and those two keep their instances.
5. **Group leakage.** A set of instances whose labels are strongly correlated is divided across splits, so a held-out instance is predicted by its own siblings. Two scans of one patient taken a fortnight apart carry nearly the same evidence, and landing one in training and one in test measures recall of a specific patient rather than [[Generalization]]. The remedy is to split on the group rather than on the row, which is what scikit-learn's `GroupKFold` and `StratifiedGroupKFold` guarantee, by requiring that every sample in a validation fold comes from a group absent from the paired training fold. [[Data Augmentation]] holds the sharpest instance in the vault, where the copies descended from one original are not independent draws; the general statement is here and that note keeps its case.
6. **The data generation process itself.** The leak is in how the data came to exist, so catching it needs an understanding of who collected what, when, and under what circumstances, rather than any inspection of the columns. The standing case is Zech, Badgeley, Liu, Costa, Titano and Oermann (*PLOS Medicine* 15(11): e1002683, 2018), who trained pneumonia detectors on chest radiographs from three hospital systems and then scored them across systems. Pneumonia prevalence was $34.2\%$ at one site against $1.2\%$ and $1.0\%$ at the other two, so sorting the joint set by hospital system alone reaches $0.861$ area under the ROC curve, and a network trained to name the hospital from the image alone gets it right on $99.95\%$ of one site's radiographs and identifies the department inside a site perfectly. What it reads is acquisition machinery: a metal token the technician places in the corner of the field of view, laterality labels, and the inverted colour scheme and burned-in text that mark a portable scan. The portable scan is ordered for a patient too unstable to travel to radiology, and portable inpatient radiographs carried $41.1\%$ pneumonia against $32.8\%$ in the emergency department, so the artifact is a proxy for the prevalence at the place it was taken. The joint model scored $0.931$ internally and $0.815$ on the third site.

   Normalizing per source, so that every source arrives with comparable means and variances, is the remedy usually reached for and it is a partial one. It equalizes the marginal distribution of the features, which is the covariate shift case in Moreno-Torres, Raeder, Alaiz-Rodríguez, Chawla and Herrera's terms (*Pattern Recognition* 45(1), 2012), and it touches neither a difference in the class prior $P(y)$ across sources nor a difference in $P(y \mid \mathbf{x})$. The radiograph case is the first of those, a prior that differs tenfold between sites, plus a feature the network can read the site off. A per-column mean and variance does nothing to a metal token in the corner of an image. Treating normalization as the remedy is the overstatement; treating it as the part of the problem that is cheap to remove is right.

### Finding one

Detection is a set of habits rather than a test, because the condition above is about provenance and provenance is not in the array. Kaufman, Rosset and Perlich group the habits into three, and all three need domain knowledge to be worth anything.

- **Look at the data before modelling it.** Measure each feature's predictive power against the label and treat an unusually high [[Correlation]] as a question rather than a result: find out how that column is generated and whether a relationship that strong is something the world could actually supply. The single-feature sweep is incomplete by construction, since two features can be innocent apart and leak together, so a suspiciously strong pair is worth the same question.
- **Read the fitted model after the fact.** A feature behaving surprisingly inside the fit, carrying far more weight than its meaning justifies, is the second signal, and a model whose overall score is better than the problem should allow is the third. [[Feature Importance]] owns the removal experiment that measures the first of those: take a feature or a set of them out, refit, and see how far the score falls. A newly added column that improves the model sharply is either very good or leaking, and the score alone cannot tell those apart; the check is the recomputation in the formal statement, run on that column.
- **Field-test early.** A gap between the estimated out-of-sample score and the realised one on live traffic is what leakage looks like from outside, and it is the only measurement that is not taken inside the contaminated file.

Finding a leak is easier than removing one. Kaufman, Rosset and Perlich's own position is that fully repairing a leak after the fact is usually very hard and sometimes impossible, and that the durable fix is structural: tag every observation with what it is legitimate for, then keep a strict separation between what may be learned from and what is being predicted. In this vault's tooling that separation is a [[Pipeline]], which fits every transformer inside the training fold and replays it on the held-out fold, so a mean, a median or a category vocabulary cannot be estimated across the boundary even by accident, and the split is the first thing that happens to the [[Training Set]] rather than a step someone remembers. The size of what that buys is worth one number: scikit-learn's own demonstration selects 25 of 10,000 pure-noise features on 200 instances with randomly assigned labels, and reports $0.76$ accuracy when the selection runs before the split against $0.50$, which is chance, when it runs after.
