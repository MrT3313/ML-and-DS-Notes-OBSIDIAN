---
note_kind: index
aliases:
  - datasets
  - dataset repositories
  - where to find data
  - open data
  - data sources
up: "[[Machine Learning]]"
---

Places to obtain a dataset when you do not have one of your own. What qualifies is a source of data: a repository you download from, a portal cataloguing other repositories, or a maintained list. Individual datasets do not qualify, and neither do the notes about what to do with data once you have it. Links checked September 2026; entries the book recommended that have since changed are marked rather than dropped, because the drift is itself worth knowing.

## Repositories

### Direct repositories

- **OpenML**, `https://www.openml.org/`. The one that connects to the rest of this vault: datasets are addressable by ID and fetchable with `sklearn.datasets.fetch_openml`, so an experiment reproduces from a notebook with no manual download step.
- **Kaggle Datasets**, `https://www.kaggle.com/datasets`. Competition and community data, and the only entry here that ships public notebooks beside the data, so you have someone else's score to measure yours against.
- **UC Irvine Machine Learning Repository**, `https://archive.ics.uci.edu/datasets/`. Small, clean, decades-old benchmark sets. Good for teaching and for sanity-checking an implementation, nearly useless as evidence a method works at scale, since almost nothing here has the size or the mess of real data.
- **AWS Open Data Registry**, `https://registry.opendata.aws/`. The opposite end: genomics, satellite imagery, climate reanalysis, sets too large to download and meant to be processed in place on S3.
- **TensorFlow Datasets**, `https://www.tensorflow.org/datasets`. Curated sets wrapped as ready-to-iterate pipelines with fixed canonical splits, which matters because everyone reporting on them split the data the same way.
- **Papers with Code**, `https://paperswithcode.com/`. **Shut down July 2025**; the address now redirects to Hugging Face's trending papers. It indexed benchmark to leaderboard to implementation. The archive lives on GitHub under `paperswithcode/paperswithcode-data`, and Hugging Face Datasets is the closest live replacement. The near-identical `paperswithcode.co` is an unaffiliated site trading on the name and was never the project's address.

### Meta portals

These catalogue repositories rather than host data, which is what you want when the question is whether anyone publishes a thing at all.

- **DataPortals**, `https://dataportals.org/`. A curated list of open data portals run by governments and institutions worldwide. Live.
- **OpenDataMonitor**, `https://project.opendatamonitor.eu/`. A **finished EU project** under FP7 grant 611988. The site and its dashboard at `opendatamonitor.eu` both resolve, but nothing has been published since 2015, so read its coverage statistics as a 2015 snapshot.

### Lists other people maintain

- **Wikipedia's list of ML datasets**, `https://en.wikipedia.org/wiki/List_of_datasets_for_machine-learning_research`. Organized by task and modality with a citation each, so it is the fastest route to the *paper* a dataset came from.
- **r/datasets**, `https://www.reddit.com/r/datasets/`. Requests and one-off scrapes. Useful for the long tail nothing else covers, with no curation and no guarantees.
