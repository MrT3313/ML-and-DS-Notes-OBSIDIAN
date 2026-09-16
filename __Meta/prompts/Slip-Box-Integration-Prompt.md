# Slip box integration prompt

Paste into a fresh conversation with an agent that has file access to the vault and can launch subagents. Replace the three bracketed values at the top. The prompt has one approval gate, after the plan and before any file is written. To run it end to end without stopping, delete the single line that begins "GATE:".

This is a different role from `Chapter-Study-Prompt.md`. The study prompt's agent is an interrogator and never writes note bodies. This prompt's agent moves notes I already wrote in my own words from the slip box into their permanent place, restating and completing them. Everything it adds beyond what the raw file says is reported back to me at the end so I can read it as a reviewer, not a reader.

---

```
You are the coordinator agent for a knowledge-integration task in an Obsidian vault. The vault root is the current working directory. You direct subagents, you own the plan, and you own the final verification. The subagents do the reading of the repository and the writing of the notes. You personally read only the files named in Phase 2 and Phase 3, plus whatever you need to verify the result.

Raw notes to integrate: [_Slip Box (Zettelkasten)/RAW HOML Ch03 Classification.md]
Chapter: [HOML Ch03 Classification]
Chapter notebook URL: [https://github.com/ageron/handson-ml3/blob/main/03_classification.ipynb]

The raw file is my own notes, taken while reading the chapter, in my own words. It is the source of truth for what I understood. The vault has a strict shape and a strict quality bar, and the job is to move every atom of the raw file into that shape, so that at the end the raw file is empty and the vault has grown by one chapter.

Hard rules for the whole session, for you and every subagent:
- Never run `git add`, `git commit`, `git stash`, `git checkout`, `git restore`, or anything else that stages, commits, or discards. `git status`, `git diff`, `git log` and `git show` are allowed, read only.
- Never delete the raw file. It is emptied in Phase 6 by truncating its contents to zero bytes, and only then.
- Never delete an existing note without listing the deletion in the plan and having it approved at the gate.
- Every file is owned by exactly one subagent per phase. Two subagents never write the same file.
- No em-dashes and no en-dashes anywhere, in prose, in tables, or in code comments. Use commas, colons, parentheses, or a new sentence. The raw file contains some; normalize them.
- Never write "e.g." Write "ex" or restructure the sentence.
- Every note body is a restatement in the vault owner's voice. The raw file's wording is preferred wherever it already says the thing. Book phrasing is never copied. Anything quoted verbatim from any source is marked as a quote and attributed.
- The scikit-learn version pinned across the vault is 1.6. Every `## Implementation` and `### In scikit-learn` section says "scikit-learn 1.6:" in the prose above its code, and the code must be correct for that version.

Work in six phases. Do not skip a phase and do not start writing files before the gate.

PHASE 1: STRUCTURE (subagents, in parallel, read only).
Launch two read-only exploration subagents at once and wait for both.

Subagent A, the vault contract. Have it read in full and report back, quoting rather than paraphrasing wherever a rule is exact: `__Meta/Quality-Bar.md` (the whole file, it is the spec), `README.md`, `Home.md`, `Notation.md`, `__Meta/Callouts.md` (only the list of callout types), every file in `__Meta/templates/`, and `.obsidian/types.json`. Its report must contain: the frontmatter contract per note_kind, the required section headings per note_kind verbatim, the fixed discharge string for a non-quantitative Formal statement, the hyperparameter test, the disqualification list, the heading rules (no H1 under Knowledge, `##` for required sections, `###` for subsections), and the deferred-decisions list with its triggers.

Subagent B, the vault as it stands. Have it produce: the full tree of `Knowledge/` and `Sources/` with one line per file giving path, note_kind, `up`, and `confidence`; the complete text of `Knowledge/Machine Learning/Machine Learning.md` including its Areas list and its What is missing list; the complete text of `Knowledge/Machine Learning/Tasks/Classification.md`; the complete text of `Sources/Books/HOML/HOML Ch02 End-to-End Machine Learning Project.md` as the exemplar of a finished chapter note; the output of `git show --stat 5a1a0c2` as the exemplar of what a chapter integration touches (new notes, the hub, Home, the container note, and backfilled `sources:` and `up:` on existing notes); every line anywhere in `Knowledge/`, `Sources/`, `Home.md` and `README.md` that mentions the chapter being integrated by number or name, because each one is a promise this chapter must keep; and every `[[wikilink]]` target in the raw file that does not resolve to an existing note. Also have it report which existing notes end with a ```base block and which do not, so new notes can follow the pattern of their neighbours rather than invent one.

Do not proceed until both reports are in. Keep both reports in your context for the rest of the session.

PHASE 2: READ THE RAW NOTES (you, personally, in full).
Read the raw file from the first line to the last with no offset and no limit. If the file is long, read it in consecutive chunks until you have seen the final line, and say the final line back to yourself so you know you reached it. Do not delegate this. Do not summarize it yet.

Then read, yourself, four exemplar notes so you know what a finished note looks like in each shape you will be producing: `Knowledge/Machine Learning/Objectives/Metrics/Root Mean Squared Error.md` (a metric, note_kind concept, with a `### In scikit-learn` subsection), `Knowledge/Machine Learning/Evaluation/Cross-Validation.md` (a method, and the precedent for amending an existing note with a `### From Chapter N` subsection instead of rewriting it), `Knowledge/Machine Learning/Models/Linear/Logistic Regression.md` (a model, note_kind method, with a hyperparameter table), and `Knowledge/Machine Learning/Tasks/Classification.md` (an index).

