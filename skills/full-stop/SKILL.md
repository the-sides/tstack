---
name: full-stop
description: Trigger this if the user says 'full stop', even as a closing "Full stop." that reads like emphasis. Takes an iota-spear change to an open pull request that is ready for the owner to merge.
---

# Full Stop

For iota-spear (`Lamp-Post-Group/iota-spear`). If the user triggered this, they
are not watching this thread and want the work done end-to-end on their own.
Your updates and final message go unread until long after you stop. You own the
work from its current state to an open pull request that is ready for a human
to merge. A human merges it from their own terminal. You never do.

If the user did not clearly say "full stop", do not assume full autonomy. If
they did, everything below applies.

The rules this skill follows live in the repo's `AGENTS.md` and
`docs/LANDING.md` ("Checkout to merge in one lap", "Agent identity", "PR status
labels"). If they changed since this skill was written, they win. Read them
when a step here does not match what the tree does.

## Who you are on GitHub

You push and open pull requests as the `iota-spear-agent[bot]` GitHub App, never
as a person. In Claude Code the SessionStart hook wires this. In Codex or
another shell, run `eval "$(bash .claude/hooks/agent-identity.sh env)"` first.
If the hook says the identity is not wired, do not push or open a pull request:
ask the user.

Never approve or merge a pull request, never run `git push --no-verify`, and
never use the owner's credentials or `SPEAR_GH_AS_OWNER`. The PreToolUse guard
refuses most of these. Do not work around it.

The user gives you permission to:

- Commit to and push this task's `work/*` branch, and open its pull request.
- Rebase that branch when it has a real conflict, and push it with
  `--force-with-lease`.
- Use the test methods that Proof needs, without asking first.

## Testing permission

A "full stop" request is the permission. If a project file or other
instruction says to ask before a test method, or to leave testing to the
user, this request replaces that rule for this task.

That includes ask-first rules for computer use, browser automation,
simulators and devices, running the app or a dev server, screenshots, and
visual checks. Use the method that gives direct proof. List it in the
report.

Rules about how to test still apply. Follow rules that name the correct
command, fixture, or check. This permission only removes the ask-first or
leave-it-to-the-user step.

It covers only the tests this task needs. It does not cover `pnpm db:reset`
with desks present, killing dev processes by pattern (every worktree's dev
server has the same command line), or anything under "Asking the user".

## Steps

1. **Start clean.** `git fetch origin`, then work in a worktree, never in the
   root checkout. For new work run `pnpm worktree:new <topic>`, which creates
   `../iota-spear-<topic>` on `work/<topic>`. For an existing pull request,
   use a worktree on its branch, and read the head's red CI jobs first: they
   are work the lane still carries.
2. **Finish the change.** Complete the goal from the user's request with the
   smallest change that works. Fix the case that happened, not every case
   that could. Keep each commit to one bucket (chassis or product) or give it a
   `cross-bucket: <reason>` trailer, and check with
   `scripts/check-bucket-separation.ts`. Put the release note in its own
   `[apps/web/]changelog.d/<UTC date>-<topic>.md` file. Regenerate generated
   artifacts in the same commit that moves them.
3. **Prove it.** Get proof for every change you made. See "Proof".
4. **Get feedback.** Use the `sol-loop` skill to review the change.
5. **Preflight.** Rebase only for a real conflict (see "Rebasing"). Then run
   `pnpm preflight` once and fix every red. If the change touches the
   published package, also run `pnpm build && pnpm build:smoke`. Start the
   CodeRabbit CLI review beside it (see "CodeRabbit CLI pilot").
6. **Push and file the PR** on the exact commit preflight and CodeRabbit
   checked. See "Filing the PR".
7. **Babysit the PR.** Let the whole CI run and each review app's first pass
   finish, then answer everything in one batch and push once. See
   "Babysitting".
8. **Stop at the gate** when the PR is ready. See "Ready gate".
9. **Hand off** with the report in "Handoff".

Stay active from step 1 to step 9. If you end your turn while checks or
reviews are pending, the work stops, because nobody will prompt you again.
Wait with a watcher (see "Waiting") so something wakes you when the PR changes.

## Proof

Proof is evidence you observed yourself in this session: command output, a
reproduction, a screenshot. Use the most direct proof the change allows:

- Reproduce the original failure, apply the fix, then watch the same
  reproduction pass.
- Run the narrow checks for what you touched while you work
  (`pnpm test <file>`, `pnpm typecheck`, `pnpm -C apps/web gate`). If a
  verification command is queued, leave it running and read
  `pnpm verify:status`. Never start a duplicate.
- Add one focused test that fails without the fix, when the behavior has none.
- A change to durable behavior is not proven until `pnpm test:integration:pg`
  is green for it.
- For user-facing work, run the app and look at it. The work is done only
  when the report shows screenshots, including loading, empty, error, denied,
  and stale states, and light and dark where it themes, plus a short try-it
  recipe. Attach them to the PR when you can upload them.

Say what each check does not prove. When you cannot find a way to prove a
change, ask the user.

## Rebasing

A move of `main` alone is not a reason to rebase. Rebase only when GitHub's
merge would conflict: `pnpm land:board -- <n>` shows it in the `merge` column,
and preflight's `github-merge` step merges the way GitHub does.

To rebase, run `pnpm land:rebase` (`pnpm land:rebase -- <base>` for a stacked
lane), then `pnpm regen`, and amend each regenerated file into the commit that
owns it. Never leave a red commit for a later regen commit. Push with
`--force-with-lease` against the head you saw before the rebase. If the lease
fails, fetch and reassess.

## CodeRabbit CLI pilot

Until 2026-11-03 (the `coderabbit-cli-pilot` BET in `docs/LANDING.md`), every
pull request an agent opens carries a CodeRabbit CLI review of its pushed
commit. Run `bash .claude/hooks/coderabbit-pilot.sh status` right before the
review:

- `on`: run the review exactly as "CodeRabbit CLI pilot" in `docs/LANDING.md`
  shows, in the background on the commit you will push.
- `skipped:no-cli` or `skipped:no-auth`: record that as `run=` and go on. Never
  sign the CLI in yourself.
- `off`: the pilot is over. Skip this section.

Put the result in the PR body in the shape `docs/LANDING.md` gives, with each
critical and major finding as a bullet. Never post the findings as review
threads. Answer them in step 7's batch, then mark each bullet `→ fixed <sha7>`
or `→ rebutted: <reason>` and update `fixed=` and `rebutted=`.

## Filing the PR

Push with `git push -u origin work/<topic>`. Check whether a PR for this branch
already exists. Review the diff against `origin/main` and confirm it matches
the goal.

PR titles become the merge commit's subject, so follow the conventions of
recently merged PRs. Prefer a short, readable title that says why the change
matters:

BAD

> ❌ perf(server): negotiate permessage-deflate on the websocket

GOOD

> ✅ perf(server): cut websocket frame size by 70%+ with gzipping

Open the description with a plain explanation of the problem, based on the
user's original request, then briefly explain the fix. Do not lead with a
list of everything you changed:

BAD

> ❌ Removed implicit workspace carry-over from every "new thread" entry point (cmd+n / cmd+shift+o, sidebar v1/v2 buttons, command palette). New threads inherit only the project from context; branch, worktree, and env mode always come from the configured defaults. Deleted buildContextualThreadOptions, startNewThreadInProjectFromContext, and the v1 sidebar's seed-context machinery.

GOOD

> ✅ My "new worktree" default was ignored when starting new threads on existing worktrees. Super unintuitive. Now your preferences always apply.

Then add, in this order:

- One short line that says how you proved the change.
- The CodeRabbit CLI pilot line and its bullets, while the pilot is on.
- `Created with <model> in <harness>.`, naming subagent models that wrote part
  of the change. Use the model name the harness reports. If it is not
  available, name the harness and do not guess.

Before you submit, check the title, description, quotes, and attachments for
secrets, customer data, and private details.

Open a real PR against `main`, not a draft. Claude Code Review reviews only the
head that is current when the PR opens and never re-reviews a later push, so
open it on the commit you mean to have reviewed.

## Babysitting

### Waiting

Never write your own watcher, polling loop, or `sleep` loop. Use one watcher
per PR: `watch_pull_request` in T3 Code, `pr-wait` everywhere else. Never run
both on the same PR. Every watcher on a GitHub account shares one rate limit.

**In T3 Code:** call `watch_pull_request` and end your turn. T3 Code wakes you
when a check fails, the required checks pass, someone comments or reviews, or
the branch conflicts.

- Only the thread that owns the PR watches it. A delegated child (a reviewer,
  a Sol round, any `delegate_task` child) never calls `watch_pull_request`.
- A wake is news. Check the ready gate yourself.
- "Stopped watching, could not read it" usually means GitHub's rate limit is
  used up. Run `gh api rate_limit --jq .resources.graphql`. If `remaining` is
  near 0, wait for `reset` with one background command, then watch again.
  Never watch again in a tight loop.

**Everywhere else:** use `scripts/pr-wait` from this skill's directory (use
its absolute path) with `--until green`. iota-spear has no Greptile or
Macroscope, so `--until ready` never finishes there.

