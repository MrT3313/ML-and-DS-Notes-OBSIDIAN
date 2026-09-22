---
note_kind: concept
aliases:
  - transfer learning
  - transfer
  - fine-tuning
  - fine tuning
  - finetuning
  - pretrained model
  - pretraining
  - source task
  - target task
up: "[[Machine Learning]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## Definition

Transfer learning reuses a model developed for one task as the starting point for a model on a second one. The first task, the source, is normally picked because cheap and abundant training data exists for it; the second, the target, is the one you actually care about, and what crosses over is whatever the source model learned that both tasks have in common rather than the source model's predictions.

Two claims about when to reach for it, and the second is the one that usually gets dropped. It is the obvious move when the target has little labelled data, because the target then pays only for adaptation instead of paying to learn a representation from nothing. It is also worth doing when the target has plenty: starting from a pretrained model frequently beats training the same architecture from scratch, which makes transfer a default rather than a fallback for the data-poor.

## VS

Against [[Zero-Shot Learning]]. That setting is this one at the endpoint of a single quantity, how many labelled examples of the target classes the target contributes. At zero there is nothing left to fit the target function against, so the only channel the source can reach the target through is a supplied description of each target class, scored in a space the source model already maps into. The description is not an extra ingredient zero-shot adds; it is the substitute for the supervision it has given up, which is why it is mandatory there and optional here.

Against [[Few-Shot Learning]]. Move a short way up the same quantity and the target contributes a handful of labelled examples per class. Fine-tuning a pretrained model on a set that small is a transfer setting distinguished by how much target supervision it has, not by a different mechanism. The containment in the other direction is worth stating carefully rather than asserting, because few-shot learning has a second route into it: a model trained episodically over many sampled class sets, with the test classes held disjoint from the training ones, satisfies the formal condition below through $\mathcal{Y}_S \neq \mathcal{Y}_T$ while reusing nothing anyone would call a pretrained model. What the condition does not capture is the difference in mechanism, and that difference is what makes [[Few-Shot Learning]] a separate note: its question is what makes a class learnable from $K$ examples, and this note's question is what makes knowledge from one problem usable on another at all.

Against [[Self-Supervised Learning]]. Not a rival. Manufacturing labels by hiding part of each instance produces a task with effectively unlimited cheap data, which is exactly the condition a source task is chosen for, so self-supervision is one recipe for obtaining the model that transfer then reuses. Ordinary [[Supervised Learning]] on a large labelled corpus is the other recipe. A pretrained model is the object; neither recipe is a competitor to what is done with it afterwards.

All three of those sort on one quantity, how much labelled data the target classes contribute, whereas [[Supervised Learning]], [[Semi-Supervised Learning]] and [[Unsupervised Learning]] sort on another, what fraction of the training instances carry labels at all. The two quantities are independent and every setting takes a value on both: a fine-tuning run is fully supervised on its source and on the few target labels it has, and is still short of coverage where it counts.

## Formal statement

**Domains and tasks.** Pan and Yang (IEEE Transactions on Knowledge and Data Engineering 22(10), 2010) give the formalization that makes the term checkable. A **domain** is a pair

$$\mathcal{D} = \{\mathcal{X}, P(X)\}$$

a feature space $\mathcal{X}$ together with a marginal distribution $P(X)$ over samples $X = \{\mathbf{x}_1, \dots, \mathbf{x}_n\} \in \mathcal{X}$. Given a domain, a **task** is a pair

$$\mathcal{T} = \{\mathcal{Y}, f(\cdot)\}$$

a label space $\mathcal{Y}$ and an objective predictive function $f$, which is never observed and has to be learned from pairs $(\mathbf{x}_i, y_i)$ with $\mathbf{x}_i \in \mathcal{X}$ and $y_i \in \mathcal{Y}$. Read probabilistically, $f(\mathbf{x})$ is $P(y \mid \mathbf{x})$, so a task can equally be written $\{\mathcal{Y}, P(Y \mid X)\}$, and that second reading is what makes the case analysis below possible.

