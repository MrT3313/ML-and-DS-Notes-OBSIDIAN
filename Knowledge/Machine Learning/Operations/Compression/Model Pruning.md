---
note_kind: method
aliases:
  - model pruning
  - network pruning
  - neural network pruning
  - weight pruning
  - magnitude pruning
  - structured pruning
  - unstructured pruning
  - pruned model
  - pruned network
  - lottery ticket hypothesis
up: "[[Model Compression]]"
sources:
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## What it does and when

Pruning removes parts of a fitted model that contribute little to its predictions, so that what is left is smaller and, under the right conditions, cheaper to store and to run. The thing being cut is the model: branches of a tree, weights or whole units of a network. That is a different operation from the pruning in [[Association Rule Learning]], where Apriori cuts candidate itemsets out of a search space before they are ever counted, and from the pruning [[Triangle Inequality]] licenses in ball trees and accelerated k-means, where distance bounds rule out comparisons that never need to be made. Search pruning leaves the answer unchanged and saves work finding it; model pruning changes the model and therefore, by some amount that has to be measured, what it predicts.

The name comes from decision trees, where a fully grown tree is cut back by removing branches that add little to the classification and mostly fit noise. Breiman, Friedman, Olshen and Stone (*Classification and Regression Trees*, 1984) made this a criterion, cost-complexity pruning, which trades a subtree's error against its number of leaves and is set out below; C4.5 (Quinlan 1993) prunes by an error estimate instead. [[Baseline Model]] already meets the result, in Holte's comparison of one-attribute rules against the pruned trees of C4. The tree as a model in its own right arrives with HOML chapter 6; this note states only the criterion that pruning a tree optimizes.

On a neural network pruning takes two forms, and the difference between them decides almost every practical question below.

- **Unstructured pruning** finds the individual weights least useful to the predictions and sets them to zero. The architecture is untouched: every matrix keeps its shape, and some of its entries are now zero.
- **Structured pruning** removes whole units, channels or filters, together with every weight feeding into or out of them. That changes the architecture itself, since a layer that had 300 units now has fewer, and it reduces the parameter count by physically shrinking the matrices.

Either way the result is sparser than what was fitted, and the reason to reach for it is a deployment budget, memory, latency or energy, that the fitted model does not meet. Among the [[Model Compression]] techniques it is the one that keeps the model you have and deletes from it: [[Knowledge Distillation]] fits a new, smaller model, [[Model Quantization]] keeps every weight at lower precision, and [[Low-Rank Factorization]] replaces a matrix by a product of thinner ones. They compose. Han, Mao and Dally's "Deep Compression" (ICLR 2016) chained pruning, quantization and Huffman coding and reduced AlexNet from 240 MB to 6.9 MB, a factor of $240/6.9 \approx 35$, and VGG-16 from 552 MB to 11.3 MB, about $49$, without loss of accuracy, with pruning alone cutting the number of connections by $9$ to $13$ times.

[[Lasso Regression]] reaches the same end state, weights at exactly zero, by a different route: an $\ell_1$ penalty applied during the fit rather than a removal applied after it.

### What storage pruning actually saves

Sparse architecture needs less storage than dense architecture only under a condition, and the condition is worth deriving because the unconditional version is the one usually repeated. A dense $m \times n$ weight matrix with values of $v$ bits costs $mn\,v$ bits. Zeroing entries saves nothing unless the matrix is stored in a sparse format, and a sparse format must record where each surviving value sits. Compressed sparse row (CSR) stores each surviving value, the column index of each surviving value, and one row pointer per row plus one, which is the $2a + n + 1$ numbers that Deep Compression states for $a$ nonzeros. With a density $d$ (the fraction of weights kept, so sparsity $s = 1 - d$) and indices of $i$ bits,

$$\text{bits}_{\text{CSR}} = d\,mn\,(v + i) + (m + 1)\,i, \qquad \text{bits}_{\text{dense}} = mn\,v$$

and CSR is smaller exactly when

$$d \;<\; \frac{v}{v + i} - \frac{(m+1)\,i}{mn\,(v + i)} \;\approx\; \frac{v}{v + i}$$

