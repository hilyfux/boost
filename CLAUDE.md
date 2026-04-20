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

## Claude Code Execution Patterns

This section guides Claude Code when running the self-evolution skill on any target — not just this project.

### Validation With Tools

The validation protocol in SKILL.md requires concrete checks. In Claude Code, use these tools:

| Validation need | Tool pattern |
|----------------|-------------|
| Check file content after edit | `Read` the changed file, confirm the change landed correctly |
| Compare before/after | `Bash` with `git diff` or store the original via `Read` before editing |
| Run tests | `Bash` to execute the project's test command |
| Check build | `Bash` to run build/lint/typecheck |
| Verify structure | `Grep` or `Glob` to confirm expected patterns exist |
| Measure output quality | `Bash` to run the artifact and inspect output |
| Regression check | `Bash` to run the existing test suite or fixture set |

Always capture baseline state before executing changes. At minimum, read the files you plan to change.

### Validation Examples

**After editing a prompt file:**
1. Read the changed file to confirm the edit is correct
2. If a test harness exists, run it and compare output to baseline
3. If no harness, produce one sample output and inspect it

**After editing code:**
1. Run the project's test suite or linter
2. If tests pass, check the specific behavior that should have changed
3. If tests fail, diagnose before declaring rollback

**After editing this skill itself:**
1. Count SKILL.md lines (soft limit ~300)
2. Check that frontmatter is valid
3. Walk through 2-3 eval fixtures mentally or by re-reading them
4. Verify reference links still resolve

### When Validation Is Not Runnable

If the target has no tests, no build, and no runnable output, say so explicitly. Propose instrumentation as the first optimization rather than skipping validation silently.

### Using Host Capabilities

Use subagents, worktrees, and other host capabilities when they earn their cost — see the Execution Topology table in SKILL.md for trigger conditions. Key principles:

- Subagents get narrow scope and a concrete deliverable. Never delegate the keep/rollback decision.
- Worktrees are for risky experiments that could destabilize the workspace. Merge only after validation.
- After any delegation returns, restate the state snapshot before acting on results.

### Context Preservation

If you notice yourself editing without knowing which experiment the edit belongs to — STOP. Re-read `SKILL.md`, rebuild state, then continue. This is the #1 failure mode in long sessions.

## Claude Code Working Style

When editing this skill project specifically:

- Keep `SKILL.md` lean. Move Claude-specific or optional guidance to `CLAUDE.md` or `references/`.
- Use the staged confirmation flow: target, goal, surface, validation, execution.
- Add surface confirmation when scope may drift.
- Ask the smallest number of clarification questions needed to unblock progress.
- When the thread becomes long or noisy, re-open `SKILL.md` and rebuild state from `references/iteration-state-snapshot.md`.
- Prefer compact XML-like tags (`<target>`, `<goal>`, `<surface>`, `<validation>`, `<next_action>`) when context is dense.
- Prefer a few concrete examples over long abstract explanation.
- Stay local by default. Use `Explore` for read-heavy evidence gathering. Use `Plan` when sequencing is the hard part.
- Prefer focused subagents with narrow responsibilities. Trigger worktrees only for broad/risky changes.
- Keep the workspace copy as source of truth; integrate validated results back explicitly.
- Avoid over-aggressive trigger wording that causes newer Claude models to overtrigger.
- Use examples and fixtures to stabilize ambiguous behavior.

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
