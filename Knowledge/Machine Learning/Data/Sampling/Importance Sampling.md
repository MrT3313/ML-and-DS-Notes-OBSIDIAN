---
note_kind: method
aliases:
  - importance sampling
  - proposal distribution
  - importance distribution
  - importance weight
  - importance weights
up: "[[Sampling]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Estimate an expectation under a distribution $P$ using draws from a different distribution $Q$, and pay for the substitution by reweighting each draw. Reach for it whenever the distribution you have to answer about is not the distribution you can sample: $P$ is expensive, slow, or impossible to draw from; $P$ has changed since the data was collected; $P$ concentrates its interesting behaviour on a region a fair draw almost never reaches, which is the rare-event case; or the draws already exist and were produced by something other than $P$, which is the off-policy case in reinforcement learning.

This is the most consequential of the sampling methods here, and the least forgiving. It is a correction applied after the fact rather than a way of collecting better data, so it works when the correction is computable and fails quietly when it is not. The result is about distributions, not about models, and holds whether or not anything is ever fitted.

## Algorithm or formula

Take $\mu = \mathbb{E}_P[f(x)]$, the quantity wanted, with $P$ the **nominal** distribution and $f$ the integrand. Let $Q$ be the **proposal distribution**, also called the importance distribution, the one you can actually draw from. The identity is one line of multiplying by $Q(x)/Q(x)$:

$$\mathbb{E}_P[f(x)] = \int f(x)P(x)\,dx = \int f(x)\frac{P(x)}{Q(x)}Q(x)\,dx = \mathbb{E}_Q\!\left[f(x)\frac{P(x)}{Q(x)}\right]$$

so an expectation under $P$ has been rewritten as an expectation under $Q$ of a different function. The factor $w(x) = P(x)/Q(x)$ is the **importance weight**, or likelihood ratio: it says how much more often $P$ would have produced this $x$ than $Q$ did. The estimator follows directly,

$$\hat{\mu} = \frac{1}{n}\sum_{i=1}^{n} f(x_i)\,\frac{P(x_i)}{Q(x_i)}, \qquad x_i \sim Q$$

and it is unbiased, $\mathbb{E}_Q[\hat{\mu}] = \mu$, because each term has expectation $\mu$ by the identity above and the mean of $n$ such terms therefore does too.

**The condition on $Q$.** The identity needs $Q$ to put mass wherever the thing being integrated is nonzero. Stated exactly, following Owen (*Monte Carlo theory, methods and examples*, theorem 9.1):

$$Q(x) > 0 \quad \text{whenever} \quad f(x)P(x) \neq 0$$

Strict inequality, and it is a condition on where $Q$ is **positive**, not on where it is non-negative. A condition of the form "$Q(x) \ge 0$ whenever $P(x) \ne 0$" constrains nothing at all, since every probability density is non-negative everywhere by definition, so it is satisfied by every $Q$ including ones that make the estimator badly wrong. The real condition is absolute continuity of $P$ with respect to $Q$ on the region that contributes: $Q$ may be zero where $f P$ is zero, and nowhere else. Where $Q$ is allowed to vanish you will never draw an $x$, so the undefined $P(x)/0$ never actually appears; where $Q$ vanishes and $fP$ does not, an entire region of the answer is silently missing from every sample you will ever take.

**Variance is what decides whether it works.** Unbiasedness is cheap. Usability is not. With $Q$ positive on the support $\mathcal{Q}$,

$$\operatorname{Var}_Q[\hat{\mu}] = \frac{\sigma_Q^2}{n}, \qquad \sigma_Q^2 = \int_{\mathcal{Q}} \frac{\big(f(x)P(x)\big)^2}{Q(x)}\,dx - \mu^2$$

The $Q(x)$ in the denominator is the whole story: where $Q$ is small and $|f|P$ is not, the integrand is large, and if $Q$'s tails thin out faster than $|f|P$'s the integral diverges and $\sigma_Q^2 = \infty$. The estimator is then still unbiased and has infinite variance, which is worse than it sounds: a finite sample returns a perfectly ordinary looking number, the sample variance also returns an ordinary looking number, and neither converges. Owen's own summary is that importance sampling "can also backfire, yielding an estimate with infinite variance when simple Monte Carlo would have had a finite variance", and that it is the hardest variance reduction method to use well.

**Diagnosing weight concentration.** The available check is the effective sample size, computed from the weights actually drawn,

$$n_{\text{eff}} = \frac{\left(\sum_{i=1}^{n} w_i\right)^2}{\sum_{i=1}^{n} w_i^2}, \qquad w_i = \frac{P(x_i)}{Q(x_i)}$$

which equals $n$ when the weights are equal and falls toward 1 as one weight dominates. A run with $n = 10^5$ and $n_{\text{eff}} = 12$ has effectively twelve observations. The caveat is that $n_{\text{eff}}$ is computed from the same weights the estimate used, so badly skewed weights can give both a bad estimate and a reassuring diagnostic.

**Self-normalized importance sampling.** When $P$ or $Q$ is known only up to a normalizing constant, which is the usual situation in Bayesian work, only the unnormalized ratio $w_u(x) = P_u(x)/Q_u(x)$ is computable. Dividing by the sum of the weights makes the unknown constant cancel:

$$\tilde{\mu} = \frac{\sum_{i=1}^{n} f(x_i)\,w_u(x_i)}{\sum_{i=1}^{n} w_u(x_i)}$$

