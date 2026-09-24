---
note_kind: method
aliases:
  - low-rank factorization
  - low rank factorization
  - low-rank decomposition
  - low-rank weight approximation
  - weight factorization
  - compact convolutional filter
  - compact convolutional filters
up: "[[Model Compression]]"
sources:
  - "[[DMLS Ch07 Model Deployment and Prediction Service]]"
confidence: draft
---

## What it does and when

Low-rank factorization replaces a high-dimensional weight tensor with a product of lower-dimensional ones. The key idea is that a large weight matrix in a trained network usually carries far less independent information than its size suggests, so it can be rewritten as two thin matrices whose product is close to it, and the layer then stores and multiplies by the thin pair instead. It cuts the parameter count $P$ of a layer, which is the factor of [[Model Compression]]'s size $S = P \cdot b$ it attacks, and it leaves the bytes per parameter alone, so it composes with [[Model Quantization]], which cuts exactly that factor. Among the members that cut $P$ it is the one that works inside a layer, keeping the model and its layer structure while thinning individual matrices, where [[Knowledge Distillation]] replaces the model outright with a smaller student and [[Model Pruning]] zeroes or removes weights rather than rewriting them as a product.

Reach for it on a trained model whose large layers have a spectrum that decays, meaning a handful of directions carry most of what the matrix does, and when the thing to reduce is a large fully connected layer or a convolution whose kernel tensor can be unfolded into a matrix. It is the wrong tool when the matrix is close to full rank, since then any rank low enough to save parameters throws away most of the matrix, and when the layers are already small, since the saving only exists below a break-even rank derived below.

There are two ways the idea reaches a network, and they are different enough that the second is arguably not this method at all:

- **Factorizing a trained layer.** Decompose the weights of an existing model, replace each chosen layer with its factors, and fine-tune to recover the accuracy the approximation cost. The recovery step borrows the mechanics of fine-tuning from [[Transfer Learning]], though with source and target task identical it is not transfer in that note's sense: nothing crosses a gap between problems, and the network is only nursed back to the task it already had. This is low-rank factorization proper, and it works on any architecture with large linear or convolutional layers.
- **Designing compact filters in from the start.** Build the network out of small, cheap blocks, such as 1×1 convolutions or depthwise separable convolutions, and train it from scratch. The over-parameterized convolution filters of a conventional design are replaced with compact blocks, which both reduces the number of parameters and increases speed. [[Edge Computing]] is the setting these designs were built for, MobileNets being named for the mobile devices it targets. This is an architecture choice made before training, only meaningful for convolutional networks, and whether it counts as low-rank factorization is taken up at the end of the next section.

That split is what "only works on some architectures" means, and it means two different things for the two routes: compact filters exist only for convolutional layers and have to be designed in before training, while factorizing a trained layer needs a layer large enough, and a spectrum steep enough, for the factors to be both smaller and accurate.

## Algorithm or formula

Write a layer's weight matrix as $\mathbf{W} \in \mathbb{R}^{d_{\text{out}} \times d_{\text{in}}}$, so a fully connected layer computes $\mathbf{W}\mathbf{x}$ for an input $\mathbf{x} \in \mathbb{R}^{d_{\text{in}}}$. Replace it with a product

$$\mathbf{W} \approx \mathbf{A}\mathbf{B}, \qquad \mathbf{A} \in \mathbb{R}^{d_{\text{out}} \times r}, \quad \mathbf{B} \in \mathbb{R}^{r \times d_{\text{in}}}$$

and compute $\mathbf{A}(\mathbf{B}\mathbf{x})$, two thin multiplies in place of one wide one. The product has rank at most $r$, which is where the name comes from.

### When it saves anything

The original layer holds $d_{\text{out}}d_{\text{in}}$ parameters and the factored one holds $d_{\text{out}}r + rd_{\text{in}} = r(d_{\text{out}} + d_{\text{in}})$. The factorization is smaller exactly when

