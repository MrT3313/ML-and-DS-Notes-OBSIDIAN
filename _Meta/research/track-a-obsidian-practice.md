---
note_kind: meta
title: "Track A: Obsidian and note-taking practice"
date: 2026-09-14
---

Second full pass, researched from scratch. Companion memos: [[track-a-tooling-bases]] (properties, Bases, Dataview, templates) and [[track-a-stem-atomicity]] (theorems and proofs). About 1,700 words: the verification table alone is ten rows.

## 1. Verification of the previous memo's claims

| # | Prior claim | Verdict | Evidence |
|---|---|---|---|
| 1 | Anti-folder argument targets topic folders, not folders as such; nobody objects to kind folders | **Partly verified** | Tietze: "Creating categories is a top-down process" [Z2]. Matuschak's note never mentions folders [A6]. But per-se objectors exist: Sascha Fast, "One folder and that's it" [ZF4], 7,400 notes in one folder [ZF5], folders "mostly ornamental if you use the system-wide search" [ZF6]; Bob Doto, "Both zettelkasten and LYT eschew the use of folders" [D1]. Nobody argues *against* kind folders; Milo, Konik, and Steph Ango use them [L4, K1, SA1]. |
| 2 | Obsidian's docs take no position on folders versus links | **Verified** | Seven help pages read; mechanics only [O1 to O7]. |
| 3 | Split on concept count, not length (attributed to Sascha) | **Partly verified** | Quote exists but is by forum user andang76, about structure notes: "I split structure notes when they grows not in terms of 'size' (number of links), but in terms of number of concepts" [ZF3]. Not Sascha, not about content notes. |
| 4 | Atomicity is an output notes approach, not an entry gate | **Verified** | Sascha (2025): "notes are not atomic for a long time, but continuously approach atomicity as a desired outcome" [Z4]. |
| 5 | Matuschak names the multi-source failure as "no accumulation" | **Verified** | "With this approach, there's no accumulation" [A4]. |
| 6 | Do not stack citekeys; state divergence in prose | **Contradicted in part** | Sascha's own example stacks them: "Your best description of what the technique is about.[citekey1][citekey2]... According to author 1, the step differs: z.[citekey1]". "Don't hide behind any authors!" holds [ZF1]. emps: minor differences collapse into one note; crucial ones get a note per source plus a comparison note [ZF1]. |
| 7 | Matuschak abandoned evergreen notes for flat outlines | **Not verified** | The Patreon post is paywalled; no public primary text says "abandoned"; his notes site describes the system in present tense [A8]. Only a secondary paraphrase exists [W1]. Do not assert it. |
| 8 | Milo's MOC substance is paywalled | **Partly verified** | Free pages give the essentials: squeeze point "when your unsorted knowledge becomes so messy it overwhelms and discourages you"; a MOC is "a note where you collect a bunch of related notes, then you stare at it, and think" [L1, L2]. Milo also publishes a folder framework (ACE: Atlas, Calendar, Efforts) and says "structure must be earned" [L4]. |
| 9 | Claim-phrase titles cost tree browsers | **No source either way** | Matuschak's title notes never mention file lists [A2, A3]; davecan grants "lexical titles still have value for organizing concepts" [OF10]. |
| 10 | Structure notes become necessary at 500 to 1500 notes | **Verified with nuance** | Hubs appeared "between 500 and 700 notes"; insufficient "between 1000 and 1500" [Z3]. |

## 2. Scoping and titling

Confidence: high. Would change my mind: a source showing claim titles working in an alphabetical explorer.

- Matuschak: notes "only about one thing, but which, as much as possible, capture the entirety of that thing"; "no clear litmus test or correct answer here, just a bunch of tradeoffs" [A1].
- zettelkasten.de's operational tests: easy to name, understandable at a glance, nothing removable, nothing missing [Z1].
- Sascha 2025: split when you "discover mistakes where notes contain multiple ideas" or when active work needs it; atomicity is "part of the journey" [Z4].
- Titles: Matuschak wants complete claims because "those titles become an abstraction for the note itself" [A2, A3]. He exempts outline notes and definitions.
- **Folder-first conflict.** Claim titles are long and sort by first word in the explorer. No source addresses this. Inference: noun titles for notes navigated by folder, with the claim as the first line of the body.
- For theorems, proofs, and derivations see [[track-a-stem-atomicity]].

## 3. Connecting notes: maps and structure notes

Confidence: high on content, medium on timing. Would change my mind: evidence that squeeze-point maps fail learners following a textbook order.

