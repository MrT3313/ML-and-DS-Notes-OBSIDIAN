---
note_kind: concept
aliases:
  - natural label
  - natural labels
  - natural ground truth
  - behavioral label
  - behavioral labels
  - behavioural labels
  - implicit label
  - implicit labels
  - explicit label
  - explicit labels
up: "[[Training Set]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## Definition

A task has natural labels when the system can evaluate its own predictions, in whole or in part, because the outcome turns up on its own and nobody has to be asked for it. Three cases make the shape clear. A route's arrival time is estimated when the trip starts, and when the trip ends the system knows how long it actually took. A stock price is predicted two minutes ahead, and two minutes later the price is read off the tape. A recommendation is served, and whether the user clicked it is something the serving system already records for other reasons.

## Formal statement

The precise content is a contract. A prediction $\hat{y}$ is served at time $t$. Its label is natural when there exists an event $e$ observable in the system's own state and a bounded delay $\tau$ such that the state at $t + \tau$ determines $y$, with no human asked to supply it.

Each clause carries weight.

- **Observable in the system's own state.** If determining $y$ requires reaching outside the running system, to a person, to a vendor, to a court record, the label is not natural however inevitable the outcome is. The trip duration qualifies because the application is already watching the trip.
- **A bounded $\tau$.** An outcome that arrives eventually with no bound on when is not a label you can plan a training set around. That bound is [[Feedback Loop Length]] and it is the quantity that decides whether a natural label is any use.
- **Determines $y$.** The state at $t + \tau$ has to fix the label, not merely suggest it. Where it fixes it only for part of what was predicted, the task is partially natural, which is the ordinary case rather than a degenerate one: a click tells you the item you served was taken and tells you nothing at all about the items you did not serve.

What the contract does and does not buy:

| claim | holds | why |
|---|---|---|
| a label arrives without anyone being asked | yes | this is the contract itself, and it is the entire economic argument for the arrangement |
| the label arrives within a known time | yes | $\tau$ is bounded by assumption, and its actual value is [[Feedback Loop Length]] |
| the label is correct | no | the observed event stands in for the target and is not the target. A click records that an item was chosen, not that it was any good, and a purchase later returned was still a click |
| the label is unbiased over the input space | no | it is defined only over what the system chose to serve, so the distribution of labelled instances is the distribution of the serving policy rather than of the world |
| the label is independent of the model that produced the prediction | no | the model decides what is served, what is served decides which labels come into existence, and those labels train the next model, so the system is generating the evidence about itself |

The last two rows are one mechanism and it is worth stating as a claim that could be found false: **what is never shown is never clicked**, so an absent positive signal confounds "the user did not want this" with "the user never saw this", and no amount of data fixes it, because more data is more data collected under the same policy. The standard measurement of this is Joachims et al. (SIGIR 2005), who compared clickthrough against manual relevance judgements with eye tracking and found clicks informative but biased by presentation order, with users placing substantial trust in the ranking they were shown. Their conclusion is the usable one: clicks are unreliable as absolute relevance judgements and reasonably accurate as relative preferences, ex a clicked result preferred over a higher ranked result the user skipped.

### Explicit and implicit feedback

These two words are fixed by the recommender and information retrieval literature, and they are about where a signal came from, not about how confident anyone is in it.

- **Explicit feedback** is a preference the user states outright, because stating a preference was the point of the action: a star rating, a like, a thumbs down.
- **Implicit feedback** is inferred from behaviour the user had some other reason for: a click, a purchase, a watch, time spent on a page. Hu, Koren and Volinsky (ICDM 2008) state the defining asymmetry, that implicit feedback carries no direct input from users about their preferences and in particular lacks substantial evidence about which items a user dislikes. Negative signal is not recorded anywhere; it is presumed from absence.

Both of the recommender labels above are therefore implicit, and this is the part most often garbled. A recommendation that goes unclicked for some period, presumed to be bad, is implicit by definition: a negative presumed from the lack of a positive is the textbook case of what the word means, and it cannot be explicit, since the user never said anything. A clicked recommendation presumed to be good is implicit too, for the same reason in the other direction: the user clicked, they did not say they liked it. To get an explicit label out of a recommender you have to ask, with a rating widget or a thumbs down, and then you have a different signal with different problems, collected from the small fraction of users who bother.

### Behavioral labels

Natural labels inferred from user behaviour, clicks and ratings among them, are called behavioral labels. One caution is worth stating because the two vocabularies overlap without matching: a rating is explicit feedback and a click is implicit, so "behavioral" spans both and is not a synonym for "implicit". What makes a label behavioural is that a user action produced it. What makes it implicit is that the user was not expressing a preference when they acted.

## Where it is used

[[Feedback Loop Length]] is the quantity that decides whether a natural label is usable in practice rather than merely available in principle, since a label that arrives after the decision it was meant to inform is not a label anyone can act on. [[Hand Labeling]] is the alternative when no natural label exists, and it is also the complement when one does, because auditing a natural label against a hand labelled slice is the only way to check the "correct" row of the table above.

[[Continual Learning]] is what a standing supply of natural labels makes possible: the whole practice depends on data arriving after deployment already carrying its target, and a task with natural labels supplies that for free while a task without one has to pay a labelling bill on every cadence. [[Model Rot]] is the decay that arrangement is defending against, which is why the supply matters more than its cleanliness.

[[Data Source]] is the cross-domain account of where these signals physically come from: behavioural tracking of what a user clicked and how long they lingered is system generated by origin and user data by law, so the labels this note treats as free are carrying privacy obligations that the logs beside them do not.
