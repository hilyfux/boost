# Host Adaptation Best Practices

Use this reference when tuning `self-evolution` specifically for Claude Code and Codex.

The goal is not to fork the method. The goal is to keep one shared optimization loop in `SKILL.md` while adapting prompt surfaces, memory surfaces, and execution patterns to each host.

## Shared Principle

- Keep the core method in `SKILL.md`.
- Keep host-specific behavior in adapter files such as `CLAUDE.md`, `AGENTS.md`, and `agents/openai.yaml`.
- Keep examples, fixtures, and templates in `references/`.
- Do not let host-specific wording leak into the shared method unless it is genuinely cross-host.

## Claude Code

Claude Code responds best when instructions are explicit, concrete, and not artificially over-aggressive.

Use these patterns:

- Say exactly what behavior is desired instead of relying on implication.
- Use normal trigger language like "use this when..." instead of escalating every rule into `MUST` language, which can increase overtriggering on newer Claude models.
- Keep the response format explicit and stable.
- Use examples for ambiguous or failure-prone behaviors.
- Treat subagents as focused workers with narrow responsibilities, clear descriptions, and limited tools.
- Prefer built-in or focused subagents for exploration, planning, and sidecar analysis when they preserve main-thread context.
- Keep long-running or noisy work out of the main thread when it would dilute context quality.

Claude-specific adapter files should therefore emphasize:

- concise, explicit behavior shaping
- trigger boundaries to avoid overtriggering
- when to stay local vs when to use subagents
- when to use worktrees for risky branches
- context recovery behavior for long conversations

## Codex

Codex works best when the persistent repo instructions behave like project operating notes, not a second copy of the skill.

Use these patterns:

- Keep `AGENTS.md` practical: how to navigate, how to validate, what source of truth to follow, and how to report evidence.
- Keep tasks well-scoped, closer to a GitHub issue than to a vague aspiration.
- Include concrete file paths, artifacts, and validation commands when known.
- Prefer explicit verification expectations so Codex can return auditable evidence.
- Preserve the workspace source of truth and describe how isolated environments or parallel agents should integrate back.
- Keep metadata like `agents/openai.yaml` concise and trigger-oriented; do not stuff the whole method into it.

Codex-specific adapter files should therefore emphasize:

- issue-style task framing
- persistent operational context in `AGENTS.md`
- explicit validation and evidence expectations
- environment consistency and source-of-truth rules
- on-demand use of parallel agents and isolated worktrees

## What to Check After Host Adaptation

- Does Claude get enough explicit structure without being over-constrained into tool or skill overtriggering?
- Does Codex get enough project context without duplicating the full method in metadata?
- Can both hosts still follow the same target / goal / validation / execution loop?
- Are adapter files clearly separated from the shared method?
- Do examples and fixtures cover host-specific failure modes?
