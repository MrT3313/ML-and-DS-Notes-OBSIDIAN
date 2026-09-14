---
note_kind: meta
title: "Track A: note atomicity for theorems, proofs, and derivations"
date: 2026-09-14
---

The previous two passes found nothing on this question. This pass searched wider and reports what exists, what does not, and reasoning clearly separated from evidence. About 1,700 words because the search log is part of the finding.

## Bottom line

No credible published source gives scoping rules for *personal notes* on theorems and proofs. What exists: (a) enforced editorial policies of public math wikis and proof libraries; (b) a few one-off practitioner posts; (c) mathematical-writing essays on when to extract a lemma; (d) pedagogy research treating proofs as modules. Only (a), (c), and (d) are credible, and none is about personal notes.

## Search log

| Target | Result |
|---|---|
| "zettelkasten mathematics / physics theorem proof atomic", "one theorem per note", "evergreen notes mathematics" | Same handful of hits each time: mildobsessions (dead), forum.zettelkasten.de threads, pqnelson's blog, the MathWiki repo |
| forum.zettelkasten.de (7 threads), forum.obsidian.md (5 threads), Hacker News (2 threads) | All read; three ZK threads and one Obsidian thread touch scoping, thinly |
| math.stackexchange, academia.stackexchange, reddit | No matching threads; reddit blocked to the search engine |
| mildobsessions: live page, Wayback CDX, archive.ph | "Under construction." Zero snapshots of the article. A 2024-12-14 homepage snapshot holds only the teaser (posted 2023-10-22, author had "6 months or so" of use) |
| Fast, Tietze, Matuschak, Nielsen, Tao, Doto, Aldrich, Gwern, Gowers | No scoping rule anywhere |
| ProofWiki, PlanetMath, Stacks, nLab, Wikipedia MoS, mathlib, Metamath | Policies read (ProofWiki via curl; the coordinator re-verified the one-theorem-per-page text) |
| Halmos, Knuth/Larrabee/Roberts, Lamport, amsthm, CASRAI, Gasarch, Richeson, Steenrod | Read except Steenrod (text not accessible) |
| Mejia-Ramos 2012, Leron 1983, Hodds/Alcock/Inglis 2014, structured derivations | Abstracts read |

## Vein 1: practitioners

Confidence: medium. Would change my mind: a recovered copy of the Holmes article or reddit access.

| Source | Scoping content | Credibility |
|---|---|---|
| Alfred Holmes, mildobsessions (2023) | Search snippet only: "one or two definitions, theorems or propositions" per note, proof "in the same note", split only if "you'd want to mention one without the other" | Unverifiable, six months of use |
| pqnelson (2022), about 7,900 slips | One slip per register (definition, theorem, proof, remark, example), titled "register. summary"; a "proof sketch" below the theorem statement; a justified step "is a branch off the slip" | Sustained practitioner, one post |
| zhaoshenzhai/MathWiki (Obsidian) | Results "broken down into their atomic components of definitions, propositions, and theorems"; theorem notes hold "both statement and proof" | Working vault, no rationale |
| forum ZK 1877 | Halmos's own practice: 1,000 sheets "each with a mathematical statement on it, a theorem, a lemma, or even a minor comment, complete with proof" | Halmos primary verified |
| forum ZK 3415 | Sascha: "Math and the other hard sciences are particularly resistant to the Zettelkasten Method." No rule offered | Declines to give rules |
| forum Obsidian 90472 | "one big note per chapter/book/lecture, which I then break down"; a formula that "will crop up again and again" gets its own note | One-off |

## Vein 2: public math wikis and proof libraries

Confidence: high (policies read directly).

| Source | Rule |
|---|---|
| ProofWiki [P1] | "The rule is to present only one theorem per page. Theorems with more than one statement have to be refactored." Each alternative proof "is placed on a subpage and transcluded as a separate section." "Lemmas that do not otherwise merit their own page, are put on a subpage of the proof page." "Corollaries are treated like lemmas." |
| PlanetMath [P2] | "Proofs may be included in the entry for the theorem itself if they are short." "If the proof is long, it should go in its own entry, attached to the theorem itself." A definition "should not have proof." Several concepts may share an entry via a "defines" field |
| Stacks Project [P3] | A tag can reference "definitions, lemmas, propositions, theorems, remarks, examples, exercises, situations and even equations, sections and items"; the tag is the unit, no size rule |
| nLab [P4] | "We want only one page on a given concept." A new page must be "linked to from at least one relevant other page" |
| Wikipedia MoS/Mathematics [P5] | Include proofs "when they expose or illuminate the concept or idea; do not include them when they serve only to establish the correctness"; a separate proof article only if the proof is "significant as a proof" and the parent "is quite long" |
| mathlib style guide [P6] | "the return type of a lemma should not be a conjunction, and one should instead prove two lemmas." "Hypotheses should not be conjunctions" |
| Metamath conventions [P7] | "A proof with the sole purpose of supporting a final proof is a lemma"; "prove reusable results separately"; break proofs beyond a few hundred steps |

## Vein 3: pedagogy literature

Confidence: medium. Would change my mind: a Scholar pass on note-taking plus proof in math-education journals.

- Mejia-Ramos, Fuller, Weber, Rhoads, Samkoff (2012): proof comprehension includes "identifying the modular structure" and "higher-level ideas"; a proof has "main components or modules" [E1].
- Leron (1983): structured proofs "arrange development in levels," main ideas on top [E2].
- Lamport (2011): hierarchical proof steps; "The reader can stop opening lower levels of the proof when satisfied" [E3].
- Hodds, Alcock, Inglis (2014): structured proofs help students state key ideas but separate "claims from their supporting evidence" [E4].
- Nothing on personal note granularity.

