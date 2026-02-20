# Plan Quality Checker Subagent Prompt Template

Dispatch as a `general-purpose` Task agent during Phase 5 plan validation. Runs in parallel with coverage-verifier and test-strategy-auditor. Checks the plan's internal consistency, clarity, and implementability.

## Template

```
You are reviewing an implementation plan for internal quality issues that would cause problems during execution. You are NOT checking design coverage (another reviewer handles that) or test strategy (another reviewer handles that). Focus on plan structure, clarity, and consistency.

## Your First Steps

1. Read the implementation plan: `{PLAN_FILE_PATH}`
2. Read the design doc for context: `{DESIGN_DOC_PATH}`

## Quality Checks

Work through every check below for every task in the plan.

### 1. Acceptance Criteria Clarity
For each acceptance criterion in every task:
- Is it **objectively verifiable**? ("User sees a confirmation" = good. "UX feels smooth" = bad.)
- Is it **specific enough** that two different engineers would implement the same thing?
- Does it avoid weasel words? ("should handle errors appropriately" = too vague)
- If you read ONLY this criterion, could you write a pass/fail test for it?

Flag: Subjective, ambiguous, or untestable criteria.

### 2. Self-Containment
For each task:
- Does it contain ALL file paths needed? (no "the file from Task 2")
- Does it include sufficient context? (no "as described above" or "see Task 1")
- Could a developer implement this task having read ONLY this task + the design doc?
- Are there implicit assumptions not stated in the task?

Flag: Cross-references to other tasks, missing file paths, assumed context.

### 3. Dependency Ordering
- Are dependencies explicitly stated in every task that has them?
- Is the ordering correct? (no task depends on something that comes after it)
- Are there hidden dependencies? (Task 5 modifies a file that Task 3 creates, but no dependency is declared)
- Would changing the order of any two adjacent tasks cause problems?

Flag: Missing dependencies, circular dependencies, incorrect ordering.

### 4. Conflicting Criteria
Check across ALL tasks:
- Do any two acceptance criteria contradict each other?
- Does any task's criteria conflict with the design doc?
- Are there tasks that would modify the same code in incompatible ways?
- Do any error handling strategies conflict? (one task retries, another fails fast)

Flag: Specific conflicting criteria with task numbers and criterion text.

### 5. Error Handling Completeness
For each task:
- Every external call (API, database, file system) has an error handling requirement in the criteria
- Every user input has validation requirements
- Failure paths are specified (what happens on error — not just "handle errors")
- Error messages are user-facing, not technical (if applicable)

Flag: External calls without error handling criteria, vague error handling.

### 6. Risk Distribution
- Are high-risk tasks front-loaded or back-loaded? (front-loading is better — fail fast)
- Do high-risk tasks have proportionally more detailed criteria and test strategy?
- Are there clusters of high-risk tasks? (risky — consider splitting or reordering)
- Is there a task that is disproportionately large or complex? (consider splitting)

Flag: Risk ordering issues, under-specified high-risk tasks.

### 7. Implementation Steps Quality
For each task's Steps section:
- Do steps follow TDD? (test first, verify fail, implement, verify pass)
- Are commit messages meaningful?
- Are steps specific enough to follow without guessing?
- Do steps match the acceptance criteria? (no step that doesn't serve a criterion)

Flag: Steps that skip TDD, steps that don't align with criteria.

## Report Format

```
### Quality Issues Found

Rate each by severity:
- **Critical**: Would cause execution failure or ambiguous implementation
- **Important**: Would cause confusion or suboptimal execution
- **Minor**: Style or minor clarity improvements

#### Critical Issues
1. **[Check category] Task N: Issue title**
   - What: [description]
   - Fix: [specific recommendation]

#### Important Issues
[same format]

#### Minor Issues
[same format]

### Plan Statistics
- Total tasks: N
- Complexity distribution: X low, Y medium, Z high
- Risk distribution: X low, Y medium, Z high
- Tasks with dependencies: N
- Deepest dependency chain: N tasks

### Assessment
- **CLEAN** — No critical or important issues
- **NEEDS FIXES** — List of specific fixes required before execution
```
```

## Orchestrator Fills In

| Placeholder | Source |
|---|---|
| `{PLAN_FILE_PATH}` | Path to plan file |
| `{DESIGN_DOC_PATH}` | Path to design doc |
