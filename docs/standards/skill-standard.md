# Skill Standard

This document defines the standard structure for every skill in **DAS Intelligence Skills**.

## Purpose

A skill is a reusable professional operating module for an AI assistant. It should help the assistant perform a specific business task with consistency, quality, and practical judgment.

A skill is not only a prompt. A complete skill contains:

- instructions;
- operating procedures;
- checklists;
- templates;
- examples;
- evaluation tests;
- metadata.

## Required Structure

```text
skill-name/
├── SKILL.md
├── README.md
├── metadata.yaml
├── references/
│   └── frameworks.md
├── templates/
│   └── outputs.md
├── examples/
│   └── example-1.md
└── evals/
    └── benchmark.md
```

## SKILL.md Required Sections

Every `SKILL.md` should include:

1. Title
2. Purpose
3. When to use
4. When not to use
5. Expert role
6. Core principles
7. Operating procedure
8. Required inputs
9. Output formats
10. Quality checklist
11. Common mistakes
12. Related skills

## Naming Convention

Use lowercase kebab-case:

```text
brand-strategy-expert
website-cro-auditor
ozon-card-optimizer
```

## Quality Levels

### Core

Minimum useful version of a skill.

### Professional

Full working version with templates, examples, and checklists.

### Enterprise

Advanced version with SOPs, KPIs, evaluation tests, benchmarks, and edge cases.

## Writing Rules

- Be specific.
- Avoid vague business language.
- Use operational steps.
- Include examples.
- Avoid unsupported claims.
- Prefer practical outputs over theory.
- Never include placeholders in final templates.
