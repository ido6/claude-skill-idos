---
name: idos
description: Ido's plan-first prompting workflow. Invoked with /idos <task>, or when Ido says "make a plan for X", "plan this properly before building", "write me a prompt/plan for X", or asks for planning before implementation. Picks the best planning model (Opus 5 or Fable 5.1), grills in frontier rounds for blind spots and unknown unknowns until the mission is fully understood, discovers and installs the skills — plus any connectors or plugins — the task needs and when each fires, writes the plan to a file, gets approval, then launches the build as a fresh-context Sonnet 5 subagent and supervises it.
---

# idos — Plan First, Build Later

Two-model workflow: **plan** on the strongest reasoning model, **build** on Sonnet 5.
This skill is the planning half. It always ends with a written plan file, an approval, and the build launched as a fresh-context Sonnet 5 subagent.

The task is whatever follows `/idos`. If nothing followed it, ask what to plan before doing anything else.

## Step 0 — Triage: is this worth planning?

Run this test first, before anything else.

**If the task touches a single file and has one obvious correct change — stop.** Say so in one line: *"This is a one-file change, `/idos` is overhead here — want me to just do it?"* Then wait. Planning a typo fix costs more than the fix.

Everything below runs only if the task fails that test — more than one file, a real decision to make, or an unclear shape.

## Step 0b — Confirm the planning model

You already know which model you are — check that first, do not ask Ido.

Pick the model this task should be planned on:

- **Fable 5.1** (`claude-fable-5-1`; Fable 5 `claude-fable-5` also fine) — fast, sharp, Ido's default. Well-scoped work, creative/UI/content tasks, single-surface features, anything where the shape is already clear and the job is a crisp plan quickly.
- **Opus 5** (`claude-opus-5`) — deepest reasoning. Ambiguous, architectural, multi-system or high-risk work: many moving parts, unclear requirements, real trade-offs, security or data-integrity concerns.

Then:
- **You are Fable 5.1 or Opus 5** → say which in one line and continue. If the other would fit better, say so in that same line — Ido can restart on it from his model picker if he wants. Do not stall waiting for a switch.
- **You are Sonnet (or smaller)** → say so once, warn that the plan will be shallower, and continue anyway. Sonnet is the build model, not the planning model — but a Sonnet plan beats no plan, and a mid-session switch may not be possible.

You cannot switch models yourself, and nothing in this skill depends on one. Never tell Ido to run `/model` or any slash command; never claim you switched.

## Step 1 — Read the ground, then grill for blind spots

**Read first. Never ask Ido what the repo can tell you.** Finding *facts* is your job, never his — the *decisions* are his.

If the task touches existing code, orient before asking a single question: list the working directory, read the entry points and the files the task names, check `package.json` / `pyproject.toml` / equivalent for the real stack, skim the config and test setup, note the conventions actually in use. Use the `Explore` subagent for anything that needs a broad sweep. Questions about stack, structure, or existing style are answers you should already have. If mid-grilling a question turns out to need a fact from the environment, go look it up (or dispatch the subagent) instead of asking Ido — and don't block the round on it: only the questions downstream of that fact wait.

**Ido's repos carry decisions already made — read those before asking anything twice:**
- The project's `CLAUDE.md` gotchas and the auto-memory index for this project (`~/.claude/projects/<project-slug>/memory/MEMORY.md`). A rule recorded there is settled; cite it, don't re-ask it.
- Existing `PLAN-*.md` files in the repo touching the same area. idoGen alone has 20+; their section 2 decisions still hold unless the code has moved on. Reuse, don't re-grill.
- Git state: are you in a `.claude/worktrees/*` checkout or the main one, and does `git status` show foreign hunks from another session? Both change how the build must stage and where the plan lives.

**Then grill.** The goal is not to collect requirements — it is to surface **blind spots and unknown unknowns** until the mission is fully understood by both of you. Assume the first framing of the task is incomplete; your job is to find what Ido hasn't thought to say.

Map the interrogation as a **design tree**: every decision branches into the decisions that hang off it. Work the tree in **rounds**. The **frontier** is every question whose prerequisites are already settled — askable *now* without guessing at answers you haven't heard yet. Each round:

