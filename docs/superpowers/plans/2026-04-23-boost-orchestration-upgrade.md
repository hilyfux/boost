# Boost Orchestration Upgrade Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade `boost` skill from v1.3.0 to v1.4.0 so PDCAA phases routinely dispatch Claude Code subagents/worktrees and coordinate with `superpowers:*` skills under a flipped "max-efficiency" default.

**Architecture:** Spec-driven doc edits. Three files change: `SKILL.md` (5 surgical additions), `references/orchestration.md` (new, pattern-clinic style), `references/eval-fixtures.md` (+2 fixtures). Version bump. Installed copies synced last.

**Tech Stack:** Markdown, YAML frontmatter, git, `superpowers:using-git-worktrees`.

**Spec:** `docs/superpowers/specs/2026-04-23-boost-orchestration-upgrade-design.md`

---

## File Structure

| File | Change | Purpose |
|------|--------|---------|
| `SKILL.md` | Modify (5 additions) | Main skill logic — per-phase Handoff blocks, Defaults section, topology table rewrite |
| `references/orchestration.md` | Create | Tacit-knowledge pattern clinic (§1–6) |
| `references/eval-fixtures.md` | Append 2 fixtures | Regression guards for flipped default + skip path |
| `~/.codex/skills/boost/` | Sync | Codex-side installed copy |
| `~/.claude/skills/boost/` | Sync | Claude-side installed copy |

---

## Task 0: Set up isolated worktree

**Files:**
- Create: worktree at `.worktrees/boost-orchestration-upgrade/` (dogfooding the new default: rollback-sensitive changes to SKILL.md itself)

- [ ] **Step 1: Verify no uncommitted changes on main**

Run: `git status --short`
Expected: empty output (clean working tree). If not clean, commit/stash before proceeding.

- [ ] **Step 2: Invoke `superpowers:using-git-worktrees` skill**

Use the Skill tool to invoke `superpowers:using-git-worktrees`. Tell it: "Create a worktree for the boost orchestration upgrade (v1.3.0 → v1.4.0). Base branch: main. Feature branch: boost-orchestration-upgrade."

The skill will produce: worktree path and branch name. Record both in the implementation context.

- [ ] **Step 3: Change into worktree directory and confirm state**

Run: `cd <worktree-path> && git status && git log --oneline -3`
Expected: clean tree on branch `boost-orchestration-upgrade`; last 3 commits match main.

- [ ] **Step 4: Commit baseline marker**

```bash
git commit --allow-empty -m "chore: start boost v1.4.0 orchestration upgrade"
```

Expected: one empty commit so subsequent changes are unambiguous to review.

---

## Task 1: Add two new eval-fixtures (TDD-style — define success first)

**Files:**
- Modify: `references/eval-fixtures.md` (append after line 595, before "## Minimal Rubric")

- [ ] **Step 1: Read current tail of eval-fixtures.md to confirm anchor**

Run: `grep -n "## Minimal Rubric" references/eval-fixtures.md`
Expected: one line, around line 596.

- [ ] **Step 2: Append Fixture 29: Default Parallel Check**

Insert the following block immediately before `## Minimal Rubric` in `references/eval-fixtures.md`:

```markdown
## Fixture 29: Default Parallel Check

### User request

`用 boost 帮我完成这 10 个文件的字段重命名。`

### Must do

- After the Do phase, enter Check with the default: three concurrent sub-checks (diff-read + test-run + regression-spot).
- Run the three sub-checks in parallel (one tool-use block with multiple calls or parallel subagents), not serially.
- Merge sub-check results on the main thread before the Align decision.

### Must not do

- Do not execute the three sub-checks in sequence when they have no dependency on each other.
- Do not delegate the keep/rollback decision to any sub-check agent.

### Pass criteria

- The Check phase visibly performs at least 3 parallel verifications.
- Main thread produces a single merged Check output consumed by Align.

## Fixture 30: Skip With Reason

### User request

`boost 顺手修一个文档里的 typo：把 "reciever" 改成 "receiver"。`

### Must do

- Classify the change as trivial (single token, 1 line, comment-level).
- Skip the `superpowers:requesting-code-review` default and the `Agent(superpowers:code-reviewer)` default.
- Record the skip in the Log's `action_reason` field — name which default was skipped and the concrete reason.

### Must not do

- Do not open a worktree, do not dispatch parallel subagents, do not request code review for a 1-token typo.
- Do not skip the default silently — the Log must carry a skip reason.

### Pass criteria

- The Log's `action_reason` mentions the skipped default explicitly (e.g., "skipped requesting-code-review: change is 1 token, trivial per SKILL.md definition").
- No subagent or worktree is invoked for this fixture.
```

