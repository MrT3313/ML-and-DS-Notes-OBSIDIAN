---
note_kind: meta
title: "Quality Bar"
aliases:
  - Quality Bar
---

# Quality Bar

What a note in this vault has to contain before it counts as done. Derived from the notes themselves, not imposed from outside: if a rule here disagrees with what the majority of existing notes do, the notes are probably right and this file needs updating.

## 1. Frontmatter contract

Every note under `Knowledge/` carries:

| key          | required                   | values                                                                                                          |
|--------------|----------------------------|-----------------------------------------------------------------------------------------------------------------|
| `note_kind`  | yes                        | `concept`, `method`, `index`, `source`, `dataset`, `meta`                                                       |
| `aliases`    | yes for concept and method | every name you would plausibly type in a `[[link]]`, including plurals and abbreviations                        |
| `up`         | yes                        | the single parent note, as `"[[Name]]"`. Leaves point at their task or domain hub, never at `[[Home]]` directly |
| `sources`    | yes for concept and method | list of source notes, as `"[[HOML Ch01 ...]]"`                                                                  |
| `confidence` | yes except `index`         | `draft`, `solid`, `verified`                                                                                    |

Two rules that are easy to break:

- The key is `confidence`, not `con`. A typo here makes the note invisible to every Bases query.
- `up` is a link, not a folder. Folder placement and `up` answer different questions: the folder says what kind of thing this is, `up` says what it is part of. `Logistic Regression` lives in `Models/Linear/` and has `up: "[[Classification]]"`. Both are correct.

Notes outside `Knowledge/` carry their own contract: `source` and `dataset` under `Sources/`, `meta` under `__Meta/`. Their keys are given with their sections in section 2.

`__Meta/Callouts.md` is exempt from this document entirely, being a syntax reference copied from the Obsidian documentation rather than an authored note.

`note_kind` is not queried today. No Bases block filters on it; nine display it as a column. It is authoring discipline, a label you write correctly so the note is shaped correctly, and nothing more. Making it queryable means writing a Bases view that filters on it, and until that exists a wrong `note_kind` breaks no query.

`confidence` is aspirational in the same way. Every note carrying the key is `draft`; `solid` and `verified` have never been used. The vocabulary stays as it is. `solid` is not self-assigned, it is earned by surviving the review protocol in section 4.

## 2. Required sections by kind

Headings are `##`. Subsections inside a required section are `###`.

Under `Knowledge/`, reserve `#` for nothing: the note title is the filename, and an H1 only repeats it. This holds without exception across the knowledge base. Outside it, a note opens with an H1 carrying its name when it is a document read front to back: this file, `README`, the prompts, a book container note, a dataset note. A chapter note is the exception and takes no H1, because it is reached through a link, from its container note and from the `sources:` of every note it produced, which makes it an atom like the notes under `Knowledge/`. An `index` takes no H1 wherever it lives, as its required shape already implies: a scope statement, before any heading.

Notes under `Knowledge/` are cumulative. They are the vault's standing account of a topic, revised and sharpened as more sources are read, so they are organized by the topic and never by the source that happened to supply a piece of it. **A heading that names a source is a defect**: `### From Chapter 3`, `### What the chapter says`, `### Géron's version`. Each one freezes the topic into per-source layers, and the layers multiply with every book added, until the note is a stack of reading notes rather than one account of the thing. New material is merged into the section it belongs in by topic, and a subsection is named for what it contains: `### Out-of-fold predictions`, not `### From Chapter 3`. `Cross-Validation`, `Testing Set`, `Scikit-Learn Estimator API` and `Error Analysis` all carried such headings and were normalised on sight.

Attribution is `sources:` in the frontmatter, plus a `[[link]]` in the prose wherever naming the source actually adds something: a specific worked example, a quotation, a claim the source is unusual in making, or how much weight it gives a topic. `[[HOML Ch03 Classification|HOML chapter 3]]` is concrete and navigable. "The chapter" is neither, and it is an anonymous referent besides, since a reader arriving by link from anywhere else has no idea which chapter is meant. An ordinary fact needs no citation in the body at all; it is simply stated, and `sources:` already records where the reading came from.

Three things are not this defect. An `index` naming the chapter that will bring an unwritten method, which its required shape asks for. A forward promise about scheduled material, ex "gradient descent and regularization from chapter 4". And library history, ex "`root_mean_squared_error` arrived in scikit-learn 1.4", which is a fact about the tool rather than a source being used as an authority.

When a new source contradicts what a note already says, the answer is research, not a second layer. Settle which version is correct against a primary source, rewrite the note to say that, and record the correction. Two headings in one note each asserting a different thing is the one outcome that is never acceptable.