The trade is real and worth one sentence: $\tilde{\mu}$ is a ratio of two estimates rather than an average, so it is biased at finite $n$ while $\hat{\mu}$ is not, though it is consistent, converging to $\mu$ with probability 1 (Owen, theorem 9.2). It also demands more of $Q$, needing $Q(x) > 0$ wherever $P(x) > 0$, even where $f$ is zero, because the denominator integrates $f \equiv 1$. In exchange it is exact when $f$ is constant, which $\hat{\mu}$ is not, so probabilities estimated for a set and its complement actually sum to one.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| the proposal | $Q$ | none, and there is no safe default | a $Q$ placing more mass where $\vert f \vert P$ is large cuts $\sigma_Q^2$, and the optimum for the unbiased estimator is $Q(x) \propto \vert f(x)\vert P(x)$ | make $Q$ heavier tailed than $P$, never lighter, and check $n_{\text{eff}}$ on a pilot run before trusting the estimate |
| sample size | $n$ | none, you choose it | variance falls as $\sigma_Q^2/n$, provided $\sigma_Q^2$ is finite, and buys nothing at all when it is not | raise it until the standard error is small enough, and treat $n_{\text{eff}}$ rather than $n$ as the count that matters |
| normalization | - | unbiased $\hat{\mu}$ where $P/Q$ is computable | switching to $\tilde{\mu}$ trades unbiasedness for the ability to work with unnormalized densities, and for exactness on constant $f$ | forced to self-normalized whenever a normalizing constant is unknown, otherwise a judgement between the two |
| weight truncation | $w_{\max}$ | none, weights uncapped | capping $w_i$ at $w_{\max}$ bounds the variance and introduces bias that grows as the cap tightens | cap only when $n_{\text{eff}}$ shows a few weights dominating, and report that the estimate is now biased |

The seed of the draw from $Q$ is pinned rather than tuned. It changes which $x_i$ come back and therefore the number, but no value is better than another, see [[Random Seed]].

## Failure modes

- $Q$ missing part of $P$'s support. Wherever $Q(x) = 0$ and $f(x)P(x) \ne 0$, that region contributes to $\mu$ and can never appear in the sample. The estimator returns a confident number that is wrong by exactly the missing integral, no error is raised, and no diagnostic computed from the drawn sample can see it, because the evidence is precisely what was not drawn.
- Reading the support condition as "$Q(x) \ge 0$ when $P(x) \ne 0$" and concluding that any proposal will do. The condition is vacuous as stated, every density satisfies it, and acting on it produces the failure above.
- Weight concentration, with $n_{\text{eff}} \ll n$. A handful of draws carry nearly all the mass, the estimate is effectively an average of a few points, and the reported sample size says otherwise.
- Infinite variance with finite-sample output that looks fine. A $Q$ with lighter tails than $|f|P$ gives $\sigma_Q^2 = \infty$, yet every run returns a number and an apparently reasonable standard error. The symptom is that the estimate does not settle as $n$ grows, and occasionally jumps by orders of magnitude when one extreme draw lands.
- Using the unbiased estimator when only unnormalized densities are available. $P_u/Q_u$ is off by an unknown constant factor $c/b$, so $\hat{\mu}$ is scaled by that factor and is not an estimate of $\mu$ at all. The self-normalized form is the one whose constants cancel.
- Correcting a distribution shift with weights estimated from the same data whose shift is in question. $\hat{P}/\hat{Q}$ inherits the error in both estimates, and the ratio amplifies it in exactly the tail regions where the weights are largest and matter most.

## Implementation

There is no scikit-learn implementation of importance sampling, in 1.6 or since, and none is expected: this is an estimator for an expectation rather than a transform of a feature matrix, so it does not fit the estimator API at all. What exists instead is the four-line estimator, written where it is needed.

NumPy 2.x, the unbiased estimator and its self-normalized sibling, with `p` and `q` any callables returning densities:

```python
import numpy as np

rng = np.random.default_rng(42)

x = q_sample(rng, n=100_000)          # draws from the proposal Q
w = p(x) / q(x)                       # importance weights

mu_hat = np.mean(f(x) * w)            # unbiased, needs normalized p and q
mu_snis = np.sum(f(x) * w) / np.sum(w)  # self-normalized, constants cancel

n_eff = w.sum() ** 2 / np.square(w).sum()   # report this next to the estimate
```

Where it genuinely lives, since naming the real homes is more use than a library call would be.

**Off-policy evaluation in reinforcement learning.** Estimating the value of a target policy from trajectories generated by a different behaviour policy is the importance sampling problem exactly, with $P$ the target policy's distribution over trajectories and $Q$ the behaviour policy's. Precup, Sutton and Singh set out the connection in *Eligibility Traces for Off-Policy Policy Evaluation* (ICML 2000, pages 759 to 766), which is where the eligibility trace algorithms get their per-step ratios. The support condition bites here in its most concrete form: an action the behaviour policy never takes carries no information about what the target policy would have got for taking it.

**The clipped ratio in PPO.** Schulman, Wolski, Dhariwal, Radford and Klimov, *Proximal Policy Optimization Algorithms* (2017), define

$$r_t(\theta) = \frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)}$$

which is the importance weight of the current policy against the one that collected the data, and their objective is

$$L^{\text{CLIP}}(\theta) = \hat{\mathbb{E}}_t\Big[\min\big(r_t(\theta)\hat{A}_t,\; \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)\,\hat{A}_t\big)\Big]$$

with $\epsilon = 0.2$ a typical value. The paper's own name for $r_t(\theta)$ is the probability ratio, and the clip is weight truncation from the hyperparameter table doing its usual job: it bounds how far one reweighted sample can move the update, accepting bias to stop the variance. This is what "useful in policy-based reinforcement learning" means once it is made precise.

**Monte Carlo integration generally.** Rare event simulation in finance and insurance, high energy physics, Bayesian normalizing constants, and rendering in computer graphics all use it as the standard variance reduction technique, and Owen's chapter 9 is the treatment to check a claim against.