- [ ] **Step 3: Verify fixture insertion**

Run: `grep -c "## Fixture 29\|## Fixture 30" references/eval-fixtures.md`
Expected: `2`

Run: `grep -n "## Minimal Rubric" references/eval-fixtures.md`
Expected: line number has shifted by ~40+ from its previous position, confirming insertion.

- [ ] **Step 4: Commit**

```bash
git add references/eval-fixtures.md
git commit -m "test(boost): add fixtures 29–30 for default-parallel and skip-with-reason"
```

---

## Task 2: Bump SKILL.md frontmatter version to 1.4.0

**Files:**
- Modify: `SKILL.md:3`

- [ ] **Step 1: Apply version edit**

Change SKILL.md line 3:
- Old: `version: "1.3.0"`
- New: `version: "1.4.0"`

- [ ] **Step 2: Verify**

Run: `grep '^version:' SKILL.md`
Expected: `version: "1.4.0"`

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "chore(boost): bump skill version to 1.4.0"
```

---

## Task 3: Insert Parallelization & Isolation Defaults section

**Files:**
- Modify: `SKILL.md` — insert new section immediately before `## Execution Topology` (currently line 296)

- [ ] **Step 1: Locate the anchor**

Run: `grep -n "^## Execution Topology" SKILL.md`
Expected: one line (was 296 before Task 2; should still be 296 since Task 2 only changed line 3 length marginally).

- [ ] **Step 2: Insert new section before the anchor**

Use the Edit tool. Replace this anchor (exact, including surrounding lines):

```
This log is the mandatory cycle output. It appears in the visible response, not hidden.

## Execution Topology
```

with:

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
```

- [ ] **Step 3: Verify**

Run: `grep -n "^## Parallelization & Isolation Defaults\|^## Execution Topology" SKILL.md`
Expected: two lines, `Parallelization & Isolation Defaults` preceding `Execution Topology` by ~18 lines.

- [ ] **Step 4: Commit**

```bash
git add SKILL.md
git commit -m "feat(boost): add Parallelization & Isolation Defaults section to SKILL.md"
```

---

## Task 4: Rewrite Execution Topology table (3 cols → 4 cols, +1 row, +2 rules)

**Files:**
- Modify: `SKILL.md` — replace the existing Execution Topology table and its "Why" wording

- [ ] **Step 1: Apply Edit to replace old table and add rules**

Replace this exact block:

```
## Execution Topology

Use the lightest topology that fits. Trigger heavier ones by condition, not by habit.

| Condition | Topology | Why |
|-----------|----------|-----|
| Single-scope edit, next step obvious | `local` | No overhead |
| 2+ independent exploration directions | `delegated` (parallel subagents) | Explore in parallel, converge in main thread |
| Read-heavy evidence gathering | `delegated` (Explore subagent) | Preserve main-thread context quality |
| Risky/broad change, needs cheap rollback | `isolated-worktree` | Main workspace stays clean |
| Comparing two competing approaches | `delegated` + `isolated-worktree` | Compare without pollution |
```

with:

```
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
```

- [ ] **Step 2: Verify new table and rules**

Run: `grep -n "| Condition | Topology | How (concrete) | Guardrail |" SKILL.md`
Expected: one line.

Run: `grep -c "^| " SKILL.md`
Expected: greater than the pre-change count by ~3 (table row count + Defaults table rows already in place).

Run: `grep -n "Main thread always owns" SKILL.md`
Expected: one line (the new rule).

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "feat(boost): expand Execution Topology table with concrete tools and guardrails"
```

---

## Task 5: Add Handoff block to each PDCAA phase

**Files:**
- Modify: `SKILL.md` — 5 insertions, one per phase subsection

The five PDCAA phase headings are at these approximate locations (before any Task 3/4 inserts): `### 1. Plan` @ L103, `### 2. Do` @ L113, `### 3. Check` @ L122, `### 4. Align` @ L143, `### 5. Act` @ L166. After Task 3/4 the offsets shift but the content anchors stay the same — use the content anchors below.

- [ ] **Step 1: Insert Plan phase Handoff block**

Anchor (end of Plan phase output block):

```
> **Plan:** <当前最小动作及其理由>

### 2. Do
```

Replace with:

