# Research Loop Notes

Use this reference when the compact `SKILL.md` is not enough and the agent needs the fuller research-loop rationale.

## Universal Principles from Autoresearch

Karpathy's `autoresearch` demonstrated a tight autonomous research loop on ML training. The patterns below are the domain-agnostic principles extracted from that work, adapted for any optimization target.

### 1. Three-Role Governance

Every optimization loop needs three clearly separated roles:

| Role | Autoresearch example | General equivalent |
|------|---------------------|-------------------|
| Charter (human-owned) | `program.md` | Confirmed target + goal + constraints |
| Mutable surface (agent-owned) | `train.py` | The files, configs, prompts, or steps the agent can change |
| Locked evaluator (nobody touches) | `prepare.py` + val_bpb | The benchmark, test suite, review basis, or acceptance criteria that stays fixed for this round |

The key insight: when the same loop can freely change both the artifact and the thing that judges it, all comparisons become meaningless. Lock the evaluator.

### 2. Ratchet Property

The working state always represents the current best. Failed experiments are reverted, not accumulated.

- In git repos: branch tip = validated best. Rollback = `git reset`.
- In non-git targets (workflows, processes, docs): save a snapshot before each experiment. Revert to it on failure.
- In ephemeral targets (prompts, configs): keep a "last known good" copy.

The ratchet prevents drift: you never end up with a workspace full of half-tested changes.

### 3. Persistent Experiment Ledger

Every attempt — including failures — gets noted in conversation context before moving on. The ledger prevents rediscovering dead ends. It lives in the agent's memory during the session, not in files. No tracking files, no extra artifacts.

### 4. Advance Only on Evidence

Do not keep a change because it "looks right." Run a concrete check, compare against baseline, and only then decide. This is operationalized in the Validation Protocol in SKILL.md.

### 5. Simplicity Criterion

All else being equal, simpler is better. A marginal improvement that adds ugly complexity is not worth keeping. This applies across domains:

- Code: fewer lines, fewer abstractions, fewer edge cases
- Prompts: shorter, clearer, fewer conditionals
- Workflows: fewer steps, fewer handoffs, fewer failure modes
- Configs: fewer parameters, fewer overrides

### 6. Crash Resilience

Experiments can fail in ways beyond "didn't improve" — they can crash, error, or produce invalid output. When this happens:

1. Log the failure (what was tried, what error occurred)
2. Revert to the last known good state
3. Skip ideas that are broken at the root — do not debug endlessly
4. Try the next hypothesis

This prevents the loop from getting stuck on one broken path.

### 7. Autonomous Continuity

Once the charter is set, the agent drives the full loop without pausing for permission. The human's role shifts from operator to research advisor — they set direction through the charter and review accumulated results.

In our skill, this maps to: once target + goal are clear, own the method. Ask the user about goals, not about how to validate or what to try next.

## How These Differ from Autoresearch

Our skill generalizes autoresearch in several ways:

| Autoresearch (ML-specific) | Self-evolution (general) |
|---------------------------|------------------------|
| Fixed target: `train.py` | Arbitrary target with resolution protocol |
| Fixed metric: `val_bpb` | Agent-selected metrics from target's observability |
| Fixed time budget: 5 min | No fixed budget — scope appropriate to the target |
| Human writes `program.md` | Agent builds charter from user's goal via Socratic discovery |
| Single-file mutable surface | Multi-file, multi-dimension surface with explicit boundaries |
| Local execution only | Local / delegated / isolated topology |
| No problem modeling | Symptom → impact → cause → evidence model before acting |
| Stateless per iteration | Context drift guard, state snapshots, session recovery |

The core loop is the same: charter → explore → hypothesize → change → measure → keep or revert → repeat. The difference is flexibility.

## Architecture Roles

- `Main thread / lead researcher` — owns target resolution, charter, topology choice, final integration, validation, and keep / revise / rollback / switch.
- `Sidecar explorers` — gather bounded evidence without owning the final decision.
- `Sidecar workers` — implement a bounded change on a clearly named surface.
- `Isolated branch or worktree` — acts as the experiment sandbox for risky or conflicting paths.
- `Ledger memory` — stores baseline, experiment intent, result, and rejected ideas.

Keep the primary path, final validation, and keep / revise / rollback / switch decision in the main thread.

## Exit Criteria

A pass is complete only when one is true:

- a concrete optimization was executed and validated with evidence
- a concrete experiment was selected with metrics, guardrails, and a next action
- the task was deferred to a more specialized skill with the reason made explicit
- the pass is blocked by a named missing dependency, authority, or evidence, with the unblock condition stated

If the user asked for optimization rather than analysis only, the pass should also show at least one executed-and-validated iteration, or a clearly explained blocker preventing the first iteration.
