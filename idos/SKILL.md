---
name: idos
description: Ido's plan-first prompting workflow. Invoked with /idos <task>, or when Ido says "make a plan for X", "plan this properly before building", "write me a prompt/plan for X", or asks for planning before implementation. Picks the best planning model (Opus 5 or Fable 5), asks unlimited clarifying questions in rounds, discovers which skills the task needs and when each fires, writes the plan to a file, gets approval, then hands off to Sonnet 5 for the build.
---

# idos — Plan First, Build Later

Two-model workflow: **plan** on the strongest reasoning model, **build** on Sonnet 5.
This skill is the planning half. It always ends with a written plan file, an approval, and an explicit model-switch summary.

The task is whatever follows `/idos`. If nothing followed it, ask what to plan before doing anything else.

## Step 0 — Confirm the planning model

You already know which model you are — check that first, do not ask Ido.

Pick the model this task should be planned on:

- **Fable 5** (`claude-fable-5`) — fast, sharp. Well-scoped work, creative/UI/content tasks, single-surface features, anything where the shape is already clear and the job is a crisp plan quickly.
- **Opus 5** (`claude-opus-5`) — deepest reasoning. Ambiguous, architectural, multi-system or high-risk work: many moving parts, unclear requirements, real trade-offs, security or data-integrity concerns.

Then:
- **You are already the right one** → say so in one line and continue.
- **You are the other one of the two** → say which one fits better and why, and ask whether to switch or continue anyway. If Ido says continue, continue — do not stall.
- **You are neither (e.g. Sonnet)** → stop. Tell Ido to switch with `/model` to the recommended one and re-run `/idos`. Sonnet is the build model, not the planning model.

You cannot switch models yourself. Recommend and pause; never claim you switched.

## Step 1 — Ask everything (no limit)

Better questions now, better build later. There is **no cap** on questions.

Use `AskUserQuestion` for structured choices — **max 4 questions per call**, so run repeated rounds of up to 4 until nothing material is unresolved. Use plain text for open-ended things. Group related questions into the same round so Ido is not drip-fed.

Cover, as relevant:
- **Goal & success** — what "done" looks like, and how it gets verified.
- **Scope & non-goals** — explicitly in, explicitly out.
- **Constraints** — stack, existing code, style, deadlines, must-use / must-avoid tools.
- **Inputs & data** — sources, formats, examples, edge cases.
- **Audience & surface** — who uses it, on what (CLI, web, mobile, API).
- **Quality bar** — tests, accessibility, performance, security expectations.
- **Preferences** — design direction, terseness, immutability, anything Ido cares about.

Keep going until a competent stranger could build the right thing from the plan alone. Never guess on anything that changes the outcome.

## Step 2 — Map and source the skills

Go phase by phase (using the phase list you're about to lock in Step 3, or a rough mental pass if phases aren't final yet). For each phase, find the best skill coverage — installed first, external second. Do this for every phase, not just the ones that feel skill-shaped; a phase with no good skill is still worth one search before you conclude that.

1. **Check installed first.** Survey the session's available skills list. If one genuinely fits a phase, use it — no need to search further for that phase.
2. **Search external for gaps.** For any phase without solid installed coverage, invoke `find-skills` (or run `npx skills find <query>` directly) with a query specific to that phase — e.g. "stripe webhook", not just "payments". Check skills.sh's leaderboard for well-known options first.
3. **Verify before recommending.** Apply `find-skills`' quality bar: prefer 1K+ installs, be skeptical under 100; prefer reputable sources (`anthropics`, `vercel-labs`, and other well-known orgs); check the source repo isn't a ghost repo.
4. **No hit is a valid outcome.** If nothing external clears the bar, say so for that phase and proceed without one — don't force a weak match in just to fill the slot.

Only include skills the task genuinely benefits from — no padding for coverage's sake.

For each skill you land on, record: **which phase**, **why**, and **source** — `installed` or `external: <owner/repo@skill>`. External skills are not installed yet at this point; that happens in Step 4, after Ido approves the list.

## Step 3 — Write the plan to a file

Fill in `plan-template.md` (next to this file) and write the result to **`PLAN-<slug>.md`** in the working directory — `<slug>` being a short kebab-case name for the task. Never leave the plan as chat-only: the build session starts fresh and cannot see this conversation.

Match depth to the task. A plan longer than the work it describes is its own problem.

## Step 4 — Approval gate

Show Ido the plan (or a tight summary of it plus the file path), **and the external-skill install list from Step 2 as its own callout** — name, phase, install count/source, install command. Ask for approval on both. Apply any edits he asks for and rewrite the file. **Do not proceed until he approves.**

On approval:
- For each approved external skill, run `npx skills add <owner/repo@skill> -g -y`.
- If an install fails or the package doesn't exist as named, say so plainly — don't paper over it — and continue with the rest.
- Mark each skill in the plan's skill map as `installed` once done.

If Ido declines an external skill, drop it from the plan's skill map and proceed without it — don't substitute a different one without asking.

Do not start building here — the planning model plans, Sonnet builds.

## Step 5 — Handoff summary (mandatory, always last)

Once approved, end the turn with this block. It is not optional and never gets compressed away:

```
PLAN READY — PLAN-<slug>.md

Model now:      <Opus 5 | Fable 5>  — planning (done)
Switch to:      Sonnet 5  (/model → claude-sonnet-5)
Switch when:    now, before implementation starts
Then run:       read PLAN-<slug>.md and execute it
```

If the plan has phases that want a different model (e.g. a hard architectural phase mid-build), add one line per switch under `Switch when:` so Ido knows every model change up front — which model, at which phase, and why.

