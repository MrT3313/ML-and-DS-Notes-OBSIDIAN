---
note_kind: method
aliases:
  - data augmentation
  - augmentation
  - augment
  - augmented data
  - training set augmentation
  - training set expansion
  - synthetic training data
  - image augmentation
  - shifted copies
up: "[[Training Set]]"
sources:
  - "[[HOML Ch03 Classification]]"
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Data augmentation grows the [[Training Set]] by generating new instances from the ones you already have, without collecting anything and without paying for a label: either by applying a transformation directly to an instance, which changes the input while leaving the label correct, or, at one remove, by fitting a generator to the instances you hold and drawing new ones from it. It is therefore the cheapest available answer to [[Insufficient Training Data]], and because the model now sees each original under several presentations instead of one, it is a direct attack on [[Overfitting]]: capacity that would otherwise memorise the exact pixel layout of one image has to account for four more that carry the same label. Reach for it when instances are scarce or when a class the model confuses is under-represented, which is the diagnosis [[Error Analysis]] produces. Its sibling remedy is [[Feature Engineering]], which changes *what* the model sees rather than *how much* of it.

The family has three branches, and they differ in how far the generated instance stands from a real one:

- **Label-preserving transformations** modify an instance directly and keep its label attached. On images that is cropping, flipping, rotating, inverting, erasing part of the picture, and the one-pixel shifts worked through below. This branch is what the rest of the note is about.
- **[[Perturbation]]** adds a small deliberate change rather than a named geometric transformation, and is label preserving in exactly the same way. It carries a note of its own because its copies are also precisely what an attacker constructs to push a model into answering wrongly, so the adversarial material, and the way the same operation reappears inside [[Semi-Supervised Learning]] with a different purpose, lives there rather than here.
- **Synthesis** does not start from one individual instance at all. It is set out at the end of this section.

The first two branches rest on one condition, and it is a condition about the world rather than about the code. **The transformation has to be label preserving, and it has to be a variation that actually occurs in the data the model will meet.** Shift a handwritten digit one pixel and it is still that digit, so the copy is a legitimate training instance. Rotate a $6$ far enough and it is a $9$, so the copy carries a lie. Mirror a photograph of a cat and you get a cat; mirror a photograph of a page of text and you get nothing anyone will ever photograph. The second half of the condition is the half that gets dropped: a transformation that is safely label preserving but never occurs in reality still costs its full share of training time, and all it buys is invariance to something the model will never encounter.

Why the model needs telling at all is worth being explicit about. A linear classifier over raw pixels carries one weight per pixel position, so the pixel at index $j$ and the pixel at index $j+1$ are unrelated coordinates as far as the hypothesis is concerned; there is nothing in the parameterisation that says a shifted image is the same image. A k-nearest-neighbours classifier, the model refitted in the worked run below, is no better off: it scores similarity by Euclidean distance between raw 784-vectors, and shifting a digit one pixel moves it a long way in that metric. Augmentation teaches the invariance by example, one copy at a time. The alternative is to build the invariance into the architecture so it holds by construction rather than by demonstration, which is what convolution does and what the later network chapters are for.

The technique goes by two names, **data augmentation** and **training set expansion**, quoted together in [[HOML Ch03 Classification|HOML chapter 3]] as "data augmentation or training set expansion". How much weight that source gives it is worth recording once, because it is easy to misread: it appears there only as exercise 2 and its solution, headed `## 2. Data Augmentation` in the notebook, and not in the chapter's prose, with the [[Error Analysis]] material that most obviously motivates it carrying no augmentation text at all. The worked run is on [[MNIST]], and it refits the [[Classification]] model tuned by the preceding exercise on the enlarged set.

### Synthesis

The third branch sidesteps data collection altogether. Instead of transforming instances one at a time, fit a generative model to the data you hold and sample new instances out of it, or build a simulator of the process that produced the data and render instances from that. The dependence on real instances is still there, which is what keeps synthesis inside this family rather than outside it, but the dependence is mediated by a fitted model or a hand-built simulator instead of being applied to one row at a time. That is the clause the opening paragraph carries, and it is the whole of the difference: a shifted digit is a real digit moved, and a sampled digit is a draw from something fitted to real digits.

How well it works is the part of this subsection that goes stale fastest, so it is dated rather than stated flat. Writing in 2022, [[DMLS Ch04 Training Data|Huyen]] puts synthesis as very far from working at scale on many problems. **As of September 2026 that blanket claim no longer holds uniformly.** What replaces it is not the opposite claim but a split one: synthesis is routine in some settings and still unreliable in others, and a failure mode that had no name in 2022 now has one.

