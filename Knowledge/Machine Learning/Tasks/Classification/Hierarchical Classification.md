---
note_kind: concept
aliases:
  - hierarchical classification
  - hierarchical classifier
  - hierarchical classifiers
  - hierarchical multiclass classification
up: "[[Classification]]"
sources:
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

Hierarchical classification helps when the class count is high. The label set is given a tree, each internal node holds a classifier over its own children, and a prediction is a path from the root down to a leaf: a first classifier sorts the example into a large group, a second sorts it into one of that group's subgroups, and so on until a leaf is reached. It is a way of decomposing one hard decision into several easy ones.

Product classification is the worked case. The first problem separates electronics, home and kitchen, fashion, pet supplies and the rest. For an item that landed in fashion, the second problem separates shoes, shirts, jeans and accessories. Note what supplied that tree: the catalogue already had it. The hierarchy is an input taken from the problem domain, not something the fit discovers, and that is the condition on when this decomposition is available at all. A label set with no natural grouping does not get one by wishing.

The arrangement described here, one multiclass classifier per parent node deciding among that node's children, is standardly called the **local classifier per parent node** approach. Silla and Freitas's survey of hierarchical classification (*Data Mining and Knowledge Discovery*, 2011) is where the vocabulary is unified, and it names three local arrangements and one non-local alternative: **local classifier per node**, one binary classifier per node in the tree answering "does this example belong under me?"; **local classifier per parent node**, the one above; **local classifier per level**, one multiclass classifier per level of the tree rather than per node; and the **global** or **big-bang** approach, a single classifier fitted over the entire hierarchy at once, which gives up the modularity the local arrangements are built on. The survey's name for ignoring the tree and fitting one flat $K$-way model is **flat classification**.

## Formal statement

### Classifier count

Take a balanced tree of branching factor $b$ and depth $d$, so the leaves are the classes and

$$K = b^{d}$$

The classifiers sit at the internal nodes, which are the nodes at levels $0$ through $d-1$; the leaves hold no classifier because there is nothing left to decide once one is reached. Level $i$ holds $b^{i}$ nodes, so the count is a geometric sum:

$$\sum_{i=0}^{d-1} b^{i} = \frac{b^{d} - 1}{b - 1}$$

Each of those is a $b$-way problem. The flat arrangement is the comparison: one classifier, one $K$-way problem.

That states exactly what is bought and what is paid. Bought: every decision is $b$-way instead of $K$-way, and the training set a node sees shrinks with depth, since a node at level $i$ is only ever handed the rows whose label falls in its own subtree, roughly $m / b^{i}$ of the $m$ rows. Paid: $d$ decisions per prediction instead of one, and $\frac{b^{d}-1}{b-1}$ fitted models instead of one.

Work the product example as a two-level taxonomy with $b = 10$ and $d = 2$, so $K = 100$ leaf categories. The tree needs $\frac{100 - 1}{9} = 11$ classifiers: one at the root over the ten coarse groups, and one inside each group over its ten leaves. Every one of them is a ten-way problem, and none of them is a hundred-way problem.

Against the flat decompositions the vault already owns, at $K = 100$: [[One-versus-Rest]] trains $K = 100$ classifiers and [[One-versus-One]] trains $\frac{K(K-1)}{2} = 4950$. The hierarchy trains $11$. For $K = b^{d}$ the node count $\frac{b^{d}-1}{b-1}$ is below both as soon as $b$ is small relative to $K$, which is the whole point of making the tree deep rather than wide.

The honest qualification is that those numbers count different things. The $11$ nodes hold $b$-way problems, and the $100$ and $4950$ count binary problems, so with a binary-only base algorithm each node decomposes again: $11 \times 10 = 110$ binary classifiers if every node uses one-versus-rest, or $11 \times 45 = 495$ if every node uses one-versus-one. Compared like that the hierarchy still crushes flat one-versus-one, $495$ against $4950$, but it is slightly worse than flat one-versus-rest on classifier count alone, $110$ against $100$. What it wins there is not the count but the size of each fit, since a flat one-versus-rest classifier is handed all $m$ rows and a level-one node classifier is handed about $m/10$.

### Error propagation down the path

A prediction is correct only if every decision on the path is correct, which is where the decomposition charges its real price. Let each level be correct with probability $p$, and treat the levels as independent. Then

$$P(\text{leaf correct}) = p^{d}$$

Accuracy decays geometrically in depth. At $p = 0.95$, a two-level tree lands at $0.95^{2} \approx 0.90$ and a four-level tree at $0.95^{4} \approx 0.81$, so a tree made deeper to keep $b$ small pays for that in the exponent.

