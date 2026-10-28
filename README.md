# tstack

My agent skills for [iota-spear](https://github.com/Lamp-Post-Group/iota-spear).
They follow its `AGENTS.md` and `docs/LANDING.md`: work in `work/*`
worktrees, rebase instead of merging `main` in, run `pnpm preflight`, and never
push, approve, or merge. A human merges. They work in Claude
Code and Codex, and use [T3 Code](https://github.com/pingdotgg/t3code) features
when they are available.

## Skills

| Skill | Say | What it does |
|---|---|---|
| `full-stop` | "full stop" | Finishes an iota-spear change with nobody watching: proves it, reviews it, rebases it onto the latest `main` with every commit green, and stops before any push or PR with a video, screenshots, and a drafted PR for you to approve. |
| `sol-loop` | "sol loop" | Reviews a change with GPT Sol in a loop until both agents agree. |
| `modernize` | "modernize" | Rebases an iota-spear PR or branch onto the latest `main` with `land:rebase` and `regen`, checks every commit is green, and decides if it should still exist. Commits, never pushes. |

`full-stop` uses `sol-loop` and `modernize`. Install all three.

## Install

```bash
cp -R skills/* ~/.claude/skills/   # Claude Code
cp -R skills/* ~/.codex/skills/    # Codex
```

## Requirements

- An authenticated [GitHub CLI](https://cli.github.com) (`gh`).
- `sol-loop` needs a subagent on GPT Sol. T3 Code does this with
  `delegate_task`. Otherwise you need a harness that can run that model.

## License

[MIT](LICENSE). Forked from [t3dotgg/tstack](https://github.com/t3dotgg/tstack).
