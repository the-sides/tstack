---
name: modernize
description: >-
  For getting old iota-spear PRs and branches up to date with the latest
  main, with every commit green. Use when asked to rebase a PR or branch, or
  "modernize". full-stop uses it to finish on the latest main.
---

# Modernize

For iota-spear (`Lamp-Post-Group/iota-spear`). Modernize the idea, not just the
branch. A clean rebase does not prove that the PR still belongs in the product.

The rules this skill follows live in the repo's `AGENTS.md` and
`docs/LANDING.md`. If they changed since this skill was written, they win.

## Guardrails

- `git fetch origin` before anything, then read `git log -1 origin/main`. Bare
  `main` only moves on fetch, so it can look current while stale.
- Work in a worktree, never in the root checkout, and never in a worktree
  another session uses. Treat existing worktree changes as someone else's.
  `pnpm land:board -- <n>` shows in its `held` column whether a worktree already
  has the PR's branch checked out.
- You act on GitHub as the `iota-spear-agent[bot]` App. In Codex or another
  shell, run `eval "$(bash .claude/hooks/agent-identity.sh env)"` first.
- Modernizing authorizes rebasing and updating the PR's `work/*` branch. It does
  not authorize approving, merging, closing, changing the base branch, or
  expanding scope.
- Never use an unqualified force push or `git push --no-verify`. Update a
  rebased branch with `--force-with-lease` against the head SHA you saw before
  the rebase.
- Do not post a PR comment unless the user asked for one. Start every comment
  you post with this callout, using the verified public model name (or the
  harness name if it is not exposed) and the name of the person who started
  the session:

  ```md
  > [!NOTE]
  > 🤖 **<model> responding on behalf of <user>**

  Actual comment starts here.
  ```

- Never apply the `cleared` or `held` labels.

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

## Choose a disposition before pushing

Use one of these recommendations:

- **MODERNIZE**: the goal is still useful and the implementation remains the
  right shape. Bring it current (below), test, and update the branch.
- **REWORK**: the goal is still useful, but a mechanical rebase would preserve
  the wrong implementation. Explain the smallest credible rewrite and stop
  before pushing unless the user also asked for that rework.
- **CLOSE**: the PR is superseded, obsolete, directionally wrong, or no longer
  worth its cost. Cite concrete evidence and stop. Do not spend time rebasing an
  obsolete branch, and do not close it without explicit permission.

Be willing to recommend `CLOSE`. Age alone is not evidence, and sunk effort is
not a reason to keep a PR alive.

## Bring it current without losing intent

For a `MODERNIZE` disposition:

1. Run `pnpm land:board -- <n>`. A move of `main` alone is not a reason to
   rebase. If its `merge` column is clean and nobody asked for a rebase, skip
   to step 5: the branch needs checks, not a new base. A rebase request from
   the user, or a call from `full-stop`, always rebases onto the latest
   `origin/main`. A branch with no PR skips `land:board`.
2. In a worktree on the PR's branch, run `pnpm land:rebase` (or
   `pnpm land:rebase -- <base>` for a stacked PR). It rebuilds the generated
   ledger at each commit. It stops on real conflicts as usual.
3. Resolve each conflict by the PR's current intent and `main`'s newer
   invariants. After editing, stage with `git add --resolved` and review the
   staged diff. If a conflict reveals a product or architecture disagreement,
   stop and reconsider the disposition instead of guessing.
4. Run `pnpm regen` (it must follow every rebase) and amend each regenerated
   file into the commit that owns it. Never leave a red commit for a separate
   regen commit. Compare the pre-rebase and post-rebase series with
   `git range-diff` and inspect the final diff. Confirm no intended behavior
   disappeared and no already-landed behavior came back.
5. **Make every commit green.** Run `scripts/check-commit` from this skill's
   directory (use its absolute path) on each commit of the branch:

   ```bash
   git rebase --reschedule-failed-exec --exec "bash <abs path>/check-commit" origin/main
   ```

   It typechecks and runs the unit tests for each area the commit touches
   (root, `apps/web`, `apps/qa`). When a commit fails, the rebase stops on it.
   Fix it in that commit, `git commit --amend`, and `git rebase --continue`.
   Never fix a red commit with a later commit. Use `<base>` in place of
   `origin/main` for a stacked branch. Record each commit's result.
6. Run `pnpm preflight` once and fix every red. If the PR touches the published
   package, also run `pnpm build && pnpm build:smoke`. Tell real failures apart
   from runner or infrastructure flakes, and from reds already on `main`.
   Preflight runs on the tip only. It covers the database suites and the
   GitHub-style merge that step 5 does not.
7. Push with `--force-with-lease` tied to the previously observed remote head,
   only when the branch already has a PR. If the lease fails, fetch and
   reassess rather than overwriting new work. A branch `full-stop` is building
   is never pushed.

Do not merge the PR.

## Report

Lead with `MODERNIZE`, `REWORK`, or `CLOSE` and the reason. Then summarize:

- meaningful changes on `main`;
- conceptual conflicts or confirmation that none were found;
- the rebase and push result, if performed, and the new head SHA;
- the `origin/main` SHA the branch now sits on;
- each commit's check result, the preflight result, and what they do not
  prove;
- the remaining decision or work, with a link to the PR.

Separate observed facts from judgment. Say what could not be verified instead
of manufacturing confidence.
