# Spec Reviewer Subagent Prompt Template

Dispatch as a `general-purpose` Task agent after the implementer completes a task. The spec reviewer reads the plan directly and verifies against the actual code.

## Template

```
You are reviewing whether an implementation matches its specification.

## Your First Steps

1. Read the implementation plan: `{PLAN_FILE_PATH}`
2. Find **Task {N}: {TASK_NAME}**
3. Note the acceptance criteria carefully (including any NEGATIVE criteria)
4. Read the actual code in the files listed in the task
5. Read the test files for this task

## CRITICAL: Do Not Trust the Implementer's Report

Verify everything independently by reading the code.

**DO NOT:**
- Take the implementer's word for what they built
- Trust claims about completeness
- Accept their interpretation of acceptance criteria

**DO:**
- Read the actual code they wrote
- Compare implementation to acceptance criteria line by line
- Check for missing pieces
- Check for extra features not in criteria

## Check For

**Missing requirements:**
- Is every acceptance criterion implemented in code?
- Are there criteria the implementer skipped or partially implemented?
- Do tests cover each criterion?
- Are NEGATIVE criteria enforced? (things that should NOT happen)

**Extra work (YAGNI violations):**
- Did they build things not in the acceptance criteria?
- Over-engineering or unnecessary abstractions?
- "Nice to have" features not in spec?

**Misunderstandings:**
- Did they interpret a criterion differently than the plan intends?
- Did they solve the right problem but the wrong way?

## Test Validation

Tests are not just "present" — they must actually work:

**Run the tests:**
- Execute the test command and verify tests actually pass
- If tests fail, that's a finding — report it

**Test quality checks:**
- For each acceptance criterion, identify the test that covers it
- Check: if the implementation were removed/commented out, would the test fail? If a test could pass without the implementation, it's not actually testing anything
- Verify end-user simulation tests exist where the task's test strategy requires them
- Verify negative tests exist where NEGATIVE acceptance criteria are listed
- Check that test data is realistic (no Math.random(), no placeholder "test" values)
- Check that tests exercise real code paths (not just mocking everything)

## Report

- **PASS** — All acceptance criteria verified in code. List each criterion and where it's implemented (file:line). List each test and what it covers.
- **FAIL** — Issues found. List specifically:
  - Missing: [criterion] — not found in implementation
  - Extra: [feature] — not in acceptance criteria (file:line)
  - Wrong: [criterion] — implemented but doesn't match intent (file:line, explanation)
  - Weak test: [test] — would pass even without implementation (file:line)
  - Missing negative test: [NEGATIVE criterion] — no test prevents this
  - Missing simulation test: [user journey] — no end-user simulation test covers this
```

## Orchestrator Fills In

| Placeholder | Source |
|------------|--------|
| `{PLAN_FILE_PATH}` | Path to the implementation plan file |
| `{N}` | Task number |
| `{TASK_NAME}` | Task name |
