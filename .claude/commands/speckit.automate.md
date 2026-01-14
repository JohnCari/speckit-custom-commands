---
description: Automated speckit workflow that runs all phases (specify → clarify → plan → tasks → implement → test) in sequence. Only invoke when user explicitly requests /speckit.automate
---

## User Input

```text
$ARGUMENTS
```

The feature description or @file reference provided above is passed to the specify phase. No additional input is needed for subsequent phases.

## Outline

Automate the complete speckit workflow: specify → clarify-auto → plan → tasks → checklist-auto → analyze-auto → implement → test

### Pre-Flight Directory Check (REQUIRED)

Before starting any speckit phase, ensure you are in the repository root where `.specify/` lives:

1. **Locate the speckit root**: Find the directory containing `.specify/` folder
2. **Navigate there**: `cd` to that directory before running any speckit commands
3. **Verify**: Run `ls -d .specify` to confirm `.specify/` is in current directory

```bash
# Find and navigate to speckit root
cd "$(dirname "$(find /workspaces -name '.specify' -type d 2>/dev/null | head -1)")"
ls -d .specify  # Should output: .specify
```

**Why this matters**: All speckit scripts use `git rev-parse --show-toplevel` to determine where to create branches and specs. If you're in a nested git repo (like supabase-etl/), specs will be created in the wrong location.

### Execution Steps

**Continuous Execution**: After completing each step below, IMMEDIATELY invoke the next skill without stopping. Do not show intermediate summaries or wait for confirmation between phases.

1. **Specify**: Invoke `speckit.specify` with args: `$ARGUMENTS` → creates spec.md
2. **Clarify**: Invoke `speckit.clarify-auto` (no args) → auto-resolves ambiguities in spec.md
3. **Plan**: Invoke `speckit.plan` (no args) → creates plan.md
4. **Tasks**: Invoke `speckit.tasks` (no args) → creates tasks.md
5. **Checklist**: Invoke `speckit.checklist-auto` (no args) → auto-generates requirement quality checklists
6. **Analyze**: Invoke `speckit.analyze-auto` (no args) → auto-fixes consistency issues across artifacts
7. **Implement**: Invoke `speckit.implement` (no args) → builds the feature
   - **⚠️ MANDATORY**: After implement completes, you MUST immediately invoke speckit.test - do NOT stop here
8. **Test**: Invoke `speckit.test` (no args) → verifies quality gates
   - **⚠️ THIS STEP IS REQUIRED** - the workflow is NOT complete until test runs
   - Only after test completes should you provide the final summary

### Rules

- Use the Skill tool for each invocation
- Run all phases automatically without pausing
- **CRITICAL - TEST IS MANDATORY**: The workflow is NOT complete after implement. You MUST invoke speckit.test after speckit.implement completes. Never end the workflow without running test.
- **CRITICAL**: After each phase completes, IMMEDIATELY invoke the next skill in the same response - do NOT stop to show intermediate results or wait for user acknowledgment
- Only provide a final summary after ALL phases complete successfully, or stop immediately if a phase fails
- Each skill invocation should be in a new tool call immediately following the previous phase's completion
- Only the specify phase receives arguments (the user's input)
- Subsequent phases use artifacts from prior phases
- Stop and report if any phase fails
- **CRITICAL**: Always run the Pre-Flight Directory Check before Phase 1 (Specify)
- The `.specify/` directory location determines where feature branches and specs are created
- Never run speckit commands from inside nested git repositories (like supabase-etl/)
