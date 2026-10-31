---
name: obsidian-add-links-to-note
description: Adds wikilinks to Obsidian note body text with Russian inflection. Use when asked to link terms in a note.
license: MIT
user-invocable: true
metadata:
  deprecated: no
  tags: [obsidian]
---

# Add Links to Note

## Requirements

User gave an Obsidian note and asked to put links in its body text.

## Instructions

1. Link terms as `[[Target|surface]]` so they read as natural inflected Russian.
2. Target is the exact note title (or `Note#Section`). Forward links to non-existing notes are allowed — report them after editing.
3. Surface (after `|`) is the word as the sentence needs it: correct Russian case/number/gender, lowercase unless a proper noun. Omit the alias only when surface equals target exactly.
4. Span may be one word, several words, or a phrase — prefer the most specific existing note (e.g. `[[История живописи]]` over `[[История]] [[Живопись|живописи]]`).
5. Density: link a term once per note; skip generic words (`функция`, `структура`, `люди`) with no dedicated note. If nothing qualifies, say the note needs no body links and change nothing.
6. Uncertainty → ask, never guess: multiple candidate targets with no contextual cue, unclear whether a term deserves its own note, no exact note and only an approximate one — leave unlinked, list each case to the user after editing. Exception: when surrounding context clearly disambiguates, link the fitting target and report the choice.
7. Change link markup only — no prose rewrites.

## Examples

Bad: `Дело в [[История (Наука)]], а не в людях.`
Good: `Дело в [[История (Наука)|истории]], а не в людях.`

Bad: `Интерфейсы в [[Go Язык программирования]] - инструмент...`
Good: `Интерфейсы в [[Go Язык программирования|Go]] - инструмент`
(use the alias only if the sentence needs the short form; omit it when surface equals target).

## After Editing

Report: links pointing to non-existing notes; every ambiguity left unresolved and what needs deciding.
