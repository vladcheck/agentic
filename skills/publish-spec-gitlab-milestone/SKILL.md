---
name: publish-spec-gitlab-milestone
description: Use when publishing an approved spec as a GitLab milestone with a ready-for-agent anchor issue. Not for ticket breakdowns or milestone admin.
---

# Publish a spec as a GitLab milestone

Publish the complete spec in the milestone description. Use the anchor issue only as a pointer to that description and as the carrier of the `ready-for-agent` triage label.

## Procedure

1. Obtain `<repo>`, the numeric REST `<project-id>`, `<milestone-title>`, the approved spec, and, when converting an existing plain issue, its `<issue-iid>`. Use `<milestone-id>` only for the global numeric REST milestone `id`, never for a project-scoped IID.

2. Write the complete spec to a temporary `<spec-file>`. Preserve the approved content and use this English section template:

   ```markdown
   # Problem Statement

   # Solution

   # User Stories

   # Implementation Decisions

   # Testing Decisions

   # Out of Scope

   # Further Notes
   ```

3. Create the milestone and save the complete create response:

   ```bash
   glab api projects/<project-id>/milestones -X POST \
     -f title="<milestone-title>" > <milestone-response>
   ```

   Extract `<milestone-id>` and `<milestone-url>` directly from that response:

   ```bash
   jq -r '.id' <milestone-response>
   jq -r '.web_url' <milestone-response>
   ```

   Require a nonempty numeric `id` and a nonempty `web_url` before continuing. Do not recover the ID by listing milestones or indexing an array.

4. Put the complete spec into the milestone description, addressing the milestone by its global REST ID:

   ```bash
   glab api projects/<project-id>/milestones/<milestone-id> -X PUT \
     -f description=@<spec-file>
   ```

5. Ensure the `ready-for-agent` triage label exists:

   ```bash
   glab api projects/<project-id>/labels/ready-for-agent
   ```

   If the label lookup returns 404, create the label:

   ```bash
   glab api projects/<project-id>/labels -X POST \
     -f name="ready-for-agent" \
     -f color="#95a5a6"
   ```

   Stop on lookup errors other than 404.

6. Write a temporary `<pointer-file>` containing only the anchor text and milestone URL:

   ```markdown
   The complete specification is maintained in the milestone description:

   <milestone-url>

   Use the milestone description as the single source of truth. This anchor issue tracks implementation readiness.
   ```

   Keep the full spec out of the anchor issue.

7. Create or reuse the anchor issue:
   - For a new anchor issue, run:

     ```bash
     glab issue create --repo <repo> \
       --title "Spec: <milestone-title>" \
       --description-file <pointer-file> \
       --milestone "<milestone-title>" \
       --label ready-for-agent \
       --yes
     ```

     Capture the created issue's project-scoped `<issue-iid>`.
   - To convert an existing plain issue, replace its body with the pointer and attach the milestone and triage label:

     ```bash
     glab issue update <issue-iid> --repo <repo> \
       --description-file <pointer-file> \
       --milestone "<milestone-title>" \
       --label ready-for-agent
     ```

8. Verify the persisted milestone description and issue metadata before reporting completion:

   ```bash
   glab api projects/<project-id>/milestones/<milestone-id> > <milestone-verification>
   jq -e --rawfile spec <spec-file> \
     '.description == $spec' <milestone-verification>
   glab issue view <issue-iid> --repo <repo>
   ```

   Confirm that the milestone response has the expected `<milestone-id>` and title and that the comparison returns `true`. Confirm that the anchor issue has the `ready-for-agent` triage label, belongs to `<milestone-title>`, and contains only the pointer to the milestone description.
