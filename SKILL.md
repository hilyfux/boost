---
name: self-evolution
description: Use when the user asks to optimize, improve, iterate, diagnose, evolve, monitor, stabilize, raise quality, reduce cost, reduce failure, raise conversion, or help an object get better over time. Triggers on software systems, workflows, prompts, skills, team processes, product surfaces, services, datasets, content pipelines, agents, support flows, or any observable artifact with a goal. Also triggers in Chinese on 优化、持续进化、监控问题、提出优化方案、定义验证指标、验证效果、迭代改进、提升质量、降低成本、减少失败、提高稳定性、提高转化. Structures an observe-diagnose-optimize-validate-iterate loop with explicit target, goal, validation, mutable-surface, and execution confirmation gates. The target can be this skill itself only when the user explicitly names it.
---

# Self Evolution

## Overview

Treat the thing to be improved as a target object. Model its goals, observability, mutable surface, fixed evaluator, constraints, failure modes, optimization options, and validation criteria, then run a disciplined iteration loop until the next-best improvement is implemented, validated, or clearly queued.

This skill is intentionally generic. Apply it to software, prompts, skills, documents, operating procedures, agents, support flows, content systems, data processes, or any object with observable behavior and improvable outcomes.

## Quick Reference

| Phase | Key Action | Gate / Output |
|-------|-----------|---------------|
| Resolve | Name the target object | `Confirmed Target: I will optimize X, not Y.` |
| Clarify | List `Open Questions` if any | Ask the minimum needed before locking plan |
| Frame | Goal, constraints, stakeholders | `Confirmed Goal: Success means Z, not shallow proxy.` |
| Charter | Define the research charter and mutable surface | `Confirmed Surface: I will change S, not locked evaluator E.` |
| Observe | Map observability surface | Evidence over intuition |
| Model | Symptom → Impact → Cause → Confidence → Evidence | Don't jump symptom → solution |
| Prioritize | Score by impact × confidence × cost × reversibility × learning | Pick one highest-leverage candidate |
| Topology | Trigger local vs delegated vs isolated worktree paths on demand | Use topology only when it earns its cost |
| Experiment | Hypothesis, change, metrics, baseline, guardrails, window, stop | `Confirmed Validation: Judge by M, protect G.` |
| Execute | Apply change on confirmed scope | `Confirmed Execution: Next I will A on B.` |
| Validate | Compare vs baseline, separate signal from noise | Decide keep / revise / rollback / switch |
| Loop | Queue next iteration or stop | State exactly why the loop ends |

## When to Use

Use when:

- no more specialized skill covers the target
- the user wants an explicit improvement loop, not one-off advice
- the task needs observability, prioritization, experimentation, or validation
- the agent is expected to diagnose and move toward execution

Do not use when:

- a domain-specific skill already dominates the task
- the user wants explanation, brainstorming, or theory only
- the target has no meaningful observable signals and the user refuses instrumentation
- the task is purely clerical

When in doubt, use this skill to structure the pass and hand off the implementation step to a more specialized skill if one exists.

## Target Resolution Rule

Resolve the target object before anything else. Priority order:

1. If the user names a concrete object, file, repository, workflow, service, document, prompt, or project, that named thing is the target.
2. If the user says "当前项目", "这个项目", "this project", "this repo", "this workflow", or points at surrounding workspace artifacts, the target is that project or artifact, not this skill.
3. Treat `self-evolution` itself as the target only when the user explicitly says so with wording such as "优化这个 skill", "improve this skill itself", "把这个技能当成对象", or equivalent.
4. If still ambiguous, pick the most external user-facing object, not the skill definition that is currently loaded.

Never default to self-modification just because this skill is present in context. Never treat invocation of `$self-evolution` as permission to rewrite `self-evolution` itself.

## Confirmation Protocol

State the relevant confirmations as short one-liners before each stage. Skip a gate only when the preceding context makes it unambiguous.

| Gate | When to state it | Format |
|------|------------------|--------|
| Target | Before analysis or edits | `Confirmed Target: I will optimize <target>, not <wrong alternative>.` |
| Goal | Before choosing metrics or experiments | `Confirmed Goal: Success means <outcome>, not <shallow proxy>.` |
| Surface | Before designing changes | `Confirmed Surface: I will change <mutable surface>, not <locked surface>.` |
| Validation | Before execution | `Confirmed Validation: Judge by <metric or evidence>, protect <guardrails>.` |
| Execution | Before editing or running an experiment | `Confirmed Execution: Next I will <action> on <scope>.` |