```
> **Plan:** <当前最小动作及其理由>

**Handoff (default; skip only when marked):**

- Baseline capture → `Agent(subagent_type=boost-observer)` for boost-specific targets; `Agent(subagent_type=Explore)` for general codebase reads. Skip only when target is a single file < 200 lines.
- Scope ambiguous / multi-subsystem → `superpowers:brainstorming` before the first Do.
- Multi-step implementation → `superpowers:writing-plans`.
- Stuck on design choice → enter AutoResearch (see below).

### 2. Do
```

- [ ] **Step 2: Insert Do phase Handoff block**

Anchor:

```
> **Do:** <执行了什么，结果是什么>

### 3. Check
```

Replace with:

```
> **Do:** <执行了什么，结果是什么>

**Handoff (default; skip only when marked):**

- Feature / behavior change on code → `superpowers:test-driven-development`.
- Bug / test failure / unexpected behavior → `superpowers:systematic-debugging`.
- Broad (≥ 3 files) or rollback-sensitive change → `superpowers:using-git-worktrees`.
- 2+ independent tasks or files → `superpowers:dispatching-parallel-agents`.

### 3. Check
```

- [ ] **Step 3: Insert Check phase Handoff block**

Anchor:

```
> - Future checkability: preserved / degraded — <note>

### 4. Align
```

Replace with:

```
> - Future checkability: preserved / degraded — <note>

**Handoff (default; skip only when marked):**

- Non-trivial change → parallel sub-checks (diff-read + test-run + regression-spot) in one tool-use block.
- About to claim "complete / fixed / passing" → `superpowers:verification-before-completion`.
- Evidence needs independent read → `Agent(subagent_type=superpowers:code-reviewer)` or `Agent(subagent_type=challenger)` as second opinion.

### 4. Align
```

- [ ] **Step 4: Insert Align phase Handoff block**

Anchor:

```
> **Align:** <drift detected: none / type + correction taken>

### 5. Act
```

Replace with:

```
> **Align:** <drift detected: none / type + correction taken>

**Handoff (default; skip only when marked):**

- Drift triggered by ≥ 2 signals in the same cycle → re-read Stable Contract AND consider `superpowers:brainstorming` to re-charter.
- AutoReceive marked a Contract Candidate → pause PDCAA; run explicit escalation before continuing.

### 5. Act
```

- [ ] **Step 5: Insert Act phase Handoff block**

Anchor:

```
> **[boost · Iter N]** Target: <target> | Goal: <goal> | 上轮: <result> → 本轮: <next focus>

## AutoResearch
```

Replace with:

```
> **[boost · Iter N]** Target: <target> | Goal: <goal> | 上轮: <result> → 本轮: <next focus>

**Handoff (default; skip only when marked):**

- `decision == complete` on non-trivial change → `superpowers:requesting-code-review`.
- `complete` + ready to merge or PR → `superpowers:finishing-a-development-branch`.
- `decision == research` → AutoResearch; when ≥ 2 candidates, parallel `Agent(innovator)` + `Agent(pragmatist)` with `Agent(challenger)` as reviewer.

## AutoResearch
```

- [ ] **Step 6: Verify all 5 handoff blocks present**

Run: `grep -c "^\*\*Handoff (default; skip only when marked):" SKILL.md`
Expected: `5`

- [ ] **Step 7: Commit**

```bash
git add SKILL.md
git commit -m "feat(boost): add per-phase Handoff blocks naming concrete skills/tools"
```

---

## Task 6: Add orchestration.md to References list

**Files:**
- Modify: `SKILL.md` — References section

- [ ] **Step 1: Apply Edit**

Replace this line:

```
- [references/methodology.md](references/methodology.md) — full methodology: research frames, topology, autonomous iteration, validation protocol, ratchet/rollback patterns
```

with:

```
- [references/methodology.md](references/methodology.md) — full methodology: research frames, topology, autonomous iteration, validation protocol, ratchet/rollback patterns
- [references/orchestration.md](references/orchestration.md) — concrete skill/subagent/worktree dispatch patterns, parallelization hotspots, anti-patterns
```

- [ ] **Step 2: Verify**

