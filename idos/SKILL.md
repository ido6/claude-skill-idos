---
name: idos
description: Ido's plan-first workflow. /idos <task>, or "make a plan for X" / "plan this before building". Grills for blind spots in short rounds, sources skills/connectors, writes PLAN-<slug>.md, gets approval, then runs the build as the idos-builder Sonnet subagent and supervises it.
---

# idos — Plan First, Build Later

Plan on the model this chat runs (Fable 5.1 or Opus 5); build on Sonnet 5 via the `idos-builder` subagent. This skill is the planning half. It ends with a plan file, an approval, a launched build, and a `BUILD DONE` report.

The task is whatever follows `/idos`. If nothing followed it, ask what to plan.

## Token discipline (applies to every step)

Every byte pulled into this context is re-read on every later turn, and a planning run is 50–80 turns. So:

- **Read the repo through the `Explore` subagent**, one call with a precise brief, one report back. Never `cat` whole files into this context. Read a file here only if you will quote it in the plan — and then only the lines you need.
- **Batch independent lookups into one tool call.** Ten sequential one-liners cost ten turns of context.
- The project `CLAUDE.md` is already in context. Do not read it again.
- **Everything you say to Ido is super-caveman.** Status lines ≤ 1 line. Explanations ≤ 3 fragment lines. No "plan in one breath" paragraphs, no recaps, no rationale. The plan file is the only place for normal prose.

## Step 0 — Triage

One file, one obvious change → say *"one-file change, `/idos` is overhead — want me to just do it?"* and wait. Otherwise continue.

## Step 0b — Note the model

- **Fable 5.1 or Opus 5** → say which in one line and continue. If the other fits better, say so in that line. Do not stall.
- **Sonnet or smaller** → **stop and wait.** One line: *"On Sonnet. Switch to Fable 5.1 or Opus 5 in the model picker, then say go."* Do nothing until he answers. If he says "continue anyway", plan on Sonnet with a one-line warning. The transcript is tiny at this point, so his switch costs nothing.

You cannot switch models yourself; only Ido can, and only here at the start. The build never needs a switch — `idos-builder` is pinned to Sonnet by its own definition.

## Step 1 — Read the ground, then grill for blind spots

**Facts are your job; decisions are Ido's. Never ask what the repo can tell you.**

Before the first question, via `Explore`: entry points and the files the task names, real stack (`package.json` etc.), test setup, conventions. Plus, already-settled decisions you must cite instead of re-asking:
- the auto-memory index (`~/.claude/projects/<project-slug>/memory/MEMORY.md`) and the `CLAUDE.md` gotchas already in context;
- existing `PLAN-*.md` in the repo for the same area — grep for the area, read section 2 of the hits only;
- git state: worktree or main checkout, foreign hunks from another session.

If a question mid-grilling needs a fact, look it up (or send `Explore`) — don't ask Ido, and don't block the round; only downstream questions wait.

**Then grill.** The goal is blind spots and unknown unknowns, not a requirements list. Assume the first framing is incomplete.

Work a **design tree in frontier rounds**: each round asks every question whose prerequisites are settled, with your recommended answer on each; wait; recompute the frontier from the answers; repeat. No cap on rounds. A question that depends on another still open this round waits for the next.

**Super-caveman questions — hard caps:** question ≤ 10 words, one decision. Header 1–2 words. Options ≤ 4 words; option description ≤ 8 words or none. Recommended option first, tagged `(Recommended)`, nothing more. Example: *"Grid tap → open expanded view?"* with *Yes, everywhere (Recommended) / Phone only / Keep as is*. Setup a question would need is your homework, not question text. Use `AskUserQuestion` (max 4 per call) for choices; plain text only for truly open-ended, and even then one short line. "You decide" or a skipped question → take the recommendation, record it as *assumed*.

Known unknowns to start from:
- **Goal & success** — what done looks like, how it is verified.
- **Scope & non-goals.**
- **Constraints** — deadlines, must-use / must-avoid tools.
- **Inputs & data** — sources, formats, edge cases.
- **Audience & surface.**
- **Quality bar + real gate** — tests, a11y, perf, security; on idoGen the gate is `pnpm build`, not `tsc`.
- **Language & direction** — Hebrew-first RTL client sites; idoGen mixes Hebrew and English. Any UI phase says which.
- **Paid calls** — Gemini, WaveSpeed, fal, ElevenLabs, OpenAI. Default: none during the build; cap any exception.
- **Delivery** — commit only (default), `ship.ps1`, or push. On git-linked Vercel repos push = production deploy. The builder never pushes.
- **Preferences** — design direction, terseness.

Unknown unknowns to hunt: unstated assumptions (name each), failure modes that pass technically and fail the mission, second-order effects, what a domain expert would expect to hear and didn't, contradictions (repo vs Ido, Ido vs Ido — raise, never pick silently).

Done when the frontier is empty and a competent stranger could build the right thing from the plan alone. Confirm the shared understanding, then move on.

**Persist as you go.** Create the plan draft at the first settled answers; append each round's decisions immediately. After a compaction, read the draft and resume from its `Status:` — never restart the grilling.

Close by sketching the ordered phase list (names only). Step 2 needs it.

## Step 2 — Map capabilities per phase

