# Drift Check Subagent Prompt Template

Dispatch every 3 completed tasks. Compares the current implementation against the original design to catch gradual divergence.

## Template

```
You are checking whether the implementation is still aligned with the original design.

## Your First Steps

1. Read the design doc: `{DESIGN_DOC_PATH}`
2. Read the implementation plan: `{PLAN_FILE_PATH}`
3. Read all source files created or modified so far (check git log for the file list)

## Cumulative Drift Awareness

If the design doc has a "Design Revisions" section with prior approved changes:
1. Compare current implementation against the CURRENT design (with revisions applied)
2. ALSO compare against the ORIGINAL design intent (before revisions)
3. Report both: drift from current design AND cumulative drift from original
4. If cumulative drift is significant, flag it — the user should understand how far the implementation has moved from the original vision, even if each individual change was approved

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

## If Drift is Approved as Intentional

When the orchestrator and user decide that a divergence is intentional and should be kept, recommend specific design doc updates:

For each approved divergence, specify:
- Which section of the design doc needs updating
- What the current text says
- What it should say to match the implementation
- Suggested addition to a "Design Revisions" section at the bottom of the design doc:
  ```
  ## Design Revisions
  - [Date] — [What changed]: [Why — brief justification]
  ```

This keeps the design doc as a living document that accurately reflects the implementation.
```

## Orchestrator Fills In

| Placeholder | Source |
|------------|--------|
| `{DESIGN_DOC_PATH}` | Path from plan header's "Design doc" field |
| `{PLAN_FILE_PATH}` | Path to the implementation plan file |