$$r(d_{\text{out}} + d_{\text{in}}) < d_{\text{out}}d_{\text{in}} \iff r < \frac{d_{\text{out}}d_{\text{in}}}{d_{\text{out}} + d_{\text{in}}}$$

dividing both sides by the positive $d_{\text{out}} + d_{\text{in}}$. The right-hand side is half the harmonic mean of the two dimensions, so it sits below the smaller of them: for a square $d \times d$ matrix it is $d/2$, and a rank-$r$ factorization of a square layer saves nothing until $r$ is under half the width. The compression ratio of the layer is

$$C = \frac{d_{\text{out}}d_{\text{in}}}{r(d_{\text{out}} + d_{\text{in}})}$$

The multiply-adds for one input follow the same counts, $d_{\text{out}}d_{\text{in}}$ against $r(d_{\text{out}} + d_{\text{in}})$, so on paper the arithmetic saving and the parameter saving are the same inequality. On hardware they are not, which is a failure mode below.

### The best factors are the truncated SVD

The singular value decomposition writes any $\mathbf{W}$ as a sum of rank-one pieces ordered by size,

$$\mathbf{W} = \mathbf{U}\boldsymbol\Sigma\mathbf{V}^{T} = \sum_{i=1}^{q} \sigma_i\,\mathbf{u}_i\mathbf{v}_i^{T}, \qquad \sigma_1 \ge \sigma_2 \ge \dots \ge \sigma_q \ge 0, \quad q = \min(d_{\text{out}}, d_{\text{in}})$$

with orthonormal columns $\mathbf{u}_i$ and $\mathbf{v}_i$. Keep the first $r$ terms, $\mathbf{W}_r = \sum_{i=1}^{r}\sigma_i\mathbf{u}_i\mathbf{v}_i^{T}$. The Eckart and Young theorem ("The approximation of one matrix by another of lower rank", *Psychometrika* 1, 1936) says no matrix of rank at most $r$ is closer to $\mathbf{W}$ in the Frobenius norm, and the error it leaves is exactly the discarded tail of the spectrum:

$$\min_{\operatorname{rank}(\mathbf{X}) \le r} \lVert \mathbf{W} - \mathbf{X} \rVert_F = \lVert \mathbf{W} - \mathbf{W}_r \rVert_F = \sqrt{\sum_{i=r+1}^{q} \sigma_i^{2}}$$

Mirsky (1960) extended the same optimality to every unitarily invariant norm, which includes the spectral norm, where the error is the first discarded singular value alone, $\lVert \mathbf{W} - \mathbf{W}_r \rVert_2 = \sigma_{r+1}$. So the factors are $\mathbf{A} = \mathbf{U}_r\boldsymbol\Sigma_r$ and $\mathbf{B} = \mathbf{V}_r^{T}$, the singular values folded into either side, and how good a rank-$r$ approximation can possibly be is read off the spectrum before anything is built: the fraction of the squared Frobenius norm retained is $\sum_{i \le r}\sigma_i^{2} / \sum_{i}\sigma_i^{2}$. A matrix whose singular values fall off quickly factorizes well at small $r$; a matrix with a flat spectrum does not factorize well at any $r$ that saves parameters.

The same decomposition appears in [[Normal Equation]], used there for something else entirely: to compute the pseudoinverse that solves a least-squares fit robustly. Here it compresses a weight matrix; there it solves for one.

### Convolutional layers

A convolution kernel is a four-way tensor, $D_K \times D_K$ spatially, $M$ input channels, $N$ output channels, and it can be unfolded into a matrix and factorized the same way, or decomposed with a tensor method. Two papers from 2014 are the standing cases of factorizing a trained convolutional network. Denton, Zaremba, Bruna, LeCun and Fergus (NeurIPS 2014) approximate each convolutional layer with SVD-based and clustering decompositions and then fine-tune the layers above until the prediction performance is restored, reporting 2 to 3 times faster convolutional layers within about one percent of the original accuracy and a 5 to 13 times reduction in the fully connected layers' weights. Jaderberg, Vedaldi and Zisserman (BMVC 2014) exploit redundancy across channels to build a low-rank basis of filters that are each rank one in the spatial domain, a $D_K \times D_K$ filter replaced by a vertical $D_K \times 1$ filter followed by a horizontal $1 \times D_K$ one, reporting 2.5 times faster with no loss of accuracy and 4.5 times faster with under one percent loss on a text recognition network. Applied to AlexNet, SVD-based factorization took the stored model from 240 MB to 48 MB, $C = 5$, at a top-1 cost of 57.2 to 56.0 percent (as tabulated by Iandola and colleagues 2016).

