---
name: gitlab-mr-roundtrip
description: "GitLab MR comment round-trip: fetch threads, judge vs code, fix in atomic commits, push, reply+resolve, watch CI. Use on MR review feedback."
license: MIT
version: "1.0.0"
user-invocable: true
metadata:
  author: valdemarcheck
  tags: [gitlab, merge-request, review]
  deprecated: no
---

# GitLab MR comment round-trip

The whole loop behind "see comments on MR X, evaluate, fix, make atomic commits, push, resolve": fetch every thread → judge each against code → fix atomically → push → reply + resolve → CI green. One pass, nothing left open except threads the owner forbade touching.

## Hard rules

- **Merge only on explicit ask.** Without human approval the MR is not ready; approval by a person or automerge set by the owner is the only green light.
- **Your MRs only.** On an MR owned by someone else, nothing changes: no fixes pushed, no replies, no resolves — track it as a dependency and report.
- **Push with hooks or not at all.** `--no-verify` stays out; a failing hook is the next reviewer. Three failed push attempts → report exact hook output and stop.
- **A thread is *settled* only when your reply note and `resolved=true` are both visible in a re-fetched discussions response.** A plan, a chat message, or "will do" settles nothing.
- **Tokens stay invisible.** Never `--show-token`, never cat a token file, never paste credentials into notes, replies, or logs.

## Procedure

1. **Pin the MR.** Take project path and iid *from the URL*, never from memory — iids collide across projects. On `scm.x5.ru` always pass `--hostname scm.x5.ru` (glab defaults to gitlab.com and 401s there).
   ```bash
   glab mr view <iid> --repo <group>/<project> --hostname scm.x5.ru
   ```
   Record: title, source branch, target branch, head SHA. Done when you can recite head SHA and both branches.

   - ❌ Reusing a remembered numeric project id — the baobab notes you fetch may be x5m-web's (`34535` is x5m-web; baobab is `108880`). "The raw notes look like they belong to a different project" is this bug.
   - ✅ `x5m/frontend/baobab/-/merge_requests/20` → path `x5m%2Ffrontend%2Fbaobab`, derived fresh from the URL.

2. **Fetch every thread, full text.** System notes are noise; bot threads (e.g. `SRV.COPILOT-*`) are not — they judge just like human ones, even when the instance marks them non-blocking (`blocking_discussions_resolved=true` says nothing about whether the point is real).
   ```bash
   glab api --paginate --hostname scm.x5.ru \
     "projects/<group%2Fproject>/merge_requests/<iid>/discussions" > /tmp/mr<iid>-discussions.json
   ```
   Filter `.[] | select(.notes[0].system == false)`. List output truncates long bodies — get the full body via `.../notes/<note-id>` (or pipe JSON to `jq`; `glab api` has no `--jq`). Per thread record: `discussion_id`, author, `resolved`, existing replies, `position.new_path`/`new_line`. Done when every non-system thread sits in one table, verdict column empty.

   - ❌ Replying to the two-line truncated preview of a diff note and confidently fixing the wrong thing.
   - ✅ Pull the single note, read the reviewer's repro, then judge.

3. **Sync before you judge.** Verify against the live branch, not stale files:
   ```bash
   git -C <worktree> fetch origin && git -C <worktree> merge --ff-only origin/<branch>
   ```
   A colleague may have rebased the branch mid-review (new SHAs for the same commits). Fast-forward or rebase onto the new head before fixing; stop on a failed ff and report rather than force it.
   - ❌ Amending an annotation fix onto the SHA you fetched an hour ago — push is rejected, or worse, silently clobbers the rebase.
   - ✅ Notice remote head changed, rebase the fix onto `d30c699d`-equivalent, continue.

4. **Judge each comment against code.** Open the file at the comment's position and reproduce the claim. Reviewer repros against real behavior are usually right; check claims about upstream APIs against real API shapes and repo docs/specs, **not** fixtures or a worker's summary. Your own past verdicts and subagent assurances get the same treatment: re-verify with a fresh command before repeating.
   Verdict per thread: **fix** (point is real) / **explain** (already handled or claim is wrong — evidence required) / **question** (answer from code). Done when every thread has a verdict and the evidence line that earned it.
   - ❌ "Well, this is all planned" — three concrete bot points about duplication, missing `extends`, wrong variable, dismissed with one sentence, zero code read.
   - ✅ "Fair challenge — re-verify rather than rely on the earlier worker's claim": `git grep` on `main` for the deleted configs, zero references, file last-touched dates attached.

