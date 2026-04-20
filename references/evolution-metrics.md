# Evolution Metrics

Use this reference when the main workflow needs help selecting metrics, designing evaluations, or distinguishing outcome metrics from diagnostic signals.

## Metric Layers

Measure the target object at more than one layer whenever possible.

### Outcome metrics

These indicate whether the object is producing better end results.

Examples:

- Conversion rate
- Task success rate
- Accuracy
- User satisfaction
- Retention
- Defect escape rate
- Time-to-resolution

### Process metrics

These explain how the object behaves while producing results.

Examples:

- Latency
- Queue time
- Handoff count
- Retry rate
- Tool-call count
- Failure mode frequency
- Rework ratio

### Guardrail metrics

These protect against false wins where one improvement harms the broader system.

Examples:

- Cost
- Error rate
- Safety incidents
- Compliance violations
- Maintenance burden
- On-call load
- Accessibility regressions

### Learning metrics

These measure whether the current iteration improved understanding, not just performance.

Examples:

- Hypotheses falsified
- Unknowns reduced
- Instrumentation coverage improved
- Reproducibility improved
- Confidence in diagnosis increased

## Metric Selection Rules

- Choose metrics that can be observed more than once.
- Prefer metrics close to the mechanism being changed.
- Pair a fast leading indicator with a slower outcome metric.
- Avoid vanity metrics that rise even when user value does not.
- Avoid too many metrics; one primary metric, two to four supporting metrics, and a few guardrails is usually enough.

## Evaluation Design

Before running an optimization, define:

- Baseline: what current performance looks like
- Comparator: what counts as better, worse, or neutral
- Time window: when the data is considered valid
- Variance risk: what noise may distort the reading
- Decision thresholds: what magnitude of change matters

Useful evaluation questions:

- Did the target object change in the intended direction?
- Was the change large enough to matter?
- Did any guardrail metric worsen?
- Was the result robust or likely due to noise?
- What new uncertainty appeared?

## Evidence Hierarchy

Prefer evidence in roughly this order:

1. Direct measurement from production-like behavior
2. Automated tests or repeated controlled runs
3. Logs, traces, structured feedback, or representative samples
4. Expert review
5. Intuition or anecdote

Use lower-grade evidence when necessary, but label its limitations explicitly.

## Starter Templates

### Optimization hypothesis

`If we change <intervention>, then <primary metric> should improve by <expected direction or threshold>, because <mechanism>.`

### Minimal evaluation set

- Primary metric:
- Supporting metrics:
- Guardrails:
- Baseline window:
- Validation window:
- Decision rule:

### End-of-iteration summary

- Change made:
- Result:
- Confidence:
- Side effects:
- What we learned:
- Next candidate iteration:
