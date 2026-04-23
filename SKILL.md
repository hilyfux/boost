---
name: boost
version: "1.4.0"
description: Use when the user asks to optimize, improve, iterate, diagnose, evolve, monitor, stabilize, raise quality, reduce cost, reduce failure, raise conversion, or help an object get better over time. Triggers on software systems, workflows, prompts, skills, team processes, product surfaces, services, datasets, content pipelines, agents, support flows, or any observable artifact with a goal. Also triggers in Chinese on 优化、持续进化、监控问题、提出优化方案、定义验证指标、验证效果、迭代改进、提升质量、降低成本、减少失败、提高稳定性、提高转化. Structures a strict PDCAA loop (Plan-Do-Check-Align-Act) anchored by a Stable Contract, with AutoResearch, AutoReceive, and Log as built-in enhancements. The target can be this skill itself only when the user explicitly names it.
hooks:
  PreToolUse:
    - matcher: "Bash|Read|Grep|Glob"
      hooks:
        - type: command
          command: "python3 -c 'import pathlib,json;f=pathlib.Path(\"/tmp/boost-drift-counter\");c=int(f.read_text())+1 if f.exists() else 1;f.write_text(str(c));print(json.dumps({\"additionalContext\":\"[boost] 已连续 \"+str(c)+\" 次探索性调用。快速自检：当前在 PDCAA 哪个阶段？Stable Contract 是否仍然锚定？如果不确定，暂停并重建状态。\"})) if c>=10 else None'"
          timeout: 5
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "python3 -c 'import pathlib,json;pathlib.Path(\"/tmp/boost-drift-counter\").write_text(\"0\");print(json.dumps({\"additionalContext\":\"[boost] 文件已修改。继续前必须回答：(1) 当前在 PDCAA 哪个阶段？目标是什么？这次编辑属于哪个实验？(2) Check 阶段——读取变更文件，对比前后，展示证据。如果无法回答 (1)，立即 STOP 并重新打开 SKILL.md 重建状态。\"}))'"
          timeout: 5
  Stop:
    - hooks:
        - type: command
          command: "python3 -c 'import pathlib,json;pathlib.Path(\"/tmp/boost-drift-counter\").unlink(missing_ok=True);print(json.dumps({\"decision\":\"approve\",\"reason\":\"[boost] 回合结束前检查：是否已输出完整 log_summary？Stable Contract 目标是否达成？如果未完成，说明停止原因。\"}))'"
          timeout: 5
---

# Boost

Execution-oriented optimization skill. Core methodology: **PDCAA** (Plan-Do-Check-Align-Act), anchored by a Stable Contract, enhanced by AutoResearch, AutoReceive, and Log.

The goal is not to answer questions but to produce stable, effective results through disciplined iteration.

## Hard Rules

These are not suggestions. Violating any is a failure.

1. Establish the Stable Contract before entering the PDCAA loop. Do not skip.
2. Each PDCAA cycle MUST produce visible output with the 11 mandatory log fields.
3. Do not edit anything before the first Plan phase reads the Stable Contract.
4. Do not declare success without evidence from the Check phase.
5. The target is never this skill unless the user explicitly names it as target.
6. If the user says "先不要改" / "analyze only" / "just diagnose", stop after the first Check.
7. After each Do, decide in Act: continue / adjust / rollback / research / receive_update / complete. Do not stack unvalidated changes.
8. Align phase is mandatory every cycle — skip detection is a failure.
9. If Align detects drift, perform Re-anchor before the next Plan: re-read the Stable Contract, clear invalid local state, restore checks, and regenerate the minimum action.

## Stable Contract

Before entering the loop, establish and confirm these four fields:

| Field | Definition |
|-------|-----------|
| **Goal** | The single objective being pursued |
| **Constraints** | Hard boundaries that must not be violated |
| **Done** | Concrete definition of completion |
| **Checks** | Mandatory verification items |

Output:

> **Stable Contract**
> - Goal: <唯一目标>
> - Constraints: <不可违反的约束>
> - Done: <完成定义>
> - Checks: <必检项>

Rules:
- If the user names a concrete object, that is the target.
- If the user says "当前项目" / "this project", the target is the surrounding project, not this skill.
- If ambiguous, ask one question to disambiguate. Only one.
- If the goal is vague ("优化一下", "make it better"), ask 1–2 sharp questions. No more.
- Do not proceed until all four fields are filled.

