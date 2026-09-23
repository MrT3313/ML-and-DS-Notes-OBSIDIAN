---
note_kind: concept
aliases:
  - inductive bias
  - inductive biases
  - learning bias
  - model assumptions
  - model assumption
  - no free lunch theorem
  - no free lunch
  - NFL theorem
  - IID
  - iid
  - i.i.d.
  - independent and identically distributed
  - smoothness assumption
  - prediction assumption
up: "[[Model]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

An inductive bias is whatever a learner uses to prefer one generalization over another beyond bare agreement with the training instances, and it is not the *bias* of the [[Bias-Variance Tradeoff]], which is one non-negative term inside a decomposition of expected squared error rather than a commitment the learner brought with it; a reader who arrived wanting that quantity wants that note. Read in the other direction it is the set of assumptions a [[Model]] family makes about the world before any data arrives, and every family carries some, whether or not whoever picked it can name them.

> [!quote] On assumptions
> ALL MODELS ARE WRONG BUT SOME ARE USEFUL ~George E. P. Box, 1979
>
> Since all models are wrong the scientist cannot obtain a "correct" one by excessive elaboration. ~George E. P. Box, 1976
>
> Essentially, all models are wrong, but some are useful. ~George E. P. Box and Norman R. Draper, 1987

Those three are not interchangeable and the familiar one is the last. "Science and Statistics" (*Journal of the American Statistical Association* 71(356), 1976, page 792) says only that all models are wrong and draws a consequence about elaboration from it. The aphorism itself first appears as a printed section heading, capitalized and with no comma, on page 202 of "Robustness in the Strategy of Scientific Model Building" (1979), over a passage whose argument is that the question to ask of a model is not "Is the model true?", to which the answer is no, but "Is the model illuminating and useful?". The comma and the "Essentially" that nearly every quotation carries were added in *Empirical Model-Building and Response Surfaces* (Box and Draper 1987, page 424).

## Formal statement

Fix a finite input space $\mathcal{X}$, a finite label set $\mathcal{Y}$, and a target $f : \mathcal{X} \to \mathcal{Y}$. A [[Training Set]] $D = \{(\mathbf{x}^i, y^i)\}_{i=1}^{m}$ pins $f$ down on the $m$ inputs it contains and says nothing whatever about the others. Write $X_D$ for those inputs and

$$V(D) \;=\; \big\{\, h : \mathcal{X} \to \mathcal{Y} \ \ \text{with} \ \ h(\mathbf{x}^i) = y^i \ \text{ for } i = 1, \dots, m \,\big\}$$

for the **version space**, the set of functions consistent with the data. An inductive bias is any restriction of $V(D)$, or any preference order over it, fixed before $D$ is seen. The two results below say what that object is worth: the first that a learner without one cannot generalize at all, the second that no particular one is better than any other in general.

### A learner with no bias cannot generalize

Take any unseen input $\mathbf{x} \notin X_D$ and any label $c \in \mathcal{Y}$. The function that agrees with $D$ on $X_D$ and returns $c$ at $\mathbf{x}$ is consistent with $D$, so it is in $V(D)$. Counting the consistent functions and the ones among them that commit to $c$ at $\mathbf{x}$,

$$|V(D)| \;=\; |\mathcal{Y}|^{\,|\mathcal{X}| - m}, \qquad \big|\{\, h \in V(D) \,:\, h(\mathbf{x}) = c \,\}\big| \;=\; |\mathcal{Y}|^{\,|\mathcal{X}| - m - 1}$$

so the version space splits into $|\mathcal{Y}|$ parts of exactly equal size over every unseen input, and it does so identically for every such input. A learner that commits to a label only where the consistent hypotheses agree therefore answers on $X_D$ and abstains everywhere else: it reproduces the training set and goes no further, which is rote lookup. Mitchell (1980), "The Need for Biases in Learning Generalizations", is the argument, and its force is that a bias is a precondition for the inductive leap rather than a stylistic preference. Every label a real learner emits on an unseen input is emitted because something other than the data broke that tie.

### No bias is better than any other on average

A bias is necessary, and none of them is free. Measure error only where the question is live, off the training inputs, under zero-one loss:

$$C(h, f) \;=\; \frac{1}{|\mathcal{X} \setminus X_D|} \sum_{\mathbf{x} \,\in\, \mathcal{X} \setminus X_D} \mathbb{1}\big[\, h(\mathbf{x}) \ne f(\mathbf{x}) \,\big]$$

and let a learning algorithm be any conditional distribution $P(h \mid D)$ over hypotheses, which covers deterministic learners as the degenerate case. Averaged uniformly over the targets that could have produced $D$,

$$\frac{1}{|V(D)|} \sum_{f \,\in\, V(D)} \; \mathbb{E}_{h \sim P(\cdot \mid D)} \big[\, C(h, f) \,\big] \;=\; 1 - \frac{1}{|\mathcal{Y}|}$$

