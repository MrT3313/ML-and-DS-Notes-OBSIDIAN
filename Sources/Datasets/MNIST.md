---
note_kind: dataset
medium: dataset
author: Yann LeCun, Corinna Cortes, Christopher J.C. Burges
year: 1998
url: https://www.openml.org/d/554
up: "[[Home]]"
aliases:
  - MNIST
  - mnist
  - mnist_784
  - MNIST-784
  - MNIST 784
  - the MNIST dataset
  - MNIST database
  - MNIST database of handwritten digits
  - handwritten digits
  - handwritten digit dataset
  - digits dataset
  - fetch_openml
  - OpenML 554
  - mnist_784.arff
---

# MNIST

70,000 images of handwritten digits, each a 28 by 28 grayscale square flattened into 784 numbers and carried alongside the digit it shows. It is the file [[HOML Ch03 Classification]] works on from its first page to its last, the only dataset behind that chapter's whole treatment of [[Classification]], so every accuracy, precision and recall figure in this vault's chapter 3 notes was computed here. Every count below was checked against the copy `scikit-learn` 1.6.1 actually downloads rather than recalled, and the two things that could not be checked are marked as such where they appear.

## Provenance

Yann LeCun, Corinna Cortes and Christopher J. C. Burges assembled MNIST, and the benchmark was introduced in LeCun, Bottou, Bengio and Haffner, *Gradient-Based Learning Applied to Document Recognition*, **Proceedings of the IEEE** 86(11), 2278-2324, 1998. The files themselves carry no documented release date, and OpenML records the source date as unknown, so the 1998 in this note's frontmatter is the year of that paper and not a verified release date for the data.

MNIST is not an original collection. It is a modified redistribution of two earlier NIST collections, Special Database 3 and Special Database 1, and every one of the changes matters for what a model trained on it can claim.

- **NIST's own split was not usable as a benchmark.** NIST designated SD-3 the training set and SD-1 the test set, but SD-3 was collected from Census Bureau employees and SD-1 from high-school students, which makes SD-3 much cleaner and much easier. A score computed across that boundary measures the boundary as much as the model. MNIST exists to remove that confound by mixing the two.
- **Both halves draw from both sources.** The 60,000 training images are 30,000 from SD-3 and 30,000 from SD-1. The 10,000 test images are 5,000 from each.
- **The writers were separated, not the images.** SD-1 holds 58,527 images from 500 writers. The writer identities were used to unscramble it, the first 250 writers went to the training half and the remaining 250 to the test half, and the halves were topped up from SD-3 (from pattern 0 for training, from pattern 35,000 for test). No writer appears on both sides of the split, and the training half comes from roughly 250 writers in total.
- **The pixels were rewritten.** The NIST originals are bilevel, pure black on pure white. Each digit was size-normalized to fit a 20 by 20 box with its aspect ratio preserved, and the anti-aliasing inside that resize is where the intermediate grey levels come from: they are an artifact of the normalization, not ink. Each normalized digit was then placed in a 28 by 28 field centered on the center of mass of its pixels. LeCun's own documentation notes that some template-based methods do better when digits are centered by bounding box instead, so the centering rule is a choice baked into the file.

The copy chapter 3 loads is a second redistribution on top of that one: OpenML dataset 554, named `mnist_784`, version 1, uploaded 2014-09-29 and carrying the licence string `Public`. It is stored as a single 785 column ARFF rather than as LeCun's four binary IDX files.

The content is unchanged. Loading OpenML 554 and comparing it against the original distribution element by element, the pixel matrix and the labels are identical, with the training half first and the test half second, both in their original order. What the ARFF container changes is the packaging: the columns acquire the names `pixel1` to `pixel784`, and the label becomes a nominal attribute, which is why the digits arrive as text. The original four files (`train-images-idx3-ubyte.gz` and its three companions) belong to LeCun's MNIST page at http://yann.lecun.com/exdb/mnist/; that host has been intermittently unavailable, so the comparison above was run against the mirror at https://ossci-datasets.s3.amazonaws.com/mnist/.

## Columns

