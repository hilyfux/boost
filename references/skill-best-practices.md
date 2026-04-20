# Skill Best Practices

Use this reference when improving the `boost` skill itself or when adapting it across hosts such as Codex and Claude Code.

This file summarizes practical guidance from official Anthropic and OpenAI materials:

- Anthropic Agent Skills best practices: `SKILL.md` is metadata-first, method-second, and reference material lives beside it under `references/`. Lean descriptions and lean main files reduce always-loaded context cost.
- OpenAI prompting and eval guidance: instructions should be clear, specific, example-driven, and iterated with explicit evaluation.
- Anthropic Claude prompting and subagent guidance: explicit instructions, focused subagents, careful trigger wording, and strong context management improve reliability.
- OpenAI Codex guidance: well-scoped issue-like tasks, persistent repo context in `AGENTS.md`, clear verification expectations, and parallel task usage improve output quality.

## Design Principles

### Keep the main file actionable

`SKILL.md` should help an agent act, not just understand. Put the minimum operating method in the main file and move supporting detail into `references/`. Target roughly 300 lines or fewer.

For Claude Code specifically, treat every extra paragraph in `SKILL.md` as competing with conversation history and tool context. Move optional explanation, host-specific tactics, and large example sets into `references/` or `CLAUDE.md`.

### Make the description a trigger surface, not a workflow summary

Frontmatter `description` decides whether the skill fires. Lead with "Use when..." and list symptoms, triggers, and keywords. Do not front-load a workflow summary, because the agent may follow the description instead of reading the main file.

### Prefer explicit selection rules

A skill is easier to trigger correctly when it says:

- when to use it
- when not to use it
- what minimum information it needs
- what a finished pass looks like
- how to resolve the target object when the skill itself is also present in context
- how to confirm the resolved target, goal, validation basis, and execution step before the iteration begins

### Separate method from host metadata

Keep cross-host logic in:

- `SKILL.md`
- `references/`

Treat host-specific files as adapters:

- `agents/openai.yaml` for Codex-side metadata
- `CLAUDE.md`, `AGENTS.md`, or similar files for persistent host-wide or project-wide rules

### Provide concrete examples

Examples reduce ambiguity and improve invocation quality. Prefer examples that look like real user requests, not abstract descriptions of intent.

Claude responds especially well when examples are compact, relevant, and structured. Prefer 3-5 small examples over one giant illustrative block.

### Keep host adapters practical

Host adapter files should answer the questions each host is most likely to need at run time:

- what is the source of truth
- how should work be scoped
- when should the host stay local vs delegate
- how should risky work be isolated
- what counts as adequate validation evidence

Do not turn adapter files into long essays. If guidance is reusable across hosts, move it back into `SKILL.md` or `references/`.

For Claude Code adapters in particular:

- prefer short operational bullets over theory
- use XML-like tags when a compact state packet would reduce ambiguity
- mention built-in `Explore` and `Plan` only for concrete triggers, not as decorative concepts

### Use staged confirmations for ambiguous workflows

If the skill can act on multiple possible targets or outcomes, require confirmations in stages: target, goal, validation, execution. This reduces accidental optimization of the wrong object and makes the iteration loop auditable.

For Claude Code, add `surface` confirmation when the mutable area and evaluator could be conflated.

### Default to action when the user asked for optimization

If the user asked to improve or evolve something, the skill should normally continue past diagnosis into execution, testing, validation, and next-iteration decisions unless the user explicitly asked for analysis only.

### Use execution topology on demand

Optimization work does not always belong in one thread and one checkout. The skill should explicitly choose among:

- direct local execution
- delegated sidecar work through subagents
- isolated experimentation through worktrees or equivalent sandboxed checkouts

The topology choice should be justified by risk, coupling, reversibility, context cost, and exploration efficiency rather than by habit.
Stay local by default. Trigger subagents or worktrees only when they clearly improve throughput, safety, rollback quality, or context isolation.
During exploration, it is often correct to keep multiple plausible paths alive for a while. The role of subagents and worktrees is to make that cheaper and safer, not to add ceremony.

Claude Code-specific note:

- built-in `Explore` is a good fit for read-heavy search, log triage, and evidence gathering that would otherwise pollute the main context
- built-in `Plan` is a good fit after target and goal are clear but sequencing remains uncertain
- custom subagents should be reserved for repeated specialized sidecars with stable prompts

### Separate mutable surface from evaluator

Autonomous optimization gets sloppy when the same loop can freely change both the artifact and the thing that judges the artifact. Borrow the `autoresearch` discipline:

- define the mutable surface explicitly
- lock the evaluator, fixture set, or review basis for the current round whenever possible
- widen scope only by an explicit re-charter, not by drift

This keeps comparisons meaningful and rollbacks auditable.

### Keep a research ledger

A good optimization loop compounds learning. Record:

- baseline
- attempted change
- expected win
- observed result
- keep / revise / rollback / switch decision
- rejected ideas worth not repeating

The ledger can be tiny, but it should survive long enough to prevent rediscovering the same dead ends.

### Design for iteration, not one-shot perfection

Expect the skill to evolve. Improvement should be driven by:

- user corrections
- invocation misses
- ambiguous outputs
- repeated manual cleanup
- validation failures

## Project Layout

This project is small by design. Each file has a distinct purpose:

- `CLAUDE.md` — project-level rules for Claude Code when editing this skill project
- `AGENTS.md` — project-level rules for Codex when editing this skill project
- `SKILL.md` — main operating method, research charter rules, and execution contract (host-agnostic)
- `references/claude-clarification-prompts.md` — reusable question patterns, tagged state packets, and Claude-facing few-shot examples
- `references/eval-fixtures.md` — regression-style fixtures and pass criteria for trigger, clarification, iteration, and anti-self-modification behavior
- `references/evolution-metrics.md` — metric and evaluation patterns used when designing validation
- `references/iteration-template.md` — reusable operating template for one full pass
- `references/iteration-state-snapshot.md` — compact state format for long or noisy conversations, including mutable surface
- `references/skill-best-practices.md` — this file: skill design, host compatibility, heuristics, failure modes, self-maintenance
- `agents/openai.yaml` — Codex-side product metadata for skill discovery and UI display, not method content

Do not over-index on `agents/openai.yaml`. The filename is product-specific but the file is required for Codex to discover and render the skill; leave it in place even when editing from Claude Code.

## Host Compatibility

This skill is written so the same method works across agent hosts, including Codex and Claude Code. Keep the core method in `SKILL.md` and `references/`. Keep host-specific adaptation in the files each host actually reads.

### Shared (cross-host)

- `SKILL.md` and `references/*.md` are the cross-host source of truth. Both hosts read them.
- Do not embed Claude-only or Codex-only tool names, slash commands, or path conventions in `SKILL.md`.
- Keep trigger keywords bilingual (Chinese + English) so both hosts can match user intent.
- Use plain markdown links (`[name](relative/path.md)`) so both hosts can resolve references.

### Claude Code

- Install under `~/.claude/skills/boost/` for user-wide scope or `.claude/skills/<project>/skills/boost/` for project scope.
- Claude Code reads `SKILL.md` frontmatter (`name`, `description`, ≤1024 chars) to decide when to load the skill.
- Project-level `CLAUDE.md` and user-level `~/.claude/CLAUDE.md` hold persistent rules that apply across all conversations. Keep multi-step procedures inside this skill so they load only when relevant.
- Claude Code ignores `agents/openai.yaml`; leaving it present does not cause harm.
- Claude Code supports project and user subagents stored as markdown files under `.claude/agents/` and `~/.claude/agents/`. Use them for focused responsibilities and separate context windows when the optimization loop benefits from parallel exploration, sidecar execution, or context isolation.
- When a risky experiment may disturb the main repo, prefer a git worktree or equivalent isolated checkout so the main workspace remains the source of truth until validation completes.
- Keep Claude-specific persistent guidance in `CLAUDE.md`; do not contaminate `SKILL.md` with Claude-only command language.
- Prefer explicit, normal-language instructions over exaggerated trigger language. Anthropic warns that newer Claude models can overtrigger when prompts are too aggressive.
- Use examples and stable output structures to reduce ambiguity.
- Treat subagents as focused specialists with narrow responsibilities and constrained tools.