```bash
pr-wait <pr> --repo Lamp-Post-Group/iota-spear --until green
```

It prints one line per real event. It exits 0 when every CI check on the head
passed, 1 when it needs you (a check failed, conflicts, PR closed), and 2 on
bad input. It gates on every CI check, including the observe-only
`qa readonly Process Twin cases (observe-only)` row, which holds no merge: if
that row alone fails, treat it as green. It reports BEHIND too. BEHIND alone is
not a reason to rebase.

In Claude Code, run it with the Monitor tool (`timeout_ms: 3600000`), so each
line wakes you. Without a monitor, run it as one long command. Start it again
after you push and after you handle an exit 1. If it says another pid already
watches the PR, do not start a second one.

With either watcher: when an event needs no action, end your turn without a
message. Never post or say "still waiting".

### GitHub rate limits

`gh pr create`, `gh pr view`, and `gh pr checks` use GraphQL. When GraphQL is
used up, the REST limit is separate and usually still has room. Use
`gh api repos/Lamp-Post-Group/iota-spear/pulls/<n>` to read a PR, and
`gh api repos/Lamp-Post-Group/iota-spear/commits/<sha>/check-runs` for checks.
If `gh pr create` fails on the GraphQL limit, open it with
`gh api repos/Lamp-Post-Group/iota-spear/pulls -f title=... -f head=... -f base=main -F body=@<file>`.

