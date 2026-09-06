# Agent Architecture

## Overview

codex-workflow-Astra uses a Root Agent plus specialized subagents.

The root agent is not replaced during escalation. It delegates bounded tasks to specialized agents and continues the main workflow after receiving results.

## Root Agent

Default:

```yaml
model: GPT-5.6 Luna
reasoning: medium
```

Responsibilities:

- own the user conversation;
- classify the next work unit;
- decide whether delegation is justified;
- merge results from subagents;
- maintain final delivery quality.

The root agent should avoid using Astra for routine implementation.

## Agent Profiles

### Luna Explorer

```yaml
model: GPT-5.6 Luna
reasoning: low
```

Purpose:

- repository exploration;
- locate files and dependencies;
- trace call chains;
- collect implementation context.

Output:

- relevant files;
- dependency map;
- risks discovered;
- recommended next step.

---

### Luna Implementer

```yaml
model: GPT-5.6 Luna
reasoning: medium
```

Purpose:

- implement planned changes;
- modify source code;
- compile;
- test;
- fix normal errors.

Upgrade to high only when implementation requires deeper reasoning.

---

### Astra Architect

```yaml
model: GPT-6 Astra
reasoning: low-medium
```

Purpose:

- architecture decisions;
- module boundaries;
- migration strategy;
- interface design;
- decomposition of complex work.

Restrictions:

- do not become the default coder;
- avoid broad repository rewriting;
- return a decision artifact.

---

### Astra Reviewer

```yaml
model: GPT-6 Astra
reasoning: low-medium
```

Purpose:

Review completed or partially completed work.

Input should be limited to:

- goal;
- design decision;
- diff;
- changed files;
- validation result;
- specific review questions.

Focus:

- correctness;
- regression risk;
- architecture consistency;
- hidden lifetime/concurrency issues.

---

### Astra Diagnostician

```yaml
model: GPT-6 Astra
reasoning: high
```

Purpose:

Only for:

- unclear crash causes;
- race conditions;
- deadlocks;
- lifetime bugs;
- undefined behavior;
- security-critical failures.

Trigger:

Luna has attempted evidence-based diagnosis twice without resolving the root cause.

## Delegation Rules

Preferred flow:

```
Luna Root
   |
   +-- Luna Explorer
   |
   +-- Astra Architect (only when design decisions exist)
   |
   +-- Luna Implementer
   |
   +-- Astra Reviewer (only for justified risk)
   |
   +-- Luna Finish
```

## Handoff Principle

Astra produces knowledge artifacts, not ownership transfer.

After Astra completes:

```
Astra decision
      |
      v
handoff packet
      |
      v
Luna implementation
```

This keeps expensive reasoning focused on high-value decisions while preserving implementation throughput.