| Source | Unit | Function | When |
|---|---|---|---|
| Sascha [Z3] | Structure note: "a Zettel about other Zettels and their relationships"; layered into main structure notes | Ordered tables of contents | Hubs at 500 to 700 notes; layers at 1000 to 1500 |
| Milo [L1, L2] | Map of Content: "gather, develop, and navigate" | A workplace: "stare at it, and think" | Mental squeeze point |
| Doto [D2] | Hubs "document existing connections"; MOCs "are places to challenge notes" | | |
| Sascha [Z5] | Link context: "always state explicitly why you made it" | | |

**Folder-first conflict.** Maps are invisible in the tree unless they have a folder. Milo's Atlas folder is the only mainstream answer that puts maps in the explorer [L4]; Sascha marks main structure notes with tags. Forum practice adds a per-folder overview note [OF5].

## 4. Folders versus links, for a folder navigator

Confidence: medium. Would change my mind: a multi-year report of topic folders surviving without reorganization.

- Consensus among folder users (Milo, Konik, Ango, amanhimself, Tim Miller): folders for kind, lifecycle, or workflow; links for topic. amanhimself: "Folders encode lifecycle, links encode topics" [AM1]. Konik: "if something could conceivably fit in two different folders I need to consolidate my folders" [K1].
- Milo's folder rule: "Test your folder structure slowly... structure must be earned"; folders are "rigid and exclusionary by their nature" yet fine for "active projects, temporary inboxes, or clearly-defined note categories like images or quotes" [L4].
- Steph Ango (Obsidian CEO): "I avoid folders because many of my entries belong to more than one area of thought"; keeps only kind folders (References, Clippings, Attachments, Daily, Templates) and organizes topics with a `categories` property viewed through Bases [SA1].
- Dissent: Sascha, one folder [ZF4, ZF5]; Doto, folders are storage and link notes are the interface [D1, D3].
- **Conflict stated plainly.** No source supports *topic* folders long term. Every source that uses folders keys them on kind or workflow. A learner who navigates by folder and wants topics in the tree is outside every author's recommendation. The nearest sanctioned pattern is Milo's Atlas: maps in a visible folder.

## 5. One idea, many sources

Confidence: high. Would change my mind: evidence that per-source notes accumulate with heavy linking.

- Matuschak: source-oriented notes mean "there's no accumulation"; concept-oriented notes are the fix [A4].
- Sascha's model [ZF1]: your own formulation, citekeys after each claim (stacked where sources agree), divergence in prose ("According to author 1, the step differs"), "Don't hide behind any authors! It is now your description of this technique. Stand behind it."
- emps [ZF1]: minor differences, one note; crucial differences, one note per source plus a comparison note.
- One literature note per source plus one concept note that owns the idea: Konik [K1], Curatella [ZF6], amanhimself [AM1].
- Tietze's caution: revising permanent notes causes "context loss and obscured intellectual history" [Z5]. Mitigation is the provenance rule above.

## 6. Splitting and parent/child

Confidence: medium (thin evidence). Would change my mind: a source recommending topic subfolders inside kind folders.

- Split trigger: concept count for structure notes [ZF3]; for content notes, Sascha's "revealed by work" [Z4]. Long notes are amalgams and refactoring is costly [ZF2].
- Parent/child is recorded in the parent's outline (structure notes, MOCs), never in folders, by every source read. Konik's Johnny Decimal depth is by kind, not topic [OF6].
- Rename updates links, so splitting is safe [O6].

## 7. Multi-year retrospectives on folder schemes

Confidence: medium. Would change my mind: a retrospective endorsing topic folders.

| Who | Span | Outcome |
|---|---|---|
| amanhimself [AM1] | 5 years | Tried PARA, Zettelkasten, Johnny Decimal, "ended up dissatisfied"; settled on 8 kind and lifecycle folders |
| makeuseof [MU1] | 3 years | "wasted months reorganizing"; flat, "no more than two or three nested levels" |
| Tim Miller [R2] | | Five workflow folders, "not for organization, but to improve my workflow" |
| Leventov [LE1] | 2 years | Evergreen notes "weakly related to the mental maps of concepts"; dense linking alone did not aid learning |
| gibraltar [OF11] | | "'notes from me' and 'notes from others'. That's it." |

Pattern (inference): survivors converge on few, shallow, kind or lifecycle folders. None endorses topic folders; none condemns kind folders.

## 8. Where authors disagree

