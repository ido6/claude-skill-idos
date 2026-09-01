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
6. Launch the build itself via the bundled `idos-builder` subagent — its agent
   definition pins `model: sonnet`, which Claude Code honors regardless of Task-tool
   parameters (only the *main* conversation's model is user-switch-only). Fresh
   context, zero manual steps, no transcript carried over; the planner supervises.

## Install

Copy `idos/` into your Claude Code skills directory, and the build-agent definition
into your agents directory:

```bash
cp -r idos ~/.claude/skills/idos
cp idos/idos-builder.md ~/.claude/agents/idos-builder.md
```

(The skill self-heals the second step: if the agent definition is missing at build
time, it copies it from the skill directory before spawning.)

Or with the [Skills CLI](https://github.com/anthropics/skills):

```bash
npx skills add <this-repo-url>
```

## Files

- `idos/SKILL.md` — the skill definition.
- `idos/plan-template.md` — the plan skeleton the skill fills in and writes to disk.

## License

MIT
