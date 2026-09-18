---
note_kind: index
aliases:
  - Vault Home
---

Root of the vault. A source of truth for machine learning, MLOps, data systems, and data science, built from book sources one chapter at a time.

## Areas

- [[Machine Learning]] the main body of the vault.
- **`Knowledge/MLOps/`** operating models in production, indexed at [[MLOps]]. [[Model Rot]] so far.
- **`Knowledge/Data Systems/`** where data lives, how it gets there and who serves it, indexed at [[Data Systems]]. Twenty-four notes from one chapter so far.
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

- [[HOML]] Hands-On Machine Learning with Scikit-Learn, Keras and TensorFlow, Géron, 2022. Chapters 1 to 4 extracted.
- [[DDIA]] Designing Data-Intensive Applications, Kleppmann and Riccomini, 2026. Chapter 1 extracted.

## Datasets
- [[California Housing]] the 1990 census extract chapter 2 runs end to end, and the file behind every worked example here.
- [[MNIST]] 70,000 handwritten digit images, 784 pixel features apiece, the file every chapter 3 example runs on.
- [[Iris]] 150 flowers, 50 of each of three species, four petal and sepal measurements in centimetres, the file chapter 4 runs its logistic and softmax examples on. Chapter 4 uses only the two petal columns and never touches the sepals.

## Reference

- [[Notation]] symbol conventions used across every note.
- [[Quality Bar]] what a note must contain before it counts as done.
- [[Callouts]] the Obsidian callout types available.

## Working notes

`_Slip Box (Zettelkasten)/` is the landing place for quick notes taken on the fly, before they are integrated into the structure above. Nothing there is expected to meet the [[Quality Bar]] yet.

## What is missing

- Data science is named in the scope above and has no folder under `Knowledge/` and no hub. Nothing on disk is filed under it; the exploration, sampling and preprocessing material sits under `Knowledge/Machine Learning/Data/`, and whether data science becomes a domain of its own or stays a name for that folder is undecided.
- `Knowledge/MLOps/` holds one note, [[Model Rot]]. Its hub promises deployment targets for a trained model, monitoring of live inputs, a retraining cadence and rollback, and none of them is written; HOML chapter 19 is the chapter that brings most of it.
- `Knowledge/Data Systems/` is one chapter old. It names the parts and describes how they are arranged, with no note yet on any single component, and databases, search indexes, batch processing and stream processing arrive with later DDIA chapters.
- `Knowledge/Mathematics/` holds three families of results, and its hub lists linear algebra, probability and statistics as undefined, each of them assumed by notes in the other domains.
