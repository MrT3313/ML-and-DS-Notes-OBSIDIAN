---
note_kind: concept
aliases:
  - perturbation
  - perturbations
  - adversarial example
  - adversarial examples
  - adversarial attack
  - adversarial attacks
  - adversarial training
  - perturbation-based method
up: "[[Data Augmentation]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

A perturbation is a small change added on purpose to a [[Training Instance]], sized so that the instance's label is still the right one afterwards. That makes it label preserving by intent, which is what puts it inside the [[Data Augmentation]] family rather than beside it. The word "noise" is used for it constantly and it is the third sense of that word now live in this vault: here noise is a signal somebody added deliberately, as against data that arrived wrong, which is [[Poor-Quality Data]], and as against the small-sample luck that [[Nonrepresentative Training Data]] calls sampling noise. A reader who arrived looking for either of those two senses is in the wrong note and should follow the link.

What earns the operation a note of its own rather than a line in the augmentation list is that it has two opposite users. Whoever is fitting the model uses the perturbed copy to teach the model where its [[Decision Boundary]] runs too close to real data. An attacker uses the same small change in the other direction, searching for the one that flips the model's answer while the true label plainly does not change. That second use is the adversarial attack, and the instance it produces is an adversarial example.

## Formal statement

Fix a model $h$, an instance $\mathbf{x}$ with true label $y$, a norm $\|\cdot\|_p$ from the [[Lp Norm]] family, and a **perturbation budget** $\rho > 0$, the radius of the ball the change is allowed to live in. The literature writes that budget $\epsilon$; this vault cannot, because $\epsilon$ is already the [[Tolerance]] and $\varepsilon$ is already the irreducible noise term, so $\rho$ carries it here and means nothing else.

A perturbed instance is

$$\mathbf{x}' = \mathbf{x} + \boldsymbol\delta, \qquad \|\boldsymbol\delta\|_p \le \rho$$

and the budget is what carries the whole label-preservation assumption. It is the formal stand-in for "small enough that the label survives", and it is a claim about the world that no part of the formula can check. Pick $\rho$ too large and the constraint is satisfied by copies whose labels are lies, which is the same failure augmentation has when a $6$ is rotated into a $9$.

$\mathbf{x}'$ is an **adversarial example** when it sits inside that ball and still changes the answer,

