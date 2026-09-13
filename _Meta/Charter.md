---
kind: meta
title: Charter
stability: durable
---

# ML Vault Charter

This vault is a single source of truth for machine learning, built up from books, papers, courses, projects, and experiments. Its purpose is expertise, not coverage. A note exists to hold something I understand well enough to explain, derive, apply, and know the limits of. Anything less is a stub, and stubs are debts.

## 1. Tree

Folders are topical and mirror the `up` hierarchy. Every topical folder has an index note of the same name. Implementations live beside the method they implement. A method with three or more satellite notes gets its own subfolder.

```
Home.md
_Meta/
  Charter.md
  Notation.md
  templates/
Sources/
  Books/HOML/            one note per chapter
  Papers/                one note per paper, citekey filename
  Courses/
Experiments/             dated, append-only
Projects/

Learning theory/         tasks, generalization, validation, metrics
  Performance measures/
Data preparation/        scaling, encoding, imputation, splitting
Mathematical foundations/
Supervised methods/
  Linear models/
  Support vector machines/
  Decision trees/
Unsupervised methods/
```

Folders are added when a topic has three notes, never before. Expected later: `Neural networks/`, `Training deep networks/`, `Architectures/`, `Generative models/`, `Reinforcement learning/`, `Deployment/`.

A note lives in exactly one folder. When it belongs in two, it lives under its primary parent and is linked from the other folder's index note with a line of commentary.

## 2. Frontmatter

```yaml
kind: concept          # concept | method | implementation | source | experiment | index | project | meta
up: "[[Supervised methods]]"
sources:               # every source this note draws on; grows over time
  - "[[HOML Ch04 Training Models]]"
aliases: []
stability: durable     # durable | evolving | dated
confidence: shaky      # shaky | solid   (solid only after review, see section 5)
```

`sources` is a list on purpose. When a second book or paper covers the same idea, it is added here and the note is revised, rather than a second note being written. That convergence is what makes the vault a source of truth instead of a shelf of summaries.

`kind` definitions: a **concept** is a definition, claim, or mechanism (metrics, losses, task types, and terms are concepts). A **method** is a named algorithm, model, or technique. An **implementation** is library-specific. A **source** records what one external artifact said. An **experiment** records something that was run. An **index** curates links with commentary.

## 3. The quality bar

A note is complete only when every section below has real content. Empty headings are not allowed; a section that cannot be filled yet is written as a question.

**Concept** (150 to 300 words)
- One-sentence statement of the idea, in my words, as the first line.
- Formal definition or formula, in LaTeX, for anything quantitative.
- Intuition: why it holds, in plain language.
- What it ignores or where it fails, with a concrete case.
- A small worked example when the idea is numeric.
- Connects to: links with a phrase saying how (depends on, contrasts with, generalizes).

**Method** (300 to 600 words)
- One line: what it does mechanically.
- The objective: what is being optimized, written out.
- Inductive bias: what the model assumes about the data.
- Assumptions and required preprocessing (scaling, encoding, independence).
- Hyperparameters as a table of tradeoffs (parameter, direction, effect), never as recommended values.
- Complexity of training and prediction where known.
- Breaks when: the honest failure mode.
- Lineage: ancestor, successor, competes with.
- Link to its implementation note.

**Implementation** (as long as needed, `stability: dated`)
- Import and minimal working call.
- Parameters worth knowing, with defaults and what changing them does.
- Gotchas: things that silently produce wrong results.
- Library version the note was written against.

**Source** (chapter or paper)
- Thesis in one line: why the artifact exists.
- Extracted: links to every note that came out of it. This list is the audit trail.
- Open questions.
- Exercises done, linking experiment notes.
- Page references stay in the source note, not in concept notes.

**Experiment** (append-only, `Experiments/YYYY-MM-DD slug.md`)
- Hypothesis, setup (data, split, seed), result, what surprised me, which notes this changed.

**Index**
- Two sentences framing the topic.
- Every child linked, each with one line saying what it contributes. An index with bare links is a folder listing and has no reason to exist.

## 4. Writing rules

- Own words only. If a sentence could be pasted back into the book unchanged, it has not been understood yet.
- Every quantitative claim has a formula. Every method has a failure mode. Every metric has an example where it is the wrong metric.
- Title concepts as noun phrases (`Recall`) or as claims (`Averaging uncorrelated errors reduces variance`). Title methods by canonical name. Add aliases for every other name the thing goes by; search depends on them.
- Link forward. Write `[[Kernel trick]]` the first time the idea appears, even if the note does not exist yet. Unresolved links are the reading queue.
- Record hyperparameters as tradeoffs, not values. Values rot; tradeoffs do not.
- Mark what will rot: API surface is `dated`, math is `durable`.
- Never edit an experiment note. Write a new one and link back.

## 5. Review and confidence

A note is `confidence: shaky` from creation until it passes review. Review means: an agent session using the chapter-study prompt has checked it for factual errors, verified every required section has content, and I have answered questions about it without looking. Only then does it become `solid`. A `solid` note that later turns out wrong goes back to `shaky` with a note about why.

The agent's role is interrogator and checker. It does not write note bodies. It finds errors, names omissions, asks questions, and tells me what to add. I do the writing, because the writing is the learning.

## 6. Per-chapter loop

1. Read the chapter once, no notes.
2. Run the notebook. Change something and watch it break.
3. Create the source note: thesis line, open questions, page references.
4. Write concept and method notes for everything worth keeping, to the quality bar in section 3, in my own words, with the book closed as much as possible.
5. Do at least two exercises. Log each as an experiment note.
6. Run the chapter-study prompt. Fix everything it finds. Repeat until it signs off.
7. Follow one forward link from an earlier chapter and update that note with what I now know.
8. Update the relevant index notes with a line of commentary per new child.

## 7. Hygiene

- Weekly: every `shaky` note either gets reviewed or gets a line in its body saying what is missing.
- After each book part: one synthesis note comparing every method covered on inductive bias, data appetite, preprocessing needs, interpretability, and failure modes. These synthesis notes are the most valuable notes in the vault.
- A new source on a topic that already has notes means revising those notes and appending to `sources`, not writing parallel notes.
