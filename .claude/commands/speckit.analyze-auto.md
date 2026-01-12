---
description: Perform a non-destructive cross-artifact consistency and quality analysis across spec.md, plan.md, and tasks.md after task generation.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Identify inconsistencies, duplications, ambiguities, and underspecified items across the three core artifacts (`spec.md`, `plan.md`, `tasks.md`) and **automatically apply recommended fixes** before implementation. This command MUST run only after `/speckit.tasks` has successfully produced a complete `tasks.md`.

## Operating Constraints

**AUTO-FIX MODE**: This command **automatically applies fixes** for all detected issues. It does NOT wait for user approval. Use `/speckit.analyze` if you want read-only analysis with manual remediation control.

**Constitution Authority**: The project constitution (`.specify/memory/constitution.md`) is **non-negotiable**. Constitution conflicts are automatically CRITICAL and will be fixed by adjusting the spec, plan, or tasks to align with constitution principles.

## Execution Steps

### 1. Initialize Analysis Context

Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` once from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS. Derive absolute paths:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- TASKS = FEATURE_DIR/tasks.md

Abort with an error message if any required file is missing (instruct the user to run missing prerequisite command).
For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

### 2. Load Artifacts (Progressive Disclosure)

Load only the minimal necessary context from each artifact:

**From spec.md:**

- Overview/Context
- Functional Requirements
- Non-Functional Requirements
- User Stories
- Edge Cases (if present)

**From plan.md:**

- Architecture/stack choices
- Data Model references
- Phases
- Technical constraints

**From tasks.md:**

- Task IDs
- Descriptions
- Phase grouping
- Parallel markers [P]
- Referenced file paths

**From constitution:**

- Load `.specify/memory/constitution.md` for principle validation

### 3. Build Semantic Models

Create internal representations:

- **Requirements inventory**: Each functional + non-functional requirement with a stable key
- **User story/action inventory**: Discrete user actions with acceptance criteria
- **Task coverage mapping**: Map each task to one or more requirements or stories
- **Constitution rule set**: Extract principle names and MUST/SHOULD normative statements

### 4. Detection Passes (Token-Efficient Analysis)

Focus on high-signal findings. Limit to 50 findings total.

#### A. Duplication Detection

- Identify near-duplicate requirements
- Mark lower-quality phrasing for consolidation

#### B. Ambiguity Detection

- Flag vague adjectives (fast, scalable, secure, intuitive, robust) lacking measurable criteria
- Flag unresolved placeholders (TODO, TKTK, ???, `<placeholder>`, etc.)

#### C. Underspecification

- Requirements with verbs but missing object or measurable outcome
- User stories missing acceptance criteria alignment
- Tasks referencing files or components not defined in spec/plan

#### D. Constitution Alignment

- Any requirement or plan element conflicting with a MUST principle
- Missing mandated sections or quality gates from constitution

#### E. Coverage Gaps

- Requirements with zero associated tasks
- Tasks with no mapped requirement/story
- Non-functional requirements not reflected in tasks

#### F. Inconsistency

- Terminology drift (same concept named differently across files)
- Data entities referenced in plan but absent in spec (or vice versa)
- Task ordering contradictions
- Conflicting requirements

### 5. Severity Assignment

- **CRITICAL**: Violates constitution MUST, missing core spec artifact, or requirement with zero coverage that blocks baseline functionality
- **HIGH**: Duplicate or conflicting requirement, ambiguous security/performance attribute, untestable acceptance criterion
- **MEDIUM**: Terminology drift, missing non-functional task coverage, underspecified edge case
- **LOW**: Style/wording improvements, minor redundancy

### 6. Auto-Apply Fixes

**For each detected issue, automatically apply the recommended fix:**

#### A. Duplication Fixes (spec.md)
- Merge duplicate requirements into single, clearer version
- Remove redundant entries
- Keep the more specific/measurable phrasing

#### B. Ambiguity Fixes (spec.md, plan.md)
- Replace vague adjectives with measurable criteria:
  - "fast" → "responds within 200ms"
  - "scalable" → "supports 10,000 concurrent users"
  - "secure" → "implements OAuth2 with JWT tokens"
- Resolve placeholders with sensible defaults based on context

#### C. Underspecification Fixes (spec.md, tasks.md)
- Add missing measurable outcomes to requirements
- Add acceptance criteria to user stories
- Add tasks for undefined components

#### D. Constitution Alignment Fixes (spec.md, plan.md, tasks.md)
- Adjust conflicting elements to align with constitution principles
- Add missing mandated sections
- Ensure quality gates are present

#### E. Coverage Gap Fixes (tasks.md)
- Add tasks for uncovered requirements
- Map orphan tasks to relevant requirements
- Add non-functional requirement tasks (performance, security, etc.)

#### F. Inconsistency Fixes (all files)
- Normalize terminology to canonical terms
- Add missing entity references
- Fix task ordering to respect dependencies
- Resolve conflicting requirements (keep more specific version)

### 7. Apply Fixes and Log Changes

For each fix applied:

1. **Modify the appropriate file** (spec.md, plan.md, or tasks.md)
2. **Log the change** in a running fix log:
   ```
   | Fix# | File | Issue | Fix Applied |
   |------|------|-------|-------------|
   | F1 | spec.md | Duplicate requirement FR-3/FR-7 | Merged into FR-3 |
   | F2 | spec.md | Vague "fast loading" | Changed to "page load < 2s" |
   | F3 | tasks.md | No task for NFR-2 (security) | Added T045 security task |
   ```

3. **Save file after each fix** to prevent data loss

### 8. Produce Summary Report

Output a Markdown report with:

```markdown
## Auto-Analysis & Fix Report