Three kinds: **skills**, **connectors** (MCP), **plugins**. Skills are checked for every phase; connectors and plugins only when a phase needs them. Never force a weak match — "nothing good found" is a valid entry. Nothing is installed until Step 4.

**2a Skills.** Installed first. Then `find-skills` / `npx skills find <specific query>`. Before recommending an external one: prefer 1K+ installs and known orgs, and **read its SKILL.md** — frontmatter plus a scan of bundled scripts for anything you would refuse to run (exfil, credential access, destructive deletes). Use `skill-scout` if installed. Record phase, why, source.

**2b Connectors** — only if a phase reads or writes live data in an external service. Check what is connected; search the registry (`mcp__mcp-registry__search_mcp_registry` via ToolSearch) or GitHub for gaps. You cannot authorize any of them. Record `connected` / `needs auth` / `not installed`.

**2c Plugins** — only if a phase wants a whole domain toolkit. `claude plugin list`, `claude plugin marketplace list|add`, `claude plugin install <plugin>@<marketplace>`, and `claude plugin details <name>` for the **projected token cost** — recommend only if the value clears it.

## Step 3 — Write the plan

Fill `plan-template.md` (next to this file) → `PLAN-<slug>.md` in the project directory the task is about (next to the code, in the current worktree). If one exists, read it and ask update-or-fresh; never overwrite silently. Record the absolute path.

Normal prose, not caveman. If `/ponytail` is stacked, the plan obeys it: fewest phases, stdlib first, nothing speculative. Match depth to the task — a plan longer than the work is its own problem.

## Step 4 — Approval gate

Show the path plus **≤ 5 caveman fragment lines** (phases in order, one clause each) and each Step 2 addition as one line (skill: name, phase, source; connector: auth state; plugin: install command, token cost). Ask for approval on the plan and each addition via `AskUserQuestion`. Apply edits. **Do not proceed until he approves.**

On approval: skills via `npx skills add <owner/repo@skill> -g -y` (check `npx` exists first); plugins via `claude plugin install`; connectors are handed to Ido with the exact step — OAuth cannot run here. Report any failed install plainly and mark real status in the capability map. Declined → drop it, don't substitute. A phase blocked on an unauthorized connector is named in the launch block.

## Step 5 — Launch the build (mandatory, always last)

Spawn the **`idos-builder`** subagent (`~/.claude/agents/idos-builder.md`, `model: sonnet` — guaranteed by Claude Code's resolution order). If the definition is missing, copy `idos-builder.md` from this skill's directory to `~/.claude/agents/` first. It starts with a fresh context: the planning transcript never reaches it, and no model switch happens anywhere.

Print this block first:

```
BUILD LAUNCHED — idos-builder subagent (Sonnet 5), fresh context

Plan:        <absolute path to PLAN-<slug>.md>
Running:     in the foreground — this turn stays busy until BUILD DONE.
             Do not close or stop the chat; Esc kills the builder mid-run.
             The gate (<real gate>) alone can take minutes.
Blocked on:  <unauthorized connectors + which phase stalls, or "nothing">
```

Then spawn **in the foreground** (`run_in_background: false`) — a background spawn ends your turn, leaves an idle-looking prompt, and invites a chat close that kills the builder. Prompt is only the pointer:

```
Execute the plan at <absolute path to PLAN-<slug>.md>.
```

Phases wanting a different model run as their own subagent with that override, plan re-read included.

**When the builder returns:**
1. Check its `Skills loaded` line against the capability map. A listed skill it never loaded → relaunch that phase alone: "re-check Phase N against `<skill>`; load it first".
2. Spawn `code-reviewer` on the diff (subagents cannot spawn subagents — this is your job). CRITICAL/HIGH go back to the builder through the plan file; MEDIUM and below are listed.
3. Close with:

```
BUILD DONE — <slug>

Model:        build ran on Sonnet 5 (idos-builder subagent); planner idle meanwhile
Built:        <one line per phase>
Verified:     <real gate + each Verify checkpoint, pass/fail>
Not verified: <what could not be checked, and why — never empty for looks>
Skills:       <loaded per phase, or "missed: <skill> in Phase N — re-checked">
Review:       <code-reviewer verdict; open MEDIUM/LOW>
Git:          <committed as <sha> | uncommitted | pushed — which files>
Open:         <blockers, section-2 deviations, anything for Ido>
```

Corrections after that flow through the plan file, then a relaunch. You never write code.

**Fallback — Agent tool unavailable:** one line for Ido, nothing else: new terminal → `claude --model claude-sonnet-5 "read <absolute path> and execute it"`.

## When the build hits a wall

Shared state between planner and builder is only `PLAN-<slug>.md`. Stall, blocker, or reality disagreeing with the plan → fix the plan (section 2 + `Status: revised`), relaunch. Re-run `/idos` only when the shape of the work changes.

**Interrupted build** (Esc, chat closed, no BUILD DONE): edits are on disk, `Progress:` says which phases finished. **Resume in the same chat** — each chat owns its worktree; a new chat cannot see the uncommitted edits and would re-plan. On "resume", skip Steps 0–4 and relaunch foreground with `Resume the plan at <absolute path> from its Progress line.` If that chat is truly gone, start `claude` inside that same worktree folder.

For a lightweight think-then-handoff without rounds, sourcing, or gates, Ido has `/planhandoff` — never both on one task.
