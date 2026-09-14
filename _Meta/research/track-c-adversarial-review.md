---
note_kind: meta
title: "Track C: adversarial review"
date: 2026-09-13
---

# Track C: adversarial review

This memo reviews the **previous** Charter and memos (2026-09-13), not the current ones. It is kept as the audit trail for [[decisions]], which records how each attack was answered. Attack 13 rests on a premise the tooling memo later overturned: Bases does have `file.backlinks` and `file.hasLink()` (see [[track-a-tooling-bases]] section 5).

Read: Charter, both tracks, summary, the study prompt, the template, Notation, Home (empty), both `.obsidian` files. No web research; anything not a file quote is my reasoning.

## Summary verdict

The sourced core (concept notes own ideas, literature notes per source, divergence in prose, folders on kind, schema drift as the killer) survives. The structural machinery does not: the four tests are waived by the tie-breaker the moment they apply, the subject tree fails them at nearly every boundary on Track B's own table, the schema contradicts the only template and the Charter's own frontmatter, and the promised Bases view of classification methods is impossible with no topic property. The Charter is out of sync with the vault on day one.

## The 21 attacks

**1. Four tests.** Unsourced (summary.md invents them). They restate "partition"; nobody verifies "stable in a year" in thirty seconds. Against the Charter's own tree with Track B sections 3 and 4: Foundations vs Methodology fails disjoint (bias-variance in PML1 statistics and ESL 7; regularization in PML1 4.5); Linear Models vs Neural Networks fails (Bishop24: a single-layer network); Kernel Methods vs Unsupervised, Probabilistic, Neural Networks fails (kernel PCA, GPs, attention as kernel regression); Probabilistic vs Unsupervised fails (PCA as latent variable model, EM, mixtures); Unsupervised is a task folder inside a family tree; exhaustive fails (RL: Géron 18, PML2 VI, no folder); stable fails by the Charter's own "expect to promote." The one folder the summary validated (Supervised) is not in the tree. *Fatal.* Show two siblings that pass disjoint on Track B's evidence.

**2. Kind/subject, dropping "type".** Reasonable, but `source_type` keeps the banned word, and the summary's domain/topic distinction ("the whole structural argument") is abandoned in the Charter. *Minor.* Show one vocabulary used end to end.

**3. Tie-breaker.** Rule 1 contradicts its own example: PCA's mechanism is an eigendecomposition (Foundations, CS189), yet it is filed by output. Rule 1 gives navigation-hostile homes: attention to Kernel Methods (PML1 15.4.2), gradient boosting to Foundations (descent in function space), dropout to Methodology (a regularizer). Rule 2 presumes a generality order among siblings that does not exist (Kernel Methods versus Unsupervised?). Ambiguous under both rules: regularization, cross-validation, kernel PCA, ELBO (Foundations, Probabilistic, or Neural Networks where Géron 17 teaches it), GPs, batch norm (Practice, Foundations, or NN), softmax (Foundations vs Linear Models; the answer changes as the learner matures). Only logistic regression and ROC are stable. *Serious.* Show the thirteen cases decided by the rule alone.

**4. Seven notes.** Unsourced. Self-violated: nine `Knowledge/` folders exist with zero notes while "Practice/ is the one exception." *Minor.* Show why seven, or exempt the initial tree.

**5. Three levels.** Unsourced; unclear whether Knowledge counts. Neural Networks/Generative/Diffusion/ hits the cap by year three, and item 1's disjoint test forbids the subdivision anyway. *Minor.* Show one deep folder that subdivides under both rules.

**6. Confidence ladder.** Three of four rungs are completeness (`stub` no body, `draft` body, `working` all sections present); only `solid` is confidence. Section 8 then denounces completeness ladders, citing [OF4]. No demotion when a later source revises a `solid` note, which is the vault's purpose, so `solid` decays silently. The prompt's Phase 4 sets `solid` on AI sign-off, not cold re-derivation. One note re-checked weekly means a 300-note vault is re-verified every six years. *Serious.* Show a trigger that fires on revision.

**7. Title split.** Admitted unsourced. The object/result call is a judgment per note ("the bias-variance decomposition" is both). Reasoning: claim titles also hit filename-forbidden characters (colon, question mark). *Minor.* Show ten notes classified without hesitation.

