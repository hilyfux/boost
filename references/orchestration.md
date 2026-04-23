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
