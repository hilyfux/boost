# Boost Orchestration Upgrade — Design Spec

- Date: 2026-04-23
- Author: linqing.wang
- Target: `boost` skill (this project)
- Version bump: `1.3.0 → 1.4.0` (minor — additive, backwards compatible)

## Problem

The current `boost` skill describes execution topology in abstract terms (`local` / `delegated` / `isolated-worktree`) and makes **zero** mention of the `superpowers:*` skills that Claude Code already exposes. Three consequences:

1. **Vague tooling.** A reader of `SKILL.md` knows *when* to delegate but not *what to call*. The Execution Topology table stops at concepts; it never names the `Agent` tool, the `Explore` subagent, or `superpowers:using-git-worktrees`.
2. **Zero skill orchestration.** None of `superpowers:brainstorming`, `systematic-debugging`, `test-driven-development`, `writing-plans`, `using-git-worktrees`, `dispatching-parallel-agents`, `verification-before-completion`, `requesting-code-review`, or `finishing-a-development-branch` appear in `SKILL.md`. Each PDCAA phase has natural handoff points to these skills, but the mapping is missing.
3. **Under-utilized parallelism.** Subagent/worktree usage is framed as an opt-in upgrade. Under a "max efficiency" operating mode — which the user explicitly requested — the default must flip: parallelize whenever plausible, isolate whenever rollback-sensitive, delegate whenever the main-thread context would otherwise be flooded.

## Goal

Make `boost` efficiently dispatch Claude Code's subagent and worktree capabilities **by default**, and explicitly coordinate with the `superpowers:*` skills that amplify each PDCAA phase — without breaking the Stable Contract, PDCAA structure, Log format, or backwards compatibility with host adapters (Claude Code, Codex).

## Non-goals

- Do not change the Stable Contract schema, the 11 Log fields, or the 5 Align drift types.
- Do not introduce new agent files in `agents/`; we reuse existing globals (`Explore`, `challenger`, `innovator`, `pragmatist`, `code-reviewer`) plus the existing `boost-observer`.
- Do not rewrite `references/methodology.md` or other theory references.
- Do not enforce TDD / code review on trivial changes (`complete` on 1-line typo stays local).

## Approach (Plan 1 from brainstorm)

Per-phase signposts in `SKILL.md` + a dedicated `references/orchestration.md` that carries the tacit-knowledge payload.

## Stable Contract for this work

- **Goal:** `boost` v1.4.0 routinely — not conditionally — uses subagents, worktrees, and `superpowers:*` skills at the right PDCAA moments.
- **Constraints:**
  - No schema breaks (Stable Contract fields, Log fields, Align drift types).
  - `SKILL.md` stays readable as a skeleton — information density must rise, ceremony must not.
  - Main thread retains all Stable Contract, Check, and Act authority (no delegation of keep/rollback decisions).
  - Sync rule applies: after validation, copy to `~/.codex/skills/boost/` and `~/.claude/skills/boost/`, copy `agents/boost-observer.md` to `~/.claude/agents/`.
- **Done:**
  - `SKILL.md` frontmatter `version: "1.4.0"`.
  - Each PDCAA phase carries an explicit `Handoff (default; skip only when marked)` block naming concrete skills/tools.
  - The Execution Topology table has 4 columns (`Condition / Topology / How (concrete) / Guardrail`) and 7 rows (one added for Check-phase parallel).
  - A new `Parallelization & Isolation Defaults` mini-section precedes the Execution Topology table.
  - `references/orchestration.md` exists with § 1–6 as outlined below, ≤ 240 lines.
  - `eval-fixtures.md` gains ≥ 1 fixture that exercises the default-parallel behavior and ≥ 1 that exercises the skip condition.
- **Checks:**
  - Read updated `SKILL.md` end-to-end; confirm no broken cross-references and frontmatter validates.
  - Grep for the exact tool names (`superpowers:using-git-worktrees`, `superpowers:dispatching-parallel-agents`, etc.) — each should appear in `SKILL.md` at least once.
  - Walk through existing `eval-fixtures.md` mentally; nothing regresses.
  - Word-count `orchestration.md`; ensure it stays under the 240-line budget.
  - Verify `SKILL.md` frontmatter `version` is `"1.4.0"`.