785 columns: 784 features and one label. Listing one row per column would produce 784 identical rows, so the table is grouped by column group instead, with the 784 pixel features described once. That is a deliberate departure from one row per column, and it is safe here only because the features genuinely are homogeneous: same meaning, same units, same dtype, same range.

| column group | count | meaning |
|---|---|---|
| `pixel1` to `pixel784` | 784 | Ink intensity at one position in the image. Integers from 0 (background) to 255 (full ink) over the whole file before any split, with all 256 values occurring. The 784 columns are the 28 by 28 image flattened in row-major order, so `pixel1` is the top-left corner, `pixel28` closes the top row, `pixel29` opens the second row and `pixel784` is the bottom-right corner. `reshape(28, 28)` recovers the picture. Loaded as `int64` by scikit-learn 1.6, not float. See `## Known quirks` below for why that surprises people |
| `class` | 1 | The digit in the image. Ten levels, `'0'` to `'9'`, as **strings**, not integers. The target, and the reason the chapter can run both [[Multiclass Classification]] on ten classes and [[Binary Classification]] on a relabelled version of it |

No column has a missing value anywhere in the 70,000 rows.

The label is close to balanced but not balanced. Counts over the whole file and over each half of the positional split:

| digit | all 70,000 | training 60,000 | test 10,000 |
|---|---|---|---|
| 0 | 6,903 | 5,923 | 980 |
| 1 | 7,877 | 6,742 | 1,135 |
| 2 | 6,990 | 5,958 | 1,032 |
| 3 | 7,141 | 6,131 | 1,010 |
| 4 | 6,824 | 5,842 | 982 |
| 5 | 6,313 | 5,421 | 892 |
| 6 | 6,876 | 5,918 | 958 |
| 7 | 7,293 | 6,265 | 1,028 |
| 8 | 6,825 | 5,851 | 974 |
| 9 | 6,958 | 5,949 | 1,009 |

Ones are the most common digit and fives the rarest, at 7,877 and 6,313 over the whole file. That is a spread of about 1.25 to 1, which is mild enough to ignore for the ten-class problem.

## Known quirks

Seven things about this file are invisible in the schema and change what a model learns or what a number means.

