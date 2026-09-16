---
note_kind: dataset
medium: dataset
author: R. Kelley Pace, Ronald Barry
year: 1997
url: https://github.com/ageron/data/tree/main/housing
up: "[[Home]]"
aliases:
  - California housing
  - California housing dataset
  - California Housing Prices
  - housing dataset
  - housing.csv
  - housing.tgz
  - cal_housing
  - fetch_california_housing
---

# California Housing

A 1990 US Census extract of Californian housing, 20,640 rows wide enough to carry a full supervised regression workflow and small enough to fit in memory. It is the worked example running through [[HOML Ch02 End-to-End Machine Learning Project]], so every number quoted in this vault's chapter 2 notes was computed on this file.

## Provenance

R. Kelley Pace and Ronald Barry built the dataset from the 1990 US Census and published it with *Sparse Spatial Autoregressions*, **Statistics and Probability Letters** 33(3), 291-297, 1997. The 20,640 observations in the paper are the 20,640 rows here.

A row is a **census block group**, not a house. A block group is the smallest geographical unit the Census Bureau publishes sample data for, typically 600 to 3,000 people. Géron's notebooks call them "districts" instead, on the grounds that "block group" reads as jargon. This matters for reading the columns: `total_rooms` is every room in the block group, and `median_house_value` is a median over the block group's houses, so a single row is a neighbourhood summary and nothing in the file describes an individual dwelling.

The copy at the URL above is not the original. Géron changed it in exactly two ways, both of them teaching devices:

- 207 values were removed at random from `total_bedrooms`, so the chapter has something to impute.
- The categorical `ocean_proximity` column was added, so the chapter has something to encode.

Everything else matches the StatLib original column for column and value for value.

## Columns

Ten columns, nine numeric and one categorical. Ranges are over the whole file, before any split.

| column | meaning |
|---|---|
| `longitude` | block group longitude, decimal degrees, -124.35 to -114.31 |
| `latitude` | block group latitude, decimal degrees, 32.54 to 41.95 |
| `housing_median_age` | median age of the houses in the block group, in years, 1 to 52 |
| `total_rooms` | rooms across every house in the block group, 2 to 39,320 |
| `total_bedrooms` | bedrooms across every house in the block group, 1 to 6,445. The only column with missing values |
| `population` | people resident in the block group, 3 to 35,682 |
| `households` | occupied housing units in the block group, 1 to 6,082 |
| `median_income` | median household income, scaled and clipped. Not dollars. See below |
| `median_house_value` | median house value in the block group, US dollars, 14,999 to 500,001. The target |
| `ocean_proximity` | rough location category, five levels |

The `ocean_proximity` levels are unbalanced: `<1H OCEAN` 9,136, `INLAND` 6,551, `NEAR OCEAN` 2,658, `NEAR BAY` 2,290, and `ISLAND` 5. Five rows in a level is few enough to break a [[Stratified Sampling|stratified split]] on that column and few enough that a train/test split can leave a level entirely unseen.

## Known quirks

Four things about this file are not visible in a schema and change what a model learns.

- **`median_house_value` is capped at 500,001 dollars.** 965 of the 20,640 rows sit exactly on that value, which is why the target's histogram has a spike in its last bin. A model fitted on it learns a price ceiling that does not exist. The minimum is 14,999 and only 4 rows are there, so the low end is not the same problem.
- **`housing_median_age` is capped at 52 years.** 1,273 rows sit on the cap, the same spike one column over.
- **`median_income` is neither dollars nor raw.** It is scaled to roughly tens of thousands of US dollars and clipped at both ends, running 0.4999 to 15.0001. 12 rows sit on the floor and 49 on the ceiling. A value of 3.87, the file's mean, is about 38,700 dollars. No plot reveals the units; only the documentation does.
- **`total_bedrooms` has 207 missing values**, 20,433 non-null out of 20,640. It is the only column with any, and the gaps are Géron's, not the Census Bureau's.

## Getting it

The folder at the URL above holds `housing.csv` and a README documenting the two tweaks. The book does not read that CSV, though: it fetches a gzipped tarball from the repository root and extracts it, so that the download is cached and the notebook is rerunnable offline. The archive is `https://github.com/ageron/data/raw/main/housing.tgz`, 449 KB, containing `housing/housing.csv`.

pandas 2.x:

```python
import tarfile
import urllib.request
from pathlib import Path

import pandas as pd

archive = Path("datasets/housing.tgz")
if not archive.is_file():
    archive.parent.mkdir(parents=True, exist_ok=True)
    urllib.request.urlretrieve(
        "https://github.com/ageron/data/raw/main/housing.tgz", archive)
with tarfile.open(archive) as tarball:
    tarball.extractall(path="datasets")

housing = pd.read_csv("datasets/housing/housing.csv")
```

For a throwaway look, pandas will read the CSV over HTTPS in one line, and it is byte-identical to the one inside the archive:

```python
housing = pd.read_csv(
    "https://raw.githubusercontent.com/ageron/data/main/housing/housing.csv")
```

## Not the same as scikit-learn's version

> [!warning]
> `sklearn.datasets.fetch_california_housing` is the same underlying census data and **is not this file**. It loads the untouched Pace and Barry original from StatLib and reshapes it. Follow the chapter with the sklearn loader and your numbers will not match the book's, with nothing in either to tell you why.

What differs:

- **No `ocean_proximity`.** The column never existed upstream. The sklearn frame is eight numeric features and a target, so there is nothing to [[One-Hot Encoding|one-hot encode]] and the [[Column Transformer]] section of the chapter has no categorical branch to build.
- **No missing values.** `total_bedrooms` is complete in the original, so there is nothing to impute and the [[Missing Value Imputation]] section has no work to do.
- **The count columns are gone.** sklearn divides them through by `households` before handing them over: `AveRooms` is `total_rooms / households`, `AveBedrms` is `total_bedrooms / households`, and `AveOccup` is `population / households`. `households` itself is dropped once it has served as the divisor. Chapter 2's [[Feature Engineering]] step, deriving `rooms_per_house` and `people_per_house` by hand, has already been done and cannot be repeated.
- **The target is rescaled.** `MedHouseVal` is in units of 100,000 dollars, so the cap reads 5.00001 rather than 500,001 and an RMSE of 0.5 there means the same error as 50,000 here.
- **The names differ**: `MedInc`, `HouseAge`, `AveRooms`, `AveBedrms`, `Population`, `AveOccup`, `Latitude`, `Longitude`, with the target as `MedHouseVal`.

What is the same: 20,640 rows in the same order, `MedInc` identical to `median_income` including its 0.4999 to 15.0001 clipping, `HouseAge` carrying the same 52-year cap, and `Population`, `Latitude` and `Longitude` unchanged.
