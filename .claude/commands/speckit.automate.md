---
description: Automated speckit workflow (specify → plan → tasks → implement → test). Only invoke when user explicitly requests /speckit.automate
---

## User Input

```text
$ARGUMENTS
```

The feature description or @file reference provided above is passed to the specify phase. No additional input is needed for subsequent phases.

## Outline

Automate the complete speckit workflow: **specify → plan → tasks → implement → test**

> **Note**: For interactive spec clarification, run `/speckit.clarify` before invoking `/speckit.automate`.

---

### Pre-Flight Checks (REQUIRED)

Before starting any phase, verify the environment:

1. **Locate speckit root**: Find the directory containing `.specify/` folder
2. **Navigate there**: `cd` to that directory before running any commands
3. **Verify**: Run `ls -d .specify` to confirm `.specify/` is in current directory

```bash
# Find and navigate to speckit root
cd "$(dirname "$(find /workspaces -name '.specify' -type d 2>/dev/null | head -1)")"
ls -d .specify  # Should output: .specify
```

**Why this matters**: All speckit scripts use `git rev-parse --show-toplevel` to determine where to create branches and specs. Running from nested git repos creates specs in wrong locations.

---

### Progress Dashboard

Track workflow progress using this format (update after each phase):

| Phase | Status | Artifacts | Notes |
|-------|--------|-----------|-------|
| Specify | ⏳ | spec.md | Creating feature specification |
| Plan | ⏸ | plan.md | Waiting |
| Tasks | ⏸ | tasks.md | Waiting |
| Implement | ⏸ | code files | Waiting |
| Test | ⏸ | test results | Waiting |

**Status legend**: ✓ Done | ⏳ In Progress | ⏸ Waiting | ✗ Failed

---

### Execution Steps

**Continuous Execution**: After completing each step, IMMEDIATELY invoke the next skill without stopping. Do not show intermediate summaries or wait for confirmation between phases.

#### Phase 1: Specify
- **Invoke**: `speckit.specify` with args: `$ARGUMENTS`
- **Creates**: `spec.md` in feature directory
- **Validation**: Verify spec.md exists and contains `## User Stories` section

#### Phase 2: Plan
- **Invoke**: `speckit.plan` (no args)
- **Creates**: `plan.md` with technical implementation approach
- **Validation**: Verify plan.md exists and contains `## Implementation Approach` section

#### Phase 3: Tasks
- **Invoke**: `speckit.tasks` (no args)
- **Creates**: `tasks.md` with actionable task breakdown
- **Validation**: Verify tasks.md exists and contains task entries

#### Phase 4: Implement
- **Invoke**: `speckit.implement` (no args)
- **Creates**: Code files per tasks.md
- **⚠️ MANDATORY**: After implement completes, you MUST immediately invoke speckit.test

#### Phase 5: Test
- **Invoke**: `speckit.test` (no args)
- **Verifies**: Build, tests, lint, format checks pass
- **⚠️ THIS STEP IS REQUIRED** - the workflow is NOT complete until test runs

---

### Artifact Validation

After each phase, validate before proceeding:

```
Phase N completes →
  Check artifact exists and isn't empty →
  Check for required sections →
  If valid: proceed to Phase N+1
  If invalid: report error and stop
```

**Required artifacts**:
- `spec.md`: Must have User Stories section
- `plan.md`: Must have Implementation Approach section
- `tasks.md`: Must have task entries (checkbox items)

---

### Error Handling

If a phase fails:

1. **Stop immediately** - do not proceed to next phase
2. **Report the failure** with specific error details
3. **Suggest remediation**:
   - For specify failures: Check feature description clarity
   - For plan failures: Review spec.md for completeness
   - For tasks failures: Review plan.md structure
   - For implement failures: Check tasks.md and codebase patterns
   - For test failures: Run `/speckit.test fix` to auto-fix lint/format issues

---

### Completion Summary

Only after ALL phases complete successfully, provide a summary:

```markdown
## Workflow Complete ✓

| Phase | Status | Duration |
|-------|--------|----------|
| Specify | ✓ | Xm Xs |
| Plan | ✓ | Xm Xs |
| Tasks | ✓ | Xm Xs |
| Implement | ✓ | Xm Xs |
| Test | ✓ | Xm Xs |

**Feature**: [feature name]
**Branch**: [branch name]
**Artifacts created**: spec.md, plan.md, tasks.md, [code files]

### Next Steps
- Review generated code
- Run additional manual testing if needed
- Create PR when ready
```

---

### Rules

- Use the Skill tool for each invocation
- Run all phases automatically without pausing
- **CRITICAL - TEST IS MANDATORY**: Never end the workflow without running test
- **CRITICAL**: After each phase completes, IMMEDIATELY invoke the next skill
- Only provide a final summary after ALL phases complete successfully
- Only the specify phase receives arguments (the user's input)
- Subsequent phases use artifacts from prior phases
- Stop and report if any phase fails
- **CRITICAL**: Always run Pre-Flight Checks before Phase 1
- Never run speckit commands from inside nested git repositories
