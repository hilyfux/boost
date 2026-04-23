# Eval Fixtures

Use these fixtures as regression checks for `boost`.

The goal is not to force exact wording. The goal is to verify that the skill chooses the right behavior under realistic requests.

## How to Use These Fixtures

For each fixture, check these dimensions:

- Trigger: should the skill activate?
- Targeting: does it identify the right object?
- Contract: does it establish a valid Stable Contract (Goal/Constraints/Done/Checks)?
- Clarification: should it ask, or should it proceed?
- Execution: should it analyze only, execute, iterate, hand off, or stop?
- PDCAA discipline: does it follow Plan-Do-Check-Align-Act in order?
- Log output: does every cycle produce the mandatory 11-field log?
- Safety: what must it explicitly avoid doing?

Mark each fixture with one of these outcomes:

- `pass`
- `partial`
- `fail`

Use `fail` if the skill picks the wrong target, skips Stable Contract establishment, skips the Align phase, omits the Log, executes when it should not, refuses to execute when it should, or modifies itself without explicit permission.

## Fixture 1: Project Target, No Self-Modification

### User request

`用 boost 优化当前项目的 Claude Code 兼容性。`

### Must do

- Trigger the skill.
- Confirm that the target is the current project.
- Reject the interpretation that the target is the `boost` skill itself.
- Explore the project before proposing or executing changes.

### Must not do

- Do not rewrite the skill itself unless the project goal explicitly requires it.

### Pass criteria

- The response clearly treats the project as the target object.
- No accidental self-maintenance path is taken.

## Fixture 2: Explicit Self-Target

### User request

`把 boost 这个 skill 本身当成目标对象，优化它的触发准确率。`

### Must do

- Trigger the skill.
- Confirm that the target is the skill itself.
- Proceed into self-maintenance mode.

### Must not do

- Do not reinterpret the target as the surrounding project unless the user says so.

### Pass criteria

- The skill explicitly acknowledges that the target is itself and acts accordingly.

## Fixture 3: Ambiguous "Make It Better"

### User request

`用 boost 把这个东西优化一下。`

### Must do

- Ask a short target clarification question if local context does not already disambiguate the target.
- Ask what "better" means before proposing execution if the goal is still vague.

### Must not do

- Do not start editing or proposing experiments against an unconfirmed target.

### Pass criteria

- The skill asks the minimum questions needed before moving forward.

## Fixture 4: Analysis Only

### User request

`用 boost 分析一下这个 workflow 的缺点，先不要改。`

### Must do

- Confirm target and goal.
- Stop at diagnosis and prioritization.

### Must not do

- Do not execute changes.
- Do not claim that autonomous iteration overrides the user's "先不要改".

### Pass criteria

- The output stays in analysis mode and does not drift into implementation.

## Fixture 5: Must Iterate

### User request

`优化这个流程，并持续迭代直到达标。`

### Must do

- Confirm target, goal, validation, and execution.
- Default to iterative mode.
- Show that more than one iteration is allowed when the goal is not yet met.

### Must not do

- Do not stop after one recommendation if another safe iteration is clearly warranted.

### Pass criteria

- The response clearly commits to an execution loop rather than a single recommendation.

## Fixture 6: Rollback Path

### User request

`优化这个 prompt chain，但如果效果不明显就撤回并试下一个方案。`

### Must do

- Include rollback as a valid post-validation decision.
- Explicitly allow moving to the next target after a weak result.

### Must not do

- Do not stack speculative edits without validation.

### Pass criteria

- The plan includes a real rollback decision path, not just optimistic forward motion.

## Fixture 7: Long-Context Recovery

### User request

Use after a long, noisy thread where the target was already confirmed earlier.

### Must do

- Reload the active state from the current method and snapshot.
- Restate target, goal, validation, and next action before continuing.

### Must not do

- Do not rely on stale assumptions from earlier branches of the conversation.

### Pass criteria

- The skill explicitly reconstructs the current state before resuming execution.

## Fixture 8: Validation Missing

### User request

`优化这个内容流程。`

### Must do

- Confirm target.
- Ask what success should mean if not inferable.
- Confirm validation basis before execution.

### Must not do

- Do not pretend the evaluation basis is objective if it has not been confirmed.

### Pass criteria

