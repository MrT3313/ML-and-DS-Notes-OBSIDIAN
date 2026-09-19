---
note_kind: index
aliases:
  - Machine Learning Operations
  - ML Ops
up: "[[Home]]"
---

Everything that happens to a model after it ships, kept as its own domain because it answers a different question than the one `Knowledge/Machine Learning/` answers. That domain covers building a model: framing the task, preparing the data, fitting, and estimating how well the fitted thing generalizes. This one covers operating the model that came out, where it runs, whether it is still working today, and what you do when it stops. A note earns a place here when it is about a model already in production rather than a model being built, which usually means it only makes sense once there is live traffic and a clock running. [[DevOps]], in `Knowledge/Systems Architecture/Operations/`, is the same practice carried out on backend services and the data infrastructure beneath them, and this domain is that practice on a system whose behaviour is learned from data rather than written by hand, which adds concerns that have no counterpart there: a model degrades while every component around it keeps working, so watching the input distribution, choosing a retraining cadence, and keeping the previous model available to return to all belong here and nowhere in that one. Where anything at all runs is vocabulary that sits outside this domain too, in `Knowledge/Systems Architecture/Hosting/`: [[Cloud Computing]] and [[Self-Hosting]] for who owns the machines, [[Serverless]] for renting the execution rather than the machine, and [[Cloud-Native Architecture]] for what a service has to look like to suit either. What belongs here is the model-specific half, what serving a fitted model asks that serving an ordinary service does not. Today the domain holds one note, the reason the clock matters at all.

## Areas

- **Decay over time.** [[Model Rot]], the drift of the production distribution away from the training distribution, and the reason a shipped model has a maintenance cost at all.

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