Run: `grep -n "references/orchestration.md" SKILL.md`
Expected: one line inside the References section.

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "docs(boost): reference orchestration.md from SKILL.md"
```

---

## Task 7: Create references/orchestration.md (new file)

**Files:**
- Create: `references/orchestration.md`

This file is the **tacit-knowledge payload**. Format: pattern clinic, not decision tables. Target ≤ 240 lines.

- [ ] **Step 1: Write the complete file in one pass**

Create `references/orchestration.md` with this exact content:

````markdown
# Orchestration Patterns

SKILL.md handoff defaults cover 80% of common cases. This file covers the 20%: the "looks like X, actually is Y" edge judgments, parallelization hotspots, and anti-patterns.

Read these as few-shot examples. The point is recognition — when a new situation matches a pattern below, you should feel the match rather than look it up.

---

## § 1. How to read this file

- Patterns are stated as **user fragment → surface reading → real situation → handoff → anti-signal**.
- Examples are intentionally short; the goal is to form matchable shapes in context, not to be exhaustive.
- When SKILL.md's handoff block already gives a clear answer, stay there. Come here only when the default feels wrong for the case in front of you.
- Never delegate understanding. Subagents take bounded, read-heavy, or independently-verifiable work. They never take keep/rollback authority.

---

## § 2. Handoff patterns — by PDCAA phase

### Plan

**Pattern: Scope disguise**
- **User:** "优化一下搜索，让它更快。"
- **Looks like:** Plan → straight to reading code.
- **Actually is:** No Goal/Constraints/Done. Design Gate fails.
- **Handoff:** `superpowers:brainstorming` to build spec first.
- **Anti-signal:** User already named P95 / QPS / specific file → enter PDCAA directly.

**Pattern: Baseline flooding main context**
- **User:** "帮我优化这个 50-文件 monorepo 的测试稳定性。"
- **Looks like:** Read files to form baseline.
- **Actually is:** Main thread context will be destroyed after 5-10 reads.
- **Handoff:** `Agent(subagent_type=Explore)` or `Agent(subagent_type=boost-observer)` with bounded deliverable.
- **Anti-signal:** Target is 1 file < 200 lines → read it directly.

**Pattern: "Just diagnose"**
- **User:** "先分析一下，别改。"
- **Looks like:** Normal boost run.
- **Actually is:** Stop after first Check; PDCAA does not continue into Do.
- **Handoff:** No skill handoff. Honor the stop.
- **Anti-signal:** None — this is a hard rule.

**Pattern: Known target, trivial read**
- **User:** "这个 100 行的 util.py 看起来有 bug，优化一下。"
- **Looks like:** Default → invoke boost-observer.
- **Actually is:** Single file under 200 lines; subagent adds overhead, no gain.
- **Handoff:** Skip baseline subagent; read directly. Record skip reason in Log.
- **Anti-signal:** If the same file references 5+ other modules, treat as multi-file.

### Do

**Pattern: Bug masquerading as feature**
- **User:** "帮我加一个搜索功能。"
- **Later turns up:** Search existed, has been failing silently.
- **Handoff:** Switch to `superpowers:systematic-debugging`, not `test-driven-development`.
- **Anti-signal:** Greenfield truly new behavior with no prior implementation → TDD is correct.

**Pattern: Broad edit without worktree**
- **User:** "把 logger 从 winston 换成 pino，全项目替换。"
- **Looks like:** Edit + run tests.
- **Actually is:** ≥ 3 files, rollback-sensitive across boundaries.
- **Handoff:** `superpowers:using-git-worktrees` before the first edit.
- **Anti-signal:** The replacement is mechanical in 1 file only.

**Pattern: Serialized-when-parallel**
- **User:** "给 5 个独立服务各加一个 /health 端点。"
- **Looks like:** Edit one, edit next, edit next.
- **Actually is:** 5 independent deliverables; parallel subagents halve wall-clock.
- **Handoff:** `superpowers:dispatching-parallel-agents`.
- **Anti-signal:** Services share a changed module (ordering dependency) → serialize.

**Pattern: Refactor sneaking into Do**
- **User:** "修这个 bug。"
- **During Do:** Agent also rewrites the surrounding module.
- **Handoff:** Stop; restart Plan with explicit scope; consider `superpowers:writing-plans` for the refactor as its own pass.
- **Anti-signal:** The refactor is 1–3 lines required for the fix to compile.

### Check

**Pattern: "Tests pass" ≠ "goal advanced"**
- **Situation:** Existing tests green after edit.
- **Actually is:** The edit's intended behavior is not covered by any test.
- **Handoff:** Add a new test (TDD red-green) OR produce a new sample output and inspect before claiming Check passed.
- **Anti-signal:** New test was written first in the Do phase → evidence exists already.

**Pattern: Self-confirmation vs real evidence**
- **Situation:** Agent says "looks correct" without reading or running anything.
- **Handoff:** `superpowers:verification-before-completion` before the next Act.
- **Anti-signal:** Verification genuinely was run and logged — do not ceremonially re-run.

**Pattern: Missing regression scope**
- **Situation:** Change works on the happy path, but touches a shared util used by 7 callers.
- **Handoff:** `Agent(subagent_type=Explore)` regression scan of the util's callers as one of the parallel Check sub-calls.
- **Anti-signal:** Change is isolated and has no callers beyond its file.

**Pattern: Single-line-change ceremony**
- **Situation:** 1-line typo fix; default would request code review and 3-way Check.
- **Handoff:** Skip both; record skip in `action_reason`.
- **Anti-signal:** The 1-line change is in a security-sensitive path (auth, crypto) → keep the review.

### Align

**Pattern: Silent constraint relaxation**
- **Situation:** Agent accepts user's "just this once, bypass the validator."
- **Handoff:** Emit Align drift `constraint_drift`; re-read Stable Contract; do not bypass unless explicitly escalated.
- **Anti-signal:** User escalated formally via Contract Candidate.

**Pattern: Local-problem hijack**
- **Situation:** Debugging a flaky test turns into rewriting the test harness.
- **Handoff:** Emit Align drift `local_problem_hijack`; park the harness rewrite as a separate Goal.
- **Anti-signal:** The harness is the cause of flakes; then it *is* the Goal, and a Contract Candidate should formalize it.

**Pattern: Contract Candidate never escalated**
- **Situation:** User said "also mobile must not regress"; agent filed it only in Runtime State.
- **Handoff:** Promote to `Contract Candidate: Constraints += "mobile rendering unchanged"`; pause PDCAA until acknowledged.
- **Anti-signal:** The constraint was already in Constraints.

### Act

**Pattern: `complete` before independent review**
- **Situation:** 5-file refactor; agent declares `complete` after its own Check.
- **Handoff:** `superpowers:requesting-code-review` before the completion stands.
- **Anti-signal:** Change is pure format / typo.

**Pattern: `research` without parallel candidates**
- **Situation:** AutoResearch triggered; agent evaluates candidates one by one on main thread.
- **Handoff:** Spawn candidates as parallel `Agent` calls, each in its own worktree when they would mutate files.
- **Anti-signal:** Candidates differ only in a parameter value (e.g., learning rate 0.01 vs 0.001) — one harness suffices.

**Pattern: Rollback when adjust would do**
- **Situation:** One Check failed; agent rolls back the whole Do.
- **Handoff:** Use `adjust`, not `rollback`, when the direction is correct and the failure is a named, fixable gap.
- **Anti-signal:** The Check reveals a constraint violation — then rollback is correct.

---

## § 2.5 Parallelization hotspots

| Before (serial) | After (parallel) |
|-----------------|------------------|
| `grep` three patterns in sequence, read each result | One `Agent(subagent_type=Explore)` call that returns all three findings |
| Test two competing approaches one after another in the main workspace | Two `Agent(..., isolation: "worktree")` calls concurrently; compare returned deliverables |
| Check = read diff → run tests → regression-scan, one after the next | One tool-use block: `Read` diff + `Bash` test run + `Agent(Explore)` regression scan, all in one message |
| AutoResearch probe runs in main workspace, pollutes it | Each probe runs in a dedicated worktree; failed probes are discarded without clean-up cost |
| Read baseline, then read related docs | Two parallel subagents: one on code baseline, one on docs |
| Code review, then lint, then typecheck | `Agent(superpowers:code-reviewer)` in parallel with `Bash` lint and `Bash` typecheck |

Rule of thumb: if two work-units have **no shared mutable state** and **no ordering dependency**, they should run in parallel.

---

## § 3. Topology gotchas — anti-pattern library

**Worktree for a 1-line edit** → the 30s setup/merge cost is worse than the potential rollback cost. *Right:* edit in main, `git restore` on failure.

**Explore subagent for 3-read evidence** → main thread reads 3 files faster than it restates state after a subagent. *Right:* direct `Read` ×3 in one tool-use block.

**"Based on your findings, fix it"** — prompt that delegates synthesis → violates CLAUDE.md's *Never delegate understanding*. *Right:* ask the subagent for findings only; main thread writes the fix after restating target/goal.

**3-view parallel (`challenger/innovator/pragmatist`) with only 1 candidate approach** → the three views degenerate into echo. *Right:* invoke the triangle only when AutoResearch has ≥ 2 candidates with non-trivial tradeoffs.

**Worktree experiment never merged** → orphan branches accumulate. *Right:* `superpowers:using-git-worktrees` produces a branch with an explicit merge/cleanup step; finish with `superpowers:finishing-a-development-branch`.

**Parallel subagents sharing mutable state** (e.g., both editing `SKILL.md`) → race / one overwrites the other. *Right:* either serialize, or split the file by section before dispatch.

**Over-applied default on trivial work** (typo + worktree + code review) → ceremony > value. *Right:* apply the "Non-trivial change" definition from SKILL.md; skip defaults on trivial; record skip in `action_reason`.

---

## § 4. Handoff templates

### Explore subagent — minimum prompt skeleton

```
Target: <file path or system area>
Goal: <what specific question to answer>
Return format: bullet list, ≤ 150 words, cite file:line for each claim.
Do not propose fixes. Read-only.
```

### Worktree open checklist (before invoking `superpowers:using-git-worktrees`)

1. Main branch clean (`git status` empty)?
2. Branch name derived from the task (`<topic>-<yyyy-mm-dd>` or similar).
3. Baseline commit marker so diff review is unambiguous.
4. Work scope fits in the worktree without leaking to main (no shared caches / logs outside the repo).
5. A named merge path: direct merge, PR, or `superpowers:finishing-a-development-branch`.

### Restate after delegated return

```
<target>: <restated from Stable Contract>
<goal>:   <restated>
<baseline>: <what we knew before delegation>
<delegation_result>: <what the subagent produced, 1 line>
<next_action>: <what the main thread does now>
```

### Contract Candidate escalation

```
**Contract Candidate:** <field> → <proposed new value>
Evidence: <what in the last cycle forced this>
Decision needed from: <user / overwhelming evidence>
Cycle paused until resolution.
```

### Parallel Check sub-call template

In one tool-use block, issue:

1. `Read` the diff or changed files (main thread).
2. `Bash` run the relevant test command.
3. `Agent(subagent_type=Explore)` for regression scan of dependent callers.

Merge results on main thread. Act chooses `continue` / `adjust` / `rollback` from the merged view.

---

## § 5. End-to-end orchestration demo

Demonstration of how a single boost run naturally triggers the full orchestration chain. Output is log-style, not prose.

```
User: "帮我在这个 API 项目里加一个审计日志。"

