---
name: modernize
description: >-
  For getting old iota-spear PRs and branches up to date with the latest
  main by rebasing, with every commit green. Commits, never pushes. Use when
  asked to rebase or update a PR or branch, or "modernize". full-stop uses it
  to finish on the latest main.
---

# Modernize

For iota-spear (`Lamp-Post-Group/iota-spear`). Modernize the idea, not just the
branch. A clean merge or rebase does not prove that the PR still belongs in the product.

The rules this skill follows live in the repo's `AGENTS.md` and
`docs/LANDING.md`. If they changed since this skill was written, they win.

## Guardrails

- `git fetch origin` before anything, then read `git log -1 origin/main`. Bare
  `main` only moves on fetch, so it can look current while stale.
- Confirm you're in a worktree. The branch is the user's own and already has
  one. Check that the current directory is a linked worktree, not the root
  checkout: `git rev-parse --git-dir` and `git rev-parse --git-common-dir`
  differ in a linked worktree. Work there, on its branch. Do not create
  another worktree. If you are in the root checkout, ask the user which
  worktree to use.
- Never stash or discard uncommitted changes in it. If there are any, ask the
  user before rebasing. (`full-stop` commits its work before it calls you.)
- Modernizing authorizes rebasing the branch onto the latest `origin/main` and
  committing in it. It does not authorize pushing, approving, merging,
  closing, commenting, changing the base branch, or expanding scope. The user
  pushes.
- **Rebase, never merge.** Bring the branch current with `pnpm land:rebase`,
  never by merging `main` into it. This holds for a branch already on `origin`
  too: CI prices a force-push over a rebase by the branch's own commits
  (`scripts/check-bucket-separation.ts`, "FORCE-PUSH FALLBACK"), so it does not
  go red for being rewritten.
- **Hands off a PR someone else holds.** If it carries `wip: <name>` for
  someone other than the user, `walk-in-progress`, `in-code-review`, or `held`,
  do not rewrite it. Report and ask.

## Establish the PR's intent

For a local branch with no PR yet (for example one `full-stop` is building),
use the branch's commits and the task's goal in place of the PR's history,
and skip the GitHub fields.

Resolve the PR and record its URL, state, base, head branch, current head SHA,
author, `status:` label, checks, and review state. Read the head's red CI jobs:
they are work the lane still carries. Read the title, body, linked issues,
commits, review threads, and complete diff. State the smallest coherent goal
the PR was trying to accomplish before judging its implementation.

Record the merge base with `origin/main` (or the PR's actual base, for a
stacked PR), so the original PR and later `main` changes can be compared
separately.

## Check what changed around it

Inspect commits on `main` since the PR diverged. Search merged and open PRs
when titles, code paths, or behavior suggest a replacement or competing
implementation. Read `CHANGELOG.md`, `changelog.d/`, and `docs/DEFERRED.md` for
landed work and open bets in the same area.

Check by content, not ancestry: cherry-picks and PR replays rewrite hashes, so
`origin/main..<branch>` can list commits that already landed. `git cherry`
marks a commit that already has an equivalent upstream with `-`, and it fails
open, so confirm with `git diff origin/main <head> -- <touched paths>`.

Look beyond overlapping lines:

- Has the same user-facing problem already been solved?
- Did APIs, types, schemas, manifests, or architectural boundaries change? Did
  a doctrine or ADR land that this PR now breaks?
- Do both sides now claim ownership of the same behavior or invariant?
- Was the code being changed deleted, moved, or intentionally simplified?
- Would landing the PR restore an old behavior or create two ways to do the
  same thing?
- Is the original benefit still large enough to justify the maintenance cost?

Do not describe a PR as current merely because Git reports no conflicts.

## Choose a disposition before rebasing

Use one of these recommendations:

- **MODERNIZE**: the goal is still useful and the implementation remains the
  right shape. Bring it current (below), test, and update the branch.
- **REWORK**: the goal is still useful, but bringing it current as-is would preserve
  the wrong implementation. Explain the smallest credible rewrite and stop
  before rebasing unless the user also asked for that rework.
- **CLOSE**: the PR is superseded, obsolete, directionally wrong, or no longer
  worth its cost. Cite concrete evidence and stop. Do not spend time updating an
  obsolete branch, and do not close it without explicit permission.

Be willing to recommend `CLOSE`. Age alone is not evidence, and sunk effort is
not a reason to keep a PR alive.

## Bring it current without losing intent

For a `MODERNIZE` disposition, first check whether the branch is on
`origin` with `git ls-remote --heads origin <branch>`, and record its remote
head SHA if it is. The user needs it to push safely.

1. **Decide whether it needs a new base.** A move of `main` alone is not a
   reason. Run the merge check on its own, not the whole preflight:
   `node --experimental-strip-types --no-warnings=ExperimentalWarning scripts/check-github-merge.ts`
   (add `--base <branch>` for a stacked branch). If it merges cleanly and
   nobody asked, skip to step 4: the branch needs checks, not a new base. A
   request from the user, or a call from `full-stop`, always rebases onto the
   latest `origin/main`.
2. **Rebase.** Run `pnpm land:rebase` (or `pnpm land:rebase -- <base>` for a
   stacked branch). It rebuilds the generated ledger at each commit and stops
   on real conflicts. Resolve each by the branch's current intent and
   `main`'s newer invariants, stage with `git add --resolved`, review the
   staged diff, and `git rebase --continue`. If a conflict reveals a product
   or architecture disagreement, `git rebase --abort` and reconsider the
   disposition instead of guessing.
3. **Regenerate.** Run `pnpm regen` and amend each regenerated file into the
   commit that owns it. Never leave a red commit for a separate regen commit.
   Compare the old and new series with `git range-diff` and inspect the final
   diff against `origin/main`: no intended behavior disappeared and no
   already-landed behavior came back.
4. **Make every commit green.** Run `scripts/check-commit` from this skill's
   directory (use its absolute path) on every commit of the branch:

   ```bash
   git rebase --reschedule-failed-exec --exec "bash <abs path>/check-commit" origin/main
   ```

   It typechecks and runs the unit tests for each area the commit touches
   (root, `apps/web`, `apps/qa`). When a commit fails, the rebase stops on it.
   Fix it in that commit, `git commit --amend`, and `git rebase --continue`.
   Never fix a red commit with a later commit. Use `<base>` in place of
   `origin/main` for a stacked branch. Record each commit's result. Do not
   leave the worktree mid-rebase: finish or `git rebase --abort` before you
   report.
5. Run `pnpm preflight` once and fix every red. If the branch touches the
   published package, also run `pnpm build && pnpm build:smoke`. Tell real
   failures apart from runner or infrastructure flakes, and from reds already
   on `main`. Preflight runs on the tip only. It covers the database suites
   and GitHub's merge, which step 4 does not.
6. **Do not push.** Hand the user the command. For a branch already on
   `origin`, pin the lease to the remote head you recorded, so the push fails
   instead of overwriting anything pushed since:
   `git push --force-with-lease=<branch>:<remote head sha> origin <branch>`.
   For a branch never pushed: `git push -u origin <branch>`.

Do not merge the PR.

## Report

Lead with `MODERNIZE`, `REWORK`, or `CLOSE` and the reason. Then summarize:

- meaningful changes on `main`;
- conceptual conflicts or confirmation that none were found;
- the rebase result, the new head SHA, and the exact push command for the
  user;
- the `origin/main` SHA the branch now sits on;
- each commit's check result, the preflight result, and what they do not
  prove;
- the remaining decision or work, with a link to the PR.

Separate observed facts from judgment. Say what could not be verified instead
of manufacturing confidence.
