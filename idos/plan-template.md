# PLAN — <task name>

**Planned on:** <Opus 5 | Fable 5> · **Build on:** Sonnet 5
**Status:** approved

## 1. Objective
<One paragraph: what gets built, and the definition of done.>

## 2. Assumptions & decisions
<What the Q&A settled. One bullet per decision, so nothing gets re-litigated during the build.>

- **<topic>:** <decision>

## 3. Phases

### Phase 1 — <name>
- **Do:** <what happens>
- **Files/areas:** <paths or surfaces touched>
- **Skills:** <skill name> — <why, at this point>
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

## 6. Model switches
- **Now → Sonnet 5** (`claude-sonnet-5`) before Phase 1.
- <Any mid-build switch: which model, which phase, why.>
