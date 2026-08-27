---
description: Finish and clean up the current git worktree (commit, merge to source branch, verify, remove worktree + branch + prune) and sweep detached orphan worktrees.
---

Finish the current git worktree safely.

## Steps

1. **Detect source branch.** Use the worktree branch's upstream/tracking branch.
   If none, pick the nearest local branch via `git merge-base` (usually `main` or
   `staging`). If ambiguous, ask the user before proceeding.

2. **Commit.** Commit all changes in this worktree. If nothing to commit, say so.

3. **Gate — clean tree.** `git status --porcelain` must be empty (no modified,
   no untracked). If not clean: STOP and report.

4. **Merge.** Merge the worktree branch into the source branch (fast-forward if
   possible, otherwise a merge commit). If conflict: STOP and report.

5. **Gate — reachable.** `git merge-base --is-ancestor <worktree-HEAD> <source-branch>`
   must pass. If it fails: STOP, do not delete anything.

6. **Cleanup this worktree.** `git worktree remove <path>` → `git worktree prune`
   → `git branch -d <branch>` (skip branch delete if the worktree was detached).

7. **Sweep orphans.** From `git worktree list`, for every *other* worktree:
   - detached HEAD **and** clean working tree **and** HEAD reachable from any
     local branch → `git worktree remove` + `git worktree prune`.
   - on a named branch, or not reachable → report only, do NOT remove
     (it belongs to another active session).

8. **Confirm.** Run `git worktree list` — only the main worktree should remain
   (plus any branch-named worktrees intentionally left). Report a short summary:
   what was removed, which commits landed on which branch, what was left and why.

## Hard rules

- Never run cleanup before: working tree clean AND merge succeeded AND worktree
  commits reachable from the source branch.
- The orphan sweep only ever touches **detached** worktrees.
