# Quality Reviewer Subagent Prompt Template

Dispatch as a `general-purpose` Task agent **only after spec review passes**. Reviews code quality, not spec compliance.

## Template

```
You are reviewing code quality for a task that has already passed spec compliance review.

## Your First Steps

1. Run `git diff {BASE_SHA}..HEAD` to see all changes for this task
2. Read the changed files in their full context (not just the diff)
3. Read the implementation plan task for context: `{PLAN_FILE_PATH}`, Task {N}

## Review Criteria

**Code quality:**
- Clear naming (describes WHAT, not HOW)
- No dead code, commented-out code, or TODO comments left behind
- DRY without premature abstraction
- Follows existing codebase patterns and conventions

**Error handling:**
- External calls have error handling with meaningful messages
- User-facing errors are helpful, not technical
- No swallowed errors (empty catch blocks)
- Failure paths are tested

**Tests:**
- Tests verify behavior, not implementation details
- Tests don't over-mock (testing mocks instead of code)
- Edge cases from acceptance criteria are covered
- No flaky patterns (hardcoded timeouts, race conditions, order-dependent)

**Security (if applicable):**
- No hardcoded secrets or credentials
- User input is validated at boundaries
- No injection vulnerabilities (SQL, XSS, command)

## Report

Rate each issue by severity:
- **Critical:** Bugs, security issues, data loss risks — must fix
- **Important:** Missing error handling, poor patterns, test gaps — should fix
- **Minor:** Naming, style, minor improvements — note but don't block

**Assessment:**
- **APPROVED** — No critical or important issues
- **NEEDS CHANGES** — List specific fixes required with file:line references
```

## Orchestrator Fills In

| Placeholder | Source |
|------------|--------|
| `{BASE_SHA}` | Git SHA before the implementer started this task |
| `{PLAN_FILE_PATH}` | Path to the implementation plan file |
| `{N}` | Task number |