### Compact convolutional filters, and whether they belong here

SqueezeNet (Iandola, Han, Moskewicz, Ashraf, Dally and Keutzer, 2016) is built from **fire modules**: a squeeze layer of 1×1 filters that reduces the number of channels, feeding an expand layer with a mix of 1×1 and 3×3 filters. The design rules are to use 1×1 filters where possible, since a 1×1 filter has $1/9$ the parameters of a 3×3 one, and to keep few input channels going into the 3×3 filters that remain. The result matched AlexNet's ImageNet accuracy with 50 times fewer parameters, 4.8 MB against 240 MB.

MobileNets (Howard and colleagues, 2017) replace each standard convolution with a **depthwise separable** one: a depthwise convolution applying one $D_K \times D_K$ filter to each of the $M$ input channels separately, then a pointwise 1×1 convolution mixing the $M$ channels into $N$. Counting multiply-adds over a $D_F \times D_F$ feature map, the standard convolution costs $D_K^{2}MND_F^{2}$, the depthwise step $D_K^{2}MD_F^{2}$, and the pointwise step $MND_F^{2}$, so the ratio is

$$\frac{D_K^{2}MD_F^{2} + MND_F^{2}}{D_K^{2}MND_F^{2}} = \frac{D_K^{2}MD_F^{2}}{D_K^{2}MND_F^{2}} + \frac{MND_F^{2}}{D_K^{2}MND_F^{2}} = \frac{1}{N} + \frac{1}{D_K^{2}}$$

The parameter counts, $D_K^{2}M + MN$ against $D_K^{2}MN$, give the same ratio with $D_F^{2}$ absent throughout. At $D_K = 3$ and a few hundred output channels that is a little over $1/9$, which is the 8 to 9 times less computation the paper reports; its full network drops from 29.3 to 4.2 million parameters and from 4866 to 569 million multiply-adds, at an ImageNet accuracy of 70.6 percent against 71.7 for the same network with full convolutions.

Whether compact filters are a *type* of low-rank factorization depends on which of the two designs is meant. The depthwise separable convolution is a genuine factorization of the kernel: writing the depthwise kernel $\hat{K}_{i,j,m}$ and the pointwise weights $p_{m,n}$, the equivalent standard kernel is $K_{i,j,m,n} = \hat{K}_{i,j,m}\,p_{m,n}$, so for each input channel $m$ the $D_K^{2} \times N$ slice is the outer product of one spatial filter and one column of pointwise weights, a rank-one slice. That is a low-rank structure, but one chosen before training and learned directly, not a decomposition of trained weights. The fire module is not a factorization in the linear-algebra sense at all: there is a nonlinearity between the squeeze and the expand layers, so the pair does not multiply out to a single low-rank kernel, and swapping a 3×3 filter for a 1×1 one is a change of architecture, not an approximation of anything. Cheng, Wang, Zhou and Zhang's survey (2017) accordingly keeps "transferred/compact convolutional filters" as a category separate from low-rank factorization, applicable to convolutional layers only and supporting training from scratch only, whereas low-rank factorization covers convolutional and fully connected layers and supports both pretrained models and training from scratch. The accurate statement is that compact convolutional filters are a neighbouring technique that shares the goal and, in the depthwise separable case, the structure, rather than a type of low-rank factorization.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| rank | $r$ | none; there is no standard default | the approximation error $\sqrt{\sum_{i > r}\sigma_i^{2}}$ falls and the accuracy before fine-tuning rises, while the layer's parameter count $r(d_{\text{out}} + d_{\text{in}})$ grows linearly until, at the break-even rank $d_{\text{out}}d_{\text{in}}/(d_{\text{out}} + d_{\text{in}})$, the factorization is no smaller than the original | read the spectrum first and pick the smallest $r$ retaining a chosen fraction of $\sum\sigma_i^{2}$, then confirm on held-out data after fine-tuning; a single rank for every layer is rarely right, since layers differ in how steep their spectra are |
| layers factorized | | none | factorizing more layers compounds the saving and compounds the error, since each approximated layer feeds a perturbed input to the next | start with the layers that hold the most parameters (usually the fully connected ones) or the most time (usually the early convolutions), and add layers one at a time while the held-out score stays within budget |
| fine-tuning budget | | none | more passes over the training data recover more of the accuracy the approximation cost, up to the original model's level, at the cost of training time | train until the held-out score stops improving; Denton and colleagues recovered their accuracy in under two passes over ImageNet, which is the order of magnitude to expect for a mild approximation |

