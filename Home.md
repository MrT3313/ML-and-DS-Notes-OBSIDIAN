---
note_kind: index
aliases:
  - Vault Home
---

Root of the vault. A source of truth for machine learning, MLOps, data systems, and data science, built from book sources one chapter at a time.

## Areas

Four domains, each answering a different question, none of them subordinate to another. Their sizes differ only because of which chapters have been extracted so far.

- **`Knowledge/Machine Learning/`** how a model is specified, fitted and evaluated, and how the data it is fitted on is obtained, labelled and repaired, indexed at [[Machine Learning]].
- **`Knowledge/Data Systems/`** where data lives, how it gets there and who serves it, indexed at [[Data Systems]].
- **`Knowledge/Systems Architecture/`** the concepts that hold for any software system regardless of what it stores, what is required of such a system before anything about its arrangement is settled, how work is distributed across machines, how processes that share no memory pass data to one another, who owns the machines and who operates them, indexed at [[Systems Architecture]].
- **`Knowledge/Mathematics/`** the results the other domains lean on, indexed at [[Mathematics]], which lists them.

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

## Sources

- [[HOML]] Hands-On Machine Learning with Scikit-Learn, Keras and TensorFlow, Géron, 2022.
- [[DDIA]] Designing Data-Intensive Applications, Kleppmann and Riccomini, 2026.
- [[DMLS]] Designing Machine Learning Systems, Huyen, 2022.

## Datasets

- [[California Housing]] a 1990 census extract, and the file behind every worked example here.
- [[MNIST]] 70,000 handwritten digit images, 784 pixel features apiece.
- [[Iris]] 150 flowers, 50 of each of three species, four petal and sepal measurements in centimetres.

## Reference

- [[Notation]] symbol conventions used across every note.
- [[Quality Bar]] what a note must contain before it counts as done.
- [[Callouts]] the Obsidian callout types available.

## Working notes

`_Slip Box (Zettelkasten)/` holds quick notes taken on the fly, before integration, and nothing there meets the [[Quality Bar]] yet.