1. Ask the whole frontier at once. There is **no cap** on questions or rounds.
2. Attach your **recommended answer** to every question — never make Ido pick blind.
3. Wait for his answers. A question whose answer depends on another question still open this round belongs to a *later* round, not this one.
4. His answers reshape the tree: settled decisions push the frontier outward and unblock new questions. Recompute and go again.

**Keep every question short and plain.** One decision per question, one or two sentences, everyday words. If a question needs a paragraph of setup, the setup is your homework — do the reading, then ask the short version. Answer options and recommendations get one line each. A question Ido has to reread has already failed.

Use `AskUserQuestion` for structured choices — **max 4 questions per call**, recommended option listed first — and plain text for open-ended things. Group related questions into the same round so Ido is not drip-fed. Ido prefers defaults already decided: if he answers "you decide" or skips a question, take your recommended answer and record it in the plan as *assumed*, not *decided*.

Start from the known unknowns the repo can't answer:
- **Goal & success** — what "done" looks like, and how it gets verified.
- **Scope & non-goals** — explicitly in, explicitly out.
- **Constraints** — deadlines, must-use / must-avoid tools, anything the code doesn't reveal.
- **Inputs & data** — sources, formats, examples, edge cases.
- **Audience & surface** — who uses it, on what (CLI, web, mobile, API).
- **Quality bar** — tests, accessibility, performance, security expectations — and the project's **real gate** (on idoGen that is `pnpm build`; `tsc` clean has shipped broken code there five times).
- **Language & direction** — Ido's client sites are Hebrew-first RTL; idoGen mixes Hebrew and English. Any UI phase must say which, and whether `dir`/bidi is in play.
- **Paid calls** — will building or verifying spend real money (Gemini, WaveSpeed, fal, ElevenLabs, OpenAI)? Default is **no paid calls during the build**; if verification needs one, cap it explicitly.
- **Delivery** — commit only, `ship.ps1`, or push? On git-linked Vercel repos (idoGen) **`git push` is a production deploy**. Default is commit-only; the builder never pushes.
- **Preferences** — design direction, terseness, anything Ido cares about.

Then hunt the unknown unknowns — the questions Ido didn't know needed asking:
- **Unstated assumptions** — what is the request silently taking for granted? Name each one and ask.
- **Failure modes** — what would make this succeed technically and still fail the mission?
- **Second-order effects** — what does this change break, complicate, or obligate later?
- **The unsaid** — what did the request leave out that a domain expert would expect to hear?
- **Contradictions** — repo vs. Ido, or Ido's own answers vs. each other: raise them, never silently pick one.

The grilling is done only when the **frontier is empty**: every branch of the tree visited, nothing left silently assumed, and a competent stranger could build the right thing from the plan alone. Confirm the shared understanding of the mission with Ido before moving on. Never guess on anything that changes the outcome.

**Persist answers as they land.** Create the plan draft file as soon as the first answers settle (Step 3 finalizes it) and append each round of decisions to it immediately. A long Q&A thread is exactly what context compaction eats — and re-asking Ido questions he already answered wastes his time. The file remembers; the transcript may not. **If you find yourself mid-`/idos` after a compaction, read the draft first and resume from its `Status:` line — never restart the grilling.**

**Close Step 1 by sketching the phase list** — just the ordered phase names, no detail yet. Step 2 maps capabilities onto these phases, so they have to exist first.

## Step 2 — Map and source the capabilities

