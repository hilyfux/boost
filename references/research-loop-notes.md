# Research Loop Notes

Use this reference when the compact `SKILL.md` is not enough and the agent needs the fuller research-loop rationale.

## AutoResearch Transfer Rules

Adapt these ideas from Karpathy's `autoresearch` pattern to general optimization work:

- Separate the research charter from the mutable artifact.
- Prefer a narrow mutable surface.
- Lock the evaluator when possible.
- Keep a visible baseline.
- Log experiment intent and outcome.
- Advance only on evidence.
- Use topology to make research cheaper, not to look sophisticated.

## Architecture Roles

- `Main thread / lead researcher` — owns target resolution, charter, topology choice, final integration, validation, and keep / revise / rollback / switch.
- `Sidecar explorers` — gather bounded evidence without owning the final decision.
- `Sidecar workers` — implement a bounded change on a clearly named surface.
- `Isolated branch or worktree` — acts as the experiment sandbox for risky or conflicting paths.
- `Ledger memory` — stores baseline, experiment intent, result, and rejected ideas.

Keep the primary path, final validation, and keep / revise / rollback / switch decision in the main thread.

## Exit Criteria

A pass is complete only when one is true:

- a concrete optimization was executed and its validation plan is stated
- a concrete experiment was selected with metrics, guardrails, and a next action
- the task was deferred to a more specialized skill with the reason made explicit
- the pass is blocked by a named missing dependency, authority, or evidence, with the unblock condition stated

If the user asked for optimization rather than analysis only, the pass should also show at least one executed-and-validated iteration, or a clearly explained blocker preventing the first iteration.
