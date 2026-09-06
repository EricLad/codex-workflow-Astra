# Model + Reasoning Routing

The workflow uses two independent decisions:

1. **Which model is sufficient for this unit of work?**
2. **How much reasoning should that model use?**

Never use reasoning level as a substitute for correct model selection, and never use a stronger model merely because a high reasoning level exists.

## Baseline

```text
Default model:     GPT-5.6 Luna
Default reasoning: medium

Astra entry:       low
Astra max:         manual/explicit exceptional escalation only
```

## Luna matrix

| Reasoning | Use when | Avoid when |
|---|---|---|
| `none` | deterministic/mechanical operations with almost no semantic judgment | actual code logic, debugging, tool-heavy sequences |
| `low` | small local edits, straightforward fixes, exact build/config changes | nontrivial state flow or uncertain behavior |
| `medium` | normal implementation, ordinary bugs, tests, refactors, routine agent work | hard root cause, high-risk system behavior |
| `high` | complex bounded implementation, moderately hard debug, careful multi-component changes | architectural decisions or high-risk unresolved diagnosis |
| `xhigh` | rare complex implementation where Luna remains clearly the right model | using it as a ritual before Astra escalation |
| `max` | normally disabled by workflow policy | routine development; repeated failed debugging |

### Luna default rule

If the task is ordinary software engineering and there is no explicit high-risk trigger, choose **Luna `medium`**.

### Luna promotion rule

Promote reasoning by one level when there is evidence the current level is insufficient:

- important constraints are being missed;
- several interacting state transitions must be reasoned through;
- debugging requires a deeper causal model;
- the implementation is bounded but genuinely complex.

Do not automatically climb every level. A difficult task may jump directly from Luna `high` to Astra when the problem category changes.

## Astra matrix

| Reasoning | Use when | Typical output |
|---|---|---|
| `low` | important review, normal architecture decision, task decomposition, adjudicating alternatives | decision + concise rationale + constraints |
| `medium` | cross-module architecture, migration strategy, integration planning, complex review | structured plan / tradeoff decision |
| `high` | hard root cause, concurrency, lifetime, UB, security-sensitive reasoning, data integrity | evidence chain + diagnosis + fix constraints |
| `xhigh` | exceptional system-level problem with several interacting failure modes | deep diagnosis / system design judgment |
| `max` | exceptionally consequential final judgment when lower levels have a demonstrated gap | final high-confidence assessment |

### Astra entry rule

Always enter Astra at the **lowest level appropriate to the problem category**.

Examples:

```text
"Should this module expose callbacks or an interface?"
-> Astra low

"Design a migration across several services with compatibility constraints"
-> Astra medium

"Intermittent shutdown crash involving callbacks, ownership and threads"
-> Astra high

"Rare multi-process data corruption with several plausible causal chains"
-> Astra xhigh
```

Do not use `high` merely because Astra is expensive or important. Higher effort is justified by reasoning complexity, not model prestige.

## Two-dimensional router

```text
                    LOW RISK                         HIGH RISK
                    ────────                         ─────────
Mechanical          Luna none/low
Routine coding      Luna medium
Complex coding      Luna high
Architecture                          Astra low/medium
Hard diagnosis                                      Astra high
Exceptional                                         Astra xhigh
Max                                                 explicit only
```

## Escalation gate

Escalate from Luna to Astra if one or more gates are true.

### Gate A — Decision leverage

A wrong decision would force substantial rework or lock in a poor interface/architecture.

### Gate B — Hidden failure risk

The defect can be silent or nondeterministic, especially:

- race/deadlock;
- lifetime/ownership;
- UB/memory safety;
- authentication/authorization;
- cryptography;
- schema/data integrity;
- protocol compatibility.

### Gate C — Diagnostic exhaustion

Two evidence-based Luna attempts have failed on the same underlying problem.

### Gate D — Critical independent review

The implementation is done, but an independent stronger review is justified by failure impact.

### Gate E — Constraint conflict

Multiple important constraints conflict and require a deliberate tradeoff rather than straightforward implementation.

## De-escalation gate

De-escalate Astra → Luna as soon as Astra has produced a stable artifact that can drive implementation:

- selected design;
- root cause;
- bounded implementation plan;
- prioritized review findings;
- integration strategy.

Do not ask Astra to perform routine follow-through after that point.

## Retry policy

### Correct pattern

```text
Luna medium/high
  ↓ attempt 1 + evidence
  ↓ attempt 2 + new evidence
Astra high diagnosis
  ↓
Luna implementation
```

### Wasteful pattern

```text
Luna medium
Luna high
Luna xhigh
Luna max
Astra low
Astra medium
Astra high
Astra xhigh
Astra max
```

The workflow explicitly rejects exhaustive ladder climbing.

## Review routing

| Change | Reviewer |
|---|---|
| formatting / comments / deterministic rename | Luna |
| ordinary feature + good tests | Luna |
| important local behavior | Luna high or Astra low if independent review adds value |
| core/public interface | Astra low/medium |
| cross-module architecture | Astra medium |
| concurrency / lifetime / security / data integrity | Astra high |

## Parallelism

Parallelism is also cost-routed.

Prefer:

```text
1 Astra planner (only if needed)
        ↓
N independent Luna implementers
        ↓
Luna integration
        ↓
1 Astra reviewer (only if risk gate is met)
```

Avoid:

```text
N Astra agents doing routine implementation in parallel
```

Astra parallelism is justified only when multiple independent hard-reasoning tasks exist.

## Context budget

Model routing is incomplete without context routing.

For Luna implementation, provide enough repository context to execute safely.

For Astra, prefer compact high-information context:

- relevant interfaces;
- call graph slice;
- diff;
- failing logs/tests;
- explicit constraints;
- prior hypotheses and outcomes.

Do not spend Astra context on large amounts of unrelated source merely because the repository is available.

## Final heuristic

When two routes appear equally likely to succeed, choose the cheaper route.

When the cheaper route begins producing evidence of insufficient reasoning, escalate promptly instead of allowing repetitive retries.