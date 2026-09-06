# Handoff Packets

A handoff exists to transfer **only the information needed for the next model to do its job correctly**.

It prevents two common sources of waste:

1. the expensive model rereads the whole repository;
2. the receiving model repeats investigation already completed by the previous model.

## General handoff

```text
ROUTE HANDOFF
Model: <GPT-5.6 Luna | GPT-6 Astra>
Reasoning: <none | low | medium | high | xhigh | max>
Purpose: <planning | implementation | diagnosis | review | integration>

Goal:
<one bounded outcome>

Context:
<minimal relevant repository/system context>

Constraints:
- <constraint 1>
- <constraint 2>

Evidence:
- <tests/logs/diff/observations>

Decisions already made:
- <decision that must not be reopened without evidence>

Success criteria:
- <observable condition 1>
- <observable condition 2>

Return:
<exact artifact expected from the receiving model>
```

## Astra planning packet

Use when Luna or the user needs a high-leverage design decision.

```text
ASTRA PLAN REQUEST
Reasoning: low | medium

Problem:
<what needs to be designed>

Current architecture:
<only relevant modules/interfaces>

Requirements:
- ...

Constraints:
- compatibility
- performance
- threading
- API stability
- other relevant constraints

Known options:
- option A
- option B

Decision needed:
<the exact question Astra must settle>

Return:
1. selected approach
2. rationale
3. rejected alternatives that matter
4. implementation boundaries
5. acceptance/validation criteria
6. material risks
```

After Astra returns this artifact, route coding to Luna.

## Luna implementation packet

Use after requirements or Astra planning are sufficiently clear.

```text
LUNA IMPLEMENTATION REQUEST
Reasoning: medium | high

Goal:
<feature/fix>

Design decision:
<short authoritative summary>

Files/components likely involved:
- ...

Must preserve:
- ...

Acceptance criteria:
- ...

Validation:
- targeted tests
- affected build target
- smoke check if relevant

Do:
implement, validate, fix ordinary failures, and report the final diff summary.

Do not:
reopen the architecture decision unless repository evidence shows it is invalid.
```

## Astra diagnostic packet

Use after a hard problem qualifies for escalation.

```text
ASTRA DIAGNOSTIC REQUEST
Reasoning: high | xhigh

Symptom:
<observable failure>

Expected behavior:
<expected result>

Reproduction:
<steps / frequency / environment>

Relevant path:
<functions/classes/threads/components>

Evidence:
<logs, stack, failing test, traces>

Attempt 1:
- hypothesis:
- change/test:
- result:

Attempt 2:
- hypothesis:
- change/test:
- result:

Constraints:
<what cannot be broken or changed>

Return:
1. most likely root cause
2. evidence chain
3. confidence and competing explanation if material
4. bounded fix strategy
5. areas that should not be changed
6. validation required to confirm the fix
```

Once diagnosis is actionable, Luna implements it.

## Astra review packet

Use for independent critical review.

```text
ASTRA REVIEW REQUEST
Reasoning: low | medium | high

Goal:
<what the change is supposed to accomplish>

Acceptance criteria:
- ...

Design decision:
<why implementation took this shape>

Changed files / diff:
<diff or concise changed-file set>

Relevant contracts:
<interfaces/invariants/threading/data rules>

Validation already passed:
- ...

Known concerns:
- ...

Review focus:
1. correctness
2. regressions
3. invariant violations
4. concurrency/lifetime/security/data integrity when applicable
5. unnecessary complexity
6. missing high-value validation

Return findings as:
- severity
- evidence/location
- failure mode
- recommended fix

Do not request unrelated cleanup or stylistic refactors.
```

## Luna finish packet

Use after review.

```text
LUNA FINISH REQUEST
Reasoning: low | medium

Original goal:
...

Accepted review findings:
1. ...
2. ...

Required edits:
...

Validation to rerun:
...

Return:
- changes applied
- checks passed/failed
- remaining material risk
```

## Parallel worker packet

Use for independent implementation workstreams.

```text
LUNA PARALLEL WORKER
Reasoning: medium

Workstream:
<single independent responsibility>

Owned files/components:
<bounded ownership>

Shared contracts:
<interfaces that must not be changed without coordinator agreement>

Dependencies:
<input expected from other workstreams, if any>

Acceptance criteria:
...

Validation:
...

Return:
- changed files
- interface changes
- validation evidence
- integration notes
```

Parallel workers must not silently edit the same owned surface unless the coordinator explicitly permits overlap.

## Integration packet

```text
INTEGRATION REQUEST
Preferred model: GPT-5.6 Luna
Reasoning: medium | high

Workstreams:
- A: result + validation
- B: result + validation
- C: result + validation

Shared contracts changed:
- ...

Known conflicts:
- ...

Goal:
integrate without reopening settled architecture.

Validation:
<minimum integration-level checks>

Escalate to Astra only if integration exposes a new architecture/high-risk reasoning problem.
```

## Packet quality rules

A good packet is:

- **bounded** — one next job;
- **evidence-rich** — facts instead of long narrative;
- **decision-preserving** — settled questions are not repeatedly reopened;
- **context-minimal** — enough to reason correctly, not everything available;
- **testable** — success criteria are observable.

A bad packet looks like:

```text
Please read the entire repository, understand everything, figure out what the other model did, then continue.
```

That defeats the cost-control purpose of the workflow.