5. **Fix in atomic commits.** One commit per logical change, in the ticket's worktree. Stage deliberately per file; if a commit sweeps extra content, undo with `git reset --soft HEAD~1` and redo. Verify each commit carries exactly its files:
   ```bash
   git show --stat HEAD
   ```
   Message follows the repo convention, ticket key first, Russian fine: `TEXCOB-1040: fix(ci): ...`. After **all** fixes, run the repo's full check suite once (order-dependent fixture breakage shows up only there) — not just the touched test.
   - ❌ `git commit -a` — six review fixes plus half the scratch files in one blob; the reviewer can't map any comment to any diff.
   - ✅ Six commits, each named after the thread it closes; reply cites its SHA.

6. **Push.** Plain push first attempt. Branch already pushed → **new commit on top**; amend only local-only commits, and rewrite pushed history only on your unreviewed branch with `--force-with-lease`. Done when `git ls-remote origin <branch>` equals your local HEAD.
   - ❌ `git push --no-verify` because the hook complains about one file — the hook just filed a real review comment you skipped.
   - ✅ `git push origin <branch>`; rejected → ff-sync (step 3) → push again.

7. **Settle every thread: reply, then resolve.** A note alone does not close a thread. Write the body to a file first (quoting), then:
   ```bash
   glab api -X POST --hostname scm.x5.ru \
     "projects/<path>/merge_requests/<iid>/discussions/<discussion_id>/notes" -f "body=$(cat /tmp/reply.md)"
   glab api -X PUT --hostname scm.x5.ru \
     "projects/<path>/merge_requests/<iid>/discussions/<discussion_id>" -f resolved=true
   ```
   Reply in the reviewer's language, cite the fix SHA and the test that locks it, keep technical tokens verbatim, drop internal codenames. On a **merged** MR the threaded-reply endpoint 404s — post an MR-level note instead and say what it covers. Never blindly repeat a POST you already saw succeed. Done = re-run step 2 and confirm your `note_id` and `resolved=true` on every targeted thread, everything else unchanged.
   - ❌ Reporting "all comments addressed" while five threads still show `resolved=false` and the merge gate sits on `blocking_discussions_resolved`.
   - ✅ Five replies with `reply_note_id`s, five PUTs, re-fetch shows the table closed.

8. **Watch CI to a terminal state.** Poll the head pipeline on the fix SHA:
   ```bash
   glab mr view <iid> --repo <path> --hostname scm.x5.ru --output json | jq -r '.head_pipeline.status'
   ```
   Right after a push the status reads transient (`checking`/`pending` even for a healthy pipeline) — re-poll, report only the final verdict with the SHA it belongs to. Some projects have no MR pipeline at all (boilerplate repos: CI lives in the paired application MR) — confirm from the merge-request fields, then report that, don't invent a green check.
   - ❌ "Pipeline failed" from one snapshot taken mid-check, three minutes before it goes green.
   - ✅ `success` on `4cf0174`, reported with the SHA.

## Failure triage

Consult on the first error; each signature has one cause and one action.

| Signature | Cause | Action |
| --- | --- | --- |
| DNS/timeout on `scm.x5.ru` | VPN off | Retry once, then tell the user to turn the VPN on; never loop the same call |
| `401` mentioning `gitlab.com` | Wrong host — glab defaults there | Add `--hostname scm.x5.ru`; check `glab auth status --hostname scm.x5.ru`; token at `~/.secrets/gitlab-token` |
| GETs 200, POST/PUT `403` | `read_api` token can't write | Error body names the missing permission ("Merge Request: Create") — get an `api`-scope token; a `PRIVATE-TOKEN` header may work where `Authorization: Bearer` 403s |
| `404` on notes/discussions for an iid that exists in the UI | Wrong numeric project id (iid collision) | Re-derive path from the MR URL (step 1) |
| Push rejected / non-fast-forward | Remote moved (colleague rebase) | Fetch, rebase onto new head, `--force-with-lease` (own branch only) |
| `404` posting a threaded reply | MR merged; threads are sealed | MR-level note, state what it answers |
| `assignee_ids` won't clear | Form encoding ignored | `PUT` with JSON body `{"assignee_ids": []}` + `Content-Type: application/json` |

## Subagents

Parallel fixers are fine; uncoordinated closers are not.

- **One thread-owner per MR.** The implementer of the code answers comments about it. Two agents replying to one thread produces two half-answers.
- Give the dispatch contract: *every assigned thread ends settled (reply+resolve) or with an explain-verdict backed by code evidence*, in the reviewer's language, citing SHAs. Non-blocking bot threads are in scope.
- Demand a structured receipt so step 7's Done check is mechanical: per thread `{verdict, commit, discussion_id, reply_note_id, resolved}` + `{push_attempts, hook_output, head_sha}`.
- Fixers working in side branches land via cherry-pick onto the MR branch after the full suite runs — never "join them later" by hand.
- ❌ "Dispatch agents to handle comments" with no closing criterion → agent reports "all planned", zero replies, user finds it two hours later.
- ✅ Agent that under-delivered gets reopened with: explain the verdict to me, or fix, commit atomically, reply, resolve. Receipt verified against a re-fetch.
