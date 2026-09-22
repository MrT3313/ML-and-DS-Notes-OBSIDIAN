---
note_kind: concept
aliases:
  - nonprobability sampling
  - non-probability sampling
  - nonprobability sample
  - convenience sampling
  - snowball sampling
  - judgement sampling
  - judgment sampling
  - quota sampling
up: "[[Sampling]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## Definition

Nonprobability sampling is any selection of data not governed by a probability criterion: who or what ends up in the sample is decided by availability, by a person's judgement, by a quota, or by who was already in the sample, and never by a draw whose odds anyone wrote down. No unit's inclusion probability is known, and usually some units have none at all.

These procedures are the commonest way to produce [[Nonrepresentative Training Data]]. The division of labour between the two notes is worth stating once: that note owns the mismatch between the sample and the population, what it is and what it does to a fitted model, and this one owns the procedures that cause it. A reader who wants to know why the model is wrong wants that note. A reader who wants to know which collection habit did it wants this one.

They are also the quick and easy way to get data in hand and a project off the ground, which is why they are everywhere. The honest position is not that they are forbidden but that what they buy is speed and what they cost is the ability to say anything about the population afterwards.

## Formal statement

The defect is exactly one quantity: the inclusion probability $\pi_i = P(i \in S)$ is unknown, or is zero for some units. What that costs is best seen against what probability sampling gets for free.

Under a probability design, with $\pi_i$ known and positive for every unit, the population total $T = \sum_{i \in U} y_i$ is estimated by the Horvitz-Thompson estimator

$$\hat{T} = \sum_{i \in S} \frac{y_i}{\pi_i}$$

and it is unbiased in one line. Write $Z_i = 1$ when $i \in S$ and $0$ otherwise, so $\mathbb{E}[Z_i] = \pi_i$. Then

$$\mathbb{E}\big[\hat{T}\big] = \mathbb{E}\left[\sum_{i \in U} \frac{y_i}{\pi_i} Z_i\right] = \sum_{i \in U} \frac{y_i}{\pi_i} \pi_i = T$$

The whole argument is the cancellation of $\pi_i$ against itself, and it needs $\pi_i$ known (or the division cannot be performed) and $\pi_i > 0$ (or the division is undefined and the unit contributes nothing at any sample size).

Nonprobability sampling has neither, so there is no design-unbiased estimator of a population quantity to be had from it. The sharper consequence is the one that catches people: the bias is not estimable from inside the sample. Every diagnostic available at fitting time reads the sample, the size of the bias is a statement about the units that are not in it, and growing $n$ produces more units from the same frame rather than the units that were missing. A larger nonprobability sample is a more precise estimate of the wrong thing.

### The four selection rules

The family is named for what is absent, so what distinguishes its members is the rule that stands in for the missing draw.

| Rule | What decides membership | Worked case |
| --- | --- | --- |
| Convenience sampling | availability, and nothing else | the corpus that was already sitting in a bucket, used because it was there |
| Snowball sampling | existing members choose the next ones | scraping legitimate accounts on a social platform with no access to its database: start from a handful, scrape everyone they follow, repeat |
| Judgement sampling | an expert decides what to include | a specialist picks the cases worth labelling |
| Quota sampling | fixed counts per slice, filled in any order and without randomization inside a slice | a survey wanting 100 responses from each of under 30, 30 to 60, and over 60, regardless of what the real age distribution is |

Quota sampling is the one most often mistaken for [[Stratified Sampling]], and the difference is the whole of this note. Both fix the share each group holds. Stratified sampling then draws at random inside each group, so $\pi_i$ is known and positive within the stratum. Quota sampling fills the quota with whoever is to hand, so inside the group nothing is known at all, and the matched margins buy no more than the appearance of a matched sample.

Snowball sampling has a second defect of its own: membership is transmitted along the network's own edges, so the sample inherits the network's clustering, and units in a component nobody started from have $\pi_i = 0$ by construction rather than by accident.

## Where it is used

The live case is the training data under large language models. They are not fitted on a sample representative of all possible text. They are fitted on text that could be collected: web crawls, encyclopedia dumps, forum archives, whatever had a bulk download. That is convenience sampling at scale, and the selection criterion is what a crawler could reach rather than what the language actually contains, which is the mechanism behind [[Nonrepresentative Training Data]] in the most widely deployed models there are. [[Data Source]] is where those corpora come from, since every origin it names has its own reachability and its own contract on quality.

The remedy, where there is one, is to go back up to [[Sampling]] and pick a design with known positive inclusion probabilities, which usually means naming the population first and discovering that it was never written down.