the correction term being about $i/(n(v+i))$ per weight and negligible for any real layer. At 32-bit values and 32-bit indices the break-even is $d < 1/2$: a network pruned to half its weights stores in no less space than the dense original. At 16-bit values and 32-bit indices it is $d < 1/3$, and at 8-bit quantized values with 32-bit indices $d < 1/5$, so the more aggressively the values are compressed the more sparsity it takes for the index overhead to pay off. PyTorch's CSR tensors default to 64-bit indices, which puts the break-even at $d < 32/96 = 1/3$ for 32-bit values: measured on a pruned $300 \times 784$ layer in PyTorch 2.14, CSR used $1.50$, $1.20$, $0.90$ and $0.30$ times the dense bytes at sparsities $0.5$, $0.6$, $0.7$ and $0.9$, which is $3d + 0.0026$ to four figures.

The same formula says the saving is smaller than $1/d$ even when there is one. At $s = 0.9$ with 32-bit values and indices, the ratio is $d(v+i)/v = 0.1 \times 2 = 0.2$, a factor of $5$ and not the factor of $10$ the parameter count suggests. Deep Compression spends much of its effort on exactly this term, storing the difference between successive indices rather than the absolute index, in $8$ bits for convolutional layers and $5$ for fully connected ones.

### What speed pruning actually buys

Unstructured sparsity rarely makes dense hardware faster. A GPU or a vectorized CPU multiplies dense blocks, and a matrix with scattered zeros is still a dense matrix unless the kernel is written to skip them, which costs irregular memory access. Deep Compression measured $3$ to $4$ times layerwise speedups from pruning, but at batch size $1$, where a fully connected layer is a matrix-vector product and bound by memory traffic rather than arithmetic, and its authors state that with batching, where weights can be blocked and reused, the pruned network no longer shows its advantage. Structured pruning does not have this problem. Removing a whole filter or unit leaves a smaller dense layer, which runs on the same dense kernels and is faster in proportion to the work removed; Li, Kadav, Durdanovic, Samet and Graf ("Pruning Filters for Efficient ConvNets", ICLR 2017) reduce inference cost for VGG-16 on CIFAR-10 by up to $34$ percent this way without needing sparse libraries. The price is coarser granularity: removing a unit removes every weight attached to it, good and bad.

### What pruning finds

Two ICLR 2019 papers read the same procedure in opposite ways, and both are worth carrying because the disagreement is about what the kept weights are worth.

- **The lottery ticket hypothesis** (Frankle and Carbin): a dense, randomly initialized network contains a subnetwork, the winning ticket, that trained in isolation from its original initial values reaches comparable test accuracy in a similar number of iterations. They find such tickets by iterative magnitude pruning followed by resetting the survivors to their initial values, at $10$ to $20$ percent of the original size on fully connected and convolutional networks for MNIST and CIFAR-10, and report that the same sparse structure randomly reinitialized trains far worse. On that reading, pruning finds a lucky initialization.
- **"Rethinking the Value of Network Pruning"** (Liu, Sun, Zhou, Huang and Darrell): for every structured pruning method they examined, fine-tuning the pruned model gave comparable or worse accuracy than training the same pruned architecture from random initialization, and with a well chosen learning rate the winning-ticket initialization brought no improvement over random. On that reading the inherited weights are not what matters and pruning is a form of architecture search.

The two are narrower in disagreement than their headlines. Frankle and Carbin report themselves that on the deeper VGG-19, at the learning rate Liu and co-authors used, iterative pruning does not find winning tickets and does no better than random reinitialization, and that finding them at higher learning rates required warmup. What survives both papers is that the pruned *structure* is useful, and that whether the pruned *weights* are depends on the training setup.

How much weight any single pruning result can carry was measured directly. Blalock, Gonzalez Ortiz, Frankle and Guttag ("What Is the State of Neural Network Pruning?", MLSys 2020) aggregated results from $81$ papers and found the literature too fragmented to say which method is best or how much progress thirty years have made: $49$ datasets and $132$ architectures across those papers, the most common pairing (VGG-16 on ImageNet) used by only $22$ of them, and more than a quarter of the papers comparing against no earlier pruning method at all. Among the findings that did replicate: magnitude pruning substantially compresses networks at little accuracy cost, many methods beat random pruning at high sparsity, allocating sparsity per layer or globally beats pruning every layer uniformly, and pruned models sometimes beat their own original architecture but rarely beat a better architecture.