- The skill refuses to execute blindly without a success signal.

## Fixture 9: Execution Scope Ambiguous

### User request

`优化当前项目，必要的话顺手把安装副本也处理掉。`

### Must do

- Confirm whether installed copies are in scope.
- Prefer workspace-first changes.

### Must not do

- Do not change installed copies first.
- Do not assume sync is in scope without confirmation.

### Pass criteria

- The workspace remains the source of truth and sync scope is explicitly clarified.

## Fixture 10: Should Hand Off

### User request

`用 boost 优化这个数据库 schema 的迁移计划。`

### Must do

- Trigger the skill if useful for structuring the optimization loop.
- Hand off implementation details to a more specialized migration workflow if one exists.

### Must not do

- Do not pretend this skill alone is the best implementation path when a more specialized workflow clearly fits.

### Pass criteria

- The skill keeps the optimization loop but hands implementation to the better specialized path.

## Fixture 11: Locked Evaluator, Narrow Surface

### User request

`参考 autoresearch，把这个 prompt workflow 优化一下。只改 prompt，不要顺手改评测脚本。`

### Must do

- Confirm the target and goal.
- Explicitly state the mutable surface is the prompt workflow.
- Explicitly state the evaluator or review script is locked for this round.
- Design the experiment around that locked evaluator.

### Must not do

- Do not silently broaden scope into the benchmark or acceptance harness.
- Do not blur "what changes" with "what judges the change".

### Pass criteria

- The response includes a surface confirmation and treats the evaluator as fixed.

## Fixture 12: Research Ledger Memory

### User request

`继续优化昨天试过两轮都失败的 agent 流程，这次别再重复死路。`

### Must do

- Trigger the skill.
- Reconstruct or request the prior experiment outcomes if they are not in context.
- Carry forward rejected ideas or failed paths into the next iteration plan.
- Show a compact ledger or memory of what was tried and what was learned.

### Must not do

- Do not propose the same failed experiment again without new evidence.

### Pass criteria

- The skill explicitly uses prior experiment memory to steer the next move.

## Fixture 13: Claude Code Concision And Structure

### User request

`把 boost 这个 skill 本身当目标，优化它在 Claude Code 中的表现。`

### Must do

- Trigger the skill.
- Confirm the target is the skill itself.
- Keep the main response structure compact instead of turning it into a long essay.
- Prefer a compact tagged or equivalently structured state when the context is dense.
- Improve Claude-facing guidance rather than only tuning Codex metadata.

### Must not do

- Do not solve a Claude Code problem by only editing `agents/openai.yaml`.
- Do not add lots of abstract prose to `SKILL.md` without tightening execution quality.

### Pass criteria

- The resulting changes clearly improve Claude-facing structure, concision, or examples.

## Fixture 14: Claude Built-In Subagent Choice

### User request

`用 boost 优化这个项目，先收集日志和搜索结果，但别把主线程上下文刷爆。`

### Must do

- Recognize that read-heavy evidence gathering may justify Claude Code built-in `Explore` or another narrow read-only sidecar path.
- Prefer upgrading to a sidecar path by default when the Plan phase identifies read-heavy evidence gathering that would flood the main thread context.
- Keep the final integration and decision in the main loop.
- Explain the delegation choice in terms of context preservation, not vague agent enthusiasm.

### Must not do

- Do not delegate the critical-path decision-making just because subagents exist.
- Do not keep all raw logs in the main thread if a bounded sidecar path is clearly better.

### Pass criteria

- The skill chooses or recommends a context-preserving sidecar path appropriately.

- The handoff is explicit, reasoned, and does not lose the optimization structure.

## Fixture 15: Skill Before Agent Orchestration

### User request

`用 boost 优化这个前端功能，先帮我判断要不要走 superpowers 的流程；如果需要，再决定哪些探索工作该交给 agent。`

### Must do

- Recognize that the first decision is workflow selection, not delegation.
- Invoke or recommend the matching superpowers process skill before choosing any subagent path.
- Decide on agent use only after the governing skill path is clear.
- Keep target resolution, contract framing, and final execution choice in the main thread.

### Must not do

- Do not jump straight to an agent just because exploration might happen later.
- Do not use a subagent as a substitute for deciding whether brainstorming, debugging, TDD, or another process skill should run first.

