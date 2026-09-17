---
note_kind: dataset
medium: dataset
author: Edgar Anderson, R. A. Fisher
year: 1936
url: https://scikit-learn.org/1.6/datasets/toy_dataset.html#iris-dataset
up: "[[Home]]"
aliases:
  - iris
  - Iris
  - iris dataset
  - Iris dataset
  - iris flower dataset
  - iris plants dataset
  - Fisher's iris
  - Fisher iris
  - Anderson's iris
  - load_iris
  - sklearn iris
  - iris.csv
  - iris.data
  - bezdekIris.data
---

# Iris

150 iris flowers, four measurements of each in centimetres, labelled with which of three species the plant is. It is the smallest file this vault works on, 2,734 bytes, and it ships inside scikit-learn rather than being downloaded. [[HOML Ch04 Training Models]] reaches for it at the point where the chapter turns from regression to [[Classification]], and every [[Decision Boundary]] picture in that chapter is drawn on two of its four columns, because two columns are what a flat picture has room for.

## Provenance

The measurements are Edgar Anderson's and the analysis is R. A. Fisher's, and the two are routinely collapsed into one name. Anderson, a botanist, measured the plants; Fisher printed the table and built [[Classification|discriminant analysis]] on it in R. A. Fisher, *The use of multiple measurements in taxonomic problems*, **Annals of Eugenics** 7(2), 179-188, 1936. Fisher's own sentence introducing the data, on page 179, credits the measurements to Anderson explicitly. Anderson published his own account of the collection as *The irises of the Gaspé Peninsula*, 1935. The 150 rows are Fisher's Table I on page 180, in the column order still used today and grouped fifty setosa, fifty versicolor, fifty virginica.

The dataset reached machine learning through the UCI Machine Learning Repository, which lists Fisher as the creator and does not mention Anderson at all. That is where the single-name attribution comes from.

**Two rows of the UCI copy do not match Fisher's paper**, and this is the one thing about the provenance that changes numbers. UCI's own `iris.names` records the discrepancy, credited to Steve Chadwick:

- sample 35 should be `4.9, 3.1, 1.5, 0.2` and the UCI file has `0.2` as `0.1`, one wrong value in the fourth column.
- sample 38 should be `4.9, 3.6, 1.4, 0.1` and the UCI file has `4.9, 3.1, 1.5, 0.1`, two wrong values in the second and third columns.

UCI ships both versions in the same archive. `iris.data` is the copy with the errors; `bezdekIris.data` is the corrected copy, and the two differ in exactly those two rows and nowhere else. scikit-learn shipped the erroneous copy until version 0.20, whose changelog entry on `load_iris` reads "Fixed two wrong data points according to Fisher's paper. The new version is the same as in R, but not as in the UCI Machine Learning Repository." From 0.20 onward, and so in 1.6, the bundled `iris.csv` is the corrected copy: it agrees with `bezdekIris.data` on all 150 rows and differs from `iris.data` at samples 35 and 38 and nowhere else. Its per-species means reproduce Fisher's Tables II and VIII to every printed digit, which is the check to run if you want to confirm the file in front of you is his rather than UCI's.

The correction has a consequence the changelog does not mention. Both errors push their row to `4.9, 3.1, 1.5, 0.1`, and sample 10 genuinely reads that, so in the UCI copy samples 10, 35 and 38 are three identical rows. Restoring Fisher's values breaks the triple apart. The uncorrected file holds three redundant rows against the scikit-learn file's one, so a deduplication step behaves differently on the two copies.

## Columns

Five columns: four numeric features and the species label. Every measurement is a length in centimetres recorded to one decimal place. Nothing in the raw file says so, because the CSV has no column names at all, only a first line giving `150,4,setosa,versicolor,virginica`; the names and the units are supplied by `load_iris`, which is the only place the `(cm)` in the feature names comes from. Ranges are over all 150 rows, before any split. Standard deviations are the sample form, dividing by $m - 1$, which is what `pandas` returns by default; the population form differs in the third decimal at this size.

