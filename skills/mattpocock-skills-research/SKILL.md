---
name: mattpocock-skills-research
description: Use when evaluating upstream agent skills for adoption with unclear maturity, maintenance, or issues. Not for authoring, fixes, or repo reviews.
---

# Research upstream skills

1. Identify the upstream as `<owner>/<repo>`, the candidate skill names, and the adoption question. Inspect local copies first, starting with the default `~/.claude/skills/<name>/SKILL.md` and any configured runtime-specific skill roots; record differences from upstream rather than assuming local files are current.

2. Establish repository context and read its own documentation before interpreting directory names:
   ```bash
   gh repo view <owner>/<repo> --json nameWithOwner,description,defaultBranchRef,url
   gh api repos/<owner>/<repo>/contents --jq '.[].path'
   gh api repos/<owner>/<repo>/contents/<path> -H "Accept: application/vnd.github.raw+json"
   ```
   Read the relevant README files, contribution or support docs, release notes, package or plugin manifests, distribution configuration, and the candidate skill's metadata.

3. Derive each skill's status from repository-specific evidence. Distinguish documented stable, beta, experimental, in-progress, deprecated, and undocumented states using docs, layout, distribution metadata, and releases together. Treat directory names and inclusion in a distributed package as signals only when that repository explains their meaning; label conflicting or missing evidence as unknown.

4. Search community reports for each candidate and its relevant workflow terms:
   ```bash
   gh search issues --repo <owner>/<repo> "<skill-name>" --limit 100 --json number,title,state,url
   ```
   Repeat with aliases and symptom keywords when needed. Review matching titles first, then inspect relevant issues or pull requests individually:
   ```bash
   gh api repos/<owner>/<repo>/issues/<number> --jq '{title,state,body,html_url,comments_url}'
   ```
   Follow the comments URL for discussion when the body does not resolve the report. Separate confirmed defects, limitations, questions, and already-fixed reports; do not treat search counts or absent issues as proof of adoption, quality, or stability.

5. Compare the local and upstream evidence: declared status, distribution, maintenance activity, unresolved impact, relevance to the intended use, and local modifications. Mark stale local findings and inaccessible or ambiguous sources explicitly.

6. Synthesize one concise recommendation per skill: **adopt**, **adopt with constraints**, or **defer**. Cite the inspected documentation and issue URLs for each material claim, state residual uncertainty, and base adoption on fitness plus corroborating evidence rather than popularity or issue absence alone.
