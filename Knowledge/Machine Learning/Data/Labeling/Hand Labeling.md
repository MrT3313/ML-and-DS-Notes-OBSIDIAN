---
note_kind: method
aliases:
  - hand labeling
  - hand labelling
  - hand labels
  - manual labeling
  - manual annotation
  - annotation
  - annotator
  - annotators
  - human annotation
up: "[[Training Set]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Hand labeling is people looking at instances and writing down the target for each one, so that a [[Training Set]] acquires the label vector $\mathbf{y}$ it did not arrive with. Nearly everything in production today is a [[Supervised Learning]] model, and that single fact is what makes this an infrastructure problem rather than a chore: a supervised system is bounded on both sides by its labels, by how many of them exist and by how right they are. Neither bound has anything to do with the algorithm. Too few labels is [[Insufficient Training Data]] and shows up as variance; labels that are wrong are a target the model will fit faithfully, and a perfect fit to a guess is a fit to a guess.

You reach for hand labeling when the target is not recorded anywhere and cannot be derived. Check the alternatives first, in this order, because each of them is cheaper per label.

- [[Natural Labels]] are the case where nobody labels anything, because the system observes the outcome itself. If the task has them, hand labeling is the wrong tool for the bulk of the work and belongs only on a held-out slice used to audit the natural ones.
- [[Weak Supervision]] replaces the annotator with a labeling function, so a subject matter expert writes a heuristic once instead of applying it a hundred thousand times. It buys scale and privacy and pays in label noise.
- [[Active Learning]] keeps the annotator and cuts how many instances reach them, by ordering the queue so the model sees the instances it is most confused about first.

The honest reading is that these are rarely exclusive. A real project hand labels a seed set, uses it to write and score labeling functions or to seed an active loop, and goes on hand labeling the slices where neither works.

## Algorithm

The steps are ordered, and the ordering is most of the content. Write the unlabelled pool as $U = \{\mathbf{x}^{(i)}\}_{i=1}^{n}$ and the label space as $\mathcal{Y} = \{1, \dots, K\}$.

1. **Fix the problem definition and the label schema before anything is labelled.** What $\mathcal{Y}$ is, what each class means, and which class an instance gets when it could plausibly take two. This is a modelling decision, not a clerical one.
2. **Write the guideline, and write it around the boundary cases.** The classes nobody disputes need no guidance. Everything an annotator will actually hesitate over is a worked example in the document, with the answer and the reason.
3. **Choose and qualify the annotators.** Train them on the guideline, then have them label a gold set whose answers are already settled, and measure each one against it. That per-annotator accuracy is used twice later: to decide who adjudicates, and to weight conflicting labels.
4. **Decide the replication $r$**, how many annotators label each instance. This is the knob that decides whether disagreement is even visible: at $r = 1$ there is no second opinion and no agreement statistic exists.
5. **Label**, recording for every label which annotator produced it, when, and against which version of the guideline.
6. **Adjudicate the disagreements** by a rule fixed in advance, not case by case: majority vote, an expert's tie-break, or a weighted vote using the accuracies from step 3.
7. **Measure agreement** over the replicated instances rather than eyeballing it, which is [[Label Multiplicity]] and its statistics.
8. **Feed the disagreements back into the schema and the guideline**, then relabel whatever the revision invalidated, and bump the guideline version so step 5's record stays meaningful.

Step 1 sits first because of what step 8 costs. A schema settled after labelling has begun does not cost the time to change the document; it costs relabelling everything already done under the old schema, and the bill grows with every instance labelled before the problem was understood. Most of the disagreement measured in step 7 is a symptom of step 1 having been skipped: annotators who disagree are usually reading an ambiguous schema correctly and differently, rather than reading a clear one carelessly.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| annotators per instance | $r$ | 1 | each instance comes back with more independent judgements, so the recorded label depends less on one person's reading and an agreement statistic becomes computable at all; cost and elapsed time grow linearly in $r$ | $r = 1$ where a qualification run already shows near-total agreement. Raise to 3 or more only on the slices where agreement is poor, rather than uniformly across the pool, since uniform replication spends most of its budget confirming the easy instances |
| schema granularity | $K$ | the fewest classes that answer the question being asked | finer distinctions are recorded, and each class gets fewer instances while the number of boundary cases between classes grows, so agreement falls and the per-class supports thin toward [[Class Imbalance]] | split a class only when the measured disagreement is concentrated inside it, and merge two classes whose confusion is what annotators disagree about most |
| adjudication rule | - | majority vote over the $r$ labels | - | weight by the per-annotator accuracies from the gold set when they differ materially; reserve expert tie-break for the instances where the weighted vote is close, since it is the expensive option |

