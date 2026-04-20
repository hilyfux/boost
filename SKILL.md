---
name: boost
description: Use when the user asks to optimize, improve, iterate, diagnose, evolve, monitor, stabilize, raise quality, reduce cost, reduce failure, raise conversion, or help an object get better over time. Triggers on software systems, workflows, prompts, skills, team processes, product surfaces, services, datasets, content pipelines, agents, support flows, or any observable artifact with a goal. Also triggers in Chinese on 优化、持续进化、监控问题、提出优化方案、定义验证指标、验证效果、迭代改进、提升质量、降低成本、减少失败、提高稳定性、提高转化. Structures an observe-diagnose-optimize-validate-iterate loop with explicit target, goal, validation, mutable-surface, and execution confirmation gates. The target can be this skill itself only when the user explicitly names it.
---

# Boost

Generic optimization loop for any target with observable behavior and a goal.

## Hard Rules

These are not suggestions. Violating any is a failure.

1. Follow Steps 1–7 in order. Do not skip or reorder.
2. Each step MUST produce visible output under its step heading.
3. Do not edit anything before Step 3 (Observe) is complete.
4. Do not declare success without evidence from Step 6 (Validate).
5. The target is never this skill unless the user explicitly names it as target.
6. If the user says "先不要改" / "analyze only" / "just diagnose", stop after Step 4.
7. After each executed change, decide: keep / rollback / next. Do not stack unvalidated changes.

## Step 1: Target

Name the object being optimized.

- If the user names a concrete object, that is the target.
- If the user says "当前项目" / "this project", the target is the surrounding project, not this skill.
- If ambiguous, ask one question to disambiguate. Only one.

Output:

> **Confirmed Target:** <the object you will optimize>

Do not proceed until the target is named.

## Step 2: Goal

State what success means in concrete terms.

If the user's intent is vague ("优化一下", "make it better"), ask 1–2 sharp questions:

- "最让你头疼的问题是什么？" / "What hurts most?"
- "如果只改一件事，你最希望看到什么变化？" / "If you could fix one thing, what would it be?"

Do not ask more than 2 questions. Do not ask about how to validate or what metrics to track — figure those out yourself.

Output:

> **Confirmed Goal:** <what success looks like>

Do not proceed until the goal is actionable.

## Step 3: Observe

Read the actual artifact using tools. Do not guess from memory. Capture the current state as baseline before any change.

Output:

> **Baseline:** <specific observations about current state>

Do not propose or execute changes until this step is complete.

## Step 4: Diagnose

Build a problem model: Symptom → Impact → Cause → Confidence → Evidence.

Pick the single highest-leverage intervention. Do not present a menu of equal options — choose one and justify it.

Output:

> **Diagnosis:** <the problem, with evidence>
>
> **Intervention:** <one specific change and why>

## Step 5: Execute

Apply the change. State exactly what you changed and where.

If the user requested analysis only, stop here and output the plan without executing.

Output:

> **Executed:** <what was changed, on what scope>

## Step 6: Validate

Run a concrete check. Read the changed file, run tests, diff before/after, or produce output and inspect it. "I believe it improved" is not validation — show evidence.

Output:

> **Checked:** <what you verified and how>
>
> **Before → After:** <concrete comparison>
>
> **Decision:** keep / rollback / next — with evidence

## Step 7: Next

If the goal is met, state why and stop.

If not, emit the visible state checkpoint and loop back to Step 3:

> **[boost · Iter N]** 上轮：<result> → 本轮：<next target>

### Stop when

- The goal is met
- The user asks to pause
- Further change would cause more harm than benefit
- You need information only the user can provide

State exactly why you are stopping.

## When to Use

Use when the user wants to improve, optimize, diagnose, or iterate on something and no more specialized skill covers the target.

Do not use for explanation-only, brainstorming-only, or clerical tasks.

## References

Consult these for deeper methodology when a step needs more context:

- [references/methodology.md](references/methodology.md) — full methodology: research frames, topology, autonomous iteration, validation protocol, ratchet/rollback patterns
- [references/iteration-template.md](references/iteration-template.md) — structured template for one pass
- [references/iteration-state-snapshot.md](references/iteration-state-snapshot.md) — compact state for long threads
- [references/eval-fixtures.md](references/eval-fixtures.md) — regression checks for skill behavior
- [references/evolution-metrics.md](references/evolution-metrics.md) — metric and evaluation patterns
- [references/skill-best-practices.md](references/skill-best-practices.md) — skill design and host adaptation guidance