Three kinds of capability can cover a phase: **skills** (knowledge/workflow), **connectors** (MCP servers — live access to an external service's data), and **plugins** (bundles of skills/commands/agents/MCP for a whole domain).

Walk the phase list from Step 1. Skills get checked for every phase; connectors and plugins only when their trigger fires.

**One rule governs all three: never force a weak match in to fill a slot.** "Nothing good found for this phase" is a valid, honest outcome — record it and move on.

Nothing here gets installed yet. That happens in Step 4, after Ido approves.

### 2a — Skills (every phase)

1. **Installed first.** Survey the session's skills list. A genuine fit ends the search for that phase.
2. **Search the gaps.** Invoke `find-skills` (or `npx skills find <query>`) with a query specific to the phase — "stripe webhook", not "payments". Check the skills.sh leaderboard for well-known options first.
3. **Verify before recommending.** Prefer 1K+ installs and reputable sources (`anthropics`, `vercel-labs`, and other known orgs); be skeptical under 100 installs or a ghost source repo. Popularity is not safety — **read the external SKILL.md before recommending it**: frontmatter plus a scan of any bundled scripts/commands for things you would refuse to run yourself (network exfil, credential access, destructive deletes). If the `skill-scout` skill is installed, run its vetting step instead of improvising one.

Record: phase, why, and source — `installed` or `external: <owner/repo@skill>`.

### 2b — Connectors (only if a phase needs live external data)

**Trigger:** a phase has to read or write real data in an external service — Figma files, Linear/Jira issues, Notion pages, a Supabase/Postgres database, Slack, Vercel deployments, Sentry errors. If every phase is self-contained in the repo, skip this entirely.

1. **Check what's already connected.** The session lists its MCP servers, including which are connected, still connecting, and which need authentication. An already-connected server needs nothing.
2. **Search for gaps.** If the session exposes connector-registry search tools (`mcp__mcp-registry__search_mcp_registry` via ToolSearch), use them to find the service the phase needs; otherwise search GitHub for `<service> mcp server`. Either way, record what you found: Ido does the actual connector install by hand (Step 4).
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

Write the plan in **normal prose, not caveman** — Sonnet reads it cold, and the caveman rule already exempts multi-step sequences. Chat stays caveman. If Ido stacked `/ponytail` (or any minimalism mode) onto this run, the plan obeys it: fewest phases, stdlib before dependencies, nothing speculative.

Match depth to the task. A plan longer than the work it describes is its own problem.

## Step 4 — Approval gate

Show Ido the plan (or a tight summary of it plus the file path), **and everything Step 2 wants to add as its own callout**, split into what you can do and what only he can:

- **Skills to install** — name, phase, install count/source.
- **Connectors** — name, phase, and whether it needs auth.
- **Plugins** — name, phase, install command, projected token cost.

Ask for approval on the plan and on each addition. Apply any edits he asks for and rewrite the file. **Do not proceed until he approves.**

On approval:
- **Skills** — run `npx skills add <owner/repo@skill> -g -y` for each approved one. Confirm `npx` exists first (`Get-Command npx` / `command -v npx`); if it's missing, hand Ido the exact command instead of failing silently.
- **Plugins** — run `claude plugin install <plugin>@<marketplace>` (adding the marketplace first if needed).
- **Connectors** — **hand these to Ido, don't attempt them.** Give the exact step: claude.ai connector settings for claude.ai connectors, or `claude mcp add ...` / `/mcp` in an interactive session for the rest. OAuth cannot run from inside this session, so a connector is only ready when he says it is.

If an install fails or the package doesn't exist as named, say so plainly — don't paper over it — and continue with the rest. Mark each entry's real status in the plan's capability map.

If Ido declines something, drop it from the map and proceed without it — don't substitute a different one without asking.

**If a phase depends on a connector that isn't authorized yet, say so in the handoff** — the build session will hit a wall there otherwise.

Do not start building here — the planning model plans, Sonnet builds.

## Step 5 — Launch the build (mandatory, always last)

The planner never writes code — but it does launch the builder. On approval, spawn the **`idos-builder`** subagent. Its definition lives at `~/.claude/agents/idos-builder.md` with `model: sonnet` in the frontmatter — Claude Code's model-resolution order honors the agent definition's model even when the Agent/Task tool exposes no per-invocation `model` parameter, so the Sonnet override is guaranteed. **If the definition is missing, copy `idos-builder.md` from this skill's directory to `~/.claude/agents/` first, then spawn.** (Only the user can switch the *main* conversation's model; subagent models are the sanctioned exception.)

