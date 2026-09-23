---
note_kind: method
aliases:
  - behavioral testing
  - behavioural testing
  - behavioral test
  - behavioural test
  - invariance test
  - invariance tests
  - directional expectation test
  - directional expectation tests
  - perturbation test
  - perturbation tests
  - minimum functionality test
  - MFT
  - CheckList
up: "[[Testing Set]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

A behavioural test constructs an input, or modifies one it already has, and asserts a property of the prediction. It does not draw a sample and report a mean, and that is what separates it from every [[Performance Measure]] in this vault: the reading is a pass or a fail on each case, aggregated as a failure rate over the suite, and that rate is not a number you can rank candidate models by, because its denominator is whatever you chose to write rather than a draw from the deployment distribution.

Reach for it once the aggregate score is acceptable and the remaining question is whether the model is right for reasons that will survive inputs slightly unlike the ones in the file. A [[Testing Set]] answers how often the model is right on data like this, and answers it as an estimator, with an interval. A behavioural test answers whether the model does one specific thing you can name, and answers it yes or no. The second question is the one a good score cannot settle, because a model can reach any accuracy you like by exploiting a regularity of the file that nobody intends to ship.

The suite is also a regression harness, which is most of its practical value. Once a test is written it runs again on every later candidate, so a failure that reappears is a regression against a fixed expectation rather than a fresh discovery, and this is the one form of model evaluation that behaves like the test suite of an ordinary program.

**The family is one thing rather than a list of tricks.** Three of the four types below are the test types of Ribeiro, Wu, Guestrin and Singh's CheckList (ACL 2020), which organizes them as a matrix: capabilities of the model down the side, test types across the top, one cell per pair. The shared shape is what makes them a family. Each type fixes a way of manufacturing inputs and a property to assert about the prediction, and none of them computes a score to compare against a rival's score.

| test type | what is constructed | the assertion | needs a label | the reading |
|---|---|---|---|---|
| minimum functionality (MFT) | simple inputs whose right answer is known because you built them that way | $h(\mathbf{x}) = y$ | yes, by construction | failure rate |
| invariance (INV) | $\tau(\mathbf{x})$, for a $\tau$ that must leave the true label alone | $h(\tau(\mathbf{x})) = h(\mathbf{x})$ | no | failure rate |
| directional expectation (DIR) | $\tau(\mathbf{x})$, for a $\tau$ whose effect on the true label has a known sign | $h(\tau(\mathbf{x})) \le h(\mathbf{x})$, or $h(\tau(\mathbf{x})) = y^{*}$ | no | failure rate |
| perturbation, or robustness | a noised copy of the whole evaluation split | none per instance | yes | the fall in the score |

Where the output is continuous rather than a label, both perturbation-based assertions take a tolerance $\eta$ and become $|h(\tau(\mathbf{x})) - h(\mathbf{x})| \le \eta$ and $h(\tau(\mathbf{x})) \le h(\mathbf{x}) + \eta$. Choosing $\eta$ is a modelling decision and not a detail: it is the size of a prediction change you are prepared to call no change.

The fourth row is not one of CheckList's types. [[DMLS Ch06 Model Development and Offline Evaluation|Huyen]] lists the perturbation test beside the other two perturbation-based ones as a separate evaluation method, and it differs from them in kind, because its reading is a number rather than a verdict. In the CheckList matrix the same concern appears as a capability, robustness, tested with an invariance test over typos rather than with an aggregate rescoring.

The operation these tests run on an instance is the same one [[Perturbation]] and [[Data Augmentation]] run, and the split between those notes and this one is what becomes of the modified instance. **There the perturbed instance is manufactured to be trained on**, carrying a label copied from its original. **Here the perturbed instance, or the perturbed split, is scored**, and in the two perturbation-based test types it carries no label at all, because what is asserted is a relation between two predictions rather than agreement with a target.

## Algorithm

