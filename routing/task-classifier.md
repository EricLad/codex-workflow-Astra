# Task Classifier

## Purpose

Classify incoming engineering tasks before execution.

## Normal Engineering

Select Normal mode when:

- change is limited in scope;
- no architecture decision is required;
- existing design remains valid;
- implementation can be verified through build/test.

Default:

GPT-5.6 Luna
Reasoning: max fast

## Architecture Mode

Select Architecture mode when:

- module boundaries need redesign;
- API ownership is unclear;
- multiple designs have significant tradeoffs;
- refactoring changes system structure.

Use:

GPT-6 Astra Architect

## Long Running Engineering Mode

Select when:

- work spans many context windows;
- project requires long-term decisions;
- many iterations and experiments are expected;
- consistency across milestones is critical.

Use:

GPT-6 Astra Project Lead
+
GPT-5.6 Luna Max workers

## Priority

Prefer quality and completion reliability over minimizing individual responses.