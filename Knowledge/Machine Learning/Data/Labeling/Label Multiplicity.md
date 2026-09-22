---
note_kind: concept
aliases:
  - label multiplicity
  - label ambiguity
  - conflicting labels
  - annotator disagreement
  - inter-annotator agreement
  - interannotator agreement
  - inter-rater reliability
  - Cohen's kappa
  - Krippendorff's alpha
up: "[[Hand Labeling]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## Definition

Several people label the same instance and return different answers, so the instance carries more than one candidate label and something has to decide which one the training set records. It goes under two names, label multiplicity and label ambiguity, and the conflict is in the labels rather than in the data: the instance is one instance and has not changed, it is the readings of it that differ.

It is what scale produces rather than an accident of sloppiness. Getting enough labelled data means drawing on several sources and several annotators, who differ in expertise and in accuracy, and any two such readers applied to the same instance will eventually disagree. A project that never sees multiplicity has usually not looked, because it set the replication in [[Hand Labeling]] to one.

## Formal statement

Two annotators label the same $n$ instances into $K$ classes. Let $p_o$ be the observed agreement, the fraction of instances on which they assigned the same class, and let $p_e$ be the agreement expected by chance from their two marginal distributions,

$$p_e = \sum_{k=1}^{K} p_{1k} \, p_{2k}$$

where $p_{jk}$ is the fraction of instances annotator $j$ assigned to class $k$. Cohen's kappa is the observed agreement with that chance level subtracted out and rescaled to the room left above it (Cohen, "A Coefficient of Agreement for Nominal Scales", *Educational and Psychological Measurement*, 1960):

$$\kappa = \frac{p_o - p_e}{1 - p_e}$$

$\kappa = 1$ is total agreement, $\kappa = 0$ is agreement no better than two people answering independently with those marginals, and $\kappa < 0$ is agreement worse than chance, which happens and is informative when it does. The upper bound is $\kappa \le 1$ and there is no useful lower bound to quote, since how negative it can go depends on the marginals.

The correction is the whole point, and the reason is the base rate. Raw agreement is inflated by whatever both annotators say most of the time. Two people labelling transactions as fraud or not, each calling 5 percent of them fraud, agree by chance alone at

$$p_e = 0.95^2 + 0.05^2 = 0.905$$

so a reported 92 percent agreement between them is

$$\kappa = \frac{0.92 - 0.905}{1 - 0.905} = 0.158$$

which is almost nothing. The 92 percent was bought by both of them saying "not fraud", and it says nothing about whether they agree on the cases the model is being built for.

### Where kappa misleads

Kappa has a known and well documented failure, and it is worth carrying because it arrives exactly in the situation labelling under [[Class Imbalance]] produces. Feinstein and Cicchetti set out the two paradoxes in 1990 ("High agreement but low kappa: I. The problems of two paradoxes", *Journal of Clinical Epidemiology*): kappa can be low despite high observed agreement, and unbalanced marginals can return a higher kappa than more balanced ones. So a low kappa is not by itself evidence that the annotators are unreliable, and a kappa compared across two datasets with different prevalences is not comparing like with like. The honest reading is to report $p_o$, the marginals and $\kappa$ together, and to treat kappa as a correction whose size you can see rather than as a score.

### More than two annotators

Krippendorff's alpha is the generalization, and it is built on disagreement rather than agreement:

$$\alpha = 1 - \frac{D_o}{D_e}$$

$D_o$ is the observed disagreement, the average distance between pairs of values assigned to the same unit, and $D_e$ is the disagreement expected if values were assigned at random subject only to how often each value occurs overall. $\alpha = 1$ is perfect reliability, $\alpha = 0$ is chance, and negative values indicate systematic disagreement.

What it generalizes over is three things at once, and each of them is a situation kappa simply cannot be computed in. Any number of annotators, not two. A rating matrix with holes in it, since not every annotator has to have seen every instance and nothing is dropped to force a rectangle. And any level of measurement, nominal, ordinal, interval or ratio, because the distance between two values is supplied as a function rather than assumed to be "same or different"; on an ordinal schema that means two annotators who chose adjacent categories are scored as closer than two who chose opposite ends, which kappa cannot express.

With the nominal distance function, two annotators and no missing ratings, alpha and kappa are answering the same question, and what still separates them is how each estimates chance: kappa multiplies the two annotators' own marginals together, while alpha computes its expected disagreement from the pooled distribution of all values. Alpha is therefore not kappa extended, and the two do not have to agree to three decimal places on the same table.

The operational point of all of this is that it turns "the annotators disagree" into a number that can be tracked, compared between guideline versions, and held to a threshold. That number only exists when the replication $r$ in [[Hand Labeling]] is greater than one, which is what that knob is for.

## Where it is used

[[Hand Labeling]] is where multiplicity arises and where it is dealt with: step 4 of that protocol decides whether disagreement is visible at all, step 6 adjudicates it, and step 7 is the measurement above. The remedy that works best is upstream of all three, and the raw material of this note is emphatic about it: a clear problem definition reduces the problem. Most annotator disagreement is not carelessness, it is several people reading an ambiguous schema correctly and differently, so the first response to a poor kappa is to look at which class boundary the disagreements cluster on and rewrite the guideline there, not to hire better annotators.

[[Poor-Quality Data]] is the neighbouring pathology and the two are worth keeping apart, because the remedies do not transfer. That note owns data that arrived wrong: errors, outliers, noise and missing values in $\mathbf{X}$, fixed by cleaning, dropping or imputing. This note owns labels produced inconsistently over instances that arrived perfectly fine, fixed by a clearer schema, adjudication and replication. Cleaning a feature column does nothing for a contested label, and adjudicating a label does nothing for a corrupted feature.

[[Weak Supervision]] is the same problem in programmatic form. Once heuristics rather than people emit the labels, two labeling functions firing opposite answers on one instance is a conflicting label by exactly this definition, which is why combining, denoising and reweighting the functions is not an optional refinement of that method but the step that makes it produce a single target at all. The difference is that a labeling function's disagreement is reproducible and can be inspected, where an annotator's cannot.

[[Class Imbalance]] is the condition under which the kappa paradox above bites, since a skewed target is precisely a skewed marginal.

Lineage is what makes any of this actionable, and it has two halves. The general half, provenance of a record, knowing which origin a row came from and who is answerable for it, is a property of the data system and is owned by [[Data Source]]. The label half belongs here: knowing which annotator produced which label, under which guideline version, at what measured accuracy against a gold set. That record is what makes a disagreement adjudicable rather than merely visible, since a weighted vote needs the accuracies, and it is what makes a mysterious model failure traceable, since pooling labels from several sources and several annotators without keeping track of which came from where leaves no evidence to follow when the model behaves strangely on one slice.
