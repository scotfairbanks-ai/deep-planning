# Why Deep Planning Exists

## The Problem With Multi-Skill Planning Workflows

AI coding assistants have a well-understood failure mode: they're great at writing individual functions but struggle with complex, multi-step projects. "Planning" skills attempt to solve this by structuring work into phases, but they introduce their own failure modes.

### Failure 1: Skill Transition Loss

Multi-skill workflows (e.g., `brainstorm` → `write-plan` → `execute`) require the AI to invoke the next skill at the right time. In practice:

- The instruction to invoke the next skill is at the bottom of a long conversation
- By the time the AI finishes the current phase, it often forgets to transition
- There's no enforcement mechanism — it's just a text instruction saying "now invoke X"
- Hooks can't fix this: parent session hooks don't fire for subagent tool calls ([GitHub #26923](https://github.com/anthropics/claude-code/issues/26923), [#20243](https://github.com/anthropics/claude-code/issues/20243))

**Solution:** Merge phases that share a session into a single skill. `design-and-plan` covers design through plan-writing as one continuous flow with no transition to forget.

### Failure 2: Information Loss at the Subagent Boundary

When executing plans with subagents, the orchestrator must pass task context to each subagent. Most approaches have the orchestrator paste task text into the subagent prompt. This creates a lossy intermediary:

- Orchestrator summarizes or paraphrases (losing detail)
- Orchestrator omits context it thinks is "obvious"
- Orchestrator misses acceptance criteria or constraints
- The subagent builds what the orchestrator described, not what the plan specified

**Solution:** Subagents read the plan file directly. The orchestrator tells the subagent "read this file, implement Task N." The plan file is the single source of truth, not the orchestrator's interpretation of it.

### Failure 3: Implicit Quality Gates

Most planning workflows say "review for edge cases" or "check for gaps" but don't make these mandatory phases with explicit stop conditions. The result: the AI skips them because they feel optional, especially when the conversation is long.

**Solution:** Hard gates between phases. Each phase has an explicit "DO NOT proceed until [condition]" requirement. Gap analysis is two mandatory passes with user sign-off, not a suggestion buried in instructions.

### Failure 4: Tasks That Assume Context

Plan tasks often say "modify the file from Task 1" or "using the pattern described above." When a subagent implements Task 3 in isolation, it has no idea what "the file from Task 1" refers to.

**Solution:** Every task must be self-contained. Repeat the file paths, repeat the context, repeat the acceptance criteria. A subagent should be able to implement any single task having read only that task and the design doc. Redundancy in task descriptions is a feature, not a problem.

### Failure 5: Gradual Drift

Over 5-10 tasks, small deviations accumulate. Each task might pass its own spec review, but the aggregate implementation diverges significantly from the original design. By the time you notice, it's expensive to fix.

**Solution:** Drift checks every 3 tasks. A dedicated subagent reads the original design doc and compares it to the current state of the implementation. Divergence is caught early when it's cheap to correct.

### Failure 6: Infinite Review Loops

When a spec or quality review fails, the system dispatches the implementer to fix the issues, then re-runs the review. But some tasks have ambiguous acceptance criteria or fundamentally wrong implementation approaches. Without a retry ceiling, the system loops indefinitely — burning tokens, context, and time with no resolution.

**Solution:** Explicit retry policy with escalation. Spec review gets 2 retries before the task spec is rewritten. Quality review gets 2 retries before escalating to the user. Any combination exceeding 4 review cycles on one task triggers a hard stop. The system never loops silently.

### Failure 7: Lost Progress on Interruption

AI coding sessions can be interrupted — context window compression, crashes, user stepping away. If progress isn't persisted, all completed work is invisible to a new session. The system might re-execute already-completed tasks, or worse, skip them and proceed with an incomplete foundation.

**Solution:** Completion markers in the plan file itself. After each task passes all reviews, its acceptance criteria checkboxes are marked complete and a status annotation is added with the git SHA. A new session can scan the plan file, find the first incomplete task, and resume from there.

### Failure 8: Integration Failures Hidden by Task Isolation

Each task is verified in isolation — its own tests pass, its own spec review passes. But Task 5 might break Task 2's functionality without anyone noticing until the final test run. By then, the root cause is hard to trace because multiple tasks have been layered on top.

**Solution:** Cross-task regression testing after every task. Once a task passes all reviews, run the full test suite — not just the current task's tests. If anything regresses, the cause is the task that just completed, and it's fixed immediately before proceeding.

### Failure 9: Tests That Don't Catch Bugs

The most insidious failure: tests exist and pass, but they don't actually validate anything meaningful. Tests that over-mock, use fake data, or test implementation details instead of behavior can pass even when the implementation is deleted. They provide false confidence and hide real bugs.

**Solution:** Multi-layered test quality enforcement:
- **Implementer** must verify tests fail when implementation is removed
- **Spec reviewer** must identify which test covers each acceptance criterion and flag tautological tests
- **Quality reviewer** checks for over-mocking, flaky patterns, and test isolation
- **Negative tests** are mandatory for security/permission/validation criteria
- **End-user simulation tests** are mandatory for user-facing features, mirroring real user behavior
- **Fake data is banned** — no Math.random(), no placeholder values, no fabricated formulas

### Failure 10: Silent Coverage Loss at the Design-to-Task Boundary

A design doc describes 15 components, 8 error handling strategies, and 12 interaction flows. The planner creates tasks for most of them — but silently drops a few. Each task passes its own spec review because the spec reviewer only checks the task's acceptance criteria, not whether the design is fully covered. Nobody notices until the feature ships incomplete.

Similarly, gap analysis surfaces edge cases and integration issues that get added to the design doc — but never become acceptance criteria or tasks. The gap was identified but never implemented.

**Solution:** Design-to-task traceability with explicit coverage verification:
- **Design element numbering** — every component, flow, and strategy in the design gets an ID (D1, D2, D3)
- **Coverage matrix** — after writing tasks, every design element must map to at least one task. Uncovered elements are flagged.
- **Gap-to-criteria tracing** — every gap analysis finding must map to at least one acceptance criterion
- **Plan review validation** — Phase 5 internal validation explicitly checks design coverage and gap coverage before presenting to the user
- **Transparency** — any changes made during internal validation are disclosed to the user before approval

### Failure 11: Anchoring Bias in Gap Analysis

When the same agent runs multiple gap analysis passes, it anchors to its initial findings. Pass 2 finds issues similar to Pass 1 — not fundamentally different ones. The agent's mental model of the design's weaknesses doesn't shift between passes.

**Solution:** Independent subagent gap analysis with convergence:
- Each pass dispatches a fresh subagent with no knowledge of previous findings
- Each subagent explores both the design doc AND the actual codebase (not just the plan)
- Between passes, the orchestrator fixes critical/important findings and saves the updated design doc
- Later subagents analyze a progressively stronger design, naturally finding deeper issues
- The loop continues until no critical findings remain (convergence) or a ceiling is reached (5 passes max → escalate to user)

## Design Principles

1. **Eliminate transitions, don't enforce them.** If two phases must happen in sequence within the same session, put them in one skill.
2. **Single source of truth.** The plan file — not the orchestrator's memory — is authoritative. Subagents read it directly.
3. **Mandatory beats optional.** Quality gates are phases with hard stops, not suggestions in prose.
4. **Self-contained tasks.** Redundancy in task descriptions is a feature. Each task works in isolation.
5. **Verify alignment continuously.** Don't wait until the end to check if you built the right thing.
6. **Fail fast with escalation.** Review loops have ceilings. When something isn't working, stop and escalate — don't loop silently.
7. **Persist progress.** Completed work is recorded in the plan file. Interruptions don't lose progress.
8. **Test integration continuously.** Don't wait for the end to discover that tasks broke each other.
9. **Tests must catch real bugs.** A test that passes without the implementation is worse than no test — it provides false confidence.
10. **Trace design to tasks explicitly.** Every design element must map to a task. Every gap finding must map to a criterion. Coverage matrices catch what intuition misses.
11. **Fresh eyes beat repeated passes.** Independent subagents with no knowledge of previous findings produce more diverse analysis than the same agent running multiple passes.
