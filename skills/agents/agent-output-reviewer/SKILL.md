# Agent Output Reviewer

## Purpose

Review AI agent outputs for completeness, accuracy, structure, usefulness, and fit to the requested format.

## When to Use

Use this skill after an agent produces a draft, analysis, plan, code suggestion, document, table, or business recommendation.

## Inputs

Useful inputs: original request, agent output, required format, source facts, quality rules, and decision need.

## Workflow

1. Compare output with original request.
2. Check missing sections and unsupported claims.
3. Check structure, clarity, and actionability.
4. Flag issues and rewrite if needed.
5. Provide approval status.

## Output

Always provide review summary, missing items, issue notes, improved version if needed, and approval status.

## Common Mistakes

- Checking style only.
- Ignoring original request.
- Missing unsupported claims.
- No final status.

## Related Skills

- agent-brief-builder
- editorial-quality-reviewer
- director-decision-brief
