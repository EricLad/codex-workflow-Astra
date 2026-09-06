# Escalation Policy

## Principle

Do not escalate based only on task size. Escalate when higher reasoning provides unique value.

## Luna Max remains default

Use GPT-5.6 Luna Max for:

- implementation;
- debugging;
- refactoring;
- testing loops.

## Escalate to Astra Architect

Conditions:

- architecture decisions are required;
- module ownership is unclear;
- multiple valid designs exist;
- long-term maintainability is affected.

## Escalate to Astra Diagnostician

Conditions:

- crash root cause is unclear;
- concurrency issue exists;
- lifetime or ownership bug suspected;
- two evidence-based Luna attempts failed.

## After Astra

Astra returns:

- decision artifact;
- root cause;
- constraints;
- implementation guidance.

Luna resumes implementation.