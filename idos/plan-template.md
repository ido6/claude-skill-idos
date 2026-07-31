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

## 5. Skill map
<One row per skill, in firing order. Source is `installed` or `external`.>

| # | Skill | Phase | Source | Status |
|---|-------|-------|--------|--------|
| 1 | `<skill>` | Phase <n> | installed | ready |
| 2 | `<owner/repo@skill>` | Phase <n> | external (<installs>, <source repo>) | installed / declined |

<Phases with no good skill match, if any: `Phase <n> — no skill found, proceeding without one`>

## 6. Model switches
- **Now → Sonnet 5** (`claude-sonnet-5`) before Phase 1.
- <Any mid-build switch: which model, which phase, why.>
