---
note_kind: method
aliases:
  - weak supervision
  - weakly supervised
  - programmatic labeling
  - programmatic labelling
  - labeling function
  - labeling functions
  - LF
  - LFs
  - data programming
  - Snorkel
up: "[[Training Set]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

People already label by heuristic. An expert asked why a discharge summary counts as a case of pneumonia will, pressed hard enough, name a rule: the word appears near a radiology finding, the chart carries a matching billing code, a lookup in the drug table returns an antibiotic. Weak supervision takes that rule seriously enough to write it down as a program, and then runs the program over every instance instead of running the expert over every instance. The unit of expert effort stops being one label and becomes one heuristic, which is the whole of the idea: a heuristic is reusable and a label is not.

The nearest neighbour is [[Self-Supervised Learning]], and the two are genuinely different things that both end with labels nobody wrote by hand. Self-supervision derives the label from the instance itself, by withholding a part of the instance and asking the model to recover it, so the label is already inside the data and the construction is mechanical. Weak supervision derives the label from a heuristic a person wrote *about* the instance, so the label comes from outside the data and carries whatever the person believed. The first needs no domain knowledge and the second is nothing but domain knowledge.

Reach for it when unlabelled instances are abundant, the label definition is something an expert can articulate as rules, and the binding cost is annotation. Four things change when labels come from functions rather than from people, and they are the argument for the method:

| hand labelling | programmatic labelling |
|---|---|
| cost is paid per instance, and paid again at expert rates when the judgement needs a specialist | cost is paid per heuristic, and the heuristic is a file: it can be versioned, reviewed, shared and reused by anyone else in the organisation |
| the data has to reach the annotators, so every instance is seen by a person | the functions can be written against a cleared subsample and then applied to the rest, so the bulk of the data is labelled without anyone reading an individual record |
| time scales linearly in the number of labels wanted | time is spent once on the functions, and applying them to a thousand instances or a million costs essentially the same |
| a change to the label definition means relabelling from scratch | a change means editing the functions and reapplying them, which is minutes |

The left column is [[Hand Labeling]]'s own account of its difficulties and belongs to that note; what matters here is that each difficulty is structural rather than a matter of working harder, which is why the right column is a different method and not a better-run version of the same one. The sibling remedies for the same shortage are [[Semi-Supervised Learning]], which propagates a small set of real labels through structural assumptions about the unlabelled data, and [[Data Augmentation]], which manufactures instances rather than labels.

## Algorithm

Write the unlabelled instances as $S$, the labelling functions as $\lambda_1, \dots, \lambda_m$, and $\lambda(x) = \Lambda \in \{-1, 0, 1\}^{m}$ for the vector of votes on one instance in the binary case, where $0$ means the function abstained. Stacking those vectors over $S$ gives the label matrix.

1. **Fix the label space and hand label a small development set.** A few hundred instances, labelled properly, held out and never trained on. Weak supervision does not require them, and without them nothing in the rest of this protocol tells you whether a function is accurate or merely confident.
2. **Write the labelling functions.** Each encodes one heuristic and nothing else, so that it can be judged, kept or dropped on its own. Four kinds cover most of what gets written: a keyword match on the text, a regular expression, a lookup in an existing database or ontology, and the output of some other model that already exists. A function returns a class or it abstains.
3. **Apply every function to every instance**, producing the label matrix. Abstention is part of the interface, not a failure: a heuristic that fires on two percent of the data and is right when it fires is a good function, and forcing it to guess on the other ninety-eight percent would destroy exactly the property that makes it useful. What the label model estimates is precisely the distinction between a function that is silent and a function that is wrong.
4. **Fit the label model on the label matrix alone**, with no access to any true label, obtaining an estimated accuracy and coverage per function.
5. **Emit probabilistic labels.** Each instance gets $P(y \mid \Lambda)$ rather than a class. Instances on which every function abstained have no information in them and are dropped at this point rather than silently assigned a majority class.
6. **Train the end model on the probabilistic labels** with a noise-aware loss, so that an instance the functions half agree on contributes half as much pull as one they all agree on.
7. **Score against the development set from step 1, and iterate on the functions.** The loop that produces a usable training set is written function, measure, rewrite, not written once.

