---
note_kind: meta
title: "Track A: Obsidian tooling, properties, Bases, Dataview, templates"
date: 2026-09-14
---

Split out of Track A because every schema rule depends on it. Docs read from obsidian.md/help and the obsidian-help GitHub repository (Bases folder committed 2026-09-04; Properties page 2026-08-08). Obsidian 1.14.1 is current. The coordinator independently re-read the Bases syntax, functions, and views pages from the raw repository and confirmed section 5.

## 1. Properties

Confidence: high. Would change my mind: a docs page describing a type-migration cost.

| Item | Docs |
|---|---|
| Types | Text, List, Number, Checkbox, Date, Date and time, Tags [O2] |
| Vault-wide type | "Once a property type is assigned to a property name, all properties with that name across your vault will use the same type." [O2] |
| Changing a type | One click on the type icon, or the Properties view. Docs do not call it costly. Inference: the cost is old values that fail to parse under the new type, not the switch itself. **The prior memo overstated this.** |
| Nesting, bulk edit, markdown | All unsupported: "properties are meant for small, atomic bits of information" [O2] |
| Global rename | "right-clicking it in the All properties view" [O2] |
| `types.json` | Storage file for property types; undocumented in the help repo |

## 2. Forum evidence on property sprawl

Confidence: high. Would change my mind: a thread attributing search failure to raw count with consistent names.

- **Thread 117075** (Lise, 2026-08-08, 15,000 pages, 6 years): "I have too many properties that I don't need, too many inconsistencies, and it's starting to affect how I search." The concrete failure: "`accommodation`, for example, was used as a value for `type` and `keywords` and `travel-acc` among other properties." Verdict: she names both count and inconsistency; the quoted damage is near-synonym keys. Planned remedy: templates with suggesters and deciding "what I might use `type` for in the future (or ditch it)" [OF1].
- **Thread 63891** (2023): kept `aliases`, `type`, `created_at`, `status`; dropped title, dates, `related`, `citation`, `series`, "collecting that information for no real reason." Opi's rule: "If a property is not used to answer a relevant question, it should not be included" [OF2].
- Steph Ango: "Property names and values should aim to be reusable across categories"; "Short property names are faster to type" [SA1].

## 3. Aliases

Confidence: high.

Aliases appear in link suggestions, become the link display text (`[[Artificial Intelligence|AI]]`), and drive unlinked mentions in the Backlinks pane [O5]. Search supports `[aliases:Name]` [O9].

## 4. Maturity versus confidence

Confidence: medium. Would change my mind: a multi-year retrospective showing stage fields kept current.

- Appleton publishes seedling / budding / evergreen plus "planted" and "last tended" dates; she asserts continuous tending and gives no maintenance data [M1, M2].
- Forum 21485 (austin, 2021): "no note is ever done," so seed / sapling / evergreen lacks utility. Of his own replacement tags: "In practice, I never use this tag." A reply admits `#stalled` "never gets used" [OF4]. The alternative he actually uses is topic-driven review, not a status field.
- No long-run study exists in either direction. The evidence for "ladders go unused" is two forum posters; the evidence against is absent. **Thinner than the previous memo implied.**

## 5. Bases capabilities

Confidence: high on the docs, medium on edge behaviour. Would change my mind: a changelog entry adding or removing a listed feature.