### Pruning and bias

Pruning can introduce bias into a model, and the evidence for exactly what that means is specific. Hooker, Courville, Clark, Dauphin and Frome ("What Do Compressed Deep Neural Networks Forget?", 2019) trained populations of $30$ models per setting on CIFAR-10, ImageNet and CelebA, pruned by magnitude to sparsities of $0.3$, $0.5$, $0.7$ and $0.9$, and compared them to unpruned populations. Top-line accuracy barely moved (ImageNet top-1 from $76.68$ to $75.87$ percent at $s = 0.5$) while the damage concentrated: the number of ImageNet classes with a statistically significant change in recall grew from $170$ at $s = 0.5$ to $372$ at $s = 0.7$ and $637$ at $s = 0.9$. They define a **pruning identified exemplar** (PIE) as an instance whose most frequent predicted label across the pruned population differs from that across the unpruned one. At $s = 0.9$ these were $10.27$ percent of the ImageNet test set, and the unpruned model itself classified them far worse ($39.81$ percent top-1 on PIEs against $76.75$ overall). A human study found PIEs over-index on mislabelled, low-quality, multi-object and fine-grained images, and the authors' summary is that compression impairs the long tail of less frequent instances. Pruned models were also more sensitive to corruptions and natural adversarial images, increasingly so at higher sparsity, and the quantization methods they tested showed less disparate impact than pruning.

The finding about protected attributes is in the companion paper (Hooker, Moorosi, Clark, Bengio and Denton, "Characterising Bias in Compressed Models", 2020), on CelebA with the task of predicting blond hair. Pruning to $s = 0.95$ cost only $94.73 - 93.39 = 1.34$ points of accuracy, but the false positive rate for the Male subgroup rose by $49.54$ percent relative to baseline against $6.32$ percent for not Male and $12.72$ percent overall, and the authors tie it to representation: blond not-Male faces are $14$ percent of the training set and blond Male faces $0.85$ percent. So "can introduce bias" means precisely this: pruning spends its accuracy loss disproportionately on rare and atypical instances and on underrepresented subgroups, while the aggregate hides it, which is the arithmetic [[Slice-Based Evaluation]] sets out.

## Algorithm

The protocol is the same loop for magnitude pruning, saliency pruning and structured pruning; only the score in step 2 and the unit of removal in step 3 differ. Write the network's weights as $\mathbf{w}$, the loss as $E(\mathbf{w})$, a binary mask of the same shape as $\mathbf{m}$, and the target sparsity as $s$.

1. **Train to convergence.** Pruning scores are read off a fitted model, and both scoring rules below assume the weights sit at a minimum.
2. **Score every candidate for removal.** For weight magnitude pruning the score is $\lvert w_k \rvert$. For Optimal Brain Damage (LeCun, Denker and Solla, NeurIPS 1989) it is the saliency
   $$s_k = \tfrac{1}{2}\, h_{kk}\, w_k^{2}$$
   with $h_{kk}$ the diagonal entry of the Hessian of $E$, derived below. For structured pruning the score belongs to a group, typically the norm of a unit's or filter's weights.
