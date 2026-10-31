---
name: gitlab-eas-tickets-publish
description: Use when publishing an approved EAS spec and dependency-ordered tickets to GitLab, optionally with an MR. Not for standalone specs or tickets.
---

# Publish an EAS specification, tickets, and implementation MR

Coordinate the complete GitLab publishing workflow while keeping the detailed spec and ticket procedures in their dedicated skills.

## Select the workflow

- For a spec-only request, use **publish-spec-gitlab-milestone** and stop.
- For a ticket-only request, use **publish-tickets-gitlab** and stop.
- For the combined workflow, use **publish-spec-gitlab-milestone** first, then **publish-tickets-gitlab**, and continue here only if the user requested an implementation MR.

## Prerequisites

- Confirm that the user approved the exact specification and dependency-ordered ticket breakdown. Never publish an unapproved draft or quiz version.
- Resolve `<repo>`, `<project-id>`, the milestone title, and the triage label from the target project. Use `<issue-iid>` and `<mr-iid>` for project-scoped IIDs. Use `<milestone-id>` only for the global numeric REST milestone `id`, never for an IID.
- Match the language of the approved artifacts and project conventions. Keep code identifiers and branch names in the repository's established language.
- Create or update GitLab objects only within the scope the user requested.

## Publish and verify

1. Publish the approved specification through **publish-spec-gitlab-milestone**. Retain the milestone title, `<milestone-id>`, and anchor `<issue-iid>` for the remaining steps.
2. Publish the approved breakdown through **publish-tickets-gitlab**. That workflow creates issues strictly in dependency order so every blocker has a real `<issue-iid>` before its dependents are created.
3. Retain the ticket-to-issue mapping, verification evidence, and unblocked frontier returned by the ticket workflow. Do not rerun `glab issue create` or infer IIDs from ticket numbers. Treat `## Blocked by` references such as `- #<issue-iid> (<title>)` as the portable dependency contract recorded by that workflow.
4. Stop here when no implementation MR was requested. Otherwise, continue with the approved implementation scope.

## Open an implementation MR

1. Create a dedicated branch from fresh `origin/main` in a separate worktree, leaving any occupied or dirty primary clone untouched:

   ```bash
   git fetch origin main
   git worktree add -b <branch> <worktree> origin/main
   ```

2. Work through `git -C <worktree> ...` rather than changing the caller's directory. Implement only the approved issue scope.
3. Run the project's local checks before pushing. Record the exact commands and their results. Resolve failures before pushing; never claim unrun or failed checks as passing.
4. Push `<branch>` and write the MR description to a file. Include a concise implementation summary, `Closes #<issue-iid>`, and the exact verification commands and results.
5. Create the MR without auto-merge:

   ```bash
   glab mr create --repo <repo> --source-branch <branch> --target-branch main --title "<mr-title>" --description-file <file> --yes
   ```

6. Return the `<mr-iid>` and verification summary. Leave the MR open for review. Never enable auto-merge. Never merge the MR in this workflow; a merge requires separate, explicit user approval.

## GitLab CLI safeguards

- Use `glab issue list --output json --jq ...`, not `--json`.
- Use `--description-file <file>` for issue and MR bodies to preserve Markdown and avoid shell-quoting errors. Use `--yes` for noninteractive creation.
- `glab api` has no `--jq` flag in the supported CLI contract; pipe API JSON to `jq` when filtering is needed.
- Address milestone REST reads and updates as `projects/<project-id>/milestones/<milestone-id>` with the global numeric `<milestone-id>`.
- Before retrying a write after an interrupted or unclear response, query GitLab for the current state and reuse any object already created. Treat remote state as the source of truth and avoid duplicate issues, milestones, or MRs.