## Runtime State

Tracks mutable per-cycle state. Separate from the Stable Contract — never confuse the two.

```yaml
runtime_state:
  current_step: ""
  observations: []
  local_issues: []
  feedback: []
  validation_result: {}
  next_candidates: []
  drift_flags: []
```

Runtime State is updated freely during execution. The Stable Contract is not.

## Default Path vs Slow Path

Default mode is the lightweight PDCAA loop below. The slow path is an optional upgrade protocol, not the default behavior.

Trigger a slow-path gate only when the fast path stops being cheap or trustworthy:
- **Design Gate** — ambiguous scope, multiple subsystems, or broad multi-file changes before the first real Plan
- **Execution Gate** — rollback-sensitive work, competing approaches, or context-heavy sidecar work that benefits from worktrees or delegated execution
- **Quality Gate** — before declaring complete when the change is broad, risky, or user-visible enough to justify stronger verification or review

These gates refine execution topology and rigor around PDCAA. They do not replace AutoResearch, AutoReceive, or Log.

## PDCAA Loop

Execute in strict order every cycle. Each phase produces visible output.

### 1. Plan

- Read Goal / Constraints / Done / Checks from the Stable Contract
- Define the current minimum action
- The action must: directly serve the goal, be low-risk, be verifiable

Output:

> **Plan:** <当前最小动作及其理由>

### 2. Do

- Execute the current minimum action
- Record: execution result, new observations, new questions

Output:

> **Do:** <执行了什么，结果是什么>

### 3. Check

Answer all five questions:

1. Did this action advance the goal?
2. Did it violate any constraint?
3. Did it introduce new problems?
4. Is the result verifiable with evidence?
5. Does it preserve the ability to check in subsequent cycles?

Run a concrete verification: read the changed file, run tests, diff before/after, or produce output and inspect. "I believe it improved" is not validation — show evidence.

Output:

> **Check:**
> - Goal advancement: yes/no — <evidence>
> - Constraint violation: none / <which one>
> - New problems: none / <what>
> - Verifiable: yes/no — <how verified>
> - Future checkability: preserved / degraded — <note>

### 4. Align

Detect drift across five dimensions:

| Drift Type | Signal |
|-----------|--------|
| Goal drift | Target fuzzes or broadens without re-charter |
| Constraint drift | Constraints silently relaxed or ignored |
| Check drift | Verification items skipped or weakened |
| Done-criteria drift | Completion bar moves without explicit agreement |
| Local-problem hijack | A side issue displaces the main objective |

If any drift is detected:

1. Re-read the Stable Contract
2. Clear invalid local state
3. Restore check items
4. Regenerate the current minimum action

Output:

> **Align:** <drift detected: none / type + correction taken>

### 5. Act

Based on Check and Align results, choose exactly one:

| Decision | When |
|----------|------|
| `continue` | Goal advancing, no drift, next step clear |
| `adjust` | Right direction but approach needs refinement |
| `rollback` | Violated constraint, introduced regression, or no gain |
| `research` | Path unclear, trigger AutoResearch |
| `receive_update` | New external input requires integration |
| `complete` | Done criteria met with evidence |

Output:

> **Act:** <decision> — <reason>

If not `complete`, emit the iteration checkpoint and loop back to Plan:

> **[boost · Iter N]** Target: <target> | Goal: <goal> | 上轮: <result> → 本轮: <next focus>

## AutoResearch

On-demand exploration, not a standing committee. Distilled from Karpathy's `autoresearch` pattern: an autonomous loop where the agent owns hypothesis formation, experiment execution, and result evaluation — advancing only on evidence, never on intuition.

### Core principles (from autoresearch)

1. **Separated concerns** — every research loop needs three clearly separated roles: a charter (Stable Contract), a mutable surface (the thing you change), and a locked evaluator (the thing that judges the change). When the same loop can freely change both artifact and evaluator, all comparisons become meaningless.
2. **Ratchet property** — the working state always represents the current best. Improvements are committed; failures are reverted. No half-tested changes accumulate. Single lineage, not population diversity.
3. **Fixed comparison budget** — each probe gets a bounded effort. This makes results comparable across experiments rather than biased by how long each one ran. Not "try A for an hour, B for five minutes" but "each gets the same budget."
4. **Simplicity criterion** — a tiny improvement that adds significant complexity is rejected. Complexity degrades across sessions; guard against it explicitly.
5. **Crash resilience** — failed experiments are logged, reverted, and skipped. Do not debug endlessly — log the failure, revert, try the next hypothesis.
6. **Autonomous continuity** — once the charter is set, the agent drives the full loop without pausing for permission. The human's role is research advisor, not operator.