**Mode**: Automatic (fixes applied without confirmation)
**Files Modified**: [list]

### Issues Found and Fixed

| Fix# | Severity | File | Issue | Fix Applied |
|------|----------|------|-------|-------------|
| F1 | HIGH | spec.md | Duplicate requirement | Merged FR-3/FR-7 |
| F2 | MEDIUM | spec.md | Vague term "fast" | Quantified to "< 200ms" |
| F3 | CRITICAL | tasks.md | Missing coverage | Added security task T045 |

### Coverage After Fixes

| Metric | Before | After |
|--------|--------|-------|
| Requirements Coverage | 85% | 100% |
| Ambiguity Count | 5 | 0 |
| Duplication Count | 3 | 0 |
| Constitution Violations | 1 | 0 |

### Files Updated
- spec.md: N changes
- plan.md: N changes
- tasks.md: N changes

**Status**: All issues resolved. Ready for `/speckit.implement`.
```

### 9. Proceed to Next Phase

After all fixes are applied:
- Confirm all CRITICAL issues are resolved
- Output: "Analysis complete. All issues auto-fixed. Proceeding to next phase."
- The workflow continues automatically

## Auto-Fix Guidelines

### What Gets Fixed Automatically

| Issue Type | Auto-Fix Action |
|------------|-----------------|
| Duplicate requirements | Merge into clearer version |
| Vague adjectives | Replace with measurable criteria |
| Missing task coverage | Add appropriate tasks |
| Terminology drift | Normalize to canonical term |
| Constitution violations | Adjust to align with principles |
| Orphan tasks | Map to relevant requirements |
| Missing acceptance criteria | Generate based on requirement |
| Placeholder text | Replace with contextual defaults |

### Fix Priorities

1. **CRITICAL first** - Constitution violations, missing coverage
2. **HIGH second** - Duplicates, conflicts, security/performance ambiguities
3. **MEDIUM third** - Terminology, edge cases
4. **LOW last** - Style improvements

### Safety Rules

- **Never delete requirements** - only merge duplicates
- **Never remove tasks** - only add or reorder
- **Never contradict constitution** - always align with principles
- **Preserve user intent** - fixes should clarify, not change meaning
- **Log everything** - all changes must be traceable

## Operating Principles

### Auto-Fix Mode

- **NEVER wait for user input** - this is fully automatic
- **Fix all issues** - don't leave issues for manual resolution
- **Log all changes** - transparency is critical
- **Proceed when done** - continue to next phase automatically

### Analysis Guidelines

- **Prioritize constitution alignment** (these are always CRITICAL)
- **Use sensible defaults** for ambiguous terms
- **Maintain artifact consistency** across all three files
- **Report zero issues gracefully** (emit success report)

## Context

$ARGUMENTS
