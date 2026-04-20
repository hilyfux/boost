---
name: boost
description: Use when the user asks to optimize, improve, iterate, diagnose, evolve, monitor, stabilize, raise quality, reduce cost, reduce failure, raise conversion, or help an object get better over time. Triggers on software systems, workflows, prompts, skills, team processes, product surfaces, services, datasets, content pipelines, agents, support flows, or any observable artifact with a goal. Also triggers in Chinese on 优化、持续进化、监控问题、提出优化方案、定义验证指标、验证效果、迭代改进、提升质量、降低成本、减少失败、提高稳定性、提高转化. Structures an observe-diagnose-optimize-validate-iterate loop with explicit target, goal, validation, mutable-surface, and execution confirmation gates. The target can be this skill itself only when the user explicitly names it.
---

# Boost

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
| Validate | Run check, compare baseline, verify guardrails | Evidence-based keep / revise / rollback / switch |
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
3. Treat `boost` itself as the target only when the user explicitly says so with wording such as "优化这个 skill", "improve this skill itself", "把这个技能当成对象", or equivalent.
4. If still ambiguous, pick the most external user-facing object, not the skill definition that is currently loaded.

Never default to self-modification just because this skill is present in context. Never treat invocation of `$boost` as permission to rewrite `boost` itself.

## Confirmation Protocol

Confirmations are declarations, not questions. State them as short one-liners so the user can redirect if wrong, but do not block on user approval for operational gates.

| Gate | Ask user? | Format |
|------|-----------|--------|
| Target | Only if ambiguous | `Confirmed Target: I will optimize <target>, not <wrong alternative>.` |
| Goal | Only if genuinely unclear | `Confirmed Goal: Success means <outcome>, not <shallow proxy>.` |
| Surface | No — decide autonomously | `Confirmed Surface: I will change <mutable surface>, not <locked surface>.` |
| Validation | No — design autonomously | `Confirmed Validation: Judge by <metric or evidence>, protect <guardrails>.` |
| Execution | No — declare and proceed | `Confirmed Execution: Next I will <action> on <scope>.` |

### Goal Discovery (Socratic)

When the user's goal is vague ("优化一下", "make it better", "提升质量"), do not guess the intent and rush in. Instead, ask 1-3 sharp, Socratic questions that help the user clarify what actually matters. These questions target the **why and what**, never the **how**.

Good goal-discovery questions:

- "这个系统最让你头疼的问题是什么？" / "What hurts most right now?"
- "如果只能改一件事，你最希望看到什么变化？" / "If you could fix one thing, what would it be?"
- "你说的'质量不好'，能举一个最近的具体例子吗？" / "Can you give me a concrete recent example of the problem?"
- "这个改进是为了谁？用户、团队、还是系统稳定性？" / "Who benefits most from this improvement?"

Stop asking as soon as the target and desired outcome are clear enough to act on. Do not over-interview.

### Clarification Boundaries

Once the goal is clear, everything operational is yours to decide. Do not ask the user about execution details.

**Ask the user (goal layer only):**

- Which object to optimize, when multiple plausible targets exist
- What outcome matters, when the user's intent is genuinely unclear
- What the real pain point is, when "improve" is too vague to act on

**Decide autonomously (never ask):**

- What metrics to use — infer from the goal and the target's observability
- What experiments to try — pick the highest-leverage candidate
- How to validate — design the check from the target's structure
- What surface to change — derive from the goal and constraints
- When to rollback vs iterate — decide from evidence

Rules:

- Never ask the user how to validate. Figure it out.
- Never ask the user what metrics to track. Infer them.
- Never ask the user what to try next. Pick the best option and execute.
- Do not ask questions whose answers are recoverable from local context.
- Do not manufacture questions if the user already supplied enough information.

## Research System Frame

Before optimization, map the target into these roles:

- `Research charter` — the objective, guardrails, and stopping rule. Equivalent to `program.md`.
- `Mutable surface` — the files, prompts, workflow steps, or parameters allowed to change this round.
- `Locked evaluator` — the benchmark, review basis, fixture set, or acceptance test that should not drift during the experiment.
- `Experiment ledger` — the in-context record of what was tried, what happened, and what was decided. Lives in conversation memory, not in files.
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
9. **Validate** — run the validation protocol below. Do not skip this step. Do not declare success without evidence.
10. **Update the ledger and queue the next loop** — in context, record what was tried, the result, the decision, and the next hypothesis. Do not create files for tracking — use conversation memory.

### Validation Protocol

After every executed change, before declaring keep / revise / rollback / switch:

1. **State what you will check** — name the specific file, output, behavior, metric, or test that should show improvement.
2. **Run the check** — execute a concrete verification action: run a command, read the changed file, diff before/after, invoke a test, or produce the artifact and inspect it. "I believe it improved" is not validation.
3. **Compare against baseline** — state the before state and the after state side by side. If you did not record a baseline before execution, acknowledge the gap.
4. **Check guardrails** — verify that nothing protected by guardrails got worse. Name what you checked.
5. **Declare the decision** — keep / revise / rollback / switch, with the evidence that supports it.

