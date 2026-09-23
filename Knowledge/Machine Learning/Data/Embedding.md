---
note_kind: concept
aliases:
  - embedding
  - embeddings
  - embedding space
  - embedding vector
  - learned embedding
  - positional embedding
  - positional embeddings
  - positional encoding
  - discrete embedding
  - continuous embedding
  - dense representation
  - distributed representation
up: "[[Feature Engineering]]"
sources:
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

An embedding is a vector that represents a piece of data, and all the vectors one algorithm produces for one type of data make up that algorithm's embedding space, every vector in a space carrying the same number of entries. What separates an embedding from any other list of numbers is that the coordinates are fitted rather than assigned, and that the geometry is the product: distance and direction in the space are meant to stand for relationships between the items, so two items the data treats alike land near each other.

## Formal statement

Let $\mathcal{V}$ be the set of items to be represented, the distinct words of a corpus or the levels of a categorical [[Feature]]. An embedding is a map into a real space of fixed width,

$$e: \mathcal{V} \to \mathbb{R}^{d}, \qquad d \text{ chosen in advance}, \qquad d \ll |\mathcal{V}| \text{ in the usual case}$$

and the inequality is the whole of why it is worth doing. [[One-Hot Encoding]] is also a map out of $\mathcal{V}$, into $\{0,1\}^{|\mathcal{V}|}$, but its width is $|\mathcal{V}|$ rather than a number you picked, and it places every pair of distinct levels at the same distance,

$$\lVert \mathbf{e}_j - \mathbf{e}_k \rVert_2 = \sqrt{2} \quad \text{for all } j \neq k$$

so it encodes identity and nothing else. Fixing $d$ independently of $|\mathcal{V}|$ makes $e$ a compression rather than a relabelling, and it forces the $d$ coordinates to be spent on whatever distinctions the fitting objective rewards.

The $|\mathcal{V}| \times d$ table of coordinates is a parameter block, trained by the same gradient that trains everything downstream of it. Bengio, Ducharme, Vincent and Jauvin (JMLR 3, 1137 to 1155, 2003) set out the arrangement that is still in use, a model that learns "simultaneously (1) a distributed representation for each word along with (2) the probability function for word sequences, expressed in terms of these representations", which is where the term *distributed representation* comes from and why the coordinates are described as learned rather than designed. That the resulting geometry is not an accident is a measured claim: Mikolov, Yih and Zweig (NAACL-HLT 2013, 746 to 751) report that a relation between two words shows up as a roughly constant vector offset across pairs, so that the vector for *king* minus *man* plus *woman* lands near the vector for *queen*, and they score that arithmetic on syntactic and semantic analogy sets rather than on examples.

**Positional encoding, written out.** Where the items being embedded are positions in a sequence and the frequencies are fixed rather than fitted, the map has a closed form. With $pos$ the position and $i$ indexing coordinate pairs,

$$PE_{(pos,\,2i)} = \sin\!\left(\frac{pos}{10000^{2i/d}}\right), \qquad PE_{(pos,\,2i+1)} = \cos\!\left(\frac{pos}{10000^{2i/d}}\right)$$

with wavelengths running in geometric progression from $2\pi$ to $10000 \cdot 2\pi$ (Vaswani et al., NeurIPS 2017). The vector is added to the item's own embedding, which is why the two have to share the width $d$.

**Why this is a Fourier feature and not an analogy.** Write $\omega_i = 10000^{-2i/d}$, so coordinate pair $i$ is $(\sin \omega_i pos, \cos \omega_i pos)$, a single sinusoid at a fixed frequency. The whole encoding is therefore a bank of $d/2$ Fourier basis functions evaluated at the position. Take the inner product of two such vectors and the angle-sum identity collapses it,

$$PE_{pos}^{\top} PE_{pos+k} = \sum_{i=0}^{d/2 - 1} \Big[ \sin(\omega_i pos)\sin\big(\omega_i (pos+k)\big) + \cos(\omega_i pos)\cos\big(\omega_i (pos+k)\big) \Big] = \sum_{i=0}^{d/2 - 1} \cos(\omega_i k)$$

which depends on the offset $k$ alone and not on where in the sequence the pair sits. That is exactly the shift-invariant form a random Fourier feature map is built to produce. Rahimi and Recht (NeurIPS 2007) define the general family: draw $\omega$ from $p$, the Fourier transform of a shift-invariant kernel $k$, draw $b$ uniformly from $[0, 2\pi]$, set $z_\omega(\mathbf{x}) = \sqrt{2}\cos(\omega^{\top}\mathbf{x} + b)$, stack $D$ of them and normalise by $\sqrt{D}$, and the inner product $z(\mathbf{x})^{\top} z(\mathbf{y})$ is an unbiased estimate of $k(\mathbf{x} - \mathbf{y})$. The sinusoidal encoding is the special case in which the frequencies are laid on a fixed geometric grid instead of sampled, and the input is a scalar position instead of a feature vector. Tancik et al. (NeurIPS 2020) run the same map on real-valued coordinates, $\gamma(\mathbf{v}) = [\cos(2\pi \mathbf{B}\mathbf{v}), \sin(2\pi \mathbf{B}\mathbf{v})]^{\top}$ with the entries of $\mathbf{B}$ drawn from a Gaussian, and cite Rahimi and Recht as the origin. What a Fourier feature buys, kernel approximation with a controllable bandwidth, is not treated here; that belongs with kernel methods, which this vault has not reached.