A new `note_kind` is earned when the thing shares no required sections with any existing kind. `dataset` shares none with `source`, so it is a kind. A metric shares all three of `concept`'s and needs only one more, so it is a loosening of `concept`, not a kind of its own.

### `concept`

```
## Definition
## Formal statement
## Where it is used
```

- **Definition** is one or two sentences, in your own words. If it reproduces the book's phrasing, it does not count. When the source uses the term without defining it and you supplied the definition, say so here.
- **Formal statement** carries the mathematics. If the idea genuinely is not quantitative, discharge the section with exactly this string and nothing else: **Not further quantitative at this depth.** Then say why. The string is fixed so the discharged notes can be found in one search; "Not quantitative.", "Not quantitative as a law." and the rest are drift and get normalised on sight.
- **Where it is used** is where the note earns its place in the graph. At least two outbound `[[links]]`, each with the phrase that explains the relationship. A bare list of links does not count.

Three optional sections, all in use:

- `## VS` for a note whose whole job is contrast against its siblings.
- `## Hyperparameters` for a concept that carries configuration. Metrics are the case: `precision_score` takes `average`, `pos_label` and `zero_division`, `fbeta_score` takes `beta`, and those change the number that comes back. Same table as `method` uses, same test for what counts.
- `### In scikit-learn`, a subsection naming the function or class that implements the concept and the arguments that matter. It belongs under whichever section it serves, usually `## Formal statement`.

### `method`

```
## What it does and when
## Algorithm            (protocols)   or   ## Algorithm or formula   (models and transforms)
## Hyperparameters
## Failure modes
## Implementation
```

- **What it does and when** states the use case and the condition under which you would reach for this over its alternatives.
- **Algorithm** is the heading for a protocol you carry out, written as numbered steps: cross-validation, holdout, imputation, EDA. **Algorithm or formula** is the heading for a model or transform with something to write down, a fitted equation or a closed form: linear and logistic regression, the scalers, the encoders. Pick by what the body actually contains, not by where the note is filed.
- **Hyperparameters** is conditional. Required, as a table with columns name, symbol, default, effect of increasing, how to tune, when the method has tunable parameters. Discharged in one line when it has none: "None for the plain fit." for a model, naming the variants that add them, and the same line for a protocol with nothing genuinely tunable. EDA and `Pipeline` are the standing cases; they have none.

  The test: a parameter counts if changing it changes the fitted or transformed output. Speed, memory, verbosity, container type and plot appearance do not count, however many of them the API exposes. `n_jobs`, `verbose`, `copy`, `sparse_output`, `figsize` and `bins` are not hyperparameters and do not belong in the table.
- **Failure modes** is at least three bullets, each naming a concrete circumstance, not a generic caution.
- **Implementation** is runnable code with the library version pinned in the prose above it. Not every subject has code to write. Where the domain has no pinned library, the section does the same job by other means: it names the systems that actually implement the method and says what differs between their implementations, which is what makes an abstract mechanism concrete enough to check a claim against. Where not even that can be named, discharge the section with exactly this string and nothing else: **No implementation to name at this depth.** Then say why. The string is fixed for the same reason the Formal statement discharge is, so the discharged notes can be found in one search.

### `index`

```
(one-paragraph scope statement, before any heading)
## Methods
(Bases block)
## What is missing
```

- The scope statement says what qualifies for this index and names the paradigm it sits under.
- **Methods** lists written notes with a one-line characterization each. Unwritten ones go as plain text naming the chapter that will bring them.
- The list heading is named for what the list holds. `## Methods` is the default and covers most indexes. A domain or vault hub ([[Home]], [[Machine Learning]]) uses `## Areas`, listing folders rather than methods. An index whose members are neither uses their name: `Open Data Repositories` lists repositories under `## Repositories`, and that is correct rather than a deviation, because "methods" would be a lie about what is in the list. Only this heading varies. The scope statement, the Bases block and `## What is missing` are the same in every index.
- The Bases block is part of the shape, not decoration, and every index carries one, hubs included. `Machine Learning` is currently the one index without one; that is the exception to correct, not the precedent to copy.
- **What is missing** is the honest gap list. An index with no `## What is missing` is claiming completeness.
- `index` is earned by having methods to list, not by what the note is about. A task with none is a `concept` carrying its own definition and any task-level mathematics, and converts to `index` when chapters bring it methods: the definition becomes the scope statement, and task-level measures move to their own note. `Association Rule Learning` is the standing case, a task in `Tasks/` whose support and confidence belong to every algorithm rather than to one.

### `source`

`source` comes in two shapes, and both are legitimate.

**Chapter notes**, one per chapter read:

```
## Thesis
## Extracted
## Open questions
```