### Clarification Loop

Any of these count as confirmation-worthy uncertainty: unclear target, unclear success goal, unclear constraints, unclear validation basis, unclear execution authority.

Sequence:

1. List the unresolved points as `Open Questions: q1; q2; q3`.
2. Ask only the smallest set of questions needed to unblock the next iteration.
3. Wait for answers on target and goal before locking the plan.
4. Restate the resolved understanding in compact form once answers arrive.

Rules:

- Prefer short, concrete questions over broad discovery.
- Ask in the order target → goal → mutable surface / locked surface → constraints → validation → execution authority.
- Do not ask questions whose answers are recoverable from local context.
- Do not manufacture questions if the user already supplied enough information.

## Research System Frame

Before optimization, map the target into these roles:

- `Research charter` — the objective, guardrails, and stopping rule. Equivalent to `program.md`.
- `Mutable surface` — the files, prompts, workflow steps, or parameters allowed to change this round.
- `Locked evaluator` — the benchmark, review basis, fixture set, or acceptance test that should not drift during the experiment.
- `Experiment ledger` — the compact record of baseline, attempted change, result, and decision.
- `Topology` — whether the research runs locally, through delegated sidecars, or in isolated workspaces.

If any of these are missing and the task is otherwise opaque, instrument them before making broad changes.

## Workflow

After the confirmation gates clear, run this loop:

1. **Explore** the target object. Read the actual artifact. Do not edit blind.
2. **Define the mutable surface and locked evaluator** — explicitly say what can change and what will judge the change. Narrow the surface if rollback would otherwise be messy.
3. **Map observability** — inputs, process signals, outputs, side effects, human feedback, system feedback, environmental factors. If observability is weak, propose instrumentation as the first optimization.
4. **Build the problem model** — Symptom → Impact → Suspected cause → Confidence → Evidence → Missing evidence. Distinguish root causes, trigger conditions, amplifiers, and one-off anomalies.
5. **Prioritize** candidates by expected impact, confidence, cost, time-to-validate, reversibility, regression risk, and learning value. Prefer changes that are both useful and information-rich.
6. **Consider execution topology on demand** — Stay local by default. During exploration, open delegated or isolated paths only when they materially improve speed, safety, or learning.
7. **Design the experiment** — Hypothesis ("if we change X, metric Y should improve because Z"), change, leading + lagging metrics, baseline, guardrails, sample or time window, stop condition.
8. **Execute** when the user wants action and the environment allows it; otherwise deliver a plan specific enough to execute.
9. **Validate** — compare outcome vs baseline, separate signal from noise, note side effects. Decide keep / revise / rollback / switch.
10. **Update the ledger and queue the next loop** — record what changed, what improved, what failed, and the next highest-leverage target.

## Execution Topology On-Demand Strategy

Do not assume all optimization work should happen in the main thread and main workspace, but do not force delegation or isolation by ceremony either.

- `local` — do the work directly in the current thread when the task is small, tightly coupled, and the next step is obvious.
- `delegated` — use subagents for bounded sidecar work such as exploration, code review, fixture design, targeted implementation, or validation that can run in parallel without blocking the immediate next local step.
- `isolated-worktree` — use a worktree or equivalent isolated checkout when the change is high-risk, broad, likely to conflict, likely to require rollback, or benefits from side-by-side comparison.

- Prefer `local` for small single-scope edits.
- Prefer `delegated` when a read-heavy or implementation side task would flood the main context.
- Prefer `isolated-worktree` when one path may destabilize the workspace or needs cheap rollback.
- Combine `delegated` and `isolated-worktree` only when both clearly improve safety or throughput.
- Do not delegate or isolate by ceremony.
- Keep the primary path, final validation, and keep / revise / rollback / switch decision in the main thread.

## Autonomous Iteration Contract

After the four gates clear, default to autonomous iteration unless the user explicitly asked for analysis only ("先不要改", "analyze only", "just diagnose", "先不要动").

Autonomous means:

- pick the single best next intervention rather than listing many equal options
- keep the research charter stable unless new evidence justifies reopening it
- preserve a clear boundary between mutable surface and locked evaluator
- use execution topology only when it improves learning speed, safety, or throughput enough to justify the added coordination
- execute when the environment allows
- validate against the confirmed basis
- keep, revise, rollback, or switch based on observed evidence
- start the next iteration if the goal has not yet been met