### Codex

- Install under `~/.codex/skills/boost/`.
- Codex reads `agents/openai.yaml` for `display_name`, `short_description`, and `default_prompt`. Keep that file present and in sync with the main method.
- Codex also reads `SKILL.md` for the method. The same frontmatter does not break Codex discovery.
- Persistent Codex-wide or project-level rules belong in `AGENTS.md` or the host's equivalent memory file, not inside this skill.
- Codex supports subagent delegation. Use it for well-scoped read, implementation, or verification tasks that can run in parallel with main-thread work or keep alternative exploration paths separated.
- Keep the critical-path integration and final validation in the main thread even when subagents are used.
- Keep `agents/openai.yaml` concise and trigger-oriented; put operational detail in `AGENTS.md`, `SKILL.md`, and `references/`.
- Write Codex-facing instructions as if preparing a concise issue: concrete scope, explicit artifacts, and named validation expectations.
- Keep `AGENTS.md` practical and persistent: navigation hints, source-of-truth rules, validation commands, evidence expectations, and integration constraints.

### Cross-host sync rule

When updating the method:

1. Change the workspace copy first: `/Users/wanglinqing/Desktop/workspace-desktop/skills/boost/`.
2. Validate the skill structure (line count, frontmatter, links, fixtures).
3. Copy updated files into `~/.claude/skills/boost/`.
4. Copy updated files into `~/.codex/skills/boost/`.
5. Verify the installed copies match the workspace source.
6. Do not move the project directory during installation. Always copy.

Never maintain divergent versions of `SKILL.md` across hosts. Divergence defeats the cross-host design.

## Applying the Skill to Itself

This skill may be used on itself, but only as an explicit exception. Do this only when the user clearly names the skill as the target object:

- Object: the `boost` skill
- Goal: improve usefulness, clarity, triggerability, or outcome quality
- Observability: invocation quality, ambiguity, missed cases, validation failures, user corrections
- Optimizations: tighten instructions, clarify outputs, add references, improve evaluation design

If the user asked to optimize a project, repo, workflow, or surrounding artifact, do not switch to self-maintenance. The same object-oriented workflow still applies — the object just happens to be this skill.

## Optimization Heuristics

- Optimize observability before optimizing behavior when the system is opaque.
- Optimize charter clarity before optimization breadth when the change surface is fuzzy.
- Explore before editing; validate before committing; rollback before compounding a bad path.
- Prefer one decisive experiment over many vague suggestions.
- Prefer bounded delegated sidecars over bloating the main context when the work naturally splits.
- Prefer isolated worktrees when a change is broad, conflict-prone, or expensive to roll back in-place.
- Keep multiple plausible paths alive during exploration when doing so increases learning speed more than it increases coordination cost.
- Prefer a narrow mutable surface and a stable evaluator for each iteration.
- Prefer compact tagged state packets when Claude needs to reload the active state in a noisy thread.
- Stay local when extra topology adds ceremony without enough gain.
- Prefer metrics that can be measured repeatedly.
- Use leading indicators for fast feedback and lagging indicators for real outcome validation.
- Treat instrumentation gaps as defects, not just inconveniences.
- Preserve reversibility when confidence is low.
- Separate "did it change?" from "did it improve?".
- Separate local optimization from global optimization; a faster subsystem can still hurt the whole.
- Record why an idea was rejected when the same idea is likely to resurface later.

## Common Failure Modes

Watch for these common mistakes:

- Optimizing the wrong object because the target was never named
- Skipping explicit target confirmation when multiple interpretations are plausible
- Mistaking "use this skill" for "modify this skill"
- Rewriting the skill when the real target object is the surrounding project or artifact
- Stopping at recommendations when the user asked for actual optimization
- Making one change and declaring victory without testing, validation, or a next-iteration decision
- Continuing to stack edits after evidence already says to roll back
- Observing only outcome metrics and ignoring process signals
- Letting the mutable surface silently expand until no experiment is comparable anymore
- Changing the evaluator mid-round and then claiming an improvement
- Confusing anecdotes with evidence
- Shipping changes without a baseline
- Declaring success without guardrails
- Treating every issue as a root-cause problem when some are capacity or policy constraints
- Making the loop so heavy that iteration becomes slower than the value created
- Writing a Claude-facing skill as a long essay instead of an operational guide
- Editing installed host copies first and forgetting the workspace source of truth
- Serializing clearly parallel sidecar work in the main thread until context quality collapses
- Running a risky broad experiment directly in the main workspace when isolation was cheap
- Letting `SKILL.md` grow past the soft size limit so the always-loaded cost climbs on every invocation

