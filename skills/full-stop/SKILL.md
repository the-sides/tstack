---
name: full-stop
description: Trigger this if the user says 'full stop', even as a closing "Full stop." that reads like emphasis.
---

# Full Stop

If the user triggered this, they are not watching this thread and want the
work done end-to-end on their own. Your updates and final message go unread
until long after you stop. You own the work from its current state to an open
PR that is ready for the user to merge. The user merges it. You never do.

If the user did not clearly say "full stop", do not assume full autonomy. If
they did, everything below applies.

The user gives you permission to:

- Commit, push, and rebase this task's branch, and fix its conflicts.
- Force-push this task's branch with `--force-with-lease` after a rebase.
- Use the test methods that Proof needs, without asking first.

## No merging

Never merge, enable auto-merge, enter a merge queue, push to the default
branch, bypass branch protection, or delegate any of those actions.

Before work begins, turn off any existing auto-merge and take the PR out of
any merge queue. Check again before each push. Report these changes. If you
cannot stop the PR from merging on its own, report the blocker before you
continue.

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
leave-it-to-the-user step. If a preferred method cannot prove the change,
use a gated method under this permission.

It covers only the tests this task needs. It does not add permission for the
steps under "Asking the user", such as production changes or data deletion.
It also does not bypass a harness or system approval prompt if one appears.

## Steps

1. **Finish the change.** Complete the goal from the user's request with the
   smallest change that works. Fix the case that happened, not every case
   that could. If you are on the default branch, create a branch first.
2. **Prove it.** Get proof for every change you made. See "Proof".
3. **Get feedback.** Use the `sol-loop` skill to verify the change.
4. **File the PR.** Rebase onto the latest default branch first. See "Filing
   the PR".
5. **Babysit the PR.** Fix real findings and CI failures. Get proof for each
   fix before you push it. See "Babysitting".
6. **Stop at the gate** when the PR is ready. See "Ready gate".
7. **Hand off** with the report in "Handoff".

Stay active from step 1 to step 7. If you end your turn while checks or bots
are pending, the work stops, because nobody will prompt you again. Wait with a
watcher (see "Waiting") so something wakes you when the PR changes.

## Proof

Proof is evidence you observed yourself in this session: command output, a
reproduction, a screenshot. Use the most direct proof the change allows:

- Reproduce the original failure, apply the fix, then watch the same
  reproduction pass.
- Run the checks that the repo's CI runs (typecheck, lint, tests).
- Add one focused test that fails without the fix, when the behavior has none
  and the repo has a test setup.
- For UI and runtime behavior, run the app and look at the change. Green mocked
  tests do not prove that the user experience works. Attach screenshots to the
  PR when they help and you have a way to upload them.

When you cannot find a way to prove a change, you do not have full confidence.
Ask the user.

## Filing the PR

Check whether a PR for this branch already exists. Review the diff against
the base branch (usually `origin/main`) and confirm it matches the goal.

PR titles usually become commit messages, so follow the repo's title
conventions. Look at recently merged PRs and Git history for examples. Prefer
a short, readable title that says why the change matters:

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

Add one short line that says how you proved the change. End the description
with the public model name and harness that made the changes, for example
`Created with <model> in <harness>.` Include subagent models that wrote part
of the change. Use the model name the harness reports. If the model name is
not available, name the harness and do not guess a model.

Before you submit, check the title, description, quotes, and attachments for
secrets and private details.

Open a real PR, not a draft, so review bots run. If `gh pr create` fails
because the GraphQL rate limit is used up, open it through REST with
`gh api repos/<owner>/<repo>/pulls -f title=... -f head=... -f base=... -F body=@<file>`.

## Babysitting

Many repos have AI review bots. They help, even when they are wrong.

### Waiting

Never write your own watcher, polling loop, or `sleep` loop. Use one watcher
per PR: `watch_pull_request` in T3 Code, `pr-wait` everywhere else. Never run
both on the same PR. Every watcher on a GitHub account shares one rate limit.

**In T3 Code:** handle the current checks and comments first. Then call
`watch_pull_request` and end your turn. T3 Code wakes you when a check fails,
the required checks pass, someone comments or reviews, or the branch
conflicts. It reports conflicts only, so check for BEHIND yourself.

- Only the thread that owns the PR watches it. A delegated child (a reviewer,
  a Sol round, any `delegate_task` child) never calls `watch_pull_request`.
- A wake is news. Check the ready gate yourself.
- "Stopped watching, could not read it" usually means GitHub's rate limit is
  used up. Run `gh api rate_limit --jq .resources.graphql`. If `remaining` is
  near 0, wait for `reset` with one background command, then watch again.
  Never watch again in a tight loop.