The choice of decomposition is a switch rather than a dial: plain SVD on an unfolded matrix, a CP or Tucker decomposition of the four-way convolution kernel, or a data-aware fit that minimizes the error of the layer's outputs on real inputs rather than the error of its weights. Each changes the factors returned, and the choice is made by the layer type and the tooling at hand rather than tuned.

## Failure modes

- **Choosing $r$ from the parameter budget instead of the spectrum.** Picking "rank 64 because that gives a five times smaller layer" says nothing about how much of the matrix survives. On a $512 \times 1024$ matrix with a flat spectrum, a random Gaussian one, rank 64 keeps only 29 percent of the squared Frobenius norm; the same rank on a matrix whose singular values decay keeps nearly all of it. The spectrum is free to compute and should be looked at before a rank is fixed.
- **Choosing $r$ at or above the break-even rank.** Above $d_{\text{out}}d_{\text{in}}/(d_{\text{out}} + d_{\text{in}})$ the "compressed" layer has more parameters than the original and does more arithmetic, and nothing raises an error: at $512 \times 1024$ the break-even is about 341, and rank 400 gives $C = 0.85$.
- **Factorizing without fine-tuning.** The truncated SVD is optimal for the weights of one layer in isolation, not for the network's predictions. Errors in successive layers compound, and a harsh approximation that looked tolerable layer by layer can cost far more accuracy end to end. Denton and colleagues fix the approximated layer and fine-tune the layers above it until the original performance returns, and Cheng and colleagues list the need for extensive retraining as a general drawback of the approach.
- **Minimizing the weight error when the output error is what matters.** $\lVert \mathbf{W} - \mathbf{A}\mathbf{B} \rVert_F$ weights every direction of input space equally, but real inputs occupy only some of them, so the layer's actual error $\lVert (\mathbf{W} - \mathbf{A}\mathbf{B})\mathbf{x} \rVert$ can be larger or smaller than the weight error suggests. Jaderberg and colleagues compare filter reconstruction with data reconstruction, fitting the factors to reproduce the layer's outputs on training data, for this reason.
- **Saving parameters and not wall-clock time.** Two small matrix multiplies are not automatically faster than one large one: each call has fixed overhead, thin matrices use the hardware less efficiently, and a convolution split into several stages can need extra memory reordering between them. Denton and colleagues found it often difficult to get speedups close to the theoretical gains from counting operations, and Jaderberg and colleagues found that one of their two schemes, implemented with standard convolution routines, needed so many extra memory reorderings and library calls that the approximation's speedup was negated. The only honest figure is a measured latency on the target hardware.
- **Presenting `TruncatedSVD` as model compression.** scikit-learn's `TruncatedSVD` factorizes a data matrix to reduce the number of features, which is a different operation on a different object and belongs to [[Dimensionality Reduction]]; applying it to a feature matrix compresses the data, not the model.

## Implementation

