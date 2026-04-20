# Iteration State Snapshot

Use this reference when the conversation has become long, noisy, or branched and the agent needs to recover the active optimization state without rereading the entire transcript.

## Purpose

The snapshot is not a summary of the whole conversation. It is a minimal reloadable state for continuing the current iteration safely.

Use it to preserve:

- what is being optimized
- what outcome is being pursued
- how success will be judged
- what must not get worse
- what happens next
- what is still unresolved

## Snapshot Format

Use this exact compact structure:

```text
Target:
Goal:
Surface:
Validation:
Guardrails:
Next Action:
Open Questions:
```

## Writing Rules

- Keep each line short and operational.
- Prefer explicit nouns over pronouns.
- Prefer the current confirmed state over historical detail.
- If something is unknown, write `Unknown` rather than guessing.
- Replace stale state instead of appending to it forever.

## Mandatory Reload Checkpoints

Do not wait for drift to happen. Proactively reload at these moments:

| Checkpoint | Action |
|-----------|--------|
| Before each new iteration cycle | Re-read snapshot. If any field is unclear, re-open `SKILL.md`. |
| After large tool output (big grep, long test, etc.) | Write a one-line state anchor before proceeding. |
| After subagent returns | Restate the optimization frame before integrating results. |
| After context compaction (system truncates history) | Full reload: re-open `SKILL.md`, re-read snapshot, rebuild state. |

## When to Refresh the Snapshot

Refresh when:

- target, goal, validation, or next action changes
- an iteration completes (keep/rollback decision made)
- the user corrects a misunderstanding
- the agent returns from a subagent or worktree

## Recovery Protocol

When drift is detected (editing without knowing which experiment, target fuzzing, skipping validation):

1. STOP the current action.
2. Re-open `SKILL.md`.
3. Re-open this reference and any needed references.
4. Restate the latest valid snapshot.
5. Confirm the snapshot matches the user's intent.
6. Continue only after the frame is coherent again.

## Example

```text
Target: Claude Code compatibility of the boost project
Goal: Make Claude handle target confirmation and staged questioning more reliably
Surface: Edit workspace SKILL.md and host adapters, not installed copies first
Validation: Fewer target-confusion failures and clearer staged confirmations
Guardrails: Do not break Codex compatibility or move the project directory
Next Action: Update project-level guidance and sync installed copies
Open Questions: Whether to add few-shot clarification examples next
```
