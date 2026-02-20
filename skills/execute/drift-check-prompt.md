# Drift Check Subagent Prompt Template

Dispatch every 3 completed tasks. Compares the current implementation against the original design to catch gradual divergence.

## Template

```
You are checking whether the implementation is still aligned with the original design.

## Your First Steps

1. Read the design doc: `{DESIGN_DOC_PATH}`
2. Read the implementation plan: `{PLAN_FILE_PATH}`
3. Read all source files created or modified so far (check git log for the file list)

## Check For

**Architectural drift:**
- Are components organized as the design specified?
- Are data flows matching the design?
- Are interfaces between components as designed?
- Are the right patterns being used (not substituted for convenience)?

**Scope drift:**
- Has anything been added that wasn't in the design?
- Has anything been dropped or simplified beyond what the design allows?
- Are there "temporary" workarounds that deviate from the design?

**Accumulation drift:**
- Do small per-task deviations add up to a significant divergence?
- Has the overall shape of the implementation shifted from the design?

## Report

- **NO DRIFT** — Implementation matches design. Brief summary of alignment.
- **MINOR DRIFT** — Small deviations that may be acceptable. List each with whether it's intentional improvement or accidental divergence.
- **SIGNIFICANT DRIFT** — Implementation has diverged from design in ways that need discussion. List each divergence with:
  - What the design says
  - What was actually built
  - Which tasks contributed to the drift
  - Recommendation: realign to design, or update design to match implementation
```

## Orchestrator Fills In

| Placeholder | Source |
|------------|--------|
| `{DESIGN_DOC_PATH}` | Path from plan header's "Design doc" field |
| `{PLAN_FILE_PATH}` | Path to the implementation plan file |
