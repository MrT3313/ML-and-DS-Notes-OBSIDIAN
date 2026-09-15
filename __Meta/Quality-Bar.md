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
| `note_kind`  | yes                        | `concept`, `method`, `index`, `source`, `meta`                                                                  |
| `aliases`    | yes for concept and method | every name you would plausibly type in a `[[link]]`, including plurals and abbreviations                        |
| `up`         | yes                        | the single parent note, as `"[[Name]]"`. Leaves point at their task or domain hub, never at `[[Home]]` directly |
| `sources`    | yes for concept and method | list of source notes, as `"[[HOML Ch01 ...]]"`                                                                  |
| `confidence` | yes except `index`         | `draft`, `solid`, `verified`                                                                                    |

Two rules that are easy to break:

- The key is `confidence`, not `con`. A typo here makes the note invisible to every Bases query.
- `up` is a link, not a folder. Folder placement and `up` answer different questions: the folder says what kind of thing this is, `up` says what it is part of. `Logistic Regression` lives in `Models/Linear/` and has `up: "[[Classification]]"`. Both are correct.

## 2. Required sections by kind

Headings are `##`. Reserve `#` for nothing; the note title is the filename. Subsections inside a required section are `###`.

### `concept`

```
## Definition
## Formal statement
## Where it is used
```

- **Definition** is one or two sentences, in your own words. If it reproduces the book's phrasing, it does not count. When the source uses the term without defining it and you supplied the definition, say so here.
- **Formal statement** carries the mathematics. If the idea genuinely is not quantitative, say so explicitly and say why, in the form used across this vault: "Not further quantitative at this depth."
- **Where it is used** is where the note earns its place in the graph. At least two outbound `[[links]]`, each with the phrase that explains the relationship. A bare list of links does not count.

Optional extra sections seen in practice: `## VS` for a note whose whole job is contrast against its siblings.

### `method`

```
## What it does and when
## Algorithm            (models)   or   ## Algorithm or formula   (protocols)
## Hyperparameters
## Failure modes
## Implementation
```

- **What it does and when** states the use case and the condition under which you would reach for this over its alternatives.
- **Hyperparameters** is a table with columns: name, symbol, default, effect of increasing, how to tune. If the method has none, write "None for the plain fit" and say which variants add them.
- **Failure modes** is at least three bullets, each naming a concrete circumstance, not a generic caution.
- **Implementation** is runnable code with the library version pinned in the prose above it.

### `index`

```
(one-paragraph scope statement, before any heading)
## Methods
(Bases block)
## What is missing
```

- The scope statement says what qualifies for this index and names the paradigm it sits under.
- **Methods** lists written notes with a one-line characterization each. Unwritten ones go as plain text naming the chapter that will bring them.
- **What is missing** is the honest gap list. An index with no `## What is missing` is claiming completeness.
- A domain or vault hub ([[Home]], [[Machine Learning]]) is also `index`, but uses `## Areas` in place of `## Methods`, listing folders rather than methods.
- `index` is earned by having methods to list, not by what the note is about. A task with none is a `concept` carrying its own definition and any task-level mathematics, and converts to `index` when chapters bring it methods: the definition becomes the scope statement, and task-level measures move to their own note. `Association Rule Learning` is the standing case, a task in `Tasks/` whose support and confidence belong to every algorithm rather than to one.

### `source`

```
## Thesis
## Extracted
## Open questions
```

- **Thesis** is the chapter argued in one paragraph, in your own words.
- **Extracted** is every note this chapter produced, grouped by theme.
- **Open questions** are the things the chapter left unresolved for you, each phrased so a later chapter can settle it.

## 3. What disqualifies a note

- A quantitative claim with no formula anywhere in the note.
- A `## Where it is used` section with fewer than two outbound links.
- Book phrasing copied rather than restated.
- Placement commentary in the body. Where a note lives is expressed by the folder tree and by `up`, never by a sentence in the prose explaining why it was filed somewhere. Such sentences go stale the first time anything moves.
- `confidence: solid` on a note you have not been questioned on.

## 4. Review protocol

The role of a review agent working from `__Meta/prompts/Chapter-Study-Prompt.md`:

- **Interrogator and checker, never author.** It does not write note bodies, even when asked. Writing is the learning; the agent's job is to verify what was written is correct and complete.
- **Audit against this file**, by kind: missing sections, factual errors with the wrong line quoted and the correction stated, quantitative claims lacking formulas.
- **Question before answering.** It asks, waits, evaluates, and only then explains. At least one question per note must be unanswerable by someone who merely memorized the note.
- **Formulas in its output are for verification**, so you can check yours against them, never for pasting.
- **Uncertainty is stated, not guessed.** If unsure whether something is wrong, it says so and names what to check in the book.
- **No praise padding.** "Correct" is a complete evaluation.
