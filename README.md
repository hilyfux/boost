# boost

`boost` is an agent skill for optimizing a target object through a disciplined research loop instead of one-shot advice.

## Description

A cross-host agent skill for iterative optimization with explicit target resolution, locked evaluators, experiment memory, and validation-driven keep / revise / rollback decisions.

It is designed for tasks like:

- improving prompts, workflows, agents, services, or repositories
- diagnosing why a system is underperforming
- choosing one concrete next experiment instead of listing vague options
- iterating with validation, rollback, and experiment memory

The current version is written to work across both Claude Code and Codex, with extra attention to Claude Code prompt structure and context management.

## Core Idea

The skill treats the thing being improved as a target object and drives it through a compact loop:

1. Confirm the target.
2. Confirm the goal.
3. Confirm the mutable surface and locked evaluator.
4. Map observability and form a problem model.
5. Choose one experiment.
6. Execute, validate, and decide `keep`, `revise`, `rollback`, or `switch`.

The method borrows from Karpathy-style `autoresearch` patterns:

- narrow mutable surface
- stable evaluator
- visible baseline
- experiment ledger
- evidence-based advancement

## Repository Layout

- `SKILL.md`
  The main cross-host skill definition. Keep this lean and operational.

- `CLAUDE.md`
  Claude Code-specific project adapter and execution guidance.

- `AGENTS.md`
  Codex-specific project adapter and repository working rules.

- `agents/openai.yaml`
  Codex metadata used for discovery and UI rendering.

- `references/claude-clarification-prompts.md`
  Claude-friendly confirmation wording, tagged state packets, and few-shot examples.

- `references/iteration-template.md`
  Full working template for a single optimization pass.

- `references/iteration-state-snapshot.md`
  Compact reloadable state for long or noisy threads.

- `references/eval-fixtures.md`
  Regression fixtures for trigger behavior, clarification, iteration, and host-specific behavior.

- `references/research-loop-notes.md`
  Fuller notes on research-loop rationale, roles, and exit criteria.

- `references/skill-best-practices.md`
  Authoring guidance for improving this skill itself.

## Install

### Claude Code

Clone or copy this directory to:

```bash
~/.claude/skills/boost/
```

Claude Code reads `SKILL.md` as the main skill definition and uses `CLAUDE.md` as project guidance when you work on this repository.

### Codex

Clone or copy this directory to:

```bash
~/.codex/skills/boost/
```

Codex uses both `SKILL.md` and `agents/openai.yaml`.

## Usage

Example requests:

- `Use $boost to improve this prompt chain without increasing token cost.`
- `用 $boost 优化这个 workflow，先确认目标和验证标准，再做一轮可执行优化。`
- `Treat the boost skill itself as the target object and improve its Claude Code behavior.`

Typical output structure:

```text
Target
Open Questions
Goal and Constraints
Research Charter and Surfaces
Observability and Problem Model
Chosen Experiment
Execution or Plan
Result and Next Iteration
```

For Claude Code, a compact tagged structure is also supported:

```text
<target>...</target>
<goal>...</goal>
<surface>...</surface>
<validation>...</validation>
<next_action>...</next_action>
```

## Development Notes

- The workspace copy is the source of truth.
- Installed copies under `~/.claude/skills/` and `~/.codex/skills/` should be updated by copying from the workspace.
- Backup files such as `*.bak` and `*.bak.*` are intentionally ignored by git.
- Keep `SKILL.md` short. Push extra rationale, examples, and host-specific tactics into `references/`, `CLAUDE.md`, or `AGENTS.md`.

## Testing

This repository is maintained by behavior-driven iteration rather than unit tests alone.

Use these checks when changing the skill:

- verify trigger quality from realistic user prompts
- verify the target is resolved correctly
- verify Claude Code behavior stays concise and structured
- verify the skill does not accidentally self-modify unless explicitly asked
- verify experiment and rollback logic still appears in outputs

The main regression reference is:

- `references/eval-fixtures.md`

## Git Hygiene

This repository ignores backup files through `.gitignore`:

```text
*.bak
*.bak.*
```

That keeps working backups out of commits while preserving the main skill files in version control.
