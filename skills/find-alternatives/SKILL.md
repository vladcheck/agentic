---
name: find-alternatives
description: Use this skill when the user asks to find an alternative to a tool or an equivalent tool but with some key difference provided by user.
license: MIT
user-invocable: true
metadata:
  deprecated: no
---

## Goal

Help the user discover, evaluate, and compare tools or products. Provide honest, useful recommendations with clear reasoning.

## When to use

Use this skill when the user asks:
- "find me a popular and free tool for X that does Y"
- "find me better alternatives for X"
- "what are the best alternatives to X?"
- "compare X with other tools"
- "is there a free alternative to X?"

## Process

### 1. Clarify the request

If unclear, ask:
- What is the primary use case?
- What is your budget? (free / paid / enterprise)
- What platform or ecosystem? (web, macOS, CLI, self-hosted, etc.)
- What are dealbreakers or must-have features?
- Who will use it? (individual, team, size)

### 2. Find candidates

Use available tools to research:
- Web search for current top recommendations
- GitHub search for open-source options
- Package managers (npm, pypi, homebrew, etc.) for popularity signals
- Review sites (G2, Capterra, TrustRadius, Product Hunt) when relevant

### 3. Evaluate

For each candidate, gather:
- **Pricing**: free tier, paid plans, open-source
- **Key features**: how it solves the use case
- **Popularity**: install count, GitHub stars, community activity
- **Trade-offs**: strengths, limitations, best-fit audience
- **Ecosystem fit**: integrations, platform support

### 4. Present results

Structure the response:
1. **Quick recommendation**: top pick and why
2. **Comparison table**: name, price, key features, best for
3. **Detailed breakdown**: 2-5 top options with pros/cons
4. **Honest guidance**: who each option is best for
5. **Next steps**: how to try or decide

---

## Principles

- Be honest about weaknesses, not just strengths.
- Prefer options with strong community adoption unless the user asks for niche tools.
- Mention free/open-source options prominently when budget is a concern.
- Avoid affiliate bias; rank by fit, not by referral potential.
- Update recommendations if you know a tool is deprecated or no longer maintained.

---

## Output format

Use concise markdown. Include links when possible. Keep the focus on actionable comparison.

Example opening:

> For **X use case**, my top recommendation is **ToolName** because **reason**. If you need **specific requirement**, consider **AlternativeName** instead.
