---
note_kind: concept
aliases:
  - offline learning
  - batch training
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

In batch learning the system is trained once on the whole available dataset, then deployed without further learning. Incorporating new data means retraining from scratch on old plus new and redeploying.

## Formal statement

Training consumes all $m$ instances of $D$ at once to produce $\boldsymbol\theta^{*}$; a fixed $h_{\boldsymbol\theta^{*}}$ serves every query until the next retrain. Cost of an update is a full pass over $|D_{\text{old}}| + |D_{\text{new}}|$ instances.

**Not further quantitative at this depth.**

Batch learning is a training regime, and the update cost above is the whole of its mathematics at this depth: it states when training runs and on how many instances, not how the parameters are fitted. A later chapter would add the per-algorithm cost of that full pass, ex the cost of the [[Normal Equation]] in $m$ and $n$ or the epoch count of [[Batch Gradient Descent]].

## Where it is used

The alternative is [[Online Learning]]. A fixed model decays as the world moves, which is [[Model Rot]]; the retrain cadence is the lever. Batch learning fails outright when the data exceeds memory, which is where [[Out-of-Core Learning]] comes in.
