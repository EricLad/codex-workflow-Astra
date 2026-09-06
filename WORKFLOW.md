# Execution Workflow

This document defines the operational state machine for `codex-workflow-astra`.

## State 0 — Intake

Capture:

- requested outcome;
- explicit constraints;
- repository rules;
- acceptance criteria;
- likely affected subsystem;
- reversibility and failure cost.

Do not begin by choosing the strongest model. Begin by choosing the **cheapest model that is likely to complete the next meaningful step correctly**.

## State 1 — Route

Classify the next unit of work.

```text
L0  mechanical                 -> Luna none/low
L1  routine engineering        -> Luna medium
L2  complex implementation     -> Luna high
A1  architecture/critical plan -> Astra low/medium
A2  hard diagnosis/high risk   -> Astra high
A3  exceptional final judgment -> Astra xhigh/max
```

Routing is performed per **unit of work**, not once for the entire user request.

A single feature may therefore use several routes:

```text
Astra medium: choose architecture
Luna medium: implement module A
Luna medium: implement module B
Luna high: resolve integration failure
Astra high: diagnose race condition
Luna medium: implement diagnosed fix
Astra low: review critical diff
Luna low: apply review cleanup
```

## State 2 — Inspect minimally

Read enough code to establish the relevant execution path and constraints.

Prefer:

1. repository instructions;
2. directly named files;
3. symbols/callers related to the requested behavior;
4. tests that define current behavior;
5. adjacent implementation only when needed.

Avoid broad repository scans when the task is bounded.

For Astra, keep context especially narrow. Provide a curated context packet first and expand only when the decision requires it.

## State 3 — Plan at the required depth

### Luna work

For L0/L1, do not over-plan. Use a short internal sequence and execute.

For L2, identify:

- affected components;
- invariants;
- expected changes;
- validation commands;
- rollback path if risky.

### Astra work

For A1/A2, the output should be a decision artifact, not an open-ended essay.

A useful Astra result includes:

- decision or root cause;
- evidence;
- rejected alternatives when materially relevant;
- implementation constraints;
- validation criteria;
- unresolved risk.

Once this artifact exists, de-escalate.

## State 4 — Execute with Luna by default

Implementation belongs to Luna unless implementation itself still requires Astra-class reasoning.

During implementation:

- preserve existing behavior outside scope;
- make the smallest coherent change;
- avoid opportunistic refactors unless required;
- follow existing repository patterns before introducing abstractions;
- keep generated changes reviewable;
- update tests only when they add meaningful behavioral coverage.

## State 5 — Validate incrementally

Use the smallest check that can falsify the current change.

Typical order:

```text
focused unit test
  -> affected package/target build
  -> targeted integration test
  -> broader regression suite only if justified
```

Do not repeatedly run expensive validation when no new evidence or code change justifies it.

## State 6 — Failure loop

When validation fails:

```text
observe
  ↓
form/update hypothesis
  ↓
make bounded change
  ↓
run focused check
  ↓
collect evidence
```

### Attempt counter

Count attempts by **underlying hypothesis**, not by individual shell command.

After two evidence-based attempts fail to resolve the same problem:

1. stop speculative edits;
2. revert or isolate failed experiments when appropriate;
3. build a Diagnostic Packet;
4. route to Astra at the risk-appropriate reasoning level.

A third Luna attempt is allowed only when the second attempt produced materially new evidence that changes the hypothesis.

## State 7 — Astra diagnosis

A Diagnostic Packet should contain:

```text
Symptom
Expected behavior
Reproduction steps
Relevant logs/errors
Relevant code path
Changes already attempted
Result of each attempt
Current hypotheses
Constraints
```

Astra should return:

```text
Most likely root cause
Confidence / uncertainty
Evidence chain
Recommended fix
What not to change
Validation required
```

Then return to Luna for implementation.

## State 8 — Review gate

Determine whether independent Astra review is justified.

### Skip Astra review when

- the change is low risk;
- behavior is strongly covered by targeted tests;
- the implementation is conventional;
- failure is easy to detect and reverse.

### Use Astra review when

- the change modifies a core/public contract;
- concurrency/lifetime/security/data-integrity behavior is involved;
- architecture has materially changed;
- the diff is difficult to reason about locally;
- failure could be silent, costly, or difficult to recover from.

Default review reasoning:

```text
ordinary important diff -> Astra low
cross-module/core diff   -> Astra medium
concurrency/security     -> Astra high
```

## State 9 — Review packet

Do not ask Astra to “review the whole repo” by default.

Provide:

```text
Goal
Acceptance criteria
Design decision
Diff / changed files
Relevant interfaces
Tests and results
Known risks
Review questions
```

Astra review should focus on:

1. correctness;
2. hidden regressions;
3. violated invariants;
4. concurrency/lifetime/security issues when relevant;
5. unnecessary complexity;
6. missing validation that could plausibly catch a real defect.

Review findings must be actionable and prioritized.

## State 10 — Luna finish

Luna applies accepted review findings and performs targeted validation.

Do not send cosmetic or mechanical review fixes back to Astra.

If a review finding reveals a new hard reasoning problem, create a new A2 unit of work rather than keeping all remaining work on Astra.

## State 11 — Integration and cleanup

When parallel agents, worktrees, or temporary branches were used:

- verify each workstream's validation evidence;
- integrate in dependency order;
- resolve conflicts deliberately;
- rerun the smallest integration-level checks that cover the merged result;
- remove temporary worktrees and branches created only for completed work unless retention is required.

Do not delete user-created branches or persistent worktrees unless explicitly authorized.

## State 12 — Completion

Report only material information:

- what changed;
- important design decisions;
- tests/builds/checks run;
- unresolved risks or blockers;
- model handoff still required, if runtime switching was unavailable.

A successful workflow should normally spend most implementation turns on Luna and only a minority of high-value reasoning turns on Astra.