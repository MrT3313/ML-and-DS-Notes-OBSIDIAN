---
note_kind: index
aliases:
  - Machine Learning Operations
  - ML Ops
up: "[[Machine Learning]]"
---

Everything that happens to a model after it ships: where it runs, whether it is still working today, and what you do when it stops. The *Ops* is borrowed from [[DevOps]], and what comes with it is the sense of operationalizing a thing, which means deploying it, monitoring it and maintaining it. That gloss is the whole boundary, and it is what stops a phrase as loose as "bringing machine learning into production" from quietly swallowing the work of building the model: the rest of the domain carries a model from vocabulary through fitting to an honest estimate of how it generalizes and ends there, and this is the fitted artifact from that point on, through deployment, monitoring, retraining and rollback. A note earns a place here when it is about a model already in production rather than a model being built, which usually means it only makes sense once there is live traffic and a clock running. Much of the industry literature draws the term wider than that, and the wider sense is the common one rather than a fringe usage: Google Cloud's practitioner guidance makes ML development, meaning experimentation, data preparation and model training, the first of seven MLOps lifecycle processes, Sculley and colleagues' account of technical debt in machine learning systems puts the model code in a small black box surrounded by data collection, feature extraction, configuration and serving infrastructure and argues that the surround is where the cost actually lives, and the continuous-delivery treatment of the subject starts at dataset discovery and versioning. This vault uses the narrower boundary anyway, because the pre-deployment half already has owners: `Data/`, `Evaluation/` and `Composition/` hold it, and a second home here would give the same material two. The view that spans both halves has a name and a note of its own and is not a second definition of this one: [[Machine Learning Systems Design]] takes the built half and the deployed half together, along with everyone who has a stake in either, and the objectives and requirements the whole is finally checked against are fixed there rather than here. [[DevOps]], in `Knowledge/Systems Architecture/Operations/`, is the same practice carried out on backend services and the data infrastructure beneath them, and what sits here is that practice on a system whose behaviour is learned from data rather than written by hand, which adds concerns that have no counterpart there: a model degrades while every component around it keeps working, so watching the input distribution, choosing a retraining cadence, and keeping the previous model available to return to all belong here and nowhere in that one. Where anything at all runs is separate vocabulary again, in `Knowledge/Systems Architecture/Hosting/`: [[Cloud Computing]] and [[Self-Hosting]] for who owns the machines, [[Serverless]] for renting the execution rather than the machine, and [[Cloud-Native Architecture]] for what a service has to look like to suit either. What belongs here is the model-specific half, what serving a fitted model asks that serving an ordinary service does not.

## Concepts

- **[[Production Machine Learning]]**, the setting the whole folder exists for, in which several parties each want something different from one model and the data underneath it will not hold still, so there is no single score left to rank candidates by.
- **[[Model Rot]]**, the drift of the production distribution away from the training distribution, and the reason a shipped model has a maintenance cost at all.
- **[[Continual Learning]]**, the standing answer to that drift, going on fitting the deployed model from data that arrives after it shipped, at the price of forgetting what it already knew.

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