**scikit-learn has no model-compression API.** `sklearn.decomposition.TruncatedSVD` and `sklearn.utils.extmath.randomized_svd` in scikit-learn 1.6 compute truncated SVDs of a data matrix $\mathbf{X}$ of $m$ instances by $n$ features, for [[Dimensionality Reduction]], and there is no scikit-learn model whose weights are a matrix you would factorize this way. The honest demonstration is the mathematics itself on a weight matrix, in NumPy 2.x (run here with 2.0.2), printing the parameter count, the ratio and the Frobenius error, and checking the error against the Eckart and Young tail:

```python
import numpy as np

rng = np.random.default_rng(42)
d_out, d_in = 512, 1024

# A stand-in for a trained weight matrix whose spectrum decays
# (trained layers often do; a random Gaussian matrix does not).
Q1, _ = np.linalg.qr(rng.standard_normal((d_out, d_out)))
Q2, _ = np.linalg.qr(rng.standard_normal((d_in, d_out)))
spectrum = np.exp(-np.arange(d_out) / 40.0)
W = Q1 @ np.diag(spectrum) @ Q2.T                   # shape (512, 1024)

U, s, Vt = np.linalg.svd(W, full_matrices=False)   # s is sorted, largest first
r_breakeven = d_out * d_in / (d_out + d_in)

for r in (16, 64, 128, 341, 400):
    A = U[:, :r] * s[:r]                           # (d_out, r): singular values folded in
    B = Vt[:r, :]                                  # (r, d_in)
    err = np.linalg.norm(W - A @ B, "fro")
    tail = np.sqrt(np.sum(s[r:] ** 2))             # Eckart-Young: error is the discarded tail
    params = r * (d_out + d_in)
    print(f"r={r:4d}  params={params:7d} / {d_out * d_in}  "
          f"ratio={d_out * d_in / params:5.2f}  "
          f"rel_err={err / np.linalg.norm(W, 'fro'):.2e}  "
          f"tail_matches={np.isclose(err, tail)}")

print(f"break-even rank: {r_breakeven:.1f}")

# The same experiment on a matrix with a flat spectrum
G = rng.standard_normal((d_out, d_in))
sg = np.linalg.svd(G, compute_uv=False)
kept = np.sum(sg[:64] ** 2) / np.sum(sg ** 2)
print(f"random Gaussian matrix, r=64 keeps {kept:.1%} of the squared Frobenius norm")
```

It prints a ratio of 5.33 at a relative error of 0.20 for $r = 64$, 2.67 at 0.04 for $r = 128$, a ratio of 1.00 at the break-even rank of about 341, 0.85 at $r = 400$, and 29.2 percent retained for the Gaussian matrix at $r = 64$, with the measured error matching the tail formula at every rank.

Factorized layers are actually built in a deep learning framework. In PyTorch 2.x, `torch.linalg.svd(A, full_matrices=True)` returns `U, S, Vh` (it supersedes the older `torch.svd`), and a trained `torch.nn.Linear`, whose `weight` has shape `(out_features, in_features)`, is replaced by a pair of `Linear` layers with no nonlinearity between them:

```python
import torch
from torch import nn

def factorize_linear(layer: nn.Linear, r: int) -> nn.Sequential:
    U, S, Vh = torch.linalg.svd(layer.weight.detach(), full_matrices=False)
    first = nn.Linear(layer.in_features, r, bias=False)                 # B = V_r^T
    second = nn.Linear(r, layer.out_features, bias=layer.bias is not None)  # A = U_r S_r
    with torch.no_grad():
        first.weight.copy_(Vh[:r, :])
        second.weight.copy_(U[:, :r] * S[:r])
        if layer.bias is not None:
            second.bias.copy_(layer.bias)
    return nn.Sequential(first, second)
```

The replacement is then fine-tuned with the rest of the network. Depthwise separable convolutions are built directly rather than derived: `torch.nn.Conv2d(M, M, kernel_size=3, groups=M)` is the depthwise step and `torch.nn.Conv2d(M, N, kernel_size=1)` the pointwise one.
