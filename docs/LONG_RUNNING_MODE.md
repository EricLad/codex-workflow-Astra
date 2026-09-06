# Long Running Engineering Mode

## Purpose

Long Running Engineering Mode is used when a software task requires persistent architectural context across many development cycles. It uses GPT-6 Astra as a project-level reasoning lead while GPT-5.6 Luna remains the implementation worker.

## Enable When

Use this mode when one or more conditions apply:

- the task is expected to span multiple context windows;
- the task contains many iterations of implementation, testing, and correction;
- architectural decisions must remain consistent over time;
- the cost of forgetting previous decisions is high;
- the work is a major migration or refactor.

Examples:

- Qt application architecture migration;
- backend service redesign;
- database schema migration;
- plugin framework creation;
- large module decomposition.

## Roles

### Astra Project Lead

Model: GPT-6 Astra
Reasoning: low/medium

Responsibilities:

- maintain long-term design direction;
- record important decisions;
- resolve architectural conflicts;
- review major milestones;
- prevent repeated failed approaches.

Astra should not perform routine coding.

### Luna Workers

Model: GPT-5.6 Luna
Reasoning: low/medium/high depending on task.

Responsibilities:

- repository exploration;
- implementation;
- testing;
- build fixes;
- incremental refactoring.

## Execution Pattern

```text
Astra Project Lead
        |
        | architecture decisions
        v
Luna Explorer
        |
        v
Luna Implementer
        |
        v
Astra Review (when justified)
        |
        v
Luna Finish
```

## Cost Principle

Do not use Astra because a task is large. Use Astra when maintaining historical reasoning provides more value than the additional model cost.

Small tasks remain Luna-only.
