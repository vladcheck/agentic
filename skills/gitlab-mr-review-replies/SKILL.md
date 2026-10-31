---
name: gitlab-mr-review-replies
description: Use when verifying GitLab MR feedback, posting review replies, and resolving threads. Not for MR creation, general reviews, or pipeline issues.
---

# GitLab MR Review Replies

Answer review feedback only after checking the latest code. Keep replies and resolution limited to the requested discussion threads.

## Procedure

1. **Establish the target and access.** Identify `<repo>`, `<project-id>`, `<mr-iid>`, `<worktree>`, and `<branch>`. Check authentication without revealing credentials:
   ```bash
   glab auth status --hostname <gitlab-host>
   ```
   Never use `--show-token`, print tokens, or include tokens in replies. GET requests may work with a `read_api` token, but POST notes and PUT `resolved=true` require an API token with the `api` scope and a sufficient project role. Treat 401 as an authentication problem; 403 can indicate insufficient token scope or project permissions.

2. **Update the worktree to the latest origin branch.** Verify against current code, never stale files:
   ```bash
   git -C <worktree> fetch origin
   git -C <worktree> merge --ff-only origin/<branch>
   ```
   Stop if the fast-forward update fails; do not inspect stale code or rewrite history to force the update.

3. **Collect every discussion thread.** Paginate the discussions API and save the response:
   ```bash
   glab api --paginate projects/<project-id>/merge_requests/<mr-iid>/discussions > /tmp/mr-discussions.json
   ```
   Inspect all notes in each thread. Skip system notes using the note's `system` field, including commit additions, approvals, assignments, and thread-resolution events. For each reviewer thread, record the `<discussion-id>`, existing `<note-id>` values, author, body, resolved state, and file/line position from `notes[0].position.new_path` and `notes[0].position.new_line` when present.

4. **Verify each comment directly against the code.** Read the relevant files and tests in the updated worktree. Map each comment to the required behavior and check the implementation itself; worker claims, summaries, and review replies are not proof.

5. **Classify each non-system thread.** Write replies in English by default, or in the repository language only when the user explicitly requests it. Keep identifiers and commands unchanged.
   - **Implemented:** The current code satisfies the feedback. Prepare a short reply stating what changed and include the commit SHA when known.
   - **Question:** If the current change answers the question, prepare a brief explanation grounded in the code. Otherwise, do not reply or resolve unless the user explicitly instructs otherwise.
   - **Not implemented:** Do not reply and do not resolve unless the user explicitly instructs otherwise.
   - **Joke or off-topic:** Reply briefly, professionally, and in kind; leave the thread unresolved by default.

6. **Post notes and resolve only the authorized threads.** By default, post substantive replies only for implemented feedback and questions answered by the current change. Resolve a thread only when the user requested resolution and the feedback is implemented or the question is answered by the change. Do not broaden the requested scope.
   Write each reply to a file, then post it to its `<discussion-id>`:
   ```bash
   glab api projects/<project-id>/merge_requests/<mr-iid>/discussions/<discussion-id>/notes -X POST -f body="$(cat /tmp/mr-review-reply.md)"
   ```
   Resolve only an authorized thread:
   ```bash
   glab api projects/<project-id>/merge_requests/<mr-iid>/discussions/<discussion-id> -X PUT -f resolved=true
   ```

7. **Re-fetch and verify the final state.** Repeat the paginated discussions request into a new file. Confirm that every targeted thread contains the new reply, record each returned `<note-id>`, and verify `resolved=true` for every thread authorized for resolution. Confirm that all other threads retain their prior state. If a mutation or verification fails, report the exact persisted state and do not blindly repeat a successful POST.

## GitLab and CI Gotchas

- The CI Lint API at `/ci/lint?include_jobs=true` can report a job as included while the real pipeline excludes it. Jobs without explicit `rules` can be invisible in merge request pipelines. Verify against the real pipeline's `/jobs` API rather than trusting CI Lint alone.
- Job-level `when: manual` with `allow_failure: true` creates a non-blocking, press-to-run job. Put it in its own stage after the gating stage when an explicit dependency is required.
- In glab 1.126.0, `glab api` has no `--jq` flag. Pipe API JSON to `jq` when filtering; never use `glab api --jq`.