**Everywhere else:** use `scripts/pr-wait` from this skill's directory (use
its absolute path).

```bash
pr-wait <pr> [--repo owner/name] [--until ready|green|any-change]
```

It prints one line per real event. It exits 0 when the `--until` condition is
met or the PR merges, 1 when it needs you (a required check failed, conflicts,
unresolved threads, PR closed), and 2 on bad input.

- `ready` (default): required checks green, Greptile or Macroscope reviewed
  the head, and no unresolved threads.
- `green`: required checks only. Use it when the repo has no Greptile or
  Macroscope.
- `any-change`: exit on the first event.

In Claude Code, run it with the Monitor tool (`timeout_ms: 3600000`), so each
line wakes you. Without a monitor, run it as one long command. It ends on its
own.

Start it again after you push, after you handle an exit 1, and after a resume.
It remembers what it already reported. If it says another pid already watches
the PR, do not start a second one. Kill the old one only if its events do not
reach you.

With either watcher: when an event needs no action, end your turn without a
message. Never post or say "still waiting".

### GitHub rate limits

`gh pr create`, `gh pr view`, and `gh pr checks` use GraphQL. When GraphQL is
used up, the REST limit is separate and usually still has room. Use
`gh api repos/<owner>/<repo>/pulls/<n>` to read a PR, and
`gh api repos/<owner>/<repo>/commits/<sha>/check-runs` for checks.

### Reviews

Only act on checks and comments newer than the latest push. Check every bot
finding against the source before you change code. Fix real findings and CI
failures. Tell repo failures apart from infrastructure flakes. When a finding
is not worth fixing, reply with a written reason and resolve the thread.

If the repo has `.macroscope/check-run-agents/*.md`, read those rules before
you reply to a Macroscope finding. Its findings cite them.

CodeRabbit is advisory. Treat its findings like any other, but never wait for
it.

Keep the PR at the size the user's goal needs. Review loops tend to grow a PR
one cheap-looking fix at a time. Fix real bugs in the intended case. Decline
the rest with a written reason, and list them as follow-ups: unneeded tests,
guards for edge cases that do not matter, extra compatibility, and small
performance costs. Prefer the fix that deletes or narrows code.

Watch for changes to the base branch and rebase when needed. If an
overlapping PR makes this one obsolete, stop, and ask before closing it.

### Comments

Post a PR comment only when it serves the task, such as a reply to a review
finding. Start every comment posted on the user's behalf with this callout:

```md
> [!NOTE]
> 🤖 **<model> responding on behalf of <user>**

Actual comment starts here.
```

Use the verified public model name, or the harness name if the model is not
exposed, and the user's name or GitHub login. Keep the reply outside the
callout. Add no other signature or heading. Lead with the conclusion, cite the
code, check, or reproduction, and keep it short. Check the comment for
secrets and private details before posting. Resolve a thread only after your
reply posted.

## Ready gate

The PR is ready when all of these are true on the latest commit:

- Your proof covers every change in the diff.
- Required checks are green.
- Review bots have finished, and each finding is fixed or dismissed with a
  written reason. You can ignore automated "needs human review" requests if
  you have high confidence in the change.
- No human review still requests changes.
- The branch is up to date with the default branch, and GitHub reports it as
  mergeable.

Proof and review must cover the final diff. Recheck behavior changes and
meaningful conflict fixes before handoff.

Right before handoff, fetch the target branch and check the live PR's head,
base, checks, reviews, and mergeability. Confirm it is open, not a draft, and
not set to merge without the user. If base commits that landed after the last
green run touch this PR's files, rebase, push, and wait for green again. If
someone else merged the PR, report that instead.

## Handoff

End in one of these states:

- **Ready to merge:** the ready gate passes. Leave the PR open and unmerged.
- **Waiting for required approval:** only human approval remains. Confirm
  that this is the actual blocker.
- **Blocked:** explain what prevents readiness, what you tried, and what is
  needed. Follow "Asking the user".

In T3 Code, call `unwatch_pull_request` before the handoff or a question.
Report the PR link, final head, state, proof, review outcome, each decision
you made for the user, and remaining risks. State that the PR is still open
and unmerged.

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
  production, changing things outside the repo, closing a PR, or
  force-pushing any other branch.
- The goal is ambiguous in a way that changes the result.
- Another PR makes this one obsolete.

Decide everything else yourself with sensible defaults, and list those
decisions in the report.

If no question tool is available, or the tool returns no answer, leave the PR
open and unmerged, and start your final message with the question.