- **Thesis** is the chapter argued in one paragraph, in your own words.
- **Extracted** is every note this chapter produced, grouped by theme.
- **Open questions** are the things the chapter left unresolved for you, each phrased so a later chapter can settle it.

**Container notes**, one per book:

```
## Chapters
```

A container carries the book's frontmatter (`medium: book`, `author`, `year`, `url`) and a list of its chapters, written ones as links and unwritten ones as plain text. It has no thesis, nothing extracted and no open questions, because those live in the chapter notes it points at. This is a shape, not a chapter note left unfinished.

### `dataset`

For a data file the vault works on, under `Sources/Datasets/`. Frontmatter is `note_kind: dataset`, `medium: dataset`, `author`, `year`, `url`, `up`, `aliases`. No `sources`, because the note is itself the source. No `confidence`, because the claims are checkable against the file.

```
(one-paragraph statement of what the file is, before any heading)
## Provenance
## Columns
## Known quirks
## Getting it
```

- **Provenance** is who built it, from what, and what anyone has changed since. A teaching copy that differs from the original says exactly how it differs.
- **Columns** is a table, one row per column, each with its meaning, its units and its range over the whole file before any split. Units that are not what the name suggests are stated here.
- **Known quirks** is what a schema does not show and a model would learn anyway: caps, clipping, missing values, unbalanced levels. Each with the row count behind it.
- **Getting it** is runnable loading code and the exact URL, including which of several copies the book actually reads.

`aliases` matter more here than elsewhere: filenames, loader functions and the book's own name for the thing are all names you would plausibly type in a link.

## 3. What disqualifies a note

- A quantitative claim with no formula anywhere in the note.
- A `## Where it is used` section with fewer than two outbound links.
- Book phrasing copied rather than restated.
- Placement commentary in the body. Where a note lives is expressed by the folder tree and by `up`, never by a sentence in the prose explaining why it was filed somewhere. Such sentences go stale the first time anything moves.
- A heading that names a source, and "the chapter" used as a bare subject in the prose. A note under `Knowledge/` is organized by its topic, with attribution in `sources:` and in `[[links]]`. See section 2.
- `confidence: solid` on a note you have not been questioned on.

## 4. Review protocol

The role of a review agent working from `__Meta/prompts/Chapter-Study-Prompt.md`:

- **Interrogator and checker, never author.** It does not write note bodies, even when asked. Writing is the learning; the agent's job is to verify what was written is correct and complete.
- **Audit against this file**, by kind: missing sections, factual errors with the wrong line quoted and the correction stated, quantitative claims lacking formulas.
- **Question before answering.** It asks, waits, evaluates, and only then explains. At least one question per note must be unanswerable by someone who merely memorized the note.
- **Formulas in its output are for verification**, so you can check yours against them, never for pasting.
- **Uncertainty is stated, not guessed.** If unsure whether something is wrong, it says so and names what to check in the book.
- **No praise padding.** "Correct" is a complete evaluation.

## 5. New kinds

Two kinds have been proposed and neither was added. **`process`**, for a workflow you run end to end rather than a method you call, and **`architecture`**, for a network topology. When they were proposed, three notes in the vault were process-shaped and none was architecture-shaped, and three does not earn a kind.

They stay out, and no chapter number brings them back. A trigger set from a survey of chapters not yet read is a decision made on the weakest evidence available, and it does not survive a second book, where "chapter 10" names two different things. The question reopens on evidence from notes actually written, and the evidence is one specific thing: a note that cannot be written to any existing kind without the required sections of that kind being actively wrong for it. Not "would read better under a new kind", and not "does not fit neatly". A run that meets a genuine case writes the note to the kind that fits least badly, records the strain in its report and in the `## What is missing` of the index that owns the note, and moves on. A kind is argued for when those records have accumulated to the point where the strain is systematic rather than incidental, and the standing precedent for "not yet" is three.

Integrating a chapter is not the moment to redesign the vocabulary. New material merges into the notes that already exist, under the topic they belong to, and a contradiction found on the way is settled in the run that finds it, against a primary source, by rewriting the note to say one thing. Reaching for a new kind is not how a contradiction gets resolved; research is.

One condition attaches to any kind that is eventually added: **every existing note is re-audited against it in the same sitting.** Not later, not incrementally. A kind introduced without a backfill splits the vault into notes that were considered for it and notes that never were, and no query on that kind can tell the two apart, so every later query returns a quiet undercount rather than an error (Golder and Huberman 2006, on tag vocabularies that drift after the fact). If there is no appetite for the backfill, there is no appetite for the kind. The backfill is also the honest measure of what a kind costs, and it grows with the vault: every rule in this file has to still work at several hundred notes across more than one subject, which is where this is heading.
