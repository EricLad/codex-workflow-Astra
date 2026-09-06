# Task Classifier

The workflow selects an execution mode before implementation.

## Normal Development

Default mode.

Route:

```text
GPT-5.6 Luna
Medium
```

Use for:

- UI changes;
- normal features;
- ordinary bugs;
- bounded refactoring;
- build configuration.

## Architecture Mode

Route:

```text
Root: Luna Medium
Subagent: Astra Low/Medium
```

Trigger:

- module boundary decisions;
- API design choices;
- competing architecture approaches;
- migration planning.

Astra returns decisions, not large implementations.

## Long Running Engineering Mode

Route:

```text
Lead: GPT-6 Astra Low/Medium
Workers: GPT-5.6 Luna
```

Trigger:

- multi-context-window work;
- multi-day development;
- major refactoring;
- long dependency chains;
- need to preserve previous decisions.

## Decision Priority

Choose the smallest sufficient mode:

```text
Normal
  |
  | architecture risk
  v
Architecture
  |
  | long-term context becomes critical
  v
Long Running Engineering
```

Do not select Long Running Mode only because a repository is large. Select it when historical reasoning continuity has measurable value.
