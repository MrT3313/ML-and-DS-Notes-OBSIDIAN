---
note_kind: meta
title: "Research summary"
date: 2026-09-14
---

# Research summary

Second pass, from scratch. Memos: [[track-a-obsidian-practice]] (A), [[track-a-tooling-bases]] (A-tools), [[track-a-stem-atomicity]] (A-stem), [[track-b-domain-hierarchies]] (B), [[track-c-adversarial-review]] (C). Decisions and their status: [[decisions]]. Track C is kept as its own memo because it is the audit trail for every rule the Charter changed.

## Findings that shape the vault

**Track A: practice**

1. The anti-folder argument (Tietze, Matuschak, Obsidian Rocks) targets top-down topic categories, but per-se objectors exist and were missed before: Sascha Fast keeps 7,400 notes in one folder and calls folders "mostly ornamental"; Bob Doto says zettelkasten and LYT "eschew" folders. Nobody argues against folders keyed on kind or workflow, and every multi-year folder user (Milo, Konik, Ango, amanhimself, Tim Miller) converges on exactly those (A §1, §4, §7). **A folder tree keyed on workflow role at the top is tolerated by all and practiced by many; a topic tree is endorsed by no one.**
2. Obsidian's documentation takes no position (A §1).
3. The concept-count split rule is real but belongs to a forum user writing about structure notes, not to Sascha Fast on content notes. Atomicity as an output over time is Sascha's, verbatim (A §1).
4. Matuschak's "no accumulation" argument for concept-oriented notes is verbatim (A §1). The provenance model is Sascha's: own formulation, source links after each claim (stacked where sources agree), divergence in prose, "Don't hide behind any authors!" The previous "do not stack citekeys" rule was wrong (A §5).
5. The claim that Matuschak abandoned evergreen notes cannot be verified from any public primary text and is not asserted (A §1).
6. Milo's free material covers the essentials: the squeeze point, the MOC as a thinking place, and a folder framework (ACE) with the rule "structure must be earned" (A §3, §4).
7. No source addresses the cost of claim-phrase titles in an alphabetical file explorer (A §2).

**Track A: tooling**

8. **Bases has backlinks.** `file.backlinks` since 1.9.7 (August 2025), heavy and non-refreshing, and the documented idiom `file.hasLink(this.file)` gives a refreshing reverse lookup. Summaries and one-level grouping exist. No inline fields, no body content (A-tools §5, §6). The previous memo's "no reverse-link access" was outdated.
9. Dataview's last release was April 2025, not June 2024; 637 open issues, no maintenance statement. Datacore is installable, JavaScript-only, and self-described as work in progress (A-tools §7).
10. Property types are enforced vault-wide, but changing one is a click; the cost is old values that fail to parse. The six-year post-mortem names both count and inconsistency, and the quoted damage is near-synonym keys (A-tools §1, §2).
11. Maturity ladders: two forum posters say status tags go unused, including their own replacements; Appleton asserts tending with no data. The evidence is thin both ways (A-tools §4).
12. Core Templates: three variables, no logic. The All properties sidebar is the native drift audit (A-tools §9, §10).

**Track A: STEM atomicity**

13. No published source scopes personal notes for theorems and proofs. What exists is enforced policy on ProofWiki (one theorem per page, lemmas on subpages), PlanetMath (short proof inside the theorem entry, long proof separate, definitions carry no proof), mathlib (split conjunctions), Halmos ("never repeat a proof"), and Lamport and Leron on hierarchical proofs (A-stem). The Charter reasons from these analogues and labels every rule as reasoning.

**Track B: domain hierarchies**

14. Every graduate reference text files by model family with foundations front-loaded (PML1, PRML, ESL loosely, Bishop 2024); PML2 alone uses purpose; Géron uses lifecycle; every course uses teaching order, which scatters each concept to where it was first needed (B §1, §3, §10).
15. Regularization appears in four to five places per book; the CS229 notes and ESL both give generalization its own unit (B §4).
16. Fairness has no chapter in any textbook but a lecture in seven of thirteen course offerings, never with a parent unit (B §7). Causality sits with graphical models in PML2 4.7. Deployment exists only in Géron and Goodfellow 11.
17. Growth: transformers, generative models, graph learning, RL, and LLM lifecycle are the new divisions; SVMs, naive Bayes, and graphical models are vanishing from fast-path course lectures while every reference text keeps them (B §9).
18. CMU 10-701 was fetched (Spring 2026, plus 2024, 2021, 2015). Bishop 2024's appendices exist (cited by the authors' solutions manual) but their titles were not read (B verification).

