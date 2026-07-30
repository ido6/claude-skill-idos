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

## Step 2 — Map the skills

Survey the session's available skills; use `find-skills` for anything not installed. Pick only what the task genuinely benefits from — no padding.

For each skill, record **which phase it fires in and why**:
- `python-testing` → test-writing phase, to hit the coverage bar.
- `security-review` → before any auth/input-handling code, and again before commit.
- `frontend-design` → UI-build phase, to avoid template-looking output.

If a needed skill is not installed, say so and give the install line (`npx skills add ...`) rather than silently skipping it.

## Step 3 — Write the plan to a file

Fill in `plan-template.md` (next to this file) and write the result to **`PLAN-<slug>.md`** in the working directory — `<slug>` being a short kebab-case name for the task. Never leave the plan as chat-only: the build session starts fresh and cannot see this conversation.

Match depth to the task. A plan longer than the work it describes is its own problem.

## Step 4 — Approval gate

Show Ido the plan (or a tight summary of it plus the file path) and ask for approval. Apply any edits he asks for and rewrite the file. **Do not proceed to handoff until he approves.** Do not start building here — the planning model plans, Sonnet builds.

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

