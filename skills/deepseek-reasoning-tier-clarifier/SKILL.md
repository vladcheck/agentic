---
name: deepseek-reasoning-tier-clarifier
description: Use when user asks about reasoning effort or when task scope is ambiguous.Analyze task complexity and recommend DeepSeek reasoning tier (low/medium/high/max).
license: MIT
compatibility: opencode, deepseek-v4
version: "1.0.0"
metadata:
  author: valdemar_check <valdemarcheck@yandex.ru>
  audience: developers
  domain: ai-agent-orchestration
  triggers: reasoning, thinking-mode, task-complexity, model-selection
  tags: [utility, guide]
  deprecated: no
user-invocable: true
---

# DeepSeek Reasoning Tier Classifier

## Purpose

- Analyze the current task prompt for complexity signals
- Check conversation history for related context or prior decisions
- Optionally inspect referenced codebase files to assess technical scope
- Map task characteristics to a DeepSeek reasoning tier recommendation
- Present the recommendation with clear rationale and request confirmation if overriding user preference

Use this skill when:
- User explicitly asks which reasoning tier to use
- Task prompt is ambiguous about required reasoning depth
- You detect a mismatch between user-specified tier and task complexity
- Before executing a high-cost `max` tier request on a potentially trivial task

Do NOT use me for:
- Simple factual questions with no reasoning component
- Tasks where user has already confirmed their tier choice
- Non-DeepSeek model sessions

## Requirements

- Any DeepSeek model that supports reasoning tiers is available;

## Instructions

### Step 1: Gather context
1. Read the latest user message and identify the core task
2. Scan the last 5-10 messages for:
   - Prior tier selections or preferences
   - Related code snippets or file references
   - User statements about speed vs accuracy priorities
3. If the task references specific files, read them to assess:
   - Codebase size and structure
   - Presence of complex logic, async operations, or external integrations
   - Error handling requirements

### Step 2: Classify task signals
Use these heuristics to assign a preliminary tier:

| Signal | Suggested tier |
|--------|---------------|
| Keywords: `bash`, `grep`, `one-liner`, `list`, `simple`, `quick` | `low` |
| Keywords: `debug`, `refactor`, `optimize`, `multi-step`, `plan` | `medium` |
| Keywords: `architecture`, `race condition`, `algorithm`, `novel`, `research` | `high` |
| Keywords: `prove`, `verify`, `formal`, `novel algorithm`, `high-stakes` | `max` |
| File references to complex systems (distributed, async, ML pipelines) | +1 tier |
| User explicitly states `use max` or `speed is critical` | Respect user override |

### Step 3: Validate against constraints
Before finalizing recommendation:
- If task involves file I/O or external APIs, ensure tier supports tool use
- If user has low latency requirements, cap at `medium` unless they confirm otherwise
- If using `V4 Flash`, note that `high`/`max` tiers hit ~12k thinking token ceiling

## Output format

Present your recommendation using this structure:

```
## Reasoning tier recommendation

**Task**: [brief summary]

**Analysis**:
- Complexity signals: [list key observations]
- Context notes: [relevant history or file insights]
- Constraints: [latency, model, or user prefs]

**Recommended tier**: `low` | `medium` | `high` | `max`

**Rationale**: [1-2 sentences explaining the mapping]

**Next step**:
- If tier matches user request: "Proceed with `{{tier}}` reasoning."
- If tier differs: "Your request specified `{{user_tier}}`, but analysis suggests `{{recommended_tier}}`. Switch? (y/n)"
```

## Limitations

- DeepSeek API does not support dynamic tier switching mid-generation. Any change requires a new API call.
- `V4 Flash` caps effective thinking tokens at ~12k. Route `high`/`max` tasks to `V4 Pro` if available.
- User may have cost sensitivity. Always surface the cost/latency tradeoff when recommending a higher tier.
- If conversation context is sparse, default to `medium` and ask clarifying questions rather than guessing `max`.

## Examples

User: "[calls the skill] Write a small bash script to search for files with grep where 'import' appears"

1. Detect keywords: `bash`, `grep`, `small` → signals `low`
2. Check history: no prior tier preference found
3. No file references to inspect
4. User did not specify a tier, so no override needed
5. Output recommendation for `low` with rationale

User: "[calls the skill] Debug this race condition in our WebSocket handler" + attaches `websocket.ts`

1. Detect keyword: `debug`, `race condition` → signals `high`
2. Read `websocket.ts`: confirms async state management, multiple event listeners
3. Complexity elevated by file analysis → confirm `high` or consider `max`
4. Output recommendation for `high` with note that `max` is available if deeper verification is needed

## Validation loop

After providing a recommendation:
1. Wait for user confirmation if tier differs from their stated preference
2. If user accepts, proceed with the task using the agreed tier
3. If user rejects, note their choice and proceed with their selected tier
4. Log the decision in conversation for future reference (helps with pattern learning)

## Troubleshooting
