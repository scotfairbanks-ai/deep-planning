# Deep Planning

A skill system for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that turns ideas into validated designs and robust implementation plans, then executes them with quality gates that prevent the most common AI-assisted development failures.

## The Problem

AI coding assistants are great at writing individual functions but struggle with complex, multi-step projects:

- **Information loss:** When an orchestrator paraphrases a plan for subagents, details get dropped. The implementation silently drifts from the design.
- **Skipped quality gates:** Gap analysis and edge case review don't happen unless you manually prompt for them every time.
- **Transition failures:** Multi-skill workflows (design → plan → execute) break when the AI forgets to invoke the next skill.
- **Incomplete plans:** Tasks reference "see above" or assume context that isolated subagents don't have.
- **Infinite review loops:** Review failures loop endlessly with no escalation or retry ceiling.
- **Lost progress:** Session interruptions lose all knowledge of completed work.
- **Silent integration failures:** Tasks pass in isolation but break each other.
- **False-confidence tests:** Tests exist and pass but don't catch real bugs.

## The Solution

Two skills that address these failures directly.

### `design-and-plan`

Combines design exploration and plan writing into **one continuous flow** — no skill transition to forget. Built-in hard gates enforce:

1. **Context exploration** before asking questions
2. **Collaborative design** with user approval per section
3. **Visual design** for UI features (wireframes, component trees, interaction flows)
4. **Framework constraints** surfaced early (React, React Native, Vue, etc.)
5. **Mandatory gap analysis** — 2 passes: edge cases/error states, then integration points
6. **Self-contained task planning** where every task includes all context needed
7. **Test strategy per task** — unit tests, end-user simulation tests, and negative tests
8. **Plan review** that reads back the full plan to catch ambiguity, gaps, and conflicts

### `execute`

Runs implementation plans with subagent isolation and quality reviews. Key innovation: **subagents read the plan file directly** instead of receiving paraphrased context from the orchestrator.

- **Pre-execution checklist** verifies the environment before starting
- **Per-task git SHA checkpoints** enable safe rollback
- Per-task pipeline: implementer → spec review → quality review → (framework review) → regression tests
- **Retry policy with escalation** — no infinite review loops
- **Cross-task regression testing** catches integration failures immediately
- **Progress persistence** — completed tasks are marked in the plan file for resumability
- Every 3 tasks: **drift check** against original design with design doc updates
- Hard stops on review failures — no "close enough"

## Installation

Copy the skills to your Claude Code skills directory:

```bash
cp -r skills/* ~/.claude/skills/
```

## Usage

### Design and Planning

When starting a new feature or non-trivial task:

```
Use the design-and-plan skill to design and plan [your feature]
```

The skill walks through: exploration → design → gap analysis → planning → review → save.

Plans are saved to `docs/plans/` with both a design doc and implementation plan.

### Execution

After saving a plan:

```
Use the execute skill to implement docs/plans/2025-01-15-feature-plan.md
```

The skill dispatches isolated subagents per task with spec, quality, and framework reviews.

### Resuming After Interruption

If a session is interrupted, start a new session and run execute on the same plan file. The skill scans for completion markers and resumes from the first incomplete task.

## Subagent Pipeline

Each task passes through up to 5 review stages:

| Stage | What It Checks | When |
|-------|---------------|------|
| **Implementer** | Builds the feature with TDD | Every task |
| **Spec Reviewer** | Acceptance criteria met, tests validate real behavior | Every task |
| **Quality Reviewer** | Code quality, error handling, test quality, performance | Every task |
| **Framework Reviewer** | Framework-specific patterns, accessibility, composition | UI tasks only |
| **Regression Tests** | Full test suite — no previous tasks broken | Every task |
| **Drift Check** | Implementation matches original design | Every 3rd task |

## Works With

These skills complement [superpowers](https://github.com/anthropics/superpowers-claude-code) and optionally reference several of its skills during execution:

- `superpowers:using-git-worktrees` — workspace isolation
- `superpowers:test-driven-development` — TDD discipline in subagents
- `superpowers:verification-before-completion` — final verification gate
- `superpowers:finishing-a-development-branch` — branch cleanup

Deep Planning works without superpowers installed — you'll just handle worktree setup and branch management yourself.

## Customization

### Save Location

By default, plans save to `docs/plans/YYYY-MM-DD-<topic>-{design,plan}.md`. To change this, update the Phase 6 section in `design-and-plan/SKILL.md`.

### Project-Specific Rules

Add project-specific planning rules to your `CLAUDE.md`:

```markdown
## Planning
- Always include database migration tasks for schema changes
- Error handling must follow our retry-with-backoff pattern
- All new endpoints need rate limiting
```

These rules are picked up during the exploration phase and influence both design and planning.

### Framework Reviewer

The framework reviewer supports React, React Native, Vue, and general web patterns out of the box. To add support for other frameworks (Svelte, Angular, Flutter, etc.), edit `execute/framework-reviewer-prompt.md` and add a section for the framework's specific patterns.

## Why This Exists

See [docs/why-this-exists.md](docs/why-this-exists.md) for the detailed analysis of 9 failure modes in multi-skill planning workflows and how Deep Planning addresses each one.

## License

MIT — see [LICENSE](LICENSE)
