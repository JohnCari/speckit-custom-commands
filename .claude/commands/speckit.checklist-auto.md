---
description: Auto-generate checklists for all relevant domains by detecting context from spec/plan and using recommended defaults.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Automatically detect relevant domains from the feature spec and plan, then generate appropriate requirement quality checklists **without user interaction**. This command uses recommended defaults for all decisions and validates requirements against the generated checklists.

**AUTO MODE**: This command does NOT ask clarifying questions. It automatically detects domains, selects appropriate depth/focus, and generates all relevant checklists. Use `/speckit.checklist` if you want interactive control over checklist generation.

## Checklist Purpose: "Unit Tests for English"

**CRITICAL CONCEPT**: Checklists are **UNIT TESTS FOR REQUIREMENTS WRITING** - they validate the quality, clarity, and completeness of requirements in a given domain.

**NOT for verification/testing**:
- ❌ NOT "Verify the button clicks correctly"
- ❌ NOT "Test error handling works"

**FOR requirements quality validation**:
- ✅ "Are visual hierarchy requirements defined with measurable criteria?"
- ✅ "Is 'fast loading' quantified with specific thresholds?"

## Execution Steps

### 1. Setup

Run `.specify/scripts/bash/check-prerequisites.sh --json` from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS list. All file paths must be absolute.

For single quotes in args like "I'm Groot", use escape syntax: e.g `'I'\''m Groot'` (or double-quote if possible: `"I'm Groot"`).

### 2. Load Feature Context

Read from FEATURE_DIR:
- **spec.md**: Feature requirements and scope (REQUIRED)
- **plan.md**: Technical details, dependencies (if exists)
- **tasks.md**: Implementation tasks (if exists)

### 3. Auto-Detect Relevant Domains

Scan spec.md and plan.md to detect which domains are relevant:

| Domain | Detection Signals | Checklist File |
|--------|-------------------|----------------|
| **UX** | UI, interface, layout, visual, display, screen, page, component, button, form, navigation | `ux.md` |
| **API** | endpoint, REST, GraphQL, request, response, HTTP, route, controller, service | `api.md` |
| **Security** | auth, authentication, authorization, password, token, JWT, OAuth, permission, role, encryption, sensitive | `security.md` |
| **Performance** | fast, slow, latency, throughput, load, scale, concurrent, cache, optimize | `performance.md` |
| **Data** | entity, model, schema, database, field, relationship, CRUD, store, persist | `data.md` |
| **Integration** | external, API, third-party, webhook, sync, import, export, connect | `integration.md` |
| **Accessibility** | a11y, accessible, screen reader, keyboard, WCAG, ARIA | `accessibility.md` |
| **Error Handling** | error, exception, failure, fallback, recovery, retry, timeout | `errors.md` |

**Detection Algorithm**:
1. Scan spec.md for domain keywords (case-insensitive)
2. Count occurrences per domain
3. Generate checklist for domains with ≥3 signal matches
4. Always include `requirements.md` (general quality) if spec exists

### 4. Apply Recommended Defaults

Use these defaults for ALL checklist generation (no user input):

| Setting | Default Value | Reasoning |
|---------|---------------|-----------||
| **Depth** | Standard | Balanced coverage without exhaustive detail |
| **Audience** | Reviewer (PR) | Most common use case |
| **Focus** | All detected domains | Maximum coverage |
| **Risk Priority** | Security > Performance > Data > UX | Industry standard |
| **Item Count** | 10-20 per checklist | Manageable but thorough |

### 5. Generate Checklists

For each detected domain, generate a checklist file:

1. **Create directory**: `FEATURE_DIR/checklists/` if it doesn't exist

2. **Generate checklist file** for each domain:
   - Filename: `[domain].md`
   - Follow template structure from `.specify/templates/checklist-template.md`
   - Number items sequentially (CHK001, CHK002, etc.)