$$\|\boldsymbol\delta\|_p \le \rho \quad \text{and} \quad h(\mathbf{x}') \ne y$$

and the search for one is the constrained maximization of the loss over the ball,

$$\boldsymbol\delta^{*} = \arg\max_{\|\boldsymbol\delta\|_p \le \rho} \; \ell\big(h(\mathbf{x} + \boldsymbol\delta), \, y\big)$$

Adversarial examples were first reported by Szegedy, Zaremba, Sutskever, Bruna, Erhan, Goodfellow and Fergus (2013), who solved a box-constrained version of that search and found two things worth separating. The perturbation can be small enough that a person does not see it and the network still changes its label; and the same perturbation transfers, fooling a different network trained on a different subset of the data, which rules out the easy explanation that it is one model's private quirk.

### The one-step instance

Linearize the loss around $\mathbf{x}$ and the maximization has a closed form under an $\ell_\infty$ budget, since the best a bounded step can do against a linear function is move every coordinate to the edge of its allowance in the direction the gradient points. That is the **fast gradient sign method** of Goodfellow, Shlens and Szegedy (2014):

$$\mathbf{x}' = \mathbf{x} + \rho \cdot \mathrm{sign}\big(\nabla_{\mathbf{x}} \, \ell(h(\mathbf{x}), y)\big)$$

One backward pass and it is done, which is the point of it. Reading the magnitudes off that formula is what gives those authors their explanation for why models are so easy to fool: every one of the $n$ coordinates moves by exactly $\rho$, so the step has $\ell_\infty$ length $\rho$ and $\ell_2$ length $\rho\sqrt{n}$, and the change it produces in a linear score is $\rho \sum_j |\theta_j|$, which grows with the number of features while the individual change per pixel stays beneath notice. Their claim, against the intuition of the time, is that the vulnerability comes from models being too *linear* rather than too nonlinear.

Taking that step repeatedly and projecting back into the ball after each one is projected gradient descent, a slower and considerably stronger attack than the single step.

### Adversarial training, which is the defensive use

Turn the attack into the inner loop of the fit and the defence writes itself as a saddle point problem. Madry, Makelov, Schmidt, Tsipras and Vladu (2017) state it as minimizing, over the parameters, the expected *worst case* loss inside the ball:

$$\min_{\boldsymbol\theta} \; \mathbb{E}_{(\mathbf{x}, y) \sim p} \Big[ \max_{\|\boldsymbol\delta\|_p \le \rho} \; \ell\big(h_{\boldsymbol\theta}(\mathbf{x} + \boldsymbol\delta), \, y\big) \Big]$$

The inner maximum is approximated by projected gradient descent on the input, the outer minimum by ordinary gradient descent on the parameters, evaluated at the perturbed points rather than the original ones. This is what makes "helps the model learn weak spots in its decision boundary" a checkable statement rather than a slogan: for every instance, the inner problem finds the point in its neighbourhood the model handles worst, and the outer problem charges the fit for *that* point instead of for the instance. The gradient the optimizer follows is therefore always aimed at whichever piece of boundary currently runs closest to real data.

### What this buys and what it does not

- **Robustness is bought against one threat model, not against attacks in general.** The guarantee is indexed by the pair $(p, \rho)$ the fit was run against. Nothing in the min-max objective says anything about a larger $\rho$, a different norm, or a change that is not additive in the input at all, which covers rotations, crops, and anything done to the physical object before the sensor sees it. Madry and co-authors are explicit that what they offer is security against a *first-order* adversary, which names the class rather than promising all of them.
- **Clean accuracy pays for it.** Tsipras, Santurkar, Engstrom, Turner and Madry (2019) show the trade is not an artefact of an under-tuned optimizer: in a simple data model they construct, they prove that no classifier can be simultaneously maximally accurate and robust, and the same tension appears empirically on real datasets. Robust accuracy rises and standard accuracy falls, and the exchange rate is not something the practitioner gets to opt out of.
- **Nothing above certifies anything.** The inner maximization is solved approximately, so a model that survives the attack you ran has been shown to survive that attack and nothing more.

Adversarial examples at depth are owed by no chapter in either book on this vault's current reading, so the treatment stops at the definition and the two uses above. Certified defences, the attack taxonomy and the transfer literature are not covered here.

## VS

The vault owner's question, written against the perturbation-based semi-supervised method, was whether it is not simply data augmentation. The honest answer is neither yes nor no.

**The operation really is identical.** Both apply a small change to an instance under the assumption that the label should survive it, and nothing about $\boldsymbol\delta$ itself says which method is being run. What differs is what is done with the result, and that difference is not cosmetic.

- In [[Data Augmentation]], the perturbed copy is **added to the training set carrying the original's label**. $(\mathbf{x}, y)$ goes in and $(\mathbf{x} + \boldsymbol\delta, y)$ comes out beside it. So the method needs a label before it can start, and what it produces is more supervision: with $k$ perturbed copies per original, the labelled set grows from $m$ rows to $(k+1)m$.
- In the perturbation-based semi-supervised method ([[Semi-Supervised Learning]]), the perturbed copy is **never labelled at all**, and the instance it came from need not have a label either. The assumption that the label should not change is used as a *constraint on an unlabelled instance* rather than as permission to copy a label onto a new row. It enters the objective as a consistency penalty between the model's answer at the instance and its answer at the perturbed instance,

$$J \;=\; \frac{1}{|D_L|}\sum_{(\mathbf{x},\, y) \,\in\, D_L} \ell\big(h(\mathbf{x}),\, y\big) \;+\; \alpha \, \frac{1}{|D_U|}\sum_{\mathbf{x} \,\in\, D_U} d\Big(h(\mathbf{x}), \; h(\mathbf{x} + \boldsymbol\delta)\Big)$$

  with $d$ a divergence between two outputs and $\alpha$ the [[Regularization]] strength, written $\alpha$ here for the reason every other penalty weight in this vault is. There is no $y$ anywhere in that second term. [[Semi-Supervised Learning]] carries the variants and what each one picks $\boldsymbol\delta$ to be. What grows is not the labelled set but the number of places the fitted function is pinned down: each unlabelled instance contributes one neighbourhood over which $h$ is required to be flat.

**So: one enlarges supervision, the other propagates it.** Augmentation manufactures new labelled pairs out of pairs you already had, and spends them on the same fit you were already running. The semi-supervised method manufactures no pairs at all; it spends the unlabelled data on the *shape* of $h$, flattening it over small neighbourhoods so that the few labels you do have reach further across the input space than they otherwise could. Same operation, two purposes, and the purpose is what decides which one you are doing.

The case that makes the question better than it looks is the one where the two nearly collapse together and still stay apart. **Virtual adversarial training** (Miyato, Maeda, Koyama and Ishii) picks $\boldsymbol\delta$ by the adversarial search from the section above, taking the direction that most changes the model's own predicted distribution,

$$\boldsymbol\delta_{\text{vadv}} = \arg\max_{\|\boldsymbol\delta\|_2 \le \rho} \; D_{\mathrm{KL}}\Big(p(y \mid \mathbf{x}; \hat{\boldsymbol\theta}) \;\big\|\; p(y \mid \mathbf{x} + \boldsymbol\delta; \hat{\boldsymbol\theta})\Big)$$

and then uses that perturbation for consistency instead of for attack, penalizing the divergence it found. The perturbation is adversarial; the objective is semi-supervised; the true label is consulted at no point in either step, which is precisely why the method runs on unlabelled data at all. The operation is shared right down to how $\boldsymbol\delta$ is chosen, and the two methods are still doing different things with it.

One further difference is a tendency and not a rule, so it settles nothing on its own: augmentation usually *samples* $\boldsymbol\delta$ at random, while the adversarial and consistency uses usually *search* for it. Virtual adversarial training is on the searching side and is semi-supervised, so the search is not what makes the distinction either.

## Where it is used

[[Data Augmentation]] is the family this sits in, and perturbation is the member of it whose copies are also what an attacker builds, which is why the adversarial material lands here rather than there. [[Decision Boundary]] is what an adversarial example is evidence about: a step of norm at most $\rho$ that changes the prediction proves the boundary passes within $\rho$ of a genuine instance, so the size of the smallest successful $\boldsymbol\delta$ is a direct measurement of how close the boundary runs. [[Semi-Supervised Learning]] is the other use of the same operation, spending it on unlabelled instances as a constraint rather than on labelled ones as a copy, which is the contrast set out above. The same operation is also run as a measurement rather than as training data, where a perturbed copy of the [[Testing Set]] is scored and the fall in the score is the reading, which is the perturbation test of [[Behavioral Testing]]: there the perturbed split is scored, and here the perturbed instance is manufactured to be trained on.

[[Regularization]] is the category adversarial training falls into under that note's own test, since the min-max objective is a constraint imposed on purpose that is *meant* to cost clean accuracy in exchange for a smaller gap on perturbed inputs; the consistency penalty in the previous section is more literally so, being a term added to the [[Cost Function]] with $\alpha$ in front of it. [[Generalization]] is what both uses are aimed at and is also where the limit shows honestly: robustness inside one ball is not the same quantity as expected loss on the deployment distribution, and the Tsipras result above is the case where the two move in opposite directions.
