---
note_kind: concept
aliases:
  - zero-shot
  - zero shot learning
  - zero-shot classification
  - ZSL
  - zero-data learning
up: "[[Machine Learning]]"
sources:
  - "[[DMLS Ch01 Overview of Machine Learning Systems]]"
confidence: draft
---

## Definition

Zero-shot learning asks a model to assign instances to classes for which it has never seen a single labelled example, scoring each candidate class through a supplied description of that class rather than through instances of it. The description is not an optional accessory: it is the channel the transfer runs through, and with nothing standing in for the missing labels the setting has no solution at all.

## VS

[[Few-Shot Learning]] hands the model a handful of labelled examples of each target class, so the class is learned from instances; zero-shot hands it none, which is exactly why a description of the class is mandatory here and usually absent there. The two are the same axis at different values, the number of labelled examples per target class, but they differ in kind at the endpoint rather than only in degree, because at zero the description has to do work no number of examples is doing.

The rest of this folder is sorted on a different question, and pretending otherwise would make the placement look neater than it is. [[Supervised Learning]], [[Semi-Supervised Learning]] and [[Unsupervised Learning]] are separated by what fraction of the training instances carry labels, which is a property of the training set taken as a whole. Zero-shot is separated by how many labelled examples exist for the classes you actually want predictions on, which is a property of one region of the label space. The two can be answered independently: a zero-shot classifier is normally fitted by ordinary [[Supervised Learning]] on fully labelled data for the seen classes, so on the folder's axis it sits at the supervised end while having zero labels where it counts. What is short is coverage, not supervision.

One more separation, because the word invites it. Reusing what a model learned on a source distribution or task to do better on a target one is the general case, and zero-shot learning is its limit: the target classes contribute zero labels and the only channel left is the shared semantic space. Every zero-shot method is a transfer method. The converse fails, since fine-tuning on a small labelled target set is transfer and is plainly not zero-shot.

## Formal statement

**The setting.** Let $Y_s$ be the seen classes and $Y_u$ the unseen classes, with $Y_s \cap Y_u = \varnothing$. Training data is $D = \{(\mathbf{x}^{(i)}, y^{(i)})\}_{i=1}^{m}$ with every $y^{(i)} \in Y_s$, and not one labelled instance of any $y \in Y_u$ is available at any point before evaluation. Lampert, Nickisch and Harmeling (CVPR 2009) state it as learning a classifier $f: X \to Z$ for a label set $Z$ disjoint from the training label set $Y$, and observe that an ordinary multiclass classifier cannot do this, since it learns one parameter vector per training class and there is none to learn for a class with no instances.

**The side information.** Every class in $Y_s \cup Y_u$ carries a representation $s(y)$ in a shared semantic space $S$, supplied from outside rather than estimated from target instances. Xian, Lampert, Schiele and Akata (TPAMI 2019) put the requirement plainly: "some form of side information is required to share information between classes so that the knowledge learned from seen classes is transfered to unseen classes". In practice $s(y)$ is a human-specified attribute vector, a word embedding, a position in a hierarchy such as WordNet, or a textual description of the class.

**What is learned.** A compatibility score $f(\mathbf{x}, s(y))$, usually factored as a learned mapping $\phi$ of the input into $S$ scored against the class representation, so that $f(\mathbf{x}, s(y)) = \langle \phi(\mathbf{x}), s(y) \rangle$ or a distance in $S$. Prediction in the classical setting is

$$\hat{y} = \arg\max_{y \in Y_u} f\big(\mathbf{x}, s(y)\big)$$

and this is the entire mechanism. The maximisation ranges over class descriptions rather than over per-class parameters fitted from data, so a class that was never trained on can still be scored: its description lives in the same space the input was mapped into. Prototypical networks make the substitution explicit, replacing the class prototype normally averaged from labelled examples with an embedding $g(\mathbf{v}_y)$ of a class meta-data vector, leaving the rest of the classifier untouched (Snell, Swersky and Zemel 2017).

**What the transfer is between.** Classes, not tasks. In the classical formulation the task is one fixed [[Classification]] problem, the seen and unseen classes belong to it alike, and the attribute space is engineered so that the two sets share coordinates. The Animals with Attributes benchmark introduced by Lampert et al. is the standing example: over 30,000 animal images across 50 classes, each class described by 85 semantic attributes, with training and test classes held disjoint. Relatedness there is a design decision about the attribute vocabulary, not a happy accident of which tasks were trained first.

**What is guaranteed.** Not much, and conditionally. Palatucci, Pomerleau, Hinton and Mitchell (NIPS 2009) named the modern term and gave the semantic output code classifier a PAC treatment, establishing conditions under which novel classes can be predicted accurately rather than a general result that they can. Xian et al. also define **generalized zero-shot learning**, in which the search space at evaluation includes $Y_s$ as well as $Y_u$, and note that restricting test instances to unseen classes has been criticised as a restrictive setup. The generalized numbers are markedly worse, because the seen classes absorb the probability mass.

**The prompting sense is a different claim.** Since Brown et al. (2020) the term has been used for something structurally analogous and operationally distinct. There, zero-shot means the model "is only given a natural language instruction describing the task" with no demonstrations, and in every one of their settings the model "is applied without any gradient updates or fine-tuning". The instruction occupies the slot the attribute vector occupied, a description standing in for labelled data, which is why the same word fits. Two things do not carry over. Nothing is trained, so a "shot" has become an item in the context window rather than a gradient step. And the disjointness above, $Y_s \cap Y_u = \varnothing$, is an assertion nobody can check against a web-scale pretraining corpus: Li and Flanigan (AAAI 2024) found that datasets released before a model's training data was collected scored markedly better than those released after, and that on tasks where contamination was ruled out the models showed no statistically significant improvement over majority-class baselines in either the zero-shot or the few-shot setting. So zero-shot applied to a pretrained language model is a claim about what was in the prompt, not a verified claim about what was in the training data.

**Not further quantitative at this depth.**

The argmax above and the disjointness condition are the whole of what the setting itself fixes; everything numeric past that belongs to a particular compatibility function and its loss, which differ between the attribute, embedding and generative families. The accuracy bounds exist, in Palatucci et al., but they are stated per method under assumptions about the semantic space rather than as a property of zero-shot learning as such.

## Where it is used

It is one of the two answers to [[Insufficient Training Data]], the one for when the count of labelled examples for the target classes is zero rather than merely small, and a description of those classes is what stands in. It is therefore the standing exception in [[Machine Learning Applicability]] to the requirement that data for the task exist, and the exception is conditional rather than free: the side information has to exist and has to be good, which moves the cost rather than removing it. The mapping into the semantic space is normally carried by a model pretrained elsewhere, and [[Self-Supervised Learning]] is how such a model is usually obtained without paying for labels. It is stated over [[Classification]] in every formulation above, since the object being extended is a label set. And it is a strong test of [[Generalization]], which ordinarily assumes production data is drawn from the training distribution: here the classes at prediction time are by construction ones the training distribution never contained.

Its sibling on the same axis is [[Few-Shot Learning]]; the family it joins on the supervision axis is [[Supervised Learning]], [[Semi-Supervised Learning]] and [[Unsupervised Learning]], with the caveat about what the two axes actually measure set out above.