1. **Name the capability under test.** One behaviour the model is supposed to have, stated so that a failure is recognisable: handles negation, ignores which city was named, does not become more positive when insulted. The suite is organized by capability, not by the code that generates the cases. Where the capability comes from is usually [[Error Analysis]] on a real split, since a pattern among the mistakes is what names a behaviour worth asserting in the first place.
2. **Pick the test type for it.** Whether the correct answer can be constructed decides between the minimum functionality test and the two perturbation-based types, and whether the transformation's effect on the label is zero or signed decides between invariance and directional expectation.
3. **Generate the cases.** From scratch through a template plus a lexicon, which gives control and a known label, or by transforming rows of an existing set, which gives volume and needs no label. The CheckList authors add a third route, filling template slots with the suggestions of a masked language model, plus a library of general-purpose transformations for typos, name swaps and location swaps.
4. **Write the assertion as code**, with a tolerance $\eta$ where the output is continuous, and with the direction fixed where the type is directional. This is the step that has to be reviewed by somebody who knows the domain, because an assertion that is not actually implied by the task will fail on correct models.
5. **Run it and compute the failure rate.** With a suite $T$ of cases and $A_t$ the assertion attached to case $t$,

   $$\text{failure rate} = \frac{1}{|T|} \sum_{t=1}^{|T|} \mathbb{1}\big[\neg A_t\big]$$

   reported per capability and per test type, never pooled into one number across the suite, because the mix of cases is something you chose.
6. **Triage by severity rather than by rate.** A capability failing on $5$ percent of cases can matter more than one failing on $70$ percent, depending on what the prediction is used for, so each failing test carries a severity judgement and the queue is ordered by that.
7. **Fix, then re-run the whole suite.** Keep the suite in version control beside the model code and run it on every candidate. Resist the shortcut of adding the failing cases to the training set: it makes the suite green, it does not supply the capability, and the suite has stopped being a test the moment the model is fitted on it.

### Invariance tests, and what they are not

An invariance test fixes an instance, applies a transformation that must leave the true label alone, and requires the prediction not to move:

$$h(\tau(\mathbf{x})) = h(\mathbf{x}) \qquad \text{for every } \mathbf{x} \text{ in the suite, where } y(\tau(\mathbf{x})) = y(\mathbf{x})$$

The objection to raise here is that this cannot be a reasonable demand, since if the transformation takes information out of the input then it has taken away something the model needed, and a prediction that refuses to move is a model ignoring a feature that matters. The objection dissolves once the operation is read exactly, and the answer is a distinction rather than a yes or a no.

**An invariance test changes an input while holding the others fixed. It does not remove one.** $\tau$ maps an instance to another instance of the same shape, every coordinate still present, and the assertion is about which changes the prediction is required to be insensitive to. That is a claim about the behaviour of the function, not about how much information the function has to work with.

Removing a column and measuring what the fit loses is a different operation with a different name, a different formula and a different answer. It is ablation, it belongs to [[Feature Importance]], and it is written as a difference of two risks, $I_j = \mathcal{L}(h_{-j}) - \mathcal{L}(h)$, where $h_{-j}$ is refitted without the column. A large $I_j$ is a finding about how much the problem needs feature $j$; an invariance failure is a finding about what the fitted function is sensitive to. Neither converts into the other, and a model can be perfectly invariant to swapping one person's name for another while depending heavily on every column it holds.

The transformations that make this concrete are the ones the CheckList authors ran against a commercial sentiment model. Changing a location name inside a tweet, Chicago to Dallas, must not change the predicted sentiment, because sentiment towards an airline does not depend on the destination city; that test failed on $20.8$ percent of its cases. Replacing a neutral word with another neutral word, swapping two adjacent characters to make a typo, and substituting one person's name for another are the same shape, and all of them are substitutions.

One case does genuinely delete material and is still not an ablation: adding or removing a URL or an @-handle in a tweet. What is deleted there is content the label is independent of, not a column of the design matrix. Every instance still carries every feature; one of them holds different text. The assertion is unchanged, that the prediction be insensitive to content the label does not depend on.

### Directional expectation tests

A directional expectation test also changes an input, and differs from the invariance test only in that the true label's response has a known sign instead of being zero. With the convention that the score should not rise,

$$h(\tau(\mathbf{x})) \le h(\mathbf{x}) + \eta$$

and the other form of the assertion names a target label outright, $h(\tau(\mathbf{x})) = y^{*}$, where the transformation is strong enough to fix what the answer must become. Appending "You are lame." to a tweet cannot make its sentiment more positive, so the assertion is that the predicted sentiment does not rise; that test failed on $34.6$ percent of its cases against the same commercial model. The targeted form appears in the CheckList authors' paraphrase tests, where replacing the country named in only one of two questions guarantees that the pair is no longer a duplicate.