A subagent starts with a **fresh context by construction**: it sees only its prompt, not the planning transcript. That kills the token waste automatically — nothing for Ido to type, no slash command, no model switch in the main conversation. (Only Ido could switch that model, and doing so mid-session would re-send the whole transcript as uncached input on the new model, since prompt caches are per-model; the plan file already carries everything the build needs.)

First print this block, so Ido knows what is about to happen and that the turn will stay busy:

```
BUILD LAUNCHED — idos-builder subagent (Sonnet 5), fresh context

Plan:        <absolute path to PLAN-<slug>.md>
Running:     in the foreground — this turn stays busy until BUILD DONE.
             Do not close or stop the chat; Esc kills the builder mid-run.
             The gate (`pnpm build`) alone can take minutes.
Blocked on:  <unauthorized connectors + which phase stalls, or "nothing">
```

Then spawn it **in the foreground** (`run_in_background: false`). Your very next action depends on its result and nothing useful happens meanwhile — and a background spawn ends your turn, leaves an idle prompt that looks finished, and invites Ido to close the chat, which kills the builder (that is exactly how the first real run died at the `pnpm build` gate). The agent definition already carries the execution rules, so the prompt is only the pointer:

```
Execute the plan at <absolute path to PLAN-<slug>.md>.
```

You are the supervisor, not the builder: when the subagent returns, read its report against the plan, surface blockers, and push corrections through the plan file — edit it, then relaunch against the revised plan. Still no code from you.

If the plan has phases that want a different model (e.g. a hard architectural phase mid-build), launch that phase as its own subagent with that model override, plan file re-read included — the plan (section 2 decisions plus phase checkpoints) is the only state that crosses subagent boundaries.

**When the builder reports done, review before you report.** Spawn the `code-reviewer` agent (`~/.claude/agents/code-reviewer.md`, Ido's ECC rule: every code change gets reviewed) on the build's diff — subagents cannot spawn subagents, so this is your job. CRITICAL or HIGH findings go back to the builder through the plan file; MEDIUM and below get listed, not fixed. Then close with:

```
BUILD DONE — <slug>

Built:       <one line per phase, what changed>
Verified:    <the real gate + each Verify checkpoint, pass/fail>
Not verified: <anything the plan wanted checked that could not be — say why>
Review:      <code-reviewer verdict; open MEDIUM/LOW items>
Git:         <committed as <sha> | uncommitted | pushed — and which files>
Open:        <blockers, deviations recorded in section 2, anything for Ido>
```

Short, honest, no padding. "Not verified" is never empty just because it reads better empty.

**Fallback — the Agent tool is unavailable in this session:** hand Ido exactly one line, nothing else: new terminal → `claude --model claude-sonnet-5 "read <absolute path> and execute it"`.

## When the build hits a wall

The build agent's only shared state with the planner is `PLAN-<slug>.md`. If it stalls, reports a blocker, or reality disagrees with the plan: fix the plan file — mark what changed in section 2, flip `Status:` to `revised` — and relaunch the subagent against it. Its report says where it stopped; the revised plan tells the next run what changed.

**If the builder was interrupted** (Esc, chat closed, session stopped, no BUILD DONE ever arrived): nothing is lost — its edits are on disk and the plan's `Progress:` line says which phases finished. **Resume in the same chat that was running the build, never a new one.** Each chat owns its own worktree, so the uncommitted edits exist only in that chat's folder; a fresh chat opens a different worktree and cannot see them, and it would also re-plan from zero. When Ido says "resume" / "continue the build" in that chat, skip Steps 0–4 entirely and relaunch `idos-builder` (foreground) with `Resume the plan at <absolute path> from its Progress line.` It verifies the finished phases against the working tree and continues; it does not redo them. Only if that chat is truly gone: start `claude` *inside that same worktree folder* and resume there. Re-running `/idos` is for when the shape of the work changes, not for small corrections; a plan that must be re-planned from scratch every time it meets reality is a plan that was written too rigidly.

Note: this skill is the heavyweight path. For a lightweight think-then-handoff without planning rounds, capability sourcing, or approval gates, Ido has the separate `/planhandoff` skill — don't invoke both on the same task.