| column | meaning | units | range over all 150 rows |
|---|---|---|---|
| `sepal length (cm)` | length of a sepal, the outer leaf that encloses the bud before it opens | centimetres, one decimal place | 4.3 to 7.9, mean 5.843, sd 0.828, 35 distinct values |
| `sepal width (cm)` | width of the same sepal | centimetres, one decimal place | 2.0 to 4.4, mean 3.057, sd 0.436, 23 distinct values |
| `petal length (cm)` | length of a petal | centimetres, one decimal place | 1.0 to 6.9, mean 3.758, sd 1.765, 43 distinct values |
| `petal width (cm)` | width of the same petal | centimetres, one decimal place | 0.1 to 2.5, mean 1.199, sd 0.762, 22 distinct values |
| `target` | the species. The label, and the reason the file supports both [[Binary Classification]] and [[Multiclass Classification]] | integer code, not a string | `0` setosa, `1` versicolor, `2` virginica, 50 rows each. `iris.target_names` maps the codes back to names |

One measurement per plant, not an average over the flowers on it, which is part of why the last two species overlap as much as they do.

## Known quirks

These things about this file are invisible in a schema and change what a model learns or what a score means.

- **Setosa is linearly separable from the other two; versicolor and virginica are not separable from each other.** Setosa's petal length runs 1.0 to 1.9 cm and every one of the other 100 rows is 3.0 or above, so a 1.1 cm band contains zero rows; petal width gives the same picture with a 0.4 cm band between setosa's 0.6 maximum and the others' 1.0 minimum. Versicolor and virginica have no such gap: 21 of the 50 versicolor rows sit at petal length 4.5 or more, inside virginica's range, and 16 of the 50 virginica rows sit at 5.1 or less, inside versicolor's. The best straight line that exists anywhere in the petal length by petal width plane, found by exhaustive search rather than by fitting, still puts 3 of those 100 rows on the wrong side. Fisher says the same of all four measurements at once on page 188, that no certain diagnosis of these two species could rest on a single flower's measurements. One clean boundary and one messy one in the same picture is exactly what makes the file useful for demonstrating [[Logistic Regression]] and [[Softmax Regression]], and useless as evidence that either one is any good.
- **The two petal columns carry nearly all the signal, and they duplicate each other.** Correlation with the integer target is 0.9565 for petal width and 0.9490 for petal length, against 0.7826 for sepal length and -0.4267 for sepal width. The two petal columns correlate 0.9629 with one another, close enough to be one feature counted twice. Dropping the sepals costs almost nothing, which is why the chapter can plot in two dimensions without apologising for it.
- **A rule with no fitting in it gets 96 percent.** Call it setosa below 0.8 cm of petal width, versicolor from there up to 1.7, virginica above: 144 of 150 rows correct. Both thresholds have slack, but not the same amount. The lower one can sit anywhere in the empty 0.6 to 1.0 band with no change at all. The upper one cannot: 1.7 and 1.8 both give 144, while moving it down to 1.6 costs two rows and gives 142, so the round number is not the good one. No pair of thresholds on any single column does better, petal length reaching 143 and the two sepal columns 112 and 62, counting cuts at the midpoints between adjacent observed values. Any classifier fitted here is competing against 96.0 percent, so a high score on this file is a property of the file and not a finding about the model.
- **The classes are exactly balanced, 50 and 50 and 50.** That is rare enough to be worth saying out loud, and it makes [[Accuracy]] a defensible summary here in a way it is not on [[MNIST]], where the chapter 3 five-versus-rest target puts 90.965 percent of the training rows in one class. The majority-class baseline is 33.3 percent. There is no [[Class Imbalance]] to work around, which also means nothing in chapter 4 exercises the machinery chapter 3 built for it.
- **The balance does not survive the split the chapter actually takes.** `train_test_split(X, y, random_state=42)` with no `stratify` argument turns 50/50/50 into a training set of 35 setosa, 39 versicolor and 38 virginica across 112 rows, and a test set of 15, 11 and 12 across 38. The spread is harmless at this size but it is real, and [[Stratified Sampling]] is what would remove it.
- **The three species were not sampled the same way.** The setosa and versicolor plants grew together in one colony. The virginica plants did not, and Fisher flags it himself on page 185: the virginica sample "differs from the two other samples in not being taken from the same natural colony as they were, a circumstance which might considerably disturb both the mean values and their variabilities." Whatever separates virginica from the rest therefore includes a difference of place, and a model trained here cannot tell that apart from a difference of species.
- **No missing values, and one exact duplicate.** All 600 feature values and all 150 labels are present. Rows 102 and 143, counting from 1, are the identical record `5.8, 2.7, 5.1, 1.9, virginica`, so the file holds 149 distinct rows rather than 150. A random split can put the two copies on opposite sides of the train/test line.

