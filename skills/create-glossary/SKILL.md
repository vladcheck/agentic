---
name: create-glossary
description: Use this skill when you want to create a glossary of files in current working directory
---

# Create Glossary

## Overview

Build a single self-contained HTML page listing links to all content files in the working directory, recursively, grouped by folder and in reading order. Links must actually resolve — a glossary with broken links is worse than none.

## When to Use

- User asks for a "glossary", "index", "contents page", or "список ссылок" of files/folders
- Course repos, lesson dumps, document collections needing a navigable index

Not for: API docs (use doc generators) or cases where plain `tree` output suffices.

## Core Pattern

1. **Scan** recursively. Include content files (`.html`, `.pdf`, `.mp4`); exclude the glossary file itself, dotfiles (`.DS_Store`), and junk. Add supplementary files (code snippets, zips) only if asked.
2. **Order numerically, not lexicographically.** Sort by the integer in each filename, so `1, 2, 10` never becomes `1, 10, 2`. Don't rely on zero-padding.
3. **Group** by top-level folder (h2), then subfolder (h3). Use `<ol>` for ordered items.
4. **Clean display titles**: strip vendor prefixes (`[Brand] `), the numeric sort key, extension, trailing spaces. Clean group headings too.
5. **Percent-encode every href** with a script, never by hand. Python: `urllib.parse.quote(path, safe='/')` — encodes spaces (`%20`) and brackets (`%5B` `%5D`), keeps Cyrillic literal. Raw spaces/brackets in `href` are invalid HTML; browsers tolerate them on `file://`, but parsers and copied links break.
6. **Verify mechanically, not by eye.** Decode every href and assert the file exists:

```python
import re, urllib.parse, pathlib, sys
root = pathlib.Path('.')
html = (root / 'glossary.html').read_text(encoding='utf-8')
hrefs = [h for h in re.findall(r'href="([^"]+)"', html) if not h.startswith('#')]
broken = [h for h in hrefs if not (root / urllib.parse.unquote(h)).exists()]
print(f'{len(hrefs)} links, {len(broken)} broken')
sys.exit(1 if broken else 0)  # fix and re-run until 0 broken
```

7. **Relative paths only** — the glossary must survive the folder being moved or zipped.

## Quick Reference

| Problem | Fix |
|---|---|
| Link 404s after click | href not encoded — rebuild with `urllib.parse.quote` |
| `1, 10, 2` order | sort by integer from filename, not string |
| Glossary lists itself | exclude output filename during scan |
| Ugly titles | strip prefix + number + extension + trailing space |

## Common Mistakes

- **Raw hrefs with spaces/brackets.** `[Course] 01 Intro .html` as a literal `href` is invalid HTML — encode: `%5BCourse%5D%2001%20Intro%20.html`. Note the trailing space before `.html`: encode it (`%20.html`), never trim the filename.
- **Lexicographic sort.** String sort puts `10` before `2`. Parse the number.
- **Listing the glossary itself on re-runs**, plus dotfiles like `.DS_Store`. Filter by extension allowlist and output-name exclusion.
- **Verifying one link in a browser.** Run the decode-and-exists script over ALL hrefs; one broken out of a hundred is still a failure.
- **Absolute paths** (`/Users/...`, `file://`) — break when the folder moves.
- **Guessing filenames instead of scanning.** Real dumps have inconsistent naming (`QA-сессия` vs `QA - сессия`, stray double spaces). Read the filesystem, never retype names.

## Definition of Done

Every href percent-encoded and mechanically verified (0 broken); groups in folder order, items in numeric order, titles cleaned; single self-contained file, relative links only.