Where it is established:

- **Simulation for perception.** In robotics and autonomous driving a renderer supplies images together with their labels for free, since the generator already knows what it drew, and it supplies dangerous edge cases that nobody wants to stage in reality. The practice that is actually reported to work is hybrid rather than synthetic-only: pretrain on the simulated set, fine-tune on a smaller real one, because the real data is what carries sensor noise, lighting, material response and the long tail of things the simulator was never told about. The residual gap between the two distributions is the sim-to-real gap, and it is the reason the real fine-tuning step has not gone away.
- **Generated text in language model training.** Training partly on text produced by another model is ordinary practice, both as distillation of a large model into a smaller one and as deliberately generated instructional text mixed with filtered real text. The Phi models from Microsoft, from 2023 onward, are the best documented case of the second, and they reached capability at a parameter count well below what the comparable models trained on unfiltered web text needed.
- **Privacy-constrained domains**, where a synthetic record stands in for a real one that cannot be shared. This is the weakest of the three on evidence: the mechanics are routine, and whether a given synthetic set preserves the utility of the original while genuinely protecting the individuals in it is decided per release, not by the technique.

The failure mode that cuts the other way is **model collapse**. Shumailov, Shumaylov, Zhao, Papernot, Anderson and Gal (*Nature* 631, 755, 2024) show that a generative model trained on the output of the previous generation of itself, repeatedly, loses the tails of the original distribution first and degrades across the whole of it after, and they show it holds for large language models, variational autoencoders and Gaussian mixture models alike. The effect matters because scraped training data is no longer reliably human-written.

The evidence on how inevitable that is, is genuinely mixed, and the disagreement is specific enough to state. Gerstgrasser and co-authors (2024) report that the collapse result depends on each generation's data *replacing* its predecessor's: when the synthetic generations *accumulate* alongside the original real data instead, the test error is bounded rather than growing with the number of iterations. So the defensible 2026 statement is narrower than the headline. Recursive training on generated data collapses when the real data is thrown away, and keeping the real data in the mixture is what prevents it, which makes synthesis a question of what fraction of the training set it may be rather than a question of whether it works at all.

## Algorithm

Write the training set as $D_{\text{train}} = \{(\mathbf{x}^{(i)}, y^{(i)})\}_{i=1}^{m}$ and the chosen transformations as $\mathcal{T} = \{t_1, \dots, t_k\}$, each $t_j$ a map on inputs alone.

1. **Split first.** Hold out the [[Testing Set]], and fix the cross-validation folds, using only original instances. Nothing has been generated yet at this point, and that ordering is not negotiable (see step 7).
2. **Choose $\mathcal{T}$.** Every $t_j$ must satisfy $y(t_j(\mathbf{x})) = y(\mathbf{x})$ for every $\mathbf{x}$ in the data, not merely for the examples you happened to look at, and must correspond to a variation the deployment data exhibits.
3. **Generate.** For each training instance and each $t_j$, emit $(t_j(\mathbf{x}^{(i)}), y^{(i)})$. The label is copied across unchanged; that is the entire point.
4. **Concatenate, do not replace.** The augmented set is the originals plus the copies,
   $$D_{\text{aug}} = D_{\text{train}} \cup \{(t_j(\mathbf{x}^{(i)}), y^{(i)})\}_{i \le m,\, j \le k}, \qquad |D_{\text{aug}}| = (k+1)\,m$$
   With $m = 60{,}000$ MNIST training images and $k = 4$ one-pixel shifts (left, right, up, down), that is $5 \times 60{,}000 = 300{,}000$ instances.
5. **Shuffle.** The copies come out grouped by transformation, all the left-shifts in one block and all the right-shifts in the next. Any estimator that consumes the rows in order, a [[Stochastic Gradient Descent Classifier]] above all, would see one direction at a time. One `np.random.permutation` over the augmented arrays before fitting is enough to break the blocks up.
6. **Fit on $D_{\text{aug}}$, score on the untouched test set.** The test set is never augmented. Augmenting it would change the question being asked rather than the answer being given, and a score on transformed test images is a score on data nobody is going to send you.
7. **Keep the bookkeeping.** Record, for each generated row, the index of the original it came from.

