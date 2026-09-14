# Vault design: full re-research and revision

You are the **coordinator**. You do not perform web research yourself. You delegate research to subagents, verify what they return, resolve conflicts, and write the final documents. Your value is judgment and synthesis, not fetching.

The vault root is the current working directory.

---

## 1. Mission

Redo the research behind this vault's design from scratch, recheck every assumption in the existing documents, and rewrite them.

**Deliverables, all inside `_Meta/`:**

| File | Action |
|---|---|
| `_Meta/research/track-a-obsidian-practice.md` | rewrite |
| `_Meta/research/track-b-domain-hierarchies.md` | rewrite |
| `_Meta/research/summary.md` | rewrite |
| `_Meta/Charter.md` | rewrite |
| `_Meta/research/decisions.md` | create: a decision log (see section 9) |

Add further research memos if a track earns one. Do not create notes, folders, or templates anywhere outside `_Meta/`. The Charter *describes* the vault structure; the user builds it.

Treat the current contents of these files as a **draft by a previous agent that you are auditing**, not as settled fact. A substantial fraction of it is that agent's invention rather than a research finding. Section 6 tells you exactly which parts.

---

## 2. Who this is for

- A learner working toward graduate-level competence in data science and machine learning, currently taking an applied ML course and reading Géron's *Hands-On Machine Learning*. Not yet at graduate level; the vault must serve a beginner now and hold graduate depth later **without reorganization**.
- The vault is meant to become a **definitive source of truth**. When two sources disagree or one goes deeper, the existing note is **revised, not duplicated**.
- Primary sources: textbooks and papers. Must also accommodate courses, websites, videos, personal projects, and experiments, added over years.
- **Navigation order: file explorer first, then search, then links.** The folder tree is the primary interface. **Structure not visible in the tree is structure that will not be used.** This constraint outranks every convention. Where best practice and folder-first navigation conflict, say so explicitly and choose in favor of navigation the user will actually use.

---

## 3. Current vault state

Verify this yourself before starting; do not trust this summary.

```
Home.md          (empty)
Glossary.md      (empty; may be redundant with concept notes plus aliases)
Notation.md      (note_kind: meta; math symbol conventions, in active use)
_Meta/
  Charter.md
  prompts/Chapter-Study-Prompt.md    an interrogation protocol; cites Charter sections by number
  templates/Chapter Notes.md          stub: note_kind, title, book, chapter, Github, up
  research/                           the three memos
```

**Obsidian configuration, confirmed from `.obsidian/core-plugins.json`:** core plugins only. Properties, Templates, Backlinks, Outgoing Links, Graph, Tag pane, Bases, Canvas, Daily Notes, Sync are enabled. **Dataview is NOT installed.** Any recommendation depending on a community plugin must be flagged as such, with a core-plugin alternative given.

**Existing de facto conventions to respect or explicitly overturn:** `note_kind` as the frontmatter discriminator (already in `Notation.md` and the template), `confidence: solid` as a value (already in the study prompt), and the `## Extracted` heading (the study prompt parses it).

---

## 4. How to operate

### Delegate research to subagents
- Run tracks **in parallel**, one subagent per track.
- Give each subagent a **self-contained brief**: it has none of your context. State the audience, the folder-first constraint, the plugin situation, the output path, the word cap, the citation requirement, and the style rules in section 10.
- **Assign every subagent a unique output path.** Never let two subagents write to the same file. This went wrong before and produced two competing memos at one path.
- Require every subagent to **fetch and read primary sources**, not rely on search snippets, and to **state explicitly when a claim could not be verified** rather than asserting it.
- Require each memo to **grade its own confidence per section** (high, medium, low) and to name what would change its mind.

### Verify what comes back
Do not accept subagent output at face value. Spot-check load-bearing claims yourself, especially anything in section 6A. If a subagent reports a finding that contradicts the existing documents, that is a signal to investigate, not to average the two.

