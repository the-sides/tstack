---
name: full-stop
description: Trigger this if the user says 'full stop', even as a closing "Full stop." that reads like emphasis. Finishes an iota-spear change and stops before the pull request, with a video or screenshots for the user to approve.
---

# Full Stop

For iota-spear (`Lamp-Post-Group/iota-spear`). If the user triggered this, they
are not watching this thread and want the work done end-to-end on their own.
Your updates go unread until long after you stop. You own the work from its
current state to a finished, committed branch, plus the evidence the user
needs to judge it by eye: a video, screenshots, or both.

Then you stop. Do not push and do not open a pull request. The user looks at
what you recorded, and decides whether it looks how they want before anything
reaches GitHub.

If the user did not clearly say "full stop", do not assume full autonomy. If
they did, everything below applies.

The repo rules this skill follows live in `AGENTS.md` and `docs/LANDING.md`.
If they changed since this skill was written, they win.

The user gives you permission to:

- Commit to this task's `work/*` branch in its own worktree.
- Bring that branch onto the latest `origin/main` as often as this skill
  says, the way `modernize` does it.
- Run the app, its dev servers, and a browser, and use the test methods that
  Proof needs, without asking first.

It does not give you permission to push, open or comment on a pull request,
or touch GitHub in any way that others can see.

## Testing permission

A "full stop" request is the permission. If a project file or other
instruction says to ask before a test method, or to leave testing to the
user, this request replaces that rule for this task.

That includes ask-first rules for computer use, browser automation, screen
recording, simulators and devices, running the app or a dev server,
screenshots, and visual checks.

Rules about how to test still apply. Follow rules that name the correct
command, fixture, launch lane, or check. This permission only removes the
ask-first or leave-it-to-the-user step.

It covers only the tests this task needs. It does not cover `pnpm db:reset`
with desks present, the shared `process-twin` lane from a worktree, killing
dev processes by pattern (every worktree's dev server has the same command
line), or anything under "Asking the user".

## Steps

1. **Confirm you're in a worktree.** The user's work has already started in
   one. Check that the current directory is a linked worktree, not the root
   checkout: `git rev-parse --git-dir` and `git rev-parse --git-common-dir`
   differ in a linked worktree. Work there, on its branch, and build on its
   commits and uncommitted changes. Do not create another worktree. If you are
   in the root checkout, ask the user which worktree to use. Then
   `git fetch origin`. If the branch has a PR, read its head's red CI jobs:
   they are work the lane still carries.
2. **Finish the change.** Complete the goal from the user's request with the
   smallest change that works. Fix the case that happened, not every case
   that could. Keep each commit to one bucket (chassis or product) or give it a
   `cross-bucket: <reason>` trailer, and check with
   `scripts/check-bucket-separation.ts`. Put the release note in its own
   `[apps/web/]changelog.d/<UTC date>-<topic>.md` file. If you edit
   `apps/web/DESIGN.md`, add one standalone change note,
   `apps/web/design-changes/<YYYY-MM-DD>-<topic>.md`, in the same commit (CI
   reds the PR without it). If the change rests on a business, legal, or trade
   fact nobody on the team can confirm alone, add an entry to
   `apps/web/docs/ASSUMPTIONS_TO_WITNESS.md` with the question, who can answer
   it, and what the product does until then. Regenerate generated artifacts in
   the same commit that moves them.
3. **Prove it works.** See "Proof".
4. **Get feedback.** Use the `sol-loop` skill to review the change. Do not let
   it push.
5. **Bring it current.** Use the `modernize` skill on this branch. It brings it
   onto the latest `origin/main`, runs `pnpm regen`, makes every commit green
   with its `check-commit` script, and runs `pnpm preflight` on the tip. See
   "Latest main, green commits".
6. **Show it.** Record the video and screenshots on the final commit. See
   "Show it".
7. **Check main one last time.** See "Latest main, green commits".
8. **Draft the PR, but do not open it.** See "PR draft".
9. **Hand off** with the report in "Handoff".

Stay active from step 1 to step 9. If you end your turn early, the work stops,
because nobody will prompt you again.

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

Say what each check does not prove. When you cannot find a way to prove a
change, ask the user.

## Latest main, green commits

The finished branch must sit on the newest `origin/main` you can find by the
end of the run, and every commit on it must pass its tests, not just the tip.

**Step 5.** Run `modernize` on the branch. It records its disposition, then:

- brings the branch current. A branch that was never pushed is rebased with
  `pnpm land:rebase`, which costs no CI run. A branch already on `origin` gets
  `origin/main` merged into it instead, because rewriting a pushed `work/*`
  branch reds CI's push-range check. Either way it runs `pnpm regen`;
- runs its `scripts/check-commit` on every commit it can still amend. A red
  commit is fixed in that commit, never by a later one;
- runs `pnpm preflight` on the tip, plus `pnpm build && pnpm build:smoke` if
  the change touches the published package.

If `modernize` says `REWORK` or `CLOSE` (for example, `main` already solved the
problem), stop and ask the user instead of recording.

**Step 7.** After recording, `git fetch origin` again. If `origin/main` is
still an ancestor of `HEAD`, you are done. If it moved:

1. Run step 5 again.
2. Retake the recordings when the new `main` commits touch `apps/`, or any
   file the change touches. Otherwise keep them, and say in the handoff which
   base they were recorded on.
3. Check again.

Stop after two extra rounds. If `main` is still moving, hand off on the last
base you finished, and say how many commits behind it is.

## Show it

The user will decide from what you record whether the change looks right, so
record what they would have clicked through themselves. Record on the final
commit, after step 5. A recording of an earlier commit does not count, except
as step 7 allows.

