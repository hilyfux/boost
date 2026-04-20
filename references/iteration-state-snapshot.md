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

## When to Refresh

Refresh the snapshot when:

- the target is newly confirmed
- the goal changes
- the validation basis changes
- the next action changes
- the user corrects a misunderstanding
- the thread becomes long enough that the active plan may be forgotten

## Recovery Protocol

When drift is suspected:

1. Re-open `SKILL.md`.
2. Re-open any needed references.
3. Restate the latest valid snapshot.
4. Confirm whether the snapshot still matches the user's intent.
5. Continue only after target, goal, surface, validation, and next action are coherent again.

## Example

```text
Target: Claude Code compatibility of the self-evolution project
Goal: Make Claude handle target confirmation and staged questioning more reliably
Surface: Edit workspace SKILL.md and host adapters, not installed copies first
Validation: Fewer target-confusion failures and clearer staged confirmations
Guardrails: Do not break Codex compatibility or move the project directory
Next Action: Update project-level guidance and sync installed copies
Open Questions: Whether to add few-shot clarification examples next
```
