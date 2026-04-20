# CLAUDE.md

This project is the source workspace for the `self-evolution` skill.

## Project Purpose

- The workspace copy is the source of truth for this skill.
- Installed copies live under `~/.codex/skills/self-evolution/` and `~/.claude/skills/self-evolution/`.
- Update the workspace copy first, then sync installed copies by copying, never by moving the project directory.

## Targeting Rules

- If the user asks to optimize "当前项目", "这个项目", "this project", or the current repo, the target is this project and its files.
- Do not treat the loaded `self-evolution` skill as the target object unless the user explicitly says to optimize the skill itself.
- Do not rewrite installed copies first. Make changes in this workspace, validate them, then copy the updated files to installed locations.

## Files That Matter

- `SKILL.md` is the main skill logic and workflow contract.
- `references/` contains reusable support material and templates.
- `agents/openai.yaml` is Codex-side metadata, not the method itself.
- `AGENTS.md` is the Codex-side project adapter; keep host-specific rules aligned across both adapter files.

## Claude Code Working Style

- Keep `SKILL.md` lean. If guidance is Claude-specific or optional, prefer `CLAUDE.md` or `references/` over bloating the main skill file.
- Use the staged confirmation flow defined in `SKILL.md`: target, goal, surface, validation, and execution.
- Add surface confirmation when scope may drift: mutable surface first, locked evaluator second.
- Be explicit and concrete about the desired behavior. Prefer clear positive instructions over vague intent.
- Ask the smallest number of clarification questions needed to unblock meaningful progress.
- Match instruction freedom to task fragility. Use low-freedom wording for risky sequences, medium freedom for preferred templates, and high freedom for context-dependent diagnosis.
- When the thread becomes long or noisy, re-open `SKILL.md` and rebuild the active state from `references/iteration-state-snapshot.md` before continuing.
- Prefer editing the skill method and references over adding extra project documentation.
- Keep changes concise, operational, and host-compatible.
- Avoid over-aggressive trigger wording in host-specific guidance. Do not turn every preference into `MUST use` language, because newer Claude models may overtrigger.
- When context packaging gets complex, prefer compact XML-like tags such as `<target>`, `<goal>`, `<surface>`, `<validation>`, and `<next_action>`.
- Prefer a few concrete examples over long abstract explanation. For recurring ambiguous patterns, keep 3-5 compact examples in references.
- Stay local by default. Use Claude Code built-in `Explore` for read-only evidence gathering that would otherwise flood the main context. Use built-in `Plan` when sequencing is the hard part after target and goal are already clear.
- Prefer focused subagents with narrow responsibilities, clear descriptions, and only the tools they need. Create a custom subagent only if the same specialized sidecar keeps recurring.
- Trigger an isolated worktree or equivalent checkout only when one exploration or implementation path is broad, risky, or rollback-sensitive enough to justify separation.
- Keep the workspace copy as the source of truth even when worktrees or subagents are used; integrate validated results back into this project explicitly.
- Treat the main thread as the research lead and subagents as lab assistants: they may explore or implement bounded paths, but they do not own the final judgment.
- When Claude Code-specific guidance diverges from Codex-specific guidance, keep the shared method in `SKILL.md` and isolate host-specific behavior in `CLAUDE.md` or `AGENTS.md`.
- Use examples and fixtures to stabilize ambiguous behavior instead of endlessly stacking abstract prose.

## Claude Fast Path

When invoking this skill in Claude Code, prefer this compact structure over long prose:

```text
<target>...</target>
<goal>...</goal>
<surface>...</surface>
<validation>...</validation>
<next_action>...</next_action>
```

If the request is ambiguous, ask at most the minimum next question needed to fill one of those fields.

## Sync Rule

After editing this project:

1. Validate the workspace copy.
2. Copy updated files into `~/.codex/skills/self-evolution/`.
3. Copy updated files into `~/.claude/skills/self-evolution/`.
4. Verify the installed copies match the workspace source.

Never move the project directory during installation or sync.