**8. Quality bar.** Géron 4 yields roughly ten methods and five concepts: 6,000 to 10,000 words per chapter at the stated targets, nineteen chapters, one semester. "Sources and where they disagree" is empty with one source, so nothing reaches `working` until a second book is read. The normal equation has no hyperparameters but "never as prose" forces an empty table. Softmax fits 150 words; bias-variance needs a derivation 400 words cannot hold. *Serious.* Show one chapter completed to bar in the hours a course allows.

**9. Eight kinds.** Track A implication 1 itself lists "courses" as a kind. No home for: a dataset (Géron 2 housing), a library (Géron is "lifecycle plus tooling"), a book or course as a whole (`source` is one chapter; `up` on chapter notes points nowhere), a comparison note (Track A section 4 requires one; the Charter names it and gives it no kind), conceptual exercises with no hypothesis (most of Géron 4's twelve), a derivation, a cheat sheet, an open question (Inbox forever). `implementation` fails the concept-count split test: it cannot be cited without its method. `index` is a role: Home, Knowledge Index, and a Classification index are all indexes, and the last has no folder. *Serious.* Show a Géron chapter's every artefact filed without a "misc."

**10. "Top level splits by kind."** False. Knowledge holds four kinds; Inbox any; root mixes meta and index. Honest description: the top level splits by workflow role (capture, knowledge, reading record, runs, builds, infrastructure), and subject beats kind in the only folder that matters. The "three clicks" argument for co-locating concept and method is not applied to source and experiment notes on the same idea. *Serious.* Show why kind-purity is claimed for a tree that abandons it at level two.

**11. Family axis.** Track B section 2: PML2, the newest graduate text, dropped family for purpose, which "absorbs new subfields best"; ESL has "no stated axis." The Charter's authorities for family are PRML (2006) and PML1, and its own flagged risks (generative, transformers, RL, fairness) are exactly the new subfields. For the beginner, Géron 2 shatters into Practice (cleaning, scaling) and Methodology (splits, grid search), the two leftover folders, and "the ML workflow" has no home. Reasoning: task or lifecycle serves year one, family serves year three, and the choice must be argued against the graduate source that rejected family. *Serious.* Show where Géron 2's notes land and why PML2's axis loses.

**12. Maintenance.** Twenty minutes for four tasks, one of which (derive a method cold) is twenty minutes alone. [OF4] and [OF1] are evidence that rituals lapse. The cross-cutting gap is patched by the least reliable mechanism, a manual weekly backlink patrol. *Serious.* Show a month of logs.

**13. No topics property.** Track A section 8: cross-cutting concepts "are reachable only if they are encoded as properties." Summary collision 2: they "surface through frontmatter and Bases, the only secondary axis." The Charter reverses both without argument, then promises "every classification method" as a Bases view, which no property supports. Backlinks are also invisible in the explorer, the client's first interface. *Fatal.* Show the `.base` query that lists classification methods.

**14. Empty Practice/.** All nine folders are empty; the exception is the rule. Géron 2 fills Practice in week two, so "delete in a year" is dead. *Minor.* Show the count of earned versus planned folders.

**15. Sources by media.** Encodes `source_type` twice (folder and property). A course with a textbook splits across Courses and Books; a paper with a talk gets two notes or a wrong folder. The prompt's path `Sources/Books/HOML/` adds a per-book level the Charter never defines. *Minor.* Show the per-book rule.

**16. Inbox.** For a person who files everything, the inbox is always empty (unused) or the dump for item 9's homeless kinds. Its only sourced use [OF4] is a link-first user's. *Minor.* Show an inbox under twenty items after a year.

**17. Schema vs vault.** Template has `book`, `chapter`, `Github` (capitalised: drift on day one), `up`; lacks `aliases`, `confidence`, `sources`, `source_type`, `author`, `year`, `citekey`. types.json fixes `chapter` as number, unmentioned. Every existing note, the Charter included, carries `title` and `date`, neither in the schema, under "kind-specific additions, and nothing else." `sources` on a source note and `confidence` on a meta note are meaningless. *Serious.* Show the template and the Charter's own frontmatter passing the Charter.

**18. Usable as reference.** 2,358 words; "which folder, kind, sections, properties" spans sections 2 to 6 plus Open placements. No one-screen card. *Minor.* Show the answer found in under a minute.

**19. Nine steps.** Reading Géron without running code (step 2) is not how the book works. Twelve exercises to experiment notes, a four-phase AI session that blocks per note, index commentary: reasoning, ten to twenty hours per chapter. The loop reduces by inspection to steps 4 and 9 (extract into existing or new notes, update `sources`, set confidence). *Serious.* Show one chapter done in one week.

**20. Citations.** See below. *Serious in aggregate.*

**21. Other.** (a) Concept and method are two kinds for one idea: "gradient descent" the concept and the method, with overlapping sections (Definition vs What it does), is duplication by another name. (b) Track A implication 2 wants one overview note per folder; the tree has one index for nine. (c) Notation.md, which every formula rule leans on, mixes $\textbf{x}^i$ and $\mathbf{x}^{(i)}$ and covers five symbols. (d) Home.md is empty. (e) No naming rule for implementation notes, so explorer sort separates them from their method. (f) `up` may point at "index or concept," the drift section 4 forbids. *Serious (a), minor (rest).*

## Year three failure scenarios

1. Neural Networks holds eighty flat files. Training tricks fit Methodology too, so disjoint fails and the tests forbid subdividing; the learner subdivides anyway in the explorer and the tests become a dead letter.
2. Exam week, "every classification method": no folder, no property, no Bases view, no Classification index because indexes are per folder. The learner creates `Classification/` in the explorer, duplicating logistic regression's home.
3. "Gradient descent" went `solid` after Géron 4. Goodfellow 8 doubles it with Adam and saddle points; property still says `solid`; the weekly re-derive habit lapsed in October.
4. Three books in: "Logistic regression" concept, method, sklearn and PyTorch implementations. Four files, disagreement recorded in one.
5. A Papers template grows `venue` and `doi`, a Courses template `course` and `week`, beside `Github` and `book`. Twenty-five properties: [OF1] reproduced.

## Citation mismatches

- Section 2 cites Track A section 3 for "no source objects to folders keyed on kind." Section 3 does not say it; "Where practitioners disagree" does, from silence. Obsidian Rocks' "folders are binary" applies to every folder, kind included.
- "Closed, stable, exhaustive, disjoint by construction": from summary.md, an invention, not a track.
- "Deep texts use family (Murphy, Bishop, ESL)": Track B gives PML2 purpose, ESL "no stated axis."
- "Track B shows task-first trees fragment families": Track B's table labels Géron "lifecycle plus tooling," then calls it task-first.
- Open placements: "no pre-2022 source has a home for" generative models. Track B says this of transformers; generative models have Goodfellow 20 and Géron 17.
- Section 6: crucial disagreements "earn a comparison note." Track A section 4: a note per source plus a comparison note. The dropped half conflicts with "never duplicated."
- Section 4: "Bases has no reverse-link access" is not in Track A section 8; it comes from the brief.
- Section 8 cites [OF4] against completeness ladders while defining one.
- "Four or more divisions at once" is the summary's gloss; Track B section 3 gives no count.
- Seven notes, twenty notes, three levels, ten seconds, twenty minutes: no source.
- The prompt now cites sections 5 and 8 correctly; the summary's housekeeping note is stale.

## What is actually well supported

Concept note owns the idea, one literature note per source, `sources` as a list, divergence in prose, no citekey stacking [A4, ZF1, OF2]. Folders on kind rather than topic [Z2, K1, OF3]. Schema drift, not count, as the failure; vault-wide property types [OF1, O2]. Bases reads frontmatter only; no inline fields [O8]. Split on concept count; atomicity as output [ZF3, Z4]. Prose index notes [OF5]. Aliases as the search lever [O1, O5]. Confidence over completeness [OF4], though not the four-rung form. Track B's stable core as folder candidates, and the honest flagging of deep learning, generative, graphical, and fairness placements.

## Confidence in my own attacks

| Attacks | Grade | What would change it |
|---|---|---|
| 1, 13 | high | A working Bases task view, or two siblings that pass disjoint |
| 3 | high | A defined generality order among siblings |
| 6, 8, 12, 19 | medium | Time logs from one real chapter; my hour estimates are reasoning |
| 9, 10, 21a | medium | A filed Géron chapter with no homeless artefacts |
| 11 | medium | An argument for family that engages PML2's purpose axis |
| 17, 20 | high | File comparisons; only a rewrite changes them |
| 2, 4, 5, 7, 14, 15, 16, 18 | low to medium | Minor; any stated rationale would suffice |
