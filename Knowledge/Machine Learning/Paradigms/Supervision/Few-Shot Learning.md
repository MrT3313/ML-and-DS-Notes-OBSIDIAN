---
note_kind: concept
aliases:
  - few-shot
  - few shot learning
  - FSL
  - one-shot learning
  - N-way K-shot
up: "[[Machine Learning]]"
sources:
  - "[[DMLS Ch01 Overview of Machine Learning Systems]]"
confidence: draft
---

## Definition

Few-shot learning is the setting where a model has to recognise a class from a small number of labelled examples of it, conventionally one to five, instead of the hundreds or thousands an ordinary supervised fit consumes. The usual route to it is to train over many small tasks rather than over one fixed label space, so what the model acquires is the ability to adapt to an unfamiliar set of classes rather than a decision rule for a particular set.

## VS

[[Zero-Shot Learning]] is the same axis at its endpoint: there the count of labelled target examples is zero and a description of each class has to stand in for them, whereas here actual labelled instances exist and no side information is normally needed. That difference is one of kind and not only of degree, which is why the two are separate notes rather than two readings on one dial. Larochelle, Erhan and Bengio (2008) reconcile them neatly by treating one-shot learning as the case of zero-data learning "where the description of a class is a prototypical sample from that class".

The rest of this folder sorts on a different question, and saying so is more useful than forcing the analogy. [[Supervised Learning]], [[Semi-Supervised Learning]] and [[Unsupervised Learning]] are separated by what fraction of the training instances carry labels, a property of the training set as a whole. Few-shot learning is separated by how many labelled examples exist for the classes you want predictions on, a property of one region of the label space. The two questions are independent: every instance in a few-shot episode is labelled, so on the folder's axis the setting is fully supervised, and what is scarce is coverage of the target classes rather than supervision. [[Semi-Supervised Learning]] is the nearest-looking sibling and is still a different thing, since its scarcity is of labels against a large pool of unlabelled instances from the same classes, while few-shot scarcity is of instances of the class at all.

Against transfer in general: reusing what was learned on a source distribution to help on a target one is the broad case, and few-shot learning is the version where the target contributes only a handful of labelled examples. Fine-tuning a pretrained model on a small labelled set is squarely transfer, and it becomes few-shot only once that set is small enough for the count per class to be the binding constraint.

## Formal statement

**The episode.** An **N-way K-shot** episode fixes two numbers. $N$ is the count of distinct novel classes the model must discriminate in this episode; $K$ is the count of labelled examples supplied per class. The $N \times K$ labelled items form the **support set** $S = \{(\mathbf{x}^{(i)}, y^{(i)})\}_{i=1}^{NK}$, and a disjoint set of unlabelled items drawn from the same $N$ classes forms the **query set** $Q$, on which accuracy is reported. $K = 1$ is one-shot; $K$ of roughly 1 to 5 is the usual few-shot range. Setting $K = 0$ empties the support set, at which point the episode is unsolvable unless side information per class is supplied, which is [[Zero-Shot Learning]] rather than a smaller case of this one.

**Ordering hazard.** The letters are not standardised. Some papers write k-way n-shot, binding $k$ to classes and $n$ to examples, so before reading any reported number check which letter carries which quantity. A 5-way 1-shot result and a 1-way 5-shot result are not comparable and one of them is not even a classification problem.

**Episodic training.** Matching Networks (Vinyals et al. 2016) established the regime of training under the same conditions as the test, sampling a fresh support and query set from a fresh set of classes at every step, so that a new label space costs no fine-tuning. What is optimised over episodes is adaptation to a class set, not a mapping to one fixed set of labels.

**A concrete instance.** Prototypical Networks (Snell, Swersky and Zemel 2017) is the simplest thing that works and states the whole method in two lines. Embed the support items with $f_\phi$ and average per class to get a prototype

$$\mathbf{c}_k = \frac{1}{|S_k|} \sum_{(\mathbf{x}^{(i)}, y^{(i)}) \in S_k} f_\phi\big(\mathbf{x}^{(i)}\big)$$

then classify a query item by a softmax over negative distances to the prototypes

$$p_\phi(y = k \mid \mathbf{x}) = \frac{\exp\big({-d\big(f_\phi(\mathbf{x}), \mathbf{c}_k\big)}\big)}{\sum_{k'} \exp\big({-d\big(f_\phi(\mathbf{x}), \mathbf{c}_{k'}\big)}\big)}$$

with $d$ a distance in the embedding space. The $K$ of the episode appears in exactly one place, as $|S_k|$, the number of vectors the prototype averages, which is why the method degrades gracefully as $K$ shrinks and why the same equations extend to the zero-shot case by replacing $\mathbf{c}_k$ with an embedding of class meta-data.

**The prompting sense is a different claim.** Since Brown et al. (2020) "few-shot" is also used for a setting in which nothing is fitted at all: the model "is given a few demonstrations of the task at inference time as conditioning, but no weight updates are allowed", with $K$ typically in the range 10 to 100, bounded by the context window rather than by how many labels anyone could afford. The word is the same and the mechanism is not. A shot has migrated from a labelled example that produces a gradient step into an in-context demonstration that produces none, so a few-shot prompt trains nothing and leaves no artefact behind, and $K$ has moved an order of magnitude. This is the largest source of cross-talk between the two literatures, and a reported few-shot number means nothing until you know which sense it is in. Li and Flanigan (AAAI 2024) add a further caution about the prompting sense: on tasks where contamination of the pretraining corpus could be ruled out, models showed no statistically significant improvement over majority-class baselines in either the zero-shot or the few-shot setting, so some of what reads as few-shot ability is the task having been seen before.

**Not further quantitative at this depth.**

The N-way K-shot episode with its support and query sets is the exact parameterization of the setting, and it fixes what is supplied and what is measured rather than what any algorithm does with it. Sample-complexity results for learning from $K$ examples per class exist but attach to particular method families and assumptions on the embedding, not to the setting as such.

## Where it is used

It is one of the two answers to [[Insufficient Training Data]], the one for when a handful of labelled examples of the target classes does exist, against [[Zero-Shot Learning]] for when none does. It is also condition 6 of [[Machine Learning Applicability]] read from the human side: the requirement that a problem be repetitive is a statement about the gap between the few examples a person needs and the many an algorithm needs, and few-shot learning is the research programme aimed at closing it. The embedding $f_\phi$ that makes an episode solvable is normally pretrained before any episode is seen, and [[Self-Supervised Learning]] is the usual way to obtain it without paying for labels. Every formulation above is stated over [[Classification]], since an episode is defined by a set of $N$ classes to tell apart. And the query set is a [[Generalization]] measurement in miniature, held out from the support set exactly as a test set is held out from a [[Training Set]], with the difference that here the class set itself is new at every evaluation.

The family it joins on the supervision axis is [[Supervised Learning]], [[Semi-Supervised Learning]] and [[Unsupervised Learning]], with the caveat above about what that axis actually measures.

### On the child and the cat

The stock motivation, that a child shown a few pictures of cats recognises cats afterwards, is a rhetorical analogy rather than a result anyone established. The research it points at is Lake, Salakhutdinov and Tenenbaum's 2015 work on one-shot concept learning (*Science* 350(6266), 1332 to 1338), whose subject is handwritten characters from unfamiliar alphabets, whose model is a specific Bayesian program induction account, and whose comparison is against human participants on the Omniglot benchmark. That is a measured claim on a narrow domain. The cat is an illustration of it and carries no evidence of its own.