Step 7 exists because of step 1, and the reason deserves stating plainly. **Augmenting before the split puts transformed copies of a single original on both sides of the boundary.** The test set then contains near duplicates of training instances, the model is being asked to recognise images it has effectively already seen, and the reported score is inflated by an amount nobody can measure afterwards. That is [[Data Snooping Bias]] in one of its least visible costumes: no analyst ever looked at the test set, and the leak happened anyway, mechanically, in a list comprehension.

The same non-independence survives into [[Cross-Validation]]. The $k+1$ rows descended from one original are not $k+1$ independent draws, so a plain `KFold` over $D_{\text{aug}}$ will routinely leave a copy in the validation fold while its four siblings sit in the training folds. A nearest-neighbour model scores close to perfectly on that arrangement and has learned nothing. Group the copies with their original and split on the group, which is what `GroupKFold` and `StratifiedGroupKFold` are for. MNIST carries a coarser version of the same hazard before any augmentation happens, since its 60,000 training images come from roughly 250 writers; the file's own positional split at row 60,000 is a writer boundary, which is why slicing at that row is correct and a random re-split would not be.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| transformation set | $\mathcal{T}$ | none, augmentation is opt-in | each transformation added teaches one more invariance and adds $m$ rows; each also has to be argued label preserving on its own | include only variations the deployment data actually shows, and drop any whose label preservation you cannot state as a rule |
| shift magnitude | $\delta$ | 1 pixel on MNIST | copies drift further from the original, so the invariance taught is broader; past the frame's margin the transform clips ink against the border and the label stops being safe | bound it by real variation. MNIST digits are normalized into a 20 by 20 box inside a 28 by 28 field, leaving roughly four pixels of margin per side, so single-digit pixel counts are the honest range |
| rotation angle | $\theta$ | not used on digits | small angles are plausible handwriting variation; large ones destroy the label, since a rotated $6$ becomes a $9$ | the hard ceiling is the angle at which any class maps onto another, not a number found by search. On digits that ceiling is low enough to leave rotation out entirely |
| copies per original | $k$ | 4 on MNIST, one shift per direction | the fitted set grows to $(k+1)m$ and training cost grows with it, while the added rows are new presentations of information already present rather than new information | raise until the held-out score stops moving, then stop. Beyond that point you are paying $m$ rows per increment for nothing |
| resampling policy | | once, up front | resampling a fresh random transform per epoch means the model never sees the same copy twice, which regularizes harder than a fixed enlarged set of the same nominal size; it also makes the training set non-reproducible without a [[Random Seed]] | fixed and materialized for a one-shot fit; resampled per epoch for iterative training, which is what the Keras preprocessing layers named below do |

## Failure modes

- **The transformation changes the true label.** Rotating MNIST digits far enough turns a $6$ into a $9$; mirroring turns a $2$ into something no one writes. Horizontal flips are safe on photographs of cats and unsafe on photographs of text, on road signs, and on anything chiral, where the mirror image is a different object rather than the same object seen differently. The failure is silent: training proceeds, the loss goes down, and the model has been taught that two classes are the same class.
- **Augmenting before the split, and splitting without grouping.** Generate copies over the full dataset and then split, and transformed near duplicates of test instances end up in training, which inflates the test score by an unmeasurable amount ([[Data Snooping Bias]]). Split correctly but then run `KFold` over the augmented training array, and the same leak reappears one level down, with each held-out copy sitting next to four siblings in the training folds.
- **Augmentation that does not match real variation.** MNIST images are already size-normalized and centered on the center of mass of their pixels, so aggressive scale or translation jitter teaches invariance to a variation the file does not contain. The copies are label preserving and still worthless: they consume the full training cost of $m$ extra rows per transformation and improve nothing.
- **Shifting past the margin.** With `cval=0` and `mode="constant"` the border is filled with background, so a shift larger than the digit's margin clips ink off the edge. The copy is a truncated digit that still carries the original label, which is the first failure mode arriving by degrees rather than all at once.
- **The cost is real and the information gain is not.** $k$ copies per instance multiply the fitted set by $k+1$, and roughly multiply training time by the same factor, while adding no genuinely new information, only new presentations of information already in the file. The MNIST run shows the trade honestly: quintupling the training set moves test accuracy from $0.9714$ to $0.9763$, less than half a point. For the k-nearest-neighbours model it refits, the bill is worse than that, because a k-NN query scans the stored training set, so the fivefold growth lands on *prediction* cost too, which is why scoring that fit runs into minutes.
- **Forgetting to shuffle.** Copies are appended in blocks by transformation, so any order-sensitive fit sees all the left-shifts before any of the right-shifts.

## Implementation