Stable Contract:
- Goal: All mutation endpoints emit an audit log record containing user, action, before, after, ts.
- Constraints: No schema break; no latency regression > 5ms P95.
- Done: New audit events appear in logs on POST/PUT/DELETE; regression tests green.
- Checks: Run test suite; grep for audit emission in changed handlers; P95 latency check.

[Iter 1]
Plan: Scope ambiguous (storage? async? retention?). Handoff → superpowers:brainstorming.
Do:   Brainstorming yields spec; returning with explicit storage + async queue decision.
Check: Spec approved by user.
Align: No drift.
Act:  continue. Handoff → superpowers:writing-plans to sequence 6 tasks.

[Iter 2]
Plan: Plan ready. Work is rollback-sensitive (touches request lifecycle). Handoff → superpowers:using-git-worktrees.
Do:   Inside worktree, 2 independent files (middleware, queue client) — handoff → superpowers:dispatching-parallel-agents. Each uses superpowers:test-driven-development.
Check: Parallel: Read diff + Bash run tests + Agent(Explore) scan dependent handlers. All green.
Align: No drift.
Act:  continue.

[Iter 3]
Plan: Integrate middleware into app bootstrap. Single-scope edit. No delegation.
Do:   Edit app.ts; write integration test.
Check: superpowers:verification-before-completion because we are about to claim complete. P95 measured, within budget.
Align: No drift.
Act:  complete. Handoff → superpowers:requesting-code-review.
Review: approved with one nit (log schema field name). Fix inline.
Act:  complete (post-review). Handoff → superpowers:finishing-a-development-branch to merge worktree back.
```

---

## § 6. Calibration — reading your own Log

Use the Log to detect orchestration drift, not just content drift.

- **Rollback spikes after delegated runs** → subagent briefs were too broad, or ownership leaked. Tighten prompts; re-read § 3.
- **Main thread re-reads the same large file across cycles** → under-delegated baseline. Next Plan should open with `Agent(boost-observer)` or `Agent(Explore)`.
- **`avg_tools_per_cycle` rises sharply** — see `evolution-metrics.md` for the definition. Ceremony creep; check whether skipped-default reasons are being recorded; if yes, refine the threshold.
- **Repeated Contract Candidate in Runtime State but never escalated** → AutoReceive routing is too conservative; promote at the next Align.
- **Parallel Check sub-calls never appear in the Log** → default silently skipped; the skip-with-reason fixture (Fixture 30) should have caught this.
````

- [ ] **Step 2: Verify file exists and line count within budget**

Run: `wc -l references/orchestration.md`
Expected: ≤ 260 lines (small buffer over the 240 target to accommodate markdown rendering).

Run: `grep -c "^## §\|^### Pattern:" references/orchestration.md`
Expected: at least 20 (6 section headings + ~15 pattern entries).

- [ ] **Step 3: Spot-check content presence**

Run: `grep -c "superpowers:brainstorming\|superpowers:using-git-worktrees\|superpowers:test-driven-development\|superpowers:dispatching-parallel-agents\|superpowers:verification-before-completion\|superpowers:requesting-code-review\|superpowers:systematic-debugging\|superpowers:writing-plans\|superpowers:finishing-a-development-branch" references/orchestration.md`
Expected: ≥ 10 hits (each major skill appears at least once; several appear multiple times).

- [ ] **Step 4: Commit**

```bash
git add references/orchestration.md
git commit -m "feat(boost): add orchestration.md pattern clinic for v1.4.0"
```

---

## Task 8: Full validation walkthrough

**Files:**
- Read: `SKILL.md`, `references/orchestration.md`, `references/eval-fixtures.md`

This task runs the Done-criteria checks from the spec.

- [ ] **Step 1: Confirm frontmatter version**

Run: `grep -n '^version:' SKILL.md`
Expected: `version: "1.4.0"` on line 3.

- [ ] **Step 2: Confirm all 5 Handoff blocks**

Run: `grep -c "^\*\*Handoff (default; skip only when marked):" SKILL.md`
Expected: `5`

- [ ] **Step 3: Confirm topology table is 4-column with 7 data rows**

Run: `awk '/^## Execution Topology/,/^## Stop When/' SKILL.md | grep -c "^| "`
Expected: `9` (1 header + 1 separator + 7 data rows).

- [ ] **Step 4: Confirm tool names appear in SKILL.md**

Run:

```bash
for name in \
  "superpowers:brainstorming" \
  "superpowers:writing-plans" \
  "superpowers:test-driven-development" \
  "superpowers:systematic-debugging" \
  "superpowers:using-git-worktrees" \
  "superpowers:dispatching-parallel-agents" \
  "superpowers:verification-before-completion" \
  "superpowers:requesting-code-review" \
  "superpowers:finishing-a-development-branch" \
  "Agent(subagent_type=boost-observer)" \
  "Agent(subagent_type=Explore)" \
  "Agent(subagent_type=superpowers:code-reviewer)"