## Changes — SKILL.md

Net delta: **+40~45 lines** (from 332 → ~375). Acceptable break of the 300-line soft limit; density remains high because additions are table rows and one-line handoff bullets, not prose.

### Change 1: Frontmatter `version` bump

`"1.3.0" → "1.4.0"` — minor bump per the project's Versioning Rule (new capability, new reference file, backwards compatible).

### Change 2: Per-phase `Handoff` blocks

Append one `Handoff (default; skip only when marked)` block to each of the five PDCAA phases. Language deliberately reads as **default behavior** (not "consider this"), forcing an explicit skip reason in the Log when bypassed.

**Plan:**

```
Handoff (default; skip only when marked):
- Baseline capture → default Agent(subagent_type=boost-observer) when target is this skill or boost-specific artifact; default Agent(subagent_type=Explore) for general codebase reads; skip only when target is a single file < 200 lines
- Scope ambiguous / multi-subsystem → default superpowers:brainstorming before the first Do
- Multi-step implementation → default superpowers:writing-plans
- Stuck on design choice → enter AutoResearch (see below)
```

**Do:**

```
Handoff (default; skip only when marked):
- Feature / behavior change on code → default superpowers:test-driven-development
- Bug / test failure / unexpected behavior → default superpowers:systematic-debugging
- Broad (≥ 3 files) or rollback-sensitive change → default superpowers:using-git-worktrees
- 2+ independent tasks / files → default superpowers:dispatching-parallel-agents
```

**Check:**

```
Handoff (default; skip only when marked):
- Non-trivial change → default parallel sub-checks (diff-read + test-run + regression-spot)
- About to claim "complete / fixed / passing" → default superpowers:verification-before-completion
- Evidence needs independent read → Agent(subagent_type=superpowers:code-reviewer) or Agent(subagent_type=challenger) as a second opinion
```

**Align:**

```
Handoff (default; skip only when marked):
- Drift triggered by 2+ signals in the same cycle → re-read Stable Contract AND consider superpowers:brainstorming to re-charter
- AutoReceive marked Contract Candidate → pause PDCAA; run explicit escalation protocol before continuing
```

**Act:**

```
Handoff (default; skip only when marked):
- decision == complete on non-trivial change → default superpowers:requesting-code-review
- complete + ready to merge / PR → default superpowers:finishing-a-development-branch
- decision == research → AutoResearch; when ≥ 2 candidates, parallel Agent(innovator) + Agent(pragmatist) + Agent(challenger) as reviewer
```

### Change 3: New mini-section "Parallelization & Isolation Defaults"

Placed between the existing Execution Topology heading and the topology table. Establishes the flipped default as a first-class rule.

```
## Parallelization & Isolation Defaults

Max-efficiency mode: default to parallel / isolated / delegated; skip only with stated reason.

| Scenario | Default | Skip only when |
|----------|---------|----------------|
| Plan baseline | Agent(boost-observer) or Agent(Explore) | Target is 1 file < 200 lines |
| Do: 2+ independent files/modules | superpowers:dispatching-parallel-agents | Explicit ordering dependency |
| AutoResearch ≥ 2 candidates | One worktree + one Agent per candidate | Candidates differ only in a parameter value |
| Check on non-trivial change | Parallel: diff-read / test-run / regression-spot | Change is 1 line |
| ≥ 3 files touched or rollback-sensitive | superpowers:using-git-worktrees | Trivially revertible (< 30s manual) |
| `complete` on non-trivial change | superpowers:requesting-code-review + Agent(superpowers:code-reviewer) | Pure format / typo |

Skipping requires an entry in the Log's `action_reason` field naming which default was skipped and why.

"Non-trivial change" = any single change that touches ≥ 2 files OR ≥ 20 lines OR user-facing behavior OR touches SKILL.md. Trivial = everything else (typos, comment-only, single-line internal tweaks).
```

### Change 4: Execution Topology table — expand to 4 columns, add 1 row

Replace the current 3-column / 5-row table with:

| Condition | Topology | How (concrete) | Guardrail |
|-----------|----------|----------------|-----------|
| Single-scope edit, next step obvious | `local` | Main thread uses Edit/Write directly | — |
| Read-heavy baseline (≥ 3 files or large logs) | `delegated` | `Agent(subagent_type=Explore)` or `Agent(subagent_type=boost-observer)` | Read-only; restate Contract on return |
| 2+ independent explorations | `delegated` (parallel) | `superpowers:dispatching-parallel-agents` | Each subagent has one bounded deliverable |
| Competing approaches / second opinion needed | `delegated` (parallel) | Parallel `Agent(challenger)` + `Agent(innovator)` + `Agent(pragmatist)` | Decision authority stays on main thread |
| Risky / broad change, cheap rollback needed | `isolated-worktree` | `superpowers:using-git-worktrees` (never hand-write `git worktree`) | Merge only after Check passes |
| Side-by-side comparison of 2 approaches | `delegated` + `isolated-worktree` | Two `Agent(…, isolation: "worktree")` calls | Equal budget; locked evaluator |
| Non-trivial Check | `delegated` (parallel) | Diff-read + `Agent(Explore)` regression scan + `Bash` test run | Results merge on main thread; no delegated keep/rollback |

Plus two explicit rules beneath the table:

```
Rules:
- Main thread always owns: Stable Contract, Check evidence review, Act decision, state snapshot. Subagents and worktrees never own keep/rollback.
- After any subagent / worktree returns, restate <target> / <goal> / <baseline> / <next_action> before using its result.
```

### Change 5: References list

Add one bullet pointing to the new file:

```
- [references/orchestration.md](references/orchestration.md) — concrete skill/subagent/worktree dispatch patterns, parallelization hotspots, anti-patterns
```

## Changes — references/orchestration.md (new, ~220 lines)

**Design principle (from user directive): tacit knowledge first.** This file is not a decision table — it is a pattern clinic. The reader should form pattern-recognition through concrete user-message fragments, not through abstract rules.

### § 1 How to read this file (~10 lines)

`SKILL.md` handoff defaults cover 80% of common cases. This file covers **"looks like X, actually is Y"** edge judgments. Read the patterns as few-shot examples, not as lookup rules.

### § 2 Handoff patterns — by PDCAA phase (~80 lines)

5–6 patterns per phase. Each pattern ≤ 10 lines, formatted:

```
### Pattern: <name>
**User:** "<verbatim-style user fragment>"
**Looks like:** <surface interpretation>
**Actually is:** <real situation>
**Handoff:** <concrete tool>
**Anti-signal:** <when this pattern does NOT apply>
```

Must-have patterns (each phase):

- **Plan** — scope-disguise ("优化一下搜索" masking Design Gate); multi-file baseline flooding main context; "just diagnose" → single Check only; known-target skip the baseline subagent
- **Do** — bug masquerading as feature (TDD vs systematic-debugging); broad edit without worktree; 2+ files improperly serialized; refactor inside Do without rolling out a plan
- **Check** — "tests pass" ≠ "goal advanced"; self-confirmation vs real evidence; missing regression scope; skipping verification-before-completion on "obvious" wins
- **Align** — drift silently relaxed; local-problem hijack; Contract Candidate that should have escalated
- **Act** — `complete` before independent review; `research` without parallel candidates; using rollback when adjust was enough

### § 2.5 Parallelization hotspots (~35 lines)

Catalogue the highest-return parallel opportunities in a typical boost run. Each entry: **before / after** diff.

- Multi-file grep — one `Agent(Explore)` query instead of 3 serial greps
- 2 candidate approaches — two `Agent(isolation: worktree)` concurrent instead of sequential
- Check trio (diff-read / test-run / regression-spot) — three concurrent tool calls, not serial
- AutoResearch probe — candidate experiments in worktrees, not in main workspace
- Baseline reading + doc scanning — two parallel subagents
- Code review + post-review lint/typecheck — parallel, not sequential

### § 3 Topology gotchas — anti-pattern library (~45 lines)