3. **Checklist content rules** (same as interactive version):

   **CORE PRINCIPLE - Test the Requirements, Not the Implementation**:
   - **Completeness**: Are all necessary requirements present?
   - **Clarity**: Are requirements unambiguous and specific?
   - **Consistency**: Do requirements align with each other?
   - **Measurability**: Can requirements be objectively verified?
   - **Coverage**: Are all scenarios/edge cases addressed?

   **REQUIRED PATTERNS**:
   - ✅ "Are [requirement type] defined/specified/documented for [scenario]?"
   - ✅ "Is [vague term] quantified/clarified with specific criteria?"
   - ✅ "Are requirements consistent between [section A] and [section B]?"

   **PROHIBITED PATTERNS**:
   - ❌ "Verify", "Test", "Confirm", "Check" + implementation behavior
   - ❌ References to code execution or system behavior

### 6. Auto-Validate Requirements

For each generated checklist item:

1. **Scan spec.md** for evidence that the requirement is addressed
2. **Mark item status**:
   - `[x]` - Requirement is clearly defined in spec
   - `[ ]` - Requirement is missing or unclear
3. **Add validation note** after each item:
   - If passed: `[AUTO-VALIDATED: Found in Spec §X.Y]`
   - If failed: `[AUTO-FLAGGED: Not found in spec]`

### 6.5 Auto-Resolve Flagged Items

**For each flagged item**, attempt automatic resolution:

1. **Analyze the gap**: Determine what's missing from spec.md
2. **Apply sensible defaults** if the gap can be filled automatically
3. **Update spec.md** with the auto-generated content
4. **Mark item as resolved**: `[x]` with note `[AUTO-RESOLVED: Added to Spec]`
5. **If cannot resolve**: Keep flagged with note `[UNRESOLVED: Requires manual review]`

**Auto-Resolution Rules**:

| Gap Type | Can Auto-Resolve? | Default Resolution |
|----------|-------------------|-------------------|
| Missing logging requirements | ✅ Yes | Add "Security-relevant events (auth failures, permission denials) SHOULD be logged with timestamp and context" |
| Missing validation behavior | ✅ Yes | Add "Input validation follows framework defaults; invalid input returns descriptive error" |
| Unquantified performance term | ✅ Yes | Replace with industry default (e.g., "fast" → "< 200ms", "scalable" → "supports 1000 concurrent users") |
| Missing edge case | ✅ Yes | Add edge case with graceful degradation behavior |
| Missing dual-failure scenario | ✅ Yes | Add "Simultaneous failures trigger graceful shutdown with state preservation" |
| Complex security requirements | ❌ No | Flag for manual review - security needs human judgment |
| Domain-specific business logic | ❌ No | Flag for manual review - requires domain expertise |
| Compliance/regulatory items | ❌ No | Flag for manual review - legal implications |

**After auto-resolution**:
- Re-validate the updated spec against checklists
- Update checklist files with new pass/flag status

### 7. Generate Summary Report

After all checklists are generated, output:

```markdown
## Auto-Checklist Generation Complete

**Mode**: Automatic (recommended defaults applied)
**Defaults Used**:
- Depth: Standard
- Audience: Reviewer (PR)
- Risk Priority: Security > Performance > Data > UX

### Domains Detected & Checklists Generated

| Domain | Signals Found | Checklist | Items | Passed | Failed |
|--------|---------------|-----------|-------|--------|--------|
| Security | 8 | security.md | 15 | 12 | 3 |
| API | 12 | api.md | 18 | 15 | 3 |
| Data | 6 | data.md | 12 | 10 | 2 |
| UX | 4 | ux.md | 10 | 8 | 2 |

### Checklists Created
- `FEATURE_DIR/checklists/requirements.md` (general quality)
- `FEATURE_DIR/checklists/security.md`
- `FEATURE_DIR/checklists/api.md`
- `FEATURE_DIR/checklists/data.md`
- `FEATURE_DIR/checklists/ux.md`

### Validation Summary
- **Total Items**: N
- **Auto-Passed**: N (requirements clearly defined)
- **Auto-Resolved**: N (gaps filled with sensible defaults)
- **Remaining Flagged**: N (require manual review)

### Auto-Resolved Items

| Checklist | Item | Resolution Applied |
|-----------|------|-------------------|
| security.md | CHK007 | Added logging requirements to NFR section |
| errors.md | CHK014 | Added dual-failure edge case |

### Remaining Flagged Items (Manual Review Needed)

| Checklist | Item | Why Not Auto-Resolved |
|-----------|------|----------------------|
| security.md | CHK009 | TLS validation requires security review |

**Next**: Proceeding to `/speckit.analyze-auto`
```