and the right-hand side contains no reference to $P(\cdot \mid D)$ at all. The derivation is two lines: fix $h$ and an unseen $\mathbf{x}$, and under the uniform average $f(\mathbf{x})$ ranges over $\mathcal{Y}$ equally often by the counting above, so $h$ is wrong there in a fraction $1 - 1/|\mathcal{Y}|$ of the targets whatever $h$ happens to say; averaging over the unseen inputs and then over $h$ changes nothing, because the value being averaged was already constant.

This is the **no free lunch theorem** for supervised learning, due to Wolpert (1996), "The Lack of A Priori Distinctions Between Learning Algorithms", who states it over priors on targets rather than the single uniform average used here. Three consequences are worth separating.

- **Superiority is always relative to an assumed class of problems.** Any two algorithms $A$ and $B$ have the same average, so every class of targets on which $A$ beats $B$ is paid for by a class of exactly matching size on which $B$ beats $A$. "This model is better" is an incomplete sentence; the completion names the problems.
- **Nothing observed on the training set closes the gap.** The average is taken over targets that all agree with $D$ exactly, so a low training error, a small hypothesis class and a large $m$ do not by themselves buy off-training-set error. What buys it is the assumption that the real target is drawn from somewhere other than uniformly.
- **The uniform average is itself an assumption, and a strong one.** Real problems are not drawn uniformly from all functions of $\mathcal{X}$ into $\mathcal{Y}$; the ones anybody works on are overwhelmingly smooth, compositional and low in complexity. The theorem is not the claim that model choice does not matter. It is the claim that model choice can only matter through a commitment about which targets are plausible, which is exactly what an inductive bias is.

Wolpert and Macready (1997), "No Free Lunch Theorems for Optimization", prove the same shape of result for search: averaged over all objective functions, every algorithm produces the same distribution of observed values, so no optimizer is better than random enumeration in general either.

### The assumptions a model family actually makes

Seven are worth being able to name, because each is checkable against a candidate model and against the data about to be fitted to it rather than only agreeable to read.

| assumption | what it constrains | which families actually make it |
|---|---|---|
| **Prediction** | the problem, not the model: that $Y$ can be predicted from $X$ at all, so some $h$ beats the best constant guess | every supervised model without exception. It is condition 2 of [[Machine Learning Applicability]], and successive rolls of a fair die is the standing case where it fails, since the outcome depends on nothing observable |
| **IID** | how rows relate to each other and to production: each $(\mathbf{x}^i, y^i)$ drawn independently from one fixed joint distribution $p$, and $p$ the same at serving time | nearly all supervised learning, [[Linear Regression]], [[Logistic Regression]] and [[Softmax Regression]] included. Not a neural network assumption specifically, and it is what licenses the split at all |
| **Smoothness** | the function class: $f$ lies in a set of functions sending similar inputs to similar outputs | every supervised method that generalizes. [[Instance-Based Learning]] assumes it most nakedly, a nearest-neighbour prediction being the assumption executed rather than merely relied on, and [[Perturbation]] is the measurement of how locally false it is for a fitted model |
| **Tractability** | what the training objective can compute about a latent representation $\mathbf{z}$ of the input | latent-variable generative models, and not in the direction usually stated. Set out below |
| **Linear boundaries** | the shape of the [[Decision Boundary]]: every tie set between two classes is a hyperplane, so every decision region is convex | [[Logistic Regression]], [[Softmax Regression]] and the [[Stochastic Gradient Descent Classifier]]. Escaped without leaving the family by fitting the same linear model on a feature map, which is what [[Polynomial Regression]] and [[Feature Crossing]] supply |
| **Conditional independence** | the joint over features given the class factorizes, $P(x_1, \dots, x_n \mid y) = \prod_{j} P(x_j \mid y)$ | a naive Bayes classifier, which has no note here. [[Feature Crossing]] exists because the assumption fails: a crossed column is an interaction term handed to a model that cannot represent one |
| **Normality** | the shape of a distribution, either a feature column or a model's error term | not the fit. [[Linear Regression]] by least squares and the [[Normal Equation]] need no normality anywhere; the $t$ and $F$ intervals quoted around the coefficients do, and so does a Gaussian naive Bayes likelihood. [[Skewed Data]] measures the departure with $\gamma_1$ and [[Feature Distribution Transformation]] is the correction |

**What the IID assumption licenses.** Draw the held-out rows independently from the same $p$ as everything else, and for a hypothesis $h$ fixed before the draw the held-out average is an unbiased estimator of the quantity [[Generalization]] defines,

$$\mathbb{E}\big[\hat{\mathcal{L}}_{\text{test}}(h)\big] = \mathcal{L}_{\text{gen}}(h), \qquad \operatorname{Var}\big[\hat{\mathcal{L}}_{\text{test}}(h)\big] = \frac{\operatorname{Var}_p\big[\ell(h(\mathbf{x}), y)\big]}{m_{\text{test}}}$$

