---
description: Run tests, linting, and quality checks to verify code meets constitution Quality Gates.
handoffs:
  - label: Fix Issues
    agent: speckit.implement
    prompt: "Fix the failing checks identified in the test report"
  - label: Analyze Quality
    agent: speckit.analyze
    prompt: "Analyze the codebase for consistency issues"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Comprehensive QA validation that acts as a "software tester" to verify:
1. **Code Quality** - Build passes, tests pass, lint clean
2. **Specification Compliance** - Code implements spec.md requirements
3. **Acceptance Validation** - All acceptance scenarios have passing tests
4. **Requirement Traceability** - Clear mapping from requirements to code to tests

This command runs AFTER `/speckit.implement` and validates the implementation is complete and correct.

---

## Execution Modes

| $ARGUMENTS | Behavior |
|------------|----------|
| *(empty)* | Full validation: code quality + specification compliance |
| `quick` | Fast checks only: lint + format (skip build/test/spec) |
| `fix` | Run all checks, auto-fix issues, re-invoke implement if needed, re-verify |
| `test` or `tests` | Only run test command |
| `lint` | Only run lint command |
| `build` | Only run build command |
| `trace` | Generate requirement traceability matrix only |
| `spec` | Validate code against spec.md only (skip build/lint) |
| `coverage` | Run tests with coverage, map to user stories |
| `full` | All checks + coverage report + detailed traceability |

---

## Execution Phases

### Phase 1: Load Feature Context

Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse JSON for:
- `FEATURE_DIR` - Feature directory path
- `FEATURE_SPEC` - Path to spec.md
- `IMPL_PLAN` - Path to plan.md
- `TASKS` - Path to tasks.md

If the script fails or plan.md is missing, abort with:
```
ERROR: No plan.md found. Run /speckit.plan first to create the implementation plan.
```

### Phase 2: Parse Technical Context

Read `FEATURE_DIR/plan.md` and extract:
- `LANGUAGE` - Language/Version (normalize to: rust, python, typescript, go, etc.)
- `TEST_CMD` - Testing command from plan.md

### Phase 3: Build Check Matrix

Based on `LANGUAGE`, determine commands:

**Rust**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `cargo build --all-targets` | N/A |
| Test | `{TEST_CMD}` | N/A |
| Lint | `cargo clippy --all-targets -- -D warnings` | `cargo clippy --fix --allow-dirty --allow-staged` |
| Format | `cargo fmt --check` | `cargo fmt` |
| Coverage | `cargo tarpaulin --out Json` | N/A |

**Python**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | N/A | N/A |
| Test | `{TEST_CMD}` | N/A |
| Lint | `ruff check .` | `ruff check --fix .` |
| Format | `ruff format --check .` | `ruff format .` |
| Coverage | `pytest --cov --cov-report=json` | N/A |

**TypeScript/JavaScript/Node.js**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `npm run build` | N/A |
| Test | `{TEST_CMD}` | N/A |
| Lint | `npm run lint` | `npm run lint -- --fix` |
| Format | `npx prettier --check .` | `npx prettier --write .` |
| Coverage | `npm test -- --coverage` | N/A |

**Go**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `go build ./...` | N/A |
| Test | `{TEST_CMD}` | N/A |
| Lint | `golangci-lint run` | `golangci-lint run --fix` |
| Format | `gofmt -l .` | `gofmt -w .` |
| Coverage | `go test -coverprofile=coverage.out ./...` | N/A |

---

## Phase 4: Code Quality Checks

Run each check in order: **build → test → lint → format**

