# Claude Clarification Prompts

Use this reference when Claude Code needs compact, repeatable wording to confirm the target, goal, validation basis, or execution scope before starting an iteration.

Keep questions short. Ask only what is needed to unblock the next step.

For Claude Code, prefer compact tagged packaging when the thread is long or the prompt mixes state, instructions, and examples.

## 1. Target Confirmation

Use when the target object could be interpreted in more than one way.

### Direct confirmation

`Confirmed Target: I will optimize <target>, not <wrong alternative>.`

### Ask when ambiguous

`Before I start, please confirm the target: should I optimize <option A> or <option B>?`

### Ask when the loaded skill is a tempting wrong target

`To confirm, do you want me to optimize the current project/artifact, or do you want me to optimize the self-evolution skill itself?`

## 2. Goal Confirmation

Use when the user asks to "optimize", "improve", or "make it better" without a clear outcome.

### Direct confirmation

`Confirmed Goal: Success means <desired outcome>, not merely <shallow proxy>.`

### Ask when vague

`What does "better" mean here: higher quality, lower cost, faster execution, stronger compatibility, or something else?`

### Ask when two outcomes conflict

`Which goal should I prioritize first: <goal A> or <goal B>?`

## 3. Constraint Confirmation

Use when the likely optimization could affect scope, compatibility, or safety.

### Direct confirmation

`Confirmed Constraints: I must preserve <constraint 1> and <constraint 2> while improving <goal>.`

### Ask when scope is unclear

`Should I keep full backward compatibility with the current behavior, or can I change the workflow if it improves the result?`

## 4. Surface Confirmation

Use when Claude might accidentally broaden scope or alter the evaluator.

### Direct confirmation

`Confirmed Surface: I will change <mutable surface>, not <locked evaluator or out-of-scope surface>.`

### Ask when scope may drift

`Should I only change <mutable surface>, or is <possible broader surface> also in scope?`

### Ask when the evaluator should stay fixed

`Should I treat <benchmark / review basis / fixture set> as locked while I optimize the target?`

## 5. Validation Confirmation

Use when success criteria are not yet explicit.

### Direct confirmation

`Confirmed Validation: We will judge improvement by <metric or evidence>, while protecting <guardrails>.`

### Ask when no metric is available

`What should count as proof that this got better: a measurable metric, fewer user corrections, clearer output, or another signal?`

### Ask when guardrails are missing

`What must not get worse while I optimize this?`

## 6. Execution Confirmation

Use right before editing files or starting an experiment.

### Direct confirmation

`Confirmed Execution: Next I will <specific action> on <target scope>.`

### Ask when action scope is unclear

`Should I only redesign the method and prompts, or should I also change project-level files such as CLAUDE.md and installed copies?`

## 7. Minimal Question Set

When the request is ambiguous, prefer this order and stop as soon as the iteration becomes safe:

1. `What exactly is the target object?`
2. `What outcome should improve first?`
3. `What surface can change, and what should stay locked?`
4. `What must stay unchanged?`
5. `How should we judge success?`
6. `Am I cleared to make the changes directly once confirmed?`

## 8. Tagged State Pattern

Use this when the thread is busy and Claude needs a compact reloadable state:

```text
<target>...</target>
<goal>...</goal>
<surface>...</surface>
<validation>...</validation>
<next_action>...</next_action>
```

## 9. Good Interaction Pattern

Use this compact sequence:

```text
Confirmed Target: I believe the target is <target>, not <wrong alternative>.
Open Questions:
1. <smallest unresolved question>
2. <next unresolved question>

After you confirm these, I will lock:
- Goal
- Surface
- Validation basis
- Execution plan
```

## 10. Few-Shot Examples

Use these as style anchors, not rigid templates.

<examples>
<example>
User: 用这个 skill 优化一下当前项目。
Assistant:
<target>当前项目</target>
<goal>Unknown</goal>
<surface>Unknown</surface>
<next_action>Ask for the primary success outcome</next_action>

What outcome should improve first: quality, speed, cost, or something else?
</example>

<example>
User: 参考 autoresearch 优化这个 prompt workflow，但别动评测脚本。
Assistant:
<target>prompt workflow</target>
<goal>Improve the workflow using an autoresearch-style loop</goal>
<surface>Change the prompt workflow only</surface>
<validation>Keep the evaluation script fixed for this round</validation>
<next_action>Inspect the workflow and design one constrained experiment</next_action>
</example>

<example>
User: 把 self-evolution 这个 skill 本身当目标，优化它在 Claude Code 里的表现。
Assistant:
<target>self-evolution skill</target>
<goal>Improve Claude Code execution quality</goal>
<surface>Edit workspace skill files and Claude-specific adapters first</surface>
<validation>Judge by clearer triggering, fewer ambiguous branches, and better Claude-specific guidance</validation>
<next_action>Read SKILL.md, CLAUDE.md, and Claude-facing references before editing</next_action>
</example>
</examples>

## 11. Anti-Patterns

Avoid these question styles:

- broad discovery prompts with too many asks at once
- questions that can be answered from local context
- abstract questions with no operational consequence
- repeating confirmations after the user already answered clearly