### Reviews

Wait until the whole CI run and each review app's first pass, including
Claude Code Review, have finished. Change nothing before that. Then answer
every review thread in one batch, with every CodeRabbit CLI finding beside
them, and push once:

- Check every finding against the source before you change code.
- Fix what is real. Tell repo failures apart from runner or infrastructure
  flakes. A runner fault is re-run (`gh run rerun <id> --failed`), not fixed.
- Answer what is not real with evidence, and resolve the thread after your
  reply posts.
- Move what is not blocking to a follow-up and list it in the report.

Keep the PR at the size the user's goal needs. Fix real bugs in the intended
case. Decline unneeded tests, guards for edge cases that do not matter, extra
compatibility, and small performance costs. Prefer the fix that deletes or
narrows code. Get proof for each fix before you push it.

If an overlapping PR makes this one obsolete, stop, and ask before closing it.

### Comments

Post a PR comment only when it serves the task, such as a reply to a review
finding. Start every comment you post with this callout:

```md
> [!NOTE]
> 🤖 **<model> responding on behalf of <user>**

Actual comment starts here.
```

Use the verified public model name, or the harness name if the model is not
exposed, and the name of the person who started the session. Keep the reply
outside the callout. Add no other signature or heading. Lead with the
conclusion, cite the code, check, or reproduction, and keep it short. Check
the comment for secrets and private details before posting.

### Status labels

The PR's status is exactly one `status:` label (`docs/LANDING.md`, "PR status
labels: who writes which"). The reconciler writes `ci-running`, `ci-red`,
`needs-QA-walk`, `needs-rebase`, and `awaiting-deploy`. Do not set those by
hand. For the PR you drive, swap in a person's label in the same step that
changes its state:

- `status: needs-design-review` when a UI change is ready for design review.
- `status: needs-product-decision` when a call only the owner or product can
  make stops the work.
- `status: blocked` when something outside the PR has to happen first, with a
  comment that says what.

Never apply `cleared` or `held`, never invent a label, and do not set
`status: ready-for-review` unless a QA walk passed on the final head.

## Ready gate

The PR is ready when all of these are true on the latest head:

- Your proof covers every change in the diff, including fixes from review.
- `pnpm land:merge <n>:<full head sha> --dry-run` passes. It checks that the PR
  is open at that head, based on `main`, not a draft, `MERGEABLE`, every check
  row green (the observe-only row excepted), no unresolved review thread, and
  no outstanding change request, and that `main`'s own CI is green. It writes
  nothing. Exit 3 means a guard refused: do what it says. Exit 4 means re-run
  it.
- Claude Code Review has posted its pass and its threads are resolved.
- The CodeRabbit pilot line is complete, while the pilot is on.

`land:merge` itself (without `--dry-run`) is the owner's. Never run it.

## Handoff

End in one of these states:

- **Ready to merge:** the ready gate passes. Leave the PR open. Give the owner
  the exact line to run from their own terminal:
  `pnpm land:merge <n>:<full head sha>`. A human who is not the PR's author
  approves before the merge.
- **Waiting on a person:** only a human step remains (approval, QA walk,
  design review, a product decision). Confirm that this is the actual blocker
  and that the status label says so.
- **Blocked:** explain what prevents readiness, what you tried, and what is
  needed. Set `status: blocked` with a comment. Follow "Asking the user".

In T3 Code, call `unwatch_pull_request` before the handoff or a question.
Report the PR link, final head SHA, state, proof (with screenshots for
user-facing work), what each check does not prove, review outcome, each
decision you made for the user, follow-ups, and remaining risks. State that
the PR is still open and unmerged.

## Asking the user

A question in a normal message goes unseen, and the thread looks done. To ask
the user something, use the harness's question tool so the thread shows that
it is waiting: `AskUserQuestion` in Claude Code, `request_user_input` in
Codex, or the question tool your harness has. Put all the context in the
question: the PR link, what blocks you, and the options with your
recommendation first.

Ask only when you cannot continue alone:

- You cannot prove the change.
- The ready gate cannot pass, and you cannot fix that yourself.
- The next step is outside the permissions above: deleting data, touching
  production or Doppler, changing things outside the repo, closing a PR, or
  pushing any branch that is not this task's `work/*` branch.
- The goal is ambiguous in a way that changes the result, or needs a product,
  design, scope, or sequencing decision. Never record such a decision as the
  owner's or anyone else's when they did not make it.
- Another PR makes this one obsolete.

Decide everything else yourself with sensible defaults, and list those
decisions in the report.

If no question tool is available, or the tool returns no answer, leave the PR
open and unmerged, and start your final message with the question.
