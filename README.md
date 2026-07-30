# idos — Plan First, Build Later

A Claude Code skill implementing a two-model workflow: **plan** on the strongest reasoning
model (Opus 5 or Fable 5), **build** on Sonnet 5.

Invoke with `/idos <task>`. It will:

1. Confirm it's running on the right planning model (Opus 5 for ambiguous/architectural work,
   Fable 5 for well-scoped/creative work) — and tell you to switch if it isn't.
2. Ask every clarifying question needed, in rounds, with no cap.
3. Map which installed skills the task needs and when each one fires.
4. Write a full plan to `PLAN-<slug>.md` in your working directory.
5. Get your approval on the plan.
6. Hand off with an explicit block telling you which model to switch to and when.

## Install

Copy `idos/` into your Claude Code skills directory:

```bash
cp -r idos ~/.claude/skills/idos
```

Or with the [Skills CLI](https://github.com/anthropics/skills):

```bash
npx skills add <this-repo-url>
```

## Files

- `idos/SKILL.md` — the skill definition.
- `idos/plan-template.md` — the plan skeleton the skill fills in and writes to disk.

## License

MIT