PHASE 3: INVENTORY AND PLAN (you, personally).
Build the plan in this order and write it to a scratch file outside the vault, not into the vault.

Step 3a, the inventory. Split the raw file into atoms. An atom is the smallest piece that lands in one place: usually one heading with its bullets and code, sometimes a single bullet or a single code block when its siblings go elsewhere. Number the atoms in file order. Every line of the raw file belongs to exactly one atom. Code blocks that only regenerate the book's figures, and code that is formatting only, are still atoms; they are allowed to be dropped, but only by an explicit decision with a reason.

Step 3b, the destination table. One row per atom: atom number, raw heading, action, target path, note_kind, `up`, what this atom contributes, and why here. Actions are exactly one of: CREATE a new note, AMEND an existing note (say which section), RENAME or MOVE an existing note (old path to new path), DELETE an existing note (with the reason and where its content goes), DROP the atom (with the reason). A raw atom may feed several rows, ex: the SGD threshold bullets feed both a model note and a metric note. Every unresolved `[[link]]` from the Phase 1 report must appear as a CREATE row or the plan is incomplete.

Step 3c, the structural decisions. Decide and write down, with the reason, every placement question the chapter raises, checked against the Areas list in the machine learning hub and against the folder rule (the folder says what kind of thing a note is, `up` says what it is part of). At minimum decide: where classification metrics live, and note that at least one existing note names a folder for them that does not exist, so the plan must say which of the existing promises is kept and which is corrected; whether the classification task varieties (binary, multiclass, multilabel, multioutput) become separate concept notes and where, following the precedent of how paradigms are split into subfolders; where the binary-to-multiclass strategies go, given that scikit-learn implements them as meta-estimators that act on estimators, which is the stated job of the `Composition/` folder, while the raw notes file them under classification; where each classifier model lands under `Models/` and whether it earns a new family folder; whether a dataset note is required, given that the quality bar wants one per dataset a chapter works on and the raw file's code plainly runs on one without ever naming it; whether the `Classification` index's scope statement and Methods list change shape now that methods and measures exist; whether any existing note should convert between `concept` and `index` under the rule that `index` is earned by having methods to list; and which deferred decision in the quality bar, if any, this chapter's trigger touches. Do not add a new note_kind.

Step 3d, the promises. For every line the Phase 1 report found that mentions this chapter, write the exact edit that retires the promise: the machine learning hub's What is missing list and its Areas paragraphs, the `Performance Measure` note, the `Classification` index, the `Logistic Regression` hyperparameter table's "how to tune" cell for the threshold, the `HOML` container note (plain text chapter line becomes a link), `Home.md`'s Sources line, and `README.md`'s layout table. Add any others the report found.

Step 3e, the chapter note. Plan `Sources/Books/HOML/[chapter file name].md` from `__Meta/templates/Chapter Notes.md`: frontmatter with `note_kind: source`, `medium: book`, `chapter:` as a number, `up: "[[HOML]]"`, `url:` set to the notebook URL above, and aliases for the chapter number and the chapter title; then `## Thesis`, `## Extracted` grouped by theme with a group for notes amended rather than produced, and `## Open questions`. It takes no H1. Also plan which earlier open questions, in the chapter 1 and chapter 2 notes, this chapter settles or partly settles, and what edit records that.

Step 3f, the research list. For every claim in the raw file that is uncertain, marked with a question mark, internally inconsistent, or likely wrong, write a research item: the claim, what to check, and which primary source settles it. Primary sources, in order of trust, are the scikit-learn 1.6 API reference and user guide, the chapter notebook at the URL above, and the book's own text if the agent has it. Do the same for every scikit-learn call in the raw file: the class or function name, its module, its default arguments, and its behaviour on the version pinned must be confirmed, not assumed. Include the arithmetic: every formula in the raw file gets rederived or checked against a reference.

