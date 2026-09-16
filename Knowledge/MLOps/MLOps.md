---
note_kind: index
aliases:
  - Machine Learning Operations
  - ML Ops
up: "[[Home]]"
---

Everything that happens to a model after it ships, kept as its own domain because it answers a different question than the one `Knowledge/Machine Learning/` answers. That domain covers building a model: framing the task, preparing the data, fitting, and estimating how well the fitted thing generalizes. This one covers operating the model that came out, where it runs, whether it is still working today, and what you do when it stops. A note earns a place here when it is about a model already in production rather than a model being built, which usually means it only makes sense once there is live traffic and a clock running. Today the domain holds one note, the reason the clock matters at all.

## Areas

- **Decay over time.** [[Model Rot]], the drift of the production distribution away from the training distribution, and the reason a shipped model has a maintenance cost at all.

## What is missing

Nearly everything, and the gap is a known one: the last section of chapter 2, launch, monitor and maintain, produced no note whatsoever, which is why this folder holds a single note taken from chapter 1. The honest list, in the order the book will force it:

- Deployment targets: where a trained model actually runs once it leaves a notebook. Chapter 19, Training and Deploying at Scale, is the chapter that brings this and most of the rest of this list.
- Monitoring and alerting on live inputs, the machinery that would let [[Model Rot]] be detected rather than merely defined. [[Model Rot]] already says detection requires monitoring in production and points at chapter 2 for it, a pointer nothing currently satisfies.
- A retraining cadence: how often a batch-trained model is refit, and on what signal rather than on habit. [[Batch Learning]] names the schedule as its standing cost and does not say how the schedule is chosen.
- Rollback, what you do with the previous model when the new one is worse in production.
- Where the confidence interval on a shipped model's error belongs. Chapter 2 puts a bootstrap interval around the test-set error, `Knowledge/Mathematics/` lists the bootstrap itself as missing, and whether the interval is an evaluation concern or an operational one is unsettled.

```base
filters:
  and:
    - file.hasLink(this.file)
    - file.inFolder("Knowledge")
views:
  - type: table
    name: Linked here
    order:
      - file.name
      - note_kind
      - confidence
```
