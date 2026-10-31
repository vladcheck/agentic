---
name: opsx-spec-validate
description: Use when you need to validate if your openspec artifacts violate any specs defined by openspec.
license: MIT
user-invocable: true
metadata:
  deprecated: no
tools: openspec
---

## Files to check

- proposal.md - initial artifact
- design.md - builds on proposal.md
- specs/* - builds on proposal.md
- tasks.md - builds on every spec above

### Rules for proposal.md
- Has sections: Why, What Changes, Capabilities (New + Modified), Impact
- Capabilities use kebab-case names
- Each new capability listed in New Capabilities has a corresponding spec file in specs/
- Each modified capability listed in Modified Capabilities has a corresponding delta spec file in specs/
- Breaking changes marked with **BREAKING**
- Concise (1-2 pages)

### Rules for design.md
- Has sections: Context, Goals/Non-Goals, Decisions, Risks/Trade-offs, Migration Plan, Open Questions
- Each Decision explains rationale (why X over Y?)
- Risks have mitigation
- Migration Plan has concrete steps
- No empty sections

### Rules for spec files (all)
- Scenario headers: exactly 4 hashtags (#### Scenario: ...)
- Requirement headers: exactly 3 hashtags (### Requirement: ...)
- Every requirement has at least one scenario
- SHALL/MUST for normative statements
- Delta section headers valid: only ## ADDED, ## MODIFIED, ## REMOVED, ## RENAMED
- No empty delta sections

### Cross-artifact consistency
- Every new capability in proposal.md → has spec file in specs/
- Every modified capability in proposal.md → has delta spec file in specs/
- No extra spec files without corresponding proposal entries
- No extra proposal entries without corresponding spec files

## Algorithm

1. Ask a subagent to verify if your specs violate openspec conventions
2. Fix violations

