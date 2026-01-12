---
description: Auto-clarify underspecified areas in the spec by automatically accepting recommended options without user interaction.
handoffs:
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create a plan for the spec. I am building with...
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

Goal: Detect and reduce ambiguity or missing decision points in the active feature specification by **automatically accepting recommended options** and recording the clarifications directly in the spec file.

**AUTO MODE**: This command does NOT wait for user input. It automatically selects the best option for each clarification question based on best practices, common patterns, and risk reduction. Use `/speckit.clarify` if you want interactive control over each decision.

Note: This clarification workflow is expected to run (and be completed) BEFORE invoking `/speckit.plan`.

Execution steps:

1. Run `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` from repo root **once** (combined `--json --paths-only` mode / `-Json -PathsOnly`). Parse minimal JSON payload fields:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - (Optionally capture `IMPL_PLAN`, `TASKS` for future chained flows.)
   - If JSON parsing fails, abort and instruct user to re-run `/speckit.specify` or verify feature branch environment.
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. Load the current spec file. Perform a structured ambiguity & coverage scan using this taxonomy. For each category, mark status: Clear / Partial / Missing. Produce an internal coverage map used for prioritization (do not output raw map unless no questions will be asked).

   Functional Scope & Behavior:
   - Core user goals & success criteria
   - Explicit out-of-scope declarations
   - User roles / personas differentiation

   Domain & Data Model:
   - Entities, attributes, relationships
   - Identity & uniqueness rules
   - Lifecycle/state transitions
   - Data volume / scale assumptions

   Interaction & UX Flow:
   - Critical user journeys / sequences
   - Error/empty/loading states
   - Accessibility or localization notes

   Non-Functional Quality Attributes:
   - Performance (latency, throughput targets)
   - Scalability (horizontal/vertical, limits)
   - Reliability & availability (uptime, recovery expectations)
   - Observability (logging, metrics, tracing signals)
   - Security & privacy (authN/Z, data protection, threat assumptions)
   - Compliance / regulatory constraints (if any)

   Integration & External Dependencies:
   - External services/APIs and failure modes
   - Data import/export formats
   - Protocol/versioning assumptions

   Edge Cases & Failure Handling:
   - Negative scenarios
   - Rate limiting / throttling
   - Conflict resolution (e.g., concurrent edits)

   Constraints & Tradeoffs:
   - Technical constraints (language, storage, hosting)
   - Explicit tradeoffs or rejected alternatives

   Terminology & Consistency:
   - Canonical glossary terms
   - Avoided synonyms / deprecated terms

   Completion Signals:
   - Acceptance criteria testability
   - Measurable Definition of Done style indicators

   Misc / Placeholders:
   - TODO markers / unresolved decisions
   - Ambiguous adjectives ("robust", "intuitive") lacking quantification

   For each category with Partial or Missing status, add a candidate question opportunity unless:
   - Clarification would not materially change implementation or validation strategy
   - Information is better deferred to planning phase (note internally)

3. Generate (internally) a prioritized queue of candidate clarification questions (maximum 5). Apply these constraints:
    - Maximum of 5 total questions for auto mode.
    - Each question must be answerable with EITHER:
       - A short multiple‑choice selection (2–5 distinct, mutually exclusive options), OR
       - A one-word / short‑phrase answer.
    - Only include questions whose answers materially impact architecture, data modeling, task decomposition, test design, UX behavior, operational readiness, or compliance validation.
    - Ensure category coverage balance: attempt to cover the highest impact unresolved categories first.
    - Exclude questions already answered, trivial stylistic preferences, or plan-level execution details.
    - Favor clarifications that reduce downstream rework risk or prevent misaligned acceptance tests.
    - If more than 5 categories remain unresolved, select the top 5 by (Impact * Uncertainty) heuristic.