## Vein 4: mathematical-writing conventions

Confidence: medium (Steenrod unread).

- amsthm: plain (Theorem, Lemma, Corollary, Proposition), definition (Definition, Example), remark (Remark, Notation, Claim) styles; no size rule [W1].
- Halmos (1970): advisors split on "Break long proofs up into lemmas" versus "Don't." Firm rule: "never repeat a proof"; a repeated argument means "there is a lemma that is worth finding, formulating, and proving" [W2].
- Knuth, Larrabee, Roberts (1989): "Motivate the reader... Lemma 1 is motivated by the fact that its converse is true"; proofs must "respect the necessary partial ordering" [W3].
- CASRAI, Richeson, Gasarch: a lemma is "a supporting result proved specifically because it is needed as a step toward a theorem"; "Claim" for in-proof steps too small for lemmas [W4].

## What does not exist

- No book, paper, or essay on scoping personal notes for theorems, proofs, or derivations.
- No such rule from Fast, Tietze, Matuschak, Doto, Aldrich, Gwern, Tao, or Gowers.
- No archived copy of the Holmes article.
- No empirical study of note granularity for mathematical learning.

## Reasoning from first principles (not evidence)

Confidence: low. Any published rule would replace these. Each item names the analogue it is reasoned from.

1. **Unit = one labelled environment** (ProofWiki, PlanetMath, Stacks, amsthm). One note per Definition, Theorem, Lemma, Corollary, Example, or Remark that you would cite on its own.
2. **Proof stays with its statement** unless it is long or there are several (PlanetMath, ProofWiki, Halmos's sheets). Default: proof inside the theorem note; a separate "Proof of X (method)" note only when long or when a second proof exists.
3. **Definitions carry no proofs** (PlanetMath). A definition note states the object and links out; equivalence of two definitions is a theorem note.
4. **Extract a lemma only on reuse** (Halmos, Metamath, Holmes snippet). A step gets its own note when a second result cites it. This is the same "split when cited from outside" trigger the previous Charter used, now with an analogue behind it.
5. **Split conjunctions** (mathlib). "A and B" becomes two notes unless the proof is shared.
6. **Derivations are hierarchical, not atomized** (Leron, Lamport, structured derivations). An ML derivation (bias-variance decomposition, normal equations, ELBO, backprop) is one note of top-level steps, linking out to steps that are theorems in their own right; never split at each algebra line.
7. **A hub per topic lists statements without proofs** (ProofWiki "Properties of X", Mejia-Ramos "modular structure"). This is what a map note is for.

## Could not verify

- Holmes article body (snippets only). Steenrod. Full text of Leron 1983, Fuller 2011, and the one 2023 empirical Zettelkasten study (not math-specific). Reddit. ProofWiki Help:Multiple Proofs.

## Sources

- [P1] https://proofwiki.org/wiki/Help:Proofs ; https://proofwiki.org/wiki/Help:Editing/House_Style
- [P2] https://planetmath.org/planetmathcontentandstyleguide1
- [P3] https://stacks.math.columbia.edu/about ; https://stacks.math.columbia.edu/tags
- [P4] https://ncatlab.org/nlab/show/HowTo
- [P5] https://en.wikipedia.org/wiki/Wikipedia:Manual_of_Style/Mathematics ; https://en.wikipedia.org/wiki/Wikipedia:WikiProject_Mathematics/Proofs
- [P6] https://leanprover-community.github.io/contribute/style.html
- [P7] https://us.metamath.org/mpeuni/conventions.html
- [E1] https://eric.ed.gov/?id=EJ948400 (Mejia-Ramos et al 2012, abstract)
- [E2] https://eric.ed.gov/?id=EJ277000 (Leron 1983, abstract)
- [E3] https://lamport.azurewebsites.net/pubs/proof.pdf
- [E4] https://arxiv.org/abs/1806.06892 (abstract; secondary for Hodds et al)
- [W1] https://www.math.uni-bielefeld.de/~rost/amslatex/doc/amsthdoc.pdf
- [W2] https://sites.math.washington.edu/~lind/Resources/Halmos.pdf
- [W3] https://jmlr.csail.mit.edu/reviewing-papers/knuth_mathematical_writing.pdf
- [W4] https://casrai.org/guides/mathematics-paper-writing-conventions-theorem-proof-structure ; https://blog.computationalcomplexity.org/2023/08/theorems-and-lemmas-and-proofs-oh-my.html ; https://divisbyzero.com (2008 lemma post)
- Practitioners: https://texnicalstuff.blogspot.com/2022/04/zettelkasten-workflow.html ; https://github.com/zhaoshenzhai/MathWiki ; forum.zettelkasten.de/discussion/ 1516, 1877, 1946, 2014, 2019, 2604, 3415 ; forum.obsidian.md/t/ 3394, 16842, 65861, 90472, 102215 ; https://news.ycombinator.com/item?id=23387139 ; https://terrytao.wordpress.com/career-advice/write-down-what-youve-done/ ; https://boffosocko.com/2022/10/27/thoughts-on-zettelkasten-numbering-systems/
- Failed: mildobsessions.com article (dead, no archive); Steenrod text; semanticscholar page for Mejia-Ramos.