**The condition.** Given a source domain and task $(\mathcal{D}_S, \mathcal{T}_S)$ and a target domain and task $(\mathcal{D}_T, \mathcal{T}_T)$, transfer learning aims to improve the learning of the target predictive function $f_T$ in $\mathcal{D}_T$ using knowledge from $\mathcal{D}_S$ and $\mathcal{T}_S$, where

$$\mathcal{D}_S \neq \mathcal{D}_T \quad \text{or} \quad \mathcal{T}_S \neq \mathcal{T}_T$$

That inequality is the whole of the definition, and it is what stops the term from meaning nothing. If both hold with equality, source and target are the same problem and there is only [[Supervised Learning]] on more data: nothing is transferred because there is no gap to transfer across.

**What the inequality can mean.** Each pair has two components, so the condition decomposes into four cases, and which one holds is what separates the named sub-settings from each other.

| what differs | condition | reading |
|---|---|---|
| feature space | $\mathcal{X}_S \neq \mathcal{X}_T$ | the two domains are not even described in the same coordinates, as with source documents in one language and target documents in another |
| marginal | $P(X_S) \neq P(X_T)$ | same coordinates, different distribution over them, as with source and target documents on different topics |
| label space | $\mathcal{Y}_S \neq \mathcal{Y}_T$ | the source predicts two classes and the target predicts ten, or a disjoint set of them |
| conditional | $P(Y_S \mid X_S) \neq P(Y_T \mid X_T)$ | same labels, different rule mapping inputs to them, including class proportions that differ sharply between source and target |

The first two are the ways a **domain** differs and the last two are the ways a **task** differs, and the split is load bearing rather than tidy: a target task that differs requires some labelled target data to fit $f_T$ against, while a difference confined to the marginal can in principle be handled with no target labels at all. So the four cases predict the supervision budget, not just the vocabulary.

**What is guaranteed.** Nothing, and this is the part the enthusiasm around pretrained models routinely omits. There is no result saying the source helps. The named failure is **negative transfer**, which Pan and Yang define as the case where the source domain data and task contribute to reduced performance on the target, leaving the learner worse off than it would have been with no transfer at all. Their framing of the precondition is the useful part: most transfer methods work by implicitly assuming source and target are related, where two domains count as related when some relationship, explicit or implicit, holds between their feature spaces. Relatedness is therefore an assumption the method rests on and not a bonus when it happens to hold, and the empirical result they cite from Rosenstein and colleagues is that brute-force transfer between two sufficiently dissimilar tasks hurts the target. No procedure checks the assumption before you pay for the transfer, which is why "when to transfer" is a separate open question from "what to transfer" and "how to transfer".

One boundary worth marking, because the vocabulary here is slippery. **Domain adaptation** is the specific case in the second row of the table, same task and same feature space with a shifted marginal, and it is a sub-setting of transfer rather than a synonym for it. Calling every reuse of a pretrained model domain adaptation loses the distinction the table exists to make.

## Where it is used

It is one of four standing answers to [[Insufficient Training Data]], and the only one that does not try to produce the missing labels: the other three are [[Weak Supervision]], which writes noisy heuristics that emit labels, [[Semi-Supervised Learning]], which grows a small seed set using structural assumptions about the unlabelled pool, and [[Active Learning]], which still buys ground truth but spends the budget on the instances the model gains most from. Its two limiting cases, ordered by how much target supervision survives, are [[Few-Shot Learning]] at a handful of labelled examples per class and [[Zero-Shot Learning]] at none. [[Self-Supervised Learning]] is how the source model is most often obtained without paying for labels anywhere in the chain, which is what makes an abundant source task cheap rather than merely large. Against [[Data Augmentation]], the contrast is where the extra information comes from: augmentation manufactures more presentations of the data you already hold, while transfer imports a model fitted on data you do not hold and never will.