Two things sit outside the table on purpose. The guideline version is pinned rather than tuned: it changes what comes back, which is exactly why every label is stamped with it, but there is no direction to turn it in. Annotator pay, session length and tooling latency move how fast and how expensively the labels arrive and not which labels arrive, so they fail the test the same way `n_jobs` does.

## Failure modes

- **It is expensive, and the price is set by who has to do it.** Anyone can mark whether an image contains a cat. Deciding whether a chest X-ray shows an effusion, or whether a contract clause is an indemnity, needs a subject matter expert, and the cost per label moves by orders of magnitude when it does. The failure is a project plan costed at crowd rates for a task that turns out to need a radiologist.
- **It is a threat to data privacy, because somebody has to look at the data.** The method's requirement is that a human reads the instance, and there are instances no human outside the system is permitted to read: medical records, private messages, anything under a contractual or statutory restriction on disclosure. Shipping such data to annotators is not a slow version of the right answer, it is not available at all, which is what makes [[Weak Supervision]] structurally rather than economically attractive here.
- **It is slow, and slow labelling is slow iteration.** Time to label scales roughly linearly with the number of labels wanted, so the labelling step, not the fitting step, sets how long it takes to answer a question about the model. The cost of that compounds rather than sitting still: a model whose retraining is gated on a labelling run that takes weeks is a model that cannot follow an environment that moves in days, which is [[Model Rot]] accumulating for the whole length of the gap. The requirements move too, and each change of requirement means relabelling rather than refitting.
- **The schema changes mid-project.** A class is split, a boundary is redrawn, a new class is added for the cases nobody anticipated. Every instance labelled under the old schema is now labelled under a different question from the instances labelled after, and the two sets are silently incompatible: nothing errors, the file still has one label per row, and the model is fitted on a target that means two things. The fix is relabelling, and the size of the bill is the number of instances labelled before anyone noticed.
- **Disagreement left unmeasured.** At $r = 1$ every label looks equally authoritative, because there is nothing to compare it against, and a genuinely ambiguous schema is indistinguishable from a clear one. The disagreement did not go away; it was recorded as a fact instead. See [[Label Multiplicity]].
- **Labels pooled from several sources without provenance.** Data from multiple sources, annotated by multiple people at different levels of expertise, merged into one file with no record of which label came from where, produces a model that fails in ways nobody can trace, because the evidence needed to trace it was never written down. This is the failure that step 5 of the protocol exists to prevent, and it is adjacent to but not the same as [[Poor-Quality Data]], which is about the instances rather than the labels.

## Implementation

There is no scikit-learn call for this, and there will not be one: the method's central step is a person reading something. What implements it is software that presents instances to people, collects what they say, and reconciles the results. The systems below are named as instances of that, not described as products, and every line carries a date, because a roster of commercial tools goes stale faster than anything else in this vault. Tooling for machine learning operations in depth, this market included, belongs to [[DMLS]] chapter 10 and is not settled here.

Verified as of September 2026:

- **Label Studio** (HumanSignal) is the self-hosted, open source general annotation server: one deployment covers text, image, audio and video through configurable task interfaces, with a paid enterprise edition on top.
- **CVAT** (cvat.ai) is the same shape specialised to images and video, with an open source self-hosted tier alongside a managed cloud one.
- **Prodigy** (Explosion) is the single-machine, scriptable, paid option, built so the queue is ordered by a model rather than fixed, which makes it [[Active Learning]] delivered as a labelling tool instead of as a loop you write yourself.
- **Amazon SageMaker Ground Truth** is the managed form, where you rent the workforce as well as the tool, choosing between Amazon Mechanical Turk, a vendor from the AWS Marketplace, or your own private workforce. Its documentation now opens with a notice, quoted verbatim: "Amazon SageMaker Ground Truth is no longer open to new customers. Existing customers can continue to use the service as normal." That is the sharpest available argument for dating every line above.

What actually differs between them is three things, and each maps onto a failure mode. Where the data goes decides whether the privacy failure applies at all, and self-hosting is the only answer to it that does not involve a contract. Who supplies the people decides the cost and the qualification story, since a rented crowd has to be qualified per project while an internal team is qualified once. And whether the queue is model-ordered decides how much of the pool ever reaches a person.
