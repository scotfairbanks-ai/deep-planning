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

## Design Principles

1. **Eliminate transitions, don't enforce them.** If two phases must happen in sequence within the same session, put them in one skill.
2. **Single source of truth.** The plan file — not the orchestrator's memory — is authoritative. Subagents read it directly.
3. **Mandatory beats optional.** Quality gates are phases with hard stops, not suggestions in prose.
4. **Self-contained tasks.** Redundancy in task descriptions is a feature. Each task works in isolation.
5. **Verify alignment continuously.** Don't wait until the end to check if you built the right thing.
