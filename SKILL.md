---
name: codex-workflow-astra
description: Cost-aware software engineering workflow that routes routine implementation to GPT-5.6 Luna and escalates architecture, difficult debugging, high-risk reasoning, and critical review to GPT-6 Astra.
---

# Codex Workflow Astra

Use this skill to execute software-engineering tasks with a cost-aware two-model policy.

## Priority

The user's explicit instructions take precedence over this skill. Target-repository rules such as `AGENTS.md`, build requirements, safety constraints, and repository-specific policies remain binding. If this skill conflicts with a higher-priority instruction, follow the higher-priority instruction and state the conflict only when it materially changes execution.

Do not stop merely because a routine detail is unspecified. Infer low-risk details from the repository and continue. Ask only when missing information could materially change the result or cause an irreversible/destructive action.

## Objective

Maximize engineering quality per unit of expensive-model usage.

Default policy:

- GPT-5.6 Luna is the implementation and throughput model.
- GPT-6 Astra is the decision, diagnosis, orchestration, and critical-review model.
- Reasoning effort is selected independently from the model.
- After Astra resolves the high-value question, return implementation work to Luna immediately.

The canonical loop is:

`Astra Think → Luna Build → Astra Judge → Luna Finish`

Not every task needs every stage. Routine tasks should remain Luna-only.

## 1. Classify before acting

Classify the next meaningful unit of work into one of these classes.

### L0 — Mechanical

Examples:

- rename or deterministic text replacement;
- formatting or comments;
- repository search and inventory;
- trivial build-file edits with an exact requested change;
- obvious include/import fixes.

Route: **Luna `none` or `low`**.

Prefer `low` when code semantics are involved. Use `none` only when the work is genuinely mechanical and the runtime supports it.

### L1 — Routine engineering

Examples:

- normal feature implementation from clear requirements;
- CRUD and ordinary data-flow changes;
- conventional UI work;
- normal tests;
- ordinary bug fixes;
- CMake/package/build configuration;
- bounded refactoring with clear behavior.

Route: **Luna `medium`**.

This is the default route.

### L2 — Complex implementation

Examples:

- multi-file implementation with nontrivial state flow;
- moderately difficult debugging;
- performance-sensitive implementation with measurable criteria;
- complex parser/serialization logic;
- implementation requiring careful interaction among several known components.

Route: **Luna `high`**.

Do not escalate merely because the change touches many files. Escalate because the reasoning risk is high, not because the diff is large.

### A1 — High-value decision

Examples:

- architecture and module boundaries;
- competing interface designs;
- significant migration strategy;
- cross-cutting design with several viable approaches;
- task decomposition where a wrong plan would create substantial rework;
- independent review of an important change.

Route: **Astra `low`**, increasing to `medium` only when the problem genuinely requires deeper cross-system reasoning.

### A2 — High-risk / hard diagnosis

Examples:

- concurrency, deadlocks, shutdown ordering, races;
- object lifetime, ownership, use-after-free, undefined behavior;
- exception boundaries that may terminate a process;
- authentication, authorization, cryptography, protocol compatibility;
- schema/data migrations with rollback or integrity risk;
- difficult root-cause analysis after disciplined Luna attempts fail.

Route: **Astra `high`**.

Use `xhigh` only for unusually difficult system-level reasoning. Do not automatically use `max`.

### A3 — Exceptional final judgment

Use only when the problem is both exceptionally difficult and consequential, and additional reasoning depth is justified by evidence.

Route: **Astra `max` only by explicit escalation**.

The workflow must never drift into `max` as a normal fallback.

## 2. Escalation rules

Escalate Luna → Astra when at least one of these is true:

1. The task requires a consequential architecture/interface decision with multiple credible alternatives.
2. The task contains concurrency, lifetime, UB, security, migration, or protocol-integrity risk that exceeds routine implementation.
3. A root cause remains unclear after **two evidence-based Luna attempts** on the same failure.
4. The implementation is complete but a critical change warrants independent high-confidence review.
5. Conflicting constraints require broader reasoning to determine the correct tradeoff.

Do not escalate for:

- ordinary compilation errors;
- mechanical multi-file edits;
- normal tests;
- routine documentation;
- clear implementation work already specified by a plan;
- large repository size by itself.

## 3. Retry budget

For a nontrivial failure, use a hypothesis-driven loop:

