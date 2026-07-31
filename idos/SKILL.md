---
name: idos
description: Ido's plan-first prompting workflow. Invoked with /idos <task>, or when Ido says "make a plan for X", "plan this properly before building", "write me a prompt/plan for X", or asks for planning before implementation. Picks the best planning model (Opus 5 or Fable 5), asks unlimited clarifying questions in rounds, discovers and installs the skills — plus any connectors or plugins — the task needs and when each fires, writes the plan to a file, gets approval, then hands off to Sonnet 5 for the build.
---

# idos — Plan First, Build Later

Two-model workflow: **plan** on the strongest reasoning model, **build** on Sonnet 5.
This skill is the planning half. It always ends with a written plan file, an approval, and an explicit model-switch summary.

The task is whatever follows `/idos`. If nothing followed it, ask what to plan before doing anything else.

## Step 0 — Triage: is this worth planning?

Run this test first, before anything else.

**If the task touches a single file and has one obvious correct change — stop.** Say so in one line: *"This is a one-file change, `/idos` is overhead here — want me to just do it?"* Then wait. Planning a typo fix costs more than the fix.

Everything below runs only if the task fails that test — more than one file, a real decision to make, or an unclear shape.

## Step 0b — Confirm the planning model

You already know which model you are — check that first, do not ask Ido.

Pick the model this task should be planned on:

- **Fable 5** (`claude-fable-5`) — fast, sharp. Well-scoped work, creative/UI/content tasks, single-surface features, anything where the shape is already clear and the job is a crisp plan quickly.
- **Opus 5** (`claude-opus-5`) — deepest reasoning. Ambiguous, architectural, multi-system or high-risk work: many moving parts, unclear requirements, real trade-offs, security or data-integrity concerns.

Then:
- **You are already the right one** → say so in one line and continue.
- **You are the other one of the two** → say which one fits better and why, and ask whether to switch or continue anyway. If Ido says continue, continue — do not stall.
- **You are neither (e.g. Sonnet)** → stop. Tell Ido to switch with `/model` to the recommended one and re-run `/idos`. Sonnet is the build model, not the planning model.

You cannot switch models yourself. Recommend and pause; never claim you switched.

## Step 1 — Read the ground, then ask everything

**Read first. Never ask Ido what the repo can tell you.**

If the task touches existing code, orient before asking a single question: list the working directory, read the entry points and the files the task names, check `package.json` / `pyproject.toml` / equivalent for the real stack, skim the config and test setup, note the conventions actually in use. Use `Explore` for anything that needs a broad sweep. Questions about stack, structure, or existing style are answers you should already have.

**Then ask everything that's left.** There is **no cap** on questions.

Use `AskUserQuestion` for structured choices — **max 4 questions per call**, so run repeated rounds of up to 4 until nothing material is unresolved. Use plain text for open-ended things. Group related questions into the same round so Ido is not drip-fed.

Cover what the repo can't answer:
- **Goal & success** — what "done" looks like, and how it gets verified.
- **Scope & non-goals** — explicitly in, explicitly out.
- **Constraints** — deadlines, must-use / must-avoid tools, anything the code doesn't reveal.
- **Inputs & data** — sources, formats, examples, edge cases.
- **Audience & surface** — who uses it, on what (CLI, web, mobile, API).
- **Quality bar** — tests, accessibility, performance, security expectations.
- **Preferences** — design direction, terseness, anything Ido cares about.

If reading the repo contradicts something Ido said, raise it rather than silently picking one.

Keep going until a competent stranger could build the right thing from the plan alone. Never guess on anything that changes the outcome.

**Close Step 1 by sketching the phase list** — just the ordered phase names, no detail yet. Step 2 maps capabilities onto these phases, so they have to exist first.

## Step 2 — Map and source the capabilities