## Prompt and Skill Quality Checks

Use these checks when editing the skill:

- Is the trigger surface specific enough to activate reliably?
- Is the method concise enough to scan quickly?
- Does the Claude-facing path stay concise, structured, and example-backed?
- Are there concrete examples of correct invocation?
- Is there a reusable output structure?
- Are evals or validation expectations explicit?
- Are host-specific details separated from the cross-host method?
- Do host adapters tell Claude and Codex what they each need without duplicating the full shared method?
- Can the agent tell when to stop, defer, or hand off?
- Can the agent tell when to stay local, when to delegate, and when to isolate in a worktree?
- Does the skill force a target confirmation step before modifying anything ambiguous?
- Does the skill force a surface confirmation step before broad edits or benchmark changes?
- Does the skill preserve experiment memory so failed branches are not rediscovered?
- Does the skill collect and resolve open questions before locking the goal and execution plan?
- Does the skill define what happens after validation: keep, revise, rollback, or switch?
- Does the skill continue iterating until a goal or explicit stop condition is reached?
- Does `SKILL.md` stay under roughly 300 lines so always-loaded context stays small?
- Does the description lead with "Use when..." rather than a workflow summary?

## Evaluation Discipline

Do not improve the skill by taste alone. Track observable indicators such as:

- trigger accuracy
- number of times the user has to restate the target object
- rate of accidental self-modification when the target was actually a project or artifact
- rate of iterations started before target, goal, and validation were explicitly confirmed
- rate of vague recommendations with no experiment
- rate of optimization requests that stop before any executed validation loop
- rate of missing metrics or guardrails
- usefulness of the first actionable step
- host drift between Codex and Claude installs
- drift between `CLAUDE.md`, `AGENTS.md`, and the shared method
- rate of Claude-side overtriggering caused by overly aggressive host instructions
- rate of Codex tasks that fail because project context or validation expectations were too vague
- rate of missed delegation opportunities on clearly parallelizable work
- rate of risky experiments executed in the main workspace without isolation

When changing the skill, compare:

- before and after invocation quality
- before and after output completeness
- whether the new wording reduces ambiguity
- whether the new structure makes execution faster

Support these comparisons with the regression fixtures in `references/eval-fixtures.md`. A good skill project should not rely only on prose principles; it should also preserve a small set of stable behavioral checks.

## Safe Maintenance Pattern

When updating this skill:

1. Commit before experimenting — this project is tracked by git, which is the primary rollback path. Manual `.bak` files are optional extra insurance.
2. Change the workspace copy first.
3. Validate the skill structure: line count, frontmatter, link integrity, fixture walk-through.
4. Sync the installed copies by copying, not moving.
5. Verify the installed copies match the workspace source.
6. Remove the backup only after the sync has been verified.

## Anti-Patterns

Avoid these common failures:

- making the trigger description too broad and generic
- letting the skill treat its own presence in context as evidence that it is the target object
- letting the agent start editing before it has confirmed which object is being optimized
- letting the agent start iterating before unresolved questions, goals, and validation criteria are confirmed
- treating optimization as advice generation instead of a continuing execution loop
- forcing a topology ceremony before exploration has revealed whether parallel paths or isolation are actually useful
- forcing all execution through the main thread even when subagents or worktrees would reduce risk or context loss
- triggering subagents or worktrees by default instead of by need
- stuffing host-specific setup into the main method
- adding long theory sections with no operational value
- ending with advice instead of an experiment or action
- maintaining divergent versions across hosts
- moving the project directory when only installation sync is needed
- deleting `agents/openai.yaml` because the filename looks product-specific; Codex needs it
- turning `AGENTS.md` or `CLAUDE.md` into a duplicate copy of the full method
