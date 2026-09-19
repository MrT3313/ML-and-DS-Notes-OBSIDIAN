# Systems Architecture, Machine Learning, and Data Science Notes

An [Obsidian](https://obsidian.md/) vault of systems architecture, machine learning, and data science notes, built from numerous book & online sources.

The notes are restatements, not transcriptions. A note does not count as done until it meets the [Quality Bar](__Meta/Quality-Bar.md).

> [!IMPORTANT]
> If you are here for a summary of a textbook, this is not that. If you are here for a linked reference you can navigate by concept, it is.

## Use

> [!CAUTION]
> Personal study notes, shared in case they are useful. Fork it, take what you want, no attribution needed. Corrections are welcome as issues, though I may be slow, and I will want to verify anything against the source before it lands.

## Opening it

1. Clone the repo.
2. In Obsidian, choose **Open folder as vault** and point it at the clone.

No community plugins are required. The vault uses core plugins only, so there is nothing to install and nothing to trust. Obsidian 1.9 or later is recommended, since the Quality Bar describes index notes built on Bases.

You can also just read the Markdown on GitHub. Nothing here depends on Obsidian to be legible, though `[[wikilinks]]` will not resolve in GitHub's renderer.

## Start here

- **[Home](Home.md)** is the root index and the intended entry point.
- **[Notation](Notation.md)** covers the symbol conventions used across every note. Read it before the math.
- **[Quality Bar](__Meta/Quality-Bar.md)** defines what a finished note contains. It is the closest thing this vault has to a spec.

## Layout

| Path                        | What lives there                                                                                                      |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `Knowledge/`                | The permanent notes.                                                                                                  |
| `Sources/`                  | One note per source, plus one per chapter extracted from it, and one per dataset those chapters work on.              |
| `_Slip Box (Zettelkasten)/` | Landing place for quick notes taken on the fly, before integration. Nothing here is expected to meet the quality bar. |
| `__Meta/`                   | The vault's own documentation: the quality bar, callout reference, note templates, and the study prompt.              |

## Conventions

Notes under `Knowledge/` carry a frontmatter contract, and the required sections vary by `note_kind`. The Quality Bar is the authority on all of it, including what disqualifies a note. A template per note kind, with the frontmatter and headings already in place, is in `__Meta/templates/Note Kinds/`.

## What git tracks

`.obsidian/` ships only the config that makes the vault behave consistently: app settings, which core plugins are on, the templates folder, and property types. Everything per-machine or per-person is gitignored, so pulling will not overwrite your own setup: workspace layout, graph view, appearance, hotkeys, community plugins, themes, and `.trash/` all stay local.

## On the source material

Every note names its sources in frontmatter, and each source has a note of its own under `Sources/`. That is where attribution lives, and it grows as the vault does.

No source text is reproduced here. Notes are restatements and anything quoted directly is marked as a quote and attributed. They are not a substitute for the works they draw on: they are what is left after reading them, and they assume you can reach the original when the details matter.