Writes the "**don't** upgrade" cases so the flipped default doesn't over-ceremonize. Each anti-pattern 3–5 lines, with "wrong → right" rewrite.

- 1-file, 1-line edit → no worktree
- 3-read-or-less evidence → no Explore subagent
- Subagent prompt that says "based on your findings, fix it" → violates CLAUDE.md's *Never delegate understanding*
- challenger/innovator/pragmatist dispatched with only 1 candidate approach → 3-view degenerates
- worktree experiment never merged back → orphan branches
- parallel subagents with shared mutable state (e.g., both writing the same file) → serialize instead

### § 4 Handoff templates (~30 lines)

Copy-paste ready:

- Minimum prompt skeleton for `Agent(subagent_type=Explore)` — target, goal, return format
- 5-step checklist before opening a worktree via `superpowers:using-git-worktrees`
- Restate template after any delegated return (`<target>/<goal>/<baseline>/<next_action>`)
- Contract Candidate escalation wording
- Parallel Check sub-call template (how to fan out diff-read, test-run, regression-scan simultaneously)

### § 5 End-to-end orchestration demo (~30 lines)

One fully-narrated boost run that shows *every* upgrade triggering naturally:

Starting from a user asking "帮我加个审计日志" →

1. Plan triggers `superpowers:brainstorming` (scope ambiguous)
2. After spec, `superpowers:writing-plans` produces implementation plan
3. Do opens a worktree via `superpowers:using-git-worktrees`
4. Inside worktree, 2 independent files → `superpowers:dispatching-parallel-agents`
5. Each parallel thread uses `superpowers:test-driven-development`
6. Check fans out into diff-read + test-run + regression-spot concurrently
7. `superpowers:verification-before-completion` guards the `complete` claim
8. Act triggers `superpowers:requesting-code-review`
9. Finally `superpowers:finishing-a-development-branch` handles merge

Narrative is 30 lines of Log-style entries, not prose.

### § 6 Calibration (~10 lines)

How to read the Log to detect over- or under-delegation. Reuses existing metrics in `evolution-metrics.md`:

- Many rollbacks after delegated runs → sub-delegation quality issue
- Main thread re-reads large files repeatedly across cycles → under-delegating baseline
- `avg_tools_per_cycle` rising sharply → ceremony creeping in

## Changes — eval-fixtures.md

Add two fixtures:

1. **Fixture: default-parallel-check.** Given a 10-file edit, the skill should produce a Check phase that fans out to 3 concurrent sub-calls; regressing this means the default got silently relaxed.
2. **Fixture: skip-with-reason.** Given a 1-line typo fix, the skill should skip `superpowers:requesting-code-review` and **explicitly record the skip reason** in `action_reason`; regressing this means the default over-ceremonized trivial work.

Fixture format follows the existing style in `eval-fixtures.md`.

## Sync plan (unchanged protocol)

After validation in this workspace:

1. Copy updated `SKILL.md`, `references/orchestration.md`, `references/eval-fixtures.md` to `~/.codex/skills/boost/`.
2. Copy the same files to `~/.claude/skills/boost/`.
3. Verify installed copies match workspace source.

## Rollout / risk

- **Risk: over-ceremony creep.** Flipped default could cause the skill to spawn subagents/worktrees for trivial tasks. **Mitigation:** explicit "skip only when" thresholds + skip-reason requirement in Log + the new skip fixture.
- **Risk: `SKILL.md` readability.** +40~45 lines past the 300-line soft limit. **Mitigation:** additions are table rows + one-line handoff bullets, density stays high; no prose padding.
- **Risk: Codex host quirks.** Existing hook uses `python3` to avoid `$` issues; no new hooks in this upgrade. **Mitigation:** no action required.

## Open items (deliberately deferred)

- Whether to promote `orchestration.md` § 5 (end-to-end demo) into its own file later if it grows — **defer** until we see real usage.
- Whether to add a `boost-challenger` / `boost-validator` custom agent — **deferred**; reusing globals (`challenger`, `code-reviewer`) first, add custom only if evidence of insufficiency appears.
- Whether to add a CronCreate-style periodic self-audit of Log patterns — **defer** to a later version.
