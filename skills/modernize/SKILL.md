---
name: modernize
description: >-
  For getting old iota-spear PRs and branches up to date with the latest
  main, with every commit green. Use when asked to rebase or update a PR or
  branch, or "modernize". full-stop uses it to finish on the latest main.
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
  user before bringing the branch current. (`full-stop` commits its work before it calls you.)
- You act on GitHub as the `iota-spear-agent[bot]` App. In Codex or another
  shell, run `eval "$(bash .claude/hooks/agent-identity.sh env)"` first.
- Modernizing authorizes bringing the branch current and pushing its `work/*`
  branch. It does not authorize approving, merging the PR, closing,
  changing the base branch, or expanding scope.
- **Never rewrite a branch that is already on `origin`.** CI prices each push
  range, and a force-push over a rebased `work/*` ref reds
  `push-range-unresolvable`. A pushed branch is brought current by merging
  `main` into it and pushing plain, as a fast-forward. Only a branch that was
  never pushed is rebased. Never use `git push --no-verify`.
- **Hands off a PR someone else holds.** If it carries `wip: <name>` for
  someone other than the user, `walk-in-progress`, `in-code-review`, or `held`,
  do not push. Report and ask.
- Do not post a PR comment unless the user asked for one. The App posts it,
  so add no signature or attribution callout. Lead with the conclusion, name
  the exact SHAs (for example "Published at `<sha>` by fast-forward from
  `<sha>`"), cite the check or code, and keep it short.
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
- **REWORK**: the goal is still useful, but bringing it current as-is would preserve
  the wrong implementation. Explain the smallest credible rewrite and stop
  before pushing unless the user also asked for that rework.
- **CLOSE**: the PR is superseded, obsolete, directionally wrong, or no longer
  worth its cost. Cite concrete evidence and stop. Do not spend time updating an
  obsolete branch, and do not close it without explicit permission.

Be willing to recommend `CLOSE`. Age alone is not evidence, and sunk effort is
not a reason to keep a PR alive.

## Bring it current without losing intent

For a `MODERNIZE` disposition, first check whether the branch is on
`origin`: `git ls-remote --heads origin <branch>`. Record its remote head SHA if
it is. That decides how it is brought current.

1. **Decide whether it needs a new base.** A move of `main` alone is not a
   reason. If preflight's `github-merge` step would merge it cleanly and
   nobody asked, skip to step 4: the branch needs checks, not a new base. A
   request from the user, or a call from `full-stop`, always brings it onto the
   latest `origin/main`.
2. **Bring it current.**
   - *Pushed branch:* `git merge --no-ff origin/main` (or the PR's base, for a
     stacked PR). Resolve each conflict, then `git add --resolved` and review
     the staged diff. Run `pnpm regen` and amend what it writes into the merge
     commit. The merge commit's message names the `main` SHA it took and how
     each meaningful conflict was resolved.
   - *Never-pushed branch:* `pnpm land:rebase` (or
     `pnpm land:rebase -- <base>`). It rebuilds the generated ledger at each
     commit and stops on real conflicts. Resolve them, then run `pnpm regen`
     and amend each regenerated file into the commit that owns it. Compare
     the old and new series with `git range-diff`.

   Either way, resolve conflicts by the branch's current intent and `main`'s
   newer invariants. If a conflict reveals a product or architecture
   disagreement, stop and reconsider the disposition instead of guessing.
   Inspect the final diff against `origin/main`: no intended behavior
   disappeared and no already-landed behavior came back.
3. **Make every new commit green.** Run `scripts/check-commit` from this
   skill's directory (use its absolute path). It typechecks and runs the unit
   tests for each area the commit touches (root, `apps/web`, `apps/qa`).
   - *Never-pushed branch:* every commit, with
     `git rebase --reschedule-failed-exec --exec "bash <abs path>/check-commit" origin/main`.
     When a commit fails, the rebase stops on it. Fix it in that commit,
     `git commit --amend`, and `git rebase --continue`.
   - *Pushed branch:* every commit after the remote head, including the merge
     commit. Check each out in turn (`git rev-list --reverse <remote head>..HEAD`)
     and run the script, then return to the branch. Fix a red unpushed commit in
     place, before anything is pushed. Commits already on `origin` cannot be
     rewritten. If one of those is red, say so in the report.

   Never fix a red commit with a later commit while it can still be amended.
   Record each commit's result.
4. Run `pnpm preflight` once and fix every red. If the branch touches the
   published package, also run `pnpm build && pnpm build:smoke`. Tell real
   failures apart from runner or infrastructure flakes, and from reds already
   on `main`. Preflight runs on the tip only. It covers the database suites and
   the GitHub-style merge that step 3 does not.
5. **Push,** only when the branch already has a PR: `git push origin HEAD`, a
   plain fast-forward. If it is refused as non-fast-forward, someone pushed:
   fetch and reassess rather than overwriting their work. A branch
   `full-stop` is building is never pushed.

Do not merge the PR.

## Report

Lead with `MODERNIZE`, `REWORK`, or `CLOSE` and the reason. Then summarize:

- meaningful changes on `main`;
- conceptual conflicts or confirmation that none were found;
- whether it was merged or rebased, the push result if performed, and the new
  head SHA;
- the `origin/main` SHA the branch now sits on;
- each commit's check result, the preflight result, and what they do not
  prove;
- the remaining decision or work, with a link to the PR.

Separate observed facts from judgment. Say what could not be verified instead
of manufacturing confidence.