| Area | Docs |
|---|---|
| Data read | Frontmatter properties; `file.name`, `file.basename`, `file.path`, `file.folder`, `file.ext`, `file.size`, `file.ctime`, `file.mtime`, `file.properties`, `file.tags` ("Includes inline tags"), `file.links` ("including frontmatter"), `file.embeds` [O8b, O8c] |
| **Backlinks** | **Yes.** `file.backlinks`: "List of backlink files. Note: This property is performance heavy. When possible, reverse the lookup and use `file.links`. Does not automatically refresh results when the vault is changed." Added 1.9.7 (2025-08-05). **The prior memo's "no reverse-link access" is outdated.** [O8b, CL1] |
| Reverse lookup idiom | `file.hasLink(otherFile)`; in a sidebar base, `file.hasLink(this.file)` "replicate[s] the backlinks pane". When embedded, `this` is the embedding note [O8b] |
| Views | table, list, cards, kanban (1.14), map (needs the official Maps plugin) [O8d] |
| Filters | `and` / `or` / `not`, global or per view; functions `hasTag`, `hasLink`, `inFolder`, `hasProperty` [O8b, O8c] |
| Sort, group | Multi-property sort; "grouping by only one property" [O8d] |
| Summaries | Per-column: Average, Min, Max, Sum, Median, Unique, Filled, Empty and more; custom formulas (1.10.3) [O8b] |
| Embedding | `![[File.base]]`, `![[File.base#View]]`, or a ```base code block in any note [O8a, O8b] |
| No `from` | "There is no `from` or `source` like in SQL or Dataview" [O8b] |
| Inline `key:: value` | Never mentioned. Not read (inference from silence) |
| Body, headings, tasks | Not mentioned; only inline tags and links come from the body |

## 6. Aggregates and rollups

Confidence: high. Group-by exists (one property); per-group counts undocumented; no rollup construct. Partial substitute: `file.backlinks` with list functions, non-refreshing.

## 7. Dataview and Datacore status (GitHub, 2026-09-13)

Confidence: high.

| | Dataview | Datacore |
|---|---|---|
| Last release | 0.5.70, **2025-04-07** | 0.1.29, 2026-03-23 |
| Last commit | 2025-04-08 (dependabot only since) | 2026-06-22 |
| Open issues | 637 | 64 |
| Declared status | None | "work-in-progress re-imagining of Dataview"; "the non-Javascript functionality is not available yet" [DC1] |

**The prior memo's "last committed June 2024" was wrong**; the stall began April 2025. Datacore is installable but JavaScript-only and drops the Dataview query language.

## 8. Dataview versus Bases

Confidence: high on facts. Decision left to the coordinator.

- Dataview can, Bases cannot: inline `[key:: value]` fields; auto-refreshing `file.inlinks`; `file.tasks`; DQL `GROUP BY` and `FLATTEN`; DataviewJS.
- Bases has, Dataview lacks: core maintenance; in-place editing; cards, kanban, map; no code in notes.
- Costs of Dataview: 17 months without a release; a second metadata syntax invisible to Bases, property search, and the Properties view; query blocks fail silently when the plugin breaks.

## 9. Templates

Confidence: high.

- Core variables: `{{title}}`, `{{date}}` (default `YYYY-MM-DD`), `{{time}}` (default `HH:mm`), with Moment.js format strings after a colon [O3].
- Absent: logic, prompts, auto-apply on new note, folder targeting.
- Related core helpers: default location for new notes; right-click folder, New note; Unique note creator; Daily notes template.
- Templater (community, flagged): folder templates, prompts, JavaScript, file moves [T1].

## 10. Other core features for a folder-first user

Confidence: high.

- Folder notes: not core; open feature requests only [OF12].
- File explorer sorts by name, modified, or created; no manual order [O10].
- **Properties view** ("All properties"): every property in the vault with type and frequency, sortable by frequency, right-click to rename globally. This is the native schema-drift audit [O11].
- Search operators: `[prop]`, `[prop:value]`, `path:`, `file:`, `tag:` [O9].

## Could not verify

- Whether `file.backlinks` works inside filters, as opposed to formulas (forum titles suggest friction; not read).
- Per-group counts in group-by views.
- Bases changes in 1.11.x (changelog feed began at 1.12).
- Inline-field exclusion is inferred from silence.

## Sources

- [O2] https://obsidian.md/help/properties
- [O3] https://obsidian.md/help/plugins/templates
- [O5] https://obsidian.md/help/aliases
- [O8a] https://obsidian.md/help/bases
- [O8b] https://obsidian.md/help/bases/syntax (raw: github.com/obsidianmd/obsidian-help/blob/master/en/Bases/Bases%20syntax.md)
- [O8c] https://obsidian.md/help/bases/functions
- [O8d] https://obsidian.md/help/bases/views
- [O9] https://obsidian.md/help/plugins/search
- [O10] https://obsidian.md/help/plugins/file-explorer
- [O11] https://obsidian.md/help/plugins/properties
- [CL1] https://obsidian.md/changelog/2025-08-05-desktop-v1.9.7/ ; https://obsidian.md/changelog/2025-11-11-desktop-v1.10.3/
- [OF1] https://forum.obsidian.md/t/yaml-and-template-spring-cleaning-after-6-years-and-way-too-many-properties/117075
- [OF2] https://forum.obsidian.md/t/obsidian-properties-best-practices-and-why/63891
- [OF4] https://forum.obsidian.md/t/alternative-to-tracking-how-done-a-note-is-eg-seed-sapling-evergreen-tree/21485
- [OF12] https://forum.obsidian.md/t/95077 ; /t/108760 (folder note feature requests)
- [M1] https://maggieappleton.com/garden-history ; [M2] https://maggieappleton.com/evergreens
- [SA1] https://stephango.com/vault
- [T1] https://silentvoid13.github.io/Templater/
- [DC1] https://github.com/blacksmithgu/datacore (README, ROADMAP) ; https://github.com/blacksmithgu/obsidian-dataview (releases, commits, issue 1825)
- Failed (404): obsidian.md/help/bases/filters, /bases/formulas, /bases/embed (recovered from the GitHub raw files)
