# idos — Plan First, Build Later

A model-neutral planning skill for Ido's Claude Code and GPT workflows. Suggests planning and implementation models, uncovers blind spots with short questions, records a plan, then builds after approval.

Fable 5.1 and GPT Astra are user choices when the host offers them. Neither is required. Model names/IDs come from the current host; this skill does not add cross-provider integration.

## Workflow

1. Read relevant project context and existing decisions.
2. Suggest planning and build models with short reasons. The same model is allowed.
3. Ask questions in dependency-ordered rounds until material uncertainty is resolved.
4. Write PLAN-<slug>.md with scope, decisions, verification, build choice and delivery permissions.
5. Get approval. If the selected model requires a manual switch, stop before implementation and wait for confirmation.
6. Execute directly or through supported delegation, checkpoint progress, verify and review.

Recommendations never switch models automatically. A different chosen build model never silently falls back to the current model. The optional Claude agent uses `model: inherit`; an approved alternative requires a supported override. Settings can override requests, so reports distinguish requested from observed models. See [Claude Code model resolution](https://code.claude.com/docs/en/sub-agents#choose-a-model).

## Install

From the repository root, PowerShell for Claude Code:

```powershell
New-Item -ItemType Directory -Force "$HOME/.claude/skills/idos", "$HOME/.claude/agents" | Out-Null
Copy-Item -Path "./idos/*" -Destination "$HOME/.claude/skills/idos" -Recurse -Force
Copy-Item -LiteralPath "./idos/idos-builder.md" -Destination "$HOME/.claude/agents/idos-builder.md" -Force
```

For hosts discovering user skills in ~/.agents/skills (including Codex):

```powershell
New-Item -ItemType Directory -Force "$HOME/.agents/skills/idos" | Out-Null
Copy-Item -Path "./idos/*" -Destination "$HOME/.agents/skills/idos" -Recurse -Force
```

Invoke /idos in Claude Code or $idos in Codex, followed by the task. Tools depend on the host. The agent definition is a Claude adapter; other hosts use its execution guidance directly or through their own delegation tools.

Upgrade both skill and installed agent copy. These commands replace files; preserve intentional customizations first. Running sessions may retain older instructions; check loaded agent/model settings before the next launch.

## Resume and cost

Say "resume" in the original chat. The skill checks workspace, working tree, progress and in-flight work before continuing. It never resumes another chat's task without explicit authorization.

Targeted reads, bounded exploration and relevant capability discovery keep startup lean. No forced research agents, marketplace sweeps, context clearing or fixed token-savings promises.

## Files

- idos/SKILL.md: planning, suggestions, approval and supervision.
- idos/plan-template.md: durable decisions, checkpoints and execution choice.
- idos/idos-builder.md: model-neutral execution guidance and optional Claude agent.

Build approval does not imply paid calls or deployment. Explicitly authorized releases follow project gates. Reports separate verified results, missing checks and requested versus observed models.

## License

MIT