## Getting it

The file ships inside the package, at `sklearn/datasets/data/iris.csv`, and nothing is downloaded. `iris.filename` reports `'iris.csv'`.

scikit-learn 1.6:

```python
from sklearn.datasets import load_iris

iris = load_iris(as_frame=True)

X = iris.data           # DataFrame, 150 by 4, float64, column names carry "(cm)"
y = iris.target         # Series of int64: 0 setosa, 1 versicolor, 2 virginica
iris.target_names       # array(['setosa', 'versicolor', 'virginica'], dtype='<U10')
iris.frame              # DataFrame, 150 by 5, the features plus a 'target' column

X_arr, y_arr = load_iris(return_X_y=True)                  # two NumPy arrays
X_df, y_ser = load_iris(return_X_y=True, as_frame=True)    # the pandas pair
```

Three things about that call are worth knowing in 1.6.

- **Both arguments are keyword only.** The signature is `load_iris(*, return_X_y=False, as_frame=False)`, so `load_iris(True)` raises rather than doing anything.
- **`as_frame=True` needs pandas and changes the type of everything.** Left at its `False` default, `iris.data` is a `(150, 4)` float64 ndarray, `iris.target` a `(150,)` int64 ndarray, and `iris.frame` is `None`. `return_X_y=True` drops the `Bunch` and hands back the pair directly; the two arguments compose.
- **The rows are in species order and nothing shuffles them.** Rows 0 to 49 are setosa, 50 to 99 versicolor, 100 to 149 virginica. Slicing the file positionally the way chapter 3 slices [[MNIST]] would hand you a single-class [[Training Set]], so a randomized splitter is not optional here.

The chapter fits three models, all on `load_iris(as_frame=True)`, and it never touches the two sepal columns.

The raw file that `load_iris` reads is published at https://github.com/scikit-learn/scikit-learn/blob/1.6.1/sklearn/datasets/data/iris.csv, and its accompanying description at `sklearn/datasets/descr/iris.rst` is the text `iris.DESCR` returns. The UCI archive, holding both the erroneous `iris.data` and the corrected `bezdekIris.data`, is at https://archive.ics.uci.edu/dataset/53/iris.

> [!warning]
> `fetch_openml('iris')` is **not this file.** OpenML carries dozens of active datasets named `iris`, so the call warns that "Multiple active versions of the dataset matching the name iris exist" and returns version 1, dataset id 61. That copy is an ARFF upload of UCI's `iris.data`, and samples 35 and 38 in it carry the two wrong rows described in Provenance. Its labels also arrive as the strings `'Iris-setosa'`, `'Iris-versicolor'` and `'Iris-virginica'` rather than as the integers 0, 1 and 2, so `iris.target_names[iris.target]` and every comparison built on it fails. Nothing in the values themselves announces the difference. Use `load_iris`.
