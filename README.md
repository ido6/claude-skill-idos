# idos — Plan First, Build Later

A Claude Code skill implementing a two-model workflow: **plan** on the strongest reasoning
model (Opus 5 or Fable 5), **build** on Sonnet 5.

Invoke with `/idos <task>`. It will:

1. Confirm it's running on the right planning model (Opus 5 for ambiguous/architectural work,
   Fable 5 for well-scoped/creative work) — and tell you to switch if it isn't.
2. Grill you in frontier rounds — a design tree of questions with a recommended answer on
   each — hunting blind spots and unknown unknowns until the mission is fully understood.
3. Map which installed skills the task needs and when each one fires.
4. Write a full plan to `PLAN-<slug>.md` in your working directory.
5. Get your approval on the plan.
6. Launch the build itself as a Sonnet 5 subagent in a fresh context — zero manual
   steps, no transcript carried over (per-model prompt caches make a mid-session
   `/model` switch re-send everything) — then supervise it and report progress.

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