Independence and identical distribution are the only inputs to either statement, which is why this assumption is the load-bearing one: it is what makes a [[Testing Set]] a measurement rather than a second sample of the same rows. Each half fails in its own way and the failures have their own notes. Identical distribution fails across the train and serve boundary as [[Data Mismatch]], and at collection time as [[Nonrepresentative Training Data]]. Independence fails when rows come in groups, several readings per patient or near duplicates on both sides of the split, which [[Data Leakage]] carries as group leakage, and it fails by construction on time series, where [[Random Sampling]] is the wrong split. The fixed-in-advance $h$ is the third condition and its failure is [[Data Snooping Bias]].

### What is assumed about the latent posterior

A latent-variable generative model posits a joint $p_{\theta}(\mathbf{x}, \mathbf{z}) = p(\mathbf{z})\, p_{\theta}(\mathbf{x} \mid \mathbf{z})$ and wants the marginal likelihood of the data it can see:

$$p_{\theta}(\mathbf{x}) = \int p_{\theta}(\mathbf{x} \mid \mathbf{z})\, p(\mathbf{z}) \, d\mathbf{z}, \qquad p_{\theta}(\mathbf{z} \mid \mathbf{x}) = \frac{p_{\theta}(\mathbf{x}, \mathbf{z})}{p_{\theta}(\mathbf{x})}$$

so the posterior over the latent is tractable exactly when that integral is. For a Gaussian mixture or a hidden Markov model it is, and expectation-maximization uses the exact posterior in its E step. For a decoder $p_{\theta}(\mathbf{x} \mid \mathbf{z})$ that is a nonlinear function of $\mathbf{z}$ it is not, and that is the case the interesting models live in. Kingma and Welling (2014), "Auto-Encoding Variational Bayes", build the variational autoencoder on precisely that intractability: introduce a recognition model $q_{\phi}(\mathbf{z} \mid \mathbf{x})$ from a family that can be evaluated and sampled, and note the exact identity

$$\log p_{\theta}(\mathbf{x}) \;=\; \underbrace{\mathbb{E}_{q_{\phi}(\mathbf{z} \mid \mathbf{x})}\big[\log p_{\theta}(\mathbf{x} \mid \mathbf{z})\big] \;-\; D_{\mathrm{KL}}\big(q_{\phi}(\mathbf{z} \mid \mathbf{x}) \,\big\|\, p(\mathbf{z})\big)}_{\text{evidence lower bound } \mathcal{L}(\theta, \phi; \, \mathbf{x})} \;+\; D_{\mathrm{KL}}\big(q_{\phi}(\mathbf{z} \mid \mathbf{x}) \,\big\|\, p_{\theta}(\mathbf{z} \mid \mathbf{x})\big)$$

The final term is non-negative and cannot be computed, which is the whole reason $\mathcal{L}$ is a bound rather than the thing itself, and maximizing $\mathcal{L}$ is what the model does instead of maximizing the likelihood. So the assumption is not that the posterior is tractable. It is that the posterior is **approximable**: that some member of the chosen $q_{\phi}$ family sits close enough to $p_{\theta}(\mathbf{z} \mid \mathbf{x})$ in the sense of that last divergence for the bound to be worth optimizing. A universal claim over generative models fails a second time besides, since an autoregressive model factorizes $p(\mathbf{x}) = \prod_{j} p(x_j \mid x_{<j})$ and has no latent variable to have a posterior over, and a generative adversarial network never evaluates a likelihood at all. The variational family, the reparameterization used to differentiate through the sampling step, and the architectures that carry them all belong to a deep learning treatment that no note in this vault yet holds, so the assumption is corrected here rather than developed.

## Where it is used

[[Unreasonable Effectiveness of Data]] is the contest this is one side of: "Mind" in that argument means an inductive bias, and the two results above are what stop the mind side from being a taste, since without a bias there is no generalization and with the wrong one there is no advantage. [[Generalization]] is what a bias is spent to buy, and the no free lunch theorem is the statement that nobody gets it for nothing: the gap between training error and expected error is closed by an assumption or it is not closed.

The relation to the other sense of the word is a real one and not only a collision. A family's inductive bias is what fixes the squared-bias term of the [[Bias-Variance Tradeoff]] before any data arrives, since that term measures how far the family's average prediction sits from the truth and the family is exactly what the assumption chose. So a bias that matches the problem shows up as [[Underfitting]] avoided at no cost in variance, and one that does not is the bias-dominated regime with more data unable to help. [[Overfitting]] is the complementary failure, a family whose assumptions were too weak to pin anything down. [[Regularization]] is the case where the bias is explicit and tunable: a penalty on the weights is a stated preference among solutions the data cannot tell apart, which is Mitchell's definition with a dial on it.

[[Model Debugging]] is where the list becomes operational, its first named cause of failure being a model's assumptions violated by the data, which is this note read as a checklist. [[Model Selection]] is where the no free lunch theorem bites hardest, since it says the comparison is only ever valid over the class of problems the candidates were compared on. And [[Machine Learning Applicability]] owns the first row of the table as a gate on the whole project rather than on one model: if $Y$ cannot be predicted from $X$, no assumption anyone adds afterwards will recover it.
