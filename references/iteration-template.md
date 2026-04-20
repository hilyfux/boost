# Iteration Template

Use this template when the user asks to optimize a target object and you need a compact but executable working format.

## 1. Target Brief

```text
Target Object:
Target Confirmation:
Open Questions:
Goal:
Goal Confirmation:
Mutable Surface:
Locked Evaluator:
Surface Confirmation:
Scope:
Stakeholders:
Constraints:
Time Horizon:
```

## 2. Observability Surface

Capture what can be observed before proposing changes.

```text
Inputs:
Process Signals:
Outputs:
Side Effects:
Human Feedback:
System Feedback:
Known Blind Spots:
```

## 3. Problem Model

List only the highest-value problems first.

```text
Symptom:
Impact:
Suspected Cause:
Confidence:
Evidence:
Missing Evidence:
```

## 4. Opportunity Scorecard

Use a lightweight score so prioritization is visible and repeatable.

```text
Option:
Expected Impact: 1-5
Confidence: 1-5
Implementation Cost: 1-5
Validation Speed: 1-5
Regression Risk: 1-5
Learning Value: 1-5
Decision:
```

## 5. Experiment Card

Choose one experiment unless parallel work is clearly justified.

```text
Hypothesis:
Change:
Primary Metric:
Supporting Metrics:
Guardrails:
Baseline:
Validation Window:
Stop Condition:
Rollback Condition:
Validation Confirmation:
Execution Plan:
Execution Confirmation:
```

## 6. Experiment Ledger Entry

Record every meaningful branch in one compact line or block so the next iteration can reuse the learning.

```text
Baseline:
Attempt:
Expected Win:
Observed Result:
Decision: Keep / Revise / Rollback / Switch
Why:
Rejected Idea Memory:
```

## 7. Execution Topology

Use this section only when multiple paths, delegated sidecars, or isolated experiments are relevant. Skip it when the local path is sufficient.

```text
Triggered: yes / no
Topology: local / delegated / isolated-worktree / delegated+isolated-worktree
Why this topology:
Exploration branches:
Main-thread responsibility:
Delegated subtasks:
Delegated ownership boundaries:
Worktree purpose:
Primary path selection rule:
Integration plan:
Rollback path:
```

## 8. Decision After Validation

```text
Decision: Keep / Revise / Rollback / Switch
Reason:
Next Iteration Trigger:
```

## 9. End-of-Iteration Review

Close the loop explicitly.

```text
What Changed:
Observed Result:
Confidence:
Side Effects:
Decision:
Next Iteration:
```