4. **Auto-accept loop (NON-INTERACTIVE)**:

   Process ALL questions automatically without waiting for user input:

   For each question in the prioritized queue:

   a. **Analyze and determine the recommended option** using the same criteria as interactive mode:
      - Best practices for the project type
      - Common patterns in similar implementations
      - Risk reduction (security, performance, maintainability)
      - Alignment with any explicit project goals or constraints visible in the spec

   b. **Log the auto-selection** in a summary table (build this as you go):
      ```
      | Q# | Question | Auto-Selected | Reasoning |
      |----|----------|---------------|-----------|
      | 1  | [brief question] | [selected option] | [1-sentence reasoning] |
      | 2  | [brief question] | [selected option] | [1-sentence reasoning] |
      ```

   c. **Record the recommendation as the answer** - treat it exactly as if the user had replied "yes" or "recommended"

   d. **Immediately apply to spec** (same as step 5 in interactive mode):
      - Ensure `## Clarifications` section exists
      - Add `### Session YYYY-MM-DD (Auto)` subheading
      - Append: `- Q: <question> → A: <auto-selected answer> [AUTO]`
      - Apply clarification to appropriate spec section

   e. **Proceed to next question** without stopping

   After processing all questions, continue to step 5.

   If no valid questions exist at start, immediately report: "No critical ambiguities detected. Spec is ready for planning."

5. Integration (same as interactive mode but applied automatically in step 4):
    - Maintain in-memory representation of the spec plus raw file contents.
    - For the first integrated answer in this session:
       - Ensure a `## Clarifications` section exists (create it just after the highest-level contextual/overview section per the spec template if missing).
       - Under it, create (if not present) a `### Session YYYY-MM-DD (Auto)` subheading for today.
    - Append a bullet line: `- Q: <question> → A: <final answer> [AUTO]`.
    - Then immediately apply the clarification to the most appropriate section(s):
       - Functional ambiguity → Update or add a bullet in Functional Requirements.
       - User interaction / actor distinction → Update User Stories or Actors subsection with clarified role, constraint, or scenario.
       - Data shape / entities → Update Data Model (add fields, types, relationships) preserving ordering.
       - Non-functional constraint → Add/modify measurable criteria in Non-Functional / Quality Attributes section.
       - Edge case / negative flow → Add a new bullet under Edge Cases / Error Handling.
       - Terminology conflict → Normalize term across spec.
    - If the clarification invalidates an earlier ambiguous statement, replace that statement.
    - Save the spec file AFTER each integration.
    - Preserve formatting: do not reorder unrelated sections; keep heading hierarchy intact.

6. Validation (performed after EACH write plus final pass):
   - Clarifications session contains exactly one bullet per auto-accepted answer (no duplicates).
   - Total questions ≤ 5.
   - Updated sections contain no lingering vague placeholders the answer was meant to resolve.
   - No contradictory earlier statement remains.
   - Markdown structure valid; only allowed new headings: `## Clarifications`, `### Session YYYY-MM-DD (Auto)`.
   - Terminology consistency: same canonical term used across all updated sections.

7. Write the updated spec back to `FEATURE_SPEC`.

8. Report completion:

   Output a summary in this format:

   ```markdown
   ## Auto-Clarification Complete

   **Mode**: Automatic (all recommendations accepted)
   **Questions processed**: N
   **Spec updated**: [path to spec.md]

   ### Auto-Selected Clarifications

   | Q# | Question | Selected | Reasoning |
   |----|----------|----------|-----------|
   | 1  | ... | ... | ... |
   | 2  | ... | ... | ... |

   ### Sections Updated
   - [list of sections touched]

   ### Coverage Summary

   | Category | Status |
   |----------|--------|
   | Functional Scope | Resolved / Clear / Deferred |
   | Domain Model | ... |
   | ... | ... |

   **Next**: Run `/speckit.plan` to create the implementation plan.
   ```

   - Number of questions auto-answered.
   - Path to updated spec.
   - Sections touched (list names).
   - Coverage summary table listing each taxonomy category with Status.
   - Suggested next command: `/speckit.plan`

Behavior rules:

- **NEVER wait for user input** - this is auto mode
- If no meaningful ambiguities found, respond: "No critical ambiguities detected. Spec is ready for planning." and suggest proceeding.
- If spec file missing, instruct user to run `/speckit.specify` first.
- Never exceed 5 total questions.
- Avoid speculative tech stack questions unless the absence blocks functional clarity.
- If all categories are Clear, output a compact coverage summary then suggest advancing.
- Mark all auto-selected answers with `[AUTO]` tag in the spec for traceability.

Context for prioritization: $ARGUMENTS