### Positional embeddings

Order has to be supplied explicitly to any model whose layers are indifferent to the order of their inputs. Four constructions, and the first is the one that does not work.

**The raw index.** Feed $pos \in \{0, 1, 2, \dots\}$ as a single numeric feature. The failure is that an index is an unbounded magnitude and a model reads it as one: to a linear layer, position $500$ is five hundred times position $1$, a ratio that means nothing, and the differences between adjacent positions are swamped at the far end of a long sequence. Any position past the longest sequence seen in training also falls outside the range the weights were ever fitted on.

**Fixed discrete embeddings.** The sinusoidal scheme above. Every coordinate is bounded in $[-1, 1]$ regardless of $pos$, so the magnitude problem is gone; the vector is computed from a formula rather than stored, so it costs no parameters and can be evaluated at a position longer than anything trained on. Vaswani et al. chose it for the offset property derived above, that a shift by $k$ acts on $PE_{pos}$ in a way that does not depend on $pos$.

**Learned discrete embeddings.** A lookup table $\mathbf{P} \in \mathbb{R}^{L \times d}$ holding one trainable row per position, initialised at random and fitted with the rest of the model, which is ordinary [[Feature Engineering|feature engineering]] machinery pointed at the index instead of at a category. It costs $Ld$ parameters and hard-caps usable sequence length at $L$, since position $L+1$ has no row. Gehring, Auli, Grangier, Yarats and Dauphin (ICML 2017) used exactly this, adding an absolute position embedding to each word embedding, and it predates the sinusoidal form; Sukhbaatar, Szlam, Weston and Fergus (NeurIPS 2015) carry an earlier version still, a learned matrix $T_A$ whose $i$-th row is added to the $i$-th memory vector. Vaswani et al. tried the learned table against their own sinusoids and report that the two produced nearly identical results, so the fixed scheme is chosen for its extrapolation and its zero parameter cost rather than for accuracy.

**Continuous embeddings.** The thing being embedded is a real-valued coordinate rather than an integer index: a time in seconds, a point in space, an angle. There is no table to build, because there is no finite set of inputs to enumerate, so the representation has to be a function evaluated at the coordinate, which is what $\gamma(\mathbf{v})$ above is.

The split between discrete and continuous is that one question: whether the thing being embedded comes from a countable set you can list and tabulate, or from a continuum where you can only evaluate. Discrete admits both a stored table and a formula; continuous admits only a formula. Note that the two cuts are independent, since fixed against learned asks where the coordinates come from and discrete against continuous asks what the input is, and the sinusoidal scheme sits at fixed-and-discrete while $\gamma$ sits at fixed-and-continuous.

## Where it is used

[[Dimensionality Reduction]] is the vault's other route to a short dense vector, and the two are not the same operation. Reduction starts from an existing numeric feature space and projects it onto fewer axes, so the information is already in the matrix and the method chooses what to discard. An embedding starts from items that carry no numeric representation at all, a word or a ZIP code or a position, and fits coordinates for them against a downstream objective, so the information arrives from the task rather than from the input.

That distinction is worth holding onto because the tooling invites confusion. **scikit-learn has no embedding API.** Nothing in version 1.6 produces a learned dense representation of a category or of a position: `PCA` and `TruncatedSVD` are reduction, taking a numeric matrix in and returning a narrower numeric matrix, and reaching for either as a substitute answers a different question. Embeddings are fitted in a deep learning framework or loaded from a model trained elsewhere.

[[One-Hot Encoding]] is the encoding an embedding replaces once cardinality is high. That note names the failure directly, a ZIP code column with thousands of levels becoming thousands of near-empty columns with too few instances per level to estimate anything, and a learned embedding is one of the two standard answers to it. [[Feature Hashing]] is the other, and the two differ in what they fix the width with: hashing pins the output width by a hash function chosen in advance and accepts collisions, while an embedding pins it by the parameter block and fits what goes in each coordinate.

[[Zero-Shot Learning]] and [[Few-Shot Learning]] both rest on a space of this kind. Zero-shot scores a compatibility $\langle \phi(\mathbf{x}), s(y) \rangle$ between an input mapped into a shared semantic space and a class description living in that same space, which works only because the class representation and the input representation are vectors of the same width with a meaningful inner product. Few-shot averages the support items of a class into a prototype $\mathbf{c}_k$ and classifies by distance to it, which works only because averaging and distance mean something in the space $f_\phi$ maps into. Both are cases of the definition above taken seriously: the geometry is the mechanism, not a visualisation of it.

[[Feature Engineering]] is the parent activity, and an embedding is its extraction branch, pulling a numeric representation out of a raw signal that had none. The map $\phi$ that note composes with the hypothesis is the same object here, with the difference that $\phi$ carries fitted parameters of its own and is trained rather than specified.