For each check:
1. Print: `Running {check}...`
2. Execute command
3. Record: exit code, output, execution time
4. Continue to next check (don't stop on failure)

---

## Phase 5: Specification Compliance (NEW)

### 5.1 Parse Specification Artifacts

Read `FEATURE_DIR/spec.md` and extract:

1. **User Stories**: Match pattern `### User Story N - [Title] (Priority: PX)`
   - Extract: story ID, title, priority

2. **Acceptance Scenarios**: Match pattern `**Given** [X], **When** [Y], **Then** [Z]`
   - Extract: precondition, action, expected outcome
   - Associate with parent user story

3. **Functional Requirements**: Match pattern `**FR-XXX**:`
   - Extract: requirement ID, description

4. **Success Criteria**: Match pattern `**SC-XXX**:`
   - Extract: criteria ID, measurable outcome

### 5.2 Parse Task Completion

Read `FEATURE_DIR/tasks.md` and extract:

1. **Completed Tasks**: Match pattern `[X]` or `[x]`
   - Extract: task description, file paths mentioned
   - Extract: user story tags like `[US1]`, `[US2]`

2. **Pending Tasks**: Match pattern `[ ]`
   - Flag as incomplete implementation

### 5.3 Generate Requirement Traceability Matrix

Map the chain: **User Story → Tasks → Code Files → Test Files → Test Results**

```
For each User Story:
  1. Find tasks tagged with [USn]
  2. Extract file paths from task descriptions
  3. Find corresponding test files (*_test.*, test_*.*, *.spec.*)
  4. Check if tests pass for those files
  5. Report traceability status
```

Output format:
```markdown
### Requirement Traceability Matrix

| User Story | Tasks | Code Files | Test Files | Test Status |
|------------|-------|------------|------------|-------------|
| US1: Create Account | T001, T002 | src/auth.py | tests/test_auth.py | ✓ 4/4 PASS |
| US2: Password Reset | T003 | src/reset.py | tests/test_reset.py | ⚠ 2/3 PASS |
| US3: Profile Edit | T004 | - | - | ✗ NOT IMPLEMENTED |
```

---

## Phase 6: Acceptance Validation (NEW)

For each acceptance scenario in spec.md:

1. **Parse scenario**: Given [precondition], When [action], Then [expected]

2. **Search for matching test**:
   - Look for test functions containing keywords from the scenario
   - Match test assertions to the "Then" clause
   - Check if test passed in Phase 4 results

3. **Report coverage**:
   - ✓ **Covered**: Test exists and passes
   - ⚠ **Partial**: Test exists but fails OR only partial assertion match
   - ✗ **Missing**: No test found for this scenario

Output format:
```markdown
### Acceptance Scenario Validation

| # | Scenario (Given/When/Then) | Test | Status |
|---|---------------------------|------|--------|
| 1 | Given user on signup, When enters valid email, Then account created | test_signup_success | ✓ PASS |
| 2 | Given duplicate email, When submits form, Then error shown | test_duplicate_error | ✓ PASS |
| 3 | Given expired token, When resets password, Then error shown | - | ✗ NO TEST |
```

---

## Phase 7: Coverage Analysis (for `coverage` and `full` modes)

1. Run coverage tool for the language
2. Parse coverage report (JSON format preferred)
3. Map coverage to user stories via file paths from tasks.md

Output format:
```markdown
### Coverage by User Story

| User Story | Priority | Files | Coverage | Target | Status |
|------------|----------|-------|----------|--------|--------|
| US1: Create Account | P1 | src/auth.py, src/models/user.py | 92% | 85% | ✓ |
| US2: Password Reset | P2 | src/reset.py | 78% | 75% | ✓ |
| US3: Profile Edit | P3 | - | 0% | 60% | ✗ |

**Coverage Targets**: P1 ≥85%, P2 ≥75%, P3 ≥60%
```

---

## Phase 8: Generate Report

### If ALL checks pass:

```markdown
## ✓ Quality Validation Passed

**Stack**: {LANGUAGE} | **Feature**: {FEATURE_NAME}

### Code Quality
| Check  | Status | Time  |
|--------|--------|-------|
| Build  | ✓ PASS | X.Xs  |
| Test   | ✓ PASS | X.Xs  |
| Lint   | ✓ PASS | X.Xs  |
| Format | ✓ PASS | X.Xs  |

### Specification Compliance
| Metric | Value |
|--------|-------|
| User Stories Implemented | X/Y (Z%) |
| Acceptance Scenarios Covered | X/Y (Z%) |
| Requirements Traced | X/Y (Z%) |
| Scope Drift | None detected |

### Summary
✓ **Constitution Quality Gates**: All passed
✓ **Specification Compliance**: 100%
✓ **Ready for submission**
```

### If ANY checks fail:

```markdown
## ✗ Quality Validation Failed

**Stack**: {LANGUAGE} | **Feature**: {FEATURE_NAME}

### Code Quality
| Check  | Status | Time  | Issues |
|--------|--------|-------|--------|
| Build  | ✓ PASS | X.Xs  | -      |
| Test   | ✗ FAIL | X.Xs  | N failures |
| Lint   | ⚠ WARN | X.Xs  | N warnings |
| Format | ✓ PASS | X.Xs  | -      |

### Specification Compliance
| User Story | Status | Tests | Issues |
|------------|--------|-------|--------|
| US1 | ✓ COMPLETE | 4/4 | - |
| US2 | ⚠ PARTIAL | 2/3 | Missing: expired token test |
| US3 | ✗ MISSING | 0/2 | Not implemented |

### Failed Checks Detail

<details>
<summary>Test Failures (click to expand)</summary>

```
[Test failure output here]
```
</details>

### Auto-Fixable Issues
| Issue | Fix Command |
|-------|-------------|
| Lint errors (5) | `/speckit.test fix` |
| Format issues (3) | `/speckit.test fix` |
| Missing test for US2.3 | `/speckit.test fix` (generates stub) |

### Manual Fixes Required
| Issue | Action |
|-------|--------|
| US3 not implemented | Run `/speckit.implement` |
| Test assertion wrong | Fix test logic in test_reset.py:45 |

### Summary
- **Code Quality**: 3/4 passed
- **Specification Compliance**: 67%
- **Blocking Issues**: 2
```

---

## Phase 9: Auto-Remediation (fix mode)

When `$ARGUMENTS` is `fix`:

### Level 1: Code Quality Fixes (Auto-apply)

```
1. Run lint fix command
2. Run format fix command
3. Re-run lint/format checks to verify
```

### Level 2: Specification Fixes (Generate + Apply)

```
For each missing acceptance scenario test:
  1. Generate test stub based on Given/When/Then
  2. Add to appropriate test file
  3. Mark as TODO for manual implementation

Template:
def test_{scenario_keywords}():
    """
    Acceptance Scenario: {scenario}
    Given: {given}
    When: {when}
    Then: {then}
    """
    # TODO: Implement this acceptance test
    raise NotImplementedError("Generated stub - implement this test")
```

### Level 3: Implementation Fixes (Handoff)

```
If implementation gaps detected (incomplete user stories):
  1. Generate specific fix instructions
  2. Invoke speckit.implement with:
     "Fix the following implementation gaps:
      - US2: Missing test for 'expired token' scenario
      - US3: Not implemented - see spec.md for requirements"
  3. After implement completes, re-run all checks
```

### Final Re-verification

After all fixes applied:
1. Re-run ALL checks (Phase 4-7)
2. Report: "Auto-fixes applied. Results:"
3. Show updated report

---

## Behavior Rules

1. **Always validate against spec.md** - Code must implement what's specified
2. **Use TEST_CMD from plan.md** - Don't override user's test command
3. **Continue on failure** - Run all checks to collect full picture
4. **Actionable output** - Every failure includes how to fix it
5. **Respect constitution** - Frame results in terms of Quality Gates
6. **Auto-fix aggressively** - In fix mode, fix everything possible
7. **Handoff implementation gaps** - Use speckit.implement for code fixes
8. **Re-verify after fixes** - Always confirm fixes worked

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| plan.md missing | ERROR: "Run /speckit.plan first" |
| spec.md missing | WARN: "No spec.md found. Running code quality checks only." |
| tasks.md missing | WARN: "No tasks.md found. Cannot trace requirements." |
| Language not recognized | Ask: "Could not detect language. Which stack? (rust/python/node/go)" |
| No tests found | WARN: "No tests found. Acceptance scenarios cannot be validated." |
| Tool not installed | SKIP that check, note: "⚠ Skipped: {tool} not found" |
| Command timeout (>5min) | Kill, report: "⚠ {check} timed out" |

---

## Context

This command is the **final validation step** in the speckit workflow:

```
/speckit.specify → /speckit.plan → /speckit.tasks → /speckit.implement → /speckit.test
```

**Role**: speckit.test is the "software tester" that validates speckit.implement's "developer" work.

After `/speckit.test` passes with 100% specification compliance, the code is ready for PR submission.
