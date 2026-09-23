---
note_kind: concept
aliases:
  - data snooping
  - snooping bias
  - data snooping bias
  - snooping
  - peeking at the test set
up: "[[Testing Set]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

Data snooping is what happens when you look at the held-out data before the model is finished, and then make choices informed by what you saw. Every decision you make afterwards, which features to build, which model to try, which transformation looks promising, is now conditioned on the [[Testing Set]], so its score stops being an honest estimate of [[Generalization]] error and becomes a score the choices were fitted to.

The analyst is the mechanism, and naming that is what keeps this apart from its nearest neighbour. Snooping is *selection*: you are choosing among candidates on the strength of the evidence those candidates were supposed to be judged by, which is why its content below is an inequality over a search rather than a statement about any one column. The other family is held-out or label information reaching a *fitted transformer* or a feature value with nobody looking at anything, and that is [[Data Leakage]]. The two meet only where a statistic is fitted across the split, ex a scaler or an imputer estimated on all $m$ rows, and those cases belong to leakage because the contamination is in the transformer rather than in your choices.

## Formal statement

The test estimate is unbiased only while the hypothesis is chosen independently of the test sample. If $h$ is fixed before $D_{\text{test}}$ is drawn,

$$\mathbb{E}\big[\hat{\mathcal{L}}_{\text{test}}(h)\big] = \mathcal{L}_{\text{gen}}(h)$$

Once the choice of $h$ depends on $D_{\text{test}}$, write it $h = \hat{h}(D_{\text{test}})$. Selection picks whichever candidate the test sample happened to favour, so the expectation falls below the true risk,

$$\mathbb{E}\big[\hat{\mathcal{L}}_{\text{test}}(\hat{h})\big] \le \mathcal{L}_{\text{gen}}(\hat{h})$$

with the gap widening as the number of candidates grows and as $m_{\text{test}}$ shrinks. The human version is the same inequality with the search running inside your head rather than in a loop, which makes the number of candidates unbounded and unrecorded.

## Where it is used

It is the reason the test set is split off before any exploration, and the reason all plotting, summarizing, and correlation hunting happen on the [[Training Set]] alone. It is why [[Model Selection]] and hyperparameter tuning run against [[Holdout Validation]] or [[Cross-Validation]], never against the test set: those are the sacrificial splits, and they are allowed to become contaminated because the test set is what stays clean. Snooping and [[Overfitting]] are the same mechanism at different levels: overfitting is a model fitted to training noise, snooping is an analyst fitted to test noise. A split that silently reshuffles between runs, discussed in [[Random Sampling]], produces the same contamination without anyone ever deciding to peek, and is therefore [[Data Leakage]] rather than snooping.

The uncomfortable consequence is that the bias is unmeasurable after the fact. Once you have seen the test set there is no correction to apply and no diagnostic to run; the only remedy is fresh data.