Worth being exact about the operation, because it attracts the same confusion the invariance test attracts. The input is **changed in a known direction**, not removed. Deleting an input and watching whether the output moves the wrong way is ablation again, and ablation carries no expected direction at all: taking information away from a model can move a prediction either way, and there is no sign to assert. The directional test needs a $\tau$ whose effect on the true label can be signed, which is why every instance of it is an addition or a substitution rather than a deletion.

### Minimum functionality tests

The type with no transformation in it. A minimum functionality test is a collection of deliberately easy inputs whose correct answer is known because you constructed them to have it, checked with the plainest assertion there is, $h(\mathbf{x}) = y$. The analogy the CheckList authors draw is the unit test, and its purpose is the same: to catch a model that handles complex inputs by shortcut without holding the capability the inputs appear to require. A model that scores well on a benchmark can still fail a page of trivially negative sentences, and only a test built to be trivial detects that.

The evidence for bothering is the strongest single number in that paper. Negation tested from the template `I {NEGATION} {POS_VERB} the {THING}.`, filled from prebuilt lexicons, failed on $76.4$ percent of its cases against a commercial sentiment model, a higher failure rate than either perturbation-based type produced on the same model. Nothing was perturbed to get that; the cases were simply written.

The price is that the label must be known by construction. Templated text supplies it free, since a sentence assembled from a negation and a positive verb is negative whatever else is in it, which is a large part of why behavioural testing was formalised on language tasks. A table of numeric columns usually does not supply it, and the substitute is a weaker assertion, a bound the prediction must stay inside rather than a label it must equal, as in the implementation below.

### Perturbation tests

The aggregate cousin. Noise the evaluation split, rescore, and read the fall:

$$\Delta = \hat{P}\big(h, D_{\text{test}}\big) - \hat{P}\big(h, \tau(D_{\text{test}})\big)$$

with $\tau$ applied row by row at a noise level chosen to resemble the mess real inputs arrive with. The larger $\Delta$ is, the more sensitive the model is to noise, and the reason to care is a maintenance cost rather than an accuracy cost: a model whose score moves a long way under small input noise will move again every time an upstream sensor, a scraper or a join changes slightly, so the deployment needs watching in proportion to $\Delta$.

**$\Delta$ and an invariance failure rate are not the same measurement, and the gap between them is arithmetic.** Write $a$ for accuracy on the clean split, $a'$ for accuracy on the perturbed one, $u$ for the share of instances the perturbation breaks, right becoming wrong, and $v$ for the share it accidentally repairs, wrong becoming right. Then

$$\Delta = a - a' = u - v$$

while the share of predictions that moved at all, which is what the invariance assertion counts, is at least $u + v$, and exceeds it whenever a wrong label is replaced by a different wrong label. So $\Delta = 0$ is consistent with every prediction in the split having changed, provided the breakages and the repairs cancelled. A flat perturbation test has not shown that the model is invariant. It has shown that two error flows were the same size.

## Hyperparameters

None. The knobs a suite exposes belong to its content rather than to the protocol: the number of cases generated per template sets only the precision of the failure rate, since a rate $\hat{p}$ read off $N$ cases carries a standard error of $\sqrt{\hat{p}(1 - \hat{p})/N}$, and the severity attached to a failing test decides which failures are escalated rather than which tests fail.

## Failure modes

- **Reading the failure rate as a performance measure.** Its denominator was written, not sampled, so $20$ percent on a suite of $500$ templated cases estimates nothing about production, and generating ten times as many cases from the template you care most about moves the rate without changing a single fact about the model. Two candidates can only be compared on identical suites, and even then the comparison is per capability rather than on one pooled figure.
- **An invariance test whose transformation is not actually label preserving.** If $\tau$ moves the true label, the assertion demands a wrong answer, so every pass is a defect recorded as a success. This is the same condition [[Data Augmentation]] states for its transformations, and it fails the same way: rotate a $6$ far enough and it is a $9$.
- **Testing a deletion and calling the result invariance.** Removing an input and declaring the prediction's movement a failure measures what [[Feature Importance]] measures, and licenses no conclusion about invariance at all. The reverse error is quieter: deleting an input, seeing the prediction hold still, and concluding the model is robust, when what has been shown is that the column was not carrying much.
- **Patching the suite instead of the model.** Adding the failing cases to the training data turns the test set into training data, and the capability can still be absent, since a template is a narrow slice of the behaviour it was meant to stand for. Keep the generators, regenerate the cases with fresh lexicon fills after the fix, and see whether the rate holds.
- **Concluding robustness from a flat perturbation test.** $\Delta = u - v$, so the breakages and the accidental repairs cancel in the reading, and a split whose every prediction changed can score $\Delta = 0$. Counting the predictions that moved is a different measurement and has to be taken separately.
- **Asserting a direction the task does not imply.** A directional test is only as good as the claim that the true label responds with that sign, and a plausible-sounding direction that does not actually hold turns a correct model into a failing one, which then gets "fixed".

