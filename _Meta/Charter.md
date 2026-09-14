---
note_kind: meta
title: "Charter"
date: 2026-09-14
---

# Charter

Working reference. Keep it open while taking notes. Research: [[track-a-obsidian-practice]] (A), [[track-a-tooling-bases]] (A-tools), [[track-a-stem-atomicity]] (A-stem), [[track-b-domain-hierarchies]] (B), [[track-c-adversarial-review]] (C). Every decision and its status is in [[decisions]]. Rules marked **(reasoning)** are this Charter's own judgment, not a research finding.

## 1. Purpose

This vault is my single source of truth for data science and machine learning, built from fundamentals toward graduate depth. When a new source covers an idea already here, the existing note is revised and the disagreement is recorded inside it; nothing is duplicated. I navigate by folder first, so the folder tree is the interface, and that constraint outranks every convention below.

## 2. Folder tree

```
Home.md                     entry point: links to Knowledge/Maps and this Charter
Notation.md                 symbol conventions
_Meta/                      Charter, templates, prompts, research, decisions
Inbox/                      capture only; emptied weekly
Knowledge/                  what I know; revised forever
  Foundations/              probability, statistics, linear algebra, optimization, information theory
  Generalization/           bias-variance, regularization, cross-validation, model selection, metrics
  Linear Models/            linear and logistic regression, GLMs
  Kernel Methods/           SVMs, kernels, Gaussian processes, nearest neighbours
  Trees and Ensembles/
  Neural Networks/          deep learning; subfolders when earned (see below)
  Probabilistic Models/     graphical models, latent variables, Bayesian inference, causality
  Unsupervised/             clustering, dimensionality reduction
  Reinforcement Learning/
  Workflow/                 data preparation, pipelines, deployment, monitoring, fairness, interpretability
  Reference/                datasets, libraries, tools, benchmarks
  Maps/                     topic maps: tasks (Classification, Regression, ...), reading orders, course maps
Sources/                    what I read; one note per book, course, paper, video, page, plus one per chapter
  Books/<Book>/  Courses/<Course>/  Papers/  Videos/  Web/
Experiments/                what I ran; dated
Projects/                   what I built
```

**Why this shape.** The top level splits by *workflow role* (capture, knowledge, reading record, runs, builds, infrastructure), which is what every multi-year folder user converges on and no source objects to (A §4, §7). Inside `Knowledge/`, folders follow *reference order*: the model-family axis with foundations front-loaded, which every graduate reference text uses (B §1, §3), plus two areas the family axis cannot hold: `Generalization/` (the CS229 notes' Part III and ESL chapter 7) and `Workflow/` (Géron's lifecycle chapters, which no theory text files anywhere, B §7, §10). This is a hybrid, said plainly: kind at the top, subject underneath (C attack 10).