**Track C: adversarial review**

19. The four tests failed on the previous tree's own evidence; the tie-breaker contradicted its PCA example; three of four confidence rungs measured completeness; the quality bar implied 6,000 to 10,000 words per Géron chapter; the schema did not match the template or the Charter's own frontmatter; the "every classifier" Bases view had no property to run on (C attacks 1, 3, 6, 8, 13, 17).

## Where the tracks conflict

| Conflict | Resolution |
|---|---|
| A says topic folders are endorsed by nobody; B says the field has a stable family hierarchy that every reference uses | Family folders inside `Knowledge/` are kept because the user navigates by folder; the departure from A is named in the Charter and its cost is stated. Maps in a visible folder (Milo's Atlas) are the sanctioned bridge. |
| A-tools shows backlinks are queryable; C attacked "no topics property" on the premise that they are not | C's attack is answered by the finding, not by adding a property. Kept without `topics`. |
| A says maps come at a squeeze point; a folder navigator needs something at the top of each area on day one | Maps are created at squeeze points; areas need no index note because the explorer already lists them. Only cross-folder lookups (tasks, reading orders) get maps. |
| A says atomicity is an output; A-stem's analogues (ProofWiki) enforce it at entry | Entry rule for theorem notes, output rule for everything else; both labelled. |
| B §3: PML2's purpose axis absorbs new subfields best; B §1: family is what references use | Family chosen (user decision); new subfields get `Reinforcement Learning/`, `Workflow/`, and subfolders inside `Neural Networks/`. |
| C says fairness needs a home that is not a leaf; B says no source supplies one | `Workflow/`, flagged as least-wrong. |

## Where best practice conflicts with folder-first navigation

Resolved in favour of navigation each time; the cost is stated.

1. **Topic folders.** Practice: none. Charter: nine subject areas inside `Knowledge/`. Cost: cross-cutting concepts (regularization, PCA, kernels, EM) have one home decided by an arbiter and are reachable elsewhere only by links, maps, and the embedded base.
2. **Claim titles.** Practice: Matuschak's complete phrases. Charter: noun phrases. Cost: the claim moves to the first body line and is not visible in the explorer.
3. **Maps at a squeeze point.** Practice: wait. Charter: a `Maps/` folder exists from the first task lookup. Cost: some maps will be thin for a year.
4. **Media-type source folders.** Practice: one flat sources folder or none. Charter: `Books/`, `Courses/`, `Papers/`, `Videos/`, `Web/`. Cost: `medium` is encoded twice, and a course with a textbook is two notes that link.
5. **Structure must be earned.** Practice: create nothing in advance. Charter: a closed list of twelve areas, each created on first use. Cost: none in the tree; the cost is that the list is decided before the material exists, which is exactly what Tietze warns against, accepted because the reference ToCs are the material.

## What changed versus the previous research, and why

| Change | Why |
|---|---|
| Bases can read backlinks; the Dataview recommendation rests on new grounds | Docs re-read from the raw help repository, 2026-03-26 commit (A-tools §5) |
| Dataview stall dated April 2025, not June 2024 | GitHub releases (A-tools §7) |
| "Do not stack citekeys" reversed | Sascha's own example stacks them (A §1) |
| Concept-count split re-attributed | Forum user, structure notes (A §1) |
| Per-se folder objectors added (Fast, Doto) | Missed before (A §1) |
| Matuschak abandonment claim dropped | Unverifiable (A §1) |
| Milo's public material rated sufficient; ACE folder framework added | Free pages read (A §3, §4) |
| Multi-year retrospectives added | New section (A §7) |
| STEM atomicity: analogues found, rules labelled reasoning | New memo (A-stem) |
| Fairness: lecture in 7 of 13 course offerings | Course schedules read (B §7) |
| CMU 10-701 fetched; Bishop 2024 appendices confirmed to exist | Gaps closed (B verification) |
| Course coverage grew from three to thirteen offerings, with 2018 versus 2026 comparison | Growth evidence (B §2, §9) |
| Géron Part I mapped to PML1 and ESL chapter by chapter | For the user's current reading (B §10) |
| Property-type change cost downgraded | Docs say one click (A-tools §1) |
| Maturity-ladder evidence downgraded to thin | Two posters (A-tools §4) |
| Adversarial review added and kept | Brief section 5, Track C |
