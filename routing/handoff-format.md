# Agent Handoff Format

## Purpose

Prevent context loss between Luna and Astra agents.

## Luna to Astra

Provide:

- task objective;
- current implementation state;
- relevant files;
- attempted solutions;
- failures and evidence;
- explicit question.

## Astra to Luna

Return:

- decision or diagnosis;
- reasoning summary;
- constraints;
- affected components;
- implementation steps;
- validation requirements.

## Rule

Do not transfer unnecessary repository history. Transfer only decision-relevant context.