**Named cost.** Topic folders are outside every practitioner's recommendation (A §4), and task lookup ("every classifier") is not a click path. Both are absorbed by `Knowledge/Maps/`, the one sanctioned way to put topic structure in the tree (Milo's Atlas folder, A §3). Teaching order (Géron's chapter sequence) is a map note, not the tree, because notes taken in teaching order scatter one concept across the places it was first met (B §10).

**Which folders exist on day one.** The twelve `Knowledge/` folders above are a closed list. Each directory is created when its first note is written, not before (Milo: "structure must be earned", A §4). The list itself does not change without amending this Charter.

**When a subfolder is created (reasoning).** Only inside a `Knowledge/` area, only one level deep, and only when all three hold: the parent holds more notes than one explorer screen shows (about 25); the subfolder would take at least five of them; its name is a part or chapter title in an arbiter text (PML1, PML2, Bishop and Bishop 2024). Rationale: the reference texts already have exactly three levels (part, chapter, section), and a section is note-sized, so the vault needs area, subfolder, note and nothing deeper. First expected case: `Neural Networks/Generative Models/`, the fastest-growing division (B §9).

**Where a note goes, decided in this order.**

1. **A general principle goes to `Generalization/`; a family-specific technique stays with its family** and links to the principle. Test: does the technique exist outside this family? Ridge penalty, early stopping, cross-validation: yes, so `Generalization/` (ridge regression itself is a `Linear Models/` note that links to the L2 penalty). Dropout, batch normalization, tree pruning: no, so they stay with their family. **(reasoning**, forced by regularization appearing in four to five places per book, B §4).
2. **Otherwise, where Murphy PML1 files it.** PML1 is the designated arbiter because it is the most recent full-coverage reference with an explicit family axis (B §1). Mapping: chapters 2 to 8 to `Foundations/` (except 4.5, 4.7, 5.1.3 to 5.1.6, 5.2, 5.4, which are `Generalization/`); 9 to 12 `Linear Models/`; 13 to 15 `Neural Networks/`; 16 to 17 `Kernel Methods/`; 18 `Trees and Ensembles/`; 20 to 22 `Unsupervised/`; 19 and 23 `Neural Networks/`.
3. **Not in PML1: PML2, then Bishop and Bishop 2024, then Géron.** PML2 Part II and chapters 4, 28, 29, 36 to `Probabilistic Models/`; Part IV to `Neural Networks/` until a generative subfolder is earned; 34 to 35 to `Reinforcement Learning/`; 33 to `Workflow/`. Géron 2 (launch, monitor, maintain) and 19 to `Workflow/`.
4. **Not in any arbiter:** the most general folder that fits, and the note says so in its first line.

Every other membership is a link in the body, an alias, and a line in the relevant map. No note is ever copied.

## 3. Note kinds

Closed list. Every note has exactly one `note_kind`. Adding a kind is a Charter amendment.

| kind | one-line definition | lives in |
|---|---|---|
| `concept` | an idea, object, definition, theorem, or derivation. What something **is**. Comparisons between methods are concept notes. | `Knowledge/` |
| `method` | a procedure with inputs, outputs, hyperparameters, and code. What I **do**. | `Knowledge/` |
| `reference` | a dataset, library, tool, or benchmark: described, not derived. | `Knowledge/Reference/` |
| `source` | my record of one book, chapter, paper, course, module, video, or page. | `Sources/` |
| `experiment` | one run: hypothesis, setup, result. | `Experiments/` |
| `project` | multi-session work with a goal and an outcome. | `Projects/` |
| `index` | a map: navigation plus commentary for a topic, task, reading order, or folder. | `Knowledge/Maps/`, or the folder it indexes |
| `meta` | vault infrastructure. | `_Meta/`, or root |

`implementation` is no longer a kind: code is a required section of the `method` note (C attack 9; it could never be cited without its method, A §6).

## 4. Frontmatter schema

Everything in frontmatter. Inline `key:: value` fields are forbidden: Bases does not read them (A-tools §5). Dataview is not installed and stays out: Bases now reads backlinks, and Dataview has had no release since April 2025 (A-tools §5, §7).

| property | type | on which kinds | allowed values |
|---|---|---|---|
| `note_kind` | Text | all | the eight kinds above |
| `aliases` | List | all | acronyms, expansions, plurals, notation, other textbooks' names; may be empty |
| `up` | Text (a link) | all | the parent: a map, a book note for a chapter, a method for an experiment |
| `sources` | List of links | concept, method, reference | links to `source` notes. **Several entries is the normal case.** |
| `confidence` | Text | concept, method | `draft`, `solid` (section 8) |
| `verified` | Date | concept, method | date of the last cold re-derivation; set only on promotion |
| `medium` | Text | source | `book`, `paper`, `course`, `video`, `web` |
| `author`, `year`, `url` | Text, Number, Text | source, reference | |
| `chapter` | Number | source (chapter notes only) | already typed as Number in this vault |
| `date` | Date | experiment, meta | |
| `method` | Text (a link) | experiment | the method note tested |
| `status`, `started` | Text, Date | project | `active`, `paused`, `done` |

**Rules that keep this from rotting.** A property is added only when a Bases view or a search needs it (A-tools §2). Never introduce a near-synonym: the six-year post-mortem's damage was one value scattered across `type`, `keywords`, and `travel-acc`, not the number of fields (A-tools §2). Property types are enforced vault-wide, so a type is chosen once (A-tools §1). `meta` notes may carry `title` and `date`; no other kind carries `title`, because the filename is the title. Weekly, the **All properties** sidebar is the drift audit (A-tools §10).

**No `topics` property.** Cross-cutting membership is carried by body links and read back two ways: the backlinks pane, and an embedded base that lists every `Knowledge/` note linking to the current one (A-tools §5). Every map note and every cross-cutting concept note (regularization, gradient descent, cross-validation, PCA, kernels, maximum likelihood, EM) carries this block:

```base
filters:
  and:
    - file.hasLink(this.file)
    - file.inFolder("Knowledge")
views:
  - type: table
    name: Linked here
    order:
      - file.name
      - note_kind
      - confidence
```

It costs no schema and cannot drift; its one failure is a note that forgets to link the map, which section 6 makes a writing rule.

## 5. Quality bar

A note is complete when it has every section listed. Lengths are targets, not limits. **(reasoning)** Calibrated so that one Géron chapter fits in one week: the previous bar implied 6,000 to 10,000 words per chapter (C attack 8).

| kind | required sections | length |
|---|---|---|
| `concept` | Definition (one or two sentences, my words) / Formal statement (formula in [[Notation]] conventions, or "not quantitative") / Where it is used (links to methods and maps) / Sources (who says what, and where they differ) | 80 to 300 words |
| `method` | What it does and when / Algorithm or formula / **Hyperparameters** (table, or "none") / **Failure modes** (at least one) / **Implementation** (minimal code or a repo link, with library and version) / Sources | 150 to 500 words plus code |
| `reference` | What it is / How I use it / Gotchas / Link | 50 to 200 words |
| `source` (chapter, paper, video, page) | properties header / Thesis in two sentences, book closed / `## Extracted` / Open questions | 100 to 300 words |
| `source` (whole book or course) | properties header / chapter or module list with links / Verdict | grows |
| `experiment` | Hypothesis / Setup (data, method link, hyperparameters, seed) / Result / Interpretation | 100 to 300 words |
| `project` | Goal / dated Log / Outcome / Notes spawned | grows |
| `index` | Purpose in one line / annotated list or the embedded base / What is missing | 50 to 300 words |
| `meta` | no bar | any |

The `## Extracted` heading name is fixed: `_Meta/prompts/Chapter-Study-Prompt.md` parses it.

**Scoping a note.** One idea, completely (A §2). Split when a note holds two ideas that would each be cited alone; the trigger is a citation from outside, not length (A §6). Atomicity is approached over time, not required at creation (A §2).

**Theorems, proofs, derivations (reasoning; no published rule exists, A-stem).** One note per definition, theorem, or lemma you would cite on its own. The proof stays inside the theorem note unless it is long or a second proof exists. A definition note carries no proof. A derivation (bias-variance decomposition, normal equations, ELBO, backprop) is one note of top-level steps that links out to steps that are theorems in their own right; never split at each algebra line. Extract a lemma only when a second result needs it.

## 6. Writing rules

- **Own words.** Close the book, then write. If a sentence could have been pasted, rewrite it.
- **Formulas.** Every quantitative claim carries its formula in LaTeX, using [[Notation]]. A claim about magnitude, rate, or complexity without a formula is incomplete.
- **Failure modes.** Every `method` names at least one condition under which it fails or misleads.
- **Aliases.** Every acronym, plural, notation, and other textbook's name goes on the canonical note. This matters more here than in a link-first vault because search is my second move (A-tools §3).
- **Forward links.** Link a concept the first time it appears, inside the sentence that says why it matters. **Link every map the note belongs to** (its task, its reading order); that link is what makes the map's embedded base find the note.
- **Hyperparameters** as a table: name, symbol, default, effect of increasing it, how to tune it.
- **Disagreeing sources.** Write my own formulation and stand behind it. Cite each claim by linking the source note; several links after one claim is fine where sources agree. Where they differ, say who says what in prose. Minor differences collapse into one note; a crucial one gets a comparison `concept` note (A §5).
- **Titles (reasoning).** Noun phrases for every `Knowledge/` note; the claim goes in the first line of the body. Claim-phrase titles are long and sort by first word in the explorer, and no source addresses that cost (A §2).

## 7. Source workflow

One loop for a chapter, a paper, a module, or a video. The chapter version:

1. **Before reading:** create the book note once (`Sources/Books/<Book>/`), then the chapter note from the template, `up` pointing at the book note.
2. **Read, run the code, mark the text.** Write nothing in the vault yet.
3. **Extract.** For each idea worth keeping: search first, aliases included. If a note exists, revise it, add this chapter to `sources`, and record any divergence in prose. If not, create it in the home folder from section 2 and link the maps it belongs to. This step is the vault's purpose.
4. **Close the chapter note:** thesis in two sentences with the book closed; every note touched listed under `## Extracted`; open questions.
5. **Review:** run `_Meta/prompts/Chapter-Study-Prompt.md`. Exercises that ran code become `experiment` notes linking their `method`; conceptual exercises are answered inside the chapter note. Promote what passed (section 8).

A paper's thesis is its claim plus the evidence offered. A course module is one source note; the course itself is the parent note. **Projects** link to the methods they used; concepts discovered while building get ordinary notes, and the project note lists what it spawned.

## 8. Review: draft to trusted

Two values. `draft` is everything not yet trusted. `solid` means I re-derived or explained it cold, without opening the note, after the study prompt's interrogation; promotion sets `verified` to today. A completeness ladder is not tracked because the evidence, thin as it is, says such ladders go unmaintained (A-tools §4), and the previous four rungs were three completeness stages in disguise (C attack 6).

**Decay.** A `solid` note revised by a later source stays `solid` in its frontmatter but is caught by this base, which lives in `Home.md`:

```base
filters:
  and:
    - confidence == "solid"
    - file.mtime > verified
views:
  - type: table
    name: Solid but edited since verified
    order:
      - file.name
      - verified
      - file.mtime
```

Each entry is either re-derived (update `verified`) or demoted to `draft`. Trivial edits will also land here; that is the accepted cost of an automatic trigger. **(reasoning)**: the date comparison follows the Bases docs but has not been run in this vault; test it on the first solid note.

**The AI's role is** to audit a note against this Charter, list missing sections, quote wrong lines and correct them, flag phrasing copied from a source, ask questions only understanding can answer, name concepts with no note yet, and propose links and map memberships.

**The AI's role is not** to write note bodies, supply formulas for pasting, decide what deserves a note, or set `confidence` or `verified`. Only I promote a note. The AI may recommend a promotion and may refuse to agree with one.

## 9. Maintenance

**Weekly, ten minutes (reasoning; the previous list was four tasks in twenty minutes, C attack 12).**
- Clear the "Solid but edited since verified" base: re-derive or demote.
- File `Inbox/` to zero.
- Open **All properties**, sorted by frequency; any property name not in section 4 is renamed or deleted now.

**Per milestone (end of a book, course, or semester).**
- Rewrite the commentary in every map touched; add maps that a squeeze point revealed (A §3).
- Create any subfolder that now passes the section 2 rule; delete none, because none is created empty.
- Re-read this Charter, amend it, and add a row to [[decisions]].

## Flagged uncertainties

Placements decided by rule, not evidence, and most likely to move:

- **Nearest neighbours in `Kernel Methods/`.** PML1 files kNN under exemplar methods (16) beside kernels (17); the shared mechanism is a similarity function. Reasonable, not sourced.
- **Causality in `Probabilistic Models/`.** PML2 puts structural causal models inside the graphical-models chapter (4.7) but the causality chapter (36) under Action. Chosen for the mechanism; flagged.
- **Fairness and interpretability in `Workflow/`.** No textbook has a fairness chapter; seven course offerings teach it as a late standalone lecture with a different neighbour each time (B §7). `Workflow/` is the least wrong home, not a sourced one.
- **Learning with fewer labels (PML1 19) and graph learning (PML1 23) in `Neural Networks/`.** Both are mostly deep-learning material today; either could become its own area.
- **`Neural Networks/` as one area among nine.** The largest structural disagreement in the literature (B §6). Subfolders are allowed precisely so this area can hold Bishop and Bishop's twenty chapters without moving anything; the first promotion will be generative models.
- **Graphical models in `Probabilistic Models/`.** Sources disagree four ways (B §6); absent from every 2026 fast-path course, present in every reference text.
- **Both Bases blocks** are written from the current docs and untested here. `file.hasLink(this.file)` in an embedded block, and a date comparison against `verified`, are the two things to try first.
- **Template.** `_Meta/templates/Chapter Notes.md` was updated to this schema; a book-note and a concept-note template do not yet exist.