**Where it goes.** Save everything under the worktree's
`.local/full-stop/<topic>/` (gitignored). Name files so they sort in viewing
order: `01-queue-before-light.png`, `02-queue-after-light.png`,
`10-walkthrough.webm`. Never commit them.

**Run the app the repo's way.** Use the lane in `.claude/launch.json` that
serves what you changed: `process-twin-worktree` for a desk in a worktree
(never `process-twin`, which is the shared desk), `design-gallery` for a
component in the showroom, `design-mocks` for committed mockups, `qa` for
Pathfinder. Wait until the page renders real data, not a spinner. If the app
will not boot, say so in the handoff. Do not record a broken page as if it
were the change.

**Video,** for anything with motion, a flow across several screens, or an
interaction (open, drag, submit, undo). Use the harness's own browser
recording when it has one (in T3 Code, `preview_recording_start` and
`preview_recording_stop`). Otherwise write a short Playwright script outside
the repo and run it with the worktree's own Playwright
(`apps/web/node_modules/.bin/playwright` or `pnpm -C apps/web exec node
<script>`), with `browser.newContext({ recordVideo: { dir, size } })`. Keep each
video to one flow, under about a minute, at a pace a person can follow. Pause
on each state for a second.

**Screenshots,** for every screen the change touches:

- Before and after, from `origin/main` and from the branch, at the same URL,
  size, and data, so they can be compared side by side.
- Each state the change can show: loading, empty, error, denied, and stale,
  plus the opened or expanded state when something is folded away at first
  load.
- Light and dark, where the surface themes.
- Desktop width, and a narrow width when the surface is responsive.

Look at every image yourself before you hand it over. If one shows the wrong
thing, a spinner, a dev overlay, or an error you did not mean to capture,
retake it.

**Changes with nothing to see** (a script, a kernel primitive, a migration):
record the terminal instead. Save the reproduction before and after, and the
check output, as text files in the same folder.

## PR draft

Write the pull request you would open, so the user can file it once they
approve the visuals. Save it as `.local/full-stop/<topic>/pr.md`. Do not open
it.

Match the PRs that land in iota-spear:

**Title.** One line that says what changes for the person using it, with the
area tag recent PRs use: `[WEB]`, `[CHASSIS]`, `[INFRA]`, `[CI]`,
`[SECURITY]`, or a conventional `fix(web):` / `feat(payroll):`. Check the last
few merged PRs for the current habit. For example:

> ✅ [WEB] Take cost-rate journey dates from one clock read
>
> ✅ fix(bank-feed): date open AR/AP items by their own date (migration 0325)
>
> ❌ [WEB] Refactor lead-detail loaders and add helpers

**Body,** in this order:

1. **Summary.** The problem in plain words first, from the user's request,
   then what changes. Name who decided each product call ("decided by Jacob
   Sides unless a line names someone else"). Never attribute a decision to
   someone who did not make it. Say which bucket the commits are in, and where
   the release notes and design notes are.
2. **Verification.** What ran, on which commit: each commit's `check-commit`
   result, preflight on the tip, PG suites, and the reproduction.
3. **Screenshots.** Name the ones you recorded. The user attaches them when
   filing.
4. **What this does NOT prove.** No browser walk on a live company, suites not
   run, and anything else left unchecked.
5. **Decisions still open.** Each question, and whom it is for, pointing at its
   `ASSUMPTIONS_TO_WITNESS.md` entry when there is one.
6. **Follow-ups (not in this PR).**

End with the harness's attribution line, for example
`🤖 Generated with [Claude Code](https://claude.com/claude-code)`, or the
equivalent for your harness. Check the draft for secrets, customer data, and
private details.

## Handoff

Lead with the evidence, not the summary. Show the screenshots inline in your
final message, in viewing order, each with a one-line caption saying what it
shows and what to look for. Link each video and file by absolute path.

Then report:

- The worktree path, branch, final commit SHA, and the `origin/main` SHA it
  sits on, with the time you last fetched. State that nothing was pushed and
  no PR exists.
- Each commit with its `check-commit` result, and the preflight result on the
  tip.
- A try-it recipe: the launch lane, the URL, and the clicks that reach the
  change, so the user can see it live.
- Proof: the other checks you ran, their results, and what they do not prove.
- The Sol review outcome.
- Each decision you made for the user, follow-ups, and remaining risks.
- The path to `pr.md`.

Close with what happens next: when the user approves the visuals, the PR is
filed by "Checkout to merge in one lap" in `docs/LANDING.md`. That includes
the CodeRabbit CLI pilot run while its BET is open, and pushing as
`iota-spear-agent[bot]`. If they want changes, they say what, and you make
them and record again.

## Asking the user

A question in a normal message goes unseen, and the thread looks done. To ask
the user something, use the harness's question tool so the thread shows that
it is waiting: `AskUserQuestion` in Claude Code, `request_user_input` in
Codex, or the question tool your harness has. Put all the context in the
question: what blocks you and the options, with your recommendation first.

Ask only when you cannot continue alone:

- You cannot prove the change, or the app will not boot to show it.
- The next step is outside the permissions above: pushing, anything on
  GitHub, deleting data, touching production or Doppler, or changing things
  outside the repo.
- The goal is ambiguous in a way that changes the result, or needs a product,
  design, scope, or sequencing decision. Never record such a decision as the
  owner's or anyone else's when they did not make it.

Decide everything else yourself with sensible defaults, and list those
decisions in the report. A design choice you could not settle is better shown
than asked: record both versions and let the user pick from the screenshots.

If no question tool is available, or the tool returns no answer, start your
final message with the question.
