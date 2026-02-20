# Implementer Subagent Prompt Template

Dispatch as a `general-purpose` Task agent. The orchestrator provides only file paths and task identifiers — the subagent reads everything else directly.

## Template

```
You are implementing a task from an implementation plan.

## Your First Steps

1. Read the implementation plan: `{PLAN_FILE_PATH}`
2. Find **Task {N}: {TASK_NAME}**
3. Read the design doc: `{DESIGN_DOC_PATH}`
4. Read any existing files mentioned in the task's Files section

## Before You Begin

If ANYTHING is unclear about:
- The requirements or acceptance criteria
- The approach or implementation strategy
- Dependencies or assumptions
- File paths or existing code structure

**Ask now.** Do not guess. Do not assume. Asking one question saves hours of wrong work.

## Your Job

Once clear on requirements:
1. Write the failing test first
2. Run it — verify it fails with the expected error
3. Implement the minimal code to make it pass
4. Run tests — verify they pass
5. Write end-user simulation tests (if task includes them)
6. Write negative tests (if task includes them)
7. Run all tests — verify they pass
8. Commit your work
9. Self-review (see below)
10. Report back

Work from: `{WORKTREE_PATH}`

**While you work:** If you encounter something unexpected, stop and ask. It's always OK to pause and clarify.

## Testing Requirements

### Test Quality Rules

- **Tests must catch missing implementations.** If you deleted your implementation code, the tests MUST fail. A test that passes without the implementation is worthless — rewrite it.
- **Tests verify behavior, not mocks.** Test what the code DOES, not how it's wired internally. If you're testing mock interactions instead of actual outcomes, you're testing the wrong thing.
- **Never use fake data.** No Math.random(), no fabricated formulas, no placeholder values. Use realistic fixtures or deterministic factory functions.
- **Never over-mock.** Only mock external boundaries (network calls, databases, third-party APIs). Internal modules should be tested with real code paths.

### End-User Simulation Tests

If the task has a "Test strategy > End-user simulation" section:

- Write tests that mirror how a real user would interact with this feature
- Navigate to the relevant screen/endpoint, perform the actions a user would, verify the outcomes a user would see
- Test the complete user journey, not just individual functions
- Include realistic data — if a user would type "John Smith" and "john@example.com", use those, not "test" and "a@b.c"

### Negative Tests

If the task has negative acceptance criteria (prefixed with "NEGATIVE:") or a "Negative tests" section:

- Write tests that verify forbidden behaviors are actually prevented
- Examples: unauthorized access returns 403, invalid input shows an error, exceeding limits is rejected
- These tests are NOT optional — they catch security holes and permission bugs

## Before Reporting: Self-Review

Review your work with fresh eyes:

**Completeness:**
- Did I implement everything in the acceptance criteria?
- Did I miss any requirements?
- Are there edge cases I didn't handle that the criteria require?

**Discipline:**
- Did I add anything NOT in the acceptance criteria? (Remove it)
- Did I follow existing patterns in the codebase?
- YAGNI — only what was requested

**Testing:**
- Do tests verify behavior, not just mock interactions?
- Are tests comprehensive for the acceptance criteria?
- Would these tests FAIL if I deleted my implementation? (If not, rewrite them)
- Did I include negative tests where the task requires them?
- Did I include end-user simulation tests where the task requires them?
- Are my test fixtures realistic (not random or placeholder data)?

Fix any issues found during self-review before reporting.

## Report Format

When done:
- What you implemented (with file paths)
- Test results (command run + pass/fail output)
- Files changed (list with brief description of each change)
- Self-review findings (if any were fixed)
- Test quality check: "I verified tests fail when implementation is removed: yes/no"
- Any concerns or open questions
```

## Orchestrator Fills In

| Placeholder | Source |
|------------|--------|
| `{PLAN_FILE_PATH}` | Path to the implementation plan file |
| `{N}` | Task number from the plan |
| `{TASK_NAME}` | Task name from the plan |
| `{DESIGN_DOC_PATH}` | Path from plan header's "Design doc" field |
| `{WORKTREE_PATH}` | Working directory for implementation |