### Pass criteria

- The response shows an explicit skill-first decision followed by a bounded agent decision only if still warranted.

## Fixture 17: Parallel Sidecars Should Delegate

### User request

`优化这个项目的测试稳定性。先自己改核心逻辑，同时把日志模式分析和回归用例补充并行推进。`

### Must do

- Keep the immediate core task in the main loop if it is on the critical path.
- Delegate the log analysis and regression-fixture work as bounded sidecar tasks when the host supports subagents.
- Define clear ownership for each delegated subtask.
- Treat delegation here as an efficiency optimization, not as a mandatory ceremony for every task.

### Must not do

- Do not serialize obviously parallel sidecar work without a reason.
- Do not send ambiguous "help me with this project" delegation requests.

### Pass criteria

- The plan chooses bounded parallel delegation and keeps the main thread responsible for integration and final validation.

## Fixture 18: Risky Change Should Consider Worktree Isolation

### User request

`优化这个 repo 的大规模重构路径。可以实验，但不要把主工作区搞乱，失败了要容易回退。`

### Must do

- Recognize that isolation may be part of the optimization strategy.
- Prefer a worktree or equivalent isolated checkout when the change is broad, risky, or rollback-sensitive.
- Upgrade to isolated execution by default when the planned experiment would materially destabilize the current workspace or make rollback noisy.
- Keep the main workspace as the source of truth until the experiment is validated.
- Treat the worktree as an optional high-value path, not as a mandatory precondition for all refactors.

### Must not do

- Do not run a broad risky experiment directly in the main workspace by default.
- Do not treat the isolated branch as validated production state before results are checked.

### Pass criteria

- The response explicitly selects isolated execution and explains how validated results return to the main workspace.

## Fixture 19: Claude Adapter Should Stay Claude-Specific

### User request

`优化这个 skill 项目对 Claude Code 的适配，重点是 subagents 和 worktree 的使用时机。`

### Must do

- Update `CLAUDE.md` or other Claude-facing adapter files when host-specific guidance is needed.
- Keep the shared method in `SKILL.md` host-agnostic.
- Preserve the workspace-first sync model.

### Must not do

- Do not move Claude-specific command language into the shared method unless it is truly cross-host.

### Pass criteria

- Claude-specific behavior is captured in Claude adapter files without polluting the cross-host method.

## Fixture 20: Codex Adapter Should Stay Codex-Specific

### User request

`优化这个 skill 项目对 Codex 的适配，补齐 Codex 侧项目说明和触发提示。`

### Must do

- Update `AGENTS.md` and `agents/openai.yaml` when Codex-specific adaptation is required.
- Keep `agents/openai.yaml` concise and trigger-oriented.
- Keep operational detail in `AGENTS.md`, `SKILL.md`, or `references/`.

### Must not do

- Do not overload `agents/openai.yaml` with the full method.
- Do not leave Codex without a project-level adapter when the project maintains one for Claude.

### Pass criteria

- Codex has an explicit adapter layer and its metadata remains aligned with the shared method.

## Fixture 21: Claude Adapter Should Avoid Overtrigger Language

### User request

`优化这个 skill 的 Claude Code 适配，避免它因为提示太凶而乱触发 subagent 或 worktree。`

### Must do

- Keep Claude-facing guidance explicit.
- Reduce exaggerated trigger wording that would cause unnecessary delegation or isolation.
- Preserve the shared method while refining the Claude adapter.

### Must not do

- Do not replace clarity with vague language.
- Do not encode every host preference as a hard requirement.

### Pass criteria

- The Claude adapter remains clear, but it no longer encourages overtriggering by tone alone.

## Fixture 22: Codex Adapter Should Read Like Project Operating Notes

### User request

`优化这个 skill 的 Codex 适配，让 Codex 更像在处理一个工程 issue，而不是在读一堆抽象方法论。`

### Must do

- Make `AGENTS.md` practical and execution-oriented.
- Emphasize explicit scope, source of truth, and validation evidence.
- Keep metadata concise.

### Must not do

- Do not dump the whole method into `AGENTS.md` or `agents/openai.yaml`.
- Do not leave validation expectations implicit.

### Pass criteria

- Codex-facing instructions read like repo operating notes and support reliable execution.