- **The 5-versus-rest target is badly imbalanced, and that is the whole point of the chapter's accuracy argument.** 5,421 of the 60,000 training rows are fives and 54,579 are not, so fives are 9.035 percent of the training half. A model that answers "not a five" for every image is right 90.965 percent of the time without looking at a single pixel. That figure is reproducible: `DummyClassifier` scores exactly 0.90965 on each of the chapter's three cross-validation folds. It is the number that makes [[Accuracy]] useless here and forces the chapter into [[Precision]] and [[Recall]]. See [[Class Imbalance]].
- **Most of the file is zero.** 80.86 percent of the 54,880,000 pixel entries are exactly 0. 65 of the 784 columns are 0 in all 70,000 rows, and 67 are 0 in all 60,000 training rows, so those columns are constant and carry no information whatsoever. A further 290 columns are zero in at least 99 percent of rows and 387 in at least 95 percent. This bites during [[Feature Scaling]]: a constant column has zero variance, and [[Standardization]] would divide by zero (scikit-learn's `StandardScaler` substitutes a scale of 1 and leaves the column at zero rather than raising), while a near-constant column has a tiny standard deviation and its handful of nonzero values get amplified out of proportion.
- **The images are grayscale, not black and white.** All 256 integer values from 0 to 255 occur. The NIST source images were bilevel and the grey came from anti-aliasing during the 20 by 20 resize, so thresholding back to two values is a modelling decision you are making, not a return to the original.
- **The split is positional, not random.** Rows 0 to 59,999 are the [[Training Set]] and rows 60,000 to 69,999 the [[Testing Set]], and the chapter takes them with plain slicing rather than with a randomized splitter. That is correct here rather than lazy, because the boundary is the writer boundary described in Provenance: the two halves have disjoint writers, and a random re-split would put the same hand on both sides and inflate every test score.
- **The file is in its original order and nothing permutes it, but the folds are still sound.** LeCun's construction already interleaved the classes, so the file is not sorted by digit: the longest run of identical consecutive labels anywhere in the 70,000 rows is 5. A plain `KFold(n_splits=3)` on the training half, with no shuffling at all, gives three held-out folds containing every one of the ten classes, holding 1,775, 1,829 and 1,817 fives. The chapter's notebook flags the general hazard rather than a problem with this file, in a code comment on its `StratifiedKFold` loop reading, quoting the notebook exactly, `add shuffle=True if the dataset is not already shuffled`. Whether the book's prose also states that the training set arrives shuffled is **unverified**: the third edition notebook contains no such sentence and the book text is paywalled, so check the printed chapter around the `X_train, X_test` line if the claim matters.
- **The labels are strings.** `y.dtype` is `object` and the values are `'5'`, not `5`. Every comparison in the chapter needs the quotes, as in `y_train == '5'`, and a silent `y_train == 5` returns all-`False` with no error and no warning. Cast with `y.astype(np.uint8)` when you want arithmetic on them.
- **Rows are not independent.** 60,000 training images come from roughly 250 writers, so a few hundred hands account for the whole training half and many rows share one. Any resampling that assumes independent rows is assuming something the file does not provide.

## Getting it

Chapter 3 does not read LeCun's original files. It fetches the OpenML copy, dataset id **554**, name `mnist_784`, version 1, at **https://www.openml.org/d/554**, which is the URL scikit-learn itself reports back in `mnist.url`.

scikit-learn 1.6:

```python
from sklearn.datasets import fetch_openml

mnist = fetch_openml('mnist_784', as_frame=False)

X, y = mnist.data, mnist.target      # (70000, 784) int64, (70000,) object of str
X_train, X_test, y_train, y_test = X[:60000], X[60000:], y[:60000], y[60000:]

y_train_5 = (y_train == '5')         # the binary target the chapter opens with
y_test_5 = (y_test == '5')

some_digit = X[0]                    # the first image, a five
some_digit.reshape(28, 28)           # back to a picture
```

Four things about that call are worth knowing in 1.6.

- **Name resolution.** `version` defaults to `'active'`, and `mnist_784` has exactly one active version, id 554, so `fetch_openml('mnist_784')` and `fetch_openml(data_id=554)` return the same file today. Pinning `data_id=554` is the more durable spelling, and it is what the second edition's notebook spelled out as `version=1`, an argument the third edition dropped.
- **`as_frame` defaults to `'auto'`**, which for this dense dataset means you get a pandas `DataFrame` and `Series`. `as_frame=False` is what makes `mnist.data` a NumPy array, and the chapter needs that: `X[0]`, `X[:100]` and `reshape(28, 28)` are array operations.
- **`parser` defaults to `'auto'`, and has since 1.4**, where it was `'liac-arff'` before. `'auto'` selects `liac-arff` for sparse ARFF and `pandas` for everything else, and `mnist_784` is dense, so 1.6 uses the pandas parser and pandas must be installed. That parser change is why `X` now arrives as `int64` while older printed outputs, including the ones saved in the chapter's own notebook, show `float64`. Nothing is wrong; the chapter casts with `X_train.astype("float64")` before scaling anyway. At `int64` the full 70,000 by 784 array occupies about 439 MB in memory, and the 60,000 row training half about 376 MB.
- **The download is cached** under `~/scikit_learn_data`, about 15 MB gzipped, so only the first call reaches the network.

The raw ARFF sits at https://openml.org/data/v1/download/52667/mnist_784.arff with md5 `0298d579eb1b86163de7723944c7e495`, and OpenML also publishes a Parquet copy at https://data.openml.org/datasets/0000/0554/dataset_554.pq. Neither is what you want for following the chapter; `fetch_openml` handles both the download and the parse.

> [!warning]
> `sklearn.datasets.load_digits` is a **different dataset** and is not this file. It holds 1,797 images at 8 by 8 rather than 70,000 at 28 by 28, so 64 features rather than 784, with integer intensities running 0 to 16 rather than 0 to 255. It ships inside scikit-learn and needs no download, which is exactly why it gets reached for by mistake. It comes from the UCI optical recognition set, built from NIST forms by an entirely different route: 32 by 32 bitmaps reduced by counting the on-pixels in each non-overlapping 4 by 4 block. The ten classes are the only thing the two files share, and no number in chapter 3 reproduces on it.