If the host environment supports tool execution (e.g., shell commands, file reads, test runners), use those tools to validate. If the environment does not support execution, state the validation plan explicitly enough that the user or a follow-up session can run it.

Never skip validation because the change "looks right." Never batch multiple unvalidated changes before checking any of them.

## Execution Topology

Use the lightest topology that fits. Trigger heavier ones by condition, not by habit.

| Condition | Topology | Why |
|-----------|----------|-----|
| Single-scope edit, next step obvious | `local` | No overhead |
| 2+ independent exploration directions | `delegated` (parallel subagents) | Explore in parallel, converge in main thread |
| Read-heavy evidence gathering that would flood context | `delegated` (Explore subagent) | Preserve main-thread context quality |
| Risky/broad change, needs cheap rollback | `isolated-worktree` | Main workspace stays clean |
| Trying two competing approaches side by side | `delegated` + `isolated-worktree` | Compare without pollution |

Rules:
- Main thread always owns: charter, final validation, keep/rollback decision, state snapshot.
- Subagents get: narrow scope, clear deliverable, bounded tools. Not "help me optimize."
- Worktrees get: one experiment path. Merge back only after validation passes.

## Autonomous Iteration Contract

Once the target and goal are clear, drive the full loop autonomously: diagnose, design experiments, pick metrics, execute, validate, and iterate. Do not stop to ask the user what to try, how to measure, or how to validate. The user gave you the goal; you own the method.

Exception: stop if the user explicitly asked for analysis only ("先不要改", "analyze only", "just diagnose", "先不要动").

Autonomous means:

- pick the single best next intervention rather than listing many equal options
- design your own experiment: hypothesis, change, metrics, guardrails, stop condition
- select validation methods from the target's structure — tests, diffs, output comparison, metric checks
- execute when the environment allows
- validate every executed change using the validation protocol — run a concrete check, compare against baseline, verify guardrails
- check multiple dimensions when relevant: correctness, performance, compatibility, regressions
- keep, revise, rollback, or switch based on observed evidence, not assumption
- start the next iteration if the goal has not yet been met
- keep the research charter stable unless new evidence justifies reopening it
- preserve a clear boundary between mutable surface and locked evaluator

Do not stop after one recommendation when the next safe action is obvious and executable. Do not pause to ask the user about operational details you can resolve yourself.

## Ratchet and Rollback

Adopt the ratchet property: the working state always represents the current best. Failed experiments do not accumulate.

When git is available, use it naturally: commit before experimenting, revert on failure. Do not create extra tracking files.

After each executed change, declare one:

- `keep` — improved the target without violating guardrails. Commit stays.
- `revise` — right direction but needs refinement. Keep the commit, iterate on it.
- `rollback` — hurt the target, violated guardrails, or delivered no meaningful gain. Revert the commit.
- `switch` — inconclusive; revert and try a different hypothesis.

If an experiment crashes or errors (not just "doesn't improve"), log the failure, revert, and try the next idea. Do not debug endlessly — skip ideas that are broken at the root.

Simplicity criterion: all else being equal, simpler is better. A marginal improvement that adds ugly complexity is not worth keeping.

## Lightweight Execution

This is a thinking methodology, not infrastructure. Keep it weightless:

- Do not create tracking files, ledger files, or planning documents. Use conversation context.
- If a temporary file is unavoidable (e.g., test output), put it in `/tmp` or the project's temp directory — never in a location requiring user permission. Delete it when done.
- Do not leave behind any artifact that the user didn't ask for.
- The skill's footprint after execution should be zero, except for the changes the user wanted.

## Stop Conditions

Continue iterating until one is true:

- the confirmed goal is met
- further change would violate guardrails
- remaining uncertainty cannot be resolved without new user input, authority, or external evidence
- the next iteration has lower expected value than its cost or risk
- the user asks to pause

When stopping, say exactly why.

## Context Preservation

Long tasks cause context loss — the agent forgets the method and starts freestyling. Prevent this with proactive checkpoints, not just reactive drift detection.

**Mandatory reload checkpoints** — re-read the skill and rebuild state at these moments:

1. Before each new iteration cycle (not just the first)
2. After any large tool output that may have pushed the method out of context
3. After returning from a delegated subagent (its results may have shifted focus)
4. When the agent notices it is acting without the target/goal/validation frame

**State snapshot** — write at the end of each iteration, before starting the next:

```text
Target: | Goal: | Surface: | Validation: | Guardrails: | Next Action:
```

Keep it on one or two lines. The snapshot is a reload anchor, not a summary.

**Drift signals** — if any of these appear, pause and reload before continuing:
- Target fuzzes or broadens without explicit re-charter
- Agent proposes changes without stating which experiment they belong to
- Multiple iterations pass without a validation check
- User corrects what is being optimized

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

- "Use $boost to improve our checkout funnel. Map the observability, pick one experiment, and define validation."
- "Use $boost on this prompt chain. I want higher answer quality without increasing token cost."
- "Treat this skill itself as the target object and improve its triggerability, clarity, and evaluation design."
- "Use $boost to turn this workflow into an autoresearch-style loop with a locked evaluator and a visible experiment ledger."
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