- **Folders:** Sascha and Doto, one folder; Milo, ACE folders plus maps; Konik, Johnny Decimal kind folders; Ango, root plus reference folders plus a property.
- **Map role:** Doto, hubs documentary versus MOCs generative; Sascha, ordered tables of contents.
- **Titles:** Matuschak, claims; davecan, nouns keep value; Milo's maps, topic nouns.
- **Citekeys:** Sascha stacks them; the previous memo said not to.
- **Split triggers:** Matuschak, tradeoffs; Sascha, revealed by work; both reject a gate.
- **Doto says LYT eschews folders; Milo publishes a folder framework.**

## Could not verify

- Matuschak's Patreon "Five years of evergreen notes" (paywalled; archive blocked).
- Milo's ACCESS definitions beyond a tweet snippet.
- Konik's 2025 setup.
- Any source on claim-title cost in alphabetical explorers.

## Sources

- [Z1] https://zettelkasten.de/atomicity/guide/
- [Z2] https://zettelkasten.de/posts/no-categories/
- [Z3] https://zettelkasten.de/posts/three-layers-structure-zettelkasten/
- [Z4] https://zettelkasten.de/posts/principle-of-atomicity-difference-between-principle-and-implementation/
- [Z5] https://zettelkasten.de/introduction/
- [ZF1] https://forum.zettelkasten.de/discussion/1890/how-should-one-approach-making-in-text-citations-for-similar-ideas-from-different-sources
- [ZF2] https://forum.zettelkasten.de/discussion/1651/how-long-a-zettel-is-too-long
- [ZF3] https://forum.zettelkasten.de/discussion/3283/is-there-a-better-way-to-organise-structure-notes
- [ZF4] https://forum.zettelkasten.de/discussion/1934
- [ZF5] https://forum.zettelkasten.de/discussion/247
- [ZF6] https://forum.zettelkasten.de/discussion/2754
- [A1] https://notes.andymatuschak.org/Evergreen_notes_should_be_atomic
- [A2] https://notes.andymatuschak.org/Evergreen_note_titles_are_like_APIs
- [A3] https://notes.andymatuschak.org/Prefer_note_titles_with_complete_phrases_to_sharpen_claims
- [A4] https://notes.andymatuschak.org/Evergreen_notes_should_be_concept-oriented
- [A6] https://notes.andymatuschak.org/Prefer_associative_ontologies_to_hierarchical_taxonomies
- [A8] https://notes.andymatuschak.org/About_these_notes
- [W1] https://writingslowly.com/2024/09/18/how-to-write.html
- [L1] https://www.linkingyourthinking.com/ideaverse/youre-already-doing-it-sort-of
- [L2] https://blog.linkingyourthinking.com/notes/lyt-faq
- [L4] https://blog.linkingyourthinking.com/notes/ace-folder-framework
- [D1] https://writing.bobdoto.computer/zettelkasten-linking-your-thinking-and-nick-milos-search-for-ground/
- [D2] https://writing.bobdoto.computer/misconceptions-about-the-relationship-between-permanent-and-evergreen-notes/
- [D3] https://writing.bobdoto.computer/how-i-use-clogs-to-organize-my-writing-files/
- [K1] https://www.eleanorkonik.com/p/yet-another-hot-take-on-folders-versus-tags
- [SA1] https://stephango.com/vault
- [R1] https://obsidian.rocks/maps-of-content-effortless-organization-for-notes/
- [R2] https://obsidian.rocks/how-i-use-folders-in-obsidian/
- [AM1] https://amanhimself.dev/blog/minimal-obsidian-vault-structure/
- [MU1] https://www.makeuseof.com/i-wish-i-knew-these-before-creating-my-obsidian-vault/
- [LE1] https://engineeringideas.substack.com/p/reflection-on-two-years-of-writing
- [O1 to O7] https://obsidian.md/help/ pages: manage-notes, Create a vault, Internal links, Tags, Properties, Manage vaults, bases
- [O6] https://obsidian.md/help/Files+and+folders/Manage+notes
- [OF5] https://forum.obsidian.md/t/how-to-organize-an-overview-note-for-a-folder/92397
- [OF6] https://forum.obsidian.md/t/increasingly-atomic-folders-a-workflow/14345
- [OF10] https://forum.obsidian.md/t/whether-to-use-sentences-or-key-phrases-in-the-titles-of-notes/16718
- [OF11] https://forum.obsidian.md/t/build-the-structure-of-a-vault/89980
- Failed to load: patreon.com/posts/five-years-of-109216672 (403), obsidian.rocks/how-to-organize-your-notes-in-obsidian/ (404), glasp.co transcript of Milo's ACCESS video (403).