### Trigger conditions

Activate only when:
- Path is unclear — cannot define a verifiable minimum action
- Multiple conflicting approaches — 2+ viable paths with unclear tradeoffs
- Consecutive low-gain cycles (2+) — the fast path is exhausted
- Results contradict expectations — evidence invalidates the working model
- Actions repeatedly rejected — the current direction is blocked

### Process

1. List candidate approaches (max 3)
2. Compare each against Stable Contract — which best serves Goal within Constraints?
3. Propose a minimum probe action — the smallest experiment that distinguishes candidates
4. Return to Plan with the probe. AutoResearch does NOT make keep/rollback decisions or modify the Stable Contract.

### Schema

```yaml
research_result:
  triggered: true|false
  reason: ""
  options:
    - action: ""
      expected_gain: ""
      risk: ""
  recommended_probe: ""
```

### Output

> **AutoResearch:**
> - Trigger: <why activated>
> - Candidates: <action — expected gain — risk> (per candidate)
> - Probe: <minimum experiment to distinguish candidates>
> - Recommendation: <which candidate and why>

## AutoReceive

The always-on listener. Grounded in cybernetic feedback theory (Wiener): a control system that classifies incoming signals, routes them by impact, and protects the stable boundary from uncontrolled mutation.

### Core principles (from cybernetics)

1. **Negative feedback (stabilizing)** — when the system deviates from the Stable Contract (goal drift, constraint violation), corrective signals pull it back. This is the primary function: detect deviation, trigger correction. The Stable Contract is the homeostatic setpoint.
2. **Positive feedback (amplifying)** — when evidence confirms the current direction (Check passes, metrics improve), the signal reinforces. Not all feedback is corrective; confirmations matter too.
3. **Wiener filter (signal vs. noise)** — not every incoming signal warrants action. Separate actionable feedback from noise. "Interesting but unclear" is a valid classification — record it, don't act on it.
4. **Requisite variety (Ashby)** — the classification system must match the complexity of disturbances it faces. If all feedback is bucketed as "high/low", nuanced signals get misrouted. The schema below provides enough variety for typical optimization loops.
5. **Reflexivity (von Foerster)** — the system's own Check results, Align drift signals, and Log patterns are valid feedback. A pattern of consecutive `adjust` decisions is itself a signal. The system must observe its own observation process.

### Classification

```yaml
feedback_event:
  type: new_requirement | new_constraint | execution_error | tool_feedback | validation_feedback | new_observation | env_change
  priority: low | medium | high
  source: user | system | self
  payload: ""
```

### Routing

| Tier | Signal type | Integration point | Timing |
|------|-----------|-------------------|--------|
| **Immediate** | Errors, constraint violations, user corrections | Current Check/Align | This cycle |
| **Deferred** | New observations, low-priority suggestions | Next Plan | Next cycle |
| **Escalated** | New requirements, new constraints, evidence that Done/Constraints are wrong | Contract Candidate | After explicit marking |

Default behavior: update Runtime State only. Do NOT modify Stable Contract directly.
New requirements and new constraints enter the loop as feedback first; crossing into the Stable Contract requires explicit escalation.

If modification to Stable Contract is warranted, mark explicitly:

> **Contract Candidate:** <proposed change to which field> — <evidence/reason>

Apply only after user confirms or evidence is overwhelming.

## Log

Every PDCAA cycle MUST record:

```text
round_id:         <cycle number, e.g. 1, 2, 3>
goal:             <current goal summary>
current_action:   <what was done>
action_reason:    <why this action>
execution_result: <what happened>
check_result:     <verification outcome>
align_result:     <drift signal or none>
feedback:         <any received input>
decision:         <continue/adjust/rollback/research/receive_update/complete>
next_step:        <what happens next>
log_summary:      <one-line digest of this cycle>
```

This log is the mandatory cycle output. It appears in the visible response, not hidden.

## Parallelization & Isolation Defaults

Max-efficiency mode: default to parallel / isolated / delegated; skip only with a stated reason.

