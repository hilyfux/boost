# AGENTS.md

This project is the source workspace for the `self-evolution` skill.

## Purpose

- The workspace copy is the source of truth for this skill.
- Installed copies live under `~/.codex/skills/self-evolution/` and `~/.claude/skills/self-evolution/`.
- Update this workspace first, validate it, then sync installed copies by copying.

## Codex Working Style

- Use `SKILL.md` as the cross-host method source.
- Resolve the target object before editing anything.
- Confirm the mutable surface and locked evaluator before broad changes.
- Treat the user request like a compact GitHub issue: target, scope, file paths, validation, and constraints should be made explicit whenever possible.
- Stay local by default. Trigger subagents only when bounded sidecar work clearly improves throughput, context isolation, or validation speed.
- Trigger isolated worktrees only when a risky, conflicting, or rollback-sensitive path clearly benefits from separation.
- Keep the critical-path integration and final validation decisions in the main thread.
- Treat worktrees and subagents as on-demand accelerators, not mandatory steps.
- Treat the main thread as the research lead: it owns the charter, baseline, ledger, final evaluation, and keep / revise / rollback / switch call.
- Prefer well-scoped changes over broad vague missions; if the task is too fuzzy, tighten the scope before acting.
- Return auditable evidence whenever possible: what changed, what was run, and what validated the result.
- Keep persistent operating notes here and keep the reusable method in `SKILL.md`.

## Files That Matter

- `SKILL.md` is the host-agnostic method and output contract.
- `references/` contains templates, fixtures, and host-compatibility guidance.
- `CLAUDE.md` is the Claude Code adapter for this project.
- `agents/openai.yaml` is Codex-side skill metadata and should stay aligned with the current method.
- `references/host-adaptation-best-practices.md` summarizes the current Claude Code and Codex adaptation heuristics.

## Sync Rule

After editing this project:

1. Validate the workspace copy.
2. Copy updated files into `~/.codex/skills/self-evolution/`.
3. Copy updated files into `~/.claude/skills/self-evolution/`.
4. Verify the installed copies match the workspace source.

Never move the project directory during installation or sync.