Do not stop after one recommendation when the next safe action is obvious and executable. Do not override an explicit "analysis only" request.

## Rollback Decision

After each executed change, declare one:

- `keep` — improved the target without violating guardrails
- `revise` — right direction but needs refinement
- `rollback` — hurt the target, violated guardrails, or delivered no meaningful gain
- `switch` — inconclusive; try a different hypothesis next

Never accumulate speculative edits without evaluating whether each one earned its place.

## Stop Conditions

Continue iterating until one is true:

- the confirmed goal is met
- further change would violate guardrails
- remaining uncertainty cannot be resolved without new user input, authority, or external evidence
- the next iteration has lower expected value than its cost or risk
- the user asks to pause

When stopping, say exactly why.

## Context Drift Guard

Long or noisy threads can dilute the method. On any drift signal — target fuzzes; many unrelated subtopics; agent acts without restating target/goal/validation/execution; current plan diverges from the last confirmed plan; user corrects what is being optimized — pause, re-open `SKILL.md`, and rebuild state from `references/iteration-state-snapshot.md` before continuing.

Compact state snapshot format (end of each meaningful turn):

```text
Target:
Goal:
Surface:
Validation:
Guardrails:
Next Action:
Open Questions:
```

Keep the snapshot minimal. Do not let it grow into a second transcript.

## Output Contract

Unless the user requests another format, structure the response as:

1. `Target` (with Target Confirmation)
2. `Open Questions` (only if any remain)
3. `Goal and Constraints` (with Goal Confirmation)
4. `Research Charter and Surfaces` (with Surface Confirmation)
5. `Observability and Problem Model`
6. `Chosen Experiment` (with Validation Confirmation)
7. `Execution Topology` (only when relevant)
8. `Execution or Plan` (with Execution Confirmation)
9. `Result, Ledger Update, and Next Iteration`

For autonomous runs, keep updating the same structure across iterations rather than restarting from scratch. Favor tables and tight bullets over long prose.

When a host benefits from stronger structure, a compact tagged variant is also acceptable, for example:

```text
<target>...</target>
<goal>...</goal>
<surface>...</surface>
<validation>...</validation>
<next_action>...</next_action>
```

## Execution Modes

Pick the lightest mode that still moves the target forward:

- `diagnose` — map observability, build the problem model, identify the next move
- `experiment` — design one concrete change with metrics, guardrails, and a window
- `execute` — implement the chosen change directly when allowed
- `delegate` — split bounded subtasks across subagents when parallel exploration or context isolation improves the iteration
- `isolate` — run one candidate path in a worktree or equivalent isolated checkout when risk, breadth, conflict, or rollback needs justify separation
- `iterate` — run the explore-execute-test-validate loop until goal or stop condition
- `maintain` — improve this skill itself, only when the user explicitly names it as target

Default to `iterate` when the user asks for optimization and analysis-only is not specified.

## Trigger Examples

- "Use $self-evolution to improve our checkout funnel. Map the observability, pick one experiment, and define validation."
- "Use $self-evolution on this prompt chain. I want higher answer quality without increasing token cost."
- "Treat this skill itself as the target object and improve its triggerability, clarity, and evaluation design."
- "Use $self-evolution to turn this workflow into an autoresearch-style loop with a locked evaluator and a visible experiment ledger."
- "参考 Karpathy 的 autoresearch 思路，把这个 agent workflow 改成有研究章程、固定评测面和实验账本的闭环。"
- "把这个技能当成一个对象来进化，不要只给建议，要直接修改能提高触发率和可执行性的部分。"

## References

- [references/iteration-template.md](references/iteration-template.md) — full operating template for one pass
- [references/iteration-state-snapshot.md](references/iteration-state-snapshot.md) — compact state for long threads
- [references/claude-clarification-prompts.md](references/claude-clarification-prompts.md) — staged confirmation wording
- [references/evolution-metrics.md](references/evolution-metrics.md) — metric and evaluation patterns
- [references/eval-fixtures.md](references/eval-fixtures.md) — regression checks for skill behavior
- [references/research-loop-notes.md](references/research-loop-notes.md) — fuller notes on autoresearch transfer, roles, and exit criteria
- [references/skill-best-practices.md](references/skill-best-practices.md) — skill design, host compatibility, project layout, heuristics, failure modes, self-maintenance
