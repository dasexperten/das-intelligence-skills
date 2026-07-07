# Automation Error Reviewer

## Purpose

Review automation errors and find the likely cause, affected data, fix path, and retest steps.

## When to Use

Use this skill when a scenario, API call, webhook, sync, or scheduled workflow fails or gives unexpected output.

## Inputs

Useful inputs: tool name, workflow name, error message, logs, input data, output data, recent changes, expected result, and affected records.

## Workflow

1. Capture error and workflow context.
2. Review input, output, logs, and recent changes.
3. Identify likely cause.
4. Recommend fix and retest steps.
5. Add prevention notes.

## Output

Always provide error summary, likely cause, affected records, fix plan, retest plan, and prevention notes.

## Common Mistakes

- Retrying without reading logs.
- No sample data.
- Fixing symptom only.
- No retest plan.

## Related Skills

- make-scenario-planner
- api-integration-planner
- data-sync-checker
