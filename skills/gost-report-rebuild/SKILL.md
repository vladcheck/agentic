---
name: gost-report-rebuild
description: Use when rebuilding existing GOST 7.32 reports to DOCX/PDF with ensure_env.py and soffice. Not for new reports, non-GOST files, or DOCX editing.
---

# GOST 7.32 Report Rebuild

Rebuild an existing GOST 7.32 report project reproducibly. Preserve the report builder and its conventions; change report source only to fix build failures.

## Paths and inputs

- Use `<project>` as the report project root and working directory.
- Use `<gost-report-skill>` as the installed `gost-report` skill directory containing `scripts/ensure_env.py`.
- Resolve figures from `<project>/docs/figures/`; pass bare filenames to `r.figure()`.
- Read personal data from the default configuration locations `~/.config/gost-report/config` and `<course>/.gost-report.env`; never hardcode it.

## Build the DOCX

1. Set the working directory to `<project>` and run:

   ```bash
   python3 <gost-report-skill>/scripts/ensure_env.py .gost-report/build.py
   ```

2. Require exit status 0 and a single `Wrote ...docx` line. Require zero `FAIL` entries. Accept a `WARN` only when it belongs to the same category as the established baseline.
3. If a `FAIL` occurs, edit only the report source and rerun the build. Preserve these source conventions:
   - Do not use em or en dashes in prose.
   - Create figure, table, listing, and formula handles through `r.ref.*` before their first citation; do not introduce forward references.
   - Cite every `r.listing` in the text.
4. Rebuild twice from unchanged source. Extract `word/document.xml` from each DOCX and compare its MD5 digest; the digests must match. The outer ZIP digest may differ because `docProps/core.xml` contains timestamps; this is expected and is not a failure.

## Build the PDF

1. If the builder consumes rendered scheme figures, first re-export them from the cleaned `.kicad_sch` with `kicad-cli sch export svg`, then convert SVG to PNG with `rsvg-convert`, so title and visibility changes reach the report.
2. Run the conversion next to the generated DOCX, using the actual basename reported by the builder:

   ```bash
   soffice --headless --convert-to pdf "<name>.docx" --outdir .
   ```

3. If `soffice` is unavailable, report the exact error and stop. Never substitute a fake PDF.

## Prohibitions

- Never read or parse a `.docx` with the Read tool; verify it through file size and the rebuild log only.
- Never modify the installed `gost-report` skill source.
- Never commit `.kicad_prl` or default `.kicad_pro` churn as design changes.