### Denoising without ground truth

The labels are noisy because the heuristics are noisy, and the step that makes the method work rather than merely cheap is estimating how noisy each one is. "Noisy" here means a label generated approximately, by a rule that is right most of the time and wrong the rest of it, which is a different thing from [[Poor-Quality Data]]'s sense of a record that arrived wrong: there the corruption is in $\mathbf{x}$ and predates the modelling, here the corruption is in $y$ and was manufactured deliberately with a known mechanism.

Data programming (Ratner, De Sa, Wu, Selsam and Ré 2016) writes that mechanism down as a generative model over the true label $Y \in \{-1, 1\}$ and the observed votes $\Lambda$, with a per-function accuracy $\alpha_i$ and a per-function coverage $\beta_i$, the probability that function $i$ votes at all:

$$\mu_{\alpha,\beta}(\Lambda, Y) = \frac{1}{2}\prod_{i=1}^{m}\Big(\beta_i \alpha_i \mathbb{1}\{\Lambda_i = Y\} + \beta_i (1 - \alpha_i)\mathbb{1}\{\Lambda_i = -Y\} + (1 - \beta_i)\mathbb{1}\{\Lambda_i = 0\}\Big)$$

The parameters are fitted by maximum likelihood on the votes with $Y$ marginalised out, which is what makes the whole thing possible with no labels anywhere in it:

$$(\hat\alpha, \hat\beta) = \arg\max_{\alpha, \beta} \sum_{x \in S} \log \sum_{y' \in \{-1, 1\}} \mu_{\alpha,\beta}\big(\lambda(x), y'\big)$$

Read the inner sum and the source of the signal is clear: the only observable is the pattern of agreement and disagreement among the functions. A function that agrees with the others wherever they both vote is pushed toward a high $\hat\alpha_i$; one that disagrees with the consensus is pushed toward a low one and down-weighted accordingly. Nothing checks any function against truth, and nothing has to. Correlations between functions enter the same model as dependencies among the $\Lambda_i$, which is why they have to be declared or learned rather than assumed away (see the failure modes).

The end model is then fitted to the posterior $P(y \mid \Lambda)$ rather than to a hardened class, by minimising the noise-aware empirical risk. For a linear model over features $f(x)$ with logistic loss and regularisation $\rho$:

$$\hat{w} = \arg\min_{w} \frac{1}{|S|}\sum_{x \in S} \mathbb{E}_{(\Lambda, Y) \sim \mu_{\hat\alpha,\hat\beta}}\Big[\log\big(1 + e^{-w^{T}f(x)Y}\big) \,\Big|\, \Lambda = \lambda(x)\Big] + \rho \lVert w \rVert^{2}$$

The expectation is the entire difference from an ordinary fit. Take the $\arg\max$ of the posterior instead and every instance pulls with full weight whether the functions were unanimous or split down the middle, which throws away the one thing the label model was fitted to produce.

### What the sample complexity result says

The result that makes the approach non-obvious is about rates. Under the conditions Ratner et al. state, chiefly that the family contains the true vote distribution and that the true label is independent of the features given the votes, reaching expected loss within $\epsilon$ of the best in the model class needs $m = O(1)$ labelling functions and $|S| = \tilde{O}(\epsilon^{-2})$ **unlabelled** instances. That is the same asymptotic scaling a supervised method gets from $\tilde{O}(\epsilon^{-2})$ **labelled** instances.