1. collect evidence;
2. state the current hypothesis internally;
3. make one bounded change;
4. run the smallest meaningful verification;
5. update the hypothesis from the result.

Do not repeat essentially the same unsuccessful approach more than twice.

After two failed evidence-based attempts on the same underlying problem:

- stop speculative patching;
- package the evidence;
- escalate diagnosis to Astra;
- once Astra identifies the likely root cause and fix strategy, return implementation to Luna.

## 4. De-escalation rule

Astra is not the default implementation worker.

When Astra has produced a usable:

- architecture decision;
- implementation plan;
- root-cause diagnosis;
- review finding;
- risk assessment;

capture that result in a concise handoff packet and route execution back to Luna unless the remaining work still requires Astra-class reasoning.

Never keep Astra active merely because Astra started the task.

## 5. Review policy

Use Astra review only when risk justifies it.

### Luna-only review is sufficient for

- low-risk local changes;
- conventional feature implementation;
- straightforward tests and build changes;
- mechanical refactors with strong automated verification.

### Astra review is appropriate for

- public/core interfaces;
- concurrency and lifetime behavior;
- security-sensitive logic;
- data integrity/migrations;
- large architecture changes;
- changes whose failure would be expensive or difficult to detect.

For Astra review, provide a **Review Packet** rather than asking it to reread the entire repository by default:

- goal and acceptance criteria;
- design decisions;
- changed files or diff;
- only the relevant interfaces/call chain;
- validation results;
- known risks;
- specific review focus.

Expand repository inspection only when the evidence indicates it is necessary.

## 6. Validation policy

Validation must be proportional to change risk.

For each implementation:

1. run targeted tests for changed behavior when available;
2. run type/lint/static checks relevant to the changed area when useful;
3. build affected targets/packages;
4. run a minimal smoke test when it adds meaningful confidence.

Do not automatically run broad, expensive test suites for reversible low-impact changes when targeted verification is sufficient.

Broaden testing when:

- targeted checks fail;
- the change affects shared/core behavior;
- the repository requires full validation;
- the risk profile justifies it.

## 7. Parallel work

Parallelize only genuinely independent workstreams.

Prefer multiple Luna workers for independent implementation tasks. Use Astra to coordinate parallel work only when decomposition, integration, or conflict resolution itself needs higher-level reasoning.

Avoid multiple Astra workers unless independent hard problems truly require them.

When temporary worktrees or branches are created, track ownership and clean them up after successful integration unless the user or repository workflow requires retaining them.

## 8. Runtime model selection

This skill defines routing policy; it must not pretend the runtime switched models when it did not.

If the current environment can choose model and reasoning per subagent/thread:

- delegate using the selected route;
- pass only the context needed for that task;
- keep expensive-model context narrow.

If the current environment cannot perform automatic model selection:

- produce a handoff block with the exact target model, reasoning effort, task, evidence, constraints, and success criteria;
- do not claim that the route has been executed by another model.

Use this format:

```text
ROUTE HANDOFF
Model: GPT-5.6 Luna | GPT-6 Astra
Reasoning: none | low | medium | high | xhigh | max
Purpose: <implementation | diagnosis | planning | review>
Task: <bounded task>
Context: <minimal sufficient context>
Constraints: <must preserve>
Success criteria: <observable completion conditions>
Evidence: <tests/logs/diff when relevant>
Return: <what the receiving model must produce>
```

## 9. Completion criteria

A task is complete only when:

- requested behavior is implemented or the requested analysis is finished;
- applicable acceptance criteria are satisfied;
- appropriate validation has been run, or the inability to run it is stated;
- failed experimental changes are not left behind;
- temporary artifacts/worktrees/branches created by this workflow are cleaned up unless intentionally retained;
- the final response states what changed, validation performed, and any remaining material risk.

Do not add speculative improvements outside scope solely because a stronger model is available.

## 10. Default decision table

When uncertain, prefer the cheaper sufficient route:

```text
Mechanical?                       -> Luna low
Normal engineering?              -> Luna medium
Complex but bounded implementation? -> Luna high
Architecture/critical decision?  -> Astra low/medium
Hard diagnosis/high-risk logic?  -> Astra high
Exceptional system problem?      -> Astra xhigh
Max?                             -> explicit exceptional escalation only
```

Always return to Luna after the Astra-only reasoning step unless the next step independently qualifies for Astra.