# tstack

My agent skills. They work in Claude Code and Codex. Some use
[T3 Code](https://github.com/pingdotgg/t3code) features when they are
available and fall back to native tools when they are not.

None of these skills merge a PR. They stop when the PR is ready, and you merge
it.

## Skills

| Skill | Say | What it does |
|---|---|---|
| `full-stop` | "full stop" | Takes a change to an open PR with nobody watching: proves it, reviews it, files it, babysits it, and stops when it is ready for you to merge. |
| `sol-loop` | "sol loop" | Reviews a change with GPT Sol in a loop until both agents agree. |
| `modernize` | "modernize" | Rebases an old PR and decides if it should still exist. |

`full-stop` uses `sol-loop`. Install both.

## Install

```bash
cp -R skills/* ~/.claude/skills/   # Claude Code
cp -R skills/* ~/.codex/skills/    # Codex
```

## Requirements

- An authenticated [GitHub CLI](https://cli.github.com) (`gh`).
- `python3` for `full-stop`'s `pr-wait` script.
- `sol-loop` needs a subagent on GPT Sol. T3 Code does this with
  `delegate_task`. Otherwise you need a harness that can run that model.

## License

[MIT](LICENSE). Forked from [t3dotgg/tstack](https://github.com/t3dotgg/tstack).