Two readings, and the second is the one usually got wrong. The substitution being offered is unlabelled data for labelled data at the same rate, with the number of functions held constant: you are not paying $\tilde{O}(\epsilon^{-2})$ in expert effort, you are paying $O(1)$ in expert effort and $\tilde{O}(\epsilon^{-2})$ in a resource that is nearly free. And the guarantee is a rate, not a level. The floor term does not vanish: if the functions are collectively wrong about a region, no quantity of unlabelled data recovers the truth there, because there is nothing in $S$ that knows it.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| number of labelling functions | $m$ | none, you write them | more votes per instance, so coverage rises and the accuracy estimates get sharper; past the point where new functions restate old ones it adds correlation rather than evidence | add functions until coverage over the pool stops rising, and judge each addition on the development set rather than on the count. The theory asks for $m = O(1)$, not for $m$ large |
| coverage of a function, complement of its abstention rate | $\beta_i$ | whatever the heuristic happens to fire on | the function votes on more instances, raising overall coverage and lowering the fraction dropped in step 5; a function widened past the region its heuristic is actually right about loses accuracy as it gains coverage | widen only while empirical accuracy on the development set holds. Two narrow accurate functions beat one broad mediocre one, because the label model can tell them apart and cannot rescue the latter |
| class balance prior | $P(y)$ | estimated from the votes, or passed in | shifting mass toward a class shifts every probabilistic label toward it, and the end model's decision threshold moves with it | pass the real prior when you know it from the domain or can estimate it from the development set. Leaving it to be inferred from the votes makes it a function of what the functions happened to cover, which is not the same quantity |
| decision threshold on the probabilistic labels | $t$ | none, the probabilities are used as they are | hardening at a higher $t$ keeps only confidently labelled instances, so the training set shrinks and gets cleaner | do not harden at all unless the end model cannot consume soft targets. If it must be hardened, pick $t$ on the development set against the operating measure, the same sweep [[Precision-Recall Tradeoff]] describes |
| dependency structure among functions | | independent, unless declared | declaring a dependency stops the label model from reading two copies of one heuristic as two independent confirmations; declaring dependencies that are not there costs parameters and sharpness | declare the ones you know from how the functions were written, ex two regular expressions over the same field, and check the rest against the empirical conflict and overlap rates |

Pinned rather than tuned: the label model's optimiser settings and its random seed, which move the fitted accuracies a little and are fixed once ([[Random Seed]]), and the label space itself, which is a problem definition rather than a knob.

## Failure modes

- **Correlated labelling functions.** Three functions that are three spellings of one heuristic agree on everything, and the label model, whose only evidence is agreement, reads that as three independent confirmations. Their estimated accuracies all go up, their shared blind spot is now backed by three votes instead of one, and the probabilistic labels are confidently wrong exactly where the heuristic fails. This is the failure the method is most prone to and the least visible, because the diagnostic that would catch it, disagreement, is the thing that has been removed.
- **A function that is systematically wrong rather than noisily wrong.** Denoising assumes errors that scatter. A function that is always wrong on one segment, ex a keyword that means the opposite thing in one customer's documents, has an error that is a function of the instance, and no estimate of a scalar $\hat\alpha_i$ can represent that. The label model averages the segment's failures into the function's overall accuracy and reports a perfectly reasonable number.
- **Writing the functions by looking at the instances they will label.** Developing heuristics against the pool, or worse against the development set, tunes them to that particular data and the measured accuracies stop predicting anything. It is [[Data Snooping Bias]] arriving through a side door: the split was respected, the functions were not.
- **Coverage gaps.** If no function fires on a whole region of the input space, every instance there abstains out and is dropped at step 5. The training set is now missing a region, the end model has never seen it, and nothing in the pipeline raises an error, because dropping abstentions is the correct behaviour instance by instance. Coverage has to be measured per class and per segment, not in aggregate.
- **Shipping with no hand labels at all.** The method genuinely does not require them, and a pipeline built without any has no measurement of whether its functions are accurate, no way to compare two versions of a function, and no development set on which to set the class balance or a threshold. A few hundred honest labels is the cheapest instrument available and is the difference between iterating and guessing.
- **Hardening the probabilistic labels immediately.** Taking $\arg\max$ of $P(y \mid \Lambda)$ before training discards the uncertainty the label model exists to estimate, and the noise-aware loss above degenerates into an ordinary fit on labels that are now simply wrong some fraction of the time.