## Implementation

**scikit-learn has no behavioural-testing API.** Nothing in 1.6 takes a transformation and an assertion and hands back a failure rate, and nothing generates test cases. The nearest-named thing in the library, `sklearn.utils.estimator_checks.check_estimator`, tests that an estimator *class* honours the [[Scikit-Learn Estimator API]] and says nothing whatever about what a fitted model predicts. This costs little, because an assertion is a comparison between two arrays and the harness is a few lines.

scikit-learn 1.6, NumPy 2.x and pandas 2.x, on [[California Housing]] with a gradient-boosted regressor, so the outputs are continuous and every assertion carries a tolerance:

```python
import numpy as np
from sklearn.datasets import fetch_california_housing
from sklearn.ensemble import HistGradientBoostingRegressor
from sklearn.metrics import root_mean_squared_error
from sklearn.model_selection import train_test_split

X, y = fetch_california_housing(as_frame=True, return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
model = HistGradientBoostingRegressor(random_state=42).fit(X_train, y_train)

rng = np.random.default_rng(42)
base = model.predict(X_test)
tolerance = 0.01                     # in units of $100,000, the target's units

def failure_rate(passed):
    return 1.0 - np.mean(passed)

# INV: moving a district about ten metres must not move its predicted value
X_inv = X_test.copy()
X_inv["Latitude"] += rng.normal(0.0, 1e-4, len(X_inv))
X_inv["Longitude"] += rng.normal(0.0, 1e-4, len(X_inv))
inv_passed = np.abs(model.predict(X_inv) - base) <= tolerance
print("INV failure rate", failure_rate(inv_passed))

# DIR: one extra standard deviation of median income must not lower the prediction
X_dir = X_test.copy()
X_dir["MedInc"] += X_train["MedInc"].std()
dir_passed = model.predict(X_dir) >= base - tolerance
print("DIR failure rate", failure_rate(dir_passed))

# MFT: a constructed district at the top of the income range must not be
# predicted into the bottom decile of the target. The expected answer here
# comes from domain knowledge, not from any row of the file.
rich = X_train.median().to_frame().T
rich["MedInc"] = X_train["MedInc"].quantile(0.99)
mft_passed = model.predict(rich) >= np.quantile(y_train, 0.10)
print("MFT failure rate", failure_rate(mft_passed))

# perturbation test: rescore the whole split under one percent column noise
X_noisy = X_test + rng.normal(0.0, 0.01 * X_train.std().to_numpy(), X_test.shape)
clean = root_mean_squared_error(y_test, base)
noisy = root_mean_squared_error(y_test, model.predict(X_noisy))
print("RMSE", round(clean, 4), "->", round(noisy, 4), "delta", round(noisy - clean, 4))
```

Four things in that code are the whole method and everything else is scaffolding: the transformation, the tolerance, the comparison, and the mean of the booleans. Note that the three per-instance tests never touch `y_test`, which is the point of the two perturbation-based types, and that the perturbation test at the end is the only one that needs it, because its reading is a score rather than a relation between predictions.

What implements this at scale, and none of it is in scikit-learn. **`checklist`** (Ribeiro, Wu, Guestrin and Singh, ACL 2020; `github.com/marcotcr/checklist`) carries the templates, the lexicons, the masked-language-model fills, the general-purpose transformations and the three test classes as objects, plus a visual summary of the matrix. **Robustness Gym** (Goel, Rajani, Vig, Taschdjian, Bansal and Ré, NAACL 2021 demonstrations) puts four evaluation paradigms behind one interface, subpopulations, transformations, evaluation sets and adversarial attacks, which is the harness in which behavioural tests and [[Slice-Based Evaluation]] are the same kind of object. Both are shaped for language tasks. For a tabular model there is no package worth adding: the harness above plus whatever test runner the project already uses is the implementation, and keeping the assertions in the repository's own test suite is what makes them run on every candidate.