**scikit-learn has no data augmentation API.** There is no transformer in scikit-learn 1.6 that generates augmented copies: `sklearn.preprocessing` transforms rows one for one and never emits more rows than it was given, and the `sklearn.datasets.make_*` family fabricates datasets from distributions you specify, with no real instances involved at any point, which is not the synthesis branch either, since that fits its generator to data you actually hold. Augmentation is written by hand, or comes from another library. Oversampling a minority class with SMOTE is the nearest ready-made thing, and it lives in the separate `imbalanced-learn` package, not in scikit-learn.

Shifting an image by whole pixels is `scipy.ndimage.shift`. SciPy 1.14:

```python
import numpy as np
from scipy.ndimage import shift

def shift_image(image, dx, dy):
    image = image.reshape((28, 28))
    shifted_image = shift(image, [dy, dx], cval=0, mode="constant")
    return shifted_image.reshape([-1])
```

The signature is `shift(input, shift, output=None, order=3, mode='constant', cval=0.0, prefilter=True)`. The shift vector is given in array-index order, so it reads `[rows, columns]`, which is `[dy, dx]` and not `[dx, dy]`; `mode="constant"` with `cval=0` fills the vacated border with background instead of wrapping ink around from the far edge. `order=3` is cubic spline interpolation by default, and it is the parameter to think about only when the displacement is fractional or the transform is a rotation, because those are the cases where the interpolator has to invent intermediate values rather than move existing samples.

Building and shuffling the enlarged arrays, then refitting. scikit-learn 1.6:

```python
X_train_augmented = [image for image in X_train]
y_train_augmented = [label for label in y_train]

for dx, dy in ((-1, 0), (1, 0), (0, 1), (0, -1)):        # left, right, up, down
    for image, label in zip(X_train, y_train):
        X_train_augmented.append(shift_image(image, dx, dy))
        y_train_augmented.append(label)

X_train_augmented = np.array(X_train_augmented)          # (300000, 784)
y_train_augmented = np.array(y_train_augmented)

shuffle_idx = np.random.permutation(len(X_train_augmented))
X_train_augmented = X_train_augmented[shuffle_idx]
y_train_augmented = y_train_augmented[shuffle_idx]

from sklearn.neighbors import KNeighborsClassifier
knn_clf = KNeighborsClassifier(n_neighbors=4, weights="distance")
knn_clf.fit(X_train_augmented, y_train_augmented)
knn_clf.score(X_test, y_test)                            # 0.9763
```

Those settings are not arbitrary: `n_neighbors=4, weights="distance"` is what a `GridSearchCV` had already selected over `n_neighbors` in $\{3, 4, 5, 6\}$ and `weights` in `{"uniform", "distance"}`, searched on the first 10,000 images.

The measured numbers, on the 10,000 image test set:

| model | test accuracy | test error rate |
|---|---|---|
| `KNeighborsClassifier()` with defaults, on the 60,000 originals | $0.9688$ | $0.0312$ |
| tuned, on the 60,000 originals | $0.9714$ | $0.0286$ |
| tuned, on the 300,000 augmented | $0.9763$ | $0.0237$ |

The accuracy gain is $0.0049$, which sounds like nothing until it is read as a change in the error rate: $0.0237 / 0.0286 - 1 = -0.171$, so about one error in six disappears. [[HOML Ch03 Classification|HOML chapter 3]]'s solution notebook makes exactly this move, writing "By simply augmenting the data, we've got a 0.5% accuracy boost. Perhaps it does not sound so impressive, but it actually means that the error rate dropped significantly" and then printing `error_rate_change = -17%`. Reporting the error rate rather than the accuracy is the right habit whenever accuracy is already near $1$, because the headroom, not the score, is what improved.

Keeping the group bookkeeping from step 7 of the algorithm, so that a later cross-validation cannot split a family. scikit-learn 1.6:

```python
from sklearn.model_selection import StratifiedGroupKFold

groups = np.tile(np.arange(len(X_train)), 5)   # copy i of original j carries group j
groups = groups[shuffle_idx]                   # permute alongside X and y

cv = StratifiedGroupKFold(n_splits=3)          # every copy of an original lands in one fold
```

In the deep learning chapters the route is different in form and identical in principle: Keras preprocessing layers such as `RandomTranslation`, `RandomRotation` and `RandomFlip` sit inside the model and resample a fresh transform every epoch rather than materializing a fixed enlarged array, which is the resampling policy row of the table above turned on. That is chapter 14's material and is out of scope here.