## Implementation

**scikit-learn 1.6 has nothing for this.** There is no labelling function abstraction, no label model, and no loss that consumes a posterior over classes as a target: `sklearn.semi_supervised` holds `SelfTrainingClassifier`, `LabelPropagation` and `LabelSpreading`, which propagate a small set of real labels through structural assumptions about the unlabelled data and never generate a label from a heuristic, which is [[Semi-Supervised Learning]] and a different method. The noise-aware objective could be fitted by passing the posterior as `sample_weight` over duplicated rows, one copy per class, but that is a construction you write yourself and not an API.

The labelling function abstraction, the label model and the denoising step come from Snorkel, the system built around data programming at Stanford: Ratner, De Sa, Wu, Selsam and Ré, "Data Programming: Creating Large Training Sets, Quickly" (NeurIPS 2016) is the paradigm and the theory, and Ratner, Bach, Ehrenberg, Fries, Wu and Ré, "Snorkel: Rapid Training Data Creation with Weak Supervision" (VLDB 11(3), 2017) is the end-to-end system and the interface most later tools copied. Ratner et al., "An Overview of Weak Supervision" (2018) is the short version.

Status of the open source library, checked on 22 September 2026: `snorkel-team/snorkel` on GitHub is not archived, but its last release is v0.10.0 of 27 February 2024 and its last substantive commit is from the same week, with one README link update in April 2026. The project README states that the team's effort has moved to Snorkel Flow, a commercial platform. Read that as a stable and unmaintained reference implementation rather than as an actively developed library, and check the date on this paragraph before relying on it.

snorkel 0.10.0, which requires Python 3.11 or later. Note the library's label convention, which is not the papers': classes are $0, \dots, k-1$ and abstention is $-1$, where the binary model in the mathematics above uses $\{-1, +1\}$ for the classes and $0$ for abstention.

```python
import re

from snorkel.labeling import labeling_function, PandasLFApplier, LFAnalysis
from snorkel.labeling.model import LabelModel

ABSTAIN, NEGATIVE, POSITIVE = -1, 0, 1

@labeling_function()
def lf_keyword(x):                                  # keyword heuristic
    return POSITIVE if "refund" in x.text.lower() else ABSTAIN

@labeling_function()
def lf_regex(x):                                    # regular expression
    return POSITIVE if re.search(r"\border(ed)?\b.*\bnever\b", x.text, re.I) else ABSTAIN

@labeling_function()
def lf_lookup(x):                                   # database lookup
    return NEGATIVE if x.sender_domain in known_good_domains else ABSTAIN

lfs = [lf_keyword, lf_regex, lf_lookup]
L_train = PandasLFApplier(lfs=lfs).apply(df=df_train)     # (n_instances, n_lfs)

label_model = LabelModel(cardinality=2)
label_model.fit(L_train=L_train, class_balance=[0.7, 0.3], n_epochs=500, seed=42)
probs_train = label_model.predict_proba(L_train)          # (n_instances, 2)
```

`LabelModel.fit(L_train, Y_dev=None, class_balance=None, **kwargs)` is where the class balance prior of the table enters; passing `Y_dev` instead lets it be estimated from the development set. `LFAnalysis(L=L_train, lfs=lfs).lf_summary(Y=Y_dev)` is the instrument the failure modes above ask for, reporting per function its coverage, its overlaps and conflicts with the others, and, when a development label vector is supplied, its empirical accuracy. `filter_unlabeled_dataframe` drops the instances every function abstained on, which is step 5.
