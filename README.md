# lean-skills

Skills that keep a codebase lean, so every reader, human or agent, spends
less context on text the code does not need. Both run only when you invoke
them.

| Skill | Does |
| --- | --- |
| [`commentdiet`](skills/commentdiet/SKILL.md) | Code review focused on comments: clearer names and code instead of explanatory comments, trimmed comments, no noise or stale ones |
| [`docdiet`](skills/docdiet/SKILL.md) | Bounded documentation review and update that keeps canonical docs accurate and compact |

## Install

```bash
git clone https://github.com/Juan23/lean-skills.git ~/Projects/lean-skills
```

Then link each skill into your agent's skills folder:

```bash
for s in commentdiet docdiet; do ln -s ~/Projects/lean-skills/skills/$s ~/.claude/skills/$s; done
```

For Codex, link into `~/.codex/skills` the same way.

## commentdiet

| Invocation | Reviews |
| --- | --- |
| `/commentdiet` | Your branch's changes since the default branch, before you open a pull request |
| `/commentdiet <ref>` | Changes since a commit, branch or tag |
| `/commentdiet pr <number>` | A pull request |
| `/commentdiet repo` or `/commentdiet <path>` | Every source file in the repo or a folder |

It reports and changes nothing; append `fix` to apply the findings. In Codex,
invoke it as `$commentdiet`.

## License

MIT. Use it, modify it, and redistribute it freely.