3. **Prune.** Set $m_k = 0$ for the lowest-scoring candidates until the fraction removed reaches this round's target, and replace $\mathbf{w}$ by $\mathbf{m} \odot \mathbf{w}$. Han, Pool, Tran and Dally (NeurIPS 2015) set the threshold per layer as a quality parameter times the standard deviation of that layer's weights; ranking all layers' weights together is global pruning, which Blalock and co-authors found beats uniform per-layer pruning.
4. **Fine-tune with the mask fixed.** Continue training only the surviving weights, keeping their trained values rather than reinitializing them, at a reduced learning rate (Han and co-authors used $1/10$ and $1/100$ of the original). This step is what the method rests on: without it, their AlexNet lost accuracy once only a third of the connections were left, and with it, only once a tenth were left.
5. **Repeat steps 2 to 4** until the target sparsity is reached. Iterating, rather than removing everything at once, raised Han and co-authors' pruning rate on AlexNet from $5$ to $9$ times at no loss of accuracy. With $n$ equal rounds each keeping a fraction $r$ of the survivors, $r^{n} = 1 - s$, so
   $$r = (1 - s)^{1/n}$$
   and reaching $s = 0.9$ in five rounds removes about $37$ percent of the survivors each round. Zhu and Gupta ("To prune, or not to prune", 2017) instead raise the sparsity gradually during training, from $s_i$ to $s_f$ over $n$ pruning steps spaced $\Delta t$ apart from step $t_0$:
   $$s_t = s_f + (s_i - s_f)\left(1 - \frac{t - t_0}{n\,\Delta t}\right)^{3}$$
   which prunes fast early, while there are many redundant weights, and slowly near the end. This is the schedule Hooker and co-authors used.
6. **Materialize and evaluate.** Store the result in a sparse format only if the density clears the storage inequality above, or physically rebuild the smaller layers after structured pruning. Then score the pruned model on held-out data, overall and per slice, against both the unpruned model and a dense model of the pruned size trained from scratch.

Blalock and co-authors write the same loop as their Algorithm 1, which is the common skeleton of the $81$ papers they surveyed.

### Why saliency and not magnitude

Optimal Brain Damage asks what deleting a weight costs in loss rather than how large it is. A second-order Taylor expansion of $E$ around the trained weights gives, for a perturbation $\delta \mathbf{w}$,

$$\delta E = \sum_k g_k\, \delta w_k + \tfrac{1}{2}\sum_k h_{kk}\, \delta w_k^{2} + \tfrac{1}{2}\sum_{k \neq l} h_{kl}\, \delta w_k\, \delta w_l + O(\lVert \delta \mathbf{w} \rVert^{3})$$

and three approximations cut it down. At convergence the gradient $g_k$ is zero, which removes the first term; the cross terms $h_{kl}$ are ignored, which treats deletions as independent; and the cubic remainder is dropped. Deleting weight $k$ means $\delta w_k = -w_k$, so $\delta E \approx \frac{1}{2} h_{kk} w_k^{2}$, which is the saliency. Magnitude pruning is the special case that assumes every $h_{kk}$ is equal. On their digit recognizer, deleting by saliency increased the loss significantly less than deleting by magnitude, the prediction tracked the measured loss for up to about $30$ percent of the parameters deleted, and past that the neglected cross terms and higher-order terms took over.

### The criterion in trees

A tree $T$ with $\lvert \tilde{T} \rvert$ leaves and training error (or, in scikit-learn, total sample-weighted leaf impurity) $R(T)$ is scored by

$$R_\alpha(T) = R(T) + \alpha\, \lvert \tilde{T} \rvert, \qquad \alpha \ge 0$$

and pruning picks the subtree of the fully grown tree that minimizes it. Each extra leaf has to buy at least $\alpha$ of error reduction to stay. For an internal node $t$ with branch $T_t$, collapsing the branch to a leaf is worthwhile once $\alpha$ exceeds

$$\alpha_{\text{eff}}(t) = \frac{R(t) - R(T_t)}{\lvert \tilde{T}_t \rvert - 1}$$

