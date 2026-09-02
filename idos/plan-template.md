# PLAN — <task name>

**Date:** <YYYY-MM-DD> · **Planned on:** <Opus 5 | Fable 5> · **Build on:** Sonnet 5
**Status:** draft ← flip to `approved` at the approval gate; `revised` if edited during the build
**Progress:** not started ← the builder rewrites this after every phase (`Phase 2 done 14:03 — tests green`) so a killed run resumes instead of restarting

## 1. Objective
<One paragraph: what gets built, and the definition of done.>

## 2. Assumptions & decisions
<What the Q&A settled. One bullet per decision, so nothing gets re-litigated during the build. Append here as rounds complete — do not wait until the plan is final.>

- **<topic>:** <decision>

## 3. Phases

### Phase 1 — <name>
- **Do:** <what happens>
- **Files/areas:** <paths or surfaces touched>
- **Skills (load with the Skill tool before the first edit):** <skill name> — <why, at this point>
- **Verify:** <concrete checkpoint proving the phase worked>

### Phase 2 — <name>
- **Do:**
- **Files/areas:**
- **Skills:**
- **Verify:**

<Repeat per phase. Keep phases in execution order.>

## 4. Risks & open questions
- **<risk>:** <impact> → <fallback>

## 5. Capability map
<One row per capability, in firing order. Type is skill / connector / plugin.>

| # | Capability | Type | Phase | Source | Status |
|---|-----------|------|-------|--------|--------|
| 1 | `<skill>` | skill | Phase <n> | installed | ready |
| 2 | `<owner/repo@skill>` | skill | Phase <n> | external (<installs>, <repo>) | installed / declined |
| 3 | `<service>` | connector | Phase <n> | MCP | connected / **needs auth** / declined |
| 4 | `<plugin>@<marketplace>` | plugin | Phase <n> | marketplace (<token cost>) | installed / declined |

<Phases with no match, if any: `Phase <n> — nothing found, proceeding without.`>

**Blockers:** <Any connector still unauthorized, and which phase stalls on it. Delete this line if none.>

## 6. Build launch
- **Build agent:** Sonnet 5 subagent (fresh context), launched by the planner at approval.
- **Real gate:** <the command that proves the tree is good — e.g. `pnpm build`, not just `tsc`>
- **Verification harness:** <how checkpoints get checked when the app can't be driven live — jsdom tests, isolated HTML stage, measurement — or "live app at <url>">
- **Paid calls:** none | <which calls, how many, why>
- **Ship:** commit only (default) | `ship.ps1` | push — <who does it; the builder never pushes>
- <Any phase wanting a different model: which model, which phase, why — runs as its own subagent.>
