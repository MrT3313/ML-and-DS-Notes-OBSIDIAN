---
note_kind: concept
aliases:
  - feature generalization
  - feature coverage
  - coverage of a feature
up: "[[Feature Engineering]]"
sources:
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

Feature generalization asks whether a column will still be populated, and will still mean the same thing, on data the model has not seen. It is a property of a **column**, and that is what separates it from [[Generalization]], which is a property of a **fitted model** and is that model's error on data it was not fitted on. The word is shared and the subject is not: one asks how well $h$ does away from the rows it was fitted on, the other asks whether $x_j$ is even there to be read. A reader who arrived looking for the model's error wants the other note.

Two things about the vocabulary, said here so that nothing below overstates its standing. **Neither "feature generalization" nor "coverage" in the sense used below is an established term of art**; both are [[DMLS Ch05 Feature Engineering|Huyen]]'s usage, and a search of the wider literature under either name turns up very little. The quantity itself is entirely standard under other names: the missingness rate or the proportion of missing values in the statistics literature, and **completeness** in the data quality literature, where it is one of the dimensions Wang and Strong (*JMIS* 12(4), 1996) derived and which Batini and Scannapieco carry with the same ratio behind it. Saying "coverage" is fine as long as it is glossed, and the gloss is needed, because the bare word already means several other things nearby: a confidence interval's coverage probability, the fraction of code a test suite exercises, and the share of a catalogue a recommender is able to reach.

The second thing is an assessment rather than a method, and it is Huyen's: measuring this is a great deal less scientific than measuring [[Feature Importance]], and doing it well takes intuition and subject matter expertise on top of any statistics. Nothing below turns that into a procedure it is not.

## Formal statement

The one quantity here with a definition behind it is **coverage**, the share of rows on which the column actually has a value. With $m$ instances and column $j$,

$$\operatorname{cov}(j) \;=\; \frac{1}{m}\sum_{i=1}^{m} \mathbb{1}\big[x^{(i)}_j \text{ observed}\big]$$

so $\operatorname{cov}(j) \in [0, 1]$, and it is exactly one minus the column's missingness rate. The denominator is the full row count of whichever set is being measured, and naming it matters, because both claims that hang off coverage compare the same quantity computed over two different sets.

**A sharp gap in coverage across the split is evidence about the split, not about the column.** Compute $\operatorname{cov}_{\text{train}}(j)$ and $\operatorname{cov}_{\text{test}}(j)$ separately. A column's rate of presence is a property of the distribution its rows were drawn from, so a genuinely random split leaves the two equal up to sampling error, and the size of that error is known: writing $p = \operatorname{cov}(j)$ over the whole file, the split-level estimate has standard error

$$\operatorname{SE} \;=\; \sqrt{\frac{p(1-p)}{m_{\text{test}}}}$$

which puts the claim within reach of being checked rather than eyeballed. A gap of many multiples of that is not sampling noise, and what it indicts is the procedure that produced the two sets: a time-ordered file cut at a date, a sort left in place before an index-based split, a merge in which different sources fed different parts of the range. That is the same failure [[Nonrepresentative Training Data]] names on the collection side and [[Data Mismatch]] names as an outcome, arriving here through the split instead. It is also one of the standing signals that [[Data Leakage]] is present, and for a reason the gap itself does not show: a column whose presence tracks the position of a row in the file is a column that encodes something about when or where that row came from, and that is precisely the shape a leaked feature takes.

**A column present on very few rows is unlikely to generalize, and that is a rule of thumb rather than a result.** The reasoning behind it is that a coefficient or a split condition fitted from a handful of rows is fitted to those rows, so the column adds variance without adding much signal, which is [[Overfitting]] reached through sparsity.

What makes it only a rule of thumb is the **missingness mechanism**, and the case that breaks it is MNAR. Write the presence pattern as a column of its own,

$$r^{(i)}_j \;=\; \mathbb{1}\big[x^{(i)}_j \text{ observed}\big]$$

so that $\operatorname{cov}(j)$ is nothing but the mean of $r_j$. Under missing completely at random, $r_j$ is independent of everything in the table and carries no information, and a low-coverage column really is close to useless. Under missing not at random the value is absent *because of what it is*, so the distribution of $r_j$ depends on the value that is gone and $y \not\perp r_j$ in general: the fact of the blank is itself predictive even where $\operatorname{cov}(j)$ is tiny. A field that only high earners decline to fill in is the standing case, at one percent coverage with the other ninety-nine percent of rows telling you something. Keeping that information is what a missingness indicator column is for, and it is the one part of an MNAR column that can be kept, since no procedure using observed data alone recovers the values themselves.

Two riders on that, both of which cut the other way. **The indicator can be predictive under MAR too**, where missingness depends on columns you do hold, so a useful indicator is not by itself evidence of MNAR; what is specific to MNAR is that $r_j$ carries information about the absent *value*, which is why the mechanism is non-ignorable for inference rather than merely useful for prediction. And **a predictive missingness indicator is a fragile feature in exactly this note's sense**: what it encodes is the current reason a field goes blank, so an upstream change that alters that reason breaks the column silently, with the coverage figure moving and the model's score moving with it and nothing raising an error. [[Missing Value Imputation]] carries the three mechanisms and the argument that which one holds is assumed from how the data was collected rather than read off the file, which means the exception to this rule of thumb is not one a statistic can flag for you.

## Where it is used

[[Generalization]] is the note this one is most likely to be confused with and the reason the first paragraph exists: model error against column availability, one word over two subjects. [[Feature Importance]] is the contrast the raw material draws explicitly, and the two are complements rather than competitors, since importance measures what a column is worth on the rows that have it while coverage measures how many rows those are. A column can score high on one and low on the other, and that combination is the one most worth investigating, because a rare column that matters enormously is either a genuine specialist signal or a leak.

[[Data Leakage]] takes coverage divergence as one of its signals, for the reason given above. [[Nonrepresentative Training Data]] and [[Data Mismatch]] hold the general statement of what a coverage gap is evidence for, one on the collection side and one as the outcome, and those two carry the names `sampling bias`, `distribution shift` and `train-serving skew`, so the vocabulary is theirs and is not restated here.

[[Missing Value Imputation]] owns the mechanism that decides whether low coverage is fatal or informative, and it owns the indicator column that is how a low-coverage MNAR field is actually kept. [[Feature Engineering]] is the parent activity: coverage is one of the two readings taken on a candidate column before it is allowed into $\phi$, the other being importance.