Three kinds of capability can cover a phase: **skills** (knowledge/workflow), **connectors** (MCP servers — live access to an external service's data), and **plugins** (bundles of skills/commands/agents/MCP for a whole domain).

Walk the phase list from Step 1. Skills get checked for every phase; connectors and plugins only when their trigger fires.

**One rule governs all three: never force a weak match in to fill a slot.** "Nothing good found for this phase" is a valid, honest outcome — record it and move on.

Nothing here gets installed yet. That happens in Step 4, after Ido approves.

### 2a — Skills (every phase)

1. **Installed first.** Survey the session's skills list. A genuine fit ends the search for that phase.
2. **Search the gaps.** Invoke `find-skills` (or `npx skills find <query>`) with a query specific to the phase — "stripe webhook", not "payments". Check the skills.sh leaderboard for well-known options first.
3. **Verify before recommending.** Prefer 1K+ installs and reputable sources (`anthropics`, `vercel-labs`, and other known orgs); be skeptical under 100 installs or a ghost source repo.

Record: phase, why, and source — `installed` or `external: <owner/repo@skill>`.

### 2b — Connectors (only if a phase needs live external data)

**Trigger:** a phase has to read or write real data in an external service — Figma files, Linear/Jira issues, Notion pages, a Supabase/Postgres database, Slack, Vercel deployments, Sentry errors. If every phase is self-contained in the repo, skip this entirely.

1. **Check what's already connected.** The session lists its MCP servers, including which are connected, still connecting, and which need authentication. An already-connected server needs nothing.
2. **Search for gaps.** Load the registry tools via `ToolSearch` (`mcp__mcp-registry__search_mcp_registry`, `suggest_connectors`, `list_connectors`) and search for the service the phase needs.
3. **Record the auth reality.** Most connectors need OAuth or an API token. **You cannot authorize them** — the user does that in their claude.ai connector settings, or via `claude mcp add ...` / `/mcp` in an interactive session. Note per connector whether it's already authorized, needs auth, or needs installing from scratch.

Record: which phase, why, and status — `connected` / `needs auth` / `not installed`.

### 2c — Plugins (only if a phase wants a whole domain toolkit)

**Trigger:** a phase would pull in several related skills/commands/agents at once, or a vendor ships an official plugin for exactly this stack. One skill's worth of need is a skill, not a plugin — don't reach for a plugin to solve a single problem.

- `claude plugin list` — what's already installed.
- `claude plugin marketplace list` — configured marketplaces. Add one with `claude plugin marketplace add <github-repo-or-url>`.
- `claude plugin install <plugin>@<marketplace>` — the install command to hand over.
- `claude plugin details <name>` — component inventory and **projected token cost**. Check this before recommending: a plugin that loads a large surface into every session is a real cost, so only recommend one whose value clears it.

Record: which phase, why, install command, and token cost from `details`.

## Step 3 — Write the plan to a file

Fill in `plan-template.md` (next to this file) and write the result to **`PLAN-<slug>.md`** — `<slug>` being a short kebab-case name for the task.

- **Where:** the project directory the task is about, not wherever the session happens to have started. If those differ, write it next to the code.
- **Already exists?** Read it first and ask whether to update it or start fresh. Never silently overwrite a plan.
- **Record the absolute path.** Steps 4 and 5 both need it — a bare filename won't resolve if the build session starts in a different directory.

Never leave the plan as chat-only: the build session starts fresh and cannot see this conversation.

Match depth to the task. A plan longer than the work it describes is its own problem.

## Step 4 — Approval gate

Show Ido the plan (or a tight summary of it plus the file path), **and everything Step 2 wants to add as its own callout**, split into what you can do and what only he can:

- **Skills to install** — name, phase, install count/source.
- **Connectors** — name, phase, and whether it needs auth.
- **Plugins** — name, phase, install command, projected token cost.

Ask for approval on the plan and on each addition. Apply any edits he asks for and rewrite the file. **Do not proceed until he approves.**

On approval:
- **Skills** — run `npx skills add <owner/repo@skill> -g -y` for each approved one.
- **Plugins** — run `claude plugin install <plugin>@<marketplace>` (adding the marketplace first if needed).
- **Connectors** — **hand these to Ido, don't attempt them.** Give the exact step: claude.ai connector settings for claude.ai connectors, or `claude mcp add ...` / `/mcp` in an interactive session for the rest. OAuth cannot run from inside this session, so a connector is only ready when he says it is.

If an install fails or the package doesn't exist as named, say so plainly — don't paper over it — and continue with the rest. Mark each entry's real status in the plan's capability map.

If Ido declines something, drop it from the map and proceed without it — don't substitute a different one without asking.

**If a phase depends on a connector that isn't authorized yet, say so in the handoff** — the build session will hit a wall there otherwise.

Do not start building here — the planning model plans, Sonnet builds.

## Step 5 — Handoff summary (mandatory, always last)

Once approved, end the turn with this block. It is not optional and never gets compressed away:

```
PLAN READY — <absolute path to PLAN-<slug>.md>

Model now:      <Opus 5 | Fable 5>  — planning (done)
Switch to:      Sonnet 5  (/model → claude-sonnet-5)
Switch when:    now, before implementation starts
Then run:       read <absolute path> and execute it
Blocked on:     <unauthorized connectors + which phase stalls, or "nothing">
```

Use the **absolute** path in both lines — the build session may start somewhere else entirely.

If the plan has phases that want a different model (e.g. a hard architectural phase mid-build), add one line per switch under `Switch when:` so Ido knows every model change up front — which model, at which phase, and why.