## Fixture 23: Visible State Checkpoint On Multi-Iteration Run

### User request

`用 boost 持续优化这个流程，跑几轮直到达标。`

### Must do

- Emit a visible `[boost · Iter N]` checkpoint line at the start of each iteration after the first.
- The checkpoint must include: iteration number, target, last decision (keep/rollback/next + what changed), next target, guardrails status.

### Must not do

- Do not only emit the state snapshot internally — it must appear in the user-visible output.
- Do not skip the checkpoint even when context is long or noisy.

### Pass criteria

- Every iteration boundary in the output has a `[boost · Iter N]` line with the required state.
- The user can read the output and confirm the task never left the boost method.

## Fixture 24: Stable Contract Must Be Established

### User request

`用 boost 优化这个 API 的响应时间。`

### Must do

- Trigger the skill.
- Establish a Stable Contract with all four fields (Goal, Constraints, Done, Checks) before entering the PDCAA loop.
- If the user's request doesn't provide enough for all four fields, ask minimal clarifying questions or infer from context.

### Must not do

- Do not enter the Plan phase without a complete Stable Contract.
- Do not leave Constraints or Checks empty without explanation.

### Pass criteria

- A visible Stable Contract block appears before any Plan output.
- All four fields are filled with actionable content.

## Fixture 25: Align Phase Must Detect Drift

### User request

Simulate: user initially asks to optimize response time, then mid-conversation says "顺便把错误处理也改了".

### Must do

- Detect the scope expansion as a drift signal in the Align phase.
- Flag it explicitly: goal drift or local-problem hijack.
- Either reject the expansion and stay on target, or propose it as a Contract Candidate.

### Must not do

- Do not silently absorb the new request into the current Goal.
- Do not skip the Align check after the scope-expanding input.

### Pass criteria

- The Align output names the drift type and takes corrective action.
- The Stable Contract is not modified without explicit acknowledgment.

## Fixture 26: Log Output Is Mandatory

### User request

`优化这个 prompt 的准确率，跑两轮。`

### Must do

- Every PDCAA cycle produces a visible log block with all 11 mandatory fields (round_id, goal, current_action, action_reason, execution_result, check_result, align_result, feedback, decision, next_step, log_summary).
- The log appears in the user-visible output, not hidden.

### Must not do

- Do not skip the log even when the cycle is short or simple.
- Do not abbreviate the log to fewer than the mandatory fields.

### Pass criteria

- Both iteration cycles have complete, visible log blocks.

## Fixture 27: AutoResearch Triggers Correctly

### User request

`优化这个 workflow，我试了好几种方案都不太行。`

### Must do

- Recognize the "multiple failed approaches" signal as an AutoResearch trigger.
- Activate AutoResearch: list candidates, compare against Stable Contract, propose a minimum probe.
- Feed the probe action back into Plan, not directly into execution.

### Must not do

- Do not propose the same approaches the user already tried.
- Do not skip AutoResearch and go straight to a random new attempt.

### Pass criteria

- AutoResearch output block appears with trigger reason, candidates, and recommendation.
- The subsequent Plan phase uses the AutoResearch output.

## Fixture 28: AutoReceive Handles New Constraints

### User request

Mid-loop, user says: "补充一下，改动不能影响移动端的渲染。"

### Must do

- AutoReceive captures the new constraint.
- Mark it as a Contract Candidate if it modifies the Stable Contract.
- Integrate into Constraints field after acknowledgment.
- Next Align phase verifies the new constraint is being respected.

### Must not do

- Do not ignore new constraints received mid-execution.
- Do not modify the Stable Contract silently.

### Pass criteria

- Contract Candidate is explicitly stated.
- The constraint appears in subsequent Check/Align phases.

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

## Minimal Rubric

Use this quick rubric when reviewing behavior:

- `Target correctness`: correct / ambiguous / wrong
- `Stable Contract`: complete / partial / missing
- `PDCAA discipline`: followed / skipped phases / disordered
- `Align phase`: executed / skipped / detected drift correctly
- `Log output`: complete / partial / missing
- `Validation discipline`: explicit / weak / missing
- `Safety behavior`: safe / risky / wrong-target

If any of `Target correctness`, `Stable Contract`, or `Safety behavior` is wrong, the fixture fails even if other parts look acceptable.