do
  hits=$(grep -c "$name" SKILL.md)
  echo "$hits $name"
done
```

Expected: every entry ≥ 1 hit.

- [ ] **Step 5: Confirm orchestration.md reference link**

Run: `grep -c "\[references/orchestration.md\](references/orchestration.md)" SKILL.md`
Expected: `1`

- [ ] **Step 6: Confirm orchestration.md size budget**

Run: `wc -l references/orchestration.md`
Expected: between 180 and 260 lines.

- [ ] **Step 7: Confirm fixtures 29 and 30 present**

Run: `grep -c "^## Fixture 29\|^## Fixture 30" references/eval-fixtures.md`
Expected: `2`

- [ ] **Step 8: Read-through of SKILL.md**

Run: `wc -l SKILL.md`
Expected: between 370 and 410 lines (≈ 40-45 line delta from the pre-change 332).

Use the Read tool to view SKILL.md end-to-end. Confirm:
- No broken markdown (tables well-formed, headings in order).
- Frontmatter YAML valid (version + hooks intact).
- Handoff blocks read naturally within each PDCAA phase.
- Parallelization & Isolation Defaults section precedes Execution Topology.

- [ ] **Step 9: Walk existing eval-fixtures mentally**

Use the Read tool on `references/eval-fixtures.md`. For each of Fixtures 1–28, confirm that nothing in the SKILL.md or orchestration.md changes would cause a regression in the fixture's Pass criteria. Spot-check:

- Fixture 3 (ambiguous) — Plan handoff still asks before brainstorming kicks in (skill respects "先不要改" class stop rules).
- Fixture 14 (built-in subagent choice) — the new table row "Read-heavy baseline" explicitly names `Agent(Explore)`, strengthening this fixture.
- Fixture 15 (skill-before-agent) — new Handoff blocks put skills ahead of agents in the Plan phase, strengthening the fixture.
- Fixture 18 (risky change worktree) — new defaults table makes worktree the default for ≥ 3 files, strengthening this fixture.

If any regression is suspected, fix inline in SKILL.md / orchestration.md and re-run steps 1–8.

- [ ] **Step 10: Commit validation notes if any inline fixes were needed**

```bash
git status
# If changes: git add -A && git commit -m "fix(boost): validation-pass adjustments"
# If clean: no commit needed.
```

---

## Task 9: Merge worktree back, sync installed copies, tag

**Files:**
- All edits above merged back to main
- Sync to `~/.codex/skills/boost/` and `~/.claude/skills/boost/`
- Copy `agents/boost-observer.md` to `~/.claude/agents/` (unchanged; re-sync for safety)

- [ ] **Step 1: Push / finish the worktree branch**

Use the Skill tool to invoke `superpowers:finishing-a-development-branch`. Tell it: "The boost orchestration upgrade branch is ready. Merge to main locally (no PR needed — single-maintainer repo). Run the sync procedure after merge."

Expected: the skill walks through the merge and returns control.

If the skill does not automate merge for local-only workflow, do it manually:

```bash
cd <main-repo-path>
git merge --no-ff boost-orchestration-upgrade -m "feat(boost): v1.4.0 orchestration upgrade"
git worktree remove <worktree-path>
git branch -d boost-orchestration-upgrade
```

Expected: `git log --oneline -3` shows the merge commit at HEAD on main.

- [ ] **Step 2: Tag v1.4.0**

```bash
git tag v1.4.0
```

- [ ] **Step 3: Sync workspace → ~/.codex/skills/boost/**

```bash
rsync -av --delete \
  --exclude=.git --exclude=.worktrees --exclude=docs \
  /Users/wanglinqing/Desktop/workspace-desktop/skills/boost/ \
  /Users/wanglinqing/.codex/skills/boost/