### Ask the user, do not assume
**Stop and ask** whenever a genuine conflict or open question would change:
- the folder tree,
- the frontmatter schema,
- any closed list (note kinds, confidence values, allowed property values),
- or a rule the user will follow daily.

Ask in batches, with a concrete recommendation and the tradeoff for each, not an open-ended survey. Do **not** ask about matters you can settle from the research or by sensible default; decide those and log them in `decisions.md`. Do **not** silently pick a side on a real fork.

---

## 5. Research tracks

Tracks A and B are mandatory. Track C is new and mandatory. Add more if warranted.

### Track A: Obsidian and note-taking practice
How experienced practitioners scope individual notes (atomicity, titling, one note versus two); how they connect notes into topic maps, Maps of Content, or content webs; how they balance folders against links **for someone who navigates by folder**; how they handle one idea covered by multiple sources; how they handle one idea that decomposes into subtopics; how they use properties, aliases, and templates to keep a vault consistent over years.

Prefer primary sources: Obsidian's own documentation, the Obsidian forum, zettelkasten.de and its forum, Andy Matuschak's evergreen notes, Nick Milo / Linking Your Thinking on MOCs, Maggie Appleton, Eleanor Konik, Bob Doto. **Name where authors disagree rather than blending them into a false consensus.**

Also required, because the previous pass failed at it: see B1 in section 6.

### Track B: domain hierarchies
How graduate-level data science and machine learning is actually organized by the people who teach it. Use the **actual tables of contents** of four to five standard textbooks (Murphy PML1 and PML2, Bishop PRML and Bishop & Bishop 2024, Hastie/Tibshirani/Friedman ESL, Goodfellow/Bengio/Courville, and at least one applied text, preferably Géron HOML 3e) and two to three graduate course syllabi (CS229, CS189, CMU 10-701 or 10-715, CS231n, CS224n).

Extract: the recurring top-level divisions; the competing organizing axes (task, model family, mathematical foundation, inference paradigm, lifecycle) and what each costs; the **cross-cutting concepts** that appear under several divisions at once, with the specific chapters where each appears; concrete placement disagreements; how foundations are handled; where the field is growing fastest.

**Do not produce a full topic skeleton.** The output is a reference to check a proposed structure against. End with test questions a proposed tree must pass.

### Track C: adversarial review of the existing design (new)
A subagent that reads the current `_Meta/Charter.md` and the three memos and **argues against them**. Its brief: find every claim that is unsourced, every rule that is arbitrary, every internal inconsistency, and every place the design will fail in year three. It should specifically attack the items listed in section 6C. Output to `_Meta/research/track-c-adversarial-review.md`. You may keep this memo in the final deliverables or fold it into `summary.md`; say which you did and why.

---

## 6. Claims to re-verify, grouped by how much to trust them

### 6A. Sourced claims that are load-bearing. Re-verify each; the design collapses if any is wrong.

1. The canonical anti-folder argument (Tietze, Matuschak, Obsidian Rocks) targets **topic** folders chosen top-down, **not folders as such**, and no source objects to folders keyed on note kind. *This single claim justifies the entire top-level structure. Verify it hard, and look for sources that do object to folders per se.*
2. Obsidian's own documentation takes no position on folders versus links.
3. Split a note on **concept count, not length**.
4. Atomicity is an **output** notes approach over time (Sascha Fast), not an entry gate.
5. Matuschak's concept-orientation argument names the multi-source accumulation failure ("there's no accumulation").
6. Provenance: do not stack citekeys; state divergence in prose and stand behind your own formulation.
7. The documented long-vault metadata failure is **schema drift** (near-synonym property names), not property count. Source: a six-year, 15,000-note forum post-mortem.
8. Obsidian enforces property **type** vault-wide, making early type choices expensive to reverse.
9. Maturity ladders (seedling / budding / evergreen) go unused in practice; a **confidence** axis that triggers an action is maintained where a completeness ladder is not.
10. **Bases** (verify against current docs, this is evolving): supports table, list, cards, kanban, map; filters, sorting, grouping, formulas; embeddable in notes; reads frontmatter properties, `file.tags`, `file.links`, and file metadata; has **no reverse-link/backlink access**; does not read Dataview inline `key:: value` fields; no note-content or task access.
11. Dataview's author last committed June 2024; its successor Datacore is still incomplete. **Re-check both, and re-examine the recommendation not to install Dataview.**
12. Core Templates supports only `{{title}}`, `{{date}}`, `{{time}}`, with no logic, prompts, auto-apply, or folder targeting.

