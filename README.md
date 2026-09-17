# ML and DS Notes

An Obsidian vault of machine learning and data science notes, built from numerous book & online sources.

The notes are restatements, not transcriptions. Every concept is written in my own words against a stated source, and a note does not count as done until it meets the [Quality Bar](__Meta/Quality-Bar.md). If you are here for a summary of a textbook, this is not that. If you are here for a linked reference you can navigate by concept, it is.

## Opening it

1. Clone the repo.
2. In Obsidian, choose **Open folder as vault** and point it at the clone.

No community plugins are required. The vault uses core plugins only, so there is nothing to install and nothing to trust. Obsidian 1.9 or later is recommended, since the [Quality Bar](__Meta/Quality-Bar.md) describes index notes built on Bases.

You can also just read the Markdown on GitHub. Nothing here depends on Obsidian to be legible, though `[[wikilinks]]` will not resolve in GitHub's renderer.

## Start here

- **[Home](Home.md)** is the root index and the intended entry point.
- **[Notation](Notation.md)** covers the symbol conventions used across every note. Read it before the math.
- **[Quality Bar](__Meta/Quality-Bar.md)** defines what a finished note contains. It is the closest thing this vault has to a spec.

## Layout

| Path | What lives there |
|------|------------------|
| `Knowledge/` | The permanent notes, filed by what kind of thing they are. `Machine Learning/` is the main body, `MLOps/` is early. |
| `Sources/` | One note per source, plus one per chapter extracted from it, and one per dataset those chapters work on. Currently HOML (Géron, 2022), chapters 1 to 4, and three datasets, California housing, MNIST and iris. |
| `_Slip Box (Zettelkasten)/` | Landing place for quick notes taken on the fly, before integration. Nothing here is expected to meet the quality bar. |
| `__Meta/` | The vault's own documentation: the quality bar, callout reference, note templates, and the study prompt. |

Folder placement and the `up` frontmatter key answer different questions. The folder says what kind of thing a note is; `up` says what it is part of. A note can sit in `Models/Linear/` and point `up` at `[[Classification]]`, and both are correct.

## Conventions

Notes under `Knowledge/` carry a frontmatter contract: `note_kind`, `aliases`, `up`, `sources`, and `confidence`. Required sections vary by `note_kind` (`concept`, `method`, `index`, `source`, `meta`). The [Quality Bar](__Meta/Quality-Bar.md) is the authority on all of it, including what disqualifies a note. Templates for new source and chapter notes are in `__Meta/templates/`.

## What git tracks

`.obsidian/` ships only the config that makes the vault behave consistently: app settings, which core plugins are on, the templates folder, and property types. Per-machine and per-person state is gitignored, so pulling will not overwrite your own setup:

- Your workspace layout, graph view settings, appearance, and hotkeys stay local.
- Community plugins and themes are not vendored into the repo. Install whatever you like; git will not see them.
- `.trash/` is ignored, so notes you delete locally are not pushed back.

If you fork this and want your own Obsidian preferences tracked, that is a `.gitignore` change on your side.

## On the source material

Every note names its sources in frontmatter, and each source has a note of its own under `Sources/`. That is where attribution lives, and it grows as the vault does.

No source text is reproduced here. Notes are restatements in my own words, and anything quoted directly is marked as a quote and attributed. They are not a substitute for the works they draw on: they are what is left after reading them, and they assume you can reach the original when the details matter.

## Use

Personal study notes, shared in case they are useful. Fork it, take what you want, no attribution needed. Corrections are welcome as issues, though I may be slow, and I will want to verify anything against the source before it lands.