The asymmetry is the part worth acting on. An error high in the tree is unrecoverable: once the root sends an example into the wrong subtree, the subtree holding its true class is never entered, and every classifier below is irrelevant to that example no matter how good it is. An error at the last level costs one sibling confusion. So the levels are not equally valuable, and the top of the tree is where accuracy is worth buying. That is the falsifiable claim here: raising $p$ at the root moves $p^{d}$ more than raising $p$ at a leaf's parent, and the two are measurable separately by scoring each level's classifier on its own.

Two caveats attach to the formula. The independence assumption is an approximation, since an example that is hard for the root is usually hard for the node below it too, and correlated errors make the true path accuracy differ from $p^{d}$. And a single $p$ across levels is a simplification; with per-level accuracies $p_{1}, \dots, p_{d}$ the product is $\prod_{i=1}^{d} p_{i}$ and the geometric decay is the equal-accuracy case of it. Silla and Freitas state the problem descriptively, that an error at a given class level is propagated downwards the hierarchy unless some procedure for avoiding it is used, and give no separate name for it; the named problems adjacent to it in that survey are different things. **Blocking** is what happens when a confidence threshold stops an example from being passed further down, which trades an unrecoverable error for a less specific answer, and **class-membership inconsistency** is a local classifier per node or per level arrangement predicting classes at different levels that cannot both hold. Neither is the same as propagation, and the per-parent-node arrangement above avoids inconsistency by construction, because a node only ever chooses among its own children.

### High cardinality in the label space

When the number of classes is high, the task is **high cardinality**, and that is the condition under which the tree is worth its cost. At small $K$ a flat model is simpler, fits once, and cannot propagate an error it never makes. The cost of the hierarchy, many fitted models and a $p^{d}$ ceiling, is only worth paying when the flat $K$-way decision is the thing that is failing.

The word `cardinality` carries a second and unrelated sense in this vault, and the two are worth keeping apart. In [[One-Hot Encoding]] and [[Feature Engineering]] it counts the distinct values a categorical **feature** takes, and high cardinality there is a problem about how wide the encoded input becomes: a ZIP code column with thousands of levels becomes thousands of near-empty columns. Here it counts the values the **label** takes, and high cardinality is a problem about how many decisions the model has to make. Same word, two different problems, and neither note claims the bare word as a name for itself.

### In scikit-learn

scikit-learn 1.6 ships no hierarchical classifier. `sklearn.multiclass` holds `OneVsRestClassifier`, `OneVsOneClassifier` and `OutputCodeClassifier`, all of them flat, and nothing in the library takes a class tree as an argument. A caller who wants this fits the nodes by hand: one ordinary estimator per internal node, each trained only on the rows whose label falls in that node's subtree, and prediction chained from the root down.

```python
import numpy as np
from sklearn.linear_model import LogisticRegression

# y_coarse and y_fine are the two levels of the catalogue's own taxonomy,
# supplied with the data rather than learned from it
root_clf = LogisticRegression().fit(X_train, y_coarse)

leaf_clf = {}
for group in np.unique(y_coarse):
    rows = (y_coarse == group)                  # this node only ever sees its own subtree
    leaf_clf[group] = LogisticRegression().fit(X_train[rows], y_fine[rows])

groups = root_clf.predict(X_new)                # first decision, unrecoverable if wrong
preds = np.array([leaf_clf[g].predict(x.reshape(1, -1))[0]
                  for g, x in zip(groups, X_new)])
```

Each fitted node obeys the [[Scikit-Learn Estimator API]] like any other estimator, so the nodes compose with [[Pipeline]] and are cross-validated individually. What the hand-rolled version does not get for free is hierarchical scoring, since `accuracy_score` on the leaves treats a neighbouring-leaf error and a wrong-subtree error as the same miss. `hiclass`, a scikit-learn-contrib package, supplies all three local arrangements as estimators, `LocalClassifierPerParentNode`, `LocalClassifierPerNode` and `LocalClassifierPerLevel`, each taking any scikit-learn classifier as its base estimator, along with metrics that read the tree.

## Where it is used

[[Multiclass Classification]] is the task this decomposes, and it is the note that owns $K$ and the sum rule over the classes; hierarchical classification changes how the one-of-$K$ decision is reached and not what is being decided. [[Classification]] is the parent task, and this sits in it as an arrangement of classifiers rather than as a classifier.

[[One-versus-Rest]] and [[One-versus-One]] are the flat decompositions this is the alternative to, compared above by classifier count and by how much of the training set each fit sees; all three are meta-estimators that classify nothing themselves. [[Binary Classification]] is what every node reduces to once a binary-only base algorithm is used, since each $b$-way node problem is itself decomposed.

[[One-Hot Encoding]] is where the other sense of cardinality lives, counting the levels of a feature rather than the classes of a label, which is why the bare word is ambiguous across the vault and is disambiguated in both notes. [[Machine Learning Systems Design]] is where the tree comes from, since a class taxonomy is a requirement taken from the business domain and fixed before any fitting, which makes it a design input rather than a modelling choice.