### 6B. Known gaps and unverified items. Resolve, or mark explicitly as unresolved.

1. **Atomicity for mathematical and STEM material. This is the single biggest gap and it matters most here.** Two independent research passes both failed to find a credible published source on how to scope notes containing theorems, proofs, and derivations. The one promising source is now a dead page. Search harder: mathematicians' and physicists' Zettelkasten writing, academic PKM literature, lecture-note design, textbook pedagogy research. If it genuinely does not exist, say so plainly and reason from first principles, labelled as reasoning rather than evidence.
2. The widely repeated claim that Matuschak abandoned evergreen notes for flat per-project outlines. Previously found unverified. Settle it.
3. Nick Milo's MOC substance sits partly behind a paid course, so prior confidence was only medium. Find what is publicly verifiable.
4. Bishop & Bishop (2024) appendices were reconstructed from Crossref, not read. Verify.
5. CMU 10-701's syllabus was never successfully fetched. Get it or an equivalent.
6. Fairness had no canonical placement in any textbook read. Confirm or overturn.

### 6C. NOT research findings. These are the previous agent's inventions. Attack them hardest.

Each of these is currently presented in the Charter as though settled. None is sourced. Keep, modify, or discard each on the merits, and **say which you did**.

1. **The four tests** for whether a folder level is legitimate: children must be *closed* (listable today), *stable* (list holds in a year), *exhaustive* (every note fits one), *disjoint* (no note fits two). Invented. Is it right? Is it usable?
2. The **kind / subject** two-axis vocabulary, and the deliberate dropping of the word "type" to avoid collision with Obsidian's property types.
3. The two-place tie-breaker: home is where the note's **mechanism** belongs, not where it is applied; if undecidable in ten seconds, it goes in the **more general** folder.
4. The threshold of **seven notes** before a new folder may be created. Arbitrary.
5. The **three-level depth cap** inside `Knowledge/`. Arbitrary.
6. The confidence ladder **`stub` / `draft` / `working` / `solid`** and its promotion conditions, in particular "`solid` means I re-derived it cold."
7. The **title convention split**: noun phrase for objects and definitions, claim phrase for results. Both prior passes flagged this as synthesis, not a sourced position.
8. The **quality bar**: the required section list and word-count target for each note kind. Entirely invented.
9. The **eight note kinds** as a closed list: concept, method, implementation, source, experiment, project, index, meta.
10. **A known internal inconsistency.** The Charter claims the top level "splits by note kind," but `Knowledge/` contains four kinds at once (concept, method, implementation, index). The split is actually a hybrid. Either fix the design or fix the justification, and do not paper over it.
11. The choice of **method family** as the subject axis inside `Knowledge/`, with **task-based lookup** named as the accepted casualty. Re-open this. Consider seriously whether a learner in an *applied* course reading Géron is better served by a task axis or a lifecycle axis, and whether the axis that suits a beginner is the axis that suits them in year three.
12. The weekly and per-milestone **maintenance habits**. Invented.
13. The decision to have **no `topics` property**, relying on links plus the backlinks pane. Re-examine, given that Bases cannot query backlinks, which means cross-cutting concepts are invisible to every automated view.

### 6D. Structural questions deliberately left open. Close them or re-flag them.