### 8. Proceed to Next Phase

After validation and auto-resolution:
- Output completion summary with resolution stats
- **Always proceed** to next phase (never halt)
- Remaining flagged items are informational for future improvement

**Output format**:
```markdown
## Checklist Validation Complete

**Results**: N total | N passed | N auto-resolved | N flagged

[If items were auto-resolved]
✓ Auto-resolved N gaps in spec.md

[If items remain flagged]
ℹ N items flagged for future review (not blocking)

Proceeding to `/speckit.analyze-auto`
```

## Domain-Specific Checklist Templates

### requirements.md (Always Generated)

General requirements quality checklist:

```markdown
# Requirements Quality Checklist: [FEATURE NAME]

**Purpose**: Validate overall specification quality
**Created**: [DATE] (Auto-generated)
**Mode**: Automatic

## Completeness

- [ ] CHK001 - Are all user stories defined with clear acceptance criteria? [Completeness]
- [ ] CHK002 - Are functional requirements testable and unambiguous? [Completeness]
- [ ] CHK003 - Are non-functional requirements specified with measurable targets? [Completeness]
- [ ] CHK004 - Are edge cases and error scenarios documented? [Coverage]

## Clarity

- [ ] CHK005 - Are vague terms ("fast", "secure", "scalable") quantified? [Clarity]
- [ ] CHK006 - Are ambiguous requirements clarified with examples? [Clarity]
- [ ] CHK007 - Is terminology consistent throughout the spec? [Consistency]

## Traceability

- [ ] CHK008 - Do all requirements have unique identifiers? [Traceability]
- [ ] CHK009 - Are dependencies between requirements documented? [Traceability]
- [ ] CHK010 - Are assumptions explicitly stated? [Assumptions]
```

### security.md (If Security Signals Detected)

```markdown
# Security Requirements Checklist: [FEATURE NAME]

**Purpose**: Validate security requirement quality
**Created**: [DATE] (Auto-generated)

## Authentication

- [ ] CHK001 - Are authentication requirements specified for all protected resources? [Coverage]
- [ ] CHK002 - Is session management behavior defined (timeout, renewal)? [Completeness]
- [ ] CHK003 - Are credential storage requirements specified? [Security, Gap]

## Authorization

- [ ] CHK004 - Are role/permission requirements clearly defined? [Clarity]
- [ ] CHK005 - Are access control rules specified for all resources? [Coverage]

## Data Protection

- [ ] CHK006 - Are data encryption requirements specified (at rest, in transit)? [Completeness]
- [ ] CHK007 - Are PII handling requirements documented? [Compliance]
- [ ] CHK008 - Are data retention/deletion requirements defined? [Coverage]
```

### api.md (If API Signals Detected)

```markdown
# API Requirements Checklist: [FEATURE NAME]

**Purpose**: Validate API requirement quality
**Created**: [DATE] (Auto-generated)

## Endpoints

- [ ] CHK001 - Are all endpoints documented with HTTP methods? [Completeness]
- [ ] CHK002 - Are request/response formats specified? [Clarity]
- [ ] CHK003 - Are error response codes and formats defined? [Coverage]

## Contracts

- [ ] CHK004 - Are authentication requirements specified per endpoint? [Security]
- [ ] CHK005 - Are rate limiting requirements quantified? [Clarity]
- [ ] CHK006 - Are versioning requirements documented? [Completeness]

## Integration

- [ ] CHK007 - Are timeout/retry requirements specified? [Coverage]
- [ ] CHK008 - Are external dependency failure modes documented? [Edge Cases]
```

## Behavior Rules

- **NEVER wait for user input** - this is fully automatic
- **Auto-detect ALL relevant domains** - don't limit to one
- **Use recommended defaults** - Standard depth, Reviewer audience
- **Auto-validate against spec** - mark items as passed/failed
- **Generate ALL checklists** - maximum coverage
- **Log everything** - transparency on what was generated
- **AUTO-RESOLVE BY DEFAULT** - Attempt to fix all flagged items with sensible defaults
- **NEVER HALT** - Always proceed to next phase, even with remaining flags
- **TRANSPARENCY** - Clearly report what was auto-resolved vs what remains flagged

## Context

$ARGUMENTS
