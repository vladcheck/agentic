---
name: git-rebuild-branch
description: Use when rebuilding a git branch by cherry-picking only the commits unique to it onto another branch — e.g. re-creating a feature branch on top of main while dropping commits inherited from an intermediate branch. Triggers include "cherry-pick commits that belong to X but not Y", "rebase branch onto main keeping only its own commits", "reconstruct branch without a parent branch's commits".
license: MIT
user-invocable: true
metadata:
  deprecated: no
---

# git-rebuild-branch

Rebuild a branch by cherry-picking only the commits that are unique to it onto a target branch, then swap the rebuilt branch back into place under the original name.

## Inputs (collect all before acting)

1. **Target branch** — the branch to cherry-pick onto (e.g. `main`).
2. **Source branch** — the branch whose commits you want (e.g. `B`).
3. **Ignore branch** *(optional)* — a branch whose commits must be excluded (e.g. `A`). Given `main -> A -> B`, ignoring `A` keeps commits in `B` that are NOT in `A`.

If any of the first two is missing, ask the user before proceeding.

## Commit range

- Ignore branch given: `<ignore>..<source>` (commits in source, not in ignore).
- No ignore branch: `<target>..<source>` (commits in source, not in target).

`git cherry-pick <range>` applies these oldest-first automatically.

## Workflow

1. **Preview** the commits so the user can confirm they are correct:
   `git log --oneline <range>`
2. **Build the command list** using a temporary branch name (`<source>-rebuilt`). Every command below is mandatory and part of the sequence — the safety anchor and verifications are not optional extras:
   ```
   git log --oneline <range>
   git rev-parse <source>
   git checkout <target>
   git rev-parse --abbrev-ref HEAD
   git checkout -b <source>-rebuilt
   git cherry-pick <range>
   git status
   git log --oneline <target>..HEAD
   git branch -D <source>
   git branch --list <source>
   git branch -m <source>-rebuilt <source>
   git log --oneline <target>..<source>
   ```
3. **Get approval.** Present exactly:
   > Here is what I'm going to do:
   > `<commands>`
   > Do you approve this?

   Do NOT run anything until the user approves.
4. **Execute one command at a time.** Run each command as a separate invocation. Never chain with `&&`, `;`, or `||`. After each command, confirm its result matches the table below before running the next.
5. **Save the output of `git rev-parse <source>`** (step 2, second command) — this SHA is the backtracking escape hatch.
6. If `cherry-pick` hits a conflict, stop and surface it — resolve, then `git cherry-pick --continue`. Never `--skip` or `--abort` without asking.

## Expected result per command

| Command | Expect |
| --- | --- |
| `git rev-parse <source>` | a SHA — save it (backtracking anchor) |
| `git checkout <target>` → next `git rev-parse --abbrev-ref HEAD` | `<target>` |
| `git checkout -b <source>-rebuilt` | branch created; HEAD is `<source>-rebuilt` |
| `git cherry-pick <range>` → `git status` + `git log --oneline <target>..HEAD` | clean tree; same commits (minus SHAs) as the preview |
| `git branch -D <source>` → `git branch --list <source>` | empty |
| `git branch -m <source>-rebuilt <source>` | `<source>` exists, `<source>-rebuilt` gone |
| Final `git log --oneline <target>..<source>` | exactly the intended commits |

## Backtracking

The saved SHA from `git rev-parse <source>` lets you undo everything — the original commits are not garbage-collected while that SHA is known.

| Situation | Recovery |
| --- | --- |
| Cherry-pick conflict you don't want to resolve | `git cherry-pick --abort`, then `git checkout <target>` and delete the temp branch — original `<source>` is untouched. |
| Rebuilt branch looks wrong, source NOT yet deleted | `git checkout <source>`; `git branch -D <source>-rebuilt`. Nothing lost. |
| Already deleted/renamed and result is wrong | `git branch -f <source> <saved-SHA>` to restore the original, then remove the bad branch. |
| Lost the SHA too | `git reflog` — find the pre-rebuild `<source>` entry and `git branch -f <source> <reflog-SHA>`. |

Prefer the earliest recovery point: the further you get, the more you rely on the saved SHA / reflog.

## Common mistakes

- **Wrong range direction.** `A..B` = commits reachable from B but not A. Reversing it drops everything you wanted.
- **Chaining commands.** The user requires one command per invocation; a chained failure leaves the repo half-migrated.
- **Deleting the source branch before the rebuild succeeds.** Only run `git branch -D <source>` after all cherry-picks land.
- **Skipping the preview.** Always show `git log --oneline <range>` so the commit selection is confirmed before any destructive step.
