---
note_kind: index
aliases:
  - classifier
  - classifiers
  - classification task
up: "[[Machine Learning]]"
---

Every method that predicts a discrete class, label, or category for an input: spam or not, cat or dog or car. A [[Supervised Learning]] task, separated from [[Regression]] by its target being drawn from a finite unordered set rather than from a continuum. Four varieties sit under it, told apart by how many labels one instance may carry and how many values each label may take: [[Binary Classification]], one label with two values; [[Multiclass Classification]], one label with three or more mutually exclusive values; [[Multilabel Classification]], several binary labels at once; [[Multioutput Classification]], several labels each with three or more values. Whichever variety is in play, a classifier is judged in one of two ways. Measures taken on predicted labels are read off a [[Confusion Matrix]] of predictions against truth, and each is a ratio of its cells rather than an independent measurement. Measures taken on predicted probabilities are not: [[Log Loss]] reads the probability itself and no confusion matrix can produce it.

## Methods

- [[Logistic Regression]]: linear score through a sigmoid, thresholded. The baseline.
- [[Softmax Regression]]: one linear score per class, turned into a distribution over the classes and fitted as a single model over all $K$ of them at once. The alternative to wrapping a binary classifier in [[One-versus-Rest]] or [[One-versus-One]], since the classes compete inside one optimization rather than being fitted apart and compared afterwards.
- [[Stochastic Gradient Descent Classifier]]: a linear classifier fitted one instance at a time, its `loss` deciding which linear model is actually being fitted. The choice when the training set is too large to pass over whole, or arrives as a stream.

Meta-estimators, which classify nothing themselves and instead extend a binary classifier to the other three varieties:

- [[One-versus-Rest]]: one binary classifier per class, each trained on the whole set, highest score wins. The cheaper decomposition in classifier count and the usual default.
- [[One-versus-One]]: one binary classifier per pair of classes, each trained only on those two classes, most votes wins. Worth its extra classifiers when the base algorithm's fit cost grows faster than linearly in the number of instances.
- [[Classifier Chain]]: one binary classifier per label, each fed the labels predicted before it, so the labels are not assumed independent of one another.

Not yet written, with the chapter that brings each:

- Support vector machines, chapter 5.
- Decision trees, chapter 6.
- Random forests, voting, bagging and boosting, chapter 7.
- k-nearest neighbours, which chapter 3 uses for its multilabel and multioutput examples but which no HOML chapter takes as its subject.

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
    sort:
      - property: file.name
        direction: ASC
      - property: confidence
        direction: DESC

```

## What is missing

- No classifier note for the families the later chapters own: support vector machines, decision trees, and the ensembles. Until those are written, every model on this list is linear and the index offers no alternative to a [[Decision Boundary]] that is a flat surface.
- k-nearest neighbours is used by chapter 3 and owned by no chapter, so it is reachable only through the notes that mention it in passing, and the one link already pointing at it resolves to nothing.
- The multiclass and multilabel metrics are thin. What exists covers the averaging arguments (`macro`, `micro`, `weighted`, `samples`) and stops there, with nothing on the measures native to a $K \times K$ matrix rather than built by averaging binary ones.
- Calibration of predicted probabilities appears nowhere, although the gap between a score and a probability is raised every time a threshold is set. A classifier is calibrated when its predicted probabilities match observed frequencies, and ranking and calibration are independent, so [[Log Loss]] separates two models that a [[ROC Curve]] cannot. [[Softmax Regression]] is where the gap already shows on this list, since regularization flattens the probabilities it returns without changing which class wins. Neither the measurement nor the repair has a note.
- Nothing here yet on what to do when the confusions [[Error Analysis]] surfaces are structural rather than fixable by more data, which is where a model that builds in the invariance replaces one that learns it from [[Data Augmentation]]. That argument belongs to the deep learning chapters and no note carries it.
