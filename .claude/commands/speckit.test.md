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

Verify that the implementation meets the constitution's Quality Gates:
1. **Build passes** - Code compiles without errors
2. **Tests pass** - All tests succeed
3. **Lint clean** - No linting errors or warnings

This command runs AFTER `/speckit.implement` and uses the Technical Context from `plan.md` to determine the appropriate test commands.

## Execution Modes

| $ARGUMENTS | Behavior |
|------------|----------|
| *(empty)* | Run all checks: build → test → lint → format |
| `quick` | Fast checks only: lint + format (skip build/test) |
| `fix` | Run all checks, then auto-fix fixable issues, then re-verify |
| `test` or `tests` | Only run test command |
| `lint` | Only run lint command |
| `build` | Only run build command |
| `full` | All checks + coverage report if available |

## Execution Steps

### 1. Load Feature Context

Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse JSON for FEATURE_DIR. All paths must be absolute.

For single quotes in args like "I'm Groot", use escape syntax: e.g `'I'\''m Groot'` (or double-quote if possible: `"I'm Groot"`).

If the script fails or plan.md is missing, abort with:
```
ERROR: No plan.md found. Run /speckit.plan first to create the implementation plan.
```

### 2. Parse Technical Context

Read `FEATURE_DIR/plan.md` and extract the Technical Context section:

```markdown
## Technical Context

**Language/Version**: [Extract this - e.g., "Rust 1.75", "Python 3.11"]
**Testing**: [Extract this - e.g., "cargo test", "pytest", "npm test"]
**Primary Dependencies**: [Extract for context]
```

Store these values:
- `LANGUAGE` = extracted Language/Version (normalize to lowercase base language: "rust", "python", "typescript", "go", etc.)
- `TEST_CMD` = extracted Testing value (USE THIS DIRECTLY for test execution)

### 3. Build Check Matrix

Based on `LANGUAGE`, determine the full command set:

**Rust**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `cargo build --all-targets` | N/A |
| Test | `{TEST_CMD}` (from plan.md) | N/A |
| Lint | `cargo clippy --all-targets -- -D warnings` | `cargo clippy --fix --allow-dirty --allow-staged` |
| Format | `cargo fmt --check` | `cargo fmt` |

**Python**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | N/A | N/A |
| Test | `{TEST_CMD}` (from plan.md) | N/A |
| Lint | `ruff check .` | `ruff check --fix .` |
| Format | `ruff format --check .` | `ruff format .` |

**TypeScript/JavaScript/Node.js**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `npm run build` (if script exists) | N/A |
| Test | `{TEST_CMD}` (from plan.md) | N/A |
| Lint | `npm run lint` (if script exists) | `npm run lint -- --fix` |
| Format | `npx prettier --check .` | `npx prettier --write .` |

**Go**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `go build ./...` | N/A |
| Test | `{TEST_CMD}` (from plan.md) | N/A |
| Lint | `golangci-lint run` | `golangci-lint run --fix` |
| Format | `gofmt -l .` | `gofmt -w .` |

**Java**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `mvn compile` | N/A |
| Test | `{TEST_CMD}` (from plan.md) | N/A |
| Lint | `mvn checkstyle:check` | N/A |
| Format | N/A | N/A |

**Swift**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `swift build` | N/A |
| Test | `{TEST_CMD}` (from plan.md) | N/A |
| Lint | `swiftlint` | `swiftlint --fix` |
| Format | `swift-format lint -r .` | `swift-format -i -r .` |

**C/C++**:
| Check | Command | Fix Command |
|-------|---------|-------------|
| Build | `make` or `cmake --build .` | N/A |
| Test | `{TEST_CMD}` (from plan.md) | N/A |
| Lint | `clang-tidy` | N/A |
| Format | `clang-format --dry-run` | `clang-format -i` |

### 4. Apply Execution Mode

Based on `$ARGUMENTS`, filter the check matrix:

- **empty**: Run all checks (build → test → lint → format)
- **quick**: Only lint + format
- **fix**: Run all checks, then run fix commands for failed lint/format, then re-run checks
- **test/tests**: Only test
- **lint**: Only lint
- **build**: Only build
- **full**: All checks + add coverage flags (e.g., `cargo test` → `cargo tarpaulin`, `pytest` → `pytest --cov`)

### 5. Pre-flight Checks

Before executing, verify tools are available:

```bash
# For Rust
command -v cargo > /dev/null 2>&1 || echo "⚠ cargo not found"
command -v rustfmt > /dev/null 2>&1 || echo "⚠ rustfmt not found. Install with: rustup component add rustfmt"
cargo clippy --version > /dev/null 2>&1 || echo "⚠ clippy not installed. Install with: rustup component add clippy"

# For Node.js
command -v npm > /dev/null 2>&1 || echo "⚠ npm not found"
[ -f package.json ] && jq -e '.scripts.test' package.json > /dev/null 2>&1 || echo "ℹ No test script in package.json"
[ -f package.json ] && jq -e '.scripts.lint' package.json > /dev/null 2>&1 || echo "ℹ No lint script in package.json"

# For Python
command -v python > /dev/null 2>&1 || command -v python3 > /dev/null 2>&1 || echo "⚠ python not found"
command -v ruff > /dev/null 2>&1 || echo "ℹ ruff not found, trying flake8"

# For Go
command -v go > /dev/null 2>&1 || echo "⚠ go not found"
command -v golangci-lint > /dev/null 2>&1 || echo "ℹ golangci-lint not found (optional)"
```

Skip checks gracefully if the tool is not available (don't fail entirely).

### 6. Execute Checks

Run each check in order, capturing:
- Exit code (0 = pass, non-zero = fail)
- stdout/stderr output
- Execution time

**Execution order**: build → test → lint → format

For each check:
1. Print: `Running {check}...`
2. Execute command
3. Record result (pass/fail, time, output)
4. Continue to next check (don't stop on failure)

### 7. Report Results

#### If ALL checks pass:

```markdown
## ✓ All Quality Checks Passed

**Stack**: {LANGUAGE} (from plan.md)

| Check  | Status | Time  |
|--------|--------|-------|
| Build  | ✓ PASS | X.Xs  |
| Test   | ✓ PASS | X.Xs  |
| Lint   | ✓ PASS | X.Xs  |
| Format | ✓ PASS | X.Xs  |

**Constitution Quality Gates**: All passed ✓
**Ready for submission** ✓
```

#### If ANY checks fail:

```markdown
## ✗ Quality Checks Failed

**Stack**: {LANGUAGE} (from plan.md)

| Check  | Status | Time  | Issues |
|--------|--------|-------|--------|
| Build  | ✓ PASS | X.Xs  | -      |
| Test   | ✗ FAIL | X.Xs  | N failures |
| Lint   | ✗ FAIL | X.Xs  | N issues |
| Format | ✓ PASS | X.Xs  | -      |

### Failed: Test

<details>
<summary>View test output</summary>

```
[Include relevant test failure output here]
```

</details>

### Failed: Lint

<details>
<summary>View lint output</summary>

```
[Include lint errors/warnings here]
```

</details>

### Summary

- **Checks**: N | **Passed**: N | **Failed**: N
- **Auto-fixable**: N (lint/format issues)

### Next Steps

1. **Auto-fix available**: Run `/speckit.test fix` to automatically fix lint/format issues
2. **Manual fixes needed**:
   - [List specific files:lines that need manual fixes]
   - [Provide brief guidance on what to fix]

Or use handoff: `/speckit.implement Fix the failing tests...`
```

### 8. Handle Fix Mode

If `$ARGUMENTS` is `fix`:

1. Run all checks first (as above)
2. For each failed lint/format check, run the corresponding fix command
3. After fixes applied, re-run ALL checks
4. Report final results with note: "Auto-fixes applied. Re-ran verification."

### 9. Handle Edge Cases

| Scenario | Behavior |
|----------|----------|
| plan.md missing | ERROR: "Run /speckit.plan first" |
| Language not recognized | Ask: "Could not detect language from plan.md. Which stack? (rust/python/node/go)" |
| Test command empty | WARN: "No test command in plan.md. Skipping tests." |
| Tool not installed | SKIP that check, note in output: "⚠ Skipped: {tool} not found" |
| Command times out (>5min) | Kill process, report: "⚠ {check} timed out after 5 minutes" |
| No tests found | INFO: "ℹ No tests found. Consider adding tests." |

## Behavior Rules

1. **Always read from plan.md** - Never guess the stack; use what the user specified
2. **Use TEST_CMD directly** - Don't override the test command from plan.md
3. **Continue on failure** - Run all checks even if some fail (collect full picture)
4. **Actionable output** - Every failure should include how to fix it
5. **Respect constitution** - Frame results in terms of Quality Gates
6. **Be helpful on fix mode** - Auto-fix what can be fixed, clearly report what can't

## Context

This command is the **final step** in the speckit workflow:

```
/speckit.specify → /speckit.clarify → /speckit.plan → /speckit.tasks → /speckit.implement → /speckit.test
```

After `/speckit.test` passes, the code is ready for PR submission.
