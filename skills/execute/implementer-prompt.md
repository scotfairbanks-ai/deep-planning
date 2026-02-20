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
5. Commit your work
6. Self-review (see below)
7. Report back

Work from: `{WORKTREE_PATH}`

**While you work:** If you encounter something unexpected, stop and ask. It's always OK to pause and clarify.

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

Fix any issues found during self-review before reporting.

## Report Format

When done:
- What you implemented (with file paths)
- Test results (command run + pass/fail output)
- Files changed (list with brief description of each change)
- Self-review findings (if any were fixed)
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