the error the branch saves per leaf it spends. Weakest-link pruning collapses the node with the smallest $\alpha_{\text{eff}}$, recomputes, and repeats, which produces a nested sequence of subtrees each optimal over an interval of $\alpha$; the value of $\alpha$ is then chosen by [[Cross-Validation]], so it plays the same role as the regularization strength $\alpha$ in [[Regularization]]. C4.5 takes a different route: it replaces each leaf's observed error rate with a pessimistic upper confidence bound on it and collapses a subtree when a single leaf would have a pessimistic error no larger than the subtree's.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| target sparsity | $s$ | none, chosen per deployment | fewer weights kept, so smaller and possibly cheaper, with accuracy holding flat up to a point and then falling; the loss falls disproportionately on rare slices before it shows in the aggregate | the largest $s$ at which held-out accuracy, overall and on the worst slice, stays within tolerance. Check storage against $d < v/(v+i)$ before counting a saving |
| schedule | $n$, or $s_i, t_0, \Delta t$ | one-shot, $n = 1$ | more rounds, or a slower gradual ramp, remove less per step and let fine-tuning recover in between, which reaches a higher sparsity at the same accuracy (5 to 9 times on AlexNet) at the cost of more training | iterate whenever the target sparsity is high; the cubic schedule of Zhu and Gupta needs only its start, end and spacing |
| granularity | | individual weights | moving from weights to units, channels or filters makes the result a smaller dense model that runs faster on ordinary hardware, but removes good weights along with bad ones, so accuracy falls sooner for the same parameter reduction | decided by the target hardware: structured when the goal is latency on dense kernels, unstructured when the goal is storage or sparse kernels exist |
| scope | | per layer | global ranking lets sparsity fall where weights are least useful, so some layers are pruned far harder than others | prefer global or per-layer allocation over uniform per-layer pruning. `global_unstructured` below put $80$ percent sparsity on one layer and $50$ on the other for an overall target of $0.8$ |
| scoring criterion | $\lvert w_k \rvert$ or $s_k$ | magnitude | categorical. Saliency accounts for curvature, so it removes weights that are small but important last; magnitude is cheaper and is what most libraries implement | magnitude unless the Hessian diagonal is affordable and the sparsity is modest |
| fine-tuning length and rate | | none, set per model | more fine-tuning recovers more of the accuracy lost in each round; too high a rate discards the inherited weights, which is the regime Liu and co-authors found equivalent to training from scratch | a reduced rate ($1/10$ of the original in Han and co-authors), long enough that validation accuracy stops improving |
| complexity parameter (trees) | $\alpha$, `ccp_alpha` | `0.0`, no pruning | larger $\alpha$ charges more per leaf, so a smaller tree with more bias and less variance, down to the root alone | cross-validate over the $\alpha$ values the pruning path returns; never pick by the test score |

The random seed that initializes the network before pruning is pinned rather than tuned; under the lottery ticket reading it is not neutral, since the winning ticket is tied to its initial values, which is why pruning experiments should report results over several seeds ([[Random Seed]]).

## Failure modes

- **Reporting a parameter reduction as a speedup.** A network pruned to $s = 0.9$ unstructured has a tenth of the nonzero weights and, on a GPU running dense kernels at a normal batch size, takes the same time as before, since the zeros are multiplied like any other number. Deep Compression's own speedups hold at batch size $1$ and vanish under batching. Report measured latency on the target hardware, or prune structurally.
- **Reporting a parameter reduction as a storage saving.** Zeros stored densely cost exactly as much as the weights they replaced, and PyTorch's mask-based pruning holds the original weights and a same-size float mask side by side, measured at $2.0$ times the dense bytes until `prune.remove` is called. Even stored sparsely, $s = 0.5$ with 32-bit values and indices saves nothing, and $s = 0.9$ saves a factor of $5$, not $10$.
- **Pruning without fine-tuning.** Without retraining, Han and co-authors' AlexNet began losing accuracy with a third of its connections left instead of a tenth, so one-shot pruning of a deployed model with no training loop available caps the achievable sparsity well below what the literature reports.
- **The aggregate hides the damage on a subgroup.** ImageNet top-1 moving from $76.68$ to $75.02$ percent at $s = 0.7$ reads as a free compression, while $372$ classes changed recall significantly; on CelebA a $1.34$ point accuracy drop came with a $49.54$ percent rise in the Male false positive rate. The check is [[Slice-Based Evaluation]] of the pruned model against the unpruned one, with the worst slice reported beside the pooled score.
- **Comparing only against the unpruned original.** A pruned large network that beats its own dense parent may still lose to a smaller, better-designed dense network of the same size, which Blalock and co-authors found is the usual outcome, and to the pruned architecture trained from scratch, which Liu and co-authors found matched fine-tuning for structured pruning. Both are the baselines a pruning result has to clear.
- **Choosing the tree's $\alpha$ on the test set.** The pruning path yields a whole sequence of trees, and picking the one that scores best on the test set turns it into a validation set. Choose $\alpha$ by cross-validation on the training split.

