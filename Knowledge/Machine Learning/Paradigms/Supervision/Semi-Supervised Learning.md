---
note_kind: concept
aliases:
  - semi-supervised
  - partially labeled learning
  - self-training
  - pseudo-labeling
  - pseudo-labelling
  - consistency regularization
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---
## Definition

Semi-supervised learning trains on a small labeled set together with a large unlabeled one. The unlabeled data reveals the shape of the distribution; the few labels name the parts of it.

## Formal statement

$D = D_L \cup D_U$ with $|D_L| \ll |D_U|$, where $D_L$ carries labels and $D_U$ does not. Most algorithms combine an unsupervised step on all of $D$ with a supervised step on $D_L$, and what joins the two steps is an objective. Two families of objective cover most of the field, and they differ in what they assume about the unlabeled data.

### Self-training with a confidence threshold

The older family, traceable to Scudder (*IEEE Transactions on Information Theory* 11(3), 1965), builds the untaught machine out of the taught one by feeding it its own output in place of a teacher. Fix a confidence threshold $\tau \in (0, 1)$ and iterate:

1. Fit $h$ on $D_L$.
2. Predict on $D_U$, obtaining a distribution $P(y_k \mid \mathbf{x})$ over classes for every unlabeled $\mathbf{x}$.
3. For each $\mathbf{x}$ whose top predicted probability clears the threshold, $\max_k P(y_k \mid \mathbf{x}) \geq \tau$, attach the **pseudo-label** $\hat{y} = \arg\max_k P(y_k \mid \mathbf{x})$ and move $(\mathbf{x}, \hat{y})$ from $D_U$ into the labeled set.
4. Refit on the enlarged labeled set and repeat, stopping when no instance clears $\tau$ or a round budget runs out.

$\tau$ is what makes this an algorithm instead of a slogan, and naming it is the point: "add the predictions you are confident about" fixes no procedure until the bar is written down. The modern statement of the same rule makes it a mask on the unlabeled loss rather than a transfer between sets. Sohn and colleagues (2020) write it as

$$\ell_u = \frac{1}{\mu B} \sum_{b=1}^{\mu B} \mathbb{1}\big(\max(q_b) \geq \tau\big)\, H\big(\hat{q}_b,\, q_b\big)$$

with $q_b$ the model's predicted distribution for unlabeled instance $b$, $\hat{q}_b = \arg\max(q_b)$ the hard pseudo-label, $H$ cross-entropy, and the indicator $\mathbb{1}$ zeroing out every instance below the bar. The total objective is the supervised loss on the labeled batch plus $\lambda_u \ell_u$.

**The failure it invites.** A pseudo-label above threshold re-enters training as if it were ground truth, so a confident error becomes a training target, the refit makes the model more confident in it, and the mistake compounds over rounds. Nothing in the loop can catch it, because the only evidence available is the model's own output, which is the thing that went wrong. This is **confirmation bias**, and it is a recognised named failure rather than a hypothetical: it is why $\tau$ is a real trade and not a formality. A high $\tau$ admits fewer pseudo-labels of better average quality; a low $\tau$ admits far more at worse quality, so the parameter trades the quantity of unlabeled data reaching the loss against its correctness. Sohn and colleagues measure the curve directly: $\tau = 0.95$ gave the lowest error on their benchmark, raising it to $0.97$ or $0.99$ cost little, and small values cost more than one and a half accuracy points.

### The consistency assumption under perturbation

The second family assumes something about the data rather than about the model's confidence: that a small perturbation to an instance should not change its label, so the model's output ought to be stable under one. Nothing about that assumption needs a label to state, which is what lets it be enforced on $D_U$. Written as an objective, it adds to the supervised loss a term penalizing disagreement between the output on an instance and the output on a perturbed copy of it:

$$\mathcal{L} \;=\; \underbrace{\frac{1}{|D_L|} \sum_{(\mathbf{x},\, y) \,\in\, D_L} \ell\big(h(\mathbf{x}),\, y\big)}_{\text{supervised, on } D_L} \;+\; \alpha \underbrace{\frac{1}{|D_U|} \sum_{\mathbf{x} \,\in\, D_U} d\big(h(\mathbf{x}),\, h(\mathbf{x} + \delta)\big)}_{\text{consistency, on } D_U}$$

where $d$ measures distance between two outputs, squared $L_2$ in the original form and cross-entropy in later ones, and $\alpha$ sets how much the unlabeled term counts against the labeled one, written $\alpha$ rather than the literature's $\lambda$ because it is the weight on a penalty term and this vault reserves one symbol for that role throughout. Oliver and colleagues (2018) state the plain version as minimizing $d\big(f_\theta(\mathbf{x}), f_\theta(\hat{\mathbf{x}})\big)$ over the unlabeled data and adding it to the classification loss as a regularizer scaled by a weighting hyperparameter. In the original formulations $h$ is itself stochastic, under dropout or random augmentation, so both invocations are perturbed and the two terms differ even when the same instance goes into each.

What $\delta$ is, is where the variants part company: stochastic transformations and injected noise in the first proposals (Bachman and colleagues 2014, Sajjadi and colleagues 2016), the adversarial direction chosen to move the output most in Virtual Adversarial Training (Miyato and colleagues), and strong image augmentation in the recent methods, which pair a weak perturbation for producing the pseudo-label with a strong one for enforcing it.

**This is not [[Data Augmentation]], although the operation is identical.** Augmentation adds the perturbed copy to the training set carrying the original's label, while the consistency method leaves the instance unlabeled and uses the assumption as a constraint on the model's output; [[Perturbation]] carries the full answer.

## Where it is used

It sits between [[Supervised Learning]] and [[Unsupervised Learning]] on the supervision axis and is the practical answer to label cost. Typical pipelines run [[Clustering]] first and then propagate the few labels through the clusters, which is a third family alongside the two above and rests on its own assumption, that instances in one cluster share a label. It is also one of four standing answers to [[Insufficient Training Data]], and the one that needs a seed: [[Weak Supervision]] generates labels from heuristics and needs none, [[Transfer Learning]] imports a model fitted elsewhere, and [[Active Learning]] buys ground truth but chooses which instances are worth buying. What separates this one from the rest is that it manufactures its extra labels out of a structural assumption about the unlabeled pool, so the assumption is the thing to check when it fails.