Step 3g, the work packets. Group the destination rows into packets such that no file appears in two packets, and packets are independent of one another. Then fix, before any packet is dispatched, the final title of every note that will be created or renamed, because every packet links to notes owned by other packets and the link text must already be right. Each packet contains: the atom text verbatim, the target file or files and the action on each, the note_kind and its exact required sections from the Phase 1 report, `up`, a seed list of aliases (every name a person would type in a link, including plurals, abbreviations, and the scikit-learn identifier), the outbound links this note must make and the phrase that explains each relationship, the notes elsewhere that will link back to it, the research items that belong to it, the exemplar note to read first, and the style rules from the hard rules above. Every packet also carries: "Do not touch any file not named in this packet. Do not run git write commands. Report back before finishing."

Write the plan as: the destination table, the structural decisions with reasons, the promise edits, the chapter note outline, the research list, and the packet list. Then present it to me in full.

GATE: Stop here and wait for my explicit approval of the plan. Apply any changes I ask for to the plan before continuing. Do not write any file inside the vault before I say go.

PHASE 4: EXECUTE (subagents, in parallel where packets are independent).
Dispatch every packet to its own writing subagent. Independent packets go out together in one batch. A packet that amends a file another packet creates waits for that packet to finish. Each writing subagent, in this order:
1. Reads `__Meta/Quality-Bar.md` and the exemplar note named in its packet.
2. Reads every existing note it is told to link to, so the link phrase is true and the `up` chain is consistent.
3. Does the research in its packet against primary sources, and records for each item: the finding, the source, and whether it confirms, corrects, or extends the raw notes. Research is for correctness and completeness. It is not licence to replace the raw file's wording with a source's wording.
4. Writes or amends the note to the contract for its kind. New notes get `confidence: draft`, never anything higher. Amended notes get this chapter appended to `sources:` and, where the amendment is more than a sentence, a `### From Chapter N` subsection in the pattern of the exemplar rather than a rewrite of what earlier chapters said. A ```base block is appended only where the note's neighbours in the same folder carry one.
5. Self-checks before reporting: every required section present and non-empty, `## Where it is used` has at least two outbound links each carrying its relationship phrase, every quantitative claim has a formula somewhere in the note, the discharge string is exact if used, no H1, no dash characters of either kind, no "e.g.", no placement commentary in the body, frontmatter keys spelled exactly, `up` is a link and is never `[[Home]]` for a leaf.
6. Reports back with: files written or changed, with a one-line summary each; every fact in the raw notes it corrected, with the raw wording, the corrected wording, and the source; every claim it added that the raw notes did not contain, with the source; every research item it could not settle, stated as uncertain rather than guessed; and any deviation from its packet with the reason.

Keep every report. You will need them in Phase 6.

PHASE 5: VERIFY (you, personally, with scripts where possible).
Run these checks over the whole vault and fix what fails, either yourself for one-line fixes or by sending the owning subagent back for anything larger:
- Every `[[wikilink]]` in `Knowledge/`, `Sources/`, `Home.md` and `README.md` resolves to an existing file, allowing for `|alias` display text and `#heading` suffixes.
- Every note under `Knowledge/` has `note_kind`, `up`, and (where its kind requires them) `aliases`, `sources`, and `confidence`, with the keys spelled exactly. Every `up` value resolves.
- Every new or amended note carries the required `##` sections for its kind and no `#` heading.
- No file under `Knowledge/`, `Sources/`, `Home.md` or `README.md` contains an em-dash, an en-dash, or the string "e.g.".
- Every note the chapter produced or amended is listed under `## Extracted` in the chapter note, and every note listed there exists and names the chapter in its `sources:`.
- Every promise line from Phase 3d is gone or rewritten.
- The `HOML` container links the chapter, `Home.md` counts it, and `README.md`'s layout table counts it.
- Every atom in the inventory is accounted for: walk the destination table row by row and confirm the content actually landed where the row says, in substance, not just by file existence. An atom marked DROP has its reason in the plan.
- `git status --short` shows only the files the plan named. Anything else is a mistake to reverse by hand, never with a git command.

PHASE 6: EMPTY THE RAW FILE AND REPORT.
Only when every check in Phase 5 passes, truncate the raw file to zero bytes. Do not delete it and do not move it. Then give me the final report, which has exactly these parts:
1. A table of every file created, amended, renamed, moved or deleted, one row each, with the reason in a few words.
2. Structural decisions taken, one line each, with the reason.
3. Corrections: every place the notes now say something different from what my raw file said, with the raw wording, the new wording, and the source. This is the most important section. Writing is how I learn, so I need to know exactly where my understanding was wrong.
4. Additions: every substantive claim in the vault that came from research rather than from my raw file, with the source, so I can read them as a reviewer.
5. Unsettled: every research item no subagent could settle from a primary source, with what to check in the book.
6. The chapter note's open questions, restated, and which earlier open questions this chapter settled.
7. The output of `git status --short`, unstaged, uncommitted.
No praise padding and no summary of what a good job was done. Facts, decisions, corrections, and gaps.
```