## Implementation

In scikit-learn 1.6 the one thing that prunes is minimal cost-complexity pruning on trees, through the `ccp_alpha` argument of `DecisionTreeClassifier` and `DecisionTreeRegressor` (and the tree ensembles) and the `cost_complexity_pruning_path` method that returns the effective alphas of the nested subtree sequence. scikit-learn 1.6 and NumPy 2.x:

```python
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.tree import DecisionTreeClassifier

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42)

# the effective alpha at which each subtree in the nested sequence becomes optimal
path = DecisionTreeClassifier(random_state=42).cost_complexity_pruning_path(X_train, y_train)
ccp_alphas, impurities = path.ccp_alphas, path.impurities   # ascending; impurities[i] is R(T) at ccp_alphas[i]

# choose alpha by cross-validation on the training split, never by the test score
search = GridSearchCV(DecisionTreeClassifier(random_state=42),
                      {"ccp_alpha": ccp_alphas[:-1]},         # the last alpha collapses the tree to its root
                      cv=5)
search.fit(X_train, y_train)

full = DecisionTreeClassifier(random_state=42).fit(X_train, y_train)
pruned = search.best_estimator_
print(full.get_n_leaves(), pruned.get_n_leaves())            # 18 11
print(full.score(X_test, y_test), pruned.score(X_test, y_test))
```

On this split the path has $13$ subtrees, cross-validation picks $\alpha \approx 0.0031$, and the tree goes from $18$ leaves to $11$ with test accuracy $0.923$ and $0.937$. On $143$ test rows that difference is two rows, which is not evidence that pruning helped; the defensible reading is that seven leaves were removed at no measurable cost. `ccp_alpha=0.0` is the default, so an unconfigured tree is never pruned.

**Nothing in scikit-learn prunes a neural network.** Its `MLPClassifier` and `MLPRegressor` expose their weights as `coefs_`, and zeroing entries by hand is possible, but there is no mask, no schedule and no fine-tuning with the mask held, so it is not the protocol above. The real API is `torch.nn.utils.prune` in PyTorch (checked against 2.14), whose functions are `l1_unstructured`, `random_unstructured`, `ln_structured`, `random_structured`, `global_unstructured`, `custom_from_mask`, `identity`, `remove` and `is_pruned`. PyTorch 2.14:

```python
import torch.nn as nn
import torch.nn.utils.prune as prune

model = nn.Sequential(nn.Linear(784, 300), nn.ReLU(), nn.Linear(300, 10))

# unstructured: zero the 90% of weights with the smallest |w| in the first layer
prune.l1_unstructured(model[0], name="weight", amount=0.9)

# structured: zero the half of the output units (rows, dim=0) with the smallest L2 norm
prune.ln_structured(model[2], name="weight", amount=0.5, n=2, dim=0)

# global: rank both layers' weights together and keep the top 20% overall
other = nn.Sequential(nn.Linear(784, 300), nn.ReLU(), nn.Linear(300, 10))
prune.global_unstructured([(other[0], "weight"), (other[2], "weight")],
                          pruning_method=prune.L1Unstructured, amount=0.8)

# ... fine-tune here: gradients flow through the mask, so pruned weights stay at zero ...

prune.remove(model[0], "weight")   # fold the mask in: weight becomes an ordinary parameter again
```

The mechanics matter for the failure modes. Pruning a parameter renames it to `weight_orig`, adds a buffer `weight_mask`, and recomputes `weight` as their product on every forward pass through a hook, so a pruned layer holds more memory than an unpruned one until `prune.remove` makes the zeros permanent. Even then `weight` is a dense tensor with zeros in it: storing it sparsely is a separate step (`weight.detach().to_sparse_csr()`, whose indices default to 64-bit), and `ln_structured` zeroes whole rows without shrinking the matrix, so realizing the speedup of structured pruning means building a smaller `nn.Linear` and copying the surviving rows and the matching columns of the next layer into it.
