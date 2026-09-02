# idos — Plan First, Build Later

A Claude Code skill implementing a two-model workflow: **plan** on a strong reasoning
model (Fable 5.1 or Opus 5), **build** on Sonnet 5 — with no model switching anywhere.
The build model is pinned by a bundled subagent definition; the planning model is
whatever the chat started on.

Invoke with `/idos <task>`. It will:

1. **Triage.** A one-file, one-obvious-change task gets "this is overhead, want me to
   just do it?" — planning a typo fix costs more than the fix.
2. **Note the model.** Fable 5.1 / Opus 5 continue; Sonnet warns the plan will be
   shallower and continues anyway. It never asks you to switch.
3. **Read the ground.** Repo, entry points, stack, project `CLAUDE.md` gotchas,
   auto-memory, prior `PLAN-*.md` files in the same area, git state (worktree? foreign
   hunks?). Facts are its job; decisions are yours.
4. **Grill for blind spots.** A design tree worked in frontier rounds: every askable
   question at once, a recommended answer on each, super-caveman (≤ 10 words, one
   decision, options ≤ 4 words). It hunts unstated assumptions, failure modes, second-order
   effects and contradictions until nothing is silently assumed. Answers are persisted
   to the plan draft as they land, so compaction cannot eat them.
5. **Map capabilities.** Installed skills first, then skills.sh for gaps (vetted before
   recommending), connectors and plugins only when a phase needs them. "Nothing good
   found" is a valid answer.
6. **Write the plan** to `PLAN-<slug>.md` next to the code, from `plan-template.md`,
   and get your approval. The template asks for the project's real build gate, a
   verification harness, a paid-API-call budget and a ship decision.
7. **Launch the build** as the bundled `idos-builder` subagent (`model: sonnet`), in the
   foreground, with a fresh context — the planning transcript never reaches it. When
   it returns, the planner runs `code-reviewer` on the diff, bounces CRITICAL/HIGH
   findings back through the plan file, and closes with a `BUILD DONE` block:
   built / verified / not verified / review / git state / open items.

The builder rewrites the plan's `Progress:` line after every phase. If a run is
interrupted, say "resume" **in the same chat** — it relaunches from that line instead of
redoing the work. (Each chat owns its own worktree; a new chat cannot see the
uncommitted edits.)

## Install

Copy `idos/` into your Claude Code skills directory, and the build-agent definition
into your agents directory:

```bash
cp -r idos ~/.claude/skills/idos
cp idos/idos-builder.md ~/.claude/agents/idos-builder.md
```

(The skill self-heals the second step: if the agent definition is missing at build
time, it copies it from the skill directory before spawning.)

## Files

- `idos/SKILL.md` — the skill definition.
- `idos/plan-template.md` — the plan skeleton the skill fills in and writes to disk.
- `idos/idos-builder.md` — the Sonnet 5 build subagent (goes in `~/.claude/agents/`).

## Opinionated by design

Built for Claude Code CLI and one person's setup. The builder never pushes (on
git-linked Vercel repos a push is a production deploy), makes no paid API calls unless
the plan allows them, stages only its own hunks when other sessions share the repo,
runs the project's real gate (`pnpm build`, not just `tsc`) before reporting, and
loads each phase's listed skills with the Skill tool before its first edit.

## License

MIT