```

Expected: rsync output lists updated SKILL.md, references/orchestration.md, references/eval-fixtures.md.

- [ ] **Step 4: Sync workspace → ~/.claude/skills/boost/**

```bash
rsync -av --delete \
  --exclude=.git --exclude=.worktrees --exclude=docs \
  /Users/wanglinqing/Desktop/workspace-desktop/skills/boost/ \
  /Users/wanglinqing/.claude/skills/boost/
```

Expected: same file set updated under the Claude install path.

- [ ] **Step 5: Sync agents/boost-observer.md → ~/.claude/agents/**

```bash
cp /Users/wanglinqing/Desktop/workspace-desktop/skills/boost/agents/boost-observer.md \
   /Users/wanglinqing/.claude/agents/boost-observer.md
```

Expected: no error; `diff` between the two returns empty.

- [ ] **Step 6: Verify installed copies match workspace**

```bash
diff /Users/wanglinqing/Desktop/workspace-desktop/skills/boost/SKILL.md \
     /Users/wanglinqing/.codex/skills/boost/SKILL.md
diff /Users/wanglinqing/Desktop/workspace-desktop/skills/boost/SKILL.md \
     /Users/wanglinqing/.claude/skills/boost/SKILL.md
diff /Users/wanglinqing/Desktop/workspace-desktop/skills/boost/references/orchestration.md \
     /Users/wanglinqing/.codex/skills/boost/references/orchestration.md
