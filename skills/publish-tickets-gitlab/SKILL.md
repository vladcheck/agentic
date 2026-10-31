---
name: publish-tickets-gitlab
description: Use when publishing an approved breakdown as dependent GitLab issues in an existing milestone. Not for unapproved drafts or non-GitLab trackers.
---

# Publish an approved breakdown to GitLab

Publish the approved numbered breakdown as GitLab issues while preserving its dependency graph and repository tracker conventions.

## Prerequisites

- Require explicit user approval of the numbered breakdown, including titles, dependencies, and deliverables. Never publish the quiz or discovery version.
- Follow the repository's tracker conventions for artifact language, title format, triage label, milestone, and assignees. Resolve the triage label from those conventions rather than assuming a label name.
- Confirm that the target milestone already exists. Do not create a milestone as part of publication.
- Resolve `<repo>`, the numeric REST `<project-id>`, the exact milestone title, and, when a description edit is requested, the global numeric REST `<milestone-id>`. Use `<issue-iid>` for each project-scoped issue IID.
- Record blocking edges in a textual `## Blocked by` section as a portable, reviewable dependency contract, even when native links are unavailable or unused.

## Issue body template

Use this template for every ticket. Follow the approved breakdown's artifact language and preserve its deliverables and acceptance criteria:

```markdown
## Parent
<milestone-url> — ticket <ticket-number>/<ticket-total>

## What to build
<deliverable and scope>

## Acceptance criteria
- [ ] <observable criterion>

## Blocked by
- #<issue-iid> (<blocker-title-without-ticket-number-prefix>)
```

For a ticket without blockers, replace the final list item with `None (can start immediately)`. For multiple blockers, list every real blocker issue IID.

## Procedure

1. **Publish every ticket in strict dependency order.** Create or repurpose blockers before dependents. For each ticket, write its body to `<ticket-body-file>` and substitute the real issue IID of every blocker. Then choose exactly one action:
   - Only when the user explicitly requested anchor repurposing for this ticket, update the existing anchor issue instead of creating a replacement:
     ```bash
     glab issue update <issue-iid> --repo <repo> --title "<ticket-number>: <title>" --description-file <ticket-body-file>
     ```
     Preserve its labels, milestone, and assignees unless the user requests changes. Record its existing issue IID in the ticket mapping.
   - Otherwise, create a new issue:
     ```bash
     glab issue create --repo <repo> \
       --title "<ticket-number>: <title>" \
       --description-file <ticket-body-file> \
       --label <triage-label> \
       --milestone "<milestone-title>" \
       --yes
     ```
     Capture the project-scoped issue IID from the create command's output immediately.

   Never infer an IID from the ticket number, assume contiguous IIDs, or process a dependent before all of its blockers have captured IIDs. Without an explicit repurposing request, leave the anchor issue unchanged. If creation fails, stop that dependency path and resolve the failure before continuing; do not invent a reference or blindly recreate earlier issues.

2. **Assign issues only when requested.** Run `glab issue update <issue-iid> --repo <repo> --assignee <user>` for each IID captured in the ticket mapping. Iterate over those captured IIDs, including a repurposed anchor issue; never generate a hard-coded numeric range with `seq` or assume the IIDs are contiguous.

3. **Edit the milestone description only when requested.** Use the global numeric REST `<milestone-id>`, not an IID, for both GET and PUT. Fetch the remote description as the source of truth immediately before editing, and pipe API JSON to `jq` because `glab api` has no `--jq` flag:
   ```bash
   glab api projects/<project-id>/milestones/<milestone-id> | jq -r '.description' > <milestone-description-file>
   glab api -X PUT projects/<project-id>/milestones/<milestone-id> -f description=@<milestone-description-file>
   ```
   Apply the requested edit to the freshly fetched content between these commands. If a planned edit was not pushed or the remote state is uncertain, fetch the description again and reapply the edit instead of overwriting it from a stale local file.

4. **Verify the complete published set.** List all issues and compare every captured IID with the result:
   ```bash
   glab issue list --repo <repo> --all --output json --jq '.[] | {iid, title, state, labels: [.labels[]], milestone: .milestone.title, assignees: [.assignees[].username]}'
   ```
   Confirm the expected count, titles, triage label, milestone, and requested assignee on every published issue. Spot-read at least two or three ticket body files to confirm that blocker references match the captured IID mapping. Use issue states from the listing when calculating the frontier; do not trust write-command output alone.

5. **Report the publication result.** Include a ticket-to-issue IID mapping table, the frontier of tickets with no open blockers, and the dependency graph in prose. Do not close or modify the anchor issue or any other parent or specification issue except for the explicitly requested anchor repurposing in Step 1.
