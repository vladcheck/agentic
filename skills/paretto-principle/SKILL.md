---
name: paretto-principle
description: Use when user asks "what are the core concepts of X", wants to learn a domain quickly, asks about essential/fundamental/high-leverage topics, mentions Pareto/80-20/essentials, or needs orientation in an unfamiliar subject area
license: MIT
user-invocable: true
metadata:
  deprecated: no
---

# Pareto Principle Research

## Overview
Research any domain to identify the ~20% of concepts that produce ~80% of understanding. Find what matters, skip what doesn't.

## When to Use

- "What are the core concepts of DevOps?"
- "What do I need to know about Kubernetes?"
- "Give me the essentials of SSR"
- "What should I learn first about X?"
- User mentions Pareto principle, 80/20, or "high-leverage concepts"
- User needs rapid orientation in an unfamiliar domain

**When NOT to use:**
- User asks for comprehensive coverage or deep-dive
- User needs implementation details (use domain-specific skill instead)
- User asks for a single fact or answer (just answer directly)

## Core Pattern

```
[ONE topic?] → Receive topic → Gather sources → Research broadly → Find connections → Distill → Present
```

### STEP 0 — Single-Subject Gate

Run this algorithm on the user's message. Mechanical where possible, minimal judgment where needed.

```
1. Split on these separators (first match wins, case-insensitive):
   - " vs " or " versus " → always split (these separate topics)
   - " or ", " and ", " also " → split, then filter below
   - top-level commas (not inside parentheses) → split, then filter below

2. Filter: discard pieces that are framing, not topics.
   A piece is NOT a topic if it is just a verb/instruction phrase
   like "research", "give me", "tell me about", "explain", "what about".
   Keep pieces that name a domain, technology, or subject.

3. Evaluate:
   IF remaining topic-like pieces > 1:
     STOP. List them. Ask user to pick one. Discard rest permanently.
   ELSE:
     PROCEED to step 1.
```

| Input | Split on | Raw pieces | After filter | Action |
|-------|----------|-----------|-------------|--------|
| "Kubernetes" | none | ["Kubernetes"] | 1 | Proceed |
| "DevOps and Kubernetes" | "and" | ["DevOps", "Kubernetes"] | 2 | Stop |
| "monitoring, logging, and tracing" | "and" | 3 pieces | 3 | Stop |
| "React vs Vue" | "vs" | ["React", "Vue"] | 2 | Stop |
| "Research core concepts of Flexbox" | none | 1 | 1 | Proceed |
| "Load the skill then follow it. Core concepts of Git." | none | 1 | 1 | Proceed |
| "JAMstack (JavaScript, APIs, Markup)" | none (parens) | 1 | 1 | Proceed |
| "SSR? Also, hydration." | "also" | ["SSR?", "hydration."] | 2 | Stop |
| "containerization and how to monitor it" | "and" | ["containerization", "how to monitor it"] | 2 | Stop |
| "frontend (React, Vue, testing, CSS)" | none (parens) | 1 | 1 | Proceed |

**If STOP: you may not continue.** The only permitted action is asking the user to pick one subject. Once they answer, permanently discard the others — they do not exist for the rest of this session. Do not mention them again.

1. **Receive topic** — Confirm scope with the user. If ambiguous ("Kubernetes" could mean ops, architecture, or dev), ask.
2. **Gather sources** — If user provides links, start there and crawl outward. If not, search with available tools (webfetch). If stuck, ask user for starting links.
3. **Research broadly** — Scan 3-5+ sources. Note which concepts recur. Read widely enough to distinguish universal concepts from source-specific opinions.
4. **Find connections** — Map how concepts relate. Which are prerequisites for others? Which do practitioners reference most? Don't overthink it — look for patterns.
5. **Distill** — Identify the 20% using these concrete methods. Do not rely on intuition. Use evidence from sources:

   **Source concurrence:** Tally every concept across all sources. Concepts appearing in 80%+ of sources are strong candidates. If 4 out of 5 independent guides mention it, it's not an opinion — it's consensus.

   **Dependency chain:** Map which concepts are prerequisites for others. If concept B only makes sense after understanding concept A, then A is more foundational. Rank concepts by how many others depend on them. A concept that 5 others build on belongs in the 20%; a concept that nothing else depends on does not.

   **First-hour placement:** What do sources teach first? What appears in "Getting Started" or "Fundamentals" sections? What do they relegate to "Advanced" or "Additional Topics"? Early placement across multiple sources is a strong signal.

   **Signal phrases in source text:** Look for phrases like "the most important thing is," "at its core," "everything else builds on," "the foundation of," "if you only learn one thing." These are authors explicitly signaling foundational concepts.

   **Removal test:** For each candidate concept, ask: if I deleted this from the explanation, would the rest still hold together? If removing it breaks understanding of multiple other concepts, it stays. If removal has no ripple effect, it goes.

   **Do not pad.** If the evidence points to 3 concepts, output 3. If it points to 11, output 11. The domain decides the number, not aesthetics.
6. **Present** — Output structured findings (see Output Format below).

## Output Format

Your response must have these sections, in this order:

### 1. Decision Table (required)

List every candidate concept found across sources. This table is mandatory — it shows your work before distillation.

| Concept | Sources (of N) | Confidence (%) | Essential? (Y/N) | Why essential / Why not |
|---------|----------------|----------------|-------------------|------------------------|

### 2. Core Concepts (essential only)

Define only the concepts marked Y. Each with:

```
**N. Concept Name** — intelligible definition in plain language. *Why it matters: one line.*
```

Skip the non-essential ones — they already had their say in the table.

### 3. How They Connect

Brief explanation of relationships between the essential concepts.

### 4. Starting Point

The single concept to begin with, if applicable.

### 5. Sources

A list of links used in research. Every concept in the table must be traceable to at least one of these.

---

There is no fixed number of essential concepts. A tight, well-scoped domain might need only 3 concepts to unlock 80% of understanding. A broad, interconnected domain might genuinely need 12. The count is an output of your research, not a target you aim for. Let the domain decide. If a concept doesn't unlock understanding of others, it doesn't belong — regardless of how many you have so far.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Presenting facts, not concepts | Concepts are mental models that transfer. Facts are one-off. |
| Skipping connections | Show how concepts relate, not just a flat list. |
| Researching too narrowly | Read enough sources to distinguish universal from specific. |
| Overthinking | Useful distillation beats academic perfection. Ship it. |

## Guardrails

1. **Don't fabricate.** Every concept must be traceable to research. If sources disagree, note the disagreement — don't resolve it by guessing.
2. **Don't overclaim.** Mark your confidence: distinguish "universally agreed" from "appears important across sources."
3. **Respect the other 80%.** Pareto analysis is for orientation. Explicitly acknowledge depth exists beyond these essentials.
4. **Ask when stuck.** If you can't identify the 20%, ask the user for starting links or clarification rather than guessing.
5. **Stay at concept level.** Don't drift into implementation tutorials or syntax. Mental models, not code.

## Red Flags — STOP Immediately

If you catch yourself doing any of these, you've already violated the skill:

- Skipping Step 0 entirely and jumping straight to research
- Finding a separator ("and", "or", "vs", "also", comma) but proceeding anyway
- Rationalizing that separators don't count because the topics are "related"
- Even mentioning the discarded subject after the user picked one
- Picking the first subject and silently ignoring the rest
- Writing code examples or CLI commands instead of concepts
- Answering from memory without doing any research

**These all mean: stop, course-correct, follow the skill.**