diff /Users/wanglinqing/Desktop/workspace-desktop/skills/boost/references/orchestration.md \
     /Users/wanglinqing/.claude/skills/boost/references/orchestration.md
```

Expected: all four diffs empty.

- [ ] **Step 7: Announce completion**

Report back:

- Workspace at v1.4.0, tagged.
- Installed copies (~/.codex, ~/.claude) synced.
- New file `references/orchestration.md`.
- New fixtures 29–30.
- Spec: `docs/superpowers/specs/2026-04-23-boost-orchestration-upgrade-design.md`.
- Plan: `docs/superpowers/plans/2026-04-23-boost-orchestration-upgrade.md`.

---

## Self-review notes (for the plan writer)

- **Spec coverage:** every `## Changes —` section of the spec has at least one task in this plan (SKILL.md → Tasks 2–6, orchestration.md → Task 7, eval-fixtures.md → Task 1, sync → Task 9). Stable Contract Done criteria each map to a validation step in Task 8.
- **Placeholder check:** no TBD / "similar to" references. Every edit shows the full before/after text.
- **Type / name consistency:** `Agent(subagent_type=superpowers:code-reviewer)` used uniformly (not bare `code-reviewer`). `Agent(subagent_type=boost-observer)` and `Agent(subagent_type=Explore)` used uniformly. All `superpowers:*` skill names match the available-skills list.
- **Order invariant:** fixtures land before SKILL.md changes (Task 1 before Tasks 2–6), mirroring TDD-style "define success first" even for documentation work. Sync is last, after all in-repo edits are verified.