| Scenario | Default | Skip only when |
|----------|---------|----------------|
| Plan baseline capture | `Agent(subagent_type=boost-observer)` for boost-specific targets; `Agent(subagent_type=Explore)` for general codebase | Target is 1 file < 200 lines |
| Do: 2+ independent files/modules | `superpowers:dispatching-parallel-agents` | Explicit ordering dependency between files |
| AutoResearch with ≥ 2 candidates | One worktree + one `Agent` per candidate | Candidates differ only in a parameter value |
| Check on non-trivial change | Parallel: diff-read + test-run + regression-spot | Change is 1 line |
| ≥ 3 files touched or rollback-sensitive | `superpowers:using-git-worktrees` | Trivially revertible (< 30s manual revert) |
| `complete` on non-trivial change | `superpowers:requesting-code-review` + `Agent(subagent_type=superpowers:code-reviewer)` | Pure format / typo |

Skipping a default requires an entry in the Log's `action_reason` field naming which default was skipped and why.

"Non-trivial change" = any single change that touches ≥ 2 files OR ≥ 20 lines OR user-facing behavior OR touches `SKILL.md`. Trivial = everything else (typos, comment-only, single-line internal tweaks).

## Execution Topology

Use the lightest topology that fits. Trigger heavier ones by condition, not by habit. Under max-efficiency mode (see Parallelization & Isolation Defaults), the heavier rows are the default, not the exception.

| Condition | Topology | How (concrete) | Guardrail |
|-----------|----------|----------------|-----------|
| Single-scope edit, next step obvious | `local` | Main thread uses Edit/Write directly | — |
| Read-heavy baseline (≥ 3 files or large logs) | `delegated` | `Agent(subagent_type=Explore)` or `Agent(subagent_type=boost-observer)` | Read-only; restate Contract on return |
| 2+ independent explorations | `delegated` (parallel) | `superpowers:dispatching-parallel-agents` | Each subagent has one bounded deliverable |
| Competing approaches / second opinion | `delegated` (parallel) | Parallel `Agent(challenger)` + `Agent(innovator)` + `Agent(pragmatist)` | Decision authority stays on main thread |
| Risky / broad change, cheap rollback needed | `isolated-worktree` | `superpowers:using-git-worktrees` (never hand-write `git worktree`) | Merge only after Check passes |
| Side-by-side comparison of 2 approaches | `delegated` + `isolated-worktree` | Two `Agent(..., isolation: "worktree")` calls | Equal budget; locked evaluator |
| Non-trivial Check | `delegated` (parallel) | Diff-read + `Agent(Explore)` regression scan + `Bash` test run | Results merge on main thread; no delegated keep/rollback |

Rules:

- Main thread always owns: Stable Contract, Check evidence review, Act decision, state snapshot. Subagents and worktrees never own keep/rollback.
- After any subagent / worktree returns, restate `<target> / <goal> / <baseline> / <next_action>` before using its result.

## Stop When

- Done criteria met (with evidence)
- User asks to pause
- Further change would cause more harm than benefit
- Information needed that only the user can provide

State exactly why you are stopping. Do not use for explanation-only, brainstorming-only, or clerical tasks.

## References

Consult these for deeper methodology when a phase needs more context:

- [references/methodology.md](references/methodology.md) — full methodology: research frames, topology, autonomous iteration, validation protocol, ratchet/rollback patterns
- [references/iteration-template.md](references/iteration-template.md) — structured template for one PDCAA cycle
- [references/iteration-state-snapshot.md](references/iteration-state-snapshot.md) — compact state for long threads
- [references/eval-fixtures.md](references/eval-fixtures.md) — regression checks for skill behavior
- [references/evolution-metrics.md](references/evolution-metrics.md) — metric and evaluation patterns
- [references/skill-best-practices.md](references/skill-best-practices.md) — skill design and host adaptation guidance
- [references/quick-reference.md](references/quick-reference.md) — minimal system prompt and quick start checklist
- [references/claude-clarification-prompts.md](references/claude-clarification-prompts.md) — reusable question patterns and few-shot examples for target/goal discovery
- [references/host-adaptation-best-practices.md](references/host-adaptation-best-practices.md) — Claude Code and Codex host adaptation guidance
- [references/research-loop-notes.md](references/research-loop-notes.md) — Karpathy's autoresearch: three-file architecture, ratchet, fixed budget, simplicity criterion
- [references/feedback-loop-notes.md](references/feedback-loop-notes.md) — cybernetic feedback theory: Wiener, Ashby, von Foerster applied to AutoReceive
