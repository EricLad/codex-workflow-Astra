# Context Budget Policy

## Goal

Maintain quality while avoiding unnecessary context consumption.

## Rules

- Do not ask Astra to reread the entire repository for reviews.
- Prefer targeted files, diffs, and decision packets.
- Keep handoff messages concise and evidence-based.
- Use Long Running mode only when historical decisions provide value.

## Review Context

Preferred input:

- goal;
- design;
- changed files;
- diff;
- test result;
- known risks.

Avoid:

- unrelated source files;
- duplicated explanations;
- full repository dumps without reason.