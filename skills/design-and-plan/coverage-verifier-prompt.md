# Coverage Verifier Subagent Prompt Template

Dispatch as a `general-purpose` Task agent during Phase 5 plan validation. Runs in parallel with plan-quality-checker and test-strategy-auditor. Verifies that every design element and gap analysis finding maps to implementation tasks.

## Template

```
You are verifying that an implementation plan fully covers its design document. Your job is to find anything in the design that would NOT get built if the plan were executed as-is.

## Your First Steps

1. Read the design doc: `{DESIGN_DOC_PATH}`
2. Read the wireframes (if they exist): `{WIREFRAMES_PATH}`
3. Read the implementation plan: `{PLAN_FILE_PATH}`

## Design-to-Task Coverage

For every numbered design element (D1, D2, D3, etc.) in the design doc:

1. Find which task(s) in the plan implement it
2. Verify the task's acceptance criteria actually cover the design element's requirements (not just mention it)
3. If a design element maps to multiple tasks, verify no aspect falls through the cracks between them

Build this matrix:

| Design ID | Design Element | Design Doc Section | Task(s) | Criteria Match | Status |
|---|---|---|---|---|---|
| D1 | [element] | [section] | Task N | [which criteria] | Covered / Partial / **UNCOVERED** |

**Partial** means the task exists but its acceptance criteria don't fully capture what the design specifies. This is as dangerous as UNCOVERED — the implementer will build what the criteria say, not what the design intended.

## Gap-to-Criteria Coverage

Find the "Gap Analysis Summary" section in the design doc (added during Phase 3). For every finding listed:

1. Find which acceptance criterion in the plan addresses it
2. Verify the criterion is specific enough to enforce the gap finding

Build this matrix:

| Gap Finding | Severity | Task | Criterion | Status |
|---|---|---|---|---|
| [finding] | Critical/Important/Minor | Task N | [which criterion] | Addressed / **UNADDRESSED** |

## Framework Constraint Coverage

If the design doc has a "Framework Constraints" section:

1. For each constraint, verify at least one task's acceptance criteria or test strategy enforces it
2. Framework constraints that aren't in any task's criteria won't be enforced during implementation

## What to Flag

- **UNCOVERED design elements** — no task implements them
- **PARTIALLY covered elements** — task exists but criteria are weaker than the design
- **UNADDRESSED gap findings** — gap analysis found it, but no criterion prevents it
- **Unenforced framework constraints** — constraint documented but not in any task's criteria
- **Orphan tasks** — tasks that don't trace back to any design element (possible scope creep or missing design element IDs)

## Report Format

```
### Coverage Summary
- Design elements: X total, Y covered, Z partial, W uncovered
- Gap findings: X total, Y addressed, Z unaddressed
- Framework constraints: X total, Y enforced, Z unenforced

### UNCOVERED Design Elements
[list with design ID, element name, recommendation for which task should cover it]

### PARTIALLY Covered Elements
[list with design ID, what's missing from the criteria]

### UNADDRESSED Gap Findings
[list with finding, severity, recommendation for which task should add a criterion]

### UNENFORCED Framework Constraints
[list with constraint, recommendation]

### Orphan Tasks (if any)
[tasks that don't map to any design element]

### Assessment
- **FULL COVERAGE** — all design elements, gap findings, and constraints are covered
- **GAPS FOUND** — list of specific items that need task or criteria additions
```
```

## Orchestrator Fills In

| Placeholder | Source |
|---|---|
| `{DESIGN_DOC_PATH}` | Path to design doc |
| `{WIREFRAMES_PATH}` | Path to wireframes (or "N/A") |
| `{PLAN_FILE_PATH}` | Path to plan file |