1. `Neural Networks/` as a sibling folder versus deep learning as a parallel subtree. Previously called the choice most likely to fail.
2. Generative models (VAEs, GANs, flows, diffusion): currently inside `Neural Networks/`, flagged as the fastest-growing division.
3. Graphical models: currently in `Probabilistic Models/`; sources disagree four ways.
4. Regularization: currently in `Methodology/`; the worst cross-cutting offender.
5. Fairness, interpretability, causality, MLOps: currently in a `Practice/` folder created empty, which violates the "structure must be earned" principle.
6. `Foundations/` as a peer folder versus nested.
7. Whether `Sources/` should subdivide by media type (Books, Papers, Courses, Videos, Web) or whether that is itself drift.
8. Whether `Glossary.md` should exist at all, given concept notes carry aliases.

---

## 7. Output: the research memos

Each memo:
- YAML frontmatter: `note_kind: meta`, `title`, `date`.
- **Under 1500 words** unless you justify otherwise in one line at the top.
- Dense: tables and tight bullets, not prose paragraphs.
- Inline citations with URLs; a Sources list of URLs actually read at the end.
- Per-section confidence grading.
- An explicit "could not verify" list.

`summary.md` must contain: the findings from each track that should shape the vault; every place the tracks conflict with each other; every place best practice conflicts with folder-first navigation, with the conflict resolved in favor of navigation and the cost stated; and a list of what changed versus the previous version of the research and why.

---

## 8. Output: the Charter

A **working reference the user keeps open while taking notes.** Not an essay. Target two to three pages. Required sections, in this order:

1. **Purpose**, in three sentences.
2. **Folder tree**: the initial tree, plus the rule for when a new folder is created, plus the rule for where a note goes when it belongs in two places. Must be navigable by someone clicking through folders.
3. **Note kinds**: a closed list with a one-line definition of each and the folder each lives in.
4. **Frontmatter schema**: the properties every note carries, with allowed values. Must support a note drawing on **multiple sources**.
5. **Quality bar per note kind**: the sections a note of each kind must contain before it counts as complete, plus rough length targets.
6. **Writing rules**: own words, formulas for quantitative claims, failure modes for methods, aliases, forward links, how hyperparameters are recorded.
7. **Source workflow**: the loop for working through a book chapter, a paper, a course module, or a video, from first read to finished notes, including how experiments and projects link back.
8. **Review**: how a note moves from draft to trusted, and what the AI's role in that review is and is not.
9. **Maintenance**: weekly and per-milestone habits.

Rules for the Charter:
- **Justify structural choices with one-line references to the research memos**, so the user can see which finding drove which rule.
- Do **not** build a full topic skeleton. The Charter defines how the tree grows, not the whole tree.
- Where a rule is your reasoning rather than a research finding, **label it as such**. Do not launder invention as evidence. This is the previous version's main failure.
- Where you are uncertain about a domain placement, **say so** rather than picking silently. Keep a flagged-uncertainties section.
- If you change a Charter section number, update the references in `_Meta/prompts/Chapter-Study-Prompt.md`, which cites the quality-bar and review sections by number, and preserve the `## Extracted` heading name that the same prompt parses.

---

## 9. Output: the decision log

`_Meta/research/decisions.md`. One row or short block per decision, covering every item in sections 6C and 6D and anything else you settled:

- the decision,
- kept / modified / discarded versus the previous version,
- the evidence or reasoning behind it,
- confidence,
- what would change it.

This is how the user audits your judgment without re-reading everything.

---

## 10. Constraints

- Write **only** inside `_Meta/`. Create nothing elsewhere.
- Do not build a full topic skeleton.
- Where best practice and folder-first navigation conflict, say so in the Charter and choose in favor of navigation the user will actually use.
- If uncertain about a domain placement, say so rather than picking silently.
- Verify before asserting. Mark unverifiable claims as unverified. Never present reasoning as a citation.
- **Writing style, follow exactly: never use em-dashes or en-dashes. Use commas, parentheses, colons, or separate sentences. Never use "e.g."; write "ex" or rephrase.** Pass this rule to every subagent.

## 11. Finish by reporting

- What changed versus the previous design, and which finding drove each change.
- What you could not verify.
- Every question you need answered, if any remain.
- The single weakest point in the new design.
