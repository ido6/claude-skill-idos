---
name: idos
description: Plan before building. Uncover blind spots with short questions, suggest suitable models, persist decisions, and execute after approval. Use for /idos, $idos, or planning-before-implementation requests.
---

# idos — Plan First, Build Later

Model- and provider-neutral. Fable 5.1 and GPT Astra are Ido's choices when available, not mandatory models or interchangeable API IDs. Use the current host's tools. Recommendations never authorize switching or building.

## 1. Orient and suggest models

Use the task in the invocation or preceding request; a trailing /idos still refers to that request. Ask what to plan only when no task exists. Small changes get short plans, not extra triage permission rounds.

Read exposed current host/model information. If unavailable, say unknown; never infer model identity from branding or old messages. Use advertised model names/IDs and supported controls. Do not scan credentials or whole configuration files.

Briefly show:
- Planning: <available model> — <task-specific reason>.
- Implementation: <available model; same model allowed> — <reason>.
- Current: <observed model or unknown>; recommendations only.

Consider Fable 5.1 and GPT Astra when available. Prefer reasoning capacity for uncertain architecture and a capable economical coding model for settled work. Do not rank families categorically or promise free switching/savings. Respect the user's chosen model.

Continue planning on the current model unless Ido requests a switch or pause; then stop dependent work until confirmation. At approval record the accepted build model and execution method. Never silently implement on a different model.

## 2. Read narrowly, ask clearly

Read relevant project instructions (CLAUDE.md / AGENTS.md), task files, applicable memory, prior plan decisions and git/worktree state. Reuse loaded instructions; read missing relevant sections only. Do not inspect unrelated chats or projects.

- Prefer targeted search and bounded reads. Delegate broad exploration only when useful; request concise findings with paths and evidence. Small lookups need no agent.
- Batch independent reads. Avoid whole-file dumps, repeated inventories and automatic agent launches.
- Do not infer token costs from transcript size or sum duplicate streaming usage records.
- Facts are your job; ask Ido for choices the environment cannot settle.

Questions: short, plain, one decision each. Aim for about 10 words and 4-word options; clarity wins over hard caps. Recommended option first, brief reason when useful. Use available structured-question tools within their actual limits, or concise plain text.

Work in dependency-ordered rounds: ask only questions whose prerequisites are settled, then uncover downstream decisions. Hunt unstated assumptions, contradictions, hidden dependencies and outcomes that pass tests but fail the mission. No fixed checklist, question quota or endless search for every imaginable risk.

Relevant probes: success, scope, mobile/RTL, design preservation, real verification gate, live access, paid calls and delivery. Derive project-specific commands and constraints from current evidence.

"You decide" authorizes your recommendation; record it. Skipped/unanswered questions are not approval. Leave consequential choices pending; low-impact defaults require explicit assumptions. Do not re-ask settled answers or existing permissions.

## 3. Persist plan and capabilities

Use [plan-template.md](plan-template.md) for PLAN-<slug>.md in the task's working directory. Keep depth proportional. Update the matching task plan; never overwrite an unrelated plan.

Create a draft as decisions settle, append each round's answers. Record exact workspace, branch if applicable, progress and verification evidence. A plan is durable context, not proof that code/checks match it.

Map only useful capabilities to phases. Reuse installed skills and connected tools first. Search externally only for a concrete missing capability; no mandatory marketplace sweep. Inspect external instructions and executable content before recommending. Popularity is not proof of safety. Install authorized additions using supported tools; do not invent CLI flags, token estimates or auth status.

Read required skills before affected work using available loaders or SKILL.md paths. Reuse content already loaded in the same context. Report missing capabilities. For idoGen media work load applicable image/video/acting craft skills before writing generation prompts.

## 4. Approve plan and build choice

Show the plan link and up to five short lines: outcome, scope, verification, build model/method, additions needing approval.

Ask for approval once unless already explicitly granted. Approval covers the stated build choice, not a hidden fallback. "Plan only" ends here; otherwise approval authorizes the scoped build.

Unavailable chosen model: request another choice or explicit continuation on the current one. A requested switch stays pending until confirmed. Preserve separate authorization for paid calls, live changes and releases; do not re-ask permissions already granted.

## 5. Execute on the accepted model

Choose one supported execution path:
- Current model selected: execute here after approval, or delegate when useful.
- Different model with supported delegation: use its verified host ID, absolute plan path and workspace. Cross-provider execution requires a real configured integration.
- Manual switch required: save the plan, say "Plan ready. Switch to <model>, then say go." Stop before implementation. Never issue user-only commands or claim a switch happened. If identity cannot be read, label it user-confirmed.

Claude Code adapter: [idos-builder.md](idos-builder.md) defaults to model: inherit. Use a supported per-invocation override only for an approved alternative. Check loaded agent and relevant settings before launch; older copies may still pin Sonnet. Do not rewrite a shared agent during an active run. Other hosts use their available delegation or execute directly; Claude-specific tool names are not required.

Prefer foreground delegation where supported; otherwise collect results through available wait tools. Remain responsible for completion. Announce launch success only after confirmation. Record agent identifier and observed model when exposed, separating requested from observed. Configuration can override requests: pause/report detected mismatches before edits when possible.

Pass plan path, workspace and only essential missing context. Separate agents still have their own instructions/tools and may inherit context; never promise zero tokens or zero history.

Read the execution body of idos-builder.md for direct and delegated builds. Verify workspace and existing task ownership; never launch two writers for the same work.

## 6. Verify, review, finish

Run relevant plan gates and preserve outputs. Repeat expensive passed checks only for changed code/environment or unresolved evidence. Verify UI in the running app when possible; label live/device checks not performed.

Review the scoped diff independently when available and warranted, otherwise review directly and state that. Resolve material findings and rerun affected checks; avoid unbounded review loops. A missed required skill triggers a targeted compliance review after loading it, not blind reimplementation.

Mark complete only after required work/checks finish; otherwise record blocked or partially verified. Brief report:
- Built.
- Model: requested / observed or unverified.
- Verified; not verified and why (none is valid).
- Skills/review.
- Git/release state; open items.

## Resume and scope

Resume only in the original chat and workspace. Asked elsewhere: identify the original chat and stop; do not resume or message it without explicit authorization. Chats can share directories; verify ownership instead of assuming one worktree per chat.

In the original chat read Progress, git status and recent evidence. Skip settled planning, continue unfinished work. Missing tool output means unknown result, not proof a process died or a chat closed. Check in-flight work before relaunching.

Record small corrections in the plan. Ask only about material scope/model/authorization changes. Do not clear context, create another chat or migrate work merely to reduce tokens.
