---
description: Automate the full speckit workflow from specification to implementation and testing
---

## User Input

```text
$ARGUMENTS
```

The feature description or @file reference provided above is passed to the specify phase. No additional input is needed for subsequent phases.

## Outline

Automate the complete speckit workflow: specify → clarify-auto → plan → tasks → checklist-auto → analyze-auto → implement → test

### Execution Steps

1. **Specify**: Invoke `speckit.specify` with args: `$ARGUMENTS` → creates spec.md
2. **Clarify**: Invoke `speckit.clarify-auto` (no args) → auto-resolves ambiguities in spec.md
3. **Plan**: Invoke `speckit.plan` (no args) → creates plan.md
4. **Tasks**: Invoke `speckit.tasks` (no args) → creates tasks.md
5. **Checklist**: Invoke `speckit.checklist-auto` (no args) → auto-generates requirement quality checklists
6. **Analyze**: Invoke `speckit.analyze-auto` (no args) → auto-fixes consistency issues across artifacts
7. **Implement**: Invoke `speckit.implement` (no args) → builds the feature
8. **Test**: Invoke `speckit.test` (no args) → verifies quality gates

### Rules

- Use the Skill tool for each invocation
- Run all phases automatically without pausing
- Only the specify phase receives arguments (the user's input)
- Subsequent phases use artifacts from prior phases
- Stop and report if any phase fails
