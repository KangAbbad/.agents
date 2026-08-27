---
name: worktree-finish
description: Finish and clean up a git worktree safely — commit, merge to the source branch, verify commits are reachable, then remove the worktree + branch + prune, and sweep detached orphan worktrees. Use when the user says "finish worktree", "cleanup worktree", "selesai/beres worktree", "done with this worktree", or invokes /worktree-finish. Prevents the common failure where a detached worktree is left registered after work completes.
---

# Worktree Finish

Safe teardown of a git worktree after work is done. Gates every destructive step
so no committed work is lost and no other session's worktree is disturbed.

## When to Use

- User finished work in a `git worktree` and wants it merged back and cleaned up.
- User reports leftover / stale / detached worktrees from earlier sessions.
- Trigger phrases: "finish worktree", "cleanup worktree", "beres worktree",
  "selesaikan worktree", "/worktree-finish".

## Procedure

1. **Detect source branch.** The worktree branch's upstream/tracking branch. If
   none, nearest local branch via `git merge-base` (usually `main` / `staging`).
   Ambiguous → ask the user.

2. **Commit** all changes in the worktree. Nothing to commit → say so.

3. **Gate — clean tree.** `git status --porcelain` empty (modified *and*
   untracked). Not clean → STOP, report.

4. **Merge** worktree branch → source branch (fast-forward if possible, else
   merge commit). Conflict → STOP, report.

5. **Gate — reachable.** `git merge-base --is-ancestor <worktree-HEAD> <source-branch>`
   passes. Fails → STOP, delete nothing.

6. **Cleanup this worktree.** `git worktree remove <path>` →
   `git worktree prune` → `git branch -d <branch>` (skip branch delete if
   the worktree HEAD was detached).

7. **Sweep orphans.** From `git worktree list`, each *other* worktree:
   - detached HEAD **and** clean **and** reachable from any local branch →
     `git worktree remove` + `git worktree prune`.
   - on a named branch, or not reachable → report only, never remove. A
     branch-named worktree belongs to another active session.

8. **Confirm.** `git worktree list` shows only the main worktree (plus
   deliberately-kept branch worktrees). Summarize: removed what, commits landed
   where, left what and why.

## Hard Rules

- No cleanup before: working tree clean **and** merge succeeded **and** worktree
  commits reachable from the source branch.
- Orphan sweep touches **detached** worktrees only.
- On any STOP condition, report and wait — do not improvise a workaround.

## One-line form

> Deteksi source-branch (upstream, atau merge-base terdekat; ambigu → tanya).
> Commit, merge branch worktree ke source-branch, verifikasi commit reachable,
> lalu cleanup worktree + branch + prune. Sapu leftover: worktree lain yang
> detached + bersih + reachable → remove + prune; di branch bernama atau tak
> reachable → lapor. Stop kalau konflik atau working tree tak bersih.
